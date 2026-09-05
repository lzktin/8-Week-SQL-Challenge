### September 5 - Danny's Diner
![Case Study 1, Danny's Diner](imgs/caseStudy01.png)

## Table of Contents
1. [Business Problem](#business-problem)
2. [Tables and Data Structure](#tables-and-data-structure)
3. [Question and Solution](#question-and-solution)

Note: The context of this case study is sourced from the [challenge website](https://8weeksqlchallenge.com/case-study-1/) by Danny Ma.

## Business Problem
Danny's new restaurant has been operating for a few months. He wants to know the visiting and spending patterns of his customers, if he should expand the current customer loyalty program, aiming to deliver a better and more personalized experience for his loyal customers.

## Tables and Data Structure
![Table relationship diagram](imgs/relations.png)
```sql -- Add 3 backticks followed by sql
TABLE sales {
  "customer_id" VARCHAR(1) -- foreign key (members)
  "order_date" DATE
  "product_id" INTEGER -- foreign key (menu)
}

TABLE menu {
  "product_id" INTEGER -- primary key
  "product_name" VARCHAR(5)
  "price" INTEGER
}

TABLE members {
  "customer_id" VARCHAR(1) -- primary key
  "join_date" TIMESTAMP
}
``` -- Add 3 backticks

## Question and Solution
1. 