# Cyclistic Bike-Share Analysis Case Study

## 1. Business Task

### 1.1. Background

Cyclistic is a fictional bike-sharing company, offering flexible riding plans of single-ride pass, full-day pass and annual membership. Single-ride and full-day pass users are considered as casual riders, annual membership holders are considered as Cylistic members.

The company found that annual members could contribute more profits than causual users. They believed that converting more casual users into annual members could lead their business to grow continuously. They planned to develop marketing strategies to achieve that goal.

### 1.2. Task

In order to guide the development of the marketing program, as a data analyst in the marketing team, I was assigned a task to figure out the difference between the usage of annual members and casual riders.

### 1.3. Key Stakeholder

-   Director of Marketing
-   Cyclistic Executive Team



## 2. Data Preparation

### 2.1. Data Source

Cyclistic's historical trip data in July, 2023.

### 2.2. Data Storage

The data was made available by Motivate International Inc. under sharing license.

Data Source Link: https://divvy-tripdata.s3.amazonaws.com/index.html

### 2.3. Data Credibility

The data is reliable and original because it is a first-party dataset. 

The data is comprehensive with information, such as bike type, usage duration, start/end location. It has more than 760,000 records.

The data is current, because it is updated regularly and the July 2023 dataset is the lastest one on the server.



## 3. Data Process

### 3.1. Choose Tools

MS Excel, SQL (with Google BigQuery), RStudio

### 3.2. Process Steps

-   Openned the data file (.csv) with MS Excel.

-   Checked the file for integrity. The data was in good condition. Some values in column start/end_station_name and column start/end_station_id. Meanwhile, values in column start/end_station_id were in different naming convention.

-   Formatted the column started_at and ended_at as yyyy/m/d h:mm:ss

-   Created a new column ride_length and formatted as hh:mm:ss. Calculated each value with column ended_at minus column started_at.

-   Created a new column day_of_week. Used the =WEEKDAY command to calculate the weekday from column started_at. Formatted the column as number without decimal, so that 1 means Sunday and 7 means Saturday.

-   Randomly sampling records.

    ```markdown
    Step1: Created a new column "tokeep".
    
    Step2: Inputed the code in Excel cell to randomly select records to analyze.
           =if(rand()>0.9987,"keep","")
    
    Step3: Copied and pasted cell value in column "tokeep" to a new column.
    
    Step4: Dropped the orginal "tokeep" column.
    
    Step5: Filtered rows with value "keep" in column "tokeep".
    
    Step6: Copied all filtered rows to new sheet, with 946 records.
    ```
    
-   Sorted the data by column started_at, in ascending order.

-   Saved the csv file.



## 4. Data Analysis with SQL

Uploaded the csv file to Google BigQuery.

### 4.1. Count total rows

```sql
SELECT count(*) FROM `project.dataset.table`
# 946
```

### 4.2. Count numbers of bike types

```sql
SELECT rideable_type, count(rideable_type) as total_num
FROM `project.dataset.table`
GROUP BY rideable_type
```

| **rideable_type** | **total_num** |
| ----------------- | ------------- |
| electric_bike     | 472           |
| classic_bike      | 450           |
| docked_bike       | 24            |

```markdown
Electric and classic are the most popular bikes.
```

### 4.3. Show members preference for bike types

```sql
SELECT 
  member_casual as member_type,
  count(case rideable_type when "electric_bike" then "electric_bike" END) as electric_bike,
  count(case rideable_type when "classic_bike" then "classic_bike" END) as classic_bike,
  count(case rideable_type when "docked_bike" then "docked_bike" END) as docked_bike,
FROM `project.dataset.table`
GROUP BY member_casual
```

| **member_type** | **electric_bike** | **classic_bike** | **docked_bike** |
| --------------- | ----------------- | ---------------- | --------------- |
| casual          | 205               | 185              | 24              |
| member          | 267               | 265              | 0               |

```markdown
Both members have similar interest in electric bikes.
Annual members prefer classic bikes more than casual users.
```

### 4.4. Count ride length for different member type

```sql
SELECT 
	member_casual as member_type, 
	time(timestamp_millis(cast(avg(datetime_diff(ended_at,started_at,millisecond)) as int64))) as avg_ride_length, 
    min(ride_length) as min_ride_length, 
    max(ride_length) as max_ride_length,
FROM `project.dataset.table`
GROUP BY member_casual
```

| **member_type** | **avg_ride_length** | **min_ride_length** | **max_ride_length** |
| --------------- | ------------------- | ------------------- | ------------------- |
| casual          | 00:22:49.710000     | 00:00:02            | 03:31:00            |
| member          | 00:12:34.060000     | 00:00:02            | 03:41:41            |

```markdown
The table shows that casual users have longer average ride length than annual members.
```

### 4.5. Count weekday frequency for two types of users

```sql
SELECT day_of_week,
count(day_of_week) as casual_weekday,
FROM `project.dataset.table`
WHERE member_casual = "casual"
GROUP BY day_of_week
ORDER BY day_of_week
```

| **day_of_week** | **casual_weekday** |
| --------------- | ------------------ |
| 1               | 86                 |
| 2               | 53                 |
| 3               | 55                 |
| 4               | 32                 |
| 5               | 45                 |
| 6               | 57                 |
| 7               | 86                 |

```sql
SELECT day_of_week,
count(day_of_week) as member_weekday,
FROM `project.dataset.table`
WHERE member_casual = "member"
GROUP BY day_of_week
ORDER BY day_of_week
```

| **day_of_week** | **member_weekday** |
| --------------- | ------------------ |
| 1               | 72                 |
| 2               | 92                 |
| 3               | 73                 |
| 4               | 55                 |
| 5               | 81                 |
| 6               | 61                 |
| 7               | 98                 |

```markdown
It is clear that casual users prefer to rent bike on Weekends, and annual member prefer to rent on Monday and Saturday. Both types of users rent with the least frequency on Wednesday.

The reason behind the scenes may be casual users like to use Cyclistic service when having fun outside during weekends, and annual members subscribe the service for work commuting.

Compare the result with the members preference for bike types and ride length. We can find that electric bikes are popular in both members types. It may because people like to ride longer for leisure during weekends. When it comes to work commuting, annual members prefer classic bike for short-time ride.
```



## 5. Data Visualization

Find viz in file "/R_Markdown/Cyclistic Case Study Viz.Rmd" and "/R_Markdown/Cyclistic Case Study Viz.html"



## 6. Action to Take

To achieve the business task, which is tranfering casual users to annual members, I have the following three recommendations to apply in the marketing strategies.

-   Encourage casual users to use bike-rent service for work commuting.
-   Encourage casual users to use bike-rent service for short-time ride.
-   Encourage casual users to try on classic bike. We can demonstrate that classic bike is more healthier.
