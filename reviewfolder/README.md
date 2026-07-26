# Programming & Compiler Mental Notes

### Grammar

A ***formal grammar*** is a set of rules that define what makes valid code in a programming language. Think of it like the grammar of English: “The cat sat” is valid, but “Cat the sat” is not.

For programming languages, a grammar specifies:

* **What tokens are valid** - keywords like `def`, `if`, `return` ; *operators* like `+`, `-`, `*`; *literals* like `42`, `true`
* **How tokens can be combined** - `1 + 2` is valid, `+ + 1` is not
* **The structure of programs** - functions contain statements, statements contain expressions

---

## I. Hardware Fundamentals

### The CPU
If you want to create a “computer” from scratch, you need to start by defining an **abstract model** for your computer. This *abstract model* is also referred to as ***Instruction Set Architecture (ISA)***

A **CPU** is an implementation of an **Instruction Set Architecture (ISA)** — an abstract model defining data types, registers, hardware support, and I/O. Together these make up **Machine Language**, the lowest-level language of computing.

The CPU continuously runs a **fetch-decode-execute** loop:

1. **Fetch** — Retrieve the instruction addressed by the **Instruction Pointer (IP)** / Program Counter (PC).
2. **Decode** — Interpret the **opcode** (a unique encoding for an operation — the "atoms of computing"), plus any operands (arguments) and optional prefix (behavioral modifier).
3. **Execute** — The **Control Unit (CU)** dispatches signals to functional units like the **ALU** (Arithmetic Logic Unit), which performs the operation on register values. Results are written back to memory if applicable.

> Modern CPUs accelerate this loop with **instruction pipelining** and **speculative execution**.

---

### Opcodes in Practice


Instructions are comprised of instruction code (aka operation code, in short **opcode** or *p-code*) which are directly executed by the *CPU*. An *opcode* can either **have operand(s)** or **no operand**. For example, in an 8-bit machine where instructions are 8 bits, an *opcode* load might be defined by the 4 bits `0011` followed by the second 4 bits as operand with `0101`, making up the instruction `00110101` in Machine Language. The opcode for incrementing by `1` of the previously loaded value could be defined by `1000` with *no operand*.


In an 8-bit machine where instructions are 8 bits:
- `LOAD 0101` → `00110101` — the first 4 bits (`0011`) are the opcode for *load*, the second 4 bits (`0101`) are the operand.
- `INCREMENT` → `1000` — the opcode for *increment by 1*, no operand needed.

Since opcodes are the atoms of computing, they are presented in an **opcode table** (e.g., the [x86 opcode reference](https://www.felixcloutier.com/x86/)).

---

### Assembly Language


Since it’s hard to remember the opcodes by their bit-patterns, we can assign abstract symbols to opcodes matching their operations by name. This way, we can create ***Assembly language*** from the Machine Language. 

Because bit-patterns are hard to remember, **Assembly Language** assigns abstract human-readable symbols to opcodes:

```
00110101  →  LOAD 0101
```

In the previous Machine Language example above, `00110101` (means load the binary 0101), we can define the symbol `LOAD` referring to `0011` as a higher level abstraction so that `00110101` can be written as `LOAD 0101`.

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


## Building a Parser

A parser takes raw source code (a string) and converts it into a structured **Abstract Syntax Tree (AST)** that a computer can evaluate or compile. Think of source code as a sentence and the AST as its grammatical diagram. It captures *structure*, not just text. 

Here is the step-by-step process of building one from scratch using Rust and the `pest` crate:

### 1. Setting Up the Lexer/Grammar

**Grammar rules are built from expressions** (hence "parsing expression grammar"). These expressions are a terse, formal description of how to parse an input string.

**Expressions are composable**: they can be built out of other expressions and nested inside of each other to produce arbitrarily complex rules (although you should break very complicated expressions into multiple rules to make them easier to manage).

**PEG expressions** are suitable for both *high-level meaning*, like "a function signature, followed by a function body", and *low-level meaning*, like "a semicolon, followed by a line feed". The combining form "followed by", the sequence operator, is the same in either case.

---

###### Looking at macros first helps with understanding grammars and pest grammar:


| Within Macros |  Explanation |
|---------|---------|
| `$x:ty`  | Macro capture (here a `$x` is the capture and `ty` means `x` must be type). |
|  `$x:block`   | A block `{}` of statements or expressions, e.g., `{ let x = 5; }` |
|  `$x:expr`    | An expression, e.g., `x`, `1 + 1`, `String::new()` or `vec![]` |
|  `$x:ident`   | An identifier, for example in `let x = 0;` the identifier is `x`. |
|  `$x:item`    | An item, like a function, struct, module, etc. |
|  `$x:lifetime` | A lifetime (e.g., `'a`, `'static`, etc.). |
|  `$x:literal` | A literal (e.g., `3`, `"foo"`, `b"bar"`, etc.). |
|  `$x:meta`    | A meta item; the things that go inside `#[…]` and `#![…]` attributes. |
|  `$x:pat`     | A pattern, e.g., `Some(x)`, `(17, 'a')` or <code>x&vert;x</code>. |
|  `$x:pat_param`| Subset of patterns without top-level &vert;, e.g., `Some(x)` or `x`. |
|  `$x:path`    | A path (e.g., `foo`, `::std::mem::replace`, `transmute::<_, int>`). |
|  `$x:stmt`    | A statement, e.g., `let x = 1 + 1;`, `String::new();` or `vec![];` |
|  `$x:tt`      | A single token tree, [see here](https://stackoverflow.com/a/40303308) for more details. |
|  `$x:ty`      | A type, e.g., `String`, `usize` or `Vec<u8>`. |
|  `$x:vis`    | A visibility modifier;  `pub`, `pub(crate)`, etc. |

---

#### Pest 
First, you need rules that define what makes valid code. Using a parser generator (like the `pest` crate in Rust), you define a grammar file (e.g., `grammar.pest`). The tool uses this grammar to validate the text and tokenize the raw string into an iterator of parsed "pairs" or tokens (e.g., pulling out operators, numbers, and nested expressions).





 The `pest` parser generator uses this grammar to validate the text and tokenize the raw string into an iterator of parsed "pairs" (tokens). 

```rust
#[derive(pest_derive::Parser)]
#[grammar = "grammar.pest"]
struct CalcParser;

```









| Syntax | Meaning | Example |
|--------|---------|---------|
| `"text"` | Match exact text | `"def"` matches the keyword def |
| `~` | Sequence (then) | `"if" ~ "(" ~ Expr ~ ")"` matches `if` followed by `(` |
| `\|` | Choice (or) | `"true" \| "false"` matches either |
| `*` | Zero or more ([Kleene star](https://en.wikipedia.org/wiki/Kleene_star)) | `Stmt*` matches any number of statements |
| `+` | One or more | `ASCII_DIGIT+` matches one or more digits |
| `?` | Optional | `ReturnType?` matches zero or one return type |
| `{ }` | Rule definition | `Add = { "+" }` defines a rule |
| `_{ }` | Silent rule | `_{ Expr }` matches but does not appear in AST |
| `@{ }` | Atomic rule | `@{ ASCII_DIGIT+ }` matches as a single token |
| `SOI` | Start of input | Beginning of the source code |
| `EOI` | End of input | End of the source code |


##### example calculcator

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

| Syntax           | Meaning                           | Syntax                  | Meaning              |
|:----------------:|:---------------------------------:|:-----------------------:|:--------------------:|
| `foo =  { ... }` | [regular rule]                    | `baz = @{ ... }`        | [atomic]             |
| `bar = _{ ... }` | [silent]                          | `qux = ${ ... }`        | [compound-atomic]    |
| `#tag = ...`     | [tags]                            | `plugh = !{ ... }`      | [non-atomic]         |
| `"abc"`          | [exact string]                    | `^"abc"`                | [case insensitive]   |
| `'a'..'z'`       | [character range]                 | `ANY`                   | [any character]      |
| `foo ~ bar`      | [sequence]                        | <code>baz \| qux</code> | [ordered choice]     |
| `foo*`           | [zero or more]                    | `bar+`                  | [one or more]        |
| `baz?`           | [optional]                        | `qux{n}`                | [exactly *n*]        |
| `qux{m, n}`      | [between *m* and *n* (inclusive)] |                         |                      |
| `&foo`           | [positive predicate]              | `!bar`                  | [negative predicate] |
| `PUSH(baz)`      | [match and push]                  | `PUSH_LITERAL("a")`     | [push without match] |
| `POP`            | [match and pop]                   | `PEEK`                  | [match without pop]  |
| `DROP`           | [pop without matching]            | `PEEK_ALL`              | [match entire stack] |






### 2. Defining the Abstract Syntax Tree (AST)
The AST captures the *meaning* and *structure* of the code, not just the raw text. In Rust, this is typically done using `enum` to represent different node types:
- **Terminal Nodes (Leaves):** e.g., `Node::Int(i32)` for raw numbers.
- **Unary Expressions:** e.g., `Node::UnaryExpr { op, child }` for operations like `-5`.
- **Binary Expressions:** e.g., `Node::BinaryExpr { op, lhs, rhs }` for operations like `1 + 2`. *(Note: `Box<Node>` is used for the children to allow recursive nesting).*

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

// these are potential leaves in an ast
// think of an ast as a composite design pattern where
// the leaves are the core data types such int
// the composits are the structs that have child/inner nodes or other structs
// and parse allows you work on composits and leaves the exact same way
// parse and will figure out the method depending on the component
// think of program like the main container



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

> **Note:** `Box<Node>` is required for the children to allow recursive nesting. Without `Box`, Rust wouldn't be able to calculate the size of the `Node` enum at compile time.

### 3. The Main Parsing Loop

The main entry point (often just `parse(source: &str)`) takes the raw string, feeds it to the parser generator, and loops through the resulting top-level tokens. When it finds a top-level expression, it passes that token to a recursive builder function to construct the AST.


```rust
pub fn parse(source: &str) -> std::result::Result<Vec<Node>, pest::error::Error<Rule>> {
    let mut ast = vec![];
    // Tokenize the string using the 'Program' rule
    let pairs = CalcParser::parse(Rule::Program, source)?;
    
    // Iterate through top-level expressions
    for pair in pairs {
        if let Rule::Expr = pair.as_rule() { // every line is an expression
            ast.push(build_ast_from_expr(pair));
// expr is could be a term
// (which could be a data type or unary / binary expression with parathesis),
// unary - operator followed by term,
// or binary - which has a term followed by operator and a term 
// binary can additionally have any numbers of operator and term combos,
        }
    }
    Ok(ast)
}

```

### 4. Parsing Expressions & Associativity (the composits)

This is the core logic that converts generic `pest` tokens into your custom AST nodes by 
looking at the rule:
- **Unary Expressions:** Grab the operator and the single child term, then construct the Unary node.
- **Binary Expressions:** Handling chained operations (like `1 + 2 + 3`) requires **left-associativity** so it evaluates as `(1 + 2) + 3`. Instead of using pure recursion (which might result in right-associativity), this is often handled using a `loop`:
    1. Parse the initial Left-Hand Side (LHS), operator, and Right-Hand Side (RHS).
    2. Build the first `BinaryExpr` node.
    3. **The Loop:** If there is another operator immediately following in the token stream, take the expression you *just built* and make it the **new LHS**, grab the next number as the RHS, and build a new, nested `BinaryExpr`. 


```rust
fn parse_unary_expr(pair: pest::iterators::Pair<Rule>, child: Node) -> Node {
    Node::UnaryExpr { // first symbol should be an operator, then we need to put the child in a box to allowa recursive nesting 
        op: match pair.as_str() {
            "+" => Operator::Plus,
            "-" => Operator::Minus,
            _ => unreachable!(),
        },
        child: Box::new(child),
    }
}

// look at struct definition to understand the order of the parameters
fn parse_binary_expr(pair: pest::iterators::Pair<Rule>, lhs: Node, rhs: Node) -> Node {
// will create a new node to insert into a syntax tree 
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
         
         // 1. Parse the initial Left-Hand Side (LHS)
         let lhspair = pair.next().unwrap();
         // code to parse LHS (could be Unary or Term)
         let mut lhs_built_term = match lhspair.as_rule() { 

                     Rule::UnaryExpr => { // dont forget unary expr is also a term
                         let mut inner = lhspair.into_inner();
                         let op = inner.next().unwrap();
                         let child = inner.next().unwrap();
                         let child = build_ast_from_term(child); // we have to process child term without op
                                                                 // because it only checks data types and expressions
                         parse_unary_expr(op, child) 
                     }
                     _ => build_ast_from_term(lhspair), // means its an expression
                 };
         
         // 2. Grab Operator and initial Right-Hand Side (RHS)
          // we are assuming its an operator 
         let op = pair.next().unwrap();
         let rhspair = pair.next().unwrap(); // we are assuming its not an operator
         let mut rhs_built_term = build_ast_from_term(rhspair);
         
         // 3. Build the first BinaryExpr node by using the op we assumeed lhs and rhs we built from terms
         let mut binary_expr_retval = parse_binary_expr(op, lhs_built_term, rhs_built_term;
         
         // 4. The Loop: Handle chained operators (e.g., + 3 + 4)
         loop {
             let pair_buf = pair.next();
             if let Some(op) = pair_buf {
                 // The expression we just built becomes the *new* LHS
                 lhs_binary_expr = binary_expr_retval; 
                 adjacent_rhs_binary_expr = build_ast_from_term(pair.next().unwrap());
                 // Build a new, nested BinaryExpr
                 retval = parse_binary_expr(op, lhs_binary_expr, adjacent_rhs_binary_expr);
             } else {
                 return retval; // No more operators, return the final tree
             }
         }
     }
     Rule::Term => build_ast_from_term(pair),
     unknown => panic!("Unknown expr: {:?}", unknown),
         }


}


```



### 5. Parsing Terms (The Leaves)

When the parser drills down to a basic term (`Rule::Term`), it hits the bottom of the tree. It expects one of two things:

1. **A raw number:** It parses the string into an `i32` and returns a `Node::Int`.
2. **A nested expression:** If it hits parentheses, it recurses back up to the expression parsing logic to evaluate the inside of the parentheses first.

```rust
fn build_ast_from_term(pair: pest::iterators::Pair<Rule>) -> Node {
    let pair = pair.into_inner().next().unwrap();
    match pair.as_rule() {
        // Resolve raw strings into actual integers
        Rule::Int => {
            let int: i32 = pair.as_str().parse().unwrap();
            Node::Int(int)
        }
        // Recurse back up for nested expressions like (1 + 2)
        Rule::Expr => build_ast_from_expr(pair),
        unknown => panic!("Unknown term: {:?}", unknown),
    }
}

```




---


### The Compiler

A compiler is any program that translates (maps, encodes) a language A to language B. Each compiler has two major components:


A **compiler** is any program that translates Language A → Language B. Its two core components:

- **Frontend** — Maps source code strings to a structured format called an **Abstract Syntax Tree (AST)**.
- **Backend (Code Generator)** — Translates the AST to Bytecode, IR, or Assembly.

Most often, when we talk about compiler, we mean Ahead-Of-Time (AOT) compiler where the translation happens before execution. Another form of translation is Just-In-Time (JIT) compilation where translation happens right at the time of the execution.


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

We need a systematic way to turn any AST into LLVM IR.

The answer is recursive tree traversal. We already do this in the interpreter - walk the tree, evaluate each node. For code generation, we walk the tree and emit instructions for each node instead of computing values.

Two common patterns help structure this:

* Builder pattern - Used for LLVM IR generation
* Visitor pattern - for AST transformations

### Builder Pattern
Think of the LLVM builder like a cursor in a text editor. You position it somewhere in your code, then “type” instructions at that position. The builder keeps track of where you are and ensures instructions are added in the right place.

Let’s compare our interpreter’s recursive evaluation to the new JIT approach.

Interpreter: Walk tree, compute values
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

- **Variables** — Named storage (`n`). 
- **Functions** (`fib`) — Abstraction and reuse.
- **Parameters** (passing `n`) — Data flow.
- **Conditionals** (`if`/`else`) — Branching.
- **Operators** (`<`, `-`) — Computation.
- **Recursion** (`fib` calls `fib`) — Self-reference.
- **Call stack** (tracks each frame) — Memory management.

The AST becomes a *program to execute*, not just an expression to evaluate.


---

### Variables

Variables let us store values and refer to them by name. Without variables, every computation would need to repeat its literal inputs — variables give us **memory**.

Reassigning a variable means any code referencing it automatically uses the new value, which makes code reusable and readable.

**Where variables live:** the *environment* (storage) — a `HashMap` inside a *frame*.

- **Local** variables exist inside a function.
- **Global** variables exist outside all functions.
- Variables can be **reassigned**.

This simple mechanism — storing and looking up names — underlies all programming:

| Concept | Role |
|---|---|
| Parameters | Variables bound from function call arguments |
| Loop counters | Variables that change each iteration |
| Object fields | Variables attached to an object (in Thirdlang) |

> The environment is one of the most important data structures in any interpreter.

---

### Functions

Functions are the heart of any programming language. Without them, code must be copy-pasted every time it's reused. Functions let us:

1. **Name** a computation, to refer to it later
2. **Parameterize** it with inputs, so it works with different values
3. **Reuse** it — write once, call anywhere

**Parameters**: names for the inputs a function expects. When we bind *parameters*, we are associating names with argument values



Functions enable **abstraction**: calling `fibonacci(10)` hides *how* Fibonacci is computed behind a simple name.

#### How Function Calls Work

Evaluating `add(3, 4)` triggers a fixed sequence:

1. **Look up** the function by name
2. **Evaluate** the arguments (`3`, `4`)
3. **Create** a new frame for local variables
4. **Bind** parameters to arguments (`a = 3`, `b = 4`)
5. **Execute** the function body
6. **Return** the result and pop the frame

### The Call Stack 

**Call Stack**: Runtime data structure tracking function calls. Each call pushes a frame; return pops it.


Each **frame** on the stack represents one in-progress function call. The stack **grows** on call and **shrinks** on return — last in, first out (LIFO).

**One frame per call:** two calls to `foo` each get their own `x`. Call 1's `x = 5` never interferes with call 2's `x = 10`, because they live in separate frames. This isolation is essential for recursion, where the same function appears on the stack multiple times, each with its own variables.

---

### Control Flow

Programs need to **decide** ("if logged in, show dashboard") and **repeat** ("while items remain, sum their prices"). Without control flow, code only runs straight-line, top to bottom.

#### Conditionals: `if` / `else`

An `if` expression evaluates a condition and picks a branch:

1. **Evaluate** the condition (e.g. `x > 0`)
2. **Choose** a branch — `then` if true, `else` if false
3. **Return** whatever that branch produces

#### Loops: `while`

A `while` loop repeats its body while a condition holds true, using Rust's `loop` construct with a condition check each iteration:

> After the body runs, execution returns to the top and re-checks the condition — this is what creates repetition.

#### Control Flow in Functions

- **Nested conditionals** — for when one condition isn't enough
- **`return` in loops** — exits the function immediately, even from deep inside a loop; useful for "search" patterns

#### Control Flow as Branching

Both `if` and `while` change the flow of execution, letting code:

- **Skip** code (untaken `if` branch)
- **Repeat** code (`while` body)
- **Exit early** (`return` inside a loop)

In compiled languages, these become branch instructions — the CPU jumps to different memory locations. (In Secondlang, `if` compiles to LLVM IR's `br` instruction.)

---

## Recursion

Recursion is when a function calls itself. It seems paradoxical — how can a function call itself mid-execution? — but the call stack makes it work cleanly.

> **Analogy:** Russian nesting dolls (matryoshka). Each doll contains a smaller version of itself, down to the smallest doll, which contains nothing.

### The Key Insight

Every recursive function needs two parts:

| Part | Purpose |
|---|---|
| **Base case** | Returns directly, no further recursion — the "smallest doll" |
| **Recursive case** | Calls itself on a smaller problem, then combines the result |

- No base case → recursion never stops → **stack overflow**
- No recursive case → not actually recursion, just a regular function

### Why Recursion Works in Firstlang

Three design decisions make it work:

1. **Functions are stored globally** — `factorial` can look itself up by name mid-call.
2. **Each call gets its own frame** — `factorial(3)` called from within `factorial(4)` has an `n` fully independent of the outer `n = 4`.
3. **Return values propagate correctly** — `factorial(1)` returns `1` → used by `factorial(2)` as `2 * 1 = 2` → and so on up the stack.

> Using a single global `n` instead of per-call frames would break recursion: each call would overwrite the shared `n`.

**Mutual recursion:** functions can call each other recursively.

**⚠️ Stack overflow risk:** every recursive call consumes stack memory; very deep recursion exhausts it.

> Some languages implement **tail call optimization** to run certain recursive functions in constant stack space — not implemented here, but a notable optimization.

### When to Use Recursion

Recursion fits problems with recursive structure:

- **Trees** (each subtree is a smaller tree)
- **Mathematical sequences** (Fibonacci, factorial)
- **Divide-and-conquer algorithms** (merge sort, quicksort)
- **Parsing nested structures** (JSON, HTML, an AST)

For simple loops, iteration is usually clearer — but for inherently recursive problems, recursion is more natural.



### Operators

 An operator - it tells us what to do. Values and operands are the things we operate on.



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

If you pause a program in a debugger, you are looking at its current state. The call stack tells you how the program got to this state and what local data it currently has access to within that specific function frame.
In computer science terms, adding a stack to a finite state machine turns it into a **Pushdown Automaton**—allowing it to handle nesting, recursion, and scoped memory.


- **The Concise Definition:**
      **State Machine:** A mathematical model of computation representing a system that can be in exactly one of a finite number of conditions (states) at any given time. The machine changes from one state to another (a transition) in response to specific inputs or events.

- **The Programming Definition:**
      **State Machine:** A system that maintains a "memory" of its current condition and changes that condition based on incoming events. Unlike a stateless system (like a basic calculator), a state machine's output depends on its accumulated history—which in a programming language is tracked via variables, memory heaps, and the call stack.

- **The Bullet-Point Breakdown:**
     **State Machine Core Concepts:**
     * **State:** The current condition or snapshot of the system's memory (e.g., current variable values, active call stack).
     * **Inputs/Events:** The actions or instructions fed into the system (e.g., executing the next line of code).
     * **Transitions:** The rules that dictate how the system moves from one state to the next based on the input.


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

Each **frame** is just a `HashMap` pushed and popped like any stack. Grammar grows, AST nodes multiply, but execution is still recursive tree traversal — now with scoped state.

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

Bytecode is another technique to translate source code to Machine Code is emulating the Instruction Set with a new (human-friendly) encoding (perhaps easier than assembly). 

**Bytecode** is such an intermediate language/representation, which sits between source and assembly — lower-level than source, higher-level than assembly language. It emulates an instruction set with a new, simplified encoding, and is executed by a **VM**.




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

***Is Python (or a language X) Compiled or Interpreted?***

Being AOT compiled, JIT compiled or interpreted is implementation-dependent. For example, the standard Python implementation is CPython which compiles a Python source code (in CPython VM) to CPython Bytecode (contents of .pyc) and interprets the Bytecode. However, another implementation of Python is PyPy which (more or less) compiles a Python source code (in PyPy VM) to PyPy Bytecode and JIT compiles the PyPy Bytecode to the Machine Code (and is usually faster than CPython interpreter).


#### Bytecode Structure
* `instructions` - A flat array of bytes. Opcodes and their arguments, all mixed together.
* `constants` - A table of literal values. Instead of encoding 42 in the instruction stream, we store it in the constants table and reference it by index.

```rust
pub struct Bytecode {
    pub instructions: Vec<u64>,// 64 bit instructions 
    pub constants: Vec<Node>,
}
```

---

### The Stack Machine

Machine to execute our bytecode:  Our VM is a **Stack Machine** - it keeps intermediate values on a stack.

We’ve seen two ways to execute code: interpret the AST directly, or JIT compile to native machine code. There’s a third approach that sits in between: **compile to bytecode, then interpret that.**

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
- **Bytecode** which is the program to execute
- A memory **stack array** for Keeping the Bytecode instructions that supports pushing and poping instructions. Also for intermediate values. we use a fixed array for speed. 
- An **Instruction Pointer (IP)** and **Stack Pointer (SP)**. Guiding which instruction was executed and what is next.

**Why stacks?** They handle *any* nesting automatically. For `(1 + 2) * (3 + 4)`:
- Push 1, push 2, add → stack: `[3]`
- Push 3, push 4, add → stack: `[3, 7]`
- Multiply → stack: `[21]`

Every operation pops its inputs and pushes its output. The stack naturally tracks what's "in progress."

Think of bytecode as a simplified assembly language designed for our VM. 

* OpConstant(index)	Push a constant onto the stack
* OpAdd 	Pop two values, push their sum
* OpSub	Pop two values, push their difference
* OpPop	Pop and discard the top value

Real assembly has hundreds of instructions. 

**VM execution of `1 + 2` (fetch-decode-execute):**

These four carry the arithmetic ( `OpSub` not used in this example):

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
- **Type Environment** — A `HashMap<String, Type>` mapping names to types. Also called symbol table or context

The environment is:

**Extended** when we declare a variable or enter a function (adding new bindings)
**Queried** when we reference a variable (looking up its type)
**Scoped** - inner scopes can shadow outer bindings


---

### Two-Pass Function Type Checking

Functions are trickier because we need to handle:

Parameters (types come from annotations)
Local variables (types are inferred)
Return value (must match declared return type)


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

> Even when LLVM will optimize downstream, custom passes improve compile speed, debug output readability, and can exploit language-specific knowledge LLVM can't.

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

LLVM's **`mem2reg` pass** promotes these stack slots to fast registers automatically.

**Why store parameters too?** For `add(%a, %b)`, the pattern is: (1) `alloca` stack space for `%a.addr`/`%b.addr`, (2) `store` the incoming parameters into those slots, (3) `load` the values back, (4) `add i64` the loaded values, (5) `ret` the result. This looks wasteful — why not use `%a` and `%b` directly? Because in LLVM IR, SSA values like `%a` are immutable, but in most source languages variables *can* change. Storing every variable in a stack slot up front handles mutability uniformly, and `mem2reg` cleans it up afterward when a variable is never actually reassigned.

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

Recursive functions use the standard `call` instruction:

```llvm
%result = call i64 @fib(i64 %n_minus_1)
```

#### Function Calls & Return Values

To compile a call: look up the function, compile each argument, then emit a `call` instruction. LLVM calls return either a **basic value** (an int or pointer you can use) or **nothing** (void). The pattern `try_as_basic_value().unwrap_basic()` extracts the basic-value case — safe here since our functions always return `int` — and `.into_int_value()` converts the result to the specific integer type needed downstream.

---

### LLVM State

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

The Prestage stage:

- **`Context::create()`** - The top-level container. All LLVM objects belong to a context. It manages memory and ensures thread safety.
- **`context.create_module("addition")`** - Creates a module (like a compilation unit). Our `add` function will live here.
- **`context.i32_type()`** - Gets the 32-bit integer type. LLVM is explicitly typed - we need to declare that our function works with `i32`.



1. **Pass 1 — Declare Functions** — Announce all function signatures before compiling any body. Enables mutual recursion via forward references.          
     
- **`i32_type.fn_type(&[i32_type.into(), i32_type.into()], false)`** - Creates a function type: returns `i32`, takes two `i32` parameters. The `false` means it's not variadic (doesn't take variable arguments like `printf`).
- **`module.add_function("add", fn_type, None)`** - Adds a function called "add" with this signature to our module.
- **`context.append_basic_block(add_fn, "entry")`** - Creates a basic block named "entry". A basic block is a sequence of instructions with no branches in the middle - execution flows straight through.
- **`context.create_builder()`** - The builder is our "cursor" for adding instructions. We position it at a basic block, then build instructions there.
- **`builder.position_at_end(entry)`** - Point the builder at our entry block. New instructions will go here.


2. **Pass 2 — Compile Bodies** — Generate IR instructions for each function body using the alloca/load/store pattern.

- **`add_fn.get_nth_param(0)`** - Get the first parameter. LLVM functions have an array of parameters, indexed from 0.
- **`.unwrap().into_int_value()`** - Parameters come as generic "basic values." We know ours are integers, so we convert them.
- **`builder.build_int_add(x, y, "result")`** - This emits an `add` instruction. The `"result"` is just a name for the output (helps when reading IR).
- **`builder.build_return(Some(&sum))`** - Emit a `ret` instruction to return our sum.



3. **Pass 3 — Create `@__main` Wrapper** — Wrap top-level expressions (e.g., `fib(10)`) in `__main` as the JIT entry point, then verify the module.

- **`module.create_jit_execution_engine(OptimizationLevel::None)`** - Creates a JIT compiler. LLVM takes our IR and compiles it to native x86/ARM code *right now*, in memory.
- **`execution_engine.get_function::<unsafe extern "C" fn(i32, i32) -> i32>("add")`** - Look up our compiled function. The type signature tells Rust how to call it.
- **`add.call(1, 2)`** - Call the native function! This jumps directly to machine code - no interpretation, no overhead.


---

### Running a Program: Source to Result

What happens when you run `cargo run -- examples/fibonacci.sl`:

1. **Parse** the source file → Typed AST (with `Unknown` types).
2. **Type check** → Typed AST (all types resolved).
3. **Optimize** (optional) → Simplified AST.
4. **Compile** → LLVM IR.
5. **JIT** → Native machine code.
6. **Execute** → Result.

All in a fraction of a second. Each step transforms the program into a different representation, getting closer and closer to something the CPU can execute directly.

**Calling the JIT-compiled code:**
1. Create a code generator — sets up the LLVM context, module, and builder (the workspace).
2. Compile the program to IR.
3. Create a JIT execution engine from the verified module.
4. Get a pointer to `@__main` — our entry point.
5. Call it and return the result.

The JIT engine compiles IR to native machine code on the fly, then executes it — this is much faster than interpretation, because the CPU is running actual machine code, not walking a tree.

> The `unsafe` block required when calling JIT output signals that we are invoking raw machine code — we must trust that our code generator produced valid IR.

---

### LLVM Optimization Passes

After code generation, LLVM passes transform naive IR into efficient code. Passes are chained in a **pipeline string** (e.g., `"mem2reg,dce,instcombine,simplifycfg"`).

**Why optimize ourselves if LLVM will do it anyway?**

| Reason | Benefit |
|---|---|
| **Learning** | Implementing optimizations teaches how production compilers actually work |
| **Simplicity** | A simpler AST means simpler, less error-prone code generation |
| **Debug output** | Optimized IR is easier to read when printed for debugging |
| **Specialized optimizations** | You may know language-specific facts LLVM cannot infer |
| **Compile time** | A simpler AST means less work for LLVM, so compilation is faster |

| Pass | What It Does |
|---|---|
| **`mem2reg`** | Promotes `alloca` stack slots to SSA registers — the most impactful pass |
| **`dce`** | Dead Code Elimination — removes instructions whose results are never used |
| **`instcombine`** | Merges redundant instructions; constant folds; strength-reduces (`mul x, 2` → `shl x, 1`) |
| **`simplifycfg`** | Removes empty blocks, merges single-predecessor blocks, simplifies trivial branches |

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

**Each pass in isolation** (simple, minimal examples):

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

`instcombine` — simplifies arithmetic patterns: `sub i64 %x, 1` → `add i64 %x, -1`; `mul i64 %x, 2` → `shl i64 %x, 1` (shift left); constant-folds `add i64 3, 4` → `7`.

`simplifycfg` — removes empty basic blocks, merges blocks with a single predecessor, and simplifies trivially-true/false branches.

**LLVM preset pipelines:**

| Level | Description |
|---|---|
| `default<O0>` | No optimization (verification only) |
| `default<O1>` | Light optimization |
| `default<O2>` | Standard optimization (recommended) |
| `default<O3>` | Aggressive optimization |

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
4. **`run_passes`** — takes the comma-separated pass list (or a preset like `"default<O2>"`) and applies it to the module.

---

## VIII. Object-Oriented Concepts (Classes)

### Why Classes?

Without classes, related data is **scattered** this works, but there are problems:

No semantic grouping - Nothing says x1 and y1 belong together. They’re just two independent integers.
Easy to mix up - What if you accidentally use x1 with y2? The compiler won’t catch it.
Can’t pass as a unit - You can’t write def distance(p1, p2). You need def distance(x1, y1, x2, y2).
No encapsulation - The distance formula is scattered across your code. If you want to change it, you need to find every place you computed it.
We need a way to group related data and attach behavior to that data. and thats what classes gives us


```
# Without classes — error-prone
distance(x1, y1, x2, y2)

# With classes — clear, grouped, safe
p1.distance(p2)
```

> **Classes analogy:** Think of a filing cabinet. Without classes you have loose papers (`x1`, `y1`). Classes are folders that group related papers together — and know what operations to perform on them.




---

### OOP Vocabulary

| Concept | Description | Example |
|---|---|---|
| **Class** | Blueprint for creating objects | `class Point { ... }` |
| **Object** | An instance of a class | `p = new Point(1, 2)` |
| **Field** | Data stored in an object | `self.x`, `self.y` |
| **Method** | Function attached to a class | `def distance(self)` |
| **Constructor** | Initializes a new object | `def __init__(self)` |
| **Destructor** | Cleans up before deletion | `def __del__(self)` |

---

### Classes as Custom Types

Classes define **new types** in a **nominal type system** — types are identified by their names:

```
class Point   { x: int  y: int  ... }
class Counter { count: int  ... }

def move(p: Point, dx: int) -> Point { ... }
#         ^^^^^  Point is now a first-class type like int
```

---

### Design Decisions

We implement a **subset** of OOP — deliberately simple:

**No Inheritance** — Many OOP languages support `class B extends class A`. We skip this because it adds significant complexity (vtables, dynamic dispatch), and composition over inheritance is often preferred anyway. The core concepts are clearer without it.

**Explicit Memory Management** — Instead of GC, we use explicit `delete`:
```
p = new Point(1, 2)
delete p   # programmer's responsibility
```
This mirrors C++ and teaches how memory actually works. Understanding manual management helps you appreciate what GC, reference counting, and ownership models solve.

**What we include vs exclude:**

| Included | Excluded |
|---|---|
| Class definition with fields | Inheritance (vtables, dynamic dispatch) |
| Methods with `self` | Interfaces / Traits |
| Constructor `__init__` | Visibility (`public`/`private`) |
| Destructor `__del__` | Static methods |
| `new` / `delete` | Operator overloading |
| Field access `p.x` / `self.x` | |
| Method calls `p.method()` | |
| Classes as types `other: Point` | |

---

### Memory Model: Stack vs Heap

In most primitive/variable contexts, values live on the **stack**. Objects live on the **heap** because they must outlive the function that created them and can be shared across references.

| | Stack | Heap |
|---|---|---|
| **Management** | Automatic (LIFO with call frames) | Manual (`new`/`delete`) |
| **Speed** | Fast (just move a pointer) | Slower (system call to OS) |
| **Size** | Limited (~few MB) | Large (all available RAM) |
| **Lifetime** | Tied to function scope | Until explicitly freed |

> **Analogy:** The stack is a stack of cafeteria trays — same size, add/remove from top only. The heap is a parking lot — park anywhere, leave as long as you want, but you must retrieve it or it stays forever (memory leak).

---

### Constructors & `new`

The **constructor** (`__init__`) initializes a new object. It always takes `self` as the first parameter.

When you write `p = new Point(10, 20)`:
1. **Calculate size** — `Point` has two `i64` fields → 16 bytes.
2. **Call `malloc`** — Ask the OS for 16 bytes; get back a pointer.
3. **Initialize fields** — Zero-initialize all fields as a safety baseline.
4. **Call `__init__`** — Runs with the new pointer as `self`; sets `self.x = 10`, `self.y = 20`.
5. **Return pointer** — `p` now holds the heap address of the object.

**Memory layout:**

```
class Point {
    x: int    # offset 0,  8 bytes (i64)
    y: int    # offset 8,  8 bytes (i64)
}             # total: 16 bytes

LLVM: %Point = type { i64, i64 }
```

Field order (tracked via `field_order` in `ClassInfo`) determines the memory layout — order matters!

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

**Failure:** Our constructors cannot fail. Real languages handle this via exceptions (Java, Python), `Result`/`Option` (Rust), or factory methods. We keep it simple — constructors always succeed.

---

### The `self` Parameter

`self` is a **pointer** to the object the method was called on. It is always the first parameter of every method — explicit, like Python:

```python
# Thirdlang / Python style — explicit self
def get_x(self) -> int {
    return self.x
}

# Call: p.get_x()  →  compiled as: Point__get_x(p)
```

| Language | Self/This |
|---|---|
| Python | `def method(self):` — explicit |
| Rust | `fn method(&self)` — explicit |
| Java / C++ | `this` — implicit |
| Thirdlang | `def method(self)` — explicit |

Explicit `self` makes it clear: **methods are just functions that receive the object as their first argument**.

**In codegen,** `self` is just stored as a local variable pointer, exactly like any other parameter — there's no special-casing:
```rust
Expr::SelfRef => {
    let ptr = self.variables.get("self").ok_or("'self' not in scope")?;
    self.builder.build_load(ptr_type, *ptr, "self").unwrap()
}
```

---

### Methods & Field Access

Methods compile to regular functions with the naming convention `ClassName__methodName` — avoiding collisions between classes and making ownership clear.

**Field access via `getelementptr` (GEP):**

GEP calculates the memory address of a struct field without reading memory — it is pointer arithmetic that knows struct layouts:

```llvm
; return self.x
%x_ptr = getelementptr %Point, ptr %self, i32 0, i32 0  ; pointer to field 0
%x     = load i64, ptr %x_ptr
ret i64 %x

; self.x = 42
%x_ptr = getelementptr %Point, ptr %self, i32 0, i32 0
store i64 42, ptr %x_ptr
```

**Method call `p.get_x()` → `call @Point__get_x(ptr %p)`** — the object is passed as the first argument.

> **Important:** When you pass an object as a parameter, you pass a **pointer** — not a copy. Modifying `other.x` inside a method modifies the original object. This is **reference semantics**.

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

| Method | LLVM Function |
|---|---|
| `Point.__init__` | `@Point____init__` |
| `Point.get_x` | `@Point__get_x` |
| `Counter.increment` | `@Counter__increment` |

This avoids name collisions between classes and lets the JIT find the right function at call sites.

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

After `delete`, `p` still holds the old address — but accessing it is **undefined behavior**.

---

### LLVM Codegen for Classes — Summary

| Thirdlang | LLVM IR |
|---|---|
| `class Point { x: int }` | `%Point = type { i64 }` |
| `new Point(10)` | `call @malloc` + `call @Point____init__` |
| `p.x` | `getelementptr` + `load` |
| `p.x = 5` | `getelementptr` + `store` |
| `p.method()` | `call @Point__method(ptr %p)` |
| `delete p` | `call @Point____del__` + `call @free` |

**Class compilation happens in six phases:**
1. **Declare libc functions** — `malloc` and `free`.
2. **Create class struct types** — define an LLVM struct for each class.
3. **Declare methods** — create function signatures (enables cross-method and forward calls).
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

> The code that runs isn't interpreted — it's real compiled machine code, the same as if you'd written it in C or Rust. Objects really live on the heap. Methods really jump to function addresses. When this calls `malloc`, it's calling the actual C `malloc`.

**Performance considerations:**

| What We Do | What Real Compilers Add |
|---|---|
| Direct field access via GEP (fast) | Inline method calls when possible |
| Static method calls — no vtable lookup | Escape analysis (stack-allocate short-lived objects) |
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

1. **Delete what you allocate** — every `new` has a matching `delete`.
```
p = new Point(1, 2)
# ... use p ...
delete p
```

2. **One owner** — have one clear owner responsible for deletion:
```
def make_point() -> Point {
    return new Point(1, 2)   # Caller owns this
}
p = make_point()
delete p   # Caller is responsible
```

3. **Null after delete** (in languages that support it):
```
delete p
p = null   # Mark as invalid — prevents use-after-free
```

---

### Memory Safety Bugs

Languages without GC (C, C++) require explicit `new`/`delete`, introducing high-risk bugs:

| Bug | Description | Code |
|---|---|---|
| **Memory Leak** | Forgot `delete`; memory consumed until exit | `p = new Point(1,2)` — never freed |
| **Use After Free** | Access after `delete` — undefined behavior | `delete p; p.x` |
| **Double Free** | `delete` twice — corrupts allocator | `delete p; delete p` |
| **Dangling Pointer** | Two refs to same object; one deletes it | `q = p; delete p; q.x` |

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

| Approach | Pros | Cons |
|---|---|---|
| **Manual (C, C++)** | Fast, predictable, teaches fundamentals | Error-prone |
| **Garbage Collection (Java, Python)** | Safe, convenient | Runtime overhead, GC pauses |
| **Reference Counting (Swift, Python)** | Predictable cleanup | Cycle leaks, overhead |
| **Ownership (Rust)** | Safe with no runtime cost | Complex ownership rules |

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

### Testing the Pipeline

Test at multiple levels:

| Level | What to Test |
|---|---|
| **Unit** | Individual functions — parser output, type checker rules |
| **Integration** | Multiple components — parse + typecheck + codegen |
| **End-to-end** | Full programs from source to result |
| **Snapshot** | IR and error message output — catch regressions |

Start with integration tests for full programs, then add unit tests for complex logic. Add a regression test for every bug fixed.

---

## X. Language Progression

The four stages of building a language from scratch:

| Feature | Calculator | Firstlang | Secondlang | Thirdlang |
|---|---|---|---|---|
| Grammar size | ~18 lines | ~70 lines | ~77 lines | ~140 lines |
| Type System | None | Dynamic | Static | Static + Classes |
| Variables | No | Yes | Yes | Yes |
| Functions | No | Yes | Yes | Yes + Methods |
| Classes | No | No | No | Yes |
| Memory | Stack | Stack | Stack | Stack + Heap |
| Execution | Interpreter / VM / JIT | Interpreter | LLVM JIT | LLVM JIT |

Each stage adds one layer of abstraction:
1. **Calculator** — Learn the basics: parsing, AST, evaluation.
2. **Firstlang** — Add programming: variables, functions, control flow.
3. **Secondlang** — Add types: static checking, LLVM compilation.
4. **Thirdlang** — Add OOP: classes, objects, heap memory management.

---

### What to Explore Next

| Direction | Topics |
|---|---|
| **Language Features** | Inheritance (vtables, dynamic dispatch), interfaces/traits, generics (`List<T>`), closures, pattern matching, algebraic data types |
| **Type System** | Nullability (`Point?`), reference vs owned types, Hindley-Milner full inference |
| **Memory Management** | Garbage collection (mark-and-sweep, generational), reference counting, Rust-style ownership |
| **Execution Models** | AOT compilation to executables, bytecode VMs, transpilation to JS/C/WASM |
| **Optimizations** | Inlining, escape analysis (stack-allocate short-lived objects), devirtualization |
| **Tooling** | Debuggers, formatters, Language Server Protocol (LSP) for IDE support |

The concepts you've learned — grammars, ASTs, type systems, code generation — appear everywhere: SQL, GraphQL, YAML, regex, CSS, template engines. You now have the foundation to understand, modify, or create any of them.

---

*All paths lead to the same truth: computing is structured transformation — from text, to trees, to bytes, to machine code, to electrons.*
