# Fresh Segments 🍊 - Interest Analysis - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%238%20-%20Fresh%20Segments/README.md).

--- 

## Interest Analysis

1. Which interests have been present in all `month_year` dates in our dataset?
2. Using this same `total_months` measure - calculate the cumulative percentage of all records starting at 14 months - which `total_months` value passes the 90% cumulative percentage value?
3. If we were to remove all `interest_id` values which are lower than the `total_months` value we found in the previous question - how many total data points would we be removing?

---

## 1. Which interests have been present in all `month_year` dates in our dataset?

* Use **GROUP BY** to group results by interest name.
* Use **HAVING** to select only interests that have the number of **DISTINCT** `month_year` values equal to the number of total **DISTINCT** `month_year` values.

```sql
SELECT
  interest_name
FROM fresh_segments.interest_metrics
JOIN fresh_segments.interest_map
  ON id = interest_id::integer
GROUP BY interest_name
HAVING COUNT(DISTINCT month_year) =
  (SELECT COUNT(DISTINCT month_year) FROM fresh_segments.interest_metrics);
```

### Answer

There are 480 interests in such conditions. Here are the first 10:

| interest_name                                     |
| ------------------------------------------------- |
| Accounting & CPA Continuing Education Researchers |
| Affordable Hotel Bookers                          |
| Aftermarket Accessories Shoppers                  |
| Alabama Trip Planners                             |
| Alaskan Cruise Planners                           |
| Alzheimer and Dementia Researchers                |
| Anesthesiologists                                 |
| Apartment Furniture Shoppers                      |
| Apartment Hunters                                 |
| Apple Fans                                        |

---

## 2. Using this same `total_months` measure - calculate the cumulative percentage of all records starting at 14 months - which `total_months` value passes the 90% cumulative percentage value?

* In `interest_month`, calculate the total months for interest.
* In `cte_count`, for each number of months, calculate the number of interests.
* In the outer query, calculate the cumulative percentage.

```sql
WITH interest_month AS (
  SELECT
    interest_name,
    COUNT(DISTINCT month_year) AS in_month
  FROM fresh_segments.interest_metrics
  JOIN fresh_segments.interest_map
    ON id = interest_id::integer
  GROUP BY interest_name
),

cte_count AS (
  SELECT
    in_month,
    COUNT(in_month) AS interest_count
  FROM interest_month
  GROUP BY in_month
  ORDER BY in_month DESC
)

SELECT
  *,
  ROUND(
    SUM(interest_count) OVER(ORDER BY in_month DESC)::numeric
    /(SUM(interest_count) OVER()) 
    *100
  ,2) AS cumulative_percentage
FROM cte_count;
```

### Answer

| in_month | interest_count | cumulative_percentage |
| -------- | -------------- | --------------------- |
| 14       | 480            | 39.97                 |
| 13       | 82             | 46.79                 |
| 12       | 65             | 52.21                 |
| 11       | 94             | 60.03                 |
| 10       | 86             | 67.19                 |
| 9        | 95             | 75.10                 |
| 8        | 67             | 80.68                 |
| 7        | 90             | 88.18                 |
| 6        | 33             | 90.92                 |
| 5        | 38             | 94.09                 |
| 4        | 31             | 96.67                 |
| 3        | 15             | 97.92                 |
| 2        | 12             | 98.92                 |
| 1        | 13             | 100.00                |

---

## 3. If we were to remove all `interest_id` values which are lower than the `total_months` value we found in the previous question - how many total data points would we be removing?

We'll use `interest_month` and `cte_count` from the previous exercise to help. As well as `cumulative_percentage` as:

```sql
cumulative_percentage AS (
SELECT
  *,
  ROUND(
    SUM(interest_count) OVER(ORDER BY in_month DESC)::numeric
    /(SUM(interest_count) OVER()) 
    *100
  ,2) AS cumulative_percentage
FROM cte_count
)
```

With these, use **SUM** to calculate the number of interests that would be removed:

```sql
SELECT
  SUM(interest_count) AS interest_lost_count
FROM cumulative_percentage
WHERE cumulative_percentage > 90;
```

### Answer

| interest_lost_count |
| ------------------- |
| 142                 |

