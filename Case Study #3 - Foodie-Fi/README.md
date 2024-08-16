# Case Study #3 - Foodie-Fi 🍕

<p align="center">
<img src="https://8weeksqlchallenge.com/images/case-study-designs/3.png" width=50% height=50%>

Case Study [Source](https://8weeksqlchallenge.com/case-study-3/)

---

## Introduction

Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!

Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

---

## Available data

Danny has shared the data design for Foodie-Fi and also short descriptions on each of the database tables - our case study focuses on only 2 tables but there will be a challenge to create a new table for the Foodie-Fi team.

All datasets exist within the foodie_fi database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.

### Table 1: plans

| plan_id | plan_name     | price  |
| ------- | ------------- | ------ |
| 0       | trial         | 0.00   |
| 1       | basic monthly | 9.90   |
| 2       | pro monthly   | 19.90  |
| 3       | pro annual    | 199.00 |
| 4       | churn         |        |

### Table 2: subscriptions

This table has more than 2 thousand rows. Here are the first 10:

| customer_id | plan_id | start_date               |
| ----------- | ------- | ------------------------ |
| 1           | 0       | 2020-08-01T00:00:00.000Z |
| 1           | 1       | 2020-08-08T00:00:00.000Z |
| 2           | 0       | 2020-09-20T00:00:00.000Z |
| 2           | 3       | 2020-09-27T00:00:00.000Z |
| 3           | 0       | 2020-01-13T00:00:00.000Z |
| 3           | 1       | 2020-01-20T00:00:00.000Z |
| 4           | 0       | 2020-01-17T00:00:00.000Z |
| 4           | 1       | 2020-01-24T00:00:00.000Z |
| 4           | 4       | 2020-04-21T00:00:00.000Z |
| 5           | 0       | 2020-08-03T00:00:00.000Z |

---

## Case Study Questions

This case study is split into an initial data understanding question before diving straight into data analysis questions before finishing with 1 single extension challenge.

A. Customer Journey
B. Data Analysis Questions
C. Challenge Payment Question
D. Outside The Box Questions
