# Java Interview Preparation Guide 2026 — Top 100 Questions

> **Target:** Java 17 LTS + Java 21+ features | Theory + Coding | Complexity Analysis
> **Last Updated:** March 2026

---

## Table of Contents

| # | Section | Questions | Range |
|---|---------|-----------|-------|
| 1 | Core Java Fundamentals | 17 | Q1–Q17 |
| 2 | Collections Framework | 14 | Q18–Q31 |
| 3 | Multithreading & Concurrency | 17 | Q32–Q48 |
| 4 | Modern Java Features (8–21+) | 14 | Q49–Q62 |
| 5 | Exception Handling | 7 | Q63–Q69 |
| 6 | JVM & Memory Management | 9 | Q70–Q78 |
| 7 | Design Patterns | 9 | Q79–Q87 |
| 8 | Coding Problems | 13 | Q88–Q100 |
| — | Quick Reference Cheat Sheet | — | — |

---

## Memory Aid — Java Version Landmarks

```
J8  = Lambdas + Streams + Optional + Default Methods        (2014)
J9  = Modules (JPMS) + JShell + Private Interface Methods    (2017)
J11 = var + HTTP Client + String strip/isBlank     LTS       (2018)
J14 = Records (preview)                                      (2020)
J15 = Sealed Classes (preview) + Text Blocks                 (2020)
J16 = Records (final) + Pattern matching instanceof          (2021)
J17 = Sealed Classes (final) + Pattern switch (preview) LTS  (2021)
J21 = Virtual Threads + Seq Collections + Record Patterns + Pattern switch (final) LTS (2023)
J22 = Statements before super (preview) + Stream Gatherers   (2024)

Mnemonic: "Lambdas → Modules → var → Records → Sealed → Virtual Threads"
          "L-M-V-R-S-VT"  ➜  "Learn Modern Versatile Robust Scalable Virtual Tech"
```

---

# Section 1 — Core Java Fundamentals (Q1–Q17)

---

### Q1. What are the four pillars of OOP in Java? Explain each with a real-world example.

**Concept:** OOP is built on Encapsulation (data hiding), Abstraction (hiding complexity), Inheritance (code reuse via parent-child), and Polymorphism (one interface, multiple behaviors). Java enforces these through access modifiers, abstract classes/interfaces, `extends`/`implements`, and method overriding/overloading.

```java
// --- Encapsulation ---
public class BankAccount {
    private double balance; // hidden state

    public double getBalance() { return balance; }
    public void deposit(double amount) {
        if (amount > 0) balance += amount; // controlled access
    }
}

// --- Abstraction ---
public abstract class Shape {
    abstract double area(); // what, not how
}

// --- Inheritance ---
public class Circle extends Shape {
    private double radius;
    Circle(double r) { this.radius = r; }

    @Override
    double area() { return Math.PI * radius * radius; } // Polymorphism too
}

// --- Polymorphism (runtime) ---
Shape s = new Circle(5);
System.out.println(s.area()); // 78.5398... — Circle's implementation called
```

**Output:** `78.53981633974483`

| Pillar | Mechanism | Keyword/Feature |
|--------|-----------|-----------------|
| Encapsulation | Access modifiers | `private`, getters/setters |
| Abstraction | Abstract classes, interfaces | `abstract`, `interface` |
| Inheritance | Class hierarchy | `extends`, `implements` |
| Polymorphism | Method overriding/overloading | `@Override`, method dispatch |

**Complexity:** N/A (conceptual)

**Edge Cases:** Java doesn't support multiple class inheritance (diamond problem). Use interfaces instead. Since Java 8, interfaces can have `default` methods, blurring the abstraction/inheritance line.

**Interview Tip:** Interviewers want concrete examples, not textbook definitions. Relate each pillar to a system you've built. Mention Java 17 sealed classes as a modern abstraction tool.

**Follow-ups:** How does `sealed` enhance abstraction? Can you achieve polymorphism without inheritance?

---

### Q2. Explain access modifiers in Java. What is the exact visibility of each?

**Concept:** Java has four access levels controlling member visibility: `private` (class-only), default/package-private (package-only), `protected` (package + subclass), and `public` (everywhere). Choosing the right modifier is key to encapsulation.

```java
package com.example;

public class AccessDemo {
    private   int a = 1;   // only within AccessDemo
              int b = 2;   // within com.example package
    protected int c = 3;   // com.example + subclasses anywhere
    public    int d = 4;   // everywhere

    private void secret() { }       // only this class
    void packageMethod() { }        // package-level default
    protected void familyMethod() { } // package + subclass
    public void openMethod() { }    // unrestricted
}
```

| Modifier | Same Class | Same Package | Subclass (other pkg) | Everywhere |
|----------|:----------:|:------------:|:--------------------:|:----------:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| default | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**Memory Aid:** Think **P-D-P-P** (Private → Default → Protected → Public) as increasing circles of visibility: **"Please Don't Punch People"**

**Edge Cases:**
- `protected` members are accessible in the same package even without subclassing — a common surprise.
- Top-level classes can only be `public` or default (not `private` or `protected`).
- Inner classes can be `private`.

**Interview Tip:** Be precise about `protected` — it's not "subclass only." It includes the whole package. This is a frequent trick question.

**Follow-ups:** What access does an inner class have to its enclosing class? How does module system (`module-info.java`) interact with access modifiers?

---

### Q3. Explain `this` vs `super` keywords with examples.

**Concept:** `this` refers to the current object instance and is used for field disambiguation, constructor chaining, and passing the current object. `super` refers to the parent class and is used to call parent constructors or overridden methods.

```java
class Animal {
    String name;
    Animal(String name) { this.name = name; } // 'this' disambiguates
    void speak() { System.out.println(name + " makes a sound"); }
}

class Dog extends Animal {
    String breed;

    Dog(String name, String breed) {
        super(name);             // MUST be first statement — calls Animal(String)
        this.breed = breed;      // 'this' for current class field
    }

    @Override
    void speak() {
        super.speak();           // calls Animal.speak()
        System.out.println(name + " barks! Breed: " + breed);
    }

    Dog getThis() { return this; } // return current instance
}

// Usage
Dog d = new Dog("Rex", "Husky");
d.speak();
```

**Output:**
```
Rex makes a sound
Rex barks! Breed: Husky
```

| Feature | `this` | `super` |
|---------|--------|---------|
| Refers to | Current object | Parent class |
| Constructor call | `this(args)` — must be first line | `super(args)` — must be first line |
| Can be used in static context? | ❌ | ❌ |
| Chaining | Chain constructors in same class | Call parent constructor |

**Edge Cases:**
- `this()` and `super()` cannot coexist in the same constructor — both require the first line.
- If no explicit `super()`, compiler inserts `super()` (no-arg) automatically; compile error if parent lacks a no-arg constructor.
- Java 22 previews "statements before `super()`" — lets you validate arguments first.

**Interview Tip:** Mention the Java 22 preview feature for bonus points. Interviewers love candidates who track JEP evolution.

**Follow-ups:** What happens if a parent has no no-arg constructor and child doesn't call `super(args)`? Can `this` be `null`?

---

### Q4. What is the difference between `static` and instance members?

**Concept:** Static members belong to the class itself and are shared across all instances; they're loaded once when the class is loaded. Instance members belong to each object and are created per instantiation. Static context cannot access instance members directly.

```java
public class Counter {
    static int totalCount = 0;  // shared — one copy in Method Area
    int instanceId;             // per-object — stored in Heap

    Counter() {
        totalCount++;                     // class-level increment
        this.instanceId = totalCount;     // instance-level assignment
    }

    static void showTotal() {
        System.out.println("Total: " + totalCount);
        // System.out.println(instanceId); // ❌ compile error
    }

    void showInstance() {
        System.out.println("Instance #" + instanceId + " of " + totalCount);
    }
}

// Usage
Counter c1 = new Counter();
Counter c2 = new Counter();
Counter.showTotal();    // Total: 2
c1.showInstance();      // Instance #1 of 2
```

| Aspect | `static` | Instance |
|--------|----------|----------|
| Belongs to | Class | Object |
| Memory | Method Area (one copy) | Heap (per object) |
| Access | `ClassName.member` | `objectRef.member` |
| Can access instance members? | ❌ (no `this`) | ✅ |
| Can access static members? | ✅ | ✅ |
| Loaded when? | Class loading | Object creation |

**Edge Cases:**
- Static methods can be "hidden" (not overridden) in subclasses — this is **method hiding**, not polymorphism.
- `static` blocks execute once during class loading in textual order.
- Static fields of an interface are implicitly `public static final`.

**Interview Tip:** Emphasize that `static` methods resolve at compile time (early binding), while instance methods use late binding (vtable lookup). This is a key JVM internals distinction.

**Follow-ups:** Can static methods be overridden? What's the difference between class loading and initialization?

---

### Q5. How does immutability work in Java? How to create an immutable class?

**Concept:** An immutable object's state cannot change after construction. `String` is the classic example. Creating a custom immutable class requires: final class, private final fields, no setters, defensive copies of mutable fields, and returning copies from getters.

```java
// Traditional approach (pre-Java 16)
public final class Money {
    private final String currency;
    private final double amount;
    private final List<String> tags;  // mutable field!

    public Money(String currency, double amount, List<String> tags) {
        this.currency = currency;
        this.amount = amount;
        this.tags = List.copyOf(tags);  // defensive copy — Java 10+
    }

    public String getCurrency() { return currency; }
    public double getAmount() { return amount; }
    public List<String> getTags() { return tags; } // already unmodifiable
}

// Modern approach — Java 16+ Record (immutable by design)
public record MoneyRecord(String currency, double amount, List<String> tags) {
    public MoneyRecord {  // compact constructor
        tags = List.copyOf(tags);  // defensive copy
    }
}
```

**Immutability Checklist:**

| Rule | Why |
|------|-----|
| `final` class | Prevent subclass from adding mutable state |
| `private final` fields | No external modification |
| No setters | No state mutation |
| Defensive copy in constructor | Caller can't mutate internal state via original reference |
| Return copies from getters | Caller can't mutate internal state via returned reference |

**Memory Aid: "FNDDR"** — **F**inal class, **N**o setters, **D**efensive copies in, **D**efensive copies out, p**R**ivate final fields.

**Edge Cases:**
- `final` on a reference prevents reassignment but not mutation of the referred object (`final List<> l` — you can still `l.add()`).
- Records give you immutability "almost free" but you still need defensive copies for mutable field types.
- Date fields: use `java.time` (already immutable) instead of `java.util.Date`.

**Interview Tip:** Always mention Records as the modern approach. Show you know both the old-school and modern patterns. Highlight that `List.copyOf()` returns an unmodifiable list and throws `NullPointerException` on null elements.

**Follow-ups:** Is `String` truly immutable or can reflection break it? How do Records handle equals/hashCode?

---

### Q6. Explain the `final` keyword in all its contexts.

**Concept:** `final` prevents modification: final variables can't be reassigned, final methods can't be overridden, and final classes can't be subclassed. It's a compile-time guarantee with JVM optimization implications.

```java
// 1. Final variable
final int MAX = 100;
// MAX = 200; // ❌ compile error

// 2. Final reference (object itself is mutable!)
final List<String> list = new ArrayList<>();
list.add("Hello"); // ✅ OK — modifying object, not reference
// list = new ArrayList<>(); // ❌ compile error — reassigning reference

// 3. Final method — cannot be overridden
class Parent {
    final void critical() { System.out.println("Cannot override me"); }
}

// 4. Final class — cannot be extended
final class Utility {
    static void helper() { /* ... */ }
}
// class SubUtil extends Utility {} // ❌ compile error

// 5. Final parameter — cannot be reassigned inside method
void process(final String input) {
    // input = "new"; // ❌ compile error
    System.out.println(input.toUpperCase()); // ✅ OK
}

// 6. Blank final — assigned exactly once in constructor
class Config {
    final String env;   // blank final
    Config(String env) { this.env = env; } // must assign here
}
```

| Context | What `final` does | Common use |
|---------|-------------------|------------|
| Local variable | No reassignment | Effectively-final for lambdas |
| Instance field | Must be assigned by end of constructor | Immutable classes |
| Static field | Constant (with `static`) | `public static final` constants |
| Method | Cannot be overridden | Template method pattern |
| Class | Cannot be subclassed | Immutability, security (`String`) |
| Parameter | No reassignment in method body | Clarity, lambda capture |

**Edge Cases:**
- **Effectively final:** Since Java 8, local variables used in lambdas don't need explicit `final` if they're never reassigned. The compiler treats them as effectively final.
- **Blank finals:** Instance `final` fields can be uninitialized at declaration if every constructor assigns them.
- **`final` does NOT mean thread-safe** — but the JVM guarantees that `final` fields are visible to all threads after construction (JMM guarantee).

**Interview Tip:** The JMM guarantee about `final` fields is advanced and impresses interviewers. Also mention that `final` can enable JIT optimizations (inlining, constant folding).

**Follow-ups:** What are effectively final variables? Does `final` impact performance? How does `final` interact with serialization?

---

### Q7. What is the difference between `==` and `.equals()`?

**Concept:** `==` compares references (memory addresses) for objects, and values for primitives. `.equals()` compares logical content and can be overridden. The default `Object.equals()` uses `==`, so custom classes must override it for meaningful comparison.

```java
// Primitive comparison — == compares values
int a = 5, b = 5;
System.out.println(a == b);          // true

// String comparison — tricky!
String s1 = "hello";                 // String pool
String s2 = "hello";                 // same pool reference
String s3 = new String("hello");     // new heap object

System.out.println(s1 == s2);        // true  (same pool ref)
System.out.println(s1 == s3);        // false (different objects)
System.out.println(s1.equals(s3));   // true  (same content)

// Integer cache surprise
Integer x = 127, y = 127;
System.out.println(x == y);          // true  (cached: -128 to 127)
Integer p = 128, q = 128;
System.out.println(p == q);          // false (outside cache)
System.out.println(p.equals(q));     // true

// Custom class — must override equals + hashCode
public record Point(int x, int y) {} // Records auto-generate both!
Point p1 = new Point(1, 2);
Point p2 = new Point(1, 2);
System.out.println(p1 == p2);        // false (different objects)
System.out.println(p1.equals(p2));   // true  (Record auto-equals)
```

| Aspect | `==` | `.equals()` |
|--------|------|-------------|
| Compares | References (objects) / Values (primitives) | Logical equality |
| Default behavior | Identity check | Same as `==` (Object class) |
| Can override? | No | Yes |
| Null safe? | Yes (`null == null` is true) | No (`null.equals()` throws NPE) |
| For primitives? | ✅ | ❌ (primitives don't have methods) |

**Edge Cases:**
- **Integer cache:** `-128` to `127` are cached by `Integer.valueOf()`. Autoboxing uses `valueOf()`, so `==` works within this range.
- **String pool:** Literal strings are interned. `new String()` bypasses the pool.
- **Always override `hashCode()` when overriding `equals()`** — contract requires that equal objects have equal hash codes.

**Interview Tip:** Demonstrate knowledge of the Integer cache and String pool — these are classic trick questions. Mention `Objects.equals(a, b)` for null-safe comparison.

**Follow-ups:** What is the equals-hashCode contract? What happens if you override equals but not hashCode?

---

### Q8. Explain method overloading vs overriding. How does the JVM resolve each?

**Concept:** Overloading is compile-time polymorphism — same method name with different parameter lists in the same class. Overriding is runtime polymorphism — a subclass provides a specific implementation of a parent's method. The JVM uses static dispatch for overloading and dynamic dispatch (vtable) for overriding.

```java
class Calculator {
    // Overloading — resolved at COMPILE time by parameter types
    int add(int a, int b)          { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c)   { return a + b + c; }
}

class Animal {
    // Method to be overridden
    String sound() { return "..."; }
}

class Cat extends Animal {
    @Override  // Overriding — resolved at RUNTIME by actual object type
    String sound() { return "Meow"; }
}

// Runtime polymorphism in action
Animal a = new Cat();
System.out.println(a.sound()); // "Meow" — Cat's version, not Animal's
```

**Output:** `Meow`

| Aspect | Overloading | Overriding |
|--------|-------------|------------|
| Binding | Compile-time (static) | Runtime (dynamic) |
| Where | Same class (or inherited) | Subclass |
| Parameters | Must differ | Must be identical |
| Return type | Can differ | Same or covariant (subtype) |
| Access | Can differ | Same or wider (not narrower) |
| Exceptions | Can differ | Same or narrower (not wider for checked) |
| `static` methods | Can overload | Cannot override (only hide) |
| `private` methods | Can overload | Cannot override (not inherited) |
| `final` methods | Can overload | Cannot override |

**Edge Cases:**
- Overloading resolution considers widening (`int` → `long`), boxing (`int` → `Integer`), and varargs, in that priority order.
- Covariant return: overriding method can return a subtype (e.g., `Object` → `String`).
- Bridge methods: the compiler generates synthetic bridge methods for covariant returns and generics to maintain bytecode compatibility.

**Interview Tip:** Explain the JVM dispatch mechanism: overloading → `invokestatic`/`invokevirtual` with compile-time signature; overriding → `invokevirtual` with vtable lookup at runtime. Mentioning `invokedynamic` (used for lambdas) scores extra points.

**Follow-ups:** Can you overload by return type alone? What are bridge methods? How does `invokedynamic` work?

---

### Q9. What is the difference between abstract classes and interfaces? When to use which?

**Concept:** Abstract classes provide partial implementation with state (fields); interfaces define contracts with no instance state (pre-Java 8) and now support default/static/private methods. Since Java 8+, the line has blurred, but abstract classes still hold state and constructors while interfaces offer multiple inheritance of behavior.

```java
// Abstract class — has state, constructors, mixed access
public abstract class Vehicle {
    private String vin;           // instance state
    protected int speed = 0;

    Vehicle(String vin) { this.vin = vin; } // constructor

    abstract void accelerate();    // must implement
    void brake() { speed = 0; }   // concrete method
}

// Interface — Java 17+ style with default, static, private methods
public interface Electric {
    int MAX_CHARGE = 100;                    // implicitly public static final

    void charge();                           // abstract

    default double range(double efficiency) { // default method (Java 8+)
        return calculateBase() * efficiency;
    }

    static Electric create() {               // static method (Java 8+)
        return () -> System.out.println("Charging...");
    }

    private double calculateBase() {         // private method (Java 9+)
        return MAX_CHARGE * 4.5;
    }
}

// Sealed interface — Java 17 (restrict implementations)
public sealed interface Payment permits CreditCard, UPI, Cash {}
record CreditCard(String number) implements Payment {}
record UPI(String id) implements Payment {}
record Cash(double amount) implements Payment {}
```

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Fields | Instance + static, any access | `public static final` only |
| Constructors | ✅ Yes | ❌ No |
| Methods | Abstract + concrete | Abstract + default + static + private |
| Multiple inheritance | ❌ Single extends | ✅ Multiple implements |
| State (instance fields) | ✅ | ❌ |
| Access modifiers on methods | Any | `public` (abstract/default) or `private` |
| `sealed` support | ✅ (Java 17) | ✅ (Java 17) |

**Decision Guide:**
- Use **interface** when: defining a capability/contract, need multiple inheritance, or API boundary.
- Use **abstract class** when: sharing state among related classes, need constructors, or partial implementation with protected helpers.

**Interview Tip:** Mention sealed interfaces (Java 17) as the modern way to define closed type hierarchies. Combined with records, they enable algebraic data types in Java — this is very relevant for 2026 interviews.

**Follow-ups:** What happens with diamond problem in default methods? How do sealed classes change design decisions?

---

### Q10. Explain the String pool, `String` vs `StringBuilder` vs `StringBuffer`.

**Concept:** `String` is immutable and stored in a special memory region (String pool) for reuse. `StringBuilder` is a mutable, non-thread-safe alternative for string manipulation. `StringBuffer` is the thread-safe (synchronized) version of `StringBuilder`. For single-threaded concatenation, `StringBuilder` is preferred.

```java
// String pool behavior
String a = "Java";                // pool
String b = "Java";                // same pool reference
String c = new String("Java");    // heap (bypasses pool)
String d = c.intern();            // explicitly add to pool / get pool ref

System.out.println(a == b);       // true  — same pool object
System.out.println(a == c);       // false — different objects
System.out.println(a == d);       // true  — intern returns pool ref

// StringBuilder vs StringBuffer
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World").append("!").reverse();
System.out.println(sb);           // !dlroW olleH

// Performance comparison (concatenation in loops)
// ❌ BAD — creates O(n) String objects
String result = "";
for (int i = 0; i < 1000; i++) result += i; // O(n²)

// ✅ GOOD — single mutable buffer
StringBuilder fast = new StringBuilder();
for (int i = 0; i < 1000; i++) fast.append(i); // O(n)
```

| Feature | `String` | `StringBuilder` | `StringBuffer` |
|---------|----------|-----------------|----------------|
| Mutable | ❌ | ✅ | ✅ |
| Thread-safe | ✅ (immutable) | ❌ | ✅ (synchronized) |
| Performance | Slow for concat | Fast | Slower than SB (sync overhead) |
| Memory | String pool + heap | Heap only | Heap only |
| Since | JDK 1.0 | JDK 1.5 | JDK 1.0 |

**Edge Cases:**
- Java compiler optimizes `"a" + "b"` to `"ab"` at compile time (constant folding).
- Since Java 9, `+` in loops uses `invokedynamic` with `StringConcatFactory` — still slower than explicit `StringBuilder` for large loops but much better than pre-J9.
- `String.intern()` can cause memory leaks if overused (interned strings live in the string pool which is in the heap since Java 7).

**Interview Tip:** Know that the String pool moved from PermGen to Heap in Java 7. Mention `invokedynamic`-based string concatenation (Java 9+) to show depth.

**Follow-ups:** How does `String.intern()` work internally? What are compact strings (Java 9)? How does the JIT optimize string concatenation?

---

### Q11. What are Wrapper classes? Explain autoboxing, unboxing, and caching.

**Concept:** Wrapper classes (`Integer`, `Double`, etc.) wrap primitives as objects for use with generics/collections. Autoboxing converts primitives to wrappers automatically; unboxing does the reverse. The JVM caches certain wrapper values for performance.

```java
// Autoboxing and unboxing
Integer boxed = 42;              // autoboxing: int → Integer (valueOf)
int unboxed = boxed;             // unboxing: Integer → int (intValue)

// Cache trap!
Integer a = 127, b = 127;
System.out.println(a == b);      // true  — cached

Integer c = 128, d = 128;
System.out.println(c == d);      // false — outside cache!
System.out.println(c.equals(d)); // true  — always use equals for wrappers

// Unboxing NPE danger
Integer nullVal = null;
// int x = nullVal;              // ❌ NullPointerException at runtime!

// Performance: avoid autoboxing in tight loops
// ❌ BAD
Long sum = 0L;
for (long i = 0; i < 1_000_000; i++) sum += i; // creates ~1M Long objects

// ✅ GOOD
long sumPrim = 0L;
for (long i = 0; i < 1_000_000; i++) sumPrim += i;
```

**Cache Ranges:**

| Wrapper | Cached Range | Configurable? |
|---------|-------------|---------------|
| `Byte` | -128 to 127 (all) | No |
| `Short` | -128 to 127 | No |
| `Integer` | -128 to 127 | Yes (`-XX:AutoBoxCacheMax`) |
| `Long` | -128 to 127 | No |
| `Character` | 0 to 127 | No |
| `Boolean` | `TRUE`, `FALSE` | No (only two values) |
| `Float` | None | — |
| `Double` | None | — |

**Interview Tip:** Explain that `Integer.valueOf(int)` uses the cache while `new Integer(int)` (deprecated since Java 9, removed in Java 16) always creates a new object. This is why `==` behaves differently for cached vs uncached values.

**Follow-ups:** Why aren't `Float`/`Double` cached? What is `-XX:AutoBoxCacheMax`? How does Project Valhalla (value types) aim to solve boxing overhead?

---

### Q12. Explain `hashCode()` and `equals()` contract. Why must they be consistent?

**Concept:** The contract states: if `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must be true. The reverse is not required (hash collisions are allowed). Breaking this contract causes HashMap/HashSet to malfunction because they use hashCode for bucket placement and equals for key matching.

```java
// ❌ BROKEN — equals without hashCode
class BrokenKey {
    int id;
    BrokenKey(int id) { this.id = id; }

    @Override
    public boolean equals(Object o) {
        return o instanceof BrokenKey bk && bk.id == this.id;
    }
    // Missing hashCode! Uses Object.hashCode() — memory-based
}

Map<BrokenKey, String> map = new HashMap<>();
map.put(new BrokenKey(1), "one");
System.out.println(map.get(new BrokenKey(1))); // null! Different hashCodes

// ✅ CORRECT — with Records (auto-generated equals + hashCode)
record Key(int id) {}
Map<Key, String> map2 = new HashMap<>();
map2.put(new Key(1), "one");
System.out.println(map2.get(new Key(1))); // "one" ✅

// ✅ CORRECT — manual implementation
class ProperKey {
    int id;
    String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof ProperKey pk)) return false;
        return id == pk.id && Objects.equals(name, pk.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name); // uses same fields as equals
    }
}
```

**The Contract:**

| Rule | Description |
|------|-------------|
| Consistent | Multiple calls to `hashCode()` must return same value (within JVM session) |
| Equal ⇒ Same hash | `a.equals(b)` → `a.hashCode() == b.hashCode()` |
| Same hash ⇏ Equal | Hash collision is allowed |
| Reflexive | `a.equals(a)` is `true` |
| Symmetric | `a.equals(b)` ⟺ `b.equals(a)` |
| Transitive | `a.equals(b)` ∧ `b.equals(c)` → `a.equals(c)` |

**Edge Cases:**
- Mutable fields in `hashCode()`: if a key's hash changes after insertion into a `HashMap`, the entry becomes unreachable (memory leak!).
- `instanceof` check in `equals()` handles `null` automatically (returns `false`).
- Use `Objects.hash()` for concise multi-field hash computation; use `Objects.equals()` for null-safe comparison.

**Interview Tip:** Use pattern matching `instanceof` (Java 16+) in `equals()` for cleaner code. Mention that Records auto-generate both methods correctly — this is the modern best practice.

**Follow-ups:** How does HashMap handle hash collisions? What makes a good hash function? Can hashCode return a constant (like `return 1`)? (Legal, but terrible — O(n) lookups.)

---

### Q13. What are Generics? Explain type erasure, bounded types, and wildcards.

**Concept:** Generics provide compile-time type safety without runtime overhead. Java uses type erasure, meaning generic type info is removed at compile time and replaced with `Object` (or the bound). Wildcards (`?`, `? extends T`, `? super T`) add flexibility for method parameters.

```java
// Basic generic class
public class Box<T> {
    private T item;
    public void set(T item) { this.item = item; }
    public T get() { return item; }
}

// Bounded type — T must be Comparable
public <T extends Comparable<T>> T findMax(List<T> list) {
    return list.stream().max(Comparator.naturalOrder()).orElseThrow();
}

// Wildcards — PECS: Producer Extends, Consumer Super
public void printAll(List<? extends Number> list) {   // read (produce) Numbers
    for (Number n : list) System.out.println(n);
    // list.add(1); // ❌ compile error — can't add to ? extends
}

public void addIntegers(List<? super Integer> list) {  // write (consume) Integers
    list.add(1);     // ✅ OK
    list.add(2);
    // Integer x = list.get(0); // ❌ returns Object, not Integer
}

// Type erasure in action
Box<String> strBox = new Box<>();
Box<Integer> intBox = new Box<>();
System.out.println(strBox.getClass() == intBox.getClass()); // true! Same class after erasure
```

**PECS Rule (Producer Extends, Consumer Super):**

| Wildcard | Read | Write | Use case |
|----------|------|-------|----------|
| `? extends T` | ✅ as T | ❌ | Reading/producing values |
| `? super T` | ❌ (only as Object) | ✅ of T and subtypes | Writing/consuming values |
| `?` (unbounded) | ✅ as Object | ❌ | Don't care about type |

**Memory Aid:** **PECS** = **P**roducer **E**xtends, **C**onsumer **S**uper. Think of it from the collection's perspective: if the collection produces values for you, use `extends`; if it consumes values from you, use `super`.

**Edge Cases:**
- Cannot create generic arrays: `new T[10]` is illegal (type erasure).
- Cannot use primitives: `List<int>` is illegal; use `List<Integer>`.
- Type erasure means no `instanceof T` at runtime (use `Class<T>` token pattern).
- Multiple bounds: `<T extends Comparable<T> & Serializable>` — class bound first, then interfaces.

**Interview Tip:** Explain type erasure with a bytecode example: `List<String>` becomes `List<Object>` in bytecode. Mention that this is why Java generics are "not reified" (unlike C# or Kotlin inline classes).

**Follow-ups:** What are type witnesses? How does the diamond operator `<>` work? What is a raw type and why is it dangerous?

---

### Q14. Explain the `Comparable` vs `Comparator` interfaces with lambda examples.

**Concept:** `Comparable<T>` defines a natural ordering within the class itself (`compareTo`). `Comparator<T>` defines external, reusable ordering strategies (`compare`). Since Java 8, `Comparator` is commonly used with lambdas and method references.

```java
// Comparable — natural ordering (single strategy, inside the class)
public record Employee(String name, int salary) implements Comparable<Employee> {
    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary); // natural order: by salary
    }
}

// Comparator — external, flexible strategies
List<Employee> team = List.of(
    new Employee("Alice", 90000),
    new Employee("Bob", 75000),
    new Employee("Charlie", 90000)
);

// Lambda comparators
Comparator<Employee> byName     = Comparator.comparing(Employee::name);
Comparator<Employee> bySalDesc  = Comparator.comparingInt(Employee::salary).reversed();
Comparator<Employee> bySalName  = Comparator.comparingInt(Employee::salary)
                                            .thenComparing(Employee::name); // chained

List<Employee> sorted = team.stream().sorted(bySalName).toList();
sorted.forEach(System.out::println);
// Employee[name=Bob, salary=75000]
// Employee[name=Alice, salary=90000]
// Employee[name=Charlie, salary=90000]

// Null-safe comparator
Comparator<Employee> nullSafe = Comparator.nullsLast(byName);
```

| Feature | `Comparable<T>` | `Comparator<T>` |
|---------|-----------------|-----------------|
| Method | `compareTo(T o)` | `compare(T o1, T o2)` |
| Where defined | Inside the class | External (separate class/lambda) |
| Strategies | Single (natural order) | Multiple |
| Modifies class | Yes (implements) | No |
| Functional interface | No (not typically used as lambda) | Yes (`@FunctionalInterface`) |
| Null handling | Manual | `nullsFirst()` / `nullsLast()` |

**Edge Cases:**
- `compareTo` must be consistent with `equals` for correct behavior in `TreeSet`/`TreeMap`.
- `Integer.compare(a, b)` is preferred over `a - b` to avoid integer overflow.
- `Comparator.comparing()` with key extractor can throw NPE if the key is null — use `nullsFirst`/`nullsLast` wrapper.

**Interview Tip:** Show fluency with `Comparator.comparing()`, `thenComparing()`, `reversed()`, and `nullsFirst()`/`nullsLast()`. These are expected in 2026 interviews. Mention that Records make natural ordering trivial to implement.

**Follow-ups:** What happens if `compareTo` is inconsistent with `equals` in a TreeSet? How does `thenComparing` work internally?

---

### Q15. What are Java Records (Java 16+) and Sealed Classes (Java 17)?

**Concept:** Records are immutable data carriers with auto-generated `equals()`, `hashCode()`, `toString()`, and accessors. Sealed classes restrict which classes can extend them, enabling exhaustive pattern matching. Together, they form Java's answer to algebraic data types.

```java
// Record — transparent data carrier
public record Point(int x, int y) {
    // Compact constructor for validation
    public Point {
        if (x < 0 || y < 0) throw new IllegalArgumentException("Negative coordinates");
    }

    // Custom method
    public double distanceTo(Point other) {
        return Math.sqrt(Math.pow(x - other.x, 2) + Math.pow(y - other.y, 2));
    }
}

Point p1 = new Point(3, 4);
System.out.println(p1);           // Point[x=3, y=4]   (auto toString)
System.out.println(p1.x());       // 3                  (accessor, no "get" prefix)
System.out.println(p1.equals(new Point(3, 4))); // true  (auto equals)

// Sealed class — restricted hierarchy (Java 17)
public sealed interface Shape
    permits Circle, Rectangle, Triangle {}

public record Circle(double radius)              implements Shape {}
public record Rectangle(double width, double h)  implements Shape {}
public final class Triangle                      implements Shape {
    double base, height;
    Triangle(double b, double h) { base = b; height = h; }
}

// Exhaustive pattern matching (Java 21 — pattern switch)
public static double area(Shape shape) {
    return switch (shape) {
        case Circle c    -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.h();
        case Triangle t  -> 0.5 * t.base * t.height;
        // No default needed! Sealed = exhaustive
    };
}
```

**Records — What You Get (and Don't):**

| Auto-generated | Restrictions |
|---------------|-------------|
| `equals()`, `hashCode()` | Cannot extend another class (implicitly extends `Record`) |
| `toString()` | All fields are `final` |
| Component accessors (`x()`, `y()`) | Cannot declare additional instance fields |
| Canonical constructor | Can be generic, implement interfaces |

**Sealed Classes — Permitted Subclass Rules:**

| Modifier on subclass | Meaning |
|---------------------|---------|
| `final` | No further subclassing |
| `sealed` | Continues the restriction chain |
| `non-sealed` | Opens the hierarchy |

**Edge Cases:**
- Records can implement interfaces but cannot extend classes.
- Records can have static fields and methods, just not instance fields beyond components.
- Sealed classes and their permitted subclasses must be in the same module (or same compilation unit if unnamed module).
- `non-sealed` is the escape hatch — any class can extend a `non-sealed` subclass.

**Interview Tip:** Show Records + Sealed + Pattern Matching together as a trio. This is the pattern-oriented programming style Java 17/21 enables and is a hot topic in 2026 interviews.

**Follow-ups:** Can records be serialized? How do record patterns (Java 21) work for destructuring? What is the difference between records and Lombok `@Value`?

---

### Q16. How does `instanceof` work, including pattern matching (Java 16+)?

**Concept:** `instanceof` checks if an object is an instance of a class/interface at runtime. Java 16 added pattern matching which combines the type check and cast into a single expression, eliminating redundant explicit casts.

```java
// Traditional instanceof (pre-Java 16)
Object obj = "Hello World";
if (obj instanceof String) {
    String s = (String) obj;       // redundant cast
    System.out.println(s.length());
}

// Pattern matching instanceof (Java 16+)
if (obj instanceof String s) {     // check + cast in one
    System.out.println(s.length()); // s is in scope and already cast
}

// Negation pattern — s scoped after the if block
if (!(obj instanceof String s)) {
    return;    // early exit
}
System.out.println(s.length());    // s is in scope here!

// Java 21 — Record patterns (destructuring)
record Point(int x, int y) {}
Object obj2 = new Point(3, 4);

if (obj2 instanceof Point(int x, int y)) {  // destructure!
    System.out.println("x=" + x + ", y=" + y); // x=3, y=4
}

// Java 21 — Nested record patterns
record Line(Point start, Point end) {}
Object obj3 = new Line(new Point(0, 0), new Point(5, 5));

if (obj3 instanceof Line(Point(int x1, int y1), Point(int x2, int y2))) {
    System.out.println("Length: " + Math.sqrt((x2-x1)*(x2-x1) + (y2-y1)*(y2-y1)));
}
```

**Pattern Matching Evolution:**

| Java Version | Feature |
|-------------|---------|
| Java 16 | `instanceof` pattern matching |
| Java 17 (preview) | Pattern matching for `switch` |
| Java 21 (final) | Record patterns + pattern switch finalized |
| Java 21 | Guarded patterns (`case T t when condition`) |

**Edge Cases:**
- `null instanceof X` is always `false` — no NPE.
- Pattern variable scope follows flow analysis: it's in scope where the compiler can prove the match succeeded.
- Cannot use pattern matching in `switch` with `null` unless explicitly listed: `case null -> ...`

**Interview Tip:** Show the evolution from Java 16 → 21. Demonstrate nested record patterns — this is the cutting-edge feature interviewers ask about.

**Follow-ups:** How does guarded pattern matching work (`when` clause)? Can you pattern match on sealed classes without a default?

---

### Q17. What is the `var` keyword (Java 10+)? What are its limitations?

**Concept:** `var` enables local variable type inference — the compiler infers the type from the initializer. It reduces boilerplate without sacrificing type safety. It's purely a compiler feature; bytecode is identical to explicit typing.

```java
// Basic usage
var list = new ArrayList<String>();    // inferred as ArrayList<String>
var map = Map.of("a", 1, "b", 2);    // inferred as Map<String, Integer>
var stream = list.stream();            // inferred as Stream<String>

// Enhanced for and try-with-resources
for (var entry : map.entrySet()) {     // Map.Entry<String, Integer>
    System.out.println(entry.getKey() + "=" + entry.getValue());
}

try (var reader = new BufferedReader(new FileReader("file.txt"))) {
    var line = reader.readLine();      // String
}

// Lambda parameter types (Java 11+)
BiFunction<String, String, String> concat = (var a, var b) -> a + b;
// Useful for adding annotations: (@NonNull var a, @NonNull var b) -> a + b
```

**Where `var` CAN and CANNOT be used:**

| ✅ Can use | ❌ Cannot use |
|-----------|-------------|
| Local variables with initializer | Fields (instance/static) |
| For-each loop variable | Method parameters |
| Try-with-resources variable | Method return types |
| Lambda parameters (Java 11) | Catch parameters |
| | Initializer is `null` (`var x = null;` ❌) |
| | Array initializer (`var arr = {1,2,3};` ❌) |

**Edge Cases:**
- `var` is not a keyword — it's a reserved type name. You can still have a variable named `var` (but shouldn't).
- `var` with diamond operator: `var list = new ArrayList<>()` infers `ArrayList<Object>`, not what you want. Use `var list = new ArrayList<String>()`.
- `var` doesn't work with method references or lambdas without explicit type context.

**Interview Tip:** Stress that `var` is about readability, not laziness. Use it when the type is obvious from the right-hand side (e.g., constructor call). Avoid when it obscures meaning (e.g., `var result = someMethod()`).

**Follow-ups:** Does `var` affect runtime performance? Can `var` infer intersection types? What about `var` in lambda parameters?

---

# Section 2 — Collections Framework (Q18–Q31)

---

### Q18. Explain the Java Collections Framework hierarchy.

**Concept:** The Collections Framework is a unified architecture for representing and manipulating collections. The root interfaces are `Iterable` → `Collection` (with sub-interfaces `List`, `Set`, `Queue`) and separately `Map`. Java 21 introduced `SequencedCollection`, `SequencedSet`, and `SequencedMap` to add encounter-order operations.

```
                        Iterable<T>
                            │
                      Collection<T>  ←── SequencedCollection<T> (Java 21)
                     ╱      │      ╲
                List<T>   Set<T>   Queue<T>
                  │         │         │
            ArrayList   HashSet   PriorityQueue
            LinkedList  TreeSet   ArrayDeque
            Vector      LinkedHashSet
                          │
                     SortedSet<T>
                     NavigableSet<T>

                       Map<K,V>  ←── SequencedMap<K,V> (Java 21)
                     ╱    │    ╲
               HashMap  TreeMap  LinkedHashMap
               Hashtable  ConcurrentHashMap
                    │
              SortedMap<K,V>
              NavigableMap<K,V>
```

**Java 21 Sequenced Collections:**

```java
// New methods on SequencedCollection (Java 21)
SequencedCollection<String> seq = new LinkedHashSet<>(List.of("a", "b", "c"));
seq.getFirst();        // "a"
seq.getLast();         // "c"
seq.addFirst("z");     // adds at beginning
seq.reversed();        // reversed view

SequencedMap<String, Integer> smap = new LinkedHashMap<>();
smap.put("one", 1);
smap.put("two", 2);
smap.firstEntry();     // one=1
smap.lastEntry();      // two=2
smap.pollLastEntry();  // removes and returns two=2
```

| Interface (Java 21) | Supertype of | Key new methods |
|---------------------|-------------|-----------------|
| `SequencedCollection` | `List`, `LinkedHashSet`, `SortedSet` | `getFirst()`, `getLast()`, `reversed()` |
| `SequencedSet` | `LinkedHashSet`, `SortedSet` | Above + `addFirst()`, `addLast()` |
| `SequencedMap` | `LinkedHashMap`, `SortedMap` | `firstEntry()`, `lastEntry()`, `pollFirstEntry()` |

**Interview Tip:** Knowing the Java 21 Sequenced Collections retrofit is a differentiator. Mention that `ArrayList`, `LinkedList`, and `LinkedHashSet` all now implement `SequencedCollection`.

**Follow-ups:** Why was there no `getFirst()`/`getLast()` before Java 21? How does `reversed()` work — does it copy?

---

### Q19. Compare ArrayList vs LinkedList vs Vector.

**Concept:** `ArrayList` is a resizable array (O(1) random access, O(n) insert/delete in middle). `LinkedList` is a doubly-linked list (O(1) insert/delete at ends, O(n) random access). `Vector` is a synchronized `ArrayList` (legacy, avoid in new code).

```java
// ArrayList — default choice for most use cases
List<String> arrayList = new ArrayList<>();
arrayList.add("A");           // O(1) amortized
arrayList.get(0);             // O(1)
arrayList.add(0, "B");        // O(n) — shifts elements
arrayList.remove(0);          // O(n)

// LinkedList — rarely the right choice in practice
LinkedList<String> linked = new LinkedList<>();
linked.addFirst("A");         // O(1)
linked.addLast("B");          // O(1)
linked.get(5);                // O(n) — traversal needed
// LinkedList implements both List and Deque

// Correct sizing to avoid resizing
List<String> sized = new ArrayList<>(1000); // initial capacity
```

| Operation | ArrayList | LinkedList | Vector |
|-----------|-----------|------------|--------|
| `get(i)` | **O(1)** | O(n) | O(1) |
| `add(e)` (end) | **O(1)** amortized | **O(1)** | O(1) |
| `add(i, e)` (middle) | O(n) | O(n)* | O(n) |
| `remove(i)` | O(n) | O(n)* | O(n) |
| Memory | Compact (array) | High (node + 2 pointers per element) | Compact |
| Thread-safe | ❌ | ❌ | ✅ (synchronized) |
| Growth | 50% (`newCap = oldCap + oldCap >> 1`) | N/A | 100% |
| Cache-friendly | ✅ (contiguous memory) | ❌ (scattered nodes) | ✅ |

*LinkedList `add`/`remove` at a known node is O(1), but finding the node is O(n).

**Edge Cases:**
- LinkedList uses more memory per element (~3x) due to node overhead.
- `ArrayList` is almost always faster in practice due to CPU cache locality.
- Use `ArrayDeque` instead of `LinkedList` for queue/deque operations — it's faster.
- For thread-safety, use `CopyOnWriteArrayList` or `Collections.synchronizedList()`, not `Vector`.

**Interview Tip:** In 2026, the correct answer is "use `ArrayList` by default." The only good use case for `LinkedList` is a `Deque`. Mention that `ArrayList` resizes by 50% (not doubling) since JDK source.

**Follow-ups:** What happens when `ArrayList` exceeds capacity? Why is `ArrayDeque` faster than `LinkedList`? What is `CopyOnWriteArrayList`?

---

### Q20. How does HashMap work internally? Explain the Java 8+ tree-ification.

**Concept:** HashMap stores key-value pairs in an array of buckets (initially 16, load factor 0.75). Each bucket holds a linked list of entries. Since Java 8, when a bucket exceeds 8 entries (and capacity ≥ 64), the linked list is converted to a red-black tree for O(log n) worst-case lookup instead of O(n).

```java
// Internal structure (simplified)
// Node<K,V>[] table;  — the bucket array
// Each bucket: LinkedList of Node(hash, key, value, next)
//   OR TreeNode (red-black tree) if threshold exceeded

Map<String, Integer> map = new HashMap<>(16, 0.75f); // capacity, loadFactor
map.put("Alice", 90);
// Step 1: hash = hash("Alice")  → spread high bits: h ^ (h >>> 16)
// Step 2: index = hash & (capacity - 1)  — bitwise AND for bucket
// Step 3: Insert at bucket[index]

// When size > capacity * loadFactor → resize (double capacity, rehash all)

// Tree-ification thresholds
// TREEIFY_THRESHOLD = 8   (list → tree when bucket has 8+ nodes AND table.length ≥ 64)
// UNTREEIFY_THRESHOLD = 6 (tree → list when bucket shrinks below 6)
// MIN_TREEIFY_CAPACITY = 64 (won't treeify if table is small, resizes instead)
```

**HashMap Internals Flow:**

```
put(key, value)
  ├─ hash = key.hashCode() ^ (key.hashCode() >>> 16)  // perturbation
  ├─ bucket = hash & (table.length - 1)
  ├─ if bucket empty → insert new Node
  ├─ if key exists (equals check) → replace value
  ├─ if bucket is LinkedList and size ≥ 8 → treeify to Red-Black Tree
  └─ if size > threshold → resize (2x capacity, rehash)

get(key)
  ├─ hash → bucket index
  ├─ if bucket is LinkedList → traverse, compare with equals(): O(n)
  └─ if bucket is Red-Black Tree → tree search: O(log n)
```

| Parameter | Value | Purpose |
|-----------|-------|---------|
| Default capacity | 16 | Initial bucket array size |
| Load factor | 0.75 | Resize when 75% full |
| Treeify threshold | 8 | Convert list → tree at 8 nodes |
| Untreeify threshold | 6 | Convert tree → list at 6 nodes |
| Min treeify capacity | 64 | Won't treeify if table < 64 |

**Complexity:**

| Operation | Best/Average | Worst (pre-J8) | Worst (J8+) |
|-----------|:------------:|:--------------:|:-----------:|
| `put` | O(1) | O(n) | O(log n) |
| `get` | O(1) | O(n) | O(log n) |
| `remove` | O(1) | O(n) | O(log n) |

**Edge Cases:**
- Keys must have immutable `hashCode`/`equals` — mutable keys lead to lost entries.
- `HashMap` allows one `null` key and multiple `null` values.
- Tree-ification requires keys to implement `Comparable`; if not, it falls back to `identityHashCode` ordering.
- Initial capacity should be a power of 2 (HashMap rounds up automatically).

**Interview Tip:** Draw the internal structure on a whiteboard. Explain the hash perturbation function (`h ^ (h >>> 16)`) that mixes high bits into low bits to reduce collisions. This shows deep understanding.

**Follow-ups:** Why is load factor 0.75? How does resize work? What is the difference between Java 7 (head insertion, infinite loop risk) and Java 8 (tail insertion) during resize?

---

### Q21. Compare HashMap, LinkedHashMap, TreeMap, and ConcurrentHashMap.

**Concept:** These are the four main Map implementations with different ordering guarantees and thread-safety. HashMap is unordered; LinkedHashMap maintains insertion (or access) order; TreeMap maintains sorted key order; ConcurrentHashMap is thread-safe without locking the entire map.

```java
// HashMap — no order guarantee
Map<String, Integer> hashMap = new HashMap<>();

// LinkedHashMap — insertion order preserved
Map<String, Integer> linked = new LinkedHashMap<>();
linked.put("B", 2); linked.put("A", 1); linked.put("C", 3);
System.out.println(linked.keySet()); // [B, A, C] — insertion order

// LinkedHashMap — access order (LRU cache)
Map<String, Integer> lru = new LinkedHashMap<>(16, 0.75f, true); // accessOrder=true
lru.put("A", 1); lru.put("B", 2); lru.put("C", 3);
lru.get("A"); // access A
System.out.println(lru.keySet()); // [B, C, A] — A moved to end (most recent)

// TreeMap — natural sorted order
Map<String, Integer> tree = new TreeMap<>();
tree.put("banana", 2); tree.put("apple", 1); tree.put("cherry", 3);
System.out.println(tree.keySet()); // [apple, banana, cherry] — sorted

// ConcurrentHashMap — thread-safe
ConcurrentHashMap<String, Integer> conc = new ConcurrentHashMap<>();
conc.put("key", 1);
conc.compute("key", (k, v) -> v + 1);  // atomic operation
```

| Feature | HashMap | LinkedHashMap | TreeMap | ConcurrentHashMap |
|---------|---------|---------------|---------|-------------------|
| Order | None | Insertion/Access | Sorted (key) | None |
| Null key | ✅ (one) | ✅ (one) | ❌ | ❌ |
| Null values | ✅ | ✅ | ✅ | ❌ |
| Thread-safe | ❌ | ❌ | ❌ | ✅ |
| Get/Put | O(1) | O(1) | O(log n) | O(1) avg |
| Implements | `Map` | `Map`, `SequencedMap` | `NavigableMap`, `SortedMap` | `ConcurrentMap` |

**Edge Cases:**
- `TreeMap` requires keys to be `Comparable` or a `Comparator` in the constructor.
- `ConcurrentHashMap` does NOT lock the entire map — it uses bucket-level (segment) locking (Java 8+: CAS + synchronized on bins).
- `LinkedHashMap` can be used as an LRU cache by overriding `removeEldestEntry()`.

**Interview Tip:** Be ready to implement a simple LRU cache using `LinkedHashMap` — this is a classic interview question.

**Follow-ups:** How does ConcurrentHashMap achieve concurrency internally? How would you build an LRU cache with LinkedHashMap?

---

### Q22. What is the difference between `HashSet`, `LinkedHashSet`, `TreeSet`?

**Concept:** All three implement `Set` (unique elements). Internally, each wraps its corresponding `Map`: `HashSet` uses `HashMap`, `LinkedHashSet` uses `LinkedHashMap`, `TreeSet` uses `TreeMap`. The differences mirror their underlying maps.

```java
Set<String> hash   = new HashSet<>(List.of("C", "A", "B"));
Set<String> linked = new LinkedHashSet<>(List.of("C", "A", "B"));
Set<String> tree   = new TreeSet<>(List.of("C", "A", "B"));

System.out.println(hash);   // [A, B, C] or any order (unordered)
System.out.println(linked); // [C, A, B]              (insertion order)
System.out.println(tree);   // [A, B, C]              (sorted order)
```

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|---------------|---------|
| Order | None | Insertion | Sorted |
| Backed by | HashMap | LinkedHashMap | TreeMap |
| Add/Remove/Contains | O(1) | O(1) | O(log n) |
| Null | ✅ (one) | ✅ (one) | ❌ |
| `SequencedSet` (Java 21) | ❌ | ✅ | ✅ (`SortedSet`) |

**Interview Tip:** When asked "which Set to use," say: HashSet by default; LinkedHashSet when iteration order matters; TreeSet when you need sorted iteration or range queries (`headSet`, `tailSet`, `subSet`).

**Follow-ups:** How does HashSet ensure uniqueness internally? What is `EnumSet` and why is it faster?

---

### Q23. Explain `ConcurrentHashMap` internals (Java 8+).

**Concept:** `ConcurrentHashMap` provides thread-safe operations without locking the entire map. Since Java 8, it uses an array of nodes with CAS (Compare-And-Swap) for insertions and `synchronized` on individual bins for modifications. It supports concurrent reads without locking and uses the same treeify mechanism as HashMap.

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Atomic compound operations
map.putIfAbsent("key", 1);                         // only if absent
map.computeIfAbsent("count", k -> 0);              // lazy init
map.merge("key", 1, Integer::sum);                  // atomic merge
map.compute("key", (k, v) -> v == null ? 1 : v + 1); // atomic compute

// Parallel bulk operations (Java 8+)
map.forEach(2, (k, v) -> System.out.println(k + "=" + v)); // parallelism threshold = 2
long sum = map.reduceValuesToLong(1, Long::sum, 0L);

// ❌ WRONG — not atomic!
if (!map.containsKey("key")) {
    map.put("key", value);  // race condition between check and put!
}
// ✅ CORRECT
map.putIfAbsent("key", value);
```

**ConcurrentHashMap vs HashMap vs Hashtable:**

| Feature | HashMap | Hashtable | ConcurrentHashMap |
|---------|---------|-----------|-------------------|
| Thread-safe | ❌ | ✅ (entire map locked) | ✅ (per-bin locking) |
| Null key/value | ✅/✅ | ❌/❌ | ❌/❌ |
| Performance (concurrent) | N/A (unsafe) | Poor (global lock) | Excellent |
| Iterator | Fail-fast | Fail-fast | Weakly consistent |
| Atomic operations | ❌ | ❌ | `compute`, `merge`, `putIfAbsent` |
| Since | JDK 1.2 | JDK 1.0 | JDK 1.5 (rewritten in JDK 8) |

**Edge Cases:**
- `ConcurrentHashMap.size()` may be inaccurate during concurrent modifications — use `mappingCount()` for long-valued size.
- `keySet()`, `values()`, `entrySet()` return weakly consistent views — they reflect some but not necessarily all concurrent modifications.
- No `null` keys or values — `null` is used internally as a sentinel.

**Interview Tip:** Explain the Java 8 CAS + synchronized approach vs the pre-8 `Segment`-based locking. Mention atomic compound operations (`compute`, `merge`) as the proper way to do check-then-act.

**Follow-ups:** What is the difference between fail-fast and weakly consistent iterators? How does CAS work at the CPU level?

---

### Q24. What are fail-fast vs fail-safe iterators?

**Concept:** Fail-fast iterators throw `ConcurrentModificationException` if the collection is structurally modified during iteration (e.g., `ArrayList`, `HashMap`). Fail-safe (weakly consistent) iterators work on a copy or snapshot of the data, so they never throw CME but may not reflect latest changes (e.g., `ConcurrentHashMap`, `CopyOnWriteArrayList`).

```java
// Fail-fast — throws ConcurrentModificationException
List<String> list = new ArrayList<>(List.of("A", "B", "C"));
try {
    for (String s : list) {
        if (s.equals("B")) list.remove(s); // ❌ CME!
    }
} catch (ConcurrentModificationException e) {
    System.out.println("Fail-fast triggered!");
}

// ✅ CORRECT — use Iterator.remove()
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("B")) it.remove(); // safe
}

// ✅ CORRECT (Java 8+) — removeIf
list.removeIf(s -> s.equals("B"));

// Fail-safe — no exception
ConcurrentHashMap<String, Integer> cmap = new ConcurrentHashMap<>();
cmap.put("A", 1); cmap.put("B", 2);
for (var entry : cmap.entrySet()) {
    cmap.put("C", 3); // no exception — weakly consistent
}
```

| Feature | Fail-Fast | Fail-Safe (Weakly Consistent) |
|---------|-----------|-------------------------------|
| Exception on modification | `ConcurrentModificationException` | None |
| Works on | Original collection | Copy/snapshot or concurrent structure |
| Memory overhead | None | Copy overhead (COW) or none (CHM) |
| Reflects latest state | N/A (fails) | May not |
| Examples | `ArrayList`, `HashMap`, `HashSet` | `ConcurrentHashMap`, `CopyOnWriteArrayList` |

**Edge Cases:**
- Fail-fast is best-effort — detection uses an internal `modCount` field, not synchronization. It's possible (though rare) to miss a concurrent modification.
- `CopyOnWriteArrayList` creates a new array on every write — good for read-heavy, write-rare scenarios.
- `removeIf()` is the cleanest way to remove during iteration on any `Collection`.

**Interview Tip:** Know three ways to safely remove during iteration: `Iterator.remove()`, `removeIf()`, and using a concurrent collection. Explain the `modCount` mechanism.

**Follow-ups:** How does `modCount` work internally? When would you choose `CopyOnWriteArrayList`?

---

### Q25. Explain `PriorityQueue` and `ArrayDeque`.

**Concept:** `PriorityQueue` is a min-heap-based queue that orders elements by natural ordering or a `Comparator` — `poll()` always returns the smallest element. `ArrayDeque` is a resizable circular array implementing `Deque` — faster than `LinkedList` for both stack and queue operations.

```java
// PriorityQueue — min-heap by default
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.addAll(List.of(30, 10, 20));
System.out.println(minHeap.poll()); // 10 (smallest)
System.out.println(minHeap.poll()); // 20

// Max-heap using reversed comparator
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.addAll(List.of(30, 10, 20));
System.out.println(maxHeap.poll()); // 30 (largest)

// Custom object priority
record Task(String name, int priority) {}
PriorityQueue<Task> tasks = new PriorityQueue<>(Comparator.comparingInt(Task::priority));
tasks.add(new Task("Low", 3));
tasks.add(new Task("Critical", 1));
System.out.println(tasks.poll().name()); // Critical

// ArrayDeque — stack and queue
Deque<String> stack = new ArrayDeque<>();
stack.push("A"); stack.push("B");
System.out.println(stack.pop());  // B (LIFO)

Deque<String> queue = new ArrayDeque<>();
queue.offer("A"); queue.offer("B");
System.out.println(queue.poll()); // A (FIFO)
```

| Feature | PriorityQueue | ArrayDeque |
|---------|--------------|------------|
| Ordering | Priority (heap) | FIFO/LIFO (insertion) |
| `peek`/`poll` | O(1) / O(log n) | O(1) / O(1) |
| `add`/`offer` | O(log n) | O(1) amortized |
| Null elements | ❌ | ❌ |
| Thread-safe | ❌ | ❌ |
| Use as | Priority queue | Stack or Queue |

**Interview Tip:** Say "use `ArrayDeque` instead of `Stack` (legacy) or `LinkedList` (for queue)." `ArrayDeque` is the modern recommendation for both stack and queue use cases.

**Follow-ups:** How is the heap maintained during insert/remove? What is the time complexity of `PriorityQueue.remove(Object)`?

---

### Q26. How do `Collections.unmodifiableList()` vs `List.of()` vs `List.copyOf()` differ?

**Concept:** All three produce unmodifiable lists but differ in backing, null handling, and identity. `unmodifiableList` wraps the original (changes to original reflect through). `List.of()` creates a structurally immutable list (no `null`, not backed by anything). `List.copyOf()` copies then wraps, disconnecting from the source.

```java
// Collections.unmodifiableList — wraps original (view)
List<String> original = new ArrayList<>(List.of("A", "B"));
List<String> unmod = Collections.unmodifiableList(original);
original.add("C");                  // original modified
System.out.println(unmod);          // [A, B, C] — reflects change!
// unmod.add("D");                  // ❌ UnsupportedOperationException

// List.of — truly immutable, no nulls
List<String> immutable = List.of("A", "B", "C");
// immutable.add("D");             // ❌ UnsupportedOperationException
// List.of("A", null);             // ❌ NullPointerException

// List.copyOf — snapshot copy, no nulls
List<String> copy = List.copyOf(original);
original.add("D");
System.out.println(copy);           // [A, B, C] — NOT affected
```

| Feature | `unmodifiableList` | `List.of()` | `List.copyOf()` |
|---------|--------------------|-------------|-----------------|
| Backed by original | ✅ | ❌ | ❌ |
| Allows `null` | ✅ | ❌ | ❌ |
| Truly immutable | ❌ (original can change) | ✅ | ✅ |
| Serializable | ✅ | ✅ | ✅ |
| Since | JDK 1.2 | JDK 9 | JDK 10 |

**Interview Tip:** For new code, prefer `List.of()` for literal lists and `List.copyOf()` for defensive copies. Mention that `List.copyOf()` returns the same instance if the source is already an unmodifiable list (optimization).

**Follow-ups:** Can you sort an unmodifiable list? What is `Collections.singletonList()` vs `List.of(x)`?

---

### Q27. What is the difference between `Iterator` and `ListIterator`?

**Concept:** `Iterator` provides one-directional (forward) traversal with `remove()`. `ListIterator` extends it with bidirectional traversal (`previous()`), element replacement (`set()`), insertion (`add()`), and index access. Only available for `List` implementations.

```java
List<String> list = new ArrayList<>(List.of("A", "B", "C", "D"));

// Iterator — forward only
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    if (s.equals("B")) it.remove();
}

// ListIterator — bidirectional, can modify
ListIterator<String> lit = list.listIterator(list.size()); // start at end
while (lit.hasPrevious()) {
    String s = lit.previous();
    System.out.print(s + " "); // D C A (reverse traversal)
}

lit = list.listIterator();
while (lit.hasNext()) {
    String s = lit.next();
    if (s.equals("C")) lit.set("C_MODIFIED");   // replace
    if (s.equals("A")) lit.add("A2");            // insert after current
}
System.out.println(list); // [A, A2, C_MODIFIED, D]
```

| Feature | Iterator | ListIterator |
|---------|----------|-------------|
| Direction | Forward only | Bidirectional |
| `remove()` | ✅ | ✅ |
| `set()` | ❌ | ✅ |
| `add()` | ❌ | ✅ |
| `nextIndex()` / `previousIndex()` | ❌ | ✅ |
| Available for | Any `Collection` | `List` only |

**Interview Tip:** Know when to use `ListIterator` vs streams. For in-place modification during iteration, `ListIterator` is the right tool. For filtering/transforming into new collections, use streams.

---

### Q28. What is `EnumSet` and `EnumMap`? Why are they optimized?

**Concept:** `EnumSet` and `EnumMap` are specialized implementations for enum keys. `EnumSet` uses a bitmask (one bit per enum constant) making operations extremely fast (O(1) with bitwise operations). `EnumMap` uses a simple array indexed by enum ordinal, with no hashing needed.

```java
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

// EnumSet — internally a long bitmask (up to 64 enum constants)
EnumSet<Day> weekdays = EnumSet.range(Day.MON, Day.FRI);  // MON-FRI
EnumSet<Day> weekend  = EnumSet.complementOf(weekdays);    // SAT, SUN
EnumSet<Day> all      = EnumSet.allOf(Day.class);

// EnumMap — internally an Object[] indexed by ordinal
EnumMap<Day, String> schedule = new EnumMap<>(Day.class);
schedule.put(Day.MON, "Standup");
schedule.put(Day.FRI, "Retro");
```

| Feature | EnumSet | EnumMap |
|---------|---------|---------|
| Internal structure | Bit vector (`long`) | Array indexed by ordinal |
| Performance | Bitwise ops (extremely fast) | Direct array access (O(1)) |
| Memory | 1 bit per constant | 1 reference per constant |
| Null key | ❌ | ❌ |
| Iteration order | Enum declaration order | Enum declaration order |

**Interview Tip:** Always mention `EnumSet`/`EnumMap` when discussing enum best practices — they're vastly more efficient than `HashSet`/`HashMap` for enums.

---

### Q29. How does `TreeMap` work? What is a Red-Black Tree?

**Concept:** `TreeMap` is a `NavigableMap` backed by a Red-Black Tree (self-balancing BST). It maintains keys in sorted order, guaranteeing O(log n) for get/put/remove. A Red-Black Tree ensures balance through coloring rules: every node is red or black, root is black, red nodes have only black children, and all paths from root to null have the same black-node count.

```java
TreeMap<String, Integer> map = new TreeMap<>();
map.put("banana", 2);
map.put("apple", 1);
map.put("cherry", 3);

System.out.println(map.firstKey());        // "apple"
System.out.println(map.lastKey());         // "cherry"
System.out.println(map.headMap("cherry")); // {apple=1, banana=2}
System.out.println(map.subMap("apple", "cherry")); // {apple=1, banana=2}
System.out.println(map.ceilingKey("b"));   // "banana" (≥ "b")
System.out.println(map.floorKey("b"));     // "banana" (≤ "b" — no, "b" > "banana" is false. Actually: "banana" ≤ "b"? Lexicographically "b" < "banana", so floor("b") = "apple")
```

| Operation | Time Complexity |
|-----------|:--------------:|
| `get`, `put`, `remove` | O(log n) |
| `firstKey`, `lastKey` | O(log n) |
| `headMap`, `tailMap`, `subMap` | O(log n) + O(k) for iteration |

**Edge Cases:**
- Keys must be `Comparable` or provide a `Comparator` at construction.
- `null` keys are NOT allowed (throws NPE) — can't compare `null`.
- `headMap(key)` is exclusive of the given key by default; `headMap(key, true)` is inclusive.

**Interview Tip:** Explain why Red-Black Trees are preferred over AVL trees in Java: RB trees have faster insertion/deletion (fewer rotations) while AVL trees have faster lookups (stricter balancing). Java chose RB for the general-purpose map use case.

**Follow-ups:** What are the rotation operations in a Red-Black Tree? How does `NavigableMap` differ from `SortedMap`?

---

### Q30. Explain `Collections.synchronizedList()` vs `CopyOnWriteArrayList`.

**Concept:** `synchronizedList` wraps any list with synchronized methods (coarse-grained locking). `CopyOnWriteArrayList` creates a new internal array on every write operation, making reads lock-free. Choose synchronized for write-heavy, COW for read-heavy.

```java
// Synchronized list — every method is synchronized
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
synchronized (syncList) {   // MUST manually sync during iteration!
    for (String s : syncList) { /* ... */ }
}

// CopyOnWriteArrayList — snapshot iterator, no CME
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
cowList.add("A");
cowList.add("B");
for (String s : cowList) {
    cowList.add("C"); // safe — iterator sees snapshot [A, B]
}
System.out.println(cowList); // [A, B, C, C] (added twice — two iterations)
```

| Feature | `synchronizedList` | `CopyOnWriteArrayList` |
|---------|--------------------|------------------------|
| Read speed | Contended (lock) | Lock-free (fast) |
| Write speed | Fast (in-place) | Slow (copies entire array) |
| Iteration safety | Must manually synchronize | Snapshot (always safe) |
| Memory | Low | High on writes |
| Best for | Write-heavy | Read-heavy, write-rare (e.g., listeners) |

**Interview Tip:** Mention that `CopyOnWriteArrayList` is ideal for event listener lists — rarely modified, frequently iterated. Emphasize the manual synchronization requirement for `synchronizedList` iteration.

---

### Q31. What are `Map.of()`, `Map.entry()`, and `Map.ofEntries()` factory methods?

**Concept:** Java 9 introduced `Map.of()` for up to 10 key-value pairs and `Map.ofEntries()` for more. These create truly unmodifiable maps. `Map.entry()` creates an individual immutable entry for use with `ofEntries()`.

```java
// Map.of — up to 10 pairs
Map<String, Integer> small = Map.of("a", 1, "b", 2, "c", 3);

// Map.ofEntries — unlimited pairs
Map<String, Integer> large = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2),
    Map.entry("c", 3),
    Map.entry("d", 4)
    // ... unlimited
);

// Null keys or values → NullPointerException
// Duplicate keys → IllegalArgumentException
// Map.of("a", 1, "a", 2); // ❌ IllegalArgumentException

// Collectors.toUnmodifiableMap for streams
Map<String, Integer> fromStream = List.of("a", "bb", "ccc")
    .stream()
    .collect(Collectors.toUnmodifiableMap(s -> s, String::length));
```

**Interview Tip:** Know that `Map.of()` returns an iteration-order-randomized map (since Java 9) — don't depend on order. For ordered unmodifiable maps, wrap a `LinkedHashMap` with `Collections.unmodifiableMap`.

---

# Section 3 — Multithreading & Concurrency (Q32–Q48)

---

### Q32. What is the Java Thread lifecycle? Explain all states.

**Concept:** A Java thread goes through six states defined in `Thread.State`: NEW (created, not started), RUNNABLE (eligible to run or running), BLOCKED (waiting for monitor lock), WAITING (waiting indefinitely), TIMED_WAITING (waiting with timeout), and TERMINATED (completed).

```java
Thread t = new Thread(() -> {
    try {
        Thread.sleep(1000);   // TIMED_WAITING
    } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
});

System.out.println(t.getState()); // NEW
t.start();
System.out.println(t.getState()); // RUNNABLE
// During sleep → TIMED_WAITING
t.join();
System.out.println(t.getState()); // TERMINATED
```

```
NEW ──start()──→ RUNNABLE ←──────→ RUNNING
                    │   ↑               │
                    │   │ notify/       │
                    │   │ timeout       │
                    ↓   │               │
              WAITING / TIMED_WAITING   │
                    │                   │
           synchronized ──→ BLOCKED     │
                                        ↓
                                   TERMINATED
```

| State | Trigger | Exit |
|-------|---------|------|
| NEW | `new Thread()` | `start()` |
| RUNNABLE | `start()`, notified, lock acquired | Scheduler decides |
| BLOCKED | Waiting for `synchronized` lock | Lock acquired |
| WAITING | `wait()`, `join()`, `LockSupport.park()` | `notify()`, thread completes, `unpark()` |
| TIMED_WAITING | `sleep(ms)`, `wait(ms)`, `join(ms)` | Timeout or notification |
| TERMINATED | `run()` completes or exception | — |

**Interview Tip:** Distinguish BLOCKED (waiting for monitor lock) from WAITING (waiting for a condition). A thread is BLOCKED only when trying to enter a `synchronized` block held by another thread.

**Follow-ups:** What's the difference between `sleep()` and `wait()`? Can a TERMINATED thread be restarted?

---

### Q33. What are the ways to create threads in Java?

**Concept:** There are four main ways: extending `Thread`, implementing `Runnable`, implementing `Callable<V>` (returns a result), and using `ExecutorService`. In modern Java, prefer `Executors` or Virtual Threads (Java 21).

```java
// 1. Extend Thread (not recommended — wastes inheritance)
class MyThread extends Thread {
    @Override
    public void run() { System.out.println("Thread: " + getName()); }
}
new MyThread().start();

// 2. Implement Runnable (functional interface)
Runnable task = () -> System.out.println("Runnable: " + Thread.currentThread().getName());
new Thread(task).start();

// 3. Callable + Future (returns result, throws checked exception)
Callable<Integer> callable = () -> { Thread.sleep(100); return 42; };
ExecutorService exec = Executors.newSingleThreadExecutor();
Future<Integer> future = exec.submit(callable);
System.out.println(future.get()); // 42 (blocks until complete)
exec.shutdown();

// 4. Virtual Threads (Java 21) — lightweight, managed by JVM
Thread vt = Thread.ofVirtual().name("virtual-1").start(() -> {
    System.out.println("Virtual thread: " + Thread.currentThread());
});
vt.join();

// Virtual Thread factory
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> System.out.println("Task on virtual thread"));
}
```

| Method | Return value | Exception handling | Modern? |
|--------|-------------|-------------------|---------|
| `extends Thread` | None | Unchecked only | ❌ |
| `Runnable` | None | Unchecked only | ✅ |
| `Callable<V>` | `V` (via `Future`) | Checked + Unchecked | ✅ |
| Virtual Thread (J21) | Via Callable/Future | Both | ✅✅ |

**Interview Tip:** Always mention Virtual Threads as the Java 21 evolution. Say: "For I/O-bound tasks, Virtual Threads are the future. For CPU-bound tasks, platform thread pools remain relevant."

**Follow-ups:** What is the difference between `start()` and `run()`? Can a Runnable return a result? (Indirectly, via shared state or wrapping in Callable.)

---

### Q34. Explain `synchronized`, `volatile`, and Atomic variables.

**Concept:** `synchronized` provides mutual exclusion (only one thread enters the block) and visibility (changes are flushed to main memory). `volatile` ensures visibility only (no caching in CPU registers) without atomicity for compound operations. Atomic variables (`AtomicInteger`, etc.) provide lock-free atomic operations using CAS.

```java
// synchronized — mutual exclusion + visibility
class Counter {
    private int count = 0;
    public synchronized void increment() { count++; } // lock on 'this'
    public synchronized int getCount() { return count; }
}

// volatile — visibility only, no atomicity for compound ops
class Flag {
    private volatile boolean running = true;  // visible across threads
    public void stop() { running = false; }   // single write — volatile is sufficient
    public void run() {
        while (running) { /* work */ }        // reads latest value
    }
}

// AtomicInteger — lock-free atomic operations
AtomicInteger atomicCount = new AtomicInteger(0);
atomicCount.incrementAndGet();           // atomic i++
atomicCount.compareAndSet(1, 2);         // CAS: if current==1, set to 2
atomicCount.updateAndGet(x -> x * 2);   // atomic arbitrary update
atomicCount.accumulateAndGet(5, Integer::sum); // atomic accumulate

// ❌ volatile is NOT enough for count++
// count++ = read + increment + write (3 steps, not atomic)
volatile int badCount = 0;
// badCount++; // Race condition! Use AtomicInteger instead
```

| Feature | `synchronized` | `volatile` | `Atomic*` |
|---------|---------------|------------|-----------|
| Mutual exclusion | ✅ | ❌ | ❌ (lock-free) |
| Visibility | ✅ | ✅ | ✅ |
| Atomicity | ✅ (block) | ❌ (single r/w only) | ✅ (single operation) |
| Performance | Slowest (lock overhead) | Fast | Fast (CAS) |
| Use case | Complex critical sections | Simple flags/status | Counters, accumulators |

**Memory Aid:** **S-V-A** = **S**afe (synchronized, full safety), **V**isible (volatile, just visibility), **A**tomic (atomic, lock-free single ops).

**Edge Cases:**
- `volatile` arrays: only the reference is volatile, not the elements.
- `synchronized` on `null` reference → NPE.
- `AtomicReference` for atomic reference swaps (useful for lock-free data structures).
- Java 9+ added `VarHandle` for fine-grained memory ordering control.

**Interview Tip:** Explain the Java Memory Model (JMM) concept of "happens-before": synchronized release happens-before the next acquire; volatile write happens-before the next volatile read. This is what guarantees visibility.

**Follow-ups:** What is a memory barrier? How does `LongAdder` improve on `AtomicLong`? What are VarHandles?

---

### Q35. What is the Executor Framework? Explain thread pool types.

**Concept:** The Executor Framework (`java.util.concurrent`) decouples task submission from execution. `ExecutorService` manages a thread pool, reusing threads to avoid creation overhead. Java provides factory methods for common pool configurations.

```java
// Fixed thread pool — bounded, for known concurrency
ExecutorService fixed = Executors.newFixedThreadPool(4);

// Cached thread pool — unbounded, creates threads on demand
ExecutorService cached = Executors.newCachedThreadPool();

// Single thread executor — sequential execution
ExecutorService single = Executors.newSingleThreadExecutor();

// Scheduled — for delayed/periodic tasks
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);
scheduled.scheduleAtFixedRate(() -> System.out.println("tick"), 0, 1, TimeUnit.SECONDS);

// Work-stealing pool (Java 8+) — ForkJoinPool, good for recursive tasks
ExecutorService stealing = Executors.newWorkStealingPool();

// Custom ThreadPoolExecutor for fine-grained control
ThreadPoolExecutor custom = new ThreadPoolExecutor(
    2,                // core pool size
    10,               // max pool size
    60, TimeUnit.SECONDS, // keep-alive for excess threads
    new ArrayBlockingQueue<>(100), // bounded work queue
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
);

// Proper shutdown
fixed.shutdown();                       // graceful — finish queued tasks
fixed.awaitTermination(30, TimeUnit.SECONDS);
// fixed.shutdownNow();                // aggressive — interrupt running + discard queued
```

| Pool Type | Core Size | Max Size | Queue | Use Case |
|-----------|-----------|----------|-------|----------|
| `newFixedThreadPool(n)` | n | n | Unbounded `LinkedBlockingQueue` | Known parallelism |
| `newCachedThreadPool()` | 0 | Integer.MAX | `SynchronousQueue` | Short-lived tasks |
| `newSingleThreadExecutor()` | 1 | 1 | Unbounded | Sequential tasks |
| `newScheduledThreadPool(n)` | n | Integer.MAX | `DelayedWorkQueue` | Periodic tasks |
| `newWorkStealingPool()` | Runtime CPUs | — | Per-thread deque | Recursive/parallel |
| `newVirtualThreadPerTaskExecutor()` | — | — | — | I/O-bound (Java 21) |

**Rejection Policies (when queue is full):**

| Policy | Behavior |
|--------|----------|
| `AbortPolicy` (default) | Throws `RejectedExecutionException` |
| `CallerRunsPolicy` | Submitting thread runs the task |
| `DiscardPolicy` | Silently discards |
| `DiscardOldestPolicy` | Discards oldest queued task |

**Interview Tip:** Never use `Executors.newCachedThreadPool()` in production without understanding its risk — it can create unlimited threads. Always prefer `ThreadPoolExecutor` with bounded queue and explicit rejection policy.

**Follow-ups:** What is the difference between `submit()` and `execute()`? How does ForkJoinPool work? What is `CallerRunsPolicy` and when is it useful?

---

### Q36. How does `CompletableFuture` work? Show common patterns.

**Concept:** `CompletableFuture` (Java 8+) is a `Future` that can be completed manually and supports non-blocking chaining of operations. It's the backbone of async programming in Java, supporting map/flatMap-style transformations, combining futures, and error handling.

```java
// Basic async computation
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // Runs on ForkJoinPool.commonPool() by default
    return fetchFromDB(); // simulated I/O
});

// Chaining transformations
CompletableFuture<Integer> result = CompletableFuture
    .supplyAsync(() -> "Hello")          // async start
    .thenApply(s -> s + " World")        // transform (sync)
    .thenApply(String::length)           // another transform
    .thenApplyAsync(len -> len * 2);     // async transform

// Combining two futures
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "World");
CompletableFuture<String> combined = f1.thenCombine(f2, (a, b) -> a + " " + b);
System.out.println(combined.get()); // "Hello World"

// Wait for all / any
CompletableFuture<Void> all = CompletableFuture.allOf(f1, f2);
CompletableFuture<Object> any = CompletableFuture.anyOf(f1, f2);

// Error handling
CompletableFuture<String> safe = CompletableFuture
    .supplyAsync(() -> { throw new RuntimeException("oops"); return ""; })
    .exceptionally(ex -> "fallback: " + ex.getMessage())
    .thenApply(String::toUpperCase);
System.out.println(safe.get()); // "FALLBACK: OOPS"

// Java 9+ timeout support
CompletableFuture<String> withTimeout = future
    .orTimeout(5, TimeUnit.SECONDS)           // throws TimeoutException
    .completeOnTimeout("default", 5, TimeUnit.SECONDS); // returns default
```

**Key Methods:**

| Method | Input | Output | Async? |
|--------|-------|--------|--------|
| `thenApply(fn)` | T → U | `CF<U>` | Sync |
| `thenApplyAsync(fn)` | T → U | `CF<U>` | Async |
| `thenCompose(fn)` | T → `CF<U>` | `CF<U>` | Like flatMap |
| `thenCombine(cf, fn)` | Two CFs → result | `CF<V>` | Combine |
| `exceptionally(fn)` | Throwable → T | `CF<T>` | Error fallback |
| `handle(fn)` | (T, Throwable) → U | `CF<U>` | Both success/error |

**Memory Aid:** `thenApply` = map, `thenCompose` = flatMap, `thenCombine` = zip.

**Interview Tip:** Show that you understand `thenCompose` (flatMap) vs `thenApply` (map) — this is the most confused pair. If the function itself returns a `CompletableFuture`, use `thenCompose` to avoid `CF<CF<T>>`.

**Follow-ups:** What pool does `supplyAsync` use by default? How do you cancel a CompletableFuture? How does Structured Concurrency (Java 21) improve on CompletableFuture?

---

### Q37. Explain Java 21 Virtual Threads in depth.

**Concept:** Virtual Threads (Project Loom, finalized in Java 21) are lightweight threads managed by the JVM, not the OS. They're mounted on platform (OS) threads via a scheduler. When a virtual thread blocks (I/O, sleep), it's unmounted from its carrier thread, freeing it for other virtual threads. This enables millions of concurrent threads for I/O-bound workloads.

```java
// Creating virtual threads
Thread vt1 = Thread.ofVirtual().name("vt-1").start(() -> {
    System.out.println(Thread.currentThread());
});

// Virtual thread per task executor (most common pattern)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 100_000).forEach(i ->
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1)); // blocks but doesn't pin carrier
            return i;
        })
    );
} // auto-shutdown — waits for all tasks

// Thread.startVirtualThread — shorthand
Thread.startVirtualThread(() -> System.out.println("Quick virtual task"));

// Comparison: 100K platform threads = OOM; 100K virtual threads = fine
```

**Virtual Threads vs Platform Threads:**

| Feature | Platform Thread | Virtual Thread |
|---------|----------------|----------------|
| Managed by | OS | JVM scheduler |
| Memory | ~1 MB stack | ~few KB (growable) |
| Count limit | Thousands (OS limit) | Millions |
| Blocking cost | Expensive (wastes OS thread) | Cheap (unmounts) |
| Scheduling | OS preemptive | JVM cooperative (work-stealing FJP) |
| `synchronized` | Fine | Pins carrier thread! ⚠️ |
| `ReentrantLock` | Fine | Fine (recommended) |
| Best for | CPU-bound work | I/O-bound work |

**Critical Pitfalls:**
- **Pinning:** `synchronized` blocks and native methods pin the virtual thread to its carrier (defeating the purpose). Use `ReentrantLock` instead.
- **Thread-local abuse:** Virtual threads are cheap to create — don't pool them. Thread-local data should be minimal or use Scoped Values (Java 21 preview).
- **CPU-bound tasks:** Virtual threads don't help for CPU-bound work — you still need platform threads for parallelism.

```java
// ❌ BAD — synchronized pins virtual thread
synchronized (lock) {
    socket.read(); // pinned! Carrier thread blocked!
}

// ✅ GOOD — ReentrantLock doesn't pin
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    socket.read(); // virtual thread unmounts, carrier freed
} finally {
    lock.unlock();
}
```

**Interview Tip:** The two key points are: (1) Virtual threads are for I/O-bound concurrency, not CPU parallelism. (2) Replace `synchronized` with `ReentrantLock` to avoid pinning. Mention that Spring Boot 3.2+ supports virtual threads out of the box.

**Follow-ups:** What is thread pinning? How do Scoped Values differ from ThreadLocal? How does the virtual thread scheduler work?

---

### Q38. What is Structured Concurrency (Java 21+)?

**Concept:** Structured Concurrency (preview in Java 21, second preview in Java 23) treats concurrent tasks as a structured unit — when the scope completes, all subtasks are guaranteed to complete or be cancelled. This eliminates thread leaks, cancellation bugs, and observability issues common with unstructured `CompletableFuture` usage.

```java
// Java 21 (preview) — StructuredTaskScope
import java.util.concurrent.StructuredTaskScope;

record UserProfile(String user, String orders) {}

UserProfile fetchProfile(String userId) throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        // Fork subtasks
        Subtask<String> userTask   = scope.fork(() -> fetchUser(userId));
        Subtask<String> ordersTask = scope.fork(() -> fetchOrders(userId));

        scope.join();            // wait for both
        scope.throwIfFailed();   // propagate first failure

        // Both succeeded
        return new UserProfile(userTask.get(), ordersTask.get());
    }
    // scope close guarantees: all subtasks done or cancelled
}

// ShutdownOnSuccess — return first successful result, cancel rest
String findFastest(String query) throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
        scope.fork(() -> searchGoogle(query));
        scope.fork(() -> searchBing(query));
        scope.fork(() -> searchDuck(query));

        scope.join();
        return scope.result(); // first successful result
    }
}
```

**Structured vs Unstructured Concurrency:**

| Aspect | CompletableFuture (Unstructured) | StructuredTaskScope (Structured) |
|--------|----------------------------------|----------------------------------|
| Lifecycle | Unbounded — futures can outlive parent | Bounded — scope limits lifetime |
| Cancellation | Manual, error-prone | Automatic on scope close |
| Error propagation | Must handle each future | `throwIfFailed()` — unified |
| Thread leaks | Possible | Impossible (scope enforced) |
| Observability | Hard to trace parent-child | Thread dump shows scope hierarchy |

**Edge Cases:**
- Structured Concurrency is still preview as of Java 23 — use `--enable-preview`.
- It's designed to work with Virtual Threads — fork thousands of subtasks efficiently.
- `ShutdownOnFailure` cancels remaining subtasks on first failure; `ShutdownOnSuccess` cancels remaining on first success.

**Interview Tip:** Position Structured Concurrency as the successor to `CompletableFuture.allOf()` for fork-join patterns. Emphasize the "structured" aspect — like structured programming replaced `goto`, structured concurrency replaces unstructured thread management.

**Follow-ups:** How does cancellation propagate? Can you nest `StructuredTaskScope`s?

---

### Q39. Explain `ReentrantLock` vs `synchronized`.

**Concept:** `ReentrantLock` is an explicit lock from `java.util.concurrent.locks` offering features `synchronized` lacks: tryLock, timed lock, interruptible lock, fairness policy, and multiple conditions. Both are reentrant (the same thread can acquire the lock multiple times).

```java
ReentrantLock lock = new ReentrantLock(true); // fair lock

// Basic usage
lock.lock();
try {
    // critical section
} finally {
    lock.unlock(); // MUST be in finally!
}

// Try lock — non-blocking
if (lock.tryLock(2, TimeUnit.SECONDS)) {
    try {
        // acquired within 2 seconds
    } finally {
        lock.unlock();
    }
} else {
    System.out.println("Could not acquire lock");
}

// Condition variables (like wait/notify but more flexible)
Condition notEmpty = lock.newCondition();
Condition notFull  = lock.newCondition();

// Producer
lock.lock();
try {
    while (queue.isFull()) notFull.await();
    queue.add(item);
    notEmpty.signal();
} finally { lock.unlock(); }
```

| Feature | `synchronized` | `ReentrantLock` |
|---------|---------------|-----------------|
| Lock acquisition | Implicit (block entry) | Explicit (`lock()`) |
| Release | Implicit (block exit) | Explicit (`unlock()` in `finally`) |
| Try lock | ❌ | ✅ `tryLock()` |
| Timed lock | ❌ | ✅ `tryLock(time)` |
| Interruptible | ❌ | ✅ `lockInterruptibly()` |
| Fairness | No guarantee | Configurable |
| Conditions | One (wait/notify) | Multiple (`newCondition()`) |
| Virtual Thread pinning | ✅ Pins carrier | ❌ Doesn't pin |

**Interview Tip:** In Java 21+, the strongest argument for `ReentrantLock` over `synchronized` is virtual thread compatibility — `synchronized` pins the carrier thread, `ReentrantLock` doesn't.

**Follow-ups:** What is a fair vs unfair lock? What is `ReadWriteLock`? What is `StampedLock`?

---

### Q40. What is a deadlock? How to detect and prevent it?

**Concept:** Deadlock occurs when two or more threads each hold a resource and wait for a resource held by another, forming a circular wait. Four conditions must all hold simultaneously: Mutual Exclusion, Hold and Wait, No Preemption, Circular Wait. Breaking any one condition prevents deadlock.

```java
// ❌ Deadlock example
Object lockA = new Object();
Object lockB = new Object();

Thread t1 = new Thread(() -> {
    synchronized (lockA) {
        Thread.sleep(100);
        synchronized (lockB) { /* ... */ } // waiting for lockB held by t2
    }
});

Thread t2 = new Thread(() -> {
    synchronized (lockB) {
        Thread.sleep(100);
        synchronized (lockA) { /* ... */ } // waiting for lockA held by t1
    }
});
// DEADLOCK! t1 holds lockA, needs lockB; t2 holds lockB, needs lockA

// ✅ FIX 1: Consistent lock ordering
Thread t1Fixed = new Thread(() -> {
    synchronized (lockA) {       // always lock A first
        synchronized (lockB) { /* ... */ }
    }
});
Thread t2Fixed = new Thread(() -> {
    synchronized (lockA) {       // always lock A first (same order!)
        synchronized (lockB) { /* ... */ }
    }
});

// ✅ FIX 2: tryLock with timeout
ReentrantLock lock1 = new ReentrantLock();
ReentrantLock lock2 = new ReentrantLock();

boolean acquiredBoth = false;
if (lock1.tryLock(1, TimeUnit.SECONDS)) {
    try {
        if (lock2.tryLock(1, TimeUnit.SECONDS)) {
            try { acquiredBoth = true; /* work */ }
            finally { lock2.unlock(); }
        }
    } finally { lock1.unlock(); }
}
```

**Prevention Strategies:**

| Strategy | Breaks condition | How |
|----------|-----------------|-----|
| Lock ordering | Circular Wait | Always acquire locks in same global order |
| `tryLock` with timeout | Hold and Wait | Release if can't acquire all |
| Lock-free algorithms | Mutual Exclusion | CAS-based data structures |
| `jstack` / JMX | Detection | Identify deadlocked threads at runtime |

**Memory Aid:** Deadlock needs **MHNC**: **M**utual exclusion, **H**old and wait, **N**o preemption, **C**ircular wait. Break any one = no deadlock.

**Interview Tip:** Be ready to write a deadlock example AND its fix on a whiteboard. Mention `jstack <pid>` for detection and `ThreadMXBean.findDeadlockedThreads()` for programmatic detection.

**Follow-ups:** What is livelock? What is thread starvation? How does `jstack` identify deadlocks?

---

### Q41. What is a race condition? Give an example and fix.

**Concept:** A race condition occurs when the correctness of a program depends on the timing/ordering of thread execution. The classic case is check-then-act: checking a condition and acting on it non-atomically, allowing another thread to change the state between the check and act.

```java
// ❌ Race condition: check-then-act
class UnsafeCounter {
    private int count = 0;
    void increment() { count++; } // read-modify-write: NOT atomic!
}

// ❌ Race condition: singleton (broken double-checked locking without volatile)
class BrokenSingleton {
    private static BrokenSingleton instance;
    static BrokenSingleton getInstance() {
        if (instance == null) {           // Thread A checks
            instance = new BrokenSingleton(); // Thread B also checks, both create
        }
        return instance;
    }
}

// ✅ Fix 1: AtomicInteger
class SafeCounter {
    private final AtomicInteger count = new AtomicInteger(0);
    void increment() { count.incrementAndGet(); } // atomic CAS
}

// ✅ Fix 2: Proper double-checked locking
class SafeSingleton {
    private static volatile SafeSingleton instance; // volatile required!
    static SafeSingleton getInstance() {
        if (instance == null) {
            synchronized (SafeSingleton.class) {
                if (instance == null) {
                    instance = new SafeSingleton();
                }
            }
        }
        return instance;
    }
}
```

**Interview Tip:** Explain why `volatile` is needed in double-checked locking: without it, the JVM may reorder the write to `instance` before the constructor completes, allowing another thread to see a partially constructed object.

**Follow-ups:** What is the difference between a race condition and a data race? What is the Java Memory Model?

---

### Q42. Explain `wait()`, `notify()`, `notifyAll()` with a Producer-Consumer example.

**Concept:** These are `Object` methods for inter-thread communication. A thread calls `wait()` to release the monitor lock and sleep until notified. `notify()` wakes one waiting thread; `notifyAll()` wakes all. They must be called within a `synchronized` block on the same object.

```java
class BoundedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;

    BoundedBuffer(int capacity) { this.capacity = capacity; }

    public synchronized void put(T item) throws InterruptedException {
        while (queue.size() == capacity) {  // MUST use while, not if (spurious wakeup)
            wait();  // release lock and wait
        }
        queue.add(item);
        notifyAll(); // wake waiting consumers
    }

    public synchronized T take() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();
        }
        T item = queue.poll();
        notifyAll(); // wake waiting producers
        return item;
    }
}

// Modern alternative: BlockingQueue
BlockingQueue<String> bq = new ArrayBlockingQueue<>(10);
bq.put("item");   // blocks if full
bq.take();        // blocks if empty
```

**Key Rules:**
- Always call `wait()` inside a `while` loop (not `if`) — spurious wakeups can occur.
- Always use `notifyAll()` over `notify()` to avoid missed signals in multi-consumer scenarios.
- Must hold the monitor lock when calling `wait()`/`notify()` — otherwise `IllegalMonitorStateException`.

**Interview Tip:** Mention that `BlockingQueue` is the modern replacement for manual wait/notify Producer-Consumer. Show both approaches to demonstrate breadth.

---

### Q43. What are `CountDownLatch`, `CyclicBarrier`, `Semaphore`, and `Phaser`?

**Concept:** These are high-level synchronization aids in `java.util.concurrent`. `CountDownLatch` is a one-time countdown (threads wait until count reaches 0). `CyclicBarrier` is reusable (threads wait until N arrive). `Semaphore` controls access to a resource pool. `Phaser` is a flexible, reusable barrier with phases.

```java
// CountDownLatch — one-shot, count down to zero
CountDownLatch latch = new CountDownLatch(3);
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        doWork();
        latch.countDown(); // decrement
    }).start();
}
latch.await(); // blocks until count == 0
System.out.println("All 3 tasks complete");

// CyclicBarrier — reusable, all threads arrive then proceed together
CyclicBarrier barrier = new CyclicBarrier(3, () -> System.out.println("Phase complete"));
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        phase1();
        barrier.await();   // all 3 must arrive before any proceeds
        phase2();
        barrier.await();   // barrier resets automatically!
    }).start();
}

// Semaphore — control concurrent access
Semaphore sem = new Semaphore(3); // 3 permits
sem.acquire();     // get permit (blocks if none available)
try {
    accessResource();
} finally {
    sem.release();  // return permit
}
```

| Feature | CountDownLatch | CyclicBarrier | Semaphore | Phaser |
|---------|---------------|---------------|-----------|--------|
| Reusable | ❌ (one-shot) | ✅ | ✅ | ✅ |
| Waits for | Count → 0 | All parties arrive | Permit available | Phase advance |
| Who counts | Any thread | Only waiting threads | Any thread | Registered parties |
| Use case | "Wait for N tasks" | "Meet at checkpoint" | "Connection pool" | "Multi-phase tasks" |

**Interview Tip:** Know the difference between CountDownLatch (one-way: workers count down, waiter unblocks) and CyclicBarrier (all threads wait for each other). Mention Phaser as the flexible replacement that supports dynamic party registration.

---

### Q44. What is `ThreadLocal`? When should you use it vs Scoped Values (Java 21)?

**Concept:** `ThreadLocal` provides per-thread storage — each thread has its own copy of the variable. Useful for thread-specific context (e.g., user session, DB connection). Java 21 introduces Scoped Values (preview) as a lighter, immutable alternative designed for Virtual Threads.

```java
// ThreadLocal
ThreadLocal<String> userContext = new ThreadLocal<>();
userContext.set("Alice");
System.out.println(userContext.get()); // "Alice" in this thread
userContext.remove(); // MUST remove to prevent memory leaks!

// ThreadLocal with initial value
ThreadLocal<SimpleDateFormat> dateFormat =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

// ❌ Problem with Virtual Threads:
// 1 million virtual threads × ThreadLocal = 1 million copies = memory pressure
// ThreadLocal is mutable and inherited, making lifecycle hard to track

// ✅ Scoped Values (Java 21 preview) — immutable, bounded lifetime
// ScopedValue<String> USER = ScopedValue.newInstance();
// ScopedValue.where(USER, "Alice").run(() -> {
//     System.out.println(USER.get()); // "Alice"
//     handleRequest(); // USER accessible in called methods
// });
// USER.get() throws outside scope — no leaks!
```

| Feature | ThreadLocal | ScopedValue (Java 21+) |
|---------|------------|----------------------|
| Mutability | Mutable | Immutable (rebindable) |
| Lifecycle | Unbounded (must call `remove()`) | Bounded by scope |
| Memory | Per-thread copy | Shared, stack-like |
| Virtual Thread friendly | ❌ (memory waste) | ✅ (designed for it) |
| Inheritance | `InheritableThreadLocal` | Automatic in structured scope |

**Interview Tip:** For Java 21+ interviews, position ScopedValues as the replacement for ThreadLocal in virtual thread contexts. Mention that ThreadLocal is still fine for platform threads.

---

### Q45. How does `ForkJoinPool` work? Explain work-stealing.

**Concept:** `ForkJoinPool` is designed for divide-and-conquer parallelism. Tasks are split (`fork()`) into subtasks and results are combined (`join()`). Each worker thread has a local deque: it pushes/pops its own tasks from one end, and "steals" tasks from other threads' deques when idle.

```java
class SumTask extends RecursiveTask<Long> {
    private final int[] arr;
    private final int start, end;
    private static final int THRESHOLD = 1000;

    SumTask(int[] arr, int start, int end) {
        this.arr = arr; this.start = start; this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += arr[i];
            return sum;
        }
        int mid = (start + end) / 2;
        SumTask left  = new SumTask(arr, start, mid);
        SumTask right = new SumTask(arr, mid, end);
        left.fork();           // submit left to queue
        long rightResult = right.compute(); // compute right in current thread
        long leftResult  = left.join();     // get left's result
        return leftResult + rightResult;
    }
}

ForkJoinPool pool = ForkJoinPool.commonPool();
int[] data = IntStream.range(0, 1_000_000).toArray();
long result = pool.invoke(new SumTask(data, 0, data.length));
```

**Interview Tip:** Mention that parallel streams use `ForkJoinPool.commonPool()` internally. Be aware that `commonPool` size = `Runtime.getRuntime().availableProcessors() - 1`.

---

### Q46. What are `ReadWriteLock` and `StampedLock`?

**Concept:** `ReadWriteLock` allows multiple concurrent readers but exclusive writers. `StampedLock` (Java 8) adds an optimistic read mode that doesn't acquire a lock at all — it just validates that no write occurred, offering higher throughput for read-dominated workloads.

```java
// ReadWriteLock
ReadWriteLock rwLock = new ReentrantReadWriteLock();
rwLock.readLock().lock();    // multiple readers OK
try { read(); } finally { rwLock.readLock().unlock(); }

rwLock.writeLock().lock();   // exclusive
try { write(); } finally { rwLock.writeLock().unlock(); }

// StampedLock — optimistic reads
StampedLock stampedLock = new StampedLock();

// Optimistic read (no actual lock!)
long stamp = stampedLock.tryOptimisticRead();
int x = this.x, y = this.y;   // read fields
if (!stampedLock.validate(stamp)) { // check if a write happened
    stamp = stampedLock.readLock(); // fallback to read lock
    try { x = this.x; y = this.y; } finally { stampedLock.unlockRead(stamp); }
}
```

| Feature | ReentrantReadWriteLock | StampedLock |
|---------|----------------------|-------------|
| Optimistic reads | ❌ | ✅ |
| Reentrant | ✅ | ❌ |
| Condition support | ✅ | ❌ |
| Performance (reads) | Good | Excellent |
| Complexity | Low | Higher |

**Interview Tip:** `StampedLock` is the performance choice for read-heavy scenarios, but mention it's not reentrant and doesn't support conditions.

---

### Q47. Explain `Exchanger` and `Phaser`.

**Concept:** `Exchanger` is a synchronization point where two threads swap objects. `Phaser` is a flexible, reusable synchronization barrier that supports dynamic registration/deregistration of parties and multiple phases.

```java
// Exchanger — two threads swap data
Exchanger<String> exchanger = new Exchanger<>();

new Thread(() -> {
    String received = exchanger.exchange("from-thread-1"); // blocks until partner
    System.out.println("T1 got: " + received); // "from-thread-2"
}).start();

new Thread(() -> {
    String received = exchanger.exchange("from-thread-2");
    System.out.println("T2 got: " + received); // "from-thread-1"
}).start();

// Phaser — dynamic parties, multiple phases
Phaser phaser = new Phaser(1); // register self
for (int i = 0; i < 3; i++) {
    phaser.register(); // dynamically add party
    new Thread(() -> {
        phaser.arriveAndAwaitAdvance(); // phase 0
        doPhase1();
        phaser.arriveAndAwaitAdvance(); // phase 1
        doPhase2();
        phaser.arriveAndDeregister();   // done, leave
    }).start();
}
phaser.arriveAndDeregister(); // main deregisters
```

**Interview Tip:** `Phaser` is less commonly asked about but shows deep knowledge. Position it as the flexible replacement for both `CountDownLatch` (use `arriveAndDeregister`) and `CyclicBarrier` (use `arriveAndAwaitAdvance`).

---

### Q48. What are `LongAdder` and `LongAccumulator`? When to use over AtomicLong?

**Concept:** `LongAdder` and `LongAccumulator` (Java 8+) reduce contention by maintaining multiple internal cells that different threads update independently. The sum is computed on demand. They're significantly faster than `AtomicLong` under high contention.

```java
// AtomicLong — single value, CAS contention at high load
AtomicLong atomicLong = new AtomicLong(0);
atomicLong.incrementAndGet(); // CAS loop — threads compete on one value

// LongAdder — distributed cells, low contention
LongAdder adder = new LongAdder();
adder.increment();             // updates thread-local cell
adder.add(10);
long total = adder.sum();      // sums all cells (not perfectly consistent under concurrency)

// LongAccumulator — generalized accumulation
LongAccumulator maxFinder = new LongAccumulator(Long::max, Long.MIN_VALUE);
maxFinder.accumulate(42);
maxFinder.accumulate(99);
System.out.println(maxFinder.get()); // 99
```

| Feature | AtomicLong | LongAdder | LongAccumulator |
|---------|-----------|-----------|-----------------|
| Operation | Any atomic op | Add/increment only | Custom binary op |
| Contention | High (single CAS target) | Low (distributed cells) | Low |
| Read cost | O(1) | O(cells) — aggregates | O(cells) |
| Best for | Low-medium contention | High contention counters | Custom aggregation |

**Interview Tip:** Use `AtomicLong` for low contention or when exact reads are needed. Use `LongAdder` for high-contention counters (e.g., metrics). This shows awareness of real-world performance tuning.

---

# Section 4 — Modern Java Features (Q49–Q62)

---

### Q49. Explain Lambda Expressions and Functional Interfaces (Java 8+).

**Concept:** A lambda is a concise way to implement a functional interface (an interface with exactly one abstract method). Lambdas enable functional programming patterns in Java — passing behavior as data. The syntax is `(parameters) -> expression` or `(parameters) -> { statements; }`.

```java
// Functional interface
@FunctionalInterface
interface Transformer<T> {
    T transform(T input);
    // Can have default and static methods
}

// Lambda implementations
Transformer<String> upper = s -> s.toUpperCase();
Transformer<Integer> doubler = x -> x * 2;

// Built-in functional interfaces (java.util.function)
Predicate<String> isLong     = s -> s.length() > 5;
Function<String, Integer> len = String::length;     // method reference
Consumer<String> printer      = System.out::println;
Supplier<List<String>> listMaker = ArrayList::new;   // constructor reference
UnaryOperator<String> trim    = String::trim;        // T → T
BinaryOperator<Integer> sum   = Integer::sum;        // (T,T) → T
BiFunction<String, Integer, String> repeat = String::repeat; // Java 11

// Composition
Predicate<String> isLongAndUpperCase = isLong.and(s -> s.equals(s.toUpperCase()));
Function<String, String> trimAndUpper = ((Function<String, String>) String::trim)
    .andThen(String::toUpperCase);
```

**Core Functional Interfaces:**

| Interface | Method | Signature | Use |
|-----------|--------|-----------|-----|
| `Predicate<T>` | `test(T)` | T → boolean | Filtering |
| `Function<T,R>` | `apply(T)` | T → R | Transformation |
| `Consumer<T>` | `accept(T)` | T → void | Side effects |
| `Supplier<T>` | `get()` | () → T | Lazy creation |
| `UnaryOperator<T>` | `apply(T)` | T → T | Same-type transform |
| `BinaryOperator<T>` | `apply(T,T)` | (T,T) → T | Reduction |
| `BiFunction<T,U,R>` | `apply(T,U)` | (T,U) → R | Two-arg transform |

**Interview Tip:** Know all core functional interfaces cold. Be ready to convert an anonymous inner class to a lambda to a method reference. Mention `@FunctionalInterface` annotation for compile-time validation.

**Follow-ups:** What are effectively final variables? Can lambdas throw checked exceptions? How does `invokedynamic` implement lambdas?

---

### Q50. Master the Stream API with examples.

**Concept:** Streams provide a declarative, pipeline-based approach to process collections. A stream pipeline has a source, zero or more intermediate operations (lazy), and a terminal operation (triggers processing). Streams are not data structures — they don't store elements.

```java
List<String> names = List.of("Alice", "Bob", "Charlie", "David", "Eve");

// Filter + Map + Collect
List<String> result = names.stream()
    .filter(n -> n.length() > 3)     // intermediate (lazy)
    .map(String::toUpperCase)        // intermediate
    .sorted()                        // intermediate
    .toList();                       // terminal (Java 16+)
// [ALICE, CHARLIE, DAVID]

// Reduce
int totalLength = names.stream()
    .mapToInt(String::length)
    .sum(); // 23

// Grouping
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
// {3=[Bob, Eve], 5=[Alice, David], 7=[Charlie]}

// Partitioning
Map<Boolean, List<String>> parts = names.stream()
    .collect(Collectors.partitioningBy(n -> n.length() > 3));

// flatMap — flatten nested structures
List<List<Integer>> nested = List.of(List.of(1,2), List.of(3,4));
List<Integer> flat = nested.stream()
    .flatMap(Collection::stream)
    .toList(); // [1, 2, 3, 4]

// Parallel stream (use for CPU-bound, large datasets)
long count = names.parallelStream()
    .filter(n -> n.length() > 3)
    .count();

// Collectors.teeing (Java 12+) — two collectors at once
var stats = names.stream().collect(Collectors.teeing(
    Collectors.counting(),
    Collectors.joining(", "),
    (count2, joined) -> "Count: " + count2 + " Names: " + joined
));
```

**Key Intermediate vs Terminal Operations:**

| Intermediate (lazy) | Terminal (eager) |
|---------------------|------------------|
| `filter`, `map`, `flatMap` | `collect`, `toList()`, `forEach` |
| `sorted`, `distinct`, `limit` | `reduce`, `count`, `sum` |
| `peek`, `skip`, `takeWhile` | `findFirst`, `findAny`, `anyMatch` |
| `mapToInt/Long/Double` | `min`, `max`, `average` |

**Edge Cases:**
- Streams are single-use — cannot be reused after a terminal operation.
- `peek` is for debugging only — it's not guaranteed to execute for all elements in short-circuit operations.
- `parallelStream` is not always faster — overhead can outweigh gains for small datasets.
- `Stream.toList()` (Java 16) returns an unmodifiable list; `collect(Collectors.toList())` returns a modifiable list.

**Interview Tip:** Be ready to solve any collection-processing problem using streams. Know `groupingBy`, `partitioningBy`, `teeing`, and `toMap` collectors inside out.

**Follow-ups:** What is stream laziness? How does parallel stream use ForkJoinPool? What are `Gatherers` (Java 22 preview)?

---

### Q51. Explain `Optional` — best practices and anti-patterns.

**Concept:** `Optional<T>` is a container that may or may not hold a non-null value. It's designed to be a return type for methods that might not have a result, replacing `null` returns. It encourages explicit handling of absence.

```java
// Creating Optionals
Optional<String> present = Optional.of("Hello");
Optional<String> empty   = Optional.empty();
Optional<String> maybe   = Optional.ofNullable(possiblyNull); // null-safe

// Using Optional (functional style)
String result = maybe
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase)
    .orElse("DEFAULT");

// orElseGet — lazy default (preferred over orElse for expensive defaults)
String lazy = maybe.orElseGet(() -> computeExpensiveDefault());

// orElseThrow (Java 10+)
String value = maybe.orElseThrow(); // throws NoSuchElementException
String custom = maybe.orElseThrow(() -> new NotFoundException("Not found"));

// ifPresentOrElse (Java 9+)
maybe.ifPresentOrElse(
    val -> System.out.println("Found: " + val),
    ()  -> System.out.println("Empty")
);

// or() — lazy alternative Optional (Java 9+)
Optional<String> fallback = maybe.or(() -> Optional.of("fallback"));

// stream() — converts to 0-or-1 element stream (Java 9+)
List<String> list = optionals.stream()
    .flatMap(Optional::stream) // filter out empties
    .toList();
```

**Best Practices vs Anti-Patterns:**

| ✅ Do | ❌ Don't |
|------|---------|
| Return `Optional` from methods | Use `Optional` as method parameter |
| Chain `map`/`flatMap`/`filter` | Call `get()` without `isPresent()` |
| Use `orElse`/`orElseGet` | Use `Optional` for fields |
| Use `orElseThrow` for required values | Use `Optional` in collections |
| Use `Optional.stream()` for flattening | Use `Optional.of(null)` — throws NPE |

**Interview Tip:** Show that you treat `Optional` as a monadic container, not a null-check wrapper. Chain operations functionally. Never call `.get()` directly.

---

### Q52. What are method references? Explain all four kinds.

**Concept:** Method references are a shorthand for lambdas that call an existing method. There are four kinds: static method, instance method of a particular object, instance method of an arbitrary object of a given type, and constructor reference.

```java
// 1. Static method reference: ClassName::staticMethod
Function<String, Integer> parse = Integer::parseInt;  // s -> Integer.parseInt(s)

// 2. Instance method of particular object: instance::method
String prefix = "Hello ";
Function<String, String> greeter = prefix::concat;    // s -> prefix.concat(s)

// 3. Instance method of arbitrary object: ClassName::instanceMethod
Function<String, String> upper = String::toUpperCase;  // s -> s.toUpperCase()
BiFunction<String, String, Boolean> starts = String::startsWith; // (s, prefix) -> s.startsWith(prefix)

// 4. Constructor reference: ClassName::new
Supplier<ArrayList<String>> listFactory = ArrayList::new;
Function<String, Integer> intFactory = Integer::new;  // deprecated, but syntactically valid
```

| Kind | Syntax | Lambda equivalent |
|------|--------|-------------------|
| Static | `ClassName::staticMethod` | `x -> ClassName.staticMethod(x)` |
| Bound instance | `instance::method` | `x -> instance.method(x)` |
| Unbound instance | `ClassName::method` | `(obj, x) -> obj.method(x)` |
| Constructor | `ClassName::new` | `x -> new ClassName(x)` |

**Interview Tip:** The tricky one is #3 (unbound instance) — the first parameter becomes `this`. For example, `String::length` as a `Function<String, Integer>` means `s -> s.length()`.

---

### Q53. What are default and static methods in interfaces (Java 8+)?

**Concept:** Default methods provide interface evolution without breaking existing implementations. Static methods provide utility methods on the interface. Java 9 added private methods for code reuse within default methods.

```java
public interface Loggable {
    // Abstract (must implement)
    String getName();

    // Default method (Java 8+)
    default void log(String message) {
        System.out.println(formatLog(getName(), message));
    }

    // Static method (Java 8+)
    static Loggable create(String name) {
        return () -> name;  // lambda for functional interface? No — has default.
    }

    // Private method (Java 9+) — reuse in default methods
    private String formatLog(String name, String msg) {
        return "[%s] %s: %s".formatted(java.time.Instant.now(), name, msg);
    }
}

// Diamond problem resolution
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }

class C implements A, B {
    @Override
    public void hello() {
        A.super.hello(); // explicit resolution required
    }
}
```

**Interview Tip:** Know the diamond problem resolution rule: if two interfaces provide conflicting default methods, the implementing class MUST override and explicitly choose (using `InterfaceName.super.method()`).

---

### Q54. Explain Java 17 Sealed Classes in depth with pattern matching.

**Concept:** Sealed classes restrict which classes can extend them using `permits`. Combined with pattern matching `switch` (Java 21), the compiler can verify exhaustiveness — all subtypes are handled. This enables safe, compiler-checked algebraic data types.

```java
// Sealed hierarchy
public sealed interface Expr permits Num, Add, Mul, Neg {}
public record Num(double value)         implements Expr {}
public record Add(Expr left, Expr right) implements Expr {}
public record Mul(Expr left, Expr right) implements Expr {}
public record Neg(Expr expr)            implements Expr {}

// Exhaustive pattern matching (Java 21)
public static double evaluate(Expr expr) {
    return switch (expr) {
        case Num(double v)          -> v;
        case Add(var left, var right) -> evaluate(left) + evaluate(right);
        case Mul(var left, var right) -> evaluate(left) * evaluate(right);
        case Neg(var inner)         -> -evaluate(inner);
        // No default needed — sealed + permits = exhaustive!
    };
}

// Guarded patterns (Java 21)
public static String describe(Expr expr) {
    return switch (expr) {
        case Num(double v) when v == 0 -> "zero";
        case Num(double v) when v < 0  -> "negative number";
        case Num n                      -> "number: " + n.value();
        case Add a                      -> "addition";
        case Mul m                      -> "multiplication";
        case Neg n                      -> "negation";
    };
}

// Usage
Expr expr = new Add(new Num(3), new Mul(new Num(2), new Num(5)));
System.out.println(evaluate(expr)); // 13.0
```

**Interview Tip:** This pattern (sealed interface + records + pattern switch) is the single most important modern Java pattern for 2026 interviews. It replaces the Visitor pattern with much cleaner code.

---

### Q55. Explain Java 21 Pattern Matching for switch (finalized).

**Concept:** Pattern matching for switch (finalized in Java 21) allows matching on types, record patterns (destructuring), guarded patterns (`when` clauses), and null handling. Combined with sealed types, it provides exhaustive, expressive branching.

```java
// Type patterns
static String format(Object obj) {
    return switch (obj) {
        case null            -> "null";
        case Integer i       -> "int: " + i;
        case Long l          -> "long: " + l;
        case Double d        -> "double: %f".formatted(d);
        case String s        -> "string: \"%s\"".formatted(s);
        case int[] arr       -> "int array of length " + arr.length;
        default              -> "other: " + obj.getClass().getSimpleName();
    };
}

// Record patterns with guards
record Point(int x, int y) {}
static String classify(Point p) {
    return switch (p) {
        case Point(int x, int y) when x == 0 && y == 0 -> "origin";
        case Point(int x, int y) when x == 0            -> "on Y-axis";
        case Point(int x, int y) when y == 0            -> "on X-axis";
        case Point(int x, int y) when x == y            -> "on diagonal";
        case Point(int x, int y)                         -> "(%d, %d)".formatted(x, y);
    };
}

// Null handling (must be explicit)
// Before Java 21: switch(null) → NullPointerException
// Java 21: case null is a valid pattern
```

**Switch Pattern Matching Rules:**

| Rule | Description |
|------|-------------|
| Dominance | More specific patterns must come before general ones |
| Exhaustiveness | All types must be covered (sealed types or default) |
| Null | `null` must be handled explicitly; otherwise NPE |
| Guards | `when` clause added after pattern: `case T t when condition` |

**Interview Tip:** Stress that `case null` is now a first-class pattern. Show nested record patterns for deep destructuring.

---

### Q56. What are Text Blocks (Java 15) and String Templates (Java 21 preview)?

**Concept:** Text Blocks are multi-line string literals delimited by `"""`. They handle indentation, escaping, and formatting cleanly. String Templates (preview in Java 21, removed/reworked in later versions) aimed to provide safe interpolation.

```java
// Text Block (Java 15 — finalized)
String json = """
        {
            "name": "Alice",
            "age": 30,
            "city": "Mumbai"
        }
        """;
// Indentation is stripped based on closing """ position

// Useful methods with text blocks
String html = """
        <html>
            <body>
                <p>%s</p>
            </body>
        </html>
        """.formatted("Hello!"); // String.formatted() (Java 15)

// String methods (Java 11+)
"  hello  ".strip();           // "hello" (Unicode-aware, unlike trim())
"  hello  ".stripLeading();    // "hello  "
"  hello  ".stripTrailing();   // "  hello"
"".isBlank();                  // true
"hello\nworld".lines().toList(); // ["hello", "world"]
"ha".repeat(3);                // "hahaha"

// Java 12+
"hello".indent(4);             // "    hello\n"
"hello".transform(s -> s.toUpperCase()); // "HELLO"
```

**Interview Tip:** Know the indentation rules for text blocks: the compiler uses the position of the closing `"""` to determine the common whitespace prefix to strip. Always mention `.formatted()` as the modern replacement for `String.format()`.

---

### Q57. Explain `switch` expressions (Java 14+).

**Concept:** Switch expressions return a value and use arrow syntax (`->`) for concise, fall-through-free cases. They can be used as statements or expressions. Combined with pattern matching (Java 21), they become the primary branching construct.

```java
// Switch expression with arrow syntax
int day = 3;
String dayName = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    case 4, 5 -> "Thu or Fri";  // multiple labels
    default -> "Other";
};

// yield for multi-statement blocks
String result = switch (day) {
    case 1, 2, 3, 4, 5 -> {
        System.out.println("Weekday");
        yield "workday"; // yield returns value from block
    }
    case 6, 7 -> "weekend";
    default -> throw new IllegalArgumentException();
};

// Exhaustiveness — enum switch doesn't need default if all values covered
enum Season { SPRING, SUMMER, FALL, WINTER }
String weather = switch (season) {
    case SPRING -> "mild";
    case SUMMER -> "hot";
    case FALL   -> "cool";
    case WINTER -> "cold";
    // no default needed — all enum values covered
};
```

**Interview Tip:** Distinguish `yield` (returns value from a block in switch expression) from `return` (exits the method).

---

### Q58. What is the Java Module System (JPMS, Java 9)?

**Concept:** The Java Platform Module System (Project Jigsaw) provides strong encapsulation at the package level. A module declares what it exports (accessible to other modules) and what it requires (dependencies). It replaced the monolithic `rt.jar` with modular JDK.

```java
// module-info.java
module com.myapp {
    requires java.sql;            // depends on java.sql module
    requires transitive java.logging; // transitive dependency
    exports com.myapp.api;        // only this package is accessible
    opens com.myapp.model to com.fasterxml.jackson.databind; // reflection access
}
```

| Directive | Purpose |
|-----------|---------|
| `requires` | Declares dependency on another module |
| `requires transitive` | Dependency is propagated to consumers |
| `exports` | Makes package accessible to all modules |
| `exports ... to` | Makes package accessible to specific modules |
| `opens` | Allows reflection access |
| `provides ... with` | Service provider |
| `uses` | Service consumer |

**Interview Tip:** JPMS is less commonly asked in depth but knowing the basics (why modules, what `module-info.java` does, `exports` vs `opens`) shows breadth.

---

### Q59. What are Java 21 Sequenced Collections?

**Concept:** Java 21 introduced `SequencedCollection`, `SequencedSet`, and `SequencedMap` to provide a uniform API for collections with a defined encounter order. Previously, getting the first/last element required different methods for different collections.

```java
// Before Java 21 — inconsistent APIs
List<String> list = new ArrayList<>(List.of("a", "b", "c"));
list.get(0);                            // first
list.get(list.size() - 1);             // last

LinkedHashSet<String> lhs = new LinkedHashSet<>(List.of("a", "b", "c"));
lhs.iterator().next();                  // first (awkward!)
// last? No direct way!

// Java 21 — unified SequencedCollection API
SequencedCollection<String> seq = new ArrayList<>(List.of("a", "b", "c"));
seq.getFirst();     // "a"
seq.getLast();      // "c"
seq.addFirst("z");  // [z, a, b, c]
seq.addLast("d");   // [z, a, b, c, d]
seq.removeFirst();  // [a, b, c, d]
seq.reversed();     // reversed view: [d, c, b, a]

SequencedMap<String, Integer> sMap = new LinkedHashMap<>();
sMap.put("one", 1); sMap.put("two", 2);
sMap.firstEntry();      // one=1
sMap.lastEntry();       // two=2
sMap.pollFirstEntry();  // removes and returns one=1
sMap.sequencedKeySet(); // SequencedSet<String>
```

**Interview Tip:** This is a small but frequently asked Java 21 feature. The key insight is that `reversed()` returns a view, not a copy.

---

### Q60. Explain the HTTP Client API (Java 11+).

**Concept:** The `java.net.http` package (Java 11) provides a modern, non-blocking HTTP client supporting HTTP/1.1 and HTTP/2. It replaces `HttpURLConnection` with a builder-based, immutable API supporting async operations via `CompletableFuture`.

```java
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)
    .connectTimeout(Duration.ofSeconds(10))
    .followRedirects(HttpClient.Redirect.NORMAL)
    .build();

// Synchronous GET
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .header("Accept", "application/json")
    .GET()
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.statusCode()); // 200
System.out.println(response.body());

// Asynchronous GET
CompletableFuture<HttpResponse<String>> asyncResponse =
    client.sendAsync(request, HttpResponse.BodyHandlers.ofString());

asyncResponse.thenAccept(r -> System.out.println(r.body()));

// POST with body
HttpRequest postReq = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString("""
        {"name": "Alice", "email": "alice@example.com"}
        """))
    .build();
```

**Interview Tip:** Know synchronous (`send`) vs async (`sendAsync`), and the builder pattern. Mention HTTP/2 support with server push capability.

---

### Q61. What are `Stream.toList()`, `Stream.mapMulti()`, and Stream `Gatherers` (Java 22)?

**Concept:** `toList()` (Java 16) is a concise terminal operation returning an unmodifiable list. `mapMulti()` (Java 16) is a more efficient alternative to `flatMap` for simple cases. `Gatherers` (Java 22 preview, Java 24 finalized) enable custom intermediate operations.

```java
// toList() — Java 16
List<String> list = Stream.of("a", "b", "c").toList(); // unmodifiable

// mapMulti — Java 16 (imperative flatMap)
List<Integer> numbers = Stream.of(1, 2, 3)
    .<Integer>mapMulti((num, consumer) -> {
        consumer.accept(num);
        consumer.accept(num * 10);
    })
    .toList(); // [1, 10, 2, 20, 3, 30]

// Gatherers (Java 22 preview) — custom intermediate operations
// Sliding window example
List<List<Integer>> windows = Stream.of(1, 2, 3, 4, 5)
    .gather(Gatherers.windowSliding(3))
    .toList();
// [[1,2,3], [2,3,4], [3,4,5]]

// Fixed-size groups
List<List<Integer>> groups = Stream.of(1, 2, 3, 4, 5)
    .gather(Gatherers.windowFixed(2))
    .toList();
// [[1,2], [3,4], [5]]
```

**Interview Tip:** `toList()` vs `collect(Collectors.toList())`: the former returns unmodifiable, the latter returns modifiable. This is a common trick question.

---

### Q62. What is `record` pattern matching and deconstruction (Java 21)?

**Concept:** Record patterns allow you to destructure records directly in `instanceof` checks and `switch` expressions, extracting components into variables. Nested patterns enable deep destructuring of record hierarchies.

```java
record Address(String city, String zip) {}
record Person(String name, int age, Address address) {}

// Nested destructuring in switch
static String describeLocation(Object obj) {
    return switch (obj) {
        case Person(var name, var age, Address(var city, _)) when age >= 18
            -> name + " (adult) lives in " + city;
        case Person(var name, _, Address(_, var zip))
            -> name + " has zip: " + zip;
        default -> "unknown";
    };
}

// Unnamed variables (_) — Java 22 (finalized)
// Use _ when you don't need a component
if (obj instanceof Person(var name, _, _)) {
    System.out.println("Name: " + name); // only care about name
}
```

**Interview Tip:** Unnamed variables (`_`) for unused bindings is a Java 22 feature showing you're current. Combined with nested record patterns, this is how modern Java replaces complex if-else chains.

---

# Section 5 — Exception Handling (Q63–Q69)

---

### Q63. Explain the exception hierarchy — Checked vs Unchecked.

**Concept:** All exceptions extend `Throwable`. `Error` (unrecoverable JVM issues) and `RuntimeException` subclasses are unchecked (no `throws` declaration required). All other `Exception` subclasses are checked (must be caught or declared). Checked exceptions enforce handling at compile time.

```
                    Throwable
                   ╱         ╲
              Error         Exception
           (unchecked)     ╱         ╲
           │           RuntimeException   IOException (checked)
           │           (unchecked)        SQLException (checked)
      OutOfMemoryError  │                 ...
      StackOverflowError │
                     NullPointerException
                     ArrayIndexOutOfBoundsException
                     IllegalArgumentException
                     ClassCastException
                     ArithmeticException
```

| Type | Must catch/declare? | Examples | When to use |
|------|:-------------------:|---------|-------------|
| Checked | ✅ | `IOException`, `SQLException` | Recoverable conditions |
| Unchecked | ❌ | `NullPointerException`, `IllegalArgumentException` | Programming errors |
| Error | ❌ | `OutOfMemoryError`, `StackOverflowError` | Never catch (usually) |

**Interview Tip:** The debate on checked vs unchecked is a classic. Know both sides: checked exceptions force handling (safety) but clutter code; unchecked are clean but can be missed. Modern Java and frameworks (Spring) lean toward unchecked.

---

### Q64. How does `try-with-resources` work? Explain `AutoCloseable`.

**Concept:** Try-with-resources (Java 7) automatically closes resources that implement `AutoCloseable` at the end of the try block, even if an exception occurs. It eliminates resource leak bugs and is cleaner than try-finally.

```java
// try-with-resources — automatic closing
try (var reader = new BufferedReader(new FileReader("file.txt"));
     var writer = new BufferedWriter(new FileWriter("out.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        writer.write(line);
        writer.newLine();
    }
} // reader and writer auto-closed (in reverse order!)

// Suppressed exceptions
// If close() throws AND the try block throws, close() exception is suppressed
// Access via: mainException.getSuppressed()

// Custom AutoCloseable
public class DatabaseConnection implements AutoCloseable {
    public DatabaseConnection() { System.out.println("Connected"); }
    public void query(String sql) { /* ... */ }

    @Override
    public void close() {  // called automatically
        System.out.println("Disconnected");
    }
}

try (var db = new DatabaseConnection()) {
    db.query("SELECT * FROM users");
} // "Disconnected" printed automatically

// Java 9+ — effectively final variables in try-with-resources
BufferedReader reader = new BufferedReader(new FileReader("f.txt"));
try (reader) {   // no need to re-declare!
    reader.readLine();
}
```

**Interview Tip:** Know the closing order (reverse of declaration) and suppressed exceptions mechanism. Mention that `Closeable extends AutoCloseable` — `Closeable.close()` can only throw `IOException`, while `AutoCloseable.close()` can throw any `Exception`.

---

### Q65. How to create custom exceptions? Best practices.

**Concept:** Custom exceptions extend `Exception` (checked) or `RuntimeException` (unchecked). They should carry meaningful context, use descriptive names, and provide constructors that chain the cause.

```java
// Custom unchecked exception (most common in modern Java)
public class OrderNotFoundException extends RuntimeException {
    private final String orderId;

    public OrderNotFoundException(String orderId) {
        super("Order not found: " + orderId);
        this.orderId = orderId;
    }

    public OrderNotFoundException(String orderId, Throwable cause) {
        super("Order not found: " + orderId, cause);
        this.orderId = orderId;
    }

    public String getOrderId() { return orderId; }
}

// Usage
public Order findOrder(String id) {
    return repository.findById(id)
        .orElseThrow(() -> new OrderNotFoundException(id));
}

// Custom checked exception (for recoverable conditions)
public class InsufficientFundsException extends Exception {
    private final double deficit;

    public InsufficientFundsException(double deficit) {
        super("Insufficient funds. Deficit: " + deficit);
        this.deficit = deficit;
    }

    public double getDeficit() { return deficit; }
}
```

**Best Practices:**
- Name exceptions with `Exception` suffix and descriptive prefix.
- Include context (IDs, amounts) as fields — don't just use a message string.
- Always provide a constructor that takes a `Throwable cause` for exception chaining.
- Prefer unchecked exceptions for programming errors; checked for recoverable conditions.
- Don't catch `Exception` or `Throwable` broadly — catch specific types.

**Interview Tip:** Show that you include meaningful context in exceptions and chain causes properly. Mention that modern frameworks prefer unchecked exceptions.

---

### Q66. What is the exception catch order rule? Explain multi-catch.

**Concept:** Catch blocks are evaluated top to bottom. More specific exceptions must be caught before more general ones — otherwise, the specific catch becomes unreachable (compile error). Multi-catch (`catch (A | B e)`) handles multiple unrelated exceptions with the same logic.

```java
try {
    riskyOperation();
} catch (FileNotFoundException e) {
    // most specific first
} catch (IOException e) {
    // then parent
} catch (Exception e) {
    // most general last
}

// Multi-catch (Java 7+) — same handler for multiple types
try {
    parseAndProcess();
} catch (NumberFormatException | DateTimeParseException e) {
    // e is effectively final — cannot reassign
    System.out.println("Parse error: " + e.getMessage());
}

// ❌ WRONG — parent before child
// catch (IOException e) { }
// catch (FileNotFoundException e) { } // unreachable — compile error!
```

**Interview Tip:** In multi-catch, the exception variable is effectively final. You cannot reassign it. Also note that multi-catch types cannot have a parent-child relationship.

---

### Q67. Explain `finally` block behavior and edge cases.

**Concept:** `finally` always executes after try/catch, even if an exception is thrown, a return is encountered, or the catch block throws. The only exceptions: `System.exit()`, JVM crash, or daemon thread termination.

```java
// finally always runs
static int tricky() {
    try {
        return 1;
    } finally {
        return 2; // ⚠️ Overrides the return! (bad practice, compiler warning)
    }
}
System.out.println(tricky()); // 2 (not 1!)

// finally with exception
try {
    throw new RuntimeException("try");
} catch (RuntimeException e) {
    throw new RuntimeException("catch"); // this is thrown
} finally {
    // runs before "catch" exception propagates
    System.out.println("finally runs"); // always prints
    // ⚠️ If finally throws, it REPLACES the catch exception (lost!)
}
```

**Edge Cases:**
- `return` in `finally` overrides `return` in `try`/`catch` — avoid this.
- Exception in `finally` replaces the original exception — use try-with-resources instead.
- `finally` does NOT run if: `System.exit()` is called, JVM crashes, or thread is killed.
- In try-with-resources, `close()` executes BEFORE `finally`.

**Interview Tip:** Never return or throw from `finally` — it masks exceptions. This is the most common interview trap.

---

### Q68. What are suppressed exceptions?

**Concept:** When try-with-resources is used and both the try block AND the `close()` method throw exceptions, the `close()` exception is suppressed — attached to the primary exception via `addSuppressed()`. This prevents losing either exception.

```java
public class Resource implements AutoCloseable {
    public void use() { throw new RuntimeException("use failed"); }
    @Override
    public void close() { throw new RuntimeException("close failed"); }
}

try (var r = new Resource()) {
    r.use();
} catch (RuntimeException e) {
    System.out.println("Primary: " + e.getMessage());   // "use failed"
    for (Throwable t : e.getSuppressed()) {
        System.out.println("Suppressed: " + t.getMessage()); // "close failed"
    }
}
```

**Interview Tip:** Suppressed exceptions only exist in try-with-resources. In traditional try-finally, the `finally` exception replaces the original (which is worse).

---

### Q69. Explain `NullPointerException` improvements in Java 14+.

**Concept:** Java 14 introduced Helpful NullPointerExceptions that pinpoint exactly which variable was `null` in a chain of calls, making debugging much easier. Enabled by default since Java 15.

```java
record Address(String city) {}
record Person(Address address) {}

Person person = new Person(null);
person.address().city().length();
// Pre-Java 14: NullPointerException (no detail)
// Java 14+:    NullPointerException: Cannot invoke "Address.city()" because
//              the return value of "Person.address()" is null

// Also works for arrays
int[][] arr = new int[2][];
arr[1][0] = 5;
// Java 14+: NullPointerException: Cannot store to int array because "arr[1]" is null
```

**Interview Tip:** Mention that this is enabled by default since Java 15 (`-XX:+ShowCodeDetailsInExceptionMessages`). It dramatically reduces debugging time for NPEs.

---

# Section 6 — JVM & Memory Management (Q70–Q78)

---

### Q70. Explain JVM Architecture — Heap, Stack, Method Area.

**Concept:** The JVM organizes memory into several runtime data areas: Heap (shared, object storage), Stack (per-thread, method frames with locals/operand stack), Method Area/Metaspace (class metadata, static fields), Program Counter (per-thread, current instruction), and Native Method Stack.

```
┌─────────────────────────── JVM Memory ───────────────────────────┐
│                                                                   │
│  ┌─── Heap (shared across threads) ──────────────────────────┐   │
│  │  Young Generation        │  Old Generation                │   │
│  │  ┌─────┬───────┬───────┐ │                                │   │
│  │  │Eden │  S0   │  S1   │ │  Tenured Space                │   │
│  │  └─────┴───────┴───────┘ │                                │   │
│  └───────────────────────────┴────────────────────────────────┘   │
│                                                                   │
│  ┌─── Metaspace (native memory, replaces PermGen since J8) ──┐   │
│  │  Class metadata, method bytecode, constant pool            │   │
│  └────────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─── Per-Thread Areas ──────────────────────────────────────┐   │
│  │  Thread Stack: Stack frames (locals, operand stack, etc.) │   │
│  │  PC Register: Address of current instruction              │   │
│  │  Native Method Stack: For JNI calls                       │   │
│  └────────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────┘
```

| Area | Shared? | Contains | Tuning Flag |
|------|:-------:|----------|-------------|
| Heap | ✅ | Objects, arrays | `-Xms`, `-Xmx` |
| Metaspace | ✅ | Class metadata | `-XX:MetaspaceSize` |
| Stack | ❌ (per-thread) | Local vars, method frames | `-Xss` |
| PC Register | ❌ (per-thread) | Current instruction pointer | — |

**Edge Cases:**
- `String` pool is in the Heap (since Java 7, moved from PermGen).
- PermGen was removed in Java 8, replaced by Metaspace (uses native memory).
- Virtual Thread stacks are on the Heap, not in OS thread stacks.

**Interview Tip:** Draw this diagram. Mention the Young → Old generation promotion and that Metaspace can grow unbounded (unlike PermGen which had fixed size).

---

### Q71. How does Garbage Collection work? Explain G1, ZGC, and Shenandoah.

**Concept:** GC automatically reclaims unreachable objects. Modern JVM uses generational collection: short-lived objects in Young Gen (collected frequently, cheaply), long-lived in Old Gen (collected less frequently). G1 is the default since Java 9; ZGC and Shenandoah target ultra-low pause times.

```
Allocation → Eden → Minor GC → Survivor (S0↔S1) → Age threshold → Old Gen → Major/Full GC
```

| Collector | Pause Target | Heap Size | Default? | Best For |
|-----------|:----------:|:---------:|:--------:|----------|
| G1GC | ~200ms | Medium-Large | ✅ (since J9) | General purpose |
| ZGC | < 1ms | Large (TB-scale) | ❌ | Low-latency |
| Shenandoah | < 10ms | Medium-Large | ❌ | Low-latency |
| Serial | Unpredictable | Small | ❌ | Small apps, containers |
| Parallel | Throughput | Large | ❌ | Batch processing |

**G1GC (Garbage-First):**
- Divides heap into ~2048 equal regions (Eden, Survivor, Old, Humongous).
- Collects regions with most garbage first (hence "Garbage-First").
- Concurrent marking; stop-the-world for young and mixed collection.
- Tuning: `-XX:MaxGCPauseMillis=200` (default target).

**ZGC (Java 15+ production-ready):**
- Sub-millisecond pauses regardless of heap size (up to 16 TB).
- Concurrent relocation using colored pointers (metadata in pointer bits).
- Almost all GC work is concurrent — only brief pauses for root scanning.
- Tuning: `-XX:+UseZGC -XX:+ZGenerational` (Java 21+: generational ZGC).

**Interview Tip:** For 2026, know that Generational ZGC (Java 21) is the future default for latency-sensitive apps. Mention that it became the default in Java 23.

---

### Q72. How does the String Pool work? Explain `intern()`.

**Concept:** The String pool (part of the Heap since Java 7) stores unique `String` literals. When the compiler encounters a string literal, it checks the pool first. `intern()` manually adds a string to the pool and returns the pooled reference.

```java
String s1 = "hello";                // goes to pool
String s2 = "hello";                // reuses pool entry
String s3 = new String("hello");    // new object on heap (also puts "hello" in pool)
String s4 = s3.intern();            // returns pool reference

System.out.println(s1 == s2);       // true  (same pool ref)
System.out.println(s1 == s3);       // false (different objects)
System.out.println(s1 == s4);       // true  (intern returns pool ref)
```

**Java 9 Compact Strings:**
- Strings internally use `byte[]` instead of `char[]`.
- Latin-1 characters use 1 byte per char; others use 2 bytes (UTF-16).
- Flag `coder` indicates encoding: LATIN1 (0) or UTF16 (1).
- Transparent to API — all `String` methods work identically.

**Interview Tip:** Mention that `intern()` can cause memory leaks if overused (pooled strings are not easily GC'd). Also mention compact strings as a Java 9 optimization — it halves memory for ASCII-heavy applications.

---

### Q73. What are memory leaks in Java? Common patterns and prevention.

**Concept:** Although Java has GC, memory leaks occur when objects are no longer needed but still referenced, preventing GC from collecting them. Common causes: static collections, unclosed resources, listeners not deregistered, and inner class references.

```java
// ❌ Leak 1: Static collection that only grows
class Cache {
    static final Map<String, Object> cache = new HashMap<>();
    static void add(String key, Object val) { cache.put(key, val); }
    // Never evicted! Grows forever.
}

// ✅ Fix: Use WeakHashMap, Caffeine cache, or bounded cache
Map<String, Object> weakCache = new WeakHashMap<>(); // entries GC'd when key unreachable

// ❌ Leak 2: Unclosed resources
void leak() {
    Connection conn = DriverManager.getConnection(url); // never closed!
}
// ✅ Fix: try-with-resources
try (var conn = DriverManager.getConnection(url)) { /* work */ }

// ❌ Leak 3: Inner class holds reference to outer class
class Outer {
    byte[] largeData = new byte[10_000_000];
    class Inner { /* implicitly holds reference to Outer → largeData not GC'd */ }
}
// ✅ Fix: Use static inner class
class Outer {
    byte[] largeData = new byte[10_000_000];
    static class Inner { /* no reference to Outer */ }
}

// ❌ Leak 4: ThreadLocal not cleaned up
ThreadLocal<byte[]> tl = new ThreadLocal<>();
tl.set(new byte[1_000_000]);
// If thread lives in a pool, this is never GC'd!
// ✅ Fix: Always call tl.remove() in a finally block
```

**Interview Tip:** Memory leaks are a senior-level topic. Mention tools for detection: `jmap`, `jhat`, Eclipse MAT, VisualVM. Show you think about `WeakReference`, `SoftReference`, and `PhantomReference`.

---

### Q74. Explain ClassLoader hierarchy and delegation model.

**Concept:** ClassLoaders load classes from bytecode into the JVM. They follow a parent-delegation model: a classloader first delegates to its parent before attempting to load the class itself. This ensures core Java classes (loaded by Bootstrap) are never overridden.

```
Bootstrap ClassLoader (C/C++, loads java.base: java.lang.*, java.util.*, etc.)
    ↓ delegates up
Platform ClassLoader (loads java.sql, java.xml, etc.)
    ↓ delegates up
Application ClassLoader (loads classpath: your application classes)
    ↓ delegates up
Custom ClassLoader (optional: for plugins, hot-reloading, etc.)
```

| ClassLoader | Loads | Parent |
|-------------|-------|--------|
| Bootstrap | Core Java (`java.lang`, `java.util`) | None (null) |
| Platform (Extension) | Platform modules (`java.sql`) | Bootstrap |
| Application (System) | Classpath (`-cp`) | Platform |
| Custom | Defined by developer | Application (usually) |

**Interview Tip:** Explain why `String.class.getClassLoader()` returns `null` — `String` is loaded by Bootstrap, which is not a Java object. Know that module system (Java 9) changed the classloader hierarchy — there are only 3 built-in classloaders now.

---

### Q75. What is the JIT Compiler? How does it optimize code?

**Concept:** The JIT (Just-In-Time) compiler converts frequently executed bytecode ("hot spots") into native machine code at runtime. It profiles the running application and applies aggressive optimizations that static compilers cannot (because they know the actual execution patterns).

**Key JIT Optimizations:**

| Optimization | What it does |
|-------------|-------------|
| Method inlining | Replaces method call with method body |
| Loop unrolling | Expands loop iterations to reduce branch overhead |
| Dead code elimination | Removes unreachable or unused code |
| Escape analysis | Allocates objects on stack if they don't "escape" the method |
| Lock elision | Removes synchronization on thread-local objects |
| Constant folding | Evaluates constant expressions at compile time |
| Tiered compilation | Interprets → C1 (quick compile) → C2 (optimized compile) |

```java
// Escape analysis example — JIT may stack-allocate this
void process() {
    Point p = new Point(1, 2);  // p doesn't escape method
    int sum = p.x + p.y;       // JIT may eliminate object creation entirely
}
```

**Interview Tip:** Mention tiered compilation (default since Java 8): code starts interpreted, then C1 compiles it quickly, then C2 optimizes hot methods aggressively. Know that GraalVM offers an alternative JIT.

---

### Q76. What are `WeakReference`, `SoftReference`, and `PhantomReference`?

**Concept:** These are reference types in `java.lang.ref` that interact with the GC differently. Strong references (normal) prevent GC. Soft references are cleared only under memory pressure. Weak references are cleared at the next GC cycle. Phantom references are used for post-mortem cleanup.

| Reference Type | GC Behavior | Use Case |
|---------------|-------------|----------|
| Strong (`T ref = obj`) | Never collected while reachable | Default |
| `SoftReference<T>` | Collected under memory pressure | Memory-sensitive caches |
| `WeakReference<T>` | Collected at next GC | Canonicalized mappings, WeakHashMap |
| `PhantomReference<T>` | Enqueued after finalization | Resource cleanup (Cleaner API) |

```java
// WeakReference — collected at next GC
WeakReference<BigObject> weak = new WeakReference<>(new BigObject());
System.out.println(weak.get()); // BigObject
System.gc();
System.out.println(weak.get()); // null (likely)

// SoftReference — collected only when JVM needs memory
SoftReference<byte[]> cache = new SoftReference<>(new byte[10_000_000]);
// Stays alive as long as memory is available

// WeakHashMap — entries removed when key is unreachable
Map<Key, Value> map = new WeakHashMap<>();
```

**Interview Tip:** `WeakHashMap` is used in `ThreadLocalMap` internally. `Cleaner` (Java 9) replaces `finalize()` using `PhantomReference` for deterministic cleanup.

---

### Q77. How does `finalize()` work? Why was it deprecated?

**Concept:** `finalize()` was called by the GC before reclaiming an object, intended for cleanup. It was deprecated in Java 9 and marked for removal because: unpredictable timing, performance cost (requires two GC cycles), can resurrect objects, and doesn't guarantee execution.

```java
// ❌ DEPRECATED — don't use
class Resource {
    @Override
    protected void finalize() throws Throwable {
        try { cleanup(); } finally { super.finalize(); }
    }
}

// ✅ MODERN — use Cleaner (Java 9+)
class Resource implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    private final Cleaner.Cleanable cleanable;

    Resource() {
        State state = new State(/* native resource */);
        this.cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() { cleanable.clean(); }

    // Must be static to avoid preventing GC of Resource
    private static class State implements Runnable {
        @Override public void run() { /* release native resource */ }
    }
}
```

**Interview Tip:** Know the replacement chain: `finalize()` → `Cleaner` API (Java 9) → try-with-resources (preferred). Mention that `finalize()` is marked `@Deprecated(forRemoval = true)` since Java 18.

---

### Q78. What are Compact Strings and Compressed Oops?

**Concept:** Compact Strings (Java 9) store Latin-1 strings as `byte[]` with 1 byte per char instead of 2, halving memory for ASCII-heavy applications. Compressed Oops (Ordinary Object Pointers) use 32-bit references on 64-bit JVMs for heap sizes < 32 GB, reducing pointer overhead.

```java
// Compact Strings — automatic, transparent
String ascii = "Hello";     // stored as byte[] with LATIN1 encoding (1 byte/char)
String unicode = "こんにちは"; // stored as byte[] with UTF16 encoding (2 bytes/char)
// API is identical — optimization is internal
```

| Feature | Effect | Enabled By Default? |
|---------|--------|:-------------------:|
| Compact Strings | Halves memory for ASCII strings | ✅ (Java 9+) |
| Compressed Oops | 32-bit refs on 64-bit JVM (heap < 32 GB) | ✅ |
| Compressed Class Pointers | 32-bit class metadata pointers | ✅ |

**Interview Tip:** Compact Strings are a free win — no code changes needed. Mention that this is why Java 9+ `String` has a `coder` field (LATIN1 or UTF16) alongside the `value` byte array.

---

# Section 7 — Design Patterns (Q79–Q87)

---

### Q79. Implement Singleton — all thread-safe variations.

**Concept:** Singleton ensures a class has only one instance with global access. Thread-safe implementations must handle concurrent `getInstance()` calls. The best modern approach is using an `enum`.

```java
// 1. Enum Singleton (BEST — Joshua Bloch recommended)
public enum ConfigManager {
    INSTANCE;
    private final Map<String, String> config = new HashMap<>();
    public String get(String key) { return config.get(key); }
    public void set(String key, String value) { config.put(key, value); }
}
// Usage: ConfigManager.INSTANCE.get("key");

// 2. Bill Pugh (Initialization-on-Demand Holder)
public class Singleton {
    private Singleton() {}
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() { return Holder.INSTANCE; }
}

// 3. Double-Checked Locking (volatile required!)
public class DCLSingleton {
    private static volatile DCLSingleton instance;
    private DCLSingleton() {}
    public static DCLSingleton getInstance() {
        if (instance == null) {
            synchronized (DCLSingleton.class) {
                if (instance == null) {
                    instance = new DCLSingleton();
                }
            }
        }
        return instance;
    }
}
```

| Approach | Thread-safe | Lazy | Serialization-safe | Reflection-safe |
|----------|:-----------:|:----:|:------------------:|:--------------:|
| Enum | ✅ | ❌ (eager) | ✅ | ✅ |
| Bill Pugh | ✅ | ✅ | ❌ (needs `readResolve`) | ❌ |
| DCL | ✅ | ✅ | ❌ | ❌ |
| `synchronized` method | ✅ | ✅ | ❌ | ❌ |

**Interview Tip:** Always say "enum singleton" first. Then discuss DCL and Bill Pugh as alternatives. Mention that enum singletons are immune to reflection attacks (JVM prevents enum instantiation via reflection).

---

### Q80. Explain the Factory Method and Abstract Factory patterns.

**Concept:** Factory Method defines an interface for creating objects, letting subclasses decide which class to instantiate. Abstract Factory provides a family of related objects without specifying concrete classes.

```java
// Factory Method — using sealed interfaces (Java 17+)
sealed interface Notification permits EmailNotification, SMSNotification, PushNotification {}
record EmailNotification(String to, String subject) implements Notification {}
record SMSNotification(String phone, String message) implements Notification {}
record PushNotification(String token, String payload) implements Notification {}

// Modern factory using pattern matching
class NotificationFactory {
    static Notification create(String type, Map<String, String> params) {
        return switch (type) {
            case "email" -> new EmailNotification(params.get("to"), params.get("subject"));
            case "sms"   -> new SMSNotification(params.get("phone"), params.get("message"));
            case "push"  -> new PushNotification(params.get("token"), params.get("payload"));
            default      -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}
```

**Interview Tip:** Show the modern Java 17+ approach with sealed interfaces and records. It's cleaner than the traditional abstract class hierarchy.

---

### Q81. Explain the Builder pattern with modern Java.

**Concept:** Builder separates object construction from representation, allowing step-by-step creation of complex objects. With Java Records, you can combine the Builder pattern with immutable records for the best of both worlds.

```java
// Builder for complex immutable object
public record HttpRequest(
    String method, String url, Map<String, String> headers,
    String body, Duration timeout, boolean followRedirects
) {
    // Private constructor enforces builder usage
    public static Builder builder(String method, String url) {
        return new Builder(method, url);
    }

    public static class Builder {
        private final String method, url;
        private final Map<String, String> headers = new LinkedHashMap<>();
        private String body;
        private Duration timeout = Duration.ofSeconds(30);
        private boolean followRedirects = true;

        private Builder(String method, String url) {
            this.method = method;
            this.url = url;
        }

        public Builder header(String k, String v) { headers.put(k, v); return this; }
        public Builder body(String body)           { this.body = body; return this; }
        public Builder timeout(Duration t)         { this.timeout = t; return this; }
        public Builder followRedirects(boolean f)  { this.followRedirects = f; return this; }

        public HttpRequest build() {
            return new HttpRequest(method, url, Map.copyOf(headers), body, timeout, followRedirects);
        }
    }
}

// Usage
var req = HttpRequest.builder("POST", "https://api.example.com")
    .header("Content-Type", "application/json")
    .body("{\"key\": \"value\"}")
    .timeout(Duration.ofSeconds(10))
    .build();
```

**Interview Tip:** Combine Builder with Records for immutable result objects. Mention that Lombok's `@Builder` generates this boilerplate.

---

### Q82. Explain the Strategy pattern with lambdas.

**Concept:** Strategy encapsulates interchangeable algorithms behind a common interface. With Java 8+ lambdas, the strategy interface becomes a functional interface, and strategies become lambdas — no need for separate classes.

```java
// Strategy as a functional interface
@FunctionalInterface
interface PricingStrategy {
    double calculatePrice(double basePrice, int quantity);
}

class OrderProcessor {
    private final PricingStrategy strategy;

    OrderProcessor(PricingStrategy strategy) { this.strategy = strategy; }

    double processOrder(double basePrice, int qty) {
        return strategy.calculatePrice(basePrice, qty);
    }
}

// Lambda strategies — no classes needed!
PricingStrategy regular  = (price, qty) -> price * qty;
PricingStrategy bulk     = (price, qty) -> price * qty * (qty > 100 ? 0.8 : 0.9);
PricingStrategy premium  = (price, qty) -> price * qty * 1.1 + 50; // premium service fee

var order = new OrderProcessor(bulk);
System.out.println(order.processOrder(10.0, 150)); // 1200.0 (20% discount)
```

**Interview Tip:** Show that lambdas eliminate the need for separate strategy classes — the `java.util.function` package is essentially a built-in strategy framework.

---

### Q83. Explain the Observer pattern (and modern alternatives).

**Concept:** Observer defines a one-to-many dependency: when one object (subject) changes, all dependents (observers) are notified. Modern Java replaces the classic pattern with `Flow` API (reactive streams, Java 9+), `PropertyChangeListener`, or framework-specific events.

```java
// Modern approach — java.util.concurrent.Flow (Java 9+)
// Publisher → Subscriber with backpressure

// Simple implementation using functional interfaces
class EventBus<T> {
    private final List<Consumer<T>> subscribers = new CopyOnWriteArrayList<>();

    void subscribe(Consumer<T> handler) { subscribers.add(handler); }
    void unsubscribe(Consumer<T> handler) { subscribers.remove(handler); }

    void publish(T event) {
        subscribers.forEach(s -> s.accept(event));
    }
}

// Usage
EventBus<String> bus = new EventBus<>();
bus.subscribe(msg -> System.out.println("Logger: " + msg));
bus.subscribe(msg -> System.out.println("Analytics: " + msg));
bus.publish("User logged in");
// Logger: User logged in
// Analytics: User logged in
```

**Interview Tip:** Mention that `java.util.Observable` was deprecated in Java 9. The modern alternatives are `Flow` API, application event systems (Spring Events), or reactive libraries (Project Reactor, RxJava).

---

### Q84. Explain the Decorator pattern.

**Concept:** Decorator dynamically adds behavior to objects without modifying their class, by wrapping the original in a decorator that implements the same interface. Java I/O streams are the classic example.

```java
// Interface
interface DataSource {
    String read();
    void write(String data);
}

// Base implementation
record FileDataSource(String filename) implements DataSource {
    public String read() { return "raw data from " + filename; }
    public void write(String data) { /* write to file */ }
}

// Decorators
class EncryptionDecorator implements DataSource {
    private final DataSource wrapped;
    EncryptionDecorator(DataSource wrapped) { this.wrapped = wrapped; }

    public String read() { return decrypt(wrapped.read()); }
    public void write(String data) { wrapped.write(encrypt(data)); }

    private String encrypt(String s) { return "ENC(" + s + ")"; }
    private String decrypt(String s) { return s.replace("ENC(", "").replace(")", ""); }
}

class CompressionDecorator implements DataSource {
    private final DataSource wrapped;
    CompressionDecorator(DataSource wrapped) { this.wrapped = wrapped; }

    public String read() { return decompress(wrapped.read()); }
    public void write(String data) { wrapped.write(compress(data)); }

    private String compress(String s) { return "ZIP(" + s + ")"; }
    private String decompress(String s) { return s.replace("ZIP(", "").replace(")", ""); }
}

// Stack decorators
DataSource source = new CompressionDecorator(
    new EncryptionDecorator(
        new FileDataSource("data.txt")
    )
);
source.write("Hello"); // writes ZIP(ENC(Hello)) to file
```

**Interview Tip:** Point to `java.io` as the textbook decorator: `BufferedReader(new InputStreamReader(new FileInputStream("f")))`. Each layer adds functionality.

---

### Q85. Explain the Prototype pattern.

**Concept:** Prototype creates new objects by cloning an existing prototype instance instead of constructing from scratch. Useful when object creation is expensive or complex.

```java
// Using Cloneable (traditional approach)
public class Configuration implements Cloneable {
    private Map<String, String> settings;
    private List<String> features;

    public Configuration(Map<String, String> settings, List<String> features) {
        this.settings = new HashMap<>(settings);
        this.features = new ArrayList<>(features);
    }

    @Override
    public Configuration clone() {
        try {
            Configuration copy = (Configuration) super.clone();
            copy.settings = new HashMap<>(this.settings);  // deep copy
            copy.features = new ArrayList<>(this.features); // deep copy
            return copy;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // can't happen
        }
    }
}

// Modern approach — copy constructor or Records with wither methods
record Config(String env, int timeout, List<String> features) {
    Config withTimeout(int newTimeout) {
        return new Config(env, newTimeout, features);
    }
    Config withEnv(String newEnv) {
        return new Config(newEnv, timeout, features);
    }
}
```

**Interview Tip:** The modern approach favors copy constructors or "with" methods on Records over `Cloneable` (which Joshua Bloch called "broken"). Mention that Records naturally support this pattern.

---

### Q86. Explain the Template Method pattern.

**Concept:** Template Method defines the skeleton of an algorithm in a base class, deferring specific steps to subclasses. The base class controls the overall flow; subclasses fill in the details.

```java
abstract class DataProcessor {
    // Template method — final to prevent override of the flow
    public final void process() {
        readData();
        transformData();
        if (shouldValidate()) {  // hook method
            validateData();
        }
        writeData();
    }

    abstract void readData();
    abstract void transformData();
    abstract void writeData();

    // Hook — default behavior, subclass can override
    boolean shouldValidate() { return true; }
    void validateData() { System.out.println("Default validation"); }
}

class CSVProcessor extends DataProcessor {
    void readData()      { System.out.println("Read CSV"); }
    void transformData() { System.out.println("Parse CSV rows"); }
    void writeData()     { System.out.println("Write to DB"); }
}
```

**Interview Tip:** Identify `HttpServlet.service()` as a real-world Template Method — it dispatches to `doGet()`, `doPost()`, etc.

---

### Q87. Modern Java pattern: Sealed interfaces + Records as Algebraic Data Types.

**Concept:** Combining sealed interfaces with records creates discriminated unions / algebraic data types (ADTs) in Java. This replaces the Visitor pattern for type-safe, exhaustive processing of closed type hierarchies.

```java
// ADT for a Result type (like Rust's Result<T, E>)
public sealed interface Result<T> {
    record Success<T>(T value) implements Result<T> {}
    record Failure<T>(String error, Exception cause) implements Result<T> {}
}

// Usage with pattern matching
static <T> String describe(Result<T> result) {
    return switch (result) {
        case Result.Success<T>(var value) -> "OK: " + value;
        case Result.Failure<T>(var error, var cause) ->
            "FAIL: " + error + (cause != null ? " [" + cause.getMessage() + "]" : "");
    };
}

// ADT for a JSON value
sealed interface JsonValue {
    record JsonString(String value)            implements JsonValue {}
    record JsonNumber(double value)            implements JsonValue {}
    record JsonBool(boolean value)             implements JsonValue {}
    record JsonArray(List<JsonValue> elements)  implements JsonValue {}
    record JsonObject(Map<String, JsonValue> fields) implements JsonValue {}
    record JsonNull()                          implements JsonValue {}
}

static String toJson(JsonValue value) {
    return switch (value) {
        case JsonString(var s)   -> "\"" + s + "\"";
        case JsonNumber(var n)   -> String.valueOf(n);
        case JsonBool(var b)     -> String.valueOf(b);
        case JsonNull()          -> "null";
        case JsonArray(var elems) -> elems.stream().map(e -> toJson(e))
                .collect(Collectors.joining(", ", "[", "]"));
        case JsonObject(var fields) -> fields.entrySet().stream()
                .map(e -> "\"" + e.getKey() + "\": " + toJson(e.getValue()))
                .collect(Collectors.joining(", ", "{", "}"));
    };
}
```

**Interview Tip:** This is the most modern pattern in Java 17/21. It replaces Visitor, instanceof chains, and enum-based dispatch with compiler-verified exhaustive matching. This is the "killer feature" to demonstrate in 2026 interviews.

---

# Section 8 — Coding Problems (Q88–Q100)

---

### Q88. Reverse a String — multiple approaches.

```java
public class StringReverse {
    // Approach 1: StringBuilder
    static String reverse1(String s) {
        return new StringBuilder(s).reverse().toString();
    }

    // Approach 2: Two pointers (char array)
    static String reverse2(String s) {
        char[] arr = s.toCharArray();
        int left = 0, right = arr.length - 1;
        while (left < right) {
            char temp = arr[left];
            arr[left++] = arr[right];
            arr[right--] = temp;
        }
        return new String(arr);
    }

    // Approach 3: Stream (functional)
    static String reverse3(String s) {
        return new StringBuilder(s).reverse().toString();
        // Pure stream approach (inefficient, just for demonstration):
        // IntStream.range(0, s.length())
        //     .mapToObj(i -> String.valueOf(s.charAt(s.length() - 1 - i)))
        //     .collect(Collectors.joining());
    }
}
```

**Time:** O(n) | **Space:** O(n) for new string

**Edge Cases:** Empty string, single char, palindrome, Unicode surrogate pairs (StringBuilder.reverse handles them correctly).

**Interview Tip:** Use `StringBuilder.reverse()` for production code. Show the two-pointer approach for algorithmic interviews.

---

### Q89. Check if a String is a palindrome.

```java
static boolean isPalindrome(String s) {
    // Clean: lowercase, remove non-alphanumeric
    String cleaned = s.toLowerCase().replaceAll("[^a-z0-9]", "");
    int left = 0, right = cleaned.length() - 1;
    while (left < right) {
        if (cleaned.charAt(left++) != cleaned.charAt(right--)) return false;
    }
    return true;
}

// Test
System.out.println(isPalindrome("A man, a plan, a canal: Panama")); // true
System.out.println(isPalindrome("race a car"));                     // false
```

**Time:** O(n) | **Space:** O(n) for cleaned string

**Edge Cases:** Empty string (true), single char (true), mixed case, special characters.

**Interview Tip:** Mention that you can do it without creating a cleaned string by using two pointers that skip non-alphanumeric characters — O(1) extra space.

---

### Q90. Check if two strings are anagrams.

```java
// Approach 1: Frequency count (optimal)
static boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] freq = new int[26]; // assuming lowercase a-z
    for (char c : s.toCharArray()) freq[c - 'a']++;
    for (char c : t.toCharArray()) freq[c - 'a']--;
    for (int f : freq) if (f != 0) return false;
    return true;
}

// Approach 2: Sorting (simpler but slower)
static boolean isAnagram2(String s, String t) {
    if (s.length() != t.length()) return false;
    char[] a = s.toCharArray(), b = t.toCharArray();
    Arrays.sort(a);
    Arrays.sort(b);
    return Arrays.equals(a, b);
}

// Test
System.out.println(isAnagram("listen", "silent")); // true
System.out.println(isAnagram("hello", "world"));   // false
```

**Time:** O(n) frequency | O(n log n) sorting | **Space:** O(1) frequency (fixed array) | O(n) sorting

**Edge Cases:** Empty strings (true), different lengths (false), Unicode characters (use HashMap instead of array).

**Interview Tip:** Use the frequency-count approach for O(n). If Unicode is required, use `HashMap<Character, Integer>`.

---

### Q91. Find the first non-repeating character in a String.

```java
static char firstNonRepeating(String s) {
    // LinkedHashMap preserves insertion order
    Map<Character, Integer> freq = new LinkedHashMap<>();
    for (char c : s.toCharArray()) {
        freq.merge(c, 1, Integer::sum);
    }
    return freq.entrySet().stream()
        .filter(e -> e.getValue() == 1)
        .map(Map.Entry::getKey)
        .findFirst()
        .orElseThrow(() -> new NoSuchElementException("All characters repeat"));
}

// One-pass alternative using index array
static int firstUniqChar(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;
    for (int i = 0; i < s.length(); i++) {
        if (freq[s.charAt(i) - 'a'] == 1) return i;
    }
    return -1;
}

System.out.println(firstNonRepeating("aabbcdd")); // 'c'
```

**Time:** O(n) | **Space:** O(1) (bounded by alphabet size)

---

### Q92. Two Sum — find indices of two numbers that add to target.

```java
static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>(); // value → index
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[]{ seen.get(complement), i };
        }
        seen.put(nums[i], i);
    }
    throw new IllegalArgumentException("No solution");
}

// Test
System.out.println(Arrays.toString(twoSum(new int[]{2, 7, 11, 15}, 9))); // [0, 1]
```

**Time:** O(n) | **Space:** O(n) for HashMap

**Edge Cases:** Duplicate values, negative numbers, no solution, single element.

**Interview Tip:** This is THE most common interview question. Know the HashMap approach cold. Mention that if the array is sorted, you can use two pointers for O(1) space.

---

### Q93. Sliding Window — Maximum sum subarray of size K.

```java
static int maxSumSubarray(int[] arr, int k) {
    if (arr.length < k) throw new IllegalArgumentException("Array too small");

    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += arr[i]; // first window

    int maxSum = windowSum;
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k]; // slide: add right, remove left
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}

// Test
System.out.println(maxSumSubarray(new int[]{2, 1, 5, 1, 3, 2}, 3)); // 9 (5+1+3)
```

**Time:** O(n) | **Space:** O(1)

**Edge Cases:** k = 1, k = array length, all negative numbers.

**Interview Tip:** Sliding window is a fundamental pattern. Know both fixed-size (this problem) and variable-size (e.g., longest substring without repeating).

---

### Q94. Reverse a Linked List — iterative and recursive.

```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

// Iterative — O(n) time, O(1) space
static ListNode reverseIterative(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;  // save next
        curr.next = prev;           // reverse pointer
        prev = curr;                // advance prev
        curr = next;                // advance curr
    }
    return prev; // new head
}

// Recursive — O(n) time, O(n) stack space
static ListNode reverseRecursive(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverseRecursive(head.next);
    head.next.next = head; // reverse the pointer
    head.next = null;      // break forward link
    return newHead;
}
```

**Time:** O(n) both | **Space:** O(1) iterative, O(n) recursive (call stack)

**Interview Tip:** Be able to trace through both approaches on a whiteboard with a 3-4 node example. The iterative approach is preferred for production (no stack overflow risk).

---

### Q95. Detect a cycle in a LinkedList (Floyd's Tortoise and Hare).

```java
static boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;           // 1 step
        fast = fast.next.next;      // 2 steps
        if (slow == fast) return true; // they meet → cycle exists
    }
    return false; // fast reached end → no cycle
}

// Find the cycle start node
static ListNode findCycleStart(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            slow = head; // reset slow to head
            while (slow != fast) { // move both at same speed
                slow = slow.next;
                fast = fast.next;
            }
            return slow; // meeting point is cycle start
        }
    }
    return null;
}
```

**Time:** O(n) | **Space:** O(1)

**Interview Tip:** Explain WHY Floyd's algorithm works: when slow has traveled distance `d`, fast has traveled `2d`. If there's a cycle of length `c`, they'll meet when `2d - d = nc` → `d = nc`. For finding the start, explain the mathematical proof if asked.

---

### Q96. Binary Tree Inorder, Preorder, Postorder traversals.

```java
class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}

// Recursive
static void inorder(TreeNode node, List<Integer> result) {
    if (node == null) return;
    inorder(node.left, result);
    result.add(node.val);        // Left → Root → Right
    inorder(node.right, result);
}

static void preorder(TreeNode node, List<Integer> result) {
    if (node == null) return;
    result.add(node.val);         // Root → Left → Right
    preorder(node.left, result);
    preorder(node.right, result);
}

static void postorder(TreeNode node, List<Integer> result) {
    if (node == null) return;
    postorder(node.left, result);
    postorder(node.right, result);
    result.add(node.val);         // Left → Right → Root
}

// Iterative inorder using stack
static List<Integer> inorderIterative(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();
        result.add(curr.val);
        curr = curr.right;
    }
    return result;
}
```

**Time:** O(n) | **Space:** O(h) where h = tree height (O(n) worst case, O(log n) balanced)

**Memory Aid:** **I**norder = **I**ncreasing (for BST). **Pre**order = **Pre**fix (root first). **Post**order = **Post**mortem (root last, used for deletion).

---

### Q97. Level-order traversal (BFS) of a Binary Tree.

```java
static List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null)  queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}

// Example tree:     1
//                 /   \
//                2     3
//               / \
//              4   5
// Output: [[1], [2, 3], [4, 5]]
```

**Time:** O(n) | **Space:** O(n) for the queue

---

### Q98. Find duplicates using HashMap and Streams.

```java
// Approach 1: HashMap frequency count
static List<Integer> findDuplicates(int[] nums) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);
    return freq.entrySet().stream()
        .filter(e -> e.getValue() > 1)
        .map(Map.Entry::getKey)
        .toList();
}

// Approach 2: Set — seen before?
static List<Integer> findDuplicates2(int[] nums) {
    Set<Integer> seen = new HashSet<>();
    Set<Integer> duplicates = new LinkedHashSet<>();
    for (int n : nums) {
        if (!seen.add(n)) duplicates.add(n);
    }
    return new ArrayList<>(duplicates);
}

// Approach 3: Pure Stream
static List<Integer> findDuplicates3(int[] nums) {
    return Arrays.stream(nums)
        .boxed()
        .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
        .entrySet().stream()
        .filter(e -> e.getValue() > 1)
        .map(Map.Entry::getKey)
        .toList();
}

System.out.println(findDuplicates(new int[]{1, 2, 3, 2, 4, 3, 5})); // [2, 3]
```

**Time:** O(n) all approaches | **Space:** O(n)

**Interview Tip:** Show all three approaches. The Set approach (2) is the cleanest and most efficient for just finding duplicates.

---

### Q99. Group Anagrams together using Streams and HashMap.

```java
static Map<String, List<String>> groupAnagrams(List<String> words) {
    return words.stream()
        .collect(Collectors.groupingBy(word -> {
            char[] chars = word.toLowerCase().toCharArray();
            Arrays.sort(chars);
            return new String(chars); // sorted key: "eat" → "aet"
        }));
}

// Test
var result = groupAnagrams(List.of("eat", "tea", "tan", "ate", "nat", "bat"));
// {"aet": ["eat", "tea", "ate"], "ant": ["tan", "nat"], "abt": ["bat"]}
result.forEach((k, v) -> System.out.println(k + " → " + v));
```

**Time:** O(n × k log k) where n = number of words, k = max word length | **Space:** O(n × k)

**Edge Cases:** Empty strings, single character strings, case sensitivity.

**Interview Tip:** The key insight is the canonical form: sorted characters as the grouping key. Alternative: use character frequency as key (avoids sorting, O(n × k) time).

---

### Q100. Solve FizzBuzz with Streams — simple but elegant.

```java
// Classic
static List<String> fizzBuzz(int n) {
    return IntStream.rangeClosed(1, n)
        .mapToObj(i -> {
            if (i % 15 == 0) return "FizzBuzz";
            if (i % 3 == 0)  return "Fizz";
            if (i % 5 == 0)  return "Buzz";
            return String.valueOf(i);
        })
        .toList();
}

// Pattern matching approach (Java 21+)
static String classify(int n) {
    return switch (n % 15) {
        case 0                          -> "FizzBuzz";
        case 3, 6, 9, 12               -> "Fizz";
        case 5, 10                      -> "Buzz";
        default                         -> String.valueOf(n);
    };
}

fizzBuzz(15).forEach(System.out::println);
```

**Interview Tip:** FizzBuzz is a warm-up question. Write it quickly and correctly. Using streams shows modern Java fluency. The pattern matching switch is a nice Java 21 touch.

---

# Quick Reference Cheat Sheet — 100 Questions in One Liner Each

| # | Topic | One-Liner |
|---|-------|-----------|
| **Q1** | OOP Pillars | Encapsulation (data hiding), Abstraction (hide complexity), Inheritance (reuse), Polymorphism (one interface, many forms) |
| **Q2** | Access Modifiers | private < default(package) < protected(package+subclass) < public; "Please Don't Punch People" |
| **Q3** | this vs super | `this` = current object/constructor; `super` = parent class/constructor; both must be first line in constructor |
| **Q4** | static vs instance | Static = class-level (Method Area), resolved at compile time; Instance = object-level (Heap), resolved at runtime |
| **Q5** | Immutability | Final class + private final fields + no setters + defensive copies in/out; Records are immutable by default |
| **Q6** | final keyword | Variables (no reassign), methods (no override), classes (no extend); JMM guarantees visibility of final fields after construction |
| **Q7** | == vs equals | `==` compares references; `.equals()` compares content; Integer cache: -128 to 127; always override hashCode with equals |
| **Q8** | Overload vs Override | Overloading = compile-time (different params); Overriding = runtime (vtable dispatch, same signature, covariant return OK) |
| **Q9** | Abstract vs Interface | Abstract = state + constructors + partial impl; Interface = contracts + default/static/private methods + multiple inheritance |
| **Q10** | String/SB/SBuf | String = immutable + pool; StringBuilder = mutable + fast; StringBuffer = mutable + synchronized; use SB for concatenation |
| **Q11** | Wrapper/Autoboxing | Autoboxing uses `valueOf()` (cached -128 to 127); unboxing null → NPE; avoid in tight loops |
| **Q12** | hashCode/equals | Equal objects MUST have equal hashCodes; Records auto-generate both; use Objects.hash() for manual impl |
| **Q13** | Generics | Type erasure at compile time; PECS: Producer Extends, Consumer Super; no primitives, no generic arrays |
| **Q14** | Comparable/Comparator | Comparable = natural order inside class (`compareTo`); Comparator = external strategy (lambdas + chaining) |
| **Q15** | Records/Sealed | Records = immutable data + auto equals/hashCode/toString; Sealed = restricted hierarchy; together = ADTs |
| **Q16** | Pattern matching | Java 16: `instanceof` + cast combo; Java 21: record patterns, nested destructuring, guarded patterns |
| **Q17** | var keyword | Local type inference; no fields/params/returns; not a keyword (reserved type name); use when type is obvious |
| **Q18** | Collections hierarchy | Iterable→Collection→(List,Set,Queue); Map is separate; Java 21 adds SequencedCollection/Set/Map |
| **Q19** | ArrayList/LinkedList | ArrayList: O(1) get, O(n) insert; LinkedList: O(n) get, O(1) ends; always prefer ArrayList + ArrayDeque |
| **Q20** | HashMap internals | Array of buckets + linked list/red-black tree (J8+); treeify at 8 nodes + 64 capacity; hash perturbation: h^(h>>>16) |
| **Q21** | Map implementations | HashMap (unordered) vs LinkedHashMap (insertion/access order) vs TreeMap (sorted) vs ConcurrentHashMap (thread-safe) |
| **Q22** | Set implementations | HashSet (HashMap), LinkedHashSet (LinkedHashMap), TreeSet (TreeMap); order/null differences mirror maps |
| **Q23** | ConcurrentHashMap | CAS + per-bin synchronized (J8+); no null keys/values; atomic ops: compute, merge, putIfAbsent |
| **Q24** | Fail-fast/safe | Fail-fast: CME on modification (ArrayList); Fail-safe: snapshot/weakly consistent (ConcurrentHashMap, COW) |
| **Q25** | PQ/ArrayDeque | PriorityQueue = min-heap (poll O(log n)); ArrayDeque = circular array (best stack/queue, O(1) ops) |
| **Q26** | Unmodifiable lists | `unmodifiableList` = view (reflects changes); `List.of` = truly immutable; `List.copyOf` = defensive copy |
| **Q27** | Iterator/ListIterator | Iterator = forward + remove; ListIterator = bidirectional + set + add + index; List only |
| **Q28** | EnumSet/EnumMap | Bitmask (EnumSet) and ordinal-indexed array (EnumMap); vastly faster than Hash equivalents for enums |
| **Q29** | TreeMap/Red-Black | Self-balancing BST; O(log n) all ops; NavigableMap methods: ceiling, floor, headMap, subMap |
| **Q30** | synchronizedList/COW | synchronizedList = coarse lock (manual sync for iteration); COW = copy-on-write (read-heavy, write-rare) |
| **Q31** | Map.of/ofEntries | Java 9 factory methods; truly unmodifiable; no null; no duplicates; random iteration order |
| **Q32** | Thread lifecycle | NEW→RUNNABLE→(BLOCKED\|WAITING\|TIMED_WAITING)→TERMINATED; 6 states in Thread.State |
| **Q33** | Creating threads | Thread, Runnable, Callable+Future, Virtual Threads (J21); prefer executors or virtual threads |
| **Q34** | sync/volatile/atomic | synchronized = mutex+visibility; volatile = visibility only; Atomic = lock-free CAS ops |
| **Q35** | Executor Framework | Fixed, Cached, Single, Scheduled, WorkStealing pools; always use bounded queue + rejection policy |
| **Q36** | CompletableFuture | Async chaining: thenApply(map), thenCompose(flatMap), thenCombine(zip), exceptionally(recover) |
| **Q37** | Virtual Threads | Lightweight JVM-managed threads; millions OK; for I/O-bound; avoid synchronized (pins carrier); use ReentrantLock |
| **Q38** | Structured Concurrency | StructuredTaskScope (J21 preview): fork-join with guaranteed lifecycle; ShutdownOnFailure/Success |
| **Q39** | ReentrantLock | tryLock, timed lock, fairness, multiple conditions; doesn't pin virtual threads (unlike synchronized) |
| **Q40** | Deadlock | MHNC conditions; fix: consistent lock ordering, tryLock timeout; detect: jstack, ThreadMXBean |
| **Q41** | Race condition | Check-then-act non-atomic; fix: synchronized, AtomicInteger, CAS; volatile NOT enough for count++ |
| **Q42** | wait/notify | Must be in synchronized; always use while loop (spurious wakeup); modern: BlockingQueue |
| **Q43** | Latch/Barrier/Semaphore | CountDownLatch (one-shot), CyclicBarrier (reusable sync point), Semaphore (permit pool), Phaser (flexible) |
| **Q44** | ThreadLocal | Per-thread storage; MUST remove in finally; J21: ScopedValues (immutable, bounded, virtual-thread-friendly) |
| **Q45** | ForkJoinPool | Divide-and-conquer + work-stealing; RecursiveTask/Action; powers parallel streams |
| **Q46** | RW/StampedLock | ReadWriteLock = multiple readers OR one writer; StampedLock = adds optimistic reads (highest performance) |
| **Q47** | Exchanger/Phaser | Exchanger = two-thread data swap; Phaser = dynamic parties + multi-phase (replaces Latch+Barrier) |
| **Q48** | LongAdder | Distributed cells for low-contention counting; faster than AtomicLong under high contention; sum() aggregates |
| **Q49** | Lambdas/FI | Lambda = anonymous function for @FunctionalInterface; core: Predicate, Function, Consumer, Supplier |
| **Q50** | Stream API | Source→intermediate(lazy)→terminal(eager); filter/map/flatMap/sorted → collect/reduce/toList |
| **Q51** | Optional | Container for nullable return; map/flatMap/filter/orElse; never use for fields/params; never call get() raw |
| **Q52** | Method references | 4 kinds: static (Class::static), bound (obj::method), unbound (Class::method), constructor (Class::new) |
| **Q53** | Default/static methods | Default = interface evolution; static = utility on interface; private (J9) = code reuse in defaults |
| **Q54** | Sealed Classes | `sealed permits` restricts subclasses; + pattern switch = exhaustive matching; replaces Visitor pattern |
| **Q55** | Pattern switch (J21) | Type patterns + record patterns + guards (`when`) + null handling; exhaustive with sealed types |
| **Q56** | Text Blocks | `"""` multi-line strings; indentation stripped by closing `"""`; `.formatted()` for interpolation |
| **Q57** | Switch expressions | Arrow syntax (no fall-through); `yield` for blocks; exhaustive for enums; return values |
| **Q58** | JPMS (Modules) | `module-info.java`: requires, exports, opens; strong encapsulation at package level |
| **Q59** | Sequenced Collections | Java 21: getFirst/getLast/reversed on List, LinkedHashSet, SortedSet; SequencedMap: firstEntry/lastEntry |
| **Q60** | HTTP Client (J11) | Builder-based, immutable; sync (`send`) and async (`sendAsync`); HTTP/2 support; replaces HttpURLConnection |
| **Q61** | toList/mapMulti/Gatherers | `toList()` (J16, unmodifiable); `mapMulti` (imperative flatMap); Gatherers (J22+, custom intermediate ops) |
| **Q62** | Record patterns (J21) | Destructure records in instanceof/switch; nested patterns; unnamed variables `_` (J22) |
| **Q63** | Exception hierarchy | Throwable→(Error,Exception); Error+RuntimeException=unchecked; rest=checked |
| **Q64** | try-with-resources | Auto-closes AutoCloseable; suppressed exceptions; reverse closing order; J9: effectively final vars |
| **Q65** | Custom exceptions | Extend RuntimeException (unchecked, modern) or Exception (checked); include context fields + cause chain |
| **Q66** | Catch order | Specific before general (compile error otherwise); multi-catch: `catch (A \| B e)` — e is effectively final |
| **Q67** | finally edge cases | Always runs (except System.exit/JVM crash); return in finally overrides try's return; never throw from finally |
| **Q68** | Suppressed exceptions | try-with-resources: close() exception is suppressed; access via getSuppressed(); better than try-finally |
| **Q69** | Helpful NPE (J14) | Pinpoints exactly which reference was null in a chain; enabled by default since J15 |
| **Q70** | JVM Architecture | Heap (objects), Stack (per-thread frames), Metaspace (class metadata, replaces PermGen since J8), PC Register |
| **Q71** | Garbage Collection | G1 (default, region-based), ZGC (<1ms pause, TB-scale), Shenandoah (<10ms); generational: Young→Old |
| **Q72** | String pool/intern | Pool in Heap (since J7); literals auto-interned; `intern()` returns pool ref; Compact Strings (J9): byte[] + coder |
| **Q73** | Memory leaks | Static collections, unclosed resources, non-static inner classes, ThreadLocal; fix: weak refs, TWR, static inner |
| **Q74** | ClassLoader | Bootstrap→Platform→Application; parent-delegation model; class loaded once per classloader |
| **Q75** | JIT Compiler | Tiered: interpret→C1→C2; optimizations: inlining, escape analysis, lock elision, dead code elimination |
| **Q76** | Weak/Soft/Phantom refs | Strong>Soft (memory pressure)>Weak (next GC)>Phantom (post-mortem); WeakHashMap, Cleaner API |
| **Q77** | finalize deprecation | Unpredictable, slow, can resurrect objects; use Cleaner (J9+) or try-with-resources; deprecated forRemoval |
| **Q78** | Compact Strings/Oops | Compact Strings: byte[] with LATIN1/UTF16 flag; Compressed Oops: 32-bit refs on 64-bit JVM for <32GB heap |
| **Q79** | Singleton | Enum (best), Bill Pugh (lazy), DCL (volatile!); enum is reflection+serialization safe |
| **Q80** | Factory | Factory Method = subclass decides; Abstract Factory = family of related objects; use sealed+records for modern |
| **Q81** | Builder | Step-by-step complex object construction; combine with Records for immutable results; fluent API |
| **Q82** | Strategy | Interchangeable algorithms via functional interface; lambdas replace separate strategy classes |
| **Q83** | Observer | One-to-many notification; modern: Flow API (J9), event bus, reactive streams; Observable deprecated J9 |
| **Q84** | Decorator | Wrap objects to add behavior; same interface; java.io streams are classic example |
| **Q85** | Prototype | Clone/copy instead of construct; modern: copy constructors or Record "with" methods; avoid Cloneable |
| **Q86** | Template Method | Base class defines algorithm skeleton; subclasses fill steps; HttpServlet.service() is real-world example |
| **Q87** | Sealed + Records ADT | Sealed interface + records = algebraic data types; exhaustive pattern switch replaces Visitor; KILLER pattern for 2026 |
| **Q88** | Reverse String | StringBuilder.reverse() or two-pointer swap on char[]; O(n) time, O(n) space |
| **Q89** | Palindrome | Two-pointer from ends inward after cleaning; O(n) time; handle case + non-alphanum |
| **Q90** | Anagram check | Frequency array (int[26]) or sort-compare; O(n) vs O(n log n); HashMap for Unicode |
| **Q91** | First non-repeating | LinkedHashMap frequency count + stream filter; or two-pass with int[26]; O(n) |
| **Q92** | Two Sum | HashMap: complement = target - num; one-pass O(n) time O(n) space; sorted → two pointers O(1) space |
| **Q93** | Sliding window | Fixed size K: maintain window sum, slide by add right/remove left; O(n) time O(1) space |
| **Q94** | Reverse LinkedList | Iterative: prev/curr/next pointer dance O(1) space; Recursive: O(n) stack |
| **Q95** | Detect cycle | Floyd's: slow (1 step) + fast (2 steps); meet → cycle; find start: reset slow to head, advance both by 1 |
| **Q96** | Tree traversals | Inorder(L-Root-R), Preorder(Root-L-R), Postorder(L-R-Root); iterative inorder uses stack |
| **Q97** | Level-order BFS | Queue + level size loop; O(n) time, O(n) space for queue |
| **Q98** | Find duplicates | HashSet.add returns false for duplicates; or groupingBy + filter count>1; O(n) |
| **Q99** | Group anagrams | Sorted chars as key → groupingBy collector; O(n·k·log k) time |
| **Q100** | FizzBuzz | IntStream + mapToObj; or pattern match switch on n%15; write it fast and correctly |

---

> **Final Interview Tips:**
> - Always mention Java version for features (e.g., "Records, available since Java 16").
> - Lead with the modern approach, then show you know the traditional way.
> - For coding, state complexity BEFORE being asked.
> - Draw diagrams for HashMap, JVM memory, and thread states.
> - The sealed + records + pattern switch trio is the #1 Java 17/21 topic for 2026.
> - Virtual Threads are the #1 concurrency topic — know pinning and when to use them.

---
*End of Java Interview Preparation Guide 2026*
