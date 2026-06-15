# Programming & Compiler Mental Notes

---

## I. Hardware Fundamentals

### The CPU

A **CPU** is an implementation of an **Instruction Set Architecture (ISA)** — an abstract model defining data types, registers, hardware support, and I/O. Together these make up **Machine Language**, the lowest-level language of computing.

The CPU continuously runs a **fetch-decode-execute** loop:

1. **Fetch** — Retrieve the instruction addressed by the **Instruction Pointer (IP)** / Program Counter (PC).
2. **Decode** — Interpret the **opcode** (a unique encoding for an operation — the "atoms of computing"), plus any operands (arguments) and optional prefix (behavioral modifier).
3. **Execute** — The **Control Unit (CU)** dispatches signals to functional units like the **ALU** (Arithmetic Logic Unit), which performs the operation on register values. Results are written back to memory if applicable.

> Modern CPUs accelerate this loop with **instruction pipelining** and **speculative execution**.

---

### Opcodes in Practice

In an 8-bit machine where instructions are 8 bits:
- `LOAD 0101` → `00110101` — the first 4 bits (`0011`) are the opcode for *load*, the second 4 bits (`0101`) are the operand.
- `INCREMENT` → `1000` — the opcode for *increment by 1*, no operand needed.

Since opcodes are the atoms of computing, they are catalogued in an **opcode table** (e.g., the [x86 opcode reference](https://www.felixcloutier.com/x86/)).

---

### Assembly Language

Because bit-patterns are hard to remember, **Assembly Language** assigns abstract human-readable symbols to opcodes:

```
00110101  →  LOAD 0101
```

A utility program called an **Assembler** translates Assembly back to Machine Language.

> Assembly is *high-level* relative to machine code, but *low-level* relative to C, Rust, or Python. High-level and low-level are **relative terms** that convey the amount of abstraction involved.

---

### Memory Layout

When a program is loaded into memory, it creates a **process** with three key regions:

| Region | Purpose |
|---|---|
| **Static** | Global variables and constants |
| **Stack** | Function frames and local variables — auto-managed, LIFO |
| **Heap** | Dynamically allocated data shared across functions and threads |

- **Stack analogy:** A notepad — write, tear off when done.
- **Heap analogy:** A whiteboard — write, stays until you explicitly erase it.

Objects live on the heap because they need to *outlive* the function that created them.

---

## II. The Language Hierarchy & Translation

Informally, a **language** is structured text with syntax and semantics. Source code written in a programming language needs:
1. A **translator** of some sort — converts it to another language/format.
2. An **executor** of some sort — runs the translated commands to produce output.

### Levels of Abstraction

```
Machine Language  →  Assembly  →  IR  →  Bytecode  →  Source Language
     (lowest)                                              (highest)
```

| Level | Description |
|---|---|
| **Machine Language** | Raw bit-pattern opcodes directly executed by the CPU |
| **Assembly Language** | Human-readable symbols mapped to opcodes; translated by an Assembler |
| **IR (Intermediate Representation)** | Any format between source and assembly; converting between IRs is called *lowering* |
| **Bytecode** | An IR emulating a simplified instruction set; executed by a Virtual Machine (VM) |

---

### The Compiler

A **compiler** is any program that translates Language A → Language B. Its two core components:

- **Frontend** — Maps source code strings to an **Abstract Syntax Tree (AST)**.
- **Backend (Code Generator)** — Translates the AST to Bytecode, IR, or Assembly.

| Translation Type | When It Happens | Examples |
|---|---|---|
| **AOT (Ahead-of-Time)** | Before execution | `rustc`, `gcc` |
| **JIT (Just-in-Time)** | During execution | V8 (JS), PyPy (Python) |
| **Transpiler** | Source-to-source | Python → Java |

---

### Virtual Machines (VMs)

Hardware instructions are vendor-specific — Intel and AMD instructions differ. A **Virtual Machine (VM)** abstracts away hardware details so that code compiled to the VM's language becomes **platform-agnostic**.

The most famous example: the **Java Virtual Machine (JVM)**. Any valid Java Bytecode runs on any platform with a **Java Runtime Environment (JRE)**, regardless of where it was compiled.

---

## III. The Execution Pipeline & AST

### The Pipeline

```
Source → Tokens → AST → Output
```

Walking through `1 + 2`:

1. **Grammar** — Rules defining valid syntax. `1 + 2` matches; `+ + 1` doesn't.
2. **Lexer (Tokenizer)** — Breaks source into meaningful chunks: `"1 + 2"` → `[1, +, 2]`. Tracks source location for error messages.
3. **Parser** — Builds a tree structure from tokens: `+` at the root, `1` and `2` as children — this is the **AST**.
4. **Interpreter/Evaluator** — Walks the AST recursively and computes the result.

> **AST analogy:** Think of source code as a sentence and the AST as its grammatical diagram. Just as "The cat sat" is diagrammed into subject/verb, `1 + 2` is diagrammed into left/operator/right. The AST captures **structure**, not just text.

---

### The Interpreter & Recursion

The CPU is the *ultimate* interpreter — it executes opcodes one at a time. Our software interpreter does the same at a higher level by **walking the AST recursively**:

To evaluate `1 + 2`:
1. Evaluate left (`1`) → `1`
2. Evaluate right (`2`) → `2`
3. Apply operator (`+`) → `3`

If the left side were `(3 + 4)`, we'd recursively evaluate it first. This is why trees are powerful: **structure determines evaluation order**. Parse → AST → recursive eval is the foundation of every interpreter — Python, Ruby, JavaScript all do this with more node types.

---

### State & Functions

A calculator is **stateless** — input goes in, output comes out, nothing persists. A real language is a **state machine**:
- **Variables** — Named storage (memory).
- **Conditionals** — Branching (`if`/`else`).
- **Loops** — Repetition (`while`).

The AST becomes a *program to execute*, not just an expression to evaluate.

---

### Function Call Sequence

For `add(3, 4)`:

1. **Look up** — Find the function value by name.
2. **Evaluate arguments** — Compute `3` and `4`.
3. **Push frame** — Create a fresh environment (scope) on the call stack.
4. **Bind parameters** — `a = 3`, `b = 4`.
5. **Execute body** — Run function instructions.
6. **Pop frame** — Return result to the caller; the frame below becomes current again.

> **Call stack analogy:** A stack of sticky notes. Each call writes variables on a new note and puts it on top. Returning tears it off. This is why `inner()`'s variables don't overwrite `outer()`'s — they're on different notes.

Each frame is just a `HashMap` pushed and popped like any stack. **Grammar grows**, **AST nodes multiply**, but execution is still recursive tree traversal — now with scoped state.

---

### The REPL

A **Read-Eval-Print Loop** provides an interactive environment:

1. **Read** — Get a line of input.
2. **Eval** — Parse and execute it.
3. **Print** — Show the result.
4. **Loop** — Return to step 1.

A working REPL with variables, functions, conditionals, operators, recursion, and a call stack means you have a **real programming language**.

---

## IV. Bytecode & Virtual Machines

**Bytecode** sits between source and assembly — lower-level than source, higher-level than machine code. It emulates an instruction set with a new, simplified encoding, and is executed by a **VM**.

### Why Bytecode?

| Approach | Trade-off |
|---|---|
| **Direct AST interpretation** | Simple to implement, but slow |
| **Bytecode + VM** | Moderate complexity, significantly faster, highly portable |
| **JIT to native machine code** | Fastest, but complex (LLVM dependency, platform-specific) |

- **Simpler than JIT** — No LLVM dependency; works everywhere.
- **Faster than AST** — Bytecode is compact and cache-friendly.
- **Portable** — Same bytecode runs on any machine with your VM.

This is how Python (CPython), Java (JVM), and Ruby (YARV) work: compile source to bytecode once, run the bytecode interpreter wherever you need it.

---

### The Stack Machine

A **stack machine** has two components:
- A **stack array** for intermediate values.
- An **Instruction Pointer (IP)** and **Stack Pointer (SP)**.

**Why stacks?** They handle *any* nesting automatically. For `(1 + 2) * (3 + 4)`:
- Push 1, push 2, add → stack: `[3]`
- Push 3, push 4, add → stack: `[3, 7]`
- Multiply → stack: `[21]`

Every operation pops its inputs and pushes its output. The stack naturally tracks what's "in progress."

**VM execution of `1 + 2` (fetch-decode-execute):**

```
OpConstant(1)  →  push 1       [stack: 1      ]
OpConstant(2)  →  push 2       [stack: 1, 2   ]
OpAdd          →  pop 2, pop 1, push 3  [stack: 3]
OpPop          →  return 3
```

The VM reads instructions left-to-right with the IP:
1. **Fetch** — Read next byte from `bytecode.instructions[ip]`
2. **Decode** — Match on the opcode
3. **Execute** — Manipulate the stack
4. **Repeat** — Increment IP; continue until out of instructions

---

## V. Types & Type Inference

### Static vs. Dynamic Typing

Types act as **contracts** — when you write `a: int`, you promise `a` will always be an integer. The compiler enforces that promise.

| Approach | When Checked | Examples |
|---|---|---|
| **Static** | Compile time (before running) | Rust, C, Java, Haskell |
| **Dynamic** | Runtime (while running) | Python, JavaScript, Ruby |

**Trade-offs:**
- **Static** — Catches bugs early, enables better performance, requires upfront annotations.
- **Dynamic** — More flexible, faster to prototype, but bugs can hide until runtime.

---

### Types Enable Fast Code

When the compiler knows `x` and `y` are `i64`, it generates a **single CPU instruction** for `x * y`.

Without types, a dynamic interpreter must at runtime:
1. Check what type `x` is.
2. Look up the multiplication operation for that type.
3. Check operand compatibility.
4. Handle potential type errors.
5. Finally, multiply.

This overhead makes dynamically typed languages **10–100× slower** for numeric work. This is why JIT compilers like **V8** (JavaScript) and **PyPy** (Python) invest heavily in *type speculation* — guessing types to generate fast paths.

---

### Type Inference

**Inference** means the compiler deduces types automatically — no annotations needed in most cases. Types **flow forward** from known sources (literals, annotated parameters) through operations into variables.

```
let x = 1 + 2   // x inferred as Int — no annotation needed
```

> **Type inference analogy:** Like solving a crossword puzzle. Some squares have letters (explicit annotations); others are blank (`Unknown`). Constraints like "this is added to an int, so it must be int" fill in the blanks.

Key mechanisms:

- **Unification** — Checks if two types are compatible and finds a common type. Resolving `Unknown` with a concrete type is how the compiler *learns* what an unknown type should be.
- **Type Environment** — A `HashMap<String, Type>` mapping names to types. Extended on declaration, queried on reference, scoped to allow shadowing.

---

### Two-Pass Function Type Checking

Functions can call each other (mutual recursion), requiring two passes:

1. **Pass 1 — Collect Signatures** — Scan all function definitions and record their type signatures *before* checking any bodies. This ensures `foo` can call `bar` even if `bar` is defined later.
2. **Pass 2 — Check Bodies** — Walk each function body, inferring and unifying types throughout.

**Per-expression pattern:** recursively type-check sub-expressions → apply typing rule → set type on this node.

Type inference works by:
1. Starting with **known types** — literals (`42` → `Int`, `true` → `Bool`) and annotated parameters.
2. **Flowing types** through expressions — operators, calls, assignments.
3. **Recording** types in the environment so variables can be looked up later.
4. **Unifying** types — checking compatibility and resolving `Unknown`.
5. **Reporting errors** when types conflict.

---

### Type Inference Approaches

| Approach | Description |
|---|---|
| **Hindley-Milner** | Infers polymorphic types like `fn identity<T>(x: T) -> T` without any annotations |
| **Local Inference** | Requires annotations at function boundaries; infers types *within* function bodies |

---

## VI. Compiler Optimizations

Optimizations simplify the AST before code generation, producing faster output, cleaner debug trees, and less work for the backend. They are chained into a **pass pipeline** — order matters!

```
AST → [Constant Folding] → [Algebraic Simplification] → [Dead Code Elimination] → Optimized AST
```

| Optimization | Description |
|---|---|
| **Constant Folding** | `1 + 2` → `3` at compile time |
| **Algebraic Simplification** | `x * 0` → `0`, strength reduction |
| **Dead Code Elimination** | Remove unreachable branches |
| **Common Subexpression Elimination** | Compute identical sub-expressions once |
| **Loop Unrolling** | Replace loops with repeated sequential code |
| **Inlining** | Substitute function body directly at call site |
| **Tail Call Optimization** | Convert tail recursion into a flat loop |

> Even when LLVM will optimize downstream, custom passes improve: compile speed, debug output readability, and can exploit language-specific knowledge LLVM can't.

---

## VII. LLVM IR & Advanced Code Generation

> LLVM is like a universal translator for CPUs. You speak LLVM IR; LLVM translates it to x86, ARM, WebAssembly — whatever you need. Write your compiler frontend once; LLVM gives you every platform for free.

**LLVM IR** is a universal, low-level assembly language not tied to any specific CPU. It is used by Rust, Swift, Julia, Kotlin/Native, and more. By targeting LLVM IR, you get world-class optimizations for free.

---

### Core IR Mechanics

#### SSA (Static Single Assignment)

In LLVM IR, every variable is assigned **exactly once**. This makes optimizations like dead code elimination and constant propagation trivial — the compiler always knows exactly where each value was defined.

```llvm
%1 = add i64 3, 4     ; %1 is assigned once
%2 = mul i64 %1, 2    ; %2 is assigned once
```

#### Mutability: Alloca / Load / Store

Because SSA values are immutable, mutable variables use stack slots:

```llvm
%x.addr = alloca i64          ; reserve stack space
store i64 5, i64* %x.addr    ; write (mutate)
%x = load i64, i64* %x.addr  ; read
```

LLVM's **`mem2reg` pass** promotes these stack slots to fast registers automatically. This pattern is simple to generate and LLVM optimizes it away.

#### Basic Blocks & Branching

Conditionals require separate **basic blocks** (`entry`, `then`, `else`, `merge`). Each block ends with a **terminator** — either `ret` (return) or `br` (branch):

```llvm
entry:
  %cmp = icmp sgt i64 %a, %b          ; signed greater than → i1 boolean
  br i1 %cmp, label %then, label %else

then:
  ...
  br label %merge

else:
  ...
  br label %merge

merge:
  ...
```

#### Phi Nodes (φ)

In SSA form, if a variable's value depends on which branch was taken, a **phi node** selects the correct value based on the incoming basic block:

```llvm
merge:
  %x = phi i64 [ %x.then, %then ], [ %x.else, %else ]
```

"If we came from `%then`, use `%x.then`. If we came from `%else`, use `%x.else`." Phi nodes are the **only** way to merge values from different control flow paths in SSA.

#### Recursion

Recursive functions use the standard `call` instruction — calling the same function from within itself:

```llvm
%result = call i64 @fib(i64 %n_minus_1)
```

---

### LLVM State

LLVM maintains state via:

| Component | Role |
|---|---|
| **Context** | Workspace for all LLVM objects |
| **Module** | Container for functions (one per compilation unit) |
| **Builder** | Inserts IR instructions into a basic block |
| **`variables` map** | Maps variable names → stack pointers (`alloca` results) |
| **`functions` map** | Maps function names → LLVM function objects |
| **`current_fn`** | The function being compiled (needed to create new basic blocks) |

---

### The Three-Pass Compilation Process

1. **Pass 1 — Declare Functions** — Announce all function signatures to LLVM before compiling any body. Enables mutual recursion via forward references.
2. **Pass 2 — Compile Bodies** — Generate IR instructions for each function body using the alloca/load/store pattern for variables.
3. **Pass 3 — Create `@__main` Wrapper** — Wrap top-level expressions (e.g., `fib(10)`) in a `__main` function as the JIT entry point, then verify the module.

**Full pipeline:**
```
Source → Parse → Type Check → Optimize → Codegen → LLVM IR → JIT → Execute
```

**JIT execution:**
1. Create a JIT execution engine from the verified module.
2. Get a function pointer to `@__main`.
3. Call it — LLVM compiles IR to native machine code on the fly and executes it.

> The `unsafe` block required when calling JIT output signals that we are invoking raw machine code — we must trust that our code generator produced valid IR.

---

## VIII. Object-Oriented Concepts

Classes provide four core benefits:

| Benefit | Description |
|---|---|
| **Grouping** | Related data lives together in one structure |
| **Methods** | Functions that explicitly operate on the grouped data |
| **Encapsulation** | Data and behavior bundled in one place |
| **New Types** | `Point` becomes a first-class type just like `int` |

---

## IX. Debugging

### Compiler Pipeline Debugging

> Your language is a pipeline: `Source → Tokens → AST → Output`. When something breaks, find which stage produced the wrong output.

**Systematic approach:**

1. **Reproduce** — Find the smallest input that triggers the bug.
2. **Isolate** — Which stage is producing wrong output?
3. **Inspect** — Print the data at that stage.
4. **Fix** — Change the code.
5. **Verify** — Re-run the test.

**Common symptoms and remedies:**

| Symptom | Action |
|---|---|
| Parse error | Simplify input; inspect lexer tokens |
| Wrong result | Print the AST; check grammar operator precedence |
| Program crash | Add debug prints; run `cargo clippy` |
| Infinite loop | Add print statements inside the `eval` loop |
| Precedence wrong | Print AST and verify grammar rule order |

**Debugging tips:**
- Test each feature in isolation — don't write 100 lines then debug.
- Use the REPL for quick experiments.
- Print the AST — structure reveals bugs that output doesn't.
- Check operator precedence by inspecting which node is the AST root.

---

### Memory Safety Bugs (Manual Memory Management)

Languages without garbage collection (C, C++) require explicit `new`/`delete`, introducing high-risk classes of bugs:

| Bug | Description | Example |
|---|---|---|
| **Memory Leak** | Forgot `delete`; memory consumed until program exits | `p = new Point(1,2)` — never freed |
| **Use After Free** | Accessing memory after `delete` — **undefined behavior** | `delete p; p.x` |
| **Double Free** | Calling `delete` twice — corrupts memory allocator | `delete p; delete p` |
| **Dangling Pointer** | Two variables point to the same object; one deletes it — the other points to dead memory | `q = p; delete p; q.x` |

```cpp
// Memory Leak
def leak() {
    p = new Point(1, 2)
    // Oops — forgot delete p!
}  // Memory lost forever

// Use After Free
p = new Point(1, 2)
delete p
p.x   // BUG: undefined behavior

// Double Free
delete p
delete p  // BUG: undefined behavior

// Dangling Pointer
q = p
delete p
q.x   // BUG: q now points to freed memory
```

---

*All paths lead to the same truth: computing is structured transformation — from text, to trees, to bytes, to machine code, to electrons.*
