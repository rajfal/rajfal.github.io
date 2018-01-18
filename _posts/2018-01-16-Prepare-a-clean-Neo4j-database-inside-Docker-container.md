---
layout: post
title: "Prepare a fresh Neo4j graph database inside a Docker container"
comments: false
description: "workaround on initializing a clean database inside a Neo4j container in Docker"
categories: [neo4j, docker, graph.db]
tags: [neo4j, docker, graph.db]
keywords: "neo4j, Docker, graph database, create database, delete database, Docker container, workaround"
---

> #### *Prerequisites:*{: style="color: red"}
> - [a working Docker Neo4j container](/2018/Docker-Neo4j-container-setup/)
> - you are working with 3.3.1 version of the Neo4j Community Edition (CE)

---

#### *Your options*{: style="color: black"}

By default, once installed Neo4j CE comes with `graph.db` database that is awaiting to be populated with nodes, labels, properties
and relationships. You will find this database at `~/neo4j/data/databases/graph.db/`. If you have not created any nodes or imported
any data then `graph.db` remains empty or clean.

If that is the case then you don't need to read the rest of this post.

However, if you either have already done some work inside the default `graph.db` database, or wish to use a different one 
for your CSV data import then you will need to make some changes to make that possible.

####Already played with Neo4j?####

If you have already run some queries then you will notice that your Neo4j Browser's Database Information tab will display
something like this:

![Graph.db already contains nodes and relationships](/assets/images/graph_nodes_present.png)

Therefore, before you can import fresh data, you will need to clear out all nodes, labels, properties and relationships. On the surface this would appear to be very simple; and it should be but it's not.

You can go to your Neo4j Browser and run the following command:
```sql
MATCH (n) DETACH DELETE n;
```
The command above is designed to remove all nodes and all the information related to it. However, that is not what happens in reality.

Yes, the system does say the follow:
```bash
Deleted 3320 nodes, deleted 12675 relationships, completed after 174 ms.
```
But Neo4j Browser's Database Information tab displays something unexpected:

![Graph.db still contains node labels and node properties](/assets/images/labels_properties_still_there.png)


- prepare a Cypher script, docker_soil_survey_import.cql, that will instruct Neo4j to create required nodes and relationships
- ensure that survey.csv and docker_soil_survey_import.cql files are moved to `~/neo4j/import` directory
- run the data import using **neo4j-shell**. This is a command line client that communicates directly with your Neo4j database
- confirm that all records are in and generate a meta-graph that should be exactly like the data model we created[[^1]]

#### What tool should I use for importing data into Neo4j?

There are a number of tools we can use to import external data into a Neo4j graph:

**Neo4j Browser** - it will run LOAD CSV statements but only one at a time

**neo4j-shell** - is a command line utility that comes pre-installed with Neo4j and will run multi-statement Cypher scripts to run against a graph database. Each statement must be terminated with a semicolon, **(;)**

**neo4j-import** - is a command line utility that comes pre-installed with Neo4j and is designed for bulk loading massive datasets that exceed 10 million records. You can also use this tool to test far smaller datasets. However, note that your CSV files must follow a very specific format[[^1]]

**LazyWebCypher** - an online web app that will run multi-statement Cypher scripts even against your own local Neo4j instance[[^2]]

**cycli** - Cypher command-line interface[[^3]]


#### How to import data into Neo4j using neo4j-shell

1. Cypher is the language for querying and manipulating Neo4j graph databases[[^4]] 

```sql
// import Hort_Client nodes
CREATE INDEX ON :Hort_Client(client);
CREATE INDEX ON :Hort_Client(name);

USING PERIODIC COMMIT 1000
LOAD CSV WITH HEADERS FROM "file:///soil_survey.csv" AS line
WITH line LIMIT 10000
MERGE (hc:Hort_Client {client: line.Hort_Client, name: 'hc_' + line.Hort_Client});
```
Also:
  : - importing `Hort_Client` nodes  
  : - `CREATE INDEX` - adds an index for each property of the node. Note that we have two, `client` and `name`
  : -`USING PERIODIC COMMIT 1000` - every 1000 records/lines are treated as a single transaction after which those records will be written to disk
  : -`LOAD CSV WITH HEADERS FROM` - 'soil\_survey.csv' file has headers that we added manually using `sed`
  : -`LIMIT 10000` - a maximum number of lines that you wish to import. Even if there are more lines in the file, the import will stop at the limit. This is also good for testing, if you don't want to load that 200M+ record file just yet
  : -`MERGE` - since there are many instances of the `Hort_Client` inside the import file, we only want to create a single unique node

```sql
// import Soil_Service nodes
CREATE INDEX ON :Soil_Service(ss_id);
CREATE INDEX ON :Soil_Service(name);

USING PERIODIC COMMIT 1000
LOAD CSV WITH HEADERS FROM "file:///soil_survey.csv" AS line
WITH line LIMIT 10000
MERGE (ss:Soil_Service {ss_id: line.Soil_Service, name: '_' + line.Soil_Service});
```
Also:
  : - as above but import `Soil_Service` nodes

```sql
// Hort_Client-->Soil_Service

USING PERIODIC COMMIT 1000
LOAD CSV WITH HEADERS FROM "file:///soil_survey.csv" AS line
WITH line LIMIT 10000
MATCH (hc:Hort_Client {client: line.Hort_Client})
MATCH (ss:Soil_Service {ss_id: line.Soil_Service})
MERGE (hc)-[:REQUESTS]->(ss);
```
Also:  
  : - `MATCH` - locate `Hort_Client` and `Soil_Service` nodes whose properties match the values of the current file line being read in  
  : - `MERGE (hc)-[:REQUESTS]->(ss)` - once the two above nodes are found, create a relationship between them, labelled `REQUESTS`
  : - import/ stores files, such as Json or CSV, that you can import to graph. I also put my Cypher scripts in here. More on that later
  : - conf/ stores a customised neo4j.conf file. More later about modifying specific settings
  
The resulting Cypher file will be a series of statements that will index node properties, create nodes and build relationships between them.  

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

