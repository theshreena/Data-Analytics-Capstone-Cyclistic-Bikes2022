# Capstone Project

# Google-Data-Analytics-Capstone-Project-2022

## ****Finding how Cyclistic Bikes Annual members and Casual riders use bikes****

The objective is to analyse how causual riders and annual membership use bikes differently. 

Provided with a fictional company called Cyclistic bikes. The data for this company is taken from the following website (https://divvy-tripdata.s3.amazonaws.com/index.html).

Used Postgres as the database. 

To access the server, used DataGrip.

Used PostgreSQL script to import, clean and compile the Cyclistic Bike Capstone project for Google Data Analytics Certification.

The steps taken are as follows:

1. Import and compile divvy-trip data CSV documents into one table (12 months)
2. Inspect for anomalies and outliers
3. Create final table to work on the analysis

### Inspect for anomalies or outliers:

To check if there are any discrepancies on the total count of each columns,

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

To check if ride_id has any discrepancies in the length,

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

To find what and how many types of bikes are available,

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

To check if there is any NULL present by comparing with the previous count query,

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

Null identified in start_station_name, start_station_id, end_station_name and end_station_id,

```sql
SELECT count(ride_id) as null_count
FROM tripdata
WHERE start_station_id is null;

#Result
+----------+
|null_count|
+----------+
|698441    |
+----------+

SELECT count(ride_id) as null_count
FROM tripdata
WHERE end_station_id is null;

#Result
+----------+
|null_count|
+----------+
|746820    |
+----------+
```

To find the membership types and existing numbers of the members,

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

To check if date and time are accurate or not,

```sql
SELECT count(ride_id) As difference
FROM tripdata
WHERE ended_at-started_at <='0' ;

#Result
+----------+
|difference|
+----------+
|656       |
+----------+
```

Create new table to exclude NULL,

```sql
CREATE VIEW view_cyclisticbikes AS
SELECT ride_id,
       rideable_type,
       started_at,
       ended_at,
       start_station_name,
       start_station_id,
       end_station_name,
       end_station_id,
       member_casual
FROM tripdata
WHERE start_station_id IS NOT NULL 
AND end_station_id IS NOT NULL;

#Result
+----------------+-------------+--------------------------+--------------------------+------------------------+----------------+--------------------------+--------------+-------------+
|ride_id         |rideable_type|started_at                |ended_at                  |start_station_name      |start_station_id|end_station_name          |end_station_id|member_casual|
+----------------+-------------+--------------------------+--------------------------+------------------------+----------------+--------------------------+--------------+-------------+
|89E7AA6C29227EFF|classic_bike |2021-02-12 16:14:56.000000|2021-02-12 16:21:43.000000|Glenwood Ave & Touhy Ave|525             |Sheridan Rd & Columbia Ave|660           |member       |
|0FEFDE2603568365|classic_bike |2021-02-14 17:52:38.000000|2021-02-14 18:12:09.000000|Glenwood Ave & Touhy Ave|525             |Bosworth Ave & Howard St  |16806         |casual       |
+----------------+-------------+--------------------------+--------------------------+------------------------+----------------+--------------------------+--------------+-------------+
```

Create table to convert date and time to duration and to check duration anomalies,

```sql
CREATE VIEW view_convert_duration AS
SELECT ride_id,
       started_at,
       ended_at,
       date(started_at) AS start_date,
       date(ended_at) AS end_date,
       To_char(started_at,'HH24:MI:SS') AS ride_starttime,
       To_char(ended_at, 'HH24:MI:SS') AS ride_endtime,
       To_char(ended_at-started_at,'HH24:MI:SS') AS duration
FROM tripdata;

#Result
+----------------+--------------------------+--------------------------+----------+----------+--------------+------------+--------+
|ride_id         |started_at                |ended_at                  |start_date|end_date  |ride_starttime|ride_endtime|duration|
+----------------+--------------------------+--------------------------+----------+----------+--------------+------------+--------+
|89E7AA6C29227EFF|2021-02-12 16:14:56.000000|2021-02-12 16:21:43.000000|2021-02-12|2021-02-12|16:14:56      |16:21:43    |00:06:47|
|0FEFDE2603568365|2021-02-14 17:52:38.000000|2021-02-14 18:12:09.000000|2021-02-14|2021-02-14|17:52:38      |18:12:09    |00:19:31|
+----------------+--------------------------+--------------------------+----------+----------+--------------+------------+--------+

SELECT count(ride_id)
FROM view_convert_duration
WHERE duration = '00:00:00';

#Result
+-----+
|count|
+-----+
|507  |
+-----+
```

Cleaned, filtered and excluded anomalies in the  final table to work on the analysis, 

```sql
CREATE VIEW view_finalcyclisticsbikes AS
SELECT ride_id,
       rideable_type,
       started_at,
       ended_at,
       To_char(ended_at-started_at,'HH24:MI:SS') AS duration,
       concat(start_station_name,' ',start_station_id) AS start_location,
       concat(end_station_name,' ',end_station_id) AS end_location,
       member_casual
FROM tripdata
WHERE start_station_id IS NOT NULL
AND end_station_id IS NOT NULL
AND ended_at-started_at != '0' OR ended_at-started_at < '0';

#Result
+----------------+-------------+--------------------------+--------------------------+--------+----------------------------+------------------------------+-------------+
|ride_id         |rideable_type|started_at                |ended_at                  |duration|start_location              |end_location                  |member_casual|
+----------------+-------------+--------------------------+--------------------------+--------+----------------------------+------------------------------+-------------+
|89E7AA6C29227EFF|classic_bike |2021-02-12 16:14:56.000000|2021-02-12 16:21:43.000000|00:06:47|Glenwood Ave & Touhy Ave 525|Sheridan Rd & Columbia Ave 660|member       |
|0FEFDE2603568365|classic_bike |2021-02-14 17:52:38.000000|2021-02-14 18:12:09.000000|00:19:31|Glenwood Ave & Touhy Ave 525|Bosworth Ave & Howard St 16806|casual       |
+----------------+-------------+--------------------------+--------------------------+--------+----------------------------+------------------------------+-------------+
```


