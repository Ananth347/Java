# Java Core Concepts — Tutorial Notes

Topics covered: **File I/O**, **Annotations**, **Enums**, **Nested Classes**, **Functional Programming**

---

## 1. Java File I/O

### What it is, in plain words
File I/O just means reading data from a file into your program, or writing data from your program out to a file. Think of it like opening a notebook (reading) or writing into one (writing). Java gives you several "pens and readers" depending on what kind of data you're dealing with — plain text, raw bytes (like images), or entire files at once.

### The two families of I/O
- **Byte streams** (`InputStream` / `OutputStream`) — work with raw bytes. Use these for images, videos, PDFs, or any binary data.
- **Character streams** (`Reader` / `Writer`) — work with text, and automatically handle character encoding (like UTF-8). Use these for `.txt`, `.csv`, `.json`, etc.

A simple rule: **if a human would open the file and read words, use character streams. If not, use byte streams.**

### Old style vs modern style
Java originally gave us `FileReader`, `FileWriter`, `BufferedReader`, etc. Since Java 7/8, there's a much simpler, safer API called **NIO.2** (`java.nio.file`), centered around the `Path` and `Files` classes. In real projects today, you'll mostly use `Files` for simple tasks and `BufferedReader`/`BufferedWriter` when you need line-by-line control.

### Reading a whole text file (modern, simple way)
```java
import java.nio.file.*;
import java.util.List;

public class ReadExample {
    public static void main(String[] args) throws Exception {
        Path path = Path.of("data.txt");

        // Read everything into one String
        String content = Files.readString(path);
        System.out.println(content);

        // Or read line by line into a List
        List<String> lines = Files.readAllLines(path);
        lines.forEach(System.out::println);
    }
}
```

### Writing to a text file
```java
import java.nio.file.*;

public class WriteExample {
    public static void main(String[] args) throws Exception {
        Path path = Path.of("output.txt");
        Files.writeString(path, "Hello, this is a new file!\n");

        // Append instead of overwrite
        Files.writeString(path, "Second line\n", StandardOpenOption.APPEND);
    }
}
```

### Reading a large file efficiently (line by line, low memory use)
When a file is huge, you don't want to load it all into memory at once. `BufferedReader` reads line by line:
```java
import java.io.*;

public class BufferedReadExample {
    public static void main(String[] args) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } // file is auto-closed here
    }
}
```

### Why `try-with-resources` matters
Files are a limited resource — the OS only allows so many open at once, and forgetting to close one causes leaks or locked files. `try-with-resources` (the `try (...)` syntax above) automatically closes the file for you, even if an exception happens. **Always use it** instead of manually calling `.close()`.

### Working with binary files (images, zip, etc.)
```java
import java.nio.file.*;

public class BinaryExample {
    public static void main(String[] args) throws Exception {
        byte[] data = Files.readAllBytes(Path.of("photo.jpg"));
        Files.write(Path.of("copy.jpg"), data);
    }
}
```

### Checking, creating, deleting files
```java
Path path = Path.of("notes.txt");

Files.exists(path);          // true/false
Files.createFile(path);      // creates a new empty file
Files.delete(path);          // deletes it
Files.createDirectories(Path.of("logs/2026")); // creates nested folders
```

### Interview-relevant points
- Know the difference between `InputStream`/`OutputStream` (bytes) and `Reader`/`Writer` (characters).
- `BufferedReader`/`BufferedWriter` wrap around basic streams to reduce the number of actual disk operations (better performance).
- `Files` (NIO.2) is preferred in modern code for simplicity and better error handling (throws clear exceptions like `NoSuchFileException`).
- Always close resources — `try-with-resources` is the standard, professional way.

---

## 2. Annotations

### What it is, in plain words
An annotation is a **label you attach to your code** that doesn't change what the code does by itself, but gives extra information to the compiler, tools, or frameworks. Think of it like a sticky note on a document — it doesn't change the document's content, but it tells someone (a tool, a framework, another developer) something important about it.

You've already used these constantly in Spring Boot: `@RestController`, `@Autowired`, `@Service`, `@Entity` — these are all annotations telling Spring "treat this class specially."

### Built-in annotations you already know
```java
@Override
public String toString() {
    return "Example";
}

@Deprecated
public void oldMethod() { }

@SuppressWarnings("unchecked")
public void riskyCast() { }
```
- `@Override` — tells the compiler "I intend to override a parent method." If you misspell the method name, the compiler will now catch it as an error instead of silently creating a new method.
- `@Deprecated` — marks something as outdated; using it triggers a compiler warning.
- `@SuppressWarnings` — tells the compiler to hide specific warnings for that code block.

### Creating your own annotation
```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)  // available at runtime (so we can read it via reflection)
@Target(ElementType.METHOD)          // can only be applied to methods
public @interface LogExecutionTime {
}
```

Usage:
```java
public class ReportService {
    @LogExecutionTime
    public void generateReport() {
        // business logic
    }
}
```

By itself, this annotation does nothing — it's just a marker. To make it *actually do something* (like log execution time), you need something reading it, usually via **reflection** or an **AOP framework** (like Spring AOP).

### The meta-annotations (annotations about annotations)
- `@Retention` — controls how long the annotation is kept:
  - `SOURCE` — thrown away after compilation (e.g. `@Override`)
  - `CLASS` — kept in the `.class` file, but not available at runtime
  - `RUNTIME` — available at runtime via reflection (needed for frameworks like Spring)
- `@Target` — restricts where the annotation can be applied (class, method, field, parameter, etc.)
- `@Inherited` — subclasses inherit the annotation from their parent class
- `@Documented` — includes the annotation in generated JavaDocs

### Reading annotations with reflection (how frameworks like Spring work under the hood)
```java
import java.lang.reflect.Method;

public class AnnotationReader {
    public static void main(String[] args) throws Exception {
        Method method = ReportService.class.getMethod("generateReport");
        if (method.isAnnotationPresent(LogExecutionTime.class)) {
            System.out.println("This method wants execution time logged!");
        }
    }
}
```
This is essentially what Spring, Hibernate, and Jackson do internally: they scan your classes at startup, look for annotations like `@Entity` or `@RestController`, and configure behavior automatically based on what they find.

### Annotations with values
```java
public @interface ApiEndpoint {
    String path();
    String method() default "GET";
}

@ApiEndpoint(path = "/users", method = "POST")
public void createUser() { }
```

### Interview-relevant points
- Annotations don't execute logic on their own — something (reflection, a framework, the compiler) must read and act on them.
- `RetentionPolicy.RUNTIME` is what allows frameworks like Spring to detect annotations dynamically at startup.
- This is the exact mechanism behind Spring's dependency injection and Hibernate's ORM mapping.

---

## 3. Enums

### What it is, in plain words
An enum (short for "enumeration") is a special class that represents a **fixed set of constant values**. Instead of using loose strings or numbers to represent something like "status" (which is error-prone — typos, invalid values), you define exactly what the valid options are.

Bad way (error-prone):
```java
String status = "APPROVED"; // could easily be misspelled "APPROOVED" and the compiler won't catch it
```

Good way (type-safe):
```java
public enum Status {
    PENDING, APPROVED, REJECTED
}

Status status = Status.APPROVED; // compiler guarantees this is always a valid value
```

### Basic usage
```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

public class EnumBasics {
    public static void main(String[] args) {
        Day today = Day.MONDAY;

        if (today == Day.MONDAY) {
            System.out.println("Start of the work week");
        }

        // switch works cleanly with enums
        switch (today) {
            case MONDAY -> System.out.println("Ugh, Monday");
            case FRIDAY -> System.out.println("Almost there");
            default -> System.out.println("Just another day");
        }
    }
}
```

### Enums are more powerful than in most languages — they can have fields, constructors, and methods
This is the part that surprises people coming from languages where enums are just plain integers. In Java, an enum is a **full class**.

```java
public enum PaymentStatus {
    PENDING(1, "Payment initiated but not confirmed"),
    SUCCESS(2, "Payment completed successfully"),
    FAILED(3, "Payment could not be processed");

    private final int code;
    private final String description;

    // enum constructor — always private (implicitly)
    PaymentStatus(int code, String description) {
        this.code = code;
        this.description = description;
    }

    public int getCode() {
        return code;
    }

    public String getDescription() {
        return description;
    }
}
```

Usage:
```java
PaymentStatus status = PaymentStatus.SUCCESS;
System.out.println(status.getCode());        // 2
System.out.println(status.getDescription()); // Payment completed successfully
```

### Enums can even have different behavior per constant
```java
public enum Operation {
    ADD {
        public int apply(int a, int b) { return a + b; }
    },
    SUBTRACT {
        public int apply(int a, int b) { return a - b; }
    };

    public abstract int apply(int a, int b);
}

// Usage
int result = Operation.ADD.apply(5, 3); // 8
```
This is a clean alternative to writing a big `if/else` or `switch` block based on a "type" field — a very common pattern in real banking/fintech systems (e.g. transaction types, approval workflows).

### Useful built-in enum methods
```java
Day.values();          // array of all enum constants, in declared order
Day.valueOf("MONDAY"); // converts a String to the enum constant (throws exception if invalid)
today.name();           // "MONDAY" — the exact constant name
today.ordinal();        // 0 — position in declaration order (avoid relying on this for business logic!)
```

### Enums implementing interfaces
Enums can implement interfaces (but cannot extend another class, since they already secretly extend `java.lang.Enum`):
```java
public interface Describable {
    String describe();
}

public enum Role implements Describable {
    ADMIN, USER, GUEST;

    public String describe() {
        return "Role: " + this.name();
    }
}
```

### Interview-relevant points
- Enums are type-safe alternatives to constants — the compiler prevents invalid values, unlike Strings or ints.
- Enums can hold fields, constructors, and methods — they're real classes under the hood.
- Never rely on `ordinal()` for business logic (like storing it in a database) — if someone reorders the enum constants later, the meaning silently breaks. Use an explicit `code` field instead, like in the `PaymentStatus` example.
- Enum singletons are a common and safe way to implement the Singleton design pattern in Java.

---

## 4. Nested Classes

### What it is, in plain words
A nested class is simply **a class defined inside another class**. You use this when a class only makes sense in the context of its "outer" class — it's a way of saying "this helper class belongs to, and is tightly coupled with, this other class."

There are **four kinds**, and picking the right one matters:

| Type | Has access to outer instance? | Common use case |
|---|---|---|
| Static nested class | No | Grouping a helper class logically inside another (most common) |
| Inner class (non-static) | Yes | Needs to work closely with an instance of the outer class |
| Local class | Yes (if in an instance method) | Rarely used; class defined inside a method |
| Anonymous class | Yes (if in an instance method) | Quick, throwaway implementation of an interface/abstract class |

### 1. Static nested class
Doesn't need an instance of the outer class to exist. This is the one you'll use most often — for example, a `Builder` pattern.
```java
public class Order {
    private final String item;
    private final int quantity;

    private Order(Builder builder) {
        this.item = builder.item;
        this.quantity = builder.quantity;
    }

    public static class Builder {
        private String item;
        private int quantity;

        public Builder item(String item) {
            this.item = item;
            return this;
        }

        public Builder quantity(int quantity) {
            this.quantity = quantity;
            return this;
        }

        public Order build() {
            return new Order(this);
        }
    }
}

// Usage
Order order = new Order.Builder()
        .item("Laptop")
        .quantity(2)
        .build();
```

### 2. Inner class (non-static)
Tied to a specific instance of the outer class — it can directly access the outer object's fields, even private ones.
```java
public class Car {
    private String model = "Model X";

    class Engine {
        void start() {
            // can directly access outer class's field
            System.out.println("Starting engine of " + model);
        }
    }
}

// Usage — needs an outer instance first
Car car = new Car();
Car.Engine engine = car.new Engine();
engine.start();
```

### 3. Local class
Defined right inside a method body. Rare in day-to-day code, but occasionally useful for a one-off helper used only within that method.
```java
public void processOrder() {
    class Validator {
        boolean isValid(int amount) {
            return amount > 0;
        }
    }

    Validator validator = new Validator();
    System.out.println(validator.isValid(100));
}
```

### 4. Anonymous class
A class with no name, created and used on the spot — usually to implement an interface or abstract class quickly, without writing a whole separate file.
```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running task...");
    }
};
task.run();
```
In modern Java, anonymous classes for simple interfaces are often replaced by **lambdas** (covered in the next section) — but you'll still see anonymous classes for interfaces with more than one method, or in older codebases.

### Why use nested classes at all?
- **Encapsulation** — the nested class can be hidden from the rest of the application if it's only useful internally (`private static class`).
- **Logical grouping** — keeps closely related code together, improving readability.
- **Builder patterns**, **event listeners**, and **map entries** (`Map.Entry` is itself a nested interface) are classic real-world uses.

### Interview-relevant points
- Static nested class vs inner class is a common interview question: the key difference is whether it holds an implicit reference to an instance of the outer class.
- Non-static inner classes hold a hidden reference to their outer class instance — this can cause memory leaks if inner class instances outlive the outer object unexpectedly (common gotcha with Android/GUI event listeners).
- `Map.Entry`, `Builder` patterns, and Java's own `LinkedList.Node` are real-world examples of static nested classes.

---

## 5. Functional Programming (in Java)

### What it is, in plain words
Functional programming means treating **functions/behavior as data** — you can pass a "piece of logic" into a method, store it in a variable, or return it from another method, the same way you'd pass around a number or a String. Java added strong support for this in Java 8 through **lambdas**, **functional interfaces**, and the **Stream API**.

Before Java 8, if you wanted to pass behavior around, you needed an anonymous class (verbose). Now you can write it concisely as a lambda.

### Functional interfaces — the foundation
A functional interface is simply an interface with **exactly one abstract method**. This single-method restriction is what allows a lambda to "fill in" that method.

```java
@FunctionalInterface
public interface Greeter {
    void greet(String name);
}
```

```java
// Old way — anonymous class
Greeter g1 = new Greeter() {
    @Override
    public void greet(String name) {
        System.out.println("Hello, " + name);
    }
};

// New way — lambda expression (does exactly the same thing)
Greeter g2 = name -> System.out.println("Hello, " + name);

g2.greet("Ananth"); // Hello, Ananth
```

### The built-in functional interfaces you'll use constantly
Java provides ready-made functional interfaces in `java.util.function` so you rarely need to write your own:

| Interface | Method signature | Purpose |
|---|---|---|
| `Function<T, R>` | `R apply(T t)` | Takes an input, returns an output |
| `Consumer<T>` | `void accept(T t)` | Takes an input, returns nothing (does something with it) |
| `Supplier<T>` | `T get()` | Takes nothing, returns a value |
| `Predicate<T>` | `boolean test(T t)` | Takes an input, returns true/false |
| `BiFunction<T, U, R>` | `R apply(T t, U u)` | Takes two inputs, returns an output |

```java
import java.util.function.*;

Function<Integer, Integer> square = x -> x * x;
System.out.println(square.apply(5)); // 25

Predicate<String> isEmpty = str -> str.isEmpty();
System.out.println(isEmpty.test(""));  // true

Consumer<String> printer = msg -> System.out.println("Log: " + msg);
printer.accept("Server started");

Supplier<Double> randomValue = () -> Math.random();
System.out.println(randomValue.get());
```

### Method references — a shortcut for lambdas that just call one existing method
```java
// Instead of:
Consumer<String> printer1 = s -> System.out.println(s);

// You can write:
Consumer<String> printer2 = System.out::println;
```
Common forms: `ClassName::staticMethod`, `object::instanceMethod`, `ClassName::instanceMethod`, `ClassName::new` (constructor reference).

### The Stream API — functional-style processing of collections
Streams let you process collections (lists, sets) in a declarative way — you describe *what* you want, not *how* to loop through it.

```java
import java.util.*;
import java.util.stream.*;

public class StreamExample {
    public static void main(String[] args) {
        List<String> names = List.of("Ananth", "Ravi", "Priya", "Arun", "Kumar");

        List<String> result = names.stream()
                .filter(name -> name.length() > 4)     // keep names longer than 4 letters
                .map(String::toUpperCase)               // convert to uppercase
                .sorted()                                // sort alphabetically
                .collect(Collectors.toList());           // gather into a List

        System.out.println(result); // [ANANTH, KUMAR, PRIYA]
    }
}
```

Compare that to the old imperative way:
```java
List<String> result = new ArrayList<>();
for (String name : names) {
    if (name.length() > 4) {
        result.add(name.toUpperCase());
    }
}
Collections.sort(result);
```
Both do the same thing, but the stream version reads closer to a plain-English description of the steps.

### Common stream operations
```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Sum of even numbers
int sum = numbers.stream()
        .filter(n -> n % 2 == 0)
        .mapToInt(Integer::intValue)
        .sum();

// Count matching a condition
long count = numbers.stream().filter(n -> n > 5).count();

// Check if any/all match
boolean anyEven = numbers.stream().anyMatch(n -> n % 2 == 0);
boolean allPositive = numbers.stream().allMatch(n -> n > 0);

// Grouping (very common in real projects, e.g. grouping transactions by status)
Map<Boolean, List<Integer>> grouped = numbers.stream()
        .collect(Collectors.partitioningBy(n -> n % 2 == 0));
```

### Optional — handling "maybe there's no value" the functional way
`Optional` is designed to work well with functional style and helps avoid `NullPointerException`.
```java
Optional<String> maybeName = Optional.ofNullable(getNameFromDb());

String name = maybeName
        .map(String::toUpperCase)
        .orElse("UNKNOWN");
```

### Why this matters for your day-to-day Spring Boot work
- Stream API is used heavily for transforming DTOs, filtering results from repositories, and aggregating data (e.g. calculating totals for a report).
- `Optional` is used in Spring Data JPA — `findById()` returns `Optional<Entity>`, forcing you to explicitly handle the "not found" case instead of risking a null pointer.
- Lambdas are used constantly for things like `@Bean` definitions, comparator logic (`list.sort((a, b) -> ...)`), and event handling.

### Interview-relevant points
- A functional interface has exactly one abstract method (it can have multiple default/static methods).
- Lambdas don't create a new type — they're just a compact way to implement a functional interface.
- Streams are **lazy** — nothing actually executes until a terminal operation (like `collect`, `sum`, `forEach`) is called. `filter` and `map` are "intermediate" operations that just build up a pipeline.
- Streams can only be consumed **once** — trying to reuse a stream after a terminal operation throws an `IllegalStateException`.

---

## Quick recap

| Topic | Core idea |
|---|---|
| **File I/O** | Reading/writing files using byte streams (binary) or character streams (text); prefer `Files`/NIO.2 for simple tasks, `BufferedReader` for large files, always use try-with-resources |
| **Annotations** | Metadata labels on code, read by the compiler or frameworks (via reflection) to change behavior — the backbone of how Spring/Hibernate "magic" works |
| **Enums** | Type-safe, fixed sets of constants that can hold fields, constructors, and even per-constant behavior |
| **Nested Classes** | Classes defined inside other classes — static nested (no outer link), inner (linked to outer instance), local, and anonymous |
| **Functional Programming** | Treating behavior as data via lambdas and functional interfaces, enabling the declarative Stream API and safer null-handling with `Optional` |
