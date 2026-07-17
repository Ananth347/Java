# Java Fundamentals — Interview Q&A (Simple English)

> Easy words. Short sentences. Every answer explained clearly, step by step, with code examples.

---

## Table of Contents
- [Section A: Introduction to Java](#section-a-introduction-to-java)
- [Section B: JDK, JRE, JVM](#section-b-jdk-jre-jvm)
- [Section C: Program Execution Flow](#section-c-program-execution-flow)
- [Section D: Variables](#section-d-variables)
- [Section E: Data Types](#section-e-data-types)
- [Section F: Operators](#section-f-operators)
- [Section G: Type Casting](#section-g-type-casting)
- [Section H: Control Statements](#section-h-control-statements)
- [Section I: Loops](#section-i-loops)
- [Section J: Methods](#section-j-methods)
- [Section K: Command Line Arguments](#section-k-command-line-arguments)
- [Section L: Arrays](#section-l-arrays)
- [Section M: Strings](#section-m-strings)
- [Section N: Wrapper Classes](#section-n-wrapper-classes)
- [Section O: Autoboxing & Unboxing](#section-o-autoboxing--unboxing)
- [Section P: Enums](#section-p-enums)
- [Section Q: Packages](#section-q-packages)
- [Section R: Access Modifiers](#section-r-access-modifiers)
- [Section S: Tricky/Mixed Questions](#section-s-trickymixed-questions)

---

## Section A: Introduction to Java

### Q1. What is Java?
**Answer:** Java is a programming language. People use it to write software — like mobile apps, websites, banking systems, and games. Java was made by Sun Microsystems (now owned by Oracle), and it first came out in 1995.

### Q2. What are the main features of Java?
**Answer:**
- **Simple** — easy to learn, especially if you know C or C++
- **Object-Oriented** — code is organized around objects and classes
- **Platform Independent** — the same program can run on Windows, Linux, or Mac ("Write Once, Run Anywhere")
- **Secure** — has built-in safety checks
- **Robust** — handles errors well, has strong memory management
- **Multithreaded** — can do many tasks at the same time
- **Automatic Memory Management** — Java cleans up unused memory by itself (Garbage Collection), so you don't have to do it manually

### Q3. Why is Java called "platform independent"?
**Answer:** Because a Java program does not need to be rewritten for each different computer type. You write your code once, and it can run on any computer that has a JVM installed. This works because Java code is compiled into a middle format called **bytecode**, which is the same everywhere.

### Q4. Is Java a compiled language or an interpreted language?
**Answer:** Java is **both**, in a way.
1. First, your `.java` file is **compiled** into bytecode using `javac`
2. Then, JVM **interprets/runs** that bytecode (and also uses a JIT compiler to speed things up)

So Java uses a mix of both compiling and interpreting.

### Q5. What is the entry point of a Java program?
**Answer:** Every Java program starts running from the `main` method. This is called the **entry point**.
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

### Q6. Why is the `main` method written as `public static void main(String[] args)`?
**Answer:** Let's look at each word:
- `public` — so JVM (which is outside your class) can call this method
- `static` — so JVM can call it WITHOUT creating an object of the class first
- `void` — the method does not return any value
- `main` — this exact name is what JVM looks for to start the program
- `String[] args` — allows the program to receive command-line arguments (explained later)

---

## Section B: JDK, JRE, JVM

### Q7. What is JVM?
**Answer:** JVM means **Java Virtual Machine**. It is the program that actually runs your compiled Java bytecode. It is called "virtual" because it behaves like a computer, but it is really just software.

### Q8. What is JRE?
**Answer:** JRE means **Java Runtime Environment**. It contains the JVM plus the basic Java libraries needed to RUN a Java program. If you only want to run a Java program (not build one), JRE is enough.

### Q9. What is JDK?
**Answer:** JDK means **Java Development Kit**. It contains the JRE plus development tools, like the compiler (`javac`), needed to WRITE and BUILD Java programs.

### Q10. What is the relationship between JDK, JRE, and JVM?
**Answer:** `JDK` contains `JRE`, and `JRE` contains `JVM`. So:
```
JDK  ⊃  JRE  ⊃  JVM
```
**Simple way to remember:**
- JVM = the engine
- JRE = the whole car (engine + parts needed to drive)
- JDK = the car + a full toolbox to build and fix it

### Q11. Can I run a Java program using only JVM, without JRE?
**Answer:** No, not practically. JVM alone cannot run a real Java program because it needs the basic Java class libraries (like `System`, `String`) that come with JRE. JVM is just the engine — it needs the rest of the JRE around it to actually work.

---

## Section C: Program Execution Flow

### Q12. What happens step by step when you run a Java program?
**Answer:**
1. You write code in a `.java` file
2. `javac` (the compiler) converts it into a `.class` file (bytecode)
3. JVM's **Class Loader** loads this `.class` file into memory
4. JVM checks the bytecode for safety (Bytecode Verification)
5. JVM's **Execution Engine** runs the bytecode, using the `main` method as the starting point

```
MyApp.java --(javac)--> MyApp.class (bytecode) --(JVM)--> program runs
```

### Q13. What is bytecode?
**Answer:** Bytecode is the middle format your Java code turns into after compiling. It is not the original Java code, and it is also not the computer's final machine language. It sits in between, and it is the SAME on every computer, which is why Java programs can run anywhere a JVM is installed.

### Q14. Does JVM read `.java` files directly?
**Answer:** No. JVM never reads `.java` files. Only the `javac` compiler reads `.java` files, and it converts them into `.class` bytecode files. JVM only works with these already-compiled bytecode files.

---

## Section D: Variables

### Q15. What is a variable in Java?
**Answer:** A variable is a name given to a memory location, used to store a value that your program can use and change while it runs.
```java
int age = 25;   // "age" is a variable holding the value 25
```

### Q16. What are the rules for naming a variable in Java?
**Answer:**
- Can contain letters, digits, underscore (`_`), and dollar sign (`$`)
- Cannot start with a digit
- Cannot be a Java keyword (like `int`, `class`, `public`)
- Case-sensitive (`age` and `Age` are different variables)
- No spaces allowed

### Q17. What are the types of variables in Java?
**Answer:** There are 3 types:
1. **Local Variable** — declared inside a method; only exists while that method runs
2. **Instance Variable** — declared inside a class, but outside any method; belongs to each object separately
3. **Static (Class) Variable** — declared with the `static` keyword; shared by ALL objects of the class

```java
class Demo {
    int instanceVar = 10;         // instance variable
    static int staticVar = 20;    // static variable

    void show() {
        int localVar = 30;         // local variable
    }
}
```

### Q18. What is the difference between an instance variable and a static variable?
**Answer:**

| Instance Variable | Static Variable |
|--------------------|-------------------|
| Each object gets its own separate copy | Only ONE copy exists, shared by all objects |
| Needs an object to be accessed | Can be accessed without creating an object |
| Declared without `static` keyword | Declared with `static` keyword |

### Q19. Do local variables get a default value automatically?
**Answer:** No. Unlike instance and static variables, **local variables do NOT get a default value**. You must give them a value yourself before using them, or the code will not compile.
```java
void test() {
    int x;
    System.out.println(x);   // COMPILE ERROR: variable x might not have been initialized
}
```

---

## Section E: Data Types

### Q20. What are Data Types in Java?
**Answer:** A data type tells Java what kind of value a variable can hold, and how much memory it needs. Java has two main groups: **Primitive Data Types** and **Non-Primitive (Reference) Data Types**.

### Q21. What are the 8 primitive data types in Java?
**Answer:**

| Data Type | Size | Example Value |
|-----------|------|-----------------|
| `byte` | 1 byte | 100 |
| `short` | 2 bytes | 10000 |
| `int` | 4 bytes | 100000 |
| `long` | 8 bytes | 100000L |
| `float` | 4 bytes | 3.14f |
| `double` | 8 bytes | 3.14159 |
| `char` | 2 bytes | 'A' |
| `boolean` | 1 bit (JVM dependent) | true / false |

### Q22. What is the difference between Primitive and Non-Primitive (Reference) data types?
**Answer:**

| Primitive | Non-Primitive (Reference) |
|-----------|------------------------------|
| Stores the actual value directly | Stores a reference (address) pointing to an object |
| Fixed size, defined by Java itself | Size depends on the object |
| Examples: `int`, `char`, `boolean` | Examples: `String`, arrays, classes, interfaces |
| Cannot be `null` | Can be `null` |
| Stored in Stack (for local variables) | The object itself is stored in Heap |

### Q23. What is the default value of each primitive data type?
**Answer:** (These default values apply to instance and static variables, NOT local variables)

| Data Type | Default Value |
|-----------|-----------------|
| `byte`, `short`, `int`, `long` | 0 |
| `float`, `double` | 0.0 |
| `char` | '\u0000' (a blank/null character) |
| `boolean` | false |
| Any reference type (String, objects) | null |

### Q24. What is the difference between `float` and `double`?
**Answer:**
- `float` uses 4 bytes and gives about 6-7 correct decimal digits
- `double` uses 8 bytes and gives about 15-16 correct decimal digits (much more accurate)
- By default, any decimal number you write in Java (like `3.14`) is treated as `double`. To make it a `float`, you must add an `f` at the end, like `3.14f`.

### Q25. Why is `char` a 2-byte type in Java, while it is usually 1 byte in other languages like C?
**Answer:** Because Java's `char` uses **Unicode**, which can represent characters from almost every language in the world (not just English letters). Unicode needs more space than the older, English-only character sets, so Java gives `char` 2 bytes instead of 1.

---

## Section F: Operators

### Q26. What are Operators in Java?
**Answer:** Operators are special symbols used to perform operations on variables and values, like addition, comparison, or logical checks.

### Q27. What are the main types of operators in Java?
**Answer:**
1. **Arithmetic Operators** — `+`, `-`, `*`, `/`, `%`
2. **Relational (Comparison) Operators** — `==`, `!=`, `>`, `<`, `>=`, `<=`
3. **Logical Operators** — `&&`, `||`, `!`
4. **Assignment Operators** — `=`, `+=`, `-=`, `*=`, `/=`
5. **Unary Operators** — `+`, `-`, `++`, `--`, `!`
6. **Bitwise Operators** — `&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`
7. **Ternary (Conditional) Operator** — `? :`

### Q28. What is the difference between `==` and `.equals()`?
**Answer:**
- `==` checks if two variables point to the exact SAME memory location (for objects), or checks the actual value (for primitives)
- `.equals()` checks if the CONTENT/VALUE inside two objects is the same

```java
String s1 = new String("Hi");
String s2 = new String("Hi");
System.out.println(s1 == s2);        // false - different objects
System.out.println(s1.equals(s2));   // true - same content
```

### Q29. What is the difference between `++x` (pre-increment) and `x++` (post-increment)?
**Answer:**
- `++x` (pre-increment) increases the value of `x` FIRST, and then uses the new value
- `x++` (post-increment) uses the CURRENT value of `x` first, and then increases it afterward

```java
int x = 5;
int a = ++x;   // x becomes 6 first, then a = 6
int y = 5;
int b = y++;   // b = 5 first (old value), then y becomes 6
```

### Q30. What is the Ternary Operator?
**Answer:** It is a shortcut way of writing a simple `if-else` statement, using the format: `condition ? valueIfTrue : valueIfFalse`

```java
int age = 20;
String result = (age >= 18) ? "Adult" : "Minor";
System.out.println(result);   // Adult
```

### Q31. What is the difference between `&` / `|` and `&&` / `||`?
**Answer:**
- `&` and `|` are **bitwise** operators, but they can also be used as logical operators. They ALWAYS check BOTH sides of the condition, even if the answer is already clear from the first side
- `&&` and `||` are **logical** operators that use **short-circuit evaluation** — meaning they SKIP checking the second condition if the first one already decides the result

```java
if (false && (10 / 0 == 0)) { }   // safe - second part is never checked, no error
if (false & (10 / 0 == 0)) { }    // ERROR - both sides are always checked, division by zero happens
```

---

## Section G: Type Casting

### Q32. What is Type Casting in Java?
**Answer:** Type Casting means converting a value from one data type into another data type.

### Q33. What is Implicit Type Casting (Widening)?
**Answer:** This happens automatically, when converting a SMALLER data type into a LARGER data type. Java does this by itself, safely, without any data loss.
```java
int x = 100;
double y = x;   // automatic: int -> double (widening)
```
**Order (small to large):** `byte → short → int → long → float → double`

### Q34. What is Explicit Type Casting (Narrowing)?
**Answer:** This happens when converting a LARGER data type into a SMALLER data type. Java does NOT do this automatically — you must do it yourself, using parentheses, because there is a risk of losing data.
```java
double y = 100.99;
int x = (int) y;   // manual cast: double -> int (narrowing) - result: 100 (decimal part lost)
```

### Q35. What happens if you cast a large value into a smaller data type, and it doesn't fit?
**Answer:** The extra bits that don't fit are simply cut off/lost, which can produce a surprising, wrong-looking result.
```java
int x = 300;
byte b = (byte) x;   // byte can only hold -128 to 127, so this "wraps around"
System.out.println(b);   // prints 44 (not 300!)
```

### Q36. Can you assign an `int` value to a `byte` variable directly, without casting?
**Answer:** Only if the value is a small constant that fits within `byte`'s range (-128 to 127) AND it is known at compile time. Otherwise, you need explicit casting.
```java
byte b = 100;    // OK - small constant, fits in byte range
int x = 100;
byte b2 = x;     // COMPILE ERROR - x is a variable, needs explicit (byte) cast
```

---

## Section H: Control Statements

### Q37. What are Control Statements in Java?
**Answer:** Control Statements decide WHICH parts of your code should run, based on certain conditions. They control the "flow" of the program.

### Q38. What is an `if` statement?
**Answer:** It runs a block of code only if a certain condition is `true`.
```java
int age = 20;
if (age >= 18) {
    System.out.println("You are an adult");
}
```

### Q39. What is an `if-else` statement?
**Answer:** It runs one block of code if the condition is `true`, and a different block if the condition is `false`.
```java
if (age >= 18) {
    System.out.println("Adult");
} else {
    System.out.println("Minor");
}
```

### Q40. What is an `if-else-if` ladder?
**Answer:** It is used when you need to check MULTIPLE conditions, one after another, and only run the block for the FIRST condition that matches.
```java
if (marks >= 90) {
    System.out.println("Grade A");
} else if (marks >= 75) {
    System.out.println("Grade B");
} else if (marks >= 50) {
    System.out.println("Grade C");
} else {
    System.out.println("Fail");
}
```

### Q41. What is a `switch` statement? Give an example.
**Answer:** A `switch` statement checks a single value against many possible cases, and runs the matching block. It is often a cleaner alternative to a long `if-else-if` ladder.
```java
int day = 3;
switch (day) {
    case 1: System.out.println("Monday"); break;
    case 2: System.out.println("Tuesday"); break;
    case 3: System.out.println("Wednesday"); break;
    default: System.out.println("Invalid day");
}
```

### Q42. Why do we use `break` inside a `switch` statement?
**Answer:** `break` stops the `switch` from continuing to the NEXT case after a match is found. Without `break`, Java will keep running every case below the matching one too — this is called **fall-through**, and it is usually NOT what you want (unless done on purpose).
```java
int day = 2;
switch (day) {
    case 1: System.out.println("Mon");
    case 2: System.out.println("Tue");   // matches here
    case 3: System.out.println("Wed");   // still runs! (no break above)
}
// Output: Tue  Wed   (fall-through happened)
```

### Q43. Can a `switch` statement work with `String` values?
**Answer:** Yes, since Java 7, `switch` supports `String` values, in addition to `int`, `char`, `byte`, `short`, and `enum` types.
```java
String day = "MON";
switch (day) {
    case "MON": System.out.println("Monday"); break;
    case "TUE": System.out.println("Tuesday"); break;
}
```

---

## Section I: Loops

### Q44. What is a loop in Java?
**Answer:** A loop is used to run the same block of code multiple times, without writing it again and again.

### Q45. What are the types of loops in Java?
**Answer:**
1. **`for` loop** — best when you know exactly how many times to repeat
2. **`while` loop** — best when you don't know the exact number of repeats, and the condition is checked BEFORE running the code
3. **`do-while` loop** — similar to `while`, but the condition is checked AFTER running the code, so it always runs AT LEAST once
4. **Enhanced `for` loop (for-each)** — used to easily go through every item in an array or collection

### Q46. Give an example of a `for` loop.
**Answer:**
```java
for (int i = 1; i <= 5; i++) {
    System.out.println(i);
}
// prints 1 2 3 4 5
```
A `for` loop has 3 parts: **initialization** (`int i = 1`), **condition** (`i <= 5`), and **update** (`i++`).

### Q47. Give an example of a `while` loop.
**Answer:**
```java
int i = 1;
while (i <= 5) {
    System.out.println(i);
    i++;
}
```
The condition is checked FIRST. If it's `false` right away, the loop body never runs even once.

### Q48. Give an example of a `do-while` loop.
**Answer:**
```java
int i = 1;
do {
    System.out.println(i);
    i++;
} while (i <= 5);
```
The code inside `do { }` runs FIRST, and THEN the condition is checked. This means a `do-while` loop always runs **at least once**, even if the condition is `false` from the very start.

### Q49. What is the difference between `while` and `do-while` loops?
**Answer:**

| `while` loop | `do-while` loop |
|--------------|-------------------|
| Condition checked BEFORE running the code | Condition checked AFTER running the code |
| Might run ZERO times, if condition is false at start | Always runs AT LEAST ONCE |

### Q50. What is a for-each loop? Give an example.
**Answer:** It is a simpler way to go through every item in an array or collection, without needing to manage an index number yourself.
```java
int[] numbers = {10, 20, 30};
for (int num : numbers) {
    System.out.println(num);
}
```

### Q51. What is the difference between `break` and `continue`?
**Answer:**
- `break` **completely stops** the loop, and moves to the code right after the loop
- `continue` **skips only the current repeat**, and moves to the NEXT repeat of the loop

```java
for (int i = 1; i <= 5; i++) {
    if (i == 3) break;
    System.out.println(i);
}
// prints: 1 2   (loop stops completely at 3)

for (int i = 1; i <= 5; i++) {
    if (i == 3) continue;
    System.out.println(i);
}
// prints: 1 2 4 5   (only 3 is skipped, loop continues)
```

### Q52. What is an infinite loop? How can it happen by mistake?
**Answer:** An infinite loop is a loop that never stops running, because its condition never becomes `false`. This often happens by mistake, for example if you forget to update the loop variable.
```java
int i = 1;
while (i <= 5) {
    System.out.println(i);
    // forgot to write i++  - this loop will run forever!
}
```

---

## Section J: Methods

### Q53. What is a Method in Java?
**Answer:** A method is a named block of code that performs a specific task. You can call (run) a method whenever you need that task done, instead of writing the same code repeatedly.
```java
void greet() {
    System.out.println("Hello!");
}
```

### Q54. What are the parts of a method definition?
**Answer:**
```java
public static int add(int a, int b) {
    return a + b;
}
```
- `public static` — access modifier + optional keywords
- `int` — return type (the type of value this method gives back)
- `add` — method name
- `(int a, int b)` — parameters (inputs the method needs)
- `{ return a + b; }` — method body (the actual code, plus the returned result)

### Q55. What is the difference between a method Parameter and an Argument?
**Answer:**
- A **Parameter** is the variable listed in the method's definition
- An **Argument** is the actual value you pass in when you CALL the method

```java
void greet(String name) {    // "name" is a parameter
    System.out.println("Hello " + name);
}
greet("Raj");   // "Raj" is the argument
```

### Q56. What is Method Overloading?
**Answer:** Having multiple methods with the SAME name, but DIFFERENT parameter lists, within the same class.
```java
int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }
```

### Q57. What is the difference between `void` and other return types?
**Answer:** `void` means the method does NOT return any value. Any other type (like `int`, `String`, `boolean`) means the method MUST return a value of that exact type, using the `return` keyword.
```java
void printMessage() {                 // returns nothing
    System.out.println("Hi");
}
int square(int x) {                    // must return an int
    return x * x;
}
```

### Q58. What does "pass by value" mean in Java? Does Java support "pass by reference"?
**Answer:** Java always uses **pass by value**. This means when you call a method, Java copies the VALUE of the variable and passes that copy — not the original variable itself.

For primitives, this means changes inside the method do NOT affect the original variable.
```java
void change(int x) {
    x = 100;
}
int a = 5;
change(a);
System.out.println(a);   // still 5, unchanged
```

For objects, Java still passes a COPY of the reference (the object's address). This means you CAN change the object's internal data through this reference, but you CANNOT make the original variable point to a totally different object.
```java
void changeName(Student s) {
    s.name = "Changed";     // this DOES affect the original object's data
    s = new Student();       // this does NOT affect the original reference
}
```
This is why some people say Java "looks like" pass by reference for objects, but it is technically still pass by value — the value being passed is just the reference/address itself.

### Q59. Can a method return more than one value?
**Answer:** Not directly. A Java method can only return ONE value. But you can return multiple values indirectly, by:
- Returning an array
- Returning a `List` or other collection
- Returning a custom object/class that holds multiple fields

---

## Section K: Command Line Arguments

### Q60. What are Command Line Arguments in Java?
**Answer:** These are values you can pass to your Java program from OUTSIDE, when you start it from the command line/terminal, instead of hardcoding them inside the code.

```java
public class Greet {
    public static void main(String[] args) {
        System.out.println("Hello, " + args[0]);
    }
}
```
If you run: `java Greet Raj`, the output will be: `Hello, Raj`

### Q61. What type is `args` in `public static void main(String[] args)`?
**Answer:** `args` is an **array of Strings** (`String[]`). Every value you type after the class name on the command line becomes one item in this array.

### Q62. What happens if you try to access `args[0]` but no arguments were given?
**Answer:** You get an `ArrayIndexOutOfBoundsException`, because the `args` array would be empty (length 0), and there is no index 0 to access.

### Q63. Can command line arguments be numbers? How do you use them as numbers?
**Answer:** Command line arguments always arrive as **Strings**, even if they look like numbers. To use them as numbers, you must convert them yourself, using methods like `Integer.parseInt()`.
```java
public static void main(String[] args) {
    int num = Integer.parseInt(args[0]);   // convert String to int
    System.out.println(num + 10);
}
```

---

## Section L: Arrays

### Q64. What is an Array in Java?
**Answer:** An array is a container that holds a FIXED number of values, of the SAME data type, in one variable.
```java
int[] numbers = {10, 20, 30, 40, 50};
```

### Q65. How do you declare and create an array in Java?
**Answer:**
```java
int[] numbers;                        // declaration
numbers = new int[5];                 // creation - array of size 5
int[] numbers2 = new int[]{1, 2, 3};  // declaration + creation + initialization together
int[] numbers3 = {1, 2, 3};           // shortcut form
```

### Q66. How do you access elements of an array?
**Answer:** Using an index number, starting from `0`.
```java
int[] numbers = {10, 20, 30};
System.out.println(numbers[0]);   // 10
System.out.println(numbers[2]);   // 30
```

### Q67. What happens if you access an array index that doesn't exist?
**Answer:** Java throws an `ArrayIndexOutOfBoundsException` at runtime.
```java
int[] numbers = {10, 20, 30};   // valid indices: 0, 1, 2
System.out.println(numbers[5]); // ArrayIndexOutOfBoundsException
```

### Q68. Can Java arrays hold different data types together?
**Answer:** No, a normal array can only hold ONE data type. All elements of `int[]` must be `int`. If you need to store different types together, you would use an `Object[]` array (holding everything as `Object`), or better, use a class to group different types together.

### Q69. What is a multi-dimensional array? Give an example.
**Answer:** It is an "array of arrays" — commonly used to represent things like grids or tables.
```java
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};
System.out.println(matrix[1][2]);   // 6 (row 1, column 2)
```

### Q70. How do you find the length of an array?
**Answer:** Using the `.length` property (NOT a method — no parentheses).
```java
int[] numbers = {10, 20, 30};
System.out.println(numbers.length);   // 3
```

### Q71. Is array size fixed in Java? Can you resize it later?
**Answer:** Yes, array size is **fixed** once it is created — you cannot make it bigger or smaller afterward. If you need a resizable list-like structure, use `ArrayList` from the Collections framework instead.

### Q72. What is the default value of array elements after creation, before you set them?
**Answer:** Same as the default values for that data type:
- Numeric types (`int`, `double`, etc.) → `0` or `0.0`
- `boolean` → `false`
- `char` → `'\u0000'`
- Reference types (like `String[]`) → `null`

```java
int[] numbers = new int[3];
System.out.println(numbers[0]);   // 0 (default value)
```

---

## Section M: Strings

### Q73. What is a String in Java?
**Answer:** A String is a sequence of characters, used to represent text. In Java, `String` is actually a CLASS, not a primitive type, even though it's used very often like one.
```java
String name = "Hello";
```

### Q74. Why is String called immutable in Java?
**Answer:** Immutable means "cannot be changed" after creation. Once a String object is created, its content can never be modified. If you try to "change" a String, Java actually creates a brand NEW String object behind the scenes, instead of changing the original one.
```java
String s = "Hello";
s.concat(" World");        // this does NOT change s
System.out.println(s);      // still prints "Hello"

s = s.concat(" World");    // now s points to a NEW String object
System.out.println(s);      // prints "Hello World"
```

### Q75. Why did Java make String immutable? What are the benefits?
**Answer:**
- **Security** — Strings are used everywhere (file paths, network addresses, database URLs); if they could change unexpectedly, it could cause serious security problems
- **String Pool optimization** — since Strings never change, Java can safely let many variables share the same String object, saving memory
- **Thread safety** — an immutable object can be safely shared between multiple threads, since nothing can change it
- **Safe as HashMap keys** — since a String's content and hash code never change, it makes a reliable key for hash-based collections

### Q76. What is the String Pool?
**Answer:** It is a special memory area (inside the Heap) where Java stores String literal values. If you create the same text using a literal more than once, Java reuses the SAME object, instead of creating a new one every time — this saves memory.
```java
String s1 = "Hello";
String s2 = "Hello";
System.out.println(s1 == s2);   // true - both point to the same pooled object
```

### Q77. What is the difference between `String s1 = "Hello"` and `String s2 = new String("Hello")`?
**Answer:** `"Hello"` (literal) checks the String Pool first, and reuses an existing object if the same text already exists there. `new String("Hello")` FORCES the creation of a brand new object in the regular Heap, even if the same text already exists in the pool.
```java
String s1 = "Hello";
String s2 = new String("Hello");
System.out.println(s1 == s2);        // false - different objects
System.out.println(s1.equals(s2));   // true - same content
```

### Q78. What is the difference between `String`, `StringBuilder`, and `StringBuffer`?
**Answer:**

| String | StringBuilder | StringBuffer |
|--------|-----------------|-----------------|
| Immutable | Mutable (can be changed) | Mutable (can be changed) |
| Not thread-safe by nature (but safe since it never changes) | Not thread-safe | Thread-safe (synchronized) |
| Slower for repeated changes (creates new objects) | Faster for repeated changes | Slower than StringBuilder, due to thread-safety checks |

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");
System.out.println(sb);   // "Hello World" - same object was changed
```

### Q79. Why should you use `StringBuilder` instead of `String` when doing many concatenations, like inside a loop?
**Answer:** Because each time you "change" a `String` (like using `+` to combine text), Java secretly creates a BRAND NEW String object, and throws away the old one. If you do this many times inside a loop, it wastes a lot of memory and time, creating many unused objects along the way. `StringBuilder` changes its OWN internal content directly, without creating new objects each time, making it much faster for repeated changes.

```java
// BAD - creates many unnecessary String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;   // creates a new String object every single time!
}

// GOOD - much faster
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);   // modifies the same object
}
```

### Q80. What are some commonly used String methods?
**Answer:**
| Method | What it does |
|--------|----------------|
| `length()` | returns the number of characters |
| `charAt(index)` | returns the character at a given position |
| `substring(start, end)` | returns part of the String |
| `toUpperCase()` / `toLowerCase()` | changes text case |
| `trim()` | removes extra spaces from both ends |
| `equals()` | compares content of two Strings |
| `equalsIgnoreCase()` | compares content, ignoring uppercase/lowercase |
| `split(regex)` | breaks a String into an array, based on a separator |
| `replace(old, new)` | replaces parts of the String |
| `contains(text)` | checks if a String contains certain text |
| `indexOf(text)` | finds the position of certain text |

---

## Section N: Wrapper Classes

### Q81. What are Wrapper Classes in Java?
**Answer:** Wrapper Classes are special classes that let you use primitive data types (`int`, `char`, `boolean`, etc.) AS OBJECTS. Every primitive type has a matching wrapper class.

| Primitive | Wrapper Class |
|-----------|-----------------|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

### Q82. Why do we need Wrapper Classes? Isn't a primitive type enough?
**Answer:** Wrapper Classes are needed because:
- Java Collections (like `ArrayList`, `HashMap`) can only store OBJECTS, not primitive values directly
- Wrapper classes provide useful built-in methods (like `Integer.parseInt()` to convert String to int)
- They allow a value to be `null`, which primitives cannot represent
- They are needed for certain features like Generics, which only work with objects, not primitives

```java
ArrayList<int> list = new ArrayList<>();       // ERROR - not allowed
ArrayList<Integer> list = new ArrayList<>();   // correct - uses wrapper class
```

### Q83. How do you convert a primitive to its Wrapper class, and back again?
**Answer:**
```java
int a = 10;
Integer obj = Integer.valueOf(a);   // primitive to wrapper (boxing)
int b = obj.intValue();              // wrapper to primitive (unboxing)
```

### Q84. What is the difference between `Integer.parseInt()` and `Integer.valueOf()`?
**Answer:**
- `Integer.parseInt(String)` converts a String into a primitive `int`
- `Integer.valueOf(String)` converts a String into an `Integer` OBJECT (wrapper)

```java
int x = Integer.parseInt("100");     // returns primitive int
Integer y = Integer.valueOf("100");  // returns Integer object
```

---

## Section O: Autoboxing & Unboxing

### Q85. What is Autoboxing?
**Answer:** Autoboxing is when Java **automatically** converts a primitive value into its matching Wrapper class object, without you writing any conversion code yourself.
```java
int a = 10;
Integer obj = a;   // Autoboxing - Java automatically wraps 10 into an Integer object
```

### Q86. What is Unboxing?
**Answer:** Unboxing is the opposite — Java **automatically** converts a Wrapper class object back into its primitive value.
```java
Integer obj = 20;
int b = obj;   // Unboxing - Java automatically extracts the primitive value 20
```

### Q87. When did Autoboxing and Unboxing get introduced in Java?
**Answer:** These features were introduced in **Java 5**. Before that, you had to manually convert between primitives and wrapper objects yourself, using methods like `.intValue()` or `Integer.valueOf()`.

### Q88. Can Autoboxing/Unboxing cause a `NullPointerException`? How?
**Answer:** Yes! If a Wrapper object is `null`, and Java tries to unbox it into a primitive, it will throw a `NullPointerException`, because there is no actual value to unwrap.
```java
Integer obj = null;
int x = obj;   // NullPointerException - cannot unbox null into a primitive
```

### Q89. Does Autoboxing affect performance?
**Answer:** Yes, slightly. Autoboxing/Unboxing creates extra wrapper objects behind the scenes, which takes extra memory and processing time compared to using primitives directly. This matters most inside loops that run many times, where repeated autoboxing can slow things down noticeably.

### Q90. What is a common tricky mistake with Wrapper class comparison using `==`?
**Answer:** Java caches small Integer wrapper values (from -128 to 127) for performance. This means comparing small cached values with `==` might accidentally seem to "work," while larger values will NOT behave the same way — this is a common and confusing bug.
```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b);   // true - both use the SAME cached object (small value)

Integer c = 200;
Integer d = 200;
System.out.println(c == d);   // false - these are DIFFERENT objects (outside cache range)
```
**Best practice:** always use `.equals()` to compare Wrapper class objects, never `==`.

---

## Section P: Enums

### Q91. What is an Enum in Java?
**Answer:** `enum` is a special data type used to define a FIXED set of constant values. For example, days of the week, or directions (NORTH, SOUTH, EAST, WEST) never change, so an enum fits perfectly.
```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

### Q92. How do you use an enum in your code?
**Answer:**
```java
Day today = Day.MONDAY;
System.out.println(today);   // MONDAY
```

### Q93. Why use enums instead of just plain constants like `int` or `String`?
**Answer:**
- **Type safety** — the compiler makes sure you can only use valid, defined values; you cannot accidentally assign an invalid value
- **Readability** — `Day.MONDAY` is much clearer than a plain number like `1`
- Enums can have their OWN fields, constructors, and methods, giving them extra power
- Enums work well with `switch` statements

### Q94. Can an enum have fields, constructors, and methods?
**Answer:** Yes!
```java
enum Level {
    LOW(1), MEDIUM(2), HIGH(3);

    private int value;
    Level(int value) {
        this.value = value;
    }
    public int getValue() {
        return value;
    }
}
System.out.println(Level.HIGH.getValue());   // 3
```

### Q95. What are `ordinal()` and `name()` methods in an enum?
**Answer:**
- `ordinal()` returns the POSITION of the value, starting from 0
- `name()` returns the exact NAME of the constant, as a String

```java
Day today = Day.WEDNESDAY;
System.out.println(today.ordinal());   // 2 (0=MONDAY, 1=TUESDAY, 2=WEDNESDAY)
System.out.println(today.name());      // WEDNESDAY
```

### Q96. Can you use an enum inside a `switch` statement?
**Answer:** Yes, this is a very common use case.
```java
switch (today) {
    case MONDAY: System.out.println("Start of week"); break;
    case FRIDAY: System.out.println("Almost weekend"); break;
    default: System.out.println("Regular day");
}
```

---

## Section Q: Packages

### Q97. What is a Package in Java?
**Answer:** A package is a way to GROUP related classes and interfaces together, similar to how folders organize files on your computer.
```java
package com.mycompany.app;
```

### Q98. Why do we use packages?
**Answer:**
- **Avoids naming conflicts** — two classes can have the same name, as long as they are in different packages
- **Better organization** — related classes are grouped together, making code easier to find and manage
- **Access control** — the "default" access level works at the package level
- **Reusability** — packages can be reused across different projects

### Q99. What is the difference between a built-in package and a user-defined package?
**Answer:**
- **Built-in packages** already come with Java, like `java.util` (collections), `java.io` (input/output), `java.lang` (basic classes, automatically available without import)
- **User-defined packages** are created by developers themselves, to organize their own project's classes

### Q100. How do you import a class from another package?
**Answer:** Using the `import` keyword at the top of your file.
```java
import java.util.ArrayList;   // importing a single class
import java.util.*;           // importing everything inside java.util (not usually best practice)
```

### Q101. Do you need to import classes from `java.lang`?
**Answer:** No. The `java.lang` package (which contains basic classes like `String`, `System`, `Math`, `Object`) is automatically imported into every Java program — you never need to write `import java.lang.String;` yourself.

---

## Section R: Access Modifiers

### Q102. What are Access Modifiers in Java?
**Answer:** Access Modifiers control WHO can access a class, method, or variable — from the same class, same package, subclasses, or from anywhere.

### Q103. What are the 4 access modifiers in Java?
**Answer:**

| Modifier | Same Class | Same Package | Subclass (different package) | Different Package |
|----------|:---:|:---:|:---:|:---:|
| `private` | Yes | No | No | No |
| default (no keyword) | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes | No |
| `public` | Yes | Yes | Yes | Yes |

### Q104. What is the "default" access modifier?
**Answer:** If you don't write ANY access modifier at all, Java gives it the **default** (also called "package-private") access level. This means it can only be accessed by code within the SAME package — not from outside, even by subclasses in different packages.
```java
class Demo {   // no modifier written = default access
    int value;   // default access - only visible within the same package
}
```

### Q105. Can a top-level class be declared `private` or `protected`?
**Answer:** No. A top-level class (a class not nested inside another class) can only be `public` or default (no modifier) — NOT `private` or `protected`. Only nested/inner classes can use all 4 access modifiers.

### Q106. Simple way to remember all 4 access modifiers?
**Answer:**
- `private` = "Only me"
- default = "Only my neighborhood (same package)"
- `protected` = "My neighborhood, plus my children (subclasses), wherever they live"
- `public` = "Everyone, everywhere"

---

## Section S: Tricky/Mixed Questions

### Q107. Why does Java have both primitive types AND wrapper classes for the same thing?
**Answer:** Primitives are fast and memory-efficient, good for everyday calculations. Wrapper classes exist because certain features (Collections, Generics, allowing `null`) only work with objects, not primitives. Java keeps both, so you get speed when you need it, and object features when you need those instead.

### Q108. Why do local variables need to be initialized before use, but instance/static variables don't?
**Answer:** Instance and static variables automatically get default values (`0`, `false`, `null`, etc.) when memory is set up for them. Local variables do NOT get this automatic treatment — Java forces you to set a value yourself, mainly to avoid accidental bugs from using an unintended, forgotten value.

### Q109. Is a `String` array (`String[]`) the same as an `ArrayList<String>`?
**Answer:** No. A `String[]` array has a FIXED size once created, and cannot grow or shrink. An `ArrayList<String>` can grow and shrink dynamically, and comes with many built-in helper methods (`add()`, `remove()`, `contains()`, etc.), making it more flexible for most real-world use cases.

### Q110. What is the difference between compile-time errors and runtime errors, using a simple example?
**Answer:**
- A **compile-time error** happens BEFORE the program runs, usually due to a mistake in code structure, like a missing semicolon or wrong data type. Example: `int x = "Hello";` fails to compile.
- A **runtime error/exception** happens WHILE the program is running, like dividing by zero, or accessing an invalid array index. Example: `int result = 10 / 0;` compiles fine, but crashes when it actually runs.

---

**That's a complete, simple-English Q&A on Java Fundamentals.** Go through each section, try writing small test programs yourself for the trickier answers (like Wrapper class caching, or pass-by-value with objects) — actually SEEING the output is the best way to make these concepts stick permanently.
