
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
```
-- Updating Renée in Popularity table  
UPDATE  `ACNH_Villager_Project.VillagerPopularity`  
SET  name = "Renée"  
WHERE  name = "Renee";
```
```
-- Searching for duplicate "O'Hare"  
SELECT  name, villager_name, tier, rank, animal  
FROM  `ACNH_Villager_Project.VillagerPopularity`  AS p  
FULL  OUTER  JOIN  `ACNH_Villager_Project.VillagerInfo`  AS i  
ON p.name = i.villager_name  
WHERE villager_name LIKE  "%Hare%"  
OR  name  LIKE  "%Hare%"  
ORDER  BY  name  ASC;
```
 ```
-- Updating O'Hare in Popularity table  
UPDATE  `ACNH_Villager_Project.VillagerPopularity`  
SET  name = "O'Hare"  
WHERE  name = "OHare";
```
```
-- Searching for "Wart Jr." duplicate  
SELECT  name, villager_name, tier, rank, animal  
FROM  `ACNH_Villager_Project.VillagerPopularity`  AS p  
FULL  OUTER  JOIN  `ACNH_Villager_Project.VillagerInfo`  AS i  
ON p.name = i.villager_name  
WHERE villager_name LIKE  "%Wart%"  
OR  name  LIKE  "%Wart%"  
ORDER  BY  name  ASC;
```
```
-- Updating Wart Jr. in Popularity table  
UPDATE  `ACNH_Villager_Project.VillagerPopularity`  
SET  name = "Wart Jr."  
WHERE  name = "WartJr";
```
There were **5 records** in the VillagerInfo table that had a match in the VillagerPopularity table that weren’t recognized due to differences in the data input. Two of our villagers (Buck and Spork, had nicknames in the VillagerPopularity table. Three of our villagers (Renée, O’Hare, and Wart Jr.) were in the VillagerPopularity table, but without the appropriate symbols and spelling in their names. After updating the VillagerPopularity table, there are **22 records with NULL values** in our VillagerInfo table. For the purposes of this project, I will *ignore* these villagers going forward as there is insufficient data.
```
SELECT  name, villager_name, tier, rank, animal  
FROM  `ACNH_Villager_Project.VillagerPopularity`  AS p  
INNER  JOIN  `ACNH_Villager_Project.VillagerInfo`  AS i  
ON p.name = i.villager_name  
ORDER  BY  name  ASC;  
--Since we are running an INNER JOIN, any records with NULL values in the VillagerINFO table are not included
```
## 1.4 Cleaning: Transforming Data - Birthday Date
```
ALTER  TABLE  `ACNH_Villager_Project.VillagerInfo`  
ADD  COLUMN birthday_day STRING;
```
```
ALTER  TABLE  `ACNH_Villager_Project.VillagerInfo`  
ADD  COLUMN birthday_month STRING;
```
```
UPDATE  `ACNH_Villager_Project.VillagerInfo`  
SET birthday_day = TRIM(SUBSTRING(birthday, 1, 2), '-')  
WHERE birthday IS  NOT  NULL;  
```
```
UPDATE  `ACNH_Villager_Project.VillagerInfo`  
SET birthday_month = TRIM(SUBSTRING(birthday, 3, 4), '-')  
WHERE birthday IS  NOT  NULL;
```
```
--Our birthday_day column has records with days in both D and DD format, to fix:

UPDATE  `ACNH_Villager_Project.VillagerInfo`  
SET birthday_day =  
CASE  
WHEN  LENGTH(birthday_day) = 1  THEN  CONCAT('0', birthday_day)  
ELSE birthday_day  
END  
WHERE birthday IS  NOT  NULL;
```
```
-- Adding a column to turn our month names “MMM” to MM - keeping as a string to learn how to CONCAT two strings as a date
ALTER  TABLE  `ACNH_Villager_Project.VillagerInfo`  
ADD  COLUMN birthday_mm STRING;
```
```
UPDATE  `ACNH_Villager_Project.VillagerInfo`  
SET birthday_mm =  
CASE  
WHEN birthday_month = 'Jan'  THEN  '01'  
WHEN birthday_month = 'Feb'  THEN  '02'  
WHEN birthday_month = 'Mar'  THEN  '03'  
WHEN birthday_month = 'Apr'  THEN  '04'  
WHEN birthday_month = 'May'  THEN  '05'  
WHEN birthday_month = 'Jun'  THEN  '06'  
WHEN birthday_month = 'Jul'  THEN  '07'  
WHEN birthday_month = 'Aug'  THEN  '08'  
WHEN birthday_month = 'Sep'  THEN  '09'  
WHEN birthday_month = 'Oct'  THEN  '10'  
WHEN birthday_month = 'Nov'  THEN  '11'  
WHEN birthday_month = 'Dec'  THEN  '12'  
ELSE  NULL  
END  
WHERE birthday IS  NOT  NULL;
```
```
SELECT DATE(2023, CAST(birthday_mm AS INT64), CAST(birthday_day AS INT64)) AS birthday_date  
FROM  `ACNH_Villager_Project.VillagerInfo`;
```
```
ALTER  TABLE  `ACNH_Villager_Project.VillagerInfo`  
ADD  COLUMN birthday_date DATE;
```
```
UPDATE`ACNH_Villager_Project.VillagerInfo`  
SET birthday_date =  
DATE(2023, CAST(birthday_mm AS INT64), CAST(birthday_day AS INT64))  
WHERE birthday IS  NOT  NULL;
--SUCCESS! Our birthday has now been added as date for us to analyze
```
Below:
 - Creating a column of each villager’s Zodiac sign
 -  At each tier, the rank starts over at 1, so we're creating a column for the total, overall rank of all villagers
```
**WITH VillagerBirthday AS (SELECT villager_name, tier, rank, birthday_date, CAST(birthday_mm AS INT64) AS monthint, CAST(birthday_day AS INT64) AS dayint  
FROM  `ACNH_Villager_Project.VillagerInfo`  as i  
JOIN  `ACNH_Villager_Project.VillagerPopularity`  AS p  
ON i.villager_name = p.name  
ORDER  BY villager_name)  
  
SELECT i.villager_name, b.tier, b.rank, ROW_NUMBER() OVER(ORDER  BY  tier  ASC, rank  ASC) AS total_rank, b.birthday_date,  
CASE  
WHEN (monthint = 03  AND dayint >=21) OR (monthint = 04  AND dayint <=19) THEN  'Aries'  
WHEN (monthint = 4  AND dayint >= 20) OR (monthint = 5  AND dayint <= 20) THEN  'Taurus'  
WHEN (monthint = 5  AND dayint >= 21) OR (monthint = 6  AND dayint <= 20) THEN  'Gemini'  
WHEN (monthint = 6  AND dayint >= 21) OR (monthint = 7  AND dayint <= 20) THEN  'Cancer'  
WHEN (monthint = 7  AND dayint >= 21) OR (monthint = 8  AND dayint <= 20) THEN  'Leo'  
WHEN (monthint = 8  AND dayint >= 21) OR (monthint = 9  AND dayint <= 20) THEN  'Virgo'  
WHEN (monthint = 9  AND dayint >= 21) OR (monthint = 10  AND dayint <= 20) THEN  'Libra'  
WHEN (monthint = 10  AND dayint >= 21) OR (monthint = 11  AND dayint <= 20) THEN  'Scorpio'  
WHEN (monthint = 11  AND dayint >= 21) OR (monthint = 12  AND dayint <= 20) THEN  'Sagittarius'  
WHEN (monthint = 12  AND dayint >= 21) OR (monthint = 1  AND dayint <= 20) THEN  'Capricorn'  
WHEN (monthint = 1  AND dayint >= 21) OR (monthint = 2  AND dayint <= 20) THEN  'Aquarius'  
WHEN (monthint = 2  AND dayint >= 21) OR (monthint = 3  AND dayint <= 20) THEN  'Pisces'  
ELSE  'na'  
END  AS zodiac_sign  
FROM  `ACNH_Villager_Project.VillagerInfo`  AS i  
INNER  JOIN VillagerBirthday AS b  
ON i.villager_name = b.villager_name  
ORDER  BY  tier  ASC, rank  ASC**
```
