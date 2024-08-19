# Foodie-Fi 🥑 - Challenge Payment Question

Information regarding this study can be found [here](https://github.com/rodrigueslara/8-week-sql-challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/README.md).

---

The Foodie-Fi team wants you to create a new `payments table` for the year 2020 that includes amounts paid by each customer in the `subscriptions` table with the following requirements:

* monthly payments always occur on the same day of month as the original `start_date` of any monthly paid plan
* upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
* upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
* once a customer churns they will no longer make payments

Example outputs for this table might look like the following:
