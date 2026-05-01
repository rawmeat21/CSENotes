# Java I/O Streams — Godmode Reference

  

---

  

## 1. Big Picture: Java I/O Architecture

  

### The Core Problem

  

Every program needs to move data: from disk, from network, from memory. Java's `java.io` package abstracts all of these into a single mental model: **a stream is a sequential flow of data**. You never think about where the data lives — you just read from or write to a stream.

  

### The Two Axes of Java I/O

  

Java I/O is organized along two orthogonal axes:

  

| Axis | Option A | Option B |

|---|---|---|

| **Unit** | Byte (8-bit raw) | Character (16-bit Unicode) |

| **Role** | Node stream (talks to source/sink) | Filter stream (wraps another stream) |

  

These two axes together define every class in `java.io`.

  

### Byte Streams vs Character Streams

  

| | Byte Streams | Character Streams |

|---|---|---|

| Base classes | `InputStream` / `OutputStream` | `Reader` / `Writer` |

| Unit | 1 byte (0–255) | 1 char (UTF-16 code unit) |

| Encoding | None — raw bytes | Handled via `Charset` |

| Use for | Images, audio, binary data, serialization | Text files, source code, config |

| Danger | None for binary | `FileReader`/`FileWriter` use platform default charset |

  

**Rule**: If you're moving binary data, use byte streams. If you're moving human-readable text, use character streams — but always specify the charset explicitly.

  

### Design Philosophy of `java.io`

  

`java.io` is built on the **Decorator pattern**. You compose streams:

  

```

FileInputStream (node)

→ BufferedInputStream (performance filter)

→ DataInputStream (type-aware filter)

```

  

This means every node stream is kept minimal (just talks to the source), and every filter stream adds exactly one concern. The cost: verbose constructor nesting. The benefit: unlimited composability.

  

---

  

## 2. Class Hierarchy

  

```

Object

│

┌────────────────┴─────────────────┐

InputStream OutputStream

│ │

┌───────┼──────────────┐ ┌──────────┼──────────────┐

FileIS FilterIS ObjectIS FileOS FilterOS ObjectOS

│ │

┌────┴────────┐ ┌───────┴──────────┐

BufferedIS DataIS BufferedOS DataOS

PrintStream (also FilterOS)

  

Object

│

┌────────────────┴─────────────────┐

Reader Writer

│ │

┌───────┼──────────┐ ┌────────┼──────────────┐

FileR InputStreamR BufferedR FileW OutputStreamW BufferedW

PrintWriter (also Writer)

```

  

`RandomAccessFile` is standalone — it implements both `DataInput` and `DataOutput` interfaces, but does NOT extend `InputStream` or `OutputStream`.

  

### Node Streams vs Filter Streams

  

**Node streams** connect directly to a data source or sink:

  

| Node Stream | Source/Sink |

|---|---|

| `FileInputStream` | File on disk |

| `FileOutputStream` | File on disk |

| `FileReader` | File on disk (chars) |

| `FileWriter` | File on disk (chars) |

| `ByteArrayInputStream` | byte[] in memory |

| `StringReader` | String in memory |

  

**Filter streams** wrap another stream and add behavior:

  

| Filter Stream | Behavior Added |

|---|---|

| `BufferedInputStream` | Internal byte[] buffer for bulk reads |

| `DataInputStream` | Read primitives (`readInt`, `readDouble`, etc.) |

| `ObjectInputStream` | Deserialize Java objects |

| `InputStreamReader` | Byte-to-char bridge with charset decoding |

| `BufferedReader` | Line-level buffering; `readLine()` |

| `PrintWriter` | Formatted text output, no checked exceptions |

  

**`CharacterInputStream` does not exist as a real class.** The conceptual equivalent is `InputStreamReader`, which is the adapter that converts a byte stream into a character stream.

  

---

  

## 3. Deep Dive: Each Class

  

---

  

### FileInputStream / FileOutputStream

  

**Purpose**: Read/write raw bytes directly from/to a file.

  

**Internal working**: Delegates to native OS file I/O. Every `read()` call is a syscall. No internal buffering.

  

**When to use**: Almost never alone — always wrap with `BufferedInputStream`. Direct use is only justified when you need one massive single read (e.g., `read(byte[], 0, fileSize)`).

  

**Performance**: Each single-byte `read()` = one JNI call = one syscall. On modern hardware: ~1–10µs per call. Reading 1MB byte-by-byte = ~1 second. Wrapped in `BufferedInputStream`: milliseconds.

  

**Common mistakes**:

- Forgetting to close (use try-with-resources)

- Not wrapping with a buffer

- Opening in write mode without `append=true` when you need to append

  

```java

try (FileInputStream fis = new FileInputStream("data.bin");

FileOutputStream fos = new FileOutputStream("copy.bin")) {

byte[] buf = new byte[8192];

int n;

while ((n = fis.read(buf)) != -1) {

fos.write(buf, 0, n);

}

}

```

  

---

  

### FileReader / FileWriter

  

**Purpose**: Read/write text files as characters.

  

**Internal working**: Thin wrappers around `InputStreamReader(new FileInputStream(...))` and `OutputStreamWriter(new FileOutputStream(...))`. The dangerous part: they use **the platform's default charset** (`Charset.defaultCharset()`), which is JVM-version and OS-dependent.

  

**When to use**: Only in throwaway scripts where encoding correctness doesn't matter. In production code, use `InputStreamReader`/`OutputStreamWriter` with an explicit charset instead.

  

**Performance**: Same syscall-per-character issue as `FileInputStream`. Wrap with `BufferedReader`/`BufferedWriter`.

  

**Common mistakes**:

- Using on a server that moves between Windows (Cp1252) and Linux (UTF-8)

- Not calling `flush()` before close (though `close()` calls `flush()` internally)

  

```java

try (BufferedReader br = new BufferedReader(

new InputStreamReader(new FileInputStream("file.txt"), StandardCharsets.UTF_8))) {

String line;

while ((line = br.readLine()) != null) {

System.out.println(line);

}

}

```

  

---

  

### BufferedInputStream / BufferedOutputStream

  

**Purpose**: Add an in-memory buffer to any byte stream, drastically reducing syscall frequency.

  

**Internal working**: Maintains an internal `byte[]` buffer (default 8192 bytes). `read()` fills the entire buffer in one syscall, then serves subsequent `read()` calls from memory until the buffer is exhausted. `write()` accumulates bytes until the buffer is full, then flushes once.

  

**When to use**: Wrap every `FileInputStream`/`FileOutputStream` unless you're doing single large bulk reads (`read(byte[], 0, size)`).

  

**Performance**: Reduces syscall count by up to `bufferSize` times. For small-read patterns, difference is 100–1000x. For already-large bulk reads, adds negligible overhead.

  

**Common mistakes**:

- Forgetting to call `flush()` on `BufferedOutputStream` before closing (close does it, but explicit flush matters mid-stream)

- Double-buffering: wrapping `BufferedInputStream` around another `BufferedInputStream`

- Using default buffer size (8192) for network streams where MTU alignment (1500 bytes) matters

  

```java

try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("in.bin"), 16384);

BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("out.bin"), 16384)) {

byte[] buf = new byte[4096];

int n;

while ((n = bis.read(buf)) != -1) {

bos.write(buf, 0, n);

}

}

```

  

---

  

### BufferedReader / BufferedWriter

  

**Purpose**: Buffer character-stream I/O. `BufferedReader` adds `readLine()`; `BufferedWriter` adds `newLine()`.

  

**Internal working**: Maintains a `char[]` buffer (default 8192 chars). `readLine()` scans the buffer for `\n`, `\r`, or `\r\n` — platform-independent line ending detection.

  

**When to use**: Reading text files line by line. Almost all text-file processing in Java goes through `BufferedReader.readLine()`.

  

**Performance**: The `readLine()` method avoids char-by-char scanning at the syscall level. In Java 8+, `BufferedReader.lines()` returns a lazy `Stream<String>`.

  

**Common mistakes**:

- `readLine()` returns `null` at EOF, not an empty string — failing to check causes `NullPointerException`

- `BufferedWriter.newLine()` writes the platform line separator (`System.lineSeparator()`); use this for files meant to be edited on the same OS. For cross-platform text protocols (HTTP, SMTP), write `\r\n` explicitly.

  

```java

try (BufferedWriter bw = new BufferedWriter(

new OutputStreamWriter(new FileOutputStream("out.txt"), StandardCharsets.UTF_8))) {

bw.write("Line one");

bw.newLine();

bw.write("Line two");

}

```

  

```java

try (var lines = Files.lines(Path.of("big.txt"), StandardCharsets.UTF_8)) {

lines.filter(l -> l.startsWith("ERROR"))

.forEach(System.out::println);

}

```

  

---

  

### DataInputStream / DataOutputStream

  

**Purpose**: Read/write Java primitive types in a portable binary format.

  

**Internal working**: `DataOutputStream.writeInt(int v)` writes exactly 4 bytes in **big-endian** order, always. `DataInputStream.readInt()` reconstructs the int from those 4 bytes. This is `java.io`'s portable binary format — identical on all platforms.

  

**When to use**: Binary file formats, custom network protocols where you control both ends, writing typed records to disk.

  

**When NOT to use**: If the other end is not Java, byte order may not match. For interop, use `ByteBuffer` with explicit endianness.

  

**Common mistakes**:

- Reading in a different order than writing (reads are sequential and stateful)

- Mixing with object serialization — `DataOutputStream` and `ObjectOutputStream` are separate protocols

  

```java

try (DataOutputStream dos = new DataOutputStream(

new BufferedOutputStream(new FileOutputStream("record.bin")))) {

dos.writeInt(42);

dos.writeDouble(3.14);

dos.writeUTF("hello");

}

  

try (DataInputStream dis = new DataInputStream(

new BufferedInputStream(new FileInputStream("record.bin")))) {

int i = dis.readInt();

double d = dis.readDouble();

String s = dis.readUTF();

}

```

  

---

  

### ObjectInputStream / ObjectOutputStream

  

**Purpose**: Serialize and deserialize entire Java object graphs.

  

**Internal working**: See Section 4 (Serialization). The stream writes a binary header, then recursively traverses the object graph. Each class descriptor, field name, field type, and field value is written. Circular references are handled via a reference table.

  

**When to use**: Caching object state, simple IPC between Java processes. Avoid for long-term storage or public APIs.

  

**Common mistakes**:

- Not calling `reset()` on `ObjectOutputStream` when reusing it (causes memory leak via retained object reference table)

- Deserializing from untrusted sources (critical security risk — see Section 4)

- Forgetting that `ObjectOutputStream` must be constructed before `ObjectInputStream` on the receiving end

  

```java

record Point(int x, int y) implements Serializable {}

  

try (ObjectOutputStream oos = new ObjectOutputStream(

new BufferedOutputStream(new FileOutputStream("obj.ser")))) {

oos.writeObject(new Point(10, 20));

}

  

try (ObjectInputStream ois = new ObjectInputStream(

new BufferedInputStream(new FileInputStream("obj.ser")))) {

Point p = (Point) ois.readObject();

}

```

  

---

  

### PrintWriter / PrintStream

  

**Purpose**: Formatted text output. No checked exceptions — errors are silently swallowed (check `checkError()`).

  

**Internal working**: `PrintWriter` wraps any `Writer`. `PrintStream` wraps any `OutputStream` and does byte-level output with character conversion. `System.out` is a `PrintStream`.

  

**Difference**:

  

| | `PrintStream` | `PrintWriter` |

|---|---|---|

| Wraps | `OutputStream` | `Writer` |

| Char encoding | Uses default charset internally | Uses underlying writer's charset |

| Auto-flush trigger | Every `println()` if autoFlush=true | Only on `println()`, `printf()`, `format()` if autoFlush=true |

| Use for | `System.out`/`System.err` | Writing formatted text to files |

  

**When to use**: `PrintWriter` for writing formatted reports or log output to files. `PrintStream` for console output (`System.out`).

  

**Common mistakes**:

- Ignoring silent error absorption — always call `checkError()` in production

- Creating a `PrintWriter` without autoFlush for network streams (output never arrives)

  

```java

try (PrintWriter pw = new PrintWriter(

new BufferedWriter(new FileWriter("report.txt")))) {

pw.printf("%-20s %5d%n", "Total records", 1024);

pw.printf("%-20s %5.2f%n", "Average score", 87.5);

if (pw.checkError()) throw new IOException("PrintWriter encountered an error");

}

```

  

---

  

### RandomAccessFile

  

See Section 5 for the full deep dive. Summary:

  

**Purpose**: Random seek-and-read/write to any position in a file.

  

**Internal working**: Wraps OS `lseek` + `read`/`write`. Maintains a file pointer internally.

  

**Modes**: `"r"` (read-only), `"rw"` (read-write), `"rws"` (sync writes to storage), `"rwd"` (sync data only).

  

```java

try (RandomAccessFile raf = new RandomAccessFile("data.bin", "rw")) {

raf.seek(100);

raf.writeInt(9999);

raf.seek(0);

int first = raf.readInt();

}

```

  

---

  

## 4. Serialization

  

### How `ObjectOutputStream` Works Internally

  

1. Writes a **stream magic** number (`0xACED`) and stream version.

2. For each object: checks if it has been serialized before. If yes, writes a **reference handle** (prevents infinite loops in circular graphs).

3. Writes the **class descriptor**: fully qualified class name, `serialVersionUID`, field names and types.

4. Recursively writes field values. For object-type fields, recurses.

5. If the class has `writeObject(ObjectOutputStream)`, calls it (custom serialization).

  

Deserialization is the exact inverse. **No constructor is called** during deserialization — the object is allocated raw via `Unsafe.allocateInstance()` and then fields are set directly.

  

### `Serializable` Interface

  

A marker interface — it carries no methods. Declaring it tells the JVM: "this class participates in the serialization protocol." If you serialize an object whose class does NOT implement `Serializable`, you get `NotSerializableException`.

  

### `transient` Keyword

  

Marks a field to be **excluded** from serialization. The field will be its default value (`null`, `0`, `false`) after deserialization.

  

Use for: derived/cached values, security-sensitive data (passwords, keys), non-serializable fields (threads, sockets, file handles).

  

```java

class Session implements Serializable {

private String userId;

private transient Socket connection; // excluded

private transient String cachedToken; // excluded

}

```

  

### `serialVersionUID`

  

A `long` that acts as the version stamp for a class. If the `serialVersionUID` in the stream doesn't match the class's `serialVersionUID`, deserialization throws `InvalidClassException`.

  

If you don't declare it, the JVM **computes one automatically** from the class structure. Any change to the class (even adding a method) can change the computed UID, breaking deserialization of previously saved data.

  

```java

class Config implements Serializable {

@Serial

private static final long serialVersionUID = 1L;

private String host;

private int port;

}

```

  

**Rule**: Always declare `serialVersionUID` explicitly in any `Serializable` class.

  

### Serialization Pitfalls

  

| Pitfall | Impact | Mitigation |

|---|---|---|

| No `serialVersionUID` | Class refactor breaks deserialization | Declare it explicitly |

| Version mismatch | `InvalidClassException` at runtime | Increment `serialVersionUID` deliberately |

| Deserialization of untrusted data | **Remote code execution** | Use `ObjectInputFilter` (Java 9+); prefer JSON/Protobuf |

| Serializing `HashMap` with custom `equals`/`hashCode` | Keys may not be found after deserialization | Implement `readObject` to rehash |

| Parent class not Serializable | Parent fields not saved | Parent must have a no-arg constructor |

| Memory bomb | Attacker sends a stream with millions of nested objects | Set max depth/array size with `ObjectInputFilter` |

  

**Security warning**: Java deserialization has been the source of critical CVEs (Apache Commons Collections exploit, etc.). Never deserialize data from untrusted sources without `ObjectInputFilter` or better yet, migrate to a safe format.

  

```java

ObjectInputStream ois = new ObjectInputStream(inputStream);

ois.setObjectInputFilter(info -> {

if (info.depth() > 5) return ObjectInputFilter.Status.REJECTED;

if (info.arrayLength() > 1000) return ObjectInputFilter.Status.REJECTED;

return ObjectInputFilter.Status.ALLOWED;

});

```

  

---

  

## 5. RandomAccessFile — Special Section

  

### How It Differs from Streams

  

| | Streams | `RandomAccessFile` |

|---|---|---|

| Access pattern | Sequential only | Random (seek anywhere) |

| Direction | Read-only OR write-only | Both in one object |

| Hierarchy | `InputStream`/`OutputStream` | Standalone (`DataInput` + `DataOutput`) |

| Position tracking | Implicit (linear) | Explicit via `getFilePointer()` / `seek()` |

  

### File Pointer

  

`RandomAccessFile` maintains a single **file pointer** — an offset in bytes from the start of the file.

  

```

File: [ H | e | l | l | o | | W | o | r | l | d ]

0 1 2 3 4 5 6 7 8 9 10

^

file pointer = 6

```

  

- `seek(long pos)`: Move pointer to absolute position

- `getFilePointer()`: Returns current pointer position

- `length()`: Returns file size in bytes

- `seek(raf.length())`: Moves to end (for appending)

- `skipBytes(int n)`: Advance pointer by n bytes

  

### Real-World Use Cases

  

| Use Case | Why RAF |

|---|---|

| Fixed-width record databases | Read record N by seeking to `N * recordSize` |

| Log file tailing | Seek to last known position, read new content |

| B-tree / index files | Navigate to arbitrary block offsets |

| Patch/in-place file update | Overwrite specific bytes without rewriting whole file |

| Memory-mapped file companion | When NIO `MappedByteBuffer` is overkill |

  

```java

// Fixed-width record store: 4 bytes int ID + 32 bytes name = 36 bytes/record

static final int RECORD_SIZE = 36;

  

void updateRecord(RandomAccessFile raf, int index, int id, String name) throws IOException {

raf.seek((long) index * RECORD_SIZE);

raf.writeInt(id);

byte[] nameBytes = name.getBytes(StandardCharsets.UTF_8);

byte[] padded = Arrays.copyOf(nameBytes, 32);

raf.write(padded);

}

  

int readId(RandomAccessFile raf, int index) throws IOException {

raf.seek((long) index * RECORD_SIZE);

return raf.readInt();

}

```

  

---

  

## 6. Encoding & Characters

  

### The Charset Problem

  

Every text file is bytes on disk. To interpret bytes as characters, you need to know the **encoding**. The encoding is metadata — it's not stored in the file (except for Unicode BOM, which is optional and fragile).

  

### Why `FileReader`/`FileWriter` Are Dangerous

  

```java

new FileReader("file.txt")

// equivalent to:

new InputStreamReader(new FileInputStream("file.txt"), Charset.defaultCharset())

```

  

`Charset.defaultCharset()` is determined at JVM startup from the system locale. On:

- Linux (en_US.UTF-8): UTF-8

- Windows: Cp1252 (Western European)

- Some CI/CD containers: US-ASCII

  

**A file written on Linux may silently corrupt on Windows** — special characters (€, ñ, 中) become garbage bytes.

  

Since Java 17, `Charset.defaultCharset()` defaults to UTF-8 by default on all platforms (JEP 400), but explicitly specifying charset is still best practice.

  

### The Right Pattern

  

```java

// Reading — always explicit charset

try (BufferedReader br = new BufferedReader(

new InputStreamReader(new FileInputStream("file.txt"), StandardCharsets.UTF_8))) {

// ...

}

  

// Writing — always explicit charset

try (BufferedWriter bw = new BufferedWriter(

new OutputStreamWriter(new FileOutputStream("file.txt"), StandardCharsets.UTF_8))) {

// ...

}

  

// Java 11+ — Files API (always specify charset)

List<String> lines = Files.readAllLines(Path.of("file.txt"), StandardCharsets.UTF_8);

Files.writeString(Path.of("file.txt"), content, StandardCharsets.UTF_8);

```

  

### Common Charsets

  

| Constant | Name | Use |

|---|---|---|

| `StandardCharsets.UTF_8` | UTF-8 | Universal default |

| `StandardCharsets.UTF_16` | UTF-16 | Java internal string encoding |

| `StandardCharsets.ISO_8859_1` | Latin-1 | Legacy Western European |

| `StandardCharsets.US_ASCII` | ASCII | 7-bit pure ASCII |

  

---

  

## 7. Performance & Optimization

  

### The Cost of Unbuffered I/O

  

```

Unbuffered FileInputStream.read() → JNI boundary → OS read() syscall → kernel → disk

~1–10 µs per call

```

  

For a 1MB file read byte-by-byte: ~1,000,000 syscalls × 1µs = **~1 second**.

With 8KB buffer: 128 reads × 1µs = **~0.1 ms** — 10,000x faster.

  

### Buffering Decision Table

  

| Scenario | Buffer? | Buffer Size |

|---|---|---|

| Small file read (<1MB) | Yes, default 8KB | 8192 |

| Large file sequential copy | Yes, larger | 64KB–256KB |

| Fixed-size record reads | Match record size | `N * recordSize` |

| Network stream (TCP) | Yes, match MTU | 4096–8192 |

| Memory-only streams | Unnecessary | — |

| NIO with `MappedByteBuffer` | Unnecessary | OS handles it |

  

### Best Practices for Large Files

  

1. **Use NIO `Files.copy()` or `FileChannel.transferTo()`** — OS-level zero-copy, avoids Java heap entirely.

2. **Memory-mapped I/O** (`FileChannel.map()`) for random-access large files — OS manages paging.

3. **Stream API** (`Files.lines()`) for line-by-line text — lazy, O(1) memory.

4. **Avoid loading entire file into memory** (`Files.readAllBytes()`) for files > 100MB.

5. **Buffer size tuning**: For disk I/O, 64KB–256KB buffers align with OS read-ahead. For network, 8KB is typically optimal.

  

```java

// Zero-copy large file transfer (NIO)

try (FileChannel src = FileChannel.open(Path.of("source.bin"), StandardOpenOption.READ);

FileChannel dst = FileChannel.open(Path.of("dest.bin"),

StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {

src.transferTo(0, src.size(), dst);

}

  

// Memory-mapped read (best for random-access large files)

try (FileChannel fc = FileChannel.open(Path.of("data.bin"), StandardOpenOption.READ)) {

MappedByteBuffer buf = fc.map(FileChannel.MapMode.READ_ONLY, 0, fc.size());

int value = buf.getInt(1024); // random access at byte 1024

}

```

  

---

  

## 8. Modern Java I/O: `java.io` vs `java.nio`

  

### Overview Comparison

  

| | `java.io` | `java.nio` |

|---|---|---|

| Model | Blocking, stream-oriented | Non-blocking available, buffer-oriented |

| Abstraction | Streams (sequential) | Channels + Buffers |

| Direction | One-way (read OR write) | Bidirectional (Channel reads AND writes) |

| Non-blocking? | No | Yes (via `Selector`) |

| Memory | Byte-by-byte / char-by-char | Works on `ByteBuffer` chunks |

| Large files | `BufferedInputStream` | `MappedByteBuffer` (zero-copy) |

| File ops | Limited | `Files`, `Path`, `DirectoryStream` |

| Introduced | Java 1.0 | Java 1.4 (NIO), Java 7 (NIO.2) |

  

### `java.nio` Core Concepts

  

**Channel**: A bidirectional conduit to a resource (file, socket). Reads/writes into/from `ByteBuffer`.

  

**Buffer**: A fixed-capacity container with a `position`, `limit`, and `capacity`. You write data to the buffer, then `flip()` it before reading.

  

```java

// NIO Channel + Buffer pattern

try (FileChannel fc = FileChannel.open(Path.of("file.bin"),

StandardOpenOption.READ, StandardOpenOption.WRITE)) {

ByteBuffer buf = ByteBuffer.allocate(1024);

int bytesRead = fc.read(buf);

buf.flip(); // switch from write mode to read mode

while (buf.hasRemaining()) {

byte b = buf.get();

}

}

```

  

**`Path` + `Files` (NIO.2)**: The modern replacement for `java.io.File`. Immutable, composable, charset-aware, and supports symbolic links.

  

```java

Path p = Path.of("/data", "records", "2024.csv");

boolean exists = Files.exists(p);

Files.copy(source, dest, StandardCopyOption.REPLACE_EXISTING);

Files.move(source, dest, StandardCopyOption.ATOMIC_MOVE);

long size = Files.size(p);

Stream<Path> children = Files.list(p.getParent());

```

  

### When to Prefer NIO

  

| Use Case | Prefer |

|---|---|

| Sequential text file processing | `java.io` + `BufferedReader` |

| Sequential binary copy | `Files.copy()` (NIO.2) |

| Large random-access files | `FileChannel` + `MappedByteBuffer` |

| Network server (many connections) | `java.nio` + `Selector` (non-blocking) |

| File system operations | `java.nio.file.Files` always |

| Interoperability with old APIs | `java.io` |

  

---

  

## 9. Common Interview Questions

  

### Q: `PrintWriter` vs `BufferedWriter`

  

| | `PrintWriter` | `BufferedWriter` |

|---|---|---|

| Throws checked exceptions? | No (silently catches) | Yes (`IOException`) |

| Formatted output? | Yes (`printf`, `format`) | No |

| Buffered internally? | No (unless you wrap a `BufferedWriter`) | Yes |

| `newLine()`? | No (use `println()`) | Yes |

| Detect errors? | `checkError()` | Exceptions surface naturally |

  

**Correct combo**: `new PrintWriter(new BufferedWriter(new OutputStreamWriter(fos, UTF_8)))`.

  

### Q: `DataInputStream` vs `ObjectInputStream`

  

| | `DataInputStream` | `ObjectInputStream` |

|---|---|---|

| Handles | Java primitives | Full Java object graphs |

| Protocol | Simple type-prefixed binary | Full serialization protocol |

| Performance | Fast | Expensive (class descriptor overhead) |

| Versioning | Manual (you own the format) | `serialVersionUID` based |

| Security risk | None | High (untrusted deserialization) |

  

### Q: `FileReader` vs `InputStreamReader`

  

| | `FileReader` | `InputStreamReader` |

|---|---|---|

| Charset | Platform default (dangerous) | Explicitly specified |

| Source | Only `File` | Any `InputStream` |

| Flexibility | None | Wraps any byte source (network, memory, etc.) |

  

**Always use `InputStreamReader` with explicit charset.** `FileReader` is a convenience shortcut that creates subtle bugs.

  

### Trick Scenarios

  

**Q: You create a `BufferedOutputStream`, write to it, and the file is empty. Why?**

A: You forgot to call `flush()` or `close()`. The data is still in the buffer.

  

**Q: Can you read a file that's being written by another process?**

A: Yes, using `RandomAccessFile` in `"r"` mode or `FileChannel` with `READ` option. There are no guaranteed semantics, but you can read whatever bytes are committed. On Linux, this is fine. On Windows, file locking may block it.

  

**Q: What happens if you call `readObject()` and the class has a new field added since serialization?**

A: If `serialVersionUID` matches, the new field gets its default value. If `serialVersionUID` doesn't match, `InvalidClassException` is thrown.

  

**Q: Why does `ObjectOutputStream` need to be created before `ObjectInputStream` in a pipe?**

A: `ObjectOutputStream` writes the stream header first. `ObjectInputStream` reads and validates the header on construction. If the reader is created first, it blocks waiting for the header that hasn't been written yet — **deadlock**.

  

**Q: What is the return value of `InputStream.read()` and why is it `int` and not `byte`?**

A: `read()` returns `int` (0–255) to accommodate the EOF sentinel value of `-1`. A `byte` can only hold -128 to 127 and cannot represent 256 distinct values. The extra bit is used for EOF.

  

---

  

## 10. Best Practices Cheat Sheet

  

### What to Use Where

  

| Context | Input | Output |

|---|---|---|

| Competitive programming (fast I/O) | `BufferedReader` + `StreamTokenizer` | `PrintWriter` |

| Text file processing | `BufferedReader` / `Files.lines()` | `BufferedWriter` + `OutputStreamWriter` |

| Binary file copy | `Files.copy()` | (same) |

| Config files (UTF-8 text) | `Files.readAllLines()` | `Files.writeString()` |

| Custom binary protocol | `DataInputStream` (buffered) | `DataOutputStream` (buffered) |

| Object persistence (internal) | `ObjectInputStream` | `ObjectOutputStream` |

| Large file random access | `RandomAccessFile` / `FileChannel` | `RandomAccessFile` / `FileChannel` |

| Backend file processing | `FileChannel.transferTo()` | (zero-copy) |

| Log writing | `PrintWriter` (autoFlush) | |

  

### Do's

  

- Always use try-with-resources for all I/O

- Always specify charset explicitly (`StandardCharsets.UTF_8`)

- Always buffer byte streams wrapping file or network sources

- Declare `serialVersionUID` in every `Serializable` class

- Use `Files` (NIO.2) for file system operations instead of `java.io.File`

- Call `checkError()` on `PrintWriter` when writing to files

- Use `Files.lines()` (lazy stream) for large text files instead of `readAllLines()`

  

### Don'ts

  

- Don't use `FileReader`/`FileWriter` in production (charset is implicit)

- Don't read from untrusted `ObjectInputStream` without `ObjectInputFilter`

- Don't use single-byte `read()` without buffering

- Don't call `reset()` on `ObjectOutputStream` sparingly — call it after each object when reusing the stream, or use a fresh stream

- Don't suppress or ignore `IOException` silently

- Don't use `new File()` for new code — use `Path.of()` instead

- Don't use `DataOutputStream` with `ObjectOutputStream` on the same stream (protocols clash)

  

### Competitive Programming Fast I/O

  

```java

public static void main(String[] args) throws IOException {

BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

PrintWriter pw = new PrintWriter(new BufferedWriter(new OutputStreamWriter(System.out)));

StreamTokenizer st = new StreamTokenizer(br);

  

st.nextToken(); int n = (int) st.nval;

long sum = 0;

for (int i = 0; i < n; i++) {

st.nextToken();

sum += (long) st.nval;

}

pw.println(sum);

pw.flush();

}

```

  

This pattern is ~3–5x faster than `Scanner` for large inputs, because `Scanner` uses regex internally.