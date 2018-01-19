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

**\*Original MySQL data:**
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
| Hort_Client | Contractor | Region   | Locality | Soil_Service | Solution | Soil_Issue | Date_Reported | Date_Actioned | DaysToAction |
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
|         160 |       1250 | Eastling | 1715     |         3855 |     2786 | Erosion    | 2009-08-10    | 2009-09-28    |           49 |
|         175 |        415 | Swifford | 552      |         2362 |     6684 | Erosion    | 2008-04-08    | 2008-04-08    |            0 |
|         160 |       1250 | Eastling | 937      |         2106 |    10773 | Erosion    | 2011-10-24    | 2012-02-13    |          112 |
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
3 rows in set (0.00 sec)

---
