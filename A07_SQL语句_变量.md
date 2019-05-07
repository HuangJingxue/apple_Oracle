
## 替代变量
先到环境变量中查找，如果没有采用交互模式

*&+标识符*
```sql
SQL> select ename,&c2 from emp;
Enter value for c2: deptno
old   1: select ename,&c2 from emp
new   1: select ename,deptno from emp

ENAME			 DEPTNO
-------------------- ----------
SMITH			     20
ALLEN			     30
WARD			     30
JONES			     20

ENAME			 DEPTNO
-------------------- ----------
MARTIN			     30
BLAKE			     30
CLARK			     10
SCOTT			     20

ENAME			 DEPTNO
-------------------- ----------
KING			     10
TURNER			     30
ADAMS			     20
JAMES			     30

ENAME			 DEPTNO
-------------------- ----------
FORD			     20
MILLER			     10

14 rows selected.
```


*&&+标识符  就会有隐士的define操作作为会话环境变量*
```sql
SQL> select * from (select rownum rn,a.* from (select ename,sal from emp order by sal desc)a) where rn between &&p*5-5+1 and &P+4;
Enter value for p: 1
old   1: select * from (select rownum rn,a.* from (select ename,sal from emp order by sal desc)a) where rn between &&p*5-5+1 and &P+4
new   1: select * from (select rownum rn,a.* from (select ename,sal from emp order by sal desc)a) where rn between 1*5-5+1 and 1+4

	RN ENAME		       SAL
---------- -------------------- ----------
	 1 KING 		      5000
	 2 FORD 		      3000
	 3 SCOTT		      3000
	 4 JONES		      2975
	 5 BLAKE		      2850

```

环境变量