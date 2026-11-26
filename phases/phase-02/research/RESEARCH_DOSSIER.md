# Phase 2 Research Dossier — Complexity and Performance Reasoning

> **Document Type:** Proposed Pre-Material Research & Planning Dossier (**not** final teaching materials — do not generate Phase 2 teaching blocks until ChatGPT reviews and approves this dossier)
> **Approval Status:** `Proposed — awaiting ChatGPT (Curriculum Owner & Instructor) review and approval`
> **Phase Authority:** ChatGPT (Curriculum Owner & Instructor) — **22 focused learner-study hours total** (per `MASTER_ROADMAP.md`; block targets are flexible planning estimates, not hard deadlines; mastery takes priority)
> **Advance-Planning & Continuity Notice:** Live learning continuity remains unchanged in `PROGRESS.md` and `NEXT_SESSION.md` (Active Phase: **Phase 1**, Block 1 studied, Block 2 next active study block; Phase 1 `not approved`). This dossier was prepared in advance at the learner's explicit request strictly for curriculum planning so Phase 2 materials can be cleanly authorized later without delaying morning study sessions. No Phase 2 teaching blocks, `README.md`, `notes/`, `lab/`, `checkpoint.md`, or progress updates have been created.
> **Knowledge-Depth Model:** Uses the repository's **exactly three official knowledge levels** (`Awareness`, `Working Understanding`, `Practical Competence` — per `DEC-002` and `MASTER_ROADMAP.md`), paired with Bloom's taxonomy cognitive levels for every block learning objective. Mechanism-level depth and production relevance are dimensions applied across these three levels, not additional levels.

---

## 1. Research Methodology & Verification Policy

### Selection Criteria
1. **Primary & Authoritative Sources First:** Prioritized recognized university courses and open textbooks (MIT OpenCourseWare 6.006 Spring 2020, Princeton University *Algorithms, 4th ed.* booksite, Pat Morin's *Open Data Structures*), foundational peer-reviewed research (Crosby & Wallach, USENIX Security 2003), primary systems-engineering practitioners (Brendan Gregg, Vyacheslav Egorov / `mraleph`, Colin Scott / Jeff Dean latency work, Cloudflare engineering postmortem), and official runtime documentation (Node.js `node:perf_hooks`).
2. **Free Accessibility First for Mandatory Resources:** Every **Mandatory** resource in this dossier is freely accessible online without a paywall. Commercial textbooks (CLRS 4th ed. 2022, Sedgewick & Wayne print 4th ed., Skiena 3rd ed.) and supplementary deep dives are classified strictly as **Supplementary** or **Reference**.
3. **Production Backend Reasoning Over Interview Tricks:** Selected concepts and readings directly build quantitative performance judgment for backend systems (Node.js + PostgreSQL specialization with ecosystem breadth) rather than competitive-programming or LeetCode interview drills.
4. **Curated Workload Fit:** Mandatory reading/viewing assignments are scoped strictly to targeted chapters, lectures, or subsections so the 7-block required core has an honest midpoint (`~20.25 h`) within the **22-hour** roadmap allocation, while transparently showing that its upper estimate (`22.5 h`) exceeds 22 hours by 30 minutes (`~20.5–25.6 h` if all 7 optional assessments are attempted), with remaining flexibility depending on where the learner falls within the estimated range; additional active analysis and experimentation should take priority over rushing through the material.
5. **Strict Non-Fabrication & Live Verification Policy:** All mandatory URLs, course titles, chapter/section headings, and lecture numbers were live-verified on `2026-11-26`. Details not directly inspectable (such as exact page numbers of commercial books or URLs blocked by automated bot interstitials) are transparently labeled `[Unverified]` or noted in Section 12. Broken or timed-out resources (e.g., `accidentallyquadratic.tumblr.com`, which timed out during verification) were excluded rather than cited speculatively.

---

## 2. Phase 2 Overview, Purpose, Prerequisites & Longitudinal Connections

### 2.1 Why Phase 2 Exists and What It Achieves
In `LEARNING_JOURNEY.md`, **Phase 2: Complexity and Performance Reasoning** completes **Arc I — Foundations of Engineering Reasoning (Phases 1–2)**.

- **Central Driving Question (`LEARNING_JOURNEY.md`):**
  > *"Why does code that feels instant at 100 items collapse at 1,000,000 items, and how do we predict bottlenecks before production fails?"*
- **Core Purpose:** Equip a backend engineer to reason quantitatively about **how computational time and memory costs grow as input volume increases**, compare data structures and algorithmic patterns honestly across time/space/ordering/worst-case trade-offs, recognize why real hardware (constant factors, memory locality, I/O latency) can overturn naive Big-O predictions, spot hidden quadratic/memory regressions in backend request handlers (including AI-generated code), and verify theoretical hypotheses through controlled empirical scaling experiments without falling for microbenchmark traps.
- **Why Phase 2 Is NOT a Generic DSA Interview Course:**
  This phase does **not** optimize for timed whiteboard puzzles, exotic dynamic programming contests, manual Red-Black/AVL tree rotation proofs, or graph-algorithm olympiads. Every mathematical tool and data structure earns its place by answering a concrete production backend question: *Will this endpoint survive 100× data growth? Where does memory or tail latency blow up? What is the true bottleneck—algorithmic growth, constant factors/locality, or external I/O?*

### 2.2 Why Phase 2 Follows Phase 1 (Inherited Knowledge)
Phase 1 (*Engineering Mindset & AI-Augmented Engineering*) established the qualitative judgment habits that Phase 2 now makes quantitative:
- **Block 1 (Scale & Time):** Introduced *scale* as a core software engineering axis and noted intuitively why linear/super-linear processes break under growth; Phase 2 formalizes growth curves mathematically.
- **Block 2 (Problem Framing & Bounds):** Trained asking about input bounds, payload limits, and latency requirements before coding; Phase 2 shows how to turn those bounds into time/memory cost predictions.
- **Block 3 (Correctness, Termination & ISO 25010):** Defined *Total Correctness* (termination/liveness) and *Performance Efficiency*; Phase 2 analyzes how non-trivial complexity turns technically terminating algorithms (`O(2^N)`, `O(N^2)`) into effective availability outages.
- **Block 4 (Constraints & Trade-offs):** Established Richards & Ford's First Law (*everything is a trade-off*) and multi-dimensional costs; Phase 2 applies that directly to **time-space trade-offs** and data-structure selection.
- **Block 5 (Evidence-Based Engineering):** Taught falsifiable hypotheses, baseline measurements, and Gregg's *Passive Benchmarking Anti-Method*; Phase 2 pairs theoretical complexity predictions with controlled Node.js empirical scaling measurements and *Active Benchmarking*.
- **Block 6 (Simplicity vs. Accidental Complexity):** Distinguished essential vs. accidental complexity; Phase 2 prevents premature micro-optimization while eliminating genuinely catastrophic accidental `O(N^2)` patterns.
- **Block 7 & Block 8 (AI Verification & Engineering Ownership):** Demonstrated that AI assistants routinely produce plausible, happy-path code that hides subtle bugs or inefficient scans (`O(N^2)` lookups inside loops) and that engineers must independently verify scaling claims and revisit triggers.

### 2.3 Prerequisite Knowledge Required Before Starting Phase 2
- **Phase 1 Mindset & Habits:** Problem framing, trade-off reasoning, falsifiable hypotheses, and basic Node.js script execution (`node`, `node --test`).
- **Foundational Programming Literacy:** Variables, arrays, objects/`Map`/`Set`, loops, conditionals, and basic function calls/recursion in JavaScript (already confirmed in Phase 1 Diagnostic).
- **Basic Algebra & Logarithms (Refreshed Inline in Block 1):** Exponents, base-2 logarithms (`log2 N` intuition: how many times you can halve `N` before reaching `1`), and simple arithmetic/geometric series sums (`1 + 2 + ... + N = N(N+1)/2` and `1 + 2 + 4 + ... + N = 2N - 1`). Any missing math refresher will be woven invisibly into Block 1 & Block 2 prose per `teaching-material-design/SKILL.md`—no separate math prerequisite gate is needed.

### 2.4 Downstream Phases That Depend on Phase 2
- **Phase 3 (Operating Systems):** Builds on Phase 2's conceptual memory-hierarchy intuition (cache vs. RAM vs. disk orders of magnitude) to explain virtual memory, page faults, CPU caches, and OS I/O mechanics.
- **Phase 4 (Computer Networking):** Builds on Phase 2's I/O-vs-CPU latency dominance math to ground RTT, handshakes, and throughput limits.
- **Phase 5 (JavaScript Language Internals):** Connects Phase 2's abstract array/map/object cost models and microbenchmark warmup/DCE caveats to real V8 heap allocation, GC, hidden classes, and JS runtime semantics.
- **Phase 6 (Concurrency & Asynchronous Systems):** Extends Phase 2's single-threaded cost model to concurrent contention, tail latency amplification, and event-loop blocking hazards.
- **Phase 10 (Database Engineering):** Directly relies on Phase 2's `O(N)` full scan vs. `O(log N)` ordered search vs. `O(1)` hash lookup and disk-I/O dominance to explain B-Tree indexes, hash indexes, and query execution plans.
- **Phase 14 (Database Integration & Data Access):** Directly operationalizes Phase 2's conceptual N+1 round-trip cost and batching trade-offs in PostgreSQL + Node.js.
- **Phase 16 (Caching Engineering):** Relies on Phase 2's latency hierarchy and time-space trade-offs to reason about cache hit-ratio economics.
- **Phase 19 (Backend Security):** Expands Phase 2's awareness of Algorithmic Complexity DoS (hash flooding, catastrophic regex backtracking / ReDoS) into adversarial threat modeling and production mitigations.
- **Phase 25 (Performance Engineering):** Deepens Phase 2's foundational benchmarking hygiene and scaling predictions into production CPU/memory profiling, flame graphs, queueing analysis, and load testing.

---

## 3. Explicit Curriculum Scope Boundaries & Topic Triage

### 3.1 Anti-Overlap Boundary Matrix (Phase 2 vs. Later Phases)

| Later Phase | What Belongs in Phase 2 (Allowed Here) | What Must Remain Deferred (Explicitly Excluded Here) |
| :--- | :--- | :--- |
| **Phase 3: Operating Systems** | Conceptual latency hierarchy (L1/L2 cache vs. RAM vs. SSD orders of magnitude) and intuitive **spatial locality** (why contiguous arrays outperform pointer chasing). | Virtual memory, paging, TLBs, OS process/thread scheduling, context switches, syscalls, kernel page cache internals. |
| **Phase 4: Computer Networking** | Order-of-magnitude network RTT intuition (why 500 sequential remote calls swamp in-memory CPU work). | TCP/UDP, DNS, TLS handshakes, sockets, framing, HTTP/2/3, packet loss, congestion control. |
| **Phase 5: JS Language Internals** | Using clean JavaScript (`Array`, `Map`, `Set`, loops, recursion) to analyze complexity and run scaling scripts; conceptual awareness that JIT warmup, DCE, and GC affect timing. | V8 hidden classes/inline caches, Ignition/Sparkplug/Maglev/TurboFan internals, GC generations, closures/prototypes, microtasks. |
| **Phase 6: Concurrency & Async** | Single-threaded sequential word-RAM analysis; brief awareness that an `O(N^2)` synchronous handler stalls other waiting requests. | Event-loop phases, threads/mutexes/semaphores, race conditions, deadlocks, async backpressure. |
| **Phase 10: Database Engineering** | Conceptual difference between `O(N)` sequential scan, `O(log N)` ordered search / balanced tree capability (range queries), and `O(1)` expected hash lookup. | B+Tree disk-page layouts, PostgreSQL MVCC, WAL, `EXPLAIN (ANALYZE, BUFFERS)`, SQL query planner internals, isolation levels. |
| **Phase 11: Node.js Runtime** | Using `performance.now()` (`node:perf_hooks`) or `process.hrtime.bigint()` strictly as a **Supporting** monotonic timer. | libuv thread pool, Buffer/Stream backpressure internals, event-loop lag monitoring, process lifecycle/signals. |
| **Phase 14: DB Integration** | Recognizing the conceptual `O(N)` round-trip scaling of N+1 remote lookups vs. batched `O(1)` round-trip + in-memory `Map` join. | `pg.Pool` sizing, SQL joins/CTEs, ORM query generation, connection leaks, schema migrations. |
| **Phase 19: Backend Security** | Understanding the abstract *complexity mechanism* behind hash-table collision degradation (`O(1)` expected → `O(N^2)` total across `N` colliding operations in vulnerable/non-randomized hash tables) and catastrophic regex backtracking (`O(2^N)` / `O(N^2)`) as distinct worst-case complexity risks. | Cryptographic SipHash internals, V8 hash-table internals (Phase 5), WAF tuning, OWASP security hardening, penetration testing, input sanitization frameworks. |
| **Phase 25: Performance Eng.** | Multi-`N` empirical scaling scripts, warmup & anti-DCE discipline, monotonic clocks, median/p95 awareness, Gregg's *Active Benchmarking* mindset. | Flame graphs, V8 CPU/heap profilers, load-testing suites (`k6`/`autocannon`), queueing theory, PostgreSQL query tuning, AI inference batching. |

### 3.2 Explicitly Excluded Competitive-Programming / Over-Scoped Topics
To prevent turning Phase 2 into an interview-prep cram course, the following are **excluded from Phase 2**:
- Formal proofs of the Master Theorem (Case 1–3 continuous proofs) or Akra-Bazzi theorem *(we teach intuitive recursion trees + `Awareness` of Master Theorem's existence)*.
- Manual AVL / Red-Black tree rotation mechanics and deletion rebalancing proofs *(we teach balanced search trees at `Working Understanding` of capabilities/costs and `Awareness` of height balancing)*.
- Advanced competitive-programming algorithms: multi-dimensional DP contests, graph shortest-path/flow olympiads (Bellman-Ford, Floyd-Warshall, Dinic's max-flow), KMP/Z-algorithm/suffix-automata string contests, segment trees / Fenwick trees.
- Formal potential-function amortized proofs for Fibonacci heaps or splay trees *(we teach Aggregate Analysis + Banker's/Accounting intuition on dynamic array doubling)*.

### 3.3 Candidate Topic Evaluation & Triage Table

| Candidate Topic | Classification | Why It Matters to a Backend Engineer | Target Depth | Prerequisites | Verification Method |
| :--- | :---: | :--- | :---: | :--- | :--- |
| What complexity measures (word-RAM abstraction) | **Essential** | Prevents confusing abstract op counts with exact ms runtime | `Working Understanding` | Basic JS | Explain model + where unit-cost breaks |
| Time & space complexity (input vs. auxiliary space; cumulative allocation vs. peak live memory) | **Essential** | Prevents hidden OOM crashes and separates quadratic copying churn from peak live memory | `Working Understanding` (Block 1) → `Practical Competence` (Blocks 2, 6) | Basic JS | Derive time, peak live aux space & cumulative allocation on code |
| Asymptotic notation (`O`, `Ω`, `Θ`) & limitations | **Essential** | Precise technical communication; avoids "Big-O = worst-case" myth | `Working Understanding` | Basic algebra/logs | Audit & correct misused notation claims |
| Common complexity classes & input growth | **Essential** | Predicts collapse from `N=100` to `N=1,000,000` | `Working Understanding` | Logs & exponents | Compute growth tables & crossover points |
| Constant factors & lower-order terms | **Essential** | Explains why `O(N)` beats `O(log N)` at small `N` or high constant | `Working Understanding` | Growth curves | Calculate crossover & defend algorithm choice |
| Loops, dependent nested loops, multi-variable (`N, M`) | **Essential** | Backend handlers join/filter two different collection sizes | `Practical Competence` | Block 1 | Trace & derive exact multi-variable bounds |
| Recursion, call-stack space & recursion trees | **Essential** | Predicts stack overflow & divide-and-conquer costs cleanly | `Working Understanding` | Loops & functions | Draw recursion tree + compute stack depth |
| Formal Master Theorem proofs | **Outside / Deferred** | Overly academic for Phase 2 (`Awareness` only) | `Awareness` | Recurrences | Know when recurrence trees suffice vs. theorem |
| Best, average/expected, worst-case analysis | **Essential** | Separates marketing happy-path from tail-latency/adversarial risk | `Working Understanding` | Block 1–2 | Diagnose worst-case triggers in real inputs |
| Amortized analysis (dynamic array doubling) | **Essential** | Explains why `Array.push` is `O(1)` amortized yet can still spike | `Working Understanding` | Series sums | Prove `+k` vs `×2` total cost via aggregate method |
| Algorithmic Complexity DoS (hash flooding / ReDoS) | **Essential (Mechanism)** | Shows worst-case complexity causes real production outages | `Working Understanding` (mechanism) / `Awareness` (defenses) | Hashing & worst-case | Trace how adversarial input triggers `O(N^2)`/`O(2^N)` |
| Core data structures (arrays, lists, hash tables, heaps, balanced trees conceptually) | **Essential** | Daily backend choice for lookup, dedup, ordering, Top-K | `Practical Competence` (arrays/hash/heaps) / `Working Understanding` (trees/lists) | Blocks 1–3 | Select & justify structure for concrete workloads |
| Searching, sorting & lookup patterns | **Essential** | Deciding sort-once + binary search vs. hash map vs. Top-K heap | `Working Understanding` | Block 4 structures | Compare total cost across `Q` queries & `N` items |
| Time-space trade-offs | **Essential** | Core backend lever: spend `O(N)` RAM to eliminate `O(N^2)` CPU | `Practical Competence` | Space complexity | Refactor quadratic code & quantify RAM cost |
| Hardware latency hierarchy & data locality | **Essential** | Explains why arrays beat linked lists & I/O swamps CPU micro-tweaks | `Working Understanding` | Block 1 & 4 | Estimate RAM/latency budget for an endpoint |
| Backend anti-patterns (scans, dedup, shift, spread, N+1 I/O) | **Essential** | Directly eliminates the #1 source of accidental backend slowdowns | `Practical Competence` | Blocks 1–5 | Audit, classify, and refactor realistic JS handlers |
| Pre-implementation resource estimation | **Essential** | Catches OOM and latency violations before writing code | `Practical Competence` | Block 5 | Back-of-the-envelope memory & ops sizing |
| Controlled Node.js measurement vs. theory & benchmark pitfalls | **Essential** | Connects Phase 1 Block 5 evidence to real scaling curves safely | `Practical Competence` | Blocks 1–6 + Phase 1 B5 | Design, run & interpret multi-`N` `perf_hooks` lab |
| V8 JIT/GC internals, OS paging, SQL `EXPLAIN`, flame graphs | **Outside / Deferred** | Dedicated to Phases 3, 5, 10, 11, 25 | — | — | Briefly referenced as `Supporting`/`Context` only |

---

## 4. Proposed Phase-Level Mastery Outcomes (8 Outcomes — For ChatGPT Approval)

*Note: `MASTER_ROADMAP.md` sets Phase 2's title and 22-hour allocation but does not yet list canonical numbered Phase 2 mastery outcomes. The 8 outcomes below are proposed to mirror Phase 1's 10 canonical outcomes and await ChatGPT's review and approval.*

By the end of Phase 2, the learner should be able to:

1. **Outcome 1 (`Working Understanding`):** Explain what computational complexity measures (abstract operation counts and auxiliary memory under the word-RAM model) and clearly separate theoretical asymptotic growth from wall-clock execution time.
2. **Outcome 2 (`Working Understanding`):** Use `O`, `Ω`, and `Θ` accurately across common complexity classes (`O(1)`, `O(log N)`, `O(N)`, `O(N log N)`, `O(N^2)`, `O(2^N)`), avoiding the errors of equating Big-O with "worst-case" or "exact execution speed".
3. **Outcome 3 (`Practical Competence`):** Derive time and auxiliary space complexity (including call-stack space) for realistic iterative code (including dependent nested loops, hidden library scans, and multi-variable inputs `N, M`) and foundational recursive/divide-and-conquer patterns via recursion trees.
4. **Outcome 4 (`Working Understanding`):** Distinguish **best-case**, **average-case / expected**, **worst-case**, and **amortized** complexity—explaining precisely why amortized ≠ average-case and how worst-case degradation (e.g., hash collisions, adversarial inputs) threatens backend reliability.
5. **Outcome 5 (`Practical Competence`):** Compare core data structures (dynamic arrays, linked lists, hash tables, sorted arrays / balanced search trees conceptually, binary heaps) and searching/sorting patterns to choose appropriate structures and justified time-space trade-offs for backend workloads.
6. **Outcome 6 (`Practical Competence`):** Diagnose and refactor hidden algorithmic and memory inefficiencies in realistic backend request-handling code (repeated scans, accidental `O(N^2)` joins/deduplication/array mutations, unbounded memory loads, AI-generated complexity traps, and conceptual N+1 I/O scaling).
7. **Outcome 7 (`Working Understanding`):** Reason quantitatively about constant factors, crossover points, memory locality, hardware latency orders of magnitude (cache vs. RAM vs. disk vs. network), and pre-implementation resource estimation to identify what truly dominates latency and memory.
8. **Outcome 8 (`Practical Competence`):** Design and execute controlled Node.js multi-`N` scaling experiments (`node:perf_hooks`) to test theoretical predictions empirically while identifying and avoiding microbenchmark pitfalls (DCE, JIT warmup, GC noise, unrepresentative inputs, and passive benchmarking).

---

## 5. Complete Block Sequence, Narrative Spine & Workload Summary (22 Hours)

### 5.1 Phase 2 Block Overview Table

*The authoritative Phase 2 allocation remains **22 focused learner-study hours (`1,320 min`)** per `MASTER_ROADMAP.md`. All per-block durations are honest, flexible planning estimates—not strict deadlines.*

| Block | Exact Proposed Title | Roadmap Target | Primary Target Level | Outcomes Supported | Honest Required Core (No Opt. Assess.) | Optional Assessment | Honest Total (With Opt. Assess.) |
| :---: | :--- | :---: | :--- | :---: | :---: | :---: | :---: |
| **1** | Computational Cost, Growth Rates, and Asymptotic Notation | 3.0 h | `Working Understanding` | 1, 2 | 140–175 min (2.3–2.9 h) | 20–25 min | 160–200 min (2.7–3.3 h) |
| **2** | Analyzing Loops, Recursion, and Recurrence Reasoning | 3.0 h | `Practical Competence` | 2, 3 | 150–185 min (2.5–3.1 h) | 20–25 min | 170–210 min (2.8–3.5 h) |
| **3** | Worst-Case, Average-Case, Amortized Analysis, and Algorithmic Risk | 3.0 h | `Working Understanding` | 4 | 155–190 min (2.6–3.2 h) | 20–25 min | 175–215 min (2.9–3.6 h) |
| **4** | Core Data Structures, Hashing, Searching, and Sorting Trade-Offs | 3.5 h | `Practical Competence` | 4, 5 | 175–215 min (2.9–3.6 h) | 25–30 min | 200–245 min (3.3–4.1 h) |
| **5** | Beyond Big-O: Constant Factors, Hardware Reality, and Data-Access Cost | 2.5 h | `Working Understanding` | 1, 7 | 115–150 min (1.9–2.5 h) | 20–25 min | 135–175 min (2.25–2.9 h) |
| **6** | Algorithmic Performance Reasoning in Backend Request Handling | 3.0 h | `Practical Competence` | 3, 5, 6, 7 | 135–175 min (2.25–2.9 h) | 20–25 min | 155–200 min (2.6–3.3 h) |
| **7** | Empirical Measurement vs. Theory, Benchmarking Pitfalls, and Phase 2 Capstone Lab | 4.0 h | `Practical Competence` | 1–8 (Integrative) | 210–260 min (3.5–4.3 h) | 25–30 min | 235–290 min (3.9–4.8 h) |
| **Total** | **7 Connected Phase 2 Blocks** | **22.0 h** | — | **All 8 Outcomes** | **1,080–1,350 min (18.0–22.5 h)** | **150–185 min (2.5–3.1 h)** | **1,230–1,535 min (20.5–25.6 h)** |

### 5.2 Causal Narrative Spine (Why Every Block Follows the Previous One)
1. **Block 1 → Block 2:** Block 1 gives us the mathematical language (`O`, `Ω`, `Θ`, word-RAM model, growth curves) to describe *how* cost scales—creating the immediate practical question: *"How do I reliably derive those bounds from real loops, multi-input joins, built-in array methods, and recursive code?"*
2. **Block 2 → Block 3:** Block 2 teaches counting operations in deterministic code—exposing an immediate puzzle: *"What if the same operation runs in `O(1)` on one input, `O(N)` on another, or spikes once every 1,024 appends when resizing?"*
3. **Block 3 → Block 4:** Block 3's dynamic-array doubling reveals that *memory layout and structural invariants* govern cost—motivating a systematic comparison of core backend data structures (arrays, lists, hash tables, balanced trees, heaps) and searching/sorting patterns.
4. **Block 4 → Block 5:** Block 4 equips us to select `O(1)` vs `O(log N)` vs `O(N)` structures—raising the production reality check: *"Why does an `O(N)` contiguous array sometimes crush an `O(N)` linked list or an `O(log N)` tree at small `N`, and why does one remote DB call overwhelm a million CPU ops?"*
5. **Block 5 → Block 6:** Equipped with both asymptotic analysis and hardware/I/O cost intuition, Block 6 applies them directly to real backend request handlers, batch pipelines, and AI-generated code to diagnose and refactor hidden performance traps.
6. **Block 6 → Block 7:** Block 6 produces theoretical refactorings and cost predictions—leaving the decisive Phase 1 Block 5 question: *"How do we empirically verify our complexity predictions in Node.js without being fooled by JIT warmup, dead-code elimination, GC noise, or passive benchmarking—and prove end-to-end mastery?"*

### 5.3 Workload Adequacy Assessment (Honest Alignment with the 22-Hour Allocation)
- **Verdict:** The approved **22-hour (`1,320 min`)** allocation in `MASTER_ROADMAP.md` remains unchanged. Across all 7 proposed blocks, the honest required-core estimate is **`18.0–22.5 hours` (`1,080–1,350 min`)**, with a midpoint of **`~20.25 hours` (`1,215 min`)**. While the midpoint fits inside 22 hours, **the upper estimate (`22.5 h`) exceeds the 22-hour allocation by 30 minutes**—so the curriculum fits near the ceiling of the allocation rather than with wide slack.
- **Transparency on Optional Assessments:** All 7 closed-book assessments remain strictly optional (`+2.5–3.1 h` total if all seven are attempted), yielding a combined range of **`20.5–25.6 hours`** (midpoint `~23.0 h`). That upper-end and optional-assessment overrun is reported honestly as actual Phase 2 study effort under flexible, mastery-first pacing rather than obscured behind program-level contingency assumptions. **No roadmap allocation change is authorized or requested.**
- **Alternative if ChatGPT Wants Even Tighter Pacing:** Moving Princeton 1.4's memory section in Block 5 from Mandatory to Supplementary would trim `~15–20 min`; however, retaining it directly strengthens pre-implementation memory estimation.

---

## 6. Detailed Block-by-Block Research Breakdown

---

### Block 1: Computational Cost, Growth Rates, and Asymptotic Notation *(Target: 3.0 hours)*

- **Purpose:** Build a rigorous mental model of what computational complexity actually measures (abstract operation counts and auxiliary memory under the word-RAM model) versus what it does *not* measure (exact wall-clock milliseconds), and establish accurate mastery of `O`, `Ω`, `Θ`, and common growth classes.
- **Dependencies:** Phase 1 Blocks 1, 2, 3, 5 (scale axis, verifiable bounds, termination, empirical reasoning).
- **Prepares For:** Phase 2 Block 2 (deriving bounds from code), Phase 2 Block 5 (where word-RAM & asymptotic bounds break down), Phase 10 (scan vs. index scaling).
- **Primary Target Level:** `Working Understanding` | **Phase 2 Outcomes Supported:** Outcomes 1, 2.

#### Learning Objectives
- **LO 1.1 (`Working Understanding` | Bloom: *Understand*):** Explain what **time complexity** and **space complexity** (input space vs. **auxiliary space**) actually measure under the **word-RAM model of computation**, and why unit-cost operation counts are distinct from wall-clock runtime.
- **LO 1.2 (`Working Understanding` | Bloom: *Analyze*):** Distinguish **Big-O (`O`)** (asymptotic upper bound), **Big-Omega (`Ω`)** (asymptotic lower bound), and **Big-Theta (`Θ`)** (asymptotic tight bound), correcting the pervasive misconception that "`O` means worst-case" or "`O` means exact execution time."
- **LO 1.3 (`Working Understanding` | Bloom: *Apply / Evaluate*):** Compare common complexity classes (`O(1)`, `O(log N)`, `O(N)`, `O(N log N)`, `O(N^2)`, `O(N^3)`, `O(2^N)`) quantitatively as input size `N` grows from `100` to `1,000,000`, explaining how growth rates dominate constant factors at large `N`—and when constant factors and lower-order terms still dominate at small `N`. *(Scoped intentionally to `Working Understanding` in Block 1 alongside Outcomes 1–2; hands-on code derivation advances to `Practical Competence` in Block 2.)*
- **LO 1.4 (`Working Understanding` | Bloom: *Evaluate*):** Identify the explicit **limitations of asymptotic analysis** (hidden constant factors, crossover regions, fixed-word/uniform-memory assumptions) so Big-O is never misused as a standalone runtime predictor.

#### Concepts Covered & Explicitly Deferred
- **Covered (Essential):** Word-RAM model (unit-cost primitive ops & memory cells); input size `N` (what `N` actually measures: records, bytes, bits); time vs. auxiliary space complexity; `O`, `Ω`, `Θ` formal intuition (`c > 0`, `n0`); growth-rate hierarchy (`1 < log N < sqrt(N) < N < N log N < N^2 < N^3 < 2^N < N!`); constant factors & lower-order terms; logarithm base-2 intuition (halving).
- **Covered (Supporting):** High-level mention that real computers have CPU caches, disks, and networks (unpacked in Block 5 and Phases 3–4).
- **Explicitly Deferred:** Formal epsilon-delta / calculus limit proofs (L'Hôpital's rule, Stirling's approximation); little-`o` / little-`ω` proofs; P vs. NP / NP-completeness theory; hardware cache line mechanics (Block 5 / Phase 3).

#### Terminology Inventory *(Internal Authoring Reference — Weave Progressively in Prose)*
| Term | Official Target Level | Why It Matters in Phase 2 |
| :--- | :---: | :--- |
| Word-RAM Model of Computation | `Working Understanding` | Makes explicit the idealized machine assumptions behind every complexity claim. |
| Time Complexity vs. Auxiliary Space | `Working Understanding` *(Block 1)* → `Practical Competence` *(Blocks 2, 6)* | Separates CPU operation growth from extra memory consumption beyond the input itself. |
| Asymptotic Notation (`O`, `Ω`, `Θ`) | `Working Understanding` | Distinguishes upper bound, lower bound, and tight bound—eliminating fuzzy "Big-O" misuse. |
| Order of Growth & Dominant Term | `Working Understanding` | Explains why `N^2` crushes `100 N log N` once data scales up. |
| Constant Factor & Crossover Point | `Working Understanding` | Prevents blindly assuming an asymptotically faster algorithm is faster at small `N`. |

#### Authoritative Resources & Assigned Scope
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | Pat Morin, *Open Data Structures (in pseudocode)* (Carleton University) | [1.1](https://opendatastructures.org/ods-python/1_1_Need_Efficiency.html), [1.3](https://opendatastructures.org/ods-python/1_3_Mathematical_Background.html), [1.4](https://opendatastructures.org/ods-python/1_4_Model_Computation.html), [1.5](https://opendatastructures.org/ods-python/1_5_Correctness_Time_Comple.html) | Sections `1.1` (*The Need for Efficiency*), `1.3.1` & `1.3.3` (*Exponentials/Logarithms* & *Asymptotic Notation* — skip `1.3.2`/`1.3.4`), `1.4` (*The Model of Computation*), and `1.5` (*Correctness, Time Complexity, and Space Complexity*) | ~25–30 min | Word-RAM assumptions, log intuition, Big-O basics, and distinction between correctness, time, and space. |
| **Mandatory 2** | Free | MIT OpenCourseWare 6.006 *Introduction to Algorithms* (Spring 2020 — Ku, Demaine, Solomon) | [Lecture 1: Introduction](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/mit6_006s20_lec1/) | Lecture 1 Notes PDF — sections on *Problems vs. Algorithms*, *Word-RAM Model*, and *Asymptotic Notation (`O`, `Ω`, `Θ`)* | ~20–25 min | Formal yet concise precision on `O` vs `Ω` vs `Θ` and how computational problems map to asymptotic bounds. |
| **Supplementary** | Commercial book | Thomas H. Cormen et al. (CLRS), *Introduction to Algorithms* (4th ed., MIT Press, 2022) | *(Print/commercial; ISBN 978-0262046305)* | Chapter 3 (*Characterizing Running Times* — `3.1 O, Ω, Θ-notation`, `3.2 Formal definitions`) *(4th ed. ToC verified; exact page numbers `[Unverified]`)* | Optional | Rigorous formal proofs of asymptotic properties for learners who own the book. |

#### Common Misconceptions & Accuracy Guardrails
- **Misconception:** *"Big-O always means worst-case runtime."* → **Correction:** `O(...)` means **asymptotic upper bound**, completely orthogonal to case analysis. An algorithm can have a best-case running time of `O(N)`, a worst-case running time of `O(N^2)`, or a worst-case running time of `Ω(N log N)`.
- **Misconception:** *"`O(N)` is always faster than `O(N^2)` in production."* → **Correction:** Only for sufficiently large `N` beyond the **crossover point**. `5000 * N` is slower than `0.5 * N^2` for all `N < 10,000`!
- **Misconception:** *"Space complexity only counts variables I explicitly name."* → **Correction:** Auxiliary space includes temporary arrays/maps/strings created inside helpers (e.g., `.slice()`, `.map()`, `.filter()`, `JSON.stringify()`) and recursion call-stack frames.

#### Practical Exercises & Assessment Approach
- **Exercise 1 (`LO 1.3`, `LO 1.4` — ~20–25 min): Scaling Explosion & Crossover Math (Online-Store Context).** Compare four candidate order-matching approaches (`500`, `40 * log2(N)`, `12 * N`, `0.2 * N^2` abstract operations) at `N = 50`, `500`, `5,000`, `100,000`, and `1,000,000`. Compute approximate operation counts, find the exact crossover point between `12 * N` and `0.2 * N^2`, and explain why testing only at `N = 50` misleads a team before a flash sale.
- **Exercise 2 (`LO 1.1`, `LO 1.2` — ~25–30 min): Technical Claim & Notation Audit.** Evaluate 6 realistic engineering statements from a PR/Slack thread (e.g., *"Big-O tells us our worst-case latency in milliseconds"*, *"Since `2N + 500` is `O(N^2)`, saying `O(N^2)` is technically false"*, *"Copying an array with `arr.map(...)` is `O(1)` space because we only used one variable"*). Classify each as True, Technically True but Loose/Misleading, or False, and rewrite each precisely using `O`/`Ω`/`Θ` and auxiliary space.
- **Assessment Approach:** 3 inline Pause & Checks (deferred Self-Check Guidance) + Optional Closed-Book Assessment (`~20–25 min`, 5 synthesis questions, strictly optional).

#### Study-Time Breakdown (Block 1)
- Guided Lesson + Pause & Checks 1–3: `~50–65 min` *(flexible priority)*
- Mandatory 1 (Open Data Structures `1.1, 1.3–1.5`): `~25–30 min`
- Mandatory 2 (MIT 6.006 Lecture 1 notes): `~20–25 min`
- Exercise 1: `~20–25 min` | Exercise 2: `~25–30 min`
- **Total Required Core (without optional assessment):** **`~140–175 min (~2.3–2.9 h)`**
- Optional Closed-Book Assessment: `~20–25 min` *(optional)*
- **Total (with optional assessment):** **`~160–200 min (~2.7–3.3 h)`** *(Roadmap target: `3.0 h`)*

---

### Block 2: Analyzing Loops, Recursion, and Recurrence Reasoning *(Target: 3.0 hours)*

- **Purpose:** Develop reliable hands-on skill (`Practical Competence`) in deriving exact time and auxiliary space complexity from real iterative and recursive backend code—especially multi-variable inputs, dependent nested loops, hidden library-call costs, and foundational divide-and-conquer recursion trees.
- **Dependencies:** Block 1 (`O`, `Ω`, `Θ`, auxiliary space, logarithm intuition).
- **Prepares For:** Block 3 (case-dependent loops & amortized resizing), Block 4 (merge sort / quicksort / tree / heap recurrences), Block 6 (auditing real backend request handlers).
- **Primary Target Level:** `Practical Competence` | **Phase 2 Outcomes Supported:** Outcomes 2, 3.

#### Learning Objectives
- **LO 2.1 (`Practical Competence` | Bloom: *Apply / Analyze*):** Derive tight time and auxiliary space bounds for sequential code, independent nested loops, **dependent triangular loops** (`j = i + 1` → `Θ(N^2)`), **multiplicative/halving loops** (`i *= 2` → `Θ(log N)`), and **multi-variable backend inputs** (`O(N + M)`, `O(N * M)`, `O(N log M)`).
- **LO 2.2 (`Practical Competence` | Bloom: *Analyze*):** Spot **hidden non-`O(1)` costs inside single-line loops** (such as `Array.includes`, `Array.indexOf`, `Array.splice`/`shift`, string concatenation/slicing, and array spread-copying `[...acc, item]` inside `.reduce()`) that silently turn linear-looking loops into `O(N^2)` or `O(N * L)` work—explicitly distinguishing **total cumulative allocation/copying** (`Θ(N^2)`) from **peak live auxiliary memory** (which depends on object lifetimes and runtime GC behavior) and **allocation churn**.
- **LO 2.3 (`Working Understanding` | Bloom: *Analyze*):** Trace **recursive algorithms** to determine both total work and **call-stack auxiliary space (`O(max recursion depth)`)**, identifying when unbounded recursion creates stack-overflow risks in backend trees/JSON payloads.
- **LO 2.4 (`Working Understanding` | Bloom: *Apply*):** Use **recursion trees** to solve the four canonical backend recurrence shapes—linear reduction `T(n) = T(n-1) + Θ(1)`, halving search `T(n) = T(n/2) + Θ(1)`, balanced split-and-merge `T(n) = 2T(n/2) + Θ(n)`, and branching `T(n) = 2T(n-1) + Θ(1)`—with `Awareness` of the Master Theorem's role without requiring formal Master Theorem proofs. *(Note: Block 2 intentionally mixes `Practical Competence` on iterative/hidden-cost code analysis with `Working Understanding` on recursion trees so the 3.0-hour target remains realistic.)*

#### Concepts Covered & Explicitly Deferred
- **Covered (Essential):** Operation counting in loops; arithmetic series sum (`N(N-1)/2`); geometric series sum (`1 + 1/2 + ...`); multi-variable analysis (`N` orders vs. `M` promos vs. `L` string length — never collapse two independent variables into one `N`!); hidden built-in costs in JS arrays/strings; **cumulative allocation/copying vs. peak live auxiliary memory vs. GC pressure** (why `[...acc, item]` in `.reduce()` performs `Θ(N^2)` cumulative element copies and allocation churn while live reachable auxiliary data at each step is `Θ(N)`, with actual peak RAM depending on object lifetimes and GC timing); recursion call-stack frames & space complexity; recursion-tree visual analysis (levels × work per level) for `Θ(log N)`, `Θ(N)`, `Θ(N log N)`, `Θ(2^N)`.
- **Covered (Context/Awareness):** Master Theorem as a closed-form shortcut for `T(n) = aT(n/b) + f(n)` (`Awareness` only).
- **Explicitly Deferred:** Substitution-method inductive proofs; continuous Master Theorem proofs; tail-call optimization compiler internals & V8 GC generation mechanics (Phase 5); dynamic programming memoization state-space design contests.

#### Terminology Inventory *(Internal Authoring Reference)*
| Term | Official Target Level | Why It Matters in Phase 2 |
| :--- | :---: | :--- |
| Multi-Variable Complexity (`N, M, K`) | `Practical Competence` | Real APIs join `N` orders against `M` rules; collapsing to `O(N^2)` hides which dimension actually grows. |
| Dependent Nested Loop (Triangular Sum) | `Practical Competence` | Pairwise checks (e.g., duplicate detection) still scale quadratically (`N(N-1)/2 = Θ(N^2)`). |
| Hidden Operation Cost | `Practical Competence` | One `.includes()` or `.shift()` inside a `for` loop multiplies complexity by `N`. |
| Cumulative Allocation vs. Peak Live Auxiliary Memory | `Working Understanding` | Prevents conflating `Θ(N^2)` cumulative copying/GC churn (e.g., `[...acc, item]` in `.reduce()`) with `Θ(N^2)` peak live memory. |
| Call-Stack Auxiliary Space | `Working Understanding` | Recursive category/JSON traversal can blow the stack even when heap usage is small. |
| Recursion Tree & Recurrence Relation | `Working Understanding` | Explains *why* binary search is `Θ(log N)` and merge sort is `Θ(N log N)` mechanically rather than by memorization. |
| Master Theorem | `Awareness` | Recognizes the standard theorem for divide-and-conquer recurrences without bogging Phase 2 down in proofs. |

#### Authoritative Resources & Assigned Scope
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | Robert Sedgewick & Kevin Wayne (Princeton), *Algorithms, 4th ed.* Booksite | [Section 1.4 — Analysis of Algorithms](https://algs4.cs.princeton.edu/14analysis/) | *Mathematical models* (tilde approximations, order-of-growth classifications) and *Designing faster algorithms* (3-Sum `N^3` → `N^2 log N`) *(defer Memory usage & Observations to Block 5)* | ~25–30 min | Concrete derivation from nested loops to `N^3`, `N^2`, `N^2 log N`, and order-of-growth tables. |
| **Mandatory 2** | Free | MIT OpenCourseWare 6.006 (Spring 2020) | [Recitation 1 Notes](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/mit6_006s20_r01/) & [Lecture 3: Sorting](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/mit6_006s20_lec3/) | Recitation 1 (asymptotic & loop analysis examples) + Lecture 3 **only** the Merge Sort recursion-tree & recurrence-solving excerpt (~2 pages) | ~25–30 min | Systematic loop counting and step-by-step recursion-tree analysis of `T(n) = 2T(n/2) + Θ(n)`. |
| **Supplementary** | Free | Jeff Erickson, *Algorithms* (UIUC) | [jeffe.cs.illinois.edu/teaching/algorithms/](https://jeffe.cs.illinois.edu/teaching/algorithms/) | Chapter 1 (*Recursion* — recursion trees overview) *(Note: site may present a browser check on automated fetch; exact subsection numbering `[Unverified]`)* | Optional | Clearer visual intuition for recursion trees if learner wants extra practice. |
| **Supplementary** | Commercial book | CLRS, *Introduction to Algorithms* (4th ed., 2022) | *(Print/commercial)* | Chapter 2 (`2.2 Analyzing algorithms`) & Chapter 4 (`4.4 The recursion-tree method`) *(4th ed. section titles verified; page numbers `[Unverified]`)* | Optional | Formal textbook treatment of loop cost sums and recursion trees. |

#### Practical Exercises & Assessment Approach
- **Exercise 1 (`LO 2.1`, `LO 2.2` — ~25–30 min): Multi-Variable & Hidden-Cost Code Audit.** Given 6 compact JavaScript backend snippets (including: matching `N` orders to `M` promo rules; pairwise fraud overlap check; halving binary-search loop; draining a queue with `while (q.length) q.shift()`; accumulating an array with `[...acc, row]` in `.reduce()`; filtering `N` items against an allowlist array of size `M`), derive tight time complexity (in exact variables `N, M`) and peak live auxiliary space complexity—explicitly marking the dominating line, and for the `[...acc, row]` snippet distinguishing **cumulative copying/allocation (`Θ(N^2)`)** from **live reachable auxiliary memory (`Θ(N)`)** and GC-dependent transient peak memory.
- **Exercise 2 (`LO 2.3`, `LO 2.4` — ~25–30 min): Recursion Tree & Stack-Space Analysis.** Given 3 recursive backend utilities (a recursive category-tree depth/price aggregator, a divide-and-conquer range splitter `2T(n/2) + O(n)`, and a naive recursive promo-combination generator `2T(n-1) + O(1)`), draw the recursion tree (levels, branching factor, work per level), derive total time complexity and maximum call-stack auxiliary space, and identify the exact input shape that triggers stack overflow or exponential freeze.
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~20–25 min`).

#### Study-Time Breakdown (Block 2)
- Guided Lesson + Pause & Checks 1–3: `~50–65 min` *(flexible priority)*
- Mandatory 1 (Princeton 1.4 selected sections): `~25–30 min`
- Mandatory 2 (MIT 6.006 Recitation 1 + Lecture 3 recurrence excerpt): `~25–30 min`
- Exercise 1: `~25–30 min` | Exercise 2: `~25–30 min`
- **Total Required Core (without optional assessment):** **`~150–185 min (~2.5–3.1 h)`**
- Optional Closed-Book Assessment: `~20–25 min` *(optional)*
- **Total (with optional assessment):** **`~170–210 min (~2.8–3.5 h)`** *(Roadmap target: `3.0 h`)*

---

### Block 3: Worst-Case, Average-Case, Amortized Analysis, and Algorithmic Risk *(Target: 3.0 hours)*

- **Purpose:** Resolve the critical distinction between best-case, average/expected-case, worst-case, and amortized costs—preventing one of the most common engineering reasoning errors (**confusing amortized with average-case**) and demonstrating how worst-case complexity triggers real backend tail-latency spikes and Algorithmic Complexity DoS outages.
- **Dependencies:** Blocks 1–2 (asymptotic bounds, geometric series intuition).
- **Prepares For:** Block 4 (hash table resizing/collisions, quicksort worst-case vs. expected), Block 7 (interpreting p99/tail spikes during empirical benchmarking), Phase 19 (ReDoS & hash-flooding security defenses), Phase 21 (tail-latency reliability).
- **Primary Target Level:** `Working Understanding` | **Phase 2 Outcomes Supported:** Outcome 4.

#### Learning Objectives
- **LO 3.1 (`Working Understanding` | Bloom: *Analyze*):** Contrast **best-case**, **average-case** (distributional assumption over inputs), **expected-case** (internal algorithm randomization), and **worst-case** complexity, explaining why backend SLA/SLO guarantees rely primarily on worst-case or high-probability expected bounds rather than best-case or naive average assumptions.
- **LO 3.2 (`Working Understanding` | Bloom: *Apply / Analyze*):** Apply the **Aggregate Method** (supported by intuitive accounting/banker's intuition) to prove why **dynamic array doubling** achieves **`O(1)` amortized** append cost over any sequence of `N` appends, whereas fixed-increment resizing (`+k`) degrades to **`Θ(N^2)` total / `Θ(N)` amortized**.
- **LO 3.3 (`Working Understanding` | Bloom: *Evaluate*):** Explain precisely why **Amortized ≠ Average-Case**—and why an **`O(1)` amortized** operation can *still* cause an individual `O(N)` tail-latency spike on a live latency-sensitive request!
- **LO 3.4 (`Working Understanding` | Bloom: *Analyze*):** Explain the abstract complexity mechanism of **Algorithmic Complexity Attacks**—distinguishing (a) worst-case hash-table collision degradation (`O(1)` expected → `O(N^2)` total across `N` colliding operations, historically demonstrated on specific non-randomized hash-table implementations by Crosby & Wallach, USENIX Security 2003—explicitly separated from modern native JavaScript `Map`/`Set` runtime implementations, whose V8 hash internals remain deferred to Phase 5), and (b) **catastrophic regex backtracking (ReDoS)** as a separate, concrete algorithmic-complexity availability risk (`Awareness` of production defenses, deferred to Phase 19).

#### Concepts Covered & Explicitly Deferred
- **Covered (Essential):** Input-distribution dependence; best vs. average vs. expected (randomized) vs. worst-case; dynamic array memory allocation & resizing (`2×` doubling vs. `+k` linear growth); Aggregate Method amortized proof (`N + (1 + 2 + 4 + ... + N) < 3N → Θ(1)` amortized); Banker's/Accounting intuition; tail-latency implication of amortized spikes; abstract hash-table collision degradation mechanism (kept in historical context via Crosby & Wallach 2003 without implying native JavaScript `Map`/`Set` shares those historical vulnerabilities); **catastrophic regex backtracking (ReDoS)** as a distinct concrete worst-case blowup risk.
- **Covered (Awareness):** Seeded/randomized hashing (e.g., SipHash concept) and linear-time regex matching at high level as mitigation strategies (V8 hash internals deferred to Phase 5; production security hardening deferred to Phase 19).
- **Explicitly Deferred:** Formal Potential Method proofs (`Φ(D_i)`); Fibonacci heaps / splay trees; V8 `Map`/`Set` internal hashing implementation (Phase 5); cryptographic hash-function math & WAF rules (Phase 19); garbage-collection pause mechanics (Phases 5 & 11).

#### Terminology Inventory *(Internal Authoring Reference)*
| Term | Official Target Level | Why It Matters in Phase 2 |
| :--- | :---: | :--- |
| Best-Case vs. Average-Case vs. Worst-Case | `Working Understanding` | Prevents trusting happy-path speed when skewed or adversarial inputs arrive in production. |
| Expected Time (Randomized) vs. Distributional Average | `Working Understanding` | Separates algorithms that *control* randomness internally from fragile assumptions about user inputs. |
| Amortized Analysis (Aggregate Method) | `Working Understanding` | Deterministic worst-case average per operation across *any* sequence—no probability assumptions needed. |
| Amortized Spike / Tail-Latency Hazard | `Working Understanding` | Explains why `O(1)` amortized does *not* guarantee every single HTTP request finishes in `O(1)`. |
| Algorithmic Complexity Attack (Hash Collisions & ReDoS) | `Working Understanding` (mechanism) / `Awareness` (mitigations) | Shows that worst-case complexity is a real operational risk—separating abstract hash-collision mechanics (Crosby & Wallach 2003) from modern JS `Map`/`Set` internals, and treating ReDoS as a separate concrete risk. |

#### Authoritative Resources & Assigned Scope
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | MIT OpenCourseWare 6.006 (Spring 2020) | [Lecture 2: Data Structures](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/mit6_006s20_lec2/) | Lecture 2 Notes PDF — *Sequence Interface*, *Static vs. Dynamic Arrays*, and *Amortized Analysis* of table doubling | ~25 min | Exact mechanism of why doubling yields `O(1)` amortized appends and how dynamic arrays compare to static arrays/linked lists. |
| **Mandatory 2** | Free | Pat Morin, *Open Data Structures* | [Section 2.1 — ArrayStack](https://opendatastructures.org/ods-python/2_1_ArrayStack_Fast_Stack_O.html) | Subsection `2.1.2 Growing and Shrinking` and `2.1.3 Summary` (shrinking hysteresis: why shrinking at `n/2` causes thrashing vs. shrinking at `n/3` or `n/4`!) | ~15–20 min | Crucial resizing-thrashing failure mode (grow at `N`, shrink at `N/2` → alternating append/pop triggers `O(N)` every op!). |
| **Mandatory 3** | Free | Scott A. Crosby & Dan S. Wallach (Rice Univ., USENIX Security 2003) | [Denial of Service via Algorithmic Complexity Attacks](https://www.usenix.org/conference/12th-usenix-security-symposium/denial-service-algorithmic-complexity-attacks) | Abstract, Section 1 (*Introduction*), and conceptual overview of the Hash-Table collision attack *(skip Perl/Squid/Bro C-code exploit internals)* | ~15–20 min | Historical empirical proof on specific 2003 non-randomized hash-table implementations illustrating the abstract worst-case `O(N^2)` collision mechanism—contextualized rather than treated as proof of vulnerability in modern native JS `Map`/`Set`. |
| **Supplementary** | Commercial book | CLRS, *Introduction to Algorithms* (4th ed., 2022) | *(Print/commercial)* | Chapter 17 (*Amortized Analysis* — `17.1 Aggregate analysis`, `17.2 Accounting method`, `17.4 Dynamic tables`) *(4th ed. titles verified; pages `[Unverified]`)* | Optional | Formal multi-method treatment of amortized analysis and dynamic tables. |

#### Practical Exercises & Assessment Approach
- **Exercise 1 (`LO 3.2`, `LO 3.3` — ~25–30 min): Dynamic Buffer Resizing Analysis.** A Node.js batch ingestion buffer starts at capacity `10` and must accept `N = 100,000` events. Compare three resizing policies mathematically using the Aggregate Method: (a) grow by `+100` slots each time, (b) grow by `×2` each time, and (c) grow by `×2` when full AND shrink by `÷2` immediately when half full under an alternating push/pop workload. Quantify total element copies, amortized per-op cost, and peak single-operation latency spike for each.
- **Exercise 2 (`LO 3.1`, `LO 3.3`, `LO 3.4` — ~25–30 min): Case-Analysis & Production Risk Audit.** Given 4 backend scenarios: (a) a naive custom or historical non-randomized hash table keyed by untrusted HTTP query parameters (explicitly distinguishing the abstract collision mechanism from modern runtime-managed JS `Map`/`Set` implementations), (b) an in-memory append buffer doubling at `1,048,576` items during a latency-critical checkout request, (c) deterministic quicksort choosing the first element as pivot on timestamp-sorted order exports, and (d) claiming *"our operation is `O(1)` amortized, which means on average random inputs take `O(1)`"*. Diagnose the exact case-analysis error and explain the concrete production failure mode for each.
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~20–25 min`).

#### Study-Time Breakdown (Block 3)
- Guided Lesson + Pause & Checks 1–3: `~50–65 min` *(flexible priority)*
- Mandatory 1 (MIT 6.006 Lecture 2 notes): `~25 min`
- Mandatory 2 (Open Data Structures `2.1.2–2.1.3`): `~15–20 min`
- Mandatory 3 (Crosby & Wallach USENIX '03 targeted): `~15–20 min`
- Exercise 1: `~25–30 min` | Exercise 2: `~25–30 min`
- **Total Required Core (without optional assessment):** **`~155–190 min (~2.6–3.2 h)`**
- Optional Closed-Book Assessment: `~20–25 min` *(optional)*
- **Total (with optional assessment):** **`~175–215 min (~2.9–3.6 h)`** *(Roadmap target: `3.0 h`)*

---

### Block 4: Core Data Structures, Hashing, Searching, and Sorting Trade-Offs *(Target: 3.5 hours)*

- **Purpose:** Connect abstract complexity directly to the mechanics of core backend data structures and fundamental searching/sorting patterns so the learner can select structures by workload (exact lookup vs. ordered range vs. Top-K vs. append/iterate) and engineer deliberate **time-space trade-offs**.
- **Dependencies:** Blocks 1–3 (asymptotic bounds, recursion trees, expected vs. worst-case vs. amortized cost).
- **Prepares For:** Block 5 (cache locality of arrays vs. linked lists & open addressing vs. chaining), Block 6 (replacing linear scans with hash maps/heaps in backend handlers), Phase 10 (PostgreSQL B-Tree vs. Hash indexes).
- **Primary Target Level:** `Practical Competence` | **Phase 2 Outcomes Supported:** Outcomes 4, 5.

#### Learning Objectives
- **LO 4.1 (`Practical Competence` | Bloom: *Analyze / Evaluate*):** Compare exact operation costs (index access, search, append, front/middle insert/delete, memory overhead) of **contiguous dynamic arrays** vs. **linked lists**—explicitly qualifying that `O(1)` linked-list insertion/deletion applies **only when the relevant node or insertion position is already known** (locating that position still requires `O(N)` traversal)—and explain why contiguous arrays dominate most backend sequential workloads despite identical `O(N)` scans.
- **LO 4.2 (`Practical Competence` | Bloom: *Apply / Analyze*):** Explain abstract **hash-table** mechanics (hash function, bucket array, **load factor `α = n/m`**, collision resolution via **separate chaining** vs. **open addressing / linear probing**, dynamic rehashing) and apply hash-based `Map`/`Set` structures appropriately to exact-key lookup and deduplication workloads—explicitly framing **expected `O(1)`** as the **conventional hash-table cost model** (conditional on hash distribution and bounded load factor, not an unconditional runtime guarantee), separating abstract collision mechanics from modern JS runtime implementation details (V8 internals deferred to Phase 5), and recognizing memory overhead and inability to serve ordered range queries efficiently.
- **LO 4.3 (`Working Understanding` | Bloom: *Analyze*):** Contrast **unsorted scan (`O(N)`)**, **sorted array + binary search (`O(log N)` search, `O(N)` insert)**, **balanced search trees (`O(log N)` search/insert/delete + ordered range/min/max/prefix queries — conceptual capability without manual rotation proofs)**, and **binary heaps / priority queues (`O(log N)` push/pop-min, `O(1)` peek, `O(N log K)` Top-K)**—establishing the conceptual bridge to Phase 10 database indexes.
- **LO 4.4 (`Working Understanding` | Bloom: *Evaluate*):** Compare foundational **sorting trade-offs**—elementary `O(N^2)` sorts (insertion sort's strength on tiny or nearly sorted data), `O(N log N)` comparison sorts (merge sort deterministic worst-case + stability + `O(N)` space vs. quicksort in-place expected `O(N log N)` / `O(N^2)` worst-case + randomization; conceptual `Ω(N log N)` comparison lower bound), and `Awareness` of linear-time counting/radix sort for bounded integer keys—and apply **time-space trade-offs** cleanly. *(Note: Block 4 intentionally mixes `Practical Competence` on daily backend data-structure selection/application with `Working Understanding` on ordered trees and sorting trade-offs within its 3.5-hour target.)*

#### Concepts Covered & Explicitly Deferred
- **Covered (Essential):** Dynamic arrays vs. singly/doubly linked lists (qualifying `O(1)` insert/delete as conditional on already holding the target node/position reference, vs. `O(N)` traversal to find it); abstract hash tables (conventional expected `O(1)` cost model—not an unconditional runtime guarantee—load factor, chaining vs. linear probing, resizing, collision worst-case, and explicit separation from V8 `Map`/`Set` internals); binary search precondition (sorted random-access array); balanced BST capability/cost invariants (`Working Understanding` of *what* balance buys and *why* unbalanced BSTs degrade to `O(N)` linked lists; `Awareness` of AVL/Red-Black/B-Tree balancing); binary heap implicit array layout & Top-K pattern (`O(N log K)` vs. `O(N log N)` full sort); comparison sorting trade-offs (stability, in-place vs. extra memory, nearly-sorted behavior, `Ω(N log N)` lower bound); time-space trade-offs (indexing/hash maps, precomputation, caching/memoization).
- **Covered (Awareness):** Non-comparison linear sorting (counting sort / radix sort when key range `k` is bounded); Timsort/hybrid sorts (combining merge/insertion sort in real runtimes like V8/Python).
- **Explicitly Deferred:** Implementing AVL/Red-Black tree rotations from scratch; B+Tree disk-block splitting & PostgreSQL index pages (Phase 10); graph algorithms (BFS/DFS/shortest paths); string tries/suffix trees; V8 `Map`/`Set` internal hash-table implementation & plain-Object hidden classes (Phase 5).

#### Terminology Inventory *(Internal Authoring Reference)*
| Term | Official Target Level | Why It Matters in Phase 2 |
| :--- | :---: | :--- |
| Contiguous Dynamic Array vs. Linked List | `Practical Competence` | Trade-off between `O(1)` indexing/locality (arrays) vs. `O(1)` insert/delete *only when the node/position is already known* (linked lists—locating a position still costs `O(N)` traversal). |
| Hash Table, Load Factor & Collisions | `Practical Competence` | Conventional expected `O(1)` cost model (not an unconditional guarantee) behind `Map`/`Set`, deduplication, joins, caches, and hash indexes—distinct from runtime-specific V8 internals. |
| Separate Chaining vs. Linear Probing | `Working Understanding` | Contrasts pointer-list buckets against cache-friendly contiguous probing & clustering risk. |
| Binary Search & Ordered Search Trees | `Working Understanding` | Explains the structural price and payoff of maintaining sorted order (`O(log N)` range queries). |
| Binary Heap / Priority Queue (Top-K) | `Practical Competence` | Avoids sorting `N` items (`O(N log N)`) when only the top `K` or next-due job is needed (`O(N log K)`). |
| Sort Stability & Comparison Lower Bound (`Ω(N log N)`) | `Working Understanding` | Explains why no general comparison sort beats `N log N` and why stability matters for multi-key sorts. |
| Time-Space Trade-Off | `Practical Competence` | Consciously exchanging `O(N)` auxiliary memory to collapse `O(N^2)` CPU work into `O(N)`. |

#### Authoritative Resources & Assigned Scope
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | MIT OpenCourseWare 6.006 (Spring 2020) | [Lecture 4: Hashing](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/mit6_006s20_lec4/) | Lecture 4 Notes PDF — *Direct Access Array*, *Hashing*, *Chaining*, *Universal/Good Hashing*, *Load Factor & Expected `O(1)`* | ~20–25 min | Why direct-access arrays waste huge space and how hashing + resizing achieves expected `O(1)` in `O(N)` space. |
| **Mandatory 2** | Free | Robert Sedgewick & Kevin Wayne (Princeton), *Algorithms, 4th ed.* Booksite | [3.1 Elementary Symbol Tables](https://algs4.cs.princeton.edu/31elementary/) & [3.4 Hash Tables](https://algs4.cs.princeton.edu/34hash/) | Section `3.1` (*Sequential search* vs. *Binary search in an ordered array* & cost comparison summary) + Section `3.4` (*Chaining* vs. *Linear probing* conceptual trade-offs) | ~25–30 min | Clear cost table contrasting sequential search, binary search, and hash tables, plus why ordered operations separate trees from hashes. |
| **Mandatory 3** | Free | Pat Morin, *Open Data Structures* | [10.1 BinaryHeap](https://opendatastructures.org/ods-python/10_1_BinaryHeap_Implicit_Bi.html) & [11.1 Comparison-Based Sorting](https://opendatastructures.org/ods-python/11_1_Comparison_Based_Sorti.html) | Section `10.1` (implicit array heap layout & `add`/`remove` `O(log N)` costs) + Section `11.1` (Merge-Sort, Quicksort, & `Ω(N log N)` lower-bound intuition — skip formal proofs) | ~20–25 min | Why heaps live inside flat arrays without pointers, and the exact trade-offs among `O(N log N)` sorting algorithms. |
| **Supplementary** | Free | Princeton *Algorithms, 4th ed.* Booksite | [3.3 Balanced Search Trees](https://algs4.cs.princeton.edu/33balanced/) | Conceptual overview of why 2-3 / balanced trees guarantee `O(log N)` height *(skip Red-Black Java rotation code)* | Optional | Deeper intuition on balanced search trees before Phase 10 B-Trees. |
| **Supplementary** | Commercial book | CLRS (4th ed., 2022) | *(Print/commercial)* | Chapter 11 (*Hash Tables* — `11.1–11.4`, noting 4th ed.'s updated linear-probing cache emphasis) *(4th ed. titles verified; pages `[Unverified]`)* | Optional | Deep theoretical treatment of hashing and open addressing. |

#### Practical Exercises & Assessment Approach
- **Exercise 1 (`LO 4.1`, `LO 4.2`, `LO 4.3` — ~25–30 min): Backend Workload → Data Structure Decision Matrix.** Given 5 concrete online-store backend workloads — (1) exact lookup of active promo by `code` (1M lookups/min, rare updates), (2) finding all orders created between timestamp `T1` and `T2` sorted chronologically, (3) deduplicating 200,000 incoming webhook `eventId`s in memory, (4) maintaining the Top-10 highest-spending customers out of 2,000,000 streaming order records without holding all 2M sorted in memory, and (5) maintaining a FIFO job queue where jobs are enqueued at the tail and dequeued at the head — select the best structure (dynamic array, circular queue/linked list, hash table, sorted array/balanced search tree, min-heap), justify time & space complexity, and name the exact failure mode if the wrong structure (e.g., hash table for range query, or plain array `.shift()` for FIFO queue) is chosen.
- **Exercise 2 (`LO 4.3`, `LO 4.4` — ~25–30 min): Search, Sort, and Time-Space Trade-Off Analysis.** Compare three strategies for serving `Q` price-lookup queries over an unsorted catalog of `N` products: (A) linear scan per query `O(Q * N)`, (B) sort once + binary search `O(N log N + Q log N)`, and (C) build a hash map once + exact lookup `O(N + Q)` (conventional expected cost model) with `O(N)` extra space. Compare the **asymptotic and abstract operation-count regimes** of `(N, Q)` where each strategy dominates—explicitly noting why exact real-world crossover points cannot be determined from asymptotic expressions alone without explicit constants and a defined cost model—explain what changes if the query shifts from exact `productId` lookup to *"all products priced between USD 10 and USD 25"*, and explain why insertion sort is still used inside modern hybrid sorts for tiny partitions.
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~25–30 min`).

#### Study-Time Breakdown (Block 4)
- Guided Lesson + Pause & Checks 1–3: `~60–75 min` *(flexible priority)*
- Mandatory 1 (MIT 6.006 Lecture 4 Hashing): `~20–25 min`
- Mandatory 2 (Princeton `3.1` & `3.4` targeted): `~25–30 min`
- Mandatory 3 (Open Data Structures `10.1` & `11.1` targeted): `~20–25 min`
- Exercise 1: `~25–30 min` | Exercise 2: `~25–30 min`
- **Total Required Core (without optional assessment):** **`~175–215 min (~2.9–3.6 h)`**
- Optional Closed-Book Assessment: `~25–30 min` *(optional)*
- **Total (with optional assessment):** **`~200–245 min (~3.3–4.1 h)`** *(Roadmap target: `3.5 h`)*

---

### Block 5: Beyond Big-O — Constant Factors, Hardware Reality, and Data-Access Cost *(Target: 2.5 hours)*

- **Purpose:** Break the naive textbook illusion that "Big-O tells the whole performance story." Build durable **data-access cost intuition** (CPU cache vs. RAM vs. SSD vs. network RTT), explain **spatial locality** and **crossover points**, and train practical **pre-implementation back-of-the-envelope resource estimation** (memory and latency budgets) before writing code.
- **Dependencies:** Blocks 1, 4 (word-RAM assumptions, array vs. linked list layouts, time-space trade-offs).
- **Prepares For:** Block 6 (spotting I/O dominance & memory blowups in backend handlers), Block 7 (interpreting measured curves vs. theory), Phase 3 (OS caches/virtual memory), Phase 4 (network RTT), Phase 10 (disk vs. RAM DB scans), Phase 16 (caching payoff economics), Phase 25 (performance engineering).
- **Primary Target Level:** `Working Understanding` | **Phase 2 Outcomes Supported:** Outcomes 1, 7.

#### Learning Objectives
- **LO 5.1 (`Working Understanding` | Bloom: *Evaluate*):** Explain the four core reasons **theoretical Big-O diverges from practical runtime**: hidden constant factors, non-dominant terms at practical `N`, non-uniform memory access (**memory hierarchy & spatial locality**), and runtime overhead (allocation, serialization, garbage collection).
- **LO 5.2 (`Working Understanding` | Bloom: *Analyze*):** Use order-of-magnitude **hardware latency numbers** (L1/L2 cache ~ns vs. main RAM ~100 ns vs. SSD ~10–100 µs vs. datacenter/cross-region network RTT ~0.5–100 ms) to explain why contiguous array scans outperform pointer-chasing linked lists at the same `Θ(N)` and why **external I/O / network round-trips dominate CPU micro-optimizations** in typical backend services.
- **LO 5.3 (`Practical Competence` | Bloom: *Apply / Evaluate*):** Perform lightweight **back-of-the-envelope resource estimation** (estimating working-set memory footprint under explicit illustrative object-overhead assumptions, operation counts, and I/O round-trip latency budgets) before implementation to catch gross OOM or SLA violations early. *(Note: Intentionally elevated to `Practical Competence` under Block 5's `Working Understanding` primary target and scoped specifically to rapid back-of-the-envelope arithmetic in Exercise 2 so it fits cleanly inside the 2.5-hour block without requiring profiling tools.)*
- **LO 5.4 (`Working Understanding` | Bloom: *Analyze*):** Distinguish systematically between **algorithmic inefficiency** (bad scaling curve), **implementation/environmental overhead** (constant factors, locality, allocation churn), and **external I/O bottlenecks** when reasoning about slow backend behavior.

#### Concepts Covered & Explicitly Deferred
- **Covered (Essential):** Limitations of the word-RAM uniform-cost assumption; constant factors & practical crossover points; conceptual memory hierarchy (L1/L2 cache → RAM → SSD → datacenter network → cross-region internet) and orders of magnitude (`10^2×` to `10^8×` gaps); **spatial locality** (contiguous arrays vs. scattered pointer-linked nodes); I/O-bound vs. CPU-bound dominance intuition; back-of-the-envelope memory & latency budgeting (record size × count × explicit illustrative overhead assumption, recognizing that actual object/`Map` overhead varies by runtime representation and workload; `QPS × ops/req`).
- **Explicitly Deferred:** CPU cache associativity/MESI coherence protocols & virtual memory page tables (Phase 3); TCP slow-start/BDP (Phase 4); exact V8 object header byte layouts / pointer compression (Phase 5); PostgreSQL `shared_buffers` & sequential-vs-random disk planner cost constants (Phase 10); Little's Law & queueing theory (Phase 25).

#### Terminology Inventory *(Internal Authoring Reference)*
| Term | Official Target Level | Why It Matters in Phase 2 |
| :--- | :---: | :--- |
| Memory Hierarchy & Latency Orders of Magnitude | `Working Understanding` | Anchors intuition: 1 network hop (~1 ms) ≈ 10,000 RAM reads ≈ 1,000,000 L1 cache hits. |
| Spatial Locality (Contiguous vs. Pointer Chasing) | `Working Understanding` | Explains why two `Θ(N)` structures can differ by 5–20× in real execution speed. |
| CPU-Bound vs. I/O-Bound Dominance | `Working Understanding` | Stops engineers from spending days saving 0.2 ms of CPU while ignoring 800 ms of sequential I/O. |
| Back-of-the-Envelope Resource Estimation | `Practical Competence` | Predicts memory exhaustion and latency budget overruns before writing a single line of code. |

#### Authoritative Resources & Assigned Scope
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | Colin Scott (UC Berkeley, evolving Jeff Dean's LADIS 2009 / Norvig numbers) | [Latency Numbers Every Programmer Should Know (Interactive)](https://colin-scott.github.io/personal_website/research/interactive_latency.html) | Explore current-decade order-of-magnitude latency gaps: L1/L2 cache, main RAM, SSD random read, datacenter packet round-trip, cross-region RTT | ~10–15 min | Visceral order-of-magnitude scale intuition (ns vs. µs vs. ms) for data-access cost reasoning. |
| **Mandatory 2** | Free | Robert Sedgewick & Kevin Wayne (Princeton), *Algorithms, 4th ed.* Booksite | [Section 1.4 — Analysis of Algorithms](https://algs4.cs.princeton.edu/14analysis/) | *Observations*, *Coping with dependence on inputs*, and *Memory usage* subsections (how object/array/reference memory footprints add up) | ~15–20 min | Concrete mental model for estimating memory usage (primitive values vs. object/pointer overhead) and noticing real-world runtime factors beyond Big-O. |
| **Supplementary** | Free | Peter Norvig | [Teach Yourself Programming in Ten Years — Appendix](https://norvig.com/21-days.html) | Appendix table *"Approximate timing for various operations on a typical PC"* *(Note: site presented automated interstitial on raw HTTP fetch; verify in browser)* | Optional | Classic original companion reference alongside Colin Scott's interactive version. |
| **Supplementary** | Commercial book | Sedgewick & Wayne, *Algorithms* (4th ed., print, 2011) | *(Print/commercial)* | Section 1.4 (*Caveats* — large constants, non-dominant terms, system effects, caching/GC) *(Print subsection noted; exact page numbers `[Unverified]`)* | Optional | Print expansion of the caveats summarized directly in our guided lesson. |

#### Practical Exercises & Assessment Approach
- **Exercise 1 (`LO 5.1`, `LO 5.2`, `LO 5.4` — ~20–25 min): Constant Factors, Locality, and Dominance Reasoning.** Analyze three real-world puzzles: (a) An engineer proposes replacing a 20-element `Array` scan (`O(N)`) with a balanced tree (`O(log N)`) to "speed up config lookup"—prove using constant factors and spatial locality why this likely makes it *slower* and more complex; (b) Both a contiguous array and a linked list take `Θ(N)` to sum `1,000,000` integers—explain mechanically why the array runs much faster on modern hardware; (c) A checkout endpoint spends `0.8 ms` sorting 50 cart items in JS and `420 ms` making 30 sequential `14 ms` calls to an inventory service—classify where optimization effort actually pays off.
- **Exercise 2 (`LO 5.3` — ~25–30 min): Pre-Implementation Back-of-the-Envelope Sizing (Online-Store Context).** Product proposes caching the entire order-item history of the past 12 months inside a single Node.js process's in-memory `Map` so export endpoints use expected `O(1)` lookup time. Given: `5,000,000` orders, average `4` items per order, `~120 bytes` raw payload per item, on a `2 GB` RAM container—apply an **illustrative exercise assumption** of `3×–4×` runtime object/`Map` memory overhead *(explicitly not a universal or guaranteed conservative multiplier; actual JS object and `Map` memory overhead varies by runtime representation, key/value shapes, and workload, with V8 memory internals deferred to Phase 5)*. Estimate total memory footprint step-by-step (both raw payload and under the illustrative assumption), determine whether the proposal is feasible, and state what alternative design boundary is required before writing code.
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~20–25 min`).

#### Study-Time Breakdown (Block 5)
- Guided Lesson + Pause & Checks 1–3: `~45–60 min` *(flexible priority)*
- Mandatory 1 (Colin Scott Latency Numbers): `~10–15 min`
- Mandatory 2 (Princeton 1.4 Observations & Memory Usage): `~15–20 min`
- Exercise 1: `~20–25 min` | Exercise 2: `~25–30 min`
- **Total Required Core (without optional assessment):** **`~115–150 min (~1.9–2.5 h)`**
- Optional Closed-Book Assessment: `~20–25 min` *(optional)*
- **Total (with optional assessment):** **`~135–175 min (~2.25–2.9 h)`** *(Roadmap target: `2.5 h`)*

---

### Block 6: Algorithmic Performance Reasoning in Backend Request Handling *(Target: 3.0 hours)*

- **Purpose:** Translate Blocks 1–5 directly into day-to-day backend engineering practice—recognizing the recurring hidden complexity and memory traps inside API handlers, batch processors, and AI-generated code, and refactoring them cleanly using explicit time-space trade-offs without premature overengineering.
- **Dependencies:** Blocks 1–5 (multi-variable analysis, hidden built-in costs, hash tables/heaps, I/O dominance, memory estimation) + Phase 1 Blocks 6 & 7 (simplicity vs. accidental complexity, AI output verification).
- **Prepares For:** Block 7 (empirically testing and proving these refactorings in Node.js), Phases 13–14 (backend service & PostgreSQL integration engineering).
- **Primary Target Level:** `Practical Competence` | **Phase 2 Outcomes Supported:** Outcomes 3, 5, 6, 7.

#### Learning Objectives
- **LO 6.1 (`Practical Competence` | Bloom: *Analyze*):** Diagnose the **six recurring backend complexity anti-patterns** in realistic Node.js handler/service code: (1) repeated linear scans / `O(N * M)` in-memory joins, (2) `O(N^2)` event deduplication via `.includes()`, (3) quadratic array/copying traps (`.shift()` queue drain; `[...acc, item]` inside `.reduce()` producing `Θ(N^2)` cumulative copying/allocation and GC pressure, distinct from peak live auxiliary memory), (4) unbounded full-load + sort instead of bounded Top-K / pagination bounds, (5) conceptual **N+1 sequential I/O scaling**, and (6) catastrophic worst-case regex/parsing traps (ReDoS).
- **LO 6.2 (`Practical Competence` | Bloom: *Create / Evaluate*):** Refactor inefficient backend request-handling logic into asymptotically superior alternatives (e.g., `O(N * M)` → `O(N + M)` via `Map` indexing, `O(N^2)` → `O(N)` via `Set`, `O(N log N)` → `O(N log K)` bounded Top-K), explicitly quantifying both the **time gained** and the **auxiliary memory / complexity trade-off paid**.
- **LO 6.3 (`Practical Competence` | Bloom: *Evaluate*):** Audit **AI-generated backend code** specifically for silent scaling regressions (code that looks clean and passes small unit tests at `N = 5` but collapses at `N = 50,000`), applying Phase 1 Block 7 verification discipline to computational cost.
- **LO 6.4 (`Working Understanding` | Bloom: *Evaluate*):** Apply a concise **Pre-Implementation Complexity & Bounds Review Checklist** (input ceiling, growth variables, worst-case trigger, memory bound, I/O round-trips) while refusing **premature optimization** when `N` is provably tiny and bounded (connecting back to Phase 1 Block 6 simplicity).

#### Concepts Covered & Explicitly Deferred
- **Covered (Essential):** Hidden `O(N * M)` joins (`orders.map(o => users.find(...))`); `O(N^2)` deduplication; `Array.shift()` vs. index-pointer/circular queue; immutable spread-in-reduce `[...acc, item]` trap (`Θ(N^2)` total cumulative copying/allocation and GC allocation churn—explicitly distinguished from **peak live auxiliary memory**, which is `Θ(N)` reachable data plus runtime/GC-dependent transient retention); bounded Top-K (`O(K)` space) vs. full sort (`O(N log N)` time, `O(N)` space); conceptual N+1 I/O round-trip scaling (`(N + 1) * RTT` vs. `2 * RTT` batched lookup + in-memory `Map`); real-world catastrophic backtracking outage case study (Cloudflare July 2, 2019 WAF regex outage); AI-generated complexity traps; knowing when *not* to optimize (tiny bounded `N`, e.g., `<= 10` tax rules).
- **Explicitly Deferred:** Writing SQL `JOIN`s, `LIMIT/OFFSET` vs. cursor pagination SQL mechanics, or `pg` connection-pool batching (Phases 9, 10, 14); Node.js Stream chunked processing & backpressure internals (Phase 11); distributed pagination/fan-out (Phases 22, 27).

#### Terminology Inventory *(Internal Authoring Reference)*
| Term | Official Target Level | Why It Matters in Phase 2 |
| :--- | :---: | :--- |
| Accidentally Quadratic (`O(N^2)` in Disguise) | `Practical Competence` | Most backend algorithmic outages come from innocently nesting a linear helper inside a loop. |
| In-Memory Hash Join / Index Build (`O(N + M)`) | `Practical Competence` | Standard backend pattern replacing `O(N * M)` repeated `.find()` lookups. |
| Cumulative Allocation vs. Peak Live Auxiliary Memory (Allocation Churn) | `Working Understanding` | `[...acc, item]` in `.reduce()` incurs `Θ(N^2)` cumulative copying/time and GC pressure even though live reachable auxiliary data is `Θ(N)` (actual peak RAM depends on lifetimes and GC timing). |
| Conceptual N+1 I/O Scaling | `Working Understanding` | Unifies asymptotic reasoning with Block 5 latency math before Phase 14 database integration. |
| Catastrophic Backtracking (Real-World `O(2^N)` / `O(N^2)`) | `Working Understanding` (mechanism) | Concrete production proof (Cloudflare 2019) that a single bad regex complexity curve can cause global outage. |

#### Authoritative Resources & Assigned Scope
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | Pat Morin, *Open Data Structures* | [Section 1.7 — List of Data Structures](https://opendatastructures.org/ods-python/1_7_List_Data_Structures.html) | Full short summary table comparing list, unordered set (`USet`), sorted set (`SSet`), and priority queue operation costs | ~10–15 min | Compact engineering reference map for matching backend access patterns to data structures. |
| **Mandatory 2** | Free | John Graham-Cumming (Cloudflare Engineering Blog, July 12, 2019) | [Details of the Cloudflare outage on July 2, 2019](https://blog.cloudflare.com/details-of-the-cloudflare-outage-on-july-2-2019/) | Targeted sections: *Executive summary*, *What happened* (regex CPU exhaustion mechanism & backtracking explanation), and *The fix* *(skip low-level Lua/WAF deployment minutiae)* | ~15–20 min | Real-world production postmortem proving how worst-case algorithmic complexity (`.*(?:.*=.*)`) can saturate global backend CPU in seconds. |
| **Supplementary** | Free | Russ Cox | [Regular Expression Matching Can Be Simple And Fast](https://swtch.com/~rsc/regexp/regexp1.html) | Opening problem statement & exponential backtracking graph vs. linear NFA approach *(skip C Thompson-NFA implementation details)* | Optional | Clear visual proof of why backtracking regex engines hit `O(2^N)` while linear engines stay `O(N)`. |

#### Practical Exercises & Assessment Approach
- **Exercise 1 (`LO 6.1`, `LO 6.2` — ~35–45 min): Backend Request-Handler Complexity Audit & Refactor.** Given a realistic ~60-line Node.js online-store batch-processing module containing 4 planted backend scaling traps — (1) enriching `N` orders with `M` customer profiles via `.find()`, (2) deduplicating `E` incoming webhook events via `seen.includes(...)`, (3) accumulating transformed records via `items.reduce((acc, x) => [...acc, transform(x)], [])`, and (4) fetching promo metadata one-by-one inside a `for` loop (`N` sequential async calls) — analyze the exact time complexity, **cumulative copying/allocation vs. peak live auxiliary space** (explicitly explaining why trap #3 creates `Θ(N^2)` cumulative allocation and GC pressure rather than `Θ(N^2)` live reachable memory), and I/O round-trip scaling of each, refactor all 4 cleanly, and document the exact time-space trade-off for each fix.
- **Exercise 2 (`LO 6.3`, `LO 6.4` — ~25–30 min): AI-Generated Snippet Audit & "Do Not Over-Optimize" Judgment.** Audit two short AI-generated Node.js functions: one that silently blows up at `N = 50,000` (needs refactoring), and one operating over a provably bounded list of `<= 8` shipping regions where an AI proposed a complex custom hash-indexed cache wrapper (needs rejection as premature optimization per Phase 1 Block 6!). Justify both decisions quantitatively using growth bounds, constant factors, and simplicity.
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~20–25 min`).

#### Study-Time Breakdown (Block 6)
- Guided Lesson + Pause & Checks 1–3: `~50–65 min` *(flexible priority)*
- Mandatory 1 (Open Data Structures `1.7` cost summary): `~10–15 min`
- Mandatory 2 (Cloudflare July 2, 2019 outage postmortem targeted): `~15–20 min`
- Exercise 1: `~35–45 min` | Exercise 2: `~25–30 min`
- **Total Required Core (without optional assessment):** **`~135–175 min (~2.25–2.9 h)`**
- Optional Closed-Book Assessment: `~20–25 min` *(optional)*
- **Total (with optional assessment):** **`~155–200 min (~2.6–3.3 h)`** *(Roadmap target: `3.0 h`)*

---

### Block 7: Empirical Measurement vs. Theory, Benchmarking Pitfalls, and Phase 2 Capstone Lab *(Target: 4.0 hours)*

- **Purpose:** Unite theoretical complexity reasoning with empirical verification (continuing Phase 1 Block 5's scientific method). Teach why naive benchmarks deceive engineers (DCE, JIT warmup, GC noise, single-`N` bias, passive benchmarking), how to measure multi-`N` scaling curves safely in Node.js using monotonic high-resolution timers, and consolidate all of Phase 2 in a hands-on **Pure Node.js Capstone Scaling Laboratory**.
- **Dependencies:** All Phase 2 Blocks 1–6 + Phase 1 Blocks 5, 7, 8 (scientific method, anti-methods, independent verification, engineering ownership).
- **Prepares For:** Phase 3 (OS hardware effects behind measured jitter), Phase 5 (V8 JIT & GC mechanics behind warmup/noise), Phase 25 (production profiling, flame graphs, load testing).
- **Primary Target Level:** `Practical Competence` | **Phase 2 Outcomes Supported:** Outcomes 1–8 (Integrative Capstone).

#### Learning Objectives
- **LO 7.1 (`Working Understanding` | Bloom: *Evaluate*):** Explain the complementary roles and limits of **theoretical complexity analysis** vs. **empirical benchmarking**—why measurements alone can never *prove* an asymptotic bound (finite `N`, machine/input specificity) and why theory alone cannot predict exact wall-clock latency (constant factors, JIT, locality, GC).
- **LO 7.2 (`Practical Competence` | Bloom: *Analyze*):** Diagnose the five foundational **microbenchmarking pitfalls**: (1) **Dead-Code Elimination (DCE)** & constant folding (as possible optimization-related distortions under specific conditions when an engine can prove a computation is side-effect-free and unused—not an automatic elimination of every unused call—plus ensuring any result-consumption sink does not itself dominate the measurement), (2) **JIT warmup** (cold vs. steady-state execution), (3) **GC & allocation noise**, (4) **wall-clock `Date.now()` instability** vs. monotonic high-resolution clocks (`performance.now()` / `process.hrtime.bigint()`), and (5) **Passive Benchmarking** (single-run, mean-only, unrepresentative `N` or skewed input distribution).
- **LO 7.3 (`Practical Competence` | Bloom: *Apply / Analyze*):** Design and run a controlled **multi-`N` doubling/scaling experiment** in Pure Node.js 26 LTS (`node:perf_hooks`) with warmup, lightweight observable result consumption (avoiding both DCE risk and sink-dominated measurements), multiple trials (median & spread), and varying input sizes (`N, 2N, 4N, 8N`) to empirically distinguish `O(1)`/`O(log N)`, `O(N)`/`O(N log N)`, and `O(N^2)` growth curves—and separate algorithmic slowness from environmental/constant-factor noise.
- **LO 7.4 (`Practical Competence` | Bloom: *Evaluate / Create*):** Synthesize **all seven Phase 2 blocks** in an end-to-end **Phase 2 Capstone Scaling Lab**: statically analyze a realistic multi-operation online-store module, predict scaling ratios, measure empirically, refactor with explicit time-space trade-offs, verify improvements, and defend remaining measurement limitations honestly.

#### Concepts Covered & Explicitly Deferred
- **Covered (Essential):** Scientific partnership between theory (predicts scaling ratio) and measurement (corroborates/falsifies hypothesis and reveals constant factors/crossovers); monotonic high-resolution timing (`performance.now()`, `process.hrtime.bigint()`) vs. `Date.now()`; **Dead-Code Elimination (DCE)** as a conditional optimization distortion (when an optimizing engine proves a computation has no observable effects—requiring observable yet negligible-cost result consumption that does not dominate the benchmark); JIT warmup concept (cold vs. steady state — conceptual `Working Understanding`, V8 tier internals deferred to Phase 5); GC/allocation jitter & multi-trial summaries (median, min/max/p95 spread); Active vs. Passive Benchmarking (Brendan Gregg); empirical doubling-ratio check (`T(2N)/T(N)` → `~1×`, `~2×`, `~4×`) and its caveats!
- **Explicitly Deferred:** V8 bytecode/TurboFan disassembly (`--print-opt-code`, `--trace-deopt`) (Phase 5); Linux `perf`, flame graphs, Node `--cpu-prof` / heap snapshots (Phases 23 & 25); HTTP load generators & distributed benchmarking (Phase 25).

#### Terminology Inventory *(Internal Authoring Reference)*
| Term | Official Target Level | Why It Matters in Phase 2 |
| :--- | :---: | :--- |
| Monotonic High-Resolution Clock (`performance.now` / `hrtime`) | `Practical Competence` | Immune to system clock adjustments and provides sub-millisecond precision. |
| Dead-Code Elimination (DCE) & Observable Result Sinks | `Working Understanding` | Under specific conditions (provably side-effect-free, unused computation), an optimizing engine may eliminate benchmark work; making results observable with a negligible sink avoids both DCE distortion and sink-dominated timing. |
| JIT Warmup (Cold vs. Steady State) | `Working Understanding` | First iterations include compilation overhead; comparing cold vs. warmed runs invalidates benchmarks. |
| Empirical Scaling / Doubling Ratio (`T(2N) / T(N)`) | `Practical Competence` | Practical way to check whether real measured growth matches predicted `O(N)` (`~2×`) or `O(N^2)` (`~4×`). |
| Active Benchmarking (Gregg) | `Working Understanding` | Verifying *what* is actually executing and limiting the benchmark rather than blindly trusting a printed number. |

#### Authoritative Resources & Assigned Scope
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | Brendan Gregg | [Active Benchmarking](https://www.brendangregg.com/activebenchmarking.html) | *Summary*, *Passive Benchmarking*, *Active Benchmarking*, *Problem Checklist*, and *Statistical Analysis* overview *(skip Linux `perf`/DTrace tool commands until Phases 23/25)* | ~15–20 min | Core mental model: *"you benchmark A, actually measure B, and conclude C"*—and how active verification prevents benchmark delusions. |
| **Mandatory 2** | Free | Vyacheslav Egorov (`mraleph`, former V8 engineer) | [microbenchmarks fairy tale](https://mrale.ph/blog/2012/12/15/microbenchmarks-fairy-tale.html) & [Microbenchmarks are experiments](https://mrale.ph/blog/2024/11/27/microbenchmarks-are-experiments.html) | Read *microbenchmarks fairy tale* (core sections on LICM, constant propagation, DCE, and *Moral*) + opening thesis of *Microbenchmarks are experiments* (*"Benchmarks are not numerology"*) *(skip assembly dumps)* | ~20–25 min | Why optimizing JIT compilers can silently distort naive JS loop benchmarks under specific optimization conditions and why benchmarks must be treated as falsifiable experiments. |
| **Mandatory 3** | Free | Node.js Official Documentation (Node.js 26 LTS — pinned Phase 2 runtime) | [Performance measurement APIs (`node:perf_hooks`, v26.x)](https://nodejs.org/docs/latest-v26.x/api/perf_hooks.html) & [Process API (`process.hrtime.bigint`, v26.x)](https://nodejs.org/docs/latest-v26.x/api/process.html#processhrtimebigint) | Targeted skim: `performance.now()` (monotonic millisecond timestamp) and `process.hrtime.bigint()` *(Release status verified via [nodejs/Release](https://github.com/nodejs/Release): Node.js 26 `Lithium` is Active LTS as of Nov 26, 2026)* | ~10 min | Safe, standard, zero-dependency Node.js 26 LTS timing primitives for Block 7's scaling experiments. |
| **Supplementary** | Free | V8 Engineering Blog | [How V8 measures real-world performance](https://v8.dev/blog/real-world-performance) | Conceptual skim on why synthetic microbenchmarks diverge from real workloads | Optional | Reinforces limits of microbenchmarks in production engines. |

#### Pre-Lab Exercise & Proposed Phase 2 Capstone Practical Laboratory
- **Pre-Lab Exercise (`LO 7.2` — ~15–20 min): Spot the Broken Microbenchmark.** Inspect a realistic 30-line Node.js benchmark snippet that commits 4 classic flaws: uses `Date.now()` around a single run, discards a pure loop's computed value without making it observable (creating DCE distortion risk) *or* logs inside the hot loop so I/O dominates timing, benchmarks only `N = 50` on already-sorted input without warmup, and mixes different input-allocation costs inside the timed section. Diagnose each flaw and rewrite the measurement harness cleanly with lightweight observable result consumption.
- **Proposed Phase 2 Capstone Practical Laboratory (`phases/phase-02/lab/` — to be authored/executed ONLY when Phase 2 is unlocked, NOT now):**
  - **Strict Phase 2 Scope Boundary:** **Pure Node.js 26 LTS only** (`.mjs`, zero external `npm` dependencies, built-in `node:perf_hooks` + `node:test` + `node:assert/strict` — Node.js 26 `Lithium` is the approved Phase 2 runtime, Active LTS as of November 26, 2026, distinct from Alpha/Current release tracks such as Node.js 27 Alpha; Phase 1's Node.js 22 Maintenance LTS pin remains unchanged). No HTTP servers, no PostgreSQL database setup, no external benchmarking frameworks, no V8 profiler flags.
  - **Scenario (Online-Store Continuity):** An existing pure-JS online-store analytics & checkout helper module (`storeBatchEngine.mjs`) serves three core operations: (A) enriching `N` orders with `M` customer/promo catalog entries, (B) deduplicating `N` incoming payment/webhook events, and (C) extracting the Top-`K` highest-value orders from `N` records. Everything passed unit tests at `N = 50`, but staging batch runs at `N = 50,000` timed out.
  - **Four Integrative Lab Stages (`~100–120 min` total):**
    1. **Stage 1 — Static Theoretical Prediction (`~25–30 min`, Blocks 1–4):** Before running any benchmark, analyze every operation statically: derive tight time and auxiliary space complexity (in exact variables `N, M, K`), predict the empirical doubling ratio `T(2N)/T(N)`, estimate auxiliary memory at `N = 100,000`, and write falsifiable predictions.
    2. **Stage 2 — Controlled Baseline Scaling Experiment (`~25–30 min`, Blocks 5 & 7):** Build a clean `node:perf_hooks` multi-`N` scaling script (`N = 1,000; 2,000; 4,000; 8,000; 16,000`) with warmup, anti-DCE checksum sink, and multiple trials (reporting median and min/max spread). Compare observed scaling ratios against Stage 1 predictions and explain any crossover or GC/warmup anomalies honestly.
    3. **Stage 3 — Complexity Refactoring & Correctness Preservation (`~30–35 min`, Blocks 4 & 6):** Refactor each bottleneck using appropriate data structures (`Map` index, `Set` deduplication, bounded `O(K)` min-heap / partial selection) while preserving **100% identical outputs verified by `node:test`** (never trading correctness for speed!). Document exact time-space trade-offs.
    4. **Stage 4 — After-Measurement Comparison & Honest Performance Defense (`~20–25 min`, Blocks 5 & 7 + Phase 1 B8):** Re-run the exact same controlled scaling script on the refactored module (including larger `N = 100,000` where the old version collapsed), record before/after medians and scaling ratios, explain what additional memory was traded for the speedup, and explicitly state what this in-memory benchmark **did not prove** (e.g., real PostgreSQL I/O, network RTT, multi-request concurrency, long-term GC pressure).
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Synthesis Assessment (`~25–30 min`, strictly optional).

#### Study-Time Breakdown (Block 7)
- Guided Lesson + Pause & Checks 1–3: `~50–65 min` *(flexible priority)*
- Mandatory 1 (Brendan Gregg *Active Benchmarking*): `~15–20 min`
- Mandatory 2 (`mraleph` microbenchmarking articles targeted): `~20–25 min`
- Mandatory 3 (Node.js `node:perf_hooks` / `hrtime` skim): `~10 min`
- Pre-Lab Exercise: `~15–20 min` | 4-Stage Capstone Scaling Lab: `~100–120 min`
- **Total Required Core (without optional assessment):** **`~210–260 min (~3.5–4.3 h)`**
- Optional Closed-Book Synthesis Assessment: `~25–30 min` *(optional)*
- **Total (with optional assessment):** **`~235–290 min (~3.9–4.8 h)`** *(Roadmap target: `4.0 h`)*

---

## 7. Coverage Mapping Against Proposed Phase 2 Mastery Outcomes (1–8)

| # | Proposed Phase 2 Mastery Outcome | Primary Block(s) | Supporting Block(s) | Mapped Block LOs | Core Free Mandatory Sources | Verification / Demonstration Path |
| :---: | :--- | :---: | :---: | :--- | :--- | :--- |
| **1** | Explain what complexity measures (word-RAM) & separate growth from wall-clock runtime | Block 1, Block 5 | Block 7 | `LO 1.1`, `LO 1.4`, `LO 5.1`, `LO 7.1` | Open Data Structures `1.1, 1.4, 1.5`; MIT 6.006 Lec 1; Princeton `1.4` | Notation/claim audit + constant-factor/locality puzzles |
| **2** | Use `O`, `Ω`, `Θ` accurately across complexity classes without Big-O myths | Block 1 | Block 2 | `LO 1.2`, `LO 1.3`, `LO 2.1` | MIT 6.006 Lec 1; Open Data Structures `1.3`; Princeton `1.4` | Scaling growth tables, crossover math, notation audit |
| **3** | Derive time & auxiliary space complexity for loops, multi-variable inputs, hidden costs & recursion trees | Block 2 | Block 6, Block 7 | `LO 2.1`, `LO 2.2`, `LO 2.3`, `LO 2.4`, `LO 6.1` | Princeton `1.4`; MIT 6.006 Rec 1 & Lec 3 (recurrence excerpt) | Multi-variable & hidden-cost code audit + recursion trees & stack depth |
| **4** | Distinguish best, average/expected, worst-case & amortized complexity + algorithmic DoS risk | Block 3 | Block 4 | `LO 3.1`, `LO 3.2`, `LO 3.3`, `LO 3.4`, `LO 4.2` | MIT 6.006 Lec 2; Open Data Structures `2.1.2–2.1.3`; Crosby & Wallach (USENIX '03) | Aggregate-method dynamic buffer proof (`+k` vs `×2` vs thrashing) + production risk audit |
| **5** | Compare core data structures & searching/sorting patterns with justified time-space trade-offs | Block 4 | Block 6, Block 7 | `LO 4.1`, `LO 4.2`, `LO 4.3`, `LO 4.4`, `LO 6.2` | MIT 6.006 Lec 4; Princeton `3.1` & `3.4`; Open Data Structures `10.1`, `11.1`, `1.7` | Workload→data-structure decision matrix + `(N, Q)` regime comparison |
| **6** | Diagnose & refactor hidden backend & AI-generated algorithmic/memory traps | Block 6 | Block 2, Block 7 | `LO 6.1`, `LO 6.2`, `LO 6.3`, `LO 6.4`, `LO 7.4` | Open Data Structures `1.7`; Cloudflare July 2, 2019 Outage Postmortem | Backend handler audit & refactor + AI snippet critique + premature-optimization check |
| **7** | Reason quantitatively about constant factors, locality, latency hierarchy & resource sizing | Block 5 | Block 6, Block 7 | `LO 5.1`, `LO 5.2`, `LO 5.3`, `LO 5.4` | Colin Scott Interactive Latency Numbers; Princeton `1.4` Memory Usage | Locality/I/O-dominance puzzles + back-of-the-envelope memory/latency sizing |
| **8** | Run controlled multi-`N` Node.js scaling experiments & avoid microbenchmark pitfalls | Block 7 | Block 5 | `LO 7.1`, `LO 7.2`, `LO 7.3`, `LO 7.4` | Brendan Gregg *Active Benchmarking*; `mraleph` microbenchmark articles; Node.js `perf_hooks` API | Broken-benchmark audit + 4-stage Pure Node.js Capstone Scaling Lab |

---

## 8. Teaching Standards & Authoring Contract for Future Phase 2 Blocks

When ChatGPT approves this dossier and authorizes authoring Phase 2 teaching materials, every block file (`phases/phase-02/materials/block-01-...` through `block-07-...`) **must** follow these established repository standards:

1. **Natural Bangla-English Code-Switching (`DEC-004`):** Explain concepts, derivations, mechanisms, and trade-offs primarily in natural Bangla prose; keep common everyday English words, technical terminology, code, complexity notation (`` `O(N log N)` ``, `` `Θ(N^2)` ``), commands, identifiers, and original source titles in English; avoid forced Bangla translations and phonetic Bangla transliteration of English words.
2. **Connected Guided Discovery (`DEC-003` & `teaching-material-design/SKILL.md`):** Open straight into a compelling backend scaling problem → let the learner feel the limitation → build intuition → introduce formal notation/terms *after* intuition → explain step-by-step mechanisms, invariants, trade-offs, and failure modes → motivate the next section causally. Never dump upfront terminology tables, prerequisite boxes, or reference-manual clutter ahead of the guided lesson.
3. **Flexible Study Time Guide Placement:** Place the **Study Time Guide immediately after the `#` block title and before the guided lesson**, itemizing the guided lesson + Pause & Checks, each mandatory resource individually, each exercise/stage individually, optional assessment, and totals both with and without the optional assessment. All later time mentions must match the guide identically.
4. **Forward-Reference Seam Protocol (`LEARNING_JOURNEY.md` line 303):** Whenever citing later-phase systems (CPU caches, network RTT, V8 JIT/GC, PostgreSQL B-Trees, HTTP endpoints), explain them in 1–2 plain, accessible sentences inline (`Supporting` / `Context` level) and never require their internal mechanics to solve Phase 2 exercises.
5. **Evolving Online-Store Spine (Without Forcing):** Use our recurring e-commerce online store (orders, promo codes, webhook events, product catalogs) wherever it naturally clarifies backend scaling; switch freely to cleaner native examples (e.g., sorting numbers, array doubling, abstract operation tables) where simpler.
6. **Deferred Self-Check Guidance & Learner Ownership:** Provide Self-Check Guidance strictly at the end of the file and strictly for inline Pause & Checks 1–3; never include answer keys for mandatory exercises, the Block 7 Capstone Lab, or Optional Closed-Book Assessments.
7. **Pinned Phase 2 Node.js Runtime & Release-Track Precision:** Pin **Node.js 26 LTS (`Lithium`)** for all Phase 2 practical experiments and the Block 7 capstone (per curriculum-owner decision recorded in Section 12.3; Active LTS as of November 26, 2026 per [nodejs/Release](https://github.com/nodejs/Release), while Node.js 22 and 24 are Maintenance LTS and Node.js 27 is Alpha). Always distinguish **Active LTS** from **Alpha** and **Current** releases—never claim Node.js 26 is "the latest release" merely because it is Active LTS. Do not alter Phase 1's Node.js 22 environment.
8. **Formatting Hygiene:** Keep complexity notation and code identifiers inside backticks (`` `O(N)` ``) and avoid bare unescaped `$` outside code spans so Markdown/KaTeX renders cleanly everywhere.

---

## 9. Technical Accuracy Guardrails & Mathematical Verification Notes

Every future Phase 2 teaching block must uphold these verified mathematical and systems invariants:

1. **Formal Asymptotic Definitions (Verified against CLRS 4th ed. Ch. 3 & MIT 6.006 Lec 1):**
   - `f(n) = O(g(n))` iff there exist positive constants `c > 0` and `n0 >= 1` such that `0 <= f(n) <= c * g(n)` for all `n >= n0` (*asymptotic upper bound*).
   - `f(n) = Ω(g(n))` iff there exist `c > 0` and `n0 >= 1` such that `0 <= c * g(n) <= f(n)` for all `n >= n0` (*asymptotic lower bound*).
   - `f(n) = Θ(g(n))` iff `f(n) = O(g(n))` AND `f(n) = Ω(g(n))` (*asymptotic tight bound*).
   - **Mandatory Precision Rule:** `2n + 10` is *technically* `O(n^2)` (since `n^2` upper-bounds it), while its **tight** bound is `Θ(n)`. Never claim "`O(n)` means worst-case" — always state the case (*best / average / expected / worst / amortized*) separately from the bound (`O` / `Ω` / `Θ`).
2. **Verified Loop & Recurrence Derivations:**
   - Triangular dependent loop (`for i = 0..n-1`, `for j = i+1..n-1`): exact inner iterations = `n(n - 1) / 2 = Θ(n^2)` (NOT `O(n)` because the inner bound depends on `i`).
   - Balanced divide-and-conquer (`T(n) = 2T(n/2) + Θ(n)`): `log2(n) + 1` levels, each doing `Θ(n)` work → `Θ(n log n)` time; merge sort uses `Θ(n)` auxiliary array space + `Θ(log n)` call-stack space = `Θ(n)` auxiliary space total.
   - Halving binary search (`T(n) = T(n/2) + Θ(1)`): `Θ(log n)` time; `Θ(1)` auxiliary space iteratively vs. `Θ(log n)` call-stack space recursively.
3. **Amortized vs. Average-Case Precision (Verified against MIT 6.006 Lec 2, ODS 1.5 & 2.1, CLRS Ch. 17):**
   - **Amortized analysis** makes **zero probabilistic assumptions** about inputs; it bounds the *worst-case total cost* of **any** sequence of `N` operations divided by `N`.
   - **Average-case analysis** assumes an input probability distribution; **expected-time analysis** averages over internal random choices made by the algorithm (e.g., randomized quicksort, universal hashing).
   - **Operational Caveat:** An `O(1)` amortized append still incurs an `O(N)` resize pause when capacity doubles—critical for understanding single-request tail latency!
   - **Dynamic Array Shrinking Hysteresis (ODS 2.1.2):** Doubling at `n == cap` and halving immediately at `n == cap / 2` breaks `O(1)` amortized cost under alternating push/pop at the boundary (`Θ(N)` per op!). Shrinking only when `n <= cap / 4` (or `cap / 3`) restores `O(1)` amortized cost.
4. **Hash-Table Cost Model vs. Runtime Implementation Precision:**
   - Expected `O(1)` lookup/insert/delete is the **conventional abstract hash-table cost model** (conditional on hash distribution and bounded load factor), **not** an unconditional runtime guarantee.
   - Historical USENIX Security 2003 collision attacks (Crosby & Wallach) illustrate the abstract `O(N^2)` collision-degradation mechanism on specific non-randomized implementations of that era; future lessons must **not** imply that modern native JavaScript `Map`/`Set` necessarily suffers those exact historical vulnerabilities (V8's internal hash-table design remains deferred to Phase 5). Keep **ReDoS** distinct as a separate concrete worst-case risk.
5. **Cumulative Allocation vs. Peak Live Auxiliary Memory Precision:**
   - Patterns such as `items.reduce((acc, item) => [...acc, item], [])` incur **`Θ(N^2)` total time and `Θ(N^2)` cumulative element copying/allocation** (heavy allocation churn and GC pressure), whereas live reachable auxiliary data at any step is `Θ(N)` and actual peak RAM depends on object lifetimes and runtime GC cycles. Never conflate quadratic cumulative allocation with quadratic peak live memory.
6. **Theoretical Complexity vs. Practical Runtime & Benchmarking Limits:**
   - Word-RAM assumes uniform `O(1)` memory access and fixed-size words; real hardware has multi-level caches, non-uniform RAM/SSD/network latencies spanning 6–8 orders of magnitude, and runtime JIT/GC overhead.
   - **DCE & Observable Sinks:** Dead-code elimination is a conditional optimization distortion (occurring when an optimizing engine can prove a computation is side-effect-free and unused, not automatically on every unused function call); benchmarks must make intended results observable via a lightweight sink that does not itself dominate the timing.
   - Empirical benchmarks over finite `N` can **corroborate or falsify** a scaling hypothesis and measure practical constant factors/crossover points, but can **never mathematically prove** an asymptotic bound (`n → ∞`).

---

## 10. Phase 2 Completion Criteria & Checkpoint Expectations

Consistent with `AGENTS.md`, `PROGRESS.md`, and `.agents/skills/phase-checkpoint-handoff/SKILL.md`:

- **Learner-Controlled Block Pacing:** Optional closed-book assessments are learner-controlled and never block studying the next block (`Studied — mastery not formally verified` vs. `Mastery verified`).
- **Required Evidence for Phase 2 Checkpoint Approval (ChatGPT Final Authority):**
  1. Learner-authored conceptual notes, Pause & Checks, and mandatory exercises across Blocks 1–7 (`phases/phase-02/notes/block-01-notes.md` through `block-07-notes.md`).
  2. Verified, runnable Pure Node.js scaling experiments and refactored code from the Block 7 Capstone Lab in `phases/phase-02/lab/`, including static predictions, controlled before/after `node:perf_hooks` measurements, passing `node:test` correctness proofs, and honest limitation defense.
  3. Updated, learner-verified entries in `BACKEND_KNOWLEDGE_INDEX.md` (covering core Phase 2 complexity/data-structure/benchmarking concepts at their official depth levels).
  4. Compiled mastery handoff packet in `phases/phase-02/checkpoint.md` acknowledging any open gaps honestly.
  5. **Explicit `approved` sign-off from ChatGPT (Curriculum Owner & Instructor)** before unlocking Phase 3 (*Operating Systems*).

---

## 11. How Phase 2 Prepares the Learner for Phase 3 and Later Engineering Work

Phase 2 ends right at the boundary of the **word-RAM abstraction**:
- In Block 5 and Block 7, the learner discovers *empirically* that contiguous arrays crush linked lists despite identical `Θ(N)` complexity, that `100 ns` RAM access vs. `1 ns` L1 cache vs. `100 µs` SSD changes real performance dramatically, and that Node.js timing shows mysterious warmup and GC spikes.
- That directly creates the exact unanswered engineering questions that launch **Arc II (Phases 3–6)**:
  - **Phase 3 (Operating Systems):** *What actually happens inside the CPU, virtual memory, OS page cache, and disk I/O beneath those latency numbers?*
  - **Phase 4 (Computer Networking):** *Why does a single network round-trip take milliseconds and how do bytes actually cross machines?*
  - **Phase 5 (JavaScript Language Internals):** *What is V8 actually doing during JIT warmup, object allocation, and garbage collection?*
  - **Phase 6 (Concurrency & Asynchronous Systems):** *What happens when 500 concurrent requests execute our `O(N)` and `O(N^2)` handlers at the same time?*
- And later in **Arc III–VI**, Phase 2's mental models directly unlock **Phase 10 (Database Engineering — B-Tree & Hash Indexes)**, **Phase 14 (Eliminating N+1 DB Calls)**, **Phase 16 (Caching Economics)**, **Phase 19 (ReDoS & Hash-Flooding Security)**, and **Phase 25 (Production Performance Engineering)**.

---

## 12. Identified Gaps, Uncertainties, Unresolved Questions & Limitations

### 12.1 Resource Accessibility & Verification Notes (Verified `2026-11-26`)
- **All Mandatory Resources Are Free & Live-Verified (`HTTP 200 OK`):**
  - Open Data Structures (`opendatastructures.org/ods-python/` — Sections `1.1, 1.3, 1.4, 1.5, 1.7, 2.1, 10.1, 11.1`)
  - MIT OCW 6.006 Spring 2020 (`ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/` — Lectures `1, 2, 3, 4` and Recitation `1` resource pages)
  - Princeton *Algorithms, 4th ed.* Booksite (`algs4.cs.princeton.edu/` — Sections `1.4, 3.1, 3.4`; `3.3` Supplementary)
  - Crosby & Wallach USENIX Security 2003 (`usenix.org/.../denial-service-algorithmic-complexity-attacks`)
  - Colin Scott Interactive Latency Numbers (`colin-scott.github.io/.../interactive_latency.html`)
  - Cloudflare Engineering Blog July 2, 2019 Outage (`blog.cloudflare.com/details-of-the-cloudflare-outage-on-july-2-2019/`)
  - Brendan Gregg *Active Benchmarking* (`brendangregg.com/activebenchmarking.html`)
  - Vyacheslav Egorov `mraleph` (`mrale.ph/blog/2012/12/15/microbenchmarks-fairy-tale.html` and `mrale.ph/blog/2024/11/27/microbenchmarks-are-experiments.html`)
  - Node.js Official `node:perf_hooks` & `process.hrtime.bigint` Docs for Node.js 26 LTS (`nodejs.org/docs/latest-v26.x/api/perf_hooks.html`, `nodejs.org/docs/latest-v26.x/api/process.html#processhrtimebigint`) and Official Release Schedule (`github.com/nodejs/Release` — `schedule.json` verified `2026-11-26`: Node.js 26 `Lithium` Active LTS since `2026-10-28` [Maintenance `2027-10-20`, EOL `2029-04-30`]; Node.js 24 `Krypton` Maintenance LTS since `2026-10-20`; Node.js 22 `Jod` Maintenance LTS since `2025-10-21`; Node.js 27 Alpha since `2026-10-28` [Current `2027-04-22`])
- **Live Verification Catch (Prevented Scope Error):** Princeton's *online* Section 1.4 (`algs4.cs.princeton.edu/14analysis/`) includes `Observations`, `Mathematical models`, `Designing faster algorithms`, `Coping with dependence on inputs`, and `Memory usage`, but does **not** reproduce the print textbook's standalone *"Caveats"* or *"Doubling ratio"* headings. Accordingly, this dossier assigns only verified live online sections as Mandatory and handles constant-factor/hardware caveats directly in Block 5 alongside Colin Scott's latency table.
- **Excluded / Automated-Fetch-Limited Resources:**
  - `accidentallyquadratic.tumblr.com` timed out during live verification and was excluded rather than cited unreliably.
  - `norvig.com/21-days.html` and `jeffe.cs.illinois.edu/teaching/algorithms/` returned automated browser-check interstitials on raw HTTP fetch, so both are listed strictly as **Supplementary** (not Mandatory).
  - Commercial books (CLRS 4th ed. 2022, Sedgewick & Wayne print 4th ed.) are **Supplementary** only; exact page numbers are marked `[Unverified]`.

### 12.2 Remaining Curriculum Decisions for ChatGPT (Curriculum Owner & Instructor) Final Approval
Before generating any Phase 2 teaching blocks, ChatGPT should give final sign-off on two remaining planning details:
1. **Final Approval of the 8 Proposed Phase 2 Mastery Outcomes & 7-Block Split:** Confirm the 8 proposed outcomes (Section 4) and 7-block sequence totaling `22.0 h` reference target (`18.0–22.5 h` honest required core, midpoint `~20.25 h`) for final approval.
2. **Optional Pre-Block-1 Algebra/Logarithm Warm-Up:** Confirm whether weaving base-2 logarithm & simple series-sum intuition inline inside Blocks 1–2 prose (our recommended default, matching `teaching-material-design/SKILL.md`) is sufficient, or whether a tiny 15-minute optional refresher appendix is desired.

### 12.3 Approved Phase 2 Runtime Decision & Governance Boundary Note
- **Approved Phase 2 Runtime (Curriculum Owner Decision):** **Node.js 26 LTS (`Lithium`)** is pinned for all Phase 2 practical experiments and the Block 7 capstone. Verified against [https://github.com/nodejs/Release](https://github.com/nodejs/Release) as of **November 26, 2026**: Node.js 26 is **Active LTS** (started `2026-10-28`), Node.js 22 (`Jod`) and Node.js 24 (`Krypton`) are **Maintenance LTS**, and Node.js 27 is in **Alpha** (started `2026-10-28`, Current `2027-04-22`). Active LTS is explicitly distinguished from Alpha and Current releases—Node.js 26 is pinned because it is Active LTS, not because it is the newest release line overall. **Phase 1's pinned Node.js 22 environment remains untouched.**
- **Advance-Planning Governance Boundary:** Per `AGENTS.md` Boundary #2 and `DEC-001` #5, normal execution creates phase folders just-in-time as each phase unlocks. Creating and revising `phases/phase-02/research/RESEARCH_DOSSIER.md` ahead of Phase 1 completion was explicitly requested strictly for curriculum planning and ChatGPT review. **No Phase 2 teaching blocks, notes, labs, or progress updates should be generated** until ChatGPT gives final dossier approval and explicitly authorizes next steps.
