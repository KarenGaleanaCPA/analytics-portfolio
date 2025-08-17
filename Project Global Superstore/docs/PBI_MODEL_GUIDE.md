# Power BI Model Guide — Global Superstore

This guide explains how the `.pbix` connects to the CSV exports from the Python pipeline and how the model is organized for performance and clarity.

---

## 1) Files & Locations

- **Power BI file:** `reports/powerbi/Global Superstore.pbix`
- **CSV outputs (created by the notebook):** `data/processed/`
  - `fact_order_lines.csv`
  - `dim_customer.csv`
  - `dim_product.csv`
  - `dim_region.csv`
  - `dim_market.csv`

> Keep large/raw sources in `data/raw/` (excluded from git). Track `.pbix` with Git LFS.

---

## 2) Connect the Model to CSVs (once)

1. Open the `.pbix` in Power BI Desktop.
2. **Transform Data** → verify each table points at `data/processed/*.csv`.
3. If the path changed, update the **folder parameter** (recommended) or the individual file sources:
   - `Home → Transform data → Data source settings → Change Source`
4. **Close & Apply** then **Refresh**.

**Tip (parameterize the path):**
- Create a parameter `DataFolder` (type *Text*) with value like `data/processed/`.
- In each query, build the path: `Csv.Document(File.Contents(DataFolder & "fact_order_lines.csv"), ...)`
- This makes the repo portable (clone → refresh).

---

## 3) Recommended Model (Star Schema)

**Fact**
- `FactOrders` (from `fact_order_lines.csv`): one row per order line

**Dimensions**
- `DimCustomer` (`dim_customer.csv`)
- `DimProduct` (`dim_product.csv`)
- `DimRegion` (`dim_region.csv`)
- `DimMarket` (`dim_market.csv`)
- `DimDate` (created in Power Query or DAX; marks Date table)

**Relationships (single direction, many-to-one)**
- `FactOrders[CustomerID]  →  DimCustomer[CustomerID]`
- `FactOrders[ProductID]   →  DimProduct[ProductID]`
- `FactOrders[Region]      →  DimRegion[Region]`
- `FactOrders[Market]      →  DimMarket[Market]`
- `FactOrders[Order Date]  →  DimDate[Date]`  *(Date table = marked)*

> Avoid bi-directional filters unless truly required. It keeps measures predictable and fast.

---

## 4) Performance Practices

- Use **Import** mode (CSV) for speed.
- Keep columns **typed** (dates, integers, decimals). Avoid `Any`.
- Remove unused columns in Power Query.
- Prefer **measures** over calculated columns for aggregations.
- Keep visuals per page to a reasonable count (10–12 max).

---

## 5) Refresh & Reproducibility

- The notebook re-creates `data/processed/*.csv`. After you run it, open the `.pbix` and refresh.
- Pin library versions via `requirements.txt` and keep KPI definitions in `docs/DASHBOARD_STORY.md`.
- If publishing to Power BI Service, store files in OneDrive/GitHub-backed folder or use a **Gateway** for local paths.

---

## 6) Pages (suggested)

1. **Overview** — KPI cards, trendlines, target vs actual.
2. **Drivers** — Region / Market / Segment / Category / Sub-Category.
3. **Exceptions (Late ≥ 5 days)** — table + slicers + drill-through.
4. **Actions & Scenarios** — recommended levers and impact.

> Rename these pages to match your actual `.pbix`. Keep page names short and action-oriented.



### Targets table (recommended)
- Import `data/sample/targets.csv` as table **Targets** (KPI, TargetValue).
- Bind KPI cards to the target-aware measures from `docs/DAX_CATALOG.md` (variance in pp).
- You can replace the CSV with an *Enter data* table in production.
