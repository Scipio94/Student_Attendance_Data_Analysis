~~~ SQL
/*Local Absence Policy*/
CREATE TEMP TABLE Local AS
SELECT
  CAST(Student_School_ID AS int64) AS ID,
  Student_First_Name AS First, 
  Student_Last_Name AS Last, 
  Student_Grade_Level AS Grade,
  School_Lateness_Count AS Late_Count,
  School_Absence_Count AS Absence_Count,
   CASE 
    WHEN student_grade_level IN ('0','1','2') THEN 'PS'
    WHEN student_grade_level IN ('3','4','5') THEN 'IS'
    WHEN student_grade_level IN ('6','7','8') THEN 'MS'
    WHEN student_grade_level IN ('9','10','11','12') THEN 'HS'
    END AS Campus
FROM `my-data-project-36654.FA_Student_Attendance.Attendance_Snapshot`
WHERE Student_School_ID <> '101FakeKid';

/*Local Chronic Absenteeism w/o Tardies Individual*/
SELECT 
  ID, 
  First, 
  Last, 
  Grade, 
  Absence_Count, 
    CASE
    WHEN Absence_Count >= 12 THEN 'Exceeds 12 Absences'
    WHEN Absence_Count < 12 THEN 'Does Not Exceed 12 Absences'
    END AS Chronic_Absentee
FROM Local
ORDER BY Absence_Count DESC; 

/*Avg Absences w/o Tardies Grade and Campus*/
SELECT 
  DISTINCT Grade, 
  ROUND(AVG(Absence_Count) OVER (PARTITION BY Grade)) AS Grade_Avg_Abs,
  Campus,
  ROUND(AVG(Absence_Count) OVER (PARTITION BY Campus)) AS Campus_Avg_Abs,
  ROUND(AVG(Absence_Count) OVER ()) AS Overall_AVG
FROM Local
ORDER BY Grade ;

/*Avg Absences w Tardies Grade and Campus*/
SELECT 
  DISTINCT Grade, 
  ROUND(AVG(Absence_Count + (Late_Count * .33) ) OVER (PARTITION BY Grade)) AS Grade_Avg_Abs,
  Campus,
  ROUND(AVG(Absence_Count + (Late_Count * .33)) OVER (PARTITION BY Campus)) AS Campus_Avg_Abs,
  ROUND(AVG(Absence_Count + (Late_Count * .33)) OVER ()) AS Overall_AVG
FROM Local
ORDER BY Grade;

/*Count and Percentage of Students that are chronically absent w/o tardies*/ -- Local policy >=12 absences
SELECT
  ROUND(AVG(CASE WHEN Absence_Count >= 12 THEN 1 ELSE 0  END),2) AS Exceed_12_Abs_Percentage,
  COUNT (CASE WHEN Absence_Count >= 12 THEN 'Chronic' END) AS Exceed_12_Count,
  COUNT (CASE WHEN Absence_Count < 12 THEN ' Not Chronic' END) AS Not_Exceed_12_Count,
  COUNT (CASE WHEN Absence_Count >= 12 THEN 'Chronic' END) + COUNT (CASE WHEN Absence_Count < 12 THEN ' Not Chronic' END) AS Total
FROM Local;

/*Count and Percentage of Students that are chronically absent w/o tardies*/-- Grade
SELECT
  Grade,
  ROUND(AVG(CASE WHEN Absence_Count >= 12 THEN 1 ELSE 0  END),2) AS Exceed_12_Abs_Percentage,
  COUNT (CASE WHEN Absence_Count >= 12 THEN 'Chronic' END) AS Exceed_12_Count,
  COUNT (CASE WHEN Absence_Count < 12 THEN ' Not Chronic' END) AS Not_Exceed_12_Count,
  COUNT (CASE WHEN Absence_Count >= 12 THEN 'Chronic' END) + COUNT (CASE WHEN Absence_Count < 12 THEN ' Not Chronic' END) AS Total
FROM Local
GROUP BY Grade;

/*Count and Percentage of Students that are chronically absent w/o tardies*/-- Campus
SELECT
  Campus,
  ROUND(AVG(CASE WHEN Absence_Count >= 12 THEN 1 ELSE 0  END),2) AS Exceed_12_Abs_Percentage,
  COUNT (CASE WHEN Absence_Count >= 12 THEN 'Chronic' END) AS Exceed_12_Count,
  COUNT (CASE WHEN Absence_Count < 12 THEN ' Not Chronic' END) AS Not_Exceed_12_Count,
  COUNT (CASE WHEN Absence_Count >= 12 THEN 'Chronic' END) + COUNT (CASE WHEN Absence_Count < 12 THEN ' Not Chronic' END) AS Total
FROM Local
GROUP BY Campus;

/*Local Chronic Absenteeism w/ Tardies Individual*/
SELECT 
  ID, 
  First, 
  Last, 
  Grade,
  Campus,
  Absence_Count, 
  Late_Count, 
  Absence_Count + (Late_Count/3) AS Total_Absences --Absence Count + .33 * Late_Count
FROM Local
ORDER BY Total_Absences DESC; 

/*Count and Percentage of Students that are chronically absent w/ tardies*/ -- Local policy >=12 absences
SELECT 
 ROUND(AVG(CASE WHEN sub.Total_Absences >= 12 THEN 1 ELSE 0  END),2) AS Exceed_12_Abs_Percentage,
  COUNT (CASE WHEN sub.Total_Absences >= 12 THEN 'Chronic' END) AS Exceed_12_Count,
  COUNT (CASE WHEN sub.Total_Absences  < 12 THEN ' Not Chronic' END) AS Not_Exceed_12_Count,
  COUNT (CASE WHEN sub.Total_Absences  >= 12 THEN 'Chronic' END) + COUNT (CASE WHEN Total_Absences  < 12 THEN ' Not Chronic' END) AS Total
FROM
(SELECT 
  ID, 
  First, 
  Last, 
  Grade,
  Campus,
  Absence_Count, 
  Late_Count, 
  Absence_Count + (Late_Count/3) AS Total_Absences --Absence Count + .33 * Late_Count
FROM Local) AS sub;

/*Count and Percentage of Students that are chronically absent w/ tardies*/ -- Grade -- Local policy >=12 absences
SELECT 
  sub.Grade,
 ROUND(AVG(CASE WHEN sub.Total_Absences >= 12 THEN 1 ELSE 0  END),2) AS Exceed_12_Abs_Percentage,
  COUNT (CASE WHEN sub.Total_Absences >= 12 THEN 'Chronic' END) AS Exceed_12_Count,
  COUNT (CASE WHEN sub.Total_Absences  < 12 THEN ' Not Chronic' END) AS Not_Exceed_12_Count,
  COUNT (CASE WHEN sub.Total_Absences  >= 12 THEN 'Chronic' END) + COUNT (CASE WHEN Total_Absences  < 12 THEN ' Not Chronic' END) AS Total
FROM
(SELECT 
  ID, 
  First, 
  Last, 
  Grade,
  Campus,
  Absence_Count, 
  Late_Count, 
 Absence_Count + (Late_Count/3) AS Total_Absences --Absence Count + .33 * Late_Count
FROM Local) AS sub
GROUP BY sub.Grade;

/*Count and Percentage of Students that are chronically absent w/ tardies*/ -- Campus -- Local policy >=12 absences
SELECT 
  sub.Campus,
 ROUND(AVG(CASE WHEN sub.Total_Absences >= 12 THEN 1 ELSE 0  END),2) AS Exceed_12_Abs_Percentage,
  COUNT (CASE WHEN sub.Total_Absences >= 12 THEN 'Chronic' END) AS Exceed_12_Count,
  COUNT (CASE WHEN sub.Total_Absences  < 12 THEN ' Not Chronic' END) AS Not_Exceed_12_Count,
  COUNT (CASE WHEN sub.Total_Absences  >= 12 THEN 'Chronic' END) + COUNT (CASE WHEN Total_Absences  < 12 THEN ' Not Chronic' END) AS Total
FROM
(SELECT 
  ID, 
  First, 
  Last, 
  Grade,
  Campus,
  Absence_Count, 
  Late_Count, 
  Absence_Count + (Late_Count/3) AS Total_Absences --Absence Count + .33 * Late_Count
FROM Local) AS sub
GROUP BY sub.Campus;

----
/*Using 19 as a cut off as per state rate 10%*/
SELECT
  ROUND(AVG(CASE WHEN Absence_Count >= 19 THEN 1 ELSE 0  END),2) AS Exceed_12_Abs_Percentage,
  COUNT (CASE WHEN Absence_Count >= 19 THEN 'Chronic' END) AS Exceed_12_Count,
  COUNT (CASE WHEN Absence_Count < 19 THEN ' Not Chronic' END) AS Not_Exceed_12_Count,
  COUNT (CASE WHEN Absence_Count >= 19 THEN 'Chronic' END) + COUNT (CASE WHEN Absence_Count < 19 THEN ' Not Chronic' END) AS Total
FROM Local;
~~~
