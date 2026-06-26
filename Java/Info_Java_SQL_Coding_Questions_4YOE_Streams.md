# Infosys Interview — Top Java & SQL Coding Questions (4 Years Experience)
### With Detailed Logic Explanations, Java 8 Streams & Lambdas

This guide compiles the most frequently asked **Java coding** and **SQL coding** questions in Infosys technical interviews for developers with ~4 years of experience, based on recent (2025–2026) candidate-reported interview patterns. Every question includes a **traditional/loop-based solution**, a **Java 8 Streams + Lambda equivalent** where applicable, and a **detailed line-by-line explanation of the logic** — not just the code.

---

# PART A: JAVA CODING QUESTIONS

## Section 1: Arrays & Collections (Core Logic + Streams)

### 1. Find duplicate elements in an array/list.

**Traditional approach:**
```java
public static List<Integer> findDuplicates(int[] arr) {
    Set<Integer> seen = new HashSet<>();
    List<Integer> duplicates = new ArrayList<>();
    for (int num : arr) {
        if (!seen.add(num)) {
            duplicates.add(num);
        }
    }
    return duplicates;
}
```

**Java 8 Streams approach:**
```java
List<Integer> duplicates = Arrays.stream(arr)
    .boxed()
    .collect(Collectors.groupingBy(n -> n, Collectors.counting()))
    .entrySet().stream()
    .filter(entry -> entry.getValue() > 1)
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());
```

**Logic explained in detail:**
- `Arrays.stream(arr).boxed()` converts the primitive `int[]` into a `Stream<Integer>`, because collectors like `groupingBy` need object types, not primitives.
- `Collectors.groupingBy(n -> n, Collectors.counting())` is the key step — it groups every element by itself (the classifier `n -> n` means "group by the value"), and for each group, the downstream collector `Collectors.counting()` counts how many times that value occurred. The result is a `Map<Integer, Long>` like `{5=2, 3=1, 7=3}`.
- We then stream over this map's `entrySet()`, `filter()` keeps only entries where the count (`entry.getValue()`) is greater than 1 — meaning the number repeated.
- `.map(Map.Entry::getKey)` extracts just the number itself (discarding the count), since that's what we want to return.
- **Why this matters in interviews**: this question tests whether you understand `groupingBy` with a downstream collector, which is one of the most commonly probed Stream API patterns.
- **Complexity**: O(n) time, O(n) space either way.

### 2. Remove duplicate elements from a list using Streams.
```java
List<Integer> uniqueList = list.stream()
    .distinct()
    .collect(Collectors.toList());
```
**Logic explained**: `distinct()` is an intermediate stream operation that uses each element's `equals()`/`hashCode()` to filter out elements that have already been seen earlier in the stream — internally it behaves similarly to checking membership in a `HashSet` as it processes elements in encounter order. It's stateful (it must remember what it's already seen) but is the cleanest way to deduplicate in one line.

### 3. Find the second largest number in an integer array.

**Traditional approach:**
```java
public static int secondLargest(int[] arr) {
    int largest = Integer.MIN_VALUE, secondLargest = Integer.MIN_VALUE;
    for (int num : arr) {
        if (num > largest) {
            secondLargest = largest;
            largest = num;
        } else if (num > secondLargest && num != largest) {
            secondLargest = num;
        }
    }
    return secondLargest;
}
```

**Java 8 Streams approach:**
```java
int secondLargest = Arrays.stream(arr)
    .boxed()
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst()
    .orElseThrow(() -> new NoSuchElementException("No second largest element"));
```

**Logic explained in detail:**
- `.distinct()` first removes duplicate values — without this, an array like `[5, 5, 3]` would incorrectly treat the second `5` as the "second largest" instead of `3`.
- `.sorted(Comparator.reverseOrder())` sorts the stream in descending order, so the largest element is now at index 0, second largest at index 1, and so on.
- `.skip(1)` discards the first element (the largest), leaving the second largest at the front of the remaining stream.
- `.findFirst()` is the terminal operation that pulls out that front element, wrapped in an `Optional<Integer>` (since the stream could theoretically be empty if there's no second-largest element).
- `.orElseThrow(...)` unwraps the `Optional`, throwing a clear exception if no second-largest value exists (e.g., the array had only one distinct value).
- **Why interviewers like this question**: it tests whether you remember `distinct()` must come before sorting/skipping (a common bug), and whether you handle the `Optional` correctly instead of just calling `.get()` blindly.

### 4. Find common elements between two arrays.
```java
List<Integer> common = Arrays.stream(arr1)
    .filter(x -> Arrays.stream(arr2).anyMatch(y -> y == x))
    .boxed()
    .collect(Collectors.toList());
```
**Logic explained**: for every element `x` in the first array, `filter()` keeps it only if `anyMatch()` finds at least one matching element `y` in the second array. `anyMatch()` is a **short-circuiting** terminal operation — it stops scanning `arr2` as soon as it finds one match, rather than checking every element. **Caveat to mention in the interview**: this nested-stream approach is O(n×m) — for better performance on large arrays, convert `arr2` to a `Set` first and use `set.contains(x)` instead, which brings it down to O(n+m).

### 5. Sort a list of strings by their length using Streams.
```java
List<String> sorted = strings.stream()
    .sorted(Comparator.comparingInt(String::length))
    .collect(Collectors.toList());
```
**Logic explained**: `Comparator.comparingInt(String::length)` builds a comparator that extracts each string's length (via the method reference `String::length`, equivalent to the lambda `s -> s.length()`) and compares strings based on that extracted key, rather than their natural (alphabetical) ordering. To sort by length descending, you'd chain `.reversed()`: `Comparator.comparingInt(String::length).reversed()`.

### 6. Sort a list of custom objects (e.g., `Employee`) by multiple fields using Streams.
```java
List<Employee> sorted = employees.stream()
    .sorted(Comparator.comparing(Employee::getDepartment)
                       .thenComparing(Employee::getSalary, Comparator.reverseOrder()))
    .collect(Collectors.toList());
```
**Logic explained**: `Comparator.comparing(Employee::getDepartment)` establishes the primary sort key (department, ascending alphabetically by default). `.thenComparing(Employee::getSalary, Comparator.reverseOrder())` adds a secondary sort key — when two employees are in the same department (a "tie" on the first key), they're then sorted by salary descending. This chaining pattern (`comparing().thenComparing()`) is a very frequently tested Stream API skill since multi-field sorting comes up constantly in real reporting/business logic.

### 7. Find the frequency of each element in an array/list using Streams.
```java
Map<Integer, Long> frequencyMap = Arrays.stream(arr)
    .boxed()
    .collect(Collectors.groupingBy(n -> n, Collectors.counting()));
```
**Logic explained**: same `groupingBy` + `counting()` pattern as Question 1, but here we want the full frequency map itself rather than filtering it down to just duplicates. This is one of the single most repeated Java 8 Stream questions across Infosys-style interviews because it tests the core "group and aggregate" mental model that maps directly to SQL's `GROUP BY ... COUNT(*)`.

### 8. Find the sum and average of all elements in an integer array using Streams.
```java
int[] arr = {10, 20, 30, 40};
IntSummaryStatistics stats = Arrays.stream(arr).summaryStatistics();
System.out.println("Sum: " + stats.getSum());
System.out.println("Average: " + stats.getAverage());
System.out.println("Max: " + stats.getMax());
System.out.println("Min: " + stats.getMin());
```
**Logic explained**: `IntStream.summaryStatistics()` is a terminal operation that computes count, sum, min, max, and average **in a single pass** over the data, returning an `IntSummaryStatistics` object with getter methods for each. This is more efficient than calling `.sum()` and then separately calling `.average()`, which would require iterating the stream twice (and a stream can only be consumed once anyway — calling a second terminal operation on the same stream throws an `IllegalStateException`).

### 9. Print numbers from a list that are multiples of 5, using Streams.
```java
List<Integer> multiplesOf5 = numbers.stream()
    .filter(n -> n % 5 == 0)
    .collect(Collectors.toList());
```
**Logic explained**: `filter()` is an intermediate, lazy operation — it doesn't actually run until a terminal operation (`collect()` here) is called. Each element is tested against the predicate `n % 5 == 0`, and only elements that return `true` survive into the resulting stream/list.

### 10. Join a list of strings with `[` as prefix, `]` as suffix, and `,` as delimiter.
```java
String result = list.stream()
    .collect(Collectors.joining(", ", "[", "]"));
```
**Logic explained**: `Collectors.joining(delimiter, prefix, suffix)` is a specialized collector purpose-built for exactly this kind of string concatenation — it avoids the awkward manual `StringBuilder` + loop + trailing-comma-removal logic developers often write by hand. For example, `["a", "b", "c"]` becomes the string `"[a, b, c]"`.

---

## Section 2: Strings (Core Logic + Streams)

### 11. Check if a string is a palindrome — including a Streams-based version.

**Traditional approach:**
```java
public static boolean isPalindrome(String str) {
    int left = 0, right = str.length() - 1;
    while (left < right) {
        if (str.charAt(left) != str.charAt(right)) return false;
        left++;
        right--;
    }
    return true;
}
```

**Java 8 Streams approach:**
```java
public static boolean isPalindromeStream(String str) {
    return IntStream.range(0, str.length() / 2)
        .allMatch(i -> str.charAt(i) == str.charAt(str.length() - 1 - i));
}
```
**Logic explained in detail**: `IntStream.range(0, str.length() / 2)` generates indices `0, 1, 2, ...` only up to the **midpoint** of the string — we never need to check past the middle since each comparison covers a pair from both ends. `allMatch()` is a short-circuiting terminal operation: for each index `i`, it checks whether the character at position `i` equals the character at the mirrored position from the end (`length - 1 - i`); the moment any pair doesn't match, it immediately returns `false` without checking the rest. If every pair matches, it returns `true`.

### 12. Check if two strings are anagrams using Streams.
```java
public static boolean areAnagramsStream(String s1, String s2) {
    if (s1.length() != s2.length()) return false;
    char[] arr1 = s1.toCharArray();
    char[] arr2 = s2.toCharArray();
    Arrays.sort(arr1);
    Arrays.sort(arr2);
    return Arrays.equals(arr1, arr2);
}
```
**Logic explained**: two strings are anagrams if and only if they contain exactly the same characters in the same quantities — sorting both strings' characters produces a canonical form where anagrams become byte-for-byte identical arrays, so a simple `Arrays.equals()` check confirms it. The length check upfront is an important short-circuit: two strings of different lengths can never be anagrams, so there's no point sorting them.

### 13. Find the frequency of each character in a string using Streams.
```java
Map<Character, Long> charFrequency = str.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(c -> c, Collectors.counting()));
```
**Logic explained**: `str.chars()` returns an `IntStream` of the character codes in the string (not a `Stream<Character>` directly — this is a common gotcha, since `String` doesn't have a built-in method that streams `Character` objects). `.mapToObj(c -> (char) c)` converts each `int` code point back into a boxed `Character` object so we can use object-based collectors. Then the same `groupingBy` + `counting()` pattern from Question 7 tallies up how many times each character appears.

### 14. Find the first non-repeating character in a string using Streams.
```java
public static Character firstNonRepeatingStream(String str) {
    Map<Character, Long> freq = str.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(c -> c, LinkedHashMap::new, Collectors.counting()));
    return freq.entrySet().stream()
        .filter(entry -> entry.getValue() == 1)
        .map(Map.Entry::getKey)
        .findFirst()
        .orElse(null);
}
```
**Logic explained**: the key detail here is the **three-argument overload** of `groupingBy(classifier, mapFactory, downstream)` — by explicitly supplying `LinkedHashMap::new` as the map factory, we force the resulting map to preserve **insertion order** (the order characters first appeared in the string). A plain `groupingBy(classifier, downstream)` would use a `HashMap` internally, which doesn't guarantee any particular iteration order — and for this problem, "first" non-repeating character is meaningless without preserving original order. Once we have the ordered frequency map, we filter for count `== 1` and take the first match.

### 15. Reverse each word in a sentence using Streams.
```java
String reversed = Arrays.stream(sentence.split(" "))
    .map(word -> new StringBuilder(word).reverse().toString())
    .collect(Collectors.joining(" "));
```
**Logic explained**: `sentence.split(" ")` breaks the sentence into a `String[]` of words, which we stream over. `.map()` transforms each word individually by wrapping it in a `StringBuilder` (which has a built-in `.reverse()` method) and converting it back to a `String`. Finally, `Collectors.joining(" ")` stitches the reversed words back together with single spaces between them, recreating sentence structure.

### 16. Check if a string contains only digits/letters using Streams.
```java
boolean isNumeric = str.chars().allMatch(Character::isDigit);
boolean isAlphabetic = str.chars().allMatch(Character::isLetter);
```
**Logic explained**: `str.chars()` produces an `IntStream` of character codes, and `allMatch(Character::isDigit)` checks — short-circuiting on the first failure — whether every single character satisfies the given predicate (here, a method reference to the static utility method `Character.isDigit(int)`).

---

## Section 3: Linked Lists, Trees & Recursion

### 17. Reverse a singly linked list.
```java
public static ListNode reverseList(ListNode head) {
    ListNode prev = null, current = head;
    while (current != null) {
        ListNode next = current.next;  // save the next node before we overwrite the link
        current.next = prev;            // flip current node's pointer backward
        prev = current;                 // move prev forward to current
        current = next;                 // move current forward to the saved next
    }
    return prev;  // prev now points to the new head (the old tail)
}
```
**Logic explained in detail**: this is one of the most-asked linked list questions because it's deceptively simple but easy to get wrong under pressure. The trick is that you cannot simply do `current.next = prev` first, because doing so would destroy your only reference to the rest of the original list — so you **must** save `current.next` into a temporary variable (`next`) *before* you rewire `current.next`. Each iteration: (1) remember where we were headed, (2) point backward instead, (3) shift both trackers one step forward. When `current` becomes `null` (we've walked off the end), `prev` is sitting on what used to be the last node — now the new head. **Time O(n), Space O(1)** — this in-place property is usually worth mentioning explicitly, since a follow-up question often asks "can you do this without extra space?"

### 18. Detect a cycle in a linked list (Floyd's Algorithm).
```java
public static boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;        // moves 1 step
        fast = fast.next.next;   // moves 2 steps
        if (slow == fast) return true;  // they've met -> cycle exists
    }
    return false;  // fast reached the end -> no cycle
}
```
**Logic explained**: imagine two runners on a circular track, one twice as fast as the other — the faster one will eventually lap the slower one if the track is a loop. If there's **no** cycle, `fast` will simply reach the end of the list (`null`) before ever meeting `slow`, since there's no loop to "lap" around. The `fast != null && fast.next != null` check in the loop condition prevents a `NullPointerException` when `fast` is racing ahead toward the end of a non-cyclic list.

### 19. Merge two sorted linked lists.
```java
public static ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(-1);
    ListNode tail = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) {
            tail.next = l1;
            l1 = l1.next;
        } else {
            tail.next = l2;
            l2 = l2.next;
        }
        tail = tail.next;
    }
    tail.next = (l1 != null) ? l1 : l2;  // attach whichever list still has remaining nodes
    return dummy.next;
}
```
**Logic explained**: the `dummy` node is a classic linked-list trick — it gives us a safe placeholder to start building from, so we don't have to write special-case logic for "is this the very first node I'm attaching?" We always return `dummy.next` at the end (skipping the dummy itself) to get the real merged head. Inside the loop, we compare the current heads of both lists and attach the smaller one to our growing result, advancing only that list's pointer. Once one list is exhausted, the remaining list is already sorted, so we can just attach it wholesale rather than continuing to compare node-by-node.

### 20. Find the Longest Common Subsequence (LCS) of two strings — Dynamic Programming.
```java
public static int lcs(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[m][n];
}
```
**Logic explained in detail**: `dp[i][j]` is defined as "the length of the longest common subsequence between the first `i` characters of `s1` and the first `j` characters of `s2`." The table is 1-indexed on purpose (size `(m+1) × (n+1)`) so that `dp[0][anything]` and `dp[anything][0]` naturally represent "comparing against an empty string," which is always `0` — this avoids messy boundary-condition special-casing.
- **If the current characters match** (`s1.charAt(i-1) == s2.charAt(j-1)`): this character can be part of the common subsequence, so we take the best answer *without* either of these two characters (`dp[i-1][j-1]`) and add 1 for this newly matched character.
- **If they don't match**: this character pair can't both be in the subsequence together, so we take the better of two options — drop the current character from `s1` (`dp[i-1][j]`) or drop the current character from `s2` (`dp[i][j-1]`) — and carry forward whichever was larger.
- The final answer sits in the bottom-right corner, `dp[m][n]`, representing the full comparison of both complete strings.
- **Why Infosys asks this**: it's a clean test of whether you genuinely understand dynamic programming state design, not just memorized code — interviewers often ask you to trace through the table by hand for a small example like `"ABCBDAB"` and `"BDCABA"`.

### 21. Find the Longest Increasing Subsequence (LIS) in an array — Dynamic Programming.
```java
public static int lengthOfLIS(int[] nums) {
    int[] dp = new int[nums.length];
    Arrays.fill(dp, 1);  // every single element is, by itself, an increasing subsequence of length 1
    int maxLen = 1;
    for (int i = 1; i < nums.length; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        maxLen = Math.max(maxLen, dp[i]);
    }
    return maxLen;
}
```
**Logic explained**: `dp[i]` represents "the length of the longest increasing subsequence that **ends exactly at index `i`**." For each index `i`, we look back at every earlier index `j`; if `nums[j] < nums[i]`, that means we could extend whatever subsequence ended at `j` by adding `nums[i]` onto it, giving a candidate length of `dp[j] + 1`. We take the maximum across all valid `j` candidates. The overall answer is the maximum value anywhere in the `dp` array (not necessarily `dp[n-1]`, since the longest subsequence might not end at the very last element). **This is the O(n²) version** — interviewers may ask you to optimize to O(n log n) using binary search with a separate "tails" array, so be ready to mention that as a follow-up.

### 22. Find the height (max depth) of a binary tree using recursion.
```java
public static int maxDepth(TreeNode root) {
    if (root == null) return 0;  // base case: an empty tree has height 0
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```
**Logic explained**: this is a textbook example of recursive "trust the recursion" thinking — you don't need to manually trace every path; you simply trust that `maxDepth(root.left)` correctly returns the height of the left subtree, and likewise for the right. The current node then adds 1 to whichever subtree was taller, since the current node itself is one more level above its tallest child.

---

## Section 4: Concurrency & Design (Frequently Asked at 4 YOE)

### 23. Demonstrate and explain a deadlock scenario, and how to fix it.
```java
public class DeadlockExample {
    static final Object lockA = new Object();
    static final Object lockB = new Object();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            synchronized (lockA) {
                System.out.println("Thread 1: holding lockA");
                try { Thread.sleep(100); } catch (InterruptedException e) {}
                synchronized (lockB) {
                    System.out.println("Thread 1: acquired lockB");
                }
            }
        });
        Thread t2 = new Thread(() -> {
            synchronized (lockB) {
                System.out.println("Thread 2: holding lockB");
                try { Thread.sleep(100); } catch (InterruptedException e) {}
                synchronized (lockA) {
                    System.out.println("Thread 2: acquired lockA");
                }
            }
        });
        t1.start();
        t2.start();
    }
}
```
**Logic explained**: Thread 1 grabs `lockA` and then waits for `lockB`; meanwhile Thread 2 grabs `lockB` and waits for `lockA`. Neither thread will ever release what it's holding, because each is stuck waiting for the *other's* lock — this is called a **circular wait**, and it's one of the four necessary conditions for deadlock (along with mutual exclusion, no preemption, and hold-and-wait). **The fix**: enforce a consistent global lock ordering across all threads — e.g., always require acquiring `lockA` before `lockB`, everywhere in the codebase, regardless of which thread is running. If both threads must follow that order, the circular wait condition becomes impossible.

### 24. Implement a thread-safe Singleton using double-checked locking.
```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {                    // first check (no locking) - fast path
            synchronized (Singleton.class) {
                if (instance == null) {             // second check (with locking) - safety
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
**Logic explained in detail**: the **first `if (instance == null)` check** exists purely for performance — once the instance is created, every subsequent call skips the expensive `synchronized` block entirely and just returns the already-built object. The **second check inside the synchronized block** is the actual safety net: imagine two threads both pass the first check simultaneously (because the instance was still null for both) — only one of them can enter the `synchronized` block at a time; the second thread to enter will see that `instance` is no longer null (the first thread already built it) and will correctly skip re-creating it. The **`volatile` keyword on the field is critical and often the part candidates forget** — without it, due to JVM instruction reordering during object construction, another thread could potentially see a non-null reference to a *partially constructed* object, leading to subtle bugs. `volatile` prevents this by ensuring the write to `instance` is only visible to other threads after the constructor has fully finished.

### 25. Implement a producer-consumer pattern using `wait()`/`notify()` or `BlockingQueue`.
```java
public class ProducerConsumerExample {
    private static final BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10);

    public static void main(String[] args) {
        Thread producer = new Thread(() -> {
            for (int i = 0; i < 20; i++) {
                try {
                    queue.put(i);  // blocks automatically if the queue is full
                    System.out.println("Produced: " + i);
                } catch (InterruptedException e) {}
            }
        });
        Thread consumer = new Thread(() -> {
            for (int i = 0; i < 20; i++) {
                try {
                    int val = queue.take();  // blocks automatically if the queue is empty
                    System.out.println("Consumed: " + val);
                } catch (InterruptedException e) {}
            }
        });
        producer.start();
        consumer.start();
    }
}
```
**Logic explained**: `BlockingQueue` (here `LinkedBlockingQueue`, bounded to capacity 10) internally handles all the thread coordination that you'd otherwise have to write manually with `wait()`/`notify()` and explicit locks. `put()` automatically **blocks the producer thread** if the queue is already full, preventing it from overwhelming the consumer. `take()` automatically **blocks the consumer thread** if the queue is empty, preventing it from consuming data that doesn't exist yet. This is almost always preferred over manual `wait()`/`notify()` in real production code, since hand-rolled implementations are notoriously easy to get subtly wrong (missed notifications, spurious wakeups); interviewers want to see you know the higher-level `java.util.concurrent` tool exists and why it's safer.

---

# PART B: SQL CODING QUESTIONS

## Section 5: SQL Query Writing (Joins, Subqueries, Window Functions)

### 26. Write a query to find the second-highest salary without using `LIMIT`/`TOP`.
```sql
SELECT MAX(salary) AS second_highest_salary
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```
**Logic explained**: the inner subquery `SELECT MAX(salary) FROM employees` finds the single highest salary in the entire table. The outer query then says "now find the maximum salary among everyone who earns *less than* that highest value" — by excluding the true maximum, the next `MAX()` necessarily lands on the second-highest distinct salary. **Why this approach over `LIMIT`/`OFFSET`**: it's portable across databases that don't support `LIMIT` syntax identically (e.g., older SQL Server uses `TOP`), making it a more universal answer interviewers like to see.

### 27. Write a query to find the Nth highest salary using a window function.
```sql
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk = 3;  -- change 3 to whatever N you need
```
**Logic explained**: `DENSE_RANK() OVER (ORDER BY salary DESC)` assigns rank 1 to the highest salary, rank 2 to the next distinct salary, and so on — crucially, **`DENSE_RANK` doesn't skip rank numbers when there are ties** (unlike plain `RANK()`, which would skip, e.g., two people tied for rank 1 means the next person gets rank 3, not 2). This makes `DENSE_RANK` the correct choice when "Nth highest" should mean "Nth highest *distinct* value," which is almost always the intended meaning in interviews. We wrap this in a subquery because window functions can't be directly referenced in a `WHERE` clause in the same query level where they're computed.

### 28. Write a query to find employees who earn more than their department's average salary (correlated subquery).
```sql
SELECT e.emp_id, e.name, e.department, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e.department
);
```
**Logic explained**: this is a **correlated subquery** — notice that the inner query references `e.department` from the *outer* query. This means the inner query cannot be computed once and reused; it must be **re-executed once for every single row** of the outer table, each time computing "what's the average salary specifically within this row's department?" This is different from a plain (non-correlated) subquery, which runs exactly once regardless of how many outer rows there are. **Performance note worth mentioning**: for very large tables, this can be slower than an equivalent join-based approach using a pre-aggregated subquery joined back to the main table, since the correlated version recalculates the department average repeatedly.

### 29. Write a query to find duplicate records in a table based on a combination of columns.
```sql
SELECT name, email, COUNT(*) AS occurrence_count
FROM customers
GROUP BY name, email
HAVING COUNT(*) > 1;
```
**Logic explained**: `GROUP BY name, email` clusters together all rows that share the *same combination* of name and email — this is the key detail, since grouping by just one column wouldn't correctly detect duplicates defined by a pair of columns. `COUNT(*)` then counts how many rows fall into each group. The `HAVING COUNT(*) > 1` clause filters to only groups with more than one row — and crucially, this filtering **must use `HAVING`, not `WHERE`**, because `WHERE` filters individual rows *before* grouping happens, and at that stage the aggregate count doesn't exist yet.

### 30. Write a query to delete duplicate rows while keeping only one copy (using window functions).
```sql
DELETE FROM customers
WHERE id IN (
    SELECT id FROM (
        SELECT id,
               ROW_NUMBER() OVER (PARTITION BY name, email ORDER BY id) AS rn
        FROM customers
    ) sub
    WHERE rn > 1
);
```
**Logic explained**: `ROW_NUMBER() OVER (PARTITION BY name, email ORDER BY id)` assigns a sequential number (1, 2, 3...) to each row *within each group* of matching `name`+`email` combinations, ordered by `id` so the lowest `id` in each group always gets row number 1. Any row with `rn > 1` is, by definition, a duplicate beyond the first occurrence — so deleting all rows where `rn > 1` leaves exactly one (the earliest, lowest-`id`) copy of each duplicate group intact.

### 31. Write a query to find the top 3 highest-paid employees in each department.
```sql
SELECT emp_id, name, department, salary
FROM (
    SELECT emp_id, name, department, salary,
           RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk <= 3;
```
**Logic explained**: `PARTITION BY department` is the crucial piece — it tells the window function to **reset its ranking calculation separately for each department**, rather than ranking across the entire company. So engineering's top earner gets rank 1 within engineering, and sales' top earner *independently* gets rank 1 within sales, regardless of how their actual salary numbers compare to each other. We use `RANK()` here (which does skip numbers on ties, e.g., two people tied for 1st means the next person is rank 3) — if the interviewer specifically wants ties to *not* cause skipped positions, you'd swap in `DENSE_RANK()` instead, as discussed in Question 27.

### 32. Write a query using `LAG()`/`LEAD()` to compare each row with the previous/next row — e.g., month-over-month sales change.
```sql
SELECT month, sales,
       sales - LAG(sales) OVER (ORDER BY month) AS change_from_previous_month
FROM monthly_sales;
```
**Logic explained**: `LAG(sales) OVER (ORDER BY month)` looks "backward" one row (by default) within the result set ordered by month, pulling in the *previous* row's sales value alongside the current row — without `LAG`/`LEAD`, doing this would normally require a self-join on `month - 1`, which is clunkier and less efficient. Subtracting that lagged value from the current row's `sales` gives the change since last month. The very first row will have `NULL` for the lagged value (there's no row before it), which is expected and usually handled with `COALESCE(LAG(sales) OVER (...), 0)` if a zero default is preferred.

### 33. Write a query to calculate a running total (cumulative sum) of sales over time.
```sql
SELECT month, sales,
       SUM(sales) OVER (ORDER BY month) AS running_total
FROM monthly_sales;
```
**Logic explained**: this is the same `SUM()` aggregate function you already know, but the `OVER (ORDER BY month)` clause turns it into a **window function** instead of a `GROUP BY` aggregate — meaning instead of collapsing all rows into one total, it computes a *running* sum that, for each row, includes that row and all rows before it (the default window frame when `ORDER BY` is specified without an explicit frame is "from the start up to the current row"). This avoids writing a correlated subquery or self-join to achieve the same cumulative effect.

### 34. Write a query to find departments where the average salary exceeds the company-wide average salary.
```sql
SELECT department, AVG(salary) AS dept_avg
FROM employees
GROUP BY department
HAVING AVG(salary) > (SELECT AVG(salary) FROM employees);
```
**Logic explained**: the subquery `(SELECT AVG(salary) FROM employees)` computes a single, fixed number — the overall company-wide average — completely independent of any grouping. The outer query groups employees by department and computes each department's own average via `AVG(salary)` in the `HAVING` clause, comparing it against that fixed company-wide number. Since this subquery doesn't reference anything from the outer query's current group, it's a **non-correlated subquery** and only needs to execute once, regardless of how many departments exist — making it more efficient than a correlated alternative.

### 35. Write a query using a CTE (Common Table Expression) to simplify a multi-step query — e.g., finding the top-selling product per category.
```sql
WITH ranked_products AS (
    SELECT product_id, product_name, category, total_sales,
           ROW_NUMBER() OVER (PARTITION BY category ORDER BY total_sales DESC) AS rn
    FROM products
)
SELECT product_id, product_name, category, total_sales
FROM ranked_products
WHERE rn = 1;
```
**Logic explained**: the `WITH ranked_products AS (...)` clause defines a **named, temporary result set** that exists only for the duration of this query — it computes the ranking once and gives it a clear, reusable name. The final `SELECT` then simply filters that named result down to `rn = 1` (the top seller per category). This achieves the exact same result as nesting the ranking subquery directly inside a `FROM (...) AS ranked` clause (like in Question 31), but a CTE is generally considered more readable, especially when a query has multiple sequential steps — and a CTE can be referenced multiple times later in the same query without repeating its definition, which a plain subquery cannot do.

### 36. Write a query to find employees who do NOT have any assigned manager, and separately, employees with no department (handling NULLs correctly).
```sql
SELECT emp_id, name
FROM employees
WHERE manager_id IS NULL;
```
**Logic explained**: this looks trivially simple, but it's a commonly asked "gotcha" question because candidates often mistakenly write `WHERE manager_id = NULL`, which **always returns zero rows** — in SQL, `NULL` represents "unknown," and comparing anything to an unknown value using `=` also yields an unknown (not `true`), so the row is excluded. You must use the special `IS NULL` (or `IS NOT NULL`) operator specifically designed to test for the presence/absence of a NULL value.

### 37. Write a query to find all customers who have never placed an order (using `LEFT JOIN` with `IS NULL`, as an alternative to `NOT IN`).
```sql
SELECT c.customer_id, c.name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```
**Logic explained**: a `LEFT JOIN` keeps **every** row from `customers`, even ones with no matching row in `orders` — for those unmatched customers, every column coming from the `orders` side (like `o.order_id`) will simply be `NULL`. So filtering for `WHERE o.order_id IS NULL` precisely isolates customers for whom *no* matching order row existed at all. **Why this is often preferred over `NOT IN (SELECT customer_id FROM orders)`**: the `NOT IN` approach can silently return zero rows (an easy-to-miss bug) if even a single `customer_id` in the `orders` table happens to be `NULL`, because `NOT IN` against a list containing `NULL` evaluates to unknown for every comparison. The `LEFT JOIN` approach avoids this NULL trap entirely.

### 38. Write a query to pivot row data into columns — e.g., showing total sales per quarter as separate columns.
```sql
SELECT 
    SUM(CASE WHEN quarter = 'Q1' THEN sales ELSE 0 END) AS Q1_sales,
    SUM(CASE WHEN quarter = 'Q2' THEN sales ELSE 0 END) AS Q2_sales,
    SUM(CASE WHEN quarter = 'Q3' THEN sales ELSE 0 END) AS Q3_sales,
    SUM(CASE WHEN quarter = 'Q4' THEN sales ELSE 0 END) AS Q4_sales
FROM quarterly_sales;
```
**Logic explained**: this is the classic "conditional aggregation" pivot pattern, useful in databases (like older MySQL versions) that don't have a native `PIVOT` operator. Each `CASE WHEN quarter = 'Q1' THEN sales ELSE 0 END` expression effectively says "only count this row's sales value toward the Q1 column if it actually belongs to Q1; otherwise contribute zero." Wrapping each of these in `SUM()` then aggregates across all rows, producing one row of output with four separate quarter-total columns instead of four separate rows.

---

## Tips for the Combined Java + SQL Coding Round

- **For Java Streams questions**, always mention the **terminal vs. intermediate** distinction when relevant (e.g., `filter`/`map` are lazy; `collect`/`forEach`/`reduce` actually trigger execution) — interviewers frequently ask this as a quick follow-up.
- **Streams cannot be reused** — calling a second terminal operation on an already-consumed stream throws `IllegalStateException`. Mentioning this proactively signals real hands-on experience, not just memorized syntax.
- **For SQL, always clarify `WHERE` vs. `HAVING`** out loud when filtering on an aggregate — this distinction is one of the most common places candidates lose points even when the final query "looks" right.
- **Window functions (`RANK`, `DENSE_RANK`, `ROW_NUMBER`, `LAG`, `LEAD`) are increasingly central** to Infosys's SQL round even for backend (not just data) roles — make sure you can write the `PARTITION BY` / `ORDER BY` clause from memory, not just recognize it when reading.
- **For DP problems (LCS, LIS)**, narrate your **state definition** (`dp[i][j]` or `dp[i]` means what, exactly?) before writing the transition — interviewers weigh this articulation heavily, often more than the code itself.
- **Practice tracing through a small example by hand** for any DP or two-pointer solution before the interview — being asked to "walk me through this with a small input" live is extremely common.

---

*This document combines current (2025–2026) Infosys candidate-reported Java and SQL coding interview patterns, with particular emphasis on Java 8 Streams/Lambda usage (a heavily tested area at the 4-years-experience level) and SQL window functions/correlated subqueries. Practice writing both the traditional and Streams versions of each Java solution — interviewers sometimes specifically ask "can you also do this with Streams?" as a follow-up even after a correct loop-based answer.*
