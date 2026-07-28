# 🧠 Programming & Compiler Mental Notes

> A practical map from source text to syntax trees, bytecode, machine code, memory, and
> network protocols—with the same ideas viewed from both the builder's and reverse
> engineer's perspective.

![Markdown](https://img.shields.io/badge/format-Markdown-blue?style=flat-square) ![Rust](https://img.shields.io/badge/primary%20language-Rust-orange?style=flat-square) ![Scope](https://img.shields.io/badge/scope-compilers%20%7C%20systems%20%7C%20networking-informational?style=flat-square) ![Status](https://img.shields.io/badge/status-living%20document-brightgreen?style=flat-square)

## 🗺️ How to Use These Notes

These notes are organized as a **first-pass curriculum** and a **long-term reference**.
On the first read, move top to bottom; afterward, use the goal index to jump directly to
the layer you are building or debugging:

- 🏗️ **Building a language:** follow grammar → AST → interpreter → types → IR → code
  generation → runtime.
- ⚙️ **Learning low-level systems:** follow CPU → assembly → memory → executable formats →
  ABI → networking.
- 🔍 **Reverse engineering:** read the same pipeline backward, starting with bytes,
  instructions, runtime state, and observable behavior.
- 🐛 **Debugging:** reduce the problem to one stage, inspect that stage's input and output,
  and record the smallest reproducible example.

### Recommended Top-to-Bottom Reading Order

The document is arranged as a dependency ladder. Read these phases **in order** on a
first pass; use the goal index afterward as a reference.

| Phase                           | Build understanding in this order                                   | Why it comes here                                  |
| ------------------------------- | ------------------------------------------------------------------- | -------------------------------------------------- |
| **1 · Foundations**             | grammar → CPU/memory → translation levels → CS concept map          | Establish what software eventually controls        |
| **2 · Language front end**      | hand-built parser → Pest PEGs → AST                                 | Turn text into trusted structure                   |
| **3 · Execution & runtimes**    | recursion → bytecode → hosted runtime → Lua case study              | Give syntax behavior, state, calls, and allocation |
| **4 · Semantics & native code** | types → optimization → LLVM → Inkwell → classes → debugging         | Define meaning, emit native code, and verify it    |
| **5 · Safe implementation**     | low-level memory → idiomatic Rust → patterns → FP → algorithms → ML | Build reliable tools and reusable reasoning habits |
| **6 · Machine & OS boundary**   | processes/VM → executables/linking/ABI → bare metal → concurrency   | Explain how compiled artifacts actually run        |
| **7 · Connected systems**       | system design → sockets/protocols → `smoltcp` → QUIC                | Move from one process to bounded distributed state |
| **8 · UI & applications**       | browser engine → Actix/Wasm/DOM → SDL → Bevy/Diesel/Tauri           | Apply the earlier models at host and UI boundaries |
| **9 · Read programs backward**  | x64 triage → CTF/Nightmare → authorized memory-tooling labs         | Recover hidden behavior using all prior layers     |
| **10 · Daily practice**         | terminal/Git/builds → Python automation → workflow → progression    | Make the knowledge repeatable                      |

> 🧭 **First-pass rule:** when a later section mentions an unfamiliar earlier concept,
> follow its link backward, learn that prerequisite, then return. The later sections
> deliberately reuse the same ideas—**state machines, ownership, representation,
> invariants, and trust boundaries**—at larger scales.

### Jump to a Goal

| Goal                        | Start here                                                                            | Then continue to                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| 🧱 Build a parser           | [Grammar](#-grammar)                                                                  | [Building a Parser](#-building-a-parser)                                              |
| 🦗 Design a PEG grammar     | [Pest in Practice](#-pest-peg-parsing-in-practice)                                    | [Execution Pipeline](#-iii-the-execution-pipeline--ast)                               |
| 🌳 Build an interpreter     | [Execution Pipeline](#-iii-the-execution-pipeline--ast)                               | [Bytecode & VMs](#-iv-bytecode--virtual-machines)                                     |
| 🏠 Build a hosted runtime   | [Hosted Runtime Design](#-hosted-language-runtime-design-in-rust)                     | [Minimal Lua Interpreter](#-building-a-minimal-lua-interpreter-in-rust)               |
| 🏷️ Add static types         | [Types & Type Inference](#-v-types--type-inference)                                   | [LLVM IR](#-vii-llvm-ir--advanced-code-generation)                                    |
| 🏭 Test LLVM codegen        | [Inkwell Test Patterns](#-testing-llvm-codegen-with-inkwell)                          | [Compiler Debugging](#compiler-pipeline-debugging)                                    |
| 🧠 Understand memory        | [Memory Layout](#memory-layout)                                                       | [Low-Level Memory](#-low-level-memory-binary-analysis--language-runtime-architecture) |
| 🖥️ Understand the OS        | [Processes & the Kernel](#-operating-systems-from-the-bottom-up)                      | [Executables & Linking](#-executable-files-linkers-abis--ffi)                         |
| 🔍 Reverse x64 Windows      | [x64 Windows Reversing](#-practical-x64-windows-reverse-engineering)                  | [Calling Conventions](#6-calling-conventions)                                         |
| 🧩 Reverse a binary or VM   | [Reverse Engineering](#reverse-engineering--debugging-compiled-binaries)              | [Reversing a Binary](#7-reversing-a-binary-bytecode-vm-or-language)                   |
| 🦀 Write safer Rust tools   | [Idiomatic Rust](#-idiomatic-rust-for-compilers-parsers-and-low-level-tools)          | [Networking](#-networking-protocols--wire-formats)                                    |
| 🐍 Translate Python to Rust | [Rust/Python Idioms](#9-rust-and-python-similar-task-different-contract)              | [Functional Rust](#λ-functional-programming-in-rust)                                  |
| λ Learn functional Rust     | [Functional Rust](#λ-functional-programming-in-rust)                                  | [Algorithms](#-algorithms-in-idiomatic-rust)                                          |
| 🧮 Practice algorithms      | [Algorithms in Rust](#-algorithms-in-idiomatic-rust)                                  | [Compiler Applications](#13-where-algorithms-appear-in-language-tools)                |
| 🤖 Understand ML            | [Machine Learning](#-machine-learning-foundations)                                    | [System Design](#-system-design-patterns-in-rust)                                     |
| 🏗️ Design a service         | [System Design in Rust](#-system-design-patterns-in-rust)                             | [Networking](#-networking-protocols--wire-formats)                                    |
| 🔩 Write low-level Rust     | [Executables & Linking](#-executable-files-linkers-abis--ffi)                         | [Bare Metal & Unsafe](#-bare-metal-unsafe-rust--chromium-scale-integration)           |
| 🧱 Design Rust APIs         | [Idiomatic Rust](#-idiomatic-rust-for-compilers-parsers-and-low-level-tools)          | [Rust Patterns & Recipes](#-practical-rust-patterns--cookbook-recipes)                |
| 📡 Embed a TCP/IP stack     | [smoltcp](#-embedded-networking-with-smoltcp)                                         | [QUIC with Quinn](#-quic-networking-with-quinn)                                       |
| ⚡ Learn QUIC               | [QUIC with Quinn](#-quic-networking-with-quinn)                                       | [Browser Engines](#-how-browser-engines-turn-bytes-into-pixels)                       |
| 🕸 Build Rust web code       | [Rust Web Boundaries](#-rust-web-boundaries-actix-web--wasm-bindgen)                  | [Applied Rust](#-applied-rust-architectures-bevy-diesel--tauri)                       |
| 🌳 Control the browser DOM  | [DOM from Rust](#14-the-dom-is-a-host-owned-object-graph)                             | [Wasm Ownership](#17-dom-event-listeners-need-lifetime-design)                        |
| 🌐 Understand a browser     | [Browser Engines](#-how-browser-engines-turn-bytes-into-pixels)                       | [Rust Web Boundaries](#-rust-web-boundaries-actix-web--wasm-bindgen)                  |
| 🎮 Build a native loop      | [SDL3](#-native-event-loops--multimedia-with-sdl3)                                    | [Bevy](#1-bevy-data-oriented-application-structure)                                   |
| 🚀 Ship a Rust service      | [Zero to Production](#4-zero-to-production-service-engineering)                       | [Actix Web](#2-actix-web-application-shape)                                           |
| 🧰 Build applied Rust apps  | [Applied Rust](#-applied-rust-architectures-bevy-diesel--tauri)                       | [Practical Workflow](#-practical-low-level-workflow)                                  |
| 🛡️ Study memory tooling     | [Low-Level Memory](#-low-level-memory-binary-analysis--language-runtime-architecture) | [Authorized Memory Labs](#19-authorized-memory-inspection--tooling-labs)              |
| ⌨️ Master developer tools   | [Developer Tooling](#-developer-tooling-the-missing-semester)                         | [Practical Workflow](#-practical-low-level-workflow)                                  |
| 🐍 Automate repetitive work | [Practical Python Automation](#-practical-python-automation)                          | [Practical Workflow](#-practical-low-level-workflow)                                  |

### Reading Conventions

- 📖 **Definition** introduces a concept.
- 🧠 **Mental model** gives a useful approximation, not a formal guarantee.
- 🧭 **Invariant** is a condition that must remain true for an implementation to be
  correct.
- 🚧 **Boundary** is where data changes representation or trust level.
- ⚠️ **Failure mode** names what can go wrong and why.
- 🦀 **Rust examples** favor clarity and explicit checks over cleverness.

---

## 📚 Table of Contents

<details>
<summary>Click to expand all 41 sections</summary>

1. [🗺️ How to Use These Notes](#-how-to-use-these-notes)
2. [🧩 Grammar](#-grammar)
3. [🔧 I. Hardware Fundamentals](#-i-hardware-fundamentals)
4. [🪜 II. The Language Hierarchy & Translation](#-ii-the-language-hierarchy--translation)
5. [🧭 A Computer Science Concept Map](#-a-computer-science-concept-map)
6. [🧱 Building a Parser](#-building-a-parser)
7. [🦗 Pest PEG Parsing in Practice](#-pest-peg-parsing-in-practice)
8. [🌳 III. The Execution Pipeline & AST](#-iii-the-execution-pipeline--ast)
9. [🔁 Recursion](#-recursion)
10. [🧮 IV. Bytecode & Virtual Machines](#-iv-bytecode--virtual-machines)
11. [🏠 Hosted Language Runtime Design in Rust](#-hosted-language-runtime-design-in-rust)
12. [🌙 Building a Minimal Lua Interpreter in Rust](#-building-a-minimal-lua-interpreter-in-rust)
13. [🔖 V. Types & Type Inference](#-v-types--type-inference)
14. [🚀 VI. Compiler Optimizations](#-vi-compiler-optimizations)
15. [🏭 VII. LLVM IR & Advanced Code Generation](#-vii-llvm-ir--advanced-code-generation)
16. [🧪 Testing LLVM Codegen with Inkwell](#-testing-llvm-codegen-with-inkwell)
17. [📦 VIII. Object-Oriented Concepts (Classes)](#-viii-object-oriented-concepts-classes)
18. [🐛 IX. Debugging](#-ix-debugging)
19. [🧠 Low-Level Memory, Binary Analysis & Language Runtime Architecture](#-low-level-memory-binary-analysis--language-runtime-architecture)
20. [🦀 Idiomatic Rust for Compilers, Parsers, and Low-Level Tools](#-idiomatic-rust-for-compilers-parsers-and-low-level-tools)
21. [🧱 Practical Rust Patterns & Cookbook Recipes](#-practical-rust-patterns--cookbook-recipes)
22. [λ Functional Programming in Rust](#λ-functional-programming-in-rust)
23. [🧮 Algorithms in Idiomatic Rust](#-algorithms-in-idiomatic-rust)
24. [🤖 Machine Learning Foundations](#-machine-learning-foundations)
25. [💻 Operating Systems from the Bottom Up](#-operating-systems-from-the-bottom-up)
26. [🔗 Executable Files, Linkers, ABIs & FFI](#-executable-files-linkers-abis--ffi)
27. [🔩 Bare-Metal, Unsafe Rust & Chromium-Scale Integration](#-bare-metal-unsafe-rust--chromium-scale-integration)
28. [🧵 Concurrency, Atomics & Memory Models](#-concurrency-atomics--memory-models)
29. [🏢 System Design Patterns in Rust](#-system-design-patterns-in-rust)
30. [🌐 Networking, Protocols & Wire Formats](#-networking-protocols--wire-formats)
31. [📡 Embedded Networking with `smoltcp`](#-embedded-networking-with-smoltcp)
32. [⚡ QUIC Networking with Quinn](#-quic-networking-with-quinn)
33. [🌐 How Browser Engines Turn Bytes into Pixels](#-how-browser-engines-turn-bytes-into-pixels)
34. [🕸 Rust Web Boundaries: Actix Web & wasm-bindgen](#-rust-web-boundaries-actix-web--wasm-bindgen)
35. [🎮 Native Event Loops & Multimedia with SDL3](#-native-event-loops--multimedia-with-sdl3)
36. [🧰 Applied Rust Architectures: Bevy, Diesel & Tauri](#-applied-rust-architectures-bevy-diesel--tauri)
37. [🔍 Practical x64 Windows Reverse Engineering](#-practical-x64-windows-reverse-engineering)
38. [🔨 Developer Tooling: The Missing Semester](#-developer-tooling-the-missing-semester)
39. [🐍 Practical Python Automation](#-practical-python-automation)
40. [🧰 Practical Low-Level Workflow](#-practical-low-level-workflow)
41. [🛤️ X. Language Progression](#-x-language-progression)

</details>

---

## 🧩 Grammar

A _**formal grammar**_ is a set of rules that define what makes valid code in a
programming language. Think of it like the grammar of English: “The cat sat” is valid,
but “Cat the sat” is not.

> 🧠 **Mental model:** a grammar is the language's structural contract; the parser is
> the program that checks that contract and turns matching text into structure.

For programming languages, a grammar specifies:

- **What tokens are valid** - keywords like `def`, `if`, `return` ; _operators_ like
  `+`, `-`, `*`; _literals_ like `42`, `true`
- **How tokens can be combined** - `1 + 2` is valid, `+ + 1` is not
- **The structure of programs** - functions contain statements, statements contain
  expressions

---

## 🔧 I. Hardware Fundamentals

### The CPU

If you want to create a “computer” from scratch, you need to start by defining an
**abstract model** for your computer. This _abstract model_ is also referred to as
_**Instruction Set Architecture (ISA)**_

A **CPU** is an implementation of an **Instruction Set Architecture (ISA)** — an
abstract model defining data types, registers, hardware support, and I/O. Together these
make up **Machine Language**, the lowest-level language of computing.

The CPU continuously runs a **fetch-decode-execute** loop:

> 🔁 **Core loop:** **fetch** an instruction, **decode** what it means, then
> **execute** its effect.

1. **Fetch** — Retrieve the instruction addressed by the **Instruction Pointer (IP)** /
   Program Counter (PC).
2. **Decode** — Interpret the **opcode** (a unique encoding for an operation — the
   "atoms of computing"), plus any operands (arguments) and optional prefix (behavioral
   modifier).
3. **Execute** — The **Control Unit (CU)** dispatches signals to functional units like
   the **ALU** (Arithmetic Logic Unit), which performs the operation on register values.
   Results are written back to memory if applicable.

> Modern CPUs accelerate this loop with **instruction pipelining** and **speculative
> execution**.

---

### Number Systems: Binary, Decimal, Hexadecimal

CPUs are built from circuits that are either **on** or **off** — so at the hardware
level, everything is represented in **binary (base-2)**: only the digits `0` and `1`.

| System          | Base | Digits           | Example |
| --------------- | ---- | ---------------- | ------- |
| **Binary**      | 2    | `0`, `1`         | `1101`  |
| **Decimal**     | 10   | `0`–`9`          | `126`   |
| **Hexadecimal** | 16   | `0`–`9`, `A`–`F` | `0xA1D` |

Each system is just a different way of writing the same value — e.g. `1101` in binary is
`13` in decimal. Binary gets unwieldy fast (the decimal number `250` is `11111010` in
binary), so **hexadecimal** (prefixed `0x`) is the standard shorthand in computing: each
hex digit maps cleanly to exactly 4 binary bits, so `250` is simply `0xFA`. This is why
opcodes, memory addresses, and register dumps are almost always shown in hex.

---

### Opcodes in Practice

Instructions are comprised of instruction code (aka operation code, in short **opcode**
or _p-code_), directly executed by the _CPU_. An opcode can either **have operand(s)**
or **no operand**.

In an 8-bit machine where instructions are 8 bits:

- `LOAD 0101` → `00110101` — the first 4 bits (`0011`) are the opcode for _load_, the
  second 4 bits (`0101`) are the operand.
- `INCREMENT` → `1000` — the opcode for _increment by 1_, no operand needed.

Since opcodes are the atoms of computing, they are presented in an **opcode table**
(e.g., the [x86 opcode reference](https://www.felixcloutier.com/x86/)).

**A few opcodes worth knowing by name:**

- **`nop`** (`0x90` on x86, "no operation") — the CPU does nothing and advances to the
  next instruction. It's a useful placeholder, and understanding it makes reading
  disassembly less intimidating: not every instruction needs to matter.
- **`cmp`** — compares two values.
- **`jmp` / `je` / `jne`** ("jump", "jump if equal", "jump if not equal") —
  conditionally redirect execution to a different address instead of continuing
  sequentially. This mechanism — compare, then conditionally jump — is called
  **branching**, and it's the opcode-level foundation of every `if`/`else` you write in
  a high-level language.

---

### Assembly Language

Because bit-patterns are hard to remember, **Assembly Language** assigns abstract,
human-readable symbols to opcodes matching their operations by name:

```
00110101  →  LOAD 0101
```

Here, `0011` is abstracted to the symbol `LOAD`, so `00110101` can be written as
`LOAD 0101` — a higher-level, human-readable form of the same instruction.

A utility program called an **Assembler** translates Assembly back to Machine Language.

> Assembly is _high-level_ relative to machine code, but _low-level_ relative to C,
> Rust, or Python. High-level and low-level are **relative terms** that convey the
> amount of abstraction involved.

---

### Memory Layout

When a program is loaded into memory, it creates a **process** with three key regions:

| Region     | Purpose                                                        |
| ---------- | -------------------------------------------------------------- |
| **Static** | Global variables and constants                                 |
| **Stack**  | Function frames and local variables — auto-managed, LIFO       |
| **Heap**   | Dynamically allocated data shared across functions and threads |

- **Stack analogy:** A notepad — write, tear off when done.
- **Heap analogy:** A whiteboard — write, stays until you explicitly erase it.

Objects live on the heap because they need to _outlive_ the function that created them.

**How an OS knows to run a program at all:** each OS defines its own **executable
format** — a file layout the OS knows how to parse and load. On Windows this is the **PE
(Portable Executable)** format; on Linux it's **ELF**. Both split the file into named
sections, most notably a `.text` section (the compiled opcodes — the actual program
code) and a `.data` section (initial values for global variables). This is why an
executable built for one OS won't run on another: the OS's loader is looking for a
specific format it doesn't recognize.

---

## 🪜 II. The Language Hierarchy & Translation

Informally, a **language** is structured text with syntax and semantics. Source code
written in a programming language needs:

1. A **translator** of some sort — converts it to another language/format.
2. An **executor** of some sort — runs the translated commands to produce output.

### Levels of Abstraction

```
Machine Language  →  Assembly  →  IR  →  Bytecode  →  Source Language
     (lowest)                                              (highest)
```

| Level                                | Description                                                                         |
| ------------------------------------ | ----------------------------------------------------------------------------------- |
| **Machine Language**                 | Raw bit-pattern opcodes directly executed by the CPU                                |
| **Assembly Language**                | Human-readable symbols mapped to opcodes; translated by an Assembler                |
| **IR (Intermediate Representation)** | Any format between source and assembly; converting between IRs is called _lowering_ |
| **Bytecode**                         | An IR emulating a simplified instruction set; executed by a Virtual Machine (VM)    |

---

### Classifying Languages

Languages can be described along two independent axes:

| Axis                    | Options                           | Meaning                                                                                                     |
| ----------------------- | --------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **What it executes on** | **Compiled** vs. **Interpreted**  | Compiled code runs directly on the CPU; interpreted code is translated on the fly by another program        |
| **How it executes**     | **Imperative** vs. **Functional** | Imperative runs instructions top-to-bottom; functional resolves through function declaration and evaluation |

These combine independently — e.g. C is compiled + imperative, Java is interpreted (its
bytecode runs on the JVM) + imperative, Haskell is compiled + functional. An interpreted
language can still be _distributed_ as something that looks compiled, by bundling the
interpreter together with the source/bytecode it runs.

> There's no "best" combination — different points on these axes trade off differently
> for different problems, and (per the Crash Course discussion) _every_ language
> ultimately answers to the same question: how does this text become something the CPU
> executes?

### Compilers

A **compiler** is any program that translates (maps, encodes) Language A → Language B.
Its two core components:

- **Frontend** — Maps source code strings to a structured format called an **Abstract
  Syntax Tree (AST)**.
- **Backend (Code Generator)** — Translates the AST to Bytecode, IR, or Assembly.

Most often, when we talk about compiler, we mean Ahead-Of-Time (AOT) compiler where the
translation happens before execution. Another form of translation is Just-In-Time (JIT)
compilation where translation happens right at the time of the execution.

| Translation Type        | When It Happens  | Examples               |
| ----------------------- | ---------------- | ---------------------- |
| **AOT (Ahead-of-Time)** | Before execution | `rustc`, `gcc`         |
| **JIT (Just-in-Time)**  | During execution | V8 (JS), PyPy (Python) |
| **Transpiler**          | Source-to-source | Python → Java          |

### Virtual Machines (VMs)

Hardware instructions are vendor-specific — Intel and AMD instructions differ. A
**Virtual Machine (VM)** abstracts away hardware details so that code compiled to the
VM's language becomes **platform-agnostic**.

The most famous example: the **Java Virtual Machine (JVM)**. Any valid Java Bytecode
runs on any platform with a **Java Runtime Environment (JRE)**, regardless of where it
was compiled.

---

## 🧭 A Computer Science Concept Map

The archived
[Carl Cheo computer-science overview](https://web.archive.org/web/20240228005454/https://carlcheo.com/compsci)
groups beginner concepts into algorithms/data structures, artificial intelligence,
architecture, concurrency, security, and development methodology. The analogies are
useful entry points; the tables below tighten them into contracts that transfer to
language implementation and reverse engineering.

### 1. The Major Areas Reinforce Each Other

| Area                           | Central question                                     | Language/reversing connection                       |
| ------------------------------ | ---------------------------------------------------- | --------------------------------------------------- |
| Algorithms and data structures | How is information represented and transformed?      | ASTs, symbol tables, graphs, worklists              |
| Artificial intelligence        | How can a system search, learn, or choose?           | optimizers, heuristics, autotuning                  |
| Computer architecture          | What physically executes an operation?               | instruction sets, caches, ABI, memory layout        |
| Concurrency                    | What can overlap, and how is shared state protected? | parallel passes, runtimes, event loops              |
| Security                       | What does an input or caller have permission to do?  | parser limits, sandboxing, binary analysis          |
| Development methodology        | How is uncertain work divided and validated?         | compiler stages, tests, reproducible investigations |

> 🧠 **Learning rule:** an analogy starts a model; an invariant finishes it. Ask what
> the analogy hides—costs, edge cases, ownership, failure, or adversarial behavior.

### 2. Complexity Describes Growth

Big-O notation describes an **asymptotic upper bound**. It is often used for worst-case
time, but Big-O itself does not mean “worst case”; best, average, and amortized behavior
must be named separately.

| Growth       | Example               | What happens as input doubles |
| ------------ | --------------------- | ----------------------------- |
| `O(1)`       | indexed array access  | roughly unchanged             |
| `O(log n)`   | binary search         | one extra step                |
| `O(n)`       | scan every token      | roughly doubles               |
| `O(n log n)` | comparison sort       | a little more than doubles    |
| `O(n²)`      | compare every pair    | roughly quadruples            |
| `O(2ⁿ)`      | enumerate all subsets | roughly squares the work      |

For a compiler, complexity is only part of the cost:

```text
total cost ≈ algorithmic growth
           × constant work per item
           × allocation/cache behavior
           × number of repeated passes
```

### 3. Data Structures Encode Operations

| Structure              | Strength                         | Language-tool example                   |
| ---------------------- | -------------------------------- | --------------------------------------- |
| contiguous array/`Vec` | iteration and indexed access     | tokens, bytecode, constant pools        |
| linked structure       | stable links, local insertion    | intrusive runtime lists in rare cases   |
| stack                  | last-in-first-out                | parser states, calls, expression eval   |
| queue                  | first-in-first-out               | BFS, event processing                   |
| hash table             | expected fast key lookup         | symbols, interners, caches              |
| tree                   | hierarchy or ordered partition   | syntax, scopes, search indexes          |
| graph                  | arbitrary relationships          | control flow, call graph, dependencies  |
| heap/priority queue    | repeatedly select best candidate | schedulers, Dijkstra, best-first search |

Choose a structure from the operations and invariants, not from its familiarity.

### 4. Search Strategies Trade Certainty for Speed

| Strategy            | Decision rule                                  | Typical weakness                      |
| ------------------- | ---------------------------------------------- | ------------------------------------- |
| greedy              | take the best immediate choice                 | local choice may block global optimum |
| hill climbing       | move to a better neighboring state             | local maxima and plateaus             |
| random restart      | run local search from several initial states   | more work; still no general guarantee |
| simulated annealing | sometimes accept worse moves, less often later | schedule-sensitive and approximate    |
| dynamic programming | reuse overlapping subproblem results           | state space may still be too large    |
| exhaustive search   | inspect every candidate                        | combinatorial explosion               |

Compiler register allocation, instruction selection, inlining, and layout can all use
heuristics because globally optimal solutions may be too expensive. Record when a pass
is exact, approximate, or nondeterministic.

### 5. Computability Is Not Complexity

| Question        | Meaning                                                    |
| --------------- | ---------------------------------------------------------- |
| **Computable?** | Can any algorithm solve the problem for every valid input? |
| **Decidable?**  | Can an algorithm always halt with yes/no?                  |
| **Tractable?**  | Can it be solved with acceptable resources?                |
| **Verifiable?** | Can a proposed answer be checked efficiently?              |

The **halting problem** shows that no general algorithm can decide whether every
arbitrary program will halt. A practical analyzer can still prove termination for a
restricted language or return **unknown** when its proof is insufficient.

```rust
enum Analysis<T> {
    Proven(T),
    Disproven { reason: String },
    Unknown { limit_reached: bool },
}
```

This three-way result is more honest than treating “not proven” as “false.”

### 6. Concurrency Is Not Parallelism

| Concept        | Precise idea                                    |
| -------------- | ----------------------------------------------- |
| concurrency    | tasks have overlapping lifetimes/progress       |
| parallelism    | work executes at the same physical instant      |
| race condition | behavior depends on an uncontrolled ordering    |
| mutex          | one holder at a time enters a critical section  |
| semaphore      | a bounded number of permits guard a resource    |
| deadlock       | tasks wait in a cycle that cannot make progress |

A compiler can be concurrent without being parallel—for example, an event loop can
interleave file reads and diagnostics on one thread. Parallel compilation needs actual
simultaneous workers and data dependencies that permit them to run safely.

### 7. Security Is Boundary Engineering

| Principle          | Concrete question                                        |
| ------------------ | -------------------------------------------------------- |
| least privilege    | What is the smallest capability this component needs?    |
| defense in depth   | Which independent checks contain one failure?            |
| complete mediation | Is every sensitive operation authorized?                 |
| secure defaults    | Is the safe behavior selected without extra setup?       |
| fail closed        | Does an error deny access rather than silently allow it? |
| auditability       | Can important decisions be reconstructed later?          |

Cryptography provides tools for confidentiality, integrity, authentication, and
signatures; it does not automatically make the surrounding parser, protocol, key
storage, randomness, or authorization correct. Use established cryptographic libraries
and protocols rather than inventing production cryptography.

### 8. Methodology Is a Feedback Loop

| Style              | Useful when                                   | Main risk                                 |
| ------------------ | --------------------------------------------- | ----------------------------------------- |
| sequential plan    | requirements and interfaces are stable        | late discovery invalidates earlier work   |
| iterative delivery | uncertainty is high and feedback is available | unmanaged scope and architectural drift   |
| prototype          | testing a risky idea or interface             | prototype accidentally becomes production |
| experiment         | comparing measurable hypotheses               | weak controls produce misleading results  |

The compiler pipeline benefits from small, testable increments:

```text
define one contract
→ implement one stage
→ inspect its artifact
→ test boundaries and failures
→ connect it to the next stage
```

> ➡️ **Next:** the parser section applies this map to the first concrete language stage:
> source text becomes a bounded, structured AST.

---

## 🧱 Building a Parser

A parser takes raw source code (a string) and converts it into a structured **Abstract
Syntax Tree (AST)** that a computer can evaluate or compile. Think of source code as a
sentence and the AST as its grammatical diagram. It captures _structure_, not just text.

> 🧭 **Parser invariant:** every accepted input produces a well-formed structure, and
> every rejected input produces a bounded, useful diagnostic rather than a crash.

Here is the step-by-step process of building one from scratch using Rust and the `pest`
crate:

### 1. Setting Up the Lexer/Grammar

**Grammar rules are built from expressions** (hence "parsing expression grammar"). These
expressions are a terse, formal description of how to parse an input string.

**Expressions are composable**: they can be built out of other expressions and nested
inside of each other to produce arbitrarily complex rules (although you should break
very complicated expressions into multiple rules to make them easier to manage).

**PEG expressions** are suitable for both _high-level meaning_, like "a function
signature, followed by a function body", and _low-level meaning_, like "a semicolon,
followed by a line feed". The combining form "followed by", the sequence operator, is
the same in either case.

---

#### Macro Captures Used in Grammar Code

| Within Macros  | Explanation                                                                 |
| -------------- | --------------------------------------------------------------------------- |
| `$x:ty`        | Macro capture (here a `$x` is the capture and `ty` means `x` must be type). |
| `$x:block`     | A block `{}` of statements or expressions, e.g., `{ let x = 5; }`           |
| `$x:expr`      | An expression, e.g., `x`, `1 + 1`, `String::new()` or `vec![]`              |
| `$x:ident`     | An identifier, for example in `let x = 0;` the identifier is `x`.           |
| `$x:item`      | An item, like a function, struct, module, etc.                              |
| `$x:lifetime`  | A lifetime (e.g., `'a`, `'static`, etc.).                                   |
| `$x:literal`   | A literal (e.g., `3`, `"foo"`, `b"bar"`, etc.).                             |
| `$x:meta`      | A meta item; the things that go inside `#[…]` and `#![…]` attributes.       |
| `$x:pat`       | A pattern, such as `Some(x)`, `(17, 'a')`, or an OR-pattern.                |
| `$x:pat_param` | A pattern accepted as a parameter without a top-level OR-pattern.           |
| `$x:path`      | A path (e.g., `foo`, `::std::mem::replace`, `transmute::<_, int>`).         |
| `$x:stmt`      | A statement, e.g., `let x = 1 + 1;`, `String::new();` or `vec![];`          |
| `$x:tt`        | A single token tree.                                                        |
| `$x:ty`        | A type, e.g., `String`, `usize` or `Vec<u8>`.                               |
| `$x:vis`       | A visibility modifier; `pub`, `pub(crate)`, etc.                            |

---

#### Pest

First, you need rules that define what makes valid code. Using a parser generator (like
the `pest` crate in Rust), you define a grammar file (e.g., `grammar.pest`). The tool
uses this grammar to validate the text and tokenize the raw string into an iterator of
parsed "pairs" (tokens) — e.g., pulling out operators, numbers, and nested expressions.

```rust
#[derive(pest_derive::Parser)]
#[grammar = "grammar.pest"]
struct CalcParser;
```

| Syntax   | Meaning                    | Example                                                |
| -------- | -------------------------- | ------------------------------------------------------ |
| `"text"` | Match exact text           | `"def"` matches the keyword def                        |
| `~`      | Sequence (then)            | `"if" ~ "(" ~ Expr ~ ")"` matches `if` followed by `(` |
| `\|`     | Choice (or)                | `"true" \| "false"` matches either                     |
| `*`      | Zero or more (Kleene star) | `Stmt*` matches any number of statements               |
| `+`      | One or more                | `ASCII_DIGIT+` matches one or more digits              |
| `?`      | Optional                   | `ReturnType?` matches zero or one return type          |
| `{ }`    | Rule definition            | `Add = { "+" }` defines a rule                         |
| `_{ }`   | Silent rule                | `_{ Expr }` matches but does not appear in AST         |
| `@{ }`   | Atomic rule                | `@{ ASCII_DIGIT+ }` matches as a single token          |
| `SOI`    | Start of input             | Beginning of the source code                           |
| `EOI`    | End of input               | End of the source code                                 |

##### Example: Calculator Grammar

```pest
num = @{ int ~ ("." ~ ASCII_DIGIT*)? ~ (^"e" ~ int)? }
int = { ("+" | "-")? ~ ASCII_DIGIT+ }

operation = _{ add | subtract | multiply | divide | power }
    add      = { "+" }
    subtract = { "-" }
    multiply = { "*" }
    divide   = { "/" }
    power    = { "^" }

expr = { term ~ (operation ~ term)* }
term = _{ num | "(" ~ expr ~ ")" }

calculation = _{ SOI ~ expr ~ EOI }

WHITESPACE = _{ " " | "\t" }
```

#### Cheat sheet

| Syntax           | Meaning                         | Syntax              | Meaning            |
| :--------------: | :-----------------------------: | :-----------------: | :----------------: |
| `foo = { ... }`  | regular rule                    | `baz = @{ ... }`    | atomic             |
| `bar = _{ ... }` | silent                          | `qux = ${ ... }`    | compound-atomic    |
| `#tag = ...`     | tags                            | `plugh = !{ ... }`  | non-atomic         |
| `"abc"`          | exact string                    | `^"abc"`            | case insensitive   |
| `'a'..'z'`       | character range                 | `ANY`               | any character      |
| `foo ~ bar`      | sequence                        | `baz \| qux`        | ordered choice     |
| `foo*`           | zero or more                    | `bar+`              | one or more        |
| `baz?`           | optional                        | `qux{n}`            | exactly _n_        |
| `qux{m, n}`      | between _m_ and _n_ (inclusive) |                     |                    |
| `&foo`           | positive predicate              | `!bar`              | negative predicate |
| `PUSH(baz)`      | match and push                  | `PUSH_LITERAL("a")` | push without match |
| `POP`            | match and pop                   | `PEEK`              | match without pop  |
| `DROP`           | pop without matching            | `PEEK_ALL`          | match entire stack |

### 2. Defining the Abstract Syntax Tree (AST)

The AST captures the _meaning_ and _structure_ of the code, not just the raw text. In
Rust, this is typically done using `enum` to represent different node types:

- **Terminal Nodes (Leaves):** e.g., `Node::Int(i32)` for raw numbers.
- **Unary Expressions:** e.g., `Node::UnaryExpr { op, child }` for operations like `-5`.
- **Binary Expressions:** e.g., `Node::BinaryExpr { op, lhs, rhs }` for operations like
  `1 + 2`. _(Note: `Box<Node>` is used for the children to allow recursive nesting)._

The tree naturally encodes the order of operations through its nesting.

```pest
Program = _{ SOI ~ Expr ~ EOF }

Expr = { BinaryExpr | UnaryExpr | Term }

Term = { Int | "(" ~ Expr ~ ")" }

UnaryExpr = { Operator ~ Term }

// Allow expressions to start with a unary expression (e.g., -1 + 2)
BinaryExpr = { (UnaryExpr | Term) ~ (Operator ~ Term)+ }

Operator = { "+" | "-" }

Int = @{ ASCII_DIGIT+ }

WHITESPACE = _{ " " | "\t" | "\r" | "\n" }

EOF = _{ EOI | ";" }
```

```rust
pub enum Operator {
    Plus,
    Minus,
}

pub enum Node {
    // Terminal Nodes (Leaves)
    Int(i32),
    // Unary Expressions (e.g., -5)
    UnaryExpr {
        op: Operator,
        child: Box<Node>,
    },
    // Binary Expressions (e.g., 1 + 2)
    BinaryExpr {
        op: Operator,
        lhs: Box<Node>,
        rhs: Box<Node>,
    },
}
```

> **Note:** `Box<Node>` is required for the children to allow recursive nesting. Without
> `Box`, Rust wouldn't be able to calculate the size of the `Node` enum at compile time.

### 3. The Main Parsing Loop

The main entry point (often just `parse(source: &str)`) takes the raw string, feeds it
to the parser generator, and loops through the resulting top-level tokens. When it finds
a top-level expression, it passes that token to a recursive builder function to
construct the AST.

```rust
pub fn parse(source: &str) -> std::result::Result<Vec<Node>, pest::error::Error<Rule>> {
    let mut ast = vec![];
    let pairs = CalcParser::parse(Rule::Program, source)?;

    for pair in pairs {
        if let Rule::Expr = pair.as_rule() {
            ast.push(build_ast_from_expr(pair));
        }
    }
    Ok(ast)
}
```

### 4. Parsing Expressions & Associativity (the composits)

This is the core logic that converts generic `pest` tokens into your custom AST nodes by
looking at the rule:

- **Unary Expressions:** Grab the operator and the single child term, then construct the
  Unary node.
- **Binary Expressions:** Handling chained operations (like `1 + 2 + 3`) requires
  **left-associativity** so it evaluates as `(1 + 2) + 3`. Instead of using pure
  recursion (which might result in right-associativity), this is often handled using a
  `loop`:
  1. Parse the initial Left-Hand Side (LHS), operator, and Right-Hand Side (RHS).
  2. Build the first `BinaryExpr` node.
  3. **The Loop:** If there is another operator immediately following in the token
     stream, take the expression you _just built_ and make it the **new LHS**, grab the
     next number as the RHS, and build a new, nested `BinaryExpr`.

```rust
fn parse_unary_expr(pair: pest::iterators::Pair<Rule>, child: Node) -> Node {
    Node::UnaryExpr {
        op: match pair.as_str() {
            "+" => Operator::Plus,
            "-" => Operator::Minus,
            _ => unreachable!(),
        },
        child: Box::new(child),
    }
}

fn parse_binary_expr(pair: pest::iterators::Pair<Rule>, lhs: Node, rhs: Node) -> Node {
    Node::BinaryExpr {
        op: match pair.as_str() {
            "+" => Operator::Plus,
            "-" => Operator::Minus,
            _ => unreachable!(),
        },
        lhs: Box::new(lhs),
        rhs: Box::new(rhs),
    }
}
```

```rust
fn build_ast_from_expr(pair: pest::iterators::Pair<Rule> ) -> Node {
match pair.as_rule() {
     Rule::Expr => build_ast_from_expr(pair.into_inner().next().unwrap()),
     Rule::UnaryExpr => {
       let mut pair = pair.into_inner();
       let op = pair.next().unwrap();
       let child = pair.next().unwrap();
       let child = build_ast_from_term(child);
       parse_unary_expr(op, child)
   }
     Rule::BinaryExpr => {
         let mut pair = pair.into_inner();

         let lhspair = pair.next().unwrap();
         let mut lhs_built_term = match lhspair.as_rule() {
                     Rule::UnaryExpr => {
                         let mut inner = lhspair.into_inner();
                         let op = inner.next().unwrap();
                         let child = inner.next().unwrap();
                         let child = build_ast_from_term(child);
                         parse_unary_expr(op, child)
                     }
                     _ => build_ast_from_term(lhspair),
                 };

         let op = pair.next().unwrap();
         let rhspair = pair.next().unwrap();
         let mut rhs_built_term = build_ast_from_term(rhspair);

         let mut binary_expr_retval = parse_binary_expr(op, lhs_built_term, rhs_built_term);

         loop {
             let pair_buf = pair.next();
             if let Some(op) = pair_buf {
                 lhs_binary_expr = binary_expr_retval;
                 adjacent_rhs_binary_expr = build_ast_from_term(pair.next().unwrap());
                 retval = parse_binary_expr(op, lhs_binary_expr, adjacent_rhs_binary_expr);
             } else {
                 return retval;
             }
         }
     }
     Rule::Term => build_ast_from_term(pair),
     unknown => panic!("Unknown expr: {:?}", unknown),
  }
}
```

### 5. Parsing Terms (The Leaves)

When the parser drills down to a basic term (`Rule::Term`), it hits the bottom of the
tree. It expects one of two things:

1. **A raw number:** It parses the string into an `i32` and returns a `Node::Int`.
2. **A nested expression:** If it hits parentheses, it recurses back up to the
   expression parsing logic to evaluate the inside of the parentheses first.

```rust
fn build_ast_from_term(pair: pest::iterators::Pair<Rule>) -> Node {
    let pair = pair.into_inner().next().unwrap();
    match pair.as_rule() {
        Rule::Int => {
            let int: i32 = pair.as_str().parse().unwrap();
            Node::Int(int)
        }
        Rule::Expr => build_ast_from_expr(pair),
        unknown => panic!("Unknown term: {:?}", unknown),
    }
}
```

---

## 🦗 Pest PEG Parsing in Practice

[Pest](https://pest.rs/book/) turns a **parsing expression grammar** (PEG) into a Rust
parser. It is a good fit for a small language because the grammar remains readable while
Rust code handles AST construction, validation, diagnostics, and later compiler phases.

> 🧠 **Mental model:** a Pest grammar recognizes concrete syntax. Your Rust conversion
> code decides which recognized details become meaningful AST nodes.

### 1. PEG Choice Is Ordered

In a PEG, `a | b` means **try `a` first; try `b` only if `a` fails**. This differs from
thinking of alternatives as an unordered mathematical set.

```pest
keyword_if = { "if" }
identifier = @{ ASCII_ALPHA ~ (ASCII_ALPHANUMERIC | "_")* }

// Put the more specific alternative first.
atom = { keyword_if | identifier }
```

| Operator | Meaning            | Compiler use                         |
| -------- | ------------------ | ------------------------------------ |
| `~`      | Sequence           | `let` then name then `=` then value  |
| \|       | Ordered choice     | statement form or expression form    |
| `*`      | Zero or more       | arguments, statements, suffixes      |
| `+`      | One or more        | digits, identifier characters        |
| `?`      | Optional           | type annotation or trailing comma    |
| `&rule`  | Positive lookahead | Require a prefix without consuming   |
| `!rule`  | Negative lookahead | Reject reserved words or terminators |

Repetition binds more tightly than predicates, then sequence, then ordered choice. Add
parentheses when a reader might otherwise have to remember that precedence.

### 2. Whitespace Is a Grammar Policy

When rules named `WHITESPACE` or `COMMENT` exist, Pest can insert them implicitly
between sequence and repetition expressions. That is convenient for language-level
tokens, but dangerous inside indivisible tokens such as identifiers and numbers.

```pest
WHITESPACE = _{ " " | "\t" | NEWLINE }
COMMENT = _{ "//" ~ (!NEWLINE ~ ANY)* }

identifier = @{ (ASCII_ALPHA | "_") ~ (ASCII_ALPHANUMERIC | "_")* }
integer = @{ "-"? ~ ASCII_DIGIT+ }
```

| Modifier | Parse-tree effect                      | Whitespace effect             | Typical use             |
| -------- | -------------------------------------- | ----------------------------- | ----------------------- |
| `_`      | Silent: omit this rule                 | Normal                        | punctuation, whitespace |
| `@`      | Atomic: inner rules become silent      | Disable implicit whitespace   | identifiers, numbers    |
| `$`      | Compound atomic: keep inner rule pairs | Disable implicit whitespace   | structured string token |
| `!`      | Non-atomic inside an atomic context    | Re-enable implicit whitespace | interpolation           |

**Atomic does not mean thread-safe.** Here it means “match this grammar region as one
whitespace-sensitive unit.”

### 3. Anchor Complete Inputs

Without `SOI` and `EOI`, a parser may recognize only a valid prefix and leave unexpected
text behind.

```pest
program = { SOI ~ statement* ~ EOI }

statement = _{ let_stmt | print_stmt }
let_stmt = { "let" ~ identifier ~ "=" ~ expression ~ ";" }
print_stmt = { "print" ~ expression ~ ";" }

expression = { term ~ (add_op ~ term)* }
add_op = { "+" | "-" }
term = _{ integer | identifier | "(" ~ expression ~ ")" }
```

| Entry rule      | Appropriate contract                                  |
| --------------- | ----------------------------------------------------- |
| `program`       | Must consume the whole source file                    |
| `expression`    | May parse a nested region selected by another rule    |
| editor fragment | May intentionally accept incomplete, recoverable text |

### 4. `Pair` Is a Concrete-Syntax View, Not the AST

Each `Pair<Rule>` identifies the matched rule, matched text, byte span, and nested pairs.
Convert it promptly into your own AST so the rest of the compiler is independent of
Pest's tree shape.

```rust
use pest::iterators::Pair;
use std::ops::Range;

#[derive(Debug)]
struct Spanned<T> {
    node: T,
    bytes: Range<usize>,
}

#[derive(Debug)]
enum Expr {
    Integer(i64),
    Name(String),
    Add(Box<Spanned<Expr>>, Box<Spanned<Expr>>),
}

fn byte_range(pair: &Pair<'_, Rule>) -> Range<usize> {
    let span = pair.as_span();
    span.start()..span.end()
}
```

Pest spans are **byte offsets** into the original UTF-8 input. If diagnostics also need
line and column, compute or cache that view deliberately; never treat a byte offset as
a character index.

### 5. Keep Syntax and Semantic Errors Separate

| Failure layer | Example                           | Best owner           |
| ------------- | --------------------------------- | -------------------- |
| Grammar       | Missing `)`                       | Pest parse error     |
| AST lowering  | Integer does not fit in `i64`     | AST conversion error |
| Name binding  | Unknown variable                  | Resolver             |
| Type checking | Add integer to function           | Type checker         |
| Codegen       | Unsupported target representation | Backend              |

```rust
#[derive(Debug)]
enum FrontendError {
    Syntax(Box<pest::error::Error<Rule>>),
    IntegerOutOfRange { text: String, bytes: Range<usize> },
}

fn parse_integer(pair: Pair<'_, Rule>) -> Result<Spanned<Expr>, FrontendError> {
    let bytes = byte_range(&pair);
    let text = pair.as_str();
    let value = text.parse::<i64>().map_err(|_| {
        FrontendError::IntegerOutOfRange {
            text: text.to_owned(),
            bytes: bytes.clone(),
        }
    })?;

    Ok(Spanned {
        node: Expr::Integer(value),
        bytes,
    })
}
```

### 6. Use Pratt Parsing for Expression Precedence

Pest's `PrattParser` maps a flat operator/operand stream into the intended precedence
and associativity. Define operators from **lowest to highest precedence**.

```rust
use pest::pratt_parser::{Assoc, Op, PrattParser};
use std::sync::LazyLock;

static PRATT: LazyLock<PrattParser<Rule>> = LazyLock::new(|| {
    PrattParser::new()
        .op(Op::infix(Rule::add, Assoc::Left)
            | Op::infix(Rule::subtract, Assoc::Left))
        .op(Op::infix(Rule::multiply, Assoc::Left)
            | Op::infix(Rule::divide, Assoc::Left))
        .op(Op::infix(Rule::power, Assoc::Right))
        .op(Op::prefix(Rule::negate))
});
```

| Expression   | Required shape   | Reason                              |
| ------------ | ---------------- | ----------------------------------- |
| `2 + 3 * 4`  | `2 + (3 * 4)`    | Multiplication is higher            |
| `10 - 3 - 2` | `(10 - 3) - 2`   | Subtraction is left-associative     |
| `2 ^ 3 ^ 2`  | `2 ^ (3 ^ 2)`    | Power is commonly right-associative |
| `-x ^ 2`     | Language-defined | Prefix precedence must be explicit  |

> 🧭 **Invariant:** after AST construction, precedence is encoded by tree shape. Later
> compiler phases should not need to remember the grammar's precedence table.

### 7. Test the Grammar at Three Scales

| Scale       | Test input                              | Assertion                            |
| ----------- | --------------------------------------- | ------------------------------------ |
| Rule        | one identifier, literal, or operator    | exact match and rejection cases      |
| Construct   | one expression or statement             | AST shape, span, associativity       |
| Program     | a small file                            | complete parse and diagnostics       |
| Adversarial | truncation, deep nesting, huge literals | bounded behavior and clear rejection |

```rust
#[test]
fn multiplication_binds_more_tightly_than_addition() {
    let ast = parse_expression("2 + 3 * 4").expect("valid test expression");
    assert_eq!(format!("{ast:?}"), "Add(Integer(2), Multiply(Integer(3), Integer(4)))");
}

#[test]
fn program_rejects_trailing_garbage() {
    assert!(LanguageParser::parse(Rule::program, "print 1; ???").is_err());
}
```

Snapshotting a debug string is useful early, but mature compilers should compare typed
AST values so formatting changes do not break semantic tests.

> ➡️ **Next:** the execution-pipeline section treats the AST as the trusted input to an
> interpreter or backend. Pest-specific details stop here; the language model continues.

---

## 🌳 III. The Execution Pipeline & AST

### The Pipeline

```
Source → Tokens → AST → Output
```

Walking through `1 + 2`:

1. **Grammar** — Rules defining valid syntax. `1 + 2` matches; `+ + 1` doesn't.
2. **Lexer (Tokenizer)** — Breaks source into meaningful chunks: `"1 + 2"` →
   `[1, +, 2]`. Tracks source location for error messages.
3. **Parser** — Builds a tree structure from tokens: `+` at the root, `1` and `2` as
   children — this is the **AST**.
4. **Interpreter/Evaluator** — Walks the AST recursively and computes the result.

> **AST analogy:** Think of source code as a sentence and the AST as its grammatical
> diagram. Just as "The cat sat" is diagrammed into subject/verb, `1 + 2` is diagrammed
> into left/operator/right. The AST captures **structure**, not just text.

We need a systematic way to turn any AST into LLVM IR.

The answer is recursive tree traversal. We already do this in the interpreter - walk the
tree, evaluate each node. For code generation, we walk the tree and emit instructions
for each node instead of computing values.

Two common patterns help structure this:

- Builder pattern - Used for LLVM IR generation
- Visitor pattern - for AST transformations

### Builder Pattern

Think of the LLVM builder like a cursor in a text editor. You position it somewhere in
your code, then “type” instructions at that position. The builder keeps track of where
you are and ensures instructions are added in the right place.

Let’s compare our interpreter’s recursive evaluation to the new JIT approach: the
interpreter walks the tree and computes values directly, while the builder walks the
tree and emits instructions instead.

---

### The Interpreter & Recursion

The CPU is the _ultimate_ interpreter — it executes opcodes one at a time. Our software
interpreter does the same at a higher level by **walking the AST recursively**:

To evaluate `1 + 2`:

1. Evaluate left (`1`) → `1`
2. Evaluate right (`2`) → `2`
3. Apply operator (`+`) → `3`

If the left side were `(3 + 4)`, we'd recursively evaluate it first. This is why trees
are powerful: **structure determines evaluation order**. Parse → AST → recursive eval is
the foundation of every interpreter — Python, Ruby, JavaScript all do this with more
node types.

---

### State & Functions

A calculator is **stateless** — input goes in, output comes out, nothing persists. A
real language is a **state machine**:

| Language feature | Example             | Role in execution                  |
| ---------------- | ------------------- | ---------------------------------- |
| **Variables**    | `n`                 | Named storage                      |
| **Functions**    | `fib`               | Abstraction and reuse              |
| **Parameters**   | passing `n`         | Data flow into a call              |
| **Conditionals** | `if` / `else`       | Control-flow branching             |
| **Operators**    | `<`, `-`            | Computation                        |
| **Recursion**    | `fib` calling `fib` | Self-reference                     |
| **Call stack**   | one frame per call  | Tracks local execution and returns |

The AST becomes a _program to execute_, not just an expression to evaluate.

---

### Variables

Variables let us store values and refer to them by name. Without variables, every
computation would need to repeat its literal inputs — variables give us **memory**.

Reassigning a variable means any code referencing it automatically uses the new value,
which makes code reusable and readable.

**Where variables live:** the _environment_ (storage) — a `HashMap` inside a _frame_.

- **Local** variables exist inside a function.
- **Global** variables exist outside all functions.
- Variables can be **reassigned**.

This simple mechanism — storing and looking up names — underlies all programming:

| Concept       | Role                                         |
| ------------- | -------------------------------------------- |
| Parameters    | Variables bound from function call arguments |
| Loop counters | Variables that change each iteration         |
| Object fields | Variables attached to an object              |

> The environment is one of the most important data structures in any interpreter.

---

### Functions

Functions are the heart of any programming language. Without them, code must be
copy-pasted every time it's reused. Functions let us:

1. **Name** a computation, to refer to it later
2. **Parameterize** it with inputs, so it works with different values
3. **Reuse** it — write once, call anywhere

**Parameters**: names for the inputs a function expects. When we bind _parameters_, we
are associating names with argument values.

Functions enable **abstraction**: calling `fibonacci(10)` hides _how_ Fibonacci is
computed behind a simple name.

#### How Function Calls Work

Evaluating `add(3, 4)` triggers a fixed sequence:

1. **Look up** the function by name
2. **Evaluate** the arguments (`3`, `4`)
3. **Create** a new frame for local variables
4. **Bind** parameters to arguments (`a = 3`, `b = 4`)
5. **Execute** the function body
6. **Return** the result and pop the frame

### The Call Stack

**Call Stack**: Runtime data structure tracking function calls. Each call pushes a
frame; return pops it.

Each **frame** on the stack represents one in-progress function call. The stack
**grows** on call and **shrinks** on return — last in, first out (LIFO).

**One frame per call:** two calls to `foo` each get their own `x`. Call 1's `x = 5`
never interferes with call 2's `x = 10`, because they live in separate frames. This
isolation is essential for recursion, where the same function appears on the stack
multiple times, each with its own variables.

---

### Control Flow

Programs need to **decide** ("if logged in, show dashboard") and **repeat** ("while
items remain, sum their prices"). Without control flow, code only runs straight-line,
top to bottom.

#### Conditionals: `if` / `else`

An `if` expression evaluates a condition and picks a branch:

1. **Evaluate** the condition (e.g. `x > 0`)
2. **Choose** a branch — `then` if true, `else` if false
3. **Return** whatever that branch produces

#### Loops: `while`

A `while` loop repeats its body while a condition holds true, using Rust's `loop`
construct with a condition check each iteration:

> After the body runs, execution returns to the top and re-checks the condition — this
> is what creates repetition.

#### Control Flow in Functions

- **Nested conditionals** — for when one condition isn't enough
- **`return` in loops** — exits the function immediately, even from deep inside a loop;
  useful for "search" patterns

#### Control Flow as Branching

Both `if` and `while` change the flow of execution, letting code:

- **Skip** code (untaken `if` branch)
- **Repeat** code (`while` body)
- **Exit early** (`return` inside a loop)

In compiled languages, these become branch instructions — the CPU jumps to different
memory locations. (In Secondlang, `if` compiles to LLVM IR's `br` instruction.)

---

## 🔁 Recursion

Recursion is when a function calls itself. It seems paradoxical — how can a function
call itself mid-execution? — but the call stack makes it work cleanly.

> **Analogy:** Russian nesting dolls (matryoshka). Each doll contains a smaller version
> of itself, down to the smallest doll, which contains nothing.

### The Key Insight

Every recursive function needs two parts:

| Part               | Purpose                                                      |
| ------------------ | ------------------------------------------------------------ |
| **Base case**      | Returns directly, no further recursion — the "smallest doll" |
| **Recursive case** | Calls itself on a smaller problem, then combines the result  |

- No base case → recursion never stops → **stack overflow**
- No recursive case → not actually recursion, just a regular function

### Why Recursion Works

Three design decisions make it work:

1. **Functions are stored globally** — `factorial` can look itself up by name mid-call.
2. **Each call gets its own frame** — `factorial(3)` called from within `factorial(4)`
   has an `n` fully independent of the outer `n = 4`.
3. **Return values propagate correctly** — `factorial(1)` returns `1` → used by
   `factorial(2)` as `2 * 1 = 2` → and so on up the stack.

> Using a single global `n` instead of per-call frames would break recursion: each call
> would overwrite the shared `n`.

**Mutual recursion:** functions can call each other recursively.

**⚠️ Stack overflow risk:** every recursive call consumes stack memory; very deep
recursion exhausts it.

> Some languages implement **tail call optimization** to run certain recursive functions
> in constant stack space — not implemented here, but a notable optimization.

### When to Use Recursion

Recursion fits problems with recursive structure:

- **Trees** (each subtree is a smaller tree)
- **Mathematical sequences** (Fibonacci, factorial)
- **Divide-and-conquer algorithms** (merge sort, quicksort)
- **Parsing nested structures** (JSON, HTML, an AST)

For simple loops, iteration is usually clearer — but for inherently recursive problems,
recursion is more natural.

### Operators

An **operator** tells us what to do. Values and operands are the things we operate on.

| Operator                  | Example                                                 | Explanation                                                           | Overloadable?  |
| ------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------- | -------------- |
| `!`                       | `ident!(...)`, `ident!{...}`, `ident![...]`             | Macro expansion                                                       |                |
| `!`                       | `!expr`                                                 | Bitwise or logical complement                                         | `Not`          |
| `!=`                      | `expr != expr`                                          | Nonequality comparison                                                | `PartialEq`    |
| `%`                       | `expr % expr`                                           | Arithmetic remainder                                                  | `Rem`          |
| `%=`                      | `var %= expr`                                           | Arithmetic remainder and assignment                                   | `RemAssign`    |
| `&`                       | `&expr`, `&mut expr`                                    | Borrow                                                                |                |
| `&`                       | `&type`, `&mut type`, `&'a type`, `&'a mut type`        | Borrowed pointer type                                                 |                |
| `&`                       | `expr & expr`                                           | Bitwise AND                                                           | `BitAnd`       |
| `&=`                      | `var &= expr`                                           | Bitwise AND and assignment                                            | `BitAndAssign` |
| `&&`                      | `expr && expr`                                          | Short-circuiting logical AND                                          |                |
| `*`                       | `expr * expr`                                           | Arithmetic multiplication                                             | `Mul`          |
| `*=`                      | `var *= expr`                                           | Arithmetic multiplication and assignment                              | `MulAssign`    |
| `*`                       | `*expr`                                                 | Dereference                                                           | `Deref`        |
| `*`                       | `*const type`, `*mut type`                              | Raw pointer                                                           |                |
| `+`                       | `trait + trait`, `'a + trait`                           | Compound type constraint                                              |                |
| `+`                       | `expr + expr`                                           | Arithmetic addition                                                   | `Add`          |
| `+=`                      | `var += expr`                                           | Arithmetic addition and assignment                                    | `AddAssign`    |
| `,`                       | `expr, expr`                                            | Argument and element separator                                        |                |
| `-`                       | `- expr`                                                | Arithmetic negation                                                   | `Neg`          |
| `-`                       | `expr - expr`                                           | Arithmetic subtraction                                                | `Sub`          |
| `-=`                      | `var -= expr`                                           | Arithmetic subtraction and assignment                                 | `SubAssign`    |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Function and closure return type                                      |                |
| `.`                       | `expr.ident`                                            | Field access                                                          |                |
| `.`                       | `expr.ident(expr, ...)`                                 | Method call                                                           |                |
| `.`                       | `expr.0`, `expr.1`, and so on                           | Tuple indexing                                                        |                |
| `..`                      | `..`, `expr..`, `..expr`, `expr..expr`                  | Right-exclusive range literal                                         | `PartialOrd`   |
| `..=`                     | `..=expr`, `expr..=expr`                                | Right-inclusive range literal                                         | `PartialOrd`   |
| `..`                      | `..expr`                                                | Struct literal update syntax                                          |                |
| `..`                      | `variant(x, ..)`, `struct_type { x, .. }`               | “And the rest” pattern binding                                        |                |
| `...`                     | `expr...expr`                                           | (Deprecated, use `..=` instead) In a pattern: inclusive range pattern |                |
| `/`                       | `expr / expr`                                           | Arithmetic division                                                   | `Div`          |
| `/=`                      | `var /= expr`                                           | Arithmetic division and assignment                                    | `DivAssign`    |
| `:`                       | `pat: type`, `ident: type`                              | Constraints                                                           |                |
| `:`                       | `ident: expr`                                           | Struct field initializer                                              |                |
| `:`                       | `'a: loop {...}`                                        | Loop label                                                            |                |
| `;`                       | `expr;`                                                 | Statement and item terminator                                         |                |
| `;`                       | `[...; len]`                                            | Part of fixed-size array syntax                                       |                |
| `<<`                      | `expr << expr`                                          | Left-shift                                                            | `Shl`          |
| `<<=`                     | `var <<= expr`                                          | Left-shift and assignment                                             | `ShlAssign`    |
| `<`                       | `expr < expr`                                           | Less than comparison                                                  | `PartialOrd`   |
| `<=`                      | `expr <= expr`                                          | Less than or equal to comparison                                      | `PartialOrd`   |
| `=`                       | `var = expr`, `ident = type`                            | Assignment/equivalence                                                |                |
| `==`                      | `expr == expr`                                          | Equality comparison                                                   | `PartialEq`    |
| `=>`                      | `pat => expr`                                           | Part of match arm syntax                                              |                |
| `>`                       | `expr > expr`                                           | Greater than comparison                                               | `PartialOrd`   |
| `>=`                      | `expr >= expr`                                          | Greater than or equal to comparison                                   | `PartialOrd`   |
| `>>`                      | `expr >> expr`                                          | Right-shift                                                           | `Shr`          |
| `>>=`                     | `var >>= expr`                                          | Right-shift and assignment                                            | `ShrAssign`    |
| `@`                       | `ident @ pat`                                           | Pattern binding                                                       |                |
| `^`                       | `expr ^ expr`                                           | Bitwise exclusive OR                                                  | `BitXor`       |
| `^=`                      | `var ^= expr`                                           | Bitwise exclusive OR and assignment                                   | `BitXorAssign` |
| <code>&vert;</code>       | <code>pat &vert; pat</code>                             | Pattern alternatives                                                  |                |
| <code>&vert;</code>       | <code>expr &vert; expr</code>                           | Bitwise OR                                                            | `BitOr`        |
| <code>&vert;=</code>      | <code>var &vert;= expr</code>                           | Bitwise OR and assignment                                             | `BitOrAssign`  |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code>                     | Short-circuiting logical OR                                           |                |
| `?`                       | `expr?`                                                 | Error propagation                                                     |                |

---

#### State machine

If you pause a program in a debugger, you are looking at its current state. The call
stack tells you how the program got to this state and what local data it currently has
access to within that specific function frame. In computer science terms, adding a stack
to a finite state machine turns it into a **Pushdown Automaton**—allowing it to handle
nesting, recursion, and scoped memory.

- **Concise definition:** A mathematical model of computation representing a system that
  can be in exactly one of a finite number of conditions (states) at any given time. The
  machine changes from one state to another (a transition) in response to specific
  inputs or events.
- **Programming definition:** A system that maintains a "memory" of its current
  condition and changes that condition based on incoming events. Unlike a stateless
  system (like a basic calculator), a state machine's output depends on its accumulated
  history — which in a programming language is tracked via variables, memory heaps, and
  the call stack.

**Core concepts:**

- **State** — the current condition or snapshot of the system's memory (e.g., current
  variable values, active call stack).
- **Inputs/Events** — the actions or instructions fed into the system (e.g., executing
  the next line of code).
- **Transitions** — the rules that dictate how the system moves from one state to the
  next based on the input.

---

### Function Call Sequence

For `add(3, 4)`:

1. **Look up** — Find the function value by name.
2. **Evaluate arguments** — Compute `3` and `4`.
3. **Push frame** — Create a fresh environment (scope) on the call stack.
4. **Bind parameters** — `a = 3`, `b = 4`.
5. **Execute body** — Run function instructions.
6. **Pop frame** — Return result to the caller; the frame below becomes current again.

> **Call stack analogy:** A stack of sticky notes. Each call writes variables on a new
> note and puts it on top. Returning tears it off. This is why `inner()`'s variables
> don't overwrite `outer()`'s — they're on different notes.

Each **frame** is just a `HashMap` pushed and popped like any stack. Grammar grows, AST
nodes multiply, but execution is still recursive tree traversal — now with scoped state.

---

### The REPL

A **Read-Eval-Print Loop** provides an interactive environment:

1. **Read** — Get a line of input.
2. **Eval** — Parse and execute it.
3. **Print** — Show the result.
4. **Loop** — Return to step 1.

A working REPL with variables, functions, conditionals, operators, recursion, and a call
stack means you have a **real programming language**. The culminating proof is recursive
Fibonacci:

```
def fib(n) {
    if (n < 2) { return n } else { return fib(n - 1) + fib(n - 2) }
}
fib(10)   # → 55
```

**Multi-line input:** a REPL needs to know when a statement is _incomplete_ (e.g., mid
function body) versus ready to evaluate. One approach: track **bracket depth** — count
unmatched `{`, `(`, `[` (ignoring characters inside strings). While depth is positive,
keep reading lines and show a continuation prompt (`...`); once it returns to zero,
evaluate the accumulated input.

---

## 🧮 IV. Bytecode & Virtual Machines

**Bytecode** is another technique for translating source code toward machine code: it
emulates the instruction set with a new, simplified, human-friendly encoding — an
intermediate representation that sits between source and assembly, lower-level than
source but higher-level than assembly language. It's executed by a **VM**.

### Why Bytecode?

| Approach                       | Trade-off                                                  |
| ------------------------------ | ---------------------------------------------------------- |
| **Direct AST interpretation**  | Simple to implement, but slow                              |
| **Bytecode + VM**              | Moderate complexity, significantly faster, highly portable |
| **JIT to native machine code** | Fastest, but complex (LLVM dependency, platform-specific)  |

- **Simpler than JIT** — No LLVM dependency; works everywhere.
- **Faster than AST** — Bytecode is compact and cache-friendly.
- **Portable** — Same bytecode runs on any machine with your VM.

This is how Python (CPython), Java (JVM), and Ruby (YARV) work: compile source to
bytecode once, run the bytecode interpreter wherever you need it.

_**Is Python (or a language X) Compiled or Interpreted?**_

Being AOT compiled, JIT compiled or interpreted is implementation-dependent. For
example, the standard Python implementation is CPython which compiles a Python source
code (in CPython VM) to CPython Bytecode (contents of .pyc) and interprets the Bytecode.
However, another implementation of Python is PyPy which (more or less) compiles a Python
source code (in PyPy VM) to PyPy Bytecode and JIT compiles the PyPy Bytecode to the
Machine Code (and is usually faster than CPython interpreter).

#### Bytecode Structure

- `instructions` - A flat array of bytes. Opcodes and their arguments, all mixed
  together.
- `constants` - A table of literal values. Instead of encoding 42 in the instruction
  stream, we store it in the constants table and reference it by index.

```rust
pub struct Bytecode {
    pub instructions: Vec<u64>,// 64 bit instructions
    pub constants: Vec<Node>,
}
```

---

### The Stack Machine

We've seen two ways to execute code: interpret the AST directly, or JIT compile to
native machine code. There's a third approach that sits in between: **compile to
bytecode, then interpret that.** Our VM is a **Stack Machine** — it keeps intermediate
values on a stack.

```rust
const STACK_SIZE: usize = 2048;

pub struct VM {
    bytecode: Bytecode,
    stack: [Node; STACK_SIZE], // nodes are leafs in the abstract syntax tree
    stack_ptr: usize, // points to the next free space
}
```

```rust
pub enum OpCode {
    OpConstant(u16), // pointer to constant table
    OpPop,           // pop is needed for execution
    OpAdd,
    OpSub,
    OpPlus,
    OpMinus,
}
```

A **stack machine** has three components:

- **Bytecode** — the program to execute.
- A **stack array** supporting push/pop, used to hold intermediate values. A fixed-size
  array is used for speed.
- An **Instruction Pointer (IP)** and **Stack Pointer (SP)**, tracking which instruction
  is executing and what's next.

**Why stacks?** They handle _any_ nesting automatically. For `(1 + 2) * (3 + 4)`:

- Push 1, push 2, add → stack: `[3]`
- Push 3, push 4, add → stack: `[3, 7]`
- Multiply → stack: `[21]`

Every operation pops its inputs and pushes its output. The stack naturally tracks what's
"in progress."

Think of bytecode as a simplified assembly language designed for our VM.

- OpConstant(index) Push a constant onto the stack
- OpAdd Pop two values, push their sum
- OpSub Pop two values, push their difference
- OpPop Pop and discard the top value

Real assembly has hundreds of instructions.

**VM execution of `1 + 2` (fetch-decode-execute):**

These four carry the arithmetic (`OpSub` not used in this example):

```
OpConstant(1)  →  push 1                  [stack: 1   ]
OpConstant(2)  →  push 2                  [stack: 1, 2]
OpAdd          →  pop 2, pop 1, push 3    [stack: 3   ]
OpPop          →  return 3
```

The VM reads instructions left-to-right with the **Instruction Pointer** (`ip`):

1. **Fetch** — Read next byte from `bytecode.instructions[ip]`
2. **Decode** — Match on the opcode
3. **Execute** — Manipulate the stack
4. **Repeat** — Increment IP; continue until out of instructions

---

## 🏠 Hosted Language Runtime Design in Rust

This section draws from
[Writing Interpreters in Rust: a Guide](https://rust-hosted-langs.github.io/book/introduction.html).
Its central problem is deeper than parsing: how can a Rust host safely implement a
dynamic guest language whose objects, aliases, and garbage-collection rules are not
known to Rust's borrow checker?

> 🏠 **Host/guest boundary:** Rust owns the runtime implementation; the guest language
> defines the behavior of values that live inside it.

### 1. Host Language Versus Guest Language

- The **host language** implements the runtime. Here, the host is Rust.
- The **guest language** is parsed, compiled, and executed by that runtime.
- The **mutator** is ordinary guest-program execution that allocates and changes
  objects.
- The **collector** traces, moves, marks, or reclaims guest objects.

Rust statically enforces Rust ownership. It cannot automatically prove the guest
language's runtime ownership model.

```text
Rust compiler proves host-level invariants
        ↓
runtime abstraction validates guest references
        ↓
guest program manipulates dynamic objects
```

This means runtime authors must design a safe public abstraction around a small,
carefully reviewed unsafe core.

### 2. Separate Runtime Layers

A useful architecture has explicit boundaries:

```text
source text
    ↓ parser
guest syntax / AST
    ↓ compiler
bytecode + literal pools
    ↓ virtual machine
dynamic values + call frames
    ↓ allocation interface
heap objects + object headers
    ↓ collector
mark / trace / reclaim
```

Each layer should depend on the narrowest interface below it:

| Layer         | Should understand              | Should not depend on                |
| ------------- | ------------------------------ | ----------------------------------- |
| **Parser**    | tokens, grammar, AST nodes     | heap-block layout                   |
| **Compiler**  | AST, constants, bytecode       | raw-pointer manipulation            |
| **VM**        | instructions, values, handles  | parser internals                    |
| **Allocator** | sizes, alignments, object kind | source-language grammar             |
| **Collector** | roots and tracing metadata     | syntax or source-level declarations |

### 3. Allocation Is More Than Getting Bytes

A runtime allocation request must answer:

| Allocation property | Question the runtime must answer                        |
| ------------------- | ------------------------------------------------------- |
| **Size**            | How many bytes are required?                            |
| **Alignment**       | At which address boundary may the object begin?         |
| **Type**            | Which runtime object layout is being allocated?         |
| **Tracing**         | Does it contain references to other managed objects?    |
| **Initialization**  | When do all fields become safe for the GC to observe?   |
| **Header lookup**   | How does a payload locate its metadata?                 |
| **Movement**        | May collection change the object's address?             |
| **Failure**         | Does failure trigger GC, return an error, or terminate? |

Conceptually:

```rust
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
enum RuntimeType {
    Text,
    Pair,
    Function,
    Bytecode,
}

#[derive(Clone, Copy, Debug)]
struct ObjectHeader {
    kind: RuntimeType,
    size_bytes: u32,
    marked: bool,
}
```

The real header layout is part of the runtime ABI. Changing it affects allocation,
tracing, debugging, and possibly serialized heap images.

### 4. Bump Allocation Mental Model

A bump allocator keeps a cursor inside a block:

```text
block start                         block end
| allocated objects | free space          |
                    ^
                  cursor
```

To allocate:

1. round the cursor up to the required alignment;
2. check whether `aligned_cursor + size` fits;
3. reserve that range;
4. advance the cursor;
5. initialize the object before exposing it.

Bump allocation is fast because the common path is arithmetic plus a bounds check. It
does not individually reclaim objects; a collector or arena reset handles reclamation.

Checked alignment helper:

```rust
fn align_up(offset: usize, alignment: usize) -> Option<usize> {
    if !alignment.is_power_of_two() {
        return None;
    }
    offset.checked_add(alignment - 1).map(|n| n & !(alignment - 1))
}

assert_eq!(align_up(13, 8), Some(16));
```

### 5. Runtime Values

A dynamically typed language binds type information to values at runtime:

```rust
#[derive(Clone, Debug)]
enum Value {
    Nil,
    Bool(bool),
    Int(i64),
    Float(f64),
    Text(GcRef),
    Function(FunctionId),
}

#[derive(Clone, Copy, Debug, PartialEq, Eq, Hash)]
struct GcRef(u32);

#[derive(Clone, Copy, Debug, PartialEq, Eq, Hash)]
struct FunctionId(u32);
```

This handle-based sketch keeps raw heap addresses out of most code. A real runtime may
use direct pointers, indices, generations, tagged words, or a mixture.

| Representation decision | Common choices                             |
| ----------------------- | ------------------------------------------ |
| Small integers          | Inline/tagged or boxed                     |
| Booleans and `nil`      | Immediate singleton values or heap objects |
| Strings                 | Interned, uniquely allocated, or mixed     |
| Object addresses        | Stable or movable during collection        |
| Equality                | Identity, contents, or type-dependent      |
| Stale handles           | Generation counters, tables, or validation |

### 6. Tagged Values and Tagged Pointers

Aligned pointers have low bits that are normally zero. Some runtimes use those bits as a
small type tag:

```text
aligned pointer: ppppppppppppp000
tagged value:    pppppppppppppttt
```

Possible tags might mean:

```text
000 → heap object pointer
001 → small integer
010 → symbol/interned identifier
011 → special value
```

| Tagged-value advantage                                 | Tagged-value cost                                               |
| ------------------------------------------------------ | --------------------------------------------------------------- |
| Identify common types without reading an object header | Masking and shifting require carefully reviewed unsafe code     |
| Keep small integers out of the heap                    | The inline integer range becomes smaller                        |
| Fit a complete dynamic value in one machine word       | Available tag bits depend on guaranteed alignment               |
| Reduce indirection for common values                   | Debuggers and sanitizers see encoded pointers                   |
| Improve locality for value stacks and register arrays  | Portability and pointer-provenance rules need deliberate design |

Keep the encoded representation private. Expose safe constructors and accessors that
validate tags before decoding.

### 7. Object Headers Versus Inline Tags

Inline tags cannot represent unlimited types. A hybrid runtime may use:

```text
value word
    ├── immediate integer/special value
    ├── directly tagged common object
    └── generic object pointer ──→ header contains full RuntimeType
```

The header is also a natural place for:

| Header field           | Purpose                                      |
| ---------------------- | -------------------------------------------- |
| Mark bits              | Record tracing state                         |
| Size or size class     | Locate boundaries and choose reclamation bin |
| Generation or age      | Support generational collection              |
| Forwarding information | Redirect references after movement           |
| Immutable/frozen state | Enforce guest-language mutation rules        |
| Identity hash          | Preserve stable identity-based hashing       |

Every extra header field costs memory for every object, so header design is a workload
trade-off.

### 8. Mutator Scopes and Rooting

The mutator and collector have conflicting views:

- normal execution wants stable access to individual objects;
- a moving collector may need exclusive access to the whole heap and must update every
  reference.

A safe design ensures they cannot run simultaneously.

```text
enter mutator scope
    ↓ obtain scoped handles
parse / compile / execute / allocate
    ↓ all temporary borrows end
leave mutator scope
    ↓ collector may trace or move objects
```

Long-lived guest references should normally be represented as traceable roots or
handles, not as unconstrained Rust references.

| Root source                   | Why it keeps objects alive                      |
| ----------------------------- | ----------------------------------------------- |
| VM value stack/register array | Contains currently executing values             |
| Active call frames            | Hold locals, closures, and return state         |
| Globals and modules           | Remain reachable by name                        |
| Interned symbols              | Are retained by the runtime's intern table      |
| Host-visible native handles   | Represent guest objects borrowed by native code |
| Pending exceptions            | Carry values until handled                      |
| Executable compiler constants | Remain reachable from live functions            |

If one live reference is absent from the root graph, a collector may reclaim a reachable
object.

### 9. Garbage-Collection Invariants

The exact collector can vary, but these invariants transfer:

| GC invariant       | Required property                                             |
| ------------------ | ------------------------------------------------------------- |
| Reachability       | Every live object is reachable through traceable edges        |
| Initialization     | Traceable fields are initialized before GC can observe them   |
| Layout knowledge   | The collector knows each object's layout or tracing procedure |
| Write barriers     | Mutator writes preserve the collector's required metadata     |
| Movement           | Every reference is updated when an object moves               |
| Reclamation        | No valid handle can reach reclaimed memory                    |
| Finalization       | Objects cannot resurrect through an untracked reference       |
| Collection failure | The heap remains in a defined state                           |

Basic tracing shape:

```text
roots
  ↓ mark
reachable object graph
  ↓ sweep unmarked
free space
```

Do not hold a normal Rust reference across an operation that may allocate if allocation
can trigger a moving collection.

### 10. Register-Based Bytecode

The hosted-languages guide uses a register-oriented VM influenced by Lua. Each compiled
function owns bytecode and literals, while call frames provide that function's
registers.

```rust
#[derive(Clone, Copy, Debug)]
struct Register(u16);

#[derive(Clone, Copy, Debug)]
struct LiteralId(u16);

#[derive(Clone, Debug)]
enum Opcode {
    LoadLiteral {
        dst: Register,
        literal: LiteralId,
    },
    Move {
        dst: Register,
        src: Register,
    },
    Add {
        dst: Register,
        left: Register,
        right: Register,
    },
    Return {
        src: Register,
    },
}

struct Bytecode {
    code: Vec<Opcode>,
    literals: Vec<Value>,
    register_count: u16,
}
```

The compiler sees a construction interface:

- append an instruction;
- append/deduplicate a literal;
- reserve a register;
- patch a forward jump after its destination is known.

The VM sees an execution interface:

- fetch the instruction at the instruction pointer;
- advance or replace the instruction pointer;
- access frame registers;
- switch bytecode when calling or returning.

These are two views of the same data, much like ELF's linking and execution views.

### 11. Call Frames

A register VM frame may contain:

```rust
struct CallFrame {
    function: FunctionId,
    instruction_pointer: usize,
    registers: Vec<Value>,
    return_destination: Option<Register>,
}
```

Real implementations often store all registers in one contiguous VM stack and let each
frame refer to a window. That avoids one `Vec` allocation per call.

| Frame invariant     | Required relationship                              |
| ------------------- | -------------------------------------------------- |
| Instruction pointer | Refers to the frame's own bytecode                 |
| Register index      | Is below the function's declared register count    |
| Arguments           | Occupy the calling convention's expected registers |
| Return destination  | Belongs to the caller and remains valid            |
| Live values         | Are visible to the garbage collector               |

### 12. Safe-Unsafe Boundary Checklist

| Area             | Review requirement                                                   |
| ---------------- | -------------------------------------------------------------------- |
| Isolation        | Keep raw allocation and pointer arithmetic in one small module       |
| Safety contract  | State alignment, initialization, aliasing, and lifetime requirements |
| Tagged pointers  | Never expose an encoded value as directly dereferenceable            |
| Domain types     | Use newtypes for IDs, sizes, offsets, handles, and registers         |
| GC coordination  | Make collection and mutation mutually exclusive                      |
| Edge tests       | Cover zero, maximum, overflow, and alignment-edge allocations        |
| Dynamic checks   | Run Miri and sanitizers where applicable                             |
| Fuzzing          | Fuzz bytecode verification and object decoding                       |
| Stable identity  | Prefer handles when raw addresses may move                           |
| Allocation audit | Treat every allocation point as a possible collection point          |

---

## 🌙 Building a Minimal Lua Interpreter in Rust

This section follows the incremental idea from
[Build a Lua Interpreter in Rust](https://wubingzheng.github.io/build-lua-in-rust/en/ch01-00.hello_world.html):
implement one tiny program through the complete pipeline, then grow the language without
discarding the architecture.

> 🌱 **Growth strategy:** make one tiny program work **end to end**, then expand one
> language feature and one test at a time.

### 1. A Small Program Contains Many Concepts

```lua
print "hello, world!"
```

Even this one line crosses the complete language pipeline:

| Stage              | Work required for `print "hello, world!"`            |
| ------------------ | ---------------------------------------------------- |
| Tokenizer          | Recognize an identifier and string literal           |
| Parser             | Build a function-call expression                     |
| Value model        | Represent a dynamic string and callable value        |
| Name resolution    | Resolve the global name `print`                      |
| Compiler           | Store the string constant and emit call instructions |
| Calling convention | Place the argument where the callee expects it       |
| Native boundary    | Invoke the standard-library implementation           |
| VM                 | Fetch, decode, and execute the generated bytecode    |

This is a good vertical slice: tiny surface area, complete pipeline.

### 2. Define Bytecode Before Filling in the Pipeline

Bytecode becomes a contract between compiler and VM:

```text
source
  ↓ lexer + parser + compiler
bytecode chunk
  ↓ virtual machine
observable result
```

For the tiny program:

```rust
#[derive(Clone, Copy, Debug)]
struct Register(u8);

#[derive(Clone, Copy, Debug)]
struct ConstantId(u16);

#[derive(Clone, Debug)]
enum Instruction {
    GetGlobal {
        dst: Register,
        name: ConstantId,
    },
    LoadConstant {
        dst: Register,
        constant: ConstantId,
    },
    Call {
        function: Register,
        argument_count: u8,
        result_count: u8,
    },
    Return,
}
```

Semantic trace:

```text
GetGlobal r0, "print"       r0 = built-in print function
LoadConstant r1, "hello"    r1 = string value
Call r0, 1, 0               call r0 with one argument, no results
Return                      finish the chunk
```

The exact bytecode is an implementation decision, not part of the Lua language
specification. Another compatible Lua implementation may use different instructions.

### 3. Chunk, Constants, and Globals Are Different

```rust
struct Chunk {
    code: Vec<Instruction>,
    constants: Vec<Value>,
    register_count: u8,
}

type Globals = std::collections::HashMap<String, Value>;
```

- The **constant table** belongs to compiled code and is addressed by compact indexes.
- The **global table** belongs to execution state and is addressed by language-level
  names.
- Bytecode can first load a name from constants, then use that name to query globals.

Keeping these separate prevents compile-time representation from being confused with
runtime state.

### 4. Dynamic Values as an Enum

```rust
type Builtin = fn(&[Value]) -> Result<Vec<Value>, RuntimeError>;

#[derive(Clone)]
enum Value {
    Nil,
    Bool(bool),
    Integer(i64),
    Float(f64),
    String(String),
    Builtin(Builtin),
}

#[derive(Debug)]
enum RuntimeError {
    UnknownGlobal(String),
    RegisterOutOfRange,
    ConstantOutOfRange,
    NotCallable,
    WrongArgumentCount,
}
```

The enum makes every operation explicitly handle the dynamic type. Later, heap-backed
strings, tables, closures, and userdata may use reference-counted or garbage-collected
handles instead of owned `String`.

### 5. Keep the First VM Loop Obvious

```rust
fn execute(chunk: &Chunk, globals: &Globals) -> Result<(), RuntimeError> {
    let mut registers = vec![Value::Nil; chunk.register_count as usize];
    let mut ip = 0;

    while let Some(instruction) = chunk.code.get(ip) {
        ip += 1;

        match instruction {
            Instruction::GetGlobal { dst, name } => {
                let name = match chunk.constants.get(name.0 as usize) {
                    Some(Value::String(name)) => name,
                    _ => return Err(RuntimeError::UnknownGlobal("<invalid name>".into())),
                };
                let value = globals
                    .get(name)
                    .cloned()
                    .ok_or_else(|| RuntimeError::UnknownGlobal(name.clone()))?;
                *registers
                    .get_mut(dst.0 as usize)
                    .ok_or(RuntimeError::RegisterOutOfRange)? = value;
            }
            Instruction::LoadConstant { dst, constant } => {
                let value = chunk
                    .constants
                    .get(constant.0 as usize)
                    .cloned()
                    .ok_or(RuntimeError::ConstantOutOfRange)?;
                *registers
                    .get_mut(dst.0 as usize)
                    .ok_or(RuntimeError::RegisterOutOfRange)? = value;
            }
            Instruction::Call {
                function,
                argument_count,
                result_count,
            } => {
                if *result_count != 0 {
                    return Err(RuntimeError::WrongArgumentCount);
                }
                let function_index = function.0 as usize;
                let args_start = function_index + 1;
                let args_end = args_start + *argument_count as usize;
                let args = registers
                    .get(args_start..args_end)
                    .ok_or(RuntimeError::RegisterOutOfRange)?;

                match registers.get(function_index) {
                    Some(Value::Builtin(function)) => {
                        let results = function(args)?;
                        if !results.is_empty() {
                            return Err(RuntimeError::WrongArgumentCount);
                        }
                    }
                    _ => return Err(RuntimeError::NotCallable),
                }
            }
            Instruction::Return => return Ok(()),
        }
    }

    Ok(())
}
```

This version favors clarity over optimization. Before adding features, add a bytecode
verifier so malformed instructions cannot index arbitrary registers or constants.

### 6. Module Layout That Can Grow

```text
src/
├── main.rs          # command-line entry point
├── lexer.rs         # source → tokens
├── parser.rs        # tokens → syntax / emitted bytecode
├── bytecode.rs      # instructions, chunk, indexes
├── value.rs         # dynamic runtime values
├── vm.rs            # execution state and dispatch loop
├── builtins.rs      # print and later standard-library functions
└── error.rs         # lexical, parse, compile, runtime errors
```

Split by runtime domain and responsibility. Do not make every internal type public.

### 7. Lexer and Parser Contract

The tiny grammar can be:

```text
program  := statement* EOF
statement := IDENTIFIER STRING
```

Tokens:

```rust
#[derive(Debug, PartialEq)]
enum Token<'a> {
    Identifier(&'a str),
    StringLiteral(String),
    End,
}
```

Even at this stage, retain source spans:

```rust
#[derive(Clone, Copy, Debug)]
struct Span {
    start: usize,
    end: usize,
}

struct Spanned<T> {
    item: T,
    span: Span,
}
```

Spans let later compiler and runtime errors point back to the original source.

### 8. Grow by Preserving Invariants

| Feature             | Compiler/runtime addition                   |
| ------------------- | ------------------------------------------- |
| Multiple statements | Loop until end-of-file                      |
| Local variables     | Scope table + register allocation           |
| Assignment          | L-value representation + store instruction  |
| Arithmetic          | Numeric coercion rules + arithmetic opcodes |
| Tables              | Heap object + indexed/name lookup           |
| Conditionals        | Conditional jump + patching                 |
| Loops               | Backward jumps + break targets              |
| Functions           | Prototypes, frames, call/return             |
| Closures            | Upvalues and open/closed capture state      |
| Garbage collection  | Roots, tracing, reclamation                 |

A jump offset may begin as forward-only and later need to become signed when loops add
backward edges. Representation choices should evolve when language semantics demand it.

### 9. Closures and Escaping Upvalues

An inner function may outlive the outer frame it references:

```lua
function make_counter()
    local count = 0
    return function()
        count = count + 1
        return count
    end
end
```

While the outer call is active, `count` can be an **open upvalue** referring to a stack
slot. When that frame returns, the runtime must **close** the upvalue by moving or
copying the captured value into heap-managed storage.

```text
active outer frame:
closure ──→ open upvalue ──→ stack slot

after outer return:
closure ──→ closed upvalue ──→ heap value
```

All closures sharing the same captured local must share the same upvalue cell, not
independent copies.

### 10. Differential and Layered Testing

| Test layer          | Assertion                                                   |
| ------------------- | ----------------------------------------------------------- |
| **Lexer**           | Exact token and span sequence                               |
| **Parser/compiler** | Exact instructions and constants                            |
| **Verifier**        | Malformed register, constant, and jump indexes are rejected |
| **VM**              | Hand-built chunks execute independently of parsing          |
| **End-to-end**      | Source text produces expected output                        |
| **Differential**    | Visible behavior matches reference Lua where intended       |
| **Snapshot**        | Bytecode listings and diagnostics remain readable/stable    |

When a test fails, ask which boundary first differs:

```text
source → tokens → chunk → VM state → output
```

---

## 🔖 V. Types & Type Inference

### Static vs. Dynamic Typing

Types act as **contracts** — when you write `a: int`, you promise `a` will always be an
integer. The compiler enforces that promise.

> 🏷️ **Type-system goal:** reject impossible or unsafe programs early while preserving
> useful expressiveness.

| Approach    | When Checked                  | Examples                 |
| ----------- | ----------------------------- | ------------------------ |
| **Static**  | Compile time (before running) | Rust, C, Java, Haskell   |
| **Dynamic** | Runtime (while running)       | Python, JavaScript, Ruby |

**Trade-offs:**

- **Static** — Catches bugs early, enables better performance, requires upfront
  annotations.
- **Dynamic** — More flexible, faster to prototype, but bugs can hide until runtime.

---

### Types Enable Fast Code

When the compiler knows `x` and `y` are `i64`, it generates a **single CPU instruction**
for `x * y`.

Without types, a dynamic interpreter must at runtime:

1. Check what type `x` is.
2. Look up the multiplication operation for that type.
3. Check operand compatibility.
4. Handle potential type errors.
5. Finally, multiply.

This overhead makes dynamically typed languages **10–100× slower** for numeric work.
This is why JIT compilers like **V8** (JavaScript) and **PyPy** (Python) invest heavily
in _type speculation_ — guessing types to generate fast paths.

---

### Type Inference

**Inference** means the compiler deduces types automatically — no annotations needed in
most cases. Types **flow forward** from known sources (literals, annotated parameters)
through operations into variables.

```
let x = 1 + 2   // x inferred as Int — no annotation needed
```

> **Type inference analogy:** Like solving a crossword puzzle. Some squares have letters
> (explicit annotations); others are blank (`Unknown`). Constraints like "this is added
> to an int, so it must be int" fill in the blanks.

Key mechanisms:

- **Unification** — Checks if two types are compatible and finds a common type.
  Resolving `Unknown` with a concrete type is how the compiler _learns_ what an unknown
  type should be.
- **Type Environment** — A `HashMap<String, Type>` mapping names to types. Also called
  symbol table or context

The environment is:

- **Extended** when we declare a variable or enter a function (adding new bindings)
- **Queried** when we reference a variable (looking up its type)
- **Scoped** — inner scopes can shadow outer bindings

---

### Two-Pass Function Type Checking

Functions are trickier because we need to handle:

- **Parameters** (types come from annotations)
- **Local variables** (types are inferred)
- **Return value** (must match declared return type)

Functions can call each other (mutual recursion), requiring two passes:

1. **Pass 1 — Collect Signatures** — Scan all function definitions and record their type
   signatures _before_ checking any bodies. This ensures `foo` can call `bar` even if
   `bar` is defined later.
2. **Pass 2 — Check Bodies** — Walk each function body, inferring and unifying types
   throughout.

**Per-expression pattern:** recursively type-check sub-expressions → apply typing rule →
set type on this node.

Type inference works by:

1. Starting with **known types** — literals (`42` → `Int`, `true` → `Bool`) and
   annotated parameters.
2. **Flowing types** through expressions — operators, calls, assignments.
3. **Recording** types in the environment so variables can be looked up later.
4. **Unifying** types — checking compatibility and resolving `Unknown`.
5. **Reporting errors** when types conflict.

---

### Type Inference Approaches

| Approach            | Description                                                                        |
| ------------------- | ---------------------------------------------------------------------------------- |
| **Hindley-Milner**  | Infers polymorphic types like `fn identity<T>(x: T) -> T` without any annotations  |
| **Local Inference** | Requires annotations at function boundaries; infers types _within_ function bodies |

---

### The Typed (Decorated) AST

Secondlang's type checker doesn't just validate types — it produces a new tree where
**every expression carries its inferred type**. This is called a **decorated** or
**annotated** AST:

```rust
/// A typed expression: expression + its inferred type
pub struct TypedExpr {
    pub expr: Expr,
    pub ty: Type,
}
```

Nodes start as `Type::Unknown` and get filled in during inference. Grammar-wise,
Secondlang barely changes from Firstlang — the expression rules (`Expr`, `Comparison`,
`Additive`, ...) are identical. Only `Function`, `TypedParam`, `ReturnType`, `Type`, and
`Assignment` gain optional type annotations; the real growth happens in the compiler,
not the grammar.

---

## 🚀 VI. Compiler Optimizations

Optimizations simplify the AST before code generation, producing faster output, cleaner
debug trees, and less work for the backend. They are chained into a **pass pipeline** —
order matters!

> ⚠️ **Optimization invariant:** the transformed program must preserve every behavior
> the language specification says is observable.

```
AST → [Constant Folding] → [Algebraic Simplification] → [Dead Code Elimination] → Optimized AST
```

| Optimization                         | Description                                    |
| ------------------------------------ | ---------------------------------------------- |
| **Constant Folding**                 | `1 + 2` → `3` at compile time                  |
| **Algebraic Simplification**         | `x * 0` → `0`, strength reduction              |
| **Dead Code Elimination**            | Remove unreachable branches                    |
| **Common Subexpression Elimination** | Compute identical sub-expressions once         |
| **Loop Unrolling**                   | Replace loops with repeated sequential code    |
| **Inlining**                         | Substitute function body directly at call site |
| **Tail Call Optimization**           | Convert tail recursion into a flat loop        |

> Even when LLVM will optimize downstream, custom passes improve compile speed, debug
> output readability, and can exploit language-specific knowledge LLVM can't.

---

## 🏭 VII. LLVM IR & Advanced Code Generation

> LLVM is like a universal translator for CPUs. You speak LLVM IR; LLVM translates it to
> x86, ARM, WebAssembly — whatever you need. Write your compiler frontend once; LLVM
> gives you every platform for free.
>
> 🏭 **Backend boundary:** your frontend must produce **valid, well-typed IR**; LLVM
> cannot repair incorrect language semantics or undefined assumptions.

**LLVM IR** is a universal, low-level assembly language not tied to any specific CPU. It
is used by Rust, Swift, Julia, Kotlin/Native, and more. By targeting LLVM IR, you get
world-class optimizations for free.

---

### Core IR Mechanics

#### SSA (Static Single Assignment)

In LLVM IR, every variable is assigned **exactly once**. This makes optimizations like
dead code elimination and constant propagation trivial — the compiler always knows
exactly where each value was defined.

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

LLVM's **`mem2reg` pass** promotes these stack slots to fast registers automatically.

**Why store parameters too?** For `add(%a, %b)`, the pattern is: (1) `alloca` stack
space for `%a.addr`/`%b.addr`, (2) `store` the incoming parameters into those slots, (3)
`load` the values back, (4) `add i64` the loaded values, (5) `ret` the result. This
looks wasteful — why not use `%a` and `%b` directly? Because in LLVM IR, SSA values like
`%a` are immutable, but in most source languages variables _can_ change. Storing every
variable in a stack slot up front handles mutability uniformly, and `mem2reg` cleans it
up afterward when a variable is never actually reassigned.

#### Basic Blocks & Branching

Conditionals require separate **basic blocks** (`entry`, `then`, `else`, `merge`). Each
block ends with a **terminator** — either `ret` (return) or `br` (branch):

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

In SSA form, if a variable's value depends on which branch was taken, a **phi node**
selects the correct value based on the incoming basic block:

```llvm
merge:
  %x = phi i64 [ %x.then, %then ], [ %x.else, %else ]
```

"If we came from `%then`, use `%x.then`. If we came from `%else`, use `%x.else`." Phi
nodes are the **only** way to merge values from different control flow paths in SSA.

#### Recursion

Recursive functions use the standard `call` instruction:

```llvm
%result = call i64 @fib(i64 %n_minus_1)
```

#### Function Calls & Return Values

To compile a call: look up the function, compile each argument, then emit a `call`
instruction. LLVM calls return either a **basic value** (an int or pointer you can use)
or **nothing** (void). The pattern `try_as_basic_value().unwrap_basic()` extracts the
basic-value case — safe here since our functions always return `int` — and
`.into_int_value()` converts the result to the specific integer type needed downstream.

---

### LLVM State

| Component           | Role                                                            |
| ------------------- | --------------------------------------------------------------- |
| **Context**         | Workspace for all LLVM objects                                  |
| **Module**          | Container for functions (one per compilation unit)              |
| **Builder**         | Inserts IR instructions into a basic block                      |
| **`variables` map** | Maps variable names → stack pointers (`alloca` results)         |
| **`functions` map** | Maps function names → LLVM function objects                     |
| **`current_fn`**    | The function being compiled (needed to create new basic blocks) |

---

### The Three-Pass Compilation Process

The Prestage stage:

- **`Context::create()`** - The top-level container. All LLVM objects belong to a
  context. It manages memory and ensures thread safety.
- **`context.create_module("addition")`** - Creates a module (like a compilation unit).
  Our `add` function will live here.
- **`context.i32_type()`** - Gets the 32-bit integer type. LLVM is explicitly typed - we
  need to declare that our function works with `i32`.

#### Pass 1 — Declare Functions

Announce all function signatures before compiling any body. This enables mutual
recursion through forward references.

- **`i32_type.fn_type(&[i32_type.into(), i32_type.into()], false)`** - Creates a
  function type: returns `i32`, takes two `i32` parameters. The `false` means it's not
  variadic (doesn't take variable arguments like `printf`).
- **`module.add_function("add", fn_type, None)`** - Adds a function called "add" with
  this signature to our module.
- **`context.append_basic_block(add_fn, "entry")`** - Creates a basic block named
  "entry". A basic block is a sequence of instructions with no branches in the middle -
  execution flows straight through.
- **`context.create_builder()`** - The builder is our "cursor" for adding instructions.
  We position it at a basic block, then build instructions there.
- **`builder.position_at_end(entry)`** - Point the builder at our entry block. New
  instructions will go here.

#### Pass 2 — Compile Bodies

Generate IR instructions for each function body using the alloca/load/store pattern.

- **`add_fn.get_nth_param(0)`** - Get the first parameter. LLVM functions have an array
  of parameters, indexed from 0.
- **`.unwrap().into_int_value()`** - Parameters come as generic "basic values." We know
  ours are integers, so we convert them.
- **`builder.build_int_add(x, y, "result")`** - This emits an `add` instruction. The
  `"result"` is just a name for the output (helps when reading IR).
- **`builder.build_return(Some(&sum))`** - Emit a `ret` instruction to return our sum.

#### Pass 3 — Create the `@__main` Wrapper

Wrap top-level expressions, such as `fib(10)`, in `__main` as the JIT entry point, then
verify the module.

- **`module.create_jit_execution_engine(OptimizationLevel::None)`** - Creates a JIT
  compiler. LLVM takes our IR and compiles it to native x86/ARM code _right now_, in
  memory.
- **`execution_engine.get_function::<unsafe extern "C" fn(i32, i32) -> i32>("add")`** -
  Look up our compiled function. The type signature tells Rust how to call it.
- **`add.call(1, 2)`** - Call the native function! This jumps directly to machine code -
  no interpretation, no overhead.

---

### Running a Program: Source to Result

What happens when you run `cargo run -- examples/fibonacci.sl`:

1. **Parse** the source file → Typed AST (with `Unknown` types).
2. **Type check** → Typed AST (all types resolved).
3. **Optimize** (optional) → Simplified AST.
4. **Compile** → LLVM IR.
5. **JIT** → Native machine code.
6. **Execute** → Result.

All in a fraction of a second. Each step transforms the program into a different
representation, getting closer and closer to something the CPU can execute directly.

**Calling the JIT-compiled code:**

1. Create a code generator — sets up the LLVM context, module, and builder (the
   workspace).
2. Compile the program to IR.
3. Create a JIT execution engine from the verified module.
4. Get a pointer to `@__main` — our entry point.
5. Call it and return the result.

The JIT engine compiles IR to native machine code on the fly, then executes it — this is
much faster than interpretation, because the CPU is running actual machine code, not
walking a tree.

> The `unsafe` block required when calling JIT output signals that we are invoking raw
> machine code — we must trust that our code generator produced valid IR.

---

### LLVM Optimization Passes

After code generation, LLVM passes transform naive IR into efficient code. Passes are
chained in a **pipeline string** (e.g., `"mem2reg,dce,instcombine,simplifycfg"`).

**Why optimize ourselves if LLVM will do it anyway?**

| Reason                        | Benefit                                                                   |
| ----------------------------- | ------------------------------------------------------------------------- |
| **Learning**                  | Implementing optimizations teaches how production compilers actually work |
| **Simplicity**                | A simpler AST means simpler, less error-prone code generation             |
| **Debug output**              | Optimized IR is easier to read when printed for debugging                 |
| **Specialized optimizations** | You may know language-specific facts LLVM cannot infer                    |
| **Compile time**              | A simpler AST means less work for LLVM, so compilation is faster          |

| Pass              | What It Does                                                                              |
| ----------------- | ----------------------------------------------------------------------------------------- |
| **`mem2reg`**     | Promotes `alloca` stack slots to SSA registers — the most impactful pass                  |
| **`dce`**         | Dead Code Elimination — removes instructions whose results are never used                 |
| **`instcombine`** | Merges redundant instructions; constant folds; strength-reduces (`mul x, 2` → `shl x, 1`) |
| **`simplifycfg`** | Removes empty blocks, merges single-predecessor blocks, simplifies trivial branches       |

**Before/after example — `increment` method (14 → 4 instructions):**

```llvm
; BEFORE (naive codegen)
define i64 @Counter__increment(ptr %self) {
entry:
  %self1 = alloca ptr                     ; alloca for parameter
  store ptr %self, ptr %self1             ; store param to stack
  %self2 = load ptr, ptr %self1           ; load self
  %field_ptr = getelementptr %Counter, ptr %self2, i32 0, i32 0
  %field = load i64, ptr %field_ptr       ; load self.value
  %add = add i64 %field, 1
  %self3 = load ptr, ptr %self1           ; load self AGAIN
  %field_ptr4 = getelementptr %Counter, ptr %self3, i32 0, i32 0
  store i64 %add, ptr %field_ptr4
  %self5 = load ptr, ptr %self1           ; load self A THIRD TIME
  %field_ptr6 = getelementptr %Counter, ptr %self5, i32 0, i32 0
  %field7 = load i64, ptr %field_ptr6     ; load for return
  ret i64 %field7
}

; AFTER mem2reg → instcombine → dce
define i64 @Counter__increment(ptr %self) {
entry:
  %field = load i64, ptr %self            ; no alloca, GEP simplified away
  %add = add i64 %field, 1
  store i64 %add, ptr %self
  ret i64 %add                            ; return register directly
}
```

**Each pass in isolation:**

`mem2reg` — promotes an alloca'd variable straight to a value:

```llvm
; before                       ; after
%x = alloca i64                %val = 42
store i64 42, ptr %x
%val = load i64, ptr %x
```

`dce` — drops instructions whose results are never read:

```llvm
; before                        ; after
%unused = add i64 %a, %b        %result = mul i64 %c, %d
%result = mul i64 %c, %d        ret i64 %result
ret i64 %result
```

`instcombine` — simplifies arithmetic patterns: `sub i64 %x, 1` → `add i64 %x, -1`;
`mul i64 %x, 2` → `shl i64 %x, 1` (shift left); constant-folds `add i64 3, 4` → `7`.

`simplifycfg` — removes empty basic blocks, merges blocks with a single predecessor, and
simplifies trivially-true/false branches.

**LLVM preset pipelines:**

| Level         | Description                         |
| ------------- | ----------------------------------- |
| `default<O0>` | No optimization (verification only) |
| `default<O1>` | Light optimization                  |
| `default<O2>` | Standard optimization (recommended) |
| `default<O3>` | Aggressive optimization             |

**CLI usage:**

```bash
thirdlang examples/point.tl                        # Run without optimization
thirdlang -O examples/point.tl                     # Run with optimization
thirdlang --passes "mem2reg,dce" examples/point.tl # Custom pass pipeline
thirdlang --ir examples/point.tl                   # Print unoptimized IR
thirdlang --ir -O examples/point.tl                # Print optimized IR
thirdlang --passes "default<O2>" examples/point.tl # LLVM O2 preset
```

**Setting up the pass manager** requires:

1. **Initialize Native Target** — required before creating a `TargetMachine`.
2. **Get Target Triple** — the host machine description (e.g., `x86_64-apple-darwin`).
3. **Create TargetMachine** — needed for target-specific optimizations.
4. **`run_passes`** — takes the comma-separated pass list (or a preset like
   `"default<O2>"`) and applies it to the module.

---

## 🧪 Testing LLVM Codegen with Inkwell

The [`tests/all` suite in Inkwell](https://github.com/TheDan64/inkwell/tree/master/tests/all)
is useful as **executable API documentation**. Its files are organized around LLVM
concepts—builders, blocks, contexts, modules, targets, types, values, passes, debug
information, object files, and the execution engine—rather than around one large demo.

### 1. Read Upstream Tests as a Capability Map

| Upstream test area                        | Question for your compiler                        |
| ----------------------------------------- | ------------------------------------------------- |
| `test_types.rs` / `test_values.rs`        | Are guest types represented exactly as intended?  |
| `test_builder.rs` / `test_basic_block.rs` | Is every block well formed and terminated?        |
| `test_module.rs`                          | Can modules verify, link, clone, and serialize?   |
| `test_execution_engine.rs`                | Does generated IR compute the correct result?     |
| `test_targets.rs` / `test_object_file.rs` | Can the backend emit for the selected target?     |
| `test_passes*.rs`                         | Does optimization preserve behavior?              |
| `test_debug_info.rs`                      | Can source spans survive into debugger metadata?  |
| `test_attributes.rs`                      | Are calling and optimization contracts attached?  |
| `test_intrinsics.rs`                      | Are target-independent LLVM operations declared?  |
| `test_memory_buffer.rs`                   | Can IR/bitcode enter through a byte-oriented API? |

Inkwell's `tests/all/main.rs` declares these as modules so Cargo builds them into a
single integration-test binary. It also uses `cfg` conditions for LLVM-version-sensitive
tests. That is a strong model for a backend supporting several LLVM versions:

```text
shared semantic contract
    ├── always-on type, value, builder, and module tests
    ├── target-dependent emission tests
    └── LLVM-version-gated pass and feature tests
```

### 2. Verify Every Constructed Module

LLVM IR is typed, but a sequence of individually valid builder calls can still produce
an invalid function or module. Run the verifier immediately after the smallest useful
construction.

```rust
use inkwell::context::Context;
use inkwell::module::Module;

fn build_add<'ctx>(context: &'ctx Context) -> Module<'ctx> {
    let module = context.create_module("arithmetic");
    let builder = context.create_builder();
    let i64_type = context.i64_type();
    let function_type = i64_type.fn_type(
        &[i64_type.into(), i64_type.into()],
        false,
    );
    let function = module.add_function("add_i64", function_type, None);
    let entry = context.append_basic_block(function, "entry");
    builder.position_at_end(entry);

    let lhs = function
        .get_nth_param(0)
        .expect("parameter 0 exists")
        .into_int_value();
    let rhs = function
        .get_nth_param(1)
        .expect("parameter 1 exists")
        .into_int_value();
    let sum = builder
        .build_int_add(lhs, rhs, "sum")
        .expect("builder is positioned in a block");
    builder
        .build_return(Some(&sum))
        .expect("return type matches function type");

    module.verify().expect("generated module must be valid");
    module
}
```

> 🧭 **Backend invariant:** each reachable basic block ends with exactly one terminator,
> every SSA use is dominated by its definition, and instruction/result types agree.

### 3. Test at Increasing Levels of Meaning

| Test level       | What it catches                              | Typical assertion                 |
| ---------------- | -------------------------------------------- | --------------------------------- |
| Builder result   | Missing insertion point or invalid operation | `Result` is `Ok`                  |
| Function verify  | Local control-flow/type defects              | `function.verify(true)`           |
| Module verify    | Cross-function/module defects                | `module.verify().is_ok()`         |
| IR inspection    | Names, structure, attributes, calling form   | selected normalized IR fragments  |
| JIT execution    | Wrong semantics despite valid IR             | returned value equals oracle      |
| Object emission  | Target/data-layout/relocation mistakes       | object parses and symbols exist   |
| Differential run | Optimization or backend disagreement         | interpreter result equals JIT/AOT |

Do not snapshot an entire LLVM module for every test. Full snapshots are noisy across
LLVM versions. Prefer semantic execution tests plus focused IR assertions for features
whose representation matters.

### 4. JIT Lookup Is an Unsafe ABI Boundary

`ExecutionEngine::get_function` is unsafe because Rust must promise that the requested
function-pointer type matches the generated function's ABI and signature. The engine
must also outlive the pointer.

```rust
use inkwell::OptimizationLevel;

type AddI64 = unsafe extern "C" fn(i64, i64) -> i64;

#[test]
fn generated_add_has_the_expected_semantics() {
    let context = Context::create();
    let module = build_add(&context);
    let engine = module
        .create_jit_execution_engine(OptimizationLevel::None)
        .expect("native JIT is available");

    // SAFETY:
    // - generated `add_i64` uses the C ABI;
    // - it accepts exactly two i64 values and returns one i64;
    // - `engine` remains alive while the JIT function is called.
    let add = unsafe {
        engine
            .get_function::<AddI64>("add_i64")
            .expect("generated function exists")
    };

    // SAFETY: the lookup contract above establishes the callable signature.
    assert_eq!(unsafe { add.call(20, 22) }, 42);
}
```

Wrap this unsafe lookup behind a small, typed backend API. Do not let arbitrary strings
and arbitrary Rust function-pointer types spread through the compiler.

### 5. Build a Compiler Compatibility Matrix

| Dimension      | Minimum matrix                                       |
| -------------- | ---------------------------------------------------- |
| LLVM version   | each version/feature combination you claim           |
| Optimization   | unoptimized plus the production pass pipeline        |
| Target         | host plus every cross-target you distribute          |
| Execution mode | interpreter/reference, JIT, and AOT where applicable |
| Numeric edges  | zero, extrema, overflow policy, NaN where applicable |
| Control flow   | both branch arms, loop zero/one/many iterations      |
| Debug mode     | verifier and debug-info checks enabled               |

For your own language, a high-value oracle is:

```text
source
  ├── tree-walking interpreter ──→ expected value
  ├── unoptimized LLVM JIT ──────→ actual value A
  └── optimized LLVM JIT/AOT ───→ actual value B

require expected == A == B
```

When those disagree, preserve the source, seed, IR before/after optimization, LLVM
version, target triple, and data layout. That turns a mysterious codegen bug into a
reproducible backend test.

> ➡️ **Next:** object layout applies the same type, memory, and code-generation rules to
> user-defined data. The debugging section then shows how to isolate failures by stage.

---

## 📦 VIII. Object-Oriented Concepts (Classes)

### Why Classes?

Without classes, related data is **scattered**. This works, but causes problems:

- **No semantic grouping** — nothing says `x1` and `y1` belong together; they're just
  two independent integers.
- **Easy to mix up** — accidentally using `x1` with `y2` isn't caught by the compiler.
- **Can't pass as a unit** — you can't write `distance(p1, p2)`; you need
  `distance(x1, y1, x2, y2)`.
- **No encapsulation** — the distance formula is scattered across your code, so changing
  it means finding every place it was computed.

We need a way to group related data and attach behavior to it — that's what classes give
us.

```
# Without classes — error-prone
distance(x1, y1, x2, y2)

# With classes — clear, grouped, safe
p1.distance(p2)
```

> **Classes analogy:** Think of a filing cabinet. Without classes you have loose papers
> (`x1`, `y1`). Classes are folders that group related papers together — and know what
> operations to perform on them.

---

### OOP Vocabulary

| Concept         | Description                    | Example               |
| --------------- | ------------------------------ | --------------------- |
| **Class**       | Blueprint for creating objects | `class Point { ... }` |
| **Object**      | An instance of a class         | `p = new Point(1, 2)` |
| **Field**       | Data stored in an object       | `self.x`, `self.y`    |
| **Method**      | Function attached to a class   | `def distance(self)`  |
| **Constructor** | Initializes a new object       | `def __init__(self)`  |
| **Destructor**  | Cleans up before deletion      | `def __del__(self)`   |

---

### Classes as Custom Types

Classes define **new types** in a **nominal type system** — types are identified by
their names:

```
class Point   { x: int  y: int  ... }
class Counter { count: int  ... }

def move(p: Point, dx: int) -> Point { ... }
#         ^^^^^  Point is now a first-class type like int
```

---

### Design Decisions

We implement a **subset** of OOP — deliberately simple:

**No Inheritance** — Many OOP languages support `class B extends class A`. We skip this
because it adds significant complexity (vtables, dynamic dispatch), and composition over
inheritance is often preferred anyway. The core concepts are clearer without it.

**Explicit Memory Management** — Instead of GC, we use explicit `delete`:

```
p = new Point(1, 2)
delete p   # programmer's responsibility
```

This mirrors C++ and teaches how memory actually works. Understanding manual management
helps you appreciate what GC, reference counting, and ownership models solve.

**What we include vs exclude:**

| Included                        | Excluded                                |
| ------------------------------- | --------------------------------------- |
| Class definition with fields    | Inheritance (vtables, dynamic dispatch) |
| Methods with `self`             | Interfaces / Traits                     |
| Constructor `__init__`          | Visibility (`public`/`private`)         |
| Destructor `__del__`            | Static methods                          |
| `new` / `delete`                | Operator overloading                    |
| Field access `p.x` / `self.x`   |                                         |
| Method calls `p.method()`       |                                         |
| Classes as types `other: Point` |                                         |

---

### Memory Model: Stack vs Heap

In most primitive/variable contexts, values live on the **stack**. Objects live on the
**heap** because they must outlive the function that created them and can be shared
across references.

|                | Stack                             | Heap                       |
| -------------- | --------------------------------- | -------------------------- |
| **Management** | Automatic (LIFO with call frames) | Manual (`new`/`delete`)    |
| **Speed**      | Fast (just move a pointer)        | Slower (system call to OS) |
| **Size**       | Limited (~few MB)                 | Large (all available RAM)  |
| **Lifetime**   | Tied to function scope            | Until explicitly freed     |

> **Analogy:** The stack is a stack of cafeteria trays — same size, add/remove from top
> only. The heap is a parking lot — park anywhere, leave as long as you want, but you
> must retrieve it or it stays forever (memory leak).

---

### Constructors & `new`

The **constructor** (`__init__`) initializes a new object. It always takes `self` as the
first parameter.

When you write `p = new Point(10, 20)`:

1. **Calculate size** — `Point` has two `i64` fields → 16 bytes.
2. **Call `malloc`** — Ask the OS for 16 bytes; get back a pointer.
3. **Initialize fields** — Zero-initialize all fields as a safety baseline.
4. **Call `__init__`** — Runs with the new pointer as `self`; sets `self.x = 10`,
   `self.y = 20`.
5. **Return pointer** — `p` now holds the heap address of the object.

**Memory layout:**

```
class Point {
    x: int    # offset 0,  8 bytes (i64)
    y: int    # offset 8,  8 bytes (i64)
}             # total: 16 bytes

LLVM: %Point = type { i64, i64 }
```

Field order (tracked via `field_order` in `ClassInfo`) determines the memory layout —
order matters!

---

### Constructor Patterns

**Default values:**

```
class Config {
    value: int
    enabled: bool

    def __init__(self) {
        self.value = 42       # Default
        self.enabled = true   # Default
    }
}
```

**Computed initialization:**

```
class Square {
    side: int
    area: int

    def __init__(self, side: int) {
        self.side = side
        self.area = side * side   # Computed from input
    }
}
```

**Validation (clamping, since we have no exceptions):**

```
class PositiveInt {
    value: int

    def __init__(self, v: int) {
        if (v < 0) {
            self.value = 0   # Clamp to valid range
        } else {
            self.value = v
        }
    }
}
```

**Failure:** Our constructors cannot fail. Real languages handle this via exceptions
(Java, Python), `Result`/`Option` (Rust), or factory methods. We keep it simple —
constructors always succeed.

---

### The `self` Parameter

`self` is a **pointer** to the object the method was called on. It is always the first
parameter of every method — explicit, like Python:

```python
# Thirdlang / Python style — explicit self
def get_x(self) -> int {
    return self.x
}

# Call: p.get_x()  →  compiled as: Point__get_x(p)
```

| Language   | Self/This                      |
| ---------- | ------------------------------ |
| Python     | `def method(self):` — explicit |
| Rust       | `fn method(&self)` — explicit  |
| Java / C++ | `this` — implicit              |
| Thirdlang  | `def method(self)` — explicit  |

Explicit `self` makes it clear: **methods are just functions that receive the object as
their first argument**.

**In codegen,** `self` is just stored as a local variable pointer, exactly like any
other parameter — there's no special-casing:

```rust
Expr::SelfRef => {
    let ptr = self.variables.get("self").ok_or("'self' not in scope")?;
    self.builder.build_load(ptr_type, *ptr, "self").unwrap()
}
```

---

### Methods & Field Access

Methods compile to regular functions with the naming convention `ClassName__methodName`
— avoiding collisions between classes and making ownership clear.

**Field access via `getelementptr` (GEP):**

GEP calculates the memory address of a struct field without reading memory — it is
pointer arithmetic that knows struct layouts:

```llvm
; return self.x
%x_ptr = getelementptr %Point, ptr %self, i32 0, i32 0  ; pointer to field 0
%x     = load i64, ptr %x_ptr
ret i64 %x

; self.x = 42
%x_ptr = getelementptr %Point, ptr %self, i32 0, i32 0
store i64 42, ptr %x_ptr
```

**Method call `p.get_x()` → `call @Point__get_x(ptr %p)`** — the object is passed as the
first argument.

> **Important:** When you pass an object as a parameter, you pass a **pointer** — not a
> copy. Modifying `other.x` inside a method modifies the original object. This is
> **reference semantics**.

---

### Method Patterns

**Calling methods on `self`** — methods can call other methods on the same object:

```
class Calculator {
    value: int

    def __init__(self) { self.value = 0 }

    def add(self, n: int) {
        self.value = self.value + n
    }

    def double(self) {
        self.add(self.value)   # Call another method on self
    }
}
```

**Returning self's type** — enables builder pattern:

```
class Builder {
    value: int

    def __init__(self) { self.value = 0 }

    def set_value(self, v: int) -> Builder {
        self.value = v
        return self   # Return the same object
    }
}

b = new Builder()
b.set_value(10)
```

**Method naming convention** (`ClassName__methodName` in LLVM):

| Method              | LLVM Function         |
| ------------------- | --------------------- |
| `Point.__init__`    | `@Point____init__`    |
| `Point.get_x`       | `@Point__get_x`       |
| `Counter.increment` | `@Counter__increment` |

This avoids name collisions between classes and lets the JIT find the right function at
call sites.

---

### Destructors & `delete`

The **destructor** (`__del__`) is called automatically when you `delete` an object:

```
delete p
```

Behind the scenes:

1. Call `Point__del(p)` — runs any cleanup code.
2. Call `free(p)` — returns memory to the OS.

```llvm
; delete p
call void @Point__del(ptr %p)   ; destructor (if defined)
call void @free(ptr %p)         ; free heap memory
```

After `delete`, `p` still holds the old address — but accessing it is **undefined
behavior**.

---

### LLVM Codegen for Classes — Summary

| Thirdlang                | LLVM IR                                  |
| ------------------------ | ---------------------------------------- |
| `class Point { x: int }` | `%Point = type { i64 }`                  |
| `new Point(10)`          | `call @malloc` + `call @Point____init__` |
| `p.x`                    | `getelementptr` + `load`                 |
| `p.x = 5`                | `getelementptr` + `store`                |
| `p.method()`             | `call @Point__method(ptr %p)`            |
| `delete p`               | `call @Point____del__` + `call @free`    |

**Class compilation happens in six phases:**

1. **Declare libc functions** — `malloc` and `free`.
2. **Create class struct types** — define an LLVM struct for each class.
3. **Declare methods** — create function signatures (enables cross-method and forward
   calls).
4. **Compile class bodies** — generate method implementations.
5. **Compile top-level code** — generate the `@__main` wrapper.
6. **Verify module** — check the IR is well-formed.

---

### Complete Codegen Example

Tracing `Counter` from source to LLVM IR:

```
class Counter {
    count: int
    def __init__(self) { self.count = 0 }
    def increment(self) -> int {
        self.count = self.count + 1
        return self.count
    }
}
c = new Counter()
c.increment()
```

Generated IR:

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

> The code that runs isn't interpreted — it's real compiled machine code, the same as if
> you'd written it in C or Rust. Objects really live on the heap. Methods really jump to
> function addresses. When this calls `malloc`, it's calling the actual C `malloc`.

**Performance considerations:**

| What We Do                              | What Real Compilers Add                              |
| --------------------------------------- | ---------------------------------------------------- |
| Direct field access via GEP (fast)      | Inline method calls when possible                    |
| Static method calls — no vtable lookup  | Escape analysis (stack-allocate short-lived objects) |
| Objects laid out contiguously in memory | Field alignment optimization, dead field elimination |

---

### Memory Layout Trace

Tracing `new Point(10, 20)` followed by `delete p` at the address level:

**`new Point(10, 20)`:**

1. `malloc(16)` → returns pointer `0x1000`
2. `Point____init__(0x1000, 10, 20)` — sets `x=10` at offset 0, `y=20` at offset 8
3. `p` holds `0x1000`

**`delete p`:**

1. `Point____del__(0x1000)` — runs destructor cleanup
2. `free(0x1000)` — returns memory to OS
3. `p` still holds `0x1000` — but it is **invalid** (dangling pointer)

---

### Best Practices (Manual Memory)

#### 1. Delete What You Allocate

Every `new` has a matching `delete`.

```
p = new Point(1, 2)
# ... use p ...
delete p
```

#### 2. Keep One Clear Owner

Have one clear owner responsible for deletion:

```
def make_point() -> Point {
    return new Point(1, 2)   # Caller owns this
}
p = make_point()
delete p   # Caller is responsible
```

#### 3. Clear or Invalidate After Release

Set the reference to null after deletion when the language supports it:

```
delete p
p = null   # Mark as invalid — prevents use-after-free
```

---

### Memory Safety Bugs

Languages without GC (C, C++) require explicit `new`/`delete`, introducing high-risk
bugs:

| Bug                  | Description                                 | Code                               |
| -------------------- | ------------------------------------------- | ---------------------------------- |
| **Memory Leak**      | Forgot `delete`; memory consumed until exit | `p = new Point(1,2)` — never freed |
| **Use After Free**   | Access after `delete` — undefined behavior  | `delete p; p.x`                    |
| **Double Free**      | `delete` twice — corrupts allocator         | `delete p; delete p`               |
| **Dangling Pointer** | Two refs to same object; one deletes it     | `q = p; delete p; q.x`             |

```cpp
// Memory Leak
def leak() {
    p = new Point(1, 2)
    // forgot delete p — memory lost until program exits
}

// Use After Free — undefined behavior
p = new Point(1, 2)
delete p
p.x           // BUG

// Double Free — undefined behavior
delete p
delete p      // BUG

// Dangling Pointer
q = p         // q and p point to same object
delete p
q.x           // BUG — q points to freed memory
```

---

### Memory Management Approaches

| Approach                               | Pros                                    | Cons                        |
| -------------------------------------- | --------------------------------------- | --------------------------- |
| **Manual (C, C++)**                    | Fast, predictable, teaches fundamentals | Error-prone                 |
| **Garbage Collection (Java, Python)**  | Safe, convenient                        | Runtime overhead, GC pauses |
| **Reference Counting (Swift, Python)** | Predictable cleanup                     | Cycle leaks, overhead       |
| **Ownership (Rust)**                   | Safe with no runtime cost               | Complex ownership rules     |

---

## 🐛 IX. Debugging

### Compiler Pipeline Debugging

> Your language is a pipeline: `Source → Tokens → AST → Output`. When something breaks,
> find which stage produced the wrong output.

**🐛 Systematic approach:**

1. **Reproduce** — Find the smallest input that triggers the bug.
2. **Isolate** — Which stage is producing wrong output?
3. **Inspect** — Print the data at that stage.
4. **Fix** — Change the code.
5. **Verify** — Re-run the test.

| Symptom          | Action                                           |
| ---------------- | ------------------------------------------------ |
| Parse error      | Simplify input; inspect lexer tokens             |
| Wrong result     | Print the AST; check grammar operator precedence |
| Program crash    | Add debug prints; run `cargo clippy`             |
| Infinite loop    | Add print statements inside the `eval` loop      |
| Precedence wrong | Print AST and verify grammar rule order          |

**Debugging tips:**

- Test each feature in isolation — don't write 100 lines then debug.
- Use the REPL for quick experiments.
- Print the AST — structure reveals bugs that output doesn't.
- Check operator precedence by inspecting which node is the AST root.

---

### Testing the Pipeline

Test at multiple levels:

| Level           | What to Test                                             |
| --------------- | -------------------------------------------------------- |
| **Unit**        | Individual functions — parser output, type checker rules |
| **Integration** | Multiple components — parse + typecheck + codegen        |
| **End-to-end**  | Full programs from source to result                      |
| **Snapshot**    | IR and error message output — catch regressions          |

Start with integration tests for full programs, then add unit tests for complex logic.
Add a regression test for every bug fixed.

---

### Reverse Engineering & Debugging Compiled Binaries

Everything above is about debugging _your own_ compiler pipeline while you have the
source. A different (and complementary) skill is understanding **someone else's
already-compiled binary**, where you only have opcodes — this is the mirror image of
everything in [Hardware Fundamentals](#-i-hardware-fundamentals): instead of going source
→ assembly → opcodes, you're going opcodes → assembly → (partial) understanding.

**Static vs. dynamic analysis:**

| Approach             | What it means                                                        | When it's used                                                    |
| -------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **Static analysis**  | Disassemble a binary _without running it_ — read the opcodes as text | Auditing a binary before executing it (e.g. static inspection)    |
| **Dynamic analysis** | Attach a **debugger** to a _running_ process and observe it live     | Understanding behavior that only appears at runtime               |
| **Decompiling**      | Attempt to reconstruct high-level source from disassembly            | Best-effort only — decompilers guess at structure and can mislead |

> Disassembly always reflects what's actually executing; decompiling does not — it's a
> reconstruction, not a fact.

**Debugger fundamentals** (same core loop regardless of tool — gdb, LLDB, x64dbg,
WinDbg):

- **Attach** — connect the debugger to a running process (or launch the process under
  it).
- **Breakpoint** — mark an address so execution pauses when the CPU reaches it.
  Breakpoints can be **code breakpoints** (pause on an instruction) or **memory/data
  breakpoints** (pause when a memory address is read or written).
- **Step** — advance execution one instruction (or line) at a time once paused.
  - **Step into** a `call` — follow execution into the called function.
  - **Step over** a `call` — run the whole function without descending into it, landing
    on the next instruction after it returns.
  - **Execute until return** — run until the current function's `ret`, useful for
    "bubbling up" from a low-level instruction to the higher-level function that led
    there.
- **Inspect** — read registers, the call stack, and memory (typically shown as hex +
  ASCII) while paused.

**Reading assembly you didn't write:** the general strategy is to establish _context_
first — find a landmark (a known string, a known memory value, a function you already
understand) — then work outward from it one instruction at a time. `mov`, `add`, `sub`,
`cmp`, and `jmp`/`je`/`jne` (conditional branches) cover the majority of what you'll
need to trace through simple control flow; not every instruction needs to be understood
to follow the logic.

**Why this connects back to compilers:** a debugger is doing, by hand, exactly what your
[interpreter's `eval` loop](#the-interpreter--recursion) and the
[CPU's fetch-decode-execute cycle](#the-cpu) do automatically — stepping through
instructions one at a time and tracking state. Understanding one deepens the others.

---

### Following Unfamiliar Call Chains

A function rarely acts alone — it calls other functions, which call others still.
Visualizing this as a **function chain** makes a deep call stack easier to hold in your
head:

```
handle_input() → validate() → normalize_data() → apply_rule() → write_result()
```

When stepping through a debugger, working from a low-level instruction _back up_ through
a chain like this — using **execute-until-return** repeatedly — is sometimes called
**bubbling up**: you keep surfacing to the next caller until you reach the higher-level
function that actually matters to you. It's the same recursive-structure idea as
[the call stack](#the-call-stack), just observed from the outside instead of written
from the inside.

A general four-step approach for making sense of _any_ unfamiliar system's behavior — a
program, an API, a bug report — mirrors this:

1. **Identify** what you're trying to understand or verify.
2. **Understand** what data or code path is actually involved.
3. **Locate** it — find the concrete function, variable, or address.
4. **Observe/verify** — confirm your understanding matches what actually happens.

---

### Pointers & Shared Libraries

Two more building blocks that show up constantly once you're reading real systems code:

- **Pointers** — a variable that stores _the address of_ another value rather than the
  value itself. `int *y = &x;` makes `y` point at `x`; dereferencing (`*y`) reads or
  writes through that address. Pointers are how languages like C/C++/Rust let code
  directly reference and manipulate memory rather than only working with copies.
- **Shared/dynamic libraries** (`.dll` on Windows, `.so` on Linux, `.dylib` on macOS) —
  compiled code that's loaded into a program _at runtime_ rather than baked into the
  executable at compile time, so multiple programs can share one copy of common
  functionality (e.g. a UI toolkit) without each program bundling its own copy. This is
  the runtime cousin of the [`Assembler`/linker relationship](#assembly-language) — a
  linker resolves _static_ dependencies at build time, while a loader resolves _dynamic_
  ones each time the program starts.

---

## 🧠 Low-Level Memory, Binary Analysis & Language Runtime Architecture

Understanding memory is useful in three directions at once:

1. **Building a language** — you must decide how values, objects, strings, stack frames,
   and garbage or ownership metadata are represented.
2. **Debugging low-level software** — you must connect a source-level value to bytes,
   addresses, registers, and instructions.
3. **Reversing a language or binary format** — you infer those same representation
   decisions from observable behavior.

Use these techniques only on software, memory dumps, packet captures, and systems you
own or are explicitly authorized to study. Prefer an isolated VM, disposable test
program, local server, or purpose-built target.

> ⚠️ **Safety and authorization boundary:** low-level visibility is powerful. Practice
> on software and systems you own or have explicit permission to inspect.

---

### 1. Bytes, Addresses, Pointers, and Views

A **byte** is data. An **address** is a number naming a location. A **pointer** is a
typed or untyped value containing an address. The same bytes may be interpreted through
different views:

```text
bytes:      48 65 6c 6c 6f 00
ASCII:       H  e  l  l  o \0
u16 (LE):  0x6548 0x6c6c 0x006f
```

The bytes do not change; the interpretation does:

| Context                  | Interpretation applied to the same bytes         |
| ------------------------ | ------------------------------------------------ |
| Disassembler             | Opcodes, operands, and instruction boundaries    |
| Executable/packet parser | Header fields, lengths, flags, and sections      |
| Virtual machine          | Tagged values, handles, or bytecode instructions |
| Debugger                 | Runtime objects, fields, pointers, and strings   |
| File-format reversing    | Hypothesized records and relationships           |

#### Endianness

For the integer `0x12345678`:

```text
big-endian:    12 34 56 78
little-endian: 78 56 34 12
```

Network protocols traditionally call big-endian **network byte order**. x86 and x86-64
normally use little-endian values in memory. Never decode a multi-byte integer without
deciding its width and byte order.

```rust
fn read_u32_be(input: &[u8]) -> Option<u32> {
    let bytes: [u8; 4] = input.get(..4)?.try_into().ok()?;
    Some(u32::from_be_bytes(bytes))
}

fn read_u32_le(input: &[u8]) -> Option<u32> {
    let bytes: [u8; 4] = input.get(..4)?.try_into().ok()?;
    Some(u32::from_le_bytes(bytes))
}
```

#### Pointer arithmetic is really layout arithmetic

If an array element has size `S`, element `i` begins at:

```text
element_address = base_address + i * S
```

For a record or object:

```text
field_address = object_address + field_offset
```

A multi-level pointer path repeatedly reads an address and then applies the next offset:

```text
root -> object -> component -> field
```

Do not confuse a stable _relationship_ with a stable absolute address. ASLR, allocator
behavior, and different runs can move stack, heap, libraries, and sometimes the main
executable.

---

### 2. Memory Regions and Object Lifetimes

| Region               | Typical contents                       | Lifetime            |
| -------------------- | -------------------------------------- | ------------------- |
| Code / `.text`       | Machine instructions                   | Loaded image        |
| Read-only data       | Constants, string literals             | Loaded image        |
| Static data          | Globals                                | Process             |
| Stack                | Call frames, local values, saved state | Function or scope   |
| Heap                 | Dynamic objects, collections, closures | Explicit or managed |
| Memory-mapped region | Files, shared libraries, device memory | Mapping             |

This table is a model, not a promise that every compiler and OS lays out every value
exactly this way. Optimizers may keep a local variable only in a register or remove it
entirely.

#### Stack frame mental model

```text
higher addresses
+---------------------+
| caller's frame      |
+---------------------+
| return address      |
| saved registers     |
| local variables     |
| temporary/spill data|
+---------------------+  <- stack pointer
lower addresses
```

Calling conventions define where arguments and return values go, which registers a
callee must preserve, and how the stack is aligned. A compiler backend must obey the
target ABI when calling external functions.

#### Heap lifetime hazards

| Hazard                   | What went wrong                                           |
| ------------------------ | --------------------------------------------------------- |
| **Leak**                 | Unreachable memory is never released                      |
| **Double free**          | One allocation is released twice                          |
| **Use after free**       | A stale pointer outlives its allocation                   |
| **Out-of-bounds access** | An index or pointer escapes the allocation                |
| **Uninitialized read**   | Bytes are interpreted before receiving a valid value      |
| **Data race**            | Shared state is accessed without required synchronization |

These hazards explain the trade-offs among manual allocation, garbage collection,
reference counting, region/arena allocation, and Rust-style ownership.

#### Arena allocation for compilers

AST nodes often share one phase lifetime: they are all created during parsing and
discarded together. An arena makes that lifetime explicit:

```text
source text
   ↓
[AST arena: many small nodes]
   ↓ type checking / lowering
[IR arena: many small nodes]
   ↓
discard whole phase at once
```

This can simplify ownership and improve locality, but references into an arena must
never outlive it.

---

### 3. Layout, Alignment, Padding, and `repr`

CPUs often prefer or require values at aligned addresses. A `u32` commonly has alignment
4, so a compiler may insert padding:

```rust
#[repr(C)]
struct Header {
    tag: u8,      // offset 0
                  // offsets 1..3 may be padding
    length: u32,  // offset 4
}

fn main() {
    assert_eq!(std::mem::align_of::<Header>(), 4);
    assert_eq!(std::mem::size_of::<Header>(), 8);
}
```

Key Rust rules:

- Default `repr(Rust)` does not promise a stable C-compatible field layout.
- `#[repr(C)]` requests C ABI layout and is useful at an FFI boundary.
- `#[repr(u8)]` or another integer representation can make an enum discriminant
  representation explicit.
- `#[repr(packed)]` removes some padding but can create unaligned fields. Creating a
  normal reference to an unaligned field is invalid; specialized unaligned reads are
  required.
- A Rust struct is not automatically a serialized file or wire format. Serialize each
  field explicitly.

For your own language, decide whether object layout is:

- **specified** as part of the language/ABI;
- **implementation-defined** but documented by one compiler;
- **opaque**, accessible only through generated accessors.

Opaque layouts give the runtime freedom to reorder fields or change garbage collectors.
Specified layouts make FFI and reversing easier but reduce implementation freedom.

---

### 4. Safe Binary Parsing in Rust

A binary parser should treat all offsets, sizes, tags, and counts as untrusted. Prefer
slices and checked arithmetic over raw pointers.

```rust
#[derive(Debug, PartialEq)]
struct Packet<'a> {
    kind: u8,
    payload: &'a [u8],
}

#[derive(Debug, PartialEq)]
enum ParseError {
    Truncated,
    TooLarge,
    TrailingBytes,
}

fn take<'a>(input: &mut &'a [u8], n: usize) -> Result<&'a [u8], ParseError> {
    let (head, tail) = input.split_at_checked(n).ok_or(ParseError::Truncated)?;
    *input = tail;
    Ok(head)
}

fn parse_packet(mut input: &[u8]) -> Result<Packet<'_>, ParseError> {
    let kind = take(&mut input, 1)?[0];
    let length = u16::from_be_bytes(
        take(&mut input, 2)?
            .try_into()
            .map_err(|_| ParseError::Truncated)?,
    ) as usize;

    const MAX_PAYLOAD: usize = 4096;
    if length > MAX_PAYLOAD {
        return Err(ParseError::TooLarge);
    }

    let payload = take(&mut input, length)?;
    if !input.is_empty() {
        return Err(ParseError::TrailingBytes);
    }

    Ok(Packet { kind, payload })
}

#[test]
fn parses_a_small_packet() {
    let bytes = [0x02, 0x00, 0x03, b'c', b'a', b't'];
    assert_eq!(
        parse_packet(&bytes),
        Ok(Packet { kind: 2, payload: b"cat" })
    );
}

#[test]
fn rejects_truncation() {
    assert_eq!(parse_packet(&[2, 0, 5, b'x']), Err(ParseError::Truncated));
}
```

| Parser concern     | Defensive requirement                                     |
| ------------------ | --------------------------------------------------------- |
| Lengths/counts     | Bound them before allocating or looping                   |
| Arithmetic         | Use checked addition, multiplication, and slicing         |
| Tags/enums         | Reject unknown mandatory tags and invalid values          |
| Trailing bytes     | Define whether they are permitted                         |
| Structural limits  | Cap recursion depth and decompressed output               |
| Input delivery     | Never assume one read contains a complete file or message |
| Diagnostics        | Preserve byte offsets in errors                           |
| Robustness testing | Fuzz with arbitrary byte strings                          |

The same parser architecture works for bytecode, object files, save files, image
formats, and network frames.

---

### 5. Memory Inspection as a Scientific Method

most general-purpose lesson is not a particular tool; it is the repeated workflow:

1. **Identify** a value or behavior you can control.
2. **Change one variable** while keeping other conditions stable.
3. **Observe** memory, registers, instructions, or packets.
4. **Form a hypothesis** about representation or control flow.
5. **Repeat** with a different input or a fresh run.
6. **Validate** by predicting an observation you have not yet seen.

For a safe toy example, pretend a process snapshot is just a byte vector:

```rust
fn find_u32_le(haystack: &[u8], wanted: u32) -> Vec<usize> {
    let needle = wanted.to_le_bytes();
    haystack
        .windows(needle.len())
        .enumerate()
        .filter_map(|(offset, window)| (window == needle).then_some(offset))
        .collect()
}

fn retain_changed(old: &[u8], new: &[u8], candidates: &mut Vec<usize>) {
    candidates.retain(|&i| {
        old.get(i..i + 4)
            .zip(new.get(i..i + 4))
            .is_some_and(|(a, b)| a != b)
    });
}
```

This demonstrates differential scanning without touching another process. The same
reasoning is useful for debugging your VM's heap, comparing serialized ASTs, and
locating a field in a captured memory image.

#### Static, dynamic, and forensic views

| View     | Evidence                                     | Best for                           |
| -------- | -------------------------------------------- | ---------------------------------- |
| Static   | File bytes, symbols, strings, disassembly    | Possible behavior and structure    |
| Dynamic  | Registers, memory, breakpoints, system calls | Actual behavior for one run        |
| Forensic | A saved memory image or capture              | Reconstructing prior runtime state |

Memory-forensics workflow can be generalized as:

1. preserve the original image;
2. identify the OS, architecture, and capture context;
3. inventory processes, mappings, handles, and connections;
4. narrow to relevant artifacts;
5. corroborate findings across multiple sources;
6. record offsets, hashes, and tool versions so results are reproducible.

Strings are clues, not proof. A string may be unused, encoded, copied, stale, or
unrelated.

---

### 6. Defensive Memory Safety and Hardening

most useful when it helps you recognize what your compiler or runtime must prevent.

| Failure          | Root cause                           | Language/runtime defense           |
| ---------------- | ------------------------------------ | ---------------------------------- |
| Buffer overflow  | Write exceeds allocation             | Slice bounds, checked APIs         |
| Use after free   | Lifetime not enforced                | Ownership, GC, handles/generations |
| Double free      | Ambiguous ownership                  | Move semantics, unique owner       |
| Null dereference | Missing value represented as pointer | `Option<T>` / tagged union         |
| Type confusion   | Bytes interpreted as wrong type      | Tags, validation, safe casts       |
| Integer overflow | Size calculation wraps               | Checked arithmetic and limits      |
| Data race        | Unsynchronized shared mutation       | Send/sync rules, actors, locks     |

| Platform mitigation        | What it hardens                                       |
| -------------------------- | ----------------------------------------------------- |
| **Stack canaries**         | Detect some stack overwrites before return            |
| **NX/DEP**                 | Mark data pages non-executable                        |
| **ASLR**                   | Randomize important memory regions between runs       |
| **PIE**                    | Allow the main executable to be relocated             |
| **RELRO**                  | Make selected relocation data read-only after linking |
| **Control-flow integrity** | Restrict indirect branches to approved targets        |

Mitigations are layers, not substitutes for memory-safe design.

Safe C example: bound the read to the actual destination capacity and ensure string
termination.

```c
#include <stdio.h>

int main(void) {
    char name[64];
    if (fgets(name, sizeof name, stdin) == NULL) {
        return 1;
    }
    printf("Hello, %s", name);
    return 0;
}
```

| Testing layer     | Recommended practice                                  |
| ----------------- | ----------------------------------------------------- |
| Compiler warnings | Enable them and treat new warnings seriously          |
| Native runtime    | Use sanitizers on C/C++ components                    |
| Input boundaries  | Fuzz lexers, parsers, verifiers, and decoders         |
| Numeric limits    | Test zero, maximum, and one-past-maximum values       |
| Bytecode          | Verify malformed input before execution               |
| Unsafe Rust       | Keep it small and document the relied-upon invariants |

---

### 7. Reversing a Binary, Bytecode VM, or Language

Source compilation and reverse engineering travel in opposite directions:

```text
source → tokens → AST → typed IR → machine IR → assembly → bytes
bytes  → instructions → control-flow graph → data-flow facts → hypotheses
```

| Lost information        | Reverse-engineering consequence                          |
| ----------------------- | -------------------------------------------------------- |
| Variable/function names | Recovered names are hypotheses or tool-generated labels  |
| Comments/formatting     | Intent and original organization cannot be reconstructed |
| Source distinctions     | Several source forms may produce identical instructions  |
| Call boundaries         | Inlining can merge a callee into its caller              |
| Expressions             | Constant folding can erase the original computation      |
| Source types            | Only layout and operation clues may remain               |

Therefore, decompilation is a **model**, not restoration of the original source.

#### A disciplined reversing notebook

For each discovery, record:

| Field       | Example                                             |
| ----------- | --------------------------------------------------- |
| Location    | file offset, virtual address, function symbol       |
| Observation | "four-byte big-endian value precedes payload"       |
| Hypothesis  | "value is compressed payload length"                |
| Evidence    | controlled inputs of lengths 1, 5, and 20           |
| Confidence  | low / medium / high                                 |
| Falsifier   | a capture whose value does not match payload length |

#### Recovering a bytecode format

Start with the smallest programs:

```text
0
1
1 + 2
print(1)
if true { print(1) }
```

Then compare outputs:

1. locate a header or magic number;
2. identify version, flags, and section boundaries;
3. find constant pools and encoded strings;
4. change one literal and locate the changed bytes;
5. infer opcode width and operand encoding;
6. trace stack-height changes or virtual registers;
7. build a disassembler before building a decompiler;
8. validate by round-tripping or predicting unseen encodings.

Example of a tiny, safe disassembler:

```rust
#[derive(Debug, PartialEq)]
enum Instr {
    Push(u8),
    Add,
    Print,
    Halt,
}

fn disassemble(mut bytes: &[u8]) -> Result<Vec<Instr>, &'static str> {
    let mut out = Vec::new();
    while let Some((&opcode, rest)) = bytes.split_first() {
        bytes = rest;
        match opcode {
            0x01 => {
                let (&value, rest) = bytes.split_first().ok_or("missing operand")?;
                bytes = rest;
                out.push(Instr::Push(value));
            }
            0x02 => out.push(Instr::Add),
            0x03 => out.push(Instr::Print),
            0xff => {
                out.push(Instr::Halt);
                if !bytes.is_empty() {
                    return Err("bytes after halt");
                }
                return Ok(out);
            }
            _ => return Err("unknown opcode"),
        }
    }
    Err("missing halt")
}
```

#### Designing for reversibility

| Reversibility feature           | Benefit                                         |
| ------------------------------- | ----------------------------------------------- |
| Magic number + bytecode version | Reliable format identification                  |
| Documented section table        | Predictable navigation and extension            |
| Source maps/symbol names        | Better diagnostics and debugging                |
| Official dumper/disassembler    | One canonical inspection path                   |
| Separate verifier               | Safe validation without execution               |
| Fail-closed versioning          | Unknown formats cannot be misinterpreted        |
| Deterministic output            | Reproducible builds and meaningful binary diffs |

---

## 🦀 Idiomatic Rust for Compilers, Parsers, and Low-Level Tools

The notes in this section draw on
[Idiomatic Rust Snippets](https://idiomatic-rust-snippets.org/), the
[Rust Reference](https://doc.rust-lang.org/reference/), and the Rust patterns catalogue.
The goal is not merely "code that compiles," but APIs whose types explain ownership,
state, and failure.

### 1. Ownership as a Runtime Design Lesson

Rust's three basic ownership rules are a useful model for any language:

1. each value has an owner;
2. moving a non-`Copy` value transfers ownership;
3. the value is dropped when its owner leaves scope.

```rust
fn token_count(source: &str) -> usize {
    source.split_whitespace().count()
}

fn keep_source(source: String) -> Module {
    Module { source }
}

struct Module {
    source: String,
}
```

Borrow when a function only needs temporary access. Take ownership when it must store,
consume, transfer, or transform the value.

For parser APIs:

- use `&str` instead of `&String`;
- use `&[Token]` instead of `&Vec<Token>`;
- use `String` or `Vec<T>` when the result owns its data;
- use `Cow<'a, str>` when a transformation may return either borrowed or owned text.

```rust
use std::borrow::Cow;

fn unescape_identifier(input: &str) -> Cow<'_, str> {
    if input.contains("\\-") {
        Cow::Owned(input.replace("\\-", "-"))
    } else {
        Cow::Borrowed(input)
    }
}
```

### 2. Make Illegal States Unrepresentable

Do not represent every compiler stage with one giant struct full of optional fields.

```rust
struct ParsedModule {
    ast: Ast,
}

struct TypedModule {
    ast: TypedAst,
    symbols: SymbolTable,
}

struct LoweredModule {
    ir: Ir,
}

fn typecheck(parsed: ParsedModule) -> Result<TypedModule, TypeErrors> {
    // Every successful TypedModule is guaranteed to have types and symbols.
    todo!()
}

fn lower(typed: TypedModule) -> LoweredModule {
    todo!()
}
```

The type-state version makes it impossible to call `lower` on an untyped AST without
first going through `typecheck`.

Use enums for closed sets of states or instructions:

```rust
enum Value {
    Int(i64),
    Bool(bool),
    String(String),
    Function(FunctionId),
    Nil,
}

fn truthy(value: &Value) -> bool {
    match value {
        Value::Bool(v) => *v,
        Value::Nil => false,
        Value::Int(_) | Value::String(_) | Value::Function(_) => true,
    }
}
```

Avoid a wildcard arm when adding a new variant should force every semantic decision to
be revisited.

### 3. Newtypes for Offsets, IDs, and Units

Raw `usize` values are easy to mix up:

```rust
#[derive(Clone, Copy, Debug, PartialEq, Eq, Hash)]
struct ByteOffset(usize);

#[derive(Clone, Copy, Debug, PartialEq, Eq, Hash)]
struct Register(u16);

#[derive(Clone, Copy, Debug, PartialEq, Eq, Hash)]
struct FunctionId(u32);

fn report_error(at: ByteOffset, message: &str) {
    eprintln!("byte {}: {message}", at.0);
}
```

This prevents accidentally passing a register number where a file offset is required.
The wrapper normally has no runtime cost.

| Values that look alike           | Why they need separate types               |
| -------------------------------- | ------------------------------------------ |
| Byte offset / virtual address    | Different coordinate systems               |
| Source line / source byte        | Different location units                   |
| Token ID / AST node ID           | Different entity domains                   |
| VM register / local slot         | Different storage namespaces               |
| Host-order / network-order value | Different byte encodings                   |
| Compressed / uncompressed size   | Different allocation and validation limits |

### 4. Errors Are Part of the Language Tool's Interface

Use `Option<T>` for expected absence and `Result<T, E>` for failure with a reason.
Reserve panics for broken internal invariants.

```rust
#[derive(Debug)]
enum DecodeError {
    UnexpectedEof { at: ByteOffset },
    UnknownOpcode { at: ByteOffset, opcode: u8 },
    InvalidJump { from: ByteOffset, to: ByteOffset },
}

fn opcode_name(byte: u8) -> Option<&'static str> {
    match byte {
        0x01 => Some("push"),
        0x02 => Some("add"),
        0xff => Some("halt"),
        _ => None,
    }
}

fn require_opcode(byte: u8, at: ByteOffset) -> Result<&'static str, DecodeError> {
    opcode_name(byte).ok_or(DecodeError::UnknownOpcode { at, opcode: byte })
}
```

| Diagnostic field | Question it answers                  |
| ---------------- | ------------------------------------ |
| Problem          | **What** went wrong?                 |
| Location         | **Where** did it happen?             |
| Expectation      | What was valid or expected here?     |
| Observation      | What did the compiler actually find? |
| Recovery         | What can the user do next?           |

### 5. Iterators Express Token and Byte Pipelines

Iterators are lazy and make transformations explicit:

```rust
fn decimal_literals(tokens: &[Token]) -> impl Iterator<Item = i64> + '_ {
    tokens.iter().filter_map(|token| match token {
        Token::Integer(text) => text.parse().ok(),
        _ => None,
    })
}

enum Token {
    Integer(String),
    Identifier(String),
    Plus,
}
```

Prefer iterator adapters when they clarify the pipeline. Use a normal loop when control
flow, error recovery, or mutable parser state is easier to read that way. "More
functional" is not automatically "more idiomatic."

### 6. RAII for Files, Sockets, Locks, and Temporary State

RAII means a resource is released when its guard or owner is dropped. Rust's `File`,
`TcpStream`, mutex guards, and collections already follow it.

A compiler can use a guard to restore temporary state even when a function returns
early:

```rust
use std::cell::Cell;

struct DepthGuard<'a>(&'a Cell<usize>);

impl Drop for DepthGuard<'_> {
    fn drop(&mut self) {
        self.0.set(self.0.get() - 1);
    }
}

fn enter_depth(depth: &Cell<usize>, limit: usize) -> Result<DepthGuard<'_>, &'static str> {
    if depth.get() >= limit {
        return Err("nesting limit exceeded");
    }
    depth.set(depth.get() + 1);
    Ok(DepthGuard(depth))
}
```

RAII is also a language-design option: deterministic destructors are convenient, but
their exact timing affects semantics, optimization, and cycles.

### 7. Keep `unsafe` Small and Give It a Contract

`unsafe` does not disable the borrow checker; it permits a small set of operations whose
safety the compiler cannot prove. Every `unsafe` block should have a written invariant.

```rust
fn read_unaligned_u32_le(bytes: &[u8]) -> Option<u32> {
    // A safe implementation is preferable here.
    let array: [u8; 4] = bytes.get(..4)?.try_into().ok()?;
    Some(u32::from_le_bytes(array))
}
```

Only reach for raw pointers or `read_unaligned` when a safe byte-slice solution cannot
meet the requirement. `#[repr(packed)]` is not a shortcut for parsing arbitrary bytes
into a struct.

| Unsafe concern | Review question                                       |
| -------------- | ----------------------------------------------------- |
| Ownership      | Who allocated this memory and who releases it?        |
| Lifetime       | How long is the allocation valid?                     |
| Initialization | Is every byte valid for the interpreted type?         |
| Alignment      | Does the address meet the type's requirement?         |
| Aliasing       | Are simultaneous references permitted?                |
| Concurrency    | Can another thread or interrupt mutate it?            |
| Bounds         | How are the pointer and length proven to be in range? |

### 8. Rust Anti-Patterns to Avoid

| Anti-pattern                         | Better direction                                      |
| ------------------------------------ | ----------------------------------------------------- |
| Clone to silence a borrow error      | Model ownership and borrowing deliberately            |
| Accept `&String` or `&Vec<T>`        | Accept `&str` or `&[T]` when ownership is unnecessary |
| Use `usize` for every numeric domain | Introduce newtypes for IDs, offsets, sizes, and slots |
| `unwrap()` input-dependent results   | Propagate a contextual `Result`                       |
| Index before checking length         | Use checked slicing or `get`                          |
| Return only `"failed"`               | Include error kind and byte/source offset             |
| Hold a lock across blocking/`.await` | Shorten the guard's scope                             |
| Make every AST field `pub`           | Expose a small, stable API                            |
| Cast bytes directly to a struct      | Parse layout, alignment, and endianness explicitly    |
| Recurse over attacker input          | Enforce a depth limit or use an explicit stack        |

### 9. Rust and Python: Similar Task, Different Contract

The
[Programming Idioms Rust/Python comparison](https://programming-idioms.org/cheatsheet/Rust/Python)
is useful because it holds the **task** constant while changing the language. Do not
translate syntax alone: ownership, errors, integers, strings, iteration, and concurrency
have different contracts.

| Concern   | Python tendency                       | Rust tendency                                    |
| --------- | ------------------------------------- | ------------------------------------------------ |
| typing    | values checked dynamically            | types checked before execution                   |
| ownership | garbage-collected object references   | ownership, borrowing, and deterministic drop     |
| errors    | exceptions                            | `Result<T, E>` and `?`                           |
| absence   | `None`, often checked dynamically     | `Option<T>` forces explicit handling             |
| integers  | arbitrary precision by default        | fixed-width types; explicit big integers         |
| strings   | Unicode string abstraction            | UTF-8 `String`/`str`; indexing by byte forbidden |
| iteration | iterable/generator protocol           | zero-cost `Iterator` adaptors                    |
| maps      | insertion-ordered `dict`              | choose `HashMap` or ordered `BTreeMap`           |
| mutation  | object mutability is runtime behavior | binding/reference mutability is explicit         |

#### Iteration with Index and Value

```python
for index, value in enumerate(items):
    print(index, value)
```

```rust
for (index, value) in items.iter().enumerate() {
    println!("{index}: {value}");
}
```

`iter()` borrows values, `iter_mut()` allows mutation, and `into_iter()` consumes the
collection. That choice is part of the API, not incidental syntax.

#### Parse, Filter, and Collect Errors

Python commonly raises on the first invalid conversion:

```python
values = [int(text) for text in fields if text.strip()]
```

Rust can make the same short-circuit contract explicit:

```rust
fn parse_nonempty(fields: &[String]) -> Result<Vec<i64>, std::num::ParseIntError> {
    fields
        .iter()
        .filter(|text| !text.trim().is_empty())
        .map(|text| text.parse::<i64>())
        .collect()
}
```

Because `Result` implements `FromIterator`, collecting stops on the first error. If all
errors matter—such as compiler diagnostics—collect successes and diagnostics into
separate typed outputs instead.

#### Unicode Requires a Unit

```python
prefix = text[:5]
```

```rust
fn first_five_scalars(text: &str) -> String {
    text.chars().take(5).collect()
}
```

| Unit             | Rust view/API                | Suitable for                        |
| ---------------- | ---------------------------- | ----------------------------------- |
| byte             | `as_bytes()`, byte ranges    | protocols, files, UTF-8 validation  |
| Unicode scalar   | `chars()`                    | code points and many language rules |
| grapheme cluster | Unicode-segmentation library | user-perceived characters           |

Five bytes, five Unicode scalar values, and five visible characters are not equivalent.

#### Maps Encode Ordering Expectations

```rust
use std::collections::{BTreeMap, HashMap};

let fast_lookup: HashMap<&str, i32> = [("parse", 1), ("typecheck", 2)]
    .into_iter()
    .collect();

let deterministic_output: BTreeMap<&str, i32> =
    fast_lookup.into_iter().collect();
```

Use a deterministic map or sort keys before emitting compiler artifacts, diagnostics,
snapshots, or protocol output that must be reproducible.

> 🧠 **Translation habit:** ask “what behavior does this idiom promise?” before asking
> “what Rust syntax looks like the Python syntax?”

---

## 🧱 Practical Rust Patterns & Cookbook Recipes

[Rust Design Patterns](https://rust-unofficial.github.io/patterns/) separates reusable
Rust knowledge into **idioms**, **design patterns**, and **anti-patterns**. The
[Rust Cookbook](https://rust-lang-nursery.github.io/rust-cookbook/intro.html) complements
that vocabulary with small recipes for common tasks.

> 🧠 **Selection rule:** use a pattern because its trade-off matches the problem, not
> because the pattern has a familiar name.

### 1. Idiom, Pattern, or Recipe?

| Kind               | Purpose                                               | Example                      |
| ------------------ | ----------------------------------------------------- | ---------------------------- |
| **Idiom**          | Conventional Rust expression of a common idea         | `?`, iterators, `Default`    |
| **Design pattern** | Reusable structure for a recurring design problem     | Builder, Command, RAII guard |
| **Anti-pattern**   | Tempting approach whose costs usually exceed benefits | Clone-to-fix-borrowing       |
| **Recipe**         | Focused example that accomplishes one practical task  | Stream a file line by line   |

Patterns are not automatically abstractions worth keeping. A direct struct constructor
is clearer than a builder with no optional choices; an enum can be clearer than a
trait-object hierarchy with only two closed variants.

### 2. Pattern Selection Table

| Problem shape                                     | Rust-oriented pattern               |
| ------------------------------------------------- | ----------------------------------- |
| Several values share one primitive representation | **Newtype**                         |
| Construction has many optional settings           | **Builder**                         |
| Cleanup must happen on every exit path            | **RAII guard**                      |
| Actions must be queued, logged, or undone         | **Command**                         |
| Behavior varies behind one stable interface       | Trait/generic **Strategy**          |
| Legal operations depend on current state          | **Typestate** or an exhaustive enum |
| A closed set of variants has different data       | Enum + `match`                      |
| Heterogeneous extensions are loaded dynamically   | Trait objects                       |

### 3. Newtypes Encode Meaning

Newtypes prevent values with the same representation from being mixed:

```rust
#[derive(Clone, Copy, Debug, Eq, PartialEq)]
struct ByteOffset(usize);

#[derive(Clone, Copy, Debug, Eq, PartialEq)]
struct VirtualAddress(u64);

#[derive(Clone, Copy, Debug, Eq, PartialEq)]
struct RequestId(u64);

fn seek_to(offset: ByteOffset) {
    println!("seek to byte {}", offset.0);
}
```

| Newtype benefit    | Result                                                |
| ------------------ | ----------------------------------------------------- |
| Domain separation  | Offsets, addresses, counts, and IDs cannot be swapped |
| Central validation | Constructors can enforce range or formatting rules    |
| API clarity        | Signatures explain what a number means                |
| Future evolution   | Representation can change behind the public type      |

Implement only operations that make semantic sense. Adding two byte lengths may be
valid; adding two virtual addresses usually is not.

### 4. Builder for Complicated Construction

Rust has no overloaded functions or default arguments. A builder is useful when a
configuration has required fields plus several optional policies:

```rust
#[derive(Debug)]
struct CompilerOptions {
    target: String,
    optimize: bool,
    warnings_as_errors: bool,
    max_errors: usize,
}

#[derive(Debug)]
struct CompilerOptionsBuilder {
    target: String,
    optimize: bool,
    warnings_as_errors: bool,
    max_errors: usize,
}

impl CompilerOptions {
    fn builder(target: impl Into<String>) -> CompilerOptionsBuilder {
        CompilerOptionsBuilder {
            target: target.into(),
            optimize: false,
            warnings_as_errors: false,
            max_errors: 20,
        }
    }
}

impl CompilerOptionsBuilder {
    fn optimize(mut self, enabled: bool) -> Self {
        self.optimize = enabled;
        self
    }

    fn warnings_as_errors(mut self, enabled: bool) -> Self {
        self.warnings_as_errors = enabled;
        self
    }

    fn max_errors(mut self, limit: usize) -> Self {
        self.max_errors = limit.max(1);
        self
    }

    fn build(self) -> CompilerOptions {
        CompilerOptions {
            target: self.target,
            optimize: self.optimize,
            warnings_as_errors: self.warnings_as_errors,
            max_errors: self.max_errors,
        }
    }
}
```

```rust
let options = CompilerOptions::builder("wasm32-unknown-unknown")
    .optimize(true)
    .warnings_as_errors(true)
    .max_errors(50)
    .build();
```

| Use a builder when...                               | Prefer a constructor when...               |
| --------------------------------------------------- | ------------------------------------------ |
| Many settings are optional                          | Every argument is required                 |
| Defaults should remain centralized                  | There are only a few obvious fields        |
| Construction needs validation or staged preparation | A struct literal is already clear          |
| Adding an option should not break callers           | The type is private and construction local |

### 5. RAII Guards Make Cleanup Structural

An RAII guard acquires or changes state during construction and restores/finalizes it in
`Drop`. Cleanup then occurs on normal return, early `?`, and panic unwinding.

```rust
use std::cell::Cell;

struct RecursionGuard<'a> {
    depth: &'a Cell<usize>,
}

impl Drop for RecursionGuard<'_> {
    fn drop(&mut self) {
        self.depth.set(self.depth.get() - 1);
    }
}

fn enter_recursion(
    depth: &Cell<usize>,
    limit: usize,
) -> Result<RecursionGuard<'_>, &'static str> {
    if depth.get() >= limit {
        return Err("recursion limit exceeded");
    }

    depth.set(depth.get() + 1);
    Ok(RecursionGuard { depth })
}
```

The guard should mediate access to the resource when using the resource after
finalization would be invalid. Common examples are mutex guards, temporary directory
owners, transaction guards, scoped tracing spans, and VM root guards.

> ⚠️ **Drop is not a general error channel:** destructors cannot conveniently return
> cleanup errors. Provide an explicit `finish`, `commit`, or `flush` operation when the
> caller must observe failure.

### 6. Command Pattern for Compiler Passes

The Command pattern turns an action into a value that can be stored and invoked later.
Trait objects suit an open, heterogeneous pass pipeline:

```rust
#[derive(Debug)]
struct Module {
    instructions: Vec<String>,
}

trait Pass {
    fn name(&self) -> &'static str;
    fn run(&self, module: &mut Module) -> Result<(), String>;
}

struct PassPipeline {
    passes: Vec<Box<dyn Pass>>,
}

impl PassPipeline {
    fn run(&self, module: &mut Module) -> Result<(), String> {
        for pass in &self.passes {
            pass.run(module)
                .map_err(|error| format!("{}: {error}", pass.name()))?;
        }
        Ok(())
    }
}
```

| Representation         | Best fit                                        |
| ---------------------- | ----------------------------------------------- |
| Enum of commands       | Closed command set; exhaustive dispatch         |
| `fn` pointers/closures | Small stateless actions                         |
| `Box<dyn Command>`     | Open plugin-like set with per-command state     |
| Generic command type   | One command type on a performance-critical path |

Undo requires more than an `undo()` method name. The command must retain enough prior
state, and failure halfway through a multi-command transaction needs an explicit
rollback policy.

### 7. Strategy: Static or Dynamic Dispatch

```rust
trait DiagnosticSink {
    fn emit(&mut self, message: &str);
}

fn type_check<S: DiagnosticSink>(source: &str, sink: &mut S) {
    if source.is_empty() {
        sink.emit("source is empty");
    }
}
```

| Dispatch form      | Benefit                                | Cost/constraint                        |
| ------------------ | -------------------------------------- | -------------------------------------- |
| Generic `S: Trait` | Inlining and static type knowledge     | One instantiation per concrete type    |
| `&mut dyn Trait`   | Runtime-pluggable heterogeneous values | Indirect call and object-safety limits |
| Enum + `match`     | Exhaustive, compact closed set         | Adding a variant changes central code  |

Prefer generics at reusable performance-sensitive boundaries and trait objects where
runtime extensibility is the actual requirement.

### 8. Cookbook Mindset

The Rust Cookbook covers practical categories such as algorithms, command-line tools,
encoding, filesystems, concurrency, networking, and asynchronous work.

| Recipe category | Transferable lesson                                |
| --------------- | -------------------------------------------------- |
| Command line    | Preserve non-UTF-8 paths with `args_os`            |
| File I/O        | Stream large input and buffer many small writes    |
| Encoding        | Make byte order and text format explicit           |
| Concurrency     | Select ownership transfer before shared mutation   |
| Networking      | Bound reads, writes, timeouts, and decoded sizes   |
| Compression     | Treat expanded size and archive paths as untrusted |
| Async           | Bound task counts, channels, and wait time         |

Cookbook snippets are starting points. Before adopting a recipe, check current crate
documentation, pin an intentional dependency range, test errors, and add limits suitable
for the input's trust level.

### 9. Stream Large Text Instead of Loading It Whole

```rust
use std::fs::File;
use std::io::{self, BufRead, BufReader};
use std::path::Path;

fn count_nonempty_lines(path: &Path) -> io::Result<usize> {
    let file = File::open(path)?;
    let reader = BufReader::new(file);
    let mut count = 0;

    for line in reader.lines() {
        if !line?.trim().is_empty() {
            count += 1;
        }
    }

    Ok(count)
}
```

| Input shape                       | Preferred approach                |
| --------------------------------- | --------------------------------- |
| Small trusted UTF-8 configuration | `fs::read_to_string`              |
| Large line-oriented text          | `BufReader::lines`                |
| Arbitrary binary input            | `Read` into a bounded byte buffer |
| Many small output writes          | `BufWriter` + explicit `flush`    |
| In-memory test fixture            | `Cursor<Vec<u8>>`                 |

### 10. Spawn Programs Without Re-parsing a Shell String

```rust
use std::io;
use std::process::{Command, Output};

fn rustc_version() -> io::Result<Output> {
    Command::new("rustc")
        .arg("--version")
        .output()
}

fn checked_output() -> Result<String, String> {
    let output = rustc_version().map_err(|error| error.to_string())?;
    if !output.status.success() {
        return Err(format!("rustc exited with {}", output.status));
    }

    String::from_utf8(output.stdout).map_err(|error| error.to_string())
}
```

Passing arguments separately avoids shell quoting and injection problems. Use a shell
only when shell semantics—pipes, expansion, or redirection—are deliberately required,
and never concatenate untrusted text into a shell program.

### 11. Recipe Adoption Checklist

| Review area  | Question                                                     |
| ------------ | ------------------------------------------------------------ |
| Ownership    | Can inputs be borrowed rather than cloned?                   |
| Errors       | Does every failure preserve useful context?                  |
| Limits       | Are memory, recursion, file, task, and output sizes bounded? |
| Portability  | Do path, process, newline, and filesystem assumptions hold?  |
| Dependencies | Is the crate current, maintained, and necessary?             |
| Security     | Which bytes, paths, URLs, and arguments are untrusted?       |
| Testing      | Are success, boundary, partial-failure, and cleanup tested?  |

---

## λ Functional Programming in Rust

[`fp-core.rs`](https://github.com/JasonShin/fp-core.rs) presents functional-programming
vocabulary and purely functional data structures for Rust. Most of the underlying ideas
also appear directly in standard Rust through closures, iterators, algebraic data types,
`Option`, `Result`, and immutable-by-default bindings.

> λ **Mental model:** functional programming makes transformation and composition
> primary. Rust adds explicit ownership, lifetimes, and controlled mutation to that
> model.

### 1. Pure Core, Imperative Shell

A **pure** function depends only on its arguments and does not mutate external state.
The same input therefore produces the same output.

```rust
#[derive(Clone, Copy)]
struct Token {
    kind: TokenKind,
    width: usize,
}

fn total_width(tokens: &[Token]) -> usize {
    tokens.iter().map(|token| token.width).sum()
}
```

| Pure core                           | Imperative shell                        |
| ----------------------------------- | --------------------------------------- |
| AST transformations                 | reading files                           |
| type rules                          | printing diagnostics                    |
| byte decoding from a supplied slice | receiving network packets               |
| optimization over an explicit IR    | timers, randomness, database operations |

Push I/O and nondeterminism to narrow adapters. A pure compiler core is easier to test,
cache, parallelize, and compare with another implementation.

### 2. Higher-Order Functions and Closures

A **higher-order function** accepts or returns another function. A closure can capture
state from its environment.

```rust
fn retain_matching<T>(
    values: Vec<T>,
    mut predicate: impl FnMut(&T) -> bool,
) -> Vec<T> {
    values
        .into_iter()
        .filter(|value| predicate(value))
        .collect()
}

fn at_least(minimum: i64) -> impl Fn(&i64) -> bool {
    move |value| *value >= minimum
}
```

The returned closure **partially applies** `minimum`: it turns a two-input idea into a
reusable one-input predicate.

### 3. Composition Is Typed Pipelining

```rust
fn compose<A, B, C>(
    first: impl Fn(A) -> B,
    second: impl Fn(B) -> C,
) -> impl Fn(A) -> C {
    move |input| second(first(input))
}

let trim = |text: String| text.trim().to_owned();
let length = |text: String| text.len();
let trimmed_length = compose(trim, length);

assert_eq!(trimmed_length("  AST  ".to_owned()), 3);
```

| Pipeline stage | Type               |
| -------------- | ------------------ |
| trim           | `String -> String` |
| length         | `String -> usize`  |
| composition    | `String -> usize`  |

Compiler passes compose similarly, but fallibility and diagnostics often make the real
type `Input -> Result<Output, Error>`.

### 4. Algebraic Data Types Carry Meaning

Rust enums are **sum types**: a value is one variant. Structs and tuples are **product
types**: a value contains all fields.

```rust
enum Expr {
    Integer(i64),
    Name(SymbolId),
    Call {
        callee: Box<Expr>,
        arguments: Vec<Expr>,
    },
}
```

```text
Expr = Integer(i64)
     + Name(SymbolId)
     + Call(Expr × List<Expr>)
```

This algebra explains exhaustiveness, representation choices, serialization tags, and
why an enum often models runtime or protocol state better than several booleans.

### 5. `map`, `and_then`, and Context

| Operation   | Shape                           | Meaning                        |
| ----------- | ------------------------------- | ------------------------------ |
| `map`       | `F<A>` + `(A -> B)` → `F<B>`    | transform a value in context   |
| `and_then`  | `F<A>` + `(A -> F<B>)` → `F<B>` | chain a context-producing step |
| `unwrap_or` | `F<A>` + fallback → `A`         | leave the context deliberately |

```rust
fn identifier_length(source: &str) -> Result<usize, FrontendError> {
    lex_one_identifier(source)
        .map(|token| token.text.len())
}

fn resolve_identifier(
    source: &str,
    symbols: &SymbolTable,
) -> Result<SymbolId, FrontendError> {
    lex_one_identifier(source)
        .and_then(|token| symbols.resolve(token.text).map_err(FrontendError::from))
}
```

`Option` carries possible absence; `Result` carries possible failure. Their combinators
preserve that context without nested control flow.

### 6. Monoids Describe Safe Reduction

A monoid has an associative combine operation and an identity element.

| Type/operation       | Identity   | Compiler use          |
| -------------------- | ---------- | --------------------- |
| integers under `+`   | `0`        | instruction counts    |
| strings under append | empty      | generated text        |
| sets under union     | empty set  | data-flow facts       |
| lists under concat   | empty list | collected diagnostics |

Associativity permits chunked or parallel reduction, but floating-point addition is not
mathematically associative at machine precision. Deterministic builds must define
reduction order where rounding matters.

### 7. Referential Transparency Enables Reasoning

If an expression can be replaced by its value without changing behavior, it is
referentially transparent. That supports:

| Technique                        | Required caution                              |
| -------------------------------- | --------------------------------------------- |
| memoization                      | inputs must capture every dependency          |
| common subexpression elimination | operation must be safe to reuse               |
| lazy evaluation                  | effects and resource lifetime change timing   |
| parallel evaluation              | operations must not depend on hidden ordering |
| property-based testing           | generators must respect input invariants      |

Functional terminology is useful, but do not force abstractions where ownership becomes
opaque. Prefer ordinary iterators and explicit enums until a more general abstraction
demonstrably improves the code.

> ➡️ **Next:** algorithms apply these transformation, reduction, and invariant ideas to
> concrete problems with measurable resource costs.

---

## 🧮 Algorithms in Idiomatic Rust

This section expands the algorithm categories from
[Idiomatic Rust Snippets](https://idiomatic-rust-snippets.org/algorithms/intro.html):
sorting, searching, graph traversal, and dynamic programming. The emphasis here is on
interfaces and invariants useful in compilers, interpreters, and low-level tools.

> 🧠 **Algorithm-selection rule:** choose from the **contract and constraints**, not
> from whichever algorithm name is easiest to remember.

### 1. Start with the Contract

Before choosing an algorithm, define its contract:

| Contract dimension | Decision to make                                   |
| ------------------ | -------------------------------------------------- |
| Input              | Shape, ownership, and validity                     |
| Output             | Required result and failure representation         |
| Comparison         | Ordering and equality rules                        |
| Mutation           | Whether the input may be changed                   |
| Scale              | Expected data size and worst-case limit            |
| Stability          | Whether equal items must preserve relative order   |
| Invalid input      | Reject, ignore, repair, or return a partial result |

Prefer slices for algorithms that do not need to own or resize data:

```rust
fn is_sorted<T: Ord>(items: &[T]) -> bool {
    items.windows(2).all(|pair| pair[0] <= pair[1])
}

fn reverse_in_place<T>(items: &mut [T]) {
    items.reverse();
}
```

`&[T]` accepts arrays, vectors, boxed slices, and subslices without allocation.

### 2. Complexity Is a Growth Model

| Complexity      | Typical example       | Interpretation                         |
| --------------- | --------------------- | -------------------------------------- |
| \(O(1)\)        | Index an array        | Work does not grow with input length   |
| \(O(\log n)\)   | Binary search         | Repeatedly halves the search space     |
| \(O(n)\)        | Scan for a token      | Visits input once                      |
| \(O(n \log n)\) | Comparison sort       | Common efficient general sorting bound |
| \(O(n^2)\)      | Compare every pair    | Often acceptable only for small inputs |
| \(O(2^n)\)      | Enumerate all subsets | Becomes infeasible quickly             |

| Performance dimension  | Why it matters                                         |
| ---------------------- | ------------------------------------------------------ |
| **Space complexity**   | Bounds extra memory use                                |
| **Amortized cost**     | Spreads occasional expensive work over many operations |
| **Worst-case latency** | Matters for services, tools, and real-time paths       |
| **Constant factors**   | Separate practical implementations with similar big-O  |
| **Locality**           | Contiguous access can beat pointer-heavy structures    |

### 3. Sorting Choices

| Algorithm   | Average time    | Worst time      | Extra space     | Stable?    |
| ----------- | --------------- | --------------- | --------------- | ---------- |
| Bubble sort | \(O(n^2)\)      | \(O(n^2)\)      | \(O(1)\)        | Yes        |
| Quicksort   | \(O(n \log n)\) | \(O(n^2)\)      | Recursion stack | Usually no |
| Merge sort  | \(O(n \log n)\) | \(O(n \log n)\) | \(O(n)\)        | Yes        |
| Heap sort   | \(O(n \log n)\) | \(O(n \log n)\) | \(O(1)\)        | No         |

In application code, use the standard library unless implementing an algorithm for
learning or a specialized constraint:

```rust
let mut diagnostics = vec![
    ("warning", 20),
    ("error", 5),
    ("note", 12),
];

diagnostics.sort_by_key(|(_, byte_offset)| *byte_offset);
```

Rust slice sorting:

- `sort` and `sort_by_key` preserve the relative order of equal elements;
- `sort_unstable` and `sort_unstable_by_key` may be faster or use less temporary memory
  when stability is unnecessary.

#### In-Place Quicksort for Study

```rust
fn quick_sort<T: Ord>(items: &mut [T]) {
    if items.len() < 2 {
        return;
    }

    let pivot = partition(items);
    let (left, right_with_pivot) = items.split_at_mut(pivot);
    let (_, right) = right_with_pivot.split_first_mut().expect("pivot exists");

    quick_sort(left);
    quick_sort(right);
}

fn partition<T: Ord>(items: &mut [T]) -> usize {
    let last = items.len() - 1;
    items.swap(items.len() / 2, last);

    let mut store = 0;
    for current in 0..last {
        if items[current] < items[last] {
            items.swap(current, store);
            store += 1;
        }
    }

    items.swap(store, last);
    store
}
```

> 🧭 **Partition invariant:** values left of the boundary are smaller than the pivot;
> unchecked values remain in the middle.

```text
[ values < pivot | unchecked values | pivot ]
                    ^
                  current
```

After partitioning, the pivot is in its final position. The two remaining slices can be
sorted independently.

### 4. Linear and Binary Search

Linear search works on any sequence:

```rust
fn find_token(tokens: &[Token], wanted: &Token) -> Option<usize> {
    tokens.iter().position(|token| token == wanted)
}

#[derive(PartialEq)]
enum Token {
    Identifier(String),
    Number(i64),
    Plus,
}
```

Binary search requires sorted input:

```rust
let symbols = ["add", "main", "print", "read"];
assert_eq!(symbols.binary_search(&"print"), Ok(2));
assert_eq!(symbols.binary_search(&"parse"), Err(2));
```

For duplicate values, `binary_search` may return any matching position. Use a boundary
search when the first valid insertion point matters:

```rust
fn lower_bound<T: Ord>(items: &[T], target: &T) -> usize {
    let mut low = 0;
    let mut high = items.len();

    while low < high {
        let mid = low + (high - low) / 2;
        if &items[mid] < target {
            low = mid + 1;
        } else {
            high = mid;
        }
    }

    low
}

assert_eq!(lower_bound(&[1, 2, 2, 2, 5], &2), 1);
assert_eq!(lower_bound(&[1, 2, 2, 2, 5], &4), 4);
```

> 🧭 **Loop invariant:** the answer always remains inside the current search interval.

- indexes below `low` contain values less than `target`;
- indexes at or above `high` are not part of the remaining uncertainty;
- the answer remains in `low..=high`.

### 5. Breadth-First Search

BFS explores by increasing edge distance. Use `VecDeque`, not `Vec::remove(0)`.

```rust
use std::collections::VecDeque;

fn bfs_distances(graph: &[Vec<usize>], start: usize) -> Vec<Option<usize>> {
    let mut distance = vec![None; graph.len()];
    let mut queue = VecDeque::new();

    if start >= graph.len() {
        return distance;
    }

    distance[start] = Some(0);
    queue.push_back(start);

    while let Some(node) = queue.pop_front() {
        let next_distance = distance[node].expect("queued nodes have a distance") + 1;

        for &neighbor in &graph[node] {
            if neighbor < graph.len() && distance[neighbor].is_none() {
                distance[neighbor] = Some(next_distance);
                queue.push_back(neighbor);
            }
        }
    }

    distance
}
```

| BFS application          | Why breadth-first order helps                   |
| ------------------------ | ----------------------------------------------- |
| Unweighted shortest path | First arrival uses the fewest edges             |
| Nearest state            | Explores states by increasing transition count  |
| Dependency distance      | Assigns layers from a starting node             |
| Level-order traversal    | Visits a tree one depth at a time               |
| Minimum transformations  | Finds the smallest number of uniform-cost steps |

Mark a node visited when enqueuing it, not when dequeuing, to avoid duplicate queue
entries.

### 6. Depth-First Search

DFS explores one path before backtracking. An explicit stack avoids overflowing the Rust
call stack on a deep graph:

```rust
fn dfs_order(graph: &[Vec<usize>], start: usize) -> Vec<usize> {
    let mut order = Vec::new();
    let mut visited = vec![false; graph.len()];
    let mut stack = vec![start];

    while let Some(node) = stack.pop() {
        if node >= graph.len() || visited[node] {
            continue;
        }

        visited[node] = true;
        order.push(node);

        // Reverse so the first listed neighbor is visited first.
        stack.extend(graph[node].iter().rev().copied());
    }

    order
}
```

| DFS application               | Useful traversal property                  |
| ----------------------------- | ------------------------------------------ |
| AST/syntax-tree walk          | Follow a subtree before its siblings       |
| Reachability                  | Explore an entire connected region         |
| Cycle detection               | Track active recursion/visit state         |
| Strongly connected components | Use DFS ordering in classic SCC algorithms |
| Postorder code generation     | Process children before parents            |
| GC marking                    | Trace object references to completion      |

Choose recursive DFS when the input depth is strictly bounded and recursion clarifies
the algorithm. Otherwise prefer an explicit stack.

### 7. Topological Sorting

A directed acyclic graph can be ordered so every dependency appears before its users.
Kahn's algorithm repeatedly removes nodes with zero incoming edges:

```rust
use std::collections::VecDeque;

fn topological_sort(graph: &[Vec<usize>]) -> Option<Vec<usize>> {
    let mut incoming = vec![0usize; graph.len()];

    for neighbors in graph {
        for &neighbor in neighbors {
            if neighbor >= graph.len() {
                return None;
            }
            incoming[neighbor] += 1;
        }
    }

    let mut ready: VecDeque<_> = incoming
        .iter()
        .enumerate()
        .filter_map(|(node, &count)| (count == 0).then_some(node))
        .collect();

    let mut order = Vec::with_capacity(graph.len());

    while let Some(node) = ready.pop_front() {
        order.push(node);
        for &neighbor in &graph[node] {
            incoming[neighbor] -= 1;
            if incoming[neighbor] == 0 {
                ready.push_back(neighbor);
            }
        }
    }

    (order.len() == graph.len()).then_some(order)
}
```

If not every node is emitted, the graph contains a cycle.

| Topological-sort use    | Dependency represented by each edge          |
| ----------------------- | -------------------------------------------- |
| Module build order      | Imported module must be available first      |
| Constant initialization | Dependency value must be initialized first   |
| IR scheduling           | Producer operation must precede its consumer |
| Pass/migration ordering | Prerequisite step must run first             |
| Cycle diagnosis         | Unemitted nodes reveal a dependency cycle    |

### 8. Dynamic Programming

Dynamic programming applies when:

- the problem decomposes into subproblems;
- subproblems overlap;
- the best answer can be built from smaller answers.

Two styles:

- **memoization:** recursive, cache results on demand;
- **tabulation:** iterative, fill a table in dependency order.

Example: longest common subsequence length with rolling rows:

```rust
fn lcs_length<T: Eq>(left: &[T], right: &[T]) -> usize {
    let mut previous = vec![0usize; right.len() + 1];
    let mut current = vec![0usize; right.len() + 1];

    for left_item in left {
        for (column, right_item) in right.iter().enumerate() {
            current[column + 1] = if left_item == right_item {
                previous[column] + 1
            } else {
                current[column].max(previous[column + 1])
            };
        }

        std::mem::swap(&mut previous, &mut current);
        current.fill(0);
    }

    previous[right.len()]
}

assert_eq!(lcs_length(b"compiler", b"compare"), 5);
```

The full \(O(mn)\) table is unnecessary when each row depends only on the previous row.
This reduces extra space to \(O(n)\).

| DP use in language tools | Optimized subproblem                        |
| ------------------------ | ------------------------------------------- |
| Sequence/tree diff       | Best alignment of smaller prefixes/subtrees |
| Parser recovery          | Lowest-cost repair path                     |
| Instruction selection    | Cheapest covering of IR patterns            |
| Parenthesization         | Lowest-cost evaluation order                |
| Diagnostic edit distance | Minimum edits between names                 |
| Layout/line breaking     | Best formatting up to each position         |

### 9. Weighted Graphs and Dijkstra's Algorithm

Use Dijkstra's algorithm when edge weights are non-negative and the goal is the
shortest distance from one source. The key operation is **relaxation**:

```text
candidate = distance[current] + edge_weight
if candidate < distance[neighbor]:
    distance[neighbor] = candidate
```

Rust's `BinaryHeap` is a max-heap, so reverse the comparison to make the cheapest
pending state come out first:

```rust
use std::cmp::Ordering;
use std::collections::BinaryHeap;

#[derive(Clone, Copy, Debug, Eq, PartialEq)]
struct Edge {
    to: usize,
    cost: u64,
}

#[derive(Clone, Copy, Debug, Eq, PartialEq)]
struct State {
    node: usize,
    cost: u64,
}

impl Ord for State {
    fn cmp(&self, other: &Self) -> Ordering {
        other
            .cost
            .cmp(&self.cost)
            .then_with(|| self.node.cmp(&other.node))
    }
}

impl PartialOrd for State {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        Some(self.cmp(other))
    }
}

fn dijkstra(graph: &[Vec<Edge>], start: usize) -> Option<Vec<Option<u64>>> {
    if start >= graph.len() {
        return None;
    }

    let mut distance = vec![None; graph.len()];
    let mut frontier = BinaryHeap::new();
    distance[start] = Some(0);
    frontier.push(State {
        node: start,
        cost: 0,
    });

    while let Some(State { node, cost }) = frontier.pop() {
        if distance[node].is_some_and(|best| cost > best) {
            continue; // Stale queue entry.
        }

        for edge in &graph[node] {
            if edge.to >= graph.len() {
                return None;
            }

            let candidate = cost.checked_add(edge.cost)?;
            let improves = distance[edge.to].is_none_or(|best| candidate < best);

            if improves {
                distance[edge.to] = Some(candidate);
                frontier.push(State {
                    node: edge.to,
                    cost: candidate,
                });
            }
        }
    }

    Some(distance)
}
```

Important details:

- Skip stale heap entries instead of trying to decrease a key in place.
- Use checked addition when costs can come from untrusted input.
- `None` represents infinity without reserving a magic integer value.
- Dijkstra is incorrect for negative edges; use Bellman–Ford or reformulate the
  problem.
- With an adjacency list and binary heap, the usual bound is
  \(O((V + E)\log V)\).

Compiler and reversing uses include weighted instruction selection, cheapest conversion
paths, control-flow navigation, and finding a minimum-cost sequence of state
transitions.

### 10. Disjoint Sets (Union-Find)

A disjoint-set structure maintains equivalence classes. It supports:

- `find(x)` — return the representative of `x`;
- `union(a, b)` — merge the two sets.

Path compression and union by rank make a long sequence of operations nearly linear:

```rust
#[derive(Debug)]
struct DisjointSet {
    parent: Vec<usize>,
    rank: Vec<u8>,
}

impl DisjointSet {
    fn new(len: usize) -> Self {
        Self {
            parent: (0..len).collect(),
            rank: vec![0; len],
        }
    }

    fn find(&mut self, item: usize) -> Option<usize> {
        let parent = *self.parent.get(item)?;
        if parent != item {
            let root = self.find(parent)?;
            self.parent[item] = root;
        }
        Some(self.parent[item])
    }

    fn union(&mut self, left: usize, right: usize) -> Option<bool> {
        let mut left_root = self.find(left)?;
        let mut right_root = self.find(right)?;

        if left_root == right_root {
            return Some(false);
        }

        if self.rank[left_root] < self.rank[right_root] {
            std::mem::swap(&mut left_root, &mut right_root);
        }

        self.parent[right_root] = left_root;
        if self.rank[left_root] == self.rank[right_root] {
            self.rank[left_root] = self.rank[left_root].saturating_add(1);
        }

        Some(true)
    }
}
```

| Union-find application | Equivalence being maintained                 |
| ---------------------- | -------------------------------------------- |
| Kruskal's algorithm    | Vertices already connected by selected edges |
| Type unification       | Type variables known to be equal             |
| Alias analysis         | Values that may refer to the same region     |
| Connected components   | Nodes belonging to one component             |
| Symbols/relocations    | Records known to represent one entity        |

The invariant is that every parent chain ends at a root whose parent is itself.

### 11. More Dynamic Programming: 0/1 Knapsack

The 0/1 knapsack problem chooses each item at most once while maximizing value under a
capacity limit. A one-dimensional table is enough if capacities are visited in reverse:

```rust
#[derive(Clone, Copy, Debug)]
struct Item {
    weight: usize,
    value: u64,
}

fn knapsack(items: &[Item], capacity: usize) -> Option<u64> {
    let mut best = vec![0u64; capacity.checked_add(1)?];

    for item in items {
        if item.weight == 0 {
            return None; // Define zero-weight semantics explicitly.
        }

        for remaining in (item.weight..=capacity).rev() {
            let with_item = best[remaining - item.weight].checked_add(item.value)?;
            best[remaining] = best[remaining].max(with_item);
        }
    }

    best.last().copied()
}
```

Reverse iteration is the essential invariant. Forward iteration would allow the newly
updated entry to be reused, silently changing the problem into **unbounded** knapsack.

Language-tool analogies:

- select optimizations under a compile-time budget;
- choose instructions under a byte-size limit;
- pack constants or sections under an alignment/size constraint;
- choose diagnostic repairs with bounded cost.

The table uses \(O(\text{capacity})\) memory and the algorithm uses
\(O(\text{items} \times \text{capacity})\) time. This is pseudo-polynomial: a numerically
large capacity can still make it impractical.

### 12. Cryptographic Algorithms: Learn the Shape, Use a Library

Cryptography combines algorithms with strict protocol rules. A mathematically correct
primitive can still be insecure because of nonce reuse, weak randomness, timing leaks,
bad serialization, missing authentication, or poor key handling.

| Goal                         | Appropriate primitive                         |
| ---------------------------- | --------------------------------------------- |
| Detect accidental corruption | Non-cryptographic checksum                    |
| Integrity without a key      | Cryptographic hash                            |
| Integrity with a shared key  | Message authentication code (MAC)             |
| Confidentiality + integrity  | Authenticated encryption with associated data |
| Store passwords              | Password KDF with salt and work factor        |
| Establish a shared secret    | Authenticated key-exchange protocol           |
| Prove who signed data        | Digital signature                             |

| Cryptographic rule        | Practical meaning                                       |
| ------------------------- | ------------------------------------------------------- |
| Authenticate encryption   | Confidentiality alone does not prevent tampering        |
| Respect nonce rules       | Public does not mean reusable                           |
| Distinguish salts/keys    | A salt is neither a password nor an encryption key      |
| Hashes are one-way        | Hashing is not encryption                               |
| Use a password KDF        | Fast general hashes are unsuitable for password storage |
| Compare in constant time  | Use the library's authenticator comparison              |
| Minimize key exposure     | Keep keys scoped and never log them                     |
| Version the format        | Record algorithms and parameters alongside ciphertext   |
| Use audited constructions | Do not invent production ciphers or protocols           |

#### Educational Building Block: Modular Exponentiation

RSA and Diffie–Hellman use modular arithmetic, but real implementations also require
large integers, carefully generated parameters, padding/encoding rules, side-channel
resistance, and secure randomness. The following is only the square-and-multiply
arithmetic pattern:

```rust
fn mod_pow(mut base: u128, mut exponent: u128, modulus: u128) -> Option<u128> {
    if modulus == 0 {
        return None;
    }

    base %= modulus;
    let mut result = 1 % modulus;

    while exponent != 0 {
        if exponent & 1 == 1 {
            result = result.checked_mul(base)? % modulus;
        }
        exponent >>= 1;
        if exponent != 0 {
            base = base.checked_mul(base)? % modulus;
        }
    }

    Some(result)
}
```

This function is useful for learning exponentiation by squaring, but it is **not**
constant time and ordinary `u128` multiplication can overflow before reduction. It is
not a production cryptographic implementation.

#### Cryptography as a Parser and Protocol Problem

Treat authenticated data as a precisely encoded structure:

```text
version || algorithm_id || nonce || associated_data || ciphertext || tag
```

The decoder should:

1. bound all lengths before allocating;
2. reject unknown or forbidden algorithm identifiers;
3. authenticate before releasing plaintext;
4. reject duplicate/replayed nonces when the protocol requires it;
5. keep parsing errors from becoming a useful oracle;
6. preserve exact bytes used as associated data;
7. fail closed.

This is directly relevant to designing a language runtime: signed packages, bytecode
modules, plugin manifests, update files, and network messages are all binary formats
with trust boundaries.

### 13. Where Algorithms Appear in Language Tools

| Language-tool problem           | Useful algorithm/data structure |
| ------------------------------- | ------------------------------- |
| Token lookup                    | Hash table or trie              |
| Source-offset lookup            | Binary search                   |
| AST walk                        | DFS                             |
| Shortest unweighted path in CFG | BFS                             |
| Module dependency order         | Topological sort                |
| Recursive call groups           | Strongly connected components   |
| Type equivalence classes        | Union-find/disjoint set         |
| Data-flow analysis              | Worklist fixed-point iteration  |
| Register allocation             | Graph coloring or linear scan   |
| Constant propagation            | Lattice + worklist              |
| Garbage-collector marking       | DFS/BFS over object graph       |
| Diagnostic suggestions          | Edit distance                   |
| Instruction scheduling          | Dependency DAG                  |

### 14. Worklist Fixed-Point Pattern

Many compiler analyses repeatedly propagate facts until nothing changes:

```text
initialize facts
enqueue affected nodes

while worklist is not empty:
    node = pop
    new_fact = transfer(predecessor facts)
    if new_fact changed:
        store it
        enqueue successors
```

| Fixed-point component | Requirement                                   |
| --------------------- | --------------------------------------------- |
| Fact domain           | Finite height or another convergence argument |
| Transfer function     | Monotone with respect to the fact ordering    |
| Change detection      | Reliable equality or improvement test         |
| Dependencies          | Precise successors to re-enqueue              |
| Tests                 | Deterministic expected final facts            |

### 15. Algorithm Engineering Checklist

| Area            | Preferred practice                                                       |
| --------------- | ------------------------------------------------------------------------ |
| Standard tools  | Use library sorting/searching in production                              |
| Invariants      | State them beside non-obvious loops                                      |
| Arithmetic      | Check indexes, capacities, and input-derived costs                       |
| Depth           | Avoid recursion over unbounded adversarial input                         |
| Allocation      | Preallocate only from trustworthy estimates                              |
| Queues/heaps    | Use `VecDeque` for FIFO and `BinaryHeap` for priority                    |
| Membership      | Use a set or boolean vector instead of repeated scans                    |
| Reproducibility | Preserve stable ordering when output must be deterministic               |
| Edge cases      | Test empty, duplicate, cyclic, disconnected, and maximum inputs          |
| Performance     | Benchmark realistic workloads; big-O is not the whole result             |
| Generics        | Generalize only when reuse improves without hiding the algorithm         |
| Failure         | Return `Option`/`Result` when absence or failure is part of the contract |

---

## 🤖 Machine Learning Foundations

The [Machine Learning Glossary](https://ml-cheatsheet.readthedocs.io/en/latest/index.html)
organizes the field around linear/logistic regression, calculus, linear algebra,
forward propagation, backpropagation, activations, losses, optimizers, regularization,
and model architectures. The goal here is a systems-level mental model, not a substitute
for a statistics course or a production ML framework.

### 1. Learning Means Fitting a Parameterized Function

```text
input features x
    ↓
model f(x; parameters θ)
    ↓
prediction ŷ
    ↓ compare with target y
loss L(ŷ, y)
    ↓ optimizer updates θ
```

| Term           | Meaning                                     |
| -------------- | ------------------------------------------- |
| feature        | measured or derived model input             |
| label/target   | desired output used for supervised learning |
| parameter      | value learned during training               |
| hyperparameter | configuration chosen outside training       |
| inference      | compute a prediction with fixed parameters  |
| training       | adjust parameters to reduce an objective    |
| epoch          | one pass over the training data             |
| batch          | subset used for one update                  |

### 2. Split Data by Purpose

| Split      | Used for                                | Must not be used for              |
| ---------- | --------------------------------------- | --------------------------------- |
| training   | fitting parameters                      | final performance claim           |
| validation | selecting models and hyperparameters    | repeated unbiased final reporting |
| test       | final estimate after choices are frozen | tuning                            |

**Data leakage** occurs when training receives information that would not exist at real
prediction time or indirectly sees validation/test targets. Split related samples by
user, machine, project, time, or other dependency when random rows would leak context.

### 3. Linear Regression in Small Rust

For one feature:

```text
prediction = weight × x + bias
error      = prediction - target
MSE        = mean(error²)
```

```rust
#[derive(Debug, Clone, Copy)]
struct LinearModel {
    weight: f64,
    bias: f64,
}

impl LinearModel {
    fn predict(self, x: f64) -> f64 {
        self.weight * x + self.bias
    }

    fn train_step(&mut self, samples: &[(f64, f64)], rate: f64) {
        if samples.is_empty() {
            return;
        }

        let count = samples.len() as f64;
        let (weight_gradient, bias_gradient) =
            samples.iter().fold((0.0, 0.0), |(dw, db), &(x, target)| {
                let error = self.predict(x) - target;
                (dw + (2.0 / count) * error * x, db + (2.0 / count) * error)
            });

        self.weight -= rate * weight_gradient;
        self.bias -= rate * bias_gradient;
    }
}
```

This example is intentionally small. Production training needs numerical checks,
vectorized/tensor operations, reproducibility controls, efficient batching, and mature
libraries.

### 4. Gradient Descent Is Local Iterative Optimization

```text
θ(next) = θ(current) - learning_rate × ∇L(θ)
```

| Symptom                     | Possible cause                                |
| --------------------------- | --------------------------------------------- |
| loss explodes               | rate too high, poor scaling, numerical issue  |
| loss barely moves           | rate too low, saturation, weak features       |
| training good/test poor     | overfitting or leakage                        |
| training and test both poor | underfitting, bad representation, bad labels  |
| results vary wildly         | seed, ordering, nondeterminism, small dataset |

Feature normalization changes optimization geometry. It does not create information
that the features do not contain.

### 5. Classification and Regression Need Different Outputs

| Task                      | Output                            | Common loss/metric examples            |
| ------------------------- | --------------------------------- | -------------------------------------- |
| regression                | continuous value                  | MSE, MAE, RMSE                         |
| binary classification     | probability/logit for two classes | binary cross-entropy, precision/recall |
| multiclass classification | class distribution/logits         | cross-entropy, top-k accuracy          |
| ranking                   | relative ordering                 | pairwise/listwise objectives           |
| clustering                | unlabeled grouping                | distance/internal validity measures    |

Accuracy can be misleading on imbalanced data. A detector that predicts “safe” for
99.9% of inputs can score highly while missing every rare failure. Choose metrics from
the cost of false positives and false negatives.

### 6. Neural Networks Compose Differentiable Layers

```text
input
  → affine transform (weights × input + bias)
  → activation
  → more layers
  → output
  → loss
```

| Concept         | Role                                             |
| --------------- | ------------------------------------------------ |
| weight          | learned connection strength                      |
| bias            | learned offset                                   |
| activation      | nonlinear transformation                         |
| forward pass    | compute predictions and intermediates            |
| backpropagation | apply chain rule from loss toward earlier layers |
| optimizer       | turn gradients into parameter updates            |
| regularization  | discourage brittle overfit solutions             |

ReLU is simple and common; sigmoid maps to `(0, 1)` but can saturate; softmax converts
logits into a normalized multiclass distribution. Match the final activation, loss, and
target representation.

### 7. Regularization Controls Generalization

| Technique               | Intuition                                        |
| ----------------------- | ------------------------------------------------ |
| L1 penalty              | encourage sparse parameters                      |
| L2 penalty/weight decay | discourage large parameters                      |
| dropout                 | randomly omit activations during training        |
| early stopping          | stop when validation performance stops improving |
| data augmentation       | encode valid input-preserving transformations    |
| ensembling              | combine models to reduce some variance           |

Regularization cannot repair mislabeled data, a broken split, or a metric unrelated to
the real objective.

### 8. Numerical and Systems Boundaries

| Boundary             | Engineering question                                 |
| -------------------- | ---------------------------------------------------- |
| data loader          | Are order, decoding, and augmentation reproducible?  |
| tensor shape         | Are dimensions and broadcasting intentional?         |
| numeric precision    | Can overflow, underflow, or cancellation occur?      |
| accelerator transfer | Is copy/synchronization cost measured?               |
| checkpoint           | Are model, optimizer, schema, and version recorded?  |
| inference service    | Are latency, batching, memory, and fallback bounded? |
| monitoring           | Can drift and quality degradation be detected?       |

### 9. ML and Language Tools

| Possible use                 | Deterministic guardrail                     |
| ---------------------------- | ------------------------------------------- |
| code completion/ranking      | parser/type checker rejects invalid output  |
| optimization heuristic       | semantic equivalence tests                  |
| decompiler naming suggestion | evidence and analyst confirmation           |
| fuzz-input prioritization    | coverage and reproducible seeds             |
| anomaly detection in traces  | retain raw evidence and explain uncertainty |
| performance autotuning       | benchmark budget and safe fallback          |

Do not replace a cheap exact rule with a probabilistic model merely because ML is
available. Use ML when approximation creates value and its uncertainty can be contained.

> ➡️ **Next:** operating-system and executable chapters show how programs receive
> processes, virtual memory, files, linking, and hardware access before those programs
> become concurrent or distributed.

---

## 💻 Operating Systems from the Bottom Up

This section connects the compiler's output to the operating system that actually runs
it. It is based on [Computer Science from the Bottom Up](https://www.bottomupcs.com/),
especially its chapters on the operating system, processes, virtual memory, ELF, and
dynamic linking.

> 🖥️ **OS mental model:** the kernel owns privileged resources and exposes controlled
> operations to processes through system calls.

### 1. The Kernel as a Privileged Resource Manager

An application cannot safely control the processor, physical memory, disks, and network
devices directly. The **kernel** owns those shared resources and exposes controlled
operations to user programs.

```text
user program
    ↓ library/API call
system-call boundary
    ↓ privilege transition
kernel validates request
    ↓
device, filesystem, scheduler, or memory manager
```

The boundary changes both **privilege** and **trust**:

| Phase      | Responsibility                                        |
| ---------- | ----------------------------------------------------- |
| Request    | User code supplies an operation and arguments         |
| Transition | The processor enters privileged mode                  |
| Validation | Kernel checks pointers, lengths, permissions, handles |
| Execution  | Kernel performs or rejects the operation              |
| Return     | Result/error is copied back to user mode              |

The exact instruction used to enter the kernel depends on the architecture. Older
systems often used a general software trap; modern processors usually provide a faster
system-call instruction. User code normally reaches it through a standard library
wrapper rather than hand-written assembly.

#### System Calls Versus Library Calls

| Kind             | Where it runs       | What it does                                |
| ---------------- | ------------------- | ------------------------------------------- |
| **Library call** | Current process     | Ordinary computation, formatting, buffering |
| **System call**  | Kernel entry        | Requests a privileged operation             |
| Library wrapper  | User + kernel paths | May make zero, one, or several system calls |

For example, buffered output may append bytes to a user-space buffer many times and
perform a `write` system call only when the buffer is full or flushed.

```text
printf / println / buffered writer
       ↓ formatting and buffering in user space
write-like system call
       ↓ kernel copies or maps data toward a file/device
```

This distinction explains why source-level operations do not map one-to-one to kernel
operations.

### 2. What a Process Contains

A **program** is an executable representation stored somewhere. A **process** is a
running instance plus all state required to stop and resume it.

| Process component     | What it represents                                    |
| --------------------- | ----------------------------------------------------- |
| Process ID            | Kernel-visible identity                               |
| Virtual address space | Code, data, heap, stack, mappings                     |
| Register state        | Instruction pointer, stack pointer, general registers |
| File-descriptor table | Open files, pipes, sockets, devices                   |
| Credentials           | User/group identity and permissions                   |
| Signal state          | Masks, pending signals, handlers                      |
| Scheduling state      | Running, runnable, sleeping, stopped                  |
| Accounting data       | CPU time, memory use, I/O statistics                  |

The kernel does not need to leave every process's register values in the CPU. During a
context switch it saves the outgoing process's state to memory and restores the incoming
process's saved state.

#### Common Process States

```text
             event / wakeup
        ┌────────────────────┐
        ↓                    │
    runnable ──scheduled──→ running
        ↑                    │
        │                    ├── waits for I/O ──→ sleeping
        │                    ├── preempted ──────→ runnable
        │                    └── exits ──────────→ terminated
        └──────── I/O completes ────────────────┘
```

Names vary by operating system, but the distinction is important:

| Process state         | Meaning                                                |
| --------------------- | ------------------------------------------------------ |
| **Running**           | Currently executing on a CPU                           |
| **Runnable**          | Able to execute but waiting for CPU time               |
| **Sleeping/blocked**  | Waiting for an event, timer, lock, or I/O              |
| **Stopped**           | Deliberately suspended                                 |
| **Terminated/zombie** | Finished; exit information may await parent collection |

### 3. File Descriptors: Small Handles to Kernel Objects

On Unix-like systems, a file descriptor is a small process-local integer that indexes a
kernel-maintained table.

| Conventional descriptor | Meaning         |
| ----------------------- | --------------- |
| `0`                     | Standard input  |
| `1`                     | Standard output |
| `2`                     | Standard error  |

The descriptor is not the file itself. It is a handle to an open kernel object with
associated access mode and current state.

```text
process-local fd 3
        ↓
kernel open-file description
        ↓
filesystem object, pipe, socket, terminal, or device
```

This uniform interface is why many different resources support operations resembling
`read`, `write`, and `close`.

| Descriptor consequence | Practical meaning                                       |
| ---------------------- | ------------------------------------------------------- |
| Process-local scope    | Inheritance/transfer requires an explicit OS mechanism  |
| Reference semantics    | `close` releases one reference, not always the object   |
| Aliasing               | Multiple descriptors can share one open-file state      |
| Number reuse           | A closed descriptor value may later name something else |
| No global identity     | The integer is meaningful only with its process context |

### 4. `fork`, `exec`, and Process Creation

Unix process creation is often explained as two separate operations:

1. **`fork`** creates a child based on the calling process.
2. **`exec`** replaces the current process image with a new program.

Conceptually:

```text
shell process
    ↓ fork
shell parent + child copy
                   ↓ redirect descriptors / set environment
                   ↓ exec
              requested program
```

The child does not normally require an eager physical copy of every memory page.
Operating systems use **copy-on-write (COW)**:

| Event                    | COW behavior                                              |
| ------------------------ | --------------------------------------------------------- |
| Immediately after `fork` | Parent and child map the same read-only frames            |
| Read                     | Both processes continue sharing the frame                 |
| First write              | A protection fault transfers control to the kernel        |
| Fault handling           | Kernel copies the page and grants private writable access |

This makes `fork` much cheaper when the child quickly calls `exec`.

`exec` does not create a second process ID. It replaces the current address space,
register setup, and executable image while preserving selected process state such as the
process identity and descriptors that are not marked close-on-exec.

### 5. Context Switching and Scheduling

A **context switch** changes which thread or process owns a CPU:

1. enter the kernel because of a timer, interrupt, system call, or blocking event;
2. save the current execution context;
3. update the current process's state;
4. choose a runnable process;
5. restore its saved context;
6. return to user mode at its saved instruction pointer.

Context switches are necessary but not free:

| Cost source           | Why it costs time                               |
| --------------------- | ----------------------------------------------- |
| Register save/restore | Execution context moves between CPU and memory  |
| Scheduler work        | Kernel chooses and accounts for the next task   |
| Cache disruption      | New code/data displaces useful cache lines      |
| TLB effects           | Address translations may need refilling         |
| Branch prediction     | Predictor history may no longer match           |
| Core migration        | Warm state may not exist on the destination CPU |

**Preemptive scheduling** lets the kernel interrupt a running task. **Cooperative
scheduling** relies on tasks yielding voluntarily. General-purpose operating systems are
normally preemptive; async runtimes often use cooperative tasks inside a process.

### 6. Signals and Asynchronous Events

A signal is a small asynchronous notification delivered to a process or thread:

| Signal category    | Example meaning                         |
| ------------------ | --------------------------------------- |
| Control request    | Terminate, stop, or continue            |
| Memory fault       | Invalid address or protection violation |
| Arithmetic fault   | Invalid numeric operation               |
| Child notification | A child stopped, continued, or exited   |
| Timer              | Alarm or scheduled notification         |
| Application event  | User-defined signal                     |

Signal handlers run in an unusual context. Ordinary application invariants may be
temporarily broken, so only operations documented as safe in a signal handler should be
performed there.

| Runtime interaction    | Signal-related use                          |
| ---------------------- | ------------------------------------------- |
| Stack overflow         | Detect guard-page faults                    |
| Memory management      | Handle deliberately fault-driven techniques |
| Profiling              | Trigger periodic sampling                   |
| Subprocess supervision | Observe child state changes                 |
| Graceful shutdown      | Request orderly termination                 |
| Debugging              | Implement traps and breakpoints             |

Do not use a signal handler as if it were a normal callback.

### 7. Virtual Memory Is Address Translation

Virtual memory is often described as disk pretending to be RAM. That describes **swap**,
which is only one possible consequence. The central idea is that programs use **virtual
addresses**, and the operating system plus hardware translate them to physical memory.

```text
virtual address used by instruction
        ↓ split into page number + page offset
page table translates virtual page
        ↓
physical frame + unchanged offset
        ↓
physical memory location
```

| Virtual-memory term | Meaning                                                   |
| ------------------- | --------------------------------------------------------- |
| **Address space**   | Addresses a process can potentially name                  |
| **Virtual page**    | Fixed-size block in a virtual address space               |
| **Physical frame**  | Fixed-size block in physical memory                       |
| **Page table**      | Mapping and permissions from pages to frames              |
| **Mapping**         | Association with memory, a file, or another kernel object |

Two processes can use the same virtual address while mapping it to different physical
frames. Conversely, two virtual pages can intentionally map the same physical frame for
shared memory.

### 8. Page Tables and Page Permissions

If the page size is \(2^n\), the low \(n\) address bits are the offset inside a page and
the remaining bits identify the virtual page.

For a 4 KiB page:

```text
4 KiB = 4096 bytes = 2^12

virtual address:
+---------------------------+------------+
| virtual page number       | 12-bit     |
|                           | page offset|
+---------------------------+------------+
```

Translation changes the page number but preserves the offset:

```text
virtual page 0x1234, offset 0x056
        ↓ page-table lookup
physical frame 0x09ab, offset 0x056
```

A page-table entry commonly includes:

| PTE field category | What it controls                         |
| ------------------ | ---------------------------------------- |
| Presence           | Whether a valid mapping currently exists |
| Permissions        | Read, write, and execute access          |
| Privilege          | User-accessible or kernel-only           |
| Accessed bit       | Whether the page has been referenced     |
| Dirty bit          | Whether the page has been modified       |
| Cache policy       | How accesses interact with CPU caches    |
| Architecture bits  | Target-specific translation controls     |

These permissions provide isolation and enable mitigations such as non-executable data
pages.

### 9. Multi-Level Page Tables

A flat page table for every possible virtual page would waste large amounts of memory,
especially for sparse address spaces. Multi-level page tables divide the virtual page
number into indexes:

```text
virtual address
    ↓
level-1 index → level-2 index → ... → page-table entry
                                           ↓
                                      physical frame
```

Intermediate tables only need to exist for populated parts of the address space.

This resembles a radix tree: each group of address bits selects the next table. Exact
levels and bit divisions are architecture-specific.

### 10. The TLB

Walking page tables for every memory access would be slow. The processor therefore uses
a **Translation Lookaside Buffer (TLB)**, a small cache of recent virtual-to-physical
translations.

```text
virtual address
    ↓
TLB hit? ── yes ─→ physical address
    │
    no
    ↓
page-table walk → cache translation → retry access
```

A context switch may require translations to be invalidated, tagged by address-space
identity, or otherwise managed so one process cannot use another process's mappings.

Performance implications:

- poor locality can increase both data-cache and TLB misses;
- large pages cover more memory per TLB entry but reduce allocation flexibility;
- changing mappings requires synchronization with CPUs that may cache the old
  translation;
- a compiler's object layout and traversal order can affect page and cache locality.

### 11. Page Faults

A **page fault** means an address translation or permission check could not complete
normally. It is not automatically a bug.

| Fault cause                       | Possible kernel response                  |
| --------------------------------- | ----------------------------------------- |
| Valid file-backed page not loaded | Read or map the page, then retry          |
| Copy-on-write write               | Allocate/copy a private frame, then retry |
| Stack growth within allowed range | Map another stack page                    |
| Swapped-out page                  | Restore it from backing storage           |
| Write to read-only mapping        | Deliver an access violation               |
| Unmapped address                  | Deliver an access violation               |

This distinction matters when debugging: the hardware mechanism is a page fault in all
cases, but the operating system may resolve some transparently and report others to the
process.

### 12. `mmap`, Shared Memory, and the Page Cache

A memory mapping connects virtual addresses to a backing object.

- **Anonymous mapping:** zero-initialized memory not backed by a normal file.
- **Private file mapping:** file content is visible, but writes become private through
  copy-on-write.
- **Shared file mapping:** writes may become visible through the shared mapping and
  eventually reach the file.
- **Shared anonymous mapping:** processes intentionally share frames.

The **page cache** lets the kernel cache file contents in memory. Ordinary file I/O and
memory-mapped I/O may ultimately interact with the same cached pages.

> 🧠 **Page-cache mental model:** file I/O and mapped memory can meet in the kernel's
> cached view of file contents.

```text
file on storage
      ↕
kernel page cache
   ↙       ↘
read/write  mmap views
```

This helps explain why:

- a recently read file may be fast to read again;
- writing does not always mean bytes reached durable storage immediately;
- mapped file accesses can fault pages in lazily;
- executable code and shared libraries can be backed by cached file pages.

### 13. From `exec` to `main`

`main` is usually not the first instruction executed.

```text
exec request
    ↓
kernel reads executable metadata
    ↓
map loadable segments and create initial stack
    ↓
load requested program interpreter / dynamic linker
    ↓
resolve required libraries and relocations
    ↓
transfer control to executable entry point (`_start`)
    ↓
language/runtime startup
    ↓
call user `main`
    ↓
run termination and destructor logic
```

The initial stack can contain arguments, environment entries, and an **auxiliary
vector** with information supplied by the kernel. The runtime startup code uses this
state to initialize the process before calling `main`.

| Runtime startup responsibility | Purpose                                 |
| ------------------------------ | --------------------------------------- |
| Heap/GC initialization         | Prepare managed allocation              |
| Built-in registration          | Install core types and native functions |
| Thread-local state             | Establish per-thread runtime context    |
| Arguments/environment          | Convert OS input into language values   |
| Standard streams               | Connect language I/O to OS handles      |
| Panic/exception handling       | Define top-level failure behavior       |
| Module constructors            | Initialize global/module state          |
| Language entry point           | Call user `main` or equivalent          |
| Exit conversion                | Map the language result to an OS status |

### 14. Bottom-Up Debugging Questions

When a program behaves unexpectedly, walk downward through the abstractions:

1. What source-level operation failed?
2. Which library or runtime function implements it?
3. Did it make a system call?
4. What arguments crossed the kernel boundary?
5. Which descriptor, mapping, or process state did the kernel use?
6. Was the process running, runnable, or blocked?
7. Did an address translation, permission check, or page fault occur?
8. Which executable segment, symbol, or relocation produced the address?

Then walk back upward and explain how the low-level event created the source-level
symptom.

---

## 🔗 Executable Files, Linkers, ABIs & FFI

Compiling instructions is only part of producing a usable program. The result must also
describe how code and data are arranged, which external names it needs, where it may be
loaded, and how functions exchange values.

> 🔗 **Toolchain handoff:** the compiler emits code and metadata, the linker resolves
> relationships, and the loader creates the running image.

### 1. Source Files, Object Files, and Executables

```text
source files
    ↓ compiler
object files
    ↓ linker + libraries
executable or shared library
    ↓ operating-system loader
running process
```

An **object file** usually contains:

| Object-file component | Purpose                                       |
| --------------------- | --------------------------------------------- |
| Code/static data      | Instructions and compile-time data            |
| Symbol table          | Defined and unresolved names                  |
| Relocations           | Places whose addresses need later repair      |
| Debug information     | Maps instructions and values back to source   |
| Target metadata       | Identifies architecture, ABI, and file format |

The linker combines object files, resolves symbols, lays out sections, applies
relocations, and emits the final executable or library.

### 2. Common Sections

Exact names differ among ELF, PE/COFF, Mach-O, and individual toolchains, but these
categories are common:

| Section              | Typical contents                        | Usual permissions     |
| -------------------- | --------------------------------------- | --------------------- |
| `.text`              | Machine instructions                    | Read + execute        |
| `.rodata`            | String literals and immutable constants | Read                  |
| `.data`              | Initialized writable globals            | Read + write          |
| `.bss`               | Zero-initialized globals                | Read + write          |
| Symbol/string tables | Names used by linking or debugging      | Read                  |
| Relocations          | Places whose addresses need adjustment  | Loader/linker data    |
| Debug sections       | Source lines, types, local names        | Not needed to execute |

`.bss` does not need to store a file full of zero bytes. The executable records the
required size, and the loader supplies zero-initialized memory.

### 3. Symbols and Name Resolution

A **symbol** associates a name with a function, object, section, or address.

| Symbol kind   | Linker meaning                            |
| ------------- | ----------------------------------------- |
| **Defined**   | Provided by the current object/library    |
| **Undefined** | Required from another object/library      |
| **Local**     | Private to one object or translation unit |
| **Global**    | Participates in cross-object resolution   |
| **Weak**      | May be replaced by a stronger definition  |

Languages that support overloading or namespaces often use **name mangling** to encode
more information into a linker-visible name:

```text
source name: math::max(i32, i32)
mangled name: implementation-specific encoded symbol
```

An ABI should not rely on an unstable language-specific mangling scheme unless both
sides use compatible toolchains.

### 4. Relocations and Position Independence

An object file often cannot know final addresses. Instead it contains a placeholder plus
a relocation such as:

```text
"At offset 0x24, insert the address of symbol print_value."
```

The linker or loader computes the final value after section placement.

**Position-independent code (PIC)** avoids assuming one fixed load address. Shared
libraries normally need PIC, and position-independent executables allow the operating
system to randomize the program's base address.

| Coordinate                   | Meaning                                 |
| ---------------------------- | --------------------------------------- |
| **File offset**              | Position inside the executable file     |
| **Virtual address (VA)**     | Address where bytes appear in a process |
| **Relative virtual address** | Offset from an image or module base     |

Never mix these three without an explicit conversion.

### 5. Static and Dynamic Linking

| Model           | What is included                                    | Main trade-off                                 |
| --------------- | --------------------------------------------------- | ---------------------------------------------- |
| Static linking  | Required library code is copied into the executable | Larger files, fewer runtime dependencies       |
| Dynamic linking | Executable refers to libraries loaded at runtime    | Smaller files, version and deployment concerns |

Dynamic linking involves at least three parties:

1. the compiler emits calls and symbol references;
2. the linker records dependencies and creates the required tables;
3. the loader maps libraries and resolves runtime addresses.

This is why "it compiled" does not guarantee "it launches." A binary may still have a
missing library, incompatible ABI, unsupported architecture, or loader configuration
problem.

### 6. Calling Conventions

A calling convention answers:

| ABI question       | What must be specified                         |
| ------------------ | ---------------------------------------------- |
| Arguments          | Registers/stack locations by type and position |
| Return value       | Register, memory, or hidden return pointer     |
| Caller-preserved   | Registers the caller must save if needed       |
| Callee-preserved   | Registers the callee must restore if used      |
| Stack cleanup      | Which side removes argument storage            |
| Stack alignment    | Required alignment at call boundaries          |
| Variadic arguments | Encoding and retrieval rules                   |

Conceptual call sequence:

```text
caller:
    place arguments
    save caller-preserved state if needed
    call function

callee:
    establish frame
    save callee-preserved state
    execute body
    place return value
    restore state
    return
```

Compiler backends must implement the target convention exactly. A mismatch can look like
random register corruption even when both functions are individually correct.

### 7. ABI Versus API

- An **API** describes source-level usage: functions, types, and expected behavior.
- An **ABI** describes binary-level compatibility: data layout, symbol naming, calling
  convention, alignment, and register rules.

Two libraries may have the same conceptual API but incompatible ABIs. ABI stability
matters for shared libraries, plugins, operating-system interfaces, and foreign-function
interfaces.

### 8. Rust FFI Boundary

Use an explicit C ABI and C-compatible types when exposing a simple function:

```rust
#[unsafe(no_mangle)]
pub extern "C" fn checked_add(left: i32, right: i32, out: *mut i32) -> bool {
    let Some(sum) = left.checked_add(right) else {
        return false;
    };

    let Some(out) = (unsafe { out.as_mut() }) else {
        return false;
    };

    *out = sum;
    true
}
```

The `unsafe` operation is small, and its required condition is obvious: the incoming
pointer must either be null or point to a valid writable `i32`.

| FFI concern       | Required practice                                            |
| ----------------- | ------------------------------------------------------------ |
| Struct layout     | Use `#[repr(C)]` for C-shared records                        |
| Integer widths    | Use fixed-width types where layout matters                   |
| Allocation        | Define who allocates and who frees                           |
| Unwinding         | Do not let Rust panics cross a foreign ABI                   |
| Rust-native types | Do not expose references, `String`, `Vec`, or trait objects  |
| Pointer contract  | Document nullability, alignment, lifetime, and thread safety |
| Versioning        | Evolve exported interfaces deliberately                      |

### 9. ELF Has a Linking View and an Execution View

ELF deliberately describes the same file in two ways:

| View           | Main structure               | Consumer                   | Purpose                                   |
| -------------- | ---------------------------- | -------------------------- | ----------------------------------------- |
| Linking view   | Sections and section headers | Compiler, linker, debugger | Organize code, data, symbols, relocations |
| Execution view | Segments and program headers | Kernel and dynamic loader  | Map byte ranges with runtime permissions  |

One or more sections can contribute to one loadable segment.

```text
sections:  .text + .rodata ──→ read/execute load segment
sections:  .data + .bss   ──→ read/write load segment
```

This is why "section" and "segment" are related but not interchangeable.

| Program-header field | Meaning                                           |
| -------------------- | ------------------------------------------------- |
| `p_type`             | Loadable segment, interpreter, dynamic data, etc. |
| `p_offset`           | Byte offset in the file                           |
| `p_vaddr`            | Virtual address after loading                     |
| `p_filesz`           | Bytes present in the file                         |
| `p_memsz`            | Bytes required in memory                          |
| `p_flags`            | Read/write/execute permissions                    |
| `p_align`            | Required file/memory alignment relationship       |

When `p_memsz` is larger than `p_filesz`, the loader supplies zero-filled memory for the
difference. This is how zero-initialized data can occupy memory without occupying the
same number of bytes in the file.

### 10. ELF Entry Point and Program Interpreter

An ELF header includes an entry-point virtual address. A program header of type
`PT_INTERP` can name the program interpreter, normally the dynamic linker for a
dynamically linked executable.

Conceptually:

```text
kernel maps executable segments
    ↓ sees PT_INTERP
kernel maps dynamic linker
    ↓ starts dynamic linker
dynamic linker loads dependencies and resolves relocations
    ↓ transfers control to executable entry point
startup code eventually calls main
```

The executable is still native machine code. Here, "interpreter" means a loader that
finishes preparing the native image, not a source-language interpreter.

### 11. Dynamic Symbols, Relocations, GOT, and PLT

A shared library can be loaded at different addresses in different processes, so some
addresses cannot be finalized until runtime.

- `.dynsym` contains symbols needed for dynamic linking.
- `.dynstr` contains names referenced by dynamic symbol entries.
- dynamic relocation sections describe runtime fixups.
- the **Global Offset Table (GOT)** holds runtime-resolved addresses used by
  position-independent code.
- on architectures/toolchains that use it, the **Procedure Linkage Table (PLT)**
  provides call stubs that lead through resolver-managed slots.

Simplified lazy-resolution path:

```text
first call through PLT stub
    ↓ unresolved GOT slot
dynamic resolver finds symbol in loaded libraries
    ↓ writes resolved address into slot
later calls jump through the resolved slot
```

Systems may instead resolve symbols eagerly at startup. Details vary by architecture,
ABI, linker, and hardening settings.

#### Safe Inspection Commands

For a trusted local ELF file:

```sh
file ./program
readelf -h ./program       # ELF header
readelf -l ./program       # program headers / segments
readelf -S ./program       # section headers
readelf -s ./program       # symbols
readelf -r ./program       # relocations
readelf -d ./program       # dynamic entries and dependencies
objdump -d ./program       # disassembly
```

Prefer parsers such as `readelf` and `objdump` for initial inspection of untrusted
files. Do not execute an unknown binary merely to discover its dependencies.

---

## 🔩 Bare-Metal, Unsafe Rust & Chromium-Scale Integration

[Comprehensive Rust](https://google.github.io/comprehensive-rust/index.html) includes
specialized material on bare-metal development, unsafe Rust, and using Rust inside
Chromium. Together they show three versions of the same lesson: make environmental and
safety assumptions explicit at boundaries.

> 🔩 **Low-level rule:** when the operating system or type system cannot enforce an
> assumption, **write the assumption down as a contract**.

### 1. Hosted Versus Bare-Metal Programs

A hosted program starts inside an operating-system process. The OS and standard library
provide files, sockets, threads, clocks, allocation, arguments, and process exit.

A bare-metal program may start directly at a reset vector with no OS, loader, filesystem,
allocator, or console.

```text
reset
  ↓
startup code establishes stack and memory
  ↓
initialize hardware and interrupt table
  ↓
enter firmware main loop or scheduler
```

Rust library layers:

| Layer   | Provides                                                               |
| ------- | ---------------------------------------------------------------------- |
| `core`  | Language fundamentals, slices, iterators, `Option`, `Result`, atomics  |
| `alloc` | `Box`, `Vec`, `String`, `Arc`, and other allocation-backed collections |
| `std`   | OS-backed files, networking, threads, clocks, process APIs, and more   |

`#![no_std]` removes the dependency on `std`; it does not remove `core`. The `alloc`
crate is optional and requires a global allocator supplied by the platform.

### 2. Minimal `no_std` Shape

The exact entry point and linker setup are target-specific, but the source commonly has
this shape:

```rust
#![no_std]
#![no_main]

use core::hint::spin_loop;
use core::panic::PanicInfo;

#[panic_handler]
fn panic(_info: &PanicInfo<'_>) -> ! {
    loop {
        spin_loop();
    }
}

#[unsafe(no_mangle)]
pub extern "C" fn firmware_entry() -> ! {
    loop {
        spin_loop();
    }
}
```

This is not a complete bootable image:

| Target requirement      | Responsibility                                      |
| ----------------------- | --------------------------------------------------- |
| Linker layout           | Place code, data, stack, and device regions         |
| Entry contract          | Use the correct reset symbol and calling convention |
| Stack                   | Establish a valid initial stack                     |
| Static memory           | Copy initialized data and zero `.bss`               |
| Panic strategy          | Define terminal failure behavior                    |
| Build/runner config     | Select target, image format, and launch mechanism   |
| Hardware initialization | Configure clocks, memory, pins, and peripherals     |

Those responsibilities resemble a language runtime's startup sequence. Building
firmware makes the usually hidden path from executable entry point to language-level
`main` visible.

### 3. Hardware-Abstraction Layers

A useful embedded stack:

```text
application
    ↓
board support package (specific board and pins)
    ↓
hardware abstraction layer (typed peripheral operations)
    ↓
peripheral access crate (register-level device description)
    ↓
memory-mapped registers
```

Keep application logic testable on the host by depending on small traits:

```rust
trait OutputPin {
    type Error;

    fn set_high(&mut self) -> Result<(), Self::Error>;
    fn set_low(&mut self) -> Result<(), Self::Error>;
}

fn blink_once<P: OutputPin>(pin: &mut P) -> Result<(), P::Error> {
    pin.set_high()?;
    pin.set_low()
}
```

For real code, return a named error type or the trait's associated error. The important
design idea is that the application owns behavior while the board layer owns volatile
register details.

### 4. Memory-Mapped I/O

Peripherals often expose control and status registers at fixed addresses. Ordinary
loads and stores are insufficient because the compiler may remove or combine them.
Volatile access says the access itself is observable.

```rust
use core::ptr::NonNull;

#[derive(Clone, Copy)]
struct Register32 {
    address: NonNull<u32>,
}

impl Register32 {
    /// # Safety
    ///
    /// `address` must be aligned, valid for 32-bit volatile accesses for the
    /// lifetime of this value, and refer to a register whose access rules permit
    /// reads and writes performed through this wrapper.
    unsafe fn new(address: usize) -> Option<Self> {
        if address % core::mem::align_of::<u32>() != 0 {
            return None;
        }

        NonNull::new(address as *mut u32).map(|address| Self { address })
    }

    fn read(&self) -> u32 {
        // SAFETY: Construction requires a valid readable MMIO register.
        unsafe { self.address.as_ptr().read_volatile() }
    }

    fn write(&mut self, value: u32) {
        // SAFETY: Construction requires a valid writable MMIO register.
        unsafe { self.address.as_ptr().write_volatile(value) }
    }
}
```

The wrapper is only honest if its constructor's contract matches the hardware manual.

| Volatile does **not** guarantee | Separate requirement                            |
| ------------------------------- | ----------------------------------------------- |
| Data-race safety                | Atomics, critical sections, or exclusive access |
| Multi-register atomicity        | Device-specific transaction protocol            |
| CPU/device ordering             | Correct memory barriers/fences                  |
| Side-effect-free reads          | Register semantics from the hardware manual     |
| Peripheral ownership            | A higher-level ownership policy                 |

Avoid implementing `Send` or `Sync` automatically for MMIO wrappers. Decide those
properties from the hardware sharing rules.

### 5. Interrupts and Shared State

An interrupt can occur between ordinary instructions, so code shared with an interrupt
handler is concurrent code.

| Interrupt concern   | Preferred approach                              |
| ------------------- | ----------------------------------------------- |
| Small shared values | Atomics with a justified ordering               |
| Compound state      | Target-provided critical section                |
| Data handoff        | Interrupt-safe SPSC queue                       |
| Handler duration    | Record minimal work and return quickly          |
| Forbidden work      | Avoid blocking, allocation, and unbounded loops |

```rust
use core::sync::atomic::{AtomicBool, Ordering};

static SAMPLE_READY: AtomicBool = AtomicBool::new(false);

fn interrupt_handler() {
    SAMPLE_READY.store(true, Ordering::Release);
}

fn main_loop_step() {
    if SAMPLE_READY.swap(false, Ordering::Acquire) {
        process_sample();
    }
}

fn process_sample() {
    // Ordinary application work.
}
```

Acquire/release here publishes work associated with the flag, but the exact design must
match how the sample data itself is stored. `volatile` is not a substitute for atomics.

### 6. What `unsafe` Actually Permits

Unsafe Rust enables a small set of operations that the compiler cannot prove safe:

| Unsafe capability       | Proof obligation moved to the programmer           |
| ----------------------- | -------------------------------------------------- |
| Dereference raw pointer | Validity, alignment, initialization, and aliasing  |
| Mutable static state    | Synchronization and exclusive access               |
| Access union field      | Active representation is valid for the field       |
| Call unsafe function    | Every documented precondition is satisfied         |
| Implement unsafe trait  | The implementation upholds the trait-wide contract |

`unsafe` does not disable the borrow checker or turn Rust into C. It transfers proof
obligations from the compiler to the programmer.

A good unsafe abstraction has:

1. a documented safety contract;
2. validation performed in safe code where possible;
3. the smallest practical unsafe block;
4. a safe interface that cannot violate the invariant;
5. tests, fuzzing, and dynamic checking appropriate to its risk.

```rust
/// Views `len` bytes beginning at `pointer`.
///
/// # Safety
///
/// `pointer` must be non-null and aligned for `u8`, reference `len`
/// initialized bytes in one allocation, and remain valid for `'a`.
unsafe fn bytes_from_raw<'a>(pointer: *const u8, len: usize) -> &'a [u8] {
    // SAFETY: The caller promises every requirement of `from_raw_parts`.
    unsafe { core::slice::from_raw_parts(pointer, len) }
}
```

Use an explicit unsafe block even inside an `unsafe fn`; the function declares what the
caller must prove, while the block marks where the implementation relies on that proof.
The `unsafe_op_in_unsafe_fn` lint helps enforce this distinction.

### 7. Unsafe Review Questions

| Review area      | Question                                                 |
| ---------------- | -------------------------------------------------------- |
| Invariant        | What exact fact makes the operation memory safe?         |
| Ownership        | Who owns the allocation or device?                       |
| Lifetime         | How long does it remain valid?                           |
| Pointer validity | Is it aligned and within one allocation?                 |
| Initialization   | Are bytes valid before the typed read?                   |
| Aliasing         | Can mutable access overlap?                              |
| Concurrency      | Can a thread, signal, interrupt, or FFI call mutate it?  |
| Layout           | Is representation guaranteed by `repr` or a wire format? |
| Arithmetic       | Can an offset/length wrap before validation?             |
| Unwinding        | Can panic leave foreign code or hardware invalid?        |
| Safe API         | Can any safe caller reach undefined behavior?            |

Comments should explain the proof, not restate the operation:

```rust
// Weak:
// SAFETY: dereference pointer.

// Useful:
// SAFETY: `cursor < end` was checked above; both pointers are derived
// from the same allocation, and the parser holds the only mutable borrow.
```

### 8. Chromium as a Large Mixed-Language Case Study

Chromium is a large C++ codebase with its own build, dependency, test, and security
policies. Rust must integrate into that environment instead of assuming a Cargo-only
project.

**🔑 Key ideas from the Comprehensive Rust Chromium material:**

- Chromium uses GN and Ninja; a whole Rust crate is a compilation unit.
- A `rust_static_library` rule declares its crate root and complete source list.
- The Chromium rule supplies support for features, tests, and C++ interoperability.
- Unsafe Rust is forbidden for a Rust static library by default and requires an
  explicit `allow_unsafe = true` decision.
- C++/Rust boundaries should use generated, type-aware bridges where possible.
- Third-party crates are reviewed as source dependencies, not treated as opaque
  packages.

Conceptual GN target:

```gn
import("//build/rust/rust_static_library.gni")

rust_static_library("parser_core") {
  crate_root = "lib.rs"
  sources = [
    "lib.rs",
    "decoder.rs",
  ]
}
```

Large-repository implications:

- List inputs so incremental builds know exactly what invalidates a target.
- Avoid crates whose build scripts secretly compile C++, run bindgen, or perform
  arbitrary actions that the repository build graph cannot represent.
- Audit transitive dependencies, procedural macros, build scripts, maintenance state,
  filesystem/network access, and unsafe code.
- Keep ownership clear at C++/Rust boundaries.
- Do not pass language-native containers or exceptions across an unstable ABI.
- Translate errors deliberately; do not unwind a Rust panic through C++.
- Test the bridge from both sides and include malformed inputs.

### 9. Lessons for Your Own Language

Bare metal, unsafe code, and Chromium integration all expose runtime assumptions:

| Concern             | Your language must define                                       |
| ------------------- | --------------------------------------------------------------- |
| Startup             | Entry point, stack, globals, constructors, termination          |
| Memory              | Allocation, alignment, initialization, aliasing, lifetime       |
| Hardware I/O        | Volatile operations, barriers, register ownership               |
| Interrupts/threads  | Synchronization and memory model                                |
| Foreign code        | ABI, layout, ownership, errors, unwinding                       |
| Build integration   | Compilation unit, inputs, generated files, dependency policy    |
| Unsafe escape hatch | Which invariants become the programmer's responsibility         |
| Packages            | Trust, auditing, reproducibility, version and feature selection |

A small language can begin with a hosted interpreter, but writing these contracts early
makes later native compilation, FFI, embedded targets, and reverse engineering much
easier to reason about.

### 10. Embedded Rust Book Versus the Embedonomicon

The [Embedded Rust Book](https://docs.rust-embedded.org/book/) focuses on practical
embedded development and safe abstractions. The
[Embedonomicon](https://docs.rust-embedded.org/embedonomicon/) goes underneath runtime
crates and constructs a `#![no_std]` Cortex-M application from the linker and ABI upward.

| Resource            | Best used for                                        |
| ------------------- | ---------------------------------------------------- |
| Embedded Rust Book  | Setup, HALs, portability, concurrency, static safety |
| Embedonomicon       | Reset, linker scripts, symbols, vector tables, ABI   |
| Device/PAC docs     | Exact register addresses and bit meanings            |
| Architecture manual | Exceptions, instruction behavior, memory ordering    |
| Board schematic     | Pin routing, clocks, power, and attached components  |

> 🧠 **Layering rule:** an embedded abstraction is trustworthy only when its type-level
> contract agrees with the linker layout, architecture manual, and device register
> behavior underneath it.

### 11. Linker Scripts Are Part of the Program

On a microcontroller, the executable must place bytes where the hardware expects them.
A linker script normally defines memory regions and output sections:

```ld
MEMORY
{
  FLASH : ORIGIN = 0x00000000, LENGTH = 256K
  RAM   : ORIGIN = 0x20000000, LENGTH = 64K
}

ENTRY(Reset);

SECTIONS
{
  .vector_table ORIGIN(FLASH) :
  {
    LONG(ORIGIN(RAM) + LENGTH(RAM));
    KEEP(*(.vector_table.reset_vector));
  } > FLASH

  .text :
  {
    *(.text .text.*);
  } > FLASH
}
```

| Linker-script element | Purpose                                                 |
| --------------------- | ------------------------------------------------------- |
| `MEMORY`              | Names address ranges and their sizes                    |
| `ENTRY(symbol)`       | Selects the executable entry point                      |
| `SECTIONS`            | Maps input sections into output sections                |
| `> FLASH` / `> RAM`   | Selects the destination memory region                   |
| `KEEP(...)`           | Prevents garbage collection of hardware-referenced data |
| `EXTERN(symbol)`      | Forces the linker to search for a required symbol       |
| `/DISCARD/`           | Removes sections the runtime deliberately does not use  |

Hardware can reference a vector-table entry without a normal machine-code call. That
means the linker may see no ordinary reference; `KEEP` or an equivalent mechanism
prevents the required entry from being discarded.

### 12. Reset and Vector-Table Contracts

For Cortex-M, the first vector-table entries include the initial stack pointer and reset
handler. The reset handler has no caller to return to, so its return type is `!`.

```rust
#![no_std]

#[unsafe(no_mangle)]
pub unsafe extern "C" fn Reset() -> ! {
    // Copy initialized data, zero .bss, initialize runtime state,
    // then call the application entry point.
    loop {
        core::hint::spin_loop();
    }
}

#[unsafe(link_section = ".vector_table.reset_vector")]
#[unsafe(no_mangle)]
pub static RESET_VECTOR: unsafe extern "C" fn() -> ! = Reset;
```

| Attribute                         | Required proof                                     |
| --------------------------------- | -------------------------------------------------- |
| `#[unsafe(no_mangle)]`            | The global symbol name cannot collide              |
| `#[unsafe(export_name = "...")]`  | The chosen exported name is globally unique        |
| `#[unsafe(link_section = "...")]` | The item is valid in the destination memory region |
| `extern "C"`                      | The hardware/foreign caller follows that ABI       |
| `-> !`                            | Control never returns to a nonexistent caller      |

The Rust ABI is not a stable hardware boundary. Use the architecture/runtime's required
ABI and inspect the final ELF, not merely the source.

### 13. Exception and Interrupt Vector Tables

When an exception occurs, the processor suspends the current flow, saves architectural
state according to its rules, and selects a handler through a vector table.

```text
external/internal event
    ↓ hardware exception entry
save required state
    ↓ vector-table lookup
run handler
    ↓ exception return
restore interrupted state
```

| Vector-table invariant | Requirement                                               |
| ---------------------- | --------------------------------------------------------- |
| Placement              | Table begins at the architecture's expected address       |
| Entry order            | Each slot corresponds to the documented exception         |
| Validity               | Every usable slot holds a correctly typed handler address |
| Defaults               | Unimplemented handlers enter a safe diagnostic halt       |
| Override               | Application handlers replace defaults at link time        |
| Retention              | Link-time garbage collection cannot remove the table      |

Compile-time defaults can be supplied in the linker script and overridden by an
application-defined symbol. This gives static customization without a runtime lookup.

### 14. Typestate for Peripheral State Machines

The Embedded Rust Book models peripherals as state machines. Consuming a value during a
transition ensures the old state cannot still be used.

```rust
use core::marker::PhantomData;

struct Disabled;
struct Input;
struct Output;

struct Pin<State> {
    number: u8,
    _state: PhantomData<State>,
}

impl Pin<Disabled> {
    fn into_input(self) -> Pin<Input> {
        configure_as_input(self.number);
        Pin {
            number: self.number,
            _state: PhantomData,
        }
    }

    fn into_output(self) -> Pin<Output> {
        configure_as_output(self.number);
        Pin {
            number: self.number,
            _state: PhantomData,
        }
    }
}

impl Pin<Output> {
    fn set_high(&mut self) {
        write_output_high(self.number);
    }
}

fn configure_as_input(_pin: u8) {}
fn configure_as_output(_pin: u8) {}
fn write_output_high(_pin: u8) {}
```

`Pin<Input>` has no `set_high` method, so that invalid operation fails at compile time.
The marker types are zero-sized and normally disappear after optimization.

| Typestate benefit        | Cost/trade-off                                       |
| ------------------------ | ---------------------------------------------------- |
| Illegal calls disappear  | More generic types and `impl` blocks                 |
| Transitions are explicit | State changes may consume and return values          |
| No runtime tag needed    | Dynamic state may require an enum instead            |
| Ownership is visible     | Shared peripherals need an intentional sharing model |

Use typestate when the state graph is small and correctness-critical. Use a runtime enum
when states genuinely arrive dynamically or must live in one heterogeneous collection.

### 15. HAL Portability

`embedded-hal`-style traits separate applications and device drivers from a specific
microcontroller implementation:

```text
application
    ↓ generic driver
embedded HAL traits
    ↓ chip-family HAL
peripheral access crate
    ↓ volatile registers
hardware
```

| Layer                   | Owns                                           |
| ----------------------- | ---------------------------------------------- |
| Application             | Product behavior                               |
| Driver                  | Device protocol, such as a sensor over SPI     |
| HAL trait               | Portable capability contract                   |
| Chip HAL                | Pins, clocks, buses, DMA, and peripheral setup |
| Peripheral access crate | Register-level representation                  |
| Board support package   | Board-specific pin/component wiring            |

Generic drivers should depend on the smallest capability trait they need. Avoid leaking
one board's concrete pin or peripheral types into reusable application logic.

### 16. Embedded Concurrency Choices

Interrupt handlers are concurrent with the main loop. A `static mut` increment is not
automatically atomic: it may compile into load → modify → store and lose an interrupt's
update.

| Environment            | Appropriate synchronization                             |
| ---------------------- | ------------------------------------------------------- |
| No interrupts/one loop | Ordinary exclusive ownership                            |
| Main + interrupts      | Atomics, interrupt-safe queue, or critical section      |
| DMA + CPU              | Ownership transfer plus required memory barriers        |
| Multiple cores         | SMP-safe atomics/locks; interrupt masking is not enough |
| Real-time tasks        | Priority-aware framework and bounded operations         |

`Send` means a value can safely move between execution contexts; `Sync` means a shared
reference can be used safely from multiple contexts. Treat an interrupt handler as a
separate execution context when reasoning about shared state.

Critical sections that merely disable interrupts on one core do not protect against
another core. The proof must match the hardware topology.

### 17. Inspect the Artifact, Not Only the Source

| Question                        | Useful inspection                                 |
| ------------------------------- | ------------------------------------------------- |
| Is the entry point correct?     | ELF header and symbol table                       |
| Is the vector table retained?   | Section dump and raw bytes                        |
| Are code/data in valid regions? | Program/section headers plus linker map           |
| How large is each section?      | `size`/`cargo size`-style report                  |
| What does reset execute?        | Disassembly from the reset symbol                 |
| Did typestate disappear?        | Compare optimized assembly/code size              |
| Does it boot without hardware?  | QEMU or another architecture-appropriate emulator |

Keep the linker map, target specification, disassembly, and exact firmware hash with a
release. They are part of the evidence needed to reproduce a low-level bug.

---

## 🧵 Concurrency, Atomics & Memory Models

Concurrency adds a second ordering problem. A compiler already reasons about the order
of expressions; concurrent code must also reason about what other threads are allowed to
observe.

> 🧵 **Concurrency rule:** shared state is only understandable when **ownership,
> synchronization, and ordering** are all explicit.

### 1. Processes, Threads, Tasks, and Events

| Unit       | Memory relationship                         | Scheduled by           |
| ---------- | ------------------------------------------- | ---------------------- |
| Process    | Usually isolated address space              | Operating system       |
| Thread     | Shares process memory                       | Operating system       |
| Async task | Shares process memory, cooperatively yields | Async runtime          |
| Actor      | Communicates through messages               | Runtime or application |

**Concurrency** means multiple activities make progress over overlapping time.
**Parallelism** means work literally executes at the same instant on different cores or
processors.

### 2. The Core Shared-State Problem

A source expression that looks like one update:

```text
counter = counter + 1
```

normally contains several lower-level actions:

```text
load counter
add 1
store counter
```

Two threads can interleave those actions and lose an update. This is a **data race**
when unsynchronized conflicting accesses violate the language's memory model.

### 3. Synchronization Tools

| Tool                   | Best mental model                                    |
| ---------------------- | ---------------------------------------------------- |
| **Mutex**              | One owner of protected state at a time               |
| **Read/write lock**    | Many readers or one writer                           |
| **Condition variable** | Sleep until protected state may have changed         |
| **Semaphore**          | Finite pool of permits                               |
| **Channel**            | Transfer values/events between participants          |
| **Atomic**             | Indivisible primitive operation with chosen ordering |
| **Barrier**            | Wait for every participant to reach one phase        |

Choose the tool based on the invariant, not on familiarity.

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (send, receive) = mpsc::channel();

    let worker = thread::spawn(move || {
        let result = (1..=100).sum::<u64>();
        send.send(result).expect("receiver should still exist");
    });

    let result = receive.recv().expect("worker should send a result");
    worker.join().expect("worker should not panic");
    assert_eq!(result, 5050);
}
```

Message passing can simplify ownership: the sender transfers a value instead of giving
multiple threads mutable access to it.

### 4. Atomics and Ordering

An atomic operation prevents tearing, but memory ordering controls how operations around
it may be observed.

| Ordering  | Mental model                                                     |
| --------- | ---------------------------------------------------------------- |
| `Relaxed` | Atomicity only; no cross-thread ordering guarantee               |
| `Acquire` | Later operations cannot move before the acquire                  |
| `Release` | Earlier operations cannot move after the release                 |
| `AcqRel`  | Acquire + release for a read-modify-write operation              |
| `SeqCst`  | Strongest common ordering; one global order for these operations |

Use the weakest ordering only when you can explain the proof. A mutex or channel is
often clearer and fast enough.

```rust
use std::sync::atomic::{AtomicBool, Ordering};

static READY: AtomicBool = AtomicBool::new(false);

// Producer publishes ordinary data first, then:
READY.store(true, Ordering::Release);

// Consumer waits, then may observe data published before the release:
while !READY.load(Ordering::Acquire) {
    std::hint::spin_loop();
}
```

This fragment illustrates ordering but is not a complete queue. Correct lock-free
structures also need careful lifetime, progress, and reclamation design.

### 5. Compiler Implications

A language implementation must define:

| Memory-model topic     | Required language rule                           |
| ---------------------- | ------------------------------------------------ |
| Atomic operations      | Which reads, writes, and updates are indivisible |
| Data race              | Which conflicting accesses constitute one        |
| Race semantics         | Error, undefined behavior, or defined outcome    |
| Happens-before         | Which synchronization creates ordering           |
| Transfer               | Which values may move between threads            |
| Shared mutation        | Whether shared references permit mutation        |
| Thread-local lifecycle | Initialization and destructor behavior           |

Optimizers rely on these rules. If the language declares data races invalid, the
optimizer may assume they never occur and transform code accordingly.

### 6. Concurrency Failure Modes

| Failure mode           | Symptom                                               |
| ---------------------- | ----------------------------------------------------- |
| **Deadlock**           | Participants wait forever in a dependency cycle       |
| **Livelock**           | Work continues but no useful progress occurs          |
| **Starvation**         | One participant repeatedly loses resource access      |
| **Priority inversion** | High-priority work waits on lower-priority work       |
| **Lost wakeup**        | Notification occurs before the waiter observes it     |
| **ABA problem**        | A→B→A fools a compare-and-swap equality check         |
| **False sharing**      | Independent data causes needless cache-line coherence |

Debug concurrent systems with event traces, stable IDs, timeouts, and explicit state
transitions. Adding print statements can change timing and hide the bug.

---

## 🏢 System Design Patterns in Rust

[DesignDeck](https://deckly.dev/designdeck/) presents system-design decisions as a
connected set of trade-offs: traffic, data, consistency, caching, partitioning,
messaging, and failure handling. This section translates several of those decisions
into Rust-shaped interfaces.

> 🏗️ **Design principle:** begin with **quantities, guarantees, and failure behavior**;
> only then choose components.

### 1. Start with Quantities and Guarantees

Before drawing services and databases, write down:

| Design input       | Questions to answer                                      |
| ------------------ | -------------------------------------------------------- |
| Traffic            | Peak reads and writes per second?                        |
| Payloads           | Typical and maximum request/response sizes?              |
| Storage            | Daily growth and retention period?                       |
| Latency            | Acceptable median and tail latency?                      |
| Reliability        | Availability, recovery time, and recovery point targets? |
| Consistency        | What does each operation guarantee?                      |
| Message behavior   | Is loss, duplication, or reordering acceptable?          |
| Security           | Where are trust and authorization boundaries?            |
| Dependency failure | What happens when a dependency is slow or unavailable?   |

Useful rough estimates:

```text
bandwidth        ≈ requests/second × bytes/request
daily storage    ≈ writes/second × bytes/write × 86,400
concurrent work  ≈ arrival rate × average service time
```

These are estimates, not capacity promises. Validate them with measurement and leave
headroom for bursts, retries, replication, metadata, indexes, and uneven partitions.

### 2. Cache-Aside

In cache-aside, the application checks the cache, loads a miss from the source of truth,
and then fills the cache. The cache does not independently query storage.

```text
read:
    cache hit  ──→ return
    cache miss ──→ read database ──→ populate cache ──→ return

write:
    update database ──→ invalidate or update cache
```

A small interface keeps policy separate from storage:

```rust
trait KeyValue<K, V> {
    type Error;

    fn get(&self, key: &K) -> Result<Option<V>, Self::Error>;
    fn put(&mut self, key: K, value: V) -> Result<(), Self::Error>;
}

fn cache_aside<K, V, E>(
    cache: &mut impl KeyValue<K, V, Error = E>,
    source: &impl KeyValue<K, V, Error = E>,
    key: &K,
) -> Result<Option<V>, E>
where
    K: Clone,
    V: Clone,
{
    if let Some(value) = cache.get(key)? {
        return Ok(Some(value));
    }

    let Some(value) = source.get(key)? else {
        return Ok(None);
    };

    cache.put(key.clone(), value.clone())?;
    Ok(Some(value))
}
```

| Cache decision   | Question                                                  |
| ---------------- | --------------------------------------------------------- |
| Staleness        | How long may an old value remain visible?                 |
| Negative caching | Should missing values be cached?                          |
| Stampede control | What protects a hot key during a miss?                    |
| Fill failure     | Does cache failure fail the request or only reduce speed? |
| Write policy     | Write-through, write-back, or invalidation?               |
| Eviction         | LRU, LFU, FIFO, or a domain-specific policy?              |

The database remains the source of truth unless the design explicitly chooses a
different consistency model.

### 3. Token-Bucket Rate Limiting

A token bucket allows controlled bursts while enforcing a long-term rate. Tokens refill
over time up to a fixed capacity; each accepted operation spends tokens.

```rust
use std::time::Instant;

#[derive(Debug)]
struct TokenBucket {
    capacity: f64,
    available: f64,
    refill_per_second: f64,
    last_update: Instant,
}

impl TokenBucket {
    fn new(capacity: u32, refill_per_second: f64, now: Instant) -> Option<Self> {
        if capacity == 0 || !refill_per_second.is_finite() || refill_per_second <= 0.0 {
            return None;
        }

        Some(Self {
            capacity: f64::from(capacity),
            available: f64::from(capacity),
            refill_per_second,
            last_update: now,
        })
    }

    fn try_take(&mut self, amount: u32, now: Instant) -> bool {
        let elapsed = now.saturating_duration_since(self.last_update);
        self.available = (self.available + elapsed.as_secs_f64() * self.refill_per_second)
            .min(self.capacity);
        self.last_update = now;

        let amount = f64::from(amount);
        if amount > self.available {
            return false;
        }

        self.available -= amount;
        true
    }
}
```

| Rate-limit decision | Choices to define                                 |
| ------------------- | ------------------------------------------------- |
| Identity            | User, IP, token, tenant, endpoint, or resource    |
| Scope               | Per process or coordinated across replicas        |
| Store failure       | Fail open, fail closed, or use a local fallback   |
| Over-limit action   | Reject, queue, or degrade                         |
| Client feedback     | Retry timing, remaining quota, and reset metadata |

A leaky-bucket queue smooths output at a regular rate. It is useful when work can wait;
a token bucket is useful when short bursts are legitimate.

### 4. Circuit Breakers, Timeouts, and Backoff

A circuit breaker prevents repeated calls to a dependency that is already failing:

```rust
use std::time::Instant;

#[derive(Clone, Copy, Debug)]
enum CircuitState {
    Closed { consecutive_failures: u32 },
    Open { retry_after: Instant },
    HalfOpen,
}
```

State transitions:

```text
Closed --failure threshold--> Open
Open   --cooldown elapsed---> HalfOpen
HalfOpen --probe succeeds---> Closed
HalfOpen --probe fails------> Open
```

Use the patterns together:

| Resilience pattern      | Responsibility                                  |
| ----------------------- | ----------------------------------------------- |
| **Timeout**             | Limit one attempt                               |
| **Retry budget**        | Limit total extra work                          |
| **Exponential backoff** | Increase spacing between attempts               |
| **Jitter**              | Prevent synchronized retry storms               |
| **Circuit breaker**     | Stop calls during sustained dependency failure  |
| **Load shedding**       | Reject low-priority work before global collapse |

Retry only operations that are safe to repeat, or attach an idempotency key. Never let
every layer retry independently without a shared budget; multiplicative retries can
turn a small outage into overload.

### 5. Delivery Is Not Processing

Networks and processes can fail between any two observable steps. Messages may be
delayed, duplicated, dropped, or reordered.

| Delivery model   | What the consumer must expect                     |
| ---------------- | ------------------------------------------------- |
| At most once     | Loss is possible; duplicates are suppressed       |
| At least once    | Duplicates are possible; retry covers some losses |
| Effectively once | Deduplication makes repeated processing harmless  |

Exactly-once network delivery is not a general guarantee. Systems usually approach
exactly-once **effects** through idempotency, durable deduplication, and atomic state
changes.

```rust
use std::collections::HashSet;

#[derive(Clone, Copy, Debug, Eq, Hash, PartialEq)]
struct EventId(u128);

#[derive(Debug)]
struct Event {
    id: EventId,
    delta: i64,
}

fn apply_once(
    event: &Event,
    processed: &mut HashSet<EventId>,
    total: &mut i64,
) -> Result<bool, &'static str> {
    if processed.contains(&event.id) {
        return Ok(false);
    }

    let next = total.checked_add(event.delta).ok_or("total overflow")?;
    *total = next;
    processed.insert(event.id);
    Ok(true)
}
```

This in-memory example explains the invariant but is not crash safe. In a real system,
the deduplication record and business update must be committed atomically in durable
storage, or a crash can occur between them.

An **event log** retains ordered records for replay and multiple consumers. A **queue**
typically distributes work so one consumer handles each message. Choose based on replay,
fan-out, ordering, and retention needs.

### 6. Consistent Hashing and Partition Movement

Ordinary `hash(key) % node_count` remaps most keys when the number of nodes changes.
Consistent hashing places keys and nodes in a hash space so adding or removing a node
moves a smaller portion of the data.

```text
hash ring:
0 ───── node A ───── node B ───── node C ───── max
          ↑ key x                  ↑ key y
```

Virtual nodes give each physical node multiple positions, which can improve balance.
Jump consistent hashing is another option when a compact computation is preferable to
maintaining a ring.

| Partition concern | Required decision                                         |
| ----------------- | --------------------------------------------------------- |
| Hash stability    | Is the mapping stable across versions and languages?      |
| Hot keys          | How are skewed workloads isolated or replicated?          |
| Replication       | How many owners does each partition have?                 |
| Membership        | How are node changes proposed and observed?               |
| Rebalancing       | Can reads and writes continue while data moves?           |
| Recovery          | How are partially moved partitions detected and repaired? |

If the key is a compiler module, bytecode package, or analysis artifact, stable
partitioning can also support distributed builds and content-addressed caches.

### 7. Consistency Is Per Operation

Do not label an entire system merely "strong" or "eventual." State what each operation
guarantees:

| Consistency guarantee        | Meaning                                                |
| ---------------------------- | ------------------------------------------------------ |
| Read-your-writes             | A client observes its own completed writes             |
| Monotonic reads              | A client does not move backward to older state         |
| Causal ordering              | Causally related events are observed in order          |
| Linearizable compare-and-set | The update behaves as one atomic real-time operation   |
| Bounded staleness            | Reads lag by no more than a defined time/version bound |
| Eventual convergence         | Replicas agree once updates and failures stop          |

The CAP trade-off matters during a network partition: a distributed system cannot
simultaneously guarantee both availability for every request and linearizable
consistency across the partition. Different operations may make different choices.

### 8. System-Design Review Checklist

| Area            | Review criterion                                                |
| --------------- | --------------------------------------------------------------- |
| Data ownership  | Source of truth and derived copies are explicit                 |
| Remote calls    | Every call has a timeout                                        |
| Resource bounds | Queues, retries, bodies, fan-out, and concurrency are capped    |
| Overload        | Degraded and rejected behavior is defined                       |
| Idempotency     | Repeated messages are safe or durably detected                  |
| Versioning      | Wire formats and stored schemas carry versions                  |
| Deployment      | Migration, rollback, and mixed-version behavior are defined     |
| Observability   | Saturation, errors, tail latency, and dropped work are measured |
| Privacy         | Logs and cache keys exclude secrets and sensitive data          |
| Failure testing | Slowness and partial failure are tested                         |
| Recovery        | Recovery point and recovery time objectives are stated          |
| Simplicity      | Operators can explain the important failure modes               |

---

## 🌐 Networking, Protocols & Wire Formats

Networking is low-level I/O plus a shared language. A protocol has **syntax** (frame
layout), **semantics** (what messages mean), and **state** (which messages are valid
now). That makes protocol design closely related to compiler design.

> 🌐 **Mental model:** a network protocol is a language spoken over an unreliable,
> partial-I/O boundary.

### 1. Layered Mental Model

```text
Application protocol   your messages, grammar, state machine
Transport              TCP stream or UDP datagrams
Network                IP addressing and routing
Link                   local network frames
Physical               electrical/radio/optical signals
```

An application usually reads and writes through a socket; the OS networking stack
handles the lower layers.

| Concept          | TCP                                   | UDP                                    |
| ---------------- | ------------------------------------- | -------------------------------------- |
| Abstraction      | Ordered byte stream                   | Individual datagrams                   |
| Delivery         | Retransmitted while connection lives  | Best effort                            |
| Boundaries       | Not preserved                         | Preserved per datagram                 |
| Connection state | Yes                                   | No transport connection                |
| Typical use      | Web, shells, databases, file transfer | DNS, telemetry, games, real-time media |

TCP does **not** deliver "messages." One `write` may be split across many reads, and
several writes may arrive in one read. Your protocol must provide framing.

### 2. Common Framing Strategies

| Strategy          | Example                | Trade-off                       |
| ----------------- | ---------------------- | ------------------------------- |
| Fixed size        | exactly 32 bytes       | Simple, wastes space            |
| Delimiter         | line ending `\n`       | Human-readable, escaping needed |
| Length prefix     | `u32 length` + payload | Efficient, must bound length    |
| Type-length-value | tag + length + value   | Extensible                      |
| Self-describing   | JSON/CBOR-like value   | Flexible, more overhead         |

Example binary frame:

```text
0               1               2               3
+---------------+---------------+---------------+---------------+
| version (u8)  | kind (u8)     | payload length (u16, BE)       |
+---------------+---------------+---------------+---------------+
| payload ...                                                   |
+---------------------------------------------------------------+
```

```rust
#[derive(Debug)]
struct Frame<'a> {
    version: u8,
    kind: u8,
    payload: &'a [u8],
}

fn decode_frame(bytes: &[u8]) -> Result<Frame<'_>, &'static str> {
    let header = bytes.get(..4).ok_or("short header")?;
    let length = u16::from_be_bytes([header[2], header[3]]) as usize;
    if length > 4096 {
        return Err("payload too large");
    }
    let payload = bytes.get(4..4 + length).ok_or("short payload")?;
    if bytes.len() != 4 + length {
        return Err("trailing data");
    }
    Ok(Frame {
        version: header[0],
        kind: header[1],
        payload,
    })
}
```

### 3. Reading a Length-Prefixed TCP Frame

`read_exact` loops until the requested buffer is filled or an error occurs. It is
appropriate for a fixed header and a validated payload length.

```rust
use std::io::{self, Read};
use std::net::TcpStream;

const MAX_FRAME: usize = 64 * 1024;

fn read_frame(stream: &mut TcpStream) -> io::Result<Vec<u8>> {
    let mut header = [0u8; 4];
    stream.read_exact(&mut header)?;

    let length = u32::from_be_bytes(header) as usize;
    if length > MAX_FRAME {
        return Err(io::Error::new(
            io::ErrorKind::InvalidData,
            "frame exceeds limit",
        ));
    }

    let mut payload = vec![0u8; length];
    stream.read_exact(&mut payload)?;
    Ok(payload)
}
```

Real services should also use timeouts, connection limits, authentication when needed,
and resource budgets. Portable code should not assume every operating system reports
socket timeouts with exactly the same error kind.

### 4. A Safe Loopback Protocol Lab

Bind test services to loopback so they are not exposed to the local network:

```rust
use std::io::{self, Read, Write};
use std::net::{TcpListener, TcpStream};

fn handle(mut stream: TcpStream) -> io::Result<()> {
    let mut request = [0u8; 4];
    stream.read_exact(&mut request)?;

    let response = match &request {
        b"PING" => b"PONG",
        _ => b"NOPE",
    };

    stream.write_all(response)?;
    Ok(())
}

fn main() -> io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:0")?;
    println!("listening on {}", listener.local_addr()?);

    for incoming in listener.incoming() {
        match incoming {
            Ok(stream) => {
                if let Err(error) = handle(stream) {
                    eprintln!("connection error: {error}");
                }
            }
            Err(error) => eprintln!("accept error: {error}"),
        }
    }
    Ok(())
}
```

| Lab concept        | What the example demonstrates                     |
| ------------------ | ------------------------------------------------- |
| Endpoint           | An address combined with a port                   |
| Server lifecycle   | Listen, accept, read, write, close                |
| Partial I/O        | Use helpers that complete or report failure       |
| Protocol semantics | Map a request to a defined response               |
| Error isolation    | One connection failure need not stop the listener |

### 5. Socket Lifecycle and Client/Server Roles

[Beej's Guide to Network Programming](https://beej.us/guide/bgnet/html/) frames a socket
as an operating-system handle used for network I/O. On Unix-like systems it is a file
descriptor, so familiar resource and blocking-I/O ideas apply.

Typical TCP server:

```text
resolve local address
    ↓
socket → bind → listen → accept
                         ↓
                    read/write
                         ↓
                       close
```

Typical TCP client:

```text
resolve remote address
    ↓
socket → connect → read/write → close
```

Rust wraps the resource in `TcpStream`, `TcpListener`, and `UdpSocket`. Their destructors
close the underlying socket when ownership ends, but protocol shutdown may still need an
explicit `shutdown`.

```rust
use std::io::{self, Read, Write};
use std::net::{Shutdown, TcpStream};
use std::time::Duration;

fn request(address: &str, bytes: &[u8]) -> io::Result<Vec<u8>> {
    let mut stream = TcpStream::connect(address)?;
    stream.set_read_timeout(Some(Duration::from_secs(3)))?;
    stream.set_write_timeout(Some(Duration::from_secs(3)))?;

    stream.write_all(bytes)?;
    stream.shutdown(Shutdown::Write)?;

    let mut response = Vec::new();
    stream.take(64 * 1024).read_to_end(&mut response)?;
    Ok(response)
}
```

`shutdown(Write)` sends an end-of-stream signal for the writing direction while leaving
the reading direction open. This is useful only when the application protocol defines
end-of-stream as the request boundary. Many protocols keep the connection open and use
lengths or delimiters instead.

### 6. Names, Addresses, and Ports

An endpoint combines an IP address with a transport port:

```text
IPv4 endpoint: 192.0.2.10:443
IPv6 endpoint: [2001:db8::10]:443
```

| Network name/value | Role                                           |
| ------------------ | ---------------------------------------------- |
| IP address         | Identifies an interface or routing destination |
| Port               | Identifies a transport endpoint on a host      |
| Host name          | Resolves to one or more IPv4/IPv6 addresses    |
| Client local port  | Usually assigned ephemerally by the OS         |
| Server port        | Bound to a known local address/service port    |

Do not resolve a name and then use only the first address forever. Try the returned
candidates according to policy:

```rust
use std::io;
use std::net::{TcpStream, ToSocketAddrs};
use std::time::Duration;

fn connect_any(host: &str, port: u16) -> io::Result<TcpStream> {
    let addresses = (host, port).to_socket_addrs()?;
    let timeout = Duration::from_secs(3);
    let mut last_error = None;

    for address in addresses {
        match TcpStream::connect_timeout(&address, timeout) {
            Ok(stream) => return Ok(stream),
            Err(error) => last_error = Some(error),
        }
    }

    Err(last_error.unwrap_or_else(|| {
        io::Error::new(io::ErrorKind::AddrNotAvailable, "name resolved to no addresses")
    }))
}
```

Resolution is not authentication. A successful DNS lookup says where to connect, not
whether the peer is trusted. Secure protocols still authenticate the remote identity.

### 7. Encapsulation and Byte Order

Each layer wraps the data from the layer above:

```text
[link header [IP header [TCP/UDP header [application frame]]]]
```

On receipt, the layers remove their respective headers in reverse order. This
separation lets the application use the same socket interface across different link
technologies.

Multi-byte integers need a specified wire order. Network protocols commonly use
big-endian, traditionally called **network byte order**:

```rust
let request_id = 0x1234_5678u32;
let encoded = request_id.to_be_bytes();
assert_eq!(encoded, [0x12, 0x34, 0x56, 0x78]);

let decoded = u32::from_be_bytes(encoded);
assert_eq!(decoded, request_id);
```

Never transmit an in-memory Rust struct directly:

| In-memory property     | Wire-format problem                                     |
| ---------------------- | ------------------------------------------------------- |
| Padding                | Bytes may be unstable or uninitialized                  |
| Alignment/layout       | Can vary by target and compiler settings                |
| Integer representation | May use the wrong byte order                            |
| Pointers/references    | Have no meaning in another process                      |
| Enum representation    | Is unstable without an explicit representation contract |

Serialize each field deliberately, or use a documented format and implementation.

### 8. Partial I/O, EOF, and Blocking

A successful socket read or write may transfer fewer bytes than requested.

| Operation/result | Meaning                                                  |
| ---------------- | -------------------------------------------------------- |
| `read` → `Ok(0)` | TCP end-of-stream                                        |
| `read_exact`     | Fill the buffer or report EOF/error                      |
| `write_all`      | Continue until all bytes are accepted or an error occurs |
| Timeout          | Bound waiting; does not create a message boundary        |

Blocking sockets put the calling thread to sleep until progress or an error is possible.
Non-blocking sockets instead report that the operation would block. An event loop uses
an OS readiness facility to learn which sockets may make progress.

```text
blocking model:      one waiting thread per operation or connection
readiness model:     poll/select/epoll/kqueue → operate on ready sockets
completion model:    submit I/O → receive completion later
async Rust:          futures + reactor/executor hide much of this bookkeeping
```

Readiness is a hint, not a promise that an arbitrarily large operation will finish.
Drain or fill only until the operation would block again, and keep per-connection
buffers bounded.

For TCP framing, test all of these:

```text
header split across reads
payload split across reads
header + payload + next frame in one read
peer closes halfway through a frame
declared length exceeds the limit
write accepts only a prefix
```

### 9. UDP Is Message-Oriented but Not Reliable

UDP preserves datagram boundaries, but an individual datagram may be lost, duplicated,
or reordered. If reliability matters, the application protocol must define sequence
numbers, acknowledgments, retransmission, deduplication, and congestion behavior—or use
an existing transport that already does.

```rust
use std::io;
use std::net::UdpSocket;
use std::time::Duration;

fn udp_exchange(server: &str, request: &[u8]) -> io::Result<Vec<u8>> {
    let socket = UdpSocket::bind("127.0.0.1:0")?;
    socket.connect(server)?;
    socket.set_read_timeout(Some(Duration::from_secs(1)))?;
    socket.send(request)?;

    let mut buffer = vec![0u8; 1_200];
    let received = socket.recv(&mut buffer)?;
    buffer.truncate(received);
    Ok(buffer)
}
```

This example binds to loopback for a local lab. In a real UDP protocol:

| UDP concern       | Defensive design                             |
| ----------------- | -------------------------------------------- |
| Fragmentation     | Keep datagrams within a conservative size    |
| Authenticity      | Authenticate before acting                   |
| Response matching | Use unpredictable request IDs                |
| Amplification     | Bound response size relative to request size |
| Retransmission    | Model timeout/retry as finite protocol state |

### 10. Protocol State Machines

A parser answers "is this frame shaped correctly?" A state machine answers "is this
message valid now?"

```rust
enum Session {
    AwaitHello,
    Ready { user: String },
    Closed,
}

enum Message {
    Hello { user: String },
    Data(Vec<u8>),
    Goodbye,
}

fn transition(state: Session, message: Message) -> Result<Session, &'static str> {
    match (state, message) {
        (Session::AwaitHello, Message::Hello { user }) => {
            Ok(Session::Ready { user })
        }
        (Session::Ready { user }, Message::Data(_)) => {
            // Process data, then remain ready.
            Ok(Session::Ready { user })
        }
        (Session::Ready { .. }, Message::Goodbye) => Ok(Session::Closed),
        _ => Err("message is invalid in the current state"),
    }
}
```

A compiler is also a protocol state machine:

```text
source → parsed → typed → lowered → emitted
```

### 11. Reverse Engineering an Unknown Protocol Safely

packet-analysis workflow generalizes well to your own local server, interoperability
research, and authorized captures:

1. identify the transport and endpoints;
2. capture a baseline exchange;
3. change one controllable input;
4. compare packet lengths and changed byte regions;
5. look for magic values, counters, timestamps, strings, and length prefixes;
6. test byte order and compression hypotheses;
7. infer the session state machine;
8. write a decoder that rejects malformed input;
9. validate against fresh captures you did not use to form the hypothesis.

| Observed clue                          | Plausible interpretation                       |
| -------------------------------------- | ---------------------------------------------- |
| First 2/4 bytes match remaining length | Length-prefixed framing                        |
| Many readable bytes                    | Text or lightly structured data                |
| Tiny change affects most later bytes   | Compression, encryption, or integrity data     |
| Repeated fixed prefix                  | Magic value, version, or header                |
| Monotonically changing field           | Sequence number or timestamp                   |
| Same message differs each time         | Nonce, timestamp, randomization, or encryption |

Do not assume traffic is plaintext merely because a few bytes are readable. Do not
attempt to defeat encryption or access systems without permission.

### 12. Designing a Protocol for Your Own Language

Suppose your language has a remote REPL. Separate framing, message syntax, and
semantics:

```text
Frame:
    version: u8
    message_kind: u8
    payload_length: u32 big-endian
    payload: bytes

Message kinds:
    1 = Evaluate(source_utf8)
    2 = Result(value)
    3 = Diagnostic(code, span, message)
    4 = Cancel(request_id)
```

| Protocol area       | Design requirement                               |
| ------------------- | ------------------------------------------------ |
| Versioning          | Include a version from the beginning             |
| Integers            | Define widths and byte order                     |
| Text                | Specify UTF-8 validity and normalization         |
| Resource limits     | Cap frames, strings, collections, and nesting    |
| Identifier domains  | Use newtypes for request IDs and session IDs     |
| Extensibility       | Define optional/mandatory unknown-field behavior |
| Authentication      | Authenticate before evaluating code              |
| Sandboxing          | Bound CPU, memory, and execution time            |
| Diagnostics         | Use structured fields, not scraped prose         |
| Compatibility tests | Publish input bytes with expected decoded values |

### 13. Networking Failure Modes

| Failure mode               | Safer design                                         |
| -------------------------- | ---------------------------------------------------- |
| Assume full `read`/`write` | Loop with `read_exact`/`write_all` or buffered state |
| Trust a length field       | Validate against protocol and resource limits        |
| Omit timeouts              | Bound connect, read, write, and idle time            |
| Recurse without a limit    | Enforce structural depth                             |
| Treat TCP close as a frame | Use explicit framing semantics                       |
| Parse before trust checks  | Place authentication/authorization deliberately      |
| Trust client type tags     | Validate every tag, offset, and enum value           |
| Log entire payloads        | Redact and cap diagnostic data                       |
| Use host endianness        | Specify and encode a wire byte order                 |
| Change layouts silently    | Version messages and define compatibility            |

Tests should fragment a frame at every possible byte boundary, combine several frames
into one read, truncate every field, and exercise maximum permitted sizes.

---

## 📡 Embedded Networking with `smoltcp`

[`smoltcp`](https://github.com/smoltcp-rs/smoltcp) is a standalone, event-driven TCP/IP
stack designed for bare-metal and real-time systems. It can operate without heap
allocation, making queues, socket buffers, timing, and packet work visible rather than
hiding them behind an operating-system socket API.

### 1. Stack Architecture

```text
application / language runtime
          ↕ socket send and receive buffers
SocketSet: TCP, UDP, ICMP, DHCP, DNS, raw sockets
          ↕
Interface: addressing, routes, neighbors, protocol state
          ↕
Device + RxToken / TxToken
          ↕
driver, TAP/TUN, loopback, Ethernet MAC, or radio
```

| Layer       | Owns or describes                                   |
| ----------- | --------------------------------------------------- |
| `Device`    | frame receive/transmit capability, MTU, medium      |
| `Interface` | IP addresses, routes, neighbor/protocol state       |
| `SocketSet` | heterogeneous sockets identified by stable handles  |
| socket      | protocol state plus explicitly supplied buffers     |
| application | when to poll, consume, produce, retry, and time out |

This resembles a language runtime: the device is the host boundary, the interface is
runtime-wide state, sockets are managed objects, and `poll` is the scheduler step.

### 2. Memory Is Part of the Network Contract

In hosted code, examples may provide `Vec<u8>` storage. In heapless code, the same
buffer abstractions can borrow fixed slices.

```rust
use smoltcp::socket::tcp;

static mut RX_BYTES: [u8; 1024] = [0; 1024];
static mut TX_BYTES: [u8; 1024] = [0; 1024];

// In real embedded code, obtain unique mutable access during initialization.
// The surrounding platform code must ensure these buffers are never aliased.
let socket = unsafe {
    // SAFETY: initialization runs once before interrupts/tasks can access the buffers.
    let rx = tcp::SocketBuffer::new(&mut *core::ptr::addr_of_mut!(RX_BYTES));
    let tx = tcp::SocketBuffer::new(&mut *core::ptr::addr_of_mut!(TX_BYTES));
    tcp::Socket::new(rx, tx)
};
```

Prefer linker- or platform-provided single-init cells where available; the snippet makes
the low-level ownership contract visible, not idealizes `static mut` as an everyday API.

| Fixed resource              | Behavior when exhausted                          |
| --------------------------- | ------------------------------------------------ |
| TCP receive buffer          | advertised window shrinks; application must read |
| TCP transmit buffer         | send capacity disappears until packets advance   |
| UDP packet metadata/storage | datagram send/receive can fail                   |
| neighbor cache              | entries are replaced or resolution is delayed    |
| routes/addresses            | insertion may fail at configured capacity        |
| reassembly storage          | oversized/excess fragmented packets are dropped  |

> 🧭 **Invariant:** buffer size is observable protocol behavior. Treat it as a deliberate
> capacity and backpressure decision, not an incidental allocation.

### 3. Polling Makes Scheduling Explicit

`Interface::poll(now, device, sockets)` transmits queued socket data and processes
queued device packets. `poll_at` or `poll_delay` tells an event loop when protocol timers
want attention.

```rust
use smoltcp::iface::{Interface, SocketSet};
use smoltcp::phy::Device;
use smoltcp::time::Instant;

fn drive_once(
    iface: &mut Interface,
    device: &mut impl Device,
    sockets: &mut SocketSet<'_>,
    now: Instant,
) {
    let _changed = iface.poll(now, device, sockets);

    // Inspect socket readiness and do a bounded amount of application work here.
    // Then sleep/yield until the device is ready or iface.poll_at(...) is due.
}
```

| Timing mistake                  | Consequence                                   |
| ------------------------------- | --------------------------------------------- |
| Poll too late                   | retransmissions and quality of service suffer |
| Busy-poll continuously          | wasted CPU and energy                         |
| Use a non-monotonic clock       | broken timeout/retransmission reasoning       |
| Drain unlimited ingress         | other real-time work may starve               |
| Ignore application backpressure | buffers fill and useful work is displaced     |

The current interface API warns that a full `poll()` may process an unbounded number of
queued ingress packets. In a cooperative or real-time scheduler, use
`poll_ingress_single`, `poll_egress`, and `poll_maintenance` to create explicit yield
points and bounded work.

### 4. Socket State Is a Protocol State Machine

```rust
use smoltcp::iface::{SocketHandle, SocketSet};
use smoltcp::socket::tcp;

fn service_echo(
    sockets: &mut SocketSet<'_>,
    handle: SocketHandle,
) -> Result<(), tcp::ListenError> {
    let socket = sockets.get_mut::<tcp::Socket<'_>>(handle);

    if !socket.is_open() {
        socket.listen(7000)?;
    }

    if socket.can_recv() {
        let _ = socket.recv(|bytes| {
            let consumed = bytes.len();
            // Parse or copy only what the bounded application budget permits.
            (consumed, ())
        });
    }

    Ok(())
}
```

| Readiness/state query | Meaning for application logic            |
| --------------------- | ---------------------------------------- |
| `is_open()`           | socket is not fully closed               |
| `is_active()`         | connection has active protocol state     |
| `can_recv()`          | receive would produce application data   |
| `may_recv()`          | peer may still send data in the future   |
| `can_send()`          | transmit buffer currently has capacity   |
| `may_send()`          | connection may still permit future sends |

Do not equate `can_recv() == false` with EOF. Like a VM or coroutine, a socket can be
temporarily unable to progress while remaining live.

### 5. A Networked Language Runtime

| Runtime feature       | `smoltcp` integration decision                       |
| --------------------- | ---------------------------------------------------- |
| remote REPL           | frame parser, source limit, authentication, timeout  |
| debugger protocol     | request IDs, cancellation, bounded diagnostic output |
| package transfer      | chunking, hash verification, storage budget          |
| actor/message runtime | UDP loss policy or TCP backpressure                  |
| embedded web endpoint | HTTP parser limits and one/many-connection policy    |
| telemetry             | sampling, queue eviction, reconnect/backoff          |

Keep protocol parsing independent from the socket. Feed it byte slices and test it with
fragmentation, coalescing, truncation, invalid lengths, and capacity exhaustion. Then
test the stack in a loopback or TAP-based lab you control.

### 6. Test Faults, Not Only Happy Packets

The upstream hosted examples support packet dropping, corruption, size limits, rate
limits, and PCAP capture. These are valuable language-runtime tests too:

| Injected condition | Runtime property to verify                     |
| ------------------ | ---------------------------------------------- |
| packet loss        | retry/timeout does not duplicate semantic work |
| corruption         | checksums and parsers reject damaged input     |
| tiny MTU           | fragmentation assumptions are not baked in     |
| tiny socket buffer | backpressure reaches the producer              |
| slow polling       | deadlines fail predictably                     |
| connection reset   | pending requests finish with structured errors |

Run raw-interface and fault-injection experiments only on an isolated lab interface or
network you are authorized to control.

> ➡️ **Next:** QUIC keeps UDP's user-space flexibility while adding secure connections,
> reliable streams, loss recovery, and congestion control.

---

## ⚡ QUIC Networking with Quinn

The [Quinn networking introduction](https://quinn-rs.github.io/quinn/networking-introduction.html)
compares TCP, UDP, and QUIC, then motivates QUIC through latency and head-of-line
blocking. Quinn is an async Rust implementation of QUIC.

### 1. Transport Comparison

| Property          | TCP                              | UDP                      | QUIC                                 |
| ----------------- | -------------------------------- | ------------------------ | ------------------------------------ |
| connection        | yes                              | no                       | yes                                  |
| reliability/order | one reliable ordered byte stream | neither provided         | reliable streams; optional datagrams |
| implementation    | usually kernel                   | usually kernel primitive | mainly user space over UDP           |
| security          | separate TLS                     | application-defined      | TLS integrated into the protocol     |
| multiplexing      | application protocol must add it | application-defined      | multiple independent streams         |
| migration         | connection tied to address tuple | application-defined      | connection IDs can support migration |

QUIC is not “reliable UDP.” It is a connection protocol built over UDP with its own
handshake, encryption, stream, loss-recovery, and congestion-control behavior.

### 2. Head-of-Line Blocking Has Layers

TCP exposes one ordered stream. If a packet containing earlier bytes is lost, later
bytes cannot be delivered to the application even if they belong to an unrelated
logical request.

```text
TCP connection:
request A bytes ─┐
request B bytes ─┼─ one ordered delivery sequence
request C bytes ─┘

QUIC connection:
stream A ─ independent ordered sequence
stream B ─ independent ordered sequence
stream C ─ independent ordered sequence
```

Loss on one QUIC stream does not prevent complete data on another stream from becoming
available to the application. Congestion control still couples traffic at the
connection/path level, so “independent” does not mean resource-free.

### 3. Quinn's Core Objects

```text
Endpoint
  └── Connection
      ├── bidirectional stream: SendStream + RecvStream
      ├── unidirectional SendStream
      ├── unidirectional RecvStream
      └── unreliable datagrams
```

| Object/API              | Responsibility                                  |
| ----------------------- | ----------------------------------------------- |
| `Endpoint`              | bind UDP socket, accept/connect, own QUIC state |
| `Connection`            | secure peer session and shared transport state  |
| `open_bi`/`accept_bi`   | create/accept bidirectional streams             |
| `open_uni`/`accept_uni` | create/accept one-way streams                   |
| datagram API            | bounded unreliable messages                     |
| connection stats        | observe loss, RTT, congestion, and traffic      |

### 4. A Bounded Request on an Existing Connection

```rust
use anyhow::{Context, Result};
use tokio::io::AsyncWriteExt;

const MAX_RESPONSE_BYTES: usize = 256 * 1024;

async fn request(
    connection: &quinn::Connection,
    payload: &[u8],
) -> Result<Vec<u8>> {
    let (mut send, mut receive) = connection
        .open_bi()
        .await
        .context("open QUIC stream")?;

    send.write_all(payload)
        .await
        .context("write request")?;
    send.finish().context("finish request stream")?;

    receive
        .read_to_end(MAX_RESPONSE_BYTES)
        .await
        .context("read bounded response")
}
```

The connection setup is deliberately separate because certificate trust, server name,
transport configuration, socket address, timeouts, and resumption policy belong to a
larger security contract.

### 5. Streams Need Application Framing

A QUIC stream is still a byte stream. Opening one stream per request can provide a
natural boundary, but a long-lived stream still needs explicit framing.

| Choice                   | Appropriate when                            |
| ------------------------ | ------------------------------------------- |
| one stream per request   | independent request/response exchanges      |
| long-lived framed stream | ordered session, REPL, incremental protocol |
| unidirectional stream    | logs, asset push, one-way result            |
| datagram                 | stale data is worse than missing data       |

Never allocate directly from an untrusted length field. Cap stream count, total buffered
bytes, message length, decompressed size, and work per connection.

### 6. Security and Replay

| Concern             | Design requirement                                        |
| ------------------- | --------------------------------------------------------- |
| certificate trust   | verify the intended identity; do not disable verification |
| ALPN                | negotiate the exact application protocol                  |
| 0-RTT               | early data may be replayed; allow only replay-safe work   |
| authentication      | transport identity may not equal application user         |
| migration           | authorization must not rely only on source IP             |
| key logging/tracing | keep diagnostic secrets out of production logs            |

A safe 0-RTT operation is idempotent or otherwise replay-tolerant. “Charge account,”
“delete record,” and “run build” are not safe merely because the transport accepts them
early.

### 7. Backpressure and Concurrency

```rust
use std::sync::Arc;
use tokio::sync::Semaphore;

async fn serve_connection(
    connection: quinn::Connection,
    permits: Arc<Semaphore>,
) -> anyhow::Result<()> {
    loop {
        let (send, receive) = connection.accept_bi().await?;
        let permit = Arc::clone(&permits).acquire_owned().await?;

        tokio::spawn(async move {
            let _permit = permit;
            if let Err(error) = handle_stream(send, receive).await {
                tracing::warn!(%error, "stream failed");
            }
        });
    }
}
```

The semaphore bounds active stream handlers. A production server also needs connection
limits, idle timeouts, cancellation, task supervision, per-peer policy, and a strategy
for expected connection closure.

### 8. QUIC for a Language Runtime

| Runtime feature         | Useful QUIC mapping                           |
| ----------------------- | --------------------------------------------- |
| compile request         | one bidirectional stream                      |
| diagnostic stream       | server-initiated unidirectional stream        |
| debugger session        | long-lived framed bidirectional stream        |
| cancellation            | reset/stop the request stream plus request ID |
| live telemetry          | datagrams when old samples have no value      |
| package/object transfer | independent reliable streams with hashes      |

Define semantic cancellation separately from transport reset: stopping byte delivery
does not automatically roll back compilation, database work, or external effects.

> ➡️ **Next:** browser internals follow network responses into HTML parsing, style,
> layout, painting, and script execution.

---

## 🌐 How Browser Engines Turn Bytes into Pixels

The classic [How Browsers Work](https://web.dev/articles/howbrowserswork) article gives
a useful **conceptual pipeline** for browser internals. It was originally published in
2011, and the page itself warns that parts are no longer accurate. Use it to build a
mental model—not as a current specification for Chromium, Firefox, or WebKit internals.

> 🧠 **Compiler connection:** a browser is several language implementations cooperating
> in real time: HTML builds structure, CSS computes presentation rules, JavaScript
> mutates state, and the rendering engine turns the result into pixels.

### 1. The Major Browser Subsystems

| Subsystem           | Main responsibility                            | Language/runtime analogy                 |
| ------------------- | ---------------------------------------------- | ---------------------------------------- |
| user interface      | tabs, address bar, navigation, chrome          | debugger or IDE shell                    |
| browser engine      | coordinates navigation and page lifecycle      | compilation driver                       |
| networking          | fetches resources under cache/security rules   | module loader                            |
| rendering engine    | parses, styles, lays out, and paints documents | compiler front end plus display backend  |
| JavaScript engine   | parses, optimizes, and executes scripts        | JIT/AOT language runtime                 |
| UI/graphics backend | draws platform controls and graphics           | target-specific code-generation backend  |
| storage             | cookies, caches, databases, local state        | persistent runtime state                 |
| process sandbox     | isolates sites, frames, or services            | privilege and fault-containment boundary |

Real browsers use multiple processes and threads. The exact split changes over time, so
reason in terms of **responsibilities and messages**, not one fixed process diagram.

### 2. The Rendering Pipeline

```text
network bytes
    ├─ HTML tokenizer + tree builder ───────────────→ DOM
    ├─ CSS parser ─────────────────────────────────→ style rules
    └─ scripts/fonts/images ───────────────────────→ external resources

DOM + matching/cascade + inherited values
    ↓
computed styles + render/layout objects
    ↓
layout: sizes and positions
    ↓
paint records / display list
    ↓
rasterization + compositing
    ↓
pixels
```

This diagram is a dependency graph, **not a promise of strictly sequential execution**.
Browsers stream input, discover resources early, update partial trees, invalidate only
affected work when possible, and may perform raster/compositing work elsewhere.

| Stage             | Core question                                | Typical bug or cost                         |
| ----------------- | -------------------------------------------- | ------------------------------------------- |
| tokenize          | Which HTML token is next?                    | malformed input or wrong parser state       |
| tree construction | Where does this token belong?                | surprising error recovery                   |
| style computation | Which declarations win for this element?     | cascade/specificity confusion               |
| layout            | What rectangle does each visible box occupy? | unnecessary synchronous relayout            |
| paint             | In what order are visual commands emitted?   | repainting a large invalidated region       |
| composite         | How are rendered layers combined?            | too many layers or expensive visual effects |

### 3. HTML Parsing Is an Error-Recovering State Machine

HTML is not parsed like a small, strict configuration grammar. Its tokenizer changes
state based on characters and context; tree construction tracks insertion modes and a
stack of open elements. Invalid or awkward markup is often repaired according to
defined recovery rules.

```text
bytes → tokenizer state → token
                     token + insertion mode + open-element stack
                                      ↓
                              DOM mutation/recovery
```

| Strict language parser                    | Browser HTML parser                         |
| ----------------------------------------- | ------------------------------------------- |
| usually rejects malformed syntax          | attempts deterministic recovery             |
| grammar can often stay context-free-ish   | state and tree context strongly affect work |
| parsing often completes before evaluation | scripts can interact with parsing           |
| one AST represents the program            | several cooperating trees/representations   |

When designing your own language, decide explicitly whether malformed input is:

1. **fatal** for batch compilation;
2. **recoverable** for an IDE so later diagnostics remain visible; or
3. **normalized** for a tolerant data/document format.

Recovery is part of the language contract. Test it with incomplete and adversarial
input, not just valid programs.

### 4. DOM, Style, and Layout Trees Are Different Representations

The DOM records document structure, but it is not a one-to-one map of painted objects.

| Example                          | DOM representation | Render/layout representation           |
| -------------------------------- | ------------------ | -------------------------------------- |
| `<head>` metadata                | present            | normally no visible box                |
| `display: none` subtree          | present            | omitted from layout/paint              |
| text inside styled elements      | text nodes         | one or more shaped/layout text runs    |
| pseudo-element                   | no ordinary node   | generated presentation object          |
| positioned or fragmented content | one source element | may require specialized/multiple boxes |

This is the same reason a compiler keeps several IRs:

```text
source syntax tree       answers "what was written?"
typed IR                 answers "what does it mean?"
control-flow IR          answers "what can execute next?"
machine IR               answers "how will the target do it?"

DOM                      answers "what document exists?"
computed style/layout    answers "what visual objects and geometry exist?"
display list             answers "what drawing operations must occur?"
```

Do not force one structure to answer every question. Derive representations with clear
invariants and keep mappings for diagnostics and debugging.

### 5. Scripts, Styles, and Resource Discovery Affect Order

Classic scripts can pause HTML parsing because they may inspect or mutate the document.
Deferred and asynchronous loading modes express different ordering constraints.
Browsers may also use a speculative scanner to discover likely resources while the main
parser is blocked.

| Resource behavior   | Useful mental model                                   |
| ------------------- | ----------------------------------------------------- |
| parser-blocking     | execute at a specific construction point              |
| deferred            | fetch alongside parsing, run after document parsing   |
| asynchronous        | execute when ready; do not rely on document order     |
| speculative preload | discover/fetch early without committing DOM mutations |

> ⚠️ **Ordering rule:** “downloaded” does not mean “safe to execute now.” Fetch order,
> execution order, DOM readiness, and dependency readiness are separate states.

That distinction also matters in a module loader for your own language:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
enum ModuleState {
    Discovered,
    Fetching,
    Parsed,
    Linked,
    Ready,
    Failed,
}
```

### 6. Style Is a Rule-Resolution System

Style computation combines selector matching, inheritance, defaults, origins,
importance, specificity, and source order. It resembles name resolution: many
candidates exist, but deterministic precedence chooses the effective value.

| CSS idea           | Compiler analogy                            |
| ------------------ | ------------------------------------------- |
| selector matching  | lookup candidates for a symbol              |
| cascade precedence | overload/scope priority rules               |
| inherited property | environment passed from parent scope        |
| computed value     | resolved, normalized semantic value         |
| invalidation       | dependency-driven incremental recompilation |

A useful engine design stores enough dependency information to answer:
**“If this attribute, class, or rule changed, which computed results may now be stale?”**

```rust
enum Dirty {
    Clean,
    Paint,
    LayoutAndPaint,
    RecomputeStyle,
    RebuildSubtree,
}
```

Use the smallest sound invalidation class. An incorrect narrow invalidation produces
stale output; invalidating everything preserves correctness but wastes time.

### 7. Layout, Paint, and Composite Are Separate Costs

| Change                                | Minimum conceptual work                    |
| ------------------------------------- | ------------------------------------------ |
| text color                            | paint                                      |
| width, font metrics, or position      | layout then paint                          |
| element added/removed                 | style/tree update, layout, then paint      |
| transform/opacity on a suitable layer | often composite, subject to implementation |

Layout calculates geometry and may be global or incremental. Browsers mark affected
objects dirty, then update them when needed. Reading a geometry property after a DOM
write can force pending layout to complete immediately.

```text
bad:    write style → read geometry → write style → read geometry
better: read required geometry → compute → batch writes
```

Paint walks visible objects in defined visual order and emits drawing operations.
Incremental painting tries to restrict work to invalidated regions. Compositing combines
already-rendered surfaces/layers, but layer promotion is an optimization—not a semantic
guarantee to depend on.

### 8. A Tiny Browser-Like Architecture for Your Own UI Language

```rust
struct UiEngine {
    document: DocumentTree,
    styles: StyleSheet,
    layout: LayoutTree,
    display_list: Vec<DrawCommand>,
}

enum DrawCommand {
    FillRect { x: f32, y: f32, w: f32, h: f32, rgba: [u8; 4] },
    Text { x: f32, y: f32, text: String },
}
```

Keep the passes explicit:

| Pass               | Input                           | Output/invariant                 |
| ------------------ | ------------------------------- | -------------------------------- |
| parse document     | bounded UTF-8 source            | well-formed recoverable tree     |
| resolve styles     | tree + rules                    | effective values per styled node |
| construct layout   | styled visible nodes            | boxes linked to source nodes     |
| compute geometry   | boxes + viewport/font metrics   | finite sizes and positions       |
| build display list | ordered boxes                   | backend-neutral drawing commands |
| render             | display list + graphics backend | pixels or recorded vector output |

This design can target SDL, a browser canvas, a GPU API, or a test renderer. Snapshot
the display list in tests so layout logic can be verified without comparing screenshots
for every case.

> ➡️ **Next:** Actix shows the server side of the web boundary, while `wasm-bindgen`,
> `web-sys`, and the DOM expose the browser-hosted side.

---

## 🕸 Rust Web Boundaries: Actix Web & wasm-bindgen

Actix Web and `wasm-bindgen` sit on opposite sides of the web:

```text
browser JavaScript
    ↕ wasm-bindgen glue
Rust compiled to WebAssembly
    ↕ HTTP / structured messages
Actix Web service
```

Both are boundary-design problems. Bytes and dynamic values enter Rust, become typed
data, pass through application logic, and are converted back into an external
representation.

### 1. Boundary Comparison

| Concern            | Actix Web boundary                        | `wasm-bindgen` boundary                    |
| ------------------ | ----------------------------------------- | ------------------------------------------ |
| External peer      | HTTP client                               | JavaScript host                            |
| Input conversion   | Extractors (`Path`, `Query`, `Json`)      | Generated JS/Wasm bindings                 |
| Error form         | HTTP status + response body               | `Result<T, JsValue>` / JS exception        |
| Shared state       | `web::Data<T>`                            | Exported Rust object or Wasm linear memory |
| Trust boundary     | Network request                           | Host calls and JS values                   |
| Main resource risk | Connections, bodies, tasks, backend calls | Copies, memory growth, handles, JS GC      |

---

### 2. Actix Web Application Shape

Current Actix Web 4 applications construct an `App` inside an `HttpServer` factory:

```rust
use actix_web::{get, web, App, HttpServer, Responder};

#[get("/hello/{name}")]
async fn hello(name: web::Path<String>) -> impl Responder {
    let name = name.into_inner();
    format!("Hello, {name}!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| App::new().service(hello))
        .bind(("127.0.0.1", 8080))?
        .run()
        .await
}
```

The server factory may run once per worker. Create intentionally shared state outside
the closure and clone its `web::Data` handle into each app.

### 3. Extractors Turn Requests into Typed Inputs

| Extractor       | Source in the HTTP request                  |
| --------------- | ------------------------------------------- |
| `web::Path<T>`  | Variables captured by the route             |
| `web::Query<T>` | URL query string                            |
| `web::Json<T>`  | JSON request body                           |
| `web::Data<T>`  | Application state                           |
| `HttpRequest`   | Headers, peer data, extensions, method, URI |
| `web::Bytes`    | Buffered raw body                           |

```rust
use actix_web::{web, HttpResponse, Responder};
use serde::{Deserialize, Serialize};
use std::sync::atomic::{AtomicU64, Ordering};

struct AppState {
    next_job: AtomicU64,
}

#[derive(Debug, Deserialize, Serialize)]
struct CreateJob {
    source: String,
}

#[derive(Debug, Serialize)]
struct JobAccepted {
    id: u64,
    source_bytes: usize,
}

async fn create_job(
    state: web::Data<AppState>,
    request: web::Json<CreateJob>,
) -> impl Responder {
    let id = state.next_job.fetch_add(1, Ordering::Relaxed);
    web::Json(JobAccepted {
        id,
        source_bytes: request.source.len(),
    })
}

fn configure(config: &mut web::ServiceConfig) {
    config.service(
        web::resource("/jobs")
            .route(web::post().to(create_job)),
    );
}
```

> 🚧 **Extraction is parsing:** a handler should receive values that already satisfy
> transport-level syntax, then perform application validation such as source-size,
> authorization, and allowed-language checks.

### 4. Bound Request Resources

```rust
use actix_web::web;

fn json_config() -> web::JsonConfig {
    web::JsonConfig::default()
        .limit(32 * 1024)
}
```

| Resource          | Boundary to define                              |
| ----------------- | ----------------------------------------------- |
| Request body      | Maximum compressed and decoded size             |
| JSON nesting      | Parser/format limits when configurable          |
| Path/query values | Length and allowed character policy             |
| Connections       | Worker, backlog, keep-alive, and timeout policy |
| Backend calls     | Timeout and retry budget                        |
| Spawned tasks     | Concurrency limit and cancellation behavior     |
| Response          | Maximum buffered or streamed output             |

Do not hold a synchronous mutex guard across `.await`. Prefer an atomic for small
independent counters, a short critical section for local compound state, or an
async-aware/resource-specific client pool for I/O.

### 5. Typed HTTP Errors

```rust
use actix_web::{http::StatusCode, HttpResponse, ResponseError};
use std::fmt;

#[derive(Debug)]
enum ApiError {
    InvalidSource,
    Overloaded,
}

impl fmt::Display for ApiError {
    fn fmt(&self, formatter: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            Self::InvalidSource => write!(formatter, "invalid source"),
            Self::Overloaded => write!(formatter, "service overloaded"),
        }
    }
}

impl ResponseError for ApiError {
    fn status_code(&self) -> StatusCode {
        match self {
            Self::InvalidSource => StatusCode::BAD_REQUEST,
            Self::Overloaded => StatusCode::SERVICE_UNAVAILABLE,
        }
    }

    fn error_response(&self) -> HttpResponse {
        HttpResponse::build(self.status_code())
            .body(self.to_string())
    }
}
```

| Error layer     | Suitable information                               |
| --------------- | -------------------------------------------------- |
| Internal error  | Source chain, operation, IDs, private diagnostics  |
| Log/trace event | Correlation ID, sanitized context, latency         |
| Client response | Stable code, safe message, appropriate HTTP status |

Avoid exposing filesystem paths, database errors, backtraces, secrets, or parser
internals to an untrusted client.

### 6. Middleware and Handler Responsibilities

| Concern                      | Best location                        |
| ---------------------------- | ------------------------------------ |
| Correlation/request ID       | Middleware                           |
| Access logging and timing    | Middleware                           |
| Authentication context       | Middleware/extractor                 |
| Route-specific authorization | Handler or domain service            |
| Input extraction             | Typed extractor                      |
| Business rules               | Domain function independent of Actix |
| HTTP response mapping        | Handler / `ResponseError`            |

Body-consuming extractors need special care in middleware because the body is a stream.
A middleware that consumes it must deliberately restore or replace what downstream
handlers receive.

### 7. Test the Service Without Opening a Port

```rust
use actix_web::{http::StatusCode, test, web, App};
use std::sync::atomic::AtomicU64;

#[actix_web::test]
async fn accepts_a_job() {
    let state = web::Data::new(AppState {
        next_job: AtomicU64::new(1),
    });

    let app = test::init_service(
        App::new()
            .app_data(state)
            .app_data(json_config())
            .configure(configure),
    )
    .await;

    let request = test::TestRequest::post()
        .uri("/jobs")
        .set_json(&CreateJob {
            source: "1 + 2".to_owned(),
        })
        .to_request();

    let response = test::call_service(&app, request).await;
    assert_eq!(response.status(), StatusCode::OK);
}
```

Test routing and HTTP conversion at the Actix layer, but keep most language parsing,
type checking, and compilation tests independent of the web framework.

---

### 8. What `wasm-bindgen` Generates

WebAssembly itself has a deliberately small boundary based on numeric values, linear
memory, functions, tables, imports, and exports. `wasm-bindgen` generates glue that maps
more ergonomic Rust and JavaScript values across that lower-level interface.

```text
Rust source + #[wasm_bindgen]
    ↓ rustc --target wasm32-unknown-unknown
.wasm module
    ↓ wasm-bindgen CLI/tooling
processed Wasm + JavaScript/TypeScript bindings
```

| Layer                | Responsibility                                 |
| -------------------- | ---------------------------------------------- |
| Rust compiler        | Produce the Wasm module                        |
| `wasm-bindgen` macro | Describe imported/exported boundary items      |
| Binding generator    | Produce JS glue and metadata                   |
| JavaScript loader    | Instantiate the module and satisfy imports     |
| Wasm linear memory   | Store Rust stacks, heaps, strings, and buffers |

### 9. Export Rust Functions and Types

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn read_u32_be(bytes: &[u8]) -> Result<u32, JsValue> {
    let bytes: [u8; 4] = bytes
        .get(..4)
        .ok_or_else(|| JsValue::from_str("expected four bytes"))?
        .try_into()
        .map_err(|_| JsValue::from_str("invalid byte slice"))?;

    Ok(u32::from_be_bytes(bytes))
}

#[wasm_bindgen]
pub struct Counter {
    value: u32,
}

#[wasm_bindgen]
impl Counter {
    #[wasm_bindgen(constructor)]
    pub fn new() -> Self {
        Self { value: 0 }
    }

    pub fn increment(&mut self) -> u32 {
        self.value = self.value.saturating_add(1);
        self.value
    }
}
```

An exported Rust struct appears to JavaScript through a generated wrapper/handle. Its
fields remain private unless explicitly exported through methods or accessors.

### 10. Import JavaScript Functions

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(message: &str);
}

#[wasm_bindgen]
pub fn announce(message: &str) {
    log(message);
}
```

An imported JavaScript function is an environmental dependency just like an OS system
call or host function in a custom VM. Keep imports narrow so the module's capabilities
are easy to audit and replace in tests.

### 11. Value and Ownership Boundaries

| Rust-side value      | Boundary behavior to remember                        |
| -------------------- | ---------------------------------------------------- |
| Numbers/booleans     | Converted directly when supported                    |
| `String` / `&str`    | Encoded/decoded through Wasm memory                  |
| `Vec<u8>` / `&[u8]`  | Typically copied to/from typed-array-compatible data |
| `JsValue`            | Dynamic escape hatch; loses Rust type precision      |
| Exported Rust struct | JavaScript wrapper holds a Rust-side allocation      |
| Imported JS object   | Rust holds a generated handle to a host object       |
| `Option<T>`          | Often maps to a nullable/optional host value         |

> ⚠️ **Memory-view hazard:** Wasm memory can grow. JavaScript typed-array views into old
> memory may become stale after growth, so reacquire views according to the generated
> API instead of retaining them indefinitely.

### 12. Error Design Across JavaScript

Use `Result<T, JsValue>` when a Rust export can fail for a JavaScript caller:

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn checked_frame_size(length: u32) -> Result<u32, JsValue> {
    const MAX_FRAME: u32 = 64 * 1024;
    if length > MAX_FRAME {
        return Err(JsValue::from_str("frame exceeds limit"));
    }
    Ok(length)
}
```

| Boundary choice     | Trade-off                                       |
| ------------------- | ----------------------------------------------- |
| `JsValue` error     | Natural JS interop, weak Rust-side structure    |
| Numeric error code  | Compact ABI, requires an external error table   |
| Serialized error    | Stable schema, conversion overhead              |
| Exported error type | Rich API, more bindings and lifetime management |

Do not expose Rust panics as normal validation. Convert expected failure into `Result`
and install deliberate panic diagnostics only for unexpected bugs.

### 13. Using Wasm as a Language Target

For your own language, WebAssembly can be:

| Role                      | Architecture                                    |
| ------------------------- | ----------------------------------------------- |
| Compilation target        | Source → typed IR → Wasm                        |
| Sandboxable plugin format | Host supplies a small import capability set     |
| Browser runtime           | Language VM compiled to Wasm, UI supplied by JS |
| Portable bytecode         | Server/CLI embeds a Wasm engine                 |

| Wasm language-target contract | Decision to define                                |
| ----------------------------- | ------------------------------------------------- |
| Imports                       | Module and function names                         |
| Representation                | Value and memory layouts                          |
| Allocation                    | Ownership across host/module calls                |
| Strings/errors                | Encoding and failure representation               |
| Memory growth                 | Whether growth is permitted and view invalidation |
| Resource limits               | Fuel, wall time, stack, and memory                |
| Capabilities                  | Deterministic/nondeterministic host functions     |

This is the same design work as an FFI ABI and a network protocol: **representation,
ownership, versioning, and trust** must all be explicit.

### 14. The DOM Is a Host-Owned Object Graph

As the
[`wasm-bindgen` DOM examples](https://wasm-bindgen.github.io/wasm-bindgen/examples/dom.html)
demonstrate, the browser's Document Object Model is **not stored inside Wasm linear
memory**. `web-sys` exposes typed Rust handles to JavaScript-managed objects.

```text
browser Window
    └── Document
        └── Node tree
            ├── Element
            │   └── HtmlElement / HtmlCanvasElement / ...
            └── Text

Rust/Wasm holds generated JS handles ──→ objects above
Rust values, stack, heap, strings ─────→ Wasm linear memory
```

| Type          | Role                                              |
| ------------- | ------------------------------------------------- |
| `Window`      | browser global, timers, location, event target    |
| `Document`    | element creation and document-wide lookup         |
| `Node`        | common tree operations such as `append_child`     |
| `Element`     | attributes, selectors, classes, element content   |
| `HtmlElement` | HTML-specific style, focus, and common properties |
| `EventTarget` | add/remove event listener boundary                |
| `JsValue`     | dynamic JavaScript value and error carrier        |

`web-sys` APIs are feature-gated. Enable only the browser types and methods your crate
uses; a compile error about a missing method may mean a missing Cargo feature rather
than a missing browser API.

### 15. Create and Attach DOM Nodes

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen(start)]
pub fn start() -> Result<(), JsValue> {
    let window = web_sys::window()
        .ok_or_else(|| JsValue::from_str("window is unavailable"))?;
    let document = window
        .document()
        .ok_or_else(|| JsValue::from_str("document is unavailable"))?;
    let body = document
        .body()
        .ok_or_else(|| JsValue::from_str("document has no body"))?;

    let status = document.create_element("p")?;
    status.set_id("compiler-status");
    status.set_text_content(Some("Compiler ready 🦀"));
    body.append_child(&status)?;
    Ok(())
}
```

Prefer `set_text_content` when displaying untrusted text. `set_inner_html` parses HTML,
so using it with untrusted source, diagnostics, names, or network data can create an
injection vulnerability.

| Lookup/creation API | Result shape                 | Design response                 |
| ------------------- | ---------------------------- | ------------------------------- |
| `web_sys::window()` | `Option<Window>`             | host environment may differ     |
| `window.document()` | `Option<Document>`           | worker has no `Document`        |
| `get_element_by_id` | `Option<Element>`            | missing markup is expected      |
| `query_selector`    | `Result<Option<Element>, _>` | selector may be invalid/missing |
| `create_element`    | `Result<Element, JsValue>`   | browser operation may throw     |
| `append_child`      | `Result<Node, JsValue>`      | tree mutation may fail          |

### 16. Cast Only at a Checked Boundary

A CSS selector returns a general `Element`. Use `JsCast::dyn_into` when an operation
requires a more specific browser type.

```rust
use wasm_bindgen::{JsCast, JsValue};
use web_sys::HtmlInputElement;

fn source_text(document: &web_sys::Document) -> Result<String, JsValue> {
    let element = document
        .get_element_by_id("source")
        .ok_or_else(|| JsValue::from_str("missing #source"))?;
    let input = element
        .dyn_into::<HtmlInputElement>()
        .map_err(|_| JsValue::from_str("#source is not an input element"))?;
    Ok(input.value())
}
```

| Cast style              | Use                                             |
| ----------------------- | ----------------------------------------------- |
| `dyn_ref::<T>()`        | checked borrowed view; keep original handle     |
| `dyn_into::<T>()`       | checked consuming conversion                    |
| `unchecked_ref::<T>()`  | glue boundary only when type is already proven  |
| `unchecked_into::<T>()` | rare; caller assumes the complete type contract |

Unchecked casting is analogous to `unsafe`: keep it local and document what establishes
the runtime JavaScript type.

### 17. DOM Event Listeners Need Lifetime Design

Browser callbacks are JavaScript functions, while Rust closures live in Wasm-managed
state. Dropping a `Closure` while JavaScript still holds its callback is invalid.
Calling `forget()` keeps it alive forever, which is acceptable only for a truly
page-lifetime listener.

```rust
use wasm_bindgen::{closure::Closure, JsCast, JsValue};
use web_sys::{EventTarget, MouseEvent};

struct ClickListener {
    target: EventTarget,
    callback: Closure<dyn FnMut(MouseEvent)>,
}

impl ClickListener {
    fn install(
        target: EventTarget,
        mut on_click: impl FnMut(MouseEvent) + 'static,
    ) -> Result<Self, JsValue> {
        let callback = Closure::new(move |event: MouseEvent| on_click(event));
        target.add_event_listener_with_callback(
            "click",
            callback.as_ref().unchecked_ref(),
        )?;
        Ok(Self { target, callback })
    }
}

impl Drop for ClickListener {
    fn drop(&mut self) {
        let _ = self.target.remove_event_listener_with_callback(
            "click",
            self.callback.as_ref().unchecked_ref(),
        );
    }
}
```

The application must retain the `ClickListener` for as long as the UI is active. This
RAII shape pairs registration with removal and works well inside a Rust application
struct exported through `wasm-bindgen`.

| Callback strategy        | Lifetime/result                                 |
| ------------------------ | ----------------------------------------------- |
| local `Closure`, dropped | listener becomes invalid                        |
| `closure.forget()`       | page-lifetime leak by design                    |
| store in app state       | listener lives with component/application       |
| RAII listener wrapper    | automatically unregisters when owner is dropped |

### 18. UI State Crosses Two Memory Managers

```rust
use std::{cell::Cell, rc::Rc};

let clicks = Rc::new(Cell::new(0_u32));
let callback_state = Rc::clone(&clicks);

let listener = ClickListener::install(button.into(), move |_event| {
    callback_state.set(callback_state.get().saturating_add(1));
})?;
```

| Ownership edge                  | What keeps it alive                   |
| ------------------------------- | ------------------------------------- |
| Rust state captured by closure  | Wasm closure environment              |
| `Closure` callback wrapper      | Rust owner or deliberate `forget()`   |
| DOM node                        | browser DOM tree and JS references    |
| Rust handle to DOM node         | generated JS reference table/glue     |
| typed-array view of Wasm memory | JS value, but backing memory may grow |

`Rc<Cell<T>>` is appropriate for small single-threaded browser state. Use
`Rc<RefCell<T>>` for more complex mutable state, but keep borrows short and never call
unknown JavaScript while a mutable `RefCell` borrow is held: JavaScript can re-enter
Rust through another callback.

### 19. DOM Work Is Also Performance Work

| Expensive pattern                         | Better direction                                |
| ----------------------------------------- | ----------------------------------------------- |
| alternate layout reads and writes         | group reads, compute, then group writes         |
| rebuild a large subtree for one text edit | mutate the smallest stable node                 |
| copy huge buffers for every event         | batch, reuse, or expose bounded views carefully |
| compile on every keystroke immediately    | debounce/cancel stale work                      |
| block the main thread with language work  | use a Web Worker or incremental execution       |

For a browser-hosted language:

```text
DOM input event
    ↓ capture bounded source text
debounced compile request
    ↓ lexer → parser → type checker → Wasm/backend
structured diagnostics
    ↓ escape as text, map byte spans to source view
small batched DOM update
```

Keep the compiler core independent of `web-sys`. A browser adapter should translate DOM
events into plain Rust inputs and translate structured compiler results back into UI
updates. That preserves CLI, test, server, and browser front ends over one language
implementation.

> ➡️ **Next:** SDL exposes native windows, input, timing, audio, and drawing without the
> browser's document and JavaScript host layers.

---

## 🎮 Native Event Loops & Multimedia with SDL3

The [SDL3 API by category](https://wiki.libsdl.org/SDL3/APIByCategory) is best read as a
map of platform services. SDL does not impose a game architecture; it gives a portable
boundary over windows, input, graphics, audio, timing, files, threads, and devices.

### 1. Read the API by Responsibility

| API family            | What it gives the program                    | Boundary to design carefully          |
| --------------------- | -------------------------------------------- | ------------------------------------- |
| initialization        | subsystem startup/shutdown                   | partial failure and cleanup           |
| video                 | windows, displays, surfaces                  | OS-owned handles                      |
| events                | keyboard, mouse, touch, window/device events | queue draining and latency            |
| 2D render             | draw state, textures, primitives, present    | frame lifetime and backend state      |
| GPU                   | explicit modern graphics work                | synchronization and resource lifetime |
| audio                 | devices, streams, formats                    | real-time callback constraints        |
| gamepad/joystick      | controllers, axes, buttons, hotplug          | unstable device identities            |
| time                  | ticks, delay, high-resolution timing         | units, overflow, frame pacing         |
| threads/synchronizing | worker threads, mutexes, conditions          | ownership and shutdown                |
| I/O/storage           | streams, paths, filesystem helpers           | untrusted bytes and platform paths    |
| properties/logging    | extensible metadata and diagnostics          | type agreement and observability      |

> 🆕 **SDL3 migration note:** do not paste SDL2 error checks blindly. Many SDL3
> operations—including `SDL_Init` and `SDL_CreateWindowAndRenderer`—return `bool`, with
> `true` meaning success.

### 2. Resource Lifetime Is the First Invariant

```text
SDL_Init
    ↓
create window + renderer
    ↓
poll events → update state → render → present
    ↓
destroy renderer → destroy window
    ↓
SDL_Quit
```

Every successful acquisition must have one cleanup path. Destroy resources in an order
compatible with their dependencies.

### 3. Minimal SDL3 Window and Render Loop

```c
#include <SDL3/SDL.h>
#include <stdbool.h>

int main(void) {
    SDL_Window *window = NULL;
    SDL_Renderer *renderer = NULL;

    if (!SDL_Init(SDL_INIT_VIDEO)) {
        SDL_Log("SDL_Init failed: %s", SDL_GetError());
        return 1;
    }

    if (!SDL_CreateWindowAndRenderer(
            "Language Runtime", 800, 600, 0, &window, &renderer)) {
        SDL_Log("window/renderer creation failed: %s", SDL_GetError());
        SDL_Quit();
        return 1;
    }

    bool running = true;
    while (running) {
        SDL_Event event;
        while (SDL_PollEvent(&event)) {
            if (event.type == SDL_EVENT_QUIT) {
                running = false;
            }
        }

        SDL_SetRenderDrawColor(renderer, 18, 22, 30, 255);
        SDL_RenderClear(renderer);

        SDL_FRect box = {120.0f, 90.0f, 240.0f, 120.0f};
        SDL_SetRenderDrawColor(renderer, 88, 166, 255, 255);
        SDL_RenderFillRect(renderer, &box);
        SDL_RenderPresent(renderer);
    }

    SDL_DestroyRenderer(renderer);
    SDL_DestroyWindow(window);
    SDL_Quit();
    return 0;
}
```

Check the return value of fallible drawing operations in production code. A concise
tutorial may omit checks to keep the lifecycle visible; an engine should surface errors
with operation, resource, and frame context.

### 4. Separate Platform Events from Simulation State

```text
OS/SDL events
    ↓ normalize
application commands
    ↓ update
deterministic world state
    ↓ interpolate/view
draw commands
    ↓ renderer/GPU
presented frame
```

| Layer                 | Example data                                 |
| --------------------- | -------------------------------------------- |
| raw event             | key code, device ID, window event            |
| normalized command    | `MoveLeft(true)`, `Resize { width, height }` |
| simulation state      | position, velocity, editor selection         |
| presentation snapshot | visible sprites, boxes, glyphs               |

Do not let gameplay or language semantics depend directly on platform key codes. An
input-mapping layer makes tests deterministic and lets keyboard, gamepad, touch, and
replay files produce the same commands.

### 5. Variable Rendering, Fixed Simulation

```text
elapsed = now - previous
accumulator += clamp(elapsed)

while accumulator >= fixed_step:
    update(fixed_step)
    accumulator -= fixed_step

render(interpolation = accumulator / fixed_step)
```

| Timing mistake                       | Result                           |
| ------------------------------------ | -------------------------------- |
| simulation speed tied to frame count | behavior changes across machines |
| unbounded catch-up after a pause     | spiral of death                  |
| mixed milliseconds/nanoseconds       | enormous or tiny time steps      |
| nondeterministic input inside update | replays/tests diverge            |

SDL provides timing primitives such as `SDL_GetTicksNS`; convert once into a
well-named duration type. Clamp unusually large elapsed intervals after breakpoints,
suspend/resume, or window dragging.

### 6. Rendering APIs Form a State Machine

The renderer remembers draw color, blend mode, target, viewport, clip rectangle, and
texture state. Make state transitions explicit rather than relying on whatever the
previous draw left behind.

```text
begin frame
  set target/viewport/clip
  clear
  draw ordered opaque and transparent work
  reset temporary state
present
```

| Representation | Best use                                |
| -------------- | --------------------------------------- |
| surface        | CPU-side pixels, loading/manipulation   |
| texture        | renderer-owned image used for drawing   |
| 2D renderer    | portable sprites, rectangles, simple UI |
| GPU API        | explicit pipelines, buffers, shaders    |

### 7. Audio and Device Events Need Special Discipline

Audio work runs under tight real-time constraints. Avoid blocking I/O, long locks,
allocation spikes, and logging in the time-critical path. Feed an audio stream from
bounded buffers and define underrun behavior.

Controllers can appear and disappear while the program runs:

```text
device-added → open/identify → map controls → sample
device-removed → stop using handle → release → notify application
```

Use a logical player/device handle rather than treating a temporary enumeration index
as permanent identity.

### 8. Wrap SDL Handles at a Safe Language Boundary

```rust
struct WindowHandle {
    raw: *mut sdl_sys::SDL_Window,
}

impl Drop for WindowHandle {
    fn drop(&mut self) {
        if !self.raw.is_null() {
            unsafe { sdl_sys::SDL_DestroyWindow(self.raw) };
        }
    }
}
```

The full wrapper must also define:

| Contract          | Question                                                    |
| ----------------- | ----------------------------------------------------------- |
| nullability       | Can creation or lookup return null?                         |
| thread affinity   | Which thread may call this operation?                       |
| aliasing          | Can two safe wrappers believe they uniquely own one handle? |
| callbacks         | How long do captured values live, and can calls re-enter?   |
| destruction order | Which child resources must be dropped first?                |
| strings/buffers   | Who owns bytes, and for how long are pointers valid?        |

Keep raw pointers private. A custom language runtime should expose stable integer
handles or capability objects, validate every lookup, and never let untrusted scripts
forge native addresses.

### 9. SDL as a Backend for a Small Language

```text
script/DSL
    ↓ parse + type/check capabilities
runtime commands: CreateWindow, DrawRect, PlayTone, PollInput
    ↓ validated host adapter
SDL resources and event queue
    ↓
OS window, devices, audio, graphics
```

The host adapter should enforce limits on windows, textures, audio buffers, file paths,
and per-frame commands. This turns SDL from an unrestricted FFI surface into a
capability-based standard library.

> ➡️ **Next:** Bevy builds a data-oriented scheduler and application framework over the
> same event/update/render lifecycle.

---

## 🧰 Applied Rust Architectures: Bevy, Diesel & Tauri

These frameworks solve different problems, but they all reward the same habits:
explicit data models, narrow boundaries, scheduled work, bounded resources, and
testable domain logic.

| Framework | Primary problem                       | Architectural lesson                 |
| --------- | ------------------------------------- | ------------------------------------ |
| Bevy      | data-driven games/visual applications | behavior as scheduled systems        |
| Diesel    | typed relational database access      | schema/query contract in Rust types  |
| Tauri 2   | cross-platform webview applications   | least-privilege frontend/backend IPC |

### 1. Bevy: Data-Oriented Application Structure

The [Bevy quick start](https://bevy.org/learn/quick-start/introduction/) introduces a
modular, data-focused Rust engine based on the Entity Component System (ECS) paradigm.

```text
Entity    = identity only
Component = typed data attached to an entity
System    = function that queries/transforms data
Resource  = unique world-wide data
Schedule  = when and in what constraints systems run
Plugin    = reusable bundle of systems/resources/configuration
```

| ECS concept | Compiler/runtime analogy                          |
| ----------- | ------------------------------------------------- |
| entity      | stable arena ID or object handle                  |
| component   | fact attached to an ID                            |
| query       | select IDs possessing a required fact combination |
| system      | compiler pass over selected data                  |
| resource    | target configuration, interner, diagnostic sink   |
| schedule    | pass pipeline and dependency ordering             |
| plugin      | optional language feature/backend package         |

#### A Small Scheduled Model

```rust
use bevy::prelude::*;

#[derive(Component)]
struct Velocity(Vec2);

#[derive(Resource, Default)]
struct SimulationStats {
    moved_entities: u64,
}

fn spawn(mut commands: Commands) {
    commands.spawn((
        Transform::default(),
        Velocity(Vec2::new(3.0, -1.0)),
    ));
}

fn integrate(
    time: Res<Time>,
    mut stats: ResMut<SimulationStats>,
    mut movers: Query<(&Velocity, &mut Transform)>,
) {
    for (velocity, mut transform) in &mut movers {
        transform.translation +=
            velocity.0.extend(0.0) * time.delta_secs();
        stats.moved_entities += 1;
    }
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .init_resource::<SimulationStats>()
        .add_systems(Startup, spawn)
        .add_systems(Update, integrate)
        .run();
}
```

Bevy can parallelize systems when their parameter access is compatible. A system that
reads `Res<T>` can overlap with other readers; a `ResMut<T>` or mutable component query
declares exclusive access to that data.

#### ECS Design Questions

| Question                     | Better direction                                  |
| ---------------------------- | ------------------------------------------------- |
| component or resource?       | per-entity fact vs one world-wide value           |
| component or enum field?     | queried independently vs always part of one state |
| event/message or component?  | transient notification vs persistent state        |
| one huge system?             | separate transformations with explicit ordering   |
| every entity has everything? | compose only the data each behavior requires      |

Do not use ECS merely to avoid ordinary structs. It is strongest when large collections
need different combinations of data-oriented behavior.

### 2. Diesel: Typed SQL Boundary

The [Diesel guides](https://diesel.rs/guides/) separate setup, selects, updates, inserts,
relations, schema generation, application composition, and extension. Diesel turns many
schema/query mismatches into compile-time type errors, but runtime data, connectivity,
transactions, constraints, and authorization still require deliberate handling.

```text
migration files
    ↓ create/update database schema
generated Rust schema (`table!`)
    ↓ constrains typed query expressions
Selectable / Queryable / Insertable models
    ↓
domain conversion and application rules
```

#### Separate Row, Insert, and Domain Types

```rust
use diesel::prelude::*;

#[derive(Debug, Queryable, Selectable)]
#[diesel(table_name = crate::schema::compile_jobs)]
struct JobRow {
    id: i32,
    source: String,
    status: String,
}

#[derive(Insertable)]
#[diesel(table_name = crate::schema::compile_jobs)]
struct NewJob<'a> {
    source: &'a str,
    status: &'a str,
}

fn pending_jobs(
    connection: &mut SqliteConnection,
) -> QueryResult<Vec<JobRow>> {
    use crate::schema::compile_jobs::dsl;

    dsl::compile_jobs
        .filter(dsl::status.eq("pending"))
        .order(dsl::id.asc())
        .select(JobRow::as_select())
        .load(connection)
}
```

| Type                | Owns which contract                     |
| ------------------- | --------------------------------------- |
| generated schema    | SQL table/column types                  |
| `JobRow`            | selected database row shape             |
| `NewJob<'a>`        | allowed insert fields, borrowing inputs |
| domain `CompileJob` | validated application invariants        |
| API response        | intentionally exposed external fields   |

Do not use one struct for every layer merely because the fields currently match.

#### Transactions Protect Multi-Step Invariants

```rust
fn claim_job(
    connection: &mut SqliteConnection,
    job_id: i32,
) -> QueryResult<JobRow> {
    connection.transaction(|connection| {
        use crate::schema::compile_jobs::dsl;

        diesel::update(dsl::compile_jobs.find(job_id))
            .filter(dsl::status.eq("pending"))
            .set(dsl::status.eq("running"))
            .execute(connection)?;

        dsl::compile_jobs
            .find(job_id)
            .select(JobRow::as_select())
            .first(connection)
    })
}
```

The sketch still needs an affected-row check or database constraint to distinguish
“claimed” from “already unavailable.” Transactions provide atomicity; they do not invent
the business invariant.

#### Diesel Boundary Checklist

| Area              | Decision                                                      |
| ----------------- | ------------------------------------------------------------- |
| migrations        | versioned, reviewed, reversible strategy                      |
| connection        | pool size, timeout, health, transaction lifetime              |
| async application | avoid blocking executor threads; use appropriate adapter/pool |
| query cardinality | `first`, `optional`, one, many, or paginated                  |
| updates           | distinguish absent field from explicit SQL `NULL`             |
| errors            | map not-found, constraint, transient, and internal failures   |
| observability     | record sanitized query context and latency                    |

### 3. Tauri 2: A Desktop Capability Boundary

[Tauri 2](https://v2.tauri.app/start/) combines a web frontend with native/mobile
application code. JavaScript calls registered Rust commands across IPC; this makes every
command a trust and serialization boundary.

```text
untrusted or semi-trusted webview input
    ↓ deserialize command arguments
Tauri command
    ↓ validate + authorize
domain service
    ↓ narrow OS/database/network capability
structured response/error
    ↓ serialize to frontend
```

#### Register a Narrow Command Surface

```rust
use serde::Serialize;
use std::sync::atomic::{AtomicU64, Ordering};
use tauri::State;

struct AppState {
    next_request: AtomicU64,
}

#[derive(Serialize)]
#[serde(rename_all = "camelCase")]
struct CompileAccepted {
    request_id: u64,
    source_bytes: usize,
}

#[tauri::command]
fn compile_source(
    source: String,
    state: State<'_, AppState>,
) -> Result<CompileAccepted, String> {
    const MAX_SOURCE_BYTES: usize = 256 * 1024;

    if source.len() > MAX_SOURCE_BYTES {
        return Err("source exceeds configured limit".to_owned());
    }

    let request_id =
        state.next_request.fetch_add(1, Ordering::Relaxed);

    Ok(CompileAccepted {
        request_id,
        source_bytes: source.len(),
    })
}

pub fn run() {
    tauri::Builder::default()
        .manage(AppState {
            next_request: AtomicU64::new(1),
        })
        .invoke_handler(tauri::generate_handler![compile_source])
        .run(tauri::generate_context!())
        .expect("Tauri runtime failed");
}
```

The outer `expect` handles an unrecoverable application-startup failure. Input-dependent
command failures remain structured `Result` values.

#### Capabilities and Permissions

| Boundary                 | Least-privilege question                              |
| ------------------------ | ----------------------------------------------------- |
| command registration     | Does this command need to be callable at all?         |
| capability configuration | Which windows/webviews receive which permissions?     |
| filesystem               | Which paths and operations are allowed?               |
| shell/process            | Can fixed programs replace arbitrary commands?        |
| network                  | Which destinations and protocols are necessary?       |
| plugin                   | Which plugin commands are exposed?                    |
| updater/deep links       | How are authenticity and untrusted arguments checked? |

Never turn a frontend string into an unrestricted shell command, path, SQL fragment, or
URL fetch. Parse it into a typed request and authorize the resulting operation.

#### Events, Channels, and Commands

| Mechanism     | Best fit                                   |
| ------------- | ------------------------------------------ |
| command       | bounded request/response                   |
| event         | broadcast-style notification               |
| channel       | ordered streaming progress/data            |
| managed state | shared native service/configuration handle |

For long compilation, download, or indexing jobs, return a request ID promptly and send
bounded progress events. Define cancellation and terminal states explicitly.

### 4. Zero to Production: Service Engineering

The [Zero to Production repository](https://github.com/LukeMathWalker/zero-to-production/)
is the evolving reference application for an opinionated Rust backend book. Its
newsletter service connects Actix Web, PostgreSQL/SQLx, Redis-backed sessions,
configuration, authentication, email delivery, telemetry, migrations, and API tests.

> 🚀 **Production mental model:** an HTTP handler is only the visible edge. A service is
> the complete system that can be configured, observed, migrated, tested, deployed,
> restarted, and operated without guessing.

#### Grow the System in Dependency Order

| Layer                  | Question to settle                                      |
| ---------------------- | ------------------------------------------------------- |
| health endpoint        | Can orchestration tell whether the process serves?      |
| configuration          | How do local, test, and production values differ?       |
| database migration     | How does persistent state evolve with the program?      |
| request extraction     | Which bytes become trusted domain inputs?               |
| domain/use-case logic  | Which invariant must hold independent of HTTP?          |
| outbound dependency    | How are email/cache calls bounded and retried?          |
| authentication/session | Who is acting, and how is that identity established?    |
| telemetry              | Can an operator reconstruct a failed request?           |
| deployment             | Can the artifact start reproducibly in a fresh setting? |

Build a thin vertical slice first:

```text
TCP listener
    ↓ HTTP request
router → handler → validated command
                    ↓
                 domain service
                    ↓
              repository/outbound port
                    ↓
            structured response + trace
```

Each added dependency should arrive behind a narrow interface and with a test strategy.

#### Bind the Listener Outside the Application Factory

A testable server accepts a listener chosen by its caller. Tests can bind port `0` and
ask the OS for an unused port instead of racing over a hard-coded one.

```rust
use actix_web::{web, App, HttpResponse, HttpServer};
use std::net::TcpListener;

async fn health_check() -> HttpResponse {
    HttpResponse::Ok().finish()
}

fn run(listener: TcpListener) -> std::io::Result<actix_web::dev::Server> {
    let server = HttpServer::new(|| {
        App::new().route("/health_check", web::get().to(health_check))
    })
    .listen(listener)?
    .run();

    Ok(server)
}
```

| Design choice                    | Testing/operations benefit                       |
| -------------------------------- | ------------------------------------------------ |
| caller supplies `TcpListener`    | ephemeral ports and pre-bound sockets            |
| construction returns `Server`    | test can spawn and control lifetime              |
| startup is separate from routes  | configuration errors fail before serving         |
| health check avoids dependencies | distinguishes process health from deep readiness |

Define **liveness** and **readiness** deliberately. A liveness probe should not restart
a healthy process just because a downstream database briefly fails; readiness can stop
new traffic when the service cannot fulfill requests.

#### Configuration Is Parsed Input

```rust
use serde::Deserialize;

#[derive(Debug, Deserialize)]
struct Settings {
    application: ApplicationSettings,
    database: DatabaseSettings,
}

#[derive(Debug, Deserialize)]
struct ApplicationSettings {
    host: String,
    port: u16,
    base_url: String,
}

#[derive(Debug, Deserialize)]
struct DatabaseSettings {
    host: String,
    port: u16,
    database_name: String,
    username: String,
}
```

Treat configuration like source code:

| Stage      | Configuration equivalent                         |
| ---------- | ------------------------------------------------ |
| lex/parse  | deserialize file/environment values              |
| type-check | ports are numbers; URLs and enums are valid      |
| link       | referenced files, secrets, and hosts exist       |
| execute    | initialize dependencies and start listeners      |
| diagnostic | name the field, source, bad value class, and fix |

Secrets should use a wrapper that avoids accidental `Debug`/log output. Validate all
non-secret settings at startup so failures appear before the service accepts traffic.

#### One Application Object Owns Startup

```rust
struct Application {
    port: u16,
    server: actix_web::dev::Server,
}

impl Application {
    fn port(&self) -> u16 {
        self.port
    }

    async fn run_until_stopped(self) -> std::io::Result<()> {
        self.server.await
    }
}
```

The construction path should assemble configuration, database pool, outbound clients,
secrets, middleware, and routes. This creates one place to reason about dependency
lifetimes and startup failures.

#### Integration Tests Should Use Real Boundaries Selectively

```rust
#[tokio::test]
async fn health_check_works() {
    let app = spawn_test_app().await;

    let response = reqwest::get(format!(
        "http://127.0.0.1:{}/health_check",
        app.port
    ))
    .await
    .expect("request failed");

    assert!(response.status().is_success());
    assert_eq!(response.content_length(), Some(0));
}
```

| Test type        | Real component                       | Replace/fake                     |
| ---------------- | ------------------------------------ | -------------------------------- |
| unit             | domain function                      | time, IDs, ports as values       |
| handler          | extraction/response mapping          | use-case service                 |
| API integration  | TCP/HTTP router and database         | external email/payment service   |
| migration        | clean database with migration set    | production data                  |
| end-to-end smoke | deployed artifact and critical route | destructive or expensive actions |

Give each integration test isolated database state. Parallel tests sharing tables create
false failures and conceal ordering dependencies.

#### Transactions Preserve a Use-Case Invariant

```text
begin transaction
  validate current persistent state
  write all related rows
  record an outbox event if external work must follow
commit
```

Do not hold a database transaction open while calling an email server or unrelated
network API. The database cannot atomically roll back the remote side effect. An
**outbox** records the intent in the same transaction; a worker delivers it with an
idempotency key and retry policy.

| Failure question                          | Design tool                            |
| ----------------------------------------- | -------------------------------------- |
| client retries the same request           | idempotency key/unique constraint      |
| process dies after commit                 | transactional outbox                   |
| remote service is temporarily unavailable | bounded retry with backoff             |
| request is canceled while work continues  | explicit job ownership/cancellation    |
| two workers claim one item                | atomic update/locking with state check |

#### Structured Telemetry Carries Causality

```rust
use tracing::{info, instrument};

#[instrument(
    name = "register subscriber",
    skip(repository),
    fields(request_id = %request_id)
)]
async fn register(
    request_id: &str,
    repository: &Repository,
) -> Result<(), RegisterError> {
    repository.insert_pending().await?;
    info!("subscriber stored");
    Ok(())
}
```

| Record                         | Avoid                                   |
| ------------------------------ | --------------------------------------- |
| request/trace ID               | passwords, tokens, raw session cookies  |
| route name and status class    | full sensitive request bodies           |
| sanitized error chain          | silently discarded causes               |
| dependency latency and outcome | high-cardinality secrets as field names |
| deployment/version metadata    | relying only on free-form strings       |

Propagate tracing context into blocking tasks and async work. Logs without causality
become a pile of unrelated sentences under concurrency.

#### Production Boundary Checklist

| Concern       | Minimum deliberate decision                              |
| ------------- | -------------------------------------------------------- |
| timeouts      | connect, request, database, and shutdown deadlines       |
| retries       | which errors, how many, with what backoff/jitter         |
| limits        | body size, concurrency, pool size, queue depth           |
| errors        | stable public response; detailed internal cause          |
| auth/session  | secret rotation, cookie attributes, expiry, revocation   |
| migrations    | forward/backward compatibility during rolling deployment |
| shutdown      | stop new work, drain bounded work, close resources       |
| observability | metrics, traces, sanitized structured logs               |
| supply chain  | locked dependencies, audits, reproducible build artifact |

The service is ready for production when failure paths are designed and observable—not
merely when the happy-path request returns `200`.

### 5. One Architecture Using All Four

```text
Tauri webview
  ↕ typed commands/events
Rust application core
  ├── Diesel repository → project/job metadata
  ├── compiler/runtime → parse, type, execute
  ├── Bevy view/plugin → visualization or interactive simulation
  └── Actix service → remote jobs, health, telemetry
```

| Layer            | Keep independent of                             |
| ---------------- | ----------------------------------------------- |
| domain model     | Tauri command types, Diesel row types, Bevy ECS |
| repository trait | webview and rendering                           |
| compiler core    | database connection and DOM                     |
| adapters         | one another unless orchestration requires it    |

This separation lets a CLI, server, Tauri app, and Bevy visualization reuse the same
compiler core without making the core depend on every framework.

> ➡️ **Next:** practical reversing reads framework and runtime boundaries backward:
> begin with behavior and artifacts, then infer states, layouts, IPC, queries, and calls.

---

## 🔍 Practical x64 Windows Reverse Engineering

This section draws from
[0xZ0F's Reverse Engineering Course](https://github.com/0xZ0F/Z0FCourse_ReverseEngineering),
which progresses from binary fundamentals through x64 assembly, tools, basic reversing,
DLLs, and Windows internals. The course is Windows-focused, but the analysis habits
transfer to other architectures and operating systems.

Use these techniques only on binaries you own, intentionally vulnerable exercises,
open-source programs, or targets you are explicitly authorized to analyze. Run unknown
samples only in an isolated environment designed for that purpose.

> 🔍 **Reversing habit:** separate **observation**, **hypothesis**, and **validation**.
> Decompiler output is evidence, not ground truth.

### 1. Reverse Engineering Is Model Building

The goal is not to translate every instruction into English. The goal is to recover
enough structure to explain relevant behavior.

```text
raw bytes
    ↓ executable-format metadata
instructions + data
    ↓ control-flow and calling-convention analysis
functions + variables + structures
    ↓ behavior and state analysis
high-level model
```

A productive model answers:

| Model dimension | Question                                         |
| --------------- | ------------------------------------------------ |
| Inputs          | What enters the program?                         |
| Transformation  | Which functions change it?                       |
| State           | What memory, files, or globals are read/written? |
| Decisions       | Which branches control behavior?                 |
| Dependencies    | Which OS or library services are called?         |
| Effects         | What output or side effect is produced?          |

Reverse engineering is iterative. Names and types begin as guesses and become more
precise as evidence accumulates.

### 2. Start with Binary Triage

Before debugging, collect facts without executing the target:

| Question                        | Evidence to inspect                                 |
| ------------------------------- | --------------------------------------------------- |
| What format is it?              | PE headers, magic bytes, architecture               |
| 32-bit or 64-bit?               | Machine type and optional-header format             |
| Native or managed?              | Imports, metadata directories, runtime dependencies |
| What does it import?            | DLL names and imported symbols                      |
| What does it export?            | Export table and symbol names                       |
| Are symbols/debug data present? | PDB path, symbol table, debug directory             |
| Does it contain useful text?    | UTF-8/ASCII and UTF-16 strings                      |
| Is it signed or versioned?      | Signature and version resources                     |
| Does layout look ordinary?      | Sections, permissions, sizes, entropy               |

Record a cryptographic hash before analysis so notes always refer to one exact file.

#### PE Mental Model

```text
DOS header / stub
    ↓ points to
PE signature + COFF header
    ↓
optional header
    ├── entry point
    ├── image base
    ├── alignment
    └── data directories
section table
    ├── code
    ├── read-only data
    ├── writable data
    ├── imports/exports
    └── resources/relocations
```

The term **optional header** is historical; executable images rely on it.

| PE coordinate/size  | Meaning                                      |
| ------------------- | -------------------------------------------- |
| **Raw/file offset** | Location in the file on disk                 |
| **RVA**             | Offset from the loaded image base            |
| **VA**              | Runtime address, normally `image_base + RVA` |
| **Raw size**        | Bytes stored in the file for a section       |
| **Virtual size**    | Bytes occupied by the section after loading  |

Do not paste a runtime address into a file-offset field without converting it through
the appropriate section mapping.

### 3. x64 Register Roles

Register names partly encode width:

| 64-bit | Low 32 bits | Low 16 bits | Low 8 bits |
| ------ | ----------- | ----------- | ---------- |
| `RAX`  | `EAX`       | `AX`        | `AL`       |
| `RBX`  | `EBX`       | `BX`        | `BL`       |
| `RCX`  | `ECX`       | `CX`        | `CL`       |
| `RDX`  | `EDX`       | `DX`        | `DL`       |

| Register(s)              | Common Windows x64 role                  |
| ------------------------ | ---------------------------------------- |
| `RIP`                    | Address of the next instruction          |
| `RSP`                    | Stack pointer                            |
| `RBP`                    | Optional frame/base pointer              |
| `RAX`                    | Integer return register and accumulator  |
| `RCX`, `RDX`, `R8`, `R9` | First integer/pointer argument positions |
| `XMM0`–`XMM3`            | First floating/vector argument positions |
| `XMM0`                   | Floating-point return value              |

Writing a 32-bit general register such as `EAX` clears the upper 32 bits of the
corresponding 64-bit register. Writing only `AX` or `AL` does not.

#### Volatile and Nonvolatile Registers on Windows x64

| Category    | Registers                                      | Meaning at a call                       |
| ----------- | ---------------------------------------------- | --------------------------------------- |
| Volatile    | `RAX`, `RCX`, `RDX`, `R8`–`R11`                | Caller must assume they are overwritten |
| Nonvolatile | `RBX`, `RBP`, `RDI`, `RSI`, `RSP`, `R12`–`R15` | Callee must restore them if used        |

This classification is evidence. A function that stores a value in `RBX` and restores
`RBX` before returning may be keeping that value alive across several calls.

### 4. Flags, Comparisons, and Branches

`cmp left, right` behaves like a subtraction used only to update flags:

```text
left - right → flags
```

| Flag   | Interpretation           |
| ------ | ------------------------ |
| **ZF** | Result was zero          |
| **CF** | Unsigned carry or borrow |
| **SF** | Result sign bit          |
| **OF** | Signed overflow          |
| **PF** | Parity of the low byte   |

Signed and unsigned conditional jumps interpret flags differently:

| Meaning          | Signed jump   | Unsigned jump |
| ---------------- | ------------- | ------------- |
| Equal            | `je` / `jz`   | `je` / `jz`   |
| Not equal        | `jne` / `jnz` | `jne` / `jnz` |
| Greater than     | `jg`          | `ja`          |
| Greater or equal | `jge`         | `jae`         |
| Less than        | `jl`          | `jb`          |
| Less or equal    | `jle`         | `jbe`         |

Choosing `jg` versus `ja` is a clue about whether the compiler treated a value as signed
or unsigned.

### 5. Read Instructions by Effect

Group instruction variants by the effect that matters to the current question:

| Family                     | Mental effect                      |
| -------------------------- | ---------------------------------- |
| `mov`, `movzx`, `movsx`    | Copy or extend a value             |
| `lea`                      | Compute an address-like expression |
| `add`, `sub`, `inc`, `dec` | Arithmetic/state update            |
| `and`, `or`, `xor`, `test` | Bit manipulation or flag test      |
| `cmp` + conditional jump   | Decision                           |
| `push`, `pop`              | Stack update                       |
| `call`, `ret`              | Function control flow              |
| `shl`, `shr`, `sar`        | Shift or scale                     |
| `imul`, `idiv`             | Signed multiply/divide             |

Do not assume `lea` means "load a pointer." Compilers also use it for arithmetic such
as:

```asm
lea eax, [rcx + rcx*4] ; eax = rcx * 5
```

Likewise, `xor eax, eax` is commonly a compact way to set `EAX` to zero.

### 6. Windows x64 Calling Convention

| Calling-convention concern | Windows x64 rule                                     |
| -------------------------- | ---------------------------------------------------- |
| Integer args 1–4           | `RCX`, `RDX`, `R8`, `R9`                             |
| Float/vector args 1–4      | Corresponding `XMM0`–`XMM3` positions                |
| Later arguments            | Passed on the stack                                  |
| Shadow/home space          | Caller reserves 32 bytes                             |
| Stack alignment            | Follows the ABI's 16-byte call-boundary rule         |
| Integer return             | Commonly `RAX`                                       |
| Floating return            | Commonly `XMM0`                                      |
| C++ `this`                 | Usually occupies the first integer argument position |

Conceptual call:

```c
result = transform(context, 10, buffer, length, flags);
```

Possible setup:

```asm
mov  rcx, context       ; argument 1
mov  edx, 10            ; argument 2
mov  r8, buffer         ; argument 3
mov  r9d, length        ; argument 4
mov  [rsp+20h], flags   ; argument 5, after shadow space
call transform
; integer-like result now in RAX
```

Registers may be prepared in any order. Follow definitions reaching the `call`, not the
visual order of setup instructions.

#### Inferring a Prototype

At each call site, record:

1. which argument registers are assigned;
2. which stack slots are written;
3. whether XMM registers are used;
4. what values flow into each position;
5. how the return register is used afterward;
6. whether the same function is called elsewhere with different evidence.

Then propose the narrowest type that explains all observed calls.

### 7. Function Boundaries, Prologues, and Epilogues

A traditional function may:

```text
save nonvolatile registers
allocate stack space
initialize local state
perform work and calls
restore stack/registers
return
```

But optimized functions may omit a frame pointer, reuse stack slots, inline calls, split
into hot/cold regions, or end with a tail call. Do not rely on one prologue byte
pattern.

| Boundary evidence       | Why it helps                                       |
| ----------------------- | -------------------------------------------------- |
| Direct call target      | Names a likely entry point                         |
| Unwind metadata         | Describes a recoverable function range             |
| Exported symbol         | Exposes a callable entry                           |
| Cross-references        | Show other code treating the address as a function |
| Stack setup/restoration | Suggests a coherent frame                          |
| Return/tail-call region | Closes a connected control-flow graph              |

### 8. Static and Dynamic Analysis Work Together

| Static analysis                             | Dynamic analysis                           |
| ------------------------------------------- | ------------------------------------------ |
| Does not execute the target                 | Observes one concrete execution            |
| Shows broad possible control flow           | Shows the path actually taken              |
| Good for strings, imports, cross-references | Good for runtime values and indirect calls |
| Can inspect unreachable or rare paths       | Can reveal decoded or generated data       |
| May struggle with packing/indirection       | Can be timing- or environment-dependent    |

A productive cycle:

```text
static hypothesis
    ↓ choose breakpoint and input
dynamic observation
    ↓ rename/retype/comment
better static model
    ↓
repeat
```

### 9. Debugger Controls and Their Meaning

| Debugger control               | Effect                                           |
| ------------------------------ | ------------------------------------------------ |
| **Breakpoint**                 | Stop before a chosen instruction                 |
| **Step into**                  | Execute and follow a call target                 |
| **Step over**                  | Let a call finish and stop after it              |
| **Step out**                   | Run until the current function returns           |
| **Run to cursor**              | Continue to a selected location                  |
| **Data breakpoint/watchpoint** | Stop when a memory range is accessed or modified |
| **Conditional breakpoint**     | Stop only when a value/count matches a condition |

Before stepping, write down what you expect to change. After stepping, compare the
registers, flags, stack, and relevant memory to that prediction.

#### High-Signal Breakpoint Locations

| Location                   | Question it can answer                     |
| -------------------------- | ------------------------------------------ |
| Known imported function    | Which external service is being used?      |
| Comparison before a branch | Which value decides the path?              |
| State-variable write       | Who changes the state and when?            |
| Function entry             | Which arguments arrive?                    |
| Function return            | How does the caller use the result?        |
| Parser/trust boundary      | How do raw bytes become structured values? |

Breakpoints are most useful when tied to a question.

### 10. Recovering Function Calls

Suppose disassembly prepares a format string and values before calling a printing
function. Work backward from the call:

```text
call known_print
    ↑ argument 4 came from R9
    ↑ argument 3 came from R8
    ↑ argument 2 came from RDX
    ↑ argument 1 came from RCX
```

Then inspect:

| Call-site evidence     | What it reveals                                       |
| ---------------------- | ----------------------------------------------------- |
| String at a pointer    | Format, path, command, or semantic landmark           |
| Register write width   | Likely integer width and zero/sign extension          |
| Stack arguments        | Parameters beyond the first four                      |
| XMM arguments          | Floating-point/vector parameters                      |
| Variadic behavior      | Extra arguments interpreted through a format/contract |
| Shadow/alignment setup | ABI mechanics rather than source variables            |

This often reconstructs a recognizable call without understanding the whole function.

### 11. Recovering Loops

Source keywords disappear, so identify the control-flow shape:

```text
initialize
    ↓
loop header / condition ←──────────┐
    ↓ true                         │
body                              │
    ↓                             │
update ───────────────────────────┘
    ↓ false
exit
```

| Loop clue                   | Likely meaning             |
| --------------------------- | -------------------------- |
| Backward branch             | Control-flow cycle         |
| Preheader initialization    | Initial counter or pointer |
| Increment/decrement         | Induction-variable update  |
| Comparison with a bound     | Loop termination condition |
| Fixed-stride pointer move   | Array/record traversal     |
| Repeated call/memory access | Loop body operation        |

`for`, `while`, and `do-while` loops may compile to nearly identical control-flow
graphs. Recover behavior first; choose high-level syntax later.

Example:

```asm
xor  ebx, ebx       ; i = 0
loop_start:
mov  edx, ebx       ; use i
call process_item
inc  ebx            ; i += 1
cmp  ebx, 10
jl   loop_start     ; signed i < 10
```

Possible reconstruction:

```c
for (int i = 0; i < 10; i++) {
    process_item(i);
}
```

### 12. Recovering Structures from Offset Patterns

Repeated accesses through one base register often describe fields:

```asm
mov eax, [rcx+08h]
add eax, [rcx+0Ch]
mov [rcx+10h], eax
```

Working hypothesis:

```c
struct Unknown {
    unsigned char unknown_00[8];
    int field_08;
    int field_0c;
    int field_10;
};
```

| Structure evidence              | Inference it supports           |
| ------------------------------- | ------------------------------- |
| Access width                    | Field size                      |
| Signed/unsigned branch          | Possible numeric interpretation |
| Pointer dereference             | Pointer-like field              |
| Constructor-like initialization | Initial layout and defaults     |
| Repeated offsets                | Stable field positions          |
| Type-specific imported API      | Likely semantic type            |
| Array stride                    | Element size                    |

Rename `field_08` only when evidence supports a semantic name. Until then, a stable
offset-based name is more honest than a confident guess.

### 13. Arrays and Indexing

An address expression such as:

```asm
mov eax, [rcx + rdx*4]
```

suggests:

```text
base = RCX
index = RDX
element size = 4 bytes
```

Possible source:

```c
value = array[index];
```

Scale factors 1, 2, 4, and 8 directly fit x86 addressing modes. Other structure sizes
may use several instructions or pointer increments.

### 14. DLLs, Imports, and Exports

A Windows DLL is a PE image intended to provide code or data to other modules.

| DLL concept                    | Meaning                                              |
| ------------------------------ | ---------------------------------------------------- |
| **Export table**               | Names/ordinals made available to callers             |
| **Import table**               | External symbols required from other DLLs            |
| **Import address table (IAT)** | Runtime slots containing resolved function addresses |
| **Module base**                | Address where the image is mapped                    |
| **RVA**                        | Offset relative to the module base                   |
| **Loader lock**                | Synchronization context affecting DLL initialization |

For a normal imported call, code may call indirectly through an IAT slot:

```asm
call qword ptr [imported_function_slot]
```

The instruction references a slot in the current module; the loader places the actual
library function address into that slot.

When reconstructing an undocumented exported function:

1. start from its export entry;
2. inspect all known call sites;
3. infer argument positions from the ABI;
4. observe return-value use;
5. identify referenced structures and globals;
6. build a small prototype only after the evidence agrees.

### 15. Tool-Agnostic Analysis Views

Different tools use different names, but the useful views are consistent:

| View               | Question it answers                                    |
| ------------------ | ------------------------------------------------------ |
| Hex/bytes          | What is physically encoded here?                       |
| Disassembly        | Which instructions will execute?                       |
| Decompiler         | What high-level model does the tool infer?             |
| Control-flow graph | Where can execution branch or loop?                    |
| Cross-references   | Who uses this function, string, or address?            |
| Imports/exports    | Which module boundaries are visible?                   |
| Memory map         | Which runtime regions exist and with what permissions? |
| Registers/flags    | What is the current CPU state?                         |
| Stack              | Who called whom, and what values are nearby?           |
| Breakpoints/trace  | What happened during this run?                         |

Treat a decompiler as an editable hypothesis generator. Correct its types, function
boundaries, names, and signatures as evidence improves.

### 16. A Repeatable Authorized Lab

Use a tiny program you compile yourself:

```c
#include <stdio.h>

static int sum_positive(const int *items, int count) {
    int total = 0;
    for (int i = 0; i < count; i++) {
        if (items[i] > 0) {
            total += items[i];
        }
    }
    return total;
}

int main(void) {
    int values[] = {3, -2, 7, 0};
    printf("%d\n", sum_positive(values, 4));
    return 0;
}
```

Practice:

1. compile debug and optimized builds;
2. identify the executable format and architecture;
3. locate the output string or imported print function;
4. find the call to `sum_positive`;
5. map the arguments using the platform ABI;
6. identify the loop's backward edge;
7. infer the 4-byte element stride;
8. identify the signed `> 0` test;
9. compare debug and optimized control flow;
10. annotate the binary until another reader can follow it.

### 17. CTF Challenge Patterns as Safe Practice

The public
[crackmes.one RE CTF 2026 repository](https://github.com/crackmesone/ctf-2026-challenges-public/tree/main)
contains handouts and, for some challenges, source and official writeups. Use only the
published challenge artifacts inside an isolated lab; do not transfer techniques to
systems or binaries you are not authorized to analyze.

| Challenge pattern in the repository | Skill to practice                               |
| ----------------------------------- | ----------------------------------------------- |
| custom virtual machine/compiler     | recover instruction format and VM state         |
| nested executable/library layers    | identify loaders, formats, and handoff points   |
| custom encoding/validation          | derive transforms from observed code            |
| time-derived input                  | distinguish entropy from a bounded search space |
| simulated routers/switches          | reconstruct packet and distributed state models |
| Windows screensaver/registry state  | follow OS-specific configuration and execution  |
| small server artifact               | combine static triage with controlled behavior  |

#### The Custom-VM Workflow

`FlipVM` is especially relevant to creating and reversing languages because the
repository includes a compiler, virtual machine, bytecode-like files, and a syntax
highlighter.

```text
virtual source
    ↓ compiler
custom instruction file
    ↓ VM fetch/decode/execute
virtual registers + memory + syscalls
    ↓
observable validation behavior
```

| Builder question                    | Reverser's mirror question                           |
| ----------------------------------- | ---------------------------------------------------- |
| How are opcodes encoded?            | Which bytes select each operation?                   |
| What is the operand format?         | Where are registers, immediates, and offsets?        |
| How is control flow represented?    | Which fields change the virtual instruction pointer? |
| What state does the VM own?         | Which memory region/struct stores virtual state?     |
| Which operations cross to the host? | Where are virtual syscalls dispatched?               |
| Is code transformed or randomized?  | Which representation remains invariant?              |

Start by documenting the VM format and state transition; do not begin by renaming every
host instruction.

#### Layered Challenge Triage

```text
1. hash and identify the handout
2. read the challenge's supported OS/architecture notes
3. inspect format, sections, imports, strings, and entropy
4. run only in the appropriate isolated snapshot
5. capture one baseline behavior
6. change one input
7. locate the first comparison or transformation affected
8. reconstruct the smallest relevant state machine
9. validate on a fresh input
10. compare with published source/writeup only after your attempt
```

| Evidence            | Record                                            |
| ------------------- | ------------------------------------------------- |
| artifact            | cryptographic hash and repository path            |
| environment         | OS, architecture, loader, arguments, snapshot     |
| static observation  | address/file offset, bytes, xrefs, inferred type  |
| dynamic observation | breakpoint, input, register/memory change         |
| hypothesis          | predicted transformation or state transition      |
| validation          | new case that could have disproved the hypothesis |

The simulated-network challenge is a useful reminder that an IP-looking value may be
part of a **custom model**, not a real protocol implementation. Recover the program's
actual frame layout and handlers rather than assuming standards from familiar names.

> 🛡️ **Lab rule:** keep challenge networking disconnected or routed only to controlled
> local services. Treat bundled executables as untrusted even when they are educational.

### 18. Nightmare: Memory-Corruption Labs & Mitigations

[Nightmare](https://guyinatuxedo.github.io/) organizes binary-exploitation practice
around assembly, reversing, Ghidra, GDB, Python tooling, stack bugs, and common
mitigations. The most durable lesson is not a particular payload: it is learning to
connect a source-level memory bug to machine state and then explain what each mitigation
does—and does **not**—prevent.

> 🧪 **Authorized-lab boundary:** use deliberately vulnerable CTF programs, binaries you
> compiled yourself, or targets with explicit written authorization. Keep them in an
> isolated environment. The goal here is root-cause analysis and defense.

#### Build the Model Before Studying Exploitability

```text
untrusted bytes
    ↓ parse/copy/index
stack or heap object
    ↓ bug violates bounds/lifetime/type invariant
adjacent state may change
    ↓
crash, corrupted output, or altered control/data flow
```

| Evidence layer      | Question                                                       |
| ------------------- | -------------------------------------------------------------- |
| source              | Which check or ownership rule is missing?                      |
| compiler output     | Where are objects, bounds, and branches represented?           |
| ABI/frame           | Which registers and stack slots have calling-convention roles? |
| memory map          | Which pages are readable, writable, or executable?             |
| debugger state      | Which instruction first observes the invalid state?            |
| mitigation metadata | Which hardening features are present in this build?            |

Do not equate “program crashed” with “control flow is exploitable.” Exploitability also
depends on reachable data, overwrite precision, memory layout, mitigations, allocator
behavior, concurrency, and environmental constraints.

#### Common Memory-Bug Families

| Bug class            | Broken invariant                                  | Defensive direction                       |
| -------------------- | ------------------------------------------------- | ----------------------------------------- |
| out-of-bounds write  | index/range lies within allocated object          | checked indexing, slices, fuzzing, ASan   |
| out-of-bounds read   | every read refers to initialized accessible bytes | bounds checks, length-aware parsing, MSan |
| use-after-free       | object outlives every reference                   | ownership, RAII, temporal sanitizers      |
| double free          | exactly one owner performs destruction            | single ownership and invalidation         |
| integer overflow     | size arithmetic represents intended allocation    | checked math before allocation/copy       |
| format-string misuse | data is not interpreted as a format program       | constant format strings, typed formatting |
| uninitialized data   | value is written before it is observed            | initialization, compiler warnings, MSan   |

```c
#include <stdbool.h>
#include <stddef.h>
#include <string.h>

bool copy_name(char out[32], const char *src, size_t len) {
    if (src == NULL || len >= 32) {
        return false;
    }

    memcpy(out, src, len);
    out[len] = '\0';
    return true;
}
```

The length check is part of the API contract. In real code, prefer a destination slice
or a function that receives the actual destination capacity instead of relying on an
array-looking parameter that decays to a pointer.

#### Mitigations Form Layers, Not a Substitute for a Fix

| Mitigation   | What it changes                                 | What remains true                              |
| ------------ | ----------------------------------------------- | ---------------------------------------------- |
| NX/DEP       | writable data pages are normally non-executable | data corruption and code reuse may remain      |
| ASLR         | runtime locations vary                          | information disclosure can weaken it           |
| PIE          | main executable can be relocated                | bug still exists                               |
| stack canary | detects some overwrites before returning        | not every corruption touches the canary        |
| RELRO        | hardens dynamic-link relocation metadata        | other writable data remains                    |
| CFI          | restricts allowed indirect control-flow targets | data-only attacks and logic bugs remain        |
| sanitizers   | expose violations during testing                | production builds may not use full checks      |
| safe Rust    | prevents broad classes in safe code             | `unsafe`, FFI, logic, and resource bugs remain |

Return-oriented programming is a **code-reuse** concept motivated partly by
non-executable data. At a high level, an attacker attempts to compose short existing
instruction sequences while satisfying the ABI, stack alignment, and control-flow
constraints. For defensive study, recognize why NX alone is insufficient and why ASLR,
CFI, shadow stacks, reduced gadget surfaces, and eliminating the original write
primitive matter.

#### Controlled Crash-to-Fix Workflow

```text
1. record source/binary hash, compiler, flags, architecture, and mitigations
2. reproduce with one minimal local input
3. capture the first invalid read/write—not merely the final crash
4. map the instruction back to an object, length, and ownership invariant
5. write a regression test that fails safely
6. fix the source-level invariant
7. rerun warnings, sanitizers, tests, and fuzz cases
8. compare hardened and unhardened builds to understand mitigation evidence
```

| Notebook claim                      | Strong evidence                                 |
| ----------------------------------- | ----------------------------------------------- |
| “overflow begins here”              | sanitizer report plus instruction/memory access |
| “this field controls the branch”    | data-flow trace and changed-input experiment    |
| “ASLR is active”                    | build metadata and changing runtime mappings    |
| “the fix works”                     | regression case plus broader randomized testing |
| “the issue is unreachable remotely” | proven input path and deployment configuration  |

### 19. Authorized Memory Inspection & Tooling Labs

The following sources cover dual-use tools and ideas:

- [Black Hat Rust](https://github.com/skerkour/black-hat-rust/) uses offensive-security
  projects to teach async I/O, trait-based modules, crawling, fuzzing, `no_std`,
  cross-compilation, and security thinking.
- [GameHackingCode](https://github.com/GameHackingBook/GameHackingCode) contains
  chapter-oriented Windows examples for memory, pointers, debugging, scanning, state
  machines, control flow, and graphics.
- The [Cheat Engine wiki](https://wiki.cheatengine.org/index.php?title=Main_Page)
  documents value scans, pointers, structures, debugging, assembly, and Lua automation.

Use these ideas on **your own toy process, an included tutorial target, an offline
open-source sample, or an explicitly authorized assessment**. Do not bypass
anti-cheat/DRM, alter multiplayer software, collect another user's data, conceal
activity, or deploy persistence.

#### Reframe Offensive Projects as General Systems Lessons

| Source topic                 | General-purpose lesson                             | Safe exercise                            |
| ---------------------------- | -------------------------------------------------- | ---------------------------------------- |
| concurrent reconnaissance    | bounded work queues, timeouts, backpressure        | inventory services on a loopback lab     |
| async scanner                | many I/O waits without one thread per socket       | probe ports on containers you started    |
| trait-object modules         | heterogeneous plugins behind one contract          | pluggable file-format inspectors         |
| crawler                      | URL normalization, deduplication, rate limits      | crawl a local documentation server       |
| fuzzing                      | generate inputs that violate parser assumptions    | fuzz your own language lexer/parser      |
| `no_std`/cross-compilation   | runtime dependencies, targets, object layout       | tiny freestanding diagnostic program     |
| memory scanner               | typed representation and search-space refinement   | search a byte buffer or tutorial process |
| state-machine recognition    | infer state from transitions and observable fields | reverse a toy offline game you compiled  |
| graphics/control-flow sample | API boundaries, callbacks, render-loop structure   | instrument your own SDL sample           |

Some source chapters go further into phishing, implants, worms, injection, and
evasion. Those are **not implementation exercises for these notes**. Study them as
threat-model categories: identify trust boundaries, telemetry signals, least privilege,
egress controls, code-signing policy, and incident-response evidence.

#### Bounded Concurrency for an Authorized Inventory Tool

```rust
use std::{future::Future, sync::Arc};
use tokio::sync::Semaphore;

async fn run_bounded<I, F, Fut>(items: I, limit: usize, inspect: F)
where
    I: IntoIterator<Item = String>,
    F: Fn(String) -> Fut + Clone + Send + 'static,
    Fut: Future<Output = ()> + Send + 'static,
{
    let permits = Arc::new(Semaphore::new(limit.max(1)));
    let mut tasks = Vec::new();

    for item in items {
        let permit = Arc::clone(&permits)
            .acquire_owned()
            .await
            .expect("semaphore closed");
        let inspect = inspect.clone();

        tasks.push(tokio::spawn(async move {
            inspect(item).await;
            drop(permit);
        }));
    }

    for task in tasks {
        let _ = task.await;
    }
}
```

This is an architectural sketch, not a complete network scanner. A real authorized
inventory tool also needs explicit target allowlists, per-operation deadlines, global
rate limits, cancellation, sanitized logs, bounded results, and a record of who
approved the scope.

| Concurrency control | Failure it prevents                        |
| ------------------- | ------------------------------------------ |
| semaphore           | unbounded in-flight work                   |
| timeout             | a silent peer holding work forever         |
| rate limiter        | overwhelming the target or network         |
| bounded channel     | producers exhausting memory                |
| cancellation token  | abandoned work continuing after scope ends |
| target allowlist    | accidental expansion outside authorization |

#### Memory Scanning Is Iterative Set Filtering

Cheat Engine's core scan model is general:

```text
candidate addresses = every readable, in-scope location
    ↓ filter by type/alignment/value or pattern
smaller candidate set
    ↓ change the toy program's state
filter by changed / unchanged / increased / decreased / new exact value
    ↓ repeat
one or a few hypotheses to validate in the debugger/source
```

| Scan choice           | Meaning                                                |
| --------------------- | ------------------------------------------------------ |
| exact value           | bytes decode to the specified typed value              |
| unknown initial value | begin broadly, then filter by later relationships      |
| changed/unchanged     | compare snapshots without knowing the semantic value   |
| increased/decreased   | use an observed state transition                       |
| array of bytes        | find a byte signature with explicit wildcard positions |
| region filter         | search only mappings consistent with the hypothesis    |

Value type is not decoration. The same bytes can represent an integer, float, pointer,
text fragment, bitfield, or instruction. Track **width, endianness, alignment, signedness,
and encoding**.

#### Safe Scanner Example: Search an Owned Byte Buffer

```rust
fn find_u32_le(haystack: &[u8], wanted: u32) -> Vec<usize> {
    let needle = wanted.to_le_bytes();

    haystack
        .windows(needle.len())
        .enumerate()
        .filter_map(|(offset, bytes)| (bytes == needle).then_some(offset))
        .collect()
}

fn retain_changed(
    old: &[u8],
    new: &[u8],
    candidates: &mut Vec<usize>,
    width: usize,
) {
    candidates.retain(|&offset| {
        let end = match offset.checked_add(width) {
            Some(end) if end <= old.len() && end <= new.len() => end,
            _ => return false,
        };
        old[offset..end] != new[offset..end]
    });
}
```

This teaches the algorithm without opening another process. Extend it with typed
decoders, alignment filters, masked patterns, and snapshot labels. Keep every arithmetic
operation checked because scanners operate near buffer boundaries.

#### Stable Addressing: Modules, Offsets, and Pointer Paths

A runtime address can change between launches because allocations and module bases move.
Record addresses symbolically:

```text
module base + relative virtual address
heap object reached through: stable root → field offset → field offset
```

| Observation                    | Interpretation to test                        |
| ------------------------------ | --------------------------------------------- |
| same module-relative offset    | likely code/static object within one build    |
| new heap address every launch  | allocation is dynamic                         |
| pointer chain survives restart | path may reflect stable object relationships  |
| chain fails after update       | layout/build changed; never assume permanence |
| many matching pointers         | candidates, not proof of semantic ownership   |

A pointer scan searches paths through stored addresses and offsets, then rescans after
the target moves to eliminate paths that no longer resolve. This is graph search:

```text
node = address-sized value/location
edge = dereference plus bounded field offset
goal = currently observed target object
```

Limit depth, offset range, regions, and alignment. Validate across multiple clean runs
of the owned target. A surviving path is evidence, not a stable public API.

#### From Value to Structure

Once two or more fields appear near one another, propose a layout without naming it too
early:

```text
object candidate
  +0x00  unknown pointer-sized field
  +0x08  observed f32, changes with x movement
  +0x0C  observed f32, changes with y movement
  +0x10  observed u32, bounded small range
```

| Validation experiment            | What it can reveal                          |
| -------------------------------- | ------------------------------------------- |
| watch instruction reads field    | access width and consumer                   |
| compare several object instances | repeated stride and shared layout           |
| change one controlled input      | candidate field/state relationship          |
| inspect constructor/init writes  | defaults, ownership, and object extent      |
| restart with symbols/debug build | compare inference to ground truth afterward |

Use offset-based names such as `field_0c_f32?` until evidence supports a semantic name.
This is the same discipline used when recovering an undocumented bytecode VM.

#### Debugger, Scanner, and Static Tool Answer Different Questions

| Tool/view            | Best question                                         |
| -------------------- | ----------------------------------------------------- |
| value scanner        | Where might this changing representation be stored?   |
| memory map           | Which regions could legitimately contain it?          |
| “who accesses” watch | Which instructions consume this address?              |
| breakpoint/trace     | What state led to this instruction?                   |
| disassembler         | What operations and branches exist?                   |
| decompiler           | What higher-level hypothesis fits those instructions? |
| symbols/source       | What was the intended contract?                       |

Correlate at least two evidence types before committing to a conclusion. A changing
number may be a display cache, previous-frame copy, network serialization buffer, or
derived value rather than authoritative state.

#### Array-of-Bytes Signatures Are Version-Specific Hypotheses

```text
exact opcode bytes + wildcard relocation/immediate bytes + surrounding context
```

| Weak signature                    | Stronger direction                           |
| --------------------------------- | -------------------------------------------- |
| too few common bytes              | add stable neighboring instruction structure |
| includes absolute addresses       | wildcard relocation-dependent fields         |
| matches many functions            | include unique semantic context              |
| assumes one compiler optimization | test debug/release and multiple builds       |
| silently selects first match      | require exactly one validated match          |

Signatures are useful for diagnostics and regression tooling on your own builds, but
they are brittle. Prefer exported APIs, symbols, debug information, or explicit
instrumentation when you control the program.

#### Cheat Tables and Lua Are Executable Content

The Cheat Engine wiki documents a broad Lua API for scanning, processes, memory,
debugging, forms, and automation. A table can contain scripts, so treat an untrusted
table like an untrusted executable:

| Before opening/running a table | Defensive action                              |
| ------------------------------ | --------------------------------------------- |
| unknown source                 | do not execute; inspect in an isolated lab    |
| embedded Lua/assembler         | review every enabled script and callback      |
| bundled binary/data            | hash and inventory it                         |
| process attachment             | confirm the exact tutorial/owned target       |
| file/network operations        | deny or isolate unless explicitly required    |
| kernel/DBVM feature            | avoid for ordinary learning; system-wide risk |

Prefer plain notebook observations over downloading opaque tables. Automation should
make a reproducible experiment safer, not hide what it changes.

#### Language-Builder Connection

Memory tools reveal why a language runtime needs explicit representation contracts:

| Runtime choice              | Reversing/tooling consequence                       |
| --------------------------- | --------------------------------------------------- |
| tagged value format         | type bits distinguish integer/object/immediate      |
| moving garbage collector    | raw addresses are unstable across collections       |
| handle table                | stable IDs map to movable/native objects            |
| bytecode dispatch loop      | repeated fetch/decode state exposes VM structure    |
| debug metadata              | source spans and names make diagnostics trustworthy |
| capability-based FFI        | scripts cannot forge arbitrary native pointers      |
| deterministic serialization | snapshots and replay traces become comparable       |

When building your own language, add a **read-only debug protocol** that exposes object
IDs, types, safe field summaries, stacks, and source spans. This provides legitimate
introspection without requiring arbitrary process-memory access.

### 20. Reverse-Engineering Notebook Template

```text
Target hash:
Architecture / ABI:
Image base:
Tool versions:

Question:
Observation:
Hypothesis:
Evidence for:
Evidence against:
Confidence:
Next experiment:

Address/RVA:
Proposed name:
Proposed prototype/type:
Callers:
Callees:
Important offsets:
Side effects:
```

Separating observation from inference prevents early guesses from becoming invisible
assumptions.

### 21. Common Beginner Traps

| Trap                                | Better habit                                           |
| ----------------------------------- | ------------------------------------------------------ |
| Read assembly top-to-bottom         | Follow control flow and data dependencies              |
| Treat stack adjustments as locals   | Separate ABI mechanics from source values              |
| Forget the calling convention       | Annotate argument, return, and preserved registers     |
| Treat every hex value as a pointer  | Classify it from use and valid address ranges          |
| Mix RVA/file offset/VA              | Write down the coordinate system                       |
| Trust decompiler names/types        | Verify against instructions and call sites             |
| Rename unknown fields early         | Keep stable offset-based names until evidence improves |
| Ignore width/signedness             | Track each access width and branch interpretation      |
| Step into every library call        | Stay centered on the analysis question                 |
| Inspect one call site               | Compare multiple callers and inputs                    |
| Expect source-shaped optimized code | Recover behavior before syntax                         |
| Save screenshots without context    | Record addresses, inputs, hashes, and conclusions      |

---

## 🔨 Developer Tooling: The Missing Semester

[The Missing Semester of Your CS Education](https://missing.csail.mit.edu/) treats tool
proficiency as a core computing skill: shells, development environments, debugging,
profiling, version control, packaging, automation, and code quality.

> ⌨️ **Tooling principle:** if a workflow is repeated, error-prone, or difficult to
> explain, make its inputs, command, output, and failure status reproducible.

### 1. Terminal, Shell, and Program

| Term     | Meaning                                                      |
| -------- | ------------------------------------------------------------ |
| Terminal | User interface that displays text and sends input            |
| Shell    | Language/runtime that parses commands and launches programs  |
| Program  | Executable selected by a path or `$PATH` lookup              |
| Process  | Running program with memory, descriptors, environment, state |
| Job      | Shell-managed pipeline/process group                         |

The shell is itself a small programming language. It has parsing, quoting, expansion,
variables, control flow, exit status, and composition—many of the same concerns as a
language you build yourself.

### 2. Paths and Command Resolution

| Path form    | Interpretation                                 |
| ------------ | ---------------------------------------------- |
| `/a/b`       | Absolute path from filesystem root             |
| `a/b`        | Relative to the current working directory      |
| `.`          | Current directory                              |
| `..`         | Parent directory                               |
| `~`          | Shell expansion for a user's home directory    |
| command name | Searched in the ordered directories of `$PATH` |
| `./tool`     | Exact path; bypasses `$PATH` lookup            |

Useful inspection:

```sh
pwd
command -v rustc
type cargo
printf '%s\n' "$PATH"
```

Quote variable expansions unless intentional splitting/globbing is required:

```sh
input_file="My Program.tl"
cargo run -- "$input_file"
```

The `--` convention tells many programs that later values are operands rather than
options. This matters when an untrusted filename begins with `-`.

### 3. Standard Streams and Composition

| Descriptor | Stream | Conventional role           |
| ---------- | ------ | --------------------------- |
| `0`        | stdin  | Program input               |
| `1`        | stdout | Primary machine/user output |
| `2`        | stderr | Diagnostics and errors      |

```text
producer stdout ──pipe──→ consumer stdin
producer stderr ─────────→ terminal/log
```

```sh
compiler program.tl 2>diagnostics.log \
  | disassembler \
  | sort \
  | uniq -c
```

| Operator   | Effect                               |
| ---------- | ------------------------------------ |
| `a \| b`   | Connect `a` stdout to `b` stdin      |
| `> file`   | Replace file with stdout             |
| `>> file`  | Append stdout                        |
| `2> file`  | Redirect stderr                      |
| `< file`   | Read stdin from file                 |
| `tee file` | Copy stdin to both stdout and a file |

Design CLI tools so normal output is composable on stdout and diagnostics go to stderr.
A quiet machine-readable mode makes automation more reliable than scraping decorative
terminal output.

### 4. Exit Status Is Part of the Interface

By convention, exit status `0` means success and a nonzero status means some form of
failure.

```sh
if cargo test; then
  echo "tests passed"
else
  echo "tests failed" >&2
  exit 1
fi
```

| CLI outcome                | Suggested behavior                               |
| -------------------------- | ------------------------------------------------ |
| Successful result          | Print result; exit `0`                           |
| User/input error           | Explain on stderr; exit nonzero                  |
| Internal invariant failure | Preserve diagnostics; exit nonzero               |
| Partial results            | Define explicitly; do not silently claim success |
| Broken pipe                | Handle according to CLI convention               |

### 5. Safer Shell Scripts

```bash
#!/usr/bin/env bash
set -euo pipefail

work_dir="$(mktemp -d)"
cleanup() {
  rm -rf -- "$work_dir"
}
trap cleanup EXIT

input="${1:?usage: check-program INPUT}"
output="$work_dir/program.out"

cargo run -- "$input" >"$output"
wc -c -- "$output"
```

| Strictness feature | Effect                                                 |
| ------------------ | ------------------------------------------------------ |
| `set -e`           | Exit after many unhandled command failures             |
| `set -u`           | Treat undefined variables as errors                    |
| `set -o pipefail`  | Fail a pipeline when any component fails               |
| `trap ... EXIT`    | Run cleanup on shell exit                              |
| `"$variable"`      | Preserve one argument and suppress accidental globbing |
| `--`               | End option parsing when supported                      |

These flags reduce mistakes but do not make Bash a safe general-purpose language.
Complex data structures, rich error recovery, or large scripts are signs that a Rust,
Python, or other purpose-built program may be clearer.

### 6. Search and Data Wrangling

| Task                    | Tool shape                                  |
| ----------------------- | ------------------------------------------- |
| Search text recursively | `rg PATTERN PATH`                           |
| List candidate files    | `rg --files` or `find`                      |
| Select columns/records  | `awk`                                       |
| Transform text          | `sed`                                       |
| Sort and count          | `sort`, `uniq -c`                           |
| Inspect beginning/end   | `head`, `tail`                              |
| Parse structured data   | Format-aware tool rather than fragile regex |

Example: count opcode names in a textual bytecode dump:

```sh
bytecode-dump program.bc \
  | awk '{print $2}' \
  | sort \
  | uniq -c \
  | sort -nr
```

Each pipeline stage should have one understandable contract. Save intermediate output
when a pipeline becomes hard to debug.

### 7. Environment and Reproducibility

A process inherits an environment from its parent. Environment variables are strings,
not typed configuration.

| Configuration source | Strength                           | Risk                              |
| -------------------- | ---------------------------------- | --------------------------------- |
| CLI argument         | Explicit and visible in invocation | May appear in process listings    |
| Environment variable | Convenient for deployment          | Untyped; easy to inherit silently |
| Config file          | Structured, reviewable, persistent | Needs discovery/version rules     |
| Secret store         | Access-controlled sensitive values | External dependency               |
| Compiled default     | Simple fallback                    | Requires rebuild to change        |

Precedence should be documented:

```text
explicit CLI > environment > project config > user config > built-in default
```

Never print all environment variables into shared logs; they often contain credentials
and tokens.

### 8. Debugging as Hypothesis Testing

```text
reproduce
    ↓ minimize
observe one boundary
    ↓ form hypothesis
predict a new result
    ↓ test
fix cause
    ↓ add regression test
```

| Evidence source        | Question answered                                      |
| ---------------------- | ------------------------------------------------------ |
| Logs/traces            | Which events and state transitions occurred?           |
| Debugger               | What are registers, variables, frames, and memory now? |
| System-call trace      | Which kernel services were requested?                  |
| Network capture        | Which bytes crossed the network boundary?              |
| Compiler intermediate  | Which pipeline stage first became wrong?               |
| Core dump/crash report | What state existed at failure?                         |

Do not add random logging everywhere. Put observations at representation boundaries:
tokens, AST, typed IR, bytecode, FFI, file format, network frame, or system call.

### 9. Profiling: Measure the Correct Resource

| Profile target     | Useful measurement                          |
| ------------------ | ------------------------------------------- |
| CPU time           | Samples, call stacks, instruction hotspots  |
| Wall-clock latency | End-to-end timing and blocking waits        |
| Allocation         | Count, size, lifetime, and call site        |
| I/O                | Bytes, operations, queueing, and wait time  |
| Cache behavior     | Misses, branch behavior, locality           |
| Concurrency        | Lock contention, task wait, scheduler delay |

Benchmark optimized builds and realistic inputs. Debug builds can distort compiler,
parser, and allocator performance dramatically.

```rust
use std::time::Instant;

let start = Instant::now();
let result = compile_program(source)?;
let elapsed = start.elapsed();
eprintln!(
    "compiled {} bytes into {} bytes in {elapsed:?}",
    source.len(),
    result.len()
);
```

One timing is a clue, not a benchmark. Warmup, input distribution, system load, and
variance all matter.

### 10. Git as a Content Graph

Git is easier to reason about as immutable objects and references:

```text
working tree
    ↓ git add
index / staging area
    ↓ git commit
commit object → tree → blobs
    ↑
branch reference
```

| Git concept   | Mental model                                 |
| ------------- | -------------------------------------------- |
| Commit        | Snapshot plus parent(s) and metadata         |
| Branch        | Movable name pointing to a commit            |
| Tag           | Usually a stable name for a commit           |
| Index         | Proposed next snapshot                       |
| Merge         | Commit with multiple parents                 |
| Rebase        | Recreate changes on a different parent chain |
| Detached HEAD | `HEAD` points directly to a commit           |

Use small, coherent commits. A good commit explains one change, includes its tests, and
can be reviewed or reverted without pulling in unrelated work.

### 11. High-Signal Git Tools

| Goal                           | Tool/approach                               |
| ------------------------------ | ------------------------------------------- |
| Inspect exact change           | `git diff`, `git diff --staged`             |
| Understand history             | `git log --graph --decorate --oneline`      |
| Find introduction of a line    | `git blame`, then inspect the commit        |
| Find first bad commit          | `git bisect` with a repeatable test command |
| Temporarily save local changes | `git stash` with care                       |
| Work on another branch nearby  | `git worktree`                              |
| Recover a lost reference       | `git reflog`                                |

Do not rewrite shared history casually. Branch names are references; commits remain
reachable only while some reference or reflog keeps them alive.

### 12. Build Systems and Metaprogramming

A build system is a dependency graph plus commands:

```text
source + grammar + dependencies
    ↓ compiler/build rule
generated parser + object files
    ↓ linker/package rule
artifact
```

| Build-system property | Requirement                                        |
| --------------------- | -------------------------------------------------- |
| Inputs                | Complete and explicit                              |
| Outputs               | Predictable paths and formats                      |
| Dependencies          | Correct edges between generated and consumed files |
| Incrementality        | Rebuild only invalidated nodes                     |
| Reproducibility       | Same declared inputs produce equivalent outputs    |
| Failure               | Stop and preserve the command/error that failed    |

Generated code should have one source of truth, deterministic formatting, and a clear
regeneration command. Check it into version control only when repository policy and
tool availability justify it.

### 13. Packaging and Shipping

| Artifact question | Decision to record                              |
| ----------------- | ----------------------------------------------- |
| Version           | Semantic/compatibility policy                   |
| Target            | OS, architecture, ABI, feature set              |
| Dependencies      | Resolved versions, licenses, provenance         |
| Configuration     | Build-time versus runtime values                |
| Integrity         | Hash/signature and verification path            |
| Debuggability     | Symbols, source maps, build IDs, crash metadata |
| Upgrade           | Migration and rollback behavior                 |

For a language implementation, version the source language, bytecode, package manifest,
runtime ABI, and wire protocols separately when they can evolve independently.

### 14. CI Is an Executable Contract

```text
format → lint → unit tests → integration tests → artifact checks
       → security/license checks → package
```

| CI property          | Good practice                                        |
| -------------------- | ---------------------------------------------------- |
| Reproducible locally | Every CI step has an equivalent local command        |
| Fast feedback        | Cheap deterministic checks run first                 |
| Clear failure        | Log the command, relevant versions, and failing test |
| Controlled secrets   | Expose only to trusted jobs and avoid printing them  |
| Artifact provenance  | Record source revision and toolchain                 |
| Caching              | Cache performance data, not correctness assumptions  |

### 15. Tooling for Your Own Language

| Tool               | Minimum useful capability                          |
| ------------------ | -------------------------------------------------- |
| Formatter          | Deterministic source output                        |
| Parser/AST dumper  | Inspect syntax without executing                   |
| Type checker mode  | Validate without code generation                   |
| IR/bytecode dumper | Stable machine-readable and human-readable forms   |
| Disassembler       | Decode every instruction with offsets              |
| Test runner        | Filters, stable exit status, structured results    |
| Package inspector  | Manifest, dependencies, hashes, compatibility      |
| Debugger           | Break, step, inspect state, explain source mapping |
| Profiler hooks     | Stable function/pass IDs and event timing          |

The tooling interface is part of the language. Stable exit codes, diagnostics, source
locations, and machine-readable output make editors, CI systems, and external tools
possible.

---

## 🐍 Practical Python Automation

[Automate the Boring Stuff with Python, 3rd Edition](https://automatetheboringstuff.com/3e/)
progresses from Python basics and debugging into files, command-line programs, web
scraping, spreadsheets, databases, documents, structured data, scheduling,
notifications, images, OCR, GUI control, and speech.

> 🤖 **Automation mental model:** a script is a tiny data pipeline with side effects.
> Make inputs explicit, validate the transformation, preview the output, then commit the
> external change.

### 1. Choose Automation by Boundary

| Repetitive work             | Prefer                                           |
| --------------------------- | ------------------------------------------------ |
| rename/sort local files     | `pathlib`, metadata, explicit destination plan   |
| edit structured text        | CSV/JSON/XML parser, not ad hoc string splitting |
| collect web data            | documented API first; respectful HTTP otherwise  |
| update spreadsheet values   | workbook library with formula/style awareness    |
| query durable local records | SQLite with parameters and transactions          |
| extract document text       | format-specific PDF/Word library                 |
| schedule a recurring job    | OS scheduler plus idempotent script              |
| notify a person/system      | email/message API with preview and rate limit    |
| manipulate images           | image library with original preserved            |
| drive a GUI                 | last resort; screenshots, focus, and timing vary |

Use the **highest-level stable interface** available:

```text
library/API > documented CLI > file format > browser automation > mouse coordinates
```

### 2. Every Useful Script Has the Same Skeleton

```python
from dataclasses import dataclass
from pathlib import Path


@dataclass(frozen=True)
class Rename:
    source: Path
    destination: Path


def plan_renames(root: Path) -> list[Rename]:
    plans: list[Rename] = []
    for source in sorted(root.glob("*.txt")):
        destination = source.with_name(source.stem.strip().lower() + source.suffix)
        if source != destination:
            plans.append(Rename(source, destination))
    return plans


def validate(plans: list[Rename]) -> None:
    destinations = [plan.destination for plan in plans]
    if len(destinations) != len(set(destinations)):
        raise ValueError("two inputs map to the same destination")
    if any(path.exists() for path in destinations):
        raise FileExistsError("a destination already exists")


def apply(plans: list[Rename], *, dry_run: bool) -> None:
    for plan in plans:
        print(f"{plan.source.name} -> {plan.destination.name}")
        if not dry_run:
            plan.source.rename(plan.destination)
```

| Phase    | Invariant                                                |
| -------- | -------------------------------------------------------- |
| discover | only intended inputs are selected                        |
| parse    | bytes/strings become typed values                        |
| plan     | intended changes are represented without performing them |
| validate | no collision, escape, invalid state, or missing resource |
| preview  | a human or test can inspect the plan                     |
| apply    | each external mutation is checked                        |
| verify   | final state matches the plan                             |
| report   | failures identify the item and operation                 |

### 3. Filesystem Automation Needs a Blast-Radius Limit

```python
from pathlib import Path


def resolved_child(root: Path, user_name: str) -> Path:
    root = root.resolve(strict=True)
    candidate = (root / user_name).resolve()

    if not candidate.is_relative_to(root):
        raise ValueError("path escapes the allowed root")
    return candidate
```

| Risk                          | Safer design                                  |
| ----------------------------- | --------------------------------------------- |
| broad recursive glob          | explicit root, extension, and file-count cap  |
| overwrite destination         | fail unless an explicit overwrite mode exists |
| partial multi-file operation  | plan, journal, and resumable/idempotent steps |
| extension mistaken for format | inspect magic/header and parse defensively    |
| symbolic-link escape          | resolve and re-check containment              |
| lost original                 | backup/version control or recoverable move    |

Start with a temporary directory containing synthetic files. A script that is safe only
when all inputs are perfect is not ready for valuable data.

### 4. Text and Regular Expressions

Regular expressions are small pattern languages. Use them when the input is genuinely
textual and local—not as a substitute for an HTML, JSON, CSV, or programming-language
parser.

```python
import re

LOG_LINE = re.compile(
    r"^(?P<level>INFO|WARN|ERROR)\s+"
    r"request_id=(?P<request_id>[A-Za-z0-9_-]{1,64})\s+"
    r"message=(?P<message>.*)$"
)


def parse_log_line(line: str) -> dict[str, str] | None:
    match = LOG_LINE.fullmatch(line.rstrip("\n"))
    return match.groupdict() if match else None
```

| Regex habit              | Reason                                              |
| ------------------------ | --------------------------------------------------- |
| raw string literal       | reduces double escaping                             |
| named groups             | documents field meaning                             |
| `fullmatch` when parsing | rejects trailing/leading surprises                  |
| bounded repetitions      | limits pathological work and absurd fields          |
| test near-misses         | proves the pattern rejects almost-valid input       |
| compile once             | centralizes the pattern and avoids repeated parsing |

For your own language, use regex for simple tokens only when it keeps precedence and
source spans clear. Balanced nesting and contextual syntax belong in a parser.

### 5. Structured Files Preserve Types and Context

```python
import csv
import json
from pathlib import Path


def csv_to_json(source: Path, destination: Path) -> None:
    with source.open(newline="", encoding="utf-8") as input_file:
        rows = list(csv.DictReader(input_file))

    normalized = [
        {
            "name": row["name"].strip(),
            "score": int(row["score"]),
        }
        for row in rows
    ]

    temporary = destination.with_suffix(destination.suffix + ".tmp")
    temporary.write_text(
        json.dumps(normalized, indent=2, ensure_ascii=False) + "\n",
        encoding="utf-8",
    )
    temporary.replace(destination)
```

| Format | Important edge                                   |
| ------ | ------------------------------------------------ |
| CSV    | dialect, quoting, newline handling, column names |
| JSON   | number precision, missing vs `null`, encoding    |
| XML    | namespaces, entity/security settings, mixed text |
| YAML   | implicit types and unsafe object construction    |

Parse into a validated domain type before acting. Preserve unknown fields only when the
round-trip contract requires them.

### 6. SQLite Turns a Script into a Small Durable System

```python
import sqlite3
from pathlib import Path


def record_run(database: Path, input_name: str, output_hash: str) -> None:
    with sqlite3.connect(database) as connection:
        connection.execute(
            """
            CREATE TABLE IF NOT EXISTS runs (
                input_name TEXT PRIMARY KEY,
                output_hash TEXT NOT NULL
            )
            """
        )
        connection.execute(
            """
            INSERT INTO runs(input_name, output_hash)
            VALUES (?, ?)
            ON CONFLICT(input_name)
            DO UPDATE SET output_hash = excluded.output_hash
            """,
            (input_name, output_hash),
        )
```

Placeholders keep data separate from SQL syntax. The context manager commits on success
and rolls back on an exception.

| Need                  | SQLite feature                                |
| --------------------- | --------------------------------------------- |
| avoid duplicate work  | unique key plus upsert                        |
| multi-step invariant  | transaction                                   |
| evolve stored records | numbered schema migrations                    |
| inspect/debug state   | ordinary queries and a schema                 |
| concurrent automation | short transactions and explicit busy handling |

SQLite is not merely a bigger dictionary. Define a schema, constraints, migration
strategy, backup, and failure behavior.

### 7. Web Automation Starts with Permission and Protocols

```python
import requests


def fetch_json(url: str) -> dict:
    response = requests.get(
        url,
        headers={"User-Agent": "personal-automation/1.0"},
        timeout=(3.0, 15.0),
    )
    response.raise_for_status()
    if "application/json" not in response.headers.get("content-type", ""):
        raise ValueError("unexpected content type")
    return response.json()
```

Before scraping:

| Check                    | Why                                               |
| ------------------------ | ------------------------------------------------- |
| documented API/feed      | more stable and less ambiguous                    |
| terms and authorization  | access does not imply permission to republish     |
| `robots.txt`/site policy | communicates automated-access expectations        |
| authentication boundary  | never bypass login, paywall, or access control    |
| request rate and caching | avoid unnecessary load                            |
| pagination/retry limits  | prevent infinite or explosive work                |
| data minimization        | do not collect personal/sensitive fields casually |

HTML selectors are assumptions about a changing document. Save a small permitted fixture
for parser tests, and fail visibly when required elements disappear.

### 8. Spreadsheets, Documents, and Images Have Hidden Structure

| Artifact    | Preserve/check                                         |
| ----------- | ------------------------------------------------------ |
| spreadsheet | formulas vs cached values, types, merged cells, styles |
| PDF         | pages, coordinates, text extraction order, OCR limits  |
| Word file   | paragraphs, runs, tables, styles, relationships        |
| image       | dimensions, color mode, orientation metadata, format   |
| OCR output  | confidence, reading order, and manual review           |

Never assume that a visually empty spreadsheet cell is absent, that PDF text extraction
matches reading order, or that OCR output is exact. Keep the original artifact and
render the generated result for visual verification when layout matters.

```python
from PIL import Image, ImageOps


def make_thumbnail(source: str, destination: str) -> None:
    with Image.open(source) as image:
        image = ImageOps.exif_transpose(image)
        image.thumbnail((512, 512))
        image.convert("RGB").save(destination, quality=90)
```

### 9. Scheduling Requires Idempotency

```text
scheduler triggers job
    ↓ acquire single-run lock
load checkpoint
    ↓ discover only pending work
perform bounded/idempotent steps
    ↓ atomically store checkpoint
emit summary and exit status
```

| Scheduled-job failure     | Design response                                 |
| ------------------------- | ----------------------------------------------- |
| runs twice                | idempotency key or uniqueness constraint        |
| previous run still active | lock with stale-owner policy                    |
| machine sleeps/offline    | explicit catch-up or skip policy                |
| network fails halfway     | checkpoint and bounded retry                    |
| credentials expire        | actionable alert without leaking the credential |
| output grows forever      | retention and quota policy                      |

The scheduler should invoke a normal command-line program. Keep scheduling policy out
of the transformation core so the same job can be tested manually.

### 10. Notifications Are External Mutations

Separate rendering from sending:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Message:
    recipient: str
    subject: str
    body: str


def build_summary(recipient: str, changed: int) -> Message:
    return Message(
        recipient=recipient,
        subject="Automation summary",
        body=f"Changed items: {changed}\n",
    )
```

Test the `Message` value, preview it, then let a narrow adapter send it. Use recipient
allowlists in test/staging, deduplication keys, rate limits, and a clear distinction
between retryable and permanent delivery failures. Never log credentials or full
sensitive message bodies.

### 11. GUI Automation Is a Fragile Last Resort

Mouse/keyboard automation depends on focus, coordinates, scale, timing, window state,
and accessibility. Prefer a programmatic API. If GUI control is unavoidable:

| Guardrail              | Purpose                                          |
| ---------------------- | ------------------------------------------------ |
| dedicated test profile | protects personal data and settings              |
| visible countdown      | gives the operator time to cancel                |
| fail-safe gesture/key  | stops runaway input                              |
| screenshot assertion   | confirms the expected screen before action       |
| bounded repetitions    | prevents infinite clicking/typing                |
| no irreversible action | require a human confirmation at the final commit |

Automate navigation and data entry; pause before sending, purchasing, deleting, or
publishing unless the workflow has explicit authorization and a verified preview.

### 12. CLI Design Makes Automation Reusable

```text
my-tool INPUT --output OUTPUT --dry-run --format json
```

| CLI contract  | Good behavior                                      |
| ------------- | -------------------------------------------------- |
| arguments     | explicit paths/modes; helpful validation errors    |
| stdout        | requested data, optionally machine-readable        |
| stderr        | diagnostics and progress                           |
| exit status   | stable success/failure meaning                     |
| dry run       | shows planned mutations without applying them      |
| configuration | documented precedence and no secret echo           |
| interruption  | cleans temporary state or leaves resumable journal |

Keep `main()` thin:

```text
parse CLI → load config → call pure planner → validate → preview/apply adapter → report
```

The same architecture works for a compiler driver, asset pipeline, database maintenance
job, or personal file organizer.

### 13. Automation Review Card

| Before the first real run    | After the run                                  |
| ---------------------------- | ---------------------------------------------- |
| synthetic test inputs        | verify counts, hashes, or database constraints |
| narrow input and destination | inspect a sample of outputs                    |
| dry-run plan                 | retain an audit summary                        |
| backup/recovery path         | confirm originals are recoverable              |
| timeout and item-count limit | record failures for retry                      |
| redacted logging             | remove temporary secrets/files                 |
| explicit authorization       | confirm no scope expansion                     |

> 🧭 **Progression:** automate one deterministic transformation first. Then add a CLI,
> dry-run mode, structured logs, durable checkpoints, scheduling, and notifications in
> that order.

---

## 🧰 Practical Low-Level Workflow

> 🧰 **Repeatable loop:** define → isolate → observe → test → explain → automate.

### When Building

1. State the invariant.
2. Define the representation.
3. Validate every boundary.
4. Keep unsafe or platform-specific code isolated.
5. Create a tiny executable example.
6. Inspect intermediate output.
7. Add failure-path and boundary tests.
8. Measure before optimizing.

### When Reversing or Debugging

1. Confirm authorization and isolate the target.
2. Record architecture, operating system, file format, and hashes.
3. Establish one controlled input and one observable output.
4. Locate a landmark: string, symbol, file offset, packet field, or instruction.
5. Trace data flow before guessing intent.
6. Change one variable and repeat.
7. Separate observations from hypotheses.
8. Predict a new result, then validate it.

### Questions That Transfer Across Every Layer

| Dimension       | Transferable question                                  |
| --------------- | ------------------------------------------------------ |
| Representation  | What representation am I looking at?                   |
| Ownership       | Who owns this value or resource?                       |
| Lifetime        | How long is it valid?                                  |
| Validation      | Which invariants have already been checked?            |
| Encoding        | What are byte order, width, alignment, and signedness? |
| Numeric domain  | Address, offset, index, ID, or count?                  |
| Trust           | Where does trust change?                               |
| Partial effects | Can the operation partially succeed?                   |
| State machine   | What state must come before and after it?              |
| Falsifiability  | What evidence would disprove the current model?        |

---

## 🛤️ X. Language Progression

The four stages of building a language from scratch:

> 🛤️ **Progression principle:** grow the language in vertical slices so every stage
> remains runnable, testable, and explainable.

| Feature      | Calculator             | Firstlang   | Secondlang | Thirdlang        |
| ------------ | ---------------------- | ----------- | ---------- | ---------------- |
| Grammar size | ~18 lines              | ~70 lines   | ~77 lines  | ~140 lines       |
| Type System  | None                   | Dynamic     | Static     | Static + Classes |
| Variables    | No                     | Yes         | Yes        | Yes              |
| Functions    | No                     | Yes         | Yes        | Yes + Methods    |
| Classes      | No                     | No          | No         | Yes              |
| Memory       | Stack                  | Stack       | Stack      | Stack + Heap     |
| Execution    | Interpreter / VM / JIT | Interpreter | LLVM JIT   | LLVM JIT         |

Each stage adds one layer of abstraction:

1. **Calculator** — Learn the basics: parsing, AST, evaluation.
2. **Firstlang** — Add programming: variables, functions, control flow.
3. **Secondlang** — Add types: static checking, LLVM compilation.
4. **Thirdlang** — Add OOP: classes, objects, heap memory management.

**Grammar growth, side by side** — each stage layers new pest rules onto the last
without disturbing the expression rules underneath:

```pest
# Firstlang — statements, functions, control flow
Stmt = { Function | Return | Assignment | Expr }
Function = { "def" ~ Identifier ~ "(" ~ Params? ~ ")" ~ Block }
Conditional = { "if" ~ "(" ~ Expr ~ ")" ~ Block ~ "else" ~ Block }
WhileLoop = { "while" ~ "(" ~ Expr ~ ")" ~ Block }
```

```pest
# Secondlang — same shape, with type annotations bolted on
Type = { IntType | BoolType }
TypedParam = { Identifier ~ ":" ~ Type }
ReturnType = { "->" ~ Type }
Function = { "def" ~ Identifier ~ "(" ~ TypedParams? ~ ")" ~ ReturnType? ~ Block }
```

```pest
# Thirdlang — classes and object operations
ClassDef = { "class" ~ Identifier ~ "{" ~ ClassBody ~ "}" }
FieldDef = { Identifier ~ ":" ~ Type }
MethodDef = { "def" ~ Identifier ~ "(" ~ SelfParam ~ "," ~ Params? ~ ")" ~ ReturnType? ~ Block }
NewExpr = { "new" ~ Identifier ~ "(" ~ Args? ~ ")" }
Delete = { "delete" ~ Expr }
```

> The expression rules (`Expr`, `Comparison`, `Additive`, etc.) barely change across all
> four stages — growth comes from _statements_ and _declarations_, not from how
> arithmetic is parsed.

---

### What to Explore Next

| Direction             | Topics                                                                                                                             |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Language Features** | Inheritance (vtables, dynamic dispatch), interfaces/traits, generics (`List<T>`), closures, pattern matching, algebraic data types |
| **Type System**       | Nullability (`Point?`), reference vs owned types, Hindley-Milner full inference                                                    |
| **Memory Management** | Garbage collection (mark-and-sweep, generational), reference counting, Rust-style ownership                                        |
| **Execution Models**  | AOT compilation to executables, bytecode VMs, transpilation to JS/C/WASM                                                           |
| **Optimizations**     | Inlining, escape analysis (stack-allocate short-lived objects), devirtualization                                                   |
| **Tooling**           | Debuggers, formatters, Language Server Protocol (LSP) for IDE support                                                              |

**Sketching a few of these directly on Thirdlang:**

```
# Inheritance — needs vtables + dynamic dispatch
class Animal { def speak(self) -> int { return 0 } }
class Dog extends Animal { def speak(self) -> int { return 1 } }  # override

# Interfaces/traits — polymorphism without inheritance
trait Printable { def print(self) -> int }
impl Printable for Point { def print(self) -> int { return self.x } }

# Closures — capture the enclosing scope
def make_adder(n: int) -> (int) -> int {
    return def(x: int) -> int { return x + n }
}
add5 = make_adder(5)
add5(10)   # 15

# Pattern matching + algebraic data types
enum Option<T> { Some(T), None }
match point {
    Point { x: 0, y } => "on y-axis",
    Point { x, y: 0 } => "on x-axis",
    _ => "elsewhere",
}
```

> Generics require either **monomorphization** (generate specialized code per concrete
> type — Rust's approach) or **type erasure** (use runtime type info instead — Java's
> approach).

**A minimal debugger** works the same way as our interpreter, just paused: it steps
through the AST/bytecode one node at a time and lets you inspect the environment at each
`break`:

```
(debug) break main.tl:10
(debug) run
Breakpoint hit at main.tl:10
(debug) print x
x = 42
(debug) step
```

**Real-world Rust language projects worth reading**, roughly in order of complexity:
start with smaller, approachable codebases like **[Koto](https://koto.dev/)** or
**[Rhai](https://rhai.rs/)**, then graduate to more advanced implementations like
**[Gleam](https://gleam.run/)** or **[Boa](https://boajs.dev/)** (a JS engine). They use
the same techniques covered here — grammars, ASTs, type systems — at production scale.

The concepts you've learned — grammars, ASTs, type systems, code generation — appear
everywhere: SQL, GraphQL, YAML, regex, CSS, template engines. You now have the
foundation to understand, modify, or create any of them.

---

_All paths lead to the same truth: computing is structured transformation — from text,
to trees, to bytes, to machine code, to electrons._
