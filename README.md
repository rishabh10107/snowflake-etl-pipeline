# Snowflake ETL Pipeline: Supplier Invoices, Purchase Orders & Weather

An end-to-end ETL pipeline in **Snowflake** and **Python**. It brings a distributor's purchase orders, supplier invoices, supplier master data, ZIP code geography and NOAA weather into one warehouse, then uses the combined table to check invoice accuracy.

**Author:** Rishabh Soni · [LinkedIn](https://linkedin.com/in/rishabhsoni10)

Developed from a course project in the MS in Business Analytics program at UC San Diego's Rady School of Management (2024), then extended with corrected weather matching and an invoice-accuracy analysis.

## Why it matters

Procurement teams need to know whether suppliers bill what was quoted. Here the data for that sits in five separate sources: CSV exports, an XML feed, a PostgreSQL database, a Census geography file and a Snowflake Marketplace weather dataset. This project pulls it all into one Snowflake schema, reconciles every invoice against its purchase order, and adds local weather so you can look at external conditions alongside supplier performance.

## What the pipeline does

| Step | Source | Technique |
|---|---|---|
| 1 | 41 monthly purchase-order CSVs | Snowflake internal stage, `PUT` and `COPY INTO` with type conversion during the load |
| 2 | Purchase orders | `GROUP BY` aggregation to calculate each order's value |
| 3 | Supplier transactions (XML) | Parsed with Python `ElementTree`, batch insert |
| 4–5 | Orders and invoices | Two-key join, invoiced-vs-quoted view |
| 6 | Supplier master data (PostgreSQL) | `psycopg2` extract, auto-generated DDL, table-stage load |
| 7 | Census Gazetteer and Cybersyn NOAA data (Snowflake Marketplace) | Geospatial nearest-station match with `ST_DISTANCE` and `QUALIFY ROW_NUMBER()` |
| 8 | All of the above | Final analysis table: one row per invoice, with supplier, amounts and daily high temperature |
| 9 | Final table | Discrepancy rate by supplier and by temperature band, with charts |

## Tech stack

Snowflake (stages, COPY INTO, geospatial functions, Marketplace data) · Python (pandas, ElementTree, psycopg2, matplotlib) · PostgreSQL · SQL

## Run it yourself

1. **Data.** Put these files in `data/`:
   - `Monthly_PO_Data/*.csv` (monthly purchase-order exports)
   - `Supplier_Transactions_XML.xml`
   - `2021_Gaz_zcta_national.txt` ([Census Gazetteer files](https://www.census.gov/geographies/reference-files/time-series/geo/gazetteer-files.html))
   - Load the `supplier_case` table into a local PostgreSQL database.
2. **Weather data.** In Snowflake, get the free *Weather & Environment* listing from Cybersyn on the Marketplace. Name the database `WEATHER__ENVIRONMENT`.
3. **Credentials.** Copy `.env.example` to `.env` and fill in your Snowflake and PostgreSQL details. `.env` is git-ignored.
4. **Install and run.**
   ```bash
   git clone https://github.com/rishabh10107/snowflake-etl-pipeline.git
   cd snowflake-etl-pipeline
   pip install snowflake-connector-python psycopg2-binary pandas matplotlib python-dotenv
   jupyter notebook supplier_pipeline.ipynb
   ```
