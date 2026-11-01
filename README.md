# Snowflake ETL Pipeline: Supplier Invoices, Purchase Orders & Weather

An end-to-end ETL pipeline built in **Snowflake** and **Python** that brings five separate data sources into one warehouse, then checks whether suppliers billed what they quoted.

**Author:** Rishabh Soni · [LinkedIn](https://linkedin.com/in/rishabhsoni10)

*Developed from a course project in the MS in Business Analytics program at UC San Diego's Rady School of Management (2024), then extended with corrected weather matching and an invoice-accuracy analysis.*

---

## Why it matters

Procurement teams need to know whether suppliers bill what was quoted. The data to answer that usually lives in different systems and formats. This project pulls it all into a single Snowflake schema, reconciles every invoice against its purchase order, and adds local weather for each supplier so invoice accuracy can be viewed alongside external conditions.

## Highlights

- **5 data sources** combined into one warehouse: CSV, XML, PostgreSQL, a Census file and Snowflake Marketplace data
- **2,400+ supplier transactions** parsed from XML and matched to purchase orders
- **Geospatial matching** of each supplier ZIP code to its nearest NOAA weather station using `ST_DISTANCE`
- **Data-quality fixes:** restores ZIP codes that lost their leading zero, and filters weather to true daily high temperatures
- **Credentials kept out of the code** with environment variables

## Pipeline

```mermaid
flowchart LR
    A[Monthly purchase-order CSVs] --> S[(Snowflake)]
    B[Supplier transactions XML] --> S
    C[(PostgreSQL supplier data)] --> S
    D[Census ZIP code centroids] --> S
    E[NOAA weather via<br/>Snowflake Marketplace] --> S
    S --> F[Invoice vs. purchase-order<br/>reconciliation]
    S --> G[Nearest weather station<br/>per supplier ZIP code]
    F --> H[Final analysis table]
    G --> H
    H --> I[Invoice accuracy analysis]
```

| Step | What happens | How |
|---|---|---|
| 1 | Load the monthly purchase-order files | Snowflake stage, `PUT` and `COPY INTO` with date conversion during the load |
| 2 | Calculate each order's total value | SQL aggregation of received quantity × unit price |
| 3 | Load supplier transactions from XML | Python `ElementTree`, batch insert |
| 4–5 | Match invoices to purchase orders and compare amounts | Join on supplier and purchase order, invoiced-vs-quoted view |
| 6 | Bring in supplier details from PostgreSQL | `psycopg2` extract, auto-generated table schema, Snowflake load |
| 7 | Find each supplier's nearest weather station and daily high temperature | Census ZIP centroids, `ST_DISTANCE`, `QUALIFY ROW_NUMBER()` |
| 8 | Build the final table, one row per invoice | Joins on supplier, ZIP code and transaction date |
| 9 | Analyse invoice accuracy by supplier and by temperature | pandas, matplotlib |

## Tech stack

**Snowflake** (stages, COPY INTO, geospatial functions, Marketplace data) · **Python** (pandas, ElementTree, psycopg2, matplotlib) · **PostgreSQL** · **SQL**

## Files

- `supplier_pipeline.ipynb`: the full pipeline and analysis
- `.env.example`: template for the Snowflake and PostgreSQL credentials the notebook reads
- `.gitignore`: keeps credentials and generated files out of the repo
