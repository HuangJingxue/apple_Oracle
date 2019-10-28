# O4_Oracle_Join

oracle 创建测试大表写法

```shell
create table t1 as select  level id,'level-name'||to_char(level) name from dual   connect by level <=1000;
```

