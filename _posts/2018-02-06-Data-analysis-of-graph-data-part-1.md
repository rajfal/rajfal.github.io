---
layout: post
title: "Graph database data analysis, part 1"
comments: false
description: "Cypher queries to analyze data in Neo4j"
categories: [neo4j, Cypher, data analysis]
tags: [neo4j, cypher, data import]
keywords: "neo4j, Cypher, data exploration, cypher query, cypher statements, data analysis"
---

> #### *Prerequisites:*{: style="color: red"}
> - [sample soil survey data loaded into a Neo4j graph](/2018/Import-CSV-data-into-Docker-Neo4j-container/)

---

#### Background

The other day I came across the post about [Getting Started with Data Analysis using Neo4j](https://neo4j.com/blog/getting-started-data-analysis-neo4j/)

It inspired me to look at the Soil Survey data and come up with different ways of analyzing the graph. I am going to dive straight into Cypher and explain what different statements aim to do as we go along.

#### 1. Obtain summary information

1. Let's find how many properties are in the survey data, how many different soil conditions have been found, what are they and how many soil tests have been performed.
  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)
RETURN count(DISTINCT h.name) as no_properties, count(DISTINCT s) as no_soil_issues,
collect(DISTINCT s.type) as soil_issues_present, count(DISTINCT ss) as no_analyses_completed
  ```
  Output:  
 ```bash
╒═══════════════╤══════════════════════════╤═════════════════════════════╤═══════════════════════╕
│"no_properties"│"soil_issues_investigated"│"soil_issues_present"        │"no_analyses_completed"│
╞═══════════════╪══════════════════════════╪═════════════════════════════╪═══════════════════════╡
│21             │12                        │["Erosion","LowOrganicMatter"│2670                   │
│               │                          │,"Acidification","Compaction"│                       │
│               │                          │,"LowOrganicBiota","HighAlkal│                       │
│               │                          │inity","Impermeable","HeavyMe│                       │
│               │                          │talContamination","LowPhospho│                       │
│               │                          │rus","Salinity","LowNitrogen"│                       │
│               │                          │,"LowPotassium"]             │                       │
└───────────────┴──────────────────────────┴─────────────────────────────┴───────────────────────┘
```

#### 2. What is the frequency of specific soil issues?

1. Now that we know what kind of soil problems exists in our survey data, let's seek how often each one occurs and list properties at which they are common.
  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)
RETURN s.type as soil_condition, count(h.name) as frequency, collect(h.name) as properties
ORDER BY frequency DESC
  ```
  Output:  
 ```bash
╒═════════════════════════╤═══════════╤══════════════════════════════════════════════════════════════════════╕
│"soil_condition"         │"frequency"│"properties"                                                          │
╞═════════════════════════╪═══════════╪══════════════════════════════════════════════════════════════════════╡
│"Erosion"                │20         │["hc_162","hc_167","hc_171","hc_175","hc_159","hc_174","hc_161","hc_16│
│                         │           │6","hc_164","hc_173","hc_157","hc_158","hc_168","hc_170","hc_163","hc_│
│                         │           │155","hc_165","hc_172","hc_160","hc_169"]                             │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"Compaction"             │14         │["hc_175","hc_163","hc_167","hc_171","hc_165","hc_168","hc_160","hc_17│
│                         │           │0","hc_161","hc_158","hc_155","hc_172","hc_169","hc_174"]             │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"HighAlkalinity"         │13         │["hc_175","hc_169","hc_163","hc_159","hc_166","hc_168","hc_156","hc_17│
│                         │           │2","hc_170","hc_155","hc_162","hc_165","hc_171"]                      │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"LowOrganicBiota"        │8          │["hc_175","hc_168","hc_169","hc_170","hc_159","hc_172","hc_174","hc_15│
│                         │           │8"]                                                                   │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"LowOrganicMatter"       │8          │["hc_167","hc_170","hc_163","hc_168","hc_161","hc_162","hc_169","hc_16│
│                         │           │5"]                                                                   │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"LowPhosphorus"          │7          │["hc_164","hc_174","hc_166","hc_162","hc_165","hc_157","hc_175"]      │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"Acidification"          │5          │["hc_171","hc_166","hc_156","hc_164","hc_167"]                        │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"Salinity"               │3          │["hc_155","hc_158","hc_172"]                                          │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"LowPotassium"           │1          │["hc_165"]                                                            │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"LowNitrogen"            │1          │["hc_175"]                                                            │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"HeavyMetalContamination"│1          │["hc_157"]                                                            │
├─────────────────────────┼───────────┼──────────────────────────────────────────────────────────────────────┤
│"Impermeable"            │1          │["hc_158"]                                                            │
└─────────────────────────┴───────────┴──────────────────────────────────────────────────────────────────────┘
```
#### 3. What is the frequency of specific soil conditions at a horticultural site?

1. Say that at `Hort_Client` property named `hc_175`, we want to get soil condition frequencies relevant to this property.
  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h:Hort_Client)
WHERE h.name='hc_175'
RETURN h.name, s.type as soil_condition, count(s) as no_found
ORDER BY h.name, no_found DESC
  ```
  Output:  
 ```bash
╒════════╤═════════════════╤══════════╕
│"h.name"│"soil_condition" │"no_found"│
╞════════╪═════════════════╪══════════╡
│"hc_175"│"Erosion"        │124       │
├────────┼─────────────────┼──────────┤
│"hc_175"│"LowOrganicBiota"│72        │
├────────┼─────────────────┼──────────┤
│"hc_175"│"HighAlkalinity" │45        │
├────────┼─────────────────┼──────────┤
│"hc_175"│"Compaction"     │42        │
├────────┼─────────────────┼──────────┤
│"hc_175"│"LowNitrogen"    │16        │
├────────┼─────────────────┼──────────┤
│"hc_175"│"LowPhosphorus"  │3         │
└────────┴─────────────────┴──────────┘
```

#### 4. Find the soil issues present for each `Hort_Client` and number of analyses done at each site, list the results in descending order of no_soil_analyses

1. Say that at `Hort_Client` property named `hc_175`, we want to get soil condition frequencies relevant to this property.
  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h:Hort_Client)
RETURN h.name, collect(DISTINCT s.type) as soil_condition, count(ss) as no_soil_analyses
ORDER BY no_soil_analyses DESC
  ```
  Output:  
 ```sql
╒════════╤══════════════════════════════════════════════════════════════════════╤══════════════════╕
│"h.name"│"soil_condition"                                                      │"no_soil_analyses"│
╞════════╪══════════════════════════════════════════════════════════════════════╪══════════════════╡
│"hc_175"│["Erosion","Compaction","LowOrganicBiota","HighAlkalinity","LowPhospho│302               │
│        │rus","LowNitrogen"]                                                   │                  │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_168"│["Erosion","LowOrganicMatter","Compaction","LowOrganicBiota","HighAlka│261               │
│        │linity"]                                                              │                  │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_172"│["Erosion","Compaction","LowOrganicBiota","HighAlkalinity","Salinity"]│222               │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_158"│["Erosion","Compaction","LowOrganicBiota","Impermeable","Salinity"]   │217               │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_166"│["Erosion","Acidification","HighAlkalinity","LowPhosphorus"]          │188               │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_169"│["Erosion","LowOrganicMatter","Compaction","LowOrganicBiota","HighAlka│182               │
│        │linity"]                                                              │                  │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_160"│["Erosion","Compaction"]                                              │175               │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_155"│["Erosion","Compaction","HighAlkalinity","Salinity"]                  │150               │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_170"│["Erosion","LowOrganicMatter","Compaction","LowOrganicBiota","HighAlka│148               │
│        │linity"]                                                              │                  │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_163"│["Erosion","LowOrganicMatter","Compaction","HighAlkalinity"]          │139               │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_165"│["Erosion","LowOrganicMatter","Compaction","HighAlkalinity","LowPhosph│128               │
│        │orus","LowPotassium"]                                                 │                  │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_167"│["Erosion","LowOrganicMatter","Acidification","Compaction"]           │123               │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_157"│["Erosion","HeavyMetalContamination","LowPhosphorus"]                 │118               │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_161"│["Erosion","LowOrganicMatter","Compaction"]                           │113               │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_162"│["Erosion","LowOrganicMatter","HighAlkalinity","LowPhosphorus"]       │100               │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_156"│["Acidification","HighAlkalinity"]                                    │79                │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_174"│["Erosion","Compaction","LowOrganicBiota","LowPhosphorus"]            │78                │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_159"│["Erosion","LowOrganicBiota","HighAlkalinity"]                        │69                │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_171"│["Erosion","Acidification","Compaction","HighAlkalinity"]             │56                │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_164"│["Erosion","Acidification","LowPhosphorus"]                           │39                │
├────────┼──────────────────────────────────────────────────────────────────────┼──────────────────┤
│"hc_173"│["Erosion"]                                                           │36                │
└────────┴──────────────────────────────────────────────────────────────────────┴──────────────────┘
```

#### Bonus: how to improve your Cypher query performance by over 1000%?

1. Running different queries to obtain the above result, I noticed an intriguing relationship between node-relationship path definition and the speed at which the results were retrieved. Here, I am referring to the first line of the code above,
```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h:Hort_Client)
```
While I used two other path configurations to get the same result they differed in performance. To illustrate performance of each path, I used the entire graph dataset.

__Case I - baseline__

Using a variable length path of between 1 and 2 relationships from `Soil_Issue` to `Hort_Client`.

```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[*1..2]-(h:Hort_Client)
RETURN h.name, s.type as soil_condition, count(s) as no_found
ORDER BY h.name, no_found DESC
```
```bash
Started streaming 82 records after 231 ms and completed after 232 ms.
Cypher version: CYPHER 3.3, planner: COST, runtime: INTERPRETED. 368963 total db hits in 211 ms.
```
__Case II - 580% improvement__

Using a fixed length path of exactly 2 relationships from `Soil_Issue` to `Hort_Client`.
We can also rewrite `[*2..2]` as `[*2]` 

```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[*2..2]-(h:Hort_Client)
RETURN h.name, s.type as soil_condition, count(s) as no_found
ORDER BY h.name, no_found DESC
```
```bash
Started streaming 82 records after 40 ms and completed after 40 ms.
Cypher version: CYPHER 3.3, planner: COST, runtime: INTERPRETED. 54921 total db hits in 59 ms.
```
__Case III - 1100% improvement__

Using relationships in any direction between `Soil_Service` and `Hort_Client`.

```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service){% raw %}--(h:Hort_Client){% endraw %}
RETURN h.name, s.type as soil_condition, count(s) as no_found
ORDER BY h.name, no_found DESC
```
```bash
Started streaming 82 records after 21 ms and completed after 21 ms. 
Cypher version: CYPHER 3.3, planner: COST, runtime: INTERPRETED. 39639 total db hits in 22 ms.
```
__Case IV - 1700% improvement__

Using a fully declared path spanning all nodes and relationships of interest.

```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h:Hort_Client)
RETURN h.name, s.type as soil_condition, count(s) as no_found
ORDER BY h.name, no_found DESC
```
```bash
Started streaming 82 records after 13 ms and completed after 13 ms.
Cypher version: CYPHER 3.3, planner: COST, runtime: INTERPRETED. 21299 total db hits in 14 ms.
```  
> *Conclusion: to minimise time of data retrieval specify the nodes and relationships that make up your path pattern*{: style="color: blue"}
 
 
---
***We have defined some business rules by which the data must play. We used Cypher queries to figure out where those rules are brok
en***{: style="color: green"}

---
[Back to top of page](#)

---

[^1]: 1: [Getting started with Data Analysis in Neo4j](https://neo4j.com/blog/getting-started-data-analysis-neo4j/)

