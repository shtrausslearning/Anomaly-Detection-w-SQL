## Analysing a server log

### 1. Upload data to TABLE

- Create a table <code>server_log_summary</code> header (contains 3 columns)
   - `status`, `period` & `entries`
- Then copy **table data** from file <code>data.csv</code> into table `server_log_summary`
- Finally, **sort the table** by column <code>period</code> 
- Select top 10 entries (`LIMIT` -> postgreSQL)

```sql
CREATE TABLE server_log_summary (
   status_code int,
   period timestamptz,
   entries int
);

\COPY server_log_summary(status_code, period, entries) 
FROM '/data.csv' 
DELIMITER ',' CSV HEADER;

SELECT * 
FROM server_log_summary 
ORDER BY period DESC 
LIMIT 10;
```

```
CREATE TABLE
COPY 2892
 status_code |         period         | entries 
-------------+------------------------+---------
         500 | 2020-08-01 18:00:00+00 |       0
         200 | 2020-08-01 18:00:00+00 |    4084
         404 | 2020-08-01 18:00:00+00 |       0
         400 | 2020-08-01 18:00:00+00 |      24
         500 | 2020-08-01 17:59:00+00 |       0
         404 | 2020-08-01 17:59:00+00 |       0
         200 | 2020-08-01 17:59:00+00 |    3927
         400 | 2020-08-01 17:59:00+00 |      12
         500 | 2020-08-01 17:58:00+00 |       0
         404 | 2020-08-01 17:58:00+00 |       0
(10 rows)
```

### 2. Preparing the data

- First, we should generate an axis using a cartesian join between the status codes we want to track and the times we want to monitor
- To generate the axis, we can use two nice features of PostgreSQL:
   - `generate_series`: A **function that generates a range of values** ([md example](https://github.com/shtrausslearning/PostgreSQL-tutorials/blob/main/generate_series.md))
   - `data_trunc` : date truncator (1 min interavals, nothing more)
   - `VALUES` with `FROM` : A special clause that can generate "constant tables" [example](https://github.com/shtrausslearning/PostgreSQL-tutorials/blob/main/from%20values.md)
   - After generating the axis, we left to join the actual data to get a complete series for each status code
   - The resulting data has no gaps and is ready for analysis

```sql
WITH axis AS (
   SELECT
       status_code,
       generate_series(
           date_trunc('minute', now()),
           date_trunc('minute', now() - interval '1 hour'),
           interval '1 minute' * -1
       ) AS period
   FROM (
       VALUES (200), (400), (404), (500)
   ) AS t(status_code)
)
SELECT
   a.period,
   a.status_code,
   count(*) AS entries
FROM
   axis a
   LEFT JOIN server_log l ON (
       date_trunc('minute', l.timestamp) = a.period
       AND l.status_code = a.status_code
   )
GROUP BY
   period,
   status_code;
```

### 3. Identifying anomalities

#### What we need to do:
- For each `status_code`:
   - we calculate the **last value (entries)**, **average (hour average)** & **standard deviation (hour average)**
   - Find the <code>z-score</code> using data after **2020-08-01 17:00** (last hour data)

```sql
CREATE TABLE server_log_summary (
   status_code int,
   period timestamptz,
   entries int
);

\COPY server_log_summary(status_code, period, entries) FROM '/data.csv' DELIMITER ',' CSV HEADER;

WITH stats AS (
   SELECT
       status_code,
       (MAX(ARRAY[EXTRACT('epoch' FROM period), entries]))[2] AS last_value,
       AVG(entries) AS mean_entries,
       STDDEV(entries) AS stddev_entries
   FROM
       server_log_summary
   WHERE
        period > '2020-08-01 17:00 UTC'::timestamptz
   GROUP BY
       status_code
)
SELECT * FROM stats;
```

```
CREATE TABLE
COPY 2892
 status_code | last_value |      mean_entries      |     stddev_entries     
-------------+------------+------------------------+------------------------
         500 |          0 | 0.15000000000000000000 | 0.36008473579027553993
         200 |       4084 |  2779.1000000000000000 |       689.219644702665
         404 |          0 | 0.13333333333333333333 | 0.34280333180088158345
         400 |         24 | 0.73333333333333333333 |     3.4388935285299212
(4 rows)
```
#### Calculate z-score of **last value** 

- Next, we calculate the `z-score` for the **last value** for each status code:
   - **period > '2020-08-01 17:00 UTC'::timestamptz** subset is used to calculate the `mean`, `std` 
   - We calculated the `z-score` by finding the number of standard deviations between the last value and the mean. 
   - To avoid a “division by zero” error, we transform the denominator to NULL

```
NULLIF(stddev_entries::float, 0)
```

```sql
WITH stats AS (
   SELECT
       status_code,
       (MAX(ARRAY[EXTRACT('epoch' FROM period), entries]))[2] AS last_value,
       AVG(entries) AS mean_entries,
       STDDEV(entries) AS stddev_entries
   FROM
       server_log_summary
   WHERE
       period > '2020-08-01 17:00 UTC'::timestamptz
   GROUP BY
       status_code
)   
SELECT
   *,
   (last_value - mean_entries) / NULLIF(stddev_entries::float, 0) as zscore
FROM
   stats;
```

```
CREATE TABLE
COPY 2892
 status_code | last_value |      mean_entries      |     stddev_entries     |       zscore       
-------------+------------+------------------------+------------------------+--------------------
         500 |          0 | 0.15000000000000000000 | 0.36008473579027553993 |  -0.41656861591424
         200 |       4084 |  2779.1000000000000000 |       689.219644702665 |   1.89330064810173
         404 |          0 | 0.13333333333333333333 | 0.34280333180088158345 | -0.388949934158693
         400 |         24 | 0.73333333333333333333 |     3.4388935285299212 |   6.76574208350464
(4 rows)
```

#### Conclusions:
- Looking at the `z-scores`:
   - **status code 400** received a very high **z-score of 6** 
- In the past minute:
   - we returned a 400 status code 24 times (significantly higher than the average of 0.73) in the past hour
- Looking at the **raw data**:

```sql
SELECT * 
FROM server_log_summary 
WHERE status_code = 400 
ORDER BY period 
DESC LIMIT 20;
```

```
 status_code |         period         | entries 
-------------+------------------------+---------
         400 | 2020-08-01 18:00:00+00 |      24
         400 | 2020-08-01 17:59:00+00 |      12
         400 | 2020-08-01 17:58:00+00 |       2
         400 | 2020-08-01 17:57:00+00 |       0
         400 | 2020-08-01 17:56:00+00 |       1
         400 | 2020-08-01 17:55:00+00 |       0
         400 | 2020-08-01 17:54:00+00 |       0
         400 | 2020-08-01 17:53:00+00 |       0
         400 | 2020-08-01 17:52:00+00 |       0
         400 | 2020-08-01 17:51:00+00 |       0
         400 | 2020-08-01 17:50:00+00 |       0
         400 | 2020-08-01 17:49:00+00 |       0
         400 | 2020-08-01 17:48:00+00 |       0
         400 | 2020-08-01 17:47:00+00 |       0
         400 | 2020-08-01 17:46:00+00 |       0
         400 | 2020-08-01 17:45:00+00 |       0
         400 | 2020-08-01 17:44:00+00 |       0
         400 | 2020-08-01 17:43:00+00 |       0
         400 | 2020-08-01 17:42:00+00 |       0
         400 | 2020-08-01 17:41:00+00 |       0
(20 rows)
```
