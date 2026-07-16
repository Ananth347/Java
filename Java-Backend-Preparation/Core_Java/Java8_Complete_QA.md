# Java 8 — Complete Interview Q&A
### Basic → Advanced, every feature covered, with clear explanations

---

## How to use this document
Questions are grouped by topic, and within each topic they move from basic to hard. Read a section fully before jumping to the next — later questions in a section often build on earlier ones. This is meant to be exhaustive, so it's long on purpose — treat it as a study companion, not a one-sitting read.

**Topics covered:** Why Java 8 exists · Lambda Expressions · Functional Interfaces · Method References · Default & Static Interface Methods · The Stream API · Collectors · Optional · Parallel Streams · The new Date/Time API · CompletableFuture · Nashorn · Base64 · Repeating & Type Annotations · StringJoiner · Internal JVM changes (PermGen→Metaspace) · Mixed/scenario-based questions.

---

# SECTION 1: Why Java 8 — The Big Picture (Q1–Q6)

**Q1. What was the single biggest change Java 8 brought to the language?**
It introduced functional programming style into Java — the ability to treat behavior (code) as a value that can be passed around, stored, and returned, the same way you already do with numbers or strings. Lambda expressions, functional interfaces, and the Stream API all exist to support this one shift.

**Q2. What specific problem were lambda expressions designed to solve?**
Before Java 8, passing a piece of behavior (like "how to compare two objects" or "what to run on a new thread") required wrapping it in an anonymous inner class — several lines of boilerplate just to express one small idea. Lambdas let you write that same behavior in one line, without a class wrapper.

**Q3. Name the major features introduced in Java 8.**
Lambda expressions, functional interfaces (`java.util.function` package), method references, default and static methods in interfaces, the Stream API, the Collectors utility, Optional, the new `java.time` Date/Time API, CompletableFuture, the Nashorn JavaScript engine, a built-in Base64 API, repeating annotations, type annotations, and the removal of PermGen in favor of Metaspace.

**Q4. Is Java 8 backward compatible with earlier versions?**
Largely yes — one of Java 8's design goals was adding all this new functionality (default methods especially) without breaking existing code compiled against earlier Java versions.

**Q5. What is "declarative" vs "imperative" programming, and how does Java 8 push toward the former?**
Imperative code describes *how* to do something step by step (loops, mutable counters, manual conditionals). Declarative code describes *what* you want, letting the underlying implementation figure out the how. Streams are the clearest example — `list.stream().filter(...).map(...).collect(...)` describes the desired result, not a manual loop with an accumulator variable.

**Q6. Why did Java need to add default methods to interfaces to support Streams?**
Because adding a new abstract method like `stream()` directly to the existing `Collection` interface would have broken every class in the world that already implemented `Collection` (they'd all suddenly be missing a required method). Default methods let Java add `stream()`, `forEach()`, and similar new behavior with a working implementation baked in, so existing classes keep compiling without changes.

---

# SECTION 2: Lambda Expressions (Q7–Q22)

## Basic

**Q7. What is a lambda expression?**
A compact, anonymous way to write a function — a block of code with parameters and a body, but no name, no declared return type, and no surrounding class. Syntax: `(parameters) -> expression` or `(parameters) -> { statements; }`.

**Q8. Write a lambda that takes two integers and returns their sum.**
```java
(a, b) -> a + b
```

**Q9. Do you need to specify parameter types in a lambda?**
No, usually — the compiler infers them from context (this is called **target typing**). You can specify them explicitly if you want extra clarity, but it's rarely necessary: `(int a, int b) -> a + b` works the same as `(a, b) -> a + b`.

**Q10. When do you need curly braces `{ }` in a lambda body?**
When the body has more than one statement, or when you need an explicit `return`. A single-expression body doesn't need braces or `return` — the expression's value is returned automatically.
```java
x -> x * 2                    // no braces needed
x -> { return x * 2; }        // equivalent, with braces
x -> {                        // braces required — multiple statements
    int result = x * 2;
    return result;
}
```

**Q11. Can a lambda have zero parameters?**
Yes: `() -> System.out.println("Hello")` — the empty parentheses represent no parameters.

## Intermediate

**Q12. What is "effectively final," and why does it matter for lambdas?**
A local variable is effectively final if it's assigned once and never reassigned afterward (even without the explicit `final` keyword). A lambda can only capture (use) local variables from its enclosing scope if they are effectively final. This exists because the lambda might be executed later, or on a completely different thread — Java needs a guaranteed-stable value to safely capture at the point the lambda is created, and a variable that could still change wouldn't provide that guarantee.
```java
int factor = 10; // effectively final — never reassigned
Function<Integer, Integer> multiply = x -> x * factor; // OK

int counter = 0;
counter = 5; // reassigned — NOT effectively final anymore
Function<Integer, Integer> bad = x -> x * counter; // COMPILE ERROR
```

**Q13. Can a lambda modify a variable from its enclosing scope?**
No — this is directly related to Q12. A lambda can only *read* effectively final local variables, never reassign them. To work around this in practice, developers use a mutable holder object (like a single-element array, or an `AtomicInteger`) if they genuinely need shared mutable state, though this is generally a sign you should restructure the code instead.

**Q14. What's the difference between a lambda and an anonymous inner class, beyond syntax?**
Beyond the shorter syntax, lambdas don't create a new `.class` file at compile time the way anonymous inner classes do (they're implemented differently under the hood, using a mechanism called `invokedynamic`), and `this` inside a lambda refers to the *enclosing* class instance, whereas `this` inside an anonymous inner class refers to the anonymous class instance itself.
```java
class Demo {
    Runnable lambdaVersion = () -> System.out.println(this); // "this" = Demo instance
    Runnable anonymousVersion = new Runnable() {
        public void run() { System.out.println(this); } // "this" = the anonymous Runnable instance
    };
}
```

**Q15. Can a lambda throw a checked exception?**
Only if the functional interface's abstract method signature declares that it can throw that exception. Most built-in functional interfaces (`Function`, `Predicate`, etc.) don't declare any checked exceptions, so a lambda assigned to them can't throw one directly — you'd need to catch it inside the lambda body, or use a custom functional interface that declares `throws`.

**Q16. Are lambda expressions objects in Java?**
Yes — every lambda is, under the hood, an instance of whatever functional interface it's assigned to. `Runnable r = () -> {}` creates an object that implements `Runnable`.

## Advanced

**Q17. How are lambdas actually implemented internally — are they just syntactic sugar for anonymous classes?**
No, this is a common misconception. Lambdas are compiled using an `invokedynamic` instruction, and the actual implementation class is generated **at runtime** (via a mechanism called `LambdaMetafactory`), not at compile time as a separate `.class` file the way anonymous inner classes are. This generally makes lambdas more lightweight to create (no separate class-loading overhead per lambda definition).

**Q18. Why can't a lambda access a non-effectively-final local variable, in deeper technical terms?**
Because the lambda might outlive the method call that created it (e.g., it's stored and invoked later, or run on another thread after the original method has already returned and its stack frame is gone). Java captures local variables *by value* at the point of lambda creation — if the variable could change afterward, the lambda's captured copy would be silently stale or inconsistent, so Java simply disallows this at compile time rather than allow an ambiguous runtime situation.

**Q19. What is variable capture, and does a lambda capture instance fields the same way it captures local variables?**
"Capture" refers to a lambda using a variable from its surrounding scope. Instance fields (and static fields) are captured *by reference*, not by value — meaning a lambda CAN see and even be affected by later changes to an instance field, unlike local variables, since instance fields live on the heap tied to the object, not on the stack frame that might disappear.
```java
class Counter {
    int count = 0;
    Runnable increment = () -> count++; // fine — count is a field, not a local variable, mutation allowed
}
```

**Q20. What's the scope of `this` inside a lambda versus inside a traditional anonymous class, and why does this distinction matter practically?**
Inside a lambda, `this` refers to the enclosing instance (lambdas don't introduce their own `this`). Inside an anonymous class, `this` refers to the anonymous class instance itself. This matters practically when you want a callback to refer back to the outer object naturally (lambdas make this easy and intuitive) versus needing the callback object's own identity (which requires an anonymous class or a named class instead).

**Q21. Can you overload a method that accepts different functional interfaces, and pass the same lambda to both?**
This can create ambiguity errors if the functional interfaces have compatible signatures, since the compiler can't always tell which target type you intend. In practice, if two overloaded methods take different functional interfaces with the same parameter/return shape (e.g., overloads accepting `Runnable` and `Callable<Void>`), passing a bare lambda can cause a compile-time ambiguity error, requiring an explicit cast to disambiguate: `(Runnable)(() -> {})`.

**Q22. What is a lambda's "target type," and why does it matter for type inference?**
The target type is the functional interface a lambda is being assigned/passed to — since a lambda itself has no inherent type (unlike, say, a String literal, which is always a String), the compiler infers everything about the lambda's parameter types and behavior from the context it's used in. This is why the exact same-looking lambda `x -> x + 1` can become a `Function<Integer, Integer>` in one context and a `UnaryOperator<Integer>` in another — its meaning depends entirely on where it's assigned.

---

# SECTION 3: Functional Interfaces (Q23–Q38)

## Basic

**Q23. What is a functional interface?**
An interface with exactly one abstract method (it can have any number of default or static methods in addition). This single abstract method is what a lambda expression implements when assigned to that interface.

**Q24. What does the `@FunctionalInterface` annotation do?**
It's optional, but tells the compiler to enforce the "exactly one abstract method" rule — if you accidentally add a second abstract method, compilation fails immediately with a clear error, instead of only failing later wherever someone tries to use a lambda with it.

**Q25. Can a functional interface have default or static methods?**
Yes, any number of them — the "exactly one abstract method" rule only applies to *abstract* methods. Default and static methods have bodies and don't count toward that limit.

**Q26. What is `Function<T, R>`, and give an example.**
Takes an input of type `T`, returns a result of type `R`. Method: `R apply(T t)`.
```java
Function<String, Integer> stringLength = s -> s.length();
System.out.println(stringLength.apply("hello")); // 5
```

**Q27. What is `Predicate<T>`, and give an example.**
Takes an input of type `T`, returns a `boolean`. Method: `boolean test(T t)`. Used for filtering/conditions.
```java
Predicate<Integer> isPositive = n -> n > 0;
System.out.println(isPositive.test(-5)); // false
```

**Q28. What is `Consumer<T>`, and give an example.**
Takes an input of type `T`, returns nothing — used purely for side effects (like printing). Method: `void accept(T t)`.
```java
Consumer<String> printer = s -> System.out.println("Value: " + s);
printer.accept("hello"); // Value: hello
```

**Q29. What is `Supplier<T>`, and give an example.**
Takes no input, returns a value of type `T` — essentially a "factory" or lazy value provider. Method: `T get()`.
```java
Supplier<Double> randomNumber = () -> Math.random();
System.out.println(randomNumber.get());
```

## Intermediate

**Q30. What is `BiFunction<T, U, R>`? How does it differ from `Function`?**
Takes TWO inputs (`T` and `U`), returns a result `R`. Method: `R apply(T t, U u)`. `Function` only takes one input; `BiFunction` is for operations needing two arguments, like adding two numbers.
```java
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(3, 4)); // 7
```

**Q31. What is `UnaryOperator<T>` and how does it relate to `Function<T, R>`?**
`UnaryOperator<T>` is a specialized `Function<T, T>` — where the input and output are the exact same type. It's used when a function transforms a value but keeps it the same type (e.g., doubling a number, uppercasing a string).
```java
UnaryOperator<Integer> square = x -> x * x;
```

**Q32. What is `BinaryOperator<T>` and how does it relate to `BiFunction`?**
`BinaryOperator<T>` is a specialized `BiFunction<T, T, T>` — both inputs and the output are the same type. Commonly used with `reduce()` in streams (e.g., summing, finding max).
```java
BinaryOperator<Integer> sum = (a, b) -> a + b;
```

**Q33. Are there primitive-specialized versions of these functional interfaces? Why do they exist?**
Yes — `IntFunction`, `IntPredicate`, `IntConsumer`, `IntSupplier`, `ToIntFunction`, `IntUnaryOperator`, `IntBinaryOperator`, and similar variants exist for `int`, `long`, and `double`. They exist purely for performance: using the generic `Function<Integer, Integer>` would require **autoboxing** every primitive into an `Integer` object and back, which has real memory and CPU overhead when done repeatedly (e.g., across millions of stream elements). The primitive-specialized interfaces avoid that boxing entirely.

**Q34. How can you combine/chain multiple `Function` objects together?**
Using the default methods `andThen()` and `compose()`:
```java
Function<Integer, Integer> addOne = x -> x + 1;
Function<Integer, Integer> square = x -> x * x;

Function<Integer, Integer> combined1 = addOne.andThen(square); // addOne FIRST, then square: (x+1)^2
Function<Integer, Integer> combined2 = addOne.compose(square); // square FIRST, then addOne: x^2 + 1

System.out.println(combined1.apply(2)); // (2+1)^2 = 9
System.out.println(combined2.apply(2)); // 2^2+1 = 5
```

**Q35. How can you combine multiple `Predicate` objects?**
Using the default methods `and()`, `or()`, and `negate()`:
```java
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;

Predicate<Integer> positiveAndEven = isPositive.and(isEven);
Predicate<Integer> positiveOrEven = isPositive.or(isEven);
Predicate<Integer> isNegative = isPositive.negate();
```

**Q36. What is `BiConsumer<T, U>`?**
Takes two inputs, returns nothing (side-effect only). Method: `void accept(T t, U u)`. Commonly used in `Map.forEach()`:
```java
map.forEach((key, value) -> System.out.println(key + " = " + value));
```

## Advanced

**Q37. Why does Java provide both `Supplier<T>` and just calling a method directly? What's the actual benefit of laziness here?**
A `Supplier` defers computation until `.get()` is actually called — this matters when you want to *describe* a potentially expensive computation without necessarily running it. For example, `Optional.orElseGet(Supplier)` only invokes the supplier if the Optional is actually empty, whereas passing an already-computed value (`orElse(value)`) forces that computation to happen every single time, whether or not it's needed. Suppliers turn "when do I compute this" into something the caller controls, rather than something forced eagerly at the call site.

**Q38. Can you write your own custom functional interface, and when would you need to instead of using a built-in one?**
Yes:
```java
@FunctionalInterface
interface TriFunction<A, B, C, R> {
    R apply(A a, B b, C c);
}
```
You'd write a custom one when a built-in interface doesn't fit your needed shape — for example, none of the standard `java.util.function` interfaces take **three** arguments, so if your logic genuinely needs three inputs, a custom interface like this is the clean solution rather than forcing an awkward workaround with existing types.

---

# SECTION 4: Method References (Q39–Q48)

## Basic

**Q39. What is a method reference?**
A shorthand syntax (`::`) for a lambda expression that does nothing except call one existing method. If your lambda body is purely "call this method with the given arguments," you can replace it with a method reference for better readability.

**Q40. Rewrite this lambda as a method reference: `s -> s.length()`**
```java
String::length
```

**Q41. Rewrite this lambda as a method reference: `s -> Integer.parseInt(s)`**
```java
Integer::parseInt
```

## Intermediate

**Q42. What are the four kinds of method references?**
1. Static method reference: `ClassName::staticMethod`
2. Instance method reference on a particular (already existing) object: `instance::method`
3. Instance method reference on an arbitrary object of a particular type: `ClassName::instanceMethod`
4. Constructor reference: `ClassName::new`

**Q43. Give an example of each of the four kinds.**
```java
// 1. Static method reference
Function<String, Integer> f1 = Integer::parseInt;

// 2. Instance method reference on a specific existing object
String prefix = "Hello, ";
Function<String, String> f2 = prefix::concat;

// 3. Instance method reference on an arbitrary object of a type
Function<String, Integer> f3 = String::length; // the "arbitrary object" becomes the implicit first parameter

// 4. Constructor reference
Supplier<ArrayList<String>> f4 = ArrayList::new;
```

**Q44. In `Function<String, Integer> f = String::length;`, where does the "arbitrary object" (kind #3) come from?**
The string that gets passed into `f.apply(someString)` becomes the object the instance method `length()` is called ON, rather than being passed as an explicit argument. So `f.apply("hello")` is equivalent to `"hello".length()` — the input parameter effectively becomes the receiver of the method call.

**Q45. Can a method reference point to a constructor that takes arguments?**
Yes — the functional interface's method signature determines which constructor overload gets matched.
```java
BiFunction<String, Integer, StringBuilder> f = StringBuilder::new; // matches StringBuilder(String, but wait — actually matches a 2-arg constructor if one exists that fits
Function<Integer, ArrayList<String>> f2 = ArrayList::new; // matches the ArrayList(int initialCapacity) constructor
```

## Advanced

**Q46. When can a lambda NOT be converted into a method reference?**
Whenever the lambda body does *more* than a single, direct method call with the same arguments in the same order — for example, `x -> x.length() + 1` cannot become a method reference, because there's additional logic (`+1`) beyond simply delegating to `length()`.

**Q47. Do method references have any performance advantage over an equivalent lambda?**
Not meaningfully in most cases — they compile down to essentially the same `invokedynamic`-based mechanism as lambdas. The benefit of method references is almost entirely about **readability**, not performance.

**Q48. Can you use a method reference to refer to an instance method of `this` (the current object)?**
Yes — this falls under "instance method reference on a particular object" (kind #2), where the particular object happens to be `this`:
```java
class Greeter {
    void greet(String name) { System.out.println("Hello " + name); }
    void run() {
        Consumer<String> c = this::greet; // refers to THIS object's greet() method
        c.accept("Ananth");
    }
}
```

---

# SECTION 5: Default and Static Methods in Interfaces (Q49–Q60)

## Basic

**Q49. What is a default method in an interface?**
A method inside an interface that has an actual body/implementation, marked with the `default` keyword. Implementing classes automatically get this behavior without being forced to implement it themselves, though they CAN override it if they want different behavior.

**Q50. Why were default methods introduced?**
Purely to let existing interfaces (like `List`, `Collection`, `Map`) gain new methods (like `stream()`, `forEach()`, `sort()`) without breaking every class that already implemented those interfaces before Java 8 existed. Without default methods, adding any new abstract method to `Collection` would have broken the entire Java ecosystem's existing code.

**Q51. Write an interface with one abstract method and one default method.**
```java
interface Greeting {
    void sayHello(); // abstract — must be implemented

    default void sayGoodbye() { // default — has a body, optional to override
        System.out.println("Goodbye!");
    }
}
```

**Q52. Can a class override a default method?**
Yes, exactly like it would override any regular inherited method — just declare a method with the same signature in the implementing class.

## Intermediate

**Q53. What is a static method in an interface, and how do you call it?**
A method inside an interface that belongs to the interface itself (not to instances of implementing classes), marked `static` and containing its own body. You call it directly on the interface name, like `InterfaceName.methodName()` — never on an object.
```java
interface MathUtil {
    static int square(int x) { return x * x; }
}
MathUtil.square(5); // 25 — called on the interface directly, no instance needed
```

**Q54. Can a static interface method be overridden by an implementing class?**
No — static methods aren't inherited in the same polymorphic way instance methods are. An implementing class can define its own static method with the same name, but that's just a separate, unrelated method, not an "override" in the traditional sense (no dynamic dispatch involved).

**Q55. What is the "diamond problem," and can it occur in Java with default methods?**
Yes — if a class implements two interfaces that both declare a default method with the *identical* signature, Java can't automatically decide which one to inherit, so it forces you to resolve the ambiguity explicitly, or the code won't compile.
```java
interface A { default void greet() { System.out.println("A"); } }
interface B { default void greet() { System.out.println("B"); } }

class C implements A, B {
    public void greet() {
        A.super.greet(); // explicitly pick A's version (or B.super.greet(), or write entirely new logic)
    }
}
```

**Q56. What happens if a class implements an interface with a default method, and also extends a superclass with a regular method of the same signature?**
The superclass's method always wins — "class wins over interface" is the tie-breaking rule. A concrete method inherited from a class takes priority over a default method from an interface, regardless of which one is declared "closer" in the hierarchy.

## Advanced

**Q57. Can a default method call an abstract method declared in the same interface?**
Yes — this is actually a common and useful pattern, letting a default method provide reusable logic that's partly based on behavior the implementing class must still define.
```java
interface Shape {
    double area(); // abstract — implementer must define this

    default void printArea() { // default — provides reusable logic, but depends on area()
        System.out.println("Area: " + area());
    }
}
```

**Q58. Why can't default methods be used to add new instance fields to an interface?**
Interfaces still cannot hold instance state (fields) — default methods only add *behavior*, not *data*. This preserves a core distinction between interfaces (pure contracts/behavior) and classes (which can hold both state and behavior); allowing fields would have blurred that line and introduced far more complexity (like needing a constructor mechanism for interfaces).

**Q59. Are default methods a replacement for abstract classes?**
Not really — abstract classes can still hold instance fields, constructors, and non-public members, none of which interfaces (even with default methods) can do. Default methods solve a narrower, specific problem: letting interfaces evolve over time without breaking existing implementers. For genuinely stateful shared behavior, an abstract class is still the more appropriate tool.

**Q60. Is a `private` method allowed inside an interface? (This was actually a Java 9 feature, but commonly confused with Java 8 — good to know the distinction.)**
No — private interface methods were introduced in **Java 9**, not Java 8. Java 8 only added `default` and `static` methods to interfaces; private interface methods (used to share common code between default methods without exposing it publicly) came a version later. This is a common point of confusion worth clarifying in an interview if it comes up.

---

# SECTION 6: The Stream API (Q61–Q95 — the largest section, since this is the most-asked topic)

## Basic

**Q61. What is a Stream in Java 8?**
A pipeline that describes a sequence of operations to perform over a source of data (a Collection, array, or generator) — it does NOT store any data itself, and it's **lazy**, meaning none of the described operations actually execute until a terminal operation is called.

**Q62. How do you create a Stream from a List?**
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
```

**Q63. What are the three parts of every stream pipeline?**
1. **Source** — where the data comes from (e.g., `list.stream()`).
2. **Intermediate operations** — transform the stream, return another stream, and are lazy (e.g., `filter()`, `map()`).
3. **Terminal operation** — triggers actual execution and produces a result or side effect (e.g., `collect()`, `forEach()`, `count()`).

**Q64. What does `filter()` do?**
Keeps only the elements that satisfy a given `Predicate`, discarding the rest.
```java
list.stream().filter(s -> s.startsWith("a"))
```

**Q65. What does `map()` do?**
Transforms each element into something else, using a given `Function` — the resulting stream may hold a completely different type than the original.
```java
list.stream().map(String::toUpperCase)
```

**Q66. What does `forEach()` do?**
A terminal operation that performs a given action (a `Consumer`) on each element — similar to a for-loop, but returns nothing.
```java
list.stream().forEach(System.out::println);
```

**Q67. What does `collect()` do?**
A terminal operation that gathers the stream's elements into a final result — typically a collection (List, Set, Map) or a single combined value, using a `Collector` (see Section 7).
```java
List<String> result = list.stream().collect(Collectors.toList());
```

**Q68. Is a Stream a data structure like a List or Set?**
No — Streams don't store any elements. They describe a computation to be performed on data that lives somewhere else (a Collection, array, etc.).

## Intermediate

**Q69. Why are intermediate operations called "lazy"? Demonstrate with an example.**
Because they don't actually execute any code until a terminal operation triggers the whole pipeline to run.
```java
Stream<String> stream = list.stream()
    .filter(s -> {
        System.out.println("Filtering: " + s);
        return s.startsWith("a");
    });
System.out.println("Nothing has printed yet!");
stream.forEach(System.out::println); // NOW the filtering prints happen, right before each match is printed
```

**Q70. Can you reuse a Stream after calling a terminal operation on it?**
No — once a terminal operation runs, the stream is considered consumed. Any further operation on that same stream object throws `IllegalStateException: stream has already been operated upon or closed`. You'd need to create a fresh stream from the original source to run another pipeline.

**Q71. What does `sorted()` do, and how do you sort in a custom order?**
Without arguments, it sorts using the elements' natural order (`Comparable`). With a `Comparator` argument, it sorts using your custom logic.
```java
list.stream().sorted(); // natural order
list.stream().sorted(Comparator.reverseOrder()); // custom order
```

**Q72. What does `distinct()` do, and how does it decide what counts as a duplicate?**
Removes duplicate elements from the stream, using each element's `equals()` method to determine what counts as "the same."

**Q73. What do `limit(n)` and `skip(n)` do? Give a practical combined use case.**
`limit(n)` keeps only the first n elements; `skip(n)` discards the first n elements. Combined, they implement pagination:
```java
// "Page 3" of 5 items per page (i.e., skip the first 10, then take the next 5)
list.stream().skip(10).limit(5);
```

**Q74. What is `flatMap()`, and how is it different from `map()`?**
`map()` transforms each element into exactly one new element (a 1-to-1 transformation), even if that new element is itself a collection — resulting in a "stream of collections." `flatMap()` transforms each element into a *stream* and then flattens all of those individual streams into one single combined stream — useful when you have nested structures you want merged into one flat sequence.
```java
List<List<Integer>> nested = Arrays.asList(Arrays.asList(1, 2), Arrays.asList(3, 4));

// Using map() — result is a Stream<List<Integer>>, still nested
nested.stream().map(innerList -> innerList); 

// Using flatMap() — result is a flat Stream<Integer>: 1, 2, 3, 4
nested.stream().flatMap(List::stream).collect(Collectors.toList());
```

**Q75. What does `reduce()` do? Explain with an example.**
Combines all elements of a stream into a single value, by repeatedly applying a combining function. Think of it as "folding" the stream down.
```java
int sum = Stream.of(1, 2, 3, 4).reduce(0, (a, b) -> a + b); // starts at 0, keeps adding: 10
```
The first argument (`0`) is the starting/identity value; the second is the combining logic applied between the running total and each new element.

**Q76. What's the difference between `reduce(identity, accumulator)` and the no-identity `reduce(accumulator)` overload that returns `Optional`?**
`reduce(identity, accumulator)` always returns a value directly (using the identity as a starting point, and as the result if the stream is empty). The overload without an identity, `reduce(BinaryOperator)`, returns an `Optional<T>` instead — because if the stream is completely empty, there's no sensible value to return, so it must represent "no result" explicitly rather than guessing a default.

**Q77. What do `anyMatch()`, `allMatch()`, and `noneMatch()` do?**
All three are terminal operations returning a `boolean`, checking a Predicate against the stream's elements:
- `anyMatch()`: true if AT LEAST ONE element matches.
- `allMatch()`: true if EVERY element matches.
- `noneMatch()`: true if NO element matches.
```java
boolean hasNegative = numbers.stream().anyMatch(n -> n < 0);
boolean allPositive = numbers.stream().allMatch(n -> n > 0);
```

**Q78. What do `findFirst()` and `findAny()` do, and why do they both return `Optional`?**
Both return the first (or "any") matching element, wrapped in an `Optional`, because the stream might have no elements at all — Optional forces you to explicitly handle that "nothing found" case rather than risking a `NullPointerException`.

**Q79. What is the difference between `findFirst()` and `findAny()`?**
`findFirst()` always returns the first element encountered in encounter order (relevant for ordered streams like those from a List). `findAny()` makes no such guarantee — it may return any matching element, which allows for better performance specifically in **parallel streams**, since it doesn't have to coordinate to determine which element was "technically first."

**Q80. What does `count()` do, and what type does it return?**
A terminal operation that returns the number of elements in the stream, as a `long` (not `int`).

**Q81. How do you create a Stream directly from values, without a Collection?**
```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
IntStream intStream = IntStream.range(1, 10); // 1 through 9, exclusive of 10
IntStream inclusiveRange = IntStream.rangeClosed(1, 10); // 1 through 10, inclusive
```

## Advanced

**Q82. What are `IntStream`, `LongStream`, and `DoubleStream`, and why do they exist separately from `Stream<Integer>`?**
They're primitive-specialized stream types, existing purely for performance — using `Stream<Integer>` requires boxing every `int` into an `Integer` object, adding overhead. `IntStream` (and similarly `LongStream`/`DoubleStream`) works with raw primitives directly, avoiding that boxing cost, and provides extra numeric-specific methods like `sum()`, `average()`, and `max()` that a generic `Stream<Integer>` doesn't have built in.
```java
int total = IntStream.of(1, 2, 3, 4).sum(); // 10
OptionalDouble avg = IntStream.of(1, 2, 3, 4).average(); // OptionalDouble, since average of an empty stream is undefined
```

**Q83. How do you convert between a regular `Stream<Integer>` and an `IntStream`?**
```java
IntStream intStream = list.stream().mapToInt(Integer::intValue); // boxed Stream -> primitive IntStream
Stream<Integer> boxedStream = intStream.boxed(); // primitive IntStream -> boxed Stream<Integer>
```

**Q84. Explain the "stateless vs stateful" distinction for intermediate operations, with examples of each.**
**Stateless** operations (`filter`, `map`) process each element independently — they don't need to know about any other elements to decide what to do with the current one. **Stateful** operations (`sorted`, `distinct`, `limit`) need information about other elements (the whole stream, or elements seen so far) before they can produce output — e.g., `sorted()` can't output anything until it's seen every element, since any later element could belong earlier in the sorted order. This distinction matters for performance, especially with large or infinite streams, and for how well an operation parallelizes.

**Q85. Can a Stream be infinite? How do you create one, and how do you safely consume it?**
Yes — using `Stream.iterate()` or `Stream.generate()`, which produce elements on-demand rather than from a fixed source.
```java
Stream<Integer> infinite = Stream.iterate(1, n -> n + 1); // 1, 2, 3, 4, ... forever
Stream<Double> randoms = Stream.generate(Math::random);   // infinite stream of random doubles

// MUST be combined with something that limits it, like limit(), or it will run forever:
infinite.limit(5).forEach(System.out::println); // safely prints just 1, 2, 3, 4, 5
```

**Q86. What is short-circuiting in the context of streams, and which operations exhibit it?**
A short-circuiting operation can produce a result (or terminate the whole pipeline) without necessarily processing every single element — critical for making infinite streams usable. `limit()`, `anyMatch()`, `findFirst()`, and `findAny()` are all short-circuiting: e.g., `anyMatch()` stops checking as soon as it finds one match, rather than scanning everything.

**Q87. What is the difference between `Stream.of(array)` and `Arrays.stream(array)`?**
For object arrays, they behave the same. But `Stream.of()` with a primitive array (like `int[]`) treats the *entire array* as a single element (producing a `Stream<int[]>` with one element!), whereas `Arrays.stream(intArray)` correctly produces an `IntStream` of the individual primitive elements. This is a subtle, commonly-tested gotcha.
```java
int[] arr = {1, 2, 3};
Stream.of(arr);      // Stream<int[]> containing ONE element (the whole array)
Arrays.stream(arr);  // IntStream containing THREE elements: 1, 2, 3
```

**Q88. How would you process a stream of Strings, split each into words, and get one flat stream of all words? (A classic flatMap use case.)**
```java
List<String> sentences = Arrays.asList("Hello world", "Java 8 streams");
List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList());
// [Hello, world, Java, 8, streams]
```

**Q89. What's the difference between `peek()` and `forEach()`?**
`peek()` is an **intermediate** operation, meant for debugging/inspecting elements as they flow through the pipeline without altering the stream — it returns another stream so you can keep chaining. `forEach()` is a **terminal** operation, ending the pipeline. Importantly, `peek()` is lazy like other intermediate operations — if there's no terminal operation afterward, `peek()`'s code never actually runs.

**Q90. Is stream processing guaranteed to happen in encounter order?**
For sequential streams sourced from ordered collections (like a List), yes, generally, for order-sensitive operations. For parallel streams, order is NOT guaranteed for operations like `forEach()` (elements might be processed out of order across threads) unless you specifically use `forEachOrdered()`, which forces order at some cost to parallelism benefits.

**Q91. Can a stream pipeline modify the original source collection safely while it's running?**
No — modifying the underlying source (e.g., calling `list.add()`) while a stream derived from it is actively being processed can throw `ConcurrentModificationException`, exactly like modifying a collection during a regular fail-fast iteration.

**Q92. Explain what happens step-by-step when this code runs:**
```java
List<String> result = Stream.of("banana", "apple", "cherry")
    .filter(s -> s.length() > 5)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```
1. Source: a stream of `"banana"`, `"apple"`, `"cherry"`.
2. `filter(s -> s.length() > 5)`: keeps only `"banana"` (6 letters) and `"cherry"` (6 letters) — `"apple"` (5 letters) is dropped.
3. `map(String::toUpperCase)`: transforms them to `"BANANA"` and `"CHERRY"`.
4. `sorted()`: alphabetically orders them: `"BANANA"`, `"CHERRY"`.
5. `collect(Collectors.toList())`: gathers the final result into a `List<String>`: `["BANANA", "CHERRY"]`.
Nothing actually executes until step 5 triggers the whole lazy pipeline to run.

**Q93. What happens if you call a terminal operation on an empty stream?**
It behaves sensibly according to the operation's semantics: `count()` returns 0, `collect(Collectors.toList())` returns an empty list, `findFirst()` returns an empty `Optional`, `reduce()` (without identity) returns an empty `Optional`, and `reduce(identity, ...)` returns the identity value itself.

**Q94. How would you sum only the even numbers in a list, using streams?**
```java
int sum = numbers.stream()
    .filter(n -> n % 2 == 0)
    .mapToInt(Integer::intValue)
    .sum();
```

**Q95. What's a potential downside of overusing streams for very simple operations?**
For trivial operations (like a simple loop with one line of logic), a stream pipeline can sometimes be less readable and slightly slower (due to the overhead of creating stream objects and lambda call sites) than a plain for-loop. Streams shine most with multi-step transformations, not single trivial operations — using them everywhere "because Java 8" isn't automatically better code.

---

# SECTION 7: Collectors (Q96–Q112)

## Basic

**Q96. What is `Collectors.toList()` used for?**
Gathers the elements of a stream into a `List`, as the argument to the `collect()` terminal operation.
```java
List<String> list = stream.collect(Collectors.toList());
```

**Q97. What is `Collectors.toSet()`?**
Gathers stream elements into a `Set`, automatically removing duplicates (via `equals()`/`hashCode()`).

**Q98. What is `Collectors.joining()`, and what are its overloads?**
Concatenates a stream of Strings into one combined String.
```java
Collectors.joining()                 // no separator: "abc"
Collectors.joining(", ")             // with delimiter: "a, b, c"
Collectors.joining(", ", "[", "]")   // with delimiter, prefix, and suffix: "[a, b, c]"
```

**Q99. What is `Collectors.counting()`?**
Returns the count of elements in the stream, as a `long` — often used as a "downstream" collector inside `groupingBy` (see Q104).

## Intermediate

**Q100. What is `Collectors.toMap()`, and what happens if two elements produce the same key?**
Builds a `Map` from stream elements, given a key-extraction function and a value-extraction function.
```java
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(name -> name, String::length));
```
If two elements produce a duplicate key, it throws `IllegalStateException` by default — you must provide a third argument (a merge function) to explicitly say how to combine values when keys collide:
```java
Collectors.toMap(name -> name, String::length, (existing, replacement) -> existing) // keep the first one seen
```

**Q101. What is `Collectors.groupingBy()`? Give a basic example.**
Groups stream elements by a classification function, producing a `Map` where keys are the classifier's results and values are Lists of matching elements — similar to a SQL "GROUP BY."
```java
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
// {3=[Bob], 5=[Alice, Anna], 7=[Charlie]}
```

**Q102. What is `Collectors.partitioningBy()`? How is it different from `groupingBy()`?**
Splits stream elements into exactly TWO groups based on a boolean Predicate — keyed by `true` and `false`. Unlike `groupingBy()`, which can produce any number of groups depending on the classifier, `partitioningBy()` always produces exactly these two, even if one group ends up empty.
```java
Map<Boolean, List<String>> partitioned = names.stream()
    .collect(Collectors.partitioningBy(name -> name.length() > 4));
```

**Q103. What are `Collectors.summingInt()`, `averagingInt()`, `summingDouble()`, etc.?**
Numeric aggregation collectors that compute a sum or average directly from a stream, using a function to extract the numeric value from each element.
```java
int totalLength = names.stream().collect(Collectors.summingInt(String::length));
double avgLength = names.stream().collect(Collectors.averagingInt(String::length));
```

**Q104. What is a "downstream collector," and how is it used with `groupingBy()`? Give an example.**
A downstream collector lets you apply a SECOND collecting operation *within* each group produced by `groupingBy()`, rather than just getting a plain List of each group's members.
```java
// Instead of listing every name in each length-group, count how many are in each group
Map<Integer, Long> countByLength = names.stream()
    .collect(Collectors.groupingBy(String::length, Collectors.counting()));
```

**Q105. Write a Collectors expression to group employees by department, then get the average salary within each department.**
```java
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));
```

## Advanced

**Q106. What is `Collectors.toUnmodifiableList()` (Java 10+, but relevant to know when discussing Collectors evolution)?**
Produces an immutable List directly, instead of the mutable List that `Collectors.toList()` traditionally returns — worth knowing that this specific unmodifiable variant is a later addition, not originally part of Java 8's Collectors, in case you're asked to distinguish exact version history.

**Q107. Can you nest `groupingBy()` calls to group by multiple levels?**
Yes, by using another `groupingBy()` as the downstream collector:
```java
Map<String, Map<String, List<Employee>>> byDeptThenLevel = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment,
             Collectors.groupingBy(Employee::getLevel)));
```

**Q108. What is `Collectors.mapping()`, and why would you use it as a downstream collector?**
Transforms each element within a group before it's collected — useful when you want to group by one thing, but collect just a *specific field* of each matching element rather than the whole object.
```java
// Group employees by department, but collect just their NAMES, not full Employee objects
Map<String, List<String>> namesByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment,
             Collectors.mapping(Employee::getName, Collectors.toList())));
```

**Q109. What is `Collectors.reducing()`, and how does it compare to the stream's own `reduce()` method?**
Performs a reduction operation (like `reduce()`) but packaged as a `Collector`, so it can be used as a downstream collector within `groupingBy()` or similar — something the standalone `reduce()` terminal operation can't directly do, since `reduce()` operates on a whole stream, not on sub-groups within one.

**Q110. What's the difference between `Collectors.summarizingInt()` and `Collectors.summingInt()`?**
`summingInt()` returns just a single total sum value. `summarizingInt()` returns an `IntSummaryStatistics` object bundling multiple statistics together at once — count, sum, min, max, AND average — computed in a single pass, more efficient than computing each of those separately with multiple collectors.
```java
IntSummaryStatistics stats = names.stream().collect(Collectors.summarizingInt(String::length));
stats.getMax(); stats.getMin(); stats.getAverage(); stats.getSum(); stats.getCount();
```

**Q111. How would you implement a custom Collector from scratch (conceptually, what four things does it need)?**
A custom `Collector` needs: a **supplier** (creates the initial empty result container, e.g., a new ArrayList), an **accumulator** (defines how to add one element into that container), a **combiner** (defines how to merge two partial containers together — needed for parallel streams), and a **finisher** (optional final transformation applied to the accumulated result before returning it).

**Q112. Why does `Collectors.toMap()` throw an exception on duplicate keys by default, rather than silently overwriting like a normal `Map.put()` would?**
This is a deliberate safety design: silently overwriting could hide a genuine bug (two elements accidentally mapping to the same key when you expected all keys to be unique) — throwing forces you to consciously decide, via the optional merge-function overload, exactly how duplicates should be handled, rather than an unnoticed silent data loss.

---

# SECTION 8: Optional (Q113–Q127)

## Basic

**Q113. What problem does `Optional` solve?**
It addresses the widespread problem of `NullPointerException` caused by methods returning `null` to represent "no result," which callers routinely forget to check for. `Optional<T>` is a container that's either present (holding a value) or empty, forcing callers to explicitly handle the "no value" case through its API rather than silently risking a null dereference.

**Q114. How do you create an Optional?**
```java
Optional<String> present = Optional.of("Hello");       // value must be non-null, or throws NPE immediately
Optional<String> empty = Optional.empty();               // explicitly empty
Optional<String> maybe = Optional.ofNullable(getValue()); // wraps a value that might legitimately be null
```

**Q115. What's the difference between `Optional.of()` and `Optional.ofNullable()`?**
`Optional.of(value)` throws `NullPointerException` immediately if `value` is null — use it only when you're certain the value can't be null. `Optional.ofNullable(value)` safely handles a null input by producing an empty Optional instead of throwing — use it when the value's nullness is genuinely uncertain.

**Q116. How do you check if an Optional has a value, and retrieve it?**
```java
if (optional.isPresent()) {
    String value = optional.get();
}
```
(Though this pattern, while valid, is generally considered less idiomatic than the functional-style methods covered next.)

## Intermediate

**Q117. What does `ifPresent()` do, and how is it more idiomatic than `isPresent()` + `get()`?**
`ifPresent(Consumer)` runs the given action only if a value is present, combining the check and the action into one call — avoiding the two-step "check, then unwrap" pattern, which is more error-prone (you might forget the check somewhere else and call `.get()` unsafely).
```java
optional.ifPresent(value -> System.out.println("Found: " + value));
```

**Q118. What does `orElse()` do?**
Returns the contained value if present, or a given fallback/default value if empty.
```java
String result = optional.orElse("default value");
```

**Q119. What does `orElseGet()` do, and how is it different from `orElse()`?**
`orElseGet(Supplier)` also provides a fallback, but the fallback is computed *lazily* — the Supplier is only actually invoked if the Optional is empty. `orElse(value)` takes an already-computed value as its argument, meaning that value is computed EVERY time the code runs, even when the Optional already has a value and the fallback isn't needed at all.
```java
optional.orElse(expensiveComputation());          // expensiveComputation() ALWAYS runs — wasteful if optional has a value!
optional.orElseGet(() -> expensiveComputation()); // only runs if optional is genuinely empty
```
This exact distinction is a very common interview trick question.

**Q120. What does `orElseThrow()` do?**
Returns the value if present, or throws a specified exception if empty — letting you convert "no value" into a proper application-level error where appropriate.
```java
String result = optional.orElseThrow(() -> new NoSuchElementException("Value not found"));
```

**Q121. What does `map()` do on an Optional, and why is it useful?**
Transforms the contained value (if present) using a given function, returning a new Optional wrapping the transformed result — or stays empty if the original was empty, without throwing. It lets you chain transformations without manually null-checking at every step.
```java
Optional<String> name = Optional.of("Alice");
Optional<Integer> length = name.map(String::length); // Optional[5]
```

**Q122. What does `filter()` do on an Optional?**
Keeps the Optional's value only if it satisfies a given Predicate; otherwise returns an empty Optional.
```java
Optional<String> longName = name.filter(n -> n.length() > 10); // empty, since "Alice" is only 5 chars
```

## Advanced

**Q123. Why is it generally considered bad practice to use `Optional` as a method parameter or a class field?**
`Optional` was specifically designed as a **return type** for methods where "no result" is a legitimate, expected outcome — not as a general-purpose null-replacement everywhere. Using it as a parameter forces every caller to wrap arguments unnecessarily (when they could just pass null, or better, use method overloading), and using it as a field adds needless serialization complexity and memory overhead (Optional itself is an object wrapper) without a clear benefit over a well-documented nullable field.

**Q124. What is `Optional.flatMap()`, and how does it differ from `Optional.map()`?**
Use `flatMap()` when your transformation function itself already returns an `Optional` — `map()` would end up producing a nested `Optional<Optional<T>>`, whereas `flatMap()` flattens that down to a single-level `Optional<T>`, exactly analogous to the map-vs-flatMap distinction in Streams.
```java
Optional<String> name = Optional.of("Alice");
Optional<Optional<Integer>> nested = name.map(n -> findAge(n));  // findAge returns Optional<Integer> — nested!
Optional<Integer> flat = name.flatMap(n -> findAge(n));          // correctly flattened
```

**Q125. What does `Optional.ifPresentOrElse()` do (Java 9+, but a natural follow-up question)?**
Lets you handle BOTH the present and empty cases in one call, without needing a separate `isPresent()`/`isEmpty()` check — worth noting this specific method arrived in Java 9, not Java 8, if precision matters in the conversation.
```java
optional.ifPresentOrElse(
    value -> System.out.println("Found: " + value),
    () -> System.out.println("Not found")
);
```

**Q126. Does calling `.get()` on an empty Optional throw an exception? Which one, and why is this itself a bit of a design gotcha?**
Yes — it throws `NoSuchElementException`. The gotcha: this means `Optional`, if used carelessly (calling `.get()` without checking `isPresent()` first), can still produce a runtime crash almost exactly like the `NullPointerException` it was meant to help avoid — Optional only helps if you actually use its safer functional methods (`orElse`, `ifPresent`, `map`, etc.) instead of defaulting back to unconditional `.get()` calls.

**Q127. Is `Optional` serializable, and does that affect whether it's a good fit as a class field?**
`Optional` was deliberately NOT designed to be a great fit for serialization or as an entity field (in frameworks like JPA/Hibernate, for instance) — reinforcing, alongside Q123, that its intended use is specifically as a local return type signaling "this might have no result," not as a general-purpose wrapper for any nullable value anywhere in your codebase.

---

# SECTION 9: Parallel Streams (Q128–Q138)

## Basic

**Q128. How do you create a parallel stream?**
```java
list.parallelStream(); // directly from a Collection
list.stream().parallel(); // convert an existing sequential stream to parallel
```

**Q129. Does a parallel stream automatically run faster than a sequential one?**
No — this is a common misconception. Performance depends on data size, the nature of the operations, and available CPU cores. For small datasets, the overhead of splitting work and merging results can make a parallel stream actually SLOWER than a plain sequential one.

## Intermediate

**Q130. What mechanism powers parallel streams internally?**
The common `ForkJoinPool` — the same work-stealing thread pool used for divide-and-conquer parallel tasks generally. The stream's data source is split into chunks, processed concurrently across the pool's worker threads, and the partial results are merged back together.

**Q131. When are parallel streams a good fit, and when should you avoid them?**
Good fit: large datasets, computationally expensive per-element operations, and stateless/independent operations with no shared mutable state. Avoid them: small datasets (overhead outweighs benefit), operations involving shared mutable state (risk of race conditions), and I/O-bound work (waiting on network/disk isn't sped up by more CPU-bound parallelism).

**Q132. Why is this code dangerous, and how would you fix it?**
```java
List<Integer> results = new ArrayList<>();
numbers.parallelStream().forEach(n -> results.add(n * 2));
```
`ArrayList` is not thread-safe. Multiple threads calling `add()` on it simultaneously (as happens across a parallel stream's worker threads) can corrupt its internal state or silently lose elements — a race condition. Fix: let the stream API handle thread-safe aggregation internally instead of manually mutating a shared list:
```java
List<Integer> results = numbers.parallelStream()
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

## Advanced

**Q133. Does `forEach()` on a parallel stream guarantee elements are processed in encounter order?**
No. To force order in a parallel stream, you must use `forEachOrdered()` instead — though doing so gives up some of the performance benefit of parallelism, since it requires extra coordination to reassemble results in the original sequence.

**Q134. How does the ForkJoinPool split up the source data for a parallel stream, conceptually?**
It recursively divides the data source in half (repeatedly) until each chunk is small enough to process directly, then processes those small chunks across the pool's worker threads using **work-stealing** — idle threads "steal" chunks of work from busier threads' queues to keep all cores continuously busy, rather than having a fixed, potentially unbalanced assignment of chunks to threads.

**Q135. Are all stream sources equally well-suited to being split for parallel processing?**
No — sources with efficient, precise splitting characteristics (like an `ArrayList`, backed by a plain array, which can be split at any index in O(1)) parallelize much better than sources that are expensive or awkward to split (like a `LinkedList`, which has no efficient random-access splitting point, since finding any middle point requires traversal).

**Q136. Can operations like `sorted()` or `distinct()` be parallelized effectively?**
They can be parallelized, but with more coordination overhead than simple stateless operations like `filter()`/`map()`, since they inherently require comparing/tracking information across the whole dataset (or large portions of it) rather than treating each element in complete isolation.

**Q137. What is the danger of using parallel streams inside a web server request-handling thread, in terms of shared resources?**
Parallel streams by default share the single, application-wide common `ForkJoinPool`. If many concurrent requests each trigger parallel stream operations, they're all competing for the same limited pool of worker threads — potentially causing unrelated parts of the application (including other legitimate parallel work, like `CompletableFuture` tasks that also default to the common pool) to be starved or delayed.

**Q138. How would you benchmark whether a parallel stream is actually helping performance for a specific case, rather than assuming it does?**
Measure execution time for both the sequential and parallel versions under realistic data sizes and realistic hardware (ideally using a proper benchmarking tool like JMH rather than simple manual timing, since JIT warm-up and other JVM effects can easily mislead naive manual measurements) — never assume "parallel = faster" without actually verifying it for your specific workload.

---

# SECTION 10: The New Date and Time API — `java.time` (Q139–Q153)

## Basic

**Q139. What problems did the old `java.util.Date` and `Calendar` classes have?**
They were mutable (not thread-safe — sharing a Date object across threads was risky), had a confusing API (months were zero-indexed, so January was `0`; years were often offset from 1900 in confusing ways), and mixed up date, time, and timezone handling in an inconsistent, error-prone way.

**Q140. What are the main classes in the `java.time` package?**
`LocalDate` (date only, no time), `LocalTime` (time only, no date), `LocalDateTime` (both combined, no timezone), `ZonedDateTime` (date + time + timezone), `Duration` (a time-based amount, like "5 hours"), and `Period` (a date-based amount, like "3 months").

**Q141. How do you get the current date and time?**
```java
LocalDate today = LocalDate.now();
LocalTime now = LocalTime.now();
LocalDateTime dateTime = LocalDateTime.now();
```

**Q142. How do you create a specific date, and how does month numbering work now?**
```java
LocalDate date = LocalDate.of(2026, Month.JULY, 16);
LocalDate date2 = LocalDate.of(2026, 7, 16); // month is 1-indexed now — July really is 7, unlike old Calendar's 0-indexing
```

## Intermediate

**Q143. Are `java.time` classes mutable or immutable? Why does this matter?**
All immutable. Every "modifying" method (like `plusDays()`) returns a brand-new object rather than changing the original — this makes them inherently thread-safe (no defensive copying needed to share them across threads) and prevents an entire class of bugs where one part of code unexpectedly mutates a date object that another part of code still relies on.
```java
LocalDate today = LocalDate.now();
LocalDate tomorrow = today.plusDays(1); // "today" itself is completely unchanged
```

**Q144. What's the difference between `Period` and `Duration`?**
`Period` measures a date-based amount of time — years, months, days — appropriate for calendar-style calculations. `Duration` measures a time-based amount — hours, minutes, seconds (and finer) — appropriate for precise elapsed-time calculations. Use `Period` for "how many months until my birthday," and `Duration` for "how many hours did this task take."
```java
Period period = Period.between(LocalDate.of(2020,1,1), LocalDate.of(2024,6,15));
Duration duration = Duration.between(LocalTime.of(9,0), LocalTime.of(17,30));
```

**Q145. How do you format a LocalDate into a String, and parse a String back into a LocalDate?**
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy");
String formatted = LocalDate.now().format(formatter);
LocalDate parsed = LocalDate.parse("16-07-2026", formatter);
```

**Q146. What is `ZonedDateTime`, and when would you use it over `LocalDateTime`?**
`ZonedDateTime` includes explicit timezone information; `LocalDateTime` does not. Use `ZonedDateTime` whenever timezone genuinely matters — e.g., scheduling a meeting across different regions, or storing exact event timestamps meant to be interpreted consistently regardless of the reader's local timezone. Use `LocalDateTime` when timezone is irrelevant or implicitly understood (e.g., "this event always happens at 9 AM wherever it's being viewed").

## Advanced

**Q147. Is `java.time` thread-safe, and why does that matter compared to `SimpleDateFormat`?**
Yes, fully thread-safe, because everything is immutable. This directly fixes a notorious, long-standing bug source: the old `SimpleDateFormat` class is explicitly NOT thread-safe, and sharing one instance across multiple threads (a very tempting, seemingly harmless optimization) could silently produce corrupted/incorrect formatted dates under concurrent use. `DateTimeFormatter` (the java.time equivalent) is immutable and thread-safe, so it can safely be shared and reused across threads without any such risk.

**Q148. How would you calculate someone's age given their birthdate, using `java.time`?**
```java
LocalDate birthDate = LocalDate.of(1995, 3, 20);
LocalDate today = LocalDate.now();
int age = Period.between(birthDate, today).getYears();
```

**Q149. How do you convert between the old `java.util.Date` and the new `java.time` classes, for legacy code interoperability?**
```java
// Date -> Instant -> LocalDateTime
Instant instant = oldDate.toInstant();
LocalDateTime ldt = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());

// LocalDateTime -> Instant -> Date
Instant instant2 = ldt.atZone(ZoneId.systemDefault()).toInstant();
Date newDate = Date.from(instant2);
```

**Q150. What is `Instant`, and how does it differ from `LocalDateTime`?**
`Instant` represents a single, precise point on the timeline, measured from the "epoch" (Jan 1, 1970 UTC) — it has NO concept of timezone or human-readable calendar fields (no "year," "month," etc. directly). `LocalDateTime` represents human-readable date and time fields, but with no timezone or absolute-timeline anchoring at all. They serve different purposes: `Instant` for machine-level precise timestamps, `LocalDateTime`/`ZonedDateTime` for human-facing calendar representations.

**Q151. Is `java.time` based on any pre-existing library's design?**
Yes — it was modeled closely after the well-regarded third-party **Joda-Time** library, which had already solved most of these design problems years before Java 8 officially adopted a similar approach into the standard library.

**Q152. How would you check if a given date falls on a weekend using `java.time`?**
```java
LocalDate date = LocalDate.now();
DayOfWeek day = date.getDayOfWeek();
boolean isWeekend = (day == DayOfWeek.SATURDAY || day == DayOfWeek.SUNDAY);
```

**Q153. What happens if you try to create an invalid date, like February 30th, with `LocalDate.of()`?**
It throws a `DateTimeException` immediately at the point of creation — unlike the old `Calendar` class, which would often silently "roll over" an invalid date into a different, unexpected valid date without any warning, a notorious and confusing old bug source.

---

# SECTION 11: CompletableFuture (Q154–Q165)

## Basic

**Q154. What is `CompletableFuture`, and what problem does it solve compared to plain `Future`?**
It represents a value that will become available asynchronously, and — unlike plain `Future` — lets you chain and compose what should happen next (`thenApply`, `thenCompose`, etc.) without being forced to block and wait (`get()`) in the middle of your logic.

**Q155. How do you create a CompletableFuture that runs some work asynchronously and returns a result?**
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Result");
```

**Q156. What's the difference between `runAsync()` and `supplyAsync()`?**
`runAsync(Runnable)` runs a task that returns nothing (`void`), producing `CompletableFuture<Void>`. `supplyAsync(Supplier<T>)` runs a task that returns a value of type `T`, producing `CompletableFuture<T>`.

## Intermediate

**Q157. What does `thenApply()` do?**
Transforms the eventual result once it's ready, similar to Stream's `map()` — returns a new `CompletableFuture` wrapping the transformed value.
```java
future.thenApply(result -> result.toUpperCase());
```

**Q158. What does `thenAccept()` do, and how is it different from `thenApply()`?**
Consumes the result (performs a side effect, like printing) without producing a further transformed value — analogous to Stream's `forEach()`, whereas `thenApply()` is analogous to `map()`.
```java
future.thenAccept(result -> System.out.println(result));
```

**Q159. What does `thenCompose()` do, and when should you use it instead of `thenApply()`?**
Use `thenCompose()` when your next step ITSELF returns a `CompletableFuture` — `thenApply()` would produce an awkward nested `CompletableFuture<CompletableFuture<T>>`, while `thenCompose()` flattens this into a single-level `CompletableFuture<T>`, exactly analogous to `flatMap()` vs `map()` in Streams.
```java
CompletableFuture<String> result = getUserAsync(id).thenCompose(user -> getAddressAsync(user));
```

**Q160. How do you handle exceptions in a CompletableFuture chain?**
```java
future.exceptionally(error -> {
    System.out.println("Error: " + error.getMessage());
    return "fallback value";
});
```

**Q161. What does `CompletableFuture.allOf()` do?**
Takes multiple CompletableFutures and returns a new `CompletableFuture<Void>` that completes only once ALL of the given futures have completed.
```java
CompletableFuture.allOf(future1, future2, future3).join();
```

**Q162. What does `CompletableFuture.anyOf()` do?**
Returns a new `CompletableFuture` that completes as soon as ANY ONE of the given futures completes (with that one's result) — useful for scenarios like querying multiple redundant data sources and proceeding with whichever responds first.

## Advanced

**Q163. What is `thenCombine()`, and when would you use it?**
Combines the results of two INDEPENDENT CompletableFutures once both are done, using a given combining function — useful when two unrelated async operations both need to finish before you can compute a final combined result.
```java
CompletableFuture<String> greeting = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> name = CompletableFuture.supplyAsync(() -> "Ananth");
CompletableFuture<String> combined = greeting.thenCombine(name, (g, n) -> g + ", " + n);
```

**Q164. Which thread pool does `CompletableFuture.supplyAsync()` use by default, and why might that be a problem in production?**
By default, it uses the shared, application-wide common `ForkJoinPool` (the same pool parallel streams use). In production, if many unrelated parts of an application (or many concurrent requests) all rely on this same shared pool — especially if some tasks are long-running or blocking (like slow I/O) — they can starve each other, since they're all competing for the same limited set of worker threads. It's common practice to pass a dedicated, appropriately-sized `Executor` explicitly as a second argument to avoid this.
```java
CompletableFuture.supplyAsync(() -> doWork(), myDedicatedExecutor);
```

**Q165. What is the difference between `future.get()` and `future.join()`?**
Both block and wait for the result. The difference is exception handling: `get()` throws checked exceptions (`InterruptedException`, `ExecutionException`), forcing you to handle or declare them. `join()` throws an unchecked `CompletionException` instead, which is more convenient inside lambda-heavy CompletableFuture chains where declaring checked exceptions everywhere would be cumbersome.

---

# SECTION 12: Miscellaneous Java 8 Features (Q166–Q178)

## Nashorn JavaScript Engine

**Q166. What is Nashorn?**
A JavaScript engine bundled directly into the JVM starting with Java 8, replacing the older and slower Rhino engine — it let Java applications execute JavaScript code directly.
```java
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("nashorn");
engine.eval("print('Hello from JS')");
```

**Q167. Is Nashorn still part of modern Java? Why does this matter to mention?**
No — Nashorn was deprecated in Java 11 and fully removed in Java 15. It's worth knowing as a Java 8 feature for historical/interview completeness, but shouldn't be relied on in any new system design today.

## Base64 Encoding

**Q168. What did Java 8 add for Base64 encoding, and what existed before it?**
A standard, built-in `java.util.Base64` API. Before Java 8, developers typically relied on third-party libraries (like Apache Commons Codec) since the JDK had no clean, standard way to do this.

**Q169. How do you encode and decode a String using Java 8's Base64 API?**
```java
String encoded = Base64.getEncoder().encodeToString("Hello".getBytes());
byte[] decodedBytes = Base64.getDecoder().decode(encoded);
String decoded = new String(decodedBytes);
```

**Q170. Is Base64 a form of encryption? Why is this a common misconception, and why does it matter?**
No — it's purely an **encoding** scheme for representing binary data as text, with zero security. Anyone can decode Base64 instantly without any key or secret. It's a dangerous misconception because developers sometimes mistakenly use Base64 thinking it "hides" or "protects" sensitive data like passwords — it does neither.

**Q171. What are the three Base64 variants Java 8 provides?**
`Base64.getEncoder()`/`getDecoder()` (standard), `Base64.getUrlEncoder()`/`getUrlDecoder()` (URL-and-filename-safe, using `-` and `_` instead of `+` and `/`), and `Base64.getMimeEncoder()`/`getMimeDecoder()` (MIME-friendly, inserts line breaks suitable for email content).

## Repeating and Type Annotations

**Q172. What problem do repeating annotations solve?**
Before Java 8, you couldn't apply the same annotation type more than once to the same program element — this needed an awkward manual "container" annotation wrapping an array. Java 8 lets you apply the same annotation multiple times directly, cleanly.
```java
@Repeatable(Schedules.class)
@interface Schedule { String day(); }

@interface Schedules { Schedule[] value(); }

@Schedule(day="Monday")
@Schedule(day="Wednesday") // this direct repetition wasn't legal before Java 8
public void doTask() {}
```

**Q173. Do you still need a "container" annotation behind the scenes for repeating annotations to work?**
Yes — the container annotation (`Schedules` in the example above) still has to exist and be referenced via `@Repeatable`. Java 8 just removed the need for YOU to manually write out that container wrapper syntax yourself at each usage site — the compiler handles that translation behind the scenes.

**Q174. What are Type Annotations, and what new capability do they add?**
Before Java 8, annotations could only be placed on declarations (a class, method, or field). Java 8 extended this so annotations can be placed anywhere a type is USED — including generic type arguments, casts, and more.
```java
List<@NonNull String> names; // annotation directly on a generic type argument
```

**Q175. Why do Type Annotations matter practically, if they don't change runtime behavior by themselves?**
They mainly enable more powerful static analysis tools (like the Checker Framework) to catch subtle bugs at compile time — for example, flagging an attempt to insert a null into a collection explicitly declared to never allow nulls, before the program ever actually runs.

## Internal JVM Changes

**Q176. What was PermGen, and why was it removed in Java 8?**
PermGen ("Permanent Generation") was a fixed-size memory region used to store class metadata in versions before Java 8. Its fixed size was notoriously hard to tune correctly, and applications that dynamically generated or loaded many classes (common in application servers, or frameworks doing heavy bytecode generation/proxying) frequently hit `OutOfMemoryError: PermGen space`.

**Q177. What replaced PermGen, and how does it fix the underlying problem?**
**Metaspace** — which lives in native (off-heap) memory and, critically, grows automatically by default instead of being capped at a fixed, hard-to-tune size. This largely eliminated the specific class of OutOfMemoryError that PermGen was notorious for.

**Q178. What is `StringJoiner`, and how does it relate to `Collectors.joining()`?**
`StringJoiner` is a standalone utility class (also introduced in Java 8) for efficiently building a delimited string, supporting an optional prefix and suffix — it's actually the underlying mechanism that `Collectors.joining()` uses internally when you call it as part of a stream pipeline.
```java
StringJoiner joiner = new StringJoiner(", ", "[", "]");
joiner.add("a").add("b").add("c");
System.out.println(joiner); // [a, b, c]
```

---

# SECTION 13: Mixed / Scenario-Based Questions (Q179–Q190)

**Q179. Write a stream pipeline to find the second-highest salary from a list of employees.**
```java
Optional<Double> secondHighest = employees.stream()
    .map(Employee::getSalary)
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst();
```

**Q180. Given a list of Strings, write a stream pipeline to count how many start with a vowel.**
```java
long count = words.stream()
    .filter(w -> !w.isEmpty() && "AEIOUaeiou".indexOf(w.charAt(0)) != -1)
    .count();
```

**Q181. Given a `Map<String, List<Integer>>`, write code to find the total sum of ALL values across all lists, using streams.**
```java
int total = map.values().stream()
    .flatMap(List::stream)
    .mapToInt(Integer::intValue)
    .sum();
```

**Q182. How would you safely chain three methods, each of which might return null, without a wall of null checks — using Optional?**
```java
Optional<String> result = Optional.ofNullable(getUser(id))
    .map(User::getAddress)
    .map(Address::getCity);
String city = result.orElse("Unknown");
```

**Q183. Write a Comparator using Java 8 style that sorts a list of employees first by department (ascending), then by salary (descending).**
```java
employees.sort(Comparator.comparing(Employee::getDepartment)
                          .thenComparing(Employee::getSalary, Comparator.reverseOrder()));
```

**Q184. Given a List<String>, remove duplicates while preserving original order, using streams (not LinkedHashSet directly).**
```java
List<String> unique = list.stream().distinct().collect(Collectors.toList());
// distinct() preserves encounter order for ordered streams, so this works directly
```

**Q185. Write code that reads a list of orders and groups the total order value by customer, using groupingBy + a downstream collector.**
```java
Map<String, Double> totalByCustomer = orders.stream()
    .collect(Collectors.groupingBy(Order::getCustomerName,
             Collectors.summingDouble(Order::getAmount)));
```

**Q186. What would this print, and why?**
```java
int value = 10;
Supplier<Integer> supplier = () -> value;
value = 20; // does this compile?
```
This does NOT compile — `value` is used inside a lambda, so it must be effectively final, but reassigning it (`value = 20`) after its initial assignment violates that requirement. The compiler will flag this at the `value = 20` line (or, depending on exact ordering, at the point of lambda creation) with an error about the variable needing to be effectively final.

**Q187. Trace through this code and explain the output:**
```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> result = nums.stream()
    .filter(n -> n > 10)
    .findFirst();
System.out.println(result.orElse(-1));
```
No element in `nums` is greater than 10, so `filter()` produces an empty stream, and `findFirst()` returns an empty `Optional`. `orElse(-1)` then supplies the fallback since the Optional is empty — output: `-1`.

**Q188. Given two lists of integers, write a stream-based way to find their intersection (common elements).**
```java
List<Integer> intersection = list1.stream()
    .filter(list2::contains)
    .collect(Collectors.toList());
```

**Q189. How would you use CompletableFuture to fetch data from two independent external services in parallel, then combine both results once both are ready?**
```java
CompletableFuture<String> service1 = CompletableFuture.supplyAsync(() -> callService1());
CompletableFuture<String> service2 = CompletableFuture.supplyAsync(() -> callService2());

CompletableFuture<String> combined = service1.thenCombine(service2, (r1, r2) -> r1 + " | " + r2);
String finalResult = combined.join();
```

**Q190. In a fintech context, why might you specifically prefer `Collectors.groupingBy()` with a downstream `summingDouble()` over manually looping and using a `HashMap` to tally transaction totals by account?**
The stream-based approach is declarative (states clearly "group by account, sum the amounts" in one readable line), avoids manually managing a mutable accumulator map with explicit `merge()`/`computeIfAbsent()` calls, and is naturally safe from certain classes of bugs (like forgetting to initialize an entry before adding to it) that a hand-rolled loop can introduce — while remaining just as correct and often just as performant for typical reporting/aggregation workloads.

---

# Quick-reference summary table

| Section | Topic | Question range |
|---|---|---|
| 1 | Why Java 8 — the big picture | Q1–Q6 |
| 2 | Lambda Expressions | Q7–Q22 |
| 3 | Functional Interfaces | Q23–Q38 |
| 4 | Method References | Q39–Q48 |
| 5 | Default & Static Interface Methods | Q49–Q60 |
| 6 | The Stream API | Q61–Q95 |
| 7 | Collectors | Q96–Q112 |
| 8 | Optional | Q113–Q127 |
| 9 | Parallel Streams | Q128–Q138 |
| 10 | java.time (Date/Time API) | Q139–Q153 |
| 11 | CompletableFuture | Q154–Q165 |
| 12 | Nashorn, Base64, Annotations, PermGen→Metaspace, StringJoiner | Q166–Q178 |
| 13 | Mixed / Scenario-based | Q179–Q190 |

**Total: 190 questions.**

---

## Suggested study strategy
1. **Sections 2, 3, 6, 7, 8** (Lambdas, Functional Interfaces, Streams, Collectors, Optional) are the highest-yield — expect the deepest, most frequent questioning here in any Java 8 interview.
2. **Section 11 (CompletableFuture)** matters more at companies doing real async/concurrent systems — worth extra attention for fintech/product-company interviews specifically.
3. **Section 5 (default/static methods)** and **Section 10 (java.time)** come up reliably but usually at moderate depth — know the "why," not just the syntax.
4. **Section 12** (Nashorn, Base64, annotations, PermGen) is good breadth knowledge, rarely the focus of deep follow-up questions — know it exists and its one-line purpose.
5. **Section 13** is where interviewers test whether you can actually *write* stream/Optional/CompletableFuture code under pressure, not just talk about it — practice writing these by hand, not just reading them.
