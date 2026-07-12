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
| `silver-schema-cell.png<img width="3830" height="1757" alt="image" src="https://github.com/user-attachments/assets/89316715-0f63-4b5a-88fc-bd3877cd6fd6" />
` | Explicit StructType schema definition — prevents silent type-inference errors on CSV load |
| `silver-audit-columns-cell.png`<img width="3575" height="1500" alt="image" src="https://github.com/user-attachments/assets/c32bfebd-230f-42ab-b5ac-209528e38f77" />

 | Audit columns added: FileName (lineage), IsFlagged (business rule), CreatedTS/ModifiedTS (load timestamps) |
| `silver-merge-cell.png`<img width="3760" height="1582" alt="image" src="https://github.com/user-attachments/assets/454c092d-5ff0-49fc-9ef0-ec00ae153c87" />

 | MERGE upsert pattern — only new rows inserted, existing rows preserved, pipeline reruns are safe |
| `silver-row-count.png` <img width="2942" height="640" alt="image" src="https://github.com/user-attachments/assets/9108c706-d97c-4694-abb5-36d8a4aa8f5a" />
| Post-merge validation showing 2,278 rows — confirms 654 + 762 + 862 rows loaded correctly |
| `silver-table-explorer.png` <img width="2550" height="1275" alt="image" src="https://github.com/user-attachments/assets/fb271bfb-7b0b-4bcf-a4a5-a5eb26a9c5d6" />
| Lakehouse Explorer showing sales_silver Delta table with all 13 columns |

---

## Gold Layer

| File | Description |
|---|---|
| `gold-dimdate-preview.png`<img width="3690" height="1492" alt="image" src="https://github.com/user-attachments/assets/89a5bc47-290f-4a18-b7d1-f0592df11a63" />

| Date dimension preview — distinct dates with Day, Month, Year, mmmyyyy, yyyymm columns extracted |<img width="3785" height="1757" alt="image" src="https://github.com/user-attachments/assets/086671ec-1bb2-4db0-b1a2-3ecfcc9d130e" />

| `gold-dimcustomer-preview.png` <img width="2067" height="1552" alt="image" src="https://github.com/user-attachments/assets/94a9d13f-44a9-4094-aa8c-0c416e63ce2b" />
| Customer dimension preview — First/Last name split, surrogate CustomerID assigned via monotonically_increasing_id() |
| `gold-dimproduct-preview.png` <img width="2780" height="1295" alt="image" src="https://github.com/user-attachments/assets/b952b2f1-69f8-46c5-b203-c73bd57aebd9" />
<img width="2922" height="1520" alt="image" src="https://github.com/user-attachments/assets/9169f0d2-44fd-4504-aab5-7920d78534a2" />
| Product dimension preview — ItemName and ItemInfo split from combined "Name, Variant" string |
| `gold-factsales-preview.png` <img width="3560" height="1722" alt="image" src="https://github.com/user-attachments/assets/704c5bc5-e539-4bf4-93ad-c47f444bb089" />
| Fact table preview — CustomerID and ItemID populated as integers, confirming dimension joins resolved correctly |
| `gold-all-tables.png`<img width="592" height="1550" alt="image" src="https://github.com/user-attachments/assets/7574d141-a0c9-474b-a633-0fa23e2f6154" />

 | Lakehouse Explorer showing all five Delta tables: sales_silver + four gold dimension/fact tables |

---

## Semantic Model

| File | Description |
|---|---|
| `semantic-model-relationships.png` <img width="2462" height="1697" alt="image" src="https://github.com/user-attachments/assets/9c0588c7-8835-4a5c-bedd-9ca5aaad677a" />
| Star schema diagram — factsales_gold at center, three dimensions connected via one-to-many single-direction relationships |
| `measures-table.png`<img width="3837" height="1597" alt="image" src="https://github.com/user-attachments/assets/174c061b-c84f-4d60-b421-df32cce1c8b1" />
 | Measures_table showing all 10 DAX measures — foundation measures + time intelligence (YoY, YTD, Rolling Average) |

---

## Report Pages

| File | Description |
|---|---|
| `report-executive-summary.png` | Page 1 — KPI cards (Total Revenue, Orders, Customers, AOV) + YoY trend line |
| `report-product-analysis.png` | Page 2 — Top products by revenue, units sold by category, revenue mix % |
| `report-customer-analysis.png` | Page 3 — Revenue per customer, monthly trend, rolling 3-month average |


