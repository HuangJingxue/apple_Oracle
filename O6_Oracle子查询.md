回顾：

- 单行函数：一行一处理

- 多行函数：分组处理
- 单行子查询：
- 多行子查询：

练习内容：

- 子查询用括号括起来

- 子查询放在比较操作符的右边，可读性高

- 单行比较操作符对应比较操作

重点：

子查询返回多个结果只能以下三种方式：ANY 、ALL、IN

```sql
--每个部门的最低薪水的员工信息
select t1.employee_id,t1.last_name,t1.salary,t1.department_id 
from employees t1
join
(select department_id,min(salary) minsal from employees
group by department_id)t2
on t1.department_id=t2.department_id
and t1.salary=t2.minsal;


select t1.employee_id,t1.last_name,t1.salary,t1.department_id 
from employees t1
where(t1.DEPARTMENT_ID,t1.salary) in
(select department_id,min(salary) minsal from employees
group by department_id);


--判断存在性  难点
select * from employees t  --这是取最小值
where not exists
(select 1 from employees a where a.department_id=t.department_id and a.salary>t.salary) -- 有记录返回则标记为true
子查询比主查询大的，主查询取反就取最小值


--not in中有null值结果跟想象不同,不建议not in 效率不高
select employee_id,manager_id,last_name from EMPLOYEES where manager_id not in (101,102,103,null);

select employee_id,manager_id,last_name from EMPLOYEES where manager_id not in (where manager_id is not null);


-- in 依次比较有对应就吐出来
select employee_id,manager_id,last_name from EMPLOYEES where manager_id in (101,102,103,null);
```



