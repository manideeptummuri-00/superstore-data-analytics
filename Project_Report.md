# Project Report: Superstore Business Analytics — Profitability Diagnostic

*Prepared as a consulting-style report for senior management review.*

## Executive Summary

This report presents findings from a SQL-based diagnostic of the company's retail transaction data (9,994 US orders, 2023–2026). The objective was to answer a question leadership had not yet had structured data to answer: **where is the business profitable, and where is it not?** The analysis surfaces specific loss-making sub-categories, quantifies the relationship between discounting and profit erosion, and identifies where profit is concentrated across the product portfolio. All findings are reproducible directly from the SQL scripts and views delivered alongside this report.

## Business Problem

The company has accumulated several years of granular order-level data but no structured analysis layer sitting on top of it. As a result, category, pricing, and regional decisions have been made without a clear, queryable view of profitability — only of top-line sales. This creates risk: a category or region can look successful on revenue alone while quietly destroying margin.

## Objectives

1. Build a clean, reliable, query-ready database from the raw transaction export.
2. Quantify profitability at the category, sub-category, region, segment, and customer level.
3. Determine whether discounting is a profitable growth lever or a margin risk.
4. Deliver a reusable analysis layer (SQL views) that a BI tool can sit on top of without further engineering.

## Dataset

Source: Sample Superstore order-level transactions. 10,194 raw rows across 21 fields, covering orders, customers, products, geography, and financials (sales, quantity, discount, profit). After scoping to United States data (see Methodology), the analysis dataset contains 9,994 rows across 5,111 orders, 804 customers, and 1,862 products.

## Methodology

The analysis followed a standard staging → cleaning → analysis pipeline in MySQL:

1. Raw data loaded into a staging table with no transformation, to preserve an unmodified source of truth.
2. Data quality verified explicitly (not assumed) before any cleaning occurred.
3. A cleaned, typed, feature-engineered table built on top of the staging table.
4. Thirty business questions answered directly against the cleaned table, using techniques ranging from basic aggregation to window functions.
5. The most reusable analyses wrapped into SQL views to serve as a stable interface for downstream reporting tools.

## Data Cleaning

Verification queries confirmed: zero missing values across all fields, zero duplicate primary keys, zero fully duplicated transaction rows, and zero logical date inconsistencies (no order shipped before it was placed). One material data quality decision was required: the dataset included 200 transactions from Canada, but the regional dimension (Central/East/South/West) only applies to US geography. Including Canadian rows without a valid region would have distorted every regional comparison, so the analysis was scoped to United States transactions only — a decision made explicitly and documented in the SQL scripts, not applied silently.

## Feature Engineering

Six features were engineered to support the analysis: profit margin (profit as a percentage of sales), unit price (sales normalized by quantity), shipping duration (days between order and ship date), a discount band classification, a binary loss flag, and date-part fields (year/month) to support trend analysis. These features turn raw transactional fields into the inputs needed for margin, pricing, and logistics analysis.

## SQL Analysis

Thirty business questions were answered using SQL techniques spanning filtering and sorting, grouped aggregation with HAVING-based filtering, CASE-based business bucketing, scalar and correlated subqueries, common table expressions (including multi-stage CTEs), window functions (ranking, running totals, quartile bucketing), and joins between the transaction table and derived summary tables. Three views were created — monthly performance, category/sub-category profitability, and top customers — to serve as a semantic layer for a business intelligence dashboard.

## Business Insights

- **Profit is not evenly distributed across the product portfolio.** Several sub-categories generate negative profit in aggregate, despite non-trivial sales volume — meaning revenue growth in those lines is currently working against the company's bottom line, not for it.
- **Discounting shows a clear negative relationship with profit.** Orders in higher discount bands show materially lower — frequently negative — average profit, suggesting current discount practices are eroding margin faster than they are driving profitable volume.
- **Profit contribution follows a Pareto-like pattern**, with a small number of categories responsible for a disproportionate share of total company profit — meaning protecting and growing those categories matters more than treating all categories equally.
- **Regional profitability per order varies meaningfully around the company average**, indicating that regional strategy (pricing, assortment, or fulfillment) may need to be locally tuned rather than applied uniformly.

*(Note: precise figures and rankings should be pulled live from `sql/03_business_questions.sql` / the SQL views rather than restated as fixed numbers here, so this report never drifts out of sync with the underlying data as the pipeline evolves.)*

## Recommendations

1. **Review pricing and discount policy for identified loss-making sub-categories** — either raise price floors, cap maximum discount, or reassess whether to continue carrying them.
2. **Introduce a discount ceiling tied to product-level margin data**, rather than allowing discount decisions to be made independently of profitability.
3. **Prioritize investment and marketing spend toward the highest-profit-contributing categories**, given the concentrated nature of company profit.
4. **Investigate underperforming regions individually** rather than applying a single national strategy, given the spread in regional profit-per-order.

## Business Value

This analysis converts a static transaction log into an actionable profitability diagnostic. Because the core logic is delivered as SQL views rather than a one-off report, the same analysis can be re-run automatically as new data arrives, and can plug directly into a BI dashboard for ongoing self-service monitoring — rather than requiring a fresh manual analysis each time leadership has a question.

## Conclusion

The data does not currently support a "grow sales everywhere" strategy — it supports a more targeted one: protect and expand the categories and regions already driving profit, and fix or exit the segments where growth is happening at the expense of margin. The SQL pipeline and views built in this project give the business a repeatable way to monitor that going forward, rather than a single point-in-time snapshot.
