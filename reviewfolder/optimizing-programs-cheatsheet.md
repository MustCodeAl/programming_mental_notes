# ⚡ Optimizing Programs — Rust Performance Cheat Sheet

> A practical Rust performance reference for readers who may be completely new to programming. Technical terms are still used, but they are explained in ordinary English when they appear.
> Each optimization includes a ❌ slower/less efficient approach and a ✅ improved approach.

---

💡 **How to use this guide:** Start with profiling, fix algorithmic problems first, then apply lower-level optimizations only where measurements show they matter.

> **Rust examples:** Each rule uses a ❌ / ✅ comparison. Comments explain what the computer is doing, what practical problem that causes, and then introduce the technical term. The ✅ side favors complete functions, descriptive names, useful parameters, return values, and patterns that can be adapted in real projects.
>
> **Comment style:** Plain English comes first. Technical vocabulary comes second, with its meaning explained in the same comment instead of assuming the reader already knows it.

## 🏁 Golden Rules (Read This First)

### Start from a good baseline.

Rust is already fast/memory-frugal by default — just don't forget `--release`.

```rust
// ❌
// Rust debug builds include extra checks and little compiler optimization, so
// timing them makes normal production code look much slower than it really is.
let mut sum = 0u64;
for index in 0..1_000_000 { sum += index; }


// ✅
// Measure optimized release builds. `cargo build --release` lets the compiler
// remove unnecessary work and generate the kind of machine code you would
// normally ship.
fn main() { println!("optimized build"); }
```

### Only optimize what's hot.

Optimized code is harder to read and buggier. Spend that budget only where a profiler proves it matters.

```rust
// ❌
// This spends engineering time changing code before we know whether that code is
// causing any noticeable slowdown. A hot path is code that runs very often or
// takes a large share of the program's total time.
fn cold_path(value: u64) -> u64 { (0..value).sum() }
let _ = cold_path(10);


// ✅
// Measure the real workload first, then optimize the functions that actually
// consume meaningful time or memory. This gives you a performance improvement
// users can notice instead of making random code harder to maintain.
use std::hint::black_box;

fn hot(values: &[u64]) -> u64 { values.iter().copied().sum() }
fn main() { let values = vec![1; 10_000]; black_box(hot(black_box(&values))); }
```

### Algorithms beat micro-tuning.

O(N²) → O(N log N) crushes any low-level trick. Fix the algorithm first.

```rust
// ❌
// This searches the same lists again and again. As the lists grow, the amount of
// work grows much faster than the amount of data; this is called poor algorithmic
// complexity.
for value in &left {
    for other_value in &right { if value == other_value { found += 1;
    } };
}
let left = vec![1, 2];
let right = vec![2, 3];
let mut found = 0;


// ✅
// Choose an algorithm that avoids repeated searching. A better algorithm can
// remove thousands or millions of operations, which usually matters far more than
// making one CPU instruction slightly faster.
let mut values = vec![9, 1, 7, 3];
values.sort_unstable();
let found = values.binary_search(&7).is_ok();
assert!(found);
```

### Design for the hardware.

Minimizing cache misses and branch mispredictions genuinely speeds up real CPUs.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
use std::collections::LinkedList;
let data: LinkedList<u64> = (0..1_000_000).collect();
let sum: u64 = data.iter().sum();


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
let data = vec![1u64; 1_000_000];
let sum: u64 = data.iter().copied().sum();
assert_eq!(sum, 1_000_000);
```

### Small wins compound.

Dozens of 1-2% gains add up — don't dismiss "minor" optimizations.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
let mut output = Vec::new();
for index in 0..1024 {
    output.push(index);
}
output.sort();


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
let mut output = Vec::with_capacity(1024);
output.extend(0..1024);
output.sort_unstable();
```

### Use multiple profilers.

CPU, memory, and causal profilers each surface different bottlenecks.

```rust
// ❌
// This timer answers only one question: "How long did the whole operation take?"
// If it reports 500 ms, you still do not know whether the program spent that time
// calculating, allocating memory, reading a file, waiting for the network, or
// waiting for another thread. A profiler is a tool that helps answer those
// more specific questions.
use std::time::{Duration, Instant};

fn measure_total_time(operation: impl FnOnce()) -> Duration {
    let started_at = Instant::now();
    operation();
    started_at.elapsed()
}

// ✅
// Keep a small timing helper like this for real application measurements, but
// combine it with specialized tools when a slowdown matters. For example:
// - a CPU profiler shows which functions keep the processor busy;
// - a memory profiler shows where allocations and memory growth happen;
// - tracing shows where a request spends time waiting between operations.
use std::time::{Duration, Instant};

fn measure_operation<T>(
    operation_name: &str,
    operation: impl FnOnce() -> T,
) -> (T, Duration) {
    let started_at = Instant::now();
    let result = operation();
    let elapsed = started_at.elapsed();

    eprintln!("{operation_name} finished in {elapsed:?}");
    (result, elapsed)
}
```

### Two ways to fix a hot function:

make it faster, or call it less often. The second is often the bigger, easier win.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
for _ in 0..1_000 {
    std::hint::black_box(expensive(42));
}
fn expensive(value:u64)->u64{ value*value }


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
fn expensive(value: u64) -> u64 { value * value }
let cached = expensive(42);
for _ in 0..1_000 { std::hint::black_box(cached); }
```

### Fix silly slowdowns first.

An accidental O(N²) loop, a stray clone, or unbuffered I/O beats inventing a clever optimization.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
let text = String::from("hello");
for _ in 0..1000 {
    consume(text.clone());
}
fn consume(_:String){ }


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
let src = String::from("hello");
let borrowed: &str = &src;
println!("{borrowed}");
```

### Don't compute what you don't need.

Lazy eval, early returns, and short-circuiting often beat speeding up the computation itself.

```rust
// ❌
// This creates one million squared numbers and stores all of them in a new vector
// before checking which one is large enough. Most of that calculation and memory
// is wasted because the program only needs the first matching value.
fn first_large_square_slow(limit: u64) -> Option<u64> {
    let squares: Vec<u64> = (0..1_000_000)
        .map(|number| number * number)
        .collect();

    squares.into_iter().find(|square| *square > limit)
}

// ✅
// The iterator calculates one square at a time and stops immediately after it
// finds the first match. This behavior is called lazy evaluation: work is delayed
// until it is actually needed.
fn first_large_square(limit: u64) -> Option<u64> {
    (0..1_000_000)
        .map(|number| number * number)
        .find(|square| *square > limit)
}
```

### Fast-path the common case.

If 90% of calls fit a simple pattern, special-case it before the general logic.

```rust
// ❌
// `chars().count()` decodes the text as Unicode every time, even when the input
// contains only ASCII characters. ASCII text uses one byte per character, so that
// extra Unicode decoding work is unnecessary for the common ASCII case.
fn character_count_slow(text: &str) -> usize {
    text.chars().count()
}

// ✅
// Check the cheap common case first. For ASCII, the byte length is already the
// character count. Non-ASCII text still uses the fully correct Unicode path.
// A fast path is a simple path designed for the input the program receives most
// often.
fn character_count(text: &str) -> usize {
    if text.is_ascii() {
        return text.len();
    }

    text.chars().count()
}
```

### Compress repetitive data.

Pack common values compactly; fall back to a secondary table for rare ones.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
let tags = vec![String::from("A"), String::from("A"), String::from("A")];
std::hint::black_box(tags);


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
#[derive(Clone, Copy)]
enum Tag { Common(u8), Rare(u32) }
let item = Tag::Common(3);
std::hint::black_box(item);
```

### Profile before ordering branches.

Put the statistically most frequent case first in `if`/`match`.

```rust
// ❌
// This makes the CPU repeatedly decide between paths in a way that is unnecessary
// or poorly arranged. Modern CPUs try to guess which branch will run next; a
// wrong guess, called a branch misprediction, throws away partially completed
// work.
fn classify(value:u8)->u8 { if value >= 250 { 1 } else { 0 } }
let _ = classify(1);


// ✅
// Keep the common case simple and avoid checking conditions inside a loop when
// the answer cannot change. Fewer or more predictable branches give the CPU less
// work to throw away.
fn classify(value: u8) -> u8 {
    if value < 250 { 0 }
    else { 1 }
}
```

### Cache in front of hot lookups.

A small cache absorbs repeated queries to the same few keys.

```rust
// ❌
// This calls the expensive data source every time the same user is requested.
// If `load_user_from_database` performs a database or network request, the
// program
// pays that latency again even when it already fetched the exact same user
// earlier.
fn get_user_name_slow(user_id: u64) -> String {
    load_user_from_database(user_id)
}

// ✅
// Check a small in-memory cache first. Memory lookup is usually far cheaper than
// another database or network round trip. A cache stores recently used results so
// repeated requests can reuse work that has already been completed.
use std::collections::HashMap;

struct UserCache {
    names_by_id: HashMap<u64, String>,
}

impl UserCache {
    fn new() -> Self {
        Self {
            names_by_id: HashMap::new(),
        }
    }

    fn get_user_name(&mut self, user_id: u64) -> String {
        if let Some(name) = self.names_by_id.get(&user_id) {
            return name.clone();
        }

        let name = load_user_from_database(user_id);
        self.names_by_id.insert(user_id, name.clone());
        name
    }
}

fn load_user_from_database(user_id: u64) -> String {
    format!("user-{user_id}")
}
```

### Comment the *why*.

Explain the profiling data behind a non-obvious optimization, not just what the code does.

```rust
// ❌
// This comment only repeats what the code already says. It does not tell the next
// developer what problem was measured, why this unusual code exists, or what
// could happen if it is simplified later.
if value > 0 { fast(); }
let value = 1; fn fast(){ }


// ✅
// Write the reason the code exists: include the measured behavior, the real
// bottleneck, and the trade-off. That gives future developers enough information
// to decide whether the optimization is still needed.
if likely_common_case() { fast() } else { slow() }
fn likely_common_case()->bool{true} fn fast(){ } fn slow(){ }
```


---

## 🏗️ High-Level Design

### ⚖️ The Space vs. Time Trade-off

Almost every optimization is really a decision about which resource you're willing to spend more of: memory or CPU time. Know which direction you're trading in, and pick deliberately based on what's actually scarce for your program.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
fn contains(values:&[u32], needle:u32)->bool { values.iter().any(|&value| value==needle) }
let _ = contains(&[1, 2, 3], 2);


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
use std::collections::HashMap;
let values = [10, 20, 30];
let index: HashMap<_, _> = values.iter().enumerate().map(|(index, values)| (*values, index)).collect();
assert_eq!(index.get(&20), Some(&1));
```


---

## ⌨️ Basic Coding Principles

### ✂️ Eliminate Excessive Function Calls

Move computations out of loops 🔄 whenever possible. You might even consider selective compromises of program modularity to gain greater efficiency.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
for value in &mut data {
    *value *= compute_scale();
}
fn compute_scale()->f64{ 2.0 } let mut data = vec![1.0, 2.0];


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
let scale = compute_scale();
for value in &mut data {
    *value *= scale;
}
fn compute_scale()->f64{ 2.0 } let mut data = vec![1.0, 2.0];
```

### 💾 Eliminate Unnecessary Memory References

Introduce temporary variables 📝 to hold intermediate results instead of mutating references directly.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
for &value in &data {
    *result += value;
}
let data = vec![1u64, 2, 3];
let result = &mut 0u64;


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
let mut total = 0u64;
for &value in &data { total += value; }
result = total;
let data = vec![1u64, 2, 3];
let mut result = 0;
```


---

## ⚙️ Low-Level Optimizations & Rust Patterns

### 🖲️ Memory References & Passing by Value

Eliminate unnecessary pointer dereferences by passing small types by value.

```rust
// ❌
// `Point` contains only two `f32` numbers, but this function receives a
// reference.
// That means the CPU may need an extra memory read just to reach a tiny value
// that
// would fit comfortably in registers, the CPU's fastest small storage locations.
#[derive(Clone, Copy)]
struct Point {
    horizontal: f32,
    vertical: f32,
}

fn squared_distance_slow(point: &Point) -> f32 {
    point.horizontal * point.horizontal + point.vertical * point.vertical
}

// ✅
// Small `Copy` values can often be passed directly. Rust copies the few bytes
// into
// the function, and the compiler can usually keep them in CPU registers.
// Use this for genuinely small types; large structs should normally be borrowed.
fn squared_distance(point: Point) -> f32 {
    point.horizontal * point.horizontal + point.vertical * point.vertical
}
```

### 🔄 Loop Locality & Bounds Checking

Structure loops to utilize spatial locality and use Iterators to remove hidden bounds checks.

```rust
// ❌
// Indexing asks Rust to prove on every access that `index` is inside the slice.
// The compiler can remove many of these checks, but complicated indexed loops may
// make that proof harder and the index itself does not explain what the code
// needs.
fn total_scores_slow(scores: &[u64]) -> u64 {
    let mut total = 0;

    for index in 0..scores.len() {
        total += scores[index];
    }

    total
}

// ✅
// Iterate over the values directly. The code says exactly what it means—read
// every
// score once—and gives the compiler a simple sequential memory-access pattern.
// Sequential access also helps the CPU cache, which keeps recently used memory
// nearby.
fn total_scores(scores: &[u64]) -> u64 {
    scores.iter().copied().sum()
}
```


---

## 🛡️ Bounds Safety, Unchecked Access & Pointers

### 🔓 Unchecked APIs (Bounds & Overflow Check Elimination)

Only after profiling shows bounds/overflow checks are a hot-path bottleneck and you can prove the access is always valid, use `unsafe {get_unchecked}` or similar.

```rust
// ❌
// This skips Rust's check that the requested position is actually inside the
// list. If the position is wrong, the program may read memory that belongs to
// something else or crash. That risk is called an output-of-bounds memory access.
// SAFETY: The check immediately above proves this memory access stays inside the
// slice.
unsafe { let value = *values.get_unchecked(index); std::hint::black_box(value); }
let values = [1u32, 2];
let index = 99usize;


// ✅
// Keep normal checked access unless a profiler proves the check itself is a real
// bottleneck. If unsafe access is truly needed, prove the index is valid first
// and document exactly why the unsafe read cannot leave the slice.
fn get_hot(values: &[u32], index: usize) -> u32 {
    assert!(index < values.len());
// SAFETY: The check immediately above proves this memory access stays inside the
// slice.
    unsafe { *values.get_unchecked(index) }
}
```

### 👉 Pointer Arithmetic vs. Pointer Indexing

Prefer indexing (bounds-checked slice/array access, or iterator-based traversal) over raw pointer arithmetic; reach for pointer arithmetic only in proven hot paths where you've already established the access pattern is safe.

```rust
// ❌
// A raw pointer is only a memory address. Rust cannot automatically check that
// `index` still points to a real element in `numbers`. If the math is wrong,
// the program can read unrelated memory, return incorrect data, or crash.
// Use raw pointers only when measurements prove normal safe access is too slow.
fn print_with_raw_pointer(numbers: &[i32]) {
    let first_number_address = numbers.as_ptr();

    for index in 0..numbers.len() {
        unsafe {
            println!("{}", *first_number_address.add(index));
        }
    }
}

// ✅
// A normal slice iterator expresses the same job directly: visit every number.
// Rust keeps memory access safe, the code is easier to review, and the compiler
// can usually generate machine code that is just as efficient for this traversal.
fn print_numbers(numbers: &[i32]) {
    for number in numbers {
        println!("{number}");
    }
}
```


---

## 📦 Allocation & Collection Management

### 📤 Hoisting Allocations Out of Loops

Move expensive string and heap allocations out of loops 🔄.

```rust
// ❌
// `format!` creates a brand-new `String` on every trip through the loop.
// Creating and later freeing all of those temporary strings makes the memory
// allocator do work that is not part of the actual business task.
fn format_item_labels_slow(item_count: usize) -> Vec<String> {
    let mut labels = Vec::with_capacity(item_count);

    for item_number in 0..item_count {
        labels.push(format!("item={item_number}"));
    }

    labels
}

// ✅
// Reuse one temporary string while producing each label. `clear()` removes the
// text but keeps the already-allocated memory, so the next iteration can reuse
// it.
// Clone only when a finished label must be stored independently in the returned
// list.
use std::fmt::Write;

fn format_item_labels(item_count: usize) -> Vec<String> {
    let mut labels = Vec::with_capacity(item_count);
    let mut reusable_label = String::with_capacity(32);

    for item_number in 0..item_count {
        reusable_label.clear();
        write!(&mut reusable_label, "item={item_number}").unwrap();
        labels.push(reusable_label.clone());
    }

    labels
}
```

### 📥 Reserve Capacity & Amortized Complexity

If you know (or can estimate) the final size of a collection, pre-allocate with `with_capacity`. Judge structure cost by average cost over a long sequence of operations, not the worst single call.

```rust
// ❌
// `Vec::new()` starts with no reserved storage. As more values are pushed in,
// the vector may run out of room, ask the allocator for a larger memory block,
// copy the existing values into that new block, and free the old block.
// That memory-growing step is called a reallocation.
fn build_order_ids_slow(order_count: usize) -> Vec<u64> {
    let mut order_ids = Vec::new();

    for order_number in 0..order_count {
        order_ids.push(order_number as u64);
    }

    order_ids
}

// ✅
// If the expected size is already known, reserve that space once.
// The vector can usually fill the existing memory instead of repeatedly
// growing and copying itself. This pattern is reusable anywhere a function
// builds a collection with a predictable final size.
fn build_order_ids(order_count: usize) -> Vec<u64> {
    let mut order_ids = Vec::with_capacity(order_count);

    for order_number in 0..order_count {
        order_ids.push(order_number as u64);
    }

    order_ids
}
```

### ♻️ Recycle Collections (Allocation Churn)

Inside a loop that rebuilds the same collection each iteration, `clear()` and reuse it instead of creating a new one.

```rust
// ❌
// This asks the memory allocator for new storage, copies data, or frees storage
// more often than necessary. The allocator is the part of the runtime that finds
// space in memory; calling it repeatedly adds bookkeeping work and can make the
// program pause more often.
for chunk in chunks {
    let scratch_buffer = chunk.to_vec();
    std::hint::black_box(scratch_buffer);
}
let chunks:Vec<&[u8]>=vec![b"a", b"bc"];


// ✅
// Reuse memory that already exists, reserve enough space before filling a
// collection, or borrow existing data when ownership is not needed. This reduces
// allocations and copies while keeping the code safe and predictable.
let mut scratch_buffer = Vec::with_capacity(1024);
for chunk in chunks {
    scratch_buffer.clear();
    scratch_buffer.extend_from_slice(chunk);
}
let chunks: Vec<&[u8]> = vec![b"a", b"bc"];
```

### ✍️ Append to Strings (Double-Allocation in String Building)

When building a string piece-by-piece, prefer `write!(&mut s, "...")` (via the `std::fmt::Write` trait) over `s += &format!(...)`.

```rust
// ❌
// This asks the memory allocator for new storage, copies data, or frees storage
// more often than necessary. The allocator is the part of the runtime that finds
// space in memory; calling it repeatedly adds bookkeeping work and can make the
// program pause more often.
let mut message = String::new();
for index in 0..10 {
    message += &format!("{index}, ");
}

// ✅
// Reuse memory that already exists, reserve enough space before filling a
// collection, or borrow existing data when ownership is not needed. This reduces
// allocations and copies while keeping the code safe and predictable.
use std::fmt::Write;
let mut text = String::with_capacity(64);
for index in 0..10 {
    write!(&mut text, "{index}, ").unwrap();
}
```

### 📚 Const Generics: Stack Arrays Instead of Heap `Vec`s

If a collection's size is fixed and known at compile time (a 3D vector's `[f32; 3]`, a fixed-size buffer, a small lookup table), use a plain array or a const-generic type instead of `Vec<T>`.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
let position = vec![1.0f32, 2.0, 3.0];
std::hint::black_box(position);


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
fn dot<const N: usize>(left: [f32; N], right: [f32; N]) -> f32 {
    left.into_iter().zip(right).map(|(value, other_value)| value*other_value).sum()
}
assert_eq!(dot([1.0, 2.0], [3.0, 4.0]), 11.0);
```

### 🏟️ Memory Arenas (Per-Object Allocation Overhead)

If you need to create thousands of small, short-lived objects (like parsing nodes in a compiler), do not use standard global allocations (`Box::new`). Use an Arena (Bump Allocator).

```rust
// ❌
// This asks the memory allocator for new storage, copies data, or frees storage
// more often than necessary. The allocator is the part of the runtime that finds
// space in memory; calling it repeatedly adds bookkeeping work and can make the
// program pause more often.
let nodes: Vec<Box<u32>> = (0..10_000).map(Box::new).collect();
std::hint::black_box(nodes);


// ✅
// Reuse memory that already exists, reserve enough space before filling a
// collection, or borrow existing data when ownership is not needed. This reduces
// allocations and copies while keeping the code safe and predictable.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
use bumpalo::Bump;
let arena = Bump::new();
let node = arena.alloc(("kind", 42u32));
```

### 🌍 Global Allocator (Default Allocator Contention)

For allocation-heavy workloads, swap Rust's default system allocator for a faster drop-in like `mimalloc` or `jemalloc`.

```rust
// ❌
// This asks the memory allocator for new storage, copies data, or frees storage
// more often than necessary. The allocator is the part of the runtime that finds
// space in memory; calling it repeatedly adds bookkeeping work and can make the
// program pause more often.
let strings:Vec<_>=(0..100_000).map(|index| format!("{index}")).collect();
std::hint::black_box(strings);


// ✅
// Reuse memory that already exists, reserve enough space before filling a
// collection, or borrow existing data when ownership is not needed. This reduces
// allocations and copies while keeping the code safe and predictable.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
use mimalloc::MiMalloc;
#[global_allocator]
static GLOBAL: MiMalloc = MiMalloc;
```

### 📋 Avoid Unnecessary Clones & Copies

Prefer borrowing (`&T`, `&str`, `&[T]`) and moving over `.clone()`; treat every clone in a hot path as a bug until profiling proves it is cheap.

```rust
// ❌
// This function takes ownership of the whole vector even though it only reads it.
// A caller that still needs the vector afterward may be forced to call
// `.clone()`,
// which copies every byte into new memory.
fn total_bytes_slow(bytes: Vec<u8>) -> u64 {
    bytes.into_iter().map(u64::from).sum()
}

// ✅
// Borrow the existing slice with `&[u8]`. Borrowing means the function may read
// the caller's data without taking it away or making a second copy.
// This is a common Rust API design for read-only project code.
fn total_bytes(bytes: &[u8]) -> u64 {
    bytes.iter().copied().map(u64::from).sum()
}
```

### 📦 Small-Buffer Optimization (Inline Storage)

For collections that are usually small (0–N elements with small N), use inline storage (`SmallVec`, `ArrayVec`, `ArrayString`, or a custom `enum { Inline([T; N]), Heap(Vec<T>) }`) to avoid heap allocation in the common case.

```rust
// ❌
// This asks the memory allocator for new storage, copies data, or frees storage
// more often than necessary. The allocator is the part of the runtime that finds
// space in memory; calling it repeatedly adds bookkeeping work and can make the
// program pause more often.
let mut values: Vec<u32> = Vec::new();
values.extend([1, 2, 3]);


// ✅
// Reuse memory that already exists, reserve enough space before filling a
// collection, or borrow existing data when ownership is not needed. This reduces
// allocations and copies while keeping the code safe and predictable.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
use smallvec::SmallVec;
let mut values: SmallVec<[u32; 8]> = SmallVec::new();
values.extend([1, 2, 3]);
```

### 🧹 Defer Drop (Synchronous Deallocation Stalls)

If dropping a large object (huge `Vec`, big `HashMap`) is expensive, `send` it to a background thread to be dropped instead of blocking the current one.

```rust
// ❌
// This asks the memory allocator for new storage, copies data, or frees storage
// more often than necessary. The allocator is the part of the runtime that finds
// space in memory; calling it repeatedly adds bookkeeping work and can make the
// program pause more often.
let huge = vec![0u8; 500_000_000];
drop(huge);


// ✅
// Reuse memory that already exists, reserve enough space before filling a
// collection, or borrow existing data when ownership is not needed. This reduces
// allocations and copies while keeping the code safe and predictable.
use std::sync::mpsc;
let (tx, rx) = mpsc::channel::<Vec<u8>>();
std::thread::spawn(move || for big in rx { drop(big); });
tx.send(vec![0; 10_000_000]).unwrap();
```


---

## ⚡ Instruction-Level Parallelism & Branch Optimization

### 🧮 Explicit SIMD & Auto-Vectorization

Prefer idiomatic iterators and contiguous data so LLVM auto-vectorizes; drop to explicit SIMD (`std::simd`, intrinsics, or crates like `wide`) only when the auto-vectorizer fails on a proven hot loop.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
let mut output = Vec::with_capacity(left.len());
for index in 0..left.len(){ output.push(left[index]+right[index]); }
let left = vec![1.0f32; 8];
let right = left.clone();


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
fn add(left: &[f32], right: &[f32], output: &mut [f32]) {
    for ((o, value), other_value) in output.iter_mut().zip(left).zip(right) {
        *o = *value + *other_value;
    }
}
```

### 🔀 Multiple Functional Units & Instruction-Level Parallelism

Unroll loops and use multiple accumulators to break calculation dependency chains, allowing the CPU to use multiple Arithmetic Logic Units (ALUs) concurrently.

```rust
// ❌
// Every addition depends on the total produced by the previous addition.
// The CPU must keep extending one long chain, so it has fewer independent
// additions
// available to execute at the same time.
fn sum_numbers_slow(numbers: &[u64]) -> u64 {
    let mut total = 0u64;

    for number in numbers {
        total = total.wrapping_add(*number);
    }

    total
}

// ✅
// Two running totals create two independent chains of additions. Some CPUs can
// work
// on those chains at the same time using separate arithmetic hardware. This is
// called
// instruction-level parallelism: independent CPU instructions overlap in
// execution.
fn sum_numbers(numbers: &[u64]) -> u64 {
    let mut even_positions_total = 0u64;
    let mut odd_positions_total = 0u64;

    for pair in numbers.chunks_exact(2) {
        even_positions_total = even_positions_total.wrapping_add(pair[0]);
        odd_positions_total = odd_positions_total.wrapping_add(pair[1]);
    }

    let paired_length = numbers.len() - numbers.len() % 2;
    let leftover_total = numbers[paired_length..]
        .iter()
        .copied()
        .fold(0u64, u64::wrapping_add);

    even_positions_total
        .wrapping_add(odd_positions_total)
        .wrapping_add(leftover_total)
}
```

### 🔣 Functional Style Conditional Operations

Break branching logic by using functional branching (`.map()`, `.filter()`).

```rust
// ❌
// This makes the CPU repeatedly decide between paths in a way that is unnecessary
// or poorly arranged. Modern CPUs try to guess which branch will run next; a
// wrong guess, called a branch misprediction, throws away partially completed
// work.
let mut output = Vec::new();
for value in 0..1000 {
    if value%2==0 { output.push(value*value);
    };
}

// ✅
// Keep the common case simple and avoid checking conditions inside a loop when
// the answer cannot change. Fewer or more predictable branches give the CPU less
// work to throw away.
let evens: Vec<_> = (0..10)
    .filter(|value| value % 2 == 0)
    .map(|value| value * value)
    .collect();
```

### 🔂 Loop Unswitching (Loop-Invariant Branching)

Move conditional `if` statements that do not change during the loop outside of the loop.

```rust
// ❌
// This makes the CPU repeatedly decide between paths in a way that is unnecessary
// or poorly arranged. Modern CPUs try to guess which branch will run next; a
// wrong guess, called a branch misprediction, throws away partially completed
// work.
for value in &data {
    if use_fast { fast(*value) } else { slow(*value) };
}
let data = [1, 2, 3];
let use_fast = true;
fn fast(_:i32){ } fn slow(_:i32){ };


// ✅
// Keep the common case simple and avoid checking conditions inside a loop when
// the answer cannot change. Fewer or more predictable branches give the CPU less
// work to throw away.
if negate {
    for value in &mut values {
        *value = -*value;
    }
} else {
    for value in &mut values {
        *value *= 2;
    }
}
let negate = true;
let mut values = vec![1i32, 2, 3];
```

### 🎰 Branch Prediction Hints (Branch Misprediction on Rare Paths)

For branches where one side is overwhelmingly rare (error handling, panics, one-time setup), mark the rare side `#[cold]` so the compiler optimizes the common path harder.

```rust
// ❌
// This makes the CPU repeatedly decide between paths in a way that is unnecessary
// or poorly arranged. Modern CPUs try to guess which branch will run next; a
// wrong guess, called a branch misprediction, throws away partially completed
// work.
fn parse(value:u8)->Result<u8, ()> { if value==0 { Err(()) } else { Ok(value) } }
let _ = parse(1);


// ✅
// Keep the common case simple and avoid checking conditions inside a loop when
// the answer cannot change. Fewer or more predictable branches give the CPU less
// work to throw away.
#[cold]
fn rare_error() -> ! { panic!("rare failure") }
fn hot(ok: bool) { if !ok { rare_error(); } }
```

### ➗ Strength Reduction (Expensive Ops → Cheap Ops)

Replace expensive arithmetic (division, modulo, multiply by non-constant) with cheaper equivalents the CPU can execute in fewer cycles — shifts, adds, or multiplies by a compile-time inverse.

```rust
// ❌
// This gives the CPU a long chain of dependent work or moves through memory in a
// way that prevents the processor from doing several useful operations at the
// same time.
let bucket = value % 8;
let value = 123u32;


// ✅
// Structure independent calculations and memory access so the CPU can overlap
// work. Terms such as SIMD and instruction-level parallelism mean the processor
// is doing several operations during the same stretch of time instead of waiting
// for one operation to finish before starting the next.
let value = 64u32;
let half = value >> 1;
let times8 = value << 3;
assert_eq!((half, times8), (32, 512));
```

### 🔗 Loop Fusion & Fission

Fuse consecutive loops that touch the same data to cut memory traffic; split (fission) a loop only when it enables better vectorization or cache behavior for distinct phases.

```rust
// ❌
// This gives the CPU a long chain of dependent work or moves through memory in a
// way that prevents the processor from doing several useful operations at the
// same time.
for index in 0..left.len(){ character[index]=left[index]+right[index]; }
for index in 0..left.len(){ d[index]=character[index]*2; }
let left = vec![1; 4];
let right = left.clone();
let mut character = vec![0; 4];
let mut d = character.clone();


// ✅
// Structure independent calculations and memory access so the CPU can overlap
// work. Terms such as SIMD and instruction-level parallelism mean the processor
// is doing several operations during the same stretch of time instead of waiting
// for one operation to finish before starting the next.
for value in &mut values {
    *value += 1;
    *value *= 2;
}
let mut values = vec![1, 2, 3];
```


---

## 🧪 Measurement, Testing & Caution

### 📏 Micro-Benchmarking (Dead-Code Elimination in Benchmarks)

Never trust intuition about which version is faster — measure it with a dedicated benchmark harness, and guard against the compiler "cheating" by optimizing your benchmark away.

```rust
// ❌
// This guesses about performance or measures code in a way the compiler can
// remove or distort. A benchmark is only useful when it represents real work and
// produces a result the compiler must actually compute.
let start = std::time::Instant::now();
let _ = (0..1_000_000u64).sum::<u64>();
println!("{ :? }", start.elapsed());


// ✅
// Measure realistic inputs, prevent the compiler from deleting the work being
// measured, and compare results more than once. Use the measurements to decide
// what to change instead of relying on intuition.
use std::hint::black_box;

fn hot(values: &[u64]) -> u64 { values.iter().copied().sum() }
fn main() { let values = vec![1; 10_000]; black_box(hot(black_box(&values))); }
```


---

## 🔢 Algorithms & Execution Patterns

### 📡 Batching: N+1 Queries & Batch APIs

Never run network/DB requests or heavy per-item API calls inside a loop. Batch into one bulk request, and design your own APIs to accept slices so callers are not forced into N+1 patterns.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
for record_id in record_ids {
    fetch_one(record_id);
}
let record_ids = [1, 2, 3]; fn fetch_one(_:u64){ }


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
fn fetch_many(record_ids: &[u64]) -> Vec<u64> { record_ids.iter().map(|id| id * 10).collect() }
let record_ids = [1, 2, 3, 4];
let rows = fetch_many(&record_ids);
```

### 🖼️ Sliding Windows (Unlocking $O(N)$ Speed)

Never use nested `for` loops if you can solve the problem in a single pass. A sliding window tracks a contiguous subset of data, shifting the boundaries instead of recalculating overlapping segments.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
for start in 0..=values.len()-key {
    let sum:u32=values[start..start+key].iter().sum();
    std::hint::black_box(sum);
}
let values = [1u32, 2, 3, 4];
let key = 2;


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
let values = [1, 2, 3, 4, 5];
let max3 = values.windows(3).map(|w| w.iter().sum::<i32>()).max();
assert_eq!(max3, Some(12));
```

### 🧷 Two Pointers & Fast/Slow Pointers

Turn $O(N^2)$ brute-force searches into $O(N)$ scans by traversing from both ends inward, or use different traversal speeds to detect cycles.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
for index in 0..values.len(){ for other_index in index+1..values.len(){ if values[index]+values[other_index]==target {
    return;
    } };
}
let values = [1i32, 2, 3];
let target = 4;


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
let values = [1, 2, 4, 7, 11];
let (mut l, mut r) = (0, values.len()-1);
while l < r { match (values[l]+values[r]).cmp(&9) { std::cmp::Ordering::Less=>l+=1, std::cmp::Ordering::Greater=>r-=1, _=>break } }
```

### 🌲 DFS vs. BFS: Choosing the Right Graph Traversal

Pick your traversal strategy based on what you're looking for, not habit. Depth-First Search (DFS) and Breadth-First Search (BFS) have the same $O(V + E)$ time complexity, but wildly different memory footprints and behavior depending on the shape of the graph.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
let mut stack = vec![start];
while let Some(values)=stack.pop(){ visit(values); }
let start = 0usize; fn visit(_:usize){ }


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
use std::collections::VecDeque;
let mut q = VecDeque::from([start]);
while let Some(values) = q.pop_front() {
    for &number in &graph[values] { q.push_back(number); }
}
let start = 0usize;
let graph = vec![vec![1], vec![]];
```

### 🔁 Recursion → Iteration (Call-Stack Pressure)

Convert deep or unbounded recursion into an explicit loop (or heap-allocated stack) so you do not blow the call stack and so the compiler can optimize a flat control-flow graph.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
fn walk(number:u64)->u64 { if number==0 { 0 } else { 1+walk(number-1) } }
let _ = walk(100_000);


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
let mut stack = vec![root];
while let Some(node) = stack.pop() {
    stack.extend(children(node));
}
let root = 0usize; fn children(_:usize)->Vec<usize>{ vec![] }
```

### 🧮 Dynamic Programming: Memoization vs. Tabulation

Never compute the exact same sub-problem twice. Cache redundant work!

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
fn fib(number:u64)->u64 { if number<2 {number} else { fib(number-1)+fib(number-2) } }
let _ = fib(40);


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
fn fib(number: usize) -> u64 {
    let mut fibonacci_values = vec![0u64; number.max(2)+1];
    fibonacci_values[1]=1;
    for index in 2..=number { fibonacci_values[index]=fibonacci_values[index-1]+fibonacci_values[index-2]; } fibonacci_values[number]
}
```

### 🧭 Greedy Algorithms & Heuristics

Don't brute-force an exact optimal answer if it takes $O(2^N)$ time and a "good enough" approximation (or greedy choice) takes $O(N \log N)$.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
for mask in 0usize..(1usize<<items.len()) {
    std::hint::black_box(mask);
}
let items = [1, 2, 3, 4];


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
let mut jobs = vec![(3, 4), (1, 2), (2, 3)];
jobs.sort_unstable_by_key(|&(_, end)| end);
```

### 💤 Lazy vs. Eager Evaluation (Unnecessary Upfront Computation)

Don't compute heavy data transformations until the exact moment you actually need the result. Use Lazy Iterators.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
let values:Vec<_>=(0..1_000_000).map(expensive).collect();
let _ = values.first();
fn expensive(value:u64)->u64{ value*value }


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
let first = (0..1_000_000)
    .map(|value| value * value)
    .find(|&value| value > 10_000);
assert!(first.is_some());
```

### 🔌 Short-Circuit Evaluation & Early Exit (Fail-Fast)

Order conditions and checks so the cheapest rejecting work runs first — abort before expensive computation, allocation, parsing, or I/O.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
if expensive_check(&text) && !text.is_empty() { use_it(&text); }
let text = String::new(); fn expensive_check(_: &str)->bool{true} fn use_it(_: &str){ }


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
fn valid(text: &str) -> bool {
    !text.is_empty() && text.len() < 64 && expensive_check(text)
}
fn expensive_check(_: &str)->bool{true}
```

### 🌊 Stream Processing vs. Batch Processing

Choose streaming when latency, memory footprint, or infinite/unknown input size matter; choose batch when throughput, vectorization, and simple control flow matter.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
let bytes = std::fs::read("huge.log").unwrap();
std::hint::black_box(bytes);


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
use std::io::{ self, BufRead };
for line in io::stdin().lock().lines() {
    let line = line.unwrap();
    process(&line);
}
fn process(_: &str){ }
```

### 🗜️ Compression (CPU vs. Bandwidth / Storage Trade-off)

Compress when the cost of CPU cycles to (de)compress is lower than the cost of moving or storing the uncompressed bytes — profile both sides on realistic data.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
let tiny = b"ok";
let _compressed = fake_compress(tiny);
fn fake_compress(value:&[u8])->Vec<u8>{ value.to_vec() }


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
let bytes = b"aaaaaaaaaaaaaaaa";
std::hint::black_box(bytes);
```

### 📊 Sorting Algorithm Selection

Use the standard library sort for almost everything; specialize only when profiling shows sort is hot and your data has exploitable structure (almost-sorted, tiny keys, integers in a narrow range).

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
let mut values = vec![3, 1, 2];
values.sort();


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
let mut values = vec![4, 1, 3, 2];
values.sort_unstable();
```

### 🔍 Binary Search vs. Hash Lookup

Prefer `HashMap`/`HashSet` for unstructured key lookup at scale; prefer sorted `Vec` + binary search when the set is small, ordered iteration matters, or you want simpler memory layout and better cache behavior.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
let found = values.iter().find(|&&value| value==needle);
let values = vec![1u64; 100_000];
let needle = 2;


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
let mut values = vec![9, 1, 7, 3];
values.sort_unstable();
assert!(values.binary_search(&7).is_ok());
```

### 🤖 Deterministic vs. Non-Deterministic Logic (Memoization Eligibility)

Isolate non-deterministic operations (randomness, system time, I/O) from your core logic. Favor pure, deterministic functions.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
fn score(value:u64)->u64 { value + random_value() }
fn random_value()->u64{ 4 }


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
fn score(value: u64) -> u64 { value * 37 + 11 }
let left = score(5);
let right = score(5);
assert_eq!(left, right);
```

### 🔁 Stateless vs. Stateful Design

Prefer stateless functions and components by default; introduce state deliberately, only where it earns its keep.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
static mut TOTAL:u64=0;
unsafe { TOTAL += 1; }


// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
fn normalize(value: f64, min: f64, max: f64) -> f64 { (value-min)/(max-min) }
let other_value = normalize(5.0, 0.0, 10.0);
```


---

## 🐧 Operating Systems, Kernels, Boot & User Space

### 🥾 Boot Loaders & Early Init

Boot path length is pure latency before useful work — minimize firmware/bootloader/kernel init for devices that must start fast (embedded, serverless snapshots, appliances).

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
let huge_index = build_index();
serve(huge_index);
fn build_index()->Vec<u8>{ vec![0;1_000_000] } fn serve(_:Vec<u8>){ }


// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
fn main() {
    serve();
}
fn serve() { }
```

### 🧱 Kernel vs. User Space

Cross the kernel boundary as rarely as practical on hot paths. Every syscall is a mode switch, validation, and potential scheduler decision.

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
use std::io::Write;
for right in b"hello" {
    std::io::stdout().write_all(&[*right]).unwrap();
}

// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
use std::io::{ BufWriter, Write };
let mut output = BufWriter::new(std::io::stdout().lock());
for _ in 0..1000 { writeln!(output, "x").unwrap(); }
```

### ⚡ Traps, Interrupts, Exceptions & Events

Treat asynchronous control-flow transfers as expensive — they flush pipelines, disturb cache/TLB locality, and can preempt critical sections.

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
fn on_event(){ let values:Vec<_>=(0..1_000_000).collect(); drop(values); }
on_event();


// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
use std::sync::atomic::{ AtomicBool, Ordering };
static EVENT: AtomicBool = AtomicBool::new(false);
fn handler_like() { EVENT.store(true, Ordering::Relaxed); }
```

### 🔄 Processes vs. Threads

Threads share an address space (cheap communication, careful sync); processes isolate memory (safer, more overhead to spawn and to IPC).

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
let _ = std::process::Command::new("echo").arg("work").status();


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
let h = std::thread::spawn(|| compute());
let result = h.join().unwrap();
fn compute()->u64{ 42 }
```

### 📡 IPC (Inter-Process Communication)

Pick IPC by bandwidth and latency needs — don’t use JSON-over-TCP between two processes on the same machine if shared memory fits.

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
let message = format!(r#"{ { "value":{ } } }"#, 42);
std::hint::black_box(message);


// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
use std::os::unix::net::UnixStream;
let (_a, _b) = UnixStream::pair().unwrap();
Unix-only;
```

### 🔀 I/O Multiplexing

Never block one thread per connection at scale — multiplex readiness (`epoll`/`kqueue`/`IOCP`) or use async runtimes built on them.

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
for stream in listener.incoming() {
    std::thread::spawn(move || handle(stream.unwrap()));
}
use std::net::TcpListener; let listener = TcpListener::bind("127.0.0.1:0").unwrap(); fn handle(_:std::net::TcpStream){ }


// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
```

### 🔐 Synchronization Across Processes

Process-shared synchronization must use process-shared primitives (`mutex` with shared memory, file locks, semaphores) — thread-only mutexes do not work across address spaces.

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
let lock = std::sync::Mutex::new(0u32);
std::hint::black_box(lock);


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
```


---

## 🧵 Concurrency, Parallelism & Async

### 🧵 Concurrency with Threads (Spawn & Join Overhead)

Prefer a fixed-size thread pool (or work-stealing pool) over spawning a fresh OS thread for every unit of work.

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
for job in jobs {
    std::thread::spawn(move || run(job));
}
let jobs = 0..1000; fn run(_:i32){ }


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
use rayon::prelude::*;
let squares: Vec<_> = (0u64..1000).into_par_iter().map(|value| value*value).collect();
```

### 🔒 Thread Synchronization Strategies (Beyond a Single Mutex)

Match the synchronization primitive to the access pattern — a global `Mutex` is rarely the right default for hot shared state.

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
let state = std::sync::Mutex::new(vec![0u64; 1024]);
std::hint::black_box(state);


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
use std::sync::RwLock;
let state = RwLock::new(vec![1, 2, 3]);
let readers = state.read().unwrap();
```

### ⏳ Synchronous vs. Asynchronous & Event Blocking

Never use synchronous, blocking I/O inside an `async` function.

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
async fn bad(){ std::thread::sleep(std::time::Duration::from_secs(1)); }


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
async fn load() -> std::io::Result<Vec<u8>> {
    tokio::fs::read("data.bin").await
}
```

### 🧑‍💻 Where to Put Async: App vs. Library Internals vs. Library Callers

`async` is usually great in applications, risky inside a library's internal implementation, and excellent when offered to a library's callers.

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
async fn add(left:u64, right:u64)->u64 { left+right }


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
pub async fn fetch_all(record_ids: &[u64]) -> Vec<u64> {
    record_ids.to_vec()
}
```

### 🔒 Lock Contention (The Concurrency Bottleneck)

Avoid wrapping highly-contended shared data in a `Mutex`. Prefer hardware-level Atomics, Lock-Free structures, or Message Passing (Channels).

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
let counter = std::sync::Mutex::new(0u64);
for _ in 0..1000 {
    *counter.lock().unwrap() += 1;
}

// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
use std::sync::atomic::{ AtomicU64, Ordering };
let hits = AtomicU64::new(0);
hits.fetch_add(1, Ordering::Relaxed);
```

### ⚛️ Avoid Needless Atomics (Atomic vs. Non-Atomic Reference Counting)

Use `Rc<T>` instead of `Arc<T>` whenever data never crosses a thread boundary.

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
let value = std::sync::Arc::new(String::from("local"));
let other_value = value.clone();
std::hint::black_box(other_value);


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
use std::rc::Rc;
let data = Rc::new(vec![1, 2, 3]);
let shared = Rc::clone(&data);
```

### 🚧 Cache Line Padding & False Sharing (Multi-Core Write Contention)

When multiple threads are mutating different variables, ensure those variables do not sit on the exact same CPU cache line. Force them apart using memory alignment padding.

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
#[repr(C)] struct Counters { left: std::sync::atomic::AtomicU64, right: std::sync::atomic::AtomicU64 }


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
#[repr(align(64))]
struct Padded<T>(T);
let counters = [Padded(0u64), Padded(0u64)];
```

### 🧩 System-on-Chip Awareness: Heterogeneous Cores (Uneven Core Performance)

On modern SoCs (Apple Silicon, recent Intel laptop/mobile chips, most Android phones), don't assume every core the OS reports is equally fast — spawning `available_parallelism()` identical worker threads can silently bottleneck on the slowest core in the group.

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
let number = std::thread::available_parallelism().unwrap().get();
for _ in 0..number {
    std::thread::spawn(|| heavy());
}
fn heavy(){ }


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
let workers = std::thread::available_parallelism().map_or(1, |number| number.get());
println!("workers<= {workers}");
```

### 🧵 Thread-Local Storage (Avoiding Shared-State Contention)

When each thread can own its own buffer, counter, or PRNG, use thread-local storage instead of a shared `Mutex`-guarded resource.

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
let scratch = std::sync::Mutex::new(Vec::<u8>::with_capacity(4096));
std::hint::black_box(scratch);


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
use std::cell::RefCell;
thread_local! { static BUF: RefCell<Vec<u8>> = const { RefCell::new(Vec::new()) }; }
BUF.with(|right| right.borrow_mut().extend_from_slice(b"hi"));
```

### 📡 Signal Handling (Async-Signal-Safety & Hot-Path Interference)

Keep signal handlers minimal — set a flag or write to a self-pipe — and never do heavy work, allocate, or take locks inside them.

```rust
// ❌
// This can make workers spend time waiting for locks, starting new
// operating-system threads, or fighting over the same memory instead of doing
// useful work. That waiting is called contention.
fn on_signal(){ let _v = vec![0u8;1024]; let _ = std::sync::Mutex::new(0); }
on_signal();


// ✅
// Give workers independent data when possible, reuse a fixed pool of threads, and
// choose synchronization that matches how the data is shared. The goal is to
// reduce waiting while still keeping shared data correct.
use std::sync::atomic::{ AtomicBool, Ordering };
static STOP: AtomicBool = AtomicBool::new(false);
fn signal_handler() { STOP.store(true, Ordering::Relaxed); }
```


---

## 🗄️ Data Structure Selection

### 🗃️ Vectors vs. HashMaps

Default to contiguous memory structures (`Vec`) for small/ordered data, but pivot to `HashMap` when large-scale, repeated lookups are required. Always pre-allocate capacity.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
use std::collections::HashMap;
let map:HashMap<_, _>=(0..8).map(|index|(index, index*index)).collect();


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
use std::collections::HashMap;
let small = vec![(1, "a"), (2, "b")];
let map: HashMap<_, _> = small.iter().copied().collect();
```

### ⛓️ LinkedLists vs. Contiguous Storage (Cache Locality)

❌ AVOID `LinkedList`.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
use std::collections::LinkedList;
let mut values = LinkedList::new();
for index in 0..1000 { values.push_back(index);
};


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
let mut q = std::collections::VecDeque::new();
q.push_back(1); q.push_back(2);
assert_eq!(q.pop_front(), Some(1));
```

### 🕸️ Graph Representation: Pointer Chasing vs. Index-Based Arenas

Don't model graph nodes as `Rc<RefCell<Node>>` with pointer-based edges. Store nodes in a flat `Vec` and represent edges as plain integer indices into that `Vec`.

```rust
// ❌
// This asks the memory allocator for new storage, copies data, or frees storage
// more often than necessary. The allocator is the part of the runtime that finds
// space in memory; calling it repeatedly adds bookkeeping work and can make the
// program pause more often.
use std::{ cell::RefCell, rc::Rc };
let node = Rc::new(RefCell::new(Vec::<Rc<RefCell<Vec<usize>>>>::new()));
std::hint::black_box(node);


// ✅
// Reuse memory that already exists, reserve enough space before filling a
// collection, or borrow existing data when ownership is not needed. This reduces
// allocations and copies while keeping the code safe and predictable.
#[derive(Default)]
struct Node { edges: Vec<usize> }
let graph = vec![Node { edges: vec![1] }, Node::default()];
```

### 📐 Compressed Sparse Row (CSR): The Densest Graph Layout

For large, mostly-static graphs (millions of nodes, edges rarely change), go a step further than `Vec<Vec<usize>>` and flatten all edges into a single contiguous array using the CSR format.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
let edges:Vec<Vec<usize>>=vec![vec![1, 2], vec![2], vec![]];
std::hint::black_box(edges);


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
struct Csr { offsets: Vec<usize>, edges: Vec<usize> }
let g = Csr { offsets: vec![0, 2, 3], edges: vec![1, 2, 0] };
let neighbors0 = &g.edges[g.offsets[0]..g.offsets[1]];
```

### 🌳 Binary Search Trees (BSTs)

If you need to constantly insert data and keep it perfectly sorted, a `Vec` will choke. Use Rust's `BTreeSet` to keep insertions and lookups at $O(\log N)$.

```rust
// ❌
// This repeats work, explores more possibilities than necessary, or keeps
// calculating after the answer is already known. That wastes CPU time and becomes
// much more noticeable when the input gets large.
let mut values = Vec::new();
for value in [5, 1, 4, 2, 3] {
    let index = values.partition_point(|values|values<&value);
    values.insert(index, value);
}

// ✅
// Arrange the code so each useful piece of work is done once whenever possible.
// Reuse previous results, stop early when the answer is known, and choose an
// algorithm whose amount of work grows slowly as the input grows.
use std::collections::BTreeSet;
let mut set = BTreeSet::new();
set.extend([3, 1, 2]);
assert_eq!(set.iter().copied().collect::<Vec<_>>(), vec![1, 2, 3]);
```

### 📚 Stacks

Need Last-In-First-Out (LIFO) behavior? You don't need a custom Node struct. Just use a standard `Vec` with `.push()` and `.pop()`.

```rust
// ❌
// This uses a data structure whose normal operations do more work or touch more
// memory than the job requires. The wrong structure can turn a simple lookup into
// repeated searching.
struct Node<T>{ value:T, next:Option<Box<Node<T>>> }
let _ = Node{ value:1, next:None };


// ✅
// Choose the collection around the operations the program performs most often.
// Contiguous lists are simple and memory-friendly, hash tables are useful for
// repeated key lookup, and specialized structures are worthwhile only when their
// trade-offs match the real workload.
let mut stack = Vec::new();
stack.push(10); stack.push(20);
assert_eq!(stack.pop(), Some(20));
```

### 🎲 Probabilistic Data Structures (Memory-vs-Accuracy Trade-off)

When working with massive datasets where you just need to know if something might exist, do not use a standard Hash Table. Use a Bloom Filter.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
use std::collections::HashSet;
let seen:HashSet<u64>=(0..1_000_000).collect();
std::hint::black_box(seen);


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
```


---

## 🏛️ Data Layout & Memory Footprint

### 🔢 Choosing Data Types

The narrowest type that correctly represents your value range and precision needs is usually the fastest one — every extra byte in a type is an extra byte moved through memory bandwidth and an extra byte competing for cache space.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
let pixels:Vec<u64>=vec![0; 1_000_000];
std::hint::black_box(pixels);


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
struct Pixel { r: u8, g: u8, right: u8, left: u8 }
assert_eq!(std::mem::size_of::<Pixel>(), 4);
```

### 📊 Data-Oriented Design: SoA vs. AoS (Struct Layout & Cache Utilization)

Structure your data for the CPU cache, not for human readability. Group identical properties together (Struct of Arrays) rather than grouping properties by object (Array of Structs).

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
struct Particle{ value:f32, other_value:f32, z:f32, mass:f32 }
let particles = vec![Particle{ value:0.0, other_value:0.0, z:0.0, mass:1.0 }; 1000];
let _ = &ps;


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
struct Particles { value: Vec<f32>, other_value: Vec<f32>, z: Vec<f32> }
let position = Particles { value: vec![0.0; 1000], other_value: vec![0.0; 1000], z: vec![0.0; 1000] };
let sum_x: f32 = position.value.iter().sum();
```

### 🔥 Hot/Cold Data Splitting

Split frequently accessed fields from rarely accessed ones into separate structures (or separate arrays) so hot data stays dense in cache.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
struct Row{ hot:u64, debug_name:String }
let _ = Row{ hot:1, debug_name:"x".repeat(100) };


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
struct Hot { position: [f32;3], velocity: [f32;3] }
struct Cold { name: String, debug: String }
let hot = vec![Hot { position:[0.0; 3], velocity:[0.0; 3] };
1024];
let _cold: Vec<Cold> = vec![];
```

### 📏 Struct Padding, Field Reordering & External Padding

Order fields largest-to-smallest (especially under `#[repr(C)]`) to cut internal padding. Remember trailing (external) padding: `size_of::<T>()` is rounded up to a multiple of alignment, so padding repeats for every array element.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
#[repr(C)] struct Bad { left:u8, right:u64, character:u16 }
println!("{}", std::mem::size_of::<Bad>());


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
#[repr(C)]
struct Better { left: u64, right: u32, character: u8 }
println!("{}", std::mem::size_of::<Better>());
```

### 📦 Data Layout & Enum Boxing (Oversized Enum Variants)

Keep your structs and enums small. If an enum has one massive variant, `Box` it to keep the overall footprint tiny.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
enum Message { Ping, Payload([u8;4096]) }
println!("{}", std::mem::size_of::<Message>());


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
enum Message { Tiny(u8), Huge(Box<[u8; 4096]>) }
println!("{}", std::mem::size_of::<Message>());
```

### 🧮 Bitwise Operations & Bitflags (The Boolean Bloat)

When tracking multiple true/false states, do not use arrays of `bool`s. Pack them tightly into a single integer using bitwise operations.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
struct Flags { left:bool, right:bool, character:bool, d:bool, e:bool, file:bool, g:bool, h:bool }
let _ = std::mem::size_of::<Flags>();


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
const READ: u8 = 1 << 0; const WRITE: u8 = 1 << 1;
let flags = READ | WRITE;
assert!(flags & WRITE != 0);
```

### ◀️ Logical vs. Arithmetic Bit Shifts

Use logical shifts for unsigned data and bit patterns; use arithmetic shifts only when you intentionally want sign extension on signed integers.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
let bits:i32 = -2;
let shifted = bits >> 1;
std::hint::black_box(shifted);


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
let bits: u8 = 0b1000_0000;
assert_eq!(bits >> 1, 0b0100_0000);
let signed: i8 = -8;
assert_eq!(signed >> 1, -4);
```

### 🏷️ String Interning / Flyweight Pattern (Duplicate String Storage)

When dealing with thousands of identical string values (like JSON keys, tags, or categories), do not store them as independent strings. Use String Interning.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
let tags:Vec<String>=(0..10_000).map(|_| "error".to_owned()).collect();
std::hint::black_box(tags);


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
use std::collections::HashMap;
let mut record_ids = HashMap::<String, u32>::new();
let next = record_ids.len() as u32;
let id = *record_ids.entry("status".into()).or_insert(next);
```

### 📄 Zero-Copy Parsing & Clone-On-Write (Unnecessary String Duplication)

When parsing data (like JSON or networking packets), never allocate new `String`s unless you are physically altering the text. Borrow the original buffer using `&str` or `Cow`.

```rust
// ❌
// This asks the memory allocator for new storage, copies data, or frees storage
// more often than necessary. The allocator is the part of the runtime that finds
// space in memory; calling it repeatedly adds bookkeeping work and can make the
// program pause more often.
let line = String::from("alice, 42");
let fields:Vec<String>=line.split(", ").map(str::to_owned).collect();


// ✅
// Reuse memory that already exists, reserve enough space before filling a
// collection, or borrow existing data when ownership is not needed. This reduces
// allocations and copies while keeping the code safe and predictable.
use std::borrow::Cow;
fn normalize(text: &str) -> Cow<'_, str> {
    if text.bytes().all(|right| !right.is_ascii_uppercase()) { Cow::Borrowed(text) } else { Cow::Owned(text.to_lowercase()) }
}
```


---

## 🧠 Memory Management Models

### ♻️ Memory Management Strategies: Manual Allocation vs. Garbage Collection vs. Reference Counting vs. Smart Pointers

Every memory-management strategy trades predictability and raw throughput against safety and programmer effort — know which axis your program actually needs before picking (or fighting) a language's default.

```rust
// ❌
// This asks the memory allocator for new storage, copies data, or frees storage
// more often than necessary. The allocator is the part of the runtime that finds
// space in memory; calling it repeatedly adds bookkeeping work and can make the
// program pause more often.
let value = std::sync::Arc::new(vec![1, 2, 3]);
std::hint::black_box(value);


// ✅
// Reuse memory that already exists, reserve enough space before filling a
// collection, or borrow existing data when ownership is not needed. This reduces
// allocations and copies while keeping the code safe and predictable.
use std::{ rc::Rc, sync::Arc };
let owned = Box::new(42);
let local_shared = Rc::new(42);
let thread_shared = Arc::new(42);
std::hint::black_box((owned, local_shared, thread_shared));
```

### 🌍 Static Variables vs. Global Variables

Minimize both, but understand they solve different problems — a `static` gives a value a fixed memory address and program-length lifetime; "global" describes scope (visible everywhere), which is the more dangerous property of the two.

```rust
// ❌
// This chooses a memory or global-state design without considering who owns the
// data, how long it lives, or how many parts of the program can change it. That
// can add copying, reference counting, synchronization, or hard-to-follow
// dependencies.
static mut CONFIG:u64=0;
unsafe { CONFIG=42; }


// ✅
// Pick the simplest ownership model that matches the real lifetime and sharing
// needs. Keep global mutable state rare, and use Rust's enums, borrowing, and
// smart pointers only where their behavior is actually needed.
static VERSION: &str = "1.0";
fn version() -> &'static str {VERSION}
```

### 🧬 Heterogeneous Data Structures, Unions & Enums

When elements differ in shape, use a sum type (`enum`) with dense packing; reach for `union` only when you need C-compatible overlay or proven space savings and are willing to manage safety.

```rust
// ❌
// This chooses a memory or global-state design without considering who owns the
// data, how long it lives, or how many parts of the program can change it. That
// can add copying, reference counting, synchronization, or hard-to-follow
// dependencies.
union Value { index:i64, file:f64 }
let values = Value{ index:42 };
std::hint::black_box(values);


// ✅
// Pick the simplest ownership model that matches the real lifetime and sharing
// needs. Keep global mutable state rare, and use Rust's enums, borrowing, and
// smart pointers only where their behavior is actually needed.
enum Value { Int(i64), Float(f64), Text(String) }
let values = vec![Value::Int(1), Value::Float(2.5), Value::Text("x".into())];
```

### 🪄 Macros (Codegen vs. Runtime Cost)

Use macros to eliminate repetitive boilerplate and runtime work (generate match arms, lookup tables, parsers) — not to hide heavy runtime logic that should be a plain function.

```rust
// ❌
// This adds work that is easy to overlook, such as an extra function lookup, heap
// allocation, string formatting, panic machinery, or startup work that the
// current request does not need.
macro_rules! expensive { () => { { (0..1_000_000u64).sum::<u64>() } } }
let _ = expensive!();


// ✅
// Keep frequently executed code direct and delay optional work until it is
// actually needed. Use abstractions where they improve the design, but measure
// them before paying extra cost inside a performance-critical loop.
macro_rules! table { ($($key:expr => $values:expr), * $(, )?) => { match key { $($key => $values, )* _ => 0 } }; }
let key = 2;
let value = table!(1 => 10, 2 => 20, 3 => 30);
assert_eq!(value, 20);
```


---

## 🎛️ Abstraction & Dispatch Costs

### 👉 Function Pointers vs. Generics vs. Closures

Prefer generics/`impl Fn` for hot call sites (static dispatch, inlining); use function pointers (`fn(...)`) for thin dynamic callbacks and FFI; avoid `Box<dyn Fn>` in tight loops.

```rust
// ❌
// This adds work that is easy to overlook, such as an extra function lookup, heap
// allocation, string formatting, panic machinery, or startup work that the
// current request does not need.
let file:Box<dyn Fn(u64)->u64>=Box::new(|value|value+1);
for index in 0..1000 {
    std::hint::black_box(file(index));
}

// ✅
// Keep frequently executed code direct and delay optional work until it is
// actually needed. Use abstractions where they improve the design, but measure
// them before paying extra cost inside a performance-critical loop.
fn apply<F: Fn(u32) -> u32>(value: u32, file: F) -> u32 { file(value) }
let other_value = apply(5, |value| value * 2);
```

### 🎛️ Static vs. Dynamic Dispatch (Virtual Method Table Indirection)

Prefer Generics (`impl Trait`) over Trait Objects (`Box<dyn Trait>`) unless you absolutely need a collection of mixed types.

```rust
// ❌
// This adds work that is easy to overlook, such as an extra function lookup, heap
// allocation, string formatting, panic machinery, or startup work that the
// current request does not need.
trait Op{ fn run(&self, value:u64)->u64; }
fn apply(op:&dyn Op, value:u64)->u64{ op.run(value) }


// ✅
// Keep frequently executed code direct and delay optional work until it is
// actually needed. Use abstractions where they improve the design, but measure
// them before paying extra cost inside a performance-critical loop.
trait Area { fn area(&self) -> f64; }
fn sum<T: Area>(values: &[T]) -> f64 { values.iter().map(Area::area).sum() }
```

### 💥 Panic & Exception Costs vs. `Result`

Use `Result`/`Option` for expected failure paths; reserve panics for truly unrecoverable bugs. In hot code, avoid patterns that can panic (bounds checks you could prove, `unwrap` on fallible I/O).

```rust
// ❌
// This adds work that is easy to overlook, such as an extra function lookup, heap
// allocation, string formatting, panic machinery, or startup work that the
// current request does not need.
fn parse(text:&str)->u64 { text.parse().unwrap() }
let _ = parse("123");


// ✅
// Keep frequently executed code direct and delay optional work until it is
// actually needed. Use abstractions where they improve the design, but measure
// them before paying extra cost inside a performance-critical loop.
fn parse_port(text: &str) -> Result<u16, std::num::ParseIntError> { text.parse() }
match parse_port("8080") {
    Ok(port_number) => println!("{port_number}"),
    Err(error) => eprintln!("{error}"),
}
```

### 🧱 Object-Oriented Programming Costs (Inheritance & Virtual Methods)

Treat classical OOP (deep inheritance, virtual methods, heap-allocated objects) as a design tool, not a performance default — each layer of indirection and dynamic dispatch has a measurable cost.

```rust
// ❌
// This adds work that is easy to overlook, such as an extra function lookup, heap
// allocation, string formatting, panic machinery, or startup work that the
// current request does not need.
trait Shape{ fn area(&self)->f64; }
let shapes:Vec<Box<dyn Shape>>=Vec::new();
std::hint::black_box(shapes);


// ✅
// Keep frequently executed code direct and delay optional work until it is
// actually needed. Use abstractions where they improve the design, but measure
// them before paying extra cost inside a performance-critical loop.
enum Shape { Circle(f64), Rect(f64, f64) }
fn area(text: &Shape) -> f64 { match *text { Shape::Circle(r)=>std::f64::consts::PI*r*r, Shape::Rect(w, h)=>w*h } }
```

### 📝 Logging, Tracing & Observability Overhead

Log and trace the minimum needed for operations; never format expensive strings on a disabled level; sample high-volume traces.

```rust
// ❌
// This adds work that is easy to overlook, such as an extra function lookup, heap
// allocation, string formatting, panic machinery, or startup work that the
// current request does not need.
let message = format!("state={ :? }", expensive_state());
std::hint::black_box(message);
fn expensive_state()->Vec<u8>{ vec![0;1000] }


// ✅
// Keep frequently executed code direct and delay optional work until it is
// actually needed. Use abstractions where they improve the design, but measure
// them before paying extra cost inside a performance-critical loop.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
```

### 📥 Loading Code & Data (Startup Path)

Defer work that is not needed to serve the first request — lazy-init heavy modules, load configs on demand, and prefer memory-mapping large read-only assets.

```rust
// ❌
// This adds work that is easy to overlook, such as an extra function lookup, heap
// allocation, string formatting, panic machinery, or startup work that the
// current request does not need.
let config = std::fs::read_to_string("large-config.json").unwrap_or_default();
std::hint::black_box(config);


// ✅
// Keep frequently executed code direct and delay optional work until it is
// actually needed. Use abstractions where they improve the design, but measure
// them before paying extra cost inside a performance-critical loop.
use std::sync::OnceLock;
static CONFIG: OnceLock<String> = OnceLock::new();
fn config() -> &'static str { CONFIG.get_or_init(|| std::fs::read_to_string("config.txt").unwrap_or_default()) }
```

### 🖼️ GUI & Interactive UI Performance

Keep the UI thread free — never do heavy work on the event/render thread; update only dirty regions; target stable frame budgets (e.g. 16 ms for 60 Hz).

```rust
// ❌
// This adds work that is easy to overlook, such as an extra function lookup, heap
// allocation, string formatting, panic machinery, or startup work that the
// current request does not need.
fn on_click(){ let _sum:u64=(0..100_000_000).sum(); }
on_click();


// ✅
// Keep frequently executed code direct and delay optional work until it is
// actually needed. Use abstractions where they improve the design, but measure
// them before paying extra cost inside a performance-critical loop.
let (tx, rx) = std::sync::mpsc::channel();
std::thread::spawn(move || { let result = heavy_work(); tx.send(result).ok(); });
fn heavy_work()->u64{ 42 }
```

### 🔌 Drivers & Peripherals

Talk to devices in bulk, with interrupt coalescing or polling at high rate; avoid round-tripping userspace↔driver per tiny operation.

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
for byte in payload {
    device_write(&[*byte]);
}
let payload = b"hello"; fn device_write(_: &[u8]){ }


// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
fn write_device(dev: &mut impl std::io::Write, buffer: &[u8]) -> std::io::Result<()> {
    dev.write_all(buffer)
}
```


---

## 💾 I/O Optimizations

### 🚿 Buffered I/O (Per-Write Syscall Overhead)

Never issue raw, unbuffered `read`/`write` calls in a loop. Wrap the handle in a `BufReader`/`BufWriter`.

```rust
// ❌
// `write_all` sends every tiny write through the operating system separately.
// Each system call has setup, permission checks, and bookkeeping before the bytes
// are written. Paying that setup cost thousands of times can dominate the real
// work.
use std::fs::File;
use std::io::{self, Write};
use std::path::Path;

fn write_lines_slow(path: &Path, lines: &[String]) -> io::Result<()> {
    let mut file = File::create(path)?;

    for line in lines {
        file.write_all(line.as_bytes())?;
        file.write_all(b"\n")?;
    }

    Ok(())
}

// ✅
// `BufWriter` collects many small writes in memory and sends larger chunks to the
// operating system. The technical term is buffered I/O: the same bytes are
// written,
// but the expensive operating-system boundary is crossed fewer times.
use std::io::BufWriter;

fn write_lines(path: &Path, lines: &[String]) -> io::Result<()> {
    let file = File::create(path)?;
    let mut writer = BufWriter::new(file);

    for line in lines {
        writeln!(writer, "{line}")?;
    }

    writer.flush()
}
```

### ⚙️ Syscall Batching & `io_uring` (Submission Overhead)

When you issue thousands of small I/O operations per second, batch them (or use `io_uring`) so you pay kernel transition cost once per batch instead of once per operation.

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
for chunk in chunks {
    submit_io(chunk);
}
let chunks:Vec<&[u8]>=vec![b"a", b"b"]; fn submit_io(_: &[u8]){ }


// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
fn fetch_many(record_ids: &[u64]) -> Vec<u64> { record_ids.iter().map(|id| id * 10).collect() }
let record_ids = [1, 2, 3, 4];
let rows = fetch_many(&record_ids);
```

### 🔌 Connection Pooling & Socket Options

Reuse expensive remote connections (DB, HTTP, TCP) via a pool; tune socket options only after measuring — wrong options can hurt.

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
for _ in 0..100 {
    let _ = std::net::TcpStream::connect("127.0.0.1:8080");
}

// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
```

### 💽 Memory-Mapped Files (Kernel-to-User Buffer Copying)

For reading massively large files (gigabytes in size), map them directly into virtual memory instead of chunking them through a `BufReader`.

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
let bytes = std::fs::read("huge.bin").unwrap();
std::hint::black_box(bytes);


// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
use memmap2::MmapOptions;
let file = std::fs::File::open("large.bin")?;
let map = unsafe { MmapOptions::new().map(&file)? };
std::hint::black_box(&map[..]);
# Ok::<(), std::io::Error>(())?;
```

### 🌐 Choosing Network Layers for Speed

Use the highest-level protocol that meets your latency/throughput goals — drop down a layer only when profiling shows the upper layer is the bottleneck.

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
let request = format!("POST /rpc HTTP/1.1\r\nContent-Length: { }\r\n\r\n{ }", body.len(), body);
let body = "42";


// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
```

### 📄 Data Formats, Serialization & Endianness

Pick format by read/write pattern — not by popularity. On hot paths prefer compact binary over text; fix endianness explicitly at the boundary; avoid re-serializing the same object repeatedly.

```rust
// ❌
// This crosses into the operating system or communicates with a device, file,
// network, or another process more often than necessary. Each crossing has setup
// and validation work; for system calls, that extra work is called syscall
// overhead.
for _ in 0..1000 {
    let bytes = value.to_le_bytes();
    std::hint::black_box(bytes);
}
let value = 42u64;


// ✅
// Do larger chunks of I/O at once, reuse expensive connections, and avoid
// crossing the operating-system boundary for every tiny operation. The useful
// work stays the same, but the setup cost is paid fewer times.
let number: u32 = 0x12345678;
let wire = number.to_be_bytes();
let round_trip = u32::from_be_bytes(wire);
assert_eq!(number, round_trip);
```


---

## 🖥️ What Makes Newer Computers Faster (And How Code Should Adapt)

> Performance gains across hardware generations are not uniform — modern speedups come from parallelism, memory hierarchy, and specialized units more than from higher single-thread clock speeds. Write code that feeds those strengths.

```rust
let chunks: Vec<u64> = data.chunks(1024).map(|character| character.iter().sum()).collect();
// Process nearby values together so the CPU can reuse data it has already loaded.
// This also makes it easier to split the work across CPU cores or use vector
// instructions, which are CPU instructions that perform the same operation on
// several values at once.
let data=vec![1u64;4096];
```

### 🏷️ Metadata Costs (Indexes, Schemas, Alloc Headers)

Metadata is not free — allocator headers, length fields, vtable pointers, index structures, and schema descriptors consume RAM and cache bandwidth. Keep metadata proportional to the value it provides.

```rust
// ❌
// This makes the CPU fetch more memory than it needs or jump between memory
// locations that are far apart. Main memory is much slower than the CPU, so these
// extra trips can leave the processor waiting; this is often described as poor
// cache locality.
struct Item { value:u8, len:usize, capacity:usize, type_id:u64 }
let _ = std::mem::size_of::<Item>();


// ✅
// Keep frequently used data small and close together in memory, and avoid copying
// data that can be borrowed. This gives the CPU a better chance to find the next
// values in its fast cache instead of waiting on main memory.
use std::mem::size_of;
println!("Vec metadata={ } bytes", size_of::<Vec<u8>>());
println!("slice ref={ } bytes", size_of::<&[u8]>());
```

### 📚 Library Headers (Include Cost, API Surface & Header-Only Libraries)

Treat public headers as part of your build-time performance surface — every include, template, and macro in a widely used header is paid by every translation unit that pulls it in.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
pub fn giant_generic<T:Clone+Default+std::fmt::Debug>(value:T)->T { format!("{ :? }", &value); value.clone() }


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
pub fn add(left: u64, right: u64) -> u64 { left + right }
```


---

## 🔬 Hardware-Aware Optimizations

### 📄 Huge Pages (TLB Pressure)

For multi-gigabyte working sets with random or wide sequential access, consider huge pages (2 MB / 1 GB) to cut TLB misses.

```rust
// ❌
// This gives the CPU a long chain of dependent work or moves through memory in a
// way that prevents the processor from doing several useful operations at the
// same time.
let data = vec![0u8; 4*1024*1024*1024usize.min(1_000_000)];
std::hint::black_box(data);


// ✅
// Structure independent calculations and memory access so the CPU can overlap
// work. Terms such as SIMD and instruction-level parallelism mean the processor
// is doing several operations during the same stretch of time instead of waiting
// for one operation to finish before starting the next.
// This example depends on operating-system-specific behavior, so use the matching
// API for Linux, macOS, or Windows.
let buffer = vec![0u8; 2 * 1024 * 1024];
std::hint::black_box(buffer);
```

### 🔮 Manual Prefetching (Hiding RAM Latency)

When you're about to iterate through memory in a predictable but non-linear pattern (e.g., following a list of indices), hint to the CPU to start loading the next chunk into cache before you actually need it.

```rust
// ❌
// This gives the CPU a long chain of dependent work or moves through memory in a
// way that prevents the processor from doing several useful operations at the
// same time.
for &index in &indices {
    sum += data[index];
}
let data = vec![1u64; 1024];
let indices = vec![3usize, 900, 7];
let mut sum = 0;


// ✅
// Structure independent calculations and memory access so the CPU can overlap
// work. Terms such as SIMD and instruction-level parallelism mean the processor
// is doing several operations during the same stretch of time instead of waiting
// for one operation to finish before starting the next.
#[cfg(target_arch = "x86_64")]
unsafe { use std::arch::x86_64::{ _mm_prefetch, _MM_HINT_T0 }; _mm_prefetch(pointer as *const i8, _MM_HINT_T0); }
let pointer = [0u8; 64].as_ptr();
```

### 🔗 Unified Memory Architecture (CPU-GPU Buffer Duplication)

On SoCs with an integrated GPU (Apple Silicon, most mobile chips, Intel/AMD APUs), don't blindly port desktop-GPU code that copies buffers between "CPU memory" and "GPU memory" — on these chips that memory is often the same physical RAM.

```rust
// ❌
// This optimizes the wrong resource for the hardware or workload. Different
// systems may be limited by memory bandwidth, battery power, latency deadlines,
// network traffic, accelerator usage, or startup time rather than ordinary CPU
// arithmetic.
let gpu_copy = cpu_buffer.clone();
let cpu_buffer = vec![0u8; 1024];
let _ = gpu_copy;


// ✅
// Measure the resource that actually limits this workload, then organize the
// program around that limit. Keep data movement small, batch work where possible,
// and use specialized hardware or algorithms only when they solve the measured
// problem.
fn process_shared(buffer: &[f32]) { std::hint::black_box(buffer); }
let buffer = vec![0.0f32; 1024];
process_shared(&buffer);
```

### 🏛️ RISC vs. CISC (Instruction Set Architecture Trade-offs)

Understand that x86/x86-64 (CISC) and ARM/RISC-V (RISC) execute your Rust code through fundamentally different instruction pipelines — write portable, branch-light idioms that compile well on both, and gate any architecture-specific intrinsics behind `cfg(target_arch)` instead of hardcoding one ISA.

```rust
// ❌
// This gives the CPU a long chain of dependent work or moves through memory in a
// way that prevents the processor from doing several useful operations at the
// same time.
#[cfg(any())] unsafe { std::arch::x86_64::_mm_pause(); }


// ✅
// Structure independent calculations and memory access so the CPU can overlap
// work. Terms such as SIMD and instruction-level parallelism mean the processor
// is doing several operations during the same stretch of time instead of waiting
// for one operation to finish before starting the next.
#[cfg(target_arch = "x86_64")] fn arch_name() -> &'static str { "x86_64" }
#[cfg(target_arch = "aarch64")] fn arch_name() -> &'static str { "aarch64" }
println!("{}", arch_name());
```

### 🔐 Fast Hashing Algorithms (Cryptographic Hashing Overhead)

Unless your `HashMap` is taking un-sanitized input directly from the internet, replace Rust's default hasher to instantly double your lookup speeds.

```rust
// ❌
// This optimizes the wrong resource for the hardware or workload. Different
// systems may be limited by memory bandwidth, battery power, latency deadlines,
// network traffic, accelerator usage, or startup time rather than ordinary CPU
// arithmetic.
let bytes = b"local-key";
let _ = expensive_crypto_hash(bytes);
fn expensive_crypto_hash(_: &[u8])->[u8;32]{ [0;32] }


// ✅
// Measure the resource that actually limits this workload, then organize the
// program around that limit. Keep data movement small, batch work where possible,
// and use specialized hardware or algorithms only when they solve the measured
// problem.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
use rustc_hash::FxHashMap;
let mut map = FxHashMap::default();
map.insert(1u64, "hot");
```

### 🎲 Faster RNG (CSPRNG Overhead)

If you don't need cryptographic security (e.g., simulations, games, sampling), swap a CSPRNG like `rand`'s default for a fast PRNG like `SmallRng`/`fastrand`.

```rust
// ❌
// This optimizes the wrong resource for the hardware or workload. Different
// systems may be limited by memory bandwidth, battery power, latency deadlines,
// network traffic, accelerator usage, or startup time rather than ordinary CPU
// arithmetic.
let bytes = fake_csprng(1_000_000);
fn fake_csprng(number:usize)->Vec<u8>{ vec![4;number] } let _ = bytes;


// ✅
// Measure the resource that actually limits this workload, then organize the
// program around that limit. Keep data movement small, batch work where possible,
// and use specialized hardware or algorithms only when they solve the measured
// problem.
// This example uses an external Rust crate: a reusable library that your project
// adds as a dependency in `Cargo.toml`.
use rand::{ Rng, SeedableRng };
use rand::rngs::SmallRng;
let mut rng = SmallRng::seed_from_u64(42);
let value: u32 = rng.gen();
```


---

## 🎛️ Domain-Specific: Audio, Video, VR, Mobile & Quantum

### 🔊 Audio (Real-Time Callbacks)

Audio callbacks run under hard deadlines (e.g. every few ms). Never allocate, lock, block on I/O, or take unbounded time on the audio thread.

```rust
// ❌
// This optimizes the wrong resource for the hardware or workload. Different
// systems may be limited by memory bandwidth, battery power, latency deadlines,
// network traffic, accelerator usage, or startup time rather than ordinary CPU
// arithmetic.
fn audio_callback(input:&[f32])->Vec<f32>{ input.iter().map(|value|value*0.5).collect() }
let _ = audio_callback(&[0.0; 64]);


// ✅
// Measure the resource that actually limits this workload, then organize the
// program around that limit. Keep data movement small, batch work where possible,
// and use specialized hardware or algorithms only when they solve the measured
// problem.
fn audio_callback(input: &[f32], output: &mut [f32]) {
    for (o, &index) in output.iter_mut().zip(input) { *o = index * 0.5; }
}
```

### 🎬 Video (Throughput + Latency)

Move pixels as little as possible; use hardware codecs/display paths; pipeline stages so decode, process, and present overlap.

```rust
// ❌
// This optimizes the wrong resource for the hardware or workload. Different
// systems may be limited by memory bandwidth, battery power, latency deadlines,
// network traffic, accelerator usage, or startup time rather than ordinary CPU
// arithmetic.
let frame = decode();
let frame = process(frame);
present(frame);
fn decode()->Vec<u8>{ vec![] } fn process(value:Vec<u8>)->Vec<u8>{value} fn present(_:Vec<u8>){ }


// ✅
// Measure the resource that actually limits this workload, then organize the
// program around that limit. Keep data movement small, batch work where possible,
// and use specialized hardware or algorithms only when they solve the measured
// problem.
fn grayscale_in_place(frame: &mut [u8]) {
    for px in frame.chunks_exact_mut(4) { let other_value = ((px[0] as u16+px[1] as u16+px[2] as u16)/3) as u8; px[0]=other_value; px[1]=other_value; px[2]=other_value; }
}
```

### 🥽 Virtual Reality (Motion-to-Photon)

Missed frame deadlines cause discomfort. Budget time strictly; prioritize pose prediction and late-latching over visual luxury.

```rust
// ❌
// This optimizes the wrong resource for the hardware or workload. Different
// systems may be limited by memory bandwidth, battery power, latency deadlines,
// network traffic, accelerator usage, or startup time rather than ordinary CPU
// arithmetic.
fn frame(){ expensive_render(); update_pose(); }
fn expensive_render(){ } fn update_pose(){ }


// ✅
// Measure the resource that actually limits this workload, then organize the
// program around that limit. Keep data movement small, batch work where possible,
// and use specialized hardware or algorithms only when they solve the measured
// problem.
use std::time::{ Duration, Instant };
let budget = Duration::from_millis(11);
let start = Instant::now();
render_frame();
assert!(start.elapsed() <= budget);
fn render_frame(){ }
```

### 📱 Mobile Devices

Optimize for battery, thermals, and intermittent connectivity — not just peak benchmarks on a plugged-in flagship.

```rust
// ❌
// This optimizes the wrong resource for the hardware or workload. Different
// systems may be limited by memory bandwidth, battery power, latency deadlines,
// network traffic, accelerator usage, or startup time rather than ordinary CPU
// arithmetic.
loop { if background_done() { break; } }
fn background_done()->bool{true}


// ✅
// Measure the resource that actually limits this workload, then organize the
// program around that limit. Keep data movement small, batch work where possible,
// and use specialized hardware or algorithms only when they solve the measured
// problem.
fn maybe_refresh(battery_saver: bool) {
    if battery_saver { return; }
    refresh_expensive_cache();
}
fn refresh_expensive_cache(){ }
```

### 📶 Bluetooth & BLE Optimization

Bluetooth performance is mostly about radio on-air time, connection parameters, and payload packing — not CPU micro-opts. Every extra advertisement, reconnect, or tiny packet costs energy and latency.

```rust
// ❌
// This optimizes the wrong resource for the hardware or workload. Different
// systems may be limited by memory bandwidth, battery power, latency deadlines,
// network traffic, accelerator usage, or startup time rather than ordinary CPU
// arithmetic.
for right in payload {
    send_packet(&[*right]);
}
let payload = b"hello"; fn send_packet(_: &[u8]){ }


// ✅
// Measure the resource that actually limits this workload, then organize the
// program around that limit. Keep data movement small, batch work where possible,
// and use specialized hardware or algorithms only when they solve the measured
// problem.
let sensor = [23u8, 50, 99, 7];
let packet = sensor;
send_ble(&packet);
fn send_ble(_: &[u8]){ }
```

### ⚛️ Quantum Computing (Hybrid Classical–Quantum)

Today’s quantum programs are hybrid — classical host code optimizes circuit submission, shot count, and post-processing; quantum speedup is lost if the classical pipeline is wasteful.

```rust
// ❌
// This optimizes the wrong resource for the hardware or workload. Different
// systems may be limited by memory bandwidth, battery power, latency deadlines,
// network traffic, accelerator usage, or startup time rather than ordinary CPU
// arithmetic.
for shot in 0..1000 {
    submit_quantum_job(shot);
}
fn submit_quantum_job(_:u32){ }


// ✅
// Measure the resource that actually limits this workload, then organize the
// program around that limit. Keep data movement small, batch work where possible,
// and use specialized hardware or algorithms only when they solve the measured
// problem.
let circuits = build_circuits();
let batched = submit_batch(&circuits);
let result = postprocess(batched);
fn build_circuits()->Vec<u8>{ vec![1] } fn submit_batch(value:&[u8])->Vec<u8>{ value.to_vec() } fn postprocess(value:Vec<u8>)->Vec<u8>{value}
```


---

## 🤖 Optimizing Code for AI / ML Hardware

> ML workloads are usually memory-bandwidth bound or tensor-core bound, not classic scalar-CPU bound. Optimize data movement, layout, batching, and precision before micro-tuning host-side loops.

```rust
fn fetch_many(ids: &[u64]) -> Vec<u64> { ids.iter().map(|id| id * 10).collect() }
let ids = [1,2,3,4];
let rows = fetch_many(&ids); // one batch API call shape
```


---

## 📟 Optimizing for Embedded, IoT & Constrained Devices

> On small devices the scarce resources are RAM, flash, power, and often a single slow core — optimize for size and predictability first; “throughput at all costs” techniques from servers can make things worse.

```rust
#![allow(dead_code)]
const BUF_SIZE: usize = 128;
fn process(input: &[u8], scratch: &mut [u8; BUF_SIZE]) { let number=input.len().min(BUF_SIZE); scratch[..number].copy_from_slice(&input[..number]); } // fixed memory
```


---

## 🧰 Virtualization, Emulation & Sandboxing Costs

> Each layer of virtualization or emulation multiplies overhead — avoid nested abstraction on hot paths; give VMs/containers clear CPU/memory topology; prefer paravirtualized or hardware-accelerated I/O.

```rust
fn hot_path(data: &[u8]) -> usize { data.iter().filter(|&&right| right != 0).count() }
// This code runs often, so every extra conversion or trip through another process
// is paid repeatedly. Serialization means turning data into bytes or text for
// transport, and IPC means communication between processes. Avoid those extra
// steps here unless the design truly needs them.
```


---

## ☁️ Cloud & Multi-Tenant Optimizations

> In the cloud you optimize for cost, tail latency, and noisy neighbors — not just raw single-machine throughput. Design for horizontal scale, fast startup, and efficient idle.

```rust
fn handle(req: Request) -> Response {
    if let Some(hit) = CACHE.get(&req.key) { return hit.clone(); }
    compute(req)
}
struct Request{key:u64} #[derive(Clone)] struct Response; struct C; static CACHE:C=C; impl C{fn get(&self,_:&u64)->Option<&'static Response>{None}} fn compute(_:Request)->Response{Response}
```


---

## 🔧 LLVM, Compilers & Writing Programming Languages

### 🧬 Using LLVM Effectively

LLVM optimizes what it can see and prove — feed it clear IR, stable types, and whole-program visibility when performance matters.

```rust
// ❌
// This leaves work for runtime or prevents the compiler and linker from seeing
// enough information to simplify the final program. It can also make the
// executable unnecessarily large, which hurts startup time and instruction-cache
// use.
fn compute(values:&[Box<dyn Iterator<Item=u64>>])->u64 { values.len() as u64 }


// ✅
// Let the compiler do constant work ahead of time, build in release mode, and
// enable whole-program optimization only where it helps. The compiler can then
// remove dead work, inline small hot functions, and generate instructions for the
// actual target CPU.
#[inline]
fn square(value: f32) -> f32 { value * value }
fn sum_squares(values: &[f32]) -> f32 { values.iter().copied().map(square).sum() }
```

### 🛠️ Writing Your Own Language (Performance-Relevant Choices)

Language design is performance design — value representation, evaluation strategy, and memory model set the ceiling before any optimizer runs.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
enum Value { Int(Box<i64>), Bool(Box<bool>) }
let _ = Value::Int(Box::new(42));


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
#[repr(u8)]
enum Op { Add, Sub, Mul, Div }
struct Instr { op: Op, left: u16, right: u16 }
```


---

## 🧠 Writing & Serving LLM Systems

> LLM performance is dominated by memory bandwidth, KV cache management, batching, and kernel quality — not by clever scalar micro-opts in Python glue code. Apply the general accelerator rules in Optimizing Code for AI / ML Hardware (residency, fusion, precision, prefetch); below is what is specific to LLMs.

```rust
fn fetch_many(ids: &[u64]) -> Vec<u64> { ids.iter().map(|id| id * 10).collect() }
let ids = [1,2,3,4];
let rows = fetch_many(&ids); // one batch API call shape
```


---

## 🏗️ Compiler, Build & Linking

### ⏱️ Compile-Time Evaluation (Runtime Computation of Constants)

If a value is known at compile time, never calculate it during program execution. Use `const fn`.

```rust
// ❌
// This leaves work for runtime or prevents the compiler and linker from seeing
// enough information to simplify the final program. It can also make the
// executable unnecessarily large, which hurts startup time and instruction-cache
// use.
fn table_size()->usize { (1..=16).product::<usize>() }
let _ = table_size();


// ✅
// Let the compiler do constant work ahead of time, build in release mode, and
// enable whole-program optimization only where it helps. The compiler can then
// remove dead work, inline small hot functions, and generate instructions for the
// actual target CPU.
const fn kib(number: usize) -> usize { number * 1024 }
const BUFFER: usize = kib(64);
let buffer = [0u8; BUFFER];
```

### 📞 Calling Conventions (Register Args vs. Stack Traffic)

Keep hot-path functions compatible with the platform ABI's register-argument limit so arguments stay in registers instead of spilling to the stack; avoid unnecessary large-by-value passes.

```rust
// ❌
// This leaves work for runtime or prevents the compiler and linker from seeing
// enough information to simplify the final program. It can also make the
// executable unnecessarily large, which hurts startup time and instruction-cache
// use.
#[derive(Clone)] struct Big([u64;64]);
fn hot(value:Big)->u64{ value.0[0] }


// ✅
// Let the compiler do constant work ahead of time, build in release mode, and
// enable whole-program optimization only where it helps. The compiler can then
// remove dead work, inline small hot functions, and generate instructions for the
// actual target CPU.
#[inline]
fn dot4(left: f32, right: f32, character: f32, d: f32) -> f32 { left*right + character*d }
let value = dot4(1.0, 2.0, 3.0, 4.0);
```

### 📋 Explicit Inlining (Cross-Crate Inlining Limits)

Use `#[inline]` for tiny, ultra-hot-path functions that are called thousands of times inside loops, especially across crate boundaries.

```rust
// ❌
// This leaves work for runtime or prevents the compiler and linker from seeing
// enough information to simplify the final program. It can also make the
// executable unnecessarily large, which hurts startup time and instruction-cache
// use.
pub fn add1(value:u64)->u64{ value+1 }


// ✅
// Let the compiler do constant work ahead of time, build in release mode, and
// enable whole-program optimization only where it helps. The compiler can then
// remove dead work, inline small hot functions, and generate instructions for the
// actual target CPU.
#[inline]
fn clamp01(value: f32) -> f32 { value.clamp(0.0, 1.0) }
```

### 🧊 Instruction Cache Pressure (I-Cache Bloat from Over-Inlining)

Inlining and monomorphization aren't free — don't blanket `#[inline(always)]` everything or lean on huge generic functions instantiated for dozens of types, or you'll blow out the instruction cache (I-cache) and make things slower.

```rust
// ❌
// This gives the CPU a long chain of dependent work or moves through memory in a
// way that prevents the processor from doing several useful operations at the
// same time.
#[inline(always)] fn huge(value:u64)->u64 { (0..1000).fold(value, |left, right|left.wrapping_add(right)) }


// ✅
// Structure independent calculations and memory access so the CPU can overlap
// work. Terms such as SIMD and instruction-level parallelism mean the processor
// is doing several operations during the same stretch of time instead of waiting
// for one operation to finish before starting the next.
#[inline(never)]
fn rare_big_slow_path(value: u64) -> u64 { (0..100).fold(value, |left, right| left.wrapping_mul(31).wrapping_add(right)) }
```

### 🐇 Release Mode (Debug Build Overhead)

Never benchmark or ship `cargo build` (debug mode). Always use `cargo build --release`.

```rust
// ❌
// This leaves work for runtime or prevents the compiler and linker from seeing
// enough information to simplify the final program. It can also make the
// executable unnecessarily large, which hurts startup time and instruction-cache
// use.
fn main(){ let sum:u64=(0..1_000_000).sum(); println!("{sum}"); }


// ✅
// Let the compiler do constant work ahead of time, build in release mode, and
// enable whole-program optimization only where it helps. The compiler can then
// remove dead work, inline small hot functions, and generate instructions for the
// actual target CPU.
fn main() { println!("optimized build"); }
```

### 🎯 Target Native CPU (Lowest-Common-Denominator Codegen)

If you control the deployment machine, compile for its exact CPU instead of a generic baseline.

```rust
// ❌
// This leaves work for runtime or prevents the compiler and linker from seeing
// enough information to simplify the final program. It can also make the
// executable unnecessarily large, which hurts startup time and instruction-cache
// use.
fn dot(left:&[f32], right:&[f32])->f32{ left.iter().zip(right).map(|(value, other_value)|value*other_value).sum() }
let _ = dot(&[1.0], [2.0]);


// ✅
// Let the compiler do constant work ahead of time, build in release mode, and
// enable whole-program optimization only where it helps. The compiler can then
// remove dead work, inline small hot functions, and generate instructions for the
// actual target CPU.
fn main() { }
```

### 🪶 Binary Size Reduction (Binary Size vs. Speed Trade-off)

If startup time, download size, or embedded flash space matters more than raw runtime speed, tune the release profile to shrink the binary instead of maximizing throughput.

```rust
// ❌
// This leaves work for runtime or prevents the compiler and linker from seeing
// enough information to simplify the final program. It can also make the
// executable unnecessarily large, which hurts startup time and instruction-cache
// use.
#[inline(always)] fn tiny(value:u64)->u64{ value+1 }
let _ = tiny(1);


// ✅
// Let the compiler do constant work ahead of time, build in release mode, and
// enable whole-program optimization only where it helps. The compiler can then
// remove dead work, inline small hot functions, and generate instructions for the
// actual target CPU.
```

### 🔬 Inspecting Machine Code (Verifying Generated Assembly)

For small, extremely hot functions, don't guess whether an optimization "worked" — look at the generated assembly directly.

```rust
// ❌
// This guesses about performance or measures code in a way the compiler can
// remove or distort. A benchmark is only useful when it represents real work and
// produces a result the compiler must actually compute.
fn sum(values:&[u64])->u64{ values.iter().sum() }
let _ = sum(&[1, 2, 3]);


// ✅
// Measure realistic inputs, prevent the compiler from deleting the work being
// measured, and compare results more than once. Use the measurements to decide
// what to change instead of relying on intuition.
#[no_mangle]
pub extern "C" fn hot_add(left: u64, right: u64) -> u64 { left + right }
```

### 🔗 Link-Time Optimization (Cross-Crate Optimization Boundaries)

Don't just rely on `cargo build --release`. Enable LTO in your `Cargo.toml` for production builds.

```rust
// ❌
// This leaves work for runtime or prevents the compiler and linker from seeing
// enough information to simplify the final program. It can also make the
// executable unnecessarily large, which hurts startup time and instruction-cache
// use.
pub fn hot(value:u64)->u64{ value.wrapping_mul(3) }
let _ = hot(2);


// ✅
// Let the compiler do constant work ahead of time, build in release mode, and
// enable whole-program optimization only where it helps. The compiler can then
// remove dead work, inline small hot functions, and generate instructions for the
// actual target CPU.
```

### 📈 Profile-Guided Optimization (Static Heuristics vs. Measured Profiles)

For your most performance-critical binaries, go beyond static analysis — run the program on real workloads first, then feed that data back into a second, smarter compilation pass.

```rust
// ❌
// This guesses about performance or measures code in a way the compiler can
// remove or distort. A benchmark is only useful when it represents real work and
// produces a result the compiler must actually compute.
fn branch(value:u64)->u64{ if value==0 { 1 }else{ value*2 } }
let _ = branch(3);


// ✅
// Measure realistic inputs, prevent the compiler from deleting the work being
// measured, and compare results more than once. Use the measurements to decide
// what to change instead of relying on intuition.
fn main() { representative_workload(); }
fn representative_workload(){ }
```

### 🔗 Symbol Visibility & Dead-Code Stripping

Alongside LTO and static/dynamic linking choices, export only the symbols you must — every public/`#[no_mangle]` symbol is a root the linker must keep.

```rust
// ❌
// This leaves work for runtime or prevents the compiler and linker from seeing
// enough information to simplify the final program. It can also make the
// executable unnecessarily large, which hurts startup time and instruction-cache
// use.
#[unsafe(no_mangle)] pub extern "C" fn internal_helper(value:u64)->u64{ value+1 }


// ✅
// Let the compiler do constant work ahead of time, build in release mode, and
// enable whole-program optimization only where it helps. The compiler can then
// remove dead work, inline small hot functions, and generate instructions for the
// actual target CPU.
pub(crate) fn internal_hot_path() { }
#[no_mangle]
pub extern "C" fn api_entry() { }
```

### 🔗 Dynamic Link Libraries (DLLs) vs. Static Linking

Static-link for the fastest, most predictable single binary; dynamic-link (DLLs on Windows, `.so` shared objects on Linux, `.dylib` on macOS) when you need to share code across multiple processes or patch a dependency without recompiling everything that uses it.

```rust
// ❌
// This leaves work for runtime or prevents the compiler and linker from seeing
// enough information to simplify the final program. It can also make the
// executable unnecessarily large, which hurts startup time and instruction-cache
// use.
#[no_mangle] pub extern "C" fn tiny_api()->u32{ 42 }


// ✅
// Let the compiler do constant work ahead of time, build in release mode, and
// enable whole-program optimization only where it helps. The compiler can then
// remove dead work, inline small hot functions, and generate instructions for the
// actual target CPU.
#[no_mangle]
pub extern "C" fn plugin_api(value: u32) -> u32 { value + 1 }
```


---

## 🌐 General Performance Principles (Language-Agnostic)

### 🐍 Interpreted vs. Compiled Execution

Understand which execution model your language uses, because it determines where your performance ceiling is and which of the optimizations in this document even apply.

```rust
// ❌
// This version performs extra work or uses a more expensive approach without
// showing that the extra cost is necessary. That can waste processor time,
// memory, or operating-system work as the program grows.
enum Op{ Add(u64), Mul(u64) }
fn run(ops:&[Op], mut value:u64)->u64{ for op in ops{ value=match op{ Op::Add(values)=>value+values, Op::Mul(values)=>value*values } }value }


// ✅
// This version keeps the same result while removing unnecessary work. Measure the
// real program afterward to confirm the change actually improves the resource you
// care about.
fn main() {
    let values: Vec<u64> = (0..1_000_000).collect();
    println!("{}", values.iter().sum::<u64>());
}
```

---

## 🧭 Optimization Order of Operations

1. **Measure first** — profile CPU, memory, I/O, and latency.
2. **Fix the algorithm** — complexity improvements usually beat micro-optimizations.
3. **Reduce unnecessary work** — avoid repeated computation, allocation, copying, and syscalls.
4. **Improve memory behavior** — favor contiguous data and cache-friendly layouts.
5. **Tune concurrency and I/O** — reduce contention, blocking, and excessive kernel transitions.
6. **Only then tune low-level details** — SIMD, unchecked access, branch hints, allocator swaps, and architecture-specific tricks.
