# README - General Notes
## Docker Oracle DB
Run the following command to start up a docker instance. When starting it do not start in detached mode as you need to see the password it generated for the system user. it will output it to command line. also of importance make sure to port forward 1521.

```
sudo docker run -p 1521:1521 --name oracle-database-12c lzrbear/oracle-database:12.2.0.1-ee
```

current password: ```KQYYslewzaY=1```
old password: ```znjHG+rMCA4=1``` 

notes

if you run ```docker logs {instance ID}``` it will show you the console output of starting the machine. This means we can run in deamon mode and use docker logs command to retrieve the password
i.e.
```
sudo docker run -d -p 1521:1521 --name oracle-database-12c lzrbear/oracle-database:12.2.0.1-ee
```


## SQL Developer

Make sure to always run sql developer as sudo otherwise the modules in /opt/sqldeveloper/external will fail to load. 

*TODO: Determine why this is, it could be a permission issue that can be easily fixed.*

For now to run sql developer use the following command:
```
sudo sqldeveloper
```

## SQL Plus

Sql Plus is the command line interface for connecting to an oracle DB

If you run it with the nolog flag you can avoid logging in.
```
sqlplus /nolog
```

To connect to a database:
```
connect {username}@{databasename}
```

Where database name is either the name of the PDB database (pluggable database) or the CDB (container database, i.e. the database that is hosting the pluggable databases)

e.g.

```
connect system@orclcdb
```

### Docker Notes

important file path to network config for db
```
cd $ORACLE_HOME/network/admin
```

to connect as root in the docker instance use the following command:
```
sudo docker exec -it --user root --workdir / fc8 /bin/bash
```


Notes:

If you set a password with special characters e.g. P@ssw0rd1 when logging in to sqlplus you need to wrap it in quotes e.g. conn user/"P@ssw0rd1"@database

the same applies for whenever a prompt is given in the sqlplus command line

### Tablespaces

The default directory for tablespace files is:
```
/opt/oracle/product/12.2.0.1/dbhome_1/dbs
```

Command to list all tablespaces in the connected database
```
select tablespace_name from user_tablespaces;
```

Command to list file location of all tablespaces in the connected database
```
select * from dba_data_files ;
````

FYI - A tablespace is a physical partition on the disk and unrelated to a schema, which is a logical partition in the database.


```
select * from user_tables order by table_name;

grant SELECT on object_type_definitions to REF;
grant select on CATALOGUES to REF;
grant select on PHY_NUMBER to REF;
grant select on FIELD_DEFINITIONS to REF;
grant select on ALL_VALIDATION_DEFINITIONS to REF;
grant SELECT on ATTRIBUTE_DEFINITIONS to REF;
grant SELECT on GROUPS to REF;

grant SELECT on object_type_definitions to PUBLIC;
grant select on CATALOGUES to PUBLIC;
grant select on PHY_NUMBER to PUBLIC;
grant select on FIELD_DEFINITIONS to PUBLIC;
grant select on ALL_VALIDATION_DEFINITIONS to PUBLIC;
grant SELECT on ATTRIBUTE_DEFINITIONS to PUBLIC;
grant SELECT on GROUPS to PUBLIC;

```

### Getting last run queries
This query will return the last run queries for the user named read.
```sql
SELECT            
 S.LAST_ACTIVE_TIME,     
 S.MODULE,
 S.SQL_FULLTEXT, 
 S.SQL_PROFILE,
 S.EXECUTIONS,
 S.LAST_LOAD_TIME,
 S.PARSING_USER_ID,
 S.SERVICE                                                                       
FROM
 SYS.V_$SQL S, 
 SYS.ALL_USERS U
WHERE
 S.PARSING_USER_ID=U.USER_ID 
 AND UPPER(U.USERNAME) IN UPPER('read')   
ORDER BY TO_DATE(S.LAST_LOAD_TIME, 'YYYY-MM-DD/HH24:MI:SS') desc;
```

### Unlocking an account
The following query will unlock the read account.
```sql
ALTER USER read IDENTIFIED BY only ACCOUNT UNLOCK;
```

# JAVA notes
## Manifest
TIP: When creating a manifest file make sure the last line is a new line, otherwise the line above will not be added to the manifest.


# Glassfish

To set up on docker
```
sudo docker pull glassfish:4.1-jdk8
sudo docker run -d -p 32770:4848 -p 32769:8080 --name glassfish glassfish:4.1-jdk8
```

To start the admin interface on the docker machine run the following

note: the default admin password is blank and the default admin username is admin

```
asadmin change-admin-password
asadmin enable-secure-admin
```

Port 8080 is used as the default port for the glassfish server

Port 4848 is used as the default port for the glassfish server admin tool
