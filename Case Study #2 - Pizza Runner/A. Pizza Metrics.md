# Pizza Runner 🍕 - Pizza Metrics - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md).

All the data and cleaning can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/0.%20%20Data%20%26%20Cleaning.md).

---

## Pizza Metrics

1. How many pizzas were ordered?
2. How many unique customer orders were made?
3. How many successful orders were delivered by each runner?
4. How many of each type of pizza was delivered?
5. How many Vegetarian and Meatlovers were ordered by each customer?
6. What was the maximum number of pizzas delivered in a single order?
7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
8. How many pizzas were delivered that had both exclusions and extras?
9. What was the total volume of pizzas ordered for each hour of the day?
10. What was the volume of orders for each day of the week?

---

## 1. How many pizzas were ordered?

* Use **COUNT** to calculate the amount of pizzas ordered.
* Note that this includes canceled orders.

```sql
SELECT
  COUNT(*) AS pizzas_ordered
FROM customer_orders_temp;
```

### Answer

| pizzas_ordered |
| -------------- |
| 14             |

---

## 2. How many unique customer orders were made?

* Use **COUNT** and **DISTINCT** to calculate the amount of unique customer orders made.

```sql
SELECT
  COUNT(DISTINCT order_id) AS unique_orders
FROM customer_orders_temp;
```

### Answer

| unique_orders |
| ------------- |
| 10            |

---

## 3. How many successful orders were delivered by each runner?

* Since we are only interested in successful deliveries, use **WHERE**.
* Use **COUNT** and **GROUP BY** to calculate the number of rows (successful deliveries).

```sql
SELECT
  runner_id,
  COUNT(*) AS successful_deliveries
FROM runner_orders_temp
WHERE cancellation IS null
GROUP by runner_id
ORDER BY runner_id;
```

### Answer

| runner_id | successful_deliveries |
| --------- | --------------------- |
| 1         | 4                     |
| 2         | 3                     |
| 3         | 1                     |

---

## 4. How many of each type of pizza was delivered?

* Note that you need information from `customer_orders`, `runner_orders` and `pizza_names`, so join those.
  * Number of pizzas from `customer_orders`, cancelation status from `runner_orders` and pizza names from `pizza_names`.
* Use **WHERE** since we are only interested in the pizzas delivered.
* Use **COUNT** and **GROUP BY**  to calculate the amount of pizzas delivered grouped by the pizza type.

```sql
SELECT
  pizza_name,
  COUNT(*) AS n_delivered
FROM customer_orders_temp
JOIN pizza_runner.pizza_names USING(pizza_id)
JOIN runner_orders_temp USING(order_id)
WHERE cancellation IS null
GROUP BY pizza_name
ORDER BY pizza_name;
```

### Answer

| pizza_name | n_delivered |
| ---------- | ----------- |
| Meatlovers | 9           |
| Vegetarian | 3           |

---

## 5. How many Vegetarian and Meatlovers were ordered by each customer?

* You need pizza_names from `pizza_names` so use **JOIN**.
* Use **CASE** to create two new columns: one for the number of vegetarian pizzas ordered, and the other for meat lovers.
* USE **SUM** and **GROUP BY** to identify the pizza type for each row and calculate the number of pizzas for each type.

```sql
SELECT
  customer_id,
  SUM(
    CASE
      WHEN pizza_id = 1 THEN 1
      ELSE 0
    END
  ) AS n_meat_lovers,
  SUM(
    CASE
      WHEN pizza_id = 2 THEN 1
      ELSE 0
    END
  ) AS n_vegetarian
FROM customer_orders_temp
JOIN pizza_runner.pizza_names USING(pizza_id)
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | n_meat_lovers | n_vegetarian |
| ----------- | ------------- | ------------ |
| 101         | 2             | 1            |
| 102         | 2             | 1            |
| 103         | 3             | 1            |
| 104         | 3             | 0            |
| 105         | 0             | 1            |

---

## 6. What was the maximum number of pizzas delivered in a single order?

* Use **COUNT** and **GROUP BY** to calculate the number of pizzas delivered for each order.
* Use **ORDER BY DESC** and **LIMIT** to get the higher number - the maximum number.

```sql
SELECT COUNT(order_id) AS max_pizza_order FROM customer_orders_temp
GROUP BY order_id
ORDER BY max_pizza_order DESC
LIMIT 1;
```

### Answer

| max_pizza_order |
| --------------- |
| 3               |

---

## 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

* You need `customer_orders` and `runner_orders` so use **JOIN**.
* Use **WHERE** since we are only interested in the delivered pizzas.
* USE **CASE** to create two new columns: one for pizzas with at least one change, and the other for pizzas with no changes.
* USE **SUM** and **GROUP BY** to calculate the amount of pizzas delivered for each kind.

```sql
SELECT
  customer_id,
  SUM(
    CASE
      WHEN exclusions IS null AND extras IS null THEN 1
      ELSE 0
    END) AS no_changes,
  SUM(
    CASE
      WHEN exclusions IS NOT null OR extras IS NOT null then 1
      ELSE 0
    END) as changed_pizzas
FROM customer_orders_temp
JOIN runner_orders_temp USING(order_id)
WHERE cancellation IS null
GROUP BY customer_id
ORDER BY customer_id;
```

### Answer

| customer_id | no_changes | changed_pizzas |
| ----------- | ---------- | -------------- |
| 101         | 2          | 0              |
| 102         | 3          | 0              |
| 103         | 0          | 3              |
| 104         | 1          | 2              |
| 105         | 0          | 1              |

---

## 8. How many pizzas were delivered that had both exclusions and extras?

* Use **JOIN** since you need information from customer_orders and runner_orders.
* Use **WHERE** since we are only interested in delivered pizzas.
* Use **SUM** and **CASE** to calculate the amount of pizzas delivered that had no exclusions and extras.

```sql
SELECT 
  SUM(
    CASE
      WHEN exclusions IS NOT null and extras IS NOT null THEN 1
      ELSE 0
    END
  ) AS pizzas_extras_and_exclusions
FROM customer_orders_temp
JOIN runner_orders_temp USING(order_id)
WHERE cancellation IS null;
```

### Answer

| pizzas_extras_and_exclusions |
| ---------------------------- |
| 1                            |

---

## 9. What was the total volume of pizzas ordered for each hour of the day?

* Use **EXTRACT** to get the hour each pizza was ordered.
* Use **COUNT** and **GROUP BY** to calculate the number of pizzas ordered for each hour.

```sql
SELECT
  EXTRACT(hour FROM order_time) AS hour_x,
  COUNT(*) AS pizzas_ordered
FROM customer_orders_temp
GROUP BY hour_x
ORDER BY hour_x;
```

### Answer

| hour_x | pizzas_ordered |
| ------ | -------------- |
| 11     | 1              |
| 13     | 3              |
| 18     | 3              |
| 19     | 1              |
| 21     | 3              |
| 23     | 3              |

---

## 10. What was the volume of orders for each day of the week?

* Use **TO_CHAR** to get the day of the week from `order_time`.
* Use **GROUP BY** and **COUNT** to calculate the number of pizzas ordered by day of the week.

```sql
SELECT
  TO_CHAR(order_time, 'Day') AS day_pizza,
  COUNT(order_time) AS pizzas_ordered
FROM customer_orders_temp
GROUP BY day_pizza
ORDER by pizzas_ordered DESC;
```

### Answer

| day_pizza | pizzas_ordered |
| --------- | -------------- |
| Saturday  | 5              |
| Wednesday | 5              |
| Thursday  | 3              |
| Friday    | 1              |

---

Check out solutions for the next section - [Runner and Customer Experience](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/B.%20Runner%20and%20Customer%20Experience.md)
