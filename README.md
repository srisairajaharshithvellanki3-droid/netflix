#netflix movies and tv shows data analysis using postgresql
![Netflixlogo](https://github.com/srisairajaharshithvellanki3-droid/netflix/blob/main/Netflix-Logo.jpg)
##objective
Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

Objectives
Analyze the distribution of content types (movies vs TV shows).
Identify the most common ratings for movies and TV shows.
List and analyze content based on release years, countries, and durations.
Explore and categorize content based on specific criteria and keywords.
--Netflix project--
Drop table IF Exists netflix;
CREATE TABLE netflix
(
    show_id VARCHAR(6),
	type VARCHAR(10),
	title VARCHAR(150),
	director VARCHAR(208),
	casts VARCHAR(1000),
	country VARCHAR(150),
	date_added VARCHAR(150),
	release_year INT,
	rating VARCHAR(10),
	duration VARCHAR(15),
	listed_in VARCHAR(100),
	description VARCHAR(250)
);

select * from netflix;

select
count(*) as total_content
from netflix;

select distinct type from
netflix;

--1 count the number of movies Vs Tv shows
select type,count(*) as total_content
from netflix  GROUP BY type
--2 Find the most common rating for movies and tv shows
select
type,
rating
from
(
select type,
rating,count(*),
       RANK() over(PARTITION BY type ORDER BY COUNT(*)) as ranking
from netflix group by 1,2 
) as t1
where
 ranking = 1

--3 List all movies released in a specific year (e.g.,2020)
select * from netflix
where type ='Movie'
and
release_year = 2020

--4 Find the top 5 countries with the most content on Netflix
select 
  UNNEST (STRING_TO_ARRAY(country, ',')) as new_country,
  Count(show_ID) as total_content
   from
   netflix
  GROUP BY 1
  order by 2 desc
  LIMIT 5
--5 IDentify the longest movie
select * from netflix
where
type ='Movie'
AND
duration =(SELECT MAX(duration) From netflix)
--6 Find content added in the last 5 years
Select *
from Netflix
where TO_DATE(date_added,'Month DD, YYYY')>= CURRENT_DATE - INTERVAL '5 years'
--7  Find all the movies/Tv shows by director 'BRUNO Garrotti'
   SELECT * FROM netflix
   where director ILIKE '%Rajiv chilaka%'
--8 List all TV shows with more than 5 seasons
SELECT *
FROM netflix
WHERE 
	TYPE = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1)::INT > 5
--9 count the number of content items in each genre
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(*) as total_content
FROM netflix
GROUP BY 1
--10 Find the top 10 actors who have appeared in the highest number of movies produced in India.

SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actor,
	COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
--Find each year and the average numbers of content release by India on netflix. 
-- return top 5 year with highest avg content release !
SELECT 
	country,
	release_year,
	COUNT(show_id) as total_release,
	ROUND(
		COUNT(show_id)::numeric/
								(SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100 
		,2
		)
		as avg_release
FROM netflix
WHERE country = 'India' 
GROUP BY country, 2
ORDER BY avg_release DESC 
LIMIT 5
--List all movies that are documentaries
SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries'
-- Find all content without a director
SELECT * FROM netflix
WHERE director IS NULL
-- 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

SELECT * FROM netflix
WHERE 
	casts LIKE '%Salman Khan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
- 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.



SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actor,
	COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10


/*Question 15:
Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.*/



SELECT 
    category,
	TYPE,
    COUNT(*) AS content_count
FROM (
    SELECT 
		*,
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY 1,2
ORDER BY 2
