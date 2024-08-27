# Case Study #5 - Data Mart 💰

<p align="center">
<img src="https://8weeksqlchallenge.com/images/case-study-designs/5.png" width=50% height=50%>

Case Study [Source](https://8weeksqlchallenge.com/case-study-5/)

---

## Introduction

Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh produce - Danny is asking for your support to analyse his sales performance.

In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas.

The key business question he wants you to help him answer are the following:

* What was the quantifiable impact of the changes introduced in June 2020?
* Which platform, region, segment and customer types were the most impacted by this change?
* What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?

---

## Available Data

For this case study there is only a single table: `data_mart.weekly_sales`.

The columns are pretty self-explanatory based on the column names but here are some further details about the dataset:

1. Data Mart has international operations using a multi-`region` strategy;
2. Data Mart has both, a retail and online `platform` in the form of a Shopify store front to serve their customers;
3. Customer `segment` and `customer_type` data relates to personal age and demographics information that is shared with Data Mart;
4. `transactions` is the count of unique purchases made through Data Mart and `sales` is the actual dollar amount of purchases.

Each record in the dataset is related to a specific aggregated slice of the underlying sales data rolled up into a `week_date` value which represents the start of the sales week.

Here are the first 10 rows:

| week_date | region | platform | segment | customer_type | transactions | sales    |
| --------- | ------ | -------- | ------- | ------------- | ------------ | -------- |
| 31/8/20   | ASIA   | Retail   | C3      | New           | 120631       | 3656163  |
| 31/8/20   | ASIA   | Retail   | F1      | New           | 31574        | 996575   |
| 31/8/20   | USA    | Retail   | null    | Guest         | 529151       | 16509610 |
| 31/8/20   | EUROPE | Retail   | C1      | New           | 4517         | 141942   |
| 31/8/20   | AFRICA | Retail   | C2      | New           | 58046        | 1758388  |
| 31/8/20   | CANADA | Shopify  | F2      | Existing      | 1336         | 243878   |
| 31/8/20   | AFRICA | Shopify  | F3      | Existing      | 2514         | 519502   |
| 31/8/20   | ASIA   | Shopify  | F1      | Existing      | 2158         | 371417   |
| 31/8/20   | AFRICA | Shopify  | F2      | New           | 318          | 49557    |
| 31/8/20   | AFRICA | Retail   | C3      | New           | 111032       | 3888162  |

---

## Case Study Questions

A. [Data Cleansing Steps](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/A.%20Data%20Cleansing.md)

B. [Data Exploration](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/B.%20Data%20Exploration.md)

C. [Before & After Analysis](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/C.%20Before%20%26%20After%20Analysis.md)
