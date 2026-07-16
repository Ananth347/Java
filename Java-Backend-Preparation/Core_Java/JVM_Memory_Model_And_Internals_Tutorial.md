# JVM (Java Virtual Machine) — Complete Tutorial: Memory Model & Internals

> Everything the JVM does, explained in the simplest English, with diagrams described in words and analogies.

---

## Table of Contents
1. JVM vs JRE vs JDK (the basics first)
2. What is JVM and why does Java need it?
3. Overall JVM Architecture (Big Picture)
4. Class Loader Subsystem
5. Runtime Data Areas (THE JVM MEMORY MODEL — most important section)
   - Method Area (Metaspace)
   - Heap
   - Java Stack
   - PC (Program Counter) Register
   - Native Method Stack
6. Heap Deep Dive (Young Gen, Old Gen)
7. Garbage Collection (GC) — how JVM cleans memory
8. Types of Garbage Collectors
9. Execution Engine (Interpreter, JIT Compiler, Adaptive Optimizer)
10. Java Memory Model (JMM) — for multithreading (happens-before, volatile, synchronized)
11. String Pool (String Constant Pool)
12. Types of References (Strong, Weak, Soft, Phantom)
13. Common Memory Problems (Leaks, OutOfMemoryError, StackOverflowError)
14. Tools to Monitor/Analyze JVM Memory
15. Summary Cheat Sheet

---

## 1. JVM vs JRE vs JDK (the basics first)

Before diving deep, let's clear up a very common confusion.

| Term | Full form | What it is |
|------|-----------|------------|
| **JVM** | Java Virtual Machine | The engine that actually RUNS your compiled Java code (`.class` files). It's a specification + implementation. |
| **JRE** | Java Runtime Environment | JVM + Core Libraries (like `java.lang`, `java.util`). Enough to RUN Java programs, but not to develop them. |
| **JDK** | Java Development Kit | JRE + Development tools (compiler `javac`, debugger, etc). Needed to WRITE and COMPILE Java programs. |

**Simple analogy:**
- JVM = The car engine (what actually makes things run)
- JRE = The complete car (engine + all parts needed to drive)
- JDK = The car + the full workshop/toolkit needed to build and repair cars

**Relationship:** `JDK ⊃ JRE ⊃ JVM` (JDK contains JRE, JRE contains JVM)

---

## 2. What is JVM and why does Java need it?

The **JVM (Java Virtual Machine)** is a program (an abstract computing machine) that provides a runtime environment to execute Java bytecode.

### The Java Compilation Process (Step by Step):

```
YourFile.java  --(javac compiler)-->  YourFile.class (bytecode)  --(JVM)-->  Machine Code (runs on OS/CPU)
```

1. You write code in `.java` file
2. `javac` (the Java compiler) converts it into **bytecode** (`.class` file) — this is NOT machine code, it's an intermediate, platform-independent format
3. The **JVM** reads this bytecode and converts it into actual machine code that your specific computer's CPU can understand and execute

### Why does this design make Java "Write Once, Run Anywhere" (WORA)?

Because the `.class` bytecode file is the SAME regardless of whether you're on Windows, Linux, or Mac. Only the **JVM** is different for each OS (you install a Windows JVM, a Linux JVM, a Mac JVM). The JVM acts as a **translator** between your universal bytecode and the OS-specific machine code.

**Analogy:** Think of bytecode as a book written in a universal language. The JVM is a translator who can read that book and explain it in the local language of whichever country (operating system) you're in. You write the book once; different translators (JVMs) exist for different countries (OS).

### What does JVM actually DO? (High-level responsibilities)
1. **Loads** the code (Class Loader)
2. **Verifies** the code (Bytecode Verifier — checks for security/correctness)
3. **Executes** the code (Execution Engine)
4. **Manages memory** automatically (Heap, Stack, Garbage Collection)
5. Provides **runtime environment** (security, exception handling, multithreading support)

---

## 3. Overall JVM Architecture (Big Picture)

The JVM has 3 main subsystems that work together:

```
┌─────────────────────────────────────────────────────────┐
│                         JVM                                │
│                                                              │
│  1. CLASS LOADER SUBSYSTEM                                  │
│     (Loads .class files into memory)                        │
│                    │                                         │
│                    ▼                                         │
│  2. RUNTIME DATA AREAS (Memory Areas)                        │
│     - Method Area (Metaspace)                                │
│     - Heap                                                    │
│     - Stack (per thread)                                      │
│     - PC Register (per thread)                                │
│     - Native Method Stack (per thread)                        │
│                    │                                         │
│                    ▼                                         │
│  3. EXECUTION ENGINE                                          │
│     - Interpreter                                              │
│     - JIT Compiler                                             │
│     - Garbage Collector                                        │
│                                                              │
└─────────────────────────────────────────────────────────┘
```

Let's go through each of these subsystems one by one, in detail.

---

## 4. Class Loader Subsystem

The Class Loader's job is to **load `.class` files into memory** so the JVM can use them. This happens in 3 phases: **Loading → Linking → Initialization**.

### Phase 1: Loading
The `.class` file is read and a corresponding `Class` object is created in memory (in the Method Area).

There are 3 built-in class loaders, working in a **hierarchy (parent-child delegation model)**:

| Class Loader | Loads what? |
|--------------|-------------|
| **Bootstrap Class Loader** | Loads core Java classes from `rt.jar`/core modules (e.g., `java.lang.*`, `java.util.*`). Written in native code (C/C++), not Java. This is the topmost parent. |
| **Extension/Platform Class Loader** | Loads classes from the extensions directory (`java.sql`, etc. in older Java; now part of platform modules) |
| **Application/System Class Loader** | Loads classes from your application's classpath (your own `.class` files, third-party JARs) |

**Delegation Model:** When a class needs to be loaded, the request first goes to the Bootstrap loader. If it can't find the class, it delegates down to the Extension loader, and finally to the Application loader. This is called **parent delegation** — it ensures core Java classes are always loaded by the trusted Bootstrap loader first (prevents someone from creating a fake `java.lang.String` class, for security).

### Phase 2: Linking
Linking has 3 sub-steps:

1. **Verification** — The Bytecode Verifier checks that the `.class` file is valid, properly formatted, and doesn't violate Java's security rules (e.g., doesn't cause illegal memory access). If verification fails, you get a `VerifyError`.
2. **Preparation** — Memory is allocated for static variables, and they are assigned **default values** (like `0`, `null`, `false`) — NOT the actual values yet.
3. **Resolution** — Symbolic references (like class/method/field names as text) are replaced with actual direct references (memory addresses/pointers).

### Phase 3: Initialization
Now the actual assigned values run. Static variables get their real values, and **static blocks** are executed, in the order they appear in the code.

```java
class Demo {
    static int x = 10;      // Preparation: x=0 first, then Initialization: x=10
    static {
        System.out.println("Static block runs during Initialization");
    }
}
```

**Important:** A class is loaded **lazily** — meaning it's only loaded when it is first actively used (like creating an object, calling a static method, or accessing a static variable), not when the program starts.

---

## 5. Runtime Data Areas (THE JVM MEMORY MODEL)

This is the **most important section** — it explains where JVM actually stores everything while your program runs. There are 5 main memory areas.

Some areas are **shared across all threads**, and some are **per-thread** (each thread gets its own copy).

| Memory Area | Shared or Per-Thread? |
|-------------|:---:|
| Method Area (Metaspace) | Shared (all threads) |
| Heap | Shared (all threads) |
| Java Stack | Per-Thread |
| PC Register | Per-Thread |
| Native Method Stack | Per-Thread |

---

### 5.1 Method Area (also called Metaspace in Java 8+)

**What it stores:**
- Class-level information: class name, parent class name, methods, constructors, field information
- `static` variables
- Constant pool (more on this below)
- Method bytecode itself

**Important history:** Before Java 8, this area was called **PermGen (Permanent Generation)** and had a fixed maximum size, causing frequent `OutOfMemoryError: PermGen space` errors. From **Java 8 onwards**, PermGen was removed and replaced with **Metaspace**, which lives in **native (OS) memory**, not the JVM heap, and can grow dynamically (limited by your computer's available RAM, unless you explicitly set `-XX:MaxMetaspaceSize`).

**Analogy:** Think of Method Area as a "class dictionary" — it stores the blueprint/definition information about every class, shared by everyone who needs it.

---

### 5.2 Heap

**This is the MOST important memory area** — and the one Garbage Collection focuses on.

**What it stores:**
- ALL **objects** (created using `new`)
- Instance variables (non-static fields) of those objects
- Arrays

**Key facts:**
- Shared among ALL threads (so it needs thread-safety mechanisms)
- This is where **Garbage Collection** happens
- If the heap runs out of space and can't be expanded, you get `java.lang.OutOfMemoryError: Java heap space`

We will explore the Heap's internal structure (Young Gen, Old Gen) in detail in Section 6, since this is critical for understanding Garbage Collection.

---

### 5.3 Java Stack (Thread Stack)

Each **thread** gets its own separate stack when it's created. It stores:
- **Stack Frames** — one frame is created for every method call

**What's inside a Stack Frame:**
1. **Local Variable Array** — stores local variables and method parameters
2. **Operand Stack** — used as workspace to perform intermediate calculations (like a scratchpad for bytecode instructions)
3. **Frame Data** — reference to the constant pool, and info to help return control to the calling method

```java
void methodA() {
    int x = 10;      // stored in methodA's stack frame
    methodB();
}
void methodB() {
    int y = 20;      // stored in methodB's stack frame (a NEW frame, pushed on top)
}
```

When `methodA()` calls `methodB()`, a new frame for `methodB` is **pushed** on top of the stack. When `methodB()` finishes, its frame is **popped off**, and control returns to `methodA`.

**This is why local variables and method calls follow LIFO (Last In, First Out) order — exactly like a stack of plates.**

**Important error:** If you have too many nested/recursive method calls (frames keep getting pushed without being popped), the stack runs out of space, causing `java.lang.StackOverflowError`. This is the classic result of infinite/uncontrolled recursion.

```java
void recurse() {
    recurse();  // no base case - keeps calling itself forever
}
// Eventually throws: StackOverflowError
```

---

### 5.4 PC (Program Counter) Register

Each thread also gets its own **PC Register**. It's a small memory area that stores the address of the **current instruction** being executed by that thread.

Think of it as a **bookmark** — it remembers "which line of bytecode am I currently executing" for each thread, so when the CPU switches between threads, it knows exactly where to resume.

If the method being executed is `native` (written in another language like C, not Java), the PC register value is undefined for that thread.

---

### 5.5 Native Method Stack

This is similar to the Java Stack, but it's used specifically for **native methods** — methods written in other languages (usually C/C++) and called via **JNI (Java Native Interface)**.

Example: Some low-level operations (like certain OS-level file operations or performance-critical libraries) are written in C, and Java calls them through JNI. These calls use the Native Method Stack instead of the regular Java Stack.

---

## 6. Heap Deep Dive (Young Gen, Old Gen)

Since almost all objects live in the Heap, and Garbage Collection happens here, the JVM divides the Heap into **generations**, based on a key observation:

> **"Most objects die young"** — most objects are short-lived (like temporary variables inside a loop), while some objects live for a very long time (like cached data or singleton objects).

This observation is called the **"Weak Generational Hypothesis"**, and it's why the heap is divided like this:

```
┌───────────────────────────────────────────────────────────┐
│                          HEAP                                 │
│                                                                 │
│   ┌─────────────────────────┐   ┌───────────────────────┐    │
│   │      YOUNG GENERATION     │   │    OLD GENERATION       │   │
│   │                            │   │    (Tenured)             │   │
│   │  ┌───────┐ ┌────┐ ┌────┐  │   │                          │   │
│   │  │ Eden  │ │ S0 │ │ S1 │  │   │   Long-lived objects      │   │
│   │  └───────┘ └────┘ └────┘  │   │                          │   │
│   └─────────────────────────┘   └───────────────────────┘    │
│                                                                 │
└───────────────────────────────────────────────────────────┘
```

### Young Generation
This is where ALL new objects are created first. It has 3 parts:
1. **Eden Space** — every new object is created here first
2. **Survivor Space 0 (S0)**
3. **Survivor Space 1 (S1)**

### Old Generation (Tenured Generation)
Objects that have survived many garbage collection cycles in the Young Generation get **promoted (moved)** here. This holds long-lived objects.

### How objects move (the lifecycle of an object in the heap):

1. A new object is created → placed in **Eden**
2. When Eden fills up, a **Minor GC** (small garbage collection) runs:
   - Live (still-referenced) objects are moved from Eden to one Survivor space (say S0)
   - Dead (unreferenced) objects in Eden are simply discarded
3. On the next Minor GC, live objects from Eden AND S0 are moved to S1, and their **"age" counter increases**
4. This ping-pong between S0 and S1 continues. Each time an object survives a GC cycle, its age increases
5. Once an object's age crosses a threshold (default is often 15, configurable via `-XX:MaxTenuringThreshold`), it is **promoted** to the **Old Generation**
6. When the Old Generation fills up, a **Major GC / Full GC** runs (this is slower and more expensive than Minor GC, because it scans more memory)

**Why this design?** Minor GC (on Young Gen) is fast because most objects there are already dead — the GC has very little live data to copy. This makes garbage collection in Java much more efficient than scanning the entire heap every time.

**Analogy:** Think of Eden as a "waiting room" for new employees (objects) in a company. Most interns (short-lived objects) leave quickly (get garbage collected). The few interns who stay long enough (survive multiple review cycles) eventually become permanent employees and move to a stable, senior department (Old Generation).

---

## 7. Garbage Collection (GC) — how JVM cleans memory

**Garbage Collection** is the process by which JVM automatically finds and removes objects that are no longer reachable/used by the program, freeing up heap memory. This is one of Java's biggest advantages over languages like C/C++, where you must manually free memory (`free()`), risking memory leaks or crashes.

### How does JVM know an object is "garbage" (unreachable)?

An object is considered "garbage" if there is **no live reference path** to it from any **GC Root**.

**GC Roots** are starting points that are always considered reachable/alive, such as:
- Local variables in currently executing methods (on the stack)
- Active threads
- Static variables
- JNI references

The JVM uses an algorithm called **"Mark and Sweep"** (and its variants) to find garbage:

### Mark and Sweep Algorithm
1. **Mark phase:** Starting from GC Roots, the JVM traverses (walks through) all reachable objects and "marks" them as alive
2. **Sweep phase:** Any object NOT marked (not reachable) is considered garbage and its memory is reclaimed

Many modern collectors also have a **Compact** phase — after sweeping, remaining live objects are moved together to eliminate gaps (fragmentation) in memory, making future allocation faster.

```java
public class Main {
    public static void main(String[] args) {
        Student s1 = new Student();  // s1 is a GC Root reference (local variable)
        s1 = null;                    // Now the Student object has NO reference pointing to it
        // This object is now "garbage" - eligible for Garbage Collection
        // (JVM decides WHEN to actually collect it, not immediately)
    }
}
```

**Important:** You CANNOT force garbage collection to happen immediately. Calling `System.gc()` only **suggests** to the JVM that now might be a good time to run GC — the JVM decides on its own when to actually do it.

### Types of GC runs:
- **Minor GC:** Cleans the Young Generation only (fast, frequent)
- **Major GC:** Cleans the Old Generation
- **Full GC:** Cleans the ENTIRE heap (Young + Old + sometimes Metaspace) — this is the slowest and causes the most noticeable "pause" in your application (called a **Stop-The-World** event, where all application threads are paused while GC runs)

---

## 8. Types of Garbage Collectors

Java provides multiple GC algorithms, and you can choose one based on your application's needs (using JVM flags).

| GC Type | Flag to enable | Best for | How it works |
|---------|----------------|----------|----------------|
| **Serial GC** | `-XX:+UseSerialGC` | Small applications, single-threaded environments | Uses a single thread for GC; simple, but causes long Stop-The-World pauses |
| **Parallel GC** (Throughput Collector) | `-XX:+UseParallelGC` | Applications that prioritize high throughput over pause time | Uses multiple threads for GC, still has Stop-The-World pauses, but faster overall |
| **CMS (Concurrent Mark Sweep)** | `-XX:+UseConcMarkSweepGC` | Applications needing lower pause times (Deprecated since Java 9, removed in Java 14) | Does most work concurrently WITH the application threads, minimizing pauses |
| **G1 GC (Garbage First)** | `-XX:+UseG1GC` | Default since Java 9; large heaps, balanced throughput & low pause time | Divides heap into small equal-sized regions; collects regions with the most garbage first ("garbage first") |
| **ZGC** | `-XX:+UseZGC` | Very large heaps (multi-GB to TB), ultra-low pause time (sub-millisecond) needed | Uses colored pointers and load barriers to do almost all work concurrently |
| **Shenandoah** | `-XX:+UseShenandoahGC` | Similar to ZGC — low pause time regardless of heap size | Concurrent compaction, developed by Red Hat |

**Simple summary:**
- Need low latency (fast response, minimal pauses)? → G1, ZGC, or Shenandoah
- Need maximum throughput and don't mind occasional pauses? → Parallel GC
- Small/simple app? → Serial GC (default for very small heaps)
- **Since Java 9, G1 GC is the default collector** for most applications.

---

## 9. Execution Engine (Interpreter, JIT Compiler, Adaptive Optimizer)

Once classes are loaded and memory areas are set up, the **Execution Engine** actually runs the bytecode. It has 3 components:

### 9.1 Interpreter
Reads bytecode line-by-line and executes it directly. 

**Problem:** If a method is called 10,000 times (like inside a loop), the Interpreter re-interprets the SAME bytecode 10,000 times — this is slow.

### 9.2 JIT Compiler (Just-In-Time Compiler)
To solve the Interpreter's slowness, the JIT Compiler identifies **"hot" code** (code that runs frequently — like methods called many times, or loops that run many times) and compiles it **directly into native machine code**, so it doesn't need to be interpreted again and again. This compiled native code is cached and reused.

**This is a huge performance boost** — it's why Java, despite being "interpreted", can perform close to compiled languages like C++ for long-running applications.

**JIT Compiler has sub-components:**
- **Profiler** — monitors which methods are called frequently ("hot spots") — this is actually where the name "**HotSpot JVM**" (Oracle's main JVM implementation) comes from!
- **C1 Compiler (Client Compiler)** — compiles quickly but does basic optimization; good for startup speed
- **C2 Compiler (Server Compiler)** — takes longer to compile but produces highly optimized code; used for long-running server applications

Modern JVMs use **Tiered Compilation** — starting with the Interpreter, moving "warm" code to C1, and the "hottest" code eventually to C2, for the best balance of startup speed and peak performance.

### 9.3 Garbage Collector
Already covered in detail above — it's technically also considered part of the Execution Engine's responsibilities, since it works alongside the running application.

---

## 10. Java Memory Model (JMM) — for Multithreading

This is a DIFFERENT concept from "JVM memory areas" (Heap, Stack, etc. covered above) — though people often confuse the two names!

The **Java Memory Model (JMM)** defines the rules for HOW THREADS INTERACT with memory — specifically, when a change made by one thread becomes VISIBLE to another thread, and in what ORDER operations appear to happen. This matters because of a tricky detail: for performance, both the CPU and the JVM are allowed to **reorder instructions** and use **caches**, as long as the result looks correct for a single thread. But this can cause serious bugs in multi-threaded programs.

### The Problem JMM solves

```java
class SharedData {
    boolean flag = false;
    int value = 0;
}

// Thread 1:
data.value = 42;
data.flag = true;

// Thread 2:
if (data.flag == true) {
    System.out.println(data.value);   // might print 0, NOT 42! (without proper synchronization)
}
```

Why could this happen? Because:
1. Each thread might have its OWN CPU cache, and changes made by Thread 1 might not be immediately visible to Thread 2 (visibility problem)
2. The compiler/CPU might reorder `data.value = 42` and `data.flag = true` for optimization, since they seem unrelated (reordering problem)

### Key concepts in JMM:

#### a) `volatile` keyword
Guarantees:
1. **Visibility** — any write to a `volatile` variable is immediately visible to all other threads (it's always read from/written to main memory, not a thread's local cache)
2. **Prevents reordering** — the compiler/CPU cannot reorder instructions around a `volatile` variable

```java
class SharedData {
    volatile boolean flag = false;   // now safe across threads
}
```

**Note:** `volatile` guarantees visibility and ordering, but NOT atomicity for compound operations (like `count++`, which is actually 3 steps: read, increment, write). For atomicity, you need `synchronized` or classes like `AtomicInteger`.

#### b) `synchronized` keyword
Ensures:
1. **Mutual exclusion** — only ONE thread can execute a synchronized block/method on the same object at a time (prevents race conditions)
2. **Visibility** — changes made inside a synchronized block are guaranteed to be visible to other threads that later synchronize on the same lock

```java
class Counter {
    private int count = 0;
    public synchronized void increment() {   // only one thread at a time
        count++;
    }
}
```

#### c) "Happens-Before" Relationship
This is the CORE rule of JMM. It defines an ordering guarantee: if action A "happens-before" action B, then the results of A are guaranteed to be visible to B.

**Some happens-before rules:**
- Each action in a thread happens-before every subsequent action in the SAME thread (program order)
- An unlock on a monitor (`synchronized` block exit) happens-before every subsequent lock on that SAME monitor
- A write to a `volatile` variable happens-before every subsequent read of that SAME variable
- A thread calling `Thread.start()` happens-before any action in the started thread
- All actions in a thread happen-before another thread successfully returns from a `Thread.join()` on that thread

If there's no happens-before relationship between two operations on different threads, the JVM is free to reorder them, and you can get unpredictable results — this is called a **race condition**.

**Simple takeaway:** Whenever multiple threads read/write shared data, use `synchronized`, `volatile`, or classes from `java.util.concurrent` (like `AtomicInteger`, `ConcurrentHashMap`, `ReentrantLock`) to establish proper happens-before relationships and avoid subtle, hard-to-debug bugs.

---

## 11. String Pool (String Constant Pool)

Strings are used SO often in Java that JVM has special optimization for them, stored in an area called the **String Constant Pool** (part of the Heap, since Java 7 onwards — before that it was in Method Area/PermGen).

```java
String s1 = "Hello";          // goes into String Pool
String s2 = "Hello";          // reuses the SAME object from String Pool (no new object created!)
String s3 = new String("Hello");  // forces creation of a NEW object in regular Heap (NOT pool)

System.out.println(s1 == s2);   // true (same reference, from pool)
System.out.println(s1 == s3);   // false (different objects in memory)
System.out.println(s1.equals(s3));  // true (same content)
```

**Why does this exist?** Since `String` is **immutable** in Java (once created, its value can never change), it's SAFE to reuse the same String object wherever the same literal value is used — this saves a LOT of memory, since Strings are so commonly used.

**How it works:** When you create a String using a literal (`"Hello"`), JVM checks the pool first. If an identical String already exists there, it reuses that reference instead of creating a new object. If you use `new String("Hello")`, you explicitly bypass the pool and force a new object creation (though you can manually add it to the pool later using `.intern()`).

---

## 12. Types of References (Strong, Weak, Soft, Phantom)

Normally, when we create an object, we hold a **Strong Reference** to it. But Java provides special reference types (in `java.lang.ref` package) to give the Garbage Collector hints about how "important" an object is.

| Reference Type | Garbage Collected when? | Typical Use Case |
|----------------|--------------------------|-------------------|
| **Strong Reference** | Never, as long as the reference exists (default type: `Object obj = new Object();`) | Normal everyday object usage |
| **Soft Reference** | Only when JVM is running LOW on memory (about to throw OutOfMemoryError) | Memory-sensitive caches (keep data as long as possible, but free it if memory gets tight) |
| **Weak Reference** | Collected at the VERY NEXT garbage collection cycle, even if memory is not low | `WeakHashMap`, avoiding memory leaks in caches/listeners |
| **Phantom Reference** | Object is already finalized; used to get notified AFTER an object is actually removed from memory | Cleanup actions, replacing the deprecated `finalize()` method |

```java
// Strong reference (default)
Object obj = new Object();

// Soft reference
SoftReference<Object> softRef = new SoftReference<>(new Object());

// Weak reference
WeakReference<Object> weakRef = new WeakReference<>(new Object());
```

**Simple analogy:**
- Strong reference = "I need this, don't ever throw it away"
- Soft reference = "Keep this as long as you have space, but if you're really running out of room, it's okay to discard it"
- Weak reference = "I don't really need this to persist — throw it away at your very next cleanup"
- Phantom reference = "Just tell me exactly when this is finally gone"

---

## 13. Common Memory Problems

### a) `java.lang.OutOfMemoryError: Java heap space`
The Heap is full and GC couldn't free enough space. Usually caused by:
- Creating too many objects and holding references to them unnecessarily (memory leak)
- Heap size configured too small for the application's actual needs (`-Xmx` flag too low)

### b) `java.lang.StackOverflowError`
The thread stack ran out of space — usually caused by deep or infinite recursion (a method calling itself without a proper base/stopping condition).

### c) `java.lang.OutOfMemoryError: Metaspace`
Too many classes loaded (common in applications that dynamically generate classes, like some frameworks using heavy reflection/proxies), and Metaspace ran out of native memory.

### d) Memory Leak in Java (yes, it can still happen despite Garbage Collection!)
A memory leak happens when objects are no longer NEEDED by the application, but they are still **reachable** (something still holds a reference to them), so GC cannot collect them.

**Common causes:**
- Static collections (like a `static List`) that keep growing and are never cleared
- Unclosed resources (database connections, file streams, etc.)
- Listener/callback objects that are registered but never unregistered
- Inner classes holding an implicit reference to their outer class, kept alive longer than needed
- Improper caching (not using Soft/Weak references when appropriate)

```java
class LeakExample {
    static List<Object> cache = new ArrayList<>();  // static - lives for the whole application

    void addToCache(Object obj) {
        cache.add(obj);   // keeps growing forever, objects never removed = memory leak
    }
}
```

### JVM Memory Configuration flags (useful to know):
| Flag | Meaning |
|------|---------|
| `-Xms` | Initial heap size |
| `-Xmx` | Maximum heap size |
| `-Xss` | Thread stack size |
| `-XX:MaxMetaspaceSize` | Maximum Metaspace size |
| `-XX:+UseG1GC` | Use G1 Garbage Collector |

Example: `java -Xms256m -Xmx1024m -XX:+UseG1GC MyApp`

---

## 14. Tools to Monitor/Analyze JVM Memory

| Tool | Purpose |
|------|---------|
| **jps** | Lists running Java processes and their process IDs |
| **jstat** | Shows real-time statistics about heap/GC activity |
| **jmap** | Generates heap dumps (a snapshot of all objects in memory) for analysis |
| **jstack** | Generates a thread dump (shows what every thread is doing, useful for deadlock detection) |
| **jconsole** | GUI tool for monitoring memory, threads, and CPU usage of a running JVM |
| **VisualVM** | More advanced GUI tool — profiling, heap dump analysis, memory leak detection |
| **Eclipse MAT (Memory Analyzer Tool)** | Deep heap dump analysis — great for finding exactly what's causing a memory leak |

---

## 15. Summary Cheat Sheet

| Concept | One-line meaning |
|---------|-------------------|
| JVM | Runs compiled Java bytecode; makes Java platform-independent |
| Class Loader | Loads, links, and initializes `.class` files (Bootstrap → Extension → Application) |
| Method Area / Metaspace | Stores class metadata, static variables, constant pool (native memory since Java 8) |
| Heap | Stores all objects; shared across threads; where GC happens |
| Java Stack | Stores method call frames, local variables; one per thread; LIFO order |
| PC Register | Tracks the currently executing instruction; one per thread |
| Native Method Stack | Used for native (non-Java, e.g., C/C++) method calls |
| Young Generation | Eden + 2 Survivor spaces; new objects created here; Minor GC runs here |
| Old Generation | Long-lived objects promoted from Young Gen; Major/Full GC runs here |
| Garbage Collection | Automatic memory cleanup using Mark-and-Sweep (and Compact) |
| G1 GC | Default modern collector; balances throughput and pause time |
| JIT Compiler | Converts frequently-run ("hot") bytecode into native machine code for speed |
| JMM (Java Memory Model) | Rules for thread visibility/ordering of shared memory (`volatile`, `synchronized`, happens-before) |
| String Pool | Reuses identical String literals to save memory (Strings are immutable) |
| Soft/Weak/Phantom Reference | Special references giving GC hints on how eagerly to collect an object |
| StackOverflowError | Thread stack exhausted (usually infinite/deep recursion) |
| OutOfMemoryError | Heap or Metaspace exhausted |

---

**Congratulations! You now understand the complete JVM — from how it loads your code, to how it stores data in memory, to how it automatically cleans up after itself, and how it safely handles multiple threads.** This is genuinely advanced, professional-level knowledge — most working developers only learn parts of this over years of experience.
