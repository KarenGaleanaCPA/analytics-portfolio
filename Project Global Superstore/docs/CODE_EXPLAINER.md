
# Code Explainer — Global Superstore (End‑to‑End Analytics)

This document explains how the notebook **`Global_Superstore_End_To_End.ipynb`** works so reviewers can understand the end‑to‑end pipeline (data → clean → enrich → analyze → export for Power BI). It’s written to be repository‑ready for your `docs/` folder.

---

## 1) Objective & Outputs

**Objective.** Build a disciplined analytics dataset and insights for the Global Superstore sample, with:  
- Clean, validated inputs (schema contracts, types, guardrails)  
- Business‑friendly derived features (lead time, margins, etc.)  
- Exploratory analysis (trends, drivers, discount elasticity)  
- Cohort/retention view of customers  
- **Exports** of a star schema (fact + dimensions) to feed a Power BI dashboard

**Primary outputs (saved to disk):**
- `data/processed/fact_order_lines.csv` — line‑level fact table
- `data/processed/dim_customer.csv` — customer dimension
- `data/processed/dim_product.csv` — product dimension
- `data/processed/dim_region.csv` — region/people dimension
- `data/processed/dim_market.csv` — market dimension
- (Plus ad‑hoc charts in the notebook and optional `reports/` artifacts)

---

## 2) Environment & Dependencies

**Python**: 3.10+ recommended  
**Libraries used** (detected): IPython, __future__, dataclasses, duckdb, math, matplotlib, numpy, pandas, pathlib, plotly, re, typing, warnings

If you want a minimal `requirements.txt`, start with:

```
duckdb>=0.9.2
ipython>=8.0
matplotlib>=3.7
numpy>=1.23
pandas>=1.5
plotly>=5.15
xlrd>=2.0
```

> The workbook expected by the notebook is an Excel **.xls** file, so `xlrd` is included as the engine. If you later switch to `.xlsx`, use `openpyxl` instead.

---

## 3) Inputs & Data Contracts

**Source:** Public [Global Superstore](https://www.kaggle.com/datasets/shekpaul/global-superstore) dataset from Kaggle (used for educational/portfolio purposes). Please consult the dataset page for license/terms before redistribution.


**Expected input file**: paths.raw / "GlobalSuperstore.xls"
The notebook reads three sheets (case‑insensitive): **Orders**, **Returns**, **People**.

**Schema contracts** (assertions) ensure required columns exist in each sheet. Typical required fields include:
- Orders: `Order ID`, `Order Date`, `Ship Date`, `Ship Mode`, `Customer ID`, `Customer Name`, `Segment`, `City`, `State`, `Country`, `Market`, `Region`, `Product ID`, `Category`, `Sub-Category`, `Product Name`, `Sales`, `Quantity`, `Discount`, `Profit`, `Shipping Cost`, `Order Priority`
- Returns: `Order ID`, `Returned`, `Market`
- People: `Region`, `Person` (sales/owner)

If columns are differently cased or spaced, **`standardize_columns()`** normalizes them (e.g., trims whitespace and applies title‑case while preserving known names).

---

## 4) Pipeline Overview

1) **Read & Standardize**
   - Robust Excel helpers: **`read_sheet_any_case(path, name)`** safely locates the sheet regardless of case; **`standardize_columns(df)`** cleans/normalizes headers.

2) **Join & Guardrails**
   - Merge Orders with Returns and People (by `Order ID` / `Region`) to add return flags and sales owner.
   - Guardrails applied up‑front:
     - Negative `Quantity` → absolute value
     - Clip `Discount` to `[0,1]`
     - Safe `Profit Margin = Profit / Sales` with divide‑by‑zero protection
     - Derive **`Ship Lead Time (days)`** = `Ship Date` − `Order Date`
     - Timestamp features: `Year`, `Month`

3) **Missing‑Value Strategy**
   - Snapshot of null% per column to *justify* imputations.
   - Hierarchical fill utilities:
     - **`fill_by_hierarchy()`**: fill missing values based on higher‑level groups (e.g., Country → State → City).
     - **`impute_sd()`**: targeted numeric imputations with documented defaults only when needed.

4) **Outliers & Viz‑Friendly Copy**
   - **`winsorize_group(frame, col, by=...)`**: clip extreme values within groups (used to stabilize plots for `Sales`, `Profit`, `Shipping Cost`).
   - **`robust_group_outliers()`** (where applicable) to flag/remove pathological points for analysis purposes, while keeping an untouched copy for audits.

5) **Exploratory Analysis (EDA)**
   - Monthly **Sales & Profit** trends (matplotlib).
   - Top **Sub‑Categories** by profit (Plotly bar).
   - Discount elasticity by sub‑category (see below).

6) **Discount Elasticity & Break‑Even**
   - **`fit_slope_intercept(x, y)`** fits a line (margin vs. discount).
   - **`discount_elasticity_by_subcat(orders_clean, min_rows, min_unique)`** computes per‑sub‑category slope and R² to identify where discounts most erode margins.
   - **`elasticity_with_break_even()`** estimates the **break‑even discount** d* = −b / a (intercept / slope) when feasible.

7) **SQL Analysis with DuckDB**
   - In‑memory views (**`v_orders`**, **`v_returns`**, **`v_people`**) and composable SQL for:
     - Category‑level Sales/Profit/Profit‑Margin
     - “Top product per Sub‑Category” using window functions
     - (Room to add shipping SLAs, return rates by market, etc.)

8) **Customer Cohorts & Retention**
   - Build `cohort` (first order month) and `months_since_cohort`; pivot to a retention matrix (periods 0–12 shown in‑notebook).

9) **Star Schema Export (for Power BI)**
   - **Fact**: `fact_order_lines.csv` (one row per order line; includes dates, product keys, customer keys, metrics).
   - **Dims**: `dim_customer.csv`, `dim_product.csv`, `dim_region.csv`, `dim_market.csv` (unique keys; tidy attributes).
   - Saved to: Path("data/processed"); DATA_OUT.mkdir(parents=True, exist_ok=True) (created if missing).

---

## 5) Key Functions in this Notebook

- clean, discount_elasticity_by_subcat, elasticity_with_break_even, fill_by_hierarchy, fit_slope_intercept, impute_sd, read_sheet_any_case, robust_group_outliers, standardize_columns, winsorize_group

> These helpers keep the notebook *explainable* and *reusable* across similar retail datasets.

---

## 6) How to Run (Step‑by‑Step)

### Option A — Jupyter (recommended for review)
1. Create a virtual env and install dependencies:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```
2. Put the Excel file at **`data/raw/GlobalSuperstore.xls`** (or update the path in the notebook).
3. Open the notebook and **Run All**.
4. Check outputs in **`data/processed/`** and any figures printed inline.

### Option B — Convert to script (optional)
If you prefer scripts, export key cells to `src/` files (e.g., `load.py`, `clean.py`, `features.py`, `export.py`) and keep this doc as the architectural guide.

---

## 7) Design Choices & Rationale

- **Contracts first**: fail‑fast if columns are missing → easier reproducibility.
- **Guardrails**: prevent silently broken ratios (e.g., margin), keep quantities sane.
- **Winsorization for viz**: story clarity for humans; retain raw copy for audits.
- **Elasticity**: simple linear fit per sub‑category surfaces where discounting is most harmful; R² threshold avoids over‑interpreting noise.
- **DuckDB**: lets you express complex group/window logic in standard SQL without leaving Python memory.
- **Star schema**: clean separation of facts/dims makes the Power BI model simpler and queries faster.

**Limitations**
- Linear elasticity is an approximation; curved responses or confounders may exist.
- Cohort retention is at monthly granularity; consider weekly if the business needs it.
- The sample dataset is static; real deployments need data freshness checks and scheduling.

---

## 8) What “Good” Looks Like (Reviewers’ Checklist)

- ✅ Clear inputs, contracts, and deterministic outputs  
- ✅ Reusable helpers (imports, paths, cleaning functions)  
- ✅ Traceable metrics with exact definitions  
- ✅ Exports aligned to a BI model (fact + dims)  
- ✅ Honest notes on limitations and next steps

---

## 9) Next Steps (Ideas)

- Add **shipping SLA** analysis vs. `Ship Lead Time` distributions by `Ship Mode`.
- Enrich **returns analytics** (return rates × margin impact).
- Try **regularized** models for elasticity (e.g., ridge) and price bands.
- Automate nightly runs and publish CSVs to cloud storage for Power BI refresh.
