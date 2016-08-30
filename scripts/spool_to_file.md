**WARNING:** the following script is not intended to be used in production environments. If there is not enough space in the temporary tablespace you may end up with ORA-01652 (unable to extend temp segment).
Check [Temporary Tablespace Usage](https://github.com/raf48/oracle-scripts/blob/master/scripts/tablespace_size.md) before executing!

Read file from Oracle directory with PL/SQL and spool it to a local file using SQL*Plus:

```sql
set linesize 500
set trimspool on
set feedback off
set heading off
set termout off
set echo off

spool spoolfile.txt

create global temporary table spool_tmp(
  text varchar2(4000),
  line number)
on commit preserve rows
/
declare
  type t_file is table of spool_tmp%rowtype;
  l_file_buffer t_file := t_file();
  l_file utl_file.file_type;
  l_line_buffer varchar2(4000);
  l_i number := 1;
begin
  l_file := utl_file.fopen('ORACLE_DIRECTORY', 'file.txt', 'r', 4000);
  begin
    loop
      utl_file.get_line(l_file, l_line_buffer);
      l_file_buffer.extend(1);
      l_file_buffer(l_i).text := l_line_buffer;
      l_file_buffer(l_i).line := l_i;
      l_i := l_i + 1;
    end loop;
  exception
    when no_data_found then
      null;
  end;
  utl_file.fclose(l_file);
  
  forall i in 1 .. l_file_buffer.count
    insert /*+ APPEND_VALUES */ into spool_tmp(text, line)
    values(l_file_buffer(i).text, l_file_buffer(i).line);
  commit;
exception
  when others then
    if utl_file.is_open(l_file) then
      utl_file.fclose(l_file);
    end if;
    raise;
end;
/
select text from spool_tmp order by line asc
/
truncate table spool_tmp
/
drop table spool_tmp
/

spool off
exit

```

Short SET variable summary:
```
set linesize 500 - Total number of characters to display in one line
set trimspool on - Remove trailing blanks at the end of each spooled line
set feedback off - Turn off statement confirmation messages
set heading off  - Suppress column headings
set termout off  - Do not display output from commands
set echo off     - Do not display commands on screen
```

More on Oracle SET variables: https://docs.oracle.com/cd/E11882_01/server.112/e16604/ch_twelve040.htm