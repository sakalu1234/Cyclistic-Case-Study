  						**Cyclistic-Case-Study **
						
						
I have been working on the Google Data Analytics Professional Certificate course in Coursera and basically, I have finished all of the courses, except the last one.

In the final course, they suggest creating a portfolio to showcase the ability and the understanding of how data analytics work in real life. They give study cases to us to do and include it in our portfolio when we are finished.**





**Project Introduction**


I work as a junior data analyst in the marketing team of Cyclistic, a fictional bike-sharing company based in Chicago. The executives want to develop a marketing strategy that converts casual riders to member riders since member riders are more profitable. My job for this project is to identify how casual riders and members act differently when they use our bike-sharing service and then share the information with the marketing team to develop a proper campaign to convert casual riders to member riders.

************************************************************************************

**Collecting data**


   I downloaded all the 15 historical data files, 2020/4 -2021/05, from the company cloud server. All the historical data is in comma-delimited format with 15 columns, ride ID #, ride type, start/end time, ride length (in minutes), day of the week, starting point (code, name, and latitude/longitude), ending point (code, name, and latitude/longitude), and member/casual rider.


   Because this data is stored in the company server, I am going to assume the data is reliable.

********************************************************************************
**Process** 


Since it contains over 4 million rows of data, in this case, Excel will not be an ideal tool to use.  I will be using Bigquery to manipulate the data, a fully-managed, serverless data warehouse that enables scalable analysis over petabytes of data to combine and clean the data. 
The code below merged all the data into one

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
********************************************************************************
**Cleaning**


****Some of the data is wrong, like the end_time is small than the start_time. So I decided to delete the data.


	DELETE FROM `prefab-faculty-251001.bike_share.2020_04` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2020_05` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2020_06` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2020_07` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2020_08` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2020_09` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2020_10` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2020_11` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2020_12` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2021_01` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2021_02` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2021_03` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2021_04` WHERE end_time < start_time;
	DELETE FROM `prefab-faculty-251001.bike_share.2021_05` WHERE end_time < start_time;	



****In some start_station_name and end_station_name contain irregular symbol and text, so I decided to remove it and I also to modified the latitude and longitude coordinates to match each stations. 




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



****Then I calculated the ride length time and the day of the week. By knowing how long both parties ride can help us to know our them better.

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

****************************************************************************************************************
**Analyze**

After I rechecked several times on the cleaning process, I finally started to analyze the data to identify reasons for the different behaviors of casual riders and member riders.
Since Bigquery does not provide any data visualization, I decided to use Tableau software to visualize the data.

1. We can see from here that casual riders prefer to ride longer than member riders. For both riders, the peak occurs on Saturday and Sunday.
 
	However, compared to member riders, the average time for casual riders is more fluctuant. I assumed that most member riders use our bike-sharing service to commute on regular basis and most casual riders use it for leisure purposes and these people could be visitors.

	
 ![1](https://user-images.githubusercontent.com/63176613/127579623-b6265f55-2e1a-49c9-9f71-1dfd27ff6a2c.png)
 
 
Figure 2 and Figure 3 confirm the theory. Casual riders are concentrated on the North Side community area of Chicago but member riders are more scattered throughout the city.


![2](https://user-images.githubusercontent.com/63176613/127579688-aca72d1b-6466-4c58-bd24-be380de73faf.png)

Figure 2 Casual Riders on workday
![3](https://user-images.githubusercontent.com/63176613/127579691-783eef27-b072-46e2-962f-39cd865da752.png)
	Figure 3 Member Riders on workday
 
 
 
 

2. Regardless of both member riders and casual riders, the usage of bike-sharing service reaches to the peak in 5:00 PM during workday.
 ![Picture4](https://user-images.githubusercontent.com/63176613/127579794-c72ccf81-b9b0-41d2-bcbd-5f6c48294c8b.png)	

But in weekend, the peak is 3 hours ealier, 2:00 PM
 
![Picture5](https://user-images.githubusercontent.com/63176613/127579862-acbfb851-83c1-4fb3-827a-64f28c06498a.png)



Bike-sharing service usage increase in May, ends in August. May to August will be a good time to have advertisement. 
 ![Picture6](https://user-images.githubusercontent.com/63176613/127579895-6696ddbf-05fd-463d-94c3-1ba02464a9f6.png)


This is bike usage in January and the tuning point starts on 27. But we do not have any further data in January, but still the marketing team can use it as a reference for the future.
 
![Picture7](https://user-images.githubusercontent.com/63176613/127579902-c8001509-6402-4153-a480-b0d016508962.png)



****************************************************
**Recommendation**


The purpose of marketing campaign is to convert and reaches to potential member riders. Based on the insights I gained from the data, here is what I recommend.

1.	Most of the casual riders maybe tourists. I recommend developing weekly membership for tourists. 
2.	Considering students’ summer holiday and the peak usage, the best period to have advertisement is in between May and August. 
3.	Casual riders are concentrated in Streeterville, a neighborhood in the Near North Side. Therefore, to have ads there can convert and get maximum reach for casual riders.
4.	Create different ad schedules can save money and have the maximum return. On weekends, the usage of bikes reaches the peak at 2:00 PM. Therefore, to have a campaign from 1:00 PM to 3 PM will be ideal and the workday is from 4:00 PM – 6:00 PM.
5.	The usage of bike is heavily related to the weather, California would be a good state if we want to expand the market.
