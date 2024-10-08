# Kyozou

•	The following has been requested
There is a classic employee table id, department_id, salary - all numeric fields. id - primary key
we need to select 5 department_ids with the maximum total employees salary
smth like select department_id from (select department_id, sum(salary) salary from employee group by department_id) a where a.salary = (select max(salary) from (select department_id, sum(salary) salary from employee group by department_id) b);
it works but it's far from being optimal
could you please provide 2 queries:
1.	using CTE
2.	without using CTE, the simplest possible one

---
- Will start with the specific data model
  - The data is configured as follows
    - Department ID - assume int
    - salary        - double
    - ID            - int (assume not large number of salaries)
- This is a simple DB 
- First point I'd make is that this does not appear to be an employee table - but an employee salary table

## Preparation

 - First goal will be to prepare a [SQL Server] DB with the specified data.
 - To be conformant with  the example query it would detail that the data model is: 
   - centered on department ID   
 - There is a DB Script to create the DB and table
   - Kyozo.sql

## Strategy

Reviewing what was requested it was identified that a grouping of Department ID is required. On top of that, an aggregation sum of salary [for Department ID] was requested.

An initial review of the supplied example script was done.
The example script appeared to be overkill and multiple queries where being generated.
It was determined that a simple query that aggregated and summed the salary is all that is required,


## Solution
Note: all logic is based on SQL Server

The following was identified as what would be required to generate a query that would provide the top 5 Department ID entries based on the maximum [sum] salaries for the department ID

- The following is a script containing a CTE to produce the expected result

		with top_salaries as
		(
			select  
				department_id, 
				sum(salary) salary
			from  
				employee
			group by 
				department_id	
		)
		select top 5
			department_id
		from 	
			top_salaries
		order by 
			salary desc	

- CTE is a preparation mechanism to produce a dataset that represents a presentation of data as needed. CTE is a cleaner implementation than straight temporary tables
- The previous script has a CTE that generates a simple department/salary grouping. This can then be used to generate results on department ID and salary

---
- The following is a script to produce the expected result
 
		select 
			department_id
		from
		(
			select  top 5
				department_id, 
				sum(salary) salary
			from  
				employee
			group by 
				department_id
			order by salary desc	
		) T1

The previous script uses aspects of nested queries. The inner query is what actually retrieves the result and the outer query finalized to the requested result.

---

## Conclusion
- The request was to optimize the query operation
    - This was done by minimizing the number of operations/selections being done
      - have reduced the query to use approximately 2 selections
- The example script was overworked and performed operations that were really not necessary - requiring multiple select operations 
  - the example script also appeared to only produce a single department Id - and not the requested 5
- One of the big aspects of DB design is to optimize query retrieval
  - One of the biggest features of relational DB systems is the use of indexes that provide ordering of specific parts of the data model
    - This was done in the DB specifically on the Department ID (IX_employee_department) - as this is the central piece of data that the query would be based on - and retrieving this data as quickly as possible would be achieved using the index

The use of a CTE is not indicated as a performance enhancer. It appears that CTEs promote cleanliness and maintainability of the scripts



