# Design Principles & Design Patterns — Simple Tutorial

> Written in simple English, with easy examples in Java. Read top to bottom like a tutorial.

---

## Table of Contents

**Part 1: Design Principles**
1. What are Design Principles?
2. SOLID Principles
   - S — Single Responsibility Principle
   - O — Open/Closed Principle
   - L — Liskov Substitution Principle
   - I — Interface Segregation Principle
   - D — Dependency Inversion Principle
3. DRY (Don't Repeat Yourself)
4. KISS (Keep It Simple, Stupid)
5. YAGNI (You Aren't Gonna Need It)
6. Composition over Inheritance

**Part 2: Design Patterns**
7. What are Design Patterns?
8. Singleton Pattern
9. Factory Pattern
10. Builder Pattern
11. Prototype Pattern
12. Adapter Pattern
13. Strategy Pattern
14. Observer Pattern
15. Decorator Pattern
16. Proxy Pattern
17. Summary Cheat Sheet

---

# Part 1: Design Principles

## 1. What are Design Principles?

Design Principles are simple **guidelines** that help you write better code. They are not strict rules you must follow every time — they are more like good habits that make your code easier to understand, change, and fix later.

Think of them like advice from an experienced friend: "Don't make this class do too much," or "Don't repeat the same code everywhere." Following these ideas early saves you a lot of pain later, when your project grows bigger.

---

## 2. SOLID Principles

**SOLID** is not one rule — it is 5 rules put together. Each letter in "SOLID" stands for one principle. These 5 rules were put together to help you write code that is easy to maintain and easy to extend, without breaking old code.

```
S - Single Responsibility Principle
O - Open/Closed Principle
L - Liskov Substitution Principle
I - Interface Segregation Principle
D - Dependency Inversion Principle
```

---

### S — Single Responsibility Principle (SRP)

**Simple meaning:** A class should have only **ONE reason to change**. In other words, a class should do only ONE job.

**Why?** If a class does many jobs, then changing one job might accidentally break another job. Keeping each class focused on one thing makes your code safer to change.

**Bad Example (breaks SRP):**
```java
class Employee {
    void calculateSalary() { /* salary logic */ }
    void saveToDatabase() { /* database logic */ }
    void printReport() { /* printing logic */ }
}
```
This class does THREE different jobs: salary calculation, saving data, and printing. If the database logic changes, you have to touch this same class — even though salary logic didn't change at all.

**Good Example (follows SRP):**
```java
class Employee {
    void calculateSalary() { /* salary logic */ }
}
class EmployeeRepository {
    void save(Employee e) { /* database logic */ }
}
class EmployeeReportPrinter {
    void print(Employee e) { /* printing logic */ }
}
```
Now each class has only ONE job, and one reason to change.

---

### O — Open/Closed Principle (OCP)

**Simple meaning:** A class should be **open for extension**, but **closed for modification**.

This means: you should be able to ADD new behavior to your code, WITHOUT changing the existing, already-working code.

**Bad Example (breaks OCP):**
```java
class DiscountCalculator {
    double calculate(String customerType, double amount) {
        if (customerType.equals("Regular")) {
            return amount * 0.9;
        } else if (customerType.equals("Premium")) {
            return amount * 0.8;
        }
        // every time a new customer type is added, this method must be changed again!
        return amount;
    }
}
```
Every time a NEW customer type is added, you must go back and EDIT this method again, risking breaking the old logic.

**Good Example (follows OCP):**
```java
interface Discount {
    double apply(double amount);
}
class RegularDiscount implements Discount {
    public double apply(double amount) { return amount * 0.9; }
}
class PremiumDiscount implements Discount {
    public double apply(double amount) { return amount * 0.8; }
}
// Adding a new discount type means creating a NEW class - no need to touch old code!
class VIPDiscount implements Discount {
    public double apply(double amount) { return amount * 0.7; }
}
```
Now, to add a new discount type, you just add a NEW class. The old classes are never touched.

---

### L — Liskov Substitution Principle (LSP)

**Simple meaning:** If class `B` is a subclass of class `A`, then you should be able to use an object of `B` anywhere an object of `A` is expected, WITHOUT breaking anything.

In simple words: a child class should behave in a way that doesn't surprise or break the code that uses the parent class.

**Bad Example (breaks LSP):**
```java
class Bird {
    void fly() { System.out.println("Flying"); }
}
class Ostrich extends Bird {
    void fly() {
        throw new UnsupportedOperationException("Ostrich can't fly!");
        // this breaks the expectation that all Birds can fly
    }
}
```
Since `Ostrich` is a `Bird`, code expects it to fly like any other Bird. But it throws an error instead — this SURPRISES and BREAKS the code using it.

**Good Example (follows LSP):**
```java
class Bird { }
class FlyingBird extends Bird {
    void fly() { System.out.println("Flying"); }
}
class Ostrich extends Bird {
    void walk() { System.out.println("Walking"); }
    // Ostrich doesn't claim to fly, so no broken promise
}
```
Now the class design correctly separates "birds that fly" from "birds that don't," so no false promises are made.

---

### I — Interface Segregation Principle (ISP)

**Simple meaning:** Don't force a class to implement methods it doesn't need. It's better to have many SMALL, specific interfaces than one BIG, general interface.

**Bad Example (breaks ISP):**
```java
interface Worker {
    void work();
    void eat();
}
class Robot implements Worker {
    public void work() { System.out.println("Working"); }
    public void eat() {
        // Robots don't eat! But they are FORCED to implement this method
        throw new UnsupportedOperationException();
    }
}
```
`Robot` is forced to implement `eat()`, even though it makes no sense for a robot.

**Good Example (follows ISP):**
```java
interface Workable {
    void work();
}
interface Eatable {
    void eat();
}
class Human implements Workable, Eatable {
    public void work() { System.out.println("Working"); }
    public void eat() { System.out.println("Eating"); }
}
class Robot implements Workable {
    public void work() { System.out.println("Working"); }
    // Robot only implements what it actually needs
}
```
Now each class only implements the interfaces that actually make sense for it.

---

### D — Dependency Inversion Principle (DIP)

**Simple meaning:** High-level code (important business logic) should NOT depend directly on low-level code (small details). Both should depend on an **abstraction** (an interface), instead.

**Bad Example (breaks DIP):**
```java
class MySQLDatabase {
    void save(String data) { System.out.println("Saving to MySQL"); }
}
class UserService {
    MySQLDatabase db = new MySQLDatabase();   // directly depends on MySQL
    void saveUser(String data) {
        db.save(data);
    }
}
```
If you ever want to switch from MySQL to another database, you must CHANGE the `UserService` class itself — this is a tight, risky connection.

**Good Example (follows DIP):**
```java
interface Database {
    void save(String data);
}
class MySQLDatabase implements Database {
    public void save(String data) { System.out.println("Saving to MySQL"); }
}
class MongoDatabase implements Database {
    public void save(String data) { System.out.println("Saving to Mongo"); }
}
class UserService {
    private Database db;
    UserService(Database db) {   // depends on the ABSTRACTION, not a specific database
        this.db = db;
    }
    void saveUser(String data) {
        db.save(data);
    }
}
```
Now, `UserService` doesn't care WHICH database is used — it just uses whatever `Database` implementation is given to it. You can switch databases without touching `UserService` at all.

---

## 3. DRY (Don't Repeat Yourself)

**Simple meaning:** Don't write the same piece of logic in multiple places. If you find yourself copy-pasting code, it's a warning sign — put that logic in ONE place, and reuse it from there.

**Bad Example (breaks DRY):**
```java
class OrderService {
    double calculateTotalWithTax(double price) {
        return price + (price * 0.18);
    }
}
class InvoiceService {
    double calculateTotalWithTax(double price) {
        return price + (price * 0.18);   // same logic copy-pasted here!
    }
}
```
If the tax rate ever changes, you must remember to update it in BOTH places — easy to forget one, causing a bug.

**Good Example (follows DRY):**
```java
class TaxCalculator {
    static double addTax(double price) {
        return price + (price * 0.18);
    }
}
class OrderService {
    double calculateTotal(double price) {
        return TaxCalculator.addTax(price);
    }
}
class InvoiceService {
    double calculateTotal(double price) {
        return TaxCalculator.addTax(price);
    }
}
```
Now the tax logic exists in only ONE place. If it changes, you only update it once.

**Why does DRY matter?** Repeated code means repeated bugs, and repeated maintenance work. Keeping logic in one place makes updates safer and faster.

---

## 4. KISS (Keep It Simple, Stupid)

**Simple meaning:** Keep your code as SIMPLE as possible. Don't make things complicated when a simple solution already works well.

**Bad Example (breaks KISS — too complicated):**
```java
boolean isEven(int number) {
    return ((number % 2 == 0) ? true : false) == true ? true : false;
}
```
This code technically works, but it is confusing and unnecessarily long for such a simple check.

**Good Example (follows KISS):**
```java
boolean isEven(int number) {
    return number % 2 == 0;
}
```
Same result, but much simpler and easier to read.

**Why does KISS matter?** Simple code is easier to read, easier to debug, and easier for other developers (or future-you) to understand quickly. Complicated code often hides bugs and wastes everyone's time.

---

## 5. YAGNI (You Aren't Gonna Need It)

**Simple meaning:** Don't build a feature or add extra code UNTIL you actually need it. Don't waste time preparing for things that MIGHT happen in the future but haven't happened yet.

**Bad Example (breaks YAGNI):**
```java
class PaymentService {
    void payWithCreditCard() { }
    void payWithDebitCard() { }
    void payWithCrypto() { }        // not needed yet, nobody asked for this
    void payWithBankTransfer() { }  // not needed yet either
    // built "just in case," but never actually used
}
```
Building all these extra, unused features takes time, adds complexity, and might never even be used.

**Good Example (follows YAGNI):**
```java
class PaymentService {
    void payWithCreditCard() { }
    void payWithDebitCard() { }
    // only what is ACTUALLY needed right now
    // add more later, only WHEN it's actually required
}
```

**Why does YAGNI matter?** Extra, unused code still needs to be maintained, tested, and understood by everyone on the team — even if nobody ever uses it. It's better to add features only when they are truly needed.

---

## 6. Composition over Inheritance

**Simple meaning:** When you want to reuse code between classes, prefer combining objects together (**composition**, a "HAS-A" relationship) instead of extending a parent class (**inheritance**, an "IS-A" relationship), whenever possible.

**Why?** Inheritance creates a very TIGHT connection between the parent and child class. If the parent class changes, it can unexpectedly break the child class too. Composition is more flexible — you can easily change or replace the "part" being used, without breaking everything else.

**Example using Inheritance (tightly connected):**
```java
class Engine {
    void start() { System.out.println("Engine starting"); }
}
class Car extends Engine {   // Car "IS-A" Engine? That doesn't even make logical sense!
    void drive() { System.out.println("Driving"); }
}
```
This design is confusing — a `Car` is NOT really a type of `Engine`. It just USES an engine.

**Example using Composition (better, more flexible):**
```java
class Engine {
    void start() { System.out.println("Engine starting"); }
}
class Car {
    private Engine engine = new Engine();   // Car "HAS-A" Engine - makes more sense!

    void drive() {
        engine.start();
        System.out.println("Driving");
    }
}
```
Now `Car` simply USES an `Engine` object. If you want a `Car` with a different engine (like an `ElectricEngine`), you can easily swap it out, without messing with inheritance chains.

```java
class ElectricEngine extends Engine {
    void start() { System.out.println("Electric engine starting silently"); }
}
class Car {
    private Engine engine;
    Car(Engine engine) {   // you can pass in ANY type of engine
        this.engine = engine;
    }
    void drive() {
        engine.start();
        System.out.println("Driving");
    }
}
```

**When should you still use Inheritance?** When there is a genuine, natural "IS-A" relationship — like `Dog extends Animal`, or `Car extends Vehicle`. Inheritance is not bad by itself; the advice is simply to prefer composition when the relationship is really more of a "HAS-A" or "USES-A" situation.

---

# Part 2: Design Patterns

## 7. What are Design Patterns?

A **Design Pattern** is a well-known, tested SOLUTION to a common software design problem. Think of it like a recipe — programmers have faced the same design problems many times before, and design patterns are the "best known solutions" that were found to work well, that you can reuse in your own code.

**Important:** Design patterns are NOT ready-made code you copy-paste. They are more like IDEAS or TEMPLATES that you adapt to your specific situation.

Design patterns are usually grouped into 3 categories:
1. **Creational Patterns** — deal with HOW objects are created (Singleton, Factory, Builder, Prototype)
2. **Structural Patterns** — deal with HOW classes/objects are combined together (Adapter, Decorator, Proxy)
3. **Behavioral Patterns** — deal with HOW objects communicate and behave together (Strategy, Observer)

---

## 8. Singleton Pattern (Creational)

**Simple meaning:** Make sure a class has only **ONE object (instance)** in the whole application, and provide one global way to access it.

**When to use it?** When you need exactly ONE shared object across your whole program — like a configuration manager, a logging system, or a database connection manager.

```java
class Singleton {
    private static Singleton instance;   // holds the ONE and only instance

    private Singleton() { }   // private constructor - nobody outside can create a new one

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();   // create it only the FIRST time
        }
        return instance;   // return the SAME object every time after that
    }
}

public class Main {
    public static void main(String[] args) {
        Singleton s1 = Singleton.getInstance();
        Singleton s2 = Singleton.getInstance();
        System.out.println(s1 == s2);   // true - both point to the SAME object
    }
}
```

**Key ideas:**
- The constructor is made `private`, so nobody can create new objects directly using `new`
- A `static` method (`getInstance()`) controls access, and always returns the SAME object

---

## 9. Factory Pattern (Creational)

**Simple meaning:** Instead of creating objects directly using `new` everywhere in your code, use a special "Factory" class whose only job is to create objects for you, based on some input.

**When to use it?** When object creation logic is complicated, or when you want to hide WHICH exact class is being created from the rest of your code.

```java
interface Shape {
    void draw();
}
class Circle implements Shape {
    public void draw() { System.out.println("Drawing Circle"); }
}
class Square implements Shape {
    public void draw() { System.out.println("Drawing Square"); }
}

class ShapeFactory {
    static Shape getShape(String type) {
        if (type.equals("CIRCLE")) return new Circle();
        if (type.equals("SQUARE")) return new Square();
        return null;
    }
}

public class Main {
    public static void main(String[] args) {
        Shape shape = ShapeFactory.getShape("CIRCLE");
        shape.draw();   // Drawing Circle
    }
}
```

**Why is this useful?** The code that USES the shape (`Main`) doesn't need to know the exact class name (`Circle` or `Square`) — it just asks the factory for the right shape by name. If you add a new shape later, you only update the factory, not every place that creates shapes.

---

## 10. Builder Pattern (Creational)

**Simple meaning:** Used to build a COMPLEX object, STEP BY STEP, especially when an object has MANY optional fields, and you don't want a constructor with too many confusing parameters.

**When to use it?** When creating an object needs many optional pieces of information, and you want a clean, readable way to set only the ones you need.

```java
class House {
    private String foundation;
    private String walls;
    private String roof;
    private boolean hasGarden;

    // private constructor - only the Builder can create a House
    private House(Builder builder) {
        this.foundation = builder.foundation;
        this.walls = builder.walls;
        this.roof = builder.roof;
        this.hasGarden = builder.hasGarden;
    }

    public String toString() {
        return "House with " + foundation + ", " + walls + ", " + roof + ", garden=" + hasGarden;
    }

    static class Builder {
        private String foundation;
        private String walls;
        private String roof;
        private boolean hasGarden;

        Builder setFoundation(String foundation) { this.foundation = foundation; return this; }
        Builder setWalls(String walls) { this.walls = walls; return this; }
        Builder setRoof(String roof) { this.roof = roof; return this; }
        Builder setGarden(boolean hasGarden) { this.hasGarden = hasGarden; return this; }

        House build() {
            return new House(this);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        House house = new House.Builder()
                .setFoundation("Concrete")
                .setWalls("Brick")
                .setRoof("Tiles")
                .setGarden(true)
                .build();
        System.out.println(house);
    }
}
```

**Why is this useful?** Instead of one huge constructor with many confusing parameters (`new House("Concrete", "Brick", "Tiles", true, false, ...)`), you build the object step by step, with clear, readable method names — and you can skip any optional fields you don't need.

---

## 11. Prototype Pattern (Creational)

**Simple meaning:** Instead of creating a brand new object from scratch every time, create a COPY of an already existing object. This is useful when creating a new object is slow or expensive.

**When to use it?** When object creation takes a lot of time or resources (like loading data from a database or file), and you already have a similar object ready to copy.

```java
class Sheep implements Cloneable {
    String name;
    int age;

    Sheep(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Sheep clone() {
        return new Sheep(this.name, this.age);   // creates a copy, instead of building from scratch
    }
}

public class Main {
    public static void main(String[] args) {
        Sheep original = new Sheep("Dolly", 2);
        Sheep copy = original.clone();
        System.out.println(copy.name);   // Dolly (copied from original)
    }
}
```

**Java's built-in support:** Java actually has a built-in `Cloneable` interface and `clone()` method for this exact purpose, though many developers prefer writing their own copy constructor or copy method for more control.

---

## 12. Adapter Pattern (Structural)

**Simple meaning:** Adapter helps two INCOMPATIBLE things work together, by converting one interface into another that the client expects. Think of it like a power plug adapter — it lets a plug from one country fit into a socket from another country.

**When to use it?** When you have an existing class with a certain interface, but your code expects a DIFFERENT interface, and you can't (or don't want to) change the original class.

```java
// The interface our code expects
interface MediaPlayer {
    void play(String fileName);
}

// An existing, incompatible class we cannot change
class AdvancedPlayer {
    void playMp4(String fileName) {
        System.out.println("Playing mp4 file: " + fileName);
    }
}

// Adapter makes AdvancedPlayer fit the MediaPlayer interface
class MediaAdapter implements MediaPlayer {
    AdvancedPlayer advancedPlayer = new AdvancedPlayer();

    public void play(String fileName) {
        advancedPlayer.playMp4(fileName);   // translates the call
    }
}

public class Main {
    public static void main(String[] args) {
        MediaPlayer player = new MediaAdapter();
        player.play("movie.mp4");   // Playing mp4 file: movie.mp4
    }
}
```

**Real-life analogy:** A laptop charger from the USA won't fit directly into a socket in India. You use a plug ADAPTER in between, so the charger (which you cannot change) can still work with the different socket.

---

## 13. Strategy Pattern (Behavioral)

**Simple meaning:** Strategy lets you define a FAMILY of related actions (algorithms), put each one in its own class, and SWITCH between them easily at runtime — without changing the code that uses them.

**When to use it?** When you have multiple ways of doing the same kind of task (like different payment methods, or different sorting methods), and you want to choose which one to use flexibly.

```java
interface PaymentStrategy {
    void pay(int amount);
}
class CreditCardPayment implements PaymentStrategy {
    public void pay(int amount) { System.out.println("Paid " + amount + " using Credit Card"); }
}
class PayPalPayment implements PaymentStrategy {
    public void pay(int amount) { System.out.println("Paid " + amount + " using PayPal"); }
}

class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}

public class Main {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.setPaymentStrategy(new CreditCardPayment());
        cart.checkout(500);   // Paid 500 using Credit Card

        cart.setPaymentStrategy(new PayPalPayment());
        cart.checkout(300);   // Paid 300 using PayPal
    }
}
```

**Why is this useful?** `ShoppingCart` doesn't need to know HOW each payment method works internally — it just calls `pay()` on whichever strategy is currently set. You can add new payment methods later, without changing `ShoppingCart` at all.

---

## 14. Observer Pattern (Behavioral)

**Simple meaning:** Observer lets one object (called the "Subject") automatically NOTIFY many other objects (called "Observers") whenever something changes, without the Subject needing to know exactly who those observers are in advance.

**When to use it?** When you need many parts of your program to react automatically to a single event — like updating multiple screens when data changes, or sending notifications to many subscribers.

```java
interface Observer {
    void update(String news);
}
class NewsChannel implements Observer {
    String name;
    NewsChannel(String name) { this.name = name; }
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}

class NewsPublisher {
    private List<Observer> subscribers = new ArrayList<>();

    void subscribe(Observer o) {
        subscribers.add(o);
    }
    void publish(String news) {
        for (Observer o : subscribers) {
            o.update(news);   // notify EVERYONE automatically
        }
    }
}

public class Main {
    public static void main(String[] args) {
        NewsPublisher publisher = new NewsPublisher();
        publisher.subscribe(new NewsChannel("Channel A"));
        publisher.subscribe(new NewsChannel("Channel B"));

        publisher.publish("Big news happened!");
        // Channel A received news: Big news happened!
        // Channel B received news: Big news happened!
    }
}
```

**Real-life analogy:** Think of subscribing to a YouTube channel. When the channel uploads a new video, ALL subscribers get notified automatically — the channel doesn't send a personal message to each person one by one; it just "publishes," and everyone subscribed gets updated.

---

## 15. Decorator Pattern (Structural)

**Simple meaning:** Decorator lets you ADD new behavior to an object, WITHOUT changing its original class, by "wrapping" it inside another object that adds the extra behavior.

**When to use it?** When you want to add extra features to an object flexibly, and you don't want to create many complicated subclasses for every possible combination of features.

```java
interface Coffee {
    String getDescription();
    double getCost();
}
class SimpleCoffee implements Coffee {
    public String getDescription() { return "Coffee"; }
    public double getCost() { return 2.0; }
}

// Decorator wraps a Coffee object and adds extra behavior
class MilkDecorator implements Coffee {
    private Coffee coffee;
    MilkDecorator(Coffee coffee) { this.coffee = coffee; }

    public String getDescription() { return coffee.getDescription() + " + Milk"; }
    public double getCost() { return coffee.getCost() + 0.5; }
}

class SugarDecorator implements Coffee {
    private Coffee coffee;
    SugarDecorator(Coffee coffee) { this.coffee = coffee; }

    public String getDescription() { return coffee.getDescription() + " + Sugar"; }
    public double getCost() { return coffee.getCost() + 0.2; }
}

public class Main {
    public static void main(String[] args) {
        Coffee coffee = new SimpleCoffee();
        coffee = new MilkDecorator(coffee);     // wrap with Milk
        coffee = new SugarDecorator(coffee);     // wrap again with Sugar

        System.out.println(coffee.getDescription() + " costs $" + coffee.getCost());
        // Coffee + Milk + Sugar costs $2.7
    }
}
```

**Real-life analogy:** Think of ordering a coffee, and adding milk, then sugar, then whipped cream — each addition "wraps" and adds to the previous one, without needing a separate, specifically-named class for every single possible combination (like `CoffeeWithMilkAndSugarAndCream`).

---

## 16. Proxy Pattern (Structural)

**Simple meaning:** Proxy provides a SUBSTITUTE or PLACEHOLDER object that controls access to another (real) object. The proxy looks like the real object, but it can add extra logic BEFORE or AFTER passing the request to the real object — like security checks, logging, or delaying expensive work until it's really needed.

**When to use it?** When you want to control access to an object — for example, checking permissions before allowing an action, or delaying the creation of an expensive object until it's actually needed (this specific use is called "Lazy Loading").

```java
interface Image {
    void display();
}
class RealImage implements Image {
    private String fileName;
    RealImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk();   // expensive operation
    }
    private void loadFromDisk() {
        System.out.println("Loading " + fileName + " from disk...");
    }
    public void display() {
        System.out.println("Displaying " + fileName);
    }
}

// Proxy controls when the real, expensive object is actually created
class ProxyImage implements Image {
    private RealImage realImage;
    private String fileName;

    ProxyImage(String fileName) {
        this.fileName = fileName;
    }

    public void display() {
        if (realImage == null) {
            realImage = new RealImage(fileName);   // load only when actually needed
        }
        realImage.display();
    }
}

public class Main {
    public static void main(String[] args) {
        Image image = new ProxyImage("photo.jpg");
        System.out.println("Image object created, but not loaded yet");

        image.display();   // NOW it loads from disk, and displays
        // Loading photo.jpg from disk...
        // Displaying photo.jpg

        image.display();   // second time - already loaded, no reloading needed
        // Displaying photo.jpg
    }
}
```

**Real-life analogy:** Think of a security guard (proxy) standing in front of a manager's office (real object). The guard checks who you are and decides whether to let you in, instead of you walking directly to the manager every time.

---

## 17. Summary Cheat Sheet

### Design Principles

| Principle | One-line meaning |
|-----------|--------------------|
| SRP | A class should do only ONE job |
| OCP | Add new features without changing old, working code |
| LSP | A subclass should work correctly wherever its parent class is expected |
| ISP | Don't force a class to implement methods it doesn't need |
| DIP | Depend on abstractions (interfaces), not specific implementations |
| DRY | Don't repeat the same logic in multiple places |
| KISS | Keep your code as simple as possible |
| YAGNI | Don't build features you don't need yet |
| Composition over Inheritance | Prefer "HAS-A" relationships over "IS-A" when reuse is the only goal |

### Design Patterns

| Pattern | Category | One-line meaning |
|---------|----------|--------------------|
| Singleton | Creational | Only ONE instance of a class, shared everywhere |
| Factory | Creational | A special class creates objects for you, based on input |
| Builder | Creational | Build a complex object step by step, cleanly |
| Prototype | Creational | Create new objects by copying an existing one |
| Adapter | Structural | Makes two incompatible interfaces work together |
| Strategy | Behavioral | Swap between different algorithms/behaviors at runtime |
| Observer | Behavioral | Automatically notify many objects when something changes |
| Decorator | Structural | Add extra behavior to an object by wrapping it |
| Proxy | Structural | A substitute object that controls access to the real object |

---

**Congratulations! You now understand the most important Design Principles and Design Patterns used in real-world software development.** Try to spot these patterns in real code you read — once you notice them in the wild, they become much easier to remember and use yourself.
