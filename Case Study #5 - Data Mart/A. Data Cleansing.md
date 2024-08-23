# Data Mart 💰 - Data Cleansing

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/README.md).

In a single query, perform the following operations and generate a new table in the `data_mart` schema named `clean_weekly_sales`:

* Convert the `week_date` to a DATE format
* Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
* Add a `month_number` with the calendar month for each `week_date` value as the 3rd column
* Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values
* Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the `segment` value:
  * 1 -> Young Adults
  * 2 -> Middle Aged
  * 3 or 4 -> Retirees
* Add a new `demographic` column using the following mapping for the first letter in the `segment` values:
  * C -> Couples
  * F -> Families
* Ensure all `null` string values with an `"unknown"` string value in the original `segment` column as well as the new `age_band` and `demographic` columns
* Generate a new `avg_transaction` column as the `sales` value divided by `transactions` rounded to 2 decimal places for each record

## Answer

```sql
CREATE TABLE data_mart.clean_weekly_sales AS (

  WITH cte_date AS ( 
    SELECT
      TO_DATE(week_date, 'DD/MM/YY') AS week_date,
      region,
      platform,
      segment,
      customer_type,
      transactions,
      sales
    FROM data_mart.weekly_sales
  )

  SELECT
    week_date,
    TO_CHAR(week_date, 'WW') AS week_number,
    TO_CHAR(week_date, 'MM') AS month_number,
    TO_CHAR(week_date, 'YYYY') AS calendar_year,
    region,
    platform,
    CASE
      WHEN segment LIKE 'null' THEN 'unknown'
      ELSE segment
    END AS segment, 
    CASE
      WHEN segment LIKE '%1' THEN 'Young Adults'
      WHEN segment LIKE '%2' THEN 'Middle Aged'
      WHEN segment LIKE '%3' THEN 'Retirees'
      WHEN segment LIKE '%4' THEN 'Retirees' 
      ELSE 'unknown' 
    END AS age_band,
    CASE
      WHEN segment LIKE 'C%' THEN 'Couples'
      WHEN segment LIKE 'F%' THEN 'Families'
      ELSE 'unknown'
    END AS demographic,
    customer_type,
    ROUND(sales::numeric/transactions,2) AS avg_transaction
  FROM cte_date

);
```

Here are the first 10 rows:

| week_date                | week_number | month_number | calendar_year | region | platform | segment | age_band     | demographic | customer_type | avg_transaction |
| ------------------------ | ----------- | ------------ | ------------- | ------ | -------- | ------- | ------------ | ----------- | ------------- | --------------- |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | ASIA   | Retail   | C3      | Retirees     | Couples     | New           | 30.31           |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | ASIA   | Retail   | F1      | Young Adults | Families    | New           | 31.56           |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | USA    | Retail   | unknown | unknown      | unkownn     | Guest         | 31.20           |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | EUROPE | Retail   | C1      | Young Adults | Couples     | New           | 31.42           |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | AFRICA | Retail   | C2      | Middle Aged  | Couples     | New           | 30.29           |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | CANADA | Shopify  | F2      | Middle Aged  | Families    | Existing      | 182.54          |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | AFRICA | Shopify  | F3      | Retirees     | Families    | Existing      | 206.64          |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | ASIA   | Shopify  | F1      | Young Adults | Families    | Existing      | 172.11          |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | AFRICA | Shopify  | F2      | Middle Aged  | Families    | New           | 155.84          |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | AFRICA | Retail   | C3      | Retirees     | Couples     | New           | 35.02           |

---

Check out the solutions for the next section - [Data Exploration](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/B.%20Data%20Exploration.md)
