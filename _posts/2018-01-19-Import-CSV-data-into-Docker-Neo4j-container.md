---
layout: post
title: "Import CSV data into Neo4j graph database inside a Docker container"
comments: false
description: "step by step guide on pushing data into a Neo4j container in Docker"
categories: [neo4j, docker, CSV import]
tags: [neo4j, docker, data import]
keywords: "neo4j, Docker, csv, graph database, CSV, data import, Docker container, denormalized"
---

> #### *Prerequisites:*{: style="color: red"}
> - [a working Docker Neo4j container](/2018/Docker-Neo4j-container-setup/)
> - [a CSV file of denormalized data sourced from a relational database](/2018/Extract-CSV-data-from-MySQL/) 
> - [a graph database model derived from your relational schema data](/2018/Convert-relational-schema-to-graph-database-model/) 
> - [a prepared Cypher script that will push CSV data into Neo4j](/2018/Use-Cypher-for-data-modeling-and-CSV-analysis/)
> - [a clean graph.db database](/2018/Create-a-clean-Neo4j-database-inside-Docker-container/)

---

#### *What's next?*{: style="color: black"}
- prepare a Cypher script, docker_soil_survey_import.cql, that will instruct Neo4j to create required nodes and relationships
- ensure that survey.csv and docker_soil_survey_import.cql files are moved to `~/neo4j/import` directory
- run the data import using **neo4j-shell**. This is a command line client that communicates directly with your Neo4j database
- confirm that all records are in and generate a meta-graph that should be exactly like the data model we created[[^1]]

#### What tool should I use for importing data into Neo4j?

There are a number of tools that we can use to import external data into a Neo4j graph:

**Neo4j Browser** - it will run LOAD CSV statements but only one at a time

**neo4j-shell** - is a command line utility that comes pre-installed with Neo4j and will run multi-statement Cypher scripts to run against a graph database. Each statement must be terminated with a semicolon**(;)**

**neo4j-import** - is a command line utility that comes pre-installed with Neo4j and is designed for bulk loading massive datasets that exceed 10 million records. You can also use this tool to test far smaller datasets. However, note that your CSV files must follow a very specific format[[^1]]

**LazyWebCypher** - an online web app that will run multi-statement Cypher scripts even against your own local Neo4j instance[[^2]]

**cycli** - Cypher command-line interface[[^3]]



#### How to import data into Neo4j using neo4j-shell

1. Confirm that the CSV file, `soil_survey_sample.csv`, is in place
```bash
sudo head -3 neo4j/import/soil_survey_sample.csv 
Hort_Client,Contractor,Region,Locality,Soil_Service,Solution,Soil_Issue,Date_Reported,Date_Actioned,DaysToAction
159,1091,Northbury,3656,54593,5397,Erosion,2007-05-07,2008-02-18,287
159,1091,Northbury,1516,22644,5397,Erosion,2007-05-07,2008-03-18,316
```
Also:
  : - importing ????  

2. Confirm that the Cypher file, `soil_survey_import_to_neo4j_in_docker.cql`,  is in place
```bash
sudo head -8 neo4j/import/soil_survey_import_to_neo4j_in_docker.cql 
// '----start---of---import---'

// This file is to be used for Docker installation of Neo4j


// import Hort_Client nodes
CREATE INDEX ON :Hort_Client(client);
CREATE INDEX ON :Hort_Client(name);
```

XX

#### Running preliminary data exploration on **soil_survey.csv** with your [Neo4j Browser](http://localhost:7474/)

**NB:** *Ensure you've parked your CSV file in `~/neo4j/import` and Neo4j service is running*{: style="color: red"}

1. Count total number of lines. 
```sql
LOAD CSV WITH HEADERS FROM "file:///soil_survey_sample.csv" AS line
WITH line
RETURN count(line);
```
```bash
╒════════════╕
│"line_count"│
╞════════════╡
│2837        │
└────────────┘
```

2. Get fields and values from a typical line
```sql
LOAD CSV WITH HEADERS FROM "file:///soil_survey_sample.csv" AS line 
WITH line LIMIT 1 RETURN line as fields_and_values;
```
```bash
{
  "Solution": "5397",
  "Soil_Service": "54593",
  "Region": "Northbury",
  "Contractor": "1091",
  "Soil_Issue": "Erosion",
  "Date_Reported": "2007-05-07",
  "DaysToAction": "287",
  "Date_Actioned": "2008-02-18",
  "Locality": "3656",
  "Hort_Client": "159"
}
```
3. Get fields and values from a typical line, nicely formatted
```sql
LOAD CSV WITH HEADERS FROM "file:///soil_survey_sample.csv" AS line
WITH line LIMIT 1
RETURN line.Hort_Client as Hort_Client, line.Soil_Service as Soil_Service, line.Soil_Issue as Soil_Issue, line.Solution as Solution, line.Date_Reported as Date_Reported, line.Date_Actioned as Date_Actioned, line.DaysToAction as DaysToAction, line.Contractor as Contractor, line.Locality as Locality, line.Region as Region;
```
```bash
╒═════════════╤══════════════╤════════════╤══════════╤═══════════════╤═══════════════╤══════════════╤════════════╤══════════╤═══════════╕
│"Hort_Client"│"Soil_Service"│"Soil_Issue"│"Solution"│"Date_Reported"│"Date_Actioned"│"DaysToAction"│"Contractor"│"Locality"│"Region"   │
╞═════════════╪══════════════╪════════════╪══════════╪═══════════════╪═══════════════╪══════════════╪════════════╪══════════╪═══════════╡
│"159"        │"54593"       │"Erosion"   │"5397"    │"2007-05-07"   │"2008-02-18"   │"287"         │"1091"      │"3656"    │"Northbury"│
└─────────────┴──────────────┴────────────┴──────────┴───────────────┴───────────────┴──────────────┴────────────┴──────────┴───────────┘
```
  
4. Get a count of unique values found in fields of interest
```sql
LOAD CSV WITH HEADERS FROM "file:///soil_survey_sample.csv" AS line
WITH line
WITH  line.Hort_Client as client, line.Soil_Service as service, line.Soil_Issue as issue, line.Solution as solution, line.Locality as locality, line.Region as region
RETURN count(DISTINCT client) as Hort_Client, count(DISTINCT service) as Soil_Service, count(DISTINCT issue) as Soil_Issue, count(DISTINCT solution) as Solution, count(DISTINCT locality) as Locality, count(DISTINCT region) as Region;
```
```bash
╒═════════════╤══════════════╤════════════╤══════════╤══════════╤════════╕
│"Hort_Client"│"Soil_Service"│"Soil_Issue"│"Solution"│"Locality"│"Region"│
╞═════════════╪══════════════╪════════════╪══════════╪══════════╪════════╡
│21           │2670          │12          │255       │1925      │4       │
└─────────────┴──────────────┴────────────┴──────────┴──────────┴────────┘
```

5. Derive statistics from a numerical field of interest, e.g. `DaysToAction`
```sql
LOAD CSV WITH HEADERS FROM "file:///soil_survey_sample.csv" AS line
WITH line
WITH  toInt(line.DaysToAction) as action_delay
RETURN min(action_delay) as min_delay, max(action_delay) as max_delay, avg(action_delay) as mean_delay, stDev(action_delay) as std_dev,
percentileDisc(action_delay, 0.25) as _25percentile, percentileDisc(action_delay, 0.5) as _50percentile, percentileDisc(action_delay, 0.75) as _75percentile, percentileDisc(action_delay, 0.9) as _90percentile;
```
```bash
╒═══════════╤═══════════╤══════════════════╤═════════════════╤═══════════════╤═══════════════╤═══════════════╤═══════════════╕
│"min_delay"│"max_delay"│"mean_delay"      │"std_dev"        │"_25percentile"│"_50percentile"│"_75percentile"│"_90percentile"│
╞═══════════╪═══════════╪══════════════════╪═════════════════╪═══════════════╪═══════════════╪═══════════════╪═══════════════╡
│-140       │365        │103.14698625308418│77.27355038508219│49             │84             │133            │216            │
└───────────┴───────────┴──────────────────┴─────────────────┴───────────────┴───────────────┴───────────────┴───────────────┘
```
Also:  
  : - Take note that the calculated field `min_delay` indicates an error in data, where `Date_Actioned` occurs before `Date_Reported`. How about inspecting the data and fixing this error before importing it into the graph?
  
6. View a random sample of five records from the file about to be imported
```sql
LOAD CSV WITH HEADERS FROM "file:///soil_survey_sample.csv" AS line
WITH line SKIP toInteger(100*rand())+ 1 LIMIT 5
RETURN line.Hort_Client as Hort_Client, line.Soil_Service as Soil_Service, line.Soil_Issue as Soil_Issue, line.Solution as Solution, line.Date_Reported as Date_Reported, line.Date_Actioned as Date_Actioned, line.DaysToAction as DaysToAction, line.Contractor as Contractor, line.Locality as Locality, line.Region as Region;
```
```bash
╒═════════════╤══════════════╤════════════════╤══════════╤═══════════════╤═══════════════╤══════════════╤════════════╤══════════╤═══════════╕
│"Hort_Client"│"Soil_Service"│"Soil_Issue"    │"Solution"│"Date_Reported"│"Date_Actioned"│"DaysToAction"│"Contractor"│"Locality"│"Region"   │
╞═════════════╪══════════════╪════════════════╪══════════╪═══════════════╪═══════════════╪══════════════╪════════════╪══════════╪═══════════╡
│"170"        │"3796"        │"HighAlkalinity"│"766"     │"2008-02-18"   │"2008-07-08"   │"141"         │"2295"      │"2616"    │"Eastling" │
├─────────────┼──────────────┼────────────────┼──────────┼───────────────┼───────────────┼──────────────┼────────────┼──────────┼───────────┤
│"170"        │"2135"        │"Erosion"       │"2104"    │"2008-02-18"   │"2008-11-18"   │"274"         │"2295"      │"1471"    │"Eastling" │
├─────────────┼──────────────┼────────────────┼──────────┼───────────────┼───────────────┼──────────────┼────────────┼──────────┼───────────┤
│"159"        │"52067"       │"HighAlkalinity"│"765"     │"2008-02-25"   │"2008-05-20"   │"85"          │"1091"      │"3487"    │"Northbury"│
├─────────────┼──────────────┼────────────────┼──────────┼───────────────┼───────────────┼──────────────┼────────────┼──────────┼───────────┤
│"159"        │"6116"        │"HighAlkalinity"│"765"     │"2008-02-25"   │"2008-06-24"   │"120"         │"1091"      │"409"     │"Northbury"│
├─────────────┼──────────────┼────────────────┼──────────┼───────────────┼───────────────┼──────────────┼────────────┼──────────┼───────────┤
│"160"        │"6241"        │"Erosion"       │"8775"    │"2008-02-25"   │"2008-04-15"   │"50"          │"1250"      │"2777"    │"Eastling" │
└─────────────┴──────────────┴────────────────┴──────────┴───────────────┴───────────────┴──────────────┴────────────┴──────────┴───────────┘
```


---
***You have a background understanding of Cypher and how its statements related to the graph model, plus you have been able to get a preliminary peak at the structure of the CSV file you are about to import***{: style="color: green"}

---
[Back to top of page](#)

---
[^1]: 1: [Using neo4j-import tool](https://neo4j.com/docs/operations-manual/current/tutorial/import-tool/)
[^2]: 2: [LazyWebCypher](http://www.lyonwj.com/LazyWebCypher/)
[^3]: 3: [Nicole White's Cycli](https://github.com/nicolewhite/cycli)
[^4]: 4: [Neo4j Cypher Commands Refcard](https://neo4j.com/docs/cypher-refcard/current/)

