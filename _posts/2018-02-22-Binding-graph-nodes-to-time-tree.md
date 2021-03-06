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

Now that we have the time tree in place, let's bind `Soil_Report` node using data from its two properties, `action_date` and `report_date` to specific year-month-day nodes on the time tree. As a reminder let's review the current property list for a `Soil_Report` node:

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
We will go step by step to understand exactly how this node binding works.


#### 1. Retrieve date-related properties fron the node

1. As a test run we'll use a very specific `Soil_Report` node

```sql
MATCH (s:Soil_Report{client:'171', recommendation:'6689', soil_analyst:'576'})
RETURN s.action_date, s.report_date, s.days_delayed
```
__Output:__
    
 ```bash
╒═══════════════╤═══════════════╤════════════════╕
│"s.action_date"│"s.report_date"│"s.days_delayed"│
╞═══════════════╪═══════════════╪════════════════╡
│"2011-11-07"   │"2011-07-04"   │"126"           │
└───────────────┴───────────────┴────────────────┘
```

#### 2. Split each date property string into a list of strings

1. Note that both dates have been written to the graph as strings, e.g. "2011-05-14"

```sql
MATCH (s:Soil_Report{client:'171', recommendation:'6689', soil_analyst:'576'})
WITH s, split(s.report_date, '-') as reported_on, split(s.action_date, '-')  as actioned_on
RETURN reported_on, actioned_on
```
__Output:__
    
 ```bash
╒══════════════════╤══════════════════╕
│"reported_on"     │"actioned_on"     │
╞══════════════════╪══════════════════╡
│["2011","07","04"]│["2011","11","07"]│
└──────────────────┴──────────────────┘
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


