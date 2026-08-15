## Bash String Operations (Mapped to C++ STL `std::string`)

In C++, you call methods like `s.substr()` or `s.length()`. In Bash, you use **Parameter Expansion** with special operators inside `${...}`.

### Reference Table: C++ STL vs. Bash

| C++ STL Method (`std::string s`) | Bash Equivalent             | Example / Explanation                            |
| -------------------------------- | --------------------------- | ------------------------------------------------ |
| **`s.length()` / `s.size()`**    | `${#s}`                     | Returns string length.                           |
| **`s[i]` / `s.at(i)`**           | `${s:i:1}`                  | Gets character at index `i` (0-based).           |
| **`s.substr(pos, count)`**       | `${s:pos:count}`            | Extracts `count` characters starting from `pos`. |
| **`s.substr(pos)`**              | `${s:pos}`                  | Extracts substring from `pos` to the end.        |
| **`s.append("x")` / `s += "x"`** | `s+="x"`                    | Appends text to string.                          |
| **`s.empty()`**                  | `[[ -z "$s" ]]`             | Returns true if string length is zero.           |
| **`!s.empty()`**                 | `[[ -n "$s" ]]`             | Returns true if string is non-empty.             |
| **`s1 == s2`**                   | `[[ "$s1" == "$s2" ]]`      | String equality check.                           |
| **`s1 < s2`**                    | `[[ "$s1" < "$s2" ]]`       | Lexicographical ordering comparison.             |
| **`std::toupper` (All)**         | `${s^^}`                    | Converts entire string to uppercase.             |
| **`std::tolower` (All)**         | `${s,,}`                    | Converts entire string to lowercase.             |
| **First char to upper**          | `${s^}`                     | Capitalizes only the first character.            |
| **First char to lower**          | `${s,}`                     | Lowercases only the first character.             |
| **`s.replace(pos, len, "x")`**   | `${s/pattern/replacement}`  | Replaces **first** pattern match.                |
| **`std::regex_replace`**         | `${s//pattern/replacement}` | Replaces **all** pattern matches.                |
| **`s.erase(0, len)`** (Prefix)   | `${s#pattern}`              | Removes shortest matching prefix pattern.        |
| **`s.erase()`** (Long Prefix)    | `${s##pattern}`             | Removes longest matching prefix pattern.         |
| **`s.pop_back()`** (Suffix)      | `${s%pattern}`              | Removes shortest matching suffix pattern.        |
| **`s.erase()`** (Long Suffix)    | `${s%%pattern}`             | Removes longest matching suffix pattern.         |
| **`s.find("sub")`**              | `[[ "$s" == *"sub"* ]]`     | Checks if substring exists in string.            |
| **`std::getline(cin, s)`**       | `read -r s`                 | Reads a full line of text into variable `s`.     |
|                                  |                             |                                                  |

## 3. Detailed Code Examples

### A. Substring & Indexing (`substr`)

Bash

```bash
str="HelloWorld"

# C++: str.substr(0, 5) -> "Hello"
echo "${str:0:5}"

# C++: str.substr(5) -> "World"
echo "${str:5}"

# Negative Index (Get last 5 characters)
# Note: Space required before minus sign!
echo "${str: -5}" # "World"
```

### B. Finding & Replacing Text (`replace`)

Bash

```bash
str="apple banana apple orange"

# Replace FIRST match: C++ s.replace() / s.find()
echo "${str/apple/mango}"
# Output: mango banana apple orange

# Replace ALL matches: C++ std::regex_replace
echo "${str//apple/mango}"
# Output: mango banana mango orange

# Replace only if it matches at the START (Prefix)
echo "${str/#apple/mango}"

# Replace only if it matches at the END (Suffix)
echo "${str/%orange/pineapple}"
```

### C. Prefix & Suffix Trimming (`erase` / Pattern Stripping)

Bash has a feature built specifically for stripping file extensions and path prefixes:

Bash

```bash
filepath="/var/log/app/data.tar.gz"

# Strip shortest prefix matching '*/' (Gets filename)
echo "${filepath#*/}"    # "var/log/app/data.tar.gz"

# Strip LONGEST prefix matching '*/' (Like C++ path.filename())
echo "${filepath##*/}"   # "data.tar.gz"

# Strip shortest suffix matching '.*' (Removes last extension)
echo "${filepath%.*}"    # "/var/log/app/data.tar"

# Strip LONGEST suffix matching '.*' (Removes all extensions)
echo "${filepath%%.*}"   # "/var/log/app/data"
```

### D. Changing Case (`toupper` / `tolower`)

Bash

```
text="hello WORLD"

# Upper all
echo "${text^^}"   # "HELLO WORLD"

# Lower all
echo "${text,,}"   # "hello world"

# Toggle case of first char
echo "${text^}"    # "Hello WORLD"
```

### E. Reading Strings Safely (`getline`)

In C++, you use `getline(cin, str)` to avoid breaking on spaces. In Bash, use `IFS= read -r`:

Bash

```
# -r disables backslash escaping
# IFS= prevents leading/trailing whitespace trimming
IFS= read -r user_input
```