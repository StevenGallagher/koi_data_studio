# koi_data_studio
koi data studio - AI powered data ingestion, transformation and delivery

ingestion
 - via dlt: output to a landing database / schema per source system

transformation
 - via dbt
 - landing to bronze: create views for current (active records) and history (previous records)
 - bronze to silver: create data vault using automate_dv package
 - silver to gold: create kimball style fact/dimension tables, and specific well documented metric views in a semantic layer

delivery
 - via plottly: if use case is BI
 - via fastapi: if use case is a data product / application
 
 storage
  - via sqlite: if use case is transactional workload
  - via duckdb: if use case is analytical workload