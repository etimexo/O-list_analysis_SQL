# E-Commerce Data Analysis Project
![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/banner.png)

## 📊 Overview
This project aims to derive insights from a comprehensive e-commerce dataset using SQL. The analysis includes identifying trends in sales, customer behaviors, product performance, and seller activity. The queries were designed to answer key business questions that could support data-driven decision-making for growth and customer satisfaction.

---

## 🧠 Objectives
- Detect duplicate customer entries.
- Identify top-performing product categories and states generating the highest revenue.
- Analyze total sales and revenue distribution by city and category.
- Discover product performance through customer reviews and identify the best-rated items.
- Highlight seller efficiency and contributions.

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

🗂️ Database Schema Overview
---
To understand the relationships between the various tables used in this project, I referred to the official schema diagram provided with the dataset. This schema outlines how entities such as customers, orders, order items, products, sellers, and reviews are interconnected through keys like order_id, product_id, seller_id, and customer_id.

📌 Schema Preview:

![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/schema.png)

This visual aid was especially useful during JOIN operations and while determining appropriate groupings for aggregation and analysis. It also helped in identifying the cardinality between tables (e.g., one-to-many relationships between customers and orders).

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
![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/duplicates_r.png)

**📌 Interpretation:** Duplicates may hint at system issues or multiple accounts per user. This can impact personalized marketing strategies. The result above shows that there weren't any duplicate customers.

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

![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/cities_r.png)

**📌 Interpretation:** This insight helps identify cities with high and low sales performance, enabling the business to tailor marketing strategies accordingly—such as running targeted awareness campaigns, offering special discounts, or implementing other localized promotional efforts.

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

![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/sales_by_cat_r.png)

**📌 Interpretation:** This highlights, in descending order, the product categories that generate the highest revenue. It provides valuable insight into which categories are driving sales the most, helping to inform inventory, marketing, and business strategy decisions.

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

![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/min_max_cat_r.png)

**📌 Interpretation:** This analysis identifies both the top-performing and underperforming product categories in terms of total revenue. Understanding these extremes helps businesses recognize which categories are thriving and which may require strategic improvements—such as promotional efforts, product reviews, or even reconsideration of inventory. It could mean boosting promotions for low-performing ones or doubling down on what’s already doing well. Either way, it’s a great way to stay on top of what’s working and what’s not.

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

![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/month_r.png)

**📌 Interpretation:** This breaks down the total number of orders placed each month across different years. It’s useful for spotting trends—like which months are busier and when things slow down. With this, businesses can plan ahead for high-demand periods and prep marketing or inventory strategies accordingly.

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

![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/cat_rev_r.png)

**📌 Interpretation:** This gives a look at how customers feel about each product category, on average. By seeing which categories get the highest ratings, we can tell what customers are loving the most. On the flip side, lower average scores might signal areas that need improvement—maybe in product quality, delivery, or even customer expectations.

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

![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/h_rev_r.png)

**📌 Interpretation:** This highlights the best-reviewed product in each category. It's super helpful for identifying customer favorites — those products that consistently get top ratings. Knowing which item shines the most in its category can guide restocking decisions, highlight strong performers, and even help recommend similar products to new customers.

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

![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/sellers_prod_r.png)


**📌 Interpretation:** This shows the sellers with the highest number of products sold across the platform. It's a great way to recognize the most active and high-performing sellers. These top sellers might be ideal partners for exclusive deals, promotions, or even to understand what they’re doing right in terms of product quality, pricing, or delivery.

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

![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/sellers_cat_r.png)

**📌 Interpretation:** This helps us see which sellers are dominating specific product categories. It’s useful for spotting specialists or niche sellers who are doing really well in certain segments. This kind of info can guide category-specific partnerships, promotions, or even onboarding strategies for new sellers.

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

![Schema](https://github.com/etimexo/O-list_analysis_SQL/blob/main/Images/6_month_r.png)

---

## ✅ Conclusion
The SQL analysis provides a rich foundation for understanding customer behaviors, product popularity, sales distribution, and seller performance in an e-commerce environment. These insights can guide decision-making across sales, inventory, and customer relationship strategies.

---

## ✍️ Author
**Elijah Obisesan Timilehin**  
_Self-taught Data Scientist & Physics Undergraduate_  
🔗 [LinkedIn](https://www.linkedin.com/in/teoso) | 🔗 [GitHub](https://github.com/etimexo)

---
