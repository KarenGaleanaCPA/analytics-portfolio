# Dashboard Story & User Guide — Global Superstore

 **Overview**, **Profitability**, **Returns**, and **Operations (Shipping)**. It explains what each page is for, how to read it, and which decisions it supports.

---

## 1) Audience & Goal
- **Audience:** Commercial/Ops managers and analysts
- **Goal:** Track margin and service, spot problem markets/products, and choose actions (pricing, carrier, inventory) that move KPIs.

---

## 2) Pages & Key Visuals

### A) Overview
![Overview](/reports/images/overview.png)
**What it shows**
- KPI cards: **Total Sales**, **Profit Margin %**, **Return Rate %**, **On‑Time %**.
- **World map** with pie bubbles — Sales by **Country × Category**.
- **Country table** — Sales by Category with totals.

**How to read**
1. Scan cards: are **Profit Margin %** and **On‑Time %** above target?
2. Use slicers (**Year**, **Category**, **Segment**) to focus.
3. Use the map to find **large bubbles with low margin / high returns**, then open the **Profitability** or **Returns** page to drill down.

---

### B) Profitability
![Profitability](/reports/images/profitability.png)
**What it shows**
- **Sales vs Profit Margin % — by Category** (clustered bars + line).
- **Sales vs Profit Margin % — by Sub‑Category** (bars + margin line).
- **Scatter**: *Sales vs Profit Margin % — by Product* (colored by Category).
- **Product table**: Top products with total sales, profit, margin.

**How to read**
1. Start with **Category** bars to see which areas create/erode margin.
2. Move to **Sub‑Category** — spot outliers (e.g., **Tables** margin is negative in the sample).
3. Use the **scatter** to identify specific **products** with **high sales but weak margin** → candidates for price/discount changes or sourcing reviews.

**Decisions**
- Tighten discounting thresholds where elasticity is steep.
- Replace/renegotiate low‑margin products (e.g., high shipping or return costs).

---

### C) Returns
![Returns](/reports/images/returns.png)
**What it shows**
- **Matrix**: **Return Rate %** by *Category × Market*.
- **Top 15 Countries** by **# Returned Orders** (stacked by Category).
- **Treemap**: **# Returned Orders by Sub‑Category**.

**How to read**
1. Start at the **matrix** to see **which markets** drive returns.
2. Check **Top 15 countries** for concentration (US/MX often lead).
3. Use the **treemap** to isolate **problem sub‑categories** (e.g., Binders, Accessories).

**Decisions**
- Investigate packaging/quality for high‑return sub‑categories.
- Adjust return policies or supplier contracts in problem markets.

---

### D) Operations (Shipping)
![Operations](/reports/images/operations.png)
**What it shows**
- KPI cards: **On‑Time %** and **Late %** (definition below).
- **On‑Time % over time** (Quarterly line with target/average reference).
- **Late % vs On‑Time % by Ship Mode** (stacked bars).
- **# Late Orders by Ship Mode** (stacked by Category).
- **Top 10 Countries by # Late Orders**.

**How to read**
1. Check **On‑Time %** trend vs reference to see direction of travel.
2. Compare **Ship Modes** — where is Late % concentrated?
3. Use the **Top 10 Countries** to route improvement actions.

**Decisions**
- Switch carriers/SLAs for affected modes or countries.
- Rebalance inventory to cut **Ship Lead Time** on slow lanes.

---

## 3) KPI Definitions (used in this report)
- **On‑Time %** = On‑time orders / Total orders.
- **Late %** = Late orders / Total orders, where **Late = Ship Lead Time ≥ 5 days (SLA)**.
- **Return Rate %** = Returned orders / Total orders.
- **Profit Margin %** = Profit / Sales.
- **Ship Lead Time (days)** = `Ship Date − Order Date` (calendar days).

Targets (example): OTD ≥ 95%, Late ≤ 2%, Profit Margin ≥ 12%. Keep these in your `Targets` table and cards.

---

## 4) Interactions & Tips
- **Slicers:** Year, Category, Segment.
- **Drill‑through:** from Overview map or Drivers into country‑level pages.
- **Bookmarks:** `Reset Filters`, `Focus on Late` (pre‑filtered Late = 1).
- **Accessibility:** use a consistent color for *Late* across pages; show **% + counts** together on exception visuals.

---

## 5) Insight Templates (fill with your real findings)
- **Late concentrated in Standard Class:** Late % = X% vs Y% target; **US/France** contribute ~Z% of late orders. *Action:* prioritize carrier change for Standard Class in those lanes.
- **Negative margin in Tables:** despite $A sales, margin = −B%. *Action:* review pricing/sourcing; restrict discounts above D%.
- **High returns in Binders/Accessories:** Return Rate = W% in **LATAM/EMEA**. *Action:* QA packaging and adjust suppliers.

---

## 6) DAX & Model Notes
This report uses the measures from `docs/DAX_CATALOG.md` (OTD, Late, Return Rate, YoY, Targets). Ensure `DimDate` is marked as a Date table and relationships are single‑direction from dimensions to fact for predictable filters.
