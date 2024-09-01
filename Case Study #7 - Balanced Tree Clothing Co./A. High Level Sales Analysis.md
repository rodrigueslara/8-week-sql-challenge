# Balanced Tree Clothing Co. 🌲 - High Level Sales Analysis - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./README.md).

--- 

## High Level Sales Analysis

1. What was the total quantity sold for all products?
2. What is the total generated revenue for all products before discounts?
3. What was the total discount amount for all products?

---

## 1. What was the total quantity sold for all products?

* Use **SUM** and **GROUP BY** to calculate the total quantity of products sold.

```sql
SELECT
  product_name, SUM(qty) AS total_sold
FROM balanced_tree.sales
JOIN balanced_tree.product_details
  ON prod_id = product_id
GROUP BY product_name;
```

### Answer

| product_name                     | total_sold |
| -------------------------------- | ---------- |
| White Tee Shirt - Mens           | 3800       |
| Navy Solid Socks - Mens          | 3792       |
| Grey Fashion Jacket - Womens     | 3876       |
| Navy Oversized Jeans - Womens    | 3856       |
| Pink Fluro Polkadot Socks - Mens | 3770       |
| Khaki Suit Jacket - Womens       | 3752       |
| Black Straight Jeans - Womens    | 3786       |
| White Striped Socks - Mens       | 3655       |
| Blue Polo Shirt - Mens           | 3819       |
| Indigo Rain Jacket - Womens      | 3757       |
| Cream Relaxed Jeans - Womens     | 3707       |
| Teal Button Up Shirt - Mens      | 3646       |

---

## 2. What is the total generated revenue for all products before discounts?

* Use **SUM** to calculate the total revenue before discounts.

```sql
SELECT
  product_name, SUM(s.price * qty) AS revenue_before_discount
FROM balanced_tree.sales s
JOIN balanced_tree.product_details ON prod_id = product_id
GROUP BY product_name;
```

### Answer

| product_name                     | revenue_before_discount |
| -------------------------------- | ----------------------- |
| White Tee Shirt - Mens           | 152000                  |
| Navy Solid Socks - Mens          | 136512                  |
| Grey Fashion Jacket - Womens     | 209304                  |
| Navy Oversized Jeans - Womens    | 50128                   |
| Pink Fluro Polkadot Socks - Mens | 109330                  |
| Khaki Suit Jacket - Womens       | 86296                   |
| Black Straight Jeans - Womens    | 121152                  |
| White Striped Socks - Mens       | 62135                   |
| Blue Polo Shirt - Mens           | 217683                  |
| Indigo Rain Jacket - Womens      | 71383                   |
| Cream Relaxed Jeans - Womens     | 37070                   |
| Teal Button Up Shirt - Mens      | 36460                   |

---

## 3. What was the total discount amount for all products?

* Use **SUM** to calculate the total discount amount.

```sql
SELECT
  product_name, ROUND(SUM(qty*s.price*discount::numeric/100),2) AS discount_value
FROM balanced_tree.sales s
JOIN balanced_tree.product_details ON prod_id = product_id
GROUP BY product_name;
```

### Answer

| product_name                     | discount_value |
| -------------------------------- | -------------- |
| White Tee Shirt - Mens           | 18377.60       |
| Navy Solid Socks - Mens          | 16650.36       |
| Grey Fashion Jacket - Womens     | 25391.88       |
| Navy Oversized Jeans - Womens    | 6135.61        |
| Pink Fluro Polkadot Socks - Mens | 12952.27       |
| Khaki Suit Jacket - Womens       | 10243.05       |
| Black Straight Jeans - Womens    | 14744.96       |
| White Striped Socks - Mens       | 7410.81        |
| Blue Polo Shirt - Mens           | 26819.07       |
| Indigo Rain Jacket - Womens      | 8642.53        |
| Cream Relaxed Jeans - Womens     | 4463.40        |
| Teal Button Up Shirt - Mens      | 4397.60        |


---

Check out the solutions for the next section [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./B.%20Transaction%20Analysis.md)
