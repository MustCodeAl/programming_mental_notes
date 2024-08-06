# Rust Notes

## Resources

- [The Rust Style Guide](https://doc.rust-lang.org/nightly/style-guide/index.html)
- [API Guidelines](https://rust-lang.github.io/api-guidelines/checklist.html)
- [Effective Rust](https://www.lurklurk.org/effective-rust/)
- [Building Linked List](https://rust-unofficial.github.io/too-many-lists/index.html)
- [Design Patterns](https://rust-unofficial.github.io/patterns/intro.html)
- [Breakdown Rust Notes](https://www.breakdown-notes.com/make/load/rust_cs_canvas)
- [cheats.rs](https://cheats.rs/)
- [Blessed.rs](https://blessed.rs/crates)

## Ownership and Borrowing

- The owner of an item gets to create it, read from it, update it, and drop it (CRUD).
- Borrowing does not have ownership, so the value won't be dropped when the borrow goes out of scope.
- If you mutable borrow a value, the owner can't change it until you're done borrowing it.
- **Only the item's owner can move the item.**
- **There can be any number of immutable references to the item, or a single mutable reference to the item but not both.**
- The data pointed to at the location has to be valid, and any assignments to this location have to be of valid data, enforced by the reference to have existing data at all times.
- It's **valid to read from a mutable reference, and it's valid to write to a mutable reference, and so the ability to do both at once is provided by the `std::mem::replace`.**

## Memory Management

- `isize` is the size of a pointer on your system.
- While a program is running, the memory it uses is divided into different chunks, sometimes called segments. Some of these chunks are a fixed size, such as the ones that hold the program code or the program's global data, but two of the chunks – the heap and the stack – change size as the program runs.
- The stack is used to hold state related to the currently executing function, specifically its parameters, local variables, and temporary values, held in a stack frame.
- Rust has two built-in fat pointer types: types that act as pointers but hold additional information about the thing they are pointing to.
  - **Slice:** A reference to a subset of some contiguous collection of values.
  - **Trait Object:** A reference to some item that implements a particular trait.

## Traits and Generics

- The simplest is the `Pointer` trait, which formats a pointer value for output.
- The `Borrow` and `BorrowMut` traits each have a single method (`borrow` and `borrow_mut` respectively) that has the same signature as the equivalent `AsRef` / `AsMut` trait methods.
- A function that accepts `Borrow` can receive either items or references-to-items and can work with references in either case.
- A function that accepts `ToOwned` can receive either items or references-to-items and can build its own personal copies of those items in either case.
- `AsRef` is for cheap reference to reference conversions.
- Rust allocates items on the stack by default; the `Box<T>` pointer type forces allocation to occur on the heap, which means that the allocated item can outlive the scope of the current block.
- `Rc` and `Weak<T>` types are used for reference counting and non-owning references, respectively.

## Lifetimes

- The lifetime of an item on the stack is the period where that item is guaranteed to stay in the same place.
- The scope of any reference must be smaller than the lifetime of the item that it refers to.
- **Lifetime extension:** Convert a temporary to be a new named local variable with a `let` binding.
- **Lifetime reduction:** Add an additional block `{ ... }` around the use of a reference so that its lifetime ends at the end of the new block.
- Precisely where an item gets automatically dropped depends on whether an item has a name or not.
- The Rust compiler guarantees that a `'static` item always has the same address for the entire duration of the program and never moves.

## Concurrency

- A data race occurs when two distinct threads access the same memory location, where at least one of them is a write, and there is no synchronization mechanism that enforces an ordering on the accesses.
- Use `Arc`, `Mutex`, and `RwLock` to ensure thread safety.
- Avoid lock inversion and keep lock scopes small and obvious.
- Include deadlock detection tools in your CI system.

## Miscellaneous

- `Copy` is default for variables stored on the stack.
- Move is default for heap variables.
- Structs are stored on the heap and move by default in Rust.
- Ad-hoc conversions follow `as_`, `to_`, `into_` conventions.
- Get used to the transformations of `Option` and `Result`, and prefer `Result` to `Option`.
- `enum` are useful for writing functions that take multiple types and for vecs which allow any enum type.
- String methods: `trim()`, `trim_end_matches(w)`, `strip_prefix(p)`, `lines()`, `chars()`, `char_indices()`, `split_whitespace()`, `bytes()`, and `split_at(index)`.
- Vec methods: `extend()` and `append()`.

## Error Handling

- Rust uses `Result` and `Option` types for error handling.
- `Result<T, E>` is used for functions that can return an error.
- `Option<T>` is used for values that may or may not be present.
- Use `unwrap` and `expect` cautiously; prefer `match` or combinators like `map`, `and_then`, etc.
- Use `?` operator to propagate errors.

## Testing

- Use `#[test]` attribute to mark test functions.
- Use `assert!`, `assert_eq!`, and `assert_ne!` macros for assertions.
- Run tests with `cargo test`.
- Use `#[cfg(test)]` to include test code only when running tests.

## Common Commands

- `cargo new project_name` - Create a new project.
- `cargo build` - Compile the project.
- `cargo run` - Build and run the project.
- `cargo test` - Run tests.
- `cargo doc --open` - Generate and open documentation.


## Best Practices

- Write idiomatic Rust code by following the Rust Style Guide.
- Use `clippy` to catch common mistakes and improve code quality.
- Keep functions small and focused.
- Prefer immutability and pure functions.
- Document your code with comments and `///` doc comments.
- Write tests for your code to ensure correctness.
- Use `cargo fmt` to format your code.

## Code Snippets

```rust
pub struct Ref<'a, T: 'a>(&'a T);
```

This generic data structure holds an explicit reference `&'a T`, as per the first bullet above. But the type `T` might itself contain references with some lifetime `'b`, as per the second bullet above. If `T`'s inherent lifetime `'b` were smaller than the exterior lifetime `'a` we'd have a potential disaster: the `Ref` would be holding a reference to a data structure whose own references have gone bad.

To prevent this, we need `'b` to be larger than `'a`.

One common place this shows up is when you try to move values between threads with `std::thread::spawn`. The moved values need to be of types that implement `Send`, indicating that they're safe to move between threads, but they also need to not contain any dynamic references (the `'static` lifetime bound). This makes sense when you realize that a reference to something on the stack now raises the question: which stack? Each thread's stack is independent, and so lifetimes can't be tracked between them.

---

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

- `Future` has to be `poll`ed (by the executor) to resume where it last yielded and make progress (async is lazy).

## Conclusion

Rust is a powerful and safe systems programming language that enforces memory safety and concurrency without a garbage collector. By understanding and utilizing Rust's ownership model, lifetimes, and concurrency primitives, you can write efficient and reliable code. Use the resources and best practices outlined in this guide to deepen your knowledge and improve your Rust programming skills.


---

### Futures and Async in Rust

- `Future` has to be `poll`ed (by the executor) to resume where it last yielded and make progress (async is lazy).
- `&mut Self` contains state (state machine).
- `Pin` the memory location because the future contains self-referential data.
- `Context` contains the Waker to notify the executor that progress can be made.
- `async`/`await` on futures is implemented by generators.
- `async fn` and `async` blocks return `impl Future<Output = T>`.
- Calling `.await` attempts to resolve the `Future`: if the `Future` is blocked, it yields control; if progress can be made, the `Future` resumes.

Futures form a tree of futures. The leaf futures communicate with the executor. The root future of a tree is called a **task**.

### Async Runtimes

- **tokio** (multithreaded): Thread pool with work-stealing scheduler: each processor maintains its own run queue; idle processor checks sibling processor run queues and attempts to steal tasks from them.
- **actix_rt**: Single-threaded async runtime; futures are `!Send`.
- **actix-web**: Constructs an application instance for each thread; application data must be constructed multiple times or shared between threads.

### Streams

`Stream<Item = T>` is an asynchronous version of `Iterator<Item = T>`, i.e., it does not block between each item yield.

#### Create:
- `futures::stream::iter`
- `futures::stream::once`
- `futures::stream::repeat`
- `futures::stream::repeat_with`
- `async_stream::stream`

#### Combine (n-1):
- `futures::stream::StreamExt::chain`
- `futures::stream::StreamExt::zip`
- `tokio_stream::StreamExt::merge`
- `tokio_stream::StreamMap`
- `tokio::select`

#### Split operations (1-n):
- `futures::channel::oneshot::Sender::send`
- `async_std::channel::Sender::send`

### Data Races

A data race occurs when two distinct threads access the same memory location, where at least one of them is a write, and there is no synchronization mechanism that enforces an ordering on the accesses.

#### Avoiding Data Races:
- Ensure a single writer or multiple readers (but never both).
- Use `std::sync::mpsc` for message passing.
- Use `Arc`, `Mutex`, and `RwLock` for shared-state concurrency.
- Avoid lock inversion and keep lock scopes small and obvious.
- Use deadlock detection tools like `no_deadlocks`, `ThreadSanitizer`, or `parking_lot::deadlock`.

### Conversions

Ad-hoc conversions follow `as_`, `to_`, `into_` conventions (C-CONV):

| Prefix | Cost | Ownership |
| ------ | ---- | --------- |
| `as_` | Free | borrowed -> borrowed |
| `to_` | Expensive | borrowed -> borrowed<br>borrowed -> owned (non-Copy types)<br>owned -> owned (Copy types) |
| `into_` | Variable | owned -> owned (non-Copy types) |

- Conversions prefixed `as_` and `into_` typically decrease abstraction.
- Conversions prefixed `to_` typically stay at the same level of abstraction but do some work to change from one representation to another.

### Option and Result

- Get used to the transformations of `Option` and `Result`, and prefer `Result` to `Option`.
- Use `.as_ref()` as needed when transformations involve references.
- Use them in preference to explicit `match` operations.
- Use them to transform result types into a form where the `?` operator applies.

### Pinning and Self-Referential Data Structures

- Use `Pin` to ensure that internal self-references remain valid.
- Avoid self-referential data structures or use library crates that encapsulate the difficulties (e.g., ouroborous).

### Enums

- Enums allow writing functions that take multiple types and for vecs which allow any enum type.
- The size of an enum will not exceed the largest variant and needed tag.

### String Methods

- `trim()` removes whitespace.
- `trim_end_matches(w)` removes occurrences of `w`.
- `strip_prefix(p)` removes `p` at most once.
- Split strings by `lines()`, `chars()`, `char_indices()`, `split_whitespace()`, `bytes()`, and `split_at(index)`.

### Vec Methods

- `extend()` appends to a vector by cloning contents of the second vector.
- `append()` appends by move.
- `concat()` adds two containers together.
- `join()` adds the two by a separator.
- `starts_with()`, `ends_with()`, `capacity()`, `length()`, `contains()`.
- Use `get(index)` to get options back instead of indexing collections directly.

### Power Functions

- `pow()` for power.
- `powf()` for power float.

### Option Methods

- `take()` replaces option with `None` and returns a new option with the value.
- `ok_or` turns `Some(valid_value)` into `Result::Ok(valid_value)` and `None` into `Result::Err(err_value)`.

### Hashing

- Implement `Eq` and `PartialEq` for hashing.

### Conversion Traits

- `From<T>`: Items of this type can be built from items of type `T`.
- `TryFrom<T>`: Items of this type can sometimes be built from items of type `T`.
- `Into<T>`: Items of this type can be converted into items of type `T`.
- `TryInto<T>`: Items of this type can sometimes be converted into items of type `T`.

### Iterators

#### Consumers:
- `iter()` and `fold()`
- `scan()`
- `any(P)`
- `all(p)`

#### Shortcuts:
- `sum()`
- `product()`
- `min()`, `max()`
- `min_by(f)`, `max_by(f)`
- `reduce(f)`
- `fold(f)`
- `scan(f)`

#### Single Value Selection:
- `find(p)`
- `position(p)`
- `nth(n)`

#### Testing:
- `take(n)`
- `skip(n)`
- `step_by(n)`
- `chain(other)`
- `cycle()`
- `rev()`

#### Item Transformations:
- `map(|item| {...})`
- `cloned()`
- `copied()`
- `enumerate()`
- `zip(it)`

#### Filtering:
- `filter(|item| {...})`
- `take_while()`
- `skip_while()`
- `flatten()`

---

## Adapters and Iterators

Adapters are functions that take iterators and return other iterators.

### Replacing Vector Indexing with Iterators

The first step is to replace vector indexing with direct use of an iterator in a for-each loop:

```rust
for i in 0..values.len() {
    if values[i] % 2 != 0 {
        continue;
    }
}
```

to 

```rust
for value in values.iter() {
    if value % 2 != 0 {
        continue;
    }
}
```

### Using `filter()`

An initial arm of the loop that uses `continue` to skip over some items is naturally expressed as a `filter()`:

```rust
for value in values.iter().filter(|x| *x % 2 == 0) {
}
```

### Using `take()`

Next, the early exit from the loop once 5 even items have been spotted maps to a `take(5)`:

```rust
for value in values.iter().filter(|x| *x % 2 == 0).take(5) {
    even_sum_squares += value * value;
}
```

### Using `map()`

The value of the item is never used directly, only in the `value * value` combination, which makes it an ideal target for a `map()`:

```rust   
let mut even_sum_squares = 0;
for val_sqr in values.iter().filter(|x| *x % 2 == 0).take(5).map(|x| x * x) {
    even_sum_squares += val_sqr;
}
```

### Collection-Producing Methods

Other (more obscure) collection-producing methods include:

- `collect()`: Accumulates all of the iterated items into a new collection.
- `unzip()`: Divides an iterator of pairs into two collections.
- `partition(p)`: Splits an iterator into two collections based on a predicate that is applied to each item.

## File System Operations

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

## Code Snippets

### Static Enforcement

```rust
fn foo(a: Ascii) { /* ... */ }
```

### Dynamic Enforcement

```rust
fn foo(a: u8) {
    if !a.is_valid() {
        panic!("invalid input");
    }
    // ...
}
```

### Dynamic Enforcement with `debug_assert!`

```rust
fn foo(a: u8) {
    debug_assert!(a.is_valid());
    // ...
}
```

## Concepts

### Static Enforcement

This is the process of using types to rule out bad inputs. For example, using the type `Ascii` instead of `u8` to guarantee that the highest bit is zero. This is the preferred method of enforcing validity of input.

### Dynamic Enforcement

This is the process of validating input as it is processed, or ahead of time if necessary. This is often easier to implement than static enforcement, but has several drawbacks such as runtime overhead and delayed detection of bugs.

### Destructors Never Fail

Destructors should not fail, as this will cause the program to abort. Instead, provide a separate method for checking for clean teardown, such as a `close()` method, that returns a `Result` to signal problems.

### Destructors That May Block Have Alternatives

Destructors should not invoke blocking operations, as this can make debugging much more difficult. Consider providing a separate method for preparing for an infallible, nonblocking teardown.

### Sealed Traits

Sealed traits protect against downstream implementations. Some traits are only meant to be implemented within the crate that defines them. In such cases, we can retain the ability to make changes to the trait in a non-breaking way by using the sealed trait pattern.

```rust
/// This trait is sealed and cannot be implemented for types outside this crate.
pub trait TheTrait: private::Sealed {
    // Zero or more methods that the user is allowed to call.
    fn ...();
    // Zero or more private methods, not allowed for user to call.
    #[doc(hidden)]
    fn ...();
}
// Implement for some types.
impl TheTrait for usize {
    /* ... */
}
mod private {
    pub trait Sealed {}
    // Implement for those same types, but no others.
    impl Sealed for usize {}
}
```

### Structs Have Private Fields

Making a field public is a strong commitment: it pins down a representation choice, and prevents the type from providing any validation or maintaining any invariants on the contents of the field, since clients can mutate it arbitrarily. Public fields are most appropriate for struct types in the C spirit: compound, passive data structures. Otherwise, consider providing getter/setter methods and hiding fields instead.

### Newtypes Encapsulate Implementation Details

A newtype can be used to hide representation details while making precise promises to the client.

```rust
use std::iter::{Enumerate, Skip};

pub fn my_transform<I: Iterator>(input: I) -> Enumerate<Skip<I>> {
    input.skip(3).enumerate()
}

pub struct MyTransformResult<I>(Enumerate<Skip<I>>);

impl<I: Iterator> Iterator for MyTransformResult<I> {
    type Item = (usize, I::Item);
    fn next(&mut self) -> Option<Self::Item> {
        self.0.next()
    }
}

pub fn my_transform<I: Iterator>(input: I) -> MyTransformResult<I> {
    MyTransformResult(input.skip(3).enumerate())
}
```

### Derived Trait Bounds

Generic data structures should not use trait bounds that can be derived or do not otherwise add semantic value.

```rust
// Prefer this:
#[derive(Clone, Debug, PartialEq)]
struct Good<T> { /* ... */ }

// Over this:
#[derive(Clone, Debug, PartialEq)]
struct Bad<T: Clone + Debug + PartialEq> { /* ... */ }
```

Duplicating derived traits as bounds on `Bad` is unnecessary and a backwards-compatibility hazard.

### Most Important Concepts to Know

1. **Sealed Traits**: Sealed traits are traits that are only meant to be implemented within the crate that defines them. This allows for changes to be made to the trait in a non-breaking way. To avoid frustrated users trying to implement the trait, it should be documented that the trait is sealed and not meant to be implemented outside of the current crate.
2. **Private Fields**: Making a field public is a strong commitment and should only be done for struct types in the C spirit. Otherwise, consider providing getter/setter methods and hiding fields instead.
3. **Newtypes**: Newtypes can be used to hide representation details while making precise promises to the client. This allows for the representation to change in the future without breaking client code.
4. **Derived Trait Bounds**: Generic data structures should not use trait bounds that can be derived or do not otherwise add semantic value.



