**Java Basics**

- What is bytecode in Java? _(2020 CT)_
- Java is platform independent — why/explain. _(2021 CT-I, 2021 CT-II, 2022 Exam)_
- What happens for `int x;` and `Sample y;` where Sample is a class? _(2020 CT)_
- What is a wrapper class in Java? _(2020 CT)_
- Comment on passing a primitive data type vs an object to a function. _(2020 CT)_
- We need to pass values of an int and a float to a method, and the method may change those values, which are required after return. How can this be achieved? _(2021 CT-I, 2021 CT-II)_
- Distinguish between unboxing and autoboxing. Why are these needed in Java? _(2024 Exam)_
- Turing executes the following and the answer is zero irrespective of any non-zero values — name the error and fix it: `void triangle(double b, double h) { double a; a = 1/2 * b * h; System.out.println("Area="+a); }` _(2024 Exam)_
- What you can interpret from an `X$Y.class` file? _(2021 CT-I, 2021 CT-II)_
- What is the utility of the inner class in Java? _(2021 Exam)_
- In a 2D array of integers, number of columns may vary across rows — write code to implement the array and show the number of columns in each row. _(2022 Exam)_

---

**OOP**

- Discuss access specifiers for a class in Java. _(2020 CT, 2021 CT-I, 2021 CT-II, 2022 Exam)_
- Discuss the role of access specifiers for members of a class. _(2022 Exam)_
- How can shallow and deep copy of an object be made in Java? _(2020 CT, 2021 CT-I, 2021 CT-II)_
- What is the use of the `final` keyword in Java? _(2020 CT)_
- What is function overriding in Java and its use? _(2020 CT)_
- What is the advantage of using `@Override`? _(2021 CT-I, 2021 CT-II)_
- Compare abstract class and interface in the context of ensuring design guidelines. _(2021 CT-I, 2021 CT-II)_
- Compare abstract class and interface. _(2022 Exam)_
- Compare function overloading and function overriding in Java. _(2022 Exam)_
- In Java, class X has a public method `f(int)`. Class Y extends X and has a public method `f(float)`. Given: `int i; float fl; Y c = new Y(); c.f(i); X b = c; b.f(fl);` — explain the calls to `f()`. _(2021 CT-II, 2021 Exam)_
- How does the task of a destructor get accomplished in Java? _(2021 CT-II)_
- How will you copy content of one object to another in Java? _(2021 CT-II)_
- Credential of an applicant consists of academic and skill credential. Academic credential has degree, year, institute, score. Skill credential has programming languages and tools known. Certain operations exist to access/set values. Academic and skill credentials exist only as part of credential. Design the necessary classes. _(2022 Exam)_
- Anybody designing a course of a curriculum must follow specifications like predefined max/min contact hours. One must provide content, lecture plan, and textbooks. What measures will you enforce all these? _(2020 CT, 2021 CT-II)_
- Dish is a data type with name, type (veg/fish/meat), and calorie consumption. Represent it in Java. State what OOP constructs are utilized. _(2023 CT Set II)_
- Write a Java program to build a singly linked list of CoffeeCan objects where CoffeeCan has no existence outside the list. Which OOP component is utilized? _(2023 CT Set I)_
- Write a Box class with length, breadth, height. ColorBox ISA Box with an additional color attribute. Create a Box reference for a ColorBox object and print its attributes through the reference. _(2023 CT Set III, 2024 Exam)_
- A vehicle can change gear and apply brakes. Design Bicycle and Car classes which are vehicles. _(2023 CT Set III)_
- Autorickshaw and EV are both vehicles. All vehicles can change gear and apply brakes. Design the two classes. _(2023 CT Set IV, 2024 Exam)_
- A restaurant makes parathas with various stuffings and combinations. It also offers parathas with specific stuffings depending on time-of-day at a discounted rate. Design a class in Java with methods to take orders from customers. _(2024 Exam)_
- What is the utility of packages in Java? _(2021 CT-II)_
- How does the concept of packages help in organizing a software solution? _(2021 Exam)_
- A software system has basic objects like Employee, Dept, Product, StockInfo, SalesInfo, PurchaseInfo, and modules like EmployeeManagement, SalesManagement, PurchaseManagement. Modules work with subsets of the basic objects. How will you organize your system? _(2021 CT-I, 2021 CT-II)_

---

**Threads (Concurrency)**

- Compare `start()` and `run()`. A Runnable object directly calls `run()` — what will happen? _(2021 CT-I, 2021 CT-II, 2022 Exam)_
- What are two approaches for creating threads? Which is preferred and why? _(2021 CT-II)_
- There is a list of accounts with account number and balance. Multiple threads work on the same account list and update the balance. How will you achieve proper update while allowing maximum concurrency? Provide the design. _(2021 CT-I, 2021 CT-II)_
- A stocklist has itemcode and quantity. Salespersons update the stock. Updates on different items can be simultaneous, but simultaneous update of the same item must be prevented. Design the necessary classes. _(2020 CT, 2021 Exam, 2023 CT Set I)_
- There is a list of items with itemcode, itemname, rate, and quantity. Multiple viewers can view without restriction. But updating quantity of a specific item cannot be done simultaneously; simultaneous update on different items is allowed. Design the classes and write skeleton code. _(2021 Exam)_
- A class contains an integer counting items at a store. A Producer updates this count when a new item is produced. A Consumer reduces it when an item is consumed. Both must access the count consistently. Implement this in Java. _(2023 CT Set II, Set III)_
- There is an inventory class tracking item count. A Supplier class updates count when an item is supplied. A Seller class reduces count when an item is sold. Both must access the count consistently. Write the main method for two Suppliers and one Seller. Discuss the synchronization mechanism adopted. _(2024 Exam)_
- A predesigned class Data (not designed for concurrency) has a method `modify()`. In a multithreaded environment, many threads share the same Data object and may call `modify()`. What measures will you take to prevent simultaneous modification? Describe with skeleton code. _(2021 CT-II)_
- `notifyAll()` may be preferred over `notify()` — explain with a scenario. _(2021 Exam)_
- Comment on the usage and purpose of `wait()` and `notify()`. _(2022 Exam)_
- What is the specialty of a Thread object that it executes in parallel while other objects do not? _(2024 Exam)_
- If a thread waits, how can it resume from waiting? What are the mechanisms? _(2024 Exam)_
- Do you obtain lock on a method or on an object? Discuss with code snippets. _(2024 Exam)_
- Write suitable code snippets to create n threads where n is taken as input. _(2024 Exam)_
- How can you specify the code for a thread and the data on which it works? _(2021 Exam)_
- Create a thread in two ways — Runnable and Thread class. Compare the two approaches. _(2023 CT Set IV)_
- Why is it important to control concurrency in multithreaded programming? Explain with an example and provide skeleton code. _(2022 Exam)_
- Consider Account with account number and balance. In a multithreaded environment, one may query balance, deposit, or withdraw. Query is always allowed. Only one transaction (withdraw/deposit) can be carried out at a time. At the time of withdrawal, if the amount is less than the balance it must wait till money is deposited. Provide skeleton code. _(2022 Exam)_

---

**Exception Handling**

- What is a checked exception? _(2020 CT, 2021 CT-I, 2021 CT-II)_
- How do you deal with checked exceptions? _(2021 CT-I, 2021 CT-II)_
- Ordering of exception handlers is important — explain. _(2022 Exam)_
- What is the use of `finally` in Java? _(2021 Exam)_
- A user inputs details of student objects. Write a Java program to store the inputs in a file. Did you handle any checked/unchecked exceptions here? _(2023 CT Set IV)_

---

**Enums**

- The mass and radius of different planets are given (Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune). Write a Java program that takes mass on Earth as input and calculates weight on a user-chosen planet using the formula: `mass on Earth × (G × mass of planet / radius²)`. State the OOP principles utilized. Write the main method. _(2023 CT Set IV, 2024 Exam)_

---

**IO (File Handling / Collections / Serialization)**

- How can you find the length of an array object and a String object? _(2020 CT)_
- Roll and score of students are to be stored in a data structure where one frequently looks up score by roll. Suggest the scheme in Java with justification and show how to store and retrieve. _(2020 CT, 2023 CT Set I)_
- What is the difference between array and ArrayList? _(2020 CT, 2023 CT Set I)_
- Accept a filename from the user. If it is a directory, display the files in it; otherwise display "not a directory". _(2020 CT, 2021 CT-I, 2021 CT-II)_
- A user-defined class STUDENT has roll, name, and score. Objects are to be stored in a binary file. Design the necessary class. Write code in Java to display all objects from the file. _(2021 Exam, 2023 CT Set II)_
- Assume a binary file has stored marks (integers) for all students. Write code to find the highest score. _(2022 Exam, 2024 Exam)_
- Write a Java program to read the contents of a directory and print only the files, not subdirectories. _(2023 CT Set III)_
- Write a Java program to copy the first two lines from a text file and print to the console. _(2024 Exam)_
- Write a Java program to store key-value pairs from a given list of words, where keys are the words and values are their frequencies. _(2023 CT Set II)_
- A collection in Java holds STUDENT objects. In order to use `contains()`, explain what measures you will take. _(2021 Exam)_
- A collection in Java holds STUDENT objects with roll, name, and score. The collection may be sorted by roll (ascending/descending) or score (ascending/descending). Write code to support this. _(2021 Exam)_
- Each student has roll, name, and score. Take measures and write relevant code so that standard `sort()` can sort the collection in descending order of score. _(2022 Exam)_
- Design a class so that standard `contains()` can be applied to a collection of its objects. _(2022 Exam)_
- What is the specialty and purpose of the Serializable interface? _(2022 Exam)_
- How can you set priority for elements in a priority queue? _(2021 Exam)_
- What is a collection framework? _(2022 Exam)_
- For every friend, name and date of birth (as string) are stored in an array. Design the class(es) and write a method to find friends born in a particular month (given as int). The date string may follow `dd/mm/yyyy` or `dd-mm-yyyy` format. _(2021 CT-I, 2021 CT-II)_
- Define a class for integer matrix operations. Store user inputs in an m×n array. Display the matrix. Calculate sum of border and non-border elements (not visible outside the class). Display results from main. _(2024 Exam)_
- Accept a one-line text string. Assume words are separated by spaces. Find: number of words, longest word, whether "abcd" is present, and display all words. _(2021 CT-II)_
- Write a program in Java that takes a string as input and finds and prints the longest palindrome substring. Make it menu-driven. _(2023 CT Set I)_