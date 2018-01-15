---
layout: post
title: "Setup a Neo4j container inside Docker"
comments: false
description: "step by step guide on spinnning up a Neo4j database container within Docker environment"
categories: Neo4j Docker 
keywords: "neo4j, Docker, docker container, Neo4j graph"
---

> #### *Prerequisites:*{: style="color: red"}
> - ensure you have a working Docker environment on GNU/Linux platform
> ```bash
docker --version
Docker version 1.13.1, build 092cba3
```
---

While Docker runs on your local machine, you want to apply configuration settings that will enable containerized Neo4j inside Docker to interact with files and persist data on your local file system.



The idea here is to export records from a relational database, such as MySQL, into a CSV (comma separated values) file format that will then be used to import into a Neo4j graph.

An example of a denormalized dataset sourced from several tables by combining selected fields

```sql
mysql> select * from soil_survey order by rand() limit 3;
+-------------+------------+-----------+----------+--------------+----------+---------------+---------------+---------------+--------------+
| Hort_Client | Contractor | Region    | Locality | Soil_Service | Solution | Soil_Issue    | Date_Reported | Date_Actioned | DaysToAction |
+-------------+------------+-----------+----------+--------------+----------+---------------+---------------+---------------+--------------+
|         168 |       2245 | Swifford  | 2130     |        51277 |     2118 | Compaction    | 2010-12-27    | 2011-03-14    |           77 |
|         164 |       2503 | Northbury | 502      |          545 |     7866 | Acidification | 2010-06-28    | 2010-12-06    |          161 |
|         157 |        777 | Swifford  | 22       |           67 |     5739 | Erosion       | 2013-12-23    | 2014-04-14    |          112 |
+-------------+------------+-----------+----------+--------------+----------+---------------+---------------+---------------+--------------+
3 rows in set (0.01 sec)

```

#### How to generate a CSV data file:

1. Dump selected table to a CSV text file
```sql
SELECT * FROM soil_survey INTO OUTFILE '/var/lib/mysql-files/soil_survey.csv' FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';
```
Also:  
  : - ensure fields are separated with a comma ','  
  : - database will write the file to a location requiring a root access, such as sudo, in order to move it to another location, such as your data-import-directory/  
  : - resulting CSV will NOT have any headers included, these will be added next

2. Move CSV file to data-import-directory/
```bash
sudo mv /var/lib/mysql-files/soil_survey.csv data-import-directory/
```

3. Insert 1 file headers line at the top, and save the CSV file
```bash
sed -i '1i Hort_Client,Contractor,Region,Locality,Soil_Service,Solution,Soil_Issue,Date_Reported,Date_Actioned,DaysToAction' data-import-directory/soil_survey.csv
```

4. If you make a mistake, delete the added line 
```bash
sed -i '1d' import-directory/soil_survey.csv
```

5. Preview first 3 lines
```bash
head -3  import-directory/soil_survey.csv
Hort_Client,Contractor,Region,Locality,Soil_Service,Solution,Soil_Issue,Date_Reported,Date_Actioned,DaysToAction
159,1091,Northbury,3656,54593,5397,Erosion,2007-05-07,2008-02-18,287
159,1091,Northbury,1516,22644,5397,Erosion,2007-05-07,2008-03-18,316
```

---
***You now have a workable CSV data file that you can import into a Neo4j graph***{: style="color: green"}

---
[Back to top of page](#)
