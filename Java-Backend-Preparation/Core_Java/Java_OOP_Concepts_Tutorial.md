# Java OOP (Object-Oriented Programming) — Complete Beginner-Friendly Tutorial

> Written in the simplest English possible, with real-life examples for every concept.

---

## Table of Contents
1. What is OOP and why do we need it?
2. Class and Object
3. The 4 Pillars of OOP
   - Encapsulation
   - Inheritance
   - Polymorphism
   - Abstraction
4. Constructors
5. `this` keyword
6. `super` keyword
7. Method Overloading vs Method Overriding
8. Access Modifiers
9. Static vs Instance (non-static)
10. `final` keyword
11. Abstract Class vs Interface
12. Packages
13. `Object` class — `equals()`, `hashCode()`, `toString()`
14. Association, Aggregation, Composition
15. Inner Classes
16. Enums
17. Common Interview Questions (with short answers)

---

## 1. What is OOP and why do we need it?

OOP means **Object-Oriented Programming**. It is a way of writing code where we think about the real world in terms of **objects** — just like in real life we have a "Car", a "Dog", a "Student", etc.

Before OOP, programmers used **Procedural Programming** (like in C), where a program is just a list of steps/functions. This becomes messy when programs grow large.

OOP solves this by grouping **data (variables)** and **behavior (functions)** together into a single unit called a **Class**.

**Why OOP?**
- Easier to manage large programs
- Code reusability (write once, use many times)
- Easier to maintain and update
- Matches how we think about the real world
- Better security through hiding data (Encapsulation)

**Real-life analogy:** Think of a "Car" as a blueprint. Every car has properties (color, brand, speed) and behaviors (start, stop, accelerate). OOP lets you create this blueprint once (Class) and then make many actual cars (Objects) from it.

---

## 2. Class and Object

### Class
A **class** is a blueprint or template. It does not take memory by itself (mostly) — it just defines what properties and behaviors an object will have.

```java
class Car {
    // properties (also called fields/attributes/instance variables)
    String color;
    String brand;
    int speed;

    // behavior (also called method)
    void startEngine() {
        System.out.println("Engine started!");
    }
}
```

### Object
An **object** is a real instance created from the class. It actually takes memory.

```java
public class Main {
    public static void main(String[] args) {
        Car myCar = new Car();   // 'myCar' is an object of class Car
        myCar.color = "Red";
        myCar.brand = "Toyota";
        myCar.startEngine();     // Output: Engine started!
    }
}
```

**Simple way to remember:**
- Class = Recipe
- Object = Actual dish cooked using that recipe

You can make **many objects** from one class, and each object can have different values.

```java
Car car1 = new Car();
Car car2 = new Car();
car1.color = "Red";
car2.color = "Blue";
```

---

## 3. The 4 Pillars of OOP

These 4 concepts are the **heart** of OOP. Interviewers almost always ask about these.

---

### 3.1 Encapsulation

**Definition:** Wrapping data (variables) and code (methods) together into a single unit (class), and **restricting direct access** to some of the object's components.

In simple words: **Hide the internal details, and only allow controlled access using methods.**

**How to achieve it in Java:**
1. Make variables `private`
2. Provide `public` getter and setter methods to access/modify them

```java
class BankAccount {
    private double balance;   // hidden from outside world

    // getter
    public double getBalance() {
        return balance;
    }

    // setter (with validation - this is the real power of encapsulation)
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        } else {
            System.out.println("Invalid amount!");
        }
    }
}
```

Without encapsulation, anyone could do `account.balance = -5000;` directly, which is dangerous. With encapsulation, we control HOW the balance changes.

**Real-life analogy:** Think of a capsule (medicine). The medicine (data) is hidden inside a cover, and you interact with it in a controlled way.

**Benefits:**
- Data hiding / security
- Control over data through validation
- Flexibility to change internal implementation without affecting other code

---

### 3.2 Inheritance

**Definition:** A mechanism where one class (child/subclass) acquires the properties and behaviors of another class (parent/superclass).

**Why?** To reuse code and establish a natural "IS-A" relationship.

```java
class Animal {
    void eat() {
        System.out.println("This animal eats food");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Dog barks");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog d = new Dog();
        d.eat();   // inherited from Animal
        d.bark();  // Dog's own method
    }
}
```

Here, `Dog` **IS-A** `Animal`. That's the test for inheritance — if you can say "X is a Y", inheritance fits.

**Types of Inheritance in Java:**
| Type | Example | Supported directly in Java? |
|------|---------|------------------------------|
| Single | A → B | Yes |
| Multilevel | A → B → C | Yes |
| Hierarchical | A → B, A → C | Yes |
| Multiple (via classes) | A, B → C | **No** (not allowed with classes) |
| Multiple (via interfaces) | Interface A, B → Class C | Yes |

**Why doesn't Java support multiple inheritance with classes?**
To avoid the **Diamond Problem** — if class C inherits from both A and B, and both A and B have the same method, Java won't know which one to call. Java avoids this confusion by disallowing multiple class inheritance, but allows it through interfaces (since Java 8, using default methods, this is handled cleanly).

**Important keyword:** `extends` is used for inheriting a class. `implements` is used for inheriting an interface.

---

### 3.3 Polymorphism

**Definition:** "Poly" = many, "morph" = forms. It means **one thing behaving in many forms.**

There are two types:

#### a) Compile-time Polymorphism (Method Overloading)
Same method name, different parameters, decided at **compile time**.

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }
    double add(double a, double b) {
        return a + b;
    }
    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

#### b) Runtime Polymorphism (Method Overriding)
Child class provides a specific implementation of a method already defined in the parent class. Decided at **runtime**, based on the actual object type.

```java
class Animal {
    void sound() {
        System.out.println("Animal makes a sound");
    }
}

class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("Cat meows");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal a = new Cat();  // Parent reference, Child object
        a.sound();  // Output: Cat meows (decided at RUNTIME)
    }
}
```

This is called **Dynamic Method Dispatch** — the JVM decides at runtime which `sound()` to call, based on the actual object (`Cat`), not the reference type (`Animal`).

**Real-life analogy:** A person can be a "Teacher" in a school, a "Father" at home, and a "Customer" at a shop — same person, different behavior (forms) depending on context.

---

### 3.4 Abstraction

**Definition:** Hiding the internal implementation details and showing only the essential features to the user.

In simple words: **Show WHAT it does, hide HOW it does it.**

**Real-life analogy:** When you drive a car, you use the steering wheel, brake, and accelerator. You don't need to know how the engine internally works. That complexity is hidden (abstracted) from you.

In Java, abstraction is achieved using:
1. **Abstract classes**
2. **Interfaces**

```java
abstract class Shape {
    abstract double area();   // no body - just declaration

    void display() {
        System.out.println("This is a shape");
    }
}

class Circle extends Shape {
    double radius = 5;

    @Override
    double area() {
        return 3.14 * radius * radius;
    }
}
```

The user of `Circle` just calls `area()`. They don't need to know the formula internally used.

**Encapsulation vs Abstraction (common confusion):**
| Encapsulation | Abstraction |
|---------------|-------------|
| Hides data (internal state) | Hides implementation (internal logic) |
| Achieved using private fields + getters/setters | Achieved using abstract classes/interfaces |
| Focus: "How to protect data" | Focus: "What to show to the user" |

---

## 4. Constructors

A **constructor** is a special method used to **initialize objects**. It runs automatically when an object is created using `new`.

**Rules:**
- Same name as the class
- No return type (not even `void`)
- Can be overloaded

```java
class Student {
    String name;
    int age;

    // Default constructor
    Student() {
        name = "Unknown";
        age = 0;
    }

    // Parameterized constructor
    Student(String n, int a) {
        name = n;
        age = a;
    }
}

public class Main {
    public static void main(String[] args) {
        Student s1 = new Student();             // uses default constructor
        Student s2 = new Student("Rahul", 20);   // uses parameterized constructor
    }
}
```

**Note:** If you don't write ANY constructor, Java automatically provides a free **default constructor**. But the moment you write even one constructor yourself, Java stops providing the free one.

**Constructor Overloading** = having multiple constructors with different parameter lists (like method overloading).

**Constructor Chaining:** Calling one constructor from another using `this()`.

```java
class Student {
    String name;
    int age;

    Student() {
        this("Unknown", 0);   // calls the parameterized constructor
    }

    Student(String n, int a) {
        name = n;
        age = a;
    }
}
```

---

## 5. `this` keyword

`this` refers to the **current object**. Common uses:

1. To differentiate instance variables from parameters with the same name:
```java
class Student {
    String name;
    Student(String name) {
        this.name = name;   // this.name = instance variable, name = parameter
    }
}
```

2. To call another constructor of the same class (constructor chaining) — `this(...)`
3. To pass the current object as a parameter to another method
4. To return the current object from a method

---

## 6. `super` keyword

`super` refers to the **immediate parent class**. Common uses:

1. To call the parent class constructor:
```java
class Animal {
    Animal() {
        System.out.println("Animal constructor");
    }
}
class Dog extends Animal {
    Dog() {
        super();   // calls Animal's constructor - must be the FIRST line
        System.out.println("Dog constructor");
    }
}
```

2. To call a parent class method that has been overridden:
```java
class Animal {
    void sound() {
        System.out.println("Some sound");
    }
}
class Dog extends Animal {
    void sound() {
        super.sound();  // calls Animal's version first
        System.out.println("Bark");
    }
}
```

3. To access a parent class variable that is hidden by a child class variable of the same name.

**Important:** If you don't explicitly call `super()`, Java automatically inserts a call to the parent's no-argument constructor as the first line.

---

## 7. Method Overloading vs Method Overriding

This is one of the MOST asked interview questions. Know it perfectly.

| Feature | Overloading | Overriding |
|---------|-------------|------------|
| Meaning | Same method name, different parameters | Same method name & same parameters, in parent-child classes |
| Class | Happens within the SAME class | Happens between PARENT and CHILD class |
| Binding | Compile-time (Static Binding) | Runtime (Dynamic Binding) |
| Return type | Can be different | Must be same or covariant (a subtype) |
| Access Modifier | Can be different | Cannot be more restrictive than parent's method |
| `@Override` annotation | Not used | Recommended to use |
| Purpose | Increase readability, handle different input types | Provide specific implementation in child class |

```java
// Overloading example
class MathUtils {
    int square(int x) { return x * x; }
    double square(double x) { return x * x; }
}

// Overriding example
class Parent {
    void greet() { System.out.println("Hello from Parent"); }
}
class Child extends Parent {
    @Override
    void greet() { System.out.println("Hello from Child"); }
}
```

---

## 8. Access Modifiers

Access modifiers control **who can access** a class, method, or variable.

| Modifier | Same Class | Same Package | Subclass (different package) | Different Package |
|----------|:---:|:---:|:---:|:---:|
| `private` | Yes | No | No | No |
| default (no modifier) | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes | No |
| `public` | Yes | Yes | Yes | Yes |

```java
public class Example {
    private int a;      // only inside this class
    int b;               // default - only same package
    protected int c;     // same package + subclasses anywhere
    public int d;         // accessible from anywhere
}
```

**Simple memory trick:**
- `private` = "Only me"
- default = "Only my neighborhood (package)"
- `protected` = "My neighborhood + my children (subclasses)"
- `public` = "Everyone"

---

## 9. Static vs Instance (Non-static)

### Static
- Belongs to the **class**, not to any specific object
- Shared by ALL objects — there's only ONE copy in memory
- Can be accessed without creating an object
- Loaded into memory when the class is loaded (into an area called **Method Area / Metaspace**)

```java
class Counter {
    static int count = 0;  // shared across all objects

    Counter() {
        count++;
    }
}

public class Main {
    public static void main(String[] args) {
        new Counter();
        new Counter();
        new Counter();
        System.out.println(Counter.count);  // Output: 3
    }
}
```

### Instance (non-static)
- Belongs to a specific **object**
- Each object has its OWN copy
- Needs an object to be accessed

**Static methods:**
- Can only directly access other static members
- Cannot use `this` or `super`
- Common example: `main()` method is always `static` because JVM calls it without creating an object first

**Static block:** Runs once, when the class is loaded, before `main()`.
```java
class Demo {
    static {
        System.out.println("Static block runs once when class loads");
    }
}
```

---

## 10. `final` keyword

`final` can be applied to variables, methods, and classes — but it means slightly different things each time:

| Applied to | Meaning |
|------------|---------|
| Variable | Value cannot be changed once assigned (becomes a constant) |
| Method | Cannot be overridden by a subclass |
| Class | Cannot be extended/inherited (no subclasses allowed) |

```java
final int MAX = 100;         // final variable - constant

class Parent {
    final void show() {}     // final method - cannot be overridden
}

final class Utility {}       // final class - cannot be inherited (e.g., String class is final)
```

---

## 11. Abstract Class vs Interface

Both are used for **abstraction**, but they are different.

### Abstract Class
```java
abstract class Vehicle {
    abstract void start();          // abstract method - no body
    void fuel() {                    // concrete method - has body
        System.out.println("Filling fuel");
    }
}
```
- Can have both abstract AND concrete (regular) methods
- Can have constructors
- Can have instance variables of any access type
- A class can extend only ONE abstract class (single inheritance)
- Use `extends`

### Interface
```java
interface Flyable {
    void fly();   // implicitly public and abstract (before Java 8)
}
```
- Traditionally, only abstract methods (before Java 8)
- Since Java 8: can have `default` and `static` methods with body
- Since Java 9: can have `private` methods too
- All variables are implicitly `public static final` (constants)
- No constructors
- A class can implement MULTIPLE interfaces (achieves multiple inheritance)
- Use `implements`

```java
interface Flyable {
    void fly();
    default void land() {   // default method (Java 8+)
        System.out.println("Landing safely");
    }
}

class Bird implements Flyable {
    public void fly() {
        System.out.println("Bird is flying");
    }
}
```

### When to use which?
- Use **abstract class** when classes share common code/state and have a strong "IS-A" relationship
- Use **interface** when you want to define a "CAN-DO" capability that unrelated classes can implement (e.g., `Flyable`, `Comparable`, `Runnable`)

| Feature | Abstract Class | Interface |
|---------|----------------|-----------|
| Methods | Abstract + Concrete | Abstract, default, static, private (Java 8+) |
| Variables | Any type | `public static final` only |
| Multiple Inheritance | No | Yes |
| Constructor | Yes | No |
| Keyword | `extends` | `implements` |
| Access Modifiers on methods | Any | `public` by default |

---

## 12. Packages

A **package** is a folder-like structure used to group related classes/interfaces together (like namespaces).

```java
package com.mycompany.myapp;

import java.util.ArrayList;   // importing a class from java.util package

public class Main {
    // code
}
```

**Benefits:**
- Avoids naming conflicts (two classes with the same name in different packages are fine)
- Better code organization
- Access control (default access modifier works at package level)

**Types:**
- **Built-in packages**: `java.util`, `java.io`, `java.lang` (auto-imported), etc.
- **User-defined packages**: Created by developers for their own projects

---

## 13. `Object` class — `equals()`, `hashCode()`, `toString()`

Every class in Java **implicitly extends** the `Object` class (even if you don't write `extends Object`). This gives every object some default behavior.

### `toString()`
Returns a String representation of the object. By default, it prints something like `ClassName@hashcode`, which is not very useful — so we usually override it.

```java
class Student {
    String name;
    Student(String name) { this.name = name; }

    @Override
    public String toString() {
        return "Student: " + name;
    }
}
```

### `equals()`
By default, `equals()` checks **reference equality** (are both variables pointing to the exact same object in memory?) — same as `==`. We override it to check **logical/content equality**.

```java
class Student {
    String name;
    Student(String name) { this.name = name; }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Student)) return false;
        Student s = (Student) obj;
        return this.name.equals(s.name);
    }
}
```

### `hashCode()`
Returns an integer hash value for the object, used by hash-based collections like `HashMap`, `HashSet`.

**Golden rule:** If you override `equals()`, you MUST override `hashCode()` too. Why? Because two equal objects (by `equals()`) MUST have the same `hashCode()`, or hash-based collections (like `HashMap`) will behave incorrectly (you may not find an object you stored, because it's looking in the wrong "bucket").

```java
@Override
public int hashCode() {
    return name.hashCode();
}
```

---

## 14. Association, Aggregation, Composition

These describe relationships between classes.

### Association
A general relationship where one class uses another. Can be one-to-one, one-to-many, etc. No ownership implied.
```java
class Teacher {}
class Student {
    Teacher teacher;  // Student is associated with Teacher
}
```

### Aggregation (HAS-A, weak relationship)
One class contains another, but both can exist independently. It's a "part-of" relationship, but the "part" can survive without the "whole".
```java
class Department {
    List<Professor> professors;  // Department HAS professors
    // if department closes, professors still exist elsewhere
}
```

### Composition (HAS-A, strong relationship)
One class contains another, and the contained object **cannot exist independently**. If the parent is destroyed, the child is destroyed too.
```java
class House {
    private Room room = new Room();  // Room cannot exist without House
}
```

**Simple difference:** Aggregation = weak ownership (Library HAS Books, but books can exist without that specific library). Composition = strong ownership (House HAS Rooms, rooms cannot exist without the house).

---

## 15. Inner Classes

A class defined inside another class.

**Types:**
1. **Member Inner Class** — regular class inside another class
2. **Static Nested Class** — declared `static`, doesn't need an instance of the outer class
3. **Local Inner Class** — defined inside a method
4. **Anonymous Inner Class** — a class without a name, used for one-time use (commonly with interfaces)

```java
class Outer {
    class Inner {                     // member inner class
        void show() { System.out.println("Inner class"); }
    }
    static class StaticNested {       // static nested class
        void display() { System.out.println("Static nested class"); }
    }
}

public class Main {
    public static void main(String[] args) {
        Outer outer = new Outer();
        Outer.Inner inner = outer.new Inner();   // needs outer instance
        inner.show();

        Outer.StaticNested nested = new Outer.StaticNested();  // no outer instance needed
        nested.display();
    }
}
```

**Anonymous inner class example:**
```java
interface Greeting {
    void sayHello();
}

public class Main {
    public static void main(String[] args) {
        Greeting g = new Greeting() {
            public void sayHello() {
                System.out.println("Hello!");
            }
        };
        g.sayHello();
    }
}
```

---

## 16. Enums

`enum` is a special data type used to define a fixed set of constants.

```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

public class Main {
    public static void main(String[] args) {
        Day today = Day.MONDAY;
        System.out.println(today);          // MONDAY
        System.out.println(today.ordinal()); // 0 (position, zero-indexed)
    }
}
```

Enums can also have fields, constructors, and methods:
```java
enum Level {
    LOW(1), MEDIUM(2), HIGH(3);

    private int value;
    Level(int value) { this.value = value; }
    public int getValue() { return value; }
}
```

**Why use enums instead of plain constants (like `int`)?**
- Type-safety (you can't accidentally assign an invalid value)
- More readable code
- Can have their own methods and behavior

---

## 17. Common Interview Questions (Quick Answers)

**Q1. Why is Java called "partially" object-oriented and not "fully"?**
Because Java uses primitive data types (`int`, `char`, `boolean`, etc.) which are not objects — a fully OOP language would treat everything as an object.

**Q2. Can a constructor be `private`?**
Yes — used in the **Singleton design pattern**, to prevent other classes from creating new instances.

**Q3. Can we override a `static` method?**
No. Static methods belong to the class, not the object, so they are resolved at compile-time (this is called **method hiding**, not overriding).

**Q4. Can we overload the `main()` method?**
Yes, but JVM will always call the exact signature: `public static void main(String[] args)`.

**Q5. What is the Diamond Problem?**
When a class inherits from two classes that both have a method with the same signature, the compiler doesn't know which one to use. Java avoids this with classes but handles it in interfaces using explicit override rules.

**Q6. Difference between `==` and `.equals()`?**
`==` checks reference equality (same memory address) for objects. `.equals()` checks logical/content equality (can be overridden).

**Q7. Can an abstract class have a constructor?**
Yes! Even though you can't create an object of an abstract class directly, its constructor runs when a subclass object is created (via `super()`).

**Q8. What is the difference between `String`, `StringBuilder`, and `StringBuffer`?**
- `String` is **immutable** (once created, cannot change)
- `StringBuilder` is **mutable**, **not thread-safe**, faster
- `StringBuffer` is **mutable**, **thread-safe** (synchronized), slower

**Q9. What is method hiding?**
When a subclass defines a static method with the same signature as a static method in the parent class. It's resolved at compile time based on reference type (unlike overriding).

**Q10. Can an interface extend another interface?**
Yes, using `extends` (and an interface can even extend multiple interfaces, unlike classes).

---

## Summary Cheat Sheet

| Concept | One-line meaning |
|---------|-------------------|
| Class | Blueprint of an object |
| Object | Instance of a class |
| Encapsulation | Hide data, expose via getters/setters |
| Inheritance | Reuse parent class code (IS-A) |
| Polymorphism | One name, many forms |
| Abstraction | Show what, hide how |
| Constructor | Initializes an object |
| `this` | Refers to current object |
| `super` | Refers to parent class |
| Overloading | Same method name, different parameters, same class |
| Overriding | Same method signature, parent-child classes |
| `static` | Belongs to class, shared by all objects |
| `final` | Cannot be changed/overridden/extended |
| Abstract class | Partial abstraction, single inheritance |
| Interface | Full abstraction (traditionally), multiple inheritance |

---

**Congratulations! You now know Java OOP from the ground up.** Practice by writing small programs for each concept — that's the fastest way to make it permanent.
