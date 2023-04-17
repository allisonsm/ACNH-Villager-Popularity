
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
