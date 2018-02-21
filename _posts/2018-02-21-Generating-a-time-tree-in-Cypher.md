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

I found an article entitled, Modelling Dates Using Neo4j[[^1]], where the author describes how to create a time tree. There you can find references to several other posts which illustrate how to set up a time tree.
1. GraphAware Neo4j Time Tree[[^2]]
2. Neo4j: Cypher – Creating a time tree down to the day[[^3]]
3. A multilevel indexing structure (path tree)[[^4]]

What we need to do first, is to figure out the time range that will be applicable to Soil Survey data.

We'll use some Cypher to filter the earliest and the most recent date, thus giving us the range over which the time tree will be set up.

```sql
// find the earliest and the latest dates from either of the two date properties in the Soil_Report nodes
MATCH (s:Soil_Report)
WITH [max(s.action_date), max(s.report_date)] as max_list, [min(s.action_date), min(s.report_date)] as min_list
UNWIND max_list as flat_max
UNWIND min_list as flat_min
RETURN min(flat_min) as start_date_range, max(flat_max) as end_date_range
```
__Output:__
    
 ```bash
╒══════════════════╤════════════════╕
│"start_date_range"│"end_date_range"│
╞══════════════════╪════════════════╡
│"2007-05-07"      │"2014-04-14"    │
└──────────────────┴────────────────┘
```

This gives us a seven year period that the time tree will cover down to a day, from 1st January 2007 to 31st December 2014.

#### Design plan

In both cases my exercises is going to help you to locate similar properties.

I will break my exercise into three sections:
- build a soil profile of a given `Hort_Client` site based on soil conditions discovered during multiple soil tests
- create an algorithm that will find similar properties based on the profile that we developed
- drill down to the found properties' commonolities but also expose differences from the benchmark properties


#### 1. Generate a time tree

1. Given our range of years, 2007-2014, let's create the tree
Immediately following, Cypher will create `[:NEXT]` links between each subsequent days, extending out into the future

```sql
WITH range(2007, 2014) AS years, range(1,12) as months
FOREACH(year IN years | 
  MERGE (y:Year {year: year})
  FOREACH(month IN months | 
    CREATE (m:Month {month: month})
    MERGE (y)-[:HAS_MONTH]->(m)
    FOREACH(day IN (CASE 
                      WHEN month IN [1,3,5,7,8,10,12] THEN range(1,31) 
                      WHEN month = 2 THEN 
                        CASE
                          WHEN year % 4 <> 0 THEN range(1,28)
                          WHEN year % 100 <> 0 THEN range(1,29)
                          WHEN year % 400 = 0 THEN range(1,29)
                          ELSE range(1,28)
                        END
                      ELSE range(1,30)
                    END) |      
      CREATE (d:Day {day: day})
      MERGE (m)-[:HAS_DAY]->(d))))
 
WITH *

//create [:NEXT] relationships 
MATCH (year:Year)-[:HAS_MONTH]->(month)-[:HAS_DAY]->(day)
WITH year,month,day
ORDER BY year.year, month.month, day.day
WITH collect(day) as days
FOREACH(i in RANGE(0, size(days)-2) | 
    FOREACH(day1 in [days[i]] | 
        FOREACH(day2 in [days[i+1]] | 
            CREATE (day1)-[:NEXT]->(day2))))
```
__Output:__
    
 ```bash
Added 3026 labels, created 3026 nodes, set 3026 properties, created 5939 relationships, completed after 582 ms.
```
Let's take a date of the 1st of February 2014 and find the next three days

```sql
MATCH p = (y:Year {year: 2014})-[:HAS_MONTH]->(m:Month {month: 2})-[:HAS_DAY]->(:Day {day: 1})-[:NEXT*0..3]->(day)
RETURN y,m,day, relationships(p)
```
__Output:__
  
 ![Viewing :NEXT relationships](/assets/images/time_tree_NEXT.png)
  

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

[^1]: 1: [Modelling Dates Using Neo4j](https://www.menome.com/wp/neo4j-modelling-dates/)
[^2]: 2: [GraphAware Neo4j Time Tree](https://graphaware.com/neo4j/2014/08/20/graphaware-neo4j-timetree.html)
[^3]: 3: [Neo4j: Cypher – Creating a time tree down to the day](http://www.markhneedham.com/blog/2014/04/19/neo4j-cypher-creating-a-time-tree-down-to-the-day/)
[^4]: 4: [A multilevel indexing structure (path tree)](http://neo4j.com/docs/1.9.4/cypher-cookbook-path-tree.html)


