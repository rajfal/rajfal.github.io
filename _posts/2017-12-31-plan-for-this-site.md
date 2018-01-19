---
layout: post
title: "Goal for this site"
comments: false
description: "Running relational data into a Neo4j graph"
categories: neo4j docker paperwork
keywords: "neo4j, docker, csv"
---

### Goal: import Relational data into a Connected graph

** From this denormalized MySQL data ...**

```bash
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
| Hort_Client | Contractor | Region   | Locality | Soil_Service | Solution | Soil_Issue | Date_Reported | Date_Actioned | DaysToAction |
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
|         160 |       1250 | Eastling | 1715     |         3855 |     2786 | Erosion    | 2009-08-10    | 2009-09-28    |           49 |
|         175 |        415 | Swifford | 552      |         2362 |     6684 | Erosion    | 2008-04-08    | 2008-04-08    |            0 |
|         160 |       1250 | Eastling | 937      |         2106 |    10773 | Erosion    | 2011-10-24    | 2012-02-13    |          112 |
+-------------+------------+----------+----------+--------------+----------+------------+---------------+---------------+--------------+
3 rows in set (0.00 sec)
```
** ... to this graph schema in Neo4j running in a Docker container**
Soil Survey meta-graph:  
  : - ![Soil Survey meta-graph](/assets/images/soil_survey_meta_graph.png)
---
