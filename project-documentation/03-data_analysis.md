<h1>Tools</h1>
Posit Cloud / R Studio with justification
what packages installed and loaded - tidyverse, bigrquery for API with Big Query projects

<h1>Data upload</h1>
bigrquery package, API through code
```
> bq_auth()
```

```
projectid<-"cyclistic-analysis-oct-2023"
sql<-"SELECT * FROM cyclistic-analysis-oct-2023.trips_2022.cyclistic_2022_dataset_clean"
df <- bq_project_query(projectid,sql) %>% bq_table_download
```

<h1>Analysis</h1>

<h1>Visualisations</h1>

<h1>Conclusion</h1>
