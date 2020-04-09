```wiki
运行方法：
sqlplus "/ as sysdba" @./get_oracle_info_sql.sql

文件名：get_oracle_info_sql.sql
```
```sql
/*
  运行方法 sqlplus "/ as sysdba" @./get_oracle_info_sql.sql
*/
 
SET markup html ON spool ON pre off entmap off
 
set term off
set heading on
set verify off
set feedback off
 
set linesize 2000
set pagesize 30000
set long 999999999
set longchunksize 999999
 
column index_name format a30
column table_name format a30
column num_rows format 999999999
column index_type format a24
column num_rows format 999999999
column status format a8
column clustering_factor format 999999999
column degree format a10
column blevel format 9
column distinct_keys format 9999999999
column leaf_blocks format   9999999
column last_analyzed    format a10
column column_name format a25
column column_position format 9
column temporary format a2
column partitioned format a5
column partitioning_type format a7
column partition_count format 999
column program  format a30
column spid  format a6
column pid  format 99999
column sid  format 99999
column serial# format 99999
column username  format a12
column osuser    format a12
column logon_time format  date
column event    format a32
column JOB_NAME        format a30
column PROGRAM_NAME    format a32
column STATE           format a10
column window_name           format a30
column repeat_interval       format a60
column machine format a30
column program format a30
column osuser format a15
column username format a15
column event format a50
column seconds format a10
column sqltext format a100
 
 
SET markup html off
column dbid new_value spool_dbid
column inst_num new_value spool_inst_num
select dbid from v$database where rownum = 1;
select instance_number as inst_num from v$instance where rownum = 1;
column spoolfile_name new_value spoolfile
select 'spool_'||(select name from v$database where rownum=1) ||'_'|| (select instance_number from v$instance where rownum=1)||'_'||to_char(sysdate,'yy-mm-dd_hh24.mi') as spoolfile_name from dual;
spool &&spoolfile..html
 
SET markup html off
set serveroutput on;
exec dbms_output.enable(9999999999);
exec dbms_output.put_line('<html>');
exec dbms_output.put_line('<style>');
exec dbms_output.put_line('th {font:bold 8pt Arial,Helvetica,Geneva,sans-serif; color:White; background:#0066CC;padding-left:4px; padding-right:4px;padding-bottom:2px}');
exec dbms_output.put_line('h1 {margin-bottom:-10;margin-top:20;}');
exec dbms_output.put_line('h2 {text-indent:2em;margin-bottom:-10;margin-top:-10}');
exec dbms_output.put_line('h3 {text-indent:2em;margin-bottom:-10;margin-top:-10}');
exec dbms_output.put_line('h4 {text-indent:3em;margin-bottom:-10;margin-top:-10}');
exec dbms_output.put_line('h5 {text-indent:4em;margin-bottom:-10;margin-top:-10}');
exec dbms_output.put_line('h6 {text-indent:5em;margin-bottom:-10;margin-top:-10}');
exec dbms_output.put_line('</style>');
exec dbms_output.put_line('<head>');
exec dbms_output.put_line('</head>');
exec dbms_output.put_line('<body>');
 
 
SET markup html on
prompt <h1 align="center">ORACLE数据库评估报告</h1>
prompt <hr>
select to_char(sysdate,'yyyy-mm-dd hh24:mi:ss') as report_time from dual;
 
prompt <hr><br>
prompt <h2>数据库基本信息</h2>
prompt <h4>数据库基本配置</h4>
column TITLE format a20
column INFO format a50
 
select 'DB_NAME' AS "TITLE", name AS "INFO" from v$database
UNION ALL
select 'INSTANCE_NAME',instance_name from v$instance
UNION ALL
select 'VERSION',BANNER from v$version where rownum =1
UNION ALL
select parameter,value from nls_database_parameters where parameter in ('NLS_NCHAR_CHARACTERSET', 'NLS_CHARACTERSET')
UNION ALL
SELECT UPPER(name), concat(value/1024/1024,'M') FROM v$parameter WHERE  name = 'memory_max_target'
UNION ALL
select 'DATAGUARD',decode(count(*),1,'FALSE','TRUE') from v$dataguard_config
UNION ALL
select 'DATABASE_ROLE',DATABASE_ROLE from v$database
UNION ALL
SELECT 'RAC',value FROM v$parameter where name = 'cluster_database'
UNION ALL
select 'LOG_MODE',log_mode from v$database
UNION ALL
select 'FORCE_LOGGING',FORCE_LOGGING from v$database
UNION ALL
SELECT 'SUPPLEMENTAL_LOG_DATA_MIN',supplemental_log_data_min FROM v$database
UNION ALL
SELECT 'FLASHBACK_ON',FLASHBACK_ON FROM v$database
UNION ALL
select 'STARTUP_TIME',to_char(startup_time,'yyyy-mm-dd hh24:mi:ss') from v$instance
UNION ALL
select 'LAST_RMAN_BACKUP',to_char(max(end_time),'yyyy-mm-dd hh24:mi:ss') rman_lastcompleted from v$rman_status where status = 'COMPLETED' and object_type like 'DB FULL';
prompt <br>
 
prompt <h4>Schema空间信息</h4>
select 'TOTAL_SIZE' "SCHEMA", round(sum(bytes) / 1024 / 1024 / 1024, 2) "GB"
  from dba_segments
union all
select owner, round(sum(bytes) / 1024 / 1024 / 1024, 2) "GB"
  from dba_segments
 group by owner
 order by 2 desc;
prompt
 
 
prompt <hr><br>
prompt <h2>数据库SQL质量</h2>
prompt <h4>使用资源最多的SQL语句</h4>
prompt <h5>(较高的磁盘读取(disk_reads消耗I/O)和较高的逻辑读取(buffer_gets消耗CPU)被用作衡量标准)</h5>
prompt <h5>总资源耗费(内存读取和磁盘读取)高原因：</h5>
prompt <h6>1)查询的数据量过大导致磁盘读取过多</h6>
prompt <h6>2)没有使用合适的索引，导致过多全表扫描</h6>
select sql_id "SQL ID",
sql_text ,
executions ,
round(elapsed_time / 1000000, 2) "total_time",
decode((round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2)),
0,
0.01,
(round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2))) "time_per_exec",
buffer_gets ,
disk_reads
from (select sql_id,
sql_text,
executions,
elapsed_time,
buffer_gets,
disk_reads
from v$sql
where buffer_gets > 100000
or disk_reads > 100000
order by buffer_gets + 100 * disk_reads DESC)
where rownum <= 5;
prompt
 
prompt <h4>使用CPU最多的SQL语句</h4>
prompt <h5>(较高的逻辑读取(buffer_gets消耗CPU)被用作衡量标准)</h5>
prompt <h5>cpu消耗过大原因：</h5>
prompt <h6>1)主要由于慢sql造成,慢sql包括全表扫描,扫描数据量太大,内存排序,磁盘排序,锁争用等，而这些都会导致cpu线程被大量占用</h6>
select sql_id "SQL ID",
sql_text ,
executions ,
round(elapsed_time / 1000000, 2) "TOTAL_TIME",
decode((round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2)),
0,
0.01,
(round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2))) "TIME_PER_EXEC",
buffer_gets  ,
disk_reads
from (select sql_id, sql_text, executions,elapsed_time, buffer_gets, disk_reads
from v$sql
where buffer_gets > 100000
order by buffer_gets desc)
where rownum <= 5;
prompt
 
prompt <h4>使用磁盘I/O最多的SQL语句</h4>
prompt <h5>(较高的磁盘读取(disk_reads消耗I/O)被用作衡量标准)</h5>
prompt <h5>磁盘IO耗费原因：</h5>
prompt <h6>1)频繁的对磁盘读写，查询时可以考虑添加合适的复合索引减少磁盘读取，写入时可以考虑批量提交，减少磁盘访问次数</h5>
prompt <h6>2)当向设备写入数据块或是从设备读出数据块时,请求都被安置在一个队列中等待完成，减少单位时间内的队列，能有效优化磁盘IO高的问题</h5>
select sql_id  ,
sql_text ,
executions ,
round(elapsed_time / 1000000, 2) "TOTAL_TIME",
decode((round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2)),
0,
0.01,
(round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2))) "TIME_PER_EXEC",
buffer_gets ,
disk_reads
from (select sql_id,
sql_text,
executions,
elapsed_time,
buffer_gets,
disk_reads
from v$sql
where disk_reads > 100000
order by disk_reads desc)
where rownum <= 5;
prompt
 
 
 
prompt <h4>占用总数据库时间最多的SQL语句</h4>
prompt <h5>数据库时间包含SQL解析时间和执行时长</h5>
prompt <h5>总占用数据库时间SQL是整个数据库执行的SQL中的主要组成，需要重点优化</h5>
select sql_id ,
sql_text ,
executions ,
round(elapsed_time / 1000000, 2) "TOTAL_TIME",
decode((round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2)),
0,
0.01,
(round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2))) "TIME_PER_EXEC",
buffer_gets ,
disk_reads
from (select * from v$sql order by elapsed_time desc)
where rownum <= 5;
prompt
 
prompt <h4>平均占用数据库时间最多的SQL语句</h4>
prompt <h5>(仅计入执行次数超过50次的SQL)</h5>
prompt <h5>平均占用数据库时间耗时较多原因：</h5>
prompt <h6>1)查询数据量过大，建议分批查询，减少不必要的字段与关联表</h5>
prompt <h6>2)SQL没有使用合理索引，导致大量磁盘读取，减慢查询速度</h5>
select sql_id "SQL ID",
sql_text ,
executions ,
round(elapsed_time / 1000000, 2) "TOTAL_TIME",
decode((round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2)),
0,
0.01,
(round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2))) "TIME_PER_EXEC",
buffer_gets ,
disk_reads
from (select * from v$sql order by elapsed_time/(decode(EXECUTIONS,0,1,EXECUTIONS)) desc)
where rownum <= 5 and EXECUTIONS>=50;
prompt
 
 
prompt <h4>执行次数(executions)最多的SQL语句</h4>
prompt <h5>执行次数最多的SQL需要着重优化，避免整个数据库被该慢SQL耗费较多资源</h5>
select sql_id "SQL ID",
sql_text ,
executions ,
round(elapsed_time / 1000000, 2) "TOTAL_TIME",
decode((round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2)),
0,
0.01,
(round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2))) "TIME_PER_EXEC",
buffer_gets ,
disk_reads
from (select * from v$sql where executions > 1000 order by executions desc)
where rownum <= 5;
prompt
 
 
 
 
prompt <h4>解析调用最多的SQL语句</h4>
prompt <h5>没有使用游标或者绑定变量的方式执行SQL，导致每次执行SQL时数据库均需要进行执行计划的分析，产生额外不必要的资源耗费</h5>
select sql_id "SQL ID",
sql_text ,
parse_calls ,
executions ,
round(elapsed_time / 1000000, 2) "TOTAL_TIME",
decode((round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2)),
0,
0.01,
(round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2))) "TIME_PER_EXEC",
buffer_gets ,
disk_reads
from (select *
from v$sql
where parse_calls > 1000
order by parse_calls desc)
where rownum <= 5;
prompt
 
 
prompt <h4>使用共享内存最多的SQL语句</h4>
prompt <h5>(使用共享内存大于1048576(bytes)的SQL语句会显示)</h5>
prompt <h5>共享内存耗费较高原因：</h5>
prompt <h6>   1)SQL调用的复合索引中可能存有大量多余的索引字段</h5>
prompt <h6>   2)大量内存排序</h5>
prompt <h6>   3)查询量过大，建议分批次，并使用合适的条件过滤不必要的数据</h5>
select sql_id "SQL ID",
sql_text ,
sharable_mem ,
executions ,
round(elapsed_time / 1000000, 2) "TOTAL_TIME",
decode((round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2)),
0,
0.01,
(round(elapsed_time / 1000000 /
(decode(EXECUTIONS, 0, 1, EXECUTIONS)),
2))) "TIME_PER_EXEC",
buffer_gets ,
disk_reads
from (select *
from v$sql
where sharable_mem > 1048576
order by sharable_mem desc)
where rownum <= 5;
prompt
 
 
prompt <hr><br>
prompt <h2>表及索引</h2>
prompt <h4>TOP 大表</h4>
prompt <h5>(数据库对象占用空间TOP5，可以考虑进行历史数据的归档，或将大表进行分割，减少全表扫描时的SQL耗费)</h5>
select c.owner,
c.segment_name ,
c.MB "SIZE(MB)"
from (select d.owner, d.segment_name, sum(d.bytes) / 1024 / 1024 as MB
from dba_segments d
group by d.owner, segment_name
order by MB DESC) c
where rownum <= 5;
prompt
 
 
 
prompt <h4>热点表</h4>
prompt <h5>(热点表中的数据块是整个数据库中访问最频繁的数据表)</h5>
select b.owner "SCHEMA",
b.segment_name  ,
b.segment_type  ,
b.tablespace_name  ,
s.SQL_ID "SQL ID",
s.sql_text
from v$sqltext s,
(select distinct a.owner,
a.segment_name,
a.segment_type,
a.tablespace_name
from dba_extents a,
(select dbarfil, dbablk
from (select dbarfil, dbablk from x$bh order by tch desc)
where rownum < 11) b
where a.RELATIVE_FNO = b.dbarfil
and a.BLOCK_ID <= b.dbablk
and a.block_id + a.blocks > b.dbablk) b
where s.sql_text like '%' || b.segment_name || '%'
and b.segment_type = 'TABLE'
order by s.hash_value, s.address, s.piece;
prompt
 
 
 
prompt <h4>冗余索引</h4>
prompt <h5>( Oracle中的冗余索引仅指复合索引中左字段和复合索引或单字段索引中最左字段完全相同的索引类型，由于索引占用空间并且会影响对应表的插入和更新效率，故需要判断是否需要删除不必要的索引。)</h5>
select i.TABLE_OWNER  "TAB_OWNER",
i.INDEX_OWNER  "IDX_OWNER",
i.TABLE_NAME  "TAB_NAME",
i.COLUMN_NAME  "COL_NAME",
i.INDEX_NAME  "IDX_NAME",
i.COLUMN_POSITION  "COL_POS",
wm_concat(d.INDEX_NAME) "Redundant_IDX",
d.COLUMN_POSITION "Re_IDXCOL_POS"
from dba_ind_columns i
left join dba_ind_columns d
on i.TABLE_NAME = d.TABLE_NAME
and i.COLUMN_NAME = d.COLUMN_NAME
and i.INDEX_NAME <> d.INDEX_NAME
where
i.COLUMN_POSITION = d.COLUMN_POSITION
AND i.TABLE_OWNER NOT IN ('SYS',
'SYSTEM',
'SYSMAN',
'PUBLIC',
'WMSYS',
'EXFSYS',
'MDSYS',
'OLAPSYS',
'APEX_030200','DBSNMP','CTXSYS','XDB','ORDDATA')
group by i.TABLE_OWNER,
i.TABLE_NAME,
i.COLUMN_NAME,
i.COLUMN_POSITION,
i.INDEX_NAME,
i.INDEX_OWNER,
d.COLUMN_POSITION
order by i.TABLE_OWNER,
i.TABLE_NAME,
i.COLUMN_NAME,
i.COLUMN_POSITION,
i.INDEX_NAME;
prompt
 
 
 
prompt <hr><br>
prompt <h2>数据库对象</h2>
prompt <h4>数据库对象统计信息</h4>
column owner format a20
column object_type format a20
column NUMBER format 9999999
select owner "SCHEMA", object_type, count(*) "NUMBER"
  from dba_objects
 WHERE OWNER NOT IN ('SYS',
                     'SYSTEM',
                     'OUTLN',
                     'CTXSYS',
                     'SYSMAN',
                     'XDB',
                     'MDSYS',
                     'WMSYS',
                     'OLAPSYS',
                     'APEX_030200',
                     'EXFSYS',
                     'ORDSYS',
                     'PUBLIC',
                     'DBSNMP',
                     'ORACLE_OCM',
                     'FLOWS_FILES',
                     'ORDPLUGINS',
                     'ORDDATA',
                     'APPQOSSYS',
                     'SCOTT',
                     'SI_INFORMTN_SCHEMA',
                     'OWBSYS',
                     'OWBSYS_AUDIT')
 group by owner, object_type
 order by OWNER, object_type desc;
prompt
 
 
 
prompt <h4>SCHEMA依赖关系</h4>
column OWNER format a20
column TYPE format a20
column REFERENCED_OWNER format a20
column REFERENCED_OWNER format a20
column NUMBER format 9999999
 
SELECT OWNER, TYPE, REFERENCED_OWNER, count(*) as "NUMBER"
  FROM DBA_DEPENDENCIES
 WHERE OWNER != REFERENCED_OWNER
   AND OWNER NOT IN ('SYS',
                     'SYSTEM',
                     'OUTLN',
                     'CTXSYS',
                     'SYSMAN',
                     'XDB',
                     'MDSYS',
                     'WMSYS',
                     'OLAPSYS',
                     'APEX_030200',
                     'EXFSYS',
                     'ORDSYS',
                     'PUBLIC',
                     'DBSNMP',
                     'ORACLE_OCM',
                     'FLOWS_FILES',
                     'ORDPLUGINS',
                     'ORDDATA',
                     'APPQOSSYS',
                     'SCOTT',
                     'SI_INFORMTN_SCHEMA',
                     'OWBSYS',
                     'OWBSYS_AUDIT')
 group by OWNER, TYPE, REFERENCED_OWNER
 order by OWNER;
prompt
 
 
prompt <h4>业务特殊表统计</h4>
column TYPE format a20
column NUMBER format 9999999
select 'NO_INDEX_TABLE' as "TYPE",count(*) "NUMBER"
  from dba_tables tab
 where not exists (select distinct table_owner, table_name
          from dba_indexes idx
         where tab.owner = idx.table_owner
           and tab.table_name = idx.table_name)
   and OWNER NOT IN ('SYS',
                     'SYSTEM',
                     'OUTLN',
                     'CTXSYS',
                     'SYSMAN',
                     'XDB',
                     'MDSYS',
                     'WMSYS',
                     'OLAPSYS',
                     'APEX_030200',
                     'EXFSYS',
                     'ORDSYS',
                     'PUBLIC',
                     'DBSNMP',
                     'ORACLE_OCM',
                     'FLOWS_FILES',
                     'ORDPLUGINS',
                     'ORDDATA',
                     'APPQOSSYS',
                     'SCOTT',
                     'SI_INFORMTN_SCHEMA',
                     'OWBSYS',
                     'OWBSYS_AUDIT')
UNION ALL
select 'NO_PK_TABLE',count(*) "NUMBER"
  from dba_tables tab
 where not exists (SELECT owner, table_name
          FROM dba_constraints pk
         where constraint_type = 'P'
           and tab.owner = pk.owner
           and tab.table_name = pk.table_name)
   and OWNER NOT IN ('SYS',
                     'SYSTEM',
                     'OUTLN',
                     'CTXSYS',
                     'SYSMAN',
                     'XDB',
                     'MDSYS',
                     'WMSYS',
                     'OLAPSYS',
                     'APEX_030200',
                     'EXFSYS',
                     'ORDSYS',
                     'PUBLIC',
                     'DBSNMP',
                     'ORACLE_OCM',
                     'FLOWS_FILES',
                     'ORDPLUGINS',
                     'ORDDATA',
                     'APPQOSSYS',
                     'SCOTT',
                     'SI_INFORMTN_SCHEMA',
                     'OWBSYS',
                     'OWBSYS_AUDIT');
prompt
 
 
 
 
prompt <h4>Oracle高级特性(LARGE_OBJECT_DATATYPE)</h4>
column SCHEMA format a20
column NUMBER format 9999999
 
SELECT OWNER "SCHEMA", COUNT(*) "NUMBER"
  FROM DBA_LOBS
 WHERE OWNER NOT IN ('SYS',
                     'SYSTEM',
                     'OUTLN',
                     'CTXSYS',
                     'SYSMAN',
                     'XDB',
                     'MDSYS',
                     'WMSYS',
                     'OLAPSYS',
                     'APEX_030200',
                     'EXFSYS',
                     'ORDSYS',
                     'PUBLIC',
                     'DBSNMP',
                     'ORACLE_OCM',
                     'FLOWS_FILES',
                     'ORDPLUGINS',
                     'ORDDATA',
                     'APPQOSSYS',
                     'SCOTT',
                     'SI_INFORMTN_SCHEMA',
                     'OWBSYS',
                     'OWBSYS_AUDIT')
 group by OWNER;
prompt
 
 
 
prompt <h4>未建索引的大表</h4>
select segment_name,
       bytes / 1024 / 1024 / 1024 "GB",
       blocks,
       tablespace_name
  from dba_segments
 where segment_type = 'TABLE'
   and segment_name not in (select table_name from dba_indexes)
   and bytes / 1024 / 1024 / 1024 >= 0.5
 order by GB desc
;
select segment_name, sum(bytes) / 1024 / 1024 / 1024 "GB", sum(blocks)
  from dba_segments
 where segment_type = 'TABLE PARTITION'
   and segment_name not in (select table_name from dba_indexes)
 group by segment_name
having sum(bytes) / 1024 / 1024 / 1024 >= 0.5
 order by GB desc;
prompt
 
 
 
prompt <h4>Schema统计信息</h4>
select 'TOTAL_SIZE' "SCHEMA", round(sum(bytes) / 1024 / 1024 / 1024, 2) "GB"
  from dba_segments
union all
select owner, round(sum(bytes) / 1024 / 1024 / 1024, 2) "GB"
  from dba_segments
 group by owner
 order by 2 desc;
prompt
 
 
prompt <h4>对象大小TOP10</h4>
select *
  from (select owner,
               segment_name,
               segment_type,
               round(sum(bytes) / 1024 / 1024) object_size
          from DBA_segments
         where OWNER NOT IN ('SYS',
                     'SYSTEM',
                     'OUTLN',
                     'CTXSYS',
                     'SYSMAN',
                     'XDB',
                     'MDSYS',
                     'WMSYS',
                     'OLAPSYS',
                     'APEX_030200',
                     'EXFSYS',
                     'ORDSYS',
                     'PUBLIC',
                     'DBSNMP',
                     'ORACLE_OCM',
                     'FLOWS_FILES',
                     'ORDPLUGINS',
                     'ORDDATA',
                     'APPQOSSYS',
                     'SCOTT',
                     'SI_INFORMTN_SCHEMA',
                     'OWBSYS',
                     'OWBSYS_AUDIT')
         group by owner, segment_name, segment_type
         order by object_size desc)
where rownum <= 10;
prompt
 
 
 
 
prompt <h4>表大小超过10GB未建分区的</h4>
select owner,
       segment_name,
       segment_type,
       round(sum(bytes) / 1024 / 1024 / 1024,2) object_size
  from dba_segments
where segment_type = 'TABLE'
  and bytes > 10*1024*1024*1024
  and OWNER NOT IN ('SYS',
                     'SYSTEM',
                     'OUTLN',
                     'CTXSYS',
                     'SYSMAN',
                     'XDB',
                     'MDSYS',
                     'WMSYS',
                     'OLAPSYS',
                     'APEX_030200',
                     'EXFSYS',
                     'ORDSYS',
                     'PUBLIC',
                     'DBSNMP',
                     'ORACLE_OCM',
                     'FLOWS_FILES',
                     'ORDPLUGINS',
                     'ORDDATA',
                     'APPQOSSYS',
                     'SCOTT',
                     'SI_INFORMTN_SCHEMA',
                     'OWBSYS',
                     'OWBSYS_AUDIT')
group by owner, segment_name, segment_type
order by object_size desc;
prompt
 
 
 
prompt <h4>分区最多的前10个对象</h4>
select *
  from (select table_owner, table_name, count(*) cnt
          from dba_tab_partitions
         where table_owner NOT IN ('SYS',
                     'SYSTEM',
                     'OUTLN',
                     'CTXSYS',
                     'SYSMAN',
                     'XDB',
                     'MDSYS',
                     'WMSYS',
                     'OLAPSYS',
                     'APEX_030200',
                     'EXFSYS',
                     'ORDSYS',
                     'PUBLIC',
                     'DBSNMP',
                     'ORACLE_OCM',
                     'FLOWS_FILES',
                     'ORDPLUGINS',
                     'ORDDATA',
                     'APPQOSSYS',
                     'SCOTT',
                     'SI_INFORMTN_SCHEMA',
                     'OWBSYS',
                     'OWBSYS_AUDIT')
         group by table_owner, table_name
         order by cnt desc)
where rownum <= 10;
prompt
 
 
 
prompt <h4>分区不均匀的表</h4>
select *
  from (select table_owner,
               table_name,
               max(num_rows) max_num_rows,
               trunc(avg(num_rows), 0) avg_num_rows,
               sum(num_rows) sum_num_rows,
               case
                 when sum(num_rows) = 0 then
                  0
                 else
                  trunc(max(num_rows) / trunc(avg(num_rows), 0), 2)
               end rate,
               count(*) part_count
          from dba_tab_partitions
         where table_owner NOT IN ('SYS',
                     'SYSTEM',
                     'OUTLN',
                     'CTXSYS',
                     'SYSMAN',
                     'XDB',
                     'MDSYS',
                     'WMSYS',
                     'OLAPSYS',
                     'APEX_030200',
                     'EXFSYS',
                     'ORDSYS',
                     'PUBLIC',
                     'DBSNMP',
                     'ORACLE_OCM',
                     'FLOWS_FILES',
                     'ORDPLUGINS',
                     'ORDDATA',
                     'APPQOSSYS',
                     'SCOTT',
                     'SI_INFORMTN_SCHEMA',
                     'OWBSYS',
                     'OWBSYS_AUDIT')
         group by table_owner, table_name)
 where rate > 5;
prompt
 
 
 
prompt <h4>列数量超过100个或小于2的表</h4>
select *
  from (select owner, table_name, count(*) col_count
          from dba_tab_cols
          where   owner NOT IN ('SYS',
                     'SYSTEM',
                     'OUTLN',
                     'CTXSYS',
                     'SYSMAN',
                     'XDB',
                     'MDSYS',
                     'WMSYS',
                     'OLAPSYS',
                     'APEX_030200',
                     'EXFSYS',
                     'ORDSYS',
                     'PUBLIC',
                     'DBSNMP',
                     'ORACLE_OCM',
                     'FLOWS_FILES',
                     'ORDPLUGINS',
                     'ORDDATA',
                     'APPQOSSYS',
                     'SCOTT',
                     'SI_INFORMTN_SCHEMA',
                     'OWBSYS',
                     'OWBSYS_AUDIT')
         group by owner, table_name)
 where col_count > 100
    or col_count <= 2;
prompt
 
 
 
prompt <h4>游标Cursors</h4>
prompt <h5>查看session_cached_cursors的参数设置情况，如果使用率为100%则增大这个参数值</h5>
SELECT 'session_cached_cursors' PARAMETER, 
         LPAD(VALUE, 5) VALUE, 
         DECODE(VALUE, 0, ' n/a', TO_CHAR(100 * USED / VALUE, '990') || '%') USAGE 
   FROM (SELECT MAX(S.VALUE) USED 
            FROM V$STATNAME N, V$SESSTAT S 
           WHERE N.NAME = 'session cursor cache count' 
             AND S.STATISTIC# = N.STATISTIC#), 
         (SELECT VALUE FROM V$PARAMETER WHERE NAME = 'session_cached_cursors') 
  UNION ALL 
SELECT 'open_cursors', 
         LPAD(VALUE, 5), 
         TO_CHAR(100 * USED / VALUE, '990') || '%' 
   FROM (SELECT MAX(SUM(S.VALUE)) USED 
            FROM V$STATNAME N, V$SESSTAT S 
           WHERE N.NAME IN 
                 ('opened cursors current', 'session cursor cache count') 
             AND S.STATISTIC# = N.STATISTIC# 
           GROUP BY S.SID), 
         (SELECT VALUE FROM V$PARAMETER WHERE NAME = 'open_cursors');
prompt
 
 
prompt <hr><br>
prompt <h2>附录</h2>
prompt <h5>供参考的Oracle所有参数</h5>
show parameter
 
set markup html off
exec dbms_output.put_line('</body>');
exec dbms_output.put_line('</html>');
 
spool off
exit
```
