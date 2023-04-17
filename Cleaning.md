
## 1.1 Cleaning: Creating Tables and Headers

Load VillagerInfo and VillagerPopularity tables into BigQuery for cleaning and analysis. The VillagerInfo table loaded would not detect the header row. I used a preview spreadsheet to view the column headers, then tried to locate them in SQL:
```
SELECT *  
FROM  `ACNH_Villager_Project.VillagerInfo`  
WHERE villager_name = "name";
```
This returned no results, so I used ALTER TABLE to manually adjust them:
```
ALTER  TABLE  `ACNH_Villager_Project.VillagerInfo`  
RENAME  COLUMN string_field_0 to villager_name;
```
I then ran the ALTER TABLE query for the rest of the column headers. I couldn't find a way to adjust field headers without running the query 11 times. I also used the preview panel in BigQuery to verify each field's content before continuing.
```
String_field_1 to animal;  
string_field_2 to gender;  
string_field_3 to personality;  
string_field_4 to hobby;  
string_field_5 to birthday;  
string_field_6 to catchphrase;  
string_field_7 to favorite_song;  
string_field_8 to style_one;  
string_field_9 to style_two;  
string_field_10 to color_one;  
string_field_11 to color_two;  
string_field_12 to wall;  
string_field_13 to flooring;  
string_field_14 to furniture;  
string_field_15 to filename;  
string_field_16 to uniqueid;
```
## 1.2 Cleaning: Checking for NULL Values

```
SELECT *  
FROM  `ACNH_Villager_Project.VillagerInfo`  
WHERE villager_name IS  NULL  
OR animal IS  NULL  
OR gender IS  NULL  
OR personality IS  NULL  
OR hobby IS  NULL  
OR birthday IS  NULL  
OR catchphrase IS  NULL  
OR favorite_song IS  NULL  
OR style_one IS  NULL  
OR style_two IS  NULL  
OR color_one IS  NULL  
OR color_two IS  NULL  
OR wall IS  NULL  
OR flooring IS  NULL  
OR furniture IS  NULL  
OR filename IS  NULL  
OR uniqueid IS  NULL;  
  
-- Repeat for VillagerPopularity table  
  
SELECT *  
FROM  `ACNH_Villager_Project.VillagerPopularity`  
WHERE  tier  IS  NULL  
OR  rank  IS  NULL  
OR  name  IS  NULL;  
-- Both queries returned no results
```

## 1.3 Cleaning: Duplicates & Villager Count
```
SELECT  COUNT (villager_name)  
FROM  `ACNH_Villager_Project.VillagerInfo`;
```
```
SELECT  COUNT (name)  
FROM  `ACNH_Villager_Project.VillagerPopularity`;
```
There are 391 villagers in the VillagerInfo table, and 413 in the VillagerPopularity table. I investigated what the 22 additional records were. My initial query below returned 27 records with NULL values in the VillagerInfo table. At this point I chose to SELECT the columns I wanted to focus on for this project for future reference.

```
SELECT  name, tier, rank, animal, gender, personality, hobby, birthday, favorite_song  
FROM  `ACNH_Villager_Project.VillagerPopularity`  AS p  
LEFT  JOIN  `ACNH_Villager_Project.VillagerInfo`  AS i  
ON p.name = i.villager_name  
WHERE i.animal IS  NULL;
-- This returned 27 records
```
This showed me there were more inconsistencies between my primary and foreign keys.
```
SELECT  name, COUNT(name) AS  count  
FROM  `ACNH_Villager_Project.VillagerPopularity`  AS p  
LEFT  JOIN  `ACNH_Villager_Project.VillagerInfo`  AS i  
ON p.name = i.villager_name  
GROUP  BY p.name  
HAVING  COUNT(name) > 1  
ORDER  BY  name  ASC;
-- Returned 0 records
```
```
SELECT  name, villager_name, tier, rank, animal  
FROM  `ACNH_Villager_Project.VillagerPopularity`  AS p  
FULL  OUTER  JOIN  `ACNH_Villager_Project.VillagerInfo`  AS i  
ON p.name = i.villager_name  
WHERE villager_name IS  NULL  
OR  name  IS  NULL  
ORDER  BY  name  ASC;  
-- Returned 32 records
-- However, we know there are only 21 more records in the Population table
-- Investigating the first potential match I spotted, below
```
```
SELECT  name, villager_name, tier, rank, animal  
FROM  `ACNH_Villager_Project.VillagerPopularity`  AS p  
FULL  OUTER  JOIN  `ACNH_Villager_Project.VillagerInfo`  AS i  
ON p.name = i.villager_name  
WHERE villager_name LIKE  "%Buck%"  
OR  name  LIKE  "%Buck%"  
ORDER  BY  name  ASC;
-- Returned 2 records, one for “Buck” in the i table, and one for “Buck(Brows)” in the p table
```
I updated the 5 duplicates I found using UPDATE below:
```
UPDATE  `ACNH_Villager_Project.VillagerPopularity`  
SET  name = "Buck"  
WHERE  name = "Buck(Brows)";
```
```
-- Searching for next duplicate  
SELECT  name, villager_name, tier, rank, animal  
FROM  `ACNH_Villager_Project.VillagerPopularity`  AS p  
FULL  OUTER  JOIN  `ACNH_Villager_Project.VillagerInfo`  AS i  
ON p.name = i.villager_name  
WHERE villager_name LIKE  "%Spork%"  
OR  name  LIKE  "%Spork%"  
ORDER  BY  name  ASC;
```
```
-- Updating Popularity table  
UPDATE  `ACNH_Villager_Project.VillagerPopularity`  
SET  name = "Spork"  
WHERE  name = "Crackle(Spork)";
```
 ```
-- Searching for next duplicate, "Renée"  
SELECT  name, villager_name, tier, rank, animal  
FROM  `ACNH_Villager_Project.VillagerPopularity`  AS p  
FULL  OUTER  JOIN  `ACNH_Villager_Project.VillagerInfo`  AS i  
ON p.name = i.villager_name  
WHERE villager_name LIKE  "%Ren%"  
OR  name  LIKE  "%Ren%"  
ORDER  BY  name  ASC;
```
