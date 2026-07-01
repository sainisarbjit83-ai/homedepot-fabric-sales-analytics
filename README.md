# 🏠 Home Depot Fabric Sales Analytics

End-to-end Microsoft Fabric + Power BI project demonstrating medallion architecture, 
dimensional modeling, and retail sales reporting on synthetic home-improvement sales data.

## 🏗️ Architecture

```
Raw CSVs → Bronze (Files) → Silver (Delta) → Gold (Star Schema) → Semantic Model → Power BI Report
(store)      (clean+audit)    (dim+fact)       (Direct Lake)         (report)
```

Each layer builds on the previous — raw data is never modified, transformations are 
traceable, and the pipeline is idempotent (safe to rerun without creating duplicates).

---

## ⚙️ Tech Stack

| Tool | Purpose |
|---|---|
| Microsoft Fabric Lakehouse | Unified storage (Files + Delta Tables) on OneLake |
| PySpark Notebooks | Bronze → Silver → Gold transformations |
| Delta Lake | ACID transactions, MERGE/upsert, time travel |
| Data Pipeline | Orchestration and scheduling |
| Power BI Semantic Model | Direct Lake mode — no data copy, no refresh schedule |
| DAX | Business measures including time intelligence |

---

## 🥉🥈🥇 Medallion Layers

| Layer | Table/Location | Description |
|---|---|---|
| Bronze | `Files/bronze/*.csv` | Raw CSVs stored untouched — source of truth for replay |
| Silver | `sales_silver` | Cleaned, schema-enforced, audited Delta table with MERGE upsert |
| Gold | `dimcustomer_gold` | Customer dimension — surrogate keys, First/Last name split |
| Gold | `dimdate_gold` | Calendar dimension — Day, Month, Year, formatted labels |
| Gold | `dimproduct_gold` | Product dimension — ItemName/ItemInfo split, surrogate keys |
| Gold | `factsales_gold` | Fact table — Quantity, UnitPrice, Tax with integer foreign keys |

---

## 🔑 Key Design Decisions

**Why explicit schema over inferSchema?**
Spark's inferSchema can silently misread dates as strings or prices as integers. 
Defining StructType upfront guarantees correct types on every load.

**Why MERGE instead of overwrite?**
MERGE is idempotent — rerunning the pipeline never creates duplicates. 
Only genuinely new rows are inserted; existing rows are preserved. 
Critical for production pipelines that run on a schedule.

**Why surrogate keys instead of natural keys?**
Natural keys (like CustomerName) are unstable — they can change or have typos. 
Integer surrogate keys are stable, compact, and join faster in Power BI's VertiPaq engine.

**Why star schema instead of a flat table?**
Star schema gives better VertiPaq compression, simpler DAX relationships, 
and cleaner time intelligence. A flat table repeats strings thousands of times; 
a fact table with integer keys stays lean and fast.

**Why Direct Lake mode?**
Direct Lake reads Delta Parquet files directly from OneLake — no separate data copy, 
no scheduled refresh, Import-like query speed. Best of Import and DirectQuery combined.

---

## 📐 DAX Measures

All measures live in a dedicated `Measures_table` (best practice — keeps data 
tables clean and measures easy to find).

### Foundation Measures
| Measure | DAX Pattern | Description |
|---|---|---|
| Total Revenue | SUMX | Quantity × UnitPrice, row by row |
| Total Tax | SUM | Sum of Tax column |
| Total Sales | Measure reference | Total Revenue + Total Tax |
| Total Orders | COUNTROWS | Count of all sales lines |
| Total Customers | DISTINCTCOUNT | Unique customer count |
| Avg Order Value | DIVIDE | Total Revenue ÷ Total Orders (zero-safe) |

### Time Intelligence Measures
| Measure | DAX Pattern | Description |
|---|---|---|
| Total Revenue YTD | TOTALYTD | Jan 1 to selected date, current year |
| Total Revenue LY | CALCULATE + SAMEPERIODLASTYEAR | Same period, prior year |
| YoY Growth % | DIVIDE | (This Year - Last Year) ÷ Last Year |
| Rolling 3-Month Avg | AVERAGEX + DATESINPERIOD | 3-month moving average |

---

## 📸 Screenshots

### Bronze — Raw File Storage
![Bronze Files](screenshots/bronze-files.png)
*Three yearly CSV files stored untransformed in `Files/bronze` — 
preserving original data for lineage and replay.*

### Silver — Clean & Audit Layer
![Silver Schema](screenshots/silver-schema-cell.png)
*Explicit StructType schema prevents Spark from misreading OrderDate as 
a string or Tax as an integer — no silent type errors.*

![Silver Audit Columns](screenshots/silver-audit-columns-cell.png)
*Audit columns added: FileName (lineage), IsFlagged (business rule), 
CreatedTS/ModifiedTS (load timestamps). Essential for production traceability.*

![Silver Merge](screenshots/silver-merge-cell.png)
*MERGE upsert pattern — existing rows left untouched, only new rows inserted. 
Makes pipeline reruns safe and idempotent.*

![Silver Row Count](screenshots/silver-row-count.png)
*Post-merge validation: 2,278 rows = 654 (2019) + 762 (2020) + 862 (2021). 
Confirms all three CSVs loaded correctly with no duplicates.*

### Gold — Star Schema
![Customer Dimension](screenshots/gold-dimcustomer-preview.png)
*Customer dimension: First/Last name split from full name, 
surrogate CustomerID assigned via monotonically_increasing_id() 
offset by current max ID — prevents collisions on incremental loads.*

![Product Dimension](screenshots/gold-dimproduct-preview.png)
*Product dimension: "Cordless Drill, 20V Max" split into 
ItemName + ItemInfo — enables category-level filtering in reports.*

![Fact Table](screenshots/gold-factsales-preview.png)
*Fact table: CustomerID and ItemID populated as integers (no nulls) — 
confirms left joins to dimension tables resolved correctly.*

![All Tables](screenshots/gold-all-tables.png)
*Full medallion architecture in Lakehouse Explorer: 
sales_silver + four gold Delta tables, all with Delta triangle icons.*

### Semantic Model
![Relationships](screenshots/semantic-model-relationships.png)
*Star schema in Power BI — factsales_gold at center, 
three dimensions connected via one-to-many single-direction relationships.*

![Measures Table](screenshots/measures-table.png)
*10 DAX measures in dedicated Measures_table — foundation + 
time intelligence measures including YoY Growth % and Rolling 3-Month Average.*

---

## 📁 Folder Structure

```
homedepot-fabric-sales-analytics/
│
├── data/                    # Synthetic sample CSVs (2019–2021)
│   ├── 2019.csv
│   ├── 2020.csv
│   └── 2021.csv
│
├── Notebook/                # PySpark notebooks
│   ├── Transform_data_for_Silver.ipynb
│   └── Transform_data_for_Gold.ipynb
│
├── screenshots/             # Key UI captures with descriptions in build log
│
├── docs/
│   └── build-log.md        # Step-by-step build decisions and notes
│
└── README.md
```

---

## 🚀 How to Reproduce This Project

1. Get a Microsoft Fabric trial at `app.fabric.microsoft.com`.
2. Create a workspace with Fabric capacity.
3. Create a Lakehouse named `Sales`.
4. Upload CSVs from `data/` folder into `Files/bronze/`.
5. Run `Transform_data_for_Silver` notebook.
6. Run `Transform_data_for_Gold` notebook.
7. Create a semantic model from the four gold tables.
8. Build relationships and add DAX measures from `Measures_table`.

---

## 🔮 What I'd Add Next

- **SCD Type 2** on `dimcustomer_gold` — track historical name/email changes 
  with EffectiveDate/ExpiryDate/IsCurrent columns.
- **Data quality notebook** — row count checks, null rate validation, 
  duplicate key detection with alerting after each merge.
- **Deployment Pipelines** — promote through Dev/Test/Prod workspaces.
- **Incremental refresh** — configure Power BI incremental refresh policy 
  on the semantic model for large-scale production use.
- **SalesOrderNumber in fact table** — enables precise distinct order count 
  rather than row count approximation.
