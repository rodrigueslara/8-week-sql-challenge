# Clique Bait 🎣 - Campaigns Analysis

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%236%20-%20Clique%20Bait/README.md).

--- 

## Campaigns Analysis 

Generate a table that has 1 single row for every unique visit_id record and has the following columns:

* user_id
* visit_id
* visit_start_time: the earliest event_time for each visit
* page_views: count of page views for each visit
* cart_adds: count of product cart add events for each visit
* purchase: 1/0 flag if a purchase event exists for each visit
* campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
* impression: count of ad impressions for each visit
* click: count of ad clicks for each visit
* (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

### Answer

```sql
SELECT
  user_id,
  visit_id,
  MIN(event_time) AS visit_start_time,
  SUM(
    CASE
      WHEN event_type = 1 THEN 1
      ELSE 0 END
  ) AS page_views,
  SUM(
    CASE
      WHEN event_type = 2 THEN 1
      ELSE 0 END
  ) AS cart_adds,
  SUM(
    CASE WHEN event_type = 3 THEN 1
    ELSE 0 END
  ) AS purchase,
  campaign_name,
  SUM(
    CASE
      WHEN event_type = 4 THEN 1
      ELSE 0 END
  ) AS impression,
  SUM(
    CASE
      WHEN event_type = 5 THEN 1
      ELSE 0 END
  ) AS click,
  STRING_AGG(
    CASE WHEN event_type = 2 THEN page_name END, ', ' ORDER BY sequence_number
  ) AS cart_products
FROM clique_bait.events
JOIN clique_bait.users USING(cookie_id)
JOIN clique_bait.campaign_identifier
  ON event_time BETWEEN clique_bait.campaign_identifier.start_date AND clique_bait.campaign_identifier.end_date
JOIN clique_bait.page_hierarchy USING(page_id)	 
GROUP BY user_id, visit_id, campaign_name
ORDER BY user_id, visit_start_time;
```

| user_id | visit_id | visit_start_time         | page_views | cart_adds | purchase | campaign_name                     | impression | click | cart_products                                                                         |
| ------- | -------- | ------------------------ | ---------- | --------- | -------- | --------------------------------- | ---------- | ----- | ------------------------------------------------------------------------------------- |
| 1       | 0fc437   | 2020-02-04T17:49:49.602Z | 10         | 6         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Tuna, Russian Caviar, Black Truffle, Abalone, Crab, Oyster                            |
| 1       | ccf365   | 2020-02-04T19:16:09.182Z | 7          | 3         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster, Crab, Oyster                                                                 |
| 1       | 0826dc   | 2020-02-26T05:58:37.918Z | 1          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 1       | 02a5d5   | 2020-02-26T16:57:26.260Z | 4          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 1       | f7c798   | 2020-03-15T02:23:26.312Z | 9          | 3         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Russian Caviar, Crab, Oyster                                                          |
| 1       | 30b94d   | 2020-03-15T13:12:54.023Z | 9          | 7         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Abalone, Lobster, Crab                        |
| 1       | 41355d   | 2020-03-25T00:11:17.860Z | 6          | 1         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster                                                                               |
| 1       | eaffde   | 2020-03-25T20:06:32.342Z | 10         | 8         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster           |
| 2       | 3b5871   | 2020-01-18T10:16:32.158Z | 9          | 6         | 1        | 25% Off - Living The Lux Life     | 1          | 1     | Salmon, Kingfish, Russian Caviar, Black Truffle, Lobster, Oyster                      |
| 2       | c5c0ee   | 2020-01-18T10:35:22.765Z | 1          | 0         | 0        | 25% Off - Living The Lux Life     | 0          | 0     |                                                                                       |
| 2       | e26a84   | 2020-01-18T16:06:40.907Z | 6          | 2         | 1        | 25% Off - Living The Lux Life     | 0          | 0     | Salmon, Oyster                                                                        |
| 2       | d58cbd   | 2020-01-18T23:40:54.761Z | 8          | 4         | 0        | 25% Off - Living The Lux Life     | 0          | 0     | Kingfish, Tuna, Abalone, Crab                                                         |
| 2       | 910d9a   | 2020-02-01T10:40:46.875Z | 8          | 1         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Abalone                                                                               |
| 2       | 1f1198   | 2020-02-01T21:51:55.078Z | 1          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 2       | 49d73d   | 2020-02-16T06:21:27.138Z | 11         | 9         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster |
