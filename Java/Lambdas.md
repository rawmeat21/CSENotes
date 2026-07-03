## Why would we want to pass functions to other functions?

Suppose we have Student objects. We may want to sort the list by their roll, score, etc. Normally, we would do this by maybe writing two separate sort functions like `sortByRoll()` or `sortByScore()`. These functions will have **code duplication**. The only thing that changes between these 2 functions is the sort logic, everything else is the same.

Again suppose we have an e-commerce app, where we may want to filter entries by various parameters like rating, company, etc. Writing a separate filter logic for each is tedious and stupid. We need some kind of method so that this does not happen, which is a filtering function. 


### How to pass functions?

![[Pasted image 20260703145210.png]]

![[Pasted image 20260703145227.png]]

Make an interface which defines an abstract filtering logic function. When the user wants to use `filterHotels()` they would pass in an object of `FilteringCondition`. Simple.

![[Pasted image 20260703145502.png]]

So the steps are:

1. Create an interface which has a function for filtering logic
2. To use custom filtering logic, implement the interface and override the appropriate filtering method.
3. Pass it and you're good to go.

**"Why interfaces though? such a weird choice"** - Well, what if you have multiple filtering functions? Use an abstract class, you have to define all of them, which you won't want to for sure. 

When you write:

```java
interface Predicate<T> {
    boolean test(T t);
}
```

You're writing a rule that says: _"Anyone who claims to be a `Predicate<T>` must provide a method called `test`, taking a `T`, returning a `boolean`."_ That's the entire contract. **A contract is just a promise about _what_ something can do, with zero promise about _how_.**

Any class can now "sign" this contract by using `implements Predicate<Integer>` and providing that method:

```java
class IsEven implements Predicate<Integer> {
    public boolean test(Integer x) { return x % 2 == 0; }
}

class IsPositive implements Predicate<Integer> {
    public boolean test(Integer x) { return x > 0; }
}
```

These two classes have _nothing_ to do with each other. But both satisfy the same contract, both are a `Predicate<Integer>`.

Ok, why not use normal classes? Reasons like:

- **Only one superclass is allowed.**
- **Lambdas only work with functional interfaces.**

Interfaces are the best choice for these things.


![[Pasted image 20260703151047.png]]

We can prevent the need of creating classes each time by using Anonymous Inner classes.

![[Pasted image 20260703151348.png]]

OR even better, use lambda functions, which do the exact same thing, they just make the code look better.

**This however, will not work, if you have > 1 functions**

Such interfaces which have 1 function, are called **Functional Interfaces**.

![[Pasted image 20260703151800.png]]


### Some rules of lambda expressions

1. You don't need to pass in the types of parameters.

![[Pasted image 20260703152235.png]]

2. You can eliminate the () if you have only 1 parameter.
3. If you have 1 statement in your body, you can eliminate the {} 

![[Pasted image 20260703152626.png]]

But you have the remove the `return` and well as the `;` 

![[Pasted image 20260703152940.png]]

This is how you store lambdas.


![[Pasted image 20260703154210.png]]

what does `this` refer to? the inner class? fuck no, it refers to the enclosing class Main.

![[Pasted image 20260703154316.png]]

Variables in lambdas must be final or effectively final. So this won't work lil bro.

Even this wont work:

![[Pasted image 20260703154403.png]]

This is because the lambda captures the value of PRICE (2000) into the body.

![[Pasted image 20260703155011.png]]

This is allowed. So basically you see that the reference used in the body of a lambda has to keep pointing to the same object.



