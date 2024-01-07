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

creating a table in R studio
```
trips_2022_clean <- tbl(con,"cyclistic_2022_dataset_clean")
```

performance issue, although data shown correctly in head() and view(), number of rows not shown in str(), glimpse(),
skim() wouldn't load, possibly due to size of the dataset comprising annual data (5.6m records)

resolved by


<h1>Analysis</h1>

<h1>Visualisations</h1>

<h1>Conclusion</h1>
