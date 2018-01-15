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

Unless all you need is a throw away container session, then the Neo4j Docker image will require access to selected parts of your filesystem. The changes you will make are specifically located in `~/neo4j/` directory.

#### How to configure a Neo4j Docker container:

1. Prepare the file system
```bash
cd ~
mkdir neo4j
cd neo4j
mkdir logs
mkdir data
mkdir import
mkdir conf
```
```bash
ls neo4j/
conf  data  import  logs
```
Also:  
  : - data/ stores Neo4j graph databases. The default is graph.db, but you can make others, too  
  : - logs/ stores database activity logs. You can specify how often you want these to rotate
  : - import/ stores files, such as Json or CSV, that you can import to graph. I also put my Cypher scripts in here. More on that later
  : - conf/ stores a customised neo4j.conf file. More later about modifying specific settings

2. Start Neo4j in a Docker container
```bash
sudo docker run --rm --publish=7474:7474 --publish=7687:7687 --volume=$HOME/neo4j/data:/data --volume=$HOME/neo4j/logs:/logs --volume=$HOME/neo4j/import:/var/lib/neo4j/import --volume=$HOME/neo4j/conf:/var/lib/neo4j/conf neo4j:3.3
```
Also:  
  : - wherever you see a --volume parameter, Docker is instructed to connect the directory on your local file system with its equivalent inside the Neo4j Docker container
  : - note that connecting /conf directories must be linked, if you want the container utilize specific settings inside a customized neo4j.conf configuration file. This we will do shortly
  : - parameter neo4j:x.x.x refers to the version of Neo4j image you wish to run. If that version is not yet available in your local Docker repository, Docker will download it from its Neo4j Repository [[^1]]
  : - publishing of the two ports, 7474 and 7687 will allow you to interact with the graph data via Neo4j Browser

3. Find image reference to the running Neo4j container
```bash
sudo mv /var/lib/mysql-files/soil_survey.csv data-import-directory/
```

3. Make a customised neo4j.conf configuration file
```bash
sed -i '1i Hort_Client,Contractor,Region,Locality,Soil_Service,Solution,Soil_Issue,Date_Reported,Date_Actioned,DaysToAction' data-import-directory/soil_survey.csv
```

4. Stop Docker container, upload file, and restart 
```bash
sed -i '1d' import-directory/soil_survey.csv
```

5. Output after restarting the container
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

--- 
Footnotes:
[^1]: 1. [For latest Neo4j version see, Docker Neo4j Repository](https://hub.docker.com/_/neo4j/)
