# Project Readme

## A. Problem Context
Aviation is one of the most data-intensive industries, with thousands of flights operating daily across hundreds of airports. Airlines, airport authorities, and regulators need reliable insights into air traffic patterns, flight cancellations, and route distributions to optimize operations, improve customer experience, and reduce costs.

This project focuses on building a data warehouse solution to analyze air traffic data for three major New York area airports: ISP (Long Island MacArthur Airport), JFK (John F. Kennedy International Airport), and LGA (LaGuardia Airport). By analyzing traffic patterns, cancellation trends, and route information, stakeholders can make data-driven decisions to improve airline operations and passenger satisfaction.

## B. Requirements

### 1. Requirements Analysis
- Business Personas
  - List the key stakeholders and their roles.
  - Example:
    - Data Analyst: Responsible for data analysis and reporting.
    - IT Manager: Oversees technical implementation.
- Risks
  - Data quality issues (missing fields, inconsistent formats in the provided dataset).
  - The Aviationstack API is expensive, so the project relies on a professor-provided dataset which may have limited coverage.
  - Cancellation reason data is not included in the dataset; external research is required to infer causes.
  - Team coding experience ranges from beginner to intermediate, which may slow development.
- Costs
  - Estimate the costs associated with the project.
    - Software licenses: $0
    - Real costs: $0
    - The project will utilize open source tools like python, pandas, numpy, and Apache Airflow for ETL, orchestration, and data analysis. Tableau Desktop is free for students through the Tableau for Students program using a Baruch .edu email, so no visualization licensing fees are required.
    - Optional Cost: $0-$70/month
    - In a production-level scenario we could use paid versions such as Tableau Creator at $70 per month, or a managed orchestration service like Google Cloud Composer; these are not needed for this project.
    - Data source costs: $0
    - The flight data is sourced from the Aviationstack database but is provided as a static JSON file by the professor, so there are no API subscription costs for the team. If we were to pull directly from the Aviationstack API instead, the Basic plan would cost about $50 per month for 10,000 requests, but we do not need this.
    - Cloud infrastructure costs: $0
    - The project uses Google Cloud Storage (GCS) for raw and staged data, and Google BigQuery as the data warehouse. Both fit within the GCP free tier for our expected data volume (roughly tens of GB of storage and well under 1 TB of queries per month), so no cloud costs are incurred.
    - Hardware upgrades: $0-$250+
    - If the project is running on a personal computer that meets the 8GB ram and 100gb storage than the costs are free assuming we already have these computers, but hypothetically if we didnt have these we would be looking at a fixed cost of $250 or more if we were to buy a computer for this project.
- Timeline
  - Phase 1: Requirements Gathering, Cost/Benefits/Risks, Architecture — February 28, 2026
  - Phase 2: Data Modeling — March 14, 2026
  - Phase 3: Data Pipeline Started — March 28, 2026
  - Phase 4: Data Pipeline Midway — April 11, 2026
  - Phase 5: Visualization Started — April 25, 2026
- Benefits
  - Identify peak traffic periods (by hour, day, week, month, quarter) for better resource planning.
  - Pinpoint airlines with the highest cancellation rates and suggest actionable improvements.
  - Visualize the most popular flight destinations via heat maps for route optimization.
  - Distinguish domestic vs. international flight volumes to understand airport utilization.

### 2. Business Requirements
- Analyze air traffic patterns of ISP, JFK, and LGA.
- Identify the most frequent flight destinations from each airport.
- Determine whether traffic follows a uniform or normal distribution pattern.
- Analyze domestic flight vs. international flight volumes.

### 3. Functional Requirements
- Track cancelled flights by airline (cancelled vs. non-cancelled).
- Identify the airline with the most cancellations.
- Provide recommendations to reduce flight cancellations.
- Generate traffic pattern reports filterable by hour, day, week, month, and quarter.
- Display a heat map showing the concentration of flight destinations.
- Classify and count domestic vs. international flights.

### 4. Data Requirements
- Structured flight data provided by the professor (sourced from the Aviationstack database).
- Reference data for airports (airport codes, names, locations with latitude/longitude for heat maps).
- External research data on flight cancellation causes (to supplement the dataset, which does not include cancellation reasons).

## C. Architecture

### 1. Information Architecture
- ![Information Architecture Diagram](Architectures/INFOARCH.drawio.png)

### 2. Data Architecture
- Describe the structure and flow of the data.
- Include diagrams or images if necessary. 
  - ![Data Architecture Diagram](Architectures/DATAARCH.drawio.png)

### 3. Technical Architecture
- ![Technical Architecture Diagram](Architectures/TECHARCH-Page-3.jpg)

### 4. Product Architecture  
- Provide an overview of the product's overall structure.
- Include any major components and how they interact.
- ![Product Architecture Diagram](Architectures/PRODUCT_ARCH.drawio.png)
## D. Modeling

### 1. Dimensional Modeling

In this project  we used a star schema dimensional model to organize the flight data for analytics. The central fact table is `fact_flight`, which stores the measurable flight activity records. The fact table connects to several dimension tables that describe the date, airports, airline, aircraft, route, and flight status.

  - ![Dimensional Modeling Diagram](Dimensional_modeling/FLIGHTS_DB_PNG.png)
    
### Fact Table

#### `fact_flight`

The `fact_flight` table is the center of the model and represents individual flight records. Each row contains flight-level information such as the flight date, airline, departure airport, arrival airport, aircraft, route, and flight status. It also includes operational fields such as scheduled, estimated, and actual departure and arrival times, delay minutes, cancellation flags, delay flags, landed flags, and live flight tracking fields.

The fact table contains foreign keys that connect to the dimension tables:

- `date_key`  to `dim_date`
- `departure_airport_key`  to `dim_airport`
- `arrival_airport_key`  to `dim_airport`
- `airline_key`  to `dim_airline`
- `status_key`  to `dim_flight_status`
- `aircraft_key` conects to `dim_aircraft`
- `route_key`  to `dim_route`

### Dimension Tables

#### `dim_date`

The `dim_date` table allows for time related analysis. It includes full date, day of month, day of week, day name, week of year, month number, month name, quarter number, year number, and weekend indicator. This allows users to analyze flight patterns by day, month, quarter, year, and weekend vs. weekday.

#### `dim_airport`

The `dim_airport` table stores airport reference information. It includes airport ID, IATA code, ICAO code, airport name, city, country, continent, timezone, GMT offset, latitude, longitude, and phone number. This dimension is used twice in the fact table, once for the departure airport and once for the arrival airport.

#### `dim_airline`

The `dim_airline` table stores airline information such as airline ID, IATA code, ICAO code, airline name, callsign, hub code, country, founding date, fleet size, average fleet age, IATA prefix accounting, airline status, and airline type. This supports airline level analysis such as flight volume and cancellation performance by airline.

#### `dim_flight_status`

The `dim_flight_status` table stores the flight status category and related status flags. It includes the status key, flight status, cancellation flag, delayed flag, landed flag, active flag, scheduled flag, and diverted flag. This allows the dashboard to compare cancelled, landed, active, scheduled, delayed, and diverted flights.

#### `dim_aircraft`

The `dim_aircraft` table stores aircraft and airplane reference data. It includes aircraft key, IATA type code, aircraft registration, aircraft type reference ID, plane type ID, aircraft name, IATA codes, model code, model name, plane series, plane class, production line, engines count, engines type, and plane status. This allows flight activity to be analyzed by aircraft type and model.

#### `dim_route`

The `dim_route` table stores origin-destination route information. It includes route key, origin IATA code, origin ICAO code, destination IATA code, destination ICAO code, route name, domestic flag, international flag, origin country name, destination country name, origin timezone, and destination timezone. This dimension supports route analysis, top destination analysis, and domestic vs. international flight comparisons.

### Relationships

The dimensional model follows a one-to-many relationship structure. Each dimension table has one record per descriptive entity, while the `fact_flight` table can contain many flight records connected to those dimensions.

Examples:

- One date can relate to many flights.
- One airline can operate many flights.
- One airport can appear in many departure and arrival records.
- One aircraft can be connected to many flights.
- One route can appear in many flight records.
- One flight status can describe many flights.

This model allows Power BI users to filter and analyze flight data across time, airports, airlines, aircraft, routes, and statuses.


## 2. Medallion Architecture

The project uses a Medallion Architecture to organize the flight data pipeline into three layers: Bronze, Silver, and Gold. This structure separates raw data storage, cleaned/transformed data, and business-ready analytics outputs.

### Bronze Layer: Raw Data

The Bronze layer has the original source data exactly as it was taken. This includes the raw flight dataset and reference files such as airports, airlines, aircraft, cities, and countries. These files are stored in cloud storage as the stage before any cleaning or transformation is done

[RAW DATA](Raw_data)

### Silver Layer: Cleaned and Transformed Data

The Silver layer has cleaned, standardized, and structured data. In this stage, the raw data is transformed into a dimensional model. Cleaning steps include standardizing column names, converting data types, handling missing values, creating surrogate keys, and separating the data into fact and dimension tables.

[CLEANED DATA](Project_Data)

### Gold Layer: Business-Ready Analytics

The Gold layer represents the final reporting and analytics layer. This includes aggregated metrics, dashboard-ready outputs, and Power BI visualizations used to answer the project’s business questions.

[ANALYTICS AND VISUALS](DashBoard)

  - ![Medallion Architecture Diagram](Architectures/Medallion_ARCH.drawio.png)

## E. Methodology and Implementation

### Approach

The project follows a phased delivery approach structured around the milestones set by the professor. Work was organized as a sequence of fixed-scope phases rather than time-boxed Agile sprints, with each phase producing concrete deliverables that fed directly into the next phase.

### Phases

- Phase 1 (target Feb 28, 2026): requirements gathering, cost, benefits, risks, and architecture. The team delivered the problem context, business, functional, and data requirements, and the Information, Data, Technical, and Product architecture diagrams.
- Phase 2 (target Mar 14, 2026): data modeling. The dimensional model with Fact_Flight and six dimensions was authored in DbSchema as `FLIGHT_DBS_FINAL.dbs` and exported as the ER diagram `FLIGHTS_DB_PNG.png`.
- Phase 3 (target Mar 28, 2026): data pipeline started. The initial ETL notebook handles JSON ingestion from `flightdata.zip`, reference CSV loading, and the raw-to-cleaned flight DataFrame.
- Phase 4 (target Apr 11, 2026): data pipeline midway. Dimension tables for Date, Airport, Airline, Flight_Status, Aircraft, and Route were constructed, and Fact_Flight was assembled with surrogate keys. Dimension outputs were committed under `Project_Data/dim_*.xls`.
- Phase 5 (target Apr 25, 2026): visualization started, in progress. Tableau dashboards on traffic patterns, cancellations, route distribution, and domestic vs. international composition are planned. Workbooks have not yet been committed at the time of writing.

### Metadata Management

The Data Dictionary covers every column of the flight fact and all six dimension tables, with column name, data type, description, and constraints or nullability. It is maintained in [`Data_Dictionary/Data_Dictionary_Flight.pdf`](Data_Dictionary/Data_Dictionary_Flight.pdf).

Source data is provided externally by the professor as the `flightdata.zip` archive and six reference CSV files. These files are not committed to the repository; the ETL notebook expects them in the working directory at run time.

Per-day Aviationstack JSON files inside `flightdata.zip` are flattened into Fact_Flight, Dim_Flight_Status, and Dim_Date. The flattening unpacks the nested departure, arrival, airline, flight, aircraft, and live objects, and derives delay minutes, cancellation, delayed, landed, active, scheduled, and diverted flags, plus the codeshare flag.

The six reference CSVs feed the dimensions as follows. `CIS4400_project08_references_airports.csv` populates Dim_Airport, normalizing IATA and ICAO codes and attaching city, country, latitude, longitude, timezone, and GMT offset. `CIS4400_project08_references_airlines.csv` populates Dim_Airline, normalizing IATA and ICAO codes and attaching country, callsign, hub, and fleet info. `CIS4400_project08_references_aircrafts_types.csv` provides aircraft type code, aircraft name, and plane type id used in Dim_Aircraft. `CIS4400_project08_references_airplanes.csv` provides registration, model code, model name, series, class, production line, and engines, also used in Dim_Aircraft. `CIS4400_project08_references_cities.csv` enriches Dim_Airport by mapping city IATA to city name and timezone. `CIS4400_project08_references_countries.csv` enriches both Dim_Airport and Dim_Airline by mapping ISO-2 to country name and continent.

Dim_Route is built from origin and destination IATA pairs derived from the flight JSON. Each unique pair receives a surrogate route key. A route is classified as domestic when the origin and destination country names match, joined through Dim_Airport which is itself enriched by the countries reference, and as international when they differ.

The ETL notebook defines the following helper functions.

- `clean_string(series)` strips whitespace and converts empty, `nan`, and `None` strings to NA.
- `safe_str(series)` applies `clean_string` and uppercases the result.
- `normalize_code(series)` standardizes IATA and ICAO codes to uppercase, NA-safe.
- `to_datetime_safe(series)` parses timestamps with errors coerced to NaT, UTC-aware.
- `bool_to_int(series)` converts boolean flags to 0 or 1, with NA filled as `False`.
- `make_surrogate_key(df, key_name)` generates sequential surrogate keys for a dimension.
- `read_json_files_from_zip(zip_path, max_files)` iterates JSON files inside a zip archive and unions them into a DataFrame.
- `get_nested_value(obj, key)` safely accesses a nested key from an Aviationstack JSON object.

### ETL vs. ELT

The pipeline follows an ETL pattern. In the extract step, raw flight JSON from `flightdata.zip` and the six reference CSVs are read into pandas DataFrames. In the transform step, nested JSON is flattened, codes are normalized, timestamps are parsed, status flags such as cancelled, delayed, codeshare, and live-tracked are derived, routes are classified as domestic or international, and surrogate keys are assigned, all in memory. In the load step, the curated Fact_Flight and the six dimension tables are written as CSV files into a local `warehouse_output/` directory. The six dimension outputs are also exported to Excel and committed under `Project_Data/dim_*.xls` for inspection. The final relational target is BigQuery, per the Technical Architecture.

ELT was not adopted because the source data requires substantial structural transformation, including nested JSON flattening, multi-source enrichment, and surrogate-key assignment, which is more naturally expressed in pandas than in SQL.

### Tools

- Python 3 and Jupyter Notebook are used to author and run the ETL notebook (`Cleaned _Flight_data (2).ipynb`).
- pandas and numpy handle DataFrame transforms, cleaning, type coercion, and joins.
- Standard library modules `json`, `zipfile`, and `os` are used to read JSON archives and manage paths.
- Microsoft Excel is used to convert CSV outputs to `.xls` for the committed dimension tables.
- DbSchema is used to author the Fact and Dimension model in `.dbs` format and export the ER diagram.
- Google BigQuery and Google Cloud Storage are the planned warehouse and staging targets per the Technical Architecture.
- Tableau Desktop is used to connect to the warehouse and build dashboards.
- draw.io is used for the Information, Data, Technical, and Product architecture diagrams.
- Git and GitHub are used for version control, code and documentation tracking, and pull-request review.

The pipeline is currently executed manually by running the Jupyter notebook end-to-end. A managed orchestration layer such as Apache Airflow or Google Cloud Composer is referenced in the Technical Architecture as the production-scale equivalent, but is out of scope for the present implementation.

## F. Visualization
## F. Visualization

The final interface for the project was created using Power BI. The dashboard lest users analyze flight activity for ISP, JFK, and LGA using different filters for airport, airline, flight status, date range, and hour range.

The dashboard was divided into four main report pages which are:

### 1. Dashboard Storyboard

The storyboard page shows the user flow of the dashboard. beginning with filters for origin airport, airline, flight status, and date. The visuals then update to show flight volume by month, flight status breakdown, top routes, cancelled flights, domestic vs. international flights, and flight-level detail records.

![Dashboard Storyboard](DashBoard/STORY_BOARD.png)

### 2. Air Traffic Overview

The overview page gives a summary of the flight dataset. It includes KPI cards for total flights, delayed flights, cancelled flights, landed flights, and average delay. It also includes charts for flight status breakdown, total flights by year/month, total flights by airport, and monthly flight trends by airport.

![Air Traffic Overview](DashBoard/OVERVIEW.png)

### 3. Airline Cancellation and Route Performance

This page focuses on airline and route analysis. It compares flight volume by airline, cancellations by airline, popular destinations, and origin-destination route performance. This page supports the project requirement to track cancelled flights by airline and identify high-volume routes.

![Airline Cancellation and Route Performance](DashBoard/AIRLINE_CANCELATIONS.png)

### 4. Historical Flight Performance

This page analyzes time-based flight patterns. It shows flight volume by month, day of week, and hour. These visuals help show peak periods and support resource planning for airports and airlines.

![Historical Flight Performance](DashBoard/HISTORICAL_PERFORMANCE.png)

## G. Insights
Highlight any key insights gained from the project.

The Power BI dashboard revealed several important patterns in the flight data:

1. JFK has the highest flight volume among the three airports.
The overview dashboard shows JFK with the largest total flight count, followed by LGA, while ISP has significantly lower activity. This indicates that JFK is the primary airport in the dataset for overall traffic volume.

2. Most flights are marked as landed.
The flight status breakdown shows that the majority of flights were completed successfully. Landed flights make up the largest share of the dataset, while cancelled, diverted, and unknown flights represent much smaller portions.

3. Flight volume changes noticeably by month.
Monthly flight volume is strongest around April and May, with both months reaching about 109K flights. Volume drops sharply around September and October, with both months around 39K flights, before increasing again in December.

4. Flight activity is highest during afternoon and evening hours.
The hourly chart shows that flight volume rises throughout the day and peaks around the late afternoon and evening. The highest hourly volume appears around hour 17, with about 70K flights.

5. Sunday, Thursday, and Monday have the highest flight volume.
The day-of-week chart shows Sunday, Thursday, and Monday each around 145K flights. Saturday is the lowest day, with about 132K flights.

6. Delta Air Lines has the highest flight volume.
The airline performance page shows Delta Air Lines with about 140K flights, making it the highest-volume airline in the dataset.

7. Delta Air Lines also has the highest cancellation count.
The cancellations chart shows Delta Air Lines with about 2.7K cancellations. American Airlines and WestJet follow with about 1.4K cancellations each.

8. The most popular destination is BOS.
The popular destinations chart shows BOS as the top destination with about 44K flights, followed by LAX with about 41K flights and ATL with about 36K flights.

9. Most flights are domestic.
The domestic vs. international chart shows that around 79.75% of flights are domestic, while about 20.25% are international. This suggests the dataset is mainly focused on domestic U.S. air traffic.

## H. Conclusion

-This project successfully built a data warehouse and analytics pipeline for aviation data related to ISP, JFK, and LGA. The team transformed raw flight and reference data into a dimensional model with fact and dimension tables, then used Power BI to create interactive dashboards for flight traffic analysis.

-The final dashboard allows users to evaluate total flight volume, flight status, cancellations by airline, popular destinations, route activity, and historical traffic patterns. These insights can help airport analysts, airline managers, and operations teams better understand traffic trends, identify cancellation patterns, and support planning decisions.

-Future improvements could include connecting to the live Aviationstack API, adding weather data, including cancellation reason data, and creating predictive models for delays or cancellations.

## I. References
- Provide a list of all references used in the project, formatted according to MLA style.

1. Author Last Name, First Name. *Title of Book*. Publisher, Year.
2. "Title of Article." *Name of Journal*, vol. 1, no. 1, Year, pp. 1-10.
3. *Title of Website*. Website Publisher, Year, URL.

---

*Replace placeholders like "path_to_image" with actual file paths or URLs.*
