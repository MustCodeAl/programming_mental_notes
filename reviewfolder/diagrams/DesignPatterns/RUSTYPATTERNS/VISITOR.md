

The **Visitor Pattern** is a behavioral design pattern used to separate an algorithm from the object structure on which it operates. It allows you to add new operations to existing object structures without modifying those structures.

### Core Concept
Imagine you have a complex graph of objects (like a file system or a compiler's abstract syntax tree). Instead of adding logic into every class to handle a specific task, you create a "Visitor" object that "visits" each element and performs the operation.


![Alt text](./visitorpattern.png "visotor pattern")


---

### Implementation Steps

1.  **Define the Visitor Interface**: Declare a `visit` method for each type of concrete element in the object structure.
2.  **Define the Element Interface**: Declare an `accept` method that takes a visitor as an argument.
3.  **Implement Concrete Elements**: Each class implements `accept` by calling the visitor method corresponding to its class (e.g., `visitor.visitConcreteElementA(this)`).
4.  **Implement Concrete Visitors**: Create classes that implement the visitor interface to define specific behaviors (e.g., `ExportVisitor`, `TaxCalculationVisitor`).


---

### Why Use It?

* **Open/Closed Principle**: You can introduce new behaviors without changing the element classes.
* **Single Responsibility Principle**: You can move multiple versions of the same behavior into a single class.
* **Double Dispatch**: It uses a technique where the operation executed depends on both the type of the visitor and the type of the element.

---

### Comparison: Visitor vs. Strategy

| Feature | Visitor Pattern | Strategy Pattern |
| :--- | :--- | :--- |
| **Purpose** | Performs operations across a complex object structure. | Switches between different algorithms for a single task. |
| **Target** | Multiple heterogeneous classes. | A single class or context. |
| **Structure** | Uses "Double Dispatch" (`accept` + `visit`). | Uses composition and a simple method call. |



For a deep dive into how this looks in practice, check out the [Refactoring Guru Visitor Documentation](https://refactoring.guru/design-patterns/visitor).



---

This version uses a single visitor to handle multiple shapes, keeping the logic decoupled from the data structures.




```rust
// 1. The Visitor Trait
trait ShapeVisitor {
    fn visit_circle(&self, c: &Circle);
    fn visit_rect(&self, r: &Rectangle);
}

// 2. The Element Trait
trait Shape {
    fn accept(&self, v: &dyn ShapeVisitor);
}

// 3. Concrete Elements
struct Circle { radius: f32 }
impl Shape for Circle {
    fn accept(&self, v: &dyn ShapeVisitor) { v.visit_circle(self); }
}

struct Rectangle { width: f32, height: f32 }
impl Shape for Rectangle {
    fn accept(&self, v: &dyn ShapeVisitor) { v.visit_rect(self); }
}

// 4. Concrete Visitor (The Algorithm)
struct AreaCalculator;
impl ShapeVisitor for AreaCalculator {
    fn visit_circle(&self, c: &Circle) {
        println!("Circle Area: {:.2}", 3.14 * c.radius.powi(2));
    }
    fn visit_rect(&self, r: &Rectangle) {
        println!("Rect Area: {:.2}", r.width * r.height);
    }
}

fn main() {
    let shapes: Vec<Box<dyn Shape>> = vec![
        Box::new(Circle { radius: 5.0 }),
        Box::new(Rectangle { width: 10.0, height: 2.0 }),
    ];

    let calc = AreaCalculator;
    for shape in shapes {
        shape.accept(&calc);
    }
}
```

**TL;DR:** The Visitor Pattern is an awesome behavioral trick that lets you separate an algorithm from the object structure it operates on!

Think of it like a nerdy health inspector visiting different restaurants (elements); the inspector brings their own unique checklist (the logic), so the restaurants only need a single method to let them inside without changing their menus.

To build this: 
1) create a `trait Visitor` with specific `visit` methods, 
2) add an `accept` method to your elements, and
3) let elements pass themselves to the `Visitor` to achieve double dispatch.

---


rust specific

### Simplified Breakdown
* **The Data (`Circle`, `Rectangle`)**: These are "dumb" structs. They only know how to "accept" a visitor.
* **The Logic (`AreaCalculator`)**: All the math lives here. If you want to add a `JsonExporter` later, you just create a new visitor without touching the Shape structs.
* **The Dispatch**: `shape.accept(&calc)` is the magic moment. The shape tells the visitor exactly which method to run based on its own type.

### When to avoid this in Rust?
If you know all your shapes upfront, use an **Enum** with a `match` statement. It is faster, more "idiomatic" Rust, and avoids the complexity of traits and `Box<dyn Shape>`.

Implementing the Visitor Pattern in Rust is unique because the language lacks traditional inheritance. Using **Enums** in Rust is often considered the more "idiomatic" approach for double dispatch because it leverages **pattern matching**, which is highly optimized and safer than trait objects.

Here is the simplified, complete code using **Enums**:

```rust
// 1. Define the Shape Enum
enum Shape {
    Circle { radius: f32 },
    Rectangle { width: f32, height: f32 },
}

// 2. Define the Visitor Trait
// The methods now take the specific variants or the Enum itself
trait ShapeVisitor {
    fn visit(&self, shape: &Shape);
}

// 3. Implement a Concrete Visitor
struct AreaCalculator;

impl ShapeVisitor for AreaCalculator {
    fn visit(&self, shape: &Shape) {
        // Double Dispatch via Pattern Matching
        match shape {
            Shape::Circle { radius } => {
                println!("Circle Area: {:.2}", 3.14 * radius.powi(2));
            }
            Shape::Rectangle { width, height } => {
                println!("Rect Area: {:.2}", width * height);
            }
        }
    }
}

// 4. Usage
fn main() {
    let shapes = vec![
        Shape::Circle { radius: 5.0 },
        Shape::Rectangle { width: 10.0, height: 2.0 },
    ];

    let calc = AreaCalculator;

    for shape in &shapes {
        calc.visit(shape);
    }
}
```

### Why this is better for Rust
* **No Allocation**: You don't need `Box<dyn Shape>`. The shapes are stored directly in the `Vec`.
* **Compiler Guarantees**: If you add a `Square` variant to the `Shape` enum, the Rust compiler will force you to update your `match` statements in the visitors.
* **Speed**: Pattern matching on enums is generally faster than dynamic dispatch (vtable lookups) used in trait-based visitors.

### Comparison Summary
| Feature | Trait-Based (Classic) | Enum-Based (Idiomatic Rust) |
| :--- | :--- | :--- |
| **Flexibility** | Easier to add new **Shapes**. | Easier to add new **Visitors**. |
| **Performance** | Slower (Dynamic Dispatch). | Faster (Static Dispatch/Branching). |
| **Code Style** | More boilerplate. | Clean and concise. |
