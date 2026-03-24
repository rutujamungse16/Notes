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

The Java Collections Framework is a set of interfaces and classes to store and manipulate groups of objects in a consistent way.

- Main root interfaces: `Collection` and `Map` (Map is a separate hierarchy).  
- Ready-made data structures: List, Set, Queue, Map etc.  
- Algorithms: sorting, searching, shuffling in `Collections` utility class.  

**Memory Tip:** “Collection = container for objects; `Collections` (with s) = helper static methods.”

---

## 2. Core Collection and Map Interfaces

### 2.1 Collection hierarchy

- `Collection<E>` is the root for:  
  - `List<E>` – ordered, index-based, allows duplicates.  
  - `Set<E>` – no duplicates, models mathematical set.  
  - `Queue<E>` / `Deque<E>` – for FIFO/LIFO style access.  

### 2.2 Map hierarchy

- `Map<K,V>` – key–value pairs, keys are unique.  
- Specialized subinterfaces:  
  - `SortedMap<K,V>` / `NavigableMap<K,V>` for sorted maps and range views.  

**Code glimpse:**

```java
List<String> list = new ArrayList<>();
Set<String> set = new HashSet<>();
Map<String, Integer> map = new HashMap<>();
```

**Memory Tip:** “Three main families: List = index, Set = uniqueness, Map = key→value.”

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

Used for safe element traversal and removal while iterating.

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

**Memory Tip:** “Iterable = ‘I can give you an Iterator’; Iterator = walk and (optionally) remove.”

---

## 4. List Implementations

### 4.1 ArrayList

- Backed by a growable array.  
- Fast random access: `get(i)` and `set(i)` are O(1).  
- Inserting/removing in middle is O(n) due to element shifting.  

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
String first = list.get(0);
```

### 4.2 LinkedList

- Doubly-linked list.  
- No O(1) index-based access (must traverse).  
- Efficient insert/remove at ends and via `ListIterator` at known position.  

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
| Insert/remove middle    | O(n) – shift elements                     | O(1) if position known via iterator      |
| Insert/remove at ends   | Amortized O(1) at tail, O(n) at head      | O(1) at head/tail                        |
| Typical use             | Read-heavy, index-based operations        | Many inserts/removes at ends/middle      |

**Interview Trap:** In real-world benchmarks, ArrayList often outperforms LinkedList even for some insert/remove patterns due to better cache locality and lower object overhead. Always mention “measure before choosing LinkedList”.

**Memory Tip:** “ArrayList = random list access; LinkedList = linked nodes, good for additions.”

---

## 5. Set Implementations

### 5.1 HashSet

- Backed by `HashMap<E, Object>` internally.  
- No order guarantee.  
- `null` allowed once.  
- O(1) average add/contains/remove if hash function is good.  

```java
Set<String> set = new HashSet<>();
set.add("A");
set.add("A");      // ignored (no duplicate)
```

### 5.2 LinkedHashSet

- Maintains insertion order using a linked list over hash buckets.  
- Slightly more memory than HashSet, similar average O(1) operations.  

```java
Set<String> set = new LinkedHashSet<>();
set.add("B");
set.add("A");
System.out.println(set); // [B, A]
```

### 5.3 TreeSet

- Backed by `TreeMap` (Red‑Black tree).  
- Stores elements in sorted order (natural or Comparator).  
- O(log n) add/contains/remove.  

```java
Set<String> set = new TreeSet<>();
set.add("B");
set.add("A");
System.out.println(set); // [A, B]
```

**Memory Tip:** “HashSet = hash + fast; LinkedHashSet = hash + order; TreeSet = sorted like a tree.”

---

## 6. Map Implementations

### 6.1 HashMap

- Most commonly used Map.  
- Key-based access with hash table + buckets.  
- Allows one `null` key and multiple `null` values.  

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
Integer val = map.get("A");
```

Important details:

- Uses `hashCode()` and `equals()` of keys for bucket selection and equality.  
- In Java 8+, buckets with many collisions may be converted from linked lists to balanced trees for better worst-case performance.  

### 6.2 LinkedHashMap

- HashMap + predictable iteration order (insertion or access order).  
- Useful for caches (LRU via `removeEldestEntry`).  

```java
Map<String, Integer> map = new LinkedHashMap<>(16, 0.75f, true); // access-order
```

### 6.3 TreeMap

- Sorted Map using Red‑Black tree.  
- Keys must be mutually comparable (Comparable or Comparator).  

```java
Map<String, Integer> map = new TreeMap<>();
map.put("B", 2);
map.put("A", 1);      // iteration order: A, B
```

### 6.4 ConcurrentHashMap Basics

- Thread-safe Map optimized for concurrent access.  
- Uses internal fine-grained locking and CAS operations instead of a single global lock.  
- Concurrent reads are non-blocking, writes use limited locking per bucket/segment.  

```java
Map<String, Integer> map = new ConcurrentHashMap<>();
map.put("A", 1);
map.computeIfAbsent("B", k -> 2);
```

Key behaviors:

- Iterators are weakly consistent:  
  - No `ConcurrentModificationException`.  
  - May not reflect all updates during iteration.  
- `null` keys/values are not allowed.  

**Memory Tip:** “HashMap = not thread-safe, allows null; ConcurrentHashMap = thread-safe, no null.”

---

## 7. Sorting and Comparison

### 7.1 Comparable Interface

Used for natural ordering (class defines its own default sort order).

```java
class Employee implements Comparable<Employee> {
    int id;
    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);  // natural order by id
    }
}
```

Use `Comparable<T>` when there is a single obvious ordering (e.g., `String`, `Integer`).

### 7.2 Comparator Interface

Used for custom, external sorting logic.

```java
Comparator<Employee> byName =
        (e1, e2) -> e1.name.compareTo(e2.name);
```

Java 8 style chaining:

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

- `Collections.sort(list)` delegates to `list.sort(null)` for Lists.  
- Under the hood uses a variant of TimSort, efficient on partially sorted data.

**Memory Tip:** “Comparable = class implements order; Comparator = external strategy for order.”

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

Benefits: type safety (no casts), better readability, fewer `ClassCastException` at runtime.

### 8.2 Type Parameters (E, T, K, V)

Common conventions:

- `E` – Element (collections).  
- `T` – Type (generic class/method).  
- `K` – Key, `V` – Value (maps).  
- `?` – unknown type (wildcard).  

**Memory Tip:** “Just remember K/V, E, T = Key/Value, Element, Type.”

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

You cannot add elements (except `null`) because the exact element type is unknown.

#### 8.3.2 Upper-bounded `? extends T`

“Producer Extends” – good when you only read (produce) `T`.

```java
void sum(List<? extends Number> nums) {
    double total = 0;
    for (Number n : nums) total += n.doubleValue();
    // nums.add(1); // compile error
}
```

You can safely read `Number`, but cannot add because the concrete subtype may be `Integer`, `Double`, etc.

#### 8.3.3 Lower-bounded `? super T`

“Consumer Super” – good when you write (consume) `T`.

```java
void addInts(List<? super Integer> list) {
    list.add(1);
    list.add(2);
    // Object obj = list.get(0); // only safe as Object
}
```

You can add `Integer` safely, but you can only read values as `Object`.

#### 8.3.4 PECS Rule

- Producer Extends – use `? extends T` when method only reads.  
- Consumer Super – use `? super T` when method only writes.  

**Memory Tip:** “PECS: Producer Extends, Consumer Super – read with extends, write with super.”

### 8.4 Type Erasure (Concept)

- Generics are compile-time only; at runtime they are “erased” to raw types.  
- Example: `List<String>` and `List<Integer>` both become just `List` at runtime.  
- No new class per type parameter; avoids type explosion and keeps backward compatibility.  

Consequences:

- Cannot do `new T()`, `T[] arr = new T[10]`.  
- Cannot use primitives as type arguments (`List<int>` invalid, use `List<Integer>`).  
- Cannot overload only by generic parameter types (erasure makes signatures same).  

**Interview Trap:** `if (list instanceof List<String>)` is illegal; use `instanceof List<?>` instead.

**Memory Tip:** “Generics live at compile-time, disappear at run-time.”

---

## 9. Experienced-Level Q&A (4+ Years)

Short and focused answers suitable for 4 years experience.

### Q1. Core interfaces of Collections Framework?

**A:**  
`Collection` (with subinterfaces `List`, `Set`, `Queue`) plus a separate `Map` hierarchy. Specialized views like `SortedSet`, `NavigableSet`, `SortedMap`, `NavigableMap` provide sorted and range operations.

---

### Q2. When do you choose ArrayList vs LinkedList in a real project?

**A:**  
Prefer `ArrayList` in most cases: O(1) random access, better cache locality, smaller memory footprint. Use `LinkedList` only when you have many insertions/removals at ends and few random accesses, and ideally after measuring performance.

---

### Q3. HashSet vs TreeSet vs LinkedHashSet?

**A:**  

- `HashSet`: fastest O(1) operations, no ordering guarantees.  
- `LinkedHashSet`: preserves insertion order, slightly more memory.  
- `TreeSet`: sorted order with O(log n) operations, requires elements to be comparable.  

**Interview Trap:** TreeSet elements must be mutually comparable; mixing incomparable types causes `ClassCastException`.

---

### Q4. HashMap vs LinkedHashMap vs TreeMap vs ConcurrentHashMap?

**A:**  

- `HashMap`: general-purpose, unsorted, non-thread-safe.  
- `LinkedHashMap`: preserves insertion or access order, good for LRU caches.  
- `TreeMap`: sorted keys, useful for range queries (`subMap`, `headMap`, `tailMap`).  
- `ConcurrentHashMap`: thread-safe with fine-grained locking, allows high concurrency, no `null` keys/values.  

---

### Q5. Internal working of HashMap (high-level)?

**A:**  
HashMap uses the key’s `hashCode()` (after some bit spreading) to compute a bucket index, stores entries in that bucket as nodes (linked list or tree), and on collisions chains nodes in the same bucket. When size exceeds `capacity * loadFactor`, it resizes and rehashes the entries.

---

### Q6. How does ConcurrentHashMap improve over Hashtable / synchronizedMap?

**A:**  
Instead of one global lock, ConcurrentHashMap uses internal buckets and fine-grained locks/CAS so different threads can operate on different parts concurrently. Reads are mostly lock-free, iterators are weakly consistent and do not throw `ConcurrentModificationException`.

---

### Q7. Real use of `? extends` and `? super` in APIs?

**A:**  
Use `? extends T` for read-only sources and `? super T` for write targets, e.g.:

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (int i = 0; i < src.size(); i++) {
        dest.set(i, src.get(i));
    }
}
```

Source “produces” values (`extends`), destination “consumes” them (`super`).

---

### Q8. Why generics are not reified? What problems does type erasure cause?

**A:**  
Generics use type erasure to keep backward compatibility with older Java and avoid extra runtime type information. This means you cannot create generic arrays, cannot use primitives as type arguments, cannot use `instanceof` with specific parameterized types, and overloaded methods differing only by generic parameters are not allowed.

---

### Q9. How would you design a type-safe API to sort entities using generics?

**A:**  

```java
public <T extends Comparable<? super T>> void sortNatural(List<T> list) {
    Collections.sort(list);
}

public <T> void sortWithComparator(List<T> list, Comparator<? super T> cmp) {
    list.sort(cmp);
}
```

Natural sort uses `Comparable`; custom sort uses `Comparator<? super T>` so supertypes can define comparators.

---

### Q10. Common concurrency pitfalls with Collections?

**A:**  

- Modifying non-thread-safe collections (ArrayList, HashMap) from multiple threads without external synchronization.  
- Iterating and modifying the same collection from multiple threads, causing `ConcurrentModificationException`.  
- Using `Collections.synchronizedXXX` but iterating without synchronizing on the wrapper.  
- Using `ConcurrentHashMap` but implementing multi-step “check-then-act” without atomic methods like `compute`, `merge`, `putIfAbsent`.

---

## 10. Common Traps & Memory Tips

### 10.1 Traps

- Overriding `equals()` but not `hashCode()` → broken behavior in HashMap/HashSet.  
- Using mutable objects as keys in HashMap and mutating fields that affect `equals()/hashCode()`.  
- Using raw types `List` instead of `List<String>` – loses type safety and can cause runtime `ClassCastException`.  
- Assuming `List<Integer>` is a subtype of `List<Number>` (generics are invariant).  

### 10.2 One-line Memory Tips

- “List = index, Set = unique, Map = key→value.”  
- “Hash = fast, Tree = sorted, Linked = ordered.”  
- “PECS: Producer Extends, Consumer Super – read with extends, write with super.”  
- “Generics: compile-time safety, runtime erasure.”  
- “ConcurrentHashMap: no nulls, no global lock, weakly consistent iterators.”

---
```