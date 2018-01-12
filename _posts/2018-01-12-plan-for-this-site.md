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

### Footnote

There will be some URL's references that I will place as footnotes at the bottom of the page...I think! 

This is an example for the footnote number one [[^1]]. You can even add more footnotes, with link! [[^2]]

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

### Table 1: With Alignment

| Location Reference        | Local install          | Docker container  |
| ------------------------- |:----------------------:| -----------------:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

---
Footnote:

[^1]: 1: Footnote number one yeah baby!

[^2]: 2: A footnote you can link to - [click here!](#)
