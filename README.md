# azure-databricks-airflow-example
Demo orchestrating a data pipeline based on Azure [Databricks](https://databricks.com/) jobs using [Apache Airflow](https://airflow.apache.org/).

### Apache Airflow & Databricks

Databricks offers an Airflow operator to submit jobs in Databricks. The Databricks Airflow operator calls the [Jobs Run API](https://docs.databricks.com/dev-tools/api/latest/jobs.html#jobsjobsservicerunnow) to submit jobs. These APIs automatically create new clusters to run the jobs and also terminates them after running it. This will minimize cost because in that case you will be charged at lower [Data Engineering DBUs](https://azure.microsoft.com/en-us/pricing/details/databricks/).

The [Airflow documentation](https://airflow.apache.org/docs/stable/) gives a very comprehensive overview about design principles, core concepts, best practices as well as some good working examples. In addition the [Databricks documentation](https://docs.databricks.com/dev-tools/data-pipelines.html#apache-airflow) provide further details.

#### Installation and configuration of Apache Airflow
1. Install airflow using pip: `pip install airflow`
2. Setup the database: `airflow upgradedb`
3. Start the scheduler: `airflow scheduler`
4. Start the web server: `airflow webserver`
5. Create a Access Token in your Databricks workspace, used in the connection configuration
6. Configure the connection to your Databricks workspace with below code snippet

```bash
airflow connections --add \
	--conn_id adb_workspace \
	--conn_type databricks \
	--conn_login token \
	--conn_extra '{"token": "<your-token>", "host":"<your-host>"}'
```

7. Open the airflow web UI: `http://<hostname>:8080` and click on 'Admin' and select 'Connections'. Check the adb_workspace connection.

#### Creating the Airflow DAG for the data pipline
Airflow workflows are defined in Python scripts, which provide a set of building blocks to communicate with a wide array of technologies (bash scripts, python functions etc.). Basically, a workflow consist of a series of tasks modeled as a Directed Acyclic Graph or DAG.

```python
import airflow
from airflow import DAG
from airflow.contrib.operators.databricks_operator import DatabricksRunNowOperator
from datetime import timedelta, datetime

# default arguments
args = {
    'owner': 'Airflow',
    'email': ['<your-email-address>'],
    'email_on_failure' : True,
    'depends_on_past': False,
    'databricks_conn_id': 'adb_workspace'
}

# DAG with Context Manager
# refer to https://airflow.apache.org/docs/stable/concepts.html?highlight=connection#context-manager
with DAG(dag_id='adb_pipeline', default_args=args, start_date=airflow.utils.dates.days_ago(1), schedule_interval='4 30 * * *') as dag:

	# job 1 definition and configurable through the Jobs UI in the Databricks workspace
	notebook_1_task = DatabricksRunNowOperator(
		task_id='notebook_1',
		job_id=1, 
		json= {
			"notebook_params": {
				'inPath': '/bronze/uber'
			}	
		})

	# Arguments can be passed to the job using `notebook_params`, `python_params` or `spark_submit_params`
	json_2 = {
		"notebook_params": {
			'inPath': '/bronze/kaggle'
		}
	}

	notebook_2_task = DatabricksRunNowOperator(
		task_id='notebook_2',
		job_id=2, 
		json=json_2)

	# input parameters for job 3
	json_3 = {
		"notebook_params": {
			'bronzePath': '/bronze/',
			'silverPath': '/silber'
		}
	}

	notebook_3_task = DatabricksRunNowOperator(
		task_id='notebook_3',
		job_id=3, 
		json=json_3)

	# Define the order in which these jobs must run using lists
	[notebook_1_task, notebook_2_task] >> notebook_3_task

```

To see the full list of DAGs available, run `airflow list_dags`.
If you want to test certain tasks, run `airflow test adb_pipeline notebook_2_task 2019-12-19T10:03:00`.
You can enable or trigger your DAG in the scheduler using the web UI or trigger it manually using: `airflow trigger_dag adb_pipeline`.