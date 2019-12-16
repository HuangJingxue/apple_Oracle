# O14_redologfile

```shell
SQL> select * from v$log;

    GROUP#    THREAD#  SEQUENCE#      BYTES  BLOCKSIZE	  MEMBERS ARC
---------- ---------- ---------- ---------- ---------- ---------- ---
STATUS		 FIRST_CHANGE# FIRST_TIM NEXT_CHANGE# NEXT_TIME     CON_ID
---------------- ------------- --------- ------------ --------- ----------
	 1	    1	      10  209715200	   512		1 NO
CURRENT 	       1940482 03-NOV-19   1.8447E+19			 0

	 2	    1	       8  209715200	   512		1 YES
INACTIVE	       1850537 22-OCT-19      1901774 03-NOV-19 	 0

	 3	    1	       9  209715200	   512		1 YES
INACTIVE	       1901774 03-NOV-19      1940482 03-NOV-19 	 0


SQL> select * from v$logfile;

    GROUP# STATUS  TYPE
---------- ------- -------
MEMBER
--------------------------------------------------------------------------------
IS_	CON_ID
--- ----------
	 3	   ONLINE
/u01/app/oracle/oradata/cdb1/redo03.log
NO	     0

	 2	   ONLINE
/u01/app/oracle/oradata/cdb1/redo02.log
NO	     0

    GROUP# STATUS  TYPE
---------- ------- -------
MEMBER
--------------------------------------------------------------------------------
IS_	CON_ID
--- ----------

	 1	   ONLINE
/u01/app/oracle/oradata/cdb1/redo01.log
NO	     0

alter system swith logfile;
切换logfile

```

切换redolog 夯住之后处理方式

```shell
cd $ORACLE_BASE
cd diag/rdbms/

```

