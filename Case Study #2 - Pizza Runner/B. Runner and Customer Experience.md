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

* **JOIN** `runner_orders` in `order_time`.
* We are not interested in the canceled orders, so use **WHERE**.
* Use **COUNT** and **GROUP BY** to identify the number of pizzas per order, and `EXTRACT(EPOCH FROM AVG(pickup_time - order_time)/60)` to get the minutes spent to prepare.
  * Note that **AVG** works since each pizza from the same order has the same preparation time.
* In the outer query, use **AVG** and **GROUP BY** to get the average minutes taken to prepare an order with *n* amount of pizzas.
  * For more information on **EXTRACT**, **numeric**, etc, look at previous question.

```sql
WITH order_time AS(
  SELECT
    COUNT(pizza_id) AS n_pizza,
    EXTRACT(EPOCH FROM AVG(pickup_time - order_time)/60) AS wait_time
  FROM customer_orders_temp
  JOIN runner_orders_temp USING(order_id)
  WHERE pickup_time IS NOT null
  GROUP BY order_id
)

SELECT
  n_pizza,
  ROUND(AVG(wait_time)::numeric,2) AS prep_time
FROM order_time
GROUP BY n_pizza
ORDER BY n_pizza;
```

### Answer

| n_pizza | prep_time |
| ------- | --------- |
| 1       | 12.36     |
| 2       | 18.38     |
| 3       | 29.28     |

**Yes**, there is a relationship between the number of pizzas and preparation time: As the number of pizzas increases, so does the preparation time.

---

## 4. What was the average distance traveled for each customer?

* Use **JOIN** in `order_distance` to get information from `customer_orders` and `runner_order`.
  * Note that we need to use **DISTINCT** to get rid of double `order_id` since the distance traveled for each customer is only traveled once per order: it does not depend on the number of pizzas ordered.
* We are not interested in the canceled orders, so use **WHERE**.
* **SELECT** `DISTINCT order_id`, `customer_id` and `distance_km`.
* In the outer query, use **AVG** AND **GROUP BY** to get the average distance for each customer.
  * For more information on **ROUND** and **numeric** look at question #2.

```sql
WITH order_distance AS(
  SELECT
    DISTINCT order_id,
    customer_id,
    distance_km
  FROM runner_orders_temp
  JOIN customer_orders_temp USING(order_id)
  WHERE cancellation IS null
)

SELECT
  customer_id,
  ROUND(AVG(distance_km)::numeric,2) AS avg_traveled
FROM order_distance
GROUP BY customer_id
ORDER BY customer_id;
```

### Answer

| customer_id | avg_traveled |
| ----------- | ------------ |
| 101         | 20.00        |
| 102         | 18.40        |
| 103         | 23.40        |
| 104         | 10.00        |
| 105         | 25.00        |

---

## 5. What was the difference between the longest and shortest delivery times for all orders?

* Use **MAX** and **MIN** to get the longest and shortest delivery times, and subtract them to get the difference.

```sql
SELECT
  MAX(duration_min) - MIN(duration_min) AS dif_time
FROM runner_orders_temp;
```

### Answer

| dif_time |
| -------- |
| 30       |

---

## 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

* Use **JOIN** in `order_distance` to get information from `customer_orders` and `runner_order`.
  * Note that we need to use **DISTINCT** to get rid of double `order_id` since the two or three pizzas from the same order are delivered at the same time.
* We are not interested in the canceled orders, so use **WHERE**.
* **SELECT** **DISTINCT** `runner_id`, `order_id` and `distance_km/duration_min*60` to get the average speed for each order in km/h.
  * For more information on **ROUND** and **numeric** look at question #2.

```sql
SELECT
  DISTINCT runner_id,
  order_id,
  ROUND((distance_km/duration_min*60)::numeric,2) AS avg_speed
FROM runner_orders_temp
JOIN customer_orders_temp USING(order_id)
WHERE cancellation IS null
ORDER BY runner_id, avg_speed;
```

### Answer 

| runner_id | order_id | avg_speed |
| --------- | -------- | --------- |
| 1         | 1        | 37.50     |
| 1         | 3        | 40.20     |
| 1         | 2        | 44.44     |
| 1         | 10       | 60.00     |
| 2         | 4        | 35.10     |
| 2         | 7        | 60.00     |
| 2         | 8        | 93.60     |
| 3         | 5        | 40.00     |

There isn't enough information to notice a trend, but it's worth noting that runner 2 has a difference of almost 60 km/h between their highest and lowest average speed.

---

## 7. What is the successful delivery percentage for each runner?

* Successful delivery percentage is equal to the number of successful deliveries divided by the number of total orders times 100.
  * The number of successful deliveries is `COUNT(pickup_time)`, since **COUNT** doesn't count `null` values;
  * The number of total orders is `COUNT(order_id)`.
  * Note that `::numeric` is used so that SQL gives the right value, not the truncated one.
* Use **GROUP BY** runner to get the successful delivery percentage for each runner.

```sql
SELECT
  runner_id,
  ROUND(COUNT(pickup_time)::numeric/COUNT(order_id)*100) AS successful_deliveries
FROM runner_orders_temp
GROUP BY runner_id
ORDER BY runner_id
```

### Answer

| runner_id | successful_deliveries |
| --------- | --------------------- |
| 1         | 100                   |
| 2         | 75                    |
| 3         | 50                    |

---

Check out the solutions for the next section - [Ingredient Optimisation](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/C.%20Ingredient%20Optimisation.md)
