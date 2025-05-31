# Creative Phase: CryptoFutures Environment Design

**Task ID**: DEV-001  
**Component**: CryptoFutures Environment  
**Creative Phase Focus**: Design of key components

## 1. Slippage Model Design

### Requirements

- Model should reflect real-world slippage behavior in cryptocurrency futures markets
- Slippage should scale with order size and market liquidity
- Implementation should be computationally efficient

### Design Options

#### Option 1: Linear Slippage Model

```python
def calculate_slippage(order_size, base_slippage, market_liquidity):
    # Simple linear model where slippage increases with order size
    return base_slippage * (1 + order_size / market_liquidity)
```

**Pros**: Simple to implement, computationally efficient  
**Cons**: May not capture non-linear effects in low liquidity situations

#### Option 2: Non-linear Slippage Model

```python
def calculate_slippage(order_size, base_slippage, market_liquidity, exponent=1.5):
    # Non-linear model with exponential scaling
    return base_slippage * (1 + (order_size / market_liquidity) ** exponent)
```

**Pros**: Better captures market impact in low liquidity scenarios  
**Cons**: Requires calibration of exponent parameter

#### Option 3: Market Depth-Based Model

```python
def calculate_slippage(order_size, order_book_depth):
    # Calculate based on simulated order book depth
    remaining_size = order_size
    total_cost = 0
    slippage = 0
    
    for price, size in order_book_depth:
        filled_size = min(remaining_size, size)
        total_cost += filled_size * price
        remaining_size -= filled_size
        
        if remaining_size <= 0:
            break
    
    average_price = total_cost / order_size
    slippage = (average_price / order_book_depth[0][0]) - 1
    return slippage
```

**Pros**: Most realistic model based on actual market mechanics  
**Cons**: Requires order book data or simulation, computationally expensive

### Recommended Approach

A hybrid approach that uses Option 2 by default, but can be extended to Option 3 when order book data is available:

```python
def calculate_slippage(order_size, base_slippage, market_liquidity, order_book=None, exponent=1.5):
    if order_book is not None:
        # Use order book-based calculation when available
        return calculate_order_book_slippage(order_size, order_book)
    else:
        # Fall back to parameterized model
        return base_slippage * (1 + (order_size / market_liquidity) ** exponent)
```

This approach balances realism with computational efficiency and can be easily integrated into the environment.

## 2. Funding Rate Implementation

### Requirements

- Account for periodic funding rate payments
- Apply funding rates to open positions
- Reflect funding in the reward function

### Design Options

#### Option 1: Discrete Funding Events

Handle funding as discrete events at fixed intervals (e.g., every 8 hours in Binance).

```python
def apply_funding(positions, funding_rates, timestamp):
    """Apply funding rates at discrete intervals"""
    if is_funding_time(timestamp):
        for asset_id, position in positions.items():
            position_value = position * current_price[asset_id]
            funding_payment = position_value * funding_rates[asset_id]
            return funding_payment
    return 0
```

**Pros**: Accurately models exchange mechanics  
**Cons**: Creates reward discontinuities at funding times

#### Option 2: Continuous Funding Accrual

Amortize funding payments continuously across all steps.

```python
def apply_continuous_funding(positions, funding_rates, time_fraction):
    """Apply a continuous fraction of the funding rate"""
    total_funding = 0
    for asset_id, position in positions.items():
        position_value = position * current_price[asset_id]
        # Distribute 8-hour funding rate across steps
        funding_payment = position_value * funding_rates[asset_id] * time_fraction
        total_funding += funding_payment
    return total_funding
```

**Pros**: Smoother reward signal, better for learning  
**Cons**: Less accurate to real-world mechanics

#### Option 3: Hybrid Approach

Track accrued funding separately and apply it to rewards, while still modeling discrete funding events.

```python
def track_funding(positions, funding_rates, timestamp, time_fraction):
    """Track funding separately from positions"""
    # Accrue funding continuously
    accrued_funding = 0
    for asset_id, position in positions.items():
        position_value = position * current_price[asset_id]
        accrued_funding += position_value * funding_rates[asset_id] * time_fraction
    
    # Apply funding at funding times
    if is_funding_time(timestamp):
        realized_funding = accrued_funding
        accrued_funding = 0
        return realized_funding, 0
    else:
        return 0, accrued_funding
```

**Pros**: Balances realism with learning stability  
**Cons**: More complex to implement and track

### Recommended Approach

Option 3 (Hybrid Approach) provides the best balance between realistic mechanics and stable learning. We can:

1. Track accrued funding separate from positions
2. Apply funding to account balance at funding times
3. Include accrued (but unrealized) funding in state representation
4. Consider both realized and accrued funding in reward calculation

## 3. Observation Space Design

### Requirements

- Include all relevant information for trading decisions
- Maintain compatibility with the base `CryptoEnv` observation space
- Add futures-specific information (funding rates, leverage, etc.)

### Design Options

#### Option 1: Extended Flat Vector

Simply append futures-specific features to the existing observation vector.

```python
def get_state(self):
    # Get base state from parent class
    base_state = super().get_state()
    
    # Append futures-specific features
    futures_features = np.hstack((
        self.funding_rates * 2**-10,
        self.leverage * 2**-3,
        self.margin_ratio * 2**-3,
        self.accrued_funding * 2**-15
    )).astype(np.float32)
    
    return np.hstack((base_state, futures_features))
```

**Pros**: Simple to implement, compatible with existing models  
**Cons**: Lacks structure, may be harder for model to utilize effectively

#### Option 2: Structured Dictionary Observation

Use a structured dictionary with clear semantic meaning.

```python
def get_state(self):
    return {
        "base_state": super().get_state(),
        "funding_rates": self.funding_rates,
        "leverage": self.leverage,
        "margin_ratio": self.margin_ratio,
        "accrued_funding": self.accrued_funding,
        "liquidation_prices": self.liquidation_prices
    }
```

**Pros**: Clear semantics, easier to debug and interpret  
**Cons**: Requires model architecture that can handle dictionary observations

#### Option 3: Multi-modal Observation

Separate time-series data from scalar features.

```python
def get_state(self):
    return {
        "scalar_features": np.hstack((
            self.cash * 2**-18,
            self.stocks * 2**-3,
            self.leverage * 2**-3,
            self.margin_ratio * 2**-3
        )),
        "time_series": np.vstack((
            self.price_history[-self.lookback:],
            self.tech_indicators[-self.lookback:],
            self.funding_rate_history[-self.lookback:]
        )),
        "position_info": {
            "liquidation_prices": self.liquidation_prices,
            "accrued_funding": self.accrued_funding
        }
    }
```

**Pros**: Most expressive, allows specialized processing for each data type  
**Cons**: Requires custom model architecture, more complex

### Recommended Approach

For compatibility with existing RL algorithms, Option 1 is the most practical starting point:

```python
def get_state(self):
    # Get base state from parent class
    base_state = super().get_state()
    
    # Prepare futures-specific features
    funding_features = []
    for i in range(self.lookback):
        funding_i = self.funding_rate_history[self.time - i]
        normalized_funding_i = funding_i * 2**-10
        funding_features.append(normalized_funding_i)
    
    # Other futures-specific features
    leverage_features = self.leverage * 2**-3
    margin_features = self.margin_ratio * 2**-3
    accrued_funding_features = self.accrued_funding * 2**-15
    
    # Combine all features
    futures_features = np.hstack((
        np.concatenate(funding_features),
        leverage_features,
        margin_features,
        accrued_funding_features
    )).astype(np.float32)
    
    return np.hstack((base_state, futures_features))
```

This approach maintains compatibility with existing RL algorithms while adding the necessary information for futures trading. As we progress, we can evaluate whether a more structured approach (Option 2 or 3) would provide better performance.

## Conclusion

Based on the creative exploration of these key components, we recommend:

1. **Slippage Model**: Implement the hybrid approach with a non-linear slippage model that can be extended to use order book data when available.

2. **Funding Rate Implementation**: Use the hybrid approach that tracks accrued funding separately while still modeling discrete funding events.

3. **Observation Space**: Start with the extended flat vector approach for compatibility, with the option to move to a more structured approach if needed.

These design decisions balance realism, computational efficiency, and compatibility with existing RL algorithms, providing a solid foundation for implementing the cryptocurrency futures trading environment.
