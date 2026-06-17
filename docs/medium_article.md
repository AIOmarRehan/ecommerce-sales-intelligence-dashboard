# From Raw CSV to Executive Decisions: Building a Complete Retail Analytics Pipeline

A practical walkthrough of turning 10,000 retail transactions into actionable business strategy using Python, PostgreSQL, machine learning, and interactive dashboards.

---

## The Problem

Every data team eventually faces the same challenge: a CSV file lands on your desk with a vague request to "find insights." No schema, no documentation, no context. Just rows and columns and an expectation that you will deliver something useful.

This is the story of one such file. The Superstore Sales dataset contains 9,994 rows of retail transaction data spanning four years, three product categories, and 49 U.S. states. It has become something of a rite of passage in the analytics community, but most tutorials stop at a few charts and a summary table. I wanted to go further.

The goal was to build a production-grade analytics pipeline from start to finish: profiling, cleaning, warehousing, SQL analytics, machine learning, interactive dashboards, and a final executive report with prioritized recommendations. Not just analysis, but the kind of work you could present to a VP of Sales on Monday morning.

Here is how it unfolded.

---

## Phase 1: Understanding What You Have

Before writing a single line of cleaning code, I needed to understand the dataset at a structural level. How many rows? What does each column represent? Are dates formatted consistently? Are there missing values hiding in columns that look complete?

I wrote a profiling script using Pandas and NumPy that systematically examined every aspect of the raw data. The results told a clear story:

The dataset contains 9,994 rows across 21 columns. Every column has a value. No missing data, no null entries, no blank cells. On the surface, it looks pristine. But profiling goes deeper than null counts.

Dates were stored as strings in `D/M/YYYY` format, which is ambiguous when the day is 12 or below. A date like `12/6/2016` could be June 12 or December 6 depending on which side of the Atlantic you are on. When I tested both parsing formats, `D/M/YYYY` successfully parsed all 9,994 rows while `M/D/YYYY` only parsed 4,042. The day-first interpretation was correct, but any tool that defaults to U.S. date conventions would silently misread hundreds of records.

Shipping durations ranged from 0 to 7 days with a mean of exactly 4.0, consistent with the four shipping modes in the data (Same Day, First Class, Second Class, Standard Class). No negative shipping days existed, which meant no ship-before-order anomalies to worry about.

Both Sales and Profit showed extreme right skewness (12.97 and 7.56 respectively), meaning most transactions are small while a handful of very large orders pull the mean far above the median. This matters for any statistical work downstream.

![Year-over-year revenue and profit comparison](visualizations/01_yoy_revenue_profit.png)

The dataset covers fiscal years 2014 through 2017, with revenue growing from $484K to $733K over that period.

---

## Phase 2: The Cleaning Pipeline

Clean data is not just data without nulls. It is data that has been validated, standardized, checked for logical consistency, and enriched with derived features. I built a 13-step pipeline that runs idempotently, meaning you can re-execute it safely at any time.

**Duplicate detection.** The dataset had zero exact duplicate rows, but 8 partial duplicates where the same Order ID and Product ID appeared more than once. These were resolved by keeping the first occurrence and logging each removal with row indices for auditability.

**Date standardization.** All date strings were converted to proper datetime objects using the confirmed `D/M/YYYY` format. A new `Shipping_Days` column was computed as the difference between Ship Date and Order Date.

**Invalid row removal.** The pipeline checks for impossible values: negative quantities, negative sales, quantities above 100, discounts above 100%, and ship dates before order dates. In this dataset, none were found, but the checks exist as guardrails for future data.

**Categorical validation.** Each categorical column was validated against its known set of valid values. Ship Mode has exactly 4 values, Segment has 3, Region has 4, Category has 3. Any unexpected value would be flagged. Sub-Category was cross-validated against Category to catch impossible combinations.

**Outlier detection with four methods.** This is where the pipeline gets interesting. Rather than relying on a single technique, I applied four complementary approaches:

- **IQR method** flagged 1,166 sales outliers and 1,879 profit outliers
- **Z-Score** (threshold of 3 standard deviations) caught 127 sales and 107 profit outliers
- **Modified Z-Score** using Median Absolute Deviation identified 1,946 and 1,925 outliers respectively
- **Isolation Forest** (a machine learning approach using 200 estimators at 2% contamination) detected 200 multi-dimensional anomalies

The outliers were logged but preserved. In retail, large orders and deep losses are real business events, not data errors. Removing them would hide the very patterns stakeholders need to understand.

**Feature engineering.** Eight new columns were derived: `Order_Year`, `Order_Month`, `Order_Quarter`, `Order_DayName`, `Shipping_Days`, `Profit_Margin`, `Is_Loss`, and `Discount_Tier`. These make downstream analysis significantly cleaner.

**Data type optimization.** Eight string columns were converted to Pandas `category` dtype, and integer columns were downcast, reducing memory usage from 3,420 KB to 2,514 KB, a 26% savings.

The cleaned output: 9,986 rows across 29 columns, saved to `clean_data.csv` with a 100% data quality scorecard.

---

## Phase 3: Building the Data Warehouse

CSV files are fine for analysis, but they do not scale. For serious querying, I needed a proper relational database.

I created a PostgreSQL 18 database called `working_database` and designed two complementary schemas:

**The flat table** (`sales_data`) stores all 29 columns in a single denormalized table with 15 B-tree indexes and 8 CHECK constraints. This is the workhorse for ad-hoc analytics queries where you want speed and simplicity.

**The star schema** normalizes the data into dimension and fact tables:

| Table | Rows | Role |
|---|---|---|
| dim_customer | 793 | One row per unique customer |
| dim_product | 1,862 | One row per unique product |
| dim_geography | 631 | One row per postal code |
| dim_date | 1,434 | Full calendar with year, quarter, month, day-of-week |
| dim_ship_mode | 4 | Shipping method reference |
| fact_sales | 9,986 | Transactions linked to all dimensions via foreign keys |

Four analytical views were also created: `v_monthly_sales`, `v_regional_performance`, `v_category_performance`, and `v_top_customers`.

The import pipeline runs 17 automated verification checks covering row counts, aggregate totals, cross-schema consistency, and referential integrity. All 17 passed.

---

## Phase 4: 37 SQL Queries That Answer Real Business Questions

With the data in PostgreSQL, I wrote `business_analysis.sql`, a collection of 37 queries organized into 9 sections. These are not toy queries. They use CTEs, window functions, percentile calculations, NTILE-based segmentation, and multi-table JOINs across the star schema.

### Executive KPIs

The headline numbers tell the story at a glance:

| Metric | Value |
|---|---|
| Total Revenue | $2,295,510 |
| Total Profit | $286,014 |
| Profit Margin | 12.5% |
| Total Orders | 5,009 |
| Unique Customers | 793 |
| Loss Transactions | 1,870 (18.7%) |

Revenue grew from $484K in 2014 to $733K in 2017, representing a compound annual growth rate of roughly 15%. But growth alone does not tell you whether the business is healthy.

![Monthly revenue trend with moving average overlay](visualizations/02_monthly_trend.png)

The monthly trend reveals consistent seasonality with peaks in September and November across all four years, and a notable dip in February.

Looking at the same data through moving averages and momentum indicators strips away the month-to-month noise and reveals the underlying trajectory:

![Moving averages and revenue momentum indicators](visualizations/advanced/02_moving_averages_momentum.png)

### Sales by Region and State

The West region leads in revenue at $725K (32% of total), followed by East at $678K. But the Central region, despite generating $501K in revenue, has only a 7.9% profit margin compared to 14.9% for the West. Something is wrong in the Central region.

![Sales and profit by region](visualizations/03_regional_sales.png)

Digging deeper into state-level data revealed the problem: Texas alone accounts for $25,729 in losses despite generating $170K in revenue. Ohio ($17K loss), Pennsylvania ($15.6K loss), and Illinois ($12.6K loss) compound the issue.

![Top 15 states by revenue](visualizations/04_top_states.png)

### Category and Sub-Category Performance

Office Supplies is the volume leader at 60.3% of transactions, but Technology generates the highest revenue per transaction. Furniture is the clear problem child: $741K in revenue with only $18.4K in profit, a 2.5% margin.

![Category and sub-category performance](visualizations/05_category_performance.png)

The sub-category breakdown makes it worse. Tables ($207K revenue, $17.7K loss) and Bookcases ($115K revenue, $3.5K loss) are actively destroying value. Meanwhile, Copiers (37.2% margin) and Paper (43.4% margin) are highly profitable but underrepresented in the product mix.

![Top 10 products by revenue](visualizations/06_top_products.png)

On the other end of the spectrum, the worst-performing products tell a cautionary tale. These are items that consistently generate losses, often because they are high-ticket products sold at extreme discounts:

![Worst 10 loss-making products](visualizations/07_worst_products.png)

### The Discount Problem

This was the single most important finding in the entire analysis. The discount policy has a clear profitability cliff:

| Discount Range | Transactions | Loss Rate | Net Profit |
|---|---|---|---|
| No Discount | 4,793 | 0% | +$320,661 |
| 1 to 20% | 3,801 | 14% | +$100,717 |
| 21 to 30% | 226 | 91.6% | -$10,357 |
| 31 to 50% | 310 | 91.6% | -$48,448 |
| 51 to 80% | 856 | 100% | -$76,559 |

Every transaction discounted above 50% results in a loss. One hundred percent of them. Across 856 transactions, the business gave away $76,559. Discounts above 20% collectively cost $135K in negative profit. This is not a pricing strategy. It is a profitability crisis.

![Discount impact on profitability](visualizations/09_discount_impact.png)

### Customer Insights

The Consumer segment drives 52% of revenue but has the lowest profit margin at 11.5%. Home Office customers, the smallest segment by count, deliver 14.0% margin. The business is over-indexed on its least profitable customer type.

![Customer segment performance](visualizations/08_customer_segments.png)

### Shipping and Logistics

Standard Class handles 60% of shipments with an average delivery time of 5 days. Same Day shipping accounts for only 5.4% of orders but commands premium pricing.

![Shipping performance by mode](visualizations/10_shipping_performance.png)

### Seasonality

The heatmap reveals a consistent pattern: September and November are peak months across all four years, while February is reliably weak. Q4 generates 38% of annual revenue, creating significant business risk if holiday sales underperform.

![Revenue seasonality heatmap by year and month](visualizations/11_seasonality_heatmap.png)

### Executive Summary Dashboard

All of these findings are consolidated into a single executive dashboard view:

![Executive KPI dashboard](visualizations/12_executive_dashboard.png)

---

## Phase 5: Advanced Analytics with Machine Learning

SQL answers known questions. Machine learning finds patterns you did not think to ask about.

### Time Series Decomposition

Using Statsmodels, I decomposed the monthly revenue series into trend, seasonal, and residual components. The trend line shows steady upward movement from $40K to $62K monthly. The seasonal component reveals a consistent $20K swing between peak and trough months.

![Time series decomposition showing trend, seasonal, and residual components](visualizations/advanced/01_time_series_decomposition.png)

### Revenue Forecasting

Holt-Winters exponential smoothing with additive trend and seasonality produced a 12-month forward forecast. The model captures both the upward trajectory and the September/November peaks.

![Holt-Winters revenue forecast with confidence band](visualizations/advanced/04_forecast_holt_winters.png)

### Growth Rate Analysis

Year-over-year revenue growth averaged 27.2% with a 95% confidence interval of 11.7% to 42.7%. The wide interval reflects the small sample size (only 4 years of data), but the trend is unambiguously positive.

![Year-over-year and month-over-month growth rates with confidence intervals](visualizations/advanced/03_growth_rates.png)

### Customer Segmentation with RFM and K-Means

RFM analysis (Recency, Frequency, Monetary) classified the 793 customers into five behavioral segments:

![RFM customer segments with pie chart and revenue breakdown](visualizations/advanced/05_rfm_segments.png)

The "Lost" segment contains 238 customers who have not purchased recently. "Champions" number 176 and drive disproportionate revenue. This segmentation gives the sales team a clear targeting framework.

For a more data-driven approach, K-Means clustering with the elbow method identified K=4 as optimal. The clusters align with the RFM segments but reveal additional nuance in the frequency-monetary relationship.

![K-Means clustering with elbow method and scatter plot](visualizations/advanced/06_kmeans_clustering.png)

### Customer Lifetime Value

Projecting a 3-year CLV based on purchase frequency and average order value, the median customer is worth $1,249 while the top 20 customers range from $8K to $25K in projected lifetime value.

![Customer lifetime value distribution and top customer scatter plot](visualizations/advanced/07_customer_lifetime_value.png)

### Cohort Retention

Tracking customers by the quarter of their first purchase reveals retention patterns over time. Most cohorts show initial drop-off followed by partial recovery, with the 2016 Q4 cohort achieving 78% retention at quarter 4, the strongest in the dataset.

![Cohort retention heatmap by quarter](visualizations/advanced/08_cohort_retention.png)

### Product Performance Matrix

Mapping every product on a Sales vs Profit scatter plot (with bubble size representing units sold) creates a BCG-style portfolio matrix. "Stars" (high sales, high profit) number 687 products. "Dogs" (low sales, low profit) also number 687, suggesting a bifurcated product portfolio where roughly half the catalog is underperforming.

![BCG-style product performance matrix](visualizations/advanced/09_product_matrix.png)

### ABC Inventory Classification

Applying the Pareto principle, 422 Class A products (22.7% of the catalog) generate 80% of revenue. 501 Class B products cover the next 15%. The remaining 971 Class C products (52.4% of the catalog) contribute only 5% of revenue.

![ABC analysis with product count and cumulative revenue curve](visualizations/advanced/10_abc_analysis.png)

### Price Elasticity

Plotting discount percentage against average sale value and average profit reveals a negative correlation. Higher discounts do generate larger individual sales but destroy profit margins. The slope is steep enough that the revenue gain does not offset the margin loss.

![Price elasticity analysis showing discount impact on sales and profit](visualizations/advanced/11_price_elasticity.png)

### Cross-Selling Opportunities

Analyzing which sub-categories appear together in the same orders reveals natural product pairings. The top co-occurrence pairs suggest bundling opportunities that could increase average order value without additional discounting.

![Top 15 cross-selling product pairs by co-occurrence](visualizations/advanced/12_cross_selling.png)

### Correlation Analysis

The correlation matrix across all numerical variables shows a strong negative correlation (-0.86) between discount and profit margin, confirming what the discount impact analysis already revealed. Sales and profit have a moderate positive correlation (0.48), while shipping days show no meaningful correlation with any financial metric.

![Correlation matrix heatmap across numerical variables](visualizations/advanced/13_correlation_matrix.png)

### Hypothesis Testing

Four Welch's t-tests were conducted at the 0.05 significance level:

| Test | Result |
|---|---|
| Discounted vs Non-discounted sales | Statistically significant |
| West vs East region profit | Not significant |
| Consumer vs Corporate profit | Not significant |
| High quantity vs Low quantity margin | Statistically significant |

![Hypothesis testing results table with significance indicators](visualizations/advanced/14_hypothesis_testing.png)

### Predictive Modeling

A Gradient Boosting Regressor trained on 8 features (quantity, discount, shipping days, year, month, day-of-year, week-of-year, day-of-week) achieved an R-squared near zero on held-out data. This is actually the correct result: individual transaction-level sales in retail are inherently noisy and not well-predicted by calendar and order features alone. The model's feature importance ranking confirms that quantity and discount are the strongest predictors, while temporal features contribute minimally.

![Predictive modeling with feature importance and actual vs predicted scatter](visualizations/advanced/15_predictive_modeling.png)

### Outlier Deep Dive

The Isolation Forest analysis (3% contamination) flagged 300 anomalies across the four financial dimensions. Comparing their distributions against normal transactions shows anomalies are characterized by extreme sales values, high discounts, and negative profits.

![Outlier analysis comparing anomaly distributions against normal transactions](visualizations/advanced/16_outlier_analysis.png)

---

## Phase 6: Interactive Dashboards

Static charts are useful for reports. Interactive dashboards are useful for exploration. I built three types:

### Plotly HTML Dashboards

Four standalone HTML files with embedded interactive charts that work in any browser without a server:

- `dashboard_sales.html` - Revenue trends, regional breakdown, category performance, discount impact
- `dashboard_customers.html` - RFM scatter with hover details, segment analysis, revenue distributions
- `dashboard_products.html` - Top products, sub-category profitability, treemap, margin histogram
- `dashboard_geotemporal.html` - State rankings, year-by-month heatmap, shipping analysis, day-of-week patterns

These use Plotly.js loaded from CDN with all data embedded as JSON. Every chart supports hover tooltips, zoom, pan, and PNG export.

### Streamlit Web Application

A live Streamlit dashboard with 5 tabs (Executive Overview, Sales Trends, Products, Customers, Geography) provides real-time filtering through a sidebar with Year, Category, Region, Segment, and Date Range controls. All charts update instantly when filters change. KPI cards use a dark theme with green/red conditional formatting for profit and loss indicators.

![Streamlit dashboard overview page](dashboards/1.png)
![Streamlit sales trends page](dashboards/2.png)
![Streamlit products page](dashboards/3.png)
![Streamlit customers page](dashboards/4.png)
![Streamlit geography page](dashboards/5.png)

### Standalone HTML Export

For situations where a server is not available (presentations, email attachments, offline review), an export script generates a single `dashboard.html` file containing all 14 charts with tab navigation. The file is 0.2 MB and opens in any modern browser.

---

## Phase 7: Executive Recommendations

The final deliverable is not a chart or a query. It is a set of prioritized, data-backed recommendations that a business leader can act on.

### Priority 1: Implement a Maximum Discount Policy (Impact: HIGH)

Cap discounts at 20% for all product categories. Transactions above this threshold have a combined 94% loss rate. Enforcing a 20% cap would recover an estimated $135K annually and immediately improve net profitability. This is the single highest-impact action available, and it requires no new products, no new customers, and no new infrastructure. It requires a policy change.

### Priority 2: Restructure the Furniture Category (Impact: HIGH)

Tables and Bookcases together lose $21.2K annually. If these sub-categories could achieve the overall category average margin of 12.5%, they would generate $39.9K in profit instead. That is a $61K swing from pricing and supplier renegotiation alone.

### Priority 3: State-Level Profitability Reviews (Impact: MEDIUM-HIGH)

Texas, Ohio, Pennsylvania, Illinois, and North Carolina collectively lose $78.4K despite generating $500K in revenue. These states need dedicated pricing strategy reviews to determine whether losses are driven by a small number of accounts or systemic pricing issues.

### Priority 4: Reduce Q4 Revenue Dependency (Impact: MEDIUM)

Launch targeted campaigns in historically weak months (January, February, July) to smooth the seasonal curve. February is the weakest month across all four years. Even modest improvement reduces business risk.

### Priority 5: Grow the Home Office Segment (Impact: MEDIUM)

Home Office customers deliver 14.0% margin (highest of any segment) but represent only 148 customers. Targeted acquisition campaigns for home-office professionals could grow this high-margin segment without competing in the price-sensitive Consumer space.

### Priority 6: Discontinue or Reprice the Top 10 Loss-Making Products (Impact: MEDIUM)

Ten products, all high-ticket items sold at extreme discounts (50% to 80%), account for the majority of recurring losses. GBC binding systems alone account for $22.2K in losses across 17 transactions.

### The Bottom Line

Implementing just the first two recommendations (discount cap and Furniture restructuring) would increase total profit from $286K to approximately $482K, a 68% improvement, without selling a single additional unit.

---

## What This Project Demonstrates

This is not a Kaggle competition where the goal is to maximize a metric. This is the kind of work that happens inside organizations every day, where the deliverable is not a model score but a recommendation that someone has to act on.

The pipeline covers:

- **Data profiling** with Pandas to understand structure, types, and quality
- **13-step data cleaning** with outlier detection (IQR, Z-Score, MAD, Isolation Forest), feature engineering, and type optimization
- **Data warehousing** in PostgreSQL with both flat and star schemas, constraints, indexes, and automated verification
- **37 analytical SQL queries** using CTEs, window functions, NTILE segmentation, cohort analysis, and Pareto calculations
- **Machine learning** with time series decomposition, K-Means clustering, RFM analysis, Gradient Boosting regression, and hypothesis testing
- **28 visualizations** across static charts (Matplotlib/Seaborn), interactive dashboards (Plotly), and a live web application (Streamlit)
- **Executive reporting** that translates quantitative findings into prioritized business actions

Every script in the project is idempotent and re-runnable. The database credentials are managed through environment variables. The code is documented and organized into a clear directory structure.

The tools used: Python, Pandas, NumPy, SciPy, Statsmodels, Scikit-learn, PostgreSQL, psycopg2, Matplotlib, Seaborn, Plotly, Streamlit, and Power BI.

The dataset is public. The approach is what matters.

---

*The complete source code, SQL queries, visualizations, and dashboard files are available in the project repository.*
