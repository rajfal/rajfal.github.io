---
layout: post
title: "The Vetiver Guide"
comments: true
published: true
description: "Guid to using vetiver essential oil to support ADD/ADHD, hormonal balance and skin care issues"
categories: [essential oil, vetiver, nervous system, skin, hormones]
tags: [essential oils, vetiver, ADD, skin, hormones]
keywords: "essential oils, vetiver, ADD, skin, hormones, aromatherapy"
---

> #### *This is a writing sample for health-related publications:*{: style="color: red"}
> - 
---

#### Historical, botanical and production background

Vetiver is a member of the Graminae grasses, historically harvested in India and Sri Lanka. Its specific scientific name is Vetiveria zizanioides. It is commonly known as ‘khus khus’, not to be confused with ‘couscous’.

Vetiver is a perennial grass that can reach up to five feet in height. The plant readily populates riverbanks and requires hot and humid environment to thrive. It can grow across the entire soil pH spectrum, displaying no preference for any, as long as it has full access to sun. Young plants can become stunted in the shade. This grass is characterised by the presence of extensive and deep root systems.

It is the undergrount plant parts, roots and rootless, that are the source of highly sought after oil since antiquity.  The vetiver oil is a complex essential oils, composed of several hundred of sesquiterpene derivatives that provide the benefits captured in perfume making, health restoration as well as an insect repellant.

Oil extraction begins with harvesting of 15-18 month old roots, usually around December and January. The roots are washed, sometimes dried, and subjected to steam distillation where they will yield a half to two percent of oil per weight of plant material.

[Vetiver Growing Information](http://greenharvest.com.au/Plants/Information/Vetiver.html)


#### Common application methods and uses:

1. Topical use

__Output:__
    
The essential oil can be applied directly to the area of concern. Unlike some essential oils such as cinnamon or oregano, vetiver oil can be used neat. It requires no further dilution in a carrier oil. 

[The in vitro antimicrobial evaluation of commercially essential oils and their combinations against acne.](https://www.ncbi.nlm.nih.gov/pubmed/29574906)

Vetiver has a rejuvenating effect on skin, repairing the effect of dehydration, cuts, wounds, skin irritations and inflammations. It has been reported as a solution for persistent lower back pain (lumbago) and rheumatism.

2. Aromatic use

As an aromatic medium, the oil can be diffused through a diffuser or its aroma inhaled directly. Diffuser is helpful if you wish to have the oil droplets dispersed during your sleep at night. Whereas should you be needing the grounding effect of vetiver during daytime, then perhaps carry a small 2-5ml bottle of the oil with you to inhale it directly. Vetiver assists in overcoming depression, insomnia, inability to enter deep sleep, anxiety, stress or tension.

[Modification of sleep-waking and electroencephalogram induced by vetiver essential oil inhalation.](https://www.ncbi.nlm.nih.gov/pubmed/2706972)
 
[Odors enhance slow-wave activity in non-rapid eye movement sleep.](https://www.ncbi.nlm.nih.gov/pubmed/26888107)

3. Oral use

Vetiver can be used as a dietary supplement, either in form of capsules, bought or home-made, or  mixed with honey and added to a beverage like milk or soy. As an infusion it can assist with fever, inflammation and irritability of the stomach

[Other Uses, and Utilization of Vetiver: Vetiver Oil](http://naturalingredient.org/wp/wp-content/uploads/vetiver.pdf)









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
 ╒════╤═══╤═══════════════╤════════════╤═════════╤═════════════════════════╤════════════════╤═════════════╤═════════════╤═════════════════╤══════════════════╤═══════════════╤══════════════╤══════════╕
│"Y" │"M"│"Acidification"│"Compaction"│"Erosion"│"HeavyMetalContamination"│"HighAlkalinity"│"Impermeable"│"LowNitrogen"│"LowOrganicBiota"│"LowOrganicMatter"│"LowPhosphorus"│"LowPotassium"│"Salinity"│
╞════╪═══╪═══════════════╪════════════╪═════════╪═════════════════════════╪════════════════╪═════════════╪═════════════╪═════════════════╪══════════════════╪═══════════════╪══════════════╪══════════╡
│2007│5  │0              │0           │9        │0                        │3               │0            │0            │3                │0                 │0              │0             │0         │
├────┼───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2007│6  │6              │1           │3        │0                        │3               │0            │0            │1                │3                 │2              │0             │0         │
├────┼───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2007│7  │0              │0           │6        │0                        │2               │0            │0            │2                │0                 │0              │0             │0         │
├────┼───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2007│8  │0              │2           │3        │0                        │1               │0            │0            │2                │0                 │1              │0             │0         │
├────┼───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2007│9  │0              │3           │11       │0                        │5               │0            │0            │3                │9                 │2              │0             │0         │
├────┼───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2007│10 │0              │17          │20       │1                        │4               │1            │0            │5                │3                 │1              │0             │1         │
├────┼───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2007│11 │13             │9           │21       │0                        │24              │1            │1            │17               │3                 │3              │0             │8         │
├────┼───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2007│12 │12             │27          │51       │0                        │16              │2            │0            │11               │11                │7              │0             │9         │
└────┴───┴───────────────┴────────────┴─────────┴─────────────────────────┴────────────────┴─────────────┴─────────────┴─────────────────┴──────────────────┴───────────────┴──────────────┴──────────┘
```

#### 6. Summarize soil issue frequency across months of the year
 
```sql
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
__Output:__ 

 ```bash
╒═══╤═══════════════╤════════════╤═════════╤═════════════════════════╤════════════════╤═════════════╤═════════════╤═════════════════╤══════════════════╤═══════════════╤══════════════╤══════════╕
│"M"│"Acidification"│"Compaction"│"Erosion"│"HeavyMetalContamination"│"HighAlkalinity"│"Impermeable"│"LowNitrogen"│"LowOrganicBiota"│"LowOrganicMatter"│"LowPhosphorus"│"LowPotassium"│"Salinity"│
╞═══╪═══════════════╪════════════╪═════════╪═════════════════════════╪════════════════╪═════════════╪═════════════╪═════════════════╪══════════════════╪═══════════════╪══════════════╪══════════╡
│1  │55             │233         │445      │11                       │218             │40           │19           │183              │87                │94             │12            │93        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│2  │58             │220         │391      │20                       │203             │9            │17           │107              │125               │85             │8             │51        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│3  │67             │256         │435      │17                       │228             │20           │19           │157              │95                │105            │14            │41        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│4  │25             │233         │385      │10                       │231             │16           │50           │171              │129               │74             │4             │22        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│5  │109            │255         │420      │16                       │234             │28           │15           │134              │75                │88             │12            │65        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│6  │56             │383         │573      │38                       │284             │4            │52           │238              │179               │126            │24            │42        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│7  │62             │186         │350      │5                        │163             │29           │17           │131              │98                │57             │11            │45        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│8  │59             │207         │395      │0                        │159             │18           │13           │98               │73                │68             │0             │36        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│9  │118            │240         │472      │12                       │265             │12           │2            │140              │122               │89             │9             │58        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│10 │51             │289         │496      │35                       │260             │13           │58           │184              │125               │106            │12            │47        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│11 │45             │292         │428      │2                        │238             │22           │32           │167              │129               │97             │23            │50        │
├───┼───────────────┼────────────┼─────────┼─────────────────────────┼────────────────┼─────────────┼─────────────┼─────────────────┼──────────────────┼───────────────┼──────────────┼──────────┤
│12 │69             │435         │656      │11                       │301             │28           │28           │220              │214               │66             │3             │81        │
└───┴───────────────┴────────────┴─────────┴─────────────────────────┴────────────────┴─────────────┴─────────────┴─────────────────┴──────────────────┴───────────────┴──────────────┴──────────┘
```

---
***We used the time tree to get granular results related to Soil Issues occurring in various time periods. Specific Cypher language features that yielded these results included UNWIND clause and list functions like collect(), filter() and size()***{: style="color: green"}

---
[Back to top of page](#)

---

