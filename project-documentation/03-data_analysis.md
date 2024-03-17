<h1>Tools</h1>
For the purpose of further analysis of the bike hire dataset, I decided to use Posit Cloud Studio (formerly known as R Studio) which relies on the programming language R.<br><br>
The reasons for the decision to use R include the following:<br>
<li>R enforces a structured approach to the analysis, as each step needs to be appropriately coded in the programming language,</li>
<li>Analysis is thoroughly documented and easily reproducible, analysis steps and code can be reviewed by others,</li>
<li>R includes highly specialised packages for data analysis that enable all types of transformations to datasets,</li>
<li>As opposed to Excel, R can easily and quickly process large amounts of data,</li>
<li>As opposed to SQL databases (e.g. Big Query) R has capabilities to produce varied and advanced visualisations,</li>
<li>Through usage of dedicated packages, Posit Cloud Studio can connect to Big Query and access data stored on Google Big Query Cloud through API.<br>

Key R packages installed and loaded during the analysis include: tidyverse (incl. dplyr and ggplot2), bigrquery, DBI and odbc.

<h1>Data upload</h1>
establishing connection

```
con <- dbConnect(
  bigquery(),
  project = "cyclistic-analysis-oct-2023",
  dataset = "trips_2022",
  billing = "cyclistic-analysis-oct-2023"
)
```

viewing tables in connection
```
dbListTables(con)
```

viewing fields in the table

```
dbListFields(con, "cyclistic_2022_dataset_clean")
```

creating a table in Posit Cloud studio

```
trips_2022_clean <- tbl(con,"cyclistic_2022_dataset_clean")
```

performance issue, although data shown correctly in head() and view(), number of rows not shown in str(), glimpse(),
skim() wouldn't load, possibly due to size of the dataset comprising annual data (5.6m records), possibly due to hardware limitations

completeness check in R studio done by querying subsets of the table, at the same time getting initial insights - number of unique rides per membership vs. casual

```
> trips_2022_clean%>%
   group_by(member_casual) %>%
   summarize(no_of_rides = n_distinct(ride_id))
```

Output - total number of unique ride IDs equals the total number of rows in Big Query dataset (i.e. 5,667,186), therefore connection from Google Cloud to Posit Cloud assessed as successful. Also initial insight - majority of rides (59%) are members.

|member_casual|no_of_rides|
|-------------|-----------|
|member|3345417|
|casual|2321769|



<h1>Analysis</h1>

1. no. of rides per day of week by member vs. casual

a. member

```
trips_2022_clean %>%
     group_by(day_of_week) %>%
     filter(member_casual=="member") %>%
     summarize(no_of_rides = n_distinct(ride_id))
```

output

|day_of_week|no_of_rides|
|-----------|-----------|
|Wednesday|523836|
|Thursday|532215|
|Friday|467051|
|Monday|473305|
|Sunday|387180|
|Tuesday|518584|
|Saturday|443246|

b. casual

```
trips_2022_clean %>%
     group_by(day_of_week) %>%
     filter(member_casual=="casual") %>%
     summarize(no_of_rides = n_distinct(ride_id))
```

output

|day_of_week|no_of_rides|
|-----------|-----------|
|Friday|334667|
|Monday|277649|
|Sunday|388981|
|Tuesday|263706|
|Saturday|473130|
|Thursday|309297|
|Wednesday|274339|

2. average length of rides by member vs. casual

a. member

```
trips_2022_clean %>%
     group_by(rideable_type) %>%
     filter(member_casual=="member") %>%
     summarize(avg_ride_length = mean(ride_length))
```

output

|rideable_type|avg_ride_length|   
|-------------|-----------|
|electric_bike|0-0 0 0:11:27.831325|
|classic_bike|0-0 0 0:13:54.705167|

b. casual

```
trips_2022_clean %>%
      group_by(rideable_type) %>%
      filter(member_casual=="casual") %>%
      summarize(avg_ride_length = mean(ride_length))
```

output - trips on docked bikes are much longer

|rideable_type|avg_ride_length|     
|-------------|-----------|
|electric_bike|0-0 0 0:16:10.564482|
|docked_bike|0-0 0 2:2:42.941888|
|classic_bike|0-0 0 0:28:45.184446|

3. rideable_type by member vs. casual

a. member

```
trips_2022_clean %>%
     group_by(rideable_type) %>%
     filter(member_casual=="member") %>%
     summarise(no_of_rides = n_distinct(ride_id))
```

output

|rideable_type|no_of_rides|
|-------------|-----------|
|classic_bike|1709682|
|electric_bike|1635735|

b. casual

```
trips_2022_clean %>%
      group_by(rideable_type) %>%
      filter(member_casual=="casual") %>%
      summarise(no_of_rides = n_distinct(ride_id))
```

output - insight is that casuals used docked bike, whereas members don't

|rideable_type|no_of_rides|
|-------------|-----------|
|electric_bike|1252895|
|docked_bike|177468|
|classic_bike|891406|

<h1>Visualisations</h1>

Certain insights better visible in graphical form, decided to show analyse differences in monthly distribution of rides between members and casual, and show them in a time series chart

First a new data frame was created from monthly statistics on number of rides in members and casual

a. members

```
members_trips_monthly <- trips_2022_clean %>% 
     mutate(
     month = month(started_at)
     ) %>% 
     group_by(month) %>%
     filter(member_casual == "member") %>%
     summarise(no_of_rides = n_distinct(ride_id))
```

b. casual

```
casual_trips_monthly <- trips_2022_clean %>% 
     mutate(
     month = month(started_at)
     ) %>% 
     group_by(month) %>%
     filter(member_casual == "casual") %>%
     summarise(no_of_rides = n_distinct(ride_id))
```

Based on these data frames visualisations were developed using the ggplot2 package.

a. members

```
ggplot(data = members_trips_monthly, mapping = aes(x=month, y=no_of_rides)) + geom_col(fill="darkblue")+scale_y_continuous(labels = scales::comma_format(), breaks = seq(0, 600000, 100000)) + scale_x_continuous(breaks = seq(1,12,1))
```

output
![image](https://github.com/kamil-michalski-1/Cyclistic-Bike-Share-Project/assets/147998053/7e467898-2884-4824-bb05-a5cd9b82371a)


b. casual

```
ggplot(data = casual_trips_monthly, mapping = aes(x=month, y=no_of_rides)) + geom_col(fill="orange")+scale_y_continuous(labels = scales::comma_format(), breaks = seq(0, 600000, 100000)) + scale_x_continuous(breaks = seq(1,12,1))
```

output
![image](https://github.com/kamil-michalski-1/Cyclistic-Bike-Share-Project/assets/147998053/bb677e29-4ae2-46ba-bdd0-c5066ddb599d)


<h1>Conclusion</h1>


