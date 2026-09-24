# Cohort Retention Analysis on Brazilian E-Commerce (SQL)

A SQL-based cohort retention analysis using the Olist Brazilian E-Commerce Public Dataset.

The project focuses on measuring customer retention over time, comparing monthly cohorts, analyzing repeat-purchase behavior, and calculating revenue by cohort and retention period.

The analysis uses **SQLite, SQL, pandas, seaborn, matplotlib, and Jupyter Notebook**.

## Project Objective

The main objective of this project is to build a monthly cohort retention analysis entirely in SQL.

The analysis covers:

- Delivered order filtering
- Customer cohort assignment
- Monthly customer activity
- Period number calculation
- Cohort-by-period retention matrix
- Retention rate matrix
- Average retention curve
- Month-1 cohort comparison
- Revenue by cohort and period
- Cohort size distribution

The cohort and period calculations are performed in SQL rather than pandas.

## Dataset

The project uses the **Olist Brazilian E-Commerce Public Dataset**.

### Dataset Source

The original dataset is available on Kaggle:

**[Olist Brazilian E-Commerce Public Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)**

The raw CSV files are **not included in this GitHub repository because of their file size**. To reproduce the analysis, download the dataset from Kaggle and place the required CSV files inside a local `data/` folder.

### Datasets Used

This project uses three datasets from the Olist collection:

| Dataset | File | Purpose |
|---|---|---|
| Customers | `olist_customers_dataset.csv` | Identify customers using `customer_unique_id` |
| Orders | `olist_orders_dataset.csv` | Filter delivered orders, assign cohorts, and calculate customer activity |
| Order Items | `olist_order_items_dataset.csv` | Calculate revenue by cohort and retention period |

The `customers` and `orders` tables are used for the main cohort retention analysis. The `order_items` table is joined with the customer activity data for the revenue analysis.

### Dataset Overview

| Metric | Result |
|---|---:|
| Customers rows | 99,441 |
| Orders rows | 99,441 |
| Order items rows | 112,650 |
| Delivered orders used | 96,478 |
| Monthly cohorts | 23 |

## Tools & Technologies

- **SQL / SQLite** - Main analysis and cohort calculations
- **Python** - Notebook environment
- **Pandas** - Loading and displaying SQL results
- **Seaborn** - Retention heatmap
- **Matplotlib** - Retention curve and cohort size visualization
- **Jupyter Notebook** - Analysis workflow

## Data Preparation

The datasets were loaded from CSV files into pandas and then stored as SQLite tables.

The database contains:

```text
customers
orders
order_items
```

The original tables were kept separate and joined using their relational keys.

The main customer-order relationship was created by joining:

```text
orders.customer_id
        ↓
customers.customer_id
        ↓
customer_unique_id
```

## Customer Identification

The analysis uses `customer_unique_id` as the customer identifier.

This is important because `customer_id` is associated with individual customer-order records, while `customer_unique_id` represents the same real customer across orders.

Using `customer_unique_id` allows repeat purchases from the same customer to be identified correctly.

## Cohort Assignment

Only orders with:

```sql
order_status = 'delivered'
```

were included.

This resulted in **96,478 delivered orders**.

Each customer was assigned to a monthly cohort based on their first delivered purchase.

The cohort month was calculated in SQL using:

```sql
MIN(strftime('%Y-%m', order_purchase_timestamp))
OVER (PARTITION BY customer_unique_id)
```

The analysis produced **23 monthly cohorts**.

> **Note:** The conceptual task describes the first-ever purchase. However, because Q1 restricts the dataset to delivered orders, the implemented cohort represents the customer's first delivered purchase within the analyzed population.

## Period Calculation

For every customer order, `period_number` represents the number of calendar months elapsed between the customer's cohort month and the order month.

| Period | Meaning |
|---:|---|
| 0 | Cohort month |
| 1 | One month later |
| 2 | Two months later |
| 3 | Three months later |

The period calculation was performed entirely in SQL using year-and-month arithmetic.

## Retention Matrix

The retention matrix contains:

- **Rows:** cohort month
- **Columns:** period number
- **Cells:** number of unique active customers

Customer activity was calculated using:

```sql
COUNT(DISTINCT customer_unique_id)
```

This ensures that a customer is counted only once within each cohort and period.

Period 0 represents the original cohort size.

## Retention Rate

Retention rate was calculated relative to each cohort's own period-0 size:

```text
Retention Rate = Active Customers / Cohort Size × 100
```

Therefore, period 0 is always 100%.

This normalization allows cohorts of different sizes to be compared.

## Cohort Size

The cohort size represents the number of customers in period 0.

| Cohort | Customers |
|---|---:|
| 2016-10 | 262 |
| 2017-01 | 717 |
| 2017-02 | 1,628 |
| 2017-03 | 2,503 |
| 2017-04 | 2,256 |
| 2017-05 | 3,451 |
| 2017-06 | 3,037 |
| 2017-07 | 3,752 |
| 2017-08 | 4,057 |
| 2017-09 | 4,004 |
| 2017-10 | 4,328 |
| 2017-11 | 7,060 |
| 2017-12 | 5,338 |
| 2018-01 | 6,842 |
| 2018-02 | 6,288 |
| 2018-03 | 6,774 |
| 2018-04 | 6,582 |
| 2018-05 | 6,506 |

## Average Retention

The average retention rate was calculated across observed cohorts for each period.

| Period | Average Retention |
|---:|---:|
| 1 | 5.45% |
| 2 | 0.34% |
| 3 | 0.25% |
| 4 | 0.29% |
| 5 | 0.23% |
| 6 | 0.27% |
| 7 | 0.21% |
| 8 | 0.21% |
| 9 | 0.19% |
| 10 | 0.27% |
| 11 | 0.23% |
| 12 | 0.21% |
| 13 | 0.20% |
| 14 | 0.15% |
| 15 | 0.19% |
| 16 | 0.14% |
| 17 | 0.28% |
| 19 | 0.45% |
| 20 | 0.76% |

The main early change is:

```text
Period 1 → 5.45%
Period 2 → 0.34%
```

This represents a decrease of **5.11 percentage points**.

## Month-1 Cohort Comparison

Month-1 retention was calculated for each cohort using period 1.

Among cohorts with more than 100 customers:

| Cohort | Month-1 Retention |
|---|---:|
| 2017-02 | 0.18% |
| 2017-03 | 0.44% |
| 2017-04 | 0.62% |
| 2017-05 | 0.46% |
| 2017-06 | 0.49% |
| 2017-07 | 0.53% |
| 2017-08 | 0.69% |
| 2017-09 | 0.70% |
| 2017-10 | 0.72% |
| 2017-11 | 0.57% |
| 2017-12 | 0.21% |
| 2018-01 | 0.34% |
| 2018-02 | 0.35% |
| 2018-03 | 0.40% |
| 2018-04 | 0.59% |
| 2018-05 | 0.52% |

Small cohorts can produce extreme percentages, so the comparison focuses on cohorts with more than 100 customers.

## Revenue Analysis

Revenue was calculated by cohort and period using the `order_items` table.

The revenue definition is:

```text
Revenue = price + freight_value
```

Revenue was grouped by:

- `cohort_month`
- `period_number`

Examples from the analysis include:

| Cohort | Period | Revenue |
|---|---:|---:|
| 2016-09 | 0 | 143.46 |
| 2016-10 | 0 | 46,490.66 |
| 2016-10 | 6 | 111.30 |
| 2016-10 | 9 | 356.13 |
| 2016-12 | 0 | 19.62 |
| 2016-12 | 1 | 19.62 |
| 2017-01 | 0 | 127,462.75 |
| 2017-01 | 1 | 111.07 |
| 2017-01 | 2 | 114.70 |

## Visualizations

### Retention Heatmap

The heatmap shows retention percentages by cohort and period.

- **Rows** → cohort months
- **Columns** → retention periods
- **Cells** → retention rate

### Average Retention Curve

The line chart shows how average customer retention changes across retention periods.

### Cohort Size Bar Chart

The bar chart shows the number of customers entering each monthly cohort.

## Business Insights

### Weak repeat purchasing

Average retention is 5.45% in period 1 and falls to 0.34% in period 2. Later-period retention generally remains below 1% within the observed periods.

### Sharp early retention decline

The decrease from 5.45% to 0.34% between period 1 and period 2 represents a 5.11 percentage-point decline.

### Cohort differences

Month-1 retention varies across cohorts. For example, the 2017-10 cohort has 0.72% month-1 retention, while the 2017-02 cohort has 0.18% among the larger cohorts examined.

### Growing cohort sizes

Later cohorts are substantially larger than many early cohorts. For example, the 2016-10 cohort contains 262 customers, while the 2017-11 cohort contains 7,060 customers.

## Key Findings

- 96,478 delivered orders were included in the analysis.
- The analysis contains 23 monthly cohorts.
- Period 0 retention is 100% by definition.
- Average period-1 retention is 5.45%.
- Average period-2 retention is 0.34%.
- The period-1 to period-2 decline is 5.11 percentage points.
- Month-1 retention varies across cohorts.
- Cohort sizes increased substantially over the observed period.
- Revenue was calculated by cohort and period using `price + freight_value`.
- The results indicate weak monthly repeat purchasing within the observed customer population.

## Limitations

This analysis is based on the available Olist dataset and its historical order records.

The cohort definition follows the delivered-order population because Q1 requires filtering to `order_status = 'delivered'`.

Small cohorts can produce extreme retention percentages. For example, a cohort containing one customer can show 100% retention if that customer returns.

Later retention periods contain fewer observable cohorts because newer cohorts do not have enough historical time to reach those periods before the dataset ends.

The analysis describes observed purchasing behavior but does not establish the causes of low retention. Additional segmentation by product category, customer state, order value, or other variables would be required to investigate potential causes.

## Conclusion

This project demonstrates an end-to-end SQL cohort retention workflow.

The analysis shows that the Olist customer population has weak monthly repeat purchasing, with average retention falling from 5.45% in period 1 to 0.34% in period 2.

The project combines SQL CTEs, window functions, date functions, month-difference calculations, retention matrices, cohort comparisons, revenue analysis, and visualization to provide a structured view of customer retention behavior.

## Reproducing the Analysis

After downloading the dataset from Kaggle, place the three required CSV files in:

```text
data/
├── olist_customers_dataset.csv
├── olist_orders_dataset.csv
└── olist_order_items_dataset.csv
```

The notebook then loads these files into pandas and creates the corresponding SQLite tables before running the SQL analysis.

## Project Structure

```text
cohort-retention-analysis/
│
├── visuals/
│   ├── retention_heatmap.png
│   ├── average_retention_curve.png
│   └── cohort_size_chart.png
│
├── cohort_retention_analysis.ipynb
├── queries.sql
├── note.md
└── README.md
```
