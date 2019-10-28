- SQL指导 https://oracle-base.com/articles/sql/articles-sql

  D80190GC11_les04.ppt

  ```shell
  1. 内置函数（单行函数、多行函数）
  2. 自定义函数
  ```

  ## Case-Conversion Functions

  | Function              | Result     |
  | --------------------- | ---------- |
  | LOWER('SQL Course')   | sql course |
  | UPPER('SQL Course')   | SQL COURSE |
  | INITCAP('SQL Course') | Sql Course |

  -- 索引为何比数据还要大？

  ## Using Character-Manipulation Functions

  | Function                  | Result       |
  | ------------------------- | ------------ |
  | CONCAT('Hello', 'World')  | HelloWorld   |
  | SUBSTR('HelloWorld',1,5)  | Hello        |
  | LENGTH('HelloWorld')      | 10           |
  | INSTR('HelloWorld', 'W')  | 6            |
  | LPAD(last_name,12,’*')    | `*****24000` |
  | RPAD(first_name, 12, ‘*') | `24000*****` |
  | -- 练习                   |              |

  ```sql
  取出book1
  http://www.sina.com.cn/book1.aspx
  
  ```

  ## Nesting Functions

  以上练习可以练习嵌套函数。

  

  ## Numeric Functions

  | Function         | Result | 详解                             |
  | ---------------- | ------ | -------------------------------- |
  | ROUND(45.926, 2) | 45.93  | ROUND(45.926, -1) 为个位舍弃     |
  | TRUNC(45.926, 2) | 45.92  | 将值截断为指定的小数             |
  | CEIL (2.83)      | 3      | 返回大于或等于指定数字的最小整数 |
  | FLOOR (2.83)     | 2      | 返回等于或小于指定数字的最大整数 |
  | MOD (1600, 300)  | 100    | 除法余数                         |
  |                  |        |                                  |

  ##　Working with dates

  RR Date Format：YY与当前世纪保持一致，RR选择与当前世纪距离较近的世纪

  ![1570933439522](C:\Users\Apple\AppData\Roaming\Typora\typora-user-images\1570933439522.png)

  ### Using the SYSDATE Function

-- 练习

```sql
获取昨天
```

## Date-Manipulation Functions

| Function                          | Result      |
| --------------------------------- | ----------- |
| NEXT_DAY   ('01-SEP-05','FRIDAY') | '08-SEP-05' |
| ADD_MONTHS ('31-JAN-04',1)        | '29-FEB-04' |
|                                   |             |

## Implicit Data Type Conversion

```sql
# 字符串数值相互转换

SQL> select to_number('$123,456','$99,999,999,999') from dual;

TO_NUMBER('$123,456','$99,999,999,999')
---------------------------------------
				 123456

SQL> select to_number('$123,456','$99,999,999,999') + 2000 from dual;

TO_NUMBER('$123,456','$99,999,999,999')+2000
--------------------------------------------
				      125456

SQL> select to_char(to_number('$123,456','$99,999,999,999') + 2000,'$99,999,999,999.00') from dual;

TO_CHAR(TO_NUMBER('
-------------------
	$125,456.00
                  
                  
#日期转换                  
SQL> select  to_char(sysdate,'yyyy-mm-dd day year month w ww iw' ) from dual;

select  to_char(sysdate,'yyyy-mm-dd hh24:mi:ss' ) from dual;
TO_CHAR(SYSDATE,'YYYY-MM-DDDAYYEARMONTHWWWIW')
--------------------------------------------------------------------------------
2019-10-13 sunday    twenty nineteen october   2 41 41

                  
SQL> select  to_char(sysdate,'yyyy-mm-dd hh24:mi:ss' ) from dual;

TO_CHAR(SYSDATE,'YY
-------------------
2019-10-13 11:20:13
               

```

## General Functions

- NVL (expr1, expr2)
- NVL2 (expr1, expr2, expr3)
- NULLIF (expr1, expr2)
- COALESCE (expr1, expr2, ..., exprn)   返回第一个不为空的

## Conditional Expressions

```sql
# 尽量不要用传统写法

select t.employee_id,t.last_name,t.salary,
case 
    WHEN t.SALARY >=0 and t.salary <= 1000 THEN '1*t.salary'
else
'15'
end
from hr.EMPLOYEES t;

--练习
行列转换，求出每个部门不同等级有多少人？
```

## Group Functions

> 组函数对行集进行操作，为每个组提供一个结果。
>
> 放在select list中的list，必须按照它分组。
>
> 分组时候的过滤

```sql
SELECT   job_id, SUM(salary) PAYROLL
FROM     employees
WHERE    job_id NOT LIKE '%REP%'
GROUP BY job_id
HAVING   SUM(salary) > 13000
ORDER BY SUM(salary);

```

> count知识点

count(1)、count(*) 没有意义

