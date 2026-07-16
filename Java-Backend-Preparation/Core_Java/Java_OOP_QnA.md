# Java OOP — Interview Q&A Format (With Clear Explanations)

> Every question is answered with a simple explanation, and code examples wherever needed. Read top to bottom like a tutorial, or jump to any question directly.

---

## Table of Contents
- [Section A: Basics of OOP](#section-a-basics-of-oop)
- [Section B: Class & Object](#section-b-class--object)
- [Section C: Encapsulation](#section-c-encapsulation)
- [Section D: Inheritance](#section-d-inheritance)
- [Section E: Polymorphism](#section-e-polymorphism)
- [Section F: Abstraction](#section-f-abstraction)
- [Section G: Constructors](#section-g-constructors)
- [Section H: this & super](#section-h-this--super)
- [Section I: Static & Final](#section-i-static--final)
- [Section J: Access Modifiers & Packages](#section-j-access-modifiers--packages)
- [Section K: Object Class Methods](#section-k-object-class-methods)
- [Section L: Relationships (Association/Aggregation/Composition)](#section-l-relationships)
- [Section M: Inner Classes & Enums](#section-m-inner-classes--enums)
- [Section N: Tricky/Advanced Questions](#section-n-trickyadvanced-questions)

---

## Section A: Basics of OOP

### Q1. What is Object-Oriented Programming (OOP)?
**Answer:** OOP is a programming style where we organize code around **objects** — real-world entities that combine **data** (attributes/fields) and **behavior** (methods) into one unit called a **class**. Instead of writing a program as a sequence of instructions (like in procedural programming), we model it as a collection of interacting objects, similar to how things work in the real world (a Car, a Student, a BankAccount, etc.).

### Q2. Why do we use OOP instead of procedural programming?
**Answer:**
- **Modularity:** Code is organized into self-contained classes, making large programs easier to manage
- **Reusability:** Once a class is written, it can be reused via inheritance or by creating multiple objects
- **Maintainability:** Changes to one class don't usually break unrelated code
- **Security:** Encapsulation hides internal data from outside interference
- **Real-world mapping:** Easier to design software that mirrors real-world systems

### Q3. What are the main pillars/principles of OOP?
**Answer:** There are 4 main pillars:
1. **Encapsulation** — bundling data and methods together, hiding internal details
2. **Inheritance** — reusing code from a parent class in a child class
3. **Polymorphism** — one interface, many implementations/forms
4. **Abstraction** — hiding implementation complexity, showing only essential features

### Q4. Is Java a fully object-oriented language?
**Answer:** No, Java is **not 100% purely object-oriented**. This is because Java still uses **primitive data types** (`int`, `char`, `boolean`, `double`, etc.) which are NOT objects. A truly pure OOP language (like Smalltalk) treats absolutely everything — even numbers — as objects. Java uses primitives directly for performance reasons, though it provides **Wrapper classes** (`Integer`, `Character`, `Boolean`) to treat them as objects when needed (this is called **Autoboxing/Unboxing**).

---

## Section B: Class & Object

### Q5. What is a class?
**Answer:** A class is a **blueprint or template** for creating objects. It defines what properties (fields) and behaviors (methods) the objects created from it will have. A class itself doesn't hold real data — it's just a design.

```java
class Car {
    String color;
    void drive() { System.out.println("Driving..."); }
}
```

### Q6. What is an object?
**Answer:** An object is a **real instance** of a class, created in memory using the `new` keyword. Each object has its own copy of instance variables (unless they are `static`), and can call the methods defined in its class.

```java
Car myCar = new Car();  // myCar is an object
myCar.color = "Red";
myCar.drive();
```

### Q7. What is the difference between a class and an object?
**Answer:**

| Class | Object |
|-------|--------|
| Blueprint/template | Real instance created from the blueprint |
| Declared using the `class` keyword | Created using the `new` keyword |
| Doesn't occupy memory for instance data by itself | Occupies memory (in the Heap) |
| Logical entity | Physical entity |
| You can create many objects from one class | Each object can have different data values |

### Q8. Can we create an object without using the `new` keyword?
**Answer:** Yes, there are a few other ways:
- Using **reflection**: `Class.forName("ClassName").newInstance()` or `Constructor.newInstance()`
- Using **`clone()`** method (creates a copy of an existing object)
- Using **deserialization** (converting a saved byte stream back into an object)
- String literals like `String s = "Hello";` also create an object without explicit `new` (it's fetched/created in the String Pool internally)

---

## Section C: Encapsulation

### Q9. What is Encapsulation? Explain with an example.
**Answer:** Encapsulation means wrapping data (variables) and the methods that operate on that data into a single unit (a class), and **restricting direct access** to the internal data from outside. In Java, this is done by declaring fields as `private` and providing `public` getter/setter methods to access or modify them in a controlled way.

```java
class BankAccount {
    private double balance;  // hidden - cannot be accessed directly from outside

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) balance += amount;   // validation - controlled modification
    }
}
```

Without encapsulation, code like `account.balance = -1000;` could corrupt data directly. With encapsulation, all changes go through `deposit()`, which can validate the input.

### Q10. Why is Encapsulation important?
**Answer:**
- **Data hiding/security:** Sensitive data isn't exposed directly
- **Validation control:** You can add checks before changing data (like preventing negative balances)
- **Flexibility:** You can change the internal implementation later without breaking code that uses the class (as long as the public methods stay the same)
- **Reduced coupling:** Other classes depend on the public interface, not internal details

### Q11. How do you achieve encapsulation in Java?
**Answer:**
1. Declare the class's instance variables as `private`
2. Provide `public` getter methods to read the values
3. Provide `public` setter methods to update the values (optionally with validation logic)

A fully encapsulated class is often called a **POJO (Plain Old Java Object)** or a **JavaBean** when it strictly follows this getter/setter pattern.

---

## Section D: Inheritance

### Q12. What is Inheritance? Give an example.
**Answer:** Inheritance allows a class (**child/subclass**) to acquire the fields and methods of another class (**parent/superclass**), using the `extends` keyword. It represents an **"IS-A"** relationship.

```java
class Animal {
    void eat() { System.out.println("Eating..."); }
}
class Dog extends Animal {
    void bark() { System.out.println("Barking..."); }
}
// Dog IS-A Animal, so Dog inherits eat()
```

### Q13. Why do we use Inheritance?
**Answer:**
- **Code reusability:** Avoid rewriting the same code in multiple classes
- **Method Overriding is possible:** Enables runtime polymorphism
- **Logical hierarchy:** Represents real-world "IS-A" relationships naturally
- **Extensibility:** Add new functionality to existing classes without modifying them

### Q14. What are the types of inheritance supported in Java?
**Answer:**
| Type | Description | Supported with classes? |
|------|-------------|:---:|
| Single | One class inherits from one parent | Yes |
| Multilevel | A chain: A → B → C | Yes |
| Hierarchical | Multiple children from one parent: A → B, A → C | Yes |
| Multiple | One child inherits from 2+ parent classes directly | **No** |
| Multiple (via interfaces) | A class implements multiple interfaces | Yes |

### Q15. Why doesn't Java support multiple inheritance with classes?
**Answer:** To avoid the **Diamond Problem**. If class C extends both class A and class B, and both A and B have a method with the exact same signature, the compiler wouldn't know which version C should inherit — this creates ambiguity. Java avoids this entirely by only allowing a class to `extends` ONE other class. However, multiple inheritance IS allowed through **interfaces**, because Java forces the implementing class to explicitly resolve conflicts if two interfaces have default methods with the same signature.

### Q16. Can a subclass access the private members of its parent class?
**Answer:** No. `private` members are only accessible within the class they are declared in — not even subclasses can access them directly. The subclass can only access `public`, `protected`, or default (package-level, if in the same package) members of the parent.

### Q17. What is the difference between `extends` and `implements`?
**Answer:**
- `extends` is used when a class inherits from another **class** (or when an interface extends another interface)
- `implements` is used when a class implements an **interface**
- A class can `extends` only ONE class, but can `implements` MULTIPLE interfaces

---

## Section E: Polymorphism

### Q18. What is Polymorphism? What are its types?
**Answer:** Polymorphism means "many forms" — the same method name behaves differently depending on the context. There are two types:
1. **Compile-time Polymorphism (Method Overloading)** — resolved at compile time
2. **Runtime Polymorphism (Method Overriding)** — resolved at runtime

### Q19. What is Method Overloading? Give an example.
**Answer:** Method Overloading means having **multiple methods with the same name but different parameter lists** (different number, type, or order of parameters) **within the same class**.

```java
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
}
```

The compiler decides WHICH version to call based on the arguments you pass — this happens at **compile time**, so it's also called **static binding**.

### Q20. Can we overload methods by changing only the return type?
**Answer:** **No.** If two methods have the exact same name and exact same parameter list, but different return types, this will cause a **compile-time error**, because the compiler cannot decide which method to call based on return type alone (return type isn't part of the "method signature" used for overload resolution).

```java
int add(int a, int b) { return a + b; }
double add(int a, int b) { return a + b; }  // ERROR: duplicate method
```

### Q21. What is Method Overriding? Give an example.
**Answer:** Method Overriding means a **subclass provides its own specific implementation** of a method that is already defined in its **parent class**, with the **exact same method signature** (name + parameters).

```java
class Animal {
    void sound() { System.out.println("Some generic sound"); }
}
class Cat extends Animal {
    @Override
    void sound() { System.out.println("Meow"); }
}
```

This is resolved at **runtime**, based on the actual object type — called **dynamic binding** or **dynamic method dispatch**.

### Q22. What rules must be followed for Method Overriding?
**Answer:**
- Method name and parameter list must be EXACTLY the same
- Return type must be the same, or a **covariant type** (a subtype of the original return type)
- Access modifier in the child class **cannot be more restrictive** than in the parent (e.g., if parent method is `protected`, child can make it `protected` or `public`, but NOT `private`)
- The method cannot be `static`, `final`, or `private` in the parent (these cannot be overridden)
- Checked exceptions thrown by the overriding method cannot be broader than those thrown by the parent's method

### Q23. What is Dynamic Method Dispatch?
**Answer:** It's the mechanism by which a call to an overridden method is resolved **at runtime**, rather than compile time, based on the actual object being referred to (not the reference type).

```java
Animal a = new Cat();  // reference type: Animal, object type: Cat
a.sound();              // calls Cat's sound() - decided at RUNTIME
```

Even though the reference `a` is of type `Animal`, Java looks at what the actual object is (`Cat`) when the program runs, and calls the appropriate overridden method.

### Q24. Difference between Method Overloading and Method Overriding?
**Answer:**

| Overloading | Overriding |
|-------------|------------|
| Same class | Parent-child classes |
| Different parameter list | Same parameter list |
| Resolved at compile time (static binding) | Resolved at runtime (dynamic binding) |
| Return type can differ | Return type must be same/covariant |
| Increases readability | Provides specific behavior in subclass |
| Not related to inheritance | Requires inheritance |

### Q25. Can we override a `static` method?
**Answer:** No. Static methods belong to the **class**, not to instances, so they are resolved at **compile time** based on the reference type — not the actual object. If a subclass defines a static method with the same signature as the parent, this is called **method hiding**, NOT overriding.

```java
class Parent {
    static void show() { System.out.println("Parent static"); }
}
class Child extends Parent {
    static void show() { System.out.println("Child static"); }
}
Parent p = new Child();
p.show();  // Output: "Parent static" (decided by reference type, NOT actual object!)
```

### Q26. Can constructors be overloaded? Can they be overridden?
**Answer:** Constructors **CAN be overloaded** (multiple constructors with different parameter lists in the same class). But constructors **CANNOT be overridden**, because they are not inherited by subclasses in the first place — each class must define its own constructors.

---

## Section F: Abstraction

### Q27. What is Abstraction?
**Answer:** Abstraction means **hiding the internal implementation details and exposing only the essential/relevant features** to the user. The user knows WHAT a method does, but not HOW it does it internally.

**Real-life example:** When you use a TV remote, you press a button to change the channel. You don't need to know the internal circuitry that makes it work — that complexity is hidden (abstracted away).

### Q28. How is Abstraction achieved in Java?
**Answer:** Using:
1. **Abstract classes** (partial abstraction — 0 to 100%)
2. **Interfaces** (traditionally full abstraction, though Java 8+ allows some implementation via default/static methods)

### Q29. What is an abstract class?
**Answer:** A class declared with the `abstract` keyword. It:
- **Cannot be instantiated directly** (you can't do `new AbstractClass()`)
- Can have both **abstract methods** (no body, must be implemented by subclasses) and **concrete methods** (with a body)
- Can have constructors, instance variables, static methods

```java
abstract class Shape {
    abstract double area();               // abstract method - no body
    void display() {                       // concrete method - has body
        System.out.println("This is a shape");
    }
}
```

### Q30. Can an abstract class have a constructor? Why, if it can't be instantiated?
**Answer:** **Yes**, an abstract class CAN have a constructor. Even though you can't create an object of the abstract class directly, its constructor still runs whenever a **subclass object** is created — because the subclass constructor implicitly (or explicitly) calls `super()`, which triggers the abstract class's constructor. This is useful for initializing common fields shared by all subclasses.

### Q31. Difference between Abstract Class and Interface?
**Answer:**

| Abstract Class | Interface |
|-----------------|-----------|
| Can have abstract AND concrete methods | Traditionally only abstract methods (Java 8+ allows default/static/private methods with body) |
| Can have instance variables of any type | Variables are always `public static final` (constants) |
| Supports constructors | No constructors |
| A class can extend only ONE abstract class | A class can implement MULTIPLE interfaces |
| Use `extends` | Use `implements` |
| Can have any access modifier for methods | Methods are `public` by default |
| Represents an "IS-A" relationship with shared code | Represents a "CAN-DO" capability/contract |

### Q32. When should you use an abstract class vs an interface?
**Answer:**
- Use an **abstract class** when multiple related classes share common code/state, and there's a strong "IS-A" relationship (e.g., `Dog`, `Cat` both `extends Animal`)
- Use an **interface** when you want to define a capability/contract that can apply to unrelated classes (e.g., `Comparable`, `Runnable`, `Flyable` — a `Bird` and an `Airplane` are unrelated, but both `CAN fly`)

### Q33. Can an interface have method implementations?
**Answer:** Before Java 8: No, all methods were implicitly `public abstract`. Since **Java 8**: Interfaces can have `default` methods (with a body, providing a default implementation) and `static` methods. Since **Java 9**: Interfaces can also have `private` methods (used internally to share code between default methods).

```java
interface Flyable {
    void fly();  // abstract
    default void land() {   // default method - has a body
        System.out.println("Landing...");
    }
}
```

---

## Section G: Constructors

### Q34. What is a constructor?
**Answer:** A constructor is a special block of code, similar to a method, that is automatically called when an object is created using `new`. It's used to **initialize the object's state** (assign initial values to fields).

**Rules:**
- Must have the SAME name as the class
- Has NO return type (not even `void`)
- Can be overloaded

### Q35. What is a default constructor?
**Answer:** If you don't write ANY constructor in your class, Java automatically provides a **no-argument default constructor** that initializes fields to their default values (`0`, `null`, `false`, etc.). However, the moment you write even ONE constructor yourself (of any kind), Java stops providing this automatic default constructor.

### Q36. What is Constructor Overloading?
**Answer:** Having multiple constructors in the same class, with different parameter lists — just like method overloading.

```java
class Student {
    Student() { }                          // no-arg constructor
    Student(String name) { }               // one parameter
    Student(String name, int age) { }      // two parameters
}
```

### Q37. What is Constructor Chaining?
**Answer:** Calling one constructor from another constructor within the same class (using `this(...)`), OR calling the parent class constructor from a child class constructor (using `super(...)`).

```java
class Student {
    String name;
    int age;
    Student() {
        this("Unknown", 0);   // chains to the parameterized constructor
    }
    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

**Important rule:** `this()` or `super()` must always be the **first statement** in a constructor.

### Q38. Can a constructor be `private`? What's the use case?
**Answer:** Yes. A private constructor prevents other classes from creating instances of this class using `new` from outside. This is commonly used in the **Singleton Design Pattern**, where you want to ensure only ONE instance of a class exists throughout the application.

```java
class Singleton {
    private static Singleton instance;
    private Singleton() { }  // private constructor

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

---

## Section H: this & super

### Q39. What is the `this` keyword used for?
**Answer:**
1. To refer to the **current object's** instance variable, especially when a parameter has the same name:
   ```java
   Student(String name) { this.name = name; }
   ```
2. To call another constructor in the same class (constructor chaining): `this(...)`
3. To pass the current object as an argument to another method
4. To return the current object from a method (used in method chaining / builder patterns)

### Q40. What is the `super` keyword used for?
**Answer:**
1. To call the **parent class's constructor**: `super(...)` — must be the first line in the child constructor
2. To call a **parent class's method** that has been overridden in the child class: `super.methodName()`
3. To access a **parent class's variable** that is hidden/shadowed by a child class variable of the same name

```java
class Animal {
    void sound() { System.out.println("Some sound"); }
}
class Dog extends Animal {
    void sound() {
        super.sound();   // calls Animal's sound() first
        System.out.println("Bark");
    }
}
```

### Q41. What happens if you don't explicitly call `super()` in a child class constructor?
**Answer:** Java **automatically inserts** a call to the parent's **no-argument constructor** as the very first statement. If the parent class does NOT have a no-argument constructor (only parameterized ones), this will cause a **compile-time error**, and you must explicitly call `super(appropriate arguments)`.

---

## Section I: Static & Final

### Q42. What does the `static` keyword mean?
**Answer:** `static` means the member (variable, method, or block) belongs to the **class itself**, not to any individual object. There is only ONE copy shared across ALL objects of that class. Static members can be accessed without creating an object (using `ClassName.member`).

```java
class Counter {
    static int count = 0;   // shared by all objects
    Counter() { count++; }
}
```

### Q43. Can a static method access non-static (instance) variables?
**Answer:** **No, not directly.** A static method belongs to the class and runs without any specific object existing, so it cannot directly access instance variables (which belong to specific objects). It would need an actual object reference to access instance variables.

### Q44. What is a static block? When does it run?
**Answer:** A static block (`static { ... }`) runs **once**, automatically, when the class is first **loaded into memory** by the JVM — even before the `main()` method runs. It's typically used to initialize static variables with complex logic.

```java
class Config {
    static int value;
    static {
        value = 100;  // complex initialization logic here
        System.out.println("Static block executed");
    }
}
```

### Q45. What does the `final` keyword mean when applied to different things?
**Answer:**

| Applied to | Meaning |
|------------|---------|
| Variable | Value can be assigned only once; becomes a constant afterward |
| Method | Cannot be overridden by any subclass |
| Class | Cannot be extended/inherited (no subclasses allowed) |

```java
final int MAX_USERS = 100;      // constant
class Parent {
    final void show() { }        // cannot be overridden
}
final class Utility { }          // cannot be extended (e.g., String, Integer are final classes)
```

### Q46. Why is the `String` class `final` in Java?
**Answer:** Mainly for **security, immutability, and performance**:
- Strings are heavily used (in class loading, network connections, DB URLs, etc.), and if `String` could be extended/overridden, malicious code could create a subclass that behaves unexpectedly, causing security risks
- Immutability allows safe sharing across threads and safe use as `HashMap` keys (since the hash code never changes)
- Enables the **String Pool** optimization, since Strings are guaranteed to never change

---

## Section J: Access Modifiers & Packages

### Q47. What are the 4 access modifiers in Java, and what do they mean?
**Answer:**

| Modifier | Same Class | Same Package | Subclass (diff. package) | Different Package |
|----------|:---:|:---:|:---:|:---:|
| `private` | Yes | No | No | No |
| default (no keyword) | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes | No |
| `public` | Yes | Yes | Yes | Yes |

### Q48. What is a package in Java, and why do we use it?
**Answer:** A package is a way of **grouping related classes and interfaces together**, similar to a folder structure. Benefits:
- Avoids naming conflicts (two classes with the same name can exist in different packages)
- Better code organization
- Provides package-level access control (default modifier)
- Makes large codebases easier to navigate

```java
package com.mycompany.app;
import java.util.List;   // importing from a built-in package
```

---

## Section K: Object Class Methods

### Q49. What is the `Object` class in Java?
**Answer:** `Object` is the **root/parent class of every class** in Java — every class implicitly extends `Object`, even if you don't write `extends Object` explicitly. It provides some default methods that all objects inherit, like `toString()`, `equals()`, `hashCode()`, `getClass()`, `clone()`, `wait()`, `notify()`.

### Q50. Difference between `==` and `.equals()`?
**Answer:**
- `==` checks **reference equality** — whether two variables point to the exact SAME object in memory (for objects; for primitives, it checks value equality)
- `.equals()` checks **logical/content equality** by default, `Object`'s `.equals()` behaves the same as `==`, but classes like `String` OVERRIDE it to compare actual content

```java
String s1 = new String("Hi");
String s2 = new String("Hi");
System.out.println(s1 == s2);        // false - different objects in memory
System.out.println(s1.equals(s2));   // true - same content
```

### Q51. If you override `equals()`, why must you also override `hashCode()`?
**Answer:** Because of this contract: **"If two objects are equal according to `equals()`, they MUST have the same `hashCode()`."** Hash-based collections like `HashMap` and `HashSet` use `hashCode()` first to find the right "bucket", and then use `equals()` to confirm the exact match within that bucket. If you override `equals()` without `hashCode()`, two "equal" objects might end up with different hash codes, land in different buckets, and the collection will fail to find/recognize them correctly — causing subtle, hard-to-find bugs.

### Q52. What does `toString()` do, and why do we override it?
**Answer:** By default, `Object`'s `toString()` returns something like `ClassName@1b6d3586` (class name + hashcode in hex) — not very useful for debugging. We override it to provide a meaningful, readable String representation of the object.

```java
class Student {
    String name;
    @Override
    public String toString() { return "Student: " + name; }
}
System.out.println(new Student());   // calls toString() automatically -> "Student: ..."
```

---

## Section L: Relationships

### Q53. What is Association?
**Answer:** A general relationship where one class **uses** another class, without strong ownership. It can be one-to-one, one-to-many, many-to-many.
```java
class Teacher { }
class Student {
    Teacher teacher;   // Student is associated with a Teacher
}
```

### Q54. What is Aggregation? Give an example.
**Answer:** Aggregation is a special, **weaker** form of association representing a "HAS-A" relationship, where the contained object can exist **independently** of the container.
```java
class Department {
    List<Professor> professors;   // Department HAS professors, but professors can exist without this dept
}
```

### Q55. What is Composition? Give an example.
**Answer:** Composition is a **stronger** form of "HAS-A" relationship, where the contained object **CANNOT exist independently** of the container — if the container is destroyed, the contained object is destroyed too.
```java
class House {
    private Room room = new Room();  // Room is created and destroyed along with House
}
```

### Q56. Difference between Aggregation and Composition?
**Answer:**

| Aggregation | Composition |
|-------------|--------------|
| Weak ownership | Strong ownership |
| Contained object can exist independently | Contained object cannot exist independently |
| Example: Library HAS Books (books can exist elsewhere) | Example: House HAS Rooms (rooms don't exist without the house) |
| Represented as a "shared" relationship | Represented as a "part-of/exclusive" relationship |

### Q57. What is "Favor Composition over Inheritance"? Why is this advice given?
**Answer:** This is a well-known OOP design principle. It suggests that instead of using inheritance (extending a class) to reuse code, you should often prefer composing objects together (having a class contain a reference to another class). This is because:
- Inheritance creates **tight coupling** — changes to a parent class can unexpectedly break child classes
- Java doesn't allow multiple inheritance with classes, but composition allows combining multiple behaviors freely
- Composition is more flexible — behavior can be changed at runtime by swapping the composed object

---

## Section M: Inner Classes & Enums

### Q58. What is an inner class? What types exist?
**Answer:** A class defined inside another class. Types:
1. **Member Inner Class** — a regular non-static class inside another class; needs an instance of the outer class to be created
2. **Static Nested Class** — declared `static`; does NOT need an outer class instance
3. **Local Inner Class** — defined inside a method body
4. **Anonymous Inner Class** — a class without a name, defined and instantiated in one step, often used for one-time-use implementations (like event listeners or interface implementations)

```java
interface Greeting { void sayHello(); }
Greeting g = new Greeting() {         // anonymous inner class
    public void sayHello() { System.out.println("Hi!"); }
};
```

### Q59. What is an enum in Java?
**Answer:** `enum` is a special data type representing a **fixed set of constants**. It's more powerful than plain `int` or `String` constants because enums are type-safe, can have fields, constructors, and methods.

```java
enum Level {
    LOW(1), MEDIUM(2), HIGH(3);
    private final int value;
    Level(int value) { this.value = value; }
    public int getValue() { return value; }
}
```

### Q60. Why use enums instead of plain constant integers?
**Answer:**
- **Type-safety:** You can't accidentally pass an invalid value (the compiler checks it)
- **Readability:** `Level.HIGH` is clearer than a magic number like `3`
- Enums can have their own **fields, constructors, and methods** attached
- Enums work well with `switch` statements

---

## Section N: Tricky/Advanced Questions

### Q61. Can we instantiate an abstract class?
**Answer:** No, directly you cannot do `new AbstractClass()`. However, you CAN create an **anonymous subclass** on the fly that implements all abstract methods:
```java
abstract class Shape { abstract void draw(); }
Shape s = new Shape() {           // anonymous subclass implementing draw()
    void draw() { System.out.println("Drawing"); }
};
```

### Q62. Can an interface extend multiple interfaces?
**Answer:** Yes! Unlike classes (which can only `extends` one class), an interface CAN `extends` multiple other interfaces.
```java
interface A { }
interface B { }
interface C extends A, B { }   // valid
```

### Q63. What happens if a class implements two interfaces that both have a default method with the same signature?
**Answer:** This causes a **compile-time error** due to ambiguity. The implementing class MUST override that method explicitly to resolve the conflict — often calling one of the specific interface's version using `InterfaceName.super.methodName()`.

### Q64. What is the diamond problem, and how does Java handle it for interfaces?
**Answer:** The diamond problem occurs when a class inherits from two sources that both define the same method — the compiler doesn't know which version to use. For classes, Java avoids this entirely by disallowing multiple class inheritance. For interfaces (with default methods, Java 8+), Java forces the implementing class to explicitly override and resolve the conflict, removing the ambiguity.

### Q65. Is it possible to have a constructor in an interface?
**Answer:** No. Interfaces cannot have constructors because they cannot be instantiated on their own — they only define a contract/capability that implementing classes must fulfill.

### Q66. What is the difference between "IS-A" and "HAS-A" relationships?
**Answer:**
- **IS-A** = Inheritance relationship (e.g., `Dog IS-A Animal`) — achieved via `extends`/`implements`
- **HAS-A** = Composition/Aggregation relationship (e.g., `Car HAS-A Engine`) — achieved by having a class hold a reference to another class as a field

### Q67. Can we override a `private` method?
**Answer:** No. Private methods are **not visible** to subclasses at all, so they cannot be overridden. If a subclass defines a method with the exact same signature as a private method in the parent, it's treated as an entirely NEW, independent method — not an override.

### Q68. What is the purpose of the `@Override` annotation? Is it mandatory?
**Answer:** `@Override` tells the compiler that this method is INTENDED to override a parent class/interface method. It's **not mandatory**, but it's a best practice, because if you make a typo in the method signature (like a wrong parameter type), the compiler will catch it and give an error — without `@Override`, your "overriding" method would silently become a new overloaded method instead, and the bug could go unnoticed.

### Q69. Can a class implement an interface without implementing all its methods?
**Answer:** Only if the class itself is declared `abstract`. A concrete (non-abstract) class MUST implement all abstract methods of every interface it implements — otherwise it's a compile-time error.

### Q70. What is a Marker Interface? Give an example.
**Answer:** A marker interface is an interface with **NO methods at all** — it's used purely to "tag" or "mark" a class as having a certain property, which other code (often via reflection) can check for. Examples in Java: `Serializable`, `Cloneable`, `Remote`.

```java
class Employee implements Serializable { }  // marks Employee as serializable
```

---

**That's a comprehensive Q&A covering Java OOP from fundamentals to advanced/tricky interview questions.** Go through each section, try to explain the answers in your own words, and write small code snippets yourself to lock in the understanding.
