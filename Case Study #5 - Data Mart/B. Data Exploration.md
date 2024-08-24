# Data Mart 💰 - Data Exploration - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/README.md).

Click [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/A.%20Data%20Cleansing.md) to see what I've done to clean the table.

---

## Data Exploration

1. What day of the week is used for each `week_date` value?
2. What range of week numbers are missing from the dataset?
3. How many total transactions were there for each year in the dataset?
4. What is the total sales for each region for each month?
5. What is the total count of transactions for each platform?
6. What is the percentage of sales for Retail vs Shopify for each month?
7. What is the percentage of sales by demographic for each year in the dataset?
8. Which `age_band` and `demographic` values contribute the most to Retail sales?
9. Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

---

## 1. What day of the week is used for each `week_date` value?

* Use **DISTINCT** and **TO_CHAR** to get the day of the week used.
  
```sql
SELECT
  DISTINCT TO_CHAR(week_date,'Day') AS day_update
FROM data_mart.clean_weekly_sales;
```

### Answer

| day_update |
| ---------- |
| Monday     |

---

## 2. What range of week numbers are missing from the dataset?

```sql
WITH cte_series AS (
  SELECT
    GENERATE_SERIES(1,53)::text AS all_weeks
)

SELECT
  all_weeks
FROM cte_series
LEFT JOIN data_mart.clean_weekly_sales
  ON week_number = all_weeks
WHERE week_number IS null
ORDER BY week_number;
```

### Answer

| all_weeks |
| --------- |
| 1         |
| 2         |
| 3         |
| 4         |
| 5         |
| 6         |
| 7         |
| 8         |
| 9         |
| 10        |
| 11        |
| 37        |
| 38        |
| 39        |
| 40        |
| 41        |
| 42        |
| 43        |
| 44        |
| 45        |
| 46        |
| 47        |
| 48        |
| 49        |
| 50        |
| 51        |
| 52        |
| 53        |

---

## 3. How many total transactions were there for each year in the dataset?

* Use **SUM** and **GROUP BY** to calculate the total transactions for each year.
  
```sql
SELECT
  calendar_year,
  SUM(transactions)
FROM data_mart.clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year;
```

### Answer

| calendar_year | sum       |
| ------------- | --------- |
| 2018          | 346406460 |
| 2019          | 365639285 |
| 2020          | 375813651 |

---

## 4. What is the total sales for each region for each month?

* After noticing from previous exercises that the weeks missing correspond to the first and last months, here's what we can do:
* Use **CASE**, **SUM**, and **GROUP BY** to create a column for each month and group the values by region.
* Note that we could just group it by region and month, but it would make the table a lot harder to read.

```sql
SELECT
  region,
  SUM(CASE
      WHEN month_number = '03' THEN sales
      ELSE 0
      END) AS sales_march,
  SUM(CASE
      WHEN month_number = '04' THEN sales
      ELSE 0
      END) AS sales_mpril,
  SUM(CASE
      WHEN month_number = '05' THEN sales
      ELSE 0
      END) AS sales_may,
  SUM(CASE
      WHEN month_number = '06' THEN sales
      ELSE 0
      END) AS sales_mune,
  SUM(CASE
      WHEN month_number = '07' THEN sales
      ELSE 0
      END) AS sales_july,
  SUM(CASE
      WHEN month_number = '08' THEN sales
      ELSE 0
      END) AS sales_august,
  SUM(CASE
      WHEN month_number = '09' THEN sales
      ELSE 0
      END) AS sales_september
FROM data_mart.clean_weekly_sales
GROUP BY region
ORDER BY region;
```

### Answer

| region        | sales_march | sales_mpril | sales_may  | sales_mune | sales_july | sales_august | sales_september |
| ------------- | ----------- | ----------- | ---------- | ---------- | ---------- | ------------ | --------------- |
| AFRICA        | 567767480   | 1911783504  | 1647244738 | 1767559760 | 1960219710 | 1809596890   | 276320987       |
| ASIA          | 529770793   | 1804628707  | 1526285399 | 1619482889 | 1768844756 | 1663320609   | 252836807       |
| CANADA        | 144634329   | 484552594   | 412378365  | 443846698  | 477134947  | 447073019    | 69067959        |
| EUROPE        | 35337093    | 127334255   | 109338389  | 122813826  | 136757466  | 122102995    | 18877433        |
| OCEANIA       | 783282888   | 2599767620  | 2215657304 | 2371884744 | 2563459400 | 2432313652   | 372465518       |
| SOUTH AMERICA | 71023109    | 238451531   | 201391809  | 218247455  | 235582776  | 221166052    | 34175583        |
| USA           | 225353043   | 759786323   | 655967121  | 703878990  | 760331754  | 712002790    | 110532368       |

---

## 5. What is the total count of transactions for each platform?

* Use **COUNT** and **GROUP BY** to get the total count of transactions for each platform.
  
```sql
SELECT
  platform,
  COUNT(transactions) AS count_transactions
FROM data_mart.clean_weekly_sales
GROUP BY platform;
```

### Answer

| platform | count_transactions |
| -------- | ------------------ |
| Shopify  | 8549               |
| Retail   | 8568               |

---

## 6. What is the percentage of sales for Retail vs Shopify for each month?
