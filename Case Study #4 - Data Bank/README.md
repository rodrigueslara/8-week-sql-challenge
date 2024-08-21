# Case Study #4 - Data Bank 💲

<p align="center">
<img src="https://8weeksqlchallenge.com/images/case-study-designs/4.png" width=50% height=50%>

Case Study [Source](https://8weeksqlchallenge.com/case-study-4/)

---

## Introduction

There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.

Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

---

## Available Data

### Table 1: Regions

Just like popular cryptocurrency platforms - Data Bank is also run off a network of nodes where both money and data is stored across the globe. In a traditional banking sense - you can think of these nodes as bank branches or stores that exist around the world.

This `regions` table contains the `region_id` and their respective `region_name` values.

| region_id | region_name |
| --------- | ----------- |
| 1         | Australia   |
| 2         | America     |
| 3         | Africa      |
| 4         | Asia        |
| 5         | Europe      |

### Table 2: Customer Nodes

Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data.

This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!

Below is a sample of the top 10 rows of the `data_bank.customer_nodes`.

| customer_id | region_id | node_id | start_date               | end_date                 |
| ----------- | --------- | ------- | ------------------------ | ------------------------ |
| 1           | 3         | 4       | 2020-01-02T00:00:00.000Z | 2020-01-03T00:00:00.000Z |
| 2           | 3         | 5       | 2020-01-03T00:00:00.000Z | 2020-01-17T00:00:00.000Z |
| 3           | 5         | 4       | 2020-01-27T00:00:00.000Z | 2020-02-18T00:00:00.000Z |
| 4           | 5         | 4       | 2020-01-07T00:00:00.000Z | 2020-01-19T00:00:00.000Z |
| 5           | 3         | 3       | 2020-01-15T00:00:00.000Z | 2020-01-23T00:00:00.000Z |
| 6           | 1         | 1       | 2020-01-11T00:00:00.000Z | 2020-02-06T00:00:00.000Z |
| 7           | 2         | 5       | 2020-01-20T00:00:00.000Z | 2020-02-04T00:00:00.000Z |
| 8           | 1         | 2       | 2020-01-15T00:00:00.000Z | 2020-01-28T00:00:00.000Z |
| 9           | 4         | 5       | 2020-01-21T00:00:00.000Z | 2020-01-25T00:00:00.000Z |
| 10          | 3         | 4       | 2020-01-13T00:00:00.000Z | 2020-01-14T00:00:00.000Z |

### Table 3: Customer Transactions

This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

| customer_id | txn_date                 | txn_type | txn_amount |
| ----------- | ------------------------ | -------- | ---------- |
| 429         | 2020-01-21T00:00:00.000Z | deposit  | 82         |
| 155         | 2020-01-10T00:00:00.000Z | deposit  | 712        |
| 398         | 2020-01-01T00:00:00.000Z | deposit  | 196        |
| 255         | 2020-01-14T00:00:00.000Z | deposit  | 563        |
| 185         | 2020-01-29T00:00:00.000Z | deposit  | 626        |
| 309         | 2020-01-13T00:00:00.000Z | deposit  | 995        |
| 312         | 2020-01-20T00:00:00.000Z | deposit  | 485        |
| 376         | 2020-01-03T00:00:00.000Z | deposit  | 706        |
| 188         | 2020-01-13T00:00:00.000Z | deposit  | 601        |
| 138         | 2020-01-11T00:00:00.000Z | deposit  | 520        |

## Case Study Questions

The following case study questions include some general data exploration analysis for the nodes and transactions before diving right into the core business questions and finishes with a challenging final request!

A. Customer Nodes Exploration

B. Customer Transactions

C. Data Allocation Challenge

D. Extra Challenge
