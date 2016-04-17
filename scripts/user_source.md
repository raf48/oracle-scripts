###Search all packages and view sources for "&text".

```sql
select
  type,
  name,
  text,
  line
from
  user_source
where
  instr(upper(text), upper('&text')) > 1
union all
select
  'VIEW' type,
  view_name,
  null,
  null
from
  user_views
where
  instr(upper(dbms_xmlgen.getxml(
    'select text
     from all_views
     where view_name = ''' || view_name || '''')), upper('&text')) > 1
```

If you need to see if a particular object is used you can use *_DEPENDENCIES, but it only works for static SQL:
```sql
select type, name
  from user_dependencies
 where referenced_name = upper('&object_name')
```