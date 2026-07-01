# 🏠 Home Depot Fabric Sales Analytics

End-to-end Microsoft Fabric + Power BI project built as a portfolio piece 
demonstrating medallion architecture, dimensional modeling, and retail 
sales reporting.

> **Note:** Data is synthetic/sample data styled around a home-improvement 
retail scenario. Not real Home Depot data.

---

## Architecture
Raw CSVs → Bronze (Files) → Silver (Delta) → Gold (Star Schema) → Semantic Model → Power BI Report
## Tech Stack
- Microsoft Fabric (Lakehouse, Notebooks, Data Pipeline)
- PySpark + Delta Lake
- DAX + Power BI Direct Lake
- OneLake

## Medallion Layers

| Layer | Table | Description |
|---|---|---|
| Bronze | Files/bronze/*.csv | Raw CSVs, untouched |
| Silver | sales_silver | Cleaned, audited, merged Delta table |
| Gold | dimcustomer_gold | Customer dimension with surrogate keys |
| Gold | dimdate_gold | Calendar dimension |
| Gold | dimproduct_gold | Product dimension with surrogate keys |
| Gold | factsales_gold | Fact table (Quantity, UnitPrice, Tax) |

## DAX Measures
- Total Revenue, Total Sales, Total Tax
- Total Orders, Total Customers, Avg Order Value
- Total Revenue YTD, Total Revenue LY
- YoY Growth %, Rolling 3-Month Avg Revenue

## Folder Structure
- `notebooks/` — PySpark notebooks for Silver and Gold transforms
- `data/` — Synthetic sample CSVs (2019–2021)
- `screenshots/` — Key UI captures
- `docs/build-log.md` — Step-by-step build decisions and notes
