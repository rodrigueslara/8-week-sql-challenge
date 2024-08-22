# Data Bank 💲 - Customer Transactions - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%234%20-%20Data%20Bank/README.md).

---

## Customer Transactions

1. What is the unique count and total amount for each transaction type?
2. What is the average total historical deposit counts and amounts for all customers?
3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
4. What is the closing balance for each customer at the end of the month?
5. What is the percentage of customers who increase their closing balance by more than 5%?

---

## 1. What is the unique count and total amount for each transaction type?

* Use **GROUP BY**, **COUNT** and **SUM**.
  
```sql
SELECT
  txn_type,
  COUNT(txn_type) AS count_t,
  SUM(txn_amount) AS sum_t
FROM data_bank.customer_transactions
GROUP BY txn_type;
```

### Answer

| txn_type   | count_t | sum_t   |
| ---------- | ------- | ------- |
| purchase   | 1617    | 806537  |
| deposit    | 2671    | 1359168 |
| withdrawal | 1580    | 793003  |

---

## 2. What is the average total historical deposit counts and amounts for all customers?

* In `customer_cte`, calculate wanted values for each customer.
* In the outer query, calculate the average.

```sql
WITH customer_cte AS (
  SELECT
    COUNT(customer_id) AS count_t,
    SUM(txn_amount) AS sum_t
  FROM data_bank.customer_transactions
  WHERE txn_type = 'deposit'
  GROUP BY customer_id
)

SELECT
  ROUND(AVG(count_t),2) AS avg_count,
  ROUND(AVG(sum_t),2) AS avg_sum
FROM customer_cte;
```

### Answer

| avg_count | avg_sum |
| --------- | ------- |
| 5.34      | 2718.34 |

---

## 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

* In `counter_cte`, create 3 new columns: one for counting deposits, another for purchases, and the last one for withdrawals.
* Use **TO_CHAR** to get the month of transaction.
* In the outer query, use **WHERE** to get only the rows that we want (look question), and use **COUNT** to calculate the number of rows.

```sql
WITH counter_cte AS (
  SELECT 
    customer_id,
    SUM(
      CASE
        WHEN txn_type = 'deposit' THEN 1
        ELSE 0
      END
    ) AS deposit_count,
    SUM(
      CASE
        WHEN txn_type = 'withdrawal' THEN 1
        ELSE 0
      END
    ) AS withdrawal_count,
    SUM(
      CASE
        WHEN txn_type = 'purchase' THEN 1
        ELSE 0
        END
    ) AS purchase_count,
    TO_CHAR(txn_date, 'Month') AS month_t,
    TO_CHAR(txn_date, 'mm') AS month_n
  FROM data_bank.customer_transactions
  GROUP BY customer_id, month_t, month_n
)

SELECT
  month_t,
  COUNT(*)
FROM counter_cte
WHERE deposit_count > 1 AND (withdrawal_count = 1 OR purchase_count = 1)
GROUP BY month_t,month_n
ORDER BY month_n;
```

### Answer 

| month_t   | count |
| --------- | ----- |
| January   | 115   |
| February  | 108   |
| March     | 113   |
| April     | 50    |

---

## 4. What is the closing balance for each customer at the end of the month?

* In `transaction_cte`, use **SUM** and **CASE** to create a new column with the total net balance per customer, per month (**GROUP BY**).
* In the outer query, to calculate the closing balance for each month use **SUM OVER**.
  
```sql
WITH transaction_cte AS (
  SELECT
    customer_id,
    TO_CHAR(txn_date,'Month') AS month_t,
    TO_CHAR(txn_date,'mm') AS month_n,
    SUM(
      CASE
        WHEN txn_type = 'deposit' THEN txn_amount
        ELSE -txn_amount
      END
    ) AS transaction_value
  FROM data_bank.customer_transactions
  GROUP BY customer_id,month_t, month_n
)

SELECT
  customer_id,
  month_t,
  SUM(transaction_value) OVER(PARTITION BY customer_id ORDER BY month_n) AS final_balance_per_month
FROM transaction_cte
ORDER BY customer_id, month_n;
```

### Answer

| customer_id | month_t   | final_balance_per_month |
| ----------- | --------- | ----------------------- |
| 1           | January   | 312                     |
| 1           | March     | -640                    |
| 2           | January   | 549                     |
| 2           | March     | 610                     |
| 3           | January   | 144                     |
| 3           | February  | -821                    |
| 3           | March     | -1222                   |
| 3           | April     | -729                    |
| 4           | January   | 848                     |
| 4           | March     | 655                     |
| 5           | January   | 954                     |
| 5           | March     | -1923                   |
| 5           | April     | -2413                   |
| 6           | January   | 733                     |
| 6           | February  | -52                     |
| 6           | March     | 340                     |
| 7           | January   | 964                     |
| 7           | February  | 3173                    |
| 7           | March     | 2533                    |
| 7           | April     | 2623                    |

Since there are a lot of rows, I'll only show the balances for the first 7 customers.

---

## 5. What is the percentage of customers who increase their closing balance by more than 5%?

* We will calculate the percentage of customers who increased their closing balance (last month) by more than 5% (first month).
* We'll use the tables from the previous exercise (and add month_n to `final_balance`) to answer this question:

```sql
WITH transaction_cte AS (
  SELECT
    customer_id,
    TO_CHAR(txn_date,'Month') AS month_t,
    TO_CHAR(txn_date,'mm') AS month_n,
    SUM(
      CASE
        WHEN txn_type = 'deposit' THEN txn_amount
        ELSE -txn_amount
      END
    ) AS transaction_value
  FROM data_bank.customer_transactions
  GROUP BY customer_id,month_t, month_n
),

final_balance AS (
  SELECT
    customer_id,
    month_t,
    month_n,
    SUM(transaction_value) OVER(PARTITION BY customer_id ORDER BY month_n) AS final_balance_per_month
  FROM transaction_cte
),
```

And create some new tables:

* `rank_cte` creates a new column to help identify the first and last month.
* `last_month` identifies the last month (note that for some customers this is ranked_month 2, while for others might be 3 or 4).
* `percentage_balance` calculates the percentage increase/decrease.
  * Use **WHERE** to get only 2 rows for each customer: for the first and last month.
  * Use **LEAD** to help calculate the balance, and then calculate the percentage.
* In the outer query, use **WHERE** since we are only interested in a percentage increase of more than 5%.
* Use `SELECT COUNT(DISTINCT customer_id) FROM data_bank.customer_transactions` to calculate the total number of customers, **COUNT(*)** to calculate the number of customers who increased their closing balance by more than 5% and calculate the percentage.

```sql
rank_cte AS (
  SELECT
    *,
    RANK() OVER(PARTITION BY customer_id ORDER BY month_n) AS ranked_month
  FROM final_balance
),

last_month AS (
  SELECT
    customer_id,
    MAX(ranked_month) AS last_rank
  FROM rank_cte
  GROUP BY customer_id
),

percentage_balance AS(
  SELECT
    customer_id,
    (LEAD(final_balance_per_month) OVER(PARTITION BY customer_id ORDER BY month_n) - final_balance_per_month)
    * 100::numeric
    /final_balance_per_month
    AS percentage_increase
  FROM rank_cte
  JOIN last_month USING(customer_id)
  WHERE ranked_month = 1 OR ranked_month = last_rank
)

SELECT
  ROUND(
    COUNT(*)::numeric/
    (SELECT COUNT(DISTINCT customer_id) FROM data_bank.customer_transactions) * 100
  ,2) AS percentage_customers
FROM percentage_balance
WHERE percentage_increase > 5;
```

### Answer

| percentage_customers |
| -------------------- |
| 44.20                |

---
