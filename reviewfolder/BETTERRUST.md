<div align="center">

# 🦀 Rust Notes

### Ownership · Memory · Traits · Lifetimes · Iterators · Concurrency · Async · API Design · Style

**A practical Rust study guide and engineering reference.**

![Rust](https://img.shields.io/badge/Rust-Systems%20Programming-orange?logo=rust)
![Cargo](https://img.shields.io/badge/Cargo-Tooling-blue)
![Safety](https://img.shields.io/badge/Focus-Memory%20Safety-success)
![Async](https://img.shields.io/badge/Async-Futures-purple)

</div>

---

> [!IMPORTANT]
> ## 📌 Preservation Rule
>
> **All original notes are preserved.**
>
> Formatting, examples, diagrams, comments, and Microsoft Pragmatic Rust Guideline additions are layered around the original material rather than replacing it.

> [!TIP]
> 🏢 **Microsoft-specific additions** are placed under the original section they relate to and usually live inside collapsible sections so your notes remain the primary reading path.

---

# 🧭 Quick Navigation

| Area | Sections |
| --- | --- |
| 🧠 **Fundamentals** | [Ownership](#-ownership-and-borrowing) · [Memory](#-memory-segments) · [Stack](#-stack-and-function-calls) · [Fat Pointers](#-fat-pointers) |
| 🧩 **Type System** | [Traits](#-traits-and-generics) · [Lifetimes](#-lifetimes) · [Enums](#-enums) · [Conversions](#-conversion-traits) |
| ❗ **Reliability** | [Errors](#-error-handling) · [Testing](#-testing) · [Thread Safety](#-thread-safety) |
| 🔁 **Collections** | [Strings](#-string-methods) · [Vec](#-vec-methods) · [Iterators](#-iterators-consumers) |
| ⚡ **Async** | [Async Definitions](#-async-definitions) · [Futures](#-futures-and-async-in-rust) · [Streams](#-streams) |
| 🏗️ **API Design** | [Sealed Traits](#-sealed-traits) · [Newtypes](#-newtypes-encapsulate-implementation-details) · [API Guidelines](#-concepts-to-know) |
| 🎨 **Style** | [Formatting](#-formatting) · [Macros](#-macros) · [Comments](#-comments) · [Doc Comments](#-doc-comments) |
| 🗂️ **Organization** | [Items](#-items) · [Modules](#-modules) · [Crates](#-crates) · [System](#-system) |

---

# 📚 Rust Notes

## 🔗 Resources

| Resource | Focus |
| --- | --- |
| [The Rust Style Guide](https://doc.rust-lang.org/nightly/style-guide/index.html) | Official Rust formatting conventions |
| [API Guidelines](https://rust-lang.github.io/api-guidelines/checklist.html) | Public API design |
| [Effective Rust](https://www.lurklurk.org/effective-rust/) | Idiomatic Rust |
| [Building Linked List](https://rust-unofficial.github.io/too-many-lists/index.html) | Ownership, pointers, unsafe Rust |
| [Design Patterns](https://rust-unofficial.github.io/patterns/intro.html) | Patterns and idioms |
| [Breakdown Rust Notes](https://www.breakdown-notes.com/make/load/rust_cs_canvas) | Additional Rust notes |
| [cheats.rs](https://cheats.rs/) | Rust cheat sheet |
| [Blessed.rs](https://blessed.rs/crates) | Recommended ecosystem crates |
| [High Assurance](https://highassurance.rs/chp4/sw_stack_1.html) | Secure systems programming |

<details>
<summary><strong>🏢 Microsoft Pragmatic Rust Guidelines</strong></summary>

- [Microsoft Pragmatic Rust Guidelines](https://microsoft.github.io/rust-guidelines/)
- [Microsoft Pragmatic Rust Guidelines — Complete Rules](https://microsoft.github.io/rust-guidelines/agents/all.txt)

</details>

---

# 🧠 Rust Ownership and Memory Management

## 🔐 Ownership and Borrowing

| Concept | Notes |
| --- | --- |
| **CRUD Operations** | The owner of an item can create, read, update, and drop it. |
| **Borrowing** | Borrowing does not have ownership, so the value won't be dropped when the borrow goes out of scope. |
| **Mutable Borrowing** | If you mutable borrow a value, the owner can't change it until you're done borrowing it. |
| **Ownership Movement** | Only the item's owner can move the item. |
| **References** | There can be any number of immutable references to the item, or a single mutable reference to the item, but not both. |
| **Data Validity** | The data pointed to must be valid, and any assignments to this location must be of valid data, enforced by the reference to have existing data at all times. |
| **Mutable References** | It's valid to read from a mutable reference and write to it. The ability to do both at once is provided by `std::mem::replace`. |

### 🧠 Ownership Mental Model

```text
                         VALUE
                           │
                 ┌─────────┴─────────┐
                 │                   │
               OWN                 BORROW
                 │                   │
            move / drop       ┌──────┴──────┐
                              │             │
                             &T           &mut T
                              │             │
                            read        read/write
```

### 📖 Immutable Borrow

```rust
fn print_length(value: &String) {
    // `value` is borrowed.
    // This function can read the String but does not own it.
    println!("{}", value.len());
}

fn main() {
    // `value` owns the String.
    let value = String::from("Rust");

    // Borrow the String instead of transferring ownership.
    print_length(&value);

    // Ownership never moved, so `value` remains usable.
    println!("{value}");
}
```

### ✏️ Mutable Borrow

```rust
fn append_language(value: &mut String) {
    // A mutable reference allows us to modify the borrowed value.
    value.push_str(" language");
}

fn main() {
    // The binding itself must be mutable.
    let mut value = String::from("Rust");

    // Grant temporary exclusive mutable access.
    append_language(&mut value);

    // The mutable borrow has ended.
    println!("{value}");
}
```

### 🚦 Borrowing Rule

```text
✅ Many readers:

&T
&T
&T

OR

✅ One writer:

&mut T

BUT NOT BOTH AT THE SAME TIME.
```

> [!TIP]
> A useful mental model is:
>
> **many readers OR one writer**.

### 🔄 `std::mem::replace`

```rust
fn main() {
    // Create a mutable owned String.
    let mut value = String::from("old");

    // Replace the existing value while returning the old value.
    let previous = std::mem::replace(
        &mut value,
        String::from("new"),
    );

    // The previous String is now independently owned.
    assert_eq!(previous, "old");

    // `value` contains the replacement.
    assert_eq!(value, "new");
}
```

---

## 🧠 Memory Segments

- **Memory Segments**: While a program is running, memory is divided into different chunks (segments). Some are fixed size (program code, global data), while others (heap and stack) change size as the program runs.
- **Heap and Stack**: Typically arranged at opposite ends of the program's virtual memory space, so one can grow downwards and the other upwards.

### 🗺️ Simplified Process Memory

```text
Higher Addresses

┌──────────────────────────────┐
│            Stack             │
│              ↓               │
├──────────────────────────────┤
│                              │
│          Free Space          │
│                              │
├──────────────────────────────┤
│              ↑               │
│             Heap             │
├──────────────────────────────┤
│        Global / Static       │
├──────────────────────────────┤
│         Program Code         │
└──────────────────────────────┘

Lower Addresses
```

> [!NOTE]
> This is a conceptual memory layout. Exact process layouts depend on the OS, architecture, linker, and runtime environment.

---

## 📚 Stack and Function Calls

- **Stack Usage**: The stack holds state related to the currently executing function, specifically its parameters, local variables, and temporary values.
- **Stack Frames**: When a function `f()` is called, a new stack frame is added. When `f()` returns, the stack pointer resets to the caller's stack frame.
- **Memory Reuse**: When a different function `g()` is called, the stack frame for `g()` reuses the same area of memory that `f()` previously used.

```rust
fn f() {
    // This local variable belongs to `f`'s stack frame.
    let x = 10;

    println!("{x}");
}

fn g() {
    // `g` receives a different stack frame.
    let y = 20;

    println!("{y}");
}

fn main() {
    // Push `f`'s frame.
    f();

    // After `f` returns, its frame can be reused by `g`.
    g();
}
```

```text
main()
│
├── f()
│   └── frame removed after return
│
└── g()
    └── may reuse the same stack memory
```

---

## 👉 Fat Pointers

| Type | Representation |
| --- | --- |
| **Slice** | pointer + length |
| **Trait Object** | pointer + vtable pointer |

- **Slice**: A reference to a subset of some contiguous collection of values, built from a pointer and a length field.
- **Trait Object**: A reference to an item that implements a particular trait, built from a pointer to the item and a pointer to the type's vtable.

### 🍰 Slice

```rust
fn main() {
    // Store four integers contiguously.
    let values = [10, 20, 30, 40];

    // Borrow part of the array.
    let slice: &[i32] = &values[1..3];

    // The slice logically contains a pointer and a length.
    assert_eq!(slice, &[20, 30]);
}
```

```text
&[T]

┌────────────────────┐
│ data pointer       │ ─────► elements
├────────────────────┤
│ length             │
└────────────────────┘
```

### 🎭 Trait Object

```rust
trait Speak {
    // Define behavior shared by implementations.
    fn speak(&self);
}

struct Robot;

impl Speak for Robot {
    fn speak(&self) {
        // Concrete implementation for Robot.
        println!("beep");
    }
}

fn run(value: &dyn Speak) {
    // Dynamic dispatch uses the object's vtable.
    value.speak();
}
```

```text
&dyn Speak

┌────────────────────┐
│ data pointer       │ ─────► Robot
├────────────────────┤
│ vtable pointer     │ ─────► Speak implementation
└────────────────────┘
```

---

## 🧩 Traits and Generics

- **Pointer Trait**: Formats a pointer value for output, useful for low-level debugging.
- **Borrow and BorrowMut Traits**: Methods `borrow` and `borrow_mut` have the same signature as `AsRef` and `AsMut`.
- **ToOwned Trait**: Allows functions to build their own copies of items.
- **AsRef Trait**: Used for cheap reference to reference conversions.
- **Box<T>**: Forces allocation on the heap, allowing the item to outlive the scope of the current block.
- **Rc and Weak<T>**: Used for cyclical data structures and non-owning references, respectively.
- **Cow**: "Clone-on-write" enum that can hold either owned data or a reference to borrowed data.

### 🧬 Generic Trait Bound

```rust
fn function<T: Clone>(t: T) {
    // `T: Clone` guarantees that `clone()` is available.
    let copy = t.clone();

    // Use `copy` or `t`.
}
```

Equivalent:

```rust
fn function<T>(t: T)
where
    // Keep larger trait bounds in a `where` clause.
    T: Clone,
{
    // The compiler knows that `T` implements `Clone`.
    let copy = t.clone();

    // Use `copy` or `t`.
}
```

- **Generics and Trait Bounds**: Generic types depend on their trait bounds, implying a dependency on the trait.

<details>
<summary><strong>🏢 Microsoft Guideline — Keep abstractions simple</strong></summary>

### 🪜 Abstraction Ladder

```text
Concrete Type
     │
     ▼
Generic / impl Trait
     │
     ▼
dyn Trait
```

Use only as much abstraction as necessary.

```rust
fn load(database: &Database) {
    // Use a concrete dependency when there is one meaningful implementation.
}

fn load_generic(database: impl DataAccess) {
    // Use generics when callers should provide implementations.
}

fn load_dynamic(database: &dyn DataAccess) {
    // Use dynamic dispatch when implementations are chosen at runtime.
}
```

> [!TIP]
> **Concrete before generic. Generic before dynamic.**

### 📦 Hide Incidental Ownership Machinery

Avoid exposing internal ownership details:

```rust
pub fn initialize(
    // This forces callers to know about our internal synchronization strategy.
    config: Arc<Mutex<Config>>,
) -> Arc<Server> {
    // ...
}
```

Prefer:

```rust
pub fn initialize(config: Config) -> Server {
    // Keep ownership and synchronization details private.
    // ...
}
```

Internally:

```rust
pub struct Server {
    // Shared ownership remains an implementation detail.
    inner: Arc<ServerInner>,
}

struct ServerInner {
    // Internal state is synchronized without leaking the Mutex publicly.
    config: Mutex<Config>,
}
```

### 🔎 Essential Methods Should Be Inherent

```rust
impl HttpClient {
    pub fn download(&self, url: &str) {
        // Core behavior is directly discoverable on `HttpClient`.
    }
}
```

</details>

---

## 🧬 Trait Coherence and Implementation

- **Trait Coherence**: There exists at most one `impl` of a trait for any given type.
- **Associated Types vs. Generic Types**: Use associated types for a single `impl` per type, and generic types for multiple possible `impls`.
- **Subtraits and Supertraits**: Subtraits refine their supertraits, making methods more specialized, adding guarantees, or extending functionality.

### 🔗 Associated Type

```rust
trait Container {
    // Each implementation chooses one associated item type.
    type Item;

    // Return a reference to that associated type.
    fn get(&self) -> &Self::Item;
}
```

### 🧬 Generic Trait

```rust
trait Convert<T> {
    // The same source type can potentially implement this
    // trait for multiple target types.
    fn convert(&self) -> T;
}
```

### ⬆️ Supertrait

```rust
trait Printable {
    fn print(&self);
}

trait DebugPrintable: Printable {
    // Any DebugPrintable must already satisfy Printable.
    fn debug_print(&self);
}
```

---

## ⏳ Lifetimes

- **Stack Lifetimes**: The lifetime of an item on the stack is the period where a reference to the item is guaranteed not to become invalid.
- **Lifetime Extension and Reduction**: Convert a temporary to a named local variable or add an additional block around a reference to control its lifetime.
- **Non-Lexical Lifetimes**: The compiler treats the endpoint of a reference's lifetime as the last place it's used, rather than the end of the enclosing scope.
- **'static Lifetime**: The only allowed possibility for a returned reference with no input lifetimes, guaranteeing it never goes out of scope.

> Precisely where an item gets automatically dropped depends on whether an item has a name or not.

> One way to see what the compiler calculates as an item's lifetime is to insert a deliberate error for the borrow checker (Item 15) to detect.

> if the compiler can prove to itself that there is no use of a reference beyond a certain point in the code, then it treats the endpoint of the reference's lifetime as the last place it's used, rather than the end of the enclosing scope. This feature (known as non-lexical lifetimes) allows the borrow checker to be a little bit more generous:

- What happens if there are no input lifetimes, but the output return value includes a reference anyway?
  - The only allowed possibility is for the returned reference to have a lifetime that's guaranteed to never go out of scope. This is indicated by the special lifetime `'static`, which is also the only lifetime that has a specific name rather than a placeholder label.

### 🪄 Non-Lexical Lifetimes

```rust
fn main() {
    // Create mutable owned data.
    let mut value = String::from("Rust");

    // Create an immutable reference.
    let borrowed = &value;

    // This is the last use of `borrowed`.
    println!("{borrowed}");

    // The immutable borrow has effectively ended here,
    // even though the lexical scope continues.
    value.push('!');
}
```

### ♾️ `'static`

```rust
fn language() -> &'static str {
    // String literals are embedded in the binary and live
    // for the duration of the program.
    "Rust"
}
```

---

## 🐛 Debugging

- **dbg! Macro**: Superior to `println!` for quick and dirty print logging, printing to stderr and returning its arguments.

```rust
fn main() {
    let x = 21;

    // `dbg!` prints both the expression and its value to stderr.
    // It also returns the evaluated value.
    let answer = dbg!(x * 2);

    println!("{answer}");
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Debug, Display, and diagnostics</strong></summary>

### 🐛 Public Types Should Support `Debug`

```rust
#[derive(Debug)]
pub struct Endpoint {
    // Debug output should expose useful diagnostic state.
    host: String,
    port: u16,
}
```

Sensitive values should be redacted:

```rust
pub struct ApiKey(String);

impl std::fmt::Debug for ApiKey {
    fn fmt(
        &self,
        f: &mut std::fmt::Formatter<'_>,
    ) -> std::fmt::Result {
        // Never expose the actual secret in Debug output.
        f.write_str("ApiKey([REDACTED])")
    }
}
```

### 🖨️ `Display` for Human-Readable Values

```rust
impl std::fmt::Display for Endpoint {
    fn fmt(
        &self,
        f: &mut std::fmt::Formatter<'_>,
    ) -> std::fmt::Result {
        // Produce a concise representation for humans.
        write!(f, "{}:{}", self.host, self.port)
    }
}
```

### 📊 Structured Diagnostics

Instead of:

```rust
// Unstructured production diagnostic output.
println!("opened file {}", path.display());
```

Prefer conceptually:

```rust
tracing::info!(
    // Store data as a structured field.
    file.path = %path.display(),

    // Keep the message concise and stable.
    "file opened",
);
```

</details>

---

## 📦 Heap and Ownership

- **Heap Values**: Every item has an owner, and the lifetime of heap items is tied to stack lifetimes.
- **Box<T>**: Example of heap allocation with ownership.

```rust
{
    // Allocate `Item` on the heap and store the owning Box locally.
    let b: Box<Item> =
        Box::new(Item { contents: 42 });

    // `b` owns the heap allocation.
} // `b` dropped here, so `Item` dropped too.
```

- **Ownership Chain**: The chain of ownership must end at a local variable or a global variable marked as `'static`.

### 📦 `Box<T>` Mental Model

```text
Stack                         Heap

┌──────────────────┐          ┌──────────────┐
│ Box<T> pointer   │ ───────► │      T       │
└──────────────────┘          └──────────────┘
```

---

## 🧵 Thread Safety

- **Send Trait**: Indicates types safe to move between threads.
- **'static Lifetime Bound**: Required for values moved between threads to ensure no stack references are involved.

```rust
use std::thread;

fn main() {
    // The String owns its heap allocation.
    let value = String::from("Rust");

    let handle = thread::spawn(move || {
        // `move` transfers ownership into the spawned thread.
        println!("{value}");
    });

    // Wait for the thread to complete.
    handle.join().unwrap();
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Send compatibility</strong></summary>

```rust
// Compile-time helper: compilation fails if `T` is not `Send`.
const fn assert_send<T: Send>() {}

// Compile-time helper: compilation fails if `T` is not `Sync`.
const fn assert_sync<T: Sync>() {}

// Verify thread-safety properties without runtime cost.
const _: () = assert_send::<MyType>();
const _: () = assert_sync::<MyType>();
```

For futures:

```rust
fn assert_send<T: Send>(_: T) {
    // Type-checking this function call proves the future is `Send`.
}

async fn operation() {
    // Async work goes here.
}

fn verify() {
    // Compilation fails if `operation()` produces a !Send future.
    assert_send(operation());
}
```

</details>

---

## 💡 Quick Reference: Rust Idioms

| Idiom | Description |
| --- | --- |
| **Borrow, don't clone** | Pass `&T` instead of cloning unless ownership is needed |
| **Make illegal states unrepresentable** | Use enums to model valid states only |
| **`?` over `unwrap()`** | Propagate errors, never panic in library/production code |
| **Parse, don't validate** | Convert unstructured data to typed structs at the boundary |
| **Newtype for type safety** | Wrap primitives in newtypes to prevent argument swaps |
| **Prefer iterators over loops** | Declarative chains are clearer and often faster |
| **`#[must_use]` on Results** | Ensure callers handle return values |
| **`Cow` for flexible ownership** | Avoid allocations when borrowing suffices |
| **Exhaustive matching** | No wildcard `_` for business-critical enums |
| **Minimal `pub` surface** | Use `pub(crate)` for internal APIs |

<details>
<summary><strong>🏢 Microsoft Guideline additions</strong></summary>

| Idiom | Description |
| --- | --- |
| **Concrete → generic → `dyn`** | Add abstraction only when needed |
| **Hide ownership machinery** | Keep incidental `Arc`, `Mutex`, `Box`, etc. private |
| **Strong boundary types** | Validate once, then rely on types |
| **One public path per item** | Avoid duplicate public API paths |
| **Builder validation** | Validate combined configuration inside `build()` |
| **Behavior-focused tests** | Test meaningful observable behavior |
| **Structured logging** | Prefer telemetry fields to ad-hoc strings |
| **Small futures** | Avoid unnecessarily large values across `.await` |
| **Additive Cargo features** | Features should compose |
| **Document magic values** | Explain why unusual constants exist |

</details>

---

## 🗒️ Miscellaneous

- `Copy` is default for variables stored on the stack.
- Move is default for heap variables.
- Structs are stored on the heap and move by default in Rust.
- Ad-hoc conversions follow `as_`, `to_`, `into_` conventions.
- Get used to the transformations of `Option` and `Result`, and prefer `Result` to `Option`.
- `enum` are useful for writing functions that take multiple types and for vecs which allow any enum type.
- String methods: `trim()`, `trim_end_matches(w)`, `strip_prefix(p)`, `lines()`, `chars()`, `char_indices()`, `split_whitespace()`, `bytes()`, and `split_at(index)`.
- Vec methods: `extend()` and `append()`.

> [!NOTE]
> 🧠 **Rust nuance:** stack/heap placement does not determine whether a type implements `Copy`, and structs are not automatically heap allocated. A value lives wherever its owner places it.

---

## ❗ Error Handling

- Rust uses `Result` and `Option` types for error handling.
- `Result<T, E>` is used for functions that can return an error.
- `Option<T>` is used for values that may or may not be present.
- Use `unwrap` and `expect` cautiously; prefer `match` or combinators like `map`, `and_then`, etc.
- Use `?` operator to propagate errors.

### 🧠 Error Mental Model

```text
Can it fail?
│
├── No
│
└── Yes
    │
    ├── Missing value only
    │   └── Option<T>
    │
    └── Need failure reason
        └── Result<T, E>
```

### ✅ `Result`

```rust
fn parse_number(
    input: &str,
) -> Result<i32, std::num::ParseIntError> {
    // `parse()` returns a Result instead of panicking.
    input.parse::<i32>()
}
```

### ➡️ Error Propagation with `?`

```rust
fn double(
    input: &str,
) -> Result<i32, std::num::ParseIntError> {
    // Return early with the parsing error when parsing fails.
    let number = input.parse::<i32>()?;

    // Wrap the successful result in `Ok`.
    Ok(number * 2)
}
```

### ❔ `Option`

```rust
fn first(values: &[i32]) -> Option<i32> {
    // `first()` returns `Option<&i32>`.
    // `copied()` converts it to `Option<i32>`.
    values.first().copied()
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Errors, panics, and recovery</strong></summary>

```text
What happened?
│
├── Optional value is absent
│   └── Option<T>
│
├── Expected runtime failure
│   └── Result<T, E>
│
├── Programming invariant is broken
│   └── panic!
│
└── Caller must uphold a memory-safety contract
    └── unsafe
```

### 💥 Helpful Panic Messages

Avoid:

```rust
// This confirms the invariant but gives little debugging context.
assert!(buffer.len() >= HEADER_SIZE);
```

Prefer:

```rust
assert!(
    buffer.len() >= HEADER_SIZE,

    // Include both the observed value and expected condition.
    "buffer too small: got {}, need at least {HEADER_SIZE}",
    buffer.len(),
);
```

### 🧱 Operation-Specific Library Errors

```rust
fn download() -> Result<File, DownloadError> {
    // Download failures are represented by a focused error type.
    todo!()
}

fn start_vm() -> Result<Vm, VmError> {
    // VM failures use their own meaningful error domain.
    todo!()
}
```

### 🔄 Canonical Error Conversions

```rust
impl From<std::io::Error> for AppError {
    fn from(error: std::io::Error) -> Self {
        // Centralize the canonical conversion in one place.
        Self::Io(error)
    }
}
```

```rust
fn load() -> Result<Config, AppError> {
    // `?` automatically invokes `From<std::io::Error>`.
    let bytes = std::fs::read("config.toml")?;

    // Continue along the happy path without repetitive map_err calls.
    parse_config(&bytes)
}
```

</details>

---

## 🧪 Testing

- Use `#[test]` attribute to mark test functions.
- Use `assert!`, `assert_eq!`, and `assert_ne!` macros for assertions.
- Run tests with `cargo test`.
- Use `#[cfg(test)]` to include test code only when running tests.

```rust
fn add(a: i32, b: i32) -> i32 {
    // Pure functions are especially straightforward to test.
    a + b
}

#[cfg(test)]
mod tests {
    // Import items from the parent module.
    use super::*;

    #[test]
    fn adds_numbers() {
        // Verify observable behavior.
        assert_eq!(add(2, 3), 5);
    }
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Test observable behavior</strong></summary>

Avoid tautological tests:

```rust
const MAX_USERS: usize = 100;

#[test]
fn max_users_is_100() {
    // This mostly repeats the source code rather than testing behavior.
    assert_eq!(MAX_USERS, 100);
}
```

Prefer:

```rust
#[test]
fn rejects_users_after_capacity() {
    // Create a pool with observable capacity behavior.
    let mut users = UserPool::with_capacity(2);

    // First two inserts should succeed.
    assert!(users.add(user()).is_ok());
    assert!(users.add(user()).is_ok());

    // Third insert should be rejected.
    assert!(users.add(user()).is_err());
}
```

### 📂 Integration Tests

```text
crate/
├── src/
│   └── lib.rs
└── tests/
    ├── basic.rs
    └── integration.rs
```

### 🧰 Test Utilities

```toml
[features]

# Enables APIs that exist only to support testing.
test-util = []
```

```rust
#[cfg(feature = "test-util")]
pub fn disable_certificate_validation() {
    // This escape hatch is intentionally unavailable in normal builds.
}
```

</details>

---

## 🛠️ Common Commands

```bash
# ─────────────────────────────────────────────
# 🚀 Starter commands
# ─────────────────────────────────────────────

cargo new project_name
# Create a new Cargo project.

cargo run
# Build and run the current binary.

cargo doc --open
# Generate documentation and open it in the browser.


# ─────────────────────────────────────────────
# 🔨 Build and check
# ─────────────────────────────────────────────

cargo build
# Compile the project.

cargo check
# Type-check quickly without generating the final executable.

cargo clippy
# Run Rust's linting and code-quality suggestions.

cargo fmt
# Format source code using rustfmt.


# ─────────────────────────────────────────────
# 🧪 Testing
# ─────────────────────────────────────────────

cargo test
# Run all tests.

cargo test -- --nocapture
# Run tests while allowing stdout/stderr output to remain visible.

cargo test --lib
# Run library unit tests only.

cargo test --test integration
# Run the integration test target named `integration`.


# ─────────────────────────────────────────────
# 📦 Dependencies
# ─────────────────────────────────────────────

cargo audit
# Check dependencies for known security vulnerabilities.

cargo tree
# Display the dependency graph.

cargo update
# Update dependencies while respecting Cargo.toml constraints.


# ─────────────────────────────────────────────
# 🚀 Performance
# ─────────────────────────────────────────────

cargo bench
# Run benchmark targets.
```

<details>
<summary><strong>🏢 Microsoft Guideline — Verification Pipeline</strong></summary>

```bash
cargo fmt --check
# Verify formatting without modifying files.

cargo check --all-targets
# Type-check libraries, binaries, tests, benches, and examples.

cargo clippy --all-targets --all-features
# Run Clippy against the broadest normal project configuration.

cargo test --all-features
# Exercise tests with all Cargo features enabled.

cargo audit
# Check dependency advisories.

cargo hack check --feature-powerset
# Verify feature combinations when cargo-hack is installed.

cargo miri test
# Interpret tests with Miri to detect classes of undefined behavior.

cargo bench
# Benchmark performance-sensitive paths.
```

```text
cargo fmt
    │
    ▼
cargo check
    │
    ▼
cargo clippy
    │
    ▼
cargo test
    │
    ▼
cargo audit
    │
    ▼
feature checks
    │
    ▼
Miri / benchmarks
```

</details>

---

## ✅ Best Practices

- Write idiomatic Rust code by following the Rust Style Guide.
- Use `clippy` to catch common mistakes and improve code quality.
- Keep functions small and focused.
- Prefer immutability and pure functions.
- Document your code with comments and `///` doc comments.
- Write tests for your code to ensure correctness.
- Use `cargo fmt` to format your code.

<details>
<summary><strong>🏢 Microsoft Guideline — Magic values and lint exceptions</strong></summary>

### 🔢 Document Magic Values

Avoid:

```rust
// Why exactly one day?
wait_timeout(
    Duration::from_secs(86_400),
)
.await;
```

Prefer:

```rust
/// Maximum upstream request duration.
///
/// Matches the upstream service contract and prevents
/// premature cancellation of valid long-running requests.
const UPSTREAM_TIMEOUT: Duration =
    Duration::from_secs(86_400);
```

### 🧹 Prefer `#[expect]`

```rust
#[expect(
    clippy::unused_async,

    // Explain why the unusual code is intentional.
    reason = "API remains async for upcoming I/O implementation",
)]
pub async fn ping() {
    // ...
}
```

</details>

---

# 🔀 Data Races and Concurrency

A data race occurs when two distinct threads access the same memory location, where at least one of them is a write, and there is no synchronization mechanism that enforces an ordering on the accesses.

Enforcing that there is a single writer, or multiple readers (but never both), means that there can be no data races.

> "Do not communicate by sharing memory; instead, share memory by communicating."

In Rust, equivalent functionality is included in the standard library in the `std::sync::mpsc` module: the `channel()` function returns a `(Sender, Receiver)` pair that allows values of a particular type to be communicated between threads.

### 📨 Message Passing

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    // Create a channel with sending and receiving endpoints.
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        // Ownership of `tx` is moved into the worker thread.
        tx.send("hello from worker").unwrap();
    });

    // Block until the worker sends a value.
    println!("{}", rx.recv().unwrap());
}
```

If shared-state concurrency can't be avoided, then there are some ways to reduce the chances of writing deadlock-prone code:

- Ensure that access to it is governed by some kind of external synchronization for users of a struct.
- Ensure it is thread-safe by adding internal synchronization operations.
- Use `Arc`, `Mutex`, and `RwLock`.
- Avoid interaction between the two independently locked data structures where possible.
- Use `sleep()` to help with debugging threads.
- **Avoid lock inversion**: Try to lock stuff in a good ordering, to make it more deterministic.
- Put data structures that must be kept consistent with each other under a single lock.
- Keep lock scopes small and obvious; try to make it so no locks are held at the same time. Wherever possible, use helper methods that get and set things under the relevant lock.
- Avoid invoking closures with locks held; this puts the code at the mercy of whatever closure gets added to the codebase in the future.
- Similarly, avoid returning a `MutexGuard` to a caller: it's like handing out a loaded gun from a deadlock perspective.
- Include deadlock detection tools in your CI system, such as `no_deadlocks`, `ThreadSanitizer`, or `parking_lot::deadlock`.
- As a last resort: design, document, test, and police a locking hierarchy that describes what lock orderings are allowed/required. This should be a last resort because any strategy that relies on engineers never making a mistake is obviously doomed to failure in the long term.

More abstractly, multi-threaded code is an ideal place to apply the general advice: prefer code that's so simple that it is obviously not wrong, rather than code that's so complex that it's not obviously wrong.

### 🔒 `Arc<Mutex<T>>`

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // Arc enables shared ownership across threads.
    // Mutex provides exclusive mutable access.
    let counter = Arc::new(Mutex::new(0));

    // Clone the Arc handle, not the underlying counter.
    let worker_counter = Arc::clone(&counter);

    let worker = thread::spawn(move || {
        // Lock the Mutex before mutating the value.
        let mut value =
            worker_counter.lock().unwrap();

        *value += 1;

        // The MutexGuard is dropped here, releasing the lock.
    });

    worker.join().unwrap();

    // Safely inspect the final value.
    println!("{}", *counter.lock().unwrap());
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Avoid correctness-sensitive global state</strong></summary>

Avoid:

```rust
static GLOBAL_COUNTER: AtomicUsize =
    AtomicUsize::new(0);
// This assumes one globally unique copy of the crate state.
```

Prefer:

```rust
struct Metrics {
    // State belongs to an explicit object.
    requests: AtomicUsize,
}

struct App {
    // Ownership and lifetime are now explicit.
    metrics: Arc<Metrics>,
}
```

</details>

---

# 🏗️ Rust Variable and Struct Handling

- `Copy` is default for variables stored on the stack.
- Move is default for heap variables.
- Everything that is not a copy is a move.
- Structs are stored on the heap and move by default in Rust.
- Add `mut` to a struct variable to make every field mutable.
- Structs can be nested.
- Use `self` if its value is on the stack.

### 🧱 Mutable Struct

```rust
struct User {
    // Owned heap-backed String.
    name: String,

    // Small numeric field.
    age: u8,
}

fn main() {
    // `mut` allows fields to be changed.
    let mut user = User {
        name: String::from("Albert"),
        age: 20,
    };

    // Mutate one field through the mutable binding.
    user.age += 1;
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Cheap service handles</strong></summary>

```rust
#[derive(Clone)]
pub struct Database {
    // Cloning Database only clones the Arc handle.
    inner: Arc<DatabaseInner>,
}

struct DatabaseInner {
    // Heavy shared resources live here.
}
```

</details>

---

# 🔄 Ad-hoc Conversions

Ad-hoc conversions follow `as_`, `to_`, `into_` conventions (C-CONV):

| Prefix | Cost | Ownership |
| --- | --- | --- |
| `as_` | Free | borrowed → borrowed |
| `to_` | Expensive | borrowed → borrowed |
| `to_` | Expensive | borrowed → owned (non-Copy types) |
| `to_` | Expensive | owned → owned (Copy types) |
| `into_` | Variable | owned → owned (non-Copy types) |

```text
as_*    → VIEW

to_*    → CONVERT / ALLOCATE

into_*  → CONSUME
```

- Conversions prefixed `as_` and `into_` **typically decrease abstraction**:
  - Exposing a view into the underlying representation (`as`).
  - Deconstructing data into its underlying representation (`into`).
- Conversions prefixed `to_`:
  - Typically stay at the same level of abstraction.
  - Do some work to change from one representation to another.

**When a type wraps a single value to associate it with higher-level semantics, access to the wrapped value should be provided by an `into_inner()` method**. This applies to wrappers that provide buffering like `BufReader`, encoding or decoding like `GzDecoder`, atomic access like `AtomicBool`, or any similar semantics.

### 📤 `into_inner()`

```rust
struct UserId(u64);

impl UserId {
    fn into_inner(self) -> u64 {
        // Consume the wrapper and return the underlying primitive.
        self.0
    }
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Prefer semantic types</strong></summary>

Avoid:

```rust
fn transfer(
    // Easy to accidentally swap primitive arguments.
    account: u64,
    currency: String,
    amount: u64,
) {
    // ...
}
```

Prefer:

```rust
struct AccountId(u64);
struct Currency(String);
struct Amount(u64);

fn transfer(
    // Types communicate intent and prevent argument swaps.
    account: AccountId,
    currency: Currency,
    amount: Amount,
) {
    // ...
}
```

</details>

---

# 🔁 Option and Result Transformations

- Get used to the transformations of `Option` and `Result`, and prefer `Result` to `Option`.
  - Use `.as_ref()` as needed when transformations involve references.
- Use them in preference to explicit `match` operations.
- In particular, use them to transform result types into a form where the `?` operator applies.

> *Note: This method is separate from the `AsRef` trait, even though the method name is the same.*

### 🧹 Instead of Explicit Matching

```rust
let output = match option {
    // Transform the contained value.
    Some(value) => Some(value * 2),

    // Preserve absence.
    None => None,
};
```

Use:

```rust
// `map` performs the same transformation more directly.
let output =
    option.map(|value| value * 2);
```

### 🔎 `.as_ref()`

```rust
// Own a String inside an Option.
let value =
    Some(String::from("Rust"));

// Borrow the inner String instead of moving it.
let length = value
    .as_ref()
    .map(|text| text.len());
```

---

# 📌 Data Structures and Self-Referential Data

**Data structures in Rust can move**: from the stack to the heap, from the heap to the stack, and from one place to another. If that happens, the "interior" title pointer would no longer be valid, and there's no way to keep it in sync.

A simple alternative for this case is to use the indexing approach explored earlier; **a range of offsets into the text is not invalidated by a move, and is invisible to the borrow checker because it doesn't involve references**.

### 📏 Offset-Based Alternative

```rust
use std::ops::Range;

struct Document {
    // Own the entire text.
    text: String,

    // Store offsets into `text` instead of a self-reference.
    title: Range<usize>,
}
```

A more general version of the self-reference problem turns up when the compiler deals with `async` code. Roughly speaking, the compiler bundles up a pending chunk of `async` code into a lambda, and the data for that lambda can include both values and references to those values. That's inherently a self-referential data structure, and so `async` support was a prime motivation for the `Pin` type in the standard library. **This pointer type "pins" its value in place, forcing the value to remain at the same location in memory, thus ensuring that internal self-references remain valid**.

The internal reference fields need to use raw pointers, or near relatives (e.g. `NonNull`) thereof. The type being pinned needs to not implement the `Unpin` marker trait. This trait is automatically implemented for almost every type, so this typically involves adding a (zero-sized) field of type `PhantomPinned` to the struct definition.

The item is only pinned once it's on the heap and held via `Pin`; in other words, only the contents of something like `Pin<Box<MyType>>` is pinned. This means that the internal reference fields can only be safely filled in after this point, but as they are raw pointers the compiler will give you no warning if you incorrectly set them before calling `Box::pin`.

Where possible, avoid self-referential data structures or try to find library crates that encapsulate the difficulties for you (e.g. ouroborous).

---

# 🎛️ Enums

Enums are useful because they allow writing functions that take multiple types and for vecs which allow any enum type. The size of an enum will not exceed the largest variant and needed tag.

```rust
enum Value {
    // Store an integer.
    Integer(i64),

    // Store a floating-point value.
    Float(f64),

    // Store owned text.
    Text(String),
}

let values = vec![
    Value::Integer(42),
    Value::Float(3.14),
    Value::Text(String::from("Rust")),
];
// A Vec can contain different variants because every element
// still has the same outer type: `Value`.
```

---

# 🔤 String Methods

- Chars are 4 bytes, not one in Rust.
- `trim()` removes whitespace and `trim_matches` removes strings.
- `trim_end_matches(w)` removes occurrences of `w`.
- `strip_prefix(p)` removes `p` at most once.
- You can split strings by `lines()`, `chars()`, `char_indices()`, `split_whitespace()`, `bytes()`, and `split_at(index)`.
- Indexing a string doesn't give a char.

| Method | Purpose |
| --- | --- |
| `trim()` | Remove surrounding whitespace |
| `trim_matches()` | Remove matching patterns |
| `trim_end_matches(w)` | Remove suffix matches |
| `strip_prefix(p)` | Remove one matching prefix |
| `lines()` | Iterate over lines |
| `chars()` | Iterate over Rust `char` values |
| `char_indices()` | Iterate over byte offsets + chars |
| `split_whitespace()` | Split by whitespace |
| `bytes()` | Iterate over UTF-8 bytes |
| `split_at(index)` | Split at a valid byte boundary |

```rust
fn main() {
    let text =
        "  Rust is awesome  ";

    // Remove surrounding whitespace.
    println!("{}", text.trim());

    // Iterate over whitespace-separated words.
    for word in text.split_whitespace() {
        println!("{word}");
    }
}
```

> [!NOTE]
> Rust `char` is 4 bytes, while UTF-8 encoded characters inside a `String` may occupy 1–4 bytes.

---

# 📚 Vec Methods

- `extend()` (append by clone) lets you append to a vector by cloning contents of the second vector.
- `append()` normally just appends by move.
- `concat()` adds two containers together, while `join()` adds the two by a separator.
- `starts_with()` and `ends_with()`, `capacity()`, `length()`, `contains()`.
- Indexing collections can cause panics, but using `get(index)` allows you to get options back.

### 📎 `append()`

```rust
let mut first =
    vec![1, 2];

let mut second =
    vec![3, 4];

// Move every element from `second` into `first`.
first.append(&mut second);

assert_eq!(
    first,
    vec![1, 2, 3, 4],
);

// `append` leaves the source vector empty.
assert!(second.is_empty());
```

### 🛟 Safe Indexing

```rust
let values =
    vec![10, 20, 30];

// `get` returns Option instead of panicking.
if let Some(value) =
    values.get(1)
{
    println!("{value}");
}
```

---

# 🔢 Power Functions

- `pow()` for power.
- `powf()` for power float.
- `take()` replaces option with `None` and returns a new option with the value.
- `ok_or` turns `Some(valid_value)` into `Result::Ok(valid_value)` and if it's a `None` into a `Result::Err(err_value)`.
- `hash` for mapping a value of fixed size has to implement `Eq` and `PartialEq`.

```rust
// Integer exponentiation.
let integer =
    2_i32.pow(8);

// Floating-point exponentiation.
let float =
    2.0_f64.powf(3.5);
```

### 📤 `Option::take()`

```rust
let mut value =
    Some(String::from("Rust"));

// Move the String out and replace the Option with None.
let extracted =
    value.take();

assert!(value.is_none());

assert_eq!(
    extracted.as_deref(),
    Some("Rust"),
);
```

### ➡️ `ok_or()`

```rust
let value =
    Some(42);

// Convert Option into Result.
let result =
    value.ok_or("missing value");

assert_eq!(
    result,
    Ok(42),
);
```

---

# 🔀 Conversion Traits

The four relevant traits that express the ability to convert values of a type are:

- `From<T>`: Items of this type can be built from items of type `T`.
- `TryFrom<T>`: Items of this type can sometimes be built from items of type `T`.
- `Into<T>`: Items of this type can be converted into items of type `T`.
- `TryInto<T>`: Items of this type can sometimes be converted into items of type `T`.

**Implement (just) the `Try`... trait if it's possible for a conversion to fail.**

**Implement the `From` trait for conversions.**

**Use the `Into` trait for trait bounds.**

### 📋 Conversion Cheat Sheet

| Trait | Meaning |
| --- | --- |
| `From<T>` | Infallibly build from `T` |
| `TryFrom<T>` | Fallibly build from `T` |
| `Into<T>` | Infallibly convert into `T` |
| `TryInto<T>` | Fallibly convert into `T` |

### 🔄 `From`

```rust
struct UserId(u64);

impl From<u64> for UserId {
    fn from(value: u64) -> Self {
        // Wrap a raw ID in a semantic type.
        Self(value)
    }
}

let id =
    UserId::from(42);

// `From<u64>` automatically enables `Into<UserId>`.
let another_id: UserId =
    42_u64.into();
```

However, a generic version of the function that accepts (and explicitly converts) anything satisfying `Into<IanaAllocated>`:

With this trait bound in place, the reflexive trait implementation of `From<T>` makes more sense: the combination of `From<T>` implementations and `Into<T>` trait bounds leads to code that appears to magically convert at the call site (but which is still doing safe, explicit, conversions under the covers). This pattern becomes even more powerful when combined with reference types and their related conversion traits.

There are only two coercions whose behavior can be affected by user-defined types. The first of these is when a user-defined type implements the `Deref` or the `DerefMut` trait. These traits indicate that the user-defined type is acting as a smart pointer of some sort, and in this case, the compiler will coerce a reference to the smart pointer item into being a reference to an item of the type that the smart pointer contains (indicated by its Target).

The second coercion of a user-defined type happens when a concrete item is converted to a trait object. This operation builds a fat pointer to the item; this pointer is fat because it includes both a pointer to the item's location in memory, together with a pointer to the vtable for the concrete type's implementation of the trait.

Rust includes the `as` keyword to perform explicit casts between some pairs of types. The `as` version also allows lossy conversions.

### 🪄 `Deref`

```rust
use std::ops::Deref;

struct Name(String);

impl Deref for Name {
    // Dereferencing Name produces a `str`.
    type Target = str;

    fn deref(&self) -> &Self::Target {
        // Borrow the inner String as `str`.
        &self.0
    }
}
```

---

# ♻️ Closures and Iterators

- `fnonce` consumes ownership of value.
- Statements are instructions that perform some action and don't return a value.
- Expressions evaluate to a resulting value.
- `and_then()` returns `None` if the option is `None`, else calls the `fnonce()` closure you send to it.
- `or_else()` returns the option if it contains a value, otherwise calls `f()` and returns the result.

### 🧠 Closure Traits

```text
Fn
│
└── FnMut
    │
    └── FnOnce
```

### 🔗 `and_then()`

```rust
let value =
    Some("42");

let result = value.and_then(|value| {
    // Parse the string.
    // Convert Result to Option with `.ok()`.
    value.parse::<u32>().ok()
});

assert_eq!(
    result,
    Some(42),
);
```

### 🔁 `or_else()`

```rust
let value: Option<u32> =
    None;

// Supply a fallback only when the original Option is None.
let result =
    value.or_else(|| Some(42));

assert_eq!(
    result,
    Some(42),
);
```

---

# 🔽 Iterators Consumers

- `iter()` and `fold()`, let you iterate while holding state.
- `scan()` takes an initial value for internal state and a closure with two arguments: one to a mutable reference to internal state, and the other an iterator element.
- `any(P)` tests if any element in the collection matches the predicate, returns true if at least one does.
- `all(p)` tests if every element in the collection matches the predicate and returns true if they all do.

If the body of the for loop matches one of a number of common patterns, there are more specific iterator-consuming methods that are clearer, shorter, and more idiomatic. These patterns include shortcuts for building a single value out of the collection:

- `sum()`, for summing a collection of numeric values (integers or floats).
- `product()`, for multiplying together a collection of numeric values.
- `min()` and `max()`, for finding the extreme values of a collection, relative to the Item's `Ord` implementation.
- `min_by(f)` and `max_by(f)`, for finding the extreme values of a collection, relative to a user-specified comparison function `f`.
- `reduce(f)` is a more general operation that encompasses the previous methods, building an accumulated value of the Item type by running a closure at each step that takes the value accumulated so far and the current item.
- `fold(f)` is a generalization of reduce, allowing the "accumulated value" to be of an arbitrary type (not just the `Iterator::Item` type).
- `scan(f)` generalizes in a slightly different way, giving the closure a mutable reference to some internal state at each step.

There are also methods for selecting a single value out of the collection:

- `find(p)` finds the first item that satisfies a predicate.
- `position(p)` also finds the first item satisfying a predicate, but this time it returns the index of the item.
- `nth(n)` returns the n-th element of the iterator, if available.

There are methods for testing against every item in the collection:

### ⚡ Consumer Cheat Sheet

| Method | Purpose |
| --- | --- |
| `sum()` | Sum values |
| `product()` | Multiply values |
| `min()` / `max()` | Find extremes |
| `min_by()` / `max_by()` | Custom comparison |
| `reduce()` | Combine items |
| `fold()` | Accumulate state |
| `scan()` | Stateful processing |
| `find()` | First matching item |
| `position()` | Index of match |
| `nth()` | Nth item |
| `any()` | Any item matches |
| `all()` | Every item matches |

### ➕

```rust
let values =
    [1, 2, 3, 4];

// Consume the iterator into a single sum.
let total: i32 =
    values.iter().sum();

assert_eq!(total, 10);
```

<details>
<summary><strong>🏢 Microsoft Guideline — Collection interoperability</strong></summary>

Custom collections should consider implementing:

```text
iter()
iter_mut()

IntoIterator for Collection<T>
IntoIterator for &Collection<T>
IntoIterator for &mut Collection<T>

FromIterator<T>
Extend<T>

IntoIter<T>
Iter<T>
IterMut<T>
```

Where appropriate:

```text
DoubleEndedIterator
ExactSizeIterator
```

Provide truthful `size_hint()` information because consumers such as `collect()` can use it for allocation planning.

</details>

---

# 🔧 Iterator Transforms

The `Iterator` trait has a single required method (`next`), but also provides default implementations of a large number of other methods that perform transformations on an iterator.

### 🛤️ Iteration Control

| Adapter | Purpose |
| --- | --- |
| `take(n)` | Limit output |
| `skip(n)` | Skip initial items |
| `step_by(n)` | Emit every nth item |
| `chain(other)` | Join iterators |
| `cycle()` | Repeat |
| `rev()` | Reverse |

Some of these transformations affect the overall iteration process:

- `take(n)` restricts an iterator to emitting at most `n` items.
- `skip(n)` skips over the first `n` elements of the iterator.
- `step_by(n)` converts an iterator so it only emits every `n`-th item.
- `chain(other)` glues together two iterators, to build a combined iterator that moves through one then the other.
- `cycle()` converts an iterator that terminates into one that repeats forever, starting at the beginning again whenever it reaches the end. (The iterator must support `Clone` to allow this.)
- `rev()` reverses the direction of an iterator. (The iterator must implement the `DoubleEndedIterator` trait, which has an additional `next_back` required method.)

Other transformations affect the nature of the Item that's the subject of the `Iterator`:

- `map(|item| {...})` is the most general version, repeatedly applying a closure to transform each item in turn. Several of the following entries in this list are convenience variants that could be equivalently implemented as a map.
- `cloned()` produces a clone of all of the items in the original iterator; this is particularly useful with iterators over `&Item` references. (This obviously requires the underlying `Item` type to implement `Clone`).
- `copied()` produces a copy of all of the items in the original iterator; this is particularly useful with iterators over `&Item` references. (This obviously requires the underlying `Item` type to implement `Copy`).
- `enumerate()` converts an iterator over items to be an iterator over `(usize, Item)` pairs, providing an index to the items in the iterator.
- `zip(it)` joins an iterator with a second iterator, to produce a combined iterator that emits pairs of items, one from each of the original iterators, until the shorter of the two iterators is finished.

Yet other transformations perform filtering on the Items being emitted by the `Iterator`:

- `filter(|item| {...})` is the most general version, applying a bool-returning closure to each item reference to determine whether it should be passed through.
- `take_while()` and `skip_while()` are mirror images of each other, emitting either an initial subrange or a final subrange of the iterator, based on a predicate.

The `flatten()` method deals with an iterator whose items are themselves iterators, flattening the result. On its own, this doesn't seem that helpful, but it becomes much more useful when combined with the observation that both `Option` and `Result` act as iterators: they produce either zero (for `None`, `Err(e)`) or one (for `Some(v)`, `Ok(v)`) items. This means that flattening a stream of `Option` / `Result` values is a simple way to extract just the valid values, ignoring the rest.

```rust
let values = vec![
    Some(1),
    None,
    Some(3),
];

// Convert Option values into zero-or-one-item iterators,
// then flatten them into one stream.
let values: Vec<_> =
    values
        .into_iter()
        .flatten()
        .collect();

assert_eq!(
    values,
    vec![1, 3],
);
```

Taken as a whole, these methods allow iterators to be transformed so that they produce exactly the sequence of elements that are needed for most situations.

<details>
<summary><strong>🏢 Microsoft Guideline — Allocation and throughput</strong></summary>

### 📦 Preallocate

```rust
// Reserve enough memory for the expected number of elements.
let mut output =
    Vec::with_capacity(input.len());

for value in input {
    // No repeated growth should be needed while capacity is sufficient.
    output.push(convert(value));
}
```

Often:

```rust
let output: Vec<_> =
    input
        .iter()

        // Transform each element.
        .map(convert)

        // `collect` can use iterator size hints.
        .collect();
```

### ♻️ Reuse Existing Allocations

```rust
// Allocate one buffer with reusable capacity.
let mut buffer =
    String::with_capacity(4096);

loop {
    // Remove contents without necessarily freeing the allocation.
    buffer.clear();

    // Refill the existing buffer.
    read_record(&mut buffer)?;
}
```

### 📈 Measure First

```text
Measure
  │
  ▼
Find Hot Path
  │
  ▼
Form Hypothesis
  │
  ▼
Optimize
  │
  ▼
Benchmark
  │
  ├── Faster → keep
  └── No improvement → revert
```

</details>

---

# 🧾 Code Snippets

```rust
pub struct Ref<'a, T: 'a>(
    // `Ref` explicitly borrows a value for lifetime `'a`.
    &'a T,
);
```

This generic data structure holds an explicit reference `&'a T`, as per the first bullet above. But the type `T` might itself contain references with some lifetime `'b`, as per the second bullet above. If `T`'s inherent lifetime `'b` were smaller than the exterior lifetime `'a` we'd have a potential disaster: the `Ref` would be holding a reference to a data structure whose own references have gone bad.

To prevent this, we need `'b` to be larger than `'a`.

One common place this shows up is when you try to move values between threads with `std::thread::spawn`. The moved values need to be of types that implement `Send`, indicating that they're safe to move between threads, but they also need to not contain any dynamic references (the `'static` lifetime bound). This makes sense when you realize that a reference to something on the stack now raises the question: which stack? Each thread's stack is independent, and so lifetimes can't be tracked between them.

---

```rust
pub trait Future {
    // The value produced when the future completes.
    type Output;

    fn poll(
        // `Pin` prevents moves that could invalidate self-references.
        self: Pin<&mut Self>,

        // Context carries the task's Waker.
        cx: &mut Context<'_>,
    ) -> Poll<Self::Output>;
}

pub enum Poll<T> {
    // The asynchronous computation completed.
    Ready(T),

    // The computation cannot currently make progress.
    Pending,
}
```

- `Future` has to be `poll`ed (by the executor) to resume where it last yielded and make progress (async is lazy).

---

# 🏁 Conclusion

Rust is a powerful and safe systems programming language that enforces memory safety and concurrency without a garbage collector. By understanding and utilizing Rust's ownership model, lifetimes, and concurrency primitives, you can write efficient and reliable code. Use the resources and best practices outlined in this guide to deepen your knowledge and improve your Rust programming skills.

---

# ⚡ ASync Definitions

| Term | Definition |
| --- | --- |
| **shared** | memory threads operate on regions of shared memory |
| **worker pools** | many identical threads receive jobs from a shared job queue |
| **actors** | many different job queues, one for each actor; actors communicate exclusively by exchanging messages |
| **Green threads / virtual threads** | Threads scheduled by a runtime or VM instead of directly by the OS |
| **Task** | An asynchronous green thread. |
| **Executor** | Runs asynchronous tasks.. |
| **Generator** | Used internally by the compiler. Can stop/yield execution and resume later |
| **Reactor** | Leaf futures register event sources with the reactor |
| **Runtime** | Bundles a reactor and an executor |

### 🏭 Runtime Model

```text
┌──────────────────────── Runtime ─────────────────────────┐
│                                                         │
│      ┌───────────────┐       ┌─────────────────┐        │
│      │   Executor    │       │     Reactor     │        │
│      └───────┬───────┘       └────────┬────────┘        │
│              │                        │                 │
│              ▼                        ▼                 │
│            Tasks                  I/O Events             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

# ⚙️ Futures and Async in Rust

- `Future` has to be `poll`ed (by the executor) to resume where it last yielded and make progress (async is lazy).
- `&mut Self` contains state (state machine).
- `Pin` the memory location because the future contains self-referential data.
- `Context` contains the Waker to notify the executor that progress can be made.
- `async`/`await` on futures is implemented by generators.
- `async fn` and `async` blocks return `impl Future<Output = T>`.
- Calling `.await` attempts to resolve the `Future`: if the `Future` is blocked, it yields control; if progress can be made, the `Future` resumes.

Futures form a tree of futures. The leaf futures communicate with the executor. The root future of a tree is called a **task**.

```text
Task
│
├── Future A
│   ├── Future C
│   └── Future D
│
└── Future B
    └── Leaf Future
        │
        ▼
      Reactor
```

### 💤 Async Function

```rust
async fn fetch_value() -> u32 {
    // The async function eventually resolves to this value.
    42
}

fn example() {
    // Calling an async function creates a Future.
    // It does not synchronously run the entire computation.
    let future =
        fetch_value();
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Async API design</strong></summary>

### ✅ Prefer `async fn`

```rust
async fn fetch()
    -> Result<Data, Error>
{
    // Clear and idiomatic async API.
    todo!()
}
```

### 📦 Future Size

```rust
async fn process() {
    // This large value remains part of the Future
    // if it must survive across the await point.
    let huge =
        HugeBuffer::new();

    io().await;

    // `huge` is still needed afterward.
    consume(huge);
}
```

```text
Future State
├── state discriminant
├── parameters
├── huge
└── other locals crossing `.await`
```

### 🤝 Keep Public Futures `Send`

```rust
fn assert_send<T: Send>(_: T) {
    // Compilation proves the value is Send.
}

async fn operation() {
    // ...
}

fn check() {
    // Compilation fails if this Future is !Send.
    assert_send(operation());
}
```

### 🧮 CPU-Bound Async Work Must Cooperate

```rust
for chunk in chunks {
    // Perform a reasonable unit of CPU-heavy work.
    process(chunk);

    // Give other executor tasks a chance to run.
    tokio::task::yield_now().await;
}
```

</details>

---

# 🚀 Async Runtimes

- **tokio** (multithreaded): Thread pool with work-stealing scheduler: each processor maintains its own run queue; idle processor checks sibling processor run queues and attempts to steal tasks from them.
- **actix_rt**: Single-threaded async runtime; futures are `!Send`.
- **actix-web**: Constructs an application instance for each thread; application data must be constructed multiple times or shared between threads.

```rust
#[tokio::main]
async fn main() {
    // The macro initializes a Tokio runtime,
    // then executes this async main Future.
    println!("Hello from Tokio");
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Isolate runtime-specific details</strong></summary>

```text
Business Logic
      │
      ▼
Runtime Abstraction
      │
      ├── Tokio
      ├── another runtime
      └── test runtime
```

Keep runtime-specific types out of domain APIs unless runtime integration is itself the feature.

</details>

---

# 🌊 Streams

`Stream<Item = T>` is an asynchronous version of `Iterator<Item = T>`, i.e., it does not block between each item yield.

### 🏗️ Create

- `futures::stream::iter`
- `futures::stream::once`
- `futures::stream::repeat`
- `futures::stream::repeat_with`
- `async_stream::stream`

### 🔗 Combine (n-1)

- `futures::stream::StreamExt::chain`
- `futures::stream::StreamExt::zip`
- `tokio_stream::StreamExt::merge`
- `tokio_stream::StreamMap`
- `tokio::select`

### ✂️ Split operations (1-n)

- `futures::channel::oneshot::Sender::send`
- `async_std::channel::Sender::send`

---

# 🚨 Data Races

A data race occurs when two distinct threads access the same memory location, where at least one of them is a write, and there is no synchronization mechanism that enforces an ordering on the accesses.

## 🛡️ Avoiding Data Races

- Ensure a single writer or multiple readers (but never both).
- Use `std::sync::mpsc` for message passing.
- Use `Arc`, `Mutex`, and `RwLock` for shared-state concurrency.
- Avoid lock inversion and keep lock scopes small and obvious.
- Use deadlock detection tools like `no_deadlocks`, `ThreadSanitizer`, or `parking_lot::deadlock`.

---

# 📍 Pinning and Self-Referential Data Structures

- Use `Pin` to ensure that internal self-references remain valid.
- Avoid self-referential data structures or use library crates that encapsulate the difficulties (e.g., ouroboros).

---

# ❔ Option Methods

- `take()` replaces option with `None` and returns a new option with the value.
- `ok_or` turns `Some(valid_value)` into `Result::Ok(valid_value)` and `None` into `Result::Err(err_value)`.

---

# #️⃣ Hashing

- Implement `Eq` and `PartialEq` for hashing.

```rust
use std::collections::HashMap;

fn main() {
    // Create an empty hash map.
    let mut values =
        HashMap::new();

    // String slices implement Hash + Eq,
    // so they can be used as keys.
    values.insert(
        "rust",
        1,
    );
}
```

> [!NOTE]
> Hash-map keys generally require **`Hash + Eq`**.

---

# 🔌 Adapters and Iterators

Adapters are functions that take iterators and return other iterators.

```text
Collection
    │
    ▼
 .iter()
    │
    ▼
.filter(...)
    │
    ▼
 .map(...)
    │
    ▼
 .take(...)
    │
    ▼
.collect() / .sum() / .fold(...)
```

---

## 🔢 Replacing Vector Indexing with Iterators

The first step is to replace vector indexing with direct use of an iterator in a for-each loop:

```rust
for i in 0..values.len() {
    // Indexing may panic if `i` were somehow invalid.
    if values[i] % 2 != 0 {
        continue;
    }
}
```

to:

```rust
for value in values.iter() {
    // Iterate directly over references to elements.
    if value % 2 != 0 {
        continue;
    }
}
```

---

## 🔍 Using `filter()`

An initial arm of the loop that uses `continue` to skip over some items is naturally expressed as a `filter()`:

```rust
for value in values
    .iter()

    // Keep only even values.
    .filter(|x| *x % 2 == 0)
{
    // Process each retained value.
}
```

---

## ✋ Using `take()`

Next, the early exit from the loop once 5 even items have been spotted maps to a `take(5)`:

```rust
for value in values
    .iter()

    // Retain even values.
    .filter(|x| *x % 2 == 0)

    // Stop after five matches.
    .take(5)
{
    even_sum_squares +=
        value * value;
}
```

---

## 🗺️ Using `map()`

The value of the item is never used directly, only in the `value * value` combination, which makes it an ideal target for a `map()`:

```rust
let mut even_sum_squares =
    0;

for val_sqr in values
    .iter()

    // Keep only even values.
    .filter(|x| *x % 2 == 0)

    // Process at most five.
    .take(5)

    // Transform each number into its square.
    .map(|x| x * x)
{
    // Accumulate each transformed value.
    even_sum_squares +=
        val_sqr;
}
```

Fully iterator-driven:

```rust
let even_sum_squares: i32 =
    values
        .iter()

        // Keep only even values.
        .filter(|x| **x % 2 == 0)

        // Limit the result.
        .take(5)

        // Square every remaining number.
        .map(|x| x * x)

        // Consume the iterator into a sum.
        .sum();
```

---

# 📦 Collection-Producing Methods

Other (more obscure) collection-producing methods include:

| Method | Purpose |
| --- | --- |
| `collect()` | Accumulates iterated items into a collection |
| `unzip()` | Splits pairs into two collections |
| `partition(p)` | Splits items by predicate |

### 📦 `collect()`

```rust
// Collect the range into a Vec.
let values: Vec<_> =
    (0..5).collect();
```

### ✂️ `partition()`

```rust
let values =
    vec![1, 2, 3, 4];

let (even, odd):
    (Vec<_>, Vec<_>) =
    values
        .into_iter()

        // Send matching values to the first Vec
        // and non-matching values to the second.
        .partition(
            |x| x % 2 == 0,
        );
```

---

# 📁 File System Operations

- `file()`: Reference to an open file on the system.
- `path`: Operations for inspecting paths.
- `fs`: Methods to manipulate contents of the local filesystem:
  - `read_to_string(&buffer)`
  - `copy(src, dst)`
  - `create_dir()`
  - `hard_link()`
  - `remove_dir()`
  - `remove_file(file_to_remove)`
  - `rename()`

### 📖 Read a File

```rust
use std::fs;

fn main()
    -> std::io::Result<()>
{
    // Read the entire UTF-8 file into a String.
    let contents =
        fs::read_to_string(
            "notes.txt",
        )?;

    println!("{contents}");

    // Report success.
    Ok(())
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — I/O boundaries and sans-I/O</strong></summary>

### 📥 Accept Standard I/O Traits

Instead of:

```rust
fn parse(
    // This unnecessarily restricts callers to files.
    file: std::fs::File,
) {
    // ...
}
```

prefer:

```rust
fn parse(
    // Accept anything implementing Read.
    input: impl std::io::Read,
) {
    // ...
}
```

Supported callers can include:

```text
File
TcpStream
stdin
&[u8]
UnixStream
custom readers
```

### 🔌 Sans-I/O

```text
┌──────────────────┐
│ Protocol Logic   │
└────────┬─────────┘
         │ bytes/events
         ▼
┌──────────────────┐
│ I/O Adapter      │
└────────┬─────────┘
         │
     ┌───┴────┐
     ▼        ▼
 Network     Tests
```

</details>

---

# 🧪 Code Snippets

## 🛡️ Static Enforcement

```rust
fn foo(a: Ascii) {
    // The type system guarantees `a` already satisfies
    // the invariants required by Ascii.
}
```

## 🚨 Dynamic Enforcement

```rust
fn foo(a: u8) {
    // Validate the primitive at runtime.
    if !a.is_valid() {
        panic!("invalid input");
    }

    // Continue only after validation.
}
```

## 🐛 Dynamic Enforcement with `debug_assert!`

```rust
fn foo(a: u8) {
    // Check the invariant in debug-oriented builds.
    debug_assert!(
        a.is_valid(),
    );

    // Continue under the assumption that `a` is valid.
}
```

---

# 🧠 Concepts

## 🛡️ Static Enforcement

This is the process of using types to rule out bad inputs. For example, using the type `Ascii` instead of `u8` to guarantee that the highest bit is zero. This is the preferred method of enforcing validity of input.

<details>
<summary><strong>🏢 Microsoft Guideline — Strong types guard invariants</strong></summary>

Avoid:

```rust
// Any u8, including invalid months, can be constructed.
pub struct Month(pub u8);
```

Prefer:

```rust
#[derive(
    Debug,
    Clone,
    Copy,
    PartialEq,
    Eq,
)]
pub struct Month(u8);

impl Month {
    pub fn try_new(
        value: u8,
    ) -> Result<Self, InvalidMonth> {
        // Validate the invariant exactly once.
        if (1..=12).contains(&value) {
            Ok(Self(value))
        } else {
            Err(InvalidMonth)
        }
    }
}
```

```text
External Input
      │
      ▼
Validate
      │
      ▼
 Strong Type
      │
      ▼
Rest of Program
```

</details>

---

## 🚨 Dynamic Enforcement

This is the process of validating input as it is processed, or ahead of time if necessary. This is often easier to implement than static enforcement, but has several drawbacks such as runtime overhead and delayed detection of bugs.

---

## 🧹 Destructors Never Fail

Destructors should not fail, as this will cause the program to abort. Instead, provide a separate method for checking for clean teardown, such as a `close()` method, that returns a `Result` to signal problems.

```rust
struct Connection;

impl Connection {
    fn close(
        self,
    ) -> Result<(), CloseError> {
        // Perform explicit fallible cleanup here,
        // before Drop performs final infallible teardown.
        Ok(())
    }
}
```

---

## ⏳ Destructors That May Block Have Alternatives

Destructors should not invoke blocking operations, as this can make debugging much more difficult. Consider providing a separate method for preparing for an infallible, nonblocking teardown.

---

# 🔒 Sealed Traits

Sealed traits protect against downstream implementations. Some traits are only meant to be implemented within the crate that defines them. In such cases, we can retain the ability to make changes to the trait in a non-breaking way by using the sealed trait pattern.

```rust
/// This trait is sealed and cannot be implemented
/// for types outside this crate.
pub trait TheTrait:
    private::Sealed
{
    // Public behavior available to callers.
    fn public_method(&self);

    #[doc(hidden)]
    fn private_hook(&self);
}

// Implement the public trait for supported types.
impl TheTrait for usize {
    fn public_method(&self) {
        // ...
    }

    fn private_hook(&self) {
        // ...
    }
}

mod private {
    // This trait cannot be named outside this crate.
    pub trait Sealed {}

    // Only explicitly supported types are sealed.
    impl Sealed for usize {}
}
```

---

# 🔐 Structs Have Private Fields

Making a field public is a strong commitment: it pins down a representation choice, and prevents the type from providing any validation or maintaining any invariants on the contents of the field, since clients can mutate it arbitrarily. Public fields are most appropriate for struct types in the C spirit: compound, passive data structures. Otherwise, consider providing getter/setter methods and hiding fields instead.

<details>
<summary><strong>🏢 Microsoft Guideline — Hide synchronization machinery</strong></summary>

Avoid:

```rust
pub struct Store {
    // Callers become coupled to synchronization details.
    pub data:
        Arc<Mutex<HashMap<Key, Value>>>,
}
```

Prefer:

```rust
pub struct Store {
    // Synchronization remains private.
    data:
        Arc<Mutex<HashMap<Key, Value>>>,
}

impl Store {
    pub fn get(
        &self,
        key: &Key,
    ) -> Option<Value> {
        // Public API describes the domain operation.
        todo!()
    }
}
```

</details>

---

# 🆕 Newtypes Encapsulate Implementation Details

A newtype can be used to hide representation details while making precise promises to the client.

```rust
use std::iter::{
    Enumerate,
    Skip,
};

// This exposes the exact iterator implementation type.
pub fn my_transform<I: Iterator>(
    input: I,
) -> Enumerate<Skip<I>> {
    input
        .skip(3)
        .enumerate()
}

// Hide the internal iterator composition behind a newtype.
pub struct MyTransformResult<I>(
    Enumerate<Skip<I>>,
);

impl<I: Iterator> Iterator
    for MyTransformResult<I>
{
    type Item =
        (usize, I::Item);

    fn next(
        &mut self,
    ) -> Option<Self::Item> {
        // Delegate iteration to the wrapped implementation.
        self.0.next()
    }
}

pub fn my_transform<I: Iterator>(
    input: I,
) -> MyTransformResult<I> {
    // Callers now depend on our stable wrapper,
    // not the exact internal iterator chain.
    MyTransformResult(
        input
            .skip(3)
            .enumerate(),
    )
}
```

---

# 🧬 Derived Trait Bounds

Generic data structures should not use trait bounds that can be derived or do not otherwise add semantic value.

```rust
// ✅ Prefer this:
// Trait bounds are introduced only where they are required.
#[derive(
    Clone,
    Debug,
    PartialEq,
)]
struct Good<T> {
    // ...
}

// ❌ Over this:
// These bounds unnecessarily constrain the entire type.
#[derive(
    Clone,
    Debug,
    PartialEq,
)]
struct Bad<
    T: Clone
        + Debug
        + PartialEq,
> {
    // ...
}
```

Duplicating derived traits as bounds on `Bad` is unnecessary and a backwards-compatibility hazard.

---

# ⭐ Most Important Concepts to Know

1. **Sealed Traits**: Sealed traits are traits that are only meant to be implemented within the crate that defines them. This allows for changes to be made to the trait in a non-breaking way. To avoid frustrated users trying to implement the trait, it should be documented that the trait is sealed and not meant to be implemented outside of the current crate.
2. **Private Fields**: Making a field public is a strong commitment and should only be done for struct types in the C spirit. Otherwise, consider providing getter/setter methods and hiding fields instead.
3. **Newtypes**: Newtypes can be used to hide representation details while making precise promises to the client. This allows for the representation to change in the future without breaking client code.
4. **Derived Trait Bounds**: Generic data structures should not use trait bounds that can be derived or do not otherwise add semantic value.

---

# 📐 Concepts to Know

## C-INTERMEDIATE

Functions should expose intermediate results to avoid duplicate work. This means that if a function computes interesting related data, it should be exposed in the API. For example, `Vec::binary_search` returns information about the index if found, and also the index at which the value would need to be inserted if not found.

```rust
fn foo(
    b: Bar,
) -> (bool, Option<usize>) {
    // Perform the main operation.
    let result =
        do_something(&b);

    // Preserve related useful information
    // instead of forcing callers to recompute it.
    let related_data =
        compute_related_data(&b);

    (result, related_data)
}
```

---

## C-CALLER-CONTROL

If a function requires ownership of an argument, it should take ownership of the argument rather than borrowing and cloning the argument.

### ✅ Prefer

```rust
fn foo(b: Bar) {
    // Ownership was intentionally transferred by the caller.
    use_owned_bar(b);
}
```

### ❌ Over

```rust
fn foo(b: &Bar) {
    // This forces an implicit allocation/copy policy on the caller.
    let b =
        b.clone();

    use_owned_bar(b);
}
```

---

## C-GENERIC

Functions should minimize assumptions about parameters by using generics.

```rust
fn foo<
    I: IntoIterator<Item = i64>,
>(
    iter: I,
) {
    // Accept any source that can produce i64 values.
    for item in iter {
        println!("{item}");
    }
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Builders and parameter design</strong></summary>

### 🏭 Builder Pattern

```rust
let client =
    Client::builder(dependencies)

        // Configure optional settings fluently.
        .endpoint(endpoint)
        .timeout(timeout)
        .retry_policy(policy)

        // Perform combined validation here.
        .build()?;
```

### 🧱 Cascade Complex Parameters

Avoid:

```rust
fn create_deposit(
    // Several primitives form larger domain concepts.
    bank: &str,
    customer: &str,
    currency: &str,
    amount: u64,
) {
    // ...
}
```

Prefer:

```rust
fn create_deposit(
    // Encapsulates bank + customer information.
    account: Account,

    // Encapsulates currency + numeric amount.
    amount: Money,
) {
    // ...
}
```

</details>

---

# 🧵 Types are `Send` and `Sync` where possible (C-SEND-SYNC)

`Send` and `Sync` are automatically implemented when the compiler determines it is appropriate. In types that manipulate raw pointers, be vigilant that the `Send` and `Sync` status of your type accurately reflects its thread safety characteristics.

```rust
#[test]
fn test_send() {
    fn assert_send<T: Send>() {
        // This function exists only for compile-time verification.
    }

    // Compilation proves MyStrangeType is Send.
    assert_send::<MyStrangeType>();
}

#[test]
fn test_sync() {
    fn assert_sync<T: Sync>() {
        // This function exists only for compile-time verification.
    }

    // Compilation proves MyStrangeType is Sync.
    assert_sync::<MyStrangeType>();
}
```

---

# ❗ Error types are meaningful and well-behaved (C-GOOD-ERR)

An error type is any type `E` used in a `Result<T, E>` returned by any public function of your crate. `Error` types should always implement the `std::error::Error` trait which is the mechanism by which error handling libraries like `error-chain` abstract over different types of errors, and which allows the error to be used as the `source()` of another error.

Additionally, error types should implement the `Send` and `Sync` traits. An `error` that is not `Send` cannot be returned by a `thread` run with `thread::spawn`. An `error` that is not `Sync` cannot be passed across threads using an `Arc`. These are common requirements for basic error handling in a multithreaded application.

`Send` and `Sync` are also important for being able to package a custom error into an IO error using `std::io::Error::new`, which requires a trait bound of `Error + Send + Sync`.

One place to be vigilant about this guideline is in functions that return Error trait objects, for example `reqwest::Error::get_ref`. Typically `Error + Send + Sync + 'static` will be the most useful for callers. The addition of `'static` allows the trait object to be used with `Error::downcast_ref`.

Never use `()` as an error type, even where there is no useful additional information for the error to carry. `()` does not implement `Error` so it cannot be used with error handling libraries like `error-chain`. `()` does not implement `Display` so a user would need to write an error message of their own if they want to fail because of the error. `()` has an unhelpful Debug representation for users that decide to `unwrap()` the `error`. It would not be semantically meaningful for a downstream library to implement `From<()>` for their error type, so `()` as an error type cannot be used with the `?` operator.

Instead, define a meaningful error type specific to your crate or to the individual function. Provide appropriate `Error` and `Display` impls. If there is no useful information for the error to carry, it can be implemented as a unit struct.

```rust
use std::error::Error;
use std::fmt::{
    self,
    Display,
};

// ❌ Avoid meaningless unit errors.
fn bad()
    -> Result<Wow, ()>
{
    todo!()
}

// ✅ Prefer a named domain-specific error.
fn do_the_thing()
    -> Result<Wow, DoError>
{
    todo!()
}

#[derive(Debug)]
struct DoError;

impl Display for DoError {
    fn fmt(
        &self,
        f: &mut fmt::Formatter<'_>,
    ) -> fmt::Result {
        // Error messages should normally be concise,
        // lowercase, and without trailing punctuation.
        f.write_str(
            "unable to do the thing",
        )
    }
}

impl Error for DoError {
    // No additional source error is required here.
}
```

The `error` message given by the `Display` representation of an `error` type should be lowercase without trailing punctuation, and typically concise. `Error::description()` should not be implemented. It has been deprecated and users should always use `Display` instead of `description()` to print the error.

Examples of error messages:

- `"unexpected end of file"`
- `"provided string was not 'true' or 'false'"`
- `"invalid IP address syntax"`
- `"second time provided was later than self"`
- `"invalid UTF-8 sequence of {} bytes from index {}"`
- `"environment variable was not valid unicode: {:?}"`

---

# 🔢 Binary number types provide `Hex`, `Octal`, `Binary` formatting (C-NUM-FMT)

- `std::fmt::UpperHex`
- `std::fmt::LowerHex`
- `std::fmt::Octal`
- `std::fmt::Binary`

These traits control the representation of a type under the `{:X}`, `{:x}`, `{:o}`, and `{:b}` `format!` specifiers. Implement these traits for any number type on which you would consider doing bitwise manipulations like `|` or `&`. This is especially appropriate for bitflag types. Numeric quantity types like:

```rust
// This represents a physical quantity,
// so alternate integer bases may not add much value.
struct Nanoseconds(u64);
```

probably do not need these.

If we were adding an error to represent an address failing to parse, for consistency we would want to name it in verb-object-error or verb-subject-error order like `ParseAddrError` rather than `AddrParseError`.

---

# 📥 Generic reader/writer functions take `R: Read` and `W: Write` by value (C-RW-VALUE)

The standard library contains these two impls:

```rust
impl<'a, R: Read + ?Sized>
    Read for &'a mut R
{
    // Standard library implementation.
}

impl<'a, W: Write + ?Sized>
    Write for &'a mut W
{
    // Standard library implementation.
}
```

That means any function that accepts `R: Read` or `W: Write` generic parameters by value can be called with a `&mut` reference if necessary. In the documentation of such functions, briefly remind users that a mut reference can be passed. New Rust users often struggle with this. They may have opened a file and want to read multiple pieces of data out of it, but the function to read one piece consumes the reader by value, so they are stuck. The solution would be to leverage one of the above impls and pass `&mut f` instead of `f` as the reader parameter.

---

# 🔠 Comparing and Sorting

- Process both strings from beginning to end as two sequences of maximal-length chunks, where each chunk consists either of a sequence of characters other than ASCII digits, or a sequence of ASCII digits (a numeric chunk), and compare corresponding chunks from the strings.
- To compare two numeric chunks, compare them by numeric value, ignoring leading zeroes. If the two chunks have equal numeric value, but different numbers of leading digits, and this is the first time this has happened for these strings, treat the chunks as equal (moving on to the next chunk) but remember which string had more leading zeroes.
- To compare two chunks if both are not numeric, compare them by Unicode character lexicographically, with two exceptions:
  - `_` (underscore) sorts immediately after (space) but before any other character. (This treats underscore as a word separator, as commonly used in identifiers.)
  - Unless otherwise specified, version-sorting should sort non-lowercase characters (characters that can start an UpperCamelCase identifier) before lowercase characters.
- If the comparison reaches the end of the string and considers each pair of chunks equal:
  - If one of the numeric comparisons noted the earliest point at which one string had more leading zeroes than the other, sort the string with more leading zeroes first.
  - Otherwise, the strings are equal.

---

# 🧮 Expressions

Prefer to use Rust's expression oriented nature where possible:

```rust
// ✅ Use Rust's expression-oriented design.
let x =
    if y { 1 } else { 0 };

// ❌ More verbose imperative equivalent.
let x;

if y {
    x = 1;
} else {
    x = 0;
}
```

Avoid `#[path]` annotations where possible. Prefer to use multiple imports rather than a multi-line import. However, tools should not split imports by default. In general, within expressions, prefer dereferencing to taking references, unless necessary (e.g. to avoid an unnecessarily expensive operation). Do include extraneous parentheses if it makes an arithmetic or logic expression easier to understand `((x * 15) + (y * 20)` is fine). Prefer using a unit struct (e.g., `struct Foo;`) to an empty struct (e.g., `struct Foo();` or `struct Foo {}`, these only exist to simplify code generation), but if you must use an empty struct, keep it on one line with no space between the braces: `struct Foo;` or `struct Foo {}`. For more than a few fields (in particular if the tuple struct does not fit on one line), prefer a proper struct with named fields. The same guidelines are used for untagged union declarations.

```rust
union Foo {
    // Each union field shares the same memory.
    a: A,
    b: B,
    long_name: LongType,
}
```

---

# 🎨 When deciding on style guidelines, use these guiding principles (in rough priority order)

| Priority | Principle |
| --- | --- |
| 🥇 | **readability** — scan-ability |
| 🥈 | **aesthetics** — consistent with other languages/tools |
| 🥉 | **specifics** — preventing rightward drift |
| 4️⃣ | **application** — ease of manual application |

---

# 🔤 Sorting

- As the last member of a delimited expression, delimited expressions are generally combinable, regardless of the number of members. Previously only applied with exactly one member (except for closures with explicit blocks).
- When line-breaking a binary operator, if the first operand spans multiple lines, use the base indentation of the last line.
- Use version-sort (sort `x8`, `x16`, `x32`, `x64`, `x128` in that order).
- Change "ASCIIbetical" sort to Unicode-aware "non-lowercase before lowercase".
- Format single associated type `where` clauses on the same line if they fit.

```text
x8
x16
x32
x64
x128
```

---

# 🎨 Formatting

When a name is forbidden because it is a reserved word (such as `crate`), either use a raw identifier (`r#crate`) or use a trailing underscore (`crate_`). Don't misspell the word (`krate`). Prefer to use single-letter names for generic parameters.

When writing extern items (such as `extern "C" fn`), always specify the ABI.

```rust
// ✅ Explicitly state the ABI.
extern "C" fn foo() {
    // ...
}
```

A group of imports is a set of imports on the same or sequential lines. One or more blank lines or other items (e.g., a function) separate groups of imports. Within a group of imports, imports must be version-sorted. Groups of imports must not be merged or re-ordered.

<details>
<summary><strong>🏢 Microsoft Guideline — Naming</strong></summary>

Prefer:

```text
AppConfig
BookingStore
BookingDispatcher
ClientBuilder
CallbackFn
```

over vague names such as:

```text
GlobalApplicationConfiguration
BookingManager
BookingService
ClientFactory
CallbackFunction
```

Words worth scrutinizing:

```text
Manager
Service
Helper
Utility
Processor
Factory
Handler
```

</details>

---

# 📥 Ordering list import

Names in a list import must be version-sorted, except that:

- `self` and `super` always come first if present, and
- groups and glob imports always come last if present.

This applies recursively.

```rust
use foo::bar::{
    // Simple imports first.
    a,

    // Nested paths follow version sorting.
    b::c,
    b::d,

    // Nested groups remain grouped.
    b::d::{x, y, z},

    // `self` comes first inside this nested group.
    b::{self, r, s},
};
```

---

# 🧹 Normalisation

Tools must make the following normalisations, recursively:

```text
use a::self;  → use a;

use a::{};    → (nothing)

use a::{b};   → use a::b;
```

Tools must not otherwise merge or un-merge import lists or adjust glob imports (without an explicit option). Each nested import must be on its own line, but non-nested imports must be grouped on as few lines as possible.

---

# ⛓️ Chains

A chain is a sequence of field accesses, method calls, and/or uses of the try operator `?`.

Examples:

```rust
// Small chains can remain on one line.
a.b.c().d
```

```rust
// The try operator may also appear inside chains.
foo?.bar().baz?
```

For longer chains:

```rust
let result = value
    // Borrow items.
    .iter()

    // Keep valid values.
    .filter(
        |value| value.is_valid(),
    )

    // Transform each value.
    .map(transform)

    // Materialize the result.
    .collect::<Vec<_>>();
```

---

# ➡️ Semicolons

Use a semicolon where an expression has void type, even if it could be propagated.

```rust
fn foo() {
    // ...
}

fn bar() {
    // The returned unit value is intentionally discarded.
    foo();
}
```

---

# 📏 Indentation and Line Width

| Rule | Value |
| --- | --- |
| **Indentation** | spaces |
| **Indent width** | 4 spaces |
| **Maximum line width** | 100 characters |

- Use **spaces**, not *tabs*.
- Each level of indentation must be **4 spaces**.
- The *maximum width* for a line is **100 characters**.

---

# 🧱 Block vs Visual Indentation

### ✅ Block Indent

```rust
a_function_call(
    // Arguments align by indentation level.
    foo,
    bar,
);
```

### ❌ Visual Indent

```rust
a_function_call(
    foo,
    bar,
);
// Avoid alignment schemes that depend on the function-name length.
```

---

# 🧱 Tuple Structs

```rust
pub struct Foo(
    // One tuple field per line.
    String,

    // Trailing comma keeps diffs stable.
    u8,
);
```

---

# 🧩 Traits

```rust
// Empty traits can remain on one line.
trait Foo {}

pub trait Bar {
    // Non-empty traits use normal block indentation.
    fn bar(&self);
}
```

```rust
// Put spaces around `+`.
trait Foo:
    Debug + Bar
{
}
```

---

# 🧠 Let-Else

Format the entire let-else statement on one line when short:

```rust
// Return immediately unless the Option contains exactly 1.
let Some(1) = opt else {
    return;
};
```

---

# ➕ Breaking `+` Bounds

```rust
impl Clone
    + Copy
    + Debug
```

```rust
Box<
    Clone
    + Copy
    + Debug
>
```

---

# ➕ Trailing Commas

In comma-separated lists of any kind, use a trailing comma when followed by a newline:

```rust
function_call(
    // A trailing comma keeps future edits small.
    argument,
    another_argument,
);

let array = [
    // Every multiline item gets a trailing comma.
    element,
    another_element,
    yet_another_element,
];
```

This makes moving code easier and makes diffs smaller.

---

# ⬜ Blank Lines

Separate items and statements by either zero or one blank lines.

```rust
fn foo() {
    // Related statements remain together.
    let x = ...;
    let y = ...;
    let z = ...;
}

// Separate top-level items with a blank line.
fn bar() {}

fn baz() {}
```

If a line break is required in a non-inherent `impl`:

```rust
impl Bar
    // Break immediately before `for`.
    for Foo
{
    // ...
}
```

---

# 🪄 Macros

Use `{}` for the full definition of the macro.

```rust
macro_rules! foo {
    // Macro rules go here.
}
```

Prefer to put a generics clause on one line. Break other parts of an item declaration rather than line-breaking a generics clause. If a generics clause is large enough to require line-breaking, prefer a `where` clause instead.

Do not put spaces before or after `<` nor before `>`. Only put a space after `>` if it is followed by a word or opening brace, not an opening parenthesis. Put a space after each comma. Do not use a trailing comma for a single-line generics clause.

<details>
<summary><strong>🏢 Microsoft Guideline — Macro design</strong></summary>

```text
Can normal Rust solve it?
│
├── Yes
│   └── use normal Rust
│
└── No
    │
    ├── macro_rules! sufficient?
    │   └── use macro_rules!
    │
    └── proc macro genuinely needed?
        └── proc macro
```

Prefer:

```rust
// Simple declarative code generation.
make_new_id!(
    UserId,
);
```

### 🧼 Proc-Macro Architecture

```text
foo
├── foo_proc
│   └── thin proc-macro entry point
└── foo_proc_impl
    ├── transformation logic
    └── ordinary unit tests
```

</details>

---

# 💬 Comments

Prefer line comments (`//`) to block comments (`/* ... */`).

When using line comments, put a single space after the opening sigil.

Comments should usually be complete sentences. Start with a capital letter, end with a period (`.`). An inline block comment may be treated as a note without punctuation.

Source lines which are entirely a comment should be limited to 80 characters in length (including comment sigils, but excluding indentation) or the maximum width of the line (including comment sigils and indentation), whichever is smaller.

```rust
// Parse the input before processing it.
let value =
    parse(input);
```

---

# 📖 Doc Comments

Prefer line comments (`///`) to block comments (`/** ... */`).

Prefer outer doc comments (`///` or `/** ... */`), only use inner doc comments (`//!` and `/*! ... */`) to write module-level or crate-level documentation.

Put doc comments before attributes.

For attributes with argument lists, format like functions.

```rust
/// Adds two integers.
///
/// # Examples
///
/// ```
/// // The result should be the arithmetic sum.
/// assert_eq!(add(2, 3), 5);
/// ```
pub fn add(
    a: i32,
    b: i32,
) -> i32 {
    // Return the sum as an expression.
    a + b
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Rustdoc structure</strong></summary>

```rust
/// Short summary sentence.
///
/// Extended description.
///
/// # Examples
///
/// ```
/// // Demonstrate normal usage.
/// ```
///
/// # Errors
///
/// Explain known error conditions.
///
/// # Panics
///
/// Explain panic conditions.
///
/// # Safety
///
/// Explain caller obligations for unsafe APIs.
///
/// # Abort
///
/// Explain conditions that may abort the process.
pub fn operation() {
    // ...
}
```

</details>

---

# 📦 Items

- **Items** are exportable pieces of source code: structures, functions, constants, etc. Structures, Rust's class-like abstraction, are arguably our most fundamental organization tool. The top of the program organization hierarchy.
  - A full list of language constructs considered items is available. Technically, modules are items. But for the purpose of our current code organization discussion, we'll consider them taxonomically distinct.

```rust
// Struct item.
struct User {
    name: String,
}

// Function item.
fn run() {
    // ...
}

// Constant item.
const MAX_CONNECTIONS:
    usize = 100;
```

---

# 📂 Modules

- **Modules** group related items into cohesive units. They facilitate organizing code within a project, much like namespaces.
  - Some programmers like to follow a "one module per source file" convention. But that 1:1 mapping is entirely optional. Modules are a logical, hierarchical grouping. They're not decided by the layout of a filesystem.

```rust
mod networking {
    pub fn connect() {
        // Networking-specific implementation.
    }
}
```

<details>
<summary><strong>🏢 Microsoft Guideline — Module organization</strong></summary>

```rust
pub mod storage {
    //! Persistent storage functionality.
    //!
    //! Contains connection, transaction,
    //! and query-related abstractions.
}
```

Prefer domain modules such as:

```text
auth
client
storage
request
response
protocol
```

over catch-all modules such as:

```text
util
helpers
models
misc
```

</details>

---

# 📦 Crates

- **Crates** group one or more related modules into either a library or a binary. They facilitate organizing code between projects. For libraries, visibility modifiers decide which items the module(s) export (e.g. the public API of the crate).
  - Crates can also have dependencies, which are themselves crates (e.g. 3rd party libraries used internally). Chapter 2's `rcli` tool was a binary crate that had two library crate dependencies: `rc4` and `clap`.

<details>
<summary><strong>🏢 Microsoft Guideline — Workspaces and MSRV</strong></summary>

### 🗂️ Workspace Layout

```text
project/
├── Cargo.toml
└── crates/
    ├── core/
    ├── storage/
    ├── protocol/
    └── cli/
```

### 📦 Root `Cargo.toml`

```toml
[workspace]

# Keep related crates in one workspace.
members = [
    "crates/core",
    "crates/storage",
    "crates/protocol",
    "crates/cli",
]

[workspace.dependencies]

# Centralize dependency versions and feature policy.
serde = {
    version = "1",
    default-features = false,
}

tracing = "0.1"
```

Member crate:

```toml
[dependencies]

# Reuse the workspace dependency declaration.
serde.workspace = true

# Reuse the same tracing version everywhere.
tracing.workspace = true
```

### 🦀 MSRV

```toml
[package]

# Explicitly state the minimum supported compiler version.
rust-version = "1.xx"
```

</details>

---

# 🌐 System

- **System** is the general term for a large piece of software made up of interconnected components. That could mean multiple Rust crates, libraries written in other programming languages that interoperate via CFFI, or even networked sub-services that communicate using structured formats like REST and gRPC.

```text
System
│
├── Rust API Crate
├── Rust Database Crate
├── C Library via FFI
├── Worker Service
└── REST / gRPC Services
```

<details>
<summary><strong>🏢 Microsoft Guideline — FFI and portability</strong></summary>

```text
┌──────────────────────┐
│      Rust Core       │
│ safe / idiomatic API │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│      FFI Layer       │
│ translation boundary │
└──────────┬───────────┘
           │
           ▼
      C / OS / DLL
```

### 🔓 Native Escape Hatch

```rust
pub struct Handle(
    RawHandle,
);

impl Handle {
    /// # Safety
    ///
    /// `raw` must be a valid owned native handle.
    pub unsafe fn from_raw(
        raw: RawHandle,
    ) -> Self {
        // Caller guarantees the native ownership invariant.
        Self(raw)
    }
}
```

### 🌍 Platform Isolation

```text
platform/
├── linux.rs
├── windows.rs
├── macos.rs
└── fallback.rs
```

</details>

---

# ☢️ Additional Safety Notes

> [!IMPORTANT]
> `unsafe` has a narrow technical meaning: the programmer is taking responsibility for memory-safety requirements the compiler cannot prove.

Appropriate categories include:

1. low-level abstractions
2. measured performance optimizations
3. FFI/platform interaction

Do not use `unsafe` merely because an operation is destructive or dangerous.

```rust
fn delete_database() {
    // This operation may be dangerous,
    // but it does not inherently violate memory safety.
}
```

### 🛡️ Safety Comments

```rust
unsafe {
    // SAFETY:
    // `ptr` originates from this live allocation,
    // and `index` was validated against the allocation length.
    *ptr.add(index)
}
```

Do not use `unsafe` merely to circumvent:

- `Send`
- `Sync`
- lifetimes
- aliasing
- type-system restrictions

> [!CAUTION]
> A safe API must not permit undefined behavior from any valid safe caller.

---

# 🚀 Additional Performance Notes

> [!TIP]
> Optimize **measured hot paths**, not hypothetical ones.

Watch for:

- repeated allocation
- unnecessary cloning
- collection reallocation
- repeated hashing
- pointer chasing
- lock contention
- unnecessarily large futures
- excessive task switching

### 🧅 Pointer Indirection

```text
Arc<A>
  ↓
Arc<B>
  ↓
Arc<C>
  ↓
value
```

Multiple dependent memory accesses can reduce cache efficiency.

### 📦 Immutable Owned Data

```rust
// Immutable owned UTF-8 string.
let text: Box<str> =
    String::from("Rust")
        .into_boxed_str();

// Immutable owned sequence.
let values: Box<[i32]> =
    vec![1, 2, 3]
        .into_boxed_slice();
```

---

# 🧠 Final Mental Models

## 🔐 Ownership

```text
Own it
│
├── Need another owner?
│   ├── single-threaded → Rc<T>
│   └── multi-threaded  → Arc<T>
│
├── Need mutation?
│   ├── exclusive       → &mut T
│   ├── runtime checked → RefCell<T>
│   └── synchronized    → Mutex<T> / RwLock<T>
│
└── Need heap allocation?
    └── Box<T>
```

## ❗ Error Handling

```text
Can it fail?
│
├── No
│
└── Yes
    │
    ├── Missing value only
    │   └── Option<T>
    │
    └── Need failure reason
        └── Result<T, E>
```

## 🔁 Iterator Selection

```text
transform values      → map()
remove values         → filter()
limit values          → take()
skip values           → skip()
combine iterators     → chain()
pair iterators        → zip()
add indexes           → enumerate()
find one              → find()
test any              → any()
test all              → all()
aggregate             → fold()
sum                   → sum()
build collection      → collect()
```

## 🧵 Concurrency Selection

```text
Need concurrency?
│
├── Pass ownership between threads
│   └── channel / mpsc
│
├── Shared immutable state
│   └── Arc<T>
│
├── Shared mutable state
│   ├── Mutex<T>
│   └── RwLock<T>
│
└── Async I/O
    └── Future + runtime
```

---

<div align="center">

# 🦀 Rust Development Loop

### `cargo fmt` → `cargo check` → `cargo clippy` → `cargo test` → `cargo bench`

**Correctness → Clarity → Testability → Measurement → Optimization**

</div>
