# 🚲 Cyclistic Bike-Share Analysis Project

## 🎯 Project Objectives

1. **Understand how annual members and casual riders use Cyclistic bikes differently.**
2. **Identify characteristics of casual riders who might benefit from annual membership.**
3. **Suggest digital marketing strategies to convert casual riders into members.**

---

## 🛠️ Tools & Technologies

- SQL (Data Cleaning, Transformation, Analysis)
- Tableau (Data Visualization)
- GitHub (Project versioning & documentation)

---


## Step 1:Data Quality Checks

```sql

-- Create working table
CREATE TABLE trips_cleaned LIKE divvy_trips_2019_q1;
INSERT INTO trips_cleaned SELECT * FROM trip_data.divvy_trips_2019_q1;



-- Check for duplicate trip_id
-- If result set is empty → No duplicates found
SELECT trip_id, COUNT(*) FROM trips_cleaned GROUP BY trip_id HAVING COUNT(*) > 1;

-- Check for missing values
SELECT 
  COUNT(*) AS total_rows,
  COUNT(*) - COUNT(trip_id) AS missing_trip_id,
  COUNT(*) - COUNT(start_time) AS missing_start,
  COUNT(*) - COUNT(end_time) AS missing_end,
  COUNT(*) - COUNT(usertype) AS missing_usertype
FROM trips_cleaned;

```

## Step 2: Feature Engineering

```sql

-- Add trip duration (in minutes)
ALTER TABLE trips_cleaned ADD trip_duration INT;
UPDATE trips_cleaned SET trip_duration = TIMESTAMPDIFF(MINUTE, start_time, end_time);

-- Add age from birth year
ALTER TABLE trips_cleaned ADD age INT;
UPDATE trips_cleaned SET age = YEAR(CURDATE()) - birthyear;

-- Add period of time (Morning, Afternoon, Evening, Night)
ALTER TABLE trips_cleaned ADD period_of_time TEXT;
UPDATE trips_cleaned SET period_of_time = CASE
  WHEN TIME(start_time) BETWEEN '05:00:01' AND '11:59:59' THEN 'Morning'
  WHEN TIME(start_time) BETWEEN '12:00:00' AND '17:59:59' THEN 'Afternoon'
  WHEN TIME(start_time) BETWEEN '18:00:00' AND '20:59:59' THEN 'Evening'
  ELSE 'Night'
END;

-- Add day of week
ALTER TABLE trips_cleaned ADD day TEXT;
UPDATE trips_cleaned SET day = DATE_FORMAT(start_time, '%a');

-- Add weekday/weekend classification
ALTER TABLE trips_cleaned ADD weekday_weekly TEXT;
UPDATE trips_cleaned SET weekday_weekly = CASE
  WHEN day IN ('Sat', 'Sun') THEN 'weekend'
  ELSE 'weekday'
END;

-- Add route column
ALTER TABLE trips_cleaned ADD routh TEXT;
UPDATE trips_cleaned SET routh = CONCAT(from_station_name, ' to ', to_station_name);

```


## Step 3: Exploratory Data Analysis (EDA)

```sql

** Usage Behavior: Members vs Casuals

-- Average trip duration
SELECT usertype, ROUND(AVG(trip_duration), 2) AS avg_trip_mins FROM trips_cleaned GROUP BY usertype;

-- Total trips
SELECT usertype, COUNT(*) AS total_trips FROM trips_cleaned GROUP BY usertype;

-- Time of Day Usage 
SELECT usertype,	period_of_time, COUNT(*) AS total_Usage FROM trips_cleaned GROUP BY usertype, period_of_time ORDER BY total_Usage DESC;

-- Usage by time of day
SELECT usertype, period_of_time, COUNT(*) AS total_Usage FROM trips_cleaned GROUP BY usertype, period_of_time ORDER BY total_Usage DESC;

-- Usage by day of week
SELECT usertype, day, COUNT(*) AS total_Usage FROM trips_cleaned GROUP BY usertype, day ORDER BY total_Usage DESC;






** Station Analysis (Top 10)

-- Top 10 routh
SELECT distinct routh, count(*) AS total_trips FROM trips_cleaned GROUP BY routh ORDER BY total_trips DESC LiMIT 10;

-- Top 10 start stations by usertype
SELECT usertype, from_station_name, COUNT(*) AS total_usage FROM trips_cleaned
WHERE usertype = 'Subscriber'
GROUP BY from_station_name
ORDER BY total_usage DESC
LIMIT 10;

-- (Repeat for 'Customer')

-- Top 10 end stations by usertype
SELECT usertype, to_station_name, COUNT(*) AS total_usage
FROM trips_cleaned
WHERE usertype = 'Customer'
GROUP BY to_station_name
ORDER BY total_usage DESC
LIMIT 10;

-- (Repeat for 'Subscriber')

```









