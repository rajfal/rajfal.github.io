---
layout: post
title: "Generate a time tree in Cypher to model dates"
comments: true
published: true
description: "Use Cypher to generate a year-month-day time tree"
categories: [neo4j, Cypher, date modelling, time analysis, time tree]
tags: [neo4j, cypher, date model, time tree]
keywords: "neo4j, Cypher, data exploration, cypher query, date modelling, time analysis, time tree"
---

> #### *Prerequisites:*{: style="color: red"}
> - [sample soil survey data loaded into a Neo4j graph](/2018/Import-CSV-data-into-Docker-Neo4j-container/)

---

#### Background

What I want to explore next are the soil analysis trends across time. Initially, I did not add the time dimension to
my model as a first class citizen and merely appended the original MySQL date information as a property of the `Soil_Report` node:

```python
{
  "recommendation": "4082",
  "action_date": "2013-08-12",
  "client": "157",
  "days_delayed": "56",
  "soil_analyst": "7320",
  "report_date": "2013-06-17"
}
```
Note above that the two dates, `action_date` and `report_date` are saved as strings. This data type will become tricky to parse. It would be far more efficient to create events to which these dates apply.

Now that we have data in the graph, the word 'refactoring' comes to mind. That is effectively, what we will do here. However, one step a time. Hence, why we will start with generating a time tree where dates themselves in form of year, month and day nodes will be a graph itself.

Following on from the post entitled, Getting started with Data Analysis in Neo4j[[^1]], the author makes a reference to
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

#### 3. Confirm the found properties' common characteristic and expose differences from the benchmark property

1. The similarity score indicated that `hc_168`, `hc_169` and `hc_170` are the most similar to `hc_165` than any other properties in our register. 

Now, we want to figure out whether the exact details by generating and inspecting each property's soil profile. So, we don't just want to take the algorithm's decision for granted, we want to view the details ourselves.

At least, until we can trust the algorithm.

So, what kind of soil issues do these four properties share?

  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h)
WHERE h.name IN ["hc_168", "hc_169", "hc_170", "hc_165"] //
WITH h.name as property, collect(DISTINCT s.type) as soil, count(ss) as no_soil_tests
UNWIND soil as issues
WITH property, issues order by issues, no_soil_tests
RETURN property, collect(issues) as sorted, no_soil_tests
ORDER BY property
  ```
__Output:__
    
 ```bash
╒══════════╤═══════════════════════════════════════════════════════════════════════════════════════════╤═══════════════╕
│"property"│"sorted"                                                                                   │"no_soil_tests"│
╞══════════╪═══════════════════════════════════════════════════════════════════════════════════════════╪═══════════════╡
│"hc_165"  │["Compaction","Erosion","HighAlkalinity","LowOrganicMatter","LowPhosphorus","LowPotassium"]│128            │
├──────────┼───────────────────────────────────────────────────────────────────────────────────────────┼───────────────┤
│"hc_168"  │["Compaction","Erosion","HighAlkalinity","LowOrganicBiota","LowOrganicMatter"]             │261            │
├──────────┼───────────────────────────────────────────────────────────────────────────────────────────┼───────────────┤
│"hc_169"  │["Compaction","Erosion","HighAlkalinity","LowOrganicBiota","LowOrganicMatter"]             │182            │
├──────────┼───────────────────────────────────────────────────────────────────────────────────────────┼───────────────┤
│"hc_170"  │["Compaction","Erosion","HighAlkalinity","LowOrganicBiota","LowOrganicMatter"]             │148            │
└──────────┴───────────────────────────────────────────────────────────────────────────────────────────┴───────────────┘
```

Next, we'll drill down deeper to uncover the actual soil profile figures for each `Hort_Client`

  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)<-[:REQUESTS]-(h:Hort_Client)
WHERE h.name IN ["hc_168", "hc_169", "hc_170", "hc_165"]
WITH h.name AS hcs, collect( s.type) AS soil_conditions, count(s) AS total
UNWIND soil_conditions AS list
WITH hcs, list, COUNT(list) AS count, total
with hcs, list ORDER BY list, count, total
WITH hcs, COLLECT(list) AS values, COLLECT(count) AS counts, total
UNWIND hcs AS hort_client
RETURN
  hort_client, EXTRACT(i IN RANGE(0, SIZE(values) - 1) | [values[i], toInteger((counts[i]/toFloat(total))*100)+'%']) AS soil_profile
ORDER BY hort_client
  ```
__Output:__
    
 ```bash
╒═════════════╤════════════════════════════════════════════════════════════════════════════════════════╕
│"hort_client"│"soil_profile"                                                                          │
╞═════════════╪════════════════════════════════════════════════════════════════════════════════════════╡
│"hc_165"     │[["Compaction","21%"],["Erosion","37%"],["HighAlkalinity","17%"],["LowOrganicMatter","7%│
│             │"],["LowPhosphorus","12%"],["LowPotassium","3%"]]                                       │
├─────────────┼────────────────────────────────────────────────────────────────────────────────────────┤
│"hc_168"     │[["Compaction","14%"],["Erosion","44%"],["HighAlkalinity","13%"],["LowOrganicBiota","18%│
│             │"],["LowOrganicMatter","8%"]]                                                           │
├─────────────┼────────────────────────────────────────────────────────────────────────────────────────┤
│"hc_169"     │[["Compaction","23%"],["Erosion","8%"],["HighAlkalinity","23%"],["LowOrganicBiota","22%"│
│             │],["LowOrganicMatter","21%"]]                                                           │
├─────────────┼────────────────────────────────────────────────────────────────────────────────────────┤
│"hc_170"     │[["Compaction","25%"],["Erosion","43%"],["HighAlkalinity","24%"],["LowOrganicBiota","1%"│
│             │],["LowOrganicMatter","6%"]]                                                            │
└─────────────┴────────────────────────────────────────────────────────────────────────────────────────┘
```

BTW, I was inspired by the discussion around the topic of [How to aggregate an aggregated list in cypher](https://stackoverflow.com/questions/40713658/how-to-aggregate-an-aggregated-list-in-cypher) to modify and develop this last block of Cypher code.
 
---
***We built a similarity scoring algorithm for nodes and their aggregated data, using a soil profile, feature weights and then we confirmed that indeed the similar properties shared common characteristics***{: style="color: green"}

---
[Back to top of page](#)

---

[^1]: 1: [Getting started with Data Analysis in Neo4j](https://neo4j.com/blog/getting-started-data-analysis-neo4j/)

