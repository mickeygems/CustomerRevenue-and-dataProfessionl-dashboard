# Customer Revenue Intelligence Dashboard

End-to-end customer analytics — from raw shopping data to actionable revenue insights using **Python**, **PostgreSQL**, and **Power BI**.

[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-SQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?style=flat-square&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com)

---

## Overview

Analyzes **3,900+ customer shopping behavior records** to answer business questions:

- Who are the most valuable customer segments?
- Which categories drive revenue vs. volume?
- Does discounting help — or erode margin silently?

**Stack:** Python · Pandas · Matplotlib · Seaborn · SQLAlchemy · PostgreSQL · Power BI

---

## Dashboard Preview

![Customer Revenue Dashboard](Customer_Revenue_dashboard_screenshot.png)

| Metric | Value |
|---|---|
| Total Customers | 3,900+ |
| Avg Purchase Amount | $59.76 |
| Avg Review Rating | 3.75 |
| Product Categories | Clothing, Accessories, Footwear, Outerwear |

---

## Key Insights

1. **Clothing dominates revenue and volume** — high volume can mask low profitability without margin data.
2. **73% of customers are non-subscribers** — subscribed customers show higher average spend (conversion opportunity).
3. **Young Adults and Middle-Aged customers drive the most revenue** — priority segments for retention and upsell.
4. **Discount abuse risk** — several products show 40%+ discount rates while still meeting average purchase thresholds.
5. **Loyal repeat buyers (>5 purchases) often don't subscribe** — highest-probability, lowest-cost conversion cohort.

---

## Project Structure

```
Customer-Revenue-Intelligence-Dashboard/
├── customer_shopping_behavior_dataset.csv          # Raw dataset
├── EDA_Perform_on_dataset_jupyternotebook.ipynb    # EDA, cleaning, feature engineering, DB load
├── Customer_revenue_Postgresql_queriess.sql        # 10 business SQL queries
├── Customer_Revenue_Inteliigence_Dashboard_PowerBI.pbit
├── Customer_Revenue_dashboard_screenshot.png
├── Customer_Behavior_pdf.pdf
└── README.md
```

---

## Workflow

### 1. Load & Explore (Python + Pandas)

Profile shape, dtypes, and distributions across 3,900+ records.

### 2. Clean

- Impute missing review ratings with category-level median
- Standardize column names
- Drop redundant `promo_code_used` (100% identical to `discount_applied`)

### 3. Feature Engineering

- Age segments via quantile bins (`Young Adult`, `Adult`, `Middle aged`, `Senior`)
- Purchase frequency mapped to days

### 4. Load to PostgreSQL

```python
import os
from sqlalchemy import create_engine

password = os.environ.get("DB_PASSWORD", "your_password")
engine = create_engine(
    f"postgresql+psycopg2://postgres:{password}@localhost:5432/customer_behavior"
)
df.to_sql("customer", engine, if_exists="replace", index=False)
```

### 5. SQL Business Analysis

10 queries using CTEs, window functions, `CASE`, and aggregations — see [`Customer_revenue_Postgresql_queriess.sql`](Customer_revenue_Postgresql_queriess.sql).

**Sample — customer segmentation CTE:**

```sql
WITH customer_type AS (
    SELECT customer_id, previous_purchases,
        CASE
            WHEN previous_purchases = 1 THEN 'New'
            WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
            ELSE 'Loyal'
        END AS customer_segment
    FROM customer
)
SELECT customer_segment, COUNT(*) AS "Number of Customers"
FROM customer_type
GROUP BY customer_segment;
```

### 6. Power BI Dashboard

Connected PostgreSQL to Power BI with slicers for subscription status, gender, category, and shipping type.

- **Executive Summary** — KPIs
- **Revenue Analysis** — by category and age group
- **Customer Segmentation** — subscription, discount behavior, repeat buyers

---

## Recommendations

1. **Loyalty → subscription campaign** for high-frequency non-subscribers.
2. **Audit discounts** on products with 40%+ discount rates.
3. **Prioritize Young Adult retention** — highest revenue segment.

---

## How to Run

**Prerequisites:** Python 3.8+, PostgreSQL, Power BI Desktop

```bash
git clone https://github.com/Znaxh/Customer-Revenue-Intelligence-Dashboard.git
cd Customer-Revenue-Intelligence-Dashboard

pip install pandas numpy matplotlib seaborn sqlalchemy psycopg2-binary jupyter

# Create DB: customer_behavior in PostgreSQL
# Set DB_PASSWORD, then run the notebook
jupyter notebook EDA_Perform_on_dataset_jupyternotebook.ipynb
```

Then run the SQL file in pgAdmin / `psql`, and open the Power BI template.

---

## Dataset

Source: [Customer Shopping Behavior Dataset — Kaggle](https://www.kaggle.com/)

| Column | Description |
|---|---|
| customer_id | Unique customer identifier |
| age / age_group | Age and derived segment |
| gender | Male / Female |
| item_purchased | Product name |
| category | Clothing, Accessories, Footwear, Outerwear |
| purchase_amount | Transaction value (USD) |
| review_rating | Product rating (1–5) |
| subscription_status | Yes / No |
| shipping_type | Standard, Express, etc. |
| discount_applied | Whether discount was used |
| previous_purchases | Past transaction count |
| purchase_frequency_days | Engineered frequency in days |

---

## Security Note

Do not commit real database passwords. Use environment variables:

```python
password = os.environ.get("DB_PASSWORD")
```
