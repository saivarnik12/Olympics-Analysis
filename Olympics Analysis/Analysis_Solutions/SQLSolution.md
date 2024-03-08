## 🤩 Queries

### 1. Write a query to get the teams that has won the maximum gold medals over each olympic year.

````sql
WITH gold_medals AS 
(
SELECT E.year,A.team,COUNT(*) AS no_of_gold_medals
FROM athlete_events E INNER JOIN athletes A
ON E.athlete_id= A.id
WHERE medal = 'Gold'
GROUP BY E.year,A.team)
SELECT year, team, no_of_gold_medals
FROM 
(SELECT *,
DENSE_RANK() OVER (PARTITION BY year ORDER BY no_of_gold_medals DESC) AS Rankings
FROM gold_medals) AS Rank_desc
WHERE Rankings=1; 
````

### 2. Write a query to get each teams silver medal counts and the years in which they won the maximum silver medals.


````sql
WITH Rankings AS (
SELECT A.team,E.year , COUNT(DISTINCT event) as silver_medals
,RANK() OVER(PARTITION BY team ORDER BY COUNT(DISTINCT event) DESC) AS ranks
FROM athlete_events E
INNER JOIN athletes A on E.athlete_id=A.id
WHERE medal='Silver'
GROUP BY A.team,E.year)
SELECT team,
SUM(silver_medals) as Total_Silver_Won, 
MAX(CASE WHEN ranks=1 THEN year END) as  Max_Silver_Year
FROM Rankings
GROUP BY team;
````

### 3. Write a query to get the players name who have won the maximum gold medals but never won any other medal.

````sql
WITH Data1 AS
(
SELECT A.name AS Player_Name, E.medal
FROM athlete_events E INNER JOIN athletes A
ON E.athlete_id = A.id
)
SELECT TOP 1 Player_Name, COUNT(*) AS No_of_gold_medals
FROM Data1
WHERE Player_Name NOT IN 
(SELECT Player_Name FROM Data1 WHERE medal in ('Silver','Bronze'))
and medal = 'Gold'
GROUP BY Player_Name
ORDER BY 2 DESC;
````

### 4. Write a query to get each year's maximum gold medal winning player.

````sql
WITH CTE1 AS
(
SELECT E.Year AS Year, 
A.name AS Player_Name, COUNT(*) AS no_of_gold_medals,
DENSE_RANK() OVER (PARTITION BY E.Year ORDER BY COUNT(*) DESC) AS Rankings
FROM athlete_events E INNER JOIN athletes A
ON E.athlete_id = A.id
WHERE medal='Gold'
GROUP BY E.Year, A.name
)
SELECT Year, no_of_gold_medals, 
STRING_AGG(Player_Name,' , ') AS Players
FROM CTE1
WHERE Rankings=1
GROUP BY Year,no_of_gold_medals
ORDER BY 1;
````

### 5. Write a query to get the years and the event names in which India won it's first gold, silver and bronze medal.

````sql
WITH Medal_details AS (
SELECT E.Year AS Year, 
E.event AS Event_Name, E.Medal AS Medal_Type, 
RANK() OVER (PARTITION BY E.medal ORDER BY year) AS Rankings
FROM athlete_events E INNER JOIN athletes A
ON E.athlete_id = A.id
WHERE A.team='India' and E.medal IN ('Gold','Silver','Bronze')
GROUP BY E.Year,E.event,E.Medal)
SELECT Year, Event_Name, Medal_Type
FROM Medal_details
WHERE Rankings=1;
````

### 6. Write a query to get the players name who won gold medals in both summer and winter olympics.

````sql
SELECT A.name AS Player_Name
FROM athlete_events E INNER JOIN athletes A
ON E.athlete_id = A.id
WHERE medal='Gold'
GROUP BY A.name
HAVING COUNT(DISTINCT season)=2;
````

### 7. Write a query to get the players name who won all the medals in a single olympic year.

````sql
SELECT E.year, A.name
FROM athlete_events E INNER JOIN athletes A
ON E.athlete_id = A.id
WHERE medal NOT IN ('NA')
GROUP BY E.year, A.name
HAVING COUNT(DISTINCT medal)=3
ORDER BY 1;
````

### 8. Write a query to get the players name who have won gold medal in the same event for three consecutive years. Starting from the Year 2000.

````sql
WITH Player_Details AS
(
SELECT DISTINCT E.year, name, event
FROM athlete_events E INNER JOIN athletes A
ON E.athlete_id = A.id
WHERE year>=2000 and medal='Gold' and season='Summer'
)
SELECT * 
FROM(
SELECT *, 
LAG(year,1) OVER (PARTITION BY name,event ORDER BY year) AS Prev_Year,
LEAD(year,1) OVER (PARTITION BY name,event ORDER BY year) AS Next_Year
FROM Player_Details) AS Years_Details
WHERE year=Prev_Year+4 and year=Next_Year-4
````

### And that's it! 😊
