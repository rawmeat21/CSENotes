![[Pasted image 20260829145407.png]]

Any Collection can be converted to a Stream.

Stream is a sequence of items got from a collection so that we can perform declarative and functional programming on it (map, filter, reduce).

declarative means "what to do" not "how to do".

![[Pasted image 20260829145743.png]]


Advantages:

![[Pasted image 20260829145811.png]]


How to create streams:

![[Pasted image 20260829150247.png]]

```java
Stream<Integer> intStream = Stream.of(1,2,3);// creates a Stream having 1,2,3
```

```java
Stream<Integer> limit=Stream.iterate(0, n->n+1).limit(100);
```
This is interesting. seed = 0 means initial value. How to generate next values?

Use the function! So first you get 0+1 = 1, then you get 1+1=2, ....

So final stream is 0,1,2,3,4,5,....

How to stop?? Use `.limit(number of values)`.

The function you pass must be a `UnaryOperator<T>`, ie, it's input and output must be of same type. Example: `toupper(String)` takes in a String and returns one too.

```java
Stream.generate(()->(int)Math.random()*100).limit(100);
```
This generates 100 random values.


A sexy example:

![[Pasted image 20260829151512.png]]

Each function explained:

`.stream()` - converts object to stream.
`.filter()` - Used to filter items based on a condition. Takes in a Predicate check function.
`.map()` - Used to perform a function on each value of stream
`.distinct()` - Used to make all items unique
`.sorted()` - Used to sort the stream objects. You can pass in your own comparator function
`.limit(x)` - Take only the first x elements
`.skip(x)` - Don't take the first x elements
`.collect(Collectors.toList())` - Convert Stream back to list

Another example using `.iterate()`:

![[Pasted image 20260829152127.png]]


Using `max()`:

`max()` actually returns an `Optional<T>`, so you need to do a `.get()` at the end.

```java
Integer x = Stream.iterate(0, x->x+1).limit(101).skip(1).map(x->x%(x*8)).peek(System.out.println).max().get();
```

`.peek(function to perform on object)` - Used to do something on each object. Here we print the objects (integers).

`.max(comparator)` - Returns the max. Better, think like this: You sort the stream items using comparator function that you provide and return the last element.


**Count number of distinct**

```java

List<Integer> list = Arrays.asList(...);
Long dist = list.stream().distinct().count();
```

`min()`, `max()`, `collect()`, `count()` are all terminal operations (done at the end).


To achieve parallelism, use:

```java
list.parallelStream();
```

Only use for very large data.


