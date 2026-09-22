# September 16 - Pizza Runner
![Case Study 1, Danny's Diner](imgs/caseStudy03.png)

## Business Problem
Danny has been selling monthly and annual subscriptions for his new startup Foodie-Fi, giving customers unlimited access to exclusive food videos from around the world. He wants to know about the performance of his startup.

## Tables and Data Structure
![Table relationship diagram](imgs/relations.png)

## Question and Solution
All queries are written in PostgreSQL.

Copy the solution code and execute on [DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16) to see the results!
***
### Case Study Questions
Part A: Customer Journey
1. Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.
```sql
WITH sample AS (
SELECT
  	s.*,
  	p.plan_name,
  	p.price
FROM
  	subscriptions s
INNER JOIN
  	plans p
  	ON s.plan_id=p.plan_id
WHERE
  	s.customer_id = 1 OR
  	s.customer_id = 2 OR
    s.customer_id = 11 OR
    s.customer_id = 13 OR
    s.customer_id = 15 OR
    s.customer_id = 16 OR
    s.customer_id = 18 OR
    s.customer_id = 19 
)
SELECT 
	*
FROM
	sample
ORDER BY
	customer_id ASC, start_date ASC;
```
Output:
| customer_id | plan_id | start_date | plan_name     | price  |
| ----------- | ------- | ---------- | ------------- | ------ |
| 1           | 0       | 2020-08-01 | trial         | 0.00   |
| 1           | 1       | 2020-08-08 | basic monthly | 9.90   |
| 2           | 0       | 2020-09-20 | trial         | 0.00   |
| 2           | 3       | 2020-09-27 | pro annual    | 199.00 |
| 11          | 0       | 2020-11-19 | trial         | 0.00   |
| 11          | 4       | 2020-11-26 | churn         |        |
| 13          | 0       | 2020-12-15 | trial         | 0.00   |
| 13          | 1       | 2020-12-22 | basic monthly | 9.90   |
| 13          | 2       | 2021-03-29 | pro monthly   | 19.90  |
| 15          | 0       | 2020-03-17 | trial         | 0.00   |
| 15          | 2       | 2020-03-24 | pro monthly   | 19.90  |
| 15          | 4       | 2020-04-29 | churn         |        |
| 16          | 0       | 2020-05-31 | trial         | 0.00   |
| 16          | 1       | 2020-06-07 | basic monthly | 9.90   |
| 16          | 3       | 2020-10-21 | pro annual    | 199.00 |
| 18          | 0       | 2020-07-06 | trial         | 0.00   |
| 18          | 2       | 2020-07-13 | pro monthly   | 19.90  |
| 19          | 0       | 2020-06-22 | trial         | 0.00   |
| 19          | 2       | 2020-06-29 | pro monthly   | 19.90  |
| 19          | 3       | 2020-08-29 | pro annual    | 199.00 |

Almost all customers start with a trial plan. After the week ends, some churn, while some upgrade to higher tiers.
* Customer 1 - Trial => Basic monthly plan
* Customer 2 - Trial => Pro annual plan
* Customer 11 - Trial => Churn
* Customer 13 - Trial => Basic monthly plan
* Customer 15 - Trial => Pro monthly => Churn after one month
* Customer 16 - Trial => Basic monthly plan => Pro annual plan after 4 months
* Customer 18 - Trial => Pro monthly
* Customer 19 - Trial => Pro monthly => Pro Annual
***
Part B: Data Analysis
1. How many customers has Foodie-Fi ever had?
```sql
SELECT COUNT(DISTINCT customer_id) FROM subscriptions;
```
Output:
| count |
| ----- |
| 1000  |

1000 unique customers.
***
2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```sql
SELECT
    CAST(DATE_TRUNC('month', start_date) AS DATE) AS start_month,
    COUNT(*) AS trial_plan_count
FROM
	subscriptions s
INNER JOIN
	plans p
    ON s.plan_id=p.plan_id
WHERE
	s.plan_id = 0 -- 0 is trial
GROUP BY
	start_month
ORDER BY
	start_month ASC;
```
__Explanation:__
* Use DATE_TRUNC on the `start_date` column to obtain a timestamp of the start of the month, and CAST to date format. Name this column `start_month`.
* Count the total number of trial plans and group by `start_month`.

Output:
| start_month | trial_plan_count |
| ----------- | ---------------- |
| 2020-01-01  | 88               |
| 2020-02-01  | 68               |
| 2020-03-01  | 94               |
| 2020-04-01  | 81               |
| 2020-05-01  | 88               |
| 2020-06-01  | 79               |
| 2020-07-01  | 89               |
| 2020-08-01  | 88               |
| 2020-09-01  | 87               |
| 2020-10-01  | 79               |
| 2020-11-01  | 75               |
| 2020-12-01  | 84               |

The monthly distribution is quite even, with all months having 70-90 trial plan subscriptions except Febraury, which has 68.
***
3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
```sql
SELECT
	plan_name,
	COUNT(*) AS count
FROM
	subscriptions s
INNER JOIN
	plans p
    ON s.plan_id=p.plan_id
WHERE 
	EXTRACT(YEAR FROM start_date) > 2020
GROUP BY
	plan_name; 
```
__Explanation:__
* Use EXTRACT() to obtain the year from `start_date`, and filter out rows with a year over 2020 in the WHERE clause.
* Count all the rows and group the result by `plan_name`.

Output:
| plan_name     | count |
| ------------- | ----- |
| pro annual    | 63    |
| churn         | 71    |
| pro monthly   | 60    |
| basic monthly | 8     |

The most frequent plan change in 2021 is churn, but there is a significant amount of customers who upgraded to pro annual and monthly plans. Only a small minority choose the basic monthly plan in 2021.
***
4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
SELECT
	COUNT(*) AS churn_count,
    ROUND(
      CAST(COUNT(*) AS NUMERIC)*100/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 
      1
    ) AS churn_percentage
FROM
	subscriptions s
INNER JOIN
	plans p 
	ON s.plan_id=p.plan_id
WHERE
	plan_name = 'churn';
```
__Explanation:__
* Count all the rows where a customer churned and name it `churn_count`. In a second column, CAST the count as NUMERIC and divide it by the count of 1000 unique customers, converting the result to a percentage. Use ROUND() to set the answer to 1 decimal place.


Output:
| churn_count | churn_percentage |
| ----------- | ---------------- |
| 307         | 30.7             |

307 of 1000 customers churned, 30.7%.
***
5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
WITH base AS (
SELECT 
  	customer_id,
  	plan_id,
  	LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date ASC) AS prev_plan_id
FROM
  	subscriptions s
)
SELECT
	COUNT(*) AS churn_after_trial_count,
    ROUND(
      CAST(COUNT(*) AS NUMERIC)*100/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions),
      0
    ) AS churn_after_trial_percentage
FROM
	base
WHERE
	plan_id = 4 AND -- 4 is churn
    prev_plan_id = 0; -- 0 is trial
```
__Explanation:__
* Create a CTE `base` with a new column prev_plan_id using LAG() to find the customer's previous plan id.
* Count the number of rows and percentage of total customers who picked a churn plan right after a trial.

Output:
| churn_after_trial_count | churn_after_trial_percentage |
| ----------------------- | ---------------------------- |
| 92                      | 9                            |

92 customers churn after the trial, 9% of the total customer count.
***
6. What is the number and percentage of customer plans after their initial free trial?
```sql
SELECT 
	COUNT(*) AS count_plans_after_trial,
    ROUND(
      CAST(COUNT(*) AS NUMERIC)*100/(SELECT COUNT(*) FROM subscriptions),
      0
    ) AS plans_after_trial_percentage
FROM 
	subscriptions s
WHERE 
	plan_id != 0; -- 0 is trial
```
__Explanation:__
* Count the number and calculate the percentage of rows that are not trial plans.

Output:
| count_plans_after_trial | plans_after_trial_percentage |
| ----------------------- | ---------------------------- |
| 1650                    | 62                           |

There are 1650 plans after trial, making up 62% of the total number of plans.
***
7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31
```sql
WITH filter AS (
SELECT
	ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date DESC) AS index,
    s.plan_id,
  	p.plan_name, 
  	s.start_date
FROM
	subscriptions s
INNER JOIN
	plans p 
	ON s.plan_id=p.plan_id
WHERE
	start_date <= '2020-12-31' 
)
SELECT
	plan_id,
    plan_name,
    COUNT(*) AS customer_count,
    ROUND(
      CAST(COUNT(*) AS NUMERIC)*100/(SELECT COUNT(*) FROM filter WHERE index=1),
      1
    ) AS percentage
FROM
	filter
WHERE
	index = 1
GROUP BY
	plan_id, plan_name;
```
__Explanation:__
* Index all rows in a CTE `filter` where only subscriptions with a date in 2020 are selected. Partition by `customer_id` and order by `start_date` descending, so the most recent plan gets index 1.
* Select from the CTE only rows with an index of 1, and count the number and percentage of each plan.

Output:
| plan_id | plan_name     | customer_count | percentage |
| ------- | ------------- | -------------- | ---------- |
| 0       | trial         | 19             | 1.9        |
| 1       | basic monthly | 224            | 22.4       |
| 2       | pro monthly   | 326            | 32.6       |
| 3       | pro annual    | 195            | 19.5       |
| 4       | churn         | 236            | 23.6       |

The two most popular plans at the end of 2020 are pro monthly and churn. Very few customers are on a trial plan at this point, but that is likely because of the short trial duration (1 week), after which customers must choose a higher tier or churn.
***
8. How many customers have upgraded to an annual plan in 2020?
```sql
SELECT
	COUNT(*)
FROM
	subscriptions s
WHERE
	EXTRACT(YEAR FROM start_date) = 2020 AND
    plan_id = 3; -- 3 is annual
```
Output:
| count |
| ----- |
| 195   |

195 customers upgraded to an annual plan in 2020.
***
9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```sql
WITH annual_customers AS (
SELECT 
	customer_id,
  	start_date AS annual_start,
  	(SELECT start_date FROM subscriptions WHERE customer_id=s.customer_id AND plan_id = 0) AS trial_start
FROM
	subscriptions s
WHERE
    plan_id = 3 -- 3 is annual
)
SELECT
    ROUND(AVG(annual_start-trial_start), 0) AS avg_days_to_annual
FROM
	annual_customers;
```
__Explanation:__
* Create a CTE `annual_customers` selecting only customers with an annual subscription. Create a new column `trial_start` using a subquery to find the trial start date of each customer with an annual subscription.
* Calculate the average difference between the date of annual subscription and trial start for all customers who made an annual subscription.

Output:
| avg_days_to_annual |
| ------------------ |
| 105                |

The average annual-subscription customer takes 105 days to upgrade from their trial plan.
***
10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```sql
WITH annual_customers AS (
SELECT 
	customer_id,
  	start_date AS annual_start,
  	(SELECT start_date FROM subscriptions WHERE customer_id=s.customer_id AND plan_id = 0) AS trial_start
FROM
	subscriptions s
WHERE
    plan_id = 3 -- 3 is annual
)
SELECT
	WIDTH_BUCKET(annual_start-trial_start, 0, 360, 12) AS bucket_id,
    CONCAT(
	(WIDTH_BUCKET(annual_start-trial_start, 0, 360, 12) - 1)*30,
	'-',
	WIDTH_BUCKET(annual_start-trial_start, 0, 360, 12)*30,
	' Days'
	) AS interval,
    COUNT(*) AS customer_count,
	ROUND(AVG(annual_start-trial_start), 0) AS avg_days_to_annual
FROM
	annual_customers
GROUP BY
	bucket_id, interval;
```
__Explanation:__
* Select from the same CTE `annual_customers`, but use WIDTH_BUCKET() to create 30-day buckets. Name the result `bucket_id` and use CONCAT() to translate it into a readable column `interval`. Count all customers and calculate the same average from the previous quesetion, and group by `bucket_id` and `interval`.

Output:
| bucket_id | interval     | customer_count | avg_days_to_annual |
| --------- | ------------ | -------------- | ------------------ |
| 1         | 0-30 Days    | 48             | 10                 |
| 2         | 30-60 Days   | 25             | 42                 |
| 3         | 60-90 Days   | 33             | 71                 |
| 4         | 90-120 Days  | 35             | 100                |
| 5         | 120-150 Days | 43             | 133                |
| 6         | 150-180 Days | 35             | 162                |
| 7         | 180-210 Days | 27             | 190                |
| 8         | 210-240 Days | 4              | 224                |
| 9         | 240-270 Days | 5              | 257                |
| 10        | 270-300 Days | 1              | 285                |
| 11        | 300-330 Days | 1              | 327                |
| 12        | 330-360 Days | 1              | 346                |

Wednesday has the most orders, 5.
***
11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql
WITH pro_subscribers AS (
SELECT
  	customer_id,
  	start_date
FROM
  	subscriptions s
WHERE
  	EXTRACT(YEAR FROM start_date) = 2020 AND
  	plan_id = 2 -- 2 is pro monthly
),
basic_subscribers AS (
SELECT
  	customer_id,
  	start_date
FROM
  	subscriptions s
WHERE
  	EXTRACT(YEAR FROM start_date) = 2020 AND
  	plan_id = 1 -- 1 is basic monthly
)
SELECT
	COUNT(*) AS downgrade_count
FROM
	pro_subscribers p
INNER JOIN
	basic_subscribers b
    ON p.customer_id=b.customer_id
WHERE
	p.start_date < b.start_date;
```
__Explanation:__
* Create a CTE `pro_subscribers` containing customers with a pro monthly subscription, and a CTE `basic_subscribers` containing customers with a basic monthly subscription. Select only subscriptions in 2020 and include just the `customer_id` and `start_date` columns.
* Select the count of rows from the joined table of the two CTEs where the start date of the basic tier comes after the start date of the pro tier.

Output:
| downgrade_count |
| --------------- |
| 0               |

No one downgraded from a pro monthly plan to a basic monthly plan in 2020. 
***
12. How much did each customer pay in 2020?
```sql
WITH base AS (
SELECT
  	s.customer_id AS customer_id,
    s.plan_id AS plan_id, 
  	s.start_date AS start_date,
  	p.price AS price,
    LEAD(start_date) OVER(PARTITION BY customer_id ORDER BY start_date ASC) - start_date AS time_in_tier
FROM
  	subscriptions s
INNER JOIN
  	plans p
  	ON s.plan_id=p.plan_id
WHERE
  	EXTRACT(YEAR FROM start_date) = 2020
),
payments AS (
SELECT
  	*,
  	CASE 
  		WHEN plan_id = 4 OR plan_id = 0 THEN 
  			0
        WHEN plan_id = 3 THEN
            price
  		WHEN time_in_tier IS NULL THEN 
  			(('2020-12-31'-start_date)/30 + 1) * price
  		ELSE 
  			(time_in_tier/30 + 1) * price
  	END AS payment
FROM
  	base
)
SELECT
	customer_id,
	SUM(payment) AS total_payment_in_2020
FROM
	payments
GROUP BY
	customer_id
LIMIT 50;
```
__Explanation:__
* Create a CTE `base` with a new column `time_in_tier` calculated by the difference between the `start_date` of the customer's immediate next tier and the `start_date` of the current tier.
* Create another CTE `payments` selecting from `base` which grabs all columns and creates a new one `payment` that is calculated as 0 if the customer is on trial or churn, and the total payment over the year to the end of 2020 if otherwise. Annual plans are charged once, monthly plans are charged every 30 days.
* Select the SUM of payments grouped by `customer_id`. A sample of 50 customers is shown in the output.

Output:
| customer_id | total_payment_in_2020 |
| ----------- | --------------------- |
| 1           | 49.50                 |
| 2           | 199.00                |
| 3           | 118.80                |
| 4           | 29.70                 |
| 5           | 49.50                 |
| 6           | 9.90                  |
| 7           | 198.80                |
| 8           | 139.20                |
| 9           | 199.00                |
| 10          | 79.60                 |
| 11          | 0                     |
| 12          | 39.60                 |
| 13          | 9.90                  |
| 14          | 39.60                 |
| 15          | 39.80                 |
| 16          | 248.50                |
| 17          | 248.50                |
| 18          | 119.40                |
| 19          | 258.70                |
| 20          | 218.80                |
| 21          | 119.20                |
| 22          | 238.80                |
| 23          | 199.00                |
| 24          | 39.80                 |
| 25          | 159.10                |
| 26          | 19.90                 |
| 27          | 99.50                 |
| 28          | 199.00                |
| 29          | 238.80                |
| 30          | 79.20                 |
| 31          | 318.40                |
| 32          | 129.30                |
| 33          | 79.60                 |
| 34          | 9.90                  |
| 35          | 79.60                 |
| 36          | 218.90                |
| 37          | 79.40                 |
| 38          | 238.80                |
| 39          | 49.60                 |
| 40          | 218.80                |
| 41          | 159.20                |
| 42          | 19.80                 |
| 43          | 69.40                 |
| 44          | 199.00                |
| 45          | 158.90                |
| 46          | 268.50                |
| 47          | 248.50                |
| 48          | 49.50                 |
| 49          | 278.60                |
| 50          | 119.40                |

***
Note: The context of this case study is sourced from the [challenge website](https://8weeksqlchallenge.com/case-study-3/) by Danny Ma.