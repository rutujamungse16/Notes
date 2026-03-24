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

Java is a **general-purpose, object-oriented programming language** designed to be platform-independent. It follows the principle: *"Write Once, Run Anywhere (WORA)"*.

**Memory Tip:** "Java = Platform Independence + Object-Oriented + Secure"

### Why Java?

- **Platform Independent:** Compiled code runs on any device with JVM
- **Object-Oriented:** Supports classes, inheritance, polymorphism, encapsulation
- **Secure:** Built-in security features, automatic memory management
- **Multithreaded:** Support for concurrent programming
- **Robust:** Strong type checking, exception handling, garbage collection
- **Enterprise-Ready:** Used in large-scale applications (financial, healthcare, e-commerce)

---

## Part 2: Features of Java

### 1. **Simple & Familiar Syntax**
- Similar to C++, but simpler (no pointers, no operator overloading)
- Easy to learn for developers with C/C++ background

### 2. **Object-Oriented**
- Everything is an object (except primitives)
- Supports OOPS principles: Encapsulation, Inheritance, Polymorphism, Abstraction

### 3. **Platform Independent**
- Compiled bytecode runs on any JVM
- JVM handles OS-specific operations

### 4. **Secure**
- No manual memory management (Garbage Collection)
- Bytecode verification before execution
- Built-in security manager and cryptography APIs

### 5. **Robust**
- Strong type checking at compile-time
- Comprehensive exception handling mechanism
- Automatic garbage collection prevents memory leaks

### 6. **Multithreaded**
- Built-in support for concurrent programming
- Lightweight threads within same process
- Simplifies development of responsive applications

### 7. **Distributed**
- Supports RMI (Remote Method Invocation)
- Network programming APIs
- Designed for distributed computing

### 8. **High Performance**
- Just-In-Time (JIT) compilation for faster execution
- Optimization at runtime

### 9. **Dynamic**
- Runtime type information
- Dynamic class loading
- Reflection API for runtime introspection

**Interview Tip:** When asked "Why Java?", focus on the WORA principle and mention real-world applications relevant to the domain you're applying for (e.g., microservices, big data, mobile apps).

---

## Part 3: JDK, JRE, and JVM

### Understanding the Trinity

```
┌─────────────────────────────────────────────────┐
│              JDK (Java Development Kit)         │
│  ┌──────────────────────────────────────────┐  │
│  │      JRE (Java Runtime Environment)      │  │
│  │  ┌────────────────────────────────────┐  │  │
│  │  │   JVM (Java Virtual Machine)       │  │  │
│  │  │                                    │  │  │
│  │  │  • Executes bytecode               │  │  │
│  │  │  • Heap, Stack, Garbage Collector  │  │  │
│  │  └────────────────────────────────────┘  │  │
│  │                                          │  │
│  │  • Class Libraries (rt.jar, etc.)       │  │
│  │  • Security Manager                     │  │
│  │  • JIT Compiler                         │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
│  • Compiler (javac)                           │
│  • Debugger (jdb)                             │
│  • Tools (jar, javadoc, etc.)                 │
└─────────────────────────────────────────────────┘
```

### JVM (Java Virtual Machine)

**What:** Abstract computing machine that enables a computer to run Java programs

**Key Points:**
- **Platform Dependent:** Each OS has its own JVM implementation
- **Bytecode Executor:** Converts bytecode (.class files) into machine-specific instructions
- **Memory Management:** Manages heap, stack, garbage collection
- **Provides Abstraction:** Application developers don't need to worry about OS details

**Memory Tip:** JVM = "OS Translator" - translates bytecode to machine language

### JRE (Java Runtime Environment)

**What:** Minimum set of tools needed to RUN Java applications

**Contains:**
- JVM
- Class libraries (java.lang, java.util, etc.)
- Other runtime components
- NOT included: Compiler, debugger, development tools

**Use Case:** If you only need to execute Java programs (end user scenario)

### JDK (Java Development Kit)

**What:** Complete package for Java development

**Contains:**
- JRE (everything needed to run)
- Compiler (javac)
- Debugger (jdb)
- Development tools (jar, javadoc, etc.)
- Source code of JRE

**Use Case:** For developers who write and test Java code

### Compilation and Execution Flow

```
Source Code (.java)
    ↓
[javac - JDK Compiler]
    ↓
Bytecode (.class)
    ↓
[JVM - Runtime Environment]
    ↓
Machine Code (OS-specific)
    ↓
Program Execution
```

**Interview Trap:** 
- **Question:** "Can I run a Java program without JDK?"
- **Correct Answer:** Yes, with JRE. JDK includes JRE, so JDK is a superset of JRE.

**Interview Trap:**
- **Question:** "Why is JVM called Virtual Machine?"
- **Correct Answer:** Because it's an abstract computer that doesn't physically exist. It simulates a computer's behavior in software, abstracting hardware/OS differences.

---

## Part 4: Program Structure and Main Method

### Anatomy of a Java Program

```java
// 1. Package Declaration (optional)
package com.example.learning;

// 2. Import Statements (optional)
import java.util.Scanner;
import java.util.ArrayList;

// 3. Class Declaration
public class HelloWorld {
    
    // 4. Main Method (Entry Point)
    public static void main(String[] args) {
        System.out.println("Hello, World!");
        
        // Processing arguments
        for (String arg : args) {
            System.out.println("Argument: " + arg);
        }
    }
    
    // 5. Other Methods and Members
    public void greet(String name) {
        System.out.println("Hello, " + name);
    }
}
```

### Understanding the Main Method

```java
public static void main(String[] args)
```

Breaking it down:

| Component | Explanation |
|-----------|-------------|
| `public` | Accessible from anywhere; JVM needs to access it |
| `static` | Belongs to class, not instance; JVM calls it without creating object |
| `void` | Returns nothing |
| `main` | Entry point name (predefined by JVM) |
| `String[] args` | Command-line arguments as String array |

**Memory Tip:** "**P**ublic **S**tatic **V**oid **M**ain" (PSVM)

### Command-Line Arguments

```java
public class CommandLineDemo {
    public static void main(String[] args) {
        System.out.println("Total arguments: " + args.length);
        
        for (int i = 0; i < args.length; i++) {
            System.out.println("args[" + i + "] = " + args[i]);
        }
    }
}
```

**Execution:**
```
javac CommandLineDemo.java
java CommandLineDemo Hello World Java
```

**Output:**
```
Total arguments: 3
args[0] = Hello
args[1] = World
args[2] = Java
```

**Interview Trap:**
- **Question:** "What happens if main method is not public?"
- **Correct Answer:** Compilation succeeds, but runtime throws `NoSuchMethodError` because JVM cannot access private main method.

**Interview Trap:**
- **Question:** "Can I have multiple main methods?"
- **Correct Answer:** Yes! This is called **method overloading**. JVM only calls `public static void main(String[] args)`. Others are regular methods.

```java
public class MultipleMain {
    public static void main(String[] args) {
        System.out.println("Entry point");
    }
    
    // This is a different main method
    public static void main(int[] nums) {
        System.out.println("Overloaded main");
    }
}
```

---

## Part 5: Data Types and Variables

### Classification of Data Types

```
┌─────────────────────────────┐
│    Data Types in Java       │
├─────────────────┬───────────┤
│  Primitive (8)  │ Reference │
├─────────────────┼───────────┤
│ • boolean       │ • Classes │
│ • byte          │ • Arrays  │
│ • short         │ • Strings │
│ • int           │ • Objects │
│ • long          │           │
│ • float         │           │
│ • double        │           │
│ • char          │           │
└─────────────────┴───────────┘
```

### Primitive Data Types

| Type | Size | Range | Default | Wrapper |
|------|------|-------|---------|---------|
| `byte` | 1 byte | -128 to 127 | 0 | Byte |
| `short` | 2 bytes | -32,768 to 32,767 | 0 | Short |
| `int` | 4 bytes | -2^31 to 2^31-1 | 0 | Integer |
| `long` | 8 bytes | -2^63 to 2^63-1 | 0L | Long |
| `float` | 4 bytes | ±3.4E38 (approx) | 0.0f | Float |
| `double` | 8 bytes | ±1.7E308 (approx) | 0.0d | Double |
| `char` | 2 bytes | 0 to 65,535 (Unicode) | '\u0000' | Character |
| `boolean` | 1 bit | true/false | false | Boolean |

**Memory Tip:** "BSILDCFB" = Byte, Short, Int, Long, Double, Char, Float, Boolean

### Variable Declaration and Initialization

```java
// Single variable
int age = 25;

// Multiple variables of same type
int x = 10, y = 20, z = 30;

// Without initialization (gets default value)
int count;  // Default: 0
boolean flag;  // Default: false
String name;  // Default: null
```

### Primitive vs Reference Types

```java
// PRIMITIVE TYPE
int x = 10;  // Stores actual value in stack
int y = x;   // Copy of value
y = 20;      // x is still 10 (no effect on x)

System.out.println(x);  // 10
System.out.println(y);  // 20

// REFERENCE TYPE
StringBuilder sb1 = new StringBuilder("Hello");  // Reference in stack, object in heap
StringBuilder sb2 = sb1;  // Both point to same object
sb2.append(" World");     // Modifies shared object

System.out.println(sb1);  // Hello World (affected!)
System.out.println(sb2);  // Hello World
```

**Memory Visualization:**

```
PRIMITIVE TYPE (Stack):
┌──────┐
│  x=10│
└──────┘
┌──────┐
│  y=20│
└──────┘

REFERENCE TYPE:
Stack              Heap
┌─────────────┐   ┌────────────────┐
│ sb1 ──┐     │   │ StringBuilder   │
└───────┼─────┘   │ "Hello World"   │
        │         └────────────────┘
┌─────────┴──┐
│ sb2 ───┐   │
└─────────┼──┘
          │
          └──────> (Both point to same object)
```

**Interview Trap:**
- **Question:** "What's the difference between primitive and reference types?"
- **Correct Answer:** Primitives store actual values and are allocated on stack. References store memory addresses of objects allocated on heap. Copying primitives creates independent copies, but copying references creates multiple pointers to same object.

---

## Part 6: Type Casting

### Widening (Implicit) Casting

**Definition:** Converting from smaller data type to larger data type

**Size Hierarchy:** byte → short → int → long → float → double

```java
int intValue = 100;
long longValue = intValue;  // Automatic widening
double doubleValue = longValue;  // Automatic widening

System.out.println(longValue);     // 100
System.out.println(doubleValue);   // 100.0
```

**Characteristics:**
- Happens automatically (implicit)
- No data loss
- No explicit cast operator needed

### Narrowing (Explicit) Casting

**Definition:** Converting from larger data type to smaller data type

**Requires explicit cast operator:** `(targetType) value`

```java
double doubleValue = 65.5;
int intValue = (int) doubleValue;  // Explicit cast required
byte byteValue = (byte) intValue;

System.out.println(intValue);      // 65 (decimal part lost)
System.out.println(byteValue);     // 65
```

**Characteristics:**
- Manual casting required
- Potential data loss (precision/range)
- Risk of overflow/underflow

### Integer Narrowing Example (Overflow)

```java
int largeInt = 300;
byte byteValue = (byte) largeInt;  // Overflow!

System.out.println(largeInt);   // 300
System.out.println(byteValue);  // 44 (unexpected!)
```

**Why 44?** Byte range: -128 to 127. Value 300 wraps around.

### String Conversion

```java
// String to primitive
String numStr = "123";
int num = Integer.parseInt(numStr);
double decimal = Double.parseDouble("45.67");

// Primitive to String
String str1 = String.valueOf(100);
String str2 = Integer.toString(100);
String str3 = 100 + "";  // String concatenation
```

**Interview Trap:**
- **Question:** "What happens if you try to parse 'abc' as integer?"
- **Correct Answer:** Throws `NumberFormatException` at runtime. Always validate input before parsing!

```java
try {
    int num = Integer.parseInt("abc");
} catch (NumberFormatException e) {
    System.out.println("Invalid number format");
}
```

**Interview Trap:**
- **Question:** "Is widening or narrowing casting preferred?"
- **Correct Answer:** Widening is safer (no data loss). Narrowing should be used carefully with validation to prevent data loss and overflow.

---

## Part 7: Variable Scope and Lifetime

### Types of Scope in Java

```java
public class ScopeExample {
    
    // CLASS/STATIC SCOPE
    static int staticVar = 10;  // Shared by all instances
    
    // INSTANCE SCOPE
    int instanceVar = 20;  // Each instance has its own
    
    public void method(int paramVar) {  // PARAMETER SCOPE
        // LOCAL SCOPE
        int localVar = 30;
        
        {
            // BLOCK SCOPE
            int blockVar = 40;
            System.out.println(blockVar);  // OK
        }
        // System.out.println(blockVar);  // ERROR! Out of scope
        
        System.out.println(localVar);  // OK
    }
}
```

### Scope Hierarchy (Most to Least Restricted)

1. **Block Scope** (Narrowest)
   - Variables declared in `{}` block
   - Available only within that block
   
2. **Local Scope**
   - Variables in method body
   - Available throughout method
   
3. **Parameter Scope**
   - Method parameters
   - Available in method body
   
4. **Instance Scope**
   - Instance variables (non-static)
   - Available throughout object lifetime
   
5. **Static/Class Scope** (Broadest)
   - Static variables
   - Available throughout program execution

### Variable Lifetime

**Lifetime = Scope Duration**

```java
public class LifetimeExample {
    static int staticVar = 10;      // Lives until program exits
    
    int instanceVar = 20;           // Lives until object is garbage collected
    
    public void method() {
        int localVar = 30;          // Lives until method returns
        
        {
            int blockVar = 40;      // Lives until block ends
        }
    }
}
```

### Variable Shadowing

```java
public class ShadowingExample {
    int x = 10;  // Instance variable
    
    public void method() {
        int x = 20;  // Local variable shadows instance variable
        System.out.println(x);           // 20 (uses local)
        System.out.println(this.x);      // 10 (instance variable)
    }
}
```

**Interview Trap:**
- **Question:** "What's variable shadowing and is it good practice?"
- **Correct Answer:** When a local variable has same name as instance/class variable, shadowing occurs. Local variable takes precedence. It's considered bad practice as it reduces code clarity.

---

## Part 8: Final Variables

### Final Variables (Constants)

**What:** Once assigned, value cannot be changed

```java
public class FinalVariableExample {
    
    // Final instance variable (must initialize in constructor or at declaration)
    private final int MAX_ATTEMPTS;
    
    // Final static variable (class constant)
    public static final double PI = 3.14159;
    
    // Final local variable
    public void method() {
        final String MESSAGE = "Hello";
        // MESSAGE = "World";  // ERROR: Cannot reassign final variable
    }
    
    public FinalVariableExample(int attempts) {
        MAX_ATTEMPTS = attempts;  // Initialize final variable
    }
}
```

### Final Variables - Initialization Rules

```java
public class FinalRules {
    
    // Rule 1: Static final must be initialized
    static final int CONST_A = 100;  // OK
    // static final int CONST_B;      // ERROR
    
    // Rule 2: Instance final must be initialized in constructor or at declaration
    final int instVar = 50;           // OK: initialized at declaration
    
    final int instVar2;
    
    public FinalRules(int val) {
        instVar2 = val;               // OK: initialized in constructor
    }
    
    public void method() {
        // Rule 3: Local final must be initialized before use
        final int localFinal;
        if (true) {
            localFinal = 10;          // OK: initialized
        }
        System.out.println(localFinal);  // OK to use
    }
}
```

### Blank Final Variables

**Definition:** Final variable declared but not initialized at declaration. Must be initialized in constructor.

```java
public class BlankFinalExample {
    private final String name;      // Blank final
    private final int id;           // Blank final
    
    public BlankFinalExample(String name, int id) {
        this.name = name;           // Must initialize here
        this.id = id;               // Must initialize here
    }
    
    public String getName() {
        return name;
    }
}
```

### Final with Reference Types

```java
public class FinalReference {
    static final StringBuilder sb = new StringBuilder("Hello");
    
    public static void main(String[] args) {
        // sb = new StringBuilder("World");  // ERROR: Cannot reassign
        
        sb.append(" World");  // OK: Modifying contents is allowed
        System.out.println(sb);  // Hello World
    }
}
```

**Key Point:** Final reference means reference cannot change, but object contents can be modified.

**Interview Trap:**
- **Question:** "What's the difference between `final int x = 10;` and `static final int x = 10;`?"
- **Correct Answer:** First is instance constant (each object has its own), second is class constant (shared by all objects). Static final is more memory efficient.

---

## Part 9: Operators in Java

### Classification of Operators

#### 1. **Arithmetic Operators**

```java
int a = 10, b = 3;

System.out.println(a + b);      // 13 (Addition)
System.out.println(a - b);      // 7  (Subtraction)
System.out.println(a * b);      // 30 (Multiplication)
System.out.println(a / b);      // 3  (Division - integer division)
System.out.println(a % b);      // 1  (Modulus - remainder)

System.out.println(++a);        // 11 (Pre-increment)
System.out.println(a++);        // 11 (Post-increment, then a=12)
System.out.println(--b);        // 2  (Pre-decrement)
```

#### 2. **Relational Operators** (Return boolean)

```java
int x = 10, y = 20;

System.out.println(x > y);      // false
System.out.println(x < y);      // true
System.out.println(x >= y);     // false
System.out.println(x <= y);     // true
System.out.println(x == y);     // false
System.out.println(x != y);     // true
```

#### 3. **Logical Operators**

```java
boolean a = true, b = false;

System.out.println(a && b);     // false (AND - both true)
System.out.println(a || b);     // true  (OR - at least one true)
System.out.println(!a);         // false (NOT - negation)

// Short-circuit evaluation
if (a && (5 > 3)) {             // Left true, evaluates right
    System.out.println("Both are true");
}

if (false && (5 > 3)) {         // Left false, skips right (short-circuit)
    System.out.println("Not executed");
}
```

**Memory Tip:** 
- `&&` (AND) = Friendly Friend (both happy)
- `||` (OR) = Open Opinion (at least one happy)
- `!` (NOT) = Negate/Opposite

#### 4. **Bitwise Operators**

```java
int a = 5;   // Binary: 0101
int b = 3;   // Binary: 0011

System.out.println(a & b);      // 1   (AND:   0001)
System.out.println(a | b);      // 7   (OR:    0111)
System.out.println(a ^ b);      // 6   (XOR:   0110)
System.out.println(~a);         // -6  (NOT:   inverts bits)

System.out.println(a << 1);     // 10  (Left shift:  0101 → 1010)
System.out.println(a >> 1);     // 2   (Right shift: 0101 → 0010)
System.out.println(a >>> 1);    // 2   (Unsigned right shift - same for positive)
```

**Use Cases:** Low-level programming, flag management, optimization

#### 5. **Ternary Operator**

```java
int age = 20;
String status = age >= 18 ? "Adult" : "Minor";  // Ternary
System.out.println(status);  // Adult

// Nested ternary (avoid - reduces readability)
String grade = marks >= 90 ? "A" : marks >= 80 ? "B" : marks >= 70 ? "C" : "F";
```

**Syntax:** `condition ? valueIfTrue : valueIfFalse`

#### 6. **Assignment Operators**

```java
int x = 10;

x += 5;     // x = x + 5   → 15
x -= 3;     // x = x - 3   → 12
x *= 2;     // x = x * 2   → 24
x /= 4;     // x = x / 4   → 6
x %= 4;     // x = x % 4   → 2

x &= 1;     // x = x & 1   (bitwise AND assignment)
x |= 2;     // x = x | 2   (bitwise OR assignment)
```

#### 7. **instanceof Operator**

```java
String str = "Hello";
System.out.println(str instanceof String);          // true
System.out.println(str instanceof Object);          // true

Integer num = 10;
System.out.println(num instanceof Integer);         // true
System.out.println(num instanceof Object);          // true
```

### Operator Precedence (High to Low)

```
1. Postfix operators: () . [] ++ --
2. Unary: ! ~ + - ++ --
3. Multiplication: * / %
4. Addition: + -
5. Shift: << >> >>>
6. Relational: < > <= >= instanceof
7. Equality: == !=
8. Bitwise AND: &
9. Bitwise XOR: ^
10. Bitwise OR: |
11. Logical AND: &&
12. Logical OR: ||
13. Ternary: ?:
14. Assignment: = += -= *= /= etc.
```

**Memory Tip:** "PEMDAS" (use with care - precedence differs from math)

### Integer Division and Modulus Traps

```java
System.out.println(7 / 2);         // 3 (not 3.5 - integer division)
System.out.println(7.0 / 2);       // 3.5 (float division)
System.out.println(7 % 2);         // 1 (remainder)

System.out.println(-7 % 3);        // -1 (sign of dividend, not divisor)
System.out.println(7 % -3);        // 1
```

**Interview Trap:**
- **Question:** "What's the difference between `i++` and `++i`?"
- **Correct Answer:** 
  - `i++` (post-increment): Returns current value, then increments
  - `++i` (pre-increment): Increments, then returns new value
  - In loops: `++i` is slightly more efficient (doesn't create temp)

```java
int a = 5;
System.out.println(a++);  // Prints 5, then a becomes 6
System.out.println(++a);  // a becomes 7, then prints 7
```

---

## Part 10: Control Flow - Conditional Statements

### 1. **If-Else Statements**

```java
int age = 20;

// Simple if
if (age >= 18) {
    System.out.println("Adult");
}

// If-else
if (age >= 18) {
    System.out.println("Adult");
} else {
    System.out.println("Minor");
}

// If-else-if chain
if (age < 13) {
    System.out.println("Child");
} else if (age < 18) {
    System.out.println("Teenager");
} else if (age < 65) {
    System.out.println("Adult");
} else {
    System.out.println("Senior");
}

// Single statement if (no braces)
if (age >= 18)
    System.out.println("Adult");  // Only this line is part of if
```

**Interview Trap:**
- **Question:** "What's the output?"
```java
int x = 5;
if (x > 3)
    System.out.println("Greater");
    System.out.println("Always printed");  // This is NOT part of if!
```
- **Correct Answer:** Output is "Greater" and "Always printed". Always use braces to avoid dangling else problems.

### 2. **Switch Statements**

#### Switch with Integers

```java
int day = 3;
String dayName;

switch (day) {
    case 1:
        dayName = "Monday";
        break;
    case 2:
        dayName = "Tuesday";
        break;
    case 3:
        dayName = "Wednesday";
        break;
    default:
        dayName = "Unknown";
}

System.out.println(dayName);  // Wednesday
```

#### Switch with Strings (Java 7+)

```java
String season = "Spring";

switch (season) {
    case "Spring":
        System.out.println("March-May");
        break;
    case "Summer":
        System.out.println("June-August");
        break;
    case "Fall":
        System.out.println("September-November");
        break;
    default:
        System.out.println("Unknown season");
}
```

#### Switch Fall-through (Intentional)

```java
int level = 2;

switch (level) {
    case 1:
        System.out.println("Bronze");
    case 2:
        System.out.println("Silver");  // Executes even if level=1
    case 3:
        System.out.println("Gold");
    default:
        System.out.println("Unknown");
}

// Output if level=1:
// Bronze
// Silver
// Gold
// Unknown
```

**Interview Trap:**
- **Question:** "Why do we need `break` in switch statements?"
- **Correct Answer:** Without `break`, execution continues to next case (fall-through). Usually undesired. Break prevents this unless intentional fall-through is needed.

**Interview Trap:**
- **Question:** "Can switch use double or float?"
- **Correct Answer:** No. Switch only supports: `byte`, `short`, `int`, `char`, `String`, `Enum`, and Wrapper types (`Integer`, etc.). This is a common mistake!

```java
// ERROR: switch does not support float
double score = 85.5;
// switch (score) { ... }  // Compile error
```

#### Enhanced Switch Expression (Java 14+)

```java
String day = "Monday";

String type = switch (day) {
    case "Saturday", "Sunday" -> "Weekend";
    case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday" -> "Weekday";
    default -> "Unknown";
};

System.out.println(type);  // Weekday
```

---

## Part 11: Loops

### 1. **For Loop**

```java
// Traditional for loop
for (int i = 1; i <= 5; i++) {
    System.out.println("Iteration: " + i);
}

// Infinite loop (careful!)
// for (;;) { }

// Multiple initialization/update
for (int i = 0, j = 10; i < j; i++, j--) {
    System.out.println("i=" + i + ", j=" + j);
}

// Nested loops
for (int i = 1; i <= 3; i++) {
    for (int j = 1; j <= 3; j++) {
        System.out.print(i + "-" + j + " ");
    }
    System.out.println();
}
```

### 2. **Enhanced For Loop (For-Each)**

```java
// Over arrays
int[] nums = {10, 20, 30};
for (int num : nums) {
    System.out.println(num);
}

// Over Collections
List<String> fruits = Arrays.asList("Apple", "Banana", "Mango");
for (String fruit : fruits) {
    System.out.println(fruit);
}

// Cannot access index directly
// for (String fruit : fruits) {
//     System.out.println(fruits.indexOf(fruit));  // Inefficient
// }
```

**Limitations:** Cannot modify collection while iterating, cannot access index directly.

### 3. **While Loop**

```java
int count = 0;

while (count < 5) {
    System.out.println("Count: " + count);
    count++;
}

// Reading from input
Scanner scanner = new Scanner(System.in);
String input = "";
while (!input.equals("quit")) {
    System.out.print("Enter command: ");
    input = scanner.nextLine();
}
```

### 4. **Do-While Loop**

```java
int count = 0;

do {
    System.out.println("Count: " + count);
    count++;
} while (count < 5);

// Key difference: Executes at least once
int x = 10;
do {
    System.out.println("Executed once: " + x);  // Prints even though x < 5
} while (x < 5);
```

**Memory Tip:** Do-While = "Do First, Check Later"

### Loop Comparison

| Type | When to Use | Executes if condition false? |
|------|------------|------------------------------|
| for | When iteration count known | No |
| while | When count unknown | No |
| do-while | When first execution guaranteed | Executed once |
| for-each | Iterating over collections | No |

### Break Statement

```java
// Break in for loop
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;  // Exits loop immediately
    }
    System.out.println(i);
}
// Output: 0 1 2 3 4

// Break in switch
String choice = "B";
switch (choice) {
    case "A":
        System.out.println("Option A");
        break;  // Without this, would print B too
    case "B":
        System.out.println("Option B");
        break;
}

// Break in nested loops (breaks inner loop only)
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) {
            break;  // Breaks inner loop only
        }
        System.out.print(j + " ");
    }
    System.out.println();
}
```

### Continue Statement

```java
// Continue skips to next iteration
for (int i = 0; i < 5; i++) {
    if (i == 2) {
        continue;  // Skips iteration when i=2
    }
    System.out.println(i);
}
// Output: 0 1 3 4

// Continue in while loop
int i = 0;
while (i < 5) {
    i++;
    if (i == 3) {
        continue;  // Skips to next iteration
    }
    System.out.println(i);
}
// Output: 1 2 4 5
```

### Labeled Statements

```java
// Labeled break (breaks outer loop)
outerLoop:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) {
            break outerLoop;  // Breaks OUTER loop
        }
        System.out.print(j + " ");
    }
}
// Output: 0

// Labeled continue
outerLoop:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (i == 1 && j == 1) {
            continue outerLoop;  // Continues OUTER loop
        }
        System.out.print("(" + i + "," + j + ") ");
    }
}
```

**Interview Trap:**
- **Question:** "What's the difference between `break` and `continue`?"
- **Correct Answer:**
  - `break`: Exits the loop completely
  - `continue`: Skips current iteration and moves to next
  - Both can use labels to target outer loops

---

## Part 12: Interview Quick Reference & Common Traps

### Memory Shortcuts

| Concept | Shortcut |
|---------|----------|
| Primitive vs Reference | "Primitives = Values, References = Addresses" |
| Widening vs Narrowing | "Wide Open (auto), Narrow Path (manual)" |
| JVM vs JRE vs JDK | "JVM runs, JRE runs+libs, JDK runs+develop" |
| && vs \|\| | "AND suspicious, OR open-minded" |
| Operators precedence | "PEMDAS (Math-like but different)" |
| Do-While | "Do First, Ask Later" |
| Break vs Continue | "Break = Exit, Continue = Skip" |

### Top 10 Interview Traps

#### 1. **Null Reference Errors**
```java
String str = null;
System.out.println(str.length());  // NullPointerException
```

#### 2. **String Comparison**
```java
String s1 = new String("Java");
String s2 = new String("Java");
System.out.println(s1 == s2);      // false (different objects)
System.out.println(s1.equals(s2));  // true (same content)
```

#### 3. **Integer Division**
```java
int result = 7 / 2;  // 3, not 3.5
```

#### 4. **Array Index Out of Bounds**
```java
int[] arr = {1, 2, 3};
System.out.println(arr[3]);  // ArrayIndexOutOfBoundsException
```

#### 5. **Type Casting Overflow**
```java
int big = 1000;
byte small = (byte) big;  // Overflow!
```

#### 6. **For Loop Variable Scope**
```java
for (int i = 0; i < 5; i++) { }
// System.out.println(i);  // ERROR: i out of scope
```

#### 7. **Comparing Floating Point**
```java
double d1 = 0.1 + 0.2;
double d2 = 0.3;
System.out.println(d1 == d2);  // false! Precision issues
```

#### 8. **Dangling Else**
```java
int x = 5;
if (x > 3)
    if (x > 10)
        System.out.println("Greater than 10");
else
    System.out.println("Not greater than 3");  // Belongs to inner if!
```

#### 9. **Switch without Default**
```java
int x = 5;
switch (x) {
    case 1:
        System.out.println("One");
        break;
    case 2:
        System.out.println("Two");
        break;
    // What if x is 5? Silent failure if no default
}
```

#### 10. **Mutable Final Objects**
```java
static final List<String> list = new ArrayList<>();
list.add("Item");  // OK - modifying contents
// list = new ArrayList<>();  // ERROR - reassigning reference
```

### Frequently Asked Interview Questions

**Q: What's the purpose of JVM?**
A: JVM abstracts hardware/OS differences, enabling bytecode to run on any platform. It provides memory management (garbage collection), security (bytecode verification), and optimizations (JIT compilation).

**Q: Why are there 8 primitive types in Java?**
A: Different primitive types serve different purposes - efficiency (byte, short vs int), precision (float vs double), character representation (char), and boolean logic. They optimize memory usage and performance.

**Q: Can I have a primitive as static final?**
A: Yes, and it's commonly used for constants. Example: `static final int MAX_VALUE = 100;`

**Q: What happens if I don't initialize a local variable?**
A: Compile error. Local variables must be initialized before use. Instance and static variables get default values.

**Q: How does Java achieve platform independence?**
A: Java source code compiles to platform-independent bytecode (.class files). Each platform has its own JVM that interprets this bytecode into machine-specific instructions.

**Q: Can I override static methods?**
A: No, static methods cannot be overridden. They can be hidden/shadowed by subclass static methods, but this is not polymorphism.

**Q: What's the difference between `==` and `.equals()`?**
A: `==` compares references (for objects), `.equals()` compares content. For primitives, only `==` is available.

**Q: Why should I use `StringBuilder` instead of string concatenation?**
A: Each `+` operation creates new String object (immutable). Multiple concatenations create many temporary objects. StringBuilder is mutable and more efficient for multiple operations.

---

## Part 13: Practice Scenarios & Edge Cases

### Scenario 1: Type Conversion Chain

```java
// What's the output?
byte b = 10;
short s = b;        // Widening: byte → short
int i = s;          // Widening: short → int
long l = i;         // Widening: int → long
float f = l;        // Widening: long → float (may lose precision!)
double d = f;       // Widening: float → double

System.out.println(b + " " + s + " " + i + " " + l + " " + f + " " + d);
// Output: 10 10 10 10 10.0 10.0
```

### Scenario 2: Operator Precedence

```java
// What's the output?
int result = 5 + 3 * 2 - 4 / 2;
// Precedence: * / before + -
// = 5 + 6 - 2 = 9

System.out.println(result);  // 9
```

### Scenario 3: Logical Short-Circuit

```java
// What's printed?
boolean x = false;
if (x && (System.out.println("Printed") != null)) {
    // "Printed" is never printed because x is false (short-circuit)
}

System.out.println("Done");  // This is printed
// Output: Done
```

### Scenario 4: Loop with Break

```java
// What's the output?
int sum = 0;
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;
    }
    sum += i;
}
System.out.println(sum);  // 0+1+2+3+4 = 10
```

### Scenario 5: Final Variable Assignment

```java
// Will this compile?
final int x;
x = 10;
// x = 20;  // ERROR: already assigned

System.out.println(x);  // 10 (compiles fine)
```

---

## Part 14: Time Complexity Considerations for Interviews

### Operations and Their Complexities

| Operation | Time Complexity | Note |
|-----------|-----------------|------|
| Array access by index | O(1) | Direct access |
| For loop over n elements | O(n) | Linear iteration |
| Nested for loops | O(n²) | Quadratic |
| ArrayList.add (end) | O(1) amortized | May expand |
| ArrayList.add (middle) | O(n) | Requires shifting |
| ArrayList.contains | O(n) | Linear search |
| HashMap.get | O(1) average | Hash lookup |
| String concatenation | O(n) | Creates new object |
| Switch statement | O(1) | Direct jump |

---

## Part 15: Coding Best Practices for Interviews

### 1. **Use Meaningful Names**
```java
// Bad
int x, y;
for (int i = 0; i < 10; i++) { }

// Good
int userAge, userScore;
for (int index = 0; index < students.length; index++) { }
```

### 2. **Handle Edge Cases**
```java
// Check for null
if (str != null && str.length() > 0) { }

// Check array bounds
if (arr != null && arr.length > index) { }
```

### 3. **Use Appropriate Data Structures**
```java
// For frequent lookups: HashMap
// For ordering: TreeMap or LinkedHashMap
// For duplicates: HashSet or TreeSet
```

### 4. **Write Readable Control Flow**
```java
// Avoid deeply nested conditions
if (condition1 && condition2 && condition3) {
    doSomething();
}

// Instead of:
if (condition1) {
    if (condition2) {
        if (condition3) {
            doSomething();
        }
    }
}
```

### 5. **Use Comments Wisely**
```java
// Explain WHY, not WHAT
int finalPrice = basePrice * TAX_RATE;  // Apply tax for total price
```

---

## Part 16: Experienced-Level Interview Questions (4+ Years)

### 16.1 Core Language and Memory

**Q1. Explain Java's memory model: heap vs stack vs metaspace. When does an object become eligible for garbage collection?**  
A: Local variables and references live on the stack, objects live on the heap, and class metadata is stored in metaspace. An object becomes eligible for GC when no live thread can reach it through any chain of references, including static fields and thread-local references.

**Q2. Is Java pass-by-value or pass-by-reference? Give an example that confuses people.**  
A: Java is strictly pass-by-value. For objects, the value being passed is the reference, so both caller and callee see the same underlying object, but reassigning the parameter in the method does not change the caller's reference. This confuses people when trying to swap two `Integer` or `String` references inside a method.

**Q3. How would you diagnose and fix a `java.lang.OutOfMemoryError` in production?**  
A: Identify the space (heap, metaspace, direct memory) from the error or logs, capture a heap dump using `jmap` or JVM flags, and analyze with tools like Eclipse MAT or VisualVM to find leak suspects. Fix by removing unintended strong references and introducing bounds/eviction policies.

---

### 16.2 Collections and Generics

**Q4. Internal differences between `ArrayList` and `LinkedList`. When would you prefer one over the other?**  
A: `ArrayList` uses a dynamic array (O(1) random access, O(n) middle insertions), while `LinkedList` is doubly-linked (O(1) iterator operations, O(n) random access). `ArrayList` is usually preferred; `LinkedList` is rarely better unless you have specific patterns.

**Q5. What is the difference between `HashMap`, `LinkedHashMap`, and `TreeMap`?**  
A: `HashMap` has O(1) average operations with no ordering. `LinkedHashMap` preserves insertion/access order. `TreeMap` keeps keys sorted with O(log n) operations. Choice depends on ordering needs and complexity expectations.

**Q6. Explain type erasure in generics and a pitfall it causes.**  
A: Generic type information exists only at compile time and is erased at runtime, so `List<String>` and `List<Integer>` both appear as `List`. This prevents `instanceof List<String>` and can cause heap pollution when mixing raw and parameterized types.

---

### 16.3 Concurrency Basics

**Q7. What problems do `volatile` and `synchronized` solve? Can `volatile` replace `synchronized`?**  
A: `volatile` ensures visibility and ordering for a single variable; `synchronized` ensures mutual exclusion and happens-before relationships for all variables in the block. `volatile` cannot replace `synchronized` for compound operations like `count++`.

**Q8. Explain thread safety for collections. When use `Collections.synchronizedList` vs `CopyOnWriteArrayList`?**  
A: `Collections.synchronizedList` uses a single lock for all operations. `CopyOnWriteArrayList` copies on write, making reads fast and lock-free. Use synchronized for mixed workloads, copy-on-write for read-heavy scenarios.

**Q9. How does `ThreadLocal` work, and where can it cause memory leaks?**  
A: Each thread maintains a map of `ThreadLocal` instances to values. In thread pools, failing to clear `ThreadLocal` values causes long-lived worker threads to retain references, leading to memory leaks. Always remove when done.

---

### 16.4 Exceptions, Design, and Best Practices

**Q10. How do you design your own exception hierarchy in a large codebase?**  
A: Define a small base custom exception, derive domain-specific exceptions beneath it, and translate lower-level technical exceptions into domain ones at module boundaries. Avoid throwing generic `Exception` from public APIs.

**Q11. When choose checked vs unchecked exception?**  
A: Use checked exceptions for recoverable conditions the caller should handle. Use unchecked exceptions for programming errors or unrecoverable situations where forcing handling adds noise.

**Q12. How do you ensure immutability of a class, and why is it useful?**  
A: Use `private final` fields, no setters, full initialization in constructor, and defensive copies for mutable inputs/outputs. Immutable objects are thread-safe, easier to reason about, and ideal as cache keys.

---

### 16.5 Practical Coding / Scenario Questions (4+ Years)

**Q13. Given a large list of integers, how would you find the top 10 numbers efficiently?**  
A: Maintain a min-heap (priority queue) of size 10: iterate once, push elements, and pop when size exceeds 10. This gives O(n log 10) ≈ O(n) complexity. Alternative: full sort for O(n log n), acceptable for smaller inputs.

**Q14. How would you design a retry utility with exponential backoff? What do you consider?**  
A: Accept a functional interface, configure max retries and base delay, implement retries with increasing sleep times and optional jitter to avoid thundering herd. Consider which exceptions are retryable, honor thread interrupts, and limit max backoff.

**Q15. You see high CPU and long GC pauses in a Java service. Outline your approach.**  
A: Inspect GC logs and flags for collector type and pause patterns. Use profilers or thread dumps for hotspots or stuck threads. Look for excessive allocations and unbounded caches. Tune heap/GC settings and reduce allocations.

---

**Last Updated:** February 2026  
**Prepared for:** Core Java Interview Preparation (4+ Years Experience Focus)
