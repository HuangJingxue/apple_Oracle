select table_name,high_value from dba_tab_partitions where (table_name,partition_position) in 
 (select table_name,max(partition_position) from dba_tab_partitions group
