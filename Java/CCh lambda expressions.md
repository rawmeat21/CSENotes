### 1. Section Intro: Functional Programming in Java

Nothing to see here, read on.

### 2. Lambda Expressions — Motivation

**From the slides:**

- Lambda expressions are a **"way to represent anonymous functions"**
- They enable **"behaviour parameterization"**

**The motivating example (old Java style):**

```java
button.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent event) {
        System.out.println("button clicked");
    }
});
```

**"The construction is obscure as we want to pass behaviour (an action) but we pass objects instead."** — conceptually you want to hand over _a piece of behavior/logic_, but Java's old syntax forces you to wrap that behavior inside a whole anonymous object just to satisfy the type system. This verbosity is exactly what lambdas fix.

---

### 3. Lambda Expression Syntax

The same example rewritten with a lambda:

```java
button.addActionListener(event -> System.out.println("button clicked"));
```
- **`event`** → **arguments**
- **`System.out.println("button clicked")`** → **Body of the lambda**

Key points:

- Instead of passing an object of an interface a function without a name is passed
- separates the parameter from the body of the lambda expression
- **"javac infers the type of the variable (event)"** — from the **"signature of `addActionListener()`"**
- **"Using null is another type of type inference"**

For reference, the `ActionListener` interface being satisfied:

java

```java
public interface ActionListener extends EventListener {
    public void actionPerformed(ActionEvent event);
}
```

---

### 4. Lambda Syntax Forms

```java
Runnable multiStatement = () -> {
    System.out.print("Hello");
    System.out.println(" World");
};

Runnable noArguments = () -> System.out.println("Hello World");
```

And the general math-style notation shown:

```
(x) → x+1        // Returns x+1
```

---

### 5. Defining a Lambda Expression

Definition worth remembering:

> **"A lambda expression is a concise representation of an anonymous function that can be passed around: it doesn't have a name, but it has a list of parameters, a body, a return type, and also possibly a list of exceptions that can be thrown."**

- **Anonymous** — It doesn't have an explicit name like a method would normally have
- **Function** — A lambda isn't associated with a particular class like a method is. But like a method, a lambda has a list of parameters, a body, a return type, and a possible list of exceptions that can be thrown.
- **Passed around** —A lambda expression can be passed as argument to a method or stored in a variable.
- **Concise** — We don't need to write a lot of boilerplate like we do for anonymous classes.

---

### 6. Lambda Expressions — Features & General Syntax Forms

- It allows functions to be treated as data values
- Do not have a specific name
- Not associated with any class, unlike a Java method
- Can be passed as an argument to a method or stored as a variable (**passed around**)
- Concise syntax, not verbose like inner classes


```java
(parameters) -> expression;
(parameters) -> { statements; }

() -> { return "CR"; }
() -> "CR"
```

---

### 7. Lambda Expressions — Example Use Cases

|Use|Example (verbatim from slide)|
|---|---|
|Create objects|`() -> new Mask(10)`|
|Writing Boolean expressions|`(String s) -> s.length()`|
|Extracting data from an object|`(List<String> list) -> list.isEmpty()`|
|Consuming from an object|`(Tax t) -> {System.out.print(t.collected());}`|
|Combine two values|`(Integer i) -> return "Alan" + i;`|
|Compare two objects|`(Mask m1, Mask m2)-> m1.getLayers().compareTo(m2.getLayers())`|

_(Note: the slide's own list of use-cases and the numbered examples aren't in perfectly matching order — this table pairs them as closely as the slide intends, but the raw numbered list from the slide is: 1) `()→ new Mask(10)`, 2) `(String s) → s.length()`, 3) `(List<String> list) → list.isEmpty()`, 4) `(Mask m1, Mask m2)→ m1.getLayers().compareTo(m2.getLayers())`, 5) `(Integer i) -> return "Alan" + i;`, 6) `(Tax t) -> {System.out.print(t.collected());}`)_

---

### 8. Lambdas, Closures, and Immutability
```java
final String name = getUserName();
button.addActionListener((event) -> System.out.println("hi" + name));

BinaryOperator<Long> addExplicit = (Long x, Long y) -> x+y;
```

- Immutable values
- Anonymous inner classes can only access final (local) variables of their surrounding methods
- Free variables captured by lambda should be effectively final
- This explains closure
- Lambdas close over values rather than variables

---

### 9. Capturing Lambdas

Slide example:

java

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 1554;   // <-- this line causes a compile error!
```

Java lambdas **capture a copy of the value at the time the lambda is created**. Because of that, Java enforces the local variable must be **effectively final**, meaning even if you don't write the `final` keyword, the variable must never be reassigned anywhere after its initial assignment.

**Instance fields are different**, they live on the **heap**, which _is_ shared across threads, so capturing `this.someField` inside a lambda is fine even if it changes later; you just get normal (Java memory model governed) visibility of the current value, not a frozen copy.

**Points to remember:**

- Free variables can be captured
- Lambdas can be passed as argument to methods and can access variables outside their scope
- variables have to be implicitly final
- Allowing capture of mutable local variables opens new thread-unsafe possibilities, which are undesirable
- close over values rather than variables
- Instance variables are fine because they live on the heap, which is shared across threads.

So: local variables captured by a lambda must effectively never be reassigned after being captured (that's why the `portNumber = 1554;` line in the example is a problem), but instance fields don't have this restriction since they live on the heap.

---

### 10. Functional Interfaces


```java
public interface ActionListener extends EventListener {
    public void actionPerformed(ActionEvent event);
}
```

**Definition points from the slide:**

- An interface with a **single abstract method** that is used as the type of the lambda expression
- May use more than 1 parameters
- May return a value
- May use generics
- Signature of the lambda expression should be same as the method of the functional interface
- The type checking for lambda expressions are performed by the compiler
- Ex: **Runnable, Comparator**
---

### 11. Functional Interfaces — Runnable Example

```java
Runnable r1 = () -> System.out.println("Hello World 1");   // Using a lambda

Runnable r2 = new Runnable() {                              // Using an anonymous class
    public void run() {
        System.out.println("Hello World 2");
    }
};

public static void process(Runnable r) {
    r.run();
}

process(() -> System.out.println("Hello World 3"));         // passed directly
```

This illustrates that a lambda, an anonymous class, and a variable holding either can all be passed to a method expecting the functional interface `Runnable`.

---

### 12. Functional Interfaces — Custom Interface Example (BufferedReaderProcessor)

java

```java
public static String processFile() throws IOException {
    try (BufferedReader br =
            new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine();   // This is the line that does useful work.
    }
}

@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
```

Diagram given: `BufferedReader → [BufferedReaderProcessor] → String`

This shows a **custom functional interface** (`BufferedReaderProcessor`) being defined for a specific processing task, annotated with `@FunctionalInterface`.

javac looks at the expected parameter type of `processFile` (`BufferedReaderProcessor`), sees it has one abstract method `process(BufferedReader) -> String`, and treats your lambda as an implementation of _that_ method. `@FunctionalInterface` is an annotation that makes the compiler _enforce_ this — if you accidentally add a second abstract method, compilation fails with an error, catching the mistake early rather than silently breaking lambda compatibility.

---

### 13. Execute Around Pattern

java

```java
public static String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br =
            new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br);   // Processing the BufferedReader object
    }
}

String oneLine =
    processFile((BufferedReader br) -> br.readLine());

String twoLines =
    processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

This demonstrates the **Execute Around pattern**: the "init/preparation code" and "cleanup/finishing code" (opening/closing the `BufferedReader`) stay fixed inside `processFile`, while the varying **"Task"** (what to actually do with the reader) is passed in as a lambda implementing `BufferedReaderProcessor`. That's exactly the "Task A" / "Task B" idea shown in the diagram on the previous slide — same wrapper logic, different plugged-in behavior.

---

### 14. Built-in Java Interfaces

**Table from the slide:**

|Interface name|Arguments|Returns|
|---|---|---|
|`Predicate<T>`|T|boolean|
|`Consumer<T>`|T|void|
|`Function<T,R>`|T|R|
|`Supplier<T>`|None|T|
|`UnaryOperator<T>`|T|T|
|`BinaryOperator<T>`|(T, T)|T|

**`Function<T,R>` example given:**

java

```java
Function<T,R> {
    <R> apply(<T>);
}
```

**`Predicate<T>` example:**

java

```java
Predicate<Integer> atLeast5 = x -> x > 5;

public interface Predicate<T> {
    boolean test(T t);
}
```

Diagram: `T → [Predicate] → boolean`

**More lambda shorthand examples from the slide:**

java

```java
X -> X+1;
X -> X==1;
(X,Y) -> X+1;
(String s) -> s.length();
```

**One more Predicate example:**

java

```java
Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
```

---

### 15. Boxing and Unboxing

**Definitions from the slide:**

- **"Boxing converts- mechanism to convert a primitive type into a corresponding reference type"**
- **"Unboxing converts"** _(the reverse — reference type back to primitive)_
- **"Autoboxing automatically performs boxing and/or unboxing"**
- **"Each element of a primitive array is the size of the primitive"**
- **"Boxed values use more memory"**
- **"require additional memory lookups to fetch the wrapped primitive value"**

Interface mentioned:

java

```java
ToIntFunction<T>
int apply(<T>)
```

_(The array diagrams on this slide illustrate a plain `Employee[]` array allocation and a one-dimensional array with six elements/indexes, shown to contrast how array elements/primitives are laid out in memory.)_

---

### 16. Boxing vs Unboxing — Code Example

Java generics (`Predicate<T>`) can't be parameterized with primitives — `Predicate<int>` isn't legal syntax. So when you use `Predicate<Integer>`, every `int` you pass gets **autoboxed** into an `Integer` object (and unboxed back when read). This costs:

- extra memory (an `Integer` object header + the int value, vs. just 4 bytes for a raw `int`)
- an extra pointer indirection to read the value

`IntPredicate`, `ToIntFunction<T>`, `IntToDoubleFunction`, etc. exist specifically as **primitive-specialized variants** so you can avoid this overhead in performance-sensitive code (e.g., inside Stream pipelines processing millions of elements).


```java
IntPredicate evenNumbers = (int i) -> i % 2 == 0;
evenNumbers.test(1000);

Predicate<Integer> oddNumbers = (Integer i) -> i % 2 == 1;
oddNumbers.test(1000);
```

```java
ToIntFunction<T>
IntToDoubleFunction f = a -> a+1;
```

This contrasts the **primitive-specialized** functional interface `IntPredicate` (which avoids boxing since it works directly with `int`) against the **generic** `Predicate<Integer>` (which requires autoboxing between `int` and `Integer`).

---

### 17. Target Typing

**Table repeated (same as before):**

|Interface name|Arguments|Returns|
|---|---|---|
|`Predicate<T>`|T|boolean|
|`Consumer<T>`|T|void|
|`Function<T,R>`|T|R|
|`Supplier<T>`|None|T|
|`UnaryOperator<T>`|T|T|
|`BinaryOperator<T>`|(T, T)|T|

java

```java
Function<Integer,Boolean> f = a -> a==1;
Predicate<Integer> p1 = a -> a==1;
```

**Key rule from the slide:**

> **"If a lambda has a statement expression as its body, it's compatible with a function descriptor that returns void (provided the parameter list is compatible too)."**

Example:

java

```java
// Predicate has a boolean return
Predicate<String> p = s -> list.add(s);

// Consumer has a void return
Consumer<String> b = s -> list.add(s);
```
This works because Java's rule is: if a lambda's body is a **statement expression** (like a method call whose result you're allowed to just discard), it can satisfy _either_ a functional interface that returns that value, _or_ one that returns `void` — the return value is simply thrown away in the second case. This is why the exact same lambda text compiles against two different functional interfaces depending on what the assignment/parameter context expects (that's "target typing" — the target type comes from context, not from the lambda itself).


```java
boolean test(String s) {
    return list.add(s);
}

void method1(String s) {
    list.add(s);
    return;
}
```

---

### 18. Overloading

java

```java
private void overloadedMethod(Object o) {
    System.out.print("Object");
}

private void overloadedMethod(String s) {
    System.out.print("String");
}
```

**Points from the slide:**

- `OverloadedMethod("abc");`
- **"Javac will refer to the most specific type"** — here, `String` is more specific than `Object`, so the `String` overload is chosen.

```java
overloadedMethod("abc"); // works: String is more specific than Object
```

Here there's no ambiguity: of the two overloads (`Object` and `String`), `String` is a subtype of `Object`, so it's unambiguously the "most specific" applicable match. Java picks it.

java

```java
overloadedMethod((x) -> true); // COMPILE ERROR
```

Here the two candidate overloads take `Predicate<Integer>` and `IntPredicate`. Neither interface is a subtype of the other — they're unrelated types that both happen to be structurally compatible with `(x) -> true`. Since **neither target type is more specific than the other**, javac has no tiebreaker and refuses to guess — you'd have to disambiguate manually with a cast, e.g. `overloadedMethod((IntPredicate)(x -> true))`.

---

### 19. Overloading Resolution

java

```java
overloadedMethod((x) -> true);

private interface IntPredicate {
    public boolean test(int value);
}

private void overloadedMethod(Predicate<Integer> predicate) {
    System.out.print("Predicate");
}

private void overloadedMethod(IntPredicate predicate) {
    System.out.print("IntPredicate");
}
```

**Key point from the slide:**

> **"Javac will fail to compile this as there is no such most specific target type"**

Since `(x) -> true` could match either `Predicate<Integer>` or `IntPredicate` equally well, and neither is "more specific" than the other, the compiler can't disambiguate — this is a compile error, unlike the plain-Object/String overload case above.

---

### 20. Overloading Rules

The slide lists three rules (only the headers are given, without elaboration in the extracted text):

1. **"If there is a single specific target type javac infers …."**
2. **"If there are several specific target types, ….​"**
3. **"If there are several specific target types and no most specific type,…"**

_(These bullet points are cut off in the source PDF itself — that's genuinely all the text present on that slide. Based on the two worked examples right before it: rule 1 corresponds to the `String`-vs-`Object` case (compiler picks the unique most specific type); rule 3 corresponds to the `Predicate<Integer>`-vs-`IntPredicate` case (compile error, no most specific type). Rule 2's continuation isn't present in the slide content provided.)_

---

#### Quick recap of definitions to remember for exams

- **Lambda expression**: _"a concise representation of an anonymous function that can be passed around: it doesn't have a name, but it has a list of parameters, a body, a return type, and also possibly a list of exceptions that can be thrown."_
- **Functional interface**: _"An interface with a single abstract method that is used as the type of the lambda expression."_
- **Closures / capturing**: lambdas _"close over values rather than variables"_; captured local variables must be effectively final.
- **Boxing**: _"mechanism to convert a primitive type into a corresponding reference type"_; unboxing is the reverse; autoboxing does both automatically.
- **Target typing rule**: a void-returning functional interface can accept a lambda with a statement-expression body, if the parameter list matches.