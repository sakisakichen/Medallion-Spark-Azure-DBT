## Project Intro 
This project complete data engineering pipeline using Apache Spark, Azure Databricks, and Data Build Tool (DBT) on the Azure cloud platform.   
This solution guides the process from data ingestion into the lakehouse, through data integration with Azure Data Factory (ADF), and finally to data transformation using Databricks and DBT


## Project Architecture 
![image](https://github.com/user-attachments/assets/621477c1-438a-41ff-b00c-de5353ee21db)

## Data Source 
The data used in this project is sample data (AdventureWorksLT) from an Azure SQL Database.   
It represents a fictional bicycle manufacturer, Adventure Works Cycles, and reflects various online transaction processing scenarios.

Included Scenarios:
This dataset includes data for scenarios such as manufacturing, sales, purchasing, product management, contact management, and human resources.

## Fetch Data on ADF 
(1) Data fetch from SQL database by Lookup <br>
(2) Copy data and store into Gen2 bronze folder  <br>

##  Databricks Configure & connect to ADF
<details>
<summary> </summary>
(1) Databricks Secrets Create Scope  <br>
    https://adb-3245733125902016.16.azuredatabricks.net/?o=3245733125902016#secrets/createScope
        

(2) Azure Key Vault(secret value of data container): add DNS Name & Resource ID to Databricks  
      ` Key Vault properties find Vault URI for DNS Name & Resource ID `

(3) Create Compute Cluster 
    
(4) Mount all the storage accounts to DBFS (Databricks File System) 

```python
dbutils.fs.mount(
    source= 'wasbs://bronze@medallionsaki.blob.core.windows.net',
    mount_point = '/mnt/bronze',
    extra_configs = {'fs.azure.account.key.medallionsaki.blob.core.windows.net':dbutils.secrets.get('DataBricksScope','StorageAccountKey')}
)

dbutils.fs.mount(
    source= 'wasbs://silver@medallionsaki.blob.core.windows.net',
    mount_point = '/mnt/silver',
    extra_configs = {'fs.azure.account.key.medallionsaki.blob.core.windows.net':dbutils.secrets.get('DataBricksScope','StorageAccountKey')}
)

dbutils.fs.mount(
    source= 'wasbs://gold@medallionsaki.blob.core.windows.net',
    mount_point = '/mnt/gold',
    extra_configs = {'fs.azure.account.key.medallionsaki.blob.core.windows.net':dbutils.secrets.get('DataBricksScope','StorageAccountKey')}
)
```

```python
dbutils.fs.ls('/mnt/bronze')
```

(5) Connect to ADF  <br>
(5-1) Link service create  <br>
Databaricks URL provide & Access Token( Databricks user setting- preferences to generate new token)  <br>
(5-2) Dump data into Databricks
```python
# Define widgets
dbutils.widgets.text('fileName', '', 'File Name')
dbutils.widgets.text('table_schema', '', 'Table Schema')
dbutils.widgets.text('table_name', '', 'Table Name')

fileName = dbutils.widgets.get('fileName')
tableSchema = dbutils.widgets.get('table_schema')
tableName = dbutils.widgets.get('table_name')

# Ensure tableSchema is not empty
if not tableSchema:
    raise ValueError("table_schema cannot be empty")

# Create a database if it doesn't exist, using backticks for safety
spark.sql(f"CREATE DATABASE IF NOT EXISTS `{tableSchema}`")

# If the table is not existing in the database, create it
spark.sql(f"""
          CREATE TABLE IF NOT EXISTS `{tableSchema}`.`{tableName}`
          USING PARQUET 
          LOCATION '/mnt/bronze/{fileName}/{tableSchema}.{tableName}.parquet'
          """)
```
</details>



##  DBT Core Set up & DBT Command

<details>
<summary> </summary>

```
pip install dbt-databricks
pip install databricks-cli

databricks configure --token

                 databricks host:  https://adb-3245733125902016.16.azuredatabricks.net

                 token: (new token generated on databricks user setting- preferences )

databricks secrets list-scopes
databricks fs ls
dat init medallion_dbt_spark

```

```
dbt run # for running models
dbt test # for tests
dbt snapshot # for snapshotting and slowly changing dimensions
dbt docs generate # for documentation
dbt docs serve # for documentation preview
```


</details>


## Reference 
https://github.com/airscholar/modern-data-eng-dbt-databricks-azure/blob/main/README.md
