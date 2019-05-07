# ORACLE子查询

1.工资高于BLAKE的？

```sql
SQL> select ename from emp where sal > (select sal from emp where ename='BLAKE');

ENAME
--------------------
JONES
SCOTT
KING
FORD

```

2.工资最低的人

```sql
SQL> select ename from emp where sal=(select min(sal) from emp);

ENAME
--------------------
SMITH

```

3.低于10部门最低工资的人？

```sql
SQL> select ename from emp where sal< (select min(sal) from emp where deptno=10);

ENAME
--------------------
SMITH
WARD
MARTIN
ADAMS
JAMES

```

4. 高于30部门最高工资的人

```sql
SQL> select ename from emp where sal> (select max(sal) from emp where deptno=30);

ENAME
--------------------
JONES
SCOTT
KING
FORD
```

5.工资相同的人？

```sql
SQL> select a.ename,a.sal from emp a,emp b where a.sal=b.sal and a.ename!=b.ename;

ENAME			    SAL
-------------------- ----------
MARTIN			   1250
WARD			   1250
FORD			   3000
SCOTT			   3000

```

6.blake的工资是smith的几倍？

```sql
SQL> select (select sal from emp where ename='BLAKE')/(select sal from emp where ename='SMITH') from dual;

(SELECTSALFROMEMPWHEREENAME='BLAKE')/(SELECTSALFROMEMPWHEREENAME='SMITH')
-------------------------------------------------------------------------
								   3.5625

```

7.每个部门工资最高的人？

```sql
SQL> select ename,sal,deptno from emp where sal in (select max(sal) from emp group by deptno) order by deptno;

ENAME			    SAL     DEPTNO
-------------------- ---------- ----------
KING			   5000 	10
FORD			   3000 	20
SCOTT			   3000 	20
BLAKE			   2850 	30

```

8.每个部门工资最高的前2个人？

```sql
SQL> select * from (select ename,deptno,sal,rank () over (partition by deptno order by sal desc) ord from emp) where ord<=2 ;

ENAME			 DEPTNO        SAL	  ORD
-------------------- ---------- ---------- ----------
KING			     10       5000	    1
CLARK			     10       2450	    2
SCOTT			     20       3000	    1
FORD			     20       3000	    1
BLAKE			     30       2850	    1
ALLEN			     30       1600	    2

6 rows selected.
```

9.工资最高的前5行？

```sql
SQL> select * from (select ename,deptno,sal,rank () over ( order by sal desc) ord from emp) where ord<=5 ;

ENAME			 DEPTNO        SAL	  ORD
-------------------- ---------- ---------- ----------
KING			     10       5000	    1
SCOTT			     20       3000	    2
FORD			     20       3000	    2
JONES			     20       2975	    4
BLAKE			     30       2850	    5

```

10.工资6～10名？

```sql
SQL> select * from (select ename,deptno,sal,rank () over ( order by sal desc) ord from emp) where 6<=ord and ord<=10;

ENAME			 DEPTNO        SAL	  ORD
-------------------- ---------- ---------- ----------
CLARK			     10       2450	    6
ALLEN			     30       1600	    7
TURNER			     30       1500	    8
MILLER			     10       1300	    9
WARD			     30       1250	   10
MARTIN			     30       1250	   10

6 rows selected.

```
11.分页查询：每页5行
```sql
SQL> select * from (select rownum rn,a.* from (select ename,sal from emp order by sal desc)a) where rn between &p*5-5+1 and &P+4;
Enter value for p: 1
Enter value for p: 1
old   1: select * from (select rownum rn,a.* from (select ename,sal from emp order by sal desc)a) where rn between &p*5-5+1 and &P+4
new   1: select * from (select rownum rn,a.* from (select ename,sal from emp order by sal desc)a) where rn between 1*5-5+1 and 1+4

	RN ENAME		       SAL
---------- -------------------- ----------
	 1 KING 		      5000
	 2 FORD 		      3000
	 3 SCOTT		      3000
	 4 JONES		      2975
	 5 BLAKE		      2850


```

11.随机从表中取出3行数据？

```sql
SQL> select * from (select ename,sal,deptno from emp order by dbms_random.value()) where rownum<=3;

ENAME			    SAL     DEPTNO
-------------------- ---------- ----------
ALLEN			   1600 	30
TURNER			   1500 	30
JAMES			    950 	30
```

12.查询雇员的姓名，工资，税，(1级不缴税，2-->2% ,3-->3%,4-->4%,5-->5%)

```sql
SQL> select ename,sal,grade,decode(grade,1,0,2,sal*0.02,3,sal*0.03,4,sal*0.04,5,sal*0.05) T from emp,salgrade where emp.sal between salgrade.losal and salgrade.hisal;

ENAME			    SAL      GRADE	    T
-------------------- ---------- ---------- ----------
SMITH			    800 	 1	    0
ADAMS			   1100 	 1	    0
JAMES			    950 	 1	    0
WARD			   1250 	 2	   25
MARTIN			   1250 	 2	   25
MILLER			   1300 	 2	   26
ALLEN			   1600 	 3	   48
TURNER			   1500 	 3	   45
JONES			   2975 	 4	  119
BLAKE			   2850 	 4	  114
CLARK			   2450 	 4	   98

ENAME			    SAL      GRADE	    T
-------------------- ---------- ---------- ----------
SCOTT			   3000 	 4	  120
FORD			   3000 	 4	  120
KING			   5000 	 5	  250

```

13.部门总工资和部门上缴个税总和

```sql
SQL> select deptno,sum(sal),sum(T) from (select deptno,ename,sal,grade,decode(grade,1,0,2,sal*0.02,3,sal*0.03,4,sal*0.04,5,sal*0.05) T from emp,salgrade where emp.sal between salgrade.losal and salgrade.hisal) group by deptno;

    DEPTNO   SUM(SAL)	  SUM(T)
---------- ---------- ----------
	30	 9400	     257
	20	10875	     359
	10	 8750	     374

```

14.比WARD奖金低的人？

```
SQL> select ename,comm from emp where nvl(comm,0) < (select comm from emp where ename='WARD');

ENAME			   COMM
-------------------- ----------
SMITH
ALLEN			    300
JONES
BLAKE
CLARK
SCOTT
KING
TURNER			      0
ADAMS
JAMES
FORD

ENAME			   COMM
-------------------- ----------
MILLER

12 rows selected.

```

15.奖金最高的前两名雇员？
16.工资高于本部门平均工资的人？

oracle pause命令
> 暂停屏幕输出
```sql
# 查看当前pause的状态为off
SQL> show pause;
PAUSE is OFF

# 将pasue设置为开启状态 on
SQL> set pause on;
SQL> show pause;
PAUSE is ON and set to ""

# 查看当前设置的pagesize的大小，即每页显示多少行
SQL> show pagesize;
pagesize 14

# 修改pagesize为10，每页显示10行
SQL> set pagesize 10;

# 执行查询语句
SQL> select rownum rn,ename,sal from emp;
# 需要输入enter健

	RN ENAME	     SAL
---------- ---------- ----------
	 1 SMITH	     800
	 2 ALLEN	    1600
	 3 WARD 	    1250
	 4 JONES	    2975
	 5 MARTIN	    1250
	 6 BLAKE	    2850
	 7 CLARK	    2450
# 需要输入enter健

	RN ENAME	     SAL
---------- ---------- ----------
	 8 SCOTT	    3000
	 9 KING 	    5000
	10 TURNER	    1500
	11 ADAMS	    1100
	12 JAMES	     950
	13 FORD 	    3000
	14 MILLER	    1300

14 rows selected.
```

spool
> oracle的保存执行的sql语句
```sql
# 在但前目录下创建booboo.lst文件保存sql
SQL> spool booboo append
# 执行sql查询
SQL> select * from (select ename,nvl(comm,0),rank () over (order by nvl(comm,0) desc) ord from emp) where ord <= 2;


ENAME	   NVL(COMM,0)	      ORD
---------- ----------- ----------
MARTIN		  1400		1
WARD		   500		2
# 执行sql查询
SQL> select deptno,sum(sal),sum(T) from (select deptno,ename,sal,grade,decode(grade,1,0,2,sal*0.02,3,sal*0.03,4,sal*0.04,5,sal*0.05) T from emp,salgrade where emp.sal between salgrade.losal and salgrade.hisal) group by deptno;


    DEPTNO   SUM(SAL)	  SUM(T)
---------- ---------- ----------
	30	 9400	     257
	20	10875	     359
	10	 8750	     374
# 关闭spool
SQL> spool off

# 查看当前目录下的文档
SQL> host ls
booboo.lst  rlwrap-0.30-1.el5.i386.rpm

# 打印booboo.lst到屏幕上
SQL> host cat booboo.lst
SQL> select * from (select ename,nvl(comm,0),rank () over (order by nvl(comm,0) desc) ord from emp) where ord <= 2;

ENAME      NVL(COMM,0)        ORD                                               
---------- ----------- ----------                                               
MARTIN            1400          1                                               
WARD               500          2                                               

SQL> select deptno,sum(sal),sum(T) from (select deptno,ename,sal,grade,decode(grade,1,0,2,sal*0.02,3,sal*0.03,4,sal*0.04,5,sal*0.05) T from emp,salgrade where emp.sal between salgrade.losal and salgrade.hisal) group by deptno;

    DEPTNO   SUM(SAL)     SUM(T)                                                
---------- ---------- ----------                                                
        30       9400        257                                                
        20      10875        359                                                
        10       8750        374                                                

SQL> spool off
SQL> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options


[oracle@oracle0 ~]$ ls
booboo.lst  rlwrap-0.30-1.el5.i386.rpm
[oracle@oracle0 ~]$ cat booboo.lst 
SQL> select * from (select ename,nvl(comm,0),rank () over (order by nvl(comm,0) desc) ord from emp) where ord <= 2;

ENAME      NVL(COMM,0)        ORD                                               
---------- ----------- ----------                                               
MARTIN            1400          1                                               
WARD               500          2                                               

SQL> select deptno,sum(sal),sum(T) from (select deptno,ename,sal,grade,decode(grade,1,0,2,sal*0.02,3,sal*0.03,4,sal*0.04,5,sal*0.05) T from emp,salgrade where emp.sal between salgrade.losal and salgrade.hisal) group by deptno;

    DEPTNO   SUM(SAL)     SUM(T)                                                
---------- ---------- ----------                                                
        30       9400        257                                                
        20      10875        359                                                
        10       8750        374                                                

SQL> spool off
```
host
> 执行系统命令
```sql
# 执行shell命令 pwd 打印当前路径
SQL> host pwd
/home/oracle

# 执行shell命令 free 查看当前内存使用情况
SQL> host free
             total       used       free     shared    buffers     cached
Mem:       2058756     937308    1121448          0      49720     683304
-/+ buffers/cache:     204284    1854472
Swap:      4095992          0    4095992
```