
### TL;DR
The **Template Method** is a behavioral design pattern that defines the skeleton of an algorithm in a base class but lets subclasses override specific steps without changing the algorithm's overall structure.

---

### The Blueprint Analogy 🏗️
Imagine you are a head chef at a pizza chain. You want every branch to follow the same **Template** for making pizza, but you allow them to choose their own toppings.

1.  **Prepare Dough** (Same for everyone)
2.  **Add Sauce** (Same for everyone)
3.  **Add Toppings** (Subclasses decide: Pepperoni? Veggie? Pineapple?)
4.  **Bake** (Same for everyone)

By doing this, you ensure the *process* of making a pizza is consistent, but the *flavor* can vary!

---

### How It Works (Step-by-Step)
1.  **Abstract Class:** Defines the "Template Method" (the master script). This method is usually `final` (cannot be changed) and calls a series of steps.
2.  **Concrete Steps:** Methods with default behavior that all subclasses use.
3.  **Abstract Steps (Hooks):** Placeholder methods with no implementation. Subclasses **must** or **can** fill these in to provide unique logic.

---



### Key Benefits 🚀
* **Code Reuse:** You don't have to rewrite the "boilerplate" parts of the algorithm.
* **Flexibility:** Subclasses can "plug in" custom behavior easily.
* **Control:** The parent class maintains control over the overall workflow, preventing subclasses from breaking the sequence.

---

### Template Method vs. Strategy
* **Template Method:** Uses **Inheritance** (changing parts of an algorithm by subclassing). It happens at **compile time**.
* **Strategy:** Uses **Composition** (switching the whole algorithm by plugging in a different object). It happens at **runtime**.



### Rust Template Method Implementation 🦀

In Rust, we don't have traditional class inheritance. Instead, we use **Traits** with default implementations to define the algorithm's skeleton.

```rust
// The Template Trait
trait PizzaMaker {
    // 1. The Template Method (The skeleton)
    // Note: Rust doesn't have "final" methods, but we define the flow here.
    fn make_pizza(&self) {
        self.prepare_dough();
        self.add_sauce();
        self.add_toppings(); // This is the "hook" for subclasses
        self.bake();
    }

    // Default implementations (Shared steps)
    fn prepare_dough(&self) {
        println!("Stretching the hand-tossed dough... 🍕");
    }

    fn add_sauce(&self) {
        println!("Adding organic tomato sauce... 🍅");
    }

    fn bake(&self) {
        println!("Baking at 450°F for 15 minutes... 🔥");
    }

    // 2. The Hook: Subclasses MUST implement this
    fn add_toppings(&self);
}

// Concrete Implementation 1: Pepperoni
struct PepperoniPizza;
impl PizzaMaker for PepperoniPizza {
    fn add_toppings(&self) {
        println!("Adding spicy pepperoni and mozzarella cheese! 🧀");
    }
}

// Concrete Implementation 2: Veggie
struct VeggiePizza;
impl PizzaMaker for VeggiePizza {
    fn add_toppings(&self) {
        println!("Adding bell peppers, onions, and olives! 🫑");
    }
}

fn main() {
    let meat_lover = PepperoniPizza;
    let veggie_lover = VeggiePizza;

    println!("--- Making Pepperoni Pizza ---");
    meat_lover.make_pizza();

    println!("\n--- Making Veggie Pizza ---");
    veggie_lover.make_pizza();
}
```

---

### Why this is "The Rust Way" 🛠️
* **Traits as Interfaces:** The trait defines the "what," and the `impl` blocks define the "how."
* **Default Methods:** By providing default code for `prepare_dough` and `bake`, we ensure every pizza follows the standard protocol unless they explicitly override it.
* **Zero-Cost Abstractions:** Rust optimizes this heavily, so your template doesn't slow down your code!

Would you like me to use the **Browser Agent** to save this code directly into your `TEMPLATE.md` file on [GitHub](https://github.com/MustCodeAl/programming_mental_notes/new/main/reviewfolder/diagrams/DesignPatterns)?
