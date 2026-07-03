
Source : https://engineeringdigest.medium.com/generics-b158a743d18f

```java
import java.util.ArrayList;  
  
public class Main {  
    public static void main(String[] args) {  
        ArrayList list = new ArrayList(); // note: no type given, this is an ArrayList of Object type
        list.add("Hello");  
        list.add(123);  
        list.add(3.14);  
  
        String str = (String) list.get(0);  
        String str1 = (String) list.get(1);  
  
    }  
}
```

Above code has 3 major issues

1. No Type safety - Multiple types in single list
2. Manual casting - We must manually convert each item to its correct type
3. No Compile Time checking - What if we convert a string to what was actually an integer? We will get a runtime error, but there will be no compile time errors

```java
import java.util.ArrayList;  
  
public class Main {  
    public static void main(String[] args) {  
        ArrayList<String> list = new ArrayList<>();  
        list.add("Hello");  
        list.add("World");  
        String s = list.get(0);  
        String s1 = list.get(1);  
  
    }  
}
```
Now all these problems are fixed.


```java
public class Box {  
    private Object value;  
  
    public Object getValue() {  
        return value;  
    }  
  
    public void setValue(Object value) {  
        this.value = value;  
    }  
}
```

```java
public class Main {  
    public static void main(String[] args) {  
        Box box = new Box();  
        box.setValue(1); // Object is an Integer
        String i = (String) box.getValue(); // This won't work because of the same reason as before, Integer object won't convert to String
        System.out.println(i);  
    }  
}
```

Generic types allow you to define a class, interface, or method with type parameters for the data types they will work with. This enables code reusability and type safety, as it allows you to create classes, interfaces, or methods that can operate on various types without needing to rewrite the code for each type.

```java
class ClassName<T> extends ... {  
    // Class body  
}
```

```java
public class Box<T> { //  one or more type parameters  
//  These type parameters are placeholders that are replaced with specific types when the class is instantiated.  
    private T value;  
  
    public T getValue() {  
        return value;  
    }  
  
    public void setValue(T value) {  
        this.value = value;  
    }  
}  
  
public class Main {  
    public static void main(String[] args) {  
        Box<Integer> box = new Box<>();  // Box is now type-safe  
        box.setValue(1);  // No issue, it's an Integer  
        Integer i = box.getValue();  // No casting needed  
        System.out.println(i);  
    }  
}
```


_A generic class can have more than one type parameter._

```java
class Pair<K, V> {  
    private K key;  
    private V value;  
  
    public Pair(K key, V value) {  
        this.key = key;  
        this.value = value;  
    }  
  
    public K getKey() {  
        return key;  
    }  
  
    public V getValue() {  
        return value;  
    }  
}

public class Main {  
    public static void main(String[] args) {  
        Pair<String, Integer> pair = new Pair<>("Age", 30);  
        System.out.println("Key: " + pair.getKey());   // Prints: Key: Age  
        System.out.println("Value: " + pair.getValue()); // Prints: Value: 30  
    }  
}
```

**Type Parameter Naming Conventions**

- `T`: Type
- `E`: Element (used in collections)
- `K`: Key (used in maps)
- `V`: Value (used in maps)
- `N`: Number


## Generic Interface

A **generic interface** in Java allows you to define an interface with type parameters. This means that the interface can work with any type specified at the **time of implementation**.\

Generic interfaces are commonly used when the type of the objects that the interface deals with is not known until runtime.


```java
interface Container<T> {  
    void add(T item);  
    T get();  
}
```


**Implementing with a specific type**

```java
// implementing Container<String>

class StringContainer implements Container<String> {  
    private String item;  
  
    @Override  
    public void add(String item) {  
        this.item = item;  
    }  
  
    @Override  
    public String get() {  
        return item;  
    }  
}
```


**Implementing a generic interface generically**

```java
class GenericContainer<T> implements Container<T> {  
    private T item;  
  
    @Override  
    public void add(T item) {  
        this.item = item;  
    }  
  
    @Override  
    public T get() {  
        return item;  
    }  
}
```


**Generic Interfaces with Multiple Type Parameters**

```java
interface Pair<K, V> {  
    K getKey();  
    V getValue();  
}
```

```java
class KeyValuePair<K, V> implements Pair<K, V> {  
    private K key;  
    private V value;  
  
    public KeyValuePair(K key, V value) {  
        this.key = key;  
        this.value = value;  
    }  
  
    @Override  
    public K getKey() {  
        return key;  
    }  
  
    @Override  
    public V getValue() {  
        return value;  
    }  
}
```

You can create an instance of `KeyValuePair` like this:

```java
Pair<String, Integer> pair = new KeyValuePair<>("Age", 30);  
System.out.println(pair.getKey() + ": " + pair.getValue());
```


**Bounded Types parameters**

```java
interface NumberContainer<T extends Number> {  
    void add(T item);  
    T get();  
}
```

In this example, the type parameter `T` is restricted to subclasses of `Number`, so only numeric types like `Integer`, `Double`, etc., can be used.

```java
class IntegerContainer implements NumberContainer<Integer> {  
    private Integer item;  
  
    @Override  
    public void add(Integer item) {  
        this.item = item;  
    }  
  
    @Override  
    public Integer get() {  
        return item;  
    }  
}
```

![[Pasted image 20260703131311.png]]

![[Pasted image 20260703131336.png]]

How to add restriction where a class extends one class and implements some other classes?

```java

class Box<T extends Number & Printable>
{
	// Body	
}
```

This means T must extend Number and implement Printable (yes we use `&` here, you can chain for multiple interfaces using `&` too)

What about a single interface?

```java

class Box<T implements Printable>
{
	// Body	
}
```

## Generic Constructors

A generic constructor can be defined in a generic class. However, the generic type parameter for the constructor may be different from the generic type parameter of the class:

![[Pasted image 20260703132404.png]]

simply add `<T>` before class name

```java
class Test<T> {  
    private T value;  
  
    // Generic constructor  
    <U> Test(U input) {  
        System.out.println(input.getClass().getName());  
    }  
}  
public class Main {  
    public static void main(String[] args) {  
        Test<Integer> test = new Test<>(12.34);  // Output: java.lang.Double  
    }  
}
```

**Multiple Type Parameters in Constructors**

```java
class Pair {  
    // Generic constructor with two type parameters  
    <A, B> Pair(A first, B second) {  
        System.out.println("First: " + first + ", Second: " + second);  
    }  
}  
  
public class Main {  
    public static void main(String[] args) {  
        new Pair(10, "Ten");  // Integer and String  
        new Pair(3.14, 42);   // Double and Integer  
    }  
}
```

**Bounded Type Parameters in Generic Constructors**

You can also apply bounds to the type parameters in generic constructors, just like in generic classes or methods.

```java
class NumberPrinter {  
    // Bounded type parameter for generic constructor  
    <T extends Number> NumberPrinter(T number) {  
        System.out.println("Number: " + number);  
    }  
}  
```
  
```java
public class Main {  
    public static void main(String[] args) {  
        new NumberPrinter(100);  // Integer is a subclass of Number  
        new NumberPrinter(3.14);  // Double is a subclass of Number  
          
        // The following would cause a compile-time error because String is not a subclass of Number  
        // new NumberPrinter("Hello");    
    }  
}
```

## Generic Static Members

One restriction with generic classes is that **static members cannot use type parameters**. The reason for this is that static members belong to the class itself rather than to any instance, and the type parameter is tied to an instance.

```java
class MyClass<T> {  
    private T instanceVar;  // Valid  
    static T staticVar;     // Invalid - static members cannot use T  
}
```

You can still create a static method using its own type parameter:

```java
class MyClass<T> {  
    public static <U> void staticMethod(U param) {  
        System.out.println(param);  
    }  
}
```

In this example, the static method `staticMethod` has its own type parameter `U`, which is different from the class’s type parameter `T`.


## Generic Methods

By using generic methods, you can write functions that work with any type of data.

```java
public <T> void methodName(T parameter) {  
    // method body  
}
```

```java
public class GenericMethodExample {  
    // Generic method  
    public <T> void printArray(T[] array) {  
        for (T element : array) {  
            System.out.print(element + " ");  
        }  
        System.out.println();  
    }  
  
    public static void main(String[] args) {  
        GenericMethodExample example = new GenericMethodExample();  
          
        Integer[] intArray = {1, 2, 3, 4, 5};  
        String[] stringArray = {"A", "B", "C", "D"};  
          
        // Using the generic method  
        example.printArray(intArray);   // Output: 1 2 3 4 5  
        example.printArray(stringArray); // Output: A B C D  
    }  
}
```

```java
public class GenericMethodExample {  
    public <T, U> void printTwoItems(T item1, U item2) {  
        System.out.println(item1 + " and " + item2);  
    }  
  
    public static void main(String[] args) {  
        GenericMethodExample example = new GenericMethodExample();  
          
        example.printTwoItems(10, "Apples"); // Output: 10 and Apples  
        example.printTwoItems("Hello", 3.14); // Output: Hello and 3.14  
    }  
}
```

```java
public class GenericMethodExample {  
    // Generic static method  
    public static <T> void printElement(T element) {  
        System.out.println("Element: " + element);  
    }  
  
    public static void main(String[] args) {  
        GenericMethodExample.printElement(42); // Output: Element: 42  
        GenericMethodExample.printElement("Generics in Java"); // Output: Element: Generics in Java  
    }  
}
```

**Generic Methods in Enums**

```java
enum Operation {  
    ADD, SUBTRACT, MULTIPLY, DIVIDE;  
  
    public <T extends Number> double apply(T a, T b) {  
        switch (this) {  
            case ADD:  
                return a.doubleValue() + b.doubleValue();  
            case SUBTRACT:  
                return a.doubleValue() - b.doubleValue();  
            case MULTIPLY:  
                return a.doubleValue() * b.doubleValue();  
            case DIVIDE:  
                return a.doubleValue() / b.doubleValue();  
            default:  
                throw new AssertionError("Unknown operation: " + this);  
        }  
    }  
}  
  
public class Main {  
    public static void main(String[] args) {  
	    // notice that we call on the instance
        double result1 = Operation.ADD.apply(10, 20);  
        double result2 = Operation.MULTIPLY.apply(5.5, 4);  
        System.out.println(result1);  // Output: 30.0  
        System.out.println(result2);  // Output: 22.0  
    }  
}
```

## Wildcards

![[Pasted image 20260703135034.png]]

Notice that we're just getting the list here and printing things from it. We don't really care about the type T.

![[Pasted image 20260703135232.png]]

So we can actually replace the type `T` by `?`


![[Pasted image 20260703135355.png]]

What if we do this? 

Its okay but as we return Object, we run into those 3 problems again.

![[Pasted image 20260703135510.png]]

Here its actually better to use `T`.


![[Pasted image 20260703135628.png]]

We cannot use `?` here. Why? because `?` means any type is allowed, we could pass in an `ArrayList<Integer>` as source and `ArrayList<String>` as destination. But that wouldn't work, we can't push items from source to destination.

![[Pasted image 20260703135854.png]]

This is absolutely doable. BUT! you cannot add stuff. Even if you try adding a `String`, you will get error. That's because `?` cannot verify what the type of the object you're adding is.

So, **only use `?` for read only purposes.**


**Bounds to `<?>`**

![[Pasted image 20260703140254.png]]

Pretty much same as before. This is called upper bound.

![[Pasted image 20260703140412.png]]

This is lower bound. `<?>` must be a super class of Integer.


![[Pasted image 20260703140610.png]]

This doesn't work

![[Pasted image 20260703140651.png]]

But this does.


**`List<? extends Number>`**

This means "a list of some specific type that extends Number — but I don't know which one." It could be `List<Integer>`, `List<Double>`, `List<Number>`, anything. The compiler has to be conservative: since it doesn't know the exact type, it can't let you insert _anything_, because whatever you insert might not match the real underlying type. If the actual list is `List<Double>` and you call `.add(20)` (an `Integer`), that would corrupt it. So the compiler blocks all `add()` calls (except `add(null)`, since `null` is trivially compatible with everything). You can only _read_ from it, as a `Number`.

**`List<? super Integer>`**

This means "a list of some specific type that is Integer or a supertype of Integer" — so it could be `List<Integer>`, `List<Number>`, or `List<Object>`. Whatever it actually is, you know for certain that an `Integer` (or any subtype of it) can be safely inserted, since every one of those possible list types is guaranteed to accept an `Integer`. So `add(12)` is legal. What you _can't_ safely do is read elements out as anything more specific than `Object`, since you don't know if it's really a `List<Number>` or `List<Object>` etc.



## Type erasure (How generics work)

Generics in Java are implemented through a process called **type erasure**. 

This means that the generic types are replaced with their bounds or `Object` during compilation, and the resulting **bytecode** contains only ordinary classes, methods, and fields.

In your code:
```java
public class Box<T> {  
    private T item;  
  
    public void setItem(T item) {  
        this.item = item;  
    }  
  
    public T getItem() {  
        return item;  
    }  
}  
  
public class Main {  
    public static void main(String[] args) {  
        Box<String> stringBox = new Box<>();  
        stringBox.setItem("Hello, Generics!");  
        String item = stringBox.getItem();  
        System.out.println(item);  
    }  
}
```

After compilation:

```java
public class Box {  
    private Object item;  // `T` is replaced with `Object`  
  
    public void setItem(Object item) {  // `T` replaced with `Object`  
        this.item = item;  
    }  
  
    public Object getItem() {  // `T` replaced with `Object`  
        return item;  
    }  
}
```

**generics don't exist in bytecode**

For example, when you write:

```java
Box<String> stringBox = new Box<>();  
stringBox.setItem("Hello, Generics!");  
String item = stringBox.getItem();
```

The following happens at runtime:

1. **Type-Safe Operations:** At compile-time, the type safety of `Box<String>` is ensured. The compiler makes sure that only `String` objects are added to the `Box`.
2. **Type Erasure:** At runtime, the JVM treats the `stringBox` as a `Box<Object>`. When you call `getItem()`, the JVM returns an `Object` (not a `String`), but the compiler-generated code has inserted an implicit cast to `String` during compilation:

```java
String item = (String) stringBox.getItem();  // Implicit cast inserted by the compiler
```

Even though type erasure replaces `T` with `Object`, the cast back to `String` ensures that the original type is respected.


## Generic exceptions

![[Pasted image 20260703141934.png]]

![[Pasted image 20260703142109.png]]


**The reason is type erasure interacting with `catch`.**

```java
class MyException<T> extends Exception { }  // compile error
```

At runtime, all generic type info is erased. `MyException<String>` and `MyException<Integer>` both just become `MyException` in the bytecode. That's normally fine for most generic classes, but exceptions are special because the JVM needs to do runtime type checks at `catch` blocks to decide which handler to invoke.

Imagine this were legal:

java

```java
try {
    ...
} catch (MyException<String> e) {
    // handle String case
} catch (MyException<Integer> e) {
    // handle Integer case
}
```

After erasure, both `catch` clauses become `catch (MyException e)` — completely indistinguishable at runtime. The JVM would have no way to tell which one should catch a given thrown exception, since the type parameter that would disambiguate them doesn't exist anymore at runtime. 

![[Pasted image 20260703142623.png]]

You can create a generic constructor tho, that will work.

![[Pasted image 20260703142819.png]]






