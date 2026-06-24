### The Global Row Filter (`SELECT DISTINCT ...`)

When `DISTINCT` is placed right after `SELECT`, its scope is **global** to that row.


```sql
SELECT DISTINCT name, age FROM person;
```

Here, DISTINCT means distinct **rows**. Its not that name should be distinct

### The Local Pipeline Filter (`COUNT(DISTINCT ...)`)

When `DISTINCT` is placed _inside_ a function's parentheses, its global powers are stripped away. It becomes a **local** gatekeeper, completely trapped inside that specific function.


```sql
SELECT COUNT(DISTINCT employee), SUM(DISTINCT amount) FROM sales;
```

Here, DISTINCT employee means **distinct employees**
