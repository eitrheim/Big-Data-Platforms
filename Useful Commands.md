# Useful Big Data Platform Commands (Hive, Hadoop, command line)

`ssh USER@hadoop.rcc.uchicago.edu` login to RCC, then enter password  
`ls` list all directories  
`mkdir newdir` makes new directory named newdir  
`rm -rf newdir`	deletes directory named newdir  
`clear`	clears screen  
`cd newdir`		goes into newdir  
`cd`	goes to home  
`pwd`	to see which file you're in  
`rm file`	to remove 'file'  
`ssh user@host`	connect to host as user  
`ctrl+c` halt current command  
`crtl+z` stop current command  
`ctrl+d` log out of current session  
`ctrl+w` erases one word in current line  
`ctrl+u` erases whole line  
`!!` repeat last command  
`exit` log out of current session  
`wget <url>` download file directly  
`mv orignalFile name newFileName` to change file name  
`tar zxvf instacart.tar.gz` unzip file  
`hadoop fs -put /home/$USER/data/instacart /user/$USER/instacart` to put it to hadoop

# Update all Python packages with pip  
`pip list --outdated --format=freeze | grep -v '^-e' | cut -d = -f 1 | xargs -n1 pip install -U`
or
`pip freeze --local | grep -v '^\-e' | cut -d = -f 1 | xargs -n1 pip install -U`

# Access RCC with Filezilla
hadoop.rcc.uchicago.edu  
user  
password  
port: 22  

# Web access to RCC
Access Python through virtual computer on RCC  
https://hadoop.rcc.uchicago.edu/  	

To use terminal on webpage  
[midway2-login1.rcc.uchicago.edu/main/](midway2-login1.rcc.uchicago.edu/main/)

# Hadoop (run from Linux shell)

Can use $USER instead of eitrheim  

Check to see raw file contents  
`tail /home/eitrheim/data/food-inspections.csv`  

Load file into Hadoop first  
`hdfs dfs -ls /user/eitrheim/data`  

Make directory on Hadoop  
`hdfs dfs -mkdir /user/eitrheim/data`  

Delete all files from user's hadoop directory  
`hdfs dfs -rm -r /user/eitrheim/data/*`  

Load file into HDFS from local directory  
`hdfs dfs -put /home/eitrheim/data/food-inspections.csv /user/$USER/data/food-inspections.csv`  

View and verify file contents on Hadoop  
`hdfs dfs -cat /user/$USER/data/food-inspections.csv`

# Data Definition Language (DDL) in Hive Shell

`hive`  

Drop existing database  
`drop database eitrheim cascade;`  

Create a Hive database (since the Hive space on RCC is shared, we will make it easier to identify using username)  
`create database if not exists eitrheim;`  

List all database  
`show databases;`  

Switch to your database  
`use eitrheim;`  

Show all tables under your database  
`show tables;`  

Drop table if it exists already  
`drop table foodinspections;`  

Create new table (ensure you skip header row)  
`create table FOODINSPECTIONS (INSPECTION_ID string,DBA_NAME string,AKA_NAME string,LICENSE_NUM string,FACILITY_TYPE string,RISK string, ADDRESS string, CITY string, STATE string, ZIP string, INSPECTION_DATE date, INSPECTION_TYPE string, RESULTS string, VIOLATIONS string, LATITUDE string,LONGITUDE string, LOCATION string) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n'`  
`STORED AS TEXTFILE`  
`tblproperties("skip.header.line.count"="1");`  

Show all tables under your database  
`show tables;`  

Describe table   
`describe FOODINSPECTIONS;`  

Load data from HDFS into Hive  
`LOAD DATA INPATH '/user/eitrheim/data/food-inspections.csv' INTO TABLE foodinspections;`  

Get record count  
`select count (*) from FOODINSPECTIONS;`  
`select count (*) from chicago_crimes;`  

Show first few records  
`select * from foodinspections limit 10;`  

Drop the table  
`drop table FOODINSPECTIONS;`  

Check to see if table exists  
`show tables;`  

When you drop an internal table, Hive drops the data along with the metadata.Confirm this by checking HDFS  
`hdfs dfs -ls /user/$USER/data`  
Your HDFS file has been lost. Recreate that file in HDFS. Load file into HDFS from local directory  
`hdfs dfs -put /home/$USER/data/food-inspections.csv /user/$USER/data/food-inspections.csv`  

Create an external table  
`create external table FOODINSPECTIONS (INSPECTION_ID string,DBA_NAME string,AKA_NAME string,LICENSE_NUM string,FACILITY_TYPE string,RISK string, ADDRESS string, CITY string, STATE string, ZIP string, INSPECTION_DATE date, INSPECTION_TYPE string, RESULTS string, VIOLATIONS string, LATITUDE string,LONGITUDE string, LOCATION string) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n'`  
`STORED AS TEXTFILE`  
`LOCATION '/user/eitrheim/data/'  
tblproperties("skip.header.line.count"="1"); `  

Check to see if table exists  
`show tables;`  

Load data from HDFS into Hive  
`LOAD DATA INPATH '/user/apujari/data/food-inspections.csv' INTO TABLE foodinspections;`  

Get record count  
`select count (*) from FOODINSPECTIONS;`  

Drop the table  
`drop table FOODINSPECTIONS;`  

When you drop an external table, it only deletes the metadata.Confirm this by checking HDFS  
`hdfs dfs -ls /user/$USER/data`  
Your HDFS file should still exist  

Next time before loading data from HDFS into Hive use OVERWRITE to prevent data duplication.   
`LOAD DATA INPATH '/user/eitrheim/data/food-inspections.csv' OVERWRITE INTO TABLE foodinspections;`  

Creating indexes in Hive is not recommended anymore. Hive 3.0 has dropped support for indexes.
https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Indexing

# Data Manipulation Language (DML)  in Hive Shell

Count number of grocery stores   
`select count(FACILITY_TYPE) from foodinspections where FACILITY_TYPE ="Grocery Store";`  

Count number of facilities that start with the word AMAZING  
`select count(DBA_NAME) from foodinspections where DBA_NAME like "AMAZING%";`  

Count number of facilities by type  
`select FACILITY_TYPE, count(*) from foodinspections group by FACILITY_TYPE;`  
Something is wrong with the way the table has been loaded

### Use of SERDE (serializer-deserializer)

Let's drop and recreate the table this time with SERDE

Drop the table  
`drop table FOODINSPECTIONS;`  

We will provide an additional quote property  
`create external table FOODINSPECTIONS (INSPECTION_ID string,DBA_NAME string,AKA_NAME string,LICENSE_NUM string,FACILITY_TYPE string,RISK string, ADDRESS string, CITY string, STATE string, ZIP string, INSPECTION_DATE date, INSPECTION_TYPE string, RESULTS string, VIOLATIONS string, LATITUDE string,LONGITUDE string, LOCATION string) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'`  
`WITH SERDEPROPERTIES (`
`   "separatorChar" = ",",`  
`   "quoteChar"     = '\"',`  
`   "escapeChar"    = '\\')`  
`STORED AS TEXTFILE`  
`LOCATION '/user/eitrheim/data/'`  
`tblproperties("skip.header.line.count"="1");`  

Load data from HDFS into Hive  
`LOAD DATA INPATH '/user/eitrheim/data/food-inspections.csv' INTO TABLE foodinspections;`  

Count number of facilities by type  
`select FACILITY_TYPE, count(*) from foodinspections group by FACILITY_TYPE;`  
This time the data looks good. i.e. other columns don't show up as FACILITY_TYPE  

Hive SQL using date comparisons  
`select count(*) from foodinspections`  
`where unix_timestamp(INSPECTION_DATE, 'yyyy-MM-dd') >= unix_timestamp('2010-01-01', 'yyyy-MM-dd')`  
`and unix_timestamp(INSPECTION_DATE, 'yyyy-MM-dd') <= unix_timestamp('2013-08-31', 'yyyy-MM-dd')`  

Let's now create a new table using an inner query - with the results from the previous query  
`create table FACILITY_TYPES as select FACILITY_TYPE, count(*) as COUNT from foodinspections group by FACILITY_TYPE;`  

Check to see if table exists  
`show tables;`  

Check to see if table has data  
`select * from FACILITY_TYPES;`  

Let's export this small output data to a local file. Create a file export.hql with the previous query inside the file - much like a SQL file  
`hive -f 'export.hql' >> localfile.txt`  

# File Storage Formats (Parquet)

`drop table FOODINSPECTIONS_PARQUET;`  

Let's create a Parquet table (note that Parquet does not support date type)  
`create external table FOODINSPECTIONS_PARQUET (INSPECTION_ID string,DBA_NAME string,AKA_NAME string,LICENSE_NUM string,FACILITY_TYPE string,RISK string, ADDRESS string, CITY string, STATE string, ZIP string, INSPECTION_DATE string, INSPECTION_TYPE string, RESULTS string, VIOLATIONS string, LATITUDE string,LONGITUDE string, LOCATION string) 
STORED AS PARQUET`  
`LOCATION '/user/eitrheim/data/'; `  

Set compression format  
`set parquet.compression=GZIP;`  

Copy data from the Hive to the Parquet table  
`INSERT OVERWRITE TABLE FOODINSPECTIONS_PARQUET SELECT * FROM foodinspections;`  

Show first few records and notice the change in format  
`select * from FOODINSPECTIONS_PARQUET limit 10;`  

Issue SQL query on the parquet file  
`select FACILITY_TYPE, count(*) from FOODINSPECTIONS_PARQUET group by FACILITY_TYPE;`  

# File Storage Formats (ORC /with compression)

`drop table FOODINSPECTIONS_ORC;`  

`create external table FOODINSPECTIONS_ORC (INSPECTION_ID string,DBA_NAME string,AKA_NAME string,LICENSE_NUM string,FACILITY_TYPE string,RISK string, ADDRESS string, CITY string, STATE string, ZIP string, INSPECTION_DATE string, INSPECTION_TYPE string, RESULTS string, VIOLATIONS string, LATITUDE string,LONGITUDE string, LOCATION string) 
STORED AS ORC`  
`LOCATION '/user/eitrheim/data/'`  
`tblproperties ("orc.compress"="SNAPPY");`  

Copy data from the Hive to the ORC table  
`INSERT OVERWRITE TABLE FOODINSPECTIONS_ORC SELECT * FROM foodinspections;`
