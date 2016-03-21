Sometimes, the number of cursors in the database exceeds the maximum limit, and you get the following error:

```
java.sql.SQLException: ORA-01000: maximum open cursors
```

Display which queries are causing maxing out of open cursors:

```sql
select s.username, s.sid, s.serial#, count(*)
 from v$sesstat a, v$statname b, v$session s, v$open_cursor o
where o.sid = s.sid
  and a.statistic# = b.statistic#
  and s.sid=a.sid
  and b.name = 'opened cursors current'
  and s.username is not null
group by s.username, s.sid, s.serial#
```
Monitor v$sesstat to see current maximum opened cursors:

```sql
select max(a.value) as highest_open_cur, p.value as max_open_cur
  from v$sesstat a, v$statname b, v$parameter p
 where a.statistic# = b.statistic#
   and b.name = 'opened cursors current'
   and p.name = 'open_cursors'
 group by p.value;
```

Oracle docs link: http://docs.oracle.com/cd/E40329_01/admin.1112/e27149/cursor.htm
