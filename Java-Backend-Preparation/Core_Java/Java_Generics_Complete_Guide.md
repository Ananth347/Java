# Java Generics — The Complete Guide
### Basic to Advanced, explained in simple English, with code examples

---

## How to read this document

Read this from the top, in order. Each part builds on the one before it. For every topic you will get:
- **What it is** (in simple words)
- **Why it exists** (what problem it fixes)
- **How it works** (with easy examples)
- **A common interview question**

Topics covered: Why Generics exist · Generic Classes · Generic Methods · Bounded Types · Wildcards · The PECS Principle · Type Erasure · Raw Types.

---

# PART 1: WHY DO GENERICS EXIST?

## 1.1 The problem before Generics

Before Java 5, collections like `ArrayList` could hold **any** type of object, mixed together, with no restriction.

```java
// Old code, before Generics (Java 1.4 style)
List list = new ArrayList();
list.add("Hello");
list.add(123); // no error here, even though this is a number, not text

String value = (String) list.get(1); // CRASH at runtime! ClassCastException
```

Look closely at this problem: you put a `String` and an `Integer` into the same list, and Java did not stop you. The mistake only shows up **later**, when you try to use the value and it crashes — this is a runtime error, which is much harder to catch than a compile-time error.

## 1.2 What Generics actually do

Generics let you tell Java, right at the moment you create a collection (or any generic class), **what type of objects it is allowed to hold**. Java then checks this rule for you **at compile time**, before the program even runs.

```java
// Same code, using Generics
List<String> list = new ArrayList<>();
list.add("Hello");
list.add(123); // COMPILE ERROR — caught immediately, before running the program!

String value = list.get(1); // no cast needed either — Java already knows it's a String
```

## 1.3 The two big benefits of Generics

1. **Type safety** — mistakes are caught at compile time, not at runtime. This means you find bugs while writing code, not later when the program crashes for a real user.
2. **No manual casting needed** — since Java already knows the exact type stored inside, you don't need to write `(String)` or `(Integer)` every time you read a value out.

### Common interview angle
**Q: What is the main reason Generics were added to Java?**
To catch type mistakes at compile time instead of runtime, and to remove the need for manual type casting when reading values from a collection or class.

---

# PART 2: GENERIC CLASSES

## 2.1 What is a Generic Class?

A generic class is a class that works with a **type placeholder** instead of one fixed, hardcoded type. You decide the real type later, when you actually use the class.

## 2.2 A simple example — a Box that holds one item

```java
class Box<T> {
    private T item;

    public void setItem(T item) {
        this.item = item;
    }

    public T getItem() {
        return item;
    }
}
```

Here, `T` is a **type placeholder** (also called a "type parameter"). It does not mean anything by itself — it simply means "whatever type you choose when you use this class."

```java
Box<String> stringBox = new Box<>();
stringBox.setItem("Hello");
String value = stringBox.getItem(); // no casting needed

Box<Integer> intBox = new Box<>();
intBox.setItem(100);
Integer number = intBox.getItem();
```

The same `Box` class now works safely for `String`, `Integer`, or any other type — without writing a separate class for each one.

## 2.3 Why is it called `T`? Can you use any letter?

`T` is just a naming convention (it usually means "Type"), not a rule. You can technically use any valid name, but by convention, Java programmers use single, capital letters for type parameters, so it is easy to tell them apart from real class names:

| Letter | Usually means |
|---|---|
| `T` | Type |
| `E` | Element (often used in collections, like `List<E>`) |
| `K` | Key (used in Maps) |
| `V` | Value (used in Maps) |
| `N` | Number |
| `R` | Return type |

```java
class Pair<K, V> { // a class can have MORE than one type placeholder
    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
}

Pair<String, Integer> pair = new Pair<>("age", 25);
```

## 2.4 Why can't you use a Generic type with a primitive, like `Box<int>`?

Generics only work with **objects**, not primitive types (`int`, `double`, `boolean`, etc.), because of how Generics are actually implemented internally (explained fully in Part 7, Type Erasure). This is why you must use the **wrapper class** instead:

```java
Box<int> box = new Box<>();       // COMPILE ERROR — not allowed
Box<Integer> box2 = new Box<>();  // correct — use the wrapper class Integer instead of int
```
Java automatically converts (autoboxes) a primitive `int` into an `Integer` when you add it, so this still feels natural to use in day-to-day code, even though the underlying storage is always an object.

### Common interview angle
**Q: Can you create `new T()` inside a generic class, to make a new instance of the placeholder type?**
No — this is not allowed. At the time the class is compiled, Java has no idea what real type `T` will end up being, so it cannot know how to create a new one. (This connects closely to Type Erasure, covered later in Part 7.)

---

# PART 3: GENERIC METHODS

## 3.1 What is a Generic Method?

A generic method is a single method that can work with different types, decided separately from the class it lives in. This is useful even inside a completely normal, non-generic class.

## 3.2 A simple example

```java
class Printer {
    // <T> right before the return type means: "this method has its own type placeholder"
    public <T> void printItem(T item) {
        System.out.println("Item: " + item);
    }
}

Printer printer = new Printer();
printer.printItem("Hello");  // T becomes String here
printer.printItem(123);      // T becomes Integer here
```

Notice that `Printer` itself is a completely normal class — it is only the `printItem` method that is generic.

## 3.3 Why put `<T>` before the return type?

The `<T>` right before the return type is what tells Java, "this method introduces its own new type placeholder, separate from anything else." Without it, Java would think `T` is supposed to be some already-existing, real class named `T`, and the code would not compile.

```java
public <T> T identity(T input) { // <T> declares the placeholder, then it's used as the return type AND parameter type
    return input;
}
```

## 3.4 A useful real example — finding the maximum of any comparable type

```java
public static <T extends Comparable<T>> T findMax(List<T> list) {
    T max = list.get(0);
    for (T item : list) {
        if (item.compareTo(max) > 0) {
            max = item;
        }
    }
    return max;
}

List<Integer> numbers = Arrays.asList(3, 7, 2, 9, 4);
System.out.println(findMax(numbers)); // 9

List<String> words = Arrays.asList("banana", "apple", "cherry");
System.out.println(findMax(words)); // cherry
```
(Don't worry about `extends Comparable<T>` yet — this is a Bounded Type, explained fully in the next part.)

## 3.5 Generic method vs Generic class — what's the difference?

| | Generic Class | Generic Method |
|---|---|---|
| Where the type placeholder is declared | On the class itself: `class Box<T>` | On the method itself: `public <T> void method(T x)` |
| When is the type decided | When you create the object: `new Box<String>()` | Each time you call the method, Java figures it out from the argument |
| Can a normal (non-generic) class have a generic method? | N/A | Yes — this is very common |

### Common interview angle
**Q: Can a static method be generic?**
Yes — in fact, a static method **must** declare its own type placeholder with `<T>` if it wants to be generic, because static methods do not belong to any specific object, so they cannot reuse the type placeholder from the class itself (like `T` in `class Box<T>`).

---

# PART 4: BOUNDED TYPES

## 4.1 What is a Bounded Type?

Normally, a type placeholder like `T` can be **any** type at all. A **Bounded Type** puts a limit (a "bound") on what `T` is allowed to be — for example, "T must be a Number, or something that can be compared."

## 4.2 Why would you need this?

Sometimes your generic code needs to actually **do something** with the type inside it — like adding numbers together, or comparing two values. If `T` could genuinely be anything, Java would not let you call `.compareTo()` or `.doubleValue()` on it, because a totally unknown type might not have those methods at all.

## 4.3 Upper Bounded Types — using `extends`

```java
class NumberBox<T extends Number> { // T must be Number, or a subclass of Number (like Integer, Double)
    private T value;

    public NumberBox(T value) {
        this.value = value;
    }

    public double getDoubleValue() {
        return value.doubleValue(); // allowed! Java KNOWS T is at least a Number, so doubleValue() definitely exists
    }
}

NumberBox<Integer> intBox = new NumberBox<>(10);   // OK — Integer is a Number
NumberBox<Double> doubleBox = new NumberBox<>(5.5); // OK — Double is a Number
NumberBox<String> stringBox = new NumberBox<>("x"); // COMPILE ERROR — String is not a Number
```

The keyword `extends` is used here even though `Number` is a class, not an interface — in Generics, `extends` always means "this type, or something below it in the hierarchy," no matter whether the bound is a class or an interface.

## 4.4 Why does `extends` unlock new methods inside the generic class?

Because once you write `T extends Number`, Java now knows for certain that whatever real type gets used for `T`, it will always have at least the methods that `Number` itself guarantees (like `doubleValue()`, `intValue()`). Without the bound, `T` could be absolutely anything, so Java has to assume it only has the very basic methods that every single Java object has (like `toString()`), and nothing more.

## 4.5 Multiple bounds

You can require a type to satisfy MORE than one bound at once, using `&` to combine them:

```java
class Box<T extends Number & Comparable<T>> {
    // T must be BOTH a Number AND Comparable
}
```
A rule to remember: if you combine a class and interfaces this way, the class must always be listed first, and only one class is allowed (Java does not support multiple inheritance of classes), but you can list several interfaces after it.

## 4.6 A practical example — summing a list of any Number type

```java
public static double sumAll(List<? extends Number> list) {
    double total = 0;
    for (Number n : list) {
        total += n.doubleValue();
    }
    return total;
}

System.out.println(sumAll(Arrays.asList(1, 2, 3)));       // works with Integer
System.out.println(sumAll(Arrays.asList(1.5, 2.5, 3.0))); // works with Double
```
(This example uses a Wildcard, `? extends Number`, which is explained fully next — it's closely related to Bounded Types, so it's a good preview.)

### Common interview angle
**Q: What's the difference between `T extends Number` and `T` with no bound at all?**
With no bound, `T` could be absolutely any type, so inside the generic class you can only call the most basic methods every object has (like `toString()`, `equals()`). With `T extends Number`, Java guarantees `T` is always at least a `Number`, so you can safely call `Number`'s own methods (like `doubleValue()`) directly inside the generic class.

---

# PART 5: WILDCARDS

## 5.1 What is a Wildcard?

A wildcard is written as `?`, and it means "an unknown type." It is used when you don't need to know or care about the exact type — you just need to work with a generic type in a flexible way, usually as a **method parameter**.

## 5.2 The problem wildcards solve

Imagine this method:
```java
public static void printList(List<Object> list) {
    for (Object obj : list) {
        System.out.println(obj);
    }
}
```
You might expect this to work with a `List<String>` too, since a String is an Object — but it does not:
```java
List<String> names = Arrays.asList("Alice", "Bob");
printList(names); // COMPILE ERROR!
```
This is confusing at first, but there's a good reason: `List<String>` is **not** actually considered a subtype of `List<Object>` in Java's Generics system, even though `String` IS a subtype of `Object`. If Java allowed this, you could accidentally add an `Integer` into what was supposed to be a list of only `String`s, since the method's parameter type says `List<Object>`, which looks like it should accept any object at all.

Wildcards solve exactly this problem, letting a method safely accept a generic type where the exact type inside doesn't need to be known.

## 5.3 The Unbounded Wildcard — `List<?>`

```java
public static void printList(List<?> list) { // accepts a list of ANY type
    for (Object obj : list) {
        System.out.println(obj);
    }
}

printList(Arrays.asList("Alice", "Bob"));  // works
printList(Arrays.asList(1, 2, 3));          // also works
```
Use this when your method genuinely doesn't care what type is inside the list — it just needs to read the elements as generic `Object`s (for example, just to print them).

## 5.4 The Upper Bounded Wildcard — `List<? extends T>`

This means: "a list of some unknown type, but that type must be `T` or a subclass of `T`."

```java
public static double sumAll(List<? extends Number> list) {
    double total = 0;
    for (Number n : list) {
        total += n.doubleValue();
    }
    return total;
}

sumAll(Arrays.asList(1, 2, 3));       // List<Integer> — Integer extends Number, so this is allowed
sumAll(Arrays.asList(1.1, 2.2));      // List<Double> — also allowed
```
This is used when your method only needs to **read** values out of the list, treating each one as at least a `Number`.

### Why can't you `add()` to a `List<? extends Number>`?
```java
List<? extends Number> list = new ArrayList<Integer>();
list.add(10); // COMPILE ERROR!
```
Here's the reasoning: Java only knows the list holds "some subtype of Number" — it does NOT know exactly which one. It could secretly be a `List<Integer>`, or a `List<Double>`, or a `List<Float>`. If Java let you `add(10)` (an int), but the list actually turns out to be a `List<Double>` underneath, you would have just put the wrong type into it. Since Java cannot guarantee which exact type is safe to add, it plays it safe and does not allow adding anything at all (except `null`).

## 5.5 The Lower Bounded Wildcard — `List<? super T>`

This means: "a list of some unknown type, but that type must be `T` or a **superclass/parent** of `T`."

```java
public static void addNumbers(List<? super Integer> list) {
    list.add(1);  // allowed
    list.add(2);  // allowed
}

List<Number> numberList = new ArrayList<>();
addNumbers(numberList); // works, because Number is a super type of Integer

List<Object> objectList = new ArrayList<>();
addNumbers(objectList); // also works, because Object is a super type of Integer too
```
This is used when your method needs to **add** items into the list — the exact type of the list itself is uncertain, but Java guarantees it is at least "big enough" to safely hold an `Integer` (or anything below it).

### Why can't you safely read a specific type back out of a `List<? super Integer>`?
```java
List<? super Integer> list = new ArrayList<Number>();
Integer x = list.get(0); // COMPILE ERROR!
Object obj = list.get(0); // this IS allowed
```
Java only knows the list holds "Integer, or something above it" — it could actually be a `List<Number>` or even a `List<Object>` underneath. So reading an element out, Java can only be sure it is at least an `Object` — it cannot promise you it is specifically an `Integer`.

### Common interview angle
**Q: In one sentence, when should you use `? extends` versus `? super`?**
Use `? extends` when you only need to **read** (get) values out safely. Use `? super` when you only need to **write** (add) values into it safely. This exact rule has a well-known name, covered next.

---

# PART 6: THE PECS PRINCIPLE

## 6.1 What does PECS stand for?

**"Producer Extends, Consumer Super."** This is a simple memory trick to help you remember when to use `? extends` and when to use `? super`.

## 6.2 What does "Producer" and "Consumer" mean here?

- A **Producer** is something that gives values OUT to you — you are reading/getting data FROM it. Use `? extends` here.
- A **Consumer** is something that takes values IN from you — you are writing/adding data INTO it. Use `? super` here.

## 6.3 A simple way to picture it

Think of a list as either a **fruit basket you are picking from** (a Producer — you take fruit out, so use `extends`) or an **empty basket you are putting fruit into** (a Consumer — you put fruit in, so use `super`).

## 6.4 A classic real example — `Collections.copy()`

Java's own standard library uses this exact pattern for the `copy()` method, which copies elements from a source list into a destination list:

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (int i = 0; i < src.size(); i++) {
        dest.set(i, src.get(i));
    }
}
```
- `src` (the **source**, where we read values FROM) is a **Producer** → uses `extends`.
- `dest` (the **destination**, where we write values INTO) is a **Consumer** → uses `super`.

```java
List<Integer> integers = Arrays.asList(1, 2, 3);
List<Number> numbers = new ArrayList<>(Arrays.asList(0, 0, 0));

Collections.copy(numbers, integers); // reading Integers (producer), writing into Numbers (consumer) — works!
```

## 6.5 What if you need to BOTH read and write?

If your method genuinely needs to both read values out AND add new values in, wildcards don't work well for that case at all — you should just use a plain, exact type parameter instead, like `List<T>`, with no wildcard.

### Common interview angle
**Q: Why can't you add anything (except null) to a `List<? extends Number>`?**
Because PECS tells us `? extends` is meant for **producing** (reading) only — Java cannot guarantee what the real underlying type is, so allowing you to add something could break type safety. This connects directly back to the explanation in Part 5.4.

---

# PART 7: TYPE ERASURE

## 7.1 What is Type Erasure?

This is one of the most important — and most surprising — things to understand about Java Generics. **Type Erasure** means: all the generic type information you write in your code (like `<String>` or `<Integer>`) is only used by the **compiler**, to check your code for mistakes. Once compilation is finished, all of that generic type information is completely **removed (erased)** from the actual compiled `.class` file.

## 7.2 Why does Java do this?

This design choice was made for **backward compatibility**. Generics were added in Java 5, but Java needed old code (written before Generics existed) to still work correctly alongside brand-new generic code, without needing to recompile everything in the world. By erasing generic type information down to plain, ungeneric bytecode, old and new code can work together smoothly.

## 7.3 A simple demonstration

```java
List<String> stringList = new ArrayList<>();
List<Integer> intList = new ArrayList<>();

System.out.println(stringList.getClass() == intList.getClass()); // true !!
```

This surprises many learners — at runtime, both lists are treated as being the exact same class, plain `ArrayList`, with no memory at all of ever having held `String` or `Integer` specifically. The `<String>` and `<Integer>` only existed for the compiler's benefit, while checking your code.

## 7.4 What does the compiler actually do with `T`?

During compilation, Java replaces every generic type placeholder (`T`, `E`, `K`, `V`, etc.) with either:
- `Object`, if there was no bound on it, or
- The bound itself, if you specified one (like `T extends Number` becomes `Number`).

```java
// What you write:
class Box<T> {
    private T item;
    public T getItem() { return item; }
}

// What it roughly becomes internally, after erasure:
class Box {
    private Object item;
    public Object getItem() { return item; }
}
```

## 7.5 Where does the automatic casting come from, if the type is erased?

When you write `String value = stringBox.getItem();`, the compiler secretly inserts a cast for you behind the scenes, since it already checked at compile time that this is safe:
```java
// What you write:
String value = stringBox.getItem();

// What it actually becomes after compilation:
String value = (String) stringBox.getItem();
```
This is exactly why Generics don't need you to manually write the cast yourself — the compiler is quietly doing it for you, using the type information it had before erasing it.

## 7.6 Why can't you create `new T()`, or an array of `T`, or check `instanceof T`?

All of these limitations trace directly back to Type Erasure — by the time the actual program is running, Java has no idea what `T` really was anymore (it's already been erased down to `Object` or its bound). So none of these operations, which genuinely need to know the real, exact type at runtime, are possible:
```java
class Box<T> {
    T item;

    public Box() {
        item = new T();          // COMPILE ERROR — Java doesn't know what T really is at runtime
    }

    public boolean check(Object obj) {
        return obj instanceof T; // COMPILE ERROR — same reason
    }

    public T[] makeArray() {
        return new T[10];        // COMPILE ERROR — same reason
    }
}
```

## 7.7 Why can't you overload two methods that only differ by generic type?

```java
public void process(List<String> list) { }
public void process(List<Integer> list) { } // COMPILE ERROR!
```
After Type Erasure, both of these methods end up looking **identical** — both simply become `process(List list)`, with no generic information left to tell them apart. Java sees this as trying to declare the exact same method twice, which is not allowed.

### Common interview angle
**Q: If Generics are erased at runtime, does that mean Generics provide zero real safety?**
No — this is a common misunderstanding. The safety Generics provide happens entirely at **compile time**, which is exactly when you want to catch mistakes — before the program is ever run. Type Erasure just means that this checking does not carry over into extra runtime information; the actual bugs it prevents are still very real and are still caught, just earlier (at compile time) rather than never.

---

# PART 8: RAW TYPES

## 8.1 What is a Raw Type?

A Raw Type is when you use a generic class **without** specifying any type parameter at all — basically writing old, pre-Generics-style Java code.

```java
List list = new ArrayList(); // this is a RAW TYPE — no <String>, no <Integer>, nothing
```

## 8.2 Why does Java still allow Raw Types at all?

Purely for **backward compatibility**, exactly like Type Erasure. Old code written before Java 5 (before Generics existed) still needs to compile and work correctly, so Java still allows this older style, even though it is now considered outdated and is generally discouraged in new code.

## 8.3 What do you lose by using a Raw Type?

You lose every single benefit that Generics were created to provide:

```java
List list = new ArrayList(); // Raw Type
list.add("Hello");
list.add(123); // no error at all — Raw Types skip ALL of the compiler's type checking

for (Object obj : list) {
    String value = (String) obj; // CRASH at runtime, on the 123 — back to the old, unsafe way
}
```

## 8.4 What warning does Java give you for using Raw Types?

The compiler will give you an "unchecked" warning, something like:
```
Note: Demo.java uses unchecked or unsafe operations.
```
This is a strong hint that you have accidentally lost type safety somewhere, and it is well worth paying attention to, since it usually points to a real, genuine risk in your code.

## 8.5 Raw Type vs. Wildcard `List<?>` — these look similar, but are very different!

This is a very commonly confused pair, so let's be precise:

| | `List` (Raw Type) | `List<?>` (Wildcard) |
|---|---|---|
| Type checking | None at all — behaves exactly like old pre-Java-5 code | Full type checking IS still active |
| Can you add elements? | Yes, ANYTHING, with no error at all (unsafe!) | No, you cannot add anything (except null) — Java protects you |
| Compiler warning? | Yes, "unchecked" warning | No warning — this is considered a genuinely safe, correct usage |

```java
List raw = new ArrayList();
raw.add("Hello");
raw.add(123); // allowed, but dangerous — no error, no warning message here at this exact line

List<?> wildcard = new ArrayList<String>();
wildcard.add("Hello"); // COMPILE ERROR — wildcards actively protect you from unsafe additions
```

### Common interview angle
**Q: Should you ever use Raw Types in new code you write today?**
No, essentially never. Raw Types exist purely to let very old, legacy code still compile — for any new code, you should always use proper Generics (with a specific type, or a wildcard if the type is genuinely unknown), since Raw Types throw away all of the safety benefits Generics exist to provide.

---

# PART 9: PUTTING IT ALL TOGETHER — A REALISTIC EXAMPLE

Here is a small example that uses several ideas from this document together, the way you might actually write this kind of code:

```java
class Pair<K, V> {
    private final K key;
    private final V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
}

class NumberUtils {
    // Generic method with a Bounded Type — T must be some kind of Number
    public static <T extends Number> double sum(List<T> list) {
        double total = 0;
        for (T item : list) {
            total += item.doubleValue();
        }
        return total;
    }

    // Wildcard usage, following PECS: we only READ from source, so it's a Producer -> use extends
    public static void printAll(List<? extends Number> source) {
        for (Number n : source) {
            System.out.println(n);
        }
    }

    // Wildcard usage: we only WRITE into destination, so it's a Consumer -> use super
    public static void fillWithZero(List<? super Integer> destination, int count) {
        for (int i = 0; i < count; i++) {
            destination.add(0);
        }
    }
}

public class Demo {
    public static void main(String[] args) {
        Pair<String, Integer> agePair = new Pair<>("Ananth", 26);
        System.out.println(agePair.getKey() + " is " + agePair.getValue());

        List<Integer> numbers = Arrays.asList(10, 20, 30);
        System.out.println("Sum: " + NumberUtils.sum(numbers));

        NumberUtils.printAll(numbers);

        List<Number> zeros = new ArrayList<>();
        NumberUtils.fillWithZero(zeros, 3);
        System.out.println(zeros); // [0, 0, 0]
    }
}
```

---

# Quick-reference summary table

| Topic | One-line meaning |
|---|---|
| Generic Class | A class with a type placeholder, decided when you create it (`Box<T>`) |
| Generic Method | A single method with its own type placeholder, decided at each call |
| Bounded Type | Restricts a type placeholder to a certain type or its subclasses (`T extends Number`) |
| Wildcard `?` | Represents an unknown type, mostly used as a method parameter |
| `? extends T` | Producer — you can only READ safely, not add |
| `? super T` | Consumer — you can only WRITE safely, reading only gives you Object |
| PECS | "Producer Extends, Consumer Super" — the memory trick for the two wildcards above |
| Type Erasure | All generic type info is removed after compilation — it only exists to help the compiler |
| Raw Type | Using a generic class with no type at all, old pre-Java-5 style — avoid in new code |

---

## Suggested study order for interview prep

1. **Part 1 and 2** (why Generics exist, Generic Classes) — the foundation everything else builds on.
2. **Part 3** (Generic Methods) — quick to learn once Part 2 makes sense.
3. **Part 7** (Type Erasure) — this explains the "why" behind almost every strange Generics rule (why you can't do `new T()`, why raw types exist, why overloading fails) — very commonly asked, and it makes everything else click into place.
4. **Part 4 and 5** (Bounded Types, Wildcards) — these two go together, and are frequently tested with small code snippets asking "does this compile?"
5. **Part 6** (PECS) — a favorite, short, high-value interview question — be ready to say "Producer Extends, Consumer Super" instantly, and explain why with an example.
6. **Part 8** (Raw Types) — quick to learn, mostly important so you can clearly explain why raw types are unsafe and outdated.
