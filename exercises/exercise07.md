# Module 7 - Final Project: Show Us Your Data

- **Name:** Kiruthikaa Natarajan Srinivasan  
- **Course:** Databases for Analytics  
- **Module:** 7  
- **Tooling:** PostgreSQL + pgAdmin 4 (Windows 11)

---

## Overview

For this final project, I located a real-world dataset, imported it into PostgreSQL, transformed it into a structured relational database, and verified the results using SQL queries including joins and aggregations.

Given my professional background in supply chain and EDI project management, I intentionally selected a procurement dataset that aligns with my industry experience. This allowed me to model real procurement data in a structured analytical format.

The final structure includes:
- Two dimension-style tables: `products`, `origins`
- One fact-style table: `procurement_fact`

---

## 1) Initial Data Source

I used the public Kaggle dataset:

**Gourmet Food Procurement Data**  
https://www.kaggle.com/datasets/anoopjohny/gourmet-food-procurement-data  

This dataset contains procurement records including agency purchases, product names, vendor/distributor details, quantities, weights, and total cost.

---

## 2) Data Format (Rows + Columns)

- **Format:** CSV  
- **File Name:** `Good_Food_Purchasing_Data.csv`  
- **Rows:** 17,208  
- **Columns:** 12  

The dataset was initially imported into a staging table before being normalized into relational tables.

---

## 3) Database Tables Created (Complex Structure)

Instead of keeping the dataset as a single flat file, I normalized it into three related tables:

- `products`
- `origins`
- `procurement_fact`

To verify row counts:

```sql
SELECT 'products' AS table_name, COUNT(*) AS rows FROM products
UNION ALL
SELECT 'origins', COUNT(*) FROM origins
UNION ALL
SELECT 'procurement_fact', COUNT(*) FROM procurement_fact;
```

![Row Counts](screenshots/Mod%207_All%20Tables.png)

This confirms:

- products → 5751 rows  
- origins → 5094 rows  
- procurement_fact → 17207 rows  

This satisfies the rubric requirement for a significantly complex database structure with thousands of rows.

---

## 4) Data Dictionary

### products

| Column | Type | Description |
|--------|------|-------------|
| product_id | integer | Primary key for products |
| product_name | text | Name/description of the product |

### origins

| Column | Type | Description |
|--------|------|-------------|
| origin_id | integer | Primary key for origins |
| origin_detail | text | Origin information (missing values handled using `UNKNOWN`) |

### procurement_fact

| Column | Type | Description |
|--------|------|-------------|
| fact_id | integer | Primary key for the fact table |
| product_id | integer | Foreign key to `products.product_id` |
| origin_id | integer | Foreign key to `origins.origin_id` |
| agency | text | Purchasing agency |
| time_period | text | Original time period from source file |
| vendor | text | Vendor name |
| distributor | text | Distributor name |
| units | integer | Units purchased |
| total_weight_lbs | numeric | Total weight in pounds |
| total_cost | numeric | Total procurement cost |
| record_date | date | Converted DATE field created from `time_period` |

---

## 5) Obstacles and Data Transformation

Challenges encountered:

- The dataset began as one large CSV file.
- Some `origin_detail` values were missing.
- `time_period` was stored as text.
- Numeric fields required correct typing to support aggregation.

How I addressed them:

- Imported the CSV into a staging table (`procurement_raw`) first.
- Created dimension tables using `SELECT DISTINCT`.
- Inserted an `UNKNOWN` origin record to handle null values.
- Converted `time_period` into a proper DATE column (`record_date`).
- Ensured numeric fields (`units`, `total_cost`, `total_weight_lbs`) were stored with appropriate data types.

This transformation demonstrates converting raw operational data into structured analytical information.

---

## 6) Table Structure (Data Types)

To verify data types:

```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'procurement_fact'
ORDER BY ordinal_position;
```

![Table Structure](screenshots/Mod%207_3.png)

This confirms use of:

- integer  
- text  
- numeric  
- date  

---

## 7) SELECT * From Each Table

### products

```sql
SELECT * FROM products LIMIT 10;
```

![Select Products](screenshots/Mod%207_Select%20Products.png)

---

### origins

```sql
SELECT * FROM origins LIMIT 10;
```

![Select Origins](screenshots/Mod%207_Select%20Origins.png)

---

### procurement_fact

```sql
SELECT * FROM procurement_fact LIMIT 10;
```

![Select Fact](screenshots/Mod%207_Select%20Fact.png)

These queries confirm successful data import into all tables.

---

## 8) Join Query (Required)

This query joins the fact table with both dimension tables and aggregates total procurement spend.

```sql
SELECT p.product_name,
       o.origin_detail,
       SUM(f.total_cost) AS total_spend
FROM procurement_fact f
JOIN products p ON f.product_id = p.product_id
JOIN origins o ON f.origin_id = o.origin_id
GROUP BY p.product_name, o.origin_detail
ORDER BY total_spend DESC
LIMIT 10;
```

![Join Query](screenshots/Mod%207_1.png)

This result highlights the highest-spending product and origin combinations, demonstrating how relational joins enable meaningful procurement insights.

---

## 9) Group By + Aggregate Query (Required)

This query summarizes total spending by agency.

```sql
SELECT agency,
       SUM(total_cost) AS total_spend
FROM procurement_fact
GROUP BY agency
ORDER BY total_spend DESC
LIMIT 10;
```

![Aggregate Query](screenshots/Mod%207_2.png)

This analysis shows variation in procurement spend across agencies, enabling comparative evaluation of purchasing activity.

---

## 10) Date-Based Aggregation (Transformation Proof)

This query verifies that the converted `record_date` field supports time-based analysis.

```sql
SELECT record_date,
       SUM(total_cost) AS yearly_spend
FROM procurement_fact
GROUP BY record_date
ORDER BY record_date;
```

![Date Aggregation](screenshots/Mod%207_Date.png)

This confirms that the dataset was successfully transformed from text-based time values into analyzable date-based metrics.

---

## Final Summary

This project demonstrates:

- Importing a complex real-world dataset
- Transforming raw CSV data into a structured relational model
- Creating multiple related tables
- Applying appropriate data types
- Executing joins
- Executing grouped aggregation queries
- Performing time-based analytics

