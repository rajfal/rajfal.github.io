---
layout: post
title: "Find the most frequently used words in a text file with sed"
comments: false
description: "step by step guide on generating a word frequency list"
categories: words file frequency sedL
keywords: "sed, word frequency, text file, frequency"
---

> #### *Prerequisites:*{: style="color: red"}
> - a test file, e.g. test.txt
> - an existing database schema where you can, or already have combined data from tables to obtain a denormalized dataset

---

Quite often I need to analyze a block of text to find the most frequently occuring words. I found sed command as the perfect workhorse to do all the grunt work for me. Effectively, the ultimate command is a series chained pipes feeding output from one task to another.

https://williamjturkel.net/2013/06/15/basic-text-analysis-with-command-line-tools-in-linux/

https://stackoverflow.com/questions/10552803/how-to-create-a-frequency-list-of-every-word-in-a-file

https://superuser.com/questions/661661/listing-all-words-in-a-text-file-and-finding-the-most-frequent-word

https://stackoverflow.com/questions/33055663/removing-stopwords-from-text-corpus-using-linux-commandline


This is the magic recipe:
```bash
sed -e 's/[^[:alpha:]]/ /g' test.txt | tr '\n' " " | tr -s " " | tr " " '\n' | sed '/^.$/d' | tr 'A-Z' 'a-z' | sort | uniq -c | sort -nr | nl | head -n 5

```

#### Step by step insight into how this single command line works:

1. Dump selected table to a CSV text file
```sql
SELECT * FROM soil_survey INTO OUTFILE '/var/lib/mysql-files/soil_survey.csv' FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';
```
Also:  
  : - ensure fields are separated with a comma ','  
  : - database will write the file to a location requiring a root access, such as sudo, in order to move it to another location, such as your data-import-directory/  
  : - resulting CSV will NOT have any headers included, these will be added next

2. Move CSV file to data-import-directory/
```bash
sudo mv /var/lib/mysql-files/soil_survey.csv data-import-directory/
```

3. Insert 1 file headers line at the top, and save the CSV file
```bash
sed -i '1i Hort_Client,Contractor,Region,Locality,Soil_Service,Solution,Soil_Issue,Date_Reported,Date_Actioned,DaysToAction' data-import-directory/soil_survey.csv
```

4. If you make a mistake, delete the added line 
```bash
sed -i '1d' import-directory/soil_survey.csv
```

5. Preview first 3 lines
```bash
head -3  import-directory/soil_survey.csv
Hort_Client,Contractor,Region,Locality,Soil_Service,Solution,Soil_Issue,Date_Reported,Date_Actioned,DaysToAction
159,1091,Northbury,3656,54593,5397,Erosion,2007-05-07,2008-02-18,287
159,1091,Northbury,1516,22644,5397,Erosion,2007-05-07,2008-03-18,316
```

---
***You now have a workable CSV data file that you can import into a Neo4j graph***{: style="color: green"}

---
[Back to top of page](#)
