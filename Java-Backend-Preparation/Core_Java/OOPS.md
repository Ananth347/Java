Parameters: Parameters are the variables declared in the method definition.

Arguments: Arguments are the actual values passed to the method when calling it.

```java
public class Main {

    public static void greet(String name) { // Parameter
        System.out.println("Hello " + name);
    }

    public static void main(String[] args) {
        greet("Ananth"); // Argument
    }
}
```
Explicitly: Manually have to write
Implicitly: Automatically




# OOPS (Object-Oriented Programming System)

## Definition
Object-Oriented Programming (OOP) is a programming paradigm that organizes software around objects rather than functions.
An object represents a real-world entity that contains:

- State (Data/Attributes)
- Behavior (Methods)

Examples:
- Car
- Mobile
- Bank Account
- Employee
- Student

OOP helps in:
- Code Reusability
- Maintainability
- Scalability
- Security
- Flexibility

---

## Why OOPS?

Before OOP, procedural programming was commonly used.

Problems with Procedural Programming:
- Code duplication
- Difficult maintenance
- Poor scalability
- High dependency between modules

OOP solves these problems using:

1. Encapsulation
2. Inheritance
3. Polymorphism
4. Abstraction

Benefits of OOP:

- Reusable code
- Better software design
- Easy maintenance
- Increased security
- Better modularity
- Easy testing

---

## Real World Example

Consider a Banking Application.

Objects:
- Customer
- Account
- Transaction
- ATM Card

Example:

Customer:
- Name
- Mobile Number
- Address

Behavior:
- Deposit Money
- Withdraw Money
- Check Balance

Each customer becomes an object.

```java
class Customer {

    String name;
    String mobile;

    void deposit() {}

    void withdraw() {}
}
```

---

## Class and Object

### Class

A class is a blueprint used to create objects.

Example:

```java
class Employee {

    String name;
    double salary;

    void work() {
        System.out.println("Employee Working");
    }
}
```

### Object

An object is an instance of a class.

```java
Employee emp1 = new Employee();
Employee emp2 = new Employee();
```

Object contains:

- State
- Behavior
- Identity

---

# Four Pillars of OOP

1. Encapsulation
2. Inheritance
3. Polymorphism
4. Abstraction

---

## Encapsulation

### Definition

Encapsulation is the process of wrapping data and methods into a single unit and restricting direct access to data.

Data is protected using:

- private variables
- public methods

Encapsulation = Data Hiding

---

### Example

```java
class BankAccount {

    private double balance;

    public void deposit(double amount) {
        balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

Usage:

```java
BankAccount account = new BankAccount();

account.deposit(1000);

System.out.println(account.getBalance());
```

---

### Real World Example

ATM Machine

Visible:
- Withdraw
- Deposit
- Check Balance

Hidden:
- Database Queries
- Internal Calculations
- Security Logic

---

### Advantages

- Data Security
- Better Control
- Loose Coupling
- Maintainability

---

### Interview Questions

Q: What is Encapsulation?

Q: How is Encapsulation achieved in Java?

Q: Why do we use private variables?

Q: Difference between Data Hiding and Encapsulation?

---

## Inheritance

### Definition

Inheritance allows one class to acquire properties and methods of another class.

Relationship:

IS-A Relationship

Example:

Car IS-A Vehicle

Dog IS-A Animal

SavingsAccount IS-A Account

---

### Example

```java
class Vehicle {

    void start() {
        System.out.println("Vehicle Started");
    }
}

class Car extends Vehicle {

}
```

Usage:

```java
Car car = new Car();

car.start();
```

Output:

Vehicle Started

---

### Types of Inheritance

#### Single Inheritance

```text
A -> B
```

#### Multilevel Inheritance

```text
A -> B -> C
```

#### Hierarchical Inheritance

```text
      A
     / \
    B   C
```

#### Multiple Inheritance

Not supported through classes.

Supported through Interfaces.

---

### Advantages

- Code Reusability
- Reduced Duplication
- Easier Maintenance

---

### Interview Questions

Q: What is Inheritance?

Q: Why Multiple Inheritance is not supported in Java?

Q: Difference between IS-A and HAS-A Relationship?

Q: Types of Inheritance?

---

## Polymorphism

### Definition

Polymorphism means:

One Interface, Many Forms

It allows the same method to behave differently based on the object.

---

### Compile Time Polymorphism

Method Overloading

Same method name with different parameters.

Example:

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

Resolved during Compilation.

---

### Runtime Polymorphism

Method Overriding

Child class provides its own implementation.

Example:

```java
class Animal {

    void sound() {
        System.out.println("Animal Sound");
    }
}

class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Bark");
    }
}
```

Usage:

```java
Animal animal = new Dog();

animal.sound();
```

Output:

Bark

---

### Real World Example

Payment Gateway

```java
PaymentService payment =
        new CreditCardPayment();
```

Later:

```java
payment =
        new UpiPayment();
```

Same interface.

Different implementation.

---

### Interview Questions

Q: What is Polymorphism?

Q: Difference between Overloading and Overriding?

Q: What is Runtime Polymorphism?

Q: Can static methods be overridden?

---

## Abstraction

### Definition

Abstraction means hiding implementation details and exposing only essential functionality.

Focus on:

"What to do"

instead of

"How it is done"

---

### Achieved Using

1. Abstract Class
2. Interface

---

### Example

```java
abstract class Vehicle {

    abstract void start();
}
```

```java
class Car extends Vehicle {

    @Override
    void start() {
        System.out.println("Car Started");
    }
}
```

---

### Interface Example

```java
interface PaymentService {

    void pay();
}
```

```java
class UpiPayment
        implements PaymentService {

    public void pay() {
        System.out.println("UPI Payment");
    }
}
```

---

### Real World Example

UPI Application

Visible:
- Pay Button

Hidden:
- Encryption
- Validation
- Banking APIs
- Security Logic

---

### Advantages

- Reduced Complexity
- Better Security
- Flexible Design
- Easy Maintenance

---

### Interview Questions

Q: What is Abstraction?

Q: Difference between Interface and Abstract Class?

Q: Can an Abstract Class have Constructor?

Q: Can Interface have Method Implementation?

---

## Association

Relationship between independent objects.

Example:

Employee works in Company.

```text
Employee ---- Company
```

---

## Aggregation

Weak HAS-A Relationship.

Example:

Department HAS-A Employee

Employees can exist independently.

```text
Department HAS-A Employee
```

---

## Composition

Strong HAS-A Relationship.

Example:

House HAS-A Room

Room cannot exist independently.

```text
House HAS-A Room
```

---

## Composition vs Inheritance

Inheritance:

```text
Car IS-A Vehicle
```

Composition:

```text
Car HAS-A Engine
```

Interview Rule:

Favor Composition Over Inheritance

Reasons:
- Flexible
- Less Coupling
- Easy Maintenance

---

## OOP in Spring Boot

Encapsulation:
- DTO
- Entity

Inheritance:
- BaseEntity
- CommonResponse

Polymorphism:
- Service Interfaces
- Strategy Pattern

Abstraction:
- Repository Interfaces
- Service Interfaces

---

## Quick Revision

Class → Blueprint

Object → Instance of Class

Encapsulation → Hide Data

Inheritance → Reuse Code

Polymorphism → One Interface, Many Forms

Abstraction → Hide Implementation

Association → Uses Relationship

Aggregation → Weak HAS-A

Composition → Strong HAS-A

---

## Frequently Asked Questions

1. What are the four pillars of OOP?

2. Difference between Abstraction and Encapsulation?

3. Difference between Overloading and Overriding?

4. Why Multiple Inheritance is not supported in Java?

5. What is Runtime Polymorphism?

6. What is Dynamic Method Dispatch?

7. What is IS-A Relationship?

8. What is HAS-A Relationship?

9. Composition vs Aggregation?

10. Composition vs Inheritance?

11. Can an Interface have default methods?

12. Can an Abstract Class have constructors?

13. Why is Polymorphism heavily used in Spring Boot?

14. Real-world examples of Encapsulation?

15. Real-world examples of Abstraction?
