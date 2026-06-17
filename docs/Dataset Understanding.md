# Phase 2: Dataset Understanding - Superstore Sales Dataset

---

## 1. Dataset Overview

| Property           | Value                          |
|--------------------|--------------------------------|
| **File**           | `Superstore sales dataset.csv` |
| **Encoding**       | UTF-8                          |
| **Total Rows**     | 9,994                          |
| **Total Columns**  | 21                             |
| **Date Range**     | January 3, 2014 - December 30, 2017 (4 full years) |
| **Geography**      | United States (single country) |
| **Domain**         | Retail / E-commerce Sales      |
| **Granularity**    | One row = one line item within an order |

The dataset simulates transactional sales data for a fictional U.S.-based retail superstore. Each record represents a single product line item belonging to a customer order, capturing the full lifecycle from order placement to shipment.

---

## 2. Column Dictionary

### 2.1 Identifiers & Temporal Columns

| # | Column       | Data Type | Description |
|---|--------------|-----------|-------------|
| 1 | **Row ID**       | Integer   | Sequential unique identifier for each record (1 – 9,994). |
| 2 | **Order ID**     | String    | Unique order identifier. Format: `{Prefix}-{Year}-{Sequence}` (e.g., `CA-2016-152156`). Prefix is either `CA` or `US`. |
| 3 | **Order Date**   | String (Date) | Date the order was placed. Format: `D/M/YYYY` (e.g., `8/11/2016` = Nov 8, 2016). |
| 4 | **Ship Date**    | String (Date) | Date the order was shipped. Format: `D/M/YYYY`. |
| 5 | **Ship Mode**    | String (Categorical) | Shipping method. 4 values: Standard Class (59.7%), Second Class (19.5%), First Class (15.4%), Same Day (5.4%). |

### 2.2 Customer Columns

| # | Column           | Data Type | Description |
|---|------------------|-----------|-------------|
| 6 | **Customer ID**      | String    | Unique customer identifier. Format: `{Initials}-{5-digit number}` (e.g., `CG-12520`). |
| 7 | **Customer Name**    | String    | Full name of the customer. 793 unique customers across 9,994 transactions. |
| 8 | **Segment**          | String (Categorical) | Customer business segment. 3 values: Consumer (51.9%), Corporate (30.2%), Home Office (17.8%). |

### 2.3 Geographic Columns

| # | Column          | Data Type | Description |
|---|-----------------|-----------|-------------|
| 9  | **Country**       | String    | Always `United States` - single-country dataset. |
| 10 | **City**          | String    | City name. 531 unique cities. |
| 11 | **State**         | String    | U.S. state name. 49 unique states (includes Washington D.C.). |
| 12 | **Postal Code**   | Integer   | 5-digit U.S. ZIP code. 631 unique codes. No missing values. |
| 13 | **Region**        | String (Categorical) | U.S. census-style region. 4 values: West (32.0%), East (28.5%), Central (23.2%), South (16.2%). |

### 2.4 Product Columns

| # | Column           | Data Type | Description |
|---|------------------|-----------|-------------|
| 14 | **Product ID**       | String    | Unique product identifier. Format: `{Category}-{SubCat}-{Number}` (e.g., `FUR-BO-10001798`). 1,862 unique products. |
| 15 | **Category**         | String (Categorical) | Top-level product category. 3 values: Office Supplies (60.3%), Furniture (21.2%), Technology (18.5%). |
| 16 | **Sub-Category**     | String (Categorical) | Product sub-category. 17 values (e.g., Binders, Paper, Phones, Chairs, etc.). |
| 17 | **Product Name**     | String    | Descriptive product name. 1,850 unique names (some products share names). |

### 2.5 Financial Columns

| # | Column        | Data Type | Description |
|---|---------------|-----------|-------------|
| 18 | **Sales**     | Float     | Revenue amount in USD. Range: $0.44 – $22,638.48. Mean: $229.86. |
| 19 | **Quantity**  | Integer   | Number of units ordered. Range: 1 – 14. Mean: 3.79. |
| 20 | **Discount**  | Float     | Discount applied (proportion). Range: 0.00 – 0.80. Mean: 0.156. |
| 21 | **Profit**    | Float     | Net profit in USD. Range: −$6,599.98 – $8,399.98. Mean: $28.66. |

---

## 3. Missing Values Assessment

| Column | Missing Count | Missing % |
|--------|:---:|:---:|
| Row ID | 0 | 0.00% |
| Order ID | 0 | 0.00% |
| Order Date | 0 | 0.00% |
| Ship Date | 0 | 0.00% |
| Ship Mode | 0 | 0.00% |
| Customer ID | 0 | 0.00% |
| Customer Name | 0 | 0.00% |
| Segment | 0 | 0.00% |
| Country | 0 | 0.00% |
| City | 0 | 0.00% |
| State | 0 | 0.00% |
| Postal Code | 0 | 0.00% |
| Region | 0 | 0.00% |
| Product ID | 0 | 0.00% |
| Category | 0 | 0.00% |
| Sub-Category | 0 | 0.00% |
| Product Name | 0 | 0.00% |
| Sales | 0 | 0.00% |
| Quantity | 0 | 0.00% |
| Discount | 0 | 0.00% |
| Profit | 0 | 0.00% |

**Verdict:** The dataset is complete. **Zero missing values** across all 21 columns and all 9,994 rows. No imputation is required.

---

## 4. Date Format Analysis

### Format Detected: `D/M/YYYY` (Day/Month/Year)

| Property | Order Date | Ship Date |
|----------|------------|-----------|
| Format | `D/M/YYYY` | `D/M/YYYY` |
| Successfully parsed | 9,994 / 9,994 (100%) | 9,994 / 9,994 (100%) |
| Earliest date | January 3, 2014 | January 7, 2014 |
| Latest date | December 30, 2017 | January 5, 2018 |

> **Important Note:** The dates use the **day-first** format (`D/M/YYYY`), not the U.S.-style month-first (`M/D/YYYY`). When importing this dataset into tools like Excel or pandas, explicitly specifying `dayfirst=True` is critical to avoid misinterpreting dates (e.g., `12/6/2016` is **June 12**, not December 6).

### Shipping Duration Derived Metric

| Metric | Value |
|--------|-------|
| Minimum shipping days | 0 days (Same Day) |
| Maximum shipping days | 7 days |
| Average shipping days | 4.0 days |
| Anomalies (ship before order) | **0** - No negative durations found |

Shipping duration is perfectly consistent with the Ship Mode labels (Same Day = 0, First Class ≈ 1–2, Second Class ≈ 3–4, Standard Class ≈ 5–7).

---

## 5. Duplicate Analysis

| Check | Result |
|-------|--------|
| Exact duplicate rows | **0** - No full-row duplicates |
| Duplicate (Order ID + Product ID) combinations | **8** - These represent legitimate cases where the same product appears multiple times in the same order (e.g., separate line items with different quantities) |

**Verdict:** The dataset is clean regarding duplicates. The 8 repeated Order ID + Product ID pairs are valid business scenarios, not data errors.

---

## 6. Key Data Quality Observations

### 6.1 Negative Profit (Loss-Making Transactions)

| Metric | Value |
|--------|-------|
| Rows with negative profit | 1,871 (18.7% of dataset) |
| Total cumulative loss | **−$156,131.29** |

This is a significant finding - nearly 1 in 5 transactions results in a loss. This will be a critical area for exploratory analysis.

### 6.2 Discount Distribution

| Metric | Value |
|--------|-------|
| Transactions with zero discount | 4,798 (48.0%) |
| Average discount | 15.6% |
| Maximum discount | 80% |
| Unique discount levels | 12 (suggests standardized discount tiers) |

Discounts appear at fixed intervals (0.0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8), confirming they are policy-driven, not ad hoc.

### 6.3 Outliers & Distribution Skewness

| Metric | Sales | Profit |
|--------|-------|--------|
| IQR | $192.66 | $27.64 |
| IQR-based outliers | 1,167 | 1,881 |
| Skewness | 12.973 (extreme right skew) | 7.561 (strong right skew) |

Both Sales and Profit are **heavily right-skewed**, meaning most transactions are relatively small while a few very large transactions pull the mean significantly above the median. This has implications for any statistical modeling - log transformations or robust methods may be needed.

### 6.4 Geographic Concentration

The top 3 states account for **41.2%** of all transactions:
- California: 2,001 records (20.0%)
- New York: 1,128 records (11.3%)
- Texas: 985 records (9.9%)

This heavy concentration should be noted when analyzing regional performance.

---

## 7. Dataset Strengths & Limitations

### Strengths
- **Clean data:** No missing values, no duplicates, consistent formatting
- **Rich dimensionality:** 21 columns covering temporal, geographic, customer, product, and financial dimensions
- **4-year time span:** Enables trend analysis, seasonality detection, and year-over-year comparisons
- **Realistic business scenario:** Includes loss-making transactions, varied discount strategies, and multiple customer segments

### Limitations
- **Single country:** All data is from the United States - no international analysis possible
- **No customer demographics:** Age, gender, income, etc. are not available
- **No product cost breakdown:** Only final Sales and Profit are given - COGS must be inferred
- **Fictional data:** This is a widely-used sample dataset (originally from Tableau), not from a real business
- **High skewness:** Extreme outliers in Sales/Profit may distort aggregate analyses

---

## 8. Summary Statistics at a Glance

| Metric | Value |
|--------|-------|
| Total Records | 9,994 |
| Unique Orders | ~5,009 (estimated from Order ID) |
| Unique Customers | 793 |
| Unique Products | 1,862 |
| U.S. States Covered | 49 |
| U.S. Cities Covered | 531 |
| Total Revenue (Sales) | ~$2,297,201 |
| Total Profit | ~$286,397 |
| Average Profit Margin | ~12.5% |
| Date Format | D/M/YYYY |
| Missing Values | 0 |
| Duplicate Rows | 0 |

---

*Document prepared as part of Phase 2: Understand the Data - Superstore Sales Analytics Project*