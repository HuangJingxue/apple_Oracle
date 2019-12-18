# Z3_RAM备份恢复

恢复管理器：

查看数据库中有什么？
RMAN> report schema;

使用rman对控制文件做镜像备份

RMAN> copy current controlfile to '/home/oracle/rmanbk/control01.ctl';  

查看控制文件文件的镜像备份

RMAN> list copy of controlfile;

使用rman还原丢失的控制文件

RMAN> restore controlfile from '/home/oracle/rmanbk/control01.ctl';

RMAN工具执行挂接数据命令

`RMAN> alter database mount;`

RMAN工具执行恢复数据库命令

`RMAN> recover database;`

RMAN工具执行打开数据库命令

`RMAN> alter database open resetlogs;`




```shell
[oracle@Oracle01 ~]$ rman

Recovery Manager: Release 12.2.0.1.0 - Production on Wed Dec 18 17:03:44 2019

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

RMAN> connect target /

connected to target database: CDB1 (DBID=993372953)

```

```shell
[oracle@Oracle01 ~]$ rman target sys/oracle@PDB1

Recovery Manager: Release 12.2.0.1.0 - Production on Wed Dec 18 17:07:11 2019

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

connected to target database: CDB1:ORCLPDB (DBID=2671496002)

RMAN> report schema;  #查看数据库当中有什么？

using target database control file instead of recovery catalog
Report of database schema for database with db_unique_name CDB1

List of Permanent Datafiles
===========================
File Size(MB) Tablespace           RB segs Datafile Name
---- -------- -------------------- ------- ------------------------
9    260      SYSTEM               NO      /u01/app/oracle/oradata/cdb1/orclpdb/system01.dbf
10   390      SYSAUX               NO      /u01/app/oracle/oradata/cdb1/orclpdb/sysaux01.dbf
11   100      UNDOTBS1             NO      /u01/app/oracle/oradata/cdb1/orclpdb/undotbs01.dbf
12   6        USERS                NO      /u01/app/oracle/oradata/cdb1/orclpdb/users01.dbf

List of Temporary Files
=======================
File Size(MB) Tablespace           Maxsize(MB) Tempfile Name
---- -------- -------------------- ----------- --------------------
3    64       TEMP                 32767       /u01/app/oracle/oradata/cdb1/orclpdb/temp01.dbf

RMAN> 
[oracle@Oracle01 ~]$ rman target /

Recovery Manager: Release 12.2.0.1.0 - Production on Wed Dec 18 18:13:54 2019

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

connected to target database: CDB1 (DBID=993372953)

RMAN> copy current controlfile to '/home/oracle/rmanbk/control01.ctl';

Starting backup at 18-DEC-19
using target database control file instead of recovery catalog
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=63 device type=DISK
channel ORA_DISK_1: starting datafile copy
copying current control file
output file name=/home/oracle/rmanbk/control01.ctl tag=TAG20191218T181625 RECID=4 STAMP=1027361785
channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:01
Finished backup at 18-DEC-19

Starting Control File and SPFILE Autobackup at 18-DEC-19
piece handle=/u01/app/oracle/product/12.2.0.1/db_1/dbs/c-993372953-20191218-04 comment=NONE
Finished Control File and SPFILE Autobackup at 18-DEC-19

RMAN> exit

RMAN-06900: warning: unable to generate V$RMAN_STATUS or V$RMAN_OUTPUT row
RMAN-06901: warning: disabling update of the V$RMAN_STATUS and V$RMAN_OUTPUT rows
ORACLE error from target database: 
ORA-03135: connection lost contact
Process ID: 123420
Session ID: 69 Serial number: 51549


Recovery Manager complete.


[oracle@Oracle01 ~]$ rman target /

Recovery Manager: Release 12.2.0.1.0 - Production on Wed Dec 18 18:21:38 2019

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

connected to target database: CDB1 (not mounted)

RMAN> restore controlfile from '/home/oracle/rmanbk/control01.ctl';

Starting restore at 18-DEC-19
using target database control file instead of recovery catalog
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=36 device type=DISK

channel ORA_DISK_1: copied control file copy
output file name=/u01/app/oracle/oradata/cdb1/control01.ctl
output file name=/u01/app/oracle/oradata/cdb1/control02.ctl
Finished restore at 18-DEC-19

RMAN> alter database mount;

Statement processed
released channel: ORA_DISK_1

RMAN> recover database;

Starting recover at 18-DEC-19
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=36 device type=DISK

starting media recovery

archived log for thread 1 with sequence 5 is already on disk as file /u01/app/oracle/oradata/cdb1/redo02.log
archived log for thread 1 with sequence 6 is already on disk as file /u01/app/oracle/oradata/cdb1/redo03.log
archived log for thread 1 with sequence 7 is already on disk as file /u01/app/oracle/oradata/cdb1/redo01.log
archived log file name=/u01/app/oracle/product/12.2.0.1/db_1/dbs/arch1_1_1027361609.dbf thread=1 sequence=1
archived log file name=/u01/app/oracle/product/12.2.0.1/db_1/dbs/arch1_2_1027361609.dbf thread=1 sequence=2
archived log file name=/u01/app/oracle/product/12.2.0.1/db_1/dbs/arch1_3_1027361609.dbf thread=1 sequence=3
archived log file name=/u01/app/oracle/product/12.2.0.1/db_1/dbs/arch1_4_1027361609.dbf thread=1 sequence=4
archived log file name=/u01/app/oracle/oradata/cdb1/redo02.log thread=1 sequence=5
archived log file name=/u01/app/oracle/oradata/cdb1/redo03.log thread=1 sequence=6
archived log file name=/u01/app/oracle/oradata/cdb1/redo01.log thread=1 sequence=7
media recovery complete, elapsed time: 00:00:00
Finished recover at 18-DEC-19

RMAN> alter database open resetlogs;

Statement processed

RMAN> 



Insert into hr.employees (EMPLOYEE_ID,FIRST_NAME,LAST_NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,JOB_ID,SALARY,COMMISSION_PCT,MANAGER_ID,DEPARTMENT_ID) values (208,'William','Gietz','jh@jiagouyun.com','515.123.8181',sysdate,'AC_ACCOUNT',8300,null,205,110);
```



