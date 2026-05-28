# YAGNI
***YAGNI*** is an acronym that stands for *You Aren't Going to Need It*. It’s a vital software design principle to apply as you write code.

The best code I ever wrote was code I never wrote.

If we apply **YAGNI** to design patterns, we see that the features of Rust allow us to throw out many patterns. For instance, there is no need for the strategy pattern in Rust because we can just use `trait`.



Design patterns are “general reusable solutions to a commonly occurring problem within a given context in software design”. Design patterns are a great way to describe the culture of a programming language. Design patterns are very language-specific - what is a pattern in one language may be unnecessary in another due to a language feature, or impossible to express due to a missing feature.

If overused, design patterns can add unnecessary complexity to programs. However, they are a great way to share intermediate and advanced level knowledge about a programming language.


----

## Rust Design Patterns in Plain English

## 1. Traits Instead of Inheritance

Rust does not use class inheritance like Java or C++.

Instead, Rust uses **traits**.

A trait describes behavior that a type must provide.

```rust
trait Draw {
    fn draw(&self);
}
```

Any type that implements `Draw` can be used where drawing behavior is needed.

This is Rust’s clean replacement for many object-oriented patterns.

---

## 2. Adapter Pattern in Rust

The **Adapter** pattern means wrapping one type so it works with a different interface.

Use this when you have an existing type, but it does not match the trait your code expects.

```rust
struct OldPrinter;

impl OldPrinter {
    fn print_old(&self) {
        println!("Printing the old way");
    }
}

trait Printer {
    fn print(&self);
}

struct PrinterAdapter {
    old_printer: OldPrinter,
}

impl Printer for PrinterAdapter {
    fn print(&self) {
        self.old_printer.print_old();
    }
}
```

Plain English:  
**The adapter translates one API into another API.**

---

## 3. Newtype Pattern

The **Newtype** pattern wraps an existing type in a small custom type.

This gives the value a clearer meaning and lets you implement traits safely.

```rust
struct UserId(u64);
struct OrderId(u64);
```

Even though both contain `u64`, Rust treats them as different types.

Plain English:  
**Use newtypes when two values have the same data type but different meanings.**

---

## 4. Builder Pattern in Rust

The **Builder** pattern helps create complex structs step by step.

It is useful when a struct has many optional fields.

```rust
struct ServerConfig {
    host: String,
    port: u16,
    use_tls: bool,
}

struct ServerConfigBuilder {
    host: String,
    port: u16,
    use_tls: bool,
}

impl ServerConfigBuilder {
    fn new() -> Self {
        Self {
            host: "localhost".to_string(),
            port: 8080,
            use_tls: false,
        }
    }

    fn port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }

    fn use_tls(mut self, use_tls: bool) -> Self {
        self.use_tls = use_tls;
        self
    }

    fn build(self) -> ServerConfig {
        ServerConfig {
            host: self.host,
            port: self.port,
            use_tls: self.use_tls,
        }
    }
}
```

Plain English:  
**Use a builder when normal constructors would have too many arguments.**

---

## 5. Strategy Pattern in Rust

The **Strategy** pattern means swapping out behavior without changing the main object.

In Rust, this is usually done with traits.

```rust
trait SortStrategy {
    fn sort(&self, data: &mut Vec<i32>);
}

struct QuickSort;

impl SortStrategy for QuickSort {
    fn sort(&self, data: &mut Vec<i32>) {
        data.sort();
    }
}
```

Plain English:  
**A strategy is a plug-in algorithm.**

---

## 6. Static Dispatch vs Dynamic Dispatch

Rust gives you two main ways to use traits.

### Static Dispatch

```rust
fn draw_static<T: Draw>(item: T) {
    item.draw();
}
```

Rust knows the exact type at compile time.

This is usually faster.

### Dynamic Dispatch

```rust
fn draw_dynamic(item: &dyn Draw) {
    item.draw();
}
```

Rust decides which method to call at runtime.

This is more flexible.

Plain English:  
**Use generics when you know the type shape early; use `dyn Trait` when you need mixed types at runtime.**

---

## 7. Decorator Pattern in Rust

The **Decorator** pattern wraps a value to add behavior without changing the original type.

```rust
struct LoggingWriter<W> {
    inner: W,
}
```

This is common with readers, writers, middleware, and service layers.

Plain English:  
**A decorator adds extra behavior around something else.**

---

## 8. Type State Pattern

The **Type State** pattern uses different Rust types to represent different states.

This prevents invalid actions at compile time.

```rust
struct Disconnected;
struct Connected;

struct Socket<State> {
    state: State,
}

impl Socket<Disconnected> {
    fn connect(self) -> Socket<Connected> {
        Socket { state: Connected }
    }
}
```

Plain English:  
**Make illegal states impossible by using the type system.**

---

## 9. RAII Pattern

RAII means a resource is cleaned up automatically when it goes out of scope.

Rust uses this everywhere.

```rust
{
    let file = std::fs::File::open("data.txt");
}
// file is closed automatically here
```

Plain English:  
**Rust cleans things up automatically when values are dropped.**

---

## 10. Interior Mutability

Normally, Rust requires `mut` to change data.

Interior mutability lets you mutate data through a shared reference.

Common tools:

```rust
Cell<T>
RefCell<T>
Mutex<T>
RwLock<T>
```

Plain English:  
**Use interior mutability when Rust’s normal borrowing rules are too strict, but you still need controlled mutation.**

---

## 11. Shared Ownership

Rust normally has one owner per value.

Sometimes, multiple parts of a program need to own the same value.

Use:

```rust
Rc<T>      // single-threaded shared ownership
Arc<T>     // thread-safe shared ownership
```

Plain English:  
**Use `Rc` or `Arc` when multiple owners need the same data.**

---

## 12. Command Pattern in Rust

The **Command** pattern stores an action as a value.

In Rust, this can be done with closures or trait objects.

```rust
let command = || println!("Run command");
command();
```

Or:

```rust
trait Command {
    fn execute(&self);
}
```

Plain English:  
**A command is an action you can store, pass around, and run later.**

---

## 13. Iterator Pattern in Rust

Rust has built-in iterator support.

```rust
let numbers = vec![1, 2, 3];

for number in numbers.iter() {
    println!("{number}");
}
```

Iterators allow lazy, chainable data processing.

```rust
let doubled: Vec<i32> = numbers
    .iter()
    .map(|n| n * 2)
    .collect();
```

Plain English:  
**Iterators let you process data one item at a time without exposing the collection’s internals.**

---

## 14. Error Handling Pattern

Rust avoids exceptions.

Instead, Rust uses:

```rust
Result<T, E>
Option<T>
```

Use `Result` when something can fail.

Use `Option` when something may be missing.

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err("cannot divide by zero".to_string())
    } else {
        Ok(a / b)
    }
}
```

Plain English:  
**Rust makes failure visible in the type system.**

---

## 15. Smart Pointer Patterns

Rust uses smart pointers to control ownership and memory behavior.

Common smart pointers:

| Type | Use |
| --- | --- |
| `Box<T>` | Store data on the heap |
| `Rc<T>` | Shared ownership in one thread |
| `Arc<T>` | Shared ownership across threads |
| `RefCell<T>` | Runtime-checked borrowing |
| `Mutex<T>` | Safe mutation across threads |

Plain English:  
**Smart pointers give you different ownership and borrowing behavior.**

---

## 16. Common Rust Pattern Choices

| Problem | Rust Pattern |
| --- | --- |
| Need shared behavior | Trait |
| Need optional values | `Option<T>` |
| Need recoverable errors | `Result<T, E>` |
| Need many constructor options | Builder |
| Need compile-time state safety | Type State |
| Need shared ownership | `Rc<T>` or `Arc<T>` |
| Need runtime polymorphism | `dyn Trait` |
| Need wrapper behavior | Newtype or Decorator |
| Need safe cleanup | RAII / `Drop` |
| Need lazy data processing | Iterator |

---

## 17. Rust Mindset

Rust patterns usually focus on:

* ownership
* borrowing
* lifetimes
* traits
* type safety
* explicit error handling
* zero-cost abstractions

Plain English:  
**Rust patterns are about making invalid code hard to write and valid code fast.**

----

## 1. Adapter vs. Bridge vs. Facade

* **Bridge (Before):** You use this **during early planning**. It keeps your high-level logic separated from the low-level technical code so both can change without breaking each other.
* **Adapter (After):** You use this **later on** as a quick fix. It acts like a physical plug adapter, forcing two old, incompatible pieces of code to work together.
* **Facade (New Interface):** It creates a brand-new, simplified menu to hide a massive, messy kitchen of code behind it. An **Adapter**, by contrast, just wrappers an *existing* menu to make it readable.

---

## 2. Wrappers: Proxy, Adapter, and Decorator

These three patterns all act as "wrappers" around another object, but they have completely different goals:

* **Proxy (The Guard):** Keeps the exact same interface. It stands in front of the real object to control access, check permissions, or delay loading.
* **Adapter (The Translator):** Changes the interface. It translates what the client wants into what the wrapped object actually understands.
* **Decorator (The Upgrade):** Keeps the same interface but upgrades what it does. It adds new features dynamically (like adding toppings to a pizza) without forcing you to write a whole new subclass.

> **Decorator vs. Strategy:** Think of a **Decorator** as changing the *skin* of an object (adding external features). Think of a **Strategy** as swapping out its *guts* (changing the internal logic or algorithm it uses).

---

## 3. Composite (The Tree Structure)

The **Composite** pattern is used when you want to treat a single item and a whole collection of items exactly the same way (like treating a single file and a folder full of files the same).

Because it creates a tree-like hierarchy, many other patterns work perfectly with it:

* **Iterator:** Helps you loop through all the items in the tree.
* **Visitor:** Lets you run an operation over every single item in the tree.
* **Flyweight:** Saves memory by sharing identical, tiny "leaf" pieces of the tree instead of recreating them.
* **Chain of Responsibility:** Allows an item deep in the tree to pass a request up to its parent folder.
* **Observer / State:** Allows parts of the tree to watch other parts or change how they act depending on their current situation.

---

## 4. Facade vs. Mediator

Both of these patterns are used to clear up messy code, but their communication styles differ:

* **Facade (One-Way Filter):** A simple entryway to a complex system. The complex system underneath has no idea the Facade even exists. Usually, you only need one instance of a Facade (**Singleton**).
* **Mediator (The Traffic Cop):** A central hub where multiple objects talk *to each other*. Every object knows about the Mediator, and the Mediator actively routes traffic and manages their behavior.

