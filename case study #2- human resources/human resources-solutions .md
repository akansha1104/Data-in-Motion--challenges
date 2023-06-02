#   📝 Case study #2 -- Human Resources--- Solutions!!!





##  Q1. Find the longest ongoing project for each department.


```sql
WITH cte1 AS (
  				SELECT p.id as project_id, p.name, d.id as department_id,d.name,
  				(DATE(p.end_date) - DATE(p.start_date)) as no_of_days
  				FROM projects p
  				JOIN departments d
  				ON p.department_id = d.id
  				GROUP BY d.id , p.id
  				ORDER BY no_of_days
  ), cte2 AS (
    			SELECT * , 
    			DENSE_RANK()OVER(PARTITION BY department_id order by no_of_days desc) AS rn
    			FROM cte1
    )
    	SELECT * FROM cte2
        WHERE rn = 1;

  ```
  
  ## steps
  1. First I calcualted the duration of each project by finding the difference between the start date of project and end date of it and wrapped this in cte1.
  2. Then I found the rank for each project partitioning by department id and ordering by the duration ( no of days here) . I used dense_rank function so that I get   exact rank of projects without skipping any rank as rn. This dataset has one project for each department, but in case if there were more than 1 and have same duration then rank function would give it same rank but skip the next rank. I wrapped this in cte2, as in next step I want to use the where clause to find the longest duration time. I cannot use it here as rn is a derived column and we cannot use derived column in where clause (SQL basics :bulb:).
  3. Now I used where clause to find the projects with longest duration as I have ordered the projects with no of days in descending order, so the projects having rank as 1 will have the longest duration of time.


 ## answer 
  
|  project_id |	name |	department_id |department_name | no_of_days |rn
|-------------|-----|----------------|-------|-----------|--
 |1	        | HR Project 1 |	1	|HR|	180	|1|
|2	|IT Project 1|	2|	IT|	180|	1|
|3	|Sales Project 1|	3	|Sales|	183|	1   |




## Q2. Find all employees who are not managers.


```sql
SELECT id AS employee_id, name
FROM employees e
WHER job_title NOT LIKE '%Manager%'
GROUP BY id, name
ORDER BY id;
```

## steps 
1. Here I simply used WILD CARD search to find the employees who are not managers and ordered them by ID to make the report more readible.
2. '%' is wild card search which is used with LIKE clause to filter the results.

## Answer

|employee_id|	name|
|------------|------|
|4	|Bob Miller|
|5	|Charlie Brown|
|6	|Dave Davis|




## Q3. Find all employees who have been hired after the start of a project in their department.



```sql
SELECT e.id AS employee_id, e.name, e.hire_date, p.start_date,d.name AS department_name
FROM employees e
JOIN projects p USING (department_id)
JOIN departments d ON e.department_id = d.id
WHERE DATE(e.hire_date) > DATE(p.start_date)
GROUP BY e.id, e.name, p.start_date, d.name;
```

## steps

1. First I joined employees table with projects table using department_id as common column to find the employees which are employed for specific project, starting date of project and get their hiring date as well.
2. Then i joined departments table to get the department name fpr employees and specific project they are working on.
3. Next step was to use where clause to find the employees where the hire date was more than the start date of project as I have to find out the employees who were hired after the start of project and grouped them employee_id , name , hire date , start date and department as well.
4. I found out that DAVE DAVIS was the only employee who was hired after the start of the project in sales department.


## Answer

|employee_id|	name|	hire_date|	start_date|department_name|
|--------|-------|--------|-------|-------|
|6	|Dave Davis	|2023-03-15T00:00:00.000Z|	2023-03-01T00:00:00.000Z|	Sales|




## Q4. Rank employees within each department based on their hire date (earliest hire gets the highest rank).



```sql
SELECT e.name, d.name AS department_name,
		DENSE_RANK()OVER(PARTITION BY e.department_id ORDER BY DATE(e.hire_date)) AS rnk
FROM employees e 
JOIN departments d 
ON e.department_id = d.id
GROUP BY e.name, d.name, d.id, e.hire_date, e.department_id
ORDER BY d.id;
```

## steps
1. Here I used dense_rank() function to get ranks of all employees in each department based on their hiring date. So I partitioned by department_id to get each department window and ordered the results by hiring dates in ascending order as I wanted the earliest hire as rank 1.
2. I joined employees table with departments table to get the department name as well in the output result.

## Answer

|name|department_name|rnk|
|----|------|-------|
|John Doe|	HR|	1|
|Bob Miller|	HR|	2|
|Jane Smith|	IT|	1|
|Charlie Brown|	IT	|2|
|Alice Johnson|	Sales|	1|
|Dave Davis	|Sales	|2|




## Q5. Find the duration between the hire date of each employee and the hire date of the next employee hired in the same department.



```sql
WITH cte AS (
  				SELECT e.name, d.name as department, e.hire_date ,
  				LEAD(e.hire_date,1)OVER(PARTITION BY e.department_id ORDER BY e.hire_date) AS next_hire_date
  				FROM employees e 
  				JOIN departments d
  				ON e.department_id = d.id
  )
  	SELECT *, 
    		DATE(next_hire_date) - DATE(hire_date) AS duration_in_days
    FROM cte;
```

## steps

1. To solve this question i first used LEAD() function with partition by clause to find the next row of hiring date (next_hire_date) in ascending order. This way I found the next hiring date in each department by creating a window for each department and wrapped it in cte1 (WINDOW functions basics :bulb:). I used cte here becuase I want to use this next_hire_date column in further query to find the difference. I cannot use it directly as it is a derived column.
2. I joined employees table with departments table to get the required details of employees like hiring date and department name.
3. I used DATE() function to find the difference between the next_hire_date and hire date to find the duration as duration_days.  

## Answer


|name	|department	|hire_date|	next_hire_date|	duration_in_days|
|-----|-----|-----|-----|-----|
|John Doe|	HR	|2018-06-20T00:00:00.000Z|	2021-04-30T00:00:00.000Z|	1045|
|Bob Miller	|HR	|2021-04-30T00:00:00.000Z|	null|	null|
Jane Smith|	IT|	2019-07-15T00:00:00.000Z|	2022-10-01T00:00:00.000Z|	1174|
|Charlie Brown	|IT	|2022-10-01T00:00:00.000Z	|null|	null
|Alice Johnson	|Sales|	2020-01-10T00:00:00.000Z|	2023-03-15T00:00:00.000Z|	1160|
|Dave Davis|	Sales|	2023-03-15T00:00:00.000Z|	null|	null|

    






