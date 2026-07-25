# Build-a-Data-Warehouse-with-BigQuery-Challenge-Lab


## Task 1

```
CREATE SCHEMA IF NOT EXISTS `covid`;

CREATE OR REPLACE TABLE `covid.oxford_policy_tracker`
PARTITION BY date
OPTIONS (
  partition_expiration_days = 2175
)
AS
SELECT *
FROM `bigquery-public-data.covid19_govt_response.oxford_policy_tracker`
WHERE alpha_3_code NOT IN ('GBR', 'BRA', 'CAN', 'USA');
```

## Task 2

### 1:
```
CREATE OR REPLACE TABLE `covid_data.country_area_data`
AS
SELECT *
FROM `bigquery-public-data.census_bureau_international.country_names_area`;
```
### 2:
```
UPDATE `covid_data.consolidate_covid_tracker_data` t
SET mobility = STRUCT(
  m.avg_retail AS avg_retail,
  m.avg_grocery AS avg_grocery,
  m.avg_parks AS avg_parks,
  m.avg_transit AS avg_transit,
  m.avg_workplace AS avg_workplace,
  m.avg_residential AS avg_residential
)
FROM (
  SELECT
    country_region,
    date,
    AVG(retail_and_recreation_percent_change_from_baseline) AS avg_retail,
    AVG(grocery_and_pharmacy_percent_change_from_baseline) AS avg_grocery,
    AVG(parks_percent_change_from_baseline) AS avg_parks,
    AVG(transit_stations_percent_change_from_baseline) AS avg_transit,
    AVG(workplaces_percent_change_from_baseline) AS avg_workplace,
    AVG(residential_percent_change_from_baseline) AS avg_residential
  FROM `bigquery-public-data.covid19_google_mobility.mobility_report`
  GROUP BY country_region, date
) m
WHERE t.country_name = m.country_region
  AND t.date = m.date;
```

### 3:
```
ALTER TABLE `covid_data.global_mobility_tracker_data`
ADD COLUMN population INT64,
ADD COLUMN country_area FLOAT64,
ADD COLUMN mobility STRUCT<
  avg_retail FLOAT64,
  avg_grocery FLOAT64,
  avg_parks FLOAT64,
  avg_transit FLOAT64,
  avg_workplace FLOAT64,
  avg_residential FLOAT64
>;
```

## Task 3

### 1:
```
CREATE OR REPLACE TABLE `covid_data.mobility_data`
AS
SELECT *
FROM `bigquery-public-data.covid19_google_mobility.mobility_report`;
```

### 2:
```
SELECT
  country_name,
  'population' AS missing_field
FROM `covid_data.oxford_policy_tracker_worldwide`
WHERE population IS NULL

UNION ALL

SELECT
  country_name,
  'country_area' AS missing_field
FROM `covid_data.oxford_policy_tracker_worldwide`
WHERE country_area IS NULL

ORDER BY country_name;
```

### 3:
```
UPDATE `covid_data.consolidate_covid_tracker_data` t1
SET
  population = t2.pop_data_2019
FROM (
  SELECT DISTINCT
    country_territory_code,
    pop_data_2019
  FROM `bigquery-public-data.covid19_ecdc.covid_19_geographic_distribution_worldwide`
) AS t2
WHERE t1.alpha_3_code = t2.country_territory_code;
```

## Task 4

### 1:
```
DELETE FROM `covid_data.oxford_policy_tracker_by_countries`
WHERE population IS NULL
   OR country_area IS NULL;
```
### 2:
```
CREATE OR REPLACE TABLE `covid_data.pop_data_2019`
AS
SELECT *
FROM `bigquery-public-data.covid19_ecdc.covid_19_geographic_distribution_worldwide`;
```

### 3:
```
UPDATE `covid_data.consolidate_covid_tracker_data` t
SET country_area = s.country_area
FROM `bigquery-public-data.census_bureau_international.country_names_area` s
WHERE t.country_name = s.country_name;
```

