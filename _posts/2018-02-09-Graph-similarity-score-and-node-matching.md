---
layout: post
title: "How to implement a graph similarity score to find matching nodes"
comments: true
description: "Use similarity scoring with Cypher queries to find nodes with similar attributes"
categories: [neo4j, Cypher, data analysis, similarity score]
tags: [neo4j, cypher, data import, similarity score]
keywords: "neo4j, Cypher, data exploration, cypher query, similarity score, data analysis, collect(), reduce()"
---

> #### *Prerequisites:*{: style="color: red"}
> - [sample soil survey data loaded into a Neo4j graph](/2018/Import-CSV-data-into-Docker-Neo4j-container/)

---

#### Background

Following on from the post entitled, Getting Started with Data Analysis using Neo4j[[^1]], the author makes a reference to
finding recommendations. I decided to implemented something similar with my own data. 

Imagine that you are a horticultural property investor and you know from experience that certain sets of soil problems are 
difficult and expensive to address. Whenever you consider a property, you wish to know whether it is anything like the one
you've had a disappointing experience with. Or, say that you are a contractor that has a novel method for remediating
specific soil conditions. You had quantifiable success at sites that share common characteristics and to make your operations
efficient you are targetting properties where such soil conditions exist and you are the only business in the market to
provide a lasting solution.

In both cases my exercises is going to help you to locate similar properties.

I will break my exercise into three sections:
- build a soil profile of a given `Hort_Client` site based on soil conditions discovered during multiple soil tests
- create an algorithm that will find similar properties based on the profile that we developed
- drill down to the found properties' commonolities but also expose differences from the benchmark properties


#### 1. Build a soil profile for a `Hort_Client` property

1. Let's find how many properties are in the survey data, how many different soil conditions have been found, what are they and how many soil tests have been performed. I am using Cypher's in-built [`collect()`](https://neo4j.com/docs/developer-manual/current/cypher/functions/aggregating/#functions-collect) function to amalgamate multiple values into a single list that will be displayed under its own column.

  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)
RETURN count(DISTINCT h.name) as no_properties, count(DISTINCT s) as no_soil_issues,
collect(DISTINCT s.type) as soil_issues_present, count(DISTINCT ss) as no_analyses_completed
  ```
__Output:__
    
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

1. Now that we know what kind of soil problems exists in our survey data, let's seek how often each one occurs and list properties at which they are common. The `collect()` function allows us to view all properties that share a specific soil condition.

  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)
RETURN s.type as soil_condition, count(h.name) as frequency, collect(h.name) as properties
ORDER BY frequency DESC
  ```
__Output:__
    
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

1. Say that at `Hort_Client` property named `hc_175`, we want to get soil condition frequencies relevant to this property. We achieved the desired filtering, using the [`WHERE`](https://neo4j.com/docs/developer-manual/current/cypher/clauses/where/) condition as part of the `MATCH` clause.
  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h:Hort_Client)
WHERE h.name='hc_175'
RETURN h.name, s.type as soil_condition, count(s) as no_found
ORDER BY h.name, no_found DESC
  ```
__Output:__
    
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

#### 4. Find the soil issues present at each property, the number of analyses done at each site

1. Say that at `Hort_Client` property named `hc_175`, we want to get soil condition frequencies relevant to this property.
  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h:Hort_Client)
RETURN h.name, collect(DISTINCT s.type) as soil_condition, count(ss) as no_soil_analyses
ORDER BY no_soil_analyses DESC
  ```
__Output:__ 
    
 ```bash
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
 
---
***We analysed soil survey data, using aggregation functions such as collect() and count(), and filtering with WHERE***{: style="color: green"}

---
[Back to top of page](#)

---

[^1]: 1: [Getting started with Data Analysis in Neo4j](https://neo4j.com/blog/getting-started-data-analysis-neo4j/)

