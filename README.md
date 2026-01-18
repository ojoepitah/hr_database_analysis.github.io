# Human Resources Database (SQL Analysis)

## Table of Content
- [Executive Summary](#executive-summary)
- [Project Objectives](#project-objectives)
- [Business Questions and Problems Solved](#business-questions-and-problems-solved)
- [Dataset Overview](#dataset-overview)
- [SQL Server Features and Techniques Used](#sql-server-features-and-techniques-used)
- [Data Cleaning and Transformation](#data-cleaning-and-transformation)
- [Findings and Business Value](#findings-and-business-value)
- [Skills Demonstrated](#skills-demonstrated)

## Executive Summary
This project leverages SQL Server to transform raw Human Resources data into actionable workforce insights that support data-driven HR and strategic decision-making. 

Using a dataset of over 22,000 employee records spanning 2000–2020, the analysis focused on workforce demographics, employee tenure, turnover trends, and organizational growth. A robust SQL workflow was implemented using CTEs, subqueries, window functions, and derived metrics to produce reliable and business-ready outputs.

The analysis uncovered key insights such as workforce composition by gender, race, age group, department, and location; average employment duration for terminated employees; departmental turnover rates; and year-over-year changes in employee headcount. These findings provide visibility into retention risks, workforce diversity, hiring effectiveness, and long-term organizational trends.

## Project Objectives

- Clean and standardize historical HR data for analysis readiness
- Provide visibility into workforce demographics
- Assess employee retention, turnover, and tenure patterns
- Track workforce growth and organizational trends over time
- Support strategic HR planning and operational decision-making
- Demonstrate practical SQL proficiency in a real-world business context

## Business Questions and Problems Solved

- What is the gender breakdown of employees in the company?
- What is the race/ethnicity breakdown of employees in the company?
- What is the age distribution of employees in the company?
- How many employees work at headquarters versus remote locations?
- What is the average length of employment for employees who have been terminated?
- How does the gender distribution vary across departments and job titles?
- What is the distribution of job titles across the company?
- Which department has the highest turnover rate?
- What is the distribution of employees across locations by state?
- How has the company's employee count changed over time based on hire and term dates?
- What is the tenure distribution for each department?

[Jump to Findings and Business Value](#findings-and-business-value)


## Dataset Overview

HR Data with over 22000 rows from the year 2000 to 2020.

### Key entities

| Column Name | Description | 
|-----------------|-------------|
| [id] | Unique identifier for each employee (e.g. 00-0037846, 00-0041533, etc.) |
| [first_name] | employee first name (e.g. Kimmy, Ignatius, etc.) |
| [last_name] | employee last name (e.g. Walczynski, Bittlestone, etc.) |
| [birthdate] | employee date of birth |
| [gender] | employee gender (Male/Female) |
| [race] | e.g. Hispanic or Latino, White, Black or African American |
| [department] | e.g. Engineering, Business Development, e.t.c |
| [jobtitle] | e.g. Programmer Analyst I, Business Analyst, Solutions Engineer Manager |
| [location] | employee working location (Headquarters/Remote) |
| [hire_date] | employee hire date |
| [termdate] | employee employment termination date |
| [location_city] | employee work location city. Headquarter = Cleveland, Remote = (varying) |
| [location_state] | employee work location state. Headquarter = Ohio, Remote = (varying) |

### Data challenges encountered
- Some records had negative ages and these were excluded during querying(967 records). Ages used were 18 years and above.
- Some termdates were far into the future and were not included in the analysis(1599 records). The only term dates used were those less than or equal to the current date.

## SQL Server Features and Techniques Used

- DDL (CREATE, ALTER, DROP)
- DML (INSERT, UPDATE, DELETE)
- Subqueries and CTEs
- Window functions (ROW_NUMBER, RANK, etc.)
- Error handling (TRY...CATCH)

## Data Cleaning and Transformation

- Handling missing or invalid data
- Data type conversions
- Deduplication logic
- Standardization (dates, currencies, text)
- Derived columns or calculated fields

```sql
-- SELECT * FROM hr;

ALTER TABLE HR
CHANGE COLUMN ï»¿id emp_id VARCHAR (50) NULL;

-- DESCRIBE HR;
-- SELECT birthdate FROM HR;

UPDATE HR
SET birthdate = CASE 
	WHEN birthdate LIKE '%/%' THEN DATE_FORMAT(STR_TO_DATE(birthdate, '%m/%d/%Y'),'%Y-%m-%d') 
    WHEN birthdate LIKE '%-%' THEN DATE_FORMAT(STR_TO_DATE(birthdate, '%m-%d-%Y'),'%Y-%m-%d')
    ELSE NULL
END;

ALTER TABLE HR
MODIFY COLUMN birthdate DATE;

-- DESCRIBE HR;
-- SELECT birthdate FROM HR;

UPDATE HR
SET hire_date = CASE
	WHEN hire_date LIKE '%/%' THEN DATE_FORMAT(STR_TO_DATE(hire_date, '%m/%d/%Y'), '%Y-%m-%d') 
    WHEN hire_date LIKE '%-%' THEN DATE_FORMAT(STR_TO_DATE(hire_date, '%m-%d-%Y'), '%Y-%m-%d')
    ELSE NULL
END;

ALTER TABLE HR
MODIFY COLUMN hire_date DATE;

-- DESCRIBE HR;
-- SELECT hire_date FROM HR;

UPDATE HR
SET termdate = DATE(STR_TO_DATE(termdate,'%Y-%m-%d %H:%i:%s UTC'))
WHERE termdate IS NOT NULL 
	AND termdate != '';

UPDATE HR
SET termdate = NULL
WHERE termdate = '';

ALTER TABLE HR
MODIFY COLUMN termdate DATE;

-- SELECT * FROM HR;

ALTER TABLE HR
ADD COLUMN age INT;

-- SELECT age FROM HR;

UPDATE HR
SET age = timestampdiff(YEAR,birthdate,curdate());

-- SELECT birthdate,age FROM HR;

-- SELECT 
	-- min(age) AS Youngest,
    -- max(age) AS Oldest
-- FROM HR;    

-- SELECT COUNT(age)
-- FROM HR
-- WHERE age < 18;
	
-- SELECT * FROM HR;
```

## Key Queries for Business Insight

#### 1) What is the gender breakdown of employees in the company?
   
```sql
-- 1. What is the gender breakdown of employees in the company?
SELECT gender, COUNT(*) AS count
FROM HR
WHERE age >= 18
	AND termdate IS NULL
GROUP BY gender;
```

#### 2) What is the race/ethnicity breakdown of employees in the company?

```sql
-- 2. What is the race/ethnicity breakdown of employees in the company?
SELECT race, count(*) as count
FROM HR
WHERE age >= 18
	AND termdate IS NULL
GROUP BY race
ORDER BY count DESC;
```

#### 3) What is the age distribution of employees in the company?

```sql
-- 3. What is the age distribution of employees in the company?
--- Find the min and max age:
SELECT 
	min(age) AS Youngest,
    max(age) AS Oldest
FROM HR
   WHERE age >= 18
	AND termdate IS NULL;

--Youngest = 23, Oldest = 60

SELECT
	CASE
		WHEN age >= 18 AND age <= 24 THEN '18-24'
        WHEN age >= 25 AND age <= 34 THEN '25-34'
        WHEN age >= 35 AND age <= 44 THEN '35-44'
        WHEN age >= 45 AND age <= 54 THEN '45-54'
        WHEN age >= 55 AND age <= 65 THEN '55-65'
		ELSE '65+'
    END AS Age_group,
    count(*) AS count
FROM HR
WHERE age >= 18
	AND termdate IS NULL
GROUP BY Age_group
ORDER BY Age_group;
```

#### 4) How many employees work at headquarters versus remote locations?

```sql
-- 4. How many employees work at headquarters versus remote locations?
-- SELECT * FROM HR;
SELECT location, COUNT(*) as count
FROM HR
WHERE age >= 18
	AND termdate IS NULL
GROUP BY location;
```

### 5) What is the average length of employment for employees who have been terminated?

```sql
-- 5. What is the average length of employment for employees who have been terminated?
SELECT ROUND(
			AVG(
				DATEDIFF(termdate, hire_date)
		)/365) AS avg_length_employment
FROM HR
WHERE termdate <= CURDATE()
	AND age >= 18
	AND termdate IS NOT NULL;
```

#### 6) How does the gender distribution vary across departments and job titles?

```sql
-- 6. How does the gender distribution vary across departments and job titles?
SELECT department, gender, count(*) AS count
FROM HR
WHERE age >= 18
	AND termdate IS NULL
GROUP BY department, gender
ORDER BY department;
```

#### 7) What is the distribution of job titles across the company?

```sql
-- 7. What is the distribution of job titles across the company?
SELECT jobtitle, count(*) AS count
FROM HR
WHERE age >= 18
	AND termdate IS NULL
GROUP BY jobtitle
ORDER BY jobtitle DESC;
```

### 8) Which department has the highest turnover rate?

```sql
-- 8. Which department has the highest turnover rate?
SELECT department,
		total_count,
        terminated_count,
        (terminated_count/total_count) AS termination_rate
FROM (SELECT department, 
			COUNT(*) AS total_count,
			SUM(CASE WHEN termdate IS NOT NULL
							AND age >= 18
                            AND termdate <= CURDATE()
                     THEN 1
                     ELSE 0
                END) AS terminated_count
       FROM HR
       WHERE age >= 18
       GROUP BY department
	) AS subquery
ORDER BY termination_rate DESC;
----------------------------------------------------------------------

-- Termination_PercentageRate

SELECT department,
		total_count,
        terminated_count,
        ROUND((terminated_count/total_count)*100,2) AS `% termination_rate`
FROM (SELECT department, 
			COUNT(*) AS total_count,
			SUM(CASE WHEN termdate IS NOT NULL
							AND age >= 18
                            AND termdate <= CURDATE()
                     THEN 1
                     ELSE 0
                END) AS terminated_count
       FROM HR
       WHERE age >= 18
       GROUP BY department
	) AS subquery
ORDER BY `% termination_rate` DESC;
----------------------------------------------------------------------
```

#### 9) What is the distribution of employees across locations by city and state?

```sql
-- 9. What is the distribution of employees across locations by city and state?
SELECT location_state, COUNT(*) as count
FROM HR
GROUP BY location_state;

----------------------------------------------------------------------
SELECT location_state, location_city, COUNT(*) as count
FROM HR
GROUP BY location_state, location_city
ORDER BY count DESC;
-- Could'nt group by city as there are over 100 cities captured in the data which would make the map representation messy. 
-- This is why an interative dashboard is would be a better representation of this dataset as this would provide opportunity for drilldown.
----------------------------------------------------------------------
```

### 10) How has the company's employee count changed over time based on hire and term dates?

```sql
-- 10. How has the company's employee count changed over time based on hire and term dates?
SELECT
	Year,
    hire,
    termination,
    (hire - termination) AS net_change,
    ROUND(((hire - termination)/hire)*100,2) AS `% net_change`
FROM(SELECT YEAR(hire_date) AS Year,
			COUNT(*) AS hire,
            SUM(CASE WHEN termdate IS NOT NULL AND age >= 18 AND termdate <= CURDATE()
					THEN 1
                    ELSE 0
				END) AS termination
	FROM HR
    WHERE age >= 18
    GROUP BY Year
    ) AS subquery
ORDER BY Year;
````

#### 11) What is the tenure distribution for each department?

```sql
-- 11. What is the tenure distribution for each department?
SELECT department, 
		ROUND((AVG
				(DATEDIFF(termdate, hire_date)/356)
				),
			2) AS avg_tenure
FROM HR
WHERE age >= 18
	AND termdate IS NOT NULL
    AND termdate <= CURDATE()
GROUP BY department;
```

## Findings and Business Value

- There are more male employees
- White race is the most dominant while Native Hawaiian and American Indian are the least dominant.
- The youngest employee is 20 years old and the oldest is 57 years old
- Created five age groups (18-24, 25-34, 35-44, 45-54, 55-64). A large number of employees were between 25-34 followed by 35-44 while the smallest group was 55-64.
- A large number of employees work at the headquarters versus remotely.
- The average length of employment for terminated employees is around 7 years.
- The gender distribution across departments is fairly balanced but there are generally more male than female employees.
- The Marketing department has the highest turnover rate followed by Training. The least turn over rate are in the Research and development, Support and Legal departments.
- A large number of employees come from the state of Ohio.
- The net change in employees has increased over the years.
- The average tenure for each department is about 8 years with Legal and Auditing having the highest and Services, Sales and Marketing having the lowest.

## Skills Demonstrated

- SQL Server
- Database design
- Data modeling
- Query optimization
- Data analysis
- Problem-solving

