# Java Exception Handling — The Complete Guide
### Every topic explained step by step, in simple, easy-to-read English

---

## How to read this document

Read this from top to bottom, in order — each part builds on the one before it. This document is written in plain, simple language on purpose, so every explanation should be easy to follow just by reading it once.

For every topic you'll get:
- **What it is** (in simple words)
- **Why it exists** (what problem it solves)
- **How it works** (so you understand it, not just memorize it)
- **Code example**
- **Common interview question**

---

# PART 1: THE BASICS — WHAT IS AN EXCEPTION?

## 1.1 What is an exception?

An exception is an **unexpected problem** that happens while your program is running. For example: you try to divide a number by zero, or you try to open a file that doesn't exist, or you try to use an object that is actually `null`. When this happens, Java stops the normal flow of the program and creates a special object (called an "exception object") that describes what went wrong.

## 1.2 Why does Java use exceptions instead of just crashing?

Without exception handling, a single small error (like a file not being found) would crash the entire program immediately, with no chance to fix it or tell the user what happened nicely. Exception handling lets you:
1. **Catch** the problem when it happens.
2. **Decide what to do** about it (show a friendly message, try again, use a backup plan, log it, etc.).
3. Keep the rest of the program running normally, instead of the whole application crashing.

## 1.3 A simple example of an exception happening

```java
public class Demo {
    public static void main(String[] args) {
        int result = 10 / 0; // this line causes a problem
        System.out.println("This line will never run");
    }
}
```

Output:
```
Exception in thread "main" java.lang.ArithmeticException: / by zero
    at Demo.main(Demo.java:3)
```

Here, dividing by zero is not allowed, so Java throws (creates and raises) an `ArithmeticException`. Because we didn't handle it, the program stops right there — the last line never runs.

## 1.4 What is a "stack trace"?

The text printed above (`at Demo.main(Demo.java:3)`) is called a **stack trace**. It tells you exactly:
- What went wrong (`ArithmeticException: / by zero`)
- Which method it happened in (`Demo.main`)
- Which exact line number it happened on (`Demo.java:3`)

This is extremely useful for finding and fixing bugs — always read the stack trace from the top first; that's usually where the real problem is.

---

# PART 2: THE EXCEPTION HIERARCHY (THE FAMILY TREE)

## 2.1 The full picture

Every exception in Java is a class, and all of these classes belong to one big family tree. Here is the shape of that tree — you don't need to memorize every class, just understand the shape:

```
                        Throwable
                       /          \
                  Error          Exception
                 /                /       \
        (OutOfMemoryError,    RuntimeException   (checked exceptions)
         StackOverflowError)    /        \        - IOException
                                /          \       - SQLException
                    (NullPointerException,   ...    - ClassNotFoundException
                     ArithmeticException,
                     ArrayIndexOutOfBoundsException,
                     ClassCastException, ...)
```

## 2.2 What is `Throwable`?

`Throwable` is the topmost class — everything that can be "thrown" in Java (using the `throw` keyword) must be a `Throwable` or one of its children. It has exactly two direct children: `Error` and `Exception`.

## 2.3 What is the difference between `Error` and `Exception`?

- **`Error`**: represents a serious problem that your program usually **cannot recover from**. These are typically caused by the environment the program runs in, not by a mistake in your own code's logic. Examples: `OutOfMemoryError` (the JVM ran out of memory), `StackOverflowError` (too many nested method calls, usually from infinite recursion). You almost never try to catch these, because there's usually nothing meaningful you can do to fix them at that point.
- **`Exception`**: represents a problem that your program **can, and usually should, recover from**. Examples: a file that doesn't exist, invalid user input, a network connection that dropped. This is the class you'll be working with almost all the time.

### Common interview angle
**Q: Should you ever catch an `Error`?**
Generally no. Errors like `OutOfMemoryError` mean something is seriously wrong at the system level, and trying to "handle" them usually won't fix anything — the safest thing is usually to let the program stop, fix the root cause (like a memory leak), and restart.

## 2.4 Checked exceptions vs. Unchecked exceptions — the most important distinction in this whole topic

This is the single most important thing to understand clearly, so let's go slowly.

### Checked exceptions
These are exceptions that the Java compiler **forces you to deal with** at compile time — meaning your code won't even compile unless you either catch the exception or declare that your method might throw it. They represent problems that are expected to sometimes happen in normal, correctly-written programs, and are considered outside of your code's direct control — like a file being missing, or a network connection failing.

```java
import java.io.*;

public void readFile() throws IOException { // MUST declare this, or the code won't compile
    FileReader file = new FileReader("data.txt"); // this line can throw IOException
}
```

If you remove `throws IOException` from the method above, the code simply **will not compile** — Java forces you to acknowledge this exception exists.

### Unchecked exceptions
These are exceptions the compiler does **not** force you to handle. They usually represent **programming mistakes** — bugs — like trying to use a `null` object, or accessing an array with an invalid index. Since these usually mean "something is wrong in the code itself," Java doesn't force you to write a `catch` block for every single one everywhere — that would make code messy, since almost any line of code could theoretically cause one of these.

```java
public void process(String name) {
    System.out.println(name.length()); // if name is null, throws NullPointerException — no "throws" needed, compiles fine
}
```

### How do you tell them apart just by looking at the class hierarchy?
This is the simple rule: **any exception that extends `RuntimeException` (directly or indirectly) is unchecked. Everything else that extends `Exception` (but not `RuntimeException`) is checked.**

```
Exception
   |
   +-- IOException              <- CHECKED (does not extend RuntimeException)
   +-- SQLException             <- CHECKED
   +-- RuntimeException          <- this and everything under it is UNCHECKED
          |
          +-- NullPointerException
          +-- ArithmeticException
          +-- ArrayIndexOutOfBoundsException
          +-- ClassCastException
          +-- IllegalArgumentException
```

### A simple table to lock this in

| | Checked Exception | Unchecked Exception |
|---|---|---|
| Extends | `Exception` (not `RuntimeException`) | `RuntimeException` |
| Compiler forces you to handle it? | Yes | No |
| Represents | Expected problems outside your code's control (file missing, network down) | Usually a bug in your own code (null pointer, bad array index) |
| Examples | `IOException`, `SQLException`, `ClassNotFoundException` | `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException` |

### Common interview angle
**Q: Why did Java's designers make some exceptions checked and others unchecked?**
The idea was: checked exceptions force developers to think about and handle problems that are genuinely likely to happen in correct programs due to outside factors (like a file not being there) — you can't always prevent these by writing better code. Unchecked exceptions are usually caused by a mistake in the code itself (like forgetting to check for `null`) — the fix for these is to write better code, not to force a `catch` block on every single line where they might theoretically happen.

**Q: Is this checked/unchecked design considered a good idea today?**
It's actually a bit controversial. Many developers find that checked exceptions can lead to messy code (empty catch blocks just to make the compiler happy, or `throws Exception` sprinkled everywhere without real thought). This is part of why newer Java-based frameworks (like Spring) often prefer using unchecked exceptions even for things like database errors, rather than forcing checked exceptions everywhere.

---

# PART 3: TRY, CATCH, AND FINALLY

## 3.1 The basic `try-catch` structure

```java
try {
    // code that might cause an exception
} catch (ExceptionType e) {
    // code that runs IF that exception happens
}
```

### Step by step, what actually happens
1. Java starts running the code inside `try`.
2. If a line inside `try` throws an exception, Java immediately **stops** running the rest of the `try` block (any code after the problem line is skipped).
3. Java looks at the `catch` block(s) to see if any of them match the type of exception that was thrown.
4. If a match is found, that `catch` block runs.
5. If no `try` block exception happens at all, all the `catch` blocks are simply skipped entirely, and the program continues normally after the whole try-catch.

```java
try {
    int result = 10 / 0;
    System.out.println("This never prints"); // skipped, because the line above already threw an exception
} catch (ArithmeticException e) {
    System.out.println("Cannot divide by zero!"); // this runs instead
}
System.out.println("Program continues normally"); // this always runs
```

## 3.2 Catching multiple types of exceptions

You can have several `catch` blocks after one `try`, to handle different kinds of problems differently.

```java
try {
    int[] numbers = {1, 2, 3};
    System.out.println(numbers[5]); // will throw ArrayIndexOutOfBoundsException
} catch (ArithmeticException e) {
    System.out.println("Math problem: " + e.getMessage());
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Array problem: " + e.getMessage());
}
```

### An important rule: order matters!
If one exception type is a **parent** of another, you must put the more specific (child) exception's `catch` block **first**. If you put the parent class first, it will "catch" everything, including the child type, and the code below it becomes unreachable — Java will actually give you a **compile error** for this exact mistake.

```java
try {
    // some code
} catch (Exception e) {           // this is a very general, parent-level catch
    System.out.println("General error");
} catch (ArithmeticException e) { // COMPILE ERROR — unreachable code!
    System.out.println("Math error");
}
```
The fix is to always put more specific exceptions first, general ones last:
```java
try {
    // some code
} catch (ArithmeticException e) {  // specific — checked first
    System.out.println("Math error");
} catch (Exception e) {            // general — checked last, catches anything else
    System.out.println("General error");
}
```

## 3.3 Multi-catch — catching several exception types in one block (Java 7+)

If you want to handle two or more different exceptions in **exactly the same way**, you don't need to repeat the same code in separate catch blocks. You can combine them using `|` (a pipe symbol):

```java
try {
    // some code
} catch (ArithmeticException | ArrayIndexOutOfBoundsException e) {
    System.out.println("Something went wrong: " + e.getMessage());
}
```

This avoids repeating the same handling code twice. One rule: the exception types combined with `|` must not have a parent-child relationship with each other (you can't combine a class with its own parent class this way).

## 3.4 The `finally` block

### What it is
A block of code that **always runs**, no matter what happens in the `try` block — whether an exception was thrown or not, and even if the exception was never caught at all.

```java
try {
    System.out.println("Trying...");
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Caught the error");
} finally {
    System.out.println("This always runs, no matter what");
}
```
Output:
```
Trying...
Caught the error
This always runs, no matter what
```

### Why does `finally` exist? What's it actually for?
It's meant for **cleanup work** that must happen regardless of success or failure — like closing a file, closing a database connection, or releasing a lock. Without `finally`, you'd have to write that same cleanup code in multiple places (once for the success path, and again inside every catch block) — easy to forget in one of those places.

```java
FileReader file = null;
try {
    file = new FileReader("data.txt");
    // read the file
} catch (IOException e) {
    System.out.println("Could not read file");
} finally {
    if (file != null) {
        try {
            file.close(); // ALWAYS close the file, whether reading succeeded or failed
        } catch (IOException e) {
            System.out.println("Could not close file");
        }
    }
}
```
(Later in this document, you'll see `try-with-resources`, a much cleaner way to do exactly this.)

### Does `finally` run even if there's a `return` statement inside `try` or `catch`?
Yes — this surprises a lot of learners. Even if the `try` or `catch` block has a `return` statement, `finally` still runs **before** the method actually returns.

```java
public static int test() {
    try {
        return 1;
    } finally {
        System.out.println("Finally still runs, even with a return above it!");
    }
}
// Calling test() prints "Finally still runs..." AND THEN returns 1
```

### The dangerous trap: overriding a return value inside `finally`
This is a classic trick question. If `finally` **also** has a `return` statement, it will **override** whatever the `try` or `catch` block was about to return — this is confusing and considered bad practice.

```java
public static int badExample() {
    try {
        return 1;
    } finally {
        return 2; // this OVERRIDES the "return 1" above — the method actually returns 2!
    }
}
```
**Rule to remember**: never put a `return` statement inside `finally` — it silently throws away whatever result the `try`/`catch` was going to return, and can hide real bugs.

### Does `finally` run if the JVM shuts down forcefully?
No — there's one specific case where `finally` does NOT run: if the program calls `System.exit()` inside the `try` block, or if the JVM crashes/is killed, `finally` is skipped entirely, because the whole program has already stopped.

### Common interview angle
**Q: If both `try` and `finally` have return statements, which one wins?**
The `finally` block's return always wins, silently overriding the try/catch block's return — which is exactly why writing `return` inside `finally` is considered a bad practice to avoid.

## 3.5 Is `catch` or `finally` required after `try`?

No, but you must have **at least one of them**. Valid combinations:
```java
try { } catch (Exception e) { }              // valid
try { } finally { }                           // valid — no catch, just cleanup
try { } catch (Exception e) { } finally { }   // valid — both
try { }                                       // INVALID — try alone, with nothing after it, doesn't compile
```

---

# PART 4: `throw` VS `throws` — A COMMON CONFUSION, CLEARED UP

These two keywords look almost identical but do completely different things. Let's separate them clearly.

## 4.1 `throw` — actually creating and raising an exception

`throw` is a statement you use **inside** a method's body, to actually create an exception and make it happen, right at that moment.

```java
public void checkAge(int age) {
    if (age < 18) {
        throw new IllegalArgumentException("Age must be 18 or older"); // ACTUALLY throwing an exception here
    }
    System.out.println("Age is valid");
}
```
You always use `throw` with exactly ONE exception object, and it immediately stops the normal flow of the method right there, just like any other exception being thrown.

## 4.2 `throws` — a warning label on a method

`throws` is used in a method's **signature** (the line with the method name and parameters), to declare that this method **might** throw a certain checked exception, without necessarily throwing it directly itself — often because it calls other code that might throw it.

```java
public void readFile(String path) throws IOException { // WARNING LABEL: "calling this method might cause an IOException"
    FileReader file = new FileReader(path); // this line is what could actually cause the IOException
}
```
This is purely a heads-up to whoever calls this method: "be aware, you need to either catch this exception, or pass the responsibility further up yourself."

## 4.3 Side-by-side comparison

| | `throw` | `throws` |
|---|---|---|
| Used where? | Inside a method's body | In a method's signature (declaration line) |
| What it does | Actually creates and raises ONE specific exception, right now | Declares that a method MIGHT throw certain exception type(s) |
| How many exceptions? | Exactly one, at that specific line | Can list several, separated by commas |
| Example | `throw new IOException("File missing");` | `public void readFile() throws IOException, SQLException` |

### Common interview angle
**Q: Can you use `throws` for unchecked exceptions too?**
Yes, technically you can write `throws NullPointerException` on a method, but it's not required and not common practice, since the compiler doesn't force it for unchecked exceptions — it's really only meaningful (and required) for checked exceptions.

---

# PART 5: HANDLING EXCEPTIONS THAT YOU CALL FROM OTHER METHODS

## 5.1 What happens if a method doesn't catch an exception it might cause?

If a method's code can cause a checked exception, and that method doesn't catch it itself, it must **declare** that exception with `throws` — passing the responsibility of handling it up to whoever called that method.

```java
public void readFile() throws IOException {   // doesn't handle it — passes responsibility up to its caller
    FileReader file = new FileReader("data.txt");
}

public void process() throws IOException {    // this method ALSO doesn't handle it — passes it up again
    readFile();
}

public static void main(String[] args) {
    try {
        new Demo().process(); // finally, HERE it actually gets handled
    } catch (IOException e) {
        System.out.println("Something went wrong: " + e.getMessage());
    }
}
```

## 5.2 This "passing up" idea is called propagation

**Exception propagation** simply means: if a method doesn't handle an exception itself, the exception keeps moving up the chain of method calls (from the method where it happened, back to whoever called that method, and so on) until either something catches it, or it reaches the very top (the `main` method) with still nobody catching it — in which case the program crashes and prints the stack trace, exactly like the very first example in this document.

### A useful way to picture this
Imagine three people passing a hot potato: Method A calls Method B, which calls Method C. If Method C has a problem and doesn't catch it, the problem gets handed back to Method B. If Method B doesn't catch it either, it gets handed back to Method A. If NOBODY along that chain catches it, the whole program stops.

### Common interview angle
**Q: Does a method have to declare `throws` for an unchecked exception it doesn't catch?**
No — since unchecked exceptions aren't checked by the compiler at all, they can propagate up through any number of method calls without any method along the way needing to write `throws` for them.

---

# PART 6: CUSTOM (USER-DEFINED) EXCEPTIONS

## 6.1 Why would you ever create your own exception class?

Java's built-in exceptions are quite general (`IllegalArgumentException`, `RuntimeException`, etc.). Sometimes you want an exception that describes a problem specific to YOUR application in a clear, meaningful way — like `InsufficientFundsException` in a banking app, which is much clearer to read and handle than a generic `RuntimeException` with just a text message.

## 6.2 How to create a custom checked exception

You create your own exception simply by making a class that **extends** `Exception` (for checked) or `RuntimeException` (for unchecked).

```java
// A custom CHECKED exception — extends Exception
class InsufficientFundsException extends Exception {
    public InsufficientFundsException(String message) {
        super(message); // passes the message to the parent Exception class
    }
}
```

## 6.3 How to create a custom unchecked exception

```java
// A custom UNCHECKED exception — extends RuntimeException instead
class InvalidAccountException extends RuntimeException {
    public InvalidAccountException(String message) {
        super(message);
    }
}
```

## 6.4 Using your custom exception

```java
class BankAccount {
    private double balance = 100.0;

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException("Not enough balance for this withdrawal");
        }
        balance -= amount;
    }
}

public static void main(String[] args) {
    BankAccount account = new BankAccount();
    try {
        account.withdraw(500);
    } catch (InsufficientFundsException e) {
        System.out.println("Error: " + e.getMessage());
    }
}
```

## 6.5 Should you extend `Exception` or `RuntimeException` for your own custom exception?

This is a genuinely common design decision. General guidance:
- Extend **`Exception`** (making it checked) when the problem is something the calling code can reasonably be expected to **recover from**, and you want to force whoever calls your method to consciously think about handling it — like `InsufficientFundsException`, where the calling code should clearly decide what to do (show an error, retry, etc.).
- Extend **`RuntimeException`** (making it unchecked) when the problem usually represents a **programming mistake** or something that shouldn't normally happen if the code calling it is correct — like passing an invalid, clearly-wrong argument to a method.

### Common interview angle
**Q: What's a common criticism of overusing checked custom exceptions?**
If every single method in a codebase declares long lists of custom checked exceptions, calling code ends up littered with `try-catch` blocks or long `throws` lists everywhere, even in places where there's no meaningful way to recover from the problem — many modern codebases prefer using unchecked custom exceptions more often, catching them only at a few centralized places (like a top-level error handler) instead of forcing every single method along the way to deal with them individually.

---

# PART 7: TRY-WITH-RESOURCES (JAVA 7+) — CLEANER RESOURCE CLEANUP

## 7.1 The problem this solves

Earlier in this document (Part 3.4), we saw that properly closing a resource (like a file) needs a `finally` block, and that gets messy — especially once you have more than one resource to close, or the closing itself can throw an exception too.

## 7.2 What try-with-resources looks like

```java
try (FileReader file = new FileReader("data.txt")) {
    // use the file
} catch (IOException e) {
    System.out.println("Something went wrong: " + e.getMessage());
}
// No finally block needed at all — Java automatically closes "file" for you, whether an exception happened or not!
```

### What's actually happening here
Any resource you open inside the parentheses right after `try` is **automatically closed** once the `try` block finishes — whether it finished normally, or because an exception was thrown. You don't need to write a `finally` block or manually call `.close()` yourself at all.

## 7.3 What makes something usable in try-with-resources?

Only objects that implement the `AutoCloseable` interface (which has one method: `close()`) can be used this way. Most classes that represent a resource you need to release — files, database connections, network sockets — already implement this interface in the standard library.

## 7.4 Using multiple resources at once

```java
try (FileReader file1 = new FileReader("input.txt");
     FileWriter file2 = new FileWriter("output.txt")) {
    // use both resources
} catch (IOException e) {
    System.out.println("Error: " + e.getMessage());
}
// Both file1 and file2 are automatically closed, in the REVERSE order they were opened
```

## 7.5 Why is try-with-resources considered so much better than manually using finally?

1. **Less code** — you don't need to write the manual `if (resource != null) { resource.close(); }` boilerplate every single time.
2. **Safer** — it's very easy to accidentally forget to close a resource, or to close it incorrectly, when doing it manually; try-with-resources removes that risk entirely.
3. **Better exception handling** — if closing the resource itself throws an exception (which can genuinely happen), try-with-resources handles this gracefully using something called "suppressed exceptions" (explained next), instead of accidentally hiding your original, more important exception.

## 7.6 What are "suppressed exceptions"?

Imagine this rare but real situation: your code inside `try` throws an exception, AND while Java is trying to automatically close your resource afterward, closing it *also* throws an exception. Which one do you actually see?

With try-with-resources, Java keeps the **original** exception (from inside the try block) as the main one you see, and attaches the second exception (from closing the resource) as a "suppressed" exception alongside it — meaning you don't lose either piece of information, even though only one exception is the "primary" one shown.

```java
try {
    Throwable[] suppressed = e.getSuppressed(); // you can retrieve any suppressed exceptions this way, if needed
} catch (Exception e) { }
```

### Common interview angle
**Q: What happens to a resource-closing exception if you were using an old-style manual `finally` block instead of try-with-resources?**
With manual `finally`, if `close()` throws an exception, it would actually **replace** the original exception from the try block — meaning you'd lose all information about what the ORIGINAL problem was, and only see the closing error instead. Try-with-resources fixes this by keeping the original exception as primary and attaching the closing exception as a suppressed one instead.

---

# PART 8: IMPORTANT DETAILS AND RULES ABOUT EXCEPTION OBJECTS

## 8.1 What information does an exception object actually hold?

Every exception object (since they all descend from `Throwable`) carries:
- A **message** — a short text description of what went wrong, accessible via `e.getMessage()`.
- A **stack trace** — the list of method calls that were happening when the exception was created, accessible via `e.printStackTrace()` or `e.getStackTrace()`.
- Optionally, a **cause** — another exception that was the underlying reason this one happened (covered next, in exception chaining).

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println(e.getMessage());     // "/ by zero"
    e.printStackTrace();                     // prints the full stack trace to the console
}
```

## 8.2 Exception chaining — keeping track of the "real" original cause

### The problem this solves
Sometimes you catch one exception, but you want to throw a different, more meaningful exception instead — without losing information about what the ORIGINAL underlying problem actually was.

```java
public void loadConfig() throws ConfigException {
    try {
        FileReader file = new FileReader("config.txt");
    } catch (IOException e) {
        // Throw a more meaningful custom exception, but KEEP the original exception attached as the "cause"
        throw new ConfigException("Could not load configuration", e);
    }
}
```

Here, `e` (the original `IOException`) becomes the **cause** of the new `ConfigException`. Later, if someone looks at this new exception, they can still find out the original underlying reason:

```java
try {
    loadConfig();
} catch (ConfigException e) {
    System.out.println("Message: " + e.getMessage());
    System.out.println("Original cause: " + e.getCause()); // gets back the original IOException
}
```

### Why this matters in real applications
Without chaining, you'd have to choose between showing a clear, meaningful high-level error message (losing the technical details of what actually broke) OR showing the raw technical error (which might be confusing to whoever's reading it, without a clear explanation of context). Chaining lets you have both — a clear message at the top, with the full technical history still available underneath if you need to dig deeper.

## 8.3 Rethrowing an exception

Sometimes you want to catch an exception just to do something (like logging it), and then let it continue propagating up as if you never caught it at all.

```java
public void process() throws IOException {
    try {
        readFile();
    } catch (IOException e) {
        System.out.println("Logging the error: " + e.getMessage());
        throw e; // re-throw the SAME exception, so it keeps propagating up to whoever called process()
    }
}
```

## 8.4 Custom exceptions with extra information

Your custom exception class isn't limited to just a message — you can add extra fields to carry more useful, specific information about the problem.

```java
class InsufficientFundsException extends Exception {
    private double shortfallAmount;

    public InsufficientFundsException(String message, double shortfallAmount) {
        super(message);
        this.shortfallAmount = shortfallAmount;
    }

    public double getShortfallAmount() {
        return shortfallAmount;
    }
}

// Usage:
catch (InsufficientFundsException e) {
    System.out.println("You are short by: " + e.getShortfallAmount());
}
```

---

# PART 9: COMMON JAVA EXCEPTIONS YOU SHOULD KNOW BY NAME

## 9.1 Common unchecked exceptions

**`NullPointerException`** — happens when you try to use an object reference that is actually `null`, like calling a method on it.
```java
String name = null;
System.out.println(name.length()); // NullPointerException
```

**`ArithmeticException`** — happens with an invalid math operation, most commonly dividing an integer by zero.
```java
int result = 10 / 0; // ArithmeticException
```

**`ArrayIndexOutOfBoundsException`** — happens when you try to access an array position that doesn't exist (either negative, or beyond the array's actual size).
```java
int[] arr = {1, 2, 3};
System.out.println(arr[5]); // ArrayIndexOutOfBoundsException
```

**`ClassCastException`** — happens when you try to force-convert (cast) an object into a type it isn't actually compatible with.
```java
Object obj = "Hello";
Integer number = (Integer) obj; // ClassCastException — obj is really a String, not an Integer
```

**`NumberFormatException`** — happens when you try to convert text into a number, but the text isn't actually a valid number.
```java
int number = Integer.parseInt("abc"); // NumberFormatException
```

**`IllegalArgumentException`** — happens when a method receives an argument that's simply not valid for what it needs.
```java
public void setAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("Age cannot be negative");
    }
}
```

**`IllegalStateException`** — happens when a method is called at a time when the object isn't in the right condition/state for that call to make sense.

**`UnsupportedOperationException`** — happens when you try to call a method that this particular object simply doesn't support, often seen with immutable collections.
```java
List<String> list = List.of("a", "b");
list.add("c"); // UnsupportedOperationException — this list is immutable
```

## 9.2 Common checked exceptions

**`IOException`** — a general problem related to input/output, like reading or writing files, that failed for some reason.

**`FileNotFoundException`** — a more specific type of `IOException`, happening when you try to open a file that doesn't exist at the given path.

**`SQLException`** — happens when something goes wrong while working with a database (bad query, lost connection, etc.).

**`ClassNotFoundException`** — happens when your code tries to load a class by name at runtime, but that class can't actually be found.

**`InterruptedException`** — happens when a thread that is sleeping or waiting gets interrupted by another thread before it was done waiting.

---

# PART 10: BEST PRACTICES — WRITING GOOD EXCEPTION-HANDLING CODE

## 10.1 Never leave a catch block completely empty

```java
// BAD — swallows the error silently, you'll never know something went wrong!
try {
    riskyOperation();
} catch (Exception e) {
    // nothing here — the problem just disappears silently
}
```
This is one of the worst things you can do in real code — the program keeps running as if nothing happened, but something actually DID go wrong, and now nobody will ever know. At the very minimum, log the exception:
```java
try {
    riskyOperation();
} catch (Exception e) {
    logger.error("Something went wrong during riskyOperation", e);
}
```

## 10.2 Don't catch `Exception` (or `Throwable`) too broadly, unless you really mean to

```java
// Usually too broad — catches EVERYTHING, including bugs you didn't expect, hiding real problems
try {
    doSomething();
} catch (Exception e) {
    System.out.println("Something went wrong");
}
```
Catching overly broad exception types can accidentally swallow bugs that have nothing to do with what you were trying to handle — making it much harder to find and fix real problems later. Catch the **specific** exception types you actually expect and know how to handle.

## 10.3 Never use exceptions for normal, everyday program logic

Exceptions are meant for genuinely **exceptional**, unexpected situations — not as a substitute for a simple `if` check that you already know how to test for directly.

```java
// BAD — using exceptions for something you could simply check with an "if" statement
try {
    return list.get(5);
} catch (IndexOutOfBoundsException e) {
    return null;
}

// GOOD — just check directly, no exception needed at all
if (index < list.size()) {
    return list.get(index);
} else {
    return null;
}
```
Exceptions are relatively expensive to create and throw (building a stack trace takes real work) — using them for routine, expected logic makes code both slower and harder to read.

## 10.4 Always clean up resources — prefer try-with-resources

As covered in Part 7, always make sure resources like files, database connections, and network sockets get closed, no matter what happens. Prefer try-with-resources over manually writing `finally` blocks whenever possible, since it's shorter and safer.

## 10.5 Provide clear, useful exception messages

```java
// BAD — unhelpful, doesn't say what actually went wrong or with what value
throw new IllegalArgumentException("Invalid input");

// GOOD — specific and useful, tells you exactly what was wrong
throw new IllegalArgumentException("Age must be between 0 and 120, but got: " + age);
```
A good message saves a huge amount of debugging time later — imagine trying to fix a production bug at 2 AM with only "Invalid input" to go on, versus a message that tells you exactly what value caused the problem.

## 10.6 Don't use exceptions to control normal program flow between methods

Similar to 10.3, but specifically about using exceptions as a "shortcut" way to jump between methods or exit loops, instead of using normal return values or simple flags. This makes code confusing to follow, since exceptions are supposed to represent unexpected situations, not a normal part of your program's regular logic path.

## 10.7 Preserve the original cause when wrapping exceptions

As covered in Part 8.2, always pass the original exception as the "cause" when you wrap it in a new, more meaningful exception — never silently discard the original technical details.

```java
// BAD — original exception details are lost forever
catch (IOException e) {
    throw new ConfigException("Could not load configuration");
}

// GOOD — original exception is preserved as the cause
catch (IOException e) {
    throw new ConfigException("Could not load configuration", e);
}
```

## 10.8 Only catch what you can actually do something meaningful about

If you catch an exception but have no real, meaningful way to handle or recover from it at that specific point in the code, it's often better to let it propagate up to a place in your program that DOES know what to do about it (like a centralized error handler), rather than catching it just because you technically can.

---

# PART 11: A FEW MORE ADVANCED / TRICKY DETAILS

## 11.1 Can you catch multiple unrelated exceptions in a hierarchy trick question?

If you write `catch (RuntimeException e)`, this will catch `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException`, and every other unchecked exception too — because they all descend from `RuntimeException`. This is a common thing to test in interviews: understanding that catching a parent class catches all its children too.

## 11.2 What is `Throwable.getCause()` vs `Throwable.getMessage()`?

`getMessage()` returns the short text description of THIS specific exception. `getCause()` returns another exception object — the original, underlying exception that led to this one (if any chaining was set up, as shown in Part 8.2). If there's no chained cause, `getCause()` simply returns `null`.

## 11.3 Can an exception itself throw another exception while being handled?

Yes — if code inside a `catch` block itself causes another problem, a brand-new exception gets thrown from right there, and normal exception behavior applies to it too (it propagates up from that point, potentially replacing or interacting with the exception you were originally handling, depending on exactly how the code is structured).

## 11.4 What is the difference between `e.printStackTrace()` and simply logging `e.getMessage()`?

`e.getMessage()` only gives you the short text description. `e.printStackTrace()` gives you the ENTIRE call chain of methods that were active when the exception happened, which is far more useful for actually finding and fixing the root cause of a bug. In real production applications, you'd typically use a proper logging framework to record the full stack trace, rather than printing it directly to the console.

## 11.5 Does creating an exception object automatically print anything?

No — simply writing `new IOException("problem")` just creates the exception object in memory; it does nothing visible by itself. It only actually interrupts your program's flow once you use the `throw` keyword on it.

```java
IOException e = new IOException("problem"); // just creates the object — nothing happens yet
// ... program continues normally here ...
throw e; // NOW it actually disrupts the flow and needs to be caught somewhere
```

## 11.6 What happens if an exception is thrown inside a `catch` block?

The new exception thrown inside the `catch` block takes over completely — Java stops looking at the rest of that `catch` block, and this new exception now propagates upward exactly like any other exception would, ignoring whatever was originally being handled (unless you specifically used exception chaining, as shown earlier, to keep a reference back to the original one).

---

# PART 12: PUTTING IT ALL TOGETHER — A REALISTIC EXAMPLE

Here's a small, realistic example that uses many of the ideas from this document together, the way you'd actually write this kind of code in a real application:

```java
class InsufficientFundsException extends Exception {
    private final double shortfall;

    public InsufficientFundsException(String message, double shortfall) {
        super(message);
        this.shortfall = shortfall;
    }

    public double getShortfall() {
        return shortfall;
    }
}

class BankAccount {
    private double balance;

    public BankAccount(double balance) {
        this.balance = balance;
    }

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive");
        }
        if (amount > balance) {
            throw new InsufficientFundsException(
                "Cannot withdraw " + amount + ", balance is only " + balance,
                amount - balance
            );
        }
        balance -= amount;
    }
}

public class BankDemo {
    public static void main(String[] args) {
        BankAccount account = new BankAccount(100.0);

        try {
            account.withdraw(500.0);
        } catch (InsufficientFundsException e) {
            System.out.println("Transaction failed: " + e.getMessage());
            System.out.println("You need " + e.getShortfall() + " more to complete this withdrawal.");
        } catch (IllegalArgumentException e) {
            System.out.println("Invalid request: " + e.getMessage());
        } finally {
            System.out.println("Transaction attempt finished.");
        }
    }
}
```
This single example uses: a custom checked exception with extra information, throwing it with a clear message, catching multiple specific exception types in the right order, and a `finally` block for something that should always run regardless of the outcome.

---

# Quick-reference summary table

| Concept | One-line meaning |
|---|---|
| Exception | An unexpected problem that happens while a program is running |
| Error | A serious, usually unrecoverable problem (like running out of memory) |
| Checked exception | Compiler forces you to handle it (extends Exception, not RuntimeException) |
| Unchecked exception | Compiler does NOT force you to handle it (extends RuntimeException) — usually a coding bug |
| try | Wraps code that might cause a problem |
| catch | Handles a specific type of problem if it happens |
| finally | Runs always, no matter what — used for cleanup |
| throw | Actually creates and raises one exception, right now |
| throws | A label on a method saying "I might cause this exception" |
| Propagation | An unhandled exception moving up through the chain of method calls |
| Custom exception | Your own exception class, made for a specific problem in your application |
| try-with-resources | Automatically closes a resource for you, no finally needed |
| Suppressed exception | A second problem that happened while closing a resource, kept alongside the main one |
| Exception chaining | Keeping the original exception as the "cause" of a new, more meaningful one |
| Stack trace | The list of method calls active when the exception happened — great for debugging |

---

## Suggested study order

1. **Part 1 and 2** (what an exception is, the hierarchy, checked vs unchecked) — the absolute foundation; the checked-vs-unchecked distinction gets asked in almost every interview.
2. **Part 3** (try/catch/finally, multi-catch, the finally-return trap) — extremely commonly tested with tricky little code snippets.
3. **Part 4** (throw vs throws) — a quick topic, but a very common basic interview question.
4. **Part 7** (try-with-resources) — increasingly common to be asked about, since it's now the standard, expected way to handle resources.
5. **Part 6 and 8** (custom exceptions, chaining) — shows you can design real applications, not just handle textbook errors — good to be ready to write this kind of code live in an interview.
6. **Part 9** (common exception names) — good to recognize instantly, without having to think about it.
7. **Part 10** (best practices) — often asked as "what mistakes have you seen in exception handling," so having clear, concrete answers here makes a strong impression.
