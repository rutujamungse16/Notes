# Infosys Interview Questions & Answers — SQL, Database & Coding Round
### Tailored to Rutuja Mungse's Resume (3.6 Years Experience — Java/Spring Boot Developer)

This guide focuses on the **SQL/Database round** and the **live coding round** commonly asked at Infosys for experienced backend developers — both heavily emphasized given your PostgreSQL/MySQL background and migration/transformation project work. Recent candidate reports (2026) confirm Infosys tests SQL query-writing on real schemas, window functions, and live coding (arrays, strings, linked lists, recursion/DP basics) even for experienced hires, alongside project discussion.

---

## Section 1: SQL Fundamentals & Theory

### 1. What is the difference between SQL's DDL, DML, DQL, DCL, and TCL?
- **DDL** (Data Definition Language): defines schema structure — `CREATE`, `ALTER`, `DROP`, `TRUNCATE`.
- **DML** (Data Manipulation Language): manipulates data — `INSERT`, `UPDATE`, `DELETE`.
- **DQL** (Data Query Language): retrieves data — `SELECT`.
- **DCL** (Data Control Language): manages permissions — `GRANT`, `REVOKE`.
- **TCL** (Transaction Control Language): manages transactions — `COMMIT`, `ROLLBACK`, `SAVEPOINT`.

### 2. What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?
`DELETE` removes specific rows (can use `WHERE`), is logged row-by-row, and can be rolled back. `TRUNCATE` removes all rows at once, is minimally logged (faster), resets auto-increment counters, and generally can't be rolled back in most databases once committed. `DROP` removes the entire table structure along with its data permanently.

### 3. What is a Primary Key vs. a Foreign Key? Give an example relevant to a migration/metadata system.
A **Primary Key** uniquely identifies each row and cannot be null (e.g., `asset_id` in a digital assets table). A **Foreign Key** enforces a relationship by referencing another table's primary key (e.g., `metadata.asset_id` referencing `assets.asset_id`), ensuring referential integrity — you can't insert metadata for an asset that doesn't exist.

### 4. What is normalization, and why does it matter when designing a schema for metadata records?
Normalization organizes data into related tables to eliminate redundancy and improve data integrity. For example, instead of repeating asset category names in every metadata row, you'd store categories in a separate table and reference them by ID — reducing duplication and making updates (like renaming a category) a single-row change instead of a mass update.

### 5. What are the differences between 1NF, 2NF, and 3NF?
- **1NF**: each column holds atomic (indivisible) values, no repeating groups.
- **2NF**: 1NF + every non-key column depends on the *whole* primary key (relevant for composite keys).
- **3NF**: 2NF + no transitive dependency — non-key columns depend only on the primary key, not on other non-key columns.

### 6. When would you deliberately denormalize a schema?
For read-heavy reporting/analytics scenarios where join performance becomes a bottleneck — duplicating some data to avoid expensive joins on every read, trading some redundancy and update complexity for significantly faster reads, which can be a reasonable trade-off for high-volume metadata reporting.

### 7. What is the difference between `CHAR` and `VARCHAR`?
`CHAR(n)` is fixed-length — always stores `n` characters, padding with spaces if shorter. `VARCHAR(n)` is variable-length — stores only the actual characters used plus a small length overhead. `VARCHAR` is generally preferred unless the data is genuinely fixed-length (like a 2-letter country code).

### 8. What is ACID in the context of database transactions, and why does it matter for migration jobs?
- **Atomicity**: a transaction either fully completes or fully rolls back.
- **Consistency**: the database moves from one valid state to another.
- **Isolation**: concurrent transactions don't interfere with each other.
- **Durability**: once committed, changes survive even a crash.

For a migration job, atomicity is critical — if a batch transformation partially fails, you want the whole batch rolled back rather than ending up with half-migrated, inconsistent data.

### 9. What are the different types of joins, and when would you use each?
- **INNER JOIN**: only matching rows in both tables.
- **LEFT JOIN**: all rows from the left table, matched rows from the right (NULLs where no match).
- **RIGHT JOIN**: mirror of LEFT JOIN.
- **FULL OUTER JOIN**: all rows from both tables, matched where possible.

Example: showing all assets even if some have no metadata yet would use a `LEFT JOIN` from assets to metadata.

### 10. What is a self-join, and can you give a practical example?
A self-join joins a table to itself, typically to compare rows within the same table — e.g., finding employees who report to the same manager: 
```sql
SELECT e1.name, e2.name AS colleague
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.manager_id AND e1.emp_id <> e2.emp_id;
```

### 11. What is the difference between `WHERE` and `HAVING`?
`WHERE` filters individual rows *before* grouping. `HAVING` filters *after* `GROUP BY` aggregation — used to filter on aggregate values. Example: finding departments with average salary over 50,000 requires `HAVING AVG(salary) > 50000`, since you can't use `WHERE` on an aggregate result.

### 12. What is a subquery, and what is a correlated subquery?
A **subquery** is a query nested inside another query, executed once. A **correlated subquery** references a column from the outer query and is re-executed once *per row* of the outer query. Example: finding employees who earn more than their own department's average salary requires a correlated subquery, since "department average" depends on each row's department.

### 13. What is the difference between a subquery and a JOIN — when would you prefer one over the other?
Both can solve overlapping problems, but a `JOIN` is often more efficient for combining columns from multiple tables since the optimizer can plan it as a single execution path, whereas correlated subqueries that run once per row can be slower on large datasets. Subqueries are clearer for existence checks (`EXISTS`/`IN`) or single-value lookups.

### 14. What is an index, and what's the trade-off of adding one?
An index is a separate data structure (commonly a B-tree) that speeds up row lookups on a column, avoiding a full table scan. The trade-off: indexes speed up reads but slow down writes (`INSERT`/`UPDATE`/`DELETE`) since the index must also be updated, and they consume additional storage — so they should be added selectively on columns that are actually queried/filtered/joined often.

### 15. What is the difference between a clustered and a non-clustered index?
A **clustered index** determines the physical order of data in the table — there can be only one per table (often the primary key). A **non-clustered index** is a separate structure pointing back to the actual rows — a table can have multiple non-clustered indexes.

### 16. What is a view, and why might you use one over a regular table for a reporting use case?
A view is a virtual table based on a stored query — it doesn't store data itself but presents data dynamically each time it's queried. It's useful for hiding complexity (exposing a simplified, pre-joined view of metadata to a reporting tool) or restricting access to only certain columns/rows without exposing the underlying base tables directly.

### 17. What is a stored procedure, and what's a real-world use case in a migration context?
A stored procedure is a reusable, precompiled block of SQL logic stored in the database. A relevant use case: a procedure that validates record counts between a source and target table after a migration batch completes, callable repeatedly without rewriting the validation logic each time.

### 18. What is a trigger, and can you give an example relevant to data integrity in metadata records?
A trigger automatically executes in response to `INSERT`, `UPDATE`, or `DELETE` events. Example: a trigger that automatically logs every update to a metadata record into an audit table, ensuring there's always a change history without relying on the application code to remember to log it.

### 19. What is the difference between `UNION` and `UNION ALL`?
`UNION` combines result sets from two queries and removes duplicate rows (slower, since it must check for duplicates). `UNION ALL` combines them without removing duplicates, making it faster when you know there won't be duplicates or don't care about them.

### 20. What are window functions, and why are they useful for analytical queries on large datasets?
Window functions (`ROW_NUMBER()`, `RANK()`, `LAG()`, `LEAD()`, running `SUM()`/`AVG()` with `OVER(PARTITION BY ...)`) perform calculations across a set of rows related to the current row, without collapsing them into a single grouped result the way `GROUP BY` does — useful for things like ranking records within each category or computing a running total while still returning every original row.

### 21. Write a query using a window function to find the top 3 highest-paid employees in each department.
```sql
SELECT emp_id, name, department, salary
FROM (
    SELECT emp_id, name, department, salary,
           RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk <= 3;
```
`PARTITION BY` resets the ranking per department, so each department independently gets its own top 3.

### 22. What is a CTE (Common Table Expression), and how does it improve query readability?
A CTE (`WITH name AS (...)`) defines a temporary named result set that can be referenced within the main query, making complex queries with multiple steps more readable than deeply nested subqueries, and allowing the same intermediate result to be reused multiple times in the outer query without repeating the subquery.

### 23. Write a CTE-based query to find duplicate records in a metadata table based on a combination of columns.
```sql
WITH duplicates AS (
    SELECT asset_id, file_name,
           COUNT(*) OVER (PARTITION BY file_name, file_size) AS dup_count
    FROM metadata
)
SELECT * FROM duplicates WHERE dup_count > 1;
```

### 24. What is database deadlock, and how would you detect and resolve one in PostgreSQL?
A deadlock happens when two or more transactions each hold a lock the other needs, causing both to wait indefinitely. PostgreSQL automatically detects deadlocks and aborts one of the transactions (returning a deadlock error) so the other can proceed. To avoid them: always acquire locks in a consistent order across transactions, and keep transactions short.

### 25. What is the difference between optimistic and pessimistic locking, and which would you choose for a high-concurrency migration job?
**Pessimistic locking** locks a row immediately when read, preventing other transactions from modifying it until released — safer under heavy contention but reduces concurrency. **Optimistic locking** assumes conflicts are rare and only checks for a conflict (via a version column) at commit time — better throughput when conflicts are infrequent, which fits a scenario where most migrated records aren't touched concurrently by multiple jobs.

### 26. How would you find the second-highest salary in a table without using `LIMIT`/`TOP`?
```sql
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```
This works by excluding the overall maximum, so the next `MAX()` becomes the second-highest — useful when you need a database-agnostic solution that doesn't rely on `LIMIT`/`OFFSET` syntax differences.

### 27. How would you find duplicate rows in a table, and then delete duplicates while keeping one copy?
```sql
-- Find duplicates
SELECT email, COUNT(*) 
FROM users 
GROUP BY email 
HAVING COUNT(*) > 1;

-- Delete duplicates, keeping the lowest id per group
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id) FROM users GROUP BY email
);
```

### 28. How do you optimize a slow-running query on a large PostgreSQL table, relevant to your metadata performance work?
Run `EXPLAIN ANALYZE` to see the actual execution plan and identify whether it's doing a sequential scan instead of using an index, check if appropriate indexes exist on filtered/joined columns, avoid `SELECT *` (fetch only needed columns), rewrite correlated subqueries as joins where possible, and consider partitioning very large tables by a logical key (like date) if applicable.

### 29. What is database partitioning, and when would it help with a large-volume digital asset metadata table?
Partitioning splits a large table into smaller physical pieces (e.g., by date range or category) while still being queried as one logical table. For a metadata table with years of historical records, partitioning by date means queries filtering on recent data only scan the relevant partition instead of the entire table, significantly improving performance.

### 30. What is the difference between `NOW()`, `CURRENT_TIMESTAMP`, and handling time zones in PostgreSQL?
`NOW()` and `CURRENT_TIMESTAMP` both return the current date/time, including time zone info if the column type is `TIMESTAMPTZ`. For systems syncing across distributed environments (relevant to your cross-team data sync work), it's important to consistently use `TIMESTAMPTZ` and store/convert to UTC, rather than mixing naive timestamps, to avoid bugs when teams operate across time zones.

---

## Section 2: Scenario-Based SQL Questions (Schema Design Style)

### 31. Design a simple schema for an order management system and explain your table choices.
Three core tables: `customers` (customer_id PK, name, email), `orders` (order_id PK, customer_id FK, order_date, status), and `order_items` (order_item_id PK, order_id FK, product_id FK, quantity, price). This normalizes the data — a customer can have many orders, and an order can have many items — avoiding repeating customer or product details on every row.

### 32. Given an `employees` table with `emp_id`, `name`, `manager_id`, write a query to list each employee with their manager's name.
```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;
```
A `LEFT JOIN` (self-join) ensures employees with no manager (e.g., the CEO) still appear in the result, with a NULL manager name.

### 33. Write a query to find customers who placed orders in every month of the last quarter.
```sql
SELECT customer_id
FROM orders
WHERE order_date >= DATE_TRUNC('quarter', CURRENT_DATE) - INTERVAL '3 months'
GROUP BY customer_id
HAVING COUNT(DISTINCT DATE_TRUNC('month', order_date)) = 3;
```

### 34. Write a query to find the department with the highest total salary expenditure.
```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department
ORDER BY total_salary DESC
LIMIT 1;
```

### 35. How would you write a query to reconcile record counts between a source and target table after migration (relevant to your project work)?
```sql
SELECT 
    (SELECT COUNT(*) FROM source_table) AS source_count,
    (SELECT COUNT(*) FROM target_table) AS target_count,
    (SELECT COUNT(*) FROM source_table) - (SELECT COUNT(*) FROM target_table) AS difference;
```
For deeper reconciliation, you'd also join on a unique key to find specific records present in one table but missing in the other, using a `LEFT JOIN ... WHERE target.id IS NULL` pattern.

---

## Section 3: Coding Round Questions (Java)

Infosys's live coding round for experienced hires typically covers data structures, string/array manipulation, and basic algorithmic problems (not heavy competitive-programming-level DSA) — solvable with clean, working Java code and clear explanation of your approach and complexity.

### 36. Reverse a string without using built-in reverse methods.
```java
public static String reverseString(String str) {
    char[] chars = str.toCharArray();
    int left = 0, right = chars.length - 1;
    while (left < right) {
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;
        left++;
        right--;
    }
    return new String(chars);
}
```
**Approach**: two-pointer swap from both ends toward the middle. Time complexity O(n), space O(n) for the output array.

### 37. Check if a string is a palindrome.
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
**Approach**: compare characters from both ends moving inward; mismatch means it's not a palindrome. O(n) time, O(1) extra space.

### 38. Find the first non-repeating character in a string.
```java
public static Character firstNonRepeating(String str) {
    Map<Character, Integer> freq = new LinkedHashMap<>();
    for (char c : str.toCharArray()) {
        freq.put(c, freq.getOrDefault(c, 0) + 1);
    }
    for (Map.Entry<Character, Integer> entry : freq.entrySet()) {
        if (entry.getValue() == 1) return entry.getKey();
    }
    return null;
}
```
**Approach**: `LinkedHashMap` preserves insertion order, so the first key with frequency 1 found during iteration is the answer. O(n) time.

### 39. Check if two strings are anagrams of each other.
```java
public static boolean areAnagrams(String s1, String s2) {
    if (s1.length() != s2.length()) return false;
    int[] count = new int[256];
    for (char c : s1.toCharArray()) count[c]++;
    for (char c : s2.toCharArray()) count[c]--;
    for (int c : count) {
        if (c != 0) return false;
    }
    return true;
}
```
**Approach**: count character frequency in the first string, decrement for the second — if all counts return to zero, they're anagrams. O(n) time, O(1) space (fixed-size array).

### 40. Find the missing number in an array of 1 to N.
```java
public static int findMissingNumber(int[] arr, int n) {
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : arr) {
        actualSum += num;
    }
    return expectedSum - actualSum;
}
```
**Approach**: use the formula for sum of first N natural numbers, subtract the actual sum of the array — the difference is the missing number. O(n) time, O(1) space.

### 41. Find duplicate elements in an array.
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
**Approach**: `Set.add()` returns `false` if the element already exists, letting you detect duplicates in a single pass. O(n) time, O(n) space.

### 42. Given an array, find two numbers that add up to a target sum (Two Sum problem).
```java
public static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[]{seen.get(complement), i};
        }
        seen.put(nums[i], i);
    }
    return new int[]{-1, -1};
}
```
**Approach**: store each number's index in a map as you iterate; for each new number, check if its complement (target - number) was already seen. This achieves O(n) time instead of the brute-force O(n²) nested loop.

### 43. Find the maximum subarray sum (Kadane's Algorithm).
```java
public static int maxSubArraySum(int[] arr) {
    int maxSoFar = arr[0], maxEndingHere = arr[0];
    for (int i = 1; i < arr.length; i++) {
        maxEndingHere = Math.max(arr[i], maxEndingHere + arr[i]);
        maxSoFar = Math.max(maxSoFar, maxEndingHere);
    }
    return maxSoFar;
}
```
**Approach**: at each position, decide whether extending the previous subarray or starting fresh from the current element gives a better sum; track the overall best seen. O(n) time, O(1) space.

### 44. Detect if a linked list has a cycle.
```java
public static boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```
**Approach**: Floyd's cycle detection (slow/fast pointer) — if there's a cycle, the fast pointer (moving 2 steps) will eventually meet the slow pointer (moving 1 step) inside the loop. O(n) time, O(1) space.

### 45. Reverse a singly linked list.
```java
public static ListNode reverseList(ListNode head) {
    ListNode prev = null, current = head;
    while (current != null) {
        ListNode next = current.next;
        current.next = prev;
        prev = current;
        current = next;
    }
    return prev;
}
```
**Approach**: iteratively flip each node's `next` pointer to point backward, tracking the previous node, until you reach the end. O(n) time, O(1) space.

### 46. Find the Longest Common Subsequence (LCS) of two strings — a known Infosys-asked problem.
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
**Approach**: classic dynamic programming — `dp[i][j]` represents the LCS length of the first `i` characters of `s1` and first `j` characters of `s2`. If characters match, extend the diagonal result; otherwise take the best of skipping a character from either string. O(m×n) time and space.

### 47. Find the Longest Increasing Subsequence (LIS) of an array — another known Infosys-asked problem.
```java
public static int lengthOfLIS(int[] nums) {
    int[] dp = new int[nums.length];
    Arrays.fill(dp, 1);
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
**Approach**: `dp[i]` tracks the length of the longest increasing subsequence ending at index `i`. For each element, check all previous smaller elements and extend the best one. O(n²) time — there's an O(n log n) version using binary search if asked to optimize further.

### 48. Write code to demonstrate (and explain) a deadlock scenario in Java.
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
**Explanation**: Thread 1 locks A then waits for B; Thread 2 locks B then waits for A — each holds what the other needs, so neither can proceed. **Fix**: always acquire locks in the same global order across all threads (e.g., always lock A before B), which eliminates the circular wait condition.

### 49. Given a list of objects, use Java Streams to group them by a field and count occurrences — relevant to processing migration batch records.
```java
Map<String, Long> countByStatus = records.stream()
    .collect(Collectors.groupingBy(Record::getStatus, Collectors.counting()));
```
**Approach**: `Collectors.groupingBy` groups stream elements by a classifier function (here, status), and `Collectors.counting()` as the downstream collector counts how many records fall into each group — useful for quickly summarizing how many records succeeded/failed/were skipped in a migration batch.

### 50. Implement a simple LRU (Least Recently Used) cache.
```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // true = access order
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```
**Approach**: `LinkedHashMap` with `accessOrder=true` reorders entries based on access (not just insertion), and overriding `removeEldestEntry` automatically evicts the least recently used entry once the cache exceeds capacity — a clean way to implement LRU without manually managing a doubly linked list.

---

## Tips for the SQL & Coding Rounds

- **Practice writing SQL by hand on a whiteboard/notepad** — recent candidates report being asked to design a schema and write queries live, not just answer theory questions.
- **Know your `EXPLAIN ANALYZE` output cold** — given your PostgreSQL performance optimization experience, expect a deeper follow-up on a specific query you've tuned in real work.
- **For coding questions, narrate your approach before writing code** — interviewers consistently weigh problem-solving process and complexity analysis (time/space) as much as the final working code.
- **Window functions and CTEs are increasingly tested** even for backend (not just data engineering) roles — make sure you're comfortable writing one from scratch, not just reading one.
- **LCS, LIS, linked list reversal/cycle detection, and Two Sum** are patterns specifically reported by recent Infosys candidates — these are worth practicing by hand, not just reading the solution.
- Be ready to connect a coding/SQL answer back to your **real project work** (e.g., "I used a similar grouping/counting approach when reconciling migration batch results") — this consistently impresses interviewers more than a purely theoretical answer.

---

*This document combines current (2026) Infosys candidate-reported interview patterns with SQL/database fundamentals and coding problems tailored to your PostgreSQL/MySQL and Java backend background. Practice running these queries against a real database (or an online SQL sandbox) rather than just reading them — muscle memory matters under interview pressure.*
