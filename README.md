![Medium Article](https://medium.com/@ai.omar.rehan/from-raw-csv-to-executive-decisions-building-a-complete-retail-analytics-pipeline-0d8d0699b922)

---

# Superstore Sales Analytics

An end-to-end data analytics project built on the Superstore Sales dataset. Covers the full pipeline from raw data profiling through advanced machine learning to interactive dashboards and executive reporting.

## Project Overview

This project analyzes 9,986 retail transactions spanning 4 years (2014-2017), 793 customers, 1,862 products, and 49 U.S. states. The objective is to extract actionable business insights through a structured analytics pipeline:

1. **Data Profiling** - Understand structure, types, distributions, and quality
2. **Data Cleaning** - 13-step pipeline with outlier detection, imputation, and feature engineering
3. **Data Warehousing** - PostgreSQL dual-schema design (flat table + star schema)
4. **SQL Analytics** - 37 production-grade analytical queries across 9 business domains
5. **Advanced Analytics** - Time series decomposition, K-Means clustering, RFM segmentation, predictive modeling
6. **Visualization** - 28 static charts, 4 interactive Plotly dashboards, and a Streamlit app
7. **Executive Reporting** - Data-backed strategic recommendations with prioritized action items

## Tech Stack

| Layer | Tools |
|---|---|
| **Language** | Python 3.14 |
| **Data Processing** | Pandas, NumPy |
| **Statistical Analysis** | SciPy, Statsmodels |
| **Machine Learning** | Scikit-learn (IsolationForest, KMeans, GradientBoosting, StandardScaler) |
| **Database** | PostgreSQL 18.4 |
| **Database Driver** | psycopg2-binary, python-dotenv |
| **Visualization (Static)** | Matplotlib, Seaborn |
| **Visualization (Interactive)** | Plotly |
| **Dashboard Framework** | Streamlit |
| **BI Tool** | Power BI Desktop |
| **Notebook** | Jupyter (IPython) |
| **Credentials** | Environment variables via .env |

## Project Structure

```
Data Analytics Project/
|
|-- data/
|   |-- raw/                          # Original unmodified dataset
|   |   +-- Superstore sales dataset.csv
|   +-- processed/                    # Cleaned output from pipeline
|       +-- clean_data.csv
|
|-- scripts/
|   |-- analyze_data.py               # Phase 1: Dataset profiling and statistics
|   |-- data_cleaning.py              # Phase 2: 13-step cleaning pipeline
|   |-- db_import.py                  # Phase 3: PostgreSQL schema + data import
|   |-- business_analysis.sql         # Phase 4: 37 analytical SQL queries
|   |-- advanced_analytics.py         # Phase 5: ML, clustering, forecasting
|   |-- interactive_dashboard.py      # Phase 5: Plotly HTML dashboards
|   |-- showcase.py                   # Phase 4: Chart generation from SQL results
|   |-- export_html.py                # Standalone HTML dashboard exporter
|   |-- dashboard.py                  # Streamlit live dashboard
|   |-- pull_metrics.py               # PostgreSQL metric extraction for reports
|   +-- superstore_pipeline.ipynb     # Jupyter notebook combining all phases
|
|-- visualizations/
|   |-- 01-12 (PNG)                   # Standard analysis charts
|   +-- advanced/
|       |-- 01-16 (PNG)              # Advanced analytics charts
|       +-- dashboard_*.html          # Interactive Plotly dashboards
|
|-- Queries/
|   +-- Query 1-8/                    # Power BI SQL query exports with CSV results
|
|-- dashboards/                       # Power BI dashboard screenshots
|   +-- 1-5.png
|
|-- docs/
|   |-- Dataset Understanding.md      # Phase 2 dataset documentation
|   |-- sales_performance_review.*    # Executive business report (PDF/DOCX)
|
|-- logs/                             # Execution logs from all pipeline stages
|   |-- analyze_data_output.txt
|   |-- cleaning_log.txt
|   |-- cleaning_summary.txt
|   +-- db_import_log.txt
|
|-- dashboard.html                    # Standalone interactive HTML dashboard
|-- .env                              # Local credentials (not tracked)
|-- .env.example                      # Template for credentials
|-- .gitignore
+-- README.md
```

## Pipeline Execution

Run each phase sequentially from the project root:

```bash
# Phase 1: Profile the raw dataset
python scripts/analyze_data.py > logs/analyze_data_output.txt

# Phase 2: Clean, validate, and engineer features
python scripts/data_cleaning.py

# Phase 3: Create database and import data
# (requires PostgreSQL running locally)
python scripts/db_import.py

# Phase 4: Generate charts from SQL analysis
python scripts/showcase.py

# Phase 5: Run advanced analytics (ML, forecasting)
python scripts/advanced_analytics.py

# Phase 5: Generate interactive Plotly dashboards
python scripts/interactive_dashboard.py

# Export standalone HTML dashboard
python scripts/export_html.py

# Launch Streamlit live dashboard
python -m streamlit run scripts/dashboard.py
```

## Database Schema

The PostgreSQL database `working_database` contains two complementary schemas:

### Flat Table (sales_data)

29 columns covering all transactional and derived data in a single table. Optimized with 15 B-tree indexes and 8 CHECK constraints for data integrity.

### Star Schema

| Table | Rows | Description |
|---|---|---|
| dim_customer | 793 | One row per unique customer |
| dim_product | 1,862 | One row per unique product |
| dim_geography | 631 | One row per postal code |
| dim_date | 1,434 | Calendar dimension with metadata |
| dim_ship_mode | 4 | Shipping method reference |
| fact_sales | 9,986 | Transactional facts with FK joins |

### Analytical Views

- `v_monthly_sales` - Monthly aggregated revenue and profit metrics
- `v_regional_performance` - Region and state-level performance breakdown
- `v_category_performance` - Product category and sub-category analysis
- `v_top_customers` - Top 100 customers ranked by total revenue

## SQL Analytics Coverage

The `business_analysis.sql` file contains 37 queries organized into 9 sections:

| Section | Queries | Techniques |
|---|---|---|
| Executive KPIs | 2 | Aggregate functions, window functions, LAG |
| Sales Analysis | 7 | Time series, day-of-week patterns, revenue share |
| Product Analysis | 6 | Rankings, HAVING filters, discount correlation |
| Customer Analysis | 5 | RFM (NTILE), cohort retention, CLV estimation |
| Profitability | 4 | Percentiles, cross-tab pivots, discount impact |
| Shipping & Logistics | 3 | Performance distribution, regional comparison |
| Advanced Analytics | 6 | Pareto (80/20), moving averages, MoM growth, seasonality pivots |
| Star Schema | 3 | Multi-table JOINs across dimension and fact tables |
| Executive Dashboard | 1 | CROSS JOIN with CTEs for single-query snapshot |

## Advanced Analytics

The `advanced_analytics.py` script applies data science techniques beyond SQL:

| Analysis | Methods |
|---|---|
| **Trend Analysis** | Additive time series decomposition, Holt-Winters exponential smoothing forecast, 3/6/12-month moving averages, momentum indicators |
| **Customer Segmentation** | RFM quartile-based classification, K-Means clustering (elbow method, K=4), Customer Lifetime Value (3-year projection), cohort retention heatmaps |
| **Product Analysis** | BCG matrix (Stars/Cash Cows/Question Marks/Dogs), ABC inventory classification, price elasticity (discount vs. sales/profit), cross-selling pair detection |
| **Statistical Analysis** | Correlation matrix, 4 hypothesis tests (Welch's t-test), Gradient Boosting regression (TimeSeriesSplit CV), Isolation Forest anomaly detection |

Output: 16 publication-quality charts saved to `visualizations/advanced/`.

## Key Findings

| Metric | Value |
|---|---|
| Total Revenue | $2,295,510 |
| Total Profit | $286,014 |
| Overall Margin | 12.5% |
| Total Orders | 5,009 |
| Unique Customers | 793 |
| Loss Transactions | 1,870 (18.7%) |
| YoY Avg Growth | 27.2% |

### Critical Insights

- **Furniture category** generates $741K in revenue but only 2.5% margin. Tables and Bookcases sub-categories account for $21.2K in net losses.
- **Discounts above 20%** have a 94% combined loss rate, costing $135K in negative profit across 1,392 transactions.
- **Five states** (Texas, Ohio, Pennsylvania, Illinois, North Carolina) generate $500K in combined revenue but lose $78.4K.
- **Q4 dependency**: 38% of annual revenue and 39% of profit concentrated in a single quarter.
- **Home Office segment** delivers the highest margin (14.0%) despite being the smallest segment by customer count.

## Dashboards

### Streamlit Dashboard (Live)

Interactive web app with 5 tabs, sidebar filters, and real-time chart updates:

```bash
python -m streamlit run scripts/dashboard.py
```

Tabs: Executive Overview, Sales Trends, Products, Customers, Geography.

### Static HTML Dashboard (Offline)

Self-contained HTML file with 14 interactive Plotly charts, no server required:

```bash
python scripts/export_html.py
# Open dashboard.html in any browser
```

### Plotly Interactive Dashboards

Four standalone HTML dashboards in `visualizations/advanced/`:

- `dashboard_sales.html` - Revenue trends, regional breakdown, discount impact
- `dashboard_customers.html` - RFM scatter, segment analysis, distributions
- `dashboard_products.html` - Top products, sub-category profit, treemap
- `dashboard_geotemporal.html` - State rankings, heatmap, shipping analysis

## Setup Requirements

### Prerequisites

- Python 3.10+
- PostgreSQL 14+ (running as a local service)

### Installation

```bash
# Install Python dependencies
pip install pandas numpy scipy scikit-learn statsmodels \
    psycopg2-binary python-dotenv \
    matplotlib seaborn plotly \
    streamlit jupyter

# Configure database credentials
cp .env.example .env
# Edit .env with your PostgreSQL password
```

### Database Setup

```bash
# Create the database
psql -U postgres -c "CREATE DATABASE working_database;"

# Run the import pipeline
python scripts/data_cleaning.py
python scripts/db_import.py
```

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| DB_NAME | working_database | PostgreSQL database name |
| DB_USER | postgres | Database user |
| DB_PASSWORD | (required) | Database password |
| DB_HOST | localhost | Database host |
| DB_PORT | 5432 | Database port |

## License

This project is for educational and portfolio purposes. The Superstore dataset originates from Tableau's public sample data.
