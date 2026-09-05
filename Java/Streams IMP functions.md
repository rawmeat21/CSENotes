### 1. Stream Methods (Intermediate & Terminal Operations)

#### Intermediate operations (return a new Stream — lazy, chainable)

|Method|Signature|What it does|Example|
|---|---|---|---|
|`filter`|`Stream<T> filter(Predicate<T>)`|Keeps only elements matching a condition|`.filter(d -> d.getCalories() > 350)`|
|`map`|`<R> Stream<R> map(Function<T,R>)`|Transforms each element into one new element|`.map(Dish::getName)`|
|`flatMap`|`<R> Stream<R> flatMap(Function<T, Stream<R>>)`|Transforms each element into a stream, then flattens all of them into one stream|`.flatMap(Arrays::stream)`|
|`distinct`|`Stream<T> distinct()`|Removes duplicate elements (stateful)|`.distinct()`|
|`limit`|`Stream<T> limit(long n)`|Truncates stream to at most `n` elements (short-circuiting)|`.limit(3)`|
|`skip`|`Stream<T> skip(long n)`|Discards the first `n` elements|`.skip(5)`|
|`sorted`|`Stream<T> sorted()` / `sorted(Comparator)`|Sorts the stream (stateful) — used conceptually, mentioned alongside `distinct`|`.sorted(Comparator.comparing(Dish::getCalories))`|
|`boxed`|`Stream<Integer> boxed()`|Converts a primitive `IntStream`/`LongStream`/`DoubleStream` into an object `Stream`|`IntStream.rangeClosed(2,n).boxed()`|

#### Terminal operations (end the pipeline — eager, consume the stream)

|Method|Signature|What it does|Example|
|---|---|---|---|
|`collect`|`<R,A> R collect(Collector<T,A,R>)`|Accumulates stream elements into a result using a `Collector`|`.collect(toList())`|
|`count`|`long count()`|Counts elements|`.count()`|
|`forEach`|`void forEach(Consumer<T>)`|Performs an action on each element; returns `void`|`.forEach(System.out::println)`|
|`reduce`|`T reduce(T identity, BinaryOperator<T>)`|Combines all elements into a single value (fold), immutable|`.reduce(0, (a,b) -> a+b)`|
|`anyMatch`|`boolean anyMatch(Predicate<T>)`|`true` if at least one element matches (short-circuiting)|`.anyMatch(d -> d.getCalories()<400)`|
|`allMatch`|`boolean allMatch(Predicate<T>)`|`true` if every element matches|`.allMatch(Dish::isVegetarian)`|
|`noneMatch`|`boolean noneMatch(Predicate<T>)`|`true` if no element matches|`.noneMatch(i -> candidate % i == 0)`|
|`findAny`|`Optional<T> findAny()`|Returns any matching element (short-circuiting, parallel-friendly)|`.findAny()`|
|`findFirst`|`Optional<T> findFirst()`|Returns the first element in encounter order (more constraining in parallel)|`.findFirst()`|

#### Stream creation methods

|Method|What it does|Example|
|---|---|---|
|`collection.stream()`|Creates a stream from a collection|`menu.stream()`|
|`Stream.of(...)`|Creates a stream directly from given values|`Stream.of("a","b","hello")`|
|`IntStream.rangeClosed(a, b)`|Creates a primitive int stream, `a` to `b` **inclusive**|`IntStream.rangeClosed(2, candidateRoot)`|

**General development note:** `filter`/`map`/`flatMap`/`collect`/`reduce`/`anyMatch`/`findAny` are the ones you'll reach for constantly in real code — they replace 90% of manual for-loops over collections. `flatMap` specifically is the one people forget exists and then write awkward nested loops instead.

---

### 2. Collectors (from `java.util.stream.Collectors`, usually static-imported)

#### Basic collection-builders

|Collector|What it produces|Example|
|---|---|---|
|`toList()`|`List<T>`|`.collect(toList())`|
|`toSet()`|`Set<T>`|`.collect(toSet())`|
|`toCollection(supplier)`|A specific concrete collection type|`.collect(toCollection(TreeSet::new))`|

#### Reducing/summarizing to a single value

|Collector|What it produces|Example|
|---|---|---|
|`counting()`|`Long` — count of elements|`groupingBy(Dish::getType, counting())`|
|`summingInt(fn)`|`Integer` — sum of an int property|`summingInt(Dish::getCalories)`|
|`averagingInt(fn)`|`Double` — average of an int property|`averagingInt(Employee::getSalary)`|
|`summarizingInt(fn)`|`IntSummaryStatistics` (count, sum, min, max, average all at once)|`summarizingInt(Dish::getCalories)`|
|`maxBy(comparator)`|`Optional<T>` — max element|`maxBy(Comparator.comparingInt(Dish::getCalories))`|
|`minBy(comparator)`|`Optional<T>` — min element|`minBy(comparator)`|
|`joining(delim, prefix, suffix)`|`String` — concatenates elements|`joining(",", "[", "]")`|
|`reducing(identity, mapper, op)`|Generic fold, as a collector|`reducing(" ", Dish::getName, (i,j)->i+j)`|

#### Grouping/partitioning (the "bucket" collectors)

|Collector|What it produces|Example|
|---|---|---|
|`groupingBy(classifier)`|`Map<K, List<T>>` — shorthand for `groupingBy(classifier, toList())`|`groupingBy(Dish::getType)`|
|`groupingBy(classifier, downstream)`|`Map<K, D>` — buckets, each processed by `downstream`|`groupingBy(Dish::getType, counting())`|
|`partitioningBy(predicate)`|`Map<Boolean, List<T>>` — always exactly 2 keys|`partitioningBy(Dish::isVegetarian)`|

#### Downstream-collector modifiers (used _inside_ `groupingBy`/`partitioningBy`)

|Collector|What it does|Example|
|---|---|---|
|`mapping(transformFn, downstream)`|Transforms each element first, then feeds it to another collector|`mapping(Dish::getName, toList())`|
|`collectingAndThen(downstream, finisherFn)`|Runs `downstream`, then post-processes the final result|`collectingAndThen(maxBy(cmp), Optional::get)`|

**General development note:** `groupingBy`, `partitioningBy`, `counting`, `summingInt`, `joining`, and `toList`/`toSet` are the ones you'll use in almost every real project involving reports, dashboards, or data summarization (e.g., "group orders by customer," "count users by status," "total revenue by region"). `mapping` and `collectingAndThen` are less common but essential once you need anything more than the plain default behavior.

---

### 3. Functional Interfaces (used via lambdas — conceptually "anonymous classes")

Every lambda you write (`d -> d.getCalories() > 350`) is really shorthand for an anonymous implementation of some **functional interface** (an interface with exactly one abstract method). These are the ones used throughout:

|Interface|Abstract method|Used for|Example|
|---|---|---|---|
|`Predicate<T>`|`boolean test(T t)`|`filter`, `anyMatch`, `allMatch`, `noneMatch`, `partitioningBy`|`artist -> artist.isFrom("London")`|
|`Function<T,R>`|`R apply(T t)`|`map`, classification functions in `groupingBy`, `mapping`|`artist -> artist.getNationality()`|
|`BinaryOperator<T>`|`T apply(T t1, T t2)`|`reduce`, the combining logic in `reducing()`|`(a, b) -> a + b`|
|`Consumer<T>`|`void accept(T t)`|`forEach`, `ifPresent`|`d -> System.out.println(d.getName())`|
|`Comparator<T>`|`int compare(T t1, T t2)`|`maxBy`, `minBy`, `sorted`|`Comparator.comparingInt(Dish::getCalories)`|
|`Supplier<T>`|`T get()`|Used implicitly as the "creation" role in `Collector`'s `supplier()`; also `toCollection(HashSet::new)` uses a constructor reference as a `Supplier`|`HashSet::new`|

**Why this matters for general development (not just exams):** These six interfaces are the backbone of _all_ functional-style Java, not just streams — you'll see them in `Optional.filter()`, `CompletableFuture`, event listeners, dependency injection frameworks, and anywhere APIs accept a "block of behavior" as a parameter. Recognizing "this lambda's shape matches `Function<T,R>`" or "this needs a `Predicate`" is the core skill for reading and writing idiomatic modern Java — even outside of Streams entirely.

A useful mental shortcut:

```
takes nothing, returns a value      → Supplier<T>
takes a value, returns nothing      → Consumer<T>
takes a value, returns boolean      → Predicate<T>
takes a value, returns another type → Function<T,R>
takes two values of same type,
   returns same type                → BinaryOperator<T>
takes two values, returns int
   (for ordering)                   → Comparator<T>
```