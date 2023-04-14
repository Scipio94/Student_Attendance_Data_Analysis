~~~ SQL 
/*State Absences*/
CREATE TEMP TABLE State AS
SELECT
  CAST(Student_School_ID AS int64) AS ID,
  Student_First_Name AS First, 
  Student_Last_Name AS Last, 
  Student_Grade_Level AS Grade,
  (School_Present_Count + School_Not_Taken_Count + School_Lateness_Count)/ ((School_Present_Count + School_Not_Taken_Count + School_Lateness_Count) + (School_Absence_Count)) AS Percentage,
  School_Present_Count + School_Not_Taken_Count + School_Lateness_Count AS Present_Total,
  (School_Present_Count + School_Not_Taken_Count + School_Lateness_Count) + (School_Absence_Count) AS Total_Days,
   CASE 
    WHEN student_grade_level IN ('0','1','2') THEN 'PS'
    WHEN student_grade_level IN ('3','4','5') THEN 'IS'
    WHEN student_grade_level IN ('6','7','8') THEN 'MS'
    WHEN student_grade_level IN ('9','10','11','12') THEN 'HS'
    END AS Campus
FROM `my-data-project-36654.FA_Student_Attendance.Attendance_Snapshot`
WHERE Student_School_ID <> '101FakeKid';

/*Attendance Percentages STATE*/
SELECT 
  ID,
  First, 
  Last,
  Grade, 
  Percentage,
    CASE
    WHEN Percentage <= .90 THEN 'Chronic Absentee'
    WHEN Percentage > .90 THEN 'Not Chronic Absentee'
    END AS Chronic_Absentee,
  Present_Total, 
  Total_Days
FROM State 
ORDER BY Percentage DESC;


/*Chronically Absence STATE*/ -- Individual
SELECT 
  ID,
  First, 
  Last,
  Grade, 
  Percentage,
    CASE
    WHEN Percentage <= .90 THEN 'Chronic Absentee'
    WHEN Percentage > .90 THEN 'Not Chronic Absentee'
    END AS Chronic_Absentee,
  Present_Total, 
  Total_Days
FROM State AS sub; 

/*Chronically Absence STATE*/ -- Individual
SELECT 
  sub.Chronic_Absentee,
  sub.Not_Chronic_Absentee,
  sub.Chronic_Absentee/Total AS Chronic_Absentee_Rate,
  Total 
FROM
(SELECT 
  COUNT(CASE WHEN Percentage <= .90 THEN 'Chronic Absenteeism' END) AS Chronic_Absentee,
  COUNT(CASE WHEN Percentage > .90 THEN 'Not Chroninc Absenteeism' END) AS Not_Chronic_Absentee,
  COUNT(CASE WHEN Percentage <= .90 THEN 'Chronic Absenteeism' END) + COUNT(CASE WHEN Percentage > .90 THEN 'Not Chroninc Absenteeism' END) AS Total
FROM State) AS sub; 

/*Chronically Absence STATE*/ -- Grade
SELECT 
  sub.Grade,
  sub.Chronic_Absentee,
  sub.Not_Chronic_Absentee,
  sub.Chronic_Absentee/Total AS Chronic_Absentee_Rate,
  Total 
FROM
(SELECT 
  Grade,
  COUNT(CASE WHEN Percentage <= .90 THEN 'Chronic Absenteeism' END) AS Chronic_Absentee,
  COUNT(CASE WHEN Percentage > .90 THEN 'Not Chroninc Absenteeism' END) AS Not_Chronic_Absentee,
  COUNT(CASE WHEN Percentage <= .90 THEN 'Chronic Absenteeism' END) + COUNT(CASE WHEN Percentage > .90 THEN 'Not Chroninc Absenteeism' END) AS Total
FROM State
GROUP BY Grade) AS sub
GROUP BY   
  sub.Grade,
  sub.Chronic_Absentee,
  sub.Not_Chronic_Absentee,
  Total;

/*Chronically Absence STATE*/ -- Campus
SELECT 
  sub.Campus,
  sub.Chronic_Absentee,
  sub.Not_Chronic_Absentee,
  sub.Chronic_Absentee/Total AS Chronic_Absentee_Rate,
  Total,
  SUM(Total) OVER () AS Total_Students
FROM
(SELECT 
  Campus,
  COUNT(CASE WHEN Percentage <= .90 THEN 'Chronic Absenteeism' END) AS Chronic_Absentee,
  COUNT(CASE WHEN Percentage > .90 THEN 'Not Chroninc Absenteeism' END) AS Not_Chronic_Absentee,
  COUNT(CASE WHEN Percentage <= .90 THEN 'Chronic Absenteeism' END) + COUNT(CASE WHEN Percentage > .90 THEN 'Not Chroninc Absenteeism' END) AS Total
FROM State
GROUP BY Campus) AS sub
GROUP BY   
  sub.Campus,
  sub.Chronic_Absentee,
  sub.Not_Chronic_Absentee,
  Total;

/*iReady Scores ELA*/
SELECT 
  ID,
  First, 
  Last,
  Grade, 
  Percentage,
    CASE 
    WHEN Percentage <= .90 THEN 'Chronic Absentee'
    ELSE 'Not Chronic Absentee'
    END AS Chronic_Absentee,
  ELA1.Overall_Scale_Score AS ELA1_Scale_Score,
  ELA1.Overall_Relative_Placement AS ELA1_Relative_Placement,
  CASE 
    WHEN ELA1.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
    WHEN ELA1.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
    WHEN ELA1.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
    END AS Tiers,
  ELA2.Overall_Scale_Score AS ELA2_Scale_Score,
  ELA2.Overall_Relative_Placement AS ELA2_Relative_Placement,
     CASE 
    WHEN ELA2.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
    WHEN ELA2.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
    WHEN ELA2.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
    END AS Tiers
FROM State 
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_ELA_1` AS ELA1
ON State.ID = ELA1.Student_ID
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_2_ELA` AS ELA2
ON State.ID = ELA2.Student_ID
ORDER BY Percentage DESC;

/*iReady Metrics ELA 1*/
SELECT  
  DISTINCT sub.chronic_Absentee, 
  ROUND(AVG(sub.ELA2_Scale_Score - sub.ELA1_Scale_Score) OVER (PARTITION BY sub.chronic_Absentee)) AS Avg_Scale_Score_Change,
  COUNT(*) OVER (PARTITION BY sub.chronic_absentee) AS Count,
  ROUND(COUNT(*) OVER (PARTITION BY sub.chronic_absentee)/ COUNT(*) OVER (),2) AS Absentee_Percentage,
  sub.ELA1_Tier,
  COUNT(*) OVER (PARTITION BY sub.chronic_absentee, sub.ELA1_Tier) AS ELA1_Tier_Count,
  ROUND(COUNT(*) OVER (PARTITION BY sub.chronic_absentee, sub.ELA1_Tier)/ COUNT(*) OVER (PARTITION BY sub.chronic_absentee),2) AS Tier_Percentage, 
FROM 
(SELECT 
  ID,
  First, 
  Last,
  Grade, 
  Percentage,
    CASE 
    WHEN Percentage <= .90 THEN 'Chronic Absentee'
    ELSE 'Not Chronic Absentee'
    END AS Chronic_Absentee,
  ELA1.Overall_Scale_Score AS ELA1_Scale_Score,
  ELA1.Overall_Relative_Placement AS ELA1_Relative_Placement,
  CASE 
    WHEN ELA1.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
    WHEN ELA1.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
    WHEN ELA1.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
    END AS ELA1_Tier,
  ELA2.Overall_Scale_Score AS ELA2_Scale_Score,
  ELA2.Overall_Relative_Placement AS ELA2_Relative_Placement,
     CASE 
    WHEN ELA2.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
    WHEN ELA2.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
    WHEN ELA2.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
    END AS ELA2_Tier
FROM State 
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_ELA_1` AS ELA1
ON State.ID = ELA1.Student_ID
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_2_ELA` AS ELA2
ON State.ID = ELA2.Student_ID
ORDER BY Percentage DESC) AS sub
ORDER BY sub.chronic_absentee DESC ;
-- Trend in which if you are chronically absent, less likely to be on grade level


/*iReady Metrics ELA 2*/
SELECT  
  DISTINCT sub.chronic_Absentee, 
  ROUND(AVG(sub.ELA2_Scale_Score - sub.ELA1_Scale_Score) OVER (PARTITION BY sub.chronic_Absentee)) AS Avg_Scale_Score_Change,
  COUNT(*) OVER (PARTITION BY sub.chronic_absentee) AS Count,
  ROUND(COUNT(*) OVER (PARTITION BY sub.chronic_absentee)/ COUNT(*) OVER (),2) AS Absentee_Percentage,
  sub.ELA2_Tier,
  COUNT(*) OVER (PARTITION BY sub.chronic_absentee, sub.ELA2_Tier) AS ELA2_Tier_Count,
  ROUND(COUNT(*) OVER (PARTITION BY sub.chronic_absentee, sub.ELA2_Tier)/ COUNT(*) OVER (PARTITION BY sub.chronic_absentee),2) AS Tier_Percentage, 
FROM 
(SELECT 
  ID,
  First, 
  Last,
  Grade, 
  Percentage,
    CASE 
    WHEN Percentage <= .90 THEN 'Chronic Absentee'
    ELSE 'Not Chronic Absentee'
    END AS Chronic_Absentee,
  ELA1.Overall_Scale_Score AS ELA1_Scale_Score,
  ELA1.Overall_Relative_Placement AS ELA1_Relative_Placement,
  CASE 
    WHEN ELA1.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
    WHEN ELA1.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
    WHEN ELA1.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
    END AS ELA1_Tier,
  ELA2.Overall_Scale_Score AS ELA2_Scale_Score,
  ELA2.Overall_Relative_Placement AS ELA2_Relative_Placement,
     CASE 
    WHEN ELA2.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
    WHEN ELA2.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
    WHEN ELA2.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
    END AS ELA2_Tier
FROM State 
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_ELA_1` AS ELA1
ON State.ID = ELA1.Student_ID
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_2_ELA` AS ELA2
ON State.ID = ELA2.Student_ID
ORDER BY Percentage DESC) AS sub
ORDER BY sub.chronic_absentee DESC ;
-- Trend in which if you are chronically absent, less likely to be on grade level



/*iReady Scores MATH*/
SELECT 
  ID,
  First, 
  Last,
  Grade, 
  Percentage,
    CASE 
    WHEN Percentage <= .90 THEN 'Chronic Absentee'
    ELSE 'Not Chronic Absentee'
    END AS Chronic_Absentee,
  MATH1.Overall_Scale_Score AS MATH1_Scale_Score,
  MATH1.Overall_Relative_Placement MATH1_Relative_Placement,
  CASE 
    WHEN MATH1.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
    WHEN MATH1.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
    WHEN MATH1.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
    END AS MATH1_Tier,
  MATH2.Overall_Scale_Score AS MATH2_Scale_Score,
  MATH2.Overall_Relative_Placement AS MATH2_Relative_Placement,
     CASE 
      WHEN MATH2.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
      WHEN MATH2.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
      WHEN MATH2.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
      END AS MATH2_Tier
FROM State 
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_MATH_1` AS MATH1
ON State.ID = MATH1.Student_ID
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_2_MATH` AS MATH2
ON State.ID = MATH2.Student_ID
ORDER BY Percentage DESC;

/*iReady Scores Metrics MATH 1 */
SELECT  
  DISTINCT sub.chronic_Absentee, 
  ROUND(AVG(sub.MATH2_Scale_Score - sub.MATH1_Scale_Score) OVER (PARTITION BY sub.chronic_absentee),2) AS Avg_Scale_Score_Chage,
  COUNT(*) OVER (PARTITION BY sub.chronic_absentee) AS Count,
  ROUND(COUNT(*) OVER (PARTITION BY sub.chronic_absentee)/ COUNT(*) OVER (),2) AS Absentee_Percentage,
  sub.MATH1_Tier,
  COUNT(*) OVER (PARTITION BY sub.chronic_absentee, sub.MATH1_Tier) AS MATH1_Tier_Count,
  ROUND(COUNT(*) OVER (PARTITION BY sub.chronic_absentee, sub.MATH1_Tier)/ COUNT(*) OVER (PARTITION BY sub.chronic_absentee),2) AS Tier_Percentage, 
  
FROM
(SELECT 
  ID,
  First, 
  Last,
  Grade, 
  Percentage,
    CASE 
    WHEN Percentage <= .90 THEN 'Chronic Absentee'
    ELSE 'Not Chronic Absentee'
    END AS Chronic_Absentee,
  MATH1.Overall_Scale_Score AS MATH1_Scale_Score,
  MATH1.Overall_Relative_Placement MATH1_Relative_Placement,
  CASE 
    WHEN MATH1.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
    WHEN MATH1.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
    WHEN MATH1.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
    END AS MATH1_Tier,
  MATH2.Overall_Scale_Score AS MATH2_Scale_Score,
  MATH2.Overall_Relative_Placement AS MATH2_Relative_Placement,
     CASE 
      WHEN MATH2.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
      WHEN MATH2.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
      WHEN MATH2.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
      END AS MATH2_Tier
FROM State 
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_MATH_1` AS MATH1
ON State.ID = MATH1.Student_ID
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_2_MATH` AS MATH2
ON State.ID = MATH2.Student_ID
ORDER BY Percentage DESC) AS Sub
ORDER BY sub.Chronic_Absentee;
-- Trend in which if you are chronically absent, less likely to be on grade level



/*iReady Scores Metrics MATH 2 */
SELECT  
  DISTINCT sub.chronic_Absentee,
  ROUND(AVG(sub.MATH2_Scale_Score - sub.MATH1_Scale_Score) OVER (PARTITION BY sub.chronic_absentee),2) AS Avg_Scale_Score_Chage, 
  COUNT(*) OVER (PARTITION BY sub.chronic_absentee) AS Count,
  ROUND(COUNT(*) OVER (PARTITION BY sub.chronic_absentee)/ COUNT(*) OVER (),2) AS Absentee_Percentage,
  sub.MATH2_Tier,
  COUNT(*) OVER (PARTITION BY sub.chronic_absentee, sub.MATH2_Tier) AS MATH2_Tier_Count,
  ROUND(COUNT(*) OVER (PARTITION BY sub.chronic_absentee, sub.MATH2_Tier)/ COUNT(*) OVER (PARTITION BY sub.chronic_absentee),2) AS Tier_Percentage, 
FROM
(SELECT 
  ID,
  First, 
  Last,
  Grade, 
  Percentage,
    CASE 
    WHEN Percentage <= .90 THEN 'Chronic Absentee'
    ELSE 'Not Chronic Absentee'
    END AS Chronic_Absentee,
  MATH1.Overall_Scale_Score AS MATH1_Scale_Score,
  MATH1.Overall_Relative_Placement MATH1_Relative_Placement,
  CASE 
    WHEN MATH1.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
    WHEN MATH1.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
    WHEN MATH1.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
    END AS MATH1_Tier,
  MATH2.Overall_Scale_Score AS MATH2_Scale_Score,
  MATH2.Overall_Relative_Placement AS MATH2_Relative_Placement,
     CASE 
      WHEN MATH2.Overall_Relative_Placement IN ('Early On Grade Level', 'Mid or Above Grade Level') THEN 'Tier 1'
      WHEN MATH2.Overall_Relative_Placement = '1 Grade Level Below' THEN 'Tier 2'
      WHEN MATH2.Overall_Relative_Placement IN ('2 Grade Levels Below','3 or More Grade Levels Below') THEN 'Tier 3'
      END AS MATH2_Tier
FROM State 
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_MATH_1` AS MATH1
ON State.ID = MATH1.Student_ID
INNER JOIN `my-data-project-36654.iReady_SY_2223.iReady_2_MATH` AS MATH2
ON State.ID = MATH2.Student_ID
ORDER BY Percentage DESC) AS Sub
ORDER BY sub.Chronic_Absentee;
-- Trend in which if you are chronically absent, less likely to be on grade level

~~~
