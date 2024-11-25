# README

Questions response : 

### What are each of the files for?

| file | role |
| ------ | ------ |
| models | A directory containing three subfolders, each housing a SQL model file. These SQL files execute queries against Google BigQuery. |
| schema.yml | A YAML file defining source declarations. It specifies the source tables, linking them to the appropriate GCP project and BigQuery dataset.  |
| m0_report.sql | This SQL file executes a query in BigQuery, leveraging Jinja templating (e.g., {{ source('source.name', 'source.table.name') }}) to dynamically reference source tables as defined in schema.yml. For example, it resolves to a table like my-dev-239410.my_domain_workflow.transactions. The query consolidates all transaction data, including refunds, into a unified view.  |
|  m1_report.sql | Similar to m0_report.sql, this query generates a union of detailed transactions and refunded transaction details. |
| default | Acts like the other SQL files but copies the source table and assigns a custom name derived from the DBT_TGT_TABLE_NAME argument, rather than using a default name. |
| dbt_command_loop.sh | A shell script that automates dbt run executions. It accepts two arguments:

1. A YAML configuration file (e.g., table_config.yml or preprod_table_config.yml).
2. The target environment (e.g., preprod, prod).
   
The script parses the YAML file to extract model folder names and relevant dbt arguments, then executes the dbt run command in the specified environment. |
| dbt_project | This is the core project configuration file required to run dbt. It defines critical settings such as the project name, profile, and paths for models, data, snapshots, and seeds. In this use case, tables are materialized as BigQuery tables (though in other scenarios, materializations could be views or materialized views). |
| table_config.yml |This file contains metadata about the pipeline, such as regions, dbt execution schedules (e.g., daily at 10 AM), SLAs, and details for each environment. It includes:

-> Source and target GCP projects.
-> Service accounts and credentials.
-> Paths to scripts like dbt_command_loop.sh and YAML configuration files.
-> Notification settings, such as Slack channels, to ensure visibility of pipeline execution statuses and alerts. |
| preprod_table_config.yml | Functions similarly to table_config.yml but is tailored for the preprod environment. |
### How to run this project and what results we expect.


The presence of the manifest file indicates the manner of executing the "dbt_command_loop.sh" script. It containsthe paths to the script and the table config file, the environment to use as well as other information such as the schedule and src and target projects.
The main way to run this file is to use manifest.yml file in way to execute the "dbt_command_loop.sh" script with two arguments : the first is the table_config path and the second one is the environement 
###### Expemple
```sh
./path/to/dbt_command_loop.sh /path/to/table_config.yml preprod
```
This depends if we are running locally or in Google Cloud. If it is local execution it's just about running the script as mentionned in the exemple and if it's GCP managed several ways can be use such Airflow dags on Cloud Composer or compute engine with cloud scheduler.

The script should execute DBT commands for all specified models, creating tables in the target schema in BigQuery in specific environement ( dev-uk , preprod-ca or prod-ca ) . It should also handle errors, reporting how many commands failed if any did. 
It will actually copy table from "GCP_source_project.my_domain_workflow" and "GCP_source_project.my_domain_appointment" using default model in single directory and "GCP_source_project.event" using m0_report and m1_report datasets to the "GCP_target_project.my_operatianal" dataset.

### List some of this solution's design problems and compare to dbt projects best practices.
##### Design Problems:
###### Complexity of Configuration Management:
With multiple configuration files (e.g., table_config.yml, preprod_table_config.yml), managing changes across environments can be error-prone and cumbersome.
###### Separation of Logic:
Running each model as a separate project may lead to duplicated logic and configurations
###### Error Handling: 
The error handling approach in the shell script may not provide detailed context for failures, making debugging more challenging.

##### Comparison to DBT Best Practices:

 DBT best practices recommend keeping models within a single project to facilitate better management and reduce configuration duplication. This approach also enhances collaboration and code sharing.
The use of testing and documentation is emphasized in best practices, which seems to be underrepresented in this structure.
Streamlining configurations into a more modular structure within a single DBT project is favored over separate projects.

### This project runs every model as a separated dbt project.  
Provide an alternative solution to put everything in one single dbt project that can handle all the environments(targets) and all models.
You don't need to run it, just provide a new repo containing the new solution's code.  
Provid your analysis and suggestions if there are many ways to achieve the same results.

Please see attached my code. I can optimize it using macros
Here I used one single project. I configured the "profile.yml" file so that we can mention the target dataset and GCP project and I configured the "dbt_project.yml" file to handle environements and to exclude models in case of using preprod_ca enviroement(not all table are used in the ancient preprod_table_config.yml). 
Finally I implimented the models folder with the well configured sources and the models with their yml files. 


Feel free to reach to, I will be glad to discuss my work
