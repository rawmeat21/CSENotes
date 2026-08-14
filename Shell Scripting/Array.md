## Quick Reference Cheat Sheet

|Operation|Syntax|
|---|---|
|**Declare Empty Indexed Array**|`declare -a my_arr=()`|
|**Declare Empty Associative Array**|`declare -A my_map=()`|
|**Get Element at Index `i`**|`"${my_arr[i]}"`|
|**Get Value by Key `k`**|`"${my_map[k]}"`|
|**Get All Elements**|`"${my_arr[@]}"`|
|**Get Total Count / Length**|`"${#my_arr[@]}"`|
|**Get All Indices / Keys**|`"${!my_arr[@]}"`|
|**Append Element**|`my_arr+=("new_item")`|
|**Delete Element**|`unset my_arr[i]`|


Bash supports two types of arrays:

1. **Indexed Arrays** (List ordered by numeric index: `0, 1, 2...`)
    
2. **Associative Arrays** (Key-Value pairs / Hash maps / Dictionaries: `"key" => "value"`)
    

### Part A: Indexed Arrays

#### 1. Creating / Initializing

Bash

```bash
# Method 1: Declare and assign directly
fruits=("apple" "banana" "cherry")

# Method 2: Create an empty array and append items
declare -a log_files=()
log_files+=("/var/log/syslog")
log_files+=("/var/log/auth.log")

# Method 3: Assigning specific indices
colors[0]="red"
colors[1]="blue"
colors[5]="green"  # Bash arrays can be sparse (gaps are allowed)
```

#### 2. Accessing Elements

> **Golden Rule:** Always quote array expansions with `"${arr[@]}"` to safely handle spaces in strings!

Bash

```bash
fruits=("apple pie" "banana" "cherry")

echo "${fruits[0]}"   # Output: apple pie (First element)
echo "${fruits[-1]}"  # Output: cherry (Last element)
echo "${fruits[@]}"   # Output: apple pie banana cherry (All elements)
```

#### 3. Getting Array Length & Indices

Bash

```bash
echo "${#fruits[@]}"  # Output: 3 (Total number of elements)
echo "${!fruits[@]}"  # Output: 0 1 2 (Prints all defined indices)
```

#### 4. Iterating Over an Array

Bash

```bash
fruits=("apple pie" "banana" "cherry")

# Loop over values (Most common)
for fruit in "${fruits[@]}"; do
    echo "I like $fruit"
done

# Loop over indices (Useful if you need the index position)
for i in "${!fruits[@]}"; do
    echo "Index $i holds: ${fruits[$i]}"
done
```

#### 5. Modifying & Removing

Bash

```bash
# Append a new element
fruits+=("dragonfruit")

# Update an element
fruits[1]="blueberry"

# Delete a specific element
unset fruits[0]

# Delete the entire array
unset fruits
```

### Part B: Associative Arrays (Key-Value Pairs)

Associative arrays **must** be explicitly declared using `declare -A` before use.

#### 1. Creating & Accessing

Bash

```bash
# Declare the associative array
declare -A user_roles

# Assign keys and values
user_roles["alice"]="admin"
user_roles["bob"]="developer"
user_roles["charlie"]="viewer"

# Access a value by key
echo "${user_roles["alice"]}"  # Output: admin
```

#### 2. Iterating Over Keys & Values

Bash

```bash
# Loop through Keys using !
for user in "${!user_roles[@]}"; do
    role="${user_roles[$user]}"
    echo "User: $user -> Role: $role"
done
```

### Part C: Slicing & Manipulating Arrays


```bash
numbers=(0 10 20 30 40 50 60)

# Syntax: ${array[@]:start:length}

echo "${numbers[@]:2:3}"   # Output: 20 30 40 (Start at index 2, take 3 elements)
echo "${numbers[@]:4}"     # Output: 40 50 60 (Start at index 4 to end)
```