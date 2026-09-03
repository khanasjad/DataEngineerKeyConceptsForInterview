# JAVA CODING INTERVIEW SURVIVAL GUIDE
## "I'm Not Smart But I'll Pass This Interview Somehow"

> **Philosophy**: You don't need to be genius. You need to recognize patterns and apply templates. That's it.

---

## 🎯 THE GOLDEN RULE

**90% of coding interviews = 10 patterns**

If you see X → Use Template Y → Profit

---

## 📚 TABLE OF CONTENTS
1. [How to Start ANY Problem](#how-to-start)
2. [The 10 Patterns That Save Your Life](#the-10-patterns)
3. [Java Syntax You Must Know](#java-must-know)
4. [Common Questions By Pattern](#common-questions)
5. [When You're Stuck (Panic Mode)](#panic-mode)
6. [Final Checklist Before Interview](#final-checklist)

---

## 🚀 HOW TO START ANY PROBLEM {#how-to-start}

### Step 1: Read the problem
- Underline: **input type** (array, string, tree, graph)
- Circle: **what they want** (find max, count something, return true/false)

### Step 2: Ask yourself these questions:
1. **Is it sorted?** → Binary Search or Two Pointers
2. **Find a subarray/substring?** → Sliding Window
3. **Tree mentioned?** → DFS or BFS
4. **Graph mentioned?** → DFS/BFS with visited set
5. **Count/sum something?** → Hash Map
6. **Two arrays, find common?** → Hash Set
7. **Need to track min/max continuously?** → Heap/PriorityQueue
8. **Optimization problem (min/max)?** → DP (but try greedy first)

### Step 3: Say this out loud to interviewer:
> "Let me start with a brute force approach to make sure I understand the problem correctly."

**Why?** This buys you time and shows you can think clearly.

---

## 💎 THE 10 PATTERNS THAT SAVE YOUR LIFE {#the-10-patterns}

### Pattern 1: HASH MAP (Count/Frequency/Lookup)
**When**: "Find if exists", "Count occurrences", "Two sum"

```java
// Template - MEMORIZE THIS
Map<Integer, Integer> map = new HashMap<>();

// Count frequency
for (int num : array) {
    map.put(num, map.getOrDefault(num, 0) + 1);
}

// Check if exists
if (map.containsKey(target)) {
    // do something
}

// Two Sum Pattern
Map<Integer, Integer> map = new HashMap<>();
for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i];
    if (map.containsKey(complement)) {
        return new int[]{map.get(complement), i};
    }
    map.put(nums[i], i);
}
```

**Common Questions**: Two Sum, Group Anagrams, First Unique Character

---

### Pattern 2: TWO POINTERS (Array/String)
**When**: Sorted array OR you can sort it OR palindrome check

```java
// Template 1: Opposite Direction (sorted array)
int left = 0;
int right = array.length - 1;

while (left < right) {
    if (condition) {
        // found answer
    }
    if (needToMoveLeft) left++;
    else right--;
}

// Template 2: Same Direction (slow/fast)
int slow = 0;
for (int fast = 0; fast < array.length; fast++) {
    if (array[fast] meets condition) {
        array[slow] = array[fast];
        slow++;
    }
}

// Template 3: Palindrome Check
int left = 0, right = s.length() - 1;
while (left < right) {
    if (s.charAt(left) != s.charAt(right)) return false;
    left++;
    right--;
}
return true;
```

**Common Questions**: Two Sum (sorted), Remove Duplicates, Valid Palindrome, Container With Most Water

---

### Pattern 3: SLIDING WINDOW (Subarray/Substring)
**When**: "contiguous subarray", "substring", "window of size K"

```java
// Template 1: Fixed Size Window
int windowSum = 0;
for (int i = 0; i < k; i++) {
    windowSum += array[i];
}
int maxSum = windowSum;

for (int i = k; i < array.length; i++) {
    windowSum = windowSum - array[i - k] + array[i];
    maxSum = Math.max(maxSum, windowSum);
}

// Template 2: Variable Size Window
int left = 0, maxLength = 0;
Map<Character, Integer> map = new HashMap<>();

for (int right = 0; right < s.length(); right++) {
    char c = s.charAt(right);
    map.put(c, map.getOrDefault(c, 0) + 1);

    // Shrink window if invalid
    while (window is invalid) {
        char leftChar = s.charAt(left);
        map.put(leftChar, map.get(leftChar) - 1);
        left++;
    }

    maxLength = Math.max(maxLength, right - left + 1);
}
```

**Common Questions**: Max Sum Subarray of Size K, Longest Substring Without Repeating Characters, Minimum Window Substring

---

### Pattern 4: BINARY SEARCH
**When**: "Find in sorted array", "Find first/last occurrence", array is sorted

```java
// Template 1: Standard Binary Search
int left = 0, right = array.length - 1;

while (left <= right) {
    int mid = left + (right - left) / 2;  // Avoid overflow

    if (array[mid] == target) return mid;

    if (array[mid] < target) {
        left = mid + 1;
    } else {
        right = mid - 1;
    }
}
return -1;  // Not found

// Template 2: Find First Occurrence
int left = 0, right = array.length - 1;
int result = -1;

while (left <= right) {
    int mid = left + (right - left) / 2;

    if (array[mid] == target) {
        result = mid;
        right = mid - 1;  // Keep searching left
    } else if (array[mid] < target) {
        left = mid + 1;
    } else {
        right = mid - 1;
    }
}
return result;
```

**Common Questions**: Search in Rotated Sorted Array, Find First and Last Position, Search Insert Position

---

### Pattern 5: DEPTH FIRST SEARCH (DFS) - Trees & Graphs
**When**: "Tree traversal", "Connected components", "Path exists"

```java
// Template 1: Tree DFS (Recursion)
void dfs(TreeNode root) {
    // Base case
    if (root == null) return;

    // Process current node
    System.out.println(root.val);

    // Recurse on children
    dfs(root.left);
    dfs(root.right);
}

// Template 2: Graph DFS with Visited Set
void dfs(int node, Set<Integer> visited, Map<Integer, List<Integer>> graph) {
    visited.add(node);

    for (int neighbor : graph.get(node)) {
        if (!visited.contains(neighbor)) {
            dfs(neighbor, visited, graph);
        }
    }
}

// Template 3: Return Value (Max Depth)
int maxDepth(TreeNode root) {
    if (root == null) return 0;

    int leftDepth = maxDepth(root.left);
    int rightDepth = maxDepth(root.right);

    return 1 + Math.max(leftDepth, rightDepth);
}
```

**Common Questions**: Maximum Depth of Tree, Number of Islands, Path Sum, Validate BST

---

### Pattern 6: BREADTH FIRST SEARCH (BFS) - Level Order
**When**: "Level by level", "Shortest path", "Minimum steps"

```java
// Template: BFS with Queue
Queue<TreeNode> queue = new LinkedList<>();
queue.offer(root);

while (!queue.isEmpty()) {
    int size = queue.size();  // Current level size

    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();

        // Process node
        System.out.println(node.val);

        // Add children for next level
        if (node.left != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
}

// For Graph BFS (with visited set)
Queue<Integer> queue = new LinkedList<>();
Set<Integer> visited = new HashSet<>();

queue.offer(start);
visited.add(start);

while (!queue.isEmpty()) {
    int node = queue.poll();

    for (int neighbor : graph.get(node)) {
        if (!visited.contains(neighbor)) {
            visited.add(neighbor);
            queue.offer(neighbor);
        }
    }
}
```

**Common Questions**: Binary Tree Level Order Traversal, Minimum Depth, Shortest Path in Matrix

---

### Pattern 7: BACKTRACKING (Try All Combinations)
**When**: "Generate all", "Find all combinations/permutations/subsets"

```java
// Template: CHOOSE -> EXPLORE -> UNCHOOSE
void backtrack(List<Integer> current, List<List<Integer>> result, int[] nums, int start) {
    // Add current combination to result
    result.add(new ArrayList<>(current));  // MUST COPY!

    for (int i = start; i < nums.length; i++) {
        // CHOOSE
        current.add(nums[i]);

        // EXPLORE
        backtrack(current, result, nums, i + 1);

        // UNCHOOSE
        current.remove(current.size() - 1);
    }
}

// Usage
List<List<Integer>> result = new ArrayList<>();
backtrack(new ArrayList<>(), result, nums, 0);
```

**Common Questions**: Subsets, Permutations, Combination Sum, Letter Combinations of a Phone Number

---

### Pattern 8: DYNAMIC PROGRAMMING (Build from Smaller Problems)
**When**: "Optimization problem", "Count ways", "Can you break it into subproblems?"

```java
// Template 1: 1D DP Array
int[] dp = new int[n + 1];
dp[0] = base_case;

for (int i = 1; i <= n; i++) {
    dp[i] = calculate based on previous dp values;
}

return dp[n];

// Example: Climbing Stairs
int climbStairs(int n) {
    if (n <= 2) return n;

    int[] dp = new int[n + 1];
    dp[1] = 1;
    dp[2] = 2;

    for (int i = 3; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }

    return dp[n];
}

// Template 2: 2D DP (two sequences)
int[][] dp = new int[m + 1][n + 1];

for (int i = 0; i <= m; i++) {
    for (int j = 0; j <= n; j++) {
        if (base case) {
            dp[i][j] = value;
        } else {
            dp[i][j] = calculate from dp[i-1][j], dp[i][j-1], dp[i-1][j-1];
        }
    }
}

return dp[m][n];
```

**Common Questions**: Climbing Stairs, House Robber, Coin Change, Longest Common Subsequence

---

### Pattern 9: HEAP/PRIORITY QUEUE (Find Kth Element)
**When**: "Kth largest", "Kth smallest", "Top K elements"

```java
// Template: Min Heap (smallest at top)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Template: Max Heap (largest at top)
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);

// Find Kth Largest
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
for (int num : nums) {
    minHeap.offer(num);
    if (minHeap.size() > k) {
        minHeap.poll();  // Remove smallest
    }
}
return minHeap.peek();

// Find Kth Smallest (use max heap instead)
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
for (int num : nums) {
    maxHeap.offer(num);
    if (maxHeap.size() > k) {
        maxHeap.poll();  // Remove largest
    }
}
return maxHeap.peek();
```

**Common Questions**: Kth Largest Element, Top K Frequent Elements, Merge K Sorted Lists

---

### Pattern 10: FAST & SLOW POINTERS (Linked List Cycles)
**When**: "Detect cycle", "Find middle of linked list"

```java
// Template 1: Detect Cycle
boolean hasCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;          // Move 1 step
        fast = fast.next.next;     // Move 2 steps

        if (slow == fast) return true;
    }

    return false;
}

// Template 2: Find Middle
ListNode findMiddle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    return slow;  // Slow is at middle
}
```

**Common Questions**: Linked List Cycle, Middle of Linked List, Happy Number

---

## ☕ JAVA SYNTAX YOU MUST KNOW {#java-must-know}

### Arrays
```java
// Create
int[] arr = new int[5];
int[] arr = {1, 2, 3, 4, 5};

// Common operations
Arrays.sort(arr);                    // Sort
Arrays.fill(arr, 0);                 // Fill with 0
int[] copy = Arrays.copyOf(arr, arr.length);  // Copy
Arrays.toString(arr);                // Print: [1, 2, 3]

// 2D Array
int[][] matrix = new int[rows][cols];
```

### Strings
```java
// STRINGS ARE IMMUTABLE!
String s = "hello";
char c = s.charAt(0);                // Get char
int len = s.length();                // Length
String sub = s.substring(0, 3);      // "hel"
s.toLowerCase();
s.toUpperCase();
s.trim();                            // Remove spaces

// String to char array
char[] chars = s.toCharArray();

// StringBuilder (for modifications)
StringBuilder sb = new StringBuilder();
sb.append("hello");
sb.append(" world");
sb.reverse();
String result = sb.toString();
```

### Lists
```java
// ArrayList (use this 99% of time)
List<Integer> list = new ArrayList<>();
list.add(5);
list.add(0, 10);        // Add at index
list.get(0);            // Get
list.set(0, 20);        // Update
list.remove(0);         // Remove by index
list.size();            // Size
list.contains(5);       // Check exists

// Convert to array
Integer[] arr = list.toArray(new Integer[0]);

// Convert array to list
List<Integer> list = Arrays.asList(1, 2, 3);
// OR (mutable)
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));
```

### Maps (HashMap)
```java
Map<String, Integer> map = new HashMap<>();
map.put("key", 10);
map.get("key");                      // Returns value or null
map.getOrDefault("key", 0);          // Returns 0 if not exists
map.containsKey("key");
map.remove("key");

// Iterate
for (String key : map.keySet()) {
    int value = map.get(key);
}

for (Map.Entry<String, Integer> entry : map.entrySet()) {
    String key = entry.getKey();
    int value = entry.getValue();
}
```

### Sets
```java
Set<Integer> set = new HashSet<>();
set.add(5);
set.contains(5);     // O(1) lookup
set.remove(5);
set.size();

// Convert list to set (remove duplicates)
Set<Integer> set = new HashSet<>(list);
```

### Queue
```java
Queue<Integer> queue = new LinkedList<>();
queue.offer(5);      // Add
queue.poll();        // Remove and return
queue.peek();        // Look at front
queue.isEmpty();
```

### Stack
```java
Stack<Integer> stack = new Stack<>();
stack.push(5);       // Add
stack.pop();         // Remove and return
stack.peek();        // Look at top
stack.isEmpty();
```

### PriorityQueue (Heap)
```java
// Min Heap (default)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Max Heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);

minHeap.offer(5);
minHeap.poll();      // Remove min
minHeap.peek();      // Look at min
```

### Common Tricks
```java
// Swap
int temp = a;
a = b;
b = temp;

// Max/Min
int max = Math.max(a, b);
int min = Math.min(a, b);

// Absolute value
int abs = Math.abs(-5);  // 5

// Power
int result = (int) Math.pow(2, 3);  // 8

// Integer to String
String s = String.valueOf(123);
String s = Integer.toString(123);

// String to Integer
int num = Integer.parseInt("123");

// Character operations
Character.isDigit('5');          // true
Character.isLetter('a');         // true
Character.toLowerCase('A');      // 'a'
```

---

## 📝 COMMON QUESTIONS BY PATTERN {#common-questions}

### Hash Map Pattern
- [ ] Two Sum
- [ ] Group Anagrams
- [ ] First Unique Character in String
- [ ] Subarray Sum Equals K
- [ ] Longest Substring Without Repeating Characters

### Two Pointers
- [ ] Two Sum II (sorted)
- [ ] Remove Duplicates from Sorted Array
- [ ] Valid Palindrome
- [ ] Container With Most Water
- [ ] 3Sum

### Sliding Window
- [ ] Maximum Sum Subarray of Size K
- [ ] Longest Substring Without Repeating Characters
- [ ] Minimum Window Substring
- [ ] Longest Repeating Character Replacement

### Binary Search
- [ ] Binary Search
- [ ] Search in Rotated Sorted Array
- [ ] Find First and Last Position
- [ ] Search Insert Position

### DFS (Tree/Graph)
- [ ] Maximum Depth of Binary Tree
- [ ] Validate Binary Search Tree
- [ ] Number of Islands
- [ ] Path Sum
- [ ] Invert Binary Tree

### BFS
- [ ] Binary Tree Level Order Traversal
- [ ] Minimum Depth of Binary Tree
- [ ] Rotting Oranges

### Backtracking
- [ ] Subsets
- [ ] Permutations
- [ ] Combination Sum
- [ ] Letter Combinations of a Phone Number

### Dynamic Programming
- [ ] Climbing Stairs
- [ ] House Robber
- [ ] Coin Change
- [ ] Longest Increasing Subsequence

### Heap
- [ ] Kth Largest Element
- [ ] Top K Frequent Elements
- [ ] Merge K Sorted Lists

### Fast & Slow Pointers
- [ ] Linked List Cycle
- [ ] Middle of Linked List
- [ ] Happy Number

---

## 😱 WHEN YOU'RE STUCK (PANIC MODE) {#panic-mode}

### Step 1: Don't freeze. Say this:
> "Let me think about this for a moment... I want to consider a few approaches."

### Step 2: Try these in order:

**Option A: Brute Force**
- Nested loops
- Try all combinations
- Say: "The brute force would be O(n²) but let me see if I can optimize..."

**Option B: Use a Hash Map**
- 70% of the time, a hash map helps
- Count frequencies
- Store seen values

**Option C: Sort it first**
- Sometimes sorting makes the problem trivial
- Ask: "Can I sort this?"

**Option D: Think of similar problems**
- "This reminds me of Two Sum..."
- "This is similar to finding duplicates..."

### Step 3: Talk through your thought process
```
"I notice this is asking for a contiguous subarray, so I'm thinking
sliding window might work here. Let me trace through an example..."
```

### Step 4: If still stuck, ask for a hint
> "I'm thinking this might be a dynamic programming problem, but I'm not
> sure how to define the subproblem. Could you give me a hint?"

---

## ✅ FINAL CHECKLIST BEFORE INTERVIEW {#final-checklist}

### 1 Day Before:
- [ ] Review all 10 patterns (just read the templates)
- [ ] Do 3 easy problems (to build confidence)
- [ ] Review common mistakes below

### 2 Hours Before:
- [ ] Review Java syntax section
- [ ] Do 1 easy problem (warm up)
- [ ] Review "When You're Stuck" section

### During Interview:
- [ ] **Listen carefully** - underline key words
- [ ] **Clarify inputs** - "Can the array be empty?" "Can it have negatives?"
- [ ] **Talk out loud** - explain your thinking
- [ ] **Start with brute force** - then optimize
- [ ] **Trace through example** - use small example
- [ ] **Consider edge cases** - empty input, single element, all same values
- [ ] **Test your code** - walk through line by line
- [ ] **Mention time/space complexity** - even if not asked

### Common Mistakes to Avoid:
1. **Off-by-one errors**: `i < array.length` vs `i <= array.length`
2. **Null checks**: Always check `if (root == null)` for trees
3. **Array index out of bounds**: Check `i < array.length` before accessing
4. **Integer overflow**: Use `left + (right - left) / 2` instead of `(left + right) / 2`
5. **Modifying while iterating**: Can't remove from list while in for-each loop
6. **String concatenation in loop**: Use StringBuilder for efficiency
7. **Forgetting to copy list**: `new ArrayList<>(current)` not just `current`
8. **Queue/Stack confusion**: offer/poll for Queue, push/pop for Stack

---

## 🎓 INTERVIEW DAY MANTRAS

**Remember:**
1. You don't need to solve it perfectly. You need to show you can think.
2. Talking through your process is 50% of the score.
3. It's okay to start with brute force and optimize later.
4. Asking clarifying questions shows intelligence, not weakness.
5. If stuck, ask for a hint. It's better than sitting silent.

**Before you start coding:**
- "Let me make sure I understand the problem..."
- "What should I return if the input is empty?"
- "Are there any constraints on the input size?"

**While coding:**
- "I'm using a hash map here to track..."
- "This gives us O(n) time complexity because..."
- "Let me trace through an example to verify..."

**After coding:**
- "Let me walk through a test case..."
- "Edge cases I should consider are..."
- "The time complexity is O(n) and space is O(1)..."

---

## 🚀 YOU GOT THIS!

You don't need to be the smartest person in the room. You just need to:
1. Recognize the pattern (use the checklist)
2. Apply the template (memorize the 10 templates)
3. Talk through your thinking (even if you're wrong)

That's literally it. Good luck!

---

**Last Minute Tips:**
- Breathe. You've prepared.
- The interviewer wants you to succeed.
- Perfect is the enemy of good. Working solution > Perfect solution.
- You've got 10 patterns memorized. That's more than most candidates.

**Now go ace that interview!** 💪
