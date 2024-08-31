# Balanced Tree Clothing Co. 🌲 - High Level Sales Analysis - Questions and Solutions

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./README.md).

--- 

## High Level Sales Analysis

1. What was the total quantity sold for all products?
2. What is the total generated revenue for all products before discounts?
3. What was the total discount amount for all products?

---

## 1. What was the total quantity sold for all products?

* Use **SUM** to calculate the total quantity of products sold.

```sql
SELECT
  SUM(qty) AS total_sold
FROM balanced_tree.sales
```

### Answer

| total_sold |
| ---------- |
| 45216      |

---

## 2. What is the total generated revenue for all products before discounts?

* Use **SUM** to calculate the total revenue before discounts.

```sql
SELECT
  SUM(price * qty) AS revenue_no_discount
FROM balanced_tree.sales
```

### Answer

| revenue_no_discount |
| ------------------- |
| 1289453             |

---

## 3. What was the total discount amount for all products?

* Use **SUM** to calculate the total discount amount.

```sql
SELECT
  ROUND(SUM(qty*price*discount::numeric/100),2) AS total_discount
FROM balanced_tree.sales
```

### Answer

| total_discount |
| -------------- |
| 156229.14      |

---

Check out the solutions for the next section [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./B.%20Transaction%20Analysis.md)
