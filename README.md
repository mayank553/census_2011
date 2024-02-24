# census_2011
SELECT * FROM Data1
SELECT * FROM  Data2

--number of rows in our dataset
SELECT COUNT(*) from data1
SELECT COUNT(*) from data2

--dataset for Bihar and Jharkhand
SELECT * FROM Data1
WHERE [State ] IN ( 'Bihar', 'Jharkhand') 

--population of India
SELECT FORMAT(SUM(Population), N'N0') AS Total_Population FROM Data2

--avg growth
SELECT [State ], FORMAT(ROUND(AVG(growth)*100, 2), N'N1') AS growth_percent
FROM Data1
GROUP BY [State ]

--avg sex ratio
SELECT [State ], ROUND(AVG(Sex_Ratio), 0) AS avg_sex_ratio
FROM Data1
GROUP BY [State ]
ORDER BY avg_sex_ratio DESC

--avg literacy rate
SELECT [State ], ROUND(AVG(Literacy), 0) AS avg_literacy_rate
FROM Data1
GROUP BY [State ]
HAVING ROUND(AVG(Literacy), 0)>90
ORDER BY avg_literacy_rate DESC

--3 states showing highest growth %
SELECT Top 3 [State ], CAST(ROUND(AVG(growth)*100,2)  AS DECIMAL(10,1)) AS growth_percent
FROM Data1
GROUP BY [State ]
ORDER BY growth_percent DESC

--bottom 3 states showing lowest sex ratio
SELECT TOP 3 [State ], ROUND(AVG(Sex_Ratio), 0) AS avg_sex_ratio
FROM Data1
GROUP BY [State ]
ORDER BY avg_sex_ratio ASC

--number of males nad females
SELECT 
	b.[State ], 
	sum(b.males) AS Total_males, 
	sum(b.females) AS Total_females
FROM
(SELECT 
	a.District,	
	a.[State ], 
	ROUND(a.population/(a.Sex_ratio + 1),0) AS Males, 
	ROUND((a.Population * a.Sex_Ratio)/(a.Sex_Ratio+1),0) AS Females
FROM
(SELECT 
	d1.District, 
	d1.[State ], 
	d1.Sex_Ratio/1000 AS Sex_Ratio, 
	d2.Population
FROM Data1 as D1
INNER JOIN 
Data2 as D2 ON D1.District = D2.District) AS a) AS b
GROUP BY b.[State ]

---Total literacy rate
SELECT 
	B.State, 
	sum(B.Literate_People) AS Total_Literate_People, 
	sum(B.Illiterate_People) AS Total_Illiterate_People 
FROM
(SELECT A.District, A.State, ROUND(A.literacy_ratio*A.Population, 0) AS Literate_People, ROUND((1-A.Literacy_Ratio)*A.Population,0) AS Illiterate_People
FROM
(SELECT 
	d1.District, 
	d1.[State ], 
	d1.Literacy/100 AS Literacy_Ratio, 
	d2.Population
FROM Data1 as D1
INNER JOIN 
Data2 as D2 ON D1.District = D2.District) AS A) AS B
GROUP BY B.State


--top 3 districts from each state with highest literacy rate
SELECT a.* FROM
(SELECT
	[State ], 
	District, 
	literacy, 
	RANK() OVER (PARTITION BY State ORDER BY Literacy DESC) AS rnk 
FROM Data1) AS a
WHERE a.rnk IN (1,2,3)
