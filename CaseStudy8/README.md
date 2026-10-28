# October 28 - Fresh Segments
![Case Study 8, Balanced Tree Clothing Co.](imgs/caseStudy08.png)

## Tables and Data Structure
```sql
-- Interests map
CREATE TABLE fresh_segments.interest_map (
  "id" INTEGER,
  "interest_name" TEXT,
  "interest_summary" TEXT,
  "created_at" TIMESTAMP,
  "last_modified" TIMESTAMP
);
-- Metrics table
CREATE TABLE fresh_segments.interest_metrics (
  "_month" VARCHAR(4),
  "_year" VARCHAR(4),
  "month_year" VARCHAR(7),
  "interest_id" VARCHAR(5),
  "composition" FLOAT,
  "index_value" FLOAT,
  "ranking" INTEGER,
  "percentile_ranking" FLOAT
);
```

## Question and Solution
All queries are written in PostgreSQL.

***
### Case Study Questions
1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month

```sql
ALTER TABLE fresh_segments.interest_metrics 
ALTER COLUMN month_year TYPE DATE
USING to_date(month_year, 'MM-YYYY');
```
***
2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?
```sql
SELECT
	month_year,
    COUNT(*) AS cnt_records
FROM
	fresh_segments.interest_metrics
GROUP BY
	month_year
ORDER BY
	month_year ASC NULLS FIRST;
```
Output:
| month_year | cnt_records |
| ---------- | ----------- |
|            | 1194        |
| 2018-07-01 | 729         |
| 2018-08-01 | 767         |
| 2018-09-01 | 780         |
| 2018-10-01 | 857         |
| 2018-11-01 | 928         |
| 2018-12-01 | 995         |
| 2019-01-01 | 973         |
| 2019-02-01 | 1121        |
| 2019-03-01 | 1136        |
| 2019-04-01 | 1099        |
| 2019-05-01 | 857         |
| 2019-06-01 | 824         |
| 2019-07-01 | 864         |
| 2019-08-01 | 1149        |
***
3. What do you think we should do with these null values in the fresh_segments.interest_metrics?
- Since the database is ordered chronologically, we should fill in the null values using a forward fill and backwards fill for all null values in the fields month, year and month_year.
- For nulls in id, we should replace with the string "unknown".
***
4. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?
```sql
SELECT
	SUM(CASE WHEN i.interest_id IS NOT NULL AND m.id IS NULL THEN 1 ELSE 0 END) AS in_metrics_not_in_map,
    SUM(CASE WHEN i.interest_id IS NULL AND m.id IS NOT NULL THEN 1 ELSE 0 END) AS in_map_not_in_metrics
FROM
	fresh_segments.interest_metrics i
FULL OUTER JOIN
	fresh_segments.interest_map m
    ON CAST(i.interest_id AS INT)=m.id;
```
Output:
| in_metrics_not_in_map | in_map_not_in_metrics |
| --------------------- | --------------------- |
| 0                     | 7                     |
***
5. Summarise the id values in the fresh_segments.interest_map by its total record count in this table.

```sql
SELECT
	m.id,
    m.interest_name,
    COUNT(*) AS record_count
FROM
	fresh_segments.interest_map m
INNER JOIN
	fresh_segments.interest_metrics i
    ON m.id=CAST(i.interest_id AS INT)
GROUP BY
	m.id, m.interest_name
ORDER BY 
	record_count DESC, m.id ASC
LIMIT 
	10;
```
Output:
| id  | interest_name             | record_count |
| --- | ------------------------- | ------------ |
| 4   | Luxury Retail Researchers | 14           |
| 5   | Brides & Wedding Planners | 14           |
| 6   | Vacation Planners         | 14           |
| 12  | Thrift Store Shoppers     | 14           |
| 15  | NBA Fans                  | 14           |
| 16  | NCAA Fans                 | 14           |
| 17  | MLB Fans                  | 14           |
| 18  | Nascar Fans               | 14           |
| 20  | Moviegoers                | 14           |
| 25  | Doctors                   | 14           |
***
6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.
- Use inner join so only matching records to each interest id in map are fetched from the records. That way, uncertainties with NULL interest_id in the metrics table won't distort results.

```sql
SELECT
	*
FROM
	fresh_segments.interest_map m
INNER JOIN
	fresh_segments.interest_metrics i
    ON m.id=CAST(i.interest_id AS INT)
WHERE
	i.interest_id='21246';
```
Output:
| id    | interest_name                    | interest_summary                                      | created_at          | last_modified       | _month | _year | month_year | interest_id | composition | index_value | ranking | percentile_ranking |
| ----- | -------------------------------- | ----------------------------------------------------- | ------------------- | ------------------- | ------ | ----- | ---------- | ----------- | ----------- | ----------- | ------- | ------------------ |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 |        |       |            | 21246       | 1.61        | 0.68        | 1191    | 0.25               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 | 4      | 2019  | 2019-04-01 | 21246       | 1.58        | 0.63        | 1092    | 0.64               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 | 3      | 2019  | 2019-03-01 | 21246       | 1.75        | 0.67        | 1123    | 1.14               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 | 2      | 2019  | 2019-02-01 | 21246       | 1.84        | 0.68        | 1109    | 1.07               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 | 1      | 2019  | 2019-01-01 | 21246       | 2.05        | 0.76        | 954     | 1.95               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 | 12     | 2018  | 2018-12-01 | 21246       | 1.97        | 0.7         | 983     | 1.21               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 | 11     | 2018  | 2018-11-01 | 21246       | 2.25        | 0.78        | 908     | 2.16               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 | 10     | 2018  | 2018-10-01 | 21246       | 1.74        | 0.58        | 855     | 0.23               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 | 9      | 2018  | 2018-09-01 | 21246       | 2.06        | 0.61        | 774     | 0.77               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 | 8      | 2018  | 2018-08-01 | 21246       | 2.13        | 0.59        | 765     | 0.26               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04 | 2018-06-11 17:50:04 | 7      | 2018  | 2018-07-01 | 21246       | 2.26        | 0.65        | 722     | 0.96               |
***
7. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?

```sql
SELECT 
	COUNT(*)
FROM
	fresh_segments.interest_map m
INNER JOIN
	fresh_segments.interest_metrics i
    ON m.id=CAST(i.interest_id AS INT)
WHERE
    EXTRACT(YEAR FROM m.created_at) > CAST(i._year AS INT) OR
    (EXTRACT(YEAR FROM m.created_at) = CAST(i._year AS INT) AND EXTRACT(MONTH FROM m.created_at) > CAST(i._month AS INT));
```
Output:
| count |
| ----- |
| 0     |

There are no records in which the creation date in the map table is after the date in the metrics table.
***
Note: The context of this case study is sourced from the [challenge website](https://8weeksqlchallenge.com/case-study-8/) by Danny Ma.