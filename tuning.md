### Tuning

#### Identify plan hash value
~~~
select distinct snap_id,sql_id,sql_plan_hash_value from dba_hist_active_sess_history where sql_id='&1' order by snap_id;
~~~

#### Create tuning task thanks to [DBAClass.com](https://dbaclass.com/article/how-to-run-sql-tuning-advisor-for-a-sql_id/) and [Oraclebase](https://oracle-base.com/articles/11g/automatic-sql-tuning-11gr2)

~~~
DECLARE
l_sql_tune_task_id VARCHAR2(100);
BEGIN
l_sql_tune_task_id := DBMS_SQLTUNE.create_tuning_task (begin_snap => 1,end_snap => 2,
sql_id => '87s8z2zzpsg88',
scope => DBMS_SQLTUNE.scope_comprehensive,
time_limit => 500,
task_name => 'task1',
description => 'Tuning task1 for statement 87s8z2zzpsg88');
DBMS_OUTPUT.put_line('l_sql_tune_task_id: ' || l_sql_tune_task_id);
END;
/
~~~

#### Execute tuning task thanks to [DBAClass.com](https://dbaclass.com/article/how-to-run-sql-tuning-advisor-for-a-sql_id/) and [Oraclebase](https://oracle-base.com/articles/11g/automatic-sql-tuning-11gr2)

~~~
EXEC DBMS_SQLTUNE.execute_tuning_task(task_name => 'task1');
~~~

It can take up to the indicated time in time_limit parameter, 500 sec

#### See the results of tuning task thanks to [DBAClass.com](https://dbaclass.com/article/how-to-run-sql-tuning-advisor-for-a-sql_id/) and [Oraclebase](https://oracle-base.com/articles/11g/automatic-sql-tuning-11gr2)

~~~
set long 65536
set longchunksize 65536
set linesize 100
select dbms_sqltune.report_tuning_task('task1') from dual;
~~~

#### Drop tuning task

~~~
DBMS_SQLTUNE.DROP_TUNING_TASK (task_name => 'task1');
~~~
