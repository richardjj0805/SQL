# SQL
--SQL:row_number(),MIN/AVG/MAX/Parition by

select A.employee_code,B.EMPLOYEE_NAME, A.team_name,A.period_name,number_of_days,
MAX(A.number_of_days) over (partition by A.team_name,A.period_name ) as MaxNumberOfDays ,
AVG(A.number_of_days) over (partition by A.team_name,A.period_name ) as AVGNumberOfDays ,
MIN(A.number_of_days) over (partition by A.team_name,A.period_name) as MINNumberOfDays ,
ROW_NUMBER() over (partition by A.team_name,A.Period_name order by A.NUMBER_OF_DAYS DESC ) as  rankingFromTop 
from (
select employee_code,period_name,team_name, sum(number_of_days) as number_of_days from stg_act_days
where daytype_name <> 'Field Day' 
group by 
employee_code,period_name,team_name
)  A inner join dim_act_employee B
ON a.employee_code=B.employee_code

--5 employees who took most leaves BY period
--SQL: Temp Table/Rank()/Pivot table

Select  *
into #MyTempTable 
from
(
SELECT * FROM (
select A.employee_code,B.EMPLOYEE_NAME, A.team_name,A.period_name,number_of_days,
DENSE_Rank() over (partition by A.team_name,A.Period_name order by A.NUMBER_OF_DAYS asc) as rankingFromTop 
from (
select employee_code,period_name,team_name, sum(number_of_days) as number_of_days from stg_act_days
where daytype_name <> 'Field Day'  and period_name >='2017-08' 
group by 
employee_code,period_name,team_name
)  A inner join dim_act_employee B
ON a.employee_code=B.employee_code
) A
WHERE rankingFromTop  <=5) as b

select * from #MyTempTable
SELECT EMPLOYEE_CODE,EMPLOYEE_NAME,[2017-08],[2017-09],[2017-10],[2017-11],[2017-12],[2018-01],[2018-02],[2018-03],[2018-04],[2018-05],[2018-06],[2018-07],[2018-08],rankingFromTop
FROM ( SELECT * FROM #MyTempTable ) AS SourceTable
pivot (
sum(SourceTable.number_of_days) for period_name in
 ([2017-08],[2017-09],[2017-10],[2017-11],[2017-12],[2018-01],[2018-02],[2018-03],[2018-04],[2018-05],[2018-06],[2018-07],[2018-08])) as PivotTable
