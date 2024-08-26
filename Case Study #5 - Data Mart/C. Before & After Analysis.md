# Data Mart 💰 - Before & After Analysis - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/README.md).

Click [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/A.%20Data%20Cleansing.md) to see what I've done to clean the table.

---

## Before & After Analysis

Taking the `week_date` value of `2020-06-15` as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all `week_date` values for `2020-06-15` as the start of the period after the change and the previous `week_date` values would be before

Using this analysis approach - answer the following questions:

1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
2. What about the entire 12 weeks before and after?
3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

## 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?

* In `cte_sales`, calculate the total sales for both periods.
* In the outer query, **SELECT** all the information.

```sql
WITH cte_sales AS (
  SELECT
    SUM(
      CASE
        WHEN week_date BETWEEN '2020-06-15'::date - 7*4 AND '2020-06-15'::date -7 THEN sales
        ELSE 0
      END
    ) AS sales_before,
    SUM(
      CASE
        WHEN week_date BETWEEN '2020-06-15'::date AND '2020-06-15'::date + 7*3 THEN sales
        ELSE 0
      END
    ) AS sales_after
  FROM data_mart.clean_weekly_sales
)

SELECT
  sales_before,
  sales_after,
  sales_after - sales_before AS reduction_value,
  ROUND(sales_after*100::numeric/sales_before-100,2) AS percentage_reduction
FROM cte_sales;
```

### Answer

| sales_before | sales_after | reduction_value | percentage_reduction |
| ------------ | ----------- | --------------- | -------------------- |
| 2345878357   | 2318994169  | -26884188       | -1.15                |

---

## 2. What about the entire 12 weeks before and after?

* Same as above. The only changes were the dates.

```sql
WITH cte_sales AS (
  SELECT
    SUM(
      CASE
        WHEN week_date BETWEEN '2020-06-15'::date - 7*12 AND '2020-06-15'::date -7 THEN sales
        ELSE 0
      END
    ) AS sales_before,
    SUM(
      CASE
        WHEN week_date BETWEEN '2020-06-15'::date AND '2020-06-15'::date + 7*11 THEN sales
        ELSE 0
      END) AS sales_after
  FROM data_mart.clean_weekly_sales
)

SELECT
  sales_before,
  sales_after,
  sales_after - sales_before AS reduction_value,
  ROUND(sales_after*100::numeric/sales_before-100,2) AS percentage_reduction
FROM cte_sales;
```

### Answer

| sales_before | sales_after | reduction_value | percentage_reduction |
| ------------ | ----------- | --------------- | -------------------- |
| 7126273147   | 6973947753  | -152325394      | -2.14                |

---

### 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

* In `cte_weeks`, I have all the week numbers I need.
* In cte_sales, calculate sales for each period, before and after, grouped by year.
* In the outer query, calculate variations and percentages.

```sql
WITH cte_weeks AS (
  SELECT
    TO_CHAR('2020-06-15'::date,'WW')::numeric AS week_w,
    TO_CHAR('2020-06-15'::date + 7*3,'WW')::numeric AS week_4,
    TO_CHAR('2020-06-15'::date - 7*4,'WW')::numeric AS week_minus4,
    TO_CHAR('2020-06-15'::date -7,'WW' )::numeric AS week_minus1,
    TO_CHAR('2020-06-15'::date + 7*11,'WW')::numeric AS week_12,
    TO_CHAR('2020-06-15'::date - 7*12,'WW')::numeric AS week_minus12
),

cte_sales AS (
  SELECT
    calendar_year, 
    SUM(
      CASE
        WHEN week_number::numeric BETWEEN week_w AND week_4 THEN sales
        ELSE 0
      END
    ) AS sales_4weeks_after,
    SUM(
      CASE
        WHEN week_number::numeric BETWEEN week_minus4 AND week_minus1 THEN sales
        ELSE 0
      END
    ) AS sales_4weeks_before,
    SUM(
      CASE
        WHEN week_number::numeric BETWEEN week_w AND week_12 THEN sales
        ELSE 0
      END
    ) AS sales_12weeks_after,
    SUM(
      CASE
        WHEN week_number::numeric BETWEEN week_minus12 AND week_minus1 THEN sales
        ELSE 0
      END
    ) AS sales_12weeks_before
  FROM data_mart.clean_weekly_sales, cte_weeks
  GROUP BY calendar_year
)

SELECT
  calendar_year,
  sales_4weeks_after - sales_4weeks_before AS change_value_4weeks,
  ROUND(sales_4weeks_after*100::numeric/sales_4weeks_before-100,2) AS percentage_change_4weeks,
  sales_12weeks_after - sales_12weeks_before AS change_value_12weeks,
  ROUND(sales_12weeks_after*100::numeric/sales_12weeks_before-100,2) AS percentage_change_12weeks
FROM cte_sales
ORDER BY calendar_year;
```

### Answer

| calendar_year | change_value_4weeks | percentage_change_4weeks | change_value_12weeks | percentage_change_12weeks |
| ------------- | ------------------- | ------------------------ | -------------------- | ------------------------- |
| 2018          | -3936687            | -0.19                    | 617804389            | 10.54                     |
| 2019          | 2336594             | 0.10                     | -20740294            | -0.30                     |
| 2020          | -26884188           | -1.15                    | -152325394           | -2.14                     |

---
