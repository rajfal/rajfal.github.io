---
layout: post
title: "Getting denormalized data in a CSV file format from MySQL"
comments: false
description: "step by step guide on extracting denormalized data in CSV format from MySQL"
categories: data MySQL CSV SQL
keywords: "neo4j, MySQL, csv, , CSV, data export, relational database, denormalized"
---

## Goal: to prepare a denormalized CSV data file

#### *Prerequisites:*{: style="color: red"}

> - a SQL compliant relational database, such as MySQL
> - an existing database schema where you can or already have combined data from tables to obtain a dataset of denormalized data

The objective of this step is to extract a dataset from a relational database, such as MySQL, into a CSV (comma separated values) format which will be used to import into Neo4j graph.

Here is an example of an already denormalized MySQL table, which I obtained from several source tables

```sql
mysql> select * from soil_survey order by rand() limit 3;
+-------------+------------+-----------+----------+--------------+----------+---------------+---------------+---------------+--------------+
| Hort_Client | Contractor | Region    | Locality | Soil_Service | Solution | Soil_Issue    | Date_Reported | Date_Actioned | DaysToAction |
+-------------+------------+-----------+----------+--------------+----------+---------------+---------------+---------------+--------------+
|         168 |       2245 | Swifford  | 2130     |        51277 |     2118 | Compaction    | 2010-12-27    | 2011-03-14    |           77 |
|         164 |       2503 | Northbury | 502      |          545 |     7866 | Acidification | 2010-06-28    | 2010-12-06    |          161 |
|         157 |        777 | Swifford  | 22       |           67 |     5739 | Erosion       | 2013-12-23    | 2014-04-14    |          112 |
+-------------+------------+-----------+----------+--------------+----------+---------------+---------------+---------------+--------------+
5 rows in set (0.02 sec)

```

#### Instructions

1. quickly dump selected table to a CSV text file

```sql
SELECT * FROM soil_survey INTO OUTFILE '/var/lib/mysql-files/soil_survey.csv' FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';
```
  notes:
  
  1. ensure fields are separated with a comma ','
  
  2. database will write the file to a location that you will require root access, such as sudo, in order to move to a more suitable location for further processing
  
  3. resulting CSV will NOT have any headers included. We will fix this shortly

2. as a Docker container

: prepare MySQL data exports/

:  use bash to format and inspect files and add file header row

**_COMING SOON TO A BROWSER NEAR YOU_**
