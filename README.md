# 🍏 Analyzing Apple Sales with SQL  

---  

**Apple Inc.** is one of the most valuable companies in the world, with a vast network of retail stores and millions of products sold worldwide. Understanding sales trends, warranty claims, and product performance is crucial for optimizing inventory, enhancing customer satisfaction, and making data-driven business decisions. 📊💻 

<img width="693" alt="Screenshot 2025-03-19 at 1 40 38 PM" src="https://github.com/user-attachments/assets/e8d8a871-43e4-40ef-a3f9-7543d00d7c9e" />

---
## Entity Relationship Diagram (ERD)

![erd](https://github.com/user-attachments/assets/47db4379-4365-4251-98d3-d81eac5c53c1)

---
## 📊 Project Overview  

This project analyzes **Apple sales data** using SQL to uncover valuable insights. We will explore key business metrics, including:  
✅ **Store performance by country**  
✅ **Sales trends over time**  
✅ **Warranty claim patterns**  
✅ **Product performance metrics**  

## 🔍 Key Objectives  

This project features **20 SQL queries**, categorized into three difficulty levels:  
🟢 **Easy to Medium** (10 Questions)  
🔵 **Medium to Hard** (5 Questions)  
🔴 **Complex Analysis** (5 Questions)  

Let's dive into SQL-driven Apple sales insights! 🍏📊  

---  

## 📌 20 SQL Practice Questions  

## 🟢 Easy to Medium Questions (10 Questions)

### 1. Find the number of stores in each country.
```sql
SELECT country, COUNT(DISTINCT store_id) AS store_count
FROM stores
GROUP BY country;
```

### 2. Calculate the total number of units sold by each store.
```sql
SELECT s.store_id, st.store_name, SUM(s.units_sold) AS total_units_sold
FROM sales s
JOIN
stores st
ON st.store_id = s.store_id
GROUP BY 1, 2
ORDER BY 3 DESC;
```

### 3. Identify how many sales occurred in December 2023.
```sql
SELECT COUNT(*) AS total_sales
FROM sales
WHERE sale_date BETWEEN '2023-12-01' AND '2023-12-31';
```

### 4. Determine how many stores have never had a warranty claim filed.
```sql
SELECT COUNT(*) AS stores_without_claims
FROM stores
WHERE store_id NOT IN (SELECT DISTINCT store_id FROM warranty_claims);
```

### 5. Calculate the percentage of warranty claims marked as "Warranty Void".
```sql
SELECT 
    ROUND(
        COUNT(claim_id)::numeric / (SELECT COUNT(*) FROM warranty)::numeric * 100, 
        2
    ) AS warranty_void_percentage
FROM warranty
WHERE repair_status = 'Warranty Void';
```

### 6. Identify which store had the highest total units sold in the last year.
```sql
SELECT s.store_id,
       st.store_name,
       SUM(s.units_sold) AS total_units_sold
FROM sales AS s
JOIN stores AS st ON s.store_id = st.store_id
WHERE s.sale_date >= (CURRENT_DATE - INTERVAL '1 year')
GROUP BY s.store_id, st.store_name
ORDER BY total_units_sold DESC
LIMIT 1;
```

### 7. Count the number of unique products sold in the last year.
```sql
SELECT COUNT(DISTINCT product_id) AS unique_products_sold
FROM sales
WHERE sale_date >= (CURRENT_DATE - INTERVAL '1 year');
```

### 8. Find the average price of products in each category.
```sql
SELECT 
    p.category_id,
    c.category_name,
    AVG(p.price) AS avg_price
FROM products AS p
JOIN category AS c
ON p.category_id = c.category_id
GROUP BY 1, 2
ORDER BY 3 DESC;
```

### 9. How many warranty claims were filed in 2020?
```sql
SELECT 
    COUNT(*) AS warranty_claim
FROM warranty
WHERE EXTRACT(YEAR FROM claim_date) = 2020;
```

### 10. For each store, identify the best-selling day based on the highest quantity sold.
```sql
SELECT * 
FROM (
    SELECT 
        store_id, 
        TO_CHAR(sale_date, 'Day') AS day_name, 
        SUM(quantity) AS total_unit_sold, 
        RANK() OVER (PARTITION BY store_id ORDER BY SUM(quantity) DESC) AS rank
    FROM sales
    GROUP BY 1, 2
) AS t1
WHERE rank = 1;
```

## 🔵 Medium to Hard Questions (5 Questions)

### 11. Identify the least selling product in each country for each year based on total units sold.
```sql
WITH product_rank AS (
    SELECT
        st.country,
        p.product_name,
        SUM(s.quantity) AS total_qty_sold,
        DENSE_RANK() OVER (PARTITION BY st.country ORDER BY SUM(s.quantity) DESC) AS rank
    FROM sales AS s
    JOIN stores AS st ON s.store_id = st.store_id
    JOIN products AS p ON s.product_id = p.product_id
    GROUP BY st.country, p.product_name
)
SELECT *
FROM product_rank
WHERE rank = 1;
```

### 12. Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
SELECT 
    COUNT(*) 
FROM warranty AS w
LEFT JOIN sales AS s 
ON s.sale_id = w.sale_id
WHERE 
    w.claim_date - s.sale_date <= 180;
```

### 13. Determine how many warranty claims were filed for products launched in the last two years.
```sql
SELECT 
    p.product_name,
    COUNT(w.claim_id) AS no_claim,
    COUNT(s.sale_id)
FROM warranty AS w
RIGHT JOIN sales AS s 
ON s.sale_id = w.sale_id
JOIN products AS p
ON p.product_id = s.product_id
WHERE p.launch_date >= CURRENT_DATE - INTERVAL '2 years'
GROUP BY 1
HAVING COUNT(w.claim_id) > 0;
```

### 14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```sql
SELECT 
    TO_CHAR(sale_date, 'MM-YYYY') AS month,
    SUM(s.quantity) AS total_unit_sold
FROM sales AS s
JOIN stores AS st
ON s.store_id = st.store_id
WHERE 
    st.country = 'USA'
    AND s.sale_date >= CURRENT_DATE - INTERVAL '3 year'
GROUP BY 1
HAVING SUM(s.quantity) > 5000;
```

### 15. Identify the product category with the most warranty claims filed in the last two years.
```sql
SELECT 
    c.category_name,
    COUNT(w.claim_id) AS total_claims
FROM warranty AS w
LEFT JOIN sales AS s
ON w.sale_id = s.sale_id
JOIN products AS p
ON p.product_id = s.product_id
JOIN category AS c
ON c.category_id = p.category_id
WHERE 
    w.claim_date >= CURRENT_DATE - INTERVAL '2 year'
GROUP BY 1;
```

## 🔴 Complex Questions (5 Questions)

### 16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
SELECT 
    country,
    total_unit_sold,
    total_claim,
    COALESCE(ROUND(total_claim::numeric /total_unit_sold::numeric * 100, 2), 0) AS risk
FROM (
    SELECT 
        st.country,
        SUM(s.quantity) AS total_unit_sold,
        COUNT(w.claim_id) AS total_claim
    FROM sales AS s
    JOIN stores AS st ON s.store_id = st.store_id
    LEFT JOIN warranty AS w ON w.sale_id = s.sale_id
    GROUP BY st.country
) t1
ORDER BY risk DESC;
```

### 17. Analyze the year-by-year growth ratio for each store.
```sql
WITH yearly_sales AS (
    SELECT 
        s.store_id,
        st.store_name,
        EXTRACT(YEAR FROM sale_date) AS year,
        SUM(s.quantity * p.price) AS total_sale
    FROM sales AS s
    JOIN products AS p ON s.product_id = p.product_id
    JOIN stores AS st ON st.store_id = s.store_id
    GROUP BY 1, 2, 3
    ORDER BY 2, 3
), 

growth_ratio AS (
    SELECT 
        store_name,
        year,
        LAG(total_sale, 1) OVER (PARTITION BY store_name ORDER BY year) AS last_year_sale,
        total_sale AS current_year_sale
    FROM yearly_sales
)

SELECT 
    store_name,
    year,
    last_year_sale,
    current_year_sale,
    ROUND(
        ((current_year_sale - last_year_sale)::numeric /last_year_sale::numeric) * 100, 2
    ) AS growth_ratio
FROM growth_ratio
WHERE
last_year_sale IS NOT NULL
AND
YEAR <> EXTRACT(YEAR FROM CURRENT_DATE)
ORDER BY store_name, year;
```

### 18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```sql
SELECT 
    CASE 
        WHEN p.price < 500 THEN 'Less Expensive Product'
        WHEN p.price BETWEEN 500 AND 1000 THEN 'Mid Range Product'
        ELSE 'Expensive Product'
    END AS price_segment, 
    COUNT(w.claim_id) AS total_claim
FROM warranty AS w
LEFT JOIN sales AS s 
    ON w.sale_id = s.sale_id
JOIN products AS p 
    ON p.product_id = s.product_id
WHERE claim_date >= CURRENT_DATE - INTERVAL '5 year'
GROUP BY 1;
```

### 19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```sql
WITH paid_repair AS (
    SELECT 
        s.store_id,
        COUNT(w.claim_id) AS paid_repaired
    FROM sales AS s
    RIGHT JOIN warranty AS w
    ON w.sale_id = s.sale_id
    WHERE w.repair_status = 'Paid Repaired'
    GROUP BY 1
),
total_repaired AS (
    SELECT 
        s.store_id,
        COUNT(w.claim_id) AS total_repaired
    FROM sales AS s
    RIGHT JOIN warranty AS w
    ON w.sale_id = s.sale_id
    GROUP BY 1
)
SELECT 
    tr.store_id,
    st.store_name,
    pr.paid_repaired,
    tr.total_repaired,
    ROUND(pr.paid_repaired::numeric /
         tr.total_repaired::numeric * 100, 2) AS percentage_paid_repaired
FROM paid_repair AS pr
JOIN total_repaired AS tr
ON pr.store_id = tr.store_id
JOIN stores AS st
ON tr.store_id = st.store_id;
```

### 20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```sql
WITH monthly_sales AS (
    SELECT 
        store_id,
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        SUM(p.price * s.quantity) AS total_revenue
    FROM sales AS s
    JOIN products AS p ON s.product_id = p.product_id
    WHERE sale_date >= DATE_TRUNC('month', CURRENT_DATE) - INTERVAL '4 years'
    GROUP BY 1, 2, 3
    ORDER BY 1, 2, 3
)

SELECT 
    store_id,
    month,
    year,
    total_revenue,
    SUM(total_revenue) OVER (
        PARTITION BY store_id 
        ORDER BY year, month
    ) AS running_total
FROM monthly_sales;
```

## Bonus Question

### Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```sql
SELECT 
    p.product_name,
    CASE 
        WHEN s.sale_date BETWEEN p.launch_date AND p.launch_date + INTERVAL '6 month' THEN '0-6 month'
        WHEN s.sale_date BETWEEN p.launch_date + INTERVAL '6 month' AND p.launch_date + INTERVAL '12 month' THEN '6-12'
        WHEN s.sale_date BETWEEN p.launch_date + INTERVAL '12 month' AND p.launch_date + INTERVAL '18 month' THEN '12-18'
        ELSE '18+'
    END AS plc,
    SUM(s.quantity) AS total_qty_sale
FROM sales AS s
JOIN products AS p 
ON s.product_id = p.product_id
GROUP BY 1, 2
ORDER BY 1, 3 DESC;
```

---
## Project Focus

### This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.
