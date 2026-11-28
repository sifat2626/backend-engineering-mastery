# Phase 3 Research Dossier — Operating Systems

> **Document Type:** Proposed Pre-Material Research & Planning Dossier (**not** final teaching materials — do not generate Phase 3 teaching blocks or lab files until ChatGPT reviews and approves this dossier)
> **Approval Status:** `Proposed — awaiting ChatGPT (Curriculum Owner & Instructor) review and approval`
> **Phase Authority:** ChatGPT (Curriculum Owner & Instructor) — **36 focused learner-study hours total** (per `MASTER_ROADMAP.md`; block targets are flexible planning estimates, not hard deadlines; mastery takes priority)
> **Advance-Planning & Continuity Notice:** Live learning continuity remains unchanged in `PROGRESS.md` and `NEXT_SESSION.md` (Active Phase: **Phase 1**, Block 1 studied, Block 2 next active study block; Phases 1–2 not approved; Phase 3 remains `planned` and locked). Following the repository's established pre-material planning convention (mirroring `phases/phase-02/research/RESEARCH_DOSSIER.md`), **only** `phases/phase-03/research/RESEARCH_DOSSIER.md` has been created at the learner's explicit request for curriculum planning. No Phase 3 teaching blocks (`materials/`), `lab/`, `notes/`, `README.md`, `checkpoint.md`, or progress/index updates have been created or modified.
> **Knowledge-Depth Model:** Uses the repository's **exactly three official knowledge levels** (`Awareness`, `Working Understanding`, `Practical Competence` — per `DEC-002` and `MASTER_ROADMAP.md`), paired with Bloom's taxonomy cognitive levels for every block learning objective. Mechanism-level depth and production relevance are dimensions applied across these three levels, not additional levels.

---

## 1. Research Methodology & Verification Policy

### Selection Criteria
1. **Primary & Authoritative Sources First:** Anchored on **Operating Systems: Three Easy Pieces (OSTEP), Version 1.10** by Remzi H. Arpaci-Dusseau & Andrea C. Arpaci-Dusseau as the primary conceptual textbook, paired with primary Linux documentation (**Linux `man-pages`** on `man7.org` and the **Linux Kernel Documentation** Memory Management Concepts Guide on `docs.kernel.org`) and strictly contextual **Node.js v26 LTS** `process` API references (`nodejs.org/docs/latest-v26.x/api/process.html`).
2. **100% Free Mandatory Core:** Every **Mandatory** reading and documentation reference in this dossier is freely accessible online without a paywall. Commercial reference books (e.g., Michael Kerrisk's *The Linux Programming Interface*, Brendan Gregg's *Systems Performance*) are listed strictly as **Supplementary**.
3. **Backend-Engineering Relevance Over Kernel Development:** The learner's goal is to become a production-ready, AI-augmented backend engineer specializing primarily in **Node.js and PostgreSQL**. Accordingly, Phase 3 prioritizes foundational OS mechanisms that directly explain backend behavior and production incidents (syscalls, process lifecycles, CPU scheduling/starvation, virtual vs. resident memory, page faults, reclaim/OOM, file descriptors, blocking vs. non-blocking I/O, page cache vs. `fsync`, and signals) rather than kernel-module hacking, x86 assembly drivers, or advanced academic scheduling proofs.
4. **Curated Workload Fit Within 36 Hours:** OSTEP chapters are never assigned blindly in full; every reading is scoped to verified subsections and measured page counts. Across all 8 blocks, the honest required-core estimate is **`28.0–34.2 hours`** (`1,680–2,050 min`, midpoint `~31.1 h`) inside the authoritative **36-hour (`2,160 min`)** roadmap allocation, leaving genuine room for hands-on Linux experiments, small annotated C demonstrations, and reflective note-taking (`31.1–37.9 hours`, midpoint `~34.5 h`, if all 8 optional closed-book assessments are attempted).
5. **Strict Non-Fabrication & Live Verification Policy:** All OSTEP v1.10 chapter PDFs, exact chapter/subsection numbers, page counts, `man7.org` manual pages, `docs.kernel.org` headings, `nodejs.org` v26.13.1 documentation anchors, and local Ubuntu 24.04 Linux tooling were live-verified on **`2026-11-28`**. Details not verified locally (such as exact page numbers of commercial supplementary books) are explicitly labeled `[Unverified]`. Broken or non-existent pages discovered during verification (e.g., `proc_pid_smaps_rollup(5)`, which returned HTTP 404 on `man7.org`) were excluded rather than cited speculatively.

---

## 2. Phase 3 Overview, Purpose, Prerequisites & Longitudinal Connections

### 2.1 Why Phase 3 Exists and What It Achieves
In `LEARNING_JOURNEY.md`, **Phase 3: Operating Systems** launches **Arc II — Machines, Networks, Language & Concurrency Primitives (Phases 3–6)**.

- **Central Driving Question (`LEARNING_JOURNEY.md`):**
  > *"What actually happens inside the operating system when a server starts, allocates memory, opens a socket, or handles multiple tasks?"*
- **Core Purpose:** Replace the abstract "my code runs on a computer" mental model with concrete, mechanism-level understanding of the Linux operating system running beneath every Node.js service and PostgreSQL instance. By the end of Phase 3, the learner can trace how user-space code crosses into kernel space via system calls; how Linux creates, schedules, preempts, isolates, and terminates processes; how virtual memory, page tables, TLBs, demand paging, reclaim, and the OOM killer govern real memory behavior; how file descriptors, the kernel page cache, and blocking vs. non-blocking I/O work; and how to inspect live Linux processes safely to diagnose real backend CPU, memory, FD, I/O, and signal incidents from first principles.
- **Why Phase 3 Is NOT a Kernel-Development or C-Programming Course:**
  This phase does **not** require writing a kernel, implementing a filesystem or page-table allocator, memorizing x86 registers, or undertaking a separate C programming course. Minimal, complete, heavily annotated C snippets (~15–35 lines, with every C construct explained inline in natural Bangla) are used **only** where direct POSIX system-call access (`fork`, `execve`, `waitpid`, `mmap`, `open`/`read`/`O_NONBLOCK`) makes an OS mechanism unmistakably visible in ways a high-level runtime hides.

### 2.2 Inherited Knowledge from Phases 1–2
Phase 3 builds directly on **Arc I (Phases 1–2)**:
- **Phase 1 (*Engineering Mindset & AI-Augmented Engineering*):**
  - *Block 3 (Fault → Error → Failure & Invariants):* Applied directly to OS resource invariants (reaping child processes, closing file descriptors, obeying memory bounds).
  - *Block 4 (Constraints & Trade-Offs):* Grounds OS design tensions—turnaround time vs. response time, buffered Page Cache throughput vs. `fsync` durability, virtual isolation vs. translation/context-switch overhead.
  - *Block 5 (Evidence-Based Engineering, Scientific Method & Conceptual USE Method):* Phase 1 Block 5 introduced Brendan Gregg's **Utilization, Saturation, Errors (USE)** framework and Scientific Method conceptually while explicitly deferring Linux inspection tools (`vmstat`, `ps`, `/proc`, `strace`) to Phase 3 (and deeper ops/profiling to Phases 23 & 25). Phase 3 Block 8 directly operationalizes USE + Scientific Method on real Linux OS observables.
- **Phase 2 (*Complexity and Performance Reasoning*):**
  - *Block 1 & Block 5 (Beyond Word-RAM, Hardware Latency Hierarchy & Spatial Locality):* Phase 2 established empirically and conceptually that memory is *not* uniform (`~1–10 ns` CPU cache vs. `~100 ns` RAM vs. `~10–100 µs` SSD vs. `~0.5–100 ms` network) and that contiguous arrays beat pointer chasing via spatial locality. Phase 3 reveals the OS and hardware machinery behind those numbers: **TLB locality** (Block 4), **minor vs. major page faults** hitting SSD/swap (Block 5), **kernel Page Cache vs. physical disk `fsync`** (Block 7), and **cache affinity** across CPU cores (Block 3).
  - *Block 2 & Block 6 (Memory Footprint & I/O Dominance):* Phase 2's back-of-the-envelope memory sizing and CPU-vs-I/O dominance connect directly to Linux `VmSize` vs. `VmRSS`, demand paging, blocking I/O sleep states, and OOM kills.
  - *Block 7 (Controlled Empirical Experiments):* Extends falsifiable, baseline-driven Node.js experimentation into safe, bounded Linux systems observation.

### 2.3 Prerequisite Knowledge Required Before Starting Phase 3
- **Phases 1–2 Foundations:** Falsifiable hypothesis testing, trade-off reasoning, asymptotic vs. hardware latency intuition, and running small Node.js scripts cleanly.
- **Basic Linux Terminal Literacy (Woven Inline Where Needed):** Running commands in bash, reading stdout/stderr, basic file paths. No prior Linux administration, kernel internals, or C programming mastery is assumed—any required C syntax (`int`, pointers as memory addresses, `pid_t`, `sizeof`, return value `-1` + `errno`) is explained inline right where introduced.
- **Basic Binary / Power-of-2 Intuition (Refreshed Inline in Block 4):** Hexadecimal notation (`0x...`) and powers of 2 (`2^12 = 4096 bytes = 4 KiB` page size) are introduced intuitively in Block 4 before page-table address splitting.

### 2.4 Downstream Dependency Map (Connecting Phases 1–2 → Phase 3 → Phases 4–6, 10–11, 23–26)

```text
Phase 1 (Mindset, Trade-offs, Scientific Method, Conceptual USE)
Phase 2 (Memory/Storage Latency Hierarchy, Spatial Locality, Empirical Rigor)
   │
   ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                     PHASE 3: OPERATING SYSTEMS (36 h)                        │
│  B1: Kernel/Syscalls   B2: Processes/Threads   B3: Scheduling/Context Switch │
│  B4: Virtual Mem/TLB   B5: Paging/Faults/OOM   B6: Files/Inodes/FDs          │
│  B7: Blocking/Non-Blocking I/O & Page Cache    B8: Signals & OS Diagnosis    │
└───────┬──────────┬──────────┬──────────┬──────────┬──────────┬───────────────┘
        │          │          │          │          │          │
        ▼          ▼          ▼          ▼          ▼          ▼
   Phase 4    Phase 5    Phase 6    Phase 10   Phase 11   Phases 23–26
   Networking JS Engine  Concurr.   Databases  Node.js    Linux/Docker/
   (Sockets   (V8 Heap/  (Threads/  (WAL/fsync Runtime    Perf/Distrib.
   as FDs,    GC inside  Locks/     Page Cache (libuv,    (systemd,
   Kernel     OS RSS/    Races/     vs Buffers epoll,     cgroups/NS,
   Buffers)   Virtual    Evented    Disk I/O,  Graceful   OOM limits,
              Memory)    Models)    B-Tree)    Shutdown)  perf/eBPF)
```

- **Phase 4 (*Computer Networking*):** Relies on Block 6's file descriptors (sockets are FDs subject to `RLIMIT_NOFILE`), Block 7's blocking/non-blocking I/O & kernel buffers, and Block 1's syscall boundary.
- **Phase 5 (*JavaScript Language Internals*):** Builds on Blocks 4–5 (`VmSize`, `VmRSS`, virtual address space, stack vs. heap vs. anonymous `mmap`) to explain how V8's managed JS heap and garbage collector sit *inside* the OS process memory model.
- **Phase 6 (*Concurrency and Asynchronous Systems*):** Directly inherits Block 2's process-vs-thread structural foundation, Block 3's preemptive scheduling/interleaving, and Block 7's non-blocking I/O & readiness-notification (`epoll`) concept to teach shared-state race conditions, mutexes, semaphores, deadlocks, and event-driven concurrency models (OSTEP Ch 26–34).
- **Phase 10 (*Database Engineering*):** Relies heavily on Block 5 (OS Page Cache, memory pressure, dirty pages), Block 6 (files/inodes), and Block 7 (`write()` buffering vs. `fsync()` durability barrier) to explain PostgreSQL `shared_buffers` vs. OS Page Cache, Write-Ahead Logging (WAL), and crash durability.
- **Phase 11 (*Node.js Runtime Engineering*):** Directly builds on Block 2 (threads inside a Node.js process), Block 6 (FDs & streams), Block 7 (`epoll` readiness vs. regular-file I/O thread-pool rationale), and Block 8 (`SIGTERM`/`SIGINT` graceful shutdown & exit codes) to teach V8 + libuv event-loop architecture and worker threads.
- **Phase 23 (*Linux and Backend Infrastructure*):** Expands Phase 3's OS mechanisms into production Linux administration: permissions/ACLs, users/groups, `systemd` services, filesystem mounting, reverse proxies, and host hardening.
- **Phase 24 (*Docker, Cloud, CI/CD and Deployment*):** Builds on Phase 3's processes, virtual memory, OOM killer (`Exit Code 137`), FDs, and signals (`SIGTERM` → `SIGKILL` container stop grace period) to explain Linux namespaces, `cgroups` CPU/memory quotas, and container isolation.
- **Phase 25 (*Performance Engineering*) & Phase 26 (*Distributed Systems*):** Phase 25 extends Phase 3's OS diagnostics into production `perf`/flame-graph profiling and queueing analysis; Phase 26 inherits OS partial-failure and distributed-storage context (OSTEP Ch 48–51).

---

## 3. Explicit Curriculum Scope Boundaries & Topic Triage

### 3.1 Anti-Overlap Boundary Matrix (Phase 3 vs. Later Phases)

| Later Phase | What Belongs in Phase 3 (Allowed Here) | What Must Remain Deferred (Explicitly Excluded Here) |
| :--- | :--- | :--- |
| **Phase 4: Computer Networking** | Recognizing that network sockets consume per-process **file descriptors** (`RLIMIT_NOFILE`) and are read/written via OS I/O syscalls (`Supporting` / `Context` mention). | TCP/UDP state machines, 3-way handshake, socket API programming (`socket`/`bind`/`listen`/`accept`), DNS, TLS, HTTP, packet loss, congestion control. |
| **Phase 5: JS Language Internals** | Using `process.memoryUsage()` as an OS-contextual view contrasting OS `rss` against high-level runtime memory buckets; OS stack vs. heap vs. `mmap`. | V8 heap generations (New/Old space), Scavenger/Mark-Sweep-Compact GC internals, hidden classes, JIT tiers, closures, prototypes, Promise microtasks. |
| **Phase 6: Concurrency & Async** | Introductory **structural distinction** between a process and a thread (shared address space/heap/FD table vs. private registers/PC/stack) + observing `/proc/<PID>/task`; **preemptive interleaving** conceptually; `Awareness` of why `epoll` exists. | OSTEP Ch 26–34: race-condition programming, critical sections, mutexes, spinlocks, condition variables, semaphores, deadlocks, lock-free data structures, event-loop design patterns, backpressure. |
| **Phase 10: Database Engineering** | OS **Page Cache** buffering vs. `fsync()` durability barrier (why `write()` returning does *not* guarantee data reached non-volatile disk); basic inode/file abstraction. | OSTEP Ch 40–45 (on-disk filesystem implementation, FFS, journaling/fsck, LFS, SSD FTL internals), PostgreSQL WAL architecture, MVCC, `shared_buffers` tuning, B-Tree pages, isolation levels. |
| **Phase 11: Node.js Runtime** | Contextual inspection of `process.pid`, `process.ppid`, `process.memoryUsage()`, stdio `.fd`, and basic `process.on('SIGTERM')` handling as bridges to Linux syscalls/signals. | libuv event-loop phases (`timers`, `poll`, `check`, `close`), libuv thread pool (`UV_THREADPOOL_SIZE`), Node.js Streams/Buffer backpressure internals, `worker_threads`, `cluster`, production graceful-drain HTTP frameworks. |
| **Phase 23: Linux & Infrastructure** | Safe unprivileged inspection commands (`ps`, `free`, `vmstat`, `strace` on child commands, `lsof`, `/proc/<PID>/*`, `taskset`) strictly to verify Phase 3 OS mechanisms. | Linux users/groups/chmod/chown/ACLs (OSTEP `39.16`), mounting/partitioning (`39.17`), `systemd` unit files, `journalctl`, SSH/firewalls, NGINX reverse proxies, `sysctl` production kernel tuning, package management. |
| **Phase 24: Docker, Cloud & CI/CD** | Understanding that containers run ordinary Linux processes governed by the host kernel's scheduler, virtual memory, signals (`SIGTERM` → `SIGKILL` / Exit `137`), and OOM killer (`Context` bridge). | Linux namespaces (`pid`, `net`, `mnt`), `cgroups v2` configuration, Dockerfiles, overlay filesystems, Kubernetes pods/probes, CI/CD pipelines, cloud VMs. |
| **Phase 25: Performance Engineering** | Conceptual CPU/memory/I/O bottleneck recognition using Phase 1's USE method + simple read-only Linux observables in Block 8. | Linux `perf`, eBPF/bpftrace, flame graphs, production load testing (`k6`/`autocannon`), NUMA/huge-page tuning, queueing theory. |
| **Phase 26: Distributed Systems** | Single-machine OS virtualization and local persistence foundations only. | OSTEP Ch 48–51 (Distributed Systems, NFS, AFS, RPC, distributed cache consistency, consensus/Paxos). |

### 3.2 Explicit OSTEP Version 1.10 Chapter Deferral & Exclusion Map

To maintain strict 36-hour discipline and honor curriculum phase boundaries, the following OSTEP v1.10 chapters are **explicitly excluded or deferred** from Phase 3 mandatory study:

| OSTEP v1.10 Chapters | Content | Phase 3 Disposition | Target Phase / Rationale |
| :--- | :--- | :--- | :--- |
| **Ch 9** (`cpu-sched-lottery.pdf`) | Lottery, Stride, & Linux CFS (`9.7`) | **Supplementary (CFS `9.7` optional)** | Core scheduling principles are covered in Ch 7, 8, 10; modern Linux uses EEVDF (6.6+), covered at `Awareness` in Block 3. |
| **Ch 16–17** (`vm-segmentation.pdf`, `vm-freespace.pdf`) | Segmentation & Free-Space Management | **Excluded / Supplementary** | Modern Linux/x86-64 relies on paging rather than segment-based relocation; user-space `malloc` heap allocator algorithms are unnecessary for Phase 3 goals. |
| **Ch 20** (`vm-smalltables.pdf`) | Advanced / Multi-Level Page Tables | **Supplementary** | The "linear page tables are too large" problem (`18.2`) and Linux 4-level page-table structure (`23.2`) provide the exact conceptual depth needed without bit-shifting drills. |
| **Ch 26–34** (`threads-*.pdf`) | Concurrency, Threads API, Locks, CVs, Semaphores, Concurrency Bugs, Event-Based Concurrency | **Explicitly Deferred** | Owned by **Phase 6 (Concurrency & Asynchronous Systems)** and **Phase 11 (Node.js Runtime)**. Phase 3 Block 2 covers only the structural process-vs-thread distinction. |
| **Ch 37–38** (`file-disks.pdf`, `file-raid.pdf`) | Hard Disk Drive Mechanics & RAID | **Explicitly Deferred / Excluded** | Rotating platter seek geometry and hardware RAID parity levels are outside Phase 3's backend-core priority (storage internals surface where relevant in Phases 10 & 23). |
| **Ch 40–45** (`file-implementation.pdf` – `file-integrity.pdf`) | Filesystem Implementation, FFS, FSCK/Journaling, LFS, Flash SSDs, Data Integrity | **Explicitly Deferred** | Owned by **Phase 10 (Database Engineering — storage & crash durability)** and **Phase 23 (Linux Infrastructure)**. |
| **Ch 48–51** (`dist-*.pdf`) | Distributed Systems, NFS, AFS, Distribution Summary | **Explicitly Deferred** | Owned by **Phase 26 (Distributed Systems Fundamentals)**. |
| **Security & VMM Appendices** (`vmm-intro.pdf`, `security-*.pdf`) | Virtual Machine Monitors & OS Security Chapters | **Explicitly Deferred** | Security owned by **Phases 18–19**; virtualization/containers context owned by **Phase 24**. |

### 3.3 Candidate Topic Evaluation & Triage Table

| Candidate Topic | Classification | Why It Matters to a Backend Engineer | Target Depth | Prerequisites | Verification Method |
| :--- | :---: | :--- | :---: | :--- | :--- |
| OS purpose & three abstractions (Virtualization, Concurrency, Persistence) | **Essential** | Frames what the OS mediates beneath every backend runtime | `Working Understanding` | Phases 1–2 | Map backend actions to OS abstractions |
| User space vs. kernel space, Limited Direct Execution & system calls | **Essential** | Explains privilege isolation & why syscalls cost far more than function calls | `Working Understanding` (mechanism) / `Practical Competence` (`strace` basics) | Block 1 | Trace & explain real syscalls with `strace` |
| Process abstraction, PCB, states (`R`/`S`/`D`/`T`/`Z`) & lifecycle | **Essential** | Core unit of backend deployment, monitoring, and crash isolation | `Working Understanding` | Block 1 | Inspect process states & diagnose zombie/blocked states |
| Process creation & termination (`fork`, `execve`, `wait`/`waitpid`, CoW, zombies/orphans) | **Essential** | Explains worker spawning, shells, PID exhaustion, and zombie leaks | `Working Understanding` | Block 2 | Run annotated C `fork`/`waitpid` lab & trace lifecycle |
| Processes vs. threads (introductory structural distinction) | **Essential (Intro)** | Clarifies isolation vs. shared address space before Phase 6 concurrency | `Working Understanding` (structural only) | Block 2 | Contrast shared vs. private state; inspect `/proc/PID/task` |
| Preemption, timer interrupts & context switching (direct + cache/TLB costs) | **Essential** | Explains how OS shares CPU & why excess thread/process switching hurts latency | `Working Understanding` | Blocks 1–2 | Measure voluntary vs. involuntary switches in `/proc` |
| CPU scheduling metrics (turnaround vs. response time), RR, MLFQ & multi-core affinity | **Essential** | Explains CPU starvation, convoy effect, interactive API tail latency, and `nice` | `Working Understanding` | Block 3 | Trace policy schedules & observe `taskset` contention |
| Linux `SCHED_OTHER` / CFS → EEVDF evolution | **Supporting / Context** | Keeps Linux mental model current on Kernel 6.6+/7.0 (Ubuntu 24.04) | `Awareness` | Block 3 | Distinguish scheduling goals from kernel implementation |
| Virtual address spaces, process isolation & memory layout (text/heap/`mmap`/stack) | **Essential** | Explains why processes cannot corrupt each other & what `VmSize` means | `Working Understanding` | Blocks 1–2 | Read `/proc/PID/maps` and classify memory segments |
| Address translation, Paging (`VPN`→`PFN`), Page Tables (`PTE` bits) & TLB | **Essential** | Explains hardware isolation, memory overhead & why spatial locality speeds translation | `Working Understanding` | Block 4 + Ph 2 B5 | Trace VPN→PFN lookups & analyze TLB hit/miss behavior |
| Multi-level page tables (concept) | **Essential (Concept)** | Resolves how 64-bit address spaces avoid gigabyte linear page tables | `Awareness` → light `Working Understanding` | Block 4 | Calculate linear table size problem & explain tree sparsity |
| Demand paging, lazy allocation, minor vs. major page faults & CoW faults | **Essential** | Explains `VmSize` vs. `VmRSS` divergence & sudden latency spikes on first touch/disk fault | `Working Understanding` | Block 4 | Observe 128 MiB allocate-vs-touch `VmRSS` & `minflt` jump |
| Swapping concepts, Page Cache vs. Anonymous Memory, Reclaim, Thrashing & OOM Killer | **Essential** | Directly diagnoses Linux `free -h`, swap death-spirals, and Exit Code `137` kills | `Working Understanding` (mechanism) / `Practical Competence` (diagnosis) | Block 5 | Interpret `free -h`, `/proc/PID/status`, `oom_score`, OOM scenario |
| Files, inodes, directories, hard links vs. symlinks, `unlink` on open files | **Essential** | Explains why deleting an open log file does not reclaim disk space until FD closes | `Working Understanding` | Block 1 | Reproduce unlinked-open `(deleted)` file in `/proc/PID/fd` |
| Standard streams (`0/1/2`), 3-tier kernel FD table, `fork`/`dup` & `EMFILE` FD limits | **Essential** | Explains backend log routing, pipe redirection, and socket/file FD-leak crashes | `Practical Competence` | Block 6 | Inspect `/proc/PID/fd`, trigger safe subshell `EMFILE`, fix leak |
| Device I/O basics (polling vs. interrupts vs. DMA) & Kernel Page Cache vs. `fsync` | **Essential** | Explains how CPU avoids spinning during I/O & why `write()` ≠ durable disk commit | `Working Understanding` | Blocks 1, 5, 6 | Time buffered `write()` vs. `fsync()` in `/tmp` |
| Blocking vs. Non-Blocking I/O (`O_NONBLOCK`, `EAGAIN`, short reads) & `epoll` readiness concept | **Essential** | Core OS foundation behind Node.js/NGINX handling thousands of concurrent I/O streams | `Working Understanding` (blocking/non-blocking) / `Awareness` (`epoll`) | Blocks 2, 3, 6, 7 | Observe blocking sleep vs. `EAGAIN` with `strace -tt -T` |
| Signals (`SIGTERM`, `SIGINT`, `SIGKILL`, `SIGHUP`, `SIGPIPE`, `SIGSEGV`), `kill(2)` & exit codes | **Essential** | Governs graceful server shutdown, container termination (`137`/`143`), and crash triage | `Practical Competence` | Blocks 2, 6, 7 | Handle `SIGTERM` vs. `SIGKILL` in Node.js & verify exit codes |
| Systematic backend OS diagnosis (USE + Scientific Method + `/proc`/`ps`/`vmstat`/`strace`) | **Essential** | Integrates all OS mechanisms into production incident triage | `Practical Competence` | All Phase 3 + Ph 1 B5 | Complete 4-Stage Phase 3 Capstone OS Diagnostic Lab |
| Concurrency locks/races, filesystem on-disk layout, TCP sockets, V8 GC, cgroups/systemd | **Outside / Deferred** | Dedicated to Phases 4, 5, 6, 10, 11, 23, 24 | — | — | Briefly referenced as `Supporting`/`Context` only |

---

## 4. Proposed Phase-Level Mastery Outcomes (9 Outcomes — For ChatGPT Approval)

*Note: `MASTER_ROADMAP.md` sets Phase 3's title (`Operating Systems`) and 36-hour allocation but does not yet list canonical numbered Phase 3 mastery outcomes. The 9 measurable outcomes below are proposed to mirror Phase 1's 10 canonical outcomes and Phase 2's 8 proposed outcomes, and await ChatGPT's review and approval.*

By the end of Phase 3, the learner should be able to:

1. **Outcome 1 (`Working Understanding`):** Explain the core purpose and abstractions of an operating system, why hardware enforces user space vs. kernel space privilege boundaries, and how **Limited Direct Execution** and **system calls** provide the controlled user-to-kernel entry mechanism through which unprivileged processes request privileged OS services safely at measurable overhead (distinct from hardware interrupts and exceptions).
2. **Outcome 2 (`Working Understanding`):** Trace the complete **process lifecycle** (creation via `fork()`/`execve()`, process states `Running`/`Ready`/`Blocked`/`Zombie`, waiting/reaping via `waitpid()`, and termination) and articulate the **introductory structural distinction between processes and threads** (isolated vs. shared resources) without conflating concurrency synchronization internals.
3. **Outcome 3 (`Working Understanding`):** Explain how **preemptive CPU scheduling** and **context switching** (timer interrupts, register/stack saves, cache/TLB side effects, turnaround vs. response time, Round Robin, MLFQ, and multi-core cache affinity) create CPU sharing and explain how run-queue saturation and CPU-bound work degrade backend tail latency.
4. **Outcome 4 (`Working Understanding`):** Explain **virtual address spaces** and **virtual-to-physical address translation** (memory layout, user-space allocator vs. `brk`/`mmap`, paging, page-table entries and protection bits, conceptual multi-level page tables, and **TLBs**), distinguishing Virtual Size (`VmSize`) from Resident Set Size (`VmRSS`).
5. **Outcome 5 (`Working Understanding`):** Explain **demand paging**, **minor vs. major page faults**, **Copy-on-Write (`CoW`)**, **swapping**, **Page Cache vs. anonymous memory**, **memory reclaim**, **thrashing**, and the **Linux OOM Killer**, and interpret real Linux memory telemetry (`free -h`, `/proc/<PID>/status`, `oom_score`, and contextual Node.js `process.memoryUsage()`) accurately.
6. **Outcome 6 (`Practical Competence`):** Trace the Linux **file, inode, directory, standard-stream, and 3-tier kernel file-descriptor model** (per-process FD table → system-wide open file description → inode), inspect open descriptors via `/proc/<PID>/fd` and `lsof`, and diagnose descriptor leaks (`EMFILE`) and unlinked-but-open disk-space traps.
7. **Outcome 7 (`Working Understanding`):** Explain OS-level I/O mechanisms—hardware **polling vs. interrupts vs. DMA**, **Kernel Page Cache buffering vs. `fsync()` durability**, **blocking vs. non-blocking (`O_NONBLOCK` / `EAGAIN`) execution**, short reads/writes—and explain at an `Awareness` level what problem OS readiness notification (`poll`/`epoll`) solves for high-concurrency backends.
8. **Outcome 8 (`Practical Competence`):** Explain Linux **signals** (`SIGINT`, `SIGTERM`, `SIGKILL`, `SIGHUP`, `SIGPIPE`, `SIGSEGV`, `SIGCHLD`), signal dispositions, why `SIGKILL`/`SIGSTOP` cannot be caught, and exit-status encoding (`128 + signum`, e.g., `137` and `143`), and implement/verify clean `SIGTERM` handling vs. forced termination in a Node.js process.
9. **Outcome 9 (`Practical Competence`):** Apply Phase 1's **Scientific Method** and **USE Method** alongside safe, unprivileged Linux inspection tools (`/proc/<PID>/*`, `ps`, `free`, `vmstat`, `strace` on child commands) to collect raw evidence and diagnose realistic backend OS incidents (CPU starvation, RSS growth/page faults/OOM, FD leaks, blocking I/O stalls, zombies, and abrupt signal kills).

---

## 5. Complete Block Sequence, Narrative Spine & Workload Summary (36 Hours)

### 5.1 Validation of Proposed Eight-Block Architecture

The proposed 8-block structure and 36-hour distribution (`3, 4, 4, 5, 5, 4, 5, 6 = 36 hours`) were evaluated against OSTEP v1.10 reading volumes, Linux documentation depth, and safe hands-on experiment effort. **Recommendation: Retain the exact 8 proposed blocks, titles, order, and hour allocations unchanged.**

Two key internal design refinements make this 8-block structure exceptionally cohesive:
1. **Clean Split of OSTEP Chapter 6 (*Limited Direct Execution*):** Assign Sections `6.1–6.2` (user/kernel privilege modes, trap table, system calls) to **Block 1** and Section `6.3` (timer interrupts, preemption, context switching) to **Block 3**. This prevents premature scheduling mechanics in Block 1 while grounding Block 3's scheduling policies in real kernel switching mechanics.
2. **Crisp Seam Between Block 4 and Block 5 (Paging):** Block 4 owns **virtual address spaces, static/in-RAM paging translation, page tables, and TLBs** (answering *"how does a virtual address become a physical address quickly and safely?"*), while Block 5 owns **dynamic demand paging, minor/major page faults, Copy-on-Write, swapping, reclaim, thrashing, and the OOM killer** (answering *"what happens when pages aren't yet in RAM or physical RAM runs out?"*). Similarly, **Block 6** owns POSIX files/inodes/FDs (`OSTEP 39.1–39.4, 39.6, 39.9–39.10, 39.14–39.15`), while **Block 7** moves `OSTEP 39.7 (fsync)` alongside `OSTEP Ch 36 (I/O Devices)` and Kernel `#page-cache` to unify buffered vs. durable I/O with blocking vs. non-blocking execution.

### 5.2 Phase 3 Block Overview Table

*The authoritative Phase 3 allocation remains **36 focused learner-study hours (`2,160 min`)** per `MASTER_ROADMAP.md`. All per-block durations are honest, flexible planning estimates—not strict deadlines.*

| Block | Exact Proposed Title | Roadmap Target | Primary Target Level | Outcomes Supported | Honest Required Core (No Opt. Assess.) | Optional Assessment | Honest Total (With Opt. Assess.) |
| :---: | :--- | :---: | :--- | :---: | :---: | :---: | :---: |
| **1** | The Operating System, Kernel, and System Calls | 3.0 h | `Working Understanding` | 1 | 145–180 min (2.4–3.0 h) | 20–25 min | 165–205 min (2.75–3.4 h) |
| **2** | Processes: Creation, Execution, and Lifecycle | 4.0 h | `Working Understanding` | 2 | 190–230 min (3.2–3.8 h) | 20–25 min | 210–255 min (3.5–4.25 h) |
| **3** | CPU Scheduling and Context Switching | 4.0 h | `Working Understanding` | 3 | 195–240 min (3.25–4.0 h) | 20–25 min | 215–265 min (3.6–4.4 h) |
| **4** | Virtual Address Spaces and Memory Translation | 5.0 h | `Working Understanding` | 4 | 235–290 min (3.9–4.8 h) | 25–30 min | 260–320 min (4.3–5.3 h) |
| **5** | Paging, Page Faults, and Memory Pressure | 5.0 h | `Working Understanding` | 5 | 225–275 min (3.75–4.6 h) | 25–30 min | 250–305 min (4.2–5.1 h) |
| **6** | Files, Directories, and File Descriptors | 4.0 h | `Practical Competence` | 6 | 190–225 min (3.2–3.75 h) | 20–25 min | 210–250 min (3.5–4.2 h) |
| **7** | I/O, Blocking, and Non-Blocking Execution | 5.0 h | `Working Understanding` | 7 | 220–270 min (3.7–4.5 h) | 25–30 min | 245–300 min (4.1–5.0 h) |
| **8** | Signals, Process Control, and Backend OS Diagnosis | 6.0 h | `Practical Competence` | 8, 9 (Integrative 1–9) | 280–340 min (4.7–5.7 h) | 30–35 min | 310–375 min (5.2–6.25 h) |
| **Total** | **8 Connected Phase 3 Blocks** | **36.0 h** | — | **All 9 Outcomes** | **1,680–2,050 min (28.0–34.2 h)** | **185–225 min (3.1–3.75 h)** | **1,865–2,275 min (31.1–37.9 h)** |

### 5.3 Causal Narrative Spine (Why Every Block Follows the Previous One)

1. **Block 1 → Block 2:** Block 1 establishes *why* user-space backend code cannot touch hardware directly and how it enters kernel space via system calls—raising the immediate next question: *"If the OS mediates everything for each running program, what exact unit does the kernel create, track, isolate, and clean up when we launch a server?"*
2. **Block 2 → Block 3:** Block 2 shows how Linux creates and transitions processes (`Ready`, `Running`, `Blocked`, `Zombie`) and introduces threads structurally—exposing the puzzle: *"We now have dozens of `Ready` processes and threads competing for 8 CPU cores; how does the kernel take the CPU back from a busy loop and decide who runs next without starving interactive requests?"*
3. **Block 3 → Block 4:** Block 3 explains preemptive CPU virtualization and context switching (and hints that switching processes hurts cache and translation speed)—leaving half the virtualization story unanswered: *"CPU sharing is meaningless if Process A can read or overwrite Process B's RAM; how does Linux give every process the illusion of its own private memory space?"*
4. **Block 4 → Block 5:** Block 4 reveals virtual address spaces, page tables, and TLBs mapping virtual pages to physical frames—triggering an urgent production question: *"What happens when a server allocates gigabytes of virtual memory before touching it, or when all processes combined exceed physical RAM—why do page faults, reclaim, thrashing, and the OOM killer happen?"*
5. **Block 5 → Block 6:** Having mastered CPU and Memory virtualization (OSTEP Piece 1), we step outside volatile RAM to **Persistence & External Resources** (OSTEP Piece 3): *"When a process needs to log, read config, or communicate beyond its own memory, how does Linux represent files, directories, and open streams safely per process?"*
6. **Block 6 → Block 7:** Block 6 equips the learner with file descriptors (`0, 1, 2, 3...`) and `open`/`read`/`write`/`close`—exposing the runtime execution puzzle: *"Disks and networks are thousands to millions of times slower than the CPU (Phase 2 Block 5); what happens to our process state when `read()` or `write()` cannot finish immediately, and how does non-blocking I/O avoid freezing the whole server?"*
7. **Block 7 → Block 8:** Now the learner understands every major OS subsystem a backend touches (syscalls, processes, scheduling, virtual memory/OOM, FDs, and I/O)—leaving the final operational question: *"How do humans and orchestrators control or cleanly shut down running processes via signals, and how do we synthesize all seven blocks to diagnose real production OS failures scientifically?"*

### 5.4 Workload Adequacy Assessment (Honest Alignment with the 36-Hour Allocation)
- **Verdict:** The approved **36-hour (`2,160 min`)** allocation in `MASTER_ROADMAP.md` is realistic and well-proportioned for foundational backend-oriented OS mastery. The honest required-core estimate across all 8 blocks is **`28.0–34.2 hours`** (midpoint **`~31.1 hours`**), providing **`~1.8–8.0 hours`** of natural buffer inside 36 hours for hands-on Linux command troubleshooting, reading annotated C snippets carefully, and deep note-taking.
- **Transparency on Optional Assessments:** All 8 closed-book assessments are strictly optional (`+3.1–3.75 h` combined if all eight are attempted), yielding a combined range of **`31.1–37.9 hours`** (midpoint **`~34.5 hours`**). Even with all 8 optional assessments included, the midpoint stays cleanly inside 36 hours, while the upper bound (`37.9 h`) exceeds 36 hours by `~1.9 h`—reported transparently under flexible, mastery-first pacing. **No roadmap hour change is authorized or requested.**

---

## 6. Detailed Block-by-Block Research Breakdown

---

### Block 1: The Operating System, Kernel, and System Calls *(Target: 3.0 hours)*

- **Central Question & Motivation:** *Why can't our backend code directly read disk sectors, grab all CPU cores, or peek at another server's memory—and what exact hardware-supported transition happens every time our program asks the Linux kernel for a service?* Backend engineers frequently underestimate syscall overhead (e.g., calling `write()` 1 byte at a time vs. buffered writes) or treat "kernel space" as an opaque black box. This block establishes the foundational privilege boundary that governs all of Phase 3.
- **Prerequisites & Dependencies:** Phase 1 Blocks 1, 4, 5 (abstractions, trade-offs, evidence-based verification); Phase 2 Blocks 1 & 5 (latency orders of magnitude, constant factors beyond Big-O).
- **Prepares For:** Block 2 (`fork`/`execve`/`wait` syscalls), Block 3 (hardware timer interrupts & kernel entry during preemption), Block 4 (privileged page-table changes), Block 6–7 (I/O syscalls & `strace`), Phase 4 (socket syscalls), Phase 23 (Linux kernel/user-space diagnosis).
- **Primary Target Level:** `Working Understanding` | **Phase 3 Outcomes Supported:** Outcome 1.

#### Learning Objectives
- **LO 1.1 (`Working Understanding` | Bloom: *Understand*):** Explain the core purpose of an operating system as a **resource manager and abstraction provider** across OSTEP's three pillars—**Virtualization** (CPU & memory), **Concurrency**, and **Persistence**—linking each pillar to concrete backend behavior and explicitly identifying which pillars are deepened in later phases.
- **LO 1.2 (`Working Understanding` | Bloom: *Analyze*):** Contrast **user mode (user space)** and **kernel mode (kernel space)** as hardware-enforced privilege boundaries, explaining why unprivileged applications cannot execute restricted hardware instructions or access arbitrary physical memory directly.
- **LO 1.3 (`Working Understanding` | Bloom: *Analyze*):** Trace **Limited Direct Execution (LDE)** and a **system call** end-to-end (library wrapper → trap instruction → privilege elevation → kernel trap table lookup & handler validation → return-from-trap back to user mode), contrasting syscall overhead against an ordinary user-space function call.
- **LO 1.4 (`Practical Competence` | Bloom: *Apply*):** Use `strace` safely on a child process (`strace -c` and `strace -e trace=...` without `sudo`) to observe real user-to-kernel system-call transitions made by a Linux command and a minimal Node.js script.

#### Concepts, Mechanism Depth & Terminology Inventory *(Internal Authoring Reference)*
| Concept / Term | Official Target Level | Mechanism Depth & Why It Matters in Phase 3 |
| :--- | :---: | :--- |
| Operating System Abstractions (Virtualization, Concurrency, Persistence) | `Working Understanding` | Unifying mental map separating CPU/memory sharing, concurrent execution (Phase 6), and durable I/O. |
| User Space vs. Kernel Space (Privilege Modes) | `Working Understanding` | Hardware privilege rings ensure one buggy or malicious backend process cannot crash the host or read another process's secrets. |
| Limited Direct Execution (LDE) | `Working Understanding` | Explains how OS achieves near-native CPU speed while retaining absolute control over restricted hardware ops. |
| System Call (Syscall) & Trap Table | `Working Understanding` | Controlled hardware-enforced entry points (`trap` / `syscall` instruction) into kernel handlers, distinct from ordinary function calls. |
| Library Wrapper (`libc` / Node.js) vs. Raw Syscall | `Working Understanding` | Explains why high-level code (`fs.readFileSync`, `console.log`, `printf`) ultimately invokes raw Linux syscalls like `openat`, `read`, `write`. |
| System-Call Tracing (`strace` on child commands) | `Practical Competence` | Primary empirical lens for seeing exact kernel interactions of any backend executable safely without root. |

#### Deferred Topics & Explicit Phase Boundaries
- **Explicitly Deferred:** Timer interrupts, preemption, and context switching (`OSTEP 6.3` — deferred to **Block 3**); x86-64 ring-0/ring-3 assembly calling conventions & vDSO implementation internals; kernel module programming; `ptrace` attachment to running PIDs under `yama.ptrace_scope` restrictions (noted in Section 8 safety guardrails); concurrency inside the kernel (`OSTEP 6.4` — deferred to **Phase 6**); Meltdown/Spectre hardware side-channels (security/hardware internals outside Phase 3).

#### Authoritative Resources & Assigned Scope *(Live-Verified `2026-11-28`)*
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | OSTEP v1.10 — **Chapter 2: Introduction to Operating Systems** | [`intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/intro.pdf) | Sections `2.1` (*Virtualizing The CPU*), `2.2` (*Virtualizing Memory*), `2.3` (*Concurrency*), `2.4` (*Persistence*), `2.5` (*Design Goals*), `2.7` (*Summary*) — ~11 pages *(skip `2.6 Some History`)* | ~25–30 min | Big-picture mental model of what an OS does and why virtualization, concurrency, and persistence are the core themes. |
| **Mandatory 2** | Free | OSTEP v1.10 — **Chapter 6: Mechanism: Limited Direct Execution** | [`cpu-mechanisms.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-mechanisms.pdf) | Sections `6.1` (*Basic Technique: Limited Direct Execution*), `6.2` (*Problem #1: Restricted Operations*), `6.5` (*Summary* — LDE part only) — ~6.5 pages *(defer `6.3` to Block 3; skip `6.4`)* | ~20–25 min | Step-by-step protocol of user mode vs. kernel mode, trap tables, and system calls. |
| **Mandatory 3** | Free | Linux `man-pages` — `syscalls(2)` & `strace(1)` | [`syscalls(2)`](https://man7.org/linux/man-pages/man2/syscalls.2.html), [`strace(1)`](https://man7.org/linux/man-pages/man1/strace.1.html) | `syscalls(2)`: *Description* (first 4 paragraphs — syscall vs. glibc wrapper, `errno`; skip full syscall table). `strace(1)`: *Description* + `-c`, `-e trace=...` flags only | ~10–15 min | Real Linux distinction between C library wrappers and kernel syscalls, plus safe `strace` invocation syntax. |
| **Supplementary** | Free | OSTEP v1.10 — Chapter 2.6 (*Some History*) | [`intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/intro.pdf) | Section `2.6` (pp. 12–17) | Optional | Historical evolution from batch systems to multiprogramming and Unix. |

#### Required Practical Experiment & Transfer Exercise
- **Practical Experiment 1 (`LO 1.3`, `LO 1.4` — ~20–25 min): Observing the User/Kernel Boundary Live with `strace`.** Safely run unprivileged `strace` on child commands: (a) `strace -c /bin/echo "hello"` vs. `strace -c node -e "console.log('hello')"` to compare syscall counts and see how much kernel setup (`openat`, `mmap`, `brk`, `read`, `write`) a high-level runtime performs before printing a single line; (b) trace a tiny Node.js loop that writes 1,000 small lines via unbuffered `fs.writeSync` vs. 1 batched `fs.writeSync` to `/tmp` and compare total `write` syscall counts and `strace -c` kernel time.
- **Transfer Exercise 1 (`LO 1.1`, `LO 1.2`, `LO 1.3` — ~25–30 min): Privilege Boundary & Syscall Cost Reasoning.** Given three backend scenarios—(a) a junior engineer asks *"Why can't our Node.js process read an SSD block directly from JavaScript to skip the OS?"*, (b) a logging middleware calls `write()` once per character instead of buffering a full JSON log line, and (c) an AI-generated explanation claims *"A system call is just a regular C function call into `libc`"*—diagnose each misconception mechanically using LDE, privilege transitions, and syscall overhead.
- **Assessment Approach:** 3 inline Pause & Checks (deferred Self-Check Guidance) + Optional Closed-Book Assessment (`~20–25 min`, strictly optional).

#### Flexible Study-Time Breakdown (Block 1)
- Guided Lesson + Pause & Checks 1–3: `~45–55 min` *(flexible priority)*
- Mandatory 1 (OSTEP Ch 2 `2.1–2.5, 2.7`): `~25–30 min`
- Mandatory 2 (OSTEP Ch 6 `6.1–6.2, 6.5`): `~20–25 min`
- Mandatory 3 (`syscalls(2)` & `strace(1)` targeted): `~10–15 min`
- Practical Experiment 1: `~20–25 min` | Transfer Exercise 1: `~25–30 min`
- **Total Required Core (without optional assessment):** **`~145–180 min (~2.4–3.0 h)`**
- Optional Closed-Book Assessment: `~20–25 min` *(optional)*
- **Total (with optional assessment):** **`~165–205 min (~2.75–3.4 h)`** *(Roadmap target: `3.0 h`)*

---

### Block 2: Processes: Creation, Execution, and Lifecycle *(Target: 4.0 hours)*

- **Central Question & Motivation:** *When we run `node server.mjs` or when PostgreSQL spawns a worker backend, what exact state does the Linux kernel create and track as a "process," how do `fork()`, `execve()`, and `waitpid()` govern its birth and cleanup, and how does a process structurally differ from a thread?* Understanding process lifecycles prevents production zombie-PID leaks, clarifies parent/child process trees, and builds the exact structural footing needed before Phase 6 concurrency.
- **Prerequisites & Dependencies:** Block 1 (kernel space, system calls, `strace`).
- **Prepares For:** Block 3 (scheduling `Ready` processes/threads), Block 4 (each process's private virtual address space & CoW), Block 6 (FD inheritance across `fork`/`exec`), Block 8 (signals, `SIGCHLD`, process exit status), Phase 6 (multi-threaded concurrency & shared-state races), Phase 11 (Node.js child processes & worker threads), Phase 24 (PID 1 zombie reaping inside Docker containers!).
- **Primary Target Level:** `Working Understanding` | **Phase 3 Outcomes Supported:** Outcome 2.

#### Learning Objectives
- **LO 2.1 (`Working Understanding` | Bloom: *Understand*):** Define a **process** precisely as a running program's kernel-managed execution state (address space, CPU registers/PC/stack pointer, open file descriptors, PID/PPID, metadata stored in the kernel's Process Control Block / Linux `task_struct` conceptually) rather than an executable file on disk.
- **LO 2.2 (`Working Understanding` | Bloom: *Analyze*):** Trace the Unix process creation and lifecycle API—**`fork()`**, **`execve()`**, **`wait()` / `waitpid()`**, and process exit—explaining *why* Unix separates `fork()` from `execve()` (enabling shell I/O redirection and environment setup) and conceptual **Copy-on-Write (`CoW`)** optimization.
- **LO 2.3 (`Working Understanding` | Bloom: *Analyze*):** Trace **process state transitions** (OSTEP's conceptual `Running`, `Ready`, `Blocked`/`Sleeping`, `Stopped`, `Zombie`, and `Terminated` states alongside Linux `ps` / `/proc/<PID>/status` state codes `R`, `S`, `D`, `T`, `Z`—explicitly clarifying that Linux state **`R` represents both running and runnable**, rather than providing separate status letters for OSTEP's `Running` vs. `Ready`) and explain how un-reaped **zombie processes** and **orphan processes** arise and why zombie accumulation threatens long-running backend process supervisors.
- **LO 2.4 (`Working Understanding` | Bloom: *Analyze*):** Distinguish **processes vs. threads at an introductory structural level**—specifying what threads within one process share (virtual address space, heap, global/static data, open file-descriptor table, and shared process ID / Linux **Thread Group ID `TGID`**, reported in user space as the process `PID`) vs. what each thread owns privately (its own Linux **Thread ID `TID`**, registers, program counter, and stack)—while keeping concurrency synchronization internals explicitly deferred to Phase 6.

#### Concepts, Mechanism Depth & Terminology Inventory *(Internal Authoring Reference)*
| Concept / Term | Official Target Level | Mechanism Depth & Why It Matters in Phase 3 |
| :--- | :---: | :--- |
| Process Abstraction & Machine State (PCB / `task_struct`) | `Working Understanding` | Separates static binary on disk from live, isolated kernel-managed execution context. |
| Process States (`R`, `S`, `D`, `T`, `Z`) | `Working Understanding` | Enables reading `ps` and `/proc/<PID>/status` accurately—noting Linux `R` means running or runnable (combining OSTEP `Running` and `Ready`), while `S`/`D` show interruptible vs. uninterruptible sleep and `Z` shows zombies. |
| `fork()`, `execve()`, `wait()` / `waitpid()` | `Working Understanding` | Fundamental Unix process creation, program replacement, and exit-status reaping syscalls. |
| Copy-on-Write (`CoW`) — Conceptual Intro | `Working Understanding` *(deepened in Block 5)* | Explains why `fork()` does not eagerly duplicate gigabytes of parent RAM immediately. |
| Zombie Process (`Z`) vs. Orphan Process | `Working Understanding` | Explains why a dead child lingers in the process table until its parent calls `wait()`/`waitpid()`, and how `init`/subreaper adopts orphans. |
| Process vs. Thread (Structural Distinction; `TGID`/`PID` vs. `TID`) | `Working Understanding` *(introductory structural level only)* | Establishes shared address space/FDs/process `PID` (`TGID`) vs. private stack/registers/per-thread `TID`—explaining why a Node.js process (`1` `PID`/`TGID`) exposes multiple individual thread `TID`s under `/proc/<PID>/task/<TID>`. |

#### Deferred Topics & Explicit Phase Boundaries
- **Explicitly Deferred:** OSTEP Chapters 26–34 (multi-threaded race conditions, atomicity, mutexes, locks, condition variables, semaphores, deadlocks — **Phase 6**); Linux `clone(2)` flag bitmasks and `futex(2)` internals; Node.js `child_process` / `worker_threads` / `cluster` production patterns (**Phase 11**); PID namespaces and container PID 1 zombie-reaping configuration (`tini`/`--init` — **Phase 24**, mentioned only as a 1-sentence `Context` forward hook).

#### Authoritative Resources & Assigned Scope *(Live-Verified `2026-11-28`)*
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | OSTEP v1.10 — **Chapter 4: The Abstraction: The Process** | [`cpu-intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf) | Sections `4.1`–`4.6` (*The Abstraction*, *Process API*, *Process Creation*, *Process States*, *Data Structures*, *Summary*) — ~9 pages | ~25–30 min | Complete conceptual model of process machine state, states (`Running`/`Ready`/`Blocked`), and process list/PCB. |
| **Mandatory 2** | Free | OSTEP v1.10 — **Chapter 5: Interlude: Process API** | [`cpu-api.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf) | Sections `5.1`–`5.7` (`fork`, `wait`, `exec`, *Why? Motivating The API*, *Process Control And Users*, *Useful Tools*, *Summary*) — ~10 pages | ~25–30 min | Exact behavior of `fork()` returning twice, `wait()` ordering, `exec()` image replacement, and shell redirection rationale. |
| **Mandatory 3** | Free | Linux `man-pages` — `fork(2)`, `execve(2)`, `wait(2)` / `waitpid(2)` + Node.js v26 Docs (`process.pid`, `process.ppid`) | [`fork(2)`](https://man7.org/linux/man-pages/man2/fork.2.html), [`execve(2)`](https://man7.org/linux/man-pages/man2/execve.2.html), [`wait(2)`](https://man7.org/linux/man-pages/man2/wait.2.html), [`process.pid`](https://nodejs.org/docs/latest-v26.x/api/process.html#processpid), [`process.ppid`](https://nodejs.org/docs/latest-v26.x/api/process.html#processppid) | Man-pages: *Description* (core behavior, CoW note in `fork(2)`, zombie/reaping note in `wait(2)`) & *Return Value* only *(skip full error/history lists)*. Node.js v26: brief `process.pid` & `process.ppid` entries | ~20–25 min | Authoritative Linux semantics (`waitpid` renders `wait(2)` on `man7.org`), CoW reality, and how Node.js exposes OS PID/PPID. |
| **Supplementary** | Free | Linux `man-pages` — `proc_pid_status(5)` | [`proc_pid_status(5)`](https://man7.org/linux/man-pages/man5/proc_pid_status.5.html) | `State`, `Pid`, `PPid`, `Threads` fields only | Optional | Exact definitions of `/proc/<PID>/status` process & thread count fields. |

#### Required Practical Experiment & Transfer Exercise
- **Practical Experiment 2 (`LO 2.1`–`LO 2.4` — ~35–45 min): Observing `fork()`, `waitpid()`, Zombies, and Process Threads on Ubuntu 24.04.**
  1. Compile and run a minimal ~30-line **fully annotated C program** (every C line explained inline in Bangla—`pid_t`, `fork()`, `sleep()`, `waitpid()`, `_exit()`) that forks one child which exits immediately while the parent sleeps for 5 seconds *before* calling `waitpid()`. In a second terminal (or backgrounded subshell), inspect `ps -o pid,ppid,stat,cmd` to witness the child in state `Z+` (`<defunct>` zombie) disappear cleanly the instant the parent reaps it!
  2. Run a minimal 10-second Node.js script printing `process.pid` and `process.ppid`, and inspect `/proc/<PID>/status` (`State:`, `Tgid:`, `Pid:`, `PPid:`, `Threads:`) and `ls /proc/<PID>/task/` to observe empirically that even a "single-threaded JavaScript" Node.js process shares one process `PID` (`TGID`) while containing multiple individual OS thread `TID`s inside one shared virtual address space!
- **Transfer Exercise 2 (`LO 2.2`, `LO 2.3`, `LO 2.4` — ~30–35 min): Process Lifecycle & Process-vs-Thread Architecture Audit.** Diagnose three scenarios: (a) a Node.js PDF-generator service spawns short-lived child helpers but forgets to consume exit status, eventually failing with `EAGAIN` / PID table exhaustion despite low RAM usage; (b) explaining why `fork()` followed by `execve()` allows `cmd > out.txt` without modifying `cmd`'s source code; and (c) comparing a multi-process worker pool vs. a multi-threaded worker pool strictly on **isolation (crash blast radius & memory separation)** vs. **creation/sharing trade-offs** (without touching locks/mutexes).
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~20–25 min`).

#### Flexible Study-Time Breakdown (Block 2)
- Guided Lesson + Pause & Checks 1–3: `~55–65 min` *(flexible priority)*
- Mandatory 1 (OSTEP Ch 4 `4.1–4.6`): `~25–30 min`
- Mandatory 2 (OSTEP Ch 5 `5.1–5.7`): `~25–30 min`
- Mandatory 3 (`fork(2)`/`execve(2)`/`wait(2)` targeted + Node.js `pid`/`ppid`): `~20–25 min`
- Practical Experiment 2: `~35–45 min` | Transfer Exercise 2: `~30–35 min`
- **Total Required Core (without optional assessment):** **`~190–230 min (~3.2–3.8 h)`**
- Optional Closed-Book Assessment: `~20–25 min` *(optional)*
- **Total (with optional assessment):** **`~210–255 min (~3.5–4.25 h)`** *(Roadmap target: `4.0 h`)*

---

### Block 3: CPU Scheduling and Context Switching *(Target: 4.0 hours)*

- **Central Question & Motivation:** *How does Linux take the CPU back from a runaway loop to share cores across dozens of processes, what hidden costs occur during every context switch, and why do CPU-bound workloads starve interactive backend requests and blow up `p99` latency?* Backend engineers routinely misread "average CPU 60%" as healthy while run-queue saturation and involuntary preemption silently destroy tail latency.
- **Prerequisites & Dependencies:** Block 1 (LDE, kernel mode); Block 2 (process states `Ready`/`Running`/`Blocked`, PCB); Phase 1 Block 5 (USE method: Utilization vs. Saturation; mean vs. tail latency); Phase 2 Block 5 (CPU caches & locality).
- **Prepares For:** Block 4 (TLB flush/ASID cost during context switches), Block 7 (yielding CPU when blocking on I/O), Block 8 (CPU saturation diagnosis with `vmstat` & `/proc`), Phase 6 (uncontrolled preemptive interleaving causing race conditions!), Phase 11 (CPU-bound code blocking Node.js), Phase 24 (`cgroups` CPU quotas/throttling), Phase 25 (CPU profiling & queueing latency).
- **Primary Target Level:** `Working Understanding` | **Phase 3 Outcomes Supported:** Outcome 3.

#### Learning Objectives
- **LO 3.1 (`Working Understanding` | Bloom: *Analyze*):** Trace **preemptive multitasking** via hardware **timer interrupts** and step-by-step **context switching** (saving/restoring general registers, PC, and kernel stack pointers), distinguishing **direct switch cost** (illustratively on the order of microseconds of kernel state-saving work, varying by CPU, kernel, and hardware mitigations—not a fixed universal constant) from **indirect cost** (subsequent CPU cache and TLB misses as the incoming task warms its working set).
- **LO 3.2 (`Working Understanding` | Bloom: *Analyze*):** Distinguish **turnaround time** (`T_completion - T_arrival`) from **response time** (`T_first_run - T_arrival`) and trace fundamental scheduling policies (**FIFO** and the **convoy effect**, **SJF**, **STCF**, **Round Robin** time-slice trade-offs, and overlapping CPU with **I/O**) to explain why throughput and responsiveness are fundamentally in tension.
- **LO 3.3 (`Working Understanding` | Bloom: *Evaluate*):** Explain how the **Multi-Level Feedback Queue (MLFQ)** uses past behavior to favor short/interactive I/O-bound jobs over long CPU-bound hogs without knowing job lengths in advance, and why **priority boosts** and **better accounting** are required to prevent **starvation** and scheduler gaming.
- **LO 3.4 (`Working Understanding` | Bloom: *Analyze*):** Explain key **multi-core scheduling** challenges (**cache affinity**, single-queue contention vs. multi-queue load balancing / work migration) and understand Linux's default time-sharing policy (`SCHED_OTHER`/`SCHED_NORMAL` + `nice` values in `sched(7)`), with `Awareness` of Linux's evolution from CFS to **EEVDF** (Kernel 6.6+, active on Ubuntu 24.04).

#### Concepts, Mechanism Depth & Terminology Inventory *(Internal Authoring Reference)*
| Concept / Term | Official Target Level | Mechanism Depth & Why It Matters in Phase 3 |
| :--- | :---: | :--- |
| Timer Interrupt & Preemptive Multitasking | `Working Understanding` | Prevents a single CPU-bound infinite loop from monopolizing a core forever. |
| Context Switch (Voluntary vs. Involuntary; Direct vs. Indirect Cost) | `Working Understanding` | Voluntary = process sleeps for I/O; involuntary = kernel preempts CPU hog; excessive switching degrades cache warmth. |
| Turnaround Time vs. Response Time | `Working Understanding` | Batch throughput vs. interactive API latency—core scheduling trade-off. |
| Convoy Effect & Time-Slice (Quantum) Trade-Off | `Working Understanding` | Too-long slice hurts API response time; too-short slice wastes CPU on context-switch overhead. |
| Multi-Level Feedback Queue (MLFQ) & Starvation | `Working Understanding` | Teaches how real schedulers learn workload behavior dynamically while protecting against starvation. |
| Multi-Core Cache Affinity & Load Balancing | `Working Understanding` | Explains why bouncing a process across cores invalidates L1/L2 caches and why multi-queue schedulers exist. |
| Linux `SCHED_OTHER` (`SCHED_NORMAL`), `nice` Values, CFS → EEVDF | `Working Understanding` (`SCHED_OTHER`/`nice`) / `Awareness` (CFS/EEVDF internals) | Grounds theory in real Linux scheduling behavior on modern Ubuntu 24.04 kernels (Kernel 6.6+/7.0). |
| Run Queue & CPU Saturation | `Working Understanding` | Directly connects OS scheduler queues to Phase 1 Block 5's USE method (`r` column in `vmstat`). |

#### Deferred Topics & Explicit Phase Boundaries
- **Explicitly Deferred:** OSTEP `10.2` detailed mutex/lock synchronization code (**Phase 6**); mathematical derivations of Lottery/Stride scheduling (`OSTEP Ch 9`) or EEVDF virtual-deadline red-black tree internals; Linux real-time scheduling (`SCHED_FIFO`/`SCHED_RR`/`SCHED_DEADLINE` — `Awareness` mention only); `cgroups` CFS/EEVDF CPU quota throttling in containers (**Phase 24**); CPU flame graphs (`perf` — **Phase 25**).

#### Authoritative Resources & Assigned Scope *(Live-Verified `2026-11-28`)*
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | OSTEP v1.10 — **Ch 6.3** (*Switching Between Processes*) & **Ch 7: Scheduling: Introduction** | [`cpu-mechanisms.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-mechanisms.pdf), [`cpu-sched.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched.pdf) | Ch `6.3` (pp. 7–11) + Ch `7.1`–`7.10` (pp. 1–11) — ~14.5 pages total | ~35–45 min | Timer interrupt & context-switch protocol + FIFO convoy effect, SJF/STCF, Round Robin time-slice trade-off, and I/O overlap (`7.8`). |
| **Mandatory 2** | Free | OSTEP v1.10 — **Ch 8: Scheduling: The Multi-Level Feedback Queue** | [`cpu-sched-mlfq.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-mlfq.pdf) | Sections `8.1`–`8.4` and `8.6` (*Basic Rules*, *How To Change Priority*, *The Priority Boost*, *Better Accounting*, *Summary*) — ~7 pages *(skip `8.5` detailed tuning knobs)* | ~20–25 min | How a real scheduler rewards I/O-bound interactive tasks, prevents starvation via priority boost, and stops gaming. |
| **Mandatory 3** | Free | OSTEP v1.10 — **Ch 10: Multiprocessor Scheduling** | [`cpu-sched-multi.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-multi.pdf) | OSTEP `10.1`, `10.3`–`10.5`, brief skim `10.6` (~6 pp; *skip `10.2` lock internals*) | ~15–20 min | Cache affinity and SQMS vs. MQMS multi-core load-balancing trade-offs. |
| **Mandatory 4** | Free | Linux `man-pages` — `sched(7)` | [`sched(7)`](https://man7.org/linux/man-pages/man7/sched.7.html) | *Scheduling policies*, *`SCHED_OTHER`*, and *The nice value* only | ~10 min | Real Linux `SCHED_OTHER` (`SCHED_NORMAL`) and `nice` priority semantics. |
| **Supplementary** | Free | OSTEP v1.10 — Ch 9.7 (*The Linux CFS*) & Linux Kernel EEVDF Docs | [`cpu-sched-lottery.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-lottery.pdf), [`sched-eevdf.html`](https://docs.kernel.org/scheduler/sched-eevdf.html) | OSTEP `9.7` (pp. 7–11) or high-level skim of `sched-eevdf.html` overview | Optional | Deeper historical/modern Linux fair-scheduler context (CFS → EEVDF). |

#### Required Practical Experiment & Transfer Exercise
- **Practical Experiment 3 (`LO 3.1`, `LO 3.2`, `LO 3.4` — ~30–40 min): Controlled Single-Core CPU Contention & Context-Switch Observation.** On an 8-core/16-thread Ryzen 7 7700, running 2 CPU jobs across all cores hides contention; therefore, pin two bounded 6-second Node.js scripts strictly to **CPU core 0** via unprivileged `taskset -c 0`: (a) one lightweight timer-probe script measuring actual wake-up jitter of a 10 ms timer alone vs. alongside 2 CPU-spinning workers pinned to core 0, while recording `voluntary_ctxt_switches` vs. `nonvoluntary_ctxt_switches` from `/proc/<PID>/status` and watching `vmstat 1 6` (`r` runnable queue and `cs` context switches). Pinning the test workers to core 0 keeps the remaining hardware threads free of experiment-generated contention so normal desktop/OS activity stays responsive (subject to ordinary background host load, not guaranteed 100% idle).
- **Transfer Exercise 3 (`LO 3.2`, `LO 3.3`, `LO 3.4` — ~30–35 min): Scheduling Policy Trace & Backend Starvation Diagnosis.** (1) Hand-trace 3 jobs (`A`: 120 ms CPU, `B`: 10 ms CPU, `C`: 10 ms CPU arriving at `t=0`) under FIFO vs. Round Robin (`q = 10 ms`), computing exact turnaround time and response time for each and connecting the convoy effect to an API server sharing a core with a heavy report-export worker; (2) explain why running `nice -n 19` on a background CSV-compression job improves interactive API `p99` response time without lowering total throughput.
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~20–25 min`).

#### Flexible Study-Time Breakdown (Block 3)
- Guided Lesson + Pause & Checks 1–3: `~55–65 min` *(flexible priority)*
- Mandatory 1 (OSTEP Ch `6.3` + Ch `7.1–7.10`): `~35–45 min`
- Mandatory 2 (OSTEP Ch `8.1–8.4, 8.6`): `~20–25 min`
- Mandatory 3 (OSTEP Ch `10.1, 10.3–10.6`): `~15–20 min`
- Mandatory 4 (Linux `sched(7)` targeted): `~10 min`
- Practical Experiment 3: `~30–40 min` | Transfer Exercise 3: `~30–35 min`
- **Total Required Core (without optional assessment):** **`~195–240 min (~3.25–4.0 h)`**
- Optional Closed-Book Assessment: `~20–25 min` *(optional)*
- **Total (with optional assessment):** **`~215–265 min (~3.6–4.4 h)`** *(Roadmap target: `4.0 h`)*

---

### Block 4: Virtual Address Spaces and Memory Translation *(Target: 5.0 hours)*

- **Central Question & Motivation:** *Why can two running backend processes both hold pointers to the exact same hexadecimal address (`0x55a...`) without corrupting each other's data, and what hardware/OS translation layers (Page Tables and the TLB) sit beneath every single memory access?* Without this mental model, engineers misread `VmSize` as physical RAM consumption, cannot interpret `/proc/<PID>/maps` or `SEGFAULT`s, and do not understand why spatial locality accelerates address translation itself.
- **Prerequisites & Dependencies:** Block 1 (user/kernel privilege); Block 2 (per-process address space); Block 3 (context switches); Phase 2 Block 5 (CPU cache hierarchy & spatial locality).
- **Prepares For:** Block 5 (demand paging, page faults, CoW, swap, reclaim & OOM), Block 8 (memory incident diagnosis), Phase 5 (V8 managed heap & GC inside virtual memory), Phase 10 (PostgreSQL shared memory & buffers), Phase 24 (container memory boundaries).
- **Primary Target Level:** `Working Understanding` | **Phase 3 Outcomes Supported:** Outcome 4.

#### Learning Objectives
- **LO 4.1 (`Working Understanding` | Bloom: *Understand*):** Explain the three core goals of **virtual memory** (**transparency**, **efficiency**, and **protection/isolation**) and map the regions of a Linux process's **virtual address space** (code/text, data/BSS, heap, `mmap`/shared-library/anonymous region, stack, and kernel-space boundary), contrasting user-space allocators (`malloc`/`free`) with OS memory syscalls (`brk`, `mmap(2)`).
- **LO 4.2 (`Working Understanding` | Bloom: *Apply / Analyze*):** Trace **hardware-assisted address translation** from the conceptual intuition of dynamic **base-and-bounds** relocation (and its external fragmentation limit) to **fixed-size Paging** (splitting a virtual address into **Virtual Page Number `VPN`** + **offset**, indexing the **Page Table** to retrieve a **Physical Frame Number `PFN`** and permission/present/dirty/accessed bits).
- **LO 4.3 (`Working Understanding` | Bloom: *Analyze*):** Explain why a flat linear page table is impossibly large for 64-bit address spaces (establishing conceptual `Working Understanding` / `Awareness` of **multi-level hierarchical page tables** as sparse trees) and why every uncached page-table lookup doubles memory accesses—motivating the **TLB (Translation Lookaside Buffer)**.
- **LO 4.4 (`Working Understanding` | Bloom: *Analyze*):** Trace **TLB hits vs. TLB misses**, explain how **spatial and temporal locality** (connecting back to Phase 2 Block 5!) govern TLB hit rates, explain why **context switches** affect TLB entries (flush vs. ASID/PCID), and distinguish **Virtual Size (`VmSize`)** from **Resident Set Size (`VmRSS`)** when inspecting real Linux and Node.js processes.

#### Concepts, Mechanism Depth & Terminology Inventory *(Internal Authoring Reference)*
| Concept / Term | Official Target Level | Mechanism Depth & Why It Matters in Phase 3 |
| :--- | :---: | :--- |
| Virtual Address Space & Process Isolation | `Working Understanding` | Every process sees a private address map; invalid/unauthorized accesses trigger a hardware trap (`SIGSEGV`). |
| Process Memory Layout (Text, Heap, `mmap`, Stack) | `Working Understanding` | Directly readable in `/proc/<PID>/maps` with `r-xp`, `rw-p`, `[heap]`, `[stack]` permissions. |
| User-Space Allocator (`malloc`/`free`) vs. OS (`brk`/`mmap(2)`) | `Working Understanding` | Clarifies why calling `free()` or JS GC doesn't always shrink OS `VmRSS` immediately. |
| Base-and-Bounds Relocation (Intuition) | `Working Understanding` | Pedagogical bridge showing hardware MMU translation + bounds protection before paging. |
| Paging (`VPN`, `PFN`, Offset, `4 KiB` Pages) & Page Table Entry (`PTE`) | `Working Understanding` | Core Linux memory-mapping unit (`getconf PAGESIZE = 4096`); PTE flags enforce Read/Write/Execute/User protection. |
| Multi-Level Page Tables (Conceptual Sparsity) | `Awareness` → light `Working Understanding` | Explains why a 64-bit process doesn't allocate gigabytes of page tables for unused virtual space (deep bit-math in Ch 20 deferred). |
| Translation Lookaside Buffer (TLB) & Context-Switch Cost (ASID/PCID) | `Working Understanding` | Hardware cache of recent `VPN → PFN` translations; connects directly to Phase 2 spatial locality and Block 3 context-switch cost. |
| `VmSize` (Virtual) vs. `VmRSS` (Resident) & Anonymous vs. File-Backed Mappings | `Working Understanding` | Prevents the #1 backend memory-monitoring mistake of panicking over large `VmSize` reservations. |

#### Deferred Topics & Explicit Phase Boundaries
- **Explicitly Deferred:** Demand paging, page faults, swapping, reclaim, and OOM killer (**Block 5**); segmentation (`OSTEP Ch 16`), free-space allocator algorithms (`OSTEP Ch 17`), inverted/multi-level page-table bit-manipulation drills (`OSTEP Ch 20` — Supplementary only); C pointer-bug catalog (`OSTEP 14.4` — not a C course); x86-64 Huge Pages / NUMA Zones / Compaction internals (`docs.kernel.org` `#huge-pages`, `#zones`, `#nodes`, `#compaction`); V8 managed heap layout, pointer compression, and GC generations (**Phase 5**).

#### Authoritative Resources & Assigned Scope *(Live-Verified `2026-11-28`)*
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | OSTEP v1.10 — **Ch 13: Address Spaces** & **Ch 14: Memory API** | [`vm-intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-intro.pdf), [`vm-api.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-api.pdf) | Ch `13.1`–`13.5` (~6.5 pp) + Ch `14.1`–`14.3` & `14.5` (*Types of Memory*, `malloc`/`free` overview, *Underlying OS Support* — ~4 pp; *skip `14.4` C bug catalog*) | ~25–30 min | Virtual memory goals, stack vs. heap, and how user allocators obtain memory from the OS via `brk`/`mmap`. |
| **Mandatory 2** | Free | OSTEP v1.10 — **Ch 15: Address Translation** & **Ch 18: Introduction to Paging** | [`vm-mechanism.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-mechanism.pdf), [`vm-paging.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-paging.pdf) | Ch `15.1`–`15.4` (~7 pp; *skim `15.5`*) + Ch `18.1`–`18.4, 18.6` (~8 pp; *skip `18.5` assembly trace*) | ~35–45 min | Hardware translation intuition, VPN/offset splitting, Page Table Entries, protection bits, and why linear page tables are large and slow. |
| **Mandatory 3** | Free | OSTEP v1.10 — **Ch 19: Translation Lookaside Buffers** | [`vm-tlbs.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-tlbs.pdf) | Sections `19.1`–`19.2`, `19.4`–`19.5`, `19.8` (*TLB Basic Algorithm*, *Accessing An Array*, *TLB Contents*, *Context Switches*, *Summary*) — ~7 pages *(skip `19.3` RISC SW-TLB & `19.7` MIPS bits)* | ~20–25 min | How TLBs make paging fast via locality and why context switches create TLB pressure. |
| **Mandatory 4** | Free | Linux `mmap(2)`, `proc_pid_maps(5)`, Kernel `mm/concepts.html`, & Node.js v26 `process.memoryUsage()` | [`mmap(2)`](https://man7.org/linux/man-pages/man2/mmap.2.html), [`proc_pid_maps(5)`](https://man7.org/linux/man-pages/man5/proc_pid_maps.5.html), [`mm/concepts.html`](https://docs.kernel.org/admin-guide/mm/concepts.html), [`process.memoryUsage()`](https://nodejs.org/docs/latest-v26.x/api/process.html#processmemoryusage) | `mmap(2)`: *Description* + `MAP_PRIVATE`/`MAP_SHARED`/`MAP_ANONYMOUS` only. `proc_pid_maps(5)`: full short page (~760 words). Kernel MM: *Virtual Memory Primer* & *Anonymous Memory* (~510 words). Node.js v26: `process.memoryUsage()` overview (`rss` vs. runtime fields) | ~20–25 min | Mapping abstract OSTEP theory directly onto Linux virtual memory areas (`VMAs`), permissions (`r-xp`/`rw-p`), and real Node.js process memory output. |
| **Supplementary** | Free | OSTEP v1.10 — Ch 20 (*Paging: Smaller Tables*) | [`vm-smalltables.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-smalltables.pdf) | Section `20.3` (*Multi-level Page Tables*) | Optional | Deeper walkthrough of hierarchical page-directory indexing if desired. |

#### Required Practical Experiment & Transfer Exercise
- **Practical Experiment 4 (`LO 4.1`, `LO 4.4` — ~35–45 min): Inspecting Real Virtual Address Spaces & Process Isolation.**
  1. Launch a small idle Node.js process (`node -e "console.log(process.pid, process.memoryUsage()); setTimeout(()=>{}, 30000)"`), then inspect `/proc/<PID>/maps` and `grep -E 'VmSize|VmRSS|RssAnon|RssFile' /proc/<PID>/status` to identify executable code (`r-xp`), shared libraries, `[heap]`, anonymous `rw-p` regions, and `[stack]`, and verify empirically why `VmSize` is vastly larger than `VmRSS`!
  2. Run a tiny ~20-line annotated C program that uses **`fork()`** to spawn a child, prints the identical inherited virtual address (`%p`) of a local variable in both parent and child, and mutates the variable in one process without changing the other's observed value—verifying virtual address-space isolation concretely (using `fork()` intentionally rather than two independently launched executables, because Linux **ASLR** [Address Space Layout Randomization] can assign different virtual addresses across independent process launches).
- **Transfer Exercise 4 (`LO 4.2`, `LO 4.3`, `LO 4.4` — ~35–40 min): Paging Translation, Page-Table Sizing & TLB Locality Analysis.** (1) Given a simplified 16-bit virtual address space with 4 KiB pages (`2^12` bytes) and a small 6-entry page table (including one invalid entry and one read-only entry), manually translate 4 virtual addresses into physical addresses or exact hardware faults (invalid/write-protection trap); (2) compute the raw size of a single-level linear page table for a 32-bit address space (`4 KiB` pages, `4 B` PTE = `4 MiB` per process) and explain why 48-bit x86-64 virtual spaces require hierarchical page tables; (3) explain mechanically why iterating row-by-row over a contiguous array achieves far higher TLB hit rates than random pointer chasing across scattered 4 KiB pages (connecting directly to Phase 2 Block 5!).
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~25–30 min`).

#### Flexible Study-Time Breakdown (Block 4)
- Guided Lesson + Pause & Checks 1–3: `~65–80 min` *(flexible priority)*
- Mandatory 1 (OSTEP Ch `13.1–13.5` + Ch `14.1–14.3, 14.5`): `~25–30 min`
- Mandatory 2 (OSTEP Ch `15.1–15.4` + Ch `18.1–18.4, 18.6`): `~35–45 min`
- Mandatory 3 (OSTEP Ch `19.1–19.2, 19.4–19.5, 19.8`): `~20–25 min`
- Mandatory 4 (`mmap(2)`, `proc_pid_maps(5)`, Kernel MM primer/anon, Node `memoryUsage`): `~20–25 min`
- Practical Experiment 4: `~35–45 min` | Transfer Exercise 4: `~35–40 min`
- **Total Required Core (without optional assessment):** **`~235–290 min (~3.9–4.8 h)`**
- Optional Closed-Book Assessment: `~25–30 min` *(optional)*
- **Total (with optional assessment):** **`~260–320 min (~4.3–5.3 h)`** *(Roadmap target: `5.0 h`)*

---

### Block 5: Paging, Page Faults, and Memory Pressure *(Target: 5.0 hours)*

- **Central Question & Motivation:** *What happens when a server reserves 128 MiB of virtual memory before writing to it, or when active backend processes together demand more physical RAM than the machine owns—why does latency suddenly collapse into disk-bound thrashing or end abruptly with the Linux OOM Killer?* This block explains some of the most painful real-world backend outages: silent RSS spikes after `fork()` (CoW), "Linux ate my RAM" confusion, swap thrashing, and Linux OOM kills (typically surfacing as `SIGKILL` / shell `Exit Code 137`, which still requires log/metric corroboration in Block 8).
- **Prerequisites & Dependencies:** Block 4 (paging, PTE present/dirty/accessed bits, `VmSize` vs. `VmRSS`, `mmap`); Block 2 (`fork` & CoW preview); Phase 2 Block 5 (RAM ~100 ns vs. SSD ~10–100 µs latency cliff).
- **Prepares For:** Block 7 (Page Cache shared role in buffered file I/O), Block 8 (diagnosing memory pressure & OOM `Exit Code 137`), Phase 5 (JS memory leaks growing OS RSS), Phase 10 (PostgreSQL working set & OS Page Cache), Phase 16 (cache memory budgets), Phase 24 (Docker container memory limits & cgroup OOM kills).
- **Primary Target Level:** `Working Understanding` | **Phase 3 Outcomes Supported:** Outcome 5.

#### Learning Objectives
- **LO 5.1 (`Working Understanding` | Bloom: *Analyze*):** Trace **demand paging** and **lazy physical allocation** step-by-step—explaining why `malloc()`/`mmap()` reserves virtual address space without immediately allocating physical frames, and contrasting **minor (soft) page faults** (in-RAM zero-fill / Page Cache / CoW resolution) against **major (hard) page faults** (requiring slow disk/swap I/O that blocks the process).
- **LO 5.2 (`Working Understanding` | Bloom: *Analyze*):** Explain **Copy-on-Write (`CoW`) page faults** mechanically (marking shared parent/child PTEs read-only until a write traps into the kernel, duplicates that 4 KiB page, and updates the PTE), explaining why `fork()` is fast initially yet can cause delayed RSS spikes when a parent mutates memory.
- **LO 5.3 (`Working Understanding` | Bloom: *Evaluate*):** Explain **swapping mechanisms**, **page replacement policies** (FIFO vs. LRU intuition, **Clock/LRU approximation** using accessed bits, and **clean vs. dirty page** eviction cost), and **thrashing** (working set > physical RAM causing continuous major page faults and catastrophic latency collapse).
- **LO 5.4 (`Working Understanding` | Bloom: *Analyze / Evaluate*):** Interpret Linux memory management in practice—distinguishing **Page Cache (file-backed)** vs. **Anonymous Memory** (heap/stack), **Memory Reclaim** (clean cache drop vs. dirty writeback vs. anonymous swap-out), **Linux `free -h` (`available` vs. `free` vs. `buff/cache`)**, the **OOM Killer** (`oom_score`), and contextual Node.js **`process.memoryUsage()`** (`rss`, `heapTotal`, `heapUsed`, `external`, `arrayBuffers` as OS-context buckets without V8 GC internals).

#### Concepts, Mechanism Depth & Terminology Inventory *(Internal Authoring Reference)*
| Concept / Term | Official Target Level | Mechanism Depth & Why It Matters in Phase 3 |
| :--- | :---: | :--- |
| Demand Paging & Lazy Allocation (Overcommit) | `Working Understanding` | Explains why reserving memory doesn't consume physical RAM until pages are touched (`vm.overcommit_memory = 0` on Ubuntu). |
| Minor (Soft) vs. Major (Hard) Page Fault | `Working Understanding` | Minor = resolved in RAM without backing-store I/O (no universal fixed latency; depends on CPU/kernel/allocator path); Major = requires blocking backing-store/swap I/O with substantially higher latency whose exact multiple depends on storage media, caching, and contention (no guaranteed 100×–1,000× slowdown). |
| Copy-on-Write (`CoW`) Fault | `Working Understanding` | Explains real-world RSS jumps during Redis snapshots or multi-worker process forks when pages are mutated. |
| Swap Space, Present Bit & Page Replacement (FIFO, LRU, Clock, Dirty Pages) | `Working Understanding` | Explains how the kernel decides which frames to evict under pressure and why dirty pages cost extra I/O. |
| Thrashing (Working Set > Physical RAM) | `Working Understanding` | Explains why a server under severe memory pressure can stay "alive" while repeated major faults cause severe latency collapse and plummeting useful throughput (without promising a universal 100× p99 multiplier). |
| Page Cache vs. Anonymous Memory & Reclaim | `Working Understanding` | Explains why `free` RAM near 0 on Linux is *normal* (`buff/cache` is reclaimable!) while `available` RAM is the true headroom metric. |
| Linux OOM Killer (`oom_score`) | `Working Understanding` | Explains last-resort kernel survival mechanism when reclaim fails—sacrificing a high-memory backend process via `SIGKILL`. |
| Contextual Node.js `process.memoryUsage()` (`rss` vs. V8 Heap/Buffers) | `Working Understanding` *(OS-context level only)* | Explains why Node.js `rss` is larger than `heapUsed` (includes binary/libs, stack, native Buffers, unused heap pages, and OS allocator retention). |

#### Deferred Topics & Explicit Phase Boundaries
- **Explicitly Deferred:** V8 GC algorithms, generational heap tuning (`--max-old-space-size` internals — **Phase 5**); synthetic replacement-policy workload curves (`OSTEP 22.2, 22.4, 22.6`), VAX/VMS (`OSTEP 23.1`), Meltdown/Spectre (`OSTEP 23.2` security subsections); Linux kernel `kswapd`/watermark tuning (`vm.swappiness`, `vm.dirty_ratio` — **Phase 23**); container `cgroup v2` `memory.max` OOM behavior (**Phase 24**, mentioned as a 1-sentence `Context` bridge). **Safety Hard Boundary:** Never intentionally exhaust the 16 GB host RAM or trigger the host OOM killer in any experiment!

#### Authoritative Resources & Assigned Scope *(Live-Verified `2026-11-28`)*
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | OSTEP v1.10 — **Ch 21: Beyond Physical Memory: Mechanisms** | [`vm-beyondphys.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-beyondphys.pdf) | Sections `21.1`–`21.6` (*Swap Space*, *Present Bit*, *Page Fault*, *Memory Full*, *Page Fault Control Flow*, *When Replacements Occur*) — ~7 pages | ~20–25 min | Exact hardware+OS control flow when a PTE present bit is `0`, plus background watermark reclaim. |
| **Mandatory 2** | Free | OSTEP v1.10 — **Ch 22: Beyond Physical Memory: Policies** | [`vm-beyondphys-policy.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-beyondphys-policy.pdf) | Sections `22.1`, `22.3`, `22.5`, `22.8`–`22.9`, `22.11` (*Cache Management*, *FIFO*, *LRU*, *Approximating LRU*, *Dirty Pages*, *Thrashing*) — ~6 pages *(skip `22.2, 22.4, 22.6–22.7`)* | ~20–25 min | Why LRU works via locality, why dirty pages matter, and how thrashing destroys system performance. |
| **Mandatory 3** | Free | OSTEP v1.10 — **Ch 23: Complete Virtual Memory Systems** | [`vm-complete.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-complete.pdf) | Section `23.2` (*The Linux Virtual Memory System*) — **only** *The Linux Address Space*, *Page Table Structure*, and *The Page Cache* (~5 pages; *skip `23.1 VAX/VMS`, Large Pages, & Security subsections*) | ~15–20 min | Real Linux virtual memory structure tying together Blocks 4 & 5. |
| **Mandatory 4** | Free | Linux Kernel MM Concepts + `proc_pid_status(5)` / `proc_pid_oom_score(5)` + Node.js v26 `process.memoryUsage()` | [`mm/concepts.html`](https://docs.kernel.org/admin-guide/mm/concepts.html), [`proc_pid_status(5)`](https://man7.org/linux/man-pages/man5/proc_pid_status.5.html), [`proc_pid_oom_score(5)`](https://man7.org/linux/man-pages/man5/proc_pid_oom_score.5.html), [`process.memoryUsage()`](https://nodejs.org/docs/latest-v26.x/api/process.html#processmemoryusage), [`memoryUsage.rss()`](https://nodejs.org/docs/latest-v26.x/api/process.html#processmemoryusagerss) | Kernel MM: `#page-cache`, `#anonymous-memory`, `#reclaim`, `#oom-killer` (~635 words total; *skip `#huge-pages`, `#zones`, `#nodes`, `#compaction`*). Man-pages: targeted `VmRSS`/`RssAnon`/`RssFile` & `oom_score` fields. Node.js v26: `memoryUsage()` + `memoryUsage.rss()` | ~20–25 min | Production Linux reclaim & OOM semantics and how Node.js reports OS RSS alongside runtime memory. |

#### Required Practical Experiment & Transfer Exercise
- **Practical Experiment 5 (`LO 5.1`, `LO 5.4` — ~45–55 min): Safe 128 MiB Demand-Paging, Minor Page Faults & Linux Memory Inspection.** *(Strictly bounded to `<= 128 MiB`, `< 1%` of the learner's 16 GB RAM—zero swap/OOM risk!)*
  1. Run a tiny ~30-line annotated C program (or Node.js script with `Buffer.allocUnsafe(128 * 1024 * 1024)` + step-by-step pause) that reserves `128 MiB` via `mmap(MAP_PRIVATE | MAP_ANONYMOUS)` **without touching it**, prints `VmSize` and `VmRSS` from `/proc/self/status`, then writes 1 byte every `4096` bytes (`32,768` 4 KiB steps) and re-prints `VmSize`, `VmRSS`, and minor/major fault counts via `/usr/bin/time -v`! Observe empirically: (a) before touching, `VmSize` jumps by `~128 MiB` while `VmRSS` barely moves; (b) after touching, `VmRSS` jumps by `~128 MiB` and `Minor (reclaiming a frame) page faults` increases noticeably—with `~32,768` (`128 MiB / 4 KiB`) as an **approximate baseline expectation** for 4 KiB base pages, explicitly noting that observed counts can differ substantially due to **Transparent Huge Pages (THP)**, kernel fault-around/prefaulting, or runtime/allocator implementation differences—while `Major (requiring I/O) page faults` typically remains `0` on a machine with ample free RAM!
  2. Read-only host inspection: Run `free -h`, check `MemAvailable` vs. `MemFree` vs. `Cached` in `/proc/meminfo`, inspect `/proc/<PID>/oom_score` of a temporary sleep/Node process, and run a short Node.js script printing `process.memoryUsage()` before and after allocating/filling a `64 MiB` `Uint8Array` to compare `rss` vs. `heapUsed` vs. `arrayBuffers`.
- **Transfer Exercise 5 (`LO 5.1`–`LO 5.4` — ~40–45 min): Production Memory-Pressure & OOM Incident Analysis.** Diagnose three realistic scenarios: (a) an ops alert fires because `free -h` on a 16 GB PostgreSQL + Node.js host shows `free: 280 MiB`, `buff/cache: 9.8 GiB`, `available: 9.4 GiB`—explain why restarting the server is wrong and what Linux is actually doing; (b) a 4 GB in-memory cache process calls `fork()` to run a background persistence job, initially using almost no extra RAM, then triggers an OOM kill 45 seconds later during heavy write traffic—trace the exact **CoW page-fault** mechanism; (c) distinguish a server suffering **swap thrashing** (high major faults, low CPU efficiency, extreme p99 latency) from one killed instantly by the **OOM killer**.
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~25–30 min`).

#### Flexible Study-Time Breakdown (Block 5)
- Guided Lesson + Pause & Checks 1–3: `~65–80 min` *(flexible priority)*
- Mandatory 1 (OSTEP Ch `21.1–21.6`): `~20–25 min`
- Mandatory 2 (OSTEP Ch `22.1, 22.3, 22.5, 22.8–22.9, 22.11`): `~20–25 min`
- Mandatory 3 (OSTEP Ch `23.2` Linux VM selected): `~15–20 min`
- Mandatory 4 (Kernel MM `#page-cache`/`#anonymous-memory`/`#reclaim`/`#oom-killer` + `proc` + Node `memoryUsage`): `~20–25 min`
- Practical Experiment 5: `~45–55 min` | Transfer Exercise 5: `~40–45 min`
- **Total Required Core (without optional assessment):** **`~225–275 min (~3.75–4.6 h)`**
- Optional Closed-Book Assessment: `~25–30 min` *(optional)*
- **Total (with optional assessment):** **`~250–305 min (~4.2–5.1 h)`** *(Roadmap target: `5.0 h`)*

---

### Block 6: Files, Directories, and File Descriptors *(Target: 4.0 hours)*

- **Central Question & Motivation:** *What actually happens inside the kernel when a backend opens a log file, reads config, or writes to `stdout`—why does Linux hide files, pipes, and network sockets behind tiny integer file descriptors, and how do leaked descriptors or unlinked-but-open files take down production services?* Every Node.js engineer eventually hits `EMFILE: too many open files` or a "100% full disk after deleting the 40 GB log file" incident; both become trivial once the 3-tier kernel descriptor and inode model is understood.
- **Prerequisites & Dependencies:** Block 1 (syscalls); Block 2 (`fork()`/`execve()` process creation & state).
- **Prepares For:** Block 7 (blocking/non-blocking I/O & `fsync` on FDs), Block 8 (FD-leak diagnosis in Capstone), Phase 4 (network sockets as FDs subject to `RLIMIT_NOFILE`), Phase 10 (database data & WAL files), Phase 11 (Node.js streams & `fs`), Phase 23 (Linux permissions, mounts, log rotation).
- **Primary Target Level:** `Practical Competence` | **Phase 3 Outcomes Supported:** Outcome 6.

#### Learning Objectives
- **LO 6.1 (`Working Understanding` | Bloom: *Understand*):** Explain the Linux **file, inode, and directory** abstraction—separating a directory entry (human-readable name → inode number) from the **inode** (metadata, size, link count, data location)—and explain **hard links vs. symbolic links** and why `unlink()` on an open file does *not* reclaim disk blocks until both the link count **and** open file-descriptor reference count reach zero.
- **LO 6.2 (`Working Understanding` | Bloom: *Analyze*):** Trace the **3-tier kernel descriptor architecture**—(1) per-process **File Descriptor Table** (small integers `0, 1, 2, 3...`), (2) system-wide **Open File Description** (current file offset, access/status flags like `O_APPEND` / `O_NONBLOCK`), and (3) **Inode/Vnode**—explaining **standard streams** (`0=stdin`, `1=stdout`, `2=stderr`) and how `fork()` and `dup()` share open file descriptions (enabling shell redirection and shared offsets).
- **LO 6.3 (`Practical Competence` | Bloom: *Apply / Analyze*):** Inspect live process descriptors safely via `/proc/<PID>/fd` and `lsof -p <PID>`, explain per-process FD limits (`RLIMIT_NOFILE` / `ulimit -n`), and reproduce/resolve an **`EMFILE` ("Too many open files")** descriptor-leak failure safely inside a restricted subshell.
- **LO 6.4 (`Working Understanding` | Bloom: *Understand*):** Explain why Linux's uniform descriptor abstraction (`open`, `read`, `write`, `close` interface) extends beyond regular files to **pipes** and **network sockets** (setting up Block 7 and Phase 4 without teaching networking protocols yet).

#### Concepts, Mechanism Depth & Terminology Inventory *(Internal Authoring Reference)*
| Concept / Term | Official Target Level | Mechanism Depth & Why It Matters in Phase 3 |
| :--- | :---: | :--- |
| File, Inode & Directory Entry | `Working Understanding` | Decouples file names from underlying storage identity; explains `stat(2)` and reference counting. |
| Hard Link vs. Symbolic Link & `unlink()` Semantics | `Working Understanding` | Explains the classic production outage: deleting an open log file frees zero disk space until the server closes the FD or restarts. |
| Standard Streams (`FD 0`, `FD 1`, `FD 2`) | `Practical Competence` | Explains why containerized/Linux backends log to `stdout`/`stderr` and how shells wire redirection. |
| 3-Tier Kernel Descriptor Model (FD Table → Open File Description → Inode) | `Working Understanding` | Clarifies why two independent `open()` calls have separate file offsets while `fork()`/`dup()` share one offset and status flags! |
| `open(2)` & `close(2)` Semantics (`O_CREAT`, `O_APPEND`, `O_CLOEXEC` awareness) | `Practical Competence` | Concrete syscall mechanics behind every file handle open/close in Node.js and databases. |
| File Descriptor Limits (`RLIMIT_NOFILE`, `ulimit -n`) & `EMFILE` Leaks | `Practical Competence` | Directly prevents and diagnoses production server crashes when leaked files/sockets exhaust the FD table. |
| `/proc/<PID>/fd` & `lsof` Inspection | `Practical Competence` | Live, read-only Linux visibility into every open file, pipe, or socket held by a backend process. |

#### Deferred Topics & Explicit Phase Boundaries
- **Explicitly Deferred:** OSTEP Chapters 40–45 (ext4/FFS on-disk block layout, journaling, crash consistency, SSD FTL — **Phases 10 & 23**); OSTEP `39.7 fsync()` (placed intentionally in **Block 7** alongside Page Cache I/O); OSTEP `39.11–39.13` C directory-traversal APIs; OSTEP `39.16–39.17` Linux permission bits, ACLs, and filesystem mounting (**Phase 23**); `fs.createReadStream` backpressure (**Phase 11**); TCP socket creation & port exhaustion (**Phase 4**).

#### Authoritative Resources & Assigned Scope *(Live-Verified `2026-11-28`)*
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | OSTEP v1.10 — **Chapter 39: Interlude: Files and Directories** | [`file-intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/file-intro.pdf) | Sections `39.1`–`39.4`, `39.6`, `39.9`–`39.10`, `39.14`–`39.15` (*Files/Directories*, *FS Interface*, *Creating*, *Reading/Writing*, *Shared File Table Entries: `fork`/`dup`*, *`stat`*, *`unlink`*, *Hard & Symbolic Links*) — ~11 pages *(defer `39.7` to Block 7; skip `39.5, 39.8, 39.11–39.13, 39.16–39.17`)* | ~35–40 min | Clear mental model of inodes, directories, FDs, shared open file descriptions across `fork`/`dup`, and `unlink` reference counting. |
| **Mandatory 2** | Free | Linux `man-pages` — `open(2)`, `close(2)`, `proc(5)`, `proc_pid_fd(5)` + Node.js v26 stdio `.fd` | [`open(2)`](https://man7.org/linux/man-pages/man2/open.2.html), [`close(2)`](https://man7.org/linux/man-pages/man2/close.2.html), [`proc(5)`](https://man7.org/linux/man-pages/man5/proc.5.html), [`proc_pid_fd(5)`](https://man7.org/linux/man-pages/man5/proc_pid_fd.5.html), [`process.stdout.fd`](https://nodejs.org/docs/latest-v26.x/api/process.html#processstdoutfd) | `open(2)`: *Description* & *Open file descriptions* (skip `openat2`/`O_DIRECT` deep details). `close(2)`: *Description* & *Caveats*. `proc(5)`: *Description* intro; `proc_pid_fd(5)`: full short page (~800 words). Node.js v26: `stdin`/`stdout`/`stderr.fd` quick check | ~25–30 min | Exact Linux terminology (*open file description* vs. *file descriptor*), why `close()` errors matter, and how `/proc/<PID>/fd` exposes symlinks to open targets. |
| **Supplementary** | Free | Linux `man-pages` — `dup(2)` | [`dup(2)`](https://man7.org/linux/man-pages/man2/dup.2.html) | *Description* only | Optional | Exact semantics of `dup`/`dup2` behind shell redirection (`2>&1`). |

#### Required Practical Experiment & Transfer Exercise
- **Practical Experiment 6 (`LO 6.1`, `LO 6.2`, `LO 6.3` — ~40–50 min): Live `/proc/<PID>/fd` Inspection, Unlinked Open Files, and Safe Subshell `EMFILE` Reproduction.**
  1. Start a small Node.js script that opens 2 temp files in `/tmp`, deletes (`fs.unlinkSync`) one of them while keeping its file descriptor open, and waits 15 seconds. Inspect `ls -l /proc/<PID>/fd` and `lsof -p <PID>` to observe FDs `0`, `1`, `2`, the open temp file, and the `(deleted)` unlinked file still consuming its inode until closed!
  2. Because Ubuntu 24.04 defaults to a high soft FD limit (`ulimit -Sn = 1048576` verified locally), safely run an isolated subshell with **`bash -c 'ulimit -n 32 && node fd-leak-demo.mjs'`** that opens small temp files in `/tmp` without closing them—hitting exact `EMFILE: too many open files` cleanly around ~25 files without affecting the host shell—then fix the leak with `try { ... } finally { await handle.close(); }` and verify 100 sequential opens succeed cleanly inside the same `ulimit -n 32` cap!
- **Transfer Exercise 6 (`LO 6.1`, `LO 6.2`, `LO 6.3` — ~35–40 min): 3-Tier FD Table Trace & Production Incident Diagnosis.** (1) Trace the resulting file contents and offsets when Process A opens a file twice independently (`fd3`, `fd4`) vs. opens once (`fd3`) and `fork()`s Child B (both writing 5 bytes); (2) diagnose two production tickets: Ticket A (*"Disk usage hit 100% on our API host; I ran `rm /var/log/app.log` (30 GB), `ls` confirms it's gone, yet `df -h` STILL shows 100% full!"*) and Ticket B (*"Our Node.js service only has 20 files on disk, yet under heavy traffic it crashes with `EMFILE: too many open files`—why?"* — connecting FDs to sockets and `RLIMIT_NOFILE`).
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~20–25 min`).

#### Flexible Study-Time Breakdown (Block 6)
- Guided Lesson + Pause & Checks 1–3: `~55–65 min` *(flexible priority)*
- Mandatory 1 (OSTEP Ch `39.1–39.4, 39.6, 39.9–39.10, 39.14–39.15`): `~35–40 min`
- Mandatory 2 (`open(2)`, `close(2)`, `proc(5)`, `proc_pid_fd(5)`, Node stdio `.fd`): `~25–30 min`
- Practical Experiment 6: `~40–50 min` | Transfer Exercise 6: `~35–40 min`
- **Total Required Core (without optional assessment):** **`~190–225 min (~3.2–3.75 h)`**
- Optional Closed-Book Assessment: `~20–25 min` *(optional)*
- **Total (with optional assessment):** **`~210–250 min (~3.5–4.2 h)`** *(Roadmap target: `4.0 h`)*

---

### Block 7: I/O, Blocking, and Non-Blocking Execution *(Target: 5.0 hours)*

- **Central Question & Motivation:** *When a backend calls `read()` or `write()` on a slow device or pipe, why doesn't the CPU sit spinning at 100%, why does `write()` return in microseconds even when SSDs take much longer, and how do blocking I/O, non-blocking I/O (`O_NONBLOCK`), and OS readiness notification (`epoll`) shape how servers scale?* This block forms the exact mechanical bridge from single-process OS mechanics into Phase 4 Networking, Phase 6 Concurrency, Phase 10 Database Durability, and Phase 11 Node.js/libuv!
- **Prerequisites & Dependencies:** Block 1 (syscalls & `strace`); Block 2 (`Blocked`/`Sleeping` process state); Block 3 (CPU scheduler running another `Ready` process while one sleeps); Block 5 (OS Page Cache); Block 6 (file descriptors & `open`/`read`/`write`); Phase 2 Block 5 (RAM vs. SSD vs. network latency orders of magnitude).
- **Prepares For:** Block 8 (diagnosing I/O stalls & `strace -T`), Phase 4 (network socket I/O & kernel buffers), Phase 6 (event-driven concurrency vs. thread-per-connection models), Phase 10 (PostgreSQL WAL `fsync` & durability trade-offs), Phase 11 (libuv `epoll` for sockets vs. thread pool for regular file I/O!).
- **Primary Target Level:** `Working Understanding` | **Phase 3 Outcomes Supported:** Outcome 7.

#### Learning Objectives
- **LO 7.1 (`Working Understanding` | Bloom: *Analyze*):** Explain how the OS interacts with slow hardware I/O devices—contrasting **polling** (CPU busy-wait), **hardware interrupts** (the waiting process blocks, the CPU runs another ready task, and a hardware interrupt from the device signals I/O completion so the OS can wake the waiting process), and **DMA (Direct Memory Access)** (offloading bulk byte transfers in both device-to-memory and memory-to-device directions from the CPU)—via device drivers.
- **LO 7.2 (`Working Understanding` | Bloom: *Analyze*):** Explain how the **Linux Page Cache** buffers regular-file `read(2)` and `write(2)` calls in RAM—explaining why `write()` returning success does **not** guarantee physical disk durability after a power loss, how **dirty pages** flush asynchronously, and the latency-vs-durability trade-off of **`fsync(2)`**.
- **LO 7.3 (`Working Understanding` | Bloom: *Analyze*):** Contrast **blocking I/O** (process moves to `Blocked`/`Sleeping` state on a kernel wait queue until data/space is ready) against **non-blocking I/O (`O_NONBLOCK`)** (syscall returns immediately with `-1` and `errno = EAGAIN`/`EWOULDBLOCK` if not ready), including **short reads/writes** and `EINTR` awareness, and explain why naive non-blocking retry loops degenerate into CPU-burning busy polling.
- **LO 7.4 (`Awareness` | Bloom: *Understand*):** Explain at a conceptual OS-boundary level **what problem OS I/O multiplexing / readiness notification (`poll(2)` / Linux `epoll(7)`) solves** (allowing one thread to wait in kernel sleep across thousands of non-blocking FDs and wake only on ready FDs), plus the crucial Linux regular-file distinction: **`O_NONBLOCK` does not make ordinary regular-file I/O asynchronous or non-blocking**, **`poll()` reports regular files as immediately ready**, and **`epoll_ctl()` generally rejects regular files with `EPERM`** (explaining ahead of time why runtimes like Node.js/libuv later need a worker thread pool for regular file I/O in Phase 11!).

#### Concepts, Mechanism Depth & Terminology Inventory *(Internal Authoring Reference)*
| Concept / Term | Official Target Level | Mechanism Depth & Why It Matters in Phase 3 |
| :--- | :---: | :--- |
| Polling vs. Hardware Interrupts vs. DMA | `Working Understanding` | Explains how a CPU avoids busy-waiting (process blocks while another ready task runs until a device completion interrupt arrives) and how DMA offloads bulk device-to-memory and memory-to-device transfers. |
| Device Driver Abstraction | `Awareness` | Explains how OS hides hardware-specific device registers behind uniform kernel I/O interfaces. |
| Buffered I/O (Kernel Page Cache) vs. `fsync(2)` Durability | `Working Understanding` | Core truth behind backend logging speed vs. database crash durability (direct prerequisite for Phase 10 WAL!). |
| Blocking I/O & Kernel Wait Queues | `Working Understanding` | Connects `read()`/`write()` stalls directly to Block 2's `S`/`D` process states and Block 3's scheduler. |
| Short Reads / Short Writes & `EINTR` | `Working Understanding` | Explains why `read(fd, buf, 4096)` returning `120` bytes is normal on pipes/sockets—never an error or EOF. |
| Non-Blocking I/O (`O_NONBLOCK`, `EAGAIN` / `EWOULDBLOCK`) | `Working Understanding` | Explains exact OS return contract when an application refuses to block a thread on an unready FD. |
| OS I/O Multiplexing / Readiness Notification (`poll` / `epoll` Concept) | `Awareness` | Motivates how modern single-process evented backends avoid both 10,000 blocked threads and 100% CPU busy-polling (event-loop architecture deferred to Phases 6 & 11). |
| Regular-File Non-Blocking & Multiplexing Caveat | `Awareness` | Prevents a classic misconception: on Linux, `O_NONBLOCK` does not make ordinary regular-file I/O asynchronous or non-blocking; `poll()` reports regular files as immediately ready, while `epoll_ctl()` generally rejects regular files with `EPERM`! |

#### Deferred Topics & Explicit Phase Boundaries
- **Explicitly Deferred:** OSTEP `36.8` IDE disk driver code & `36.9` history; OSTEP Ch 37–38 (HDD seek physics & RAID) and Ch 40–45 (filesystem journaling/SSDs — **Phases 10 & 23**); `epoll_create1`/`epoll_ctl`/`epoll_wait` C coding, edge-triggered vs. level-triggered tuning, `io_uring` implementation, and full event-loop concurrency architecture (**Phase 6** & **Phase 11**); libuv internals (**Phase 11**); TCP socket recv/send buffers & flow control (**Phase 4**); `O_DIRECT` database bypass tuning (**Phase 10**).

#### Authoritative Resources & Assigned Scope *(Live-Verified `2026-11-28`)*
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | OSTEP v1.10 — **Ch 36: I/O Devices** & **Ch 39.7** (*`fsync`*) | [`file-devices.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/file-devices.pdf), [`file-intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/file-intro.pdf) | Ch `36.1`–`36.5`, `36.7`, `36.10` (~8 pp, ~25–30 min; *skip `36.6, 36.8–36.9`*) + Ch `39.7` (*Writing Immediately With `fsync()`*, ~1 p, ~5–10 min) | ~30–40 min | Hardware device interaction (polling vs. interrupts vs. DMA), device drivers, and why `write()` needs `fsync()` for durability. |
| **Mandatory 2** | Free | Linux Kernel MM (`#page-cache`) + Linux `read(2)`, `write(2)`, `open(2)` (`O_NONBLOCK`) | [`mm/concepts.html#page-cache`](https://docs.kernel.org/admin-guide/mm/concepts.html#page-cache), [`read(2)`](https://man7.org/linux/man-pages/man2/read.2.html), [`write(2)`](https://man7.org/linux/man-pages/man2/write.2.html), [`open(2)`](https://man7.org/linux/man-pages/man2/open.2.html) | Kernel `#page-cache` (~100 words); `read(2)` & `write(2)`: *Description*, *Return Value* (short counts & `0` = EOF), *Errors* (`EAGAIN`/`EWOULDBLOCK`, `EINTR`); `open(2)`: `O_NONBLOCK` flag paragraph only | ~30–35 min | Exact POSIX/Linux semantics of blocking vs. `O_NONBLOCK`, partial transfers, and Page Cache dirty-page buffering. |
| **Mandatory 3** | Free | Linux `man-pages` — `epoll(7)` *(Awareness Intro Only)* | [`epoll(7)`](https://man7.org/linux/man-pages/man7/epoll.7.html) | Opening *Description* paragraphs ONLY (what `epoll` monitors and the high-level concept of waiting on multiple ready FDs; *skip Level-triggered vs. Edge-triggered Q&A & C examples*) | ~10–15 min | High-level OS intuition of readiness notification—bridging directly to Phase 6 & Phase 11 without diving into `epoll` coding. |
| **Supplementary** | Free | Linux `man-pages` — `fcntl(2)`, `pipe(2)`, `epoll_ctl(2)` | [`fcntl(2)`](https://man7.org/linux/man-pages/man2/fcntl.2.html), [`pipe(2)`](https://man7.org/linux/man-pages/man2/pipe.2.html), [`epoll_ctl(2)`](https://man7.org/linux/man-pages/man2/epoll_ctl.2.html) | `fcntl(2)`: `F_GETFL`/`F_SETFL` (toggling `O_NONBLOCK`); `pipe(2)`: *Description*; `epoll_ctl(2)`: `EPERM` error entry for regular files | Optional | Reference for toggling non-blocking mode on an already-open pipe/FD and confirming `epoll_ctl` `EPERM` on regular files. |

#### Required Practical Experiment & Transfer Exercise
- **Practical Experiment 7 (`LO 7.2`, `LO 7.3` — ~45–55 min): Tracing Blocking vs. Non-Blocking Pipe Reads (`strace -tt -T`) and Measuring Buffered `write()` vs. `fsync()` Behavior.**
  1. Compile and run a minimal ~35-line annotated C program (syntax explained inline) that creates an anonymous `pipe()`, forks a child that sleeps 1.5 seconds before writing `"ready"`, and lets the parent test two modes under unprivileged **`strace -tt -T`**:
     - **Blocking mode:** watch `read(fd, ...)` literally block in the kernel for `~1.500 s` (`<1.50...>` in `strace -T`), consuming ~0% CPU!
     - **Non-blocking mode (`O_NONBLOCK` via `fcntl`):** watch `read(fd, ...)` return immediately with **`= -1 EAGAIN (Resource temporarily unavailable)`**!
  2. Run a tiny measurement-driven Node.js script writing 500 small 256-byte records to a temp file in `/tmp` (cleaned up on exit) in two modes: (a) buffered `fs.writeSync` only (kernel Page Cache), vs. (b) `fs.writeSync` + `fs.fsyncSync` on every record, recording both wall-clock timings (`performance.now()`) and `strace -c` syscall summaries. Analyze the actual observed difference honestly rather than expecting a guaranteed order-of-magnitude gap—explaining how the measured result depends on the host filesystem (e.g., `ext4` vs. `tmpfs`), underlying storage device (NVMe vs. SATA SSD vs. virtualized storage), drive/controller caching, and kernel writeback behavior.
- **Transfer Exercise 7 (`LO 7.1`–`LO 7.4` — ~40–45 min): I/O Execution & Durability Trade-Off Audit.** (1) A developer writes a custom payment-audit logger that calls `write()` and immediately returns HTTP 200 claiming *"the record is safely persisted on SSD"*—explain the Page Cache mechanism that falsifies this claim and analyze the throughput-vs-durability trade-off of calling `fsync()` per request vs. batching; (2) trace why replacing 5,000 blocking threads with a naive `while (true) { read(fd_nonblock); }` loop turns an idle server into a 100% CPU meltdown, and explain conceptually what OS primitive (`poll`/`epoll`) resolves both extremes.
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Assessment (`~25–30 min`).

#### Flexible Study-Time Breakdown (Block 7)
- Guided Lesson + Pause & Checks 1–3: `~65–80 min` *(flexible priority)*
- Mandatory 1 (OSTEP Ch `36.1–36.5, 36.7, 36.10` + Ch `39.7`): `~30–40 min`
- Mandatory 2 (Kernel `#page-cache` + `read(2)`, `write(2)`, `open(2)` `O_NONBLOCK`): `~30–35 min`
- Mandatory 3 (`epoll(7)` conceptual intro only): `~10–15 min`
- Practical Experiment 7: `~45–55 min` | Transfer Exercise 7: `~40–45 min`
- **Total Required Core (without optional assessment):** **`~220–270 min (~3.7–4.5 h)`**
- Optional Closed-Book Assessment: `~25–30 min` *(optional)*
- **Total (with optional assessment):** **`~245–300 min (~4.1–5.0 h)`** *(Roadmap target: `5.0 h`)*

---

### Block 8: Signals, Process Control, and Backend OS Diagnosis *(Target: 6.0 hours — Integrative Capstone Block)*

- **Central Question & Motivation:** *How does Linux notify, pause, or terminate a live backend service gracefully versus forcefully, and how do we combine every Phase 3 OS mechanism with Phase 1's Scientific Method and USE Method to pinpoint real production CPU, memory, FD, I/O, and termination incidents from raw kernel evidence?* This capstone unites all 7 prior blocks into production diagnostic judgment.
- **Prerequisites & Dependencies:** All Phase 3 Blocks 1–7 + Phase 1 Block 5 (Scientific Method, Diagnostic Anti-Methods, conceptual USE Method) + Phase 2 Block 7 (falsifiable empirical discipline).
- **Prepares For:** Phase 4 (network incident debugging), Phase 11 (Node.js graceful shutdown, HTTP draining & `SIGTERM` lifecycle), Phase 20 (observability & telemetry), Phase 21 (reliability & graceful degradation), Phase 23 (Linux ops & `systemd`), Phase 24 (Docker/Kubernetes `SIGTERM` → grace period → `SIGKILL` / `Exit Code 137`), Phase 25 (production profiling).
- **Primary Target Level:** `Practical Competence` | **Phase 3 Outcomes Supported:** Outcomes 8, 9 (Integrative Outcomes 1–9).

#### Learning Objectives
- **LO 8.1 (`Practical Competence` | Bloom: *Analyze / Apply*):** Explain Linux **signals** as asynchronous kernel-to-process notifications, their three **dispositions** (`default`, `ignore`, `catch`), the backend roles of key standard signals (`SIGINT`, `SIGTERM`, `SIGKILL`, `SIGHUP`, `SIGPIPE`, `SIGSEGV`, `SIGCHLD`), and **why `SIGKILL` (9) and `SIGSTOP` (19) cannot be caught, blocked, or ignored**.
- **LO 8.2 (`Practical Competence` | Bloom: *Apply / Evaluate*):** Decode Unix **process exit statuses** (`0` success, `1–125` app errors, **`128 + signal_number`** for signal termination—specifically **`137` = `128 + 9` (`SIGKILL` termination, where an OOM kill is only one possible cause alongside manual or orchestrator `kill -9`, requiring kernel log/metric corroboration)** and **`143` = `128 + 15` (`SIGTERM`)**) and implement/verify basic graceful `SIGTERM`/`SIGINT` cleanup vs. forced `SIGKILL` termination in a Node.js v26 process (`process.on('SIGTERM')`, `process.exitCode`).
- **LO 8.3 (`Practical Competence` | Bloom: *Analyze / Evaluate*):** Connect Phase 1 Block 5's **USE Method** (*Utilization, Saturation, Errors*) and **Scientific Method** (`Question → Hypothesis → Prediction → Test → Analysis`) directly to Phase 3's safe Linux OS observables (`/proc/<PID>/{status,maps,fd,stat}`, `ps`, `free`, `vmstat`, `strace` on child commands) to diagnose CPU starvation, RSS/page-fault/OOM issues, FD leaks, I/O stalls, zombies, and signal kills without falling into Streetlight or Drunk Man anti-methods.
- **LO 8.4 (`Practical Competence` | Bloom: *Create / Evaluate*):** Execute the 4-stage **Phase 3 Integrative Backend OS Diagnostic Capstone Lab**, gathering raw reproducible evidence across controlled local Linux workloads and authoring an honest, mechanism-grounded OS Incident Diagnosis Report.

#### Concepts, Mechanism Depth & Terminology Inventory *(Internal Authoring Reference)*
| Concept / Term | Official Target Level | Mechanism Depth & Why It Matters in Phase 3 |
| :--- | :---: | :--- |
| Signal & Signal Dispositions (`Default`, `Ignore`, `Catch`) | `Practical Competence` | Core OS control channel between users, supervisors, kernel errors, and backend processes. |
| Standard Backend Signals (`SIGINT`, `SIGTERM`, `SIGKILL`, `SIGHUP`, `SIGPIPE`, `SIGSEGV`, `SIGCHLD`) | `Practical Competence` | Each maps directly to daily backend ops: Ctrl+C, clean deploy stop, hard kill/OOM, config reload, broken client pipe, invalid memory access, child exit. |
| Uncatchable Signals (`SIGKILL` / `SIGSTOP`) & `kill(2)` | `Practical Competence` | Guarantees the kernel/operator always retains ultimate control even over a hung or runaway process. |
| Exit-Status Encoding (`128 + N`: `137`, `143`, `139`) | `Practical Competence` | Distinguishes `SIGKILL` termination (`137`—where OOM is only one possible cause alongside manual/orchestrator forced kill, requiring kernel/host evidence corroboration), unhandled `SIGTERM` (`143`), and segfault (`139`) in CI/production logs. |
| Graceful Shutdown Contract (OS Level) | `Practical Competence` *(OS signal mechanics; HTTP draining deepened in Phase 11)* | Explains why ignoring `SIGTERM` or crashing on `SIGKILL` drops in-flight work and leaves corrupt temp files/locks. |
| USE Method Applied to Linux OS Observables | `Practical Competence` | Systematic triage across CPU (`vmstat`/`/proc/stat`), Memory (`free`/`/proc/status`/`oom_score`), FDs (`/proc/fd`), and I/O (`strace -T`/state `S`/`D`). |

#### Deferred Topics & Explicit Phase Boundaries
- **Explicitly Deferred:** POSIX real-time signals (`SIGRTMIN..SIGRTMAX`), async-signal-safety C handler reentrancy proofs, `sigaction(2)` SA_RESTART C internals (`Awareness` note only); production Node.js HTTP keep-alive connection draining & PostgreSQL pool teardown during shutdown (**Phases 11 & 14**); `systemd` `KillSignal`/`TimeoutStopSec` unit administration (**Phase 23**); Kubernetes `preStop` hooks & `terminationGracePeriodSeconds` (**Phase 24**); `perf`/eBPF profiling (**Phase 25**).

#### Authoritative Resources & Assigned Scope *(Live-Verified `2026-11-28`)*
| Priority | Access | Resource | Direct Verified URL | Exact Assigned Scope | Est. Time | What the Learner Should Extract |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- |
| **Mandatory 1** | Free | Linux `man-pages` — `signal(7)` & `kill(2)` | [`signal(7)`](https://man7.org/linux/man-pages/man7/signal.7.html), [`kill(2)`](https://man7.org/linux/man-pages/man2/kill.2.html) | `signal(7)`: *Signal dispositions*, *Sending a signal*, and *Standard signals* table (specifically `SIGINT`, `SIGTERM`, `SIGKILL`, `SIGSTOP`, `SIGHUP`, `SIGPIPE`, `SIGSEGV`, `SIGCHLD` — skip real-time signals & deep syscall restart list). `kill(2)`: *Description*, *Return Value*, *Errors* | ~25–30 min | Exact signal semantics, default actions, why `SIGKILL`/`SIGSTOP` cannot be caught, and how `kill(2)` delivers signals. |
| **Mandatory 2** | Free | Linux `man-pages` (`proc(5)`, `proc_pid_status(5)`, `proc_pid_stat(5)`) + Node.js v26 Docs (`Signal events`, `process.kill`, `process.exitCode`) | [`proc(5)`](https://man7.org/linux/man-pages/man5/proc.5.html), [`proc_pid_status(5)`](https://man7.org/linux/man-pages/man5/proc_pid_status.5.html), [`proc_pid_stat(5)`](https://man7.org/linux/man-pages/man5/proc_pid_stat.5.html), [`Signal events`](https://nodejs.org/docs/latest-v26.x/api/process.html#signal-events), [`process.kill`](https://nodejs.org/docs/latest-v26.x/api/process.html#processkillpid-signal), [`process.exitCode`](https://nodejs.org/docs/latest-v26.x/api/process.html#processexitcode) | `proc_pid_status(5)` & `proc_pid_stat(5)`: targeted diagnostic fields (`State`, `VmSize`, `VmRSS`, `Threads`, `voluntary`/`nonvoluntary_ctxt_switches`, `minflt`, `majflt`). Node.js v26: `Signal events` (including why `SIGKILL`/`SIGSTOP` throw if registered!), `process.kill()`, and `process.exitCode` | ~25–30 min | Exact Linux `/proc` diagnostic fields and how Node.js bridges POSIX signals safely without replacing OS semantics. |
| **Supplementary** | Free | Brendan Gregg — *The USE Method: Linux Performance Checklist* | [`usemethod_linux.html`](https://www.brendangregg.com/USEmethod/use-linux.html) *(Note: conceptual overview only — deep Linux tools like `sar`/`mpstat`/`perf` belong to Phases 23 & 25)* | Skim CPU/Memory/I/O Utilization & Saturation rows only (`Context` bridge to Phases 23 & 25) | Optional | Preview of how Phase 3's OS observables scale into production Linux checklists later. |

#### Pre-Lab Exercise & Proposed Phase 3 Integrative Capstone Laboratory
- **Pre-Lab Exercise (`LO 8.1`, `LO 8.2` — ~25–30 min): Signal Dispositions, Exit Codes & Shutdown Audit.** Diagnose 4 realistic deployment/incident logs: (a) a worker process dies silently with exit status `137` during a traffic burst (zero JS exception logged)—explain why `137` proves `SIGKILL` termination (with OOM as only one hypothesis) and state what host/kernel evidence is required to corroborate OOM vs. external kill; (b) a server receives `SIGTERM`, prints `"shutting down"`, but hangs indefinitely waiting on an open timer until killed 30 s later with `137`; (c) a developer wraps `process.on('SIGKILL', cleanup)` assuming it will flush logs before OOM kill; (d) a CLI data-pipeline script piping output into `head -n 5` abruptly exits with `SIGPIPE`. Explain the exact OS signal mechanism for each.
- **Proposed Phase 3 Integrative Backend OS Diagnostic Capstone Lab (`phases/phase-03/lab/` — to be authored/executed ONLY when Phase 3 is unlocked, NOT now):**
  - **Strict Safety & Scope Boundaries:** Strictly unprivileged user-space execution on Ubuntu 24.04; zero `sudo`, zero kernel-parameter edits, zero arbitrary PID killing (scripts target ONLY their own spawned child PIDs!), memory strictly capped `<= 128 MiB`, CPU tests bounded `<= 10 s` and pinned via `taskset -c 0`, FD-leak tests isolated inside `bash -c 'ulimit -n 32 && ...'`, all temp files created in `/tmp` and deleted cleanly.
  - **Four Integrative Lab Stages (`~140–170 min` total):**
    1. **Stage 1 — Signal Control & Graceful vs. Forced Termination (`~30–35 min`, Block 8):** Build and test a tiny self-contained Node.js worker script that registers `SIGTERM` & `SIGINT` handlers (stopping new work, finishing a 1-second mock task, setting `process.exitCode = 0`), spawn it from a controller shell, send `SIGTERM` (`kill -TERM $PID` → verify exit code `0` & clean log), then spawn a second copy and send `SIGKILL` (`kill -KILL $PID` → verify immediate termination, missed cleanup, and shell exit code `137`).
    2. **Stage 2 — Controlled Multi-Symptom Backend Probe (`~40–50 min`, Blocks 1–7):** Run 3 self-contained, short-lived synthetic local backend worker scripts (each exhibiting one controlled OS anomaly without naming the cause upfront): (A) an FD-leaking file processor inside `ulimit -n 32`, (B) a lazy-allocating memory processor touching `96 MiB` of anonymous pages, and (C) a CPU/I/O-contended worker under `taskset -c 0` and synchronous `fsync` loop.
    3. **Stage 3 — Scientific Method + USE Evidence Collection (`~40–50 min`, Block 8 + Phase 1 B5):** For each synthetic workload, state a falsifiable hypothesis first, then gather raw empirical OS evidence using unprivileged `/proc/<PID>/{status,maps,fd}`, `ps`, `vmstat 1`, `/usr/bin/time -v`, and `strace -c` / `strace -T` on child commands—eliminating at least one wrong initial hypothesis via evidence.
    4. **Stage 4 — Evidence-Based OS Incident Diagnosis Report (`~30–35 min`, Integrative):** Synthesize all observed evidence into a concise structured diagnosis report mapping each symptom → raw kernel metric → underlying OS mechanism (Blocks 1–8) → concrete backend fix → explicit Phase 3 boundary (naming which production hardening steps belong to Phases 11, 23, 24, or 25).
- **Assessment Approach:** 3 inline Pause & Checks + Optional Closed-Book Phase 3 Synthesis Assessment (`~30–35 min`, strictly optional).

#### Flexible Study-Time Breakdown (Block 8)
- Guided Lesson + Pause & Checks 1–3: `~65–80 min` *(flexible priority)*
- Mandatory 1 (`signal(7)` & `kill(2)` targeted): `~25–30 min`
- Mandatory 2 (`proc(5)`/`proc_pid_status(5)`/`proc_pid_stat(5)` + Node.js v26 signal/kill/exitCode): `~25–30 min`
- Pre-Lab Exercise: `~25–30 min` | 4-Stage Integrative OS Diagnostic Capstone Lab: `~140–170 min`
- **Total Required Core (without optional assessment):** **`~280–340 min (~4.7–5.7 h)`**
- Optional Closed-Book Synthesis Assessment: `~30–35 min` *(optional)*
- **Total (with optional assessment):** **`~310–375 min (~5.2–6.25 h)`** *(Roadmap target: `6.0 h`)*

---

## 7. Master Resource Map & Live Verification Summary

All mandatory resources below are **100% free** and were **live-verified (`HTTP 200 OK`) on `2026-11-28`**, including exact PDF chapter/section headings, PDF page counts, man-page structures, Kernel MM headings, and Node.js v26.13.1 anchor IDs.

### 7.1 OSTEP Version 1.10 Assigned Reading Map (`https://pages.cs.wisc.edu/~remzi/OSTEP/`)

| Block | OSTEP Ch. | PDF File & Verified URL | Exact Assigned Subsections | Skipped / Deferred Parts | Approx. Pages | Est. Study Time |
| :---: | :---: | :--- | :--- | :--- | :---: | :---: |
| **1** | **Ch 2** | [`intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/intro.pdf) | `2.1–2.5`, `2.7` | Skip `2.6` (History) | ~11 pp | 25–30 min |
| **1** | **Ch 6** | [`cpu-mechanisms.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-mechanisms.pdf) | `6.1–6.2`, `6.5` (LDE) | Defer `6.3` → B3; skip `6.4` | ~6.5 pp | 20–25 min |
| **2** | **Ch 4** | [`cpu-intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf) | `4.1–4.6` | Skip simulation homework | ~9 pp | 25–30 min |
| **2** | **Ch 5** | [`cpu-api.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf) | `5.1–5.7` | Skip simulation homework | ~10 pp | 25–30 min |
| **3** | **Ch 6** | [`cpu-mechanisms.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-mechanisms.pdf) | `6.3` (*Switching Between Processes*) | `6.1–6.2` done in B1; skip `6.4` | ~4.5 pp | 10–15 min |
| **3** | **Ch 7** | [`cpu-sched.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched.pdf) | `7.1–7.10` | Skip simulation homework | ~10 pp | 25–30 min |
| **3** | **Ch 8** | [`cpu-sched-mlfq.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-mlfq.pdf) | `8.1–8.4`, `8.6` | Skip `8.5` tuning knobs | ~7 pp | 20–25 min |
| **3** | **Ch 10** | [`cpu-sched-multi.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-multi.pdf) | `10.1`, `10.3–10.5`, skim `10.6` | Skip `10.2` lock code (Ph 6) | ~6 pp | 15–20 min |
| **4** | **Ch 13** | [`vm-intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-intro.pdf) | `13.1–13.5` | Skip homework | ~6.5 pp | 15 min |
| **4** | **Ch 14** | [`vm-api.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-api.pdf) | `14.1–14.3`, `14.5` | Skip `14.4` C bugs, `14.6` | ~4 pp | 10–15 min |
| **4** | **Ch 15** | [`vm-mechanism.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-mechanism.pdf) | `15.1–15.4` (skim `15.5`) | Skip simulation homework | ~7 pp | 15–20 min |
| **4** | **Ch 18** | [`vm-paging.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-paging.pdf) | `18.1–18.4`, `18.6` | Skip `18.5` asm trace | ~8 pp | 20–25 min |
| **4** | **Ch 19** | [`vm-tlbs.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-tlbs.pdf) | `19.1–19.2`, `19.4–19.5`, `19.8` | Skip `19.3` SW-TLB, `19.7` MIPS | ~7 pp | 20–25 min |
| **5** | **Ch 21** | [`vm-beyondphys.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-beyondphys.pdf) | `21.1–21.6` | Skip measurement homework | ~7 pp | 20–25 min |
| **5** | **Ch 22** | [`vm-beyondphys-policy.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-beyondphys-policy.pdf) | `22.1, 22.3, 22.5, 22.8–22.9, 22.11` | Skip `22.2, 22.4, 22.6–22.7` | ~6 pp | 20–25 min |
| **5** | **Ch 23** | [`vm-complete.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-complete.pdf) | `23.2` (Linux Addr Space, Page Table, Page Cache) | Skip `23.1` VAX/VMS, Large Pages, Security | ~5 pp | 15–20 min |
| **6** | **Ch 39** | [`file-intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/file-intro.pdf) | `39.1–39.4, 39.6, 39.9–39.10, 39.14–39.15` | Move `39.7` → B7; defer `39.16–39.17` → Ph 23 | ~11 pp | 35–40 min |
| **7** | **Ch 36** | [`file-devices.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/file-devices.pdf) | `36.1–36.5`, `36.7`, `36.10` | Skip `36.6, 36.8` IDE, `36.9` | ~8 pp | 25–30 min |
| **7** | **Ch 39** | [`file-intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/file-intro.pdf) | `39.7` (*Writing Immediately With `fsync()`*) | Rest covered in B6 / deferred | ~1 p | 5–10 min |
| **Total** | **17 distinct mandatory Chs (19 block assignments)** | **OSTEP Version 1.10** | **Surgically curated sections** | **Ch 26–34, 40–45, 48–51 deferred** | **~134.5 pp** | **~365–455 min (~6.1–7.6 h)** |

*Chapter-count note: The mandatory reading map covers **17 distinct OSTEP v1.10 chapters** (`Ch 2, 4, 5, 6, 7, 8, 10, 13, 14, 15, 18, 19, 21, 22, 23, 36, 39`) across **19 block assignments** (`Ch 6` is split across Blocks 1 & 3; `Ch 39` is split across Blocks 6 & 7). Including the **2 optional supplementary chapters** linked in Section 6 (`Ch 9` in Block 3 and `Ch 20` in Block 4), **19 distinct OSTEP chapter PDFs** are referenced across this dossier.*

### 7.2 Supporting Authoritative Resources Map (Linux `man-pages`, Linux Kernel Docs, Node.js v26 Docs)

| Source Family | Verified Resource & URL | Assigned Block(s) | Exact Targeted Scope | Role |
| :--- | :--- | :---: | :--- | :--- |
| **Linux `man-pages`** | [`syscalls(2)`](https://man7.org/linux/man-pages/man2/syscalls.2.html), [`strace(1)`](https://man7.org/linux/man-pages/man1/strace.1.html) | B1 | *Description* intro & `-c`/`-e` flags | Primary Linux reference |
| **Linux `man-pages`** | [`fork(2)`](https://man7.org/linux/man-pages/man2/fork.2.html), [`execve(2)`](https://man7.org/linux/man-pages/man2/execve.2.html), [`wait(2)`](https://man7.org/linux/man-pages/man2/wait.2.html) (`waitpid(2)`) | B2 | *Description* & *Return Value* (incl. CoW & zombie reaping) | Primary Linux reference |
| **Linux `man-pages`** | [`sched(7)`](https://man7.org/linux/man-pages/man7/sched.7.html) | B3 | *Scheduling policies*, *`SCHED_OTHER`*, *The nice value* | Primary Linux reference |
| **Linux `man-pages`** | [`mmap(2)`](https://man7.org/linux/man-pages/man2/mmap.2.html), [`proc_pid_maps(5)`](https://man7.org/linux/man-pages/man5/proc_pid_maps.5.html) | B4 | Core mapping flags (`MAP_PRIVATE`/`SHARED`/`ANONYMOUS`) & `/proc/PID/maps` format | Primary Linux reference |
| **Linux `man-pages`** | [`proc_pid_status(5)`](https://man7.org/linux/man-pages/man5/proc_pid_status.5.html), [`proc_pid_oom_score(5)`](https://man7.org/linux/man-pages/man5/proc_pid_oom_score.5.html) | B5, B8 | `State`, `Threads`, `VmSize`, `VmRSS`, `RssAnon`, `RssFile`, context switches, `oom_score` | Primary Linux reference |
| **Linux `man-pages`** | [`open(2)`](https://man7.org/linux/man-pages/man2/open.2.html), [`close(2)`](https://man7.org/linux/man-pages/man2/close.2.html), [`proc(5)`](https://man7.org/linux/man-pages/man5/proc.5.html), [`proc_pid_fd(5)`](https://man7.org/linux/man-pages/man5/proc_pid_fd.5.html) | B6, B7 | *Description*, *Open file descriptions*, `O_NONBLOCK`, *Caveats*, `/proc/PID/fd` | Primary Linux reference |
| **Linux `man-pages`** | [`read(2)`](https://man7.org/linux/man-pages/man2/read.2.html), [`write(2)`](https://man7.org/linux/man-pages/man2/write.2.html), [`epoll(7)`](https://man7.org/linux/man-pages/man7/epoll.7.html) | B7 | Blocking/non-blocking semantics, short counts, `EAGAIN`/`EINTR`; `epoll(7)` intro only (`Awareness`) | Primary Linux reference |
| **Linux `man-pages`** | [`signal(7)`](https://man7.org/linux/man-pages/man7/signal.7.html), [`kill(2)`](https://man7.org/linux/man-pages/man2/kill.2.html), [`proc_pid_stat(5)`](https://man7.org/linux/man-pages/man5/proc_pid_stat.5.html) | B8 | *Signal dispositions*, key *Standard signals*, `kill(2)` basics, `minflt`/`majflt` fields | Primary Linux reference |
| **Linux Kernel Docs** | [Memory Management Concepts Overview](https://docs.kernel.org/admin-guide/mm/concepts.html) | B4, B5, B7 | `#virtual-memory-primer`, `#anonymous-memory`, `#page-cache`, `#reclaim`, `#oom-killer` *(skip `#huge-pages`, `#zones`, `#nodes`, `#compaction`)* | Accessible official kernel overview |
| **Node.js v26 Docs** *(v26.13.1 verified)* | [`process.html`](https://nodejs.org/docs/latest-v26.x/api/process.html) (`#processpid`, `#processppid`, `#processmemoryusage`, `#processmemoryusagerss`, `#processstdoutfd`, `#signal-events`, `#processkillpid-signal`, `#processexitcode`) | B2, B4, B5, B6, B8 | Strictly selected OS-facing APIs only—never replaces OSTEP or Linux docs | Contextual backend bridge |
| **Commercial Supplementary** *(Optional)* | Michael Kerrisk, *The Linux Programming Interface* (No Starch Press, 2010) *(exact page numbers `[Unverified]`)* | B2, B6–B8 | Optional deep-dive reference for learners who own the book | Supplementary only |

---

## 8. Practical Experiment Strategy & Linux Safety Guardrails

### 8.1 Target Hardware & OS Profile (Live-Verified on Learner's Machine `2026-11-28`)
- **OS & Kernel:** Ubuntu 24.04.4 LTS (`noble`), Linux Kernel `7.0.0-44-generic` (`x86_64`, `PREEMPT_DYNAMIC`).
- **Hardware:** AMD Ryzen 7 7700 (8 physical cores / 16 hardware threads; L1d 256 KiB total, L2 8 MiB total, L3 32 MiB), 16 GB physical RAM (`~14 GiB` usable after integrated GPU reservation + `4.0 GiB` swap), `4096`-byte (`4 KiB`) page size.
- **Verified Unprivileged Tooling Installed:** `gcc 13.3.0`, `strace 6.8`, `ps`, `free`, `vmstat`, `lsof`, `taskset`, `/usr/bin/time`, `stat`, `readlink`, `kill`, `/proc/self/*`, and `node` *(see Section 12.1 note regarding local `v24.14.0` vs. pinned Phase 2/3 Node.js 26 Active LTS)*.
- **Two Critical Ubuntu 24.04 Live Discoveries That Shape Experiment Design:**
  1. **`kernel.yama.ptrace_scope = 1`:** Attaching `strace -p <PID>` to an already-running sibling process fails with `EPERM` without root. Therefore, **all Phase 3 `strace` experiments launch the target command directly as a child of `strace`** (e.g., `strace -c <cmd>`, `strace -tt -T <cmd>`), which works 100% unprivileged without `sudo` or modifying `sysctl`!
  2. **`ulimit -Sn = 1048576`:** Modern Ubuntu 24.04 shells inherit a soft open-file limit of `1,048,576`. Therefore, **all `EMFILE` FD-leak experiments run inside an isolated subshell (`bash -c 'ulimit -n 32 && ...'`)**, reproducing the exact kernel error safely with ~25 temp files without touching global limits!

### 8.2 Non-Negotiable Safety Controls
Every future Phase 3 experiment and lab script must enforce all six controls:
1. **Zero `sudo` / Zero Kernel Mutation:** Never run privileged commands, never edit `/proc/sys/*` or `sysctl`, never load kernel modules, never change global `ulimit`.
2. **Strict Memory Cap (`<= 128 MiB`):** Never allocate near host RAM limits, never trigger host swap thrashing or the host OOM killer. Demand paging and minor page faults are demonstrated safely at `64–128 MiB` (`< 1%` of 16 GB RAM).
3. **Strict CPU & Duration Bounds (`<= 10–15 s`):** CPU contention experiments pin at most 2–3 tiny workers to a single core via `taskset -c 0` with hard timeouts (`<= 10 s`), keeping the remaining hardware threads free of experiment-generated CPU contention so normal desktop/OS activity remains responsive.
4. **Strict Process-Targeting Isolation:** Signal and `kill` experiments send signals **only** to the exact child PID spawned and printed by the learner's own experiment script—never `killall`, never wildcard `pkill`, never unrelated PIDs.
5. **Clean `/tmp` Hygiene:** Any temp files created for FD or `fsync` labs are tiny (`< 1 MiB`), placed inside a dedicated `/tmp/phase3-lab-*` directory, and deleted on completion.
6. **Minimal Inline-Annotated C Policy:** Small C files (`~20–35 lines`, compiled via `gcc -Wall -Wextra`) are used strictly in Blocks 2 (`fork`/`waitpid`), 4 (virtual address isolation), 5 (`mmap` demand paging), and 7 (`pipe` + `O_NONBLOCK` `read`) where direct POSIX syscall visibility beats a high-level runtime wrapper. Every C line (headers, `pid_t`, pointers, `sizeof`, `-1`/`errno`) must be explained inline in natural Bangla—zero C prerequisite assumed!

### 8.3 Experiment Progression Matrix Across All 8 Blocks

| Block | Experiment Focus | Language / Tooling | Safety Cap / Control | Core Mechanism Proven Empirically |
| :---: | :--- | :--- | :--- | :--- |
| **1** | Syscall tracing (`echo`, `node`, unbuffered vs. batched writes) | `strace -c`, `node` | Child tracing only (`yama=1` safe), `/tmp` cleanup | Privilege transitions & syscall overhead vs. batching |
| **2** | `fork()`/`waitpid()` zombie lifecycle + Node.js process threads | Annotated C (~30 LOC) + `node` + `ps` + `/proc/PID/task` | Single child, 5 s auto-reap | `fork`/`waitpid`, `Z` state cleanup, multi-thread Node process |
| **3** | Single-core CPU contention & voluntary vs. involuntary switches | `node` + `taskset -c 0` + `/proc/PID/status` + `vmstat 1` | Pinned to core 0 only, `<= 6 s` auto-exit | Preemption, run-queue saturation, response-time jitter |
| **4** | Virtual memory layout (`/proc/PID/maps`) & address isolation | `node` + annotated C (~20 LOC) + `/proc/PID/{maps,status}` | Read-only inspection + tiny stack var | `VmSize` vs. `VmRSS`, `r-xp`/`rw-p`, isolated virtual spaces |
| **5** | Demand paging & minor faults + `free -h` / `oom_score` | Annotated C (~30 LOC) + `node` + `/usr/bin/time -v` + `free -h` | Strict `<= 128 MiB` (`<1%` RAM), zero OOM trigger | Lazy allocation, approximate `minflt` jump on touch (4 KiB baseline ~32,768; varies with THP/implementation), `available` vs. `free` |
| **6** | `/proc/PID/fd`, `(deleted)` open file, & subshell `EMFILE` | `node` + `lsof` + `/proc/PID/fd` + `bash -c 'ulimit -n 32'` | Subshell FD cap `32`, `/tmp` cleanup | 3-tier FD table, `unlink` refcount, `EMFILE` leak & fix |
| **7** | Blocking vs. `O_NONBLOCK` pipe read + `write()` vs. `fsync()` | Annotated C (~35 LOC) + `node` + `strace -tt -T` | `1.5 s` pipe wait; 500×256 B `/tmp` writes | Kernel sleep vs. `EAGAIN`; measured buffered `write()` vs. `fsync()` behavior (filesystem/device/writeback dependent) |
| **8** | **4-Stage Capstone:** Signals (`SIGTERM`/`SIGKILL`) + Multi-Symptom OS Triage | `node` + `kill` + `/proc` + `ps` + `vmstat` + `strace` | Self-spawned PIDs only, `<= 128 MiB`, `<= 10 s` | Exit `0`/`137`/`143`, USE + Scientific Method OS diagnosis |

---

## 9. Coverage Mapping Against Proposed Phase 3 Mastery Outcomes (1–9)

| # | Proposed Phase 3 Mastery Outcome | Primary Block(s) | Supporting Block(s) | Mapped Block LOs | Core Free Mandatory Sources | Verification / Demonstration Path |
| :---: | :--- | :---: | :---: | :--- | :--- | :--- |
| **1** | OS purpose, user vs. kernel privilege boundary, LDE & syscalls | Block 1 | Block 7, Block 8 | `LO 1.1–1.4` | OSTEP Ch `2`, `6.1–6.2`; `syscalls(2)`, `strace(1)` | `strace` child syscall lab + privilege/cost reasoning |
| **2** | Process lifecycle (`fork`/`execve`/`waitpid`, states, zombies) & process-vs-thread structure | Block 2 | Block 3, Block 8 | `LO 2.1–2.4` | OSTEP Ch `4`, `5`; `fork(2)`, `execve(2)`, `wait(2)`; Node `pid`/`ppid` | Annotated C zombie/reap lab + `/proc/PID/task` audit |
| **3** | Preemptive CPU scheduling, context switching, MLFQ, multi-core affinity & saturation | Block 3 | Block 7, Block 8 | `LO 3.1–3.4` | OSTEP Ch `6.3`, `7`, `8`, `10`; `sched(7)` | `taskset -c 0` contention lab + policy trace & starvation audit |
| **4** | Virtual address spaces, paging translation, PTEs, TLB & `VmSize` vs. `VmRSS` | Block 4 | Block 5, Block 8 | `LO 4.1–4.4` | OSTEP Ch `13`, `14`, `15`, `18`, `19`; `mmap(2)`, `proc_pid_maps(5)`, Kernel MM | `/proc/PID/maps` & isolation lab + VPN→PFN & TLB locality math |
| **5** | Demand paging, minor/major faults, CoW, swap, reclaim, thrashing, OOM & Linux memory | Block 5 | Block 7, Block 8 | `LO 5.1–5.4` | OSTEP Ch `21`, `22`, `23.2`; Kernel MM (`reclaim`/`oom`); Node `memoryUsage()` | Safe 128 MiB minor-fault lab + `free -h`/CoW/OOM diagnosis |
| **6** | Files, inodes, directories, stdio, 3-tier FD table, `unlink` & `EMFILE` leaks | Block 6 | Block 7, Block 8 | `LO 6.1–6.4` | OSTEP Ch `39` (selected); `open(2)`, `close(2)`, `proc_pid_fd(5)` | `/proc/PID/fd` `(deleted)` + safe `ulimit -n 32` `EMFILE` lab |
| **7** | Hardware I/O (interrupts/DMA), Page Cache vs. `fsync`, blocking vs. `O_NONBLOCK`, `epoll` concept | Block 7 | Block 6, Block 8 | `LO 7.1–7.4` | OSTEP Ch `36`, `39.7`; Kernel `#page-cache`; `read(2)`, `write(2)`, `epoll(7)` | `strace -tt -T` blocking vs. `EAGAIN` + `write`/`fsync` timing lab |
| **8** | Signals (`SIGTERM`/`SIGKILL` etc.), dispositions, exit codes (`137`/`143`) & Node.js cleanup | Block 8 | Block 2, Block 5 | `LO 8.1–8.2` | `signal(7)`, `kill(2)`; Node.js v26 `Signal events`, `exitCode` | Signal audit + Capstone Stage 1 (`SIGTERM` vs. `SIGKILL`) |
| **9** | Scientific Method + USE Method backend OS diagnosis with safe Linux observables | Block 8 | Blocks 1–7 | `LO 8.3–8.4` | `proc(5)`, `proc_pid_status(5)`, `proc_pid_stat(5)` + Phase 1 B5 USE | 4-Stage Phase 3 Integrative Backend OS Diagnostic Capstone |

---

## 10. Teaching Standards, Authoring Contract & Technical Accuracy Guardrails

### 10.1 Teaching & Authoring Contract for Future Phase 3 Blocks
When ChatGPT approves this dossier and authorizes authoring Phase 3 teaching materials, every block file (`phases/phase-03/materials/block-01-...` through `block-08-...`) **must** uphold:
1. **Natural Bangla-English Code-Switching (`DEC-004`):** Explain concepts, mechanisms, cause-and-effect, trade-offs, and exercises primarily in natural Bangla prose using natural Bangla-English code-switching—retaining common everyday English words where natural in addition to preserving original English technical terms (`System Call`, `Context Switch`, `Page Fault`, `Resident Set Size`, `File Descriptor`, `Non-Blocking I/O`, `OOM Killer`), code, commands, `/proc` fields, and authoritative English source titles (explaining each technical term clearly in Bangla on first introduction); avoid forced literal Bangla translations and phonetic Bangla transliteration of English words (Romanized Bangla).
2. **Connected Guided Discovery (`DEC-003` & `teaching-material-design/SKILL.md`):** Open straight into a vivid backend/Linux situation → let the learner feel the puzzle → build intuition → introduce formal terms *after* intuition → unpack step-by-step mechanisms, invariants, trade-offs, and production failure modes → transition causally to the next section. No upfront terminology dumps, prerequisite boxes, or reference-manual clutter.
3. **Flexible Study Time Guide Placement:** Place the Study Time Guide immediately after the `#` block title and before the guided lesson, matching every itemized reading, experiment, exercise, optional assessment, and total (with and without optional assessment) in Section 6.
4. **Native Examples Over Forced Business Metaphors:** Per `learning-standards.md`, do **not** artificially force the promotional-code store onto low-level paging bits or timer interrupts—use crisp, authentic Linux/server scenarios (e.g., an API worker process, a background CSV exporter, a log writer, a PostgreSQL-style `fsync` write) and reconnect back to our online-store backend naturally in diagnostic exercises and Block 8!
5. **Inline C Explanations:** Whenever a minimal C snippet (`~20–35 lines`) appears in Blocks 2, 4, 5, or 7, explain every unfamiliar C syntax token inline in Bangla so the learner stays focused 100% on the OS mechanism.
6. **Deferred Self-Check Guidance & Learner Ownership:** Include Self-Check Guidance strictly at the end of each block file and strictly for inline Pause & Checks 1–3; never include answer keys for mandatory experiments, transfer exercises, the Block 8 Capstone Lab, or Optional Closed-Book Assessments.

### 10.2 Technical Accuracy Guardrails (Verified Invariants)
Future Phase 3 teaching blocks must maintain these exact technical distinctions:
1. **Modern Linux Scheduler Accuracy (Kernel 6.6+ / Ubuntu 24.04 HWE 7.0):** OSTEP v1.10 (2008–2023) references Linux's **CFS** (`Completely Fair Scheduler`) in Ch 9.7 and 10.6; starting in Linux 6.6 (including the learner's Ubuntu 24.04 Kernel `7.0.0-44-generic`), Linux replaced CFS with **EEVDF (Earliest Eligible Virtual Deadline First)** under the same `SCHED_OTHER`/`SCHED_NORMAL` policy. State this accurately at `Awareness` level without teaching scheduler source internals.
2. **`proc(5)` Split in Modern `man-pages`:** In current Linux `man-pages` (6.x+), per-PID `/proc/<PID>/*` entries live on dedicated pages (`proc_pid_maps(5)`, `proc_pid_status(5)`, `proc_pid_fd(5)`, `proc_pid_stat(5)`, `proc_pid_oom_score(5)`) linked from `proc(5)`. Cite the exact verified page names.
3. **Virtual Memory vs. Physical RAM (`VmSize` ≠ `VmRSS` ≠ `PSS`):** Never equate `VmSize` with physical RAM consumption, and note clearly that `VmRSS` includes shared library mappings (`RssFile`) alongside private anonymous memory (`RssAnon`), so summing `VmRSS` across processes double-counts shared pages!
4. **Linux `free -h` Accuracy:** `free` shows completely unused frames; `buff/cache` holds reclaimable Page Cache and slab caches; **`available`** is the kernel's realistic estimate of memory available for starting new workloads without swapping. Never teach "low `free` means out of memory."
5. **Open File Description vs. File Descriptor:** Keep Linux `open(2)`'s exact distinction crisp: the **file descriptor** (`int`, e.g., `3`) is a per-process table slot pointing to a kernel **open file description** (which holds the file offset and status flags like `O_NONBLOCK` / `O_APPEND`). Two `open()` calls create two open file descriptions; `fork()` and `dup()` share one!
6. **Regular Files vs. Pipes/Sockets Under `O_NONBLOCK`, `poll(2)`, and `epoll(7)`:** Never claim that `O_NONBLOCK` makes ordinary Linux regular-file I/O asynchronous or non-blocking. Clarify the exact Linux kernel semantics: `poll(2)` reports regular files as immediately ready (even though subsequent `read()`/`write()` may still block on storage I/O or page faults), whereas `epoll_ctl(2)` generally rejects regular files with `EPERM`—directly motivating why runtimes like Node.js/libuv later use a worker thread pool for regular file I/O in Phase 11!
7. **Linux Process State `R` & `TGID`/`PID` vs. `TID` Precision:** In Linux `ps` and `/proc/<PID>/status`, state **`R` denotes both running and runnable** (combining OSTEP's conceptual `Running` and `Ready` states into one code—never map `R` exclusively to `Running`). Similarly, all threads inside one Linux process share one **Thread Group ID (`TGID`)**—exposed in user space as the process **`PID`**—while each individual thread has its own kernel **Thread ID (`TID`)** visible under `/proc/<PID>/task/<TID>`.
8. **Exit Code `137` Caveat:** In POSIX shells, `128 + 9 = 137` indicates termination by **`SIGKILL`**. While the Linux OOM killer delivers `SIGKILL` (producing `137`), an operator or orchestrator sending manual `kill -9` *also* produces `137`—so engineers must corroborate OOM kills with kernel logs/metrics rather than assuming every `137` is an OOM!
9. **Node.js Runtime Release-Track Precision:** Verified against [nodejs/Release](https://github.com/nodejs/Release) `schedule.json` on `2026-11-28`: **Node.js 26 (`Lithium`, docs verified at `v26.13.1`)** is **Active LTS** (`2026-10-28` to `2027-10-20`, EOL `2029-04-30`), distinct from Node.js 22/24 (Maintenance LTS) and Node.js 27 (Alpha). Always distinguish Active LTS from Alpha/Current release lines.

---

## 11. Assessment & Phase-Completion Expectations

Consistent with `AGENTS.md`, `PROGRESS.md`, and `.agents/skills/phase-checkpoint-handoff/SKILL.md`:

- **Learner-Controlled Block Pacing:** Formal closed-book assessments at the end of Blocks 1–8 are **strictly optional** and never block studying the next block (`Studied — mastery not formally verified` vs. `Mastery verified`).
- **Required Evidence for Phase 3 Checkpoint Approval (ChatGPT Final Authority):**
  1. Learner-authored conceptual notes, inline Pause & Checks, and mandatory transfer exercises across Blocks 1–8 (`phases/phase-03/notes/block-01-notes.md` through `block-08-notes.md`).
  2. Verified, reproducible, safe Linux experiment outputs and annotated scripts from Blocks 1–7 and the 4-stage Block 8 Capstone Diagnostic Lab in `phases/phase-03/lab/` (with raw `/proc`, `ps`, `free`, `vmstat`, `strace`, and signal exit-code evidence—never fabricated outputs).
  3. Updated, learner-verified entries in `BACKEND_KNOWLEDGE_INDEX.md` covering core Phase 3 OS concepts using the relational schema and the three official depth levels (`Awareness`, `Working Understanding`, `Practical Competence`).
  4. Compiled mastery handoff packet in `phases/phase-03/checkpoint.md` honestly acknowledging any remaining conceptual gaps.
  5. **Explicit `approved` sign-off from ChatGPT (Curriculum Owner & Instructor)** before unlocking Phase 4 (*Computer Networking*).

---

## 12. Risks, Unresolved Decisions & Resource-Access Caveats

### 12.1 Verified Technical Caveats & Local Environment Notes (`2026-11-28`)
1. **Local Node.js Version Gap (`v24.14.0` Installed vs. Node.js 26 Active LTS Target):** Local inspection shows `/home/sifat/.local/bin/node` currently symlinks to `~/.nvm/versions/node/v24.14.0/bin/node` (`Maintenance LTS`), while Phase 2 pinned **Node.js 26 Active LTS (`Lithium`)** and this Phase 3 dossier aligns with **Node.js v26 documentation (`v26.13.1`)**. *Mitigation:* All contextual Phase 3 Node.js APIs (`process.pid`, `process.ppid`, `process.memoryUsage()`, stdio `.fd`, `SIGTERM` handlers, `process.exitCode`) behave consistently across Node.js 24 and 26, and the learner should install Node.js 26 Active LTS via `nvm` before starting Phase 2/3 labs.
2. **Ubuntu 24.04 `yama.ptrace_scope = 1`:** Blocks unprivileged `strace -p <PID>` on sibling processes. *Mitigation:* Every Phase 3 `strace` experiment invokes `strace <command>` directly on a child process (verified working unprivileged on this host!).
3. **Ubuntu 24.04 Default `ulimit -Sn = 1048576`:** Makes default `EMFILE` reproduction impractical without a local cap. *Mitigation:* Block 6 & Block 8 isolate `EMFILE` labs inside `bash -c 'ulimit -n 32 && ...'` (zero host side effects).
4. **8-Core / 16-Thread CPU Masking Contention:** On Ryzen 7 7700, 2 CPU workers run in parallel without preemption unless pinned. *Mitigation:* Block 3 & Block 8 use unprivileged `taskset -c 0` (verified working locally) to isolate controlled contention cleanly onto core 0.
5. **Resource Verification Catch (`proc_pid_smaps_rollup(5)` = HTTP 404):** During live verification of `man7.org`, `proc_pid_smaps_rollup.5.html` returned `404 Not Found` (unlike `proc_pid_smaps.5.html`, `proc_pid_maps.5.html`, `proc_pid_status.5.html`, `proc_pid_fd.5.html`, `proc_pid_stat.5.html`, and `proc_pid_oom_score.5.html`, which all returned `200 OK`). Excluded cleanly so zero broken links exist in this dossier.
6. **External URL Longevity Risk:** `pages.cs.wisc.edu/~remzi/OSTEP/` is stable (OSTEP v1.10 hosted directly by the authors), and `man7.org`, `docs.kernel.org`, and `nodejs.org/docs/latest-v26.x/` are primary upstream hosts. Still, if `man7.org` is ever offline during study, every referenced Linux manual page is also accessible offline via local `man 2 fork`, `man 7 signal`, etc. on Ubuntu 24.04 (`manpages` / `manpages-dev`).

### 12.2 Key Pedagogical Risks & Mitigations
- **Risk 1 — C Syntax Friction Distracting From OS Mechanisms:** Even small C files can derail a JS-focused learner if pointers, headers, or `errno` go unexplained. *Mitigation:* Restrict C strictly to 4 tiny, complete, self-contained files (`20–35 lines`, Blocks 2, 4, 5, 7) with every C construct explained inline in Bangla, and use Node.js + standard Linux CLI inspection everywhere else.
- **Risk 2 — Scope Creep Into Concurrency (Phase 6) or Linux Ops (Phase 23):** Threads, `epoll`, permissions, and `cgroups` sit right on Phase 3's borders. *Mitigation:* Enforce the Section 3.1 Anti-Overlap Matrix strictly—keep threads to structural process-vs-thread distinction, `epoll` to conceptual `Awareness`, and defer OSTEP Ch 26–34, 40–45, and 48–51 explicitly.
- **Risk 3 — Reading Fatigue Across 17 Mandatory OSTEP Chapters:** Assigning full OSTEP chapters would exceed 36 hours easily. *Mitigation:* Every mandatory chapter is pruned to targeted subsections (`17` distinct mandatory chapters across `19` block assignments, `~134.5` total pages ≈ `~6.1–7.6 hours` across the entire 36-hour phase; `2` optional supplementary chapters bring total distinct OSTEP chapters referenced to `19` without adding to the mandatory budget), keeping primary textbook reading around ~17–21% of the total budget so active reasoning and hands-on Linux experiments dominate.

### 12.3 Unresolved Decisions Awaiting ChatGPT (Curriculum Owner & Instructor) Approval
Before authorizing any Phase 3 teaching blocks when Phase 3 unlocks, ChatGPT should review and confirm:
1. **Approval of the 9 Proposed Phase 3 Mastery Outcomes (Section 4)** and validation of the **8-Block Architecture & 36-Hour Allocation** (`3, 4, 4, 5, 5, 4, 5, 6 = 36.0 h`; honest required core `28.0–34.2 h`, midpoint `~31.1 h`).
2. **Confirmation of the Minimal Annotated C Scope:** Confirm using 4 tiny, inline-explained C snippets (Blocks 2, 4, 5, 7) for raw POSIX syscall observation (`fork`/`waitpid`, address isolation, `mmap` demand paging, `pipe` + `O_NONBLOCK`) alongside Node.js v26 and Linux CLI inspection.
3. **Confirmation of Block 7 `epoll(7)` Depth (`Awareness` Only):** Confirm introducing OS I/O readiness notification strictly at `Awareness` level in Block 7 (without `epoll` C coding or event-loop architecture) so Phase 6 and Phase 11 remain cleanly bounded.
4. **Phase 3 Runtime Pin:** Confirm carrying forward **Node.js 26 Active LTS (`Lithium`)** as the Phase 3 contextual runtime reference (matching Phase 2's approved pin and the prompt's `latest-v26.x` documentation scope).
5. **Governance Boundary Reminder:** Per `AGENTS.md` Boundary #2 and `DEC-001` #5, **no Phase 3 teaching materials, lab files, notes, `README.md`, `checkpoint.md`, or `PROGRESS.md` / `NEXT_SESSION.md` updates will be created** until Phases 1–2 are completed/approved and ChatGPT explicitly approves this dossier and authorizes Phase 3 kickoff.
