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

> *Business rule: a `Hort_Client` can hire any `Contractor` but only from its own and one region*{: style="color: blue"}

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

2. We can see the related nodes of interest in a table that shows us contractors  who worked for more than X clients. For this sample of records, we'll set X = 1
```sql
MATCH (n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(c:Contractor)
WITH c.name as contractor, c.c_id as sorting, count(DISTINCT n.name) as no_clients, 
collect(DISTINCT n.name) AS client_list
WHERE no_clients > 1
RETURN contractor, no_clients, clients 
ORDER BY sorting;
```
Output:  
  : - ```bash
╒═════════════╤════════════╤═══════════════════╕
│"contractor" │"no_clients"│"client_list"      │
╞═════════════╪════════════╪═══════════════════╡
│"contra_1250"│2           │["hc_160","hc_167"]│
└─────────────┴────────────┴───────────────────┘
```

3. A more direct graph view will disclose the suspect activities
  ```sql
MATCH (n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(c:Contractor)
WITH c.name as contractor, count(DISTINCT n.name) as no_clients
WHERE no_clients > 1
WITH contractor
MATCH path = shortestPath((n1:Hort_Client)<-[*1..2]-(c1:Contractor))
WHERE c1.name = contractor
RETURN DISTINCT path; 
  ```
  Output:
  : - it is *contra_1250*{: style="color: red"} who is moonshining for another `Hort_Client`
  ![Hort_Client with many regions](/assets/images/soil_survey_contra_and_two_clients.png)
  
 
 
---
***We have defined some business rules by which the data must play. We used Cypher queries to figure out where those rules are broken***{: style="color: green"}

---
[Back to top of page](#)

---

[^1]: 1: [Getting started with Data Analysis in Neo4j](https://neo4j.com/blog/getting-started-data-analysis-neo4j/)

