


source dbt_env_310/bin/activate
pip install dbt-databricks


pip install databricks-cli

databricks configure --token

                 https://adb-3245733125902016.16.azuredatabricks.net

                 dapi56db420b7f453d0589e7a1008ec993ad-3

databricks secrets list-scopes
databricks fs ls


dat init medallion_dbt_spark


Data Bricks Notebook 
"""  
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
"""  
