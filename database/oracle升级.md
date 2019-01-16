9.2.1 Upgrading Earlier Oracle Database Releases to Oracle Database 10g Release 2 (10.2.0.4)
For information about upgrading Oracle Databases from an earlier Oracle Database (Oracle8i, Oracle9i, or Oracle Database 10g), see Oracle Database Upgrade Guide.

If you are upgrading an Oracle RAC database, refer to Oracle Real Application Clusters Administration Guide as well.

9.2.2 Upgrading a Release 9.2 Database Not Using Oracle Label Security
To find if Oracle Label Security is in the installation, complete the following steps:

You can use one of the following methods check if Oracle Label Security is installed:

If the following script exists on the computer, Oracle Label Security is installed:

$ORACLE_HOME/rdbms/admin/catnools.sql
Check the inventory at the end of the installAction log file for the base version installation (9.2). If Oracle Label Security is installed, Label Security is listed in the inventory section of the log file.

Use the following commands to check if Oracle Label Security is installed with the 9.2 database:

$ sqlplus
SQL> CONNECT SYS AS SYSDBA
Enter password:password
SELECT * FROM V$OPTION WHERE PARAMETER = "ORACLE LABEL SECURITY";
If the command does not display any results, Oracle Label Security is not applied to the database and you can ignore this section. However, if the command returns some results, you must complete this section.

If you want to upgrade an Oracle9i release 9.2 preconfigured database, and you are not using Oracle Label Security, complete the following steps to avoid errors during the upgrade:

Use Oracle Universal Installer release 9.2 to install Oracle Label Security using the Custom installation type.

Run the $ORACLE_HOME/rdbms/admin/catnools.sql script with the SYSDBA privilege to remove Oracle Label Security components from the database.

9.3 Upgrading Oracle Database 10g Release 10.2.0.x to Oracle Database 10g Release 10.2.0.4
See one of the following sections for upgrading an Oracle Database 10g release 10.2.0.x to Oracle Database 10g release 10.2.0.4:

Upgrading a Release 10.2 Database using Oracle Database Upgrade Assistant

Manually Upgrading a Release 10.2 Database

9.3.1 Upgrading a Release 10.2 Database using Oracle Database Upgrade Assistant
After you install the patch set, you must perform the following steps on every database associated with the upgraded Oracle home:


Note:

If you do not run the Oracle Database Upgrade Assistant as described in this section, then the following errors are displayed: 
ORA-01092: ORACLE instance terminated.

ORA-39700: database must be opened with UPGRADE option.
 


Log in as the Oracle software owner user.

Set the values for the environment variables $ORACLE_HOME, $ORACLE_SID and $PATH.

For single-instance installations, if you are using Automatic Storage Management, start the Automatic Storage Management instance.

For Oracle single-instance installations, start the listener as follows:

$ lsnrctl start
Run Oracle Database Upgrade Assistant either in the interactive or noninteractive mode:

Interactive mode:

Enter the following command from the command prompt:

$ dbua
Complete the following steps displayed in the Oracle Database Upgrade Assistant screen:

On the Welcome screen, click Next.

On the Databases screen, select the name of the Oracle Database that you want to update, then click Next.


Note:

For Oracle RAC, enter the SYS password to do the upgrade. 


On the Recompile Invalid Objects screen, select the Recompile the invalid objects at the end of upgrade option, then click Next.

If you have not taken the back up of the database earlier, on the Backup screen, select the I would like to take this tool to backup the database option, stipulate the Path, then click Next.

On the Summary screen, check the summary, then click Finish.

On the End of Database Upgrade Assistant's Upgrade Results screen, click Close to exit from Oracle Database Upgrade Assistant.


Note:

If you are upgrading a database having dbcontrol configured in non-secure mode, after upgrade dbconsole will run in secure mode. 


If you are using the Oracle Recovery Manager catalog, enter the following command:

$ rman catalog username/password@alias
RMAN> UPGRADE CATALOG;
For Oracle RAC installations, start any database services that you want to use by entering the following command:

$ srvctl start service -d db_name -s service_name
Noninteractive mode:

Enter the following command to upgrade Oracle Database using Oracle Database Upgrade Assistant in noninteractive mode:

$ dbua -silent -dbname $ORACLE_SID -oracleHome 
$ORACLE_HOME -sysDBAUserName UserName -sysDBAPassword SYS_password 
-recompile_invalid_objects true
9.3.2 Manually Upgrading a Release 10.2 Database
Complete the following sections to upgrade an Oracle Database 10g release 10.2.0.x to Oracle Database 10g release 10.2.0.4:

Run the Pre-Upgrade Information Tool

Upgrading a Release 10.2 Database

Missing Components When Upgrading

9.3.2.1 Run the Pre-Upgrade Information Tool
If you are upgrading database manually, then you should analyze it by running the Pre-Upgrade Information Tool.

The Pre-Upgrade Information Tool is a SQL script that ships with Oracle Database 10.2. Complete the following procedure to run the Pre-Upgrade Information Tool:

Start the database in the UPGRADE mode:

SQL> STARTUP UPGRADE
Set the system to spool results to a log file for later analysis:

SQL> SPOOL upgrade_info.log 
Run the Pre-Upgrade Information Tool:

SQL> @?/rdbms/admin/utlu102i.sql
Turn off the spooling of script results to the log file:

SQL> SPOOL OFF
Check the output of the Pre-Upgrade Information Tool in the upgrade_info.log file. The following is an example of the output generated by the Pre-Upgrade Information Tool:

Oracle Database 10.2 Upgrade Information Utility    02-04-2008 11:48:11
.
**********************************************************************
Database:
**********************************************************************
--> name:       X102040
--> version:    10.2.0.1.0
--> compatible: 10.2.0.1
--> blocksize:  8192
.
**********************************************************************
Tablespaces: [make adjustments in the current environment]
**********************************************************************
--> SYSTEM tablespace is adequate for the upgrade.
.... minimum required size: 505 MB
.... AUTOEXTEND additional space required: 15 MB
--> UNDOTBS1 tablespace is adequate for the upgrade.
.... minimum required size: 401 MB
.... AUTOEXTEND additional space required: 376 MB
--> SYSAUX tablespace is adequate for the upgrade.
.... minimum required size: 265 MB
.... AUTOEXTEND additional space required: 15 MB
--> TEMP tablespace is adequate for the upgrade.
.... minimum required size: 58 MB
.... AUTOEXTEND additional space required: 38 MB
--> EXAMPLE tablespace is adequate for the upgrade.
.... minimum required size: 69 MB
.
**********************************************************************
Update Parameters: [Update Oracle Database 10.2 init.ora or spfile]
**********************************************************************
WARNING: --> "shared_pool_size" needs to be increased to at least 167772160
WARNING: --> "java_pool_size" needs to be increased to at least 67108864
.
**********************************************************************
Components: [The following database components will be upgraded orinstalled]
**********************************************************************
--> Oracle Catalog Views         [upgrade]  VALID
--> Oracle Packages and Types    [upgrade]  VALID
--> JServer JAVA Virtual Machine [upgrade]  VALID
--> Oracle XDK for Java          [upgrade]  VALID
--> Oracle Java Packages         [upgrade]  VALID
--> Oracle Text                  [upgrade]  VALID
--> Oracle XML Database          [upgrade]  VALID
--> Oracle Workspace Manager     [upgrade]  VALID
--> Oracle Data Mining           [upgrade]  VALID
--> Messaging Gateway            [upgrade]  VALID
--> OLAP Analytic Workspace      [upgrade]  VALID
--> OLAP Catalog                 [upgrade]  VALID
--> Oracle OLAP API              [upgrade]  VALID
--> Oracle interMedia            [upgrade]  VALID
--> Spatial                      [upgrade]  VALID
--> Oracle Ultra Search          [upgrade]  VALID
--> Oracle Label Security        [upgrade]  VALID
--> Expression Filter            [upgrade]  VALID
--> EM Repository                [upgrade]  VALID
--> Rule Manager                 [upgrade]  VALID
PL/SQL procedure successfully completed.
The following sections describe the output of the Pre-Upgrade Information Tool.

Database

This section displays global database information about the current database, such as the database name and release number before the database is upgraded.

Tablespaces

This section displays a list of tablespaces in the current database. For each tablespace, the tablespace name and minimum required size is displayed. In addition, a message is displayed if the tablespace is adequate for the upgrade. If the tablespace does not have enough free space, then space must be added to the tablespace in the current database. Tablespace adjustments must be made before the database is upgraded.

Update/Obsolete/Deprecated Parameters

These sections display a list of initialization parameters in the parameter file of the current database that should be adjusted before the database is upgraded. The adjustments must be made to the Oracle Database 10.2 init.ora or spfile.

Components

This section displays a list of database components that are upgraded or installed when the current database is upgraded.

9.3.2.2 Upgrading a Release 10.2 Database
After you install the patch set, you must perform the following steps on every database associated with the upgraded Oracle home:


Note:

If you do not run the catupgrd.sql script as described in this section and you start up a database for normal operation, then ORA-01092: ORACLE instance terminated. Disconnection forced errors will occur and the error ORA-39700: database must be opened with UPGRADE option will be in the alert log. 


Log in as the Oracle software owner user.

For Oracle RAC installations, start listener on each node of the cluster as follows:

$ srvctl start listener -n node
If you are using Automatic Storage Management, start the Automatic Storage Management instance.

For single-instance installations, start the listener as follows:

$ lsnrctl start
For single-instance installations, use SQL*Plus to log in to the database as the SYS user with SYSDBA privileges:

$ sqlplus /nolog 
SQL> CONNECT SYS AS SYSDBA
Enter password:SYS_password
Users of single-instance installations now proceed to step 7.

For Oracle RAC installations:

Use SQL*Plus to log in to the database as the SYS user with SYSDBA privileges:

$ sqlplus /nolog 
SQL> CONNECT SYS AS SYSDBA
Enter password: SYS_password
SQL> STARTUP NOMOUNT
Set the CLUSTER_DATABASE initialization parameter to FALSE:

SQL> ALTER SYSTEM SET CLUSTER_DATABASE=FALSE SCOPE=spfile; 
Shut down the database:

SQL> SHUTDOWN
Enter the following SQL*Plus commands:

SQL> STARTUP UPGRADE
SQL> SPOOL patch.log
SQL> @?/rdbms/admin/catupgrd.sql
SQL> SPOOL OFF
Review the patch.log file for errors and inspect the list of components that is displayed at the end of catupgrd.sql script.

This list provides the version and status of each SERVER component in the database.

If necessary, rerun the catupgrd.sql script after correcting any problems.

Restart the database:

SQL> SHUTDOWN IMMEDIATE
SQL> STARTUP
Run the utlrp.sql script to recompile all invalid PL/SQL packages now instead of when the packages are accessed for the first time. This step is optional but recommended.

SQL> @?/rdbms/admin/utlrp.sql


Note:

When the 10.2.0.4 patch set is applied to an Oracle Database 10g Standard Edition database, there may be 54 invalid objects after the utlrp.sql script runs. These objects belong to the unsupported components and do not affect the database operation. 
Ignore any messages indicating that the database contains invalid recycle bin objects similar to the following:

BIN$4lzljWIt9gfgMFeM2hVSoA==$0 



Run the following command to check the status of all the components after the upgrade:

SQL> SELECT COMP_NAME, VERSION, STATUS FROM SYS.DBA_REGISTRY;
In the output of the preceding command, the status of all the components should be VALID for a successful upgrade.

If you are using the Oracle Recovery Manager catalog, enter the following command:

$ rman catalog username/password@alias 
For Oracle RAC installations:

Set the CLUSTER_DATABASE initialization parameter to TRUE:

 SQL> ALTER SYSTEM SET CLUSTER_DATABASE=TRUE SCOPE=spfile; 
Restart the database:

SQL> SHUTDOWN IMMEDIATE
SQL> STARTUP
Start any database services that you want to use:

$ srvctl start service -d db_name -s service_name
To configure and secure Enterprise Manager follow these steps:

Ensure the database and Listener are operational.

In the case of a single instance, execute

emca -upgrade db
In the case of Oracle Real Application Clusters (RAC), execute

emca -upgrade db -cluster

Note:

If you are upgrading a database having dbcontrol configured in non-secure mode, after upgrade dbconsole will run in secure mode. 


9.3.2.3 Missing Components When Upgrading
When you upgrade Oracle Database 10g Release 1 (10.1.0.5) to Oracle Database 10g Release 2 (10.2.0.4), the diagnostics of the preupgrade utility script utlu102.sql may indicate that some database components on the 10g Companion CD should be installed. You should install these components from the Oracle Database 10g Release 1 (10.1.0.5) Companion CD before applying this patch set. If the catupgrd.sql script cannot upgrade a SERVER component because it was not installed from the Companion CD, then the status of the SERVER component in the patch.log file is reported as NO SCRIPT.


Note:

If the preupgrade script indicates the Server JAVA Virtual Machine's JAccelerator (NCOMP) or Oracle interMedia Image Accelerator should be installed, but they are not installed before applying the patch set, then the patch.log file contains the status of their parent components as successfully upgraded to Oracle Database 10g Release 2 (10.2.0.4) even though these components are still missing. 


If you find any component, which was identified as missing by the preupgrade utility script, was not installed before running the catupgrd.sql script, then install the missing component from the Companion CD and run the catupgrd.sql script again.

9.4 Running changePerm.sh Script on an Oracle Database Server Home

Important:

Oracle recommends using the most restrictive file permissions possible for the given implementation. Perform these optional steps only after considering all security ramifications and only if you need to share this installation. 


During patch set installation, all new files and directories are created with restricted access, by default. Users or third party applications with a different group identifier from that of the database, which try to access client-side utilities or libraries in the database home, will see permission errors when trying to access these files or directories. Perform the following steps to change the permissions:

Change to the install directory by using the following command:

$ cd $ORACLE_HOME/install
Run changePerm.sh and specify the patched server Oracle home location, before accessing client-side utilities or libraries in the database home.


Note:

If you are applying patch to Oracle RAC home, then you will need to run this script on all the nodes.