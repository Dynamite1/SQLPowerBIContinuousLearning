# Learning Logs Of PowerBI & SQL

#I will be adding new learningS of SQL and PowerBI in this projecT.

# PowerBi HeatMap

![image](https://user-images.githubusercontent.com/16399584/197061240-be0c3c91-1a77-41ab-97d7-5bee4bcec3f5.png)

# Gauge Chart and Smart Narrative 

![image](https://user-images.githubusercontent.com/16399584/197067010-0e83ba21-a0b3-4a79-95a6-2d8a5472407f.png)

# COALESCE() function to return customised value 

![image](https://user-images.githubusercontent.com/16399584/197069235-c4324c48-b5db-472e-8027-c75d23eab0f5.png)

# Highest Amount 
Window Function ( ROW_NUMBER())

![image](https://user-images.githubusercontent.com/16399584/197070950-72d9dfd9-0c23-4ab1-b024-979429e74f0f.png)

# RANK and DENSE_RANK

![image](https://user-images.githubusercontent.com/16399584/197073528-6800db69-ee5e-4c87-8574-ae7ad69dc133.png)

# Practice Writing SQL Queries using Real Dataset OLYMPICS_HISTORY


#1.	How many olympics games have been held?

SELECT  COUNT(DISTINCT Games) AS total Olypic Games
  FROM [KAGGLE].[dbo].[athlete_events$] 


#2.	List down all Olympics games held so far.

SELECT  distinct Year,Season, City
  FROM [KAGGLE].[dbo].[athlete_events$]

#3.	Mention the total no of nations who participated in each olympics game?

with total_games as (
  SELECT  ae.Games, n.region
  FROM [KAGGLE].[dbo].[athlete_events$]  ae
  join [KAGGLE].[dbo].[noc_regions$] n
  on ae.NOC= n.NOC
  group by ae.Games,n.region
 )

  select Games,COUNT(*) as total
  from total_games
  group by Games
  order by Games


#4.	Which nation has participated in all of the olympic games?

WITH total
     AS (SELECT games,
                nr.region AS country
         FROM   kaggle.dbo.athlete_events$ AS ae
                JOIN kaggle.dbo.noc_regions$ AS nr
                  ON ae.noc = nr.noc
         GROUP  BY ae.games,
                   nr.region)
SELECT DISTINCT country,
                Count(country) AS total_participated_games
FROM   total
GROUP  BY country
HAVING Count(country) = (SELECT Count(DISTINCT games) AS gm_total
                         FROM   kaggle.dbo.athlete_events$)
                         

#5.	Fetch the total no of sports played in each olympic games.

with tot_games as (
SELECT   Games ,Sport
  FROM [KAGGLE].[dbo].[athlete_events$]),
total_sport as (
select Games,count(*) as tot_games 
FROM tot_games
group by Games)

select * from total_sport


#6.	Fetch details of the oldest athletes to win a gold medal.

with cte as (

SELECT   name,Sex,team,games,city,sport, event, medal, cast( case when age = null then 0 else age end as int) as age,rank() over( order by age desc ) as Max_age
  FROM [KAGGLE].[dbo].[athlete_events$]
  where Medal= 'Gold' 
  )

select *
from cte
where Max_age=1

#7.	Find the Ratio of male and female athletes participated in all olympic games.

with cte1 as (select sum(case when Sex='M' then 1 else 0 end) as males , sum(case when Sex='F' then 1 else 0 end) as females
 from [KAGGLE].[dbo].[athlete_events$])

 select *, cast( males/cast(females as float)  as decimal(10,2)) as ratio
 from cte1

#8.	Fetch the top 5 athletes who have won the most gold medals.


SELECT Name,Team,count(1) as total_m
  FROM [KAGGLE].[dbo].[athlete_events$]
  where Medal='Gold'
  group by Name,Team
  ),

  ranked as ( 
  select *, DENSE_RANK() over ( order by total_m desc) as rnk
  from total_medal)

  select Name,Team, total_m,rnk
  from ranked
  
  where 5>= rnk

#9.	Fetch the top 5 athletes who have won the most medals (gold/silver/bronze).


with total_medal as (

SELECT Name,Team,count(1) as total_m
  FROM [KAGGLE].[dbo].[athlete_events$]
  where Medal in ('Gold','Silver','Bronze')
  group by Name,Team
  ),

  ranked as ( 
  select *, DENSE_RANK() over ( order by total_m desc) as rnk
  from total_medal)

  select Name,Team, total_m,rnk
  from ranked
  
  where 5>= rnk

#10.	Fetch the top 5 most successful countries in olympics. Success is defined by no of medals won.

SELECT region    
      ,count([Medal])
  FROM [KAGGLE].[dbo].[athlete_events$] e
  join KAGGLE.dbo.noc_regions$ r on e.NOC= r.NOC
  where Medal<>'NA'
  group by region
  order by count([Medal]) desc

#11.	List down total gold, silver and broze medals won by each country.
   with gold as(
SELECT region    
      ,count([Medal]) as total_gold
  FROM [KAGGLE].[dbo].[athlete_events$] e
  join KAGGLE.dbo.noc_regions$ r on e.NOC= r.NOC
  where Medal = 'Gold'
  group by region
 ),
 silver as (
 SELECT region    
      ,count([Medal]) as total_silver
  FROM [KAGGLE].[dbo].[athlete_events$] e
  join KAGGLE.dbo.noc_regions$ r on e.NOC= r.NOC
  where Medal = 'Silver'
  group by region)

  select g.region , g.total_gold, s.total_silver
  from gold g
  join silver s on g.region= s.region
   order by total_gold desc

  
#12.	In which Sport/event, India has won highest medals.

SELECT top 1 Sport
      ,count([Medal]) as total_gold
  FROM [KAGGLE].[dbo].[athlete_events$] e
  join KAGGLE.dbo.noc_regions$ r on e.NOC= r.NOC
  where region= 'India' and Medal<>'NA'
  group by team,Sport
  order by total_gold desc

13.	Break down all olympic games where india won medal for Hockey and how many medals in each olympic games.
  SELECT Team ,Sport,Games
      ,count([Medal]) as total_gold
  FROM [KAGGLE].[dbo].[athlete_events$] e
  join KAGGLE.dbo.noc_regions$ r on e.NOC= r.NOC
  where region= 'India' and Medal<>'NA'
  group by team,Sport,Games
  order by total_gold desc

