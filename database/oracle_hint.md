reference  http://www.cnblogs.com/jimeper/archive/2008/07/09/1238763.html


 (1)LC的命中率:

.计算公式:Library Cache Hit Ratio = sum(pinhits) / sum(pins)

SELECTSUM(pinhits)/sum(pins)   
FROM V$LIBRARYCACHE

通常在98%以上，否则，需要要考虑加大共享池，绑定变量，修改cursor_sharing等参数。

.计算共享池内存使用率:

SELECT(1- ROUND(BYTES /(&TSP_IN_M *1024*1024),2))*100|| '%'
 FROM V$SGASTAT
 WHERENAME= 'free memory'
   AND POOL = 'shared pool';

其中: &TSP_IN_M是你的总的共享池的SIZE(M)

共享池内存使用率，应该稳定在75%-90%间，太小浪费内存，太大则内存不足。

查询空闲的共享池内存:

SELECT*FROM V$SGASTAT 
 WHERENAME= 'free memory'
   AND POOL = 'shared pool';

(2)PGA的命中率:

计算公式:BP x 100 / (BP + EBP)

BP: bytes processed

EBP: extrabytesread/written

SELECT*FROM V$PGASTAT WHERENAME='cache hit percentage';

或者从OEM的图形界面中查看

我们可以查看一个视图以获取Oracle的建议值:

SELECT round(PGA_TARGET_FOR_ESTIMATE/1024/1024) target_mb,

       ESTD_PGA_CACHE_HIT_PERCENTAGE cache_hit_perc,

       ESTD_OVERALLOC_COUNT

 FROM V$PGA_TARGET_ADVICE;

The output of this query might look like the following:

 TARGET_MB CACHE_HIT_PERC ESTD_OVERALLOC_COUNT

---------- -------------- --------------------

  63              23                   367

  125              24                    30

  250              30                     3

  375              39                     0

  500              58                     0

  600              59                     0

  700              59                     0

  800              60                     0

  900              60                     0

 1000              61                     0

 1500              67                     0

 2000              76                     0

 3000              83                     0

 4000              85                     0

在此例中:PGA至少要分配375M

我个人认为PGA命中率不应该低于50%

以下的SQL统计sql语句执行在三种模式的次数: optimal memory size, one-pass memory size, multi-pass memory size:

SELECT name profile, cnt, decode(total, 0, 0, round(cnt*100/total,4)) percentage
FROM (SELECT name, value cnt, (sum(value) over ()) total FROM V$SYSSTAT WHERE name like 'workarea exec%');



计算在内存中排序的比率:

SELECT * FROM v$sysstat t WHERE NAME='sorts (memory)'—查询内存排序数
--
SELECT * FROM v$sysstat t WHERE NAME='sorts (disk)'—查询磁盘排序数
--caculate sort in memory ratio
SELECT round(&sort_in_memory/(&sort_in_memory+&sort_in_disk),4)*100||'%' FROM dual

此比率越大越好,太小整要考虑调整,加大PGA

(3)db buffer cache命中率:

计算公式:hit ratio = 1 - [physical reads/(block gets + consistent gets)]

SELECT NAME, PHYSICAL_READS, DB_BLOCK_GETS, CONSISTENT_GETS,
      1 - (PHYSICAL_READS / (DB_BLOCK_GETS + CONSISTENT_GETS)) "Hit Ratio"
FROM V$BUFFER_POOL_STATISTICS
WHERE NAME='DEFAULT';

通常应在90%以上，否则，需要调整,加大DB_CACHE_SIZE

另外一种计算命中率的方法(摘自ORACLE官方文档<<数据库性能优化>>):

SELECT NAME, VALUE
 FROM V$SYSSTAT
 WHERE NAME IN('session logical reads', 
               'physical reads', 
               'physical reads direct',
               'physical reads direct (lob)', 
               'db block gets', 'consistent gets');

命中率的计算公式为: Hit Ratio = 1 - ((physical reads - physical reads direct - physical reads direct (lob)) /

(db block gets + consistent gets - physical reads direct - physical reads direct (lob))

分别代入上一查询中的结果值,就得出了Buffer cache的命中率

-----------------------------------------------------------------------------

几个常用的检查语句

1.       查找排序最多的SQL:

SELECT HASH_VALUE, SQL_TEXT, SORTS, EXECUTIONS
  FROM V$SQLAREA
 ORDER BY SORTS DESC;

2.       查找磁盘读写最多的SQL:

SELECT * FROM

(SELECT sql_text,disk_reads "total disk" ,

executions "total exec",disk_reads/executions "disk/exec" FROM v$sql WHERE executions>0

and is_obsolete='N' ORDER BY 4 desc)

WHERE ROWNUM<11

3.       查找工作量最大的SQL(实际上也是按磁盘读写来排序的):

select
substr(to_char(s.pct, '99.00'), 2) || '%' load,
s.executions executes,
p.sql_text
from
( 
select
address,
disk_reads,
executions,
pct,
rank() over (order by disk_reads desc) ranking
from
(
select
address,
disk_reads,
executions,
100 * ratio_to_report(disk_reads) over () pct
from
sys.v_$sql
where
command_type != 47
)
where
disk_reads > 50 * executions
) s,
sys.v_$sqltext p
where
s.ranking <= 5 and
p.address = s.address
order by
1, s.address, p.piec

 4.用下列SQL工具找出低效SQL: 
SELECT EXECUTIONS , DISK_READS, BUFFER_GETS, 

ROUND((BUFFER_GETS-DISK_READS)/BUFFER_GETS,2) Hit_radio, 

ROUND(DISK_READS/EXECUTIONS,2) Reads_per_run, 

SQL_TEXT 

FROM V$SQLAREA 

WHERE EXECUTIONS>0 

AND BUFFER_GETS > 0 

AND (BUFFER_GETS-DISK_READS)/BUFFER_GETS < 0.8 

ORDER BY 4 DESC;

