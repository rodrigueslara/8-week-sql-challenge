# Case Study #7 - Balanced Tree Clothing Co. 🌲

<p align="center">
<img src="https://8weeksqlchallenge.com/images/case-study-designs/7.png" width=50% height=50%>

Case Study [Source](https://8weeksqlchallenge.com/case-study-7/)

---

## Introduction

Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!

Danny, the CEO of this trendy fashion company has asked you to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.

---

## Available Data

For this case study there is a total of 4 datasets for this case study - however you will only need to utilise 2 main tables to solve all of the regular questions, and the additional 2 tables are used only for the bonus challenge question!

### Product Details

`balanced_tree.product_details` includes all information about the entire range that Balanced Clothing sells in their store.

Here are the first 10 rows:

| product_id | price | product_name                  | category_id | segment_id | style_id | category_name | segment_name | style_name     |
| ---------- | ----- | ----------------------------- | ----------- | ---------- | -------- | ------------- | ------------ | -------------- |
| c4a632     | 13    | Navy Oversized Jeans - Womens | 1           | 3          | 7        | Womens        | Jeans        | Navy Oversized |
| e83aa3     | 32    | Black Straight Jeans - Womens | 1           | 3          | 8        | Womens        | Jeans        | Black Straight |
| e31d39     | 10    | Cream Relaxed Jeans - Womens  | 1           | 3          | 9        | Womens        | Jeans        | Cream Relaxed  |
| d5e9a6     | 23    | Khaki Suit Jacket - Womens    | 1           | 4          | 10       | Womens        | Jacket       | Khaki Suit     |
| 72f5d4     | 19    | Indigo Rain Jacket - Womens   | 1           | 4          | 11       | Womens        | Jacket       | Indigo Rain    |
| 9ec847     | 54    | Grey Fashion Jacket - Womens  | 1           | 4          | 12       | Womens        | Jacket       | Grey Fashion   |
| 5d267b     | 40    | White Tee Shirt - Mens        | 2           | 5          | 13       | Mens          | Shirt        | White Tee      |
| c8d436     | 10    | Teal Button Up Shirt - Mens   | 2           | 5          | 14       | Mens          | Shirt        | Teal Button Up |
| 2a2353     | 57    | Blue Polo Shirt - Mens        | 2           | 5          | 15       | Mens          | Shirt        | Blue Polo      |
| f084eb     | 36    | Navy Solid Socks - Mens       | 2           | 6          | 16       | Mens          | Socks        | Navy Solid     |

### Product Sales

`balanced_tree.sales` contains product level information for all the transactions made for Balanced Tree including quantity, price, percentage discount, member status, a transaction ID and also the transaction timestamp.

Here are the first 10 rows:

| prod_id | qty | price | discount | member | txn_id | start_txn_time           |
| ------- | --- | ----- | -------- | ------ | ------ | ------------------------ |
| c4a632  | 4   | 13    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| 5d267b  | 4   | 40    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| b9a74d  | 4   | 17    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| 2feb6b  | 2   | 29    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| c4a632  | 5   | 13    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
| e31d39  | 2   | 10    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
| 72f5d4  | 3   | 19    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
| 2a2353  | 3   | 57    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
| f084eb  | 3   | 36    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
| c4a632  | 1   | 13    | 21       | false  | ef648d | 2021-01-27T02:18:17.164Z |


### Product Hierarcy & Product Price

These tables are used only for the bonus question where we will use them to recreate the `balanced_tree.product_details` table.


| id  | parent_id | level_text          | level_name |
| --- | --------- | ------------------- | ---------- |
| 1   |           | Womens              | Category   |
| 2   |           | Mens                | Category   |
| 3   | 1         | Jeans               | Segment    |
| 4   | 1         | Jacket              | Segment    |
| 5   | 2         | Shirt               | Segment    |
| 6   | 2         | Socks               | Segment    |
| 7   | 3         | Navy Oversized      | Style      |
| 8   | 3         | Black Straight      | Style      |
| 9   | 3         | Cream Relaxed       | Style      |
| 10  | 4         | Khaki Suit          | Style      |
| 11  | 4         | Indigo Rain         | Style      |
| 12  | 4         | Grey Fashion        | Style      |
| 13  | 5         | White Tee           | Style      |
| 14  | 5         | Teal Button Up      | Style      |
| 15  | 5         | Blue Polo           | Style      |
| 16  | 6         | Navy Solid          | Style      |
| 17  | 6         | White Striped       | Style      |
| 18  | 6         | Pink Fluro Polkadot | Style      |

#### Product Price

| id  | product_id | price |
| --- | ---------- | ----- |
| 7   | c4a632     | 13    |
| 8   | e83aa3     | 32    |
| 9   | e31d39     | 10    |
| 10  | d5e9a6     | 23    |
| 11  | 72f5d4     | 19    |
| 12  | 9ec847     | 54    |
| 13  | 5d267b     | 40    |
| 14  | c8d436     | 10    |
| 15  | 2a2353     | 57    |
| 16  | f084eb     | 36    |
| 17  | b9a74d     | 17    |
| 18  | 2feb6b     | 29    |

## Case Study Questions

A. [High Level Sales Analysis](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./A.%20High%20Level%20Sales%20Analysis.md)

B. [Transaction Analysis](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./B.%20Transaction%20Analysis.md)

C. [Product Analysis](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./C.%20Product%20Analysis.md)

