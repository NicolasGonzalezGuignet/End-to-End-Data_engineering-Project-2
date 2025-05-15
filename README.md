# End-to-End Data Engineering Project using Azure and Snowflake

## Summary
In this project, data is extracted from various API (Weather API from openweathermap.org and Meteostat API) to create an interactive dashboard with the weather forecast for the next 24 hours, view the current weather conditions, and analyze climate variables from the past 10 years. Data ingestion is performed using Azure Data Factory, and the data is stored in a data lake (ADLS Gen2), where data is initially stored in its native format (JSON). Then, the processing and storage of the data is handled through Snowflake. First, an external stage is created to visualize the data without storing it in Snowflake, using the $ notation. Afterwards, Snowpipes are created, which, through Event Grid notifications in Azure (blob created), enable incremental ingestion into a raw layer. In this layer, the data is stored in table format, where the entire JSON file and its metadata are saved. Next, using dynamic tables (which allow dynamic transformations and reading only the new records for incremental load), we unnest the arrays in the JSON files (using the flatten function), clean and filter the data, and obtain a table with structured data. Finally, a small data warehouse is created with 2 dimension tables and 3 fact tables. This layer or schema is then connected to Power BI to create interactive dashboards.

## Tools and Technologies
- **Cloud**: Azure
- **Processing**: Snowflake
- **Storage**: Azure Data Lake Storage Gen 2 and Snowflake
- **Business Intelligence Dashboard**: Power BI
- **Data Pipelines/Orchestrator**: Azure Data Factory and Snowflake (Snowpipes and Dynamic Tables)

## Project Architecture
Here you can visualize the completed project:
<img src="https://i.imgur.com/NBxyEbc.png" alt="architecture">

## Objectives
Extract data from an API, transform it, and load it into Power BI.

## Azure Provisioned Resources
- Azure Data Factory (ADF)
- Storage Account (ADLS Gen2)
- Snowflake Instance
- Azure Event Grid Notifications

## Process Description

### Ingest data from APIs
- The data is extracted in JSON format from two APIs (https://openweathermap.org/api and https://dev.meteostat.net/api) using Azure Data Factory (ADF) and stored in an Azure    Data Lake Storage Gen2 (ADLSg2).
  #### 1st Pipeline
    This process consists of 6 copy data activities, 3 of which are used to extract current weather data and the other 3 to extract air pollution data and save them into a   
    ADLSg2. This is done for each province in the Cuyo region (Mendoza, San Juan and San Luis) (ejemplo de respuestas de la API:[Response1](ADF/Response-APIs-json/weather.json)/[Response2](ADF/Response-APIs-json/air-pollution.json))
    -  <img src="https://i.imgur.com/UzHY0bg.png" alt="1st pipeline">
    
  #### 2nd Pipeline
    Mediante 3 actividades copydata se extrea
    This array is used in a ForEach activity containing a CopyData activity that moves tables to the raw layer in ADLS Gen2
    <img src="https://i.imgur.com/O9CEDAJ.png" alt="2nd pipeline">

### 3rd Pipeline (databricks_pipeline)
- Two Databricks notebooks are created:
  1. The first ([link to notebook](pipelines/databricks/notebooks/Databricks_notebook_transform.ipynb)) transforms data (cleaning, filtering, enriching, etc.) and stores the transformed data in Delta format in the silver layer
  2. The second ([link to notebook](pipelines/databricks/notebooks/Data_Modelling.ipynb)) implements dimensional modeling (fact and dimension tables) to address business needs, storing these tables in the gold layer
  
  Model View
  
  <img src="https://i.imgur.com/RahPcBQ.png" alt="view model">



  These notebooks are added as separate activities in ADF
  
  <img src="https://i.imgur.com/CL67s0o.png" alt="Databricks pipeline">

### 4th Pipeline (ETL_Full_Load)
- This pipeline groups all pipelines into a single pipeline using the Execute Pipeline activity
  <img src="https://i.imgur.com/uUBEYPB.png" alt="Master pipeline">

### 5. Power BI Connection [Dashboard](power-bi/dashboard.png)
- A connection is established between the Databricks "gold" database and Power BI for real-time data visualization
  <img src="https://i.imgur.com/OpWGgAq.png" alt="Power BI connection">

In this [File](arm_template.zip) you can see the ARM template to provision a similar workspace in ADF.


Here you can see the [video](https://drive.google.com/file/d/1g6jlUvcwRXHP9ZWVMlkXtklbWZ4iJ9AJ/view?usp=sharing) that documents the implementation of the ETL.



 

 

 


 

