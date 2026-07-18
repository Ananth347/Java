# Java Multithreading & Concurrency — Complete Interview Q&A
### Basic → Advanced, every topic covered, with clear explanations

---

## How to use this document
Questions are grouped by topic and move from basic to hard within each section. Read a section fully before moving to the next — later questions often build on earlier ones. This is meant to be exhaustive, so treat it as a study companion, not a one-sitting read.

**Topics covered:** Threads & Lifecycle · Race Conditions & `synchronized` · `volatile` · `wait`/`notify` · The Java Memory Model · Deadlock, Livelock, Starvation · ExecutorService & Thread Pools · Callable, Future & CompletableFuture · Locks (`ReentrantLock`, `ReadWriteLock`) · CountDownLatch, CyclicBarrier & Semaphore · BlockingQueue · Atomic Classes & CAS · ThreadLocal · ForkJoinPool & Parallel Streams · Thread-safe Collections · Mixed/Scenario-based questions.

---

# SECTION 1: Threads and the Thread Lifecycle (Q1–Q20)

## Basic

**Q1. What is the difference between a process and a thread?**
A process is an independent running program with its own private memory space. A thread is a single path of execution inside a process. All threads inside the same process **share** that process's memory (the heap), but each thread still has its own private stack and program counter.

**Q2. Why does a Java program use multiple threads?**
Two main reasons: **parallelism** (using multiple CPU cores to finish work faster) and **responsiveness** (not freezing the whole application while waiting on something slow, like a network call).

**Q3. What are the two ways to create a thread in Java?**
1. Extend the `Thread` class and override `run()`.
2. Implement the `Runnable` interface and pass it to a `Thread` object.
```java
Thread t1 = new Thread() { public void run() { System.out.println("A"); } };
Thread t2 = new Thread(() -> System.out.println("B"));
```

**Q4. Why is implementing `Runnable` usually preferred over extending `Thread`?**
Java doesn't support multiple inheritance of classes — if your class already extends `Thread`, it can't extend anything else. `Runnable` also cleanly separates "the task" from "the thing that runs it," so the same task can later be handed to a thread pool instead of a raw `Thread`.

**Q5. What's the difference between calling `start()` and calling `run()` directly?**
`start()` creates a real new OS-level thread, which then calls `run()` on it. Calling `run()` directly just runs the code like a normal method call — on the current thread, no new thread is created.

**Q6. What are the states in a Java thread's lifecycle?**
NEW → RUNNABLE → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED.

**Q7. What does the RUNNABLE state actually mean?**
It means the thread is eligible to run — it could be actually executing right now, or just waiting for the OS scheduler to give it CPU time. Java's API doesn't distinguish between these two at the state level.

**Q8. What's the difference between BLOCKED and WAITING?**
BLOCKED means the thread is waiting to acquire a lock (like trying to enter a `synchronized` block someone else holds). WAITING means the thread is waiting indefinitely for another thread to do something (like calling `wait()` with no timeout, or `join()`).

**Q9. What is a daemon thread?**
A background thread that doesn't keep the JVM alive on its own. Once all non-daemon threads finish, the JVM shuts down, even if daemon threads are still running — they simply get abandoned. The JVM's Garbage Collector thread is a classic example.

**Q10. How do you make a thread a daemon thread?**
```java
Thread t = new Thread(() -> {});
t.setDaemon(true); // must be called BEFORE start()
t.start();
```

## Intermediate

**Q11. Can you restart a thread once it has finished (reached TERMINATED)?**
No — once a thread's `run()` method finishes, calling `start()` on that same `Thread` object again throws `IllegalThreadStateException`. You'd need to create a brand-new `Thread` object.

**Q12. What does `Thread.currentThread()` return?**
A reference to the `Thread` object that is currently executing that piece of code — useful for logging or checking the thread's name, priority, etc.

**Q13. What is `Thread.join()`, and what does it do?**
Makes the calling thread wait until the thread it's called on finishes execution.
```java
Thread t = new Thread(() -> doWork());
t.start();
t.join(); // main thread waits here until t finishes
System.out.println("t is done");
```

**Q14. What happens if you call `join()` on a thread that has already finished?**
It returns immediately — there's nothing to wait for, since the thread has already terminated.

**Q15. What does thread priority do, and is it reliable?**
Threads have a priority (1–10) that hints to the OS scheduler which threads to favor. It's not a guarantee — actual scheduling depends on the OS/JVM, so priority should never be relied on for program correctness.

**Q16. Can you name a thread, and why would you want to?**
```java
Thread t = new Thread(() -> {}, "Worker-1");
```
Yes — useful for debugging and logging, since a named thread is much easier to identify in a stack trace or log file than the default "Thread-0", "Thread-1", etc.

## Advanced

**Q17. What is `Thread.interrupt()`, and how should a thread respond to it?**
`interrupt()` sets an "interrupted" flag on the target thread — it doesn't forcibly stop it. A well-behaved thread should periodically check `Thread.currentThread().isInterrupted()` (or catch `InterruptedException` if it's currently blocked in a wait/sleep) and gracefully stop what it's doing.
```java
while (!Thread.currentThread().isInterrupted()) {
    // do work
}
```

**Q18. What happens if a thread is sleeping and gets interrupted?**
`Thread.sleep()` throws `InterruptedException` immediately, and the "interrupted" flag is cleared. The sleeping code should catch this and decide how to respond — usually by stopping the loop or task cleanly.

**Q19. Why is calling `Thread.stop()` considered dangerous and deprecated?**
It forcibly kills a thread at any point, potentially in the middle of updating shared data — leaving that data in a corrupted, half-updated state, with no chance for cleanup. Modern code should use interruption (a cooperative "please stop" signal) instead of forceful termination.

**Q20. What does `Thread.yield()` do?**
A hint to the scheduler that the current thread is willing to give up its CPU time slot, letting other threads of equal priority run. It's only a hint — the scheduler is free to ignore it, and it's rarely used in modern code.

---

# SECTION 2: Race Conditions and `synchronized` (Q21–Q38)

## Basic

**Q21. What is a race condition?**
A situation where the correctness of a program depends on the unpredictable timing/order in which threads access shared data — leading to inconsistent or wrong results depending on how the threads happen to interleave.

**Q22. Why isn't `count++` thread-safe, even though it looks like one operation?**
Because it's actually three separate steps: read the current value, add 1, write the new value back. Two threads can both read the same value before either writes back, causing one increment to be lost.

**Q23. What does the `synchronized` keyword do?**
Ensures only one thread at a time can execute a synchronized block or method on a given lock object — every other thread trying to enter must wait until the current thread finishes and releases the lock.

**Q24. Write a thread-safe counter using `synchronized`.**
```java
class Counter {
    private int count = 0;
    public synchronized void increment() {
        count++;
    }
}
```

**Q25. What's the difference between synchronizing a method and synchronizing a block?**
A synchronized method locks on `this` (or the Class object, for static methods) for its entire duration. A synchronized block lets you choose exactly what to lock and for how long — generally preferred, since it reduces how long other threads have to wait.

## Intermediate

**Q26. What does every Java object automatically have that makes `synchronized` possible?**
Every object has a built-in "intrinsic lock" (also called a monitor lock), inherited from `Object`. `synchronized(someObject)` acquires that specific object's lock.

**Q27. What are the TWO guarantees `synchronized` provides?**
1. **Mutual exclusion** — only one thread runs the protected code at a time.
2. **Visibility** — when a thread exits a synchronized block, its changes to shared variables become visible to the next thread that enters a synchronized block on the same lock.

**Q28. What happens to a lock if an exception is thrown inside a synchronized block?**
The lock is still automatically released — `synchronized` guarantees release no matter how the block exits (normal completion or exception).

**Q29. Can two threads run two DIFFERENT synchronized methods on the SAME object at the same time?**
No — if both methods are synchronized on the same object (e.g., both are instance synchronized methods on the same object, locking on `this`), only one of them can run at a time, even though they're different methods, because they share the same lock.

**Q30. Can a thread that already holds a lock enter another synchronized block that uses the SAME lock?**
Yes — Java's intrinsic locks are **reentrant**, meaning a thread that already holds a lock can enter another synchronized section requiring that same lock without blocking itself. This is why it's sometimes called a "reentrant lock" behavior even for the basic `synchronized` keyword.

**Q31. What is a static synchronized method locking on?**
The `Class` object itself (e.g., `MyClass.class`), not on any particular instance — meaning it locks across ALL instances of that class, not just one object.

**Q32. Why is synchronizing on a String literal or a boxed value (like `Integer`) considered risky?**
String literals and small boxed values (like small Integers, due to caching) can be unintentionally **shared** across completely unrelated parts of code that happen to use the same literal value — causing unexpected, hidden lock contention (or worse, threads that think they're using separate locks actually sharing one). It's best to use a dedicated, private `Object` as a lock.

## Advanced

**Q33. Explain how `synchronized` blocks scale under heavy contention.**
As more threads compete for the same lock, more of them end up BLOCKED, waiting their turn — throughput can degrade significantly, since only one thread makes progress at a time on that critical section, no matter how many CPU cores are available.

**Q34. What is lock contention?**
The situation where multiple threads frequently try to acquire the same lock at the same time, causing many of them to wait — reducing the actual parallelism benefit you'd otherwise get from having multiple threads/cores.

**Q35. How would you reduce lock contention in a class with a single big synchronized method?**
Break the critical section down as small as possible (synchronize only what truly needs protection, not the whole method), or use separate locks for genuinely independent pieces of data (lock splitting/striping) instead of one single lock guarding everything.

**Q36. Is `synchronized` fair — does it guarantee the longest-waiting thread gets the lock next?**
No — `synchronized` provides no fairness guarantee at all. Any waiting thread could be granted the lock next, which can, in rare cases, lead to starvation for an unlucky thread.

**Q37. Can you use `synchronized` on a static method and an instance method to protect the same shared static field safely?**
No, not safely — a static synchronized method locks on the Class object, while an instance synchronized method locks on the specific object instance. These are two DIFFERENT locks, so both methods could still run at the same time, defeating the purpose of synchronizing access to a shared static field.

**Q38. What is the performance cost of `synchronized` when there's no actual contention (only one thread ever uses it)?**
Modern JVMs optimize this heavily (through mechanisms like biased locking and lock elision in some JVM versions) — uncontended synchronized blocks are quite cheap in practice, though still not entirely free compared to no synchronization at all.

---

# SECTION 3: `volatile` (Q39–Q48)

## Basic

**Q39. What problem does `volatile` solve?**
CPU cores each have their own local cache. Without `volatile`, a write by one thread might sit in that core's cache for a while before becoming visible to other cores/threads, which could keep reading a stale value. `volatile` forces every read and write to go directly through to main memory, guaranteeing visibility.

**Q40. Give a simple example of a correct use of `volatile`.**
```java
class Flag {
    private volatile boolean running = true;
    public void stop() { running = false; }
    public void doWork() {
        while (running) { /* work */ }
    }
}
```

## Intermediate

**Q41. Does `volatile` make `count++` thread-safe?**
No. `volatile` only guarantees visibility of the LATEST value — it does nothing to make a multi-step read-modify-write operation like `count++` atomic. Another thread can still interleave between the read and the write.

**Q42. If `volatile` doesn't fix `count++`, what should you use instead?**
Either wrap the increment in a `synchronized` block, or use `AtomicInteger`, which is specifically built for atomic single-variable read-modify-write operations.

**Q43. When IS `volatile` enough by itself, with no lock needed?**
When a variable is written by one thread and only read by others (not a read-modify-write pattern) — like a simple stop-flag or a single reference being swapped out.

**Q44. Does `volatile` prevent instruction reordering?**
Yes — the JVM/CPU is not allowed to reorder operations across a volatile read or write in ways that would break the guarantees `volatile` provides, unlike with normal (non-volatile) variables, which can be reordered more freely for optimization.

## Advanced

**Q45. Can `volatile` be used on an object reference to make the WHOLE object thread-safe?**
No — `volatile` on a reference only guarantees that the reference itself (which object it points to) is visible correctly across threads. It does NOT make the object's own internal fields thread-safe if multiple threads mutate those fields concurrently.

**Q46. What's the difference between `volatile` and `synchronized` in terms of what they guarantee?**
`volatile` only guarantees visibility for a single variable. `synchronized` guarantees BOTH visibility AND atomicity (mutual exclusion) for an entire block of code, potentially involving multiple variables.

**Q47. Is `volatile` faster than `synchronized`?**
Generally yes, since it doesn't involve acquiring/releasing a lock or blocking any thread — it's a lighter-weight mechanism, but it only solves the visibility problem, not the atomicity problem.

**Q48. Give a real, practical use case for `volatile` beyond a simple stop-flag.**
The double-checked locking pattern for lazy singleton initialization uses a `volatile` field to ensure that once an object is fully constructed by one thread, other threads see a fully-initialized object (not a partially-constructed one) due to how `volatile` interacts with instruction reordering guarantees.

---

# SECTION 4: `wait()`, `notify()`, `notifyAll()` (Q49–Q60)

## Basic

**Q49. What is the purpose of `wait()`, `notify()`, and `notifyAll()`?**
They let threads coordinate around a shared condition — one thread can pause (`wait()`) until some condition becomes true, and another thread can signal (`notify()`/`notifyAll()`) that the condition may have changed.

**Q50. What rule must be followed to call `wait()`, `notify()`, or `notifyAll()`?**
They must be called from inside a `synchronized` block, on the SAME object whose lock is held — calling them outside a synchronized context throws `IllegalMonitorStateException`.

**Q51. What does `wait()` actually do?**
It releases the lock on that object AND pauses the current thread, until another thread calls `notify()`/`notifyAll()` on that same object (or a timeout passes, if using the timed overload).

## Intermediate

**Q52. What is the difference between `notify()` and `notifyAll()`?**
`notify()` wakes up ONE arbitrary waiting thread. `notifyAll()` wakes up ALL waiting threads, which then compete to re-acquire the lock one at a time.

**Q53. Why should `wait()` always be called inside a `while` loop, never an `if`?**
Because of "spurious wakeups" — the JVM is technically allowed to wake a waiting thread even without a real `notify()`. A `while` loop re-checks the condition after waking, and calls `wait()` again if it's still not satisfied; an `if` would incorrectly let the thread proceed.

**Q54. Why is `notifyAll()` usually safer than `notify()`?**
If multiple threads are waiting on the same object for DIFFERENT reasons, `notify()` might wake the "wrong" thread (whose condition isn't actually satisfied yet), potentially causing a missed signal or a deadlock-like freeze. `notifyAll()` wakes everyone to re-check, avoiding this at a small performance cost.

**Q55. Write a simple producer-consumer example using `wait()`/`notify()`.**
```java
class Buffer {
    private final List<Integer> data = new ArrayList<>();
    private final int CAPACITY = 5;

    public synchronized void produce(int value) throws InterruptedException {
        while (data.size() == CAPACITY) wait();
        data.add(value);
        notifyAll();
    }

    public synchronized int consume() throws InterruptedException {
        while (data.isEmpty()) wait();
        int value = data.remove(0);
        notifyAll();
        return value;
    }
}
```

## Advanced

**Q56. What is the difference between `wait()` and `sleep()`?**
`Thread.sleep()` pauses the current thread WITHOUT releasing any locks it holds. `wait()` RELEASES the lock it's called on, letting other threads proceed, and only resumes once notified (or a timeout passes).

**Q57. Can `wait()` be called without a timeout, and what's the risk?**
Yes, `wait()` with no arguments waits indefinitely. The risk: if the notifying thread's `notify()`/`notifyAll()` call happens to arrive before this thread starts waiting (a timing edge case), or is simply never called due to a bug, the waiting thread could be stuck forever — this is why timed waits or careful design are often preferred in production code.

**Q58. What happens if `notify()` is called but no thread is currently waiting?**
Nothing happens — the notification is simply lost; it doesn't get "saved" for a future thread that starts waiting later. This is a common source of subtle bugs if a producer notifies before a consumer has started waiting.

**Q59. Why do modern codebases often avoid hand-writing `wait()`/`notify()` logic?**
It's easy to get subtly wrong (forgetting the while-loop check, calling outside synchronized, missing a notify), and higher-level tools like `BlockingQueue`, `CountDownLatch`, and `CyclicBarrier` handle this coordination correctly and more simply, without needing to reason about spurious wakeups or missed signals manually.

**Q60. Can you use `wait()`/`notify()` with a `ReentrantLock` instead of a plain object's monitor?**
Not directly — `ReentrantLock` uses `Condition` objects (via `lock.newCondition()`) instead, which offer the equivalent of `wait()`/`notify()`/`notifyAll()` (called `await()`, `signal()`, `signalAll()`), but tied to an explicit `Lock` rather than an object's built-in intrinsic monitor.

---

# SECTION 5: The Java Memory Model & Happens-Before (Q61–Q70)

## Basic

**Q61. What is the Java Memory Model (JMM)?**
The official rulebook defining when writes by one thread are guaranteed to become visible to reads by another thread, and what reorderings the compiler/CPU are allowed to perform without breaking a program's correctness.

**Q62. Why does Java need a memory model at all?**
Without clear, defined rules, different CPU architectures and JVM implementations could behave differently and unpredictably with shared-memory concurrent code — the JMM gives developers a consistent, guaranteed contract to write against.

## Intermediate

**Q63. What is "happens-before"?**
A guarantee: if action A happens-before action B, then A's effects (its writes to memory) are guaranteed to be visible to B, and the two actions can't be reordered relative to each other in a way that would violate this.

**Q64. Name at least three specific happens-before rules.**
1. Program order: within one thread, earlier actions happen-before later actions in that same thread.
2. Monitor lock rule: unlocking a synchronized block happens-before a later thread locking that same object.
3. Volatile rule: a write to a volatile variable happens-before a later read of that same variable.
4. Thread start rule: everything before `thread.start()` happens-before anything the new thread does.
5. Thread join rule: everything a thread does happens-before another thread returns from `join()` on it.

**Q65. If Thread A writes to a plain (non-volatile, non-synchronized) variable and Thread B reads it right after, is B guaranteed to see the new value?**
Not necessarily — without an established happens-before relationship (via synchronized, volatile, or another JMM mechanism) connecting that specific write to that specific read, there is no guarantee at all.

## Advanced

**Q66. How does `synchronized` establish a happens-before relationship?**
Through the monitor lock rule — releasing (unlocking) a synchronized block happens-before another thread later acquiring (locking) the SAME object's monitor, guaranteeing that all changes made inside the first thread's synchronized block are visible to the second.

**Q67. Does calling `Thread.start()` establish any visibility guarantee?**
Yes — everything the starting thread did BEFORE calling `start()` is guaranteed visible to the new thread once it begins running, per the thread-start happens-before rule.

**Q68. What does the "thread join rule" guarantee, in practical terms?**
Once `thread.join()` returns successfully in the calling thread, everything the joined thread did during its execution is guaranteed to be visible to the calling thread — useful for safely reading results a worker thread produced, after confirming it has finished.

**Q69. Can the compiler or CPU reorder instructions around a synchronized block?**
Not in a way that would violate the JMM's guarantees — reordering across the boundary of a lock acquire/release is restricted specifically to preserve the mutual exclusion and visibility guarantees `synchronized` provides.

**Q70. Why is understanding the JMM important even if you never write low-level concurrency code directly?**
Because it explains WHY tools like `synchronized`, `volatile`, and `java.util.concurrent` classes actually work correctly — without this foundation, concurrency code can look correct on the surface (and even "work" in casual testing) while still having subtle, hard-to-reproduce visibility bugs that only show up under specific timing conditions in production.

---

# SECTION 6: Deadlock, Livelock, and Starvation (Q71–Q84)

## Basic

**Q71. What is a deadlock?**
A situation where two or more threads are stuck forever, each waiting for a lock that another one of them holds — none of them can ever proceed.

**Q72. Give a simple two-thread deadlock example.**
```java
// Thread 1: synchronized(lockA) { synchronized(lockB) { ... } }
// Thread 2: synchronized(lockB) { synchronized(lockA) { ... } }
```
If Thread 1 grabs `lockA` at the same moment Thread 2 grabs `lockB`, each ends up waiting forever for the lock the other one holds.

**Q73. What is livelock?**
A situation where threads are NOT blocked — they're actively running and responding to each other — but they make no real progress, because their actions keep canceling each other out in a repeating pattern.

**Q74. What is starvation?**
A situation where a thread is repeatedly denied access to a resource it needs, simply because other threads keep "winning" and getting there first — not due to a hard deadlock, just consistently unlucky/unfair scheduling.

## Intermediate

**Q75. What are the four conditions that must ALL be true for a deadlock to occur?**
1. Mutual exclusion — a resource can only be held by one thread at a time.
2. Hold and wait — a thread holds one resource while waiting for another.
3. No preemption — a resource can't be forcibly taken from a thread; it must be released voluntarily.
4. Circular wait — a cycle of threads, each waiting for a resource held by the next one in the cycle.

**Q76. How do you prevent deadlock by breaking the circular wait condition?**
Always acquire multiple locks in the same, fixed global order everywhere in your code — if every thread always locks `lockA` before `lockB` (never the reverse), a circular waiting pattern can't form.

**Q77. What is a simple analogy for livelock?**
Two people in a narrow hallway, both stepping aside to let the other pass, but both step to the same side at the same time, repeatedly — actively trying to be polite, but never actually making progress.

**Q78. How is livelock typically caused in real code?**
Often by poorly designed "detect conflict and retry" logic, where two threads both detect a conflict and both immediately retry with the exact same strategy/timing, so they keep colliding in the same way forever.

## Advanced

**Q79. How can `tryLock()` help prevent deadlock?**
Instead of blocking indefinitely waiting for a lock, `tryLock()` (optionally with a timeout) lets a thread attempt to acquire a lock and immediately know if it failed — allowing it to back off, release any locks it already holds, and retry later, rather than getting stuck holding one lock while waiting forever for another.

**Q80. How does using a fair lock help prevent starvation?**
A `ReentrantLock` constructed with `fair = true` grants the lock to whichever thread has been waiting the longest, rather than an arbitrary one — preventing any single thread from being perpetually skipped over, at some cost to overall throughput.

**Q81. How would you detect a deadlock in a running Java application?**
Using a thread dump (e.g., via `jstack`, or a profiling tool) — the JVM can detect certain deadlock cycles among threads holding intrinsic locks and reports them directly in the thread dump output, showing exactly which threads are stuck waiting on which locks.

**Q82. What's a real-world example of how deadlock might occur in a banking application?**
Transferring money between two accounts, where the transfer method locks the "from" account first, then the "to" account. If Transaction A transfers from Account X to Account Y (locking X then Y) at the same time Transaction B transfers from Account Y to Account X (locking Y then X), they can deadlock. Fix: always lock accounts in a consistent order (e.g., by account ID, lowest first) regardless of transfer direction.

**Q83. Is livelock easier or harder to detect than deadlock, and why?**
Generally harder — deadlocked threads show up clearly as BLOCKED or WAITING in a thread dump, sitting motionless. Livelocked threads appear to be actively running (high CPU usage even), making it look like the application is "doing something," even though no real progress is happening — this can be more confusing to diagnose.

**Q84. Besides consistent lock ordering, name another general strategy to reduce deadlock risk.**
Minimize the number of locks held simultaneously, and keep the time any lock is held as short as possible — the less overlap between locks, the smaller the chance of forming a circular wait.

---

# SECTION 7: ExecutorService and Thread Pools (Q85–Q100)

## Basic

**Q85. What problem does a thread pool solve?**
Creating a brand-new OS thread for every task is expensive (real creation/destruction overhead), and unlimited thread creation under heavy load can exhaust system resources. A thread pool reuses a fixed (or bounded) set of worker threads instead.

**Q86. How do you create a basic thread pool and submit a task to it?**
```java
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> System.out.println("Task running"));
executor.shutdown();
```

**Q87. What does `executor.shutdown()` do?**
Tells the pool to stop accepting new tasks, but lets currently running and already-queued tasks finish normally before the pool actually stops.

**Q88. What's the difference between `shutdown()` and `shutdownNow()`?**
`shutdown()` is graceful — finishes existing/queued tasks first. `shutdownNow()` is aggressive — attempts to stop currently running tasks immediately (via interruption) and returns the list of tasks that were still waiting in the queue, never started.

## Intermediate

**Q89. What are the common factory methods on `Executors`?**
`newFixedThreadPool(n)` (fixed size, unbounded queue), `newCachedThreadPool()` (grows as needed, reuses idle threads, unbounded), `newSingleThreadExecutor()` (exactly one thread, sequential execution), `newScheduledThreadPool(n)` (supports delayed/periodic tasks).

**Q90. What is a risk with `newCachedThreadPool()` under heavy load?**
It can create an unlimited number of threads if tasks arrive faster than they finish, since there's no cap — this can exhaust memory or other system resources under sustained heavy load.

**Q91. Why might a production system prefer constructing `ThreadPoolExecutor` directly instead of using `Executors` factory methods?**
`Executors`' convenient defaults can hide unbounded queues or unbounded thread growth, which can cause `OutOfMemoryError` under load. Constructing `ThreadPoolExecutor` directly lets you explicitly set a bounded queue size and a rejection policy (what happens when the pool is completely overwhelmed).

**Q92. What is `awaitTermination()` used for?**
Blocks the calling thread, waiting up to a given timeout for the executor to finish shutting down (after `shutdown()` was called) — useful to know for certain that all tasks have genuinely completed before proceeding.
```java
executor.shutdown();
executor.awaitTermination(5, TimeUnit.SECONDS);
```

**Q93. What happens if you submit a task to an executor after calling `shutdown()`?**
It's rejected — typically throwing a `RejectedExecutionException`, since the pool is no longer accepting new work.

## Advanced

**Q94. What are the core parameters of a `ThreadPoolExecutor` constructed directly?**
Core pool size (minimum number of threads kept alive), maximum pool size, keep-alive time (how long idle extra threads survive beyond the core size before being removed), a work queue, and a rejection handler policy for when the pool and queue are both full.

**Q95. What are the built-in rejection policies for a `ThreadPoolExecutor`?**
`AbortPolicy` (default — throws an exception), `CallerRunsPolicy` (runs the rejected task on the calling thread itself, providing natural backpressure), `DiscardPolicy` (silently drops the task), and `DiscardOldestPolicy` (drops the oldest queued task to make room for the new one).

**Q96. What is `CallerRunsPolicy`, and why is it a useful backpressure mechanism?**
When the pool is overwhelmed, instead of rejecting or discarding, it runs the new task on whichever thread SUBMITTED it — this naturally slows down the submitting thread (since it's now busy doing the work itself), which indirectly throttles how fast new work gets submitted, giving the pool time to catch up.

**Q97. What happens to unfinished, in-progress work if the JVM crashes while an ExecutorService is running?**
It's lost — thread pools and their in-flight tasks live entirely in memory and don't persist across JVM restarts; any durability requirements need to be handled separately (e.g., a persistent task queue backed by a database or message broker).

**Q98. Why is it important to always call `shutdown()` on an ExecutorService you created?**
If never shut down, the pool's threads (typically non-daemon) stay alive indefinitely, which can prevent the JVM from exiting cleanly even after your main application logic has finished.

**Q99. What is `ScheduledExecutorService` used for?**
Running tasks after a delay, or repeatedly at a fixed rate/interval — via methods like `schedule()`, `scheduleAtFixedRate()`, and `scheduleWithFixedDelay()`.

**Q100. What's the difference between `scheduleAtFixedRate()` and `scheduleWithFixedDelay()`?**
`scheduleAtFixedRate()` tries to start each execution at fixed time intervals, regardless of how long the previous execution took (executions could overlap or run back-to-back if one runs long). `scheduleWithFixedDelay()` waits for the fixed delay to pass AFTER the previous execution finishes, before starting the next one — guaranteeing no overlap.

---

# SECTION 8: Callable, Future, and CompletableFuture (Q101–Q118)

## Basic

**Q101. What's the difference between `Runnable` and `Callable`?**
`Runnable.run()` returns nothing and can't throw checked exceptions. `Callable<V>.call()` returns a value of type `V` and CAN throw checked exceptions.

**Q102. How do you submit a `Callable` to an executor and get its result?**
```java
Callable<Integer> task = () -> 42;
Future<Integer> future = executor.submit(task);
Integer result = future.get(); // blocks until the result is ready
```

**Q103. What does `Future.get()` do, and what's the main limitation of `Future`?**
It blocks the calling thread until the async task's result is ready. The limitation: there's no built-in way to react automatically once the result is ready, or to chain further async steps — you either block and wait, or manually poll `isDone()`.

## Intermediate

**Q104. What is `CompletableFuture`, and how does it improve on `Future`?**
It represents an eventual result, but supports non-blocking composition — chaining what should happen next (`thenApply`, `thenCompose`) without needing to block and wait in the middle of your code.

**Q105. How do you create a CompletableFuture that computes something asynchronously?**
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "result");
```

**Q106. What's the difference between `runAsync()` and `supplyAsync()`?**
`runAsync(Runnable)` runs a task that returns nothing, producing `CompletableFuture<Void>`. `supplyAsync(Supplier<T>)` runs a task that returns a value, producing `CompletableFuture<T>`.

**Q107. What does `thenApply()` do?**
Transforms the result once it's ready, similar to Stream's `map()`, and returns a new CompletableFuture wrapping the transformed value.
```java
future.thenApply(result -> result.toUpperCase());
```

**Q108. What does `thenAccept()` do, and how is it different from `thenApply()`?**
Consumes the result (a side effect, like printing) without producing a further value — like Stream's `forEach()`, whereas `thenApply()` is like `map()`.

**Q109. What does `thenCompose()` do, and when should you use it over `thenApply()`?**
Use it when your next step ITSELF returns a CompletableFuture — `thenApply()` would produce an awkward nested `CompletableFuture<CompletableFuture<T>>`, while `thenCompose()` flattens it to a single level, just like `flatMap()` in Streams.

**Q110. How do you handle an exception in a CompletableFuture chain?**
```java
future.exceptionally(error -> {
    System.out.println("Error: " + error.getMessage());
    return "fallback";
});
```

## Advanced

**Q111. What does `CompletableFuture.allOf()` do?**
Returns a new `CompletableFuture<Void>` that completes only once ALL the given futures have completed.

**Q112. What does `CompletableFuture.anyOf()` do?**
Returns a new `CompletableFuture` that completes as soon as ANY ONE of the given futures completes.

**Q113. What is `thenCombine()` used for?**
Combining the results of two INDEPENDENT CompletableFutures once both are done, using a given combining function.
```java
future1.thenCombine(future2, (r1, r2) -> r1 + r2);
```

**Q114. Which thread pool do `supplyAsync()`/`thenApply()` use by default, and why can this be a problem in production?**
By default, the shared, application-wide common `ForkJoinPool`. If many unrelated parts of an application share this same pool — especially with long-running or blocking tasks — they can starve each other. It's common to pass a dedicated `Executor` explicitly.

**Q115. What's the difference between `future.get()` and `future.join()`?**
Both block for the result. `get()` throws checked exceptions (`InterruptedException`, `ExecutionException`). `join()` throws an unchecked `CompletionException` instead — more convenient inside lambda chains where declaring checked exceptions is cumbersome.

**Q116. How would you run two independent async calls in parallel and combine their results once both finish?**
```java
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> callServiceA());
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> callServiceB());
String combined = f1.thenCombine(f2, (a, b) -> a + " | " + b).join();
```

**Q117. Can a `Future` (the plain, non-Completable version) be cancelled? What does that mean in practice?**
Yes, via `future.cancel(boolean mayInterruptIfRunning)`. If the task hasn't started yet, it simply won't run. If it's already running and `mayInterruptIfRunning` is true, the executing thread is interrupted (though the task must cooperate by checking its interrupted status to actually stop).

**Q118. What's the benefit of `CompletableFuture.supplyAsync(supplier, executor)` over the no-executor overload, specifically for I/O-bound tasks?**
I/O-bound tasks (like a slow network call) block their thread while waiting, which is wasteful on the shared common pool's limited thread count. Using a dedicated executor (often with a larger, appropriately-sized thread pool for I/O-bound work) prevents these blocking tasks from starving CPU-bound work that also relies on the common pool.

---

# SECTION 9: Locks — ReentrantLock and ReentrantReadWriteLock (Q119–Q132)

## Basic

**Q119. Why does `ReentrantLock` exist, given that `synchronized` already provides mutual exclusion?**
It offers capabilities `synchronized` doesn't have: `tryLock()` (non-blocking attempt), `lockInterruptibly()` (can respond to interrupts while waiting), a fairness policy option, and multiple `Condition` objects per lock.

**Q120. What is the correct pattern for using a `ReentrantLock`?**
```java
lock.lock();
try {
    // critical section
} finally {
    lock.unlock(); // MUST be here — forgetting it leaks the lock forever
}
```

**Q121. What is the single biggest practical risk of using `ReentrantLock` compared to `synchronized`?**
`synchronized` releases its lock automatically no matter how the block exits. `ReentrantLock` requires manual unlocking — if you forget the `finally { lock.unlock(); }` and an exception is thrown, the lock is never released, blocking every other thread waiting for it forever.

## Intermediate

**Q122. What does `tryLock()` do, and why is it useful?**
Attempts to acquire the lock without blocking indefinitely — returns `true`/`false` immediately (or after an optional timeout), letting a thread back off and do something else instead of waiting forever, which is useful for avoiding deadlock.
```java
if (lock.tryLock()) {
    try { /* got it */ } finally { lock.unlock(); }
} else {
    // couldn't get it — do something else
}
```

**Q123. What does `lockInterruptibly()` do?**
Lets a thread waiting to acquire the lock be interrupted (woken up early) by another thread, instead of being stuck waiting no matter what — useful for cancellable operations.

**Q124. What is a fair lock, and how do you create one?**
```java
Lock fairLock = new ReentrantLock(true);
```
A fair lock grants access to whichever thread has been waiting the longest, preventing starvation, at some cost to overall throughput compared to the default unfair mode.

**Q125. What are `Condition` objects, and why are they useful?**
Created via `lock.newCondition()`, they let a single `Lock` have MULTIPLE independent wait-sets — e.g., separate "queue not full" and "queue not empty" conditions for a bounded buffer, so you can `signal()` exactly the right group of waiting threads instead of waking everyone with a single shared wait-set.

**Q126. What is `ReentrantReadWriteLock`, and what problem does it solve?**
It solves the problem of a plain lock forcing even simultaneous READERS to take turns one at a time, which is wasteful for read-heavy, write-rare data. It provides a shared read lock (many readers at once) and an exclusive write lock (one writer, no readers, at a time).

**Q127. Write basic usage of `ReentrantReadWriteLock`.**
```java
ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
rwLock.readLock().lock();
try { /* read */ } finally { rwLock.readLock().unlock(); }

rwLock.writeLock().lock();
try { /* write */ } finally { rwLock.writeLock().unlock(); }
```

## Advanced

**Q128. Is `ReentrantLock` reentrant — can the same thread acquire it multiple times?**
Yes, exactly as the name suggests — a thread already holding the lock can acquire it again (e.g., in a recursive call) without blocking itself, but it must call `unlock()` the same number of times it called `lock()` to fully release it.

**Q129. Can a thread that holds the write lock also acquire the read lock, in `ReentrantReadWriteLock`?**
Yes — this is called "lock downgrading," and it's explicitly supported: a thread can acquire the read lock while still holding the write lock, then release the write lock, leaving itself with just the read lock. The reverse (upgrading from read to write while holding the read lock) is NOT supported and can cause a deadlock.

**Q130. What happens if multiple threads want the write lock while readers are already active in `ReentrantReadWriteLock`?**
The write lock request will block until all current readers release their read locks — but implementations often also prevent NEW readers from jumping in once a writer is waiting, to avoid writer starvation under continuous heavy read load (behavior can depend on fairness settings).

**Q131. Why might `synchronized` still be preferred over `ReentrantLock` in simple cases?**
`synchronized` is simpler (automatic lock release, less boilerplate, fewer ways to make a mistake) and the JVM has heavily optimized it over the years. `ReentrantLock` is worth the added complexity specifically when you need its extra features (tryLock, fairness, interruptibility, multiple Conditions) — not as a default replacement for every synchronized block.

**Q132. Can `StampedLock` (a more advanced lock introduced in Java 8) do something `ReentrantReadWriteLock` can't?**
Yes — `StampedLock` adds an "optimistic read" mode, which doesn't block writers at all; a reader can attempt a read optimistically and then validate afterward whether a write happened in the meantime, retrying as a normal read lock only if needed. This can offer better throughput than `ReentrantReadWriteLock` under very read-heavy, low-contention conditions, though it's more complex and doesn't support reentrancy the same way.

---

# SECTION 10: CountDownLatch, CyclicBarrier, and Semaphore (Q133–Q145)

## Basic

**Q133. What is `CountDownLatch`?**
A one-time-use synchronization tool: initialized with a count, threads can call `await()` to block until the count reaches zero, with other threads calling `countDown()` to decrease it.

**Q134. Write an example using CountDownLatch to wait for 3 worker threads to finish.**
```java
CountDownLatch latch = new CountDownLatch(3);
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        doWork();
        latch.countDown();
    }).start();
}
latch.await(); // main thread waits here until all 3 call countDown()
```

**Q135. Can a CountDownLatch be reused after its count reaches zero?**
No — once it hits zero, it's permanently "used up." For repeated synchronization rounds, use `CyclicBarrier` instead.

## Intermediate

**Q136. What is `CyclicBarrier`, and how is it different from `CountDownLatch`?**
It makes a FIXED group of threads all wait for each other at a common point — none proceed until all have arrived. Unlike CountDownLatch, once released, it automatically RESETS and can be reused for further rounds.

**Q137. Write a basic example of `CyclicBarrier`.**
```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> System.out.println("All arrived!"));
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        doWork();
        try { barrier.await(); } catch (Exception e) {}
    }).start();
}
```

**Q138. What is the optional second constructor argument to `CyclicBarrier` used for?**
A `Runnable` that automatically runs once, right when the last thread arrives and the barrier is triggered — before any of the waiting threads are released to continue.

**Q139. What is `Semaphore`, and what does it manage?**
Maintains a fixed number of "permits." Threads call `acquire()` to take a permit (blocking if none available) and `release()` to give one back — used to limit how many threads can access a resource concurrently to some chosen number N.

**Q140. Write a basic example limiting concurrent access with a Semaphore.**
```java
Semaphore semaphore = new Semaphore(3); // at most 3 threads at once
semaphore.acquire();
try {
    // limited resource access
} finally {
    semaphore.release();
}
```

## Advanced

**Q141. When would you choose CyclicBarrier over CountDownLatch?**
When you have a fixed group of threads that need to work in repeated, synchronized "rounds" — where every thread must complete step N before any of them starts step N+1 — rather than a single, one-time "wait for these things to finish."

**Q142. What's the difference between a Semaphore with 1 permit and a Lock?**
Conceptually similar (only one thread "in" at a time), but a Semaphore has no concept of "ownership" — any thread can call `release()`, not necessarily the same thread that called `acquire()`. A Lock generally must be released by the thread that acquired it.

**Q143. Can a Semaphore be used to implement a simple rate limiter or connection pool cap?**
Yes — a very common real-world use case is limiting concurrent connections to a database or external API to some fixed maximum, so the application doesn't overwhelm that external resource even under heavy internal concurrent load.

**Q144. What happens if `CyclicBarrier.await()` times out or a thread is interrupted while waiting at the barrier?**
The barrier is considered "broken" — all other threads currently waiting at that barrier will also be released with a `BrokenBarrierException`, since a proper synchronized "round" can no longer be guaranteed with a missing participant.

**Q145. Can you decrease a CountDownLatch's count by more than 1 with a single call?**
No — `countDown()` only decrements by exactly 1 per call; if you need it decremented by 3, you must call it three separate times (from the same or different threads).

---

# SECTION 11: BlockingQueue (Q146–Q155)

## Basic

**Q146. What is `BlockingQueue`, and what makes it different from a regular `Queue`?**
A `Queue` where `put()` automatically blocks the calling thread if the queue is full, and `take()` automatically blocks if the queue is empty — instead of throwing an exception or returning immediately like a normal Queue's `add()`/`remove()` would.

**Q147. Write a simple producer-consumer example using BlockingQueue.**
```java
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10);

// Producer
new Thread(() -> {
    try { queue.put(1); } catch (InterruptedException e) {}
}).start();

// Consumer
new Thread(() -> {
    try { int value = queue.take(); } catch (InterruptedException e) {}
}).start();
```

## Intermediate

**Q148. Why is BlockingQueue usually better than hand-writing wait/notify producer-consumer logic?**
It handles all the internal wait/notify-style coordination correctly and safely for you, dramatically reducing the risk of subtle bugs (like forgetting the while-loop check, or a missed notify) that come with manual implementations.

**Q149. What is `ArrayBlockingQueue`?**
A bounded, fixed-capacity BlockingQueue backed by an array — capacity must be specified at creation and cannot grow.

**Q150. What is `LinkedBlockingQueue`, and how does its default capacity compare to `ArrayBlockingQueue`?**
Backed by linked nodes, optionally bounded — if you don't specify a capacity, it's effectively unbounded, which can be risky in production if producers consistently outpace consumers (unbounded memory growth).

**Q151. What is `PriorityBlockingQueue`?**
An unbounded BlockingQueue where elements come out in priority order (like `PriorityQueue`) rather than FIFO order.

## Advanced

**Q152. What is `SynchronousQueue`, and how is it different from other BlockingQueue implementations?**
It holds ZERO elements internally — every `put()` must be matched by a simultaneous `take()` from another thread, acting as a direct hand-off between exactly one producer and one consumer at a time, with nothing ever actually "stored" in between.

**Q153. What happens if you call `offer()` (instead of `put()`) on a full BlockingQueue?**
`offer()` returns `false` immediately instead of blocking — useful when you want to know right away whether the insertion succeeded, rather than waiting.

**Q154. How would you implement a simple thread pool's internal task queue using BlockingQueue?**
A `ThreadPoolExecutor` internally uses a BlockingQueue (like `LinkedBlockingQueue` or `ArrayBlockingQueue`) to hold submitted tasks waiting to be picked up by an available worker thread — worker threads repeatedly call `queue.take()` to grab the next task, blocking naturally when there's no work available.

**Q155. Why would you choose a bounded BlockingQueue over an unbounded one in a production task queue?**
A bounded queue provides natural backpressure — if producers are creating work faster than it can be consumed, the queue fills up and `put()` blocks (or `offer()` fails), signaling the problem immediately, rather than silently growing without limit and risking an eventual `OutOfMemoryError`.

---

# SECTION 12: Atomic Classes and CAS (Q156–Q168)

## Basic

**Q156. What does CAS (Compare-And-Swap) mean?**
A single, atomic CPU instruction: "compare the current value in a memory location to an expected value; if they match, swap in a new value; if not, report failure" — all as one indivisible hardware-level step.

**Q157. What is `AtomicInteger`, and how is it different from a plain `int` guarded by `synchronized`?**
A wrapper class providing atomic, LOCK-FREE read-modify-write operations on a single integer value, built internally using CAS — achieving thread safety without ever blocking a thread, unlike `synchronized`.

**Q158. Write a thread-safe counter using AtomicInteger.**
```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet(); // atomically adds 1, thread-safe, no lock needed
```

## Intermediate

**Q159. Explain the step-by-step process CAS uses to safely update a shared value.**
1. Read the current value.
2. Compute the new value based on it.
3. Attempt a CAS: "if the value is still what I read in step 1, swap it to my computed new value."
4. If it succeeds, done. If it fails (someone else changed it in between), retry from step 1.

**Q160. Why is lock-free (CAS-based) code often faster than locking under low-to-moderate contention?**
No thread ever has to block/sleep waiting for a lock to free up — at worst, a thread just retries its CAS attempt a couple of times, avoiding the overhead of thread scheduling and context switches that blocking involves.

**Q161. What methods does `AtomicInteger` provide besides `incrementAndGet()`?**
`get()`, `set()`, `getAndIncrement()` (returns old value, then increments), `getAndAdd(n)`, `compareAndSet(expected, newValue)`, and more — covering common atomic read-modify-write patterns.

**Q162. What is `AtomicReference`, and when would you use it?**
An atomic wrapper for any object reference (not just numbers) — useful for atomically swapping out an entire object (like updating an immutable configuration object) without locking.

## Advanced

**Q163. What is the ABA problem in CAS-based concurrency?**
CAS only checks if the current value still equals the expected value. If a value changes from A → B → A before your CAS runs, the CAS succeeds (since it IS "A" again) — but something DID change in between, which could cause subtle bugs in certain data structures (like lock-free linked lists) that assumed nothing happened.

**Q164. How does `AtomicStampedReference` solve the ABA problem?**
It pairs the value with an integer "stamp" (version number) that increments on every change — so even if the value goes A→B→A, the stamp will differ, letting a CAS that checks BOTH value and stamp correctly detect the intermediate change.

**Q165. Why do Atomic classes not scale infinitely under very high contention?**
Under heavy contention, many threads' CAS attempts keep failing and retrying repeatedly (since many threads are all trying to update the same value at once), which can waste CPU cycles similarly to (or in extreme cases, worse than) lock-based contention — CAS shines most under low-to-moderate contention, not universally.

**Q166. What is `LongAdder`, and how does it differ from `AtomicLong` under high contention?**
`LongAdder` (added in Java 8) is designed for high-contention counting scenarios — internally, it splits the counter across multiple internal cells (striping), so different threads can update different cells concurrently with less CAS contention, then sums them all when you need the total. It scales better than a single `AtomicLong` when many threads are incrementing very frequently, though `get()`/`sum()` is slightly more expensive.

**Q167. Is optimistic locking (CAS) always better than pessimistic locking (synchronized/Lock)?**
No — it depends on contention level. Optimistic (CAS) approaches assume conflicts are rare and scale well under low contention, avoiding blocking overhead entirely. Pessimistic locking can perform better under very high contention, where CAS would otherwise waste a lot of effort on repeated failed retries.

**Q168. Can `compareAndSet()` be used to implement a simple non-blocking flag or state machine?**
Yes — a common pattern is using `AtomicReference<State>` or `AtomicInteger` representing enum-like states, and using `compareAndSet(oldState, newState)` to safely transition between states only if no other thread has already changed it, without needing any locks.

---

# SECTION 13: ThreadLocal (Q169–Q178)

## Basic

**Q169. What does `ThreadLocal` provide?**
A separate, independent, private copy of a variable for EACH thread — even though the `ThreadLocal` object itself is shared, each thread's `.get()`/`.set()` calls only affect its own isolated copy, invisible to other threads.

**Q170. Write a simple example of using ThreadLocal.**
```java
ThreadLocal<Integer> threadLocalValue = ThreadLocal.withInitial(() -> 0);
threadLocalValue.set(100); // only affects the CURRENT thread's copy
```

## Intermediate

**Q171. Give a practical use case for ThreadLocal.**
Giving each thread its own instance of a non-thread-safe class like `SimpleDateFormat`, avoiding the need for synchronization entirely — or storing request-scoped context (like a current user ID) accessible anywhere in that thread's call chain, without passing it as a parameter through every method.

**Q172. Why is `SimpleDateFormat` a commonly cited example for ThreadLocal?**
Because `SimpleDateFormat` is famously NOT thread-safe if shared across threads — giving each thread its own private instance via ThreadLocal sidesteps the problem entirely, without needing any locking at all.

**Q173. What is the risk of using ThreadLocal in a thread pool, specifically?**
Pooled threads are REUSED across many different, unrelated tasks over time, and are never actually destroyed between tasks. If you `set()` a ThreadLocal value during one task but never call `remove()`, that value stays attached to the pooled thread — causing a memory leak, and potentially leaking stale data into a completely unrelated, later task that happens to run on that same reused thread.

## Advanced

**Q174. How do you correctly clean up a ThreadLocal to avoid this leak?**
Always call `.remove()` in a `finally` block, right after you're done using it — especially critical in any code that might run on a pooled/reused thread.
```java
try {
    threadLocalValue.set(someValue);
    // ... use it ...
} finally {
    threadLocalValue.remove();
}
```

**Q175. Why is ThreadLocal less risky with manually-created, short-lived threads compared to pooled threads?**
Manually created threads are typically used once and then discarded — there's no "next task" to accidentally inherit stale leftover data, unlike pooled threads that are reused indefinitely across many unrelated pieces of work.

**Q176. What is `InheritableThreadLocal`, and how does it differ from `ThreadLocal`?**
It allows a CHILD thread (created by a parent thread) to inherit the parent's ThreadLocal value at the moment the child is created — a plain `ThreadLocal` does not automatically propagate its value to any new threads spawned from the current one.

**Q177. How does a web framework commonly use ThreadLocal, and what's the typical lifecycle?**
To store request-scoped context (like the current HTTP request, current user, or a request ID) — set once at the very start of handling a request (often in a filter/interceptor), read anywhere during that request's processing, and explicitly removed at the very end of request handling, to prevent leaking into whatever task the pooled thread picks up next.

**Q178. Does ThreadLocal help with sharing data BETWEEN threads?**
No — it's the exact opposite purpose. ThreadLocal is specifically for ISOLATING data so each thread has its own private copy; it does not provide any mechanism for one thread to share or communicate data to another thread.

---

# SECTION 14: ForkJoinPool and Parallel Streams (Q179–Q188)

## Basic

**Q179. What kind of problem is `ForkJoinPool` designed for?**
Divide-and-conquer problems — where a big task splits into smaller subtasks recursively, until small enough to solve directly, and results get combined back together (like parallel merge sort, or summing a huge array).

**Q180. What is "work-stealing," in simple terms?**
Each worker thread has its own task queue. If a thread finishes its own queued work early, instead of sitting idle, it "steals" a task from the back of another, still-busy thread's queue — keeping all cores continuously busy without any central coordinator.

## Intermediate

**Q181. What are `RecursiveTask` and `RecursiveAction`, and how do they differ?**
Both are base classes for ForkJoinPool tasks. `RecursiveTask<V>` is for tasks that RETURN a result. `RecursiveAction` is for tasks that don't return anything (like `Runnable`, but for the fork/join framework).

**Q182. How does `parallelStream()` relate to ForkJoinPool?**
`parallelStream()` uses this exact same ForkJoinPool mechanism internally (specifically the shared "common pool" by default) — automatically splitting the collection into chunks, processing them across worker threads with work-stealing, and merging results.

**Q183. When are parallel streams a good fit, and when should you avoid them?**
Good fit: large datasets, computationally expensive per-element work, stateless/independent operations. Avoid: small datasets (splitting/merging overhead outweighs benefit), shared mutable state (race conditions), or I/O-bound work (parallel streams don't speed up waiting on network/disk).

**Q184. Why is this code dangerous?**
```java
List<Integer> results = new ArrayList<>();
numbers.parallelStream().forEach(n -> results.add(n * 2));
```
`ArrayList` isn't thread-safe — multiple threads calling `add()` concurrently (as happens across a parallel stream's worker threads) can corrupt its internal state or lose elements. Fix: use `.collect(Collectors.toList())` instead, letting the stream API handle thread-safe aggregation.

## Advanced

**Q185. Does `forEach()` on a parallel stream guarantee processing in encounter order?**
No. Use `forEachOrdered()` to force order, though this gives up some parallelism benefit due to the extra coordination needed to reassemble results in sequence.

**Q186. Why do some data sources parallelize better than others?**
Sources with efficient, precise splitting characteristics (like `ArrayList`, which can split at any index in O(1)) parallelize much better than sources that are awkward to split (like `LinkedList`, which needs traversal to find any middle point).

**Q187. What danger exists when using parallel streams (or CompletableFuture) inside a busy web server handling many concurrent requests?**
Both default to sharing the same application-wide common `ForkJoinPool`. If many concurrent requests each trigger parallel operations, they compete for the same limited pool of worker threads — potentially starving unrelated parts of the application relying on that same shared pool.

**Q188. How would you verify whether using a parallel stream actually improves performance for a specific workload, rather than assuming it does?**
Benchmark both the sequential and parallel versions under realistic data sizes and hardware — ideally with a proper benchmarking tool (like JMH) rather than simple manual timing, since JVM warm-up effects can easily mislead naive measurements. Never assume "parallel = faster" without verifying it for the specific case.

---

# SECTION 15: Thread-Safe Collections (Q189–Q198)

## Basic

**Q189. What makes `ConcurrentHashMap` different from a plain `HashMap` wrapped with `Collections.synchronizedMap()`?**
`ConcurrentHashMap` uses fine-grained internal locking/CAS (only locking small parts of the structure when truly needed), giving much higher throughput under concurrent access. `Collections.synchronizedMap()` just wraps every method call with one single lock over the WHOLE map, so all operations from all threads serialize behind that one lock.

**Q190. Does `ConcurrentHashMap` allow null keys or values?**
No — unlike a regular HashMap. In a concurrent map, `get(key) == null` would be ambiguous (no mapping vs. mapping to null, especially with concurrent modification happening), so nulls are disallowed entirely to remove that ambiguity.

## Intermediate

**Q191. What atomic compound methods does `ConcurrentHashMap` provide, and why are they useful?**
`putIfAbsent()`, `computeIfAbsent()`, and `merge()` — these perform a "check, then act" sequence as ONE atomic step, avoiding the race window a manual two-step check-then-insert would have in concurrent code.
```java
map.computeIfAbsent("key", k -> expensiveComputation());
```

**Q192. What is `CopyOnWriteArrayList`, and what's its main trade-off?**
A thread-safe List where every mutation (add/remove/set) creates a full copy of the internal array. Reads are lock-free and fast, with no risk of `ConcurrentModificationException`. Trade-off: every write is expensive (full array copy), so it's only suited for read-heavy, write-rare scenarios (like a list of event listeners).

**Q193. What is the caveat with `Collections.synchronizedList()` when iterating?**
Individual method calls are synchronized, but COMPOUND operations (like iterating the whole list) are NOT automatically safe — you must manually synchronize on the list itself while iterating, since another thread could still modify it between individual synchronized calls otherwise.
```java
synchronized (syncList) {
    for (String item : syncList) { ... }
}
```

## Advanced

**Q194. What is a "fail-safe" iterator, and which collections provide one?**
An iterator that doesn't throw `ConcurrentModificationException` even if the collection is modified during iteration — `CopyOnWriteArrayList`'s iterator works off a snapshot taken when created; `ConcurrentHashMap`'s iterator tolerates concurrent changes, reflecting a "weakly consistent" view (guaranteed not to throw, but not guaranteed to reflect every very latest update instantly).

**Q195. How does `ConcurrentHashMap`'s `size()` method handle the fact that no single lock protects the whole map?**
It maintains striped counters (multiple internal counter cells updated via CAS) rather than one single shared counter, avoiding contention — `size()`/`mappingCount()` sums these, giving an approximation that's accurate enough for a concurrently-mutating structure, though not a strictly precise, frozen-in-time snapshot.

**Q196. What is `BlockingDeque`, and how does it relate to `BlockingQueue`?**
A double-ended version of `BlockingQueue` — supports blocking insertion/removal from BOTH ends, useful for work-stealing algorithms or scenarios needing both FIFO and LIFO blocking behavior in one structure.

**Q197. Why might `ConcurrentSkipListMap` be chosen over `ConcurrentHashMap`?**
`ConcurrentSkipListMap` maintains SORTED key order (like a thread-safe `TreeMap`) with concurrent-safe operations, while `ConcurrentHashMap` provides no ordering guarantee at all — choose it specifically when you need both thread safety AND sorted iteration/range queries.

**Q198. In a fintech-style idempotent transaction cache, why would `ConcurrentHashMap.computeIfAbsent()` be preferred over a manual "check if key exists, then insert" using a plain HashMap and synchronized block?**
`computeIfAbsent()` performs the check-and-insert as one atomic operation, eliminating the race window where two concurrent threads could both see "key doesn't exist yet" and both proceed to process the same transaction twice — exactly the kind of bug idempotency logic is meant to prevent. A manual two-step check with a plain HashMap and synchronized block CAN also be made correct, but requires much more careful, error-prone hand-coding to get right.

---

# SECTION 16: Mixed / Scenario-Based Questions (Q199–Q210)

**Q199. What will this code print, and why?**
```java
public class Demo {
    static int counter = 0;
    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            for (int i = 0; i < 1000; i++) counter++;
        };
        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start(); t2.start();
        t1.join(); t2.join();
        System.out.println(counter);
    }
}
```
It will usually print something LESS than 2000, and the exact number will vary between runs. `counter++` is not atomic, and both threads are updating it without any synchronization — some increments get lost due to the classic read-modify-write race condition.

**Q200. How would you fix Q199 using the smallest possible change?**
Replace `counter++` with an `AtomicInteger`'s `incrementAndGet()`, or wrap the increment in a `synchronized` block — either fixes the race condition, with `AtomicInteger` generally being the lighter-weight fix for a simple counter.

**Q201. Design a thread-safe singleton using double-checked locking. Why is `volatile` required on the instance field?**
```java
class Singleton {
    private static volatile Singleton instance;
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
`volatile` is required because object construction isn't atomic — without it, due to instruction reordering, another thread could see a non-null reference to a PARTIALLY constructed object (its fields not yet fully set), leading to bugs. `volatile` prevents this specific reordering issue.

**Q202. How would you implement a simple thread-safe cache with expiration, combining ConcurrentHashMap and a ScheduledExecutorService?**
Use `ConcurrentHashMap<K, CacheEntry<V>>` to store values alongside their expiration timestamp, and a `ScheduledExecutorService` running periodically to sweep and remove expired entries — reads/writes to the map are thread-safe via ConcurrentHashMap directly, while the scheduled cleanup task runs independently in the background.

**Q203. Why might increasing the number of threads in a thread pool NOT improve performance, or even make it worse?**
If the work is CPU-bound and the number of threads already matches or exceeds the number of available CPU cores, adding more threads just increases context-switching overhead without adding real parallel capacity. It can help more for I/O-bound work, where threads spend time waiting rather than actively computing.

**Q204. In a producer-consumer system using BlockingQueue, what happens if consumers process work much slower than producers create it, using an unbounded queue?**
The queue keeps growing without limit, since producers never get blocked — this can eventually lead to `OutOfMemoryError`. Using a BOUNDED queue instead would make `put()` block once full, naturally slowing producers down to match the consumers' pace (backpressure).

**Q205. Explain, step by step, why this snippet can deadlock.**
```java
synchronized (accountA) {
    synchronized (accountB) {
        transfer(accountA, accountB);
    }
}
// meanwhile, another thread runs:
synchronized (accountB) {
    synchronized (accountA) {
        transfer(accountB, accountA);
    }
}
```
Thread 1 locks `accountA` then waits for `accountB`. Thread 2 locks `accountB` then waits for `accountA`. If both threads acquire their first lock at roughly the same time, each ends up waiting forever for a lock the other holds — a classic circular-wait deadlock. Fix: always acquire locks in a consistent order (e.g., by comparing account IDs and always locking the lower ID first), regardless of transfer direction.

**Q206. How would you safely publish an object that was constructed in one thread, for use by another thread?**
Use one of the JMM's recognized happens-before mechanisms: pass it through a properly synchronized block, assign it to a `volatile` field, pass it via a thread-safe collection like `ConcurrentHashMap`, or hand it off through `Thread.start()`/`join()` — any of these establish the needed happens-before relationship, guaranteeing the receiving thread sees a fully, correctly constructed object.

**Q207. What's wrong with this attempt at lazy initialization, and how does it relate to the JMM?**
```java
class Lazy {
    private static Lazy instance;
    public static Lazy getInstance() {
        if (instance == null) {
            instance = new Lazy(); // no synchronization at all
        }
        return instance;
    }
}
```
With no synchronization or volatile at all, multiple threads calling `getInstance()` at the same time could each see `instance == null` and each construct a SEPARATE `Lazy` object (defeating the singleton guarantee), and even worse, due to instruction reordering, one thread could see a non-null but only partially-constructed instance. This needs either proper synchronization, the double-checked locking pattern from Q201, or a simpler safe alternative like an eagerly-initialized static field or an enum-based singleton.

**Q208. Describe, in your own words, a scenario where you would choose `CompletableFuture` over a simple `ExecutorService` + `Future` combination.**
When you need to CHAIN multiple dependent async steps together (e.g., "fetch user, then fetch their orders, then compute a summary") without writing nested blocking `.get()` calls, or when you need to COMBINE multiple independent async calls (like calling two microservices in parallel and merging their results) — `CompletableFuture`'s composition methods (`thenCompose`, `thenCombine`) handle this cleanly, whereas plain `Future` would force blocking at each step.

**Q209. Why is testing multithreaded code notoriously difficult, and what general strategies help?**
Race conditions and timing-dependent bugs may not show up reliably in normal test runs — they can pass thousands of times and then fail once, under just the right (or wrong) timing. Strategies: stress-testing with many concurrent threads and iterations, using thread-safety-focused testing tools, keeping critical sections as small and simple as possible to reason about, and preferring well-tested, high-level concurrency utilities (java.util.concurrent) over hand-rolled synchronization wherever possible.

**Q210. In a fintech context, why might a bounded `ThreadPoolExecutor` with `CallerRunsPolicy` be a better fit for processing incoming payment requests than an unbounded thread pool?**
An unbounded pool (or unbounded queue) risks resource exhaustion under a sudden traffic spike, potentially crashing the service entirely. A bounded pool with `CallerRunsPolicy` provides natural backpressure — once the pool and its queue are full, new incoming requests get processed on the calling thread itself (slowing down whatever is submitting them, like the web server's request-handling thread), which throttles the overall intake rate instead of letting unbounded work pile up and risk an out-of-memory crash — a much safer failure mode for a system handling real financial transactions.

---

# Quick-reference summary table

| Section | Topic | Question range |
|---|---|---|
| 1 | Threads and the Thread Lifecycle | Q1–Q20 |
| 2 | Race Conditions and synchronized | Q21–Q38 |
| 3 | volatile | Q39–Q48 |
| 4 | wait/notify/notifyAll | Q49–Q60 |
| 5 | Java Memory Model & Happens-Before | Q61–Q70 |
| 6 | Deadlock, Livelock, Starvation | Q71–Q84 |
| 7 | ExecutorService & Thread Pools | Q85–Q100 |
| 8 | Callable, Future & CompletableFuture | Q101–Q118 |
| 9 | ReentrantLock & ReentrantReadWriteLock | Q119–Q132 |
| 10 | CountDownLatch, CyclicBarrier & Semaphore | Q133–Q145 |
| 11 | BlockingQueue | Q146–Q155 |
| 12 | Atomic Classes and CAS | Q156–Q168 |
| 13 | ThreadLocal | Q169–Q178 |
| 14 | ForkJoinPool & Parallel Streams | Q179–Q188 |
| 15 | Thread-Safe Collections | Q189–Q198 |
| 16 | Mixed / Scenario-Based | Q199–Q210 |

**Total: 210 questions.**

---

## Suggested study strategy

1. **Sections 1–4** (Threads, synchronized, volatile, wait/notify) are the non-negotiable foundation — expect direct questions on the `count++` race condition and the volatile-doesn't-fix-it gotcha in almost every interview.
2. **Section 5** (JMM) is asked less often in full depth, but understanding it makes every other answer sound like real understanding instead of memorization — worth the investment.
3. **Section 6** (Deadlock/Livelock/Starvation) is extremely commonly asked, especially "how do you prevent deadlock" and "walk me through a real deadlock scenario."
4. **Sections 7–11** (ExecutorService, Future/CompletableFuture, Locks, Latches/Barriers, BlockingQueue) form the heart of most practical interview questions, especially at product and fintech companies building real concurrent systems — spend the most practice time here.
5. **Section 12** (Atomic/CAS) is a common, natural follow-up once you've shown solid understanding of synchronized/volatile.
6. **Section 13** (ThreadLocal) — the thread-pool memory-leak gotcha (Q173) is a favorite specific follow-up question.
7. **Section 14** (ForkJoinPool/Parallel Streams) ties directly back to Java 8 Streams knowledge — good to connect the two topics together in your answers.
8. **Section 15** (Thread-safe Collections) is commonly tested alongside Java Collections Framework questions — know ConcurrentHashMap deeply.
9. **Section 16** (Mixed/Scenario-based) is where interviewers test whether you can actually design and reason about concurrent systems, not just recite definitions — practice explaining these out loud, and try writing the code by hand without looking, since that's usually how it's actually tested live.
