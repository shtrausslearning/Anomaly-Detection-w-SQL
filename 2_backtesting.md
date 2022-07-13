## 1 | Finding past anomalities

#### Z-Score threshold & backtesting:
- In the previous section, **we identified an anomaly** 
    - We found an **increase in the 400 status** code because the `z-score` was **6** (which is high) 
- But how do we set the **threshold** for the z-score? (Is a z-score of 3 an anomaly?)
- To **find thresholds** that fit our needs,
    -  We can **run simulations on past data** with different values and evaluate the results
    -  **This is often called backtesting**

#### We need to do:
- calculate the <code>mean</code> and the <code>standard deviation</code> for each status code up until every row, 
- just as if it is the current value; job for a window function `WINDOW`

```sql
WITH calculations_over_window AS (
   SELECT
      status_code,
      period,
      entries,
      AVG(entries) OVER status_window as mean_entries,
      STDDEV(entries) OVER status_window as stddev_entries
   FROM
      server_log_summary
   WINDOW status_window AS (
      PARTITION BY status_code
      ORDER BY period
      ROWS BETWEEN 60 PRECEDING AND CURRENT ROW
   )
)
SELECT *
FROM calculations_over_window
ORDER BY period DESC
LIMIT 20;
```

```
CREATE TABLE
COPY 2892
 status_code |         period         | entries |      mean_entries      |     stddev_entries     
-------------+------------------------+---------+------------------------+------------------------
         200 | 2020-08-01 18:00:00+00 |    4084 |  2759.9672131147540984 |       699.597407256800
         400 | 2020-08-01 18:00:00+00 |      24 | 0.72131147540983606557 |     3.4114080550460080
         404 | 2020-08-01 18:00:00+00 |       0 | 0.13114754098360655738 | 0.34036303344446665347
         500 | 2020-08-01 18:00:00+00 |       0 | 0.14754098360655737705 | 0.35758754516763638735
         500 | 2020-08-01 17:59:00+00 |       0 | 0.16393442622950819672 | 0.37328844382740000274
         400 | 2020-08-01 17:59:00+00 |      12 | 0.32786885245901639344 |     1.5676023249473471
         200 | 2020-08-01 17:59:00+00 |    3927 |  2718.6721311475409836 |       694.466863171826
         404 | 2020-08-01 17:59:00+00 |       0 | 0.13114754098360655738 | 0.34036303344446665347
         500 | 2020-08-01 17:58:00+00 |       0 | 0.16393442622950819672 | 0.37328844382740000274
         404 | 2020-08-01 17:58:00+00 |       0 | 0.13114754098360655738 | 0.34036303344446665347
         200 | 2020-08-01 17:58:00+00 |    3850 |  2680.4754098360655738 |       690.967283512936
         400 | 2020-08-01 17:58:00+00 |       2 | 0.13114754098360655738 | 0.38623869286861001780
         404 | 2020-08-01 17:57:00+00 |       0 | 0.13114754098360655738 | 0.34036303344446665347
         400 | 2020-08-01 17:57:00+00 |       0 | 0.09836065573770491803 | 0.30027309973793774423
         500 | 2020-08-01 17:57:00+00 |       1 | 0.16393442622950819672 | 0.37328844382740000274
         200 | 2020-08-01 17:57:00+00 |    3702 |  2643.0327868852459016 |       688.414796645480
         200 | 2020-08-01 17:56:00+00 |    3739 |  2607.5081967213114754 |       688.769908918569
         404 | 2020-08-01 17:56:00+00 |       0 | 0.14754098360655737705 | 0.35758754516763638735
         400 | 2020-08-01 17:56:00+00 |       1 | 0.11475409836065573770 | 0.32137001808599097120
         500 | 2020-08-01 17:56:00+00 |       0 | 0.14754098360655737705 | 0.35758754516763638735
(20 rows)
```

- We use a window function, to calculate the **mean** and **standard deviation** over a **sliding window of 60 minutes**.
- To avoid repeating the <code>WINDOW</code> clause for every aggregate, we define a named window called “status_window.” 
- This is another nice feature of PostgreSQL
- In the results, we can now see that we have the mean and standard deviation of the previous 60 rows for every entry. 
- This is similar to the calculation we did in the previous section except this time we’ll do it for every row.
- Now we can calculate the z-score for every row:

```sql
WITH calculations_over_window AS (
   SELECT
      status_code,
      period,
      entries,
      AVG(entries) OVER status_window as mean_entries,
      STDDEV(entries) OVER status_window as stddev_entries
   FROM
      server_log_summary
   WINDOW status_window AS (
      PARTITION BY status_code
      ORDER BY period
      ROWS BETWEEN 60 PRECEDING AND CURRENT ROW
   )
),

with_zscore AS (
   SELECT
       *,
       (entries - mean_entries) / NULLIF(stddev_entries::float, 0) as zscore
   FROM
       calculations_over_window
)

SELECT
   status_code,
   period,
   zscore
FROM
   with_zscore
ORDER BY
   period DESC
LIMIT
   20;
```

```
CREATE TABLE
COPY 2892
 status_code |         period         |       zscore       
-------------+------------------------+--------------------
         200 | 2020-08-01 18:00:00+00 |   1.89256388481616
         400 | 2020-08-01 18:00:00+00 |   6.82377720547307
         404 | 2020-08-01 18:00:00+00 | -0.385316641635245
         500 | 2020-08-01 18:00:00+00 | -0.412601013654965
         500 | 2020-08-01 17:59:00+00 | -0.439162875091059
         400 | 2020-08-01 17:59:00+00 |   7.44584960215151
         200 | 2020-08-01 17:59:00+00 |   1.73993596085159
         404 | 2020-08-01 17:59:00+00 | -0.385316641635245
         500 | 2020-08-01 17:58:00+00 | -0.439162875091059
         404 | 2020-08-01 17:58:00+00 | -0.385316641635245
         200 | 2020-08-01 17:58:00+00 |   1.69259039909672
         400 | 2020-08-01 17:58:00+00 |   4.83859461395841
         404 | 2020-08-01 17:57:00+00 | -0.385316641635245
         400 | 2020-08-01 17:57:00+00 | -0.327570654259568
         500 | 2020-08-01 17:57:00+00 |    2.2397306629644
         200 | 2020-08-01 17:57:00+00 |   1.53826910501475
         200 | 2020-08-01 17:56:00+00 |   1.64277182935479
         404 | 2020-08-01 17:56:00+00 | -0.412601013654965
         400 | 2020-08-01 17:56:00+00 |   2.75460015502278
         500 | 2020-08-01 17:56:00+00 | -0.412601013654965
(20 rows)
```

- We now have z-scores for every row, and we can try to identify anomalies:

```sql
WITH calculations_over_window AS (
   SELECT
       status_code,
       period,
       entries,
       AVG(entries) OVER status_window as mean_entries,
       STDDEV(entries) OVER status_window as stddev_entries
   FROM
       server_log_summary
   WINDOW status_window AS (
       PARTITION BY status_code
       ORDER BY period
       ROWS BETWEEN 60 PRECEDING AND CURRENT ROW
   )
),

with_zscore AS (
   SELECT
       *,
       (entries - mean_entries) / NULLIF(stddev_entries::float, 0) as zscore
   FROM
       calculations_over_window
),

with_alert AS (

   SELECT
       *,
       zscore > 3 AS alert
   FROM
       with_zscore
)

SELECT
   status_code,
   period,
   entries,
   zscore,
   alert
FROM
   with_alert
WHERE
   alert
ORDER BY
   period DESC
LIMIT
   20;
```

```
CREATE TABLE
COPY 2892
 status_code |         period         | entries |      zscore      | alert 
-------------+------------------------+---------+------------------+-------
         400 | 2020-08-01 18:00:00+00 |      24 | 6.82377720547307 | t
         400 | 2020-08-01 17:59:00+00 |      12 | 7.44584960215151 | t
         400 | 2020-08-01 17:58:00+00 |       2 | 4.83859461395841 | t
         500 | 2020-08-01 17:29:00+00 |       1 | 3.00273099737938 | t
         500 | 2020-08-01 17:20:00+00 |       1 | 3.31909527471312 | t
         500 | 2020-08-01 17:18:00+00 |       1 |  3.7438474117708 | t
         500 | 2020-08-01 17:13:00+00 |       1 |  3.7438474117708 | t
         500 | 2020-08-01 17:09:00+00 |       1 | 4.36077899493003 | t
         500 | 2020-08-01 16:59:00+00 |       1 |  3.7438474117708 | t
         400 | 2020-08-01 16:29:00+00 |       1 | 3.00273099737938 | t
         404 | 2020-08-01 16:13:00+00 |       1 | 3.00273099737938 | t
         500 | 2020-08-01 15:13:00+00 |       1 | 3.00273099737938 | t
         500 | 2020-08-01 15:11:00+00 |       1 | 3.00273099737938 | t
         500 | 2020-08-01 14:58:00+00 |       1 | 3.00273099737938 | t
         400 | 2020-08-01 14:56:00+00 |       1 | 3.00273099737938 | t
         400 | 2020-08-01 14:55:00+00 |       1 | 3.31909527471312 | t
         400 | 2020-08-01 14:50:00+00 |       1 | 3.31909527471312 | t
         500 | 2020-08-01 14:37:00+00 |       1 | 3.00273099737938 | t
         400 | 2020-08-01 14:35:00+00 |       1 | 3.31909527471312 | t
         400 | 2020-08-01 14:32:00+00 |       1 | 3.31909527471312 | t
(20 rows)
```

- We decided to classify values with a z-score greater than 3 as anomalies.
- You’ll often find that 3 is the magic number in textbooks, but do not get sentimental about it, because you can definitely change it to get better results

## 2 | Adding Thresholds

- In the last lesson, we detected a large number of “anomalies” with just one entry
- This is very common in errors that do not happen very often 
- In our case, every once in a while, we get a 400 status code
- However, the standard deviation is very low because it does not happen very often so even a single error can be considered way above the acceptable value
- We do not really want to receive an alert in the middle of the night just because of one 400 status code 
- We cannot have every curious developer fiddling with the dev tools in his browser wake us up in the middle of the night
- To eliminate rows with only a few entries, we set a threshold:

```sql
WITH calculations_over_window AS (
   SELECT
       status_code,
       period,
       entries,
       AVG(entries) OVER status_window as mean_entries,
       STDDEV(entries) OVER status_window as stddev_entries
   FROM
       server_log_summary
   WINDOW status_window AS (
       PARTITION BY status_code
       ORDER BY period
       ROWS BETWEEN 60 PRECEDING AND CURRENT ROW
   )
),

with_zscore AS (
   SELECT
       *,
       (entries - mean_entries) / NULLIF(stddev_entries::float, 0) as zscore
   FROM
       calculations_over_window
),

with_alert AS (

   SELECT
       *,
       entries > 10 AND zscore > 3 AS alert
   FROM
       with_zscore
)

SELECT
   status_code,
   period,
   entries,
   zscore,
   alert
FROM
   with_alert
WHERE
   alert
ORDER BY
   period DESC;
```

```

CREATE TABLE
COPY 2892
 status_code |         period         | entries |      zscore      | alert 
-------------+------------------------+---------+------------------+-------
         400 | 2020-08-01 18:00:00+00 |      24 | 6.82377720547307 | t
         400 | 2020-08-01 17:59:00+00 |      12 | 7.44584960215151 | t
         500 | 2020-08-01 11:29:00+00 |    5001 | 3.17219844196164 | t
         500 | 2020-08-01 11:28:00+00 |    4812 | 3.39716469102639 | t
         500 | 2020-08-01 11:27:00+00 |    4443 | 3.53494000896016 | t
         500 | 2020-08-01 11:26:00+00 |    4522 | 4.12647853355536 | t
         500 | 2020-08-01 11:25:00+00 |    5567 | 6.17629336121081 | t
         500 | 2020-08-01 11:24:00+00 |    3657 | 6.86899923611412 | t
         500 | 2020-08-01 11:23:00+00 |    1512 | 6.34226066258968 | t
         500 | 2020-08-01 11:22:00+00 |    1022 | 7.68218967250475 | t
         404 | 2020-08-01 07:20:00+00 |      23 | 5.14212641009848 | t
         404 | 2020-08-01 07:19:00+00 |      20 | 6.09120069792082 | t
         404 | 2020-08-01 07:18:00+00 |      15 | 7.57547172423804 | t
(13 rows)   

```

- After eliminating potential anomalies with less than 10 entries:
    - we get much fewer and probably more relevant results

## 3 | Eliminating repeated alerts

- In the previous lesson, we eliminated potential anomalies with less than 10 entries
- Using thresholds, we were able to remove some non-interesting anomalies
- Let’s have a look at the data for status code 400 after applying the threshold:
- The first alert happened at 17:59, and a minute later:
    - the z-score was still high with a large number of entries, 
    - and so we classified the next rows at 18:00 as an anomaly as well.
- If you think of an alerting system, we want to send an alert only when an anomaly first happens. 
- We do not want to send an alert every minute until the z-score comes back below the threshold. 
- In this case, we only want to send one alert at 17:59. 
- We do not want to send another alert a minute later at 18:00.
- Let’s remove alerts where the previous period was also classified as an alert:

```sql
WITH calculations_over_window AS (
   SELECT
       status_code,
       period,
       entries,
       AVG(entries) OVER status_window as mean_entries,
       STDDEV(entries) OVER status_window as stddev_entries
   FROM
       server_log_summary
   WINDOW status_window AS (
       PARTITION BY status_code
       ORDER BY period
       ROWS BETWEEN 60 PRECEDING AND CURRENT ROW
   )
),

with_zscore AS (
   SELECT
       *,
       (entries - mean_entries) / NULLIF(stddev_entries::float, 0) as zscore
   FROM
       calculations_over_window
),

with_alert AS (

   SELECT
       *,
       entries > 10 AND zscore > 3 AS alert
   FROM
       with_zscore
),

with_previous_alert AS (
   SELECT
       *,
       LAG(alert) OVER (PARTITION BY status_code ORDER BY period) AS previous_alert
   FROM
       with_alert
)

SELECT
   status_code,
   period,
   entries,
   zscore,
   alert
FROM
   with_previous_alert
WHERE
   alert AND NOT previous_alert
ORDER BY
   period DESC;
   
```

```
CREATE TABLE
COPY 2892
 status_code |         period         | entries |      zscore      | alert 
-------------+------------------------+---------+------------------+-------
         400 | 2020-08-01 17:59:00+00 |      12 | 7.44584960215151 | t
         500 | 2020-08-01 11:22:00+00 |    1022 | 7.68218967250475 | t
         404 | 2020-08-01 07:18:00+00 |      15 | 7.57547172423804 | t
(3 rows)
```

- By eliminating alerts that were already triggered we get a very small list of anomalies that may have happened during the day 
- Looking at the results we can see what anomalies we would have discovered:
    - Anomaly in status code 400 at 17:59: We also found that one earlier  
- Anomaly in status code 404: This is a hidden anomaly that we did not know about until now
- The query can now be used to fire alerts when it encounters an anomaly

## 4 | Quiz

- “Backtesting can be used to identify past anomalies” : True
- “Backtesting is used primarily to refine the parameters” : True
- “An anomaly detection system cannot produce false alerts” : False
- “Thresholds are used to minimize false positives on very low values” : True
- “False alerts compromise the reliability of the anomaly detection system” : True
