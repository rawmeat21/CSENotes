### 1. Static nested classes

java

```java
class Outer {
    static class Nested {
        void greet() { System.out.println("hi"); }
    }
}

Outer.Nested n = new Outer.Nested();
```

- No implicit reference to an enclosing `Outer` instance.
- Can be instantiated without ever creating an `Outer`.
- Behaves essentially like a top-level class that's just namespaced under `Outer` for organizational purposes.
- Can hold static members itself.


### 2. Inner classes (member inner classes)

java

```java
class Outer {
    int x = 10;
    class Inner {
        void show() { System.out.println(x); } // accesses Outer's x directly
    }
}

Outer o = new Outer();
Outer.Inner i = o.new Inner();  // note the syntax — needs an Outer instance
```

- Every `Inner` instance implicitly holds a hidden reference to the `Outer` instance that created it (accessible via `Outer.this` from inside `Inner`).
- Cannot have `static` fields/methods itself (except constant `static final` fields) — because it's tied to an instance, not the class, so there's no single "owner" to hang class-level state off cleanly. This restriction was actually relaxed in Java 16+ for JEP 395's record support, but conceptually it still doesn't make static state, since every `Inner` is tethered to a different `Outer`.
- Cost: each instance carries that hidden reference, which is a real (if small) memory/GC consideration, it also means an `Inner` instance keeps its `Outer` instance alive as long as the `Inner` is reachable, a classic source of memory leaks (e.g., inner classes used as listeners/callbacks outliving the outer object).


### 3. Local classes

A class defined inside a method body.

java

```java
class Outer {
    void process() {
        int localVar = 5; // must be effectively final to be captured
        class Helper {
            void run() { System.out.println(localVar); }
        }
        new Helper().run();
    }
}
```

- Scoped to the method — can't be referenced outside it.
- Can capture local variables from the enclosing scope, but only if they're **effectively final** (never reassigned after initialization). This is because the local class's constructor actually _copies_ the value into a synthetic field at construction time.
- If the local class is in an instance method, it also implicitly captures `Outer.this`, just like a member inner class.


### 4. Anonymous classes

A class with no name, declared and instantiated in one expression

java

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("running");
    }
};
```

- You're simultaneously declaring an unnamed subclass of `Runnable` (well, an implementer) and creating its single instance.
- Same variable-capture rules as local classes (effectively-final only).
- Compiled to `Outer$1.class`, `Outer$2.class`, etc. — numbered rather than named, since there's no identifier to use.
- Cannot have a constructor (there's no name to give it), cannot implement multiple interfaces or extend + implement simultaneously, it either extends one class or implements one interface.
- Since Java 8, lambdas cover a lot of what anonymous classes used to do for single-abstract-method (functional) interfaces, and are generally preferred where applicable.