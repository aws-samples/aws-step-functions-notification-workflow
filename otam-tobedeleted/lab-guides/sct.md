[Back to main guide](../README.md)|[Next](dms.md)

___

# Schema Conversion Tool (SCT)

The following steps provide instructions for converting an Oracle database to an Amazon Aurora PostgreSQL database. In this activity, you perform the following tasks:
- Log in to Windows EC2 instance
- Use the Schema Conversion Tool to create a database migration project
- Use SCT to convert the Oracle schema to PostgreSQL schema and analyze schema
conversion issues
- Apply the converted schema to the Aurora PostgreSQL database

    ![Connect to EC2 Instance](images/sct.png)
___

## Task 1 - Log in to Windows EC2 instance

1. Go to the [Amazon EC2 console](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home:) and click on Instances in the left column.
2. Select the instance with the name **OracleXE-SCT** and then click the **Connect** button.

    _Note : If you need any assistance connecting to your instance, please see our [connection documentation](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/connecting_to_windows_instance.html?icmpid=docs_ec2_console)._

![Connect to EC2 Instance](images/instance-connect.png)

3. Click on **Download Remote Desktop File** to download the RDP file to connect to EC2 instance **OracleXE-SCT**. 
4. Connect to EC2 instance using below password from RDP Client. 

    **Windows password**: GPSreInvent@321

___

## Task 2 - Launch Schema Conversion Tool on the server
Now that you are connected to the EC2 instance **OracleXE-SCT**, launch the Schema Conversion Tool from Application menu or shortcut on the desktop.

1. Launch SCT from the shortcut on the desktop and create a new Database Migration Project using the tool.
![Create New Project](images/new_project.png)
2. Enter the following values into the form and then click **OK**.

    Parameter | Value
    ----------|------
    Project Name | AWS Schema Conversion Tool Oracle to Aurora PostgreSQL
    Location | Leave default
    Database type | Transactional Database (OLTP)
    Source database engine | Oracle 
    Target database engine | Amazon Aurora(PostgreSQL compatible)

    ![Create New Project](images/new_project1.png)

3. Click **Connect to Oracle** on the Select Template page. Enter the source database configurations in the form, and click **Test Connection**. Once
the connection is successfully tested, click **Next**.

    Parameter | Value
    ----------|------
    Type | SID
    Server Name | localhost
    Server Port | 1521
    Oracle SID | XE 
    User Name | hr
    Password |  hr123 
    Use SSL | Unchecked
    Save Password | Checked
    
    ![Connect to Oracle](images/sct_oracle.png)

4. Click **Connect to Amazon Aurora(PostgreSQL compatible)** on the Select Template page. Enter the source database configurations in the form, and click **Test Connection**. Once
the connection is successfully tested, click **Next**.

    Parameter | Value
    ----------|------
    Server Name | <DBInstanceEndpointAuroraPostgreSQL> from cloud formation output
    Server Port | 5432
    Database | AuroraPostgreSQLDB 
    User Name | postgres
    Password |  Aurora321 
    Use SSL | Unchecked
    Save Password | Checked
    
    ![Connect to Amazon Aurora](images/sct_aurora.png)

5. Select and Expand the `HR` schema from the left-hand panel to inspect the schema objects.
    
    ![Schema Objects](images/schema_objects.png)
___

## Task 3 - Convert the Schema Using the Schema Conversion Tool
Now that you have created a new Database Migration Project, the next step is to convert the Oracle schema of the source database to that of target Amazon Aurora PostgreSQL database.

1. Right-click on the `HR` schema from Oracle source and select **Convert Schema** to generate the data definition language (DDL) statements for the target database.

_Note: You may be prompted with a dialog box “These objects might already exist in the target database, Replace?” Select **Yes** and conversion will start._

![Convert Schema](images/convert_schema.png)

AWS SCT analyses the schema and creates a database migration assessment report for the conversion to PostgreSQL. Items with a red exclamation mark next to them cannot be directly translated from the source to the target. In this case,it includes Functions and/or Procedures and Views.

2. Click on the **View** button, and choose **Assessment Report view** to view detailed assesment report.

![Assessment Report view](images/assessment1.png)    

3. Review the assessment report and examine whether there are suggested actions to address schema conversion issues. 

4. Next, navigate to the **Action Items** tab in the report to see the items that have issues converting to the new schema. 

![Action Items](images/action-items.png) 

Check each of the issues listed and compare the contents under the source Oracle panel and the target Aurora PostgreSQL panel. Are the issues resolved? And how? SCT has proposed resolutions by generating equivalent PostgreSQL DDL to convert the objects. Additionally, SCT highlights each conversion issue where it cannot automatically generate a conversion, and provide hints on how you can successfully convert the database object.

Notice the issue highlighted in the procedure named `ADD_JOB_HISTORY`. You’ll see that SCT is unable to automatically convert procedure. You can complete one of the following actions to fix the issue:
    - Modify the objects on the source Oracle database so that AWS SCT can convert the objects to the target Aurora PostgreSQL database.
    - Instead of modifying the source schema, modify scripts that AWS SCT generates before applying the scripts on the target Aurora PostgreSQL database.

Refer to [Migration Playbook](https://d1.awsstatic.com/whitepapers/Migration/oracle-database-amazon-aurora-postgresql-migration-playbook.pdf) for best practices to convert Oracle to Aurora-PostgreSQL compatible schema. 

5. Manually fix the schema issue. Then, right-click on the `HR` schema,
and choose Create Report. Observe that the schema of the source database is now fully compatible with the target database.

6. To migrate the converted schema in the target database, Right click on the `HR` schema in the right-hand panel, and click **Apply to database**.

![Apply to database](images/apply_db.png) 

7. When prompted if you want to apply the schema to the database, click **Yes**.
8. At this point, the schema has been applied to the target database. Expand the `HR` schema to see the tables.
    _Note: If you don’t see the tables, right click on `HR` schema and select **Refresh from Database**._

### Conclusion

This part of the workshop has demonstrated how to successfully convert and migrate the schema from an Oracle database to an Amazon Aurora PostgreSQL database. The AWS Schema Conversion Tool (SCT) largely automates the schema conversion. These same steps can be followed to migrate SQL Server and Oracle workloads to other Amazon RDS engines including Aurora MySQL and MySQL.
___

[Back to main guide](../README.md)|[Next](dms.md)