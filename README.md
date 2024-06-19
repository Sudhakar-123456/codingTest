Overview
--------

This coding exercise will help us understand how you approach some of the common problems we see in data engineering. Please approach it as you would a work assignment: ask questions if things are unclear, use best practices and common software patterns, and feel free to go the extra mile to show off your skills.

You will be asked to ingest some weather and crop yield data (provided), design a database schema for it, and expose the data through a REST API. You may use whatever software tools you would like to answer the problems below, but keep in mind the skills required for the position you are applying for and how best to demonstrate them. Read through all the problems before beginning, as later problems may inform your approach to earlier problems.

You can retrieve the data required for this exercise by cloning this repository:
https://github.com/corteva/code-challenge-template

Weather Data Description
------------------------

The wx_data directory has files containing weather data records from 1985-01-01 to 2014-12-31. Each file corresponds to a particular weather station from Nebraska, Iowa, Illinois, Indiana, or Ohio.

Each line in the file contains 4 records separated by tabs: 

1. The date (YYYYMMDD format)
2. The maximum temperature for that day (in tenths of a degree Celsius)
3. The minimum temperature for that day (in tenths of a degree Celsius)
4. The amount of precipitation for that day (in tenths of a millimeter)

Missing values are indicated by the value -9999.

Yield Data Description
----------------------

The yld_data directory has a single file, US_corn_grain_yield.txt, containing a table of the total harvested corn grain yield in the United States measured in 1000s of megatons for the years 1985-2014.

---

Problem 1 - Data Modeling
-------------------------
Choose a database to use for this coding exercise (SQLite, Postgres, etc.). Design two data models: one to represent the weather data records, and one to represent the yield data records. If you use an ORM, your answer should be in the form of that ORM's data definition format. If you use pure SQL, your answer should be in the form of DDL statements.

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Solution Explanation:
For designing and implementing this case study, SQLite has been choosen as prefered DB. Reason behind choosing SQLite is its simplicity and being a file based DB, it does not require additional installation. 
Additionally since we have to create an API service also for this case study, ORM approch has been choosen to implement the Data models. ORM help us to easily interact with our Data Sources from code. flask_sqlalchemy framework has been chosen for implementing the ORM.
Two data models has been desinged representing each dataset as follows:
1. WeatherData with below fields for resenting Weather Data:
	tablename = 'weather_data'
	field details:
		1.1 station_id, String, primary_key
		1.2 date_, DateTime, primary_key
		1.3 max_temp, Integer, nullable
		1.4 min_temp, Integer, nullable
		1.5 amt_ppt, Integer, nullable

2. YieldData for resenting Yield Data
	tablename = 'yield_data'
	field details:
		2.1 year_, Integer, primary_key
		2.2 yield_, Integer

For initializing/ creating the DB, built_db.py module has been written which can be used to create the DB.
And models.py module contains the models created for this case study.
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Problem 2 - Ingestion
---------------------
Write code to ingest the weather and yield data from the raw text files supplied into your database, using the models you designed. Check for duplicates: if your code is run twice, you should not end up with multiple rows with the same data in your database. Your code should also produce log output indicating start and end times and number of records ingested.

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Solution Explanation:
For ingeting the records into respective tables in DB, module ingest_in_db.py  has been written.

To ingest only the data which is not present in DB, first we findout list of existing records in DB and compare it with all data present in the data directory which is required to ingest. Only Records which are not part of existing DB records are appended to records_to_ingest list and then ingested in DB in batch size of 100000.
Similar process is used to ingest both Weather Data & Yield Data
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Problem 3 - Data Analysis
-------------------------
For every year, for every weather station, calculate:

* Average maximum temperature (in degrees Celsius)
* Average minimum temperature (in degrees Celsius)
* Total accumulated precipitation (in centimeters)

Ignore missing data when calculating these statistics.

Design a new data model to store the results. Use NULL for statistics that cannot be calculated.

Your answer should include the new model definition as well as the code used to calculate the new values and store them in the database.

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Solution Explanation:
A third model has been desinged representing Weather Stats dataset (in models.py)as follows:
1. WeatherStats with below fields for resenting Weather Stats Data:
	tablename = 'weather_stats'
	field details:
		1.1 station_id, String, primary_key
		1.2 year_, Integer, primary_key
	    1.3 avg_max_temp, Float, nullable
	    1.4 avg_min_temp, Float, nullable
	    1.5 total_amt_ppt, Integer, nullable

for ingesting the stats, function ingest_weather_stats_data() in module ingest_in_db.py has been written. Since for the case study we are not interesting in maintinaing the history of stats at this point, this function first delete the existing stats from the DB, calculate new stats by quering the weather_data & yield_data tables in DB and ingest them in batch size of 100000

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Problem 4 - REST API
--------------------
Choose a web framework (e.g. Flask, Django REST Framework). Create a REST API with the following GET endpoints:

/api/weather
/api/yield
/api/weather/stats

Each endpoint should return a JSON-formatted response with a representation of the ingested data in your database. Allow clients to filter the response by date and station ID (where present) using the query string. Data should be paginated.

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Solution Explanation:
For creating the REST APIs, flask REST framework has been used. Additionally to ensure if APIs works as intended, unit test cases has been written for each API usin pytest framework. below are some of reasons for choosing pytest framework for testing:
1. Pytest has rich inbuilt features which require less piece of code compared to unittest.
2. Pytest provides an effective way to write and execute scalable test cases or test suites and generate extensive test reports.
3. Easy code maintainability as compared to other frameworks
