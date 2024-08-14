# Pizza Runner 🍕 - Ingredient Optimisation - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md).

All the data and cleaning can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/0.%20%20Data%20%26%20Cleaning.md).

---

## Ingredient Optimisation

1. What are the standard ingredients for each pizza?
2. What was the most commonly added extra?
3. What was the most common exclusion?
4. Generate an order item for each record in the customers_orders table in the format of one of the following:
   - Meat Lovers
   - Meat Lovers - Exclude Beef
   - Meat Lovers - Extra Bacon
   - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
   - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
7. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

---

## 1. What are the standard ingredients for each pizza?

* In `toppings_cte`, use **UNNEST** with **STRING_TO_ARRAY** to get a column with `toppings_id` for each pizza.
* In the outer query, **JOIN** `toppings_cte`and `pizza_toppings` to get the `topping_name`.
  * Note that for this to work, I converted `toppings` in `toppings_cte` to an integer, since topping_id is also an integer.
* Use **SELECT** `pizza_name` and `STRING_AGG(topping_name)`, and **GROUP BY** pizza_name to get the standard ingredients for each pizza.

```sql
WITH toppings_cte AS (
   SELECT
      pizza_name,
      UNNEST(STRING_TO_ARRAY(toppings, ','))::integer AS toppings
   FROM recipes_temp
   JOIN pizza_runner.pizza_names USING(pizza_id)
)

SELECT
   pizza_name,
   STRING_AGG(topping_name, ', ') AS standard_igredients
FROM pizza_runner.pizza_toppings
JOIN toppings_cte
   ON topping_id = toppings
GROUP BY pizza_name;
```

### Answer

| pizza_name | standard_igredients                                                   |
| ---------- | --------------------------------------------------------------------- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

---

## 2. What was the most commonly added extra?

* In `toppings_added`, get a column with every extra added.
* In the outer query, **JOIN** `pizza_toppings` to get the `topping_name`.
* Use **COUNT** and **GROUP BY** to get the number of extras added for each extra.
* Use **LIMIT** to get the most common added extra.

```sql
WITH toppings_added AS (
   SELECT
      UNNEST(STRING_TO_ARRAY(extras, ','))::integer AS extras_added
   FROM customer_orders_temp
)

SELECT
   topping_name,
   COUNT(extras_added) AS n_added
FROM toppings_added
JOIN pizza_runner.pizza_toppings
   ON topping_id = extras_added
GROUP BY topping_name
ORDER BY n_added DESC
LIMIT 1;
```

### Answer 

| topping_name | n_added |
| ------------ | ------- |
| Bacon        | 4       |

---

## 3. What was the most common exclusion?

* The same as the previous question.
  
```sql
WITH exclusions_cte AS (
   SELECT
      UNNEST(STRING_TO_ARRAY(exclusions, ','))::integer AS exclusions
   FROM customer_orders_temp
)

SELECT
   topping_name,
   COUNT(exclusions) AS n_excluded
FROM exclusions_cte
JOIN pizza_runner.pizza_toppings
   ON topping_id = exclusions
GROUP BY topping_name
ORDER BY n_excluded DESC
LIMIT 1;
```

### Answer

| topping_name | n_excluded |
| ------------ | ---------- |
| Cheese       | 4          |

---

## 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

* `exclusions_cte` and `extras_cte` are used to get the number of the pizza and a string with its extras/exclusions.
* You will notice that both have subqueries - these are used to get a column with `n_pizza` by using **ROW_NUMBER** (to make a distinction between pizzas from the same order) and a column with the exclusions/extras.
* In the outer query (that also has a subquery to get `n_pizza`), **JOIN** all tables with relevant information.
* Create a new column with **CASE** that covers all 4 cases:
  * No extras and no exclusions - the result is just the `pizza_name`.
  * Extra(s) and no exclusions - `pizza_name` + Extras (which I called `top_extra`) from `extras_cte`.
  * Exclusion(s) and no extras - `pizza_name` + Exclusions (which I called `top_excl`) from `exclusions_cte`.
  * Extra(s) and exclusion(s) - `pizza_name` + Exclusions + Extras.
  * Use **CONCAT** to get the format you want.
* Use **ORDER BY** to get the same order as the original `customer_orders`.

```sql
WITH exclusions_cte AS(
   SELECT
      n_pizza,
      STRING_AGG(topping_name,', ') AS top_excl
   FROM
      (SELECT
         ROW_NUMBER() OVER(ORDER BY order_id) AS n_pizza,
         UNNEST(STRING_TO_ARRAY(exclusions,','))::integer AS exc 
      FROM customer_orders_temp
      ) AS sub_exc
   JOIN pizza_runner.pizza_toppings
      ON topping_id = exc
   GROUP BY n_pizza
),

extras_cte AS(
   SELECT
      n_pizza,
      STRING_AGG(topping_name,', ') AS top_extra
   FROM
      (SELECT
         ROW_NUMBER() OVER(ORDER BY order_id) AS n_pizza,
         UNNEST(STRING_TO_ARRAY(extras,','))::integer AS ext 
      FROM customer_orders_temp
      ) AS sub_extra
   JOIN pizza_runner.pizza_toppings
      ON topping_id = ext
   GROUP BY n_pizza
)

SELECT 
   order_id, 
   customer_id, 
   CASE
      WHEN (top_extra IS NULL AND top_excl IS NULL) THEN pizza_name
      WHEN (top_extra IS NOT NULL AND top_excl IS NULL) THEN CONCAT(pizza_name, ' - Extra ', top_extra)
      WHEN (top_extra IS NULL AND top_excl IS NOT NULL) THEN CONCAT(pizza_name, ' - Exclude ', top_excl)
      ELSE CONCAT(pizza_name, ' - Exclude ', top_excl, ' - Extra ', top_extra)
   END AS cust_order
FROM
   (SELECT
      *,
      ROW_NUMBER() OVER(ORDER BY order_id) AS n_pizza
   FROM customer_orders_temp
   ) AS helper_row
JOIN pizza_runner.pizza_names USING(pizza_id)
LEFT JOIN extras_cte USING(n_pizza)
LEFT JOIN exclusions_cte USING(n_pizza)
ORDER BY order_id, pizza_name, exclusions DESC;
```

### Answer

| order_id | customer_id | cust_order                                                      |
| -------- | ----------- | --------------------------------------------------------------- |
| 1        | 101         | Meatlovers                                                      |
| 2        | 101         | Meatlovers                                                      |
| 3        | 102         | Meatlovers                                                      |
| 3        | 102         | Vegetarian                                                      |
| 4        | 103         | Meatlovers - Exclude Cheese                                     |
| 4        | 103         | Meatlovers - Exclude Cheese                                     |
| 4        | 103         | Vegetarian - Exclude Cheese                                     |
| 5        | 104         | Meatlovers - Extra Bacon                                        |
| 6        | 101         | Vegetarian                                                      |
| 7        | 105         | Vegetarian - Extra Bacon                                        |
| 8        | 102         | Meatlovers                                                      |
| 9        | 103         | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
| 10       | 104         | Meatlovers                                                      |
| 10       | 104         | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

If you're like me, at first sight, you're thinking this must be wrong. Why would someone choose a vegetarian pizza just to add bacon? I check every answer, I assure you this is right. People who choose vegetarian options don't have to be vegetarian. 

---

## 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

* `main_cte` will have everything we need, including `n_pizza`, to identify different pizzas in the same order.
* `toppings_id_after_exclusions_` will do exactly that: get a string of toppings id's after we remove toppings that are in `exclusions`.
* `final_toppings_id` gets a string of toppings id's after the exclusions (from the previous point) and the added extras. If there are two Bacon's in an order, it will have two `1`'s.
* `count_to_replace` will count toppings id's and `to_replace` will get `2x` topping when count is 2.
* In the outer query, all that's left to do is to use **STRING_AGG** to format everything.

```sql
WITH main_cte AS (
   SELECT
      ROW_NUMBER() OVER (ORDER BY order_id, pizza_name, exclusions DESC) AS n_pizza,
      order_id,
      customer_id,
      pizza_id,
      pizza_name,
      exclusions,
      extras,
      toppings
   FROM customer_orders_temp
   JOIN pizza_runner.pizza_names USING(pizza_id)
   JOIN recipes_temp USING(pizza_id)
),

toppings_id_after_exclusions AS (
   SELECT
      n_pizza,
      ARRAY_TO_STRING(ARRAY_AGG(e),',') t_exclusion
   FROM main_cte CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(toppings,',')) e
   WHERE NOT e = ANY(coalesce(STRING_TO_ARRAY(exclusions,','), '{}'))
   GROUP BY n_pizza 
),

final_toppings_id AS (
   SELECT
      n_pizza,
      order_id,
      customer_id,
      pizza_name,
   	ARRAY_TO_STRING(ARRAY(SELECT UNNEST(STRING_TO_ARRAY(t_final,',')::int[]) ORDER BY 1),', ') AS f_top_id
   FROM
      (SELECT
         *,
         CASE
            WHEN extras IS null THEN t_exclusion
            ELSE extras || ',' || t_exclusion
         END AS t_final
      FROM main_cte
      JOIN toppings_id_after_exclusions USING(n_pizza)
      ) h_c
),

count_to_replace AS (
   SELECT
      n_pizza,
      elmt,
      COUNT(elmt)
   FROM
      (SELECT
         n_pizza,
         UNNEST(STRING_TO_ARRAY(f_top_id,','))::int AS elmt
      FROM final_toppings_id
      ) AS helper_count
   GROUP BY n_pizza, elmt
), 

to_replace AS (
   SELECT
      n_pizza,
      CASE
         WHEN count = 2 THEN REPLACE(topping_name,topping_name,'2x ' || topping_name)
         ELSE topping_name
      END AS n_top
   FROM count_to_replace
   JOIN pizza_runner.pizza_toppings
      ON topping_id = elmt
)

SELECT
   order_id,
   customer_id,
   pizza_name || ': ' || final_top AS cust_order
   FROM
      (SELECT
         n_pizza,
         STRING_AGG(n_top,', ') AS final_top
      FROM to_replace
      GROUP BY n_pizza
      ) agg_top
JOIN final_toppings_id USING(n_pizza)
ORDER BY n_pizza;
```

### Answer

| order_id | customer_id | cust_order                                                                           |
| -------- | ----------- | ------------------------------------------------------------------------------------ |
| 1        | 101         | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 2        | 101         | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | 102         | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | 102         | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce               |
| 4        | 103         | Meatlovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 4        | 103         | Meatlovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 4        | 103         | Vegetarian: Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                       |
| 5        | 104         | Meatlovers: 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | 101         | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce               |
| 7        | 105         | Vegetarian: Bacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce        |
| 8        | 102         | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 9        | 103         | Meatlovers: 2x Bacon, BBQ Sauce, Beef, 2x Chicken, Mushrooms, Pepperoni, Salami      |
| 10       | 104         | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 10       | 104         | Meatlovers: 2x Bacon, Beef, 2x Cheese, Chicken, Pepperoni, Salami                    |

---

## 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

* We'll use the previous `final_toppings_id` for this question.
* In the subquery, use **WHERE** since we are only interested in the pizzas that have been delivered.
* In the subquery, use **UNNEST** and **STRING_TO_ARRAY**  to get a column with toppings IDs that have been used.
* Use **COUNT** and **GROUP BY** to calculate the total quantity of each ingredient used in delivered pizzas.

```sql
SELECT
   topping_name,
   COUNT(elmt) AS n_delivered
FROM
   (SELECT
      UNNEST(STRING_TO_ARRAY(f_top_id,','))::integer AS elmt
   FROM final_toppings_id
   JOIN runner_orders_temp USING(order_id)
   WHERE pickup_time IS NOT null
   ) AS helper
JOIN pizza_runner.pizza_toppings
   ON topping_id = elmt
GROUP BY topping_name
ORDER BY n_delivered DESC;
```

### Answer

| topping_name | n_delivered |
| ------------ | ----------- |
| Bacon        | 12          |
| Mushrooms    | 11          |
| Cheese       | 10          |
| Pepperoni    | 9           |
| Chicken      | 9           |
| Salami       | 9           |
| Beef         | 9           |
| BBQ Sauce    | 8           |
| Tomato Sauce | 3           |
| Onions       | 3           |
| Tomatoes     | 3           |
| Peppers      | 3           |

---

Check out the solutions for the next section - [Pricing and Ratings](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/D.%20Pricing%20and%20Ratings.md).
