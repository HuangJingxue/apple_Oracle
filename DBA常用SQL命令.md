创建表，插入数据
```sql
SQL> create table employees(employee_id number(3) not null,first_name varchar2(15),last_name varchar2(15) not null ,salary number(6,2));

Table created.

SQL> insert into employees (employee_id,first_name,last_name,salary) values (1,'apple','huang',8000);

1 row created.

SQL> insert into employees (employee_id,first_name,last_name,salary) values (2,'qinxi','huang',8000);

1 row created.
```
