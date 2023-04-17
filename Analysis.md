# ACNH Villager Popularity: SQL Analysis

## Table of Contents

 - Goal 1A: Are there more male or female villagers?
 - Goals 1B: Which villager gender is more popular?
 - Goal 2A: How many villagers are there for each personality?
 - Goal 2B: Which personality type is the most/least popular?
 - Goal 3A: How many villagers are there per animal type?
 - Goal 3B: What is the most/least popular animal type?
 - Goal 4A: What is the most/least common zodiac sign of the villagers?
 - Goal 4B: Which zodiac sign is most/least likely to be popular?
 - Goal 5A: *What is the most/least popular villager* by gender, personality, animal, and zodiac sign?
 - Goal 5B: *What is the most/least popular combination* of gender, personality, animal, and zodiac sign?

## Goal 1A: Are there more male or female villagers?

```
-- Are there more male or female villagers?  
SELECT gender, COUNT(i.villager_name) AS villager_count,  
FROM  `ACNH_Villager_Project.VillagerInfo`  AS i  
JOIN  `ACNH_Villager_Project.VillagerPopularity`  AS p  
ON i.villager_name = p.name  
GROUP  BY gender  
ORDER  BY  COUNT(i.villager_name) DESC;
```

## Goals 1B: Which villager gender is more popular?
```
SELECT gender, ROUND(AVG(total_rank), 0) as avg_rank  
FROM  `ACNH_Villager_Project.VillagerInfo`  AS i  
JOIN  `ACNH_Villager_Project.VillagerPopularity`  AS p  
ON i.villager_name = p.name  
GROUP  BY gender  
ORDER  BY avg_rank DESC;
```
