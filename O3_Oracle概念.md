
```shell
SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 ORCLPDB			  MOUNTED


SQL> alter pluggable database ORCLPDB open;

Pluggable database altered.


SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 ORCLPDB			  READ WRITE NO
	 

# 容器切到ORCLPDB
SQL> alter session set container=ORCLPDB; 

Session altered.

# 修改hr用户密码
SQL> alter user hr identified by hr;

User altered.

# 不晓得作用
SQL> startup open restrict;


# 切回容器
SQL> alter session set container=cdb$root; 
session altered.


# redo undo 都是cdb来维护的


SQL> show con_name
CON_NAME
------------------------------
ORCLPDB

```

