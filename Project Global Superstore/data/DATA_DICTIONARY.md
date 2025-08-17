# Short Data Dictionary — Global Superstore (Kaggle)

> Source: [Global Superstore (Kaggle)](https://www.kaggle.com/datasets/shekpaul/global-superstore)  
> Scope: concise reference of the **raw** fields typically present in the dataset. Field names can vary slightly by version (hyphens vs. spaces, case).

---

## Orders (line-level)
| Field | Type | Description |
|---|---|---|
| Row ID | integer | Line identifier within the dataset (not a business key). |
| Order ID | string | Unique order identifier (joins to Returns). |
| Order Date | date | Customer order date. |
| Ship Date | date | Shipment date. |
| Ship Mode | string | Shipping service level (e.g., Second Class). |
| Customer ID | string | Unique customer identifier. |
| Customer Name | string | Customer full name. |
| Segment | string | Customer segment (Consumer, Corporate, Home Office). |
| Country | string | Destination country. |
| City | string | Destination city. |
| State | string | Destination state/province. |
| Postal Code | string | Postal/ZIP code (text to preserve leading zeros). |
| Region | string | Geographic region grouping (e.g., West, Central). |
| Market | string | Higher-level market (e.g., APAC, EU, US, LATAM). |
| Product ID | string | Unique product identifier (joins to Product details). |
| Category | string | Product category (Furniture, Office Supplies, Technology). |
| Sub-Category | string | Product sub-category (e.g., Chairs, Storage). |
| Product Name | string | Product description/name. |
| Sales | decimal | Net sales at line level. |
| Quantity | integer | Units sold at line level. |
| Discount | decimal | Discount ratio (0–1). |
| Profit | decimal | Gross profit at line level. |
| Shipping Cost | decimal | Shipping charge allocated to the line (if available). |
| Order Priority | string | Priority class (e.g., Critical, High, Medium, Low). |

**Primary keys:** `Order ID` is unique at order header; `(Order ID, Row ID)` or `(Order ID, Product ID, Row ID)` often unique at line level depending on the version.  

---

## Returns
| Field | Type | Description |
|---|---|---|
| Order ID | string | Identifier of the returned order (joins to Orders). |
| Returned | boolean/string | Flag or code indicating if the order was returned. |
| Market | string | Market of the returned order (optional depending on version). |

---

## People (Sales/Region owners)
| Field | Type | Description |
|---|---|---|
| Region | string | Region name (joins to Orders.Region). |
| Person | string | Account owner / regional manager. |

---

## Derived Fields (created in the notebook for analysis)
| Field | Type | Definition |
|---|---|---|
| Ship Lead Time (days) | integer | `Ship Date - Order Date` in days. |
| Year, Month | integer/string | Extracted from `Order Date` for time series. |
| Profit Margin % | decimal | `Profit / Sales` (safe-divide). |
| Returned (joined) | boolean | From Returns table onto Orders by `Order ID`. |

---

## Relationships (typical)
- **Orders ↔ Returns**: `Orders.Order ID = Returns.Order ID` (left join from Orders).
- **Orders ↔ People**: `Orders.Region = People.Region` (left join to add owner).
- **Orders** serves as the **fact** table; product/customer/region are **dimensions** when modeling in BI.

> Note: Some Kaggle exports include minor field-name or type differences. Normalize headers and types during ingestion to keep models stable.
