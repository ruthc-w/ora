### Tablespaces and ASM disks and so on

#### Tbs usage thanks to [ZedDBA](https://blog.zeddba.com/2017/06/27/querying-tablespace-usage)
~~~
set pages 999
set lines 400
SELECT df.tablespace_name tablespace_name,
 max(df.autoextensible) auto_ext,
 round(df.maxbytes / (1024 * 1024), 2) max_ts_size,
 round((df.bytes - sum(fs.bytes)) / (df.maxbytes) * 100, 2) max_ts_pct_used,
 round(df.bytes / (1024 * 1024), 2) curr_ts_size,
 round((df.bytes - sum(fs.bytes)) / (1024 * 1024), 2) used_ts_size,
 round((df.bytes-sum(fs.bytes)) * 100 / df.bytes, 2) ts_pct_used,
 round(sum(fs.bytes) / (1024 * 1024), 2) free_ts_size,
 nvl(round(sum(fs.bytes) * 100 / df.bytes), 2) ts_pct_free
FROM dba_free_space fs,
 (select tablespace_name,
 sum(bytes) bytes,
 sum(decode(maxbytes, 0, bytes, maxbytes)) maxbytes,
 max(autoextensible) autoextensible
 from dba_data_files
 group by tablespace_name) df
WHERE fs.tablespace_name (+) = df.tablespace_name
GROUP BY df.tablespace_name, df.bytes, df.maxbytes
UNION ALL
SELECT df.tablespace_name tablespace_name,
 max(df.autoextensible) auto_ext,
 round(df.maxbytes / (1024 * 1024), 2) max_ts_size,
 round((df.bytes - sum(fs.bytes)) / (df.maxbytes) * 100, 2) max_ts_pct_used,
 round(df.bytes / (1024 * 1024), 2) curr_ts_size,
 round((df.bytes - sum(fs.bytes)) / (1024 * 1024), 2) used_ts_size,
 round((df.bytes-sum(fs.bytes)) * 100 / df.bytes, 2) ts_pct_used,
 round(sum(fs.bytes) / (1024 * 1024), 2) free_ts_size,
 nvl(round(sum(fs.bytes) * 100 / df.bytes), 2) ts_pct_free
FROM (select tablespace_name, bytes_used bytes
 from V$temp_space_header
 group by tablespace_name, bytes_free, bytes_used) fs,
 (select tablespace_name,
 sum(bytes) bytes,
 sum(decode(maxbytes, 0, bytes, maxbytes)) maxbytes,
 max(autoextensible) autoextensible
 from dba_temp_files
 group by tablespace_name) df
WHERE fs.tablespace_name (+) = df.tablespace_name
GROUP BY df.tablespace_name, df.bytes, df.maxbytes
ORDER BY 4 DESC;
~~~

#### ASM disk usage 
~~~
SELECT name, free_mb, total_mb, free_mb/total_mb*100 as percentage,usable_file_mb
FROM v$asm_diskgroup;

~~~

#### DB growth per month
~~~
select to_char(creation_time, 'RRRR-MON') "Month",
sum(bytes)/1024/1024 "MB"
from v$datafile
group by to_char(creation_time, 'RRRR-MON') order by 1;

~~~

#### Archivers size
~~~
col day format a15;
col hour format a4;
col total format 999;

select 
to_char(first_time,'yyyy-mm-dd') day,
to_char(first_time,'hh24') hour,
count(*) total, (count(*)*62914560)/1024/1024 MB_archive -- 62914560 = redo log file size in bytes
from v$log_history
group by to_char(first_time,'yyyy-mm-dd'),to_char(first_time,'hh24')
order by to_char(first_time,'yyyy-mm-dd'),to_char(first_time,'hh24') asc;

~~~

#### Archivers number, no redo log size needed
~~~
set lines 120; 
set pages 999; 
SELECT 
to_char(first_time,'YYYY-MON-DD') day,
to_char(sum(decode(to_char(first_time,'HH24'),'00',1,0)),'9999') "00",
to_char(sum(decode(to_char(first_time,'HH24'),'01',1,0)),'9999') "01",
to_char(sum(decode(to_char(first_time,'HH24'),'02',1,0)),'9999') "02",
to_char(sum(decode(to_char(first_time,'HH24'),'03',1,0)),'9999') "03",
to_char(sum(decode(to_char(first_time,'HH24'),'04',1,0)),'9999') "04",
to_char(sum(decode(to_char(first_time,'HH24'),'05',1,0)),'9999') "05",
to_char(sum(decode(to_char(first_time,'HH24'),'06',1,0)),'9999') "06",
to_char(sum(decode(to_char(first_time,'HH24'),'07',1,0)),'9999') "07",
to_char(sum(decode(to_char(first_time,'HH24'),'08',1,0)),'9999') "0",
to_char(sum(decode(to_char(first_time,'HH24'),'09',1,0)),'9999') "09",
to_char(sum(decode(to_char(first_time,'HH24'),'10',1,0)),'9999') "10",
to_char(sum(decode(to_char(first_time,'HH24'),'11',1,0)),'9999') "11",
to_char(sum(decode(to_char(first_time,'HH24'),'12',1,0)),'9999') "12",
to_char(sum(decode(to_char(first_time,'HH24'),'13',1,0)),'9999') "13",
to_char(sum(decode(to_char(first_time,'HH24'),'14',1,0)),'9999') "14",
to_char(sum(decode(to_char(first_time,'HH24'),'15',1,0)),'9999') "15",
to_char(sum(decode(to_char(first_time,'HH24'),'16',1,0)),'9999') "16",
to_char(sum(decode(to_char(first_time,'HH24'),'17',1,0)),'9999') "17",
to_char(sum(decode(to_char(first_time,'HH24'),'18',1,0)),'9999') "18",
to_char(sum(decode(to_char(first_time,'HH24'),'19',1,0)),'9999') "19",
to_char(sum(decode(to_char(first_time,'HH24'),'20',1,0)),'9999') "20",
to_char(sum(decode(to_char(first_time,'HH24'),'21',1,0)),'9999') "21",
to_char(sum(decode(to_char(first_time,'HH24'),'22',1,0)),'9999') "22",
to_char(sum(decode(to_char(first_time,'HH24'),'23',1,0)),'9999') "23"
from
v$log_history
GROUP by 
to_char(first_time,'YYYY-MON-DD')
order by to_char(first_time,'YYYY-MON-DD');

~~~
