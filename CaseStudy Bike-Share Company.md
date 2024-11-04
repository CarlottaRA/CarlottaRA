# Case Study Bike-Share Company
In 2016, Cyclistic launched a successful bike-share o ering. Since then, the program has grown
 to a 5,824 bicycles that are geotracked and locked into a network of 692 stations
across Chicago. 
type of memberships: single-ride passes (casual), full-day passes (casual), and annual memberships.
## The Goal: Convert casual users into annual members
### Question:  How do annual members and casual riders use Cyclistic bikes diferently?
The data has been made available by Motivate International Inc.( 
<https://divvybikes.com/data-license-agreement>).
The data is formated primarily in to .csv documents that I'll be using in this case study
specificly the **Divvy 2019 Q1** and **Divvy 2020 Q1** datasets.
To be able to interact better with the data I will move to SQL
To clean the data know in SQL we uplode the original files into bigquery 

#### Clean and organize the data
* Both files where modified to have the same column names, and some where ignored due to no matching column in the other dataset and it being no relevant data.
* Data about membership was modified to be in the same terms.
* Distinct funtion was used to prevent duplacates.
* Metrics such as ride_lenght and day_of_week where introduced.
* Credibility was assured by filtering.
* The data was merged into one dataset. 

##### Cleaning of Divvy 2019 Q1
```{sql}
CREATE TABLE IF NOT EXISTS bikes.Quarter12019
as 
SELECT DISTINCT trip_id as ride_id, start_time as  started_at, end_time as ended_at,from_station_id as start_station_id,
from_station_name as start_station_name,
to_station_id as end_station_id, to_station_name as end_station_name, 
(case when usertype='Subscriber'then 'member' when usertype='Customer' then 'casual' end) as member_casual,
TIMESTAMP_DIFF(end_time, start_time, SECOND) AS ride_lenght,
FORMAT_TIMESTAMP('%A', start_time) AS day_of_week
FROM `bikesharecompany-440704.bikes.Divvy_Trips_2019_Q1` 
WHERE TIMESTAMP_DIFF(end_time, start_time, SECOND) <>0
AND end_time IS NOT NULL
```
##### Cleaning of Divvy 2020 Q1
```{sql}
CREATE TABLE IF NOT EXISTS bikes.Quarter12020
as 
SELECT DISTINCT ride_id, started_at,ended_at,start_station_id,start_station_name,
end_station_id,end_station_name,member_casual,
TIMESTAMP_DIFF(ended_at, started_at, SECOND) AS ride_lenght,
FORMAT_TIMESTAMP('%A', started_at) AS day_of_week
FROM `bikesharecompany-440704.bikes.Divvy_Trips_2020_Q1`
WHERE TIMESTAMP_DIFF(ended_at, started_at, SECOND)<>0
AND ended_at IS NOT NULL
ORDER BY started_at
```
##### The data was merged into one table, bikes.Quarter12019_2020
```{sql}
CREATE TABLE bikes.Quarter12019_2020 AS
SELECT CAST(ride_id AS STRING) as ride_id,started_at,ended_at,start_station_id,start_station_name,end_station_id,end_station_name,member_casual,ride_lenght,day_of_week FROM bikesharecompany-440704.bikes.Quarter12019
UNION  ALL
SELECT * FROM bikesharecompany-440704.bikes.Quarter12020;
```
Then I drop the first two tables to be efficient with space

```{sql}
drop table bikes.Quarter12019
```
```{sql}
drop table bikes.Quarter12020
```
### Analize 
Now is time to start anwering the question; **How do annual members and casual riders use Cyclistic bikes diferently?** to answer this I'll construct diferent querys with the information given.

#### Avarage ride lenght for each type of membership
```{sql}
SELECT member_casual,avg(ride_lenght) as avg_ride_lenght, 
CONCAT(
    LPAD(CAST(FLOOR(CAST(avg(ride_lenght)as int64) / 3600) AS STRING), 2, '0'), ':',
    LPAD(CAST(FLOOR(mod( cast(avg(ride_lenght) as int64), 3600) / 60) AS STRING), 2, '0'), ':',
    LPAD(CAST(mod(cast(avg(ride_lenght)as int64) , 60) AS STRING), 2, '0')
  ) AS formated_time
FROM bikesharecompany-440704.bikes.Quarter12019_2020
group by member_casual
```


| member_casual | avg_ride_length       | formatted_time |
|---------------|-----------------------|----------------|
| casual        | 37772.2079795431      | 10:29:32       |
| member        | 4389.3426657019245    | 01:13:09       |

This shows a very infated result in de casual riders wich leads me to belive ther are some data entries of casual members were their rides where days long.
To see a true vison of the avrage user I will exclude entries with more than a 3 day ride. 
```{sql}
SELECT member_casual,avg(ride_lenght) as avg_ride_lenght, 
CONCAT(
    LPAD(CAST(FLOOR(CAST(avg(ride_lenght)as int64) / 3600) AS STRING), 2, '0'), ':',
    LPAD(CAST(FLOOR(mod( cast(avg(ride_lenght) as int64), 3600) / 60) AS STRING), 2, '0'), ':',
    LPAD(CAST(mod(cast(avg(ride_lenght)as int64) , 60) AS STRING), 2, '0')
  ) AS formated_time
FROM bikesharecompany-440704.bikes.Quarter12019_2020
where ride_lenght<259200
group by member_casual
```
| member_casual | avg_ride_length       | formatted_time |
|---------------|-----------------------|----------------|
| casual        | 511.59599840996543    | 00:08:32       |
| member        | 622.69042871856914    | 00:10:23       |

Now we can see a big difference from the previuse table wich tells us a whole different story. The previus table was usefull to know one difference 
* Casual riders are more likely to take the bike for more multiple days
This table shows that the avarage time is not that different but
* Casual riders have shorter trips by about two minutes

#### Ride count for each type of membership
```{sql}
SELECT member_casual,
count(ride_id) as total_count_rides
FROM bikesharecompany-440704.bikes.Quarter12019_2020 
group by member_casual
```
| member_casual | total_count_rides |
|---------------|-------------------|
| casual        | 30503             |
| member        | 317890            |

#### Ride count for each type of membership, for each day of the week
```{sql}
select day_of_week, count(case when member_casual='casual' then ride_id end) as count_casual,
 count(case when member_casual='member' then ride_id end) as count_member 
from bikes.Quarter12019_2020
group by day_of_week
```
| day_of_week | count_casual | count_member |
|-------------|--------------|--------------|
| Saturday    | 2617         | 39322        |
| Monday      | 7554         | 52390        |
| Friday      | 6819         | 46628        |
| Thursday    | 3169         | 48697        |
| Wednesday   | 3654         | 38431        |
| Tuesday     | 2834         | 50486        |
| Sunday      | 3856         | 41936        |

This tells us thant 
* There are significatly less rides by casual member
* Highest days for Casual riders is Monday and Friday
* Highest days form members are Monday and Tuesday

  
