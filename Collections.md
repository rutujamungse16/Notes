```markdown
# Part X: Collections and Generics – Interview Guide 2026 (4+ Years)

## Clickable Index

1. [Collections Framework Overview](#1-collections-framework-overview)  
2. [Core Collection and Map Interfaces](#2-core-collection-and-map-interfaces)  
3. [Iterable and Iterator](#3-iterable-and-iterator)  
4. [List Implementations](#4-list-implementations)  
   - [4.1 ArrayList](#41-arraylist)  
   - [4.2 LinkedList](#42-linkedlist)  
   - [4.3 Use-cases & Performance](#43-use-cases--performance-arraylist-vs-linkedlist)  
5. [Set Implementations](#5-set-implementations)  
   - [5.1 HashSet](#51-hashset)  
   - [5.2 LinkedHashSet](#52-linkedhashset)  
   - [5.3 TreeSet](#53-treeset)  
6. [Map Implementations](#6-map-implementations)  
   - [6.1 HashMap](#61-hashmap)  
   - [6.2 LinkedHashMap](#62-linkedhashmap)  
   - [6.3 TreeMap](#63-treemap)  
   - [6.4 ConcurrentHashMap Basics](#64-concurrenthashmap-basics)  
7. [Sorting and Comparison](#7-sorting-and-comparison)  
   - [7.1 Comparable](#71-comparable-interface)  
   - [7.2 Comparator](#72-comparator-interface)  
   - [7.3 Collections.sort / List.sort](#73-collectionssort-and-listsort)  
8. [Generics Fundamentals](#8-generics-fundamentals)  
   - [8.1 Generic Classes & Methods](#81-generic-classes-and-methods)  
   - [8.2 Type Parameters](#82-type-parameters-e-t-k-v)  
   - [8.3 Wildcards: ?, ? extends, ? super](#83-wildcards--extends--super)  
   - [8.4 Type Erasure](#84-type-erasure-concept)  
9. [Experienced-Level Q&A (4+ Years)](#9-experienced-level-qa-4-years)  
10. [Common Traps & Memory Tips](#10-common-traps--memory-tips)  

---

## 1. Collections Framework Overview

The Java Collections Framework is a set of interfaces and classes to store and manipulate groups of objects in a consistent way.[web:17][web:22]  

- Main root interfaces: **Collection** and **Map** (Map is separate hierarchy).[web:19][web:22]  
- Ready-made data structures: List, Set, Queue, Map etc.  
- Algorithms: sorting, searching, shuffling in `Collections` utility class.[web:17][web:22]  

**Memory Tip:** “**C**ollection = **C**ontainer for objects; `Collections` (with s) = helper **static** methods.”

---

## 2. Core Collection and Map Interfaces

### 2.1 Collection hierarchy

- `Collection<E>` is the root for:  
  - `List<E>` – ordered, index-based, allows duplicates.  
  - `Set<E>` – no duplicates, models mathematical set.  
  - `Queue<E>` / `Deque<E>` – for FIFO/LIFO style access.[web:17][web:22]  

### 2.2 Map hierarchy

- `Map<K,V>` – key–value pairs, keys are unique.[web:19][web:22]  
- Important subinterfaces:  
  - `SortedMap<K,V>` / `NavigableMap<K,V>` for sorted maps.  

**Code glimpse:**

```java
List<String> list = new ArrayList<>();
Set<String> set = new HashSet<>();
Map<String, Integer> map = new HashMap<>();
```

**Memory Tip:** “Three main families: **List = index**, **Set = uniqueness**, **Map = key→value**.”

---

## 3. Iterable and Iterator

### 3.1 Iterable

Any class that can be used in enhanced for-loop implements `Iterable<T>`.

```java
public interface Iterable<T> {
    Iterator<T> iterator();
}
```

Example:

```java
for (String s : list) {      // list is Iterable
    System.out.println(s);
}
```

### 3.2 Iterator

Used for safe element traversal and removal while iterating.[web:17][web:22]  

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    if (s.isEmpty()) {
        it.remove();         // safe removal
    }
}
```

**Interview Trap:** Removing from a List using `list.remove()` inside enhanced for-loop causes `ConcurrentModificationException`. Use `Iterator.remove()` instead.

**Memory Tip:** “**I**terable = ‘I can give you an **Iterator**’; **Iterator** = walk and (optionally) remove.”

---

## 4. List Implementations

### 4.1 ArrayList

- Backed by a growable array.  
- Fast random access: `get(i)` and `set(i)` are O(1).  
- Inserting/removing in middle is O(n) due to shifting.[web:17][web:22]  

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
String first = list.get(0);
```

### 4.2 LinkedList

- Doubly-linked list.  
- No index-based O(1) access (traversal required).  
- Efficient insert/remove at ends and when you already have `ListIterator` at position.[web:17][web:22]  

```java
List<String> list = new LinkedList<>();
list.add("A");
list.add(0, "B");       // cheap at head
```

### 4.3 Use-cases & Performance (ArrayList vs LinkedList)

| Aspect                  | ArrayList                                 | LinkedList                               |
|-------------------------|--------------------------------------------|------------------------------------------|
| Storage                 | Dynamic array                             | Doubly-linked nodes                      |
| Random access get(i)    | O(1) – very fast                          | O(n) – must traverse                     |
| Insert/remove middle    | O(n) – shift elements                     | O(1) if position known iterator-wise     |
| Insert/remove at ends   | Amortized O(1) at tail, O(n) at head      | O(1) at head/tail                        |
| Typical use             | Read‑heavy, index‑based operations        | Many inserts/removes at ends/middle      |[web:17][web:22]  

**Interview Trap:** Many interviews expect “Use ArrayList in 99% of cases; LinkedList rarely gives practical wins due to cache locality and iterator cost.” Mention this nuance.

**Memory Tip:** “Array**List** = random **List** access; Linked**List** = **Linked** nodes, good for additions.”

---

## 5. Set Implementations

### 5.1 HashSet

- Backed by `HashMap<E, Object>` internally.  
- No order guarantee.  
- `null` allowed once.  
- O(1) average add/contains/remove (good hashCode).[web:17][web:22]  

```java
Set<String> set = new HashSet<>();
set.add("A");
set.add("A");      // ignored (no duplicate)
```

### 5.2 LinkedHashSet

- Maintains insertion order using linked list over hash buckets.[web:19][web:22]  
- Slightly more memory than HashSet, similar O(1) ops.  

```java
Set<String> set = new LinkedHashSet<>();
set.add("B");
set.add("A");
System.out.println(set); // [B, A]
```

### 5.3 TreeSet

- Backed by `TreeMap` (Red‑Black tree).  
- Stores elements in sorted order (natural or custom Comparator).[web:17][web:22]  
- O(log n) add/contains/remove.  

```java
Set<String> set = new TreeSet<>();
set.add("B");
set.add("A");
System.out.println(set); // [A, B]
```

**Memory Tip:** “**Hash**Set = hash + fast; **Linked**HashSet = hash + order; **Tree**Set = sorted like a tree.”

---

## 6. Map Implementations

### 6.1 HashMap

- Most commonly used Map.  
- Key-based access with hash table + buckets.  
- Allows one `null` key and multiple `null` values.[web:15][web:22]  

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
Integer val = map.get("A");
```

Important points:  

- Good `hashCode()` and `equals()` on key are **critical**.  
- In Java 8+, buckets with many collisions may turn into balanced trees for performance.[web:22]  

### 6.2 LinkedHashMap

- HashMap + predictable iteration order (insertion or access order).[web:19][web:22]  
- Common for caches (LRU using `removeEldestEntry`).  

```java
Map<String, Integer> map = new LinkedHashMap<>(16, 0.75f, true); // access-order
```

### 6.3 TreeMap

- Sorted Map using Red‑Black tree.  
- Keys must be mutually comparable (Comparable or Comparator).[web:19][web:22]  

```java
Map<String, Integer> map = new TreeMap<>();
map.put("B", 2);
map.put("A", 1);      // iteration order: A, B
```

### 6.4 ConcurrentHashMap Basics

- Thread-safe Map optimized for concurrent access.  
- Uses internal partitioning / finer-grained locking instead of single global lock (segmentation in older JDK, CAS & bins in newer).[web:20][web:23][web:27][web:25]  
- Concurrent reads are non-blocking, writes use limited locking per bucket/segment.  

```java
Map<String, Integer> map = new ConcurrentHashMap<>();
map.put("A", 1);
map.computeIfAbsent("B", k -> 2);
```

Key behaviors:[web:20][web:23][web:27][web:25]  

- Iterators are **weakly consistent**:  
  - No `ConcurrentModificationException`.  
  - May not reflect all updates during iteration.  
- `null` keys/values are not allowed.  

**Memory Tip:** “HashMap = not thread-safe, allow null; **Concurrent**HashMap = thread-safe, **no** null.”

---

## 7. Sorting and Comparison

### 7.1 Comparable Interface

Used for natural ordering (class defines its own default sort order).[web:19][web:22]  

```java
class Employee implements Comparable<Employee> {
    int id;
    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);  // natural order by id
    }
}
```

**Rule:** implement `Comparable<T>` when there is a single obvious ordering (e.g., `String`, `Integer`).[web:19][web:22]  

### 7.2 Comparator Interface

Used for custom, external sorting logic.[web:19][web:22]  

```java
Comparator<Employee> byName =
        (e1, e2) -> e1.name.compareTo(e2.name);
```

Since Java 8, you can chain comparators:

```java
Comparator<Employee> byDeptThenName =
        Comparator.comparing(Employee::getDept)
                  .thenComparing(Employee::getName);
```

### 7.3 Collections.sort and List.sort

```java
List<String> list = new ArrayList<>(List.of("B", "A"));

// Natural order (Comparable)
Collections.sort(list);

// Custom order (Comparator)
list.sort((s1, s2) -> s2.compareTo(s1));   // reverse
```

- `Collections.sort(list)` delegates to `list.sort(null)` (for Lists).  
- Under the hood uses modified TimSort (good with partly sorted data).[web:22]  

**Memory Tip:** “**Comparable** = class **implements** order; **Comparator** = external **strategy** for order.”

---

## 8. Generics Fundamentals

### 8.1 Generic Classes and Methods

Generic class:

```java
class Box<T> {
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}
```

Generic method:

```java
public static <T> T first(List<T> list) {
    return list.isEmpty() ? null : list.get(0);
}
```

Benefits: type safety (no casts), better readability, fewer `ClassCastException` at runtime.[web:21][web:28]  

### 8.2 Type Parameters (E, T, K, V)

Conventions:[web:21][web:28]  

- `E` – Element (collections).  
- `T` – Type (generic class/method).  
- `K` – Key, `V` – Value (maps).  
- `?` – unknown type (wildcard).  

**Memory Tip:** “Just remember **K/V**, **E**, **T** = Key/Value, Element, Type.”

### 8.3 Wildcards: ?, ? extends, ? super

#### 8.3.1 Unbounded wildcard `?`

Use when you don’t care about the exact type, only that it’s some type.

```java
void printAll(List<?> list) {
    for (Object o : list) {
        System.out.println(o);
    }
}
```

Cannot add elements (except `null`) because type is unknown.[web:28]  

#### 8.3.2 Upper-bounded `? extends T`

“Producer Extends” – good when you only **read** (produce) `T`.[web:21][web:28]  

```java
void sum(List<? extends Number> nums) {
    double total = 0;
    for (Number n : nums) total += n.doubleValue();
    // nums.add(1); // compile error
}
```

You can safely **get** `Number`, but **cannot** add because real type may be `List<Integer>`, `List<Double>`, etc.

#### 8.3.3 Lower-bounded `? super T`

“Consumer Super” – good when you **write** (consume) `T`.[web:21][web:28]  

```java
void addInts(List<? super Integer> list) {
    list.add(1);
    list.add(2);
    // Object obj = list.get(0); // only safe as Object
}
```

You can add `Integer` (or subclass) safely, but only read as `Object`.

#### 8.3.4 PECS Rule

- **P**roducer **E**xtends – use `? extends T` when method only reads.  
- **C**onsumer **S**uper – use `? super T` when method only writes.[web:21][web:24][web:26]  

**Memory Tip:** “**PECS**: Producer Extends, Consumer Super – read with extends, write with super.”

### 8.4 Type Erasure (Concept)

- Generics are compile-time only; at runtime they are “erased” to raw types.[web:21][web:28]  
- Example: `List<String>` and `List<Integer>` both become just `List` at runtime.  
- No new class per type parameter; avoids type explosion, keeps backward compatibility.  

Consequences:  

- Cannot do `new T()`, `T[] arr = new T [dzone](https://dzone.com/articles/how-concurrenthashmap-works-internally-in-java)`.  
- Cannot use primitives as type arguments (`List<int>` invalid, use `List<Integer>`).  
- Cannot overload only by generic parameter types (erasure makes signatures same).  

**Interview Trap:** “Why `if (list instanceof List<String>)` is illegal?” – due to erasure, use `instanceof List<?>` only.

**Memory Tip:** “Generics live at **compile-time**, disappear at **run-time**.”

---

## 9. Experienced-Level Q&A (4+ Years)

Short and focused answers, tuned for 4 years experience.

### Q1. Core interfaces of Collections Framework?

**A:** `Collection` (with subinterfaces `List`, `Set`, `Queue`) and separate `Map` hierarchy with `SortedSet`, `NavigableSet`, `SortedMap`, `NavigableMap` as specialized views.[web:19][web:22]  

**Trap:** People say “Collection & Map” but forget Sorted/Navigable specializations.

---

### Q2. When do you choose ArrayList vs LinkedList in a real project?

**A:** Prefer `ArrayList` almost always: gives O(1) random access, good cache locality, and even for inserts it is usually faster in practice.[web:17][web:22] Use `LinkedList` only when you need many insertions/removals at ends and you’re not accessing by index; even then, measure performance.  

**Tip:** Mention GC and memory overhead of `LinkedList` nodes.

---

### Q3. HashSet vs TreeSet vs LinkedHashSet?

**A:**  

- `HashSet`: fastest O(1) operations, no ordering guarantees.  
- `LinkedHashSet`: preserves insertion order (or access order for `LinkedHashMap`), slight overhead.  
- `TreeSet`: sorted order with O(log n) operations, uses compareTo/Comparator.[web:17][web:19][web:22]  

**Trap:** TreeSet requires elements to be mutually comparable; mixing types causes `ClassCastException`.

---

### Q4. HashMap vs LinkedHashMap vs TreeMap vs ConcurrentHashMap?

**A:**  

- `HashMap`: general-purpose, no order.  
- `LinkedHashMap`: order + easy LRU cache via `removeEldestEntry`.  
- `TreeMap`: sorted keys for range queries (`subMap`, `headMap`).  
- `ConcurrentHashMap`: high-concurrency, thread-safe map (no global lock, no nulls, weakly consistent iterators).[web:20][web:23][web:27][web:25]  

**Trap:** Saying “ConcurrentHashMap uses one lock per bucket and segments = capacity” – this was roughly true pre‑Java 8; now it uses different locking schemes, so phrase as “fine-grained locking and CAS, not a single lock”.

---

### Q5. Internal working of HashMap (high-level)?

**A:**  

- Computes hash from key’s `hashCode()` (with spread).  
- Maps it to bucket index.  
- Stores key–value as node in bucket (linked list or tree).  
- On collision, adds node to bucket chain (or tree from Java 8).  
- Resize when size exceeds `capacity * loadFactor`.[web:15][web:22]  

**Trap:** Always mention importance of good `hashCode()` and `equals()` for keys.

---

### Q6. How does ConcurrentHashMap improve over Hashtable / synchronizedMap?

**A:**  

- No single lock on the whole map.  
- Uses internal sharding / bucket-level locks and CAS so multiple threads can update different parts in parallel.  
- Reads are mostly lock-free, iterators are weakly consistent and don’t throw `ConcurrentModificationException`.[web:20][web:23][web:27][web:25]  

**Tip:** Call out that `size()` may be approximate under high concurrency (in older versions).

---

### Q7. Real use of `? extends` and `? super` in APIs?

**A:**  

- `? extends T` for producers, e.g. `copy(List<? extends T> src, List<? super T> dest)` – you read from src.  
- `? super T` for consumers where you add T values.[web:21][web:28]  

Example:

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (int i = 0; i < src.size(); i++) {
        dest.set(i, src.get(i));
    }
}
```

---

### Q8. Why generics are not reified? What problems does type erasure cause?

**A:**  

- Generics use type erasure to maintain backward compatibility with pre‑Java 5 code and keep bytecode/class file size manageable.[web:21][web:28]  
- Problems: cannot `new T()`, cannot use primitives, cannot do `instanceof` with parameterized type, confusing error messages, bridge methods.  

**Tip:** Mention that this is why frameworks like Spring use `TypeToken` or reflection tricks to recover type info.

---

### Q9. How would you design a type-safe API to sort entities using generics?

**A:**  

- Use `Comparable<T>` on entity when natural order exists.  
- Provide overloaded methods with `Comparator<? super T>` for custom order.  

```java
public <T extends Comparable<? super T>> void sortNatural(List<T> list) {
    Collections.sort(list);
}

public <T> void sortWithComparator(List<T> list, Comparator<? super T> cmp) {
    list.sort(cmp);
}
```

---

### Q10. Common concurrency pitfalls with Collections?

**A:**  

- Modifying non-thread-safe collections (ArrayList, HashMap) from multiple threads without external synchronization.  
- Iterating and modifying from multiple threads causing `ConcurrentModificationException`.  
- Using `Collections.synchronizedXXX` but still iterating without synchronizing on the wrapper.  
- Using `ConcurrentHashMap` but expecting atomic “check then act” without using proper APIs (`compute`, `putIfAbsent`).[web:22][web:25][web:27]  

---

## 10. Common Traps & Memory Tips

### 10.1 Traps

- Overriding `equals()` but not `hashCode()` → broken HashMap/HashSet behavior.  
- Using mutable objects as keys in HashMap and then mutating fields that participate in `equals()/hashCode()`.  
- Using raw types `List` instead of parameterized `List<String>` – loses type safety and may cause runtime `ClassCastException`.[web:17][web:22]  
- Using `List<Number> nums = new ArrayList<Integer>();` – illegal, generics are **invariant**.[web:15][web:21][web:28]  

### 10.2 One-line Memory Tips

- “List = **index**, Set = **unique**, Map = **key→value**.”  
- “Hash = fast, Tree = sorted, Linked = ordered.”  
- “PECS: Producer **Extends**, Consumer **Super** for wildcards.”[web:21][web:24][web:26][web:28]  
- “Generics: compile-time **safety**, runtime **erasure**.”[web:21][web:28]  
- “ConcurrentHashMap: **no nulls**, **no global lock**, **weakly consistent** iterators.”[web:20][web:23][web:27][web:25]  

---
```