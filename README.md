 # 🔨 Retail Sales Analytics

End-to-end retail sales analytics project built on Microsoft Fabric 
and Power BI — covering data ingestion, medallion architecture 
(Bronze → Silver → Gold), dimensional modeling with Delta Lake, 
and executive dashboarding using Direct Lake semantic model and DAX.

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
