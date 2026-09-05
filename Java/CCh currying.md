# Streams and Currying

## Numeric Streams (Motivation)

- Java 8 introduces three primitive specialized stream interfaces to tackle this issue: `IntStream`, `DoubleStream`, and `LongStream`, to avoid boxing costs.
- These specializations aren't more complexity about streams but instead more complexity caused by boxing: the (efficiency-based) difference between `int` and `Integer` and so on.
- Example:
```java
IntStream calorieStream = menu.stream().mapToInt(d -> d.getCalories());
Stream<Integer> boxedCalorieStream = calorieStream.boxed();
```
- `range()` and `rangeClosed()` are utilized to generate `IntStream`s.

### Explanation

A `Stream<T>` is generic, which means every element has to be an *object*. `int`, `double`, and `long` are primitive types in Java, not objects, so a `Stream<Integer>` cannot literally hold `int` values, every `int` has to be wrapped ("boxed") into an `Integer` object, and every time you need the raw value back, it has to be "unboxed." This wrapping/unwrapping is not free: it allocates objects on the heap and adds CPU overhead. If you are summing a million calorie values, boxing a million `Integer` objects is wasteful.

`IntStream`, `DoubleStream`, and `LongStream` solve this by storing the *primitive* values directly, with no boxing. They are functionally almost identical to `Stream<T>` (same idea of `map`, `filter`, `reduce`, etc.), but their methods are specialized to primitives, e.g. `mapToInt`, `mapToObj`, `sum()`, `average()`, `max()`, `min()`.

Breaking down the example line by line:
```java
IntStream calorieStream = menu.stream().mapToInt(d -> d.getCalories());
```
- `menu.stream()` -> turns the `menu` collection into a normal `Stream<Dish>` (assuming `menu` is a `List<Dish>`).
- `.mapToInt(d -> d.getCalories())` -> this is a special version of `map`. Instead of returning a `Stream<Integer>` (which would box), it returns an `IntStream` directly, because the lambda produces `int` values via `getCalories()`.
- The result, `calorieStream`, is an `IntStream` -> no `Integer` objects are created.

```java
Stream<Integer> boxedCalorieStream = calorieStream.boxed();
```
- `.boxed()` explicitly converts an `IntStream` back into a `Stream<Integer>`. You'd do this if you need to pass the stream to code that expects a generic `Stream<T>` (e.g. a `Collector` that only works on objects, like `Collectors.toList()`, which needs a `List<Integer>`, not a raw `int[]`).

**Rule to remember:** stay in the primitive stream (`IntStream`/`DoubleStream`/`LongStream`) as long as possible; only call `.boxed()` at the point where you actually need object semantics (e.g., building a `List<Integer>`, or passing to an API that wants `Stream<T>`).

---

## Numeric Streams (Creating Numeric Streams)

```java
IntStream oneToHundred = IntStream.rangeClosed(1,100).filter(i->i%2==0)
IntStream oneToNinetyNine = IntStream.range(1,100).filter(i->i%2==0)
```
- Checking a string is a palindrome:
```java
boolean isPalindrome = IntStream.range(0, input.length() / 2)
    .allMatch(i -> input.charAt(i) == input.charAt(input.length() - 1 - i));
```
- Finding a number prime or nonprime using Java streams

### Explanation

**`range()` vs `rangeClosed()`** — this is a key exam point:
- `IntStream.range(a, b)` produces `a, a+1, ..., b-1`, the upper bound `b` is **exclusive**.
- `IntStream.rangeClosed(a, b)` produces `a, a+1, ..., b`, the upper bound `b` is **inclusive**.

So `IntStream.rangeClosed(1, 100)` gives you 1 through 100 (100 numbers), while `IntStream.range(1, 100)` gives you 1 through 99 (99 numbers). That's exactly why the slide's second line is named `oneToNinetyNine`.

**Palindrome check, explained in depth:**
```java
boolean isPalindrome = IntStream.range(0, input.length() / 2)
    .allMatch(i -> input.charAt(i) == input.charAt(input.length() - 1 - i));
```
- `IntStream.range(0, input.length() / 2)` generates indices `0, 1, 2, ...` up to (but not including) the halfway point of the string. You only need to check the first half against the mirrored second half — checking the whole string would be redundant work.
- `allMatch(predicate)` is a **short-circuiting terminal operation**: it returns `true` only if the predicate is `true` for *every* element, and it stops as soon as it finds one element where the predicate is `false` (it does not need to scan the rest).
- The predicate `i -> input.charAt(i) == input.charAt(input.length() - 1 - i)` compares the character at position `i` from the front with the character at position `i` from the back (`length - 1 - i` is the mirrored index).
- If every such pair matches, the string reads the same forwards and backwards → palindrome.

**Prime/nonprime via streams:** Since this is a very standard interview-style pattern, here is how you'd write it, keeping the exact same style as the palindrome example (checking divisibility with `noneMatch`/`allMatch`):

```java
static boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
                     .noneMatch(i -> candidate % i == 0);
}
```

- `noneMatch(predicate)` is the logical opposite of `allMatch`: it returns `true` if the predicate is `false` for every element (i.e., no divisor was found), meaning the number is prime. It is also short-circuiting, it stops as soon as it finds one divisor.

---

## Building Streams

- Static methods:
```java
Stream.of("", "", "", " ");
Stream.empty()
Arrays.stream(1,2,3,4)
str.chars()
```
- From files

### Explanation

These are the standard ways to *construct* a stream from scratch (as opposed to getting one from a collection via `.stream()`).

- **`Stream.of(T... values)`** — a *varargs* static factory method that builds a stream directly from the listed values. `Stream.of("a", "b", "c")` creates a `Stream<String>` containing those three elements, in that order.
- **`Stream.empty()`** — returns a stream with zero elements, typed as `Stream<T>` (Java infers `T` from context, or you specify it explicitly: `Stream.<String>empty()`). This is useful as a "safe default" — e.g., returning `Stream.empty()` from a method instead of `null`, so callers can chain stream operations without null-checking.
- **`Arrays.stream(...)`** — converts an array into a stream. Note: the slide's `Arrays.stream(1,2,3,4)` is not the actual signature — `Arrays.stream` takes an *array* as its argument, not individual varargs. The correct usage is:
```java
int[] numbers = {1, 2, 3, 4};
IntStream s = Arrays.stream(numbers);
```
Because the array is `int[]`, `Arrays.stream` returns an `IntStream` (a primitive specialization), matching the "Numeric Streams" theme from slides 1–2. For an object array `Arrays.stream(new String[]{"a","b"})` you'd get back a `Stream<String>`.
- **`str.chars()`** — a method on `String` (not `Stream`) that returns an `IntStream` of the character codes (as `int`, i.e. Unicode code points) in the string. E.g., `"abc".chars()` gives an `IntStream` of `97, 98, 99`. This is why in the "unique words" example later you sometimes see `.mapToObj(c -> (char) c)` to turn the `int` codes back into `Character` objects for display.
- **"From files"** — building a `Stream<String>` where each element is one line of a text file. This is expanded fully on the next slide.

---

## Streams from Files

```java
long NoOfUniqueWords = 0;
try {
    Stream<String> lines1 = Files.lines(Paths.get("dataFile.txt"), Charset.defaultCharset());
    NoOfUniqueWords = lines1.flatMap(lines2 -> Arrays.stream(lines2.split(" ")))
                             .distinct().count();
    System.out.println("1. Unique words are " + NoOfUniqueWords);
} catch(Exception e) {}
```
(Sample file content shown on the slide is a text file containing repeated occurrences of the word "word" across several lines.)

### Explanation

**`Files.lines(Path path, Charset cs)`** is a static method that opens the file at `path` and returns a `Stream<String>` where each element is exactly one line of the file, decoded using the given `Charset`. It is a *lazy* stream. The file is read incrementally as the stream is consumed, not loaded entirely into memory up front, which is why it's the recommended way to process large text files line by line.

Now the pipeline:
```java
lines1.flatMap(lines2 -> Arrays.stream(lines2.split(" ")))
      .distinct()
      .count();
```

Hopefully you get this one :)

---

## Infinite Streams: `iterate`

```java
Stream.iterate(0, n -> n + 2).limit(10).forEach(System.out::println);
Stream.of(1,2,3,4,5,6,7,8,9,10).?
```
- Fibonacci number:
```java
Stream.iterate(new int[]{0, 1}, arr -> new int[]{arr[1], arr[0] + arr[1]}).limit(20).forEach(t -> System.out.println("(" + t[0] + "," + t[1] + ")"));
```

### Explanation

**`Stream.iterate(T seed, UnaryOperator<T> f)`** creates an **infinite** stream. Its signature (given directly on the slide) is:
```java
static <T> Stream<T> iterate(T seed, UnaryOperator<T> f)
```
- `seed` is the first element.
- `f` is a `UnaryOperator<T>`: a function `T -> T` — applied to the previous element to produce the next one.
- The stream conceptually never ends: `seed, f(seed), f(f(seed)), f(f(f(seed))), ...`

`Stream.iterate(0, n -> n + 2).limit(10).forEach(System.out::println)`:
- Start at `0`.
- Each next element = previous `+ 2`: `0, 2, 4, 6, 8, 10, 12, 14, 16, 18, ...` (infinite).
- **`.limit(10)`** is what makes this usable: it's an intermediate operation that cuts the (conceptually infinite) stream down to only the first 10 elements. Without `.limit(...)` (or another short-circuiting operation) before a terminal operation, calling `forEach` on an infinite stream would run forever.
- `.forEach(System.out::println)`: a terminal operation that consumes each element (printing it). `System.out::println` is a **method reference**, shorthand for the lambda `x -> System.out.println(x)`.
- Output: `0 2 4 6 8 10 12 14 16 18`.

The line `Stream.of(1,2,3,4,5,6,7,8,9,10).?` is left incomplete on the slide (the `?` is literally a placeholder/exercise prompt, not real syntax), it is showing a finite alternative for contrast with the infinite `iterate` example, but no operation is specified.

---

Infinite Streams: `generate`

- It takes a lambda of type `Supplier<T>` to provide new values.
```java
Stream.generate(Math::random)
      .limit(5)
      .forEach(System.out::println);
```
- A supplier that's stateful isn't safe to use in parallel code.
```java
IntSupplier fib = new IntSupplier(){
    private int previous = 0;
    private int current = 1;
    public int getAsInt(){
        int oldPrevious = this.previous;
        int nextValue = this.previous + this.current;
        this.previous = this.current;
        this.current = nextValue;
        return oldPrevious;
    }
};
IntStream.generate(fib).limit(10).forEach(System.out::println);
```

### Explanation

**`Stream.generate(Supplier<T> s)`** is the other way to build an infinite stream. Unlike `iterate`, each new element is produced by calling a `Supplier<T>`, a functional interface with a single method `T get()` that takes **no arguments**. There is no relationship enforced between one generated value and the next (unlike `iterate`, where each value is explicitly derived from the previous one via `f`).

`Stream.generate(Math::random).limit(5).forEach(System.out::println)`:
- `Math::random` is a method reference matching `Supplier<Double>` (its signature `double random()` takes no args and returns a value, matching `T get()`).
- Each call to `Math.random()` produces a new pseudo-random double in `[0, 1)`, independent of prior calls.
- `.limit(5)` again bounds the infinite stream to 5 elements (mandatory here, since `generate` never terminates on its own).

**Why statefulness matters:** `Math::random` happens to be safe because each call is independent and thread-safe internally. But the `IntSupplier fib` example is **stateful**, it has mutable instance fields `previous` and `current` that are updated on every call, and each call's result depends on the *order* of previous calls. If you ran `IntStream.generate(fib).parallel()...`, multiple threads could call `getAsInt()` concurrently, racing on `previous`/`current` and corrupting the sequence (or throwing exceptions due to unsynchronized access). This is exactly the sentence on the slide: *"a supplier that's stateful isn't safe to use in parallel code."*

Walking through the stateful Fibonacci `IntSupplier`:
```java
IntSupplier fib = new IntSupplier(){
    private int previous = 0;
    private int current = 1;
    public int getAsInt(){
        int oldPrevious = this.previous;
        int nextValue = this.previous + this.current;
        this.previous = this.current;
        this.current = nextValue;
        return oldPrevious;
    }
};
```
- This is an **anonymous class** implementing `IntSupplier` (the primitive-specialized version of `Supplier<T>`, whose method is `int getAsInt()` instead of `T get()` — this avoids boxing each Fibonacci number into an `Integer`).
- It keeps two `private` fields as mutable state across calls: `previous` and `current`.
- Each call to `getAsInt()`:
  1. Remembers the *old* `previous` value (this is what gets returned this call).
  2. Computes `nextValue = previous + current`.
  3. Shifts the window forward: `previous` becomes the old `current`, `current` becomes `nextValue`.
  4. Returns the *old* `previous`.
- Trace of first several calls: returns `0, 1, 1, 2, 3, 5, 8, 13, 21, 34` — the Fibonacci sequence — because each call mutates the enclosed state to remember where it left off, unlike `Math::random` which needs no such memory.
- `IntStream.generate(fib).limit(10).forEach(System.out::println)` prints exactly those 10 numbers.

This is the counterpart to the `iterate`-based Fibonacci from Slide 5: `iterate` derives the next state functionally from the *previous* state (passed in as the seed/output of `f`), with no mutable fields, while `generate` here relies on a stateful object that remembers everything internally between calls with no seed/state passed through the stream API itself.

---

## Slide 7 & 8 — Parallel Streams

**[SLIDE]**
```java
public static long iterativeSum(long n) {
    long result = 0;
    for (long i = 1L; i <= n; i++) {
        result += i;
    }
    return result;
}
public static long parallelSum(long n) {
    return LongStream.iterate(1L, i -> i + 1)
                      .limit(n)
                      .reduce(0L, (a,b) -> a+b);
}
```
- (Slide 8 repeats this, then adds:) "Using the right data structure and then making it work in parallel guarantees the best performance."
```java
public static long parallelSum(long n) {
    return LongStream.iterate(1L, i -> i + 1)
                      .limit(n)
                      .parallel()
                      .reduce(0L, (a,b) -> a+b);
}
```

### Explanation

`iterativeSum` is the classic imperative for-loop way to sum `1` through `n`.

The first `parallelSum` (Slide 7) is **not actually running in parallel** — despite the name, it's just a *sequential stream* pipeline written in stream style:
- `LongStream.iterate(1L, i -> i + 1)` — an infinite stream of longs starting at 1, incrementing by 1 each step (`1, 2, 3, 4, ...`).
- `.limit(n)` — cut it down to exactly `n` elements: `1, 2, ..., n`.
- `.reduce(0L, (a,b) -> a+b)` — a terminal **reduction**: start with an identity value `0L`, and combine elements pairwise with the `(a,b) -> a+b` operator, producing a running total. `reduce(identity, accumulator)` folds the whole stream down to a single value — here, the sum `1+2+...+n`.

Slide 8's point ("using the right data structure...guarantees the best performance") sets up the punchline: **actually going parallel requires calling `.parallel()`.**
```java
LongStream.iterate(1L, i -> i + 1)
           .limit(n)
           .parallel()
           .reduce(0L, (a,b) -> a+b);
```
- `.parallel()` is an intermediate operation that marks the stream to be executed using the **Fork/Join framework** (covered on the following slides), splitting the work across multiple threads/cores.
- **Gotcha (this is a very important exam trap, and the slides flag it in the very next section):** `LongStream.iterate(...)` is fundamentally a poor fit for parallelization, because each element depends sequentially on the one before it (`i -> i+1` needs the previous `i`) and `iterate`-based streams don't split (decompose) efficiently. So simply slapping `.parallel()` onto this particular pipeline does **not** actually make it faster — it may even be slower than the sequential version, because of the coordination overhead with no real parallel gain. This is exactly what the next slides explain in more depth (see "Parallelism Wins?").

---

## Slide 9 — Parallelism Wins? (Part 1)

**[SLIDE]**
- Turning a sequential stream into a parallel one is trivial but not always the right thing to do. Performance should be measured first.
- Automatic boxing and unboxing operations can dramatically hurt performance.
- Some operations naturally perform worse on a parallel stream than on a sequential stream.
- In particular, operations such as `limit` and `findFirst` that rely on the order of the elements are expensive in a parallel stream.
- `findAny` will perform better than `findFirst` because it isn't constrained to operate in the encounter order.
- You can always turn an ordered stream into an unordered stream by invoking the method `unordered` on it.

### Explanation

This is a list of caveats about parallel streams — memorize these, they are classic exam bullet points:

1. **"Trivial but not always right" / measure first.** Calling `.parallel()` is a one-line change, so it's tempting to sprinkle it everywhere. But parallel execution has real overhead (splitting the data, coordinating threads, merging partial results), and for small workloads that overhead can dwarf any benefit. Always benchmark before/after with realistic data sizes rather than assuming `.parallel()` helps.

2. **Boxing/unboxing hurts performance.** If your parallel stream is a `Stream<Integer>` rather than an `IntStream`, every element requires boxing/unboxing on top of the parallel overhead — compounding the cost. This connects back to Slide 1: prefer primitive streams for numeric work, *especially* in parallel.

3. **Order-dependent operations are expensive in parallel.** `limit(n)` needs to know exactly which `n` elements come *first in encounter order* — in a parallel setting, different threads process different chunks concurrently, so the framework has to do extra bookkeeping/synchronization to figure out which results are "the first n" once everything comes back together. Similarly, `findFirst()` must specifically return the first element in encounter order, which again requires the same kind of coordination across parallel workers.

4. **`findAny` vs `findFirst`.** `findAny()` returns *some* matching element — it doesn't care which one — so as soon as *any* worker thread finds a match, it can return immediately without needing to coordinate with the others about ordering. This makes `findAny()` cheaper than `findFirst()` in a parallel context whenever you don't specifically need the first-in-order result.

5. **`unordered()`.** If you don't care about encounter order at all (e.g., you're just summing values, or using `findAny`), calling `.unordered()` on the stream tells the framework it's free to ignore ordering constraints, which can remove a lot of the coordination overhead described in point 3 — and can be combined with `.parallel()` for extra speed on order-insensitive operations.

---

## Slide 10 — Parallelism Wins? (Part 2: Cost Model)

**[SLIDE]**
- Consider the total computational cost of the pipeline of operations performed by the stream.
- With `N` being the number of elements to be processed and `Q` the approximate cost of processing one of these elements through the stream pipeline, the product of `N*Q` gives a rough qualitative estimation of this cost.
- A higher value for `Q` implies a better chance of good performance when using a parallel stream.
- For a small amount of data, choosing a parallel stream is almost never a winning decision.

### Explanation

This gives you a mental cost model, worth memorizing as-is for an exam:

- **`N` = number of elements**, **`Q` = per-element processing cost**. The rough total work is `N * Q`.
- Splitting work across threads has a fixed overhead (thread coordination, merging results). This overhead is only "worth it" if the total work `N * Q` is large enough to amortize it.
- **If `Q` is large** (each element takes real, non-trivial computation — e.g., some heavy math operation per element), then parallelizing has more to gain, because you're distributing genuinely expensive work.
- **If `N` is small** (few elements), the fixed overhead of setting up parallel execution dominates regardless of `Q`, so sequential execution nearly always wins. This is the direct justification for the bullet: *"for a small amount of data, choosing a parallel stream is almost never a winning decision."*

Intuition: parallelism amortizes fixed coordination costs over the *total* amount of work; if total work is small, there's nothing to amortize it against.

---

## Slide 11 — Parallelism Wins? (Part 3: Data Structure Decomposition)

**[SLIDE]**
- Take into account how well the data structure underlying the stream decomposes.
- For instance, an `ArrayList` can be split much more efficiently than a `LinkedList`; the primitive streams created with the `range` factory method can be decomposed quickly.
- You can get full control of this decomposition process by implementing your own `Spliterator`.
- Consider whether a terminal operation has a cheap or expensive merge step (for example, the `combiner` method in a `Collector`).

### Explanation

To run a stream in parallel, the underlying data source has to be **split (decomposed)** into chunks that different threads can work on independently.

- **`ArrayList` decomposes well**: it's backed by a contiguous array with random-access indexing, so splitting it into, say, two halves is an O(1) operation — just pick a midpoint index. This is why the slide singles it out as splitting "much more efficiently."
- **`LinkedList` decomposes poorly**: to find the "midpoint" node, you have to walk the list node-by-node (no random access), which is O(n) just to figure out where to split — that cost has to be paid before any actual parallel work even starts.
- **`IntStream.range(...)`-based streams decompose extremely well**, because the elements are just a numeric range with known bounds — splitting `[1, 1000)` into `[1, 500)` and `[500, 1000)` is trivial arithmetic, no traversal needed at all.
- **`Spliterator`** ("splitting iterator") is the interface that actually implements this decomposition logic under the hood — every `Stream` source has one. Its key method, `trySplit()`, attempts to partition the source into two pieces so the Fork/Join framework can hand each piece to a different worker. If you're wrapping a custom data structure and want it to parallelize well, you can implement your own `Spliterator` to control exactly how it splits (rather than relying on a generic, possibly inefficient default).
- **Merge step cost matters too.** Splitting the *input* is only half the story — after parallel workers each produce a partial result, those partial results must be **combined** back into one final result. For a `Collector` (used with `.collect(...)`), this combining logic is the `combiner` method. If merging two partial results is itself expensive (e.g., merging two large hash maps), that cost eats into any speedup gained from parallel processing. A cheap merge (e.g., adding two numbers together, as in `reduce`) preserves more of the parallel speedup; an expensive merge can cancel it out.

---

## Slide 12 — Fork/Join

**[SLIDE]**
- Diagram showing: "Recursively fork a task into smaller subtasks until each subtask is small enough" → "Evaluate all subtasks in parallel" (sequential evaluation on each leaf) → "join" → "Recombine the partial results" (join again up the tree).
- Callout box: "Ideally, all the cores should be busy invoking the worker threads for a task following the fork/join framework."

### Explanation

This is the execution model underlying `.parallel()`. It works as a **divide-and-conquer recursion**:

1. **Fork:** A big task is recursively split ("forked") into two (or more) smaller subtasks. This recursion continues until each subtask is considered small enough to just compute directly/sequentially (this threshold is sometimes called the "sequential cutoff").
2. **Sequential evaluation:** Each smallest-granularity subtask ("leaf" in the diagram) is computed directly, sequentially, by some worker thread.
3. **Join:** Once two sibling subtasks are both finished, their partial results are combined ("joined") together. This joining happens repeatedly, moving back up the recursion tree, until all partial results have been merged into the single, final answer at the root.

The callout — *"ideally, all the cores should be busy"* — states the goal of this whole scheme: by breaking work into many independent pieces, the framework tries to keep every CPU core continuously occupied with useful work throughout the computation, rather than some cores idling while others are still busy. Whether that ideal is actually reached in practice is exactly the subject of the next slide (work stealing).

---

## Slide 13 — Utilize the Cores (Work Stealing)

**[SLIDE]**
- The time taken by each subtask can dramatically vary either due to the use of an inefficient partition strategy or because of unpredictable causes like slow access to the disk or the need to coordinate the execution with external services.
- The fork/join framework works around this problem with a technique called **work stealing**.
- The tasks are more or less evenly divided on all the threads in the `ForkJoinPool`.
- Each of these threads holds a doubly linked queue of the tasks assigned to it, and as soon as it completes a task it pulls another one from the head of the queue and starts executing it.
- One thread might complete all the tasks assigned to it much faster than the others, which means its queue will become empty while the other threads are still pretty busy.
- In this case, instead of becoming idle, the thread randomly chooses a queue of a different thread and "steals" a task, taking it from the **tail** of the queue.
- This process continues until all the tasks are executed, and then all the queues become empty.
- That's why having many smaller tasks, instead of only a few bigger ones, can help in better balancing the workload among the worker threads.

### Explanation

This slide directly addresses the "ideal" stated in the Fork/Join slide: in reality, subtasks don't always take equal time (disk I/O, external service calls, or an uneven split can make some subtasks slower than others). If threads were rigidly assigned a fixed batch of work with no way to help each other, a thread that finished early would sit idle while another thread was still overloaded — wasting available CPU capacity.

**Work stealing solves this:**
- Every worker thread in the `ForkJoinPool` has its **own double-ended queue** of tasks.
- Normal operation: a thread works through **its own queue from the head** — pull a task from the head, execute it, pull the next from the head, etc.
- When a thread's own queue runs empty (it finished its assigned work early) but other threads still have pending tasks, that idle thread doesn't just sit there — it picks another thread's queue **at random** and **"steals" a task from that queue's tail** (the opposite end from where the owning thread is pulling work). Taking from the opposite end minimizes direct contention between the owner (pulling from the head) and the thief (pulling from the tail).
- This repeats continuously until every queue across every thread is empty, meaning all work is done — at that point the whole parallel computation is complete.
- **Why many small tasks help:** if you split into just 2 or 4 big chunks, an imbalance between chunks (one much slower than another) leaves a big idle gap with no smaller task available to "steal" and fill that gap efficiently. With many small tasks, imbalances get smoothed out much more finely — an idle thread can always grab another small unit of work rather than waiting for one giant chunk to free up.

---

## Slide 14 — Currying: Unit Converter

**[SLIDE]**
```java
DoubleUnaryOperator convertCtoF = curriedConverter(9.0/5, 32);
DoubleUnaryOperator convertUSDtoINR = curriedConverter(0.6, 0);
DoubleUnaryOperator convertKmtoMi = curriedConverter(0.6214, 0);

static double converter(double x, double f, double b) {
    return x * f + b;
}
```
(Comments on the slide label `f` as "Factor and baseline" and `x` as "Input value.")
- `DoubleUnaryOperator` defines a method `applyAsDouble`.
```java
double inr = convertUSDtoINR.applyAsDouble(1000);
```

### Explanation

`converter(x, f, b)` is a general-purpose linear conversion function: `result = x*f + b`. Different unit conversions (Celsius→Fahrenheit, USD→INR, km→mi) are all *linear* transformations, differing only in which `f` (factor) and `b` (baseline/offset) you plug in. The idea of currying here is: rather than calling `converter(x, f, b)` with all three arguments every time, you fix `f` and `b` once, producing a brand-new, simpler function that only still needs `x`.

`DoubleUnaryOperator` is a built-in functional interface in `java.util.function` representing a function `double -> double`. Its single abstract method is `applyAsDouble(double operand)`. It's the primitive-specialized counterpart to `UnaryOperator<Double>` (again, avoiding boxing).

`curriedConverter(f, b)` (defined in full on Slide 16, "Composition") returns a `DoubleUnaryOperator` that has `f` and `b` "baked in," leaving only `x` to be supplied later:
```java
static DoubleUnaryOperator curriedConverter(double f, double b) {
    return (double x) -> x * f + b;
}
```
So:
```java
DoubleUnaryOperator convertCtoF = curriedConverter(9.0/5, 32);
```
- This calls `curriedConverter` once, with `f = 9.0/5` and `b = 32`, and gets back a *function object* stored in `convertCtoF`. Nothing about `x` has been decided yet — `x` will be provided later, whenever you actually want to convert a specific temperature.
```java
double fahrenheit = convertCtoF.applyAsDouble(100); // 100°C -> 212.0
```
Likewise:
```java
DoubleUnaryOperator convertUSDtoINR = curriedConverter(0.6, 0);
double inr = convertUSDtoINR.applyAsDouble(1000); // convert 1000 units at factor 0.6
```
Each of `convertCtoF`, `convertUSDtoINR`, `convertKmtoMi` is a distinct, reusable, named function object, each remembering its own `f` and `b`, that you can call repeatedly with different `x` values without ever re-specifying the factor/baseline. This is the practical payoff of currying: build a family of specialized functions from one general one.

---

## Slide 15 — Currying (Formal Definition)

**[SLIDE]**
- Currying is a technique where a function `f` of two arguments (`x` and `y`, say) is seen instead as a function `g` of one argument that returns a function also of one argument.
- The value returned by the latter function is the same as the value of the original function.
- **`f(x,y) = (g(x))(y)`**
- When some but fewer than the full complement of arguments have been passed, we often say the function is **partially applied**.

### Explanation — memorize this definition precisely, it is exam-critical

This is the formal, general definition of currying, independent of any particular language:

- Start with a two-argument function `f(x, y)`.
- **Currying** transforms it into a one-argument function `g(x)` whose *return value* is itself another one-argument function. That returned function then takes `y` and produces the final result.
- Written as an equation: **`f(x, y) = (g(x))(y)`** — meaning: calling `g` with `x` gives you back a function; calling *that* function with `y` gives you the same result as calling the original two-argument `f(x, y)` directly.
- In the unit converter example: `converter(x, f, b)` is the original (here, 3-argument) function. `curriedConverter(f, b)` plays the role of `g` — instead of `g` taking one argument, it takes `f` and `b` together and returns a one-argument function of `x`. So `converter(x, f, b) == curriedConverter(f, b).applyAsDouble(x)`.
- **Partial application** is the more general, related concept: it's when you supply *some but not all* of a function's arguments up front, obtaining a new function that expects only the remaining arguments. Currying is really a specific, systematic way of building partial application one argument at a time (transforming an n-argument function into a chain of n one-argument functions). But loosely, whenever you've filled in a subset of arguments and gotten back a function awaiting the rest, you've partially applied it.

---

## Slide 16 — Partial Applications (Python: closures)

**[SLIDE]**
```python
def quad(a, b, c, x):
    return a*x**2 + b*x + c

def quad_abc(a, b, c):
    def f(x):
        return quad(a, b, c, x)
    return f

xs = range(5)
f = quad_abc(1, 2, 3)
g = quad_abc(2, 0, 1)
```
- A closure is an inner function returned from a surrounding function, with the inner function having references to the surrounding function.

### Explanation

`quad(a, b, c, x)` computes the quadratic `a*x² + b*x + c` for given coefficients `a, b, c` and input `x`.

`quad_abc(a, b, c)` demonstrates partial application in Python via a **closure**:
```python
def quad_abc(a, b, c):
    def f(x):
        return quad(a, b, c, x)
    return f
```
- `quad_abc` takes the three coefficients and defines an *inner function* `f(x)` that calls `quad`, filling in `a, b, c` from the outer function's parameters, and taking only `x` as its own parameter.
- `quad_abc` then `return`s this inner function `f` itself (not the result of calling it) — so `quad_abc(1, 2, 3)` gives you back a *callable function object*, not a number.
- **What makes `f` a closure**, per the slide's exact definition: `f` is "an inner function returned from a surrounding function, with the inner function having references to the surrounding function." Concretely, `f` refers to `a`, `b`, `c` from `quad_abc`'s scope even though, by the time you actually *call* `f(x)`, `quad_abc` itself has already finished executing and returned. Python's closures "capture" those enclosing variables so they remain accessible to the inner function indefinitely.

Usage:
```python
xs = range(5)
f = quad_abc(1, 2, 3)   # f(x) = 1*x^2 + 2*x + 3
g = quad_abc(2, 0, 1)   # g(x) = 2*x^2 + 0*x + 1
```
- `f = quad_abc(1, 2, 3)` — this call runs `quad_abc` once with `a=1, b=2, c=3`, and returns the inner `f` function with those values baked in via closure. `f` is now a reusable one-argument function.
- `g = quad_abc(2, 0, 1)` — a separate call producing a *different* closure `g`, with its own independently captured `a=2, b=0, c=1`. Each call to `quad_abc` creates a fresh, independent closure — `f` and `g` do not interfere with each other even though they were both built from the same `quad_abc` definition.
- You could now do `[f(x) for x in xs]` to evaluate `f` at every value `0,1,2,3,4` — this is exactly the pattern used with `map` on the next slide.

---

## Slide 17 — Partial Applications (`functools.partial`)

**[SLIDE]** (Image content transcribed)
```python
from functools import partial
def quad(a, b, c, x):
    return a*x**2 + b*x + c
xs = range(5)
f = partial(quad, 1, 2, 3)
g = partial(quad, 2, 0, 1)
print(list(map(f, xs)))
print(list(map(g, xs)))

#---maintaining the order of arguments---
quad_a = partial(quad, 1)
quad_ab = partial(quad_a, 2)
quad_abc = partial(quad_ab, 3)
print(list(map(quad_abc, xs)))
```
- Partial objects are callable objects created by `partial()`.
- The `partial()` function from the `functools` module provides a more flexible approach that doesn't require defining closures for specific partial applications.
- Using partial application, the parameters have to be filled in the order as they are defined in the function.
- It is possible though to partially apply a function by setting keyword arguments.

### Explanation

`functools.partial` is a **built-in, generic** way to achieve the same partial application you manually wrote as a closure on the previous slide — no need to hand-write a wrapper function yourself.

```python
f = partial(quad, 1, 2, 3)
```
- `partial(quad, 1, 2, 3)` creates a new callable — a "partial object" — that, when eventually called, will invoke `quad` with `a=1, b=2, c=3` already fixed as the *first* positional arguments, and whatever you pass to `f` becomes the *next* positional argument(s). So `f(x)` calls `quad(1, 2, 3, x)`.
- `list(map(f, xs))` applies `f` to every element of `xs` (0 through 4) and collects the results into a list — same idea as the Java `Stream.map`, just Python's built-in `map`.

**"Parameters filled in order":**
```python
quad_a = partial(quad, 1)
quad_ab = partial(quad_a, 2)
quad_abc = partial(quad_ab, 3)
```
This demonstrates that you can chain `partial` calls to fill in arguments **one at a time, strictly in the order the original function defines them**:
- `quad_a = partial(quad, 1)` fixes only `a=1`; `quad_a` still needs `b, c, x`.
- `quad_ab = partial(quad_a, 2)` — calling `partial` again on `quad_a`, supplying `2` — this fills the *next* still-open parameter, `b=2`; `quad_ab` still needs `c, x`.
- `quad_abc = partial(quad_ab, 3)` fills `c=3`; `quad_abc(x)` now finally calls `quad(1, 2, 3, x)`.
- This chaining is functionally equivalent to full manual currying, one argument at a time, but built entirely from `functools.partial` rather than hand-written closures.

**Keyword arguments exception:** the general rule is that positional partial application must proceed left-to-right in the function's declared parameter order — you can't skip ahead to fix `c` before `a` and `b` using purely positional arguments. However, `partial()` also accepts **keyword arguments**, and those can be supplied in any order/combination since they're matched by name rather than position — e.g. `partial(quad, c=3)` fixes only `c`, regardless of its position in the parameter list, leaving `a, b, x` to be supplied later (with `x` also potentially given as a keyword).

---

## Slide 18 — Currying (Python: PyMonad)

**[SLIDE]**
- Python supports currying using third-party libraries such as PyMonad (version 2.4.0).
- The `pymonad` module includes the `curry` decorator, which can be used to define functions that can be partially applied without explicit use of `functools.partial`.
- The number of arguments to be curried needs to be passed to the `curry` decorator.
- A decorator creates a kind of composite function based on the decorator and the original function being decorated.
- `f = partial(quad, 1, 2, 3) → f=quad(1,2,3)`

### Explanation

Python's standard library has no built-in `curry` keyword — the `partial`-chaining shown on Slide 17 is the "manual" way to curry. **PyMonad** is a third-party package that provides a `curry` **decorator** as a more direct, built-in-feeling way to get true curried functions.

- A **decorator** in Python is syntax (`@decorator_name` placed above a `def`) that wraps a function definition with another function, producing a new, "composite" function — the decorator can intercept calls, transform arguments, wrap the return value, etc., without you having to rewrite the original function's body.
- With PyMonad's `curry`, you specify **how many arguments should be curried** as an argument to the decorator itself (since Python functions don't have fixed arity metadata the way some functional languages do, the library needs to be told explicitly).
- Once decorated, calling the function with fewer than the full number of arguments automatically returns a new partially-applied function awaiting the rest — you get that behavior "for free," without manually writing nested closures (Slide 16) or manually chaining `partial()` calls (Slide 17).
- The last bullet — `f = partial(quad, 1, 2, 3) → f=quad(1,2,3)` — is a compact way of writing "conceptually, `f` now behaves as if it *were* `quad` already called with `1, 2, 3`, just still missing the final argument `x`" — tying this slide's `curry`-decorator idea back to the concrete `partial()` example from the previous slide.

---

## Slide 19 — Composition (Java: `curriedConverter`, and Python `CtoFconverter`)

**[SLIDE]**
- The formula to convert Celsius to Fahrenheit is `CtoF(x) = x*9/5 + 32`.
```java
static double converter(double x, double f, double b) {
    return x * f + b;
}
```
- Here `x` is the quantity you want to convert, `f` is the conversion factor, and `b` is the baseline.
```java
static DoubleUnaryOperator curriedConverter(double f, double b){
    return (double x) -> x * f + b;
}
```
```python
def CtoFconverter(f, b, x):
    return x * f + b
```
```java
DoubleUnaryOperator convertCtoF = curriedConverter(9.0/5, 32);
convertCtoF.applyAsDouble(32);
```

### Explanation

This slide gives the **full, correct definition** of `curriedConverter`, which was used but not yet fully shown back on Slide 14:
```java
static DoubleUnaryOperator curriedConverter(double f, double b){
    return (double x) -> x * f + b;
}
```
- `curriedConverter` is a **method that returns a lambda**. It takes `f` and `b` as ordinary parameters, and its body constructs and returns a new `DoubleUnaryOperator` — the lambda `(double x) -> x * f + b`. This lambda is a **closure** in the Java sense: it captures the *effectively final* values of `f` and `b` from the enclosing method call, and remembers them for as long as the returned function object exists.
- This is the direct Java analogue of the Python closure from Slide 16 (`quad_abc`): a method with some parameters, whose *body* is just "build and return a smaller function that has those parameters baked in."

`convertCtoF.applyAsDouble(32)`:
- `curriedConverter(9.0/5, 32)` builds a `DoubleUnaryOperator` where `f = 9.0/5 = 1.8` and `b = 32` are fixed.
- `.applyAsDouble(32)` then supplies `x = 32`, computing `32 * 1.8 + 32 = 57.6 + 32 = 89.6`.
- (Worth noting for understanding, separate from the slide: this call converts 32°C to Fahrenheit, giving 89.6°F, consistent with the standard formula `CtoF(x) = x*9/5 + 32` stated at the top of the slide.)

**Python equivalent, uncurried form:**
```python
def CtoFconverter(f, b, x):
    return x * f + b
```
This is presented as the direct Python counterpart to the Java `converter(x, f, b)` static method — a plain multi-argument function with no currying applied yet. (Note the Python version orders parameters `f, b, x` rather than `x, f, b` as in the Java version — the slide preserves this exact ordering difference.) To curry *this* the Python way, you'd apply the same `partial()` or closure techniques from Slides 16–18, e.g. `partial(CtoFconverter, 9/5, 32)` would leave only `x` to be supplied.

---

## Slide 20 — Composition (the `compose` function)

**[SLIDE]**
```python
def compose(f, g):
    def fn(x):
        return f(g(x))
    return fn
```
- The composition can be generalized as a reducing operation.
- The `compose()` function accepts a list of functions to be reduced by composing them pair-wise.

### Explanation

**Function composition** means building a new function by chaining two (or more) existing functions together, so that the output of one becomes the input of the next.

```python
def compose(f, g):
    def fn(x):
        return f(g(x))
    return fn
```
- `compose(f, g)` takes two functions `f` and `g` and returns a brand-new function `fn`.
- `fn(x)` is defined as `f(g(x))` — meaning: **first** apply `g` to `x`, **then** apply `f` to *that* result. Note the order: even though `f` is written first (visually, on the left) in `f(g(x))`, `g` is the one that actually runs *first* (mathematical/functional composition convention — this matches how you'd read the mathematical notation `(f ∘ g)(x) = f(g(x))`).
- Just like `curriedConverter`/`quad_abc`, `fn` is a **closure** — it captures `f` and `g` from the enclosing `compose` call and remembers them for later use, every time `fn` itself is called.

**"Generalized as a reducing operation" / "reduced pair-wise":** if you have more than two functions to chain together — say `f1, f2, f3, f4` — you don't need a special 4-function version of `compose`. Instead, you can treat composition the same way you'd treat summing a list of numbers with `reduce`: repeatedly combine two items at a time into one, then combine that result with the next item, and so on, until only one combined result remains. E.g., conceptually: `compose(f1, compose(f2, compose(f3, f4)))`, or equivalently folding the list `[f1, f2, f3, f4]` using `compose` as the combining ("reducing") operator, exactly the same shape as `Stream.reduce` or Python's own `functools.reduce`.

---

## Slide 21 — Composition (Full Worked Example)

**[SLIDE]**
```python
from functools import partial
def compose(f, g):
    def fn(x):
        return f(g(x))
    return fn
def add(x, y):
    return x + y
def mul(x, y):
    return x * y
increment = partial(add, 32)
twice = partial(mul, (9.0/5))
f = compose(twice, increment)
print(f(20))
```

### Explanation

This slide ties together **currying/partial application** (Slide 17) and **composition** (Slide 20) into one concrete pipeline — worth tracing very carefully, since it's a natural exam question ("what does this print?").

- `add(x, y) = x + y`, `mul(x, y) = x * y` — two plain two-argument functions.
- `increment = partial(add, 32)` — fixes the *first* positional argument of `add` to `32`. So `increment(y)` computes `add(32, y) = 32 + y`. (Despite the name "increment," it actually adds 32 to whatever it's given, mirroring the "+32" step of the Celsius-to-Fahrenheit formula.)
- `twice = partial(mul, 9.0/5)` — fixes the first argument of `mul` to `9.0/5 = 1.8`. So `twice(y)` computes `mul(1.8, y) = 1.8 * y`. (Again, despite the name, it multiplies by 1.8, mirroring the "×9/5" step of the same formula.)
- `f = compose(twice, increment)` — recall `compose(f, g)` returns `fn(x) = f(g(x))`, applying **`g` first**. Here `g = increment` and outer-`f` = `twice`. So:
```
f(x) = twice(increment(x))
```
- `f(20)`:
  1. `increment(20) = 32 + 20 = 52`
  2. `twice(52) = 1.8 * 52 = 93.6`
  3. `print(f(20))` outputs **`93.6`**.

Note carefully: this evaluates as `(x + 32) * 1.8`, which is a *different* order of operations than the standard Celsius-to-Fahrenheit formula `x * 1.8 + 32` used earlier in the slides (which multiplies first, then adds). This is intentional/inherent to how `compose` and the specific `partial` calls were set up in this example — it is not a mistake to "correct," but you should notice the order-of-application matters and trace it exactly as written rather than assuming it reproduces the temperature formula. If you wanted the actual Celsius→Fahrenheit order, you would need `compose(increment, twice)` instead, giving `twice` first: `twice(20) = 36`, then `increment(36) = 68` (the correct 20°C → 68°F).

This dual example is a good general lesson: **the order of arguments to `compose` determines the order of application, and it is easy to get backwards — always trace it step by step rather than assuming.**

---

## Slide 22 — Currying in Java (`Function<Double, Function<Double, Double>>`) and `compose`/`andThen`

**[SLIDE]**
```java
// Curried step 1: multiply Celsius by (9/5)
Function<Double, Function<Double, Double>> multiply = c -> factor -> c * factor;
Function<Double, Double> multiplyByNineFifths = multiply.apply(9.0 / 5.0);

// Curried step 2: add 32
Function<Double, Double> addThirtyTwo = f -> f + 32.0;

// Combine using andThen()
Function<Double, Double> celsiusToFaht = multiplyByNineFifths.andThen(addThirtyTwo);

// Test with 25 degrees Celsius
Double fahrenheit = celsiusToFahrenheit.apply(25.0);
System.out.println("25°C in Fahrenheit: " + fahrenheit);
```
- `compose`: the functionality will be added to be executed in the **first** position of the execution flow. (LIFO — Last in, First out)
- `andThen`: the functionality will be added to be executed in the **last** position of the execution flow. (FIFO — First in, First out)

### Explanation

This is genuine, "native" currying in Java (as opposed to the `DoubleUnaryOperator`-based approach from Slides 14/19, which only curried two of three arguments manually). Here it's built directly from the generic `Function<T, R>` interface, nested.

**`Function<Double, Function<Double, Double>> multiply = c -> factor -> c * factor;`**
- Read the type first: this is a function that takes a `Double` (`c`) and **returns another function** — one that itself takes a `Double` (`factor`) and returns a `Double`. This is exactly the formal currying shape from Slide 15: `f(x, y) = (g(x))(y)`, with `g = multiply`.
- The lambda `c -> factor -> c * factor` is a **curried lambda**: `c -> (factor -> c * factor)`. Reading it: given `c`, produce a new lambda `factor -> c * factor` (a closure capturing `c`); when *that* inner lambda is later called with `factor`, it computes `c * factor`.
- `multiply.apply(9.0/5.0)` calls the outer function with `c = 9.0/5.0`, giving back the *inner* function — with `c` now fixed — assigned to `multiplyByNineFifths`, of type `Function<Double, Double>`. Calling `multiplyByNineFifths.apply(x)` computes `x * (9.0/5.0)`.

**`Function<Double, Double> addThirtyTwo = f -> f + 32.0;`**
- A plain one-argument function, `addThirtyTwo(f) = f + 32.0`. (Not curried further since it only ever needed one argument.)

**Combining with `.andThen(...)`:**
```java
Function<Double, Double> celsiusToFaht = multiplyByNineFifths.andThen(addThirtyTwo);
```
- `Function<T,R>` provides two default methods for composition: **`andThen`** and **`compose`**.
- `a.andThen(b)` produces a new function that, when called with `x`, computes `b.apply(a.apply(x))` — i.e., **`a` runs first**, then its result is fed into `b`. Here: first `multiplyByNineFifths` runs (multiply by 9/5), *then* its result is fed into `addThirtyTwo` (add 32) — exactly reproducing the correct order for `CtoF(x) = x*9/5 + 32`.
- `celsiusToFaht.apply(25.0)`:
  1. `multiplyByNineFifths.apply(25.0) = 25.0 * 1.8 = 45.0`
  2. `addThirtyTwo.apply(45.0) = 45.0 + 32.0 = 77.0`
  3. Result: `77.0` (25°C is indeed 77°F).

**`compose` vs `andThen` — memorize this exactly, as given on the slide:**
- **`compose`**: the argument function is executed **first**, *before* the function you called `.compose(...)` on. The slide describes this ordering as **LIFO (Last in, First out)** — think of it as: the function passed *last* (as the argument to `.compose(...)`) is actually the one that gets executed *first*.
  - `a.compose(b)` ≡ `x -> a.apply(b.apply(x))` — `b` runs first, then `a`.
- **`andThen`**: the argument function is executed **last**, *after* the function you called `.andThen(...)` on. The slide describes this as **FIFO (First in, First out)** — the function that was there *first* (the one you called `.andThen` on) executes first, in the same order you wrote them.
  - `a.andThen(b)` ≡ `x -> b.apply(a.apply(x))` — `a` runs first, then `b`.

So `multiplyByNineFifths.andThen(addThirtyTwo)` reads left-to-right in execution order (multiply, then add) — which is why `andThen` was the right choice here to match the natural formula order, whereas using `.compose(...)` with the same two functions in the same argument positions would have executed them in the *reverse* order.

---

## Summary of Definitions to Memorize (collected from all slides)

1. **Primitive streams** (`IntStream`, `DoubleStream`, `LongStream`) exist to avoid **boxing costs**; use `.boxed()` to convert back to `Stream<Integer>`/etc. when needed.
2. **`range(a,b)`** is upper-bound **exclusive**; **`rangeClosed(a,b)`** is upper-bound **inclusive**.
3. **`flatMap`**: maps each element to a stream, then **flattens** all those streams into one single stream (contrast with `map`, which would leave you with a stream-of-streams).
4. **`Stream.iterate(seed, f)`**: `f` is a `UnaryOperator<T>` — each next element derived functionally from the previous one. Infinite unless bounded with `.limit(...)`.
5. **`Stream.generate(supplier)`**: `supplier` is a `Supplier<T>` (no-arg). No inherent relationship between consecutive elements enforced by the API itself — **a stateful `Supplier` is not safe for parallel use.**
6. **`.parallel()`** triggers Fork/Join-based multi-threaded execution — not automatically beneficial; measure first.
7. Order-sensitive ops (`limit`, `findFirst`) are costlier in parallel than order-insensitive ones (`findAny`); `.unordered()` can relax ordering constraints.
8. **Cost model**: total cost ≈ `N * Q`; parallel is more likely to win when `Q` (per-element cost) is large; **small `N` almost never benefits from parallel.**
9. **Decomposability** matters: `ArrayList` splits well (random access), `LinkedList` splits poorly (must traverse); `range()`-based streams split trivially; custom decomposition = implement your own `Spliterator`; merge/`combiner` cost also affects overall parallel benefit.
10. **Fork/Join**: recursively fork → sequential evaluation of leaves → join/recombine up the tree.
11. **Work stealing**: idle threads steal tasks from the **tail** of another thread's deque (while the owner works from the **head**); this balances load caused by uneven subtask durations; many small tasks balance better than few large ones.
12. **Currying, formal definition**: a 2-argument `f(x,y)` becomes a 1-argument function `g(x)` returning another 1-argument function; **`f(x,y) = (g(x))(y)`**.
13. **Partial application**: supplying fewer than all arguments, obtaining a function awaiting the rest.
14. **Closure**: an inner function returned from an outer function, retaining references to the outer function's variables.
15. **`functools.partial`**: fills parameters strictly in declared order for positional args; keyword args can be supplied in any order.
16. **`compose(f,g)`** (Python, hand-written): `fn(x) = f(g(x))` — `g` runs first.
17. Composition generalizes to a **reduce/fold** over a list of functions, combined pairwise.
18. **Java `Function.compose`**: argument function runs **first** (**LIFO**). **Java `Function.andThen`**: argument function runs **last** (**FIFO**).

---

## Extra Practice (beyond the slides) — for building fluency

**[ADDITIONAL — not in slides]**

1. Write an `IntStream` pipeline that sums the squares of the even numbers from 1 to 20:
```java
int sumOfSquares = IntStream.rangeClosed(1, 20)
                             .filter(i -> i % 2 == 0)
                             .map(i -> i * i)
                             .sum();
```
2. Curry a 3-argument Java method fully (all three arguments, one at a time), the way Slide 22 curried `multiply`:
```java
Function<Integer, Function<Integer, Function<Integer, Integer>>> addThree =
    a -> b -> c -> a + b + c;

int result = addThree.apply(1).apply(2).apply(3); // 6
```
3. Python: curry a 3-argument function manually with nested closures (matching Slide 16's style) instead of `functools.partial`:
```python
def add3(a):
    def inner1(b):
        def inner2(c):
            return a + b + c
        return inner2
    return inner1

result = add3(1)(2)(3)  # 6
```