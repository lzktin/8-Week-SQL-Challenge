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
__Explanation:__
* 

Output:
| null_customer_ids | null_dates | null_product_ids |
| ----------------- | ---------- | ---------------- |
| 0                 | 0          | 0                |

There are no null values.

Using the same strategy to check the `menu` and `members` tables, it is discovered they have no missing values either.

* Filter duplicate entries

```sql
WITH indexed_menu AS (
SELECT
  	*,
  	ROW_NUMBER() OVER(
    	PARTITION BY product_name, price
      	ORDER BY product_id ASC
    ) AS index
  	FROM menu
)
SELECT 
	*
FROM
	indexed_menu
WHERE 
	index = 1
ORDER BY
	product_id ASC
```
__Explanation:__
* Create a CTE with a new column `index` using ROW_NUMBER() to assign row numbers based on product_name and price. Entries with the same product_name and price will be assigned row numbers (1,2,3..) in ascending order of product_id.
* Select only entries with `index` 1 to guarantee only one instance exists for each restaurant product.

Output:
| product_id | product_name | price |
| ---------- | ------------ | ----- |
| 1          | sushi        | 10    |
| 2          | curry        | 15    |
| 3          | ramen        | 12    |

Use the same strategy on the `members` table filter entries with the same `customer_id` and `join_date`. 

The `sales` table is omitted because it has no primary key, so entries with identical `customer_id`, `product_id` and `order_date` could be a customer ordering the same item multiple times on the same date.

### Case Study Questions
1. What is the total amount each customer spent at the restaurant?
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