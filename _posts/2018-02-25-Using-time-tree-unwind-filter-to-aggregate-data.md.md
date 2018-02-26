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

Explanation of Cypher code used below:

```python
collect( n.type) as i
UNWIND i as Issues
WITH  Y, M, all_issues , Issues ORDER BY Issues
RETURN Y, M, collect(Issues) as issues, 
size(filter(x IN Issues WHERE x= 'Erosion')) as Erosion
...
```

Let's study the above code, so we understand what exactly is being done.

`n.type` represent all rows of `Soil_Issue`s that occur throughout May 2007

We then apply collect() function to turn them into a list, `i`, of the format, ['c','b', 'a']
Next, we apply UNWIND clause to turn that list into a rows-based variable called `Issues` - this will allow sorting

WITH clause then applies alphabetic sort to `Issues`, see `Issues ORDER BY Issues`

RETURN clause applies another collect() function call to gather these ordered `Issues` into a variable called `issues`

So what's with `size(filter(x IN Issues WHERE x= 'Erosion')) as Erosion`?
We use filter() function to pull out only those soil issues of the type 'Erosion'. Every item `x` is tested to see if it contains 'Erosion'. If it does then it's added to the new internal list. 

Once all the `Issues` have been filtered, we apply size() function on the new list to arrive at the total number of Erosion issues.

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

#### 3. Count frequencies of each soil issue in selected months of 2007

1. In this example, we are using the months of May and July of 2007

```sql
UNWIND [2007] as years
UNWIND [5, 7] as months
MATCH (y:Year {year: years})-[:HAS_MONTH]->(m:Month {month: months})-[:HAS_DAY]->(d:Day)<-[r:REPORTED_ON]-()-[*1..2]-(n:Soil_Issue) 
with y.year as Y, m.month as M, count(n) as all_issues , collect( n.type) as Issues
RETURN Y + ' - ' + M as Period,
size(filter(x IN Issues WHERE x= 'Erosion')) as Erosion,
size(filter(x IN Issues WHERE x= 'HighAlkalinity')) as HighAlkalinity,
size(filter(x IN Issues WHERE x= 'LowOrganicBiota')) as LowOrganicBiota
ORDER By Y, M
```
__Output:__
    
 ```bash
╒══════════╤═════════╤════════════════╤═════════════════╕
│"Period"  │"Erosion"│"HighAlkalinity"│"LowOrganicBiota"│
╞══════════╪═════════╪════════════════╪═════════════════╡
│"2007 - 5"│9        │3               │3                │
├──────────┼─────────┼────────────────┼─────────────────┤
│"2007 - 7"│6        │2               │2                │
└──────────┴─────────┴────────────────┴─────────────────┘
```

#### 4. Summarize soil issue frequency across years
 
```sql
UNWIND [2007,2008, 2009, 2010, 2011, 2012, 2013, 2014]  as years
MATCH (y:Year {year: years})-[:HAS_MONTH]->(m:Month )-[:HAS_DAY]->(d:Day)<-[r:REPORTED_ON]-()-[*1..2]-(n:Soil_Issue) 
WITH y.year as Y,  collect( n.type) as Issues
RETURN Y, 
size(filter(x IN Issues WHERE x= 'Acidification')) as Acidification,
size(filter(x IN Issues WHERE x= 'Compaction')) as Compaction,
size(filter(x IN Issues WHERE x= 'Erosion')) as Erosion,
size(filter(x IN Issues WHERE x= 'HeavyMetalContamination')) as HeavyMetalContamination,
size(filter(x IN Issues WHERE x= 'HighAlkalinity')) as HighAlkalinity,
size(filter(x IN Issues WHERE x= 'Impermeable')) as Impermeable,
size(filter(x IN Issues WHERE x= 'LowNitrogen')) as LowNitrogen,
size(filter(x IN Issues WHERE x= 'LowOrganicBiota')) as LowOrganicBiota,
size(filter(x IN Issues WHERE x= 'LowOrganicMatter')) as LowOrganicMatter,
size(filter(x IN Issues WHERE x= 'LowPhosphorus')) as LowPhosphorus,
size(filter(x IN Issues WHERE x= 'LowPotassium')) as LowPotassium,
size(filter(x IN Issues WHERE x= 'Salinity')) as Salinity
ORDER By Y
```
__Output:__ 

 ```bash
╒════╤═══════════════╤════════════╤═════════╤═════════════════════════╤════════════════╤═════════════╤═════════════╤═════════════════╤══════════════════╤═══════════════╤══════════════╤══════════╕
│"Y" │"Acidification"│"Compaction"│"Erosion"│"HeavyMetalContamination"│"HighAlkalinity"│"Impermeable"│"LowNitrogen"│"LowOrganicBiota"│"LowOrganicMatter"│"LowPhosphorus"│"LowPotassium"│"Salinity"│
╞════╪═══════════════╪════════════╪═════════╪═════════════════════════╪════════════════╪═════════════╪═════════════╪═════════════════╪══════════════════╪═══════════════╪══════════════╪══════════╡
│2007│31             │59          │124      │1                        │58              │4            │1            │44               │29                │16             │0             │18        │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2008│133            │472         │773      │15                       │462             │18           │23           │276              │225               │89             │1             │72        │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2009│110            │498         │755      │16                       │439             │30           │54           │327              │237               │112            │0             │74        │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2010│112            │578         │982      │43                       │475             │60           │61           │344              │237               │149            │5             │99        │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2011│121            │477         │769      │34                       │401             │20           │67           │252              │226               │196            │27            │75        │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2012│125            │659         │1139     │33                       │512             │56           │92           │426              │293               │273            │52            │121       │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2013│113            │462         │831      │33                       │386             │50           │13           │247              │192               │175            │43            │171       │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2014│29             │24          │73       │2                        │51              │1            │11           │14               │12                │45             │4             │1         │
└────┴───────────────┴────────────┴─────────┴─────────────────────────┴────────────────┴─────────────┴─────────────┴─────────────────┴──────────────────┴───────────────┴──────────────┴──────────┘
```


#### 5. Summarize soil issue frequency across months of a specific year, e.g. 2007
 
```sql
UNWIND [2007]  as years
MATCH (y:Year {year: years})-[:HAS_MONTH]->(m:Month )-[:HAS_DAY]->(d:Day)<-[r:REPORTED_ON]-()-[*1..2]-(n:Soil_Issue) 
WITH y.year as Y,  m.month as M, collect( n.type) as Issues
RETURN Y, M,
size(filter(x IN Issues WHERE x= 'Acidification')) as Acidification,
size(filter(x IN Issues WHERE x= 'Compaction')) as Compaction,
size(filter(x IN Issues WHERE x= 'Erosion')) as Erosion,
size(filter(x IN Issues WHERE x= 'HeavyMetalContamination')) as HeavyMetalContamination,
size(filter(x IN Issues WHERE x= 'HighAlkalinity')) as HighAlkalinity,
size(filter(x IN Issues WHERE x= 'Impermeable')) as Impermeable,
size(filter(x IN Issues WHERE x= 'LowNitrogen')) as LowNitrogen,
size(filter(x IN Issues WHERE x= 'LowOrganicBiota')) as LowOrganicBiota,
size(filter(x IN Issues WHERE x= 'LowOrganicMatter')) as LowOrganicMatter,
size(filter(x IN Issues WHERE x= 'LowPhosphorus')) as LowPhosphorus,
size(filter(x IN Issues WHERE x= 'LowPotassium')) as LowPotassium,
size(filter(x IN Issues WHERE x= 'Salinity')) as Salinity
ORDER By Y, M
```
__Output:__ 

 ```bash
╒════╤═══════════════╤════════════╤═════════╤═════════════════════════╤════════════════╤═════════════╤═════════════╤═════════════════╤══════════════════╤═══════════════╤══════════════╤══════════╕
│"Y" │"Acidification"│"Compaction"│"Erosion"│"HeavyMetalContamination"│"HighAlkalinity"│"Impermeable"│"LowNitrogen"│"LowOrganicBiota"│"LowOrganicMatter"│"LowPhosphorus"│"LowPotassium"│"Salinity"│
╞════╪═══════════════╪════════════╪═════════╪═════════════════════════╪════════════════╪═════════════╪═════════════╪═════════════════╪══════════════════╪═══════════════╪══════════════╪══════════╡
│2007│31             │59          │124      │1                        │58              │4            │1            │44               │29                │16             │0             │18        │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2008│133            │472         │773      │15                       │462             │18           │23           │276              │225               │89             │1             │72        │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2009│110            │498         │755      │16                       │439             │30           │54           │327              │237               │112            │0             │74        │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2010│112            │578         │982      │43                       │475             │60           │61           │344              │237               │149            │5             │99        │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2011│121            │477         │769      │34                       │401             │20           │67           │252              │226               │196            │27            │75        │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2012│125            │659         │1139     │33                       │512             │56           │92           │426              │293               │273            │52            │121       │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2013│113            │462         │831      │33                       │386             │50           │13           │247              │192               │175            │43            │171       │
├────┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2014│29             │24          │73       │2                        │51              │1            │11           │14               │12                │45             │4             │1         │
└────┴───────────────┴────────────┴─────────┴─────────────────────────┴────────────────┴─────────────┴─────────────┴─────────────────┴──────────────────┴───────────────┴──────────────┴──────────┘
```

#### 6. Summarize soil issue frequency across months of the year
 
```sql
UNWIND [2007]  as years
MATCH (y:Year {year: years})-[:HAS_MONTH]->(m:Month )-[:HAS_DAY]->(d:Day)<-[r:REPORTED_ON]-()-[*1..2]-(n:Soil_Issue) 
WITH y.year as Y,  m.month as M, collect( n.type) as Issues
RETURN Y, M,
size(filter(x IN Issues WHERE x= 'Acidification')) as Acidification,
size(filter(x IN Issues WHERE x= 'Compaction')) as Compaction,
size(filter(x IN Issues WHERE x= 'Erosion')) as Erosion,
size(filter(x IN Issues WHERE x= 'HeavyMetalContamination')) as HeavyMetalContamination,
size(filter(x IN Issues WHERE x= 'HighAlkalinity')) as HighAlkalinity,
size(filter(x IN Issues WHERE x= 'Impermeable')) as Impermeable,
size(filter(x IN Issues WHERE x= 'LowNitrogen')) as LowNitrogen,
size(filter(x IN Issues WHERE x= 'LowOrganicBiota')) as LowOrganicBiota,
size(filter(x IN Issues WHERE x= 'LowOrganicMatter')) as LowOrganicMatter,
size(filter(x IN Issues WHERE x= 'LowPhosphorus')) as LowPhosphorus,
size(filter(x IN Issues WHERE x= 'LowPotassium')) as LowPotassium,
size(filter(x IN Issues WHERE x= 'Salinity')) as Salinity
ORDER By Y, M
```
__Output:__ 

 ```bash
 UNWIND [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]  as months
MATCH (y:Year )-[:HAS_MONTH]->(m:Month {month: months})-[:HAS_DAY]->(d:Day)<-[r:REPORTED_ON]-()-[*1..2]-(n:Soil_Issue) 
WITH m.month as M,  collect( n.type) as Issues //, collect(DISTINCT n.type) as I_set
RETURN M, 
size(filter(x IN Issues WHERE x= 'Acidification')) as Acidification,
size(filter(x IN Issues WHERE x= 'Compaction')) as Compaction,
size(filter(x IN Issues WHERE x= 'Erosion')) as Erosion,
size(filter(x IN Issues WHERE x= 'HeavyMetalContamination')) as HeavyMetalContamination,
size(filter(x IN Issues WHERE x= 'HighAlkalinity')) as HighAlkalinity,
size(filter(x IN Issues WHERE x= 'Impermeable')) as Impermeable,
size(filter(x IN Issues WHERE x= 'LowNitrogen')) as LowNitrogen,
size(filter(x IN Issues WHERE x= 'LowOrganicBiota')) as LowOrganicBiota,
size(filter(x IN Issues WHERE x= 'LowOrganicMatter')) as LowOrganicMatter,
size(filter(x IN Issues WHERE x= 'LowPhosphorus')) as LowPhosphorus,
size(filter(x IN Issues WHERE x= 'LowPotassium')) as LowPotassium,
size(filter(x IN Issues WHERE x= 'Salinity')) as Salinity
ORDER By M
```

__PS:__ 

1. BTW, updated graph schema :)

![New graph database schema](/assets/images/soil_survey_meta_graph_II.png)
 
 
---
***We stepped through the process of binding an existing `Soil_Report` node to day nodes that are part of the previously implemented time tree - we are ready to refactor the remaining Soil Report nodes***{: style="color: green"}

---
[Back to top of page](#)

---


