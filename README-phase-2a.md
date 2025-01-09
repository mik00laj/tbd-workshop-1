IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.

![img.png](doc/figures/destroy.png)

0. The goal of this phase is to create infrastructure, perform benchmarking/scalability tests of sample three-tier lakehouse solution and analyze the results using:
* [TPC-DI benchmark](https://www.tpc.org/tpcdi/)
* [dbt - data transformation tool](https://www.getdbt.com/)
* [GCP Composer - managed Apache Airflow](https://cloud.google.com/composer?hl=pl)
* [GCP Dataproc - managed Apache Spark](https://spark.apache.org/)
* [GCP Vertex AI Workbench - managed JupyterLab](https://cloud.google.com/vertex-ai-notebooks?hl=pl)

Worth to read:
* https://docs.getdbt.com/docs/introduction
* https://airflow.apache.org/docs/apache-airflow/stable/index.html
* https://spark.apache.org/docs/latest/api/python/index.html
* https://medium.com/snowflake/loading-the-tpc-di-benchmark-dataset-into-snowflake-96011e2c26cf
* https://www.databricks.com/blog/2023/04/14/how-we-performed-etl-one-billion-records-under-1-delta-live-tables.html

2. Authors:

   ***z2***

   ***[Link to forked repo](https://github.com/mik00laj/tbd-workshop-1)***

3. Sync your repo with https://github.com/bdg-tbd/tbd-workshop-1.

4. Provision your infrastructure.

    a) setup Vertex AI Workbench `pyspark` kernel as described in point [8](https://github.com/bdg-tbd/tbd-workshop-1/tree/v1.0.32#project-setup) 

    b) upload [tpc-di-setup.ipynb](https://github.com/bdg-tbd/tbd-workshop-1/blob/v1.0.36/notebooks/tpc-di-setup.ipynb) to 
the running instance of your Vertex AI Workbench

5. In `tpc-di-setup.ipynb` modify cell under section ***Clone tbd-tpc-di repo***:

   a)first, fork https://github.com/mwiewior/tbd-tpc-di.git to your github organization.
   
![image](https://github.com/user-attachments/assets/d92da49e-a3df-4aab-b7e8-a53dfa122caf)

   b)create new branch (e.g. 'notebook') in your fork of tbd-tpc-di and modify profiles.yaml by commenting following lines:
   ```  
        #"spark.driver.port": "30000"
        #"spark.blockManager.port": "30001"
        #"spark.driver.host": "10.11.0.5"  #FIXME: Result of the command (kubectl get nodes -o json |  jq -r '.items[0].status.addresses[0].address')
        #"spark.driver.bindAddress": "0.0.0.0"
   ```
   This lines are required to run dbt on airflow but have to be commented while running dbt in notebook.
   ![image](https://github.com/user-attachments/assets/789965ce-c93e-4da7-8809-841ad505c6d4)


   c)update git clone command to point to ***your fork***.


7. Access Vertex AI Workbench and run cell by cell notebook `tpc-di-setup.ipynb`.

    a) in the first cell of the notebook replace: `%env DATA_BUCKET=tbd-2023z-9910-data` with your data bucket.
Our variables are like below:
![image](https://github.com/user-attachments/assets/a65defaa-4336-4dfc-aa98-ff26b16b6f45)


   b) in the cell:
         ```%%bash
         mkdir -p git && cd git
         git clone https://github.com/mwiewior/tbd-tpc-di.git
         cd tbd-tpc-di
         git pull
         ```
      replace repo with your fork. Next checkout to 'notebook' branch.
   I changed this by merging notebook branch to master like in a screenshot in 5 b) point.
    c) after running first cells your fork of `tbd-tpc-di` repository will be cloned into Vertex AI  enviroment (see git folder).

    d) take a look on `git/tbd-tpc-di/profiles.yaml`. This file includes Spark parameters that can be changed if you need to increase the number of executors and
  ```
   server_side_parameters:
       "spark.driver.memory": "2g"
       "spark.executor.memory": "4g"
       "spark.executor.instances": "2"
       "spark.hadoop.hive.metastore.warehouse.dir": "hdfs:///user/hive/warehouse/"
  ```

7. Explore files created by generator and describe them, including format, content, total size.

For each content the generator generate files and audit files in csv format for them. The generator works in batches and final files are loaded in 3 batches and saved in folders represented these batches, in order to Date column, which is visible in audit file, which are also generated for batches and have the same columns like below:
***DataSet, BatchID ,Date , Attribute , Value, DValue***
Not every column is filled with value, it depends of the content and if it's batch audit or a file or a generator.
So finally the generator return a catalog like below in a declared path /tmp/tpc-di with sizes written in 5th column:
```
drwxr-xr-x 2 root root  20K Jan  8 20:47 Batch1
-rw-r--r-- 1 root root  113 Jan  8 20:34 Batch1_audit.csv
drwxr-xr-x 2 root root 4.0K Jan  8 20:43 Batch2
-rw-r--r-- 1 root root  113 Jan  8 20:34 Batch2_audit.csv
drwxr-xr-x 2 root root 4.0K Jan  8 20:43 Batch3
-rw-r--r-- 1 root root  113 Jan  8 20:34 Batch3_audit.csv
-rw-r--r-- 1 root root 3.2K Jan  8 20:34 Generator_audit.csv
-rw-r--r-- 1 root root  587 Jan  8 20:47 digen_report.txt
```

And additionally we know from digen_report.txt how many records we generate:
  ***AuditTotalRecordsSummaryWriter - TotalRecords all Batches: 162228471 215250.76 records/second***

Something more about created files, one for each content:

StatusType:
  - format: txt
  - content: an abstract for a status to classify trades and their description

TaxRate:
  - format: txt
  - content: tax rate information

Date:
  - format: txt
  - content: calendar-related information, including calendar year, quarter, month, week, day details, and holiday flag

Time:
  - format: txt
  - content: time-related data formatted with a fixed schema, likely representing temporal logging or synthetic time series data for the TPC-DI workload

BatchDate:
  - format: txt
  - content: dates associated with individual batch runs in the TPC-DI benchmark, simulating batch data integration processes

HR:
  - format: txt
  - content: employee hierarchy and details (name, surname, workplace, phone number, etc.)

CustomerMgmt:
  - format: xml
  - content: customer details in XML format, including personal details, tax information, contact details, and account-related attributes

Customer:
  - format: txt
  - content: data about clients, especially financial ones, including credits, net worth, etc.

Account:
  - format: txt
  - content: data about customer accounts and their status from the broker's perspective

Prospect:
  - format: csv
  - content: information about prospects, including personal details (name, gender, marital status, income, credit rating, etc.), address, and contact information

Industry:
  - format: txt
  - content: industry classifications

FINWIRE:
  - format: binary
  - content: archival data from financial records, securities data, and company information

DailyMarket:
  - format: txt
  - content: market-related data

WatchHistory:
  - format: txt
  - content: historical watch data about trades

TradeSource (Trade i TradeHistory):
  - format: txt
  - content: files containing transaction-related data

TradeType:
  - format: txt
  - content: trade type definitions

8. Analyze tpcdi.py. What happened in the loading stage?

   This is the script that caused locally stored data to be sent to cloud storage.
    We can distinguish phases such as:
    1. data processing in tcp-di
    2. creation of storage for the data
    3. transfer of data to storage
    4. transfer of data to spark dataFrames
    ![image](https://github.com/user-attachments/assets/6919387e-91f5-4dda-8738-f2ef02d20518)


9. Using SparkSQL answer: how many table were created in each layer?

   ![image](https://github.com/user-attachments/assets/3c59b21a-f04a-4c0d-8a66-62287f0038e3)


10. Add some 3 more [dbt tests](https://docs.getdbt.com/docs/build/tests) and explain what you are testing.
    Tests are added to /tests/ folder in forked repo https://github.com/mik00laj/tbd-tpc-di/tree/main/tests.
    ```
    -- Check if primary key are not null
    SELECT sk_customer_id
    FROM {{ ref('dim_customer') }}
    WHERE sk_customer_id IS NULL

    -- Check if primary key are unique
    SELECT sk_account_id, COUNT(*) AS count
    FROM {{ ref('dim_account') }}
    GROUP BY sk_account_id
    HAVING count > 1

    -- Check if 'tax' values are bigger or equal to 0
    SELECT *
    FROM {{ ref('trades') }} 
    WHERE tax < 0
    ```

    ![image](https://github.com/user-attachments/assets/9636b62e-5e4e-4000-9b74-91c4770ba17b)


12. In main.tf update
   ```
   dbt_git_repo            = "https://github.com/mwiewior/tbd-tpc-di.git"
   dbt_git_repo_branch     = "main"
   ```
   so dbt_git_repo points to your fork of tbd-tpc-di. 
   ```
   dbt_git_repo            = "https://github.com/mik00laj/tbd-tpc-di.git"
   dbt_git_repo_branch     = "main"
   ```

12. Redeploy infrastructure and check if the DAG finished with no errors:

![image](https://github.com/user-attachments/assets/4ba15bad-544c-403c-a99a-ff43fc8f8b9d)

