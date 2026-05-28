
YAGNI
YAGNI is an acronym that stands for You Aren't Going to Need It. It’s a vital software design principle to apply as you write code.

The best code I ever wrote was code I never wrote.

If we apply YAGNI to design patterns, we see that the features of Rust allow us to throw out many patterns. For instance, there is no need for the strategy pattern in Rust because we can just use traits.



Design patterns are “general reusable solutions to a commonly occurring problem within a given context in software design”. Design patterns are a great way to describe the culture of a programming language. Design patterns are very language-specific - what is a pattern in one language may be unnecessary in another due to a language feature, or impossible to express due to a missing feature.

If overused, design patterns can add unnecessary complexity to programs. However, they are a great way to share intermediate and advanced level knowledge about a programming language.




-----

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
