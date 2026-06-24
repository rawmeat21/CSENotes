| Syntax Order          | Logical Order                                             |
| --------------------- | --------------------------------------------------------- |
| 1. `SELECT`           | **1. `FROM` / `JOIN`** (Get raw data)                     |
| 2. `FROM`             | **2. `WHERE`** (Filter individual rows)                   |
| 3. `JOIN`             | **3. `GROUP BY`** (Organize into buckets)                 |
| 4. `WHERE`            | **4. `HAVING`** (Filter the buckets)                      |
| 5. `GROUP BY`         | **5. `SELECT`** (Evaluate row expressions & give aliases) |
| 6. `HAVING`           | **6. `DISTINCT`** (Deduplicate the final rows)            |
| 7. `ORDER BY`         | **7. `ORDER BY`** (Sort the output)                       |
| 8. `LIMIT` / `OFFSET` | **8. `LIMIT` / `OFFSET`** (Slice the final count)         |

```sql
SELECT amount * 1.10 AS taxed_amount
FROM sales
WHERE taxed_amount > 100;
```

This will give error, because when we get to WHERE, it hasn't seen tax_amount!