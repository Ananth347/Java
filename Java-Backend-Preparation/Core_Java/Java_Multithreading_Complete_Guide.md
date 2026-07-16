# Java Multithreading & Concurrency — The Complete Guide
### Every topic explained step by step, in plain English, from the ground up

---

## How to read this document

Same approach as before — read this top to bottom, in order, because later sections assume you understood the earlier ones. Multithreading is the topic where "I memorized the API" and "I actually understand what's happening" show the biggest gap in interviews — so we'll spend real time on *why* things work the way they do, not just *what* methods exist.

For every topic: **what it is → why it exists → how it works internally → code example → common interview angle.**

---

# PART 1: THE FOUNDATIONS

## 1.1 What is a process, and what is a thread?

A **process** is a running program with its own private memory space — when you open two Chrome windows, or run two separate Java programs, those are two separate processes. They cannot directly see or touch each other's memory.

A **thread** is a single path of execution *within* a process. A process can have many threads, and critically, **all threads inside the same process share that process's memory** (the heap) — but each thread still gets its own private **stack** (for local variables and method call tracking) and its own program counter (tracking which line of code it's currently on).

### Why this "shared memory" detail matters more than anything else in this document
Because threads share memory, two threads can both read and write the *same* variable at the *same* time. This is exactly what makes multithreading powerful (threads can cooperate on shared data, like a shared cache or counter) — and exactly what makes it dangerous (if you're not careful, two threads can corrupt that shared data by stepping on each other). Almost every topic in this document exists to manage this one tension: shared memory is useful, but unsafe by default.

## 1.2 Why use multiple threads at all?

Two main reasons:
1. **Parallelism** — genuinely doing multiple things at the same time, using multiple CPU cores, to finish work faster (e.g., processing four chunks of a huge file on four cores simultaneously).
2. **Responsiveness / not blocking** — even on a single core, you don't want your entire application frozen while waiting on a slow operation (like a network call or disk read). A background thread can handle that wait while other threads keep the application responsive.

## 1.3 Creating a thread — two ways

### Way 1: Extend the `Thread` class
```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}

MyThread t = new MyThread();
t.start();
```

### Way 2: Implement `Runnable` (the preferred way)
```java
class MyTask implements Runnable {
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}

Thread t = new Thread(new MyTask());
t.start();

// Or, with a lambda (Runnable is a functional interface):
Thread t2 = new Thread(() -> System.out.println("Running"));
t2.start();
```

### Why Runnable is generally preferred over extending Thread
Java doesn't support multiple inheritance of classes — if your class already extends `Thread`, it can't extend anything else. Implementing `Runnable` instead keeps your class free to extend something else if needed, and it also cleanly separates "the task to run" (Runnable) from "the mechanism that runs it" (Thread) — meaning the very same Runnable can later be handed to an `ExecutorService` thread pool instead of a raw Thread, with zero changes needed.

## 1.4 `start()` vs `run()` — a classic beginner trap

```java
Thread t = new Thread(() -> System.out.println("Hello"));

t.run();    // WRONG — just calls the method normally, on the CURRENT thread. No new thread created!
t.start();  // RIGHT — creates an actual new OS-level thread, which THEN calls run() on it
```
Calling `run()` directly is exactly like calling any other regular method — it executes immediately, on whichever thread called it, and doesn't create anything new. `start()` is the only way to actually get a new thread of execution.

## 1.5 The Thread lifecycle — the states a thread passes through

```
NEW → RUNNABLE → (BLOCKED | WAITING | TIMED_WAITING) → TERMINATED
```

- **NEW**: the `Thread` object has been created (`new Thread(...)`), but `.start()` hasn't been called yet.
- **RUNNABLE**: `.start()` has been called. The thread is eligible to run — it might be actually executing right now, or just waiting for the OS scheduler to give it CPU time. Java doesn't distinguish "waiting for CPU" from "actually running" at the API level — both are just called RUNNABLE.
- **BLOCKED**: the thread is stuck waiting to acquire a lock (e.g., trying to enter a `synchronized` block that another thread currently holds).
- **WAITING**: the thread is waiting indefinitely for another thread to do something (e.g., it called `wait()` or `join()` with no timeout, and is waiting for a `notify()` or for the other thread to finish).
- **TIMED_WAITING**: like WAITING, but with a timeout (e.g., `Thread.sleep(1000)`, or `wait(1000)`) — it'll wake up on its own after the time passes, even without anyone notifying it.
- **TERMINATED**: the thread has finished running (its `run()` method completed, whether normally or via an uncaught exception).

```java
Thread t = new Thread(() -> {
    try { Thread.sleep(1000); } catch (InterruptedException e) {}
});
System.out.println(t.getState()); // NEW
t.start();
System.out.println(t.getState()); // RUNNABLE (or TIMED_WAITING if it's already sleeping)
```

## 1.6 Daemon threads

A **daemon thread** is a background helper thread that doesn't keep the JVM alive by itself. The JVM shuts down automatically once every *non-daemon* thread has finished — even if daemon threads are still running (they simply get abandoned mid-execution when that happens). The JVM's own Garbage Collector thread is a classic example of a daemon thread.

```java
Thread t = new Thread(() -> { /* background work */ });
t.setDaemon(true);  // must be called BEFORE start()
t.start();
```

### Common interview angle
**Q: Why does calling `run()` instead of `start()` not create a new thread?**
Because `run()` is just an ordinary method on the Thread/Runnable object — calling it does nothing special. Only `start()` triggers the actual OS-level thread creation, which the JVM then uses to invoke `run()` on that new thread.

---

# PART 2: THE CORE PROBLEM — RACE CONDITIONS AND SYNCHRONIZATION

## 2.1 What is a race condition?

A race condition happens when the correctness of your program's result depends on the unpredictable *timing* of how threads interleave their execution on shared data. It's called a "race" because multiple threads are effectively racing to read/write the same memory, and whoever gets there in what order determines (unpredictably) what happens.

### A concrete example — why `count++` isn't safe

It looks like one operation, but `count++` is secretly **three separate steps**:
1. Read the current value of `count`.
2. Add 1 to it.
3. Write the new value back to `count`.

```java
int count = 0;

// Thread A                          // Thread B
// reads count = 0                   
//                                   reads count = 0   (Thread B reads BEFORE Thread A writes back!)
// computes 0 + 1 = 1
// writes count = 1
//                                   computes 0 + 1 = 1
//                                   writes count = 1

// Final result: count = 1, even though TWO increments happened — one got lost!
```
If two threads both call `count++` and this exact interleaving happens, one increment is silently lost. This isn't a rare edge case — under real concurrent load, this happens constantly, and is one of the most common sources of subtle, hard-to-reproduce bugs in production systems.

## 2.2 The `synchronized` keyword — the basic fix

### What it does
`synchronized` ensures that only **one thread at a time** can execute a particular block of code (or method) on a given "lock object" — every other thread trying to enter that same synchronized section must wait until the current thread finishes and releases the lock.

```java
class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++; // now safe — only one thread can be inside this method at a time
    }
}
```

### Synchronizing a method vs. synchronizing a block
```java
// Synchronizing the whole method — locks on "this" (the object itself) for the entire method duration
public synchronized void increment() {
    count++;
}

// Synchronizing just a block — you choose exactly what to lock, and for how long
public void increment() {
    // ... some code that doesn't need protection ...
    synchronized (this) {
        count++; // only this critical section is protected
    }
    // ... more unprotected code ...
}
```
Synchronizing only the smallest necessary block (instead of the whole method) is generally better practice — it reduces how long other threads have to wait, since they're only blocked from entering that specific section, not the whole method.

### Every object in Java has a built-in "monitor lock"
`synchronized` works by using an intrinsic lock that literally every Java object automatically has (inherited from `Object`). When a thread enters a `synchronized(someObject)` block, it "acquires" that object's lock; any other thread trying to enter a synchronized section on the *same* object must wait until the first thread exits and releases it.

### `synchronized` gives you TWO guarantees, not just one
This is a commonly missed detail:
1. **Mutual exclusion** — only one thread executes the protected section at a time (this is the "obvious" part).
2. **Visibility** — when a thread exits a synchronized block, all its changes to shared variables are guaranteed to be visible to the *next* thread that enters a synchronized block on that same lock. Without this guarantee, a thread's changes could theoretically stay stuck in that CPU core's local cache and never become visible to other cores/threads at all.

### Common interview angle
**Q: What happens if a synchronized method throws an exception halfway through?**
The lock is still automatically released — `synchronized` guarantees the lock is freed no matter how the block exits (normal completion or an exception), so you never have to manually worry about "forgetting" to release it, unlike with explicit `Lock` objects (covered later).

## 2.3 The `volatile` keyword

### What problem it solves
Modern CPUs have multiple cores, each with their own local cache. When Thread A (running on Core 1) writes to a normal variable, that new value might sit in Core 1's cache for a while before it's "flushed" out to main memory — meaning Thread B (running on Core 2) could keep reading a stale, old value from its own cache, completely unaware that Thread A already changed it.

`volatile` tells the JVM: "never cache this variable locally — every read must go straight to main memory, and every write must go straight to main memory immediately." This guarantees that once one thread writes to a volatile variable, any other thread's very next read of it will see the new value.

```java
class Flag {
    private volatile boolean running = true;

    public void stop() {
        running = false; // this write is immediately visible to all other threads
    }

    public void doWork() {
        while (running) {
            // do something
        }
        // without volatile, this loop could theoretically run FOREVER,
        // because the thread might never notice "running" changed in another thread's memory
    }
}
```

### What `volatile` does NOT do — a critical distinction
`volatile` only guarantees **visibility**, not **atomicity**. It does nothing to make a multi-step operation like `count++` safe, because that's still three separate steps (read, modify, write) even if `count` is volatile — another thread can still interleave in between those three steps.

```java
private volatile int count = 0;

public void increment() {
    count++; // STILL NOT THREAD-SAFE, even with volatile! This is read-modify-write, three steps.
}
```
This exact gotcha — "volatile doesn't make count++ safe" — is one of the most commonly asked trick questions in Java concurrency interviews.

### When volatile IS enough by itself
When you have a simple flag or single value that one thread writes and others only read (not a read-modify-write pattern), like the `running` boolean example above — volatile alone is sufficient and is actually the *lightest-weight* correct solution, cheaper than a full lock.

### Common interview angle
**Q: If volatile doesn't make count++ atomic, what should you use instead?**
Either wrap the increment in a `synchronized` block, or better yet, use `AtomicInteger` (covered in Part 6), which is specifically designed to make single-variable read-modify-write operations atomic without needing a full lock.

## 2.4 `wait()`, `notify()`, and `notifyAll()`

### What problem they solve
Sometimes a thread needs to pause and wait until *some condition becomes true*, rather than just waiting for a lock to become free. For example: a consumer thread needs to wait until there's actually something in a shared buffer to consume. `wait()`/`notify()` let threads communicate this kind of "wake me up when something changes" coordination.

### The rules (all of these must be followed, or you'll get exceptions or bugs)
- `wait()`, `notify()`, and `notifyAll()` can **only** be called from inside a `synchronized` block, on the *same* object whose lock you're holding — calling them outside a synchronized context throws `IllegalMonitorStateException`.
- `wait()` does two things at once: it releases the lock on that object, AND pauses the current thread, until another thread calls `notify()`/`notifyAll()` on that same object.
- `notify()` wakes up **one** arbitrary waiting thread. `notifyAll()` wakes up **all** waiting threads (they then all compete to re-acquire the lock, one at a time).

### A classic producer-consumer example built with wait/notify
```java
class SharedBuffer {
    private final List<Integer> buffer = new ArrayList<>();
    private final int CAPACITY = 5;

    public synchronized void produce(int value) throws InterruptedException {
        while (buffer.size() == CAPACITY) {
            wait(); // buffer is full — release lock, pause here until notified
        }
        buffer.add(value);
        notifyAll(); // wake up any consumer that might be waiting for data
    }

    public synchronized int consume() throws InterruptedException {
        while (buffer.isEmpty()) {
            wait(); // buffer is empty — release lock, pause here until notified
        }
        int value = buffer.remove(0);
        notifyAll(); // wake up any producer that might be waiting for space
        return value;
    }
}
```

### Why `wait()` must ALWAYS be called inside a `while` loop, never an `if`
This is called a **spurious wakeup** — the JVM is technically allowed (by the Java specification) to wake a waiting thread up even without anyone calling `notify()` on it, in rare cases. If you used `if (buffer.isEmpty()) { wait(); }`, a spurious wakeup could let the thread proceed even though the condition (empty buffer) never actually changed. Using a `while` loop means the condition is re-checked immediately after waking up, and if it's still true, the thread just calls `wait()` again — safe no matter why it woke up.

### Why `notifyAll()` is usually safer than `notify()`
If multiple threads are waiting on the same object but for *different* reasons (e.g., some producers waiting for space, some consumers waiting for data), `notify()` might wake up the "wrong" one — a thread whose actual condition still isn't satisfied — and that thread would just go back to waiting, while the thread that *should* have been woken up stays asleep. This can cause a subtle, hard-to-diagnose deadlock-like freeze. `notifyAll()` wakes everyone up to re-check their own condition, at a small extra performance cost, avoiding this entire class of bug.

### Common interview angle
**Q: Difference between `wait()` and `sleep()`?**
`Thread.sleep()` pauses the current thread for a fixed time **without releasing any locks it holds** — other threads waiting for that lock stay blocked the whole time. `wait()` **releases the lock** it's called on, allowing other threads to proceed, and only resumes once explicitly `notify()`-ed (or a timeout passes, if you used the timed overload).

---

# PART 3: THE JAVA MEMORY MODEL (JMM) — WHY ALL OF THIS ACTUALLY WORKS

## 3.1 What the Java Memory Model is, in plain English

The JMM is the official rulebook that defines: when one thread writes to memory, exactly when (if ever) is another thread *guaranteed* to see that write? And what reorderings of your code is the compiler/CPU allowed to perform for optimization, without breaking your program's intended behavior?

This matters because, without such rules, every JVM/CPU combination could behave differently and unpredictably — the JMM is what lets you write concurrent code that behaves consistently across different hardware and JVM implementations.

## 3.2 "Happens-before" — the single most important concept in the JMM

The JMM's guarantees are expressed through a relationship called **happens-before**. If action A "happens-before" action B, it means: everything A did (all its writes to memory) is guaranteed to be visible to B, and the two actions can't be reordered relative to each other in a way that breaks this guarantee.

### The specific happens-before rules you should know
1. **Program order rule**: within a single thread, each action happens-before every action that comes later in that same thread's code (this one's obvious, but it's the baseline).
2. **Monitor lock rule**: unlocking a `synchronized` block happens-before any later thread locking that *same* object's monitor. This is exactly why synchronized gives you visibility, not just mutual exclusion.
3. **Volatile variable rule**: a write to a `volatile` variable happens-before any subsequent read of that *same* variable by another thread.
4. **Thread start rule**: everything a thread did before calling `someThread.start()` happens-before anything the newly started thread does.
5. **Thread join rule**: everything a thread does happens-before another thread successfully returns from `join()` on it (i.e., once you've confirmed a thread finished via `join()`, you're guaranteed to see everything it did).

### Why this matters practically
This is the theoretical foundation that explains *why* `synchronized` and `volatile` work the way they do — they're not magic, they're specific, well-defined ways of establishing a happens-before relationship between threads. If you write shared-memory code without establishing any happens-before relationship at all between the writing thread and the reading thread, there is **no guarantee** the reader will ever see the writer's changes — not "probably will, eventually," but genuinely no guarantee at all, by specification.

### Common interview angle
**Q: If Thread A writes to a normal (non-volatile, non-synchronized) variable, and Thread B reads it right after — will B definitely see the new value?**
Not necessarily! Without an established happens-before relationship (via synchronized, volatile, or another JMM-recognized mechanism) connecting that specific write to that specific read, there's no guarantee — this is precisely the scenario `volatile` and `synchronized` exist to fix.

---

# PART 4: PROBLEMS THAT ARISE WITH MULTIPLE LOCKS

## 4.1 Deadlock

### What it is
Two (or more) threads get stuck permanently waiting for each other, each holding a lock the other one needs.

### The classic example
```java
Object lockA = new Object();
Object lockB = new Object();

// Thread 1
synchronized (lockA) {
    // ... 
    synchronized (lockB) { // Thread 1 waits here if Thread 2 already holds lockB
        // ...
    }
}

// Thread 2 (running at the same time)
synchronized (lockB) {
    // ...
    synchronized (lockA) { // Thread 2 waits here if Thread 1 already holds lockA
        // ...
    }
}
```
If Thread 1 grabs `lockA` at the same moment Thread 2 grabs `lockB`, then Thread 1 waits forever for `lockB` (held by Thread 2), and Thread 2 waits forever for `lockA` (held by Thread 1). Neither can ever proceed — a true deadlock.

### The four conditions that must ALL be true for a deadlock to occur (useful mental model)
1. **Mutual exclusion** — a resource can only be held by one thread at a time.
2. **Hold and wait** — a thread holds one resource while waiting for another.
3. **No preemption** — a resource can't be forcibly taken away from a thread; it must be voluntarily released.
4. **Circular wait** — there's a cycle of threads, each waiting for a resource held by the next one in the cycle.
Break any *one* of these conditions, and deadlock becomes impossible.

### How to prevent deadlock in practice
The most common, practical fix is breaking the "circular wait" condition: **always acquire multiple locks in the same, fixed global order, everywhere in your codebase.** If every thread always locks `lockA` before `lockB` (never the reverse), the circular waiting pattern above can never form.

```java
// Both threads now agree: always lockA first, then lockB — no more circular wait possible
synchronized (lockA) {
    synchronized (lockB) {
        // ...
    }
}
```

Other approaches: using `tryLock()` with a timeout (covered in Part 5) instead of blocking forever, so a thread can back off and retry rather than waiting indefinitely; and minimizing how many locks you hold simultaneously in the first place.

## 4.2 Livelock

### What it is
Unlike deadlock, threads in a livelock are **not** blocked or stuck — they're actively running, actively responding to each other — but they make **zero actual progress**, because they keep reacting to each other in a way that just repeats forever.

### An analogy
Picture two people in a narrow hallway, both stepping aside to let the other pass — but they both happen to step to the *same* side at the *same* time, over and over, repeatedly blocking each other, even though both are actively trying to be polite and make progress.

### Why it happens in code
Often caused by poorly designed "back off and retry" logic — e.g., two threads that both detect a potential conflict and both immediately retry using the exact same strategy/timing, so they keep colliding in the same way indefinitely. Fixing this often involves adding some randomness to retry delays, so threads don't stay perfectly "in sync" with each other's retries.

## 4.3 Starvation

### What it is
A thread is repeatedly denied access to a resource it needs, not because of a hard deadlock, but simply because other threads keep "winning" and getting there first — indefinitely.

### A common cause
Java's thread scheduling isn't guaranteed to be fair by default — a `synchronized` block gives no promises about which waiting thread gets the lock next. If some threads are much more aggressive about repeatedly trying to acquire a heavily-contended lock, a particular unlucky thread could theoretically wait an extremely long time (in the worst case, indefinitely) while others keep cutting in front of it.

### How to address it
Use a **fair lock** — `ReentrantLock` (covered in Part 5) can be constructed with a fairness policy that grants the lock to whichever thread has been waiting the longest, preventing any one thread from being perpetually skipped over — at some cost to overall throughput, since strict fairness adds coordination overhead.

### Common interview angle
**Q: How is livelock different from deadlock?**
In deadlock, threads are blocked and doing nothing at all, waiting forever. In livelock, threads are actively running and responding to each other, but their actions keep cancelling each other out, so no real progress happens either way.

---

# PART 5: MODERN CONCURRENCY TOOLS (`java.util.concurrent`)

## 5.1 Why these tools exist

Everything in Parts 2–4 (synchronized, wait/notify, manual deadlock avoidance) is the "raw materials" level of Java concurrency — it works, but it's easy to get subtly wrong. The `java.util.concurrent` package (introduced in Java 5, expanded significantly in Java 8) gives you higher-level, pre-built, battle-tested tools for common concurrency patterns, so you rarely need to hand-write `wait()`/`notify()` logic yourself in modern code.

## 5.2 `ExecutorService` and Thread Pools

### The problem it solves
Creating a brand-new OS thread for every single task is expensive (thread creation/destruction has real overhead), and if you create threads without limit as tasks come in, you can exhaust system resources entirely under heavy load. A **thread pool** solves this by maintaining a fixed (or bounded) set of reusable worker threads that pick up tasks from a queue, one after another, instead of spinning up a new thread every time.

```java
ExecutorService executor = Executors.newFixedThreadPool(4); // pool of 4 reusable threads

executor.submit(() -> System.out.println("Task 1"));
executor.submit(() -> System.out.println("Task 2"));
// ... submit as many tasks as you like — they queue up and get picked up by the 4 available threads

executor.shutdown(); // important — tells the pool to stop accepting new tasks and shut down once current tasks finish
```

### The common factory methods in `Executors`
```java
Executors.newFixedThreadPool(n);      // exactly n threads, unbounded task queue
Executors.newCachedThreadPool();      // creates threads as needed, reuses idle ones — but can grow UNBOUNDED under heavy load
Executors.newSingleThreadExecutor();  // exactly 1 thread — tasks run one at a time, in order
Executors.newScheduledThreadPool(n);  // supports scheduling tasks to run after a delay, or repeatedly
```

### A real-world caveat worth knowing
`newCachedThreadPool()` and the default queues behind these factory methods can hide unbounded growth — if tasks come in faster than they can be processed, an unbounded queue (or unbounded thread creation) can eventually exhaust memory. In production systems, many teams construct `ThreadPoolExecutor` directly instead, so they can explicitly set a bounded queue size and a rejection policy (what to do when the pool is completely overwhelmed) rather than relying on the "convenient" defaults.

### Always remember to shut down an ExecutorService
If you never call `shutdown()`, the pool's threads stay alive indefinitely, which can prevent your application from exiting cleanly (since these are typically non-daemon threads).

```java
executor.shutdown();               // graceful — finishes queued/running tasks first, then stops
executor.shutdownNow();            // aggressive — attempts to stop currently running tasks immediately
executor.awaitTermination(5, TimeUnit.SECONDS); // blocks, waiting up to 5 seconds for shutdown to finish
```

## 5.3 `Runnable` vs `Callable`

### The difference
`Runnable.run()` returns nothing (`void`) and cannot throw a checked exception. `Callable<V>.call()` **returns a value** of type `V`, and **can** throw checked exceptions — making it the right choice whenever your background task needs to produce a result.

```java
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};

Future<Integer> future = executor.submit(task);
Integer result = future.get(); // blocks until the task finishes, then returns 42
```

## 5.4 `Future` — and its limitations

### What it is
`Future<V>` represents the eventual result of an asynchronous computation submitted to an executor. You can check if it's done (`isDone()`), cancel it (`cancel()`), or retrieve the result (`get()`, which **blocks** the calling thread until the result is ready).

```java
Future<Integer> future = executor.submit(() -> {
    Thread.sleep(2000);
    return 100;
});

System.out.println("Doing other work while task runs...");
Integer result = future.get(); // blocks here until the background task finishes
```

### The core limitation
`Future.get()` is blocking — there's no built-in way to say "run this callback automatically once the result is ready" or "chain another async step after this one finishes" using plain `Future`. You either block and wait, or you have to manually poll `isDone()` in a loop (wasteful). This exact limitation is what `CompletableFuture` was built to solve.

## 5.5 `CompletableFuture` — non-blocking, chainable async programming

### What it adds over Future
`CompletableFuture<T>` lets you describe a *pipeline* of dependent asynchronous steps — "when this finishes, then automatically do that next" — without ever needing to block and manually wait in the middle.

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // runs on a background thread (the common ForkJoinPool, by default)
    return "raw result";
});

future.thenApply(result -> result.toUpperCase())          // transform the result once ready
      .thenAccept(result -> System.out.println(result));  // consume it, print — all non-blocking
```

### The key chaining methods
```java
.thenApply(fn)     // transforms the result (like Stream's map) — returns a new CompletableFuture
.thenAccept(fn)     // consumes the result, returns nothing (like Stream's forEach)
.thenRun(fn)        // runs some code after completion, doesn't even need the result value
.thenCompose(fn)    // chains to ANOTHER CompletableFuture-returning operation (like Stream's flatMap) —
                    // use this instead of thenApply when your next step is itself async
.exceptionally(fn)  // handles any exception thrown earlier in the chain
```

### Combining multiple independent async tasks
```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

// Combine two independent futures once BOTH are done
CompletableFuture<String> combined = future1.thenCombine(future2, (a, b) -> a + " " + b);

// Wait for several futures, without caring about individual results
CompletableFuture<Void> all = CompletableFuture.allOf(future1, future2);

// Proceed as soon as ANY ONE of several futures completes
CompletableFuture<Object> any = CompletableFuture.anyOf(future1, future2);
```

### Which thread pool does the work actually run on?
By default, `supplyAsync()`/`thenApply()` etc. run on the shared, application-wide **common `ForkJoinPool`** (the same pool that powers parallel streams). In production, it's common to pass your own dedicated `Executor` as an extra argument, so that CPU-heavy or I/O-blocking tasks in one part of your application don't accidentally starve unrelated work elsewhere that also happens to rely on that same shared common pool.

```java
Executor myExecutor = Executors.newFixedThreadPool(4);
CompletableFuture.supplyAsync(() -> doWork(), myExecutor);
```

### Common interview angle
**Q: What's the key advantage of CompletableFuture over Future?**
Non-blocking composition — you can chain, combine, and react to async results automatically (`thenApply`, `thenCompose`, `exceptionally`) instead of being forced to call a blocking `get()` and manually manage sequencing yourself.

## 5.6 `ReentrantLock` — an explicit alternative to `synchronized`

### Why it exists, given that `synchronized` already works
`ReentrantLock` gives you capabilities `synchronized` simply doesn't have:
- `tryLock()` — attempt to grab the lock, but **don't block forever** if it's unavailable; you get an immediate true/false answer (optionally with a timeout).
- `lockInterruptibly()` — a thread waiting for this lock can be interrupted (woken up early) by another thread, instead of being stuck waiting no matter what.
- **Fairness policy** — can be configured to grant the lock to whichever thread has waited longest, preventing starvation (see Part 4.3).
- **Multiple Condition objects per lock** (see below) — lets you have several independent "wait groups" tied to one lock, instead of just one shared wait-set like a plain object's monitor has.

### Basic usage — and the critical "always unlock in finally" rule
```java
Lock lock = new ReentrantLock();

lock.lock();
try {
    // critical section
} finally {
    lock.unlock(); // MUST be in finally — if you forget this, and an exception is thrown above,
                   // the lock is NEVER released, and every other thread waiting for it blocks forever!
}
```
This is the single biggest practical difference from `synchronized`: `synchronized` releases its lock automatically no matter how the block exits (normal completion or exception). `ReentrantLock` requires you to unlock manually — forgetting the `finally` block is a real, common bug.

### `tryLock()` in action — avoiding indefinite blocking
```java
if (lock.tryLock()) {
    try {
        // got the lock — proceed
    } finally {
        lock.unlock();
    }
} else {
    // couldn't get the lock right now — do something else instead of blocking forever
}
```

### `Condition` objects — multiple wait-sets on one lock
```java
Lock lock = new ReentrantLock();
Condition notFull = lock.newCondition();
Condition notEmpty = lock.newCondition();

// A producer waits specifically on "notFull", a consumer waits specifically on "notEmpty" —
// so signaling one doesn't unnecessarily wake up threads waiting for the other condition,
// unlike a single object's wait()/notifyAll() which shares ONE wait-set for everything.
```

## 5.7 `ReentrantReadWriteLock`

### The problem it solves
Sometimes a shared resource is **read very often but written rarely**. A plain lock (synchronized or ReentrantLock) forces even simultaneous *readers* to take turns one at a time — wasteful, since reading doesn't actually conflict with other reading.

### How it works
`ReentrantReadWriteLock` provides two separate locks that work together: a **read lock**, which any number of threads can hold *simultaneously* (as long as no thread holds the write lock), and a **write lock**, which is fully exclusive (only one thread, and no readers, at a time).

```java
ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

rwLock.readLock().lock();
try {
    // multiple threads can be here AT THE SAME TIME, reading concurrently
} finally {
    rwLock.readLock().unlock();
}

rwLock.writeLock().lock();
try {
    // only ONE thread can be here, and no readers are allowed in at the same time
} finally {
    rwLock.writeLock().unlock();
}
```

## 5.8 `CountDownLatch`

### What it is
A simple, **one-time-use** synchronization tool: you initialize it with a count, and any number of threads can call `await()` to block until that count reaches zero — with other threads calling `countDown()` to decrease it.

### A common use case: waiting for several worker threads to finish before continuing
```java
CountDownLatch latch = new CountDownLatch(3); // expecting 3 workers to finish

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        // do some work
        latch.countDown(); // signal "I'm done"
    }).start();
}

latch.await(); // main thread blocks here until all 3 workers have called countDown()
System.out.println("All workers finished!");
```

### The key limitation
Once the count reaches zero, the latch is "used up" — you cannot reset it and reuse it for another round. If you need repeated synchronization rounds, use `CyclicBarrier` instead.

## 5.9 `CyclicBarrier`

### What it is, and how it differs from CountDownLatch
`CyclicBarrier` makes a **fixed group of threads** all wait for each other at a common point — none of them proceed past the barrier until *all* of them have arrived. Unlike `CountDownLatch`, once everyone arrives and is released, the barrier automatically **resets** and can be reused for another round of waiting.

```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> System.out.println("All threads reached the barrier!"));

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        // do some work
        try {
            barrier.await(); // wait here until all 3 threads reach this point
        } catch (Exception e) {}
        // continues here only after all 3 have arrived — and the barrier can be used again for another round
    }).start();
}
```

### When to use CyclicBarrier over CountDownLatch
When you have several threads that need to repeatedly work in synchronized "rounds" (e.g., a simulation where every thread must finish step 1 before any of them starts step 2), rather than a single one-time "wait for these things to finish."

## 5.10 `Semaphore`

### What it is
Maintains a fixed number of "permits." Threads call `acquire()` to take a permit (blocking if none are available) and `release()` to give one back. This is the general-purpose tool for **limiting how many threads can access a resource concurrently**, not just to exactly one (that's what a lock is for) but to any chosen number N.

```java
Semaphore semaphore = new Semaphore(3); // allow at most 3 concurrent threads through

semaphore.acquire(); // blocks if 3 threads are already "inside"
try {
    // access the limited resource — at most 3 threads here at once
} finally {
    semaphore.release();
}
```

### A real-world use case
Limiting concurrent connections to a database or external API to some fixed maximum, so you don't overwhelm that resource even if many application threads want to use it simultaneously.

### Common interview angle
**Q: What's the difference between a Semaphore with 1 permit and a Lock?**
They're conceptually similar (both allow only one thread "in" at a time with a single permit), but a Semaphore doesn't have the concept of "ownership" — any thread can call `release()`, not necessarily the same thread that called `acquire()`. A Lock, by contrast, generally must be released by the thread that acquired it (particularly true for ReentrantLock, which tracks the owning thread).

## 5.11 `BlockingQueue` — the easiest way to build producer-consumer systems

### What it is
A `Queue` where `put()` automatically blocks if the queue is full, and `take()` automatically blocks if the queue is empty — internally handling all the wait/notify-style coordination for you, so you never have to hand-write it yourself.

```java
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10); // bounded, capacity 10

// Producer thread
new Thread(() -> {
    try {
        for (int i = 0; i < 100; i++) {
            queue.put(i); // blocks automatically once the queue has 10 items already
        }
    } catch (InterruptedException e) {}
}).start();

// Consumer thread
new Thread(() -> {
    try {
        while (true) {
            int value = queue.take(); // blocks automatically if the queue is currently empty
            System.out.println("Consumed: " + value);
        }
    } catch (InterruptedException e) {}
}).start();
```

### Why this is almost always better than hand-writing wait/notify producer-consumer logic
Compare this to the manual `wait()`/`notify()` version from Part 2.4 — that required carefully writing the exact "while empty/full, wait" loop yourself and remembering to call `notifyAll()` at the right moments. `BlockingQueue` handles all of that internally, correctly, every time — dramatically reducing the chance of a subtle bug.

### Common implementations
- `ArrayBlockingQueue` — bounded, fixed-capacity, backed by an array.
- `LinkedBlockingQueue` — optionally bounded, backed by linked nodes (unbounded by default if no capacity given — be careful with that in production).
- `PriorityBlockingQueue` — unbounded, elements come out in priority order rather than FIFO.
- `SynchronousQueue` — holds **zero** elements; every `put()` must be matched by a simultaneous `take()` from another thread — a direct, synchronous hand-off, nothing is ever actually "stored."

---

# PART 6: LOCK-FREE PROGRAMMING — ATOMIC CLASSES AND CAS

## 6.1 What "lock-free" means, and why it's attractive

Every synchronization mechanism so far (synchronized, ReentrantLock, Semaphore) works by **blocking** other threads — making them wait. Blocking has real costs: context switches, threads sitting idle, potential for deadlock. **Lock-free** programming achieves thread safety *without* ever blocking a thread — using a special hardware-level trick called CAS.

## 6.2 CAS (Compare-And-Swap) — the mechanism behind lock-free code

### What CAS actually does
CAS is a single, atomic CPU instruction (meaning: the hardware guarantees it happens as one indivisible step, even with multiple cores) that does this: *"Compare the current value in this memory location to an expected value. If they match, swap in a new value. If they don't match (because someone else already changed it), don't swap — just tell me it failed."*

### Why this enables safe updates without locking
A thread can safely update a shared value like this:
1. Read the current value.
2. Compute the new value based on it.
3. Attempt a CAS: "if the value is still what I read in step 1, swap it to my new computed value."
4. If the CAS succeeds, done. If it failed (meaning another thread changed the value in between), **retry** the whole thing from step 1.

This is fundamentally different from locking: instead of *preventing* other threads from touching the value at all, it lets everyone try optimistically, and just retries if there was a genuine conflict — usually much cheaper than blocking, especially when conflicts are rare.

## 6.3 The Atomic classes (`AtomicInteger`, `AtomicLong`, `AtomicReference`, etc.)

### What they are
Wrapper classes around a single value that provide atomic, lock-free read-modify-write operations, built internally using CAS.

```java
AtomicInteger counter = new AtomicInteger(0);

counter.incrementAndGet();  // atomically adds 1 and returns the new value — genuinely thread-safe, no lock needed
counter.get();               // reads the current value
counter.compareAndSet(5, 10); // if current value is 5, atomically set it to 10; returns true/false whether it succeeded
counter.getAndAdd(3);         // atomically adds 3, returns the value BEFORE adding
```

### Why this solves the original `count++` problem cleanly
```java
// UNSAFE:
private volatile int count = 0;
public void increment() { count++; } // still 3 separate steps, race condition possible

// SAFE, and lock-free:
private AtomicInteger count = new AtomicInteger(0);
public void increment() { count.incrementAndGet(); } // genuinely one atomic operation
```

### Common interview angle
**Q: Why are Atomic classes often faster than synchronized for simple counters?**
Because they avoid blocking entirely under normal (low-to-moderate contention) conditions — no thread ever has to sleep and wait for a lock to free up; at worst, a thread just retries its CAS attempt a couple of times. Under very high contention, this retry overhead can add up, but it's still often cheaper than the cost of full locking with context switches.

## 6.4 The ABA problem — an advanced gotcha in CAS-based code

### What it is
CAS only checks: "is the current value still equal to what I originally read?" But imagine this sequence: a value starts as **A**, another thread changes it to **B**, and then a third thread changes it back to **A** again — all before your CAS attempt runs. Your CAS check ("is it still A?") will succeed, since the value genuinely IS "A" again — but something *did* change in between, and depending on what that value represents (e.g., a pointer in a lock-free linked structure), that intermediate change could have caused real problems your code never noticed.

### The fix: `AtomicStampedReference`
This pairs your value with an extra integer "stamp" (essentially a version number) that increments every single time the value changes — so even if the value goes A→B→A, the *stamp* will have changed, and a CAS checking both the value AND the stamp together will correctly detect that something happened in between.

```java
AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);
int[] stampHolder = new int[1];
String value = ref.get(stampHolder); // gets both the value and its current stamp

ref.compareAndSet("A", "C", stampHolder[0], stampHolder[0] + 1);
// succeeds only if BOTH the value AND the stamp still match what we expect
```

### Common interview angle
This is a genuinely advanced question — if asked, the key point to convey is simply that CAS alone can't distinguish "unchanged" from "changed and changed back," and that's what the stamped/versioned variant exists to fix.

---

# PART 7: THREAD-LOCAL STORAGE

## 7.1 What `ThreadLocal` is

`ThreadLocal<T>` gives **each thread its own independent, private copy** of a variable — even though the `ThreadLocal` object itself is shared, each thread that calls `.get()`/`.set()` on it is actually reading/writing its own isolated value, invisible to other threads.

```java
ThreadLocal<Integer> threadLocalValue = ThreadLocal.withInitial(() -> 0);

threadLocalValue.set(100); // sets the value ONLY for the current thread
System.out.println(threadLocalValue.get()); // 100, but only when read from THIS thread
```

## 7.2 Why this is genuinely useful

### Use case 1: making non-thread-safe objects safe, per-thread
Some classes, like the old `SimpleDateFormat`, are famously not thread-safe if shared across threads. Giving each thread its own private instance sidesteps the problem entirely, without needing any synchronization at all:

```java
ThreadLocal<SimpleDateFormat> dateFormat = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

// Each thread gets its OWN SimpleDateFormat instance — no sharing, no race conditions, no locking needed
String formatted = dateFormat.get().format(new Date());
```

### Use case 2: implicit context passing (very common in real web applications)
Rather than passing a "current user" or "request ID" object through every single method call in a deep call chain, you can store it in a ThreadLocal once (typically at the very start of handling a request) and retrieve it anywhere later in that same thread's execution, without threading (pun intended) the parameter through every method signature.

```java
public class RequestContext {
    private static final ThreadLocal<String> currentUserId = new ThreadLocal<>();

    public static void setUserId(String userId) { currentUserId.set(userId); }
    public static String getUserId() { return currentUserId.get(); }
}

// Set once, early in request handling:
RequestContext.setUserId("user123");

// Read anywhere later, in this same thread, without passing it as a parameter:
String userId = RequestContext.getUserId();
```

## 7.3 The memory leak danger — especially with thread pools

### Why this is a real, common production bug
Threads in a thread pool are **reused** across many different tasks over time — they're never actually destroyed between tasks. If you `set()` a value on a ThreadLocal during one task but never call `remove()`, that value stays attached to the pooled thread **forever** (or until the thread eventually dies, which in a pool might be "never" for the application's lifetime). This means:
1. The object you stored can never be garbage collected, as long as that thread lives — a slow memory leak.
2. Worse: the *next*, completely unrelated task that happens to run on that same reused thread will see the *previous* task's leftover ThreadLocal value — a subtle and confusing correctness bug (e.g., one user's request accidentally seeing another user's leftover context data).

### The fix: always clean up
```java
try {
    RequestContext.setUserId("user123");
    // ... handle the request ...
} finally {
    RequestContext.currentUserId.remove(); // ALWAYS clean up, especially in pooled-thread environments
}
```

### Common interview angle
**Q: Why is ThreadLocal risky in a thread pool specifically, but less risky with manually-created, short-lived threads?**
Because manually created threads are typically used once and then discarded — there's no "next task" to accidentally inherit stale data. Pooled threads are reused indefinitely across unrelated tasks, so any ThreadLocal value left behind by one task can leak into a completely different, later task running on that same physical thread.

---

# PART 8: FORK/JOIN AND PARALLEL STREAMS

## 8.1 The problem `ForkJoinPool` solves

Regular thread pools (like `newFixedThreadPool`) are great for many independent, roughly equal-sized tasks. But they're not ideal for **divide-and-conquer** problems — where one big task splits into smaller subtasks, which split into even smaller subtasks, recursively, until they're small enough to solve directly, and then the results get combined back together (think: parallel merge sort, or summing a huge array by splitting it in half repeatedly).

## 8.2 How `ForkJoinPool` works — the "work-stealing" algorithm

### The core idea
Each worker thread in a `ForkJoinPool` has its **own personal queue** of tasks (technically a double-ended queue). A thread normally works on tasks from its own queue. But if a thread finishes all of its own tasks early (because its part of the work happened to be smaller, or it's just faster), instead of sitting idle, it "steals" a task from the **back** of some other, still-busy thread's queue — while that other thread continues working from the **front** of its own queue. This keeps all CPU cores busy and balances the workload automatically, without any central coordinator deciding who does what.

```java
class SumTask extends RecursiveTask<Long> {
    private final int[] array;
    private final int start, end;
    private static final int THRESHOLD = 1000;

    SumTask(int[] array, int start, int end) {
        this.array = array; this.start = start; this.end = end;
    }

    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += array[i];
            return sum; // small enough — just compute directly
        }
        int mid = (start + end) / 2;
        SumTask left = new SumTask(array, start, mid);
        SumTask right = new SumTask(array, mid, end);
        left.fork();                 // hand off the left half to run asynchronously
        long rightResult = right.compute(); // compute the right half on this same thread
        long leftResult = left.join();      // wait for the left half's result
        return leftResult + rightResult;
    }
}

ForkJoinPool pool = new ForkJoinPool();
long total = pool.invoke(new SumTask(bigArray, 0, bigArray.length));
```

## 8.3 How this connects to Parallel Streams

`list.parallelStream()` uses this exact same `ForkJoinPool` mechanism internally (specifically, the shared "common pool" by default) — it automatically splits your collection into chunks, processes them across multiple threads using work-stealing, and merges the results, all hidden behind the simple `.parallelStream()` call.

### When parallel streams genuinely help vs. when they don't
- **Helps**: large datasets, CPU-intensive per-element work, stateless/independent operations.
- **Doesn't help (or actively hurts)**: small datasets (splitting/merging overhead exceeds any benefit), operations involving shared mutable state (risk of race conditions — e.g., multiple threads appending to a plain, non-thread-safe `ArrayList` at once), or I/O-bound work (parallel streams don't help you wait on a network call faster).

```java
// DANGEROUS — plain ArrayList isn't thread-safe, multiple threads writing to it concurrently
List<Integer> results = new ArrayList<>();
numbers.parallelStream().forEach(n -> results.add(n * 2)); // race condition, can corrupt the list!

// SAFE — let collect() handle proper, thread-safe aggregation internally
List<Integer> results2 = numbers.parallelStream()
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

### Common interview angle
**Q: Does using parallelStream() always make code faster?**
No — it depends heavily on data size and the nature of the work. For small collections or I/O-bound tasks, the overhead of splitting work across threads and merging results can make it *slower* than a plain sequential stream. Always benchmark rather than assume.

---

# PART 9: PUTTING IT ALL TOGETHER — A REALISTIC EXAMPLE

Here's a small idempotent request-processing example combining several tools from this document, the way you'd actually structure real production code (this style is directly relevant to fintech/payments-style systems):

```java
public class IdempotentProcessor {
    // ConcurrentHashMap: thread-safe, no coarse locking needed
    private final ConcurrentHashMap<String, CompletableFuture<String>> inFlightRequests = new ConcurrentHashMap<>();
    private final ExecutorService executor = Executors.newFixedThreadPool(8);

    public CompletableFuture<String> process(String requestId, Supplier<String> work) {
        // computeIfAbsent: ATOMIC check-then-act — no race window between "is this request already running?"
        // and "start processing it" — critical for correctness with concurrent duplicate requests
        return inFlightRequests.computeIfAbsent(requestId, id ->
            CompletableFuture.supplyAsync(work, executor)
                .whenComplete((result, error) -> inFlightRequests.remove(id)) // clean up once done
        );
    }

    public void shutdown() {
        executor.shutdown();
    }
}
```
This single example ties together: `ConcurrentHashMap` (thread-safe storage), `computeIfAbsent` (atomic check-then-act, avoiding a race condition that a plain HashMap + manual check would have), `CompletableFuture` (non-blocking async execution), and a dedicated `ExecutorService` (bounded thread pool, not relying on the shared common pool).

---

# Quick-reference summary table

| Concept | One-line purpose |
|---|---|
| Thread / Runnable | The basic unit of concurrent execution |
| synchronized | Mutual exclusion + visibility, using an object's built-in monitor lock |
| volatile | Guarantees visibility of a single variable's latest value — NOT atomicity |
| wait/notify/notifyAll | Manual thread coordination — pause until a condition changes |
| Java Memory Model / happens-before | The rulebook defining when writes become visible across threads |
| Deadlock | Threads stuck forever, each holding a lock the other needs |
| Livelock | Threads actively running but making no real progress |
| Starvation | A thread perpetually denied a resource due to unfair scheduling |
| ExecutorService / thread pools | Reusable worker threads, avoiding per-task thread creation cost |
| Callable / Future | A task that returns a value / a handle to its (blocking) eventual result |
| CompletableFuture | Non-blocking, chainable, composable asynchronous computation |
| ReentrantLock | Explicit lock with tryLock, fairness, interruptibility, multiple Conditions |
| ReentrantReadWriteLock | Many concurrent readers, one exclusive writer |
| CountDownLatch | One-time "wait for N things to finish" |
| CyclicBarrier | Reusable "wait for a group of threads to all reach this point" |
| Semaphore | Limit concurrent access to a resource to N threads |
| BlockingQueue | Auto-blocking queue — the easy way to build producer-consumer systems |
| Atomic classes / CAS | Lock-free, thread-safe single-variable updates |
| ABA problem / AtomicStampedReference | CAS's blind spot for "changed and changed back," and its fix |
| ThreadLocal | Per-thread private copy of a variable — watch for leaks in thread pools |
| ForkJoinPool / work-stealing | Efficient parallel execution for divide-and-conquer style problems |
| Parallel Streams | Automatic multi-core stream processing, built on ForkJoinPool |

---

## Suggested study order for interview prep

1. **Part 1 and 2** (thread basics, race conditions, synchronized, volatile) — the non-negotiable foundation; expect direct questions on the `count++` / volatile gotcha specifically.
2. **Part 3** (Java Memory Model, happens-before) — fewer companies ask this in full depth, but understanding it makes every other answer you give sound like genuine understanding rather than memorization.
3. **Part 4** (deadlock, livelock, starvation) — very commonly asked, especially "how do you prevent deadlock" and "explain a real deadlock scenario."
4. **Part 5** (ExecutorService, Future, CompletableFuture, locks, latches/barriers, BlockingQueue) — the heart of most practical interview questions, especially at product/fintech companies doing real concurrent systems.
5. **Part 6** (Atomic classes, CAS, ABA problem) — commonly asked as a follow-up once you've shown you understand `synchronized`/`volatile` well.
6. **Part 7** (ThreadLocal) — moderate depth expected; the thread-pool memory leak gotcha is a favorite follow-up question.
7. **Part 8** (ForkJoinPool, parallel streams) — good supporting knowledge, ties directly back to your Java 8 Streams knowledge.
8. **Part 9** — practice explaining a combined example like this one out loud; interviewers at product companies often want to see you connect multiple concepts into one coherent design, not just recite definitions.
