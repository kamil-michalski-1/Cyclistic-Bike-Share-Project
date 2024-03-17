<h1>Tools</h1>
Posit Cloud / R Studio with justification
what packages installed and loaded - tidyverse, bigrquery for API with Big Query projects, dbplyr, skimr, DBI, odbc

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

Output - total number of unique ride IDs equals the total number of rows in Big Query dataset (i.e. 5,667,186)

|member_casual|no_of_rides|
|-------------|-----------|
|member|3345417|
|casual|2321769|

<h1>Analysis</h1>

<h1>Visualisations</h1>

<h1>Conclusion</h1>
