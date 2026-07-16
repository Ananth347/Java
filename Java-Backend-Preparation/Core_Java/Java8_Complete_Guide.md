# Java 8 — The Complete Guide
### Every feature explained simply, with examples and the "why" behind each one

---

## How to read this document

Java 8 (released 2014) was the biggest change to Java since it was created — it added functional programming style to a language that had been purely object-oriented for 20 years. We'll go feature by feature, in the order that makes most sense to *learn* them (not necessarily the order they appear in the JDK release notes). Each topic has:
- **What it is** (plain English)
- **Why it exists** (the problem it solves)
- **Code example**
- **Common interview angle**

Topics covered: Lambda Expressions, Functional Interfaces, Method References, Default & Static methods in interfaces, the Stream API (in depth), Optional, the new Date/Time API, Collectors, Parallel Streams, CompletableFuture, Nashorn JS engine, Base64 encoding, Repeating Annotations, Type Annotations, and the internal changes (PermGen removal, String pool).

---

# 1. Why Java 8 happened — the core problem

Before Java 8, if you wanted to tell Java **what to do** with a piece of behavior — like "sort this list this specific way" or "run this on a new thread" — you had to wrap that behavior in an entire class, usually an anonymous inner class:

```java
// Before Java 8
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

That's a lot of ceremony (4 lines) to say "when you run this, print a message." Java 8's central idea is: **treat behavior (code) like a value** — pass it around, store it, return it — the same way you already pass around numbers or strings. This is called **functional programming**, and Java 8 bolted it onto an object-oriented language. Almost every Java 8 feature exists to support this one idea.

---

# 2. Lambda Expressions

## What it is
A lambda expression is a compact way to write an anonymous function — a block of code with parameters, but no name, no return type declaration, and no class wrapper.

## Syntax
```java
(parameters) -> expression
(parameters) -> { statements; }
```

## The same Runnable example, in Java 8:
```java
Runnable r = () -> System.out.println("Running");
```
That's it. One line instead of five. The `()` means no parameters, `->` means "goes to / does", and what follows is the body.

## More examples

```java
// No parameters
() -> System.out.println("Hello");

// One parameter (parentheses optional for single param)
name -> System.out.println("Hello " + name);

// Multiple parameters
(a, b) -> a + b;

// With explicit types (usually not needed — compiler infers them)
(int a, int b) -> a + b;

// Multiple statements need braces and an explicit return
(a, b) -> {
    int sum = a + b;
    return sum;
};
```

## Why lambdas matter
They let you pass a "recipe" for behavior directly as an argument to a method, instead of building a whole class first. This is the foundation everything else in this document is built on.

```java
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
names.sort((a, b) -> a.compareTo(b));  // pass the "how to compare" logic directly
```

## Common interview angle
**Q: Can a lambda access variables from outside its body?**
Yes, but only if those variables are "effectively final" (never reassigned after being set). This is because the lambda might run later or on a different thread, and Java needs a stable, unchanging copy of that variable's value to safely capture.

```java
int factor = 2;
Function<Integer, Integer> doubler = x -> x * factor; // OK — factor never changes
```

---

# 3. Functional Interfaces

## What it is
Lambdas need a "type" to be assigned to — you can't just have a lambda floating with no target. A **functional interface** is any interface with **exactly one abstract method**. That single method is what the lambda "becomes" when assigned to it.

You mark one (optionally) with `@FunctionalInterface` — this annotation isn't required, but it makes the compiler enforce the "only one abstract method" rule, catching mistakes early.

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

Calculator add = (a, b) -> a + b;
System.out.println(add.calculate(3, 4)); // 7
```

## The built-in functional interfaces (java.util.function)

You rarely need to write your own — Java 8 ships a standard set covering almost every use case:

| Interface | Method signature | Use case |
|---|---|---|
| `Function<T, R>` | `R apply(T t)` | Takes one input, returns a (possibly different type) output |
| `BiFunction<T, U, R>` | `R apply(T t, U u)` | Takes two inputs, returns output |
| `Predicate<T>` | `boolean test(T t)` | Takes input, returns true/false (a condition/filter) |
| `Consumer<T>` | `void accept(T t)` | Takes input, returns nothing (does something with it, e.g., print) |
| `Supplier<T>` | `T get()` | Takes no input, returns a value (a "factory") |
| `UnaryOperator<T>` | `T apply(T t)` | Function where input and output are the same type |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | BiFunction where all types are the same |

### Examples of each:

```java
Function<String, Integer> length = s -> s.length();
System.out.println(length.apply("hello")); // 5

Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4)); // true

Consumer<String> printer = s -> System.out.println("Value: " + s);
printer.accept("test"); // Value: test

Supplier<Double> randomValue = () -> Math.random();
System.out.println(randomValue.get());

UnaryOperator<Integer> square = x -> x * x;
System.out.println(square.apply(5)); // 25

BinaryOperator<Integer> sum = (a, b) -> a + b;
System.out.println(sum.apply(3, 4)); // 7
```

## Why this matters
This small set of interfaces is reused *everywhere* — in Streams, in Optional, in CompletableFuture. Learning these 5–7 interfaces well unlocks understanding of almost every other Java 8 API.

## Common interview angle
**Q: Difference between `Function` and `Consumer`?**
`Function` takes something in and gives something back (`apply`). `Consumer` takes something in and does something with it, but returns nothing (`accept`) — used purely for side effects like printing or saving.

---

# 4. Method References

## What it is
A shorthand for a lambda that does nothing but call an existing method. If your lambda body is *just* calling one method and passing along the arguments, you can replace it with a method reference using `::`.

## The four kinds

**1. Reference to a static method**
```java
// Lambda:
Function<String, Integer> f1 = s -> Integer.parseInt(s);
// Method reference:
Function<String, Integer> f2 = Integer::parseInt;
```

**2. Reference to an instance method of a particular object**
```java
String prefix = "Hello ";
// Lambda:
Function<String, String> f1 = s -> prefix.concat(s);
// Method reference:
Function<String, String> f2 = prefix::concat;
```

**3. Reference to an instance method of an arbitrary object of a particular type**
```java
// Lambda:
Function<String, Integer> f1 = s -> s.length();
// Method reference:
Function<String, Integer> f2 = String::length;
```

**4. Reference to a constructor**
```java
// Lambda:
Supplier<ArrayList<String>> f1 = () -> new ArrayList<>();
// Method reference:
Supplier<ArrayList<String>> f2 = ArrayList::new;
```

## Why it matters
It's purely about readability — `String::length` instantly tells a reader "this calls length() on whatever string comes in," which is often clearer than an equivalent lambda.

## Common interview angle
**Q: When can't you use a method reference?**
When the lambda does anything more than a single method call — e.g., `x -> x.length() + 1` cannot be a method reference, since there's extra logic (`+1`) beyond just delegating.

---

# 5. Default and Static Methods in Interfaces

## The problem this solves
Before Java 8, interfaces could only declare abstract methods (no body). This meant if you had, say, the `List` interface already implemented by a thousand classes across the world, and you wanted to add a new method to it, **every single implementing class would break** (they'd all fail to compile until they implemented the new method).

## The solution: default methods
A `default` method has a body, right inside the interface. Implementing classes automatically inherit this behavior and don't have to implement it — but *can* override it if they want something different.

```java
interface Vehicle {
    void drive(); // abstract — must be implemented

    default void honk() { // default — has a body, optional to override
        System.out.println("Beep beep!");
    }
}

class Car implements Vehicle {
    public void drive() {
        System.out.println("Car is driving");
    }
    // honk() not implemented — uses the default automatically
}
```

This is exactly how Java added `forEach()`, `stream()`, `sort()`, and dozens of other new methods to existing collection interfaces in Java 8 **without breaking any existing code** written before Java 8 existed.

## Static methods in interfaces
Interfaces can also have `static` methods — utility methods related to the interface, called on the interface itself, not on an instance.

```java
interface MathOperation {
    static int square(int x) {
        return x * x;
    }
}

MathOperation.square(5); // 25 — called directly on the interface
```

## The "diamond problem" and how Java resolves it
If a class implements two interfaces that both have a default method with the *same signature*, Java forces you to resolve the conflict explicitly — it won't guess which one you meant.

```java
interface A { default void greet() { System.out.println("A"); } }
interface B { default void greet() { System.out.println("B"); } }

class C implements A, B {
    public void greet() {
        A.super.greet(); // explicitly choose which parent's version to use
    }
}
```

## Common interview angle
**Q: Why were default methods added?**
Purely for backward compatibility — to evolve interfaces (adding new methods) without breaking every class that already implements them.

---

# 6. The Stream API (the biggest feature — covered in depth)

## What is a Stream?
A Stream is **not a data structure** — it doesn't store elements. It's a pipeline that describes a sequence of operations (filter, transform, sort, etc.) to run over a source of data (a Collection, array, or generator), and it's **lazy** — nothing actually runs until you call a "terminal" operation.

Think of it like an assembly line: you describe each station the item passes through, but the belt doesn't move until someone flips the "start" switch.

## Basic example
```java
List<String> names = Arrays.asList("Charlie", "Alice", "Bob", "Anna");

List<String> result = names.stream()
    .filter(name -> name.startsWith("A"))   // keep only names starting with A
    .map(String::toUpperCase)               // convert to uppercase
    .sorted()                                // sort alphabetically
    .collect(Collectors.toList());           // gather into a List

System.out.println(result); // [ALICE, ANNA]
```

## Streams have three parts
1. **Source** — where data comes from (`list.stream()`, `Arrays.stream(arr)`, `Stream.of(1,2,3)`)
2. **Intermediate operations** — transform the stream, return another stream, are **lazy** (don't execute immediately): `filter()`, `map()`, `sorted()`, `distinct()`, `limit()`, `skip()`, `flatMap()`
3. **Terminal operation** — triggers actual execution, produces a result or side effect, and **the stream can't be reused after this**: `collect()`, `forEach()`, `count()`, `reduce()`, `anyMatch()`, `findFirst()`

## Why "lazy" matters — a demonstration
```java
Stream<String> stream = names.stream()
    .filter(name -> {
        System.out.println("Filtering: " + name);
        return name.startsWith("A");
    });
System.out.println("Stream created, but nothing printed yet!");
stream.forEach(System.out::println); // NOW the filtering prints happen
```
Nothing inside `filter()` runs until `forEach()` (the terminal operation) is called. This laziness lets Java optimize the whole pipeline — e.g., combining multiple steps into a single pass over the data.

## Key intermediate operations, explained

**`filter(Predicate)`** — keep only elements matching a condition
```java
numbers.stream().filter(n -> n > 10)
```

**`map(Function)`** — transform each element into something else
```java
names.stream().map(String::toUpperCase)
```

**`flatMap(Function)`** — flattens nested structures. If you have a stream of lists, `flatMap` merges them into one single flat stream.
```java
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2), Arrays.asList(3, 4));

List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// [1, 2, 3, 4] — instead of a stream of lists, we get one flat stream of numbers
```

**`sorted()`** — natural order, or `sorted(Comparator)` for custom order
```java
names.stream().sorted(Comparator.reverseOrder())
```

**`distinct()`** — removes duplicates (uses `equals()`)

**`limit(n)`** — take only the first n elements
**`skip(n)`** — skip the first n elements (useful together for pagination)

```java
numbers.stream().skip(10).limit(5) // like "page 3" of 5 items per page
```

## Key terminal operations, explained

**`collect()`** — gathers stream elements into a collection or other result (covered in depth in the Collectors section below)

**`forEach(Consumer)`** — performs an action per element (like a for-loop, but no result returned)
```java
names.stream().forEach(System.out::println);
```

**`count()`** — returns the number of elements as a long

**`reduce()`** — combines all elements into one single value
```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
// starts with 0, keeps adding each element
```
Think of `reduce` as folding a list down to a single value — sum, product, max, string concatenation, etc.

**`anyMatch()`, `allMatch()`, `noneMatch()`** — return `boolean`, checking a condition against elements
```java
boolean hasNegative = numbers.stream().anyMatch(n -> n < 0);
boolean allPositive = numbers.stream().allMatch(n -> n > 0);
```

**`findFirst()`, `findAny()`** — return an `Optional` wrapping the first matching element (or empty if none found)

## Why streams matter over traditional loops
```java
// Traditional way
List<String> result = new ArrayList<>();
for (String name : names) {
    if (name.startsWith("A")) {
        result.add(name.toUpperCase());
    }
}
Collections.sort(result);
```
vs
```java
// Stream way
List<String> result = names.stream()
    .filter(name -> name.startsWith("A"))
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```
The stream version reads like a description of *what* you want, not *how* to loop and manage a mutable accumulator — this is called **declarative** style vs the old **imperative** style. It's also easier to parallelize (see Section 9).

## Common interview angle
**Q: Can you reuse a stream after a terminal operation runs?**
No — once a terminal operation executes, the stream is considered "consumed." Calling another operation on it throws `IllegalStateException`. You'd need to create a fresh stream from the source again.

**Q: Is a Stream a Collection?**
No. Collections store data in memory; Streams describe a computation over data and don't store anything themselves.

---

# 7. Collectors — turning a stream back into something useful

## What it is
`Collectors` is a utility class with pre-built strategies for the `collect()` terminal operation — the most common way to end a stream pipeline.

## The essentials

```java
// Into a List
List<String> list = stream.collect(Collectors.toList());

// Into a Set (removes duplicates)
Set<String> set = stream.collect(Collectors.toSet());

// Into a Map
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(name -> name, String::length));
// key = the name itself, value = its length

// Joining strings with a delimiter
String joined = names.stream().collect(Collectors.joining(", "));
// "Charlie, Alice, Bob"

// Joining with prefix and suffix too
String joined2 = names.stream().collect(Collectors.joining(", ", "[", "]"));
// "[Charlie, Alice, Bob]"

// Counting
long count = names.stream().collect(Collectors.counting());

// Grouping — like SQL "GROUP BY"
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
// {5=[Alice, Anna], 3=[Bob], 7=[Charlie]}

// Partitioning — splits into exactly two groups: true/false
Map<Boolean, List<String>> partitioned = names.stream()
    .collect(Collectors.partitioningBy(name -> name.length() > 4));

// Summing / averaging
int totalLength = names.stream().collect(Collectors.summingInt(String::length));
double avgLength = names.stream().collect(Collectors.averagingInt(String::length));
```

## `groupingBy` with a downstream collector (combining two operations)
```java
// Group by length, but count how many are in each group (instead of listing them)
Map<Integer, Long> countByLength = names.stream()
    .collect(Collectors.groupingBy(String::length, Collectors.counting()));
```
This is a very common real-world pattern: "group by X, then aggregate within each group."

## Common interview angle
**Q: Difference between `groupingBy` and `partitioningBy`?**
`groupingBy` can produce any number of groups (keyed by whatever the classifier function returns). `partitioningBy` always produces exactly two groups, keyed by `true`/`false` from a Predicate — it's essentially a special, faster case of grouping by a boolean condition.

---

# 8. Optional — handling "no value" without null

## The problem it solves
`NullPointerException` is one of the most common bugs in Java. When a method might not have a value to return, developers historically returned `null` — and every caller had to remember to check for it, which they often forgot.

## What Optional is
`Optional<T>` is a container that either holds a value or is empty — it forces you to explicitly handle the "no value" case instead of silently returning `null` and hoping the caller checks.

```java
Optional<String> optional = Optional.of("Hello");     // wraps a known non-null value
Optional<String> empty = Optional.empty();             // represents "no value"
Optional<String> nullable = Optional.ofNullable(getValue()); // wraps something that might be null
```

## Using it safely
```java
Optional<String> name = findUserName(123);

// Bad old way would be: if (name != null) { ... } — easy to forget!

// Safe ways with Optional:
name.ifPresent(n -> System.out.println("Found: " + n));

String result = name.orElse("Unknown");          // fallback value if empty
String result2 = name.orElseGet(() -> computeDefault()); // lazy fallback (only computed if needed)
String result3 = name.orElseThrow(() -> new RuntimeException("Not found")); // throw if empty

// Chaining transformations, only applied if a value is present
Optional<Integer> length = name.map(String::length);

// Combines with filter too
Optional<String> longName = name.filter(n -> n.length() > 5);
```

## Why `orElse()` vs `orElseGet()` matters (a common gotcha)
```java
name.orElse(expensiveComputation());   // expensiveComputation() ALWAYS runs, even if name has a value!
name.orElseGet(() -> expensiveComputation()); // only runs if name is actually empty
```
`orElse()` takes an already-computed value as its argument, so Java must evaluate it eagerly regardless. `orElseGet()` takes a Supplier (lazy), only invoked if truly needed. This is a favorite trick interview question.

## Common interview angle
**Q: Should you use Optional as a method parameter or a class field?**
Generally no — Optional was designed specifically as a **return type** for methods where "no result" is a valid, expected outcome. Using it for fields or parameters adds unnecessary wrapping overhead and isn't idiomatic Java.

---

# 9. Parallel Streams

## What it is
Any stream can be made to run its operations across multiple CPU cores simply by calling `.parallelStream()` instead of `.stream()` (or calling `.parallel()` on an existing stream).

```java
long count = hugeList.parallelStream()
    .filter(n -> n % 2 == 0)
    .count();
```

## How it works under the hood
Parallel streams split the data source into chunks and process them using the common `ForkJoinPool` (the same work-stealing pool discussed in the Multithreading document), then combine the partial results back together.

## When to use it — and when NOT to
Parallel streams are **not automatically faster**. They help when:
- The dataset is large (thousands+ elements — for small lists, the overhead of splitting/merging costs more than it saves).
- The operations per element are CPU-intensive.
- Operations are stateless and independent (no shared mutable state between elements).

They can hurt performance or cause bugs when:
- The dataset is small.
- Operations have side effects on shared mutable state (like appending to a plain `ArrayList` from multiple threads at once — not thread-safe, and result can be corrupted or wrong).
- The source is I/O-bound anyway (parallel streams don't help with waiting on a database or network call).

```java
// DANGEROUS — shared ArrayList mutated concurrently, not thread-safe
List<Integer> results = new ArrayList<>();
numbers.parallelStream().forEach(n -> results.add(n * 2)); // race condition!

// SAFE — let collect() handle the thread-safe aggregation
List<Integer> results2 = numbers.parallelStream()
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

## Common interview angle
**Q: Does a parallel stream guarantee faster execution?**
No — it depends on data size, hardware core count, and whether the work is genuinely parallelizable. Always measure/benchmark before assuming parallel is better; for small collections it's often slower due to coordination overhead.

---

# 10. The New Date and Time API (`java.time`)

## The problem it solves
The old `java.util.Date` and `Calendar` classes were notoriously bad: mutable (so not thread-safe), had confusing month-numbering (months were 0-indexed — January was `0`!), and mixed up date, time, and timezone concepts in confusing ways.

## The new classes (all immutable and thread-safe)

```java
LocalDate date = LocalDate.now();              // just a date, no time: 2026-07-16
LocalTime time = LocalTime.now();              // just a time, no date: 14:30:15
LocalDateTime dateTime = LocalDateTime.now();  // both combined
ZonedDateTime zoned = ZonedDateTime.now();     // date + time + timezone

// Creating specific dates
LocalDate birthday = LocalDate.of(1995, Month.JANUARY, 15);
// or with month as a number (this time, 1 = January, unlike the old Calendar class!)
LocalDate birthday2 = LocalDate.of(1995, 1, 15);
```

## Manipulating dates (every operation returns a NEW object — immutability again)
```java
LocalDate today = LocalDate.now();
LocalDate nextWeek = today.plusWeeks(1);
LocalDate lastMonth = today.minusMonths(1);
LocalDate nextYear = today.plusYears(1);

boolean isBefore = today.isBefore(nextWeek); // true
```

## Measuring durations and periods
```java
// Period — for date-based amounts (years, months, days)
Period period = Period.between(LocalDate.of(2020,1,1), LocalDate.of(2024,6,15));
System.out.println(period.getYears() + " years, " + period.getMonths() + " months");

// Duration — for time-based amounts (hours, minutes, seconds)
Duration duration = Duration.between(LocalTime.of(9,0), LocalTime.of(17,30));
System.out.println(duration.toHours() + " hours"); // 8 hours
```

## Formatting and parsing
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy");
String formatted = LocalDate.now().format(formatter); // "16-07-2026"
LocalDate parsed = LocalDate.parse("16-07-2026", formatter);
```

## Why immutability matters here specifically
The old `Date` class was mutable — passing a `Date` object to another method meant that method could secretly modify it, causing hard-to-trace bugs, and made it unsafe to share across threads. Every `java.time` class is immutable: any "modifying" method actually returns a brand-new object, leaving the original untouched — safe by default, no defensive copying needed.

## Common interview angle
**Q: Why was `java.util.Date` replaced?**
It was mutable (thread-unsafe), had a confusing API (0-indexed months, year offset from 1900), and mixed timezone handling poorly. `java.time` fixes all of this with immutable, clearly-separated classes (LocalDate/LocalTime/LocalDateTime/ZonedDateTime) modeled after the well-regarded Joda-Time library.

---

# 11. CompletableFuture — asynchronous programming

*(Covered in more depth in the Multithreading document — here's the Java-8-feature summary.)*

## What it is
`CompletableFuture<T>` represents a value that will be available *eventually*, and lets you chain what should happen next **without blocking** the current thread to wait for it.

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // runs on a background thread
    return "Result from async task";
});

future.thenApply(result -> result.toUpperCase())
      .thenAccept(result -> System.out.println("Got: " + result));
```

## Why it's better than the old `Future`
The old `Future` interface only let you `get()` a result — which **blocks** the calling thread until it's ready. `CompletableFuture` lets you say "when this finishes, then do this next" (`.thenApply()`, `.thenCompose()`), building a non-blocking pipeline of dependent async steps — similar in spirit to how Streams chain operations, but for asynchronous computations instead of collections.

## Combining multiple async tasks
```java
CompletableFuture<Void> combined = CompletableFuture.allOf(future1, future2, future3);
// waits for all three to finish

CompletableFuture<String> combined2 = future1.thenCombine(future2, (result1, result2) -> result1 + result2);
```

## Common interview angle
**Q: What problem does CompletableFuture solve that plain Future doesn't?**
Plain Future forces you to block (`get()`) to retrieve the result, and offers no way to react automatically when it's done or chain further async work. CompletableFuture supports callbacks, chaining, combining multiple futures, and exception handling (`.exceptionally()`) — all without blocking threads unnecessarily.

---

# 12. Nashorn JavaScript Engine

## What it is
Java 8 introduced Nashorn, a JavaScript engine built into the JVM, replacing the older, slower Rhino engine. It let Java applications execute JavaScript code directly.

```java
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("nashorn");
engine.eval("print('Hello from JavaScript')");
```

## Why it mattered (and why it's less relevant today)
It allowed mixing JS scripting logic into Java applications (e.g., letting business users write small rule scripts without recompiling Java code). **Note for the record**: Nashorn was deprecated in Java 11 and removed in Java 15, so while it's a Java 8 feature worth knowing about historically, it's not something you'd build new systems on today.

---

# 13. Base64 Encoding/Decoding

## What it is
Before Java 8, encoding/decoding Base64 required third-party libraries (like Apache Commons Codec). Java 8 added a built-in, standard API.

```java
String original = "Hello World";

// Encoding
String encoded = Base64.getEncoder().encodeToString(original.getBytes());
System.out.println(encoded); // SGVsbG8gV29ybGQ=

// Decoding
byte[] decodedBytes = Base64.getDecoder().decode(encoded);
String decoded = new String(decodedBytes);
System.out.println(decoded); // Hello World
```

## Three variants provided
```java
Base64.getEncoder();        // standard Base64
Base64.getUrlEncoder();     // URL-and-filename-safe variant (uses - and _ instead of + and /)
Base64.getMimeEncoder();    // MIME-friendly, adds line breaks for email-style content
```

## Common interview angle
**Q: Is Base64 encryption?**
No — this is a common misconception. Base64 is just an **encoding** scheme to represent binary data as text; it provides zero security since anyone can decode it instantly. Never use it to "protect" sensitive data like passwords.

---

# 14. Repeating Annotations

## The problem it solves
Before Java 8, you couldn't apply the same annotation type more than once to the same element.

```java
// Before Java 8, this would need an awkward wrapper annotation
@Schedule(day="Monday")
@Schedule(day="Wednesday")   // ERROR before Java 8 — can't repeat the same annotation
public void doTask() {}
```

## The Java 8 fix
```java
@Repeatable(Schedules.class)
@interface Schedule {
    String day();
}

@interface Schedules {
    Schedule[] value();
}

// Now this works:
@Schedule(day="Monday")
@Schedule(day="Wednesday")
public void doTask() {}
```
You still need a "container" annotation (`Schedules` here) behind the scenes, but Java 8 lets you write the repeated annotations directly without manually wrapping them yourself.

## Common interview angle
This one's asked less often in interviews than the topics above — good to know it exists, but not worth deep-diving unless you're asked directly.

---

# 15. Type Annotations

## What it is
Before Java 8, annotations could only be applied to declarations (a class, a method, a field). Java 8 extended this so annotations can be applied **anywhere a type is used** — including generic type arguments, casts, and method references.

```java
List<@NonNull String> names; // annotation directly on the generic type argument
String str = (@NonNull String) obj; // annotation on a cast
```

## Why it matters
This mainly enables better static analysis tools (like the Checker Framework) to catch bugs — for example, flagging a potential null value being placed into a list that's declared to never contain nulls — *before* runtime.

---

# 16. Internal JVM Changes (not APIs, but important to know)

## Removal of PermGen — replaced by Metaspace
Before Java 8, class metadata lived in a fixed-size memory region called PermGen ("Permanent Generation"), which frequently caused `OutOfMemoryError: PermGen space` in applications that loaded lots of classes (common in app servers). Java 8 removed PermGen entirely and moved class metadata to **Metaspace**, which lives in native (off-heap) memory and **grows automatically** by default instead of hitting a fixed ceiling.

## Common interview angle
**Q: Why did Java 8 remove PermGen?**
PermGen had a fixed size that was hard to tune correctly, and applications that dynamically generated many classes (common with frameworks doing proxying/bytecode generation) would frequently run out of PermGen space. Metaspace uses native memory and expands automatically, largely eliminating this class of OutOfMemoryError.

---

# 17. Putting it all together — a realistic combined example

Here's a snippet that uses almost everything covered above in one flow, the way you'd actually write Java 8 code day-to-day:

```java
public class EmployeeReport {
    record Employee(String name, String department, double salary) {} // (Java 16+ syntax used just for brevity here)

    public static void main(String[] args) {
        List<Employee> employees = List.of(
            new Employee("Alice", "Engineering", 95000),
            new Employee("Bob", "Sales", 65000),
            new Employee("Charlie", "Engineering", 105000),
            new Employee("Anna", "Sales", 72000)
        );

        // Group employees by department, then get average salary per department
        Map<String, Double> avgSalaryByDept = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::department,
                Collectors.averagingDouble(Employee::salary)
            ));

        avgSalaryByDept.forEach((dept, avg) ->
            System.out.printf("%s: avg salary = %.2f%n", dept, avg));

        // Find the highest-paid employee overall, safely (no null risk)
        Optional<Employee> topEarner = employees.stream()
            .max(Comparator.comparingDouble(Employee::salary));

        topEarner.ifPresentOrElse(
            e -> System.out.println("Top earner: " + e.name()),
            () -> System.out.println("No employees found")
        );
    }
}
```
This single example uses: Lambdas, method references, Streams, Collectors (groupingBy + averagingDouble), Optional, and functional interfaces (Comparator as a functional interface) — all working together.

---

# Quick-reference summary table

| Feature | One-line purpose |
|---|---|
| Lambda Expressions | Write behavior/functions inline, without a full class |
| Functional Interfaces | The "type" a lambda gets assigned to (one abstract method) |
| Method References | Shorthand (`Class::method`) for a lambda that just calls one method |
| Default Methods | Add new methods to interfaces without breaking existing implementers |
| Static Methods (interfaces) | Utility methods callable directly on the interface |
| Stream API | Declarative pipeline to process collections (filter/map/reduce/collect) |
| Collectors | Pre-built strategies to gather stream results (toList, groupingBy, joining...) |
| Optional | Explicit "may have no value" container, replacing risky nulls |
| Parallel Streams | Run stream operations across multiple CPU cores automatically |
| java.time API | Immutable, thread-safe, sane date/time handling |
| CompletableFuture | Non-blocking, chainable asynchronous computation |
| Nashorn | Built-in JavaScript engine (now deprecated/removed in later versions) |
| Base64 | Standard built-in encode/decode API (not encryption!) |
| Repeatable Annotations | Apply the same annotation multiple times to one element |
| Type Annotations | Annotate type usages, not just declarations (helps static analysis tools) |
| PermGen → Metaspace | Class metadata moved to auto-growing native memory, fixing a common OOM error |

---

## Suggested study order if you're prepping for interviews
1. Lambdas + Functional Interfaces (foundation — don't skip)
2. Method References (quick, builds directly on #1)
3. Stream API + Collectors (the single most commonly asked topic — expect deep questions here)
4. Optional (commonly asked, easy to master quickly)
5. Default/static interface methods (moderate depth expected)
6. java.time API (usually lighter questions, just know the "why" over Date/Calendar)
7. CompletableFuture (asked more at product/fintech companies doing async processing)
8. Everything else (Nashorn, Base64, repeating/type annotations, PermGen) — good to recognize, rarely deep-dived
