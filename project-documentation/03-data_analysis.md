<h1>Tools</h1>
For the purpose of further analysis of the bike hire dataset, I decided to use Posit Cloud Studio (formerly known as R Studio) which relies on the programming language R.<br><br>
The reasons for the decision to use R include the following:<br>
<li>R enforces a structured approach to the analysis, as each step needs to be appropriately coded in the programming language,</li>
<li>Analysis is thoroughly documented and easily reproducible, analysis steps and code can be reviewed by others,</li>
<li>R includes highly specialised packages for data analysis that enable all types of transformations to datasets,</li>
<li>As opposed to Excel, R can easily and quickly process large amounts of data,</li>
<li>As opposed to SQL databases (e.g. Big Query), R has capabilities to produce varied and advanced visualisations,</li>
<li>Through usage of dedicated packages, Posit Cloud Studio can connect to Big Query and access data stored on Google Big Query Cloud through API.<br>

Key R packages installed and loaded during the analysis include:<br>
<li>tidyverse (incl. dplyr, ggplot2, tidyr and forcats),</li>
<li>skimr,</li>
<li>bigrquery,</li>
<li>DBI,</li>
<li>odbc.</li>

<h1>Data upload</h1>
As the  first step in Posit Cloud Studio, a connection was established with the Big Query Cloud where the dataset was pre-validated and cleaned in previous stages. For this purpose, dedicated functions covered in packages DBI and bigrquery were used as demonstrated in the code below.

```
con <- dbConnect(
  bigquery(),
  project = "cyclistic-analysis-oct-2023",
  dataset = "trips_2022",
  billing = "cyclistic-analysis-oct-2023"
)
```

After establishing the connection, it became possible to view tables connected from Big Query Cloud to Posit Cloud Studio.
```
dbListTables(con)
```

Further, to validate the corectness of the connection, a listing of fields in the required table from Big Query Cloud was displayed in Posit Cloud Studio, based on the function below.

```
dbListFields(con, "cyclistic_2022_dataset_clean")
```

As next step, a new table was created in Posit Cloud Studio based on the connection with the cleaned dataset in Big Query Cloud.

```
trips_2022_clean <- tbl(con,"cyclistic_2022_dataset_clean")
```

To gain full comfort over the correctness of the newly created table, functions such as head(), view(), str(), glimpse(), and skim_without_charts() were applied to the "trips_2022_clean" dataset. However, despite correct display of the number and name of columns, the functions did not show the total number of rows. This was likely due to hardware and/or RAM limitations as the dataset comprised a significant number of records (over 5.6 million rows).

Completeness check in R studio was performed by querying subsets of the table, which at the same time revealed initial insights (e.g. number of unique rides for members vs. casual users).

```
> trips_2022_clean%>%
   group_by(member_casual) %>%
   summarize(no_of_rides = n_distinct(ride_id))
```

As the output of these checks, it was confirmed that the total number of unique ride IDs is equal to the total number of rows in Big Query dataset (i.e. 5,667,186). Based on this, the connection from Google Cloud to Posit Cloud was assessed as successful. Initial insight discovered during the check was that the majority of rides (59%) are members.

|member_casual|no_of_rides|
|-------------|-----------|
|member|3345417|
|casual|2321769|



<h1>Analysis</h1>

Data analysis comprised several procedures aimed at identifying patterns in user behaviour. All the procedures and their results are documented below.

<li>Number of rides per day of week by member vs. casual</li>
<br><br>
a. Member

```
trips_2022_clean %>%
     group_by(day_of_week) %>%
     filter(member_casual=="member") %>%
     summarize(no_of_rides = n_distinct(ride_id))
```

The analysis as per the code above showed the number of rides by members on particular days of the week.

|day_of_week|no_of_rides|
|-----------|-----------|
|Wednesday|523836|
|Thursday|532215|
|Friday|467051|
|Monday|473305|
|Sunday|387180|
|Tuesday|518584|
|Saturday|443246|

b. Casual

```
trips_2022_clean %>%
     group_by(day_of_week) %>%
     filter(member_casual=="casual") %>%
     summarize(no_of_rides = n_distinct(ride_id))
```

Conversely, the analysis as per this piece of code above showed the number of rides by casual users on particular days of the week.

|day_of_week|no_of_rides|
|-----------|-----------|
|Friday|334667|
|Monday|277649|
|Sunday|388981|
|Tuesday|263706|
|Saturday|473130|
|Thursday|309297|
|Wednesday|274339|

<li>Average length of rides by member vs. casual</li>
<br><br>
a. Member

```
trips_2022_clean %>%
     group_by(rideable_type) %>%
     filter(member_casual=="member") %>%
     summarize(avg_ride_length = mean(ride_length))
```

The analysis step in the code above revealed average length of ride for each rideable type for members.

|rideable_type|avg_ride_length|   
|-------------|-----------|
|electric_bike|0-0 0 0:11:27.831325|
|classic_bike|0-0 0 0:13:54.705167|

b. Casual

```
trips_2022_clean %>%
      group_by(rideable_type) %>%
      filter(member_casual=="casual") %>%
      summarize(avg_ride_length = mean(ride_length))
```

As per this analysis step, the average length of ride for each rideable type was identified for casual users. Outputs reveal that casual users' trips on docked bikes are much longer compared to other rideables and average ride lengths are longer than for members across all rideable types.

|rideable_type|avg_ride_length|     
|-------------|-----------|
|electric_bike|0-0 0 0:16:10.564482|
|docked_bike|0-0 0 2:2:42.941888|
|classic_bike|0-0 0 0:28:45.184446|

<li>Rideable type by member vs. casual</li>
<br><br>
a. Member

```
trips_2022_clean %>%
     group_by(rideable_type) %>%
     filter(member_casual=="member") %>%
     summarise(no_of_rides = n_distinct(ride_id))
```

The code above was applied to show number of rides by rideable type for members.

|rideable_type|no_of_rides|
|-------------|-----------|
|classic_bike|1709682|
|electric_bike|1635735|

b. Casual

```
trips_2022_clean %>%
      group_by(rideable_type) %>%
      filter(member_casual=="casual") %>%
      summarise(no_of_rides = n_distinct(ride_id))
```

The code above was applied to show number of rides by rideable type for casual users. As per the outputs, the main insight is that casual users hire docked bike, whereas members do not.

|rideable_type|no_of_rides|
|-------------|-----------|
|electric_bike|1252895|
|docked_bike|177468|
|classic_bike|891406|

<h1>Visualisations</h1>

Some insights are better identified based on review of data displayed in a graphical form. For this analysis, I decided to break down members and casual users' rides per month and display the results in a time series chart.

As first step, a new data frame was created with monthly number of rides in members and casual users.

a. Members

```
members_trips_monthly <- trips_2022_clean %>% 
     mutate(
     month = month(started_at)
     ) %>% 
     group_by(month) %>%
     filter(member_casual == "member") %>%
     summarise(no_of_rides = n_distinct(ride_id))
```

b. Casual

```
casual_trips_monthly <- trips_2022_clean %>% 
     mutate(
     month = month(started_at)
     ) %>% 
     group_by(month) %>%
     filter(member_casual == "casual") %>%
     summarise(no_of_rides = n_distinct(ride_id))
```

Based on these data frames, visualisations were developed using the ggplot2 package.

a. Members

```
ggplot(data = members_trips_monthly, mapping = aes(x=month, y=no_of_rides)) + geom_col(fill="darkblue")+scale_y_continuous(labels = scales::comma_format(), breaks = seq(0, 600000, 100000)) + scale_x_continuous(breaks = seq(1,12,1))
```

The output of this code is visible below.

![image](https://github.com/kamil-michalski-1/Cyclistic-Bike-Share-Project/assets/147998053/7e467898-2884-4824-bb05-a5cd9b82371a)


b. Casual

```
ggplot(data = casual_trips_monthly, mapping = aes(x=month, y=no_of_rides)) + geom_col(fill="orange")+scale_y_continuous(labels = scales::comma_format(), breaks = seq(0, 600000, 100000)) + scale_x_continuous(breaks = seq(1,12,1))
```

The output of this code is visible below.

![image](https://github.com/kamil-michalski-1/Cyclistic-Bike-Share-Project/assets/147998053/bb677e29-4ae2-46ba-bdd0-c5066ddb599d)


<h1>Conclusion</h1>
The analysis steps documented above revealed focus areas for further development of my data analytics skills. The key area is how to handle very large datasets in R, e.g. through splitting them into more manageable chunks, applying meaningful filters, and querying subsets of the full dataset for better performance.<br><br>
Full business significance of the discovered insights and recomendations are covered in the project presentation.
