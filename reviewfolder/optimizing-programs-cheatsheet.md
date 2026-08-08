# Optimizing Programs — Quick-Reference Cheat Sheet

*A condensed index of every rule in the full guide. Each line is the core takeaway — search the full doc for code examples and deep-dives on any topic.*

## 🏁 Golden Rules (Read This First)

- **Start from a good baseline.** Rust is already fast/memory-frugal by default — just don't forget `--release`.
- **Only optimize what's hot.** Optimized code is harder to read and buggier. Spend that budget only where a profiler proves it matters.
- **Algorithms beat micro-tuning.** O(N²) → O(N log N) crushes any low-level trick. Fix the algorithm first.
- **Design for the hardware.** Minimizing cache misses and branch mispredictions genuinely speeds up real CPUs.
- **Small wins compound.** Dozens of 1-2% gains add up — don't dismiss "minor" optimizations.
- **Use multiple profilers.** CPU, memory, and causal profilers each surface different bottlenecks.
- **Two ways to fix a hot function:** make it faster, or call it less often. The second is often the bigger, easier win.
- **Fix silly slowdowns first.** An accidental O(N²) loop, a stray clone, or unbuffered I/O beats inventing a clever optimization.
- **Don't compute what you don't need.** Lazy eval, early returns, and short-circuiting often beat speeding up the computation itself.
- **Fast-path the common case.** If 90% of calls fit a simple pattern, special-case it before the general logic.
- **Compress repetitive data.** Pack common values compactly; fall back to a secondary table for rare ones.
- **Profile before ordering branches.** Put the statistically most frequent case first in `if`/`match`.
- **Cache in front of hot lookups.** A small cache absorbs repeated queries to the same few keys.
- **Comment the *why*.** Explain the profiling data behind a non-obvious optimization, not just what the code does.


## 🏗️ High-Level Design

- **⚖️ The Space vs. Time Trade-off** — Almost every optimization is really a decision about which resource you're willing to spend more of: memory or CPU time. Know which direction you're trading in, and pick deliberately based on what's actually scarce for your program.

## ⌨️ Basic Coding Principles

- **✂️ Eliminate Excessive Function Calls** — Move computations out of loops 🔄 whenever possible. You might even consider selective compromises of program modularity to gain greater efficiency.
- **💾 Eliminate Unnecessary Memory References** — Introduce temporary variables 📝 to hold intermediate results instead of mutating references directly.

## ⚙️ Low-Level Optimizations & Rust Patterns

- **🖲️ Memory References & Passing by Value** — Eliminate unnecessary pointer dereferences by passing small types by value.
- **🔄 Loop Locality & Bounds Checking** — Structure loops to utilize spatial locality and use Iterators to remove hidden bounds checks.

## 🛡️ Bounds Safety, Unchecked Access & Pointers

- **🔓 Unchecked APIs (Bounds & Overflow Check Elimination)** — Only after profiling shows bounds/overflow checks are a hot-path bottleneck and you can prove the access is always valid, use `unsafe { get_unchecked }` or similar.
- **👉 Pointer Arithmetic vs. Pointer Indexing** — Prefer indexing (bounds-checked slice/array access, or iterator-based traversal) over raw pointer arithmetic; reach for pointer arithmetic only in proven hot paths where you've already established the access pattern is safe.

## 📦 Allocation & Collection Management

- **📤 Hoisting Allocations Out of Loops** — Move expensive string and heap allocations out of loops 🔄.
- **📥 Reserve Capacity & Amortized Complexity** — If you know (or can estimate) the final size of a collection, pre-allocate with `with_capacity`. Judge structure cost by average cost over a long sequence of operations, not the worst single call.
- **♻️ Recycle Collections (Allocation Churn)** — Inside a loop that rebuilds the same collection each iteration, `clear()` and reuse it instead of creating a new one.
- **✍️ Append to Strings (Double-Allocation in String Building)** — When building a string piece-by-piece, prefer `write!(&mut s, "...")` (via the `std::fmt::Write` trait) over `s += &format!(...)`.
- **📚 Const Generics: Stack Arrays Instead of Heap `Vec`s** — If a collection's size is fixed and known at compile time (a 3D vector's `[f32; 3]`, a fixed-size buffer, a small lookup table), use a plain array or a const-generic type instead of `Vec<T>`.
- **🏟️ Memory Arenas (Per-Object Allocation Overhead)** — If you need to create thousands of small, short-lived objects (like parsing nodes in a compiler), do not use standard global allocations (`Box::new`). Use an Arena (Bump Allocator).
- **🌍 Global Allocator (Default Allocator Contention)** — For allocation-heavy workloads, swap Rust's default system allocator for a faster drop-in like `mimalloc` or `jemalloc`.
- **📋 Avoid Unnecessary Clones & Copies** — Prefer borrowing (`&T`, `&str`, `&[T]`) and moving over `.clone()`; treat every clone in a hot path as a bug until profiling proves it is cheap.
- **📦 Small-Buffer Optimization (Inline Storage)** — For collections that are usually small (0–N elements with small N), use inline storage (`SmallVec`, `ArrayVec`, `ArrayString`, or a custom `enum { Inline([T; N]), Heap(Vec<T>) }`) to avoid heap allocation in the common case.
- **🧹 Defer Drop (Synchronous Deallocation Stalls)** — If dropping a large object (huge `Vec`, big `HashMap`) is expensive, `send` it to a background thread to be dropped instead of blocking the current one.

## ⚡ Instruction-Level Parallelism & Branch Optimization

- **🧮 Explicit SIMD & Auto-Vectorization** — Prefer idiomatic iterators and contiguous data so LLVM auto-vectorizes; drop to explicit SIMD (`std::simd`, intrinsics, or crates like `wide`) only when the auto-vectorizer fails on a proven hot loop.
- **🔀 Multiple Functional Units & Instruction-Level Parallelism** — Unroll loops and use multiple accumulators to break calculation dependency chains, allowing the CPU to use multiple Arithmetic Logic Units (ALUs) concurrently.
- **🔣 Functional Style Conditional Operations** — Break branching logic by using functional branching (`.map()`, `.filter()`).
- **🔂 Loop Unswitching (Loop-Invariant Branching)** — Move conditional `if` statements that do not change during the loop outside of the loop.
- **🎰 Branch Prediction Hints (Branch Misprediction on Rare Paths)** — For branches where one side is overwhelmingly rare (error handling, panics, one-time setup), mark the rare side `#[cold]` so the compiler optimizes the common path harder.
- **➗ Strength Reduction (Expensive Ops → Cheap Ops)** — Replace expensive arithmetic (division, modulo, multiply by non-constant) with cheaper equivalents the CPU can execute in fewer cycles — shifts, adds, or multiplies by a compile-time inverse.
- **🔗 Loop Fusion & Fission** — Fuse consecutive loops that touch the same data to cut memory traffic; split (fission) a loop only when it enables better vectorization or cache behavior for distinct phases.

## 🧪 Measurement, Testing & Caution

- **📏 Micro-Benchmarking (Dead-Code Elimination in Benchmarks)** — Never trust intuition about which version is faster — measure it with a dedicated benchmark harness, and guard against the compiler "cheating" by optimizing your benchmark away.

## 🔢 Algorithms & Execution Patterns

- **📡 Batching: N+1 Queries & Batch APIs** — Never run network/DB requests or heavy per-item API calls inside a loop. Batch into one bulk request, and design your own APIs to accept slices so callers are not forced into N+1 patterns.
- **🖼️ Sliding Windows (Unlocking $O(N)$ Speed)** — Never use nested `for` loops if you can solve the problem in a single pass. A sliding window tracks a contiguous subset of data, shifting the boundaries instead of recalculating overlapping segments.
- **🧷 Two Pointers & Fast/Slow Pointers** — Turn $O(N^2)$ brute-force searches into $O(N)$ scans by traversing from both ends inward, or use different traversal speeds to detect cycles.
- **🌲 DFS vs. BFS: Choosing the Right Graph Traversal** — Pick your traversal strategy based on what you're looking for, not habit. Depth-First Search (DFS) and Breadth-First Search (BFS) have the same $O(V + E)$ time complexity, but wildly different memory footprints and behavior depending on the shape of the graph.
- **🔁 Recursion → Iteration (Call-Stack Pressure)** — Convert deep or unbounded recursion into an explicit loop (or heap-allocated stack) so you do not blow the call stack and so the compiler can optimize a flat control-flow graph.
- **🧮 Dynamic Programming: Memoization vs. Tabulation** — Never compute the exact same sub-problem twice. Cache redundant work!
- **🧭 Greedy Algorithms & Heuristics** — Don't brute-force an exact optimal answer if it takes $O(2^N)$ time and a "good enough" approximation (or greedy choice) takes $O(N \log N)$.
- **💤 Lazy vs. Eager Evaluation (Unnecessary Upfront Computation)** — Don't compute heavy data transformations until the exact moment you actually need the result. Use Lazy Iterators.
- **🔌 Short-Circuit Evaluation & Early Exit (Fail-Fast)** — Order conditions and checks so the cheapest rejecting work runs first — abort before expensive computation, allocation, parsing, or I/O.
- **🌊 Stream Processing vs. Batch Processing** — Choose streaming when latency, memory footprint, or infinite/unknown input size matter; choose batch when throughput, vectorization, and simple control flow matter.
- **🗜️ Compression (CPU vs. Bandwidth / Storage Trade-off)** — Compress when the cost of CPU cycles to (de)compress is lower than the cost of moving or storing the uncompressed bytes — profile both sides on realistic data.
- **📊 Sorting Algorithm Selection** — Use the standard library sort for almost everything; specialize only when profiling shows sort is hot and your data has exploitable structure (almost-sorted, tiny keys, integers in a narrow range).
- **🔍 Binary Search vs. Hash Lookup** — Prefer `HashMap`/`HashSet` for unstructured key lookup at scale; prefer sorted `Vec` + binary search when the set is small, ordered iteration matters, or you want simpler memory layout and better cache behavior.
- **🤖 Deterministic vs. Non-Deterministic Logic (Memoization Eligibility)** — Isolate non-deterministic operations (randomness, system time, I/O) from your core logic. Favor pure, deterministic functions.
- **🔁 Stateless vs. Stateful Design** — Prefer stateless functions and components by default; introduce state deliberately, only where it earns its keep.

## 🐧 Operating Systems, Kernels, Boot & User Space

- **🥾 Boot Loaders & Early Init** — Boot path length is pure latency before useful work — minimize firmware/bootloader/kernel init for devices that must start fast (embedded, serverless snapshots, appliances).
- **🧱 Kernel vs. User Space** — Cross the kernel boundary as rarely as practical on hot paths. Every syscall is a mode switch, validation, and potential scheduler decision.
- **⚡ Traps, Interrupts, Exceptions & Events** — Treat asynchronous control-flow transfers as expensive — they flush pipelines, disturb cache/TLB locality, and can preempt critical sections.
- **🔄 Processes vs. Threads** — Threads share an address space (cheap communication, careful sync); processes isolate memory (safer, more overhead to spawn and to IPC).
- **📡 IPC (Inter-Process Communication)** — Pick IPC by bandwidth and latency needs — don’t use JSON-over-TCP between two processes on the same machine if shared memory fits.
- **🔀 I/O Multiplexing** — Never block one thread per connection at scale — multiplex readiness (`epoll`/`kqueue`/`IOCP`) or use async runtimes built on them.
- **🔐 Synchronization Across Processes** — Process-shared synchronization must use process-shared primitives (`mutex` with shared memory, file locks, semaphores) — thread-only mutexes do not work across address spaces.

## 🧵 Concurrency, Parallelism & Async

- **🧵 Concurrency with Threads (Spawn & Join Overhead)** — Prefer a fixed-size thread pool (or work-stealing pool) over spawning a fresh OS thread for every unit of work.
- **🔒 Thread Synchronization Strategies (Beyond a Single Mutex)** — Match the synchronization primitive to the access pattern — a global `Mutex` is rarely the right default for hot shared state.
- **⏳ Synchronous vs. Asynchronous & Event Blocking** — Never use synchronous, blocking I/O inside an `async` function.
- **🧑‍💻 Where to Put Async: App vs. Library Internals vs. Library Callers** — `async` is usually great in applications, risky inside a library's internal implementation, and excellent when offered to a library's callers.
- **🔒 Lock Contention (The Concurrency Bottleneck)** — Avoid wrapping highly-contended shared data in a `Mutex`. Prefer hardware-level Atomics, Lock-Free structures, or Message Passing (Channels).
- **⚛️ Avoid Needless Atomics (Atomic vs. Non-Atomic Reference Counting)** — Use `Rc<T>` instead of `Arc<T>` whenever data never crosses a thread boundary.
- **🚧 Cache Line Padding & False Sharing (Multi-Core Write Contention)** — When multiple threads are mutating different variables, ensure those variables do not sit on the exact same CPU cache line. Force them apart using memory alignment padding.
- **🧩 System-on-Chip Awareness: Heterogeneous Cores (Uneven Core Performance)** — On modern SoCs (Apple Silicon, recent Intel laptop/mobile chips, most Android phones), don't assume every core the OS reports is equally fast — spawning `available_parallelism()` identical worker threads can silently bottleneck on the slowest core in the group.
- **🧵 Thread-Local Storage (Avoiding Shared-State Contention)** — When each thread can own its own buffer, counter, or PRNG, use thread-local storage instead of a shared `Mutex`-guarded resource.
- **📡 Signal Handling (Async-Signal-Safety & Hot-Path Interference)** — Keep signal handlers minimal — set a flag or write to a self-pipe — and never do heavy work, allocate, or take locks inside them.

## 🗄️ Data Structure Selection

- **🗃️ Vectors vs. HashMaps** — Default to contiguous memory structures (`Vec`) for small/ordered data, but pivot to `HashMap` when large-scale, repeated lookups are required. Always pre-allocate capacity.
- **⛓️ LinkedLists vs. Contiguous Storage (Cache Locality)** — ❌ AVOID `LinkedList`.
- **🕸️ Graph Representation: Pointer Chasing vs. Index-Based Arenas** — Don't model graph nodes as `Rc<RefCell<Node>>` with pointer-based edges. Store nodes in a flat `Vec` and represent edges as plain integer indices into that `Vec`.
- **📐 Compressed Sparse Row (CSR): The Densest Graph Layout** — For large, mostly-static graphs (millions of nodes, edges rarely change), go a step further than `Vec<Vec<usize>>` and flatten all edges into a single contiguous array using the CSR format.
- **🌳 Binary Search Trees (BSTs)** — If you need to constantly insert data and keep it perfectly sorted, a `Vec` will choke. Use Rust's `BTreeSet` to keep insertions and lookups at $O(\log N)$.
- **📚 Stacks** — Need Last-In-First-Out (LIFO) behavior? You don't need a custom Node struct. Just use a standard `Vec` with `.push()` and `.pop()`.
- **🎲 Probabilistic Data Structures (Memory-vs-Accuracy Trade-off)** — When working with massive datasets where you just need to know if something might exist, do not use a standard Hash Table. Use a Bloom Filter.

## 🏛️ Data Layout & Memory Footprint

- **🔢 Choosing Data Types** — The narrowest type that correctly represents your value range and precision needs is usually the fastest one — every extra byte in a type is an extra byte moved through memory bandwidth and an extra byte competing for cache space.
- **📊 Data-Oriented Design: SoA vs. AoS (Struct Layout & Cache Utilization)** — Structure your data for the CPU cache, not for human readability. Group identical properties together (Struct of Arrays) rather than grouping properties by object (Array of Structs).
- **🔥 Hot/Cold Data Splitting** — Split frequently accessed fields from rarely accessed ones into separate structures (or separate arrays) so hot data stays dense in cache.
- **📏 Struct Padding, Field Reordering & External Padding** — Order fields largest-to-smallest (especially under `#[repr(C)]`) to cut internal padding. Remember trailing (external) padding: `size_of::<T>()` is rounded up to a multiple of alignment, so padding repeats for every array element.
- **📦 Data Layout & Enum Boxing (Oversized Enum Variants)** — Keep your structs and enums small. If an enum has one massive variant, `Box` it to keep the overall footprint tiny.
- **🧮 Bitwise Operations & Bitflags (The Boolean Bloat)** — When tracking multiple true/false states, do not use arrays of `bool`s. Pack them tightly into a single integer using bitwise operations.
- **◀️ Logical vs. Arithmetic Bit Shifts** — Use logical shifts for unsigned data and bit patterns; use arithmetic shifts only when you intentionally want sign extension on signed integers.
- **🏷️ String Interning / Flyweight Pattern (Duplicate String Storage)** — When dealing with thousands of identical string values (like JSON keys, tags, or categories), do not store them as independent strings. Use String Interning.
- **📄 Zero-Copy Parsing & Clone-On-Write (Unnecessary String Duplication)** — When parsing data (like JSON or networking packets), never allocate new `String`s unless you are physically altering the text. Borrow the original buffer using `&str` or `Cow`.

## 🧠 Memory Management Models

- **♻️ Memory Management Strategies: Manual Allocation vs. Garbage Collection vs. Reference Counting vs. Smart Pointers** — Every memory-management strategy trades predictability and raw throughput against safety and programmer effort — know which axis your program actually needs before picking (or fighting) a language's default.
- **🌍 Static Variables vs. Global Variables** — Minimize both, but understand they solve different problems — a `static` gives a value a fixed memory address and program-length lifetime; "global" describes scope (visible everywhere), which is the more dangerous property of the two.
- **🧬 Heterogeneous Data Structures, Unions & Enums** — When elements differ in shape, use a sum type (`enum`) with dense packing; reach for `union` only when you need C-compatible overlay or proven space savings and are willing to manage safety.
- **🪄 Macros (Codegen vs. Runtime Cost)** — Use macros to eliminate repetitive boilerplate and runtime work (generate match arms, lookup tables, parsers) — not to hide heavy runtime logic that should be a plain function.

## 🎛️ Abstraction & Dispatch Costs

- **👉 Function Pointers vs. Generics vs. Closures** — Prefer generics/`impl Fn` for hot call sites (static dispatch, inlining); use function pointers (`fn(...)`) for thin dynamic callbacks and FFI; avoid `Box<dyn Fn>` in tight loops.
- **🎛️ Static vs. Dynamic Dispatch (Virtual Method Table Indirection)** — Prefer Generics (`impl Trait`) over Trait Objects (`Box<dyn Trait>`) unless you absolutely need a collection of mixed types.
- **💥 Panic & Exception Costs vs. `Result`** — Use `Result`/`Option` for expected failure paths; reserve panics for truly unrecoverable bugs. In hot code, avoid patterns that can panic (bounds checks you could prove, `unwrap` on fallible I/O).
- **🧱 Object-Oriented Programming Costs (Inheritance & Virtual Methods)** — Treat classical OOP (deep inheritance, virtual methods, heap-allocated objects) as a design tool, not a performance default — each layer of indirection and dynamic dispatch has a measurable cost.
- **📝 Logging, Tracing & Observability Overhead** — Log and trace the minimum needed for operations; never format expensive strings on a disabled level; sample high-volume traces.
- **📥 Loading Code & Data (Startup Path)** — Defer work that is not needed to serve the first request — lazy-init heavy modules, load configs on demand, and prefer memory-mapping large read-only assets.
- **🖼️ GUI & Interactive UI Performance** — Keep the UI thread free — never do heavy work on the event/render thread; update only dirty regions; target stable frame budgets (e.g. 16 ms for 60 Hz).
- **🔌 Drivers & Peripherals** — Talk to devices in bulk, with interrupt coalescing or polling at high rate; avoid round-tripping userspace↔driver per tiny operation.

## 💾 I/O Optimizations

- **🚿 Buffered I/O (Per-Write Syscall Overhead)** — Never issue raw, unbuffered `read`/`write` calls in a loop. Wrap the handle in a `BufReader`/`BufWriter`.
- **⚙️ Syscall Batching & `io_uring` (Submission Overhead)** — When you issue thousands of small I/O operations per second, batch them (or use `io_uring`) so you pay kernel transition cost once per batch instead of once per operation.
- **🔌 Connection Pooling & Socket Options** — Reuse expensive remote connections (DB, HTTP, TCP) via a pool; tune socket options only after measuring — wrong options can hurt.
- **💽 Memory-Mapped Files (Kernel-to-User Buffer Copying)** — For reading massively large files (gigabytes in size), map them directly into virtual memory instead of chunking them through a `BufReader`.
- **🌐 Choosing Network Layers for Speed** — Use the highest-level protocol that meets your latency/throughput goals — drop down a layer only when profiling shows the upper layer is the bottleneck.
- **📄 Data Formats, Serialization & Endianness** — Pick format by read/write pattern — not by popularity. On hot paths prefer compact binary over text; fix endianness explicitly at the boundary; avoid re-serializing the same object repeatedly.

## 🖥️ What Makes Newer Computers Faster (And How Code Should Adapt)

- Performance gains across hardware generations are not uniform — modern speedups come from parallelism, memory hierarchy, and specialized units more than from higher single-thread clock speeds. Write code that feeds those strengths.
- **🏷️ Metadata Costs (Indexes, Schemas, Alloc Headers)** — Metadata is not free — allocator headers, length fields, vtable pointers, index structures, and schema descriptors consume RAM and cache bandwidth. Keep metadata proportional to the value it provides.
- **📚 Library Headers (Include Cost, API Surface & Header-Only Libraries)** — Treat public headers as part of your build-time performance surface — every include, template, and macro in a widely used header is paid by every translation unit that pulls it in.

## 🔬 Hardware-Aware Optimizations

- **📄 Huge Pages (TLB Pressure)** — For multi-gigabyte working sets with random or wide sequential access, consider huge pages (2 MB / 1 GB) to cut TLB misses.
- **🔮 Manual Prefetching (Hiding RAM Latency)** — When you're about to iterate through memory in a predictable but non-linear pattern (e.g., following a list of indices), hint to the CPU to start loading the next chunk into cache before you actually need it.
- **🔗 Unified Memory Architecture (CPU-GPU Buffer Duplication)** — On SoCs with an integrated GPU (Apple Silicon, most mobile chips, Intel/AMD APUs), don't blindly port desktop-GPU code that copies buffers between "CPU memory" and "GPU memory" — on these chips that memory is often the same physical RAM.
- **🏛️ RISC vs. CISC (Instruction Set Architecture Trade-offs)** — Understand that x86/x86-64 (CISC) and ARM/RISC-V (RISC) execute your Rust code through fundamentally different instruction pipelines — write portable, branch-light idioms that compile well on both, and gate any architecture-specific intrinsics behind `cfg(target_arch)` instead of hardcoding one ISA.
- **🔐 Fast Hashing Algorithms (Cryptographic Hashing Overhead)** — Unless your `HashMap` is taking un-sanitized input directly from the internet, replace Rust's default hasher to instantly double your lookup speeds.
- **🎲 Faster RNG (CSPRNG Overhead)** — If you don't need cryptographic security (e.g., simulations, games, sampling), swap a CSPRNG like `rand`'s default for a fast PRNG like `SmallRng`/`fastrand`.

## 🎛️ Domain-Specific: Audio, Video, VR, Mobile & Quantum

- **🔊 Audio (Real-Time Callbacks)** — Audio callbacks run under hard deadlines (e.g. every few ms). Never allocate, lock, block on I/O, or take unbounded time on the audio thread.
- **🎬 Video (Throughput + Latency)** — Move pixels as little as possible; use hardware codecs/display paths; pipeline stages so decode, process, and present overlap.
- **🥽 Virtual Reality (Motion-to-Photon)** — Missed frame deadlines cause discomfort. Budget time strictly; prioritize pose prediction and late-latching over visual luxury.
- **📱 Mobile Devices** — Optimize for battery, thermals, and intermittent connectivity — not just peak benchmarks on a plugged-in flagship.
- **📶 Bluetooth & BLE Optimization** — Bluetooth performance is mostly about radio on-air time, connection parameters, and payload packing — not CPU micro-opts. Every extra advertisement, reconnect, or tiny packet costs energy and latency.
- **⚛️ Quantum Computing (Hybrid Classical–Quantum)** — Today’s quantum programs are hybrid — classical host code optimizes circuit submission, shot count, and post-processing; quantum speedup is lost if the classical pipeline is wasteful.

## 🤖 Optimizing Code for AI / ML Hardware

- ML workloads are usually memory-bandwidth bound or tensor-core bound, not classic scalar-CPU bound. Optimize data movement, layout, batching, and precision before micro-tuning host-side loops.

## 📟 Optimizing for Embedded, IoT & Constrained Devices

- On small devices the scarce resources are RAM, flash, power, and often a single slow core — optimize for size and predictability first; “throughput at all costs” techniques from servers can make things worse.

## 🧰 Virtualization, Emulation & Sandboxing Costs

- Each layer of virtualization or emulation multiplies overhead — avoid nested abstraction on hot paths; give VMs/containers clear CPU/memory topology; prefer paravirtualized or hardware-accelerated I/O.

## ☁️ Cloud & Multi-Tenant Optimizations

- In the cloud you optimize for cost, tail latency, and noisy neighbors — not just raw single-machine throughput. Design for horizontal scale, fast startup, and efficient idle.

## 🔧 LLVM, Compilers & Writing Programming Languages

- **🧬 Using LLVM Effectively** — LLVM optimizes what it can see and prove — feed it clear IR, stable types, and whole-program visibility when performance matters.
- **🛠️ Writing Your Own Language (Performance-Relevant Choices)** — Language design is performance design — value representation, evaluation strategy, and memory model set the ceiling before any optimizer runs.

## 🧠 Writing & Serving LLM Systems

- LLM performance is dominated by memory bandwidth, KV cache management, batching, and kernel quality — not by clever scalar micro-opts in Python glue code. Apply the general accelerator rules in Optimizing Code for AI / ML Hardware (residency, fusion, precision, prefetch); below is what is specific to LLMs.

## 🏗️ Compiler, Build & Linking

- **⏱️ Compile-Time Evaluation (Runtime Computation of Constants)** — If a value is known at compile time, never calculate it during program execution. Use `const fn`.
- **📞 Calling Conventions (Register Args vs. Stack Traffic)** — Keep hot-path functions compatible with the platform ABI's register-argument limit so arguments stay in registers instead of spilling to the stack; avoid unnecessary large-by-value passes.
- **📋 Explicit Inlining (Cross-Crate Inlining Limits)** — Use `#[inline]` for tiny, ultra-hot-path functions that are called thousands of times inside loops, especially across crate boundaries.
- **🧊 Instruction Cache Pressure (I-Cache Bloat from Over-Inlining)** — Inlining and monomorphization aren't free — don't blanket `#[inline(always)]` everything or lean on huge generic functions instantiated for dozens of types, or you'll blow out the instruction cache (I-cache) and make things slower.
- **🐇 Release Mode (Debug Build Overhead)** — Never benchmark or ship `cargo build` (debug mode). Always use `cargo build --release`.
- **🎯 Target Native CPU (Lowest-Common-Denominator Codegen)** — If you control the deployment machine, compile for its exact CPU instead of a generic baseline.
- **🪶 Binary Size Reduction (Binary Size vs. Speed Trade-off)** — If startup time, download size, or embedded flash space matters more than raw runtime speed, tune the release profile to shrink the binary instead of maximizing throughput.
- **🔬 Inspecting Machine Code (Verifying Generated Assembly)** — For small, extremely hot functions, don't guess whether an optimization "worked" — look at the generated assembly directly.
- **🔗 Link-Time Optimization (Cross-Crate Optimization Boundaries)** — Don't just rely on `cargo build --release`. Enable LTO in your `Cargo.toml` for production builds.
- **📈 Profile-Guided Optimization (Static Heuristics vs. Measured Profiles)** — For your most performance-critical binaries, go beyond static analysis — run the program on real workloads first, then feed that data back into a second, smarter compilation pass.
- **🔗 Symbol Visibility & Dead-Code Stripping** — Alongside LTO and static/dynamic linking choices, export only the symbols you must — every public/`#[no_mangle]` symbol is a root the linker must keep.
- **🔗 Dynamic Link Libraries (DLLs) vs. Static Linking** — Static-link for the fastest, most predictable single binary; dynamic-link (DLLs on Windows, `.so` shared objects on Linux, `.dylib` on macOS) when you need to share code across multiple processes or patch a dependency without recompiling everything that uses it.

## 🌐 General Performance Principles (Language-Agnostic)

- **🐍 Interpreted vs. Compiled Execution** — Understand which execution model your language uses, because it determines where your performance ceiling is and which of the optimizations in this document even apply.