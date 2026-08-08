# Optimizing Programs (Rust + Systems)

- **Last reviewed:** 2026-08-08
- **Toolchain scope:** Guidance is written for current stable Rust behavior and common Linux/macOS/Windows tooling. Verify platform/toolchain specifics in linked docs before applying low-level tweaks.
- **Core principle:** **Measure first.** Optimize only verified bottlenecks.

## Table of Contents

1. [Quick reference (start here)](#quick-reference-start-here)
2. [Mindset and measurement](#mindset-and-measurement)
3. [Algorithms and high-level design](#algorithms-and-high-level-design)
4. [Data structures and data layout](#data-structures-and-data-layout)
5. [Allocation and I/O](#allocation-and-io)
6. [Concurrency and async](#concurrency-and-async)
7. [Low-level hardware/compiler topics](#low-level-hardwarecompiler-topics)
8. [Domain-specific starter checklists](#domain-specific-starter-checklists)
9. [Case studies and benchmark examples](#case-studies-and-benchmark-examples)
10. [Extended optimization pattern catalog](#extended-optimization-pattern-catalog)
11. [Performance myths (nuanced)](#performance-myths-nuanced)
12. [Profiling toolbox](#profiling-toolbox)
13. [Further reading (official docs first)](#further-reading-official-docs-first)

---

## Quick reference (start here)

### Quick checklist

- [ ] Confirm correctness first (tests pass, behavior understood).
- [ ] Build in release mode for measurement (`cargo build --release`).
- [ ] Reproduce workload realistically (input size, data shape, concurrency).
- [ ] Profile before changing code.
- [ ] Fix algorithmic complexity before micro-optimizations.
- [ ] Improve memory locality and allocation behavior.
- [ ] Re-measure and keep only changes that help enough to justify complexity.
- [ ] Re-check correctness after each optimization.

### Optimization decision tree

```text
Program is too slow?
├─ No  -> stop.
└─ Yes -> measured in release mode with representative workload?
   ├─ No  -> measure first.
   └─ Yes -> primary bottleneck?
      ├─ Algorithmic complexity -> redesign algorithm/data flow
      ├─ Memory/cache misses     -> improve layout/locality
      ├─ Allocation churn        -> reserve/reuse/arena (with guardrails)
      ├─ I/O/syscall overhead    -> batch/buffer/async pipeline
      ├─ Lock contention         -> redesign ownership/synchronization
      ├─ CPU hot loop            -> inspect asm + vectorization opportunities
      └─ Tail latency spikes     -> remove pauses/locks/GC-like bursts
```

### Quick diagnosis table

| Symptom | Likely cause | Start with |
|---|---|---|
| Slow for all inputs | Algorithmic complexity | Complexity + data flow review |
| Fast in tiny tests, slow in production | Cache, allocation, I/O, contention | Sampling profiler + perf counters |
| High CPU, low throughput | Busy-waiting, branch miss, poor vectorization | `perf stat`, flamegraph, asm view |
| Poor scaling with more cores | Lock contention, memory bandwidth limits | contention profiling + design change |
| P99 latency spikes | allocator pauses, lock convoy, sync I/O | allocation/lock tracing + buffering |
| Embedded/device misses deadlines | cache/ISR budget/power states | target-hardware tracing + no_std budget checks |

### When **not** to optimize

Do **not** optimize yet when:

- You do not have measurements.
- The code is still changing rapidly for feature reasons.
- The optimization would reduce clarity/safety with negligible measured benefit.
- The bottleneck is external (network, disk, DB) and code-level tuning is not the dominant factor.

Premature optimization often increases maintenance cost and bug risk without user-visible gains.

---

## Mindset and measurement

### Measurement-first rules

1. Benchmark/profile in **release** mode.
2. Use representative datasets and concurrency levels.
3. Track both throughput and latency percentiles (not only averages).
4. Keep a baseline and compare after each change.
5. Prefer reversible, isolated optimizations.

### Correctness before speed

- A fast wrong answer is still wrong.
- Add regression tests around hot code before risky changes (`unsafe`, custom allocators, lock-free structures).
- Use property-style checks for algorithm rewrites where practical.

### Practical measurement workflow (repeatable)

1. Define one user-visible objective (e.g., p99 latency from 180ms to <120ms).
2. Capture baseline metrics in release mode.
3. Identify top bottlenecks with a profiler.
4. Form one optimization hypothesis.
5. Implement one isolated change.
6. Re-measure and keep/discard based on results.
7. Record what changed, where, and why.

This avoids unbounded tuning sessions and makes performance work reviewable.

### Release-profile sanity checklist

- Verify `cargo build --release` for all benchmark runs.
- Ensure benchmark input cannot be constant-folded.
- Confirm CPU frequency scaling / thermal throttling is not dominating variance.
- Repeat benchmarks and compare confidence intervals, not single runs.

### Minimal benchmark harness pattern (Criterion)

```rust
use criterion::{criterion_group, criterion_main, Criterion};
use std::hint::black_box;

fn work(xs: &[u64]) -> u64 {
    xs.iter().copied().sum()
}

fn bench_work(c: &mut Criterion) {
    let data: Vec<u64> = (0..100_000).collect();
    c.bench_function("work_sum", |b| {
        b.iter(|| work(black_box(&data)))
    });
}

criterion_group!(benches, bench_work);
criterion_main!(benches);
```

Notes:

- `std::hint::black_box` reduces constant-folding/dead-code-elimination artifacts.
- Criterion provides warmup/statistics/regression detection that ad-hoc timers usually miss.

---

## Algorithms and high-level design

Big wins usually come from algorithm/data-flow changes.

### Complexity beats instruction-level tweaks

- Replacing `O(N^2)` with `O(N log N)` or `O(N)` often dominates any micro-optimization.
- Avoid repeated work in loops (batching, memoization with bounded memory, incremental updates).

### Sliding window: true `O(N)` implementation

`windows(k).map(|w| w.iter().sum())` is still `O(N * k)` because each window re-sums `k` items.

```rust
fn max_window_sum(xs: &[i64], k: usize) -> Option<i64> {
    if k == 0 || k > xs.len() {
        return None;
    }

    let mut sum: i64 = xs[..k].iter().copied().sum();
    let mut best = sum;

    for i in k..xs.len() {
        sum += xs[i] - xs[i - k];
        if sum > best {
            best = sum;
        }
    }

    Some(best)
}
```

Defined behavior:

- `k == 0` -> `None`
- `k > len` -> `None`

### Two-pointer pattern with edge-case + overflow handling

```rust
fn has_pair_with_sum(sorted: &[i64], target: i64) -> bool {
    if sorted.len() < 2 {
        return false;
    }

    let mut l = 0usize;
    let mut r = sorted.len() - 1;

    while l < r {
        match sorted[l].checked_add(sorted[r]) {
            Some(s) if s == target => return true,
            Some(s) if s < target => l += 1,
            Some(_) => r -= 1,
            None => {
                // Overflow implies magnitude too large for i64 addition.
                // Compare using wider type to preserve ordering.
                let s128 = sorted[l] as i128 + sorted[r] as i128;
                if s128 < target as i128 {
                    l += 1;
                } else {
                    r -= 1;
                }
            }
        }
    }

    false
}
```

### Greedy algorithms: validate assumptions

Greedy coin change is optimal only for some coin systems (canonical systems). For arbitrary denominations, dynamic programming may be required for optimality.

### Batching and N+1 elimination

If a loop performs remote calls (DB/network/fs metadata), you often get larger wins by collapsing calls than by speeding the loop body.

```text
N calls in loop:      latency ~= N * round_trip + compute
single batched call:  latency ~= 1 * round_trip + compute + merge
```

Start with:

- batched read APIs (`IN (...)`, bulk endpoints),
- batched writes (transactions / multi-row inserts),
- API signatures that accept slices/iterators instead of single items.

### BFS vs DFS: choose by objective

| Goal | Usually prefer | Why |
|---|---|---|
| Shortest path in unweighted graph | BFS | explores by hop distance |
| Deep exploration/backtracking | DFS (iterative) | low overhead frontier stack |
| Unbounded depth from untrusted input | iterative DFS/BFS | avoids recursion stack overflow |

### Recursion to iteration in hot or deep paths

Rust does not guarantee tail-call optimization. Convert deep recursion to explicit stack/loop when depth can grow with input.

---

## Data structures and data layout

### Choose structures for access patterns

- `Vec<T>`: contiguous data, strong cache locality.
- `VecDeque<T>`: good for queue semantics.
- `HashMap<K, V>`: average-case fast lookup; random hashing defends against collision attacks.
- Linked structures can hurt locality for traversal-heavy hot paths.

### `HashMap` hasher guidance (security/perf)

Rust standard `HashMap` uses a randomized default hasher (`RandomState`; algorithm choice is implementation detail and may change). It is generally a safer default for untrusted keys.

Use a faster non-cryptographic hasher only when:

1. Profiling shows hashing is a real bottleneck, **and**
2. Keys are trusted or collision-DoS risk is otherwise mitigated by your threat model.

### Layout and locality

- Keep hot fields together.
- Split cold metadata away from hot loops.
- Consider SoA (structure of arrays) over AoS when processing single fields across many elements.
- Measure before/after with cache-miss counters.

### \"Which structure should I try first?\" quick table

| Access pattern | Candidate | Why |
|---|---|---|
| append + iterate | `Vec<T>` | contiguous, simple, fast default |
| queue (pop front + push back) | `VecDeque<T>` | avoids frequent shifting |
| membership tests on unique keys | `HashSet<T>` | expected O(1) lookup |
| ordered lookup / range queries | `BTreeMap<K,V>` | stable ordering + range APIs |
| many tiny bit flags | bitflags/bitset | compact memory footprint |

### Hot/cold split sketch

```rust
struct EntityHot {
    pos: [f32; 3],
    vel: [f32; 3],
}

struct EntityCold {
    debug_name: String,
    ui_state: u32,
}
```

The idea is to keep frequently-touched fields cache-dense while moving rarely-read fields out of the hot walk.

### Bloom filter sanity check

For `n` inserted items, false-positive rate `p`, required bits:

`m = -n * ln(p) / (ln(2)^2)`

Example: `n = 1_000_000_000`, `p = 1%`

- `m ≈ 9.6e9 bits ≈ 1.2 GB` (not megabytes)
- Optimal number of hashes: `k ≈ (m/n) * ln(2) ≈ 7`

Use realistic memory/accuracy calculations when proposing probabilistic structures.

---

## Allocation and I/O

### Allocation principles

- `Box::new`, `Vec` growth, `String` growth generally go through allocator APIs; not one OS syscall per allocation.
- Reserve capacity when a good upper bound exists.
- Reuse buffers (`clear`) in loops to reduce churn.
- Arena/slab allocators can help some workloads, but increase lifetime complexity.

`Vec` growth policy is intentionally not a stable API guarantee. Avoid relying on exact doubling factors.

### Buffered and vectored I/O

```rust
use std::io::{self, IoSlice, Write};

fn write_record_once<W: Write>(mut w: W, key: &[u8], value: &[u8]) -> io::Result<usize> {
    let sep = b":";
    let nl = b"\n";
    let bufs = [IoSlice::new(key), IoSlice::new(sep), IoSlice::new(value), IoSlice::new(nl)];
    w.write_vectored(&bufs)
}
```

`write_vectored` may write only part of the data; production code should loop until all bytes are written (or use a helper abstraction that guarantees full writes).

If you need OS-handle interoperability, prefer modern FD/handle traits (e.g., `AsFd`/`AsHandle`) to avoid lifetime bugs around raw descriptors.

### String/bytes construction patterns

Common anti-pattern in hot paths:

- repeated `format!` + concatenation inside loops,
- repeated UTF-8 conversions without reuse.

Preferred pattern:

- reserve approximate output size,
- append into a reusable `String`/`Vec<u8>`,
- reuse buffers across requests/frames.

### `io_uring` and advanced async I/O (Linux)

`io_uring` can reduce syscall overhead and improve batching for some workloads, but gains depend on kernel version, operation mix, and queue depth. Treat as an advanced optimization after profiling conventional buffered/async pipelines.

### Memory mapping: when it helps, when it does not

`mmap` can reduce explicit copy paths and simplify random access, but it is not automatically "zero-copy" end-to-end and is not always faster than buffered I/O.

Trade-offs include:

- page-fault behavior,
- access pattern locality,
- file size and eviction pressure,
- platform-specific semantics.

Measure against buffered approaches for your workload.

---

## Concurrency and async

### Concurrency model selection

- Use threads/Rayon for CPU-bound parallelism.
- Use async/Tokio for high-concurrency I/O-bound services.
- Hybrid designs are common.

### Atomics and contention

Atomics provide synchronization guarantees, but contended updates can still serialize through cache coherency traffic. They do not make shared increments "simultaneous" in the throughput sense.

Use techniques like sharding, per-thread aggregation, or reduced write sharing to lower contention.

### Synchronization choice cheat sheet

| Situation | Typical tool | Notes |
|---|---|---|
| many reads, few writes | `RwLock` | measure write-heavy regressions |
| tiny shared counters | atomics + sharding | avoid one global hot counter |
| ownership transfer | channels | often simplifies locking model |
| CPU parallel map/reduce | Rayon | work-stealing is usually enough |

### Async entry points and blocking boundaries

```rust
#[tokio::main(flavor = "multi_thread")]
async fn main() {
    // Keep blocking CPU work off async worker threads:
    // use spawn_blocking or a dedicated CPU pool when needed.
}
```

### False sharing and cache-line padding

Cache-line padding may help when independent hot counters are written by different cores, but it increases memory footprint. Apply only after measuring cross-core invalidation issues.

### Signals and global state safety

- Keep signal handlers minimal and async-signal-safe.
- In Rust, prefer safe coordination primitives (`Atomic*`, channels, `OnceLock`) over mutable global statics.

### Tokio + Rayon together

A common production pattern is:

- Tokio for network/socket/timer orchestration,
- Rayon (or dedicated blocking pools) for CPU-heavy transforms.

Avoid long CPU loops on async executor workers.

---

## Low-level hardware/compiler topics

### Function calls, inlining, and registers

A function call does **not** inherently mean "save all CPU state" or "touch RAM." Modern compilers can inline, keep values in registers, and optimize calling conventions aggressively.

Avoid blanket rules like "never call functions in loops"; verify with profiling and generated code.

### Reading optimized assembly without overfitting

Check for:

- whether expected inlining occurred,
- whether bounds checks remain in hot loops,
- whether vector instructions appear where expected,
- whether branches match your mental model.

Then return to end-to-end metrics. Better-looking assembly is not automatically better product latency.

### Locals and references: avoid absolute claims

Locals are not guaranteed to stay in registers/L1, and mutating through a reference is not guaranteed to force a RAM write each iteration. Actual behavior depends on optimization level, aliasing, and code shape.

### Iterators vs indexing

Iterator style can improve clarity and sometimes aid optimization, but it does not inherently remove all bounds checks or guarantee SIMD. Indexed loops can optimize equally well.

Use whichever representation keeps invariants obvious; when performance differs, keep the faster one only if the measured gain justifies complexity.

### Functional combinators and branches

Combinators like `map`/`filter` do not inherently produce branchless machine code. Branch behavior depends on predicate/data distribution and compiler transforms.

### Signed shift vs division

Right shift on signed integers is arithmetic shift (sign-extending). Rust integer division `/` rounds toward zero. For negatives, `x >> n` and `x / (1 << n)` can differ.

### SIMD status

Do not assume `std::simd` portability/stability status from memory. Check current official Rust docs for your toolchain.

In practice:

- Auto-vectorization can already provide wins.
- `std::arch` intrinsics are available for target-specific paths.
- Keep SIMD paths behind clear feature/target guards and benchmark them.

If you evaluate `std::simd`, verify its current stability/support matrix directly in official docs for your current stable toolchain.

### PGO/LTO/target-cpu and architecture-specific tuning

Potentially high impact, but always measure:

- Cargo profiles (`opt-level`, `lto`, `codegen-units`, `panic`).
- PGO/BOLT-like workflows where supported.
- `-C target-cpu=native` for machine-specific binaries.

Guardrails:

- Document reproducibility implications.
- Validate on deployment hardware, not just dev machines.

### Prefetch, huge pages, unsafe micro-tuning

Use only with strong evidence from counters/profiles; these techniques are workload- and platform-sensitive and can regress performance.

### PGO quick path (conceptual)

1. Build instrumented binary.
2. Run representative production-like workload.
3. Rebuild with collected profile data.
4. Compare end-to-end metrics and binary size.

Keep a non-PGO fallback path in CI/release process.

---

## Domain-specific starter checklists

### Web services

- [ ] Measure p50/p95/p99 under realistic concurrent load.
- [ ] Eliminate N+1 DB/query patterns.
- [ ] Reuse connections and enable request/response buffering appropriately.
- [ ] Keep CPU-heavy work off async reactor threads.
- [ ] Tune allocator/profile memory if allocation is hot.

### Games / interactive software

- [ ] Track frame budget and frame-time variance (not only mean FPS).
- [ ] Keep hot frame data contiguous and cache-friendly.
- [ ] Avoid per-frame allocation churn.
- [ ] Profile on target hardware/driver stack.
- [ ] Use staged quality degradation for overload scenarios.

### Embedded systems

- [ ] Define deterministic latency/ISR budgets.
- [ ] Minimize dynamic allocation in real-time paths.
- [ ] Use `no_std`-appropriate synchronization and interrupt-safe design.
- [ ] Measure power + thermal behavior on device.
- [ ] Validate memory footprint and stack depth bounds.

### ML / LLM workloads

- [ ] Separate tokenization, model compute, and I/O/network costs.
- [ ] Measure throughput and latency per batch size.
- [ ] Validate memory bandwidth/NUMA placement effects.
- [ ] Profile kernels before custom fusion/quantization work.
- [ ] In GPU paths (e.g., `wgpu` or vendor stacks), measure transfer overlap and kernel occupancy before micro-tuning.

---

## Case studies and benchmark examples

### Representative benchmark example format (illustrative)

The table structure below is illustrative. Do not treat numbers as universal; collect your own reproducible measurements.

| Change | Benchmark setup | Before | After | Notes |
|---|---|---:|---:|---|
| Reserve vector capacity | Criterion, 1M pushes, release build | (measure) | (measure) | Often reduces realloc churn/variance |
| Batch DB fetches | realistic staging load test | (measure) | (measure) | Usually large latency wins |
| Swap AoS -> SoA in hot loop | perf + cache counters | (measure) | (measure) | Validate cache-miss delta |

### Concrete before/after template you can run

```rust
fn fill_no_reserve(n: usize) -> Vec<u64> {
    let mut v = Vec::new();
    for i in 0..n as u64 {
        v.push(i);
    }
    v
}

fn fill_reserve(n: usize) -> Vec<u64> {
    let mut v = Vec::with_capacity(n);
    for i in 0..n as u64 {
        v.push(i);
    }
    v
}
```

Benchmark these two with Criterion and record median + confidence interval.

### Concise case study (illustrative workflow)

Scenario: API endpoint has high p99 latency.

1. Measure: flamegraph shows heavy time in repeated small DB queries.
2. Change: replace per-item query loop with batched query + map join.
3. Verify correctness with endpoint regression tests.
4. Re-measure: check p50/p95/p99 and DB load.
5. Keep only if improvement is material and code remains maintainable.

(Workflow is real; exact numbers intentionally omitted unless reproducibly measured.)

### Additional mini case study: allocation churn in parser loop

Scenario: parser throughput is unstable under burst load.

1. Measure: allocation profiler shows heavy `String`/`Vec` churn in token loop.
2. Change: pre-reserve buffers + reuse per-request scratch buffers.
3. Validate: parser regression tests + malformed-input tests.
4. Re-measure: throughput and tail-latency variance.
5. Decision: keep if gains persist across input shapes (small/medium/large payloads).

Again, numbers are intentionally omitted until measured in your environment.

---

## Extended optimization pattern catalog

These are high-value patterns from day-to-day systems work. Treat them as candidates, not defaults.

### API and abstraction patterns

- Prefer static dispatch (`impl Trait`/generics) in hot paths when code-size impact is acceptable.
- Use dynamic dispatch where flexibility is needed and call frequency is low/moderate.
- Keep trait-object boundaries away from tight numeric loops if profiling says virtual-call overhead matters.

### Loop and control-flow patterns

- Fuse passes when two sequential loops touch the same data and fusion does not hurt clarity.
- Split (fission) loops when one mixed loop blocks vectorization or creates branch-heavy hot paths.
- Hoist loop-invariant work outside loops.
- Use early-exit checks for negative/rare cases where correctness allows.

### Allocation and lifetime patterns

- Use request/frame-local scratch arenas when many temporary allocations share lifetime.
- Recycle frequently used buffers (`clear`, retain capacity).
- Prefer borrowing (`&str`, `&[u8]`, `Cow`) over cloning when ownership transfer is unnecessary.

### Logging and observability overhead

- Avoid expensive formatting on disabled log levels.
- Sample high-volume traces in hot paths.
- Distinguish profiling mode from production observability budgets.

### Startup and binary-size patterns

- Defer expensive initialization until first real use when startup latency matters.
- Keep an eye on code-size growth from aggressive monomorphization/inlining.
- Reassess `lto`, `codegen-units`, and panic strategy per deployment target.

### Data movement patterns

- Prefer operating on slices/views instead of repeatedly materializing intermediate vectors.
- Batch small writes/reads to reduce syscall and framing overhead.
- Avoid unnecessary encode/decode hops across layers.

### Guardrails for `unsafe` optimization

- Document invariants in `// SAFETY:` comments.
- Add targeted tests around the invariants.
- Keep a safe equivalent reference implementation where practical for cross-checking.

---

## Performance myths (nuanced)

| Myth | Better framing |
|---|---|
| "Function calls are always expensive." | Sometimes, but compilers often inline or optimize call overhead away. |
| "Iterators are always faster than indexing." | Either can be fastest depending on code shape; inspect optimized output and measure. |
| "Atomics make concurrent updates parallel." | They ensure ordering/atomicity; contended writes may serialize via coherency. |
| "Memory mapping is always faster than buffered I/O." | Depends on access patterns, page faults, and OS behavior. |
| "Greedy is fine for coin change." | Only for specific coin systems; not generally optimal. |
| "Switching HashMap hasher is free speed." | Can help trusted workloads, but may weaken collision-DoS resistance. |

---

## Profiling toolbox

Use tools by question type:

### Micro-bench and regression detection

- **Criterion**: compare small code variants with statistics.
- **`std::hint::black_box`**: reduce benchmark elimination artifacts.

Best for: tight function comparisons and CI benchmark baselines.

### CPU hotspots and call stacks

- **`perf`** (`perf stat`, `perf record`, `perf report`) on Linux.
- **cargo-flamegraph** for visualized stack samples.

Best for: "where is time spent?" and hardware-counter correlations.

### Allocation/memory profiling

- **heaptrack**: allocation hot spots/leaks.
- **Valgrind tools** (e.g., Massif) where supported.

Best for: heap churn, footprint growth, allocation-heavy latency.

### Causal profiling

- **Causal profilers** (e.g., Coz) estimate end-to-end impact of speeding a line/function.

Best for: avoiding work on hotspots that do not move wall-clock outcomes.

### Assembly inspection

- **cargo-asm**, **Compiler Explorer**, or `rustc --emit=asm`.

Best for: checking inlining, vectorization, bounds checks, branch shape.

### Platform-specific tools

- Linux: `perf`, eBPF tooling.
- macOS: Instruments.
- Windows: WPA/ETW, Visual Studio Profiler.
- CPU vendors: Intel VTune, AMD uProf, Arm Streamline (as applicable).

Always align tool choice with the performance question.

---

## Further reading (official docs first)

### Rust and Cargo

- Rust standard library docs: <https://doc.rust-lang.org/std/>
- Rust Reference: <https://doc.rust-lang.org/reference/>
- Cargo profiles: <https://doc.rust-lang.org/cargo/reference/profiles.html>
- Rust Performance Book: <https://nnethercote.github.io/perf-book/>
- Clippy lint list: <https://rust-lang.github.io/rust-clippy/stable/index.html>
- Criterion.rs book: <https://bheisler.github.io/criterion.rs/book/>

### Compiler and low-level tooling

- LLVM docs: <https://llvm.org/docs/>
- Linux perf wiki/docs: <https://perf.wiki.kernel.org/index.php/Main_Page>

### Concurrency runtimes

- Tokio docs: <https://docs.rs/tokio/latest/tokio/>
- Rayon docs: <https://docs.rs/rayon/latest/rayon/>

### Hardware/vendor references

- Intel optimization resources: <https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html>
- AMD developer guides: <https://www.amd.com/en/developer.html>
- Arm developer docs: <https://developer.arm.com/documentation>

---

## Practical guardrails summary

Before applying advanced techniques (`unsafe`, custom allocators, SIMD intrinsics, prefetching, huge pages, cache padding, PGO, architecture-specific flags):

1. Confirm bottleneck with profiling.
2. Document assumptions/invariants.
3. Add correctness tests around changed paths.
4. Measure end-to-end impact, not only micro-benchmarks.
5. Keep a maintainable fallback when feasible.
