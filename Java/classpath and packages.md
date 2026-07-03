# Java Classpath & Packages

## 1. What the classpath actually is

The **classpath** is the list of directories (and/or `.jar` files) that `javac`/`java` treat as _starting points_ for resolving class and package names. It's conceptually like the OS `PATH` variable, but for `.class` files instead of executables.

- Set with `-cp` (or `-classpath`) on the command line.
- Can list multiple roots, separated by `:` (Linux/Mac) or `;` (Windows).

```bash
javac -cp lib/somelib.jar MyProgram.java
java  -cp .:lib/somelib.jar MyProgram
```

Build tools (Maven, Gradle) and IDEs manage the classpath automatically, but the underlying mechanism is always this simple.

---

## 2. Packages are relative to whatever you pick as the root

> "The root has to be some specific folder like `src`." 
> 
> **Reality:** _Any_ folder can be the classpath root, it's not a property of the folder, it's a choice made at compile/run time via `-cp`.

Example — same files, two different valid interpretations depending on which folder is treated as root:

```
project/
└── src/
    └── com/
        └── example/
            └── Main.java
```

- Root = `src/` → file must declare `package com.example;`
- Root = `project/` → file must declare `package src.com.example;`


**Hard constraint:** whatever root you pick, the `package` statement inside a file _must_ match the *actual folder path* from that root down to the file. Java verifies this and errors if it doesn't match.

---

## 3. Package resolution is a computed lookup, not a search

>Java never scans anything to _discover_ what's there. Resolution is entirely reactive: something in your source code already names the exact class it wants (via `import` or direct reference), and Java **computes** one exact expected file path from that name.

Remember that during compilation, you have to specify the files you're trying to compile. At this time, `java` (or `javac`) will go through your file(s) and convert each into a `.class` file. Also, it notices those `import` statements that you defined. It will use the `classpath` to find these files. How, you ask? It will first convert the import path to directory path (like `import animals.Dog` -> `animals/Dog.class`). Now the thing this, from where does it search this path from? That directory is the classpath(s). So it does something like `<classpath>/animals.Dog` to locate the file. 


With multiple classpath roots (`-cp src:lib`), Java checks the _same computed path_ against each root in order, stopping at the first match. This is the only sense in which something resembling "search across roots" happens — and even that is just trying a short, explicit list of exact candidates, not scanning contents.

---

## 4. Why `javac`/`java` need explicit files or names (no auto-discovery)

### `javac` doesn't discover source files on its own

You must give it explicit file paths. It has no notion of "compile everything in this folder."

**Exception — dependency chasing:** if `Main.java` has `import animals.Dog;` and actually uses `Dog`, compiling just `Main.java` will cause `javac` to automatically also compile `animals/Dog.java`, because it's a _named, required dependency_.

Files with no reachability relationship (no import/reference chain) must each be listed explicitly, or a shell-level trick (like `find src -name "*.java"`) must be used to enumerate them — that enumeration is done by the _shell_, not by `javac`.


|Command|What you give it|What it does|
|---|---|---|
|`javac`|file paths|compiles exactly those files (+ their named dependencies)|
|`java`|a class name|computes expected `.class` path from that name via `-cp`, loads it, runs `main`|

---

## 5. Special case: `java SomeFile.java` 

Running a `.java` file directly with `java` is a distinct convenience feature (JEP 330) with its **own default classpath rule**:

| Command                  | Default root when `-cp` is omitted       |
| ------------------------ | ---------------------------------------- |
| `javac somefile.java`    | current directory                        |
| `java SomeClassName`     | current directory                        |
| `java path/to/File.java` | **the directory containing `File.java`** |

### Experiment that demonstrated this

Consider these 2 files:


`parent/src/animals/Dog.java`
```java
package animals;

public class Dog {
    private String name;
    
    public Dog(String name)
    {
        this.name = name;
    }

    public void say()
    {
        System.out.println("I am a dog");
    }
}

```

`parent/src/main.java`
```java
import animals.Dog;

public class main {
    public static void main(String[] args) {
        System.out.println("hello");

        Dog d = new Dog("bitch");

        d.say();
    }
}
```


Running `java src/main.java` from the `parent` of `src/` still worked, even though `main.java`'s dependency (`Dog`) lived at `src/animals/Dog.java`. This looked like it should failed because the `classpath` would be `parent`. It worked because the launcher used `src/` (the folder containing the named file, `main.java`) as the implicit root, not the terminal's current directory. This is a launcher-specific default, not a general classpath rule.
`javac` would still give you an error here.

---

## 6. What `package` is actually for (it's not about finding files)

> "If `import` + classpath is enough for javac to locate a file, what's the point of `package` at all? Couldn't `import` alone give javac all the info it needs?"

**`package`** = declares the class's **actual, permanent identity**. A class's true name is `package + simple name` (e.g. `animals.Dog`), and this is baked directly into the compiled `.class` file's internal data.

The folder/package match requirement exists to guarantee that **a class's declared identity and its physical location always agree** — this is what makes the entire computed-path lookup system trustworthy in the first place.

### packages give you:

1. **Namespacing** — two unrelated classes can both be called `Dog` (e.g. `animals.Dog` and `hotdogstand.Dog`) without collision, because their real names differ. Without packages, every class in every library ever written would need a globally unique simple name.

2. **Access control** — fields/methods with no modifier (package-private) are only visible to other classes in the _same package_. This is an enforced language rule based on package membership, unrelated to file lookup.

---

## 7. Why javac _enforces_ folder-matches-package (not just style)

> "I'm directly specifying the path to the file, so Java already knows what file I mean — why does it also care about the package name? It literally has no use"


Suppose `animals/Dog.java` declared `package zoo;` instead of `package animals;`. The compiled `Dog.class`'s _internal_ recorded name is `zoo.Dog`, regardless of which folder it physically sits in.

Let's see what problems you could face if folder-package constraint was not enforced:

- `import zoo.Dog;` → computes `zoo/Dog.class` → **doesn't exist** (no such folder) → fails too. The class becomes unreachable by its **own declared name**. 
- 
- If a _separate_, correctly-placed `zoo.Dog` class exists elsewhere on the classpath, you now have **two different classes claiming the same fully-qualified identity**. 

---

## 9. Package naming: How to name packages??


Package naming is always **relative to whatever classpath root you choose**, and you choose that root. You can choose whatever you want, really, as long as you keep things consistent, by that I mean:

- defining the package names with respect to the classpath you choose
- proper import statements.

Also remember to specify the classpath during compilation. During compilation, 

- provide the classpath (or don't if you're inside the classpath already)
- provide the file paths of the java files

---

