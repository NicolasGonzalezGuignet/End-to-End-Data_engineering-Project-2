# End-to-End Data Engineering Project using Azure and Snowflake

## Summary
In this project, data is extracted from various API (Weather API from openweathermap.org and Meteostat API). Data ingestion is performed using Azure Data Factory, and the data is stored in a data lake (ADLS Gen2), where data is initially stored in its native format (JSON). Then, the processing and storage of the data is handled through Snowflake. First, an external stage is created to visualize the data without storing it in Snowflake, using the $1 notation. Afterwards, Snowpipes are created, which, through Event Grid notifications in Azure (blob created), enable incremental ingestion into a raw layer. In this layer, the data is stored in table format, where the entire JSON file and its metadata are saved. Next, using dynamic tables (which allow dynamic transformations and reading only the new records for incremental load), we unnest the arrays in the JSON files (using the flatten function), clean and filter the data, and obtain a table with structured data. Finally, a small data warehouse is created with 2 dimension tables and 3 fact tables. This layer or schema is then connected to Power BI to create interactive dashboards.

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

### 1st Pipeline (ingest_http_rebrickable)
- An [Azure Function](scripts/azure-function/web_scraping.py) activity is created to perform web scraping on the page "https://rebrickable.com/downloads/" to obtain download links for the LEGO catalog database.

  This database contains information such as:
  - Official LEGO items - Sets, Parts and Minifigs (no B-Models, Sub-Sets, MOCs)
  - Sets and Minifigs contain one or more Inventories (inventories.csv)
  - Inventories can contain Sets (inventory_sets.csv) and/or Parts (inventory_parts.csv) and/or Minifigs (inventory_minifigs.csv)
  - Part Relationship rel_types are: (P)rint, Pai(R), Su(B)-Part, (M)old, Pa(T)tern, (A)lternate
  <img src="https://i.imgur.com/LYDhQID.png" alt="Database content">
  
- Another activity converts the string of links from the previous activity into an array for processing in the next activity
- A ForEach activity containing a CopyData activity downloads the links and stores them in the raw layer of ADLS Gen2
<img src="https://i.imgur.com/efTcCUZ.png" alt="Pipeline 1">

### 2nd Pipeline (ingest_oltp_db_rebrickable)
- A LookUp activity retrieves table names from the [oltp-database](scripts/oltp-database/) (scripts to generate tables, data, and import data), returning an array of table names
- This array is used in a ForEach activity containing a CopyData activity that moves tables to the raw layer in ADLS Gen2
<img src="https://i.imgur.com/aONY9Sr.png" alt="Pipeline 2">

### 3rd Pipeline (databricks_pipeline)
- Two Databricks notebooks are created:
  1. The first ([link to notebook](pipelines/databricks/notebooks/Databricks_notebook_transform.ipynb)) transforms data (cleaning, filtering, enriching, etc.) and stores the transformed data in Delta format in the silver layer
  2. The second ([link to notebook](pipelines/databricks/notebooks/Data_Modelling.ipynb)) implements dimensional modeling (fact and dimension tables) to address business needs, storing these tables in the gold layer
  
  Model View
  
  <img src="https://i.imgur.com/6nABICN.png" alt="view model">



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



 

 

 


 

