# September 5 - Danny's Diner
![Case Study 1, Danny's Diner](imgs/caseStudy01.png)

<!-- ## Table of Contents
1. [Business Problem](#business-problem)
2. [Tables and Data Structure](#tables-and-data-structure)
3. [Question and Solution](#question-and-solution) -->

## Business Problem
Danny has been operating his new restaurant for a few months. He wants to know if he should expand the current customer loyalty program based on the visiting and spending patterns of his customers, aiming to deliver a better and more personalized experience for loyalty members.

## Tables and Data Structure
![Table relationship diagram](imgs/relations.png)

Note that the sales table has no primary key.

## Question and Solution
All queries are written in PostgreSQL.

Copy the solution code and execute on [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) to see the results!

### Check data quality
Before analysis, always check the quality of data. Here is a brief scan:

* Check missing values with SUM and CASE WHEN

```sql
SELECT
	SUM(CASE WHEN customer_id IS NULL THEN 1 ELSE 0 END) AS null_customer_ids,
    SUM(CASE WHEN order_date IS NULL THEN 1 ELSE 0 END) AS null_dates,
    SUM(CASE WHEN product_id IS NULL THEN 1 ELSE 0 END) AS null_product_ids
FROM sales;
```
Output:
| null_customer_ids | null_dates | null_product_ids |
| ----------------- | ---------- | ---------------- |
| 0                 | 0          | 0                |

Using the same strategy to check the menu table and members table, we find that they have no missing values either.

* Checking for duplicate entries

```sql

```

```sql
SELECT 
	s.customer_id,
    SUM(m.price) AS spending
FROM
	sales s
LEFT JOIN
	menu m
    ON s.product_id = m.product_id
GROUP BY
	s.customer_id
ORDER BY
	s.customer_id ASC;
```




Note: The context of this case study is sourced from the [challenge website](https://8weeksqlchallenge.com/case-study-1/) by Danny Ma.