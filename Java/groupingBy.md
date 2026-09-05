
Example:

```java
class Employee {
    private String name;
    private String department;   // "IT", "HR", "Sales"
    private int salary;
    private boolean isManager;

    // getters: getName(), getDepartment(), getSalary(), isManager()
}

List<Employee> employees = List.of(
    new Employee("Alice", "IT", 90000, true),
    new Employee("Bob", "IT", 60000, false),
    new Employee("Carol", "HR", 55000, false),
    new Employee("Dave", "HR", 70000, true),
    new Employee("Eve", "Sales", 50000, false),
    new Employee("Frank", "Sales", 85000, true)
);
```

---

### The single most important idea: `groupingBy` makes "buckets"

Forget the Java syntax for a second. Imagine you have a pile of employee index cards, and you want to sort them into folders labeled by department. That's **all** `groupingBy` does:

```
Alice(IT) ─┐
Bob(IT)   ─┼──► folder "IT"    = [Alice, Bob]
Carol(HR) ─┼──► folder "HR"    = [Carol, Dave]
Dave(HR)  ─┘
Eve(Sales)─┐
Frank(Sales)┴─► folder "Sales" = [Eve, Frank]
```

The thing that decides _which folder a card goes into_ is called the **classification function**. In code:

java

```java
Map<String, List<Employee>> byDept =
    employees.stream()
             .collect(groupingBy(Employee::getDepartment));
```

- `Employee::getDepartment` is the classification function — "look at this card, tell me which folder label to use."
- The result is a `Map<K, List<T>>` — key = folder label, value = the list of items in that folder.
- **Nothing more happens inside each folder** by default — the items just get added to a `List`, unsorted, uncounted, untouched.

That "unsorted, untouched, just collected into a list" behavior is the _default_. That default is itself a collector called `toList()`. This is the point the slides make that trips people up:

> `groupingBy(f)` is really shorthand for `groupingBy(f, toList())`

So there are secretly **two arguments** to `groupingBy`, and when you only give one, Java quietly plugs in `toList()` as the second one for you.

java

```java
// these two lines do exactly the same thing:
groupingBy(Employee::getDepartment)
groupingBy(Employee::getDepartment, toList())
```

That second argument is called the **downstream collector** — "once you've sorted the cards into folders, what do you want to _do_ with the cards inside each folder?" `toList()` says "just pile them up as a list." But you could ask for something completely different to happen _within each folder_ — and that's the whole rest of this topic.

---

### Step 1: Classification function can be _anything_, not just a getter

```java
enum SalaryBand { LOW, MID, HIGH }

Map<SalaryBand, List<Employee>> bySalaryBand =
    employees.stream().collect(groupingBy(e -> {
        if (e.getSalary() < 60000) return SalaryBand.LOW;
        else if (e.getSalary() <= 85000) return SalaryBand.MID;
        else return SalaryBand.HIGH;
    }));
```

Here the classifier isn't `Employee::getX` — it's a full lambda body that computes an enum value from multiple conditions. Same underlying idea: for each employee, run this logic, get a bucket label, drop the employee in that bucket.

Result:

```
LOW  -> [Carol(55k), Eve(50k)]
MID  -> [Bob(60k), Dave(70k)]
HIGH -> [Alice(90k), Frank(85k)]
```

**Important gotcha the slides mention:** a bucket key only appears in the map **if at least one element landed in it**. If nobody had a `LOW` salary, the key `LOW` simply wouldn't exist in the resulting map at all — not even as an empty list. This is different from `partitioningBy` (see below), which always creates both `true` and `false` keys no matter what.

---

### Step 2: `partitioningBy`

`partitioningBy` is like `groupingBy`, except the classification function must return a `boolean`, so there are always **exactly two buckets**: `true` and `false`.

java

```java
Map<Boolean, List<Employee>> managersVsNot =
    employees.stream().collect(partitioningBy(Employee::isManager));
```

Result:

```
true  -> [Alice, Dave, Frank]
false -> [Bob, Carol, Eve]
```

Use `partitioningBy` when the question is a yes/no split ("is vegetarian?", "is manager?", "is prime?"). Use `groupingBy` when there could be any number of categories (department names, salary bands, etc.).

Example from the slides, translated:

java

```java
public boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
                     .noneMatch(i -> candidate % i == 0);
}

Map<Boolean, List<Integer>> primeSplit =
    IntStream.rangeClosed(2, 50)
             .boxed()                          // IntStream -> Stream<Integer>
             .collect(partitioningBy(this::isPrime));
```

`primeSplit.get(true)` gives you all primes up to 50; `primeSplit.get(false)` gives you all composites.

---

### Step 3: Downstream collector

Ask yourself: _"I want one number/value per department, not a whole list."_ That's exactly when you swap out the default `toList()` downstream collector for something else.

#### Example: How many employees per department?

java

```java
Map<String, Long> countByDept =
    employees.stream()
             .collect(groupingBy(Employee::getDepartment, counting()));
```

Result: `{IT=2, HR=2, Sales=2}`

Read this as: "group by department, and **within each department-bucket**, instead of collecting a list, just count how many landed there."

#### Example: Highest-paid employee per department

java

```java
Map<String, Optional<Employee>> topPaidByDept =
    employees.stream()
             .collect(groupingBy(
                 Employee::getDepartment,
                 maxBy(Comparator.comparingInt(Employee::getSalary))
             ));
```

Result:

```
IT    -> Optional[Alice(90000)]
HR    -> Optional[Dave(70000)]
Sales -> Optional[Frank(85000)]
```

**Why is the value wrapped in `Optional`?** Because `maxBy` itself, on its own, always returns an `Optional<T>` (a plain stream _could_ be empty, so `maxBy` has to account for "no maximum exists"). When you plug `maxBy` in as the downstream collector, that `Optional` wrapping comes along for the ride, per-bucket.

#### Example: Total salary per department

java

```java
Map<String, Integer> totalSalaryByDept =
    employees.stream()
             .collect(groupingBy(Employee::getDepartment,
                                  summingInt(Employee::getSalary)));
```

Result: `{IT=150000, HR=125000, Sales=135000}`

#### Example: Average salary per department

java

```java
Map<String, Double> avgSalaryByDept =
    employees.stream()
             .collect(groupingBy(Employee::getDepartment,
                                  averagingInt(Employee::getSalary)));
```

**The pattern you should internalize:**

```
groupingBy( classifier ,  downstreamCollector )
             │                    │
       "which bucket?"    "what do I compute
                            from the elements
                            inside that bucket?"
```

---

### Step 4: `mapping`: transform elements _before_ they get collected inside a bucket

#### Example: Just the names of employees per department (not full Employee objects)

java

```java
Map<String, List<String>> namesByDept =
    employees.stream()
             .collect(groupingBy(Employee::getDepartment,
                                  mapping(Employee::getName, toList())));
```

Result: `{IT=[Alice, Bob], HR=[Carol, Dave], Sales=[Eve, Frank]}`

Compare this to plain `groupingBy(Employee::getDepartment)`, which would give you `{IT=[Employee@..., Employee@...], ...}` — full objects. `mapping(transformFn, downstreamCollector)` says: "for each element about to enter this bucket, first transform it with `transformFn`, _then_ hand the transformed value to `downstreamCollector`."

```
mapping( Employee::getName ,  toList() )
              │                   │
       "transform each        "then collect
        element first"         the transformed
                                values how?"
```

#### Example: Distinct salary-bands present per department

```java
Map<String, Set<SalaryBand>> bandsPresentByDept =
    employees.stream()
             .collect(groupingBy(Employee::getDepartment,
                 mapping(e -> {
                     if (e.getSalary() < 60000) return SalaryBand.LOW;
                     else if (e.getSalary() <= 85000) return SalaryBand.MID;
                     else return SalaryBand.HIGH;
                 }, toSet())));
```

This exactly mirrors the slide's `caloricLevelsByType` example — for each dish, first classify it into a `CaloricLevel`, then collect those levels per `Dish.Type` into a `Set` (deduping repeated levels).

---

### Step 5: `collectingAndThen`

Recall the "annoying `Optional`" problem from Step 3. `collectingAndThen` lets you take _any_ collector, and after it finishes, run one more transformation on whatever it produced.

java

```java
collectingAndThen( downstreamCollector ,  finisherFunction )
                          │                      │
              "do the real collecting"    "then tweak the
                                            final result"
```

#### Example: Highest-paid employee per department, but unwrap the `Optional`

java

```java
Map<String, Employee> topPaidByDept =
    employees.stream()
             .collect(groupingBy(
                 Employee::getDepartment,
                 collectingAndThen(
                     maxBy(Comparator.comparingInt(Employee::getSalary)),
                     Optional::get   // unwrap Optional<Employee> -> Employee
                 )
             ));
```

Now `topPaidByDept` is a clean `Map<String, Employee>` — no `Optional` noise. This is safe _specifically because_ we already know every bucket that exists has at least one employee in it (recall: `groupingBy` never creates empty buckets), so `.get()` will never throw.

#### Example: Make the resulting list per department unmodifiable

java

```java
Map<String, List<Employee>> immutableByDept =
    employees.stream()
             .collect(groupingBy(Employee::getDepartment,
                 collectingAndThen(toList(), Collections::unmodifiableList)));
```

Here, the downstream collector `toList()` does the real collecting, and then `Collections::unmodifiableList` wraps each department's list so it can't be modified afterward.

---

### Step 6: `toCollection` — control exactly which concrete collection type you get

`toList()`/`toSet()` let the library pick _some_ implementation (usually `ArrayList`/`HashSet`). If you specifically need a `LinkedList`, a `TreeSet`, a `LinkedHashSet`, etc., use `toCollection(constructorReference)`.

#### Example: Force a `TreeSet` (sorted) of employee names per department

java

```java
Map<String, TreeSet<String>> sortedNamesByDept =
    employees.stream()
             .collect(groupingBy(Employee::getDepartment,
                 mapping(Employee::getName, toCollection(TreeSet::new))));
```

Now each department's names come out alphabetically sorted, because they're stored in a `TreeSet`, not just an unordered `HashSet`.

```
mapping( Employee::getName ,  toCollection(TreeSet::new) )
```

Read it the same way as before — "transform, then collect" — just with a more specific downstream collector this time.

---

### Step 7: Multilevel (nested) grouping — the trick is that `groupingBy` can be its own downstream

Here's the mental leap the slides are making that's easy to miss: **the downstream collector slot can itself be another `groupingBy`.** That's it. That's the entire "multilevel grouping" concept.

```
groupingBy( outer classifier , groupingBy( inner classifier , ... ) )
```

#### Example: Group employees by department, and _within_ each department, by manager status

java

```java
Map<String, Map<Boolean, List<Employee>>> deptThenManager =
    employees.stream()
             .collect(groupingBy(Employee::getDepartment,
                       groupingBy(Employee::isManager)));
```

Result:

```
IT    -> { true=[Alice], false=[Bob] }
HR    -> { true=[Dave], false=[Carol] }
Sales -> { true=[Frank], false=[Eve] }
```

Trace how this is built, bucket by bucket:

```
Step 1 (outer groupingBy): split everyone by department
   IT:    [Alice, Bob]
   HR:    [Carol, Dave]
   Sales: [Eve, Frank]

Step 2 (inner groupingBy, run SEPARATELY on each department's list):
   IT bucket's employees --> grouped by isManager --> {true=[Alice], false=[Bob]}
   HR bucket's employees --> grouped by isManager --> {true=[Dave], false=[Carol]}
   Sales bucket's employees --> grouped by isManager --> {true=[Frank], false=[Eve]}
```

The outer `groupingBy` doesn't know or care that its downstream collector happens to be another `groupingBy` — as far as it's concerned, it's just handing each bucket's list of elements to "some collector" and storing whatever comes back as the value. It's exactly the same mechanism as `groupingBy(dept, counting())` from Step 3 — just with a fancier downstream collector plugged in.

#### Example: Three levels deep (mirrors the slide's `result5` exactly, using employees)

Group by department → then by manager status → then by salary band:

java

```java
Map<String, Map<Boolean, Map<SalaryBand, List<Employee>>>> threeLevel =
    employees.stream().collect(
        groupingBy(Employee::getDepartment,
            groupingBy(Employee::isManager,
                groupingBy(e -> {
                    if (e.getSalary() < 60000) return SalaryBand.LOW;
                    else if (e.getSalary() <= 85000) return SalaryBand.MID;
                    else return SalaryBand.HIGH;
                })
            )
        )
    );
```

Reading the type back-to-front tells you the nesting order:

```
Map<String,        Map<Boolean,       Map<SalaryBand,   List<Employee>>>>
     ↑                    ↑                   ↑                ↑
  1st level          2nd level           3rd level      final buckets
 (department)       (isManager)        (salary band)
```

You can nest as many `groupingBy` calls as you want this way — each one just becomes the downstream collector for the one before it. There's no special "n-level" syntax; it's the same two-argument `groupingBy(classifier, downstream)` pattern, recursively.

---

### Step 8: Mixing it all together

You're not limited to nesting only `groupingBy` inside `groupingBy`. Any collector can go in that downstream slot, at any level. For instance:

#### Example: Per department, per manager-status, what's the average salary?

java

```java
Map<String, Map<Boolean, Double>> avgSalaryByDeptAndManager =
    employees.stream()
             .collect(groupingBy(Employee::getDepartment,
                       groupingBy(Employee::isManager,
                                  averagingInt(Employee::getSalary))));
```

Here the **innermost** downstream collector is `averagingInt`, not another `groupingBy` — so the nesting stops at 2 levels, and the final value is a plain `Double`, not a list.

Result e.g.: `IT -> {true=90000.0, false=60000.0}`

#### Example: Per department, names of managers only, sorted

java

```java
Map<String, TreeSet<String>> managerNamesByDept =
    employees.stream()
             .filter(Employee::isManager)                       // keep only managers first
             .collect(groupingBy(Employee::getDepartment,
                       mapping(Employee::getName, toCollection(TreeSet::new))));
```

Notice you can `filter` _before_ the `groupingBy` too — decide what even enters the grouping process in the first place.

---

### Quick reference: how to build any grouping query yourself

Ask these three questions, in order:

1. **"What are my buckets?"** → that's your classification function, the first argument to `groupingBy` (or `partitioningBy` if it's a strict yes/no split).
2. **"What do I want per bucket — a list of raw items, or something computed from them?"**
    - Raw list → don't specify a downstream collector at all (defaults to `toList()`).
    - A number (count/sum/average) → `counting()`, `summingInt()`, `averagingInt()`.
    - One "best" item → `maxBy(comparator)` / `minBy(comparator)`.
    - A transformed list/set → `mapping(transformFn, toList()/toSet()/toCollection(...))`.
    - Sub-buckets within each bucket → nest another `groupingBy(...)` as the downstream.
3. **"Do I need to clean up the final shape?"** (e.g. unwrap an `Optional`, make it immutable) → wrap your downstream collector in `collectingAndThen(downstream, finisherFn)`.

Once you can answer those three questions for a problem, writing the `groupingBy(...)` call is just plugging the answers into the template:


```java
groupingBy(
    /* Q1 */ classifierFunction,
    /* Q2 (+ Q3 if needed) */ downstreamCollector
)
```