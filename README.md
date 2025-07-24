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
SELECT
	trip_id,
    Count(*) AS count
FROM	
	trips_cleaned
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




