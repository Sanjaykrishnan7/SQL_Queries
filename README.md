# SQL_Queries

# 🛒 Walmart Sales Analysis — End-to-End SQL + Python Project

## 📌 Overview

This project demonstrates a complete data pipeline for analyzing Walmart sales data using **Python for data processing** and **MySQL for business query analysis**. The goal is to uncover key sales insights, customer behavior, and performance trends across branches.

---

## 🔧 Tools & Technologies

- **Languages**: Python, SQL
- **Libraries**: `pandas`, `sqlalchemy`, `pymysql`
- **Database**: MySQL
- **Environment**: Jupyter Notebook, MySQL Workbench / phpMyAdmin

---

## 📁 Project Workflow

### 1. Load and Inspect Raw Data
- Read from CSV using Pandas.
- Perform exploratory analysis using `.head()`, `.info()`, `.describe()`.

### 2. Data Cleaning
- Remove duplicates and handle nulls.
- Format data types correctly (especially for `unit_price`, `date`, `time`).
- Add a computed `total` column:
  ```python
  df["total"] = df["unit_price"] * df["quantity"]
  ```

### 3. Load Data into MySQL
- Connect using SQLAlchemy with PyMySQL driver.
- Push cleaned DataFrame to a MySQL table `walmart`.

### 4. SQL Analysis — Key Business Questions

> All queries below are executed on the `walmart` table.

#### 🔹 Basic Stats

```sql
SELECT COUNT(*) FROM walmart;
SELECT COUNT(DISTINCT branch) FROM walmart;
SELECT MIN(quantity) FROM walmart;
```

#### 🔹 Q1: Payment Methods & Transactions

```sql
SELECT 
    payment_method,
    COUNT(*) AS no_payments,
    SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method;
```

#### 🔹 Q2: Highest-rated Category per Branch

```sql
SELECT branch, category, avg_rating
FROM (
    SELECT 
        branch,
        category,
        AVG(rating) AS avg_rating,
        RANK() OVER(PARTITION BY branch ORDER BY AVG(rating) DESC) AS rnk
    FROM walmart
    GROUP BY branch, category
) AS ranked
WHERE rnk = 1;
```

#### 🔹 Q3: Busiest Day for Each Branch

```sql
SELECT branch, day_name, no_transactions
FROM (
    SELECT 
        branch,
        DAYNAME(STR_TO_DATE(date, '%d/%m/%Y')) AS day_name,
        COUNT(*) AS no_transactions,
        RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS rnk
    FROM walmart
    GROUP BY branch, day_name
) AS ranked
WHERE rnk = 1;
```

#### 🔹 Q4: Total Quantity per Payment Method

```sql
SELECT 
    payment_method,
    SUM(quantity) AS total_qty_sold
FROM walmart
GROUP BY payment_method;
```

#### 🔹 Q5: Rating Stats Per City & Category

```sql
SELECT 
    city,
    category,
    MIN(rating) AS min_rating,
    MAX(rating) AS max_rating,
    AVG(rating) AS avg_rating
FROM walmart
GROUP BY city, category;
```

#### 🔹 Q6: Total Profit Per Category

```sql
SELECT 
    category,
    SUM(total * profit_margin) AS total_profit
FROM walmart
GROUP BY category
ORDER BY total_profit DESC;
```

#### 🔹 Q7: Most Common Payment Method per Branch

```sql
WITH cte AS (
    SELECT 
        branch,
        payment_method,
        COUNT(*) AS total_trans,
        RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS rnk
    FROM walmart
    GROUP BY branch, payment_method
)
SELECT branch, payment_method AS preferred_payment_method
FROM cte
WHERE rnk = 1;
```

#### 🔹 Q8: Categorize Sales by Shift

```sql
SELECT
    branch,
    CASE 
        WHEN HOUR(TIME(time)) < 12 THEN 'Morning'
        WHEN HOUR(TIME(time)) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS shift,
    COUNT(*) AS num_invoices
FROM walmart
GROUP BY branch, shift
ORDER BY branch, num_invoices DESC;
```

#### 🔹 Q9: Branches with Highest Revenue Decrease (2022 → 2023)

```sql
WITH revenue_2022 AS (
    SELECT 
        branch,
        SUM(total) AS revenue
    FROM walmart
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) = 2022
    GROUP BY branch
),
revenue_2023 AS (
    SELECT 
        branch,
        SUM(total) AS revenue
    FROM walmart
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) = 2023
    GROUP BY branch
)
SELECT 
    r2022.branch,
    r2022.revenue AS last_year_revenue,
    r2023.revenue AS current_year_revenue,
    ROUND(((r2022.revenue - r2023.revenue) / r2022.revenue) * 100, 2) AS revenue_decrease_ratio
FROM revenue_2022 AS r2022
JOIN revenue_2023 AS r2023 ON r2022.branch = r2023.branch
WHERE r2022.revenue > r2023.revenue
ORDER BY revenue_decrease_ratio DESC
LIMIT 5;
```

---

## 📈 Insights

- **E-Wallet** is often the most used payment method.
- Peak shopping hours vary across branches but mostly peak in **Evening**.
- **Health & Beauty** and **Electronic Accessories** are top-rated categories in most branches.
- Some branches show significant revenue decline — this identifies opportunities for improvement.

---

## 📦 Folder Structure

```
project/
│
├── project.ipynb              # Python ETL and Data Processing
├── walmart_clean_data.csv     # Cleaned data
├── walmart_queries.sql        # SQL queries
├── README.md                  # Project documentation
```

---

## ✅ Setup Instructions

1. Ensure MySQL is running and database `walmart_db` is created.
2. Install dependencies:
   ```bash
   pip install pandas sqlalchemy pymysql
   ```
3. Run `project.ipynb` to load and clean data.
4. Execute SQL queries on the `walmart` table in your MySQL environment.
