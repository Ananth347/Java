# Java Generics & Reflection API — Complete Interview Q&A
### Basic → Advanced, every topic covered, with clear explanations

---

## How to use this document
Questions are grouped by topic and move from basic to hard within each section. Read a section fully before moving to the next — later questions often build on earlier ones.

**Part 1 — Generics topics covered:** Why Generics exist · Generic Classes · Generic Methods · Bounded Types · Wildcards · The PECS Principle · Type Erasure · Raw Types · Mixed/scenario questions.

**Part 2 — Reflection API topics covered:** Why Reflection exists · The `Class` class · Field · Method · Constructor · Annotations & Reflection · Dynamic Class Loading · Invoking Methods · Mixed/scenario questions.

---
---

# PART 1: JAVA GENERICS

---

# SECTION 1: Why Generics Exist (Q1–Q8)

**Q1. What problem did Generics solve when they were added in Java 5?**
Before Generics, collections like `ArrayList` could hold any type of object mixed together, with no restriction. Mistakes (like putting a number into what was meant to be a list of text) were only caught at runtime, as a `ClassCastException`, often far away from where the actual mistake happened.

**Q2. Show an example of the problem, without Generics.**
```java
List list = new ArrayList();
list.add("Hello");
list.add(123); // no error here at all
String value = (String) list.get(1); // CRASH at runtime — ClassCastException
```

**Q3. How do Generics fix this?**
```java
List<String> list = new ArrayList<>();
list.add("Hello");
list.add(123); // COMPILE ERROR — caught immediately, before running
String value = list.get(1); // no cast needed either
```

**Q4. What are the two main benefits of using Generics?**
1. **Type safety** — mistakes are caught at compile time instead of at runtime.
2. **No manual casting needed** — since Java already knows the exact type, you don't need to write `(String)` or `(Integer)` every time you read a value.

**Q5. Do Generics exist only for collections?**
No — although collections are the most common use, Generics can be applied to any class or method you write yourself, to make it work safely with different types without duplicating code.

**Q6. What is a "type parameter"?**
A placeholder (like `T`, `E`, `K`, `V`) used in a generic class or method, standing in for a real type that gets decided later, when the class or method is actually used.

**Q7. Are Generics a runtime feature or a compile-time feature?**
Primarily a compile-time feature — the type checking and safety benefits happen while compiling your code. (The deeper reason why is explained fully in the Type Erasure section, Section 6.)

**Q8. In one sentence, why are Generics considered such an important interview topic?**
Because they touch almost every part of the Java Collections Framework and many library APIs, and because their internal behavior (especially Type Erasure) explains a lot of "strange" rules that otherwise seem confusing or arbitrary.

---

# SECTION 2: Generic Classes (Q9–Q20)

## Basic

**Q9. What is a generic class?**
A class that works with a type placeholder instead of one fixed, hardcoded type — the real type is decided later, when the class is actually used.

**Q10. Write a simple generic Box class.**
```java
class Box<T> {
    private T item;
    public void setItem(T item) { this.item = item; }
    public T getItem() { return item; }
}
```

**Q11. How do you use the Box class with different types?**
```java
Box<String> stringBox = new Box<>();
stringBox.setItem("Hello");

Box<Integer> intBox = new Box<>();
intBox.setItem(100);
```

**Q12. Is `T` a special keyword in Java?**
No — `T` is just a naming convention, not a rule. By convention, single capital letters are used for type parameters, so they're easy to distinguish from real class names (e.g., `T` for Type, `E` for Element, `K`/`V` for Key/Value).

## Intermediate

**Q13. Can a generic class have more than one type parameter?**
Yes.
```java
class Pair<K, V> {
    private K key;
    private V value;
    public Pair(K key, V value) { this.key = key; this.value = value; }
}
Pair<String, Integer> pair = new Pair<>("age", 25);
```

**Q14. Why can't you write `Box<int>`?**
Generics only work with objects, not primitive types, because of how Generics are implemented internally (Type Erasure — covered in Section 6). You must use the wrapper class instead: `Box<Integer>`.

**Q15. Does autoboxing help when using Generics with numbers?**
Yes — Java automatically converts a primitive `int` into an `Integer` object when needed, so day-to-day usage still feels natural even though the underlying storage is always an object.

**Q16. Can you create `new T()` inside a generic class?**
No, this is not allowed. At compile time, Java has no idea what the real type `T` will end up being, so it doesn't know how to construct a new instance of it.

## Advanced

**Q17. Can a generic class extend another generic class?**
Yes.
```java
class Container<T> { protected T value; }
class SpecialContainer<T> extends Container<T> {
    public void printValue() { System.out.println(value); }
}
```

**Q18. Can a subclass fix the type parameter of its generic parent class?**
Yes — a subclass doesn't have to stay generic itself.
```java
class StringContainer extends Container<String> { } // fixed to always use String
```

**Q19. Can static members of a generic class use the class's own type parameter `T`?**
No — static members belong to the class itself, not to any specific instance, but the type parameter `T` is only meaningfully tied to a specific instance (decided when that instance is created). So a static field or method cannot use the class-level `T` directly.

**Q20. Can nested generic classes exist, like `Box<Box<String>>`?**
Yes — Generics can be nested freely. `Box<Box<String>>` represents a Box that itself holds another Box, which in turn holds a String.

---

# SECTION 3: Generic Methods (Q21–Q30)

## Basic

**Q21. What is a generic method?**
A single method that can work with different types, with its own type placeholder — separate from whatever class it lives in.

**Q22. Write a simple generic method.**
```java
public <T> void printItem(T item) {
    System.out.println("Item: " + item);
}
```

**Q23. Does the class containing a generic method need to be generic itself?**
No — a completely normal, non-generic class can still have individual generic methods.

## Intermediate

**Q24. Why must you write `<T>` right before the return type in a generic method?**
It tells Java, "this method introduces its own new type placeholder." Without it, Java would think `T` refers to some already-existing, real class named `T`, and the code would fail to compile.

**Q25. Write a generic method that returns the input value unchanged (an identity method).**
```java
public <T> T identity(T input) {
    return input;
}
```

**Q26. Can a static method be generic?**
Yes — in fact, a static method MUST declare its own type placeholder with `<T>` if it wants to be generic, since static methods don't belong to any instance and so cannot reuse a class-level type parameter.
```java
public static <T> T identity(T input) { return input; }
```

**Q27. Write a generic method to find the maximum value in a list of Comparable items.**
```java
public static <T extends Comparable<T>> T findMax(List<T> list) {
    T max = list.get(0);
    for (T item : list) {
        if (item.compareTo(max) > 0) max = item;
    }
    return max;
}
```

## Advanced

**Q28. Can a generic method have multiple type parameters?**
Yes.
```java
public <K, V> void printPair(K key, V value) {
    System.out.println(key + " = " + value);
}
```

**Q29. Does Java always correctly infer the type of a generic method call, or can it need help?**
Usually it infers correctly from the arguments passed in. In rare ambiguous cases (like when the return type alone needs to determine the type, with no clear argument to infer from), you may need to specify the type explicitly: `Collections.<String>emptyList()`.

**Q30. What's the key difference between a Generic Class and a Generic Method, summarized simply?**
A Generic Class's type is decided once, when the object is created (`new Box<String>()`). A Generic Method's type is decided fresh, every time the method is called, based on the arguments passed in at that specific call.

---

# SECTION 4: Bounded Types (Q31–Q42)

## Basic

**Q31. What is a Bounded Type?**
A restriction placed on a type parameter, limiting what it's allowed to be — for example, "T must be a Number, or a subclass of Number."

**Q32. Why would you need a Bounded Type?**
So your generic code can actually call meaningful methods on the type parameter — with no bound, Java assumes `T` could be absolutely anything, so it only allows calling the most basic methods every object has (like `toString()`).

**Q33. Write a generic class using an upper bound.**
```java
class NumberBox<T extends Number> {
    private T value;
    public NumberBox(T value) { this.value = value; }
    public double getDoubleValue() { return value.doubleValue(); } // allowed! T is guaranteed to be at least a Number
}
```

## Intermediate

**Q34. Why is the keyword `extends` used even when the bound is an interface, not a class?**
In Generics, `extends` always means "this type, or something below it in the hierarchy" — regardless of whether the bound happens to be a class or an interface. This differs from normal Java class syntax, where you'd use `implements` for interfaces.

**Q35. What happens if you try `NumberBox<String>` with the bound `T extends Number`?**
Compile error — `String` is not a `Number` or a subclass of it, so it violates the declared bound.

**Q36. How do you specify multiple bounds on a single type parameter?**
Using `&` to combine them: `class Box<T extends Number & Comparable<T>>` — T must satisfy BOTH bounds at once.

**Q37. If combining a class and interfaces as multiple bounds, what rule must you follow?**
The class must always be listed first, and only one class is allowed (Java doesn't support multiple inheritance of classes), but you can list several interfaces after it.

## Advanced

**Q38. What is the difference between `T extends Number` and no bound at all, in terms of what you can do inside the class?**
With no bound, you can only call the most basic Object methods (`toString()`, `equals()`). With `T extends Number`, Java guarantees `T` is always at least a `Number`, so you can safely call `Number`'s own methods (like `doubleValue()`) directly.

**Q39. Can a bound reference the type parameter of the SAME class it's declared on? Give an example.**
Yes — this is common with `Comparable`: `class Box<T extends Comparable<T>>` — here, `T` must be comparable specifically to itself.

**Q40. Can you use a Bounded Type on a generic method as well as a generic class?**
Yes — the syntax is the same, just placed before the return type: `public static <T extends Comparable<T>> T findMax(List<T> list)`.

**Q41. Write a method with a bound requiring T to be both Serializable and Comparable.**
```java
public <T extends Comparable<T> & java.io.Serializable> T findMax(List<T> list) {
    // implementation
}
```

**Q42. Is there such a thing as a "lower bound" for a type parameter declaration itself (like `T super Number`)?**
No — lower bounds (`super`) are only allowed with **wildcards** (`? super Number`), not with a type parameter declaration directly on a class or method (`T super Number` is not valid Java syntax). This distinction is explained further in the Wildcards section.

---

# SECTION 5: Wildcards (Q43–Q56)

## Basic

**Q43. What is a wildcard in Generics?**
Written as `?`, it means "an unknown type." It's used when you don't need to know or care about the exact type, usually as a method parameter.

**Q44. Why doesn't `List<String>` work where `List<Object>` is expected, even though String is an Object?**
`List<String>` is NOT considered a subtype of `List<Object>` in Java's Generics system. If it were allowed, you could accidentally insert an `Integer` into what's supposed to be a `List<String>`, since the method's parameter type claims it accepts any Object.

**Q45. Write a method using an unbounded wildcard.**
```java
public static void printList(List<?> list) {
    for (Object obj : list) System.out.println(obj);
}
```

## Intermediate

**Q46. What does `List<? extends Number>` mean?**
"A list of some unknown type, but that type must be `Number` or a subclass of `Number`." Used when a method only needs to READ values from the list.

**Q47. Why can't you `add()` to a `List<? extends Number>`?**
Because Java only knows the list holds "some subtype of Number," not exactly which one — it could secretly be `List<Integer>`, `List<Double>`, or others. Adding the wrong type would break type safety, so Java disallows adding anything at all (except `null`).

**Q48. What does `List<? super Integer>` mean?**
"A list of some unknown type, but that type must be `Integer` or a superclass/parent of `Integer`." Used when a method needs to WRITE (add) values into the list.

**Q49. Why can you safely `add()` to a `List<? super Integer>`?**
Java guarantees the list is "at least big enough" to hold an `Integer` (it could be `List<Number>` or `List<Object>` underneath) — so adding an Integer is always safe, no matter which specific supertype the list actually is.

**Q50. Why can't you reliably read back a specific type from a `List<? super Integer>`?**
Java only knows the list holds "Integer or something above it" — it could be `List<Number>` or `List<Object>`. Reading an element out, Java can only guarantee it's at least an `Object` — it cannot promise it's specifically an `Integer`.

## Advanced

**Q51. Write a method demonstrating both `? extends` and `? super` together (like `Collections.copy`).**
```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (int i = 0; i < src.size(); i++) {
        dest.set(i, src.get(i));
    }
}
```

**Q52. Can you call `list.size()` on a `List<?>`?**
Yes — methods that don't depend on the specific element type (like `size()`, `isEmpty()`, `clear()`) work fine on wildcard types, since they don't need to know what type is inside.

**Q53. Is `List<?>` the same as a raw `List`?**
No — this is a common point of confusion, covered fully in Section 7 (Raw Types). `List<?>` still enforces type safety (you can't add anything unsafe); a raw `List` disables type checking entirely.

**Q54. Can you assign a `List<String>` to a variable of type `List<?>`?**
Yes — any parameterized list can be assigned to a `List<?>` variable, since `?` represents "some unknown, but specific, type," and `List<String>` certainly qualifies as one such possibility.

**Q55. Why would a method signature use `Comparator<? super T>` instead of just `Comparator<T>`?**
This allows a comparator written for a broader/parent type to still be used — for example, a `Comparator<Object>` can be used to compare `String` objects, since Object is a supertype of String. Using plain `Comparator<T>` would incorrectly reject this valid, more flexible comparator.

**Q56. Can a wildcard be used as the return type of a method?**
Technically yes, but it's discouraged — a wildcard return type gives the caller very little useful information about what they can do with the result (especially with `? extends`, since you couldn't safely add to it). It's usually better practice to use a concrete type parameter for return types.

---

# SECTION 6: The PECS Principle (Q57–Q64)

**Q57. What does PECS stand for?**
"Producer Extends, Consumer Super" — a memory trick for remembering when to use `? extends` versus `? super`.

**Q58. What is a "Producer" in the context of PECS?**
Something that gives values OUT to you — you're reading/getting data FROM it. Use `? extends` for this case.

**Q59. What is a "Consumer" in the context of PECS?**
Something that takes values IN from you — you're writing/adding data INTO it. Use `? super` for this case.

**Q60. Give a simple analogy for PECS.**
A fruit basket you're picking FROM is a Producer (use `extends`). An empty basket you're putting fruit INTO is a Consumer (use `super`).

**Q61. Why does `Collections.copy(List<? super T> dest, List<? extends T> src)` follow PECS correctly?**
`src` is read FROM (a Producer) → uses `extends`. `dest` is written INTO (a Consumer) → uses `super`.

**Q62. What should you use if a method needs to BOTH read from and write to the same list?**
Wildcards don't work well for that case — use a plain, exact type parameter instead, like `List<T>`, with no wildcard at all.

**Q63. Why can't you safely add to a `List<? extends Number>`? Explain using PECS.**
Because `? extends` marks it as a Producer — meant only for reading. Java cannot guarantee what the exact underlying type is, so allowing writes could break type safety; PECS tells us this case should never be used for adding elements.

**Q64. Is PECS a formal Java language rule, or a guideline?**
It's an informal guideline/memory aid for API design — the Java compiler doesn't enforce "PECS" as such, but it DOES enforce the underlying rules (`? extends` blocks most additions, `? super` limits safe reading to Object) that make PECS a genuinely useful and accurate way to remember which wildcard to choose.

---

# SECTION 7: Type Erasure (Q65–Q78)

## Basic

**Q65. What is Type Erasure?**
The process where all generic type information you write (`<String>`, `<Integer>`, etc.) is used only by the compiler to check your code, and is then completely removed from the compiled `.class` file.

**Q66. Why does Java use Type Erasure at all?**
For backward compatibility — Generics were added in Java 5, and old code written before Generics existed needed to keep working correctly alongside new generic code, without needing the whole world to recompile everything.

**Q67. Demonstrate Type Erasure with a simple example.**
```java
List<String> stringList = new ArrayList<>();
List<Integer> intList = new ArrayList<>();
System.out.println(stringList.getClass() == intList.getClass()); // true!
```
At runtime, both are treated as the exact same class, plain `ArrayList` — there's no memory of ever holding `String` or `Integer` specifically.

## Intermediate

**Q68. What does the compiler replace a type parameter `T` with, internally?**
Either `Object` (if there was no bound), or the bound itself (if you specified one — e.g., `T extends Number` becomes `Number` after erasure).

**Q69. Where does the automatic casting come from when reading a value out of a generic collection?**
The compiler secretly inserts a cast for you behind the scenes at compile time, since it already verified the type is safe. You never see this cast in your source code, but it's present in the compiled bytecode.

**Q70. Why can't you write `new T()` inside a generic class?**
Because of Type Erasure — by the time the program actually runs, Java has no idea what `T` really was anymore (it's already erased to `Object` or its bound), so it has no way to know how to construct a new instance of the real type.

**Q71. Why can't you check `obj instanceof T`?**
Same root cause — at runtime, there's no generic type information left for Java to check against; `T` has already been erased.

**Q72. Why can't you create an array of a generic type, like `new T[10]`?**
Also due to Type Erasure — arrays in Java track their actual element type at runtime for safety checks, but a generic type parameter has no real runtime type to track after erasure, making this fundamentally incompatible.

## Advanced

**Q73. Why can't you overload two methods that only differ by generic type, like `process(List<String>)` and `process(List<Integer>)`?**
After Type Erasure, both methods end up looking identical — both simply become `process(List list)`. Java sees this as declaring the exact same method twice, which isn't allowed.

**Q74. Does Type Erasure mean Generics provide zero real safety?**
No — the safety Generics provide happens entirely at compile time, which is exactly when you want to catch mistakes, before the program is ever run. Erasure just means this checking doesn't carry extra information into the runtime; the bugs it prevents are still very real and are still caught, just earlier.

**Q75. What is a "bridge method," and how does it relate to Type Erasure?**
A synthetic (compiler-generated) method the compiler automatically creates to preserve correct polymorphism when a generic method is overridden with a more specific type after erasure — you don't write these yourself; the compiler inserts them silently to keep method overriding working correctly despite erasure.

**Q76. Can you use reflection to find out what type parameter was used to create a specific generic object at runtime?**
Generally, no — because of Type Erasure, an individual object instance doesn't retain information about what type it was parameterized with. (There are some limited exceptions, like inspecting generic type information on a class's FIELD or METHOD declarations via reflection, since that structural information IS preserved in the compiled class file, even though a specific object instance doesn't carry it.)

**Q77. Why does erasure specifically choose the bound (not always Object) when replacing a bounded type parameter?**
So that code inside the generic class that relies on the bound's methods (like `T extends Number` calling `.doubleValue()`) still compiles correctly into valid bytecode — if erasure always used `Object`, those method calls wouldn't exist on `Object` and the compiled code would be invalid.

**Q78. Is Type Erasure unique to Java, or do other languages with Generics work the same way?**
It's a design choice specific to Java (made for backward compatibility with pre-Generics code) — other languages with generics, like C#, use a different approach (reified generics), where type information IS preserved and available at runtime. This is a notable, sometimes criticized, difference in Java's specific implementation.

---

# SECTION 8: Raw Types (Q79–Q88)

## Basic

**Q79. What is a Raw Type?**
Using a generic class WITHOUT specifying any type parameter at all — writing old, pre-Generics-style Java code.
```java
List list = new ArrayList(); // Raw Type
```

**Q80. Why does Java still allow Raw Types?**
Purely for backward compatibility — old code written before Java 5 needs to still compile and work, even though this style is now considered outdated and discouraged for new code.

**Q81. What do you lose by using a Raw Type?**
Every benefit Generics provide — no compile-time type checking at all, and you're back to needing manual casts, with the risk of runtime `ClassCastException`.

## Intermediate

**Q82. Show an example of the danger of using a Raw Type.**
```java
List list = new ArrayList();
list.add("Hello");
list.add(123); // no error at all, even though this is clearly wrong
for (Object obj : list) {
    String value = (String) obj; // CRASH at runtime, on the 123
}
```

**Q83. What warning does the compiler give when you use a Raw Type?**
An "unchecked" warning, something like `Note: Demo.java uses unchecked or unsafe operations.` — a strong hint that type safety has been lost somewhere in the code.

**Q84. What's the difference between a Raw Type (`List`) and a Wildcard Type (`List<?>`)?**
A Raw Type has NO type checking at all — you can add anything, unsafely, with no warning at that specific line. A Wildcard Type STILL enforces type safety — you cannot add anything (except null) to a `List<?>`, since Java is actively protecting you from an unsafe operation.

## Advanced

**Q85. Should you ever use Raw Types in new code?**
No, essentially never — for any new code, always use proper Generics (a specific type, or a wildcard if the type is genuinely unknown). Raw Types exist purely to let old legacy code still compile.

**Q86. If a method's signature uses a Raw Type parameter, can you still call it with a generic argument like `List<String>`?**
Yes — you can always pass a parameterized type where a raw type is expected (this is allowed for backward compatibility), but doing so generates an "unchecked" warning, since the method itself provides no generic type safety.

**Q87. Does using Raw Types affect performance at runtime, given that Type Erasure removes generic information anyway?**
No — since Type Erasure removes generic type information regardless, a Raw Type and its fully-generic equivalent are essentially IDENTICAL at runtime, in terms of the actual bytecode executed. The only difference is the COMPILE-TIME safety checking you lose by using the raw form.

**Q88. Can a class itself be declared without ever supporting Generics, and is that different from using a Raw Type of a generic class?**
Yes, these are different things: a class that was never written with type parameters (like `String`) is simply an ordinary, non-generic class — there's no "raw" version to speak of. A Raw Type specifically refers to using a class that WAS DECLARED generic (like `ArrayList<T>`), but choosing to omit its type parameter at the point of use.

---

# SECTION 9: Generics — Mixed / Scenario-Based Questions (Q89–Q98)

**Q89. Does this code compile? Why or why not?**
```java
List<Integer> intList = new ArrayList<>();
List<Number> numberList = intList; // does this line compile?
```
No — this does NOT compile. Even though `Integer` is a `Number`, `List<Integer>` is not considered a subtype of `List<Number>` in Java's Generics system (as explained in Section 5, Q44) — this is exactly why wildcards exist, to handle this kind of flexible relationship safely.

**Q90. Write a generic Stack class with push and pop methods.**
```java
class Stack<T> {
    private List<T> items = new ArrayList<>();
    public void push(T item) { items.add(item); }
    public T pop() {
        if (items.isEmpty()) throw new RuntimeException("Stack is empty");
        return items.remove(items.size() - 1);
    }
}
```

**Q91. Why does `Collections.max()` have the signature `static <T extends Comparable<? super T>> T max(Collection<? extends T> coll)`? Break this down.**
`Collection<? extends T>` — a Producer, since we're only reading elements out to compare, following PECS. `Comparable<? super T>` — allows a Comparable implementation from a PARENT type to still be usable (e.g., if T is a subclass that doesn't implement Comparable itself but inherits it from a parent that does).

**Q92. Given a bounded type `<T extends Animal>`, can you pass a `List<Dog>` (where Dog extends Animal) to a method expecting `List<T>`?**
Not directly through a plain `List<T>` parameter — you'd need a wildcard, `List<? extends Animal>`, to accept a `List<Dog>` safely; a bare `<T extends Animal>` on its own doesn't automatically allow subtype lists the way a wildcard does.

**Q93. Write a generic method that swaps two elements in a list.**
```java
public static <T> void swap(List<T> list, int i, int j) {
    T temp = list.get(i);
    list.set(i, list.get(j));
    list.set(j, temp);
}
```

**Q94. Why does the standard `List<T>` interface's `sort()` method use `Comparator<? super T>` as its parameter type?**
Following PECS — the Comparator is a Consumer of `T` values (it receives elements to compare), and using `? super T` allows a more general comparator (like one written for a parent type) to still be usable for sorting the list, offering more flexibility than requiring an exact `Comparator<T>`.

**Q95. Explain why this method signature is considered good practice: `public static double sum(List<? extends Number> list)`**
It uses `? extends Number` because the method only READS values from the list (a Producer, per PECS) — this allows callers to pass in a `List<Integer>`, `List<Double>`, or any other Number subtype list, giving maximum flexibility for something that's only reading, not writing.

**Q96. If you write a generic class with no bound, and store a value using `Object obj = someGenericInstance.getItem();`, is a cast needed?**
No explicit cast is written in your source code, but the compiler inserts one behind the scenes (as explained in Type Erasure) if you assign it to a more specific type than `Object`. Assigning directly to `Object` needs no cast at all, since every type is already compatible with `Object`.

**Q97. Can Generics fully replace the need for `instanceof` checks and manual casting in all situations?**
No — Generics solve type safety for collections and reusable classes/methods holding a consistent type, but there remain legitimate scenarios (like working with truly heterogeneous data, reflection-based code, or certain design patterns) where `instanceof` and casting are still necessary and appropriate.

**Q98. In a fintech-style application, why might you design a generic `Repository<T, ID>` interface, and what would `T` and `ID` typically represent?**
This is a very common pattern: `T` represents the entity type being stored (like `Transaction` or `Account`), and `ID` represents the type of that entity's unique identifier (like `Long` or `String`). A single generic interface (`Repository<T, ID>`) can then be reused across many different entity types, avoiding the need to write nearly-identical repository interfaces separately for every single entity in the system.

---
---

# PART 2: THE REFLECTION API

---

# SECTION 10: Why Reflection Exists (Q99–Q106)

**Q99. What is Reflection in Java?**
A feature that lets a program examine and act on classes, fields, methods, and constructors while it is actually running — even ones the program didn't know about while it was being written.

**Q100. Give a simple example of Reflection in action.**
```java
Person person = new Person();
Class<?> personClass = person.getClass();
System.out.println(personClass.getName()); // prints: Person
```

**Q101. Why does Spring Boot rely so heavily on Reflection?**
Spring doesn't know your class names in advance — at startup, it scans for annotated classes (like `@Component`), and uses Reflection to create instances of them, inspect their fields for annotations like `@Autowired`, and set those field values, all for classes it has never seen before while Spring itself was being written.

**Q102. Name at least three other major Java frameworks/libraries that depend on Reflection.**
Hibernate (mapping database rows to entity object fields), JUnit (finding and running `@Test`-annotated methods), and Jackson (converting JSON to/from Java objects by inspecting fields and methods).

**Q103. What are the three main costs/trade-offs of using Reflection?**
1. **Slower** — noticeably slower than normal, direct code, since Java does extra work at runtime.
2. **Less safe** — many mistakes only show up as a runtime exception, instead of being caught at compile time.
3. **Breaks encapsulation** — can access `private` fields/methods directly, ignoring the access rules the class author intended.

**Q104. Should Reflection be used for everyday, normal application code?**
Generally no — it's best reserved for framework/library-style code that genuinely needs to work with unknown classes. For normal business logic where you already know the classes involved, direct code is faster, safer, and easier to understand.

**Q105. What is the general term for what Reflection lets a program do to itself?**
Introspection (examining its own structure) combined with the ability to dynamically act on what it finds (creating objects, calling methods) — together, this is what "Reflection" refers to as a whole.

**Q106. Is Reflection unique to Java?**
No — many languages support some form of reflection (Python, C#, JavaScript all have their own versions), though the specific API and capabilities differ. Java's Reflection API is a particularly mature and heavily-used implementation of this general concept.

---

# SECTION 11: The `Class` Class (Q107–Q118)

## Basic

**Q107. What is the `Class` class?**
A real Java class (confusingly named), and every class in Java — including your own custom classes — automatically has one `Class` object representing it at runtime, holding metadata like its name, fields, methods, and constructors.

**Q108. What are the three ways to get a `Class` object?**
```java
Class<?> c1 = person.getClass();       // from an actual object
Class<?> c2 = Person.class;             // using .class syntax, if you know the name at compile time
Class<?> c3 = Class.forName("Person");  // using just the class name as TEXT, known only at runtime
```

**Q109. When would you use `Class.forName()` instead of `SomeClass.class`?**
When the class name is only available as a String at runtime — often coming from a configuration file, database, or user input — not written directly in your code.

## Intermediate

**Q110. What does `cls.getName()` return, versus `cls.getSimpleName()`?**
`getName()` returns the fully qualified name, including the package (e.g., `com.example.Person`). `getSimpleName()` returns just the short class name (`Person`).

**Q111. How do you check if a `Class` object represents an interface?**
```java
cls.isInterface(); // returns true/false
```

**Q112. How do you get a class's superclass using Reflection?**
```java
Class<?> superClass = cls.getSuperclass();
```

**Q113. How do you get the list of fields, methods, and constructors from a `Class` object?**
```java
Field[] fields = cls.getDeclaredFields();
Method[] methods = cls.getDeclaredMethods();
Constructor<?>[] constructors = cls.getDeclaredConstructors();
```

## Advanced

**Q114. What's the difference between `getFields()` and `getDeclaredFields()`?**
`getFields()` only returns PUBLIC fields, but includes ones inherited from parent classes. `getDeclaredFields()` returns ALL fields declared directly in that exact class (public, private, protected), but does NOT include inherited fields.

**Q115. Does this same `getX()` vs `getDeclaredX()` distinction apply to Methods and Constructors too?**
Yes — `getMethods()`/`getDeclaredMethods()` and the equivalent pattern for constructors follow the exact same public-and-inherited vs. all-and-only-this-class distinction.

**Q116. How can you check if a class implements a specific interface, using Reflection?**
```java
boolean implementsIt = SomeInterface.class.isAssignableFrom(cls);
```
This checks whether `cls` (or one of its ancestors) implements or extends `SomeInterface`.

**Q117. Can you get a `Class` object for a primitive type, like `int`?**
Yes — every primitive type has a corresponding `Class` object, accessible via `int.class`, `boolean.class`, etc. — useful specifically when looking up methods/constructors that take primitive parameters via Reflection.

**Q118. What does `Modifier.isPublic(cls.getModifiers())` check, and why is it structured this way (calling a static method with modifiers as an int)?**
It checks whether the class is declared `public`. Modifiers (public, private, static, final, etc.) are encoded together as a single integer using bit flags for compactness — the `Modifier` utility class provides static methods to decode and check individual flags out of that combined integer value.

---

# SECTION 12: Working with Fields (Q119–Q130)

## Basic

**Q119. What is the `Field` class used for?**
Represents one single variable (field) declared inside a class — lets you find out its name and type, and read or change its value on a specific object, at runtime.

**Q120. How do you get a specific field from a class?**
```java
Field ageField = Person.class.getDeclaredField("age");
```

**Q121. How do you read a field's value from an object using Reflection?**
```java
int value = (int) ageField.get(person); // reads "age" FROM this specific person object
```

## Intermediate

**Q122. How do you read or change a `private` field using Reflection?**
```java
Field nameField = Person.class.getDeclaredField("name");
nameField.setAccessible(true); // bypasses the normal "private" restriction
String name = (String) nameField.get(person);
nameField.set(person, "NewName");
```

**Q123. What does `setAccessible(true)` actually do?**
Tells Java to turn off the normal access check (public/private/protected) for this specific Field, Method, or Constructor object — without it, trying to access a private member throws `IllegalAccessException`.

**Q124. Why is `setAccessible(true)` considered risky?**
It directly breaks encapsulation — a core object-oriented principle where a class controls and protects its own internal data. This is powerful for frameworks, but risky in normal application code, since a private field could quietly change from somewhere unexpected.

**Q125. How do you find out a field's declared type using Reflection?**
```java
Field ageField = Person.class.getDeclaredField("age");
System.out.println(ageField.getType()); // prints: int
```

## Advanced

**Q126. Can Reflection reliably change the value of a `final` field?**
Generally no, or not reliably — Java and the JVM treat `final` fields specially (often for optimization reasons), so even with `setAccessible(true)` and calling `set()`, changing a final field's value often doesn't work as expected, or may be blocked depending on the specific situation.

**Q127. How would you list all fields of a class along with their types?**
```java
for (Field field : cls.getDeclaredFields()) {
    System.out.println(field.getName() + " : " + field.getType());
}
```

**Q128. Can you set a primitive field's value using `Field.set()`, and does it need special handling?**
Yes, but you must pass the value already boxed appropriately (Java handles the unboxing automatically) — e.g., `field.set(obj, 25)` for an `int` field works fine, since Java autoboxes/unboxes as needed during the reflective set operation.

**Q129. How would you copy all field values from one object to another of the same class, generically, using Reflection?**
```java
for (Field field : source.getClass().getDeclaredFields()) {
    field.setAccessible(true);
    field.set(target, field.get(source));
}
```

**Q130. Why might a framework like Jackson use Field-level Reflection (rather than getters/setters) to convert JSON into a Java object?**
It allows the framework to populate an object's data even without public getters/setters — directly writing to (often private) fields via Reflection, which can support simpler object designs and works consistently regardless of whether the class author chose to expose getters/setters for every field.

---

# SECTION 13: Working with Methods (Q131–Q142)

## Basic

**Q131. What is the `Method` class used for?**
Represents one single method declared inside a class — lets you find out its name, parameters, and return type, and actually call (invoke) it at runtime.

**Q132. How do you get a specific method from a class?**
```java
Method addMethod = Calculator.class.getDeclaredMethod("add", int.class, int.class);
```

**Q133. Why do you need to pass the parameter types when getting a method?**
Because a class might have multiple methods with the same name but different parameters (overloading) — Java needs the exact parameter types to know precisely which version you mean.

## Intermediate

**Q134. How do you call (invoke) a method using Reflection?**
```java
Object result = addMethod.invoke(calculator, 5, 3); // calls calculator.add(5, 3)
```

**Q135. What two things does `invoke()` need, and what does it return?**
The actual object to call the method ON, plus the arguments to pass — and it returns the result as a plain `Object`, which may need casting to the actual return type if needed.

**Q136. How do you call a `private` method using Reflection?**
```java
Method squareMethod = Calculator.class.getDeclaredMethod("square", int.class);
squareMethod.setAccessible(true);
Object result = squareMethod.invoke(calculator, 5);
```

**Q137. How do you call a static method using Reflection, and what's different about the invoke() call?**
```java
Object result = staticMethod.invoke(null, 5); // pass null as the object, since it's not tied to any instance
```

## Advanced

**Q138. What exception does `invoke()` throw if the method being called itself throws an exception, and why is this useful to know?**
`InvocationTargetException` — this WRAPS the original exception thrown inside the called method (not a Reflection problem itself). You can call `.getCause()` on it to retrieve the original, real exception.

**Q139. How do you get details about a method's parameters and return type?**
```java
System.out.println(method.getReturnType());
System.out.println(method.getParameterCount());
for (Class<?> type : method.getParameterTypes()) System.out.println(type);
```

**Q140. Can Reflection call a method with variable-length arguments (varargs)?**
Yes — when invoking a varargs method through Reflection, you typically pass the arguments packed as an array matching the varargs parameter type.

**Q141. How would you write a generic "call this method by name" utility, without knowing the exact class in advance?**
```java
public static Object callMethod(Object obj, String methodName, Object... args) throws Exception {
    Class<?>[] paramTypes = new Class<?>[args.length];
    for (int i = 0; i < args.length; i++) paramTypes[i] = args[i].getClass();
    Method method = obj.getClass().getMethod(methodName, paramTypes);
    return method.invoke(obj, args);
}
```

**Q142. Why is repeatedly calling `getDeclaredMethod()` inside a hot loop considered bad practice, and what's the fix?**
Looking up a method by name/parameters is a relatively expensive Reflection operation. The fix is to look it up ONCE (often cached in a field or a Map), and reuse that same `Method` object for repeated `invoke()` calls, rather than re-resolving it every single time.

---

# SECTION 14: Working with Constructors (Q143–Q152)

## Basic

**Q143. What is the `Constructor` class used for?**
Represents one specific constructor of a class — lets you create brand-new objects at runtime, even for a class your code never directly names with `new ClassName()`.

**Q144. How do you get a specific constructor?**
```java
Constructor<?> constructor = Person.class.getDeclaredConstructor(String.class, int.class);
```

**Q145. How do you create a new object using a Constructor object?**
```java
Object personObject = constructor.newInstance("Ananth", 26);
Person person = (Person) personObject; // cast back if needed
```

## Intermediate

**Q146. How do you create an object using a `private` constructor?**
```java
Constructor<?> constructor = Singleton.class.getDeclaredConstructor();
constructor.setAccessible(true);
Singleton instance = (Singleton) constructor.newInstance();
```

**Q147. Why is bypassing a private constructor with Reflection considered a notable limitation of the Singleton design pattern?**
A Singleton relies on a private constructor specifically to guarantee only one instance ever exists — but Reflection can bypass that restriction entirely, technically allowing a second instance to be forced into existence, which undermines the pattern's core guarantee.

**Q148. What is the older, deprecated way to create an object via Reflection, and why was it deprecated?**
`Person.class.newInstance()` — deprecated since Java 9 because it had messy exception-handling behavior (it could throw checked exceptions declared by the constructor without wrapping them consistently). The modern approach is to get the specific `Constructor` object first, then call `newInstance()` on that.

## Advanced

**Q149. Why does Reflection need the exact parameter types to find the right constructor, just like with methods?**
Because a class can have multiple overloaded constructors — Java needs the exact parameter types to uniquely identify which specific constructor you want to use.

**Q150. How would you write a generic "create object by class name" factory method?**
```java
public static Object createInstance(String className) throws Exception {
    Class<?> cls = Class.forName(className);
    Constructor<?> constructor = cls.getDeclaredConstructor();
    return constructor.newInstance();
}
```

**Q151. Can `newInstance()` throw a checked exception if the constructor's own code throws one?**
Yes — similar to `Method.invoke()`, it wraps such exceptions in an `InvocationTargetException`, from which you can retrieve the real underlying cause.

**Q152. Is this exactly how Spring creates your `@Component`/`@Service` beans?**
Yes, at a simplified level — Spring finds the right constructor for your class (often the no-argument one, or one it can supply dependencies for) using Reflection, and calls the equivalent of `newInstance()` to build your bean, all without your code (or Spring's own code) ever writing `new YourClass()` directly.

---

# SECTION 15: Annotations and Reflection (Q153–Q160)

**Q153. How does Reflection detect annotations on a class, field, or method?**
```java
if (cls.isAnnotationPresent(MyAnnotation.class)) {
    MyAnnotation annotation = cls.getAnnotation(MyAnnotation.class);
    System.out.println(annotation.value());
}
```

**Q154. Why does `@Retention(RetentionPolicy.RUNTIME)` matter so much for Reflection?**
By default, annotations are thrown away either right after compiling, or right after class loading — meaning Reflection wouldn't be able to see them while the program runs at all, unless explicitly told to keep them.

**Q155. What are the three RetentionPolicy options, and which ones can Reflection see?**
`SOURCE` (thrown away right after compiling — never visible to Reflection), `CLASS` (kept in the compiled .class file, but discarded before running — still not visible to Reflection), `RUNTIME` (kept all the way through — Reflection CAN see this one).

**Q156. What happens if you forget `@Retention(RetentionPolicy.RUNTIME)` on your own custom annotation, and then try to read it with Reflection?**
`isAnnotationPresent()` returns `false`, and `getAnnotation()` returns `null` — even though the annotation is clearly written in your source code — since it simply doesn't survive to runtime by default.

**Q157. Why do Spring's `@Component` and `@Autowired` annotations always use `RetentionPolicy.RUNTIME`?**
Because Spring's Reflection-based scanning needs to detect them while the application is actually running — without RUNTIME retention, Spring would never be able to see these annotations at all.

**Q158. How do you read annotations on a specific field or method, rather than the whole class?**
```java
Field field = cls.getDeclaredField("someField");
if (field.isAnnotationPresent(Autowired.class)) { ... }

Method method = cls.getDeclaredMethod("someMethod");
if (method.isAnnotationPresent(PostConstruct.class)) { ... }
```

**Q159. Can an annotation have parameters, and how do you read them via Reflection?**
Yes — e.g., `@Command(name = "add")` — you read them by calling the corresponding method on the retrieved annotation instance: `Command command = method.getAnnotation(Command.class); String name = command.name();`

**Q160. What is `@Target` used for, in relation to a custom annotation?**
It restricts WHERE an annotation is allowed to be used — e.g., `@Target(ElementType.FIELD)` means the annotation can only be applied to fields, not to classes or methods, and using it in the wrong place would cause a compile error.

---

# SECTION 16: Dynamic Class Loading (Q161–Q170)

## Basic

**Q161. What is Dynamic Class Loading?**
Loading a class by name, given as a String, chosen while the program is actually running — possibly a class that didn't even exist yet when your program was originally compiled.

**Q162. How do you dynamically load a class?**
```java
Class<?> cls = Class.forName("com.example.PaymentProcessor");
```

**Q163. What exception does `Class.forName()` throw if the class doesn't exist?**
A checked exception, `ClassNotFoundException` — you must catch it or declare it with `throws`.

## Intermediate

**Q164. Give a practical, real-world use case for Dynamic Class Loading.**
A plugin system, where a configuration file specifies the class name of a plugin to load — the application reads that name and loads/uses the class without ever having known about it while it was originally written.

**Q165. What is a classic, historically real example of Dynamic Class Loading in Java database programming?**
Loading a JDBC driver: `Class.forName("com.mysql.cj.jdbc.Driver");` — this loads the driver class, whose static initializer block registers it with Java's driver management system, all without directly importing or instantiating it in your code.

**Q166. Does `Class.forName()` immediately create an object of that class?**
No — it only loads and initializes the class itself (including running static initializer blocks). You still need to separately get a Constructor and call `newInstance()` to actually create an object.

## Advanced

**Q167. What's the difference between `Class.forName("com.example.Demo")` and simply writing `Demo.class`?**
`Class.forName()` uses a class name known only at runtime (as a String) and loads/initializes it immediately. `Demo.class` uses a name known at compile time, and only loads the class when it's actually first used, not necessarily immediately.

**Q168. Why is Dynamic Class Loading considered a key building block for frameworks like Spring?**
Because frameworks are written once and reused across countless different applications, each with completely different classes — the framework cannot know your class names in advance, so it needs to discover and load classes dynamically, by name, at runtime.

**Q169. Can `Class.forName()` be given an overload that controls whether the class is initialized immediately, and which class loader to use?**
Yes — `Class.forName(String name, boolean initialize, ClassLoader loader)` gives explicit control over both whether static initializers run immediately, and which specific ClassLoader should be used to find the class.

**Q170. What is a `ClassLoader`, and how does it relate to Dynamic Class Loading?**
A `ClassLoader` is the JVM component responsible for actually finding and loading class bytecode into memory. Dynamic Class Loading via `Class.forName()` ultimately relies on a ClassLoader behind the scenes to locate and read the requested class's bytecode from disk (or another source, like a network location or a plugin JAR file).

---

# SECTION 17: Invoking Methods & Creating Objects Dynamically — Putting It Together (Q171–Q180)

**Q171. Describe, step by step, the general pattern a framework like Spring follows to create and configure one of your beans.**
1. Discover the class name (via config or annotation scanning).
2. Dynamically load the class (`Class.forName()`).
3. Find the right constructor and create an instance (`getDeclaredConstructor()` + `newInstance()`).
4. Find fields marked with a dependency-injection annotation, and set their values (`getDeclaredFields()` + `setAccessible(true)` + `set()`).
5. Find and call any lifecycle methods (like ones marked `@PostConstruct`) (`getDeclaredMethods()` + `invoke()`).

**Q172. Write a small program that finds all methods on a class annotated with a custom `@Command` annotation, and calls each one automatically.**
```java
for (Method method : cls.getDeclaredMethods()) {
    if (method.isAnnotationPresent(Command.class)) {
        method.setAccessible(true);
        Object result = method.invoke(instance);
    }
}
```

**Q173. Why is Reflection considered slower than normal, direct method calls?**
Java has to do extra lookup and safety-checking work at runtime (finding the method, verifying access, matching arguments) that normal, direct code already has resolved at compile time.

**Q174. How do frameworks that use Reflection heavily minimize its performance cost?**
By caching the results of expensive Reflection lookups (like `Method`, `Field`, and `Constructor` objects) — doing the lookup work just once, typically at application startup, and reusing those cached objects for repeated operations, rather than re-resolving them on every single call.

**Q175. If Reflection is slower, why do frameworks like Spring still rely on it so heavily?**
Because the flexibility it provides — working with classes the framework has genuinely never seen before — isn't achievable any other way in plain Java. The performance cost is managed by doing the expensive lookups rarely (mostly at startup), not on every request.

**Q176. Can Reflection be used to build a simple dependency injection container from scratch? Outline the core idea.**
Yes — scan classes for a marker annotation, create instances via Reflection, inspect each instance's fields for an injection-marker annotation, resolve what object should be injected for each such field (often by matching the field's type to another managed instance), and use `Field.set()` to wire them together — this is genuinely the core mechanic behind how real DI containers work, simplified.

**Q177. What is `Proxy` (dynamic proxies), and how does it relate to Reflection?**
`java.lang.reflect.Proxy` lets you create, at runtime, an object that implements a given set of interfaces, where every method call is intercepted and routed to a handler you provide (`InvocationHandler`) — this is how many frameworks implement things like transparent logging, transaction management, or lazy loading, without the developer needing to write a real implementing class themselves.

**Q178. Give a practical example of where Java's own standard library uses dynamic proxies.**
`java.lang.reflect.Proxy` itself is used internally by things like RMI (Remote Method Invocation) and various frameworks (including parts of Spring's AOP support) to create proxy objects on the fly that implement a business interface, forwarding calls to real logic while adding cross-cutting behavior like transactions or security checks.

**Q179. Is it possible to modify a class's bytecode itself at runtime, beyond what standard Reflection allows?**
Yes, but this goes beyond the standard Reflection API — libraries like ByteBuddy, CGLIB, or ASM allow generating or modifying actual class bytecode at runtime (used by things like Hibernate's proxy generation, or Spring's CGLIB-based proxying for classes without interfaces) — a more advanced, specialized technique than the Field/Method/Constructor-based Reflection covered in this document.

**Q180. In a fintech context, why might a rules engine use Reflection to dynamically invoke different validation methods based on a transaction's type?**
Rather than hardcoding a long if/else chain checking transaction type and calling the matching validation method directly, a rules engine could look up the right validation method by name (perhaps derived from configuration, or an annotation on the method itself, like `@ValidatesType("WIRE_TRANSFER")`), then invoke it dynamically — making it much easier to add new transaction types and their validation logic without modifying and redeploying the core dispatching code.

---

# Quick-reference summary table

| Part | Section | Topic | Question range |
|---|---|---|---|
| 1 | 1 | Why Generics Exist | Q1–Q8 |
| 1 | 2 | Generic Classes | Q9–Q20 |
| 1 | 3 | Generic Methods | Q21–Q30 |
| 1 | 4 | Bounded Types | Q31–Q42 |
| 1 | 5 | Wildcards | Q43–Q56 |
| 1 | 6 | The PECS Principle | Q57–Q64 |
| 1 | 7 | Type Erasure | Q65–Q78 |
| 1 | 8 | Raw Types | Q79–Q88 |
| 1 | 9 | Mixed / Scenario-Based | Q89–Q98 |
| 2 | 10 | Why Reflection Exists | Q99–Q106 |
| 2 | 11 | The Class Class | Q107–Q118 |
| 2 | 12 | Working with Fields | Q119–Q130 |
| 2 | 13 | Working with Methods | Q131–Q142 |
| 2 | 14 | Working with Constructors | Q143–Q152 |
| 2 | 15 | Annotations and Reflection | Q153–Q160 |
| 2 | 16 | Dynamic Class Loading | Q161–Q170 |
| 2 | 17 | Invoking Methods, Putting It Together | Q171–Q180 |

**Total: 180 questions.**

---

## Suggested study strategy

1. **Generics — Sections 1, 2, 3** (why they exist, Generic Classes, Generic Methods) are the foundation — get comfortable here before moving on.
2. **Generics — Section 7 (Type Erasure)** is genuinely the highest-value topic in the whole Generics half — it explains the "why" behind nearly every strange rule (why no `new T()`, why raw types exist, why overloading by generic type fails). Interviewers love this topic because it separates people who memorized syntax from people who understand the mechanism.
3. **Generics — Sections 4, 5, 6** (Bounded Types, Wildcards, PECS) go together and are frequently tested with "does this compile?" style snippets — practice reading and predicting these quickly.
4. **Reflection — Section 10** (why it exists, the Spring connection) is the most common "explain in your own words" opener — a strong answer here sets the tone for the rest of the topic.
5. **Reflection — Sections 11–14** (Class, Field, Method, Constructor) are the mechanical core — practice writing this code by hand, especially `setAccessible(true)` and `invoke()`/`newInstance()` patterns.
6. **Reflection — Section 15 (`RetentionPolicy.RUNTIME`)** is a favorite, slightly tricky follow-up question that many developers overlook — know it cold.
7. **Reflection — Sections 16 and 17** (Dynamic Class Loading, putting it all together) are where you demonstrate real, practical understanding — be ready to describe the full Spring-style bean creation flow end to end, since this is often exactly what's asked to test genuine depth.
