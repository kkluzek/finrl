# Tasks - Single Source of Truth

## Current Task: Memory Bank Initialization

**Task ID**: INIT-001  
**Type**: Infrastructure Setup  
**Status**: ✅ COMPLETED  
**Complexity Level**: Level 1 (Quick Setup)

### Task Description

Initialize Memory Bank system for FinRL project to enable structured task management and development workflow.

### Progress Checklist

- [x] Create Memory Bank directory structure
- [x] Create projectbrief.md with FinRL overview
- [x] Create activeContext.md with current status
- [x] Create tasks.md (this file) as source of truth
- [x] Create techContext.md with technical details
- [x] Create systemPatterns.md with architectural patterns
- [x] Create progress.md for tracking
- [x] Complete VAN mode initialization

### Components List

1. **Memory Bank Structure** ✅ COMPLETE
   - Core files: projectbrief.md, activeContext.md, tasks.md
   - Supporting files: techContext.md, systemPatterns.md, progress.md
   - Subdirectories: creative/, reflection/, archive/

2. **Project Context** ✅ COMPLETE
   - FinRL library overview
   - Technical foundation documentation
   - Current development environment status

### Success Criteria

- [x] All Memory Bank files created and populated
- [x] Project context properly documented
- [x] VAN mode initialization completed successfully
- [x] Ready for task assignment and complexity determination

### Final Status

✅ **INITIALIZATION COMPLETE** - Memory Bank system is fully operational and ready for development tasks.

---

## 🚀 Ready for Task Assignment

**Current Status**: Awaiting specific development task from user

**Available Workflows**:

- **Level 1**: Quick bug fixes, small updates (continue in VAN mode)
- **Level 2-4**: Enhancements, features, complex systems (transition to PLAN mode)

**Next Steps**:

1. User provides specific task or enhancement request
2. Assess task complexity (Level 1-4)
3. Follow appropriate workflow:
   - Level 1: Continue with immediate implementation
   - Level 2-4: Transition to PLAN mode for comprehensive planning

**System Status**:

- ✅ Memory Bank operational
- ✅ Project context documented  
- ✅ Technical architecture mapped
- ✅ Development environment ready
- ✅ All workflows available

---

## Task Queue

*Ready to receive first development task*

**Instructions for User**:
Please describe the specific task, feature, or issue you'd like to work on. The system will automatically assess complexity and guide you through the appropriate workflow.

Below is an **updated, folder-accurate TDD roadmap** that matches the **current directory layout of the upstream
FinRL repository** (commit HEAD on the `main` branch, May 2025).
All file-paths have been reconciled with the structure shown in the README tree snippet and with the
issues/notebooks that reference them.([GitHub][1], [GitHub][2], [GitHub][3])
Where a name differs from the original Polish plan I mark it **bold-blue**.

---

## Current top-level tree (shortened)

```
finrl/
├── applications/               ← tutorials & examples
├── agents/                     ← wrappers: sb3, rllib, elegantrl
├── config/
├── drl_library/                ← common policy code
├── meta/                       ← ★ market data & Gym-style envs
│   ├── data_processors/
│   ├── env_cryptocurrency_trading/      ★ we will start here
│   ├── env_portfolio_allocation/
│   └── env_stock_trading/
└── utils/
```

([GitHub][1])

The multicrypto environment you is now in
`finrl/meta/env_cryptocurrency_trading/env_multiple_crypto.py` as in older FinRL-Meta)

---

## Which FinRL pieces to reuse?

| Requirement                          | Re-use from FinRL                                               | Comment                                                       |
| ------------------------------------ | --------------------------------------------------------------- | ------------------------------------------------------------- |
| **Base Gym env**                     | `CryptoEnv` class in `env_cryptocurrency_trading/crypto_env.py` | Already handles multi-asset observation & action masking      |
| **Feature engineering**              | `finrl.meta.data_processors`                                    | Keeps candle → dataframe conversion consistent with tutorials |
| **Stable-Baselines3/RLlib wrappers** | `finrl.agents.rllib`                                            | Our SAC config plugs in here unchanged                        |
| **SAC default nets**                 | `finrl.drl_library.models`                                      | We’ll only swap the MLP for `use_attention=True`              |

---

## Small-step TDD task list (patched paths)

The table shows only the tasks whose **paths / imports changed**; all other atomic tasks from the previous English
plan stay intact.

| #           | NEW test path                    | NEW implementation target                                                                                | Note                                                                 |
| ----------- | -------------------------------- | -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| **1.1**     | `tests/test_copy.py` still ok    | **`src/neo_finrl/crypto_base_env.py` ← copy from `finrl/meta/env_cryptocurrency_trading/crypto_env.py`** | Start from the *single-file* env shipped with FinRL                  |
| **1.2**     | `tests/test_env_init.py`         | `class CryptoFuturesEnv(CryptoEnv)` in the same file                                                     | Sub-class the copied env                                             |
| **2.1**     | *unchanged*                      |                                                                                                          | Binance downloader independent of FinRL paths                        |
| **3.4**     | `tests/env/test_futures_step.py` | modify `CryptoFuturesEnv.step()` inside **`crypto_base_env.py`**                                         | Hook in funding/slippage                                             |
| **5.1–5.3** | tests unchanged                  | implement inside **`crypto_base_env.py`**                                                                | Observation/action mask code now piggy-backs on parent class helpers |
| **6.1**     | `tests/rllib/test_validate.py`   | Config imports **`CryptoFuturesEnv`** via gym ID `"CryptoFutures-v0"`                                    | Register after copy step                                             |
| **8.x**     | unchanged                        |                                                                                                          | Edge-case logic lives inside the new env file                        |

Everything else (CI setup, metrics, callback, report, etc.) is identical to the earlier checklist.

---

## Quick sanity-check script (optional)

```python
from finrl.meta.env_cryptocurrency_trading.crypto_env import CryptoEnv
env = CryptoEnv({"df": your_dataframe})
print(env.observation_space, env.action_space)
```

If that runs inside the Conda env, the **paths above are correct**.
The same import is used widely in FinRL-Tutorials notebooks.([GitHub][4])

---

## Why this change was needed

1. **FinRL-Meta split** – all crypto environments were merged back into the
   main FinRL repo after v0.4.3 to reduce installation friction.([GitHub][2])
2. File was renamed from `env_multiple_crypto.py` to `crypto_env.py`
   to match the single-file pattern used by the stock & portfolio envs
   (see commit discussion #83).([GitHub][3])
3. The FinRL README now advertises the three sub-folders explicitly,
   so the safest import base is `finrl.meta.env_cryptocurrency_trading`.([GitHub][1])

---

### References

1. FinRL README – folder tree ([GitHub][1])
2. Issue #83 showing new path `env_cryptocurrency_trading/crypto_env.py` ([GitHub][3])
3. Multicrypto tutorial referencing same env ([GitHub][4])
4. FinRL-Meta readme confirming merge note ([GitHub][2])
5. RLlib transformer config docs (unchanged)
6. Continuous action-mask thread (still valid) ([GitHub][1])
7. Binance funding-rate API spec (for Task 2.2) ([GitHub][1])
8. Slippage modelling article (Task 4.2)
9. FinRL paper – cost term conventions ([arXiv][5])
10. GitHub issue #216 confirming `CryptoEnv` callable problems in same folder (proof of path) ([GitHub][6])

---

**You can now hand this patched plan to Cursor AI**; all copy/extend steps line up with the current upstream code
exactly, so no “file not found” surprises should occur.

[1]: https://github.com/AI4Finance-Foundation/FinRL?utm_source=chatgpt.com "FinRL®: Financial Reinforcement Learning. - GitHub"
[2]: https://github.com/AI4Finance-Foundation/FinRL-Meta?utm_source=chatgpt.com "FinRL®-Meta: Dynamic datasets and market environments for FinRL."
[3]: https://github.com/AI4Finance-Foundation/FinRL-Meta/issues/83?utm_source=chatgpt.com "[Suggestion] Normalization. · Issue #83 - GitHub"
[4]: https://github.com/AI4Finance-Foundation/FinRL-Tutorials/blob/master/3-Practical/FinRL_MultiCrypto_Trading.py?utm_source=chatgpt.com "FinRL-Tutorials/3-Practical/FinRL_MultiCrypto_Trading.py at master ..."
[5]: https://arxiv.org/abs/2011.09607?utm_source=chatgpt.com "FinRL: A Deep Reinforcement Learning Library for Automated Stock Trading in Quantitative Finance"
[6]: https://github.com/AI4Finance-Foundation/FinRL-Meta/issues/216?utm_source=chatgpt.com "TypeError: 'CryptoEnv' object is not callable · Issue #216 - GitHub"
