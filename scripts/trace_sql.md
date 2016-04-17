###Basic Oracle session trace

Enable the SQL Trace facility for current session by using one of the following:
```sql
begin
  dbms_session.set_sql_trace(true);
end;
```
or:
```sql
alter session set sql_trace = true;
```

Enable SQL Trace in another session:
```sql
declare
  cursor c is
  select a.sid, a.serial#
  from v$session a where a.audsid = (select sys_context('userenv','sessionid') session_id from dual);
  l_sid number;
  l_serial number;
begin
  open c;
  fetch c into l_sid, l_serial;
  close c;
  sys.dbms_system.set_sql_trace_in_session(l_sid, l_serial, true);
end;
```

View location and name of trace file:
```sql
select value from v$diag_info where name = 'Default Trace File';
```

Format SQL Trace output into human readable format.
Run from shell:
```sql
tkprof input.trc output.txt EXPLAIN=user/pass
```