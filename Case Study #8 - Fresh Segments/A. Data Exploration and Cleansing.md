# Fresh Segments 🍊 - Data Exploration and Cleansing - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%238%20-%20Fresh%20Segments/README.md).

--- 

## Data Exploration and Cleansing

1. Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a date data type with the start of the month.
2. What is count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest) with the null values appearing first?
3. What do you think we should do with these null values in the `fresh_segments.interest_metrics`?
4. How many `interest_id` values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table? What about the other way around?
5. Summarise the `id` values in the `fresh_segments.interest_map` by its total record count in this table.
6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where `interest_id = 21246` in your joined output and include all columns from `fresh_segments.interest_metrics` and all columns from `fresh_segments.interest_map` except from the `id` column.
7. Are there any records in your joined table where the `month_year` value is before the `created_at` value from the `fresh_segments.interest_map` table? Do you think these values are valid and why?

---

## 1. Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a date data type with the start of the month.

* Use **TO_DATE** to alter the data type.

```sql
ALTER TABLE fresh_segments.interest_metrics
  ALTER COLUMN month_year TYPE date USING TO_DATE(month_year, 'MM-YYYY');
```

### Answer

Here are the first 10 rows:

| _month | _year | month_year               | interest_id | composition | index_value | ranking | percentile_ranking |
| ------ | ----- | ------------------------ | ----------- | ----------- | ----------- | ------- | ------------------ |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 32486       | 11.89       | 6.19        | 1       | 99.86              |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 6106        | 9.93        | 5.31        | 2       | 99.73              |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 18923       | 10.85       | 5.29        | 3       | 99.59              |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 6344        | 10.32       | 5.1         | 4       | 99.45              |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 100         | 10.77       | 5.04        | 5       | 99.31              |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 69          | 10.82       | 5.03        | 6       | 99.18              |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 79          | 11.21       | 4.97        | 7       | 99.04              |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 6111        | 10.71       | 4.83        | 8       | 98.9               |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 6214        | 9.71        | 4.83        | 8       | 98.9               |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 19422       | 10.11       | 4.81        | 10      | 98.63              |

---

## 2. What is the count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest) with the null values appearing first?

* Use **COUNT** and **GROUP BY** to calculate the number of records.
* Use **ORDER BY** and **NULLS FIRST** to sort rows by chronological order with null value appearing first.

```sql
SELECT
  month_year,
  COUNT(*) AS total_records
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST;
```

### Answer

| month_year               | total_records |
| ------------------------ | ------------- |
| null                     | 1194          |
| 2018-07-01T00:00:00.000Z | 729           |
| 2018-08-01T00:00:00.000Z | 767           |
| 2018-09-01T00:00:00.000Z | 780           |
| 2018-10-01T00:00:00.000Z | 857           |
| 2018-11-01T00:00:00.000Z | 928           |
| 2018-12-01T00:00:00.000Z | 995           |
| 2019-01-01T00:00:00.000Z | 973           |
| 2019-02-01T00:00:00.000Z | 1121          |
| 2019-03-01T00:00:00.000Z | 1136          |
| 2019-04-01T00:00:00.000Z | 1099          |
| 2019-05-01T00:00:00.000Z | 857           |
| 2019-06-01T00:00:00.000Z | 824           |
| 2019-07-01T00:00:00.000Z | 864           |
| 2019-08-01T00:00:00.000Z | 1149          |

---

## 3. What do you think we should do with these null values in the `fresh_segments.interest_metrics`?

If the null value is in the `interest_id` column, then it doesn't make sense to keep the row, since there wouldn't be a link to the interest map table. So let's delete those rows.

```sql
DELETE FROM fresh_segments.interest_metrics
  WHERE interest_id IS NULL;
```

If we check, we will see that there's only one row left that has null values - `interest_id = 21246`.

---

## 4. How many `interest_id` values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table? What about the other way around?

* In `id_in_metrics`, calculate the number of id's in metrics but not in map.
* In `id_in_map`, calculate the number of id's in map but not metrics.
* In the outer query, **SELECT** the columns from both tables.

```sql
WITH id_in_metrics AS (
  SELECT
    COUNT(interest_id) AS id_metrics_not_map
  FROM fresh_segments.interest_metrics
  WHERE interest_id NOT IN (SELECT interest_id FROM fresh_segments.interest_map)
),

id_in_map AS (
  SELECT
    COUNT(id) AS id_map_not_metrics
  FROM fresh_segments.interest_map
  WHERE id::text NOT IN (SELECT interest_id FROM fresh_segments.interest_metrics)
)

SELECT * FROM id_in_metrics,id_in_map;
```

### Answer

| id_metrics_not_map | id_map_not_metrics |
| ------------------ | ------------------ |
| 0                  | 7                  |

---

## 5. Summarise the `id` values in the `fresh_segments.interest_map` by its total record count in this table.

* Use **COUNT** to calculate the total number of id's in `interest_map`.

```sql
SELECT
  COUNT(id) AS total_id
FROM fresh_segments.interest_map;
```

### Answer

| total_id |
| -------- |
| 1209     |

---

## 6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where `interest_id = 21246` in your joined output and include all columns from `fresh_segments.interest_metrics` and all columns from `fresh_segments.interest_map` except from the `id` column.

* Since there aren't any id's in metrics that aren't in maps (from question 4), a `LEFT JOIN interest_map` would be better.

```sql
SELECT
  _month,
  _year,
  month_year,
  interest_id,
  composition,
  index_value,
  ranking,
  percentile_ranking,
  interest_name, 
  interest_summary,
  created_at,
  last_modified
FROM fresh_segments.interest_metrics
LEFT JOIN fresh_segments.interest_map
  ON id::text = interest_id
WHERE interest_id = '21246'
ORDER BY month_year;
```

### Answer

| _month | _year | month_year               | interest_id | composition | index_value | ranking | percentile_ranking | interest_name                    | interest_summary                                      | created_at               | last_modified            |
| ------ | ----- | ------------------------ | ----------- | ----------- | ----------- | ------- | ------------------ | -------------------------------- | ----------------------------------------------------- | ------------------------ | ------------------------ |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 21246       | 2.26        | 0.65        | 722     | 0.96               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 8      | 2018  | 2018-08-01T00:00:00.000Z | 21246       | 2.13        | 0.59        | 765     | 0.26               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 9      | 2018  | 2018-09-01T00:00:00.000Z | 21246       | 2.06        | 0.61        | 774     | 0.77               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 10     | 2018  | 2018-10-01T00:00:00.000Z | 21246       | 1.74        | 0.58        | 855     | 0.23               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 11     | 2018  | 2018-11-01T00:00:00.000Z | 21246       | 2.25        | 0.78        | 908     | 2.16               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 12     | 2018  | 2018-12-01T00:00:00.000Z | 21246       | 1.97        | 0.7         | 983     | 1.21               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 1      | 2019  | 2019-01-01T00:00:00.000Z | 21246       | 2.05        | 0.76        | 954     | 1.95               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 2      | 2019  | 2019-02-01T00:00:00.000Z | 21246       | 1.84        | 0.68        | 1109    | 1.07               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 3      | 2019  | 2019-03-01T00:00:00.000Z | 21246       | 1.75        | 0.67        | 1123    | 1.14               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 4      | 2019  | 2019-04-01T00:00:00.000Z | 21246       | 1.58        | 0.63        | 1092    | 0.64               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
|        |       |                          | 21246       | 1.61        | 0.68        | 1191    | 0.25               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |

---

## 7. Are there any records in your joined table where the `month_year` value is before the `created_at` value from the `fresh_segments.interest_map` table? Do you think these values are valid and why?

* Let's check how many rows have a `month_year` value before its `created_at`.

```sql
SELECT
  COUNT(*) AS month_year_before_created_at
FROM fresh_segments.interest_metrics
JOIN fresh_segments.interest_map
  ON id::text = interest_id
WHERE month_year < created_at;
```

| month_year_before_created_at |
| ---------------------------- |
| 188                          |

While there are 188 rows where that happens, note that that in itself doesn't mean much since `month_year` is set to the first of the month. So let's check instead if there are any rows where the `month_year` value is before the `created_at` AND they are not from the same month.

```sql
SELECT
  COUNT(*) AS invalid
FROM fresh_segments.interest_metrics
JOIN fresh_segments.interest_map
  ON id::text = interest_id
WHERE month_year < DATE_TRUNC('month', created_at);
```

| invalid |
| ------- |
| 0       |

There aren't any. These values are all valid.

---

Check out the solutions for the next section [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%238%20-%20Fresh%20Segments/B.%20Interest%20Analysis.md)
