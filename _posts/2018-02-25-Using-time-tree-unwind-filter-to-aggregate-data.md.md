---
layout: post
title: "Using time tree, Unwind clause and filter() function to aggregate data in Neo4j"
comments: true
published: true
description: "Produce summarised data using a time tree, unwind and filter"
categories: [neo4j, Cypher, unwind, time analysis, time tree]
tags: [neo4j, cypher, unwind, filter, time tree]
keywords: "neo4j, Cypher, unwind, filter, cypher query, date modelling, time analysis, time tree"
---

> #### *Prerequisites:*{: style="color: red"}
> - [sample soil survey data loaded into a Neo4j graph](/2018/Import-CSV-data-into-Docker-Neo4j-container/)
> - [time tree added as year-month-day related nodes](/2018/Generating-a-time-tree-in-Cypher/)

---

#### Background

With time dimension defined as nodes for each year, month and day, let's see what we can find out about frequency of `Soil_Issue`s across different time periods.

I will also build on each successive Cypher query to create more complex and comprehensive insights.


#### 1. Find specific soil issues that occurred in a given period

1. We will use the month and year of May 2007

```sql
MATCH (y:Year {year: 2007})-[:HAS_MONTH]->(m:Month {month: 5})-[:HAS_DAY]->(d:Day)<-[r:REPORTED_ON]-()-[*1..2]-(n:Soil_Issue) 
RETURN  y.year as Y, m.month as M, count(n) as No_found, collect(DISTINCT n.type) as Range_of_issues
ORDER By y.year, m.month
```
__Output:__
    
 ```bash
╒════╤═══╤══════════╤══════════════════════════════════════════════╕
│"Y" │"M"│"No_found"│"Range_of_issues"                             │
╞════╪═══╪══════════╪══════════════════════════════════════════════╡
│2007│5  │15        │["Erosion","HighAlkalinity","LowOrganicBiota"]│
└────┴───┴──────────┴──────────────────────────────────────────────┘
```

#### 2. Split each issue category to inspect specific frequencies

1. We will use the month and year of May 2007

```sql
MATCH (y:Year {year: 2007})-[:HAS_MONTH]->(m:Month {month: 5})-[:HAS_DAY]->(d:Day)<-[r:REPORTED_ON]-()-[*1..2]-(n:Soil_Issue) 
WITH y.year as Y, m.month as M, count(n) as all_issues , collect( n.type) as i
UNWIND i as Issues
WITH  Y, M, all_issues , Issues ORDER BY Issues
RETURN Y, M, collect(Issues) as issues, 
size(filter(x IN Issues WHERE x= 'Erosion')) as Erosion,
size(filter(x IN Issues WHERE x= 'HighAlkalinity')) as HighAlkalinity,
size(filter(x IN Issues WHERE x= 'LowOrganicBiota')) as LowOrganicBiota
```
__Output:__
    
 ```bash
╒════╤═══╤══════════════════════════════════════════════════════════════════════╤═════════╤════════════════╤═════════════════╕
│"Y" │"M"│"issues"                                                              │"Erosion"│"HighAlkalinity"│"LowOrganicBiota"│
╞════╪═══╪══════════════════════════════════════════════════════════════════════╪═════════╪════════════════╪═════════════════╡
│2007│5  │["LowOrganicBiota","LowOrganicBiota","LowOrganicBiota"]               │0        │0               │1                │
├────┼───┼──────────────────────────────────────────────────────────────────────┼─────────┼────────────────┼─────────────────┤
│2007│5  │["Erosion","Erosion","Erosion","Erosion","Erosion","Erosion","Erosion"│1        │0               │0                │
│    │   │,"Erosion","Erosion"]                                                 │         │                │                 │
├────┼───┼──────────────────────────────────────────────────────────────────────┼─────────┼────────────────┼─────────────────┤
│2007│5  │["HighAlkalinity","HighAlkalinity","HighAlkalinity"]                  │0        │1               │0                │
└────┴───┴──────────────────────────────────────────────────────────────────────┴─────────┴────────────────┴─────────────────┘
```

Or, we can present the same data in a more compact single line format.

```sql
MATCH (y:Year {year: 2007})-[:HAS_MONTH]->(m:Month {month: 5})-[:HAS_DAY]->(d:Day)<-[r:REPORTED_ON]-()-[*1..2]-(n:Soil_Issue) 
WITH y.year as Y, m.month as M, count(n) as all_issues , collect( n.type) as i
UNWIND i as Issues
WITH  Y, M, all_issues , Issues ORDER BY Issues
RETURN Y, M, collect(Issues) as issues
```
__Output:__
    
 ```bash
╒════╤═══╤══════════════════════════════════════════════════════════════════════╕
│"Y" │"M"│"issues"                                                              │
╞════╪═══╪══════════════════════════════════════════════════════════════════════╡
│2007│5  │["Erosion","Erosion","Erosion","Erosion","Erosion","Erosion","Erosion"│
│    │   │,"Erosion","Erosion","HighAlkalinity","HighAlkalinity","HighAlkalinity│
│    │   │","LowOrganicBiota","LowOrganicBiota","LowOrganicBiota"]              │
└────┴───┴──────────────────────────────────────────────────────────────────────┘
```

#### 3. Assign each string value to a separate variable

1. Note that both dates have been written to the graph as strings, e.g. "2011-05-14"

```sql
MATCH (s:Soil_Report{client:'171', recommendation:'6689', soil_analyst:'576'})
WITH s, split(s.report_date, '-') as r_on, split(s.action_date, '-')  as a_on
WITH s, r_on, a_on, toInteger(r_on[0]) as r_year, toInteger(r_on[1]) as r_month, toInteger(r_on[2]) as r_day,
toInteger(a_on[0]) as a_year, toInteger(a_on[1]) as a_month, toInteger(a_on[2]) as a_day
RETURN r_year, r_month, r_day, a_year, a_month, a_day
```
__Output:__
    
 ```bash
╒════════╤═════════╤═══════╤════════╤═════════╤═══════╕
│"r_year"│"r_month"│"r_day"│"a_year"│"a_month"│"a_day"│
╞════════╪═════════╪═══════╪════════╪═════════╪═══════╡
│2011    │7        │4      │2011    │11       │7      │
└────────┴─────────┴───────┴────────┴─────────┴───────┘
```

#### 4. Matching variables to specific time tree day nodes
 
```sql
MATCH (s:Soil_Report{client:'171', recommendation:'6689', soil_analyst:'576'})
WITH s, split(s.report_date, '-') as r_on, split(s.action_date, '-')  as a_on
WITH s, r_on, a_on, toInteger(r_on[0]) as r_year, toInteger(r_on[1]) as r_month, toInteger(r_on[2]) as r_day,
toInteger(a_on[0]) as a_year, toInteger(a_on[1]) as a_month, toInteger(a_on[2]) as a_day

MATCH (y:Year {year:r_year})-[:HAS_MONTH]->(m:Month {month:r_month})-[:HAS_DAY]->(r_d:Day {day:r_day})
MATCH (y1:Year {year:a_year})-[:HAS_MONTH]->(m1:Month {month:a_month})-[:HAS_DAY]->(a_d:Day {day:a_day})

RETURN y,m,r_d, y1, m1, a_d
```
__Output:__ 

![Matching variables to time tree](/assets/images/time_tree_match_to_vars.png)


#### 5. Binding time tree day nodes to Soil_Report node
 
```sql
MATCH (s:Soil_Report{client:'171', recommendation:'6689', soil_analyst:'576'})
WITH s, split(s.report_date, '-') as r_on, split(s.action_date, '-')  as a_on
WITH s, r_on, a_on, toInteger(r_on[0]) as r_year, toInteger(r_on[1]) as r_month, toInteger(r_on[2]) as r_day,
toInteger(a_on[0]) as a_year, toInteger(a_on[1]) as a_month, toInteger(a_on[2]) as a_day

MATCH (y:Year {year:r_year})-[:HAS_MONTH]->(m:Month {month:r_month})-[:HAS_DAY]->(r_d:Day {day:r_day})
MATCH (y1:Year {year:a_year})-[:HAS_MONTH]->(m1:Month {month:a_month})-[:HAS_DAY]->(a_d:Day {day:a_day})

MERGE (s)-[:REPORTED_ON]->(r_d)
MERGE (s)-[:ACTIONED_ON]->(a_d)

RETURN ()-[:REPORTED_ON]-(s)-[:ACTIONED_ON]-()
```
__Output:__ 

![Nodes bound to time tree](/assets/images/time_tree_bound_nodes.png)


__PS:__ 

1. BTW, updated graph schema :)

![New graph database schema](/assets/images/soil_survey_meta_graph_II.png)
 
 
---
***We stepped through the process of binding an existing `Soil_Report` node to day nodes that are part of the previously implemented time tree - we are ready to refactor the remaining Soil Report nodes***{: style="color: green"}

---
[Back to top of page](#)

---


