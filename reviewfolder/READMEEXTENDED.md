# Programming & Compiler Mental Notes

---

## I. Hardware Fundamentals

### The CPU

A **CPU** is an implementation of an **Instruction Set Architecture (ISA)** — an abstract model defining data types, registers, hardware support, and I/O. Together these make up **machine language**, the lowest-level language of computing.

The CPU continuously runs a **fetch-decode-execute** loop:

1. **Fetch** — Retrieve the instruction addressed by the **Instruction Pointer (IP)** / Program Counter (PC).
2. **Decode** — Interpret the **opcode**, operands, and optional prefixes.
3. **Execute** — The **Control Unit (CU)** dispatches signals to functional units like the **ALU**, which performs the operation.

Modern CPUs accelerate this loop with **instruction pipelining**, **branch prediction**, and **speculative execution**.

### Opcodes in Practice

In an 8-bit machine where instructions are 8 bits:

```text
LOAD 0101   → 00110101
INCREMENT   → 1000
```

The first example uses the first 4 bits as the opcode and the second 4 bits as the operand.

Opcodes are the atoms of computing, so they are catalogued in **opcode tables**, such as x86 opcode references.

### Assembly Language

Because raw bit-patterns are hard to remember, **assembly language** assigns readable symbols to opcodes:

```text
00110101  →  LOAD 0101
```

An **assembler** translates assembly back into machine code.

Assembly is high-level relative to machine code, but low-level relative to C, Rust, or Python. “High-level” and “low-level” are relative terms describing abstraction.

### Memory Layout

When a program runs, it becomes a **process** with key memory regions:

| Region | Purpose |
|---|---|
| Static | Global variables and constants |
| Stack | Function frames and local variables; automatic LIFO memory |
| Heap | Dynamically allocated data shared across functions and threads |

Mental model:

| Memory | Analogy | Lifetime | Management |
|---|---|---|---|
| Stack | Notepad / cafeteria trays | Function call | Automatic |
| Heap | Whiteboard / parking lot | Until deleted | Manual or runtime-managed |

Objects often live on the heap because they may need to outlive the function that created them.

---

## II. Language Hierarchy & Translation

A programming language is structured text with syntax and semantics. Source code needs:

1. A **translator** — converts it to another language or representation.
2. An **executor** — runs the translated commands.

### Levels of Abstraction

```text
Machine Language → Assembly → IR → Bytecode → Source Language
     lowest                                      highest
```

| Level | Description |
|---|---|
| Machine Language | Raw opcode bit patterns executed by the CPU |
| Assembly | Human-readable opcode symbols translated by an assembler |
| IR | Intermediate Representation between source and assembly |
| Bytecode | VM-targeted IR executed by a virtual machine |
| Source Language | Human-facing programming language |

Converting from one IR level to another is often called **lowering**.

### The Compiler

A **compiler** is any program that translates Language A into Language B.

Core compiler components:

| Component | Role |
|---|---|
| Frontend | Converts source text into tokens, AST, and semantic information |
| Backend / Code Generator | Converts AST or IR into bytecode, LLVM IR, assembly, or machine code |

| Translation Type | When It Happens | Examples |
|---|---|---|
| AOT | Before execution | `rustc`, `gcc` |
| JIT | During execution | V8, PyPy, JVM JIT |
| Transpiler | Source-to-source | TypeScript → JavaScript |

### Virtual Machines

Hardware instructions are vendor-specific. A **Virtual Machine (VM)** abstracts hardware details so bytecode can run anywhere the VM exists.

Example: Java compiles to JVM bytecode. Any valid Java bytecode can run on any platform with a compatible Java Runtime Environment.

---

## III. Execution Pipeline & AST

### The Pipeline

```text
Source → Tokens → AST → Output
```

For `1 + 2`:

1. **Grammar** — Rules defining valid syntax.
2. **Lexer / Tokenizer** — Converts text into tokens: `[1, +, 2]`.
3. **Parser** — Builds an AST with `+` as the root.
4. **Interpreter / Evaluator** — Walks the AST and computes the result.

### AST Mental Model

Source code is like a sentence. The AST is its grammar diagram.

```text
  +
 / \
1   2
```

The AST captures structure, not just text.

### Interpreter & Recursion

To evaluate `1 + 2`:

1. Evaluate left side → `1`.
2. Evaluate right side → `2`.
3. Apply operator → `3`.

If the left side were `(3 + 4)`, evaluation recursively descends into that subtree first.

Trees are powerful because **structure determines evaluation order**.

### State & Functions

A calculator is mostly stateless. A real language is a **state machine**.

| Feature | Purpose |
|---|---|
| Variables | Named storage |
| Functions | Abstraction and reuse |
| Parameters | Data flow |
| Conditionals | Branching |
| Operators | Computation |
| Recursion | Self-reference |
| Call stack | Tracks active function calls |

### Function Call Sequence

For:

```text
add(3, 4)
```

The runtime does:

1. Look up function.
2. Evaluate arguments.
3. Push a new frame.
4. Bind parameters.
5. Execute body.
6. Pop frame and return result.

A frame is essentially a scoped map of variable names to values.

### The REPL

A **Read-Eval-Print Loop** works like this:

1. Read input.
2. Evaluate it.
3. Print result.
4. Loop.

A REPL with variables, functions, conditionals, recursion, and a call stack is already a real programming language.

---

## IV. Bytecode & Virtual Machines

**Bytecode** sits between source and assembly. It is lower-level than source but higher-level than machine code.

### Why Bytecode?

| Approach | Trade-off |
|---|---|
| Direct AST interpretation | Simple but slow |
| Bytecode + VM | Faster and portable |
| JIT to native code | Fastest but complex |

Bytecode is compact, cache-friendly, and portable.

Examples:

| Language | Execution Model |
|---|---|
| Python / CPython | Source → bytecode → VM |
| Java | Source → JVM bytecode → JVM |
| Ruby / YARV | Source → bytecode → VM |

### Stack Machine

A **stack machine** uses:

| Component | Role |
|---|---|
| Stack array | Stores intermediate values |
| IP | Instruction pointer |
| SP | Stack pointer |

For:

```text
(1 + 2) * (3 + 4)
```

Execution:

```text
push 1
push 2
add       → [3]

push 3
push 4
add       → [3, 7]

mul       → [21]
```

Every operation pops inputs and pushes output.

### VM Execution of `1 + 2`

```text
OpConstant(1)  → push 1
OpConstant(2)  → push 2
OpAdd          → pop 2, pop 1, push 3
OpPop          → return 3
```

The VM also follows fetch-decode-execute:

1. Fetch opcode at `instructions[ip]`.
2. Decode opcode.
3. Execute stack operation.
4. Increment IP.

---

## V. Types & Type Inference

### Static vs Dynamic Typing

Types are contracts.

| Approach | When Checked | Examples |
|---|---|---|
| Static | Compile time | Rust, C, Java, Haskell |
| Dynamic | Runtime | Python, JavaScript, Ruby |

Trade-offs:

| Static Typing | Dynamic Typing |
|---|---|
| Catches bugs early | Faster to prototype |
| Enables better performance | More flexible |
| Requires more upfront structure | Errors can hide until runtime |

### Types Enable Fast Code

If the compiler knows `x` and `y` are `i64`, it can generate one CPU instruction for multiplication.

Without static types, a dynamic interpreter must:

1. Check runtime type of `x`.
2. Check runtime type of `y`.
3. Resolve the operation.
4. Validate compatibility.
5. Execute or raise an error.

This overhead is why dynamic languages can be much slower for numeric-heavy workloads.

### Type Inference

**Inference** means the compiler deduces types automatically.

```text
let x = 1 + 2
```

The compiler infers:

```text
x: Int
```

Mental model: type inference is like solving a crossword puzzle. Known types fill in unknowns through constraints.

### Key Mechanisms

| Mechanism | Role |
|---|---|
| Unification | Checks compatibility and resolves unknowns |
| Type Environment | Maps names to types |
| Constraints | Rules created by expressions |
| Scoping | Allows shadowing and local bindings |

### Two-Pass Function Type Checking

Functions can call each other, including mutual recursion, so checking often needs two passes:

1. **Collect signatures** — Record all function names and types.
2. **Check bodies** — Type-check function implementations.

Per-expression pattern:

```text
type-check children → apply rule → assign type to node
```

### Inference Approaches

| Approach | Description |
|---|---|
| Hindley-Milner | Infers polymorphic types with minimal annotations |
| Local Inference | Requires annotations at boundaries, infers inside bodies |

---

## VI. Compiler Optimizations

Optimizations simplify code before or during code generation.

```text
AST → Constant Folding → Algebraic Simplification → DCE → Optimized AST
```

| Optimization | Example |
|---|---|
| Constant Folding | `1 + 2` → `3` |
| Algebraic Simplification | `x * 0` → `0` |
| Dead Code Elimination | Remove unreachable branches |
| Common Subexpression Elimination | Reuse repeated computations |
| Loop Unrolling | Replace loop with repeated body |
| Inlining | Replace call with function body |
| Tail Call Optimization | Convert tail recursion into loop |

Even when LLVM optimizes later, custom passes are useful because they improve debug output and exploit language-specific knowledge.

---

## VII. LLVM IR & Advanced Code Generation

LLVM IR is a universal low-level assembly-like language not tied to one CPU.

Many languages target LLVM, including Rust, Swift, Julia, and Kotlin/Native.

### SSA

LLVM uses **Static Single Assignment** form: each value is assigned once.

```llvm
%1 = add i64 3, 4
%2 = mul i64 %1, 2
```

SSA makes optimization easier because each value has one definition.

### Mutability: Alloca / Load / Store

Because SSA values are immutable, mutable variables use stack slots.

```llvm
%x.addr = alloca i64
store i64 5, i64* %x.addr
%x = load i64, i64* %x.addr
```

LLVM’s `mem2reg` pass can promote many stack slots into SSA registers.

### Basic Blocks & Branching

Conditionals require basic blocks:

```llvm
entry:
  %cmp = icmp sgt i64 %a, %b
  br i1 %cmp, label %then, label %else

then:
  br label %merge

else:
  br label %merge

merge:
  ret i64 0
```

Every basic block must end with a terminator such as `ret` or `br`.

### Phi Nodes

When a value depends on which branch was taken, LLVM uses a **phi node**:

```llvm
%x = phi i64 [ %x.then, %then ], [ %x.else, %else ]
```

Phi nodes merge values from different control-flow paths.

### LLVM State

| Component | Role |
|---|---|
| Context | Workspace for LLVM objects |
| Module | Container for functions and globals |
| Builder | Inserts IR instructions |
| `variables` map | Maps names to stack pointers |
| `functions` map | Maps names to LLVM functions |
| `current_fn` | Current function being compiled |

### Three-Pass Compilation

1. Declare function signatures.
2. Compile function bodies.
3. Create `@__main` wrapper.

Full pipeline:

```text
Source → Parse → Type Check → Optimize → Codegen → LLVM IR → JIT → Execute
```

### LLVM Optimization Passes

| Pass | What It Does |
|---|---|
| `mem2reg` | Promotes stack slots to SSA registers |
| `dce` | Removes dead code |
| `instcombine` | Combines and simplifies instructions |
| `simplifycfg` | Simplifies control flow |

Preset pipelines:

| Level | Description |
|---|---|
| `default<O0>` | No optimization |
| `default<O1>` | Light optimization |
| `default<O2>` | Standard optimization |
| `default<O3>` | Aggressive optimization |

CLI examples:

```bash
thirdlang examples/point.tl
thirdlang -O examples/point.tl
thirdlang --passes "mem2reg,dce" examples/point.tl
thirdlang --ir examples/point.tl
thirdlang --ir -O examples/point.tl
thirdlang --passes "default<O2>" examples/point.tl
```

---

## VIII. Object-Oriented Concepts: Classes

### Why Classes?

> Think of a filing cabinet. Secondlang gives you individual papers (`int`, `bool`). But eventually you want folders that group related papers together. Classes are those folders.

Without classes, related data is scattered:

```text
distance(x1, y1, x2, y2)
```

With classes:

```text
p1.distance_squared(p2)
```

Classes group related data and behavior into one unit.

### Problems Without Classes

| Problem | Explanation |
|---|---|
| Easy to mix up | Coordinates can be passed in the wrong order |
| No grouping | The language does not know `x` and `y` belong together |
| Verbose | Every function needs all pieces separately |
| No encapsulation | Data and behavior are separated |

### Benefits With Classes

| Benefit | Explanation |
|---|---|
| Organized | Data and behavior are grouped |
| Safer | Objects prevent accidental mixing |
| Readable | Method syntax expresses intent |
| Encapsulated | Implementation details live inside the class |

### OOP Vocabulary

| Concept | Description | Example |
|---|---|---|
| Class | Blueprint for objects | `class Point { ... }` |
| Object | Instance of a class | `p = new Point(1, 2)` |
| Field | Data stored in object | `self.x` |
| Method | Function attached to class | `def distance(self)` |
| Constructor | Initializes object | `def __init__(self)` |
| Destructor | Cleans up before deletion | `def __del__(self)` |

### Classes as Custom Types

Classes define new types.

```text
class Point {
    x: int
    y: int
}

def move(p: Point, dx: int, dy: int) -> Point {
    ...
}
```

`Point` is now a first-class type like `int` or `bool`.

This is a **nominal type system**: types are identified by name.

### Thirdlang vs Secondlang

| Concept | Secondlang | Thirdlang |
|---|---|---|
| Types | `int`, `bool` | `int`, `bool`, `ClassName` |
| User-defined types | None | Classes with fields |
| Functions | Functions only | Functions + methods |
| Memory | Stack only | Stack + heap |
| Object creation | None | `new ClassName(args)` |
| Object destruction | None | `delete obj` |
| LLVM types | `i64`, `i1` | `i64`, `i1`, struct types |

### Design Decisions

Thirdlang implements a simple OOP subset.

Included:

| Feature | Example |
|---|---|
| Class definitions | `class Point { ... }` |
| Fields | `x: int` |
| Methods | `def get_x(self) -> int` |
| Constructor | `def __init__(self)` |
| Destructor | `def __del__(self)` |
| Object creation | `new Point(1, 2)` |
| Object deletion | `delete p` |
| Field access | `p.x`, `self.x` |
| Method calls | `p.method()` |
| Classes as types | `other: Point` |

Excluded:

| Feature | Why Excluded |
|---|---|
| Inheritance | Requires vtables and dynamic dispatch |
| Interfaces / Traits | Requires trait objects or generics |
| Visibility | Everything public for simplicity |
| Static methods | Requires separate dispatch |
| Operator overloading | Requires special method resolution |

### No Inheritance

Inheritance is skipped because:

1. It adds complexity.
2. It requires dynamic dispatch for virtual methods.
3. Composition is often simpler.
4. Core OOP concepts are clearer without it.

### Explicit Memory Management

Thirdlang uses explicit `new` and `delete`:

```text
p = new Point(1, 2)
delete p
```

This teaches:

1. Objects live on the heap.
2. Memory must be freed manually.
3. Forgetting to free causes leaks.
4. Using freed memory is undefined behavior.

---

## IX. Thirdlang Syntax & AST

### Top-Level Items

Programs now contain both class definitions and statements:

```rust
pub enum TopLevel {
    Class(ClassDef),
    Stmt(Stmt),
}
```

A program becomes:

```rust
Vec<TopLevel>
```

instead of:

```rust
Vec<Stmt>
```

### Class Structure

A `ClassDef` contains:

| Field | Purpose |
|---|---|
| `name` | Class name |
| `fields` | Field definitions |
| `methods` | Method definitions |

### Class Grammar Concepts

| Concept | Meaning |
|---|---|
| `ClassDef` | Entire class: `class Name { body }` |
| `ClassBody` | Fields and methods |
| `FieldDef` | Field declaration |
| `MethodDef` | Method declaration |
| `SelfParam` | Required `self` parameter |
| `MethodParams` | Parameters after `self` |

Methods must have `self` as their first parameter.

### New Expressions

Thirdlang adds:

| Syntax | Meaning |
|---|---|
| `new ClassName(args)` | Allocate and initialize object |
| `delete obj` | Destroy and free object |
| `obj.field` | Field access |
| `obj.method(args)` | Method call |
| `self` | Current object inside method |

### Assignment Targets

Assignments can target variables or fields:

```rust
pub enum AssignTarget {
    Var(String),
    Field {
        object: Box<TypedExpr>,
        field: String,
    },
}
```

Examples:

```text
x = 10
self.x = 10
obj.field = value
```

### Expression Enum Additions

```rust
pub enum Expr {
    Int(i64),
    Bool(bool),
    Var(String),
    SelfRef,

    Unary {
        op: UnaryOp,
        expr: Box<TypedExpr>,
    },

    Binary {
        op: BinaryOp,
        left: Box<TypedExpr>,
        right: Box<TypedExpr>,
    },

    Call {
        name: String,
        args: Vec<TypedExpr>,
    },

    MethodCall {
        object: Box<TypedExpr>,
        method: String,
        args: Vec<TypedExpr>,
    },

    FieldAccess {
        object: Box<TypedExpr>,
        field: String,
    },

    New {
        class: String,
        args: Vec<TypedExpr>,
    },

    If {
        cond: Box<TypedExpr>,
        then_branch: Vec<Stmt>,
        else_branch: Vec<Stmt>,
    },

    While {
        cond: Box<TypedExpr>,
        body: Vec<Stmt>,
    },

    Block(Vec<Stmt>),
}
```

New variants:

| Variant | Purpose |
|---|---|
| `SelfRef` | The `self` keyword |
| `New` | Object creation |
| `FieldAccess` | Reading object fields |
| `MethodCall` | Calling object methods |

### Parsing a Class

For:

```text
class Point {
    x: int
    y: int

    def __init__(self, x: int, y: int) {
        self.x = x
        self.y = y
    }
}
```

Parser steps:

1. Match `class`.
2. Read identifier `Point`.
3. Parse class body.
4. Collect fields.
5. Collect methods.
6. Skip storing `self` in the method parameter list.
7. Return `ClassDef`.

### Parsing Method Body

For:

```text
self.x = x
```

The parser creates:

```text
AssignTarget::Field {
    object: SelfRef,
    field: "x"
}
```

Value:

```text
Expr::Var("x")
```

`self` is implicit in method metadata. The type checker knows every method receives `self` of the current class type.

---

## X. Thirdlang Type System

### Class Metadata

Classes need metadata for type checking:

```rust
pub struct ClassInfo {
    pub name: String,
    pub fields: HashMap<String, Type>,
    pub field_order: Vec<String>,
    pub methods: HashMap<String, MethodInfo>,
    pub has_destructor: bool,
}
```

Method metadata:

```rust
pub struct MethodInfo {
    pub name: String,
    pub params: Vec<(String, Type)>,
    pub return_type: Type,
    pub is_constructor: bool,
    pub is_destructor: bool,
}
```

Important fields:

| Field | Purpose |
|---|---|
| `fields` | Maps field names to types |
| `field_order` | Determines memory layout |
| `methods` | Maps method names to signatures |
| `has_destructor` | Tracks whether `__del__` exists |

### Type Enum

```rust
pub enum Type {
    Int,
    Bool,
    Class(String),
    Function {
        params: Vec<Type>,
        ret: Box<Type>,
    },
    Method {
        class: String,
        params: Vec<Type>,
        ret: Box<Type>,
    },
    Unit,
    Unknown,
}
```

Class types are represented as:

```rust
Type::Class("Point".to_string())
```

### Type Checking Passes

Thirdlang type checking uses multiple passes:

1. Register all classes.
2. Collect function signatures.
3. Type-check class bodies.
4. Type-check top-level statements.

This supports forward references, methods, and class-aware checking.

### Type Checking `new`

For:

```text
new Point(10, 20)
```

The type checker verifies:

1. Class exists.
2. Constructor exists if arguments are provided.
3. Argument count matches constructor parameters.
4. Argument types match.
5. Expression type is `Type::Class("Point")`.

### Type Checking `self`

`self` is valid only inside methods.

```text
self.x
```

If used outside a method, it is a type error.

### Type Checking Method Calls

For:

```text
object.method(args)
```

The checker:

1. Type-checks `object`.
2. Confirms object has a class type.
3. Looks up method in class metadata.
4. Checks argument count.
5. Checks argument types.
6. Assigns method return type.

Example:

```text
p1.distance_squared(p2)  // OK
p1.distance_squared(5)   // ERROR: expected Point, got int
p1.nonexistent()         // ERROR: method not found
```

### Type Checking Field Access

For:

```text
object.field
```

The checker:

1. Type-checks object.
2. Confirms object is a class type.
3. Looks up field in class metadata.
4. Returns field type.

---

## XI. Constructors & Object Creation

### Constructor Rules

A constructor:

| Rule | Description |
|---|---|
| Name | Always `__init__` |
| First parameter | Always `self` |
| Return type | Implicit `Unit` |
| Purpose | Initialize fields |

Example:

```text
class Point {
    x: int
    y: int

    def __init__(self, x: int, y: int) {
        self.x = x
        self.y = y
    }
}
```

### Constructor Parameters

For:

```text
new Point(10, 20)
```

Inside `__init__`:

| Parameter | Value |
|---|---|
| `self` | Newly allocated object |
| `x` | `10` |
| `y` | `20` |

### Object Creation Flow

For:

```text
p = new Point(10, 20)
```

Thirdlang does:

1. Calculate object size.
2. Call `malloc`.
3. Zero-initialize fields.
4. Call constructor.
5. Return pointer.

### Memory Layout

```text
class Point {
    x: int    // offset 0, 8 bytes
    y: int    // offset 8, 8 bytes
}
```

LLVM:

```llvm
%Point = type { i64, i64 }
```

Field order matters because it determines memory offsets.

### Constructors Without Parameters

```text
class Counter {
    count: int

    def __init__(self) {
        self.count = 0
    }
}

c = new Counter()
```

The constructor still receives `self`, but no extra arguments.

### Constructor Patterns

Default values:

```text
class Config {
    value: int
    enabled: bool

    def __init__(self) {
        self.value = 42
        self.enabled = true
    }
}
```

Computed initialization:

```text
class Square {
    side: int
    area: int

    def __init__(self, side: int) {
        self.side = side
        self.area = side * side
    }
}
```

Validation by clamping:

```text
class PositiveInt {
    value: int

    def __init__(self, v: int) {
        if (v < 0) {
            self.value = 0
        } else {
            self.value = v
        }
    }
}
```

Thirdlang constructors cannot fail. Real languages may use exceptions, `Result`, `Option`, or factory methods.

---

## XII. Methods & `self`

### Method Definition

Methods are functions inside classes.

Every method:

1. Starts with `def`.
2. Has `self` as first parameter.
3. May take extra parameters.
4. May return a value.

Example:

```text
def get_x(self) -> int {
    return self.x
}
```

### The `self` Parameter

`self` is a pointer/reference to the object the method was called on.

```text
c = new Counter()
c.increment()
```

Inside `increment`, `self == c`.

Comparison:

| Language | Self / This |
|---|---|
| Python | Explicit `self` |
| Rust | Explicit `self` / `&self` |
| Java | Implicit `this` |
| C++ | Implicit `this` |
| Thirdlang | Explicit `self` |

Explicit `self` makes object state access clear.

### Method Calls

```text
p.get_x()
```

Compiles as:

```llvm
call i64 @Point__get_x(ptr %p)
```

Methods are regular functions with the object passed as the first argument.

### Method Naming

| Method | LLVM Function |
|---|---|
| `Point.__init__` | `@Point____init__` |
| `Point.get_x` | `@Point__get_x` |
| `Counter.increment` | `@Counter__increment` |

This avoids collisions between classes.

### Methods With Parameters

```text
class Point {
    x: int
    y: int

    def distance_squared(self, other: Point) -> int {
        dx = self.x - other.x
        dy = self.y - other.y
        return dx * dx + dy * dy
    }
}
```

Objects are passed by pointer, not copied.

This means modifying another object through a parameter modifies the original object.

### Reference Semantics

```text
q = p
delete p
q.x
```

`q` points to the same object as `p`, so after `delete p`, `q` is dangling.

### Calling Methods on `self`

```text
class Calculator {
    value: int

    def __init__(self) {
        self.value = 0
    }

    def add(self, n: int) {
        self.value = self.value + n
    }

    def double(self) {
        self.add(self.value)
    }
}
```

### Returning `self`

```text
class Builder {
    value: int

    def __init__(self) {
        self.value = 0
    }

    def set_value(self, v: int) -> Builder {
        self.value = v
        return self
    }
}
```

Returning `self` enables builder-style patterns.

---

## XIII. Field Access & LLVM GEP

### Field Reads

Thirdlang:

```text
self.x
```

LLVM:

```llvm
%x_ptr = getelementptr %Point, ptr %self, i32 0, i32 0
%x = load i64, ptr %x_ptr
```

### Field Writes

Thirdlang:

```text
self.x = 42
```

LLVM:

```llvm
%x_ptr = getelementptr %Point, ptr %self, i32 0, i32 0
store i64 42, ptr %x_ptr
```

### GEP Mental Model

`getelementptr` does not load memory. It computes the address of a field.

It is pointer arithmetic that understands struct layout.

---

## XIV. Memory Management in Thirdlang

### Stack vs Heap

| Memory | Management | Speed | Size | Lifetime |
|---|---|---|---|---|
| Stack | Automatic | Fast | Limited | Function scope |
| Heap | Manual | Slower | Large | Until freed |

Secondlang uses stack-only values.

Thirdlang adds heap-allocated objects.

### Why Heap Allocation?

Objects live on the heap because:

1. They can outlive the function that created them.
2. Multiple variables can reference the same object.
3. This mirrors many OOP languages.
4. Object size and lifetime are more flexible.

### `new`

For:

```text
p = new Point(10, 20)
```

Behind the scenes:

1. Calculate size.
2. Call `malloc`.
3. Initialize fields.
4. Call constructor.
5. Return pointer.

LLVM:

```llvm
%ptr = call ptr @malloc(i64 16)
call void @Point____init__(ptr %ptr, i64 10, i64 20)
```

### `delete`

For:

```text
delete p
```

Behind the scenes:

1. Call destructor if it exists.
2. Call `free`.
3. Leave pointer invalid.

LLVM:

```llvm
call void @Point____del__(ptr %p)
call void @free(ptr %p)
```

### Destructor Rules

A destructor:

| Rule | Description |
|---|---|
| Name | `__del__` |
| Parameters | Only `self` |
| Return type | `Unit` |
| Called by | `delete` |
| Purpose | Cleanup before memory is freed |

Example:

```text
class Resource {
    id: int

    def __init__(self, id: int) {
        self.id = id
    }

    def __del__(self) {
        // cleanup
    }
}
```

### Memory Layout Trace

For:

```text
p = new Point(10, 20)
delete p
```

Allocation:

1. `malloc(16)` returns address `0x1000`.
2. Constructor sets `x = 10` at offset `0`.
3. Constructor sets `y = 20` at offset `8`.
4. `p` stores `0x1000`.

Deletion:

1. Destructor runs on `0x1000`.
2. `free(0x1000)` returns memory to OS.
3. `p` still stores `0x1000`, but the address is invalid.

### Memory Safety Bugs

| Bug | Description | Example |
|---|---|---|
| Memory leak | Forgot to delete | `p = new Point(1,2)` and never freed |
| Use after free | Access after delete | `delete p; p.x` |
| Double free | Delete twice | `delete p; delete p` |
| Dangling pointer | Another reference points to freed object | `q = p; delete p; q.x` |

Examples:

```text
def leak() {
    p = new Point(1, 2)
    // forgot delete
}

p = new Point(1, 2)
delete p
p.x

delete p
delete p

q = p
delete p
q.x
```

### Best Practices

| Practice | Reason |
|---|---|
| Delete what you allocate | Prevent leaks |
| One owner | Clarifies who frees memory |
| Null after delete | Prevent accidental reuse, if language supports null |

Example:

```text
def make_point() -> Point {
    return new Point(1, 2)
}

p = make_point()
delete p
```

### Real Memory Management Approaches

| Approach | Examples | Pros | Cons |
|---|---|---|---|
| Manual | C, C++ | Fast, predictable | Error-prone |
| Garbage Collection | Java, Python, Go | Safe, convenient | Runtime overhead |
| Reference Counting | Swift, CPython | Predictable cleanup | Cycles, overhead |
| Ownership | Rust | Safe, no runtime cost | Complex rules |
| Smart Pointers | C++ | Safer manual memory | Requires discipline |

Thirdlang does not implement these because they are complex, but it teaches why they exist.

---

## XV. LLVM Codegen for Classes

### Key Insight

```text
Classes become structs.
Objects become heap pointers.
Methods become functions.
Fields become struct elements.
```

### Compilation Phases

1. Declare libc functions: `malloc`, `free`.
2. Create class struct types.
3. Declare method signatures.
4. Compile method bodies.
5. Compile top-level code into `__main`.
6. Verify module.

### LLVM Mapping

| Thirdlang | LLVM IR |
|---|---|
| `class Point { x: int }` | `%Point = type { i64 }` |
| `new Point(10)` | `malloc` + `Point____init__` |
| `p.x` | `getelementptr` + `load` |
| `p.x = 5` | `getelementptr` + `store` |
| `p.method()` | `call @Point__method(ptr %p)` |
| `delete p` | `Point____del__` + `free` |

### Class Structs

```text
class Counter {
    count: int
}
```

LLVM:

```llvm
%Counter = type { i64 }
```

### Method as Function

```text
def get_x(self) -> int {
    return self.x
}
```

LLVM:

```llvm
define i64 @Point__get_x(ptr %self) {
entry:
    %x_ptr = getelementptr %Point, ptr %self, i32 0, i32 0
    %x = load i64, ptr %x_ptr
    ret i64 %x
}
```

### `new` Codegen

For:

```text
new Counter()
```

Compiler emits:

1. Get struct type.
2. Compute object size.
3. Call `malloc`.
4. Zero-initialize fields.
5. Call constructor.
6. Return pointer.

### Method Call Codegen

For:

```text
obj.method(arg)
```

Compiler emits:

1. Compile object expression.
2. Extract object pointer.
3. Determine class name.
4. Build function name: `ClassName__method`.
5. Pass object pointer first.
6. Compile and pass remaining arguments.
7. Emit LLVM call.

### Complete Counter Example

Source:

```text
class Counter {
    count: int

    def __init__(self) {
        self.count = 0
    }

    def increment(self) -> int {
        self.count = self.count + 1
        return self.count
    }
}

c = new Counter()
c.increment()
```

LLVM:

```llvm
%Counter = type { i64 }

define void @Counter____init__(ptr %self) {
entry:
    %count_ptr = getelementptr %Counter, ptr %self, i32 0, i32 0
    store i64 0, ptr %count_ptr
    ret void
}

define i64 @Counter__increment(ptr %self) {
entry:
    %count_ptr = getelementptr %Counter, ptr %self, i32 0, i32 0
    %count = load i64, ptr %count_ptr
    %new_count = add i64 %count, 1
    store i64 %new_count, ptr %count_ptr
    %result = load i64, ptr %count_ptr
    ret i64 %result
}

define i64 @__main() {
entry:
    %raw = call ptr @malloc(i64 8)
    call void @Counter____init__(ptr %raw)
    %result = call i64 @Counter__increment(ptr %raw)
    ret i64 %result
}
```

### JIT Execution

Execution flow:

1. Create LLVM context, module, and builder.
2. Compile AST to LLVM IR.
3. Create JIT engine.
4. Look up `__main`.
5. Execute native machine code.

The code is not interpreted at this stage. LLVM JIT compiles it to native code for the host CPU.

### Performance Considerations

Thirdlang does:

| Feature | Performance |
|---|---|
| Direct field access | Fast |
| Static method calls | No vtable overhead |
| Contiguous object layout | Cache-friendly |

Real compilers add:

| Optimization | Purpose |
|---|---|
| Method inlining | Remove call overhead |
| Escape analysis | Stack-allocate non-escaping objects |
| Field alignment | Improve memory access |
| Dead field elimination | Remove unused fields |
| Devirtualization | Convert dynamic calls to static calls |

---

## XVI. Optimizing LLVM IR

### Why Optimize?

Naive codegen is correct but inefficient.

It may create:

1. Too many `alloca`s.
2. Redundant loads.
3. Redundant stores.
4. Repeated GEPs.
5. Dead instructions.

LLVM optimization passes simplify this.

### Important Passes

| Pass | Purpose |
|---|---|
| `mem2reg` | Promotes stack allocations to SSA registers |
| `dce` | Removes unused instructions |
| `instcombine` | Combines and simplifies instructions |
| `simplifycfg` | Simplifies control flow |
| `gvn` | Removes equivalent repeated computations |

### Optimization Example

Source:

```text
class Counter {
    value: int

    def increment(self) -> int {
        self.value = self.value + 1
        return self.value
    }
}
```

Unoptimized IR may contain:

```llvm
%self1 = alloca ptr
store ptr %self, ptr %self1
%self2 = load ptr, ptr %self1
...
%self3 = load ptr, ptr %self1
...
%self5 = load ptr, ptr %self1
...
```

After `mem2reg`:

```llvm
%self is used directly
```

After `instcombine`:

```text
redundant GEPs are simplified
```

After `dce`:

```text
unnecessary final loads are removed
```

Final optimized shape:

```llvm
define i64 @Counter__increment(ptr %self) {
entry:
  %field = load i64, ptr %self
  %add = add i64 %field, 1
  store i64 %add, ptr %self
  ret i64 %add
}
```

This can reduce a method from roughly 14 instructions to 4.

### CLI Usage

```bash
thirdlang examples/point.tl
thirdlang -O examples/point.tl
thirdlang --passes "mem2reg,dce" examples/point.tl
thirdlang --ir examples/point.tl
thirdlang --ir -O examples/point.tl
thirdlang --passes "default<O2>" examples/point.tl
```

### Optimization Levels

| Level | Description |
|---|---|
| `default<O0>` | No optimization |
| `default<O1>` | Light optimization |
| `default<O2>` | Standard optimization |
| `default<O3>` | Aggressive optimization |

### Testing Optimization

Test that:

1. Optimized IR is shorter.
2. Unoptimized IR contains expected `alloca`s.
3. Optimized IR preserves program output.
4. Regression tests catch accidental IR changes.

---

## XVII. Debugging the Compiler Pipeline

### Core Debugging Strategy

Your compiler is a pipeline:

```text
Source → Tokens → AST → Type Check → IR → Output
```

When something breaks, find which stage first produced the wrong result.

### Systematic Approach

1. Reproduce with the smallest input.
2. Isolate the failing stage.
3. Inspect that stage’s output.
4. Fix the logic.
5. Add a regression test.

### Debugging Table

| Symptom | Action |
|---|---|
| Parse error | Inspect lexer tokens |
| Wrong result | Print AST |
| Type error wrong | Inspect inferred types |
| Bad IR | Print generated LLVM |
| Crash | Reduce input and run debugger |
| Infinite loop | Add tracing inside eval loop |
| Precedence bug | Check AST root |

### Debugging Tips

1. Test each feature in isolation.
2. Use the REPL for quick experiments.
3. Print ASTs frequently.
4. Check operator precedence by AST structure.
5. Add regression tests for every fixed bug.

---

## XVIII. Testing the Language

### Testing Pyramid

| Level | What to Test |
|---|---|
| Unit | Parser helpers, type rules, optimizer functions |
| Integration | Parse + typecheck + codegen |
| End-to-end | Source program → final result |
| Snapshot | IR output and error messages |
| Fuzzing | Parser edge cases |

### Practical Strategy

1. Start with integration tests for full programs.
2. Add unit tests for complex logic.
3. Snapshot generated IR and diagnostics.
4. Fuzz parser inputs.
5. Add regression tests for every bug.

Testing takes effort but saves debugging time.

---

## XIX. Language Progression

### Four Stages

| Feature | Calculator | Firstlang | Secondlang | Thirdlang |
|---|---|---|---|---|
| Grammar size | ~18 lines | ~70 lines | ~77 lines | ~140 lines |
| Type system | None | Dynamic | Static | Static + classes |
| Variables | No | Yes | Yes | Yes |
| Functions | No | Yes | Yes | Yes + methods |
| Classes | No | No | No | Yes |
| Memory | Stack | Stack | Stack | Stack + heap |
| Execution | Interpreter / VM / JIT | Interpreter | LLVM JIT | LLVM JIT |

Progression:

1. **Calculator** — Parsing, AST, evaluation.
2. **Firstlang** — Variables, functions, control flow.
3. **Secondlang** — Static types and LLVM compilation.
4. **Thirdlang** — Classes, objects, heap allocation, manual memory.

---

## XX. What Thirdlang Built

### Grammar

Adds:

1. Classes.
2. Fields.
3. Methods.
4. Constructors.
5. Destructors.
6. `new`.
7. `delete`.
8. Field access.
9. Method calls.

### Type System

Adds:

1. `Type::Class(name)`.
2. `ClassInfo`.
3. Method type checking.
4. Field type checking.
5. Constructor checking.
6. Object type inference.

### Parser

Adds parsing for:

1. `class`.
2. `self`.
3. `new`.
4. `delete`.
5. `obj.field`.
6. `obj.method(args)`.

### Type Checker

Adds:

1. Class registration.
2. Method resolution.
3. Field lookup.
4. Constructor validation.
5. Destructor validation.

### Code Generator

Adds:

1. LLVM struct types.
2. `malloc` / `free`.
3. Method compilation.
4. GEP field access.
5. Constructor calls.
6. Destructor calls.
7. LLVM pass manager integration.

---

## XXI. What to Explore Next

### Language Features

| Feature | Concepts |
|---|---|
| Inheritance | Vtables, dynamic dispatch |
| Interfaces / Traits | Polymorphism |
| Generics | Type parameters |
| Closures | Captured environments |
| Pattern Matching | Structured branching |
| Algebraic Data Types | `Option`, `Result`, enums with data |

### Type System Features

| Feature | Concepts |
|---|---|
| Nullability | `Point?` vs `Point` |
| Ownership | Move semantics |
| Borrowing | References without ownership |
| Hindley-Milner | Full type inference |
| ADTs | Product and sum types |

### Memory Management

| Feature | Concepts |
|---|---|
| Garbage Collection | Mark-and-sweep, generational GC |
| Reference Counting | Deterministic cleanup |
| Ownership | Rust-style compile-time safety |
| Smart Pointers | Safer C++ memory management |

### Execution Models

| Model | Description |
|---|---|
| AOT | Compile to standalone executable |
| Bytecode VM | Compile to bytecode and interpret |
| JIT | Compile hot code at runtime |
| Transpilation | Compile to JS, C, WASM, etc. |

### Optimizations

| Optimization | Example |
|---|---|
| Constant Folding | `1 + 2 * 3` → `7` |
| Inlining | Replace call with body |
| Escape Analysis | Stack-allocate non-escaping objects |
| Devirtualization | Replace dynamic dispatch with static call |
| Dead Code Elimination | Remove unreachable code |

### Tooling

| Tool | Purpose |
|---|---|
| REPL | Interactive execution |
| Debugger | Step through programs |
| Formatter | Consistent style |
| LSP | IDE support |
| Error Reporter | Helpful diagnostics |

---

## XXII. Exercise Ideas

### Add Static Methods

Goal:

```text
Math.max(5, 10)
```

Requirements:

1. Parse `static`.
2. Static methods do not require `self`.
3. Use `ClassName.method(args)` syntax.
4. No object allocation needed.

### Add Default Field Values

Goal:

```text
new Counter().get()
```

Requirements:

1. Store default in `FieldDef`.
2. Initialize fields before constructor.
3. Let constructor override defaults.

### Add Getter / Setter Sugar

Goal:

```text
p.x
p.x = 5
```

Could lower to:

```text
p.get_x()
p.set_x(5)
```

Requirements:

1. Parse `{ get; set; }`.
2. Generate getter and setter methods.
3. Rewrite field access to method calls.

### Add Simplified Inheritance

Goal:

```text
new Dog().speak()
```

Requirements:

1. Store parent fields first.
2. Search child methods before parent methods.
3. Use static dispatch first.
4. Add vtables later for dynamic dispatch.

### Add Null / Optional Type

Goal:

```text
p: Point? = null
```

Requirements:

1. Add `Type::Optional(Box<Type>)`.
2. Represent null as pointer `0x0`.
3. Require null checks before access.
4. Optionally add `?.` safe navigation.

### Add Generics

Goal:

```text
class Box<T> {
    value: T
}
```

Requires either:

| Strategy | Meaning |
|---|---|
| Monomorphization | Generate specialized code per type |
| Type erasure | Use runtime representation |

### Add Strings

Requires:

1. String literals.
2. String type.
3. Heap allocation.
4. Length tracking.
5. Concatenation.
6. Comparison.
7. Indexing.

### Add Arrays / Lists

Requires:

1. Array syntax.
2. Array type.
3. Bounds checking.
4. Dynamic resizing for lists.

### Add Closures

Example:

```text
def make_adder(n: int) -> (int) -> int {
    return def(x: int) -> int {
        return x + n
    }
}
```

Requires:

1. First-class function types.
2. Captured environments.
3. Function pointer + environment representation.

### Add Pattern Matching

Example:

```text
match value {
    0 => "zero",
    1 => "one",
    n => "other"
}
```

With object destructuring:

```text
match point {
    Point { x: 0, y } => "on y-axis",
    Point { x, y: 0 } => "on x-axis",
    _ => "elsewhere"
}
```

### Add ADTs

```text
enum Option<T> {
    Some(T),
    None
}

enum Result<T, E> {
    Ok(T),
    Err(E)
}
```

Pattern matching + ADTs is one of the most powerful language-design combinations.

---

## XXIII. Better Errors

### Source Locations

Track line and column numbers.

Example:

```text
Error at line 5, column 10:
    x = 1 + true
            ^^^^ expected int, got bool
```

### Error Recovery

Instead of stopping at the first error, continue parsing:

```text
Error 1: Undefined variable 'foo' at line 3
Error 2: Type mismatch at line 7
Error 3: Missing semicolon at line 12
```

### Helpful Diagnostics

Good errors explain:

1. What went wrong.
2. Where it happened.
3. What was expected.
4. How to fix it.

Example:

```text
Error: Cannot add int and bool
  --> example.tl:5:9
   |
 5 |     x = 1 + true
   |         ^^^^^^^^
   |
   = hint: did you mean to compare? Try `1 == true` or `1 != true`
```

---

## XXIV. Final Mental Model

A language implementation is structured transformation:

```text
Source
  ↓
Characters
  ↓
Tokens
  ↓
AST
  ↓
Typed AST
  ↓
Optimized AST / IR
  ↓
LLVM IR or Bytecode
  ↓
Machine Code or VM Execution
  ↓
Behavior
```

Or compressed:

```text
Text → Trees → Types → IR → Bytes → Machine Code → Electrons
```

The same ideas appear everywhere:

| Domain | Compiler-Like Concepts |
|---|---|
| SQL | Query parsing and planning |
| GraphQL | Schema + query execution |
| YAML / TOML | Parsing structured config |
| Regex | Pattern language + VM |
| CSS | Selector parsing and matching |
| Template Engines | Embedded DSLs |
| Build Systems | Dependency graphs |
| Shells | Parsing and execution |

All paths lead to the same truth:

> Computing is structured transformation — from text, to trees, to bytes, to machine code, to electrons.
