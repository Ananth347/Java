# Java Backend Developer — 25 Interview Sessions (Q&A)

Target: 1–2 years experience, Java + Spring Boot backend roles.
Format: each session is a themed round with plain-English answers — the way you'd actually explain it to an interviewer, not a textbook definition.

---

## Session 1: Core Java Basics

**Q1. Tell me about yourself.**
Keep it structured: current role and years of experience, the tech stack you work with day to day, one or two solid projects with the impact they had, and end with what you're looking for next. Practice a 60–90 second version — don't read off your resume line by line.

**Q2. What is the difference between JDK, JRE, and JVM?**
JVM is the engine that actually runs your compiled Java code — it's what makes Java "write once, run anywhere." JRE is the JVM plus the standard libraries needed to run a Java program. JDK is the full package for developers — JRE plus the compiler, debugger, and other tools needed to write and build Java code, not just run it.

**Q3. What is the difference between `==` and `equals()`?**
`==` compares references for objects — it checks whether two variables point to the exact same object in memory. For primitives, it compares actual values. `equals()` is a method that compares the actual content/state of two objects, and you can override it to define what "equal" means for your own class. For example, two separate `String` objects with the same text are `==` false but `.equals()` true.

**Q4. Why is String immutable in Java?**
Once a `String` object is created, its value can never change — any operation that looks like it's modifying a string actually creates a new object. This is done for security (strings are used in class loading, network connections, file paths — you don't want them changed underneath you), thread-safety (immutable objects are automatically safe to share across threads), and performance (Java maintains a "String pool" that reuses identical string literals, which only works safely if strings can't change).

**Q5. What is the difference between `String`, `StringBuilder`, and `StringBuffer`?**
`String` is immutable — every modification creates a new object, which gets expensive if you're doing lots of concatenation in a loop. `StringBuilder` is mutable and lets you modify the same object in place, making it much faster for heavy string manipulation, but it's not thread-safe. `StringBuffer` is the same idea as `StringBuilder` but with synchronized methods, making it thread-safe at the cost of some performance. In practice, use `StringBuilder` unless multiple threads are genuinely touching the same buffer.

**Q6. What are wrapper classes, and why do we need them?**
Wrapper classes (`Integer`, `Double`, `Boolean`, etc.) wrap primitive types into objects. We need them because Java's collections (`List`, `Map`, etc.) can only store objects, not raw primitives, and because objects can be `null` — sometimes you need to represent "no value" for a number, which a primitive `int` can't do.

---

## Session 2: OOP Fundamentals

**Q1. Explain the four pillars of OOP with examples.**
Encapsulation is bundling data and the methods that operate on it together, and hiding internal details behind private fields with public getters/setters — like a bank account class not letting outside code touch the balance directly. Inheritance lets one class reuse and extend another's behavior — a `SavingsAccount` extending a general `Account` class. Polymorphism means the same method call behaves differently depending on the actual object — calling `calculateInterest()` on different account types gives different results. Abstraction means exposing only what's necessary and hiding the complex implementation — like an interface `PaymentGateway` that hides whether it's actually calling Razorpay or Stripe underneath.

**Q2. Difference between Abstract Class and Interface?**
An abstract class can have both fully implemented methods and abstract ones, can hold state (instance fields), and a class can only extend one abstract class. An interface, traditionally, only declared method signatures with no implementation, though since Java 8 it can also have `default` and `static` methods with actual bodies. A class can implement multiple interfaces, which is how Java works around the lack of multiple inheritance. General rule of thumb: use an abstract class when subclasses share common state and behavior; use an interface when you just need to guarantee a contract/capability across unrelated classes.

**Q3. What is method overloading vs method overriding?**
Overloading is having multiple methods with the same name but different parameters within the same class — decided at compile time based on the arguments you pass. Overriding is when a subclass provides its own implementation of a method already defined in its parent class, with the same signature — decided at runtime based on the actual object type. Overloading is about "same name, different input"; overriding is about "same signature, different behavior in a subclass."

**Q4. What is constructor chaining?**
It's when one constructor calls another constructor, either in the same class using `this(...)`, or in the parent class using `super(...)`. It's useful to avoid duplicating initialization logic across multiple constructors — you write the common setup once in one constructor and have the others delegate to it.

**Q5. Can you override a static method in Java?**
No — static methods belong to the class, not to an instance, so they're resolved at compile time based on the reference type, not the actual object. If a subclass defines a static method with the same signature, it's "hiding" the parent's method, not overriding it. This is a common trick question — the behavior looks like overriding but doesn't follow polymorphic rules.

**Q6. What is the `super` keyword used for?**
`super` refers to the immediate parent class. You use it to call the parent's constructor (`super(...)`), to call a parent's method that's been overridden in the child (`super.methodName()`), or to access a parent's field if it's been shadowed by the child.

---

## Session 3: JVM & Memory Management

**Q1. Explain the JVM memory areas at a high level.**
The Heap is where all objects live — it's shared across the whole application and is where garbage collection happens. The Stack is per-thread, and stores method call frames — local variables and references, cleaned up automatically when a method returns. Metaspace (replacing the old "PermGen" from Java 8 onward) stores class metadata — the structure of your classes, not the objects themselves. There's also the Program Counter register and native method stacks, but those rarely come up in interviews at this level.

**Q2. What is Garbage Collection, and why don't we manage memory manually in Java?**
Garbage Collection is the JVM automatically finding objects that are no longer referenced by anything reachable in your program, and reclaiming that memory. This removes the whole class of bugs that plague languages like C/C++ — dangling pointers, memory leaks from forgetting to free memory, double-frees. You do still need to be careful, though — holding onto references you don't need anymore (like in static collections or caches) can still cause memory leaks even with GC.

**Q3. What is the difference between Stack and Heap memory?**
Stack memory holds local variables and method call information, is thread-specific, and is very fast because it works like a simple last-in-first-out structure. Heap memory holds actual objects, is shared across all threads, and is slower to allocate/deallocate because the GC has to track and manage it. If you get a `StackOverflowError`, it usually means deep or infinite recursion; if you get `OutOfMemoryError`, it usually means too many objects are being held in the heap without being released.

**Q4. What causes a memory leak in Java if GC is automatic?**
A memory leak happens when objects are no longer needed by your application logic, but something is still holding a reference to them, so the GC can't reclaim them. Common causes: static collections that keep growing (like a cache with no eviction policy), unclosed resources (file streams, DB connections), and listeners/callbacks that never get unregistered.

**Q5. What is the difference between `final`, `finally`, and `finalize`?**
`final` is a keyword used on variables (makes them constants), methods (can't be overridden), or classes (can't be extended). `finally` is a block that always runs after a try/catch, regardless of whether an exception occurred — typically used to close resources. `finalize()` is a method that used to be called by the GC before destroying an object, but it's deprecated now (unreliable timing) — `try-with-resources` and explicit cleanup are the modern replacements.

---

## Session 4: Collections Framework — Lists & Sets

**Q1. Difference between ArrayList and LinkedList?**
`ArrayList` is backed by a dynamic array — so accessing an element by index is very fast (constant time), but inserting or deleting in the middle is slow because everything after it has to shift. `LinkedList` is backed by a doubly linked list — inserting/deleting is fast if you already have a reference to the position, but accessing an element by index is slow because it has to walk through the list. In practice, `ArrayList` is used far more often; `LinkedList` shines mainly when you're doing a lot of insertions/deletions at the ends (like implementing a queue).

**Q2. What is the difference between `List`, `Set`, and `Map`?**
A `List` is an ordered collection that allows duplicates and lets you access elements by index. A `Set` doesn't allow duplicates and generally doesn't guarantee order (except `LinkedHashSet`/`TreeSet`). A `Map` stores key-value pairs, where each key is unique, and you look things up by key rather than by position.

**Q3. How does `HashSet` ensure there are no duplicates?**
Internally, a `HashSet` is backed by a `HashMap` — every element you add becomes a key in that map, with a dummy constant value. Since map keys must be unique, adding a duplicate simply overwrites the existing entry instead of creating a new one, which is why sets end up with no duplicates. It relies on the element's `hashCode()` and `equals()` being implemented correctly.

**Q4. Difference between `Comparable` and `Comparator`?**
`Comparable` is implemented by the class itself, defining its "natural" ordering through a single `compareTo()` method — for example, `Employee implements Comparable<Employee>` sorting by ID by default. `Comparator` is a separate class (often a lambda) you pass in when you want a *different* or *custom* sorting order without changing the original class — like sorting employees by salary in one place and by name in another.

**Q5. What happens if you don't override `hashCode()` and `equals()` for a custom object used in a `HashSet` or as a `HashMap` key?**
Java falls back to the default `Object` implementations, which compare based on memory reference, not logical content. That means two objects that "look the same" to you (like two `Employee` objects with the same ID) will be treated as different entries — leading to duplicates in a `Set`, or failed lookups in a `Map`. Whenever you use a custom class as a `HashMap` key or in a `HashSet`, you must override both methods together, consistently.

**Q6. What is the difference between `Iterator` and `ListIterator`?**
`Iterator` lets you traverse a collection only in the forward direction, and only supports removing elements. `ListIterator` (only available for `List`) lets you go both forward and backward, and also supports adding and modifying elements while iterating, not just removing them.

---

## Session 5: Collections Framework — Maps & HashMap Internals

**Q1. How does `HashMap` work internally?**
Internally, a `HashMap` is an array of "buckets." When you put a key-value pair, Java calls `hashCode()` on the key, does some bit manipulation to spread the hash out evenly, and uses that to decide which bucket the entry goes into. If two keys land in the same bucket (a "collision"), the entries are stored as a linked list within that bucket — or, since Java 8, as a balanced tree if that bucket gets too crowded (8+ entries), to keep lookups fast even in the worst case.

**Q2. What happens when you `get()` a value from a `HashMap`?**
Java calls `hashCode()` on the key you're looking up, figures out which bucket it should be in, then walks through that bucket's entries checking `equals()` against each one until it finds a match. If the key isn't found, it returns `null` — which is why `HashMap` allows one `null` key, and why you should be careful distinguishing "key not present" from "key present with null value" (use `containsKey()` for that).

**Q3. What is the load factor and why does it matter?**
The load factor (default 0.75) decides how full the `HashMap` can get before it automatically resizes (doubles its bucket array and re-distributes everything). A lower load factor means more memory usage but fewer collisions and faster lookups; a higher one saves memory but risks more collisions. The default 0.75 is a well-tested balance for most use cases.

**Q4. Difference between `HashMap`, `LinkedHashMap`, and `TreeMap`?**
`HashMap` gives no guarantee on iteration order. `LinkedHashMap` maintains insertion order (or optionally access order, useful for building an LRU cache). `TreeMap` keeps keys sorted according to their natural ordering or a custom comparator, backed by a red-black tree, so operations are slightly slower (log n) but you get sorted iteration for free.

**Q5. Is `HashMap` thread-safe? What are the alternatives?**
No, `HashMap` is not thread-safe — concurrent modification from multiple threads can corrupt its internal structure or cause an infinite loop in older Java versions. `Collections.synchronizedMap()` wraps it with synchronization but locks the whole map for every operation, which is slow under contention. `ConcurrentHashMap` is the modern, preferred choice — it locks at a much finer granularity (segments/buckets) so multiple threads can read and write different parts concurrently without blocking each other.

**Q6. What's the time complexity of `HashMap` operations?**
On average, `get`, `put`, `remove`, and `containsKey` are all O(1) — constant time — assuming a good hash function that spreads keys evenly. In the worst case (lots of collisions all landing in the same bucket), it degrades to O(log n) since Java 8 (thanks to the tree-based buckets), whereas before Java 8 it degraded all the way to O(n).

---

## Session 6: Multithreading & Concurrency

**Q1. What is Multithreading, and why do we use it?**
Multithreading means running multiple smaller units of work (threads) concurrently within a single program, so the CPU can be used more efficiently — for example, one thread handling a user request while another writes logs in the background. It's used to improve performance and responsiveness, especially for tasks that involve waiting (I/O, network calls) where one thread can do useful work while another is blocked.

**Q2. Why do we need synchronization?**
When multiple threads access and modify shared data (like a shared counter or a shared list) at the same time, you can get race conditions — inconsistent or corrupted results because operations interleave unpredictably. Synchronization ensures that only one thread can execute a critical section of code at a time, so shared state stays consistent.

**Q3. Difference between `synchronized` method and `synchronized` block?**
A `synchronized` method locks the entire method, using either the object instance (`this`) or the class object (for static methods) as the lock. A `synchronized` block lets you lock only a specific section of code, and choose exactly what object to lock on — which is generally preferred because it reduces the amount of code that's blocked, improving performance.

**Q4. What is a Deadlock?**
A deadlock happens when two or more threads are each waiting for a resource the other one is holding, so none of them can ever proceed — like two people trying to pass through a narrow door, each insisting the other go first. A classic cause is two threads locking two shared resources in a different order. It's avoided by always acquiring locks in a consistent, agreed-upon order across the whole codebase.

**Q5. What is the difference between `Runnable` and `Callable`?**
`Runnable`'s `run()` method returns nothing and can't throw checked exceptions. `Callable`'s `call()` method can return a result and is allowed to throw checked exceptions. If you need a background task's outcome (like the result of a computation), you use `Callable` with an `ExecutorService`, which gives you back a `Future` you can use to retrieve the result later.

**Q6. What is `ExecutorService` and why use it instead of creating threads manually?**
Creating a new `Thread` for every task is expensive and doesn't scale — you'd end up creating hundreds of threads for hundreds of tasks. `ExecutorService` manages a pool of reusable threads for you, so tasks are queued and executed by whichever thread is free, which is far more efficient and lets you control concurrency (e.g., "never run more than 10 tasks at once").

**Q7. What is `volatile` used for?**
Normally, each thread might cache a variable's value locally for performance, meaning one thread's update might not be immediately visible to another. Marking a variable `volatile` forces every read and write to go directly to main memory, guaranteeing that all threads always see the latest value. It solves visibility problems but doesn't make compound operations (like `count++`) atomic — for that, you'd still need synchronization or `AtomicInteger`.

---

## Session 7: Java 8+ Features — Lambdas & Streams

**Q1. What are Lambda expressions and why were they introduced?**
A lambda expression is a compact way of writing an implementation of a functional interface (an interface with one abstract method), without the verbosity of an anonymous class. They were introduced to support functional-style programming in Java — allowing behavior to be passed around like data, which makes the Stream API, event handlers, and callback-style code much shorter and more readable.

**Q2. What is a functional interface?**
It's an interface with exactly one abstract method — that single method is what a lambda expression implements. `Runnable`, `Comparator`, and `Function` are common examples. The `@FunctionalInterface` annotation isn't strictly required, but it's good practice — it makes the compiler enforce the "only one abstract method" rule and prevents accidental additions later.

**Q3. What is the Stream API and how is it different from a Collection?**
A Collection is a data structure that actually stores elements in memory. A Stream is not a data structure at all — it's a pipeline that describes a sequence of operations (filter, map, sort, etc.) to be performed on data, and it doesn't compute anything until a "terminal operation" like `collect()` or `forEach()` is called. Streams also don't modify the original source; they produce a new result.

**Q4. What's the difference between intermediate and terminal operations in Streams?**
Intermediate operations (`filter`, `map`, `sorted`) are lazy — they just describe a transformation and return another stream, without actually doing any work yet. Terminal operations (`collect`, `forEach`, `count`, `reduce`) trigger the actual execution of the whole pipeline and produce a final result. Because of this laziness, you can chain several intermediate operations efficiently — Java processes each element through the whole pipeline in one pass, rather than looping multiple times.

**Q5. What is `Optional` and what problem does it solve?**
`Optional<T>` is a container that may or may not hold a value, used as a return type to explicitly signal "this might not have a result" instead of silently returning `null`. It solves the problem of `NullPointerException` by forcing the caller to explicitly handle the empty case (`orElse`, `orElseThrow`, `isPresent`, `map`), rather than accidentally forgetting a null check.

**Q6. Can you use a stream twice?**
No — once a terminal operation has been called on a stream, that stream is considered "consumed" and can't be reused; trying to use it again throws an `IllegalStateException`. If you need to run multiple operations on the same data, you'd generate a fresh stream from the source collection each time.

---

## Session 8: Exception Handling

**Q1. Difference between Checked and Unchecked exceptions?**
Checked exceptions (like `IOException`, `SQLException`) are checked by the compiler at compile time — you must either handle them with a try-catch or declare them with `throws`. Unchecked exceptions (subclasses of `RuntimeException`, like `NullPointerException`, `ArrayIndexOutOfBoundsException`) aren't checked at compile time — they usually represent programming bugs rather than recoverable external conditions, so Java doesn't force you to handle them everywhere.

**Q2. What is the difference between `throw` and `throws`?**
`throw` is used inside a method body to actually raise/trigger an exception at that point. `throws` is used in a method's signature to declare that this method might throw a certain checked exception, so callers know they need to handle it.

**Q3. What happens if an exception is thrown inside a `finally` block?**
It suppresses any exception that was already being propagated from the try/catch block — the exception from `finally` takes over, and the original one is essentially lost unless you explicitly capture it as a "suppressed exception." This is one reason to avoid throwing exceptions inside `finally` blocks — keep them for cleanup only.

**Q4. Can you create a custom exception? Why would you?**
Yes — you create a class extending `Exception` (checked) or `RuntimeException` (unchecked). You'd do this to make error handling more meaningful and specific to your business domain — for example, throwing an `InsufficientFundsException` in a banking app is far clearer to both developers and calling code than throwing a generic `RuntimeException` with just a message.

**Q5. How do you handle exceptions globally in a Spring Boot application?**
Instead of wrapping every controller method in try-catch blocks, Spring Boot lets you define a class annotated with `@ControllerAdvice` (or `@RestControllerAdvice`), containing methods annotated with `@ExceptionHandler(SomeException.class)`. Spring automatically routes any matching exception thrown from any controller to that centralized handler, letting you return consistent, well-structured error responses across the whole application.

**Q6. What is exception chaining?**
It's when you catch one exception and wrap it inside another, passing the original as the "cause" — for example, catching a low-level `SQLException` and rethrowing it as a more meaningful `DataAccessException`, while still preserving the original stack trace via the cause. This helps you add business context without losing the original technical details needed for debugging.

---

## Session 9: Generics

**Q1. What are Generics and why do we use them?**
Generics let you write classes, interfaces, and methods that work with any type, while still catching type mismatches at compile time instead of at runtime. Before generics, collections stored plain `Object`, meaning you had to manually cast every element back to its real type, and mistakes only surfaced as a `ClassCastException` at runtime. With `List<String>`, for example, the compiler guarantees you can never accidentally put an `Integer` into that list.

**Q2. What is type erasure?**
At compile time, Java uses generic type information to check your code for correctness, but at runtime, that type information is actually erased — a `List<String>` and a `List<Integer>` are both just `List` under the hood in the compiled bytecode. This is why you can't do things like `new T[10]` or check `if (obj instanceof List<String>)` at runtime — the specific type parameter simply isn't available anymore at that point.

**Q3. What's the difference between `List<?>`, `List<Object>`, and a raw `List`?**
`List<Object>` means a list that can specifically hold any type of object, and you can add anything to it. `List<?>` (wildcard) means "a list of some specific but unknown type" — you can read from it safely, but you generally can't add to it (except `null`), since the compiler doesn't know the actual type. A raw `List` (no type parameter at all) bypasses generics entirely and is essentially legacy, pre-Java-5 style — best avoided.

**Q4. What is bounded type parameter, e.g. `<T extends Number>`?**
It restricts what types can be used with a generic class or method — `<T extends Number>` means T must be `Number` or one of its subclasses (`Integer`, `Double`, etc.), letting you safely call `Number` methods like `.doubleValue()` inside the generic code, which you couldn't do with an unbounded `<T>`.

---

## Session 10: Design Patterns

**Q1. Explain the Singleton pattern and how you'd implement it safely in a multithreaded environment.**
Singleton ensures a class has exactly one instance throughout the application, with a global access point to it — commonly used for things like configuration objects or connection managers. In a multithreaded environment, the safest common approach is either an `enum`-based singleton (thread-safe by default, handled by the JVM), or using a static holder class, or double-checked locking with a `volatile` instance field — the naive lazy version without synchronization can end up creating two instances if two threads hit it at the same time.

**Q2. What is the Builder pattern and when would you use it?**
Builder is used to construct complex objects step by step, especially when a class has many optional fields — instead of a constructor with ten parameters (hard to read and error-prone), you chain method calls like `.name("x").age(30).build()`. It's especially useful when many combinations of fields are optional, avoiding the need for dozens of overloaded constructors.

**Q3. Explain the Factory pattern.**
Factory pattern centralizes object creation logic in one place instead of scattering `new SomeClass()` calls throughout the codebase. A factory method decides which concrete class to instantiate based on some input, returning it through a common interface or abstract type — so the calling code doesn't need to know the exact implementation class, just the type it can rely on.

**Q4. What is Dependency Injection, and how does Spring implement it?**
Dependency Injection means a class doesn't create its own dependencies — instead, they're provided ("injected") from outside, usually by a framework. This makes classes easier to test (you can inject mock dependencies) and loosely coupled. Spring implements this through its IoC (Inversion of Control) container, which scans your classes, builds objects (beans), and wires their dependencies together automatically, based on annotations like `@Autowired`, `@Component`, `@Service`.

**Q5. What's the difference between Strategy pattern and simply using if-else?**
Strategy pattern encapsulates a family of interchangeable behaviors (algorithms) behind a common interface, letting you swap the actual implementation at runtime without touching the calling code. Instead of a growing chain of if-else or switch statements checking a "type" and picking behavior accordingly, you inject the right strategy object directly — this is more maintainable and follows the open-closed principle (open for extension, closed for modification).

---

## Session 11: Spring Core & Dependency Injection

**Q1. What is Inversion of Control (IoC)?**
Normally, your code controls the creation and flow of objects — you call `new` and manage the lifecycle yourself. IoC flips that: the framework (Spring) takes control of creating objects and wiring their dependencies together, and your code just declares what it needs. This is the core idea behind the whole Spring container.

**Q2. What is a Spring Bean?**
A bean is simply an object that's created, configured, and managed by the Spring IoC container, rather than by your own code directly. You mark a class to become a bean using annotations like `@Component`, `@Service`, `@Repository`, or by explicitly declaring it with `@Bean` inside a `@Configuration` class.

**Q3. Difference between `@Component`, `@Service`, and `@Repository`?**
Functionally, all three do the same thing — register a class as a Spring bean. They're really about intent and readability: `@Component` is the generic, general-purpose stereotype; `@Service` marks a class holding business logic; `@Repository` marks a class handling data access, and additionally enables Spring's automatic translation of database exceptions into its own consistent `DataAccessException` hierarchy.

**Q4. What are the different types of Dependency Injection in Spring?**
Constructor injection passes dependencies through the class's constructor — this is the recommended approach because it makes dependencies explicit and lets you make fields `final` (immutable), and it fails fast at startup if something's missing. Setter injection passes dependencies through setter methods, useful for optional dependencies. Field injection uses `@Autowired` directly on a field — it's the most concise but generally discouraged now, since it hides dependencies and makes unit testing harder (you can't easily pass mocks without reflection).

**Q5. What is a Bean's default scope, and what other scopes exist?**
The default scope is "singleton" — Spring creates exactly one instance of the bean per container, shared everywhere it's injected. "Prototype" scope creates a brand-new instance every time it's requested. There are also web-aware scopes like "request" (one instance per HTTP request) and "session" (one instance per user session), used less often outside typical web apps.

**Q6. What is `@Qualifier` used for?**
When you have multiple beans of the same type (say, two implementations of a `PaymentService` interface), Spring won't know which one to inject and will throw an error. `@Qualifier("beanName")` tells Spring exactly which specific bean to use in that particular injection point.

---

## Session 12: Spring Boot Basics

**Q1. What is Spring Boot, and how is it different from the Spring Framework?**
Spring Framework is powerful but historically required a lot of manual XML/Java configuration to set things up — data sources, view resolvers, security, etc. Spring Boot is built on top of Spring, and its whole purpose is to eliminate that boilerplate through "auto-configuration" — it looks at what's on your classpath and sensible defaults, and configures most things for you automatically, so you can get a working application running in minutes.

**Q2. What does `@SpringBootApplication` actually do?**
It's really a shortcut that combines three annotations: `@Configuration` (marks the class as a source of bean definitions), `@EnableAutoConfiguration` (turns on Spring Boot's automatic configuration based on your dependencies), and `@ComponentScan` (tells Spring to scan the current package and sub-packages for components/beans to register).

**Q3. What is auto-configuration in Spring Boot?**
Spring Boot looks at the JARs present on your classpath and configures beans automatically based on what it finds — for example, if it sees an embedded database driver and Spring Data JPA on the classpath, it'll auto-configure a `DataSource` and `EntityManager` for you without you writing that configuration yourself. You can always override any auto-configured bean by defining your own bean of that type.

**Q4. What is `application.properties`/`application.yml` used for?**
It's the central place to externalize configuration — database URLs, server port, logging levels, custom app-specific settings — so you don't hardcode them in your Java code. This makes it easy to have different configurations for different environments (dev, test, production) without changing code, often using Spring profiles.

**Q5. What are Spring Profiles?**
Profiles let you maintain separate sets of configuration for different environments — like `application-dev.properties` and `application-prod.properties` — and activate the right one using a property or environment variable (`spring.profiles.active=prod`) without changing any code. This is very commonly used to keep, say, a real payment gateway config in production but a sandbox/mock one in development.

**Q6. What is an embedded server in Spring Boot, and why does it matter?**
Traditionally, you'd build a WAR file and deploy it to an external server like Tomcat that you install and manage separately. Spring Boot embeds a server (Tomcat by default, or Jetty/Undertow) directly inside your application JAR, so your app is self-contained and runs with a simple `java -jar app.jar` — no separate server installation or configuration needed. This is a big part of why Spring Boot apps are so easy to containerize with Docker.

---

## Session 13: Spring Boot Request Flow & REST APIs

**Q1. Walk me through what happens when a REST API request hits your Spring Boot application.**
The embedded servlet container (Tomcat) receives the HTTP request first. It passes through Spring's `DispatcherServlet`, which acts as the central traffic controller — it consults the `HandlerMapping` to figure out which controller method should handle this specific URL and HTTP method. Any filters or interceptors (like authentication checks) run along the way. The matched controller method executes your business logic, often calling a service layer, which calls a repository/database layer. The response object is then converted (usually to JSON, via Jackson) and sent back through the same chain to the client.

**Q2. Difference between `@Controller` and `@RestController`?**
`@Controller` is used for traditional MVC apps where methods typically return a view name (like a Thymeleaf template) to be rendered as HTML. `@RestController` is `@Controller` combined with `@ResponseBody` on every method — meaning whatever the method returns gets serialized directly into the response body (usually JSON), which is exactly what you want for building REST APIs.

**Q3. What is the difference between `@PathVariable` and `@RequestParam`?**
`@PathVariable` extracts a value that's part of the URL path itself — like the `123` in `/users/123`. `@RequestParam` extracts a value from the query string — like the `active=true` in `/users?active=true`. Path variables usually identify a specific resource; request params usually filter, sort, or paginate a collection of resources.

**Q4. What HTTP status codes would you typically return for a REST API, and when?**
`200 OK` for a successful GET or update. `201 Created` for a successful POST that creates a new resource. `204 No Content` for a successful action with nothing to return, like a DELETE. `400 Bad Request` for invalid input from the client. `401 Unauthorized` when authentication is missing or invalid. `403 Forbidden` when the user is authenticated but doesn't have permission. `404 Not Found` when the requested resource doesn't exist. `500 Internal Server Error` for unexpected server-side failures.

**Q5. What is idempotency, and why does it matter for REST APIs?**
An operation is idempotent if calling it multiple times has the same effect as calling it once — GET, PUT, and DELETE are expected to be idempotent, while POST typically isn't. This matters a lot in real systems (especially payments) — if a network call times out and the client retries, an idempotent endpoint won't accidentally create duplicate records or charge someone twice.

**Q6. What is DTO, and why not just return your Entity directly from a Controller?**
A DTO (Data Transfer Object) is a plain object designed specifically to carry data between layers, especially over the API boundary. You avoid returning entities directly because entities are tied to your database structure (and often carry sensitive fields, internal IDs, or lazy-loaded relationships), and exposing them directly can leak internal details or cause serialization issues (like a `LazyInitializationException`). DTOs let you control exactly what shape of data goes out, independent of how it's stored internally.

---

## Session 14: Spring Data JPA & Hibernate

**Q1. What is JPA, and what is Hibernate's relationship to it?**
JPA (Java Persistence API) is a specification — a set of rules and interfaces defining how Java objects should be mapped to relational database tables (ORM — Object Relational Mapping). Hibernate is the most popular actual implementation of that specification — it does the real work of generating SQL, managing sessions, and handling the object-to-table mapping. Spring Data JPA sits one layer above both, giving you repository interfaces where common queries are generated automatically, without you writing implementation code.

**Q2. What is the difference between JPA and Hibernate?**
JPA is just an interface/specification — it doesn't do anything by itself. Hibernate is one of several implementations of that specification (EclipseLink is another). In practice, most Spring Boot projects use Hibernate under the hood while coding against the JPA interfaces, so if needed, you could theoretically swap the underlying implementation without changing much of your code.

**Q3. Difference between Lazy Loading and Eager Loading?**
Lazy loading means related data (like an order's list of items) isn't fetched from the database until you actually access it in your code. Eager loading fetches the related data immediately, together with the main entity, in the very same query (or an additional one right away). Lazy loading is usually preferred for performance, since you avoid pulling data you might not need — but it can cause a `LazyInitializationException` if you try to access that data after the database session/transaction has already closed.

**Q4. What is the N+1 query problem?**
It happens when you fetch a list of N parent entities, and then, for each one, a separate query is triggered to fetch its related child data — resulting in 1 query for the parents plus N additional queries for each one's children, instead of one single efficient query. It's a very common performance issue with lazy-loaded relationships accessed in a loop, and it's usually fixed using a `JOIN FETCH` in your query, or Spring Data's `@EntityGraph`.

**Q5. What is the first-level cache and second-level cache in Hibernate?**
The first-level cache is enabled by default and scoped to a single Hibernate session — within one transaction, if you request the same entity twice, Hibernate returns the already-loaded object instead of hitting the database again. The second-level cache is optional, shared across sessions (and potentially across the whole application), and needs to be explicitly configured with a caching provider like Ehcache or Redis — useful for data that's read often but changes rarely, like reference/lookup tables.

**Q6. What is the difference between `save()`, `saveAndFlush()`, and `persist()`?**
`save()` (Spring Data JPA specific) inserts or updates an entity and may not immediately write to the database — it can be deferred until the transaction commits or a flush happens. `saveAndFlush()` does the same but forces an immediate write to the database right away, useful when you need the generated ID or updated state immediately, like right before running a native query. `persist()` is the underlying JPA method that only handles the "insert new entity" case and returns void — it doesn't return the managed entity like `save()` does.

---

## Session 15: Transactions

**Q1. What is `@Transactional` and how does it work?**
`@Transactional` tells Spring to wrap a method's execution inside a database transaction — all the database operations inside that method either fully succeed together, or fully roll back together if something goes wrong. Under the hood, Spring uses a proxy around your bean; when you call an `@Transactional` method, you're actually calling the proxy, which starts a transaction, invokes your real method, and then commits or rolls back based on whether an exception was thrown.

**Q2. Why doesn't `@Transactional` work when you call a method from within the same class?**
Because Spring's `@Transactional` relies on proxies — Spring wraps your bean in a proxy object that intercepts external calls to add transactional behavior. If you call another `@Transactional` method directly on `this` from inside the same class, you're bypassing the proxy entirely and calling the real object directly, so none of that transactional logic kicks in. The usual fix is to move that method into a separate bean, or use self-injection/`AopContext` in specific cases.

**Q3. What are the ACID properties of a transaction?**
Atomicity means all operations in a transaction succeed together or fail together — no partial updates. Consistency means a transaction takes the database from one valid state to another, respecting all rules and constraints. Isolation means concurrent transactions don't interfere with each other's intermediate results. Durability means once a transaction is committed, the change is permanent, even if the system crashes right after.

**Q4. What is the default rollback behavior of `@Transactional`?**
By default, Spring only rolls back a transaction on unchecked exceptions (`RuntimeException` and its subclasses) — checked exceptions do NOT trigger a rollback by default. This surprises a lot of developers. If you want a checked exception to also cause a rollback, you have to explicitly declare it: `@Transactional(rollbackFor = SomeCheckedException.class)`.

**Q5. What are transaction isolation levels?**
They control how much one transaction can "see" of another transaction's uncommitted changes. From least to most strict: Read Uncommitted (can see other transactions' uncommitted changes — "dirty reads"), Read Committed (only sees committed changes, but data might change between two reads in the same transaction), Repeatable Read (same query always returns the same rows within a transaction), and Serializable (transactions behave as if run one after another, fully isolated, but at the cost of performance). Most databases default to Read Committed.

---

## Session 16: Spring Security & JWT

**Q1. What is JWT and how does it work?**
JWT (JSON Web Token) is a compact, self-contained way to represent claims (like user ID, roles, expiry) as a signed token. It has three parts separated by dots: a header (algorithm info), a payload (the actual claims/data), and a signature (created using a secret key, so the server can verify the token hasn't been tampered with). After login, the server issues a JWT to the client; the client sends it back in the `Authorization` header on every subsequent request, and the server verifies the signature to confirm the request is genuinely from an authenticated user — without needing to store session state on the server.

**Q2. What's the difference between Authentication and Authorization?**
Authentication is about verifying who you are — logging in with a username/password and proving your identity. Authorization is about what you're allowed to do once you're identified — whether your role or permissions let you access a specific resource or perform a specific action. You always authenticate first, then authorization decisions are made based on that identity.

**Q3. Why is JWT considered stateless, and why does that matter?**
Traditional session-based authentication requires the server to store session data (who's logged in, what their session ID maps to) — meaning every server instance needs access to that shared session store. JWT carries all necessary information within the token itself, so any server instance can verify it independently just by checking the signature, without needing shared session storage. This makes JWT-based auth much easier to scale horizontally across multiple servers.

**Q4. What are some security concerns with JWT?**
Since a JWT can't be easily "revoked" once issued (unlike a server-side session you can simply delete), if a token is stolen, it remains valid until it expires — so short expiry times, combined with refresh tokens, are commonly used to limit the damage. Also, JWTs shouldn't store sensitive data in the payload since it's only encoded (Base64), not encrypted, and can be read by anyone who has the token.

**Q5. What is the Spring Security filter chain, in simple terms?**
Spring Security works by intercepting every incoming request through a chain of filters before it even reaches your controller — each filter handles one specific concern (checking for a JWT, validating credentials, checking CSRF tokens, enforcing authorization rules). If a request fails any required check along the chain, it's rejected immediately with an appropriate error, and never even reaches your actual business logic.

---

## Session 17: SQL & Databases

**Q1. Write an SQL query to find the second highest salary from an Employee table.**
```sql
SELECT MAX(salary) AS second_highest_salary
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);
```
This works by first finding the overall highest salary, then finding the maximum salary that's still strictly less than that — which by definition is the second highest.

**Q2. What is the difference between `INNER JOIN`, `LEFT JOIN`, and `RIGHT JOIN`?**
`INNER JOIN` returns only the rows that have matching values in both tables. `LEFT JOIN` returns all rows from the left table, plus matching rows from the right table — filling in `NULL` for the right side when there's no match. `RIGHT JOIN` is the mirror image — all rows from the right table, with `NULL` for unmatched left-side columns.

**Q3. Difference between `WHERE` and `HAVING`?**
`WHERE` filters individual rows before any grouping happens. `HAVING` filters groups after a `GROUP BY` has been applied — so you use `HAVING` when your condition involves an aggregate function, like filtering departments where `COUNT(*) > 5`, since you can't reference an aggregate in a `WHERE` clause.

**Q4. What is normalization, and why do we do it?**
Normalization is the process of organizing database tables to reduce data redundancy and avoid update anomalies — instead of repeating the same customer's address in every single order row, you store it once in a separate `Customer` table and just reference it by ID from `Orders`. This keeps data consistent (updating an address in one place updates it everywhere it's referenced) and saves storage.

**Q5. What is an index, and how does it improve performance?**
An index is a separate data structure (usually a B-tree) that the database maintains alongside a table, allowing it to look up rows matching a condition much faster than scanning the entire table row by row. It's similar to an index at the back of a textbook — instead of reading the whole book to find a topic, you jump straight to the page. The trade-off is that indexes speed up reads but slightly slow down writes (inserts/updates), since the index itself has to be updated too.

**Q6. What is a primary key vs a foreign key?**
A primary key uniquely identifies each row in a table and can't be `NULL` or duplicated. A foreign key is a column in one table that references the primary key of another table, enforcing a relationship between the two and preventing you from inserting a value that doesn't exist in the referenced table (referential integrity).

---

## Session 18: Exception Handling & Validation in Spring Boot

**Q1. How do you validate incoming request data in a Spring Boot REST API?**
You annotate your DTO fields with Bean Validation annotations like `@NotNull`, `@NotBlank`, `@Size`, `@Email`, `@Min`/`@Max`, then add `@Valid` in front of the `@RequestBody` parameter in your controller method. If validation fails, Spring automatically throws a `MethodArgumentNotValidException`, which you typically catch in a global exception handler to return a clean, structured error response instead of a generic stack trace.

**Q2. How would you structure a consistent error response format across your whole API?**
Create a standard error response class (like `ErrorResponse` with fields for timestamp, status code, error message, and path), and use a single `@RestControllerAdvice` class with multiple `@ExceptionHandler` methods — one for validation errors, one for your custom business exceptions, one as a catch-all for unexpected errors. This way, regardless of what fails, the client always receives errors in the same predictable JSON shape.

**Q3. What is the difference between `@ControllerAdvice` and `@RestControllerAdvice`?**
`@RestControllerAdvice` is `@ControllerAdvice` combined with `@ResponseBody`, meaning the return values from its exception-handler methods are automatically serialized directly into the HTTP response body (typically JSON) — exactly what you want for REST APIs. Plain `@ControllerAdvice` is meant for traditional MVC apps where you might be returning view names instead.

**Q4. How do you return a custom HTTP status code along with an error message from an exception handler?**
You wrap your error object in a `ResponseEntity`, explicitly setting the desired status code: `return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);` — or use `@ResponseStatus` on the custom exception class itself, so Spring automatically uses that status whenever that exception type is thrown.

---

## Session 19: Microservices Basics

**Q1. What is a microservices architecture, and how is it different from a monolith?**
A monolith is a single, large application where all modules (user management, payments, inventory, etc.) are built, deployed, and scaled together as one unit. Microservices break the application into small, independently deployable services, each focused on a specific business capability, communicating with each other over the network (usually REST or messaging). This allows teams to develop, deploy, and scale each service independently, but it introduces new complexity around network communication, data consistency, and monitoring.

**Q2. How do microservices typically communicate with each other?**
Synchronously, usually via REST APIs or gRPC, where one service directly calls another and waits for a response. Asynchronously, via a message broker like Kafka or RabbitMQ, where a service publishes an event and other services react to it independently, without waiting or being tightly coupled to each other's availability. Asynchronous communication is generally preferred where possible, since it decouples services and makes the system more resilient to one service being temporarily down.

**Q3. What is an API Gateway, and why is it needed?**
An API Gateway sits in front of all your microservices as a single entry point for clients — it routes incoming requests to the right internal service, and can also handle cross-cutting concerns like authentication, rate limiting, and logging in one central place instead of duplicating that logic in every single microservice.

**Q4. What is service discovery, and why do microservices need it?**
In a dynamic environment where service instances are constantly being created, destroyed, or rescaled (especially with containers/Kubernetes), hardcoding IP addresses to call another service simply doesn't work. Service discovery (tools like Eureka, or Kubernetes' built-in DNS-based discovery) lets services register themselves and look each other up by name at runtime, so calls are automatically routed to whatever healthy instances currently exist.

**Q5. What is the Circuit Breaker pattern, and why is it used?**
When one microservice calls another and that other service is slow or failing, repeatedly retrying can make things worse — it wastes resources and can cascade the failure across your whole system. A Circuit Breaker (like Resilience4j) monitors for repeated failures, and once a threshold is crossed, it "opens" and stops making the actual call for a while, immediately failing fast or returning a fallback response instead — giving the failing service time to recover, and protecting the rest of the system from cascading failure.

---

## Session 20: Behavioral & Project-Based Questions

**Q1. Explain your project and your role in it.**
Structure it as: what the project does at a high level (one or two sentences), the tech stack you used, your specific responsibilities/contributions (not just "the team's"), and one concrete metric or outcome if possible (performance improvement, bugs reduced, feature delivered on time). Avoid a purely chronological retelling — focus on what YOU built and decided.

**Q2. What challenges did you face in your project, and how did you solve them?**
Pick a real, technical challenge — not something vague like "tight deadlines." Describe the problem specifically, what approaches you considered, what you actually did, and the outcome. For example: "We had a performance issue where a report endpoint took 8 seconds because of an N+1 query problem; I identified it using query logs, fixed it with a JOIN FETCH, and got it down to under 500ms." Concrete, specific answers are always more convincing than general ones.

**Q3. Why should we hire you?**
This isn't about generic enthusiasm — it's about mapping your specific, demonstrated skills directly to what the role needs. Mention 2–3 concrete things: a relevant technical strength backed by a real example, your ability to learn quickly (with proof), and something specific about why this company/role interests you, so it doesn't sound copy-pasted for any company.

**Q4. Describe a time you disagreed with a teammate or lead on a technical decision.**
Interviewers want to see you can disagree professionally and back your position with reasoning, while still being able to accept a final decision gracefully. Structure it as: the situation, the differing viewpoints, how you presented your reasoning (with data/evidence if possible), and the resolution — even if it wasn't your preferred outcome, show what you learned or how the team benefited.

**Q5. How do you keep yourself updated with new technologies?**
Be specific and honest rather than giving a generic "I read blogs" answer — mention actual habits: following particular blogs (like Baeldung for Java), building small side projects to try things hands-on, specific courses, or communities you're part of. Backing it up with an example of something you recently learned and actually applied is much stronger than a vague claim.

---

## Session 21: System Design Basics (for 1–2 YOE level)

**Q1. At a junior/mid level, what kind of system design questions should you expect?**
Usually not full-scale "design Twitter" style questions — more likely lighter, practical ones like "how would you design a URL shortener," "how would you handle rate limiting on an API," or "how would you design a simple notification system." They're testing whether you can reason about trade-offs and structure, not whether you've memorized a specific architecture.

**Q2. How would you design a simple URL shortener at a high level?**
You'd need an endpoint that accepts a long URL and generates a short, unique code for it (often using a counter encoded in base62, or a hash with collision handling), storing the mapping in a database. A redirect endpoint looks up the short code and issues an HTTP redirect to the original URL. For scale, you'd add caching (like Redis) for frequently accessed short URLs to avoid hitting the database on every single redirect.

**Q3. How would you approach rate limiting an API?**
Common approaches: a fixed window counter (allow N requests per fixed time window, simple but can allow bursts at window boundaries), a sliding window (smoother, more accurate, slightly more complex), or a token bucket (tokens refill at a steady rate, and each request consumes a token — allows some burstiness while still enforcing an average rate). In practice, this is usually implemented using Redis for fast, shared counters across multiple server instances.

**Q4. What's the difference between vertical scaling and horizontal scaling?**
Vertical scaling means making a single server more powerful — adding more CPU, RAM. It's simple but has a hard ceiling and a single point of failure. Horizontal scaling means adding more servers/instances and distributing the load across them (usually behind a load balancer) — it scales further and adds redundancy, but introduces complexity around state management, since you can no longer assume all requests hit the same instance.

**Q5. What is caching, and where would you typically use it in a backend system?**
Caching stores frequently accessed data in a fast-access layer (usually in-memory, like Redis) so you don't have to repeatedly hit a slower resource (like a database or an external API) for the same data. Common use cases: caching user session data, frequently-read reference data (like product categories), or expensive computation results. The tricky part is cache invalidation — making sure cached data doesn't go stale when the underlying data changes.

---

## Session 22: Testing (JUnit & Mockito)

**Q1. What is the difference between Unit Testing and Integration Testing?**
Unit testing verifies a single unit of code (usually one method or class) in complete isolation, mocking out all its dependencies — it's fast and pinpoints exactly where a bug is. Integration testing verifies that multiple components actually work correctly together — like a controller, service, and real (or in-memory) database all interacting — which is slower but catches issues that only show up when pieces are wired together.

**Q2. What is Mockito, and why do we use it?**
Mockito is a mocking framework that lets you create fake ("mock") versions of a class's dependencies, so you can test a class in isolation without needing the real database, real external API calls, or other real dependencies. You define what a mock should return when a specific method is called (`when(...).thenReturn(...)`), letting you test your logic under controlled, predictable conditions.

**Q3. What is the difference between `@Mock` and `@InjectMocks`?**
`@Mock` creates a fake/mocked instance of a dependency — for example, mocking a `Repository` so it doesn't hit a real database. `@InjectMocks` creates a real instance of the class you're actually testing (like a `Service`), and automatically injects the `@Mock` objects into it, so your test exercises the real logic of that class while its dependencies are all faked.

**Q4. What is Test-Driven Development (TDD)?**
TDD is writing the test for a piece of functionality before writing the actual implementation — you write a failing test first, then write just enough code to make it pass, then refactor while keeping the test green. It forces you to think about the expected behavior and edge cases upfront, and gives you a safety net of tests as a natural side effect of the process.

**Q5. What are some good practices for writing meaningful unit tests?**
Test one specific behavior per test method, and name the method descriptively (like `shouldThrowExceptionWhenBalanceIsInsufficient`) so a failing test tells you exactly what broke. Cover edge cases, not just the "happy path" — empty lists, nulls, boundary values. Keep tests independent of each other (no test should depend on another test having run first), and avoid testing implementation details that could change without actually breaking behavior — test what the code does, not how it does it internally.

---

## Session 23: Miscellaneous Core Concepts

**Q1. What is the difference between `this` and `super`?**
`this` refers to the current instance of the class you're in — used to refer to its own fields/methods, or to call another constructor in the same class. `super` refers to the immediate parent class — used to call the parent's constructor, or to access a parent method/field that's been overridden or shadowed by the current class.

**Q2. What is the difference between an abstract class and a normal class with all default method implementations?**
An abstract class can have unimplemented (abstract) methods, and importantly, you cannot instantiate it directly — you're forced to create a subclass and implement those abstract methods. A normal class with all methods already implemented can be instantiated on its own, and there's no compiler-enforced requirement for subclasses to override anything.

**Q3. What is the `instanceof` operator used for?**
It checks whether an object is an instance of a specific class or implements a specific interface, returning true/false — commonly used before casting an object to a more specific type, to avoid a `ClassCastException`. Since Java 16, you can also use "pattern matching for instanceof," which lets you check and cast in one step: `if (obj instanceof String s) { ... }`.

**Q4. What is the difference between composition and inheritance, and when would you prefer one over the other?**
Inheritance models an "is-a" relationship (a `SavingsAccount` is an `Account`) and reuses behavior by extending a parent class. Composition models a "has-a" relationship (a `Car` has an `Engine`) by including another object as a field, and delegating to it as needed. Composition is generally preferred where possible because it's more flexible — you can change behavior at runtime by swapping the composed object, and it avoids the tight coupling and fragile hierarchies that deep inheritance chains can create.

**Q5. What is the difference between `equals()` and `hashCode()` contract?**
If two objects are equal according to `equals()`, they must produce the same `hashCode()` — this is a strict rule. The reverse isn't required — two objects can have the same hash code without being equal (a "collision"), which is normal. Breaking this contract (overriding one but not the other) causes subtle, hard-to-debug bugs in hash-based collections like `HashMap` and `HashSet`, where lookups silently fail even though the object logically "exists" in the collection.

---

## Session 24: Rapid Fire Round

**Q1. Is Java pass-by-value or pass-by-reference?**
Java is strictly pass-by-value, always. For object references specifically, the *value of the reference* (essentially a pointer/address) is what gets copied and passed — so you can modify the object's internal state through that reference, but you can't make the original reference variable itself point to a completely different object from inside the method.

**Q2. What is the difference between `sleep()` and `wait()`?**
`sleep()` pauses the current thread for a specified time without releasing any locks it holds. `wait()` (called on an object, inside a synchronized block) releases the lock it holds and pauses until another thread calls `notify()`/`notifyAll()` on that same object — it's used for coordination between threads, not just pausing.

**Q3. Can an interface have a constructor?**
No — interfaces can't be instantiated on their own, so there's no concept of a constructor for them, even with default methods in modern Java.

**Q4. What is autoboxing and unboxing?**
Autoboxing is Java automatically converting a primitive type into its corresponding wrapper object (like `int` to `Integer`) when needed — for example, when adding an `int` into a `List<Integer>`. Unboxing is the reverse — converting a wrapper object back into its primitive form automatically, like when you use an `Integer` in an arithmetic expression.

**Q5. What is the difference between `String s1 = "abc"` and `String s2 = new String("abc")`?**
`"abc"` written as a literal is stored in the String pool, and Java reuses the same object if the identical literal appears elsewhere. `new String("abc")` explicitly forces the creation of a brand-new object on the heap, separate from the pool, even if `"abc"` already exists there — so `s1 == s2` would be false, even though `s1.equals(s2)` is true.

**Q6. What is the diamond problem, and how does Java avoid it?**
The diamond problem occurs in multiple inheritance when a class inherits from two classes that both define the same method, creating ambiguity about which version to use. Java avoids this for classes by only allowing single inheritance. For interfaces with default methods (which technically can cause a similar conflict), Java forces you to explicitly override and resolve the conflict yourself if two interfaces provide clashing default methods.

**Q7. What is the purpose of the `transient` keyword?**
It marks a field so that it's skipped during Java's default serialization process — useful for fields that shouldn't be persisted, like a cached value, a password, or a field that isn't itself serializable (like a `Thread` or a database connection object).

---

## Session 25: Wrap-Up Round — Explaining Trade-offs

**Q1. When would you choose `ArrayList` over `LinkedList` and vice versa, in a real project?**
Choose `ArrayList` by default for almost all cases — most real-world usage is more about reading/iterating than inserting in the middle, and `ArrayList` has better memory locality (faster in practice due to CPU caching), even though its theoretical middle-insertion complexity is worse. `LinkedList` only makes sense when you specifically need frequent insertions/removals at the beginning or end, like implementing a queue or deque — and even then, `ArrayDeque` is often a better-performing choice than `LinkedList` for that purpose.

**Q2. When would you use `@Transactional(readOnly = true)`?**
For methods that only fetch data and never modify it, marking the transaction as read-only lets Hibernate skip some internal bookkeeping (like dirty-checking to detect changes), which is a small performance optimization. It also serves as documentation — signaling to other developers reading the code that this method is not expected to change any data.

**Q3. In a real production system, would you prefer synchronous REST calls or asynchronous messaging between two services, and why?**
It depends on the use case: if the calling service genuinely needs an immediate response to continue (like validating a payment before showing a confirmation), synchronous REST makes sense. If the operation can happen in the background without the caller waiting (like sending a confirmation email after an order is placed), asynchronous messaging is better — it decouples the two services, so if the email service is temporarily down, it doesn't block or fail the main order flow; the message just waits in the queue until the service recovers.

**Q4. Would you always use `HashMap`, or are there cases you'd deliberately pick something else?**
`HashMap` is the right default for most general key-value lookups. But if you need predictable iteration order matching insertion, `LinkedHashMap` is better. If you need keys sorted automatically, `TreeMap` is the right call, accepting the slightly slower log(n) operations. And if multiple threads will access the map concurrently, `ConcurrentHashMap` is necessary instead of wrapping a `HashMap` with external synchronization.

**Q5. How do you decide between adding a new field via inheritance vs. composition in a real project?**
Ask whether the relationship is genuinely "is-a" and whether the subclass will always behave consistently as a specialized version of the parent in every context it's used — if there's any chance that assumption breaks down later, composition is safer, since it's much easier to change a composed dependency than to untangle a deep, brittle inheritance hierarchy after the fact. In real production codebases, composition is generally favored unless the "is-a" relationship is very clean and unlikely to change.

---

## How to use this document
- Don't just read the answers — say them out loud once each, in your own words. Reading and speaking use different memory pathways, and interviewers notice when an answer sounds memorized vs. understood.
- Pick 2–3 sessions a day rather than trying to cover all 25 in one sitting — spaced repetition sticks better than cramming.
- For every "explain X" answer, try to also prepare one small, real example from your own project at Wipro — interviewers respond much better to "here's how I actually used this" than a textbook definition alone.
