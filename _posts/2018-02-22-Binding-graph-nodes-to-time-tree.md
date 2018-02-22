---
layout: post
title: "Binding `Soil_Report` nodes to days in time tree - step by step"
comments: true
published: true
description: "Refactor nodes to connect them to new nodes in a time tree"
categories: [neo4j, Cypher, graph refactoring, time analysis, time tree]
tags: [neo4j, cypher, date model, time tree]
keywords: "neo4j, Cypher, graph refactoring, cypher query, date modelling, time analysis, time tree"
---

> #### *Prerequisites:*{: style="color: red"}
> - [sample soil survey data loaded into a Neo4j graph](/2018/Import-CSV-data-into-Docker-Neo4j-container/)
> - [time tree added as year-month-day related nodes](/2018/Generating-a-time-tree-in-Cypher/)

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

Now, let's take a date of the 1st of February 2014 and find the next three days.

```sql
MATCH p = (y:Year {year: 2014})-[:HAS_MONTH]->(m:Month {month: 2})-[:HAS_DAY]->(:Day {day: 1})-[:NEXT*0..3]->(day)
RETURN y,m,day, relationships(p)
```
__Output:__
  
 ![Viewing :NEXT relationships](/assets/images/time_tree_NEXT.png)
  

#### 2. Create relationships between days that go back in time

1. I also wanted to generate relationships tell us what is the date `[:PREVIOUS]` to the one we are currently viewing. Such as, what are the last three days before the one in question.

This particular piece of Cypher I had to create and it works in the reverse order of the one that determines the `[:NEXT]` relationships.

```sql
MATCH (year:Year)-[:HAS_MONTH]->(month)-[:HAS_DAY]->(day)
WITH year,month,day
ORDER BY year.year, month.month, day.day
WITH collect(day) as days
FOREACH(i in RANGE(size(days)-1, 0, -1) | 
    FOREACH(day2 in [days[i]] | 
        FOREACH(day1 in [days[i-1]] | 
            CREATE (day2)-[:PREVIOUS]->(day1))))
```
__Output:__
    
 ```bash
Created 2922 relationships, completed after 73 ms.
```

Now, let's take a date of the 1st of February 2014 and find the previous three days.

```sql
MATCH p = (y:Year {year: 2014})-[:HAS_MONTH]->(m:Month {month: 2})-[:HAS_DAY]->(:Day {day: 1})-[:PREVIOUS*0..3]->(day)
RETURN y,m,day, relationships(p)
```
__Output:__
  
 ![Viewing :PREVIOUS relationships](/assets/images/time_tree_PREVIOUS.png)
 

#### 3. Looking at both :NEXT and :PREVIOUS relationships
 

![Viewing :PREVIOUS+:NEXT relationships](/assets/images/time_tree_both_ways.png)
 
---
***We generated a time tree linking years, months and days with additional relationships that tell us any date's Next and Previous date***{: style="color: green"}

---
[Back to top of page](#)

---

[^1]: 1: [Modelling Dates Using Neo4j](https://www.menome.com/wp/neo4j-modelling-dates/)
[^2]: 2: [GraphAware Neo4j Time Tree](https://graphaware.com/neo4j/2014/08/20/graphaware-neo4j-timetree.html)
[^3]: 3: [Neo4j: Cypher – Creating a time tree down to the day](http://www.markhneedham.com/blog/2014/04/19/neo4j-cypher-creating-a-time-tree-down-to-the-day/)
[^4]: 4: [A multilevel indexing structure (path tree)](http://neo4j.com/docs/1.9.4/cypher-cookbook-path-tree.html)


