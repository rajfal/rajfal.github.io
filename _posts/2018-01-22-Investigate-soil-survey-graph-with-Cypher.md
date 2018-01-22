---
layout: post
title: "Investigate soil survey graph with Cypher"
comments: false
description: "Cypher queries to explore data in Neo4j"
categories: [neo4j, Cypher, data analysis]
tags: [neo4j, cypher, data import]
keywords: "neo4j, Cypher, data exploration, cypher query, cypher statements, data analysis"
---

> #### *Prerequisites:*{: style="color: red"}
> - [sample soil survey data loaded into a Neo4j graph](/2018/Import-CSV-data-into-Docker-Neo4j-container/)---

#### What can I use to import data into Neo4j?

There are a number of tools that we can use to import external data into a Neo4j graph:

**Neo4j Browser** - it will run LOAD CSV statements but only one at a time

**neo4j-shell** - is a command line utility that comes pre-installed with Neo4j and will run multi-statement Cypher scripts to run against a graph database. Each statement must be terminated with a semicolon**(;)** and separated with an empty line

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

2. Confirm that the Cypher file, `soil_survey_import_to_neo4j_in_docker.cql`,  is in place
```bash
sudo head -2 neo4j/import/soil_survey_import_to_neo4j_in_docker.cql 
CREATE INDEX ON :Hort_Client(client);
CREATE INDEX ON :Hort_Client(name);
```

3. In a terminal different to the one that Neo4j service is running in, enter the following command 
```bash
sudo docker exec -ti $(sudo docker ps --format '{% raw %}{{.Names}}{% endraw %}') bin/neo4j-shell -file import/soil_survey_import_to_neo4j_in_docker.cql
```
Sample output:  
  : - ```bash
+-------------------+
| No data returned. |
+-------------------+
Indexes added: 1
66 ms
+-------------------+
| No data returned. |
+-------------------+
Indexes added: 1
11 ms
+-------------------+
| No data returned. |
+-------------------+
Nodes created: 21
Properties set: 42
Labels added: 21
723 ms
+-------------------+
| No data returned. |
+-------------------+
Indexes added: 1
19 ms
+-------------------+
| No data returned. |
+-------------------+
Indexes added: 1
15 ms
+-------------------+
| No data returned. |
+-------------------+
Nodes created: 2670
Properties set: 5340
Labels added: 2670
980 ms
+-------------------+
| No data returned. |
+-------------------+
Relationships created: 2796
5457 ms
  ..
  ..
  ...
  ```
    
4. Get a list of node labels
```bash
sudo docker exec -ti $(sudo docker ps --format '{% raw %}{{.Names}}{% endraw %}') bin/neo4j-shell -c "CALL db.labels();"
```
Sample output:  
  : - ```bash
+----------------+
| label          |
+----------------+
| "Hort_Client"  |
| "Soil_Service" |
| "Solution"     |
| "Soil_Issue"   |
| "Soil_Report"  |
| "Contractor"   |
| "Region"       |
| "Locality"     |
+----------------+
8 rows
15 ms
```
5. Get a list of relationships
```bash
sudo docker exec -ti $(sudo docker ps --format '{% raw %}{{.Names}}{% endraw %}') bin/neo4j-shell -c "CALL db.relationshipTypes();"
```
Sample output:  
  : - ```bash
+------------------+
| relationshipType |
+------------------+
| "REQUESTS"       |
| "RECOMMENDS"     |
| "HAS"            |
| "INVESTIGATES"   |
| "CORRECTS"       |
| "DISCUSSES"      |
| "ISSUES"         |
| "SENT&#95;TO"        |
| "ACTIONS"        |
| "WORKS&#95;AT"       |
| "OPERATES&#95;IN"    |
| "PART&#95;OF"       |
+------------------+
12 rows
10 ms
```
6. Generate a meta-graph in Neo4j Browser
```sql
CALL db.schema();
```
Soil Survey meta-graph:  
  : - ![Soil Survey meta-graph](/assets/images/soil_survey_meta_graph.png)
  
  
---
***You have successfully populated Neo4j database using neo4j-shell utility and confirmed existence of new nodes and relationships***{: style="color: green"}

---
[Back to top of page](#)

---
[^1]: 1: [Using neo4j-import tool](https://neo4j.com/docs/operations-manual/current/tutorial/import-tool/)
[^2]: 2: [LazyWebCypher](http://www.lyonwj.com/LazyWebCypher/)
[^3]: 3: [Nicole White's Cycli](https://github.com/nicolewhite/cycli)


