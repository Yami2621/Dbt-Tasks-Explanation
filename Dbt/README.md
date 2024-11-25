# README

Questions response : 

### What are each of the files for?


| File | Role |
|------|------|
| **models/** | A directory containing three subfolders, each with a SQL model file that runs queries on Google BigQuery. |
| **schema.yml** | A YAML file defining source declarations, specifying source tables from the GCP project and BigQuery dataset. |
| **m0_report.sql** | Executes a query in BigQuery, using Jinja templates (e.g., `{{ source('source.name', 'source.table.name') }}`) to dynamically reference source tables defined in `schema.yml`. It consolidates all transactions and refunds into a single table. |
| **m1_report.sql** | Similar to `m0_report.sql`, but generates a union of detailed transactions and refunded transaction details. |
| **default.sql** | Similar to other SQL files, but copies the source table while assigning a custom name derived from the `DBT_TGT_TABLE_NAME` argument. |
| **dbt_command_loop.sh** | A script that automates `dbt run` commands. Accepts two arguments: a YAML file (e.g., `table_config.yml` or `preprod_table_config.yml`) and the target environment (`preprod`, `prod`, etc.). It parses the YAML to extract dbt arguments and executes the `dbt run` command in the specified environment. |
| **dbt_project.yml** | The core configuration file for running dbt. Defines project settings such as the project name, profile, and paths for models, data, snapshots, and seeds. Tables in this use case are materialized as BigQuery tables but could be views or materialized views in other cases. |
| **manifest.yml** | Metadata file containing details about regions, dbt run schedules (e.g., daily at 10 AM), SLAs, environments, source/target GCP projects, service accounts, and credentials. It also specifies paths for scripts (e.g., `dbt_command_loop.sh`) and configuration files, as well as Slack channels for notifications about pipeline status and issues. |
| **table_config.yml** | YAML configuration parsed by `dbt_command_loop.sh`. Includes model folder names (`m0_report`, `m1_report`, `default`), source/target GCP project details, and dataset/table info. These settings are used in `schema.yml` and SQL model queries. |
| **preprod_table_config.yml** | Similar to `table_config.yml` but tailored for the `preprod` environment. |

## How to Run This Project  

### Execution Modes  

1. **Local Execution**  
   - Run the `dbt_command_loop.sh` script directly with the required arguments.  
     ```bash
     ./dbt_command_loop.sh <table_config.yml_path> <environment>
     ```  
   - Replace `<environment>` with the target environment (`preprod`, `prod`, etc.).  

2. **Google Cloud Execution**  
   - Deploy the script using **Airflow DAGs** (via Cloud Composer) or **Cloud Scheduler** with Compute Engine.  

---

### Data Pipeline Overview  

- The script copies tables from the following source datasets:  
  - `GCP_source_project.my_domain_workflow`  
  - `GCP_source_project.my_domain_appointment`  
  - `GCP_source_project.event`  
- Transformed data is stored in the target dataset:  
  - `GCP_target_project.my_operational`  
- Models (`m0_report`, `m1_report`, and `default`) process and transform the data for various use cases.  

The pipeline ensures:  
- Successful execution of dbt commands for all models in the specified environment.  
- Error reporting to summarize failures and aid debugging.  

---

## Design Challenges and Comparison to dbt Best Practices  

### Design Challenges  

1. **Configuration Management Complexity**  
   - Multiple YAML configuration files (`table_config.yml`, `preprod_table_config.yml`) make managing changes across environments error-prone.  

2. **Separation of Logic**  
   - Running each model as a separate project duplicates logic and configuration, increasing maintenance overhead.  

3. **Error Handling**  
   - The shell script lacks robust error reporting, making it difficult to debug issues in complex pipelines.  

---

### Best Practices in dbt  

1. **Unified Project Structure**  
   - dbt recommends consolidating models within a single project to streamline workflows and reduce duplication.  

2. **Testing and Documentation**  
   - Incorporate tests for all models and detailed documentation in YAML files for better maintainability and data quality assurance.  

3. **Modular Configurations**  
   - Use macros and reusable configurations to simplify SQL logic and improve scalability.  

---

## Proposed Solution: Single dbt Project  

### Key Improvements  

1. **Unified Structure**  
   - Consolidate all models into one dbt project with a structured directory for sources, models, and YAML files.  

2. **Dynamic Environment Handling**  
   - Configure `profiles.yml` for dynamic environment selection (e.g., `dev-uk`, `preprod-ca`).  
   - Update `dbt_project.yml` to handle environment-specific model exclusions (e.g., exclude unnecessary tables in `preprod-ca`).  

3. **Macros for Reusability**  
   - Introduce macros to eliminate repetitive SQL logic and enhance readability.  

4. **Enhanced Error Handling**  
   - Implement detailed logging for dbt command execution to capture errors with context.  

5. **Testing and Validation**  
   - Add tests for all sources and models to ensure data integrity and compliance with business requirements.  

---

### Example Workflow in the Unified Project  

1. Configure `profiles.yml`:  
   - Define credentials and datasets for each environment.  

2. Execute dbt commands:  
   - Run transformations for all models:  
     ```bash
     dbt run --target <environment>
     ```  

3. Use macros and YAML configurations for modular and scalable workflows.  

---

Feel free to reach out for further discussions or clarifications regarding this solution. I look forward to collaborating to refine and optimize this project!  
