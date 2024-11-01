


source dbt_env_310/bin/activate
pip install dbt-databricks


pip install databricks-cli

databricks configure --token

                 https://adb-3245733125902016.16.azuredatabricks.net

                 dapi56db420b7f453d0589e7a1008ec993ad-3

databricks secrets list-scopes
databricks fs ls


dat init medallion_dbt_spark
