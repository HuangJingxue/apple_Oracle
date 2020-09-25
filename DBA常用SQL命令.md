创建表，插入数据
```sql
SQL> create table employees(employee_id number(3) not null,first_name varchar2(15),last_name varchar2(15) not null ,salary number(6,2));

Table created.

SQL> insert into employees (employee_id,first_name,last_name,salary) values (1,'apple','huang',8000);

1 row created.

SQL> insert into employees (employee_id,first_name,last_name,salary) values (2,'qinxi','huang',8000);

1 row created.
```

```sql
SQL> create table employees1(employee_id number(4) not null,employee_name varchar2(100) not null,salary number(6,2) not null,department_id number(4) not null);

Table created.

insert into employees1 (employee_id,employee_name,salary,department_id) values (10,'apple',800,1);

1 row created.

SQL> insert into employees1 (employee_id,employee_name,salary,department_id) values (11,'leo',900,2);

1 row created.
```
