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
