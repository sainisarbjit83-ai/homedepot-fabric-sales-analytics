# Notebooks

PySpark notebooks used to transform raw data through the medallion layers.
Both notebooks connect to the **Sales** Lakehouse in Microsoft Fabric.

---

## Notebooks Overview

| Notebook | Layer | Description |
|---|---|---|
| `Transform data for Silver.ipynb` | Bronze → Silver | Reads raw CSVs, applies schema, adds audit columns, merges into Delta table |
| `Transform data for Gold.ipynb` | Silver → Gold | Builds star schema — date, customer, product dimensions + fact table |

---

## Transform data for Silver

**Purpose:** Clean and standardize raw bronze data into a production-ready Delta table.

**What it does step by step:**

| Step | Code | Why |
|---|---|---|
| 1 | Define StructType schema | Prevents silent type-inference errors on CSV load |
| 2 | Load all CSVs from `Files/bronze/*.csv` | Picks up all years in one read |
| 3 | Add FileName column | Tracks which source file each row came from (lineage) |
| 4 | Add IsFlagged column | Business rule — flags orders before fiscal year start |
| 5 | Add CreatedTS / ModifiedTS | Load timestamps for audit trail |
| 6 | Null-handle CustomerName | Replaces blank/null with "Unknown" so reports never show blanks |
| 7 | Create sales_silver Delta table | Defines schema upfront with createIfNotExists |
| 8 | MERGE upsert | Inserts new rows only — idempotent, safe to rerun |
| 9 | Validate row count | SELECT COUNT(*) confirms 2,278 rows loaded |

**Key decision:** MERGE over overwrite — makes the pipeline idempotent.
Rerunning with the same data produces the same result, no duplicates.

---

## Transform data for Gold

**Purpose:** Shape clean silver data into a star schema optimized for Power BI reporting.

**What it does step by step:**

| Step | Table Built | Why |
|---|---|---|
| 1 | Load sales_silver | Starting point for all gold transforms |
| 2 | dimdate_gold | Extract Day/Month/Year/labels from distinct order dates |
| 3 | dimcustomer_gold | Deduplicate customers, split First/Last, assign surrogate CustomerID |
| 4 | dimproduct_gold | Split "ItemName, ItemInfo", assign surrogate ItemID |
| 5 | factsales_gold | Join silver to dimensions, replace strings with integer foreign keys |

**Key decisions:**
- **Left-anti join** for surrogate key generation — only assigns new IDs to 
  genuinely new records, existing IDs never change across pipeline runs.
- **monotonically_increasing_id() + MAX(existing ID) + 1** — new IDs always 
  pick up where existing ones left off, no collisions.
- **Star schema over flat table** — better VertiPaq compression, simpler DAX, 
  cleaner time intelligence in Power BI.
- **SCD Type 1** (overwrite on match) — simple implementation; 
  Type 2 (historical tracking) would be the next iteration.

---

## How to Run

1. Open Microsoft Fabric workspace `HomeDepot-Sales-Analytics`.
2. Attach the `Sales` Lakehouse to each notebook.
3. Run `Transform data for Silver` first (depends on bronze CSVs).
4. Run `Transform data for Gold` second (depends on sales_silver table).

> In production, both notebooks are orchestrated via the 
> **HomeDepot-Sales-Pipeline** Data Pipeline on a scheduled trigger.
