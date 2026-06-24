
**Monday Visitors**

|name|
|---|
|Alice|
|Bob|
|Alice|

**Tuesday Visitors**

| name    |
| ------- |
| Bob     |
| Charlie |

### `UNION ALL`

`UNION ALL` is the simplest, fastest set operation. It just appends the results.


```sql
SELECT name FROM monday_visitors
UNION ALL
```

**Output:**

|name|
|---|
|Alice|
|Bob|
|Alice|
|Bob|
|Charlie|

### `UNION`

If you drop the word `ALL` and just write `UNION`, the database gets significantly stricter.


```sql
SELECT name FROM monday_visitors
UNION
SELECT name FROM tuesday_visitors;
```

More like a set union. There will be no duplicates.

| name    |
| ------- |
| Alice   |
| Bob     |
| Charlie |
### `INTERSECT`

```sql
SELECT name FROM monday_visitors
INTERSECT
SELECT name FROM tuesday_visitors;
```

|name|
|---|
|Bob|

### `EXCEPT` / `MINUS` (A - B)


```sql
SELECT name FROM monday_visitors
EXCEPT
SELECT name FROM tuesday_visitors;
```

|name|
|---|
|Alice|
