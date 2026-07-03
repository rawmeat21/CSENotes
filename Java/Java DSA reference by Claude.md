# Java DSA Cheatsheet for C++ Competitive Programmers

Everything is `java.util.*` unless noted. Import with:

```java
import java.util.*;
import java.util.stream.*;
```

---

## 1. Containers

### General mental model

| C++                                  | Java                               | Notes                                                       |
| ------------------------------------ | ---------------------------------- | ----------------------------------------------------------- |
| `vector<T>`                          | `ArrayList<T>`                     | resizable array                                             |
| `array<T,N>` / `T[]`                 | `T[]`                              | fixed size                                                  |
| `set<T>`                             | `TreeSet<T>`                       | ordered, unique                                             |
| `unordered_set<T>`                   | `HashSet<T>`                       | unordered, unique                                           |
| `multiset<T>`                        | `TreeMap<T,Integer>` (counts)      | Java has **no multiset**, see below                         |
| `map<K,V>`                           | `TreeMap<K,V>`                     | ordered                                                     |
| `unordered_map<K,V>`                 | `HashMap<K,V>`                     | unordered                                                   |
| `pair<A,B>`                          | no builtin, use array/class/record | see below                                                   |
| `queue<T>`                           | `ArrayDeque<T>` (as Queue)         |                                                             |
| `stack<T>`                           | `ArrayDeque<T>` (as Deque)         | **don't use `java.util.Stack`**, it's legacy & synchronized |
| `priority_queue<T>`                  | `PriorityQueue<T>`                 | min-heap by default (opposite of C++!)                      |
| `deque<T>`                           | `ArrayDeque<T>`                    |                                                             |
| ordered set with order stats (pb_ds) | no direct equivalent               | `TreeSet`/`TreeMap` gets you most of the way                |

Java generics only work with **objects**, not primitives — so `ArrayList<Integer>` boxes every int. This causes real overhead in CP; that's why raw arrays are still king for hot loops.

---

### ArrayList (vector)

```java
ArrayList<Integer> v = new ArrayList<>();
v.add(5);              // push_back
v.add(0, 10);           // insert at index 0 -> O(n)
v.get(0);                // v[0]
v.set(0, 99);            // v[0] = 99
v.remove(0);             // erase by index -> O(n)
v.remove(Integer.valueOf(5)); // erase by VALUE (careful: remove(int) is by index!)
v.size();
v.isEmpty();
v.clear();
v.contains(5);            // O(n) linear search
v.indexOf(5);              // -1 if not found
Collections.sort(v);        // std::sort
Collections.reverse(v);      // std::reverse
Collections.max(v); Collections.min(v);
Collections.fill(v, 0);
List<Integer> sub = v.subList(1, 4); // [1,4) VIEW not copy, like a "slice reference"
```

Construct pre-filled:

```java
ArrayList<Integer> v = new ArrayList<>(Collections.nCopies(n, 0)); // vector<int> v(n, 0)
```

Iterate:

```java
for (int x : v) { }
for (int i = 0; i < v.size(); i++) { }
Iterator<Integer> it = v.iterator();
```

---

### C-style arrays

```java
int[] a = new int[n];                 // zero-initialized, like C++ new int[n]()
int[] a = {1, 2, 3};                   // initializer list
int[][] a = new int[n][m];             // 2D, zero-initialized
Arrays.fill(a, -1);                     // fill entire 1D array
Arrays.fill(a, from, to, -1);            // fill range [from,to)
int[] b = Arrays.copyOf(a, a.length);     // copy
int[] b = a.clone();                       // also a copy (shallow for 2D!)
Arrays.sort(a);                              // sorts primitive array in place, no custom comparator allowed for primitives
Arrays.toString(a);                           // "[1, 2, 3]" for printing/debug
Arrays.equals(a, b);
int len = a.length;                            // NOT a.length() -- arrays use .length (field), Strings/Lists use .length()/.size()
```

**Gotcha:** Java arrays have no `.length()` method — it's a `.length` **field**. `String` has `.length()` a method. `List` has `.size()`.

**Gotcha:** `Arrays.sort(int[])` uses dual-pivot quicksort (no custom comparator possible for primitives — comparator only works with `Integer[]`, boxed). For custom order on primitives, either box to `Integer[]` or sort an index array.

---

### HashSet / TreeSet (set, unordered_set)

```java
Set<Integer> s = new HashSet<>();        // unordered_set
Set<Integer> s = new TreeSet<>();         // set (sorted, red-black tree)

s.add(5);
s.remove(5);
s.contains(5);
s.size();
s.isEmpty();
```

`TreeSet` extra powers (this is your `set` with order):

```java
TreeSet<Integer> s = new TreeSet<>();
s.first();            // *s.begin()
s.last();               // *s.rbegin()
s.floor(x);              // greatest element <= x   (like prev(upper_bound))
s.lower(x);               // greatest element <  x
s.ceiling(x);              // smallest element >= x  == C++ lower_bound(x)
s.higher(x);                // smallest element >  x  == C++ upper_bound(x)
s.pollFirst();                // pop & return min
s.pollLast();                   // pop & return max
s.headSet(x);                    // all elements < x (view)
s.headSet(x, true);               // all elements <= x
s.tailSet(x);                      // all elements >= x
s.tailSet(x, false);                // all elements > x
s.subSet(a, b);                      // [a, b)
s.descendingSet();                    // reversed view
s.descendingIterator();                // iterate high to low
```

**C++ `lower_bound(x)` == Java `ceiling(x)`** **C++ `upper_bound(x)` == Java `higher(x)`** (easy to mix up — C++ names describe insertion position, Java names describe value relation)

Iterate a TreeSet in reverse:

```java
for (int x : s.descendingSet()) { }
// or
Iterator<Integer> it = s.descendingIterator();
while (it.hasNext()) System.out.println(it.next());
```

**No ordered multiset / order-statistics tree** (i.e. no `pb_ds::tree` with `find_by_order`/`order_of_key`). Common workaround: a `TreeMap<Integer,Integer>` as a Fenwick tree over compressed values, or a Binary Indexed Tree for rank queries.

---

### Multiset — Java has none, here's how you fake it

**Best option: `TreeMap<Integer, Integer>` as value→count.**

```java
TreeMap<Integer, Integer> ms = new TreeMap<>();

// insert (like multiset.insert)
void add(TreeMap<Integer,Integer> ms, int x) {
    ms.merge(x, 1, Integer::sum);   // ms[x]++ but creates entry if absent
}

// erase ONE occurrence (like multiset.erase(multiset.find(x)))
void removeOne(TreeMap<Integer,Integer> ms, int x) {
    if (!ms.containsKey(x)) return;
    if (ms.get(x) == 1) ms.remove(x);
    else ms.put(x, ms.get(x) - 1);
}

// erase ALL occurrences of x (like multiset.erase(x))
ms.remove(x);

// count(x)
int c = ms.getOrDefault(x, 0);

// min / max
int mn = ms.firstKey();
int mx = ms.lastKey();

// lower_bound(x) equivalent -> smallest key >= x
Integer lb = ms.ceilingKey(x);
// upper_bound(x) equivalent -> smallest key > x
Integer ub = ms.higherKey(x);

// total element count (with duplicates)
int total = ms.values().stream().mapToInt(Integer::intValue).sum();
```

Alternative if you never need order: `HashMap<Integer,Integer>` the same way (this is the `unordered_multiset` equivalent).

Alternative if you just need "sorted collection allowing duplicates, iterate in order": use a plain `ArrayList` and keep it sorted (or sort at the end), or use `PriorityQueue`.

---

### HashMap / TreeMap (map, unordered_map)

```java
Map<Integer,Integer> m = new HashMap<>();   // unordered_map
Map<Integer,Integer> m = new TreeMap<>();    // map (sorted)

m.put(1, 100);                // m[1] = 100
m.get(1);                       // returns null if absent! (careful, NPE risk if auto-unboxed)
m.getOrDefault(1, 0);             // safe access with default
m.containsKey(1);
m.containsValue(100);
m.remove(1);
m.size();
m.isEmpty();

// increment-style patterns (no operator[] in Java)
m.put(k, m.getOrDefault(k, 0) + 1);
m.merge(k, 1, Integer::sum);         // same thing, cleaner
m.putIfAbsent(k, 0);
m.computeIfAbsent(k, key -> new ArrayList<>()).add(v);  // map<K, vector<V>> pattern!

// iterate
for (Map.Entry<Integer,Integer> e : m.entrySet()) {
    e.getKey(); e.getValue();
}
for (int key : m.keySet()) { }
for (int val : m.values()) { }
```

TreeMap extra (mirrors TreeSet):

```java
TreeMap<Integer,Integer> m = new TreeMap<>();
m.firstKey(); m.lastKey();
m.firstEntry(); m.lastEntry();          // Map.Entry, use .getKey()/.getValue()
m.floorKey(x); m.ceilingKey(x);
m.lowerKey(x); m.higherKey(x);
m.pollFirstEntry(); m.pollLastEntry();
m.headMap(x); m.tailMap(x); m.subMap(a, b);
m.descendingMap();
NavigableSet<Integer> ks = m.descendingKeySet();
```

---

### Pair — Java has no `std::pair`

Options, roughly in order of how CP people actually use them:

**1. Simple int pair via array `int[]` (fastest, common for sorting):**

```java
int[] p = {a, b};
int[][] pairs = new int[n][2];
```

**2. Custom record (Java 16+, clean, immutable, best readability):**

```java
record Pair(int first, int second) {}
Pair p = new Pair(1, 2);
p.first(); p.second();
```

**3. Custom class if you need mutability:**

```java
class Pair {
    int first, second;
    Pair(int first, int second) { this.first = first; this.second = second; }
}
```

**4. `AbstractMap.SimpleEntry<A,B>` (built-in, no custom class needed):**

```java
import java.util.AbstractMap.SimpleEntry;
SimpleEntry<Integer,Integer> p = new SimpleEntry<>(1, 2);
p.getKey(); p.getValue();
```

For `tuple`, just extend the same idea (record with 3+ fields, or nested pairs).

---

### Queue / Deque / Stack

**Queue (FIFO)** — use `ArrayDeque` via the `Queue` interface:

```java
Queue<Integer> q = new ArrayDeque<>();
q.add(5);      // push
q.offer(5);     // push, safe version (returns false instead of throwing if fails)
q.poll();        // pop & return front, null if empty (safe)
q.remove();       // pop & return front, throws if empty
q.peek();          // front(), null if empty
q.element();        // front(), throws if empty
q.isEmpty();
```

**Stack (LIFO)** — also `ArrayDeque`, use as `Deque`:

```java
Deque<Integer> st = new ArrayDeque<>();
st.push(5);        // like C++ push
st.pop();            // like C++ pop, returns the value too
st.peek();            // top()
st.isEmpty();
```

(`java.util.Stack` exists but is a legacy `Vector` subclass, synchronized/slow — avoid it.)

**Deque (both ends)**:

```java
Deque<Integer> dq = new ArrayDeque<>();
dq.addFirst(1); dq.addLast(2);
dq.removeFirst(); dq.removeLast();
dq.peekFirst(); dq.peekLast();
dq.offerFirst(1); dq.offerLast(2);
```

---

### PriorityQueue (priority_queue)

**Default is MIN-heap** — opposite of C++'s default max-heap!

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();                  // min-heap
PriorityQueue<Integer> pq = new PriorityQueue<>(Collections.reverseOrder()); // max-heap, like default C++ pq

pq.add(5);       // push
pq.offer(5);      // push, safe
pq.poll();         // pop & return top, null if empty
pq.peek();          // top(), null if empty
pq.isEmpty();
pq.size();
```

Custom comparator (e.g. min-heap of pairs by second element):

```java
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
// or explicitly to avoid overflow issues:
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
```

**No `decrease-key`** like some C++ pb_ds structures — standard workaround is lazy deletion (push new value, skip stale entries when popped, using a "seen"/"valid" check).

**No `std::multiset`-style top-k removal either** — but PQ itself already handles duplicates fine since it's not a set.

---

## 2. Multidimensional arrays / vectors

### Fixed-size C-style

```java
int[][] a = new int[n][m];                 // all zero
int[][][] a = new int[n][m][k];              // 3D, all zero
int[][] a = {{1,2},{3,4}};                     // literal
```

### Vector of vectors (ArrayList of ArrayList)

```java
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < n; i++) adj.add(new ArrayList<>());   // vector<vector<int>> adj(n)
adj.get(0).add(5);   // adj[0].push_back(5)
```

### Pre-sized & pre-filled 2D vectors

```java
// DANGER: this shares the SAME inner list across all rows!
List<List<Integer>> bad = new ArrayList<>(Collections.nCopies(n, new ArrayList<>(Collections.nCopies(m, 0))));

// CORRECT way, must create each row separately:
List<List<Integer>> grid = new ArrayList<>();
for (int i = 0; i < n; i++)
    grid.add(new ArrayList<>(Collections.nCopies(m, 0)));
```


Same trap with raw 2D arrays does NOT happen — `new int[n][m]` correctly allocates independent rows.

But it DOES happen with arrays of arrays if you try to alias rows manually:

```java
int[] row = new int[m];
int[][] a = new int[n][];
for (int i = 0; i < n; i++) a[i] = row;   // BUG: all rows point to same array!
```

### Jagged arrays

```java
int[][] a = new int[n][];
for (int i = 0; i < n; i++) a[i] = new int[someLengthFor(i)];
```

### Copying multi-dim (careful — shallow vs deep)

```java
int[][] b = a.clone();               // shallow: rows still shared with a!
int[][] b = new int[a.length][];
for (int i = 0; i < a.length; i++) b[i] = a[i].clone();  // proper deep copy
// or:
int[][] b = Arrays.stream(a).map(int[]::clone).toArray(int[][]::new);
```

---

## 3. Strings

Java `String` is **immutable** — every concatenation makes a new object. This is the single biggest habit shift from C++.

```java
String s = "hello";
s.length();                      // NOT s.length -- method not field, opposite of arrays!
s.charAt(0);                       // s[0] -- NO operator[] on String in Java
s.substring(1, 4);                   // [1,4), like s.substr(1, 3) but Java takes END index not length
s.substring(2);                        // from index 2 to end
s + " world";                            // concatenation (creates new String)
s.equals(t);                               // NEVER use == to compare String content!
s.equalsIgnoreCase(t);
s.compareTo(t);                              // like strcmp, lexicographic
s.indexOf('l');                                // first occurrence, -1 if absent
s.indexOf('l', 3);                               // search starting at index 3
s.lastIndexOf('l');
s.contains("ell");
s.startsWith("he"); s.endsWith("lo");
s.toUpperCase(); s.toLowerCase();
s.trim();  s.strip();                             // strip is unicode-aware, prefer strip() in modern code
s.replace('l', 'L');                                // char replace
s.replace("ll", "LL");                                // substring replace
s.split(","); s.split("\\s+");                          // regex-based split! returns String[]
s.isEmpty(); s.isBlank();
s.toCharArray();                                          // convert to char[] for mutation
String.valueOf(123);                                        // to_string equivalent
String.valueOf(charArray);
Integer.parseInt(s); Long.parseLong(s); Double.parseDouble(s);
s.repeat(3);                                                  // "abc".repeat(3) == "abcabcabc" (Java 11+)
s.chars();                                                      // IntStream of char codes, for stream processing
String.join(",", list);                                          // join a list of strings
```

### Building strings efficiently — use StringBuilder
```java
StringBuilder sb = new StringBuilder();
sb.append("abc");
sb.append(123);                 // overloaded for many types
sb.insert(0, "x");
sb.deleteCharAt(2);
sb.delete(1, 3);                  // remove range [1,3)
sb.reverse();                       // in-place reverse!
sb.charAt(0);
sb.setCharAt(0, 'z');                 // mutate a char, String can't do this
sb.length();
sb.toString();                          // convert back to String
sb.setLength(0);                          // clear it (reuse buffer)
```

### char[] mutation (since String can't be mutated in place)

```java
char[] arr = s.toCharArray();
arr[0] = 'X';
String result = new String(arr);
String result = String.valueOf(arr);
```

### StringBuilder vs String vs char[] — when to use which

- Building a string piece by piece in a loop → `StringBuilder` (String concat in a loop is O(n²)).
- Need `.substring`, `.split`, regex, `.equals` semantics → `String`.
- Need in-place random-access char editing without the overhead of an object per append → `char[]`.

---

## 4. Lambdas

Java lambdas implement a **functional interface** (an interface with exactly one abstract method). Common built-in ones: `Runnable`, `Comparator<T>`, `Function<A,R>`, `BiFunction<A,B,R>`, `Predicate<T>`, `Consumer<T>`, `Supplier<T>`.

```java
Comparator<Integer> cmp = (a, b) -> a - b;
Runnable r = () -> System.out.println("hi");
Function<Integer,Integer> sq = x -> x * x;
BiFunction<Integer,Integer,Integer> add = (a, b) -> a + b;
```

### Capturing outside variables

Java lambdas can capture local variables, but **only if effectively final** (never reassigned after initialization — no mutable capture-by-reference like C++'s `[&]`).

```java
int base = 10;
Function<Integer,Integer> addBase = x -> x + base;   // OK, base never reassigned
```

To "mutate" a captured variable, wrap it in a 1-element array or an `AtomicInteger` (common CP trick):

```java
int[] counter = {0};
Runnable inc = () -> counter[0]++;    // works because the ARRAY reference is final, contents can change
```

For capturing `this` / instance fields, it works normally (that's captured by reference implicitly, no effectively-final restriction on fields).

### Recursive lambdas

Java lambdas **cannot refer to themselves by name** (no `[&](int n) -> ... { ... self(n-1) ...}` shortcut). Standard workarounds:

**1. Array/wrapper trick (most common in CP):**

```java
Function<Integer,Integer>[] fib = new Function[1];
fib[0] = n -> n <= 1 ? n : fib[0].apply(n - 1) + fib[0].apply(n - 2);
System.out.println(fib[0].apply(10));
```

**2. Just write a real method (usually cleanest and fastest — preferred for DFS/backtracking):**

```java
static int fib(int n) {
    return n <= 1 ? n : fib(n - 1) + fib(n - 2);
}
```

**3. Custom self-referencing functional interface:**

```java
interface RecFunc { int apply(int n); }
RecFunc fib = n -> n <= 1 ? n : /* still need access to itself */ 0;
// this doesn't fully solve it either -- method #1 or #2 above are the real answers
```

**In practice for CP: use a plain recursive method for DFS/backtracking. Only reach for lambdas for short comparators / stream operations.**

---

## 5. Reversing / reverse iteration

```java
Collections.reverse(list);              // reverse ArrayList in place
StringBuilder sb = new StringBuilder(s);
sb.reverse();                             // reverse a String (via StringBuilder), then .toString()

// reverse a primitive array -- no built-in, write it or use a loop
for (int i = 0, j = a.length - 1; i < j; i++, j--) {
    int tmp = a[i]; a[i] = a[j]; a[j] = tmp;
}

// or via boxed array + Collections
Integer[] boxed = ...;
Collections.reverse(Arrays.asList(boxed));  // works in place since asList is a view

// reverse iterate a List
for (int i = list.size() - 1; i >= 0; i--) { }
ListIterator<Integer> it = list.listIterator(list.size());
while (it.hasPrevious()) { int x = it.previous(); }

// reverse iterate TreeSet/TreeMap (like rbegin()/rend())
for (int x : treeSet.descendingSet()) { }
for (int k : treeMap.descendingKeySet()) { }
```

There's no `rbegin()/rend()` concept baked into iterators the way C++ has `reverse_iterator` — you either get a reversed **view** (`descendingSet`, `descendingMap`) or iterate by index/ListIterator backwards.

---

## 6. Input

Fast, CP-standard way — `BufferedReader` + `StringTokenizer` (Scanner is much slower, avoid for big inputs):

```java
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        int n = Integer.parseInt(st.nextToken());
        int m = Integer.parseInt(st.nextToken());

        int[] a = new int[n];
        st = new StringTokenizer(br.readLine());
        for (int i = 0; i < n; i++) a[i] = Integer.parseInt(st.nextToken());

        String line = br.readLine();       // read a full line (e.g. a string input)
    }
}
```

Simpler but slower (`Scanner`) — fine for small inputs:

```java
Scanner sc = new Scanner(System.in);
int n = sc.nextInt();
long L = sc.nextLong();
double d = sc.nextDouble();
String word = sc.next();          // one whitespace-delimited token
String line = sc.nextLine();       // full line
```

Fast output — batch with `StringBuilder` + one flush, or use `PrintWriter`:

```java
StringBuilder sb = new StringBuilder();
sb.append(answer).append('\n');
System.out.print(sb);              // print once at the end, not per-line System.out.println in a hot loop

PrintWriter pw = new PrintWriter(new BufferedWriter(new OutputStreamWriter(System.out)));
pw.println(answer);
pw.flush();
```

---

## 7. Sorting

### 1D primitive array

```java
int[] a = ...;
Arrays.sort(a);                       // ascending only, no comparator allowed on primitives
Arrays.sort(a, from, to);               // sort subrange [from, to)
```

### 1D boxed array or List — custom comparator allowed

```java
Integer[] a = ...;
Arrays.sort(a, (x, y) -> y - x);           // descending
Arrays.sort(a, Collections.reverseOrder());  // descending, built-in

List<Integer> list = ...;
Collections.sort(list);
list.sort((x, y) -> y - x);
list.sort(Comparator.reverseOrder());
```

### Descending sort on int[] primitive (no comparator possible) — workarounds

```java
// 1. sort then reverse
Arrays.sort(a);
// reverse manually (see section 5)

// 2. box it
Integer[] boxed = Arrays.stream(a).boxed().toArray(Integer[]::new);
Arrays.sort(boxed, Collections.reverseOrder());
```

### 2D array (like vector<pair<int,int>>, sort by column)

```java
int[][] a = ...;
Arrays.sort(a, (x, y) -> x[0] - y[0]);                       // sort by column 0
Arrays.sort(a, (x, y) -> x[0] != y[0] ? x[0] - y[0] : x[1] - y[1]);  // tie-break by column 1

// using Comparator chaining (clean multi-key sort, like C++ tie() in a custom comparator)
Arrays.sort(a, Comparator.comparingInt((int[] p) -> p[0])
                          .thenComparingInt(p -> p[1]));
Arrays.sort(a, Comparator.comparingInt((int[] p) -> p[0]).reversed());
```

### List of objects / records

```java
list.sort(Comparator.comparingInt(Person::getAge));
list.sort(Comparator.comparingInt(Person::getAge).thenComparing(Person::getName));
list.sort(Comparator.comparing(Person::getName).reversed());
```

### Custom objects implementing Comparable (like operator< overload in C++)

```java
class Point implements Comparable<Point> {
    int x, y;
    public int compareTo(Point o) { return this.x - o.x; }  // like operator<
}
Arrays.sort(pointArray);   // uses compareTo automatically
```

### N-dimensional / nested — same idea, just index deeper in the comparator

```java
// e.g., sort array of int[3] by 2nd then 3rd field
Arrays.sort(a, Comparator.<int[]>comparingInt(p -> p[1]).thenComparingInt(p -> p[2]));
```

**Gotcha:** `int[]` arrays sort **ascending only**, no custom order — this trips up a lot of C++ people expecting `sort(a, a+n, cmp)` to just work on any array type. **Gotcha:** avoid `(a,b) -> a - b` on `Integer` for large values, risk of overflow — prefer `Integer.compare(a, b)`.

---

## 8. chars & other useful bits

```java
char c = 'a';
Character.isDigit(c); Character.isLetter(c); Character.isLetterOrDigit(c);
Character.isUpperCase(c); Character.isLowerCase(c);
Character.isWhitespace(c);
Character.toUpperCase(c); Character.toLowerCase(c);
Character.getNumericValue(c);        // '7' -> 7
(int) c;                                // char to int (ASCII code), like C++
(char) (c + 1);                           // int to char, must cast back
c - '0';                                    // digit char to int, same trick as C++
(char) ('a' + 5);                             // 'f'
```

### Math / numeric utilities (things people forget don't auto-exist)

```java
Math.max(a, b); Math.min(a, b);
Math.abs(x);
Math.pow(a, b);                  // returns double, careful with int use
Math.sqrt(x);
Math.floor(x); Math.ceil(x);
Math.floorDiv(a, b); Math.floorMod(a, b);   // correct behavior for negative numbers, unlike a % b
Integer.MAX_VALUE; Integer.MIN_VALUE;
Long.MAX_VALUE; Long.MIN_VALUE;
Integer.toBinaryString(x); Integer.toHexString(x); Integer.toOctalString(x);
Integer.bitCount(x);                          // __builtin_popcount
Integer.numberOfLeadingZeros(x);
Integer.numberOfTrailingZeros(x);               // __builtin_ctz
Integer.highestOneBit(x);
Long.bitCount(x);                                 // for long/long long popcount
```

### Boxed vs primitive gotchas (this bites everyone coming from C++)

- `==` on `Integer` compares references, not values, outside the range **-128 to 127** (cached range) — always use `.equals()` or unbox first for boxed comparisons.
- Auto-unboxing a `null` (e.g. `map.get(missingKey)` then arithmetic on it) throws `NullPointerException`.
- Generic containers (`ArrayList<Integer>`, `HashMap<Integer,Integer>`, etc.) always box — for tight loops on huge N, prefer raw primitive arrays for performance.

### Arrays utility grab-bag

```java
Arrays.asList(1, 2, 3);                    // fixed-size List view (no add/remove!)
new ArrayList<>(Arrays.asList(1,2,3));      // convert to real resizable list
Arrays.binarySearch(a, key);                  // requires sorted array, returns -(insertion point)-1 if absent
Arrays.equals(a, b);
Arrays.deepToString(matrix);                    // pretty-print 2D array
Arrays.stream(a).sum();                           // sum of int[]
Arrays.stream(a).max().getAsInt();
```

### Bit manipulation (no `<bitset>`, but BitSet exists)

```java
BitSet bs = new BitSet(n);
bs.set(i); bs.clear(i); bs.get(i); bs.flip(i);
bs.cardinality();     // popcount
```

For most CP bitmask work, a plain `int`/`long` with manual `&`, `|`, `^`, `<<`, `>>` is simpler than `BitSet` — same as C++.

---

## Quick gotcha summary (things that WILL bite you coming from C++)

1. `array.length` is a field, `list.size()` and `string.length()` are methods.
2. `PriorityQueue` defaults to **min-heap** (C++ `priority_queue` defaults to max-heap).
3. No `pair`, no `tuple`, no `multiset` built in — use records/arrays and TreeMap-as-counter respectively.
4. Lambdas can't self-recurse directly — use a real method, or the array-wrapper trick.
5. Lambda captures must be effectively final — no direct mutable reference capture.
6. `Arrays.sort` on primitive arrays takes no comparator; box to `Integer[]` if you need custom order.
7. `Integer` uses reference equality with `==` outside [-128,127] — use `.equals()`.
8. `String` is immutable — use `StringBuilder` for loops of concatenation.
9. `Collections.nCopies` for a 2D list literally shares one inner list unless you loop and create rows individually.
10. `map.get(key)` returns `null` (not a default value) if key is absent — use `getOrDefault`.