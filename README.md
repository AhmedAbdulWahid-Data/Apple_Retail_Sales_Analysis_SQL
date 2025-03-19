# 🍏 Analyzing Apple Sales with SQL  

---  

**Apple Inc.** is one of the most valuable companies in the world, with a vast network of retail stores and millions of products sold worldwide. Understanding sales trends, warranty claims, and product performance is crucial for optimizing inventory, enhancing customer satisfaction, and making data-driven business decisions. 📊💻  

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

### 🟢 Easy to Medium Level  

#### 1️⃣ Find the number of stores in each country.  
```sql
SELECT country, COUNT(store_id) AS total_stores
FROM stores
GROUP BY country;
```

#### 2️⃣ Calculate the total number of units sold by each store.  
```sql
SELECT store_id, SUM(quantity) AS total_units_sold
FROM sales
GROUP BY store_id;
```

#### 3️⃣ Identify how many sales occurred in December 2023.  
```sql
SELECT COUNT(*) AS total_sales
FROM sales
WHERE sale_date BETWEEN '2023-12-01' AND '2023-12-31';
```

#### 4️⃣ Determine how many stores have never had a warranty claim filed.  
```sql
SELECT COUNT(DISTINCT store_id) AS stores_without_claims
FROM stores
WHERE store_id NOT IN (SELECT DISTINCT store_id FROM warranty_claims);
```

#### 5️⃣ Calculate the percentage of warranty claims marked as "Warranty Void".  
```sql
SELECT 
    (COUNT(*) FILTER (WHERE claim_status = 'Warranty Void') * 100.0 / COUNT(*)) AS warranty_void_percentage
FROM warranty_claims;
```

#### 6️⃣ Identify which store had the highest total units sold in the last year.  
```sql
SELECT store_id, SUM(quantity) AS total_units_sold
FROM sales
WHERE sale_date >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY store_id
ORDER BY total_units_sold DESC
LIMIT 1;
```

#### 7️⃣ Count the number of unique products sold in the last year.  
```sql
SELECT COUNT(DISTINCT product_id) AS unique_products_sold
FROM sales
WHERE sale_date >= CURRENT_DATE - INTERVAL '1 year';
```

#### 8️⃣ Find the average price of products in each category.  
```sql
SELECT category, AVG(price) AS avg_price
FROM products
GROUP BY category;
```

#### 9️⃣ How many warranty claims were filed in 2020?  
```sql
SELECT COUNT(*) AS claims_2020
FROM warranty_claims
WHERE claim_date BETWEEN '2020-01-01' AND '2020-12-31';
```

#### 🔟 For each store, identify the best-selling day based on highest quantity sold.  
```sql
SELECT store_id, sale_date, SUM(quantity) AS total_sold
FROM sales
GROUP BY store_id, sale_date
ORDER BY total_sold DESC;
```

---  

### 🔵 Medium to Hard Level  

#### 1️⃣ Identify the least selling product in each country for each year based on total units sold.  
```sql
WITH product_sales AS (
    SELECT country, YEAR(sale_date) AS year, product_id, SUM(quantity) AS total_units_sold
    FROM sales
    JOIN stores ON sales.store_id = stores.store_id
    GROUP BY country, year, product_id
)
SELECT * FROM (
    SELECT *, RANK() OVER (PARTITION BY country, year ORDER BY total_units_sold ASC) AS rnk
    FROM product_sales
) ranked
WHERE rnk = 1;
```

#### 2️⃣ Calculate how many warranty claims were filed within 180 days of a product sale.  
```sql
SELECT COUNT(*) AS claims_within_180_days
FROM warranty_claims wc
JOIN sales s ON wc.product_id = s.product_id
WHERE wc.claim_date <= s.sale_date + INTERVAL '180 days';
```

#### 3️⃣ Determine how many warranty claims were filed for products launched in the last two years.  
```sql
SELECT COUNT(*) AS recent_product_claims
FROM warranty_claims wc
JOIN products p ON wc.product_id = p.product_id
WHERE p.launch_date >= CURRENT_DATE - INTERVAL '2 years';
```

#### 4️⃣ List the months in the last three years where sales exceeded 5,000 units in the USA.  
```sql
SELECT DATE_TRUNC('month', sale_date) AS month, SUM(quantity) AS total_sold
FROM sales
JOIN stores ON sales.store_id = stores.store_id
WHERE stores.country = 'USA' AND sale_date >= CURRENT_DATE - INTERVAL '3 years'
GROUP BY month
HAVING SUM(quantity) > 5000;
```

#### 5️⃣ Identify the product category with the most warranty claims filed in the last two years.  
```sql
SELECT category, COUNT(*) AS claim_count
FROM warranty_claims wc
JOIN products p ON wc.product_id = p.product_id
WHERE wc.claim_date >= CURRENT_DATE - INTERVAL '2 years'
GROUP BY category
ORDER BY claim_count DESC
LIMIT 1;
```

---  

### 🔴 Complex Analysis  

#### 1️⃣ Determine the percentage chance of receiving warranty claims after each purchase for each country.  
```sql
SELECT country, 
    (COUNT(DISTINCT warranty_claim_id) * 100.0 / COUNT(DISTINCT sale_id)) AS claim_probability
FROM warranty_claims wc
JOIN sales s ON wc.product_id = s.product_id
JOIN stores st ON s.store_id = st.store_id
GROUP BY country;
```

#### 2️⃣ Analyze the year-by-year growth ratio for each store.  
```sql
SELECT store_id, YEAR(sale_date) AS year, 
    (SUM(quantity) - LAG(SUM(quantity)) OVER (PARTITION BY store_id ORDER BY YEAR(sale_date))) * 100.0 / LAG(SUM(quantity)) OVER (PARTITION BY store_id ORDER BY YEAR(sale_date)) AS growth_rate
FROM sales
GROUP BY store_id, year;
```

#### 3️⃣ Calculate the correlation between product price and warranty claims.  
```sql
SELECT CORR(p.price, COUNT(wc.warranty_claim_id)) AS correlation
FROM products p
JOIN warranty_claims wc ON p.product_id = wc.product_id;
```

#### 4️⃣ Identify the store with the highest percentage of "Paid Repaired" claims.  
```sql
SELECT store_id, (COUNT(*) FILTER (WHERE claim_status = 'Paid Repaired') * 100.0 / COUNT(*)) AS paid_repair_ratio
FROM warranty_claims
GROUP BY store_id
ORDER BY paid_repair_ratio DESC
LIMIT 1;
```

#### 5️⃣ Calculate the monthly running total of sales for each store.  
```sql
SELECT store_id, DATE_TRUNC('month', sale_date) AS month, SUM(quantity) OVER (PARTITION BY store_id ORDER BY DATE_TRUNC('month', sale_date)) AS running_total
FROM sales;
