# CSI 300 Fundamental Factor Strategy

A quantitative research notebook that analyses four classic equity factors on the CSI 300 universe using monthly Chinese A-share data from 2016 to 2025.

## Overview

The project builds a point-in-time panel of ~300 stocks each month (CSI 300 constituents, proxied by ETF 510300.SH holdings) and evaluates four factors with decile-sort portfolio analysis, Rank IC / ICIR statistics, and long-short return series.

| Factor | Definition | Signal direction |
|---|---|---|
| **Size** | `log(market_cap)` where `market_cap = adjusted_close × total_shares` | Small-cap outperforms (Banz 1981) — long P1, short P10 |
| **Book-to-Market (B/M)** | `equity_parent / market_cap` | High B/M outperforms — long P10, short P1 |
| **Return on Assets (ROA)** | `net_profit_parent / total_assets` (annualised, industry-neutralised) | High ROA outperforms — long P10, short P1 |
| **Abnormal Turnover** | `volume_t / mean(volume_{t-12:t-1})` | High turnover predicts underperformance — long P1, short P10 |

## Repository Structure

```
.
├── factors_analysis.ipynb        # Main analysis notebook
├── pyproject.toml                # Project metadata and dependencies
├── uv.lock                       # Locked dependency versions
└── data/                         # Input data (not committed) — see data/README_en.md
    ├── README_en.md              # Full schema reference for all data files
    ├── EOD_PRICES.parquet        # Daily price/volume data (aggregated to monthly_data.parquet for the notebook)
    ├── monthly_data.parquet      # Monthly OHLCV used directly by the notebook
    ├── BALANCE.parquet / balance.parquet
    ├── INCOME.parquet  / income.parquet
    ├── CASHFLOW.parquet          # Cash-flow statement (not used by current notebook)
    ├── CAPITAL.parquet / capital.parquet
    ├── DIVIDEND.parquet          # Dividend data (not used by current notebook)
    └── ref_data/                 # Reference / classification data — see subdirectory READMEs
        ├── README_ETF_en.md      # ETF holdings schema
        ├── README_Industry_en.md # Industry classification schema
        ├── ETF_hold_510300.SH.parquet   # CSI 300 constituent holdings (semi-annual)
        ├── ETF_hold_510500.SH.parquet   # CSI 500 constituent holdings (semi-annual)
        ├── ETF_hold_512100.SH.parquet   # CSI 1000 constituent holdings (semi-annual)
        ├── Stock_Industry_Year.parquet        # Annual stock-industry mapping (simplified)
        └── AShareSWNIndustriesClass.parquet   # Full SWICS industry change history
```

Generated figures are saved to `figures/` after running the notebook.

## Methodology

### Universe construction
Semi-annual ETF 510300.SH holdings are forward-filled to a monthly frequency. A month is only included if the most recent holding disclosure is within 185 days, giving a clean monthly CSI 300 proxy (~300 stocks/month).

### Point-in-time merging
All financial statement data (balance sheet, income statement) are merged using `announce_date` rather than `report_period` to prevent look-ahead bias. For each `(stock, month_end)` the latest announcement on or before `month_end` is used. Only `ORIGINAL` and `RESTATED` statement types are included; `_VOID` variants (pre-correction versions) are excluded. See [`data/README_en.md`](data/README_en.md) for full details on statement types.

### Factor evaluation
- **Decile sort** — stocks are ranked into 10 bins cross-sectionally each month.
- **Portfolio returns** — equal-weight (EW) and value-weight (VW) annualised returns, volatility, and average market cap per decile.
- **Rank IC** — monthly cross-sectional Spearman correlation between factor and next-month return, reported as mean IC, ICIR (annualised), and win rate.
- **Long-short (L/S)** — monthly equal-weight L/S spread, annualised return, and win rate.

### ROA industry neutralisation
ROA is demeaned within SW Level-1 industry each month to remove sector composition bias (e.g. banks have structurally low absolute ROA).

## Setup

### Prerequisites
- Python ≥ 3.13
- [uv](https://github.com/astral-sh/uv) (recommended) **or** pip

### Install dependencies

```bash
# with uv (reproduces exact locked versions)
uv sync

# or with pip
pip install -e .
```

### Data
Place the required Parquet files under `data/` as shown in the structure above. The notebook expects that directory relative to its own location.

Full column-level schemas for every data file are documented in the dedicated READMEs committed to `main`:

| File | Description |
|---|---|
| [`data/README_en.md`](data/README_en.md) | Schemas for EOD prices, income statement, balance sheet, cash-flow statement, share capital, and dividend data (time range: 2013–2025) |
| [`data/ref_data/README_ETF_en.md`](data/ref_data/README_ETF_en.md) | ETF holdings schema for CSI 300 / 500 / 1000 constituent files |
| [`data/ref_data/README_Industry_en.md`](data/ref_data/README_Industry_en.md) | SWICS Level-1 industry classification — full history and annual snapshot |

## Running the Notebook

```bash
jupyter notebook factors_analysis.ipynb
# or
jupyter lab factors_analysis.ipynb
```

Run all cells in order. Figures are written to `figures/` automatically.

## Dependencies

| Package | Purpose |
|---|---|
| `pandas` | Panel data manipulation and time-series operations |
| `numpy` | Numerical computation |
| `matplotlib` | Decile bar charts and cumulative return plots |
| `scipy` | Spearman rank correlation for IC calculation |
| `fastparquet` | Reading Parquet data files |
| `jinja2` | Styled DataFrame rendering in the notebook |
