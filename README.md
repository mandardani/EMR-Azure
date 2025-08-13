EMR Project - Part 2: Building a Data Lakehouse on Azure Databricks
This repository represents Part 2 of an EMR (Electronic Medical Records) project, focusing on building a robust data lakehouse solution on Azure Databricks.

Part 1 of this project can be found here:
https://github.com/mandardani/AzureDataFactory1

Project Overview - Part 2
This phase extends the project by incorporating external web API data, implementing a Medallion Architecture for data refinement, and orchestrating all processes using Databricks notebooks and pipelines.

Data Ingestion and Bronze Layer
We integrate data from various sources into the Bronze Layer, ensuring all raw data is captured and stored in Parquet format.

Web API Data Pulls:

Create a PySpark notebook in Databricks to call the ICD10 Web API:
https://github.com/mandardani/EMR-Azure/blob/main/bronze/webservice-data-pull/webservice_icd10_pull.ipynb

Create a PySpark notebook in Databricks to call the NPI Web API:
https://github.com/mandardani/EMR-Azure/blob/main/bronze/webservice-data-pull/webservice_npi_data_pull.ipynb

Claims and CPT Data Upload:

Claims and CPT data, initially in CSV format in the landing zone, are uploaded to the Bronze layer in Parquet format using dedicated notebooks.

Notebooks for uploading scheduled activities:
https://github.com/mandardani/EMR-Azure/tree/main/bronze/scheduled_activities

Integration with Part 1:

A pipeline will be created to orchestrate the execution of the above notebooks, alongside the pipeline from Part 1, which uploads data to the Bronze layer from SQL databases (Hospital1 & Hospital10).

Medallion Architecture
The data flows sequentially through Bronze, Silver, and Gold layers, progressively adding more structure, quality, and business value.

Silver Layer (Cleaned and Conformed Data)
Purpose: To cleanse, refine, and standardize the data from the Bronze layer, making it more reliable and usable for downstream consumption.

Characteristics:

Data undergoes initial transformations: parsing, cleaning (handling nulls, duplicates, inconsistent formats).

Enrichment: Adding metadata and schema enforcement.

Integration: Aims to provide an "enterprise view" by integrating data from different sources into common business entities (e.g., master patient data, unified provider lists).

Use: Suitable for exploratory data analysis, ad-hoc reporting, and as a source for building more refined datasets for the Gold layer.

Steps to move data to the Silver layer:

Design Common Data Model: Establish a common data model for all tables. This includes designing record-level keys (e.g., primary key + data source identifier like hospital1/hospital10) to ensure unique identification of records across integrated sources. For example, if both Hospital1 and Hospital10 have department number 001, they must be uniquely identifiable after integration.

Read Data: Ingest data from both Hospital1 and Hospital10 instances.

Data Validation: Implement validation rules. If a record fails validation (e.g., patient DOB or first name is null), set a quarantine flag to true.

Overwrite Tables (Departments, Providers, ICD, CPT): For static or slowly changing reference data, tables like Departments, Providers, ICD, and CPT will be overwritten. This data is considered read-only.

SCD Type 2 (Patient, Encounter, Transaction): For frequently changing data like patient, encounter, and transaction tables, implement Slowly Changing Dimension (SCD) Type 2. This involves adding columns like 'is_active', inserted_date, and modified_date to track changes and maintain historical versions of records.

Silver Layer Notebooks:

https://github.com/mandardani/EMR-Azure/tree/main/silver

Gold Layer (Curated/Analytics-Ready Data)
Purpose: To transform and aggregate data from the Silver layer into highly refined, optimized datasets ready for specific business intelligence, analytics, and machine learning use cases.

Characteristics:

Data is often denormalized, aggregated, and structured into star schemas (fact and dimension tables) or other user-friendly formats.

Includes business logic, KPIs, and pre-calculated metrics tailored to specific reporting needs.

Highly curated and optimized for fast query performance.

Use: Directly consumed by business users, dashboards, reporting tools, and machine learning models for high-value insights and decision-making.

Steps to move data to the Gold layer:

Recognize Dimension and Fact Tables: Identify and separate dimension data from fact data.

Filter Active Records: For SCD Type 2 data, only move active records to the Gold layer to ensure the most current view.

Filter Non-Quarantine Data: Exclude any records flagged as quarantine from the Gold layer to ensure data quality.

Add Foreign Keys: Incorporate foreign keys into fact tables (e.g., the transaction table will have foreign keys referencing Department, Patient, and Providers dimension tables) to facilitate analytical queries.

Gold Layer Notebooks:

https://github.com/mandardani/EMR-Azure/tree/main/gold

Pipeline Orchestration
We will design a series of Azure Data Factory (ADF) pipelines to orchestrate the execution of Databricks notebooks, ensuring a structured and automated data flow.

Individual Pipelines:

Pipeline for reading data from Web APIs (ICD and NPI).

Pipeline for moving and cleaning data from Bronze to Silver.

Pipeline for moving and transforming data from Silver to Gold.

Master Pipeline:

A master pipeline will be designed to run all individual pipelines in a sequential order:

Move Data from Database instances to Bronze layer (Part 1).

Retrieve NPI & ICD Data from Web APIs.

Move & Clean Data from Bronze to Silver.

Move Data from Silver to Gold.
