# Bike-Trips

## 🎯 Objective
- How do annual members and casual riders use Cyclistic bikes differently?
Purpose:
To identify the behavioral differences between annual members and casual riders in how they use Cyclistic bikes.

- Why would casual riders buy Cyclistic annual memberships?
Purpose:
To discover the characteristics and riding behavior of casual riders who may benefit from switching to an annual membership.


- How can Cyclistic use digital media to influence casual riders to become members?
Purpose:
To determine how Cyclistic can leverage digital marketing to convert casual riders into paying annual members.


## 📊 Tools Used
- SQL ( EDA , Analyizst )
- Tableau ( Vizualiztion )

## 🧠 Key Insights
📌 1. Usage Behavior Differences: Annual Members vs Casual Riders
- Trip Duration: Casual riders have longer average ride durations (~35 minutes), while annual members average around 15–20 minutes, indicating members use bikes for commuting, while casuals ride for leisure.
- Day of Week: Members primarily ride during weekdays (especially during morning and evening rush hours), while casual riders are more active on weekends.
- Start/End Stations: Members tend to start and end at the same locations (e.g., train stations, office areas), suggesting a structured travel pattern. Casual riders more often take one-way leisure rides.

📌 2. Motivations for Casual Riders to Become Members
- Popular Ride Times: Some casuals consistently ride during weekdays — mirroring member behavior — suggesting they might be open to membership with the right incentive.
- Frequent Start Locations: Many casual riders frequently start from stations in residential areas, implying a potential for regular commuting.

📌 3. Digital Media Strategy Suggestions
- Targeted Social Ads: Run geo-targeted ads focusing on users who frequently ride from residential neighborhoods to downtown, highlighting cost-saving benefits of memberships.
- Email Marketing: For registered casual users, send personalized emails when they hit a certain number of rides per month (e.g., “You’ve taken 5 trips this month — save $X by switching to a membership!”).
- In-app Promotions: Offer trial memberships or discounted first-month plans directly in the app to convert high-usage casual riders.



## 💻 Data Cleaning & Preparation (SQL)

```sql
-- Create a new table to avoid changing the original
CREATE TABLE trips_cleaned
LIKE trip_data.divvy_trips_2019_q1;


-- Copy all data from original table into the new one
INSERT INTO trips_cleaned
SELECT *
FROM trip_data.divvy_trips_2019_q1;


-- Check for duplicate rows using trip_id
-- If result set is empty → No duplicates found
-- The result is is empthy 
SELECT trip_id,Count(*) AS count
FROM  trips_cleaned
group by trip_id
HAVING count(*) > 1;


-- Data Quality Check - Verify if any NULLs exist in key columns
-- If difference = 0 → No missing data
-- The result is = 0 
SELECT 
COUNT(*) AS total_row,
COUNT(*)- COUNT(trip_id) AS missind_trip_id,
COUNT(*)- COUNT(start_time) AS missind_start,
COUNT(*)- COUNT(end_time) AS missind_end,
COUNT(*)- COUNT(usertype) AS missind_usertype
FROM trips_cleaned;



-- Data provide trip duration , but I'm not sure what unit it's in.
-- So, I decided to create a new column called 'trip_duration' and populate it using TIMESTAMPDIFF
ALTER TABLE trips_cleaned
ADD trip_duration INT;

UPDATE trips_cleaned
SET trip_duration = TIMESTAMPDIFF(MINUTE, start_time, end_time);


-- Data only have birthyear , but I need an age column for my analysis.
-- Adding age colume
ALTER TABLE trips_cleaned
ADD age INT;
-- Update age colume details.
UPDATE trips_cleaned
SET age = YEAR(current_date()) - birthyear;


-- Adding period_of_time colume
-- I need to morning, Afternoon, Evening and Night for my analysis
ALTER table trips_cleaned
ADD period_of_time TEXT;
-- Update period_of_time colume details.
UPDATE trips_cleaned
SET period_of_time = CASE
    WHEN TIME(Start_time) >= '21:00:00' OR TIME(Start_time) <= '04:59:59' THEN 'Night'  -- between and cannot use between day, use OR instead!!
    WHEN TIME(Start_time) BETWEEN '05:00:01' AND '11:59:59' THEN 'Morning'
    WHEN TIME(Start_time) BETWEEN '12:00:00' AND '17:59:59' THEN 'Afternoon'
    WHEN TIME(Start_time) BETWEEN '18:00:00' AND '20:59:59' THEN 'Evening'
    ELSE 'Other'
END;


-- Adding Day colume
ALTER table trips_cleaned
ADD Day Text;
-- Update Day colume details.
UPDATE trips_cleaned
SET Day = date_format(start_time, '%a');


-- Adding weekday_weekly colume
-- for my anlysis process.
ALTER TABLE trips_cleaned
ADD weekday_weekly TEXT;
-- Update weekday_weekly colume.
UPDATE trips_cleaned
SET weekday_weekly = CASE
	WHEN day = 'Sat' THEN 'weekend'
	WHEN day = 'Sun' THEN 'weekend'
	ELSE 'weekday'
END;


-- Here the results
![Here the results](https://i.postimg.cc/zv20ZMG4/Capture.jpg)





