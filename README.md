# 🍽️ Food Delivery Analysis with SQL

This project analyzes customer ordering behavior from a restaurant dataset. The goal is to extract insights to support marketing, growth, and customer retention strategies using SQL.

---

## 🏆 Top 3 Outlets by Cuisine Type

We identified the three most popular restaurants for each cuisine based on order count — without using SQL’s `LIMIT` or `TOP`.

```sql
WITH CTE AS (
  SELECT cuisine, restaurant_Id, COUNT(*) AS No_of_Orders
  FROM orders
  GROUP BY cuisine, restaurant_Id
)
SELECT * FROM (
  SELECT *,
    ROW_NUMBER() OVER (PARTITION BY cuisine ORDER BY No_of_Orders DESC) AS rn
  FROM CTE
) a 
WHERE rn <= 3;
```

---

## 📅 Daily New Customer Count

We tracked how many new customers were acquired each day from the restaurant’s launch date.

```sql
WITH ASP AS (
  SELECT customer_code, CAST(MIN(placed_at) AS DATE) AS first_order_date
  FROM orders
  GROUP BY customer_code
)
SELECT first_order_date, COUNT(*) AS No_of_New_Customers
FROM ASP
GROUP BY first_order_date 
ORDER BY No_of_New_Customers DESC;
```

---

## 📈 Top 5 Days with Highest Customer acquisitions

These are the five days with the most new customer acquisitions.

```sql
WITH ASP AS (
  SELECT customer_code, CAST(MIN(placed_at) AS DATE) AS first_order_date
  FROM orders
  GROUP BY customer_code
),
NewCustomersPerDay AS (
  SELECT first_order_date, COUNT(*) AS No_of_New_Customers
  FROM ASP
  GROUP BY first_order_date
)
SELECT TOP 5 first_order_date, No_of_New_Customers
FROM NewCustomersPerDay
ORDER BY No_of_New_Customers DESC;
```

---

## 🚫 Count all customers who were acquired in January 2025, placed exactly one order during that month, and did not place any orders in any other month

This query finds customers who placed only one order in January 2025 and didn’t return.

```sql
SELECT customer_code, COUNT(*) AS No_of_Orders
FROM orders
WHERE MONTH(placed_at) = 1 AND YEAR(placed_at) = 2025
  AND customer_code NOT IN (
    SELECT DISTINCT customer_code
    FROM orders
    WHERE NOT (MONTH(placed_at) = 1 AND YEAR(placed_at) = 2025)
  )
GROUP BY customer_code
HAVING COUNT(*) = 1;
```

---

## 💤 Dormant Users Acquired via Promo One Month Ago

We found users who haven’t ordered in the last 7 days but were acquired through promo a month ago.

```sql
WITH ASP AS (
  SELECT customer_code, MIN(placed_at) AS First_order_date, MAX(placed_at) AS latest_order_date
  FROM orders
  GROUP BY customer_code
)
SELECT ASP.*, orders.Promo_code_Name AS first_order_promo
FROM ASP
INNER JOIN orders ON ASP.customer_code = orders.customer_code AND ASP.First_order_date = orders.placed_at
WHERE latest_order_date < DATEADD(DAY, -7, GETDATE())
  AND First_order_date < DATEADD(MONTH, -1, GETDATE())
  AND orders.Promo_code_Name IS NOT NULL;
```

---

## 🧠 Trigger Strategy: Sending Personalized Messages to Customers After Their Third Order

Customers who are due for a personalized message after every third order.

```sql
WITH ASP AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_code ORDER BY placed_at) AS order_number
  FROM orders
)
SELECT *
FROM ASP
WHERE order_number % 3 = 0
  AND CAST(placed_at AS DATE) = CAST(GETDATE() AS DATE);
```

---

## 🎁 Promo-Only Loyal Customers

Customers who placed more than 1 order and all on promo.

```sql
SELECT customer_code, COUNT(*) AS No_of_orders, COUNT(Promo_code_Name) AS Promo_orders 
FROM orders
GROUP BY customer_code
HAVING COUNT(*) > 1 AND COUNT(*) = COUNT(Promo_code_Name);
```

---

## 📊Rate of Organic Customer Acquisition for January 2025

We calculated what percent of customers were acquired without a promo in Jan 2025.

```sql
WITH ASP AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_code ORDER BY placed_at) AS rn
  FROM orders
  WHERE MONTH(placed_at) = 1
)
SELECT COUNT(CASE WHEN rn = 1 AND Promo_code_Name IS NULL THEN customer_code END) * 100.0 / COUNT(DISTINCT customer_code)
FROM ASP;
```

---

## 📂 Analysis Categories

### 📈 Cumulative Metrics
We gathered total values over time, like total customers acquired or orders placed — useful for tracking long-term growth.

### 🎯 Performance Benchmarking
We compared restaurants and promotions to find out who or what performed the best — like finding the MVPs of your food delivery app.

### 🧮 Proportional Contribution Analysis
We looked at which cuisine types or customer segments contributed most to total orders — like seeing what slice of the pie each group owns.

### 🔍 Customer Segmentation
We grouped customers by behavior — like frequent users, one-timers, or promo-only users — to tailor strategies for each group.

### 📦 Product Segmentation
We identified which restaurants or cuisines performed best, helping prioritize what to promote.

#### 📌 Business Impact (Subsection of Product Segmentation)
This insight helps decide where the business should invest — like boosting top-performing cuisines or improving underperforming ones.

---

🛠️ **Tools Used:** SQL 

📁 **Dataset:** `orders` table containing customer, order, promo, and restaurant information.

---

✅ **Outcome:** This SQL project revealed key patterns in customer behavior and order trends, providing actionable insights to marketing, product, and growth teams.
