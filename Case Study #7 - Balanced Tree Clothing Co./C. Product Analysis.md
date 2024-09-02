# Balanced Tree Clothing Co. 🌲 - Product Analysis - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./README.md).

--- 

## Product Analysis

1. What are the top 3 products by total revenue before discount?
2. What is the total quantity, revenue and discount for each segment?
3. What is the top selling product for each segment?
4. What is the total quantity, revenue and discount for each category?
5. What is the top selling product for each category?
6. What is the percentage split of revenue by product for each segment?
7. What is the percentage split of revenue by segment for each category?
8. What is the percentage split of total revenue by category?
9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

---

## 1. What are the top 3 products by total revenue before discount?

* Use **SUM** and **GROUP BY** to calculate the total revenue before discount by product.
* Use **ORDER BY** and **LIMIT** to get the top 3.

```sql
SELECT
  product_name,
  SUM(qty * s.price) AS revenue_before_discount
FROM balanced_tree.sales s
JOIN balanced_tree.product_details
  ON product_id = prod_id
GROUP BY product_name
ORDER BY revenue_before_discount DESC
LIMIT 3;
```

### Answer

| product_name                 | revenue_before_discount |
| ---------------------------- | ----------------------- |
| Blue Polo Shirt - Mens       | 217683                  |
| Grey Fashion Jacket - Womens | 209304                  |
| White Tee Shirt - Mens       | 152000                  |

---

## 2. What is the total quantity, revenue and discount for each segment?

* Use **SUM** and **GROUP BY** to calculate quantity, revenue and discounts.
  * Note: Revenue is total revenue.

```sql
SELECT
  segment_name,
  SUM(qty) AS quantity,
  ROUND(SUM(qty*(100-discount)*s.price::numeric /100),2) AS revenue,
  ROUND(SUM(qty*discount*s.price::numeric/100),2) AS discount
FROM balanced_tree.sales s
JOIN balanced_tree.product_details
  ON product_id = prod_id
GROUP BY segment_name;
```

### Answer

| segment_name | quantity | revenue   | discount |
| ------------ | -------- | --------- | -------- |
| Shirt        | 11265    | 356548.73 | 49594.27 |
| Jeans        | 11349    | 183006.03 | 25343.97 |
| Jacket       | 11385    | 322705.54 | 44277.46 |
| Socks        | 11217    | 270963.56 | 37013.44 |

---

## 3. What is the top selling product for each segment?

* In `rank_quantity`, use **DENSE_RANK** to identify products by the total quantity sold, partitioned by segment.
* In the outer query, identify the top-selling product by using **WHERE**.

```sql
WITH rank_quantity AS (
  SELECT
    product_name,
    segment_name,
    SUM(qty) AS sold,
    DENSE_RANK() OVER(PARTITION BY segment_name ORDER BY SUM(qty) DESC) AS rank_qty
  FROM balanced_tree.sales s
  JOIN balanced_tree.product_details
    ON product_id = prod_id
  GROUP BY product_name,segment_name
)

SELECT
  segment_name,
  product_name,
  sold
FROM rank_quantity
WHERE rank_qty = 1;
```

### Answer

| segment_name | product_name                  | sold |
| ------------ | ----------------------------- | ---- |
| Jacket       | Grey Fashion Jacket - Womens  | 3876 |
| Jeans        | Navy Oversized Jeans - Womens | 3856 |
| Shirt        | Blue Polo Shirt - Mens        | 3819 |
| Socks        | Navy Solid Socks - Mens       | 3792 |

---

## 4. What is the total quantity, revenue and discount for each category?

* Very similar to question 2.

```sql
SELECT
  category_name,
  SUM(qty) AS quantity,
  ROUND(SUM(qty*(100-discount)*s.price::numeric /100),2) AS revenue,
  ROUND(SUM(qty*discount*s.price::numeric/100),2) AS discount
FROM balanced_tree.sales s
JOIN balanced_tree.product_details
  ON product_id = prod_id
GROUP BY category_name;
```

### Answer

| category_name | quantity | revenue   | discount |
| ------------- | -------- | --------- | -------- |
| Mens          | 22482    | 627512.29 | 86607.71 |
| Womens        | 22734    | 505711.57 | 69621.43 |

---

## 5. What is the top selling product for each category?

* Very similar to question 3.

```sql
WITH rank_quantity AS (
  SELECT
    product_name,
    category_name,
    SUM(qty) AS sold,
    DENSE_RANK() OVER(PARTITION BY category_name ORDER BY SUM(qty) DESC) AS rank_qty
  FROM balanced_tree.sales s
  JOIN balanced_tree.product_details
    ON product_id = prod_id
  GROUP BY product_name,category_name
)

SELECT
  category_name,
  product_name,
  sold
FROM rank_quantity
WHERE rank_qty = 1;
```

### Answer

| category_name | product_name                 | sold |
| ------------- | ---------------------------- | ---- |
| Mens          | Blue Polo Shirt - Mens       | 3819 |
| Womens        | Grey Fashion Jacket - Womens | 3876 |

---

## 6. What is the percentage split of revenue by product for each segment?

* In `revenue_segment`, calculate the total revenue for each segment.
* In the outer query, calculate the percentage split, using the total revenue calculated above.

```sql
WITH revenue_segment AS (
  SELECT
    segment_name,
    SUM(qty*s.price*(100-discount)::numeric/100) AS segment_revenue
  FROM balanced_tree.sales s
  JOIN balanced_tree.product_details
    ON prod_id = product_id 
  GROUP BY segment_name
)

SELECT
  segment_name,
  product_name, 
  ROUND(
    SUM(qty*s.price*(100-discount)::numeric/100)::numeric/segment_revenue*100
  ,2) AS percentage_revenue
FROM balanced_tree.sales s
JOIN balanced_tree.product_details
  ON prod_id = product_id
JOIN revenue_segment USING(segment_name)
GROUP BY segment_name, product_name, segment_revenue
ORDER BY segment_name, percentage_revenue DESC;
```

### Answer

| segment_name | product_name                     | percentage_revenue |
| ------------ | -------------------------------- | ------------------ |
| Jacket       | Grey Fashion Jacket - Womens     | 56.99              |
| Jacket       | Khaki Suit Jacket - Womens       | 23.57              |
| Jacket       | Indigo Rain Jacket - Womens      | 19.44              |
| Jeans        | Black Straight Jeans - Womens    | 58.14              |
| Jeans        | Navy Oversized Jeans - Womens    | 24.04              |
| Jeans        | Cream Relaxed Jeans - Womens     | 17.82              |
| Shirt        | Blue Polo Shirt - Mens           | 53.53              |
| Shirt        | White Tee Shirt - Mens           | 37.48              |
| Shirt        | Teal Button Up Shirt - Mens      | 8.99               |
| Socks        | Navy Solid Socks - Mens          | 44.24              |
| Socks        | Pink Fluro Polkadot Socks - Mens | 35.57              |
| Socks        | White Striped Socks - Mens       | 20.20              |

---

## 7. What is the percentage split of revenue by segment for each category?

* Similar to the previous question.

```sql
WITH revenue_category AS (
  SELECT
    category_name,
    SUM(qty*s.price*(100-discount)::numeric/100) AS category_revenue
  FROM balanced_tree.sales s
  JOIN balanced_tree.product_details
    ON prod_id = product_id 
  GROUP BY category_name
)

SELECT
  category_name,
  segment_name, 
  ROUND(
    SUM(qty*s.price*(100-discount)::numeric/100)::numeric/category_revenue*100
  ,2) AS percentage_revenue
FROM balanced_tree.sales s
JOIN balanced_tree.product_details
  ON prod_id = product_id
JOIN revenue_category USING(category_name)
GROUP BY category_name, segment_name, category_revenue
ORDER BY category_name, percentage_revenue DESC;
```

### Answer

| category_name | segment_name | percentage_revenue |
| ------------- | ------------ | ------------------ |
| Mens          | Shirt        | 56.82              |
| Mens          | Socks        | 43.18              |
| Womens        | Jacket       | 63.81              |
| Womens        | Jeans        | 36.19              |

---

## 8. What is the percentage split of total revenue by category?

* The percentage is calculated by: taking the revenue of each category (using **SUM** and**GROUP BY**), dividing by the total revenue (calculated in the subquery - `SELECT SUM(qty*price*(100-discount)::numeric/100) FROM balanced_tree.sales`) and multiplying by 100.

```sql
SELECT
  category_name,
  ROUND(
    SUM(
      qty*s.price*(100-discount)::numeric/100
    )::numeric
    /(SELECT SUM(qty*price*(100-discount)::numeric/100) FROM balanced_tree.sales)
    *100
  ,2) AS percentage_revenue
FROM balanced_tree.sales s
JOIN balanced_tree.product_details
  ON prod_id = product_id
GROUP BY category_name;
```

### Answer

| category_name | percentage_revenue |
| ------------- | ------------------ |
| Mens          | 55.37              |
| Womens        | 44.63              |

---

## 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)

* Number of transactions where at least 1 quantity of a product was purchased - `COUNT(qty)`
* Divided by total number of transactions `SELECT COUNT(DISTINCT txn_id) FROM balanced_tree.sales`

```sql
SELECT
  product_name,
  ROUND(COUNT(qty)::numeric/(SELECT COUNT(DISTINCT txn_id) FROM balanced_tree.sales)*100,2) AS penetration
FROM balanced_tree.sales s
JOIN balanced_tree.product_details
  ON prod_id = product_id
GROUP BY product_name
ORDER BY penetration DESC;
```

### Answer

| product_name                     | penetration |
| -------------------------------- | ----------- |
| Navy Solid Socks - Mens          | 51.24       |
| Grey Fashion Jacket - Womens     | 51.00       |
| Navy Oversized Jeans - Womens    | 50.96       |
| White Tee Shirt - Mens           | 50.72       |
| Blue Polo Shirt - Mens           | 50.72       |
| Pink Fluro Polkadot Socks - Mens | 50.32       |
| Indigo Rain Jacket - Womens      | 50.00       |
| Khaki Suit Jacket - Womens       | 49.88       |
| Black Straight Jeans - Womens    | 49.84       |
| White Striped Socks - Mens       | 49.72       |
| Cream Relaxed Jeans - Womens     | 49.72       |
| Teal Button Up Shirt - Mens      | 49.68       |

---

## 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

* In `cte_txn`, create a column with products bought in each transaction using **STRING_AGG**.
* Additionally, use **COUNT** to get the number of distinct products bought in each transaction.
* In `combinations`, first use **WHERE** to select transactions with 3 distinct products and then use **COUNT** and **GROUP BY** to count the number of times each combination was bought.
* In the outer query, use **WHERE** to select the most common combinations.

```sql
WITH cte_txn AS (
  SELECT
    txn_id,
    STRING_AGG(product_name,', ' ORDER BY product_name) AS products,
    COUNT(product_name) AS quantity
  FROM balanced_tree.sales s
  JOIN balanced_tree.product_details
    ON prod_id = product_id
  GROUP BY txn_id
),

combinations AS (
  SELECT
    products,
    COUNT(products) AS combination_sold
  FROM cte_txn
  WHERE quantity = 3
  GROUP BY products
)

SELECT
  *
FROM combinations
WHERE combination_sold =
  (SELECT
    MAX(combination_sold)
  FROM combinations);
```

### Answer

| products                                                                                  | combination_sold |
| ----------------------------------------------------------------------------------------- | ---------------- |
| Black Straight Jeans - Womens, Navy Oversized Jeans - Womens, Teal Button Up Shirt - Mens | 3                |
| Black Straight Jeans - Womens, Navy Oversized Jeans - Womens, White Tee Shirt - Mens      | 3                |
| Blue Polo Shirt - Mens, Navy Oversized Jeans - Womens, Pink Fluro Polkadot Socks - Mens   | 3                |
| Cream Relaxed Jeans - Womens, Navy Oversized Jeans - Womens, White Tee Shirt - Mens       | 3                |
| Khaki Suit Jacket - Womens, Navy Oversized Jeans - Womens, White Striped Socks - Mens     | 3                |

---
