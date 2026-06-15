

# The Complete Programming & Compiler Mental Notes

## I. Hardware Fundamentals: The CPU & Memory

The **Central Processing Unit (CPU)** is a tiny state machine continually operating on main memory (Random Access Memory, or RAM). RAM stores both code and data.

### The Instruction Cycle

A CPU is an implementation of an **Instruction Set Architecture (ISA)**. A standard ISA defines its basic elements (data types, registers, hardware supports, I/O) which make up the lowest-level language of computing: **Machine Language**.

To run code, the CPU continuously performs a fetch-decode-execute loop:

1. **Fetch:** Retrieves instructions currently addressed by the Instruction Pointer (IP), also known as the Program Counter (PC) register.
2. **Decode:** Interprets the meaning of the fetched instruction. This includes an **opcode** (a unique encoding for an operation, acting like the "atoms of computing") and optionally operands (arguments) and/or a prefix (behavioral modifier).
3. **Execute:** A Control Unit (CU) passes instruction-specific signals to functional units like the **Arithmetic Logic Unit (ALU)**, which performs mathematical operations on register values. Results are then written back to memory if applicable.

*Note: Modern CPUs rely on complex optimizations like instruction pipelining and speculative execution to speed up this cycle.*

### Memory Layout

A program must be loaded into memory, creating a process, before it can run. This sets up three special memory locations:

* **Static memory:** Stores global variables and constants.
* **Stack memory:** Stores function frames, including local variables.
> *Analogy:* The stack is like a notepad - you write, you’re done, you tear off the page.


* **Heap memory:** Stores data shared between functions and threads. Objects live on the heap because they need to outlive the function that created them.
> **Analogy:** The heap is like a whiteboard - you write, it stays until you explicitly erase it.

Modern CPUs rely on complex optimizations, like instruction pipelining10 and speculative execution11, to speed up the instruction cycle.

Think of a cpu as an implementaation of an isa (instruction set architecture) - A standard ISA defines its basic elements such as data types, register values, various hardware supports, I/O etc. and they all make up the lowest-level language of computing which is the Machine Language Instructions.
 


## All these three components

All these three components are intertwined together and learning their connections helps you understand what makes *Computing* possible. Informally, a *language* is a structured text with syntax and semantics. A *Source Code* written in a programming language needs a translator/compiler of *some sort*, to translate it to *another* language/format. Then an executor of *some sort*, to execute/run the translated commands with the goal of matching the syntax (and semantics) to *some form* of output.


### [**Instructions and the Machine Language**](https://createlang.rs/crash_course.html#instructions-and-the-machine-language)

If you want to create a “computer” from scratch, you need to start by defining an *abstract model* for your computer. This abstract model is also referred to as [**Instruction Set Architecture (ISA)**](https://en.wikipedia.org/wiki/Instruction_set_architecture) (instruction set or simply *instructions*). A [CPU](https://en.wikipedia.org/wiki/Central_processing_unit) is an *implementation* of such ISA. A standard ISA defines its basic elements such as *data types*, [*register*](https://en.wikipedia.org/wiki/Processor_register) values, various hardware supports, I/O etc. and they all make up the *lowest-level language* of computing which is the [**Machine Language**](https://en.wikipedia.org/wiki/Machine_code)** Instructions.**

Instructions are comprised of *instruction code* (aka *operation code*, in short [**opcode**](https://en.wikipedia.org/wiki/Opcode) or p-code) 
which are directly executed by the CPU. An opcode can either have operand(s) or no operand. 
For example, in an 8-bit machine where instructions are 8 bits, an opcode *load* might be defined by the 4 bits **0011** followed by the second 4 bits as operand with **0101**, making up the instruction **00110101** in Machine Language. The opcode for *incrementing by 1* of the previously loaded value could be defined by **1000** with no operand.

Since *opcodes are like atoms of computing*, they are presented in an opcode table. An example of that is the [x86 opcode reference](https://www.felixcloutier.com/x86/).
[**Assembly Language**](https://createlang.rs/crash_course.html#assembly-language)

Since it’s hard to remember the opcodes by their bit-patterns, we can assign *abstract* symbols to opcodes matching their operations by name. This way, we can create [Assembly language](https://en.wikipedia.org/wiki/Assembly_language) from the Machine Language. In the previous Machine Language example above, **00110101** (means load the binary **0101**), we can define the symbol **LOAD** referring to **0011** as a higher level abstraction so that **00110101** can be written as **LOAD 0101**.

The utility program that translates the Assembly language to Machine Language is called an [**Assembler**](https://en.wikipedia.org/wiki/Assembly_language#Assembler).


A [compiler](https://en.wikipedia.org/wiki/Compiler) is any program that translates (maps, encodes) a language A to language B. Each compiler has two major components:
- **Frontend:** deals with mapping the source code string to a structured format called [**Abstract Syntax Tree (AST)**](https://en.wikipedia.org/wiki/Abstract_syntax_tree)
- **Backend (code generator):** translates the AST into the [Bytecode](https://createlang.rs/crash_course.html#bytecode) / [IR](https://createlang.rs/crash_course.html#intermediate-representation-ir) or Assembly

Most often, when we talk about compiler, we mean [**Ahead-Of-Time (AOT)**](https://en.wikipedia.org/wiki/Ahead-of-time_compilation) compiler where the translation happens *before* execution. Another form of translation is [**Just-In-Time (JIT)**](https://en.wikipedia.org/wiki/Just-in-time_compilation) compilation where translation happens right at the time of the execution.

From the diagram above, to distinguish between a program that translates for example, Python to Assembly vs. Python to Java, the former is called compiler and the latter [**transpiler**](https://en.wikipedia.org/wiki/Source-to-source_compiler) (or source-to-source compiler).


[***Relativity of low-level, high-level***](https://createlang.rs/crash_course.html#relativity-of-low-level-high-level)

Assembly is a *high-level* language compared to the Machine Language but is considered *low-level* when viewing it from C/C++/Rust. High-level and low-level are relative terms conveying the amount of *abstractions* involved.


### [**Virtual Machine (VM)**](https://createlang.rs/crash_course.html#virtual-machine-vm)

[Instructions](https://createlang.rs/crash_course.html#instructions-and-the-machine-language) are hardware and vendor specific. That is, an Intel CPU instructions are different from AMD CPU. A [**Virtual Machine (VM)**](https://en.wikipedia.org/wiki/Virtual_machine#Process_virtual_machines) abstracts away details of the underlying hardware or operating system so that programs translated/compiled into the VM language becomes platform agnostic. A famous example is the [**Java Virtual Machine (JVM)**](https://en.wikipedia.org/wiki/Java_virtual_machine) which translates/compiles Java programs to JVM language aka Java [**Bytecode**](https://en.wikipedia.org/wiki/Java_bytecode). Therefore, if you have a valid Java Bytecode and *Java Runtime Environment (JRE)* in your system, you can execute the Bytecode, regardless on what platform it was compiled on.
---

## II. The Language Hierarchy & Translation

Informally, a language is structured text with syntax and semantics. Source code needs a translator to convert it to another format, and an executor to run it.

### Levels of Abstraction

* **Machine Language:** Raw opcodes and operands directly executed by the CPU (e.g., `00110101`).
* **Assembly Language:** Because bit-patterns are hard to remember, Assembly assigns abstract symbols to opcodes (e.g., `LOAD 0101`). An **Assembler** translates this back to Machine Language.
* **Intermediate Representation (IR):** Any representation between source code and Assembly. Going from one IR to a lower-level IR is called *lowering*.
* **Bytecode:** An intermediate representation that emulates an instruction set with a new, simplified encoding. It is executed by a **Virtual Machine (VM)** (like the JVM), making the code platform-agnostic.

*Note: High-level and low-level are relative terms. Assembly is high-level compared to Machine Code, but low-level compared to Rust or C++.*

### The Compiler

A compiler translates Language A to Language B. It consists of:

* **Frontend:** Maps source code to an **Abstract Syntax Tree (AST)**.
* **Backend (Code Generator):** Translates the AST into Bytecode, IR, or Assembly.

**Translation Types:**

* **Ahead-Of-Time (AOT):** Translation happens *before* execution.
* **Just-In-Time (JIT):** Translation happens *during* execution.
* **Transpiler:** A source-to-source compiler (e.g., translating Python to Java).

---

## III. Execution, Functions, & The AST

### The Execution Pipeline

Your language is a pipeline where data flows through stages:
`Source → Tokens → AST → Output`

Let’s walk through evaluating `1 + 2`:

1. **Grammar:** The rules defining valid code.
2. **Lexer (Tokenizer):** Breaks text into chunks (`"1 + 2"` → `[1, +, 2]`) and tracks location for error messages.
3. **Parser:** Builds a tree structure (AST) from tokens.
> **Analogy:** Think of source code as a sentence and the AST as its diagram. Just like “The cat sat” is diagrammed into subject/verb, `1 + 2` is diagrammed into left/operator/right. The AST captures *structure*, not just text.


4. **Interpreter:** The CPU is the ultimate interpreter. Our software interpreter does the same by walking the AST and evaluating nodes using **recursion** (e.g., evaluate left, evaluate right, apply operator).

### State & Functions

A simple calculator evaluates expressions—input goes in, output comes out. A real language is a **state machine**: it maintains memory (variables), can branch (conditionals), and can loop (while).

### [**Intermediate Representation (IR)**](https://createlang.rs/crash_course.html#intermediate-representation-ir)

Any representation that’s between source code and (usually) Assembly language is considered an [intermediate representation](https://en.wikipedia.org/wiki/Intermediate_representation). Mainstream languages usually have more than one such representations and going from one IR to another IR is called *lowering*.

We explore [LLVM IR](https://en.wikipedia.org/wiki/LLVM#Intermediate_representation) in detail in the [Secondlang IR chapter](https://createlang.rs/03_secondlang/ir.html).

### [**Code Generation**](https://createlang.rs/crash_course.html#code-generation)

[Code generation](https://en.wikipedia.org/wiki/Code_generation_\(compiler\)) for a compiler is when the compiler *converts an IR to some Machine Code*. But it has a wider semantic too for example, when using Rust declarative macro via `macro_rules!` to automate some repetitive implementations, you’re essentially generating codes (as well as expanding the syntax).



> *Calculator is a function - input goes in, output comes out, nothing persists. A real language is a state machine - it maintains memory (variables), can branch (conditionals), and can loop (while). The AST becomes a program to execute, not just an expression to evaluate.*

> *Think of source code as a sentence and the AST as its diagram. Just like “The cat sat on the mat” can be diagrammed into subject/verb/object,* `1 + 2` *can be diagrammed into left/operator/right. The AST is that diagram - it captures structure, not just text.*

Functions are the key to **abstraction**—hiding complexity behind a simple name. When you evaluate a function call like `add(3, 4)`, this sequence occurs:

When we evaluate a function call like `add(3, 4)`, several things happen in sequence. Understanding this sequence is key to understanding how programming languages work.


1. **Look up:** Find the function value by name.
2. **Evaluate:** Compute the arguments (`3` and `4`).
3. **Create a new frame:** Make a fresh environment on the call stack for local variables.
4. **Bind parameters:** Associate parameter names with argument values (`a = 3, b = 4`).
5. **Execute:** Run the function body.
6. **Return:** Pop the frame and give the result back to the caller.


> *The call stack is like a stack of sticky notes. Each function call writes its variables on a new note and puts it on top. When the function returns, you tear off the top note and throw it away. The note underneath becomes current again. This is why* `inner()`*’s variables don’t overwrite* `outer()`*’s - they’re on different notes.*

The call stack is what makes function calls work. Each “frame” on the stack represents one function call in progress. 

### The REPL

A Read-Eval-Print Loop is an interactive environment for your language.

1. **Read:** Get a line of input.
2. **Eval:** Parse and execute it.
3. **Print:** Show the result.
4. **Loop:** Go back to step 1.
- **Variables** (`n`) - Named storage
- **Functions** (`fib`) - Abstraction and reuse
- **Parameters** (passing `n`) - Data flow
- **Conditionals** (`if`/`else`) - Decision making
- **Operators** (`<`, `-`) - Computation
- **Recursion** (`fib` calls `fib`) - Self-reference
- **Call stack** (tracks each frame) - Memory management

If you can implement this, you have a real programming language.

Variables let us store values and refer to them by name. Without variables, we could only work with literal values - every computation would need to repeat its inputs. Variables give us *memory*.


**Frames** - one for each function call. Each function gets its own private storage.

Each of these changes builds on what you already know:

**Grammar grows** - But it’s still pest rules, just more of them.

**AST nodes multiply** - But they’re still Rust enums with the same recursive structure.

**Execution becomes stateful** - But it’s still recursive tree traversal, now with a HashMap.

**Functions add call stacks** - But each frame is just another HashMap, pushed and popped like any stack.

The *concepts* are the same. The *scale* increases.

Let’s walk through this pipeline with a simple example. Say the user types `1 + 2`:
1. **Grammar** - The rules that define what valid code looks like. Our grammar says “an expression can be a number, or two numbers with an operator between them.” The string `1 + 2` matches this pattern. The string `+ + 1` doesn’t.
2. **Lexer (Tokenizer)** - Breaks the text into meaningful chunks. `"1 + 2"` becomes three tokens: `[1, +, 2]`. The lexer also tracks where each token appears in the source code, which helps with error messages later.
3. **Parser** - Builds a tree structure from the tokens. `[1, +, 2]` becomes a tree with `+` at the top and `1` and `2` as children. This tree is the **Abstract Syntax Tree (AST)**.


## IV.  Virtual Machines

Another technique to translate source code to Machine Code is emulating the Instruction Set with a new (human-friendly) encoding (perhaps easier than assembly). [Bytecode](https://en.wikipedia.org/wiki/Bytecode) is such an *intermediate language/representation* which is lower-level than the actual programming language that it was translated from, and higher-level than Assembly language.



Now comes the payoff. We have a tree representing our program. How do we actually *compute* the result?

The CPU is the *ultimate interpreter* - it executes opcodes one at a time. Our interpreter does the same thing at a higher level: it walks the tree and evaluates each node.

The core insight is **recursion**. To evaluate `1 + 2`:
1. Evaluate the left side (`1`) → get `1`
2. Evaluate the right side (`2`) → get `2`
3. Apply the operator (`+`) → get `3`

If the left side were `(3 + 4)` instead of `1`, we’d recursively evaluate that first. This is why trees are so powerful - the structure tells us the order of operations.


 parse to AST, recursively evaluate - is the foundation of *every* interpreter. Python, Ruby, JavaScript interpreters all do this (with more node types, of course).

We’ve seen two ways to execute code: interpret the AST directly, or JIT compile to native machine code. There’s a third approach that sits in between: **compile to bytecode, then interpret that**.

Why would we do this?
- **Simpler than JIT** - No LLVM dependency, works everywhere
- **Faster than AST interpretation** - Bytecode is more compact and cache-friendly
- **Portable** - Same bytecode runs on any machine with our VM

  Why compile to bytecode and interpret that, instead of direct AST interpretation or JIT compiling?

* **Simpler than JIT:** Works everywhere, no LLVM dependency.
* **Faster than AST:** Bytecode is compact and cache-friendly.
* **Portable:** Runs on any machine with your VM.

This is how Python, Java, Ruby, and many other languages work. You compile source code to bytecode once, then run the bytecode interpreter wherever you need it.

---





### [**What Is Bytecode?**](https://createlang.rs/01_calculator/vm.html#what-is-bytecode)

Think of bytecode as a *simplified assembly language* designed for our VM. Real assembly has hundreds of instructions. Our bytecode has just four:

Now we need a machine to execute our bytecode. Our VM is a **stack machine** - it keeps intermediate values on a stack.

`bytecode` - The program to execute
- `stack` - Where we store intermediate values (a fixed-size array for speed)
- `sp` - Stack pointer: points to the next free slot




### The Stack Machine

A Stack Machine is a simple computing model with two main components:

* A **memory (stack)** array keeping intermediate values.
* An **Instruction Pointer (IP)** and **Stack Pointer (SP)**.



**The VM Execution Loop (Fetch-Decode-Execute):**

The VM reads instructions left-to-right with an **instruction pointer (IP)**. For `1 + 2`, it processes `OpConstant(0)` (push 1), `OpConstant(1)` (push 2), `OpAdd` (pop both, push 3), and `OpPop` (return 3). Each opcode manipulates the stack accordingly.


 This is a **fetch-decode-execute** loop:
1. **Fetch** - Read the next byte (`self.bytecode.instructions[ip]`)
2. **Decode** - Match on the opcode to determine what operation to perform
3. **Execute** - Manipulate the stack accordingly
4. **Repeat** - Increment IP and continue until we’re out of instructions

**Why Stacks?** Stacks handle *any* nesting automatically. For `(1 + 2) * (3 + 4)`, you push 1, push 2, add (stack now holds 3). Push 3, push 4, add (stack holds 3, 7). Multiply (stack holds 21). Every operation pops inputs and pushes outputs.  The stack naturally tracks what’s “in progress.”

---

> *Your language is a pipeline. Data flows through stages: Source → Tokens → AST → Output. When something breaks, find which stage produced the wrong output.*
1. **Reproduce** - Find the smallest input that triggers the bug
2. **Isolate** - Which stage is producing wrong output?
3. **Inspect** - Print the data at that stage
4. **Fix** - Change the code
5. **Verify** - Run your test again

This systematic approach works for any compiler bug.

## [**Print the AST**](https://createlang.rs/01_calculator/debugging.html#print-the-ast)

The AST is your program’s structure. When behavior is wrong, print it:

## [**Check Operator Precedence**](https://createlang.rs/01_calculator/debugging.html#check-operator-precedence)

## [**Test Small, Test Often**](https://createlang.rs/01_calculator/debugging.html#test-small-test-often)

Don’t write 100 lines then debug. Test each feature in isolati

## [**Use the REPL**](https://createlang.rs/01_calculator/debugging.html#use-the-repl)

The REPL is your best friend for quick experiments:

if you have wrong result make a tree and check the structure

parse errors - simply input and check syntax

Precedence wrong - Print AST, check grammar order

program crash - Print debug, run clip

Infinite loopAdd print statements in eval loop

## [**The Compilation Pipeline**](https://createlang.rs/transition_2_to_3.html#the-compilation-pipeline)

```
Source → Parse → TypeCheck → Codegen → LLVM IR → Machine Code
```


The transition from Firstlang to Secondlang illustrates a key insight in language design: **types are primarily a semantic addition, not a syntactic one**. The grammar changes are minimal, but the compiler architecture changes significantly.


With types, we tell the compiler *what kind of values* each variable can hold:

Now the compiler catches the bug *before* the program even runs. We know about the problem immediately, not months later in production.

Think of types as a *contract*. When we write `a: int`, we are promising that `a` will always be an integer. The compiler checks that we keep our promises.

Programming languages are divided into two camps:

**APPROACHWHEN TYPES ARE CHECKEDEXAMPLES**[**Static typing**](https://en.wikipedia.org/wiki/Type_system#Static_type_checking)At compile time (before running)Rust, C, Java, Haskell[**Dynamic typing**](https://en.wikipedia.org/wiki/Type_system#Dynamic_type_checking)At runtime (while running)Python, JavaScript, Ruby

Firstlang is dynamically typed. Secondlang is statically typed.

Both approaches have trade-offs:
- **Static**: Catches more bugs early, enables better performance, but requires more upfront type annotations
- **Dynamic**: More flexible, faster to prototype, but bugs can hide until runtime


[**Types Enable Fast Code**](https://createlang.rs/03_secondlang/why_types.html#types-enable-fast-code)

There is another benefit to types: **performance**.

The compiler knows that `x` is always a 64-bit integer. It knows the result is always a 64-bit integer. So it can generate a *single CPU instruction* for the multiplication:

Without types, the interpreter has to do a lot of work at runtime:
1. Check what type `x` is
2. Look up the multiplication operation for that type
3. Check if the operands are compatible
4. Handle potential type errors
5. Finally, do the multiplication

All this checking adds up. A statically typed language can be 10x to 100x faster than a dynamically typed one for number-crunching tasks.

This is why [JIT compilers](https://en.wikipedia.org/wiki/Just-in-time_compilation) like V8 (JavaScript) and PyPy (Python) spend so much effort on *type speculation* - guessing what types values have so they can generate fast code.


[*Type inference*](https://en.wikipedia.org/wiki/Type_inference) means the compiler figures out types when it can:

The [parser](https://en.wikipedia.org/wiki/Parsing) creates an [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree) where some types are marked as `Unknown` (we do not know them yet). The type checker walks through the AST, figures out all the unknown types, and checks that everything is consistent. If something is wrong (like `1 + true`), it reports an error. If everything is okay, we have a fully-typed AST ready for code generation.


*Type inference is like filling in a crossword puzzle. Some squares have letters (explicit annotations), others are blank (Unknown). You use the constraints - “this must be 5 letters”, “it crosses with CAT” - to fill in the blanks. Type inference uses constraints like “this is added to an int, so it must be int” to fill in Unknown types.*
- **Hindley-Milner**: Can infer polymorphic types like `fn identity<T>(x: T) -> T` without any annotations
- **Local inference**: Requires type annotations at function boundaries; infers types *within* function bodies

The key insight: types **flow forward** from known sources (literals, parameters) through operations into variables.

Here is the key idea: types *flow* through expressions. If we know the type of the inputs, we can figure out the type of the output.



The type “flows” from the literals, through the operator, into the variable. No explicit annotation needed.

---

## V. Types & Inference

The transition from a dynamic to a static language illustrates that **types are primarily a semantic addition, not a syntactic one**. The grammar changes are minimal, but the compiler architecture changes significantly.

| Approach | When Checked | Examples |
| --- | --- | --- |
| **Static typing** | Compile time (before running) | Rust, C, Java |
| **Dynamic typing** | Runtime (while running) | Python, JS, Ruby |

### Types Enable Fast Code


Types act as a contract. If the compiler knows `x` and `y` are 64-bit integers, it can generate a *single CPU instruction* for multiplication. Without types, an interpreter must check types, look up operations, check compatibility, and handle errors at runtime. (This is why JIT compilers like V8 spend immense effort on *type speculation*).

### Type Inference

Inference means the compiler figures out types automatically, flowing types forward from known sources (literals, parameters) through operations into variables.

* **Unification:** The process of checking if two types are compatible and finding a common type. It resolves `Unknown` types.
* **The Type Environment:** A symbol table (e.g., `HashMap<String, Type>`) that maps names to types. It is extended on declarations, queried on references, and scoped to allow shadowing.


[**Type Unification**](https://createlang.rs/03_secondlang/inference.html#type-unification)

[**Unification**](https://en.wikipedia.org/wiki/Unification_\(computer_science\)) is the process of checking if two types are compatible and finding a common type. This is a key operation in type inference.

Here is the pseudocode:

```
```

The `Unknown` case is the heart of type inference. When we unify `Unknown` with a concrete type, we *learn* what the unknown type should be.


[**The Type Environment**](https://createlang.rs/03_secondlang/inference.html#the-type-environment)

The **type environment** (also called symbol table or context) maps names to types:

```rust
type TypeEnv = HashMap<String, Type>;
```

The environment is:
- **Extended** when we declare a variable or enter a function (adding new bindings)
- **Queried** when we reference a variable (looking up its type)
- **Scoped** - inner scopes can shadow outer bindings

## [**Function Type Inference**](https://createlang.rs/03_secondlang/inference.html#function-type-inference)

Functions are trickier because we need to handle:
1. Parameters (types come from annotations)
2. Local variables (types are inferred)
3. Return value (must match declared return type)
### Function Type Inference (The Two-Pass Algorithm)

Functions are tricky because they can call each other (mutual recursion).

1. **Pass 1 (Collect Signatures):** Scan all function definitions and record their types *before* checking bodies.
2. **Pass 2 (Check Bodies):** Go through each statement and infer/unify types.


**Pass 1: Collect function signatures**

We scan all function definitions and record their types *before* checking any bodies. Why? Because functions can call each other (mutual recursion):

By collecting all signatures first, [mutual recursion](https://en.wikipedia.org/wiki/Mutual_recursion) works.

**Pass 2: Check each statement**

Now we go through each statement, inferring types as we go.

## [**Type Checking Expressions**](https://createlang.rs/03_secondlang/inference.html#type-checking-expressions)

The pattern is always the same:
1. Recursively type check sub-expressions
2. Apply the typing rule for this expression kind
3. Set the type on this expression

Type inference works by:
1. **Starting with known types**: literals (`42` → Int, `true` → Bool) and annotated parameters
2. **Flowing types through expressions**: operators, function calls, assignments
3. **Recording types in the environment**: so variables can be looked up later
4. **Unifying types**: checking compatibility and resolving `Unknown`
5. **Reporting errors**: when types do not match

The beauty is that most of the time, you only need to annotate function parameters and return types. Everything else is inferred automatically.

---

## VI. Compiler Optimizations

By simplifying the AST before code generation, we make the compiler faster, produce cleaner debug output, and exploit specialized language knowledge. Optimizations are often chained in a **pass pipeline**.

| Optimization | What It Does |
| --- | --- |
| **Constant Folding** | Evaluates known values at compile time (`1 + 2` → `3`) |
| **Algebraic Simplification** | Applies math identities / strength reduction (`x * 0` → `0`) |
| **Dead Code Elimination** | Removes unreachable code |
| **Common Subexpression Elimination** | Computes identical expressions once if used multiple times |
| **Loop Unrolling** | Replaces loops with repeated sequential code |
| **Inlining** | Replaces function calls directly with the function's body |
| **Tail Call Optimization** | Turns tail recursion into a flat loop |

[**Chaining Optimizations**](https://createlang.rs/03_secondlang/optimizations.html#chaining-optimizations)

Multiple optimization passes can be chained. This is called a [**pass pipeline**](https://en.wikipedia.org/wiki/Multi-pass_compiler):

Two passes, significant simplification. The order matters - constant folding first creates opportunities for algebraic simplification.

---

## VII. LLVM IR & Advanced Code Generation

> *LLVM is like a universal translator for CPUs. You speak LLVM IR, and LLVM translates it to x86, ARM, WebAssembly - whatever you need. You write your compiler once; LLVM gives you every platform for free.*

LLVM IR is a universal assembly language. You write your compiler frontend to output LLVM IR, and LLVM optimizes and translates it to native machine code (x86, ARM, WebAssembly).

Think of IR as a *universal assembly language*. It is low-level (close to the machine) but not tied to any specific CPU. LLVM takes IR and produces optimized machine code for whatever platform you are on.

Many languages use LLVM: Rust, Swift, Julia, Kotlin/Native, and more. By using LLVM, we get world-class optimizations for free.

You might wonder: “LLVM will optimize this anyway. Why do it ourselves?”

Good question. LLVM *will* do these optimizations. But:
1. **Learning**: Implementing optimizations helps you understand how compilers work. These are the same techniques used in production compilers.
2. **Simplicity**: Simpler AST means simpler code generation. Less can go wrong.
3. **Debug output**: When you print the AST for debugging, optimized code is easier to read.
4. **Specialized optimizations**: You might know things about your language that LLVM does not. Custom optimizations can exploit that knowledge.
5. **Compile time**: Simpler AST means less work for LLVM, which means faster compilation.

---


**Parameters** (`%a`, `%b`) come in as values
1. **Allocate stack space** with `alloca` - we create local variables `%a.addr` and `%b.addr`
2. **Store parameters** into the stack slots with `store`
3. **Load values** back from stack with `load`
4. **Add** the loaded values with `add i64`
5. **Return** the result


This pattern looks wasteful. Why not just use `%a` and `%b` directly?

The answer is **mutability**. In LLVM IR, values like `%a` are immutable - you cannot change them. But in most languages, variables *can* change:



By storing variables in stack slots (`alloca`), we can modify them:




This is called the **alloca/load/store pattern**. It is simple to generate and LLVM optimizes it away (the [mem2reg pass](https://llvm.org/docs/Passes.html#mem2reg-promote-memory-to-register) promotes stack slots to registers) when possible.


### Core IR Mechanics

* **SSA Form (Static Single Assignment):** Every variable is assigned exactly *once*. This makes optimizations like Dead Code Elimination trivial.
* **Mutability (Alloca/Load/Store):** Because SSA values are immutable, mutable variables are handled by allocating stack space (`alloca`), writing to it (`store`), and reading from it (`load`). LLVM's `mem2reg` pass later promotes these to fast registers.
* **Conditionals & Branching:** Conditionals require separate basic blocks (`entry`, `then`, `else`) and terminator instructions (`br` for branch).
* **Phi Nodes ($\phi$):** In SSA form, if a variable's value depends on a conditional branch (if/else), a Phi node merges the values, selecting the correct one based on which basic block the control flow arrived from.
* **Recursion:** Executed using the standard `call` instruction.

### The JIT Compilation Process

LLVM maintains state via a context, a module (container for functions), a builder (inserts instructions), and variable/function hashmaps.

1. **Pass 1:** Declare all functions (enables mutual recursion).
2. **Pass 2:** Compile function bodies.
3. **Pass 3:** Create an `@__main` wrapper for top-level expressions to give the JIT an entry point, then verify the module.

*Full Pipeline:* `Parse → Type Check → Optimize → Codegen to LLVM IR → JIT machine code → Execute`.



## [**IR for Conditionals**](https://createlang.rs/03_secondlang/ir.html#ir-for-conditionals)

Conditionals require **branching** - jumping to different code based on a condition:

```
```


`icmp sgt` - Integer compare, signed greater than. Returns an `i1` (1-bit integer, a boolean)
- `br i1 %cmp, label %then, label %else` - [Branch](https://en.wikipedia.org/wiki/Branch_\(computer_science\)): if `%cmp` is true, go to `then`, else go to `else`

Notice we have multiple **basic blocks** now: `entry`, `then`, and `else`. Each block ends with a **terminator** (like `ret` or `br`) that says where to go next.

## [**SSA Form**](https://createlang.rs/03_secondlang/ir.html#ssa-form)

LLVM IR uses [**Static Single Assignment (SSA)**](https://en.wikipedia.org/wiki/Static_single-assignment_form) form. This means every variable is assigned exactly once.

Consider this code:

```
```


Each name appears on the left side of exactly one assignment.

Why SSA? It makes optimization easier. The compiler always knows exactly where each value was defined. This enables powerful optimizations like [dead code elimination](https://en.wikipedia.org/wiki/Dead_code_elimination), [constant propagation](https://en.wikipedia.org/wiki/Constant_folding#Constant_propagation), and [common subexpression elimination](https://en.wikipedia.org/wiki/Common_subexpression_elimination).


[**Phi Nodes: Merging Values from Different Paths**](https://createlang.rs/03_secondlang/ir.html#phi-nodes-merging-values-from-different-paths)

SSA creates a problem with conditionals. Consider:

After the if/else, what is `x`? It depends on which branch we took. In SSA, we need different names:

```
```


The answer is a [**phi node**](https://en.wikipedia.org/wiki/Static_single-assignment_form#Converting_to_SSA) (φ). A phi node selects a value based on which block we came from:

```
merge:
  %x = phi i64 [ %x.then, %then ], [ %x.else, %else ]
  ret i64 %x
```




This reads as: “If we came from `%then`, use `%x.then`. If we came from `%else`, use `%x.else`.”

Phi nodes are the only way to merge values from different [control flow](https://en.wikipedia.org/wiki/Control_flow) paths in SSA.

## [**IR for Recursion**](https://createlang.rs/03_secondlang/ir.html#ir-for-recursion)

[Recursive](https://en.wikipedia.org/wiki/Recursion_\(computer_science\)) functions use the `call` instruction:

The `call` instruction calls a function and returns its result. Recursion is just calling the same function from within itself.


Notice the `@__main` function. This is a wrapper we generate for top-level expressions. When you write `fib(10)` at the top level, we wrap it in `__main` so the JIT has something to call.

**context** - The LLVM context. All LLVM objects belong to a context. Think of it as the “workspace” for LLVM.
- **module** - A container for functions. Think of it as a single source file or compilation unit.
- **builder** - The tool we use to create IR instructions. We position it in a basic block and it adds instructions there.
- **variables** - Maps variable names to their stack locations (pointers from `alloca`). When we see `x`, we look it up here to find where it lives in memory.
- **functions** - Maps function names to their LLVM function objects. Needed so we can call functions by name.
- **current_fn** - The function we are currently compiling. Needed to create new basic blocks for conditionals and loops.

## [**The Compilation Process**](https://createlang.rs/03_secondlang/codegen.html#the-compilation-process)

Compilation happens in three passes:


**Pass 1: Declare all functions**

Before we can compile function bodies, we need to know about all functions. Why? Because `foo` might call `bar`, and `bar` might call `foo`. We declare all functions first so calls can find their targets.

A function *declaration* tells LLVM “there is a function with this name and signature” but does not include the body yet.

**Pass 2: Compile function bodies**

Now we go through each function and generate its body - the actual instructions.

**Pass 3: Create the **`__main`** wrapper**

If there is a top-level expression (like `fib(10)`), we wrap it in a `__main` function. This gives the JIT an entry point to call.

**Verify the module**

Finally, we ask LLVM to verify that our IR is well-formed. This catches bugs in our code generator.

Variables are stored on the stack using the **alloca/load/store pattern** we discussed in the [IR chapter](https://createlang.rs/03_secondlang/ir.html#why-all-the-loading-and-storing):
1. When we declare a variable, we use `alloca` to reserve stack space and store the pointer
2. When we read a variable, we `load` from that pointer
3. When we write a variable, we `store` to that pointer

This pattern handles mutable variables naturally and LLVM optimizes it away when possible (promoting stack slots to registers).


[**Function Calls**](https://createlang.rs/03_secondlang/codegen.html#function-calls)
We look up the function, compile each argument, then emit a `call` instruction.

The `try_as_basic_value().unwrap_basic()` deserves explanation. In LLVM, function calls can return either:
- A “basic value” (like an integer or pointer) that we can use
- Nothing (for void functions)

`try_as_basic_value()` returns an enum with both possibilities. Since our functions always return `int`, we know we have a basic value and can safely unwrap it. The `into_int_value()` converts it to the specific integer type we need.


This function:
1. Creates a code generator
2. Compiles the program to IR
3. Creates a JIT execution engine
4. Gets a pointer to `__main` (our entry point)
5. Calls it and returns the result

The JIT engine compiles our IR to native machine code on the fly, then executes it. This is much faster than interpretation because we are running actual machine code, not walking a tree.

The `unsafe` block is required because we are calling raw machine code. We have to trust that our code generator produced valid code.


Here is what happens when you run `cargo run -- examples/fibonacci.sl`:
1. **Parse** the source file → Typed AST (with `Unknown` types)
2. **Type check** → Typed AST (all types resolved)
3. **Optimize** (optional) → Simplified AST
4. **Compile** → LLVM IR
5. **JIT** → Native machine code
6. **Execute** → Result

All in a fraction of a second.


Each step transforms the program into a different representation, getting closer and closer to something the CPU can execute.

---

## VIII. Object-Oriented Concepts (Classes)

What do Classes give us?

* **Grouping:** Related data lives together.
* **Methods:** Functions that explicitly operate on that data.
* **Encapsulation:** Data and behavior in one place.
* **Types:** Defines new data types (e.g., `Point` becomes a type just like `int`).

---

## IX. Debugging

### Debugging the Compiler Pipeline

> *Your language is a pipeline. Data flows through stages: Source → Tokens → AST → Output. When something breaks, find which stage produced the wrong output.*


When a compiler bugs out, systematically find which stage produced the wrong output:

1. **Reproduce:** Find the smallest failing input.
2. **Isolate & Inspect:** Print the data at that stage.
3. **Fix & Verify:** Change the code and run the test again.

The REPL is your best friend for quick experiments:

* *Parse Errors:* Check input syntax and lexer tokens.
* *Precedence Wrong:* Print the AST and check the parser's grammar order.
* *Program Crash:* Print debug info and run `cargo clippy`.
* *Infinite Loop:* Add print statements directly in the `eval` loop.

### Memory Safety Issues

Languages without automatic garbage collectors (like C/C++) require manual allocation (`new`) and deallocation (`delete`), introducing specific, high-risk bugs:

* **Memory Leak:** Forgetting to free memory. It stays allocated, eating RAM until the program exits.
* **Use After Free:** Accessing an object after it was deleted. This results in **Undefined Behavior** (anything can happen).
* **Double Free:** Deleting the same object twice. This causes memory corruption and crashes.
* **Dangling Pointer:** Occurs when multiple variables point to the same object in memory. If one variable deletes it, the remaining variables are left pointing to "dead" memory space.

### [**Memory Leak**](https://createlang.rs/04_thirdlang/memory.html#memory-leak)

Forgetting to `delete`:

```cpp
def leak() {
    p = new Point(1, 2)
    # Oops, forgot delete p!
}   # Memory is lost forever
```

The memory stays allocated until the program exits.

### [**Use After Free**](https://createlang.rs/04_thirdlang/memory.html#use-after-free)

Using an object after deleting it:

```cpp
p = new Point(1, 2)
delete p
p.x   # BUG! Memory already freed
```

This is [**undefined behavior**](https://en.wikipedia.org/wiki/Undefined_behavior) - anything can happen.

### [**Double Free**](https://createlang.rs/04_thirdlang/memory.html#double-free)

Deleting the same object twice:

```cpp
p = new Point(1, 2)
delete p
delete p   # BUG! Already freed
```

Also undefined behavior - might crash, might corrupt memory.

### [**Dangling Pointer**](https://createlang.rs/04_thirdlang/memory.html#dangling-pointer)

Multiple variables pointing to freed memory:

```cpp
p = new Point(1, 2)
q = p              # Both point to same object
delete p
q.x               # BUG! q is now dangling
```



