# Danny's Diner :stew: - Questions and Solutions

## 1. What is the total amount each customer spent at the restaurant?

* The information needed to answer this question is in two tables: `sales` and `menu`, so join those.
* Use **SUM** to calculate the total amount spent, and **GROUP BY** to group by customer.

```sql
SELECT
  customer_id,
  SUM(price) AS amount_spent
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
GROUP BY customer_id
ORDER BY customer_id
```

### Answer

| customer_id | amount_spent |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

  
## 2. How many days has each customer visited the restaurant?

* Use **COUNT** with **DISTINCT** to calculate the number of unique days the restaurant was visited.
* **GROUP BY** to group by customer.

```sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS days_visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id
```

### Answer

| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

  
## 3. What was the first item from the menu purchased by each customer?

* In the CTE, create a new column using **DENSE_RANK** and **PARTITION** ordered by order_date.
* You need `product_name`, so join `menu`.
* In the final query, use **GROUP BY** and **STRING_AGG** to get a string of products ordered by customer, and **DISTINCT** to remove duplicates.
* Use **WHERE** since we are only interested in the first order.

```sql
WITH helper AS( 
  SELECT
    customer_id,
    product_name,
    DENSE_RANK() OVER(
      PARTITION BY customer_id
      ORDER BY order_date
    ) AS order_rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu USING (product_id)
)

SELECT
  customer_id,
  STRING_AGG(DISTINCT product_name,', ') AS first_order
FROM helper
WHERE order_rank = 1
GROUP BY customer_id
ORDER BY customer_id
```

### Answer

| customer_id | first_order  |
| ----------- | ------------ |
| A           | curry, sushi |
| B           | curry        |
| C           | ramen        |

  
## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

* Use **COUNT** and **GROUP BY** to count the amount of times a product was purchased.
* Order by the number of times purchased. Since we are only interested in the most purchased item, use **LIMIT**.

```sql
SELECT
  product_name,
  COUNT(product_name) AS times_purchased
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
GROUP BY product_name
ORDER BY times_purchased DESC
LIMIT 1
```

### Answer

| product_name | times_purchased |
| ------------ | --------------- |
| ramen        | 8               |

  
## 5. Which item was the most popular for each customer?

* In `popular`, group results by customer and product.
* Use **DENSE_RANK** and **PARTITION** ordered by `COUNT(product_name)` to get a new column identifying products' popularity.
* In the outer query, use **WHERE** to get the most popular items for each customer.
* Use **STRING_AGG** to make the table easier to read.

```sql
WITH popular AS(
  SELECT
    customer_id,
    product_name,
    DENSE_RANK() OVER(
      PARTITION BY customer_id
      ORDER BY COUNT(product_name) DESC
    ) AS order_rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu USING(product_id)
  GROUP BY customer_id, product_name
)

SELECT
  customer_id,
  STRING_AGG(product_name,', ') AS most_popular
FROM popular
WHERE order_rank = 1
GROUP BY customer_id
ORDER BY customer_id
```

### Answer

| customer_id | most_popular        |
| ----------- | ------------------- |
| A           | ramen               |
| B           | ramen, curry, sushi |
| C           | ramen               |


## 6. Which item was purchased first by the customer after they became a member?

* You need information from all tables, so join those in `ranked`.
* Use **DENSE_RANK** and **WHERE** to identify the rows where the customer is already a member, ordered by the order date.
* In the outer query, use **WHERE** since we are only interested in the first order after the customer becomes a member.
  * We will assume that, for rows where the order date is the same as the member join date, the customer became a member after ordering.
* Had the customers ordered more than one product for their first purchase, note that we would have used **STRING_AGG** and **GROUP BY**, similarly to the last question, to make the table easier to read.

```sql
WITH ranked AS(
  SELECT
    customer_id,
    product_name,
    DENSE_RANK() OVER(
      PARTITION BY customer_id
      ORDER BY order_date
    ) AS rank_order
  FROM dannys_diner.sales
  JOIN dannys_diner.menu USING(product_id)
  JOIN dannys_diner.members USING(customer_id)
  WHERE join_date < order_date
)

SELECT
  customer_id,
  product_name
FROM ranked
WHERE rank_order = 1
ORDER BY customer_id
```

### Answer

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

## 7. Which item was purchased just before the customer became a member?

* You need information from all tables, so in `ranked`, join them.
* Use **DENSE_RANK** and **WHERE** to identify the rows where the customer is not a member, ordered by the order date, descending.
* In the outer query, use **WHERE** since we are only interested in the last order before the customer becomes a member.
  * We will not count purchases made on the same day as the joining date.
* Use **STRING_AGG** and **GROUP BY** to make the table easier to read.
 
```sql
WITH ranked AS(
  SELECT
    customer_id,
    product_name,
    DENSE_RANK() OVER(
      PARTITION BY customer_id
      ORDER BY order_date DESC
    ) AS rank_order
  FROM dannys_diner.sales
  JOIN dannys_diner.menu USING(product_id)
  JOIN dannys_diner.members USING(customer_id)
  WHERE join_date > order_date
)

SELECT
  customer_id,
  STRING_AGG(product_name,', ') AS purchased_before_member
FROM ranked
WHERE rank_order = 1
GROUP BY customer_id
ORDER BY customer_id
```

### Answer

| customer_id | purchased_before_member |
| ----------- | ----------------------- |
| A           | sushi, curry            |
| B           | sushi                   |

  
## 8. What is the total items and amount spent for each member before they became a member?

* You need information from all tables, so join them.
* Use **WHERE** since we are only interested in purchases made before customers A and B became members.
* Use **COUNT** to calculate the number of items purchased.
* Use **SUM** to calculate the amount spent for each customer.
* **GROUP** to group by customer.

```sql
SELECT
  customer_id,
  COUNT(product_name) AS n_items,
  SUM(price) as spent_before_member
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
JOIN dannys_diner.members USING(customer_id)
WHERE order_date < join_date
GROUP BY customer_id
ORDER BY customer_id
```

### Answer

| customer_id | n_items | spent_before_member |
| ----------- | ------- | ------------------- |
| A           | 2       | 25                  |
| B           | 3       | 40                  |

 
## 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

* You need `menu` for the prices, so we will join the table.
* With **CASE**, create a new column: if the customer bought sushi, then, for that purchase, the customer earned `2 * 10 * price` points. For everything else, they earned `10 * price` points.
* Use **SUM** to add the points earned and **GROUP BY** to group by customer.

```sql
SELECT customer_id,
  SUM(
    CASE
      WHEN product_name = 'sushi' THEN 20*price
      ELSE 10*price
    END
  ) AS points
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
GROUP BY customer_id
ORDER BY customer_id
```

### Answer

| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |

## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

* We need information from all tables, so join `menu` and `members`.
* Use **WHERE** since we are only interested in purchases before February.
* Similarly to the last question, use **CASE**, **SUM** and **GROUP BY** to calculate the amount of points earned per customer.
* Add a condition in **CASE** for points earned in the first week after a customer becomes a member: purchases made **BETWEEN** their join date and their join date + 6 days.
* Note that we could also calculate the points customer C earned by using **LEFT JOIN** members instead of an inner join.

```sql
SELECT
  customer_id,
  SUM(
    CASE 
      WHEN order_date BETWEEN join_date AND join_date + INTERVAL '6' day THEN 20*price
      WHEN product_name = 'sushi' THEN 20*price
      ELSE 10*price
    END
  ) AS points_january
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
JOIN dannys_diner.members USING(customer_id)
WHERE order_date < '2021-02-01'
GROUP BY customer_id
ORDER BY customer_id
```

### Answer

| customer_id | points_january |
| ----------- | -------------- |
| A           | 1370           |
| B           | 820            |
