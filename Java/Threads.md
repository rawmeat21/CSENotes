
![[Pasted image 20260430143901.png]]
![[Pasted image 20260430143921.png]]

### User Threads vs Daemon Threads

If there are no user threads running, the program will terminate. Otherwise, there may be daemon threads running.

![[Pasted image 20260430144335.png]]

## Let's create threads

![[Pasted image 20260430145612.png]]

```java
public class Test {  
    public static void main(String[] args) {  
        World world = new World();  
        world.start();  
        for (; ; ) {  
            System.out.println("Hello");  
        }  
    }  
}

public class World extends Thread {  
    @Override  
    public void run() {  
        for (; ; ) {  
            System.out.println("World");  
        }  
    }  
}
```


```java
public class Test {  
    public static void main(String[] args) {  
        World world = new World();  
        Thread thread = new Thread(world);  
        thread.start();  
        for (; ; ) {  
            System.out.println("Hello");  
        }  
    }  
}

public class World implements Runnable {  
    @Override  
    public void run() {  
        for (; ; ) {  
            System.out.println("World");  
        }  
    }  
}
```



![[Pasted image 20260430150836.png]]
(A small example that shows the main thread/method can finish before its child thread)

if we mark the thread as daemon by doing `th.setDaemon(true)` then it may stop after main is done. Daemon threads live to serve user threads

![[Pasted image 20260430152604.png]]

The Thread class is not abstract, nor an interface. So why not use that only?

Thread implements Runnable interface. So it has a run() method which needs to be implemented as you can see. What is target? see below.

So we create a Thread by either
1. Extending Thread itself after which we rewrite the run() method
2. Implementing Runnable (which is what Thread does anyway) and writing the run() method. Then, we pass our created class object to the Thread constructor, this is **target**. **target** is a reference to a Runnable object. So, if we set an object to target, target is not null and hence it runs the run() method of out class object.

Which method is better?- If you extend Thread, then you **cannot extend any other classes!**
Multiple inheritance doesn't exist in Java. But you can implement multiple classes, right? So, there's no problem there. 

![[Pasted image 20260430154123.png]]
When you implement Runnable

![[Pasted image 20260430154203.png]]
When you extend Thread

## Thread methods

1. **start( ):** Begins the execution of the thread. The Java Virtual Machine (JVM) calls the `run()` method of the thread.
2. **run( )**: The entry point for the thread. When the thread is started, the `run()` method is invoked. If the thread was created using a class that implements `Runnable`, the `run()` method will execute the `run()` method of that `Runnable` object.
3. **sleep(long millis):** Causes the currently executing thread to sleep (temporarily cease execution) for the specified number of milliseconds.
4. **join( ):** ==Waits for this thread to die==. When one thread calls the `join()` method of another thread, it pauses the execution of the current thread until the thread being joined has completed its execution.
5. **setPriority(int newPriority):** Changes the priority of the thread. The priority is a value between `Thread.MIN_PRIORITY` (1) and `Thread.MAX_PRIORITY` (10).
6. **interrupt():** Interrupts the thread. If the thread is blocked in a call to `wait()`, `sleep()`, or `join()`, it will throw an `InterruptedException`.
7. **yield():** `Thread.yield()` is a static method that suggests the current thread temporarily pause its execution to allow other threads of the same or higher priority to execute. It’s important to note that `yield()` is just a hint to the thread scheduler, and the actual behavior may vary depending on the JVM and OS.
8. **Thread.setDaemon(boolean)**: Marks the thread as either a daemon thread or a user thread. When the JVM exits, all daemon threads are terminated.


## Simultaneous increment

```java
class Counter {  
    private int count = 0; // shared resource  
  
    public void increment() {  
        count++;  
    }  
  
    public int getCount() {  
        return count;  
    }  
}  
  
public class MyThread extends Thread {  
    private Counter counter;  
  
    public MyThread(Counter counter) {  
        this.counter = counter;  
    }  
  
    @Override  
    public void run() {  
        for (int i = 0; i < 1000; i++) {  
            counter.increment();  
        }  
    }  
  
    public static void main(String[] args) {  
        Counter counter = new Counter();  
        MyThread t1 = new MyThread(counter);  
        MyThread t2 = new MyThread(counter);  
        t1.start();  
        t2.start();  
        
        // rule: always join all threads after you start them!
        
        try {  
            t1.join();  
            t2.join();  
        }catch (Exception e){ }  
        System.out.println(counter.getCount()); // Expected: 2000, Actual will be random <= 2000  
    }  
}
```

Fix: `synchronised`

```java
class Counter {  
    private int count = 0; // shared resource  
  
    public synchronized void increment() {  
        count++;  
    }  
  
    public int getCount() {  
        return count;  
    }  
}
```

## Synchronization

What happens when 2 threads simultaneously read a value?

![[Pasted image 20260430154451.png]]
![[Pasted image 20260430154611.png]]

An example Stack 
![[Pasted image 20260430155250.png]]

2 Threads have access to one stack object

![[Pasted image 20260430155300.png]]

Example situation: 
thread1 runs, tries to push 100 into empty stack
stackTop++, so it becomes 0, then it goes to sleep (so its taken out of context)

Now thread2 comes, does stackTop-- (=-1 again!) and then returns element at index 0.
thread1 again runs and sets array[-1]=element, and then all hell breaks loose.

	Fix for this?

prevent more than 1 thread from having access to some object at a particular time!

A Thread can acquire a 'lock', when it has the lock to the room (=object/function) ONLY it can do what it wants. Other threads have to wait.

![[Pasted image 20260430160514.png]]

You can see an object being used as a lock. You can basically use any object as a lock!
When a thread has access to a lock, only this thread can make any changes.

You can see push() and pop() are using the same lock object. Hence, these methods are bounded by this lock object. When a thread gains access to this lock object, other Threads CANNOT access any function which is using the SAME lock!!

However if the locks were different, then other threads could work with them parallely!

![[Pasted image 20260430161201.png]]

You can also use the `synchronized` keyword if you want to make the whole function synchronized. 

But then where's the lock? 

![[Pasted image 20260430161317.png]]

The lock used is the instance of the object itself! This also means that ALL `synchronized` methods are usable by only 1 Thread.

Synchronisation for static methods?

![[Pasted image 20260430161646.png]]

It uses ClassName.class (dk what is that)

![[Pasted image 20260430161802.png]]
![[Pasted image 20260430161847.png]]
![[Pasted image 20260430161918.png]]


![[Pasted image 20260430180124.png]]


![[Pasted image 20260430180158.png]]


![[Pasted image 20260430180241.png]]


![[Pasted image 20260430180310.png]]

![[Pasted image 20260430180336.png]]


Read more: https://engineeringdigest.medium.com/multithreading-in-java-39f34724bbf6

## Volatile

![[Pasted image 20260430180911.png]]

![[Pasted image 20260430180949.png]]

The threads read from a cache (Each thread runs on a core, and each core has its own cache)

![[Pasted image 20260430181603.png]]

Th2 makes an update to Flag. This is Not reflected in Thread 1's cache.

![[Pasted image 20260430181700.png]]

Even after Flag is changed in RAM, Th1 still doesn't have the updated value of Flag

![[Pasted image 20260430181808.png]]

With the volatile keyword, Threads read the variable from RAM directly!

## Thread States

![[Pasted image 20260430184932.png]]

![[Pasted image 20260430184953.png]]

![[Pasted image 20260430185156.png]]
yield() tells JVM to put the current running thread back into ready to run state. But there's no guarantee that JVM would actually listen

![[Pasted image 20260430185323.png]]
![[Pasted image 20260430185442.png]]

Difference between sleep and wait: When a thread sleeps, it will NOT relinquish any lock but when it waits it WILL relinquish the lock

**Also, wait only relinquishes the lock on the object it was called from.

![[Pasted image 20260430185829.png]]

![[Pasted image 20260430185925.png]]



![[Pasted image 20260430190120.png]]
![[Pasted image 20260430190143.png]]


Concept of thread join()- When called on a parent thread, it pauses the parent thread until its child threads finish
![[Pasted image 20260430190325.png]]

![[Pasted image 20260430190534.png]]


## Priorities

![[Pasted image 20260430190624.png]]

![[Pasted image 20260430190637.png]]
![[Pasted image 20260430190729.png]]

## Thread Scheduling

![[Pasted image 20260430190906.png]]

## Deadlocks

![[Pasted image 20260430192000.png]]


