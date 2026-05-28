

## 1. Adapter vs. Bridge

* **Timing & Intent:** * **Bridge** is designed **up-front** to let an abstraction and its implementation vary independently (making things work *before* they are designed).
* **Adapter** is **retrofitted** to make unrelated, existing classes work together (making things work *after* they are designed).


* **Interfaces:** **Facade** defines a brand-new interface, whereas **Adapter** reuses an old interface to make two existing interfaces compatible.

---

## 2. Interface & Responsibility Modifiers

This group of patterns relies on providing a level of indirection by keeping a reference to a target object, but their intents differ entirely:

| Pattern | Interface Treatment | Primary Purpose |
| --- | --- | --- |
| **Proxy** | Maintains the **same** interface. | Provides a level of indirection/control to its subject. |
| **Adapter** | Provides a **different** interface. | Changes the interface to match a client's expectations. |
| **Decorator** | Provides an **enhanced** interface. | Dynamically adds responsibilities ("changes the skin") without subclassing. |

### Decorator Deep Dive

* **vs. Strategy:** **Decorator** changes the *skin* (external responsibilities) of an object, while **Strategy** changes the *guts* (internal algorithms).
* **Transparency & Composition:** Because **Decorator** maintains interface transparency, it supports **recursive composition**—something not possible with pure Adapters.

---

## 3. Composite & Its Collaborations

* **Structure:** **Composite** and **Decorator** share very similar structural diagrams because both rely on recursive composition. However, Composite focuses on **representation** (hierarchies), while Decorator focuses on **embellishment** (adding features). They are frequently used together.

### Design Patterns That Enhance Composite:

* **Iterator:** Used to traverse a Composite structure.
* **Visitor:** Used to apply an operation across all elements of a Composite.
* **Flyweight:** Combined with Composite to allow leaf nodes to be shared efficiently.
* **Mediator:** Can be assembled out of smaller components using recursive composition.
* **Chain of Responsibility:** Allows nested components to access global properties through their parent nodes.
* **Observer:** Used to link or synchronize one object structure to another.
* **State:** Allows individual components within the Composite to alter their behavior dynamically as their state changes.

---

## 4. Subsystem Abstractions: Facade vs. Mediator

Both patterns abstract the functionality of existing classes, but they manage communication differently:

* **Facade:** * Defines a simpler, unified interface to a complex subsystem.
* Does *not* add new behavior or functionality.
* The underlying subsystem classes are **unaware** of the Facade.
* Often implemented as a **Singleton** since only one facade instance is typically needed.


* **Mediator:** * Centralizes and abstracts arbitrary, complex communication between "colleague" objects.
* Routinely **adds value** by coordinating logic.
* Is explicitly **known and referenced** by the colleague objects.



### Structural Scale

* **Abstract Factory** can act as an alternative to Facade when you need to hide platform-specific classes.
* **Flyweight** focuses on the micro-level (managing a massive quantity of tiny objects), whereas **Facade** focuses on the macro-level (a single object representing an entire subsystem).
* **Flyweight** also provides the blueprint for when and how **State** objects can be shared safely.

----

