# Data Bank 💲 - Customer Nodes Exploration - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%234%20-%20Data%20Bank/README.md).

--- 

## Customer Nodes Exploration

1. How many unique nodes are there on the Data Bank system?
2. What is the number of nodes per region?
3. How many customers are allocated to each region?
4. How many days on average are customers reallocated to a different node?
5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

---

## 1. How many unique nodes are there on the Data Bank system?

* Use **COUNT** and **DISTINCT** to calculate the number of unique nodes.

```sql
SELECT
  COUNT(DISTINCT node_id) AS unique_nodes
FROM data_bank.customer_nodes;
```
### Answer

| unique_nodes |
| ------------ |
| 5            |

---

## 2. What is the number of nodes per region?

* Use **COUNT** and **GROUP BY** to count the number of nodes per region.

```sql
SELECT
  region_name,
  COUNT(node_id) AS n_nodes
FROM data_bank.customer_nodes
JOIN data_bank.regions USING(region_id)
GROUP BY region_name
ORDER BY region_name;
```

### Answer

| region_name | n_nodes |
| ----------- | ------- |
| Africa      | 714     |
| America     | 735     |
| Asia        | 665     |
| Australia   | 770     |
| Europe      | 616     |

---

## 3. How many customers are allocated to each region?

* Use **COUNT** and **GROUP BY** to count the number of customers allocated to each region.
* Note that you have to use **DISTINCT** so it won't count the same customer more than once.

```sql
SELECT
  region_name,
  COUNT(DISTINCT customer_id) AS n_customer
FROM data_bank.customer_nodes
JOIN data_bank.regions USING(region_id)
GROUP BY region_name
ORDER BY region_name;
```

### Answer

| region_name | n_customer |
| ----------- | ---------- |
| Africa      | 102        |
| America     | 105        |
| Asia        | 95         |
| Australia   | 110        |
| Europe      | 88         |

---

## 4. How many days on average are customers reallocated to a different node?

* Use **AVG** to calculate the average number of days a customer stays on the node.

```sql
SELECT
  ROUND(AVG(end_date - start_date),2) AS average_days
FROM data_bank.customer_nodes
WHERE end_date != '9999-12-31';
```

### Answer

| average_days |
| ------------ |
| 14.63        |

---

## 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

* Use **percentile_cont** to calculate the pecentiles.
* Use **GROUP BY** to group by region.

```sql
SELECT
  region_name,
  percentile_cont(0.50) WITHIN GROUP (ORDER BY end_date-start_date) AS median,
  percentile_cont(0.80) WITHIN GROUP (ORDER BY end_date-start_date) AS precentile_80,
  percentile_cont(0.95) WITHIN GROUP (ORDER BY end_date-start_date) AS precentile_95
FROM data_bank.customer_nodes
JOIN data_bank.regions USING(region_id)
WHERE end_date != '9999-12-31'
GROUP BY region_name
ORDER BY region_name;
```

### Answer

| region_name | median | precentile_80 | precentile_95 |
| ----------- | ------ | ------------- | ------------- |
| Africa      | 15     | 24            | 28            |
| America     | 15     | 23            | 28            |
| Asia        | 15     | 23            | 28            |
| Australia   | 15     | 23            | 28            |
| Europe      | 15     | 24            | 28            |

---

Check out solutions for the next section - [Customer Transactions](https://github.com/rodrigueslara/8-week-sql-challenge/tree/main/Case%20Study%20%234%20-%20Data%20Bank)
