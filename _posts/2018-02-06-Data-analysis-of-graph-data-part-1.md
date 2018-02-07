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

The other day I came across the post about Getting [Started with Data Analysis using Neo4j](https://neo4j.com/blog/getting-started-data-analysis-neo4j/)

It inspired me to look at the Soil Survey data and come up with different ways of analyzing the graph. I am going to dive straight into Cypher and explain what different statements aim to do as we go along.

#### 1. Obtain summary information

> *Business rule: a `Hort_Client` can hire any `Contractor` but only from its own and one region*{: style="color: blue"}

1. Let's find how many properties are in the survey data, how many different soil conditions have been found, what are they and how many soil tests have been performed.
  ```sql
MATCH (h:Hort_Client)-[:HAS]->(s:Soil_Issue)<-[:INVESTIGATES]-(ss:Soil_Service)
RETURN count(DISTINCT h.name) as no_properties, count(DISTINCT s) as no_soil_issues,
collect(DISTINCT s.type) as soil_issues_present, count(DISTINCT ss) as no_analyses_completed
  ```
  Output:  
  : - ```bash
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

3. Let's use a more incisive graph approach to show us the suspect relationships
  ```sql
MATCH (n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(:Contractor)-[:OPERATES_IN]->(m:Region)
WITH n.name as hort_name, count(DISTINCT m.name) as no_regions
WHERE no_regions > 1
WITH hort_name
MATCH path = shortestPath((n1:Hort_Client)-[*1..3]-(m1:Region))
WHERE n1.name = hort_name
RETURN DISTINCT path; 
  ```
  Output:
  : - it is *hc_157*{: style="color: red"} and *hc_171*{: style="color: red"} who've done cross-border deals
  ![Hort_Client with many regions](/assets/images/soil_survey_hort_firm_sourcing_contracts_from_many_regions.png)
  
> *Business rule: a `Contractor` can have no more than X `Hort_Client`s*{: style="color: blue"}

1. See contractors and their clients
  ```sql
MATCH (n:Hort_Client)<-[:SENT_TO]-(:Soil_Report)<-[:ACTIONS]-(c:Contractor)
WITH c.name as contractor
MATCH path = shortestPath((n1:Hort_Client)<-[*1..2]-(c1:Contractor))
WHERE c1.name = contractor
RETURN DISTINCT path;
  ```
  Output:
  : - Displaying `Contractor`s and their `Hort_Client` nodes. But are there any contractors who contravene the rules?
  ![Contractors and their Hort_Clients](/assets/images/soil_survey_contractors_and_hort_firms.png)
  

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

