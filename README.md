# E-Commerce Data Analysis Project

## 📊 Overview
This project aims to derive insights from a comprehensive e-commerce dataset using SQL. The analysis includes identifying trends in sales, customer behaviors, product performance, and seller activity. The queries were designed to answer key business questions that could support data-driven decision-making for growth and customer satisfaction.

---

## 🧠 Objectives
- Detect duplicate customer entries.
- Identify top-performing product categories and states generating the highest revenue.
- Analyze total sales and revenue distribution by city and category.
- Discover product performance through customer reviews and identify the best-rated items.
- Highlight seller efficiency and contributions.
- Uncover ordering trends based on time (monthly) and customer activity.

---

## 🗂️ Data Sources (CSV Files). They can be gotten here [Kaggle Link](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- **Customers**: Includes details like customer_id,	customer_unique_id,	zip_code, city	and states.
- **Orders**: Contains information about orders including order_id,	customer_id, order_status, purchase_time, approved_time, delivered_carrier_date, delivered_cus_date	and est_delivery_date.
- **Order_items**: Links orders to products, and includes columns like order_id, product_id, seller_id,	price, freight_value, order_date.
- **Order_payments**: Links orders to products, and includes order_id, payment_type, payment_value.
- **Products**: Contains product_id and product_category_name column.
- **Product_category**: Maps product category names to English. Columns: product_category_name and product_category_eng.
- **Sellers**: Contains seller_id, zip_code, city, states.
- **Location**: Gives more details, zip_code, city, states.
- **Order_review**: Includes review_id, order_id, review_score, creation_date, answer_date.

---

## 🧹 Data Cleaning
Before diving into the analysis, I performed several data cleaning steps to ensure consistency and improve readability:
- Timestamp Conversion: Several columns contained hashtag-style timestamp formats (e.g., ######). I converted these into standard YYYY-MM-DD datetime formats, enabling easier filtering and time-based aggregation.
- Column Name Refinement: Some column names were either unclear or poorly formatted. I renamed them to follow consistent, descriptive naming conventions. For example, columns like product_category_name_english were renamed to product_category_eng for clarity.
- Column Dropping: I removed columns that were irrelevant to the scope of this analysis or had too many null values, such as the ones in the location table: latitude and longitude.

---

## 🧪 SQL Queries and Insights

### 1. 🕵️‍♂️ Checking for Duplicate Customers
This query identifies potential duplicate customers by checking how many times a `customer_id` appears.
```sql
-- Checking for duplicates
WITH duplicates AS (
	SELECT customer_id,
	city,
	states,
	ROW_NUMBER() OVER(PARTITION BY customer_id) AS frequency
	FROM customers
)
SELECT c.customer_id,
	c.city,
	c.states
FROM customers c
JOIN duplicates d ON c.customer_id = d.customer_id
WHERE d.frequency > 1;
```
**📌 Interpretation:** Duplicates may hint at system issues or multiple accounts per user. This can impact personalized marketing strategies.

---

### 2. 🌆 Total Sales by City
```sql
-- Total sales by city
SELECT
	c.city,
	SUM(i.price) AS total_sales
FROM orders o
JOIN order_items i ON o.order_id = i.order_id
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON i.product_id = p.product_id
GROUP BY c.city
ORDER BY total_sales DESC;
```

---

### 3. 🛒 Total Sales by Product Category
```sql
SELECT
	cat.product_category_eng AS category,
	SUM(i.price) AS sum_per_category
FROM orders o
JOIN order_items i ON o.order_id = i.order_id
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON i.product_id = p.product_id
JOIN product_category cat ON p.product_category_name = cat.product_category_name
GROUP BY category
ORDER BY sum_per_category DESC;
```

---

### 4. 🔝 Category with the Highest and Lowest Sales
```sql
-- Category with the highest and lowest sales
WITH revenue_by_category AS (
    SELECT
        cat.product_category_eng AS category,
        SUM(i.price) AS total_revenue_per_category
    FROM orders o
    JOIN order_items i ON o.order_id = i.order_id
    JOIN customers c ON o.customer_id = c.customer_id
    JOIN products p ON i.product_id = p.product_id
    JOIN product_category cat ON p.product_category_name = cat.product_category_name
    GROUP BY cat.product_category_eng
),
min_category AS (
    SELECT *
    FROM revenue_by_category
    ORDER BY total_revenue_per_category ASC
    LIMIT 1
),
max_category AS (
    SELECT *
    FROM revenue_by_category
    ORDER BY total_revenue_per_category DESC
    LIMIT 1
)
SELECT * FROM min_category
UNION ALL
SELECT * FROM max_category;
```

---

### 5. 📆 Total Orders by Month
```sql
-- Total Orders by month
SELECT
    EXTRACT(YEAR FROM purchase_time) AS year,
    TO_CHAR(purchase_time, 'Month') AS month_name,
    COUNT(*) AS total_orders
FROM orders
GROUP BY 1, 2, EXTRACT(MONTH FROM purchase_time)
ORDER BY 3 DESC;
```

---

### 6. 🌟 Average Review Per Category
```sql
-- Average review per category
SELECT cat.product_category_eng,
	ROUND(AVG(r.review_score), 2) AS avg_review_score
FROM product_category cat
JOIN products p ON cat.product_category_name = p.product_category_name
JOIN order_items i ON p.product_id = i.product_id
JOIN order_review r ON i.order_id = r.order_id
GROUP BY 1
ORDER BY 2 DESC;
```

---

### 7. 🥇 Product with the Highest Review Score per Category
```sql
-- Product with the highest review score per category
WITH ranked AS (
	SELECT cat.product_category_eng AS product_category, 
	p.product_id AS product_id, 
	r.review_score AS review_score,
	ROW_NUMBER() OVER(PARTITION BY cat.product_category_eng ORDER BY r.review_score DESC) AS rank
	FROM product_category cat
	JOIN products p ON cat.product_category_name = p.product_category_name
    JOIN order_items i ON p.product_id = i.product_id
    JOIN order_review r ON i.order_id = r.order_id
)
SELECT ranked.product_category, ranked.product_id, ranked.review_score
FROM ranked
WHERE ranked.rank = 1;
```

---

### 8. 👨‍💼 Top Sellers
```sql
-- Top sellers
SELECT s.seller_id,
	COUNT(i.product_id) AS no_of_products
FROM sellers s
JOIN order_items i ON s.seller_id = i.seller_id
JOIN products p ON i.product_id = p.product_id
GROUP BY 1
ORDER BY 2 DESC;
```

---

### 9. 🏷️ Top Sellers by Category
```sql
-- Top sellers by category
SELECT s.seller_id,
	cat.product_category_eng,
	COUNT(i.product_id) AS no_of_products
FROM sellers s
JOIN order_items i ON s.seller_id = i.seller_id
JOIN products p ON i.product_id = p.product_id
JOIN product_category cat ON p.product_category_name = cat.product_category_name
GROUP BY 1, 2
ORDER BY 3 DESC;
```

---

### 10. 📦 Products Not Ordered in the Last 6 Months
```sql
-- Products which haven't been ordered in the last 6 months
WITH product_orders AS (
    SELECT
        p.product_id,
        cat.product_category_eng AS product_category,
        MAX(o.purchase_time) AS last_order_date
    FROM products p
    LEFT JOIN order_items i ON p.product_id = i.product_id
    LEFT JOIN orders o ON i.order_id = o.order_id
    JOIN product_category cat ON p.product_category_name = cat.product_category_name
    GROUP BY p.product_id, cat.product_category_eng
)

SELECT
    product_id,
    product_category,
    last_order_date
FROM product_orders
WHERE last_order_date IS NULL 
   OR last_order_date < NOW() - INTERVAL '6 months'
ORDER BY product_category, last_order_date;
```

---

## ✅ Conclusion
The SQL analysis provides a rich foundation for understanding customer behaviors, product popularity, sales distribution, and seller performance in an e-commerce environment. These insights can guide decision-making across sales, inventory, and customer relationship strategies.

---

## ✍️ Author
**Elijah Obisesan Timilehin**  
_Self-taught Data Scientist & Physics Undergraduate_  
🔗 [LinkedIn](https://www.linkedin.com/in/teoso) | 🔗 [GitHub](https://github.com/etimexo)

---
