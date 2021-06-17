# Create ML Models with BigQuery ML: Challenge Lab

## Your challenge
One of the projects you are working on needs to provide analysis based on real world data that will help in the selection of new bicycle models for public bike share systems. Your role in this project is to develop and evaluate machine learning models that can predict average trip durations for bike schemes using the public data from Austin's public bike share scheme to train and evaluate your models.

Two of the senior data scientists in your team have different theories on what factors are important in determining the duration of a bike share trip and you have been asked to prioritise these to start. The first data scientist maintains that the key factors are the start station, the location of the start station, the day of the week and the hour the trip started. While the second data scientist argues that this is an over complication and the key factors are simply start station, subscriber type, and the hour the trip started.

You have been asked to develop a machine learning model based on each of these input features. Given the fact that stay-at-home orders were in place for Austin during parts of 2020 as a result of COVID-19 you will be working on data from previous years. You have been instructed to train your models on data from 2018 and then evaluate them against data from 2019 on the basis of Mean Absolute Error and the square root of Mean Squared Error.

You can access the public data for the Austin bike share scheme in your project by opening [this link to the Austin bike share dataset](https://console.cloud.google.com/projectselector2/bigquery?p=bigquery-public-data&d=austin_bikeshare&page=dataset&supportedpurview=project) in the browser tab for your lab.

As a final step you must create and run a query that uses the model that includes subscriber type as a feature, to predict the average trip duration for all trips from the busiest bike sharing station in 2019 (based on the number of trips per station in 2019) where the subscriber type is 'Single Trip'.


## Task 1: Create a dataset to store your machine learning models
Create a new dataset in which you can store your machine learning models.

### Answer for Task 1

```
bq mk bikeshare
```

## Task 2: Create a forecasting BigQuery machine learning model
Create the first machine learning model to predict the trip duration for bike trips. The features of this model must incorporate the starting station name, the hour the trip started, the weekday of the trip, and the address of the start station labeled as location. You must use 2018 data only to train this model.

### Answer for Task 2

```sql
CREATE OR REPLACE MODEL `bikeshare.model_1`
OPTIONS(model_type='linear_reg', labels=['duration_minutes']) AS
	SELECT
		start_station_name,
		EXTRACT(DAYOFWEEK FROM start_time) AS day_of_week,
		EXTRACT(HOUR FROM start_time) AS start_hour,
		address AS location,
		duration_minutes
	FROM
		`bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips
	JOIN
		`bigquery-public-data.austin_bikeshare.bikeshare_stations` AS stations
	ON trips.start_station_name = stations.name
	WHERE EXTRACT(YEAR FROM start_time) = 2018 AND duration_minutes > 0
```

## Task 3: Create the second machine learning model
Create the second machine learning model to predict the trip duration for bike trips. The features of this model must incorporate the starting station name, the bike share subscriber type and the start time for the trip. You must also use 2018 data only to train this model.

### Answer for Task 3

```sql
CREATE OR REPLACE MODEL `bikeshare.model_2`
OPTIONS(model_type='linear_reg', labels=['duration_minutes']) AS
	SELECT
	    start_station_name,
	    EXTRACT(HOUR FROM start_time) AS start_hour,
	    subscriber_type,
	    duration_minutes
	FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips
	WHERE EXTRACT(YEAR FROM start_time) = 2018
```


## Task 4: Evaluate the two machine learning models
Evaluate each of the machine learning models against 2019 data only using separate queries. Your queries must report both the Mean Absolute Error and the Root Mean Square Error.

### Answer for Task 4

```sql
SELECT SQRT(mean_squared_error) AS rmse, mean_absolute_error 
FROM ML.EVALUATE(MODEL `bikeshare.model_1`, (
	SELECT
		start_station_name,
		EXTRACT(DAYOFWEEK FROM start_time) AS day_of_week,
		EXTRACT(HOUR FROM start_time) AS start_hour,
		address AS location,
		duration_minutes
	FROM
		`bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips
	JOIN
		`bigquery-public-data.austin_bikeshare.bikeshare_stations` AS stations
	ON trips.start_station_name = stations.name
	WHERE EXTRACT(YEAR FROM start_time) = 2019)
)
```

```sql
SELECT SQRT(mean_squared_error) AS rmse, mean_absolute_error 
FROM ML.EVALUATE(MODEL `bikeshare.model_2`, (
	SELECT
		start_station_name,
		subscriber_type,
		EXTRACT(HOUR FROM start_time) AS start_hour,
		duration_minutes
	FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips
	WHERE EXTRACT(YEAR FROM start_time) = 2019)
)
```
	
## Task 5: Task 5: Use the subscriber type machine learning model to predict average trip durations
When both models have been created and evaulated, use the second model, that uses subscriber_type as a feature, to predict average trip length for trips from the busiest bike sharing station in 2019 where the subscriber type is ```Single Trip```.

### Answer for Task 5

```sql
SELECT AVG(predicted_duration_minutes) AS average_predicted_trip_duration
FROM ML.PREDICT(MODEL `bikeshare.model_2`, (
    WITH busiest_station AS (
        SELECT start_station_name AS station, COUNT(*) AS trips_count
        FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips
        WHERE EXTRACT(YEAR FROM start_time) = 2019
        GROUP BY start_station_name
        ORDER BY trips_count DESC
        LIMIT 1
    )
    SELECT      
        start_station_name,
        subscriber_type,
        EXTRACT(HOUR FROM start_time) AS start_hour,
        duration_minutes
    FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips
    WHERE EXTRACT(YEAR FROM start_time) = 2019
            AND subscriber_type = 'Single Trip'
            AND (start_station_name = (SELECT station FROM busiest_station))
    )
)
```


## Tips and Tricks
* Tip 1. You will need to combine the information from both Austin bike share tables in the public dataset to create your first model by means of a JOIN statement.

* Tip 2. You must train both models using 2018 data only and evaluate the models using 2019 data only.

* Tip 3. You must choose a model type that is suitable for forecasting label values. When evaluating the models you should use SELECT SQRT(mean_squared_error) AS rmse, mean_absolute_error FROM ML.EVALUATE(...) to return the specific model performance metrics the data scientists want to use.

* Tip 4. Your prediction queries must return the average of the predicted value output by the model for the trip duration and not just the average of the actual trip duration.

