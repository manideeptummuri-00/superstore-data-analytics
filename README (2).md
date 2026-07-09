# Superstore Business Analytics — SQL Analysis Layer

End-to-end SQL analysis of a 10,000-row retail transactions dataset, covering database design, data cleaning, feature engineering, and 30 business questions answered with production-grade SQL (MySQL).

> **Project status:** SQL phase complete. Python (EDA) and Tableau (executive dashboard) phases in progress — see [Future Improvements](#future-improvements).

## Table of Contents
- [Project Overview](#project-overview)
- [Business Problem](#business-problem)
- [Objectives](#objectives)
- [Dataset Information](#dataset-information)
- [Database Schema](#database-schema)
- [Project Architecture](#project-architecture)
- [Repository Structure](#repository-structure)
- [Technologies Used](#technologies-used)
- [Data Cleaning Process](#data-cleaning-process)
- [Feature Engineering](#feature-engineering)
- [SQL Concepts Demonstrated](#sql-concepts-demonstrated)
- [Business Questions Solved](#business-questions-solved)
- [Project Workflow](#project-workflow)
- [SQL Views Created](#sql-views-created)
- [Key Business Insights](#key-business-insights)
- [Future Improvements](#future-improvements)
- [Author](#author)
- [License](#license)

## Project Overview

This project analyzes the Sample Superstore dataset — a retail transactions dataset spanning orders, customers, products, and regions — using MySQL to answer real business questions a retail analytics or consulting team would ask: where is the company profitable, where is it losing money, and what should management do about it.

## Business Problem

A national retail chain has four years of order-level transaction data but no structured analysis layer. Leadership cannot currently answer basic questions such as which product sub-categories are unprofitable, which regions underperform, or whether discounting is helping or hurting margins. This project builds that analysis layer from raw CSV to query-ready SQL views.

## Objectives

- Design a clean, indexed, analysis-ready database from a raw CSV export
- Quantify sales and profit performance across category, region, segment, and time
- Identify specific loss-making products/sub-categories and the discount behavior driving losses
- Produce reusable SQL views to serve as a semantic layer for downstream BI tools

## Dataset Information

- **Source:** Sample Superstore dataset (retail order-level transactions)
- **Raw size:** 10,194 rows × 21 columns
- **Grain:** one row = one product line item within one order
- **Scope:** Cleaned dataset is scoped to **United States only (9,994 rows)** — see [Data Cleaning Process](#data-cleaning-process) for why
- **Coverage:** 5,111 orders, 804 customers, 1,862 products, across 4 US regions and 3 product categories

## Database Schema

**`raw_superstore`** — staging table, 1:1 with the CSV (all fields as delivered).

**`superstore`** — cleaned, analysis-ready table:

| Column | Type | Notes |
|---|---|---|
| row_id | INT (PK) | Surrogate key |
| order_id | VARCHAR(20) | Groups line items into an order |
| order_date, ship_date | DATE | Converted from text during cleaning |
| ship_mode | VARCHAR(20) | Standard/First/Second Class, Same Day |
| customer_id, customer_name | VARCHAR | Buyer identity |
| segment | VARCHAR(20) | Consumer / Corporate / Home Office |
| city, state_province, postal_code, region | VARCHAR | Geography |
| product_id, category, sub_category, product_name | VARCHAR | Product identity |
| sales, quantity, discount, profit | DECIMAL/INT | Core transaction metrics |
| profit_margin | DECIMAL | *Engineered:* profit / sales |
| unit_price | DECIMAL | *Engineered:* sales / quantity |
| shipping_days | INT | *Engineered:* ship_date − order_date |
| order_year, order_month, order_year_month | INT/VARCHAR | *Engineered:* date parts |
| discount_band | VARCHAR | *Engineered:* CASE-based bucket |
| is_loss | TINYINT | *Engineered:* 1 if profit < 0 |

**`customer_summary`** — derived summary table (lifetime sales/profit/orders per customer), built to demonstrate JOIN patterns against the fact table.

## Project Architecture

```
CSV Dataset (samplesuperstore.csv)
        │
        ▼
Raw Staging Table (raw_superstore)
        │
        ▼
Data Quality Checks (nulls, duplicates, date logic, scope)
        │
        ▼
Data Cleaning + Type Conversion
        │
        ▼
Feature Engineering (profit margin, unit price, shipping days, discount band)
        │
        ▼
Cleaned Analysis Table (superstore)
        │
        ▼
Business Analysis (30 questions: aggregates, CASE, subqueries, CTEs, window functions, joins)
        │
        ▼
SQL Views (semantic layer)
        │
        ▼
Business Intelligence Layer (Tableau — in progress)
```

## Repository Structure

```
superstore-project/
│
├── data/
│   └── samplesuperstore.csv          # Raw source dataset
│
├── sql/
│   ├── 01_database_setup.sql         # Database + staging table + CSV import
│   ├── 02_data_cleaning.sql          # Quality checks, cleaning, feature engineering, indexes
│   ├── 03_business_questions.sql     # 30 business questions across 8 SQL technique categories
│   └── README.md                     # SQL-specific setup instructions
│
├── images/                           # Reserved for Tableau dashboard screenshots (Phase 4)
│
├── README.md                         # You are here
├── Project_Report.md                 # Consulting-style write-up for a non-technical audience
├── Career_Materials.md               # Resume, LinkedIn, interview prep (kept separate from documentation)
└── LICENSE
```

## Technologies Used

- **MySQL 8.0+** — database design, querying, views
- **SQL techniques:** DDL, staging/ETL pattern, CASE, subqueries, CTEs, window functions, joins, views, indexing

## Data Cleaning Process

1. **Verified data quality before cleaning** — zero NULLs across all columns, zero duplicate Row IDs, zero fully-duplicated rows, zero logical date violations (ship date never precedes order date).
2. **Scoping decision:** the raw dataset contains 200 Canadian rows, but the `region` field (Central/East/South/West) is a US-only geographic hierarchy. Mixing Canadian rows in would corrupt every regional rollup, so the cleaned table is scoped to United States only (9,994 rows). This is a documented decision, not a silent drop — the verification query is in `02_data_cleaning.sql`.
3. **Type correction:** order_date/ship_date converted from text to DATE; postal_code kept as VARCHAR (it's an identifier, not a number).
4. **Indexing:** added on order_date, customer_id, category+sub_category, region, and order_id to support the query patterns used across the 30 business questions.

## Feature Engineering

| Feature | Formula | Business Purpose |
|---|---|---|
| profit_margin | profit / sales | Normalizes profitability across order sizes |
| unit_price | sales / quantity | Enables pricing analysis independent of order volume |
| shipping_days | ship_date − order_date | Measures fulfillment speed |
| discount_band | CASE bucket on discount | Simplifies discount-vs-profit analysis |
| is_loss | 1 if profit < 0 | Quick flag for loss-making transaction analysis |
| order_year / order_month / order_year_month | Date parts | Enables trend and seasonality analysis |

## SQL Concepts Demonstrated

SELECT/WHERE/ORDER BY · GROUP BY/HAVING · Aggregate functions · CASE expressions · Scalar, correlated, and derived-table subqueries · CTEs (including multi-stage) · Window functions (RANK, ROW_NUMBER, LAG, running totals, NTILE) · INNER/LEFT JOIN · Views · Indexing

## Business Questions Solved

The full set of 30 questions lives in [`sql/03_business_questions.sql`](sql/03_business_questions.sql), organized into 8 technique categories. Representative examples:

1. Which sub-categories are actually losing money overall? *(GROUP BY + HAVING)*
2. Does discounting hurt profit — how does profit differ across discount bands? *(CASE)*
3. Which customers have above-average total sales? *(Subquery)*
4. What is month-over-month sales growth? *(CTE + LAG)*
5. For each region, what's the best-performing category by profit? *(CTE + ROW_NUMBER)*
6. Rank the top 5 products by sales within each category. *(Window function: RANK)*
7. What is the running cumulative total of sales by month? *(Window function: running SUM)*
8. What percentage of total company profit does each category contribute? *(CTE + scalar subquery)*
9. Show each order's sales alongside that customer's lifetime value. *(INNER JOIN)*
10. Compare each region's average order profit to the company-wide average. *(Derived-table join)*

## Project Workflow

1. Load raw CSV into a staging table with `LOAD DATA INFILE` (`01_database_setup.sql`)
2. Run data quality verification queries and document findings (`02_data_cleaning.sql`)
3. Build a cleaned, typed, feature-engineered analysis table with indexes
4. Answer 30 business questions spanning every major SQL technique (`03_business_questions.sql`)
5. Wrap the most reusable queries into views for downstream BI consumption

## SQL Views Created

- **`vw_monthly_performance`** — orders, sales, profit, and margin % rolled up by month. Feeds the trend charts in the Tableau dashboard.
- **`vw_category_profitability`** — sales, profit, and margin % by category and sub-category. Feeds category/sub-category comparison charts.
- **`vw_top_customers`** — lifetime sales, profit, and order count per customer. Feeds a top-customers table/filter in the dashboard.

## Key Business Insights

- Several product sub-categories generate **negative profit overall**, despite meaningful sales volume — a direct candidate list for pricing or discontinuation review (see Q7 in `03_business_questions.sql`).
- Higher discount bands are associated with materially lower (often negative) total profit — evidence that discounting is eroding margin rather than just driving volume.
- Profitability is concentrated: a small number of categories account for a disproportionate share of total company profit (Pareto pattern).
- Regional performance varies meaningfully around the company-wide average profit per order, pointing to specific regions worth deeper investigation.

*(Exact figures and rankings are generated by running the queries in `sql/03_business_questions.sql` against the live database — intentionally not hardcoded here to avoid the README drifting out of sync with the data.)*

## Future Improvements

- [ ] Python EDA layer (Pandas/NumPy/Matplotlib) — in progress
- [ ] Tableau executive dashboard with KPI cards, trend charts, and filters — in progress
- [ ] Automated data quality test suite
- [ ] Migration of the ETL steps into a lightweight Python/SQL pipeline script

## Author

Manideep TUmmuri
B.Tech Mining Engineering, IIT (ISM) Dhanbad

