# A08_数据库语句

## DML语句

### 限制只能指定条件插入

with check option 

### MERGE语句语法

| 数据源 | 目标表   |
| ------ | -------- |
| emp    | copy_emp |

```sql
--创建目标表
create table copy_tmp as select * from emp where 1=0;

--matched-->目标表中的主键值在数据源中被找到
--matched-->目标表中的主键值在数据源中未被找到，就insert
merge into copy_tmp c
using emp e
on(c.empno=e.empno)
when matched then
update set
c.ename=e.ename,
c.job=e.job,
c.mgr=e.mgr,
c.hiredate=e.hiredate,
c.sal=e.sal,
c.comm=e.comm
c.deptno=e.deptno
when not matched then
insert values(
e.ename,
e.job,
e.mgr,
e.hiredate,
e.sal,
e.comm
e.deptno
);
```

## 数据库事务

- 从第一条DML开始
- 以下面的事件结束
  - commit、rollback
  - DDL、DCL（自动提交）
  - SQLplus退出
  - 系统崩溃

