# Phase 3, Block 1 — The Operating System, Kernel, and System Calls

## Study Time Guide

| Activity | Assigned Scope | Estimated Time |
| :--- | :--- | :---: |
| **Guided Lesson + Pause & Checks 1–3** *(main priority)* | পুরো guided lesson মনোযোগ দিয়ে পড়া এবং `phases/phase-03/notes/block-01-notes.md`-এ *(Phase 3 unlock হওয়ার পর বা আপনার নিজস্ব study notebook-এ)* Pause & Checks 1–3 লেখা | **~45–55 min** *(flexible — limit নয়)* |
| **Mandatory 1:** OSTEP v1.10 — Chapter 2 (*Introduction to Operating Systems*) | [`intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/intro.pdf) — কেবল Sections `2.1`, `2.2`, `2.3`, `2.4`, `2.5`, এবং `2.7` (~11 pages; *explicitly skip `2.6 Some History`*) | ~25–30 min |
| **Mandatory 2:** OSTEP v1.10 — Chapter 6 (*Mechanism: Limited Direct Execution*) | [`cpu-mechanisms.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-mechanisms.pdf) — কেবল Sections `6.1`, `6.2`, এবং `6.5` (LDE অংশ; ~6.5 pages; *defer `6.3` to Block 3; skip `6.4`*) | ~20–25 min |
| **Mandatory 3:** Linux `man-pages` — `syscalls(2)` & `strace(1)` | [`syscalls(2)`](https://man7.org/linux/man-pages/man2/syscalls.2.html) (*Description*-এর প্রথম ৪টি paragraph — syscall বনাম `glibc` wrapper, `errno`; full syscall table skip করুন) এবং [`strace(1)`](https://man7.org/linux/man-pages/man1/strace.1.html) (*Description* + `-c` ও `-e trace=...` flags) | ~10–15 min |
| **Practical Experiment 1 (Mandatory):** Observing the User/Kernel Boundary Live with `strace` | Ubuntu-তে unprivileged `strace -c` দিয়ে `/bin/echo` বনাম `node` তুলনা এবং `/tmp`-এ 1,000 unbuffered বনাম 1 batched `fs.writeSync` observation | ~20–25 min |
| **Transfer Exercise 1 (Mandatory):** Privilege Boundary & Syscall Cost Reasoning | ৩টি বাস্তব backend scenario-তে privilege boundary, Limited Direct Execution, syscall বনাম library call, এবং kernel-entry paths বিশ্লেষণ | ~25–30 min |
| **Total (Required Core — without Optional Assessment)** | Guided Lesson + ৩টি Mandatory Resource + Practical Experiment 1 + Transfer Exercise 1 | **~145–180 min (~2.4–3.0 h)** |
| **Optional Closed-Book Assessment** *(optional)* | Notes/tabs বন্ধ রেখে ৫টি synthesis প্রশ্নের উত্তর লেখা | ~20–25 min *(optional)* |
| **Total (with Optional Assessment)** | Required Core + Optional Closed-Book Assessment | **~165–205 min (~2.75–3.4 h)** |

> **Planning Note:** Approved Phase 3 Research Dossier-এ Block 1-এর target allocation হলো **3 hours (180 minutes)**—যা একটি flexible planning reference, কোনো strict time limit নয়। দ্রুত শেষ করার চেয়ে প্রতিটি mechanism ও cause-and-effect সম্পর্ক গভীরভাবে understand করাই main priority; প্রয়োজনে Guided Lesson বা `strace` experiment-এ অনুমানের চেয়ে বেশি সময় লাগা একদম স্বাভাবিক। প্রতিটি reading estimate কেবল নির্ধারিত sections-এর জন্য, পুরো বই বা manual page পড়ার জন্য নয়। **Optional Closed-Book Assessment** সম্পূর্ণ optional এবং কোনো progression gate নয়। *(Governance note: এই Block 1 lesson-টি Curriculum Owner-এর অনুমোদিত one-time pre-study exception হিসেবে তৈরি; Phase 3 এখনো globally unlocked নয় এবং live learning continuity `PROGRESS.md` ও `NEXT_SESSION.md`-এ অপরিবর্তিত রয়েছে।)*

---

একটি বাস্তব Linux production server-এর দৃশ্য কল্পনা করুন। ওই একটিমাত্র machine-এর ওপর একই মুহূর্তে অন্তত তিনটি আলাদা backend program চলছে:
1. আমাদের online store-এর **Node.js checkout API**,
2. একটি **PostgreSQL database**, যার memory-র ভেতর গ্রাহকদের order record ও active credentials রয়েছে, এবং
3. একজন নতুন developer-এর লেখা একটি **background CSV export script**।

আপনি যখন আপনার Node.js code-এ লেখেন `const total = a + b`, কিংবা একটি `Map` তৈরি করেন, কিংবা `fs.writeFileSync(...)` দিয়ে log ফাইলে একটি লাইন লেখেন—তখন পুরো ব্যাপারটা খুব simple মনে হয়। আপনার code এমনভাবে behave করে যেন ওই পুরো CPU, ওই পুরো RAM, আর ওই পুরো SSD একাই তার নিজের!

কিন্তু এবার একটু থেমে একটি বিপজ্জনক প্রশ্ন করুন:

> **ধরুন ওই background CSV export script-এ একটি মারাত্মক bug আছে (বা কোনো attacker ওটায় ঢুকে পড়েছে)। Script-টি যদি সরাসরি physical RAM-এর যেকোনো address overwrite করার চেষ্টা করে, PostgreSQL-এর memory পড়ে নেওয়ার চেষ্টা করে, কিংবা সরাসরি SSD controller-এ raw command পাঠিয়ে disk-এর সব partition মুছে দিতে চায়—তাহলে কে ওকে থামাবে?**

শুধু ভালো coding guideline বা developer-দের সততা দিয়ে এটা ঠেকানো সম্ভব নয়! যদি প্রতিটি program-কে hardware-এর ওপর সরাসরি পূর্ণ ক্ষমতা (direct control) দেওয়া থাকত, তবে একটিমাত্র buggy script মুহূর্তেই আপনার database corrupt করে দিত, অন্য service-এর secret চুরি করত, অথবা একটি infinite loop চালিয়ে পুরো server স্থায়ীভাবে freeze করে দিত।

আবার উল্টো চরমপন্থাটাও ভাবুন: নিরাপত্তা নিশ্চিত করতে গিয়ে যদি প্রতিটা সাধারণ `a + b` বা `for` loop চালানোর আগে কোনো central referee-র অনুমতি নিতে হতো, তবে আমাদের server-এর performance ভেঙে চুরমার হয়ে যেত!

Phase 2-এর শেষে আমরা Word-RAM abstraction-এর দেয়ালের কিনারায় এসে দাঁড়িয়েছিলাম—দেখেছিলাম বাস্তব hardware-এ cache, RAM, আর disk latency-র মাঝে বিশাল ব্যবধান আছে। Phase 3-এ আমরা সেই দেয়ালটা পার হয়ে সরাসরি ঢুকছি machine-এর আসল নিয়ন্ত্রকের ভেতর: **Operating System (OS)**।

এই পুরো Block 1-কে এক সুতোয় গেঁথে রাখবে একটি কেন্দ্রীয় প্রশ্ন:

> **একটি operating system আসলে কেন তৈরি করা হয়েছে, kernel কীভাবে user-space backend program-গুলোকে hardware-এর ওপর সরাসরি কর্তৃত্ব না দিয়েই CPU, memory, ও storage-এর নিরাপদ ও fast সেবা দেয়, user space থেকে kernel space-এ system call-এর ভেতর দিয়ে ঠিক কী ঘটে—এবং `strace` দিয়ে বাস্তব Linux machine-এ সেই সীমানাটা কীভাবে চোখে দেখা যায়?**

---

## ১. Operating System কেন আছে? Raw Hardware থেকে নিরাপদ Abstraction-এ

এক মুহূর্তের জন্য কল্পনা করুন, আপনার Linux server-এ কোনো operating system নেই—আছে কেবল raw hardware (একটি 8-core CPU, 16 GB RAM, একটি NVMe SSD, এবং একটি network card)। এখন আপনার checkout API চালাতে গেলে কী কী problem-এ পড়বেন?

1. **Hardware complexity বিস্ফোরণ:** প্রতিটি SSD model, network chip, আর memory controller-এর নিজস্ব hardware register ও low-level protocol আছে। OS না থাকলে আপনার প্রতিটি backend app-কেই প্রতিটি নির্দিষ্ট hardware model-এর জন্য আলাদা driver code লিখতে হতো—একটা নতুন disk লাগালেই সব app ভেঙে যেত!
2. **Resource sharing-এর বিশৃঙ্খলা:** একসাথে Node.js, PostgreSQL, আর একটা monitoring agent কীভাবে ওই 8টা CPU core ও 16 GB RAM ভাগ করে নেবে? কোনো referee না থাকলে প্রথম যে program চলবে, সে পুরো CPU ও RAM দখল করে বসে থাকবে।
3. **Isolation ও protection-এর অভাব:** এক program-এর bug আরেক program-এর memory বা disk data নষ্ট করে দেবে।

ঠিক এই তিনটি মৌলিক problem সমাধান করার জন্যই **Operating System (OS)** তৈরি হয়েছে। OSTEP (*Operating Systems: Three Easy Pieces*, Chapter 2)-এর দৃষ্টিতে একটি OS মূলত দুটি কাজ একসাথে করে:
- **Resource Manager (সম্পদ ব্যবস্থাপক):** সীমিত physical hardware—CPU cycles, physical RAM, disk ও I/O bandwidth—একাধিক running program-এর মাঝে নিরাপদে, ন্যায্যভাবে, এবং দক্ষতার সাথে বণ্টন করে।
- **Abstraction Provider (সহজ ও নিরাপদ interface নির্মাতা):** জটিল, vendor-specific raw hardware-কে আড়াল করে application-গুলোকে clean, portable, easy-to-use **abstractions** (যেমন process, virtual memory, file) দেয়—যাতে আপনার Node.js code একইভাবে laptop-এর NVMe SSD-তেও চলে, আবার cloud VM-এর virtual disk-এও চলে!

### Operating System-এর তিনটি মূল স্তম্ভ (OSTEP-এর "Three Easy Pieces")

OS-এর এই বিশাল কাজগুলোকে OSTEP তিনটি মূল conceptual pillar-এ সাজিয়েছে। শুরুতেই এদের মানচিত্রটা পরিষ্কার করে নিলে আপনি বুঝবেন কোন problem কোথায় পড়ে—এবং কোনটি আমরা Phase 3-এ শিখব আর কোনটি পরের phase-এ গভীরভাবে আসবে:

1. **Virtualization (ভার্চুয়ালাইজেশন — CPU ও Memory):**
   - *সমস্যা:* Machine-এ physical CPU core সংখ্যা সীমিত (যেমন 8টি) এবং physical RAM একটিই—অথচ server-এ একসাথে কয়েক ডজন বা শত শত program চলে।
   - *OS-এর সমাধান:* OS একটি physical resource-কে রূপান্তর করে অনেকগুলো **virtual** (আপেক্ষিক/নিজস্ব মনে হওয়া) copy-তে। প্রতিটি running program-এর কাছে মনে হয় তার নিজের একটি private CPU আছে (CPU virtualization — আমরা **Blocks 2–3**-এ শিখব) এবং তার নিজের একটি বিশাল private memory আছে (Memory virtualization — **Blocks 4–5**-এ শিখব)।
2. **Concurrency (একসাথে একাধিক কাজের সমন্বয়):**
   - *সমস্যা:* আধুনিক server একসাথে বহু request বা বহু thread চালায়; আবার OS নিজেও ভেতরে একসাথে অনেক event সামলায়। একাধিক execution flow যখন একই shared data স্পর্শ করে, তখন অনিয়ন্ত্রিত interleaving-এ data corrupt হতে পারে।
   - *Phase Boundary:* Phase 3-এ আমরা কেবল process ও thread-এর প্রাথমিক কাঠামোগত পার্থক্য (Block 2) এবং preemptive scheduling-এর ধারণা (Block 3) দেখব; race condition, locks, mutexes, semaphores, deadlocks, এবং event-loop concurrency-র গভীর তত্ত্ব ও বাস্তবায়ন আসবে **Phase 6 (Concurrency and Asynchronous Systems)**-এ।
3. **Persistence (স্থায়ী সংরক্ষণ):**
   - *সমস্যা:* RAM হলো volatile—power চলে গেলে বা server crash করলে RAM-এর সব data মুহূর্তেই মুছে যায়। অথচ আমাদের online store-এর orders, payments, আর logs টিকে থাকতে হবে।
   - *OS-এর সমাধান:* OS hardware storage device-গুলোকে **files** ও **directories**-এর clean abstraction-এর পেছনে পরিচালনা করে (আমরা **Blocks 6–7**-এ files, file descriptors, ও OS-level I/O mechanics শিখব; আর database-এর crash-safe WAL ও B-Tree storage internals আসবে **Phase 10**-এ, এবং production Linux filesystem administration আসবে **Phase 23**-এ)।

আর এই তিনটি স্তম্ভ বানানোর সময় OS designer-দের সবসময় Phase 1 Block 4-এর সেই চিরন্তন **trade-offs** সামলাতে হয় (OSTEP `2.5 Design Goals`): উচ্চ **performance** (ন্যূনতম overhead), কঠোর **protection ও isolation** (এক program যেন আরেক program বা OS-এর ক্ষতি না করতে পারে), **reliability** (OS নিজে crash করলে পুরো server-এর সব service একসাথে পড়ে যাবে!), এবং security ও energy efficiency।

---

## ২. Operating System, Kernel, এবং User-Space Utilities — পরিষ্কার পার্থক্য

দৈনন্দিন engineering আলোচনায় আমরা অনেক সময় "Linux", "Ubuntu", "the OS", আর "the kernel"—সবগুলোকে একাকার করে ফেলি। কিন্তু backend diagnosis করতে গেলে এই স্তরগুলোর সীমানা কাঁচের মতো স্বচ্ছ থাকা জরুরি।

একটি চলমান Ubuntu 24.04 Linux server-কে নিচ থেকে ওপরে চারটি পরিষ্কার স্তরে দেখুন:

```text
┌──────────────────────────────────────────────────────────────────────┐
│ 4. User Applications & Backend Services  (User Space)                │
│    e.g., node checkout-api.mjs, postgres, python3, custom scripts    │
├──────────────────────────────────────────────────────────────────────┤
│ 3. User-Space Libraries & System Utilities  (User Space)             │
│    e.g., glibc (C standard library), bash, ls, ps, strace, systemd   │
├═══════════════════ HARDWARE PRIVILEGE BOUNDARY ══════════════════════┤
│ 2. The Kernel  (Kernel Space — privileged core of the OS)            │
│    e.g., Linux kernel (scheduler, virtual memory, VFS, device        │
│    drivers, syscall handlers, interrupt/exception handlers)          │
├──────────────────────────────────────────────────────────────────────┤
│ 1. Physical Hardware                                                 │
│    CPU cores, MMU/TLB, RAM, NVMe SSD, Network Interface, Timers      │
└──────────────────────────────────────────────────────────────────────┘
```

প্রতিটি অংশের ভূমিকা আলাদা করে বুঝুন:

- **Kernel (কার্নেল):** এটিই operating system-এর একদম ভেতরের **privileged core software**, যা machine boot হওয়ার পর থেকে বন্ধ হওয়া পর্যন্ত memory-তে থাকে এবং hardware-এর ওপর সরাসরি পূর্ণ নিয়ন্ত্রণ রাখে। CPU scheduling, memory protection ও translation, file descriptor table, device drivers, এবং system-call handling—সবই kernel-এর ভেতরে চলে। টেকনিক্যাল অর্থে **"Linux"** হলো কেবল এই kernel-টির নাম!
- **User-Space System Libraries & Utilities:** শুধু একটি kernel নিয়ে একজন engineer কাজ করতে পারবেন না। Kernel-এর ওপরে থাকে অসংখ্য **user-space** library ও command-line program—যেমন **`glibc`** *(GNU C Library, যা application code-এর জন্য convenient wrapper functions সরবরাহ করে)*, **`bash`** (shell), `ls`, `ps`, `free`, `strace`, এবং service manager (`systemd`)। খেয়াল করুন: **`ls`, `bash`, `strace`, এমনকি `glibc`—এর কোনোটিই kernel নয়!** এরা সাধারণ user-space software, যাদের hardware-এর ওপর কোনো বিশেষ privilege নেই; এদেরও যেকোনো restricted কাজ করতে হলে kernel-এর কাছেই সাহায্য চাইতে হয়।
- **Operating System (বৃহত্তর অর্থে):** বাস্তবে যখন আমরা বলি *"আমার server-এ Ubuntu 24.04 LTS operating system চলছে"*, তখন আমরা বুঝাই **Linux kernel + তার চারপাশের core user-space libraries, shell, ও system utilities**-এর পুরো প্যাকেজটিকে। *(একটা দরকারি reading tip: OSTEP-এর মতো OS textbook-এ লেখকরা যখন সংক্ষেপে বলেন "the OS does X", তখন অধিকাংশ ক্ষেত্রে তাঁরা মূলত **kernel**-এর কাজের কথাই বোঝান—context দেখলেই তা স্পষ্ট হবে।)*
- **Backend Applications:** আমাদের Node.js API বা PostgreSQL server হলো একেবারেই user-space application, যা library (`glibc`, V8/libuv) এবং kernel-এর দেওয়া abstractions-এর ওপর দাঁড়িয়ে চলে।

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে আপনার Block 1 notes-এ সংক্ষেপে নিজের ভাষায় উত্তর লিখুন।)*
> 1. একজন teammate বললেন: *"আমার Node.js service-এ `ls` command চালানো বা `glibc` library call করা মানেই তো kernel code চালানো, কারণ এগুলো Ubuntu operating system-এর অংশ!"* **Operating system**, **kernel**, এবং **user-space libraries/utilities**-এর পার্থক্য দিয়ে বুঝিয়ে বলুন এই ধারণাটি কেন ভুল।
> 2. OSTEP-এর তিনটি স্তম্ভ—**Virtualization**, **Concurrency**, এবং **Persistence**—প্রতিটির পাশে আমাদের online-store backend (Node.js API + PostgreSQL)-এর একটি করে বাস্তব কাজ বসিয়ে দেখান কোন স্তম্ভ কোন problem সমাধান করে।

---

## ৩. Hardware Privilege Boundaries: User Mode বনাম Kernel Mode, এবং Limited Direct Execution

এবার সেই কঠিন performance বনাম control দ্বন্দ্বে ঢুকুন, যেটা আমরা শুরুতেই তুলেছিলাম।

ধরুন, kernel আমাদের Node.js API-কে CPU-তে চালাতে চায়। সবচেয়ে fast উপায় কী? **Direct Execution**—অর্থাৎ program-টির machine instructions সরাসরি physical CPU-তে চলতে দেওয়া! এতে মাঝখানে কোনো slow interpreter বা software simulator থাকে না, ফলে CPU-bound math ও loops native hardware speed-এ চলে।

কিন্তু OSTEP Chapter 6 (`6.1–6.2`) ঠিক এখানেই একটি মারাত্মক প্রশ্ন তোলে:

> **যদি আমরা একটি user program-কে সরাসরি CPU-তে চলতে দিই, তবে কীভাবে নিশ্চিত করব যে program-টি হঠাৎ disk-এ raw write করবে না, অন্য process-এর RAM পড়বে না, বা CPU-র control-ই ছিনিয়ে নেবে না?**

সমাধানটি কেবল software দিয়ে সম্ভব নয়—এর জন্য **CPU hardware-এর নিজস্ব সহযোগিতা** লাগে! এই hardware-assisted design-টির নামই **Limited Direct Execution (LDE)**।

### দুটি Hardware Privilege Mode: User Mode বনাম Kernel Mode

আধুনিক CPU-গুলো অন্তত দুটি আলাদা execution privilege mode *(x86-64 architecture-এ যাকে প্রায়ই privilege rings—যেমন unprivileged Ring 3 বনাম privileged Ring 0—বলা হয়; বিস্তারিত x86 assembly এখানে লাগবে না)* hardware-এর ভেতরেই enforce করে:

1. **User Mode (User Space):**
   - আমাদের Node.js server, PostgreSQL-এর user-space logic, `bash`, `ls`—সব application code চলে **user mode**-এ।
   - User mode-এ code সরাসরি CPU-তে চলে (full speed!), সাধারণ arithmetic, memory reads/writes (কেবল নিজের অনুমোদিত address space-এর ভেতর), loops, ও function calls স্বাধীনভাবে করতে পারে।
   - **কিন্তু restricted (privileged) operations সম্পূর্ণ নিষিদ্ধ!** User mode-এ থাকা কোনো program সরাসরি disk বা network controller-এ raw I/O instruction চালাতে পারে না, CPU-র memory-protection table বা trap table বদলাতে পারে না, অন্য process-এর memory পড়তে পারে না, বা machine halt করতে পারে না।
   - যদি কোনো buggy বা malicious user program user mode-এ থেকেও একটি restricted instruction চালানোর চেষ্টা করে, তবে **CPU hardware নিজেই তা আটকে দেয়** এবং একটি hardware exception (fault) তৈরি করে control kernel-এর হাতে তুলে দেয়—যেখানে kernel সাধারণত সেই নিয়ম-ভাঙা process-টিকে terminate করে দেয়!
2. **Kernel Mode (Kernel Space):**
   - কেবল operating system-এর **kernel** চলে **kernel mode**-এ।
   - Kernel mode-এ CPU সমস্ত privileged hardware instruction চালানোর অনুমতি দেয়—device driver দিয়ে SSD/network-এ কথা বলা, memory-mapping tables update করা, process schedule করা, ইত্যাদি।

এখনই চোখে পড়ছে কেন নামটা **Limited Direct Execution**:
- **Direct Execution** → সাধারণ code সরাসরি CPU-তে native speed-এ চলে (performance রক্ষা হয়)।
- **Limited** → hardware privilege boundary user code-কে restricted কাজ করা থেকে আটকে রাখে (control ও protection রক্ষা হয়)।

### অত্যন্ত জরুরি সূক্ষ্মতা: User Mode থেকে Kernel Mode-এ ঢোকার সব পথ System Call নয়!

এখানে একটি অত্যন্ত সাধারণ ভুল ধারণা গোড়াতেই ভেঙে দেওয়া দরকার। অনেকে ভেবে বসেন: *"Kernel mode-এ ঢোকা বা যেকোনো privileged কাজ হওয়া মানেই নিশ্চয়ই user code একটি system call করেছে!"*

**এটি সঠিক নয়।** System call হলো user space থেকে kernel-এর সেবা চাওয়ার **নিয়ন্ত্রিত voluntary পথ**—কিন্তু এটি kernel mode-এ প্রবেশের বা privileged operation ঘটার **একমাত্র** পথ নয়! CPU অন্তত **তিনটি ভিন্ন কারণে** user mode থেকে kernel mode-এ প্রবেশ করে (পাশাপাশি kernel নিজেও ভেতরে privileged housekeeping কাজ চালাতে পারে):

| Kernel-এ প্রবেশের পথ | কে/কী trigger করে? | Synchronous নাকি Asynchronous? | বাস্তব উদাহরণ ও Phase 3-এ অবস্থান |
| :--- | :--- | :--- | :--- |
| **1. System Call (Software Trap)** | User-space program **ইচ্ছাকৃতভাবে** kernel-এর কাছে একটি privileged সেবা চায় | **Synchronous** (নির্দিষ্ট instruction চালানোর ঠিক মুহূর্তেই ঘটে) | ফাইল খোলা (`openat`), লেখা (`write`), memory চাওয়া (`mmap`/`brk`) — **Block 1 (আজকের মূল ফোকাস!)** |
| **2. Hardware Exception / Fault** | User-space-এর কোনো instruction চালাতে গিয়ে CPU একটি **অস্বাভাবিক বা বিশেষ শর্ত** পায় | **Synchronous** (ঠিক ওই offending/triggering instruction-এর কারণে ঘটে) | অবৈধ memory address স্পর্শ (`SIGSEGV`), শূন্য দিয়ে ভাগ, user mode-এ privileged instruction চালানো, অথবা **page fault** — **Blocks 4–5** |
| **3. Hardware Interrupt** | CPU-র বাইরের কোনো **hardware device** স্বাধীনভাবে CPU-কে সংকেত পাঠায় | **Asynchronous** (user code তখন যে instruction-ই চালাক না কেন, যেকোনো মুহূর্তে আসতে পারে) | **Timer interrupt** (CPU preemption-এর জন্য — **Block 3**), অথবা disk/network device-এর I/O completion signal (**Block 7**) |

এই পার্থক্যটা মনে গেঁথে রাখুন: **System call হলো সেই controlled door, যেটা দিয়ে একটি unprivileged user-space program নিজে kernel-কে ডেকে নিরাপদভাবে কোনো OS service চায়।** অন্যদিকে hardware exceptions ও hardware interrupts হলো ভিন্ন দুটি hardware-driven mechanism, যেগুলো user code না চাইলেও CPU-কে kernel mode-এ নিয়ে যায়!

---

## ৪. একটি System Call ভেতরে কীভাবে কাজ করে? Wrapper থেকে Trap Table ও Kernel Validation পর্যন্ত

এবার একটা বাস্তব backend মুহূর্তের ভেতর zoom in করুন। ধরুন, আপনার Node.js checkout API-তে একটি order সফল হওয়ার পর আপনি log ফাইলে বা `stdout`-এ একটি JSON লাইন লিখলেন:

```javascript
// Node.js user-space code
fs.writeSync(fd, '{"event":"order_paid","orderId":42}\n');
```

বাইরে থেকে এটি দেখতে একদম সাধারণ একটা JavaScript function call। কিন্তু এই একটি লাইনের নিচে একাধিক user-space স্তর পার হয়ে শেষ পর্যন্ত kernel-এর ভেতরে কীভাবে ঢোকা হয়?

প্রথমে নিচের tiny, সম্পূর্ণ C উদাহরণটি দেখুন *(ভয় পাবেন না—এটা কোনো C programming course নয়; প্রতিটা unfamiliar C অংশ পাশেই বাংলায় ব্যাখ্যা করা আছে, শুধু mechanism-টা স্পষ্ট করার জন্য)*:

```c
// Minimal conceptual illustration — compiling is NOT required in Block 1
#include <unistd.h>  // POSIX wrapper declarations (যেমন write)
#include <errno.h>   // kernel error code ধারণকারী thread-local variable: errno

// ssize_t = signed integer type (কত byte লেখা হলো, অথবা error হলে -1)
ssize_t written = write(1, "order_paid\n", 11);
//                      ^  ^               ^
//                      |  |               └─ কয়টি byte লিখতে চাই (11 bytes)
//                      |  └─ user-space memory-তে data-র ঠিকানা (buffer pointer)
//                      └─ 1 = standard output (stdout) file descriptor
if (written == -1) {
    // kernel ব্যর্থ হলে glibc wrapper return করে -1 এবং আসল কারণ রাখে errno-তে
}
```

ভালো করে তাকান! `write(...)` দেখতে হুবহু একটি সাধারণ C function call-এর মতো—ঠিক যেমন OSTEP Chapter 6 (`6.2 Aside: Why System Calls Look Like Procedure Calls`) এবং Linux `syscalls(2)` বলে! কিন্তু এটি **সরাসরি kernel-এর ভেতরে jump করছে না**। আসলে এখানে দুটো আলাদা জিনিস ঘটছে:
1. আপনার code (বা Node.js-এর C++/libuv স্তর) প্রথমে call করে **`glibc`-র `write()` wrapper function**-কে—যা নিজেও এখনো **user space**-এই আছে!
2. সেই `glibc` wrapper function-টির ভেতরে লুকানো থাকে বিশেষ CPU instruction (আধুনিক x86-64 Linux-এ `syscall` instruction; OSTEP-এ সাধারণভাবে যাকে **trap instruction** বলা হয়েছে)।

### System Call-এর ৬-ধাপের পূর্ণ প্রোটোকল (OSTEP `6.2` + Linux `syscalls(2)`)

একটি `write()` system call শুরু থেকে শেষ পর্যন্ত যে ছয়টি সুনির্দিষ্ট ধাপে চলে, তার পুরো flow-টা দেখুন:

```text
  USER MODE (Unprivileged)                         KERNEL MODE (Privileged)
  ========================                         ========================
                                                   [0. Boot Time Setup]
                                                   Kernel installs Trap Table
                                                   addresses into CPU hardware
                                                   (user mode CANNOT alter it!)

  1. App calls fs.writeSync()
     -> Node C++ -> glibc write() wrapper
  2. Wrapper puts syscall number (e.g. SYS_write)
     & args (fd, buf_addr, len) into CPU registers
  3. Wrapper executes hardware trap / syscall  ───▶ [Hardware privilege switch!]
                                                    - Mode: User -> Kernel
                                                    - Saves user registers & PC
                                                      onto process kernel stack
                                                    - Jumps ONLY to trap table
                                                      handler address

                                                   4. Kernel syscall handler runs:
                                                      - Looks up syscall number
                                                      - VALIDATES all user args!
                                                        (Is fd valid? Is buf_addr
                                                         inside user space, NOT
                                                         kernel memory?)
                                                      - Executes privileged I/O
                                                      - Sets return value (bytes
                                                        written or -ERRNO)

  5. Wrapper receives raw return value        ◀─── Return-from-trap instruction:
     - If negative (-ERRNO): flips sign,            - Restores user registers & PC
       sets positive errno, returns -1              - Mode: Kernel -> User
     - Node.js converts -1/errno into JS Error
  6. Backend code resumes after writeSync()
```

এই ছয়টি ধাপের তিনটি design invariant একজন backend engineer-এর জন্য অমূল্য:

1. **কেন User Code নিজে Kernel Function-এর Address দিতে পারে না?**
   ধরুন user code যদি বলতে পারত *"kernel memory-র `0xffff8880...` address-এ jump করো"*, তবে একটা malicious program kernel-এর permission-check অংশটা এড়িয়ে সরাসরি মাঝখানের কোনো privileged code-এ লাফ দিয়ে ঢুকে পড়ত! তাই **boot-এর সময়** kernel নিজে CPU-র **Trap Table**-এ ঠিক করে দেয় কোন trap বা syscall number এলে kernel-এর *কোন নির্দিষ্ট entry point*-এ jump হবে। User program কেবল একটি সংখ্যা (**system-call number**) দেয়—address নয়!
2. **Kernel User-Supplied Arguments-কে কখনোই অন্ধভাবে বিশ্বাস করে না (OSTEP `6.2` Security Tip):**
   `write(fd, buf_addr, len)` call-এ user program নিজে একটি memory address (`buf_addr`) পাঠায়। কোনো attacker যদি ইচ্ছা করে `buf_addr`-এ **kernel memory-র** একটি address বসিয়ে দেয়, আর kernel তা যাচাই না করে ওই memory-র bytes file-এ লিখে দেয়, তবে attacker পুরো kernel ও অন্য সব process-এর গোপন memory পড়ে ফেলবে! তাই প্রতিটি syscall boundary-তে kernel বাধ্যতামূলকভাবে প্রতিটি argument (buffer pointer, length, file descriptor, permissions) যাচাই করে—ভুল বা অবৈধ হলে নিরাপদভাবে error (যেমন `-EFAULT` বা `-EBADF`) ফেরত দেয়।
3. **Wrapper Function বনাম Raw Syscall (`syscalls(2)`):**
   `glibc` wrapper কেবল registers সাজানো ও `syscall` চালানোই করে না—অনেক সময় এটি architecture/kernel-version অনুযায়ী সামঞ্জস্য করে এবং Linux-এর raw negative return code (যেমন `-ENOENT`) পেলে তার চিহ্ন উল্টে positive `errno` set করে ও caller-কে `-1` return করে! *(পাশাপাশি কিছু library function—যেমন `strlen()` বা `JSON.stringify()`—পুরোপুরি user space-এই কাজ শেষ করে, কোনো syscall-ই করে না; আবার `printf()` বা Node.js buffered stream বহু call user-space buffer-এ জমিয়ে তারপর একবারে একটি `write()` syscall করে!)*

### এক নজরে পার্থক্য: Ordinary Function Call বনাম Library Wrapper বনাম System Call

| বৈশিষ্ট্য | Ordinary User Function Call (যেমন `add(a, b)`) | Library Function (`libc` / Node.js) | System Call (যেমন `openat`, `read`, `write`) |
| :--- | :--- | :--- | :--- |
| **কোথায় চলে?** | সম্পূর্ণ User Space | User Space-এ শুরু হয়; প্রয়োজনে ভেতরে syscall করে (আবার `strlen`-এর মতো কিছু call কোনো syscall-ই করে না) | Hardware trap পেরিয়ে **Kernel Space**-এ execute হয় |
| **Privilege পরিবর্তন হয়?** | না (User → User) | শুধু ভেতরে syscall করলে হয় | **হ্যাঁ** (User → Kernel → User) |
| **Jump target কে ঠিক করে?** | Program-এর নিজের compiled/JIT code | User-space library code | কেবল boot-time **Kernel Trap Table** + syscall number |
| **Input validation?** | Application logic-এর ইচ্ছাধীন | Library-level sanity checks | **বাধ্যতামূলক strict kernel boundary validation** |
| **Relative cost** | অত্যন্ত হালকা (nanosecond scale; অনেক সময় compiler/JIT inline করে ফেলে) | ভেতরে syscall না থাকলে হালকা; syscall করলে syscall overhead যোগ হয় | তুলনামূলকভাবে উল্লেখযোগ্যভাবে ভারী (privilege switch, state save/restore, validation, CPU mitigation overhead) |

### Backend Performance-এ Syscall Overhead কেন বাস্তবেই গুরুত্বপূর্ণ?

Phase 2 Block 5-এ আপনি শিখেছিলেন: দুটি `Θ(N)` implementation-এর আসল wall-clock speed শুধু Big-O-তে ঠিক হয় না—প্রতিটি iteration-এর **constant factor** ও **system overhead** বিশাল পার্থক্য গড়ে দেয়।

এখন আপনার হাতে তার প্রথম OS-স্তরের কারণটি চলে এসেছে! একটি backend logging বা file-export module-এর দুটি design তুলনা করুন—দুটোই `1,000`টা JSON log line লিখবে (`Θ(N)` bytes):
- **Design A (Unbuffered, per-line বা per-character `write`):** প্রতিটি ছোট টুকরার জন্য আলাদাভাবে user space থেকে kernel space-এ jump করে `write()` syscall চালায় → **`1,000` (বা `50,000`) বার privilege transition + trap handling + kernel validation!**
- **Design B (User-space buffering + batched `write`):** User-space memory-তে ছোট একটি buffer-এ লাইনগুলো জমিয়ে (পুরোপুরি user mode-এ, কোনো privilege switch ছাড়া!) একবারে বড় একটি chunk দিয়ে **`1`টি মাত্র `write()` syscall** চালায়!

দুটোই ঠিক সমান bytes লিখছে, কিন্তু user↔kernel boundary পার হওয়ার সংখ্যা কমে গেছে **হাজার গুণ**! *(অবশ্যই এখানেও Phase 1-এর trade-off আছে: user-space buffer-এ data জমে থাকা অবস্থায় যদি process হঠাৎ crash করে, তবে unflushed log হারিয়ে যেতে পারে—তাই performance বনাম crash-visibility/durability-র ভারসাম্য রাখতে হয়; kernel Page Cache ও `fsync()` durability আমরা Block 7-এ গভীরভাবে দেখব।)*

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে আপনার Block 1 notes-এ সংক্ষেপে উত্তর লিখুন।)*
> 1. C-তে `write(1, "hi\n", 3)` দেখতে সাধারণ একটি function call-এর মতো, অথচ এটি একটি system call execute করে। **Library wrapper (`glibc`)**, **syscall number**, **trap instruction**, এবং **kernel trap table**—এই চারটি উপাদান ব্যবহার করে ধাপে ধাপে ব্যাখ্যা করুন কেন user code সরাসরি kernel-এর ভেতরে নিজের পছন্দমতো address-এ jump করতে পারে না।
> 2. *"CPU যখনই user mode থেকে kernel mode-এ ঢোকে, তার মানেই user-space program একটি system call করেছে"*—এই বক্তব্যটি কি সঠিক? **System call**, **hardware exception**, এবং **hardware interrupt**-এর মধ্যে পার্থক্য দেখিয়ে ২–৩ বাক্যে ব্যাখ্যা করুন।

---

## ৫. Linux-এ User/Kernel সীমানা চোখে দেখা: `strace` কীভাবে কাজ করে (এবং কী প্রমাণ করে না)

এতক্ষণ আমরা তত্ত্ব ও mechanism বুঝলাম। কিন্তু Phase 1 Block 5-এর **Evidence-Based Engineering** আমাদের শিখিয়েছে: শুধু বই পড়ে বিশ্বাস নয়—বাস্তব machine-এ empirical evidence চোখে দেখতে হবে!

Linux-এ কোনো running program ঠিক কোন কোন **system call** করছে, কী argument পাঠাচ্ছে, আর kernel কী return করছে—তা দেখার সবচেয়ে পরিষ্কার, standard tool হলো **`strace`** *(system call tracer, `strace(1)`)*।

দুটি সহজ flag দিয়েই Block 1-এ আপনি পুরো user↔kernel সীমানাটা জীবন্ত দেখতে পাবেন:
- **`strace -e trace=write <command>`:** `<command>` চালানোর সময় কেবল `write` system call-গুলো (অথবা `openat,read,write,close`-এর মতো কমা-দিয়ে একাধিক syscall) লাইন বাই লাইন দেখায়—exact arguments এবং return value সহ!
- **`strace -c <command>`:** `<command>` শুরু থেকে শেষ পর্যন্ত মোট কোন syscall **কতবার** ডাকা হয়েছে, কতগুলোতে error হয়েছে, এবং kernel-side summary সময় কত—তার একটি পরিষ্কার summary table দেখায়।

### Ubuntu 24.04-এ নিরাপদ (Unprivileged) `strace` নিয়ম

একটি গুরুত্বপূর্ণ Linux security বাস্তবতা আগে থেকেই জেনে রাখুন। Ubuntu 24.04 LTS-এ default kernel setting হলো **`kernel.yama.ptrace_scope = 1`**। এর অর্থ:
- আপনি যদি `sudo` ছাড়া অন্য কোনো already-running process-এর PID-তে `strace -p <PID>` দিয়ে attach করতে চান, তবে security-র কারণে Linux তা `EPERM` দিয়ে আটকে দেবে।
- **কিন্তু আপনি যদি `strace <command>` দিয়ে লক্ষ্য program-টিকে সরাসরি `strace`-এর child হিসেবে চালু করেন**, তবে তা **কোনো `sudo` ছাড়াই, কোনো kernel parameter না বদলেই ১০০% নিরাপদে কাজ করে!** তাই Phase 3 জুড়ে আমরা সবসময় child-process tracing (`strace -c <cmd>`, `strace -e ... <cmd>`) ব্যবহার করব।

### অত্যন্ত জরুরি Scientific Guardrail: `strace` কী প্রমাণ করে—আর কী প্রমাণ করে *না*?

Phase 2 Block 7-এ আমরা দেখেছি, যেকোনো measurement tool ব্যবহারের আগে তার সীমাবদ্ধতা জানা বাধ্যতামূলক। `strace` সম্পর্কে দুটি কথা কখনোই ভুলবেন না:

1. **`strace` যা প্রমাণ করে:**
   - একটি program চালানোর সময় বাস্তবে **কোন কোন syscall** এবং **ঠিক কতবার** invoke হয়েছে (যেমন `/bin/echo`-র কয়েক ডজন syscall বনাম `node` startup-এর কয়েক শত syscall, কিংবা `1,000` unbuffered `write` বনাম `1` batched `write`)।
   - Kernel-এ ঠিক কী arguments গেছে এবং kernel কোন exact return value বা `errno` (যেমন `ENOENT`, `EACCES`) ফেরত দিয়েছে।
2. **`strace` যা প্রমাণ করে *না*:**
   - **Untraced production latency বা throughput!** কারণ `strace` Linux-এর `ptrace` mechanism ব্যবহার করে **প্রতিটি syscall-এর ঢোকার সময় একবার এবং বের হওয়ার সময় আরেকবার** target process-টিকে থামিয়ে দেয়। এই observer overhead নিজেই execution-কে ব্যাপকভাবে ধীর করে দেয়!
   - তাই `strace -c`-এর `seconds` বা `usecs/call` কলাম কখনোই raw production benchmark হিসেবে উদ্ধৃত করবেন না—এবং কখনোই না ভেবে কোনো busy production server-এ `strace` attach করবেন না! প্রকৃত wall-clock সময় তুলনা করতে হলে `strace` ছাড়াই Phase 2 Block 7-এর মতো controlled monotonic timer (`performance.now()`) চালাতে হয়।

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে আপনার Block 1 notes-এ সংক্ষেপে উত্তর লিখুন।)*
> 1. একজন engineer `strace -c node app.mjs` চালিয়ে দেখলেন পুরো script শেষ হতে `120 ms` লেগেছে, তাই তিনি রিপোর্টে লিখলেন *"আমাদের Node.js script-এর production startup time হলো `120 ms`"*। `strace`-এর কাজের ধরন দিয়ে ব্যাখ্যা করুন কেন এই সিদ্ধান্তটি ত্রুটিপূর্ণ, এবং `strace -c` আসলে কী নির্ভরযোগ্যভাবে প্রমাণ করতে পারে।
> 2. Ubuntu 24.04-এ `sudo` ব্যবহার না করে এবং কোনো kernel setting না বদলে নিরাপদভাবে একটি command-এর system calls দেখার সঠিক উপায় কী, এবং কেন `strace -p <PID>` default অবস্থায় অন্য process-এ ব্যর্থ হয়?

---

## ৬. Block 2-এর দিকে সেতুবন্ধন: Privilege Boundary থেকে আসল "Process"-এ

একটু থেমে দেখুন Block 1-এ আপনি কোন শক্ত ভিত্তিটা তৈরি করলেন:
- আপনি এখন জানেন একটি **operating system** কেন প্রয়োজন (resource manager + abstraction provider; OSTEP-এর Virtualization, Concurrency, Persistence স্তম্ভ) এবং **kernel** বনাম **user-space libraries/utilities** বনাম **applications**-এর স্পষ্ট বিভাজন।
- আপনি **user mode বনাম kernel mode** hardware privilege boundary, **Limited Direct Execution (LDE)**, এবং **system call**-এর ৬-ধাপের প্রোটোকল (library wrapper → trap/syscall → trap table → kernel argument validation → return-from-trap → `errno`) বোঝেন—এবং system call-কে hardware exceptions ও hardware interrupts থেকে স্পষ্টভাবে আলাদা করতে পারেন।
- আপনি জানেন syscall boundary পার হওয়ার overhead backend design (যেমন buffering বনাম per-item write)-কে কীভাবে প্রভাবিত করে, এবং `strace` দিয়ে নিরাপদে সেই boundary পর্যবেক্ষণ ও তার সীমা বিচার করতে পারেন।

কিন্তু এবার আপনার `strace -c node -e "console.log('hello')"` চালানোর পরই মাথায় একটি দারুণ প্রশ্ন জাগবে:

> **আমরা তো দেখলাম `node` চালালেই kernel-এর সাথে শত শত system call হয় এবং kernel প্রতিটি running program-কে আলাদা সীমানায় রাখে। কিন্তু যখন আমরা terminal-এ `node server.mjs` লিখি বা PostgreSQL একটি নতুন worker চালু করে, তখন kernel-এর ভেতরে একটি "process" আসলে কী জিনিস? Linux কীভাবে একটি process তৈরি (`fork`, `execve`), পরিচালনা (states), ও পরিষ্কার (`waitpid`) করে—আর একটি process ও একটি thread-এর কাঠামোগত পার্থক্য কী?**

ঠিক এই প্রশ্নটির উত্তর খুঁজতেই পরের ধাপে *(Phase 3 যখন আনুষ্ঠানিকভাবে unlock হবে)* আমরা এগোব **Phase 3, Block 2 — Processes: Creation, Execution, and Lifecycle**-এ!

---

## Study Plan, Learning Objectives, Terminology & Authoritative Resources

> **Continuity & Ownership Notes:** আপনার সব Pause & Checks, Practical Experiment 1-এর নিজস্ব observation ও বিশ্লেষণ, Transfer Exercise 1, এবং (যদি করতে চান) Optional Closed-Book Assessment-এর উত্তর স্বাধীনভাবে আপনার notes-এ (`phases/phase-03/notes/block-01-notes.md` — Phase 3 unlock হওয়ার পর, অথবা তার আগে আপনার ব্যক্তিগত study notes-এ) লিখুন। AI-generated material কখনোই completed learning বা learner study hours হিসেবে গণ্য হয় না।
>
> **Progression & Verification Policy:** এই block-এর Closed-Book Assessment সম্পূর্ণ **optional** এবং কোনো mandatory progression gate নয়। পড়া, experiment, ও mandatory exercise শেষ করে assessment skip করলে এই block-এর status হবে **`Studied — mastery not formally verified`**; যথেষ্ট evidence share করে ChatGPT evaluation সম্পন্ন হলেই তা **`Mastery verified`** হিসেবে গণ্য হবে।

### Study Time Reference

*প্রতিটি activity-র বিস্তারিত flexible time breakdown দেখতে এই নথির শুরুর **Study Time Guide** দেখুন। Approved Phase 3 Research Dossier-এ Block 1-এর target allocation হলো `3 hours (180 min)`—যা একটি flexible planning reference, কোনো strict deadline নয়।*

### Block 1 Learning Objectives *(Primary Target Level: `Working Understanding` | Supported Phase 3 Outcome: Outcome 1)*

- **LO 1.1 (`Working Understanding` | Bloom: *Understand*):** Explain the core purpose of an operating system as a **resource manager and abstraction provider** across OSTEP's three pillars—**Virtualization** (CPU & memory), **Concurrency**, and **Persistence**—linking each pillar to concrete backend behavior and explicitly identifying which pillars are deepened in later phases.
- **LO 1.2 (`Working Understanding` | Bloom: *Analyze*):** Contrast **user mode (user space)** and **kernel mode (kernel space)** as hardware-enforced privilege boundaries, explaining why unprivileged applications cannot execute restricted hardware instructions or access arbitrary physical memory directly.
- **LO 1.3 (`Working Understanding` | Bloom: *Analyze*):** Trace **Limited Direct Execution (LDE)** and a **system call** end-to-end (library wrapper → trap instruction → privilege elevation → kernel trap table lookup & handler validation → return-from-trap back to user mode), contrasting syscall overhead against an ordinary user-space function call.
- **LO 1.4 (`Practical Competence` | Bloom: *Apply*):** Use `strace` safely on a child process (`strace -c` and `strace -e trace=...` without `sudo`) to observe real user-to-kernel system-call transitions made by a Linux command and a minimal Node.js script.

### Terminology Inventory

| Concept / Term | Official Target Level | Why It Matters in Block 1 |
| :--- | :---: | :--- |
| **Operating System Abstractions (Virtualization, Concurrency, Persistence)** | `Working Understanding` | Unifying mental map separating CPU/memory sharing, concurrent execution (Phase 6), and durable I/O. |
| **User Space vs. Kernel Space (Privilege Modes)** | `Working Understanding` | Hardware privilege boundary ensures one buggy or malicious backend process cannot crash the host or read another process's secrets. |
| **Limited Direct Execution (LDE)** | `Working Understanding` | Explains how the OS achieves near-native CPU speed while retaining hardware-enforced control over restricted operations. |
| **System Call (Syscall) & Trap Table** | `Working Understanding` | Controlled user-to-kernel entry mechanism (`trap`/`syscall` instruction + boot-time trap table), distinct from ordinary function calls, hardware exceptions, and hardware interrupts. |
| **Library Wrapper (`libc` / Node.js) vs. Raw Syscall** | `Working Understanding` | Explains why high-level code (`fs.writeSync`, `console.log`, `write()`) invokes user-space wrappers that trigger raw Linux syscalls (`openat`, `read`, `write`) and translate negative kernel errors to `errno`. |
| **System-Call Tracing (`strace` on child commands)** | `Practical Competence` | Primary empirical lens for observing exact kernel interactions of any backend command safely without `sudo`. |

### Mandatory Authoritative Resources *(Targeted Reading — ~55–70 min total)*

1. **Mandatory 1 (~25–30 min — assigned sections only):** Remzi H. Arpaci-Dusseau & Andrea C. Arpaci-Dusseau, *Operating Systems: Three Easy Pieces (OSTEP), Version 1.10* — **Chapter 2: Introduction to Operating Systems** ([`intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/intro.pdf)) (`LO 1.1`):
   - **Assigned scope:** কেবল Sections **`2.1`** (*Virtualizing The CPU*), **`2.2`** (*Virtualizing Memory*), **`2.3`** (*Concurrency*), **`2.4`** (*Persistence*), **`2.5`** (*Design Goals*), এবং **`2.7`** (*Summary*) (~11 pages)। **Section `2.6` (*Some History*) ও Homework explicitly skip করুন।**
   - **What to extract:** Resource manager + abstraction provider হিসেবে OS-এর মূল ভূমিকা, এবং Virtualization, Concurrency, ও Persistence—এই তিন স্তম্ভের big-picture মানচিত্র।
2. **Mandatory 2 (~20–25 min — assigned sections only):** OSTEP, Version 1.10 — **Chapter 6: Mechanism: Limited Direct Execution** ([`cpu-mechanisms.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-mechanisms.pdf)) (`LO 1.2`, `LO 1.3`):
   - **Assigned scope:** কেবল Sections **`6.1`** (*Basic Technique: Limited Direct Execution*), **`6.2`** (*Problem #1: Restricted Operations* — Aside ও Tip-সহ), এবং **`6.5`** (*Summary*-এর কেবল LDE ও restricted-operations অংশ) (~6.5 pages)। **Section `6.3` (*Switching Between Processes*) Block 3-এর জন্য defer করুন এবং Section `6.4` ও Homework skip করুন।**
   - **What to extract:** User mode বনাম kernel mode, boot-time trap table, syscall protocol, এবং কেন kernel user-supplied pointers/arguments-কে অন্ধভাবে বিশ্বাস করে না।
3. **Mandatory 3 (~10–15 min — targeted skim only):** Linux `man-pages` — [`syscalls(2)`](https://man7.org/linux/man-pages/man2/syscalls.2.html) এবং [`strace(1)`](https://man7.org/linux/man-pages/man1/strace.1.html) (`LO 1.3`, `LO 1.4`):
   - **Assigned scope:**
     - `syscalls(2)`: কেবল *Description* অংশের প্রথম ৪টি paragraph (*"The system call is the fundamental interface..."* এবং *System calls and library wrapper functions*, `errno` রূপান্তরসহ)। নিচের বিশাল *System call list* টেবিলটি পড়বেন না!
     - `strace(1)`: কেবল *Description* এবং `-c` ও `-e trace=...` options।
   - **What to extract:** বাস্তব Linux-এ raw syscall বনাম `glibc` wrapper-এর সম্পর্ক (`-1` + `errno`) এবং unprivileged child command-এ `strace` ব্যবহারের সঠিক syntax।

### Optional Supplementary Resource *(Outside Mandatory Budget — Do Not Treat as Mandatory)*

- **Supplementary (Free):** OSTEP, Version 1.10 — Chapter 2, Section `2.6` (*Some History*, pp. 12–17 of [`intro.pdf`](https://pages.cs.wisc.edu/~remzi/OSTEP/intro.pdf)) — batch systems থেকে multiprogramming ও Unix-এর ঐতিহাসিক বিবর্তন জানতে আগ্রহী হলে ঐচ্ছিকভাবে পড়তে পারেন।

---

## Practical Experiment 1 & Transfer Exercise 1 *(Mandatory — ~45–55 min total)*

> **Ownership & Safety Rules:** নিচের experiment-টি আপনার নিজের Ubuntu 24.04 machine-এ সম্পূর্ণ **unprivileged** অবস্থায় চালান (কোনো `sudo` নয়, কোনো `/proc/sys/*` পরিবর্তন নয়, সব temp ফাইল কেবল `/tmp`-এর নিজস্ব temporary folder-এ তৈরি হয়ে সাথে সাথেই মুছে যাবে)। আপনার নিজের machine-এ পাওয়া আসল output ও বিশ্লেষণ notes-এ লিখুন—কখনোই output বানিয়ে লিখবেন না। এখানে কোনো answer key দেওয়া হয়নি।

### Practical Experiment 1: Observing the User/Kernel Boundary Live with `strace` *(Mandatory — ~20–25 min | `LO 1.3`, `LO 1.4`)*

#### Step A — ছোট Linux Utility বনাম Node.js Runtime Startup তুলনা

আপনার Ubuntu terminal-এ নিচের তিনটি read-only command চালান:

```bash
# 1. কেবল write syscall লাইন বাই লাইন দেখুন
strace -e trace=write /bin/echo "hello from user space"

# 2. একটি ছোট C utility (/bin/echo)-র মোট syscall summary দেখুন
strace -c /bin/echo "hello"

# 3. একটি minimal Node.js one-liner-এর মোট syscall summary দেখুন
strace -c node -e "console.log('hello')"
```

**আপনার notes-এ রেকর্ড ও বিশ্লেষণ করুন:**
1. প্রথম command-এ `write(...)`-এর ঠিক কোন তিনটি argument এবং return value দেখতে পেলেন?
2. `/bin/echo` এবং `node -e "console.log('hello')"`—দুটোই stdout-এ মাত্র একটি শব্দ (`hello`) প্রিন্ট করেছে, অথচ দুটোর **total syscall count** এবং top syscall-গুলোর (যেমন `mmap`, `openat`, `read`, `brk`, `close`) মধ্যে কী ধরনের পার্থক্য দেখলেন? কেন একটি high-level runtime boot করতে এতগুলো kernel transition লাগে?

#### Step B — Unbuffered Per-Line Writes বনাম Batched Write (`1,000` Log Lines)

এবার একটি isolated `/tmp` directory তৈরি করে দুটি minimal Node.js command তুলনা করুন (একই `1,000`টি JSON log লাইন লিখবে; শেষে `/tmp` directory-টি নিরাপদে মুছে দেওয়া হবে):

```bash
TMPDIR=$(mktemp -d /tmp/phase3-b1.XXXXXX)

# Approach 1: প্রতিটি log line-এর জন্য আলাদা fs.writeSync (unbuffered)
strace -c -e trace=write node -e '
const fs = require("node:fs");
const fd = fs.openSync(process.argv[1], "w");
for (let i = 0; i < 1000; i++) {
  fs.writeSync(fd, `{"event":"order_check","i":${i}}\n`);
}
fs.closeSync(fd);
' "$TMPDIR/unbuffered.log"

# Approach 2: User space-এ array/string-এ জমিয়ে একবারে batched fs.writeSync
strace -c -e trace=write node -e '
const fs = require("node:fs");
const lines = [];
for (let i = 0; i < 1000; i++) {
  lines.push(`{"event":"order_check","i":${i}}\n`);
}
const fd = fs.openSync(process.argv[1], "w");
fs.writeSync(fd, lines.join(""));
fs.closeSync(fd);
' "$TMPDIR/batched.log"

# যাচাই করুন দুটি ফাইলের byte size একদম সমান, তারপর নিরাপদে পরিষ্কার করুন
wc -c "$TMPDIR/unbuffered.log" "$TMPDIR/batched.log"
rm -rf "$TMPDIR"
```

#### Expected Observation Patterns — এবং এটি কী প্রমাণ করে বনাম কী প্রমাণ করে *না*

- **Expected patterns (exact numbers machine/Node-version ভেদে সামান্য ভিন্ন হতে পারে):**
  - Step A-তে `strace -e trace=write /bin/echo ...` দেখাবে একটি `write(1, ...)` call এবং তার return value হিসেবে লেখা byte সংখ্যা। `/bin/echo`-র মোট syscall সংখ্যা থাকবে কয়েক ডজনের ঘরে, আর `node -e "console.log('hello')"`-র মোট syscall সংখ্যা হবে কয়েক শত (কারণ runtime binary, shared libraries, memory mappings, ও internal event/I/O setup করতে অনেকগুলো syscall লাগে)।
  - Step B-তে `wc -c` প্রমাণ করবে দুটি ফাইলে exact সমান bytes লেখা হয়েছে, অথচ Approach 1-এ `write` syscall-এর সংখ্যা হবে **`1,000+`** (আপনার loop-এর 1,000টি + Node.js runtime-এর নিজস্ব কয়েকটি internal `write`), আর Approach 2-এ `write` syscall নেমে আসবে **এক অঙ্কের ঘরে** (মূল ফাইল লেখার `1`টি batched `write` + Node.js runtime-এর অভ্যন্তরীণ কয়েকটি ছোট `write`)!
- **এই পরীক্ষাটি যা প্রমাণ করে:** User-space code থেকে প্রতিবার unbuffered `fs.writeSync` ডাকলে প্রতিবারই user→kernel boundary পেরিয়ে একটি বাস্তব Linux `write` syscall ঘটে; আর user-space batching দিয়ে exact একই output অনেক কম syscall-এ লেখা যায়।
- **এই পরীক্ষাটি যা প্রমাণ করে *না*:**
  - `strace -c`-এ দেখানো `seconds` বা `usecs/call` কোনো **untraced production latency** নয় (কারণ `strace` নিজেই প্রতি syscall-এ `ptrace` stop/resume overhead যোগ করে)।
  - এখান থেকে কোনো universal fixed speedup multiplier বা disk durability guarantee দাবি করা যাবে না (কারণ `write` এখানে কেবল kernel memory buffer-এ লিখে ফিরছে—physical disk flush নয়; সেটি আসবে Block 7-এ)।

---

### Transfer Exercise 1: Privilege Boundary & Syscall Cost Reasoning *(Mandatory — ~25–30 min | `LO 1.1`, `LO 1.2`, `LO 1.3`)*

নিচের **তিনটি বাস্তব backend engineering scenario** স্বাধীনভাবে বিশ্লেষণ করে আপনার notes-এ লিখুন:

1. **Scenario 1 — Direct Hardware Bypass প্রস্তাব:**
   একজন junior engineer বললেন: *"আমাদের order-export service-টা disk থেকে data পড়তে সময় নিচ্ছে। Operating system মাঝখানে থেকে সময় নষ্ট করছে—আমরা কি সরাসরি JavaScript বা একটা native C addon লিখে SSD-র raw disk sector পড়ে নিতে পারি না যাতে OS-কে পুরোপুরি bypass করা যায়?"*
   - **আপনার কাজ:** **User mode বনাম kernel mode**, **Limited Direct Execution (LDE)**, এবং **OS abstraction / isolation**-এর mechanism দিয়ে ব্যাখ্যা করুন: (ক) unprivileged user-space code সরাসরি raw SSD hardware instruction চালানোর চেষ্টা করলে CPU hardware ও kernel কী করবে, এবং (খ) যদি কোনোভাবে সব application-কে raw SSD ও physical RAM-এর পূর্ণ ক্ষমতা দিয়ে দেওয়া হতো, তবে একই server-এ চলা PostgreSQL ও অন্য backend service-গুলোর ওপর কী বিপর্যয় নামত?
2. **Scenario 2 — Per-Character Logging Middleware:**
   আমাদের একটি HTTP service-এ custom audit logger লেখা হয়েছে, যেখানে প্রতিটি incoming request-এর `250`-character (ASCII, `250 bytes`) JSON log string-কে একটি `for` loop দিয়ে **একবারে `1` byte করে** unbuffered `write()` দিয়ে log ফাইলে লেখা হচ্ছে।
   - **আপনার কাজ:** (ক) `1,000`টি request এলে এই design বনাম প্রতি request-এ `1`টি buffered/batched `write()` (বা প্রতি `20` request-এ flush করা buffer)-এর মধ্যে `write` syscall count-এর পার্থক্য হিসাব করুন; (খ) প্রতিটি extra syscall-এ ঠিক কী কী overhead যুক্ত হয় তা ৬-ধাপের syscall প্রোটোকল দিয়ে ব্যাখ্যা করুন; এবং (গ) user-space buffering-এর বিপরীতে **কোন একটি বাস্তব trade-off / failure risk** (যেমন হঠাৎ process crash) মাথায় রাখতে হবে তা উল্লেখ করুন।
3. **Scenario 3 — AI-Generated Technical Explanation Audit:**
   একজন teammate একটি AI coding assistant-কে Linux system calls সম্পর্কে জিজ্ঞাসা করলে AI নিচের উত্তরটি দিল:
   > *"In Linux, calling `write()` in C or `fs.writeSync()` in Node.js is just a normal library function call into `libc` that jumps directly to the kernel's `sys_write` memory address. Also, because the kernel protects hardware, every single time the CPU transitions from user mode to kernel mode or runs a privileged operation, it is always because a user program invoked a system call."*
   - **আপনার কাজ:** এই ছোট অনুচ্ছেদটির ভেতর লুকানো **তিনটি আলাদা technical ভুল** চিহ্নিত করুন, প্রতিটি ভুল কেন mechanism-গতভাবে অসত্য (বা নিরাপত্তা-ভঙ্গকারী) তা ব্যাখ্যা করুন (**`glibc` wrapper বনাম trap instruction**, **boot-time trap table ও syscall number বনাম direct kernel address jump**, এবং **system calls বনাম hardware exceptions ও hardware interrupts / internal kernel work**), এবং অনুচ্ছেদটিকে ২–৪ বাক্যে নিখুঁতভাবে rewrite করুন।

---

## Closed-Book Synthesis Assessment *(Optional — ~20–25 min)*

এই assessment-টি সম্পূর্ণ **optional**—এটি কোনো mandatory progression gate নয়। আপনি চাইলে:
- এই lesson, notes, ও সব browser tab বন্ধ রেখে নিচের ৫টি প্রশ্নের উত্তর নিজের ভাষায় লিখে formal evaluation-এর জন্য **ChatGPT**-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে);
- অথবা guided lesson, mandatory readings, experiment, ও exercise শেষ করে যথেষ্ট বুঝেছেন মনে করলে assessment skip করতে পারেন (`Studied — mastery not formally verified`);
- অথবা assessment ছাড়াই যেকোনো unclear concept নিয়ে ChatGPT-র সাথে আলোচনা করতে পারেন।

*(Note: `~20–25 min` কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি approximate estimate—কোনো strict time limit নয়।)*

1. **(LO 1.1 / Outcome 1)** Operating system-এর দুটি মূল ভূমিকা (**resource manager** এবং **abstraction provider**) ব্যাখ্যা করুন। OSTEP-এর তিনটি স্তম্ভ—**Virtualization**, **Concurrency**, এবং **Persistence**—কোন কোন মৌলিক hardware problem সমাধান করে, এবং একটি real Node.js + PostgreSQL server-এ প্রতিটির ভূমিকা কী?
2. **(LO 1.1, LO 1.2 / Outcome 1)** **Operating System**, **Kernel**, **User-Space System Libraries/Utilities** (`glibc`, `bash`, `strace`), এবং **Backend Applications**-এর মধ্যে স্পষ্ট সীমারেখা টানুন। কেন একটি user-space utility (`ls` বা `strace`) kernel-এর অংশ নয়?
3. **(LO 1.2, LO 1.3 / Outcome 1)** **Limited Direct Execution (LDE)** কীভাবে একইসাথে high performance (direct CPU execution) এবং system control/isolation (hardware privilege boundary) রক্ষা করে? একটি `write()` system call শুরু থেকে শেষ পর্যন্ত যে ৬টি ধাপে চলে—`glibc` wrapper, syscall number, trap instruction, boot-time trap table, kernel argument validation, এবং return-from-trap + `errno`—তা ধাপে ধাপে ব্যাখ্যা করুন।
4. **(LO 1.2, LO 1.3 / Outcome 1)** কেন *"every privileged operation or kernel-mode entry is mediated by a system call"* বলা ভুল? **System calls**, **hardware exceptions**, এবং **hardware interrupts**-এর পার্থক্য উদাহরণসহ ব্যাখ্যা করুন।
5. **(LO 1.3, LO 1.4 / Outcome 1)** একটি ordinary user-space function call-এর তুলনায় system call কেন বেশি overhead বহন করে, এবং এটি backend buffering design-এ কী প্রভাব ফেলে? Ubuntu-তে `strace` দিয়ে কীভাবে নিরাপদভাবে (unprivileged) এই behavior পর্যবেক্ষণ করা যায়, এবং `strace` output ঠিক কী প্রমাণ করে আর কী প্রমাণ করে *না*?

### Block 1 Readiness & Progression Checklist

- [ ] Guided lesson মনোযোগ দিয়ে পড়েছেন এবং Pause & Checks 1–3 নিজের notes-এ লিখে (পরে Self-Check Guidance মিলিয়ে) সম্পন্ন করেছেন।
- [ ] তিনটি Mandatory Resource (**OSTEP Ch 2** `2.1–2.5, 2.7`, **OSTEP Ch 6** `6.1–6.2, 6.5`, এবং Linux **`syscalls(2)`** + **`strace(1)`** নির্ধারিত অংশ) সম্পন্ন করেছেন।
- [ ] নিজের Ubuntu machine-এ নিরাপদে **Practical Experiment 1 (`strace`)** চালিয়ে বাস্তব observations ও সীমাবদ্ধতা নথিভুক্ত করেছেন।
- [ ] স্বাধীনভাবে **Transfer Exercise 1 (Privilege Boundary & Syscall Cost Reasoning)**-এর তিনটি scenario বিশ্লেষণ করেছেন।
- [ ] *(Optional)* Notes/tabs বন্ধ রেখে Closed-Book Synthesis Assessment Questions 1–5 লিখেছেন এবং চাইলে ChatGPT-র কাছে evaluation-এর জন্য share করেছেন।

> **Progression Status Distinction:** এই Block 1 lesson-টি Curriculum Owner-এর অনুমোদিত one-time pre-study exception হিসেবে তৈরি; Phase 3 এখনো globally unlocked নয় এবং `PROGRESS.md`/`NEXT_SESSION.md`-এর live continuity অপরিবর্তিত রয়েছে। পড়া, experiment, ও mandatory exercise শেষ করে optional assessment ছাড়া থাকলে status হয় **`Studied — mastery not formally verified`**; যথেষ্ট evidence share করে ChatGPT evaluation সম্পন্ন হলেই কেবল তা **`Mastery verified`** হিসেবে গণ্য হয়।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *নিজের notes-এ উত্তর লেখার **পরেই কেবল** নিচের guidance দেখুন। Practical Experiment 1, Transfer Exercise 1, বা Optional Closed-Book Assessment-এর কোনো answer key এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন অনুশীলন ও (চাইলে) ChatGPT review-এর জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* Ubuntu operating system-এর বৃহত্তর প্যাকেজের অংশ হওয়া মানেই kernel-এর অংশ হওয়া নয়। **Kernel** হলো privileged core software যা kernel mode-এ চলে এবং hardware নিয়ন্ত্রণ করে। বিপরীতে **`ls`** হলো একটি সাধারণ user-space utility program এবং **`glibc`** হলো একটি user-space library—দুটোই unprivileged user mode-এ চলে; hardware-সংক্রান্ত কোনো কাজ করতে হলে এদেরও kernel-এর কাছেই system call করতে হয়।
  2. *শক্তিশালী উত্তরে থাকবে:*
     - **Virtualization:** একই Linux server-এ Node.js API ও PostgreSQL একসাথে চললেও প্রতিটি process নিজের private CPU ও private virtual memory পাওয়ার অনুভূতি পায় এবং একে অপরের RAM corrupt করতে পারে না।
     - **Concurrency:** একাধিক overlapping checkout request বা internal thread একই shared state/resource স্পর্শ করলে race condition ছাড়া নিরাপদ সমন্বয় প্রয়োজন হয় *(গভীর coverage Phase 6-এ)*।
     - **Persistence:** Server restart বা power loss-এর পরও order records, PostgreSQL data files, ও logs যেন না হারায়, সেজন্য OS storage hardware-কে files ও directories-এর মাধ্যমে পরিচালনা করে।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* `write()` call করলে আপনার code প্রথমে user space-এ থাকা **`glibc` library wrapper**-এ যায়। সেই wrapper একটি **syscall number** ও arguments CPU register-এ বসিয়ে hardware **trap / `syscall` instruction** চালায়। CPU তখন privilege বাড়িয়ে kernel mode-এ ঢোকে এবং কেবল boot-এর সময় kernel-এর বসিয়ে রাখা **trap table** দেখেই নির্দিষ্ট kernel handler-এ jump করে। যদি user code নিজেই kernel memory-র ইচ্ছামতো address-এ jump করতে পারত, তবে সে kernel-এর permission ও security check বাইপাস করে পুরো system দখল করে নিতে পারত।
  2. *শক্তিশালী উত্তরে থাকবে:* বক্তব্যটি **ভুল**। **System call** হলো user program-এর ইচ্ছাকৃত (synchronous) সেবা-অনুরোধ। কিন্তু এর বাইরেও অন্তত দুটি hardware-driven পথে CPU kernel mode-এ ঢোকে: (১) **Hardware exceptions / faults** (synchronous—যেমন অবৈধ memory access বা page fault বা user mode-এ privileged instruction চালানোর চেষ্টা), এবং (২) **Hardware interrupts** (asynchronous—যেমন timer tick বা disk/network device-এর completion signal)। পাশাপাশি kernel নিজেও ভেতরে privileged কাজ চালাতে পারে।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* `strace` প্রতিটি syscall-এর entry ও exit-এ `ptrace` দিয়ে target process-টিকে থামিয়ে inspect করে—ফলে এটি ব্যাপক observer overhead যোগ করে এবং wall-clock time কৃত্রিমভাবে বাড়িয়ে দেয়। তাই `strace`-এর অধীনে পাওয়া `120 ms` কখনোই untraced production startup latency প্রমাণ করে না। `strace -c` নির্ভরযোগ্যভাবে প্রমাণ করে **কোন কোন syscall** এবং **মোট কতবার** ডাকা হয়েছে (এবং কতগুলো error হয়েছে)।
  2. *শক্তিশালী উত্তরে থাকবে:* Ubuntu 24.04-এ default `kernel.yama.ptrace_scope = 1` নিরাপত্তা নীতি অন্য already-running প্রক্রিয়ায় unprivileged `strace -p <PID>` attach করা আটকে দেয় (`EPERM`)। তাই `sudo` বা kernel parameter পরিবর্তন ছাড়া নিরাপদ উপায় হলো লক্ষ্য command-টিকে সরাসরি `strace`-এর child হিসেবে চালু করা—যেমন `strace -c <command>` বা `strace -e trace=... <command>`।
