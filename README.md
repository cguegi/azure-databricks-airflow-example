# azure-databricks-airflow-example
Demo orchestrating a data pipeline based on Azure [Databricks](https://databricks.com/) jobs using [Apache Airflow](https://airflow.apache.org/).

###Apache Airflow & Databricks

Databricks offers an Airflow operator to submit jobs in Databricks. The Databricks Airflow operator calls the [Jobs Run API](https://docs.databricks.com/dev-tools/api/latest/jobs.html#jobsjobsservicerunnow) to submit jobs. These APIs automatically create new clusters to run the jobs and also terminates them after running it. This will minimize cost because in that case you will be charged at lower [Data Engineering DBUs](https://azure.microsoft.com/en-us/pricing/details/databricks/).

The [Airflow documentation](https://airflow.apache.org/docs/stable/) gives a very comprehensive overview about design principles, core concepts, best practices as well as some good working examples.

In addition the [Databricks documentation](https://docs.databricks.com/dev-tools/data-pipelines.html#apache-airflow) provide further details.

####Installation and configuration of Apache Airflow
1. Install airflow using pip: `pip install airflow`
2. Setup the database: `airflow upgradedb`
3. Start the scheduler: `airflow scheduler`
4. Start the web server: `airflow webserver`
5. Create a Access Token in your Databricks workspace
6. Configure the connection to your Databricks workspace with below code snippet

<code>
airflow connections --add \
	--conn_id adb_workspace \
	--conn_type databricks \
	--conn_login token \
	--conn_extra '{"token": "<your-token>", "host":"<your-host>"}'
</code>

7. Open the airflow ui: http://<host>:8080 and click on 'Admin' and select 'Connections'. Check the adb_workspace connection.