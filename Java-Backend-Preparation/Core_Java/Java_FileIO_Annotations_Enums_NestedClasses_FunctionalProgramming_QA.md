# Java File I/O, Annotations, Enums, Nested Classes & Functional Programming — Complete Interview Q&A
### Basic → Advanced, every topic covered, with clear explanations

---

## How to use this document
Questions are grouped by topic and move from basic to hard within each section. Read a section fully before moving to the next.

**Part 1 — File Handling (I/O):** File Class · Byte Streams · Character Streams · Buffered Streams · Serialization · Deserialization · NIO · Paths · Files API.
**Part 2 — Annotations:** Built-in Annotations · Custom Annotations · Meta Annotations.
**Part 3 — Enums:** Enum Basics · Enum Methods · Enum with Constructor · Enum with Interface.
**Part 4 — Nested Classes:** Static Nested Class · Inner Class · Local Inner Class · Anonymous Class.
**Part 5 — Functional Programming:** Predicate · Function · Consumer · Supplier · BiFunction · BinaryOperator · UnaryOperator.

---
---

# PART 1: FILE HANDLING (I/O)

---

# SECTION 1: The `File` Class (Q1–Q12)

## Basic

**Q1. What is the `File` class used for?**
It represents a path to a file or directory on the filesystem — but it's important to know that a `File` object does NOT necessarily mean the file actually exists; it's just a representation of a path, which you can then use to check, create, delete, or inspect that location.

**Q2. How do you create a `File` object?**
```java
File file = new File("data.txt");
File file2 = new File("/home/user/documents/data.txt"); // with a full path
```

**Q3. How do you check if a file actually exists?**
```java
File file = new File("data.txt");
boolean exists = file.exists();
```

**Q4. How do you check if a `File` object represents a file or a directory?**
```java
file.isFile();       // true if it's a regular file
file.isDirectory();  // true if it's a directory
```

## Intermediate

**Q5. How do you create a new, empty file using the `File` class?**
```java
File file = new File("newFile.txt");
boolean created = file.createNewFile(); // returns false if the file already existed
```

**Q6. How do you create a new directory (or nested directories) using `File`?**
```java
File dir = new File("myFolder");
dir.mkdir();      // creates ONE directory — fails if parent folders don't exist
dir.mkdirs();     // creates the directory AND any missing parent directories along the way
```

**Q7. How do you delete a file or directory using `File`?**
```java
file.delete(); // returns true if successfully deleted, false otherwise
```

**Q8. How do you list all files inside a directory?**
```java
File dir = new File("myFolder");
String[] fileNames = dir.list();       // just the names
File[] files = dir.listFiles();         // full File objects
```

**Q9. How do you get a file's size, and its last modified time?**
```java
long sizeInBytes = file.length();
long lastModified = file.lastModified(); // a timestamp (milliseconds since epoch)
```

## Advanced

**Q10. What is the difference between an absolute path and a relative path in `File`?**
An absolute path fully specifies a location from the root of the filesystem (like `/home/user/data.txt`), while a relative path is interpreted relative to wherever the program is currently running from (the "working directory") — like just `data.txt`.
```java
System.out.println(file.getAbsolutePath()); // converts to a full absolute path
```

**Q11. Why is the `File` class considered outdated compared to the newer NIO `Path`/`Files` API?**
`File` has limited error reporting (many methods just return `true`/`false` on failure, with no details about WHY something failed), no built-in support for symbolic links, and generally weaker capabilities compared to the richer, more informative NIO API introduced in Java 7 (covered later in this document).

**Q12. Does creating a `File` object actually create anything on disk?**
No — creating a `new File("path")` only creates an in-memory Java object representing that path; nothing happens on the actual filesystem until you explicitly call a method like `createNewFile()`, `mkdir()`, or use a stream to write to it.

---

# SECTION 2: Byte Streams (Q13–Q24)

## Basic

**Q13. What are Byte Streams used for?**
Reading and writing raw binary data, one byte (or a chunk of bytes) at a time — used for any kind of file, not just text: images, videos, PDFs, or any binary format.

**Q14. What are the two main base classes for Byte Streams?**
`InputStream` (for reading bytes) and `OutputStream` (for writing bytes) — both are abstract classes, with many specific subclasses for different sources/destinations.

**Q15. How do you read a file byte-by-byte using `FileInputStream`?**
```java
try (FileInputStream fis = new FileInputStream("data.bin")) {
    int byteData;
    while ((byteData = fis.read()) != -1) { // read() returns -1 at end of file
        System.out.println(byteData);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

**Q16. How do you write bytes to a file using `FileOutputStream`?**
```java
try (FileOutputStream fos = new FileOutputStream("output.bin")) {
    fos.write(65); // writes a single byte (65 = 'A' in ASCII)
} catch (IOException e) {
    e.printStackTrace();
}
```

## Intermediate

**Q17. Why does `InputStream.read()` return an `int` instead of a `byte`?**
Because a single byte can only hold values 0–255, but `read()` needs an extra possible value, `-1`, to signal "end of file reached" — an `int` is used so that `-1` can be distinguished from every valid byte value.

**Q18. How do you read multiple bytes at once, into an array, for better performance?**
```java
byte[] buffer = new byte[1024];
int bytesRead;
while ((bytesRead = fis.read(buffer)) != -1) {
    // process bytesRead bytes from "buffer"
}
```

**Q19. Why is reading in chunks (into a byte array) faster than reading one byte at a time?**
Each individual `read()` call has real overhead (potentially involving a system call to the operating system). Reading many bytes in a single call amortizes that overhead across a whole chunk of data, instead of paying it separately for every single byte.

**Q20. What does `FileOutputStream`'s constructor's second boolean parameter do?**
```java
new FileOutputStream("data.txt", true); // "true" means APPEND mode
```
If `true`, new data is appended to the end of the existing file's content. If `false` (or omitted), the file is overwritten, discarding whatever was there before.

## Advanced

**Q21. What is the class hierarchy relationship between `FileInputStream` and `InputStream`?**
`FileInputStream` is a concrete subclass of the abstract `InputStream` class, specifically for reading bytes from a file — `InputStream` itself defines the general contract (like the `read()` method), while `FileInputStream` provides the actual file-reading implementation.

**Q22. Why should you always close a stream, and what's the modern recommended way to do it?**
Streams hold onto real operating system resources (file handles); forgetting to close them can lead to resource leaks, and on some systems, prevent the file from being accessed by other programs until closed. The modern recommended way is try-with-resources, which automatically closes the stream regardless of whether an exception occurs.
```java
try (FileInputStream fis = new FileInputStream("data.bin")) {
    // use fis
} // automatically closed here
```

**Q23. How would you copy a binary file from one location to another using Byte Streams?**
```java
try (FileInputStream fis = new FileInputStream("source.bin");
     FileOutputStream fos = new FileOutputStream("destination.bin")) {
    byte[] buffer = new byte[1024];
    int bytesRead;
    while ((bytesRead = fis.read(buffer)) != -1) {
        fos.write(buffer, 0, bytesRead);
    }
}
```

**Q24. Are Byte Streams suitable for reading text files?**
Technically yes, but not ideal — text involves character encoding (like UTF-8), which can use more than one byte per character. Byte Streams work purely with raw bytes and don't understand character encoding at all — for text, Character Streams (covered next) are the more appropriate, purpose-built tool.

---

# SECTION 3: Character Streams (Q25–Q34)

## Basic

**Q25. What are Character Streams used for?**
Reading and writing TEXT data specifically, with built-in understanding of character encoding (like UTF-8) — unlike Byte Streams, which just work with raw, encoding-unaware bytes.

**Q26. What are the two main base classes for Character Streams?**
`Reader` (for reading characters) and `Writer` (for writing characters) — the character-based equivalents of `InputStream`/`OutputStream`.

**Q27. How do you read text from a file using `FileReader`?**
```java
try (FileReader reader = new FileReader("data.txt")) {
    int charData;
    while ((charData = reader.read()) != -1) {
        System.out.print((char) charData);
    }
}
```

**Q28. How do you write text to a file using `FileWriter`?**
```java
try (FileWriter writer = new FileWriter("output.txt")) {
    writer.write("Hello, World!");
}
```

## Intermediate

**Q29. What is the core difference between Byte Streams and Character Streams?**
Byte Streams work with raw 8-bit bytes, with no understanding of text encoding. Character Streams work with 16-bit Unicode characters, and automatically handle the conversion between raw bytes and characters using a specified (or default) character encoding — making them the correct choice for text data.

**Q30. How do you specify a specific character encoding (like UTF-8) when reading/writing text?**
Use `InputStreamReader`/`OutputStreamWriter` as a "bridge," which lets you specify an encoding explicitly, since plain `FileReader`/`FileWriter` always use the platform's default encoding, which can vary between systems.
```java
try (InputStreamReader reader = new InputStreamReader(new FileInputStream("data.txt"), StandardCharsets.UTF_8)) {
    // reads with UTF-8 encoding explicitly, regardless of platform default
}
```

**Q31. Why is relying on the platform's default encoding (via plain `FileReader`) considered risky?**
Different operating systems/configurations can have different default character encodings — code that works correctly on one machine (say, one defaulting to UTF-8) might read text incorrectly (garbled characters) on another machine with a different default encoding, if the encoding isn't specified explicitly.

## Advanced

**Q32. What is `InputStreamReader`, and why is it described as a "bridge"?**
It's a Character Stream that wraps around an underlying Byte Stream (`InputStream`), converting the raw bytes it reads into characters using a specified encoding — bridging the byte-oriented and character-oriented worlds together.

**Q33. Can you mix Byte Streams and Character Streams directly on the same underlying resource?**
Not safely, generally — each maintains its own internal buffering/position tracking, and mixing reads/writes between a byte-level and character-level stream on the same underlying resource can lead to inconsistent, corrupted results. Pick one approach (byte or character) consistently for a given resource.

**Q34. Why does Character Stream reading potentially involve more overhead per call than Byte Stream reading?**
Because it needs to interpret raw bytes according to a character encoding scheme, which can involve variable numbers of bytes per character (like UTF-8, where characters can take 1 to 4 bytes) — this decoding work is extra processing beyond simply passing raw bytes through, as Byte Streams do.

---

# SECTION 4: Buffered Streams (Q35–Q44)

## Basic

**Q35. What problem do Buffered Streams solve?**
Reading or writing data one small piece at a time directly to/from a file (or network) is slow, because each individual operation can involve a real, relatively expensive system call. Buffered Streams reduce this by reading/writing larger chunks at once internally, minimizing the number of actual, expensive I/O operations.

**Q36. How do you wrap a stream to add buffering?**
```java
BufferedReader reader = new BufferedReader(new FileReader("data.txt"));
BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"));

BufferedInputStream bis = new BufferedInputStream(new FileInputStream("data.bin"));
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("output.bin"));
```

**Q37. What convenient extra method does `BufferedReader` provide that plain `FileReader` doesn't?**
`readLine()` — reads a whole line of text at once, returning it as a `String` (or `null` at end of file), instead of reading one character at a time.
```java
try (BufferedReader reader = new BufferedReader(new FileReader("data.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}
```

## Intermediate

**Q38. How does a Buffered Stream actually reduce the number of real I/O operations, internally?**
It maintains an internal in-memory buffer (byte array). When you request a small read, it actually reads a LARGE chunk from the underlying resource into that buffer in one operation, then serves your small requests directly from that in-memory buffer until it's exhausted — only going back to the actual underlying resource once the buffer needs refilling.

**Q39. Do you need to close both the Buffered wrapper stream AND the underlying stream it wraps?**
No — closing the outer Buffered stream automatically closes the underlying stream it wraps too, since it holds a reference to it internally. You only need to close the outermost wrapper.

**Q40. Why does `BufferedWriter` sometimes need an explicit `flush()` call?**
Because data written to a BufferedWriter may sit in its internal buffer, not yet actually written out to the underlying file, until the buffer fills up or is explicitly flushed (or closed) — `flush()` forces any buffered data to be written out immediately, useful when you need to guarantee data has actually reached the file at a specific point, without closing the stream yet.

## Advanced

**Q41. What is the performance difference, roughly, between unbuffered and buffered reads for a large file?**
Buffering can provide a dramatic performance improvement — potentially many times faster — for scenarios doing lots of small reads/writes, since it collapses many small, expensive underlying I/O operations into far fewer, larger ones.

**Q42. Is buffering ever unnecessary or wasteful to add?**
Yes — if you're already reading/writing large chunks at once (e.g., a single big `read(buffer)` call with a large buffer array), adding an additional Buffered wrapper provides little to no benefit, since you're not making many small calls that buffering would help consolidate.

**Q43. What is `PrintWriter`, and how does it relate to Buffered Streams?**
`PrintWriter` provides convenient formatted output methods (like `println()`, `printf()`) — it's often combined WITH a `BufferedWriter` underneath for performance, since `PrintWriter` itself doesn't add buffering, just convenience formatting methods.
```java
PrintWriter writer = new PrintWriter(new BufferedWriter(new FileWriter("output.txt")));
writer.println("Hello");
```

**Q44. Can you chain multiple stream wrappers together, like Buffered + InputStreamReader + FileInputStream?**
Yes — this is a very common pattern in Java I/O, called "stream chaining" or "decorating": each wrapper adds one specific capability (encoding conversion, buffering, convenience methods) around an underlying stream, and you can layer several together to combine their benefits.
```java
BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("data.txt"), StandardCharsets.UTF_8));
```

---

# SECTION 5: Serialization (Q45–Q56)

## Basic

**Q45. What is Serialization?**
The process of converting a Java object's state into a stream of bytes, so it can be saved to a file, sent over a network, or stored somewhere — and later reconstructed back into an equivalent object (Deserialization).

**Q46. What interface must a class implement to be serializable?**
`Serializable` — a marker interface (it has no methods at all) that simply signals to Java, "this class is allowed to be serialized."
```java
class Person implements Serializable {
    String name;
    int age;
}
```

**Q47. Why is `Serializable` called a "marker interface"?**
Because it has no methods to implement at all — its only purpose is to "mark" or tag a class as serializable, which Java's serialization mechanism checks for at runtime before allowing the object to be serialized.

**Q48. How do you serialize an object to a file?**
```java
try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.ser"))) {
    Person person = new Person("Ananth", 26);
    oos.writeObject(person);
}
```

## Intermediate

**Q49. What is `serialVersionUID`, and why is it important?**
A version number, ideally declared explicitly, that helps Java verify that a serialized object's class definition matches the class currently loaded — used to check compatibility when deserializing.
```java
class Person implements Serializable {
    private static final long serialVersionUID = 1L;
}
```

**Q50. What happens if you don't declare `serialVersionUID` explicitly?**
Java auto-generates one based on the class's structure (fields, methods, etc.) at compile time — but this can change silently if you modify the class even slightly (like adding a new field), causing an `InvalidClassException` when trying to deserialize old data with the new class version. Declaring it explicitly avoids this fragility.

**Q51. What does the `transient` keyword do?**
Marks a field to be SKIPPED during serialization — its value is not saved, and after deserialization, that field gets its default value (like `null` for objects, `0` for numbers) instead of the original value.
```java
class User implements Serializable {
    String username;
    transient String password; // never gets saved during serialization
}
```

**Q52. Why would you mark a field as `transient`?**
Commonly for security (avoiding saving sensitive data like passwords), for fields that hold non-serializable objects (like a database connection or a Thread, which cannot be meaningfully serialized), or for fields that can simply be recalculated/reinitialized after deserialization rather than needing to be saved.

## Advanced

**Q53. What happens if a class implementing `Serializable` has a field whose type does NOT implement `Serializable`?**
If that field is not marked `transient`, attempting to serialize the object throws a `NotSerializableException` at runtime — every non-transient field's type must also be serializable for the whole object to be serialized successfully.

**Q54. Are `static` fields included in serialization?**
No — `static` fields belong to the CLASS itself, not to any individual object instance, so serialization (which saves the state of a specific object) does not include them at all.

**Q55. How does inheritance affect serialization — what happens if a parent class does NOT implement `Serializable`, but the child class does?**
The parent class's fields are NOT automatically serialized along with the child's. When deserializing, Java actually calls the parent class's no-argument constructor to initialize the parent's part of the object, meaning any non-default parent field values are lost unless the parent class also implements Serializable (or you use custom serialization logic).

**Q56. What are `writeObject()`/`readObject()` custom methods, and when would you define them yourself?**
You can define private `writeObject(ObjectOutputStream out)` and `readObject(ObjectInputStream in)` methods inside a Serializable class to customize exactly how serialization/deserialization happens — useful for things like encrypting sensitive fields before saving, or handling fields that need special reconstruction logic beyond simple field-by-field copying.

---

# SECTION 6: Deserialization (Q57–Q66)

## Basic

**Q57. What is Deserialization?**
The reverse process of Serialization — reading a stream of bytes (previously created by serializing an object) and reconstructing an equivalent Java object from it.

**Q58. How do you deserialize an object from a file?**
```java
try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.ser"))) {
    Person person = (Person) ois.readObject(); // note the required cast, and checked exception
} catch (IOException | ClassNotFoundException e) {
    e.printStackTrace();
}
```

**Q59. Why does `readObject()` throw `ClassNotFoundException`, in addition to `IOException`?**
Because deserialization needs to load the actual `.class` file for the object's type to reconstruct it — if that class isn't available on the classpath at the time of deserialization, Java can't reconstruct the object, and throws this checked exception.

## Intermediate

**Q60. Is a class's constructor called during deserialization?**
No — this surprises many learners. Deserialization does NOT call the class's normal constructor; instead, the object's fields are populated directly from the serialized byte data, bypassing constructor logic entirely (except, as mentioned earlier, for a non-serializable parent class's no-argument constructor, which IS called).

**Q61. What value does a `transient` field have after deserialization?**
Its default value for that type — `null` for objects, `0`/`0.0` for numeric primitives, `false` for boolean — since transient fields are never saved and therefore have nothing to restore from.

**Q62. What happens if you try to deserialize data using a class whose `serialVersionUID` doesn't match the one stored in the serialized data?**
Java throws `InvalidClassException` — this is exactly the safety check `serialVersionUID` exists to provide, preventing you from accidentally reconstructing an object using an incompatible version of the class.

## Advanced

**Q63. Why is deserializing data from an untrusted source considered a security risk?**
Deserialization can be exploited to construct malicious objects or trigger unexpected code execution (through crafted byte streams that abuse `readObject()` logic or class behavior) — this has been a real, well-known category of security vulnerabilities in Java applications, which is why deserializing data from untrusted or external sources is generally discouraged without careful validation/safeguards.

**Q64. What's a general best practice to reduce deserialization security risk?**
Avoid deserializing data from untrusted sources entirely if possible; if unavoidable, use safeguards like allow-lists of specific classes permitted to be deserialized, or prefer safer, more transparent data formats (like JSON) with well-vetted, security-conscious libraries instead of raw Java serialization for anything crossing a trust boundary.

**Q65. Can you deserialize an object into a completely different, unrelated class than the one it was serialized from?**
No — deserialization requires the exact same class (matching `serialVersionUID`, or a version Java considers compatible) to reconstruct the object correctly; you cannot deserialize `Person` data directly into an unrelated `Employee` class, even if their fields happen to look similar.

**Q66. How does Java's built-in serialization compare to using JSON (via a library like Jackson) for saving object data?**
Java serialization is Java-specific, binary, and tightly coupled to exact class versions (fragile across changes) — but requires very little code. JSON is human-readable, language-agnostic (any system can read it, not just Java), and generally considered safer and more flexible for real-world applications (especially anything involving external systems), though it typically requires a library and a bit more setup than Java's built-in mechanism.

---

# SECTION 7: NIO (New I/O) (Q67–Q76)

## Basic

**Q67. What is NIO, and why was it introduced?**
"New I/O" (`java.nio`), introduced in Java 4 and significantly expanded in Java 7 ("NIO.2"), providing a more powerful, flexible alternative to the original `java.io` package — offering better performance for certain use cases, non-blocking I/O support, and a richer, more informative file-handling API.

**Q68. What are the core building blocks of the original NIO (Java 4)?**
**Buffers** (containers for data being read/written) and **Channels** (represent an open connection to an I/O source, like a file or network socket, that can read into or write from a Buffer).

**Q69. How is NIO's Channel + Buffer model different from the traditional Stream model?**
Traditional streams read/write data sequentially, one piece at a time, in a mostly blocking manner. NIO's Channel/Buffer model reads/writes in chunks (via Buffers) and supports non-blocking operations, where you can check if data is ready without the calling thread being forced to wait.

## Intermediate

**Q70. What is a `ByteBuffer`, and how do you use it with a `FileChannel`?**
A `ByteBuffer` is a fixed-size container for byte data, used together with Channels for reading/writing.
```java
try (FileChannel channel = FileChannel.open(Paths.get("data.bin"))) {
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    int bytesRead = channel.read(buffer);
}
```

**Q71. What does "non-blocking I/O" mean, and why is it useful?**
A non-blocking operation returns immediately, even if the data isn't ready yet, rather than making the calling thread wait (block) until it is — this is especially useful for servers handling many simultaneous connections, since a single thread can check on many channels without being stuck waiting on any one of them.

**Q72. What is a `Selector` in NIO, and what problem does it solve?**
A `Selector` lets a SINGLE thread monitor MULTIPLE channels at once, and efficiently find out which ones are ready for reading/writing — this is the foundation for building highly scalable network servers that can handle many connections without needing one dedicated thread per connection.

## Advanced

**Q73. What is "NIO.2," introduced in Java 7?**
A major expansion of the NIO package, adding the modern `Path` and `Files` APIs (covered in the next sections), symbolic link support, file system watching (`WatchService`), and generally much better, more detailed error reporting compared to the older `File` class.

**Q74. Why can NIO Channels be more efficient for large file transfers?**
Certain Channel operations, like `FileChannel.transferTo()`, can leverage the operating system's own efficient file-copying mechanisms directly (sometimes called "zero-copy" transfer), avoiding the overhead of repeatedly copying data through the Java application's own memory space.

**Q75. What is memory-mapped file I/O in NIO, and what's its benefit?**
Using `FileChannel.map()`, you can map a file's content directly into the application's memory space, letting you read/write it as if it were a regular in-memory array — this can be significantly faster for large files with random-access patterns, since the operating system handles the underlying paging efficiently.

**Q76. When would you choose classic `java.io` streams over NIO, or vice versa?**
Classic `java.io` streams are simpler and perfectly adequate for typical, straightforward file reading/writing tasks. NIO is the better choice specifically for performance-critical scenarios (large files, high-throughput network servers), non-blocking requirements, or when you need NIO.2's richer file-metadata and error-handling capabilities (`Path`/`Files`, covered next).

---

# SECTION 8: Paths (Q77–Q86)

## Basic

**Q77. What is the `Path` interface, and how does it relate to the old `File` class?**
`Path` (from `java.nio.file`) is the modern replacement for representing a file/directory location, introduced as part of NIO.2 in Java 7 — it offers a richer, more capable API than the older `File` class, though both conceptually represent "a location on the filesystem."

**Q78. How do you create a `Path` object?**
```java
Path path = Paths.get("data.txt");
Path path2 = Paths.get("/home/user", "documents", "data.txt"); // builds a path from multiple parts
```
(In more recent Java versions, `Path.of(...)` is also available as an alternative to `Paths.get(...)`.)

## Intermediate

**Q79. How do you get various pieces of information from a `Path` object?**
```java
Path path = Paths.get("/home/user/documents/data.txt");
System.out.println(path.getFileName());  // data.txt
System.out.println(path.getParent());    // /home/user/documents
System.out.println(path.getRoot());      // /
System.out.println(path.isAbsolute());   // true
```

**Q80. How do you resolve a relative path against a base path?**
```java
Path base = Paths.get("/home/user");
Path resolved = base.resolve("documents/data.txt");
System.out.println(resolved); // /home/user/documents/data.txt
```

**Q81. How do you convert a relative `Path` into an absolute one?**
```java
Path relative = Paths.get("data.txt");
Path absolute = relative.toAbsolutePath();
```

**Q82. How do you normalize a path that contains redundant elements, like `..` or `.`?**
```java
Path messy = Paths.get("/home/user/../user/./documents");
Path clean = messy.normalize(); // /home/user/documents
```

## Advanced

**Q83. How do you convert between `Path` and the older `File` class?**
```java
Path path = file.toPath();      // File -> Path
File file2 = path.toFile();     // Path -> File
```
This is useful for gradually migrating older code, or interoperating with APIs that still expect the older `File` type.

**Q84. What is the practical benefit of `Path` being an interface, rather than a concrete class like `File`?**
This allows different underlying filesystem implementations (not just the local disk — for example, paths within a ZIP file, treated as its own filesystem) to all be represented uniformly through the same `Path` interface, something the older, disk-specific `File` class doesn't support.

**Q85. How do you compare two `Path` objects to see if they point to the same actual location, accounting for differences like `..` or symbolic links?**
```java
Path p1 = Paths.get("/home/user/data.txt");
Path p2 = Paths.get("/home/user/../user/data.txt");
boolean same = Files.isSameFile(p1, p2); // resolves and compares the ACTUAL target, not just the text of the path
```

**Q86. Why might you use `Path.relativize()`?**
To compute a relative path FROM one location TO another — useful for generating relative links or references between files, given their absolute locations.
```java
Path base = Paths.get("/home/user/documents");
Path target = Paths.get("/home/user/documents/reports/2026.txt");
Path relative = base.relativize(target); // reports/2026.txt
```

---

# SECTION 9: The `Files` API (Q87–Q98)

## Basic

**Q87. What is the `Files` class, and how does it relate to `Path`?**
`Files` (from `java.nio.file`) is a utility class full of static methods that operate ON `Path` objects — creating, reading, writing, copying, moving, deleting files and directories, and much more, all in a much richer and more informative way than the older `File` class provided.

**Q88. How do you read an entire text file's content into a String using the `Files` API?**
```java
String content = Files.readString(Paths.get("data.txt")); // Java 11+
```

**Q89. How do you read all lines of a text file into a `List<String>`?**
```java
List<String> lines = Files.readAllLines(Paths.get("data.txt"));
```

**Q90. How do you write a String's content to a file using `Files`?**
```java
Files.writeString(Paths.get("output.txt"), "Hello, World!"); // Java 11+
```

## Intermediate

**Q91. How do you copy, move, and delete files using the `Files` API?**
```java
Files.copy(Paths.get("source.txt"), Paths.get("destination.txt"));
Files.move(Paths.get("old.txt"), Paths.get("new.txt"));
Files.delete(Paths.get("unwanted.txt"));
```

**Q92. How do you check if a file exists, and get its size, using `Files`?**
```java
boolean exists = Files.exists(path);
long size = Files.size(path); // in bytes
```

**Q93. How do you create a directory (including any missing parent directories) using `Files`?**
```java
Files.createDirectories(Paths.get("a/b/c")); // creates all missing folders along the way
```

**Q94. What is `Files.lines()`, and why is it especially useful for large files?**
Returns a `Stream<String>`, lazily reading the file line by line as the stream is consumed — meaning it doesn't need to load the entire file into memory at once (unlike `readAllLines()`), making it much better suited for very large files.
```java
try (Stream<String> lines = Files.lines(Paths.get("bigfile.txt"))) {
    long count = lines.filter(line -> line.contains("error")).count();
}
```

## Advanced

**Q95. What advantage does `Files`' error reporting have over the older `File` class's boolean-returning methods?**
Most `Files` methods throw a specific, descriptive `IOException` (or a subclass, like `NoSuchFileException` or `FileAlreadyExistsException`) when something goes wrong, telling you exactly what failed and why — the older `File` class's methods often just silently return `false` on failure, giving you no information about the actual cause.

**Q96. How do you walk an entire directory tree (all files and subdirectories) using the `Files` API?**
```java
try (Stream<Path> paths = Files.walk(Paths.get("myFolder"))) {
    paths.filter(Files::isRegularFile)
         .forEach(System.out::println);
}
```

**Q97. What is `Files.newBufferedReader()`/`newBufferedWriter()`, and why might you use them instead of manually wrapping streams?**
Convenience factory methods that create a properly-configured `BufferedReader`/`BufferedWriter` directly from a `Path`, with a specified character encoding — cleaner and less error-prone than manually chaining `FileReader`/`InputStreamReader`/`BufferedReader` together yourself.
```java
try (BufferedReader reader = Files.newBufferedReader(Paths.get("data.txt"), StandardCharsets.UTF_8)) {
    // read
}
```

**Q98. What are `StandardCopyOption` flags, and give an example of using one with `Files.copy()`.**
Options that control the behavior of certain `Files` operations — for example, `REPLACE_EXISTING` allows overwriting a destination file that already exists (without it, `Files.copy()` throws an exception if the destination already exists).
```java
Files.copy(source, destination, StandardCopyOption.REPLACE_EXISTING);
```

---
---

# PART 2: ANNOTATIONS

---

# SECTION 10: Built-in Annotations (Q99–Q112)

## Basic

**Q99. What is an annotation in Java?**
A special kind of metadata you can attach to code (classes, methods, fields, parameters, etc.) that doesn't directly change the program's logic itself, but provides extra information that tools, the compiler, or other code (often via Reflection) can read and act on.

**Q100. What does `@Override` do?**
Tells the compiler that this method is intended to override a method from a parent class or interface — if it doesn't actually match any method being overridden (e.g., due to a typo in the method name or wrong parameters), the compiler raises an error, catching the mistake early.
```java
@Override
public String toString() { return "Person"; }
```

**Q101. What does `@Deprecated` do?**
Marks a method, class, or field as outdated and discouraged from use — the compiler generates a warning wherever it's used elsewhere, signaling to other developers that they should look for an alternative.

**Q102. What does `@SuppressWarnings` do?**
Tells the compiler to suppress specific categories of warnings for the annotated code, most commonly `@SuppressWarnings("unchecked")` for unchecked generic operations you've deliberately reviewed and consider safe.
```java
@SuppressWarnings("unchecked")
List<String> list = (List<String>) someRawList;
```

## Intermediate

**Q103. Is using `@Override` mandatory when actually overriding a method?**
No, technically the code will still work correctly without it — but it's strongly recommended as a best practice, since it lets the compiler catch a whole category of subtle mistakes (like accidentally overloading instead of overriding, due to a typo) that would otherwise silently compile without any warning.

**Q104. What does `@FunctionalInterface` do?**
Marks an interface as intended to have exactly one abstract method (a functional interface, usable with lambda expressions) — if a second abstract method is accidentally added, the compiler raises an error immediately, rather than letting the mistake surface later wherever a lambda is used with it.

**Q105. What does `@SafeVarargs` do, and where can it be used?**
Suppresses warnings related to potentially unsafe operations involving generic varargs parameters — it can only be applied to `static` or `final` methods, or constructors, because it's only safe to make this promise on methods that can't be overridden (where behavior might change unexpectedly).

**Q106. If a method is marked `@Deprecated`, does that mean it will stop working?**
Not necessarily — `@Deprecated` is purely advisory; the method still works exactly as before. It's a signal that the method may be removed in a FUTURE version, or that a better alternative already exists, encouraging developers to migrate away from it over time.

## Advanced

**Q107. Can `@Deprecated` include additional information about why something is deprecated, and what to use instead?**
Yes, using the `since` and `forRemoval` elements (Java 9+):
```java
@Deprecated(since = "9", forRemoval = true)
public void oldMethod() { }
```
This tells tooling/developers exactly when it was deprecated, and whether it's actually planned to be removed (versus just discouraged but staying around).

**Q108. What does `@SuppressWarnings("all")` do, and why is it generally discouraged?**
It suppresses ALL categories of compiler warnings for the annotated code — generally discouraged because it can hide genuinely useful warnings you'd want to see, not just the specific one you intended to suppress; it's better practice to suppress only the specific warning category you've actually reviewed and deemed safe.

**Q109. Does `@Override` work when implementing a method from an INTERFACE, not just a parent class?**
Yes — since Java 6, `@Override` works correctly for methods implementing an interface method too, not just overriding a superclass method (this wasn't always the case in earlier Java versions).

**Q110. What happens at compile time if you use `@Override` on a method that doesn't actually override or implement anything?**
The compiler raises an error, something like "method does not override a method from its superclass" — this is exactly the safety net `@Override` provides, catching accidental typos or signature mismatches immediately.

**Q111. Are built-in annotations like `@Override` and `@Deprecated` present at runtime, or only used by the compiler?**
It depends on the specific annotation's retention policy. `@Override` is `SOURCE`-retention (used purely by the compiler, discarded immediately after compiling, never visible via Reflection). `@Deprecated` IS retained at runtime (`RUNTIME` retention), so tools and Reflection-based code CAN detect it if needed.

**Q112. Why does Java include `@FunctionalInterface` as optional, rather than mandatory for any interface with one abstract method?**
For backward compatibility and flexibility — plenty of existing interfaces with exactly one abstract method existed before Java 8 introduced lambdas, without ever being explicitly marked. Making the annotation mandatory would have broken or required updating a huge amount of pre-existing code; it remains a helpful, optional safety check instead.

---

# SECTION 11: Custom Annotations (Q113–Q124)

## Basic

**Q113. How do you define a custom annotation?**
```java
@interface MyAnnotation {
}
```
The `@interface` keyword (not just `interface`) is what defines an annotation type.

**Q114. How do you add elements (like parameters) to a custom annotation?**
```java
@interface Command {
    String name();
    int priority() default 1; // "default" provides a value if not specified when used
}
```

**Q115. How do you use a custom annotation with elements?**
```java
@Command(name = "add", priority = 2)
public void addCommand() { }
```

## Intermediate

**Q116. What does the `default` keyword do inside an annotation element declaration?**
Provides a default value to use if the annotation is applied WITHOUT explicitly specifying that particular element — making that element optional to specify at each usage site.
```java
@interface Command {
    String name();
    int priority() default 1;
}
// Both of these are valid:
@Command(name = "add")             // priority defaults to 1
@Command(name = "add", priority = 5) // priority explicitly set
```

**Q117. What types are allowed for annotation elements?**
Primitives (int, boolean, etc.), `String`, `Class`, enums, other annotations, and arrays of any of these — but NOT arbitrary objects or generic types.

**Q118. How do you define an annotation element that accepts multiple values (an array)?**
```java
@interface Tags {
    String[] value();
}
// Usage:
@Tags({"urgent", "billing", "customer-facing"})
```

**Q119. What is special about an annotation element named `value()`?**
If an annotation has exactly one element, and it's named `value`, you can apply the annotation WITHOUT explicitly naming that element:
```java
@interface Author {
    String value();
}
@Author("Ananth") // shorthand — equivalent to @Author(value = "Ananth")
```

## Advanced

**Q120. Can an annotation element have no default and be required at every usage site?**
Yes — simply don't specify a `default` for that element; then every use of the annotation MUST explicitly provide a value for it, or the code won't compile.

**Q121. Can you use a custom annotation on itself, or reference the same annotation type as an element's type?**
An annotation can reference OTHER annotation types as an element's type, but an annotation type cannot use itself directly as its own element's type (this would create a circular, unresolvable definition).

**Q122. Why is a custom annotation, by itself (with no meta-annotations), essentially useless for anything beyond documentation?**
Without a `@Retention` meta-annotation specifying `RUNTIME`, the annotation is discarded before or shortly after compilation, meaning no Reflection-based code can ever detect or act on it while the program runs — it becomes purely a compile-time or source-level note, with no functional effect (covered in detail in the next section, Meta Annotations).

**Q123. How would you design a custom `@LogExecutionTime` annotation intended to be read by a framework, to automatically measure how long an annotated method takes to run?**
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface LogExecutionTime {
}
```
A framework (or your own Reflection-based code) would then scan for methods marked with this annotation, and wrap their execution with timing logic — this exact pattern (define annotation, scan for it via Reflection, act on it) is the foundation of how many framework features work.

**Q124. Can annotation elements have complex default values, like referencing another class?**
Yes, if the element's type is `Class`:
```java
@interface Validator {
    Class<?> validatorClass() default DefaultValidator.class;
}
```

---

# SECTION 12: Meta Annotations (Q125–Q136)

## Basic

**Q125. What is a Meta Annotation?**
An annotation that is applied TO another annotation, to describe how that annotation itself should behave — for example, controlling where it can be used, or how long it should be retained.

**Q126. What does `@Retention` control?**
How long the annotation is kept around — whether it's discarded after compiling, kept in the compiled `.class` file but discarded before running, or kept all the way through to runtime (where Reflection can see it).

**Q127. What are the three `RetentionPolicy` values?**
`SOURCE` (discarded immediately after compiling — never in the compiled bytecode at all), `CLASS` (kept in the compiled `.class` file, but discarded before the program actually runs — this is the DEFAULT if `@Retention` isn't specified), `RUNTIME` (kept all the way through — visible to Reflection while the program runs).

**Q128. What does `@Target` control?**
Restricts WHERE an annotation is allowed to be applied — e.g., only on classes, only on methods, only on fields, and so on. Using it in a disallowed location causes a compile error.

## Intermediate

**Q129. Name the common `ElementType` values used with `@Target`.**
`TYPE` (classes/interfaces), `FIELD`, `METHOD`, `PARAMETER`, `CONSTRUCTOR`, `LOCAL_VARIABLE`, `ANNOTATION_TYPE` (an annotation that can itself be applied to other annotations), `PACKAGE`.

**Q130. Can `@Target` allow an annotation to be used in multiple different kinds of locations?**
Yes — you can supply an array of `ElementType` values:
```java
@Target({ElementType.METHOD, ElementType.FIELD})
@interface MyAnnotation { }
```

**Q131. What does `@Documented` do?**
A simple marker meta-annotation indicating that this annotation's presence should be included in generated JavaDoc documentation for the elements it's applied to — without it, the annotation wouldn't show up in generated documentation, even though it's still functionally present in the code.

**Q132. What does `@Inherited` do?**
Indicates that if a class is annotated with this annotation, its SUBCLASSES automatically "inherit" that same annotation too (as far as Reflection's `isAnnotationPresent()`/`getAnnotation()` checks are concerned), without needing to redeclare it explicitly on each subclass. Important limitation: `@Inherited` only works for CLASS-level annotations, not for annotations on methods or fields.

## Advanced

**Q133. Why is `@Retention(RetentionPolicy.RUNTIME)` specifically required for ANY custom annotation meant to be read via Reflection at runtime (like Spring's `@Component`)?**
Without it, the default retention (`CLASS`) discards the annotation before the program actually runs — Reflection can only see what's genuinely available while the program executes, so `RUNTIME` retention is a hard requirement for this use case, not just a nice-to-have.

**Q134. Give an example of `@Repeatable`, and explain what container annotation it requires.**
```java
@Repeatable(Schedules.class)
@interface Schedule {
    String day();
}
@interface Schedules {
    Schedule[] value(); // the required "container" annotation
}

@Schedule(day = "Monday")
@Schedule(day = "Wednesday") // allowed thanks to @Repeatable
public void doTask() { }
```

**Q135. What happens if you apply an annotation to an element type NOT listed in its `@Target`?**
The code fails to compile — this is a compile-time safety check, catching a category of misuse (using an annotation somewhere it was never intended for) before the program is ever run.

**Q136. Can a meta-annotation be applied to itself, or to other meta-annotations?**
Yes — `@Retention`, `@Target`, `@Documented`, and `@Inherited` are themselves annotations, and they carry their own meta-annotations (including `@Target(ElementType.ANNOTATION_TYPE)`, restricting them to only be usable on other annotation declarations) — this is exactly how Java enforces that these meta-annotations can only be applied to annotation types, not to regular classes or methods.

---
---

# PART 3: ENUMS

---

# SECTION 13: Enum Basics (Q137–Q148)

## Basic

**Q137. What is an enum in Java?**
A special data type that represents a fixed, predefined set of constant values — used when a variable should only ever hold one of a small, known set of options (like days of the week, or order statuses).

**Q138. How do you declare a simple enum?**
```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

**Q139. How do you use an enum value?**
```java
Day today = Day.MONDAY;
```

**Q140. Why are enums considered safer than using plain `int` constants (like `int MONDAY = 0;`) to represent a fixed set of options?**
With plain integer constants, a variable typed as `int` could accidentally be assigned ANY integer value, including ones that don't represent a valid day at all — the compiler wouldn't catch this mistake. With an enum, the compiler enforces that a `Day` variable can only ever hold one of the actual defined enum constants, catching invalid values at compile time.

## Intermediate

**Q141. Is an enum actually a class in Java, under the hood?**
Yes — every enum implicitly extends `java.lang.Enum`, and each enum constant is actually a `public static final` instance of that enum type. Enums genuinely are full classes, just with a special, restricted declaration syntax and some free built-in behavior.

**Q142. Can an enum implement an interface?**
Yes — this is covered in depth in Section 16, but briefly: `enum Day implements Schedulable { ... }` works exactly as you'd expect for a regular class.

**Q143. Can an enum extend another class?**
No — since every enum already implicitly extends `java.lang.Enum`, and Java doesn't support multiple inheritance of classes, an enum cannot extend any additional class.

**Q144. Can you use an enum in a `switch` statement?**
Yes — this is a very common, idiomatic use case:
```java
switch (today) {
    case MONDAY -> System.out.println("Start of the week");
    case FRIDAY -> System.out.println("Almost the weekend!");
    default -> System.out.println("Just another day");
}
```

## Advanced

**Q145. Can enum constants be compared using `==`?**
Yes, and it's actually the recommended way — since each enum constant is a single, unique, guaranteed instance (there's only ever one `Day.MONDAY` object in the entire JVM), `==` comparison is both safe and slightly more efficient than calling `.equals()`.

**Q146. Are enum values effectively Singletons?**
Yes, in a sense — each enum constant is guaranteed to be created exactly once by the JVM, and that same single instance is used everywhere the constant is referenced, which is exactly the guarantee a well-implemented Singleton pattern aims to provide (and in fact, using a single-value enum is a well-known, robust way to implement the Singleton pattern in Java).

**Q147. Can enum constants be used as keys in a `HashMap`?**
Yes — but `EnumMap` (covered in the Collections document) is generally preferred for enum keys specifically, since it's more efficient (backed by a simple array indexed by ordinal, no hashing needed) than a general-purpose HashMap.

**Q148. Is it possible to create additional instances of an enum type at runtime, beyond its declared constants?**
No — this is intentionally impossible. Enum constructors are always implicitly `private` (explained further in Section 15), and there is no way to call `new Day()` from outside the enum itself — the only instances that will ever exist are the ones declared directly in the enum's definition.

---

# SECTION 14: Enum Methods (Q149–Q160)

## Basic

**Q149. How do you get the name of an enum constant as a String?**
```java
Day day = Day.MONDAY;
System.out.println(day.name()); // "MONDAY"
```

**Q150. What does `ordinal()` return?**
The zero-based position of the enum constant in its declaration order.
```java
System.out.println(Day.MONDAY.ordinal()); // 0
System.out.println(Day.TUESDAY.ordinal()); // 1
```

**Q151. How do you get all values of an enum as an array?**
```java
Day[] allDays = Day.values();
for (Day d : allDays) {
    System.out.println(d);
}
```

**Q152. How do you convert a String into its matching enum constant?**
```java
Day day = Day.valueOf("MONDAY");
```

## Intermediate

**Q153. What happens if `valueOf()` is called with a String that doesn't match any enum constant?**
It throws `IllegalArgumentException` — the String must match a declared constant's name EXACTLY (including case), or this exception is thrown.

**Q154. Is `ordinal()` generally recommended to rely on in application logic? Why or why not?**
Generally not recommended for anything beyond simple internal ordering purposes — the ordinal value depends entirely on the DECLARATION ORDER of the constants, so if someone later reorders, inserts, or removes an enum constant, every ordinal value can silently shift, breaking any logic that depended on a specific ordinal number.

**Q155. What does calling `toString()` on an enum constant return, by default?**
By default, it returns the same thing as `name()` — but unlike `name()` (which is `final` and cannot be overridden), `toString()` CAN be overridden if you want a different, more human-friendly display format for the enum.

**Q156. Can you override `toString()` in an enum?**
Yes:
```java
enum Day {
    MONDAY, TUESDAY;
    @Override
    public String toString() {
        return name().charAt(0) + name().substring(1).toLowerCase(); // "Monday" instead of "MONDAY"
    }
}
```

## Advanced

**Q157. What is `compareTo()` on an enum based on?**
It compares based on `ordinal()` — the declaration order — meaning enum constants are naturally comparable in the order they were declared, without you needing to implement `Comparable` yourself (enums implement it automatically).

**Q158. How would you safely convert a String to an enum constant, handling the case where it might not match, without relying on a try-catch around `valueOf()`?**
```java
Optional<Day> day = Arrays.stream(Day.values())
    .filter(d -> d.name().equalsIgnoreCase(input))
    .findFirst();
```
This avoids relying on exception handling for a routine, expected "might not match" scenario.

**Q159. Can two different enum constants ever have the same `ordinal()` value?**
No — ordinal values are strictly assigned based on unique declaration position (0, 1, 2, ...), so every constant within a single enum type has a distinct ordinal value.

**Q160. Why is `name()` marked `final` in the base `Enum` class, unlike `toString()`?**
`name()` is meant to reliably return the EXACT constant identifier as declared in the source code — guaranteed, unchangeable, and safe to use for things like matching against `valueOf()`. `toString()`, being overridable, is meant for a potentially different, customizable DISPLAY representation — the distinction ensures there's always one guaranteed-reliable way (`name()`) to get the true underlying constant name, regardless of how `toString()` might be customized.

---

# SECTION 15: Enum with Constructor (Q161–Q172)

## Basic

**Q161. Can an enum have a constructor?**
Yes — enums can have constructors, used to associate additional data with each constant.
```java
enum Day {
    MONDAY("Start of work week"),
    SATURDAY("Weekend!");

    private final String description;

    Day(String description) {
        this.description = description;
    }

    public String getDescription() {
        return description;
    }
}
```

**Q162. How do you access the extra data attached via an enum constructor?**
```java
System.out.println(Day.MONDAY.getDescription()); // "Start of work week"
```

## Intermediate

**Q163. Why must an enum's constructor always be `private` (implicitly or explicitly)?**
Because enum constants are only ever created ONCE, by the JVM itself, at the exact points where they're declared in the enum body — there's no legitimate reason for any outside code to construct a new instance, so Java enforces this by making the constructor implicitly private (and won't even allow you to declare it `public` or `protected`).

**Q164. Can you call `new Day(...)` from outside the enum?**
No — this is a compile error. Enum constructors can genuinely only be invoked internally, at the point where each constant is declared within the enum's own body.

**Q165. Can different enum constants pass different arguments to the same constructor?**
Yes, absolutely — this is exactly the point of an enum constructor, allowing each constant to carry its own specific associated data.
```java
enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    VENUS(4.869e+24, 6.0518e6);

    private final double mass;
    private final double radius;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
    }
}
```

## Advanced

**Q166. Can an enum have multiple constructors (overloaded)?**
Yes — just like a regular class, an enum can have multiple constructors with different parameter lists, and each constant chooses which one to use based on the arguments it provides.
```java
enum Status {
    ACTIVE,                          // uses the no-arg constructor
    INACTIVE("Temporarily disabled"); // uses the one-arg constructor

    private String reason;
    Status() { }
    Status(String reason) { this.reason = reason; }
}
```

**Q167. Can enum fields be mutable, or should they always be `final`?**
They CAN technically be mutable, but it's strongly recommended to make them `final` — since enum constants are effectively singletons shared across the entire application, a mutable field risks one part of the code unexpectedly changing shared state that other, completely unrelated parts of the code also rely on.

**Q168. When does an enum's constructor actually run, relative to when the class is first used?**
All enum constants (and their constructors) are initialized once, the first time the enum class is loaded/referenced by the JVM — similar to static initialization for a regular class — not lazily, one at a time, as each individual constant happens to be accessed.

**Q169. Can an enum constructor throw an exception?**
Technically yes, but if it does, and that exception isn't handled, it will prevent the entire enum class from being loaded/initialized at all — a serious failure, since ALL constants (not just the one whose constructor failed) become unusable, since the JVM tries to initialize the whole enum type together.

**Q170. Can an enum have instance methods that behave differently based on the constructor arguments passed to each constant?**
Yes — this is a very natural and common pattern, since each constant can store different data via its constructor, and shared instance methods can then use that per-constant data to behave differently.

**Q171. Can enum constants reference each other within their own constructor arguments?**
No — this isn't possible, since all enum constants are being initialized together, in declaration order, and a constant cannot reference another constant that hasn't been fully initialized yet (particularly ones declared later in the list).

**Q172. Why is using an enum with a constructor often considered better design than using separate, parallel arrays or a plain `Map` to associate data with a fixed set of constants?**
It keeps the constant AND its associated data bundled together in one clearly-defined, type-safe place — reducing the risk of two parallel data structures (like a `String[]` of names and a separate `double[]` of associated values) accidentally getting out of sync with each other as the code evolves.

---

# SECTION 16: Enum with Interface (Q173–Q182)

## Basic

**Q173. Can an enum implement an interface?**
Yes — exactly like a regular class.
```java
interface Describable {
    String describe();
}

enum Day implements Describable {
    MONDAY, SATURDAY;

    @Override
    public String describe() {
        return "Today is " + name();
    }
}
```

**Q174. Can an enum implement MORE THAN ONE interface?**
Yes — just like a regular class, an enum can implement multiple interfaces at once, separated by commas.

## Intermediate

**Q175. Why would you have an enum implement an interface, rather than just adding methods directly to the enum itself?**
Implementing an interface allows the enum to be used POLYMORPHICALLY wherever that interface type is expected — code written against the interface type doesn't need to know or care that the actual implementation happens to be an enum, making the enum interchangeable with other classes that also implement the same interface.

**Q176. Can each enum constant provide its OWN distinct implementation of an interface method (or an abstract method declared in the enum itself)?**
Yes — this is a genuinely powerful and commonly tested feature. Each constant can have its own body with a specific method override, giving each constant fundamentally different behavior for the same method call.
```java
enum Operation {
    ADD {
        public int apply(int a, int b) { return a + b; }
    },
    MULTIPLY {
        public int apply(int a, int b) { return a * b; }
    };

    public abstract int apply(int a, int b);
}

System.out.println(Operation.ADD.apply(3, 4));       // 7
System.out.println(Operation.MULTIPLY.apply(3, 4));  // 12
```

**Q177. In the example above, what is required for this per-constant method implementation pattern to work?**
The enum must declare the method as `abstract` in its main body — this forces EVERY constant to provide its own concrete implementation (each constant's `{ ... }` block acts like a tiny anonymous subclass of the enum, overriding that abstract method).

## Advanced

**Q178. What is happening "under the hood" when an enum constant has its own method body, like `ADD { ... }` above?**
Each such constant is essentially compiled as an anonymous subclass of the enum type, overriding the abstract method specifically for that one constant — this is why different constants can have genuinely different behavior for the same method signature.

**Q179. Can an enum constant with its own body access the enum's shared instance fields and constructor-provided data?**
Yes — since it's still fundamentally the same enum type underneath (just with a per-constant method override), it has full access to any fields, other methods, and constructor-supplied data defined in the enum's main body.

**Q180. Is it possible to have BOTH an interface implementation AND per-constant abstract method overrides on the same enum?**
Yes — an enum can implement an interface (providing a shared default behavior contract) while ALSO declaring its own additional abstract method that each constant implements individually, combining both patterns as needed for the specific design.

**Q181. Why might a Strategy design pattern be naturally implemented using an enum with per-constant method bodies, instead of separate classes implementing a common interface?**
When the full set of "strategies" is fixed and known in advance (unlike a genuinely open-ended, extensible Strategy pattern), an enum keeps all the strategies together in one place, gets free extras like `values()` for iterating all strategies and `valueOf()` for parsing one by name, and avoids the need for several separate small implementation classes — a common, idiomatic Java pattern for this specific "fixed set of behaviors" scenario.

**Q182. Give a realistic example of an enum with per-constant behavior, relevant to a fintech application.**
```java
enum TransactionType {
    DEPOSIT {
        public double apply(double balance, double amount) { return balance + amount; }
    },
    WITHDRAWAL {
        public double apply(double balance, double amount) { return balance - amount; }
    };

    public abstract double apply(double balance, double amount);
}
```
This lets calling code simply call `transactionType.apply(balance, amount)` without needing an if/else or switch statement to determine the correct calculation logic — the enum constant itself already knows how to apply itself.

---
---

# PART 4: NESTED CLASSES

---

# SECTION 17: Static Nested Class (Q183–Q192)

## Basic

**Q183. What is a static nested class?**
A class declared INSIDE another class, marked `static` — it behaves like a regular top-level class in most ways, except it's namespaced inside its enclosing class, and doesn't have automatic access to an instance of the outer class.

**Q184. How do you declare and use a static nested class?**
```java
class Outer {
    static class Nested {
        void display() { System.out.println("Inside static nested class"); }
    }
}

Outer.Nested nested = new Outer.Nested(); // created WITHOUT needing an Outer instance
nested.display();
```

**Q185. Do you need an instance of the outer class to create a static nested class instance?**
No — this is the key defining feature of a static nested class; it can be created independently, without any outer class instance existing at all.

## Intermediate

**Q186. Can a static nested class access the outer class's instance (non-static) fields and methods directly?**
No — since a static nested class isn't tied to any specific outer class instance, it cannot directly access the outer class's instance members. It CAN access the outer class's `static` members directly, though.

**Q187. Why would you use a static nested class instead of a completely separate, top-level class?**
When the nested class is logically closely related to (and used primarily by) the outer class, and doesn't need standalone visibility elsewhere — nesting it communicates this close relationship clearly, keeps related code organized together, and can also limit its visibility (e.g., making it `private`) if it's purely an internal implementation detail.

**Q188. Can a static nested class be `private`?**
Yes — and this is common when the nested class is purely an internal implementation detail not meant to be used or seen outside the outer class at all.

## Advanced

**Q189. Give a real, well-known example of a static nested class from the standard Java library.**
`Map.Entry` — it's declared as a nested interface inside `Map`, representing a single key-value pair; it's closely tied conceptually to `Map` but doesn't require a `Map` instance to be referenced by type.

**Q190. Can a static nested class itself have further nested classes inside it?**
Yes — nesting can go multiple levels deep, though doing so excessively is generally considered to hurt readability, and deeply nested structures are usually a sign the code could benefit from restructuring.

**Q191. Does creating a static nested class instance keep the outer class loaded/instantiated in memory unnecessarily?**
No — since a static nested class instance doesn't hold any implicit reference to an outer class instance, it doesn't keep an outer instance alive in memory the way a non-static Inner Class instance would (covered next); this is one reason static nested classes are often preferred by default when outer-instance access genuinely isn't needed.

**Q192. Can a static nested class implement an interface or extend another class?**
Yes — a static nested class behaves like a normal top-level class in essentially every other respect (implementing interfaces, extending classes, having its own constructors, fields, and methods); "static nested" only affects its relationship to the enclosing class, not its general class capabilities.

---

# SECTION 18: Inner Class (Non-static Nested Class) (Q193–Q204)

## Basic

**Q193. What is an Inner Class (also called a non-static nested class)?**
A class declared inside another class, WITHOUT the `static` keyword — unlike a static nested class, an Inner Class instance is always tied to a specific instance of its outer class, and can freely access that outer instance's fields and methods directly.

**Q194. How do you create an instance of an Inner Class?**
```java
class Outer {
    class Inner {
        void display() { System.out.println("Inside inner class"); }
    }
}

Outer outer = new Outer();
Outer.Inner inner = outer.new Inner(); // notice: created THROUGH an outer instance
```

**Q195. Why does creating an Inner Class instance require the unusual `outer.new Inner()` syntax?**
Because every Inner Class instance is inherently tied to a SPECIFIC outer instance — Java requires you to specify exactly which outer instance the new Inner instance should be associated with, hence going through that outer object to create it.

## Intermediate

**Q196. Can an Inner Class access the outer class's private fields directly?**
Yes — this is one of the main reasons to use an Inner Class: it has full, direct access to the enclosing outer instance's fields and methods, including private ones, without needing any special access workaround.

**Q197. How does an Inner Class refer to its enclosing outer instance explicitly, if needed (e.g., if a field name is shadowed)?**
```java
class Outer {
    int value = 10;
    class Inner {
        int value = 20;
        void display() {
            System.out.println(this.value);        // 20 — the Inner class's own field
            System.out.println(Outer.this.value);   // 10 — explicitly the Outer instance's field
        }
    }
}
```

**Q198. Can an Inner Class have `static` members?**
No, generally not (with the specific exception of `static final` constant fields) — since an Inner Class instance is always tied to a specific outer instance, it fundamentally doesn't fit well with the class-wide, instance-independent nature of static members.

## Advanced

**Q199. Why can a memory leak risk arise from Inner Class usage, particularly in older Android development or long-lived applications?**
Every Inner Class instance implicitly holds a hidden reference to its enclosing outer instance — if an Inner Class instance somehow outlives its logical purpose (e.g., stored somewhere long-lived, like a static field, or an event listener that's never unregistered), it can inadvertently keep the ENTIRE outer instance alive in memory too, even if nothing else needs it anymore, because of that hidden reference.

**Q200. How is this implicit outer-instance reference actually implemented, technically?**
The compiler secretly adds a hidden field (often named something like `this$0`) to the Inner Class, automatically set to reference the specific outer instance it was created through — you never write or see this field directly in your source code, but it's genuinely present in the compiled bytecode.

**Q201. When would you prefer a static nested class over an Inner Class, purely from a design perspective?**
Whenever the nested class does NOT genuinely need direct access to the outer instance's members — preferring static nested class by default avoids the implicit outer-reference memory-leak risk described above, and is generally considered a cleaner default choice unless outer-instance access is truly required.

**Q202. Can an Inner Class extend a class, or implement an interface, just like a top-level class?**
Yes — an Inner Class supports all the same general class capabilities (extending, implementing interfaces, its own methods and fields) as any other class; only its relationship to the outer instance (and the resulting access rules) differs.

**Q203. Give a realistic, common use case for an Inner Class.**
A GUI event listener that needs direct access to the containing component's private state (like a button's click handler needing to update a private field on its enclosing window/panel class) — the Inner Class's automatic access to outer private members makes this pattern convenient.

**Q204. Can you have multiple Inner Class instances, each tied to a different outer instance, coexisting at the same time?**
Yes — since each Inner Class instance is tied to whichever specific outer instance it was created through (`outerA.new Inner()` vs `outerB.new Inner()`), you can have many Inner Class instances simultaneously, each independently associated with its own distinct outer instance.

---

# SECTION 19: Local Inner Class (Q205–Q212)

## Basic

**Q205. What is a Local Inner Class?**
A class defined INSIDE a method body (or even inside a block, like an if-statement or loop) — its scope is limited entirely to that method/block, and it cannot be used or referenced from anywhere outside it.

**Q206. Write a simple example of a Local Inner Class.**
```java
void process() {
    class Helper {
        void assist() { System.out.println("Helping..."); }
    }
    Helper helper = new Helper();
    helper.assist();
}
```

## Intermediate

**Q207. Can a Local Inner Class access local variables from the enclosing method?**
Yes, but only variables that are effectively final (assigned once, never reassigned afterward) — for the same underlying reason lambdas have this restriction (covered in the Multithreading document): the local variable's stack frame might not exist anymore by the time the Local Inner Class's method actually runs, so Java needs a guaranteed-stable, unchanging value to safely capture.

**Q208. Can a Local Inner Class be marked `public`, `private`, or `static`?**
No — none of these access modifiers make sense for a class whose scope is already limited entirely to a single method/block; a Local Inner Class simply has no visibility modifier at all in its declaration.

**Q209. When would you actually use a Local Inner Class in real code?**
When you need a small, reusable-within-this-method helper class that has no purpose or meaning anywhere outside that specific method — genuinely rare in modern code, since lambda expressions and method references (Java 8+) now cover many of the use cases that once required a Local Inner Class.

## Advanced

**Q210. Why has the practical use of Local Inner Classes declined significantly since Java 8?**
Lambda expressions and method references now handle the majority of "I need a small piece of custom behavior, scoped to right here" use cases far more concisely — a Local Inner Class is now mostly reserved for cases genuinely needing more than a single abstract method's worth of behavior (multiple methods, or actual internal state), which a lambda cannot provide.

**Q211. Can a Local Inner Class access the enclosing class's (the outer, containing class's) instance fields too, not just the local method's variables?**
Yes — a Local Inner Class defined inside an instance method has access to both the enclosing method's effectively-final local variables AND the outer class's own instance fields, combining the capabilities of both an Inner Class and a lambda's variable capture.

**Q212. Is a Local Inner Class compiled into its own separate `.class` file?**
Yes — like all nested class types, it gets compiled into its own class file, typically named something like `Outer$1Helper.class`, distinguishing it from the enclosing class's own compiled file.

---

# SECTION 20: Anonymous Class (Q213–Q224)

## Basic

**Q213. What is an Anonymous Class?**
A class with no name at all, declared and instantiated in a single expression — typically used to provide a one-off implementation of an interface or an extension of a class, right at the exact point it's needed, without a separate named class declaration anywhere.

**Q214. Write a simple example of an Anonymous Class implementing an interface.**
```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

**Q215. Can an Anonymous Class extend a regular (concrete) class instead of implementing an interface?**
Yes:
```java
Thread t = new Thread() {
    @Override
    public void run() {
        System.out.println("Thread running");
    }
};
```

## Intermediate

**Q216. Why is this pattern called "anonymous"?**
Because the class genuinely has no declared name at all — you're creating an instance of a brand-new, unnamed class definition, all inline, in the exact same expression where you'd normally just write a class name after `new`.

**Q217. Can an Anonymous Class have a constructor?**
No — since it has no name, you cannot declare a named constructor for it (a constructor must share its class's name). It can, however, use an "instance initializer block" (`{ ... }` with no method signature) to run setup code when the instance is created.

**Q218. Can an Anonymous Class access variables from its enclosing scope?**
Yes, following the same effectively-final rule as lambdas and Local Inner Classes — it can freely read effectively-final local variables from the enclosing method, and can freely read AND modify the enclosing outer class's instance fields.

**Q219. How is an Anonymous Class different from a lambda expression, even though both are often used for the same simple use cases (like implementing `Runnable`)?**
An Anonymous Class creates an actual, separate compiled class at compile time, and `this` inside it refers to the anonymous class instance itself. A lambda uses a lighter-weight `invokedynamic` mechanism (no separate class file generated at compile time), and `this` inside a lambda refers to the ENCLOSING instance, not the lambda "itself."

## Advanced

**Q220. Can an Anonymous Class implement more than one interface at once?**
No — an Anonymous Class can implement AT MOST one interface, OR extend at most one class, but never both, and never multiple interfaces simultaneously (unlike a regular named class, which can implement several interfaces at once). If you genuinely need to combine multiple interfaces, a proper named class is required instead.

**Q221. Why can't you use `@Override` errors to catch mistakes as easily in older, pre-Java-8 Anonymous Class usage for something like event listeners, compared to modern lambda usage?**
This isn't really true — `@Override` works exactly the same way for Anonymous Classes as any other class, catching signature mismatches. The comparison usually made is more about verbosity and readability: Anonymous Classes require noticeably more boilerplate syntax than an equivalent lambda for genuinely simple, single-method implementations.

**Q222. In what scenario would you still prefer an Anonymous Class over a lambda, even in modern Java?**
When you need to implement an interface or abstract class with MORE THAN ONE method (lambdas only work for functional interfaces with exactly one abstract method), or when you specifically need the Anonymous Class's own distinct identity/state (fields) that persist across multiple method calls on that same instance — something a stateless lambda expression can't naturally provide.

**Q223. What naming pattern does the compiler use for the generated class file of an Anonymous Class?**
Something like `Outer$1.class`, `Outer$2.class` (numbered sequentially based on how many anonymous classes are declared within that outer class) — since it has no real name of its own, the compiler assigns it a synthetic, numbered one internally.

**Q224. Can an Anonymous Class be used to create a one-off subclass with an overridden method, purely for a single, specific use, without formally naming and declaring a whole new class elsewhere?**
Yes — this is exactly the classic use case, especially common before Java 8: quickly customizing/overriding just one or two methods of an existing class or interface for one specific spot in the code, without the overhead of creating and naming an entirely separate, formally declared class file just for that one-off use.

---
---

# PART 5: FUNCTIONAL PROGRAMMING (FUNCTIONAL INTERFACES)

---

# SECTION 21: Predicate, Function, Consumer, Supplier (Q225–Q244)

## Basic

**Q225. What is `Predicate<T>`, and what method does it define?**
Represents a condition/test — takes an input of type `T`, returns a `boolean`. Method: `boolean test(T t)`. Commonly used for filtering.
```java
Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4)); // true
```

**Q226. What is `Function<T, R>`, and what method does it define?**
Takes an input of type `T`, transforms it, and returns a result of type `R`. Method: `R apply(T t)`.
```java
Function<String, Integer> length = s -> s.length();
System.out.println(length.apply("hello")); // 5
```

**Q227. What is `Consumer<T>`, and what method does it define?**
Takes an input of type `T`, performs some action with it (a side effect, like printing), and returns nothing. Method: `void accept(T t)`.
```java
Consumer<String> printer = s -> System.out.println("Value: " + s);
printer.accept("hello");
```

**Q228. What is `Supplier<T>`, and what method does it define?**
Takes no input at all, and returns (produces/"supplies") a value of type `T` — like a factory or a lazy value provider. Method: `T get()`.
```java
Supplier<Double> randomValue = () -> Math.random();
System.out.println(randomValue.get());
```

## Intermediate

**Q229. How would you remember the difference between these four interfaces, in one line each?**
`Predicate` — asks a yes/no question. `Function` — transforms input into output. `Consumer` — does something with input, gives nothing back. `Supplier` — gives you something, needs no input.

**Q230. How do you combine multiple `Predicate` conditions together?**
Using the default methods `and()`, `or()`, and `negate()`:
```java
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> positiveAndEven = isPositive.and(isEven);
Predicate<Integer> notPositive = isPositive.negate();
```

**Q231. How do you chain multiple `Function` transformations together?**
Using `andThen()` (applies the first function, THEN the second) or `compose()` (applies the given function FIRST, then this one):
```java
Function<Integer, Integer> addOne = x -> x + 1;
Function<Integer, Integer> square = x -> x * x;

Function<Integer, Integer> combined = addOne.andThen(square); // (x+1)^2
System.out.println(combined.apply(2)); // (2+1)^2 = 9
```

**Q232. How do you chain multiple `Consumer` actions to run in sequence, on the same input?**
Using `andThen()`:
```java
Consumer<String> print = s -> System.out.println("Printing: " + s);
Consumer<String> log = s -> System.out.println("Logging: " + s);
Consumer<String> both = print.andThen(log);
both.accept("data"); // runs print, THEN log, both on the same input
```

**Q233. Why does `Supplier` matter for LAZY computation — give a concrete example of why this matters.**
A `Supplier` defers the actual computation until `.get()` is explicitly called — `Optional.orElseGet(Supplier)` only calls the supplier if genuinely needed (the Optional is empty), avoiding wasted work compared to `orElse(alreadyComputedValue)`, which always computes its argument eagerly, whether needed or not.

## Advanced

**Q234. What are the primitive-specialized versions of `Predicate`, `Function`, `Consumer`, and `Supplier`, and why do they exist?**
`IntPredicate`, `IntFunction`, `IntConsumer`, `IntSupplier` (and similarly for `long`/`double`) exist purely for performance — avoiding the overhead of autoboxing a primitive into its wrapper object (like `int` into `Integer`) every single time, which matters especially when processing large volumes of data, like in a stream pipeline.

**Q235. What is `IntPredicate`, and how is it different from `Predicate<Integer>`?**
`IntPredicate` works directly with the primitive `int`, with method `boolean test(int value)` — no boxing/unboxing overhead — whereas `Predicate<Integer>` requires each `int` to be boxed into an `Integer` object first before it can be tested.

**Q236. Can you write a custom `Predicate` for a complex, multi-field condition on an object?**
Yes:
```java
Predicate<Employee> isSeniorAndHighEarner = emp -> emp.getYearsOfService() > 5 && emp.getSalary() > 80000;
```

**Q237. How would you use `Function.identity()`, and what does it return?**
Returns a `Function` that simply returns its input unchanged — useful in situations (like certain `Collectors.toMap()` calls) where an API requires a `Function` argument, but you genuinely just want to use the element itself as-is.
```java
Function<String, String> identity = Function.identity(); // equivalent to: s -> s
```

**Q238. Can `Consumer.andThen()` be used with more than two Consumers chained together?**
Yes — since `andThen()` itself returns another `Consumer`, you can keep chaining as many as you like: `c1.andThen(c2).andThen(c3).andThen(c4)`.

**Q239. What happens if the FIRST Consumer in an `andThen()` chain throws an exception?**
The chain stops immediately — the SECOND (and any subsequent) Consumer in the chain never runs at all, since an unhandled exception propagates immediately, aborting the rest of the sequence.

**Q240. Give a real, practical example combining `Predicate` and `Consumer` together, like a simple validation-and-processing pipeline.**
```java
Predicate<Order> isValid = order -> order.getAmount() > 0;
Consumer<Order> process = order -> System.out.println("Processing: " + order);

if (isValid.test(order)) {
    process.accept(order);
}
```

**Q241. Why might `Supplier<T>` be used as a parameter type for a logging method, instead of just passing the message directly as a `String`?**
To defer the (potentially expensive) construction of the log message string until it's actually confirmed the message will be logged (e.g., only if the current log level is enabled) — avoiding wasted work building a detailed message string that would just be discarded if that log level is currently disabled.
```java
void debug(Supplier<String> messageSupplier) {
    if (debugEnabled) {
        System.out.println(messageSupplier.get()); // only built/called if actually needed
    }
}
```

**Q242. Can a `Predicate` be negated and then combined with `and()` or `or()` in one fluent chain?**
Yes:
```java
Predicate<Integer> result = isPositive.negate().and(isEven);
```

**Q243. What's the difference between `Predicate.and()` and simply writing `a.test(x) && b.test(x)` manually?**
Functionally equivalent for a single check, but `and()` produces a REUSABLE, COMPOSED `Predicate` object that can itself be passed around, stored, or further combined — whereas manually writing `&&` requires re-typing that same combined logic everywhere it's needed, rather than capturing it once as a named, reusable value.

**Q244. Why does `Function<T, R>` need TWO generic type parameters, while `Predicate<T>` and `Consumer<T>` only need one?**
`Function` transforms an input type INTO a potentially different output type, so it needs to track both. `Predicate` always returns a `boolean` (never a variable type), and `Consumer` never returns anything at all (`void`) — so neither needs a second type parameter to describe their (fixed or nonexistent) return type.

---

# SECTION 22: BiFunction, BinaryOperator, UnaryOperator (Q245–Q258)

## Basic

**Q245. What is `BiFunction<T, U, R>`?**
Takes TWO inputs (of types `T` and `U`), and returns a result of type `R`. Method: `R apply(T t, U u)`.
```java
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(3, 4)); // 7
```

**Q246. What is `UnaryOperator<T>`?**
A specialized version of `Function<T, T>` — where the input and output are the EXACT SAME type. Represents an operation that transforms a value but keeps it the same type.
```java
UnaryOperator<Integer> square = x -> x * x;
```

**Q247. What is `BinaryOperator<T>`?**
A specialized version of `BiFunction<T, T, T>` — both inputs AND the output are all the same type.
```java
BinaryOperator<Integer> sum = (a, b) -> a + b;
```

## Intermediate

**Q248. Why does `UnaryOperator<T>` exist separately from `Function<T, R>`, when it's really just a special case?**
It makes the intent clearer in code — a method parameter typed as `UnaryOperator<Integer>` immediately signals "this transforms an Integer into another Integer," which is more specific and self-documenting than the more general `Function<Integer, Integer>`, even though they're functionally interchangeable.

**Q249. Why does `BinaryOperator<T>` exist separately from `BiFunction<T, T, T>`?**
Same reasoning as `UnaryOperator` — clearer intent in code, and additionally, `BinaryOperator` provides some useful extra static helper methods (like `minBy()`/`maxBy()`, covered next) that are specifically relevant to this "combine two values of the same type into one" use case, which the more general `BiFunction` doesn't provide.

**Q250. Where are `BinaryOperator` and `UnaryOperator` most commonly used in the Streams API?**
`BinaryOperator` is the exact type expected by `Stream.reduce()`'s combining function — since reduction combines a running accumulated value with each new element, all of the same type. `UnaryOperator` is commonly used with `List.replaceAll()`, transforming each element of a list in place.

**Q251. Write an example using `reduce()` with a `BinaryOperator`.**
```java
List<Integer> numbers = List.of(1, 2, 3, 4);
int sum = numbers.stream().reduce(0, Integer::sum); // Integer::sum matches BinaryOperator<Integer>
```

**Q252. Write an example using `List.replaceAll()` with a `UnaryOperator`.**
```java
List<Integer> numbers = new ArrayList<>(List.of(1, 2, 3));
numbers.replaceAll(n -> n * 2); // in-place transformation using a UnaryOperator
System.out.println(numbers); // [2, 4, 6]
```

## Advanced

**Q253. What do the static helper methods `BinaryOperator.minBy()` and `BinaryOperator.maxBy()` do?**
They create a `BinaryOperator` that returns whichever of its two inputs is smaller (or larger), based on a given `Comparator` — a convenient, ready-made combining function especially useful with `reduce()`.
```java
BinaryOperator<Integer> maxOperator = BinaryOperator.maxBy(Comparator.naturalOrder());
int max = numbers.stream().reduce(Integer.MIN_VALUE, maxOperator);
```

**Q254. Is `UnaryOperator<T>` assignment-compatible with a variable of type `Function<T, T>`?**
Yes — since `UnaryOperator<T>` extends `Function<T, T>` directly, any `UnaryOperator` can be used wherever a matching `Function<T, T>` is expected.

**Q255. Are there primitive-specialized versions of `BiFunction`, `BinaryOperator`, and `UnaryOperator`?**
Yes — for example, `IntBinaryOperator`, `IntUnaryOperator`, `ToIntBiFunction`, and similar variants exist for `int`/`long`/`double`, for the same boxing-avoidance performance reasons as the other functional interfaces.

**Q256. Write a custom `BiFunction` that combines a String and an Integer into a formatted String.**
```java
BiFunction<String, Integer, String> formatter = (name, age) -> name + " is " + age + " years old";
System.out.println(formatter.apply("Ananth", 26));
```

**Q257. Can `BiFunction` be composed with `andThen()`, similar to `Function`?**
Yes — `BiFunction.andThen(Function)` lets you take the (single) result of the BiFunction and pass it through an additional transformation:
```java
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
BiFunction<Integer, Integer, String> addThenDescribe = add.andThen(result -> "Result is: " + result);
```

**Q258. In a fintech context, give a realistic example of using `BinaryOperator` with `reduce()` to combine transaction amounts.**
```java
List<Double> transactionAmounts = List.of(150.0, 200.0, 75.5);
BinaryOperator<Double> sumOperator = Double::sum;
double total = transactionAmounts.stream().reduce(0.0, sumOperator);
```
This pattern is a natural fit whenever you need to combine a sequence of same-typed values (transaction amounts, account balances, quantities) down into one aggregated result, using a clean, reusable combining function.

---

# Quick-reference summary table

| Part | Section | Topic | Question range |
|---|---|---|---|
| 1 | 1 | The File Class | Q1–Q12 |
| 1 | 2 | Byte Streams | Q13–Q24 |
| 1 | 3 | Character Streams | Q25–Q34 |
| 1 | 4 | Buffered Streams | Q35–Q44 |
| 1 | 5 | Serialization | Q45–Q56 |
| 1 | 6 | Deserialization | Q57–Q66 |
| 1 | 7 | NIO | Q67–Q76 |
| 1 | 8 | Paths | Q77–Q86 |
| 1 | 9 | The Files API | Q87–Q98 |
| 2 | 10 | Built-in Annotations | Q99–Q112 |
| 2 | 11 | Custom Annotations | Q113–Q124 |
| 2 | 12 | Meta Annotations | Q125–Q136 |
| 3 | 13 | Enum Basics | Q137–Q148 |
| 3 | 14 | Enum Methods | Q149–Q160 |
| 3 | 15 | Enum with Constructor | Q161–Q172 |
| 3 | 16 | Enum with Interface | Q173–Q182 |
| 4 | 17 | Static Nested Class | Q183–Q192 |
| 4 | 18 | Inner Class | Q193–Q204 |
| 4 | 19 | Local Inner Class | Q205–Q212 |
| 4 | 20 | Anonymous Class | Q213–Q224 |
| 5 | 21 | Predicate, Function, Consumer, Supplier | Q225–Q244 |
| 5 | 22 | BiFunction, BinaryOperator, UnaryOperator | Q245–Q258 |

**Total: 258 questions.**

---

## Suggested study strategy

1. **File I/O — start with Sections 2–4** (Byte/Character/Buffered Streams) to build the mental model of stream chaining, then move to **Sections 5–6** (Serialization/Deserialization), which are commonly asked together as one topic. **Sections 7–9** (NIO, Paths, Files) matter more for practical, modern code — know `Files` well, since it's what real-world code actually uses today.
2. **Annotations — Section 12 (Meta Annotations)** is the highest-value section here; `@Retention(RUNTIME)` connects directly to your Reflection API knowledge, and interviewers often test this connection specifically.
3. **Enums — Section 16 (Enum with Interface)**, especially the per-constant method body pattern (Q176), is a favorite "write this code" interview question — practice it until you can write it without hesitation.
4. **Nested Classes — Section 18 (Inner Class)**, particularly the memory-leak risk (Q199–Q200), is a strong, specific, memorable answer to have ready if asked "when would you avoid an Inner Class."
5. **Functional Programming — Sections 21–22** overlap with your existing Java 8 Q&A document, but the `andThen()`/`compose()` chaining questions and the `BinaryOperator`/`UnaryOperator` distinction from `BiFunction`/`Function` are worth being sharp on, since they're compact, precise questions interviewers like to use to check real understanding quickly.
