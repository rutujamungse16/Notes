```markdown
# Core Java Interview Preparation Guide 2026

## Index

1. Introduction to Java  
2. Features of Java  
3. JDK, JRE, and JVM  
4. Program Structure and Main Method  
5. Data Types and Variables  
6. Type Casting  
7. Variable Scope and Lifetime  
8. Final Variables  
9. Operators in Java  
10. Control Flow – Conditional Statements  
11. Loops  
12. Interview Quick Reference & Common Traps  
13. Practice Scenarios & Edge Cases  
14. Time Complexity Considerations for Interviews  
15. Coding Best Practices for Interviews  
16. Experienced-Level Interview Questions (4+ Years)

---

## Part 1: Introduction to Java

### What is Java?

Java is a **general-purpose, object-oriented programming language** designed to be platform-independent. It follows the principle: *“Write Once, Run Anywhere (WORA)”*.[file:17]

**Memory Tip:** “Java = Platform Independence + Object-Oriented + Secure”[file:17]

### Why Java?

- **Platform Independent:** Compiled code runs on any device with JVM  
- **Object-Oriented:** Supports classes, inheritance, polymorphism, encapsulation  
- **Secure:** Built-in security features, automatic memory management  
- **Multithreaded:** Support for concurrent programming  
- **Robust:** Strong type checking, exception handling, garbage collection  
- **Enterprise-Ready:** Used in large-scale applications (financial, healthcare, e-commerce)[file:17]

---

## Part 2: Features of Java

### 1. Simple & Familiar Syntax

- Similar to C++, but simpler (no pointers, no operator overloading)  
- Easy to learn for developers with C/C++ background[file:17]

### 2. Object-Oriented

- Almost everything is an object (except primitives)  
- Supports encapsulation, inheritance, polymorphism, abstraction[file:17]

### 3. Platform Independent

- Compiled bytecode runs on any JVM  
- JVM handles OS-specific operations[file:17]

### 4. Secure

- No manual memory management (garbage collection)  
- Bytecode verification before execution  
- Security manager and crypto APIs[file:17]

### 5. Robust

- Strong type checking  
- Exception handling  
- Garbage collection helps prevent many memory leaks[file:17]

### 6. Multithreaded

- Built-in support for threads  
- Easier to write responsive and concurrent applications[file:17]

### 7. Distributed

- RMI, networking APIs  
- Designed with networked / distributed apps in mind[file:17]

### 8. High Performance

- JIT compilation  
- Runtime optimizations[file:17]

### 9. Dynamic

- Runtime type info  
- Dynamic class loading, reflection[file:17]

---

## Part 3: JDK, JRE, and JVM

### Understanding the Trinity

```text
┌─────────────────────────────────────────────────┐
│              JDK (Java Development Kit)         │
│  ┌──────────────────────────────────────────┐  │
│  │      JRE (Java Runtime Environment)      │  │
│  │  ┌────────────────────────────────────┐  │  │
│  │  │   JVM (Java Virtual Machine)       │  │  │
│  │  │                                    │  │  │
│  │  │  -  Executes bytecode               │  │  │
│  │  │  -  Heap, Stack, Garbage Collector  │  │  │
│  │  └────────────────────────────────────┘  │  │
│  │                                          │  │
│  │  -  Class Libraries (rt.jar, etc.)       │  │
│  │  -  Security Manager                     │  │
│  │  -  JIT Compiler                         │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
│  -  Compiler (javac)                             │
│  -  Debugger (jdb)                               │
│  -  Tools (jar, javadoc, etc.)                   │
└─────────────────────────────────────────────────┘
```[file:17]

### JVM (Java Virtual Machine)

- Platform-dependent implementation, but runs platform-independent bytecode  
- Executes bytecode, manages heap/stack, runs GC, enforces security[file:17]

**Memory Tip:** JVM = “OS Translator”.[file:17]

### JRE (Java Runtime Environment)

- JVM + core libraries + runtime components  
- No compiler / dev tools – used just to run apps[file:17]

### JDK (Java Development Kit)

- JRE + compiler (`javac`) + debugger + tools (`jar`, `javadoc`, etc.)  
- Used for development and running[file:17]

### Compilation & Execution Flow

```text
Source (.java)
    ↓ javac
Bytecode (.class)
    ↓ JVM
Machine code (OS-specific)
    ↓
Program runs
```[file:17]

**Interview trap:** You can run Java with only JRE, but you need JDK to compile.[file:17]

---

## Part 4: Program Structure and Main Method

### Anatomy of a Java Program

```java
package com.example.learning;

import java.util.Scanner;
import java.util.ArrayList;

public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
        for (String arg : args) {
            System.out.println("Argument: " + arg);
        }
    }

    public void greet(String name) {
        System.out.println("Hello, " + name);
    }
}
```[file:17]

### Main Method Breakdown

```java
public static void main(String[] args)
```[file:17]

- `public`: JVM must access it  
- `static`: no object needed  
- `void`: no return value  
- `main`: fixed name used by JVM  
- `String[] args`: command-line arguments[file:17]

**Memory Tip:** PSVM = Public Static Void Main.[file:17]

**Interview trap:** Non-`public` main compiles but fails at runtime (`NoSuchMethodError`).[file:17]

---

## Part 5: Data Types and Variables

### Data Types in Java

- **Primitive (8):** boolean, byte, short, int, long, float, double, char  
- **Reference:** classes, arrays, interfaces, enums[file:17]

**Memory Tip:** BSILDCFB = Byte, Short, Int, Long, Double, Char, Float, Boolean.[file:17]

### Primitive vs Reference

- Primitive stores value directly (usually on stack).  
- Reference stores address to object on heap; multiple refs can point to same object.[file:17]

**Interview trap:** Copying a reference does NOT copy the object; both refs see same changes.[file:17]

---

## Part 6: Type Casting

- **Widening:** smaller → larger type (byte→short→int→long→float→double), implicit, no cast.  
- **Narrowing:** larger → smaller type, explicit cast, may overflow or lose precision.[file:17]

**Interview trap:** `int big = 1000; byte b = (byte) big;` gives overflowed value, not 1000.[file:17]

---

## Part 7: Variable Scope and Lifetime

- Block, local, parameter, instance, static/class scope.  
- Lifetime tied to scope: block variables die at `}`, local at method end, instance at GC, static at JVM end.[file:17]

**Interview trap:** variable shadowing reduces readability; prefer different names.[file:17]

---

## Part 8: Final Variables

- `final` variable can be assigned only once.  
- `static final` usually used as constants.  
- Final reference: ref cannot change, object can (if mutable).[file:17]

**Interview trap:** `final int` vs `static final int` – one per object vs one per class.[file:17]

---

## Part 9: Operators in Java

Covers:

- Arithmetic, relational, logical, bitwise, ternary  
- Assignment and compound assignment  
- `instanceof`  
- Operator precedence and integer division pitfalls[file:17]

**Interview trap:** `i++` returns old value then increments, `++i` increments then returns.[file:17]

---

## Part 10: Control Flow – Conditionals

- `if`, `if-else`, `else-if` ladder  
- `switch` with `int`, `char`, `String`, enums  
- Enhanced switch (arrow syntax) in newer Java[file:17]

**Interview trap:** missing `{}` in if leads to unexpected extra statements executing.[file:17]

---

## Part 11: Loops

- `for`, enhanced `for-each`, `while`, `do-while`  
- `break`, `continue`, labeled `break/continue` for nested loops[file:17]

**Memory Tip:** do–while = “do first, check later”.[file:17]

---

## Part 12: Interview Quick Reference & Common Traps

Includes:

- Mnemonics for key concepts  
- 10 classic traps: nulls, `==` vs `equals`, integer division, array bounds, casting overflow, loop scope, float equality, dangling `else`, switch without `default`, mutable finals[file:17]

---

## Part 13: Practice Scenarios & Edge Cases

- Type conversion chain  
- Operator precedence expression  
- Logical short-circuit with side effects  
- Loop with `break` sum logic  
- Final variable assignment scenario[file:17]

---

## Part 14: Time Complexity Considerations

Table of common complexities (array access, loops, `ArrayList`, `HashMap`, string concatenation, etc.).[file:17]

---

## Part 15: Coding Best Practices for Interviews

- Meaningful names  
- Handle nulls and bounds  
- Right data structures  
- Flat, readable control flow  
- Comments that explain “why”, not “what”[file:17]

---

## Part 16: Experienced-Level Interview Questions (4+ Years)

### 16.1 Core Language and Memory

**Q1. Explain Java’s memory model: heap vs stack vs metaspace. When does an object become eligible for garbage collection?**  
A: Local variables and references live on the stack, objects live on the heap, and class metadata (class definitions, constant pool) is stored in metaspace in modern JVMs.[web:5][web:15] An object becomes eligible for GC when it is no longer reachable from any live thread or static reference through a chain of strong references.[web:5][web:15]

**Q2. Is Java pass-by-value or pass-by-reference? Give an example that confuses people.**  
A: Java is always pass-by-value. For objects, the value being passed is the reference, so both caller and callee see the same underlying object, but reassigning the parameter in the method does not change the caller’s reference.[web:5][web:6] This often confuses people when they try to “swap” two `Integer` or `String` references inside a method and expect the caller’s variables to be swapped.

**Q3. How would you diagnose and fix a `java.lang.OutOfMemoryError` in production?**  
A: First, identify the area (heap, metaspace, direct memory) from the error or logs; then capture a heap dump (`jmap`, JVM flags) and analyze it with tools like Eclipse MAT or VisualVM to locate leak suspects such as large unbounded collections or caches.[web:5][web:7] Finally, remove unintended strong references (e.g., static maps, listeners not removed) and introduce bounds/eviction policies where needed.[web:5][web:7]

---

### 16.2 Collections and Generics

**Q4. Internal differences between `ArrayList` and `LinkedList`. When would you prefer one over the other?**  
A: `ArrayList` is backed by a dynamic array, offering O(1) random access and amortized O(1) appends but O(n) insertion/removal in the middle due to shifting elements.[web:4][web:15] `LinkedList` is a doubly-linked list with cheap insertions/removals via iterators but O(n) access and worse cache locality; in practice, `ArrayList` is usually preferred except for very specific add/remove patterns.[web:4][web:15]

**Q5. What is the difference between `HashMap`, `LinkedHashMap`, and `TreeMap`?**  
A: `HashMap` provides O(1) average-time operations with no ordering guarantees, `LinkedHashMap` preserves insertion (or access) order using a linked list on top of the hash table, and `TreeMap` keeps keys sorted using a red–black tree with O(log n) operations.[web:5][web:15] Choice depends on whether you need ordering and the expected complexity.

**Q6. Explain type erasure in generics and a pitfall it causes.**  
A: Generics are implemented via type erasure: generic type information is used at compile time for type checks and then erased, so at runtime `List<String>` and `List<Integer>` both appear as `List`.[web:4][web:6] This prevents `instanceof List<String>` and can cause heap pollution when mixing raw types with parameterized types, leading to `ClassCastException` at runtime.[web:4][web:6]

---

### 16.3 Concurrency Basics

**Q7. What problems do `volatile` and `synchronized` solve? Can `volatile` replace `synchronized`?**  
A: `volatile` ensures visibility and ordering for reads/writes of a single variable but does not make compound actions atomic; `synchronized` provides mutual exclusion and establishes happens-before relationships for all variables accessed in the block.[web:4][web:11] `volatile` cannot replace `synchronized` for operations like `count++` that need atomicity in addition to visibility.[web:4][web:11]

**Q8. Explain thread safety for collections. When use `Collections.synchronizedList` vs `CopyOnWriteArrayList`?**  
A: `Collections.synchronizedList` wraps a list with a single lock, making all operations synchronized—suitable for moderate write frequency and mixed read/write workloads.[web:6][web:11] `CopyOnWriteArrayList` copies the array on each write but allows lock-free iteration and fast reads, so it fits read-heavy workloads with rare modifications, such as listener lists.[web:6][web:11]

**Q9. How does `ThreadLocal` work, and where can it cause memory leaks?**  
A: Each thread maintains a map of `ThreadLocal` to value, so each thread sees its own independent value.[web:4][web:11] In thread pools, if you don’t clear `ThreadLocal` values, the long-lived worker threads can retain references and cause memory leaks, so you should remove values when done.[web:4][web:11]

---

### 16.4 Exceptions, Design, and Best Practices

**Q10. How do you design your own exception hierarchy in a large codebase?**  
A: Define a small base custom exception (checked or unchecked), derive domain-specific exceptions (e.g., `ValidationException`, `BusinessException`, `IntegrationException`), and ensure each layer either handles or translates lower-level exceptions into meaningful domain ones.[web:2][web:5] Avoid throwing generic `Exception` from public APIs to keep contracts clear.[web:2][web:5]

**Q11. When choose checked vs unchecked exception?**  
A: Use checked exceptions for recoverable conditions the caller is expected to handle (validation, known IO issues), and unchecked for programming errors or unrecoverable states where forcing handling would just produce noisy boilerplate.[web:2][web:5]

**Q12. How do you ensure immutability of a class, and why is it useful?**  
A: Make the class effectively immutable by using `private final` fields, no setters, full initialization in the constructor, and defensive copies for mutable inputs/outputs.[web:5][web:6] Immutable objects are naturally thread-safe, easier to reason about, and ideal as cache keys or values in concurrent code.[web:5][web:6]

---

### 16.5 Practical Coding / Scenario Questions (4+ Years)

**Q13. Given a large list of integers, how would you find the top 10 numbers efficiently?**  
A: Maintain a min-heap (priority queue) of size 10: iterate once, push elements, and when the size exceeds 10, pop the smallest; this yields O(n log 10) ≈ O(n).[web:10][web:11] For smaller inputs, a full sort and taking the last 10 may be acceptable but is O(n log n).[web:10][web:11]

**Q14. How would you design a retry utility with exponential backoff? What do you consider?**  
A: Accept a functional interface (e.g., `Supplier<T>`), configure max retries, base delay, and multiplier, and implement retries with increasing sleep times and optional jitter to avoid thundering herd effects.[web:11][web:16] Consider which exceptions are retryable, honoring thread interrupts, logging policy, and capping the max backoff.[web:11][web:16]

**Q15. You see high CPU and long GC pauses in a Java service. Outline your approach.**  
A: Inspect GC logs and flags to see which collector is used and pause patterns, use profilers or thread dumps to locate hot methods or stuck threads, and look for excessive short-lived allocations or large retained object graphs such as unbounded caches.[web:5][web:7] Then tune heap/GC settings and fix code hotspots by reducing allocations, limiting cache size, and removing unnecessary object churn.[web:5][web:7]

---

**Last Updated:** February 2026  
**Prepared for:** Core Java Interview Preparation (4+ Years Experience Focus)
```