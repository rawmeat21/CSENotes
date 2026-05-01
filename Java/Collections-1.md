![[Pasted image 20260429231750.png]]
![[Pasted image 20260429231848.png|697]]
![[Pasted image 20260429232238.png]]
![[Pasted image 20260429232312.png]]


**Note that Collection extends Iterable. By Definition of a collection, it HAS to be iterable. Lists, Queues, Sets, all are iterable. Map is not iterable (atleast not in the same way), hence Map is not part of the Collections framework.

![[Pasted image 20260429235502.png]]

Every class that implements Collection has to implement these methods.
There is also isEmpty()
So Lists, Queues, Sets will have all these methods!

Okay, List, Queue, Set -> interfaces

ArrayList, LinkedlIst,Vector,  -> Actual classes that we can create

LinkedList implements both Deque and List

![[Pasted image 20260430000057.png]]
![[Pasted image 20260430000518.png]]

![[Pasted image 20260430001928.png]]

ListIterator is bidirectional

Interfaces are nice as we know a List will always have add(),set(),get() 
We dont have to worry about the implementations

![[Pasted image 20260430015919.png]]
![[Pasted image 20260430020100.png]]

Queue.offer() returns false when max capacity is reached
Queue.add() returns Exception

Queue.poll() is basically a pop() operation. It returns the element that was removed
Queue.peek() gets the top element

## Comparators

![[Pasted image 20260430105841.png]]

## Sets
![[Pasted image 20260430111457.png]]

HashSet uses hashing to store data. Items are not sorted, so comparator is not required. Also you can iterate through it, but you cannot expect an ordering!

LinkedHashSet extends HashSet, it keeps the order in which items were inserted

Note: you can create a hashset for any class object, this is because every class inherits from Object, which has a hashCode() and equals() method. BUT, there is a problem. By default, objects in the set with SAME data will not be regarded as same.You need the original object

To implement your own hashing,you must override these:

	boolean equals(Object o){}
and
	`int hashCode(){}`


 ![[Pasted image 20260430114300.png]]

### NavigableSet (what you want)

![[Pasted image 20260430114402.png]]

and TreeSet is the implementation of the sorted set!
you can pass in a comparator too (the way is same as priority queue) 

## Maps

Unordered map - HashMap<K,V>
Ordered map - TreeMap<K,V>

How to iterate through a map? Use MapEntry<K,V> interface (converts map to set)

![[Pasted image 20260430115342.png]]
![[Pasted image 20260430115435.png]]
![[Pasted image 20260430115524.png]]


Map.Entry Inner class
![[Pasted image 20260430115631.png]]

IMPORTANT- differrence between HashTable and HashMap
![[Pasted image 20260430115913.png]]


![[Pasted image 20260430120129.png]]
