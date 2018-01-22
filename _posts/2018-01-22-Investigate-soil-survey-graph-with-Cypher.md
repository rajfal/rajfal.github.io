---
layout: post
title: "Investigate soil survey graph with Cypher"
comments: false
description: "Cypher queries to explore data in Neo4j"
categories: [neo4j, Cypher, data analysis]
tags: [neo4j, cypher, data import]
keywords: "neo4j, Cypher, data exploration, cypher query, cypher statements, data analysis"
---

> #### *Prerequisites:*{: style="color: red"}
> - [sample soil survey data loaded into a Neo4j graph](/2018/Import-CSV-data-into-Docker-Neo4j-container/)---

#### Background

With our soil survey data in the graph, we can start inspecting for trends of interest. As the subject is a fictional environment, we can make some rules by which this 'economy' is expected to run. We can check for various metrics and look for findings that don't meet the expected norms. We will consider such discoveries as anomalous data patterns.

For example, given a pattern like
```bash
(n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(:Contractor)-[:OPERATES_IN]->(m:Region)
```
Let's say that any horticultural enterprise must not be found to hire contracting firm from more than one Region. In fact, a contracting license specifies that a Contractor can only operate within its own Region and thus must not allow itself to be drawn into deals that would require it to work cross-border.

Another limit we can place is that a Contractor must not extend its operation across some minimum number of clients. THe idea here is that each Contractor must maintain a minimum of familiarity with a given property and can not sacrifice that aspect of its business in preference to simply maximising its income across as many clients as possible but not retaining any detailed property knowledge...

#### Exploring specifics

1. Find all hort firms  who've hired contractors from  more than in one region. Licenses specify a single region permit only
```sql
MATCH (n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(:Contractor)-[:OPERATES_IN]->(m:Region)
WITH n, count(DISTINCT m.name) as region_no, collect(DISTINCT m.name) AS regions
WHERE region_no > 1
RETURN n.client, region_no, regions 
ORDER BY n.client ;
```
Output:  
  : - ```bash
╒══════════╤═══════════╤════════════════════════════════════╕
│"n.client"│"region_no"│"regions"                           │
╞══════════╪═══════════╪════════════════════════════════════╡
│"157"     │3          │["Northbury","Swifford","Eastling"] │
├──────────┼───────────┼────────────────────────────────────┤
│"171"     │3          │["Northbury","Swifford","Westshire"]│
└──────────┴───────────┴────────────────────────────────────┘
```

2. Find all contractors  who worked  for  more than  two  clients. Licenses specify a max client permit only
```sql
MATCH (n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(c:Contractor)
WITH c.name as contractor, c.c_id as cid, count(DISTINCT n.name) as no_clients, collect(DISTINCT n.name) AS clients
WHERE no_clients > 1
RETURN contractor, no_clients, clients 
ORDER BY cid;
```
Output:  
  : - ```bash
╒═════════════╤════════════╤═══════════════════╕
│"contractor" │"no_clients"│"clients"          │
╞═════════════╪════════════╪═══════════════════╡
│"contra_1250"│2           │["hc_160","hc_167"]│
└─────────────┴────────────┴───────────────────┘
```
  
  
---
***we are defining business rules for the enterprises that generate and look where the rules are broken***{: style="color: green"}

---
[Back to top of page](#)

---



