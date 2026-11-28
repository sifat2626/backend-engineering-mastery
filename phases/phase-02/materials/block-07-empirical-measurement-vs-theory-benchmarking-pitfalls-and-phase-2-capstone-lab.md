# Phase 2, Block 7 — Empirical Measurement vs. Theory, Benchmarking Pitfalls, and Phase 2 Capstone Lab

## Study Time Guide

| Activity | Assigned Scope | Estimated Time |
| :--- | :--- | :---: |
| **Guided Lesson + Pause & Checks 1–3** *(main priority)* | পুরো guided lesson মনোযোগ দিয়ে পড়া এবং `phases/phase-02/notes/block-07-notes.md`-এ Pause & Checks 1–3 লেখা | **~50–65 min** *(flexible — limit নয়)* |
| **Mandatory 1:** Brendan Gregg — *Active Benchmarking* | কেবল *Summary*, *Passive Benchmarking*, *Active Benchmarking*, *Problem Checklist*, এবং *Statistical Analysis* overview *(Linux `perf` / DTrace commands বাদ দিন)* | ~15–20 min |
| **Mandatory 2:** Vyacheslav Egorov (`mraleph`) — ২টি নিবন্ধ | *microbenchmarks fairy tale* (LICM, constant propagation, DCE, ও *Moral*) + *Microbenchmarks are experiments*-এর opening thesis (*"Benchmarks are not numerology"*) *(assembly dumps বাদ দিন)* | ~20–25 min |
| **Mandatory 3:** Official Node.js 26 LTS Documentation | `node:perf_hooks` (`performance.now()`) এবং `node:process` (`process.hrtime.bigint()`) targeted skim | ~10 min |
| **Pre-Lab Exercise (Mandatory):** Spot the Broken Microbenchmark | ~30-line ত্রুটিপূর্ণ Node.js benchmark snippet-এর ৪টি flaw শনাক্ত, ব্যাখ্যা ও clean rewrite | ~15–20 min |
| **Phase 2 Capstone Scaling Laboratory (Mandatory — 4 Stages)** | Pure Node.js 26 LTS (`storeBatchEngine.mjs`): Stage 1 (`~25–30 min`), Stage 2 (`~25–30 min`), Stage 3 (`~30–35 min`), Stage 4 (`~20–25 min`) | **~100–120 min** |
| **Total (Required Core — without Optional Assessment)** | Guided Lesson + ৩টি Mandatory Reading + Pre-Lab Exercise + 4-Stage Capstone Lab | **~210–260 min (~3.5–4.3 h)** |
| **Optional Closed-Book Synthesis Assessment** *(optional)* | Notes/tabs বন্ধ রেখে ৫টি Phase-2 integrative synthesis প্রশ্নের উত্তর লেখা | ~25–30 min *(optional)* |
| **Total (with Optional Assessment)** | Required Core + Optional Closed-Book Synthesis Assessment | **~235–290 min (~3.9–4.8 h)** |

> **Planning Note:** Approved Phase 2 Research Dossier-এ Block 7-এর roadmap target হলো **4 hours (240 minutes)**—যা একটি flexible planning reference, কোনো strict deadline নয়। তাড়াহুড়ো করে শেষ করার চেয়ে প্রতিটি experimental principle গভীরভাবে বোঝা এবং চার-ধাপের **Phase 2 Capstone Scaling Lab** সৎভাবে সম্পন্ন করাই main priority; বিশেষ করে Guided Lesson ও Capstone-এ প্রয়োজনে আনুমানিক সময়ের চেয়ে বেশি সময় লাগা একদম স্বাভাবিক। প্রতিটি reading estimate কেবল নির্ধারিত sections-এর জন্য। **Optional Closed-Book Synthesis Assessment** সম্পূর্ণ optional এবং কোনো progression gate নয়।

---

Block 6-এর শেষে আমাদের online store-এর analytics ও checkout helper module (`storeBatchEngine.mjs`)-কে আমরা কাগজে-কলমে audit করে ফেলেছি। আপনি যুক্তি দিয়ে দেখিয়েছেন:
- `.find()` সরিয়ে `Map` index বসালে `O(N * M)` থেকে `O(N + M)` expected-এ নামবে,
- `.includes()` সরিয়ে `Set` নিলে `O(N^2)` থেকে `O(N)` expected-এ নামবে,
- আর পুরো dataset sort করার বদলে bounded Min-Heap নিলে `O(N log N)` থেকে `O(N log K)` এবং `O(K)` space-এ নামবে।

এখন Phase 1 Block 5-এর সেই বৈজ্ঞানিক শৃঙ্খলাটা ফিরে আসে: **কাগজের দাবি প্রমাণিত হয় না যতক্ষণ না তা বাস্তব execution-এর সামনে যাচাই করা হচ্ছে!**

ধরুন আমাদের টিমের দুজন engineer সেই refactor-গুলোর গতি মাপতে Node.js-এ দ্রুত দুটি benchmark লিখে ফেললেন:
- **Engineer A** আমাদের `N = 50` সাইজের unit-test fixture-এর আগে ও পরে `Date.now()` বসিয়ে এক বার চালিয়ে দেখলেন দুটোরই সময় `0 ms`—আর ঘোষণা দিলেন: *"দুই implementation-ই সমান দ্রুত, refactor করে কোনো লাভ হয়নি!"*
- **Engineer B** একটি tight loop-এর ভেতর `1,000,000` বার একটি pure function ডাকলেন যার return value কোথাও ব্যবহার করা হয়নি, আর পাশাপাশি অন্য branch-এ progress দেখতে `console.log()` বসালেন—তারপর এক অবিশ্বাস্য `500×` speedup-এর screenshot Slack-এ পোস্ট করে দিলেন!

একটু থেমে ভেবে দেখুন: **দুজনেই বাস্তব code চালিয়ে "সংখ্যা" বের করেছেন—তবু দুজনের সিদ্ধান্তই মারাত্মকভাবে ভুল!**

একটি benchmark কেবল কোনো code-এর আগে-পরে timer বসিয়ে দেওয়া নয়; **একটি benchmark হলো একটি নিয়ন্ত্রিত বৈজ্ঞানিক পরীক্ষা (controlled experiment)**। আর ঠিক এখান থেকেই জন্ম নেয় Phase 2-এর শেষ ও সমন্বয়কারী block-টির কেন্দ্রীয় প্রশ্ন:

> **আমরা কীভাবে নিশ্চিত হব যে আমাদের তাত্ত্বিক complexity পূর্বাভাস বাস্তব execution-এর সাথে সত্যিই মিলছে—এবং বিভ্রান্তিকর microbenchmark-এর ফাঁদে পড়ে নিজেরাই নিজেদের ধোঁকা দিচ্ছি না?**

Block 7 কোনো বিচ্ছিন্ন নতুন অধ্যায় নয়—এটি Blocks 1–6-এর সব তাত্ত্বিক ও hardware ধারণাকে Phase 1-এর evidence-based engineering ও ownership mindset-এর সাথে একত্রিত করে। চলুন প্রথমেই স্পষ্ট করি **তত্ত্ব (theory)** আর **বাস্তব পরিমাপ (empirical measurement)**-এর সম্পর্ক ও সীমারেখা।

---

## ১. Theory বনাম Empirical Measurement: দুটি পরস্পরের পরিপূরক, কেউ কারও বিকল্প নয়

Backend engineering-এ দুটো চরমপন্থী ভুল প্রায়ই দেখা যায়:
- **ভুল ১ ("শুধু Big-O যথেষ্ট"):** *"আমার algorithm `O(log N)` আর ওটা `O(N)`—তাই কোনো measurement ছাড়াই আমারটা সবসময় দ্রুত!"* (Block 5-এ আপনি দেখেছেন constant factors, crossover points, spatial locality, runtime overhead, ও external I/O কীভাবে এই দাবি ভেঙে দেয়।)
- **ভুল ২ ("Big-O কেবল তত্ত্ব, শুধু মেপে দেখলেই হলো"):** *"তত্ত্ব বাদ দাও, আমার laptop-এ `N = 500` চালিয়ে দেখো কোনটা কম ms নেয়!"*

দুটোই বিপজ্জনক, কারণ **Theory** এবং **Empirical Measurement** দুটি একদম ভিন্ন প্রশ্নের উত্তর দেয়:

| দিক | Theoretical Complexity Analysis (Blocks 1–4) | Empirical Measurement / Benchmarking (Block 7) |
| :--- | :--- | :--- |
| **কী বলে দেয়** | Input `N, M, K` অনির্দিষ্টভাবে বাড়লে abstract operation count এবং auxiliary memory **কোন growth trend** অনুসরণ করবে, এবং worst-case / adversarial input-এ কোথায় ভেঙে পড়বে। | একটি **নির্দিষ্ট** implementation, **নির্দিষ্ট** runtime version, **নির্দিষ্ট** input shape, এবং **নির্দিষ্ট** machine/environment-এ বাস্তব execution সময়, constant factors, runtime overhead, ও practical crossover কেমন হচ্ছে। |
| **কী কখনোই বলতে পারে না** | Exact wall-clock milliseconds, real constant factors, cache/GC effects, বা ঠিক কোন `N`-এ crossover ঘটবে। | **Finite সংখ্যক input size-এর পরিমাপ কখনোই একটি asymptotic bound (`N → ∞`) গাণিতিকভাবে প্রমাণ (prove) করতে পারে না!** |
| **বৈজ্ঞানিক ভূমিকা** | **Falsifiable hypothesis তৈরি করে** (যেমন: *"input দ্বিগুণ হলে সময় প্রায় `4×` হওয়া উচিত"*)। | সেই hypothesis-টিকে **corroborate (সমর্থন) বা falsify (খণ্ডন)** করে এবং তত্ত্বের বাইরে থাকা বাস্তব overhead উন্মোচন করে। |

### তত্ত্ব সঠিক হয়েও বাস্তব পরিমাপ কেন ভিন্ন হতে পারে?

একটি সহজ উদাহরণ দেখুন। ধরুন আমাদের online store-এ একটি configuration lookup-এর জন্য:
- **Implementation A** করে `O(N)` contiguous array scan।
- **Implementation B** করে `O(log N)` pointer-ভিত্তিক balanced search tree lookup।

তাত্ত্বিক বিশ্লেষণ ১০০% সঠিক যে বড় `N`-এ `O(log N)` এগিয়ে থাকবে। কিন্তু আপনি যদি `N = 20`-এ মাপেন, তবে বাস্তব measurement-এ **Implementation A (`O(N)`) প্রায় নিশ্চিতভাবেই দ্রুততর হবে**—কারণ ছোট `N`-এ tree-র constant factors ও pointer chasing (Block 5!) asymptotic সুবিধাকে ছাড়িয়ে যায়! তত্ত্ব ভুল ছিল না, আবার `N = 20`-এর measurement-ও প্রমাণ করে না যে `N = 1,000,000`-এ array জিতবে!

অতএব পরিষ্কার নীতিটি মাথায় গেঁথে রাখুন:
> **Theory ছাড়া measurement হলো অন্ধ অনুমান (আপনি জানেনই না `N = 100,000`-এ কী ঘটবে); আর measurement ছাড়া theory হলো অসম্পূর্ণ পূর্বাভাস (আপনি জানেন না বাস্তব constants, locality, বা runtime overhead কী করছে)।**

কিন্তু পরিমাপ করতে নামলেই আপনার সামনে ওত পেতে থাকে **পাঁচটি পরিচিত microbenchmarking ফাঁদ**, যেগুলো না চিনলে আপনি সুন্দর একটা সংখ্যা পাবেন—যার সাথে বাস্তবতার কোনো সম্পর্কই নেই!

---

## ২. পাঁচটি Microbenchmarking Pitfall যা Engineer-দের নীরবে বিভ্রান্ত করে

আধুনিক JavaScript runtime (যেমন Node.js 26 Active LTS-এর engine) কোনো সরল লাইন-বাই-লাইন interpreter নয়—এটি execution চলাকালে code-কে বুদ্ধিমত্তার সাথে optimize করে এবং স্বয়ংক্রিয়ভাবে memory manage করে। ঠিক সেই ক্ষমতাগুলোই অসতর্ক microbenchmark-কে বিকৃত করে দিতে পারে *(Mandatory 2: Vyacheslav Egorov / `mraleph`)*।

চলুন পাঁচটি ফাঁদ বাস্তব JavaScript উদাহরণসহ ধাপে ধাপে চিনি।

### Pitfall 1: Dead-Code Elimination (DCE) এবং Constant Folding / Hoisting

ধরুন একজন developer আমাদের discount formula-র গতি মাপতে নিচের মতো একটি benchmark লিখলেন:

```javascript
// FLAWED: Pure computation whose return value is never observed (conditional DCE risk!)
function computeLineDiscount(priceCents, rateBps) {
  return Math.floor((priceCents * rateBps) / 10000);
}

const iterations = 1_000_000;
const t0 = performance.now();
for (let i = 0; i < iterations; i++) {
  computeLineDiscount(1999, 1500); // return value discarded, inputs constant!
}
const elapsedMs = performance.now() - t0;
```

এখানে কী ঝুঁকি তৈরি হয়েছে?
- **অত্যন্ত জরুরি Precision Guardrail:** কখনোই এমন অন্ধ দাবি করবেন না যে *"JavaScript engine সব unused function call স্বয়ংক্রিয়ভাবে মুছে দেয়!"* বাস্তবে **Dead-Code Elimination (DCE)**, **constant folding/propagation**, বা **loop-invariant code motion (LICM)** ঘটে **কেবল নির্দিষ্ট শর্ত পূরণ হলে**—যখন optimizing engine প্রমাণ করতে পারে যে computation-টি সম্পূর্ণ **side-effect-free** (বাইরের কোনো state বদলায় না) এবং এর ফলাফল program-এর কোনো **observable output**-এ প্রভাব ফেলে না *(Mandatory 2: `mraleph` 2012 — LICM, constant propagation, DCE)*!
- কিন্তু যখন ওই শর্তগুলো মিলে যায় (যেমন ওপরের মতো constant input ও unused return value), তখন engine loop-এর ভেতরের কাজটাকে আগে থেকেই হিসাব করে ফেলতে বা পুরোপুরি বাদ দিয়ে দিতে পারে—ফলে আপনি ভাবছেন `1,000,000` বার discount হিসাব মাপছেন, অথচ বাস্তবে মাপছেন একটা **খালি লুপ**!

**সঠিক প্রতিকার — Lightweight Observable Result Sink:**
Computation-টির ফলাফলকে একটি অতি হালকা accumulator-এ যুক্ত করে timed section-এর **বাইরে** observable (যেমন print বা return) করুন—যাতে:
1. Input-গুলো loop-এ বদলায় (অথবা pre-generated array থেকে আসে) এবং ফলাফল observable থাকে (ফলে DCE/constant-folding ঝুঁকি দূর হয়), **এবং**
2. **Sink-টি নিজে যেন অত্যন্ত হালকা হয় এবং measured operation-কে dominate না করে!**

```javascript
// BETTER: Lightweight observable checksum sink (printed AFTER timing ends!)
let checksum = 0;
const t0 = performance.now();
for (let i = 0; i < orders.length; i++) {
  checksum += computeLineDiscount(orders[i].priceCents, orders[i].rateBps);
}
const elapsedMs = performance.now() - t0;
// Observe checksum outside the timed region so the work stays observable without I/O inside the loop
if (checksum === -1) console.log(checksum);
```

*(খেয়াল করুন: কেউ যদি sink হিসেবে loop-এর ভেতরেই `console.log(result)` বা `JSON.stringify(result)` বসায়, তবে I/O বা serialization-ই মূল operation-এর চেয়ে শত-হাজার গুণ বেশি সময় নিয়ে পুরো benchmark নষ্ট করে দেবে!)*

---

### Pitfall 2: JIT Warmup (Cold Execution বনাম Warmed Steady State)

JavaScript runtime প্রথমবার কোনো function চালানোর সময় সেটি তুলনামূলক ধীর (**cold execution**) হতে পারে, কারণ runtime তখন code-টিকে parse, শুরুতে interpret, এবং পর্যাপ্তবার চলার পর optimize করে। কয়েকবার চলার পর code-টি স্থিতিশীল দ্রুত গতিতে পৌঁছায়—যাকে বলে **warmed steady-state execution**।

- আপনি যদি Implementation A-কে একদম প্রথমে (cold অবস্থায়) মাপেন, আর Implementation B-কে পরের ধাপে (যখন shared helper বা runtime ইতিমধ্যেই গরম হয়ে গেছে) মাপেন—তবে আপনি দুটো algorithm তুলনা করছেন না, বরং **cold startup + compilation overhead-এর সাথে warmed steady-state-এর তুলনা করছেন!**
- **সঠিক প্রতিকার:** প্রকৃত timed trials শুরু করার আগে প্রতিটি candidate implementation-এর ওপর কয়েকটি **untimed warmup run** চালিয়ে নিন (এবং যদি আপনার বাস্তব production প্রশ্নটিই হয় *"cold startup কেমন?"*, তবে সেটাকে আলাদা experiment হিসেবে স্পষ্ট উল্লেখ করুন)।
- *(Scope boundary: V8-এর internal compilation tiers, bytecode, inline caches, বা deoptimization mechanics আমরা **Phase 5 (JavaScript Language Internals)**-এ শিখব; এখানে conceptual `Working Understanding`-ই লক্ষ্য।)*

---

### Pitfall 3: Garbage Collection (GC) এবং Allocation Noise

ধরুন একটি operation প্রচুর temporary array বা object তৈরি করে (যেমন Block 6-এর `[...acc, item]` বা নতুন enriched object array)। আপনি যদি `7` বার trial চালান, তবে দেখবেন `6`টি trial নিল `~8 ms`, কিন্তু ৪ নম্বর trial হঠাৎ নিল `~24 ms`!

কেন? কারণ ঠিক ৪ নম্বর trial-এর মাঝখানে runtime-এর **garbage collector** আগের trial-গুলোতে তৈরি হওয়া unreachable object পরিষ্কার করতে সক্রিয় হয়ে থাকতে পারে!
- **দুটি বড় সতর্কতা:**
  1. একটি single benchmark run দেখে কখনোই GC behavior নিখুঁতভাবে predict করা যায় না *(V8 GC generation mechanics Phase 5-এ)*।
  2. **Algorithmic growth** (যেমন `N` দ্বিগুণ হলে median নিয়মিত `~4×` হওয়া) এবং **GC/OS jitter** (কোনো এক-দুটি trial-এ হঠাৎ spike ও বড় min–max spread)-কে কখনো এক করে ফেলবেন না!

---

### Pitfall 4: Wall-Clock `Date.now()` বনাম Monotonic High-Resolution Timers

অনেক developer অভ্যাসবশত `const start = Date.now();` দিয়ে benchmark লেখেন। এটি **দুটি কারণে** microbenchmarking-এর জন্য অনুপযুক্ত:
1. **অতি স্থূল resolution (`1 ms` integer):** `0.3 ms` বা `0.8 ms`-এর কাজ `Date.now()`-এ `0 ms` অথবা `1 ms` দেখায়—যা সম্পূর্ণ অর্থহীন!
2. **Non-monotonic wall clock:** `Date.now()` আপনার OS-এর সাধারণ দেয়াল-ঘড়ি (wall clock) পড়ে। Benchmark চলাকালে যদি OS-এর NTP time-sync ঘড়িটি কয়েক milliseconds পিছিয়ে বা এগিয়ে দেয়, তবে `Date.now() - start` ভুল (এমনকি ঋণাত্মক!) সময় দেখাতে পারে!

**Node.js 26 Active LTS-এর দুটি সঠিক, built-in, monotonic high-resolution timer** *(Mandatory 3: Official Node.js 26 Docs)*:

1. **`performance.now()` (`node:perf_hooks`):**
   - `import { performance } from "node:perf_hooks";` (সবসময় `performance.now()` হিসেবে receiver-সহ ডাকবেন!)।
   - Return type: **`<number>`** — বর্তমান Node.js process শুরু হওয়া (`0`) থেকে monotonic high-resolution **milliseconds** (sub-millisecond দশমিক অংশসহ)।
2. **`process.hrtime.bigint()` (`node:process`):**
   - `import { hrtime } from "node:process";` অথবা সরাসরি `process.hrtime.bigint()`।
   - Return type: **`<bigint>`** — monotonic high-resolution সময় **nanoseconds (`ns`)**-এ!
   - দুটি `bigint` সরাসরি বিয়োগ (`endNs - startNs`) করে exact elapsed nanoseconds পাওয়া যায়, এবং একক রূপান্তর হলো:
     - **`1 µs` (microsecond) = `1,000 ns`**
     - **`1 ms` (millisecond) = `1,000 µs` = `1,000,000 ns` (`1e6 ns`)**

```javascript
import { performance } from "node:perf_hooks";
import { hrtime } from "node:process";

// Option 1: performance.now() -> milliseconds (<number>)
const t0 = performance.now();
// ... measured operation ...
const elapsedMsFromPerf = performance.now() - t0;

// Option 2: process.hrtime.bigint() -> nanoseconds (<bigint>) -> convert to ms
const startNs = hrtime.bigint();
// ... measured operation ...
const elapsedNs = hrtime.bigint() - startNs;
const elapsedMsFromHrtime = Number(elapsedNs) / 1_000_000;
```

---

### Pitfall 5: Passive Benchmarking বনাম Brendan Gregg-এর Active Benchmarking

এতগুলো প্রযুক্তিগত খুঁটিনাটি ঠিক করার পরও সবচেয়ে বড় মানবিক ফাঁদটা থেকে যায়—যাকে systems performance engineer **Brendan Gregg** বলেছেন **Passive Benchmarking** *(Mandatory 1)*:
- মাত্র **একটি single run** চালানো,
- কেবল একটি **mean (গড়)** রিপোর্ট করা (অথচ একটিমাত্র `50 ms` GC spike পাঁচটি `5 ms` run-এর গড়কে `12.5 ms` বানিয়ে বিভ্রান্ত করে!),
- কেবল **একটিমাত্র input size** (যেমন `N = 50`) পরীক্ষা করা,
- **অবাস্তব input distribution** ব্যবহার করা (যেমন sorting মাপতে গিয়ে আগে থেকে sorted array দেওয়া, বা সব element identical রাখা!),
- এবং benchmark চলাকালে **আসলে কী কাজ চলছে তা কখনোই যাচাই না করা**!

Gregg এই বিভ্রমকে একটি অবিস্মরণীয় বাক্যে ধরেছেন:
> **"You benchmark A, actually measure B, and conclude you've measured C."**
*(অর্থাৎ আপনি ভাবছেন algorithm `A` মাপছেন, বাস্তবে হয়তো মাপছেন input-allocation বা console logging `B`, আর মিটিংয়ে গিয়ে সিদ্ধান্ত নিচ্ছেন production latency `C`!)*

আর `mraleph` তাঁর 2024 নিবন্ধ *"Microbenchmarks are experiments"*-এ ঠিক একই সতর্কবাণী দিয়েছেন: **"Benchmarks are not numerology"**—রহস্যময় সংখ্যায় চোখ বুজে বিশ্বাস না করে প্রতিটি benchmark-কে একটি যাচাইযোগ্য experiment হিসেবে পরিচালনা করতে হবে!

**Active Benchmarking-এর প্রথম ব্যবহারিক অভ্যাস (কোনো ভারী statistics ছাড়াই):**
- প্রতি input size-এ একাধিক trial চালান (যেমন `7` বা `9`টি trial)।
- একক mean-এর বদলে রিপোর্ট করুন:
  - **Median (মধ্যমা):** সব trial-এর সময় ছোট থেকে বড় সাজিয়ে মাঝখানের মান—এটি isolated GC/OS outlier spike-এ সহজে নড়ে না।
  - **Min–Max Spread (সর্বনিম্ন ও সর্বোচ্চ মান):** এটি আপনাকে সৎভাবে বলে দেয় measurement-টি কতটা স্থিতিশীল ছিল (`min = 4.1 ms, median = 4.2 ms, max = 4.4 ms` মানে স্থিতিশীল; আর `min = 4.1 ms, median = 4.5 ms, max = 28.0 ms` মানে কোনো বড় jitter বা GC pause ঘটেছে—তা লুকাবেন না!)।

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-07-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন teammate `N = 1,000` থেকে `N = 16,000` পর্যন্ত একটি function মেপে বললেন: *"দেখো, প্রতিটি ধাপে সময় প্রায় দ্বিগুণ হচ্ছে—তাহলে আমরা গাণিতিকভাবে prove করে ফেললাম যে function-টি `Θ(N)`!"* Theory এবং empirical measurement-এর সম্পর্ক ও সীমা ব্যবহার করে ব্যাখ্যা করুন কেন এই পরিমাপ hypothesis-কে **সমর্থন (corroborate)** করলেও কখনোই asymptotic bound **prove** করে না।
> 2. নিচের দুটি ভুল দাবি সংশোধন করুন:
>    - *(ক)* *"JavaScript-এ return value না নিলেই engine স্বয়ংক্রিয়ভাবে সব function call মুছে ফেলে (DCE)!"* — ঠিক কোন শর্তে DCE ঘটতে পারে, এবং কীভাবে lightweight observable sink দিয়ে তা এড়াবেন?
>    - *(খ)* *"পাঁচটি trial-এর সময় এসেছে `[4.1, 4.3, 4.2, 22.7, 4.2] ms`—এগুলোর mean (`7.9 ms`) রিপোর্ট করাই যথেষ্ট!"* — এখানে **median** ও **min–max spread** কত, এবং কেবল mean দেখানো কেন বিভ্রান্তিকর?

এখন আমরা জানি কীভাবে বিভ্রান্তিকর timer ও single-run ফাঁদ এড়াতে হয়। পরের প্রশ্নটি হলো: **কোনো একটি clean timer দিয়ে একটিমাত্র `N`-এ সংখ্যা মাপলে তো growth curve বোঝা যায় না—তাহলে `N` বাড়িয়ে কীভাবে empirically দেখব যে code-টি `O(1)`, `O(N)`, `O(N log N)`, নাকি `O(N^2)` আচরণ করছে?**

---

## ৩. Multi-`N` Empirical Scaling এবং Doubling Ratio (`R(N) = T(2N) / T(N)`)

একটি algorithm-এর বাস্তব scaling behavior চোখে দেখার সবচেয়ে পরিষ্কার, সহজ, এবং শক্তিশালী কৌশল হলো **Controlled Multi-`N` Doubling Experiment**।

এখানে আমরা কেবল একটি `N` নয়, বরং প্রতি ধাপে input size **ঠিক দ্বিগুণ (`2×`)** করে পরপর পাঁচটি আকারে চালাই:
- **`N = 1,000`**
- **`N = 2,000`**
- **`N = 4,000`**
- **`N = 8,000`**
- **`N = 16,000`**

তারপর প্রতিটি ধাপে **Doubling Ratio** হিসাব করি:

$$
R(N) = \frac{T(2N)}{T(N)}
$$

এখানে `T(N)` হলো input size `N`-এ measured **median** সময়, আর `T(2N)` হলো input size `2N`-এ measured **median** সময়।

### তাত্ত্বিক প্রত্যাশা: কোন Growth Class-এ `R(N)` কেমন হওয়ার কথা?

আদর্শ ও পরিষ্কার শর্তে (যখন dominant term প্রাধান্য পাচ্ছে এবং পরিবেশগত noise কম), বিভিন্ন complexity class-এর জন্য `R(N)`-এর **আনুমানিক তাত্ত্বিক প্রত্যাশা** নিচের মতো:

| Growth Pattern | গাণিতিক অনুপাত `f(2N) / f(N)` | আনুমানিক Doubling Ratio `R(N)` | `N = 1,000 → 16,000` অঞ্চলে ব্যাখ্যা |
| :--- | :---: | :---: | :--- |
| **`O(1)`** | `c / c = 1` | **`~1×`** | Input দ্বিগুণ হলেও সময় প্রায় স্থির থাকে। |
| **`O(log N)`** | `log2(2N) / log2(N) = 1 + 1 / log2(N)` | **Slightly above `1×`** | `N = 1,000`-এ `~11/10 ≈ 1.10×`, `N = 16,000`-এ `~15/14 ≈ 1.07×`। |
| **`O(N)`** | `(c * 2N) / (c * N) = 2` | **`~2×`** | Input দ্বিগুণ হলে সময় প্রায় দ্বিগুণ হয়। |
| **`O(N log N)`** | `2 * (1 + 1 / log2(N))` | **Slightly above `2×`** | `N = 1,000 → 16,000`-এ প্রায় `~2.1× – 2.2×` (`2×`-এর সামান্য ওপরে!)। |
| **`O(N^2)`** | `(c * (2N)^2) / (c * N^2) = 4` | **`~4×`** | Input দ্বিগুণ হলেই সময় প্রায় **চার গুণ** হয়ে যায়! |

### অত্যন্ত জরুরি সতর্কতা: এগুলো Theoretical Expectations, Guaranteed Observed Ratios নয়!

বাস্তব machine-এ আপনি যখন doubling experiment চালাবেন, তখন কখনোই `2.000×` বা `4.000×` ছাঁচে ঢালা সংখ্যা আশা করবেন না! নিচের **পাঁচটি বাস্তব কারণ** observed `R(N)`-কে তাত্ত্বিক মানের চেয়ে ওপরে বা নিচে সরিয়ে দিতে পারে:
1. **Constant Overhead / Fixed Setup:** ছোট `N`-এ (যেমন `N = 1,000`) যদি function-এর ভেতর ছোট একটা fixed overhead থাকে, তবে প্রথম দিকের `R(N)` `2×`-এর চেয়ে কম (যেমন `1.5×`) দেখাতে পারে এবং `N` বড় হলে তবেই `~2×`-এ স্থির হয়।
2. **Cache / Memory Locality Boundary (Block 5!):** `N = 4,000`-এ পুরো working set হয়তো CPU-র দ্রুত L1/L2 cache-এ এঁটে গিয়েছিল, কিন্তু `N = 8,000` বা `16,000`-এ তা cache ছাড়িয়ে main RAM-এ চলে যাওয়ায় একটি `Θ(N)` algorithm-ও ঠিক ওই ধাপে হঠাৎ `2.6×` বা `3.0×` jump দেখাতে পারে!
3. **JIT Warmup ও GC Pauses:** Warmup পর্যাপ্ত না হলে প্রথম ছোট `N`-টা তুলনামূলক ধীর দেখায় (ফলে প্রথম `R(N)` অস্বাভাবিক কম হয়!), আর বড় `N`-এ হঠাৎ GC চললে কোনো একটি ধাপের ratio অস্বাভাবিক বেশি হতে পারে।
4. **Practical Crossover Points:** দুটি term-এর যোগফল (যেমন `c1 * N + c2 * N^2`) থাকলে ছোট `N`-এ `R(N) ≈ 2×` আর বড় `N`-এ `R(N) ≈ 4×` হতে দেখা যায়!
5. **`O(N)` বনাম `O(N log N)`-এর সূক্ষ্ম ব্যবধান:** `2.0×` আর `2.15×`-এর পার্থক্য অনেক সময় বাস্তব noise-এর ভেতর মিশে যায়—তাই শুধু একটা ratio দেখেই `O(N)` আর `O(N log N)` আলাদা করা যায় না!

**অতএব নিয়ম একদম পরিষ্কার:**
> **কখনোই একটিমাত্র noisy doubling ratio দেখে কোনো algorithm-কে classify করবেন না!** একাধিক input size (`1,000 → 16,000`), একাধিক trial (median + min/max spread), এবং **static code analysis** (Blocks 1–6)—এই তিনটিকে একসাথে মিলিয়ে সিদ্ধান্ত নিন!

---

## ৪. Controlled Node.js 26 Active LTS Benchmark Harness ও Falsifiable Experiment Discipline

এবার সব নীতিকে একসাথে বসিয়ে একটি clean, zero-dependency, **Pure Node.js 26 Active LTS** benchmark harness তৈরি করি *(মনে রাখবেন: Phase 2-এর approved runtime হলো **Node.js 26 Active LTS (`Lithium`)**—যা একটি Active LTS release track, Alpha বা Current release থেকে আলাদা; Phase 1-এর Node.js 22 environment অপরিবর্তিত থাকবে)*।

### এক লাইন Benchmark Code লেখার আগে: ৬-প্রশ্নের Falsifiable Experiment Protocol

যেকোনো scaling benchmark চালানোর আগে আপনার notes-এ নিচের **৬টি প্রশ্নের** উত্তর লিখুন (Phase 1 Block 5-এর scientific method!):
1. **ঠিক কোন operation-টি মাপা হচ্ছে?** (যেমন শুধু lookup, নাকি index তৈরি + lookup একসাথে?)
2. **কোন input variable বদলাচ্ছে?** (`N` দ্বিগুণ হচ্ছে, `M` স্থির থাকছে নাকি `M`-ও `N`-এর সাথে দ্বিগুণ হচ্ছে?)
3. **Static theory কোন growth এবং কোন doubling ratio predict করছে?**
4. **ঠিক কোন observed ফলাফল আপনার prediction-কে সমর্থন (support) করবে, আর কোন ফলাফল একে খণ্ডন (falsify) করবে?**
5. **কোন কোন পরিবেশগত বা runtime কারণ (cache boundary, GC, background CPU load, constant overhead) deviation ব্যাখ্যা করতে পারে?**
6. **Output correctness স্বাধীনভাবে কীভাবে যাচাই করা হবে?** (`node:assert/strict` দিয়ে!)

### তিনটি ভুল যা পুরো Comparison-কে বাতিল (Invalidate) করে দেয়

Harness লেখার সময় এই তিনটি ভুল কখনোই করবেন না:
1. **Hot loop-এর ভেতর `console.log()` বা I/O রাখা:** এতে terminal I/O timing dominate করে মূল algorithm-এর খরচ ঢেকে দেয়।
2. **Timed section-এর ভেতর input data তৈরি বা allocate করা** (বিশেষ করে এক candidate-এ করলে আর অন্যটিতে না করলে!): যদি আপনি input generation না মেপে কেবল lookup মাপতে চান, তবে test dataset **আগে থেকেই (outside the timed section)** তৈরি করে রাখুন!
3. **One-time setup ও per-query timing গুলিয়ে ফেলা:** যদি আপনার production workload-এ প্রতি request-এ নতুন করে `Map` বানাতে হয়, তবে `Map` build সময় timed section-এর **ভেতরে** থাকবে; আর যদি `Map` startup-এ একবার তৈরি হয়ে হাজার request-এ ব্যবহৃত হয়, তবে সেটা স্পষ্টভাবে আলাদা করে মাপুন—কখনোই তুলনা করার সময় দুটির সংজ্ঞা বদলাবেন না!

### সম্পূর্ণ, Runnable Controlled Benchmark Harness (`.mjs`, Zero Dependencies)

নিচে একটি সম্পূর্ণ, zero-dependency `.mjs` উদাহরণ দেওয়া হলো যা আমাদের online store-এর **promo-tag membership check** (`Array.includes` বনাম `Set.has` build+check)-এর জন্য clean multi-`N` experiment দেখায়। *(লক্ষ করুন: আমরা এখানে কোনো fabricated timing সংখ্যা লিখিনি—আপনি নিজে Node.js-এ চালালে আপনার নিজের machine-এর প্রকৃত median, min–max spread, ও `R(N)` দেখতে পাবেন!)*

```javascript
// controlled-scaling-demo.mjs (Pure Node.js 26 Active LTS, zero external dependencies)
import assert from "node:assert/strict";
import { performance } from "node:perf_hooks";

// Candidate A: Repeated Array.includes() across N unique tags -> O(N^2) worst-case
function countAllowedNaive(incomingTags, allowedTags) {
  let matches = 0;
  for (let i = 0; i < incomingTags.length; i++) {
    if (allowedTags.includes(incomingTags[i])) {
      matches += 1;
    }
  }
  return matches;
}

// Candidate B: Build Set once + O(1) expected lookups -> O(N + M) expected time, Θ(M) aux space
function countAllowedWithSet(incomingTags, allowedTags) {
  const allowedSet = new Set(allowedTags);
  let matches = 0;
  for (let i = 0; i < incomingTags.length; i++) {
    if (allowedSet.has(incomingTags[i])) {
      matches += 1;
    }
  }
  return matches;
}

// Deterministic input generator (called OUTSIDE the timed section!)
function generateTagFixture(n) {
  const allowedTags = [];
  const incomingTags = [];
  for (let i = 0; i < n; i++) {
    allowedTags.push(`PROMO_${i}`);
    // Half match near the end of allowedTags, half miss -> representative heavy-scan workload
    incomingTags.push(i % 2 === 0 ? `PROMO_${n - 1 - (i % 100)}` : `UNKNOWN_${i}`);
  }
  return { incomingTags, allowedTags };
}

function summarizeTrials(samplesMs) {
  const sorted = samplesMs.slice().sort((a, b) => a - b);
  const mid = Math.floor(sorted.length / 2);
  const median =
    sorted.length % 2 === 1 ? sorted[mid] : (sorted[mid - 1] + sorted[mid]) / 2;
  return {
    medianMs: median,
    minMs: sorted[0],
    maxMs: sorted[sorted.length - 1],
  };
}

function benchmarkCandidate(fn, fixture, { warmupRuns = 3, trials = 7 } = {}) {
  let sinkChecksum = 0;

  // 1. Untimed warmup runs
  for (let w = 0; w < warmupRuns; w++) {
    sinkChecksum += fn(fixture.incomingTags, fixture.allowedTags);
  }

  // 2. Measured trials using monotonic performance.now()
  const samplesMs = [];
  for (let t = 0; t < trials; t++) {
    const startMs = performance.now();
    const result = fn(fixture.incomingTags, fixture.allowedTags);
    const elapsedMs = performance.now() - startMs;
    sinkChecksum += result; // lightweight observable consumption outside hot inner work
    samplesMs.push(elapsedMs);
  }

  return { ...summarizeTrials(samplesMs), sinkChecksum };
}

function runScalingExperiment() {
  const sizes = [1_000, 2_000, 4_000, 8_000, 16_000];
  let totalObservableSink = 0;
  let prevNaiveMedian = null;
  let prevSetMedian = null;

  for (const n of sizes) {
    const fixture = generateTagFixture(n);

    // Independent correctness check BEFORE timing!
    const expected = countAllowedNaive(fixture.incomingTags, fixture.allowedTags);
    const actual = countAllowedWithSet(fixture.incomingTags, fixture.allowedTags);
    assert.equal(actual, expected, `Output mismatch at N=${n}`);

    const naiveStats = benchmarkCandidate(countAllowedNaive, fixture);
    const setStats = benchmarkCandidate(countAllowedWithSet, fixture);
    totalObservableSink += naiveStats.sinkChecksum + setStats.sinkChecksum;

    const naiveRatio = prevNaiveMedian ? (naiveStats.medianMs / prevNaiveMedian).toFixed(2) : "—";
    const setRatio = prevSetMedian ? (setStats.medianMs / prevSetMedian).toFixed(2) : "—";
    prevNaiveMedian = naiveStats.medianMs;
    prevSetMedian = setStats.medianMs;

    console.log(
      `N=${String(n).padStart(6)} | ` +
      `Naive median=${naiveStats.medianMs.toFixed(3)}ms [${naiveStats.minMs.toFixed(3)}..${naiveStats.maxMs.toFixed(3)}] R=${naiveRatio}x | ` +
      `Set median=${setStats.medianMs.toFixed(3)}ms [${setStats.minMs.toFixed(3)}..${setStats.maxMs.toFixed(3)}] R=${setRatio}x`
    );
  }

  // Print aggregate sink once at the end so all trial results remain observable
  console.log(`Observable checksum: ${totalObservableSink}`);
}

runScalingExperiment();
```

*(একটি সূক্ষ্ম Active Benchmarking লক্ষণীয় বিষয়: এখানে `Set` candidate-এ `allowedSet` build করাটা timed section-এর ভেতরেই আছে—কারণ আমরা একটি প্রতি-batch one-shot call মাপছি যেখানে index আগে থেকে cached নয়! আপনি যদি startup-cached `Set` মাপতেন, তবে সেটি স্পষ্টভাবে আলাদা hypothesis হিসেবে লিখতে হতো।)*

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-07-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. ধরুন কোনো একটি operation-এর তাত্ত্বিক complexity `Θ(N * M)` (যেমন `N`টি order-কে `M`টি catalog entry-র সাথে `.find()` করে মেলানো)।
>    - *(ক)* আপনি যদি `M = 2,000` **স্থির** রেখে কেবল `N`-কে দ্বিগুণ (`1,000 → 2,000 → 4,000`) করেন, তবে তাত্ত্বিক doubling ratio `R(N)` আনুমানিক কত আশা করবেন?
>    - *(খ)* আর যদি `M`-ও প্রতি ধাপে `N`-এর সমান করে দ্বিগুণ করেন (`M = N`), তবে তাত্ত্বিক `R(N)` আনুমানিক কত আশা করবেন?
> 2. `process.hrtime.bigint()` ব্যবহার করে একটি trial-এ আপনি পেলেন `elapsedNs = 4_250_000n`। এটি কত microseconds (`µs`) এবং কত milliseconds (`ms`)? আর ধরুন `N = 4,000 → 8,000` ধাপে একটি `Θ(N)` algorithm-এর observed ratio এল `2.8×`—কেন কেবল এই একটি সংখ্যা দেখেই একে `Θ(N^2)` ঘোষণা করা ভুল হবে?

---

## ৫. Active Benchmarking ও সৎ Performance Defense: একটি In-Memory Benchmark কী প্রমাণ **করে না**?

একজন junior developer benchmark চালিয়ে বলেন: *"আমার Node.js script-এ `N = 100,000` মাত্র `35 ms`-এ শেষ হয়েছে—তাই আমাদের API `100 ms` production SLA আরামে পাস করবে!"*

একজন mature backend engineer (Phase 1 Block 8-এর ACM `1.3` honesty + Block 5-এর I/O reality!) সাথে সাথেই থেমে দুটি তালিকা আলাদা করেন:
- **এই Controlled In-Memory Benchmark কী প্রমাণ করেছে:** নির্দিষ্ট Node.js 26 runtime, নির্দিষ্ট CPU/RAM, ও নির্দিষ্ট synthetic input shape-এ আমাদের algorithmic refactor-টি in-memory CPU scaling-কে (যেমন `O(N^2)` → `O(N)` expected) empirically সমর্থন করেছে এবং output correctness অটুট রেখেছে।
- **এই In-Memory Benchmark কী কখনোই প্রমাণ করে না:**
  1. **Real PostgreSQL I/O performance** (disk read/write, query execution, index page fetch — Phase 10 ও 14),
  2. **Network RTT বা remote service behavior** (datacenter বা cross-region latency, packet loss, downstream retries — Phase 4 ও 21),
  3. **Multi-request concurrency behavior** (শত শত concurrent request একসাথে এলে event-loop contention বা resource saturation — Phase 6 ও 11),
  4. **Long-term GC pressure under production workloads** (ঘণ্টার পর ঘণ্টা চলা live process-এর heap retention ও tail pauses — Phase 5 ও 25), এবং
  5. **Production SLA compliance** (পুরো end-to-end HTTP stack ও real traffic distribution-এর latency percentiles — Phase 20 ও 25)।

এই সৎ সীমারেখা স্বীকার করা কোনো দুর্বলতা নয়—এটাই প্রমাণ করে আপনি **Active Benchmarking** বোঝেন এবং আপনার measurement-এর ওপর অন্ধ বিশ্বাস করেন না!

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-07-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন teammate আমাদের `storeBatchEngine.mjs`-এর refactored version চালিয়ে বললেন: *"আমার laptop-এ `N = 100,000` order enrich করতে median `45 ms` লেগেছে, তাই production checkout API-র `150 ms` p95 SLA নিশ্চিত!"* এই দাবির অন্তত **৩টি অপ্রমাণিত (unverified) production dimension** চিহ্নিত করে সৎ engineering ভাষায় দাবিটি পুনর্লিখন করুন।
> 2. Benchmark লেখার আগে আমাদের **৬-প্রশ্নের Falsifiable Experiment Protocol**-এ *"কোন ফলাফল আমার prediction-কে খণ্ডন (falsify) করবে?"* এবং *"correctness কীভাবে স্বাধীনভাবে যাচাই হবে?"*—এই দুটি প্রশ্ন আগে থেকেই ঠিক করে রাখা কেন জরুরি?

---

## Study Plan, Learning Objectives, Terminology & Resources

> **Continuity & Ownership Notes:** আপনার সব Pause & Checks, Pre-Lab Exercise, 4-Stage Capstone Scaling Lab-এর লিখিত বিশ্লেষণ/ফলাফল/defense, এবং (যদি করতে চান) Optional Closed-Book Synthesis Assessment-এর উত্তর স্বাধীনভাবে `phases/phase-02/notes/block-07-notes.md` ফাইলে লিখুন। AI-generated material বা agent research কখনো completed learning বা learner study hours হিসেবে গণ্য হয় না।
>
> **Progression & Verification Policy:** এই block-এর Closed-Book Synthesis Assessment সম্পূর্ণ **optional** এবং কোনো mandatory progression gate নয়। তবে **Phase 2 সম্পূর্ণ শেষ করে Phase 3 (*Operating Systems*) unlock করতে হলে** আপনার Blocks 1–7 notes, verified Capstone Lab evidence (`phases/phase-02/lab/` — Phase 2 unlock হওয়ার পর সম্পন্ন), এবং `phases/phase-02/checkpoint.md` সংকলন করে **ChatGPT (Curriculum Owner & Instructor)**-এর formal mastery review ও `approved` sign-off নিতে হবে।

### Study Time Reference

*প্রতিটি activity-র বিস্তারিত time breakdown দেখতে এই নথির শুরুর **Study Time Guide** দেখুন। Approved dossier-এ Block 7-এর roadmap target হলো `4 hours (240 min)`—যা একটি flexible planning reference, কোনো strict deadline নয়।*

### Block 7 Learning Objectives *(Primary Target Level: `Practical Competence` | Supported Phase 2 Outcomes: Outcomes 1–8 Integrative Capstone)*

- **LO 7.1 (`Working Understanding` | Bloom: *Evaluate*):** Explain the complementary roles and limits of **theoretical complexity analysis** vs. **empirical benchmarking**—why measurements alone can never *prove* an asymptotic bound (finite `N`, machine/input specificity) and why theory alone cannot predict exact wall-clock latency (constant factors, JIT, locality, GC).
- **LO 7.2 (`Practical Competence` | Bloom: *Analyze*):** Diagnose the five foundational **microbenchmarking pitfalls**: (1) **Dead-Code Elimination (DCE)** & constant folding (as conditional optimization distortions when an engine proves a computation is side-effect-free and unused—plus ensuring any result-consumption sink does not itself dominate timing), (2) **JIT warmup** (cold vs. steady-state execution), (3) **GC & allocation noise**, (4) **wall-clock `Date.now()` instability** vs. monotonic high-resolution clocks (`performance.now()` / `process.hrtime.bigint()`), and (5) **Passive Benchmarking** (single-run, mean-only, single/unrepresentative input, unverified work).
- **LO 7.3 (`Practical Competence` | Bloom: *Apply / Analyze*):** Design and run a controlled **multi-`N` doubling/scaling experiment** in Pure Node.js 26 Active LTS (`node:perf_hooks`) with warmup, lightweight observable result consumption, multiple trials (median & min/max spread), and varying input sizes (`N = 1,000; 2,000; 4,000; 8,000; 16,000`) to empirically distinguish growth curves—and separate algorithmic slowness from environmental/constant-factor noise.
- **LO 7.4 (`Practical Competence` | Bloom: *Evaluate / Create*):** Synthesize **all seven Phase 2 blocks** in an end-to-end **Phase 2 Capstone Scaling Lab**: statically analyze a realistic multi-operation online-store module, predict scaling ratios, measure empirically, refactor with explicit time-space trade-offs, verify 100% output equivalence via `node:test`, and defend remaining measurement limitations honestly.

### Terminology Inventory

| Term | Official Target Level | Why It Matters in Block 7 |
| :--- | :---: | :--- |
| **Monotonic High-Resolution Clock (`performance.now` / `hrtime.bigint`)** | `Practical Competence` | Immune to system wall-clock jumps and offers sub-millisecond / nanosecond resolution where `Date.now()` fails. |
| **Dead-Code Elimination (DCE) & Observable Result Sinks** | `Working Understanding` | Conditional compiler/runtime optimization that can distort unused pure benchmark loops; mitigated by a lightweight observable sink that never dominates timing. |
| **JIT Warmup (Cold vs. Steady State)** | `Working Understanding` | Separates initial compilation/startup cost from steady-state measurements *(V8 compilation-tier internals deferred to Phase 5)*. |
| **Empirical Scaling & Doubling Ratio (`R(N) = T(2N) / T(N)`)** | `Practical Competence` | Empirically tests predicted scaling trends (`~1×`, `~2×`, `~4×`) across `N = 1,000..16,000` while treating them as expectations rather than guaranteed observed values. |
| **Active vs. Passive Benchmarking (Gregg)** | `Working Understanding` | Prevents measuring the wrong workload (*"benchmark A, measure B, conclude C"*) through falsifiable hypotheses, multi-trial medians + min/max spread, and active verification. |

### Mandatory Authoritative Resources *(Targeted Reading — ~45–55 min total)*

1. **Mandatory 1 (~15–20 min — assigned sections only):** Brendan Gregg — [Active Benchmarking](https://www.brendangregg.com/activebenchmarking.html) (`LO 7.2`, `LO 7.3`):
   - **Assigned scope:** কেবল **Summary**, **Passive Benchmarking**, **Active Benchmarking**, **Problem Checklist**, এবং **Statistical Analysis** overview পড়ুন। **Linux `perf` এবং DTrace command details explicitly skip করুন** (সেগুলো Phases 23 ও 25-এ)।
   - **What to extract:** *"you benchmark A, actually measure B, and conclude you've measured C"* mental model এবং কীভাবে active verification benchmark বিভ্রম ঠেকায়।
2. **Mandatory 2 (~20–25 min total — assigned sections only):** Vyacheslav Egorov (`mraleph`, former V8 engineer) (`LO 7.1`, `LO 7.2`):
   - [microbenchmarks fairy tale](https://mrale.ph/blog/2012/12/15/microbenchmarks-fairy-tale.html) — কেবল **Loop-invariant code motion (LICM)**, **Constant propagation**, **Dead code elimination (DCE)**, এবং **Moral?** অংশ *(assembly dumps বাদ দিন)*।
   - [Microbenchmarks are experiments](https://mrale.ph/blog/2024/11/27/microbenchmarks-are-experiments.html) — opening thesis, বিশেষত **"Benchmarks are not numerology"** ভাবনাটি *(assembly/deep compiler internals বাদ দিন)*।
   - **What to extract:** Optimizing compiler/runtime কীভাবে নির্দিষ্ট শর্তে naive JS loop benchmark বিকৃত করতে পারে এবং কেন benchmark-কে সংখ্যাতত্ত্ব (numerology) নয় বরং falsifiable experiment হিসেবে দেখতে হয়।
3. **Mandatory 3 (~10 min — targeted skim):** Official Node.js 26 Active LTS Documentation (`LO 7.2`, `LO 7.3`):
   - [Performance measurement APIs (`node:perf_hooks`, v26.x)](https://nodejs.org/docs/latest-v26.x/api/perf_hooks.html) — **`performance.now()`** (monotonic high-resolution millisecond `<number>` timestamp, called with `performance` as receiver)।
   - [Process API (`process.hrtime.bigint`, v26.x)](https://nodejs.org/docs/latest-v26.x/api/process.html#processhrtimebigint) — **`process.hrtime.bigint()`** (monotonic high-resolution nanosecond `<bigint>` timestamp)।
   - **What to extract:** Pure Node.js 26 Active LTS-এ zero-dependency, safe monotonic timing primitives ও তাদের একক (`ms` বনাম `ns`)।

### Optional Supplementary Resource *(Outside Mandatory Budget — Do Not Treat as Mandatory)*

- **Supplementary (Free):** V8 Engineering Blog — [How V8 measures real-world performance](https://v8.dev/blog/real-world-performance) — synthetic microbenchmark কেন বাস্তব production workload থেকে আলাদা হতে পারে তার conceptual skim।

---

## Pre-Lab Exercise: Spot the Broken Microbenchmark *(Mandatory — ~15–20 min | `LO 7.2`)*

> **Ownership Rule:** এই অনুশীলনের সব flaw diagnosis এবং clean rewritten benchmark harness স্বাধীনভাবে `phases/phase-02/notes/block-07-notes.md` ফাইলে লিখুন। এখানে সম্পূর্ণ সমাধান দেওয়া হয়নি (নথির শেষে কেবল উচ্চ-স্তরের self-check criteria রয়েছে)।

**Scenario:** একজন developer নিচের ~30-line Node.js script-টি লিখে দাবি করেছেন যে তাঁর নতুন `deduplicateFast` ফাংশনটি `deduplicateNaive`-এর চেয়ে বিপুলভাবে দ্রুত। Script-টিতে **চারটি গুরুতর microbenchmarking flaw** লুকিয়ে আছে:

```javascript
// broken-benchmark.mjs — contains 4 classic microbenchmarking flaws!
function deduplicateNaive(ids) {
  const seen = [];
  for (const id of ids) {
    if (!seen.includes(id)) seen.push(id);
  }
  return seen;
}

function deduplicateFast(ids) {
  return Array.from(new Set(ids));
}

// Benchmark 1
const t1 = Date.now();
const smallSorted = Array.from({ length: 50 }, (_, i) => `ORD_${i}`);
for (let i = 0; i < 10_000; i++) {
  deduplicateNaive(smallSorted); // return value unused!
  if (i % 100 === 0) console.log("progress:", i);
}
console.log("Naive ms:", Date.now() - t1);

// Benchmark 2
const t2 = Date.now();
for (let i = 0; i < 10_000; i++) {
  const freshCopy = Array.from({ length: 50 }, (_, j) => `ORD_${j}`);
  deduplicateFast(freshCopy); // return value unused!
}
console.log("Fast ms:", Date.now() - t2);
```

**Tasks (`phases/phase-02/notes/block-07-notes.md`):**
1. **Identify All 4 Flaws:** Script-টিতে থাকা চারটি benchmarking flaw চিহ্নিত করুন (`Date.now()` + single run; unused pure computation / conditional DCE risk এবং hot-loop `console.log` I/O distortion; single tiny `N = 50` already-sorted input ও zero warmup; timed section-এর ভেতর asymmetric input allocation/setup mixing)।
2. **Explain Each Distortion Mechanism:** প্রতিটি flaw কীভাবে পরিমাপটিকে অবৈধ বা বিভ্রান্তিকর করে তোলে তা ১–২ বাক্যে ব্যাখ্যা করুন।
3. **Clean Rewrite:** Block 7-এর নীতি অনুসরণ করে Pure Node.js 26 Active LTS (`node:perf_hooks`, pre-generated multi-`N` inputs outside timed regions, warmup, multiple trials, median + min/max spread, lightweight observable checksum sink, এবং independent `assert.deepEqual` correctness check)-এ benchmark harness-টি পরিষ্কারভাবে পুনরায় লিখুন।

---

## Phase 2 Capstone Scaling Laboratory *(Mandatory — ~100–120 min | `LO 7.4`, Outcomes 1–8 Integrative)*

> **Repository Governance & Unlock Rule:** `AGENTS.md`, `PROGRESS.md`, এবং approved Phase 2 Research Dossier (Section 6, Block 7) অনুযায়ী **`phases/phase-02/lab/` ফোল্ডারের ফাইলগুলো এই pre-study authoring ধাপে তৈরি করা হয়নি**—Phase 1 `approved` হয়ে **Phase 2 unlock হওয়ার পর** আপনি নিজেই নিচের চারটি ধাপ চালিয়ে আপনার lab ফাইলগুলো তৈরি ও execute করবেন:
> - **Code & Experiments (when Phase 2 unlocks):**
>   - `phases/phase-02/lab/storeBatchEngine.mjs`
>   - `phases/phase-02/lab/storeBatchEngine.test.mjs`
>   - `phases/phase-02/lab/benchmarkScaling.mjs`
> - **Written Predictions, Observed Tables, Trade-Offs & Honest Defense:** `phases/phase-02/notes/block-07-notes.md`

### Strict Lab Environment & Scope Constraints

- **Runtime:** **Node.js 26 Active LTS (`Lithium`)** — pure ES Modules (`.mjs`)। *(Phase 1-এর Node.js 22 পরিবেশ অপরিবর্তিত রাখুন।)*
- **Zero External Dependencies:** কোনো `npm` package নয়—কেবল built-in **`node:perf_hooks`**, **`node:process`**, **`node:test`**, এবং **`node:assert/strict`**।
- **Explicitly Excluded:** কোনো HTTP server, PostgreSQL database, external benchmark framework, V8 profiler flag (`--cpu-prof`, `--trace-opt`, ইত্যাদি), বা production load generator (`k6`/`autocannon`) ব্যবহার করবেন না।

---

### Capstone Scenario & Starter Baseline Module (`storeBatchEngine.mjs`)

আমাদের online store-এর একটি pure-JavaScript analytics ও checkout helper module (`storeBatchEngine.mjs`) তিনটি মূল কাজ করে:
- **Operation A (`enrichOrdersWithCatalog`):** `N`টি order-কে `M`টি customer/promo catalog entry-র সাথে `customerId` ধরে যুক্ত করা।
- **Operation B (`deduplicatePaymentEvents`):** `N`টি incoming payment/webhook event থেকে duplicate `eventId` বাদ দেওয়া (first-seen ক্রম বজায় রেখে)।
- **Operation C (`extractTopKHighValueOrders`):** `N`টি order record থেকে সর্বোচ্চ `amountCents`-যুক্ত **Top-`K`** order বের করা (descending `amountCents`, সমান হলে ascending `orderId` deterministic tie-breaker সহ)।

Staging-এ `N = 50` সাইজের সব unit test চোখের পলকে pass করেছে—কিন্তু `N = 50,000` সাইজের batch run-এ module-টি timeout খেয়ে আটকে যাচ্ছে!

Phase 2 unlock হলে নিচের **Starter Baseline Module**-টি `phases/phase-02/lab/storeBatchEngine.mjs`-এ নিয়ে Stage 1 থেকে Stage 4 শুরু করুন:

```javascript
// phases/phase-02/lab/storeBatchEngine.mjs (Starter Baseline — BEFORE refactoring)

// Operation A: Enrich N orders with M customer/promo catalog entries
export function enrichOrdersWithCatalog(orders, catalogEntries) {
  return orders.map((order) => {
    const entry = catalogEntries.find((c) => c.customerId === order.customerId);
    return {
      orderId: order.orderId,
      customerId: order.customerId,
      amountCents: order.amountCents,
      tier: entry ? entry.tier : "GUEST",
      promoEligible: entry ? entry.promoEligible : false,
    };
  });
}

// Operation B: Deduplicate N incoming payment/webhook events (preserving first-seen order)
export function deduplicatePaymentEvents(events) {
  const seenEventIds = [];
  const uniqueEvents = [];
  for (const event of events) {
    if (!seenEventIds.includes(event.eventId)) {
      seenEventIds.push(event.eventId);
      uniqueEvents.push(event);
    }
  }
  return uniqueEvents;
}

// Operation C: Extract Top-K highest-value orders from N records
// Exact ordering contract: descending by amountCents; ties broken ascending by orderId.
export function extractTopKHighValueOrders(orders, k) {
  if (k <= 0) return [];
  const copy = orders.slice();
  copy.sort((a, b) => {
    if (b.amountCents !== a.amountCents) {
      return b.amountCents - a.amountCents;
    }
    return a.orderId < b.orderId ? -1 : a.orderId > b.orderId ? 1 : 0;
  });
  return copy.slice(0, k);
}
```

---

### Stage 1 — Static Theoretical Prediction *(~25–30 min | Integrates Blocks 1–5)*

**কোনো benchmark চালানোর আগেই** আপনার `phases/phase-02/notes/block-07-notes.md`-এ লিখুন:
1. **Exact Complexity Derivation (Operations A, B, C):** স্বাধীন variables **`N`**, **`M`**, এবং **`K`**-এ প্রতিটির tight time complexity এবং auxiliary space complexity derive করুন *(কোনো variable এক করে ফেলবেন না!)*।
2. **Doubling Ratio Predictions (`R(N) = T(2N) / T(N)`):**
   - Operation A-র জন্য **দুটি ভিন্ন experiment setup**-এ `R(N)` predict করুন: (১) `M` fixed রেখে `N` দ্বিগুণ করলে, এবং (২) `M = N` ধরে উভয়ই দ্বিগুণ করলে!
   - Operation B-র জন্য (প্রায় সব `eventId` unique ধরে) এবং Operation C-র জন্য (`K = 10` fixed রেখে `N` দ্বিগুণ করলে) `R(N)` predict করুন।
3. **Pre-Implementation Auxiliary Memory Estimate at `N = 100,000` (Block 5 Discipline):**
   - প্রতিটি operation-এর জন্য `N = 100,000` (ও ধরা যাক `M = 20,000`, `K = 10`) অবস্থায় raw record payload এবং auxiliary working set অনুমান করুন—**অবশ্যই আপনার সব assumption স্পষ্টভাবে লিখে** (কোনো unsupported universal JavaScript multiplier দাবি না করে, এবং raw payload বনাম in-memory representation বনাম total process memory-র পার্থক্য পরিষ্কার রেখে!)।
4. **Falsifiable Hypotheses (৬-প্রশ্নের Protocol):** প্রতিটির জন্য লিখুন ঠিক কোন observed pattern আপনার তাত্ত্বিক পূর্বাভাসকে সমর্থন করবে, আর কোন pattern একে খণ্ডন করবে।

---

### Stage 2 — Controlled Baseline Scaling Experiment *(~25–30 min | Integrates Blocks 5 & 7)*

`phases/phase-02/lab/benchmarkScaling.mjs` তৈরি করে baseline `storeBatchEngine.mjs`-এর ওপর নিচের কঠোর শর্তে multi-`N` scaling experiment চালান:
- **Input Sizes:** `N = 1,000; 2,000; 4,000; 8,000; 16,000` *(Operation A-র জন্য স্পষ্টভাবে উল্লেখ করুন আপনি `M` fixed রেখেছেন নাকি `M = N` রেখেছেন—সর্বোত্তম হলো `M = N` বা `M = N / 2` রেখে `O(N * M)` বিস্ফোরণ স্পষ্টভাবে দেখা, আর `K = 10` fixed রাখা!)*।
- **Harness Requirements:**
  - Monotonic `performance.now()` (`node:perf_hooks`) বা `process.hrtime.bigint()`,
  - Untimed **warmup** runs,
  - **Lightweight observable checksum sink** (timed section-এর বাইরে printed),
  - প্রতি `N`-এ **একাধিক trial** (যেমন `5–7` trials) এবং রিপোর্ট হিসেবে **median**, **min**, **max** (spread), ও **observed `R(N)`**,
  - সব test fixture **timed section-এর বাইরে** deterministically pre-generated।
- **Record Actual Results Honestly:** আপনার নিজের machine-এ পাওয়া **আসল** সংখ্যাগুলো টেবিলে লিখুন (কখনোই সংখ্যা বানিয়ে লিখবেন না!)। Stage 1-এর পূর্বাভাসের সাথে তুলনা করুন এবং কোনো crossover, warmup, cache boundary, বা GC spike দেখা গেলে তা সৎভাবে ব্যাখ্যা করুন।

---

### Stage 3 — Complexity Refactoring & Correctness Preservation *(~30–35 min | Integrates Blocks 4 & 6)*

এবার `storeBatchEngine.mjs`-এ তিনটি refactored version (যেমন `enrichOrdersWithCatalogFast`, `deduplicatePaymentEventsFast`, `extractTopKHighValueOrdersFast`) তৈরি করুন যাতে **100% identical output ও ordering semantics** রক্ষা হয়:
- **Operation A Refactor:** `Map` index দিয়ে `O(N + M)` expected time, `Θ(N + M)` auxiliary space (duplicate `customerId` থাকলে `.find()`-এর first-match আচরণ রক্ষা করুন!)।
- **Operation B Refactor:** `Set` দিয়ে `O(N)` expected time, `O(N)` auxiliary space (first-seen order রক্ষা করুন!)।
- **Operation C Refactor:** একটি **bounded `O(K)` Min-Heap** (অথবা স্পষ্টভাবে যুক্তিযুক্ত partial-selection strategy) দিয়ে `O(N log K)` time (for `K >= 2`; `O(N)` for `K = 1`) এবং `O(K)` auxiliary space—exact descending `amountCents` + ascending `orderId` tie-breaker এবং caller-এর `orders` array immutability রক্ষা করে!
- **Independent Verification Suite (`phases/phase-02/lab/storeBatchEngine.test.mjs`):**
  - Built-in `node:test` + `node:assert/strict` দিয়ে `node --test phases/phase-02/lab/storeBatchEngine.test.mjs` চালিয়ে প্রমাণ করুন যে baseline এবং refactored উভয় implementation **সব edge case-এ 100% identical output দেয়**:
    1. Empty inputs (`[]`),
    2. `catalogEntries`-এ duplicate `customerId` (প্রথম match জেতে কি না) এবং missing `customerId` (`"GUEST"` fallback),
    3. সব unique events, সব duplicate events, এবং mixed duplicate events (exact first-seen order),
    4. `K = 0`, `K = 1`, `K < N`, `K > N`, এবং সমান `amountCents`-যুক্ত order-দের deterministic `orderId` tie-breaking,
    5. Caller-এর input arrays (`orders`, `catalogEntries`, `events`) কোনো ফাংশনেই mutate হয়নি!
- **Document Exact Trade-Offs:** প্রতিটির জন্য time gain বনাম auxiliary memory cost এবং cost-model assumptions (যেমন conventional expected `O(1)` hash-table model) নথিভুক্ত করুন।

---

### Stage 4 — After-Measurement Comparison & Honest Performance Defense *(~20–25 min | Integrates Blocks 5 & 7 + Phase 1 B8)*

1. **Re-Run the Exact Same Harness:** হুবহু একই `benchmarkScaling.mjs` দিয়ে refactored ফাংশনগুলো `N = 1,000 .. 16,000`-এ মাপুন—এবং তার পাশাপাশি **`N = 100,000`** (যেখানে baseline অতি ধীর হয়ে পড়ত) চালিয়ে বাস্তব scaling যাচাই করুন।
2. **Before vs. After Comparison Table (in `notes/block-07-notes.md`):**
   - প্রতিটি operation-এর জন্য পাশাপাশি দেখান: **Before vs. After Median**, **Min–Max Spread**, **Observed `R(N)`**, **Auxiliary Memory Trade-Off**, এবং **Passing `node:test` Output** (সবই আপনার machine-এ পাওয়া আসল output!)।
3. **Honest Performance Defense Statement (What This Proves vs. What It Does NOT Prove):**
   - স্পষ্টভাবে লিখুন এই পরীক্ষাটি **কী প্রমাণ করেছে** (in-memory CPU scaling improvement + verified output equivalence on this machine & Node.js 26 Active LTS)।
   - এবং স্পষ্টভাবে ঘোষণা করুন যে এই pure in-memory benchmark **কোনোভাবেই নিচের পাঁচটি জিনিস প্রমাণ করে না**:
     - Real PostgreSQL I/O performance,
     - Network RTT বা remote service behavior,
     - Multi-request concurrency behavior,
     - Long-term GC pressure under production workloads, এবং
     - Production SLA compliance।

---

## Closed-Book Synthesis Assessment *(Optional — ~25–30 min)*

এই assessment-টি সম্পূর্ণ **optional**—এটি কোনো mandatory progression gate নয়। আপনি চাইলে:
- সব notes ও browser tab বন্ধ রেখে নিচের ৫টি Phase-2 integrative প্রশ্নের নিজের ভাষায় উত্তর `phases/phase-02/notes/block-07-notes.md` ফাইলে লিখে formal evaluation-এর জন্য **ChatGPT**-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে);
- অথবা আপনি Capstone Lab দিয়ে দক্ষতা প্রমাণ করে assessment skip করতে পারেন (মনে রাখবেন: assessment skip করলে তা স্বয়ংক্রিয়ভাবে `Mastery verified` হয় না);
- অথবা assessment ছাড়াই যেকোনো unclear concept নিয়ে ChatGPT-র সাথে আলোচনা করতে পারেন।

*(Note: `~25–30 min` কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি approximate estimate—কোনো strict time limit নয়।)*

1. **(LO 7.1 / Outcomes 1, 8)** Theoretical complexity analysis এবং empirical benchmarking-এর পরস্পর-পরিপূরক ভূমিকা ও সীমারেখা ব্যাখ্যা করুন। কেন finite input size-এর পরিমাপ কখনোই একটি asymptotic bound prove করতে পারে না, আবার কেন Big-O একাই exact wall-clock runtime predict করতে পারে না?
2. **(LO 7.2 / Outcome 8)** পাঁচটি microbenchmarking pitfall—(1) conditional DCE & constant folding, (2) JIT warmup, (3) GC/allocation noise, (4) `Date.now()` বনাম monotonic `performance.now()`/`process.hrtime.bigint()`, এবং (5) Passive Benchmarking—প্রতিটির mechanism ও প্রতিকার ব্যাখ্যা করুন। কেন lightweight observable result sink-টি নিজে measured কাজকে dominate করা চলবে না?
3. **(LO 7.3 / Outcomes 1, 7, 8)** Multi-`N` doubling ratio `R(N) = T(2N) / T(N)` ব্যবহার করে `O(1)`, `O(log N)`, `O(N)`, `O(N log N)`, এবং `O(N^2)`-এর আনুমানিক তাত্ত্বিক প্রত্যাশা ব্যাখ্যা করুন। কোন কোন বাস্তব কারণে observed `R(N)` তাত্ত্বিক মান থেকে বিচ্যুত হতে পারে, এবং কেন একটিমাত্র noisy ratio থেকে algorithm classify করা ভুল?
4. **(LO 7.2, LO 7.3 / Outcomes 7, 8)** Brendan Gregg-এর Active Benchmarking (*"benchmark A, measure B, conclude C"*) এবং `mraleph`-এর *"Benchmarks are not numerology"* ভাবনা কীভাবে একটি controlled Node.js 26 experiment design-কে (৬-প্রশ্নের falsifiable protocol, median + min/max spread, pre-generated input, ও correctness check) চালিত করে?
5. **(LO 7.4 / Integrative Outcomes 1–8)** ধরুন একটি নতুন online-store batch feature `N = 50`-এ `2 ms` নেয় কিন্তু `N = 50,000`-এ timeout করে। Blocks 1–7-এর সব ধাপ একত্রে যুক্ত করে একটি সংহত **end-to-end performance reasoning & verification workflow** বর্ণনা করুন—static multi-variable analysis, case/amortized risk, data-structure & memory-budget trade-off, hardware/I/O dominance, controlled multi-`N` Node.js verification, correctness preservation, এবং honest limitation defense সহ।

---

## Phase 2-এর সমাপ্তি ও Phase 3 (Operating Systems)-এর দিকে সেতুবন্ধন

একটু পেছনে ফিরে তাকান। **Arc I — Foundations of Engineering Reasoning (Phases 1–2)** আজ একটি অসাধারণ সন্ধিক্ষণে এসে পৌঁছাল:
- **Phase 1**-এ আপনি শিখেছেন কীভাবে সমস্যা ফ্রেম করতে হয়, invariants ও trade-offs ভাবতে হয়, AI-কে যাচাই করতে হয়, এবং কাজের দায়িত্ব নিতে হয়।
- **Phase 2**-এ আপনি সেই mindset-কে **quantitative (গাণিতিক ও পরীক্ষামূলক)** রূপ দিলেন: Word-RAM model ও `O`/`Ω`/`Θ` (Block 1), loops ও recursion trees (Block 2), best/average/expected/worst/amortized ও algorithmic DoS (Block 3), core data structures ও sorting trade-offs (Block 4), constant factors, memory hierarchy, ও back-of-the-envelope resource sizing (Block 5), real backend traps ও AI code audit (Block 6), এবং controlled Node.js scaling experiments ও honest benchmark defense (Block 7)।

কিন্তু Block 5 ও Block 7 আমাদের ঠিক **Word-RAM abstraction-এর দেয়ালের কিনারায়** দাঁড় করিয়ে দিয়েছে—এবং তৈরি করেছে চারটি গভীর অমীমাংসিত প্রশ্ন, যেগুলো সরাসরি খুলে দেবে **Arc II (Phases 3–6)**:
1. **Phase 3 — Operating Systems:** *আমাদের `~1 ns` L1 cache, `~100 ns` RAM, আর `~10–100 µs` SSD-এর পেছনে CPU, virtual memory, OS process/thread scheduling, আর disk I/O আসলে ভেতরে কীভাবে কাজ করে?*
2. **Phase 4 — Computer Networking:** *একটি network round-trip কেন milliseconds নেয় এবং দুই মেশিনের মাঝে bytes কীভাবে নির্ভরযোগ্য ও নিরাপদভাবে যাতায়াত করে?*
3. **Phase 5 — JavaScript Language Internals:** *আমাদের Node.js benchmark-এ যে JIT warmup, object memory overhead, আর GC jitter দেখেছি—V8 ভেতরে আসলে কীভাবে code execute ও heap manage করে?*
4. **Phase 6 — Concurrency and Asynchronous Systems:** *আমাদের single-threaded `O(N)` বা `O(N^2)` handler-এর ওপর যখন একসাথে `500`টি concurrent request আছড়ে পড়ে, তখন কী ঘটে?*

### Block 7 Readiness & Phase 2 Checkpoint Handoff Checklist

- [ ] Guided lesson মনোযোগ দিয়ে পড়েছেন এবং Pause & Checks 1–3 `phases/phase-02/notes/block-07-notes.md`-এ লিখে (পরে Self-Check Guidance মিলিয়ে) সম্পন্ন করেছেন।
- [ ] তিনটি Mandatory Resource (**Brendan Gregg Active Benchmarking**, **`mraleph` দুটি নিবন্ধ**, এবং **Node.js 26 `node:perf_hooks` / `process.hrtime.bigint`** নির্ধারিত অংশ) সম্পন্ন করেছেন।
- [ ] `phases/phase-02/notes/block-07-notes.md`-এ **Pre-Lab Exercise (Spot the Broken Microbenchmark)** সম্পন্ন করেছেন।
- [ ] *(Phase 2 unlock হওয়ার পর)* **4-Stage Phase 2 Capstone Scaling Laboratory** সম্পন্ন করেছেন (`phases/phase-02/lab/`-এ Pure Node.js 26 Active LTS `storeBatchEngine.mjs`, `storeBatchEngine.test.mjs` passing `node --test`, এবং `benchmarkScaling.mjs`; `notes/block-07-notes.md`-এ Stage 1 predictions, Stage 2 & 4 real measured tables, trade-offs, এবং honest limitation defense)।
- [ ] *(Optional)* Notes/tabs বন্ধ রেখে **Optional Closed-Book Synthesis Assessment** Questions 1–5 লিখেছেন।
- [ ] **Phase 2 Checkpoint Handoff (ChatGPT Final Approval):** Blocks 1–7-এর learner-authored notes, verified Capstone Lab evidence, এবং learner-verified `BACKEND_KNOWLEDGE_INDEX.md` updates `phases/phase-02/checkpoint.md`-এ সংকলন করে **ChatGPT (Curriculum Owner & Instructor)**-এর কাছে formal mastery evaluation ও `approved` sign-off-এর জন্য জমা দিন—Phase 3 (*Operating Systems*) unlock করার আগে।

---

## Self-Check Guidance for Inline Pause & Checks 1–3 *(+ Pre-Lab Self-Check Criteria)*

> *`phases/phase-02/notes/block-07-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের guidance দেখুন। Pre-Lab Exercise-এর কোনো সম্পূর্ণ rewritten code, 4-Stage Capstone Scaling Lab-এর কোনো refactored solution, বা Optional Closed-Book Synthesis Assessment-এর কোনো answer key এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও ChatGPT review-এর জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* `N = 1,000` থেকে `16,000`-এ সময় প্রায় দ্বিগুণ হওয়া `Θ(N)` hypothesis-কে **corroborate (সমর্থন)** করে, কিন্তু **prove** করে না—কারণ asymptotic bound সংজ্ঞায়িত হয় অসীমের দিকে (`N → ∞`) সব valid input-এর জন্য। Finite কয়েকটি `N`-এ পরিমাপ কেবল নির্দিষ্ট machine, runtime, ও ওই নির্দিষ্ট test input-এর আচরণ দেখায়; `N = 100,000`-এর পরে কোনো `c * N^2` term বা কোনো adversarial input হঠাৎ dominate করে বসতে পারে।
  2. *শক্তিশালী উত্তরে থাকবে:*
     - *(ক)* দাবিটি অতিরঞ্জিত। **DCE** স্বয়ংক্রিয়ভাবে সব unused call মুছে দেয় না—এটি একটি **conditional optimization** যা ঘটে কেবল যখন optimizing engine প্রমাণ করতে পারে যে computation-টি side-effect-free এবং এর ফলাফলের কোনো observable প্রভাব নেই। প্রতিকার: ফলাফলকে একটি অতি হালকা accumulator/checksum-এ যোগ করে timed section-এর **বাইরে** print/return করা (যাতে sink নিজে measured কাজকে dominate না করে)।
     - *(খ)* সাজানো trials = `[4.1, 4.2, 4.2, 4.3, 22.7] ms` → **Median = `4.2 ms`**, **Min–Max Spread = `4.1 .. 22.7 ms`**! কেবল mean (`7.9 ms`) দেখালে `22.7 ms`-এর একটিমাত্র outlier (সম্ভাব্য GC বা OS jitter) সাধারণ steady-state গতির (`~4.2 ms`) তুলনায় প্রায় দ্বিগুণ ভুল ছবি দেয়—আর min–max spread না দেখালে সেই বড় spike-টির অস্তিত্বই গোপন থেকে যায়!
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* (ক) `M = 2,000` স্থির রেখে কেবল `N` দ্বিগুণ করলে `T(2N) / T(N) = (c * 2N * M) / (c * N * M) = 2` → তাত্ত্বিক **`R(N) ≈ 2×`**! (খ) `M = N` ধরে উভয়ই দ্বিগুণ করলে কাজ `N^2`-এর সমানুপাতিক হয় → `(2N)^2 / N^2 = 4` → তাত্ত্বিক **`R(N) ≈ 4×`**!
  2. *শক্তিশালী উত্তরে থাকবে:* `4_250_000 ns ÷ 1,000 = 4,250 µs` এবং `4_250_000 ns ÷ 1,000,000 = 4.25 ms`। আর একটিমাত্র ধাপে observed `R(N) = 2.8×` দেখেই `Θ(N^2)` দাবি করা ভুল—কারণ `N = 4,000 → 8,000` ধাপে dataset-টি CPU cache ছাড়িয়ে main RAM-এ চলে যেতে পারে (spatial locality / cache boundary), অথবা ওই ধাপে একটি GC pause বা OS scheduling jitter ঘটে থাকতে পারে; তাই একাধিক `N`, একাধিক trial-এর median/spread, এবং static code analysis একসাথে দেখতে হয়।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* অন্তত তিনটি unverified dimension: (১) **PostgreSQL / disk I/O**, (২) **network RTT ও downstream service calls**, (৩) **multi-request concurrency / event-loop contention** (এছাড়াও long-term production GC pressure এবং real payload distribution / p95 tail)। সৎ পুনর্লিখন: *"আমার laptop-এ Node.js 26-এ synthetic `N = 100,000` input-এর জন্য in-memory enrichment-এর median সময় `45 ms` পাওয়া গেছে, যা পূর্বের তুলনায় বড় algorithmic উন্নতি দেখায়; তবে এটি PostgreSQL I/O, network RTT, concurrent traffic, বা production `150 ms` p95 SLA প্রমাণ করে না।"*
  2. *শক্তিশালী উত্তরে থাকবে:* আগে থেকেই falsification condition ঠিক না রাখলে মানুষ পাওয়া যেকোনো সংখ্যাকে নিজের পছন্দমতো ব্যাখ্যা করে নেয় (confirmation bias / passive benchmarking)। আর independent correctness check না থাকলে এমন একটি "দ্রুত" refactor ভুলক্রমে গ্রহণ হয়ে যেতে পারে যা আসলে ভুল উত্তর, missing item, বা ভিন্ন sort order দিচ্ছে—কারণ ভুল উত্তর দ্রুত দেওয়া কোনো performance optimization নয়!
- **Pre-Lab Exercise Self-Check Criteria *(No Rewritten Code Provided)*:**
  - আপনার diagnosis-এ চারটি flaw-ই স্পষ্টভাবে থাকা উচিত: (1) `Date.now()`-এর `1 ms` resolution ও non-monotonic wall-clock ঝুঁকি + single run, (2) unused pure return value-র conditional DCE risk **এবং** Benchmark 1-এ hot-loop `console.log()` I/O distortion, (3) single tiny `N = 50`, already-sorted/unrepresentative input ও untimed warmup-এর অনুপস্থিতি, এবং (4) Benchmark 2-এর timed loop-এর ভেতর প্রতিবার `Array.from(...)` allocation মিশিয়ে asymmetric তুলনা করা।
  - আপনার rewrite-এ `node:perf_hooks` (`performance.now()` বা `process.hrtime.bigint()`), pre-generated multi-`N` fixtures outside timed sections, untimed warmup, multiple trials (median + min/max spread), lightweight observable sink, এবং independent `node:assert/strict` correctness check থাকতে হবে।
