

### 1. Sliding Window (Fixed Size)

This pattern is perfect for finding the maximum sum of a contiguous subarray of size `k`.

```rust
fn max_sum_subarray(arr: &[i32], k: usize) -> Option<i32> {
    if arr.len() < k { return None; }

    let mut window_sum: i32 = arr[..k].iter().sum();
    let mut max_sum = window_sum;

    for i in k..arr.len() {
        window_sum += arr[i] - arr[i - k]; // Slide the window
        max_sum = max_sum.max(window_sum);
    }
    Some(max_sum)
}

```

### 2. Two Pointers (Fast and Slow)

Used here to detect a cycle in a linked list (Floyd's Cycle-Finding Algorithm).

```rust
// Simplified logic for a linked list node
fn has_cycle(head: Option<&Node>) -> bool {
    let mut slow = head;
    let mut fast = head;

    while let (Some(s), Some(f)) = (slow, fast.and_then(|n| n.next.as_deref())) {
        slow = s.next.as_deref();
        fast = f.next.as_deref(); // Fast moves twice
        
        if std::ptr::eq(s, f) { return true; }
    }
    false
}

```

### 3. Prefix Sum

Great for pre-calculating range sums in $O(1)$ time after an $O(n)$ setup.

```rust
struct PrefixSum {
    prefix: Vec<i32>,
}

impl PrefixSum {
    fn new(nums: Vec<i32>) -> Self {
        let mut prefix = vec![0; nums.len() + 1];
        for i in 0..nums.len() {
            prefix[i + 1] = prefix[i] + nums[i];
        }
        Self { prefix }
    }

    fn query(&self, left: usize, right: usize) -> i32 {
        self.prefix[right + 1] - self.prefix[left]
    }
}

```

### 4. Binary Search

The classic "divide and conquer" for sorted arrays!

```rust
fn binary_search(arr: &[i32], target: i32) -> Option<usize> {
    let (mut left, mut right) = (0, arr.len() as i32 - 1);

    while left <= right {
        let mid = left + (right - left) / 2;
        if arr[mid as usize] == target { return Some(mid as usize); }
        if arr[mid as usize] < target { left = mid + 1; } 
        else { right = mid - 1; }
    }
    None
}

```

### 5. Heaps (Priority Queue)

Rust provides `BinaryHeap` in the standard library for this pattern.

```rust
use std::collections::BinaryHeap;

fn get_k_largest(nums: Vec<i32>, k: usize) -> Vec<i32> {
    let mut heap = BinaryHeap::from(nums); // Max-heap by default
    (0..k).filter_map(|_| heap.pop()).collect()
}

```


### 6. Stacks (LIFO)

Rust's `Vec` is the standard way to implement a stack using `push` and `pop`.

```rust
fn bracket_matcher(s: &str) -> bool {
    let mut stack = Vec::new();
    for c in s.chars() {
        match c {
            '(' => stack.push(')'),
            '{' => stack.push('}'),
            '[' => stack.push(']'),
            target if Some(target) != stack.pop() => return false,
            _ => (),
        }
    }
    stack.is_empty()
}

```

### 7. Trees (DFS/BFS)

Using `Option<Box<Node>>` is the idiomatic way to handle recursive pointers in Rust.

```rust
struct Node {
    val: i32,
    left: Option<Box<Node>>,
    right: Option<Box<Node>>,
}

fn inorder_traversal(root: Option<&Node>) {
    if let Some(node) = root {
        inorder_traversal(node.left.as_deref());
        println!("{}", node.val);
        inorder_traversal(node.right.as_deref());
    }
}

```

### 8. Tries (Prefix Trees)

Excellent for autocomplete features or dictionary lookups.

```rust
use std::collections::HashMap;

#[derive(Default)]
struct TrieNode {
    children: HashMap<char, TrieNode>,
    is_end: bool,
}

impl TrieNode {
    fn insert(&mut self, word: &str) {
        let mut curr = self;
        for c in word.chars() {
            curr = curr.children.entry(c).or_default();
        }
        curr.is_end = true;
    }
}

```

### 9. Graphs (Adjacency List)

Represented efficiently using a `HashMap` or a `Vec` of `Vec`.

```rust
use std::collections::{HashMap, VecDeque};

fn bfs(graph: &HashMap<i32, Vec<i32>>, start: i32) {
    let mut visited = std::collections::HashSet::new();
    let mut queue = VecDeque::new();

    queue.push_back(start);
    visited.insert(start);

    while let Some(node) = queue.pop_front() {
        if let Some(neighbors) = graph.get(&node) {
            for &next in neighbors {
                if visited.insert(next) { queue.push_back(next); }
            }
        }
    }
}

```

### 10. Backtracking

Solving the "Subset" problem using recursion.

```rust
fn backtrack(nums: &[i32], start: usize, path: &mut Vec<i32>, res: &mut Vec<Vec<i32>>) {
    res.push(path.clone());
    for i in start..nums.len() {
        path.push(nums[i]);
        backtrack(nums, i + 1, path, res);
        path.pop(); // Undo the move
    }
}

```

### 11. Dynamic Programming (Memoization)

Using a `HashMap` or `Vec` to store previously computed results.

```rust
fn fib(n: usize, memo: &mut Vec<i64>) -> i64 {
    if n <= 1 { return n as i64; }
    if memo[n] != -1 { return memo[n]; }
    
    memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
    memo[n]
}

```

### 12. Greedy Algorithm

Solving the "Interval Scheduling" or "Coin Change" problem by making the locally optimal choice.

```rust
fn min_coins(mut amount: i32, coins: &[i32]) -> i32 {
    let mut count = 0;
    // Assume coins are sorted descending
    for &coin in coins {
        count += amount / coin;
        amount %= coin;
    }
    count
}

```

### 13. Intervals

Merging overlapping intervals using sorting and a single pass.

```rust
fn merge_intervals(mut intervals: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    intervals.sort_unstable_by_key(|i| i[0]);
    let mut merged: Vec<Vec<i32>> = Vec::new();

    for interval in intervals {
        if let Some(last) = merged.last_mut() {
            if interval[0] <= last[1] {
                last[1] = last[1].max(interval[1]);
                continue;
            }
        }
        merged.push(interval);
    }
    merged
}

```

### 14. Two Pointers (Opposite Ends)

Commonly used for reversing a string or checking palindromes.

```rust
fn is_palindrome(s: &str) -> bool {
    let chars: Vec<char> = s.chars().collect();
    let (mut i, mut j) = (0, chars.len().saturating_sub(1));
    
    while i < j {
        if chars[i] != chars[j] { return false; }
        i += 1;
        j -= 1;
    }
    true
}

```

### 15. K-Way Merge

Merging `k` sorted lists using a `BinaryHeap`.

```rust
use std::collections::BinaryHeap;
use std::cmp::Ordering;

#[derive(Eq, PartialEq)]
struct State { val: i32, list_idx: usize, elem_idx: usize }
impl Ord for State { 
    fn cmp(&self, other: &Self) -> Ordering { other.val.cmp(&self.val) } // Min-heap
}
impl PartialOrd for State {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> { Some(self.cmp(other)) }
}

```

### 16. Topological Sort (Kahn's Algorithm)

Used for resolving dependencies in a Directed Acyclic Graph (DAG).

```rust
fn topo_sort(nodes: usize, edges: Vec<(usize, usize)>) -> Vec<usize> {
    let mut in_degree = vec![0; nodes];
    let mut adj = vec![vec![]; nodes];
    for (u, v) in edges {
        adj[u].push(v);
        in_degree[v] += 1;
    }

    let mut queue: VecDeque<_> = (0..nodes).filter(|&i| in_degree[i] == 0).collect();
    let mut result = Vec::new();

    while let Some(u) = queue.pop_front() {
        result.push(u);
        for &v in &adj[u] {
            in_degree[v] -= 1;
            if in_degree[v] == 0 { queue.push_back(v); }
        }
    }
    result
}

```

