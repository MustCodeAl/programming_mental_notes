
sources: 

[Effective Rust](https://www.lurklurk.org/effective-rust/)

[Building Linked List](https://rust-unofficial.github.io/too-many-lists/index.html)

[Design Patterns](https://rust-unofficial.github.io/patterns/intro.html)

[Api guidelines](https://rust-lang.github.io/api-guidelines/checklist.html)

[Breakdown rust notes](https://www.breakdown-notes.com/make/load/rust_cs_canvas)

[cheats.rs](https://cheats.rs/)






The owner of an item gets to create it, read from it, update it, and drop it (CRUD).

borrowing does not have ownership so the value wont be dropped when borrow goes out of scope

if you mutable borrow a value the owner cant change it till your done borrowing it

<b>only the item's owner can move the item </b>


<b>There can be any number of immutable references to the item, or
a single mutable reference to the item but not both </b>

the data pointed to at the location has to be valid and that any assignments to this location has to be of
valid data and is enforced by the reference to have existing data at all times,

It's <b>valid to read from a mutable reference, and it's valid to write to a mutable
reference, and so the ability to do both at once is provided by the `std::mem::replace` </b> 


`isize` is the size of pointer on your system

----
While a program is running, the memory that it uses is divided up into different chunks, sometimes called segments.
Some of these chunks are a fixed size, such as the ones that hold the program code or the program's global data,
but two of the chunks – the heap and the stack – change size as the program runs. To allow for this, 
they are typically arranged at opposite end's of the program's virtual memory space, so one can grow downwards
and the other can grow upwards (at least until your program runs out of memory and crashes).


Of these two dynamically sized chunks, the stack is used to hold state related to the currently executing function, specifically its parameters, local variables and temporary values, held in a stack frame. When a function `f()` is called, a new stack frame is added to the stack, beyond where the stack frame for the calling function ends, and the CPU normally updates a register – the stack pointer – to point to the new stack frame.

When the inner function `f()` returns, the stack pointer is reset to where it was before the call, which will be the caller's stack frame, intact and unmodified.

When the caller invokes a different function `g()`, the process happens again, which means that the stack frame for `g()` will re-use the same area of memory that `f()` previously used.


---



Rust has two built-in fat pointer types: types that act as pointers, but which hold additional information about the thing they are pointing to.

<b>The first such type is the slice: a reference to a subset of some contiguous collection of values. </b> It's built from a (non-owning) simple pointer, together with a length field, making it twice the size of a simple pointer (16 bytes on a 64-bit platform). 

<b> several traits (`Deref[Mut]`, `AsRef[Mut]` and `Index`) that are used when dealing with reference and slice types </b>

<b>The second built-in fat pointer type is a trait object: a reference to some item that implements a particular trait.</b> It's built from a simple pointer to the item, together with an internal pointer to the type's vtable, giving a size of 16 bytes (on a 64-bit platform). The vtable for a type's implementation of a trait holds function pointers for each of the method implementations, allowing dynamic dispatch at runtime

can be converted to a trait object of type `&dyn Trait` (where the `dyn` keyword highlights the fact that dynamic dispatch is involved):



---

the simplest is the `Pointer` trait, which formats a pointer value for output. This can be helpful for low-level debugging, and the compiler will reach for this trait automatically when it encounters the `{:p}` format specifier.

The `Borrow` and `BorrowMut` traits each have a single method (`borrow` and `borrow_mut` respectively) that has the same signature as the equivalent `AsRef` / `AsMut` trait methods.


A function that accepts `Borrow` can receive either items or references-to-items, and can work with references in either case.
A function that accepts `ToOwned` can receive either items or references-to-items, and can build its own personal copies of those items in either case.



`AsRef` is for cheap reference to reference conversions. However, one of the most common ways it's used is to make functions generic over whether they take ownership or not

---

Rust allocates items on the stack by default; the `Box<T>` pointer type (roughly equivalent to C++'s `std::unique_ptr<T>`) forces allocation to occur on the heap, which in turn means that the allocated item can outlive the scope of the current block. Under the covers, `Box<T>` is also a simple 8 byte pointer value.



if item `A` has an `Rc` pointer to item `B`, and item `B` has an `Rc` pointer to `A`, then the pair will never be dropped. To put it another way: you need `Rc` to support cyclical data structures, but the downside is that there are now cycles in your data structures.

 `Weak<T>` type, which holds a non-owning reference to the underlying item (roughly analogous to C++'s `std::weak_ptr<T>`). Holding a weak reference doesn't prevent the underlying item being dropped (when all strong references are removed), so making use of the `Weak<T>` involves an upgrade to an `Rc<T>` – which can fail.

(It's also worth mentioning `Cow` here, which is an enum that can hold either owned data, or a reference to borrowed data. The peculiar name stands for "clone-on-write": a `Cow` input can stay as borrowed data right up to the point where it needs to be modified, but becomes an owned copy at the point where the data needs to be altered.)

---

<b>Trait items cannot be used unless the trait is in scope. Most Rustaceans learn this the hard way </b>

Unlike default `impls`, which provide an `impl`, generic blanket `impls` provide the `impl`, so they are not overridable.


** Trait coherence is the property that there exists at most one `impl` of a trait for any given type. **

The general rule-of-thumb is:

* Use associated types when there should only be a single `impl` of the trait per type.
* Use generic types when there can be many possible `impls` of the trait per type.


---

All of the types which `impl Subtrait` are a subset of all the types which `impl Supertrait`, or to put it in opposite but equivalent terms: all the types which `impl Supertrait` are a superset of all the types which `impl Subtrait`.


since trait bounds can only be applied to concrete types which can `impl` traits. Traits cannot `impl` other traits


Furthermore, there are no rules for how a type must `impl` both a subtrait and a supertrait. It can use the methods from either in the `impl` of the other.


```rust
fn function<T: Clone>(t: T) {
    // impl
}
```

* Without knowing anything about the `impl` of this function we could reasonably guess that `t.clone()` gets called at some point because 

* when a generic type is bounded by a trait that strongly implies it has a dependency on the trait.

*  The mental model for understanding the relationship between generic types and their trait bounds is a simple and intuitive one: generic types depend on their trait bounds.



the most simple and elegant Alternative mental model for understanding the relationship between subtraits and supertraits is: subtraits refine their supertraits.

"Refinement" is intentionally kept somewhat vague because it can mean different things in different contexts:

* a subtrait might make its supertrait's methods' impls more specialized, faster, use less memory, e.g. Copy: Clone

* a subtrait might make additional guarantees about the supertrait's methods' impls, e.g. Eq: PartialEq, Ord: PartialOrd, ExactSizeIterator: Iterator

* a subtrait might make the supertrait's methods more flexible or easier to call, e.g. FnMut: FnOnce, Fn: FnMut

* a subtrait might extend a supertrait and add new methods, e.g. DoubleEndedIterator: Iterator, ExactSizeIterator: Iterator

---




Generics give us compile-time polymorphism where trait objects give us run-time polymorphism. We can use trait objects to allow functions to dynamically return different types at run-time:



Trait objects also allow us to store heterogeneous types in collections


Trait objects are unsized so they must always be behind a pointer. We can tell the difference between a concrete type and a trait object at the type level based on the presence of the `dyn` keyword within the type:



Not all traits can be converted into trait objects. A trait is object-safe if it meets these requirements:

* trait doesn't require `Self: Sized`
* all of the trait's methods are object-safe

A trait method is object-safe if it meets these requirements:
* method requires `Self: Sized` or 
* method only uses a `Self` type in receiver position


---


Impling Debug for a type also allows it to be used within the dbg! macro which is superior to println! for quick and dirty print logging. Some of its advantages:

dbg! prints to stderr instead of stdout so the debug logs are easy to separate from the actual stdout output of our program.
dbg! prints the expression passed to it as well as the value the expression evaluated to.
dbg! takes ownership of its arguments and returns them so you can use it within expressions:


---

the <b> stack is used to hold state related to the <i> currently </i> executing function, specifically its parameters, local variables and temporary values</b>, held in a stack frame. 

The lifetime of an item on the stack is the period where that item is guaranteed to stay in the same place; in other words, this is exactly the period where a reference (pointer) to item is guaranteed not to become invalid.




the scope of any Reference must be smaller than the lifetime of the item that it refers to. However, the compiler is smarter than just assuming that a reference lasts until it is dropped – the non-lexical lifetimes feature allows reference lifetimes to be shrunk so they end at the point of last use, rather than the enclosing block


<b>Lifetime extension</b>: convert a temporary (whose lifetime only extends to the end of the expression) to be a new named local variable (whose lifetime extends to the end of the block) with a `let` binding.
<b>Lifetime reduction</b>: add an additional block `{ ... }` around use of a reference so that its lifetime ends at the end of the new block.

Annotate `let` binding with type to help with debugging.

---

Precisely where an item gets automatically dropped depends on whether an item has a name or not.

Local variables and function parameters have names, and the corresponding item's lifetime starts when the item is created and the name is populated:

For a local variable: at the `let var = ...` declaration.
For a function parameter: as part of setting up execution frame for the function invocation.
The lifetime for a named item ends when the item is either moved somewhere else, or when the name goes out of scope:



It's also possible to build an item "on the fly", as part of an expression that's then fed into something else. These unnamed temporary items are then dropped when they're no longer needed.

One way to see what the compiler calculates as an item's lifetime is to insert a deliberate error for the borrow checker (Item 15) to detect.

 if the compiler can prove to itself that there is no use of a reference beyond a certain point in the code, then it treats the endpoint of the reference's lifetime as the last place it's used, rather than the end of the enclosing scope. This feature (known as non-lexical lifetimes) allows the borrow checker to be a little bit more generous:


---






 What happens if there are no input lifetimes, but the output return value includes a reference anyway?

The only allowed possibility is for the returned reference to have a lifetime that's guaranteed to never go out of scope. This is indicated by the special lifetime `'static`, which is also the only lifetime that has a specific name rather than a placeholder label.

---

The Rust compiler guarantees that a `'static` item always has the same address for the entire duration of the program, and never moves. This means that a reference to a `'static` item has a `'static` lifetime, logically enough.

Note that a `const` global variable does not have the same guarantee: only the value is guaranteed to be the same everywhere, and the compiler is allowed to make as many copies as it likes, wherever the variable is used. These potential copies may be ephemeral, and so won't satisfy the `'static` requirements:


There's one more possible way to get something with a `'static` lifetime. The key promise of `'static` is that the lifetime should outlive any other lifetime in the program; a value that's allocated on the heap but never freed also satisfies this constraint.


---

The key thing to realize about heap values is that every item has an owner (excepting special cases like the deliberate leaks described in the previous section). For example, a simple `Box<T>` puts the `T` value on the heap, with the owner being the variable holding the `Box<T>`:

```rust
    {
        let b: Box<Item> = Box::new(Item { contents: 42 });
    } // `b` dropped here, so `Item` dropped too.
```
The owning `Box<Item>` drops its contents when it goes out of scope, so the lifetime of the Item on the heap is the same as the lifetime of the `Box<Item>` variable on the stack.

The owner of a value on the heap may itself be on the heap rather than the stack, but then who owns the owner?

```rust
    {
        let b: Box<Item> = Box::new(Item { contents: 42 });
        let bb: Box<Box<Item>> = Box::new(b); // `b` moved onto heap here
    } // `b` dropped here, so `Box<Item>` dropped too, so `Item` dropped too
```

The chain of ownership has to end somewhere, and there are only two possibilities:

The chain ends at a local variable or function parameter – in which case the lifetime of everything in the chain is just the lifetime 'a of that stack variable. When the stack variable goes out of scope, everything in the chain is dropped too.
The chain ends at a global variable marked as `'static` – in which case the lifetime of everything in the chain is `'static`. The `'static` variable never goes out of scope, so nothing in the chain ever gets automatically dropped.
As a result, the lifetimes of items on the heap are fundamentally tied to stack lifetimes.


anything that contains a reference, no matter how deeply nested, is only valid for the lifetime of the item referred to. If that item is moved or dropped, then the whole chain of data structures is no longer valid.

As a result, prefer data structures that own their contents where possible. Where that's not possible, the various smart pointer types (e.g. `Rc`) described in Item 9 can help untangle the lifetime constraints.

---

`pub struct Ref<'a, T: 'a>(&'a T);`
This generic data structure holds an explicit reference `&'a T`, as per the first bullet above. But the type `T` might itself contain references with some lifetime 'b, as per the second bullet above. If `T`'s inherent lifetime `'b` were smaller than the exterior lifetime `'a` we'd have a potential disaster: the `Ref` would be holding a reference to a data structure whose own references have gone bad.

To prevent this, we need `'b` to be larger than `'a`; 


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

* `Future` has to be `poll`ed (by the executor) to resume where it last yielded and make progress (async is lazy)
* `&mut Self` contains state (state machine)
* `Pin` the memory location because the future contains self-referential data
* `Context` contains the Waker to notify the executor that progress can be made
* `async`/`await` on futures is implemented by generators
* `async fn` and `async` blocks return `impl Future<Output = T>`
* calling `.await` attempts to resolve the `Future`: if the `Future` is blocked, it yields control; if progress can be made, the `Future` resumes
Futures form a tree of futures. The leaf futures commmunicate with the executor. The root future of a tree is called a <b> task</b>.




---

`tokio` (multithreaded):	thread pool with work-stealing scheduler: each processor maintains its own run queue; idle processor checks sibling processor run queues, and attempts to steal tasks from them

`actix_rt`	single-threaded async runtime; futures are !Send

`actix-web`	constructs an application instance for each thread; application data must be constructed multiple times or shared between threads





----


`Stream<Item = T>` is an asynchronous version of `Iterator<Item = T>`, i.e., it does not block between each item yield


Create: 

* `futures::stream::iter`,
* `futures::stream::once,` 
* `futures::stream::repeat`, 
* `futures::stream::repeat_with`, 
* `async_stream::stream`



Combine	n-1:	

* `futures::stream::StreamExt::chain,` 
* `futures::stream::StreamExt::zip`, 
* `tokio_stream::StreamExt::merge`, 
* `tokio_stream::StreamMap`, 
* `tokio::select`

Split operations:	1-n	

* `futures::channel::oneshot::Sender::send`, 
* `async_std::channel::Sender::send`





---

A data race is defined to occur when two distinct threads access the same memory location, whereat least one of them is a write, and there is no synchronization mechanism that enforces an ordering on the accesses


enforcing that there is a single writer, or multiple readers (but never both) means that there can be no data races

"Do not communicate by sharing memory; instead, share memory by communicating". Rust, equivalent functionality is included in the standard library in the `std::sync::mpsc` module: the `channel()` function returns a `(Sender, Receiver)` pair that allows values of a particular type to be communicated between threads.

If shared-state concurrency can't be avoided, then there are some ways to reduce the chances of writing deadlock-prone code:

* ensure that access to it is governed by some kind of external synchronization for users of a struct

*  ensure it is thread safe by adding internal synchronization operations

* Use `Arc`, `Mutex`, and `RwLock`

* avoid interaction between the two independently locked data structures is where possible

* Use `sleep()` to help with debugging threads

* <b> avoid lock inversion</b>: Try to lock stuff in a good ordering, to make it more deterministic

* Put data structures that must be kept consistent with each other under a single lock.

* Keep lock scopes small and obvious, try to make it so no locks are held at the same time; wherever possible, use helper methods that get and set things under the relevant lock.

* Avoid invoking closures with locks held; this puts the code at the mercy of whatever closure gets added to the codebase in the future.

* Similarly, avoid returning a `MutexGuard` to a caller: it's like handing out a loaded gun from a deadlock perspective.
Include deadlock detection tools in your CI system (Item 32), such as `no_deadlocks`, `ThreadSanitizer`, or `parking_lot::deadlock`.

* As a last resort: design, document, test and police a locking hierarchy that describes what lock orderings are allowed/required. This should be a last resort because any strategy that relies on engineers never making a mistake is obviously doomed to failure in the long term.

More abstractly, multi-threaded code is an ideal place to apply the general advice: prefer code that's so simple that it is obviously not wrong, rather than code that's so complex that it's not obviously wrong.

----

`Copy` is default for variables stored on stack

move is default for heap variables
everything that is not a copy is a move

structs are stored on heap and move by default in rust

add `mut` to struct variable when making it allows every field to be mutable
structs can be nested


 use `self` if its  value is on the stack



---

Ad-hoc conversions follow `as_`, `to_`, `into_` conventions (C-CONV)


| Prefix | Cost | Ownership |
| ------ | ---- | --------- |
| `as_` | Free | borrowed -\> borrowed |
| `to_` | Expensive | borrowed -\> borrowed<br>borrowed -\> owned (non-Copy types)<br>owned -\> owned (Copy types) |
| `into_` | Variable | owned -\> owned (non-Copy types) |


* Conversions prefixed `as_` and `into_` **typically decrease abstraction**, 
  * either exposing a view into the underlying representation (`as`) 
  * or deconstructing data into its underlying representation (`into`). 

* Conversions prefixed `to_`, on the other hand, 
  * typically stay at the same level of abstraction 
  * but do some work to change from one representation to another.

* 
**When a type wraps a single value to associate it with higher-level semantics, access to the wrapped value should be provided by an `into_inner()` method** 
    * This applies to wrappers that provide buffering like `BufReader`, 
    * encoding or decoding like `GzDecoder`, 
    * atomic access like `AtomicBool`, or any similar semantics.




---

- Get used to the transformations of `Option` and `Result`, and prefer `Result` to `Option`.
	- Use `.as_ref()` as needed when transformations involve references.
- Use them in preference to explicit `match` operations.
- In particular, use them to transform result types into a form where the `?` operator applies.

<i>1: Note that this method is separate from the `AsRef` trait, even though the method name is the same. </i>

----

<b>Data structures in Rust can move: from the stack to the heap, from the heap to the stack, and from one place to another</b>. If that happens, the "interior" title pointer would no longer be valid, and there's no way to keep it in sync.

A simple alternative for this case is to use the indexing approach explored earlier; <b>a range of offsets into the text is not invalidated by a move, and is invisible to the borrow checker because it doesn't involve references</b>

 more general version of the self-reference problem turns up when the compiler deals with `async` code4. Roughly speaking, the compiler bundles up a pending chunk of `async` code into a lambda, and the data for that lambda can include both values and references to those values.

That's inherently a self-referential data structure, and so `async` support was a prime motivation for the `Pin` type in the standard library. <b>This pointer type "pins" its value in place, forcing the value to remain at the same location in memory, thus ensuring that internal self-references remain valid</b>.


The internal reference fields need to use raw pointers, or near relatives (e.g. `NonNull`) thereof.
The type being pinned needs to not implement the `Unpin` marker trait. This trait is automatically implemented for almost every type, so this typically involves adding a (zero-sized) field of type `PhantomPinned` to the struct definition5.
The item is only pinned once it's on the heap and held via `Pin`; in other words, only the contents of something like `Pin<Box<MyType>>` is pinned. This means that the internal reference fields can only be safely filled in after this point, but as they are raw pointers the compiler will give you no warning if you incorrectly set them before calling `Box::pin`.
Where possible, avoid self-referential data structures or try to find library crates that encapsulate the difficulties for you (e.g. ouroborous).


----



`enum` are useful, because it allows to write functions that take multiple types and for vecs which allow any enum type

size of enum will not exceed largest variant and needed tag

----

string methods: 

chars are 4 bytes not one in rust

`trim()` remove write spaces and trim matches remove strings
`trim_end_matches(w)` removals occurences of w
`strip_prefix(p)` removes p at most once

u can split strings by `lines()`, `chars()`,
`char_indices()`, `split_whitespace()`,
`bytes()`, and `split_at(index)`

indexing a string doesn't give a char

---

vec methods:

`extend()`(append by clone) lets you append to vector by cloning contents of second vector

`append()` normally just appends by move

`concat()` adds two containers together, while `join()` add the two by a separator


`starts_with()` and `ends_with()`, `capacity()`, `length()`, `contains()` 


indexing collections can cause panics,
but using `get(index)` allows you to get options back

----

power(`pow()`)
power float (`powf()`)

`take()` replaces option with `None` and return new option with the value

`ok_or` turns `Some(valid_value)` in `Result::Ok(valid_value)`
and if its a `None` into a `Result::Err(err_value)`

`hash` for mapping a value of fixing sized
has to implement `Eq` and `PartialEq`

---

The four relevant traits that express the ability to convert values of a type are:

`From<T>`: Items of this type can be built from items of type `T`.

`TryFrom<T>`: Items of this type can sometimes be built from items of type `T`.

`Into<T>`: Items of this type can converted into items of type `T`.

`TryInto<T>`: Items of this type can sometimes be converted into items of type `T`.

<b> implement (just) the `Try`... trait if it's possible for a conversion to fail </b>

<b> implement the `From` trait for conversions.</b>

<b> use the `Into` trait for trait bounds </b>


However, a generic version of the function that accepts (and explicitly converts) anything satisfying `Into<IanaAllocated>`:

With this trait bound in place, the reflexive trait implementation of `From<T>` makes more sense: 

the combination of `From<T>` implementations and `Into<T>` trait bounds leads to code that appears to magically convert at the call site (but which is still doing safe, explicit, conversions under the covers), This pattern becomes even more powerful when combined with reference types and their related conversion traits


There are only two coercions whose behaviour can be affected by user-defined types. The first of these is when a user-defined type implements the `Deref` or the `DerefMut` trait. These traits indicate that the user defined type is acting as a smart pointer of some sort (Item 9), and in this case the compiler will coerce a reference to the smart pointer item into being a reference to an item of the type that the smart pointer contains (indicated by its Target).

The second coercion of a user-defined type happens when a concrete item is converted to a trait object. This operation builds a fat pointer to the item; this pointer is fat because it includes both a pointer to the item's location in memory, together with a pointer to the vtable for the concrete type's implementation of the trait – see Item 9.

Rust includes the `as` keyword to perform explicit casts between some pairs of types.

The `as` version also allows lossy conversions3:


---

`fnonce` consumers ownership of value

statements are instructions that performance some action and don't return a value

expressions evaluate to a resulting value

`and_then()` returns none if option is none, else calls the `fnonce()` closure u send to it

`or_else()` returns the option if it contains a value, otherwise calls `f()` and returns the result

---

iterators CONSUMERS


- `iter()` and `fold()`, lets you iterate while holding state

- `scan()` takes an initialize value for internal state
and closure with two arguments one to a mutable reference to internal sate,
and the other an iterator element

- `any(P)` tests if any element in the collection matches the predicate, returns true if at-least one does.



- `all(p)` tests if every element in the collection matches the predicate and returns true if they ALL do


However, if the body of the for loop matches one of a number of common patterns, there are more specific iterator-consuming methods that are clearer. shorter and more idiomatic.

These patterns include shortcuts for building a single value out of the collection:

- `sum()`, for summing a collection of numeric values (integers or floats).

- `product()`, for multiplying together a collection of numeric values.

- `min()` and `max()`, for finding the extreme values of a collection, relative to the Item's `Ord` implementation (see Item 5).
- `min_by(f)` and `max_by(f)`, for finding the extreme values of a collection, relative to a user-specified comparison function f.

- `reduce(f)` is a more general operation that encompasses the previous methods, building an accumulated value of the Item type by running a closure at each step that takes the value accumulated so far and the current item.

- `fold(f)` is a generalization of reduce, allowing the "accumulated value" to be of an arbitrary type (not just the `Iterator::Item` type).
- `scan(f)` generalizes in a slightly different way, giving the closure a mutable reference to some internal state at each step.
There are also methods for selecting a single value out of the collection:

- `find(p)` finds the first item that satisfies a predicate.

- `position(p)` also finds the first item satisfying a predicate, but this time it returns the index of the item.

- `nth(n)` returns the n-th element of the iterator, if available.
There are methods for testing against every item in the collection:

---

Iterator Transforms

The Iterator trait has a single required method (`next`), but also provides default implementations (Item 13) of a large number of other methods that perform transformations on an iterator.

Some of these transformations affect the overall iteration process:

- `take(n)` restricts an iterator to emitting at most n items.

- `skip(n)` skips over the first n elements of the iterator.

- `step_by(n)` converts an iterator so it only emits every n-th item.

- `chain(other)` glues together two iterators, to build a combined iterator that moves through one then the other.

- `cycle()` converts an iterator that terminates into one that repeats forever, starting at the beginning again whenever it reaches the end. (The iterator must support `Clone` to allow this.)

- `rev()` reverses the direction of an iterator. (The iterator must implement the DoubleEndedIterator trait, which has an additional `next_back` required method.)

Other transformations affect the nature of the Item that's the subject of the `Iterator`:

- `map(|item| {...})` is the most general version, repeatedly applying a closure to transform each item in turn. Several of the following entries in this list are convenience variants that could be equivalently implemented as a map.

- `cloned()` produces a clone of all of the items in the original iterator; this is particularly useful with iterators over `&Item` references. (This obviously requires the underlying `Item` type to implement `Clone`).

- `copied()` produces a copy of all of the items in the original iterator; this is particularly useful with iterators over `&Item` references. (This obviously requires the underlying `Item` type to implement `Copy`).

- `enumerate()` converts an iterator over items to be an iterator over `(usize, Item)` pairs, providing an index to the items in the iterator.

- `zip(it)` joins an iterator with a second iterator, to produce a combined iterator that emits pairs of items, one from each of the original iterators, until the shorter of the two iterators is finished.

Yet other transformations perform filtering on the Items being emitted by the Iterator:

- `filter(|item| {...})` is the most general version, applying a bool-returning closure to each item reference to determine whether it should be passed through.

- `take_while()` and `skip_while()` are mirror images of each other, emitting either an initial subrange or a final subrange of the iterator, based on a predicate.

The `flatten()` method deals with an iterator whose items are themselves iterators, flattening the result. On its own, this doesn't seem that helpful, but it becomes much more useful when combined with the observation that both Option and Result act as iterators: they produce either zero (for `None`, `Err(e)`) or one (for `Some(v)`, `Ok(v)`) items. This means that flattening a stream of Option / Result values is a simple way to extract just the valid values, ignoring the rest.

Taken as a whole, these methods allow iterators to be transformed so that they produce exactly the sequence of elements that are needed for most situations.

---

adapters are functions that take iterators and return other iterators

The first step is to replace vector indexing with direct use of an iterator in a for-each loop:

```rust
for i in 0..values.len() {
        if values[i] % 2 != 0 {
            continue;
        }
```

to 

```rust
 for value in values.iter() {
        if value % 2 != 0 {
            continue;
        }
```


`flat_map()` combines into one

An initial arm of the loop that uses continue to skip over some items is naturally expressed as a `filter()`:

```rust
 for value in values.iter().filter(|x| *x % 2 == 0) {
}
```

Next, the early exit from the loop once 5 even items have been spotted maps to a `take(5)`:

```rust
 for value in values.iter().filter(|x| *x % 2 == 0).take(5) {
        even_sum_squares += value * value;
    }
```


The value of the item is never used directly, only in the value * value combination, which makes it an ideal target for a `map()`:

```rust   
 let mut even_sum_squares = 0;
    for val_sqr in values.iter().filter(|x| *x % 2 == 0).take(5).map(|x| x * x)
    {
        even_sum_squares += val_sqr;
    }

```

Other (more obscure) collection-producing methods include:



Finally, there are methods that accumulate all of the iterated items into a new collection. The most important of these is `collect()`, which can be used to build a new instance of any collection type that implements the FromIterator trait.

`unzip()`, which divides an iterator of pairs into two collections.
`partition(p)`, which splits an iterator into two collections based on a predicate that is applied to each item.




---


`file()` is a reference to an open file on the system

`path` give you operations for inspecting paths

`fs` gives you methods to manipulate contents of local filesystem- `read_to_string(&buffer)`, `copy(src, dst)`, `create_dir()`, `hard_link()`, `remove_dir()`, `remove_file(file_to_remove)`, `rename()`


dir entry represents an entry inside of a directory on the file system
, `metadata()`, `file_type()`, `file_name()`, `path()`,


---


Code snippets formatted in markdown:

Static Enforcement:
```rust
fn foo(a: Ascii) { /*    / }
```

Dynamic Enforcement:
```rust
fn foo(a: u8) {
    if !a.is_valid() {
        panic!("invalid input");
    }
    // ...
}
```

Dynamic Enforcement with `debug_assert!`:
```rust
fn foo(a: u8) {
    debug_assert!(a.is_valid());
    // ...
}
```


1. Static Enforcement: This is the process of using types to rule out bad inputs. For example, using the type `Ascii` instead of `u8` to guarantee that the highest bit is zero. This is the preferred method of enforcing validity of input.

2. Dynamic Enforcement: This is the process of validating input as it is processed, or ahead of time if necessary. This is often easier to implement than static enforcement, but has several drawbacks such as runtime overhead and delayed detection of bugs.

3. Destructors Never Fail: Destructors should not fail, as this will cause the program to abort. Instead, provide a separate method for checking for clean teardown, such as a `close` method, that returns a `Result` to signal problems.

4. Destructors That May Block Have Alternatives: Destructors should not invoke blocking operations, as this can make debugging much more difficult. Consider providing a separate method for preparing for an infallible, nonblocking teardown.

Here are a few code snippets formatted in markdown for these concepts:

Static Enforcement:
```rust
fn foo(a: Ascii) { /*    / }
```

Dynamic Enforcement:
```rust
fn validate_input() {
    // Validate input
}
```

Destructors Never Fail:
```rust
impl Drop for Foo {
    fn drop(&mut self) {
        // Do teardown and ignore or log/trace any errors
    }
}

fn close(&mut self) -> Result<(), Error> {
    // Check for clean teardown
}
```

Destructors That May Block Have Alternatives:
```rust
fn prepare_for_teardown(&mut self) -> Result<(), Error> {
    // Prepare for infallible, nonblocking teardown
}
```

---

All public types implement `Debug` (C-DEBUG)
If there are exceptions, they are rare.


`Debug` representation is never empty (C-DEBUG-NONEMPTY)
Even for conceptually empty values, the `Debug` representation should never be empty.


```rust

let empty_str = "";
assert_eq!(format!("{:?}", empty_str), "\"\"");

let empty_vec = Vec::<bool>::new();
assert_eq!(format!("{:?}", empty_vec), "[]");
```



---


Destructors never fail (C-DTOR-FAIL)
Destructors are executed while panicking, and in that context a failing destructor causes the program to abort.

Instead of failing in a destructor, provide a separate method for checking for clean teardown, e.g. a `close` method, that returns a `Result` to signal problems. If that `close` method is not called, the `Drop` implementation should do the teardown and ignore or log/trace any errors it produces.


Destructors that may block have alternatives (C-DTOR-BLOCK)
Similarly, destructors should not invoke blocking operations, which can make debugging much more difficult. Again, consider providing a separate method for preparing for an infallible, nonblocking teardown.

---


Sealed traits protect against downstream implementations (C-SEALED)
Some traits are only meant to be implemented within the crate that defines them. In such cases, we can retain the ability to make changes to the trait in a non-breaking way by using the sealed trait pattern.


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

---

Structs have private fields (C-STRUCT-PRIVATE)
Making a field public is a strong commitment: it pins down a representation choice, and prevents the type from providing any validation or maintaining any invariants on the contents of the field, since clients can mutate it arbitrarily.

Public fields are most appropriate for struct types in the C spirit: compound, passive data structures. Otherwise, consider providing getter/setter methods and hiding fields instead.


---

Newtypes encapsulate implementation details (C-NEWTYPE-HIDE)
A newtype can be used to hide representation details while making precise promises to the `client`.

For example, consider a function `my_transform` that returns a compound iterator type.


```rust
use std::iter::{Enumerate, Skip};

pub fn my_transform<I: Iterator>(input: I) -> Enumerate<Skip<I>> {
    input.skip(3).enumerate()
}
We wish to hide this type from the client, so that the client's view of the return type is roughly Iterator<Item = (usize, T)>. We can do so using the newtype pattern:



use std::iter::{Enumerate, Skip};

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

Aside from simplifying the signature, this use of newtypes allows us to promise less to the client. The client does not know how the result iterator is constructed or represented, which means the representation can change in the future without breaking client code.

Rust 1.26 also introduces the impl Trait feature, which is more concise than the newtype pattern but with some additional trade offs, namely with impl Trait you are limited in what you can express. For example, returning an iterator that impls Debug or Clone or some combination of the other iterator extension traits can be problematic. In summary impl Trait as a return type is probably great for internal APIs and may even be appropriate for public APIs, but probably not in all cases. See the "impl Trait for returning complex types with ease" section of the Edition Guide for more details.


```rust
pub fn my_transform<I: Iterator>(input: I) -> impl Iterator<Item = (usize, I::Item)> {
    input.skip(3).enumerate()
}
```

 ---
 
 Data structures do not duplicate derived trait bounds (C-STRUCT-BOUNDS)
Generic data structures should not use trait bounds that can be derived or do not otherwise add semantic value. Each trait in the derive attribute will be expanded into a separate impl block that only applies to generic arguments that implement that trait.


```rust
// Prefer this:
#[derive(Clone, Debug, PartialEq)]
struct Good<T> { /* ... */ }

// Over this:
#[derive(Clone, Debug, PartialEq)]
struct Bad<T: Clone + Debug + PartialEq> { /* ... */ }
//Duplicating derived traits as bounds on Bad is unnecessary and a backwards-compatibiliity hazard. To illustrate this point, consider deriving PartialOrd on the structures in the previous example:



// Non-breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Good<T> { /* ... */ }

// Breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Bad<T: Clone + Debug + PartialEq + PartialOrd> { /* ... */ }
```

Generally speaking, adding a trait bound to a data structure is a breaking change because every consumer of that structure will need to start satisfying the additional bound. Deriving more traits from the standard library using the derive attribute is not a breaking change.

The following traits should never be used in bounds on data structures:

- `Clone`
- `PartialEq`
- `PartialOrd`
- `Debug`
- `Display`
- `Default`
- `Error`
- `Serialize`
- `Deserialize`
- `DeserializeOwned`

There is a grey area around other non-derivable trait bounds that are not strictly required by the structure definition, like `Read` or `Write`. They may communicate the intended behavior of the type better in its definition but also limits future extensibility. Including semantically useful trait bounds on data structures is still less problematic than including derivable traits as bounds.
---


**Most important concepts to know:**

1. Sealed Traits: Sealed traits are traits that are only meant to be implemented within the crate that defines them. This allows for changes to be made to the trait in a non-breaking way. To avoid frustrated users trying to implement the trait, it should be documented that the trait is sealed and not meant to be implemented outside of the current crate.

2. Private Fields: Making a field public is a strong commitment and should only be done for struct types in the C spirit. Otherwise, consider providing getter/setter methods and hiding fields instead.

3. Newtypes: Newtypes can be used to hide representation details while making precise promises to the client. This allows for the representation to change in the future without breaking client code.

4. Derived Trait Bounds: Generic data structures should not use trait bounds that can be derived or do not otherwise add semantic value.

**Code snippets:**

Sealed Traits:
```rust
pub trait TheTrait: private::Sealed {
// Zero or more methods that the user is allowed to call.
fn .   ;
// Zero or more private methods, not allowed for user to call.
[doc(hidden)]
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

Private Fields:
```rust
struct Gizmo<T>{
// ...
}
```

Newtypes:
```rust
pub struct MyTransformResult<I>(Enumerate<Skip<I>>);
impl<I: Iterator> Iterator for MyTransformResult<I>; {
type Item  (usize, I::Item);
fn next(&mut self) -> Option<Self::Item> {
self.0.next 
}
}
```

Derived Trait Bounds:
```rust
[derive(Clone, Debug, PartialEq)]
struct Gizmo<T> {
// ...
}
```

---

Generic functions can require a lot of type parameters
and trait bounds, which can make the signature of the function hard to read.


**Concepts to Know:**

1. **C-INTERMEDIATE:** Functions should expose intermediate results to avoid duplicate work. This means that if a function computes interesting related data, it should be exposed in the API. For example, `Vec::binary_search` returns information about the index if found, and also the index at which the value would need to be inserted if not found.

2. **C-CALLER-CONTROL:** If a function requires ownership of an argument, it should take ownership of the argument rather than borrowing and cloning the argument. For example, prefer `fn foo(b: Bar) { /* use b as owned, directly */ }` over `fn foo(b: &Bar) { let b  b.clone; /* use b as owned after cloning */ }`.

3. **C-GENERIC:** Functions should minimize assumptions about parameters by using generics. This means that the fewer assumptions a function makes about its inputs, the more widely usable it becomes. For example, prefer `fn foo<I: IntoIterator<Item  i64>>(iter: I) { /*    / }` over any of `fn foo(c: &[i64]) { /*    / }`, `fn foo(c: &Vec<i64>) { /*    / }`, or `fn foo(c: &SomeOtherCollection<i64>) { /*    / }` if the function only needs to iterate over the data.

```rust
fn foo(b: Bar) -> (bool, Option<usize>) {
    // do something
    (result, related_data)
}
```

Caller decides where to copy and place data (C-CALLER-CONTROL):

```rust
// Prefer this:
fn foo(b: Bar) {
    /* use b as owned, directly */
}

// Over this:
fn foo(b: &Bar) {
    let b = b.clone;
    /* use b as owned after cloning */
}
```

Minimizing assumptions about parameters by using generics (C-GENERIC):

```rust
fn foo<I: IntoIterator<Item = i64>>(iter: I) { /* ... */ }
```

---
Types are `Send` and `Sync` where possible (C-SEND-SYNC)
`Send` and `Sync` are automatically implemented when the compiler determines it is appropriate.

In types that manipulate raw pointers, be vigilant that the `Send` and `Sync` status of your type accurately reflects its thread safety characteristics. Tests like the following can help catch unintentional regressions in whether the type implements `Send` or `Sync`.


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



Error types are meaningful and well-behaved (C-GOOD-ERR)
An error type is any type `E` used in a `Result<T, E>` returned by any public function of your crate. `Error` types should always implement the `std::error::Error` trait which is the mechanism by which error handling libraries like `error-chain` abstract over different types of errors, and which allows the error to be used as the `source()` of another error.

Additionally, error types should implement the `Send` and `Sync` traits. An `error` that is not `Send` cannot be returned by a `thread` run with `thread::spawn`. An `error` that is not `Sync` cannot be passed across threads using an `Arc`. These are common requirements for basic error handling in a multithreaded application.

`Send` and `Sync` are also important for being able to package a custom error into an IO error using `std::io::Error::new`, which requires a trait bound of `Error + Send + Sync`.

One place to be vigilant about this guideline is in functions that return Error trait objects, for example reqwest::Error::get_ref. Typically Error + Send + Sync + 'static will be the most useful for callers. The addition of 'static allows the trait object to be used with Error::downcast_ref.

Never use `()` as an error type, even where there is no useful additional information for the error to carry.

`()` does not implement `Error` so it cannot be used with error handling libraries like `error-chain`.
`()` does not implement `Display` so a user would need to write an error message of their own if they want to fail because of the error.
`()` has an unhelpful Debug representation for users that decide to `unwrap()` the `error`.
It would not be semantically meaningful for a downstream library to implement `From<()>` for their error type, so `()` as an error type cannot be used with the `?` operator.
Instead, define a meaningful error type specific to your crate or to the individual function. Provide appropriate Error and Display impls. If there is no useful information for the error to carry, it can be implemented as a unit struct.


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

The `error` message given by the `Display` representation of an `error` type should be lowercase without trailing punctuation, and typically concise.

`Error::description()` should not be implemented. It has been deprecated and users should always use `Display` instead of `description()` to print the error.

Examples of error messages
- `"unexpected end of file"`
- `"provided string was not` *true* `or` *false*`"`
- `"invalid IP address syntax"`
- `"second time provided was later than self"`
- `"invalid UTF-8 sequence of {} bytes from index {}"`
- `"environment variable was not valid unicode: {:?}"`


Binary number types provide `Hex`, `Octal`, `Binary` formatting (C-NUM-FMT)
`std::fmt::UpperHex`
`std::fmt::LowerHex`
`std::fmt::Octal`
`std::fmt::Binary`
These traits control the representation of a type under the `{:X}`, `{:x}`, `{:o}`, and `{:b}` `format!` specifiers.

Implement these traits for any number type on which you would consider doing bitwise manipulations like `|` or `&`. This is especially appropriate for bitflag types. Numeric quantity types like struct `Nanoseconds(u64)` probably do not need these.

If we were adding an error to represent an address failing to parse, for consistency we would want to name it in verb-object-error or verb-subject-error order like `ParseAddrError` rather than `AddrParseError`.

---

Generic reader/writer functions take R: Read and W: Write by value (C-RW-VALUE)
The standard library contains these two impls:


```rust
impl<'a, R: Read + ?Sized> Read for &'a mut R { /* ... */ }

impl<'a, W: Write + ?Sized> Write for &'a mut W { /* ... */ }
```
That means any function that accepts `R: Read` or `W: Write` generic parameters by value can be called with a `&mut` reference if necessary.

In the documentation of such functions, briefly remind users that a mut reference can be passed. New Rust users often struggle with this. They may have opened a file and want to read multiple pieces of data out of it, but the function to read one piece consumes the reader by value, so they are stuck. The solution would be to leverage one of the above impls and pass `&mut f` instead of `f` as the reader parameter.

---

**Comparing and Sorting**

- Process both strings from beginning to end as two sequences of maximal-length chunks, where each chunk consists either of a sequence of characters other than ASCII digits, or a sequence of ASCII digits (a numeric chunk), and compare corresponding chunks from the strings.
- To compare two numeric chunks, compare them by numeric value, ignoring leading zeroes. If the two chunks have equal numeric value, but different numbers of leading digits, and this is the first time this has happened for these strings, treat the chunks as equal (moving on to the next chunk) but remember which string had more leading zeroes.
- To compare two chunks if both are not numeric, compare them by Unicode character lexicographically, with two exceptions:
	- _ (underscore) sorts immediately after (space) but before any other character. (This treats underscore as a word separator, as commonly used in identifiers.)
	- Unless otherwise specified, version-sorting should sort non-lowercase characters (characters that can start an UpperCamelCase identifier) before lowercase characters.
- If the comparison reaches the end of the string and considers each pair of chunks equal:
	- If one of the numeric comparisons noted the earliest point at which one string had more leading zeroes than the other, sort the string with more leading zeroes first.
	- Otherwise, the strings are equal.



#### Expressions
Prefer to use Rust's expression oriented nature where possible;

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

Avoid `#[path]` annotations where possible.

Prefer to use multiple imports rather than a multi-line import. However, tools should not split imports by default.




------------------


### formatting:

When a name is forbidden because it is a reserved word (such as `crate`), either use a raw identifier (`r#crate`) or use a trailing underscore (`crate_`). Don't misspell the word (`krate`).

Prefer to use single-letter names for generic parameters.

When writing extern items (such as `extern "C" fn`), always specify the ABI. For example, write `extern "C" fn foo ...`, not `extern fn foo ...`, or `extern "C" { ... }`.

A group of imports is a set of imports on the same or sequential lines. One or more blank lines or other items (e.g., a function) separate groups of imports.

Within a group of imports, imports must be version-sorted. Groups of imports must not be merged or re-ordered


Ordering list import
Names in a list import must be version-sorted, except that:

- `self` and `super` always come first if present, and
- groups and glob imports always come last if present.
  
This applies recursively. For example, `a::*` comes before `b::a` but `a::b` comes before `a::*`. E.g., `use foo::bar::{a, b::c, b::d, b::d::{x, y, z}, b::{self, r, s}};`.

Normalisation
Tools must make the following normalisations, recursively:

- `use a::self; -> use a;`
- `use a::{}; -> (nothing)`
- `use a::{b}; -> use a::b;`
Tools must not otherwise merge or un-merge import lists or adjust glob imports (without an explicit option).

Each nested import must be on its own line, but non-nested imports must be grouped on as few lines as possible.



Prefer using a unit struct (e.g., `struct Foo;`) to an empty struct (e.g., `struct Foo();` or `struct Foo {}`, these only exist to simplify code generation), but if you must use an empty struct, keep it on one line with no space between the braces: `struct Foo;` or `struct Foo {}`.

For more than a few fields (in particular if the tuple struct does not fit on one line), prefer a proper struct with named fields.

The same guidelines are used for untagged union declarations.

```rust
union Foo {
    a: A,
    b: B,
    long_name:
        LongType,
}
```
Use a semicolon where an expression has void type, even if it could be propagated. E.g.,


```rust
fn foo() { ... }

fn bar() {
    foo();
}
```


###### Indentation and line width
Use **spaces**, not *tabs*.
Each level of indentation must be **4 spaces** (that is, all indentation outside of string literals and comments must be a multiple of 4).
The *maximum width* for a line is **100 characters**.

Keep type aliases on one line when they fit. If necessary to break the line, do so before the =, and block-indent the right-hand side, 
Format associated types like type aliases. Where an associated type has a bound, put a space after the colon but not before:

Format imports on one line where possible. Don't put spaces around braces.




Prefer block indent over visual indent:

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

This makes for smaller diffs (e.g., if `a_function_call` is renamed in the above example) and less rightward drift.
For a multi-line tuple struct, block-format the fields with a field on each line and a trailing comma:


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

If the trait has bounds, put a space after the colon but not before, and put spaces around each +, e.g.,

```rust
trait Foo: Debug + Bar {}
```

Use block-indent for `impl` items. If there are no items, format the `impl` (including its `{}`) on a single line


A `let` statement can contain an `else` component, making it a let-else statement. In this case, always apply the same formatting rules to the components preceding the else block (i.e. the `let pattern: Type = initializer_expr` portion) as described for other `let` statements.

Format the entire let-else statement on a single line if all the following are true:

- the entire statement is short
- the `else` block contains only a single-line expression and no statements
- the `else` block contains no comments
- the `let` statement components preceding the `else` block can be formatted on a single line

```rust
let Some(1) = opt else { return };
```

###### Trailing commas
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

Blank lines
Separate items and statements by either zero or one blank lines (i.e., one or two newlines). E.g,

```rust
fn foo() {
    let x = ...;

    let y = ...;
    let z = ...;
}

fn bar() {}
fn baz() {}
```

Avoid line-breaking in the signature if possible. If a line break is required in a non-inherent impl, break immediately before for, block indent the concrete type and put the opening brace on its own line:

```rust
impl Bar
    for Foo
{
    ...
}
```
for modules: Use spaces around keywords and before the opening brace, no spaces around the semicolon.


Use `{}` for the full definition of the macro.

```rust
macro_rules! foo {
}
```
Prefer to put a generics clause on one line. Break other parts of an item declaration rather than line-breaking a generics clause. If a generics clause is large enough to require line-breaking, prefer a where clause instead.

Do not put spaces before or after < nor before >. Only put a space after > if it is followed by a word or opening brace, not an opening parenthesis. Put a space after each comma. Do not use a trailing comma for a single-line generics clause.


###### Comments
Prefer line comments (`//`) to block comments (`/* ... */`).

When using line comments, put a single space after the opening sigil.

Comments should usually be complete sentences. Start with a capital letter, end with a period (.). An inline block comment may be treated as a note without punctuation.

Source lines which are entirely a comment should be limited to 80 characters in length (including comment sigils, but excluding indentation) or the maximum width of the line (including comment sigils and indentation), whichever is smaller:

###### Doc comments
Prefer line comments (`///`) to block comments (`/** ... */`).

Prefer outer doc comments (`///` or `/** ... */`), only use inner doc comments (`//!` and `/*! ... */`) to write module-level or crate-level documentation.

Put doc comments before attributes.

For attributes with argument lists, format like functions.



-----



**Items** are exportable pieces of source code: structures, functions, constants, etc. Structures, Rust's class-like abstraction, are arguably our most fundamental organization tool. The top of the program organization hierarchy.

A full list of language constructs considered items is available5. Technically, modules are items. But for the purpose of our current code organization discussion, we'll consider them taxonomically distinct.

**Modules** group related items into cohesive units. They facilitate organizing code within a project, much like namespaces.

Some programmers like to follow a "one module per source file" convention. But that 1:1 mapping is entirely optional. Modules are a logical, hierarchical grouping. They're not decided by the layout of a filesystem.

**Crates** group one or more related modules into either a library or a binary. They facilitate organizing code between projects. For libraries, visibility modifiers decide which items the module(s) export (e.g. the public API of the crate).

Crates can also have dependencies, which are themselves crates (e.g. 3rd party libraries used internally). Chapter 2's rcli tool was a binary crate that had two library crate dependencies: rc4 and clap.

**System** is the general term for a large piece of software made up of interconnected components. That could mean multiple Rust crates, libraries written in other programming languages that interoperate via CFFI6, or even networked sub-services that communicate using structured formats like REST7 and gRPC8.
