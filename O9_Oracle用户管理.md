## O9_Oracle用户管理

概念：逻辑上对数据库中的海量对象进行分组

```shell
SQL> alter session set container=orclpdb;

Session altered.

SQL> create user apple identified by apple;

User created.

SQL> conn apple/apple; ##该操作会退出当前用户，需要重新登录
ERROR:
ORA-01017: invalid username/password; logon denied


Warning: You are no longer connected to ORACLE.


[oracle@Oracle01 admin]$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Mon Dec 16 18:26:40 2019

Copyright (c) 1982, 2016, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> alter session set container=orclpdb;

Session altered.

SQL> grant create session to apple;

Grant succeeded.


SQL> alter session set container=orclpdb;

Session altered.

SQL> conn apple/apple@PDB1   
Connected.

```

--with admin option 权限回收无级联，系统权限

--with grant option 权限回收有级联，对象权限



角色：一组权限的集合

```shell
SQL> alter session set container=orclpdb;  

Session altered.

SQL> create role r_clerk;

Role created.

SQL> grant create session,create table,create any index to r_clerk;

Grant succeeded.

SQL> grant r_clerk to apple;

Grant succeeded.

SQL> create role r_manager;

Role created.

SQL> grant create any table,r_clerk to r_manager;

Grant succeeded.

```

修改口令

```shell
alter user apple identified by apple;
```



授权练习

- apple用户能够访问hr.employees

```shell

[oracle@Oracle01 admin]$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Mon Dec 16 18:26:40 2019

Copyright (c) 1982, 2016, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> alter session set container=orclpdb;

Session altered.

SQL> grant select on hr.employees to hr;

SQL> select grantee,owner,table_name from dba_tab_privs where grantee='APPLE';

GRANTEE 															 OWNER	    TABLE_NAME
-------------------------------------------------------------------------------------------------------------------------------- ---------- ------------------------------
APPLE																 HR	    EMPLOYEES

SQL> select * from dba_tab_privs where grantee='APPLE';

SQL> alter pluggable database ORCLPDB open;

Pluggable database altered.

SQL> conn apple/apple@PDB1
Connected.
SQL> select * from hr.employees;

EMPLOYEE_ID FIRST_NAME		 LAST_NAME
----------- -------------------- -------------------------
EMAIL			  PHONE_NUMBER	       HIRE_DATE JOB_ID 	SALARY
------------------------- -------------------- --------- ---------- ----------
COMMISSION_PCT MANAGER_ID DEPARTMENT_ID
-------------- ---------- -------------



```











