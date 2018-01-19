---
layout: post
title: "Create a clean Neo4j graph database inside a Docker container"
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

By default, Neo4j CE comes with `graph.db` database that is awaiting to be populated with nodes, labels, properties and relationships. You will find this database at `~/neo4j/data/databases/graph.db/`. If you have not created any nodes or imported any data then `graph.db` remains empty or clean.

If that is the case then you don't need to read the rest of this post.

However, if you have already done some work inside`graph.db` then you will need to make some changes to the database before importing any CSV data.

#### Already played with Neo4j? ####

If you have already run some queries then you will notice that your Neo4j Browser's Database Information tab will display
something like this:

![Graph.db already contains nodes and relationships](/assets/images/graph_nodes_present.png)

Therefore, before you can import fresh data, you will need to clear out all nodes, labels, properties and relationships. On the surface this would appear to be very simple; and it should be but it's not.

You can go to your Neo4j Browser and run the following command:
```sql
MATCH (n) DETACH DELETE n;
```
The command above is designed to remove all nodes and all the information related to it. However, that is not what happens in reality.

Yes, the system does say the following:
```bash
Deleted 3320 nodes, deleted 12675 relationships, completed after 174 ms.
```
#### The Gotcha #### 

But Neo4j Browser's Database Information tab displays something unexpected:

![Graph.db still contains node labels and node properties](/assets/images/labels_properties_still_there.png)

You can also use these Cypher statements to confirm the above:
```sql
CALL db.labels();
```
```sql
CALL db.propertyKeys();
```

And the database size has increased!

#### The workaround

So here's what we will do:

1. Go to the terminal where you started Neo4j service inside the Docker container, and using Ctrl+C stop that service
```bash
...
2018-01-11 09:33:09.218+0000 INFO  Neo4j Server shutdown initiated by request                                   
2018-01-11 09:33:09.231+0000 INFO  Stopping...                                                                                               
2018-01-11 09:33:09.492+0000 INFO  Stopped.
```

2. Remove files under `~/neo4j/data/databases/graph.db/` directory that refer to the above labels and property keys inside the `graph.db` database
```bash
sudo rm -rf neo4j/data/databases/graph.db/*.*
```

3. Restart the Neo4j Docker container and the response you can expect to see is:
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
2018-01-11 09:39:11.347+0000 INFO  ======== Neo4j 3.3.1 ========                                                
2018-01-11 09:39:11.369+0000 INFO  Starting...                                                                  
2018-01-11 09:39:12.302+0000 INFO  Bolt enabled on 0.0.0.0:7687.                                                
2018-01-11 09:39:15.252+0000 INFO  Started.                                                                     
2018-01-11 09:39:16.093+0000 INFO  Remote interface available at http://localhost:7474/
```

4. Go back to Neo4j Browser and have a look at the Database Information tab:

![Graph.db contain no objects](/assets/images/clean_graph_db_database.png)

N.B.:

We can also wrap all of the above commands related to inside a shell script adapted from StackOverflow[[^1]]
```bash
#!/bin/sh
# script for clearing local graph.db database linked to Docker Neo4j container

echo Deleting all nodes and relationships from running Neo4j database

# delete all nodes and relationships
sudo docker exec -ti $(sudo docker ps --format '\{ \{.Names\} \}') bin/neo4j-shell -c "MATCH (n) DETACH DELETE n;"

echo Stopping Neo4j Docker container

# could also use sudo docker ps --format ''{{'.ID'}}'' 
# but that would assume that there is only a single Neo4j container running
sudo docker stop $(sudo docker ps -q --filter ancestor=neo4j:3.3)

echo Now removing graph.db labels and property keys

# remove all hanging labels and property kesys
sudo rm -rf neo4j/data/graph.db

sudo docker run --rm --publish=7474:7474 --publish=7687:7687 --volume=$HOME/neo4j/data:/data --volume=$HOME/neo4j/logs:/logs --volume=$HOME/neo4j/import:/var/lib/neo4j/import --volume=$HOME/neo4j/conf:/var/lib/neo4j/conf \
neo4j:3.3

```

---
***You have overcome a perplexing Gotcha related to DETACH DELETE Cypher command and got a graph.db database ready to receive your CSV data file***{: style="color: green"}

---
[Back to top of page](#)

---
[^1]: 1: [Delete nodes and relationships with Cypher](https://stackoverflow.com/questions/29711757/best-way-to-delete-all-nodes-and-relationships-in-cypher/29715865)

