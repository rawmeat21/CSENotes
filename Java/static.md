The `static` keyword in Java marks something as belonging to the **class itself**, rather than to any particular instance of the class.

### Static variables (class variables)

java

```java
class Counter {
    static int count = 0;  // shared across ALL instances
    int id;                // unique per instance

    Counter() {
        count++;
        id = count;
    }
}
```

There's exactly **one copy** of `count` in memory, no matter how many `Counter` objects you create. It lives in the class itself, not in any object. All instances read/write the same variable.

```java
Counter a = new Counter();
Counter b = new Counter();
System.out.println(Counter.count); // 2 — accessed via class name
```

You _can_ access a static field through an instance (`a.count`), but that's discouraged, it's misleading since it makes it look instance-specific when it isn't.

### Static methods

java

```java
class MathUtils {
    static int square(int x) {
        return x * x;
    }
}
```

Called via `MathUtils.square(5)` — no object needed.

**This is why `main` is static: the JVM calls it before any object of your class exists.**

**Key restriction:** a static method can't directly access instance fields or call instance methods, because it has no `this`. It can only touch other static members directly.

### Static blocks

java

```java
class Config {
    static Map<String, String> settings;

    static {
        settings = new HashMap<>();
        settings.put("mode", "prod");
    }
}
```

Runs once, when the class is first loaded by the JVM (not when an object is created). Useful for complex static-field initialization that a one-liner can't express.

### Static nested classes

java

```java
class Outer {
    static class Nested {
        // doesn't hold an implicit reference to an Outer instance
    }
}
```

Contrast with a non-static inner class, which implicitly holds a reference to its enclosing `Outer` instance (`Outer.this`) and can't exist without one. A static nested class is really just a regular class that's namespaced inside `Outer` for organization — you can instantiate it with `new Outer.Nested()` with no `Outer` object required.