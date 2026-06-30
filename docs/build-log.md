# Build Log — Home Depot Fabric Sales Analytics


## Phase 0 — Setup
- Workspace: `HomeDepot-Sales-Analytics`
- Capacity: Fabric trial
- Items created: Lakehouse (`Sales`), Data Pipeline

## Phase 1 — Bronze Layer
- Uploaded `2019.csv`, `2020.csv`, `2021.csv` to `Files/bronze` in the Sales Lakehouse.
- Data: synthetic home-improvement retail sales (orders, line items, customer info).
- **Decision:** kept raw files untouched in bronze for lineage/replay — no transformation at this stage.

## Phase 2 — Silver Layer
- Notebook: `Transform data for Silver`
- Loaded CSVs with an explicit schema (`StructType`) rather than `inferSchema`.
- **Decision:** explicit schema avoids silent type-inference errors and is faster on large files.
- Added audit columns: `FileName`, `IsFlagged`, `CreatedTS`, `ModifiedTS`.
- Null-handled `CustomerName` → "Unknown".
- Wrote to `sales_silver` Delta table using `MERGE` (upsert) instead of overwrite.
- **Decision:** MERGE makes reruns idempotent — safe to re-trigger the pipeline without creating duplicates.
- - Verified row count post-merge: 2,278 rows (matches sum of source CSVs: 654 + 762 + 862).

_(<img width="3570" height="1757" alt="image" src="https://github.com/user-attachments/assets/235580ad-c537-42d1-80d4-0f2b9e862b3d" />
`)_

## Phase 3 — Gold Layer
- Notebook: `Transform data for Gold`
- Built star schema: `dimdate_gold`, `dimcustomer_gold`, `dimproduct_gold`, `factsales_gold`.
- Surrogate keys generated via `monotonically_increasing_id()`, using left-anti join to only assign IDs to new records.
- **Decision:** star schema (not flat/snowflake) for VertiPaq compression and simpler DAX relationships.
- **Note:** this is SCD Type 1 (overwrite) — Type 2 (historical tracking) would need `EffectiveDate`/`IsCurrent` columns.

_(screenshot: `screenshots/gold-tables.png`)_

## Phase 4 — Semantic Model
- Created semantic model `Sales_Gold` directly from the gold Delta tables (Direct Lake mode).
- Relationships: fact-to-dimension, single-direction.
- Key DAX measures: Total Sales, YoY Sales Growth %, Total Customers, Rolling 3-Month Average.
- **Decision:** Direct Lake chosen over Import to avoid duplicate data and a separate refresh schedule.

_(screenshot: `screenshots/semantic-model-relationships.png`)_

## Phase 5 — Report
- Pages: Executive Summary, Product Breakdown, Customer Drill-through.
- Used Performance Analyzer to check visual load times.

_(screenshot: `screenshots/report-exec-summary.png`)_

## Phase 6 — Orchestration
- Data Pipeline wraps Silver + Gold notebook runs, scheduled trigger.
- **Decision:** separate notebooks per layer for easier debugging/reuse, orchestrated together via the pipeline.

## What I'd improve next
- Add SCD Type 2 on the customer dimension.
- Add a data-quality-check notebook step (row counts, null rates, duplicate keys) with alerting.
- Promote through Dev/Test/Prod workspaces using Deployment Pipelines.
