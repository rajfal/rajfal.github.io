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
> - [sample soil survey data loaded into a Neo4j graph](/2018/Import-CSV-data-into-Docker-Neo4j-container/)

---

#### Background

With our soil survey data in the graph, we can start inspecting for trends of interest. As the subject is a fictional environment, we can make some rules by which this 'economy' is expected to run. We can check for various metrics and look for findings that don't meet the expected norms. We will consider such discoveries as anomalous data patterns.

For example, given a pattern like
```bash
(n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(:Contractor)-[:OPERATES_IN]->(m:Region)
```
Let's say that any horticultural enterprise must not be found to hire contracting firm from more than one Region. In fact, a contracting license specifies that a Contractor can only operate within its own Region and thus must not allow itself to be drawn into deals that would require it to work cross-border.

Another limit we can place is that a Contractor must not extend its operation across some minimum number of clients. THe idea here is that each Contractor must maintain a minimum of familiarity with a given property and can not sacrifice that aspect of its business in preference to simply maximising its income across as many clients as possible but not retaining any detailed property knowledge...

#### Exploring specifics

1. Find all hort firms  who've hired contractors from  more than in one region. Regulations require that a hort firm only uses a single contractor.
```sql
MATCH (n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(:Contractor)-[:OPERATES_IN]->(m:Region)
WITH n.name as hort_name, n.client as sorting, count(DISTINCT m.name) as no_regions, 
collect(DISTINCT m.name) AS region_list
WHERE no_regions > 1
RETURN hort_name, no_regions, region_list 
ORDER BY sorting;
```
Output:  
  : - ```bash
╒═══════════╤════════════╤════════════════════════════════════╕
│"hort_name"│"no_regions"│"region_list"                       │
╞═══════════╪════════════╪════════════════════════════════════╡
│"hc_157"   │3           │["Northbury","Swifford","Eastling"] │
├───────────┼────────────┼────────────────────────────────────┤
│"hc_171"   │3           │["Northbury","Swifford","Westshire"]│
└───────────┴────────────┴────────────────────────────────────┘
```
Also:
  : - graph to expose the above patterns
  ```sql
  MATCH (n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(:Contractor)-[:OPERATES_IN]->(m:Region)
WITH n.name as hort_name, count(DISTINCT m.name) as no_regions
WHERE no_regions > 1
WITH hort_name
MATCH path = shortestPath((n1:Hort_Client)-[*1..3]-(m1:Region))
WHERE n1.name = hort_name
RETURN DISTINCT path; 
  ```
  : - ![Hort_Client with many regions](/assets/images/soil_survey_hort_firm_sourcing_contracts_from_many_regions.png)


2. Find all contractors  who worked  for  more than  X  clients. Regulations specify a max number of clients only. In this sample of records, we'll set X = 1
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
Also:
  : - graph that illustrates the above table
  ```sql
MATCH (n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(c:Contractor)
WITH c.name as contractor, count(DISTINCT n.name) as no_clients
WHERE no_clients > 1
WITH contractor
MATCH path = shortestPath((n1:Hort_Client)<-[*1..2]-(c1:Contractor))
WHERE c1.name = contractor
RETURN DISTINCT path;
  ```
  : - ![Contractor with two clients](/assets/images/soil_survey_contra_and_two_clients.png)
  
 
 
 
---
***we are defining business rules for the enterprises that generate and look where the rules are broken***{: style="color: green"}

---
[Back to top of page](#)

---



