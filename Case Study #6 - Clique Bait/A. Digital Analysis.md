# Clique Bait 🎣 - Digital Analysis - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%236%20-%20Clique%20Bait/README.md).

--- 

## Digital Analysis

1. How many users are there?
2. How many cookies does each user have on average?
3. What is the unique number of visits by all users per month?
4. What is the number of events for each event type?
5. What is the percentage of visits which have a purchase event?
6. What is the percentage of visits which view the checkout page but do not have a purchase event?
7. What are the top 3 pages by number of views?
8. What is the number of views and cart adds for each product category?
9. What are the top 3 products by purchases?

---

## 1. How many users are there?

* Use **COUNT** and **DISTINCT** to count the number of users.

```sql
SELECT
  COUNT(DISTINCT user_id) AS count_users
FROM clique_bait.users;
```

### Answer

| count |
| ----- |
| 500   |

---

## 2. How many cookies does each user have on average?

* In cte_user, count the number of cookis per person.
* In the outer query, calculate the average.

```sql
WITH cte_user AS (
  SELECT user_id,
  COUNT(user_id) AS count_cookies
FROM clique_bait.users
GROUP BY user_id
)

SELECT
  ROUND(AVG(count_cookies),2) AS avg_cookies
FROM cte_user;
```

### Answer

| avg_cookies |
| ----------- |
| 3.56        |

---

## 3. What is the unique number of visits by all users per month?

* Use **TO_CHAR** to get the month.
* Use **COUNT** and **GROUP BY** to calculate the number of unique visits grouped by month.
  
```sql
SELECT
  TO_CHAR(event_time, 'MM') AS month_number,
  TO_CHAR(event_time, 'Month') AS month_visit,
  COUNT(DISTINCT visit_id) AS unique_visits
FROM clique_bait.events
GROUP BY month_visit, month_number
ORDER BY month_number;
```

### Answer

| month_number | month_visit | unique_visits |
| ------------ | ----------- | ------------- |
| 01           | January     | 876           |
| 02           | February    | 1488          |
| 03           | March       | 916           |
| 04           | April       | 248           |
| 05           | May         | 36            |

---

## 4. What is the number of events for each event type?

* To get the event name, **JOIN** `event_identifier`.
* Use **COUNT** and **GROUP BY** to calculate the number of events grouped by event type.

```sql
SELECT
  event_name,
  COUNT(*) AS event_count
FROM clique_bait.events
JOIN clique_bait.event_identifier USING(event_type)
GROUP BY event_name
ORDER BY event_count DESC;
```

### Answer

| event_name    | event_count |
| ------------- | ----------- |
| Page View     | 20928       |
| Add to Cart   | 8451        |
| Purchase      | 1777        |
| Ad Impression | 876         |
| Ad Click      | 702         |

---

## 5. What is the percentage of visits which have a purchase event?

* Use **WHERE** since we are only interested in `Purchase` events.
* Use **COUNT** and calculate the percentage of visits that have a purchase event.

```sql
SELECT
  ROUND(
    COUNT(DISTINCT visit_id)::numeric /
    (SELECT COUNT(DISTINCT visit_id) FROM clique_bait.events)
    * 100
  ,2) AS percentage_purchase
FROM clique_bait.events
JOIN clique_bait.event_identifier USING(event_type)
WHERE event_name = 'Purchase';
```

### Answer

| percentage_purchase |
| ------------------- |
| 49.86               |

---

## 6. What is the percentage of visits which view the checkout page but do not have a purchase event?

* This percentage is calculated by counting the number of visits where the checkout page was viewed but there is not a purchase event, and dividing that by the total number of visits.
* To count the first, we'll look at the events and hierarchy tables: note that what we want is to count visits where `page_id` 12 appears, but 13 doesn't.
* In `visit`, we get the max value for `page_id` grouped by `visit_id`. The number of visits where the checkout page was viewed but not the purchase page is the number of rows where the max is 12 (checkout).
* In the outer query, calculate the percentage.

```sql
WITH visit AS (
  SELECT
    visit_id,
    MAX(page_id) AS max_page
  FROM clique_bait.events
  GROUP BY visit_id
  ORDER BY visit_id
)

SELECT
  ROUND(
    COUNT(max_page)::numeric/
    (SELECT COUNT(DISTINCT visit_id) FROM clique_bait.events)
    *100
  ,2) AS percentage_view_no_purchase
FROM visit
WHERE max_page = 12;
```

### Answer

| percentage_view_no_purchase |
| --------------------------- |
| 9.15                        |

---

## 7. What are the top 3 pages by number of views?

* Use **WHERE** to get only the views event.
* Use **COUNT** and **GROUP BY** to calculate the number of views grouped by page.
* Use **ORDER BY** and **LIMIT** to get the top 3.

```sql
SELECT
  page_name,
  COUNT(page_id) AS page_views
FROM clique_bait.events
JOIN clique_bait.page_hierarchy USING(page_id)
WHERE event_type = 1
GROUP BY page_name
ORDER BY page_views DESC
LIMIT 3;
```

### Answer

| page_name    | page_views |
| ------------ | ---------- |
| All Products | 3174       |
| Checkout     | 2103       |
| Home Page    | 1782       |

---

## 8. What is the number of views and cart adds for each product category?

* In `view_product`, calculate the number of views for each product category.
* In `cart_product`, calculate the number of cart additions.
* In the outer query, join both tables.
 
```sql
WITH view_product AS (
  SELECT
    product_category,
    COUNT(page_name) AS view_count
  FROM clique_bait.events
  JOIN clique_bait.page_hierarchy USING(page_id)
  WHERE event_type = 1
  GROUP BY product_category
),

cart_product AS (
  SELECT
    product_category,
    COUNT(page_name) AS cart_count
  FROM clique_bait.events
  JOIN clique_bait.page_hierarchy USING(page_id)
  WHERE event_type = 2
  GROUP BY product_category
)

SELECT
  product_category,
  view_count,
  cart_count
FROM view_product
JOIN cart_product USING(product_category)
ORDER BY product_category;
```

### Answer

| product_category | view_count | cart_count |
| ---------------- | ---------- | ---------- |
| Fish             | 4633       | 2789       |
| Luxury           | 3032       | 1870       |
| Shellfish        | 6204       | 3792       |

---

## 9. What are the top 3 products by purchases?

* In `purchase_cte`, get all `visit_id`'s where purchases are made.
* In the outer query, use **WHERE** to get rows where purchases are made and **SELECT** the page name when the product is added to the cart.
* Use **COUNT** and **GROUP BY** to count the number of purchases by product.
* Use **ORDER BY** and **LIMIT** to get the top 3.

```sql
WITH purchases_cte AS (
  SELECT 
    DISTINCT visit_id
  FROM clique_bait.events
  WHERE page_id = 13 
)

SELECT 
  page_name,
  COUNT(
    CASE
      WHEN event_type = 2 THEN visit_id
      ELSE 0
    END
  ) AS purchase_count
FROM clique_bait.events
JOIN clique_bait.page_hierarchy USING(page_id)
WHERE visit_id IN (SELECT * FROM purchases_cte) AND event_type = 2
GROUP BY page_name
ORDER BY purchase_count DESC
LIMIT 3;
```

### Answer

| page_name | purchase_count |
| --------- | -------------- |
| Lobster   | 754            |
| Oyster    | 726            |
| Crab      | 719            |

---

Check out the solutions for the next section [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%236%20-%20Clique%20Bait/B.%20Product%20Funnel%20Analysis.md)
