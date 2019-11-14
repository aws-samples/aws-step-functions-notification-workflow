[Back to main guide](../README.md)|[Next](optional1.md)

___

# Database Migration Service (DMS)

AWS Database Migration Service helps you migrate databases to AWS easily and securely. The source database remains fully operational during the migration, minimizing downtime to applications that rely on the database. The AWS Database Migration Service can migrate your data to and from most widely used commercial and open-source databases. AWS Database Migration Service can also be used for continuous data replication with high availability.

This lab will walk you through the steps to migrate data from an Oracle database hosted on an EC2 instance to Amazon Aurora PostgreSQL. 

In this activity, you perform the following tasks:

- Create DMS Replication Instance
- Create DMS source and target endpoints
- Run DMS Replication Task for full-load replication
    - Check the source database content for post-replication validation
    - Drop foreign keys and disable triggers on the target database
    - Set up and run a full-load Replication Task
    - Validate the data replication result
    - Restore the foreign keys
- Run DMS Replication Task for Change Data Capture (CDC)
    - Enable CDC on the source database
    - Set up and run a Replication Task
    - Introduce changes at the source database
    - Validate the CDC result at the target database
    - Enable the triggers after the CDC is complete

![DMS](images/dms.png)
___

## Task 1 - Create DMS Replication Instance

1. Go to the [AWS DMS console](https://console.aws.amazon.com/dms/v2/) and click on Replication Instances on the left-hand menu. This will launch the **Replication instance** screen in the Database Migration Service.
2. Click on the **Create replication instance** button on the top right side.
![Create replication instance](images/create_rep_inst.png)
3. Configure the replication instance with the following parameter values. Then, click on the **Create** button.

Parameter | Value
--- | --- 
Name | replication-instance 
Description | Oracle to Aurora DMS replication instance 
Instance Class | dms.t2.medium 
Replication engine version | 2.4.5 
VPC | vpc-xxxxx-GPSTEC315 
Multi-AZ | No 
Publicly accessible | Unchecked 

![Create replication instance](images/create_rep_inst_detail.png)

_Note:Creation of replication instance takes few minutes. While waiting for the replication instance to be created, you can proceede with creation of source and target database endpoints in the next steps. However, you can test connectivity only after the replication instance has been created._

___

## Task 2 - Create DMS source and target endpoints

1. Click on the **Endpoints** link on the left menu, and then click on Create endpoint on the top right corner.

![Create Endpoints](images/create_ep.png)

2. Enter the Connection details for source endpoint from the following parameter values. 

Parameter | Value
--- | ---
Endpoint type | Source endpoint
Endpoint identifier | oracle-source
Source engine | oracle
Server name | Get "EC2SQLInstancePrivateIP" from output of CloudFormation
Port | 1521
SSL mode | none 
User name | hr
Password | hr123
SID | XE

![Create Source Endpoints](images/create_sep.png)

3. Once the information has been entered, click **Run Test**. When the status turns to successful, click **Create endpoint**.

4. Repeat the previous steps to create the target endpoint for Aurora RDS Database with the following parameter values. 

Parameter | Value 
--- | --- 
Endpoint type | Target endpoint 
Endpoint identifier | aurora-postgresql-target 
Source engine | aurora-postgresql 
Server name | Get "DBInstanceEndpointAuroraPostgreSQL" from output of CloudFormation
Port | 5432 
SSL mode | none 
User name | postgres 
Password | Aurora321 
Database name | AuroraPostgreSQLDB 

![Create Target Endpoints](images/create_tep.png)

5. Once the information has been entered, click **Run Test**. When the status turns to successful, click **Create endpoint**.
__

## Task 3 - Run DMS Replication Task for full load replication

#### Check the source database content for post-replication validation
1. Connect to **OracleXE-SCT** EC2 instance using below password from RDP Client. 
    **Windows password**: GPSreInvent@321
2. Launch **SQL Develpoer** from the shortcut on the desktop. 
3. Right Click on `XE` under Connections and select properties to verify the following parameters.

Parameter | Value 
--- | --- 
Connection Name | XE 
Username| hr 
Password | hr123 
Save Password | checked 
Hostname | Get "EC2SQLInstancePrivateIP" from CloudFormation output 
Port| 1521 
SID | XE 

![SQLTargetDB creation](images/create_conn.png)

4. After you connect to the Oracle database successfully, run the following query on the SQL window to get a count of table sizes.

````
SELECT 'regions' TABLE_NAME, COUNT(*) FROM HR.REGIONS  UNION
SELECT 'locations' TABLE_NAME, COUNT(*) FROM  HR.LOCATIONS UNION
SELECT 'departments' TABLE_NAME, COUNT(*) FROM  HR.DEPARTMENTS UNION
SELECT 'jobs' TABLE_NAME,  COUNT(*) FROM HR.JOBS UNION
SELECT 'employees' TABLE_NAME, COUNT(*) FROM  HR.EMPLOYEES UNION
SELECT 'job_history' TABLE_NAME, COUNT(*) FROM  HR.JOB_HISTORY UNION
SELECT 'countries' TABLE_NAME, COUNT(*) FROM HR.COUNTRIES Order by TABLE_NAME;
````
![Read table size](images/table_size.png)

#### Configure the target database schema for full load replication
Before running DMS Replication Task, you need to disable the foreign keys and triggers on the target database. 

1. Right Click on `AuroraPostgreSQL` under Connections and select properties to verify the following parameters.

Parameter | Value
--- | ---
Connection Name | AuroraPostgreSQL
Username| postgres
Password | Aurora321 
Save Password | checked 
Hostname | Get `DBInstanceEndpointAuroraPostgreSQL` from CloudFormation output 
Port| 5432
Database name | AuroraPostgreSQLDB

![Aurora Connection](images/create_conn_aurora.png)

2. Drop foreign keys and disable triggers on the target Aurora database. After you connect to the Aurora PostgreSQL database successfully, run the following query on the SQL window to drop foreign keys.

**Drop foreign keys**

```
ALTER TABLE hr.countries DROP CONSTRAINT country_reg_fk;
ALTER TABLE hr.departments DROP CONSTRAINT dept_loc_fk;
ALTER TABLE hr.departments DROP CONSTRAINT dept_mgr_fk;
ALTER TABLE hr.employees DROP CONSTRAINT emp_dept_fk;
ALTER TABLE hr.employees DROP CONSTRAINT emp_job_fk;
ALTER TABLE hr.employees DROP CONSTRAINT emp_manager_fk;
ALTER TABLE hr.job_history DROP CONSTRAINT jhist_dept_fk;
ALTER TABLE hr.job_history DROP CONSTRAINT jhist_emp_fk;
ALTER TABLE hr.job_history DROP CONSTRAINT jhist_job_fk;
ALTER TABLE hr.locations DROP CONSTRAINT loc_c_id_fk;
```

![Drop foreign keys](images/drop_FK.png)

**Disable TRIGGERS**  

```
DROP TRIGGER IF EXISTS secure_employees ON hr.employees;
DROP TRIGGER IF EXISTS update_job_history ON hr.employees;

```

![Drop foreign keys](images/drop_trigger.png)

#### Configure and run Replication Task
AWS DMS uses Database Migration Task to migrate the data from source to the target database. In this part of the lab you are going to create a Database Migration Tasks for migrating the existing data.

1. Click on **Database migration tasks** on the left-hand menu, then click on the **Create task** button on the top right corner.

![Create replication task](images/create_task.png)

*Enabling the logging would help debugging issues that DMS encounters during data migration*

2. Create a data migration task with the following values for migrating the `HR` database.

Parameter | Value
--- | ---
Task identifier | oracle-migration-task
Replication instance | your replication instance
Source database endpoint | oracle-source
Target database endpoint | aurora-postgresql-target
Migration type | Migrate existing data
Start task on create | Checked
Target table preparation mode | Truncate
Include LOB columns in replication | Limited LOB mode
Max LOB size (KB) | 32
Enable validation | Unchecked
Enable CloudWatch logs | Checked

3. Expand the Table mappings section, and select Guided UI for the editing mode
4. Click on Add new selection rule button and enter the following values:
Parameter | Value
-------- | --------
Schema name | HR
Table name| %
Action | Include

5. Next, expand the Transformation rules section, and click on Add new transformation rule. Then, create the following rules:

Parameter | Value
-------- | --------
Target | Schema
Schema name | HR
Action | Make lowercase

Parameter | Value
-------- | --------
Target | Table
Schema Name | HR
Table Name | %
Action | Make lowercase

Parameter | Value
-------- | --------
Target | Column
Schema Name | HR
Table Name | %
Column Name | %
Action | Make lowercase

![Create task mappings](images/create_task_mappings.png)

6. After entering the values click on **Create task**.
7. At this point, the task should start migrating data from the source Oracle database to the Amazon Aurora RDS instance.
![Migration Task Progress](images/migration_progress.png)
8. Go to **Database migration tasks** to monitor the task progress and once the task status is **Load complete**, your data should have been replicated to the target database.
![Migration Task Progress](images/migration_complete.png)

#### Validate the data replication result 

1. Click on your task **oracle-migration-task** and scroll to the **Table statistics** section to view how many rows have been moved.
    ![Table statistics](images/table_statistics.png)
2. If there is any error, the status color changes from green to red. Click on **View logs** link for the logs.
3. On the target Aurora database, check the tables for migrated data using SQL Developer.
     ![Verify Target datavase](images/verify_target_db.png)

#### Restore the foreign keys 
1. When the full load is complete, enable the foreign key constraints.

```
### Add foreign keys on the target database
ALTER TABLE hr.locations
ADD CONSTRAINT loc_c_id_fk FOREIGN KEY (country_id) 
REFERENCES hr.countries (country_id)
ON DELETE NO ACTION;

ALTER TABLE hr.countries
ADD CONSTRAINT country_reg_fk FOREIGN KEY (region_id) 
REFERENCES hr.regions (region_id)
ON DELETE NO ACTION;

ALTER TABLE hr.employees
ADD CONSTRAINT emp_dept_fk FOREIGN KEY (department_id) 
REFERENCES hr.departments (department_id)
ON DELETE NO ACTION;

ALTER TABLE hr.job_history
ADD CONSTRAINT jhist_dept_fk FOREIGN KEY (department_id) 
REFERENCES hr.departments (department_id)
ON DELETE NO ACTION;

ALTER TABLE hr.departments
ADD CONSTRAINT dept_loc_fk FOREIGN KEY (location_id) 
REFERENCES hr.locations (location_id)
ON DELETE NO ACTION;

ALTER TABLE hr.departments
ADD CONSTRAINT dept_mgr_fk FOREIGN KEY (manager_id) 
REFERENCES hr.employees (employee_id)
ON DELETE NO ACTION;

ALTER TABLE hr.employees
ADD CONSTRAINT emp_manager_fk FOREIGN KEY (manager_id) 
REFERENCES hr.employees (employee_id)
ON DELETE NO ACTION;

ALTER TABLE hr.job_history
ADD CONSTRAINT jhist_emp_fk FOREIGN KEY (employee_id) 
REFERENCES hr.employees (employee_id)
ON DELETE NO ACTION;

ALTER TABLE hr.employees
ADD CONSTRAINT emp_job_fk FOREIGN KEY (job_id) 
REFERENCES hr.jobs (job_id)
ON DELETE NO ACTION;

ALTER TABLE hr.job_history
ADD CONSTRAINT jhist_job_fk FOREIGN KEY (job_id) 
REFERENCES hr.jobs (job_id)
ON DELETE NO ACTION;

```

___
## Task 4 - Create Change Data Capture (CDC) from the source Oracle to the target Aurora database

1. Set up Supplemental Logging for the Oracle database


___

### Conclusion


___

[Back to main guide](../README.md)|[Next](optional1.md)