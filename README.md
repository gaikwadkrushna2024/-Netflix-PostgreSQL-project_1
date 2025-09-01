# -Netflix-PostgreSQL-project_1

![Netflix Logo](https://github.com/gaikwadkrushna2024/-Netflix-PostgreSQL-project_1/blob/main/BrandAssets_Logos_01-Wordmark.jpg)
This project analyzes the Netflix dataset using PostgreSQL to find the top 10 actors who’ve appeared in the most Indian-produced movies. It uses UNNEST(), STRING_TO_ARRAY(), and TRIM() to clean and split cast data, then aggregates results with a CTE for clarity. A great example of practical SQL for entertainment data analysis.
## Objective
To analyze the Netflix dataset using PostgreSQL and identify the top 10 actors who have appeared in the most movies produced in India. This project demonstrates practical SQL techniques for handling multi-value fields, cleaning text data, and performing aggregation using Common Table Expressions (CTEs).
## Schema
CREATE TABLE netflis
(
	show_id	VARCHAR(5),
	type    VARCHAR(10),
	title	VARCHAR(250),
	director VARCHAR(550),
	casts	VARCHAR(1050),
	country	VARCHAR(550),
	date_added	VARCHAR(55),
	release_year	INT,
	rating	VARCHAR(15),
	duration	VARCHAR(15),
	listed_in	VARCHAR(250),
	description VARCHAR(550)
);

SELECT * FROM netflis;

## Business Problems and Solutions

select * from netflis

alter table netflis
rename column types_1 to type

### 1. Count the Number of Movies vs TV Shows

select types_1 ,count(*) from netflis
group by 1

### 2 Find the Most Common Rating for Movies and TV Shows

with RATING_COUNT AS (
select type,
       rating , 
	   count(*) AS reting_1
	   from netflis
	   group by 1,2 
	   ),

 KKK AS (
SELECT TYPE ,
       RATING,
	   reting_1,
	   RANK()OVER(PARTITION BY TYPE ORDER BY reting_1 desc) AS RANK_1 
	   from rating_count
	   )
SELECT TYPE ,
       RATING ,
	   RANK_1 FROM  kkk
	   WHERE RANK_1 = 1 

### 3 List All Movies Released in a Specific Year (e.g., 2020)


select * from netflis
where type = 'Movie' AND release_year= 2020



### 4 Find the Top 5 Countries with the Most Content on Netflix

select unnest(string_to_array(country , ',')) as country,
count(*) as total_country
from netflis
group by 1 
order by 2 desc
limit 5


### 5. Identify the Longest Movie 


select * from netflis
where type = 'Movie' and duration is not null
order by split_part(duration , ',' , 1) desc
limit 5

	   
### 6. Find Content Added in the Last 5 Years

select * from netflis
where to_date(date_added ,'month dd ,yyyy') >= current_date - interval '5 year'



### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

select * from netflis
where director like '%Rajiv' and (type = 'Movie' OR  type = 'TV_Show')

with vvv as (
select * ,
        unnest(string_to_array(director , ',' )) as director_1
		from netflis
) select * from vvv
where director_1 = 'Rajiv Chilaka'  and  (type = 'Movie' OR  type = 'TV Show')

### or 

WITH vvv AS (
  SELECT *,
         unnest(string_to_array(director, ',')) AS one_director
  FROM netflis
)
SELECT *
FROM vvv
WHERE one_director = 'Rajiv Chilaka'
  AND type IN ('Movie', 'TV Show');


### 8. List All TV Shows with More Than 5 Seasons

select * from netflis
where type = 'TV Show' and split_part(duration , ' ' , 1):: int >5


### 9. Count the Number of Content Items in Each Genre
select 
unnest(string_to_array(listeds_in,',' )) as genre, count(*)     
from netflis
group by 1
order by 2 desc

### 10.Find each year and the average numbers of content release in India on netflix.
with lll as (
select unnest(string_to_array(country, ',')) as country_1 , release_year ,count(*) as re
from netflis
group by 1 ,2 
order by 3 desc
)
select country_1 , release_year ,  round(avg(re)::numeric , 2 )as ee  from lll 
where country_1 = 'India'
group by 1,2
order by 3 desc
limit 5 


### 11 Get year-wise release count for India and calculate average
WITH lll AS (
  SELECT 
    TRIM(unnest(string_to_array(country, ','))) AS country_1,
    release_year,
    COUNT(*) AS re
  FROM netflis
  GROUP BY 1, 2
)
SELECT 
  release_year,
  SUM(re) AS total_releases,
  ROUND(AVG(re)::numeric, 2) AS avg_releases_per_year
FROM lll
WHERE country_1 = 'India'
GROUP BY release_year
ORDER BY release_year desc



### 12. List All Movies that are Documentaries
select * from netflis

with ttt as (
select TRIM(unnest(string_to_array(listeds_in, ','))) AS list ,
count(*) as total from netflis
group by  1
)select * from ttt
where list = 'Documentaries'

### or 
SELECT list,
       COUNT(*) AS total
FROM (
  SELECT TRIM(unnest(string_to_array(listeds_in, ','))) AS list
  FROM netflis
) AS genre_data
GROUP BY list
HAVING list = 'Documentaries';

select * from netflis
where listeds_in = 'Documentaries'



### 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
select * from netflis


SELECT *
FROM netflis
WHERE casts LIKE '%Salman Khan%'
  AND release_year >= 2016;

### or

SELECT * FROM netflis
WHERE 
	casts LIKE '%Salman Khan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10

### 14. Find all content without a director

select * from netflis

select * from netflis
where director is null


### 15. Find the top 10 actors who have appeared in the highest number of movies produced in India.



SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actor,
	COUNT(*)
FROM netflis
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10

### or

WITH exploded_casts AS (
  SELECT 
    TRIM(actor_name) AS actor
  FROM netflix,
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor_name
  WHERE country = 'India'
)

SELECT actor, COUNT(*) AS movie_count
FROM exploded_casts
GROUP BY actor
ORDER BY movie_count DESC
LIMIT 10;




  
