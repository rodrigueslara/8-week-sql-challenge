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

1. What are the standard ingredients for each pizza?

* In `toppings_cte`, use **UNNEST** with **STRING_TO_ARRAY** to get a column with `toppings_id` for each pizza.
* In the outer query, **JOIN** `toppings_cte`and `pizza_toppings` to get the `topping_name`.
  * Note that for this to work, I converted `toppings` in `toppings_cte` to an integer, since topping_id is also an integer.
* Use **SELECT** `pizza_name` and `STRING_AGG(topping_name)`, and **GROUP BY** pizza_name to get the standard ingredients for each pizza.

```sql
WITH toppings_cte AS(
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
GROUP BY pizza_name
```

### Answer

| pizza_name | standard_igredients                                                   |
| ---------- | --------------------------------------------------------------------- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

---
