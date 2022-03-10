# Capstone Project 2022

# Google-Data-Analytics-Capstone-Project-2022

## ****Compare how Cyclistic Bikes Member and Casual riders use bikes****

**The objective is to compare how casual riders and annual membership use bikes differently to help build new marketing strategy to convert casual riders to member**. 

A fictional company called Cyclistic bikes data has been provided in CSV format. The data for this company can be found in the following website [(https://divvy-tripdata.s3.amazonaws.com/index.html).](https://divvy-tripdata.s3.amazonaws.com/index.html)

Used 12 months CSV data from February 2021 till January 2022.

To start the project, the data was loaded onto DataGrip creating **divvy** database. Under divvy database, **tripdata** table was created to run the queries through PostgreSQL.

Used PostgreSQL script to clean, filter, sort and create new table to help with the analysis for the Cyclistic Bikes Capstone project of Google Data Analytics Certification.

The steps taken are as follows:

1. Imported 12 months CSV data from the server and compile into one table **tripdata**
2. Inspecting for anomalies and outliers
3. Eliminating and excluding anomalies and outliers
4. Performed calculations to analyze

### Inspecting for anomalies or outliers:

Counting each column to check if there is any anomalies present,

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

-- The result shows that there are anomalies present in start and end station
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
-- The outcome shows that all the ride_id's are 16 character in length
```

Finding how many types of bikes,

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
--There are thre types of bikes available to ride
```

Running query to confirm presence of empty or NULL in start_station_name, start_station_id, end_station_name and end_station_id,

```sql

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

-- The result shows that there is NULL or empty space present
```

Confirming membership types of riders,

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

-- ended_at > started_at is valid as end date should be greater than started date

SELECT count(ride_id) As difference
FROM tripdata
WHERE ended_at <= started_at;

#Result
+----------+
|difference|
+----------+
|201       |
+----------+
-- how many invalid entries present
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
-- Created a table to remove NULL from the entire file. This deletion updated the entire tripdata. 
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
AND (vdraft_cb.ended_at - vdraft_cb.started_at) < '24:00:00'::interval
ORDER BY (vdraft_cb.ended_at - vdraft_cb.started_at);

#Result
+----------------+-------------+--------------------------+--------------------------+---------------------------------------------+----------------------------+----------------+----------------------------+--------------+-------------+
|ride_id         |rideable_type|started_at                |ended_at                  |duration                                     |start_station_name          |start_station_id|end_station_name            |end_station_id|member_casual|
+----------------+-------------+--------------------------+--------------------------+---------------------------------------------+----------------------------+----------------+----------------------------+--------------+-------------+
|40BCB8268A5EB539|electric_bike|2022-01-27 20:38:23.000000|2022-01-27 20:40:24.000000|0 years 0 mons 0 days 0 hours 2 mins 1.0 secs|Ashland Ave & Division St   |13061           |Honore St & Division St     |TA1305000034  |member       |
|7CDFAAFCD77D3AF0|classic_bike |2022-01-01 04:48:16.000000|2022-01-01 04:50:17.000000|0 years 0 mons 0 days 0 hours 2 mins 1.0 secs|Sheffield Ave & Waveland Ave|TA1307000126    |Sheridan Rd & Irving Park Rd|13063         |member       |
+----------------+-------------+--------------------------+--------------------------+---------------------------------------------+----------------------------+----------------+----------------------------+--------------+-------------+

-- The riders who rid for 2 minutes and more are only included. Similarly, since only day pass or full day pass is available, only those customers who have duration below 24 hours valid
```

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
|4468450   |4468450            |4468450        |4468450      |4468450           |4468450         |4468450         |
+----------+-------------------+---------------+-------------+------------------+----------------+----------------+

--The count of all the columns same
```

### Performing calculations to analyze data

Simplified the data by using concatenate and adding a column ‘duration’,

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

Comparing average duration of casual and member riders in monthly basis,

```sql
SELECT to_char(started_at, 'YYYY-MM') AS monthwise,
       member_casual,
       avg(duration) AS average_duration
FROM vfinal_cb
GROUP BY monthwise, member_casual;

#Result

+---------+-------------+----------------------------------------------------+
|monthwise|member_casual|average_duration                                    |
+---------+-------------+----------------------------------------------------+
|2021-02  |casual       |0 years 0 mons 0 days 0 hours 31 mins 34.935205 secs|
|2021-02  |member       |0 years 0 mons 0 days 0 hours 15 mins 6.331606 secs |
|2021-03  |casual       |0 years 0 mons 0 days 0 hours 32 mins 38.921403 secs|
|2021-03  |member       |0 years 0 mons 0 days 0 hours 14 mins 0.382968 secs |
|2021-04  |casual       |0 years 0 mons 0 days 0 hours 32 mins 33.906141 secs|
|2021-04  |member       |0 years 0 mons 0 days 0 hours 14 mins 37.517301 secs|
|2021-05  |casual       |0 years 0 mons 0 days 0 hours 33 mins 40.572118 secs|
|2021-05  |member       |0 years 0 mons 0 days 0 hours 14 mins 44.050614 secs|
|2021-06  |casual       |0 years 0 mons 0 days 0 hours 31 mins 18.650769 secs|
|2021-06  |member       |0 years 0 mons 0 days 0 hours 14 mins 31.80862 secs |
|2021-07  |casual       |0 years 0 mons 0 days 0 hours 28 mins 57.442733 secs|
|2021-07  |member       |0 years 0 mons 0 days 0 hours 14 mins 11.970594 secs|
|2021-08  |casual       |0 years 0 mons 0 days 0 hours 27 mins 32.239953 secs|
|2021-08  |member       |0 years 0 mons 0 days 0 hours 13 mins 55.808297 secs|
|2021-09  |casual       |0 years 0 mons 0 days 0 hours 26 mins 37.412517 secs|
|2021-09  |member       |0 years 0 mons 0 days 0 hours 13 mins 30.442233 secs|
|2021-10  |casual       |0 years 0 mons 0 days 0 hours 24 mins 44.813499 secs|
|2021-10  |member       |0 years 0 mons 0 days 0 hours 12 mins 24.559898 secs|
|2021-11  |casual       |0 years 0 mons 0 days 0 hours 20 mins 32.130969 secs|
|2021-11  |member       |0 years 0 mons 0 days 0 hours 11 mins 18.935536 secs|
|2021-12  |casual       |0 years 0 mons 0 days 0 hours 20 mins 27.331143 secs|
|2021-12  |member       |0 years 0 mons 0 days 0 hours 10 mins 57.697041 secs|
|2022-01  |casual       |0 years 0 mons 0 days 0 hours 18 mins 24.239718 secs|
|2022-01  |member       |0 years 0 mons 0 days 0 hours 10 mins 37.410976 secs|
+---------+-------------+----------------------------------------------------+

--Casual riders' duration is more than members 
```

Calculating to result the same in Tableau visualization analysis,

```sql
Select member_casual, count(ride_id) AS total_count
From vfinal_cb
GROUP BY member_casual;

#Result
+-------------+-----------+
|member_casual|total_count|
+-------------+-----------+
|casual       |2009928    |
|member       |2458522    |
+-------------+-----------+

SELECT member_casual,
       to_char(started_at, 'YYYY-MM') AS monthwise,
       count(ride_id) AS count
FROM vfinal_cb
GROUP BY monthwise, member_casual;

#Result
+-------------+---------+------+
|member_casual|monthwise|count |
+-------------+---------+------+
|casual       |2021-02  |8442  |
|member       |2021-02  |33392 |
|casual       |2021-03  |74507 |
|member       |2021-03  |126512|
|casual       |2021-04  |118518|
|member       |2021-04  |172849|
|casual       |2021-05  |212944|
|member       |2021-05  |227209|
|casual       |2021-06  |298255|
|member       |2021-06  |295611|
|casual       |2021-07  |362580|
|member       |2021-07  |312929|
|casual       |2021-08  |335641|
|member       |2021-08  |322859|
|casual       |2021-09  |287920|
|member       |2021-09  |318312|
|casual       |2021-10  |185827|
|member       |2021-10  |278808|
|casual       |2021-11  |68650 |
|member       |2021-11  |178845|
|casual       |2021-12  |44292 |
|member       |2021-12  |126202|
|casual       |2022-01  |12352 |
|member       |2022-01  |64994 |
+-------------+---------+------+
--monthwise counts of casual and members
```

```sql
SELECT rideable_type,
       count(ride_id) AS count,
       member_casual
FROM vfinal_cb
WHERE member_casual = 'member'
GROUP BY rideable_type, member_casual;

#Result
+-------------+-------+-------------+
|rideable_type|count  |member_casual|
+-------------+-------+-------------+
|classic_bike |1915556|member       |
|electric_bike|542966 |member       |
+-------------+-------+-------------+

SELECT rideable_type,
       count(ride_id) AS count,
       member_casual
FROM vfinal_cb
WHERE member_casual = 'casual'
GROUP BY rideable_type, member_casual;

#Result
+-------------+-------+-------------+
|rideable_type|count  |member_casual|
+-------------+-------+-------------+
|classic_bike |1237677|casual       |
|docked_bike  |306590 |casual       |
|electric_bike|465661 |casual       |
+-------------+-------+-------------+

```

```sql
SELECT to_char(started_at,'Day') AS "day",
       member_casual,
       count(ride_id) AS no_riders
FROM vfinal_cb
GROUP BY   "day", member_casual
ORDER BY day;

##Result
+---------+-------------+---------+
|day      |member_casual|no_riders|
+---------+-------------+---------+
|Wednesday|casual       |214187   |
|Wednesday|member       |385926   |
|Tuesday  |casual       |211288   |
|Tuesday  |member       |377621   |
|Thursday |casual       |220132   |
|Thursday |member       |362470   |
|Sunday   |casual       |395709   |
|Sunday   |member       |300503   |
|Saturday |casual       |459472   |
|Saturday |member       |343744   |
|Monday   |casual       |224704   |
|Monday   |member       |335989   |
|Friday   |casual       |284436   |
|Friday   |member       |352269   |
+---------+-------------+---------+
```

Creating tableau visual graphs for clear insight on the patterns of the casual and member riders. This will help to provide recommendations on the findings.
