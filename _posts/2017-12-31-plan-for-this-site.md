---
layout: post
title: "Site's layout and content"
comments: false
description: "Plan for running data into Neo4j..."
categories: neo4j docker paperwork
keywords: "neo4j, docker, csv"
---

### Objective 1: import a denormalized CSV data into Neo4j

#### Run Neo4j in two separate environments 

1. as a local laptop install
2. as a Docker container

* prepare MySQL data exports
- use bash to format and inspect files and add file header row
+ create a Cypher import file, \*.cql
- use neo4-shell utility to import data
- mention gotchas, such as Neo4j Browser after removing graph data but leaving node labels and key properties still visible

#### Footnote

There will be some URL's references that I will place as footnotes at the bottom of the page...I think! 

This is an example for the footnote number one [[^1]]. You can even add more footnotes, with link! [[^2]]

Also, another footnote, or a pointer to section of text, [[^3]], see section C. 

<div class="divider">Code block examples</div>

```sql
mysql> select * from soil_survey_sample order by rand() limit 3;
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
| Hort_Client | Contractor | Region   | Locality | Soil_Service | Solution | Soil_Issue | Date_Reported | Date_Actioned | DaysToAction |
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
|         160 |       1250 | Eastling | 1715     |         3855 |     2786 | Erosion    | 2009-08-10    | 2009-09-28    |           49 |
|         175 |        415 | Swifford | 552      |         2362 |     6684 | Erosion    | 2008-04-08    | 2008-04-08    |            0 |
|         160 |       1250 | Eastling | 937      |         2106 |    10773 | Erosion    | 2011-10-24    | 2012-02-13    |          112 |
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
3 rows in set (0.00 sec)

```

```sql
SELECT * FROM soil_survey_sample INTO OUTFILE '/var/lib/mysql-files/soil_survey_sample.csv' FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';
```

```bash
ithilien@moon-shadow:~$ sudo mv /var/lib/mysql-files/soil_survey_sample.csv neo4j/import/
ithilien@moon-shadow:~$ sed -i '1i Hort_Client,Contractor,Region,Locality,Soil_Service,Solution,Soil_Issue,Date_Reported,Date_Actioned,DaysToAction' neo4j/import/soil_survey_sample.csv 
ithilien@moon-shadow:~$ head -3 neo4j/import/soil_survey_sample.csv 
Hort_Client,Contractor,Region,Locality,Soil_Service,Solution,Soil_Issue,Date_Reported,Date_Actioned,DaysToAction
159,1091,Northbury,3656,54593,5397,Erosion,2007-05-07,2008-02-18,287
159,1091,Northbury,1516,22644,5397,Erosion,2007-05-07,2008-03-18,316
```

#### Table 1: With Alignment

| Location Reference        | Local install          | Docker container  |
| ------------------------- |:----------------------:| -----------------:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
| Hort_Client | Contractor | Region   | Locality | Soil_Service | Solution | Soil_Issue | Date_Reported | Date_Actioned | DaysToAction |
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
|         160 |       1250 | Eastling | 1715     |         3855 |     2786 | Erosion    | 2009-08-10    | 2009-09-28    |           49 |
|         175 |        415 | Swifford | 552      |         2362 |     6684 | Erosion    | 2008-04-08    | 2008-04-08    |            0 |
|         160 |       1250 | Eastling | 937      |         2106 |    10773 | Erosion    | 2011-10-24    | 2012-02-13    |          112 |
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
3 rows in set (0.00 sec)

---
Footnote:

[^1]: 1: Footnote number one yeah baby!

[^2]: 2: A footnote you can link to - [click here!](#)

[^3]: 3: A section you can focus on - [return to topic menu!](#)
