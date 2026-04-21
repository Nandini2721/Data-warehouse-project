# Data Warehouse — Medallion Architecture

A layered ETL pipeline built on Bronze → Silver → Gold architecture, transforming raw CRM & ERP source data into analytics-ready star schema views.

**Tech:** SQL Server | ETL Pipeline | Stored Procedures | Star Schema

---

## Architecture Overview

```
[Bronze] Raw Ingestion → [Silver] Cleansed & Structured → [Gold] Analytics-Ready
```

---

## Bronze Layer — Raw Ingestion

- Created tables to store raw data exactly as received from CRM & ERP source systems, using basic data types with no constraints or transformations.
- Used a stored procedure with `BULK INSERT` to load data from CSV files directly into raw tables.
- Applied `TRUNCATE + reload` strategy — full refresh on every run ensures fresh source data.
- Preserved raw format (e.g., dates stored as INT) to maintain source data integrity as a true staging layer.
- Added logging, timestamps, and `TRY-CATCH` error handling for reliability and monitoring.

---

## Silver Layer — Cleansed & Structured

- Transformed Bronze data into clean, structured tables — including `crm_sales_details`, customer, product, and ERP tables — ready for Gold layer modeling.
- Applied transformations: deduplication via `ROW_NUMBER()`, whitespace removal with `TRIM()`, and value standardization via `CASE`.
- Handled NULLs, invalid data, and applied business rules — including recalculated sales figures and date validation.
- Added audit column `dwh_create_date` to all tables for data load tracking.

### Data Quality Checks

| Check | Description |
|---|---|
| Null Check | No NULL values in key columns |
| Duplicate Check | No duplicate records |
| Unwanted Spaces Check | All string fields trimmed |
| Standardization Check | Consistent value formats |
| Data Type / Format Check | Correct types across columns |
| Date Validation Check | Valid and in-range dates |
| Business Rule Check | Sales figures and logic verified |
| Data Consistency Check | Cross-table consistency |
| Range Check | Values within expected bounds |
| Referential Integrity Check | All foreign keys resolve |

---

## Gold Layer — Analytics-Ready Views

- Created dimension and fact **views** forming a final star schema — `dim_customers` and `dim_products` as dimensions, connected via `fact_sales`.
- Integrated data from multiple Silver tables into unified dimensions (e.g., customer data merged from CRM & ERP sources, with CRM as the primary source for gender).
- Filtered out historical product records — only current products included (`prd_end_dt IS NULL`).
- Surrogate keys generated using `ROW_NUMBER()` for each dimension.
- Provides clean, business-ready data optimized for analytics and reporting with no downstream transformations required.

### Gold Quality Checks

- Customer key uniqueness
- Product key uniqueness
- Fact-to-dimension referential integrity

---

## Summary

| Layers | Quality Checks | Schema |
|---|---|---|
| 3 (Bronze / Silver / Gold) | 10 total | Star Schema |

---

*Project inspired by [Data with Baraa](https://www.youtube.com/@DataWithBaraa)*
