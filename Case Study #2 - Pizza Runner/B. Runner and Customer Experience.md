# Pizza Runner 🍕 - Runner and Customer Experience - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md).

All the data and cleaning can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/0.%20%20Data%20%26%20Cleaning.md).

---

## Runner and Customer Experience

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
4. What was the average distance travelled for each customer?
5. What was the difference between the longest and shortest delivery times for all orders?
6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
7. What is the successful delivery percentage for each runner?

---

## 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

* Use **TO_CHAR** with `'ww'` to get the week with the week starting on 2021-01-01.
* Use **COUNT** and **GROUP BY** to calculate the number of registered runners for each week.

```sql
SELECT
  TO_CHAR(registration_date, 'ww') as week_registration,
  COUNT(runner_id) AS runners_registered
FROM pizza_runner.runners
GROUP BY week_registration
ORDER BY week_registration;
```

### Answer

| week_registration | runners_registered |
| ----------------- | ------------------ |
| 01                | 2                  |
| 02                | 1                  |
| 03                | 1                  |

---

## 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

* You need information from `customer_orders` and `runner_orders` so in `pickup_time_diff`, join those.
* Note that after the **JOIN**, you get repeated `order_id` from `customer_orders`, so we need to get rid of them by using **DISTINCT**.
* We are not interested in the canceled orders, so use **WHERE**.
* Use **SELECT** to get all you need: `order_id`, `runner_id`, `pickup_time - order_time`.
  * Use **EXTRACT** with **EPOCH** to get the number of seconds passed from the `order_time` to the `pickup_time`.
  * Note that if we had used **EXTRACT** with **MINUTE** or **DATE_PART** with `minute` we would have lost the seconds, so it would not be as accurate.
* In the outer query, use **AVG** and **GROUP BY** to calculate each runner's average time to pick up the order.
  * To get the time in minutes, divide by 60;
  * Use `::numeric` to convert it to numeric;
  * And **ROUND** to round it to 2 decimal places.

```sql
WITH pickup_time_diff AS(
  SELECT
    DISTINCT order_id,
    runner_id,
    EXTRACT(EPOCH FROM (pickup_time - order_time)) AS diff_pickup
  FROM customer_orders_temp
  JOIN runner_orders_temp USING(order_id)
  WHERE pickup_time IS NOT null
)

SELECT
  runner_id,
  ROUND((AVG(diff_pickup)/60)::numeric,2) AS avg_pickup
FROM pickup_time_diff
GROUP BY runner_id
ORDER BY runner_id;
```

### Answer

| runner_id | avg_pickup |
| --------- | ---------- |
| 1         | 14.33      |
| 2         | 20.01      |
| 3         | 10.47      |

---

## 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
