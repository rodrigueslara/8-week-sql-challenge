# Foodie-Fi 🥑 - Customer Journer

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/README.md).

---

Based off the 8 sample customers provided from the `subscriptions` table, write a brief description about each customer’s onboarding journey.

| customer_id | plan_id | start_date               |
| ----------- | ------- | ------------------------ |
| 1           | 0       | 2020-08-01T00:00:00.000Z |
| 1           | 1       | 2020-08-08T00:00:00.000Z |
| 2           | 0       | 2020-09-20T00:00:00.000Z |
| 2           | 3       | 2020-09-27T00:00:00.000Z |
| 11          | 0       | 2020-11-19T00:00:00.000Z |
| 11          | 4       | 2020-11-26T00:00:00.000Z |
| 13          | 0       | 2020-12-15T00:00:00.000Z |
| 13          | 1       | 2020-12-22T00:00:00.000Z |
| 13          | 2       | 2021-03-29T00:00:00.000Z |
| 15          | 0       | 2020-03-17T00:00:00.000Z |
| 15          | 2       | 2020-03-24T00:00:00.000Z |
| 15          | 4       | 2020-04-29T00:00:00.000Z |
| 16          | 0       | 2020-05-31T00:00:00.000Z |
| 16          | 1       | 2020-06-07T00:00:00.000Z |
| 16          | 3       | 2020-10-21T00:00:00.000Z |
| 18          | 0       | 2020-07-06T00:00:00.000Z |
| 18          | 2       | 2020-07-13T00:00:00.000Z |
| 19          | 0       | 2020-06-22T00:00:00.000Z |
| 19          | 2       | 2020-06-29T00:00:00.000Z |
| 19          | 3       | 2020-08-29T00:00:00.000Z |

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

By running:

```sql
SELECT
  customer_id,
  start_date,
  plan_name
FROM foodie_fi.subscriptions
JOIN foodie_fi.plans USING(plan_id)
WHERE customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)
ORDER BY customer_id, start_date
```

We get:

| customer_id | start_date               | plan_name     |
| ----------- | ------------------------ | ------------- |
| 1           | 2020-08-01T00:00:00.000Z | trial         |
| 1           | 2020-08-08T00:00:00.000Z | basic monthly |
| 2           | 2020-09-20T00:00:00.000Z | trial         |
| 2           | 2020-09-27T00:00:00.000Z | pro annual    |
| 11          | 2020-11-19T00:00:00.000Z | trial         |
| 11          | 2020-11-26T00:00:00.000Z | churn         |
| 13          | 2020-12-15T00:00:00.000Z | trial         |
| 13          | 2020-12-22T00:00:00.000Z | basic monthly |
| 13          | 2021-03-29T00:00:00.000Z | pro monthly   |
| 15          | 2020-03-17T00:00:00.000Z | trial         |
| 15          | 2020-03-24T00:00:00.000Z | pro monthly   |
| 15          | 2020-04-29T00:00:00.000Z | churn         |
| 16          | 2020-05-31T00:00:00.000Z | trial         |
| 16          | 2020-06-07T00:00:00.000Z | basic monthly |
| 16          | 2020-10-21T00:00:00.000Z | pro annual    |
| 18          | 2020-07-06T00:00:00.000Z | trial         |
| 18          | 2020-07-13T00:00:00.000Z | pro monthly   |
| 19          | 2020-06-22T00:00:00.000Z | trial         |
| 19          | 2020-06-29T00:00:00.000Z | pro monthly   |
| 19          | 2020-08-29T00:00:00.000Z | pro annual    |

### Customer 1

* Customer 1 started their free trial on August 1st;
* After it ended, they chose to continue with the basic subscription.

### Customer 2

* Customer 2 started their free trial on September 20th;
* After it ended, they chose to continue with the pro annual subscription.

### Customer 11

* Customer 11 started their free trial on November 19th;
* After it ended, they chose to cancel their subscription.

### Customer 13

* Customer 13 started their free trial on December 15th;
* After it ended, they chose to continue with the basic subscription;
* On March 29th, they upgraded it to pro monthly.

### Customer 15

* Customer 15 started their free trial on March 17th;
* After it ended, the subscription was upgraded to pro monthly;
* On April 19th, they chose to cancel their subscription;
* Note that after the free trial, unless manually canceled, the subscription will be upgraded to pro monthly. It is possibly what happened to customer 15 - he forgot to cancel it.

### Customer 16

* Customer 16 started their free trial on May 31st;
* After it ended, they chose to continue with the basic subscription;
* On March 29th, they upgraded it to pro annual.

### Customer 18

* Customer 18 started their free trial on July 6th;
* After it ended, the subscription was upgraded to pro monthly.

### Customer 19

* Customer 19 started their free trial on June 22nd;
* After it ended, the subscription was upgraded to pro monthly;
* On August 29th, they upgraded it to pro annual.

---

Check out solutions for the next section - [Data Analysis Questions](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/B.%20Data%20Analysis%20Questions.md)
