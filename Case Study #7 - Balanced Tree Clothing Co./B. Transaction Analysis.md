# Balanced Tree Clothing Co. 🌲 - Transaction Analysis - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./README.md).

--- 

## Transaction Analysis

1. How many unique transactions were there?
2. What is the average unique products purchased in each transaction?
3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?
4. What is the average discount value per transaction?
5. What is the percentage split of all transactions for members vs non-members?
6. What is the average revenue for member transactions and non-member transactions?

---

## 1. How many unique transactions were there?

* Use **COUNT** and **DISTINCT** to calculate the amount of unique transactions.

```sql
SELECT
  COUNT(DISTINCT txn_id) AS unique_transactions
FROM balanced_tree.sales;
```

### Answer

| unique_transactions |
| ------------------- |
| 2500                |

---

## 2. What is the average unique products purchased in each transaction?

* In `cte`, calculate the total unique products per transaction.
* In the outer query, calculate the average.

```sql
WITH cte AS (
  SELECT
    COUNT(prod_id) AS products_per_transaction
  FROM balanced_tree.sales
  GROUP BY txn_id
)

SELECT
  ROUND(AVG(products_per_transaction),2) AS average_products
FROM cte;
```

### Answer

| average_products |
| ---------------- |
| 6.04             |

---

## 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?

* I will assume that "revenue per transaction" means values after the discounts.
* In `revenue_after_discount`, calculate the revenue for each transaction.
* In the outer query, calculate the percentiles using ** percentile_cont**.

```sql
WITH revenue_after_discount AS (
  SELECT
    ROUND(SUM(qty*price*(100-discount)::numeric/100),2) AS revenue
  FROM balanced_tree.sales
  GROUP BY txn_id
)

SELECT 
  ROUND(percentile_cont(0.25) WITHIN GROUP (ORDER BY revenue)::numeric,2) AS percentile_25,
  ROUND(percentile_cont(0.50) WITHIN GROUP (ORDER BY revenue)::numeric,2) AS median,
  ROUND(percentile_cont(0.75) WITHIN GROUP (ORDER BY revenue)::numeric,2) AS percentile_75
FROM revenue_after_discount;
```

### Answer

| percentile_25 | median | percentile_75 |
| ------------- | ------ | ------------- |
| 326.41        | 441.23 | 572.76        |

---

## 4. What is the average discount value per transaction?

* In `discount_per_transaction`, calculate the total discount per transaction.
* In the outer query, calculate the average.

```sql
WITH discount_per_transaction AS (
  SELECT
    SUM(
      qty*price*discount::numeric/100
    ) AS discount
  FROM balanced_tree.sales
  GROUP BY txn_id
)

SELECT
  ROUND(AVG(discount),2) AS avg_discount
FROM discount_per_transaction;
```

### Answer

| avg_discount |
| ------------ |
| 62.49        |

---

## 5. What is the percentage split of all transactions for members vs non-members?

* Calculate the percentage - divide transactions by member (using **GROUP BY**) by total transactions `SELECT COUNT(DISTINCT txn_id)` and multiply by 100.

```sql
SELECT
  member, 
  ROUND(
    COUNT(DISTINCT txn_id)::numeric/(SELECT COUNT(DISTINCT txn_id) FROM balanced_tree.sales)*100
  ,2) AS transaction_percentage
FROM balanced_tree.sales
GROUP BY member;
```

### Answer

| member | transaction_percentage |
| ------ | ---------------------- |
| false  | 39.80                  |
| true   | 60.20                  |

---

## 6. What is the average revenue for member transactions and non-member transactions?

* Similarly to a previous question, I will assume that "revenue" means values after the discounts.
* In `total_revenue_transaction`, calculate the total revenue per transaction.
* In the outer query, group the revenue by member.

```sql
WITH total_revenue_transaction AS (
  SELECT
    member, 
    SUM(qty*price*(100-discount)::numeric/100) AS revenue
  FROM balanced_tree.sales
  GROUP BY txn_id,member
)

SELECT
  member,
  ROUND(AVG(revenue),2) AS avg_revenue
FROM total_revenue_transaction
GROUP BY member;
```

### Answer

| member | avg_revenue |
| ------ | ----------- |
| false  | 452.01      |
| true   | 454.14      |

---

Check out the solutions for the next section [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./C.%20Product%20Analysis.md)
