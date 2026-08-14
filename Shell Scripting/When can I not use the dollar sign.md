### 1. Double Parentheses `(( ... ))` — **No `$` Required**

Used exclusively for **integer arithmetic** and numeric comparisons.

Bash

```
count=5

# You can omit '$' inside (( ... ))
if (( count > 3 )); then
    echo "Count is high!"
fi

# Works for math assignments too:
(( count++ ))
(( result = count * 2 ))
```

> **Note:** **This _only_ works for numbers**. String comparisons (like checking `'y'` or `'n'`) will fail or behave unpredictably inside `(( ... ))`. Use `[[]]` instead.

### 2. Case Statements `case ... in` — **No `$` in Patterns**

While you still need `$` to evaluate the target variable at the top, you don't use `$` inside the test patterns:

Bash

```
read c

case $c in
    [Yy]) echo "YES" ;;
    [Nn]) echo "NO"  ;;
esac
```

### Quick Cheat Sheet

|Bracket / Syntax|Purpose|Needs `$` for Variables?|Example|
|---|---|---|---|
|**`(( ... ))`**|Arithmetic & numeric tests|**No**|`(( age >= 18 ))`|
|**`[[ ... ]]`**|Advanced logic & strings|**Yes**|`[[ $name == "Alice" ]]`|
|**`[ ... ]`**|Legacy POSIX test|**Yes**|`[ "$age" -gt 18 ]`|
|**`( ... )`**|Subshell execution|**Yes**|