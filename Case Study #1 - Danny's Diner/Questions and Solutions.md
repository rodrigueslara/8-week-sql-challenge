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
    )AS order_rank
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
    )AS order_rank
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
