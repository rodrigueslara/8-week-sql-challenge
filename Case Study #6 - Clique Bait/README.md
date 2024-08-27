# Case Study #6 - Clique Bait 🎣

<p align="center">
<img src="https://8weeksqlchallenge.com/images/case-study-designs/6.png" width=50% height=50%>

Case Study [Source](https://8weeksqlchallenge.com/case-study-6/)

---

## Introduction

Clique Bait is not like your regular online seafood store - the founder and CEO Danny, was also a part of a digital data analytics team and wanted to expand his knowledge into the seafood industry!

In this case study - you are required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.

---

## Available Data

For this case study there is a total of 5 datasets which you will need to combine to solve all of the questions.

### Users

Customers who visit the Clique Bait website are tagged via their `cookie_id`.

Only the first 10 rows are shown.

| user_id | cookie_id | start_date               |
| ------- | --------- | ------------------------ |
| 1       | c4ca42    | 2020-02-04T00:00:00.000Z |
| 2       | c81e72    | 2020-01-18T00:00:00.000Z |
| 3       | eccbc8    | 2020-02-21T00:00:00.000Z |
| 4       | a87ff6    | 2020-02-22T00:00:00.000Z |
| 5       | e4da3b    | 2020-02-01T00:00:00.000Z |
| 6       | 167909    | 2020-01-25T00:00:00.000Z |
| 7       | 8f14e4    | 2020-02-09T00:00:00.000Z |
| 8       | c9f0f8    | 2020-02-12T00:00:00.000Z |
| 9       | 45c48c    | 2020-02-07T00:00:00.000Z |
| 10      | d3d944    | 2020-01-23T00:00:00.000Z |

### Events

Customer visits are logged in this `events` table at a `cookie_id` level and the `event_type` and `page_id` values can be used to join onto relevant satellite tables to obtain further information about each event.

The sequence_number is used to order the events within each visit.

Only the first 10 rows are shown.

| visit_id | cookie_id | page_id | event_type | sequence_number | event_time               |
| -------- | --------- | ------- | ---------- | --------------- | ------------------------ |
| ccf365   | c4ca42    | 1       | 1          | 1               | 2020-02-04T19:16:09.182Z |
| ccf365   | c4ca42    | 2       | 1          | 2               | 2020-02-04T19:16:17.358Z |
| ccf365   | c4ca42    | 6       | 1          | 3               | 2020-02-04T19:16:58.454Z |
| ccf365   | c4ca42    | 9       | 1          | 4               | 2020-02-04T19:16:58.609Z |
| ccf365   | c4ca42    | 9       | 2          | 5               | 2020-02-04T19:17:51.729Z |
| ccf365   | c4ca42    | 10      | 1          | 6               | 2020-02-04T19:18:11.605Z |
| ccf365   | c4ca42    | 10      | 2          | 7               | 2020-02-04T19:19:10.570Z |
| ccf365   | c4ca42    | 11      | 1          | 8               | 2020-02-04T19:19:46.911Z |
| ccf365   | c4ca42    | 11      | 2          | 9               | 2020-02-04T19:20:45.274Z |
| ccf365   | c4ca42    | 12      | 1          | 10              | 2020-02-04T19:20:52.307Z |

### Event Identifier 

The `event_identifier` table shows the types of events which are captured by Clique Bait’s digital data systems.

| event_type | event_name    |
| ---------- | ------------- |
| 1          | Page View     |
| 2          | Add to Cart   |
| 3          | Purchase      |
| 4          | Ad Impression |
| 5          | Ad Click      |


### Campaign Identifier

This table shows information for the 3 campaigns that Clique Bait has ran on their website so far in 2020.

| campaign_id | products | campaign_name                     | start_date               | end_date                 |
| ----------- | -------- | --------------------------------- | ------------------------ | ------------------------ |
| 1           | 1-3      | BOGOF - Fishing For Compliments   | 2020-01-01T00:00:00.000Z | 2020-01-14T00:00:00.000Z |
| 2           | 4-5      | 25% Off - Living The Lux Life     | 2020-01-15T00:00:00.000Z | 2020-01-28T00:00:00.000Z |
| 3           | 6-8      | Half Off - Treat Your Shellf(ish) | 2020-02-01T00:00:00.000Z | 2020-03-31T00:00:00.000Z |

### Page Hierarchy

This table lists all of the pages on the Clique Bait website which are tagged and have data passing through from user interaction events.

| page_id | page_name      | product_category | product_id |
| ------- | -------------- | ---------------- | ---------- |
| 1       | Home Page      |                  |            |
| 2       | All Products   |                  |            |
| 3       | Salmon         | Fish             | 1          |
| 4       | Kingfish       | Fish             | 2          |
| 5       | Tuna           | Fish             | 3          |
| 6       | Russian Caviar | Luxury           | 4          |
| 7       | Black Truffle  | Luxury           | 5          |
| 8       | Abalone        | Shellfish        | 6          |
| 9       | Lobster        | Shellfish        | 7          |
| 10      | Crab           | Shellfish        | 8          |
| 11      | Oyster         | Shellfish        | 9          |
| 12      | Checkout       |                  |            |
| 13      | Confirmation   |                  |            |

Blanks are null values.

---

## Case study questions

1. Enterprise Relationship Diagram
2. Digital Analysis
3. Product Funnel Analysis
4. Campaigns Analysis
      
