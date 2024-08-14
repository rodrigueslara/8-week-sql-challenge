# Pizza Runner 🍕 - Pricing and Ratings + Bonus Question - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md).

All the data and cleaning can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/0.%20%20Data%20%26%20Cleaning.md).

---

## Pricing and Ratings

1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
2. What if there was an additional $1 charge for any pizza extras?
   - Add cheese is $1 extra
3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
   - customer_id
   - order_id
   - runner_id
   - rating
   - order_time
   - pickup_time
   - Time between order and pickup
   - Delivery duration
   - Average speed
   - Total number of pizzas
5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

**Bonus question** - If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

---

## 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

* Use **JOIN** to get all the information needed.
* Use **WHERE** since we are only interested in delivered orders.
* Use **SUM** and **CASE** to calculate the money Pizza Runner has made.

```sql
SELECT
   SUM(
      CASE
         WHEN pizza_id = 1 THEN 12
         ELSE 10
      END
   ) AS money_made
FROM customer_orders_temp
JOIN runner_orders_temp USING(order_id)
WHERE pickup_time IS NOT null;
```

### Answer

| money_made |
| ---------- |
| 138        |

---

## 2. What if there was an additional $1 charge for any pizza extras?

* Use **JOIN** to get all the information needed.
* Use **WHERE** since we are only interested in delivered orders.
* Use **SUM** and **CASE** to calculate the money Pizza Runner has made (with charges for extras). There are four cases:
   * Meat Lover pizza with no extras;
   * Vegetarian pizza with no extras;
   * Meat Lover pizza with extras;
   * Vegetarian pizza with extras.
* Use **ARRAY_LENGTH** to calculate the number of extras.

```sql
SELECT
   SUM(
      CASE
         WHEN pizza_id = 1 THEN 12
         ELSE 10
      END
      + COALESCE(ARRAY_LENGTH(STRING_TO_ARRAY(extras,','),1), 0)
   ) AS money_made
FROM customer_orders_temp
JOIN runner_orders_temp USING(order_id)
WHERE pickup_time IS NOT null;
```

### Answer


| money_made |
| ---------- |
| 142        |

---

## 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

* Note that it doesn't make sense for a runner assigned to a canceled order to get rated.

```sql
DROP TABLE IF EXISTS ratings;
CREATE TABLE ratings (
   order_id INTEGER,
   customer_id INTEGER,
   runner_id INTEGER,
   rating INTEGER
);

INSERT INTO ratings
(order_id, customer_id, runner_id, rating)
VALUES
   (1,101,1,4),
   (2,101,1,4),
   (3,102,1,3),
   (4,103,2,2),
   (5,104,3,5),
   (7,105,2,5),
   (8,102,2,4),
   (10,104,1,4);

SELECT * FROM ratings;
```

### Answer

| order_id | customer_id | runner_id | rating |
| -------- | ----------- | --------- | ------ |
| 1        | 101         | 1         | 4      |
| 2        | 101         | 1         | 4      |
| 3        | 102         | 1         | 3      |
| 4        | 103         | 2         | 2      |
| 5        | 104         | 3         | 5      |
| 7        | 105         | 2         | 5      |
| 8        | 102         | 2         | 4      |
| 10       | 104         | 1         | 4      |

---

## Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id, order_id, runner_id, rating, order_time, pickup_time, time between order and pickup, delivery duration, average speed and total number of pizzas

* All information has been calculated before.
* Note that this is only for delivered pizzas.

```sql
SELECT
   c.order_id,
   c.customer_id,
   r.runner_id,
   rating,
   order_time,
   pickup_time,
   ROUND((EXTRACT(EPOCH FROM (pickup_time-order_time))/60)::numeric,2) AS pickup_duration_min,
   duration_min,
   ROUND((distance_km/(duration_min::numeric/60))::numeric,2) AS avg_speed,
   COUNT(c.order_id) AS n_pizzas
FROM customer_orders_temp AS c
JOIN runner_orders_temp USING(order_id)
JOIN ratings r USING(order_id)
GROUP BY c.order_id, c.customer_id, r.runner_id, rating, order_time, pickup_time, pickup_duration_min, avg_speed, duration_min;
```

### Answer

| order_id | customer_id | runner_id | rating | order_time               | pickup_time              | pickup_duration_min | duration_min | avg_speed | n_pizzas |
| -------- | ----------- | --------- | ------ | ------------------------ | ------------------------ | ------------------- | ------------ | --------- | -------- |
| 1        | 101         | 1         | 4      | 2020-01-01T18:05:02.000Z | 2020-01-01T18:15:34.000Z | 10.53               | 32           | 37.50     | 1        |
| 2        | 101         | 1         | 4      | 2020-01-01T19:00:52.000Z | 2020-01-01T19:10:54.000Z | 10.03               | 27           | 44.44     | 1        |
| 3        | 102         | 1         | 3      | 2020-01-02T23:51:23.000Z | 2020-01-03T00:12:37.000Z | 21.23               | 20           | 40.20     | 2        |
| 4        | 103         | 2         | 2      | 2020-01-04T13:23:46.000Z | 2020-01-04T13:53:03.000Z | 29.28               | 40           | 35.10     | 3        |
| 5        | 104         | 3         | 5      | 2020-01-08T21:00:29.000Z | 2020-01-08T21:10:57.000Z | 10.47               | 15           | 40.00     | 1        |
| 7        | 105         | 2         | 5      | 2020-01-08T21:20:29.000Z | 2020-01-08T21:30:45.000Z | 10.27               | 25           | 60.00     | 1        |
| 8        | 102         | 2         | 4      | 2020-01-09T23:54:33.000Z | 2020-01-10T00:15:02.000Z | 20.48               | 15           | 93.60     | 1        |
| 10       | 104         | 1         | 4      | 2020-01-11T18:34:49.000Z | 2020-01-11T18:50:20.000Z | 15.52               | 10           | 60.00     | 2        |

---

## 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

* In `revenue`, calculate the money made by selling pizzas before paying the runners.
* In `expenses`, calculate the money spent on the runners' pay.
* In the outer query, calculate the profit.

```sql
WITH revenue AS (
   SELECT
      SUM(
         CASE
            WHEN pizza_id = 1 THEN 12
            ELSE 10
         END
      ) AS money_made
   FROM customer_orders_temp
   JOIN runner_orders_temp USING(order_id)
   WHERE pickup_time IS NOT null
),

expenses AS (
   SELECT
      SUM(distance_km) * 0.30 AS paid_runners
   FROM runner_orders_temp
)

SELECT money_made - paid_runners AS profit FROM expenses
JOIN revenue
ON True;
```

### Answer

| profit |
| ------ |
| 94.44  |

---

## Bonus Question - If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

```sql
INSERT INTO pizza_runner.pizza_names
	("pizza_id", "pizza_name")
VALUES 
	(3, 'Supreme');

INSERT INTO pizza_runner.pizza_recipes
	("pizza_id", "toppings")
VALUES 
	(3,'1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12');
```

### Answer

| pizza_id | pizza_name | toppings                              |
| -------- | ---------- | ------------------------------------- |
| 1        | Meatlovers | 1, 2, 3, 4, 5, 6, 8, 10               |
| 2        | Vegetarian | 4, 6, 7, 9, 11, 12                    |
| 3        | Supreme    | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12 |

