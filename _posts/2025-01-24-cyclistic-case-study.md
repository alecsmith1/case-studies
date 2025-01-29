---
title: "Cyclistic Bike Share Case Study"
subtitle: "Google Data Analytics Professional Certificate Case Study"
author: "Alec Smith"
date: 2025-01-24
---

Welcome! This data analysis was conducted as a capstone project for the Google Data Analytics Professional Certificate. Thank you for reading. 

<br />

## **About The Company**

- **Company –** Cyclistic - fictional Chicago bike share company
- **Offering –** single-ride passes, full-day passes, annual memberships
- **Fleet –** 5,824 bikes across 692 stations

<br />

## **Company Status**

**Profitability Assessment –** <br />
Finance analysts found that annual members are much more profitable than single-ride or day-pass casual riders. 

**Historical Marketing –** <br />
Cyclistic's marketing up to now has been geared toward general awareness and appealing to broad consumer segments.

**Hypothesis –** <br />
Lily Moreno (Director of Marketing) believes there's opportunity to convert casual riders into members and that a marketing campaign aimed at converting casual riders into annual members would help drive future growth

<br />

---

<br />

## **Defining The Problem**

**Business Task –** <br /> 
Analyze Cyclistic historical bike trip data to understand how annual members and casual riders use Cyclistic bikes differently.

**Key Stakeholders –**
- Lily Moreno – the director of marketing (my manager)
- Executive Team – the team deciding whether to approve the recommended marketing program

<br />

---

<br />

## **Data Collection**

- **Data Source –** Data used in this analysis is internal to the company and can be found here: https://divvy-tripdata.s3.amazonaws.com/index.html
- **Data License –** Licensing info can be found in the data license agreement that makes it available for public use: https://divvybikes.com/data-license-agreement
- **Data Set –** For this study, I chose to use Q1 2020 data due to its accuracy and completeness and thanks to its robust nature, consisting of hundreds of thousands of records.
- **Data Storage –** Data stored locally for the duration of the project as well as in BigQuery where SQL analysis was conducted.

_Raw dataset schema:_ <br />
<img width="279" alt="schema - 2020_Q1_trip_data_raw" src="https://github.com/user-attachments/assets/b6f8a9d9-7454-4130-ad89-3f9e330627a4" />

<br />

---

<br />

## **Data Processing** - _conducted in Excel_

I've conducted my data processing and cleaning in Excel. The dataset was largely clean and complete. However, via sorting, filtering, verifying and transforming data types, and creating new data points and fields, I was able to ensure data integrity and cleanliness, as well as set up the additional fields that would provide for robust and streamlined upcoming analysis stages.

<br />

#### **Data Cleaning - Steps Taken***

1. **Applied a filter** to the dataset for streamlined dataset assessment.
2. **Used Excel's “Remove Duplicates” tool** to check for duplicate values (no duplicates found).
3. **Checked all columns for blank fields**, removing the 1 record found with a blank field (`end_station_name` field).
4. **Checked for outliers to ensure data isn't skewed** by sorting `ride_duration_hrs` ride duration column (created in dataset augmentation section below): <br />

     _a. Removed 2,500 records with bike rental duration of zero minutes._ <br />
     _b. Removed 41 records bike rental duration longer than 366 hours (14 days)._ <br />
     _c. Removed 3,766 records with a ride route of *HQ QR - HQ QR* (ride started & finished at what appears to be Cyclistic headquarters), as these rides were almost all <1 minute or had a negative duration (end time was before start time)._ <br />
     
6. **Used formula** `=COUNTA(UNIQUE())` **to check distinct counts** of station names against station IDs to check for errant additional ID or name values (no issues found).
7. **Used formula** `=LEN()` **to check fields for uniform length** (fields checked: `ride_id`, `rideable_type`, `start_station_id`, `end_station_id`, and `member_casual`). No `=TRIM()` required.
8. **Checked data formats**, setting a uniform Date format in the `started_at` and `ended_at` columns to ensure uniformity.

_*Number of Records - Starting out: 426,887, Post-cleaning: 420,580_

<br />

#### **Data Augmentation - Steps Taken**

1. **Created field** `ride_duration_hrs` via formula: `=(D2-C2)*24` - formula subtracts start date from end date, multiplying by 24 to get hours as a whole number with 2 decimal points when formatted as a number.
2. **Created field** `start_day_of_week` via formula: `=WEEKDAY(C2,2)` - formula denotes the day of the week that the date fell on, with the `2` at the end of the equation noting that Monday is day "1" and Sunday is day "7".
3. **Created field** `route` via formula: `=CONCATENATE(H2," ","–"," ",J2)` … formula results in syntax `Start Station Name – End Station Name` and will allow for finding most frequently rented routes for each casual and member users.

_Cleaned & augmented dataset schema:_ <br />
<img width="279" alt="schema - 2020_Q1_trip_data_cleaned" src="https://github.com/user-attachments/assets/068d6d4a-4893-425c-b5ad-ddac4fdb139f" />

<br />

---

<br />

## **Data Analysis** - _conducted with SQL in BigQuery_

I've conducted my data analysis in BigQuery via SQL. The steps of this analysis, consisting of the issues I set out to resolve and the questions I set out to answer, are laid out below, along with the corresponding SQL queries used to garner the data and insights I sought.

<br />

#### **One bit of cleanup**
The `route` field had a character with an error in it, where a long hyphen “–” got turned into some sort foreign character like a “D” with a line through the left part. I modified the entries in this field.

```
UPDATE `test-project-1-coursera-course.alec_case_study_cyclistic.2020_Q1_trip_data_cleaned`
SET route = REPLACE(route, 'Ð', '-')
WHERE route LIKE '%Ð%';
```

<br />

#### **Creating a pared down table**
The dataset contained more fields than were of use in my analysis. As such, I created a new table, paring down the list of fields/columns to only those most useful to my analysis in order to streamline query results.

```
SELECT 
    started_at,
    start_day_of_week,
    ended_at,
    ride_duration_hrs,
    route,
    start_station_name,
    end_station_name,
    member_casual
FROM 
    `test-project-1-coursera-course.alec_case_study_cyclistic.2020_Q1_trip_data_pared`
```

_Pared down dataset schema:_ <br />
<img width="282" alt="schema - 2020_Q1_trip_data_pared" src="https://github.com/user-attachments/assets/994e6f33-3b09-470f-ad7b-418438ce5e16" />

<br />

#### **Getting a count of member vs casual riders**
To set the stage, I asked how many casual riders vs members there are. 

```
SELECT
    member_casual,
    COUNT(member_casual)
FROM
    `test-project-1-coursera-course.alec_case_study_cyclistic.2020_Q1_trip_data_pared`
GROUP BY
    member_casual
```

member_casual|ride_count
member|376003
casual|44577

<br />

#### **Assessing day-of-week bike usage**
Next I asked which days of the week members and casual riders used bikes in order to help form initial hypotheses about bike usage habits. With this query I was able to get an aggregate of weekdays vs weekend days bike usage.

```
SELECT 
    member_casual,
    CASE
        WHEN day_of_week BETWEEN 1 AND 5 THEN 'Weekday'
        WHEN day_of_week IN (6, 7) THEN 'Weekend'
    END AS ride_category,  -- Classifies days into Weekday or Weekend
    COUNT(*) AS ride_count -- Counts the rides for each group
FROM `test-project-1-coursera-course.alec_case_study_cyclistic.2020_Q1_trip_data_cleaned`
GROUP BY 
    member_casual, 
    ride_category
ORDER BY 
    member_casual, 
    ride_category;
```

member_casual	|day_of_week_category	|ride_count
casual	|Weekday	|22272
casual	|Weekend	|22305
member	|Weekday	|310460
member	|Weekend	|65543

<br />

#### **One bit of troubleshooting**
**Issue -** <br />
I found that in the original dataset CSV, timestamps' timezones were unmarked and listed in local time (Central Time, Chicago). However, when the dataset was uploaded to BigQuery, BigQuery had listed "UTC" with the timestamp. I originally interpreted this to mean BigQuery had adjusted the timestamps to UTC time since the time zone was unmarked in the data itself. As a result, I thought I would need to include a timezone adjustment in my SQL queries to ensure day-of-time analyses would properly reflect local Chicago time. Including a timezone adjustment in my SQL queries resulted in misleading data about times of day bikes were used.

**Troubleshooting / solution -** <br />
A deeper dive was needed so I began crosschecking the CSV with the data table in BigQuery. What I found was that BigQuery had _not_ in fact adjusted the timezone to UTC, it had simply applied the timezone label "UTC" to timestamps that were in local Chicago time. With this takeaway, I was able to conduct time-of-day analyses that provided meaningful takeaways.

<br />

#### **Assessing time-of-day bike usage**
Following the day-of-week assessment, I wanted to hone in on which hours of the day members and casual riders each used Cyclistic bikes. The below SQL queries was utilized twice: once each to garner time-of-day bike usage for each members and casual riders.

```
SELECT 
    member_casual,
    EXTRACT(HOUR FROM TIMESTAMP(started_at)) AS hour_of_day,
    COUNT(*) AS ride_count -- Count the number of rides
FROM 
    `test-project-1-coursera-course.alec_case_study_cyclistic.2020_Q1_trip_data_cleaned`
WHERE
    member_casual = 'member' -- Query was run a 2nd time to assess 'casual' rider bike usage as well
GROUP BY 
    member_casual, 
    hour_of_day
ORDER BY 
    member_casual, 
    hour_of_day;
```

_**Member** bike usage by hour:_
<img width="921" alt="time-of-day-analysis-member-bike-usage-by-hour" src="https://github.com/user-attachments/assets/a74a294c-b629-4360-a9df-cd3718879821" />

_**Casual rider** bike usage by hour:_
<img width="917" alt="time-of-day-analysis-casual-rider-bike-usage-by-hour" src="https://github.com/user-attachments/assets/cb505ec4-202a-4e21-8d20-18c58499fa4e" />

<br />

#### **Assessing average ride duration**
At this point, I wanted to further assess what the data appeared to be demonstrating, which is that members use bikes for commute purposes and casual riders use bikes for leisure. To further investigate I looked at average ride duration, anticipating that commutes to/from work would last ~20 minutes or less, while leisurely rides would longer, perhaps ~1 hour or longer.

```
SELECT 
    member_casual,
    MIN(ride_duration_hrs) AS shortest_ride_hrs,
    MAX(ride_duration_hrs) AS longest_ride_hrs,
    AVG(ride_duration_hrs) AS avg_ride_duration_hrs
FROM 
    `test-project-1-coursera-course.alec_case_study_cyclistic.2020_Q1_trip_data_pared` 
GROUP BY
    member_casual
ORDER BY
    member_casual
```

member_casual | shortest_ride_hrs | longest_ride_hrs | avg_ride_duration_hrs
casual | 0.02 | 311.37 | 1.03
member | 0.02 | 173.73 | 0.20

<br />

#### **Assessing percentage of unique routes**
Another opportunity to evaluate the data was around the percentage of unique routes ridden by each the members and the casual riders. This helped evaluate the hypothesis that members would have a higher percentage of repetition in their routes ridden since they use the bikes for the same purpose regularly around commutes, while leisure riders would have a higher percentage of unique rides due to the more diverse nature of tourism or ways people experience the city for leisure. 

```
SELECT
    member_casual,
    COUNT(DISTINCT(route)) AS unique_routes,
    COUNT(route) AS total_rides,
    (COUNT(DISTINCT(route)) / COUNT(route)) * 100 AS unique_rides_as_percent_of_total
FROM 
    `test-project-1-coursera-course.alec_case_study_cyclistic.2020_Q1_trip_data_pared` 
GROUP BY
    member_casual
ORDER BY
    unique_routes DESC;
```

member_casual | unique_routes | total_rides | unique_rides_as_percent_of_total
member | 47593 | 376003 | 12.66%
casual | 16761 | 44577 | 37.60%

<br />

#### **Assessing percentage of routes starting/ending at the same station**
In a very similar sense to the unique routes percentage assessed above, I anticipated we'd see a higher rate of member rides that started and ended at different stations, since members may be commuting from point A to point B on their ride. And in contrast, I anticipated casual riders would have a higher percentage of their rides that started and ended in the same location since leisurely riders might be traveling in a tourist loop and/or need to dock the bikes back where they started at the end of their ride to return to their vehicle.

```
SELECT 
    member_casual, 
    CASE 
        WHEN start_station_name = end_station_name THEN 'Same Station'
        ELSE 'Different Station'
    END AS same_or_diff_station,
    COUNT(*) AS ride_count, -- Total rides for each ride type
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY member_casual), 2) AS ride_percentage -- Percentage of rides
FROM `test-project-1-coursera-course.alec_case_study_cyclistic.2020_Q1_trip_data_cleaned`
GROUP BY 
    member_casual, 
    same_or_diff_station
ORDER BY 
    member_casual, 
    same_or_diff_station;
```

member_casual | same_or_diff_station | ride_count | ride_percentage
casual | Different Station | 38007 | 85.26%
casual | Same Station | 6570 | 14.74%
member | Different Station | 368522 | 98.01%
member | Same Station | 7481 | 1.99%

<br />

--- 

<br />

## **Sharing (Interpreting the Analysis)**

Takeaways are pretty clear cut from this data set. Our primary learnings fall into 3 categories:
1. There are ample casual riders with which to justify targeting campaigns toward encouraging them to become annual members.
2. Member riders tend to use bikes mostly for commute as well as tasks like errands during their week.
3. Casual riders appear to use bikes either/both more for tourism or leisure in their own city. 

Let's visualize the data to see the clear bike usage trends that support these findings.

#### **1. Members favor weekdays while casual riders ride more heavily on weekends** 

![weekday-vs-weekend-breakdown-pie-chart-member-rides](https://github.com/user-attachments/assets/4c8dec89-f1ea-45c2-948b-8dab08faac93 | width=50)
![weekday-vs-weekend-breakdown-pie-chart-casual-rides](https://github.com/user-attachments/assets/677ead20-08eb-4ad1-8968-0906a6905c5c | width=50)

member_casual	|day_of_week_category	|ride_count
casual	|Weekday	|22272
casual	|Weekend	|22305
member	|Weekday	|310460
member	|Weekend	|65543
