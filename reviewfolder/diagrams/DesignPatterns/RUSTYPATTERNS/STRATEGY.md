**TL;DR:** The [Strategy Pattern](https://rust-unofficial.github.io/patterns/patterns/behavioural/strategy.html) is a behavioral design pattern that lets you define a family of algorithms, put each of them into a separate class/struct, and make their objects interchangeable. 

Instead of writing a massive `if/else` block inside a single class to handle different variations of a task, you extract those variations into separate "Strategies." The main object then simply delegates the work to whatever Strategy object it was handed.

![Alt text](./stategy.png "strategy pattern")


### Core Concept
Imagine you are building a reporting tool. Sometimes you need to export reports as **JSON**, and other times as **Plain Text**. 

Instead of hardcoding both exporting logic paths into the `Report` struct, you define a `Formatter` interface. The `Report` struct doesn't care *how* the formatting happens; it just hands the data to whichever `Formatter` you gave it. This gives you **Dependency Inversion** and excellent **Separation of Concerns**.

---

### Implementation in Rust: Two Ways

As detailed in the [Rust Design Patterns](https://rust-unofficial.github.io/patterns/patterns/behavioural/strategy.html) documentation you have open, Rust offers two fantastic ways to handle this:

#### 1. The Classic Way (Traits)
This is exactly like traditional OOP, using a Trait to define the strategy and structs to implement it.

```rust
use std::collections::HashMap;

// 1. The Strategy Interface
trait Formatter {
    fn format(&self, data: &HashMap<String, u32>, buf: &mut String);
}

// 2. Concrete Strategies
struct Text;
impl Formatter for Text {
    fn format(&self, data: &HashMap<String, u32>, buf: &mut String) {
        for (k, v) in data {
            buf.push_str(&format!("{} {}\n", k, v));
        }
    }
}

struct Json;
impl Formatter for Json {
    fn format(&self, data: &HashMap<String, u32>, buf: &mut String) {
        // ... JSON formatting logic ...
        buf.push_str("{...json...}"); 
    }
}

// 3. The Context
struct Report;
impl Report {
    // Takes ANY strategy that implements Formatter
    fn generate<T: Formatter>(strategy: T, buf: &mut String) {
        let mut data = HashMap::new();
        data.insert("Sales".to_string(), 100);
        
        // Delegate the work!
        strategy.format(&data, buf);
    }
}

fn main() {
    let mut buffer = String::new();
    Report::generate(Text, &mut buffer); // Use Text Strategy
}
```

#### 2. The Idiomatic "Rusty" Way (Closures)
Because Rust treats functions as first-class citizens, you don't always need a whole Trait and Struct setup. You can simply pass a **Closure** as the strategy! This is incredibly common in Rust's standard library (e.g., `Option::map`).

```rust
struct Adder;

impl Adder {
    // The strategy is just a closure that takes two u8s and returns a u8
    pub fn add<F>(x: u8, y: u8, strategy: F) -> u8 
    where 
        F: Fn(u8, u8) -> u8, 
    {
        strategy(x, y)
    }
}

fn main() {
    // Strategy 1: Standard addition
    let arith_adder = |x, y| x + y;
    
    // Strategy 2: Custom logic
    let custom_adder = |x, y| 2 * x + y;

    println!("Standard: {}", Adder::add(4, 5, arith_adder)); // 9
    println!("Custom: {}", Adder::add(4, 5, custom_adder));  // 13
}
```

### Why Use It?
* **Open/Closed Principle:** You can introduce new strategies (like a `CsvFormatter`) without changing the context (`Report`) at all.
* **Runtime Swapping:** You can swap algorithms out dynamically while the program is running.
* **Crate ecosystem:** Rust's famous `serde` crate heavily uses the Strategy pattern, allowing you to easily swap out `serde_json` for `serde_yaml` or `serde_cbor` because they all adhere to the same underlying serialization strategies.
