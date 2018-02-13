---
layout: post
title: "Build a graph similarity score to find matching nodes"
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
you've had a disappointing experience with. 

Or, say that you are a contractor that has a novel method for remediating specific soil conditions. You had quantifiable success at sites that share common characteristics and to make your operations efficient you are targetting properties where such soil conditions exist and you are the only business in the market to provide a lasting solution.

#### Design plan

In both cases my exercises is going to help you to locate similar properties.

I will break my exercise into three sections:
- build a soil profile of a given `Hort_Client` site based on soil conditions discovered during multiple soil tests
- create an algorithm that will find similar properties based on the profile that we developed
- drill down to the found properties' commonolities but also expose differences from the benchmark properties


#### 1. Build a property soil profile

1. We know that each `Hort_Client` has period soil testing to determine and address ongoing or once-off `Soil_Issues`. Each property has about 5 years of soil testing data which gives us enough information to develop a soil profile for each property.

First, let's look at the summary of the property `hc_165`. We want to find what kind of soil conditions have been present and how many of each.

```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h:Hort_Client)
WHERE h.name='hc_165'
RETURN h.name, s.type as soil_condition, count(s) as no_found
ORDER BY h.name, no_found DESC  
```
__Output:__
    
 ```bash
╒════════╤══════════════════╤══════════╕
│"h.name"│"soil_condition"  │"no_found"│
╞════════╪══════════════════╪══════════╡
│"hc_165"│"Erosion"         │48        │
├────────┼──────────────────┼──────────┤
│"hc_165"│"Compaction"      │27        │
├────────┼──────────────────┼──────────┤
│"hc_165"│"HighAlkalinity"  │23        │
├────────┼──────────────────┼──────────┤
│"hc_165"│"LowPhosphorus"   │16        │
├────────┼──────────────────┼──────────┤
│"hc_165"│"LowOrganicMatter"│9         │
├────────┼──────────────────┼──────────┤
│"hc_165"│"LowPotassium"    │5         │
└────────┴──────────────────┴──────────┘
```
If we were to sum all of the soil conditions, we'd find that this property has been tested 128 times. It's clear that some properties would have been analysed more or less frequently than this particular one. So, we need a relative measure with which we can compare different `Hort_Client` sites. The easiest one to implement would be to use a percentage ratio, %.

Let's build one and examine what each part of Cypher code does, before I generate the result.

```sql
MATCH (h:Hort_Client {name :'hc_165'})-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h)
WITH h.name as property, count(s) as toto

MATCH (h:Hort_Client {name :property})-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h)
WITH property, toto, s, count(s) as total
RETURN property, s.type as soil_condition, total AS no_found, toInteger((total/toFloat((toto)))*100)+'%' AS frequency
ORDER BY total DESC
```

First thing to note is that we are using two `MATCH` clauses that target our pattern of interest. The first `MATCH` clause will give us a total count of soil tests carried out at this property. I will run a partial query statement so you can understand what the first block of code does.

```sql
MATCH (h:Hort_Client {name :'hc_165'})-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h)
RETURN h.name as property, count(s) as toto
```
__Output:__
  
```bash
╒══════════╤══════╕
│"property"│"toto"│
╞══════════╪══════╡
│"hc_165"  │128   │
└──────────┴──────┘
```

The `WITH` clause allows us to capture one specific node, h, and aggregates all soil issues using the function, count().

We can see our output because instead of using `WITH` clause we use the `RETURN` one. The second block of `MATCH` code will now receive two parameters, `property` and `toto`  with their respective values of "hc_165" and 128.

```sql
MATCH (h:Hort_Client {name :'hc_165'})-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h)
WITH h.name as property, count(s) as toto

MATCH (h:Hort_Client {name :property})-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h)
RETURN property, toto, s, count(s) as total
```
Whereas, the first `MATCH` block calculates total number of soil tests for the property, the second `MATCH` block sums the number of tests for each soil issue that was found for this `Hort_Client`. Also note that we just passed the variables, `property` and `toto` to the second `MATCH` clause

```bash
╒══════════╤══════╤═══════════════════════════╤═══════╕
│"property"│"toto"│"s"                        │"total"│
╞══════════╪══════╪═══════════════════════════╪═══════╡
│"hc_165"  │128   │{"type":"Compaction"}      │27     │
├──────────┼──────┼───────────────────────────┼───────┤
│"hc_165"  │128   │{"type":"Erosion"}         │48     │
├──────────┼──────┼───────────────────────────┼───────┤
│"hc_165"  │128   │{"type":"LowPhosphorus"}   │16     │
├──────────┼──────┼───────────────────────────┼───────┤
│"hc_165"  │128   │{"type":"HighAlkalinity"}  │23     │
├──────────┼──────┼───────────────────────────┼───────┤
│"hc_165"  │128   │{"type":"LowOrganicMatter"}│9      │
├──────────┼──────┼───────────────────────────┼───────┤
│"hc_165"  │128   │{"type":"LowPotassium"}    │5      │
└──────────┴──────┴───────────────────────────┴───────┘
```

Let's examine how % value for each `Soil_Issue` is calculated.
```python
toInteger((total/toFloat(toto))*100)+'%'
```

At this point, we need to do a couple of type conversions. Both `toto` and `total` variables are passed as integers. Dividing two integers will yield another integer. However, because we will end up with a rational number that starts with a decimal point, the end result will be a zero, 0. So we have to force a division by a Float type number. Hence, why we
apply ` toFloat(toto) ` conversion.

The result of `total` divided by the converted `toto` is then immediately multiplied by 100 to move the decimal point to the right and the `toInteger()` conversion extracts everything to the left of the decimal point as the result we want. 

The string '%' gets tacked to the end to let us know that we dealing with percentage values.


##### Evaluation steps for toInteger((total/toFloat(toto))*100)+'%'

| Calculation    | Result        | 
| -------------- |:-------------:|
| toFloat(128)   | 128.00        | 
| 48/128.00      | 0.375         |  
| 0.375x100      | 37.5          | 
| toInteger(37.5)| 37            | 
| 37+'%'         | 37%           | 


And the final result:

```bash
╒══════════╤══════════════════╤══════════╤═══════════╕
│"property"│"soil_condition"  │"no_found"│"frequency"│
╞══════════╪══════════════════╪══════════╪═══════════╡
│"hc_165"  │"Erosion"         │48        │"37%"      │
├──────────┼──────────────────┼──────────┼───────────┤
│"hc_165"  │"Compaction"      │27        │"21%"      │
├──────────┼──────────────────┼──────────┼───────────┤
│"hc_165"  │"HighAlkalinity"  │23        │"17%"      │
├──────────┼──────────────────┼──────────┼───────────┤
│"hc_165"  │"LowPhosphorus"   │16        │"12%"      │
├──────────┼──────────────────┼──────────┼───────────┤
│"hc_165"  │"LowOrganicMatter"│9         │"7%"       │
├──────────┼──────────────────┼──────────┼───────────┤
│"hc_165"  │"LowPotassium"    │5         │"3%"       │
└──────────┴──────────────────┴──────────┴───────────┘
```

#### 2. Create an algorithm that will find similar properties based on the profile that we developed

1. Once we've established the profile of the property, then we can use it as a benchmark to find properties that are like it. For example, we are interested in properties that have significant erosion and compaction problems but also have substantial problems with alkalinity, lack of organic matter and major element shortages, such as P and K.

Here is the Cypher code that would help us unearth such similarities.

```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[r:INVESTIGATES]-(ss:Soil_Service)-[r1:INVESTIGATES]->(s1:Soil_Issue)<-[:HAS]-(h1:Hort_Client)
WHERE h<>h1 AND h.name='hc_165'
RETURN h1.name AS related_hort_client, 
sum(
CASE s1.type 
  WHEN "Erosion" THEN 0.37 
  WHEN "Compaction" THEN 0.21
  WHEN "LowOrganicMatter" THEN 0.07
  WHEN "HighAlkalinity" THEN 0.17
  WHEN "LowPhosphorus" THEN 0.12
  WHEN "LowPotassium" Then 0.03  
  ELSE 0 
END
) as similarity_score
ORDER BY Score DESC
LIMIT 3 
```
__Output:__
    
 ```bash
╒═════════════════════╤══════════════════╕
│"related_hort_client"│"similarity_score"│
╞═════════════════════╪══════════════════╡
│"hc_170"             │127               │
├─────────────────────┼──────────────────┤
│"hc_169"             │127               │
├─────────────────────┼──────────────────┤
│"hc_168"             │127               │
└─────────────────────┴──────────────────┘
```

This query looks for similar properties calculating a score based a property's soil testing history. In this case, there are 3 similar properties, whose soil profile resembles `hc_165`. 

In this restricts findings to properties other than `hc_165`.
```python
WHERE h<>h1 AND h.name='hc_165'
```

In the following block of code, we are returning a single column. Everytime a specific `Soil_Issue` is found the algorithm gives it a value and adds to the previously summed figure. For example, if a property's soil test indicated that `High_Alkalinity` was found then 0.17 will be added. However, should such a test yield `LowOrganicBiota` then 0 will be added because the algorithm is not looking for this soil problem.

So you can see that each soil condition is assigned certain weights, and these weights are derived from the soil profile figures of the property, `hc_165`.

```python
sum(
CASE s1.type 
  WHEN "Erosion" THEN 0.37 
  WHEN "Compaction" THEN 0.21
  WHEN "LowOrganicMatter" THEN 0.07
  WHEN "HighAlkalinity" THEN 0.17
  WHEN "LowPhosphorus" THEN 0.12
  WHEN "LowPotassium" Then 0.03  
  ELSE 0 
END
) as similarity_score
```


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



 
---
***We analysed soil survey data, using aggregation functions such as collect() and count(), and filtering with WHERE***{: style="color: green"}

---
[Back to top of page](#)

---

[^1]: 1: [Getting started with Data Analysis in Neo4j](https://neo4j.com/blog/getting-started-data-analysis-neo4j/)

