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

# Rust Ownership and Memory Management

## Ownership and Borrowing

- **CRUD Operations**: The owner of an item can create, read, update, and drop it.
- **Borrowing**: Borrowing does not have ownership, so the value won't be dropped when the borrow goes out of scope.
- **Mutable Borrowing**: If you mutable borrow a value, the owner can't change it until you're done borrowing it.
- **Ownership Movement**: Only the item's owner can move the item.
- **References**: There can be any number of immutable references to the item, or a single mutable reference to the item, but not both.
- **Data Validity**: The data pointed to must be valid, and any assignments to this location must be of valid data, enforced by the reference to have existing data at all times.
- **Mutable References**: It's valid to read from a mutable reference and write to it. The ability to do both at once is provided by `std::mem::replace`.

## Memory Segments

- **Memory Segments**: While a program is running, memory is divided into different chunks (segments). Some are fixed size (program code, global data), while others (heap and stack) change size as the program runs.
- **Heap and Stack**: Typically arranged at opposite ends of the program's virtual memory space, so one can grow downwards and the other upwards.

## Stack and Function Calls

- **Stack Usage**: The stack holds state related to the currently executing function, specifically its parameters, local variables, and temporary values.
- **Stack Frames**: When a function `f()` is called, a new stack frame is added. When `f()` returns, the stack pointer resets to the caller's stack frame.
- **Memory Reuse**: When a different function `g()` is called, the stack frame for `g()` reuses the same area of memory that `f()` previously used.

## Fat Pointers

- **Slice**: A reference to a subset of some contiguous collection of values, built from a pointer and a length field.
- **Trait Object**: A reference to an item that implements a particular trait, built from a pointer to the item and a pointer to the type's vtable.

## Traits and Generics

- **Pointer Trait**: Formats a pointer value for output, useful for low-level debugging.
- **Borrow and BorrowMut Traits**: Methods `borrow` and `borrow_mut` have the same signature as `AsRef` and `AsMut`.
- **ToOwned Trait**: Allows functions to build their own copies of items.
- **AsRef Trait**: Used for cheap reference to reference conversions.
- **Box<T>**: Forces allocation on the heap, allowing the item to outlive the scope of the current block.
- **Rc and Weak<T>**: Used for cyclical data structures and non-owning references, respectively.
- **Cow**: "Clone-on-write" enum that can hold either owned data or a reference to borrowed data.

## Trait Coherence and Implementation

- **Trait Coherence**: There exists at most one `impl` of a trait for any given type.
- **Associated Types vs. Generic Types**: Use associated types for a single `impl` per type, and generic types for multiple possible `impls`.
- **Subtraits and Supertraits**: Subtraits refine their supertraits, making methods more specialized, adding guarantees, or extending functionality.

## Lifetimes

- **Stack Lifetimes**: The lifetime of an item on the stack is the period where a reference to the item is guaranteed not to become invalid.
- **Lifetime Extension and Reduction**: Convert a temporary to a named local variable or add an additional block around a reference to control its lifetime.
- **Non-Lexical Lifetimes**: The compiler treats the endpoint of a reference's lifetime as the last place it's used, rather than the end of the enclosing scope.
- **'static Lifetime**: The only allowed possibility for a returned reference with no input lifetimes, guaranteeing it never goes out of scope.

> Precisely where an item gets automatically dropped depends on whether an item has a name or not.

> One way to see what the compiler calculates as an item's lifetime is to insert a deliberate error for the borrow checker (Item 15) to detect.

> if the compiler can prove to itself that there is no use of a reference beyond a certain point in the code, then it treats the endpoint of the reference's lifetime as the last place it's used, rather than the end of the enclosing scope. This feature (known as non-lexical lifetimes) allows the borrow checker to be a little bit more generous:

- What happens if there are no input lifetimes, but the output return value includes a reference anyway?
    - The only allowed possibility is for the returned reference to have a lifetime that's guaranteed to never go out of scope. This is indicated by the special lifetime 'static, which is also the only lifetime that has a specific name rather than a placeholder label.




## Example Code

```rust
fn function<T: Clone>(t: T) {
    // impl
}
```

- **Generics and Trait Bounds**: Generic types depend on their trait bounds, implying a dependency on the trait.

## Debugging

- **dbg! Macro**: Superior to `println!` for quick and dirty print logging, printing to stderr and returning its arguments.

## Heap and Ownership

- **Heap Values**: Every item has an owner, and the lifetime of heap items is tied to stack lifetimes.
- **Box<T>**: Example of heap allocation with ownership.

```rust
{
    let b: Box<Item> = Box::new(Item { contents: 42 });
} // `b` dropped here, so `Item` dropped too.
```

- **Ownership Chain**: The chain of ownership must end at a local variable or a global variable marked as `'static`.

## Thread Safety

- **Send Trait**: Indicates types safe to move between threads.
- **'static Lifetime Bound**: Required for values moved between threads to ensure no stack references are involved.


---

These notes provide a comprehensive overview of Rust's ownership, memory management, traits, lifetimes, and asynchronous programming concepts.



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


---

### Data Races and Concurrency

A data race occurs when two distinct threads access the same memory location, where at least one of them is a write, and there is no synchronization mechanism that enforces an ordering on the accesses.

Enforcing that there is a single writer, or multiple readers (but never both), means that there can be no data races.

"Do not communicate by sharing memory; instead, share memory by communicating." In Rust, equivalent functionality is included in the standard library in the `std::sync::mpsc` module: the `channel()` function returns a `(Sender, Receiver)` pair that allows values of a particular type to be communicated between threads.

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

---

### Rust Variable and Struct Handling

- `Copy` is default for variables stored on the stack.
- Move is default for heap variables.
- Everything that is not a copy is a move.
- Structs are stored on the heap and move by default in Rust.
- Add `mut` to a struct variable to make every field mutable.
- Structs can be nested.
- Use `self` if its value is on the stack.

---

### Ad-hoc Conversions

Ad-hoc conversions follow `as_`, `to_`, `into_` conventions (C-CONV):

| Prefix | Cost      | Ownership                                      |
| ------ | --------- | ---------------------------------------------- |
| `as_`  | Free      | borrowed -> borrowed                           |
| `to_`  | Expensive | borrowed -> borrowed<br>borrowed -> owned (non-Copy types)<br>owned -> owned (Copy types) |
| `into_`| Variable  | owned -> owned (non-Copy types)                |

- Conversions prefixed `as_` and `into_` **typically decrease abstraction**:
  - Exposing a view into the underlying representation (`as`).
  - Deconstructing data into its underlying representation (`into`).
- Conversions prefixed `to_`:
  - Typically stay at the same level of abstraction.
  - Do some work to change from one representation to another.

**When a type wraps a single value to associate it with higher-level semantics, access to the wrapped value should be provided by an `into_inner()` method**. This applies to wrappers that provide buffering like `BufReader`, encoding or decoding like `GzDecoder`, atomic access like `AtomicBool`, or any similar semantics.

---

### Option and Result Transformations

- Get used to the transformations of `Option` and `Result`, and prefer `Result` to `Option`.
  - Use `.as_ref()` as needed when transformations involve references.
- Use them in preference to explicit `match` operations.
- In particular, use them to transform result types into a form where the `?` operator applies.

*Note: This method is separate from the `AsRef` trait, even though the method name is the same.*

---

### Data Structures and Self-Referential Data

**Data structures in Rust can move**: from the stack to the heap, from the heap to the stack, and from one place to another. If that happens, the "interior" title pointer would no longer be valid, and there's no way to keep it in sync.

A simple alternative for this case is to use the indexing approach explored earlier; **a range of offsets into the text is not invalidated by a move, and is invisible to the borrow checker because it doesn't involve references**.

A more general version of the self-reference problem turns up when the compiler deals with `async` code. Roughly speaking, the compiler bundles up a pending chunk of `async` code into a lambda, and the data for that lambda can include both values and references to those values. That's inherently a self-referential data structure, and so `async` support was a prime motivation for the `Pin` type in the standard library. **This pointer type "pins" its value in place, forcing the value to remain at the same location in memory, thus ensuring that internal self-references remain valid**.

The internal reference fields need to use raw pointers, or near relatives (e.g. `NonNull`) thereof. The type being pinned needs to not implement the `Unpin` marker trait. This trait is automatically implemented for almost every type, so this typically involves adding a (zero-sized) field of type `PhantomPinned` to the struct definition.

The item is only pinned once it's on the heap and held via `Pin`; in other words, only the contents of something like `Pin<Box<MyType>>` is pinned. This means that the internal reference fields can only be safely filled in after this point, but as they are raw pointers the compiler will give you no warning if you incorrectly set them before calling `Box::pin`.

Where possible, avoid self-referential data structures or try to find library crates that encapsulate the difficulties for you (e.g. ouroborous).

---

### Enums

Enums are useful because they allow writing functions that take multiple types and for vecs which allow any enum type. The size of an enum will not exceed the largest variant and needed tag.

---

### String Methods

- Chars are 4 bytes, not one in Rust.
- `trim()` removes whitespace and `trim_matches` removes strings.
- `trim_end_matches(w)` removes occurrences of `w`.
- `strip_prefix(p)` removes `p` at most once.
- You can split strings by `lines()`, `chars()`, `char_indices()`, `split_whitespace()`, `bytes()`, and `split_at(index)`.
- Indexing a string doesn't give a char.

---

### Vec Methods

- `extend()` (append by clone) lets you append to a vector by cloning contents of the second vector.
- `append()` normally just appends by move.
- `concat()` adds two containers together, while `join()` adds the two by a separator.
- `starts_with()` and `ends_with()`, `capacity()`, `length()`, `contains()`.
- Indexing collections can cause panics, but using `get(index)` allows you to get options back.

---

### Power Functions

- `pow()` for power.
- `powf()` for power float.
- `take()` replaces option with `None` and returns a new option with the value.
- `ok_or` turns `Some(valid_value)` into `Result::Ok(valid_value)` and if it's a `None` into a `Result::Err(err_value)`.
- `hash` for mapping a value of fixed size has to implement `Eq` and `PartialEq`.

---

### Conversion Traits

The four relevant traits that express the ability to convert values of a type are:

- `From<T>`: Items of this type can be built from items of type `T`.
- `TryFrom<T>`: Items of this type can sometimes be built from items of type `T`.
- `Into<T>`: Items of this type can be converted into items of type `T`.
- `TryInto<T>`: Items of this type can sometimes be converted into items of type `T`.

**Implement (just) the `Try`... trait if it's possible for a conversion to fail.**

**Implement the `From` trait for conversions.**

**Use the `Into` trait for trait bounds.**

However, a generic version of the function that accepts (and explicitly converts) anything satisfying `Into<IanaAllocated>`:

With this trait bound in place, the reflexive trait implementation of `From<T>` makes more sense: the combination of `From<T>` implementations and `Into<T>` trait bounds leads to code that appears to magically convert at the call site (but which is still doing safe, explicit, conversions under the covers). This pattern becomes even more powerful when combined with reference types and their related conversion traits.

There are only two coercions whose behavior can be affected by user-defined types. The first of these is when a user-defined type implements the `Deref` or the `DerefMut` trait. These traits indicate that the user-defined type is acting as a smart pointer of some sort, and in this case, the compiler will coerce a reference to the smart pointer item into being a reference to an item of the type that the smart pointer contains (indicated by its Target).

The second coercion of a user-defined type happens when a concrete item is converted to a trait object. This operation builds a fat pointer to the item; this pointer is fat because it includes both a pointer to the item's location in memory, together with a pointer to the vtable for the concrete type's implementation of the trait.

Rust includes the `as` keyword to perform explicit casts between some pairs of types. The `as` version also allows lossy conversions.

---

### Closures and Iterators

- `fnonce` consumes ownership of value.
- Statements are instructions that perform some action and don't return a value.
- Expressions evaluate to a resulting value.
- `and_then()` returns `None` if the option is `None`, else calls the `fnonce()` closure you send to it.
- `or_else()` returns the option if it contains a value, otherwise calls `f()` and returns the result.

---

### Iterators Consumers

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

---

### Iterator Transforms

The `Iterator` trait has a single required method (`next`), but also provides default implementations of a large number of other methods that perform transformations on an iterator. Some of these transformations affect the overall iteration process:

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

Taken as a whole, these methods allow iterators to be transformed so that they produce exactly the sequence of elements that are needed for most situations.

---


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


### Pinning and Self-Referential Data Structures

- Use `Pin` to ensure that internal self-references remain valid.
- Avoid self-referential data structures or use library crates that encapsulate the difficulties (e.g., ouroborous).

### Enums

- Enums allow writing functions that take multiple types and for vecs which allow any enum type.
- The size of an enum will not exceed the largest variant and needed tag.



### Option Methods

- `take()` replaces option with `None` and returns a new option with the value.
- `ok_or` turns `Some(valid_value)` into `Result::Ok(valid_value)` and `None` into `Result::Err(err_value)`.

### Hashing

- Implement `Eq` and `PartialEq` for hashing.







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


## Code snippets:

### Sealed Traits:
```rust
pub trait TheTrait: private::Sealed {
    // Zero or more methods that the user is allowed to call.
    fn .   ;
    // Zero or more private methods, not allowed for user to call.
    #[doc(hidden)]
    fn .   ;
}
// Implement for some types.
impl TheTrait for usize {
    /*    */
}
mod private {
    pub trait Sealed {}
    // Implement for those same types, but no others.
    impl Sealed for usize {}
}
```

### Private Fields:
```rust
struct Gizmo<T> {
    // ...
}
```

### Newtypes:
```rust
pub struct MyTransformResult<I>(Enumerate<Skip<I>>);
impl<I: Iterator> Iterator for MyTransformResult<I> {
    type Item = (usize, I::Item);
    fn next(&mut self) -> Option<Self::Item> {
        self.0.next()
    }
}
```

### Derived Trait Bounds:
```rust
#[derive(Clone, Debug, PartialEq)]
struct Gizmo<T> {
    // ...
}
```

---

## Concepts to Know:

### C-INTERMEDIATE:
Functions should expose intermediate results to avoid duplicate work. This means that if a function computes interesting related data, it should be exposed in the API. For example, `Vec::binary_search` returns information about the index if found, and also the index at which the value would need to be inserted if not found.

### C-CALLER-CONTROL:
If a function requires ownership of an argument, it should take ownership of the argument rather than borrowing and cloning the argument. For example, prefer `fn foo(b: Bar) { /* use b as owned, directly */ }` over `fn foo(b: &Bar) { let b = b.clone(); /* use b as owned after cloning */ }`.

### C-GENERIC:
Functions should minimize assumptions about parameters by using generics. This means that the fewer assumptions a function makes about its inputs, the more widely usable it becomes. For example, prefer `fn foo<I: IntoIterator<Item = i64>>(iter: I) { /* ... */ }` over any of `fn foo(c: &[i64]) { /* ... */ }`, `fn foo(c: &Vec<i64>) { /* ... */ }`, or `fn foo(c: &SomeOtherCollection<i64>) { /* ... */ }` if the function only needs to iterate over the data.

```rust
fn foo(b: Bar) -> (bool, Option<usize>) {
    // do something
    (result, related_data)
}
```

### Caller decides where to copy and place data (C-CALLER-CONTROL):
```rust
// Prefer this:
fn foo(b: Bar) {
    /* use b as owned, directly */
}
// Over this:
fn foo(b: &Bar) {
    let b = b.clone();
    /* use b as owned after cloning */
}
```

### Minimizing assumptions about parameters by using generics (C-GENERIC):
```rust
fn foo<I: IntoIterator<Item = i64>>(iter: I) { /* ... */ }
```

---

### Types are `Send` and `Sync` where possible (C-SEND-SYNC)
`Send` and `Sync` are automatically implemented when the compiler determines it is appropriate. In types that manipulate raw pointers, be vigilant that the `Send` and `Sync` status of your type accurately reflects its thread safety characteristics. Tests like the following can help catch unintentional regressions in whether the type implements `Send` or `Sync`.

```rust
#[test]
fn test_send() {
    fn assert_send<T: Send>() {}
    assert_send::<MyStrangeType>();
}

#[test]
fn test_sync() {
    fn assert_sync<T: Sync>() {}
    assert_sync::<MyStrangeType>();
}
```

---

### Error types are meaningful and well-behaved (C-GOOD-ERR)
An error type is any type `E` used in a `Result<T, E>` returned by any public function of your crate. `Error` types should always implement the `std::error::Error` trait which is the mechanism by which error handling libraries like `error-chain` abstract over different types of errors, and which allows the error to be used as the `source()` of another error.

Additionally, error types should implement the `Send` and `Sync` traits. An `error` that is not `Send` cannot be returned by a `thread` run with `thread::spawn`. An `error` that is not `Sync` cannot be passed across threads using an `Arc`. These are common requirements for basic error handling in a multithreaded application.

`Send` and `Sync` are also important for being able to package a custom error into an IO error using `std::io::Error::new`, which requires a trait bound of `Error + Send + Sync`.

One place to be vigilant about this guideline is in functions that return Error trait objects, for example `reqwest::Error::get_ref`. Typically `Error + Send + Sync + 'static` will be the most useful for callers. The addition of `'static` allows the trait object to be used with `Error::downcast_ref`.

Never use `()` as an error type, even where there is no useful additional information for the error to carry. `()` does not implement `Error` so it cannot be used with error handling libraries like `error-chain`. `()` does not implement `Display` so a user would need to write an error message of their own if they want to fail because of the error. `()` has an unhelpful Debug representation for users that decide to `unwrap()` the `error`. It would not be semantically meaningful for a downstream library to implement `From<()>` for their error type, so `()` as an error type cannot be used with the `?` operator.

Instead, define a meaningful error type specific to your crate or to the individual function. Provide appropriate `Error` and `Display` impls. If there is no useful information for the error to carry, it can be implemented as a unit struct.

```rust
use std::error::Error;
use std::fmt::Display;

// Instead of this...
fn do_the_thing() -> Result<Wow, ()>

// Prefer this...
fn do_the_thing() -> Result<Wow, DoError>

#[derive(Debug)]
struct DoError;

impl Display for DoError { /* ... */ }
impl Error for DoError { /* ... */ }
```

The `error` message given by the `Display` representation of an `error` type should be lowercase without trailing punctuation, and typically concise. `Error::description()` should not be implemented. It has been deprecated and users should always use `Display` instead of `description()` to print the error.

Examples of error messages:
- `"unexpected end of file"`
- `"provided string was not 'true' or 'false'"`
- `"invalid IP address syntax"`
- `"second time provided was later than self"`
- `"invalid UTF-8 sequence of {} bytes from index {}"`
- `"environment variable was not valid unicode: {:?}"`

### Binary number types provide `Hex`, `Octal`, `Binary` formatting (C-NUM-FMT)
`std::fmt::UpperHex`
`std::fmt::LowerHex`
`std::fmt::Octal`
`std::fmt::Binary`

These traits control the representation of a type under the `{:X}`, `{:x}`, `{:o}`, and `{:b}` `format!` specifiers. Implement these traits for any number type on which you would consider doing bitwise manipulations like `|` or `&`. This is especially appropriate for bitflag types. Numeric quantity types like `struct Nanoseconds(u64)` probably do not need these.

If we were adding an error to represent an address failing to parse, for consistency we would want to name it in verb-object-error or verb-subject-error order like `ParseAddrError` rather than `AddrParseError`.

---

### Generic reader/writer functions take `R: Read` and `W: Write` by value (C-RW-VALUE)
The standard library contains these two impls:
```rust
impl<'a, R: Read + ?Sized> Read for &'a mut R { /* ... */ }
impl<'a, W: Write + ?Sized> Write for &'a mut W { /* ... */ }
```

That means any function that accepts `R: Read` or `W: Write` generic parameters by value can be called with a `&mut` reference if necessary. In the documentation of such functions, briefly remind users that a mut reference can be passed. New Rust users often struggle with this. They may have opened a file and want to read multiple pieces of data out of it, but the function to read one piece consumes the reader by value, so they are stuck. The solution would be to leverage one of the above impls and pass `&mut f` instead of `f` as the reader parameter.

---

## Comparing and Sorting
- Process both strings from beginning to end as two sequences of maximal-length chunks, where each chunk consists either of a sequence of characters other than ASCII digits, or a sequence of ASCII digits (a numeric chunk), and compare corresponding chunks from the strings.
- To compare two numeric chunks, compare them by numeric value, ignoring leading zeroes. If the two chunks have equal numeric value, but different numbers of leading digits, and this is the first time this has happened for these strings, treat the chunks as equal (moving on to the next chunk) but remember which string had more leading zeroes.
- To compare two chunks if both are not numeric, compare them by Unicode character lexicographically, with two exceptions:
    - `_` (underscore) sorts immediately after (space) but before any other character. (This treats underscore as a word separator, as commonly used in identifiers.)
    - Unless otherwise specified, version-sorting should sort non-lowercase characters (characters that can start an UpperCamelCase identifier) before lowercase characters.
- If the comparison reaches the end of the string and considers each pair of chunks equal:
    - If one of the numeric comparisons noted the earliest point at which one string had more leading zeroes than the other, sort the string with more leading zeroes first.
    - Otherwise, the strings are equal.

### Expressions
Prefer to use Rust's expression oriented nature where possible:
```rust
// use
let x = if y { 1 } else { 0 };
// not
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
    a: A,
    b: B,
    long_name: LongType,
}
```

---

## When deciding on style guidelines, use these guiding principles (in rough priority order):
* readability - scan-ability
* aesthetics - consistent with other languages/tools
* specifics - preventing rightward drift
* application - ease of manual application

---

### Sorting:
* As the last member of a delimited expression, delimited expressions are generally combinable, regardless of the number of members. Previously only applied with exactly one member (except for closures with explicit blocks).
* When line-breaking a binary operator, if the first operand spans multiple lines, use the base indentation of the last line.
* Use version-sort (sort `x8`, `x16`, `x32`, `x64`, `x128` in that order).
* Change "ASCIIbetical" sort to Unicode-aware "non-lowercase before lowercase".
* Format single associated type `where` clauses on the same line if they fit.

### Formatting:
When a name is forbidden because it is a reserved word (such as `crate`), either use a raw identifier (`r#crate`) or use a trailing underscore (`crate_`). Don't misspell the word (`krate`). Prefer to use single-letter names for generic parameters. When writing extern items (such as `extern "C" fn`), always specify the ABI. For example, write `extern "C" fn foo ...`, not `extern fn foo ...`, or `extern "C" { ... }`.

A group of imports is a set of imports on the same or sequential lines. One or more blank lines or other items (e.g., a function) separate groups of imports. Within a group of imports, imports must be version-sorted. Groups of imports must not be merged or re-ordered.

### Ordering list import:
Names in a list import must be version-sorted, except that:
- `self` and `super` always come first if present, and
- groups and glob imports always come last if present.

This applies recursively. For example, `a::*` comes before `b::a` but `a::b` comes before `a::*`. E.g., `use foo::bar::{a, b::c, b::d, b::d::{x, y, z}, b::{self, r, s}};`.

### Normalisation:
Tools must make the following normalisations, recursively:
- `use a::self; -> use a;`
- `use a::{}; -> (nothing)`
- `use a::{b}; -> use a::b;`

Tools must not otherwise merge or un-merge import lists or adjust glob imports (without an explicit option). Each nested import must be on its own line, but non-nested imports must be grouped on as few lines as possible.

### Chains:
A chain is a sequence of field accesses, method calls, and/or uses of the try operator `?`. E.g., `a.b.c().d` or `foo?.bar().baz?`. Format the chain on one line if it is "small" and otherwise possible to do so. If formatting on multiple lines, put each field access or method call in the chain on its own line, with the line-break before the `.` and after any `?`. Block-indent each subsequent line:

### Use a semicolon where an expression has void type, even if it could be propagated. E.g.,
```rust
fn foo() { ... }
fn bar() {
    foo();
}
```

### Indentation and line width:
- Use **spaces**, not *tabs*.
- Each level of indentation must be **4 spaces** (that is, all indentation outside of string literals and comments must be a multiple of 4).
- The *maximum width* for a line is **100 characters**.

Keep type aliases on one line when they fit. If necessary to break the line, do so before the `=`, and block-indent the right-hand side. Format associated types like type aliases. Where an associated type has a bound, put a space after the colon but not before:

Format imports on one line where possible. Don't put spaces around braces. Prefer block indent over visual indent:

```rust
// Block indent
a_function_call(
    foo,
    bar,
);
// Visual indent
a_function_call(foo,
                bar);
```

This makes for smaller diffs (e.g., if `a_function_call` is renamed in the above example) and less rightward drift. For a multi-line tuple struct, block-format the fields with a field on each line and a trailing comma:

```rust
pub struct Foo(
    String,
    u8,
);
```

Use block-indent for trait items. If there are no items, format the trait (including its `{}`) on a single line. Otherwise, break after the opening brace and before the closing brace:

```rust
trait Foo {}
pub trait Bar {
    ...
}
```

If the trait has bounds, put a space after the colon but not before, and put spaces around each `+`, e.g.,
```rust
trait Foo: Debug + Bar {}
```

Use block-indent for `impl` items. If there are no items, format the `impl` (including its `{}`) on a single line. A `let` statement can contain an `else` component, making it a let-else statement. In this case, always apply the same formatting rules to the components preceding the else block (i.e. the `let pattern: Type = initializer_expr` portion) as described for other `let` statements.

Format the entire let-else statement on a single line if all the following are true:
- the entire statement is short
- the `else` block contains only a single-line expression and no statements
- the `else` block contains no comments
- the `let` statement components preceding the `else` block can be formatted on a single line

```rust
let Some(1) = opt else { return };
```

Break types with `+` by breaking before the `+` and block-indenting the subsequent lines. When breaking such a type, break before every `+`:

```rust
impl Clone
    + Copy
    + Debug
Box<
    Clone
  + Copy
  + Debug
>
```

Here is a reformatted version of your notes for GitHub Markdown:


## Trailing Commas
In comma-separated lists of any kind, use a trailing comma when followed by a newline:

```rust
function_call(
    argument,
    another_argument,
);

let array = [
    element,
    another_element,
    yet_another_element,
];
```

This makes moving code (e.g., by copy and paste) easier, and makes diffs smaller, as appending or removing items does not require modifying another line to add or remove a comma.

## Blank Lines
Separate items and statements by either zero or one blank lines (i.e., one or two newlines). For example:

```rust
fn foo() {
    let x = ...;
    let y = ...;
    let z = ...;
}

fn bar() {}

fn baz() {}
```

Avoid line-breaking in the signature if possible. If a line break is required in a non-inherent `impl`, break immediately before `for`, block indent the concrete type, and put the opening brace on its own line:

```rust
impl Bar
    for Foo
{
    ...
}
```

For modules: Use spaces around keywords and before the opening brace, no spaces around the semicolon.

## Macros
Use `{}` for the full definition of the macro.

```rust
macro_rules! foo {
}
```

Prefer to put a generics clause on one line. Break other parts of an item declaration rather than line-breaking a generics clause. If a generics clause is large enough to require line-breaking, prefer a `where` clause instead.

Do not put spaces before or after `<` nor before `>`. Only put a space after `>` if it is followed by a word or opening brace, not an opening parenthesis. Put a space after each comma. Do not use a trailing comma for a single-line generics clause.

## Comments
Prefer line comments (`//`) to block comments (`/* ... */`).

When using line comments, put a single space after the opening sigil.

Comments should usually be complete sentences. Start with a capital letter, end with a period (.). An inline block comment may be treated as a note without punctuation.

Source lines which are entirely a comment should be limited to 80 characters in length (including comment sigils, but excluding indentation) or the maximum width of the line (including comment sigils and indentation), whichever is smaller.

## Doc Comments
Prefer line comments (`///`) to block comments (`/** ... */`).

Prefer outer doc comments (`///` or `/** ... */`), only use inner doc comments (`//!` and `/*! ... */`) to write module-level or crate-level documentation.

Put doc comments before attributes.

For attributes with argument lists, format like functions.

## Items
- **Items** are exportable pieces of source code: structures, functions, constants, etc. Structures, Rust's class-like abstraction, are arguably our most fundamental organization tool. The top of the program organization hierarchy.
    - A full list of language constructs considered items is available. Technically, modules are items. But for the purpose of our current code organization discussion, we'll consider them taxonomically distinct.

## Modules
- **Modules** group related items into cohesive units. They facilitate organizing code within a project, much like namespaces.
    - Some programmers like to follow a "one module per source file" convention. But that 1:1 mapping is entirely optional. Modules are a logical, hierarchical grouping. They're not decided by the layout of a filesystem.

## Crates
- **Crates** group one or more related modules into either a library or a binary. They facilitate organizing code between projects. For libraries, visibility modifiers decide which items the module(s) export (e.g. the public API of the crate).
    - Crates can also have dependencies, which are themselves crates (e.g. 3rd party libraries used internally). Chapter 2's `rcli` tool was a binary crate that had two library crate dependencies: `rc4` and `clap`.

## System
- **System** is the general term for a large piece of software made up of interconnected components. That could mean multiple Rust crates, libraries written in other programming languages that interoperate via CFFI, or even networked sub-services that communicate using structured formats like REST and gRPC.
  
