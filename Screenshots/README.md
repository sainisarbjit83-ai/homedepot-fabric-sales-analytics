# Screenshots

Visual documentation of the Home Depot Fabric Sales Analytics project, 
captured at each phase of the build.

---

## Bronze Layer

| File | Description |
|---|---|
| <img width="1760" height="627" alt="image" src="https://github.com/user-attachments/assets/483dba71-9b65-4fa7-86b6-ba3a888b551d" />
`bronze-files.png` | Three raw CSV files (2019–2021) stored untransformed in `Files/bronze` — the source of truth layer |


---

## Silver Layer

| File | Description |
|---|---|
| `silver-schema-cell.png` | Explicit StructType schema definition — prevents silent type-inference errors on CSV load |
| `silver-audit-columns-cell.png` | Audit columns added: FileName (lineage), IsFlagged (business rule), CreatedTS/ModifiedTS (load timestamps) |
| `silver-merge-cell.png` | MERGE upsert pattern — only new rows inserted, existing rows preserved, pipeline reruns are safe |
| `silver-row-count.png` | Post-merge validation showing 2,278 rows — confirms 654 + 762 + 862 rows loaded correctly |
| `silver-table-explorer.png` | Lakehouse Explorer showing sales_silver Delta table with all 13 columns |

---

## Gold Layer

| File | Description |
|---|---|
| `gold-dimdate-preview.png` | Date dimension preview — distinct dates with Day, Month, Year, mmmyyyy, yyyymm columns extracted |
| `gold-dimcustomer-preview.png` | Customer dimension preview — First/Last name split, surrogate CustomerID assigned via monotonically_increasing_id() |
| `gold-dimproduct-preview.png` | Product dimension preview — ItemName and ItemInfo split from combined "Name, Variant" string |
| `gold-factsales-preview.png` | Fact table preview — CustomerID and ItemID populated as integers, confirming dimension joins resolved correctly |
| `gold-all-tables.png` | Lakehouse Explorer showing all five Delta tables: sales_silver + four gold dimension/fact tables |

---

## Semantic Model

| File | Description |
|---|---|
| `semantic-model-relationships.png` | Star schema diagram — factsales_gold at center, three dimensions connected via one-to-many single-direction relationships |
| `measures-table.png` | Measures_table showing all 10 DAX measures — foundation measures + time intelligence (YoY, YTD, Rolling Average) |

---

## Report Pages

| File | Description |
|---|---|
| `report-executive-summary.png` | Page 1 — KPI cards (Total Revenue, Orders, Customers, AOV) + YoY trend line |
| `report-product-analysis.png` | Page 2 — Top products by revenue, units sold by category, revenue mix % |
| `report-customer-analysis.png` | Page 3 — Revenue per customer, monthly trend, rolling 3-month average |


