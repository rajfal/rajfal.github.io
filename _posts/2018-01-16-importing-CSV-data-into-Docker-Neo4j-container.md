---
layout: post
title: "Import CSV data into Neo4j graph database inside a Docker container"
comments: false
description: "step by step guide on pushing data into a Neo4j container in Docker"
categories: neo4j docker CSV import
tags: [neo4j, docker, data import]
keywords: "neo4j, Docker, csv, graph database, CSV, data import, Docker container, denormalized"
---

> #### *Prerequisites:*{: style="color: red"}
> - [a working Docker Neo4j container](/2018/Docker-Neo4j-container-setup/)
> - [a CSV file of denormalized data sourced from a relational database](/2018/Extract-CSV-data-from-MySQL/) 
> - [a graph database model derived from your relational schema data](/2018/Convert-relational-schema-to-graph-database-model/) 

---

#### *What's next?*{: style="color: black"}
- prepare a Cypher script, docker_soil_survey_import.cql, that will instruct Neo4j to create required nodes and relationships
- ensure that survey.csv and docker_soil_survey_import.cql files are moved to `~/neo4j/import` directory
- run the data import using **neo4j-shell**. This is a command line client that communicates directly with your Neo4j database
- confirm that all records are in and generate a meta-graph that should be exactly like the data model we created[[^1]]

There are a number of tools that we can import external data into a Neo4j graph:
- _Neo4j Browser_ which can use LOAD CSV statements, **one at a time**
- neo4j-shell utility that will accept Cypher scripts to run against an graph database
- neo4j-import utility that will accept specifically formatted CSV files to import records in excess of 10 million records
- an online web app called, LazyWebCypher, where you can run multi-statement Cypher scripts even against your own local Neo4j instance[[^1]]
- cycli or Cypher command-line interface[[^2]]

In this exercise, we will use **neo4j-shell** which comes with the Neo4j installation.


#### How to configure a Neo4j Docker container:

1. Prepare the file system
```sql
// import Hort_Client nodes
CREATE INDEX ON :Hort_Client(client);
CREATE INDEX ON :Hort_Client(name);

USING PERIODIC COMMIT 1000
LOAD CSV WITH HEADERS FROM "file:///soil_survey.csv" AS line
WITH line LIMIT 10000
MERGE (hc:Hort_Client {client: line.Hort_Client, name: 'hc_' + line.Hort_Client});
```
```coffee
// import Hort_Client nodes
CREATE INDEX ON :Hort_Client(client);
CREATE INDEX ON :Hort_Client(name);

USING PERIODIC COMMIT 1000
LOAD CSV WITH HEADERS FROM "file:///soil_survey.csv" AS line
WITH line LIMIT 10000
MERGE (hc:Hort_Client {client: line.Hort_Client, name: 'hc_' + line.Hort_Client});
```
Also:  
  : - data/ stores Neo4j graph databases. The default is graph.db, but you can make others, too  
  : - logs/ stores database activity logs. You can specify  often you want these to rotate
  : - import/ stores files, such as Json or CSV, that you can import to graph. I also put my Cypher scripts in here. More on that later
  : - conf/ stores a customised neo4j.conf file. More later about modifying specific settings

2. Start Neo4j in a Docker container
```bash
sudo docker run --rm --publish=7474:7474 --publish=7687:7687 --volume=$HOME/neo4j/data:/data --volume=$HOME/neo4j/logs:/logs --volume=$HOME/neo4j/import:/var/lib/neo4j/import --volume=$HOME/neo4j/conf:/var/lib/neo4j/conf neo4j:3.3
```
```bash
Active database: graph.db
Directories in use:
  home:         /var/lib/neo4j
  config:       /var/lib/neo4j/conf
  logs:         /logs
  plugins:      /var/lib/neo4j/plugins
  import:       /var/lib/neo4j/import
  data:         /var/lib/neo4j/data
  certificates: /var/lib/neo4j/certificates
  run:          /var/lib/neo4j/run
Starting Neo4j.
2018-01-15 06:12:35.597+0000 INFO  ======== Neo4j 3.3.1 ========
2018-01-15 06:12:35.620+0000 INFO  Starting...
2018-01-15 06:12:36.802+0000 INFO  Bolt enabled on 0.0.0.0:7687.
2018-01-15 06:12:40.324+0000 INFO  Started.
2018-01-15 06:12:41.041+0000 INFO  Remote interface available at http://localhost:7474/
```
Also:  
  : - wherever you see a --volume parameter, Docker is instructed to connect the directory on your local file system with its equivalent inside the Neo4j Docker container
  : - note that connecting /conf directories must be linked, if you want the container utilize specific settings inside a customized neo4j.conf configuration file. This we will do shortly
  : - parameter neo4j:x.x.x refers to the version of Neo4j image you wish to run. If that version is not yet available in your local Docker repository, Docker will download it from the Docker Neo4j Repository[[^1]] 
  : - publishing of the two ports, 7474 and 7687 will allow you to interact with the graph data via your [Neo4j Browser](http://localhost:7474)

3. Find name of the currently running Neo4j container
```bash
sudo docker ps
```
```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                      NAMES
5667d59932ce        neo4j:3.3           "/docker-entrypoin..."   9 minutes ago       Up 9 minutes        0.0.0.0:7474->7474/tcp, 7473/tcp, 0.0.0.0:7687->7687/tcp   nifty_yalow
```
Also:
  : - in this instance you are wanting nifty_yalow as the name of this container. This reference will be used in other Docker container operations
  : - OR a faster solution, if you only have a single container running inside Docker[[^2]]
  ```bash
  echo $(sudo docker ps) | awk '{print $NF}'
  nifty_yalow
  ```
  
4. Copy and open a new neo4j.conf configuration file
```bash
sudo docker exec --interactive nifty_yalow cat /var/lib/neo4j/conf/neo4j.conf > neo4j/import/docker_neo4j.conf
sudo nano neo4j/import/docker_neo4j.conf
```

5. Ensure the following dbms.\* references are not commented out any more and save the file
```bash
# Enable a remote shell server which Neo4j Shell clients can log in to.
dbms.shell.enabled=true
# The network interface IP the shell will listen on (use 0.0.0.0 for all interfaces).
dbms.shell.host=0.0.0.0
# The port the shell will listen on, default is 1337.
dbms.shell.port=1337
```
6. Stop Docker from the active terminal with Ctrl+C
```bash
^C2018-01-15 06:42:47.555+0000 INFO  Neo4j Server shutdown initiated by request
2018-01-15 06:42:47.581+0000 INFO  Stopping...
2018-01-15 06:42:47.820+0000 INFO  Stopped.
```
7. Upload the new neo4j.conf file, confirm contents of `~/neo4j/conf/` directory, and restart the Neo4j Docker container 
```bash
cp neo4j/import/docker_neo4j.conf neo4j/conf/neo4j.conf
ls neo4j/conf/
neo4j.conf
```
```bash
sudo docker run --rm --publish=7474:7474 --publish=7687:7687 --volume=$HOME/neo4j/data:/data --volume=$HOME/neo4j/logs:/logs --volume=$HOME/neo4j/import:/var/lib/neo4j/import --volume=$HOME/neo4j/conf:/var/lib/neo4j/conf neo4j:3.3
```

---
***You now have a workable Neo4j Docker container with mapped file system directories and customized neo4j.conf configuration file***{: style="color: green"}

---
[Back to top of page](#)

---
[^1]: 1: [LazyWebCypher](http://www.lyonwj.com/LazyWebCypher/)
[^2]: 2: [Nicole White's Cycli](https://github.com/nicolewhite/cycli)

