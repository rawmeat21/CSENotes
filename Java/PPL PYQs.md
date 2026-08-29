# PPL Question Bank — All Papers, Categorized

Sources: 2018, 2019, 2021, 2024, 2025 Semester Exams; CT1_2022 (Sets I–IV); 2025 Class Test Set III & IV; 2024 Class Test Set III.

---

## PART A — SEMESTER EXAMS

### Pre-CO Scheme — 2018 Semester Exam (Ex/CSE/T/322A/2018)
*Answer any FIVE of the following eight questions. Full Marks: 100*

**Q1.** (a) Discuss about different kinds of data abstraction techniques. (b) Describe Von Neumann bottleneck. (c) How can a programming language be defined? *[10+2+8]*

**Q2.** (a) How would you define efficiency of a programming language? (b) Give an example of orthogonal language design. (c) Discuss about Turing tarpit. (d) State Flon's axiom. *[10+4+3+3]*

**Q3.** (a) Write code snippets to compute factorial of a number following imperative, functional and logic programming paradigm. (b) Write referentially transparent code to compute gcd of a number. Justify your answer. (c) How can functions be treated as first class data values? Give an example. *[10+6+4]*

**Q4.** (a) What is behavior parameterization? How is it implemented in Java? (b) Which of these lambda expressions are valid Function<Long,Long> implementations? Explain: (i) x → x+1; (ii) (x)→(y)→(z) → x+y+z+1. (c) Would the following Java code compile? Give reasons w.r.t functional interfaces: i. `Runnable helloWorld = () -> System.out.println("hello world");` ii. lambda used as ActionListener: `JButton button = new JButton(); String Name=getUserName(); button.addActionListener(event->System.out.println("hello"+ name));` (d) Find the String with the largest number of lowercase letters from a `List<String>` using Lambda expressions in Java. (e) Given a text file, print the duplicate words using Lambda expressions in Java. *[7+3+4+2+4]*

**Q5.** Given the Trader and Transaction classes: (a) Write code snippets using Java Streams API for: (i) list of all unique cities where traders work; (ii) print max and min of all transactions' values for traders living in Cambridge; (iii) group transactions by cities, then further categorize by whether they're expensive or not (multilevel grouping). (b) Write implementations of max(), map(), filter() and count() using only reduce and Lambda expressions (List instead of Stream allowed). *[10+10]*

**Q6.** (a) Use normal order and applicative order reduction to reduce: (i) (λx. λz.z) ((λy. yy) (λu. uu)) (ii) (λx . x x x) (λx . x x x) (iii) ((((λf.(λg.(λx.((fx)(g x)))))(λm.(λn.(n m))))(λn.z))p) (b) How would you find predecessor and successor of 2 in Lambda calculus? (c) How can the successor function be used to sum two natural numbers? Show the steps to find the sum of 2 and 5. *[(3+2+3)+8+4]*

**Q7.** (a) Write Prolog clauses to express: grandparent, cousin, sibling and mother. Given Parent(X,Y) means X is a parent of Y. (b) Write a Prolog program to sort a list of numbers via insertion sort. (c) Given: `ancestor(X,Y):-parent(X,Z),ancestor(Z,Y). ancestor(X,X). parent(amy,bob).` Place cut so that (i) all solutions found; (ii) all solutions pruned; (iii) one solution found. Show the search tree for query ancestor(X,bob) in each case. *[6+5+9]*

**Q8.** (a) Describe multimethods w.r.t OOP. (b) Compare abstract methods and higher order functions. (c) Describe currying in lambda calculus; how is it supported in Java through functional interfaces? Write code snippets. (d) Discuss width subtyping and depth subtyping and their relevance in OOP. *[4+4+7+5]*

---

### CO1 — Java Streams / Collections

**2019 – Q1** *(Group A, [CO1], answer any ONE)* (a) count the number of times each word appears; (b) identify and list the duplicate words; (c) count and print the number of different letters; (d) Write min(), average(), count() using only reduce and lambda expressions. Can it handle empty list? (e) Which of these lambda expressions are valid Function<Long,Long> implementations? Explain: (i) x→x; (ii) (x,y)→x+y+1; (iii) x→x>1. *[4+4+4+5+3=20]*

**2019 – Q2** (a) Print the result of summing up the first 20 fibonacci numbers using Streams. (b) Write an implementation of Stream's map() using only reduce and lambda expressions (List instead of Stream allowed). (c) Why are default methods introduced in Java? Give an example. Describe its resolution rules. (d) Write code using Predicate functional interface (or variations) to generate numbers divisible by 5 between [30,90]. *[3+4+10+3=20]*

**2021 – Q1** *(Group A, [CO1], answer any ONE)* (a) Print the first 4 unique letters; (b) Identify and list the duplicate words; (c) Extract the list of words that end with a number, then print the duplicates among them; (d) max(), average(), count() using only reduce and lambda expressions — can it handle empty list? (e) Compare limit() and skip(). (f) Predicate functional interface (or variations) to generate numbers divisible by 4 between [20,120]. (g) Given a Stream of words, count occurrences of each word — return a Map [<word1>→<count>, <word2>→<count>, …]. *[4+4+4+5+3+4+6=30]*

**2021 – Q2** (a) Implement Stream's filter() using only reduce and lambda expressions (List instead of Stream allowed). (b) Given 2 lists, form distinct pairs of numbers such that the sum is even. (c) Role of Optional class w.r.t streams — example with reduce. (d) From a list of numbers, print the first even number. Given Trader/Transaction classes: (e) Group traders from different cities. (f) Find the transaction with the smallest value for each year. (g) Are any traders based in Mumbai? *[4+5+6+2+5+4+4=30]*

**2024 – Q1** *(Group A, [CO1], answer any TWO)* (a) Differentiate between streams and collections. (b) Given a list of sentences in a text file, group its words into three categories by length (2-letter, 3-letter, more than 3 letters); print total words per subgroup. (c) Given two lists of words — (i) course names, (ii) classroom nos. — identify duplicate pairs formed as (course name, classroom no.). *[4+6+5=15]*

**2024 – Q2** Artist class (name, members, origin — members set may be empty). (a) Find bands with most members using lambda expressions and/or streams API. (b) Find artist with longest name — implement via a Collector, and via reduce; compare the approaches. (c) Print artists' names city-wise, done parallelly. (d) Is `y→y*5` side-effect free? Give reasons. (e) Implement max() using reduce and lambda expressions. *[4+5+2+2+2=15]*

**2024 – Q3** (a) Given a Stream of words, count occurrences of each word, e.g. [Das→3, Sarkar→2]. (b) Create a collection of n Tribonacci numbers via Java Streams API (series: 0,0,1,1,2,4,7,13,24,44,81,…) — print sum of first 20 such numbers. (c) Does reduce() implement the mutable accumulator pattern? Give reasons. (d) Count the number of lowercase letters in a string. *[5+5+3+2=15]*

**2025 – Q1** *(Group A, [CO1], answer any TWO)* (a) Differentiate between (i) "collect" and "collecting"; (ii) external and internal iteration. (b) Given a list of BCSEIII student objects, divide into 3 subgroups by attendance ('regular', 'moderately regular', 'irregular'); print average marks per group. (c) Given a text file, find any character that is repeated. *[5+5+5=15]*

**2025 – Q2** Artist class (name, members, origin). (a) Find bands with *least* members using lambda expressions and/or streams API. (b) Print artists' names city-wise, done parallelly. (c) Form a list representing pairwise summations of numbers taken from two lists — each number should appear exactly once. (d) Partition the list of natural numbers into prime and non-prime using Java Streams. *[4+3+4+4=15]*

**2025 – Q3** (a) Given two lists — (i) course names, (ii) classroom nos. — identify duplicate pairs formed as (course name, classroom no.). (b) Count occurrences of each word in a word Stream, e.g. [Das→3, Sarkar→2]. (c) Discuss mutable accumulator pattern and its effect on parallelization w.r.t Java Streams. (d) Which of these are valid Function<Double,Double> implementations? Explain: (i) x→x; (ii) (x,y)→x+y+1. *[5+5+3+2=15]*

---

### CO2 — Prolog / Logic Programming

**2019 – Q3** *(Group B, [CO2], answer any ONE)* (a) How do you represent a list in Prolog? Write code to reverse the numbers of a list. (b) Write a Prolog program to sort a list via quick sort. (c) Write Prolog code and Horn clauses to compute GCD of two numbers. (d) Discuss the occur-check problem and nonmonotonic reasoning w.r.t logic programming. *[5+5+5+5=20]*

**2019 – Q4** (a) Write a Prolog program that prints the sum of the first 20 natural numbers. Role of cut here? (b) Write Prolog code to compute factorial with and without an accumulator; compare the two approaches. (c) Write Prolog code that finds and validates the minimum of two numbers, using green cut and red cut versions; explain the difference. (d) Write Prolog clauses for grandparent and sibling, given parent(X,Y) means X is a parent of Y. *[5+6+7+2=20]*

**2021 – Q3** *(Group B, [CO2], answer any ONE)* (a) How do you represent a list in Prolog? Write code to reverse the numbers of a list. (b) Express in first-order predicate calculus: If it is raining or snowing, there is precipitation. If it is freezing and there is precipitation, it is snowing. If it is not freezing and there is precipitation, it is raining. It is snowing. (c) What answer does Prolog give for the query "Is it freezing?" and "Is it raining?" Why? Can the clauses be rearranged for better answers? (d) Write Horn clauses to compute GCD of numbers. *[5+6+6+3=20]*

**2021 – Q4** (a) Given: `ancestor(X,Y):-ancestor(Z,Y),parent(X,Z). ancestor(X,X). parent(amy,bob).` Show the search tree for query ancestor(X, bob). Justify. (b) Write Prolog code to compute factorial with and without an accumulator; compare. (c) Explain pattern directed invocation in logic programming. (d) Write Prolog clauses for cousin and sibling, given parent(X,Y) means X is a parent of Y. *[8+6+3+3=20]*

**2024 – Q4** *(Group B, [CO2], answer any ONE)* (a) Write Prolog code for (i) maximum of 3 numbers, (ii) print first 10 natural numbers, (iii) insert an element at the last position of a list, (iv) generate a list by replicating a number n, x times. (b) Build the search tree for (i) and (ii). (c) Write Horn clauses to compute GCD of numbers. *[(3x4)+5+3=20]*

**2024 – Q5** (a) Given: `ancestor(X,Y):-parent(X,Z),!,ancestor(Z,Y). ancestor(X,X). parent(amy,bob).` Show the search tree for query ancestor(amy,X). Discuss the role of cut. (b) Write Prolog code to divide a list into two parts and print all possible combinations; show working for an example input. (c) Write the Prolog program for insertion sort; show steps using unification/resolution for [3,2,1]. (d) Identify the axioms among: `natural(0). natural(2). natural(-1). natural(X):-natural(successor(X)).` *[6+5+7+2=20]*

**2025 – Q4** *(Group B, [CO2], answer any ONE)* (a) Write Prolog code for (i) splitting a list into two parts, (ii) print first 10 natural numbers, (iii) insert an element at the first position of a list, (iv) generate a list by replicating a number n, x times. (b) Build the search tree for (ii) and (iv). (c) Write a Prolog program that prints the reverse of a list; give an example query and output. *[(3x4)+5+3=20]*

**2025 – Q5** (a) Given: `ancestor(X,X):-!. ancestor(X,Y):-parent(X,Z),ancestor(Z,Y). parent(bob,amy).` Show the search tree for query ancestor(X,amy). Discuss the role of cut. (b) Write Prolog code to find the length of a list; show working for an example input. (c) Write a procedural (imperative) code and a Prolog program for insertion sort; differentiate the two approaches; show initial computation steps of the Prolog program for [3,2,1]. *[5+5+10=20]*

---

### CO3 — Lambda Calculus

**2019 – Q5** *(Group C, [CO3], answer any ONE)* (a) State two theorems of Church-Rosser. (b) How do you encode Boolean True and False in Lambda Calculus? Explain. (c) How would you extend this to encode Boolean AND and OR logic? (d) How would you represent the Fibonacci series in Lambda calculus (given Lambda encodings of one, two, etc.)? Explain the role of the combinator. *[4+3+3+10=20]*

**2019 – Q6** (a) How do you encode natural numbers in Lambda Calculus? (b) How would you compare two natural numbers in Lambda calculus? (c) Encode division of natural numbers in Lambda Calculus. (d) Define Lambda Calculus. *[2+6+9+3=20]*

**2021 – Q6** *(Group C, [CO3], answer any ONE)* (a) State two theorems of Church-Rosser. (b) How do you encode Boolean True and False in Lambda Calculus? Explain. (c) How would you extend this to encode an if-then-else construct? (d) Using (b) and (c), derive the predicate is_zero(number) (Church numerals assumed known). (e) Show the deduction steps for is_zero when the Church encoded numeral two is passed as argument. (f) How would you select one element from a pair using Lambda calculus? (g) How would you represent the Fibonacci series in Lambda calculus? Derive all functions needed; explain the role of the combinator. *[4+3+3+3+3+4+10=30]*

**2021 – Q7** (a) How do you encode natural numbers in Lambda Calculus? (b) Define exponent, multiplier, and adder. (c) Show deduction steps for 2^3, 2*2 and 3+3 in lambda calculus. (d) Use any two of these functions to encode twenty-eight given Church numerals for one and three. (d) [second part, sic] How do you derive the predecessor of a natural number in Lambda calculus? Show the deduction for any 1-digit number. (e) Encode division of natural numbers given Church numeral encoding and the predecessor function. (f) Use both normal order and applicative order reduction to reduce (λx.y)((λz.zz)(λw.w)). *[2+3+6+2+7+6+4=30]*

**2024 – Q6** *(Group C, [CO3], answer any TWO)* (a) `i=1;sum=0; while(i<n){sum+=i;i++;} Average=sum/n;` Represent this construct in lambda calculus (Church numerals, Booleans, if-then-else predicates assumed in place). Justify your answer. (b) Show the deduction steps for any number n>3. (c) Compare between Omega combinator and Y combinator. *[10+3+2=15]*

**2024 – Q7** (a) Derive the Boolean operator NOR in lambda calculus and validate its truth table (no need to define 'true'/'false'). (b) "In case there is one or more class tests, you sleep for five hours. Otherwise, you take a nine-hour long nap." Take classtest count as a natural number; represent this in lambda calculus without defining division. Derive any predicates, constructs and data types needed (no need to define Church numerals). (c) Explain by showing the deductions of any one use-case. (d) Apply Normal order reduction to ((((λf.(λg.(λx.((fx)(gx)))))(λm.(λn.(nm)))))(λn.z))p). *[4+6+3+2=15]*

**2024 – Q8** (a) Reduce (λx.y)((λx.xxx)(λx.xxx)) using both normal order and applicative order reduction. (b) Derive the Lambda calculus expression for factorial of a Church numeral using tail recursion (Church numerals, Booleans, if-then-else assumed). (c) Show the steps for factorial three (use delta reduction for multiplication). (d) Given tuples of the form (project topic, faculty name), write a lambda calculus expression to extract the project topic. *[3+7+3+2=15]*

**2025 – Q6** *(Group C, [CO3], answer any TWO)* (a) `i=1;sum=0; while(i<n){sum+=i;i++;} Average=sum/n;` Represent in lambda calculus (Church numerals, Booleans, if-then-else assumed). Justify. (b) Show deduction steps for any number n>3. (c) Apply Normal order reduction to ((((λf.(λg.(λx.((fx)(gx)))))(λm.(λn.(nm)))))(λn.z))p). *[10+3+2=15]*

**2025 – Q7** (a) Design a message filter in Lambda calculus taking two inputs — (i) message from a friend, (ii) semester time flag. If not semester time, all friend's messages are returned, otherwise none — the filter always displays "Believe in your brain, you can". Derive any predicates, combinators and data types needed. (b) Explain by showing the deductions of any one use-case. (c) "In case there is one or more class tests, you sleep for five hours. Otherwise, you take a nine-hour long nap." Represent this in lambda calculus without defining division (no need to define boolean and Church numerals). *[8+3+4=15]*

**2025 – Q8** (a) Derive the Lambda calculus expression for factorial of a Church numeral (Church numerals, Booleans, if-then-else assumed). (b) Show the steps for factorial four (use delta reduction for multiplication). (c) "In a student-teacher meeting, students sit along even numbered rows, teachers along odd numbered rows — so a student's row number is even, else odd." Write this row-checker in Lambda calculus; explain with an example value. *[7+3+5=15]*

---

### CO4 — Currying, Evaluation Semantics, Functional Interfaces

**2019 – Q7** *(Group D, [CO4], answer any ONE)* (a) Advantage of value semantics for designing parallel programming? (b) Explain currying w.r.t Lambda calculus; how is it implemented using functional interfaces such as Function<T,R>? *[4+6=10]*

**2019 – Q8** (a) Compare call-by-name and call-by-value w.r.t Lambda calculus and their applicability to modern languages. (b) Give an example implementation of higher order functions w.r.t Java Streams API. (c) Do higher order functions follow call-by-value? Give reasons. *[6+2+2=10]*

**2021 – Q8** *(Group D, [CO4], answer any ONE)* (a) Advantage of value semantics for parallel programming? (b) Explain currying w.r.t Lambda calculus; its role in higher order functions; implementation using Function<T,R>. *[4+6=10]*

**2021 – Q9** (a) Discuss applicability of call-by-name and call-by-value semantics of Lambda calculus to modern programming languages, with examples. (b) Define referential transparency and its benefit for program efficiency. (c) Which semantics do higher order functions follow? Give reasons. *[4+4+2=10]*

**2024 – Q9** *(Group D, [CO4], answer any ONE)* (a) Discuss the concept of purity reflection in functional programming. (b) Compare call-by-need and call-by-name. (c) Define functional interface. Is `java.io.Closeable` a functional interface? *[4+3+3=10]*

**2024 – Q10** (a) Discuss the applicability of normal order and applicative order reduction to modern programming languages. (b) Discuss the concept of call-by-need. *[6+4=10]*

**2025 – Q9** *(Group D, [CO4], answer any ONE)* (a) Discuss the concept of purity reflection in functional programming. (b) Analyze how memoisation is implemented in logic programming and in functional programming. (c) Define functional interface. Is `java.io.Closeable` a functional interface? *[4+4+2=10]*

**2025 – Q10** (a) Discuss the applicability of normal order reduction to modern programming languages. (b) What is the need for introducing typed lambda calculus over untyped lambda calculus? *[5+5=10]*

---

### CO5 & CO6 — Abstraction, Language Design, Paradigms, OOP

**2019 – Q9** *(Group E, [CO5, CO6], answer any TWO)* (a) Discuss different types of control abstractions. (b) Discuss von Neumann bottleneck and its effect on the imperative paradigm; name one paradigm where it can be overcome. *[10+5=15]*

**2019 – Q10** (a) Define and give one example each of orthogonality, generality, and uniformity in a programming language of your choice. (b) Compare static typing and dynamic typing. *[(4x3)+3=15]*

**2019 – Q11** (a) Compare higher order functions and abstract methods. (b) Explain functional decomposition and object-oriented decomposition; state advantages of each. (c) Why do you need a "final" class in Java even though it prevents extensibility? *[5+(4+4)+2=15]*

**2021 – Q11** *(Group E, [CO5, CO6], answer any ONE)* (a) Compare data structure and abstract data type w.r.t abstraction. (b) Discuss the bottlenecks of the imperative paradigm; name a paradigm where these are overcome and explain how. *[4+6=10]*

**2021 – Q12** (a) Compare static typing and dynamic typing w.r.t common language design criteria. (b) Define orthogonality — how is it supported (or not) in Java? Discuss. *[4+6=10]*

**2024 – Q11** *(Group E, [CO5, CO6], answer any ONE)* (a) Define higher order function. (b) Discuss the concept of data abstraction. *[4+6=10]*

**2024 – Q12** (a) What are the main drawbacks of imperative programming? (b) Discuss the role of double dispatch in OOP. (c) Discuss the concept of unit abstraction. *[3+5+2=10]*

**2025 – Q11** *(Group E, [CO5, CO6], answer any ONE)* (a) Discuss the role of regularity and its aspects in programming language design. (b) Compare OO and functional programming styles w.r.t unplanned extensions (extensibility) of code. *[6+4=10]*

**2025 – Q12** "A Planet Explorer routinely travels across the planets in the Solar System to discover life form. However, the method of exploring is different on each planet, due to the difference in atmosphere and surface composition." Write suitable class(es) and/or interface(s) to represent the scenario; analyze how single/double dispatch polymorphism is supported by your code. *[10]*

> **Note:** The 2021 paper's Group B and Group C, as scanned, skip from Q4 straight to Q6, and from Q9 straight to Q11 — no Q5 or Q10 appears in the source document.

---

## PART B — CLASS TEST I PAPERS (not CO-tagged in source)

### CT1_2022_Sep14-09-2022.pdf — Sets I–IV (Full Marks 30 each, Time: 30 min)

**Set I**
1. Reduce (λx. λz.z) ((λy. yy) (λu. uu)) following call-by-name and call-by-value. State the kind of reduction used in each step. *[4]*
2. `if(roll_no%2==0) return roll_no; else return "Odd Number";` Represent in lambda calculus; derive any predicates, constructs and data types needed. *[8]*
3. Reduce: (i) ((λx.((λy.(x y))x))(λz.w)) (ii) ((λf.((λg.((f f)g))(λh.(k h))))(λx.(λy.y))) *[6]*
4. Given a text file, group its words into three categories by length: 2-letter, 3-letter, more than 3 letters. *[6]*
5. Assuming Church numerals are in place, how can you compute "subtract five from three"? *[6]*

**Set II**
1. Given lambda expressions for zero(0), one(1), two(2), three(3), derive the number 18 using successor, add, and other binary operators (define each separately). *[4]*
2. How do you encode Boolean True and False in Lambda Calculus? Explain. *[3]*
3. Validate the truth table of NAND using the derivation from Q2. *[4]*
4. Reduce: ((((λf.(λg.(λx.((fx)(gx)))))(λm.(λn.(nm)))))(λn.z))p) *[3]*
5. Traders/Transaction classes given. (i) Group traders from different cities. (ii) Find the transaction with the smallest value for each year. *[9]*
6. Partition the list of natural numbers into prime and non-prime using Java Streams. *[7]*

**Set III**
1. Derive the Boolean operator OR in lambda calculus and validate its truth table. *[5]*
2. `int arr[]={1,2,3}; if(arr[2]%2==0) return "left"; else return "right";` Represent in lambda calculus; derive any predicates, constructs, data types needed (no need to define Church numerals or the Q1 expressions). *[12]*
3. Show the deduction steps for is_zero when the Church encoded numeral two is passed as argument. *[3]*
4. Implement min(), average(), count() using collect and lambda expressions. Can it handle an empty list? *[6]*
5. Given a list of Dish objects, partition them into high calorie and low calorie groups. *[4]*

**Set IV**
1. `if(roll_no==0) return roll_no+1; else return roll_no;` Represent in lambda calculus; derive any predicates, constructs and data types needed. *[8]*
2. Reduce: (i) ((λf.((λg.((f f)g))(λh.(k h))))(λx.(λy.y))) (ii) (λg.((λf.((λx.(f(x x)))(λx.(f(x x)))))g)) *[5]*
3. Assuming Church numerals are in place, how can you compute "subtract four from three"? *[7]*
4. Given a list of Dish objects, partition them into high calorie and low calorie groups. *[4]*
5. Given a text file, group its words into three categories by length: 2-letter, 3-letter, more than 3 letters. *[6]*

---

### 2025 Class Test I — Set III (Full Marks 30)
1. Derive the Boolean operator OR in lambda calculus and validate its truth table (2 different input combinations). *[5]*
2. Design a message filter in Lambda calculus taking two inputs — (i) message from a friend, (ii) semester time flag. If not semester time, all friend's messages returned, else none; filter always displays "No time for distraction". Derive any predicates, constructs and data types needed (no need to define the expressions derived as Ans. 1). *[6]*
3. Given a list of salaries of employees; salaries above Rs. 100,000 are taxable at 5%. Display the list of taxes payable for the taxable salaries (Java streams or Python functional programming). *[8]*
4. Given a list of BCSEIII student objects, divide into 3 subgroups by attendance (<50%, 50%-80%, >80%). Print average marks scored by each group. *[8]*
5. How does an upstream collector differ from a downstream collector in Java streams? Does collect exhibit a combination of lazy and eager evaluation when dealing with multi-level grouping? Give reasons. *[3]*

### 2025 Class Test I — Set IV (Full Marks 30)
1. Represent append and prepend operations of a list in lambda calculus (no need to derive pair/extraction). Show reductions to construct list [1,2] and extract 2 from it (Church numerals assumed). *[8]*
2. In Lambda Calculus, generate an alternate sequence of Boolean values for n=3 (n a Church numeral). *[5]*
3. Given a list of Dish objects, partition them into high calorie and low calorie groups. *[3]*
4. Form a list representing pairwise summations of numbers taken from two lists — each number appearing exactly once. Write code using Java Streams. *[4]*
5. Compare between collect and reduce. *[3]*
6. Discuss partial application in Python from the functional programming perspective. *[3]*
7. Given two lists — students and seminar topics — form the pairs using Java Streams and display them. State the functional interface used by the higher-order function(s) applied. *[4]*

### 2024 Class Test I — Set III (Full Marks 30)
1. Derive the Boolean operator OR in lambda calculus and validate its truth table. *[5]*
2. Design a message filter in Lambda calculus taking two inputs — (i) message from a friend, (ii) semester time flag. If not semester time, all friend's messages returned, else none; filter always displays "Believe in your brain, you can". Derive any predicates, constructs and data types needed (no need to define the Ans. 1 expressions). *[8]*
3. Show the deduction steps for is_zero when the Church encoded numeral two is passed as argument. *[4]*
4. Given a list of BCSEIII student objects, divide into 3 subgroups by attendance (<50%, 50%-80%, >80%). Print average marks scored by each group. *[8]*
5. Partition the first n natural numbers into two groups — prime and nonprime. Can you do it in parallel? *[5]*

---

## PART C — RECURRING QUESTION THEMES (repeats across papers)

- **Trader/Transaction Streams queries** (group by city, min/max value, etc.): 2018 Q5(a); 2021 Q2(e,f,g); CT1 Set II Q5.
- **Boolean operator encoding + truth table in Lambda calculus** (True/False, AND/OR/NOR/NAND): 2019 Q5(b,c); 2021 Q6(b); CT1 Set II Q2–3 (NAND); CT1 Set III Q1 (OR); 2024 semester Q7(a) (NOR); 2024 Class Test Set III Q1 (OR); 2025 Class Test Set III Q1 (OR).
- **is_zero deduction for Church numeral "two"**: 2021 Q6(e); CT1 Set III Q3; 2024 Class Test Set III Q3.
- **Message filter (friend's messages / semester-time flag)**: 2025 semester Q7(a); 2024 Class Test Set III Q2; 2025 Class Test Set III Q2.
- **Attendance-based subgroup partition of BCSEIII student objects**: 2025 semester Q1(b) (regular/moderately regular/irregular); 2024 Class Test Set III Q4; 2025 Class Test Set III Q4 (<50%, 50-80%, >80%).
- **Dish objects → high/low calorie partition**: CT1 Set III Q5; CT1 Set IV Q4; 2025 Class Test Set IV Q3.
- **Text file → group words by length (2-letter/3-letter/>3-letter)**: CT1 Set I Q4; CT1 Set IV Q5; 2024 semester Q1(b).
- **Factorial of a Church numeral (lambda calculus, delta reduction)**: 2024 semester Q8(b,c); 2025 semester Q8(a,b).
- **Prime/non-prime partition of natural numbers**: CT1 Set II Q6; 2025 semester Q2(d); 2024 Class Test Set III Q5 (also asks about doing it in parallel).
- **Ancestor/cut Prolog search tree**: 2018 Q7(c); 2021 Q4(a); 2024 semester Q5(a); 2025 semester Q5(a).
- **GCD via Horn clauses in Prolog**: 2019 Q3(c); 2021 Q3(d); 2024 semester Q4(c).
- **Generic multi-step lambda reduction** ((((λf.(λg.(λx.((fx)(gx)))))(λm.(λn.(nm)))))(λn.z))p): 2018 Q6(iii, structurally similar); CT1 Set II Q4; 2024 semester Q7(d); 2025 semester Q6(c).
- **Currying w.r.t Lambda calculus / Function<T,R>**: 2018 Q8(c); 2019 Q7(b); 2021 Q8(b).
- **Call-by-name vs call-by-value / call-by-need applicability to modern languages**: 2019 Q8(a); 2021 Q9(a); 2024 semester Q9(b), Q10(b); 2025 semester Q10(a).
- **Do higher order functions follow call-by-value? / their evaluation semantics**: 2019 Q8(c); 2021 Q9(c).
- **Insertion sort — Prolog (and comparison to procedural code)**: 2018 Q7(b); 2019 Q3(b) [quick sort variant]; 2025 semester Q5(c); CT1-adjacent themes not directly repeated elsewhere.
- **purity reflection / functional interface (java.io.Closeable)**: 2024 semester Q9(a,c); 2025 semester Q9(a,c) (2025 adds memoisation comparison).
- **Static vs dynamic typing**: 2019 Q10(b); 2021 Q12(a).
- **Orthogonality / generality / uniformity in language design**: 2019 Q10(a); 2021 Q12(b) (orthogonality only); 2025 semester Q11(a) (regularity).


### What to study from chapter 1:

**Paradigms / abstraction / Von Neumann bottleneck** (matches slides 1–7)

- 2018 Q1(a,c), Q2(a,c,d)
- 2019 Q9(b), Q10(a,b)
- 2021 Q11(b), Q12(b)
- 2024 Q12(a)
- 2025 Q11(a,b), Q12

**OOP as extension of imperative paradigm** (slide "Object-oriented Programming")

- 2018 Q8(a,d) — multimethods, width/depth subtyping
- 2024 Q12(b) — double dispatch
- 2025 Q12 — Planet Explorer, single/double dispatch

**Mathematical function view, referential transparency, first-class values** (slides "Functional Programming" through "First class data values")

- 2018 Q3(b,c) — referentially transparent GCD, functions as first-class values
- 2019 Q8(b) — referential transparency + program efficiency
- 2021 Q9(b) — same

**Value semantics, currying, higher-order functions** (slides "Value vs Reference Semantics", "Higher order functions")

- 2018 Q8(c) — currying + functional interfaces
- 2019 Q7(a,b), Q8(a,c) — value semantics, currying, HOF call-by-value
- 2021 Q8(a,b), Q9(c) — same set
- 2019 Q11(a) — HOF vs abstract methods

**Lambda expressions / behavior parameterization / functional interfaces** (the bulk of the deck — slides on Lambda Expressions, Capturing Lambdas, Functional Interfaces, Boxing/Unboxing, Target Typing, Overloading)

- 2018 Q4(a–e) — this is the closest direct match: behavior parameterization, valid Function<Long,Long> lambdas, whether code compiles w.r.t. functional interfaces, finding a string with lambdas, duplicate words with lambdas
- 2019 Q1(e) — valid Function<Long,Long> implementations
- 2025 Q3(d) — valid Function<Double,Double> implementations
- 2024 Q9(c) / 2025 Q9(c) — define functional interface; is `java.io.Closeable` one?


