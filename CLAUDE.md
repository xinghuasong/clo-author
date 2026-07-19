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

## Commands

```bash
# Run factor analysis notebook
cd scripts/Python && jupyter nbconvert --to notebook --execute 01_factor_analysis.ipynb

# Run ML modeling notebook
cd scripts/Python && jupyter nbconvert --to notebook --execute 02_ml_modeling.ipynb

# Generate lecture PPT
cd scripts/Python && python generate_ppt.py

# Activate environment
conda activate base

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
