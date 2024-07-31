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
