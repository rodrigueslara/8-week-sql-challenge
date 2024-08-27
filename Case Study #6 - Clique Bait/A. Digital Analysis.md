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
ORDER BY month_number
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
ORDER BY event_count DESC
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
WHERE event_name = 'Purchase'
```

### Answer

| percentage_purchase |
| ------------------- |
| 49.86               |

---

## 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
