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
SELECT store_id, SUM(units_sold) AS total_units_sold
FROM sales
GROUP BY store_id;
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
    (COUNT(CASE WHEN claim_status = 'Warranty Void' THEN 1 END) * 100.0 / COUNT(*)) AS void_percentage
FROM warranty_claims;
```

### 6. Identify which store had the highest total units sold in the last year.
```sql
SELECT store_id, SUM(units_sold) AS total_units_sold
FROM sales
WHERE sale_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
GROUP BY store_id
ORDER BY total_units_sold DESC
LIMIT 1;
```

### 7. Count the number of unique products sold in the last year.
```sql
SELECT COUNT(DISTINCT product_id) AS unique_products_sold
FROM sales
WHERE sale_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR);
```

### 8. Find the average price of products in each category.
```sql
SELECT category, AVG(price) AS avg_price
FROM products
GROUP BY category;
```

### 9. How many warranty claims were filed in 2020?
```sql
SELECT COUNT(*) AS total_claims
FROM warranty_claims
WHERE YEAR(claim_date) = 2020;
```

### 10. For each store, identify the best-selling day based on the highest quantity sold.
```sql
SELECT store_id, sale_date, SUM(units_sold) AS total_units_sold
FROM sales
GROUP BY store_id, sale_date
ORDER BY total_units_sold DESC;
```

## 🔵 Medium to Hard Questions (5 Questions)

### 11. Identify the least selling product in each country for each year based on total units sold.
```sql
WITH ProductSales AS (
    SELECT country, product_id, YEAR(sale_date) AS year, SUM(units_sold) AS total_units_sold
    FROM sales
    JOIN stores ON sales.store_id = stores.store_id
    GROUP BY country, product_id, YEAR(sale_date)
)
SELECT country, year, product_id, total_units_sold
FROM (
    SELECT *, RANK() OVER (PARTITION BY country, year ORDER BY total_units_sold ASC) AS rnk
    FROM ProductSales
) t
WHERE rnk = 1;
```

### 12. Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
SELECT COUNT(*) AS claims_within_180_days
FROM warranty_claims wc
JOIN sales s ON wc.product_id = s.product_id
WHERE DATEDIFF(wc.claim_date, s.sale_date) <= 180;
```

### 13. Determine how many warranty claims were filed for products launched in the last two years.
```sql
SELECT COUNT(*) AS claims_for_recent_products
FROM warranty_claims wc
JOIN products p ON wc.product_id = p.product_id
WHERE p.launch_date >= DATE_SUB(CURDATE(), INTERVAL 2 YEAR);
```

### 14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```sql
SELECT YEAR(sale_date) AS year, MONTH(sale_date) AS month, SUM(units_sold) AS total_units
FROM sales
JOIN stores ON sales.store_id = stores.store_id
WHERE country = 'USA' AND sale_date >= DATE_SUB(CURDATE(), INTERVAL 3 YEAR)
GROUP BY YEAR(sale_date), MONTH(sale_date)
HAVING total_units > 5000;
```

### 15. Identify the product category with the most warranty claims filed in the last two years.
```sql
SELECT p.category, COUNT(*) AS total_claims
FROM warranty_claims wc
JOIN products p ON wc.product_id = p.product_id
WHERE wc.claim_date >= DATE_SUB(CURDATE(), INTERVAL 2 YEAR)
GROUP BY p.category
ORDER BY total_claims DESC
LIMIT 1;
```

## 🔴 Complex Questions (5 Questions)

### 16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
SELECT country, 
       (COUNT(wc.claim_id) * 100.0 / COUNT(s.sale_id)) AS claim_percentage
FROM sales s
LEFT JOIN warranty_claims wc ON s.product_id = wc.product_id
JOIN stores st ON s.store_id = st.store_id
GROUP BY country;
```

### 17. Analyze the year-by-year growth ratio for each store.
```sql
WITH YearlySales AS (
    SELECT store_id, YEAR(sale_date) AS year, SUM(units_sold) AS total_units_sold
    FROM sales
    GROUP BY store_id, YEAR(sale_date)
)
SELECT store_id, year, total_units_sold,
       LAG(total_units_sold) OVER (PARTITION BY store_id ORDER BY year) AS previous_year_sales,
       (total_units_sold - LAG(total_units_sold) OVER (PARTITION BY store_id ORDER BY year)) * 100.0
        / LAG(total_units_sold) OVER (PARTITION BY store_id ORDER BY year) AS growth_percentage
FROM YearlySales;
```

### 18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```sql
SELECT price_range, 
       CORR(price, claim_count) AS correlation
FROM (
    SELECT p.product_id, p.price, 
           COUNT(wc.claim_id) AS claim_count,
           CASE 
               WHEN p.price < 100 THEN 'Low'
               WHEN p.price BETWEEN 100 AND 500 THEN 'Medium'
               ELSE 'High'
           END AS price_range
    FROM products p
    LEFT JOIN warranty_claims wc ON p.product_id = wc.product_id
    WHERE wc.claim_date >= DATE_SUB(CURDATE(), INTERVAL 5 YEAR)
    GROUP BY p.product_id, p.price
) AS price_data
GROUP BY price_range;
```

### 19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```sql
SELECT store_id, 
       (COUNT(CASE WHEN claim_status = 'Paid Repaired' THEN 1 END) * 100.0
       / COUNT(*)) AS repair_percentage
FROM warranty_claims
GROUP BY store_id
ORDER BY repair_percentage DESC
LIMIT 1;
```

### 20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```sql
SELECT store_id, YEAR(sale_date) AS year, MONTH(sale_date) AS month,
       SUM(units_sold) OVER (PARTITION BY store_id ORDER BY YEAR(sale_date),
       MONTH(sale_date)) AS running_total
FROM sales
WHERE sale_date >= DATE_SUB(CURDATE(), INTERVAL 4 YEAR);
```

## Bonus Question

### Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```sql
WITH SalesPeriod AS (
    SELECT product_id, product_name, launch_date, sale_date, units_sold,
           CASE 
               WHEN sale_date <= DATE_ADD(launch_date, INTERVAL 6 MONTH) THEN '0-6 Months'
               WHEN sale_date <= DATE_ADD(launch_date, INTERVAL 12 MONTH) THEN '6-12 Months'
               WHEN sale_date <= DATE_ADD(launch_date, INTERVAL 18 MONTH) THEN '12-18 Months'
               ELSE 'Beyond 18 Months'
           END AS sales_period
    FROM sales
)
SELECT sales_period, SUM(units_sold) AS total_units_sold
FROM SalesPeriod
GROUP BY sales_period;
```

---
## Project Focus

### This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.
