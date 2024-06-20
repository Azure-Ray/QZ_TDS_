Background
Utilizing tools like Liquibase for SQL version control ensures that SQL executions are managed systematically, thereby preventing repeated or missed executions in the production environment. Liquibase facilitates the management and control of database changes through changesets, ensuring the traceability and consistency of database modifications.

Liquibase URL: Liquibase

Usage Instructions
Register JDBC Addresses and Names

Register the required JDBC addresses and names in key-value format in the GitHub configuration file: jdbc_list.yaml. Once registered, the tool will automatically update these in the executable dropdown menu.

Create Project Folder

Use the key from the registration as the folder name, ensuring consistency. For example, for the Taiwan Payee database, create the folder at the following path:
iph_payee_tw_preprod.

Create Liquibase Master File

Create the Liquibase master file: master.xml. This file uses the changeset tag to control SQL versions and includes the following attributes:

id: A unique identifier for the changeset.
author: The name of the person or entity making the change.
logicalFilePath: The logical path to the file that uniquely identifies it within the context of Liquibase.
The sqlFile tag within the changeset specifies the path to the SQL file that needs to be executed. The path attribute is the relative path to the SQL file, and relativeToChangelogFile indicates that the path is relative to the changelog file.

Timestamp Naming for Folders and Creating Execution Scripts

Name and create folders based on the timestamp of the SQL execution, at the same level as the master file: 20240521. If the execution dates differ, multiple folders can be created.

Within these folders, create multiple SQL execution scripts: changes.sql. Ensure that these SQL scripts are idempotent (i.e., can be run multiple times without causing errors), such as table creation SQL scripts.

Important Notes
UAT (User Acceptance Testing) and PROD (Production) folders can be shared or created separately.
If folders are created separately, ensure that SQL scripts are added to both UAT/Preprod and PROD environments simultaneously to maintain consistency across environments.
For any questions or further assistance, please refer to the official documentation or contact the relevant technical support personnel.
