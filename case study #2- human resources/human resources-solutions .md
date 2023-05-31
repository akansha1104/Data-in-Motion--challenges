# 👩‍🦱 👨‍🦱 📊 📃 Case study #2 -- Human Resources--- Solutions!!!





##  Q1. Find the longest ongoing project for each department.


```sql
with cte1 as (
  				select p.id as project_id, p.name, d.id as department_id,d.name,
  				(date(p.end_date) - date(p.start_date)) as no_of_days
  				from projects p
  				join departments d
  				on p.department_id = d.id
  				group by d.id , p.id
  				order by no_of_days
  ), cte2 as (
    			select * , 
    			dense_rank()over(partition by department_id order by no_of_days desc) as rn
    			from cte1
    )
    	select * from cte2
        where rn = 1;

  ```
 ## answer 
  
|  project_id |	name |	department_id |department_name | no_of_days |rn
|-------------|-----|----------------|-------|-----------|--
 |1	        | HR Project 1 |	1	|HR|	180	|1|
|2	|IT Project 1|	2|	IT|	180|	1|
|3	|Sales Project 1|	3	|Sales|	183|	1   |




## Q2. Find all employees who are not managers.


```sql
select id as employee_id, name
from employees e
where job_title not like '%Manager%'
group by id, name
order by id;
```

## Answer

|employee_id|	name|
|------------|------|
|4	|Bob Miller|
|5	|Charlie Brown|
|6	|Dave Davis|




## Q3. Find all employees who have been hired after the start of a project in their department.



```sql
select e.id as employee_id, e.name, e.hire_date, p.start_date,d.name as department_name
from employees e
join projects p using (department_id)
join departments d on e.department_id = d.id
where date(e.hire_date) > date(p.start_date)
group by e.id, e.name, p.start_date, d.name;
```
## Answer

|employee_id|	name|	hire_date|	start_date|department_name|
|--------|-------|--------|-------|-------|
|6	|Dave Davis	|2023-03-15T00:00:00.000Z|	2023-03-01T00:00:00.000Z|	Sales|




## Q4. Rank employees within each department based on their hire date (earliest hire gets the highest rank).



```sql
select e.name, d.name as department_name,
		dense_rank()over(partition by e.department_id order by date(e.hire_date)) as rnk
from employees e 
join departments d 
on e.department_id = d.id
group by e.name, d.name, d.id, e.hire_date, e.department_id
order by d.id;
```

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
with cte as (
  				select e.name, d.name as department, e.hire_date ,
  				lead(e.hire_date,1)over(partition by e.department_id order by e.hire_date) as next_hire_date
  				from employees e 
  				join departments d
  				on e.department_id = d.id
  )
  	select *, 
    		date(next_hire_date) - date(hire_date) as duration_in_days
    from cte;
```

## Answer


|name	|department	|hire_date|	next_hire_date|	duration_in_days|
|-----|-----|-----|-----|-----|
|John Doe|	HR	|2018-06-20T00:00:00.000Z|	2021-04-30T00:00:00.000Z|	1045|
|Bob Miller	|HR	|2021-04-30T00:00:00.000Z|	null|	null|
Jane Smith|	IT|	2019-07-15T00:00:00.000Z|	2022-10-01T00:00:00.000Z|	1174|
|Charlie Brown	|IT	|2022-10-01T00:00:00.000Z	|null|	null
|Alice Johnson	|Sales|	2020-01-10T00:00:00.000Z|	2023-03-15T00:00:00.000Z|	1160|
|Dave Davis|	Sales|	2023-03-15T00:00:00.000Z|	null|	null|

    






