# Introduction

This case study is a part of Google Business Intelligence Certificate from Coursera.

As a part of this case study i have done Project Planning, Data preparation, Dashboard designing.
The dataset we will be using is entirely from the Public Dataset Google BigQuery. Here are the step-by-step instructions for analyzing a business, starting from the initial meeting to the deliverable of a dashboard that can support and answer all business needs.

# Scenario

Cyclistic, a fictional bike-share company in New York City. 
Cyclistic has partnered with the city of New York to provide shared bikes. Currently, there are bike stations located throughout Manhattan and neighboring boroughs. Customers are able to rent bikes for easy travel between stations at these locations.
Cyclistic’s Customer Growth Team is creating a business plan for next year. The team wants to understand how their customers are using their bikes; their top priority is identifying customer demand at different station locations.


# 1. Project Planning

Business intelligence strategy, which is the management of the people, processes, and tools used in the business intelligence process. BI projects are complicated, and finding ways to stay organized from the beginning of a project to the end is key to success. One way to ensure that you capture the big-picture project requirements, stay organized, and make an impact at your organization is to create comprehensive BI documents. 
Here we will create three types of documents: the Stakeholder Requirements Document, Project Requirements Document, and Strategy Document.

## Stakeholder Requirement Document

The Stakeholder Requirements Document enables us to capture stakeholder requests and requirements so that we understand their needs before planning the rest of the project details or strategy. To have a look at the stakeholder requirement documents[click here](https://docs.google.com/document/d/1M9t-kHVotXV36SaMQU91yiTHSRJIcA7A/edit?usp=sharing&ouid=101697456061317378080&rtpof=true&sd=true)

## Project Requirements Document

Once we've identified the requirements of the stakeholders, the next step involves defining the project requirements necessary to fulfill these stakeholder needs. By pinpointing these project requirements early in the process, we can ensure the success of our project and its alignment with the expectations of our stakeholders. This proactive approach not only aids in risk mitigation but also fosters clarity, effective communication, and adaptability throughout the project lifecycle, ultimately contributing to the overall success of the project.
To have a look at the project requirement documents[click here](https://docs.google.com/document/d/1xNRxtOW62ca0kMCS23J159GQ5jVuS26y/edit?usp=sharing&ouid=101697456061317378080&rtpof=true&sd=true)

## Strategy Document

At last we will create the strategy document. The Strategy Document is a collaborative place to align with stakeholders about project deliverables. These documents will help establish information about dashboard functionality and associated metrics and charts. Here we will explore what metrics will be required, how metrics are calculated, and any limitations or assumptions that exist about the data. To have a look at the strategy documents[click here]()

# 2. Data Preparation

In this step, we prepare the data using pipelines and ETL Process.
we use SQL and Google Dataflow to combine and move the key datasets, identified for the Cyclistic project into a target table. This represents the extraction phase of an ETL pipeline, when data is pulled from different sources and moved to its destination. We will use the table created in this activity to develop the final dashboard for stakeholders.

#### Data Sources
Primary dataset: 
[NYC Citi Bike Trips](https://console.cloud.google.com/marketplace/details/city-of-new-york/nyc-citi-bike?pli=1)

Secondary dataset: 
[Census Bureau US Boundaries](https://console.cloud.google.com/marketplace/product/united-states-census-bureau/us-geographic-boundaries?project=nth-clone-407307)

ZipCode Spreadsheet:
[Zipcode](https://docs.google.com/spreadsheets/d/1IIbH-GM3tdmM5tl56PHhqI7xxCzqaBCU0ylItxk_sy0/template/preview#gid=806359255)

## Extracting the data


First,we navigate to our BigQuery console. 
Search and preview the public datasets using the search bar in the Explore pane of console.
These datasets are already available to query, we find the below 3 datasets

- new_york_citibike

- geo_us_boundaries

- noaa_gsod

After that , upload the zip code dataset. 

## Loading and Transforming the Data


To load the data into the target table, Run the below code and save the table as CSV file in your drive. 


```SQL
SELECT
TRI.usertype,
ZIPSTART.zip_code AS zip_code_start,
ZIPSTARTNAME.borough borough_start,
ZIPSTARTNAME.neighborhood AS neighborhood_start,
ZIPEND.zip_code AS zip_code_end,
ZIPENDNAME.borough borough_end,
ZIPENDNAME.neighborhood AS neighborhood_end,
DATE_ADD(DATE(TRI.starttime), INTERVAL 5 YEAR) AS start_day,
DATE_ADD(DATE(TRI.stoptime), INTERVAL 5 YEAR) AS stop_day,
  WEA.temp AS day_mean_temperature,
  WEA.wdsp AS day_mean_wind_speed, 
  WEA.prcp day_total_precipitation, 
  ROUND(CAST(TRI.tripduration / 60 AS INT64), -1) AS trip_minutes,
  COUNT(TRI.bikeid) AS trip_count
FROM
  `bigquery-public-data.new_york_citibike.citibike_trips` AS TRI
INNER JOIN
  `bigquery-public-data.geo_us_boundaries.zip_codes` ZIPSTART
  ON ST_WITHIN(
    ST_GEOGPOINT(TRI.start_station_longitude, TRI.start_station_latitude),
    ZIPSTART.zip_code_geom)
INNER JOIN
  `bigquery-public-data.geo_us_boundaries.zip_codes` ZIPEND
  ON ST_WITHIN(
    ST_GEOGPOINT(TRI.end_station_longitude, TRI.end_station_latitude),
    ZIPEND.zip_code_geom)
INNER JOIN
  `bigquery-public-data.noaa_gsod.gsod20*` AS WEA
  ON PARSE_DATE("%Y%m%d", CONCAT(WEA.year, WEA.mo, WEA.da)) = DATE(TRI.starttime)
INNER JOIN
  `mydataset.nyc_zipcode` AS ZIPSTARTNAME
  ON ZIPSTART.zip_code = CAST(ZIPSTARTNAME.zip AS STRING)
INNER JOIN
  `mydataset.nyc_zipcode` AS ZIPENDNAME
  ON ZIPEND.zip_code = CAST(ZIPENDNAME.zip AS STRING)
WHERE
  WEA.wban = '94728' 
  AND EXTRACT(YEAR FROM DATE(TRI.starttime)) BETWEEN 2014 AND 2015
GROUP BY
  1,
  2,
  3,
  4,
  5,
  6,
  7,
  8,
  9,
  10,
  11,
  12,
  13
```

To have a look at the Target Table[click here](https://drive.google.com/file/d/19-uA_g7B4FLOtmxeXcMdjiy6Zo4QQ3Pv/view?usp=drive_link)


I have also executed the below query that captured data from just the summer season.


```SQL
SELECT
TRI.usertype,
TRI.start_station_longitude,
TRI.start_station_latitude,
TRI.end_station_longitude,
TRI.end_station_latitude,
ZIPSTART.zip_code AS zip_code_start,
ZIPSTARTNAME.borough borough_start,
ZIPSTARTNAME.neighborhood AS neighborhood_start,
ZIPEND.zip_code AS zip_code_end,
ZIPENDNAME.borough borough_end,
ZIPENDNAME.neighborhood AS neighborhood_end,
DATE_ADD(DATE(TRI.starttime), INTERVAL 5 YEAR) AS start_day,
DATE_ADD(DATE(TRI.stoptime), INTERVAL 5 YEAR) AS stop_day,
WEA.temp AS day_mean_temperature, 
WEA.wdsp AS day_mean_wind_speed, 
WEA.prcp day_total_precipitation,
ROUND(CAST(TRI.tripduration / 60 AS INT64), -1) AS trip_minutes,
TRI.bikeid
FROM
`bigquery-public-data.new_york_citibike.citibike_trips` AS TRI
INNER JOIN
`bigquery-public-data.geo_us_boundaries.zip_codes` ZIPSTART
ON ST_WITHIN(
ST_GEOGPOINT(TRI.start_station_longitude, TRI.start_station_latitude),
ZIPSTART.zip_code_geom)
INNER JOIN
`bigquery-public-data.geo_us_boundaries.zip_codes` ZIPEND
ON ST_WITHIN(
ST_GEOGPOINT(TRI.end_station_longitude, TRI.end_station_latitude),
ZIPEND.zip_code_geom)
INNER JOIN
`bigquery-public-data.noaa_gsod.gsod20*` AS WEA
ON PARSE_DATE("%Y%m%d", CONCAT(WEA.year, WEA.mo, WEA.da)) = DATE(TRI.starttime)
INNER JOIN
`mydataset.nyc_zipcode` AS ZIPSTARTNAME
ON ZIPSTART.zip_code = CAST(ZIPSTARTNAME.zip AS STRING)
INNER JOIN
`mydataset.nyc_zipcode` AS ZIPENDNAME
ON ZIPEND.zip_code = CAST(ZIPENDNAME.zip AS STRING)
WHERE
WEA.wban = '94728' 
AND DATE(TRI.starttime) BETWEEN DATE('2015-07-01') AND DATE('2015-09-30')
```


To have a look a look at the Summer Target Table[click here](https://drive.google.com/file/d/1W1N8Xmx1PjUNe21G3MJp0jwRsiaDGBCv/view?usp=drive_link)


# 3. Dashboard

This is the final part of the BI project. 

After obtaining the necessary data resources and getting approval from stakeholders to use them, we create a mockup design first. This way, stakeholders can get an idea of the layout that will be used at the end of the project. This way we can help stakeholders understand how their dashboard is going to look and make changes as per their convience.
Here's the mockup I created.

Now we use Tableau Software to connect the target table and visualizations.


#### Load the data into Tableau

To load the data into tableau, I have connected the Google Drive to Tableau( As i have stored the data in Google Drive ).

After loading the data, it's good to have a look at your data and try to understand the data before creating visualizations.

#### Create Charts

Now we create charts to include in our dashboards. We create charts in the sheets.

##### Sheet-1,2 : Top Trips:Start,End

As the stakeholders want to identify the customer demand in different stations based on total Trip Time, we create 2 charts i.e., Top Trips:Start Station & Top Trips:End Station
and create a dashboard named Top Trips by Station by combining these two sheets.

In these 2 Sheets we use the Sum( Trip Minutes ), User Type, Neighbourhood Start, Borough Start, Zip Code Start, Neighbourhood End, Borough End, Zip Code End.

#### Sheet-3,4 : Trip Count

Stakeholders wanted a visualization showing the percent growth in the number of trips year over year. So for that we created a visualization named Trip Count. 

In this we created 2 charts, first one showing the Total Trips by months in an Area Chart. In this chart we used Usertype, Sum( Trip Count ), Month( Start Day ).
Second One is Trip Count by Start Neighbourhood. This Chart shows information about the Trip Count in each month of 2019,2020 from Start Station. 

Both the charts gives us insights on the number of trips take over year. 

#### Sheet- 5,6 : Seasonality












