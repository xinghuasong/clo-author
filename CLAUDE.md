# CLAUDE.MD -- Factor Investing & Machine Learning Research

**Project:** 因子投资与机器学习研究
**Institution:** 
**Field:** Finance / Empirical Asset Pricing — Factor Investing, Machine Learning
**Branch:** main

---

## Core Principles

- **Plan first** -- enter plan mode before non-trivial tasks; save plans to `quality_reports/plans/`
- **Verify after** -- run and confirm output at the end of every task
- **Quality gates** -- code score >= 80/100 before commit
- **Auto-memory** -- corrections and preferences are saved automatically via Claude Code's built-in memory system

---

## Getting Started

1. Activate Conda environment: `conda activate base` (Python 3.11.5)
2. Run analysis notebooks in `scripts/Python/` in order
3. See `data/raw/data2026.csv` for the full dataset

---

## Folder Structure

```
factor-research/
├── CLAUDE.MD                    # This file
├── .claude/                     # Rules, skills, agents, hooks
├── data/
│   ├── raw/                     # Original data (data2026.csv)
│   └── cleaned/                 # Processed datasets for analysis
├── scripts/
│   └── Python/                  # Python analysis notebooks & scripts
│       ├── 01_factor_analysis.ipynb   # Factor analysis (descriptive, IC, correlation)
│       ├── 02_ml_modeling.ipynb       # ML modeling (XGBoost, Ridge, SHAP)
│       └── generate_ppt.py            # PPT lecture generation
├── paper/
│   ├── figures/
│   │   ├── factor_analysis/     # Figures from 01_factor_analysis
│   │   └── ml_modeling/         # Figures from 02_ml_modeling
│   ├── tables/
│   │   ├── factor_analysis/     # Tables from 01_factor_analysis
│   │   └── ml_modeling/         # Tables from 02_ml_modeling
│   └── talks/                   # Presentations (.pptx)
├── quality_reports/             # Plans, session logs, reviews
├── explorations/                # Research sandbox
└── templates/                   # Templates
```

---

## Data Fields

| Field | Description | Type |
|-------|-------------|------|
| Date | Trading date (YYYY/MM/DD) | Date |
| Ticker | Stock code (e.g. 000001.SZ) | str |
| Sector | Industry sector (30 sectors) | str |
| MktCap | Market capitalization | float |
| PE_TTM | P/E ratio (trailing 12M) | float |
| PB_MRQ | P/B ratio (most recent quarter) | float |
| DivYield_TTM | Dividend yield | float |
| ROE_TTM | Return on equity | float |
| Margin | Profit margin | float |
| TA_YoY | Total assets YoY growth | float |
| Salse_YoY | Sales YoY growth | float |
| TA_Turnover | Total asset turnover | float |
| MoM_30/60/180/365 | Momentum (1M/2M/6M/12M) | float |
| TurnOver_30/60/365 | Turnover rate (1M/2M/12M) | float |
| Beta_100D | 100-day market beta | float |
| Vol_90D | 90-day volatility | float |
| NVol_90D | 90-day idiosyncratic volatility | float |
| R1W | **Target:** 1-week forward return | float |

---

## Output Organization

Output organization: by-purpose

---

## Current Project State

| Component | File | Status | Description |
|-----------|------|--------|-------------|
| Data | `data/raw/data2026.csv` | ready | 154,644 obs, 263 stocks, 588 dates (2014-2026) |
| Factor Analysis | `scripts/Python/01_factor_analysis.ipynb` | planned | Descriptive stats, IC analysis, factor correlations |
| ML Modeling | `scripts/Python/02_ml_modeling.ipynb` | planned | XGBoost, Ridge, SHAP feature importance |
| Lecture PPT | `paper/talks/factor_research_lecture.pptx` | planned | Chinese lecture slides |

## CTP Trading Pipeline

| Component | Location | Status |
|-----------|----------|--------|
| DeepCTA signals | `D:\ZML\DeepCTA\` | 127因子 + LightGBM → signal_oos.parquet |
| Signal export | `D:\ZML\ctp\ops\export_and_sync.py` | TOP5 → target_position CSV → SCP to cloud |
| Tencent Cloud | `root@49.233.15.221` | 5 services + 3 timers, all active |
| Trade analysis | `D:\ZML\ctp\ops\analyze_trade.py` | Pull trades/positions/PnL from cloud |
| Deploy | `D:\ZML\ctp\deploy_ctp.sh` | One-click deploy to new cloud server |

### Signal → Trade Flow

```
Windows: synthesize_signals.py → signal_oos.parquet
    → export_and_sync.py → target_position.{intv}.completed
    → SCP → 49.233.15.221:/root/CTA/signal/
    → tp_executor.py → ctp_http_service:8084 → CTP order
```

### Risk Controls

| Layer | Controls |
|-------|----------|
| Signal | TOP5 long/short, 3σ winsorize, 15% vol target, low liquidity filter |
| Execution | ≤20 lots/order, 5% daily loss pause, night-filter day-only symbols, 12h signal TTL |
| Trade | Order tracker TTL 300s, post-restart order recovery, trade persistence to SQLite |
| Gateway | API Key auth, 120 req/min, ±1000 max position per order, account binding |
| Monitor | 5min health check, 400s dead log alert, systemd state check, PushPlus |

## Commands

```bash
# Activate environment
conda activate base

# ── CTP 交易管线 ──

# 信号生成 (Windows)
cd D:\ZML\DeepCTA\signal && python synthesize_signals.py

# 信号转换 + 上传腾讯云
cd D:\ZML\ctp && python ops\export_and_sync.py

# 交易分析
cd D:\ZML\ctp && python ops\analyze_trade.py

# 检查腾讯云管线状态
ssh root@49.233.15.221 "systemctl is-active ctp-http ctp-md deepcta-tp-executor deepcta-watchtower order-api"

# 查看交易日志
ssh root@49.233.15.221 "tail -f /root/CTA/logs/tp_executor.log"

# ── 因子分析 ──

# Run factor analysis notebook
cd scripts/Python && jupyter nbconvert --to notebook --execute 01_factor_analysis.ipynb

# Run ML modeling notebook
cd scripts/Python && jupyter nbconvert --to notebook --execute 02_ml_modeling.ipynb

# Generate lecture PPT
cd scripts/Python && python generate_ppt.py

# Update macro/futures/index data — 自动跟随期货清洗 (/macro)
conda activate ztrader && python /d/ZTrader/macro_research/full_data_pull.py

# 独立运行期货数据清洗 (到 data/cleaned/futures/)
conda activate base && python scripts/Python/clean_futures.py
```

## Macro Data

宏数据存储在 D:\ZTrader\macro_research\data\ 下，输入 `/macro` 即可增量更新。

| 类别 | 位置 | 品种数 |
|------|------|--------|
| 期货 | `D:\ZTrader\macro_research\data\futures\` | 82个品种 |
| 指数 | `D:\ZTrader\macro_research\data\indices\` | 7个指数 |
| 宏观 | `D:\ZTrader\macro_research\data\macro\` | 6个指标 |
