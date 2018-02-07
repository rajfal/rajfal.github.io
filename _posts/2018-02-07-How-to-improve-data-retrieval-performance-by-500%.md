---
layout: post
title: "How to tweak Cypher queries to improve data retrieval speeds by more than 500%"
comments: false
description: "Comparing Cypher pattern paths to improve data result performance"
categories: [neo4j, Cypher, query performance]
tags: [neo4j, cypher, graph data, query performance]
keywords: "neo4j, Cypher, query performance, cypher query, cypher performance"
---

> #### *Prerequisites:*{: style="color: red"}
> - [sample soil survey data loaded into a Neo4j graph](/2018/Import-CSV-data-into-Docker-Neo4j-container/)

---

#### Background

Neo4j Cypher Refcard[[^1]] is a good place to learn about constructing graph patterns to retrieve your data. 

I ended up experimenting with various patterns to give me the following output:

```bash
╒════════╤═════════════════════════╤══════════╕
│"h.name"│"soil_condition"         │"no_found"│
╞════════╪═════════════════════════╪══════════╡
│"hc_155"│"Erosion"                │52        │
├────────┼─────────────────────────┼──────────┤
│"hc_155"│"HighAlkalinity"         │45        │
├────────┼─────────────────────────┼──────────┤
│"hc_155"│"Compaction"             │43        │
├────────┼─────────────────────────┼──────────┤
│"hc_155"│"Salinity"               │10        │
├────────┼─────────────────────────┼──────────┤
│"hc_156"│"Acidification"          │41        │
├────────┼─────────────────────────┼──────────┤
...

├────────┼─────────────────────────┼──────────┤
│"hc_174"│"LowOrganicBiota"        │4         │
├────────┼─────────────────────────┼──────────┤
│"hc_175"│"Erosion"                │124       │
├────────┼─────────────────────────┼──────────┤
│"hc_175"│"LowOrganicBiota"        │72        │
├────────┼─────────────────────────┼──────────┤
│"hc_175"│"HighAlkalinity"         │45        │
├────────┼─────────────────────────┼──────────┤
│"hc_175"│"Compaction"             │42        │
├────────┼─────────────────────────┼──────────┤
│"hc_175"│"LowNitrogen"            │16        │
├────────┼─────────────────────────┼──────────┤
│"hc_175"│"LowPhosphorus"          │3         │
└────────┴─────────────────────────┴──────────┘
```

The above query looks at every `Hort_Client` in the graph and calculates the frequency of a `Soil_Issue` occuring at that site. Initially, I thought that if I use some way to save myself from excess typing then I will get best outcome in terms of speed. However, saving on keystrokes does not necessarily translate to getting your results faster. The more ambiguous the path specification the more nodes must be tested for matching what you want. This seriously begins to test your patience.

Running various Cypher queries, I noticed an intriguing relationship between node-relationship path definition and the speed at which the results were brought back. 

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
> *Conclusion: *{: style="color: blue"}

 
---
***The biggest takeaway - to minimise the speed of data retrieval, be specific about the nodes and relationships that make up your path pattern***{: style="color: green"}

---
[Back to top of page](#)

---

[^1]: 1: [Neo4j Cypher Refcard](https://neo4j.com/docs/cypher-refcard/current/)

