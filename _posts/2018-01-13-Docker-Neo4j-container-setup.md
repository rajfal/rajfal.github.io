---
layout: post
title: "Configure a setup of Neo4j container inside Docker"
comments: false
description: "step by step guide on spinnning up a Neo4j database container within Docker environment"
categories: Neo4j Docker 
keywords: "neo4j, Docker, docker container, Neo4j graph, neo4j.conf"
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
[^1]: 1: [Docker Neo4j Repository](https://hub.docker.com/_/neo4j/)
[^2]: 2: StackOverFlow tip: [Extract last part of string in bash](https://stackoverflow.com/questions/12426659/how-extract-last-part-of-string-in-bash)
