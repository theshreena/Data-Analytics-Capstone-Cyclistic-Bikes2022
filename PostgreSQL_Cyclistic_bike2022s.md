# Capstone Project 2022

# Google-Data-Analytics-Capstone-Project-2022

## ****Finding how Cyclistic Bikes Annual members and Casual riders use bikes****

**The objective is to analyze how casual riders and annual membership use bikes differently**. 

A fictional company called Cyclistic bikes data has been provided in CSV format. The data for this company can be found in the following website (https://divvy-tripdata.s3.amazonaws.com/index.html).

To start the project, accessed the server by using DataGrip and used PostgresSQL as the database to run the queries.

Used PostgreSQL script to clean, filter, sort and create new table to help with the analysis for the Cyclistic Bike Capstone project of Google Data Analytics Certification.

The steps taken are as follows:

1. Imported 12 months CVS data from the server and compile into one table
2. Inspecting for anomalies and outliers
3. Eliminating and excluding anomalies and outliers
4. Performed basic calculations to help compare in RStudio
5. Created final table version to upload onto RStudio for data visualization and analysis

### Inspecting for anomalies or outliers:

Checking and comparing if there are any discrepancies on the total count of each columns,

```sql
SELECT
       count(ride_id) AS count_ride,
       count(rideable_type) AS count_rideable_type,
       count(started_at) AS count_starttime,
       count(ended_at) AS count_endtime,
       count(start_station_name) AS count_startstation,
       count(end_station_name) AS count_endstation,
       count(member_casual) AS count_membership
FROM tripdata;

# Result
+----------+-------------------+---------------+-------------+------------------+----------------+----------------+
|count_ride|count_rideable_type|count_starttime|count_endtime|count_startstation|count_endstation|count_membership|
+----------+-------------------+---------------+-------------+------------------+----------------+----------------+
|5601999   |5601999            |5601999        |5601999      |4903555           |4855179         |5601999         |
+----------+-------------------+---------------+-------------+------------------+----------------+----------------+
```

Querying if ride_id has any discrepancies in the length of the text,

```sql
SELECT Length(ride_id) AS textlength
FROM tripdata
GROUP BY Length(ride_id);

#Result
+----------+
|textlength|
+----------+
|16        |
+----------+
```

Double checking if ride_id column has any empty or NULL present,

```sql
SELECT count(ride_id)
FROM tripdata
where ride_id LIKE ' ';

#Result
+-----+
|count|
+-----+
|0    |
+-----+

SELECT count(ride_id)
FROM tripdata
where ride_id IS NULL;

#Result
+-----+
|count|
+-----+
|0    |
+-----+
```

Finding how many types and numbers of bikes(initially) are available,

```sql
SELECT DISTINCT (rideable_type) AS available_bikes
FROM tripdata;

#Result
+---------------+
|available_bikes|
+---------------+
|docked_bike    |
|classic_bike   |
|electric_bike  |
+---------------+
```

```sql
SELECT rideable_type AS available_bikes,
count (rideable_type) AS numberofbikes
FROM tripdata
GROUP BY rideable_type;

#Result
+---------------+-------------+
|available_bikes|numberofbikes|
+---------------+-------------+
|classic_bike   |3244395      |
|docked_bike    |311198       |
|electric_bike  |2046406      |
+---------------+-------------+
```

Double checking if rideable_type column has any empty or NULL present, 

```sql
SELECT count(rideable_type)
FROM tripdata
where tripdata.rideable_type like ' ';

#Result
+-----+
|count|
+-----+
|0    |
+-----+

SELECT count(rideable_type)
FROM tripdata
where tripdata.rideable_type IS NULL;

#Result
+-----+
|count|
+-----+
|0    |
+-----+
```

Double checking started_at and ended_at column counts incase of any NULL present by comparing with the previous count query,

```sql
SELECT Count (started_at) AS startdate_nonull,
       count (ended_at) AS enddate_nonull
FROM tripdata
WHERE started_at IS NOT NULL
AND ended_at IS NOT NULL;

#Result
+----------------+--------------+
|startdate_nonull|enddate_nonull|
+----------------+--------------+
|5601999         |5601999       |
+----------------+--------------+
```

Running query to confirm presence of empty or NULL in start_station_name, start_station_id, end_station_name and end_station_id,

```sql
-- Identified in the count query of NULL or empty column presence which is confirmed from below query too

SELECT count(ride_id) as null_count
FROM tripdata
WHERE start_station_id is NULL
OR start_station_id = ' ';

#Result
+----------+
|null_count|
+----------+
|698441    |
+----------+

SELECT count(ride_id) as null_count
FROM tripdata
WHERE end_station_id is NULL
OR end_station_id = ' ';

#Result
+----------+
|null_count|
+----------+
|746820    |
+----------+
```

Checking empty space or NULL present in member_causal column,

```sql
SELECT count(member_casual) as null_count
FROM tripdata
WHERE member_casual is null
OR member_casual = ' ';

#Result
+----------+
|null_count|
+----------+
|0         |
+----------+
```

Confirming membership types and existing numbers of the members(initially),

```sql
SELECT DISTINCT (member_casual)
FROM tripdata;

#Result
+-------------+
|member_casual|
+-------------+
|member       |
|casual       |
+-------------+

SELECT member_casual,
count(member_casual) AS numberofmembers
FROM tripdata
GROUP BY member_casual;

#Result
+-------------+---------------+
|member_casual|numberofmembers|
+-------------+---------------+
|casual       |2529408        |
|member       |3072591        |
+-------------+---------------+
```

Checking accuracy of date and time,

```sql
SELECT count(ride_id) As difference
FROM tripdata
WHERE ended_at > started_at;

#Result
+----------+
|difference|
+----------+
|4584720   |
+----------+

SELECT count(ride_id) As difference
FROM tripdata
WHERE ended_at <= started_at;

#Result
+----------+
|difference|
+----------+
|201       |
+----------+
```

### Eliminating and excluding anomalies and outliers

Creating new table to exclude the anomalies and outliers stepwise,

```sql
--Some of the results have Limit of 2 

CREATE VIEW vdraft_cb AS
    SELECT ride_id,
       rideable_type,
       started_at,
       ended_at,
       start_station_name,
       start_station_id,
       end_station_name,
       end_station_id,
       member_casual
FROM tripdata;

#Result
+----------------+-------------+--------------------------+--------------------------+-------------------+----------------+------------------------+--------------+-------------+
|ride_id         |rideable_type|started_at                |ended_at                  |start_station_name |start_station_id|end_station_name        |end_station_id|member_casual|
+----------------+-------------+--------------------------+--------------------------+-------------------+----------------+------------------------+--------------+-------------+
|45C4CCCB83148DE5|classic_bike |2021-12-07 18:05:46.000000|2021-12-07 18:12:18.000000|Wells St & Huron St|TA1306000012    |Wells St & Walton St    |TA1306000011  |casual       |
|1F79B0418779E2DE|classic_bike |2021-12-01 05:45:55.000000|2021-12-01 05:51:36.000000|Wells St & Huron St|TA1306000012    |Kingsbury St & Kinzie St|KA1503000043  |casual       |
+----------------+-------------+--------------------------+--------------------------+-------------------+----------------+------------------------+--------------+-------------+

DELETE FROM vdraft_cb
WHERE NOT (vdraft_cb is not null);

SELECT count(ride_id)
FROM vdraft_cb;

#Result
+-------+
|count  |
+-------+
|4584921|
+-------+
```

```sql
CREATE VIEW vdraft2 AS
SELECT vdraft_cb.ride_id,
       vdraft_cb.rideable_type,
       vdraft_cb.started_at,
       vdraft_cb.ended_at,
       vdraft_cb.ended_at - vdraft_cb.started_at AS duration,
       vdraft_cb.start_station_name,
       vdraft_cb.start_station_id,
       vdraft_cb.end_station_name,
       vdraft_cb.end_station_id,
       vdraft_cb.member_casual
FROM vdraft_cb
WHERE vdraft_cb.ended_at > vdraft_cb.started_at
  AND (vdraft_cb.ended_at - vdraft_cb.started_at) > '00:02:00'::interval
ORDER BY (vdraft_cb.ended_at - vdraft_cb.started_at);

#Result
+----------------+-------------+--------------------------+--------------------------+---------------------------------------------+----------------------------+----------------+----------------------------+--------------+-------------+
|ride_id         |rideable_type|started_at                |ended_at                  |duration                                     |start_station_name          |start_station_id|end_station_name            |end_station_id|member_casual|
+----------------+-------------+--------------------------+--------------------------+---------------------------------------------+----------------------------+----------------+----------------------------+--------------+-------------+
|40BCB8268A5EB539|electric_bike|2022-01-27 20:38:23.000000|2022-01-27 20:40:24.000000|0 years 0 mons 0 days 0 hours 2 mins 1.0 secs|Ashland Ave & Division St   |13061           |Honore St & Division St     |TA1305000034  |member       |
|7CDFAAFCD77D3AF0|classic_bike |2022-01-01 04:48:16.000000|2022-01-01 04:50:17.000000|0 years 0 mons 0 days 0 hours 2 mins 1.0 secs|Sheffield Ave & Waveland Ave|TA1307000126    |Sheridan Rd & Irving Park Rd|13063         |member       |
+----------------+-------------+--------------------------+--------------------------+---------------------------------------------+----------------------------+----------------+----------------------------+--------------+-------------+

-- The riders who have ridden 2 minutes and more are only included
```

### Basic calculations to compare and double check in RStudio,

Counting each columns to make sure it has same numbers of data,

```sql
SELECT
       count(ride_id) AS count_ride,
       count(rideable_type) AS count_rideable_type,
       count(started_at) AS count_starttime,
       count(ended_at) AS count_endtime,
       count(start_station_name) AS count_startstation,
       count(end_station_name) AS count_endstation,
       count(member_casual) AS count_membership
FROM vdraft2;

#Result
+----------+-------------------+---------------+-------------+------------------+----------------+----------------+
|count_ride|count_rideable_type|count_starttime|count_endtime|count_startstation|count_endstation|count_membership|
+----------+-------------------+---------------+-------------+------------------+----------------+----------------+
|4469733   |4469733            |4469733        |4469733      |4469733           |4469733         |4469733         |
+----------+-------------------+---------------+-------------+------------------+----------------+----------------+

--Shows the columns have accurate data
```

Calculating duration to check the accuracy in RStudio,

```sql
SELECT max(duration) AS max_duration,
       min(duration) AS min_duration,
       avg(duration) AS avg_duration
FROM vdraft2

#Result
+------------+----------------------------------------------------+
|max_duration|0 years 0 mons 38 days 20 hours 24 mins 9.0 secs    |
+------------+----------------------------------------------------+
|min_duration|0 years 0 mons 0 days 0 hours 5 mins 1.0 secs       |
+------------+----------------------------------------------------+
|avg_duration|0 years 0 mons 0 days 0 hours 24 mins 56.789892 secs|
+------------+----------------------------------------------------+
```

Calculating number of monthly casual and member bikers,

```sql
SELECT date_trunc('month', started_at) AS monthwise,
       count(ride_id) AS count,
       member_casual
FROM vdraft2
GROUP BY date_trunc('month',started_at),
         member_casual;
#Result
+--------------------------+------+-------------+
|monthwise                 |count |member_casual|
+--------------------------+------+-------------+
|2021-02-01 00:00:00.000000|8463  |casual       |
|2021-02-01 00:00:00.000000|33394 |member       |
|2021-03-01 00:00:00.000000|74597 |casual       |
|2021-03-01 00:00:00.000000|126513|member       |
|2021-04-01 00:00:00.000000|118626|casual       |
|2021-04-01 00:00:00.000000|172851|member       |
|2021-05-01 00:00:00.000000|213159|casual       |
|2021-05-01 00:00:00.000000|227212|member       |
|2021-06-01 00:00:00.000000|298509|casual       |
|2021-06-01 00:00:00.000000|295613|member       |
|2021-07-01 00:00:00.000000|362774|casual       |
|2021-07-01 00:00:00.000000|312929|member       |
|2021-08-01 00:00:00.000000|335748|casual       |
|2021-08-01 00:00:00.000000|322862|member       |
|2021-09-01 00:00:00.000000|288019|casual       |
|2021-09-01 00:00:00.000000|318312|member       |
|2021-10-01 00:00:00.000000|185915|casual       |
|2021-10-01 00:00:00.000000|278812|member       |
|2021-11-01 00:00:00.000000|68685 |casual       |
|2021-11-01 00:00:00.000000|178848|member       |
|2021-12-01 00:00:00.000000|44327 |casual       |
|2021-12-01 00:00:00.000000|126202|member       |
|2022-01-01 00:00:00.000000|12369 |casual       |
|2022-01-01 00:00:00.000000|64994 |member       |
+--------------------------+------+-------------+
--monthwise counts of casual and members
```
### Cleaned, filtered, sorted and excluded anomalies to help in the analysis

```sql
CREATE VIEW vfinal_cb AS
    SELECT ride_id,
       rideable_type,
       started_at,
       ended_at,
       duration,
       concat(start_station_name,' ',start_station_id) AS start_location,
       concat(end_station_name,' ',end_station_id) AS end_location,
       member_casual
FROM vdraft2;

#Result
+----------------+-------------+--------------------------+--------------------------+---------------------------------------------+-----------------------------------------+------------------------------------+-------------+
|ride_id         |rideable_type|started_at                |ended_at                  |duration                                     |start_location                           |end_location                        |member_casual|
+----------------+-------------+--------------------------+--------------------------+---------------------------------------------+-----------------------------------------+------------------------------------+-------------+
|40BCB8268A5EB539|electric_bike|2022-01-27 20:38:23.000000|2022-01-27 20:40:24.000000|0 years 0 mons 0 days 0 hours 2 mins 1.0 secs|Ashland Ave & Division St 13061          |Honore St & Division St TA1305000034|member       |
|7CDFAAFCD77D3AF0|classic_bike |2022-01-01 04:48:16.000000|2022-01-01 04:50:17.000000|0 years 0 mons 0 days 0 hours 2 mins 1.0 secs|Sheffield Ave & Waveland Ave TA1307000126|Sheridan Rd & Irving Park Rd 13063  |member       |
+----------------+-------------+--------------------------+--------------------------+---------------------------------------------+-----------------------------------------+------------------------------------+-------------+
```
