# Danny's Diner :stew: - Questions and Solutions

## 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT customer_id, SUM(price) FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
GROUP BY customer_id
ORDER BY customer_id
```
