# Infosys Java Coding Interview Questions & Solutions
### For 4 Years of Experience

This guide compiles the most commonly asked **Java coding/DSA questions** in Infosys technical interviews for developers with around 4 years of experience, based on recent candidate interview reports (2025–2026). Infosys typically tests arrays, strings, linked lists, stacks/queues, trees, hashing, sorting/searching, and basic DP/greedy — solvable cleanly without competitive-programming-level complexity, with emphasis on clear logic, edge-case handling, and time/space complexity discussion.

---

## Section 1: Arrays

### 1. Reverse an array in place.
```java
public static void reverseArray(int[] arr) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
```
**Approach**: two-pointer swap from both ends. Time O(n), Space O(1).

### 2. Find the maximum and minimum element in an array in a single pass.
```java
public static int[] findMinMax(int[] arr) {
    int min = arr[0], max = arr[0];
    for (int num : arr) {
        if (num < min) min = num;
        if (num > max) max = num;
    }
    return new int[]{min, max};
}
```
**Approach**: track both values while iterating once instead of sorting or scanning twice. Time O(n), Space O(1).

### 3. Find the missing number in an array containing 1 to N.
```java
public static int findMissingNumber(int[] arr, int n) {
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : arr) actualSum += num;
    return expectedSum - actualSum;
}
```
**Approach**: use the arithmetic series sum formula and subtract the actual sum — the gap is the missing number. Time O(n), Space O(1).

### 4. Find all duplicate elements in an array.
```java
public static List<Integer> findDuplicates(int[] arr) {
    Set<Integer> seen = new HashSet<>();
    List<Integer> duplicates = new ArrayList<>();
    for (int num : arr) {
        if (!seen.add(num)) duplicates.add(num);
    }
    return duplicates;
}
```
**Approach**: `Set.add()` returns `false` if the value already exists, letting you flag duplicates in one pass. Time O(n), Space O(n).

### 5. Two Sum — find two numbers in an array that add up to a target.
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
**Approach**: store each visited number's index in a map; check if the current number's complement was already seen. Reduces brute-force O(n²) to O(n) time, O(n) space.

### 6. Find the maximum subarray sum (Kadane's Algorithm).
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
**Approach**: at each index, decide whether to extend the running subarray or restart from the current element. Time O(n), Space O(1).

### 7. Rotate an array to the right by K positions.
```java
public static void rotateArray(int[] arr, int k) {
    int n = arr.length;
    k = k % n;
    reverse(arr, 0, n - 1);
    reverse(arr, 0, k - 1);
    reverse(arr, k, n - 1);
}
private static void reverse(int[] arr, int start, int end) {
    while (start < end) {
        int temp = arr[start];
        arr[start] = arr[end];
        arr[end] = temp;
        start++;
        end--;
    }
}
```
**Approach**: reverse the whole array, then reverse the two segments individually — a well-known trick to rotate in-place. Time O(n), Space O(1).

### 8. Find the intersection of two arrays.
```java
public static List<Integer> intersection(int[] arr1, int[] arr2) {
    Set<Integer> set1 = new HashSet<>();
    for (int num : arr1) set1.add(num);
    List<Integer> result = new ArrayList<>();
    Set<Integer> added = new HashSet<>();
    for (int num : arr2) {
        if (set1.contains(num) && added.add(num)) {
            result.add(num);
        }
    }
    return result;
}
```
**Approach**: put the first array into a set for O(1) lookups, then check membership while scanning the second array, using a second set to avoid duplicate results. Time O(n+m), Space O(n+m).

### 9. Move all zeros in an array to the end while maintaining order of non-zero elements.
```java
public static void moveZeros(int[] arr) {
    int insertPos = 0;
    for (int num : arr) {
        if (num != 0) arr[insertPos++] = num;
    }
    while (insertPos < arr.length) {
        arr[insertPos++] = 0;
    }
}
```
**Approach**: overwrite the array from the front with non-zero elements in order, then fill the remaining positions with zeros. Time O(n), Space O(1).

### 10. Find the leaders in an array (an element is a leader if it's greater than all elements to its right).
```java
public static List<Integer> findLeaders(int[] arr) {
    List<Integer> leaders = new ArrayList<>();
    int maxFromRight = arr[arr.length - 1];
    leaders.add(maxFromRight);
    for (int i = arr.length - 2; i >= 0; i--) {
        if (arr[i] > maxFromRight) {
            maxFromRight = arr[i];
            leaders.add(maxFromRight);
        }
    }
    Collections.reverse(leaders);
    return leaders;
}
```
**Approach**: traverse from right to left, tracking the maximum seen so far — any element greater than that running max is a leader. Time O(n), Space O(n).

### 11. Find the equilibrium index of an array (where sum of elements on the left equals sum on the right).
```java
public static int equilibriumIndex(int[] arr) {
    int totalSum = 0;
    for (int num : arr) totalSum += num;
    int leftSum = 0;
    for (int i = 0; i < arr.length; i++) {
        totalSum -= arr[i];
        if (leftSum == totalSum) return i;
        leftSum += arr[i];
    }
    return -1;
}
```
**Approach**: precompute the total sum, then as you iterate, subtract the current element from the "right" running total and compare with the "left" running total. Time O(n), Space O(1).

### 12. Merge two sorted arrays into one sorted array.
```java
public static int[] mergeSortedArrays(int[] a, int[] b) {
    int[] result = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length) {
        result[k++] = (a[i] <= b[j]) ? a[i++] : b[j++];
    }
    while (i < a.length) result[k++] = a[i++];
    while (j < b.length) result[k++] = b[j++];
    return result;
}
```
**Approach**: classic merge-step from merge sort — compare front elements of both arrays and pick the smaller each time. Time O(n+m), Space O(n+m).

---

## Section 2: Strings

### 13. Reverse a string without built-in reverse methods.
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
**Approach**: two-pointer swap. Time O(n), Space O(n) for the output.

### 14. Check if a string is a palindrome.
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
**Approach**: compare characters from both ends moving inward. Time O(n), Space O(1).

### 15. Check if two strings are anagrams of each other.
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
**Approach**: count character frequencies for the first string, decrement for the second; all-zero counts confirm an anagram. Time O(n), Space O(1) (fixed-size array).

### 16. Find the first non-repeating character in a string.
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
**Approach**: `LinkedHashMap` preserves insertion order, so the first key with count 1 during iteration is the answer. Time O(n), Space O(n).

### 17. Count the occurrences of each character in a string.
```java
public static Map<Character, Integer> countChars(String str) {
    Map<Character, Integer> freq = new LinkedHashMap<>();
    for (char c : str.toCharArray()) {
        freq.put(c, freq.getOrDefault(c, 0) + 1);
    }
    return freq;
}
```
**Approach**: simple frequency map built in a single pass. Time O(n), Space O(n).

### 18. Find the longest substring without repeating characters.
```java
public static int longestUniqueSubstring(String str) {
    Set<Character> window = new HashSet<>();
    int left = 0, maxLen = 0;
    for (int right = 0; right < str.length(); right++) {
        char c = str.charAt(right);
        while (window.contains(c)) {
            window.remove(str.charAt(left));
            left++;
        }
        window.add(c);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```
**Approach**: sliding window — expand the right edge, and shrink from the left whenever a duplicate is found, tracking the largest valid window size. Time O(n), Space O(n).

### 19. Check if a string contains only digits.
```java
public static boolean isNumeric(String str) {
    if (str == null || str.isEmpty()) return false;
    for (char c : str.toCharArray()) {
        if (!Character.isDigit(c)) return false;
    }
    return true;
}
```
**Approach**: check every character against `Character.isDigit()`, handling null/empty as an edge case upfront. Time O(n), Space O(1).

### 20. Remove duplicate characters from a string while preserving order.
```java
public static String removeDuplicates(String str) {
    Set<Character> seen = new LinkedHashSet<>();
    for (char c : str.toCharArray()) seen.add(c);
    StringBuilder sb = new StringBuilder();
    for (char c : seen) sb.append(c);
    return sb.toString();
}
```
**Approach**: `LinkedHashSet` automatically deduplicates while preserving insertion order; build the final string from it. Time O(n), Space O(n).

### 21. Check if two strings are rotations of each other.
```java
public static boolean areRotations(String s1, String s2) {
    if (s1.length() != s2.length()) return false;
    return (s1 + s1).contains(s2);
}
```
**Approach**: concatenating the first string with itself contains every possible rotation of it as a substring, so checking if the second string appears within that concatenation confirms a rotation. Time O(n), Space O(n).

### 22. Count the number of vowels and consonants in a string.
```java
public static int[] countVowelsConsonants(String str) {
    int vowels = 0, consonants = 0;
    String vowelSet = "aeiouAEIOU";
    for (char c : str.toCharArray()) {
        if (Character.isLetter(c)) {
            if (vowelSet.indexOf(c) != -1) vowels++;
            else consonants++;
        }
    }
    return new int[]{vowels, consonants};
}
```
**Approach**: filter to letters only, then classify each as vowel or consonant via lookup. Time O(n), Space O(1).

---

## Section 3: Linked Lists

### 23. Reverse a singly linked list.
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
**Approach**: iteratively flip each node's `next` pointer backward while tracking the previous node. Time O(n), Space O(1).

### 24. Detect if a linked list has a cycle.
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
**Approach**: Floyd's cycle detection (slow/fast pointers) — if a cycle exists, the fast pointer eventually laps the slow one. Time O(n), Space O(1).

### 25. Find the middle element of a linked list in one pass.
```java
public static ListNode findMiddle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```
**Approach**: slow pointer moves 1 step, fast pointer moves 2 — when fast reaches the end, slow is at the middle. Time O(n), Space O(1).

### 26. Merge two sorted linked lists.
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
    tail.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```
**Approach**: use a dummy head node to simplify edge cases, then repeatedly attach the smaller of the two current nodes. Time O(n+m), Space O(1).

### 27. Remove the Nth node from the end of a linked list.
```java
public static ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(-1);
    dummy.next = head;
    ListNode slow = dummy, fast = dummy;
    for (int i = 0; i < n; i++) fast = fast.next;
    while (fast.next != null) {
        slow = slow.next;
        fast = fast.next;
    }
    slow.next = slow.next.next;
    return dummy.next;
}
```
**Approach**: advance `fast` n steps ahead first, then move both pointers together — when `fast` hits the end, `slow` is right before the node to remove. Time O(n), Space O(1).

### 28. Check if a linked list is a palindrome.
```java
public static boolean isPalindromeList(ListNode head) {
    List<Integer> values = new ArrayList<>();
    while (head != null) {
        values.add(head.val);
        head = head.next;
    }
    int left = 0, right = values.size() - 1;
    while (left < right) {
        if (!values.get(left).equals(values.get(right))) return false;
        left++;
        right--;
    }
    return true;
}
```
**Approach**: copy values into a list, then check it as a regular palindrome with two pointers (a more memory-efficient O(1)-space approach reverses the second half in place if asked to optimize). Time O(n), Space O(n).

### 29. Find the intersection point of two linked lists.
```java
public static ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode a = headA, b = headB;
    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }
    return a;
}
```
**Approach**: by switching each pointer to the other list's head once it reaches the end, both pointers travel equal total distances, guaranteeing they meet exactly at the intersection (or both reach null if there's none). Time O(n+m), Space O(1).

---

## Section 4: Stacks & Queues

### 30. Check if parentheses/brackets in a string are balanced.
```java
public static boolean isBalanced(String str) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> pairs = Map.of(')', '(', ']', '[', '}', '{');
    for (char c : str.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else if (pairs.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != pairs.get(c)) return false;
        }
    }
    return stack.isEmpty();
}
```
**Approach**: push opening brackets, and on a closing bracket, verify the top of the stack is its matching opener; the string is balanced only if the stack ends empty. Time O(n), Space O(n).

### 31. Implement a stack using an array (or explain how `Deque`/`Stack` works internally).
```java
public class ArrayStack {
    private int[] data;
    private int top = -1;

    public ArrayStack(int capacity) {
        data = new int[capacity];
    }

    public void push(int val) {
        if (top == data.length - 1) throw new RuntimeException("Stack overflow");
        data[++top] = val;
    }

    public int pop() {
        if (top == -1) throw new RuntimeException("Stack underflow");
        return data[top--];
    }

    public int peek() {
        return data[top];
    }

    public boolean isEmpty() {
        return top == -1;
    }
}
```
**Approach**: maintain a `top` index into a fixed-size array; push increments and writes, pop reads and decrements — all O(1) operations.

### 32. Implement a queue using two stacks.
```java
public class QueueUsingStacks {
    private Deque<Integer> inStack = new ArrayDeque<>();
    private Deque<Integer> outStack = new ArrayDeque<>();

    public void enqueue(int val) {
        inStack.push(val);
    }

    public int dequeue() {
        if (outStack.isEmpty()) {
            while (!inStack.isEmpty()) {
                outStack.push(inStack.pop());
            }
        }
        return outStack.pop();
    }
}
```
**Approach**: `inStack` handles enqueues; when a dequeue is needed and `outStack` is empty, transfer everything from `inStack` to `outStack`, reversing the order so the oldest element ends up on top. Amortized O(1) per operation.

### 33. Evaluate a postfix (Reverse Polish Notation) expression.
```java
public static int evalPostfix(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (String token : tokens) {
        if (token.matches("-?\\d+")) {
            stack.push(Integer.parseInt(token));
        } else {
            int b = stack.pop();
            int a = stack.pop();
            switch (token) {
                case "+": stack.push(a + b); break;
                case "-": stack.push(a - b); break;
                case "*": stack.push(a * b); break;
                case "/": stack.push(a / b); break;
            }
        }
    }
    return stack.pop();
}
```
**Approach**: push numbers onto the stack; on an operator, pop the two most recent operands, apply the operator, and push the result back. Time O(n), Space O(n).

### 34. Find the next greater element for every element in an array.
```java
public static int[] nextGreaterElement(int[] arr) {
    int[] result = new int[arr.length];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>(); // stores indices
    for (int i = 0; i < arr.length; i++) {
        while (!stack.isEmpty() && arr[stack.peek()] < arr[i]) {
            result[stack.pop()] = arr[i];
        }
        stack.push(i);
    }
    return result;
}
```
**Approach**: maintain a stack of indices whose "next greater" hasn't been found yet; whenever the current element is bigger than the stack's top, that's the answer for the popped index. Time O(n), Space O(n).

---

## Section 5: Trees & Hashing

### 35. Implement in-order, pre-order, and post-order traversal of a binary tree.
```java
public static void inorder(TreeNode root, List<Integer> result) {
    if (root == null) return;
    inorder(root.left, result);
    result.add(root.val);
    inorder(root.right, result);
}

public static void preorder(TreeNode root, List<Integer> result) {
    if (root == null) return;
    result.add(root.val);
    preorder(root.left, result);
    preorder(root.right, result);
}

public static void postorder(TreeNode root, List<Integer> result) {
    if (root == null) return;
    postorder(root.left, result);
    postorder(root.right, result);
    result.add(root.val);
}
```
**Approach**: standard recursive depth-first traversals, differing only in when the current node's value is visited relative to its children. Time O(n), Space O(h) for the recursion stack (h = tree height).

### 36. Find the height (max depth) of a binary tree.
```java
public static int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```
**Approach**: recursively compute the depth of each subtree and take the larger, adding 1 for the current node. Time O(n), Space O(h).

### 37. Check if a binary tree is balanced (height difference of subtrees at every node is at most 1).
```java
public static boolean isBalanced(TreeNode root) {
    return checkHeight(root) != -1;
}
private static int checkHeight(TreeNode node) {
    if (node == null) return 0;
    int left = checkHeight(node.left);
    if (left == -1) return -1;
    int right = checkHeight(node.right);
    if (right == -1) return -1;
    if (Math.abs(left - right) > 1) return -1;
    return Math.max(left, right) + 1;
}
```
**Approach**: compute height bottom-up, returning -1 as a sentinel the moment imbalance is detected anywhere, so the check short-circuits instead of recomputing height separately for every node. Time O(n), Space O(h).

### 38. Perform a level-order (breadth-first) traversal of a binary tree.
```java
public static List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```
**Approach**: use a queue (BFS); process one full level at a time by capturing the queue's size before enqueuing the next level's children. Time O(n), Space O(n).

### 39. Validate if a binary tree is a valid Binary Search Tree (BST).
```java
public static boolean isValidBST(TreeNode root) {
    return validate(root, null, null);
}
private static boolean validate(TreeNode node, Integer min, Integer max) {
    if (node == null) return true;
    if ((min != null && node.val <= min) || (max != null && node.val >= max)) {
        return false;
    }
    return validate(node.left, min, node.val) && validate(node.right, node.val, max);
}
```
**Approach**: pass down a valid (min, max) range for each node as you recurse — a naive check of only comparing a node to its immediate children misses violations further down the tree. Time O(n), Space O(h).

### 40. Find the Lowest Common Ancestor (LCA) of two nodes in a binary tree.
```java
public static TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) return root;
    return (left != null) ? left : right;
}
```
**Approach**: recursively search both subtrees; if one of the target nodes is found in each subtree, the current node is the LCA — otherwise propagate up whichever side found a match. Time O(n), Space O(h).

### 41. Given an array of integers, find the first pair with a given sum using hashing.
```java
public static int[] findPairWithSum(int[] arr, int targetSum) {
    Set<Integer> seen = new HashSet<>();
    for (int num : arr) {
        int complement = targetSum - num;
        if (seen.contains(complement)) {
            return new int[]{complement, num};
        }
        seen.add(num);
    }
    return null;
}
```
**Approach**: same hashing pattern as Two Sum — track seen values, check for the needed complement each step. Time O(n), Space O(n).

### 42. Group anagrams together from a list of strings.
```java
public static List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String str : strs) {
        char[] chars = str.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(str);
    }
    return new ArrayList<>(map.values());
}
```
**Approach**: sorting each string's characters produces a canonical key shared by all its anagrams, so grouping by that key naturally clusters anagrams together. Time O(n·k log k) where k is average string length, Space O(n·k).

### 43. Find the frequency of the most frequent element in an array.
```java
public static int mostFrequentCount(int[] arr) {
    Map<Integer, Integer> freq = new HashMap<>();
    int maxCount = 0;
    for (int num : arr) {
        int count = freq.merge(num, 1, Integer::sum);
        maxCount = Math.max(maxCount, count);
    }
    return maxCount;
}
```
**Approach**: `merge()` increments the count for each number as you go, tracking the running maximum in the same pass. Time O(n), Space O(n).

---

## Section 6: Sorting, Searching & Recursion

### 44. Implement binary search on a sorted array.
```java
public static int binarySearch(int[] arr, int target) {
    int low = 0, high = arr.length - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}
```
**Approach**: repeatedly halve the search space by comparing the middle element to the target. Time O(log n), Space O(1).

### 45. Implement bubble sort and explain its time complexity.
```java
public static void bubbleSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}
```
**Approach**: repeatedly swap adjacent out-of-order elements; the early-exit flag (`swapped`) avoids unnecessary passes once the array is already sorted. Time O(n²) worst case, O(n) best case (already sorted), Space O(1).

### 46. Implement merge sort.
```java
public static void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return;
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}
private static void merge(int[] arr, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;
    while (i <= mid && j <= right) {
        temp[k++] = (arr[i] <= arr[j]) ? arr[i++] : arr[j++];
    }
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    System.arraycopy(temp, 0, arr, left, temp.length);
}
```
**Approach**: classic divide-and-conquer — split the array in half recursively, then merge the sorted halves back together. Time O(n log n), Space O(n).

### 47. Implement quicksort.
```java
public static void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivotIndex = partition(arr, low, high);
        quickSort(arr, low, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, high);
    }
}
private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            int temp = arr[i]; arr[i] = arr[j]; arr[j] = temp;
        }
    }
    int temp = arr[i + 1]; arr[i + 1] = arr[high]; arr[high] = temp;
    return i + 1;
}
```
**Approach**: pick a pivot (here, the last element), partition the array so smaller elements move left and larger move right, then recursively sort each side. Time O(n log n) average, O(n²) worst case, Space O(log n) for the recursion stack.

### 48. Calculate the factorial of a number using recursion, and explain how you'd avoid stack overflow for large inputs.
```java
public static long factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```
**Approach**: simple recursive base case + recursive step. For very large `n`, an iterative version (a loop with an accumulator) avoids the risk of a `StackOverflowError` from deep recursion, since Java doesn't optimize tail calls.

### 49. Find the nth Fibonacci number efficiently (avoiding the naive exponential recursion).
```java
public static long fibonacci(int n) {
    if (n <= 1) return n;
    long[] dp = new long[n + 1];
    dp[0] = 0;
    dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```
**Approach**: bottom-up dynamic programming builds each Fibonacci number from the two before it, avoiding the redundant recomputation that plain recursion causes. Time O(n), Space O(n) (can be reduced to O(1) by keeping just the last two values).

### 50. Find the Longest Common Subsequence (LCS) of two strings.
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
**Approach**: `dp[i][j]` holds the LCS length of the first `i` characters of `s1` and first `j` of `s2`; matching characters extend the diagonal, otherwise take the better of skipping a character from either string. Time O(m×n), Space O(m×n).

### 51. Find the Longest Increasing Subsequence (LIS) in an array.
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
**Approach**: `dp[i]` tracks the longest increasing subsequence ending at index `i`, built by checking all earlier smaller elements. Time O(n²) — an O(n log n) version using binary search exists if asked to optimize.

### 52. Solve the Subset Sum problem — does any subset of an array sum to a target value?
```java
public static boolean subsetSum(int[] arr, int target) {
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;
    for (int num : arr) {
        for (int j = target; j >= num; j--) {
            if (dp[j - num]) dp[j] = true;
        }
    }
    return dp[target];
}
```
**Approach**: classic 0/1 knapsack-style DP — `dp[j]` tracks whether sum `j` is achievable; iterate target downward per number to avoid reusing the same element twice. Time O(n×target), Space O(target).

---

## Tips for the Infosys Coding Round

- **Always clarify edge cases out loud** before coding — empty input, single element, negative numbers, duplicates. Infosys interviewers consistently note this as a common candidate mistake.
- **State time and space complexity** after writing your solution, even if not asked — it signals strong fundamentals.
- **Start with the brute-force approach if stuck**, then optimize — interviewers want to see your thought process, not just a memorized final answer.
- **Two-pointer, sliding window, and hashing patterns** cover a large share of array/string questions — mastering these patterns generalizes better than memorizing individual problems.
- **LCS, LIS, and linked list problems** (cycle detection, reversal, merging) are specifically reported by recent (2026) Infosys candidates at the 4-year experience level — practice these by hand, not just by reading solutions.
- Use **clean variable names and small helper methods** in your live coding — Infosys interviewers value readable, maintainable code as much as a correct answer, consistent with real production expectations.

---

*This document is based on recent (2025–2026) Infosys candidate-reported coding interview patterns and standard DSA topics typically tested at the 4-years-experience band. Practice writing these from scratch on paper or a whiteboard, not just reading the code, since live coding rounds test recall under pressure.*
