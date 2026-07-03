## 1. Containers, Bounds, and Iterators


### Sets & Maps (Ordered & Unordered)

- **`std::set` (Ordered)** → `TreeSet<T>` (Red-Black Tree, O(logn))
    
- **`std::unordered_set`** → `HashSet<T>` (Hash Table, O(1))
    
- **`std::map` (Ordered)** → `TreeMap<K, V>` (Red-Black Tree, O(logn))
    
- **`std::unordered_map`** → `HashMap<K, V>` (Hash Table, O(1))
    

### Multiset 

Java **does not** have a built-in `Multiset`. To replicate `std::multiset`, you must use a `TreeMap<T, Integer>` where the value tracks the frequency count.

```java
TreeMap<Integer, Integer> multiset = new TreeMap<>();

// Insert (multiset.insert(x))
multiset.put(x, multiset.getOrDefault(x, 0) + 1);

// Count occurrences (multiset.count(x))
int count = multiset.getOrDefault(x, 0);

// Erase ONE instance (multiset.erase(multiset.find(x)))
if (multiset.containsKey(x)) {
    if (multiset.get(x) == 1) multiset.remove(x);
    else multiset.put(x, multiset.get(x) - 1);
}

// Erase ALL instances (multiset.erase(x))
multiset.remove(x);
```

### Lower and Upper Bounds

Java uses specific method names on its sorted structures (`TreeSet` and `TreeMap`) instead of `std::lower_bound` and `std::upper_bound`.

|C++ Concept|Condition|Java `TreeSet`|Java `TreeMap` (Keys)|
|---|---|---|---|
|`lower_bound(x)`|≥x|`set.ceiling(x)`|`map.ceilingKey(x)`|
|`upper_bound(x)`|>x|`set.higher(x)`|`map.higherKey(x)`|
|`--lower_bound(x)`|≤x|`set.floor(x)`|`map.floorKey(x)`|
|—|<x|`set.lower(x)`|`map.lowerKey(x)`|

### Iterators & `rbegin()`

Java uses `Iterator` or enhanced for-loops. For reverse iteration (`rbegin()`), sorted collections provide descending views.

Java

```java
TreeSet<Integer> set = new TreeSet<>();

// Forward Iterator (begin to end)
for (int val : set) { ... }

// Reverse Iterator (rbegin to rend)
Iterator<Integer> revIt = set.descendingIterator();
while (revIt.hasNext()) {
    int val = revIt.next();
}
```

### Pairs, Stacks, Queues, & Priority Queues

- **`std::pair`**: Java has no generic `Pair`. Use a Java **`record`** (lightweight immutable class, fast to write).
    
- **`std::stack`**: Do **not** use `java.util.Stack` (it is obsolete and synchronized/slow). Use `ArrayDeque` instead.
    
- **`std::queue`**: Use `ArrayDeque` via the `Queue` interface.
    
- **`std::priority_queue`**: Use `PriorityQueue`. It is a **Min-Heap by default** (opposite of C++).
    

Java

```
// 1. Pair Equivalent (Declare outside or inside class)
record Pair(int first, int second) {}
Pair p = new Pair(1, 2);

// 2. Stack (Using ArrayDeque)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1); 
int top = stack.peek(); 
stack.pop();

// 3. Queue
Queue<Integer> q = new ArrayDeque<>();
q.offer(1); // equivalent to push()
int front = q.peek();
q.poll();   // equivalent to pop()

// 4. Priority Queue (Min-Heap by default)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
// Max-Heap Equivalent
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
```

## 2. Arrays, Vectors, and Initialization

Java distinguishes between primitive arrays (fixed size, low overhead) and object collections (dynamic, boxed overhead).

Java

```
// 1. C-style Array (Fixed Size)
int[] arr = new int[n]; 
int[][] matrix = new int[rows][cols];

// 2. Vector Equivalent (Dynamic ArrayList)
// Note: You must use the Wrapper class 'Integer', not primitive 'int'
ArrayList<Integer> vec = new ArrayList<>();

// 3. Vector with Predefined Size and Prefilled Values
// Equivalent to: vector<int> v(n, val);
ArrayList<Integer> filledVec = new ArrayList<>(Collections.nCopies(n, val));
```

## 3. String Manipulation

In Java, `String` is **immutable**. Modifying a `String` in a loop creates O(N) copies, resulting in an O(N2) time complexity disaster. Always use `StringBuilder` for modifications.

Java

```
String s = "hello";

// Access character (s[i])
char c = s.charAt(i); 

// Substring (s.substr(start, length) in C++)
// Java uses [start, end) indices:
String sub = s.substring(1, 4); // "ell"

// String Length
int len = s.length();

// Mutable String (Essential for DSA)
StringBuilder sb = new StringBuilder("hello");
sb.append(" world");
sb.setCharAt(0, 'H');
sb.reverse();
String result = sb.toString();
```

## 4. Lambdas and Recursion Quirks

### Lambda Capturing

C++ allows capturing by reference `[&]`. Java lambdas can **only capture local variables that are `effectively final`** (meaning they are never reassigned after initialization).

To mutate an external variable inside a lambda, wrap it in a single-element array:

Java

```
int[] count = new int[1]; // Workaround to bypass effectively final rule

Runnable r = () -> {
    count[0]++; // Legal! We are mutating the array contents, not the array reference.
};
```

### Recursive Lambdas

Java cannot easily reference a local lambda inside itself because it hasn't finished initializing during definition.

- **The C++ Way:** Define a lambda that captures itself.
    
- **The Java DSA Workaround:** Just use a traditional `private` or `static` helper method in your class. It's cleaner, faster, and avoids variable scope issues.
    

If you _must_ do it inline, use an array wrapper trick:

Java

```
// Define a functional interface wrapper
interface IntConsumer { void accept(int x); }

IntConsumer[] dfs = new IntConsumer[1];
dfs[0] = (u) -> {
    if (u == 0) return;
    System.out.println(u);
    dfs[0].accept(u - 1); // Self-reference works via array pointer
};
dfs[0].accept(5);
```

## 5. Reversing and Iterating Backwards

Java

```
// Reversing an ArrayList
Collections.reverse(arrayList);

// Reversing a Primitive Array (No built-in primitive reverse, do it manually)
for (int i = 0; i < arr.length / 2; i++) {
    int temp = arr[i];
    arr[i] = arr[arr.length - 1 - i];
    arr[arr.length - 1 - i] = temp;
}

// Simple reverse loop
for (int i = arr.length - 1; i >= 0; i--) { ... }
```

## 6. Fast Input / Output

`Scanner` is slow and will TLE on large competitive programming data sets (N>105). Use `BufferedReader` and `StringTokenizer`.

Java

```
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        
        int n = Integer.parseInt(st.nextToken());
        int k = Integer.parseInt(st.nextToken());
        
        // For outputting huge chunks of text
        PrintWriter out = new PrintWriter(System.out);
        out.println(n + k);
        out.flush(); // Don't forget to flush at the end
    }
}
```

## 7. Custom Sorting

### Sorting Primitives vs Objects

- `Arrays.sort(int[] arr)` uses Dual-Pivot Quicksort (O(nlogn) average, worst-case can hit O(n2) if anti-quicksort test cases are constructed, though rare).
    
- `Arrays.sort(T[] arr)` or `Collections.sort(List<T>)` uses Stable Timsort (O(nlogn) guaranteed).
    

> ⚠️ **Gotcha:** You **cannot** pass a custom comparator to a primitive array (`int[]`). If you want to sort primitives in descending or custom order, you must either sort a 2D array, box it to an `Integer[]` array, or sort ascending and reverse it manually.

Java

```
// 1. Sort 1D Array Ascending
Arrays.sort(arr);

// 2. Sort 2D Array by first column, then second column (Custom Comparator)
int[][] matrix = {{1, 4}, {1, 2}, {3, 5}};
Arrays.sort(matrix, (a, b) -> {
    if (a[0] != b[0]) return Integer.compare(a[0], b[0]);
    return Integer.compare(a[1], b[1]);
});

// 3. Sort Object/Wrapper Lists Descending
ArrayList<Integer> list = new ArrayList<>();
list.sort((a, b) -> Integer.compare(b, a)); // Or Collections.sort(list, Collections.reverseOrder());
```

## 8. Working with Chars

Java characters (`char`) are 16-bit Unicode, but they implicitly convert to integer ASCII values exactly like C++.

Java

```
char ch = 'g';

// ASCII Math
int alphabetIndex = ch - 'a'; // 'g' - 'a' = 6
char nextChar = (char)(ch + 1); // 'h'

// Character Utility Functions
boolean isLoopLetter = Character.isLetter(ch);
boolean isDigit = Character.isDigit('5');
char upper = Character.toUpperCase(ch);
```

## 9. Things You Missed (Bonus C++ Gotchas)

### 1. The Autoboxing Trap (TLE Warning)

Using `ArrayList<Integer>` or `HashMap<Integer, Integer>` forces Java to wrap primitive `int` values into `Integer` objects. Doing this millions of times in loops causes heavy garbage collection overhead. If you're building a graph, prefer `ArrayList<Integer>[] adj = new ArrayList[N]` over `ArrayList<ArrayList<Integer>>`.

### 2. `std::bitset` Equivalent

Java has a highly optimized `java.util.BitSet` class.

Java

```
BitSet bs = new BitSet(1000);
bs.set(10);     // Set bit 10 to true
bs.clear(10);   // Set bit 10 to false
bs.flip(5);     // Toggle bit 5
boolean val = bs.get(10);
```

### 3. Object Equality (`==` vs `.equals()`)

In C++, `==` checks value equality if overloaded. In Java, `==` on objects (like `String` or `Integer` wrappers) checks **reference equality** (if they point to the same memory). Always use `.equals()` to compare values of Objects/Collections.

Java

```
Integer x = 1000, y = 1000;
if (x == y) { }       // FALSE (Different object instances)
if (x.equals(y)) { }  // TRUE (Values are equal)
```