## Why use `mutable`:

```cpp
class Report {
    std::string cachedSummary;
    bool isCached = false;
public:
    std::string getSummary() const {
        if (!isCached) {
            cachedSummary = computeExpensiveSummary(); // ERROR
            isCached = true;                            // ERROR
        }
        return cachedSummary;
    }
};
```

getSummary() is marked const — meaning "I promise not to modify the object" — but inside it, you're writing to cachedSummary and isCached. The compiler takes that promise literally and blocks any write to any member, no exceptions.

should this function even be const? Yes — from the outside, calling getSummary() twice in a row doesn't change what the Report means or how it behaves. A caller shouldn't have to think "oh, calling this might mutate the object in some observable way." The caching is purely an internal efficiency trick. That's the exact situation mutable exists for.

```cpp
class Report {
    mutable std::string cachedSummary;
    mutable bool isCached = false;
public:
    std::string getSummary() const {
        if (!isCached) {
            cachedSummary = computeExpensiveSummary(); // OK now
            isCached = true;                            // OK now
        }
        return cachedSummary;
    }
};
```

## Why use `friend` class:


The clearest case for friend classes comes up when you have two classes that are conceptually a tight pair.

Example: an iterator for a custom container.
```cpp
class IntList {
private:
    int* data;
    size_t size;

    friend class IntListIterator; // <-- grants access

public:
    IntList(std::initializer_list<int> vals) {
        size = vals.size();
        data = new int[size];
        std::copy(vals.begin(), vals.end(), data);
    }
    ~IntList() { delete[] data; }

    IntListIterator begin() { return IntListIterator(data); }
    IntListIterator end()   { return IntListIterator(data + size); }
};
```

```cpp
class IntListIterator {
    int* ptr;
public:
    IntListIterator(int* p) : ptr(p) {}
    int& operator*() { return *ptr; }
    IntListIterator& operator++() { ++ptr; return *this; }
    bool operator!=(const IntListIterator& other) const { return ptr != other.ptr; }
};
```

```cpp
IntList numbers = {1, 2, 3, 4};
for (int n : numbers) {
    std::cout << n << " ";
}
```


## Static (Early) vs Late binding

What "binding" means: binding is the process of connecting a function call in your source code to the actual piece of code that will run when that call executes. The question is when that connection gets made — while the compiler is compiling your program, or while the program is actually running.

Static binding (a.k.a. early binding) — decided at compile time.

```cpp
class Animal {
public:
    void breathe() { std::cout << "Animal breathing\n"; }
};

class Dog : public Animal {
public:
    void breathe() { std::cout << "Dog breathing\n"; }
};

int main() {
    Dog d;
    d.breathe(); // Static binding
}
```
Here, d.breathe() — the compiler looks at the static (declared) type of d, which is Dog, and hardwires the call to Dog::breathe() directly into the compiled code. No lookup happens when the program runs; the decision was permanently made at compile time. Function overloading and non-virtual member functions all resolve this way.

Dynamic binding (a.k.a. late binding or dynamic dispatch) — decided at runtime.

```cpp
class Animal {
public:
    virtual void breathe() { std::cout << "Animal breathing\n"; }
};

class Dog : public Animal {
public:
    void breathe() override { std::cout << "Dog breathing\n"; }
};

void makeItBreathe(Animal& a) {
    a.breathe(); // Dynamic binding
}

int main() {
    Dog d;
    makeItBreathe(d);
}
```


## How Does Virtual Function Works Internally?

```cpp
class Animal {
public:
    virtual void speak() { std::cout << "Animal sound\n"; }
    virtual void eat()   { std::cout << "Animal eating\n"; }
};

class Dog : public Animal {
public:
    void speak() override { std::cout << "Woof\n"; }
    void eat()   override { std::cout << "Dog eating\n"; }
};
```

The compiler builds a vtable (virtual table) — one per class.

```
Animal's vtable:            Dog's vtable:
[0] -> Animal::speak        [0] -> Dog::speak
[1] -> Animal::eat          [1] -> Dog::eat
```

```cpp
// conceptually, the compiler generates something like:
void (*Animal_vtable[])() = { &Animal::speak, &Animal::eat };
void (*Dog_vtable[])()    = { &Dog::speak,    &Dog::eat    };
```

Every object of a polymorphic class gets a hidden pointer — the vptr.

When a class has any virtual function, the compiler silently adds one extra hidden member to every object of that class: a pointer to the vtable for its actual (dynamic) type. Conceptually, Dog really looks like:


```cpp
class Dog : public Animal {
    void* __vptr;   // hidden, compiler-inserted — not written by you
    // ... any of Dog's own data members
public:
    void speak() override { std::cout << "Woof\n"; }
    void eat()   override { std::cout << "Dog eating\n"; }
};
```

This `__vptr` is set automatically inside the constructor, before your constructor body even runs — the compiler injects code that does `this->__vptr = &Dog_vtable;` whenever a Dog is constructed.

```cpp
Animal* a = new Dog();
a->speak();
```
- `new Dog()` allocates a Dog object and its constructor sets `a->__vptr = &Dog_vtable.`
- `a->speak()` — compiler sees `speak()` is virtual, so it does NOT hardcode a call to Animal::speak. Instead it generates: "follow a's vptr to get the vtable, index into slot for speak, call whatever function pointer is sitting there."

At runtime, that lookup resolves to Dog::speak, because that's what actually got stored in Dog's vtable at compile time.


## Multiple inheritance


```cpp
class A {
public:
    int x = 1;
};

class B {
public:
    int x = 2;
};

class C : public A, public B {
};
```

Multiple inheritance means C contains a full, separate copy of A's subobject and a full, separate copy of B's subobject, laid out side by side:

```
C object in memory:
+------------------+
| A subobject      |
|   x = 1          |
+------------------+
| B subobject      |
|   x = 2          |
+------------------+
```

```cpp
C c;
c.A::x = 5; // sets A's copy
c.B::x = 6; // sets B's copy
std::cout << c.A::x << " " << c.B::x; // 5 6
```

```cpp
class A {
public:
    virtual void foo() { std::cout << "A::foo\n"; }
};

class B {
public:
    virtual void bar() { std::cout << "B::bar\n"; }
};

class C : public A, public B {
public:
    void foo() override { std::cout << "C::foo\n"; }
    void bar() override { std::cout << "C::bar\n"; }
};
```

Now, C has 2 virtual pointers.

```
C object in memory:
+----------------------+
| A subobject:         |
|   __vptr_A  ---------------> C's-view-of-A vtable: [0]->C::foo
+----------------------+
| B subobject:         |
|   __vptr_B  ---------------> C's-view-of-B vtable: [0]->C::bar
+----------------------+
```

```cpp
C c;
A* a = &c;
B* b = &c;

a->foo(); // must resolve to C::foo
b->bar(); // must resolve to C::bar
```

A's virtual pointer AND B's virtual pointer both point to C's virtual table


## Diamond inheritance

```cpp
class Animal {
public:
    int age;
    virtual void identify() { std::cout << "I am an Animal\n"; }
};

class Bird : public Animal {
};

class Mammal : public Animal {
};

class Bat : public Bird, public Mammal {
};
```

This causes Bat object to have 2 instances of Animal Base class.

```
Bat object in memory:
+---------------------------+
| Bird subobject:           |
|   Animal subobject:       |
|     age                   |
+---------------------------+
| Mammal subobject:         |
|   Animal subobject:       |
|     age                   |
+---------------------------+
```

```cpp
Bat b;
b.age = 5; // ERROR: ambiguous — Bird's Animal::age or Mammal's Animal::age?
```

You need to do `b.Bird::age` or `b.Mammal::age` which is weird.


```cpp
Animal* a = &b; // ERROR: ambiguous conversion: which Animal object to point to?
```


### How to fix: 

```cpp
class Animal {
public:
    int age;
    virtual void identify() { std::cout << "I am an Animal\n"; }
};

class Bird : public virtual Animal {
};

class Mammal : public virtual Animal {
};

class Bat : public Bird, public Mammal {
};
```

```
Bat object in memory:
+---------------------------+
| Bird subobject (no Animal directly) |
+---------------------------+
| Mammal subobject (no Animal directly)|
+---------------------------+
| shared Animal subobject:  |
|   age                     |
+---------------------------+
```

How this works?

Each Bird and Mammal object contain a virtual base pointer. This points to a table. This table in turn contains an offset to the Base class object.

```
Bird object (standalone) layout:
+---------------------------+
| vbptr (virtual base ptr)  | ---> [ offset to Animal: +16 ]
+---------------------------+
| (Bird's own members, if any) |
+---------------------------+
... padding/gap ...
+---------------------------+
| Animal subobject          |
|   age                     |
+---------------------------+
```

```
Bat object layout:
+---------------------------+
| Bird subobject:           |
|   vbptr ---------------------> [ offset to shared Animal: +40 ]
+---------------------------+
| Mammal subobject:         |
|   vbptr ---------------------> [ offset to shared Animal: +24 ]
+---------------------------+
| (Bat's own members)       |
+---------------------------+
| shared Animal subobject:  |
|   age                     |
+---------------------------+
```

Now, when we do `bat.age = 5`:

Bat object uses one of its parents to get an offset to Animal object and then we can set the value of x.


## Use of `using` keyword:

- `using namespace` — pull an entire namespace's names into scope.
- Can be used to a single name too (using declaration)
```cpp
using std::cout;
using std::endl;

int main() {
    cout << "hello" << endl; // works
    std::vector<int> v;      // still need std:: here — only cout/endl were pulled in
}
```
- Can be used for aliasing
- Can be used to modify access specifiers

```cpp
class Base {
protected:
    void helper() { std::cout << "helper\n"; }
};

class Derived : private Base {
public:
    using Base::helper; // re-expose helper() as public in Derived
};

Derived d;
d.helper(); // works — normally private inheritance would hide it
```

- Use overlaoding with Base class names:

```cpp
class Base {
public:
    void greet(int x) { std::cout << "Base greet int\n"; }
    void greet(double x) { std::cout << "Base greet double\n"; }
};

class Derived : public Base {
public:
    using Base::greet; // pulls ALL Base::greet overloads into Derived's scope
    void greet(std::string x) { std::cout << "Derived greet string\n"; }
};

Derived d;
d.greet(5);      // now works — resolves to Base::greet(int)
d.greet("hi");   // resolves to Derived::greet(string)
```



