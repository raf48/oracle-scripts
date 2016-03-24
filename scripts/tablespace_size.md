Query to check Tablespace Usage:

```sql
select
  /*+ all_rows */
  fs.tablespace_name                          "Tablespace",
  df.totalspace                               "Total MB",
  (df.totalspace - fs.freespace)              "Used MB",
  fs.freespace                                "Free MB",
  round(100 * (fs.freespace / df.totalspace)) "Pct. Free"
from
  (select tablespace_name, round(sum(bytes) / 1048576) TotalSpace
   from dba_data_files
   group by tablespace_name) df,
  (select tablespace_name, round(sum(bytes) / 1048576) FreeSpace
   from dba_free_space
   group by tablespace_name) fs
where
  df.tablespace_name = fs.tablespace_name
```

Temporary Tablespace Usage:
```sql
select
  a.tablespace_name "Tablespace",
  d.mb_total  "Total MB",
  round(sum(a.used_blocks * d.block_size) / 1048576) "Used MB",
  d.mb_total - round(sum(a.used_blocks * d.block_size) / 1048576) "Free MB"
from
  v$sort_segment a,
  (
    select
      b.name,
      c.block_size,
      sum (c.bytes) / 1024 / 1024 mb_total
    from
      v$tablespace b,
      v$tempfile c
    where
      b.ts#= c.ts#
    group by
      b.name,
      c.block_size
  ) d
where
  a.tablespace_name = d.name
group by
  a.tablespace_name,
  d.mb_total
```