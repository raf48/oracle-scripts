###Simple SQL script to generate Automatic Workload Repository (AWR) report

Select SNAP_ID:
```sql
select
  snap_id,  
  end_interval_time
from
  dba_hist_snapshot
order by
  end_interval_time desc;
```

Select AWR (HTML):
```sql
select
  *
from
  table(
    sys.dbms_workload_repository.awr_report_html(
      (select dbid from v$database),
      1,
      &begin_snapshot,
      &end_snapshot)
  );
```

To create a manual snapshot execute:
```sql
begin
  dbms_workload_repository.create_snapshot;
end;
```

You can also use Oracle supplied SQL scripts from:
- $ORACLE_HOME/rdbms/admin/awrrpt.sql
- $ORACLE_HOME/rdbms/admin/awrrpti.sql
- $ORACLE_HOME/rdbms/admin/awrinput.sql
