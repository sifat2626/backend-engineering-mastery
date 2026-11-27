# Phase 2, Block 2 — Analyzing Loops, Recursion, and Recurrence Reasoning

## Study Time Guide

| Activity | Assigned Scope | Estimated Time |
| :--- | :--- | :---: |
| **Guided Lesson + Pause & Checks 1–3** *(main priority)* | পুরো guided lesson মনোযোগ দিয়ে পড়া এবং `phases/phase-02/notes/block-02-notes.md`-এ Pause & Checks 1–3 লেখা | **~50–65 min** *(flexible — limit নয়)* |
| **Mandatory 1:** Princeton — *Algorithms, 4th ed.* Booksite (`1.4`) | কেবল *Mathematical models* (tilde approximations, order-of-growth) এবং *Designing faster algorithms* (`3-Sum` `N^3` → `N^2 log N`) *(Memory usage ও Observations Block 5-এর জন্য deferred)* | ~25–30 min |
| **Mandatory 2:** MIT OCW 6.006 (Spring 2020) — *Recitation 1 & Lecture 3 Excerpt* | *Recitation 1 Notes* (asymptotic ও loop-analysis উদাহরণ) + *Lecture 3 Notes*-এর **কেবল** Merge Sort recursion-tree ও recurrence-solving অংশ (~2 pages) | ~25–30 min |
| **Exercise 1 (Mandatory):** Multi-Variable & Hidden-Cost Code Audit | ৬টি realistic Node.js backend snippet-এর exact time, auxiliary space, dominating line, ও `[...acc, row]`-এর three-way memory বিশ্লেষণ | ~25–30 min |
| **Exercise 2 (Mandatory):** Recursion Tree & Stack-Space Analysis | ৩টি recursive backend utility-র base case, recursion tree, total time, max call-stack space ও failure input বিশ্লেষণ | ~25–30 min |
| **Total (Required Core — without Optional Assessment)** | Guided Lesson + ২টি Mandatory Reading + ২টি Mandatory Exercise | **~150–185 min (~2.5–3.1 h)** |
| **Optional Closed-Book Assessment** *(optional)* | Notes/tabs বন্ধ রেখে ৫টি synthesis প্রশ্নের উত্তর লেখা | ~20–25 min *(optional)* |
| **Total (with Optional Assessment)** | Required Core + Optional Closed-Book Assessment | **~170–210 min (~2.8–3.5 h)** |

> **Planning Note:** Approved Phase 2 Research Dossier-এ Block 2-এর target allocation হলো **3 hours (180 minutes)**—যা একটি flexible planning reference, কোনো strict deadline নয়। দ্রুত শেষ করার চেয়ে প্রতিটি derivation নিজে হাতে করে বোঝাই main priority; বিশেষ করে **Guided Lesson**-এর জন্য প্রয়োজনে আনুমানিক সময়ের চেয়ে বেশি সময় লাগা একদম স্বাভাবিক। প্রতিটি reading estimate কেবল নির্ধারিত sections-এর জন্য, পুরো lecture বা বই পড়ার জন্য নয়। **Optional Closed-Book Assessment** সম্পূর্ণ optional এবং Block 3 শুরু করার জন্য কোনো mandatory gate নয়।

---

Block 1-এ আপনি complexity-র ভাষা শিখেছেন—Word-RAM model, `O` / `Ω` / `Θ`, logarithm-এর halving intuition, এবং input space বনাম auxiliary space। কিন্তু Block 1-এ প্রতিবারই expression-টা (`2n + 10`, `3n^2 + 50n + 120`, `5,000 * N`) আমাদের হাতে তৈরি করা ছিল।

বাস্তব backend engineering-এ কেউ আপনাকে সুন্দর equation বানিয়ে হাতে দেবে না—আপনার সামনে থাকবে **আসল code**।

আমাদের online store-এর একটা সোমবার সকালের PR review দিয়ে শুরু করি। Payment webhook-গুলো যেন দুবার process না হয়, সেজন্য একজন developer মাত্র ৭ লাইনের একটি helper function পাঠিয়েছেন:

```javascript
function deduplicateWebhookEvents(incomingEvents) {
  const seenIds = [];
  const uniqueEvents = [];

  for (const event of incomingEvents) {
    if (!seenIds.includes(event.id)) {
      seenIds.push(event.id);
      uniqueEvents.push(event);
    }
  }
  return uniqueEvents;
}
```

PR-টা দেখে খুব clean, simple, আর innocent মনে হচ্ছে। চোখে দেখা যাচ্ছে **একটাই মাত্র `for` loop**! অনেক developer এক নজর তাকিয়েই বলে বসেন: *"একটা `for` loop আছে, তার মানে এটা `Θ(N)`—merge করে দাও!"*

কিন্তু `50,000` unique webhook events আসতেই worker process-টা কয়েক সেকেন্ডের জন্য জমে গেল! কেন?

ভেতরের লাইনটা আবার পড়ুন: `if (!seenIds.includes(event.id))`।
- ১ম event-এ `seenIds`-এ `0`টা ID আছে → `.includes()` `0`টা comparison করে।
- ২য় event-এ `seenIds`-এ `1`টা ID আছে → `.includes()` `1`টা comparison করে।
- ৩য় event-এ `2`টা, ৪র্থ event-এ `3`টা... এভাবে `N`-তম unique event-এ `seenIds.includes()` পুরো `N - 1`টা ID এক-এক করে scan করে!
- মোট comparison: `0 + 1 + 2 + ... + (N - 1) = N(N - 1) / 2 = Θ(N^2)`!

একটাই visible `for` loop—অথচ গোপন কাজটা ছিল **`Θ(N^2)`**!

এই একটি অভিজ্ঞতা থেকেই Block 2-এর মূল নীতিটা পরিষ্কার হয়ে যায়: **কখনোই `for` keyword-এর সংখ্যা বা code-এর indentation গুনে complexity অনুমান করবেন না।** বরং যেকোনো backend function দেখলে একটি নির্দিষ্ট, পুনরাবৃত্তিযোগ্য **৬-ধাপের Analysis Method** অনুসরণ করুন:

1. **Independent input variables define করুন:** `N`, `M`, `L`—কোন variable কী নির্দেশ করছে (orders, promo rules, string length) তা আলাদা করে লিখুন।
2. **Basic operation চিহ্নিত করুন:** কোন মৌলিক কাজটি (comparison, arithmetic update, element copy) সবচেয়ে বেশিবার ঘটছে?
3. **Loop বা recursion-এর iteration count গুনুন:** প্রতিটা ধাপ ঠিক কতবার চলছে তা algebra দিয়ে লিখুন।
4. **ভেতরের helper / built-in calls-এর cost যোগ করুন:** `.includes()`, `.shift()`, `.slice()`, বা অন্য কোনো function call কখনোই অন্ধভাবে `O(1)` ধরবেন না।
5. **Tight asymptotic bound বের করুন:** Dominant term আলাদা করে `Θ` (বা যেখানে case-dependent, সেখানে `O`/`Ω`) নির্ণয় করুন।
6. **Auxiliary space আলাদাভাবে হিসাব করুন:** নতুন arrays, objects, temporary copies, বা call-stack depth কতটুকু বাড়ছে?

এখন এই ৬-ধাপের method-টিকে আমরা ধাপে ধাপে প্রতিটা real backend pattern-এর ওপর প্রয়োগ করব—প্রথমে simple ও nested loops, তারপর hidden JavaScript built-in costs, আর শেষে recursion ও recursion trees।

---

## Sequential Code, Independent Nested Loops, এবং Multi-Variable Analysis

শুরু করি একদম পরিষ্কার দুটো নিয়ম দিয়ে—যার একটাতে costs **যোগ** হয়, আরেকটাতে costs **গুণ** হয়।

### ১. Sequential Loops (পরপর চলা কাজ → যোগ হয়)

ধরুন আমাদের checkout summary endpoint-এ দুটি independent list আসে: `orders` (সংখ্যা `N`) এবং `activePromos` (সংখ্যা `M`)। Function-টি প্রথমে সব order-এর মোট টাকা যোগ করে, তারপর সব active promo-র মোট budget যোগ করে:

```javascript
function computeCheckoutSummary(orders, activePromos) {
  let totalOrderCents = 0;
  for (let i = 0; i < orders.length; i++) {
    totalOrderCents += orders[i].amountCents;
  }

  let totalPromoBudgetCents = 0;
  for (let j = 0; j < activePromos.length; j++) {
    totalPromoBudgetCents += activePromos[j].budgetCents;
  }

  return { totalOrderCents, totalPromoBudgetCents };
}
```

- **Input variables:** `N = orders.length`, `M = activePromos.length`। এরা দুটি সম্পূর্ণ **independent** সংখ্যা—store-এ `1,000,000` orders থাকতে পারে অথচ `activePromos` মাত্র `12`টি!
- প্রথম loop চলে `N` বার (`Θ(N)`); দ্বিতীয় loop চলে `M` বার (`Θ(M)`)।
- দুটি কাজ **পরপর (sequentially)** চলছে—তাই মোট সময় যোগ হবে: **`Θ(N + M)`**!
- *(আর যদি দুটো sequential loop-ই একই `orders` array-র ওপর `N` বার করে চলত, তবে `Θ(N) + Θ(N) = Θ(2N) = Θ(N)` হতো।)*
- **Auxiliary space:** শুধু চারটি local variable (`totalOrderCents`, `i`, `totalPromoBudgetCents`, `j`) এবং ছোট ২-field return object → **`Θ(1)`**।

### ২. Independent Nested Loops (ভেতরে ভেতরে চলা কাজ → গুণ হয়)

এবার ধরুন আমাদের store-এ `N`টি warehouse এবং `M`টি product SKU আছে। প্রতিটি warehouse-এ প্রতিটি SKU-র stock matrix যাচাই করে কতগুলো slot-এ stock `0` আছে তা গুনতে হবে:

```javascript
function countOutOfStockSlots(warehouses, skus) {
  let emptySlots = 0;
  for (let w = 0; w < warehouses.length; w++) {
    for (let s = 0; s < skus.length; s++) {
      if (warehouses[w].stockBySkuIndex[s] === 0) {
        emptySlots += 1;
      }
    }
  }
  return emptySlots;
}
```

- বাইরের loop-এর **প্রতিটি** `w`-এর জন্য ভেতরের `s` loop পুরো `M` বার চলে (বাইরের `w`-এর মানের ওপর `s`-এর সীমা নির্ভর করছে না—তাই এগুলো **independent nested loops**)।
- মোট comparison = `M + M + ... + M` (`N` বার) = **`N * M`** → **`Θ(N * M)`**!
- **Multi-Variable Rule (অত্যন্ত গুরুত্বপূর্ণ!):** কখনোই `N` আর `M`-কে জোর করে এক `N` বানিয়ে `Θ(N^2)` লিখবেন না! কেন? কারণ ধরুন আপনার store-এ `N = 500,000` orders এবং `M = 5`টি promo rule:
  - `Θ(N * M)` মানে `500,000 * 5 = 2,500,000` ops (সম্পূর্ণ manageable!)।
  - `Θ(N^2)` হলে হতো `250,000,000,000` ops (সম্পূর্ণ অচল!)।
  কোন dimension আসলে বড় হচ্ছে—`N`, `M`, নাকি payload length `L`—তা আলাদা না রাখলে আপনি ভুল জায়গায় optimize করবেন।
- **Auxiliary space:** `emptySlots`, `w`, `s` → **`Θ(1)`**।

---

## Dependent Triangular Loops: যখন ভেতরের Loop বাইরের Variable-এর ওপর নির্ভর করে

সব nested loop-এ ভেতরের সীমা fixed `M` বা `N` থাকে না। অনেক সময় আমরা `N`টি item-এর মধ্যে **প্রতিটি unordered pair `(i, j)` একবার করে** check করতে চাই—যেমন `N`টি delivery batch-এর মধ্যে কোনো দুটি batch-এর সময়সীমা overlap করে কি না:

```javascript
function countOverlappingBatches(batches) {
  let overlaps = 0;
  for (let i = 0; i < batches.length; i++) {
    for (let j = i + 1; j < batches.length; j++) {
      if (batches[i].startMin < batches[j].endMin &&
          batches[j].startMin < batches[i].endMin) {
        overlaps += 1;
      }
    }
  }
  return overlaps;
}
```

এখানে ভেতরের loop শুরু হচ্ছে `j = i + 1` থেকে—অর্থাৎ বাইরের `i` যত বাড়ছে, ভেতরের loop তত ছোট হচ্ছে! তাই হঠাৎ করে দেখে মনে হতে পারে *"ভেতরের loop তো পুরো `N` বার চলছে না, তাহলে কি এটা `Θ(N^2)`-এর চেয়ে কম?"*

আসুন অনুমান না করে **হাতে গুনে derive** করি (`N = batches.length`):
- `i = 0` হলে `j` চলে `1, 2, ..., N - 1` → মোট **`N - 1`** বার
- `i = 1` হলে `j` চলে `2, 3, ..., N - 1` → মোট **`N - 2`** বার
- `i = 2` হলে → মোট **`N - 3`** বার
- ...
- `i = N - 2` হলে `j` চলে শুধু `N - 1` → মোট **`1`** বার
- `i = N - 1` হলে → **`0`** বার

মোট inner-body executions:
```text
Total = (N - 1) + (N - 2) + (N - 3) + ... + 2 + 1 + 0
```

এই arithmetic series sum-টা মুখস্থ না করে Gauss-এর সহজ pairing trick দিয়ে মনে রাখুন: যোগফল `S`-কে সোজা আর উল্টো করে নিচে নিচে যোগ করুন—
```text
  S =   0   +   1   +   2   + ... + (N - 2) + (N - 1)
+ S = (N-1) + (N-2) + (N-3) + ... +    1    +    0
-------------------------------------------------------
 2S = (N-1) + (N-1) + (N-1) + ... + (N - 1) + (N - 1)   [মোট Nটি জোড়া!]
```
অর্থাৎ `2S = N * (N - 1)` →
```text
S = N * (N - 1) / 2 = (1/2) * N^2 - (1/2) * N
```

Block 1-এর নিয়মে constant factor `1/2` আর lower-order term `- (1/2) * N` বাদ পড়লে tight bound হয় **`Θ(N^2)`**!
অর্থাৎ অর্ধেক কাজ কমিয়ে অর্ধেক ত্রিভুজ (**triangular loop**) কাটলেও asymptotic growth class বদলায় না—`N` ১০ গুণ বাড়লে কাজ এখনো `100×` বাড়ে! (Auxiliary space: `Θ(1)`।)

---

## Multiplicative ও Halving Loops: কেন Loop-এর সংখ্যা `Θ(log N)` হয়

এখন পর্যন্ত প্রতিটি loop variable বেড়েছে `+1` করে (`i++`)। কিন্তু কোনো loop variable যদি প্রতি ধাপে **গুণ (`*= 2`)** বা **ভাগ (`/ 2`)** হয়, তখন iteration count আমূল বদলে যায়।

ধরুন একটি backend retry-window estimator-এ আপনি exponentially doubling batch sizes পরীক্ষা করছেন (`1, 2, 4, 8, ...` যতক্ষণ `batchSize < maxLimit`), অথবা `M`টি sorted price-tier-এ একটি target amount খুঁজতে search range প্রতি ধাপে অর্ধেক করছেন:

```javascript
// Pattern A: Multiplicative loop (step *= 2)
function countDoublingBuckets(maxLimit) {
  let buckets = 0;
  for (let size = 1; size < maxLimit; size *= 2) {
    buckets += 1;
  }
  return buckets;
}

// Pattern B: Halving loop inside an outer loop -> O(N log M)
function countOrdersAboveMidpoints(orders, sortedPriceTiers) {
  let steps = 0;
  for (let i = 0; i < orders.length; i++) {
    let span = sortedPriceTiers.length;
    while (span > 1) {
      span = Math.floor(span / 2);
      steps += 1;
    }
  }
  return steps;
}
```

ধাপে ধাপে গুনুন:
- **Pattern A (`N = maxLimit`):**
  - Iteration `0`-এ `size = 1 = 2^0`
  - Iteration `1`-এ `size = 2 = 2^1`
  - Iteration `2`-এ `size = 4 = 2^2`
  - ...
  - `k`-তম iteration-এর পর `size = 2^k`। Loop থামে যখন `2^k >= N`, অর্থাৎ **`k >= log2(N)`**!
  - তাই মোট iterations = `ceil(log2(N))` (for `N >= 1`) → **`Θ(log N)`** time, **`Θ(1)`** auxiliary space!
- **Pattern B (`N = orders.length`, `M = sortedPriceTiers.length`):**
  - বাইরের loop চলে `N` বার।
  - প্রতিবার ভেতরের `while` loop-এ `span` মান `M → M/2 → M/4 → ... → 1` হয়ে থামে `floor(log2(M))` ধাপে (`Θ(log M)`)।
  - দুটি independent loop গুণ হয়ে মোট time: **`Θ(N log M)`**, auxiliary space: **`Θ(1)`**!

*(Note on early termination: কোনো search loop-এ যদি ভেতরে `if (found) return mid;` থাকে, তবে তা নির্দিষ্ট input-এ `1` ধাপেই থেমে যেতে পারে আর অন্য input-এ পুরো `log2(M)` বা `M` ধাপ নিতে পারে। তাই early-exit loop-এর full-traversal upper bound হয় `O(log M)` বা `O(M)`—আর কোন input-এ best/average/worst case ঘটে তার বিস্তারিত কাঠামো আমরা Block 3-এ শিখব।)*

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-02-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. নিচের backend snippet-টিতে দুটি independent input আছে: `accounts` (সংখ্যা `N`) এবং `maxWindowSize` (একটি positive integer `M >= 1`)। ধাপে ধাপে derivation দেখিয়ে এর tight time complexity (exact `N, M`-এ) ও auxiliary space complexity বের করুন:
>    ```javascript
>    function auditSyncWindows(accounts, maxWindowSize) {
>      let activeAccounts = 0;
>      for (let i = 0; i < accounts.length; i++) {
>        if (accounts[i].active) activeAccounts += 1;
>      }
>      let windowSteps = 0;
>      for (let size = 1; size < maxWindowSize; size *= 2) {
>        windowSteps += 1;
>      }
>      return { activeAccounts, windowSteps };
>    }
>    ```
> 2. নিচের dependent loop-টিতে `N = scores.length` হলে `total += 1` লাইনটি **ঠিক কতবার** execute হবে? যোগফল (`sum`) লিখে algebra দিয়ে দেখান কেন এর tight bound `Θ(N^2)`:
>    ```javascript
>    function countEarlierComparisons(scores) {
>      let total = 0;
>      for (let i = 0; i < scores.length; i++) {
>        for (let j = 0; j < i; j++) {
>          total += 1;
>        }
>      }
>      return total;
>    }
>    ```

এবার ফিরে আসুন আমাদের opening webhook bug-এ। আপনি নিজের লেখা `for`/`while` loop গুনতে শিখলেন—কিন্তু modern JavaScript/Node.js backend code-এ সবচেয়ে বেশি `O(N^2)` bug তৈরি হয় নিজের লেখা nested `for` loop থেকে নয়, বরং **একটা visible loop-এর ভেতর ডাকা লুকানো built-in method থেকে!**

---

## Hidden JavaScript Operation Costs — এবং Cumulative Allocation বনাম Peak Live Memory

JavaScript-এ `.includes()`, `.indexOf()`, `.shift()`, `.splice()`, string `+`/`.slice()`, এবং `[...acc, item]` লিখতে মাত্র এক শব্দ লাগে—তাই অসতর্ক চোখ এগুলোকে `O(1)` ধরে নেয়। কিন্তু কোনো built-in method-এর cost মুখস্থ folklore দিয়ে নয়, **ওটা ভেতরে কোন মৌলিক কাজ করছে** তা ভেবে বের করতে হয়:

| Built-in / Pattern | ভেতরে কী মৌলিক কাজ হয় (Abstract Cost Model) | `N`-iteration Loop-এর ভেতর ডাকলে কী ঘটে | জরুরি Qualification (Engine Reality) |
| :--- | :--- | :--- | :--- |
| **`arr.includes(x)` / `arr.indexOf(x)`** | Unsorted array-তে শুরু থেকে এক-এক করে element compare করতে হয় → length `M` হলে worst case-এ **`O(M)`**। | `N`টি item-এর loop-এ length `M` array scan করলে **`O(N * M)`**; আর growing array (`0..N-1`)-এ ডাকলে **`O(N^2)`**! | Element আগে পাওয়া গেলে early exit হয়, কিন্তু না পাওয়া গেলে পুরো array scan করতে হয়। |
| **`arr.shift()` / `arr.splice(idx, ...)`** | Contiguous array-র শুরু থেকে প্রথম element সরালে বা মাঝখানে insert/delete করলে পরের সব element-কে এক ঘর করে সরাতে (**re-index / shift**) হতে পারে → বাকি `k` element-এর জন্য **`O(k)`**। | `N`-length queue `while (q.length) q.shift()` দিয়ে খালি করলে abstract contiguous-array model-এ `(N-1) + ... + 1 = Θ(N^2)`! | সব JS engine-এ প্রতিটি `shift()`-এর internal cost হুবহু অভিন্ন বলে অন্ধ দাবি করা যাবে না, তবে standard contiguous-array model-এ front/middle shift-কে `O(k)` ঝুঁকি হিসেবেই ধরতে হয়। |
| **String `+` / `.slice()`** | `L` length-এর immutable string copy বা process করতে abstract copying model-এ **`Θ(L)`**। | `N`টি `L`-length string process/copy করলে **`O(N * L)`**; loop-এ string-এর শেষে বারবার যোগ করে `1+2+...+N` character copy করলে abstract model-এ **`Θ(N^2)`**। | সব JS engine প্রতিটি concatenation-এ সাথে সাথে eager copy করে—এমন দাবি ভুল (engines internal rope/sliced string ব্যবহার করতে পারে; V8 internals Phase 5-এ)। তবু `L` length-এর ওপর নির্ভরশীল cost model ধরে `O(N * L)` হিসাব করাই নিরাপদ। |
| **`[...acc, item]` inside `.reduce()`** | প্রতিটা iteration-এ spread operator `...acc` আগের পুরো accumulated array-র সব element কপি করে একটি নতুন array বানায়। | Iteration `k`-এ `k`টি write → `1 + 2 + ... + N = N(N + 1)/2 = Θ(N^2)` **cumulative copying & time**! | **নিচের Three-Way Memory Distinction ভালো করে বুঝুন—এটি `Θ(N^2)` peak live memory নয়!** |

### অত্যন্ত জরুরি পার্থক্য: Cumulative Allocation বনাম Peak Live Auxiliary Memory বনাম Allocation Churn

`[...acc, item]` pattern-টা আজকের দিনে React/Node.js codebase এবং AI-generated code-এ প্রচুর দেখা যায়, কারণ immutable style দেখতে পরিচ্ছন্ন লাগে:

```javascript
function collectOrderIds(orders) {
  return orders.reduce((acc, order) => [...acc, order.id], []);
}
```

অনেক developer এখানে দুটো ভুলের একটা করে বসেন: কেউ বলেন *"একটাই `reduce` চলছে, তাই `O(N)`!"* (ভুল—time `Θ(N^2)`!), আবার কেউ বলেন *"যেহেতু `Θ(N^2)` element allocate হচ্ছে, তাই এর peak live memory-ও `Θ(N^2)`!"* (**এটাও অসতর্ক ও ভুল দাবি!**)

আসুন ঠিক কী ঘটছে তা তিন ভাগে ছুরি দিয়ে আলাদা করি (`N = orders.length`):

1. **Total Cumulative Allocation / Copying (`Θ(N^2)`):**
   - Step 1: নতুন 1-element array তৈরি (`1` write)
   - Step 2: পুরোনো `1` element copy + `1` নতুন (`2` writes)
   - Step 3: পুরোনো `2` elements copy + `1` নতুন (`3` writes)
   - ...
   - Step `N`: পুরোনো `N - 1` elements copy + `1` নতুন (`N` writes)
   - পুরো function-এর জীবদ্দশায় মোট allocated ও copied elements = `1 + 2 + ... + N = N(N + 1) / 2 = Θ(N^2)`। আর যেহেতু প্রতিটি element copy করতে সময় লাগে, তাই **total time complexity-ও `Θ(N^2)`**!
2. **Live Reachable Auxiliary Memory (`Θ(N)`):**
   - এবার যেকোনো একটি নির্দিষ্ট মুহূর্ত `k`-এ (ধরুন `k = 5,000`) ছবিটা থামান! ওই মুহূর্তে program-এর হাতে reference আছে কেবল **আগের `acc`** (length `k - 1`) আর **নতুন তৈরি হওয়া array** (length `k`)-এর। Step `1, 2, ..., k - 2`-এ তৈরি হওয়া আগের সব array-র আর কোনো reference বেঁচে নেই—সেগুলো unreachable (garbage)!
   - তাই যেকোনো মুহূর্তে **live reachable auxiliary data** সর্বোচ্চ `(N - 1) + N = 2N - 1 = Θ(N)`—`Θ(N^2)` **নয়**!
3. **Actual Peak Live Memory ও Allocation Churn (GC Pressure):**
   - তবু বাস্তব Node.js process-এ এই `Θ(N^2)` অস্থায়ী array তৈরি ও ছুড়ে ফেলাকে বলে **Allocation Churn**।
   - Garbage collector ঠিক কোন মুহূর্তে unreachable array-গুলো পরিষ্কার করবে এবং temporary object-গুলোর lifetime কতক্ষণ, তার ওপর নির্ভর করে **actual transient peak RAM** `Θ(N)`-এর চেয়ে সাময়িকভাবে অনেক ওপরে উঠতে পারে এবং বারবার GC pause হয়ে latency বাড়িয়ে দেয়! *(GC-র internal generation mechanics আমরা Phase 5-এ শিখব।)*

সংক্ষেপে মনে রাখবেন: **Quadratic cumulative allocation/copying (`Θ(N^2)`) ≠ Quadratic live reachable auxiliary memory (`Θ(N)`)**—কিন্তু এটি `Θ(N^2)` time এবং মারাত্মক **allocation churn / GC pressure** দুটোই তৈরি করে!

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-02-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> নিচের Node.js function-টি `N`টি unique promo tag নেয় (যেখানে সব tag-ই ভিন্ন এবং ছোট fixed-length string):
> ```javascript
> function normalizeUniqueTags(tags) {
>   let result = [];
>   for (let i = 0; i < tags.length; i++) {
>     if (result.indexOf(tags[i]) === -1) {
>       result = [...result, tags[i]];
>     }
>   }
>   return result;
> }
> ```
> 1. চোখে মাত্র **একটি `for` loop** থাকা সত্ত্বেও `N`টি unique tag-এর জন্য এই function-টির total time complexity কেন `Θ(N^2)`? কোন **দুটি** লাইন এই quadratic কাজের জন্য দায়ী, তা exact sum দেখিয়ে ব্যাখ্যা করুন।
> 2. এই function-এর **total cumulative allocation/copying**, **live reachable auxiliary memory**, এবং **actual peak live memory / allocation churn**-কে স্পষ্টভাবে আলাদা করে লিখুন।

এতক্ষণ আমরা কেবল iterative code (`for`, `while`, `.reduce`) analyze করলাম। কিন্তু backend-এ সব problem flat loop দিয়ে লেখা যায় না—যেমন nested product categories (`Electronics → Phones → Accessories → Cables`), nested JSON payload validation, বা divide-and-conquer range splitting। **যখন কোনো function নিজেকেই call করে (recursion), তখন তার total work আর memory কীভাবে গুনব?**

---

## Recursion, Call-Stack Auxiliary Space, এবং Stack-Overflow ঝুঁকি

একটি recursive function analyze করতে গেলে সবার আগে দুটি মৌলিক জিনিস খুঁজতে হয়:
1. **Base Case:** কোন শর্তে function-টি আর নিজেকে call না করে সরাসরি return করে? *(Base case না থাকলে বা না পৌঁছালে infinite recursion ঘটে।)*
2. **Progress Measure:** প্রতিটি recursive call কীভাবে input-কে strictly ছোট করে (যেমন `n → n - 1`, `n → n / 2`, বা parent node থেকে child node) base case-এর দিকে নিয়ে যায়?

এবার একটি সূক্ষ্ম কিন্তু অত্যাবশ্যকীয় concept: **Call-Stack Auxiliary Space**।

যখন `f(A)` call করে `f(B)`-কে, আর `f(B)` call করে `f(C)`-কে, তখন `f(A)` এবং `f(B)` এখনো শেষ হয়নি—তাদের local variables এবং *"ফিরে এসে কোন লাইন থেকে চলব"* (return address) memory-র **call stack**-এ একটার ওপর আরেকটা **stack frame** হিসেবে জমা থাকে। `f(C)` return করলেই কেবল `f(C)`-র frame pop হয়ে মুছে যায়!

এখান থেকেই একটি সোনালি নিয়ম জন্ম নেয়:
- **Total Time** = পুরো execution জুড়ে হওয়া **সব recursive call-এর কাজের যোগফল**।
- **Call-Stack Auxiliary Space** = **`O(maximum recursion depth)`**—অর্থাৎ কোনো এক মুহূর্তে call stack-এ একসাথে সর্বোচ্চ কতগুলো active frame জমে থাকে!

**বাস্তব Backend ঝুঁকি (Stack Overflow):**
ধরুন আপনি একটি online store-এর nested category tree বা client-এর পাঠানো nested JSON payload recursively traverse করছেন। সাধারণত category tree `5` বা `6` level গভীর—তাই local test-এ কোনো সমস্যা হয় না। কিন্তু যদি data-তে একটি দীর্ঘ skewed chain থাকে (`50,000` level গভীর category বা malicious deeply nested JSON `{{{{...}}}}`), তবে `max recursion depth = 50,000` হয়ে Node.js-এর finite call stack উপচে পড়ে এবং পুরো request `RangeError: Maximum call stack size exceeded` দিয়ে crash করে!

---

## Recursion Trees: চারটি মূল Recurrence Shape হাতে-কলমে সমাধান

একটি recursive function-এর মোট কাজ বের করার জন্য আমরা তার cost-কে একটি **Recurrence Relation** হিসেবে লিখি—অর্থাৎ size `n`-এর কাজের সময় `T(n)`-কে তার ছোট subproblem-গুলোর সময় (`T(n - 1)` বা `T(n/2)`) দিয়ে প্রকাশ করি।

কোনো জটিল theorem মুখস্থ না করে, একটি **Recursion Tree** এঁকে যেকোনো canonical recurrence ধাপে ধাপে সমাধান করা যায়:
1. **Base case** ও **assumptions** স্পষ্ট লিখুন (যেমন `T(1) = c` এবং সরলতার জন্য `n` হলো `2`-এর ঘাত `2^k` যখন `n/2` ভাগ হয়)।
2. প্রতিটা level-এ **node-এর সংখ্যা** এবং **প্রতিটি node-এর local work** (recursive call বাদে) বের করুন।
3. **প্রতি level-এর total work** = `(nodes at level) × (work per node)`।
4. **মোট levels** গুনে সব level-এর work যোগ করুন → **Total Time**।
5. Root থেকে deepest leaf পর্যন্ত দীর্ঘতম path-এর দৈর্ঘ্য দেখুন → **Maximum Call-Stack Depth**।

আসুন backend engineering-এ সবচেয়ে বেশি দেখা **চারটি canonical recurrence shape** সম্পূর্ণ worked code ও recursion tree সহ দেখি!

---

### Shape 1: Linear Reduction — `T(n) = T(n - 1) + Θ(1)`

**Worked Code (`N = orders.length`, `n = orders.length - idx`):**
```javascript
// Base case: idx >= orders.length (n = 0) -> O(1)
// Progress measure: remaining elements n decreases by 1 each call
function countPendingOrders(orders, idx = 0) {
  if (idx >= orders.length) return 0;
  const isPending = orders[idx].status === "PENDING" ? 1 : 0;
  return isPending + countPendingOrders(orders, idx + 1);
}
```

- **Recurrence:** `T(n) = T(n - 1) + c` (for `n >= 1`), base case `T(0) = c0`।
- **Recursion Tree (branching factor = `1`, একটি সরল চেইন):**
  ```text
  Level 0:   [n]      -> local work: c
              |
  Level 1:  [n-1]     -> local work: c
              |
  Level 2:  [n-2]     -> local work: c
              .
              .
  Level n:   [0]      -> base work: c0
  ```
- **Levels:** `n + 1` levels (`Θ(n)`)। **Work per level:** `c = Θ(1)`।
- **Total Time:** `n * c + c0 = Θ(n)`।
- **Max Call-Stack Depth (Auxiliary Space):** **`Θ(n)`**! *(খেয়াল করুন: একটা সাধারণ `for` loop-এ একই কাজ করতে time `Θ(n)` হলেও auxiliary space মাত্র `Θ(1)`—কিন্তু recursive chain-এ `n = 100,000` হলেই stack overflow হবে!)*

---

### Shape 2: Halving Search — `T(n) = T(n / 2) + Θ(1)`

**Worked Code (`n = high - low + 1`, sorted order IDs):**
```javascript
// Base case: low > high (empty range, n = 0) or match found -> O(1)
// Progress measure: search range n is at least halved each recursive call
function binarySearchOrder(sortedIds, targetId, low = 0, high = sortedIds.length - 1) {
  if (low > high) return -1;
  const mid = Math.floor((low + high) / 2);
  if (sortedIds[mid] === targetId) return mid;
  if (sortedIds[mid] < targetId) {
    return binarySearchOrder(sortedIds, targetId, mid + 1, high);
  }
  return binarySearchOrder(sortedIds, targetId, low, mid - 1);
}
```

- **Recurrence (worst case — target না পাওয়া পর্যন্ত):** `T(n) = T(floor(n / 2)) + c` (সহজ বিশ্লেষণে `T(n) = T(n/2) + c`), base case `T(0) = c0` (ফলে `T(1) = c + c0 = Θ(1)`)।
- **Recursion Tree (branching factor = `1`, প্রতি ধাপে অর্ধেক):**
  ```text
  Level 0:     [n]      -> local work: c
                |
  Level 1:    [n/2]     -> local work: c
                |
  Level 2:    [n/4]     -> local work: c
                .
                .
  Level k:     [1]      -> base work: c   (যেখানে n / 2^k = 1 => k = log2(n))
  ```
- **Levels:** `log2(n) + 1 = Θ(log n)`। **Work per level:** `c = Θ(1)`।
- **Total Time:** `c * (log2(n) + 1) = Θ(log n)`।
- **Max Call-Stack Depth (Auxiliary Space):** **`Θ(log n)`** (যেখানে iterative `while`-loop binary search-এ auxiliary space মাত্র `Θ(1)`!)।

---

### Shape 3: Balanced Split-and-Merge — `T(n) = 2T(n / 2) + Θ(n)`

**Worked Code (`n = right - left + 1`, maximum contiguous subarray sum — divide-and-conquer split + linear crossing-segment combine pass):**
```javascript
// Base case: empty range (left > right) or single element (left === right) -> O(1)
// Progress measure: range size n splits into two halves (~n/2 each)
function maxSubarraySum(netChanges, left = 0, right = netChanges.length - 1) {
  if (left > right) return 0;
  if (left === right) return netChanges[left];

  const mid = Math.floor((left + right) / 2);
  const leftBest = maxSubarraySum(netChanges, left, mid);
  const rightBest = maxSubarraySum(netChanges, mid + 1, right);

  // Linear Θ(n) combine pass: scan left half for max suffix and right half for max prefix
  let leftSuffixSum = 0;
  let bestLeftSuffix = -Infinity;
  for (let i = mid; i >= left; i--) {
    leftSuffixSum += netChanges[i];
    if (leftSuffixSum > bestLeftSuffix) {
      bestLeftSuffix = leftSuffixSum;
    }
  }

  let rightPrefixSum = 0;
  let bestRightPrefix = -Infinity;
  for (let j = mid + 1; j <= right; j++) {
    rightPrefixSum += netChanges[j];
    if (rightPrefixSum > bestRightPrefix) {
      bestRightPrefix = rightPrefixSum;
    }
  }

  const crossingBest = bestLeftSuffix + bestRightPrefix;
  return Math.max(leftBest, rightBest, crossingBest);
}
```

- **Recurrence (ধরুন `n = 2^k`):** `T(n) = 2T(n/2) + c * n`, base case `T(1) = c` (কারণ combine ধাপে বাম অর্ধেকের maximum suffix-এ `mid - left + 1` ধাপ এবং ডান অর্ধেকের maximum prefix-এ `right - mid` ধাপ মিলিয়ে মোট ঠিক `n` ধাপে best crossing segment বের হয় → `Θ(n)`)।
- **Recursion Tree (branching factor = `2`):**
  ```text
  Level 0:                 [n]                 -> 1 node  × (c * n)   = c * n
                         /     \
  Level 1:          [n/2]       [n/2]          -> 2 nodes × (c * n/2) = c * n
                    /   \       /   \
  Level 2:       [n/4] [n/4] [n/4] [n/4]       -> 4 nodes × (c * n/4) = c * n
                   .     .     .     .
  Level log2(n): [1] [1] ...       ... [1]     -> n nodes × c         = c * n
  ```
- **চমৎকার symmetry দেখুন:** প্রতি level-এ node সংখ্যা দ্বিগুণ (`2^d`) হচ্ছে, কিন্তু প্রতিটি node-এর work ঠিক অর্ধেক (`c * n / 2^d`) হয়ে যাচ্ছে—ফলে **প্রতিটি level-এর total work হুবহু `c * n`**!
- **Levels:** `log2(n) + 1` levels।
- **Total Time:** `(log2(n) + 1) * (c * n) = c * n * log2(n) + c * n = Θ(n log n)`! *(Merge Sort-ও ঠিক এই একই recurrence অনুসরণ করে।)*
- **CRITICAL Stack-Depth Distinction:** Total work হলো `Θ(n log n)`, কিন্তু depth-first execution-এ আগে পুরো `leftBest` call শেষ হয়ে তার stack frames pop হয়ে যায়, *তারপরেই* `rightBest` call শুরু হয়! তাই যেকোনো মুহূর্তে call stack-এ একসাথে active থাকে সর্বোচ্চ একটি path-এর frames → **Max Call-Stack Depth = `Θ(log n)`** (কখনোই `Θ(n log n)` নয়!)। *(দ্রষ্টব্য: Merge Sort-এর মতো algorithm-এ যদি combine ধাপে আলাদা temporary array allocate করা হয়, তবে heap auxiliary space হয় `Θ(n)`, কিন্তু call-stack depth তবু `Θ(log n)`-ই থাকে।)*

---

### Shape 4: Exponential Branching — `T(n) = 2T(n - 1) + Θ(1)`

**Worked Code (`n = items.length - idx`, কোনো memoization ছাড়া সব subset check):**
```javascript
// Base case: idx === items.length (n = 0) -> O(1)
// Progress measure: remaining items n decreases by 1 in BOTH branches
function countAllSubsets(items, idx = 0) {
  if (idx === items.length) return 1;
  const withoutCurrent = countAllSubsets(items, idx + 1);
  const withCurrent = countAllSubsets(items, idx + 1);
  return withoutCurrent + withCurrent;
}
```

- **Recurrence:** `T(n) = 2T(n - 1) + c` (for `n >= 1`), base case `T(0) = c`।
- **Recursion Tree (branching factor = `2`, প্রতি ধাপে শুধু `-1` কমে!):**
  ```text
  Level 0:               [n]                  -> 2^0 = 1 node   × c
                       /     \
  Level 1:          [n-1]   [n-1]             -> 2^1 = 2 nodes  × c
                    /   \   /   \
  Level 2:       [n-2] ... ... [n-2]          -> 2^2 = 4 nodes  × c
                   .             .
  Level n:       [0] [0] ... [0] [0]          -> 2^n leaves     × c
  ```
- **Total nodes (geometric sum):** `1 + 2 + 4 + ... + 2^n = 2^(n + 1) - 1`।
- **Total Time:** `c * (2^(n + 1) - 1) = Θ(2^n)`! (`n = 40` হলেই `> 2 × 10^12` recursive calls—endpoint সম্পূর্ণ freeze!)
- **CRITICAL Stack-Depth Distinction:** Total work হলো **`Θ(2^n)`**, কিন্তু depth-first execution-এ যেকোনো মুহূর্তে root `[n]` থেকে একটা leaf `[0]` পর্যন্ত কেবল **একটিমাত্র path** stack-এ active থাকে! তাই **Max Call-Stack Depth = `Θ(n)`** (`Θ(2^n)` **নয়**!)।

### চারটি Canonical Recurrence এক নজরে

| Recurrence (Base: `Θ(1)`) | Tree Shape | Levels | Work per Level | Total Time | Depth-First Max Call-Stack Space |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **`T(n) = T(n - 1) + Θ(1)`** | Single chain | `Θ(n)` | `Θ(1)` | **`Θ(n)`** | **`Θ(n)`** |
| **`T(n) = T(n / 2) + Θ(1)`** | Halving chain | `Θ(log n)` | `Θ(1)` | **`Θ(log n)`** | **`Θ(log n)`** |
| **`T(n) = 2T(n / 2) + Θ(n)`** | Balanced binary tree | `Θ(log n)` | `Θ(n)` | **`Θ(n log n)`** | **`Θ(log n)`** |
| **`T(n) = 2T(n - 1) + Θ(1)`** | Full branching tree | `Θ(n)` | `Θ(2^level)` | **`Θ(2^n)`** | **`Θ(n)`** |

### Master Theorem কী (শুধু `Awareness` স্তরে)

অনেক university course ও textbook-এ divide-and-conquer recurrence `T(n) = a * T(n / b) + f(n)` (যেখানে প্রতি ধাপে `a >= 1`টি subproblem তৈরি হয়, প্রতিটির size `n / b`, আর ভাগ/জোড়া লাগানোর local কাজ `f(n)`)-এর জন্য **Master Theorem** নামে একটি ready-made সূত্র আছে। এটি আমাদের পুরো recursion tree না এঁকেই সরাসরি দেখে নিতে সাহায্য করে যে root-এর কাজ `f(n)` dominate করছে, leaf-গুলো dominate করছে, নাকি প্রতি level-এর কাজ সমান (যেমন Shape 2 ও Shape 3)।

*(Scope boundary: Master Theorem শুধু divide-and-conquer `n / b` আকারের recurrence-এ কাজ করে—Shape 4-এর `T(n - 1)`-এ নয়। এর formal continuous proofs, substitution-method induction, বা Akra-Bazzi theorem Phase 2-এর বাইরে; আপনার দৈনন্দিন backend কাজের জন্য ওপরের recursion-tree reasoning-ই যথেষ্ট ও বেশি স্বচ্ছ।)*

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-02-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> নিচের divide-and-conquer function-টি `amounts` array-র `[left..right]` অংশের সব invoice amount যোগ করে—এবার খেয়াল করুন combine ধাপে কোনো loop নেই, শুধু একটি `+` যোগ!
> ```javascript
> function sumInvoiceRange(amounts, left = 0, right = amounts.length - 1) {
>   if (left > right) return 0;
>   if (left === right) return amounts[left];
>   const mid = Math.floor((left + right) / 2);
>   return (
>     sumInvoiceRange(amounts, left, mid) +
>     sumInvoiceRange(amounts, mid + 1, right)
>   );
> }
> ```
> 1. `n = right - left + 1` (ধরুন `n = 2^k`) ধরে এর base case, progress measure, এবং exact recurrence relation লিখুন। Recursion tree এঁকে প্রতি level-এর work যোগ করে দেখান কেন এর **total time complexity** `Θ(n)` (`Θ(n log n)` নয়!)।
> 2. Depth-first execution-এ এই function-এর **maximum call-stack auxiliary space** কত—এবং কেন এটি total time-এর চেয়ে ছোট?

---

## Block 3-এর দিকে সেতুবন্ধন: যখন একই Code ভিন্ন Input-এ ভিন্ন আচরণ করে

Block 2-এ আপনি একটি শক্তিশালী practical দক্ষতা অর্জন করলেন: আসল JavaScript backend code পড়ে sequential/nested/triangular/halving loops, multi-variable inputs (`N`, `M`, `L`), hidden built-in costs (`.includes()`, `.shift()`, `[...acc, item]`-এর cumulative বনাম peak live memory), এবং recursion trees ও call-stack depth methodically derive করা।

কিন্তু এবার একটা অস্বস্তিকর বাস্তব প্রশ্ন সামনে চলে আসবে:
> *একই code যদি কোনো input-এ `1` ধাপে শেষ হয় আর আরেক input-এ `N` ধাপ নেয়—কিংবা একটি dynamic array-তে অধিকাংশ append (`arr.push()`) সস্তা হওয়ার পর কোনো এক occasional append-এ capacity বাড়িয়ে বিদ্যমান elements copy করতে হয় (যার exact resize threshold implementation-dependent)—তাহলে একজন backend engineer হিসেবে আপনি best-case, average-case, worst-case, আর amortized cost-এর মধ্যে কোনটা বিশ্বাস করবেন?*

ঠিক এই প্রশ্নের উত্তর নিয়েই আসছে **Phase 2, Block 3 — Worst-Case, Average-Case, Amortized Analysis, and Algorithmic Risk**!

---

## Study Plan, Learning Objectives, Terminology & Resources

> **Continuity & Ownership Notes:** আপনার সব Pause & Checks, দুটি Mandatory Exercise, এবং (যদি করতে চান) Optional Closed-Book Assessment-এর উত্তর স্বাধীনভাবে `phases/phase-02/notes/block-02-notes.md` ফাইলে লিখুন। AI-generated material কখনো completed learning বা learner study hours হিসেবে গণ্য হয় না।
>
> **Progression & Verification Policy:** এই block-এর Closed-Book Assessment সম্পূর্ণ **optional** এবং Block 3 শুরু করার জন্য mandatory gate নয়। পড়া ও দুটি mandatory exercise শেষ করে assessment skip করলে এই block-এর status হবে **`Studied — mastery not formally verified`**; যথেষ্ট evidence share করে ChatGPT evaluation সম্পন্ন হলে তা **`Mastery verified`** হিসেবে গণ্য হবে।

### Study Time Reference

*প্রতিটি activity-র বিস্তারিত time breakdown দেখতে এই নথির শুরুর **Study Time Guide** দেখুন। Approved dossier-এ Block 2-এর target allocation হলো `3 hours (180 min)`—যা একটি flexible planning target, কোনো deadline নয়।*

### Block 2 Learning Objectives *(Primary Target Level: `Practical Competence` | Supported Phase 2 Outcomes: Outcomes 2, 3)*

- **LO 2.1 (`Practical Competence` | Bloom: *Apply / Analyze*):** Derive tight time and auxiliary-space bounds for sequential code, independent nested loops, **dependent triangular loops** (`j = i + 1` → `Θ(N^2)`), **multiplicative/halving loops** (`i *= 2` → `Θ(log N)`), and **multi-variable backend inputs** (`O(N + M)`, `O(N * M)`, `O(N log M)`).
- **LO 2.2 (`Practical Competence` | Bloom: *Analyze*):** Spot **hidden non-`O(1)` costs inside single-line loops** (`Array.includes`, `Array.indexOf`, `Array.splice`/`shift`, string concatenation/slicing, and array spread-copying `[...acc, item]` inside `.reduce()`) that turn linear-looking loops into `O(N^2)` or `O(N * L)` work—explicitly distinguishing **total cumulative allocation/copying** (`Θ(N^2)`) from **peak live auxiliary memory** (object-lifetime and runtime-GC dependent) and **allocation churn**.
- **LO 2.3 (`Working Understanding` | Bloom: *Analyze*):** Trace **recursive algorithms** to determine both total work and **call-stack auxiliary space (`O(max recursion depth)`)**, identifying when unbounded recursion creates stack-overflow risks in backend trees and JSON payloads.
- **LO 2.4 (`Working Understanding` | Bloom: *Apply*):** Use **recursion trees** to solve the four canonical backend recurrence shapes—`T(n) = T(n - 1) + Θ(1)`, `T(n) = T(n/2) + Θ(1)`, `T(n) = 2T(n/2) + Θ(n)`, and `T(n) = 2T(n - 1) + Θ(1)`—with `Awareness` of the Master Theorem's role without formal theorem proofs. *(Approved dossier note: Block 2 intentionally mixes `Practical Competence` on iterative/hidden-cost code analysis with `Working Understanding` on recursion trees so the 3.0-hour target remains realistic.)*

### Terminology Inventory

| Term | Official Target Level | Why It Matters in Block 2 |
| :--- | :---: | :--- |
| **Multi-Variable Complexity (`N, M, L`)** | `Practical Competence` | Real APIs combine separate dimensions (orders, promo rules, string lengths); collapsing them hides true bottlenecks. |
| **Dependent Nested Loop (Triangular Sum)** | `Practical Competence` | Pairwise checks still scale quadratically (`N(N - 1)/2 = Θ(N^2)`). |
| **Hidden Operation Cost** | `Practical Competence` | One `.includes()`, `.indexOf()`, or `.shift()` inside a loop multiplies total work. |
| **Cumulative Allocation vs. Peak Live Auxiliary Memory** | `Working Understanding` | Separates `Θ(N^2)` cumulative copying/GC churn (`[...acc, item]`) from `Θ(N)` live reachable memory and GC-dependent peak RAM. |
| **Call-Stack Auxiliary Space** | `Working Understanding` | Recursive traversal consumes stack frames proportional to maximum recursion depth—causing stack overflows on deep skewed inputs. |
| **Recursion Tree & Recurrence Relation** | `Working Understanding` | Mechanically derives total work and stack depth without rote memorization. |
| **Master Theorem** | `Awareness` | High-level shortcut for divide-and-conquer `aT(n/b) + f(n)` recurrences; formal proofs deferred/excluded. |

### Mandatory Authoritative Resources *(Targeted Reading — ~50–60 min total)*

1. **Mandatory 1 (~25–30 min — assigned sections only):** Robert Sedgewick & Kevin Wayne (Princeton University), *Algorithms, 4th ed.* Booksite — [Section 1.4 — Analysis of Algorithms](https://algs4.cs.princeton.edu/14analysis/) (`LO 2.1`, `LO 2.2`):
   - **Assigned scope:** কেবল **Mathematical models** (tilde approximations ও order-of-growth classifications) এবং **Designing faster algorithms** (`3-Sum` `N^3` → `N^2 log N` আলোচনা)। **Explicitly defer *Memory usage* এবং *Observations* to Block 5।**
   - **What to extract:** Nested loops থেকে কীভাবে exact sum (`N^3`, `N^2`, `N^2 log N`) ও order-of-growth derive করা হয়।
2. **Mandatory 2 (~25–30 min — assigned sections only):** MIT OpenCourseWare 6.006 *Introduction to Algorithms* (Spring 2020) (`LO 2.1`, `LO 2.3`, `LO 2.4`):
   - [Recitation 1 Notes](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/mit6_006s20_r01/) — asymptotic ও loop-analysis উদাহরণ।
   - [Lecture 3: Sorting](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/mit6_006s20_lec3/) — **কেবল Merge Sort recursion-tree এবং recurrence-solving অংশটুকু (~2 pages); পুরো lecture পড়ার দরকার নেই।**
   - **What to extract:** Systematic loop counting এবং `T(n) = 2T(n/2) + Θ(n)`-এর ধাপে ধাপে recursion-tree সমাধান।

### Optional Supplementary Resources *(Outside Mandatory Budget — Do Not Treat as Mandatory)*

- **Supplementary (Free):** Jeff Erickson, *Algorithms* (UIUC) — [jeffe.cs.illinois.edu/teaching/algorithms/](https://jeffe.cs.illinois.edu/teaching/algorithms/) — Chapter 1 (*Recursion* — recursion trees overview)।
- **Supplementary (Commercial Book):** Thomas H. Cormen et al. (CLRS), *Introduction to Algorithms* (4th ed., MIT Press, 2022) — Chapter 2 (`2.2 Analyzing algorithms`) এবং Chapter 4 (`4.4 The recursion-tree method`)।

---

## Practical Exercises *(Mandatory — ~50–60 min total)*

> **Ownership Rule:** উভয় exercise-এর সব derivation, recursion tree, এবং explanation স্বাধীনভাবে `phases/phase-02/notes/block-02-notes.md` ফাইলে লিখুন। এখানে কোনো answer key দেওয়া হয়নি।

### Exercise 1: Multi-Variable & Hidden-Cost Code Audit *(Mandatory — ~25–30 min | `LO 2.1`, `LO 2.2`)*

নিচের **৬টি compact JavaScript backend snippet** (আমাদের online-store backend থেকে নেওয়া) analyze করুন। প্রতিটির জন্য লিখুন:
1. প্রতিটি **independent input variable** স্পষ্টভাবে define করুন (`N`, `M`, এবং প্রাসঙ্গিক হলে `L`)।
2. **Time complexity** exact variables-এ derive করুন (ধাপে ধাপে counting বা sum দেখিয়ে—শুধু final notation লিখবেন না)। যেখানে সব input-এ কাজ একই, সেখানে tight `Θ` bound দিন; আর যেখানে early exit-এর কারণে cost input-নির্ভর (যেমন search বা `.includes()`), সেখানে স্পষ্টভাবে **worst-case tight bound (`Θ`)** এবং general upper bound (`O`) আলাদা করে উল্লেখ করুন।
3. **Peak live auxiliary space** derive করুন।
4. **Dominating line / operation** চিহ্নিত করুন।
5. **Snippet 5-এর জন্য অতিরিক্ত বাধ্যতামূলক বিশ্লেষণ:** স্পষ্টভাবে তিন ভাগে আলাদা করুন—(ক) **Total cumulative copying/allocation**, (খ) **Live reachable auxiliary data**, এবং (গ) **Actual peak live memory ও allocation churn** কীভাবে object lifetimes ও runtime GC-র ওপর নির্ভর করে।

```javascript
// Snippet 1: Matching N orders against M promo rules
function countEligibleOrderRulePairs(orders, promoRules) {
  let matches = 0;
  for (let i = 0; i < orders.length; i++) {
    for (let j = 0; j < promoRules.length; j++) {
      if (orders[i].subtotalCents >= promoRules[j].minOrderCents) {
        matches += 1;
      }
    }
  }
  return matches;
}

// Snippet 2: Pairwise fraud-overlap checking (fixed-size integer hashes/IDs)
function countSharedCardAlerts(orders) {
  let alerts = 0;
  for (let i = 0; i < orders.length; i++) {
    for (let j = i + 1; j < orders.length; j++) {
      if (
        orders[i].cardHash === orders[j].cardHash &&
        orders[i].userId !== orders[j].userId
      ) {
        alerts += 1;
      }
    }
  }
  return alerts;
}

// Snippet 3: Halving binary-search-style loop on sorted weight thresholds
function findShippingTierIndex(sortedThresholds, weightGrams) {
  let low = 0;
  let high = sortedThresholds.length - 1;
  while (low <= high) {
    const mid = Math.floor((low + high) / 2);
    if (sortedThresholds[mid] === weightGrams) return mid;
    if (sortedThresholds[mid] < weightGrams) {
      low = mid + 1;
    } else {
      high = mid - 1;
    }
  }
  return -1;
}

// Snippet 4: Draining an in-memory notification queue (contiguous-array cost model)
function drainNotificationQueue(queue) {
  let processedWeight = 0;
  while (queue.length > 0) {
    const job = queue.shift();
    processedWeight += job.weight;
  }
  return processedWeight;
}

// Snippet 5: Accumulating export rows with spread inside .reduce()
function buildExportRows(orders) {
  return orders.reduce(
    (acc, order) => [...acc, { id: order.id, totalCents: order.amountCents }],
    []
  );
}

// Snippet 6: Filtering N cart items against an allowlist array of M numeric SKU IDs
function filterAllowedCartItems(cartItems, allowedSkuIds) {
  const validItems = [];
  for (const item of cartItems) {
    if (allowedSkuIds.includes(item.skuId)) {
      validItems.push(item);
    }
  }
  return validItems;
}
```

*(Snippet 6 Follow-up Question: যদি `skuId` fixed-size integer না হয়ে সর্বোচ্চ `L` characters দৈর্ঘ্যের string হতো এবং প্রতিটি string equality check abstract model-এ সর্বোচ্চ `O(L)` character comparison নিত, তবে worst-case time complexity exact `N`, `M`, ও `L`-এ কীভাবে বদলাত?)*

---

### Exercise 2: Recursion Tree & Stack-Space Analysis *(Mandatory — ~25–30 min | `LO 2.3`, `LO 2.4`)*

নিচের **৩টি recursive backend utility** (কোনো memoization ছাড়া) analyze করুন। প্রতিটির জন্য লিখুন:
1. **Base case** ও **progress measure** চিহ্নিত করুন।
2. **Recursion tree** আঁকুন বা পরিষ্কারভাবে বর্ণনা করুন (levels, branching factor, প্রতি node-এ local work, প্রতি level-এ total work)।
3. **Total time complexity** derive করুন (Utility 2 ও 3-এর exact recurrence relation লিখে সমাধান করুন)।
4. **Maximum call-stack auxiliary space** derive করুন (total work-এর সাথে পার্থক্য স্পষ্ট করে)।
5. **Failure input shape & operational backend consequence:** ঠিক কোন input shape-এ এটি **stack overflow** অথবা **exponential CPU freeze** ঘটাবে, এবং live checkout/catalog backend-এ এর বাস্তব পরিণতি কী হবে?

```javascript
// Utility 1: Recursive category-tree depth and sales aggregator
// Each node has { ownSalesCents: number, children: Node[] }
function summarizeCategoryTree(node) {
  if (!node) {
    return { maxDepth: 0, totalCents: 0 };
  }
  let maxChildDepth = 0;
  let totalCents = node.ownSalesCents;
  for (const child of node.children) {
    const sub = summarizeCategoryTree(child);
    if (sub.maxDepth > maxChildDepth) {
      maxChildDepth = sub.maxDepth;
    }
    totalCents += sub.totalCents;
  }
  return { maxDepth: 1 + maxChildDepth, totalCents };
}

// Utility 2: Divide-and-conquer buy-then-sell max price spread over hourly prices
function maxBuySellSpread(prices, left = 0, right = prices.length - 1) {
  if (left >= right) {
    return 0;
  }
  const mid = Math.floor((left + right) / 2);
  const bestLeft = maxBuySellSpread(prices, left, mid);
  const bestRight = maxBuySellSpread(prices, mid + 1, right);

  let minLeft = prices[left];
  for (let i = left + 1; i <= mid; i++) {
    if (prices[i] < minLeft) minLeft = prices[i];
  }
  let maxRight = prices[mid + 1];
  for (let j = mid + 2; j <= right; j++) {
    if (prices[j] > maxRight) maxRight = prices[j];
  }
  const crossSpread = Math.max(0, maxRight - minLeft);
  return Math.max(bestLeft, bestRight, crossSpread);
}

// Utility 3: Naive recursive promo-combination counter (no memoization or pruning)
function countExactDiscountCombos(discounts, remainingTargetCents, index = 0) {
  if (index === discounts.length) {
    return remainingTargetCents === 0 ? 1 : 0;
  }
  const skipCurrent = countExactDiscountCombos(
    discounts,
    remainingTargetCents,
    index + 1
  );
  const useCurrent = countExactDiscountCombos(
    discounts,
    remainingTargetCents - discounts[index],
    index + 1
  );
  return skipCurrent + useCurrent;
}
```

---

## Closed-Book Synthesis Assessment *(Optional — ~20–25 min)*

এই assessment-টি সম্পূর্ণ **optional**—Block 3 শুরু করার জন্য এটি সম্পন্ন করা বা ChatGPT-র approval নেওয়া mandatory নয়। আপনি চাইলে:
- এই lesson ও সব browser tab বন্ধ রেখে নিচের ৫টি প্রশ্নের নিজের ভাষায় উত্তর `phases/phase-02/notes/block-02-notes.md` ফাইলে লিখে formal evaluation-এর জন্য **ChatGPT**-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে);
- অথবা আপনি যথেষ্ট বুঝেছেন মনে করলে assessment skip করে Block 3-এ এগোতে পারেন (`Studied — mastery not formally verified`);
- অথবা assessment না করেই যেকোনো unclear concept নিয়ে প্রশ্ন ও আলোচনা করতে পারেন।

*(Note: `~20–25 min` কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি approximate estimate—কোনো strict time limit নয়।)*

1. **(LO 2.1 / Outcomes 2, 3)** Sequential loops, independent nested loops, dependent triangular loops (`j = i + 1`), এবং multiplicative/halving loops (`i *= 2`) কীভাবে analyze করতে হয় তা একটি করে সংক্ষিপ্ত algebraic derivation সহ ব্যাখ্যা করুন। কেন `j = i + 1` হওয়া সত্ত্বেও triangular loop এখনো `Θ(N^2)`?
2. **(LO 2.1 / Outcome 3)** Backend API বিশ্লেষণে `N`, `M`, এবং `L`-কে আলাদা independent variable হিসেবে রাখা কেন জরুরি? `Θ(N + M)`, `Θ(N * M)`, এবং `Θ(N log M)`-এর প্রতিটি কখন ঘটে তা বাস্তব online-store উদাহরণ দিয়ে দেখান।
3. **(LO 2.2 / Outcome 3)** কেন শুধু visible `for` keyword গুনে JavaScript code-এর complexity বলা যায় না? `.includes()`/`.indexOf()`, `.shift()`/`.splice()`, এবং string concatenation/slicing-এর underlying cost model ব্যাখ্যা করুন (এবং কোথায় engine-specific qualification মনে রাখা দরকার তা উল্লেখ করুন)।
4. **(LO 2.2 / Outcome 3)** `items.reduce((acc, item) => [...acc, item], [])`-এর উদাহরণ দিয়ে **total cumulative allocation/copying (`Θ(N^2)`)**, **live reachable auxiliary memory (`Θ(N)`)**, এবং **actual peak live memory / allocation churn**-এর পার্থক্য ব্যাখ্যা করুন।
5. **(LO 2.3, LO 2.4 / Outcome 3)** চারটি canonical recurrence (`T(n) = T(n-1) + Θ(1)`, `T(n) = T(n/2) + Θ(1)`, `T(n) = 2T(n/2) + Θ(n)`, `T(n) = 2T(n-1) + Θ(1)`)-এর recursion tree structure, **total work**, এবং **depth-first maximum call-stack space** তুলনা করুন—বিশেষত ব্যাখ্যা করুন কেন `2T(n/2) + Θ(n)`-এ stack depth `Θ(log n)` এবং `2T(n-1) + Θ(1)`-এ stack depth `Θ(n)`।

### Block 2 Readiness & Progression Checklist

- [ ] Guided lesson মনোযোগ দিয়ে পড়েছেন এবং Pause & Checks 1–3 `phases/phase-02/notes/block-02-notes.md`-এ লিখে (পরে Self-Check Guidance মিলিয়ে) সম্পন্ন করেছেন।
- [ ] দুটি Mandatory Reading (**Princeton Algorithms 1.4** নির্ধারিত অংশ এবং **MIT 6.006 Recitation 1 + Lecture 3** Merge Sort recurrence excerpt) সম্পন্ন করেছেন।
- [ ] `phases/phase-02/notes/block-02-notes.md` ফাইলে স্বাধীনভাবে **Exercise 1 (Multi-Variable & Hidden-Cost Code Audit)** এবং **Exercise 2 (Recursion Tree & Stack-Space Analysis)** সম্পন্ন করেছেন।
- [ ] *(Optional)* Notes/tabs বন্ধ রেখে Closed-Book Synthesis Assessment Questions 1–5 লিখেছেন এবং formal evaluation-এর জন্য ChatGPT-র কাছে share করেছেন।

> **Progression Status Distinction:** Block 3 কখন শুরু করবেন তা আপনার সিদ্ধান্ত। পড়া ও দুটি mandatory exercise শেষ করে optional assessment ছাড়া এগোলে এই block-টির status হবে **`Studied — mastery not formally verified`**। যথেষ্ট evidence share করে ChatGPT evaluation সম্পন্ন হলেই তা **`Mastery verified`** হিসেবে গণ্য হবে।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`phases/phase-02/notes/block-02-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের guidance দেখুন। Exercise 1, Exercise 2, বা Optional Closed-Book Assessment-এর কোনো solution এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও (চাইলে) optional ChatGPT review-এর জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* প্রথম loop `accounts` array-র ওপর `N` বার চলে (`Θ(N)`)। দ্বিতীয় loop-এ `size` প্রতিবার `2` গুণ হয় (`1, 2, 4, ..., 2^k`), তাই এটি থামে `ceil(log2(M))` ধাপে—অর্থাৎ `M >= 2`-এর জন্য `Θ(log M)` (আর `M = 1` হলে `0` বার)। দুটি loop **sequentially** চলায় মোট time যোগ হবে: **`Θ(N + log M)`** (যেহেতু `N` ও `M` independent, তাই দুটোকেই আলাদা রাখতে হবে; `M = 1` হলে তা `Θ(N)` হয়)। Auxiliary space হলো **`Θ(1)`** (শুধু কয়েকটি scalar variable ও fixed ২-field return object)।
  2. *শক্তিশালী উত্তরে থাকবে:* `i = 0` হলে `j` চলে `0` বার; `i = 1` হলে `1` বার; `i = 2` হলে `2` বার; ...; `i = N - 1` হলে `N - 1` বার। মোট execution = `0 + 1 + 2 + ... + (N - 1) = N * (N - 1) / 2 = (1/2) * N^2 - (1/2) * N`। Leading constant `1/2` এবং lower-order term `-(1/2) * N` বাদ পড়লে tight bound হলো **`Θ(N^2)`**।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* সব `N`টি tag unique হওয়ায় `i`-তম iteration-এ `result`-এর length থাকে `i`। এখানে **দুটি** লাইনই প্রতিবার `Θ(i)` কাজ করে: (ক) `result.indexOf(tags[i])` পুরো `i`টি element scan করে `-1` ফেরত দেয় (`0 + 1 + ... + (N - 1) = N(N - 1)/2` comparisons); এবং (খ) `result = [...result, tags[i]]` আগের `i`টি element copy করে নতুন `i + 1` length-এর array বানায় (`1 + 2 + ... + N = N(N + 1)/2` writes)। উভয় মিলিয়ে মোট work `Θ(N^2)`।
  2. *শক্তিশালী উত্তরে থাকবে:* (ক) **Total cumulative allocation/copying:** `1 + 2 + ... + N = N(N + 1)/2 = Θ(N^2)` elements allocated ও copied। (খ) **Live reachable auxiliary memory:** যেকোনো step-এ কেবল আগের `result` ও নতুন `result` reachable, তাই live reachable auxiliary data হলো **`Θ(N)`** (`Θ(N^2)` নয়)। (গ) **Actual peak live memory ও allocation churn:** দ্রুত `Θ(N^2)` অস্থায়ী array তৈরি হয়ে পরিত্যক্ত হওয়ায় মারাত্মক **allocation churn / GC pressure** তৈরি হয়, এবং object lifetimes ও GC cycle-এর সময়ের ওপর নির্ভর করে actual transient peak memory `Θ(N)`-এর চেয়ে সাময়িকভাবে বেশি হতে পারে।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* **Base case:** `left > right` (`n = 0`) বা `left === right` (`n = 1`), কাজ `Θ(1)`। **Progress measure:** `n = right - left + 1` প্রতিটি recursive call-এ অর্ধেক হয় (`n → n / 2`)। **Recurrence:** `T(n) = 2T(n / 2) + Θ(1)` (দুটি অর্ধেক subproblem, কিন্তু combine-এ শুধু একটি `Θ(1)` যোগ)। Recursion tree-তে level `d`-এ `2^d`টি node এবং প্রতিটির local work `c` → প্রতি level-এর work হলো `c * 2^d`। Level `0` থেকে `log2(n)` পর্যন্ত মোট কাজ = `c * (1 + 2 + 4 + ... + n) = c * (2n - 1) = Θ(n)`! *(এখানে leaf-গুলো মোট কাজ dominate করে; তাই `Θ(n log n)` নয়, **`Θ(n)`**!)*
  2. *শক্তিশালী উত্তরে থাকবে:* Depth-first execution-এ বাম অর্ধেকের (`left..mid`) recursive call পুরো শেষ হয়ে তার সব stack frames pop হওয়ার *পরেই* ডান অর্ধেকের (`mid + 1..right`) call শুরু হয়। তাই যেকোনো মুহূর্তে call stack-এ root (`n`) থেকে একটি leaf (`1`) পর্যন্ত কেবল **একটিমাত্র halving path** active থাকে—সর্বোচ্চ `log2(n) + 1` frames → **Max Call-Stack Auxiliary Space = `Θ(log n)`** (total time `Θ(n)`-এর চেয়ে অনেক ছোট!)।
