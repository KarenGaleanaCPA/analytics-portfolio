# DAX Catalog — Global Superstore

> Paste these measures into your model and adjust table/column names as needed. Keep a single source of truth here.

---

## 0) Date Table (required for time intelligence)

```DAX
DimDate =
ADDCOLUMNS (
    CALENDAR ( DATE(2010,1,1), DATE(2025,12,31) ),
    "Year", YEAR([Date]),
    "Month", FORMAT([Date], "MMM"),
    "YearMonth", FORMAT([Date], "YYYY-MM")
)
```

After creating it: **Table tools → Mark as date table → Date column**.

---

## 1) Core Measures

```DAX
-- Row / base measures
Orders := COUNTROWS(FactOrders)

Sales := SUM(FactOrders[Sales])

Profit := SUM(FactOrders[Profit])

Profit Margin % := DIVIDE([Profit], [Sales])

Quantity := SUM(FactOrders[Quantity])

Avg Shipping Cost := AVERAGE(FactOrders[Shipping Cost])
```

---

## 2) Shipping Timeliness & Late Logic

```DAX
-- Flags (use calculated columns only if you need row-level logic in slicers)
OnTime Flag := IF ( FactOrders[DaysLate] < 5, 1, 0 )

Late Flag (>=5d) := IF ( FactOrders[DaysLate] >= 5, 1, 0 )

-- Measures
OnTime Orders := SUM(FactOrders[OnTime Flag])

Late Orders (>=5d) := SUM(FactOrders[Late Flag (>=5d)])

OTD % := DIVIDE([OnTime Orders], [Orders])

Late % (>=5d) := DIVIDE([Late Orders (>=5d)], [Orders])

Avg Ship Lead Time (days) := AVERAGE(FactOrders[Ship Lead Time (days)])
```

> If you don’t have `DaysLate`, derive it in Power Query or as a calculated column using your delivery/target dates.

---

## 3) Returns

```DAX
Returns := CALCULATE ( [Orders], FactOrders[Returned] = TRUE() )

Return Rate % := DIVIDE ( [Returns], [Orders] )
```

---

## 4) Targets & Variance (example pattern)

Create a small table `Targets` with columns:
- `KPI` (e.g., "OTD %", "Late % (>=5d)", "Profit Margin %")
- `TargetValue` (numeric, e.g., 0.95, 0.02, 0.12)

```DAX
Target (Selected) :=
VAR sel = SELECTEDVALUE(Targets[KPI])
RETURN
    SWITCH (
        TRUE(),
        sel = "OTD %", CALCULATE( MAX(Targets[TargetValue]), Targets[KPI] = "OTD %" ),
        sel = "Late % (>=5d)", CALCULATE( MAX(Targets[TargetValue]), Targets[KPI] = "Late % (>=5d)" ),
        sel = "Profit Margin %", CALCULATE( MAX(Targets[TargetValue]), Targets[KPI] = "Profit Margin %" ),
        BLANK()
    )

Variance to Target := [OTD %] - [Target (Selected)]
```

Use separate cards per KPI or a KPI slicer bound to `Targets[KPI]`.

---

## 5) YoY / Time Intelligence

```DAX
Sales LY := CALCULATE ( [Sales], SAMEPERIODLASTYEAR ( DimDate[Date] ) )

Sales YoY Δ := [Sales] - [Sales LY]

Sales YoY % := DIVIDE ( [Sales YoY Δ], [Sales LY] )

OTD % LY := CALCULATE ( [OTD %], SAMEPERIODLASTYEAR ( DimDate[Date] ) )

OTD YoY Δ (pp) := [OTD %] - [OTD % LY]
```

> Ensure `DimDate` is marked as a date table and related to `FactOrders[Order Date]`.

---

## 6) Late Concentration & Top-N

```DAX
Late Orders by Segment :=
SUMMARIZE ( FactOrders, DimCustomer[Segment], "LateOrders", [Late Orders (>=5d)] )

Top N Sub-Category by Late :=
VAR N = 5
RETURN
TOPN ( N, SUMMARIZE ( DimProduct, DimProduct[Sub-Category], "Late", [Late Orders (>=5d)] ), [Late], DESC )
```

Use as the basis for bar charts or tables to show concentration.

---

## 7) Dynamic Titles (optional)

```DAX
Title - Late Focus :=
VAR geo = COALESCE( SELECTEDVALUE(DimRegion[Region]), "All Regions" )
VAR period = MIN(DimDate[YearMonth]) & " – " & MAX(DimDate[YearMonth])
RETURN
"Late ≥5d — " & geo & " (" & period & ")"
```

Bind this measure to a card and use it as a page title.

---

## 8) Quality Checks

```DAX
Rows Mismatch (Orders vs Fact) :=
VAR factRows = [Orders]
VAR distinctOrderIDs = DISTINCTCOUNT(FactOrders[Order ID])
RETURN
IF ( factRows < distinctOrderIDs, 1, 0 )
```

Surface this as a small badge; it helps catch unexpected duplicates.

---

## 9) Bookmarks & States

- Create a **Reset Filters** bookmark.
- Create a **Focus on Late** bookmark where `Late Flag (>=5d) = 1` and helpful slicers are pre-set.
- Add buttons for quick navigation.

---

## 10) Formatting & Accessibility

- Keep a consistent color for *Late* across pages.
- Show **% and counts** together on exception tables.
- Limit decimals; use thousand separators.
- Provide tooltips with exact definitions (copy from `docs/DASHBOARD_STORY.md`).


---

## 4b) Dedicated Variance Measures (no slicer)

> Use these if you store targets in a `Targets` table with fixed rows.

```DAX
Target - OTD % := CALCULATE ( MAX(Targets[TargetValue]), Targets[KPI] = "OTD %" )
Target - Late % (>=5d) := CALCULATE ( MAX(Targets[TargetValue]), Targets[KPI] = "Late % (>=5d)" )
Target - Profit Margin % := CALCULATE ( MAX(Targets[TargetValue]), Targets[KPI] = "Profit Margin %" )

OTD Variance (pp) := [OTD %] - [Target - OTD %]
Late Variance (pp) := [Late % (>=5d)] - [Target - Late % (>=5d)]
Profit Margin Variance (pp) := [Profit Margin %] - [Target - Profit Margin %]
```
