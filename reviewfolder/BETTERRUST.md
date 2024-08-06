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

## Async and Futures

- `Future` has to be `poll`ed (by the executor) to resume where it last yielded and make progress.
- `Pin` the memory location because the future contains self-referential data.
- `Context` contains the Waker to notify the executor that progress can be made.
- `async`/`await` on futures is implemented by generators.
- `Stream<Item = T>` is an asynchronous version of `Iterator<Item = T>`.

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

## Useful Crates

- `serde` for serialization and deserialization.
- `tokio` for asynchronous programming.
- `reqwest` for making HTTP requests.
- `clap` for command-line argument parsing.
- `rand` for random number generation.

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
