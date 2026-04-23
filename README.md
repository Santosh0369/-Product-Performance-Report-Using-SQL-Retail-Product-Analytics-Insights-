# 📊 Product Performance Report Using SQL (Retail Product Analytics & Insights)

---

## 1️⃣ Executive Summary

This project focuses on building a **comprehensive product performance report** using SQL on a retail dataset. The goal is to evaluate how individual products contribute to overall business performance by analyzing sales, revenue, and customer interactions.

By combining transactional data with product attributes, the analysis provides actionable insights into product success, enabling better inventory planning, pricing strategies, and business decision-making.

---

## 2️⃣ Business Problem

Retail companies often face challenges such as:

* Identifying top-performing and underperforming products
* Understanding which products drive the most revenue
* Making informed decisions on pricing and inventory
* Tracking product-level performance over time

Without a structured reporting system, businesses struggle to optimize their product portfolio.

This project addresses these challenges through a SQL-driven product analysis report.

---

# 📦 Product Performance Analytics (SQL)

## 📌 Overview

This project creates a **product-level analytics report** using SQL by building the view:
`gold.report_products`.

It transforms raw transactional data into **insightful product performance metrics**, helping businesses evaluate product success, customer reach, and revenue contribution.

---

## 🎯 Objectives

* Analyze product performance across multiple dimensions
* Segment products based on **revenue contribution**
* Calculate essential **product KPIs**
* Support decision-making for **inventory, pricing, and marketing**

---

## 🏗️ Data Model

The report is built using:

* `gold.fact_sales` → sales transactions
* `gold.dim_products` → product details

---

## ⚙️ Key Features

### 1️⃣ Product Information Enrichment

* Product Name
* Category & Subcategory
* Cost

---

### 2️⃣ Product Segmentation

Products are classified based on **total revenue**:

* 🔥 **High-Performer** → Sales > 50,000
* ⚖️ **Mid-Range** → Sales between 10,000–50,000
* 📉 **Low-Performer** → Sales < 10,000

---

### 3️⃣ Aggregated Metrics

* 🧾 Total Orders
* 👥 Total Customers (unique)
* 💰 Total Sales
* 📦 Total Quantity Sold
* ⏳ Product Lifespan (months)
* 📅 Last Sale Date

---

### 4️⃣ Key Performance Indicators (KPIs)

* 📊 **Recency** → Months since last sale
* 💵 **Average Order Revenue (AOR)**
* 📈 **Average Monthly Revenue**
* 🏷️ **Average Selling Price**

---

## 🧠 SQL Logic Breakdown

### 🔹 Step 1: Base Query

* Joins `fact_sales` with `dim_products`
* Extracts essential transactional + product attributes
* Filters valid sales records

---

### 🔹 Step 2: Product Aggregation

* Groups data at the product level
* Calculates totals and averages
* Computes lifespan and last sale date

---

### 🔹 Step 3: Final Transformation

* Segments products based on revenue
* Calculates KPIs:

  * Recency
  * AOR
  * Monthly revenue

---

## 🧾 SQL Implementation

```sql
-- Create View: gold.report_products

IF OBJECT_ID('gold.report_products', 'V') IS NOT NULL
    DROP VIEW gold.report_products;
GO

CREATE VIEW gold.report_products AS

WITH base_query AS (
    SELECT
        f.order_number,
        f.order_date,
        f.customer_key,
        f.sales_amount,
        f.quantity,
        p.product_key,
        p.product_name,
        p.category,
        p.subcategory,
        p.cost
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_products p
        ON f.product_key = p.product_key
    WHERE order_date IS NOT NULL
),

product_aggregations AS (
    SELECT
        product_key,
        product_name,
        category,
        subcategory,
        cost,
        DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan,
        MAX(order_date) AS last_sale_date,
        COUNT(DISTINCT order_number) AS total_orders,
        COUNT(DISTINCT customer_key) AS total_customers,
        SUM(sales_amount) AS total_sales,
        SUM(quantity) AS total_quantity,
        ROUND(AVG(CAST(sales_amount AS FLOAT) / NULLIF(quantity, 0)), 1) AS avg_selling_price
    FROM base_query
    GROUP BY
        product_key,
        product_name,
        category,
        subcategory,
        cost
)

SELECT 
    product_key,
    product_name,
    category,
    subcategory,
    cost,
    last_sale_date,
    DATEDIFF(MONTH, last_sale_date, GETDATE()) AS recency_in_months,

    CASE
        WHEN total_sales > 50000 THEN 'High-Performer'
        WHEN total_sales >= 10000 THEN 'Mid-Range'
        ELSE 'Low-Performer'
    END AS product_segment,

    lifespan,
    total_orders,
    total_sales,
    total_quantity,
    total_customers,
    avg_selling_price,

    -- Average Order Revenue (AOR)
    CASE 
        WHEN total_orders = 0 THEN 0
        ELSE total_sales / total_orders
    END AS avg_order_revenue,

    -- Average Monthly Revenue
    CASE
        WHEN lifespan = 0 THEN total_sales
        ELSE total_sales / lifespan
    END AS avg_monthly_revenue

FROM product_aggregations;
```
<img width="1007" height="640" alt="image" src="https://github.com/user-attachments/assets/913e1800-b942-430a-8700-0aeec79b81ff" />

---

## 🚀 Use Cases

* 📊 Product performance dashboards (Power BI / Tableau)
* 📦 Inventory optimization
* 💰 Pricing strategy analysis
* 🎯 Marketing & promotion targeting
* 🛒 Product portfolio evaluation

---

## 🧩 Future Enhancements

* Add **profit margin analysis (Revenue - Cost)**
* Introduce **seasonality trends**
* Build **product recommendation insights**
* Integrate **real-time sales tracking**

---




