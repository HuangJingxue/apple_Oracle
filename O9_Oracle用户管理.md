## O9_Oracle用户管理

查看权限相关的数据字典

```shell
desc dict

SQL> select table_name from dict where table_name like '%PRIV%';
DBA_ROLE_PRIVS


SQL> desc v$fixed_table
 Name					   Null?    Type
 ----------------------------------------- -------- ----------------------------
 NAME						    VARCHAR2(128)
 OBJECT_ID					    NUMBER
 TYPE						    VARCHAR2(5)
 TABLE_NUM					    NUMBER
 CON_ID 					    NUMBER

```

用户管理

```shell
# 创建用户
create user qinxi identified by '123' default tablespace TBSAP1 temporary tablesapce tmp account unlock quota unlimitied on tbsap1;

#删除用户
drop user qinxi;

#授权
grant create table to qinxi;

# 授权连接数据库
grant create session to qinxi;
```

查看数据文件名

```shell
select file_name from dba_data_files；

create tabespace c_tbs1 datafile '' size 100m autoextend on next 20m maxsize 20g;
```





system privileges:

```shell
with admin option
```

with grant option



create user contianer all;

grant create table to user contianer all;

```shell
cd $ORACLE_HOME/rdbms/admin
ls -al *pwd*
-rw-r--r--. 1 oracle oinstall 1150 Jul 17  2006 undopwd.sql
-rw-r--r--. 1 oracle oinstall 4462 Jan  6  2016 utlpwdmg.sql
```

show parameter fail;

show parameter pre;



数据库账号密码丢失

```shell
[oracle@Oracle01 admin]$ cd $ORACLE_HOME/dbs
[oracle@Oracle01 dbs]$ ll
total 20
-rw-rw----. 1 oracle oinstall 1544 Oct 13 09:17 hc_cdb1.dat
-rw-r--r--. 1 oracle oinstall 3079 May 15  2015 init.ora
-rw-r-----. 1 oracle oinstall   24 Sep 22 13:36 lkCDB1
-rw-r-----. 1 oracle oinstall 3584 Oct 14 12:24 orapwcdb1        sys用户密码文件
-rw-r-----. 1 oracle oinstall 3584 Oct 20 17:03 spfilecdb1.ora

[oracle@Oracle01 admin]$ orapwd
Usage: orapwd file=<fname> force=<y/n> asm=<y/n>
       dbuniquename=<dbname> format=<12/12.2>
       delete=<y/n> input_file=<input-fname>
       sys=<y/password/external(<sys-external-name>)>
       sysbackup=<y/password/external(<sysbackup-external-name>)>
       sysdg=<y/password/external(<sysdg-external-name>)>
       syskm=<y/password/external(<syskm-external-name>)>
```





