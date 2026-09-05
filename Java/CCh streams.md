## Streams and Functional Programming in Java — Full Tutorial

### 1. External Iteration

This is the "old style" of iterating, **you** control the loop.

java

```java
int count = 0;
Iterator<Artist> iterator = allArtists.iterator();
while (iterator.hasNext()) {
    Artist artist = iterator.next();
    if (artist.isFrom("London")) {
        count++;
    }
}
```

Equivalent enhanced for-loop version (also external iteration):

java

```java
int count = 0;
for (Artist a : allArtists) {
    if (a.isFrom("London")) {
        count++;
    }
}
```

![[Pasted image 20260829144343.png]]

**Key points to remember:**

- Inherently serial in nature
- Hard to parallelize

**Why:** with external iteration, _your code_ asks the collection "do you have a next element?" and "give me the next element" step by step. That interaction is a strict, one-at-a-time conversation between your code and the collection, there's no room for the collection to decide "I'll hand off different elements to different threads."

---

### 2. Internal Iteration

Instead of you pulling elements out one by one, you _describe what you want done_, and the library iterates for you.

java

```java
long count = allArtists.stream()
                        .filter(artist -> artist.isFrom("London"))
                        .count();
```

![[Pasted image 20260829144356.png]]

**Key points:**

- Instead of returning an `Iterator` to control the iteration, it returns the equivalent interface in the internal iteration world: **Stream**.
- A Stream is a tool for building up complex operations on collections using a functional approach.
- The functions performed are:
    - Finding all the artists from London
    - Counting a list of artists

**How to read this syntax:** `.stream()` converts a collection into a `Stream`. Then you chain **operations** (`filter`, `map`, etc.) that each return a new stream, and finally a **terminal operation** (`count`, `collect`, etc.) that produces the actual result.

---

### 3. Streams

**Definition:**

> Streams can be defined as a sequence of elements from a source that supports data processing operations.

Also stated:

- Collections are data structures focusing on storing and accessing of elements.
- Streams are about expressing computations.
- Unlike collection, stream provides an interface to a sequence of specific type of elements.

#### Source

- Streams consume data from a data providing source such as collections, arrays, or I/O resources.
- Streams from an ordered collection preserve the ordering.

#### Data processing operations

- Supports both database-like operations and functional programming operations to manipulate data.
- Operations can be executed in sequence or in parallel.


```java
long count = allArtists.stream()
                        .filter(artist -> artist.isFrom("London"))
                        .count();
```

---

### 4. filter → map → collect (worked example)

Using a `Dish` class with this shape (used throughout the slides):

```java
class Dish {
    private final String name;
    private final boolean vegetarian;
    private final int calories;
    private final Type type;

    public Dish(String name, boolean vegetarian, int calories, Type type) { ... }
    public String getName() { ... }
    public boolean isVegetarian() { ... }
    public int getCalories() { ... }
    public Type getType() { ... }
    public String toString() { ... }
    public enum Type { MEAT, FISH, OTHER }
}
```

Example pipeline:

java

```java
menu.stream()
    .filter(d -> d.getCalories() > 350)
    .map(d1 -> d1.getName())
    .collect(toList());
```

Read this left to right: "start from `menu`'s stream, keep only dishes with more than 350 calories, transform each remaining dish into just its name, and collect the names into a `List`."

---

### 5. Stream Operations

**Two categories (verbatim):**

- **Intermediate**: `filter`, `map` -> these return another Stream, so you can chain more operations after them.
- **Terminal**: `collect`, `count` -> these end the pipeline and produce a non-stream result (a value, a collection, etc.).

#### Characteristics of stream operations:

- **Pipelining**
- **Laziness**
- **Short-circuiting**
- **Internal Iterations**

Example demonstrating pipelining and short-circuiting:

java

```java
menu.stream()
    .filter(d -> d.getCalories() > 350)
    .map(d1 -> d1.getName())
    .limit(3)
    .collect(toList());
```

**Explanations given in the slide:**

- **Loop fusion (pipelining)**: filter and map are two separate operations that are merged into one pass (the stream doesn't fully materialize an intermediate list after `filter` and then loop again for `map` — it processes each element through the whole chain in one traversal).
- **Short circuiting**: despite the fact that there are many high-calorie dishes, only the first 3 are selected (once `limit(3)` is satisfied, the stream stops pulling more elements).

**Laziness** (general concept, ties into the above): intermediate operations like `filter`/`map` don't actually run anything when you call them, they just build up a description of the pipeline. Nothing happens until a terminal operation (like `collect`) is invoked. This is what allows short-circuiting to work efficiently.

---

### 6. Stream vs Collection (verbatim comparison table)

|Stream|Collection|
|---|---|
|fixed data structure whose elements are computed on demand|every element is computed before it is added to a collection|
|lazily constructed collection|eagerly constructed collection|
|Consumer driven|Supplier driven|
|Traversable exactly once|No such restriction|
|Stream is a set of values spread out in time|A set of values spread out in space|
|Internal iteration|External iteration|

**"Traversable exactly once" is important:** once you call a terminal operation on a stream, that stream is _consumed_. You cannot reuse the same `Stream` object for a second terminal operation, you'd get an `IllegalStateException`. If you need to iterate again, call `.stream()` on the source collection again.

java

```java
Stream<String> s = list.stream();
s.forEach(System.out::println);
s.forEach(System.out::println); // throws IllegalStateException: stream has already been operated upon or closed
```

---

### 7. External vs Internal Iteration

**Internal Iteration:**

- Processing of elements can be done in parallel or in a different order that is more optimized.
- Stream library can automatically choose a data representation and implementation of parallelism to match the machine hardware.

**External Iteration:**

- Programmer needs to implement parallelism and define the order in which the elements of a collection can be processed.
- Committed to a single-threaded step-by-step sequential iteration.

---

### 8. Filtering

**Key points:**

- Where clause of a select statement (i.e., it's like SQL's `WHERE`)
- Takes a `Predicate` object as an argument
- Returns a stream including all elements that match with the predicate
- If you're refactoring legacy code, the presence of an `if` statement in the middle of a `for` loop is a pretty strong indicator that you really want to use `filter`.

![[Pasted image 20260830153410.png]]

A `Predicate<T>` is a functional interface with one abstract method: `boolean test(T t)`. `filter()` takes a `Predicate<T>` and keeps only elements for which `test()` returns `true`.

**Code example:

java

```java
List<String> WithNos = Stream.of("a", "1ab", "1A", "2A")
                              .filter(d2 -> Character.isDigit(d2.charAt(0)))
                              .collect(toList());
```

Walkthrough: `Stream.of(...)` creates a stream directly from values (not from a collection). `filter` keeps only strings whose first character is a digit → result: `["1ab", "1A", "2A"]`.

The slide's diagrams illustrate that `filter` inspects every element and only lets matching ones through to the output stream (green squares pass, non-matching are dropped), and applying the same idea to numbers keeps only elements meeting the predicate while dropping the rest.

---

### 9. Truncating a Stream

**`limit(n)`:**

- Streams support the `limit(n)` method, which returns another stream that's no longer than a given size.
- The requested size is passed as argument to `limit`.
- If the stream is ordered, the first elements are returned up to a maximum of `n`.

**`skip(n)`:**

- Streams support the `skip(n)` method to return a stream that discards the first `n` elements.
- If the stream has fewer elements than `n`, then an empty stream is returned.

java

```java
List<Integer> firstThree = numbers.stream().limit(3).collect(toList());
List<Integer> afterFirstThree = numbers.stream().skip(3).collect(toList());
```

Common pattern, pagination-style processing:

java

```java
List<Dish> page2 = menu.stream()
                        .skip(5)   // skip page 1 (5 items)
                        .limit(5)  // take page 2 (next 5 items)
                        .collect(toList());
```

---

### 10. Mapping

![[Pasted image 20260904131200.png]]

**Key points:**

- The function is applied to each element, mapping it into a new element.
- The word _mapping_ is used because it has a meaning similar to _transforming_ but with the nuance of "creating a new version of" rather than "modifying".
- Example use case: converting strings to uppercase equivalents.

**Map's signature (diagram):**

```
T ──► [ Function ] ──► R (T and R can be different)
```

`map` takes a `Function<T, R>`: a functional interface with `R apply(T t)`, and produces a new stream where each element has been transformed.

**Imperative version:**

java

```java
List<String> collected = new ArrayList<>();
for (String string : asList("a", "b", "hello")) {
    String uppercaseString = string.toUpperCase();
    collected.add(uppercaseString);
}
```

**Stream version:**

java

```java
List<String> collected = Stream.of("a", "b", "hello")
                                .map(st -> st.toUpperCase())
                                .collect(toList());
```

Notice: `map` doesn't change the original strings (Strings are immutable anyway) — it produces _new_ elements in a _new_ stream, which is exactly the "creating a new version of" nuance mentioned above.

Another example set up (from slide, for you to complete as practice):

java

```java
List<String> words = Arrays.asList("Java8", "Lambdas", "In", "Action");
List<Integer> wordLengths = words.stream()
                                  .map(String::length)
                                  .collect(toList());
// [5, 7, 2, 6]
```

#### Mapping pitfall: mapping to arrays

The slide poses the question: _how could you return a list of all the unique characters for a list of words?_

java

```java
List<String> word1 = Arrays.asList("Hi", "Hello", "Hi", "Hi", "Hello", "Hell", "Heaven");
distinctLetters = word1.stream()
                        .map(w -> w.split(""))
                        .distinct()
                        .collect(toList());
```

**This is wrong / doesn't do what you want**. The problem: `map(w -> w.split(""))` turns each word into a `String[]` (an array), so you end up with a `Stream<String[]>`: a stream _of arrays_. `distinct()` then compares whole arrays (by reference/equality of the array objects), not individual letters, so it does **not** give you unique letters across all words.

---

### 11. FlatMap

```java
List<String> uniqueChars = word1.stream()
                                 .map(w -> w.split(""))
                                 .flatMap(Arrays::stream)
                                 .distinct()
                                 .collect(toList());
```

This is also correct:

```java
List<String> uniqueChars = word1.stream()
                                 .map(w -> w.split(""))
                                 .flatMap(arr -> Arrays.stream(arr))
                                 .distinct()
                                 .collect(toList());
```

![[Pasted image 20260904131141.png]]
![[Pasted image 20260904131404.png]]

![[Pasted image 20260904131515.png]]


#### FlatMap examples

**Form a pair of numbers taking each number from two lists of numbers:**

java

```java
numbers1.stream()
        .flatMap(i -> numbers2.stream()
        .map(k -> new int[]{i, k}))
        .collect(toList());
```

```java
numbers1.stream().
        .flatMap(i -> numbers2.stream().map(j -> new int[]{i, j}))
        .collect(Collectors.toList());
```

Filter:

```java
numbers1.stream().
        .flatMap(i -> numbers2.stream().filter(j -> (i + j) % 3 == 0).map(j -> new int[]{i, j}))
        .collect(Collectors.toList());
```

**Form a list of numbers that represents pairwise summations of numbers taking each number from two lists of numbers. Each number should appear exactly once in the list.**

java

```java
List<Integer> numbers1 = Arrays.asList(1, 2, 3);
List<Integer> numbers2 = Arrays.asList(1, 2, 3, 4);

numbers1.stream()
        .flatMap(i -> numbers2.stream().map(j -> (i + j)))
        .forEach(k -> System.out.println(" " + k));

// output: 2 3 4 5 3 4 5 6 4 5 6 7
```

**The core idea to remember:** use `map` when each input element maps to exactly one output element; use `flatMap` when each input element maps to _zero or more_ output elements (like a word → its letters, or a number → a set of pairs) and you want a single flat stream out.

The rule to follow is: in the lambda inside flatMap, you must somehow map from an **object of a stream** to a **stream**.

```java

List<List<String>> phoneNumbers = customers.stream().map(customer -> customer.getPhoneNumbers()).collect(Collectors.toList()); // using a map, returns nested lists.

// using flatMap, just return a stream
List<String> phones = customers.stream().flatMap(customer -> customers.getPhoneNumbers().stream()).collect(Collectors.toList());

```

---

### 12. Finding and Matching

**`anyMatch`:**

- Takes a **predicate** as argument and returns `true` if there is at least one element matching the criteria from the stream.

**Also listed:**

- `allMatch(Predicate)`
- `noneMatch(Predicate)`

**Example question (verbatim):** _Is there any non-vegetarian dish in the menu that results in less than 400 calories?_

java

```java
menu.stream().anyMatch(d -> !d.isVegetarian() && d.getCalories() < 400)
```

To round this out for you (same pattern applies to the other two):

java

```java
boolean allVeg = menu.stream().allMatch(Dish::isVegetarian);
boolean noneOver1000 = menu.stream().noneMatch(d -> d.getCalories() > 1000);
```

These are identical btw:
```java
// Using a Lambda Expression:
boolean allVeg = menu.stream().allMatch(d -> d.isVegetarian());

// Using a Method Reference (shorthand):
boolean allVeg = menu.stream().allMatch(Dish::isVegetarian);
```
In the 2nd case, `allMatch()` takes in a `Predicate<T>`, now this uses a `bool test(T obj)` function, so you _can_ pass in another function like `Dish::isVegetarian` which has the same signature.

All three of these are **terminal** operations that return a `boolean`, and all three are **short-circuiting**, they don't necessarily need to check every element.

---

### 13. Short Circuiting

**Key points:**

- The matching functions do not need to process the entire stream to give the result.
- They can turn an infinite stream to constant size.
- Examples of short-circuiting operations: `limit`, `findFirst` (more constraining for parallel streams), `findAny`.

**Example:**

```java
menu.stream()
    .filter(d -> !d.isVegetarian() && d.getCalories() < 400)
    .findAny()
    .ifPresent(d -> System.out.println(d.getName()));
```

**Why `findFirst` is "more constraining for parallel streams":** `findFirst` must respect the encounter order of the stream — even when running in parallel, it has to figure out which element is truly "first," which limits how freely the work can be parallelized. `findAny` has no such constraint, any matching element will do, so it's more parallel-friendly.

Note that `findAny`/`findFirst` return an `Optional<T>`.

---

### 14. Optional

**Definition:**

> The `Optional<T>` class (`java.util.Optional`) is a container class to represent the existence or absence of a value.

```java
if (value != null) value.someMethod();
```

`Optional` exists to avoid bugs related to null checking, instead of returning `null` and hoping every caller remembers to check for it, a method returns an `Optional<T>` that explicitly forces you to consider the "absent" case.

**Optional's key methods:**

- `isPresent()` -> returns `true` if Optional contains a value, `false` otherwise.
- `T get()` -> returns the value if present; otherwise it throws a `NoSuchElementException`.
- `ifPresent(Consumer<T> block)` -> executes the given block if a value is present.
- `T orElse(T other)` -> returns the value if present; otherwise it returns a default value.

java

```java
Optional<Dish> spicy = menu.stream()
                            .filter(d -> !d.isVegetarian() && d.getCalories() < 400)
                            .findAny();

if (spicy.isPresent()) {
    System.out.println(spicy.get().getName());
}

// cleaner, idiomatic way:
spicy.ifPresent(d -> System.out.println(d.getName()));

// with a default fallback:
Dish chosen = spicy.orElse(defaultDish);
```

---

### 15. Predicate, Map and FlatMap Questions

1. Given a list, square each number
2. Find a number from a given list, whose squares are divisible by 3
3. Given 2 lists of numbers, form pairs of numbers such that the sum of the numbers is even
4. Count the number of lowercase letters in a String (hint: look at the `chars` method on `String`).

Since this is a tutorial, here's how you'd solve each with what you've learned so far:

**1. Square each number:**

java

```java
List<Integer> squares = numbers.stream()
                                .map(n -> n * n)
                                .collect(toList());
```

**2. Find numbers whose squares are divisible by 3:**

java

```java
List<Integer> result = numbers.stream()
                               .filter(n -> (n * n) % 3 == 0)
                               .collect(toList());
```

**3. Pairs of numbers (from two lists) whose sum is even:**

java

```java
List<int[]> evenSumPairs =
    numbers1.stream()
            .flatMap(i -> numbers2.stream()
                                   .filter(j -> (i + j) % 2 == 0)
                                   .map(j -> new int[]{i, j}))
            .collect(toList());
```

**4. Count lowercase letters in a String:**

java

```java
long lowerCount = someString.chars()
                             .filter(Character::isLowerCase)
                             .count();
```

(`String.chars()` returns an `IntStream` of the character codes, which is why `filter` takes an `int`-based predicate here, `Character::isLowerCase` matches that signature.)

---

### 16. Terminal Operations

**Key points:**

- So far, the terminal operations are found to return a `boolean` (`allMatch` and so on), `void` (`forEach`), an `Optional` object (`findAny` and so on).

**Reducing:**

- Combines all elements of the stream repeatedly to produce a single value as result. This is called **fold**.

---

### 17. Reducing

**Motivating example: summing up elements:**

java

```java
int sum = 0;
for (int x : numbers) {
    sum += x;
}
```

**Key point (verbatim):**

> The `reduce` operation abstracts over this pattern of repeated application.

**`reduce` takes two arguments (verbatim):**

- An initial value, here `0`.
- A `BinaryOperator<T>` to combine two elements and produce a new value; here you use the lambda `(accumulator, element) -> accumulator + element`.


```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
```

#### `REDUCE(0, (A,B) -> A+B)` diagram, traced through

For the stream `[4, 5, 3, 9]`:

```
0 + 4  = 4
4 + 5  = 9
9 + 3  = 12
12 + 9 = 21   <- final result: Integer 21
```

So `reduce` starts with the initial value (`0`), combines it with the first element to get an intermediate result, then keeps combining that running result with the next element, and so on, until one final value comes out.

---

### 18. Mutable Accumulator vs Fork Join

This slide explains how `reduce` can be **parallelized** using a fork/join (divide and conquer) strategy.

For stream `[1,2,3,4,5,6,7,8]` split into two chunks `[1,2,3,4]` and `[5,6,7,8]`:

```
First chunk: 0+1=1, 1+2=3, 3+3=6, 6+4=10
Second chunk: 0+5=5, 5+6=11, 11+7=18, 18+8=26

Combine: 10 + 26 = 36
```

**Key requirements for this parallelization to be valid:**

- Only for **associative** operations
- The operations must be **non-interfering**, that is, does not affect the data source
- **Stateless** and **deterministic**

These are the same rules that govern any operation you hand to a parallel stream: the accumulator function must give the same result regardless of the order operands are combined in (associativity), it must not read/mutate any shared external state (statelessness/non-interference), and repeated runs must produce the same answer (determinism). Addition satisfies all of these; something like "subtract" would not (not associative), and anything that mutates a shared list while streaming over it would violate non-interference.

---

### 19. Stream Operations: Stateless vs Stateful (See this again)

#### Stateless operations

- `map()` and `filter()` are stateless:
    - Take an input stream
    - Process each element of the stream
    - Produce 0 or 1 result in the output stream

Also mentioned as needing only small, bounded internal state: `sum`, `max`, `reduce`, `limit`, `skip`

- Need an internal state to accumulate the results
- The state is small and bounded
- It does not depend on the stream being processed

#### Stateful operations

`sort`, `distinct`:

- Take an input stream
- Process each element of the stream
- Produce 1 result in the output stream
- To compute they need the previous history (like `distinct`)
- Stateful operations → Unbounded storage space

**Important note:**

> The stream operations that do not pose an order are easier for parallelization.  
> Stream poses an encounter order in which each element is operated upon. This depends on both:
> 
> - the source of the data
> - the operation performed on the stream

Intuition: `filter`/`map` can decide the fate of one element in isolation, so they parallelize trivially. `sort`/`distinct` need to compare an element against _all_ the others seen so far, so they need to accumulate growing state as they go (which is why "unbounded storage space" is listed), this makes them harder to parallelize efficiently.

---

### 20. Version 1 (unrefactored)

java

```java
List<Artist> musicians = album.getMusicians()
                               .collect(toList());

List<Artist> bands = musicians.stream()
                               .filter(artist -> artist.getName().startsWith("S"))
                               .collect(toList());

Set<String> origins = bands.stream()
                            .map(artist -> artist.getNationality())
                            .collect(toSet());
```

#### Uses and Misuses (reasons this Version 1 style is bad)

- It's harder to read what's going on because the ratio of boilerplate code to actual business logic is worse.
- It's less efficient because it requires creating new collection objects at each intermediate step.
- It clutters your method with meaningless garbage variables that are needed only as intermediate results.
- It makes operations harder to automatically parallelize.

#### Version 2 (refactored, a single chained pipeline)

java

```java
Set<String> origins = album.getMusicians()
                            .filter(artist -> artist.getName().startsWith("A"))
                            .map(artist -> artist.getNationality())
                            .collect(toSet());
```

**Takeaway/exam point:** prefer chaining operations directly on a stream into one pipeline, rather than breaking it into multiple intermediate `List`/`Set` variables, this is both more readable and more efficient (no wasted intermediate collections), and easier to parallelize.

---

### 21. Collecting Stream Elements

**Key points (verbatim):**

- `collect` is a terminal operation that summarizes the stream while collecting the result.
- `collect(toList())`

java

```java
List<Dish> result = menu.stream().collect(toList());
```

**Note (important):**

> If the stream is parallel, and the Collector is concurrent, and either the stream is unordered or the collector is unordered, then a concurrent reduction will be performed.

---

### 22. Collecting Streams: The `Collector` interface

**Key points:**

- Collection, Collector, and collect are different (words that look similar but mean different things:`Collection` is the data structure interface, `Collector` is the recipe object, `collect` is the stream method).
- `Collector` interface: a general-purpose construct for producing complex values from streams.
- `Collector<T,A,R>`:
    - `T` -> Generic type of Stream elements
    - `A` -> Accumulator type
    - `R` -> Type of elements resulting from the collect operation

```java
R collect(Collector<? super T, A, R> collector)
```

---

### 23. Collectors

**Key points:**

- Collector applies a transforming function to the elements.
- For example, in `toList()` it is the identity transformation.
- Accumulates the results in a data structure.
- Predefined collectors can be created from the factory methods provided by the `Collectors` class.
- Collectors that are used are commonly statically imported from the `java.util.stream.Collectors` class.
- Normally when we create a collection, we specify the concrete type of the collection by calling the appropriate constructor:

java

```java
  List<Artist> artists = new ArrayList<>();
```

- But when you're calling `toList` or `toSet`, you don't get to specify the concrete implementation of the `List` or `Set`.
- Under the hood, the streams library is picking an appropriate implementation for you.

Practically, this means you typically do:

java

```java
import static java.util.stream.Collectors.*;
// ...
List<String> names = menu.stream().map(Dish::getName).collect(toList());
```

---

### 24. Collecting: how it actually works internally

![[Pasted image 20260904142934.png]]

The slide's diagram walks through processing a stream of transactions with a `Collector`:

1. Traverse each transaction in the stream.
2. Extract the transaction's currency.
3. Add the currency/transaction pair to the grouping map.

So a `Collector` is essentially a plug-in strategy: for every element in the stream, it runs a "transforming function," and then adds the transformed result into a growing result structure.

---

### 25. The Collector Functions

**A Collector is specified by four functions:**

1. Creation of a new result container, `supplier()`
2. Incorporating a new data element into a result container, `accumulator()`
3. Combining two result containers into one, `combiner()`
4. Performing an optional final transform on the container,`finisher()`

Also noted:

> Collectors also have a set of characteristics, such as `Collector.Characteristics.CONCURRENT`, that provide hints that can be used by a reduction implementation to provide better performance.

**How a sequential implementation works:**

- Create a single result container using the supplier function.
- And invoke the accumulator function once for each input element.

**How a parallel implementation works:**

- Partition the input
- Create a result container for each partition
- Accumulate the contents of each partition into a subresult for that partition
- Use the combiner function to merge the subresults into a combined result
- The combiner may fold state, returns a `BinaryOperator`

This is the exact same fork/join pattern you saw with `reduce` earlier: `Collector` is really the general, mutable version of that idea (supplier = "start", accumulator = "combine one more element in", combiner = "merge two partial results", finisher = "final polish").

---

### 26. Collecting Streams

- Reducing and summarizing stream elements to a single value
- Grouping elements
- Partitioning elements

---

### 27. Reducing and Summarizing

**Counting:**

> Count the no of menu items

java

```java
long countingDish = menu.stream().collect(Collectors.counting());
```

**`maxBy()` and `minBy()`:**

```java
Comparator<Dish> dishCaloriesComp = Comparator.comparing(Dish::getCalories);
Optional<Dish> TastyDish = menu.stream().collect(maxBy(dishCaloriesComp));
```

`Comparator.comparing()` diagram noted in slide: it's a `Function` that extracts a key, wrapped into a `Comparator`.

`maxBy`/`minBy` return an `Optional<T>` because the stream could be empty (no maximum exists on an empty stream).

---

### 28. Summarizing

**Collectors mentioned:**

- `Collectors.summingInt()`

java

```java
  menu.stream().collect(summingInt(d -> d.getCalories()));
```

- `averagingInt()`
- `summarizingInt()`
- `IntSummaryStatistics`

To fill this in with usage: `summingInt`/`averagingInt` give you a single `int`/`double` result. `summarizingInt` gives you back an `IntSummaryStatistics` object bundling count, sum, min, max, and average all at once:

java

```java
IntSummaryStatistics stats = menu.stream()
                                  .collect(summarizingInt(Dish::getCalories));
stats.getMax();
stats.getMin();
stats.getAverage();
stats.getSum();
stats.getCount();
```

---

### 29. Joining Strings


```java
String results = menu.stream()
                      .filter(d -> d.isVegetarian())
                      .map(d -> d.getName())
                      .collect(Collectors.joining(",", "[", "]"));
```


```java
results = menu.stream().collect(reducing(" ", Dish::getName, (i, j) -> i + j));
```

Labelled in slide:

```
reducing(" ",     Dish::getName,          (i, j) -> i + j)
   ↑                    ↑                        ↑
Initial Value    Identity function/          Binary operator
                  transformation
```

**Note:**

> `joining()` internally makes use of a `StringBuilder` to append the generated strings into one.

`Collectors.joining(delimiter, prefix, suffix)` is the friendly, purpose-built way to concatenate strings from a stream, you'd use it instead of manually reducing, in practice.

---

### 30. Reducing (as a collector)

```java
results = menu.stream()
              .collect(reducing(" ", Dish::getName, (i, j) -> i + j));

Optional<Dish> mostCalorieDish =
    menu.stream()
        .collect(reducing((d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

This second form:`reducing` with just a `BinaryOperator` and no initial value, returns an `Optional<Dish>` (again because the stream might be empty), and picks whichever dish has more calories at each step, ending with the overall highest-calorie dish.

---

### 31. Reduce vs Collect

**Key points:**

- The `reduce` method is meant to combine two values and produce a new one; it's an **immutable** reduction.
- In contrast, the `collect` method is designed to **mutate** a container to accumulate the result it's supposed to produce.
- Using the `reduce` method with the wrong semantic is also the cause of a practical problem: this reduction process can't work in parallel because the concurrent modification of the same data structure operated by multiple threads can corrupt the `List` itself.
- The `collect` method is useful for expressing reduction working on a mutable container but crucially in a parallel-friendly way.

In short: don't try to do something like `stream.reduce(new ArrayList<>(), (list, x) -> { list.add(x); return list; })` — that mutates a shared list unsafely under parallelism. Use `collect(toList())` instead, which is built around the supplier/accumulator/combiner pattern specifically so it's safe for concurrent partial accumulation.

---

### 32. Partitioning

java

```java
Map<Boolean, List<Dish>> mapResults =
    menu.stream().collect(partitioningBy(d -> d.isVegetarian()));
```

`partitioningBy` splits the stream into exactly two buckets keyed by `true`/`false`: here, Vegetarian vs NonVegetarian.

`partitioningBy` always produces a `Map<Boolean, List<T>>` with exactly two keys (`true` and `false`), even if one bucket ends up empty, this is different from `groupingBy` (next section), which only creates keys it actually encounters.

---

### 33. Prime vs Non-Prime example

java

```java
public boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
                     .noneMatch(i -> candidate % i == 0);
}

public Map<Boolean, List<Integer>> partitionPrimes(int n) {
    return IntStream.rangeClosed(2, n)
                     .boxed()
                     .collect(partitioningBy(number -> isPrime(number)));
}
```

New elements introduced here that are worth knowing for your own code:

- `IntStream.rangeClosed(a, b)` produces a primitive `int` stream from `a` to `b` **inclusive**.
- `.boxed()` converts an `IntStream` into a `Stream<Integer>` (needed because `partitioningBy` and most `Collectors` work on object streams, not primitive streams).

---

### 34. Grouping

java

```java
menu.stream().collect(groupingBy(d -> d.getType()));
```

**Key point:**

> We call this Function a _classification_ function because it's used to classify the elements of the stream into different groups.

![[Pasted image 20260904192939.png]]

`menu` → grouped into FISH / MEAT / OTHERS buckets, using `Dish::getType`.

`groupingBy` produces a `Map<K, List<T>>` where `K` is whatever type your classification function returns, and each key maps to a list of all elements that produced that key.

---

### 35. Grouping

![[Pasted image 20260904193223.png]]

The slide's diagram traces one element through the process:

```
Stream: ..., prawns, ...
   |
   v
Classification function (Apply) --> Key: FISH
   |
   v
Classify item into list ------------> add "salmon" to FISH's list

Resulting Grouping map:
  FISH  -> [salmon]
  MEAT  -> [pork, beef, chicken]
  OTHER -> [pizza, rice, french fries]
```

Each incoming element gets run through the classification function to determine which bucket (key) it belongs to, then it's added to that bucket's list.

---

### 36. Grouping with a more complex classification function

**Key point:**

> It isn't always possible to use a method reference as a classification function, because you may wish to classify using something more complex than a simple property accessor.


```java
public enum Category { DIET, NORMAL, FAT }
```

java

```java
menu.stream().collect(groupingBy(d -> {
    if (d.getCalories() <= 400) return Category.DIET;
    else if (d.getCalories() <= 700) return Category.NORMAL;
    else return Category.FAT;
}));
```

Resulting type: `Map<Category, List<Dish>>`

Whereas `d -> d.getType()` was a simple property accessor (could've been `Dish::getType`), here the classification logic is a multi-branch calculation. This is why it needs to be a full lambda body rather than a method reference.

---

### 37. Extracting Group-wise Features

**Key point:**

> Using a collector created with a two-argument version of the `Collectors.groupingBy` factory method. It accepts a second argument of type collector besides the usual classification function.  
> The regular one-argument `groupingBy(f)`, where f is the classification function, is in reality just shorthand for `groupingBy(f, toList())`.

This is a crucial mental model: `groupingBy(classifier)` always secretly means `groupingBy(classifier, toList())`, and you can swap out that 2nd argument (the "downstream collector") for something other than `toList()` to compute something different _per group_.

So, the downstream collector is a function you apply on the 'streams' created under each bucket.

---

### 38. Multilevel Collection (brief intro slide)

**Key point:**

> Second level collector may not always subgroup.

(Restates the three categories again: Reducing and summarizing stream elements to a single value / Grouping elements / Partitioning elements, this is the same list as slide 26, used here to set up multilevel collecting.)

---

### 39. Do You Remember?
```java
long countingDish = menu.stream().collect(Collectors.counting());

Comparator<Dish> dishCaloriesComp = Comparator.comparing(Dish::getCalories);
Optional<Dish> TastyDish = menu.stream().collect(maxBy(dishCaloriesComp));
```

This slide is a deliberate recap before moving to combining `groupingBy` with a downstream collector.

---

### 40. Group-wise Features (using a downstream collector)

java

```java
Map<Dish.Type, Long> typesCount =
    menu.stream().collect(groupingBy(Dish::getType, counting()));
// {MEAT=3, FISH=2, OTHER=4}
```

For the highest-calorie dish per type:

```
{FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[Burger]}
```

**Key points:**

- `groupingBy` works in terms of "buckets."
- The first `groupingBy` creates a bucket for each key. You then collect the elements in each bucket with the downstream collector.
- Each bucket gets associated with the key provided by the classifier function.
- The `groupingBy` operation then uses the downstream collector to collect each bucket and makes a map of the results.

java

```java
Optional<Dish> TastyDish = menu.stream().collect(maxBy(dishCaloriesComp));

menu.stream().collect(groupingBy(x -> x.getType(), maxBy(dishCaloriesComp)));
```

The second line reuses the exact same `maxBy` collector — but instead of applying it to the whole stream once, `groupingBy` applies it **separately within each type-bucket**, giving you the tastiest dish _per type_.

---

### 41. Collecting — grouping + maxBy walked through in detail

java

```java
menu.stream().collect(groupingBy(d -> d.getType(),
    maxBy(Comparator.comparingInt(d -> d.getCalories()))));
```

Result type: `Map<Dish.Type, Optional<Dish>>`  
Result: `{FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[Burger]}`

**Key points (verbatim):**

- The values in this Map are Optionals because this is the resulting type of the collector generated by the `maxBy` factory method.
- If there's no Dish in the menu for a given type, that type won't have an `Optional.empty()` as value; it won't be present at all as a key in the Map.
- The `groupingBy` collector lazily adds a new key in the grouping Map only the first time it finds an element in the stream.

That last point matters: `groupingBy` doesn't pre-create empty buckets for every possible enum value — a key only shows up in the result if at least one element was actually classified into it.

---

### 42. Extracting Group Features — using `mapping` as a downstream collector

java

```java
albums.collect(groupingBy(Album::getMainMusician,
    mapping(Album::getName, toList())));
```

**Key point (verbatim):**

> In the same way that a collector is a recipe for building a final value, a downstream collector is a recipe for building a part of that value, which is then used by the main collector.  
> This method takes two arguments: a function transforming the elements in a stream and a further collector accumulating the objects resulting from this transformation.

Bigger worked example (verbatim):

java

```java
Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType =
    menu.stream().collect(
        groupingBy(Dish::getType,
            mapping(dish -> {
                if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                else return CaloricLevel.FAT;
            }, toSet())));
```

So `mapping(transformFn, downstreamCollector)` lets you transform each element _within_ a group before it gets collected by the group's downstream collector — here, converting each `Dish` into a `CaloricLevel` before collecting the (deduplicated, since `toSet()`) levels for that type.

---

### 43. Collecting and Then Wrapping — `collectingAndThen`

java

```java
Map<Dish.Type, Dish> result4 =
    menu.stream().collect(groupingBy(d -> d.getType(),
        collectingAndThen(
            maxBy(Comparator.comparingInt(d -> d.getCalories())),
            s -> s.get())));
```

**Key points (verbatim):**

- This factory method takes two arguments: the collector to be adapted, and a transformation function, and returns another collector.
- This additional collector acts as a wrapper for the old one and maps the value it returns using the transformation function as the last step of the collect operation.

Signature (verbatim):

java

```java
collectingAndThen(Collector<T,A,R> downstream, Function<R,RR> finisher)
```

The slide's diagram traces this concretely: `groupingBy` splits the stream into FISH/MEAT/OTHER substreams → each substream is independently processed by `collectingAndThen(maxBy(...))` → `maxBy` returns the most-caloric dish wrapped in an `Optional` (e.g. `Optional[pork]`) → the transformation function `Optional::get` unwraps it → `collectingAndThen` returns just the plain `Dish` value extracted from the former `Optional` → these become the values of the grouping map.

Why this matters practically: without `collectingAndThen`, `groupingBy(type, maxBy(...))` would give you `Map<Dish.Type, Optional<Dish>>`. Wrapping with `collectingAndThen(..., Optional::get)` cleans that up to a plain `Map<Dish.Type, Dish>`, since you already know every bucket is non-empty (a bucket only exists if at least one dish was classified into it — see the lazy-key point from slide 41).

---

### 44. Any Type of Collection — `toCollection`

java

```java
Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType =
    menu.stream().collect(
        groupingBy(Dish::getType, mapping(
            dish -> {
                if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                else return CaloricLevel.FAT;
            },
            toCollection(HashSet::new))));
```

Labelled in slide: `toCollection(HashSet::new)` is the **downstream collector** here.

This shows that wherever `toSet()`/`toList()` would go, you can instead use `toCollection(supplier)` to control exactly which concrete collection implementation gets built (here, forcing a `HashSet` specifically, rather than whatever the library would pick by default for `toSet()`).

---

### 45. Extracting Group-wise Features — Multilevel Grouping

**Key points (verbatim):**

- The regular one-argument `groupingBy(f)`, where `f` is the classification function, is in reality just shorthand for `groupingBy(f, toList())`.
- To perform a two-level grouping, you can pass an inner `groupingBy` to the outer `groupingBy`.

The slide's diagram shows a Menu split first by Category (DIET/NORMAL/FAT), and then _within_ each of those, split again by `Type` (OTHER/MEAT/FISH), giving a tree-shaped, two-level classification.

---

### 46. More Groupings — N-level grouping

java

```java
Map<Dish.Type, Map<Boolean, Map<Category, List<Dish>>>> result5 =
    menu.stream().collect(groupingBy(d -> d.getType(),
        groupingBy(d1 -> d1.isVegetarian(),
            groupingBy(dish -> {
                if (dish.getCalories() <= 400) return Category.DIET;
                else if (dish.getCalories() <= 700) return Category.NORMAL;
                else return Category.FAT;
            }))));
```

General signature (verbatim):

java

```java
groupingBy(Function<T, K> classifier, Collector<T,A,D> downstream)
```

**Key point (verbatim):** _How to achieve n-level groupings._

The pattern: each `groupingBy` is the _downstream collector_ of the one before it, so you can nest as many levels as you want. Reading `result5` from the outside in: group by `Type`, and within each type-group, group by `isVegetarian()`, and within each of _those_ sub-groups, group by calorie `Category`. The resulting map's structure mirrors the nesting exactly: `Type → Boolean → Category → List<Dish>`.

---

### Putting it all together — a mental checklist for writing your own stream code

1. **Start:** get a `Stream<T>` — from `.stream()` on a collection, `Stream.of(...)`, or `IntStream.rangeClosed(...)`/`.boxed()` for numeric ranges.
2. **Intermediate operations** (return a new stream, lazy, chainable): `filter(Predicate)`, `map(Function)`, `flatMap(Function)`, `distinct()`, `sorted()`, `limit(n)`, `skip(n)`.
    - Use `filter` when you want an `if` inside a loop.
    - Use `map` when each element becomes exactly one new element.
    - Use `flatMap` when each element becomes zero-or-more new elements that should end up flattened into one stream.
3. **Terminal operation** (ends the pipeline, eager): `collect(...)`, `count()`, `reduce(...)`, `forEach(...)`, `anyMatch/allMatch/noneMatch(...)`, `findAny()/findFirst()`, `min(...)/max(...)`.
    - A stream can only have **one** terminal operation run on it — after that, it's consumed.
4. **`collect` + `Collectors`** is your workhorse for building results:
    - `toList()`, `toSet()`, `toCollection(Supplier)`
    - `joining(delim, prefix, suffix)`
    - `counting()`, `summingInt()`, `averagingInt()`, `summarizingInt()`
    - `maxBy(Comparator)`, `minBy(Comparator)`
    - `groupingBy(classifier)` / `groupingBy(classifier, downstreamCollector)` — nest for multilevel grouping
    - `partitioningBy(predicate)` — always exactly two buckets, `true`/`false`
    - `mapping(function, downstreamCollector)` — transform elements _within_ a group before collecting them
    - `collectingAndThen(collector, finisherFunction)` — post-process a collector's result (e.g. unwrap an `Optional`)
5. **`reduce` vs `collect`:** use `reduce` for pure, immutable combination of values (like summing numbers) that's safe in parallel; use `collect` when the result is being built up in a mutable container (like a `List`/`Map`) — `collect` is the parallel-safe way to do that.
6. **Parallelism:** operations you hand to a stream (accumulators, combiners, comparators, classification functions) must be stateless, non-interfering (don't touch the underlying data source or shared mutable state), and use associative combining logic if you want correct, efficient parallel execution.