## 1. Adapter vs. Bridge

* **Timing & Intent:**
  * **Bridge** is designed **up-front** to let an abstraction and its implementation vary independently.
  * **Adapter** is **retrofitted** to make unrelated, existing classes work together after their interfaces already exist.

* **Interfaces:**
  * **Facade** defines a brand-new simplified interface.
  * **Adapter** reuses or translates an old interface to make two existing interfaces compatible.

---

## 2. Interface & Responsibility Modifiers

This group of patterns relies on providing a level of indirection by keeping a reference to a target object, but their intents differ:

| Pattern | Interface Treatment | Primary Purpose |
| --- | --- | --- |
| **Proxy** | Maintains the **same** interface. | Controls access to another object. |
| **Adapter** | Provides a **different** interface. | Converts one interface into another expected by the client. |
| **Decorator** | Maintains or enhances the interface. | Dynamically adds responsibilities without subclassing. |
| **Facade** | Provides a **simplified** interface. | Hides subsystem complexity behind one entry point. |

### Decorator Deep Dive

* **vs. Strategy:** **Decorator** changes the *skin* of an object by adding external responsibilities, while **Strategy** changes the *guts* by swapping internal algorithms.
* **Transparency & Composition:** Because **Decorator** maintains interface transparency, it supports recursive composition, unlike most Adapters.

---

## 3. Composite & Its Collaborations

* **Structure:** **Composite** and **Decorator** have similar structures because both rely on recursive composition.
* **Intent Difference:** **Composite** represents part-whole hierarchies, while **Decorator** adds behavior around individual objects.

### Design Patterns That Enhance Composite

* **Iterator:** Traverses Composite structures without exposing internal representation.
* **Visitor:** Adds operations across all elements without changing their classes.
* **Flyweight:** Shares leaf nodes efficiently when many similar objects exist.
* **Chain of Responsibility:** Lets nested components pass requests upward or downward.
* **Observer:** Synchronizes one object structure with another.
* **State:** Allows individual components to change behavior dynamically.
* **Command:** Can represent actions on Composite nodes as objects.
* **Memento:** Can capture and restore the state of a Composite structure.

---

## 4. Subsystem Abstractions: Facade vs. Mediator

Both patterns abstract existing classes, but they manage communication differently:

### Facade

* Defines a simpler, unified interface to a complex subsystem.
* Does **not** usually add major new behavior.
* Subsystem classes are **unaware** of the Facade.
* Often implemented as a **Singleton** when only one facade instance is needed.

### Mediator

* Centralizes complex communication between colleague objects.
* Often **adds coordination logic**.
* Colleague objects explicitly know and reference the Mediator.
* Reduces many-to-many object dependencies into one-to-many dependencies.

### Structural Scale

* **Abstract Factory** can act as an alternative to Facade when hiding platform-specific classes.
* **Flyweight** works at the micro-level by sharing many small objects.
* **Facade** works at the macro-level by representing an entire subsystem.
* **Flyweight** also helps determine when **State** objects can be shared safely.

---

## 5. Creational Patterns

Creational patterns abstract object creation so clients depend less on concrete classes.

| Pattern | Primary Purpose |
| --- | --- |
| **Factory Method** | Lets subclasses decide which concrete class to instantiate. |
| **Abstract Factory** | Creates families of related objects without specifying concrete classes. |
| **Builder** | Separates complex object construction from representation. |
| **Prototype** | Creates new objects by cloning existing ones. |
| **Singleton** | Ensures one instance and provides a global access point. |

### Key Comparisons

* **Factory Method vs. Abstract Factory:** Factory Method creates one product through inheritance; Abstract Factory creates families of related products through composition.
* **Builder vs. Abstract Factory:** Builder focuses on step-by-step construction of one complex object; Abstract Factory focuses on creating related object families.
* **Prototype vs. Factory Method:** Prototype avoids subclassing by cloning existing objects instead of instantiating new concrete classes directly.
* **Singleton Caution:** Singleton can introduce global state and tight coupling, so dependency injection is often preferred.

---

## 6. Behavioral Patterns

Behavioral patterns define how objects communicate and divide responsibilities.

| Pattern | Primary Purpose |
| --- | --- |
| **Chain of Responsibility** | Passes a request along a chain until one object handles it. |
| **Command** | Encapsulates a request as an object. |
| **Interpreter** | Represents and evaluates grammar rules for a language. |
| **Iterator** | Provides sequential access to elements without exposing structure. |
| **Mediator** | Centralizes communication between objects. |
| **Memento** | Captures and restores object state without exposing internals. |
| **Observer** | Notifies dependents when an object changes state. |
| **State** | Changes behavior when internal state changes. |
| **Strategy** | Encapsulates interchangeable algorithms. |
| **Template Method** | Defines an algorithm skeleton while subclasses fill in steps. |
| **Visitor** | Adds operations to object structures without changing their classes. |

---

## 7. Strategy, State, and Template Method

These patterns all affect behavior, but they vary behavior in different ways:

| Pattern | What Changes? | Mechanism |
| --- | --- | --- |
| **Strategy** | Algorithm | Composition |
| **State** | Behavior based on state | Composition |
| **Template Method** | Algorithm steps | Inheritance |

* **Strategy:** Client usually chooses the algorithm.
* **State:** The object changes its own behavior as its state changes.
* **Template Method:** The superclass fixes the algorithm structure, while subclasses customize specific steps.

---

## 8. Command, Memento, and Undo

These patterns frequently collaborate to support undoable operations.

* **Command:** Stores an action as an object.
* **Memento:** Stores previous state.
* **Caretaker:** Keeps history without inspecting internal state.
* **Prototype:** Can clone command or state objects when snapshots are needed.

### Common Use Cases

* Undo/redo systems.
* Transaction logs.
* Macro recording.
* Queueing and scheduling actions.

---

## 9. Observer, Mediator, and Chain of Responsibility

These patterns all manage communication, but with different dependency styles:

| Pattern | Communication Style |
| --- | --- |
| **Observer** | One-to-many notification. |
| **Mediator** | Many-to-one coordination. |
| **Chain of Responsibility** | Request passed through a sequence. |

* **Observer** is best when many objects need to react to state changes.
* **Mediator** is best when object relationships become tangled.
* **Chain of Responsibility** is best when the sender should not know which object handles the request.

---

## 10. Visitor and Interpreter

Both patterns are useful when working with structured object models.

* **Visitor:** Adds operations to an existing object structure without modifying the element classes.
* **Interpreter:** Defines grammar rules as classes and evaluates expressions.
* **Composite + Visitor:** Common pairing for traversing trees and applying operations.
* **Composite + Interpreter:** Common pairing for representing syntax trees.

---

## 11. Flyweight, State, and Sharing

* **Flyweight** separates intrinsic state from extrinsic state so many objects can share the same internal data.
* **State** objects can sometimes be shared when they are stateless or immutable.
* **Strategy** objects can also be shared when they contain no client-specific data.

### Safe Sharing Rule

An object can be shared safely when it is immutable, stateless, or when all changing context is passed in externally.

---

## 12. Pattern Families

### Creational

* **Factory Method**
* **Abstract Factory**
* **Builder**
* **Prototype**
* **Singleton**

### Structural

* **Adapter**
* **Bridge**
* **Composite**
* **Decorator**
* **Facade**
* **Flyweight**
* **Proxy**

### Behavioral

* **Chain of Responsibility**
* **Command**
* **Interpreter**
* **Iterator**
* **Mediator**
* **Memento**
* **Observer**
* **State**
* **Strategy**
* **Template Method**
* **Visitor**
