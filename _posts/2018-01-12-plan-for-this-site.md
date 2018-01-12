---
layout: post
title: "Site's layout and content"
comments: false
description: "Plan for running data into Neo4j..."
keywords: "neo4j, docker, csv"
---

## Objective 1: import a denormalized CSV data into Neo4j

### Run Neo4j in two separate environments 

1. as a local laptop install
2. as a Docker container

* prepare MySQL data exports
- use bash to format and inspect files and add file header row
+ create a Cypher import file, \*.cql
- use neo4-shell utility to import data
- mention gotchas, such as Neo4j Browser after removing graph data but leaving node labels and key properties still visible

<div class="divider">Code block examples</div>

```sql
SELECT DISTINCT Region FROM soil_survey;
UPDATE soil_survey SET Region='Westshire' WHERE Region='NT';
```

```bash
head soil_survey
```
```cypher
USING PERIODIC COMMIT 1000
LOAD CSV WITH HEADERS FROM "file:///med-train-set.txt" AS line
WITH line LIMIT 10000
MERGE (c:Contractor {c_id: line.Contractor, name: 'contra_' + line.Contractor});

//count rels by node label
start n = node(*) MATCH (n)-[r]-() RETURN DISTINCT labels(n), type(r), count(r) as rel_count ORDER BY rel_count DESC;
```
