# Java Collections Framework & Multithreading — Interview Q&A
### Basic → Advanced, for product-company interview prep (Razorpay / CRED / Groww / PhonePe / Juspay style)

---

# PART 1: JAVA COLLECTIONS FRAMEWORK

## Section A — Basics (Q1–Q15)

**Q1. What is the Java Collections Framework?**
A unified architecture (interfaces, implementations, algorithms) for storing and manipulating groups of objects. Core interfaces: `Collection`, `List`, `Set`, `Queue`, `Map` (Map is not a `Collection`, it's a separate hierarchy of key-value pairs).

**Q2. What's the difference between `Collection` and `Collections`?**
`Collection` is the root interface of the framework (List, Set, Queue extend it). `Collections` is a utility class with static helper methods (`sort()`, `reverse()`, `synchronizedList()`, `unmodifiableList()`, etc.).

**Q3. Array vs ArrayList — key differences?**
Arrays have fixed size and can hold primitives; ArrayList is resizable and holds only objects (autoboxing wraps primitives). ArrayList gives you built-in methods (add, remove, contains) that arrays don't.

**Q4. ArrayList vs LinkedList — when to use which?**
- ArrayList: backed by a dynamic array. O(1) random access (`get(i)`), O(n) insert/delete in the middle (shifting elements).
- LinkedList: doubly linked list. O(1) insert/delete at ends, O(n) random access (must traverse).
Use ArrayList when reads dominate; LinkedList when you do frequent insert/delete at head/tail (or need it as a Deque).

**Q5. What is the difference between `List`, `Set`, and `Map`?**
- `List`: ordered, allows duplicates, index-based access.
- `Set`: no duplicates, no guaranteed index access.
- `Map`: key-value pairs, keys unique, values can duplicate.

**Q6. How does `HashSet` ensure uniqueness?**
Internally backed by a `HashMap` — every element you add becomes a key in that map with a dummy constant value (`PRESENT`). Uniqueness comes from the same `hashCode()`/`equals()` contract HashMap uses for keys.

**Q7. Difference between `HashSet`, `LinkedHashSet`, and `TreeSet`?**
- `HashSet`: no ordering guarantee, O(1) average add/contains.
- `LinkedHashSet`: maintains insertion order, slightly more overhead (extra linked list).
- `TreeSet`: sorted order (natural or via Comparator), O(log n) operations, backed by a Red-Black tree (`TreeMap` internally).

**Q8. What is the `equals()` and `hashCode()` contract?**
1. If two objects are equal via `equals()`, they **must** have the same `hashCode()`.
2. Equal hashCodes do NOT imply the objects are equal (hash collisions are allowed).
3. hashCode() must be consistent — same object → same hash across calls (unless fields used in it change).
Breaking this contract silently corrupts hash-based collections (you can "lose" objects in a HashSet/HashMap).

**Q9. Can a `List` contain duplicate elements? Can a `Set`?**
List: yes. Set: no — adding a duplicate (per equals/hashCode) is a no-op and `add()` returns `false`.

**Q10. What is an `Iterator`? How is it different from a for-each loop?**
`Iterator` is an object that lets you traverse a collection and safely `remove()` elements during traversal. A for-each loop uses an Iterator under the hood but doesn't expose it — so you can't call `remove()` inside a for-each without a `ConcurrentModificationException`.

**Q11. What's the difference between `Iterator` and `ListIterator`?**
`Iterator` only goes forward and only supports `remove()`. `ListIterator` (List-only) goes both directions and additionally supports `add()` and `set()` during iteration.

**Q12. What is autoboxing and how does it relate to Collections?**
Collections can't store primitives directly — `int` gets autoboxed to `Integer`, etc. This has a performance cost (object creation, unboxing on retrieval) — relevant when discussing why primitive collections (like Eclipse Collections / Trove) exist for performance-critical code.

**Q13. What is the default initial capacity and load factor of a `HashMap`?**
Initial capacity = 16, load factor = 0.75. Resize (rehash to double capacity) triggers when `size > capacity * loadFactor`.

**Q14. Difference between `Comparable` and `Comparator`?**
- `Comparable`: defines the class's *natural* ordering, implemented inside the class via `compareTo()`. Only one natural order possible.
- `Comparator`: external, defines *custom* ordering via `compare()`. You can have many Comparators for the same class (e.g., sort by name, then by age).

**Q15. What does `Collections.unmodifiableList()` do, and is it truly immutable?**
It wraps a list in a read-only view — calling `add()`/`remove()` on the wrapper throws `UnsupportedOperationException`. But it's **not deeply immutable**: if the underlying list is mutated directly, or objects inside it are mutable, those changes are visible through the wrapper.

---

## Section B — Intermediate (Q16–Q32)

**Q16. How does `HashMap` work internally (pre-Java 8)?**
An array of buckets (`Node[] table`). `hashCode()` of the key is spread/hashed, then reduced with `(n-1) & hash` to pick a bucket index. Collisions in the same bucket are handled with a **linked list** (chaining). Lookup: compute bucket → walk the list comparing keys with `equals()`.

**Q17. What changed in HashMap in Java 8?**
When a bucket's linked list grows beyond a threshold (`TREEIFY_THRESHOLD = 8`, and table size ≥ 64), that bucket is converted into a **Red-Black tree** instead of a linked list — turning worst-case lookup from O(n) to O(log n). It converts back to a list if entries drop below `UNTREEIFY_THRESHOLD = 6`.

**Q18. Why does a bad `hashCode()` implementation degrade HashMap performance?**
If all keys return the same hashCode, they all land in one bucket — the map effectively becomes a linked list (or tree after Java 8), turning O(1) average operations into O(n) (or O(log n) post-treeification, but still degraded).

**Q19. HashMap vs Hashtable vs ConcurrentHashMap?**
- `HashMap`: not thread-safe, allows one null key and multiple null values.
- `Hashtable`: thread-safe via synchronizing every method (coarse-grained lock on the whole table) — legacy, slow under contention, no nulls allowed.
- `ConcurrentHashMap`: thread-safe with much finer-grained locking (bucket-level in Java 7, CAS + synchronized-per-bin in Java 8+), high concurrency, no null keys/values allowed.

**Q20. What is `LinkedHashMap` used for?**
A HashMap that also maintains a doubly linked list across entries, preserving insertion order (or access order if configured). Commonly used to implement an **LRU cache** by overriding `removeEldestEntry()`.

**Q21. How would you implement an LRU cache using Java collections?**
```java
class LRUCache<K,V> extends LinkedHashMap<K,V> {
    private final int capacity;
    LRUCache(int capacity) {
        super(capacity, 0.75f, true); // true = access-order
        this.capacity = capacity;
    }
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return size() > capacity;
    }
}
```

**Q22. What is `fail-fast` vs `fail-safe` iteration?**
- Fail-fast (ArrayList, HashMap, HashSet default iterators): throws `ConcurrentModificationException` if the collection is structurally modified during iteration (detected via a `modCount` check).
- Fail-safe (CopyOnWriteArrayList, ConcurrentHashMap's iterator): iterates over a snapshot or tolerates concurrent modification without throwing, but may not reflect the very latest updates.

**Q23. What causes `ConcurrentModificationException` and how do you avoid it?**
Caused by modifying a collection's structure (add/remove) while iterating with a fail-fast iterator, other than through the iterator's own `remove()`. Avoid it by using `Iterator.remove()`, `CopyOnWriteArrayList`, `ConcurrentHashMap`, or collecting changes and applying them after iteration (e.g., `removeIf()`).

**Q24. What is `TreeMap` and how does it maintain order?**
A `NavigableMap` backed by a Red-Black tree, keeping keys sorted (natural order or Comparator). Gives O(log n) get/put/remove and supports range operations like `floorKey()`, `ceilingKey()`, `headMap()`, `tailMap()`.

**Q25. Difference between `poll()`, `peek()`, and `remove()` in Queue?**
- `poll()`: retrieves and removes the head, returns `null` if empty.
- `peek()`: retrieves head without removing, returns `null` if empty.
- `remove()`: retrieves and removes head, throws `NoSuchElementException` if empty (mirrors `element()` which peeks and throws).

**Q26. What is `PriorityQueue` and how is it ordered?**
A queue backed by a binary heap; elements are ordered by natural ordering or a supplied Comparator — NOT insertion order. `poll()` always returns the smallest (or highest priority) element. Insertion/removal is O(log n).

**Q27. What is `Deque` and how does it differ from `Queue` and `Stack`?**
`Deque` (double-ended queue) allows insertion/removal from both ends — it can act as both a FIFO queue and a LIFO stack. `ArrayDeque` is generally preferred over the legacy `Stack` class (which is synchronized and slower) and over `LinkedList` for stack/queue use cases.

**Q28. Why is `Vector`/`Stack` considered legacy?**
Both synchronize every method, causing performance overhead even in single-threaded use. Modern alternatives: `ArrayList` (instead of Vector), `ArrayDeque` (instead of Stack), or `Collections.synchronizedList()` / `CopyOnWriteArrayList` when thread safety is actually needed.

**Q29. What is `CopyOnWriteArrayList` and when would you use it?**
A thread-safe List variant where every mutation (add/remove/set) creates a fresh copy of the underlying array. Reads are lock-free and never throw `ConcurrentModificationException`. Best for read-heavy, write-rare scenarios (e.g., listener lists) — writes are expensive since they copy the whole array.

**Q30. What is the difference between `Arrays.asList()` and `List.of()`?**
- `Arrays.asList()`: fixed-size list backed by the array — you can `set()` elements (reflected in the array) but `add()`/`remove()` throw `UnsupportedOperationException`.
- `List.of()` (Java 9+): fully immutable — even `set()` throws. Also disallows nulls (throws NPE if you try to add null).

**Q31. What is `Collections.synchronizedList()` and what's a common pitfall with it?**
It wraps a list so individual method calls are synchronized. Pitfall: **compound operations** (like iterating, or check-then-act) are NOT atomic — you must manually synchronize on the returned list when iterating:
```java
synchronized(list) {
    for (Object o : list) { ... }
}
```

**Q32. How do you sort a `List` of custom objects by multiple fields?**
Using `Comparator` chaining (Java 8+):
```java
list.sort(Comparator.comparing(Employee::getDept)
                     .thenComparing(Employee::getSalary, Comparator.reverseOrder()));
```

---

## Section C — Advanced (Q33–Q45)

**Q33. Walk through what happens internally when you call `hashMap.put(key, value)`.**
1. Compute `hash(key)` — HashMap applies a supplemental hash function (`h ^ (h >>> 16)`) to spread bits and reduce collisions from poor hashCode implementations.
2. Determine bucket index: `(n - 1) & hash` where n = table length.
3. If bucket is empty, insert a new Node there.
4. If bucket has entries, walk the chain (or tree): if a key matches (`==` or `.equals()`), overwrite its value; otherwise append a new node (or insert into the tree).
5. If chain length exceeds `TREEIFY_THRESHOLD` (8) and table size ≥ 64, treeify the bucket.
6. If `size > capacity * loadFactor`, resize (double capacity, rehash all entries into new buckets).

**Q34. Why does HashMap use power-of-2 table sizes?**
So bucket index calculation `(n-1) & hash` works as a fast bitmask instead of a modulo operation, and so that on resize, each old bucket's entries split cleanly into exactly two new buckets (based on one extra bit), avoiding a full rehash-and-redistribute from scratch.

**Q35. What is the difference between `HashMap` resizing behavior and `ArrayList` resizing behavior?**
ArrayList grows by 1.5x (in most JDKs, roughly `oldCapacity + oldCapacity/2`) and copies the whole array. HashMap doubles its capacity and **rehashes every entry** because bucket index depends on capacity — an O(n) operation, which is why pre-sizing a HashMap when you know approximate size (`new HashMap<>(expectedSize)`) is a common optimization.

**Q36. What's the time complexity of common operations across List/Set/Map implementations?**

| Operation | ArrayList | LinkedList | HashSet/HashMap | TreeSet/TreeMap |
|---|---|---|---|---|
| get/access by index | O(1) | O(n) | N/A | N/A |
| add/put | O(1) amortized | O(1) at ends | O(1) avg | O(log n) |
| contains/get by key | O(n) | O(n) | O(1) avg | O(log n) |
| remove | O(n) | O(1) at ends, O(n) middle | O(1) avg | O(log n) |

**Q37. What is `WeakHashMap` and where is it used?**
A Map whose keys are held via `WeakReference`s. If a key has no other strong references, the GC can reclaim it and its entry is automatically removed from the map. Useful for caches where entries should disappear once nothing else references the key (avoids memory leaks).

**Q38. What is `IdentityHashMap`?**
Uses reference equality (`==`) instead of `.equals()`/`hashCode()` to compare keys. Useful in scenarios like object graph traversal (cycle detection, serialization) where you must distinguish distinct objects that happen to be `.equals()`.

**Q39. How does `ConcurrentHashMap` achieve thread safety in Java 8+ without a single global lock?**
It no longer uses segment-based locking (Java 7's 16-segment model). Instead:
- Uses `CAS` (Compare-And-Swap, via `Unsafe`/`VarHandle`) for inserting into an empty bucket — lock-free.
- Uses **synchronized blocks scoped to individual bins** (the first node of a bucket) only when there's an actual collision or tree operation — not the whole map.
- Size tracking uses a striped counter (`CounterCell[]`) to avoid contention on a single counter under high concurrency.
This gives much higher throughput than Hashtable's whole-map lock.

**Q40. Does `ConcurrentHashMap` allow null keys or values? Why?**
No — unlike HashMap. The reason (per Doug Lea): in a concurrent map, `map.get(key) == null` is ambiguous — it could mean "no mapping" or "mapping to null" — and there's no safe way to distinguish this from another thread possibly removing the entry at the same instant, so nulls are disallowed to eliminate that ambiguity entirely.

**Q41. What is the difference between `Collections.emptyList()`, `new ArrayList<>()`, and `null` as a return value for "no results"?**
`Collections.emptyList()` returns an immutable, shared, zero-allocation singleton list — safe to iterate, avoids NPEs at call sites, and is the recommended practice over returning `null` (which forces every caller to null-check).

**Q42. How would you make a custom class usable as a HashMap key correctly?**
Override both `equals()` and `hashCode()` consistently, based on the same immutable fields. If the fields used in hashCode can change after insertion, the object becomes "lost" in the map (you can no longer find it because its bucket location no longer matches its current hash) — so keys should ideally be immutable.

**Q43. What is the difference between `Set.of()` and `HashSet` regarding iteration order?**
`Set.of()` (Java 9+ immutable set) has an unspecified and even randomized iteration order (intentionally randomized per JVM run to prevent code from relying on it) — even more unpredictable than `HashSet`, which is stable within a single run but not across runs/versions.

**Q44. What's the difference between shallow copy and deep copy in the context of Collections (e.g. `new ArrayList<>(oldList)`)?**
`new ArrayList<>(oldList)` creates a new list object but copies only the references to the elements (shallow copy) — mutating an element object affects both lists. A deep copy would require cloning each element individually.

**Q45. When would you choose `EnumMap`/`EnumSet` over `HashMap`/`HashSet`?**
When keys are from an `enum` type. `EnumMap`/`EnumSet` are backed by arrays indexed by the enum's ordinal — extremely fast (no hashing needed) and memory-compact, and iteration follows natural enum order.

---

# PART 2: MULTITHREADING & CONCURRENCY

## Section A — Basics (Q46–Q58)

**Q46. What is a process vs a thread?**
A process is an independent execution unit with its own memory space. A thread is a lightweight unit of execution within a process, sharing the process's memory (heap) but with its own stack, program counter, and registers.

**Q47. How do you create a thread in Java? Two ways?**
1. Extend `Thread` and override `run()`.
2. Implement `Runnable` and pass it to a `Thread` (preferred, since Java doesn't support multiple inheritance and Runnable decouples the task from the threading mechanism).
```java
Runnable task = () -> System.out.println("running");
new Thread(task).start();
```

**Q48. What's the difference between `start()` and `run()`?**
`start()` creates a new OS thread and eventually calls `run()` on it. Calling `run()` directly just executes the code on the **current thread** — no new thread is created, defeating the purpose.

**Q49. What are the states in a thread's lifecycle?**
NEW → RUNNABLE → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED. RUNNABLE covers both "ready to run" and "actually running" (Java doesn't distinguish these at the API level, unlike OS-level states).

**Q50. What does the `synchronized` keyword do?**
Ensures only one thread can execute a synchronized block/method on a given object (its intrinsic monitor lock) at a time — providing both **mutual exclusion** and **visibility** (changes made inside a synchronized block are visible to the next thread that acquires the same lock).

**Q51. Difference between synchronizing a method vs a block?**
Synchronizing an entire method locks on `this` (or the Class object for static methods) for the method's full duration. Synchronizing a block lets you lock only the critical section and choose the lock object explicitly — generally preferred for finer-grained, less contended locking.

**Q52. What is a race condition?**
When multiple threads access and modify shared data concurrently without proper synchronization, and the final result depends on the unpredictable timing/interleaving of their execution — leading to incorrect or inconsistent results.

**Q53. What is the `volatile` keyword?**
Guarantees visibility: writes to a volatile variable by one thread are immediately visible to other threads (bypassing CPU-core caches, no reordering across it). It does **not** provide atomicity for compound operations like `count++` (read-modify-write is still 3 separate steps).

**Q54. Why is `count++` not thread-safe even with `volatile int count`?**
`count++` is read → increment → write, three separate steps. Two threads can both read the same value before either writes back, losing an increment. `volatile` only ensures each read/write sees the latest value — it doesn't make the whole sequence atomic. Use `AtomicInteger` or `synchronized` instead.

**Q55. What is a deadlock? Give a simple example.**
Two or more threads waiting forever for locks the others hold. Classic example: Thread A locks `lockX` then tries to lock `lockY`; Thread B locks `lockY` then tries to lock `lockX` — both wait forever. Fix: always acquire locks in a consistent global order.

**Q56. What is `Thread.sleep()` vs `Object.wait()`?**
`Thread.sleep(ms)` pauses the current thread without releasing any locks it holds. `wait()` (must be called inside a synchronized block on that object) releases the object's lock and waits until `notify()`/`notifyAll()` is called on the same object.

**Q57. What is thread priority — does it guarantee execution order?**
Threads have a priority (1–10) hinting to the OS scheduler which threads to favor, but it's not a guarantee — actual scheduling is OS/JVM dependent and priority should never be relied on for correctness.

**Q58. What is a daemon thread?**
A background thread (set via `setDaemon(true)`) that doesn't prevent JVM shutdown — the JVM exits once only daemon threads remain (e.g., GC thread). Regular (non-daemon) threads keep the JVM alive until they finish.

---

## Section B — Intermediate (Q59–Q75)

**Q59. What is the Java Memory Model (JMM) and why does it matter?**
The JMM defines the rules for how/when writes by one thread become visible to reads by another thread, and what reorderings the compiler/CPU are allowed to do. Without understanding it, code can appear to work in testing but fail unpredictably in production due to caching and instruction reordering across cores.

**Q60. What is "happens-before"?**
A JMM guarantee: if action A happens-before action B, then A's effects (including all its memory writes) are guaranteed visible to B. Established by things like: unlocking a monitor happens-before a later thread locking the same monitor; a volatile write happens-before a subsequent volatile read of the same variable; `Thread.start()` happens-before anything in the new thread; a thread's actions happen-before another thread observing it has terminated via `join()`.

**Q61. What is `ExecutorService` and why use it over raw `Thread` creation?**
A higher-level abstraction that manages a pool of reusable threads and a task queue, avoiding the cost of creating/destroying a native thread per task. Also provides lifecycle management (`shutdown()`), task submission with results (`Future`), and scheduling.

**Q62. What are the common types of thread pools in `Executors`?**
- `newFixedThreadPool(n)`: fixed number of threads, unbounded queue.
- `newCachedThreadPool()`: creates threads as needed, reuses idle ones, unbounded — risky under heavy load (can spawn unlimited threads).
- `newSingleThreadExecutor()`: one thread, tasks run sequentially.
- `newScheduledThreadPool(n)`: supports delayed/periodic task execution.
*(In practice, many teams now prefer constructing `ThreadPoolExecutor` directly for explicit control over queue bounds and rejection policy, since the `Executors` factory methods can hide unbounded queues/threads that cause OOM under load.)*

**Q63. What is the difference between `Runnable` and `Callable`?**
`Runnable.run()` returns nothing and can't throw checked exceptions. `Callable<V>.call()` returns a value of type V and can throw checked exceptions — used with `ExecutorService.submit()` to get a `Future<V>`.

**Q64. What is a `Future`? What's the problem with it?**
Represents the pending result of an asynchronous computation (`get()` blocks until it's done). Problem: `Future.get()` is blocking, and Futures can't easily be composed/chained (e.g., "when this finishes, do that") — which is what `CompletableFuture` was introduced to solve.

**Q65. What is `CompletableFuture` and how does it improve on `Future`?**
Supports non-blocking composition and chaining: `.thenApply()`, `.thenCompose()`, `.thenCombine()`, `.exceptionally()`, and running callbacks automatically when a stage completes, without manually blocking on `get()`. It also supports combining multiple async computations (`allOf`, `anyOf`).

**Q66. What is `ReentrantLock` and how is it different from `synchronized`?**
An explicit lock (`java.util.concurrent.locks`) offering features `synchronized` lacks: `tryLock()` (non-blocking attempt), `lockInterruptibly()` (can respond to interrupts while waiting), fairness policy (FIFO ordering option), and the ability to have multiple `Condition` objects (like multiple wait-sets) per lock. Must be manually released in a `finally` block — unlike `synchronized`, which releases automatically.

**Q67. What is `ReentrantReadWriteLock` used for?**
Allows multiple threads to read concurrently (shared lock) but only one thread to write (exclusive lock, blocking all readers/writers) — ideal for read-heavy, write-rare data structures where plain mutual exclusion would be overly restrictive.

**Q68. What is `CountDownLatch`?**
A one-time synchronization aid: initialized with a count; threads call `await()` to block until the count reaches zero via other threads calling `countDown()`. Common use: wait for N worker threads to finish before proceeding. Cannot be reset/reused.

**Q69. What is `CyclicBarrier` and how does it differ from `CountDownLatch`?**
`CyclicBarrier` makes a fixed number of threads wait for each other at a common barrier point, then releases them all together — and can be **reused** for multiple rounds (unlike CountDownLatch, which is one-shot). Also supports an optional action run once all threads arrive.

**Q70. What is a `Semaphore`?**
Maintains a set of permits; threads call `acquire()` (blocks if no permits available) and `release()`. Used to limit concurrent access to a resource (e.g., allow at most N threads into a connection pool at once).

**Q71. What is `ThreadLocal` and when would you use it?**
Gives each thread its own independent copy of a variable — no synchronization needed since there's no sharing. Common uses: per-thread `SimpleDateFormat` instances (not thread-safe otherwise), storing request-scoped context (e.g., a request ID or user session in a web app) accessible without passing it through every method call.

**Q72. What's a memory leak risk with `ThreadLocal`, especially in thread pools?**
If you don't call `remove()` after use, the value stays attached to the pooled thread (since pooled threads are reused, not destroyed) — causing stale data to leak into unrelated future tasks on that thread, and preventing GC of the stored object as long as the thread lives.

**Q73. What are Atomic classes (`AtomicInteger`, `AtomicLong`, etc.) and how do they achieve thread safety without locks?**
They use CAS (Compare-And-Swap) — a hardware-supported atomic instruction: "if the current value equals the expected value, swap it with the new value" in one atomic step. If another thread changed it in between, the CAS fails and the operation retries. This gives lock-free thread safety, generally faster than synchronized under low-to-moderate contention.

**Q74. What is the producer-consumer problem, and how do you implement it in Java?**
A pattern where producer threads generate data and consumer threads process it, coordinated via a shared buffer. Simplest Java implementation uses `BlockingQueue` (e.g., `LinkedBlockingQueue`), where `put()` blocks if the queue is full and `take()` blocks if it's empty — handling all the wait/notify coordination internally.

**Q75. What is `BlockingQueue` and name a few implementations.**
A `Queue` that blocks on `put()` when full and `take()` when empty, instead of throwing or returning null. Implementations: `ArrayBlockingQueue` (bounded, array-backed), `LinkedBlockingQueue` (optionally bounded), `PriorityBlockingQueue` (unbounded, priority-ordered), `SynchronousQueue` (zero capacity — a direct hand-off between one producer and one consumer).

---

## Section C — Advanced (Q76–Q90)

**Q76. Explain deadlock, livelock, and starvation — how do they differ?**
- **Deadlock**: threads block each other forever, holding locks the other needs.
- **Livelock**: threads keep actively responding to each other (e.g., both repeatedly backing off and retrying) but make no actual progress — they're not blocked, just stuck in a loop.
- **Starvation**: a thread is perpetually denied access to a resource because other threads (often higher priority, or simply "unlucky" scheduling) keep getting it first.

**Q77. How do you prevent deadlock?**
1. Always acquire multiple locks in a fixed global order across all threads.
2. Use `tryLock()` with a timeout instead of blocking indefinitely, and back off/retry on failure.
3. Minimize the scope/number of locks held simultaneously.
4. Use higher-level concurrency utilities (BlockingQueue, ConcurrentHashMap) that avoid manual lock ordering altogether.

**Q78. What is the difference between `synchronized` and `java.util.concurrent.locks.Lock` in terms of what happens if an exception is thrown inside the critical section?**
`synchronized` automatically releases the monitor lock even if an exception propagates out of the block. With explicit `Lock`, you must release manually — if you forget the `finally { lock.unlock(); }`, the lock leaks and is never released, causing all other threads to block forever. Correct pattern:
```java
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

**Q79. What is false sharing, and how does it relate to multithreading performance?**
When independent variables used by different threads happen to sit on the same CPU cache line, writes by one thread invalidate the cache line for other cores even though they're accessing logically unrelated data — causing unnecessary cache coherence traffic and performance degradation. Mitigated via padding (e.g., `@Contended` annotation, or manually spacing fields across cache lines — 64 bytes typically).

**Q80. Explain the internal working of `ConcurrentHashMap`'s `size()` method — why isn't it trivially accurate?**
In a highly concurrent map, no single lock protects the whole structure, so a precise live count would require locking everything (defeating the purpose). Instead, Java 8's ConcurrentHashMap maintains striped counters (`CounterCell[]`) updated via CAS per bucket-group to reduce contention, and `size()`/`mappingCount()` sums them — giving an approximation that's accurate "enough" for a concurrently-mutating structure, not a strictly linearizable snapshot.

**Q81. What is `ForkJoinPool` and the work-stealing algorithm?**
A specialized executor for divide-and-conquer parallel tasks (`RecursiveTask`/`RecursiveAction`), where each worker thread has its own double-ended task queue. When a thread's queue is empty, it "steals" tasks from the *tail* of another busy thread's queue (while the owner works from the *head*) — balancing load across cores automatically without central coordination. Backs Java's parallel streams (`.parallelStream()`).

**Q82. What is the difference between `synchronized` blocks and `Lock`-based approaches in terms of fairness?**
`synchronized` provides no fairness guarantee — any waiting thread could acquire the lock next, potentially causing starvation for some. `ReentrantLock(true)` can be constructed with a fairness policy, granting the lock to the longest-waiting thread first — at some throughput cost.

**Q83. What's the difference between `wait()`/`notify()` and `Condition` objects (from `Lock`)?**
`wait()`/`notify()` work on an object's single implicit monitor/wait-set. `Condition` (via `lock.newCondition()`) lets a single `Lock` have **multiple** independent wait-sets — e.g., separate "queue not full" and "queue not empty" conditions for a bounded buffer, letting you `signal()` exactly the right waiting group instead of waking every waiter with `notifyAll()`.

**Q84. How would you implement a simple thread-safe bounded blocking queue from scratch (conceptually) without using `java.util.concurrent`?**
Use a `synchronized` block guarding the internal array/list and two conditions: producers `wait()` while the queue is full, consumers `wait()` while it's empty; after each successful `put`/`take`, call `notifyAll()` (not just `notify()`, to avoid missed-signal issues when there are both producer and consumer waiters on the same monitor).

**Q85. What is "spurious wakeup" and why should `wait()` always be called in a loop?**
The JVM/OS may wake a waiting thread even without an explicit `notify()` (a rare but legal occurrence per the JMM spec). So the condition must always be re-checked in a `while` loop, not an `if`:
```java
synchronized(lock) {
    while (!conditionMet) {
        lock.wait();
    }
    // proceed
}
```

**Q86. Explain the difference between `notify()` and `notifyAll()`, and why `notifyAll()` is usually safer.**
`notify()` wakes exactly one arbitrary waiting thread; `notifyAll()` wakes all of them (they then re-compete for the lock and re-check their wait condition). If different threads are waiting for different conditions on the same monitor, `notify()` can wake the "wrong" thread (one whose condition isn't actually satisfied) leading to a missed signal/deadlock — `notifyAll()` avoids this at a small performance cost.

**Q87. How does `CompletableFuture.supplyAsync()` decide which thread pool runs the task?**
By default it uses the common `ForkJoinPool.commonPool()`. You can supply your own `Executor` as a second argument — important in production, since sharing the common pool with CPU-bound and I/O-bound (blocking) tasks can starve other parts of the application that also depend on it.

**Q88. What is the difference between optimistic and pessimistic locking, and how does CAS relate to this in Java?**
Pessimistic locking (e.g., `synchronized`) assumes conflicts are likely and blocks other threads out entirely during the critical section. Optimistic locking (CAS-based, as in Atomic classes) assumes conflicts are rare — it proceeds without blocking, then checks at commit time if the value changed underneath it, retrying if so. Optimistic approaches scale better under low contention but can waste work (retries) under high contention.

**Q89. What is the "ABA problem" in CAS-based concurrency, and how is it addressed?**
CAS only checks if the current value equals the expected value — but if the value changed from A→B→A between the read and the CAS, the CAS succeeds even though the value did change in between, potentially causing subtle bugs (e.g., in lock-free stacks/linked structures). Addressed via `AtomicStampedReference`, which pairs the value with a version stamp that increments on every change, so A→B→A is detected as different (stamp differs) even though the value is the same.

**Q90. In a Spring Boot banking/fintech context, why would you prefer `ConcurrentHashMap` + atomic operations over `synchronized` blocks for something like an in-memory idempotency cache?**
`ConcurrentHashMap` provides fine-grained, high-throughput thread safety out of the box, and its atomic compound methods — `putIfAbsent()`, `computeIfAbsent()`, `merge()` — let you implement check-then-act idempotency logic (e.g., "insert only if this transaction key doesn't already exist") as a single atomic call, avoiding the race window and coarse contention a hand-rolled `synchronized` block around a `HashMap` would introduce under concurrent transaction load.

---

## How to use this
- Go section by section (Basic → Advanced) rather than jumping around — later answers assume you already know the earlier concepts (e.g., Q76–Q90 assume you're comfortable with Q59–Q75).
- For the Advanced sections especially, be ready to draw the HashMap bucket/tree structure or the deadlock lock-ordering scenario on a whiteboard — interviewers at product companies often want you to reason out loud, not just recite the answer.
- Good follow-up depth for a fintech-domain interview: be ready to connect Q39/Q40/Q90 (ConcurrentHashMap internals) to real idempotency/ledger design questions, since that's a very common systems-design thread at payments companies.
