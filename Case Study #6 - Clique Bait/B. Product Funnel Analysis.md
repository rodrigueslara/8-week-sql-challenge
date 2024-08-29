# Clique Bait 🎣 - Product Funnel Analysis

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%236%20-%20Clique%20Bait/README.md).

--- 

## Product Funnel Analysis

Using a single SQL query - create a new output table which has the following details:

* How many times was each product viewed?
* How many times was each product added to cart?
* How many times was each product added to a cart but not purchased (abandoned)?
* How many times was each product purchased?

Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables - answer the following questions:

1. Which product had the most views, cart adds and purchases?
2. Which product was most likely to be abandoned?
3. Which product had the highest view to purchase percentage?
4. What is the average conversion rate from view to cart add?
5. What is the average conversion rate from cart add to purchase?

### Answer

* In `page_views`, calculate product views.
* In `cart_add`, calculate the number of times each product was added to a cart.
* In `purchase_count`, calculate product purchases.
* In the outer query, join all tables and calculate the number of times each product was added to a cart but not purchased.
* Note that `product_category` was added when not asked to make the creation of the second table easier. 

```sql
CREATE TABLE clique_bait.product_info AS (
  WITH page_views AS (
    SELECT
      page_name,
      COUNT(page_name) AS page_views
    FROM clique_bait.events
    JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE page_id BETWEEN 3 AND 11 AND event_type = 1
    GROUP BY page_name
  ),

  cart_add AS (
    SELECT
      page_name,
      COUNT(page_name) AS cart_addition
    FROM clique_bait.events
    JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE page_id BETWEEN 3 AND 11 AND event_type = 2
    GROUP BY page_name
  ),

  purchase_count AS (
    SELECT
      page_name,
      COUNT(page_name) AS purchase_count
    FROM clique_bait.events
    JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE event_type = 2
      AND visit_id IN (SELECT visit_id FROM clique_bait.events WHERE event_type = 3)
    GROUP BY page_name
  )

  SELECT
    product_category,
    page_name,
    page_views,
    cart_addition,
    cart_addition - purchase_count AS cart_with_no_purchase,
    purchase_count
  FROM page_views
  JOIN cart_add USING(page_name)
  JOIN purchase_count USING(page_name)
  JOIN clique_bait.page_hierarchy USING(page_name)
  ORDER BY product_category, product_id
);
```

| product_category | page_name      | page_views | cart_addition | cart_with_no_purchase | purchase_count |
| ---------------- | -------------- | ---------- | ------------- | --------------------- | -------------- |
| Fish             | Salmon         | 1559       | 938           | 227                   | 711            |
| Fish             | Kingfish       | 1559       | 920           | 213                   | 707            |
| Fish             | Tuna           | 1515       | 931           | 234                   | 697            |
| Luxury           | Russian Caviar | 1563       | 946           | 249                   | 697            |
| Luxury           | Black Truffle  | 1469       | 924           | 217                   | 707            |
| Shellfish        | Abalone        | 1525       | 932           | 233                   | 699            |
| Shellfish        | Lobster        | 1547       | 968           | 214                   | 754            |
| Shellfish        | Crab           | 1564       | 949           | 230                   | 719            |
| Shellfish        | Oyster         | 1568       | 943           | 217                   | 726            |

Aggregating further:

* Use **GROUP BY** and **SUM** to aggregate everything by `product_category`.

```sql
CREATE TABLE clique_bait.product_category_info AS (
  SELECT
    product_category, 
    SUM(page_views) AS page_views, 
    SUM(cart_addition) AS cart_addition, 
    SUM(cart_with_no_purchase) AS cart_with_no_purchase, 
    SUM(purchase_count) AS purchase_count
  FROM clique_bait.product_info
  GROUP BY product_category
  ORDER BY product_category
);
```

| product_category | page_views | cart_addition | cart_with_no_purchase | purchase_count |
| ---------------- | ---------- | ------------- | --------------------- | -------------- |
| Fish             | 4633       | 2789          | 674                   | 2115           |
| Luxury           | 3032       | 1870          | 466                   | 1404           |
| Shellfish        | 6204       | 3792          | 894                   | 2898           |

---

## 1. Which product had the most views, cart adds and purchases?

* There are not a lot of products, so we could just look at the table. However, let's imagine that there are a lot of them.

Most views:

```sql
SELECT
  page_name,
  page_views
FROM clique_bait.product_info
ORDER BY page_views DESC
LIMIT 1;
```

| page_name | page_views |
| --------- | ---------- |
| Oyster    | 1568       |

Most cart additions:

```sql
SELECT
  page_name,
  cart_addition
FROM clique_bait.product_info
ORDER BY cart_addition DESC
LIMIT 1;
```

| page_name | cart_addition |
| --------- | ------------- |
| Lobster   | 968           |

Most purchased:

```sql
SELECT
  page_name,
  purchase_count
FROM clique_bait.product_info
ORDER BY purchase_count DESC
LIMIT 1;
```

| page_name | purchase_count |
| --------- | -------------- |
| Lobster   | 754            |

---

## 2. Which product was most likely to be abandoned?

```sql
SELECT
  page_name,
  cart_with_no_purchase FROM clique_bait.product_info
ORDER BY cart_with_no_purchase DESC
LIMIT 1;
```

| page_name      | cart_with_no_purchase |
| -------------- | --------------------- |
| Russian Caviar | 249                   |

---

## 3. Which product had the highest view to purchase percentage?

```sql
SELECT
  page_name,
  ROUND(purchase_count*100::numeric/page_views,2) AS view_to_purchase_percentage
FROM clique_bait.product_info
ORDER BY view_to_purchase_percentage DESC
LIMIT 1;
```

| page_name | view_to_purchase_percentage |
| --------- | --------------------------- |
| Lobster   | 48.74                       |

---

## 4. What is the average conversion rate from view to cart add?

```sql
SELECT
  ROUND(AVG(cart_addition*100::numeric/page_views),2) AS view_to_cart_rate
FROM clique_bait.product_info;
```

| view_to_cart_rate |
| ----------------- |
| 60.95             |

---

## 5. What is the average conversion rate from cart add to purchase?

```sql
SELECT
  ROUND(AVG(purchase_count*100::numeric/cart_addition),2) AS cart_to_purchase_rate
FROM clique_bait.product_info;
```

| cart_to_purchase_rate |
| --------------------- |
| 75.93                 |

---
