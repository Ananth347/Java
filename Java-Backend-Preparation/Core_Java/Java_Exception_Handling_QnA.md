# Java Exception Handling — Interview Q&A Format (With Clear Explanations)

> Every question is answered with a simple explanation, and code examples wherever needed. Structured basic to advanced, step by step.

---

## Table of Contents
- [Section A: Basics of Exception Handling](#section-a-basics-of-exception-handling)
- [Section B: Exception Hierarchy](#section-b-exception-hierarchy)
- [Section C: try, catch, finally](#section-c-try-catch-finally)
- [Section D: Checked vs Unchecked Exceptions](#section-d-checked-vs-unchecked-exceptions)
- [Section E: throw vs throws](#section-e-throw-vs-throws)
- [Section F: Custom (User-Defined) Exceptions](#section-f-custom-user-defined-exceptions)
- [Section G: Multi-catch, Nested try, and Order of Catch Blocks](#section-g-multi-catch-nested-try-and-order-of-catch-blocks)
- [Section H: try-with-resources](#section-h-try-with-resources)
- [Section I: Exception Chaining](#section-i-exception-chaining)
- [Section J: Common Built-in Exceptions](#section-j-common-built-in-exceptions)
- [Section K: Best Practices](#section-k-best-practices)
- [Section L: Tricky/Advanced Questions](#section-l-trickyadvanced-questions)

---

## Section A: Basics of Exception Handling

### Q1. What is an Exception in Java?
**Answer:** An **Exception** is an event that disrupts the normal flow of a program's execution. It occurs during runtime when something unexpected happens — like dividing by zero, accessing an invalid array index, or trying to open a file that doesn't exist. Java represents exceptions as **objects**, and provides a mechanism to "catch" and handle them gracefully, instead of letting the program crash abruptly.

```java
int a = 10 / 0;   // throws ArithmeticException at runtime
```

### Q2. What is Exception Handling? Why is it needed?
**Answer:** Exception Handling is the process of **responding to unexpected events (exceptions)** in a controlled way, so the program doesn't crash and can either recover, log the issue, or fail gracefully with a meaningful message.

**Why needed?**
- Prevents abrupt program termination
- Allows meaningful error messages to be shown to the user
- Allows cleanup of resources (closing files, DB connections) even when errors occur
- Separates normal program logic from error-handling logic, making code cleaner
- Helps in debugging by preserving the error's context (stack trace)

### Q3. What is an Error, and how is it different from an Exception?
**Answer:**
- An **Exception** represents a problem that a well-written application **should try to catch and handle** (e.g., invalid user input, file not found) — often recoverable
- An **Error** represents a **serious problem**, usually outside the application's control, related to the environment in which the application runs (e.g., running out of memory, stack overflow). Errors are generally **not meant to be caught/handled** by application code — they indicate something seriously wrong at the JVM/system level.

```java
StackOverflowError     // an Error - recursion went too deep
OutOfMemoryError       // an Error - JVM ran out of heap space
```

### Q4. What is the difference between Exception and Error?

| Exception | Error |
|-----------|-------|
| Represents a recoverable condition | Represents a serious, usually unrecoverable condition |
| Caused by the application logic (bad input, bugs) | Caused by the environment/JVM (memory, stack) |
| Should be caught/handled | Generally should NOT be caught/handled |
| Example: `NullPointerException`, `IOException` | Example: `OutOfMemoryError`, `StackOverflowError` |
| Subclass of `java.lang.Exception` | Subclass of `java.lang.Error` |

### Q5. What is a "Throwable" in Java?
**Answer:** `Throwable` is the **topmost parent class** for anything that can be thrown and caught in Java's exception mechanism. Both `Exception` and `Error` are direct subclasses of `Throwable`. This means both can be thrown using `throw` and caught using `catch`, though as explained above, catching `Error` types is usually discouraged.

---

## Section B: Exception Hierarchy

### Q6. Explain the complete Exception class hierarchy in Java.
**Answer:**
```
                     Object
                        │
                    Throwable
                    /        \
               Exception      Error
               /      \           \
     (Checked)    RuntimeException  (e.g. OutOfMemoryError,
   Exceptions      (Unchecked)       StackOverflowError)
   (e.g. IOException,  /    \
   SQLException)      /      \
                NullPointerException, ArithmeticException,
                ArrayIndexOutOfBoundsException,
                ClassCastException, NumberFormatException, etc.
```

- **`Throwable`** — the root class for all errors and exceptions
- **`Exception`** — for conditions a program might want to catch; splits into **Checked** (direct subclasses of `Exception`, excluding `RuntimeException`) and **Unchecked** (`RuntimeException` and its subclasses)
- **`Error`** — for serious problems the application usually shouldn't try to catch

### Q7. Is `RuntimeException` a checked or unchecked exception?
**Answer:** `RuntimeException` (and ALL of its subclasses, like `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException`) is **unchecked**. Unchecked exceptions are NOT required to be declared or caught by the compiler.

### Q8. Name some commonly occurring built-in exceptions in Java.
**Answer:**
- `NullPointerException` — trying to use a reference that points to `null`
- `ArithmeticException` — invalid arithmetic operation (like dividing an int by zero)
- `ArrayIndexOutOfBoundsException` — accessing an array index that doesn't exist
- `ClassCastException` — invalid typecasting between incompatible types
- `NumberFormatException` — trying to convert an invalid String to a number (e.g., `Integer.parseInt("abc")`)
- `IllegalArgumentException` — a method received an argument that is inappropriate
- `IOException` — problem during input/output operations (checked)
- `SQLException` — problem during database operations (checked)
- `FileNotFoundException` — a file being accessed doesn't exist (checked)

---

## Section C: try, catch, finally

### Q9. What is the purpose of `try`, `catch`, and `finally` blocks?
**Answer:**
- **`try`** — wraps the code that might throw an exception
- **`catch`** — catches and handles a specific type of exception thrown inside the `try` block
- **`finally`** — contains code that **always executes**, whether an exception occurred or not (typically used for cleanup, like closing files/connections)

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Cannot divide by zero: " + e.getMessage());
} finally {
    System.out.println("This always runs");
}
```

### Q10. Can we have a `try` block without a `catch` block?
**Answer:** Yes, but only if it's followed by a `finally` block (`try-finally`), OR if it's a `try-with-resources` statement (Java 7+, explained later). A plain `try` block alone (with neither `catch` nor `finally`) is **not allowed** — it will cause a compile-time error.

```java
try {
    // some code
} finally {
    // cleanup code - this is valid even without catch
}
```

### Q11. Can we have multiple `catch` blocks for a single `try` block?
**Answer:** Yes. This is useful when the `try` block might throw different types of exceptions, and you want to handle each one differently.

```java
try {
    int[] arr = new int[5];
    arr[10] = 50 / 0;
} catch (ArithmeticException e) {
    System.out.println("Arithmetic problem");
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Array index problem");
}
```

### Q12. Does `finally` ALWAYS execute, no matter what?
**Answer:** `finally` executes in almost all cases — whether an exception is thrown or not, and even if the `try`/`catch` block has a `return` statement. There are only a few rare exceptions to this:
- If `System.exit()` is called inside the `try` or `catch` block (this terminates the JVM immediately)
- If the JVM crashes or is forcibly killed (e.g., power failure, `kill -9` on the process)
- If the thread executing the `try` block is killed/interrupted abruptly

### Q13. If both `try` and `finally` have a `return` statement, which one wins?
**Answer:** The `return` in the `finally` block **overrides** the `return` in the `try`/`catch` block. This is considered **bad practice** because it silently discards the original return value (and even silently discards any exception that was about to be thrown!), which can hide bugs.

```java
static int test() {
    try {
        return 1;
    } finally {
        return 2;   // this value (2) is what actually gets returned!
    }
}
```

### Q14. What happens if an exception occurs inside the `finally` block itself?
**Answer:** If the `finally` block itself throws an exception, it will **override/replace** any exception that was being propagated from the `try` or `catch` block. The original exception is effectively lost (unless you handle it carefully, e.g., using suppressed exceptions in try-with-resources).

### Q15. Can we nest a `try-catch` block inside another `try` block?
**Answer:** Yes, this is called **Nested try-catch**. It's useful when a certain block of risky code within an already-risky block needs its own separate handling.

```java
try {
    try {
        int x = 10 / 0;
    } catch (ArrayIndexOutOfBoundsException e) {
        System.out.println("Inner catch - won't catch ArithmeticException");
    }
} catch (ArithmeticException e) {
    System.out.println("Outer catch handles it: " + e.getMessage());
}
```

---

## Section D: Checked vs Unchecked Exceptions

### Q16. What is a Checked Exception? Give examples.
**Answer:** A **Checked Exception** is an exception that the **compiler forces you to handle** — either by catching it with a `try-catch`, or by declaring it in the method signature using `throws`. If you don't do either, your code won't even compile. Checked exceptions represent conditions that a well-written program should anticipate and recover from (e.g., a file might not exist, a network connection might fail).

**Examples:** `IOException`, `SQLException`, `FileNotFoundException`, `ClassNotFoundException`

```java
void readFile() throws IOException {   // must declare or handle
    FileReader fr = new FileReader("data.txt");
}
```

### Q17. What is an Unchecked Exception? Give examples.
**Answer:** An **Unchecked Exception** is an exception that the compiler does **NOT force you to handle**. These usually represent **programming bugs/mistakes** (like a null reference, invalid array index, bad arithmetic) that could technically occur ANYWHERE in the code, so forcing every method to declare them would be impractical. All unchecked exceptions are subclasses of `RuntimeException`.

**Examples:** `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException`, `ClassCastException`, `NumberFormatException`, `IllegalArgumentException`

### Q18. What is the key difference between Checked and Unchecked exceptions?

| Checked Exception | Unchecked Exception |
|--------------------|------------------------|
| Checked at COMPILE time | Checked at RUNTIME |
| Must be handled (try-catch) or declared (`throws`) | No such requirement |
| Represents predictable, recoverable external conditions | Represents programming logic errors/bugs |
| Direct subclass of `Exception` (excluding `RuntimeException`) | Subclass of `RuntimeException` |
| Example: `IOException`, `SQLException` | Example: `NullPointerException`, `ArithmeticException` |

### Q19. Why does Java have the concept of Checked Exceptions? Is this design controversial?
**Answer:** The idea behind checked exceptions is to **force developers to think about and handle failure conditions** that are outside their control (like external file systems, network, databases) — improving reliability. However, this design IS somewhat **controversial** in the programming world. Critics argue that checked exceptions:
- Lead to messy code with excessive `try-catch` blocks or exceptions being blindly propagated with `throws`
- Encourage bad practices like catching an exception and doing nothing (`catch (Exception e) {}`), just to satisfy the compiler
- Many modern languages (like Kotlin, C#) deliberately do NOT have checked exceptions, considering it a design mistake

### Q20. Is `NullPointerException` checked or unchecked?
**Answer:** Unchecked. It's a subclass of `RuntimeException`, and typically represents a programming bug (forgetting to initialize an object before using it) rather than an external, predictable failure condition.

---

## Section E: throw vs throws

### Q21. What is the `throw` keyword used for?
**Answer:** `throw` is used to **explicitly throw an exception object** yourself, from within your code — either a built-in exception or a custom one.

```java
void checkAge(int age) {
    if (age < 18) {
        throw new IllegalArgumentException("Age must be 18 or above");
    }
}
```

### Q22. What is the `throws` keyword used for?
**Answer:** `throws` is used in a **method signature** to declare that this method MIGHT throw one or more checked exceptions, and it is NOT handling them itself — the responsibility is passed to whoever calls this method.

```java
void readFile(String path) throws IOException {
    FileReader fr = new FileReader(path);  // might throw IOException
}
```

### Q23. Difference between `throw` and `throws`?

| `throw` | `throws` |
|---------|----------|
| Used to actually throw an exception instance | Used to declare that a method might throw an exception |
| Used INSIDE a method body | Used in the METHOD SIGNATURE |
| Followed by an exception OBJECT: `throw new Exception();` | Followed by exception CLASS NAME(s): `throws IOException` |
| Can only throw ONE exception at a time | Can declare MULTIPLE exceptions, comma-separated |
| Used for both checked and unchecked exceptions | Mainly relevant for checked exceptions |

```java
void process() throws IOException, SQLException {   // throws: declares possibility
    if (someCondition) {
        throw new IOException("File error");          // throw: actually throws it
    }
}
```

### Q24. Can we throw multiple exceptions from a single method?
**Answer:** A method can only throw ONE exception at a time when it actually executes (`throw` triggers only one exception object per execution path). But a method CAN **declare** multiple possible exception types in its signature using `throws`, separated by commas:
```java
void method() throws IOException, SQLException, ClassNotFoundException { }
```

---

## Section F: Custom (User-Defined) Exceptions

### Q25. What is a Custom (User-Defined) Exception? Why do we create one?
**Answer:** A Custom Exception is an exception class that YOU define, usually by extending `Exception` (for a checked exception) or `RuntimeException` (for an unchecked exception). We create custom exceptions when built-in exceptions don't clearly express the specific error condition of our own business logic.

```java
class InsufficientBalanceException extends Exception {
    public InsufficientBalanceException(String message) {
        super(message);
    }
}

class BankAccount {
    double balance = 1000;
    void withdraw(double amount) throws InsufficientBalanceException {
        if (amount > balance) {
            throw new InsufficientBalanceException("Insufficient funds for this withdrawal");
        }
        balance -= amount;
    }
}
```

### Q26. Should a custom exception extend `Exception` or `RuntimeException`?
**Answer:** It depends on the situation:
- Extend **`Exception`** if you want it to be a **checked exception** — meaning callers of your method are FORCED to handle it (good when the failure is expected/recoverable and the caller genuinely needs to deal with it, e.g., `InsufficientBalanceException`)
- Extend **`RuntimeException`** if you want it to be an **unchecked exception** — meaning callers are NOT forced to handle it (good for representing programming errors or conditions that shouldn't normally occur, so you don't want to clutter every calling method with `try-catch`)

### Q27. What are the best practices when creating a custom exception?
**Answer:**
- Name it clearly, ending with "Exception" (e.g., `InvalidUserInputException`)
- Provide multiple constructors — at least a no-arg one, one accepting a `String message`, and ideally one accepting `(String message, Throwable cause)` for exception chaining
- Always call `super(message)` to properly set the exception's message
- Keep it focused — don't create a generic "MyException" for everything; create specific exceptions for specific failure conditions
- Document what conditions cause this exception to be thrown (using Javadoc comments)

---

## Section G: Multi-catch, Nested try, and Order of Catch Blocks

### Q28. What is Multi-catch (Java 7+)? Give an example.
**Answer:** Multi-catch allows you to catch **multiple exception types in a single `catch` block**, using the `|` (pipe) symbol, if you want to handle them the exact same way. This reduces code duplication.

```java
try {
    // risky code
} catch (IOException | SQLException e) {
    System.out.println("Error occurred: " + e.getMessage());
}
```

**Rule:** The exception types combined with `|` must NOT have a parent-child relationship with each other (you can't combine `Exception` and `IOException` together, since `IOException` is already a subtype of `Exception` — that would be redundant/ambiguous).

### Q29. Does the ORDER of `catch` blocks matter?
**Answer:** **Yes, absolutely.** You must always put **more specific exception types BEFORE more general ones**. If you put a general exception (like `Exception`) FIRST, it will catch everything, and the more specific `catch` blocks written after it become **unreachable code**, causing a **compile-time error**.

```java
try {
    int x = 10 / 0;
} catch (Exception e) {                 // too general - catches everything
    System.out.println("General error");
} catch (ArithmeticException e) {       // COMPILE ERROR: unreachable code!
    System.out.println("Arithmetic error");
}
```

**Correct order:**
```java
try {
    int x = 10 / 0;
} catch (ArithmeticException e) {       // specific FIRST
    System.out.println("Arithmetic error");
} catch (Exception e) {                 // general LAST
    System.out.println("General error");
}
```

### Q30. What happens if none of the `catch` blocks match the thrown exception?
**Answer:** The exception is NOT caught by this `try-catch`, and it **propagates up the call stack** to the calling method. If it's still not caught anywhere up the chain, it eventually reaches the JVM's default exception handler, which prints the **stack trace** to the console and **terminates the thread** (or the whole program, if it's the main thread).

---

## Section H: try-with-resources

### Q31. What is try-with-resources? Why was it introduced?
**Answer:** Introduced in **Java 7**, `try-with-resources` is a special form of the `try` statement that **automatically closes resources** (like file streams, database connections, sockets) when the `try` block finishes — whether it completes normally or throws an exception. This eliminates the need to manually write a `finally` block just to close resources, reducing boilerplate code and preventing resource leaks (a very common bug before Java 7).

```java
// OLD WAY (before Java 7) - verbose and error-prone
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("data.txt"));
    System.out.println(br.readLine());
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (br != null) {
        try { br.close(); } catch (IOException e) { e.printStackTrace(); }
    }
}

// NEW WAY (Java 7+) - try-with-resources
try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    System.out.println(br.readLine());
} catch (IOException e) {
    e.printStackTrace();
}
// br is automatically closed here, no need for finally!
```

### Q32. What is required for a class to be used inside try-with-resources?
**Answer:** The class must implement the **`AutoCloseable`** interface (or its subinterface `Closeable`, commonly used for I/O classes), which requires implementing a single method: `close()`. When the `try` block finishes (normally or due to an exception), the JVM automatically calls `.close()` on the resource(s).

```java
class MyResource implements AutoCloseable {
    public void close() {
        System.out.println("Resource closed automatically");
    }
}
try (MyResource r = new MyResource()) {
    System.out.println("Using resource");
}
```

### Q33. Can we declare multiple resources in a single try-with-resources statement?
**Answer:** Yes, separated by semicolons. They are closed in the **reverse order** of their declaration (the last opened resource is closed first).

```java
try (FileReader fr = new FileReader("a.txt");
     BufferedReader br = new BufferedReader(fr)) {
    // use both resources
}
// br.close() is called FIRST, then fr.close()
```

### Q34. What are "suppressed exceptions" in try-with-resources?
**Answer:** If BOTH the `try` block AND the automatic `close()` call throw exceptions, the exception from the `try` block is the one that actually gets thrown/propagated, and the exception from `close()` gets attached to it as a **"suppressed exception"** (instead of being lost, as would happen with old-style manual `finally` blocks). You can retrieve suppressed exceptions using `getSuppressed()` on the main exception.

---

## Section I: Exception Chaining

### Q35. What is Exception Chaining (or "Exception Wrapping")?
**Answer:** Exception Chaining is the practice of **catching one exception and throwing a different, more meaningful exception**, while still preserving the ORIGINAL exception as the "cause". This is useful when a low-level technical exception (like `SQLException`) needs to be translated into a more meaningful, higher-level exception for the calling code (like `UserNotFoundException`), without losing the original debugging information.

```java
class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message, Throwable cause) {
        super(message, cause);   // preserves the original cause
    }
}

void findUser(int id) {
    try {
        // some database call that throws SQLException
    } catch (SQLException e) {
        throw new UserNotFoundException("Could not find user with id " + id, e);  // chaining
    }
}
```

### Q36. How do you retrieve the original ("root") cause of a chained exception?
**Answer:** Using the `.getCause()` method, which returns the original `Throwable` that was passed when the new exception was constructed. You can even chain through multiple layers by calling `.getCause()` repeatedly until it returns `null` (meaning you've reached the true root cause).

```java
try {
    findUser(5);
} catch (UserNotFoundException e) {
    System.out.println("Error: " + e.getMessage());
    System.out.println("Root cause: " + e.getCause());   // prints the original SQLException
}
```

---

## Section J: Common Built-in Exceptions

### Q37. What causes a `NullPointerException`?
**Answer:** It occurs when you try to use a reference variable that currently points to `null`, as if it were pointing to a real object — like calling a method on it, or accessing a field.
```java
String s = null;
System.out.println(s.length());  // NullPointerException
```

### Q38. What causes an `ArrayIndexOutOfBoundsException`?
**Answer:** It occurs when you try to access an array element using an index that is either negative or greater than/equal to the array's length.
```java
int[] arr = new int[5];   // valid indices: 0 to 4
System.out.println(arr[5]);  // ArrayIndexOutOfBoundsException
```

### Q39. What causes a `ClassCastException`?
**Answer:** It occurs when you try to cast an object to a type that it's not actually compatible with, at runtime.
```java
Object obj = "Hello";        // obj is actually a String
Integer num = (Integer) obj; // ClassCastException - String cannot be cast to Integer
```

### Q40. What causes a `NumberFormatException`?
**Answer:** It occurs when you try to convert a String into a numeric type, but the String's content is not a valid number format.
```java
int x = Integer.parseInt("abc");  // NumberFormatException
```

### Q41. What causes a `ConcurrentModificationException`?
**Answer:** It occurs when a collection (like an `ArrayList`) is **structurally modified** (elements added/removed) while it's being **iterated** using a `for-each` loop or `Iterator`, without using the iterator's own safe removal methods.
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));
for (Integer i : list) {
    if (i == 2) list.remove(i);   // ConcurrentModificationException
}
```
**Fix:** Use `Iterator.remove()` instead of modifying the list directly during iteration, or use classes designed for concurrent modification like `CopyOnWriteArrayList`.

---

## Section K: Best Practices

### Q42. What are some best practices for exception handling in Java?
**Answer:**
1. **Catch specific exceptions, not generic ones** — avoid `catch (Exception e)` unless truly necessary; it can hide bugs by catching things you didn't intend to catch
2. **Never leave a `catch` block empty** — at minimum, log the exception; silently swallowing exceptions makes debugging a nightmare
3. **Don't use exceptions for normal program flow/control** — exceptions are relatively expensive (creating a stack trace has a performance cost) and should represent truly exceptional conditions, not routine logic
4. **Always clean up resources** — use `try-with-resources` instead of manual `finally` blocks wherever possible
5. **Preserve the original exception** when wrapping it in a new one (use exception chaining, pass the original as the "cause")
6. **Include meaningful messages** in exceptions — a message like "Error" is useless; "Failed to connect to database at host X" is helpful
7. **Don't catch `Throwable` or `Error`** unless you have an extremely specific, well-understood reason — you generally can't safely recover from Errors
8. **Fail fast** — validate inputs early and throw exceptions immediately when something is wrong, rather than letting bad data propagate deep into the system
9. **Document exceptions** your public methods can throw, using Javadoc `@throws` tags

### Q43. Why is it considered bad practice to catch a generic `Exception` (or `Throwable`)?
**Answer:** Because it can accidentally catch and "swallow" exceptions you never intended to handle — including serious bugs (like `NullPointerException` from a real programming error) that you'd actually want to know about and fix, not silently hide. It makes debugging much harder, since the real underlying problem gets masked.

### Q44. Why should exceptions not be used for normal control flow?
**Answer:** Exceptions in Java are relatively **expensive to create**, mainly because constructing an exception object captures the entire **stack trace** at that point in time — this has real performance overhead. Using exceptions for routine logic (like using an exception to break out of a loop, instead of a proper condition) is both a performance anti-pattern and makes code harder to read/understand, since exceptions are meant to signal something UNUSUAL, not everyday logic.

---

## Section L: Tricky/Advanced Questions

### Q45. What is the difference between `Exception` and `RuntimeException`?
**Answer:** `RuntimeException` is actually a **subclass of `Exception`**. The key practical difference: any class that extends `Exception` directly (not through `RuntimeException`) becomes a **checked exception**, while any class that extends `RuntimeException` becomes an **unchecked exception**. So the distinction isn't about the class hierarchy itself, but about WHERE in that hierarchy your exception sits.

### Q46. Can we catch an `Error` in Java? Should we?
**Answer:** Technically, yes — `Error` extends `Throwable`, and you can write `catch (Error e)` or even `catch (Throwable t)`. However, it's generally **not recommended**, because Errors (like `OutOfMemoryError`, `StackOverflowError`) usually indicate a serious problem with the JVM/environment itself that application code typically cannot meaningfully recover from.

### Q47. What is the difference between `final`, `finally`, and `finalize()`?
**Answer:** These three sound similar but are completely different concepts:

| Keyword | Purpose |
|---------|---------|
| `final` | A modifier — makes a variable a constant, a method un-overridable, or a class un-extendable |
| `finally` | A block in exception handling that always executes after try/catch, used for cleanup |
| `finalize()` | A method (now **deprecated** since Java 9, removed in later versions) that used to be called by the Garbage Collector just before an object was destroyed, to allow cleanup — replaced by better alternatives like try-with-resources and `Cleaner` |

### Q48. Can a `catch` block itself throw an exception?
**Answer:** Yes. If a `catch` block throws a NEW exception, it propagates up the call stack just like any other exception, and can be caught by an outer `try-catch` (if one exists) or propagate further up.

### Q49. What is the difference between `e.printStackTrace()` and `e.getMessage()`?
**Answer:**
- `e.getMessage()` returns only the **descriptive message String** that was passed when the exception was created (e.g., "File not found")
- `e.printStackTrace()` prints the **entire call stack trace** — showing the exception type, message, AND the exact sequence of method calls that led to the exception, which is far more useful for debugging

### Q50. If a method overrides another method, can it throw broader checked exceptions than the parent method?
**Answer:** **No.** An overriding method can throw the SAME checked exceptions as the parent method, FEWER checked exceptions, or more SPECIFIC (narrower/subclass) checked exceptions — but it **cannot throw new or broader checked exceptions** that the parent method didn't declare. This rule exists to preserve **polymorphism safety** — code calling the parent method through a reference shouldn't suddenly face an exception type it never expected.

```java
class Parent {
    void process() throws IOException { }
}
class Child extends Parent {
    void process() throws FileNotFoundException { }  // OK - FileNotFoundException IS-A IOException (narrower)
    // void process() throws Exception { }             // ERROR - broader than parent's IOException
}
```
(Note: this restriction applies only to CHECKED exceptions; unchecked exceptions have no such restriction.)

### Q51. What happens when an exception is thrown inside a `static` initializer block?
**Answer:** If a `static` block throws an exception that isn't a `RuntimeException` or `Error`, it must be either caught within the block or it will result in a compile-time error (checked exceptions can't propagate out of static initializers cleanly). If an unchecked exception occurs and isn't caught, it gets wrapped into an `ExceptionInInitializerError`, and the class fails to load, which can cause `NoClassDefFoundError` on subsequent attempts to use that class.

### Q52. Is it possible to have code AFTER a `finally` block that never executes?
**Answer:** Yes — if the `finally` block itself contains a `return`, `break`, `continue`, or throws an exception, control transfers immediately, and any code written after the `finally` block within the same flow might never be reached, OR it silently discards whatever was about to be returned/thrown from the `try`/`catch`. This is exactly why returning from `finally` is considered bad practice (see Q13).

### Q53. What is the difference between `Exception` and Compile-Time Errors (like syntax errors)?
**Answer:** A **compile-time error** (like a missing semicolon, undeclared variable) means the code doesn't even compile — it's a problem with the code's structure/syntax, caught by the compiler BEFORE the program ever runs. An **Exception** is a RUNTIME event — the code compiles fine and starts running, but something unexpected happens WHILE it's executing (like invalid user input or a network failure). They are completely different categories of problems.

### Q54. Can we re-throw the same exception after catching it?
**Answer:** Yes, this is a common pattern — you catch an exception (perhaps to log it or do partial cleanup), and then re-throw the SAME exception object so that it continues propagating up to the caller.
```java
try {
    riskyOperation();
} catch (IOException e) {
    logger.error("Something went wrong", e);
    throw e;   // re-throwing the same exception after logging it
}
```

### Q55. What's the difference between throwing a new exception vs re-throwing the caught exception?
**Answer:** Re-throwing the SAME caught exception (`throw e;`) preserves the ORIGINAL stack trace exactly as it was when the exception was first created. Throwing a brand NEW exception (`throw new SomeException(...)`) creates a fresh stack trace starting from that new `throw` point — if you want to preserve the original context/cause, you must explicitly pass the original exception as the "cause" (exception chaining, see Section I).

---

**That's a comprehensive Q&A on Java Exception Handling — from the basics of what an exception is, through checked/unchecked distinctions, try-with-resources, custom exceptions, and tricky edge cases interviewers commonly ask about.** Practice writing small test programs for each scenario — actually triggering these exceptions yourself is the fastest way to make this knowledge stick.
