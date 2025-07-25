#  Cyclistic Bike-Share Analysis Project

##  Project Objectives

1. **Understand how annual members and casual riders use Cyclistic bikes differently.**
2. **Identify characteristics of casual riders who might benefit from annual membership.**
3. **Suggest digital marketing strategies to convert casual riders into members.**

---

##  Tools & Technologies

- SQL (Data Cleaning, Transformation, Analysis)
- Tableau (Data Visualization)
- GitHub (Project versioning & documentation)

##  Interactive Dashboard (Tableau)

An interactive dashboard was created using Tableau to visually support the analysis and reveal patterns between members and casual riders.

- Breakdown by user type 
- Relationship Between Trip Duration and Rider Age
- Peak Usage Times: Subscribers vs. Casual Riders
- Top 10 start stations
- top 10 end stations

 [View the Tableau Dashboard](https://public.tableau.com/views/TRIPS_17534161166800/Dashboard1#1)


---


### Step 1:Data Quality Checks

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

### Step 2: Feature Engineering

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


### Step 3: Exploratory Data Analysis (EDA)

```sql

** Usage Behavior: Members vs Casuals

-- Average age
SELECT usertype, CEiL(AVG(age)) AS avg_age FROM trips_cleaned GROUP BY usertype;

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
SELECT distinct usertype, routh, count(*) AS total_trips FROM trips_cleaned GROUP BY usertype, routh ORDER BY total_trips DESC LIMIT 10;
-- (Repeat for 'Customer')

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

---

###  Step 4: Key Insights Summary
- Total Usage: Subscribers accounted for 61,195 trips, while Casual Riders made 978 trips.

- Average Age: Subscribers are slightly older than Casual Riders, with an average age of ~44 years old compared to ~37 for Casual Riders.

- Trip Duration: Casual riders ride longer (avg. ~35 mins) vs Subscribers (~12 mins)

- Time Patterns: Subscribers ride more on weekdays about ~10k trips per day, 
                during commuting hours (~22k trips on Morning and ~26k trips on Afternoon; 
                casual riders peak on Friday (~200 trips) and Saturday (~300 trips)

- Station Use: Members use consistent stations (office/train hubs transfering), casuals prefer tourist/leisure spots

- Repeated Routes: Members tend to have recurring routes (commute patterns)

  ---

  ###  Step 5: Answering the Objectives & Recommendations

### Objective 1: Understand how annual members and casual riders use Cyclistic bikes differently.
 
#### Trip Duration:
- Members average about 12 minutes per trip, indicating their use for short, quick commutes.
- Casual riders consistently cycle for much longer, averaging around 35 minutes per trip. This suggests their usage is more for leisure or extended sightseeing.
  Basically, members use bikes for their daily trips, making them quick and routine. Casual riders, however, use them for longer, fun rides like exploring or sightseeing.
#### Route Patterns
- Members tend to use the same routes repeatedly, especially during morning and afternoon hours. This aligns with typical daily commuting patterns.
- Casual riders do not show clear or repetitive route patterns, reinforcing the assumption that they use the bikes for exploration or recreational travel rather than fixed daily commutes.
#### Time of Usage
- Members show peak usage during weekdays, particularly in the morning and evening rush hours, which corresponds to typical commuting times.
- Casual riders have their highest usage on Fridays and Saturdays, periods when people generally engage in leisure activities or travel.
#### Station Usage
- Members frequently use stations located near business districts or public transportation hubs as their starting and ending points.
- Casual riders typically choose stations closer to tourist attractions or recreational areas.

  ---

### Objective 2: Identify characteristics of casual riders who might benefit from annual membership.
- High Frequency of Use: They use our bikes almost every day, or at least very frequently.
- Shorter Trip Durations: Their ride lengths are shorter than the typical casual rider's average of 35 minutes, often falling closer to the member's average of 12-15 minutes.
- Repetitive Routes: They tend to use the same routes repeatedly, even if not daily.
- Weekday Usage: While primarily casual users, they occasionally use bikes on weekdays, especially during commuting hours.

  ---
### Objective 3: Suggest digital marketing strategies to convert casual riders into members
- Our casual customers are, on average, around 37 years old. That means they might not be as active on social media as younger folks. Plus, since many are travelers, they're often busy exploring and aren't glued to their phones checking social media all the time. So, we shouldn't put all our marketing money into social media. Instead, a smarter move would be to reach them where they are: by partnering with GPS or navigation apps that people frequently use when they're looking for directions or exploring. That way, we can connect with them right when they need a ride.

---










