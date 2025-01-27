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

## **Defining The Problem**

**Business Task –** <br /> 
Analyze Cyclistic historical bike trip data to understand how annual members and casual riders use Cyclistic bikes differently.

**Key Stakeholders –**
- Lily Moreno – the director of marketing (my manager)
- Executive Team – the team deciding whether to approve the recommended marketing program

<br />

## **Data Collection**

- **Data Source–** Data used in this analysis is internal to the company and can be found here: https://divvy-tripdata.s3.amazonaws.com/index.html
- **Data License –** Licensing info can be found in the data license agreement that makes it available for public use: https://divvybikes.com/data-license-agreement
- **Data Set –** For this study, I chose to use Q1 2020 data due to its accuracy and completeness and thanks to its robust nature, consisting of hundreds of thousands of records.
- **Data Storage –** Data stored locally for the duration of the project as well as in BigQuery where SQL analysis was conducted.

<br />

## **Data Processing**

I've conducted my data processing and cleaning in Excel. The dataset was largely clean and complete. However, via sorting, filtering, verifying and transforming data types, and creating new data points and fields, I was able to ensure data integrity and cleanliness, as well as set up the additional fields that would provide for robust and streamlined upcoming analysis stages.

#### **Data Cleaning - Steps Taken**

1. **Applied a filter** to the dataset for streamlined dataset assessment.
2. **Used Excel's “Remove Duplicates” tool** to check for duplicate values (no duplicates found).
3. **Checked all columns for blank fields**, removing the 1 record found with a blank field (`end_station_name` field).
4. **Checked for outliers to ensure data isn't skewed** by sorting `ride_duration_hrs` ride duration column (created in dataset augmentation section below):<br />
     a. Removed 2,500 records with bike rental duration of zero minutes.
     b. Removed 41 records bike rental duration longer than 366 hours (14 days).
     c. Removed 3,766 records with a ride route of *HQ QR - HQ QR* (ride started & finished at what appears to be Cyclistic headquarters), as these rides were almost all <1 minute or had a negative duration (end time was before start time). 
6. **Used formula `=COUNTA(UNIQUE())` to check distinct counts** of station names against station IDs to check for errant additional ID or name values (no issues found).
7. **Used formula `=LEN()` to check fields for uniform length** (fields checked: `ride_id`, `rideable_type`, `start_station_id`, `end_station_id`, and `member_casual`). No `=TRIM()` required.
8. **Checked data formats**, setting a uniform Date format in the `started_at` and `ended_at` columns to ensure uniformity.

