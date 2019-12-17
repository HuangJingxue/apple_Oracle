# Z2_ORACLE体系结构



[Oracle体系结构详解](https://blog.csdn.net/angelxf520/article/details/82432193)



物理文件

```shell
SQL> select name from v$datafile;

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/oradata/cdb1/system01.dbf
/u01/app/oracle/oradata/cdb1/sysaux01.dbf
/u01/app/oracle/oradata/cdb1/undotbs01.dbf
/u01/app/oracle/oradata/cdb1/pdbseed/system01.dbf
/u01/app/oracle/oradata/cdb1/pdbseed/sysaux01.dbf
/u01/app/oracle/oradata/cdb1/users01.dbf
/u01/app/oracle/oradata/cdb1/pdbseed/undotbs01.dbf
/u01/app/oracle/oradata/cdb1/orclpdb/system01.dbf
/u01/app/oracle/oradata/cdb1/orclpdb/sysaux01.dbf
/u01/app/oracle/oradata/cdb1/orclpdb/undotbs01.dbf
/u01/app/oracle/oradata/cdb1/orclpdb/users01.dbf

SQL> select member from v$logfile;

MEMBER
--------------------------------------------------------------------------------
/u01/app/oracle/oradata/cdb1/redo03.log
/u01/app/oracle/oradata/cdb1/redo02.log
/u01/app/oracle/oradata/cdb1/redo01.log


SQL> select name from v$controlfile;

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/oradata/cdb1/control01.ctl
/u01/app/oracle/oradata/cdb1/control02.ctl


SQL> select name from v$archived_log;

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/product/12.2.0.1/db_1/dbs/arch1_10_1019655515.dbf
/u01/app/oracle/product/12.2.0.1/db_1/dbs/arch1_11_1019655515.dbf


SQL> select name,value from v$diag_info;


```
自动跟踪诊断工具

> 命令行工具ADRCI(Automatic Diagnostic Repository Command Interpreter)

```shell
[oracle@Oracle01 ~]$ adrci
adrci> show problem

```
查看oracle报错代码含义
```shell
[oracle@Oracle01 ~]$ oerr ORA 700
00700, 00000, "soft internal error, arguments: [%s], [%s], [%s], [%s], [%s], [%s], [%s], [%s], [%s], [%s], [%s], [%s]"
// *Cause:  Internal inconsistency that will not crash a process
// *Action: Report as a bug - the first argument is the internal error.

```



