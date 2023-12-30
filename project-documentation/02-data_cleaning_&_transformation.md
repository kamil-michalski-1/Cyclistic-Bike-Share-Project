<h1>Data cleaning</h1>
Further procedures were conducted to confirm the integrity of data and better understand its structure and potential limitations. The aspects reviewed included:
<li>Check what rideable types are visible in the dataset and confirm that all trips had a meaningful rideable type indicated. Three rideable types were noted with no incorrect or unexpected values in the dataset.</li>

```sql
  SELECT rideable_type, COUNT(ride_id) AS trips_per_rideable_type
  FROM `trips_2022.trips_2022`
  GROUP BY rideable_type
```
|Row|rideable_type|trips_per_rideable_type|
|:-:|:-:|:-:|
|1|classic_bike|2601214|
|2|docked_bike|177474|
|3|electric_bike|2889029|

<li>Values of member vs. casual data were examined to check for incorrect or unexpected values. No irregularities were noted in the dataset.</li>

```sql
  SELECT member_casual, COUNT(ride_id) as trips_per_user_type
  FROM `trips_2022.trips_2022`
  GROUP BY member_casual
```

|Row|member_casual|trips_per_user_type|
|:-:|:-:|:-:|
|1|member|3345685|
|2|casual|2322032|

<li>The dataset was examined for correctness of trip start and end times. For this purpose maximum and minimum ride times were identified based on a calculation from start and end timestamps. The minimum ride time had a negative value, which pointed to potential issues in data integrity. Further procedures to understand the incidence of negative ride times and impact on the reliability of the dataset were conducted in next steps.</li>

```sql
  SELECT MAX(ended_at - started_at) AS max_ride_length,
  MIN(ended_at - started_at) AS min_ride_length
  FROM `trips_2022.trips_2022`
```

|Row|max_ride_length|min_ride_length|
|:-:|:-:|:-:|
|1|0-0 0 689:47:15|0-0 0 -172:33:21|

<li>The dataset was queried to identify and review all records with negative ride times - as per code below. This revealed that only 100 records had ride times with a negative value. No pattern was detected for entries with negative ride times, e.g. against days of the week, rideable types or user types (member vs. casual).</li>

```sql
  SELECT ride_id, rideable_type, started_at,ended_at, member_casual,
  (ended_at - started_at) AS ride_length, FORMAT_DATE('%A', started_at) AS day_of_week
  FROM `trips_2022.trips_2022`
  WHERE TIMESTAMP_DIFF(ended_at,started_at,second) < 0
```

```sql
  SELECT rideable_type, COUNT(ride_id) AS number_of_trips
  FROM `trips_2022.trips_2022`
  WHERE TIMESTAMP_DIFF(ended_at,started_at,second) < 0
  GROUP BY rideable_type
```

|Row|rideable_type|number_of_trips|
|:-:|:-:|:-:|
|1|classic_bike|28|
|2|electric_bike|72|


```sql
  SELECT member_casual, COUNT(ride_id) AS number_of_trips
  FROM `trips_2022.trips_2022`
  WHERE TIMESTAMP_DIFF(ended_at,started_at,second) < 0
  GROUP BY member_casual
```

|Row|member_casual|number_of_trips|
|:-:|:-:|:-:|
|1|casual|55|
|2|member|45|

```sql
  SELECT FORMAT_DATE('%A', started_at) AS day_of_week, COUNT(ride_id) AS number_of_trips
  FROM `trips_2022.trips_2022`
  WHERE TIMESTAMP_DIFF(ended_at,started_at,second) < 0
  GROUP BY day_of_week
  ORDER BY number_of_trips DESC
```

|Row|day_of_week|number_of_trips|
|:-:|:-:|:-:|
|1|Sunday|40|
|2|Tuesday|23|
|3|Saturday|12|
|4|Thursday|9|
|5|Monday|8|
|6|Friday|6|
|7|Wednesday|2|

<li>In addition to above checks, records with ride time of 0 were reviewed. Based on query below, it was identified that there are 431 such records. Checks similar to the previous point showed no correlation of 0 ride times with particular rideable types, days of the week or user types (member vs. casual).</li>

```sql
  SELECT ride_id, rideable_type, started_at,ended_at, member_casual,
  (ended_at - started_at) AS ride_length, FORMAT_DATE('%A', started_at) AS day_of_week
  FROM `trips_2022.trips_2022`
  WHERE TIMESTAMP_DIFF(ended_at,started_at,second) = 0
```

```sql
  SELECT rideable_type, COUNT(ride_id) AS number_of_trips
  FROM `trips_2022.trips_2022`
  WHERE TIMESTAMP_DIFF(ended_at,started_at,second) = 0
  GROUP BY rideable_type
```

|Row|rideable_type|number_of_trips|
|:-:|:-:|:-:|
|1|classic_bike|98|
|2|docked_bike|6|
|3|electric_bike|327|


```sql
  SELECT member_casual, COUNT(ride_id) AS number_of_trips
  FROM `trips_2022.trips_2022`
  WHERE TIMESTAMP_DIFF(ended_at,started_at,second) = 0
  GROUP BY member_casual
```

|Row|member_casual|number_of_trips|
|:-:|:-:|:-:|
|1|casual|208|
|2|member|223|

```sql
  SELECT FORMAT_DATE('%A', started_at) AS day_of_week, COUNT(ride_id) AS number_of_trips
  FROM `trips_2022.trips_2022`
  WHERE TIMESTAMP_DIFF(ended_at,started_at,second) = 0
  GROUP BY day_of_week
  ORDER BY number_of_trips DESC
```


|Row|day_of_week|number_of_trips|
|:-:|:-:|:-:|
|1|Saturday|83|
|2|Thursday|70|
|3|Friday|63|
|4|Tuesday|59|
|5|Sunday|58|
|6|Monday|52|
|7|Wednesday|46|

<h1>Data transformation</h1>
Based on data initial data validation and data cleaning procedures described in previous steps a final dataset for analysis was prepared. The following data transformations were applied:
<li>Calculated column with ride length was added,</li>
<li>Calculated column with day of week for trip start was added,</li>
<li>Columns with start stations, end stations, and geodata (longitute and latitude) were not included in the final table for analysis due to data integrity issues.</li>
<li>Records with ride times below or equal to zero were excluded from the final table for analysis.</li>
<br>
All of the transformations were reflected by applying the code below. Final cleaned & transformed dataset was saved as a Big Query table and named "cyclistic_2022_dataset_clean".

```sql
  SELECT ride_id, rideable_type, started_at,ended_at, member_casual,
  (ended_at - started_at) AS ride_length, FORMAT_DATE('%A', started_at) AS day_of_week
  FROM `trips_2022.trips_2022`
  WHERE TIMESTAMP_DIFF(ended_at,started_at,second) > 0
```

<h1>Conclusion</h1>
Detailed data cleaning procedures revealed additional cases of potential data quality issues as 531 records were identified as having negative or 0 ride times. Similarly to gaps in station and geolocation data identified during the initial data validation, in a real-life scenario this would require follow-up with business owners and data engineering team. Due to the small scale of the issues with incorrect ride times (531 out of 5,667,717 records, 0.01%), it was decided to eliminate these records for the purpose of this analysis, so that they do not skew further calculations.
