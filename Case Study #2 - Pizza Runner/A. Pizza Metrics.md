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
FROM customer_orders_temp
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
FROM customer_orders_temp
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
ORDER BY runner_id
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
ORDER BY pizza_name
```

### Answer

| pizza_name | n_delivered |
| ---------- | ----------- |
| Meatlovers | 9           |
| Vegetarian | 3           |

---

# 5. How many Vegetarian and Meatlovers were ordered by each customer?

* You need pizza_names from `pizza_names` so use **JOIN**.
* Use **COUNT** and **GROUP BY** to calculate the amount of pizzas ordered, grouped by customer and pizza_name.

```sql
SELECT
  customer_id,
  pizza_name,
  COUNT(*) AS n_ordered
FROM customer_orders_temp
JOIN pizza_runner.pizza_names USING(pizza_id)
GROUP BY customer_id, pizza_name
ORDER BY customer_id
```

| customer_id | pizza_name | n_ordered |
| ----------- | ---------- | --------- |
| 101         | Meatlovers | 2         |
| 101         | Vegetarian | 1         |
| 102         | Meatlovers | 2         |
| 102         | Vegetarian | 1         |
| 103         | Meatlovers | 3         |
| 103         | Vegetarian | 1         |
| 104         | Meatlovers | 3         |
| 105         | Vegetarian | 1         |

---
