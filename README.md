So, I have been working on the Google Data Analytics Professional Certificate course in Coursera and I have finished all of the courses, except the last one.
In the final course, they suggest creating a portfolio to showcase the ability and the understanding of how data analytics work in real life. They give study cases to us to do and include it in our portfolio when we are finished. 

So, I chose to pick Cyclistic, a fictional company based in Chicago. In this case study, I work as a junior data analyst in the marketing team of Cyclistic. The top managers think member riders are much more profitable, so they want to find a way to convert casual riders to member riders. My job for this case is to gain insights and identify the trend from the provided data to help our marketing team to deploy their marketing strategy.

First, I downloaded all the 15 historical data files, 2020/4 -2021/05, from the company cloud server. Because it contains over 4 million rows of data, Excel is not going to be the ideal tool to clean the data unless I do it file by file and is going to take a while.  

Instead of doing it on my laptop, I used Google Bigquery,a fully-managed, serverless data warehouse that enables scalable analysis over petabytes of data to combine and clean the data. 

**I first megered all the 15 files by uisng union all function.**

************************

![Merge](https://user-images.githubusercontent.com/63176613/127413315-b5ce9a60-5501-4c44-87d1-9dd7947cff83.png)

with all_data as (
SELECT * EXCEPT (start_station_id, end_station_id) FROM `prefab-faculty-251001.bike_share.2020_04` 
union all
SELECT * EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2020_05` 
union all
SELECT * EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2020_06`
union all
SELECT* EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2020_07` 
union all
SELECT* EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2020_08` 
union all
SELECT* EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2020_09` 
union all
SELECT* EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2020_10` 
union all
SELECT* EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2020_11` 
union all
SELECT* EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2020_12` 
union all
SELECT* EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2021_01` 
union all
SELECT* EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2021_02` 
union all
SELECT* EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2021_03` 
union all
SELECT * EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2021_04` 
union all
SELECT* EXCEPT (start_station_id,end_station_id) FROM `prefab-faculty-251001.bike_share.2021_05`), 
************

**Then I calculated the ride_lenghth_time and the day of week.**

********************

![agg_data](https://user-images.githubusercontent.com/63176613/127413327-3364cb46-ad65-454f-9871-f79ac9f7e176.png)

agg_data as (
SELECT
    *,
    TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS ride_length_minute, #calculat the time between end and start, and I have been used another table to delete rows where start_time > end_time.
    CASE
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 1 THEN 'Sunday'    
      WHEN EXTRACT(DAYOFWEEK
    FROM
      started_at) = 2 THEN 'Monday'
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 3 THEN 'Tuesday'
      WHEN EXTRACT(DAYOFWEEK
    FROM
      started_at) = 4 THEN 'Wednesday'
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 5 THEN 'Thursday'
      WHEN EXTRACT(DAYOFWEEK
    FROM
      started_at) = 6 THEN 'Friday'
    ELSE
    'Saturday'
  END
    AS day_of_week
  FROM
    all_data),
    
****************************************************    
**I found there were some station name contains irregular symbol or text like Wood St & Taylor St (Temp). So I decide to remove it, in case it causes duplication. I modified the latitude and longitude coordinates to match each station’s coordinates**

************************

![Clean_station name](https://user-images.githubusercontent.com/63176613/127413336-85076efa-4c77-449d-987e-9b2573b5dbba.png)

  clean_station_longlag AS(
  SELECT
  ride_id,
  start_lat,
  start_lng,
  end_lat,
  end_lng,
  TRIM(REPLACE(start_station_name, '(*)', '')) start_station_name, #delete station name with * sight
  TRIM(REPLACE(end_station_name, '(*)', '')) end_station_name
  FROM (
    SELECT
  ride_id,
  round(start_lat,6) as start_lat,#modify latitude 
  round(start_lng,6) as start_lng,#modify longitude 
  round(end_lat,6) as end_lat,
  round(end_lng,6) as end_lng,
  TRIM(REPLACE(start_station_name, '(Temp)', '')) start_station_name,#delete station name with Temp
  TRIM(REPLACE(end_station_name, '(Temp)', '')) end_station_name
  FROM
    agg_data )),
****************************************************

**Last, I just joined the clean data together**

************************
![last](https://user-images.githubusercontent.com/63176613/127413452-7e2d4a50-6743-4b42-ba14-a18151675cf0.png)

  clean_data as (
  SELECT
    ad.ride_id,   #Trying to find a way not typing so many, but i failed
    ad.rideable_type,
    ad.started_at,
    ad.ended_at,
    cl.start_station_name,
    cl.end_station_name,
    cl.start_lat,
    cl.start_lng,
    cl.end_lat,
    cl.end_lng,
    ad.member_casual,
    ad.ride_length_minute,
    ad.day_of_week
  FROM
    agg_data ad
  LEFT JOIN  #Join agg_data and clean_station_longlag together
    clean_station_longlag cl
  ON
    ad.ride_id = cl.ride_id)
************************************************

************************************************
