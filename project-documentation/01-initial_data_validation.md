<h1>Data source</h1>
Data was downloaded from the link below. The repository covered Chicago's Divvy bike-sharing service trip records since January 2013 until today (download date - 15 October 2023). For the purpose of my analysis, full data on 2022 trips were selected as a representative period (to account for seasonal fluctuations).<br>

[https://divvy-tripdata.s3.amazonaws.com/index.html](https://divvy-tripdata.s3.amazonaws.com/index.html)

<h1>Data upload</h1>
Initial data validation was conducted in Google Big Query. In order to upload the data to Big Query workspace, the following steps were completed:
  <li>Data was downloaded in 12 separate CSV files with monthly records</li>
  <li>For ease of batch upload to Big Query, a dedicated bucket was created in Google Cloud Storage service where all the monthly files were uploaded</li>
  <li>Monthly CSV files from the bucket were loaded to Big Query to create a single table with full 2022 data</li>
  <li>It was identified that the CSV file with September 2022 data was omitted in the initial upload due to a different naming convention. To remediate this, the September 2022 file was uploaded into a separate table in Big Query workspace and inserted into the comprehensive 2022 table using the query below</li>
  
```sql
  INSERT INTO `trips_2022.trips_2022`
  SELECT *
  FROM `trips_2022.trips_202209`
```

<li>Dataset schema was reviewed using an in-built Big Query functionality. The schema with data types is presented in the table below.</li>

|Field name|Type|
|:--------:|:--:|
|ride_id   |STRING|
|rideable_type|STRING|	
|started_at|TIMESTAMP|
|ended_at|TIMESTAMP|
|start_station_name|STRING|
|start_station_id|STRING|
|end_station_name|STRING|
|end_station_id|STRING|
|start_lat|FLOAT|
|start_lng|FLOAT|
|end_lat|FLOAT|
|end_lng|FLOAT|
|member_casual|STRING|

<h1>Initial data validation</h1>
Checks were conducted over the uploaded dataset to identify potential data quality issues and confirm data reliability.
The following aspects were validated:
  <li>Duplicate ride IDs - query as per below. No duplicates were identified - number of unique ride IDs was equal to the total number of records.</li>

```sql
  SELECT COUNT(ride_id) AS all_ride_ids, COUNT (DISTINCT ride_id) AS unique_ride_ids
  FROM `trips_2022.trips_2022`
```

|Row|all_ride_ids|unique_ride_ids|
|:-:|:----------:|:--------------:|
|1  |5667717     |5667717         |


  <li>Min. and max. values of bike rental start and end dates were identified - query as per below. Values were in line with expectations, e.g. minimum start and end dates 1 January 2022, maximum start date 31 December 2022, maximum end day 2 January 2023 (this was considered correct due to the possible occurrence of extended rentals commenced on 31 December and not closed until after New Year's Day - these trips were kept in the dataset).</li>

```sql
  SELECT CAST(MIN(started_at) AS date) AS min_started_at,
  CAST(MAX(started_at) AS date) AS max_started_at,
  CAST(MIN(ended_at) AS date) AS min_ended_at,
  CAST(MAX(ended_at) AS date) AS max_ended_at
  FROM `trips_2022.trips_2022`
```


|Row|min_started_at|max_started_at|min_ended_at|max_ended_at|
|---|:------------:|:------------:|:----------:|:----------:|
|1	|2022-01-01    |2022-12-31    |  2022-01-01|2023-01-02  |


<li>Numbers of rides in particular months were obtained - query as per below. This helped assure that the source datasets included complete data covering all the months in 2022 and that the variability between months remains within a reasonable range that can be explained with seasonal trends. No irregularities were noted.</li>

```sql
  SELECT EXTRACT(MONTH FROM started_at) AS started_at_month,
  COUNT(*) AS number_of_rides
  FROM `trips_2022.trips_2022`
  GROUP BY started_at_month
  ORDER BY started_at_month
```

|Row|started_at_month|number_of_rides|
|---|:--------------:|:-------------:|
|1	|1               |103770         |
|2	|2               |115609         |
|3  |3               |284042         |
|4  |4               |371249         |
|5  |5               |634858         |
|6  |6               |769204         |
|7	|7               |823488         |
|8	|8               |785932         |
|9  |9               |701339         |
|10 |10              |558685         |
|11	|11              |337735         |
|12	|12              |181806         |

<li>A query was run to identify blank values in any column in the table - see code below. The query returned 1,298,357 records, which revealed potential data integrity issues requiring further investigation in next steps.</li>

```sql
  SELECT *
  FROM `trips_2022.trips_2022`
  WHERE ride_id IS NULL OR
  rideable_type IS NULL OR
  started_at IS NULL OR ended_at IS NULL OR
  start_station_name IS NULL OR
  start_station_id IS NULL OR
  end_station_name IS NULL OR
  end_station_id IS NULL OR
  start_lat IS NULL OR
  start_lng IS NULL OR
  end_lat IS NULL OR
  end_lng IS NULL OR
  member_casual IS NULL
```
<li>A more detailed query was run to identify the incidence of blanks, incl. which fields and in what months had blank values. The query showed that the blanks are found in columns with station names, station IDs and geodata across all the months in the dataset.</li>

```sql
  SELECT
  EXTRACT(month FROM started_at) AS month,
  SUM(CASE WHEN ride_id IS NULL THEN 1 ELSE 0 END) AS ride_id_null_count,
  SUM(CASE WHEN rideable_type IS NULL THEN 1 ELSE 0 END) AS rideable_type_null_count,
  SUM(CASE WHEN started_at IS NULL THEN 1 ELSE 0 END) AS started_at_null_count,
  SUM(CASE WHEN ended_at IS NULL THEN 1 ELSE 0 END) AS ended_at_null_count,
  SUM(CASE WHEN start_station_name IS NULL THEN 1 ELSE 0 END) AS start_station_name_null_count,
  SUM(CASE WHEN start_station_id IS NULL THEN 1 ELSE 0 END) AS start_station_id_null_count,
  SUM(CASE WHEN end_station_name IS NULL THEN 1 ELSE 0 END) AS end_station_name_null_count,
  SUM(CASE WHEN end_station_id IS NULL THEN 1 ELSE 0 END) AS end_station_id_null_count,
  SUM(CASE WHEN start_lat IS NULL THEN 1 ELSE 0 END) AS start_lat_null_count,
  SUM(CASE WHEN start_lng IS NULL THEN 1 ELSE 0 END) AS start_lng_null_count,
  SUM(CASE WHEN end_lat IS NULL THEN 1 ELSE 0 END) AS end_lat_null_count,
  SUM(CASE WHEN end_lng IS NULL THEN 1 ELSE 0 END) AS end_lng_null_count,
  SUM(CASE WHEN member_casual IS NULL THEN 1 ELSE 0 END) AS member_casual_null_count
  FROM `trips_2022.trips_2022`
  GROUP BY month
  ORDER BY month
```


|Row|month|ride_id_null_count|rideable_type_null_count|started_at_null_count|ended_at_null_count|start_station_name_null_count|start_station_id_null_count|end_station_name_null_count|end_station_id_null_count|start_lat_null_count|start_lng_null_count|end_lat_null_count|end_lng_null_count|member_casual_null_count|
|:-:|:-:|:-:|:-:|:-:|:-:|:---:|:---:|:---:|:---:|:-:|:-:|:-:|:-:|:-:|
|1	|1  |0  |0  |0  |0  |16260|16260|17927|17927|0  |0  |86 |86 |0  |
|2	|2  |0  |0  |0  |0  |18580|18580|20355|20355|0  |0  |77 |77 |0  |
|3	|3  |0  |0  |0  |0  |47246|47246|51157|51157|0  |0  |266|266|0  |
|4	|4  |0  |0  |0  |0  |70887|70887|75288|75288|0  |0  |317|317|0  |
|5	|5  |0  |0  |0  |0  |86704|86704|93171|93171|0  |0  |722|722|0  |
|6  |6  |0  |0  |0  |0  |92944|92944|100152|100152|0|0|1055|1055|0  |
|7  |7  |0  |0  |0  |0  |112031|112031|120951|120951|0|0|947|947|0  |
|8  |8  |0  |0  |0  |0  |112037|112037|120522|120522|0|0|843|843|0  |
|9  |9  |0  |0  |0  |0  |103780|103780|111185|111185|0|0|712|712|0  |
|10 |10 |0  |0  |0  |0  |91355 |91355 |96617 |96617 |0|0|475|475|0  |
|11 |11 |0  |0  |0  |0  |51957 |51957 |54259 |54259 |0|0|230|230|0  |
|12 |12 |0  |0  |0  |0  |29283 |29283 |31158 |31158 |0|0|128|128|0  |

<h1>Conclusions</h1>
Based on initial validation procedures it was confirmed that the dataset includes details of bike trips in line with the intended period of analysis covering the entire 2022 with no duplicated records. As an area of concern from the point of view of data quality, it was identified that station IDs, names or geodata are missing in a significant proportion of records - 23% of all rides (1,298,357 out of 5,667,717). While the analysis doesn't aim to provide insights on routes or geography of bike-share usage, incomplete geodata lower the overall level of confidence in the integrity of the dataset.<br><br>In a real-life scenario this discovery would prompt further validation with involvement of business process owners and data engineers to understand the root causes and assess impact on remaining data. Such follow-up procedures would aim to enrich the dataset (if possible), gain full comfort on the reliability of data in remaining columns with no blanks, and help identify improvements in the business process, technology or data engineering that would prevent this from re-occurring.
<br><br>For the purposes of this capstone project, the dataset will be further analysed to draw insights on user behaviour and columns with blank values will be excluded at further stages of analysis. An appropriate disclosure will be included in the summary presentation and brought to stakeholders' attention.
