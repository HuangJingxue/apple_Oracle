# Z1_Oracle数据泵导入导出

用途：

以逻辑结构为单位进行的备份

跨用户移动数据

跨数据库移动数据库

为测试保存源是的数据状态

对数据库进行版本升级



逻辑导出的注意事项：

执行逻辑导出时一定要注意字符集！最好使用包含中文的小表进行测试

```shell
SQL> select * from nls_database_parameters;

PARAMETER															 VALUE
-------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
NLS_RDBMS_VERSION														 12.2.0.1.0
NLS_NCHAR_CONV_EXCP														 FALSE
NLS_LENGTH_SEMANTICS														 BYTE
NLS_COMP															 BINARY
NLS_DUAL_CURRENCY														 $
NLS_TIMESTAMP_TZ_FORMAT 													 DD-MON-RR HH.MI.SSXFF AM TZR
NLS_TIME_TZ_FORMAT														 HH.MI.SSXFF AM TZR
NLS_TIMESTAMP_FORMAT														 DD-MON-RR HH.MI.SSXFF AM
NLS_TIME_FORMAT 														 HH.MI.SSXFF AM
NLS_SORT															 BINARY
NLS_DATE_LANGUAGE														 AMERICAN

PARAMETER															 VALUE
-------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
NLS_DATE_FORMAT 														 DD-MON-RR
NLS_CALENDAR															 GREGORIAN
NLS_NUMERIC_CHARACTERS														 .,
NLS_NCHAR_CHARACTERSET														 AL16UTF16
NLS_CHARACTERSET														 AL32UTF8
NLS_ISO_CURRENCY														 AMERICA
NLS_CURRENCY															 $
NLS_TERRITORY															 AMERICA
NLS_LANGUAGE															 AMERICAN

SQL> !echo $NLS_LANG


SQL> !echo $LANG
en_US.UTF-8

[root@Oracle01 oracle]# echo $LANG
en_US.UTF-8


INSERT INTO hr.employees(employee_id,first_name,last_name,email,hire_date,job_id) VALUES (207,'黄','雪','hh@163.com',sysdate,'MK_REP');



```





## 创建备份用户和备份目录

```shell
SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 ORCLPDB			  MOUNTED
SQL> alter pluggable database ORCLPDB open;

Pluggable database altered.

SQL> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 ORCLPDB			  READ WRITE NO
SQL> alter session set container=ORCLPDB; 

Session altered.

SQL> alter user hr identified by hr;

User altered.

SQL> select username from dba_users where INHERITED='NO';

USERNAME
--------------------------------------------------------------------------------
PDBADMIN
HR

SQL> select owner,table_name from dba_tables where owner='HR';                                             

OWNER
--------------------------------------------------------------------------------
TABLE_NAME
--------------------------------------------------------------------------------
HR
REGIONS

HR
LOCATIONS

HR
DEPARTMENTS


OWNER
--------------------------------------------------------------------------------
TABLE_NAME
--------------------------------------------------------------------------------
HR
JOBS

HR
EMPLOYEES

HR
JOB_HISTORY


OWNER
--------------------------------------------------------------------------------
TABLE_NAME
--------------------------------------------------------------------------------
HR
TB2

HR
TB1

HR
TB3


OWNER
--------------------------------------------------------------------------------
TABLE_NAME
--------------------------------------------------------------------------------
HR
TBEMP2

HR
TBEMP

HR
TBEMP1


OWNER
--------------------------------------------------------------------------------
TABLE_NAME
--------------------------------------------------------------------------------
HR
COUNTRIES


13 rows selected.

SQL> grant dba to dp identified by dp;

Grant succeeded.

SQL> create or replace directory dp_dir as '/home/oracle';

Directory created.

SQL> grant read,write on directory dp_dir to dp;

Grant succeeded.


```

## 执行备份

```shell
# hr对备份目录没有权限
[oracle@Oracle01 admin]$ expdp hr/hr@PDB1 directory=dp_dir dumpfile=hr_PDB1.dmp logfile=hr_PDB1.log schemas=hr

Export: Release 12.2.0.1.0 - Production on Mon Dec 16 13:43:18 2019

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
ORA-39002: invalid operation
ORA-39070: Unable to open the log file.
ORA-39087: directory name DP_DIR is invalid


[oracle@Oracle01 admin]$ expdp dp/dp@PDB1 directory=dp_dir dumpfile=hr_PDB1.dmp logfile=hr_PDB1.log schemas=hr

Export: Release 12.2.0.1.0 - Production on Mon Dec 16 13:46:08 2019

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
Starting "DP"."SYS_EXPORT_SCHEMA_01":  dp/********@PDB1 directory=dp_dir dumpfile=hr_PDB1.dmp logfile=hr_PDB1.log schemas=hr 
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Processing object type SCHEMA_EXPORT/STATISTICS/MARKER
Processing object type SCHEMA_EXPORT/USER
Processing object type SCHEMA_EXPORT/SYSTEM_GRANT
Processing object type SCHEMA_EXPORT/ROLE_GRANT
Processing object type SCHEMA_EXPORT/DEFAULT_ROLE
Processing object type SCHEMA_EXPORT/TABLESPACE_QUOTA
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/SEQUENCE/SEQUENCE
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/COMMENT
Processing object type SCHEMA_EXPORT/PROCEDURE/PROCEDURE
Processing object type SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
Processing object type SCHEMA_EXPORT/VIEW/VIEW
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/TRIGGER
. . exported "HR"."EMPLOYEES"                            17.08 KB     107 rows
. . exported "HR"."LOCATIONS"                            8.437 KB      23 rows
. . exported "HR"."JOB_HISTORY"                          7.195 KB      10 rows
. . exported "HR"."JOBS"                                 7.109 KB      19 rows
. . exported "HR"."DEPARTMENTS"                          7.125 KB      27 rows
. . exported "HR"."COUNTRIES"                            6.367 KB      25 rows
. . exported "HR"."TB2"                                  5.546 KB       6 rows
. . exported "HR"."REGIONS"                              5.546 KB       4 rows
. . exported "HR"."TB1"                                  5.515 KB       4 rows
. . exported "HR"."TB3"                                      0 KB       0 rows
. . exported "HR"."TBEMP"                                    0 KB       0 rows
. . exported "HR"."TBEMP1"                                   0 KB       0 rows
. . exported "HR"."TBEMP2"                                   0 KB       0 rows
Master table "DP"."SYS_EXPORT_SCHEMA_01" successfully loaded/unloaded
******************************************************************************
Dump file set for DP.SYS_EXPORT_SCHEMA_01 is:
  /home/oracle/hr_PDB1.dmp
Job "DP"."SYS_EXPORT_SCHEMA_01" successfully completed at Mon Dec 16 13:47:55 2019 elapsed 0 00:01:41


```

## 查看导出文件

```shell
[root@Oracle01 oracle]# ls -l hr_PDB1.dmp hr_PDB1.log 
-rw-r-----. 1 oracle oinstall 802816 Dec 16 13:47 hr_PDB1.dmp
-rw-r--r--. 1 oracle oinstall   2851 Dec 16 13:47 hr_PDB1.log

strings hr_PDB1.dmp 查看里面存储内容

```

## 测试导出文件能否正常导入

```shell
SQL> select count(*) from hr.employees;

  COUNT(*)
----------
       107

SQL> drop user hr cascade;
drop user hr cascade
*
ERROR at line 1:
ORA-01940: cannot drop a user that is currently connected  --删除不掉，由于还有会话连接

select username,serial#, sid from v$session;  ---查询用户会话
alter system kill session 'serial#, sid ';---删除相关用户会话

SQL> drop user hr cascade;

User dropped.

SQL> select count(*) from hr.employees;
select count(*) from hr.employees
                        *
ERROR at line 1:
ORA-00942: table or view does not exist

```

导入HR用户

```shell
[oracle@Oracle01 admin]$ impdp dp/dp@PDB1 directory=dp_dir dumpfile=hr_PDB1.dmp logfile=hr_PDB1.log schemas=hr

Import: Release 12.2.0.1.0 - Production on Mon Dec 16 14:14:40 2019

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
Master table "DP"."SYS_IMPORT_SCHEMA_01" successfully loaded/unloaded
Starting "DP"."SYS_IMPORT_SCHEMA_01":  dp/********@PDB1 directory=dp_dir dumpfile=hr_PDB1.dmp logfile=hr_PDB1.log schemas=hr 
Processing object type SCHEMA_EXPORT/USER
Processing object type SCHEMA_EXPORT/SYSTEM_GRANT
Processing object type SCHEMA_EXPORT/ROLE_GRANT
Processing object type SCHEMA_EXPORT/DEFAULT_ROLE
Processing object type SCHEMA_EXPORT/TABLESPACE_QUOTA
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/SEQUENCE/SEQUENCE
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
. . imported "HR"."EMPLOYEES"                            17.08 KB     107 rows
. . imported "HR"."LOCATIONS"                            8.437 KB      23 rows
. . imported "HR"."JOB_HISTORY"                          7.195 KB      10 rows
. . imported "HR"."JOBS"                                 7.109 KB      19 rows
. . imported "HR"."DEPARTMENTS"                          7.125 KB      27 rows
. . imported "HR"."COUNTRIES"                            6.367 KB      25 rows
. . imported "HR"."TB2"                                  5.546 KB       6 rows
. . imported "HR"."REGIONS"                              5.546 KB       4 rows
. . imported "HR"."TB1"                                  5.515 KB       4 rows
. . imported "HR"."TB3"                                      0 KB       0 rows
. . imported "HR"."TBEMP"                                    0 KB       0 rows
. . imported "HR"."TBEMP1"                                   0 KB       0 rows
. . imported "HR"."TBEMP2"                                   0 KB       0 rows
Processing object type SCHEMA_EXPORT/TABLE/COMMENT
Processing object type SCHEMA_EXPORT/PROCEDURE/PROCEDURE
Processing object type SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
Processing object type SCHEMA_EXPORT/VIEW/VIEW
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/TRIGGER
Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Processing object type SCHEMA_EXPORT/STATISTICS/MARKER
Job "DP"."SYS_IMPORT_SCHEMA_01" successfully completed at Mon Dec 16 14:15:04 2019 elapsed 0 00:00:22

```

测试导入结果

```shell
SQL> select count(*) from hr.employees;

  COUNT(*)
----------
       107

```

