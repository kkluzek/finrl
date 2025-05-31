# Technical Context: FinRL

## Technology Stack

### Core Python Dependencies

- **Python**: 3.7+ (Conda environment active)
- **Deep Learning**: PyTorch, TensorFlow (for neural networks)
- **RL Libraries**: Stable Baselines 3, ElegantRL, RLlib
- **Data Processing**: NumPy, Pandas, SciPy
- **Financial Data**: yfinance, alpaca-trade-api, ccxt
- **Visualization**: Matplotlib, Plotly

### Development Environment

- **Package Management**: Poetry (pyproject.toml) + setuptools (setup.py)
- **Environment**: Conda (.conda/ directory present)
- **Testing**: pytest (unit_tests/ directory)
- **Linting**: Pre-commit hooks configured
- **Documentation**: Sphinx (docs/ directory)

### Architecture Overview

#### Core Modules (`finrl/`)

1. **Agents**: DRL algorithm implementations
   - A2C, A3C, DDPG, PPO, SAC, TD3, TRPO
   - ElegantRL and Stable Baselines 3 integrations

2. **Environments**: Financial market simulations
   - Stock trading environments
   - Portfolio optimization environments
   - Cryptocurrency trading environments
   - High-frequency trading environments

3. **Data Processing**: Market data handlers
   - Data downloaders for various sources
   - Preprocessors for feature engineering
   - Technical indicators computation

4. **Meta Components**: System utilities
   - Environment builders
   - Data processors
   - Paper trading simulation

### External Integrations

- **Data Sources**: Yahoo Finance, Alpaca, Binance, CCXT
- **Backtesting**: Pyfolio integration
- **Deployment**: Docker support
- **Version Control**: Git with GitHub workflows

### File Structure Patterns

```
finrl/
├── agents/           # DRL algorithm implementations
├── applications/     # Specific use cases
├── meta/            # Core system components
└── __init__.py

examples/            # Usage demonstrations
unit_tests/         # Test suite
docs/               # Documentation
docker/             # Containerization
```

### Dependencies Management

- **Poetry**: Primary dependency management
- **Requirements.txt**: Fallback pip installation
- **Setup.py**: Package distribution
- **Conda**: Environment isolation

### Development Workflow

- **Testing**: Unit tests with pytest
- **Quality**: Pre-commit hooks for code quality
- **Documentation**: Auto-generated docs with Sphinx
- **CI/CD**: GitHub Actions integration

### Performance Considerations

- **GPU Support**: CUDA-enabled for accelerated training
- **Memory**: Large datasets require efficient handling
- **Parallel Processing**: Multi-environment training support
- **Scalability**: Distributed training capabilities
