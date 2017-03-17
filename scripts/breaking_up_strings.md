### Breaking Up a Delimited String

Breaking a simple string with space as delimiter:
```sql
select
  regexp_substr(str, '[^ ]+', 1, level)
from
  dual
connect by
  regexp_substr(str, '[^ ]+', 1, level) is not null
```

Or we could use **regexp_count**:
```sql
select
  regexp_substr(str, '[^ ]+', 1, rownum)
from
  dual
connect by
  level <= regexp_count(str, ' ') + 1
```

If you are selecting more than one row, in Oracle Database 12c, you can take advantage of the new LATERAL join syntax.
With the **LATERAL** clause, you create an inline view that can reference values from the table you are joining to:
```sql
select
  val
from
  test,
  lateral (select
             regexp_substr(str, '[^,]+', 1, rownum) val
           from
             dual
           connect by
             level <= regexp_count(str, ',') + 1)
```