
Depth-first search(DFS), breadth-first search(BFS) and two pointers, make up a good portion of all interview problems. DFS in particular can be used to solve a wide range of problems from tree to graph to combinatorial problems and is very useful in tech interviews

***if you can't figure out a dynamic programming solution, you can always do DFS + memoization which does the same thing.***

Master: Linked List, Array, Hash Map, Stack, Queue, Sorting.

*A good-enough rule of thumb is an algorithm named after a person(s) you can safely ignore.*

#### 1. House Robber (Dynamic Programming)
This approach uses iteration with space optimization to calculate the maximum loot without ever robbing two adjacent houses.

```rust
fn calculate_max_robbery_loot(house_values: Vec<i32>) -> i32 {
    // Tracks the max loot possible up to two houses ago
    let mut max_loot_two_houses_back = 0;
    // Tracks the max loot possible up to the previous house
    let mut max_loot_one_house_back = 0;

    for current_house_value in house_values {
        // Store the previous best before updating
        let previous_best = max_loot_one_house_back;
        
        // At each house, decide: 
        // 1. Rob current house + loot from 2 houses back
        // 2. Skip current house and keep loot from 1 house back
        max_loot_one_house_back = std::cmp::max(
            max_loot_one_house_back, 
            max_loot_two_houses_back + current_house_value
        );
        
        // Move the window forward: "one back" becomes "two back" for the next iteration
        max_loot_two_houses_back = previous_best;
    }

    // The result is the maximum accumulated value at the end of the street
    max_loot_one_house_back
}
```

#### 2. Robot Paths (Unique Paths)
This implementation finds the number of unique paths from the top-left to the bottom-right of a grid using a space-optimized 1D DP array.

```rust
fn calculate_unique_robot_paths(total_rows: i32, total_columns: i32) -> i32 {
    let num_rows = total_rows as usize;
    let num_cols = total_columns as usize;
    
    // Initialize a row representing the number of ways to reach each column.
    // Every cell in the first row has exactly 1 way to be reached (moving only right).
    let mut paths_to_current_row = vec![1; num_cols];

    // Iterate through each row starting from the second one
    for _current_row_index in 1..num_rows {
        // Iterate through each column starting from the second one
        for column_index in 1..num_cols {
            // The paths to the current cell is the sum of:
            // - paths_to_current_row[column_index]: ways from the cell directly above
            // - paths_to_current_row[column_index - 1]: ways from the cell to the left
            paths_to_current_row[column_index] += paths_to_current_row[column_index - 1];
        }
    }

    // The last element contains the total unique paths to the bottom-right corner
    paths_to_current_row[num_cols - 1]
}
```

#### How to calculate MOD from scratch
The rule to calculate `x % y` is:

- `if x < y`, `return x`
- `else` subtract `y` from `x` until `x < y`

```rust
fn mod(mut x: u32, mut y: u32) -> u32{
  while (x >= y) {
    x -= y
  }
  return x
}
```

### 1. Sliding Window (Fixed Size)

This pattern is perfect for finding the maximum sum of a contiguous subarray of size `k`.

```rust

fn max_sum_subarray(arr: &[i32], k: usize) -> Option<i32> {
    // Step 1: Handle the edge case where the array is shorter than the window
    if arr.len() < k { 
        return None; 
    }

    // Step 2: Calculate the sum of the very first window manually
    let mut current_window_sum: i32 = 0;
    for i in 0..k {
        current_window_sum += arr[i];
    }
    
    let mut absolute_max_sum: i32 = current_window_sum;

    // Step 3: Slide the window across the rest of the array
    for current_index in k..arr.len() {
        let index_leaving_window = current_index - k;
        let element_leaving_window = arr[index_leaving_window];
        let element_entering_window = arr[current_index];
        
        // The magic O(1) mathematical operations!
        current_window_sum = current_window_sum - element_leaving_window + element_entering_window;
        
        // Step 4: Update the maximum sum if we found a new high score
        if current_window_sum > absolute_max_sum {
            absolute_max_sum = current_window_sum;
        }
    }
    
    Some(absolute_max_sum)
}


```

### 2. Two Pointers (Fast and Slow)

Commonly known as Floyd’s Cycle-Finding Algorithm (the "Tortoise and the Hare"), this pattern detects cycles in linked structures using two pointers moving at different speeds.

```rust
// A simplified representation of a linked list node for logic demonstration
fn detect_linked_list_cycle(head_node: Option<&Node>) -> bool {
    // Initialize both pointers at the start of the list
    let mut slow_pointer = head_node;
    let mut fast_pointer = head_node;

    // Use 'and_then' to safely check if the fast pointer can move two steps ahead
    while let (Some(slow), Some(fast)) = (slow_pointer, fast_pointer.and_then(|node| node.next_node.as_deref())) {
        // Move the slow pointer one step
        slow_pointer = slow.next_node.as_deref();
        // Move the fast pointer two steps (the next of the current 'fast' node)
        fast_pointer = fast.next_node.as_deref();
        
        // If the fast pointer eventually catches up to the slow pointer, a cycle exists
        // We use std::ptr::eq to compare the memory addresses of the nodes
        if std::ptr::eq(slow, fast) { 
            return true; 
        }
    }
    
    // If the fast pointer reaches the end (None), no cycle was detected
    false
}
```

### 3. Prefix Sum

This technique is ideal for Range Sum Queries, allowing you to calculate the sum of any subarray in constant time after a one-time linear preprocessing step.

```rust
struct RangeSumEngine {
    // We store an extra element at the start (usually 0) to simplify boundary logic
    accumulated_prefix_sums: Vec<i32>,
}

impl RangeSumEngine {
    /// Pre-calculates the running totals of the input numbers in O(n) time.
    fn new(input_numbers: Vec<i32>) -> Self {
        let mut prefix_buffer = vec![0; input_numbers.len() + 1];
        
        for current_index in 0..input_numbers.len() {
            // Each position stores the sum of all elements before it
            prefix_buffer[current_index + 1] = prefix_buffer[current_index] + input_numbers[current_index];
        }
        
        Self { accumulated_prefix_sums: prefix_buffer }
    }

    /// Retrieves the sum of elements from 'left_index' to 'right_index' (inclusive) in O(1).
    fn get_sum_in_range(&self, left_index: usize, right_index: usize) -> i32 {
        // Formula: Sum(left..right) = Prefix[right + 1] - Prefix[left]
        self.accumulated_prefix_sums[right_index + 1] - self.accumulated_prefix_sums[left_index]
    }
}
```

### 4. Binary Search

The classic "divide and conquer" algorithm for finding a target in a sorted collection by repeatedly 
halving the search space.

```rust
fn perform_binary_search(sorted_array: &[i32], target_value: i32) -> Option<usize> {
    // Initialize boundary pointers. We use i32 to safely handle the -1 case for 'right'
    let mut low_boundary = 0;
    let mut high_boundary = sorted_array.len() as i32 - 1;

    while low_boundary <= high_boundary {
        // Calculate the midpoint. 
        // We use (low + (high - low) / 2) to prevent integer overflow on large arrays.
        let mid_index = low_boundary + (high_boundary - low_boundary) / 2;
        let current_value = sorted_array[mid_index as usize];

        if current_value == target_value {
            // Target found! Return the index as a usize
            return Some(mid_index as usize);
        } else if current_value < target_value {
            // Target is in the upper half; move the low boundary up
            low_boundary = mid_index + 1;
        } else {
            // Target is in the lower half; move the high boundary down
            high_boundary = mid_index - 1;
        }
    }

    // Loop finished without finding the target
    None
}

```


##### Recursive approach:

```rust

fn binary_search(numbers: &[u32], target: u32) -> Option<u32> {
    if numbers.is_empty() {
        return None;
    }

    let mid = numbers.len() / 2;

    if numbers[mid] == target {
        Some(numbers[mid]) // Found it!
    } else if numbers[mid] < target {
        // Search the right half (exclude mid)
        binary_search(&numbers[mid + 1..], target)
    } else {
        // Search the left half (exclude mid)
        binary_search(&numbers[..mid], target)
    }
}
```

### 5. Heaps (Priority Queue)

Rust provides `BinaryHeap` in the standard library for this pattern.

```rust
use std::collections::BinaryHeap;

/// Extracts the top k largest elements from a collection.
fn get_top_k_largest_elements(input_numbers: Vec<i32>, k_count: usize) -> Vec<i32> {
    // BinaryHeap is a max-heap by default in Rust.
    // .from() converts the Vec into a heap in O(n) time.
    let mut max_priority_queue = BinaryHeap::from(input_numbers);
    
    // Extract elements one by one. pop() returns Option<T>.
    (0..k_count)
        .filter_map(|_| max_priority_queue.pop())
        .collect()
}
```


### 6. Stacks (LIFO)

Rust's `Vec` is the standard way to implement a stack using `push` and `pop`.

```rust

fn check_balanced_brackets(input_string: &str) -> bool {
    let mut bracket_stack = Vec::new();
    
    for current_character in input_string.chars() {
        match current_character {
            // Push the expected closing counterpart onto the stack
            '(' => bracket_stack.push(')'),
            '{' => bracket_stack.push('}'),
            '[' => bracket_stack.push(']'),
            // If it's a closing bracket, it must match the last pushed expectation
            closing_bracket if Some(closing_bracket) != bracket_stack.pop() => return false,
            _ => (), // Ignore non-bracket characters
        }
    }
    // If the stack is empty, all brackets were correctly matched and closed
    bracket_stack.is_empty()
}

```

### 7. Trees (DFS/BFS)

Using `Option<Box<Node>>` is the idiomatic way to handle recursive pointers in Rust.

```rust
struct TreeNode {
    value: i32,
    left_child: Option<Box<TreeNode>>,
    right_child: Option<Box<TreeNode>>,
}

fn perform_inorder_traversal(node_ref: Option<&TreeNode>) {
    if let Some(current_node) = node_ref {
        // Recursive call on the left child using .as_deref() to go from 
        // Option<Box<TreeNode>> to Option<&TreeNode>
        perform_inorder_traversal(current_node.left_child.as_deref());
        
        println!("Visited node: {}", current_node.value);
        
        // Recursive call on the right child
        perform_inorder_traversal(current_node.right_child.as_deref());
    }
}
```

### 8. Tries (Prefix Trees)

Excellent for autocomplete features or dictionary lookups.

```rust
use std::collections::HashMap;

#[derive(Default)]
struct TrieNode {
    children_map: HashMap<char, TrieNode>,
    is_terminal_node: bool,
}

impl TrieNode {
    fn insert_word(&mut self, word_to_insert: &str) {
        let mut current_pointer = self;
        for character in word_to_insert.chars() {
            // entry() allows us to find or create the character node efficiently
            current_pointer = current_pointer.children_map.entry(character).or_default();
        }
        // Mark the end of a valid word
        current_pointer.is_terminal_node = true;
    }
}
```

### 9. Graphs (Adjacency List)

Represented efficiently using a `HashMap` or a `Vec` of `Vec`.

```rust
use std::collections::{HashMap, VecDeque, HashSet};

fn breadth_first_search(graph_adjacency_list: &HashMap<i32, Vec<i32>>, starting_node: i32) {
    let mut visited_nodes = HashSet::new();
    let mut traversal_queue = VecDeque::new();

    traversal_queue.push_back(starting_node);
    visited_nodes.insert(starting_node);

    while let Some(current_node) = traversal_queue.pop_front() {
        if let Some(neighbors_list) = graph_adjacency_list.get(&current_node) {
            for &neighbor_node in neighbors_list {
                // insert() returns false if the value was already present
                if visited_nodes.insert(neighbor_node) {
                    traversal_queue.push_back(neighbor_node);
                }
            }
        }
    }
}

```

### 10. Backtracking

Solving the "Subset" problem using recursion.

```rust
fn generate_subsets(
    number_pool: &[i32], 
    current_index: usize, 
    current_path: &mut Vec<i32>, 
    all_subsets_result: &mut Vec<Vec<i32>>
) {
    // Record the current state of the path as a valid subset
    all_subsets_result.push(current_path.clone());
    
    for i in current_index..number_pool.len() {
        // Step forward: add element to path
        current_path.push(number_pool[i]);
        
        // Recurse to build further combinations
        generate_subsets(number_pool, i + 1, current_path, all_subsets_result);
        
        // Step back: remove element to try different branches (Backtrack)
        current_path.pop();
    }
}
```

### 11. Dynamic Programming (Memoization)

Using a `HashMap` or `Vec` to store previously computed results.

```rust
fn compute_fibonacci_with_memo(n_target: usize, result_cache: &mut Vec<i64>) -> i64 {
    // Base cases
    if n_target <= 1 { return n_target as i64; }
    
    // Check if result is already in the cache (initialized with -1)
    if result_cache[n_target] != -1 { 
        return result_cache[n_target]; 
    }
    
    // Recursive relation with storage
    let calculation = compute_fibonacci_with_memo(n_target - 1, result_cache) + 
                      compute_fibonacci_with_memo(n_target - 2, result_cache);
                      
    result_cache[n_target] = calculation;
    calculation
}

```

### 12. Greedy Algorithm

Solving the "Interval Scheduling" or "Coin Change" problem by making the locally optimal choice.

```rust
fn calculate_minimum_coins_required(mut remaining_amount: i32, coin_denominations: &[i32]) -> i32 {
    let mut total_coin_count = 0;
    
    // For greedy to work here, denominations must be sorted in descending order
    for &coin_value in coin_denominations {
        // Take as many of the current largest coin as possible
        total_coin_count += remaining_amount / coin_value;
        
        // Update the remaining balance
        remaining_amount %= coin_value;
    }
    total_coin_count
}

```

### 13. Intervals

Merging overlapping intervals using sorting and a single pass.

```rust
fn merge_overlapping_intervals(mut input_intervals: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    // Sort intervals by their start time to ensure linear processing
    input_intervals.sort_unstable_by_key(|interval| interval[0]);
    
    let mut merged_list: Vec<Vec<i32>> = Vec::new();

    for current_interval in input_intervals {
        if let Some(last_merged) = merged_list.last_mut() {
            // Check if current interval starts before or at the end of the previous one
            if current_interval[0] <= last_merged[1] {
                // Merge by extending the end time to the maximum of both
                last_merged[1] = last_merged[1].max(current_interval[1]);
                continue;
            }
        }
        // No overlap found, add as a new interval
        merged_list.push(current_interval);
    }
    merged_list
}

```

### 14. Two Pointers (Opposite Ends)

Commonly used for reversing a string or checking palindromes.

```rust
fn is_palindrome_check(input_text: &str) -> bool {
    let character_buffer: Vec<char> = input_text.chars().collect();
    let mut left_pointer = 0;
    let mut right_pointer = character_buffer.len().saturating_sub(1);
    
    while left_pointer < right_pointer {
        if character_buffer[left_pointer] != character_buffer[right_pointer] {
            return false;
        }
        left_pointer += 1;
        right_pointer -= 1;
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
struct NodeState { 
    value: i32, 
    list_index: usize, 
    element_index: usize 
}

// Custom Ord implementation to turn BinaryHeap into a min-heap
impl Ord for NodeState { 
    fn cmp(&self, other: &Self) -> Ordering { 
        other.value.cmp(&self.value) // Flip comparison for min-heap behavior
    } 
}

impl PartialOrd for NodeState {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> { 
        Some(self.cmp(other)) 
    }
}

```

### 16. Topological Sort (Kahn's Algorithm)

Used for resolving dependencies in a Directed Acyclic Graph (DAG).

```rust
fn perform_topological_sort(total_nodes: usize, dependency_edges: Vec<(usize, usize)>) -> Vec<usize> {
    let mut incoming_degree_count = vec![0; total_nodes];
    let mut adjacency_list = vec![vec![]; total_nodes];
    
    // Build graph and count how many prerequisites each node has
    for (source, destination) in dependency_edges {
        adjacency_list[source].push(destination);
        incoming_degree_count[destination] += 1;
    }

    // Initialize queue with all nodes that have no prerequisites
    let mut zero_prerequisite_queue: VecDeque<_> = (0..total_nodes)
        .filter(|&index| incoming_degree_count[index] == 0)
        .collect();
        
    let mut sorted_order_result = Vec::new();

    while let Some(current_node) = zero_prerequisite_queue.pop_front() {
        sorted_order_result.push(current_node);
        
        for &neighbor_node in &adjacency_list[current_node] {
            // Decrement dependency count for neighbors
            incoming_degree_count[neighbor_node] -= 1;
            
            // If all prerequisites are met, add to queue
            if incoming_degree_count[neighbor_node] == 0 {
                zero_prerequisite_queue.push_back(neighbor_node);
            }
        }
    }
    sorted_order_result
}

```

