# Programming & Compiler Mental Notes

> A practical map from source text to syntax trees, bytecode, machine code, memory, and
> network protocols—with the same ideas viewed from both the builder's and reverse
> engineer's perspective.

## How to Use These Notes

These notes work best as a reference rather than a book that must be read strictly front
to back:

- **Building a language:** follow grammar → AST → interpreter → types → IR → code
  generation → runtime.
- **Learning low-level systems:** follow CPU → assembly → memory → executable formats →
  ABI → networking.
- **Reverse engineering:** read the same pipeline backward, starting with bytes,
  instructions, runtime state, and observable behavior.
- **Debugging:** reduce the problem to one stage, inspect that stage's input and output,
  and record the smallest reproducible example.

### Quick Navigation

| Goal                   | Start here                                                                  | Then continue to                                                                     |
| ---------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Build a parser         | [Grammar](#grammar)                                                         | [Building a Parser](#building-a-parser)                                              |
| Build an interpreter   | [Execution Pipeline](#iii-the-execution-pipeline--ast)                      | [Bytecode & VMs](#iv-bytecode--virtual-machines)                                     |
| Add static types       | [Types & Type Inference](#v-types--type-inference)                          | [LLVM IR](#vii-llvm-ir--advanced-code-generation)                                    |
| Understand memory      | [Memory Layout](#memory-layout)                                             | [Low-Level Memory](#low-level-memory-binary-analysis--language-runtime-architecture) |
| Reverse a binary or VM | [Reverse Engineering](#reverse-engineering--debugging-compiled-binaries)    | [Reversing a Binary](#7-reversing-a-binary-bytecode-vm-or-language)                  |
| Write safer Rust tools | [Idiomatic Rust](#idiomatic-rust-for-compilers-parsers-and-low-level-tools) | [Networking](#networking-protocols--wire-formats)                                    |

### Reading Conventions

- **Definition** introduces a concept.
- **Mental model** gives a useful approximation, not a formal guarantee.
- **Invariant** is a condition that must remain true for an implementation to be
  correct.
- **Boundary** is where data changes representation or trust level.
- **Failure mode** names what can go wrong and why.
- Code examples favor clarity and explicit checks over cleverness.

---

## Grammar

A _**formal grammar**_ is a set of rules that define what makes valid code in a
programming language. Think of it like the grammar of English: “The cat sat” is valid,
but “Cat the sat” is not.

For programming languages, a grammar specifies:

- **What tokens are valid** - keywords like `def`, `if`, `return` ; _operators_ like
  `+`, `-`, `*`; _literals_ like `42`, `true`
- **How tokens can be combined** - `1 + 2` is valid, `+ + 1` is not
- **The structure of programs** - functions contain statements, statements contain
  expressions

---

## I. Hardware Fundamentals

### The CPU

If you want to create a “computer” from scratch, you need to start by defining an
**abstract model** for your computer. This _abstract model_ is also referred to as
_**Instruction Set Architecture (ISA)**_

A **CPU** is an implementation of an **Instruction Set Architecture (ISA)** — an
abstract model defining data types, registers, hardware support, and I/O. Together these
make up **Machine Language**, the lowest-level language of computing.

The CPU continuously runs a **fetch-decode-execute** loop:

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

## II. The Language Hierarchy & Translation

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

---

## Building a Parser

A parser takes raw source code (a string) and converts it into a structured **Abstract
Syntax Tree (AST)** that a computer can evaluate or compile. Think of source code as a
sentence and the AST as its grammatical diagram. It captures _structure_, not just text.

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

|      Syntax      |             Meaning             |       Syntax        |      Meaning       |
| :--------------: | :-----------------------------: | :-----------------: | :----------------: |
| `foo = { ... }`  |          regular rule           |  `baz = @{ ... }`   |       atomic       |
| `bar = _{ ... }` |             silent              |  `qux = ${ ... }`   |  compound-atomic   |
|   `#tag = ...`   |              tags               | `plugh = !{ ... }`  |     non-atomic     |
|     `"abc"`      |          exact string           |      `^"abc"`       |  case insensitive  |
|    `'a'..'z'`    |         character range         |        `ANY`        |   any character    |
|   `foo ~ bar`    |            sequence             |    `baz \| qux`     |   ordered choice   |
|      `foo*`      |          zero or more           |       `bar+`        |    one or more     |
|      `baz?`      |            optional             |      `qux{n}`       |    exactly _n_     |
|   `qux{m, n}`    | between _m_ and _n_ (inclusive) |                     |                    |
|      `&foo`      |       positive predicate        |       `!bar`        | negative predicate |
|   `PUSH(baz)`    |         match and push          | `PUSH_LITERAL("a")` | push without match |
|      `POP`       |          match and pop          |       `PEEK`        | match without pop  |
|      `DROP`      |      pop without matching       |     `PEEK_ALL`      | match entire stack |

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

### The Compiler

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

---

### Virtual Machines (VMs)

Hardware instructions are vendor-specific — Intel and AMD instructions differ. A
**Virtual Machine (VM)** abstracts away hardware details so that code compiled to the
VM's language becomes **platform-agnostic**.

The most famous example: the **Java Virtual Machine (JVM)**. Any valid Java Bytecode
runs on any platform with a **Java Runtime Environment (JRE)**, regardless of where it
was compiled.

---

## III. The Execution Pipeline & AST

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

- **Variables** — Named storage (`n`).
- **Functions** (`fib`) — Abstraction and reuse.
- **Parameters** (passing `n`) — Data flow.
- **Conditionals** (`if`/`else`) — Branching.
- **Operators** (`<`, `-`) — Computation.
- **Recursion** (`fib` calls `fib`) — Self-reference.
- **Call stack** (tracks each frame) — Memory management.

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

## Recursion

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

## IV. Bytecode & Virtual Machines

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

## V. Types & Type Inference

### Static vs. Dynamic Typing

Types act as **contracts** — when you write `a: int`, you promise `a` will always be an
integer. The compiler enforces that promise.

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

## VI. Compiler Optimizations

Optimizations simplify the AST before code generation, producing faster output, cleaner
debug trees, and less work for the backend. They are chained into a **pass pipeline** —
order matters!

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

## VII. LLVM IR & Advanced Code Generation

> LLVM is like a universal translator for CPUs. You speak LLVM IR; LLVM translates it to
> x86, ARM, WebAssembly — whatever you need. Write your compiler frontend once; LLVM
> gives you every platform for free.

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

## VIII. Object-Oriented Concepts (Classes)

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

## IX. Debugging

### Compiler Pipeline Debugging

> Your language is a pipeline: `Source → Tokens → AST → Output`. When something breaks,
> find which stage produced the wrong output.

**Systematic approach:**

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
everything in [Hardware Fundamentals](#i-hardware-fundamentals): instead of going source
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

## Low-Level Memory, Binary Analysis & Language Runtime Architecture

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

The bytes do not change; the interpretation does. This is the core idea behind:

- decoding instructions in a disassembler;
- parsing an executable or packet header;
- interpreting a tagged value in a VM;
- viewing a runtime object in a debugger;
- reverse engineering an unknown file format.

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

- **Leak** — allocated memory becomes unreachable but is never released.
- **Double free** — one allocation is released twice.
- **Use after free** — a stale pointer is used after its allocation ended.
- **Out-of-bounds access** — an index or pointer escapes the allocation.
- **Uninitialized read** — bytes are interpreted before being given a valid value.
- **Data race** — threads access shared state without required synchronization.

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

Parser checklist:

- bound every length and collection count;
- use `checked_add`, `checked_mul`, and checked slicing;
- reject unknown mandatory tags and invalid enum values;
- decide whether trailing bytes are allowed;
- cap recursion depth and decompressed output size;
- never assume one read returns an entire file or network message;
- preserve byte offsets in errors so malformed inputs are debuggable;
- fuzz the parser with arbitrary byte strings.

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

Important platform mitigations:

- **Stack canaries** detect some stack overwrites before a function returns.
- **NX/DEP** marks data pages non-executable.
- **ASLR** randomizes important memory regions between runs.
- **PIE** allows the main executable to be relocated.
- **RELRO** makes selected relocation data read-only after linking.
- **Control-flow integrity** restricts indirect branches to valid targets.

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

Compiler/runtime testing ideas:

- build with warnings enabled and treat them seriously;
- use sanitizers on C/C++ runtime components;
- fuzz lexer, parser, bytecode verifier, deserializer, and protocol decoder;
- test zero, maximum, and one-past-maximum lengths;
- run malformed bytecode through a verifier before execution;
- keep `unsafe` code small and document its invariants.

---

### 7. Reversing a Binary, Bytecode VM, or Language

Source compilation and reverse engineering travel in opposite directions:

```text
source → tokens → AST → typed IR → machine IR → assembly → bytes
bytes  → instructions → control-flow graph → data-flow facts → hypotheses
```

Information is lost during compilation:

- variable and function names may disappear;
- comments and formatting disappear;
- multiple source forms optimize to the same instructions;
- inlining erases a call boundary;
- constant folding erases the original expression;
- types may be only partially recoverable.

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

If you want your own language tools to be easy to debug:

- include a bytecode version and magic number;
- keep a documented section table;
- provide source maps and optional symbol names;
- ship an official bytecode dumper/disassembler;
- separate verification from execution;
- make unknown versions fail closed;
- use deterministic output when possible.

---

## Idiomatic Rust for Compilers, Parsers, and Low-Level Tools

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

Good newtype candidates:

- byte offset vs. virtual address;
- source line vs. source byte;
- token ID vs. AST node ID;
- VM register vs. local slot;
- host-order vs. network-order integer;
- compressed size vs. uncompressed size.

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

Error messages for a compiler should answer:

- what went wrong;
- where it happened;
- what was expected;
- what was found;
- what the user can do next.

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

Useful review questions:

- Who allocated this memory?
- How long is it valid?
- Is it initialized for this type?
- Is the pointer aligned?
- Is aliasing permitted?
- Can another thread mutate it?
- How is its length proven?

### 8. Rust Anti-Patterns to Avoid

- Cloning an AST subtree just to satisfy a borrow error without understanding ownership.
- Accepting `&String` or `&Vec<T>` when `&str` or `&[T]` is enough.
- Using `usize` for every ID, offset, and size even when the concepts differ.
- Calling `unwrap()` on input-dependent parsing or network operations.
- Indexing byte slices before validating their lengths.
- Mapping malformed input to a generic "failed" message with no offset.
- Holding a mutex guard across a blocking operation or `.await`.
- Exposing all internal AST fields as `pub`, which freezes implementation details.
- Casting a byte buffer directly to a struct and assuming layout, padding, alignment,
  and endianness match.
- Using recursion on attacker-controlled structure without a depth limit.

---

## Networking, Protocols & Wire Formats

Networking is low-level I/O plus a shared language. A protocol has **syntax** (frame
layout), **semantics** (what messages mean), and **state** (which messages are valid
now). That makes protocol design closely related to compiler design.

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

This intentionally simple lab teaches:

- address + port endpoints;
- listening, accepting, reading, and writing;
- partial-I/O-safe helpers;
- request/response semantics;
- per-connection error isolation.

### 5. Protocol State Machines

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

### 6. Reverse Engineering an Unknown Protocol Safely

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

Clues:

- the first 2 or 4 bytes match the remaining length → likely framing;
- many readable bytes → text or lightly structured data;
- tiny input change modifies most later bytes → compression, encryption, or integrity
  data;
- repeated fixed prefix → magic/version/header;
- monotonically changing field → sequence number or timestamp;
- same message differs each time → nonce, timestamp, randomized compression, or
  encryption.

Do not assume traffic is plaintext merely because a few bytes are readable. Do not
attempt to defeat encryption or access systems without permission.

### 7. Designing a Protocol for Your Own Language

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

Design checklist:

- include a version from the beginning;
- define integer widths and byte order;
- specify UTF-8 validity and normalization policy;
- cap frame, string, collection, and nesting sizes;
- distinguish request IDs from session IDs with newtypes;
- define behavior for unknown optional and mandatory fields;
- authenticate before evaluating code;
- run untrusted code in a sandbox with CPU, memory, and time limits;
- make diagnostics structured rather than scraping human-readable text;
- publish test vectors: bytes plus expected decoded values.

### 8. Networking Failure Modes

- assuming `read` or `write` transfers the full buffer;
- allocating directly from an untrusted length field;
- missing timeouts, allowing idle connections forever;
- recursive decoding without a depth limit;
- treating TCP connection close as a complete application message;
- parsing before authentication or authorization boundaries are clear;
- trusting client-supplied type tags or offsets;
- logging secrets or entire untrusted payloads;
- using host endianness as a wire format;
- changing a message layout without a version strategy.

Tests should fragment a frame at every possible byte boundary, combine several frames
into one read, truncate every field, and exercise maximum permitted sizes.

---

## Executable Files, Linkers, ABIs & FFI

Compiling instructions is only part of producing a usable program. The result must also
describe how code and data are arranged, which external names it needs, where it may be
loaded, and how functions exchange values.

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

- machine code and static data;
- a symbol table describing known and unresolved names;
- relocation records describing addresses the linker must repair;
- debug information that maps instructions back to source;
- metadata identifying the target architecture and format.

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

- A **defined symbol** is provided by the current object or library.
- An **undefined symbol** is required from another object or library.
- A **local symbol** is private to one translation unit or object.
- A **global symbol** can participate in cross-object linking.
- A **weak symbol** may be replaced by a stronger definition.

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

Useful distinction:

- **File offset:** position inside the executable file.
- **Virtual address:** address where bytes appear in a process.
- **Relative virtual address:** offset from an image or module base.

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

- Where are integer, pointer, floating-point, and aggregate arguments placed?
- Where is the return value placed?
- Which registers must the caller preserve?
- Which registers must the callee preserve?
- Who adjusts the stack after the call?
- What stack alignment is required?
- How are variadic arguments represented?

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

FFI checklist:

- use `#[repr(C)]` for structs shared with C;
- use fixed-width integers when width matters;
- define who allocates and who frees each resource;
- do not let Rust panics unwind through a foreign ABI boundary;
- avoid passing Rust references, `String`, `Vec<T>`, or trait objects directly;
- document nullability, alignment, lifetime, thread-safety, and ownership;
- version exported interfaces deliberately.

---

## Concurrency, Atomics & Memory Models

Concurrency adds a second ordering problem. A compiler already reasons about the order
of expressions; concurrent code must also reason about what other threads are allowed to
observe.

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

- **Mutex:** grants one thread exclusive access to protected state.
- **Read/write lock:** permits many readers or one writer.
- **Condition variable:** lets a thread sleep until shared state changes.
- **Semaphore:** limits access to a finite set of permits.
- **Channel:** transfers values or events between tasks.
- **Atomic:** performs specific indivisible operations on supported primitive values.
- **Barrier:** waits until all participants reach the same phase.

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

- which operations are atomic;
- what constitutes a data race;
- whether a data race is an error, undefined behavior, or a defined result;
- what synchronization creates a happens-before relationship;
- whether values can move between threads;
- whether shared references permit mutation;
- how thread-local storage and destructors behave.

Optimizers rely on these rules. If the language declares data races invalid, the
optimizer may assume they never occur and transform code accordingly.

### 6. Concurrency Failure Modes

- **Deadlock:** participants wait forever in a dependency cycle.
- **Livelock:** participants keep reacting but make no useful progress.
- **Starvation:** one participant repeatedly loses access to a resource.
- **Priority inversion:** high-priority work waits on lower-priority work.
- **Lost wakeup:** a notification occurs before the waiter correctly observes it.
- **ABA problem:** a value changes from A to B and back to A, fooling a compare-and-swap
  check.
- **False sharing:** independent variables occupy one cache line and cause needless
  coherence traffic.

Debug concurrent systems with event traces, stable IDs, timeouts, and explicit state
transitions. Adding print statements can change timing and hide the bug.

---

## Practical Low-Level Workflow

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

- What representation am I looking at?
- Who owns this value or resource?
- How long is it valid?
- Which invariants have already been checked?
- What is the byte order, width, alignment, and signedness?
- Is this an address, offset, index, ID, or count?
- Where does trust change?
- Can this operation partially succeed?
- What state must come before and after it?
- What evidence would prove my current model wrong?

---

## X. Language Progression

The four stages of building a language from scratch:

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
