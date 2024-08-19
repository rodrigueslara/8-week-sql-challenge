# Foodie-Fi :avocado: - Data Analysis Questions - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/README.md).

---

## Data Analysis Questions

1. How many customers has Foodie-Fi ever had?
2. What is the monthly distribution of `trial` plan `start_date` values for our dataset - use the start of the month as the group by value.
3. What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each `plan_name`.
4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
6. What is the number and percentage of customer plans after their initial free trial?
7. What is the customer count and percentage breakdown of all 5 `plan_name` values at `2020-12-31`?
8. How many customers have upgraded to an annual plan in 2020?
9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

---

## 1. How many customers has Foodie-Fi ever had?

* Use **COUNT** and **DISTINCT** to calculate the number of Foodie-Fi customers.

```sql
SELECT
  COUNT(DISTINCT customer_id) AS n_customers
FROM foodie_fi.subscriptions;
```

### Answer

| n_customers |
| ----------- |
| 1000        |

---

## 2. What is the monthly distribution of `trial` plan `start_date` values for our dataset - use the start of the month as the group by value.

* Use **WHERE** since we are only interested in the trial plan - `plan_id = '0'`.
* Use **TO_CHAR** with **'Month'** to get the month the trial plan started.
* Use **COUNT** and **GROUP BY** to calculate the the montly distribution.
* Note that I added another **TO_CHAR** to order the table by month.

```sql
SELECT
  TO_CHAR(start_date,'MM') AS n_month,
  TO_CHAR(start_date,'Month') AS trial_start,
  COUNT(start_date) AS distribution
FROM foodie_fi.subscriptions
WHERE plan_id = '0'
GROUP BY trial_start, n_month
ORDER BY n_month;
```

### Answer

| n_month | trial_start | distribution |
| ------- | ----------- | ------------ |
| 01      | January     | 88           |
| 02      | February    | 68           |
| 03      | March       | 94           |
| 04      | April       | 81           |
| 05      | May         | 88           |
| 06      | June        | 79           |
| 07      | July        | 89           |
| 08      | August      | 88           |
| 09      | September   | 87           |
| 10      | October     | 79           |
| 11      | November    | 75           |
| 12      | December    | 84           |

---

## 3. What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

* Use **JOIN** to get all the information needed.
* Use **WHERE** to get all events after 2020.
* Use **COUNT** and **GROUP BY** to calculate the total number of events and group it by trial.

```sql
SELECT
  plan_name,
  COUNT(plan_name) AS n_events
FROM foodie_fi.subscriptions
JOIN foodie_fi.plans USING(plan_id)
WHERE start_date >= '2021-01-01'
GROUP BY plan_name
ORDER BY n_events DESC;
```

### Answer

| plan_name     | n_events |
| ------------- | -------- |
| churn         | 71       |
| pro annual    | 63       |
| pro monthly   | 60       |
| basic monthly | 8        |

---

## 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

* Use **JOIN** to get all the information needed.
* Use **WHERE** since we are only interested in churned customers.
* **SELECT**:
  * **COUNT** to calculate the number of customers churned.
  * To calculate the percentage of customers churned, use **COUNT** to count all customers churned, divide by the number of distinct customers (that we know from a previous question is 1000), and multiply by 100. Use **ROUND** to round to 1 decimal place.

```sql
SELECT
  COUNT(customer_id) AS churned_customers,
  ROUND(
    100 * COUNT(*)::numeric/
    (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions)
  ,1) AS percentage_churn
FROM foodie_fi.subscriptions
JOIN foodie_fi.plans USING(plan_id)
WHERE plan_name = 'churn';
```

### Answer

| churned_customers | percentage_churn |
| ----------------- | ---------------- |
| 307               | 30.7             |

---

## 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

* In `cte_ranked`, use **JOIN** to get all the information needed.
* In `cte_ranked`, use **RANK** and **PARTITION** to identify the number of events for each customer.
* In the outer query, use **WHERE**: we want rows where `plan_name` is churn and where it's the second event for that customer - `ranked_plan = 2`.
* Use **COUNT** to calculate the number of customers that have churned straight after their initial free trial.
* To calculate the percentage, use **COUNT** to count all customers that have churned straight after their initial free trial, divide by 1000 - the number of distinct customers, and multiply by 100. Use **ROUND** to round to the nearest whole number.
  * Note that instead of using 1000, you could use `SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions` like in the previous question. I used 1000 to make it easier to read.

```sql
WITH cte_ranked AS (
  SELECT
    plan_name,
    RANK() OVER(PARTITION BY customer_id ORDER BY start_date) AS ranked_plan
  FROM foodie_fi.subscriptions
  JOIN foodie_fi.plans USING(plan_id)
)

SELECT
  COUNT(*) AS churned_after_trial,
  ROUND(COUNT(*)::numeric/1000*100) AS percentage_churned_after_trial
FROM cte_ranked
WHERE ranked_plan = 2 AND plan_name = 'churn';
```

### Answer

| churned_after_trial | percentage_churned_after_trial |
| ------------------- | ------------------------------ |
| 92                  | 9                              |

---

## 6. What is the number and percentage of customer plans after their initial free trial?

* `cte_ranked` is the same as the one in the previous question.
* The difference in the outer query is that we are not interested in customers who have churned straight after their initial free trial.
  * Note that in **WHERE** `ranked_plan` gives us the customer plan after the free trial.
  * And since we want a number and percentage for each trial, use **GROUP BY**.

```sql
WITH cte_ranked AS (
  SELECT
    plan_name,
    RANK() OVER(PARTITION BY customer_id ORDER BY start_date) AS ranked_plan
  FROM foodie_fi.subscriptions
  JOIN foodie_fi.plans USING(plan_id)
)

SELECT
  plan_name,
  COUNT(*) AS number_after_trial,
  ROUND(COUNT(*)::numeric/1000*100,1) AS percentage_after_trial
FROM cte_ranked
WHERE ranked_plan = 2 AND plan_name != 'churn'
GROUP BY plan_name
ORDER BY percentage_after_trial DESC;
```

### Answer

| plan_name     | number_after_trial | percentage_after_trial |
| ------------- | ------------------ | ---------------------- |
| basic monthly | 546                | 54.6                   |
| pro monthly   | 325                | 32.5                   |
| pro annual    | 37                 | 3.7                    |

Note that if we add the percentages, we get 90,8%. If we sum the churned percentage from the previous question (9,2% without rounding), we get 100%.

---

## 7. What is the customer count and percentage breakdown of all 5 `plan_name` values at `2020-12-31`?

* In `cte_ranked`, use **JOIN** to get all the information needed.
* Use **WHERE** since we are only interested in events before 2020-12-31.
* In `cte_ranked`, use **RANK** and **PARTITION** to identify the last event for each customer before 2020-12-31.
* In the outer query, use **WHERE** to get the last event before the given date.
* Customer and percentage calculation is the same as in previous questions.

```sql
WITH cte_ranked AS (
  SELECT
    plan_name,
    RANK() OVER(PARTITION BY customer_id ORDER BY start_date DESC) AS ranked_plan
  FROM foodie_fi.subscriptions
  JOIN foodie_fi.plans USING(plan_id)
  WHERE start_date <= '2020-12-31'
)

SELECT
  plan_name,
  COUNT(plan_name) AS n_customers,
  ROUND(COUNT(plan_name)::numeric/1000*100,1) AS percentage_customers FROM cte_ranked
WHERE ranked_plan = 1
GROUP by plan_name
ORDER BY percentage_customers DESC;
```

### Answer

| plan_name     | n_customers | percentage_customers |
| ------------- | ----------- | -------------------- |
| pro monthly   | 326         | 32.6                 |
| churn         | 236         | 23.6                 |
| basic monthly | 224         | 22.4                 |
| pro annual    | 195         | 19.5                 |
| trial         | 19          | 1.9                  |

---

## 8. How many customers have upgraded to an annual plan in 2020?

* Use **JOIN** to get all the information needed.
* Use **WHERE to:
  * Get all rows where the customer changed plans in 2020.
  * Get rows where the customer upgraded to an annual plan.
* Use **COUNT** to calculate the number of customers that have upgraded to an annual plan in 2020.
  * Note that I used **DISTINCT** to make sure that a customer who changed to an annual, then to basic, then back to annual (as an example) wouldn't be counted twice.
  
```sql
SELECT
  COUNT(DISTINCT customer_id) AS annual_upgrade
FROM foodie_fi.subscriptions
JOIN foodie_fi.plans USING(plan_id)
WHERE EXTRACT(YEAR FROM start_date) = '2020' AND plan_name = 'pro annual';
```

### Answer

| annual_upgrade |
| -------------- |
| 195            |

---

## 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

* In `trial_date`, we get the `start_date` for the trial plan.
* In  `annual_date`, we get the `start_date` for the annual plan.
* In the outer query, **JOIN** both tables, calculate the average and round it.

```sql
WITH trial_date AS (
  SELECT
    customer_id,
    start_date
  FROM foodie_fi.subscriptions
  JOIN foodie_fi.plans USING(plan_id)
  WHERE plan_name = 'trial'
),

annual_date AS (
  SELECT
    customer_id,
    start_date AS end_date
  FROM foodie_fi.subscriptions
  JOIN foodie_fi.plans USING(plan_id)
  WHERE plan_name = 'pro annual'
)

SELECT
  ROUND(AVG(end_date - start_date), 2) AS avg_days
FROM trial_date
JOIN annual_date USING(customer_id);
```

### Answer

| avg_days |
| -------- |
| 104.62   |

---

## 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

* `trial_date` and `annual_date` are the same as in the previous question.
* In the outer query:
  * Divide `end_date - start_date` by 30 to get the window it's in: first (0-29), second (30-59), etc...
  * Subtract by `0.01`. Since 0-29 is not what we want, we'll just subtract by a small value so that it won't affect the result but will give what is asked. Note that if it took a customer 60 days to subscribe to an annual plan: 60/30 = 2. But the customer would belong in the first window. So 60/30-0,1 = 1,9.
  * Use **TRUNC** to round down.
  * Add 1 so that the first window is 1 and not 0, which is not needed but it makes it easier to read.
* Use **COUNT** and **GROUP BY** to calculate the days a customer took to subscribe to an annual plan grouped by 30-day periods.

```sql
WITH trial_date AS (
  SELECT
    customer_id,
    start_date
  FROM foodie_fi.subscriptions
  JOIN foodie_fi.plans USING(plan_id)
  WHERE plan_name = 'trial'
),

annual_date AS (
  SELECT
    customer_id,
    start_date AS end_date
  FROM foodie_fi.subscriptions
  JOIN foodie_fi.plans USING(plan_id)
  WHERE plan_name = 'pro annual'
)

SELECT
  TRUNC((end_date - start_date)::numeric/30-0.01) + 1 AS interval_30_days,
  COUNT(*) AS annual_sub
FROM trial_date
JOIN annual_date USING(customer_id)
GROUP BY interval_30_days
ORDER BY interval_30_days;
```

### Answer

| interval_30_days | annual_sub |
| ---------------- | ---------- |
| 1                | 49         |
| 2                | 24         |
| 3                | 34         |
| 4                | 35         |
| 5                | 42         |
| 6                | 36         |
| 7                | 26         |
| 8                | 4          |
| 9                | 5          |
| 10               | 1          |
| 11               | 1          |
| 12               | 1          |

* 49 customers took less than 31 days to subscribe to an annual plan;
* 24 customers took more than 30 days and less than 61;
* 34 customers took more than 60 days and less than 91.
* ...

---

## How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

* In `cte_ranked`, use **WHERE** to:
  * Get rows where the event involves a basic monthly or a pro monthly.
  * Get rows where the event happened in 2020.
* Use **RANK** to identify events ordered by `start_date`.
* In the outer query, use **WHERE** to get all events where the second event involves a basic monthly subscription (which means that the first involves a pro monthly).
* Use **COUNT** to count all the events.

```sql
WITH cte_ranked AS (
  SELECT *,
  RANK() OVER(PARTITION BY customer_id ORDER BY start_date) AS ranked_date
  FROM foodie_fi.subscriptions
  JOIN foodie_fi.plans USING(plan_id)
  WHERE plan_name IN ('basic monthly','pro monthly') AND EXTRACT(YEAR FROM start_date) = '2020'
)

SELECT
  COUNT(*) AS downgrade_monthly
FROM cte_ranked
WHERE plan_name = 'basic monthly' AND ranked_date = 2;
```

### Answer

| downgrade_monthly |
| ----------------- |
| 0                 |

---

Check out solutions for the next section - [Challenge Payment Question](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/C.%20Challenge%20Payment%20Question.md)
