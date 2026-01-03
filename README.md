# GENESIS-AI — AI Trading Platform

GENESIS-AI is a modular research and execution platform for algorithmic trading powered by machine learning. It provides data ingestion, feature engineering, model training, backtesting, paper-trading and live-trading adapters so you can iterate quickly from idea → research → production.

IMPORTANT: This project is provided for educational and research purposes only. Trading involves substantial risk. This repository is NOT financial advice. See the Legal & Risk section below.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Status](#status)
- [Key Features](#key-features)
- [Repository Layout](#repository-layout)
- [Quickstart](#quickstart)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Typical Workflows](#typical-workflows)
  - [Data ingestion](#data-ingestion)
  - [Training](#training)
  - [Backtesting](#backtesting)
  - [Paper trading](#paper-trading)
  - [Live trading](#live-trading)
- [Modeling & Evaluation](#modeling--evaluation)
- [Backtesting engine notes](#backtesting-engine-notes)
- [Deployment](#deployment)
- [Development & Contribution](#development--contribution)
- [Security, Legal & Risk Disclaimer](#security-legal--risk-disclaimer)
- [Acknowledgements & References](#acknowledgements--references)
- [License](#license)

---

## Project Overview

GENESIS-AI aims to accelerate research and disciplined deployment of ML-driven trading strategies by providing:

- Clean, versioned data pipeline connectors (CSV, exchange APIs, historical ticks)
- Feature-store and standard preprocessing/normalization utilities
- Trainer modules for supervised & reinforcement-learning agents (LSTM/Transformer/RL)
- Vectorized backtesting with configurable execution model, commission, slippage
- Paper-trade adapter for simulated live runs and a production adapter for brokers/exchanges
- Experiment logging and model checkpointing

Primarily focused on mid-frequency trading strategies, but adaptable to other horizons.

---

## Status

- Core data pipeline, trainer and backtester: alpha (research-ready)
- Paper trading adapters: experimental
- Live trading adapters: scaffolded (exercise caution; test extensively)
- CI / tests: partial

Contributions are welcome to harden, test and expand adapters.

---

## Key Features

- Modular architecture: swap data sources, models and execution adapters
- Supports supervised prediction (price/returns classification/regression) and RL agents
- Built-in evaluation metrics (Sharpe, Sortino, max drawdown, CAGR, hit rate)
- Config-driven experiments (YAML)
- Experiment logging using MLFlow (configurable) + tensorboard support
- Docker-friendly for reproducible research environments

---

## Repository Layout

- /data/         — sample datasets & ingestion scripts
- /docs/         — design docs and how-tos
- /genesis/      — core package (data, models, trainers, backtest, brokers)
- /notebooks/    — research notebooks
- /scripts/      — convenience scripts (train, backtest, trade)
- /configs/      — example experiment configs
- /tests/        — unit & integration tests

---

## Quickstart

Clone the repo and create an environment:

```bash
git clone https://github.com/malkinz22/GENESIS-AI.git
cd GENESIS-AI
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Populate credentials in `.env` (see Configuration). Prepare or download sample data in `data/`.

Example: run a simple backtest using an example config:

```bash
python scripts/backtest.py --config configs/backtest/mean_reversion.yml
```

Train a model:

```bash
python scripts/train.py --config configs/train/lstm_classifier.yml
```

Run paper-trading (simulated live):

```bash
python scripts/trade.py --config configs/trade/paper.yml
```

---

## Requirements

- Python 3.9+
- pandas, numpy, scikit-learn, torch (or tensorflow), mlflow, matplotlib
- Optional: ccxt (exchange connectors), alpaca-trade-api, ib_insync (brokers)
- Docker (recommended for reproducible runs)

See `requirements.txt` for exact pinned dependencies.

---

## Installation

1. Clone repository
2. Create virtual environment and install dependencies
3. (Optional) Build Docker image:

```bash
docker build -t genesis-ai:latest .
docker run -it --env-file .env -v $(pwd)/data:/app/data genesis-ai:latest
```

---

## Configuration

GENESIS-AI uses YAML configs and environment variables for secrets. Example environment variables (use a `.env` file or system envs):

- EXCHANGE_API_KEY
- EXCHANGE_API_SECRET
- BROKER_API_KEY
- BROKER_API_SECRET
- MLFLOW_TRACKING_URI
- DATA_PATH (default: ./data)
- MODEL_PATH (default: ./models)

Example config snippet (configs/backtest/mean_reversion.yml):

```yaml
experiment:
  name: mean_reversion_m1
  seed: 42

data:
  source: csv
  path: data/BTCUSD_1m.csv
  start: 2020-01-01
  end: 2021-12-31
  features:
    - sma_20
    - sma_50
    - rsi_14

model:
  type: supervised
  algorithm: lstm
  input_size: 10
  hidden_size: 64
  epochs: 50
  batch_size: 256

backtest:
  initial_capital: 10000
  commission: 0.0005
  slippage: 0.0002
  position_size: 0.02
  execution_model: immediate
```

---

## Typical Workflows

Data ingestion
- Run scrapers or converters in `genesis.data.sources` to create cleaned CSV or parquet.
- Use `scripts/ingest.py` to transform raw exchange/historical data into standardized format.

Training
- Compose features in `genesis.data.features`.
- Train models via `scripts/train.py --config <config>`.
- Save model checkpoints to `models/` and log experiments to MLflow.

Backtesting
- Use `scripts/backtest.py` with a config; the engine supports vectorized or event-driven simulations.
- Inspect metrics & trade logs in `results/<experiment>/`.

Paper trading
- Use `scripts/trade.py` with `mode: paper`. Trades are simulated but execution latencies and commission models mimic the live adapter.

Live trading
- Use `scripts/trade.py` with `mode: live` and a properly configured broker adapter.
- Always run with small sizes, monitoring & circuit-breakers enabled.

Example commands:

```bash
python scripts/train.py --config configs/train/transformer_regressor.yml
python scripts/backtest.py --config configs/backtest/transformer_regressor.yml
python scripts/trade.py --config configs/trade/paper.yml
```

---

## Modeling & Evaluation

Supported model families:
- Time-series supervised: LSTM, CNN, Transformer, Gradient-boosted trees
- Reinforcement learning: PPO, DDPG, SAC (adapter pattern to plug RL libs)
- Baselines: Momentum, Mean-reversion, VWAP, Buy-and-hold

Evaluation metrics:
- Cumulative return, CAGR
- Sharpe ratio (annualized)
- Sortino ratio
- Max drawdown
- Win ratio, average win/loss
- Transaction counts, turnover

Use cross-validation and walk-forward analysis. Keep test data strictly out-of-sample.

---

## Backtesting Engine Notes

- Execution model supports:
  - immediate (fills at next bar open)
  - realistic (volume/slippage model)
  - market-sim (order book simulation, if historical L2 data available)
- Configure commission and slippage carefully to avoid overestimating strategy performance.
- Use risk controls: max position size, max portfolio exposure, per-trade stop-loss/TP, emergency kill-switch.

---

## Deployment

- Containerize using provided Dockerfile.
- Use process supervision (systemd, Docker + restart policies) for live adapters.
- Monitor using Prometheus/Grafana, and log trades and model drift.
- Schedule retraining jobs (cron / Airflow) with careful model validation.

---

## Development & Contribution

Contributions welcome — please follow these steps:

1. Fork the repo
2. Create a feature branch
3. Add tests for new functionality
4. Open a pull request with a clear description

See CONTRIBUTING.md (if present) for details.

Please include reproducible examples and small datasets for new features.

---

## Security, Legal & Risk Disclaimer

This repository is for educational research. Live trading with real capital can result in loss of principal. The author(s) are not financial advisors. Always:

- Test strategies extensively with realistic costs and slippage
- Start with small capital when moving to live trading
- Ensure API keys/credentials are stored securely (never commit to git)
- Implement circuit breakers and monitoring
- Consult a licensed financial professional before trading

---

## Acknowledgements & References

- Backtesting libraries: Backtrader, Zipline (inspiration)
- ML tooling: PyTorch, TensorFlow, MLflow
- Exchange APIs: CCXT, Alpaca, IB

Add references to academic papers or blogs relevant to the models you implement.

---

## License

This project is licensed under the MIT License. See LICENSE for details.

---

If you'd like, I can:
- create a ready-to-commit README.md and open a draft PR
- generate example config files for a specific strategy (e.g., BTC mean reversion)
- scaffold a Docker Compose file and GitHub Actions workflow for CI/CD

Tell me which you'd like next.
