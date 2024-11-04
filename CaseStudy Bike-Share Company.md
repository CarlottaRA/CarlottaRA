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

#### Clean and organize the data
* Both files where modified to have the same column names, and some where deleted due to no matching column in the other dataset and it being no relevant data.
* Data about membership was modified to be in the same terms.
* No duplicates where found.
* Metrics such as ride_lenght and day_of_week where introduced.
* Credibility was assured by filtering and conditional formating,
* Two issues were found, 
    one data entry with no end of trip data,
    209 data entries with no diference between start time and end time,
    all of them where expluded from this analysis
* The data was merged into one dataset. 

#### Analize 

Pivot tables where used to see more into the data 

The avarage ride lenght for the diferent types of members
| Type_membership      | avg_ride_lenght   |
|----------------|----------|
| Casual         | 1:25:05  |
| Member         | 0:13:15  |

The avarage ride lenght per day of the week for the diferent types of members
|Avg_ride_lenght/day_of_week |Sunday      | Monday        |Tuesday        | Wednesday       |Thursday         | Friday       | Saturday       |
|-------------------|---------|---------|---------|---------|---------|---------|---------|
| Casual            | 1:24:21 | 1:06:10 | 1:09:44 | 1:08:40 | 2:09:33 | 1:35:37 | 1:22:31 |   
| Member            | 0:16:13 | 0:13:42 | 0:12:49 | 0:11:52 | 0:11:47 | 0:13:17 | 0:16:14 |    

The avarage ride lenght and ride count per month for the diferent types of members
|Avg_ride_lenght/month | Jan       | Feb       | March       |
|-------------------|---------|---------|---------|
| Casual            | 1:59:11 | 2:10:41 | 0:59:10 |
| Member            | 0:13:02 | 0:13:04 | 0:13:36 |

| Etiquetas de fila | Jan       | Feb       | March       |
|-------------------|--------|--------|--------|
| Casual            | 12387  | 15498  | 43548  |
| Member            | 234769 | 220263 | 265281 |


And finally the count of rides per day of the week
|sum_rides/day_of_week |Sunday      | Monday        |Tuesday        | Wednesday       |Thursday         | Friday       | Saturday        | Total |
|-------------------|--------|--------|--------|--------|--------|--------|--------|---------------|
| Casual            | 18652  | 6694   | 7972   | 8363   | 7771   | 8508   | 13473  | 71433         |
| Member            | 60197  | 110430 | 127974 | 121903 | 125228 | 115168 | 59413  | 720313        |

From this we get an idea in whish aspects do the casual members differ from de annual members, and we can see that
* Casual riders have longer ride lenght
* Annual members ride more trips than the casuals.
* Casual members ride more on the weekends

To be able to interact better with the data I will move to SQL
