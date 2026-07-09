# Career Materials — Superstore Business Analytics Project

This file holds everything career-related for this project: resume copy, LinkedIn description, elevator pitch, STAR explanation, and interview preparation. Kept separate from `README.md` so the README stays pure technical documentation.

---

## Resume

**Project Title:** Superstore Business Analytics — SQL, Python & Tableau Pipeline

**Technologies Used:** MySQL, Python (Pandas, NumPy, Matplotlib), Tableau

**Resume Bullet Points (ATS-friendly):**

- Designed and implemented a MySQL data pipeline for a 10,000-row retail transactions dataset, including staging tables, 5-point data quality validation, and a cleaned analytical schema with 8 engineered features (profit margin, discount bands, shipping duration).
- Solved 30 business questions using advanced SQL (CTEs, window functions, correlated subqueries, joins, views) to quantify profit drivers, identifying multiple sub-categories generating negative profit despite high sales volume.
- Built 3 reusable SQL views as a semantic layer for a Tableau executive dashboard, reducing downstream query complexity and enabling self-service reporting for category and regional performance.

*(Bullets will be extended once the Python and Tableau phases are complete — each new phase gets its own bullet rather than inflating these.)*

---

## LinkedIn Description

Built an end-to-end business analytics project analyzing 10,000+ retail transactions using MySQL, Python, and Tableau. Designed a full ETL pipeline — from raw CSV staging through data quality validation, feature engineering, and 30 business questions solved with advanced SQL (CTEs, window functions, joins, views). Key finding: several product sub-categories were generating negative profit despite strong sales volume, directly traceable to discount policy. Project simulates the kind of profitability diagnostic a retail analytics or consulting team would run for leadership. Repository and full write-up: [GitHub link].

---

## Elevator Pitch (30–60 seconds)

"I built an end-to-end analytics project on a 10,000-row retail dataset to answer a question every retail business asks: where are we actually making money, and where are we losing it? I started in SQL — designing a proper staging-to-analysis pipeline, validating data quality instead of assuming it, and engineering features like profit margin and discount bands. Then I used SQL techniques ranging from window functions to CTEs to answer 30 real business questions. The standout finding was that certain sub-categories were consistently unprofitable despite decent sales volume — driven almost entirely by aggressive discounting. I packaged the reusable queries into SQL views so they could plug directly into a Tableau dashboard for non-technical stakeholders. It's the kind of diagnostic work an analytics or consulting team would actually deliver to leadership."

---

## STAR Explanation

**Situation:** A retail dataset with 4 years of order-level transactions existed but had no structured analysis layer — leadership couldn't answer basic profitability questions.

**Task:** Build a complete SQL-based analysis pipeline: clean the data properly, engineer useful features, and answer real business questions about where the company is profitable and where it isn't.

**Action:** Designed a staging → cleaning → analysis table architecture in MySQL. Ran explicit data quality checks rather than assuming the data was clean. Made and documented a scoping decision (excluding non-US rows that didn't fit the regional hierarchy). Wrote 30 business questions spanning aggregate functions, CASE logic, subqueries, CTEs, window functions, and joins. Wrapped the most reusable logic into SQL views.

**Result:** Identified specific loss-making sub-categories and quantified the relationship between discount level and profit — insights directly usable by a merchandising or pricing team. The output views are now feeding a Tableau dashboard layer.

---

## Top Business Insights

- Several sub-categories are unprofitable overall despite meaningful sales volume.
- Higher discount bands correlate with lower (often negative) total profit.
- Profit contribution is concentrated in a small number of categories (Pareto-style pattern).
- Regional average profit per order varies meaningfully from the company-wide baseline.

## Project Challenges

- **Ambiguous geography:** the dataset mixed US and Canadian rows under a US-only regional hierarchy. Resolved by explicitly scoping to US data and documenting the decision, rather than silently dropping rows.
- **Separating staging from analysis:** initially tempting to clean data directly in one table; instead built a proper raw → cleaned pipeline to mirror real-world ETL practice and keep the process re-runnable.
- **Choosing the right SQL technique per question:** deciding when a correlated subquery was necessary versus when a simpler GROUP BY/HAVING would do, to avoid over-engineering queries.

## Business Impact

The output (specifically the SQL views) is designed to plug directly into a BI tool, meaning the insights aren't just static findings — they're a live semantic layer a company could hand to a merchandising, pricing, or regional sales team with no further SQL work required.

---

## SQL Concepts Used

DDL & staging design · Data quality validation queries · CASE expressions · Aggregate functions (SUM, AVG, COUNT, COUNT DISTINCT) · GROUP BY / HAVING · Scalar, correlated, and derived-table subqueries · CTEs (single and multi-stage) · Window functions (RANK, ROW_NUMBER, LAG, running SUM, NTILE) · INNER/LEFT JOIN · Views · Indexing strategy

---

## Possible Interview Questions & Sample Answers

**Q: Why did you exclude the Canadian rows instead of keeping all the data?**
A: The `region` field only had valid values for US rows (Central/East/South/West) — Canadian rows had no equivalent regional dimension. Keeping them in would have silently corrupted every regional rollup in the analysis. I documented the decision and the verification query rather than just deleting rows quietly, which is what I'd do on a real project so the choice is auditable.

**Q: What's the difference between WHERE and HAVING, and where did you use each?**
A: WHERE filters individual rows before aggregation; HAVING filters groups after aggregation. I used HAVING in the "which sub-categories are unprofitable overall" question, because I needed to sum profit per sub-category first and then filter on that summed value — you can't do that with WHERE.

**Q: Why use a CTE instead of a subquery?**
A: CTEs make multi-step logic readable — especially when a query needs to build an intermediate aggregation and then rank or filter on top of it, like the "best category per region" question, which needed both a GROUP BY step and a ROW_NUMBER() step. Nesting that as a raw subquery would be harder to read and debug.

**Q: What's a correlated subquery, and can you give an example from your project?**
A: A correlated subquery references a column from the outer query, so it re-evaluates per row instead of computing once. I used one to find products priced above their own category's average unit price — the inner query's `WHERE category = outer.category` makes it correlated.

**Q: Why did you create views instead of just running queries directly in Tableau?**
A: Views give Tableau (or any BI tool) a stable, pre-aggregated table to connect to, so the dashboard doesn't need to replicate business logic or run expensive raw queries against the full fact table every time someone opens it. It also means the logic lives in one place — the database — rather than being duplicated across dashboard calculated fields.

**Q: How did you validate that your data was actually clean instead of just assuming it?**
A: I ran explicit checks before touching the data — counting NULLs per column, checking for duplicate primary keys and duplicate full rows, and verifying that ship dates were never earlier than order dates. All of those came back clean, which I documented rather than assumed.
