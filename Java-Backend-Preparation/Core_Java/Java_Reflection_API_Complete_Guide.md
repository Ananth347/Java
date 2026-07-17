# Java Reflection API — The Complete Guide
### Basic to Advanced, explained in simple English, with code examples

---

## How to read this document

Read this from the top, in order. Each part builds on the one before it. For every topic you will get:
- **What it is** (in simple words)
- **Why it exists** (what problem it fixes)
- **How it works** (with easy examples)
- **A common interview question**

Topics covered: What Reflection is and why it matters · The `Class` class · Fields · Methods · Constructors · Full Reflection walkthrough · Dynamic Class Loading · Invoking Methods and Creating Objects Dynamically.

---

# PART 1: WHAT IS REFLECTION, AND WHY DOES IT MATTER?

## 1.1 What is Reflection?

Reflection is a feature in Java that lets your program **look at itself while it is running**. Using Reflection, code can examine classes, fields (variables), methods, and constructors — even ones it did not know about while it was being written — and it can even **call methods** or **create objects** dynamically, all at runtime.

## 1.2 A simple way to picture this

Normally, when you write Java code, you already know everything about the classes you are using — their names, their fields, their methods — because you wrote them, or you looked them up in documentation, and the compiler checks all of it before your program even runs.

Reflection flips this around: instead of knowing everything in advance, your program can **ask questions about a class while it is running** — "What fields does this class have?", "What methods can I call on this object?" — and then act on the answers, even for a class it has never seen before.

## 1.3 A first, very simple example

```java
class Person {
    private String name = "Ananth";
}

public class Demo {
    public static void main(String[] args) throws Exception {
        Person person = new Person();

        Class<?> personClass = person.getClass(); // asking Java: "what class is this object?"
        System.out.println(personClass.getName()); // prints: Person
    }
}
```
Here, we are not directly reading the `Person` class ourselves — we are asking the object itself, at runtime, "what class are you?" — and Java tells us.

## 1.4 Why does Reflection matter so much? Why is it used everywhere in Spring Boot?

This is the big, practical reason to learn Reflection well. Think about how Spring Boot actually works:

```java
@Component
class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

When your application starts, Spring does **not** know your class names in advance — it was not written specifically for `UserService`. Instead, at startup, Spring:
1. **Scans** your project to find classes marked with annotations like `@Component`.
2. Uses **Reflection** to look at each class's fields, and finds ones marked `@Autowired`.
3. Uses **Reflection** to create an instance of the class (even though it never saw `UserService` while Spring itself was being written).
4. Uses **Reflection** to set the value of the `userRepository` field directly, even though it is `private`.

None of this would be possible without Reflection — Spring genuinely could not work the way it does without the ability to inspect and act on classes it has never seen before, at runtime. This single reason is exactly why Reflection is considered such an important topic — almost every major Java framework (Spring, Hibernate, JUnit, Jackson) depends on it heavily behind the scenes.

## 1.5 What is the trade-off? Is Reflection always a good idea?

Reflection is powerful, but it comes with real costs, which is why you should not use it for everyday, normal code:
- **Slower** — Reflection operations are noticeably slower than writing normal, direct code, since Java has to do extra work figuring things out at runtime instead of already knowing them from compile time.
- **Less safe** — since the compiler cannot check reflective code the normal way (like checking if a method actually exists), many mistakes only show up when the program actually runs, as a runtime exception — instead of being caught early, at compile time.
- **Breaks encapsulation** — Reflection can access `private` fields and methods directly, ignoring the access rules the original class author intended, which can lead to fragile, hard-to-maintain code if overused.

### Common interview angle
**Q: Why does Spring Boot rely so heavily on Reflection?**
Because Spring needs to work with classes it has never seen before — your own application's classes — and it needs to create objects, read annotations, and set field values for these unknown classes at runtime. Reflection is the only way to do this, since Spring cannot know your class names or structure in advance.

---

# PART 2: THE `Class` CLASS — YOUR STARTING POINT

## 2.1 What is the `Class` class?

This is a genuinely confusing name at first, so let's be careful: `Class` (with a capital C) is itself a real Java class, and every single class in Java — including your own custom classes — automatically has one, single `Class` object that represents it at runtime. This `Class` object holds all the metadata (information) about that class: its name, its fields, its methods, its constructors, and more.

## 2.2 Three ways to get a `Class` object

```java
// Way 1: from an actual object, using getClass()
Person person = new Person();
Class<?> class1 = person.getClass();

// Way 2: using the .class syntax, if you already know the class name at compile time
Class<?> class2 = Person.class;

// Way 3: using Class.forName(), with just the class name as TEXT — this is how Reflection can work
// with a class name that is only known at RUNTIME, not written directly in your code
Class<?> class3 = Class.forName("Person");
```

## 2.3 Why are there three different ways? When do you use each one?

- `object.getClass()` — use this when you already have an actual object in hand, and want to find out its exact class.
- `ClassName.class` — use this when you already know the class name while writing your code (this is the simplest, safest option, and the compiler checks it).
- `Class.forName("ClassName")` — use this when the class name is only available as a **String**, often coming from a configuration file, a database, or user input — the class name is not known until the program is actually running. This is the option most used by frameworks like Spring, since they typically read class names out of configuration or annotations, not from hardcoded Java code.

## 2.4 What can you find out from a `Class` object?

```java
Class<?> cls = Person.class;

System.out.println(cls.getName());          // fully qualified name, like "com.example.Person"
System.out.println(cls.getSimpleName());    // just the short name, like "Person"
System.out.println(cls.getSuperclass());    // the parent class
System.out.println(cls.isInterface());      // true or false
System.out.println(Modifier.isPublic(cls.getModifiers())); // checks if the class is public
```

## 2.5 Getting the list of Fields, Methods, and Constructors

```java
Class<?> cls = Person.class;

Field[] fields = cls.getDeclaredFields();           // all fields, including private ones
Method[] methods = cls.getDeclaredMethods();          // all methods, including private ones
Constructor<?>[] constructors = cls.getDeclaredConstructors(); // all constructors
```

### Common interview angle
**Q: What's the difference between `getFields()` and `getDeclaredFields()`?**
`getFields()` only returns **public** fields, but it includes ones inherited from parent classes too. `getDeclaredFields()` returns **all** fields declared directly in that exact class — public, private, and protected — but it does NOT include fields inherited from a parent class. This same pattern (`getX()` vs `getDeclaredX()`) applies to Methods and Constructors too.

---

# PART 3: WORKING WITH FIELDS

## 3.1 What is the `Field` class?

`Field` is a class in the Reflection API that represents one single variable (field) declared inside a class. Using a `Field` object, you can find out its name, its type, and — importantly — you can read or even change its value on a specific object, at runtime.

## 3.2 Getting a specific field

```java
class Person {
    private String name = "Ananth";
    public int age = 26;
}

Class<?> cls = Person.class;

Field nameField = cls.getDeclaredField("name"); // gets the field named "name"
Field ageField = cls.getDeclaredField("age");
```

## 3.3 Reading a field's value from an object

```java
Person person = new Person();
Field ageField = Person.class.getDeclaredField("age");

int value = (int) ageField.get(person); // reads the "age" field's value FROM this specific person object
System.out.println(value); // 26
```

## 3.4 Reading and changing a `private` field — the famous Reflection trick

This is one of the most well-known things Reflection can do, and it is exactly what makes it both powerful and a little dangerous.

```java
Person person = new Person();
Field nameField = Person.class.getDeclaredField("name");

nameField.setAccessible(true); // THIS is the key step — it bypasses the normal "private" restriction

String name = (String) nameField.get(person); // now we CAN read the private field
System.out.println(name); // Ananth

nameField.set(person, "Bora"); // we can even CHANGE the private field's value directly!
System.out.println(person.getClass().getDeclaredField("name").get(person)); // Bora
```

## 3.5 What does `setAccessible(true)` actually do, and why is it needed?

Normally, Java's access rules (`private`, `protected`, `public`) are enforced strictly — code outside the class is simply not allowed to touch a `private` field directly. `setAccessible(true)` tells Java: "please turn off that access check for this specific `Field` (or `Method`, or `Constructor`) object, just for me." Without calling this first, trying to `get()` or `set()` a private field throws an `IllegalAccessException`.

### Why is this considered risky?
This directly breaks **encapsulation** — one of the core ideas of object-oriented programming, where a class is supposed to control and protect its own internal data. Reflection lets you bypass that protection entirely. This is powerful for frameworks (which genuinely need this ability to work), but risky if used carelessly in regular application code, since it can make code fragile and hard to reason about — a private field could quietly change from somewhere completely unexpected.

## 3.6 Finding a field's type

```java
Field ageField = Person.class.getDeclaredField("age");
System.out.println(ageField.getType()); // prints: int
```

### Common interview angle
**Q: If a field is `final`, can Reflection still change its value?**
Generally, no — or at least, not reliably. Java (and the JVM) treats `final` fields specially, especially for optimization reasons, so even using `setAccessible(true)` and `set()`, changing a `final` field's value often does not work as expected, or may be blocked entirely depending on the exact situation. This is a well-known limitation worth mentioning if asked.

---

# PART 4: WORKING WITH METHODS

## 4.1 What is the `Method` class?

`Method` is a class in the Reflection API that represents one single method declared inside a class. Using a `Method` object, you can find out its name, its parameter types, its return type — and, importantly, you can actually **call (invoke) it**, at runtime, even on an object your code did not create directly by name.

## 4.2 Getting a specific method

```java
class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}

Class<?> cls = Calculator.class;

// You must tell Java the method's name AND its exact parameter types, since methods can be overloaded
Method addMethod = cls.getDeclaredMethod("add", int.class, int.class);
```

### Why do you need to pass the parameter types too?
Because a class might have several methods with the **same name** but different parameters (this is called overloading) — like `add(int, int)` and `add(double, double)`. Java needs the exact parameter types to know precisely which version of the method you mean.

## 4.3 Calling (invoking) a method using Reflection

```java
Calculator calculator = new Calculator();
Method addMethod = Calculator.class.getDeclaredMethod("add", int.class, int.class);

Object result = addMethod.invoke(calculator, 5, 3); // calls calculator.add(5, 3) — but through Reflection
System.out.println(result); // 8
```

Notice that `invoke()` needs two things: the actual object to call the method ON (`calculator`), and the arguments to pass to it (`5, 3`). The result comes back as a plain `Object`, since Reflection cannot know the specific return type at compile time — you may need to cast it to the real type if you need it.

## 4.4 Calling a `private` method using Reflection

Exactly like with private fields, you need `setAccessible(true)` first.

```java
class Calculator {
    private int square(int x) {
        return x * x;
    }
}

Calculator calculator = new Calculator();
Method squareMethod = Calculator.class.getDeclaredMethod("square", int.class);
squareMethod.setAccessible(true); // required, since square() is private

Object result = squareMethod.invoke(calculator, 5);
System.out.println(result); // 25
```

## 4.5 Calling a static method using Reflection

```java
class MathUtils {
    public static int square(int x) {
        return x * x;
    }
}

Method staticMethod = MathUtils.class.getDeclaredMethod("square", int.class);
Object result = staticMethod.invoke(null, 5); // pass "null" as the object, since static methods don't belong to an instance
System.out.println(result); // 25
```
Notice: for a static method, the first argument to `invoke()` is `null`, since a static method is not tied to any particular object.

## 4.6 Getting details about a method

```java
Method addMethod = Calculator.class.getDeclaredMethod("add", int.class, int.class);

System.out.println(addMethod.getName());          // "add"
System.out.println(addMethod.getReturnType());    // "int"
System.out.println(addMethod.getParameterCount()); // 2
for (Class<?> paramType : addMethod.getParameterTypes()) {
    System.out.println(paramType); // "int", "int"
}
```

### Common interview angle
**Q: What exception does `invoke()` throw, and why is that useful to know?**
`invoke()` can throw an `InvocationTargetException` — this specifically wraps any exception that happened INSIDE the method you called (not a problem with Reflection itself, but a real error thrown by the method's own code). You can call `.getCause()` on it to get back the original, real exception that actually happened inside the method.

---

# PART 5: WORKING WITH CONSTRUCTORS

## 5.1 What is the `Constructor` class?

`Constructor` is a class in the Reflection API that represents one specific constructor of a class. Using it, you can create brand-new objects at runtime — even for a class your code never directly names with `new ClassName()`.

## 5.2 Getting a specific constructor

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

Class<?> cls = Person.class;
Constructor<?> constructor = cls.getDeclaredConstructor(String.class, int.class);
```
Just like with methods, you must specify the exact parameter types, in case the class has more than one constructor (constructor overloading).

## 5.3 Creating a new object using Reflection

```java
Constructor<?> constructor = Person.class.getDeclaredConstructor(String.class, int.class);
Object personObject = constructor.newInstance("Ananth", 26);

Person person = (Person) personObject; // cast back to the real type, if you need to use it normally afterward
```

This is genuinely important: this is exactly how Spring creates your `@Component`/`@Service` classes — it finds the right constructor using Reflection, and calls `newInstance()` to build the object, all without your code (or Spring's own code) ever writing `new UserService()` directly.

## 5.4 Creating an object using a `private` constructor

```java
class Singleton {
    private Singleton() { } // private constructor, meant to block normal "new Singleton()" calls
}

Constructor<?> constructor = Singleton.class.getDeclaredConstructor();
constructor.setAccessible(true); // bypass the private restriction

Singleton instance = (Singleton) constructor.newInstance();
```
This is a well-known (and slightly controversial) trick: Reflection can bypass even a `private` constructor meant specifically to prevent an object from being created — for example, a class specifically designed to only ever allow ONE instance (a Singleton pattern) can technically still have a second instance forced into existence using Reflection, which is worth knowing as a real limitation of that design pattern.

## 5.5 A simpler shortcut — `Class.newInstance()` and its problem

An older way to create an object was:
```java
Person person = (Person) Person.class.newInstance(); // OLD, DEPRECATED approach
```
This only works for a class with a public, no-argument (empty parentheses) constructor, and it has been **deprecated** since Java 9, because it had messy exception-handling behavior. The modern, recommended approach is always to properly get the specific `Constructor` object first, and call `newInstance()` on THAT:
```java
Person person = (Person) Person.class.getDeclaredConstructor().newInstance(); // modern, correct approach
```

### Common interview angle
**Q: Why does Reflection need to know the exact parameter types to find the right constructor/method?**
Because Java supports overloading (multiple constructors or methods with the same name, but different parameters). Reflection has no way to guess which specific version you mean just from the name alone — it needs the exact parameter types to uniquely identify the one you want.

---

# PART 6: A FULL REFLECTION WALKTHROUGH

## 6.1 Putting fields, methods, and constructors together

Here's one complete, small example, showing everything from Parts 2 through 5 working together on a single class:

```java
class Employee {
    private String name;
    private double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    private double calculateBonus() {
        return salary * 0.1;
    }
}

public class ReflectionDemo {
    public static void main(String[] args) throws Exception {
        Class<?> cls = Class.forName("Employee"); // getting the Class object by name (as text)

        // Create a new object using a constructor found through Reflection
        Constructor<?> constructor = cls.getDeclaredConstructor(String.class, double.class);
        Object employeeObject = constructor.newInstance("Ananth", 50000.0);

        // Read a private field's value
        Field nameField = cls.getDeclaredField("name");
        nameField.setAccessible(true);
        System.out.println("Name: " + nameField.get(employeeObject));

        // Call a private method
        Method bonusMethod = cls.getDeclaredMethod("calculateBonus");
        bonusMethod.setAccessible(true);
        Object bonus = bonusMethod.invoke(employeeObject);
        System.out.println("Bonus: " + bonus);
    }
}
```
Output:
```
Name: Ananth
Bonus: 5000.0
```

Notice: we never wrote `new Employee(...)` directly in our code — everything, including creating the object, reading a private field, and calling a private method, was done entirely through Reflection.

## 6.2 Reading Annotations using Reflection

Reflection can also detect **annotations** on a class, field, or method — this is exactly how Spring finds your `@Component`, `@Autowired`, and `@RequestMapping` annotations.

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME) // IMPORTANT — explained in Part 6.3 below
@interface MyAnnotation {
    String value();
}

@MyAnnotation("This is a demo class")
class Demo { }

Class<?> cls = Demo.class;
if (cls.isAnnotationPresent(MyAnnotation.class)) {
    MyAnnotation annotation = cls.getAnnotation(MyAnnotation.class);
    System.out.println(annotation.value()); // "This is a demo class"
}
```

## 6.3 Why does `@Retention(RetentionPolicy.RUNTIME)` matter so much here?

This is a small detail that trips a lot of people up, but it's genuinely important. Annotations, by default, only exist in your source code and are thrown away either right after compiling, or right after the class is loaded — meaning Reflection would not be able to see them at all while the program is actually running, unless you explicitly tell Java to keep them around.

```java
@Retention(RetentionPolicy.SOURCE)  // thrown away immediately after compiling — Reflection can NEVER see this one
@Retention(RetentionPolicy.CLASS)   // kept in the compiled .class file, but still thrown away before running — Reflection still can't see it
@Retention(RetentionPolicy.RUNTIME) // kept all the way through — Reflection CAN see this one while the program runs
```
This is exactly why annotations like Spring's `@Component` and `@Autowired` are always declared with `RetentionPolicy.RUNTIME` — without it, Spring's Reflection-based scanning would never be able to detect them at all.

### Common interview angle
**Q: If you write your own custom annotation but forget `@Retention(RetentionPolicy.RUNTIME)`, what happens when you try to read it with Reflection?**
`isAnnotationPresent()` will simply return `false`, and `getAnnotation()` will return `null`, even though the annotation is clearly written right there in your source code — because, by default, annotations do not survive all the way to runtime, and Reflection can only see what is actually available while the program is running.

---

# PART 7: DYNAMIC CLASS LOADING

## 7.1 What is Dynamic Class Loading?

Normally, when your Java program starts, it already knows (from your written code) exactly which classes it will use, and the JVM loads them as needed while running. **Dynamic Class Loading** means loading a class **by name, as a String**, chosen while the program is actually running — possibly a class that did not even exist yet when your program was originally written and compiled.

## 7.2 The basic way — `Class.forName()`

```java
Class<?> cls = Class.forName("com.example.PaymentProcessor");
```
This one line does two important things: it finds the class file with that exact name, and it **loads and initializes** it into the JVM — including running any static initializer blocks inside that class — even though your code never wrote `import com.example.PaymentProcessor;` or referenced it directly anywhere.

## 7.3 Why is this genuinely useful? A real, practical example

Imagine you are building a plugin system for an application, where users can add new features by dropping in their own custom classes, and your application should be able to pick them up **without you needing to know their class names in advance**.

```java
// Read the class name from a configuration file, at runtime
String className = configFile.getProperty("plugin.class"); // e.g., "com.plugins.EmailPlugin"

Class<?> pluginClass = Class.forName(className); // load whatever class the config file happens to name
Object plugin = pluginClass.getDeclaredConstructor().newInstance(); // create an instance of it

Method runMethod = pluginClass.getMethod("run");
runMethod.invoke(plugin); // run the plugin, even though your code has never seen this exact class before
```
This exact pattern — reading a class name from configuration, then loading and using it through Reflection — is genuinely how many real plugin systems and frameworks (including parts of Spring and JDBC drivers) actually work.

## 7.4 A classic real-world use — loading a JDBC database driver

Before newer JDBC versions made this automatic, developers had to manually write:
```java
Class.forName("com.mysql.cj.jdbc.Driver");
```
This line loads the MySQL driver class, which — through its static initializer block — registers itself with Java's driver management system, all without your code ever writing `new Driver()` directly, or importing that class by name anywhere in your source code.

## 7.5 What happens if the class name doesn't exist?

`Class.forName()` throws a checked exception, `ClassNotFoundException`, if no class with that exact name can be found — this is a **checked** exception, meaning you must either catch it or declare it with `throws`, since class names coming from outside sources (like a configuration file) are genuinely likely to sometimes be wrong or missing.

```java
try {
    Class<?> cls = Class.forName("com.example.NonExistentClass");
} catch (ClassNotFoundException e) {
    System.out.println("Could not find that class: " + e.getMessage());
}
```

## 7.6 `Class.forName()` vs simply writing `SomeClass.class`

| | `Class.forName("com.example.Demo")` | `Demo.class` |
|---|---|---|
| When is the class name known | Only at runtime — comes from a String | At compile time — written directly in your code |
| Does it load/initialize the class right away | Yes, immediately (by default) | No, only when the class is actually first used |
| Typical use case | Frameworks, plugins, config-driven systems | Everyday, ordinary code |

### Common interview angle
**Q: Why is Dynamic Class Loading considered a key building block for frameworks like Spring?**
Because frameworks are written once, and then reused across countless different applications, each with completely different classes. The framework's own code cannot possibly know your class names in advance — Dynamic Class Loading (`Class.forName()`, combined with Reflection to inspect and use the loaded class) is exactly what lets a generic framework work with classes it has genuinely never seen before.

---

# PART 8: INVOKING METHODS AND CREATING OBJECTS DYNAMICALLY — TYING IT ALL TOGETHER

## 8.1 The full, general pattern frameworks like Spring actually follow

Now that you've seen each individual piece, here is how they combine into the general pattern that a framework like Spring genuinely follows when starting up your application:

```java
// Step 1: Discover the class name (from a config file, an annotation scan, or similar)
String className = "com.example.UserService";

// Step 2: Dynamically load the class
Class<?> cls = Class.forName(className);

// Step 3: Find the right constructor, and create a new instance
Constructor<?> constructor = cls.getDeclaredConstructor();
Object instance = constructor.newInstance();

// Step 4: Find fields marked with a specific annotation (like @Autowired), and set their values
for (Field field : cls.getDeclaredFields()) {
    if (field.isAnnotationPresent(Autowired.class)) {
        field.setAccessible(true);
        Object dependency = resolveDependency(field.getType()); // find the right object to inject
        field.set(instance, dependency);
    }
}

// Step 5: Find and call a specific method (like one marked @PostConstruct)
for (Method method : cls.getDeclaredMethods()) {
    if (method.isAnnotationPresent(PostConstruct.class)) {
        method.setAccessible(true);
        method.invoke(instance);
    }
}
```
This is, at a simplified level, genuinely close to what Spring is actually doing behind the scenes for every single `@Component` in your application — load the class, create it, inspect its fields and methods for annotations, and call/set them as needed, all through Reflection.

## 8.2 A simple, complete example — a tiny "mini-framework" using Reflection

```java
import java.lang.annotation.*;
import java.lang.reflect.*;

@Retention(RetentionPolicy.RUNTIME)
@interface Command {
    String name();
}

class Calculator {
    @Command(name = "add")
    public int add(int a, int b) {
        return a + b;
    }

    @Command(name = "multiply")
    public int multiply(int a, int b) {
        return a * b;
    }
}

public class MiniFramework {
    public static void main(String[] args) throws Exception {
        Class<?> cls = Calculator.class;
        Object instance = cls.getDeclaredConstructor().newInstance();

        // Find every method marked with @Command, and run it automatically
        for (Method method : cls.getDeclaredMethods()) {
            if (method.isAnnotationPresent(Command.class)) {
                Command command = method.getAnnotation(Command.class);
                Object result = method.invoke(instance, 5, 3);
                System.out.println(command.name() + " -> " + result);
            }
        }
    }
}
```
Output:
```
add -> 8
multiply -> 15
```
This tiny example is genuinely the same basic idea behind much bigger frameworks — scanning for annotated methods, and calling them automatically without hardcoding their names anywhere.

## 8.3 Performance consideration — should you use Reflection in a tight loop?

Reflection operations (especially `invoke()` and `newInstance()`) are meaningfully slower than calling a method or creating an object normally, since Java has to do extra lookup and safety-checking work at runtime that normal code already has figured out at compile time. For code that runs a huge number of times very quickly (like inside a tight loop processing millions of items), this overhead can genuinely matter. Frameworks that use Reflection heavily often work around this with caching (looking up a `Method` or `Constructor` object just once, and reusing that same object many times) rather than calling `getDeclaredMethod()` repeatedly.

### Common interview angle
**Q: If Reflection is slower, why do frameworks like Spring still use it so heavily, instead of avoiding it?**
Because the flexibility Reflection provides — being able to work with classes the framework has never seen before — is not something you can get any other way in plain Java. In practice, frameworks minimize the performance cost by doing most of the expensive Reflection work (finding classes, methods, fields, annotations) just **once**, typically at application startup, rather than repeatedly during normal request handling — so the slower parts happen only rarely, not on every single request.

---

# Quick-reference summary table

| Concept | One-line meaning |
|---|---|
| Reflection | Letting a program inspect and act on classes, fields, and methods at runtime |
| `Class` class | The starting point — one object per class, holding all its metadata |
| `Class.forName("...")` | Loads a class by name, given only as a String, at runtime |
| `Field` | Represents one variable in a class — can read/change its value, even if private |
| `Method` | Represents one method in a class — can call (invoke) it, even if private |
| `Constructor` | Represents one constructor — can create new objects, even from a private constructor |
| `setAccessible(true)` | Bypasses Java's normal private/protected access rules |
| `RetentionPolicy.RUNTIME` | Required on a custom annotation for Reflection to be able to see it while running |
| Dynamic Class Loading | Loading and using a class whose name is only known while the program runs |
| `invoke()` | Calls a method found through Reflection, on a given object, with given arguments |
| `newInstance()` | Creates a new object using a `Constructor` found through Reflection |

---

## Suggested study order for interview prep

1. **Part 1** (what Reflection is, and why Spring needs it) — this is the most commonly asked "explain in your own words" question, and understanding the Spring connection makes your answer sound like real understanding, not memorized definitions.
2. **Part 2** (the `Class` class, and the three ways to get one) — the starting point for everything else; know `Class.forName()` well, since it connects directly to Dynamic Class Loading later.
3. **Part 3 and 4** (Fields and Methods, including `setAccessible(true)`) — very commonly tested with small code examples; be ready to explain exactly why `setAccessible(true)` is needed and what risk it introduces.
4. **Part 5** (Constructors, and `newInstance()`) — ties directly into how Spring actually creates your beans; good to practice writing this kind of code by hand.
5. **Part 6.2 and 6.3** (Reading annotations, and `RetentionPolicy.RUNTIME`) — a favorite, slightly trickier follow-up question, since it explains a subtle but important detail many developers overlook.
6. **Part 7** (Dynamic Class Loading) — good to connect clearly back to real examples, like JDBC drivers or plugin systems, to show practical understanding.
7. **Part 8** (putting it all together, and the performance trade-off) — a strong way to end an interview answer on this topic, showing you understand both the power AND the cost of using Reflection.
