# Cohort Retention Analysis on Brazilian E-Commerce (SQL)

## Methodology

The analysis was performed using the Olist Brazilian E-Commerce Public Dataset.

### Data Loading

The `customers`, `orders`, and `order_items` tables were loaded into a SQLite database.

The `customers` and `orders` tables were used for the cohort retention analysis, while `order_items` was used to calculate revenue by cohort and period.

### Data Filtering

Only orders with `order_status = 'delivered'` were included in the analysis.

This resulted in 96,478 delivered orders.

### Customer Identification

`customer_unique_id` was used as the customer identifier instead of `customer_id`.

This is important because `customer_unique_id` represents the same real customer across different orders, while `customer_id` can represent individual customer-order records.

Using `customer_unique_id` allows repeat purchases from the same customer to be tracked correctly.

### Cohort Assignment

Each customer was assigned to a monthly cohort based on their first delivered purchase.

The cohort month was calculated in SQL using:

```sql
MIN(strftime('%Y-%m', order_purchase_timestamp))
OVER (PARTITION BY customer_unique_id)
```

The analysis produced 23 monthly cohorts.

### Period Calculation

For each customer order, the number of months between the order month and the cohort month was calculated as `period_number`.

- Period 0 = cohort month
- Period 1 = one month after the cohort month
- Period 2 = two months after the cohort month
- and so on

The period calculation was performed entirely in SQL using year-and-month arithmetic.

### Retention Matrix

The retention matrix was created by grouping customers by:

- `cohort_month`
- `period_number`

The number of active customers was calculated using:

```sql
COUNT(DISTINCT customer_unique_id)
```

This ensures that a customer is counted only once within each cohort and period.

### Retention Rate

Retention rate was calculated as the percentage of the original cohort that remained active in each period:

```text
Retention Rate = Active Customers / Cohort Size × 100
```

Period 0 is therefore 100% for every cohort.

### Average Retention

The average retention rate was calculated for each period across the observed cohorts.

The retention curve was then plotted to visualize how customer retention changed over time.

### Revenue Analysis

Revenue was calculated by cohort and period using the `order_items` table.

Revenue was defined as:

```text
price + freight_value
```

## Retention Results

The average retention results were:

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

The largest early decline occurs between period 1 and period 2:

```text
5.45% → 0.34%
```

This represents a decrease of **5.11 percentage points**.

## Cohort Comparison

Month-1 retention varied across cohorts.

Among cohorts with more than 100 customers:

- **2017-10:** 0.72% month-1 retention
- **2017-02:** 0.18% month-1 retention

This shows that repeat-purchase behavior was not identical across all cohorts.

Small cohorts were treated carefully because very small sample sizes can produce extreme retention percentages.

## Key Findings

- The analysis contains **96,478 delivered orders** and **23 monthly cohorts**.
- Period 0 retention is **100%** by definition.
- Average retention falls to **5.45% in period 1**.
- Average retention falls further to **0.34% in period 2**.
- Later-period retention generally remains below **1%**.
- Month-1 retention varies between cohorts, with larger cohorts ranging from **0.18% to 0.72%** in the observed comparison.
- Cohort sizes increased substantially over time, with examples including **262 customers in 2016-10**, **4,057 in 2017-08**, and **7,060 in 2017-11**.
- Revenue was calculated by cohort and period using `price + freight_value`.

## Business Insights

1. **Weak repeat purchasing:** The retention results indicate weak monthly repeat purchasing. Only 5.45% of customers were retained on average in period 1, and this decreased to 0.34% in period 2.

2. **Sharp early drop:** The 5.11 percentage-point decline from period 1 to period 2 shows that the largest loss in repeat activity occurs early in the customer lifecycle.

3. **Cohort differences:** Month-1 retention varied across cohorts. For example, the 2017-10 cohort had 0.72% retention compared with 0.18% for the 2017-02 cohort among the larger cohorts examined.

4. **Potential marketplace behavior:** The low later-period retention may indicate that many purchases are occasional rather than frequent monthly purchases. This is a hypothesis based on the observed pattern and would require additional analysis to identify the underlying causes.

## Visualizations

The project includes:

- **Retention Heatmap** — shows retention percentages by cohort and period.
- **Average Retention Curve** — shows how average retention changes over time.
- **Cohort Size Bar Chart** — shows the number of customers entering each monthly cohort.

## Notes

The conceptual task defines a cohort using a customer's first-ever purchase. However, Q1 restricts the analysis to delivered orders. Therefore, the implemented cohort represents the customer's **first delivered purchase within the analyzed population**.

Later periods contain fewer observable cohorts because newer cohorts do not have enough time to reach those periods before the dataset ends.

The retention analysis describes observed customer purchasing behavior and does not by itself establish the causes of low retention.
