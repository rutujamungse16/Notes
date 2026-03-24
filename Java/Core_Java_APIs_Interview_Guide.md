# Core Java APIs - Complete Interview Guide 2026

## Table of Contents (Click to Navigate)

### [Part 1: Object Class Methods](#part-1-object-class-methods)
- [1.1 equals() Method](#11-equals-method)
- [1.2 hashCode() Method](#12-hashcode-method)
- [1.3 toString() Method](#13-tostring-method)
- [1.4 clone() Method](#14-clone-method)
- [1.5 equals() and hashCode() Contract](#15-equals-and-hashcode-contract)

### [Part 2: String Essentials](#part-2-string-essentials)
- [2.1 String Immutability](#21-string-immutability)
- [2.2 String Pool](#22-string-pool)
- [2.3 String Methods](#23-string-methods)
- [2.4 String Comparison](#24-string-comparison)

### [Part 3: StringBuilder vs StringBuffer](#part-3-stringbuilder-vs-stringbuffer)
- [3.1 StringBuffer (Thread-Safe)](#31-stringbuffer-thread-safe)
- [3.2 StringBuilder (Not Thread-Safe)](#32-stringbuilder-not-thread-safe)
- [3.3 Comparison and When to Use](#33-comparison-and-when-to-use)

### [Part 4: Wrapper Classes and Autoboxing](#part-4-wrapper-classes-and-autoboxing)
- [4.1 Wrapper Classes](#41-wrapper-classes)
- [4.2 Autoboxing](#42-autoboxing)
- [4.3 Unboxing](#43-unboxing)
- [4.4 Integer Caching](#44-integer-caching)

### [Part 5: Utility Classes](#part-5-utility-classes)
- [5.1 Math Class](#51-math-class)
- [5.2 Random Class](#52-random-class)
- [5.3 System Class](#53-system-class)

### [Part 6: Date and Time - Legacy](#part-6-date-and-time---legacy)
- [6.1 Legacy Date Class](#61-legacy-date-class)
- [6.2 Calendar Class](#62-calendar-class)
- [6.3 SimpleDateFormat](#63-simpledateformat)

### [Part 7: Date and Time - Modern API](#part-7-date-and-time---modern-api)
- [7.1 LocalDate](#71-localdate)
- [7.2 LocalTime](#72-localtime)
- [7.3 LocalDateTime](#73-localdatetime)
- [7.4 Instant](#74-instant)
- [7.5 ZonedDateTime](#75-zoneddatetime)

### [Part 8: Formatting and Parsing Dates](#part-8-formatting-and-parsing-dates)
- [8.1 DateTimeFormatter](#81-datetimeformatter)
- [8.2 Parsing Strings to Dates](#82-parsing-strings-to-dates)
- [8.3 Custom Patterns](#83-custom-patterns)

### [Part 9: Interview Traps & Best Practices](#part-9-interview-traps--best-practices)
- [9.1 Common Mistakes](#91-common-mistakes)
- [9.2 Best Practices](#92-best-practices)

### [Part 10: Experienced-Level Q&A (4+ Years)](#part-10-experienced-level-qa-4-years)
- [10.1 Advanced String Concepts](#101-advanced-string-concepts)
- [10.2 Date/Time Handling at Scale](#102-datetime-handling-at-scale)
- [10.3 Performance Considerations](#103-performance-considerations)

---

## Part 1: Object Class Methods

Every class in Java inherits from `Object` class. Understanding its methods is crucial.

### 1.1 equals() Method

**Purpose:** Compare if two objects are equal.

**Default Behavior:** Compares memory addresses (like `==`).

```java
public class Student {
    String name;
    int id;
    
    // Default equals (compares reference)
    // s1.equals(s2) checks if s1 and s2 point to same object
    
    // Override for meaningful comparison
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;           // Same reference
        if (obj == null) return false;
        if (getClass() != obj.getClass()) return false;
        
        Student other = (Student) obj;
        return this.id == other.id && 
               this.name.equals(other.name);
    }
}

public class Main {
    public static void main(String[] args) {
        Student s1 = new Student();
        s1.name = "Alice";
        s1.id = 1;
        
        Student s2 = new Student();
        s2.name = "Alice";
        s2.id = 1;
        
        System.out.println(s1 == s2);           // false (different objects)
        System.out.println(s1.equals(s2));      // true (same content)
    }
}
```

**Memory Tip:** "equals() = Content comparison, == = Reference comparison"[web:35][web:36]

### 1.2 hashCode() Method

**Purpose:** Return a unique integer hash for the object. Used in HashMaps and HashSets.

**Default Behavior:** Returns hash based on object's memory address.

```java
public class User {
    String email;
    
    @Override
    public int hashCode() {
        return email.hashCode();  // Hash based on email
    }
    
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof User)) return false;
        return this.email.equals(((User) obj).email);
    }
}

public class Main {
    public static void main(String[] args) {
        Map<User, String> map = new HashMap<>();
        User u1 = new User();
        u1.email = "alice@example.com";
        
        map.put(u1, "Alice");
        System.out.println(map.get(u1));  // Alice
    }
}
```

**Why hashCode() matters:**
- HashMap uses hashCode() to find bucket quickly
- Then uses equals() to verify exact match
- If hashCode() is inconsistent, lookups fail

**Memory Tip:** "hashCode() = Quick lookup, equals() = Exact verification"[web:35][web:37]

### 1.3 toString() Method

**Purpose:** Return string representation of object.

**Default Behavior:** Returns `ClassName@HashCode`.

```java
public class Person {
    String name;
    int age;
    
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

public class Main {
    public static void main(String[] args) {
        Person p = new Person();
        p.name = "Bob";
        p.age = 30;
        
        System.out.println(p);  // Person{name='Bob', age=30}
    }
}
```

### 1.4 clone() Method

**Purpose:** Create a copy of an object.

**Default Behavior:** Throws `CloneNotSupportedException`.

```java
public class Book implements Cloneable {
    String title;
    String author;
    
    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();  // Shallow copy
    }
}

public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        Book b1 = new Book();
        b1.title = "Java";
        b1.author = "Author";
        
        Book b2 = (Book) b1.clone();  // Deep copy
        System.out.println(b1 == b2);           // false (different objects)
        System.out.println(b1.title.equals(b2.title));  // true (same content)
    }
}
```

**Shallow vs Deep Copy:**
- **Shallow:** Copies only primitives; references point to same objects
- **Deep:** Copies everything including nested objects

### 1.5 equals() and hashCode() Contract

**Contract Rules:**[web:37][web:38]
1. If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` MUST be true
2. If `a.hashCode() == b.hashCode()`, `a.equals(b)` may be true or false
3. If `a.equals(b)` is false, hashCodes can be same (hash collision)

**Interview Trap:**
- **Q:** "I only override equals(). Is that enough?"
- **A:** No! If you override equals(), you MUST override hashCode(). Otherwise, HashMap/HashSet won't work correctly.

```java
// WRONG: Only overrides equals
public class BadClass {
    String id;
    
    @Override
    public boolean equals(Object obj) {
        return this.id.equals(((BadClass) obj).id);
    }
    // Missing hashCode()!
}

// In HashMap:
Map<BadClass, String> map = new HashMap<>();
BadClass b1 = new BadClass();
b1.id = "1";
map.put(b1, "value1");

BadClass b2 = new BadClass();
b2.id = "1";
System.out.println(map.get(b2));  // null! (because hashCode differs)
```

---

## Part 2: String Essentials

### 2.1 String Immutability

**What:** Once a String is created, it cannot be changed. Any operation creates a new String.

```java
public class Main {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = s1.concat(" World");
        
        System.out.println(s1);  // Hello (unchanged)
        System.out.println(s2);  // Hello World (new object)
        System.out.println(s1 == s2);  // false (different objects)
    }
}
```

**Why Immutable?**
✅ Thread safety (no synchronization needed)  
✅ Easier to cache (String pool)  
✅ Safer for keys in HashMap  
✅ Security (credentials not modified)  

**Memory Tip:** "String = Immutable = Cannot change after creation"[web:40][web:46]

### 2.2 String Pool

**What:** Special memory region where unique String literals are stored. Saves memory.

```java
public class Main {
    public static void main(String[] args) {
        // Both point to same object in String pool
        String s1 = "Hello";
        String s2 = "Hello";
        System.out.println(s1 == s2);          // true (same reference)
        System.out.println(s1.equals(s2));     // true (same content)
        
        // Created in heap (not pool)
        String s3 = new String("Hello");
        System.out.println(s1 == s3);          // false (different reference)
        System.out.println(s1.equals(s3));     // true (same content)
        
        // intern() moves to pool
        String s4 = s3.intern();
        System.out.println(s1 == s4);          // true (now in pool)
    }
}
```

```
String Pool (Heap region):
┌─────────────┐
│  "Hello"    │ ← s1, s2 point here
└─────────────┘

Heap:
┌─────────────┐
│  "Hello"    │ ← s3 points here (separate)
└─────────────┘
```

**Interview Trap:**
- **Q:** "Why `s1 = "Hello"; s2 = "Hello"; s1 == s2` is true?"
- **A:** String literals are automatically interned to the String pool. Both reference the same object.

### 2.3 String Methods

```java
String s = "Hello World";

s.length();           // 11
s.charAt(0);          // 'H'
s.substring(0, 5);    // "Hello"
s.toUpperCase();      // "HELLO WORLD"
s.toLowerCase();      // "hello world"
s.trim();             // Removes leading/trailing spaces
s.split(" ");         // ["Hello", "World"]
s.contains("World");  // true
s.startsWith("Hello");// true
s.endsWith("World");  // true
s.replace('o', '0');  // "Hell0 W0rld"
s.indexOf("o");       // 4 (first occurrence)
s.lastIndexOf("o");   // 7 (last occurrence)
s.isEmpty();          // false
s.compareTo("Hello World");  // 0 (equal)
```

### 2.4 String Comparison

```java
String s1 = "Hello";
String s2 = new String("Hello");
String s3 = "Hello";

// == : Reference comparison
System.out.println(s1 == s2);      // false (different objects)
System.out.println(s1 == s3);      // true (same pool reference)

// equals() : Content comparison
System.out.println(s1.equals(s2)); // true (same content)
System.out.println(s1.equals(s3)); // true (same content)

// equalsIgnoreCase() : Case-insensitive
System.out.println("hello".equalsIgnoreCase("HELLO"));  // true

// compareTo() : Lexicographic comparison
System.out.println("Apple".compareTo("Banana"));  // negative (Apple comes first)
System.out.println("Banana".compareTo("Apple"));  // positive
System.out.println("Apple".compareTo("Apple"));   // 0 (equal)
```

**Memory Tip:** "Use equals() for String comparison, == for reference"

---

## Part 3: StringBuilder vs StringBuffer

### 3.1 StringBuffer (Thread-Safe)

Synchronized; slower but safe for multi-threaded environments.

```java
public class Main {
    public static void main(String[] args) {
        StringBuffer sb = new StringBuffer("Hello");
        
        sb.append(" World");
        System.out.println(sb);  // Hello World
        
        sb.insert(5, ",");
        System.out.println(sb);  // Hello, World
        
        sb.delete(5, 6);
        System.out.println(sb);  // Hello World
        
        sb.reverse();
        System.out.println(sb);  // dlroW olleH
    }
}
```

### 3.2 StringBuilder (Not Thread-Safe)

Not synchronized; faster for single-threaded use.

```java
public class Main {
    public static void main(String[] args) {
        StringBuilder sb = new StringBuilder("Hello");
        
        sb.append(" World");
        System.out.println(sb);  // Hello World
        
        sb.insert(5, ",");
        System.out.println(sb);  // Hello, World
        
        sb.deleteCharAt(5);
        System.out.println(sb);  // Hello World
    }
}
```

### 3.3 Comparison and When to Use

| Feature | String | StringBuffer | StringBuilder |
|---------|--------|--------------|---------------|
| Immutable | ✅ Yes | ❌ No | ❌ No |
| Thread-Safe | ✅ Yes | ✅ Yes | ❌ No |
| Performance | Slow (creates new) | Medium | **Fast** |
| Use Case | Constants | Multi-threaded | Single-threaded |

**Performance Comparison:**

```java
// BAD: Creates many String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // O(n²) complexity - creates 1000 new strings!
}

// GOOD: Single object
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);  // O(n) complexity
}
String result = sb.toString();
```

**Memory Tip:** "String = Immutable, StringBuffer = Thread-safe, StringBuilder = Fast"

**Interview Trap:**
- **Q:** "Should I always use StringBuilder?"
- **A:** No. Use String for constants. Use StringBuilder for single-threaded concatenation. Use StringBuffer in multi-threaded code (rare in practice).

---

## Part 4: Wrapper Classes and Autoboxing

### 4.1 Wrapper Classes

Wrapper around primitives to treat them as objects.

| Primitive | Wrapper |
|-----------|---------|
| byte | Byte |
| short | Short |
| int | Integer |
| long | Long |
| float | Float |
| double | Double |
| char | Character |
| boolean | Boolean |

```java
public class Main {
    public static void main(String[] args) {
        // Creating wrapper objects
        Integer i = new Integer(10);
        Double d = new Double(3.14);
        Boolean b = new Boolean(true);
        
        // Getting primitive values
        int x = i.intValue();
        double y = d.doubleValue();
        boolean z = b.booleanValue();
        
        // Useful methods
        System.out.println(Integer.MAX_VALUE);  // 2147483647
        System.out.println(Integer.MIN_VALUE);  // -2147483648
        System.out.println(Integer.parseInt("42"));   // 42
        System.out.println(Double.parseDouble("3.14")); // 3.14
    }
}
```

### 4.2 Autoboxing

Automatic conversion from primitive to wrapper:

```java
public class Main {
    public static void main(String[] args) {
        // Autoboxing: int → Integer
        Integer i = 10;           // Equivalent to Integer.valueOf(10)
        Double d = 3.14;
        
        // List requires objects, not primitives
        List<Integer> list = new ArrayList<>();
        list.add(5);              // Autoboxing: 5 → Integer(5)
        list.add(10);
        list.add(15);
    }
}
```

### 4.3 Unboxing

Automatic conversion from wrapper to primitive:

```java
public class Main {
    public static void main(String[] args) {
        Integer i = 10;
        
        // Unboxing: Integer → int
        int x = i;                // Equivalent to i.intValue()
        int y = i + 5;            // Automatically unboxed
        
        // Null unboxing throws exception
        Integer nullInt = null;
        // int z = nullInt;       // NullPointerException!
    }
}
```

### 4.4 Integer Caching

Java caches Integer objects from -128 to 127 for performance:

```java
public class Main {
    public static void main(String[] args) {
        Integer a = 100;
        Integer b = 100;
        System.out.println(a == b);  // true (cached)
        
        Integer c = 200;
        Integer d = 200;
        System.out.println(c == d);  // false (not cached, different objects)
        
        System.out.println(c.equals(d));  // true (content same)
    }
}
```

**Memory Tip:** "Autoboxing = Primitive to Wrapper, Unboxing = Wrapper to Primitive"

**Interview Trap:**
- **Q:** "Why is `Integer a = 100; Integer b = 100; a == b` true?"
- **A:** Because 100 is within cached range (-128 to 127). For 200, it's false.

---

## Part 5: Utility Classes

### 5.1 Math Class

```java
public class Main {
    public static void main(String[] args) {
        Math.abs(-10);              // 10
        Math.sqrt(16);              // 4.0
        Math.pow(2, 3);             // 8.0
        Math.max(10, 20);           // 20
        Math.min(10, 20);           // 10
        Math.ceil(3.2);             // 4.0
        Math.floor(3.8);            // 3.0
        Math.round(3.5);            // 4
        Math.random();              // 0.0 to 1.0
        Math.sin(Math.PI / 2);      // 1.0
        Math.cos(0);                // 1.0
    }
}
```

### 5.2 Random Class

```java
import java.util.Random;

public class Main {
    public static void main(String[] args) {
        Random r = new Random();
        
        int randomInt = r.nextInt();         // Any int
        int randomInRange = r.nextInt(100);  // 0-99
        double randomDouble = r.nextDouble();// 0.0-1.0
        boolean randomBool = r.nextBoolean();
        
        // Seed for reproducibility
        Random r2 = new Random(42);  // Same seed = same sequence
    }
}
```

### 5.3 System Class

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Output");    // Print to console
        System.err.println("Error");     // Print to error stream
        
        long time = System.currentTimeMillis();  // Current time in ms
        
        System.getProperty("java.version");      // Java version
        System.getProperty("os.name");           // OS name
        
        System.exit(0);              // Exit JVM
        
        // Array copy
        int[] src = {1, 2, 3, 4, 5};
        int[] dest = new int[5];
        System.arraycopy(src, 0, dest, 0, 5);
    }
}
```

---

## Part 6: Date and Time - Legacy

### 6.1 Legacy Date Class

**Note:** Date class is mostly deprecated. Use modern API instead.

```java
import java.util.Date;

public class Main {
    public static void main(String[] args) {
        Date d = new Date();  // Current date/time
        System.out.println(d);  // Wed Feb 04 20:12:00 IST 2026
        
        long time = d.getTime();  // Milliseconds since epoch
        Date d2 = new Date(time);
    }
}
```

**Problems with Date:**
- Mutable (can be changed)
- Not thread-safe
- Confusing API (year starts from 1900)
- Difficult timezone handling

### 6.2 Calendar Class

Better than Date, but still considered legacy:

```java
import java.util.Calendar;
import java.util.Date;

public class Main {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        
        cal.set(2026, 1, 4);  // Year, Month (0-indexed), Day
        
        cal.get(Calendar.YEAR);        // 2026
        cal.get(Calendar.MONTH);       // 1 (Feb, 0-indexed)
        cal.get(Calendar.DAY_OF_MONTH);// 4
        
        cal.add(Calendar.DAY_OF_MONTH, 5);  // Add 5 days
        
        Date d = cal.getTime();
    }
}
```

**Problems:**
- Mutable
- Month is 0-indexed (confusing)
- Not thread-safe

### 6.3 SimpleDateFormat

Format and parse dates (legacy):

```java
import java.text.SimpleDateFormat;
import java.util.Date;

public class Main {
    public static void main(String[] args) throws Exception {
        SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");
        
        // Parse String to Date
        Date d = sdf.parse("04/02/2026");
        
        // Format Date to String
        String formatted = sdf.format(d);
        System.out.println(formatted);  // 04/02/2026
    }
}
```

---

## Part 7: Date and Time - Modern API

### 7.1 LocalDate

Date without time (year-month-day):

```java
import java.time.LocalDate;

public class Main {
    public static void main(String[] args) {
        LocalDate today = LocalDate.now();
        System.out.println(today);  // 2026-02-04
        
        LocalDate date = LocalDate.of(2026, 2, 4);
        
        date.getYear();      // 2026
        date.getMonth();     // FEBRUARY
        date.getDayOfMonth();// 4
        date.getDayOfWeek(); // WEDNESDAY
        
        LocalDate tomorrow = today.plusDays(1);
        LocalDate nextMonth = today.plusMonths(1);
        
        LocalDate yesterday = today.minusDays(1);
        
        boolean isLeap = LocalDate.of(2024, 1, 1).isLeapYear();
    }
}
```

### 7.2 LocalTime

Time without date (hour-minute-second):

```java
import java.time.LocalTime;

public class Main {
    public static void main(String[] args) {
        LocalTime now = LocalTime.now();
        System.out.println(now);  // 20:12:30.123456789
        
        LocalTime time = LocalTime.of(14, 30, 0);  // 2:30 PM
        
        time.getHour();      // 14
        time.getMinute();    // 30
        time.getSecond();    // 0
        
        LocalTime later = time.plusHours(2);
    }
}
```

### 7.3 LocalDateTime

Date + Time combined:

```java
import java.time.LocalDateTime;

public class Main {
    public static void main(String[] args) {
        LocalDateTime now = LocalDateTime.now();
        System.out.println(now);  // 2026-02-04T20:12:30.123456789
        
        LocalDateTime dt = LocalDateTime.of(2026, 2, 4, 14, 30, 0);
        
        dt.getYear();
        dt.getMonth();
        dt.getHour();
        dt.getMinute();
        
        LocalDateTime future = dt.plusDays(5).plusHours(2);
    }
}
```

### 7.4 Instant

Point in time (UTC):

```java
import java.time.Instant;

public class Main {
    public static void main(String[] args) {
        Instant now = Instant.now();
        System.out.println(now);  // 2026-02-04T14:42:30.123456789Z
        
        long epochSeconds = now.getEpochSecond();
        long nanos = now.getNano();
        
        Instant past = now.minusSeconds(3600);  // 1 hour ago
        
        // Compare
        System.out.println(now.isAfter(past));   // true
    }
}
```

### 7.5 ZonedDateTime

Date + Time + Timezone:

```java
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;

public class Main {
    public static void main(String[] args) {
        ZonedDateTime zdt = ZonedDateTime.now();
        System.out.println(zdt);  // 2026-02-04T20:12:30.123+05:30[Asia/Kolkata]
        
        LocalDateTime dt = LocalDateTime.now();
        ZonedDateTime inUTC = dt.atZone(ZoneId.of("UTC"));
        ZonedDateTime inPST = dt.atZone(ZoneId.of("America/Los_Angeles"));
        
        ZonedDateTime converted = inUTC.withZoneSameInstant(ZoneId.of("Asia/Kolkata"));
    }
}
```

---

## Part 8: Formatting and Parsing Dates

### 8.1 DateTimeFormatter

Modern way to format/parse dates:

```java
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public class Main {
    public static void main(String[] args) {
        LocalDate date = LocalDate.of(2026, 2, 4);
        
        // Predefined formatters
        String iso = date.format(DateTimeFormatter.ISO_DATE);
        System.out.println(iso);  // 2026-02-04
        
        // Custom formatter
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
        String formatted = date.format(formatter);
        System.out.println(formatted);  // 04/02/2026
    }
}
```

### 8.2 Parsing Strings to Dates

```java
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public class Main {
    public static void main(String[] args) {
        String dateStr = "04/02/2026";
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
        
        LocalDate date = LocalDate.parse(dateStr, formatter);
        System.out.println(date);  // 2026-02-04
        
        // ISO format (no formatter needed)
        LocalDate isoDate = LocalDate.parse("2026-02-04");
        System.out.println(isoDate);  // 2026-02-04
    }
}
```

### 8.3 Custom Patterns

```
Pattern Symbols:
y  → Year (2026)
M  → Month (02)
d  → Day (04)
H  → Hour (24-hour: 14)
h  → Hour (12-hour: 02)
m  → Minute (30)
s  → Second (45)
S  → Millisecond (123)
a  → AM/PM
E  → Day name (Wed)
z  → Timezone (IST)
```

**Examples:**

```java
DateTimeFormatter f1 = DateTimeFormatter.ofPattern("dd/MM/yyyy");
DateTimeFormatter f2 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
DateTimeFormatter f3 = DateTimeFormatter.ofPattern("E, MMM dd, yyyy");  // Wed, Feb 04, 2026
DateTimeFormatter f4 = DateTimeFormatter.ofPattern("hh:mm a");          // 02:30 PM
```

---

## Part 9: Interview Traps & Best Practices

### 9.1 Common Mistakes

**Trap 1: Forgetting to override hashCode() with equals()**

```java
// WRONG
public class User {
    String email;
    
    @Override
    public boolean equals(Object obj) {
        return this.email.equals(((User) obj).email);
    }
    // Missing hashCode()!
}

// HashMap lookup fails for "equal" objects
Map<User, String> map = new HashMap<>();
User u1 = new User();
u1.email = "alice@example.com";
map.put(u1, "Alice");

User u2 = new User();
u2.email = "alice@example.com";
System.out.println(map.get(u2));  // null! Should be "Alice"
```

**Trap 2: String concatenation in loops**

```java
// SLOW O(n²)
String result = "";
for (int i = 0; i < 10000; i++) {
    result += i;  // Creates new String each time!
}

// FAST O(n)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

**Trap 3: Using Date/Calendar instead of LocalDate**

```java
// OLD (avoid)
Date d = new Date();
Calendar cal = Calendar.getInstance();
cal.set(2026, 1, 4);  // Month is 0-indexed!

// NEW (prefer)
LocalDate date = LocalDate.of(2026, 2, 4);
LocalDateTime dt = LocalDateTime.now();
```

**Trap 4: Mutable wrapper objects in HashMap**

```java
// PROBLEM: Wrapper becomes immutable after boxing
Integer key = 100;
Map<Integer, String> map = new HashMap<>();
map.put(key, "value");
// key is immutable now, but original int is not guaranteed
```

**Trap 5: NullPointerException from unboxing**

```java
Integer i = null;
// int x = i;  // NullPointerException!

// Fix: Check for null
if (i != null) {
    int x = i;
}
```

### 9.2 Best Practices

✅ **Always override both equals() and hashCode()**  
✅ **Use LocalDate/LocalDateTime instead of Date/Calendar**  
✅ **Use StringBuilder for string concatenation in loops**  
✅ **Prefer String for constants, StringBuilder for dynamic content**  
✅ **Use DateTimeFormatter for consistent date formatting**  
✅ **Mark hashCode() with @Override annotation**  
✅ **Always check for null before unboxing**  
✅ **Use try-catch for parsing dates (DateTimeException)**  
✅ **Store dates in UTC, convert to local timezone for display**  

---

## Part 10: Experienced-Level Q&A (4+ Years)

### 10.1 Advanced String Concepts

**Q1. Explain String interning. Why should we care in production systems?**

A: String interning moves a String to the String pool, saving memory. Useful for large datasets of repetitive strings. However, excessive interning can degrade performance.[web:40][web:43]

```java
String s1 = new String("Hello");
String s2 = s1.intern();

System.out.println(s1 == s2);  // false (s1 not in pool)
// After interning, multiple references to same pool entry
```

**Q2. Why does String immutability matter in concurrent environments?**

A: Immutable Strings are inherently thread-safe. Multiple threads can safely share references without synchronization. Mutable strings would require locks.[web:40]

```java
// Thread-safe (no locking needed)
String shared = "immutable";
executor.execute(() -> System.out.println(shared));
executor.execute(() -> System.out.println(shared));
```

**Q3. What's the memory cost of StringBuilder vs repeated String concatenation?**

A: String concatenation creates O(n²) intermediate objects. StringBuilder uses a dynamic array buffer, costing O(n) memory and time.[web:40][web:43]

```
String concatenation: "a" + "b" + "c"
Step 1: Creates "ab"
Step 2: Creates "abc"
Total: 2 intermediate strings

StringBuilder: sb.append("a").append("b").append("c")
Total: 1 final string
```

### 10.2 Date/Time Handling at Scale

**Q4. How would you handle timezone-aware date operations across a distributed system?**

A: Store all dates in UTC (Instant), convert to local timezone only for display. Use ZonedDateTime for user-facing operations.[web:41][web:44]

```java
// Store in UTC
Instant stored = Instant.now();

// Display in user's timezone
ZonedDateTime userTime = stored.atZone(ZoneId.of("Asia/Kolkata"));
System.out.println(userTime);

// Compare globally (always works)
Instant i1 = Instant.now();
Instant i2 = stored;
System.out.println(i1.isAfter(i2));
```

**Q5. How do you handle date parsing from untrusted sources without exceptions?**

A: Use try-catch, validate format, or use TemporalQuery for flexible parsing.

```java
try {
    LocalDate date = LocalDate.parse(input, DateTimeFormatter.ISO_DATE);
} catch (DateTimeParseException e) {
    // Log and use default or reject
    logger.error("Invalid date: " + input);
    date = LocalDate.now();
}
```

**Q6. What's the performance implication of using LocalDateTime vs Instant?**

A: Instant is faster (single long value). LocalDateTime includes timezone logic. Use Instant for internal storage, LocalDateTime for display.

### 10.3 Performance Considerations

**Q7. How would you optimize equals() and hashCode() for million-object datasets?**

A: Cache hashCode, use lazy evaluation, avoid expensive field comparisons in equals()[web:35][web:37]

```java
public class OptimizedUser {
    String id;
    String email;
    private int hashCodeCache = -1;
    
    @Override
    public int hashCode() {
        if (hashCodeCache == -1) {
            hashCodeCache = id.hashCode();  // Only hash ID, not email
        }
        return hashCodeCache;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof OptimizedUser)) return false;
        OptimizedUser other = (OptimizedUser) obj;
        return this.id.equals(other.id);  // Only compare ID
    }
}
```

**Q8. Should I use StringBuilder or String in a multi-threaded web service?**

A: Use StringBuilder in request handlers (thread-confined). Use String for immutable shared data.

```java
// Per-request (each thread has its own)
public String handleRequest(String input) {
    StringBuilder response = new StringBuilder();  // Local to this thread
    response.append(input).append(" processed");
    return response.toString();
}

// Shared (immutable)
private static final String HEADER = "Response Header";  // Shared by all threads
```

**Q9. How would you implement a cache that respects equals() and hashCode() contract?**

A: Use LinkedHashMap with proper equals/hashCode, or use library caches.

```java
class CachedUserRepository {
    private final Map<String, User> cache = new LinkedHashMap<String, User>() {
        protected boolean removeEldestEntry(Map.Entry eldest) {
            return size() > 1000;  // LRU eviction
        }
    };
    
    User getUser(String id) {
        return cache.computeIfAbsent(id, this::fetchFromDB);
    }
}
```

**Q10. In a financial system, why is LocalDate better than Date for storing transaction dates?**

A: LocalDate is immutable, doesn't include timezone (dates are absolute), and has clearer semantics. Date is mutable and timezone-aware, causing bugs.

```java
// Good: Immutable, no timezone confusion
LocalDate transactionDate = LocalDate.of(2026, 2, 4);

// Avoid: Mutable, timezone-aware (problematic for absolute dates)
Date d = new Date(1744939800000L);  // What timezone?
```

---

## Summary Checklist for Interview

- [ ] Understand equals() and hashCode() contract
- [ ] Know when to override equals() and hashCode()
- [ ] Understand String immutability and String pool
- [ ] Know StringBuilder vs StringBuffer use cases
- [ ] Understand autoboxing and unboxing
- [ ] Know Integer caching range (-128 to 127)
- [ ] Be comfortable with Math and Random utilities
- [ ] Avoid legacy Date/Calendar; use LocalDate/LocalDateTime
- [ ] Know how to format/parse dates with DateTimeFormatter
- [ ] Understand timezone handling with ZonedDateTime
- [ ] Be ready to discuss performance tradeoffs
- [ ] Know common String/Date mistakes in production

---

**Last Updated:** February 2026  
**Prepared for:** Core Java APIs Interview Preparation (4+ Years Experience Focus)  
**References:** GeeksforGeeks, Baeldung, Oracle Java Documentation
