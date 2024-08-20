# Foodie-Fi 🥑 - Challenge Payment Question

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/README.md).

---

The Foodie-Fi team wants you to create a new `payments table` for the year 2020 that includes amounts paid by each customer in the `subscriptions` table with the following requirements:

* monthly payments always occur on the same day of month as the original `start_date` of any monthly paid plan
* upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
* upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
* once a customer churns they will no longer make payments

Example outputs for this table might look like the following:

| customer_id | plan_id | plan_name     | payment_date             | amount | payment_order |
| ----------- | ------- | ------------- | ------------------------ | ------ | ------------- |
| 1           | 1       | basic monthly | 2020-08-08T00:00:00.000Z | 9.90   | 1             |
| 1           | 1       | basic monthly | 2020-09-08T00:00:00.000Z | 9.90   | 2             |
| 1           | 1       | basic monthly | 2020-10-08T00:00:00.000Z | 9.90   | 3             |
| 1           | 1       | basic monthly | 2020-11-08T00:00:00.000Z | 9.90   | 4             |
| 1           | 1       | basic monthly | 2020-12-08T00:00:00.000Z | 9.90   | 5             |
| 2           | 3       | pro annual    | 2020-09-27T00:00:00.000Z | 199.00 | 1             |
| 13          | 1       | basic monthly | 2020-12-22T00:00:00.000Z | 9.90   | 1             |
| 15          | 2       | pro monthly   | 2020-03-24T00:00:00.000Z | 19.90  | 1             |
| 15          | 2       | pro monthly   | 2020-04-24T00:00:00.000Z | 19.90  | 2             |
| 16          | 1       | basic monthly | 2020-06-07T00:00:00.000Z | 9.90   | 1             |
| 16          | 1       | basic monthly | 2020-07-07T00:00:00.000Z | 9.90   | 2             |
| 16          | 1       | basic monthly | 2020-08-07T00:00:00.000Z | 9.90   | 3             |
| 16          | 1       | basic monthly | 2020-09-07T00:00:00.000Z | 9.90   | 4             |
| 16          | 1       | basic monthly | 2020-10-07T00:00:00.000Z | 9.90   | 5             |
| 16          | 3       | pro annual    | 2020-10-21T00:00:00.000Z | 189.10 | 6             |
| 18          | 2       | pro monthly   | 2020-07-13T00:00:00.000Z | 19.90  | 1             |
| 18          | 2       | pro monthly   | 2020-08-13T00:00:00.000Z | 19.90  | 2             |
| 18          | 2       | pro monthly   | 2020-09-13T00:00:00.000Z | 19.90  | 3             |
| 18          | 2       | pro monthly   | 2020-10-13T00:00:00.000Z | 19.90  | 4             |
| 18          | 2       | pro monthly   | 2020-11-13T00:00:00.000Z | 19.90  | 5             |
| 18          | 2       | pro monthly   | 2020-12-13T00:00:00.000Z | 19.90  | 6             |
| 19          | 2       | pro monthly   | 2020-06-29T00:00:00.000Z | 19.90  | 1             |
| 19          | 2       | pro monthly   | 2020-07-29T00:00:00.000Z | 19.90  | 2             |
| 19          | 3       | pro annual    | 2020-08-29T00:00:00.000Z | 199.00 | 3             |

---

## Answer


* In `cte_series`, the column `payment_date` is created by using **GENERATE_SERIES**.
* In `cte_payment`, I use **LEAD** to prepare the creation of `amount`, use **WHERE** to remove all rows that **GENERATE_SERIES** created that don't interest us, and **RANK** to create `payment_order`.
* In the outer query, **SELECT** all needed columns and create `amount` with the help of previous columns created by **LEAD**:
  * According to the second point, "upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately" so when the upgrade is made before the month is over, the previous plan is basic, and the current is something other than basic (note that trial and churn are not in the table), the amount paid is the current plan's price minus the previous plan's.
   
```sql
WITH cte_series AS (
  SELECT
    *,
    GENERATE_SERIES(
      start_date,
      CASE
        WHEN plan_name = 'churn' THEN null
        ELSE '2020-12-31'::date
      END,
      CASE
        WHEN plan_id = 3 THEN INTERVAL '1 YEAR'
        ELSE INTERVAL '1 MONTH'
      END
    ) AS payment_date,
    COALESCE(LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date),'2020-12-31'::date) AS next_date
  FROM foodie_fi.plans
  JOIN foodie_fi.subscriptions USING(plan_id)
  WHERE EXTRACT(YEAR FROM start_date) = '2020' AND plan_name != 'trial'
),

cte_payment AS (
  SELECT
    *,
    RANK() OVER(PARTITION BY customer_id ORDER BY payment_date) AS payment_order,
    LEAD(payment_date,-1) OVER(PARTITION BY customer_id ORDER BY payment_date) AS previous_payment,
    LEAD(price,-1) OVER(PARTITION BY customer_id ORDER BY payment_date) AS previous_amount,
    LEAD(plan_name,-1) OVER(PARTITION BY customer_id ORDER BY payment_date) AS previous_plan
  FROM cte_series
  WHERE next_date>payment_date
)

SELECT
  customer_id,
  plan_id,
  plan_name,
  payment_date,
  CASE
    WHEN
      plan_name != 'basic monthly'
      AND previous_plan = 'basic monthly'
      AND AGE(payment_date,previous_payment) < INTERVAL '1 MONTH'
    THEN price - previous_amount
    ELSE price
  END AS amount,
  payment_order
FROM cte_payment
ORDER BY customer_id, payment_date
```
