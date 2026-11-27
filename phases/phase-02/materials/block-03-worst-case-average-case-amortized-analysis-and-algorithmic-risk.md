# Phase 2, Block 3 — Worst-Case, Average-Case, Amortized Analysis, and Algorithmic Risk

## Study Time Guide

| Activity | Assigned Scope | Estimated Time |
| :--- | :--- | :---: |
| **Guided Lesson + Pause & Checks 1–3** *(main priority)* | পুরো guided lesson মনোযোগ দিয়ে পড়া এবং `phases/phase-02/notes/block-03-notes.md`-এ Pause & Checks 1–3 লেখা | **~50–65 min** *(flexible — limit নয়)* |
| **Mandatory 1:** MIT OCW 6.006 (Spring 2020) — *Lecture 2: Data Structures* | Lecture 2 Notes PDF — *Sequence Interface*, *Static vs. Dynamic Arrays*, এবং table doubling-এর *Amortized Analysis* | ~25 min |
| **Mandatory 2:** Pat Morin — *Open Data Structures* (`2.1 ArrayStack`) | কেবল Subsections `2.1.2 Growing and Shrinking` এবং `2.1.3 Summary` *(shrinking hysteresis ও thrashing)* | ~15–20 min |
| **Mandatory 3:** Crosby & Wallach (USENIX Security 2003) | *Abstract*, *Section 1 (Introduction)*, এবং hash-table collision attack-এর conceptual overview *(Perl/Squid/Bro C-code exploit internals বাদ দিন)* | ~15–20 min |
| **Exercise 1 (Mandatory):** Dynamic Buffer Resizing Analysis | Capacity `10` থেকে `N = 100,000` events-এ `+100`, `×2`, এবং `×2 / ÷2` thrashing policy-র exact Aggregate Method তুলনা | ~25–30 min |
| **Exercise 2 (Mandatory):** Case-Analysis & Production Risk Audit | ৪টি বাস্তব backend scenario-র case-analysis বা reasoning error এবং production failure mode নির্ণয় | ~25–30 min |
| **Total (Required Core — without Optional Assessment)** | Guided Lesson + ৩টি Mandatory Reading + ২টি Mandatory Exercise | **~155–190 min (~2.6–3.2 h)** |
| **Optional Closed-Book Assessment** *(optional)* | Notes/tabs বন্ধ রেখে ৫টি synthesis প্রশ্নের উত্তর লেখা | ~20–25 min *(optional)* |
| **Total (with Optional Assessment)** | Required Core + Optional Closed-Book Assessment | **~175–215 min (~2.9–3.6 h)** |

> **Planning Note:** Approved Phase 2 Research Dossier-এ Block 3-এর roadmap target allocation হলো **3 hours (180 minutes)**—যা একটি flexible planning reference, কোনো strict limit নয়। দ্রুত শেষ করার চেয়ে প্রতিটি derivation ও distinction হাতে-কলমে বুঝে নেওয়াই main priority; বিশেষ করে **Guided Lesson**-এ প্রয়োজনে আনুমানিক সময়ের চেয়ে বেশি সময় লাগা একদম স্বাভাবিক। প্রতিটি reading estimate কেবল নির্ধারিত sections-এর জন্য, পুরো lecture বা paper পড়ার জন্য নয়। **Optional Closed-Book Assessment** সম্পূর্ণ optional এবং Block 4 শুরু করার জন্য কোনো mandatory gate নয়।

---

Block 2-এর শেষে আপনি আসল JavaScript backend code পড়ে loops, hidden built-in costs, আর recursion trees derive করা শিখেছেন। ধরে নিন সেই দক্ষতা কাজে লাগিয়ে আপনি আমাদের online store-এর checkout ও order-ingestion backend-এর সব accidental nested loop সরিয়ে ফেলেছেন। Staging environment-এ সব test সুন্দর green, গড় response time মাত্র কয়েক milliseconds।

তারপর এক busy campaign সপ্তাহে production-এ পরপর তিনটি অদ্ভুত ঘটনা ঘটল—যার কোনোটাতেই কোনো syntax bug বা accidental nested `for` loop নেই:

1. **Order Export Timeout:** আমাদের report service-এর একটি sorting routine staging-এ random sample data-তে চমৎকার `O(N log N)` speed-এ চলছিল—কিন্তু production-এ যখন database থেকে `createdAt` timestamp অনুযায়ী আগে থেকেই sorted `100,000` order export করা হলো, তখন সেই একই routine হঠাৎ মিনিটের পর মিনিট আটকে গেল!
2. **Checkout Tail-Latency Spike:** আমাদের in-memory audit buffer-এ event append করা **`O(1)` amortized**—৯৯.৯% checkout request-এ কাজটা microsecond-এ শেষ হয়। অথচ হঠাৎ কোনো একটি দুর্ভাগা live checkout request-এ ঠিক ওই একটিমাত্র append call কয়েক দশ milliseconds থমকে গিয়ে downstream timeout টেনে আনল!
3. **Modest Request, `100%` CPU:** কোনো বিশাল traffic surge ছাড়াই একটিমাত্র সাধারণ আকারের HTTP request—হয় অস্বাভাবিক কিছু query parameter সহ, নয়তো একটি ছোট malformed promo string সহ—আমাদের backend worker-এর CPU আটকে দিল!

একটু থেমে ভাবুন: **একই code যদি কোনো input-এ ঝড়ের গতিতে চলে, অন্য input-এ ধসে পড়ে, আর `O(1)` amortized হওয়া সত্ত্বেও কোনো কোনো single request-এ হঠাৎ বড় latency spike ঘটায়—তাহলে একজন backend engineer হিসেবে কোন complexity সংখ্যার ওপর ভরসা করবেন, আর কখন সেই সংখ্যা আপনাকে ভুল নিরাপত্তা দেবে?**

এই একটি প্রশ্নই পুরো Block 3-এর মূল মেরুদণ্ড। প্রথমে আমরা পরিষ্কার করব **best-case, average-case, expected-case, ও worst-case**-এর সূক্ষ্ম পার্থক্য; তারপর গাণিতিকভাবে prove করব **amortized analysis** ও **shrinking hysteresis**; আর শেষে দেখব কীভাবে attacker-রা worst-case complexity-কে অস্ত্র বানিয়ে **Algorithmic Complexity Attacks** ঘটায়।

---

## ১. চারটি ভিন্ন প্রশ্ন: Best-Case, Average-Case, Expected-Case, ও Worst-Case

Block 1-এ আপনি একটি অত্যন্ত জরুরি নিয়ম শিখেছিলেন, যেটা এখানে আরেকবার মাথায় গেঁথে নেওয়া দরকার:

> **Asymptotic bound (`O`, `Ω`, `Θ`) এবং Case Category (`best`, `average`, `expected`, `worst`) দুটি সম্পূর্ণ আলাদা dimension!**

`O(...)` মানেই কখনো স্বয়ংক্রিয়ভাবে worst-case নয়। আপনি যেমন *worst-case* cost-এর জন্য tight bound `Θ(N)` বা upper bound `O(N)` লিখতে পারেন, তেমনি *best-case* cost-এর জন্যও `O(1)` বা `Θ(1)` লেখা বৈধ। তাই কোনো algorithm-এর complexity বলার সময় দুটি কথা **আলাদাভাবে** বলতে হয়: *(১) আপনি কোন case-এর কথা বলছেন*, এবং *(২) সেই case-এর asymptotic bound কী*।

এবার আসুন একটি চেনা backend function আর একটি পরিচিত sorting উদাহরণ দিয়ে **চারটি case category** ছুরি দিয়ে আলাদা করি।

ধরুন আমাদের order lookup helper-টি একটি unsorted array-তে `targetId` খুঁজছে:

```javascript
function findOrderIndex(orders, targetId) {
  for (let i = 0; i < orders.length; i++) {
    if (orders[i].id === targetId) {
      return i; // early exit on match
    }
  }
  return -1;
}
```

এখানে `N = orders.length` হলে একই `N` সাইজের ভিন্ন ভিন্ন input-এর জন্য `findOrderIndex` একদম আলাদা পরিমাণ কাজ করে:

### ক. Best-Case Complexity (সবচেয়ে ভাগ্যবান input)
- **সংজ্ঞা:** Size `N`-এর **সবচেয়ে অনুকূল** input-টিতে algorithm যত কম কাজ করে।
- **উদাহরণ:** `orders[0].id === targetId` হলে প্রথম comparison-এই function return করে → **Best-case tight bound = `Θ(1)`**।
- **Backend reality:** Production SLA বা SLO guarantee-র জন্য best-case কার্যত **মূল্যহীন**! কোনো customer বা attacker আপনাকে প্রতিশ্রুতি দেবে না যে তাদের খুঁজতে চাওয়া order সবসময় array-র একদম শুরুতেই বসে থাকবে।

### খ. Average-Case Complexity (Input-এর Probability Distribution-এর ওপর গড়)
- **সংজ্ঞা:** Size `N`-এর সব সম্ভাব্য input-এর ওপর একটি **নির্দিষ্ট probability distribution ধরে নিয়ে** গড় (mathematical expectation) cost হিসাব করা।
- **উদাহরণ (`findOrderIndex`):** ধরুন আপনি **assume** করলেন: (১) `targetId` সবসময় `orders`-এর ভেতর আছে, এবং (২) `0` থেকে `N - 1` প্রতিটি index-এ থাকার সম্ভাবনা সমান (`1 / N`)। তখন গড় comparison সংখ্যা:
  ```text
  Average comparisons = (1 + 2 + 3 + ... + N) / N
                      = [N * (N + 1) / 2] / N
                      = (N + 1) / 2  =  Θ(N)
  ```
- **আরও ভয়ংকর উদাহরণ (আমাদের opening Order Export Timeout!):** ধরুন একটি **deterministic Quicksort** সবসময় subarray-র **প্রথম element-কে pivot** হিসেবে বেছে নেয় *(Quicksort-এর full implementation ও comparison Block 4-এ; এখানে শুধু pivot বাছাইয়ের প্রভাবটুকু দেখুন)*।
  - যদি input array-র সব `N!` permutation সমান সম্ভাব্য (uniformly random) হয়, তবে এর **average-case** সময় হয় **`Θ(N log N)`**।
  - কিন্তু আপনার online store-এর order export-গুলো কি uniformly random? **না!** Database থেকে timestamp অনুযায়ী export করলে ওগুলো প্রায়ই **আগে থেকেই sorted** (অথবা reverse-sorted) থাকে!
  - আগে থেকে sorted array-তে প্রতিবার প্রথম element (সবচেয়ে ছোট মান)-কে pivot ধরলে বাম পাশে থাকে `0`টি element আর ডান পাশে `N - 1`টি element! তখন recurrence দাঁড়ায় `T(N) = T(N - 1) + Θ(N)` → `(N - 1) + (N - 2) + ... + 1 = Θ(N^2)`!
- **মূল শিক্ষা:** Average-case analysis-এর সবচেয়ে বড় দুর্বলতা হলো এর **input-distribution assumption**। বাস্তব production data প্রায় কখনোই বইয়ের মতো uniform random নয়—তা সময়ক্রমে sorted, skewed, বা malicious হতে পারে। Assumption ভাঙলেই আপনার "average-case" প্রতিশ্রুতি ভেঙে পড়ে!

### গ. Expected-Case Complexity (Randomized Algorithm-এর নিজস্ব Randomness-এর ওপর গড়)
এবার একটি অত্যন্ত সূক্ষ্ম কিন্তু শক্তিশালী পার্থক্য দেখুন—অনেক engineer **Average-Case** আর **Expected-Case**-কে এক করে ফেলেন, অথচ দুটোর উৎস সম্পূর্ণ ভিন্ন!

- **সংজ্ঞা:** Algorithm-টি যখন execution-এর ভেতরে **নিজেই random choice করে** (যেমন random number generator দিয়ে pivot বাছা বা random hash seed ব্যবহার করা), তখন **যেকোনো একটি নির্দিষ্ট (এমনকি সবচেয়ে জঘন্য বা attacker-তৈরি) input-এর ওপরও** algorithm-টির নিজের internal random choices-এর গড় cost হলো **Expected-Case Complexity**!
- **উদাহরণ (Randomized Quicksort):** প্রতিবার প্রথম element না নিয়ে যদি algorithm-টি `[left..right]` রেঞ্জ থেকে **নিজে uniformly random একটি pivot** বেছে নেয়, তবে আপনার database থেকে আসা timestamp-sorted order export (বা কোনো attacker-এর বানানো input)-এর ওপরও এর **expected running time হলো `Θ(N log N)`**!
- **পার্থক্যটা চোখে আঙুল দিয়ে দেখুন:**
  - **Distributional Average-Case:** *"আমি আশা করছি user-রা আমাকে random input পাঠাবে।"* (Control আছে user/attacker-এর হাতে—ভঙ্গুর!)
  - **Randomized Expected-Case:** *"User বা attacker যে input-ই পাঠাক না কেন, আমি ভেতরে নিজের randomness ব্যবহার করে গড়ে ভালো behavior নিশ্চিত করছি।"* (Control আছে algorithm-এর হাতে—তাই কোনো প্রমাণহীন distributional average-এর চেয়ে adversarial fixed input-এর বিরুদ্ধে এটি অনেক বেশি শক্তিশালী!)
  *(অত্যন্ত জরুরি সীমারেখা: কেবল **expected running time = `Θ(N log N)`** প্রমাণ হওয়াই স্বয়ংক্রিয়ভাবে কোনো **high-probability bound** প্রতিষ্ঠা করে না, কিংবা কোনো একটি individual execution-এ long tail না ঘটার guarantee দেয় না! Expectation হলো algorithm-টির internal random choices-এর ওপর গাণিতিক গড়—একটি নির্দিষ্ট run-এ খারাপ tail ঘটার সম্ভাবনা আসলেই অতি ক্ষুদ্র কি না তা জানতে হলে আলাদা **probability / tail analysis** প্রয়োজন। তাই randomized expected-case কোনো unjustified distributional average-এর চেয়ে অনেক শক্তিশালী হলেও, এটি নিজে থেকে প্রতিটি single request-এর জন্য hard per-run latency bound নয়।)*

### ঘ. Worst-Case Complexity (সবচেয়ে প্রতিকূল input-এর সর্বোচ্চ সীমা)
- **সংজ্ঞা:** Size `N`-এর **যেকোনো** বৈধ input-এর জন্য algorithm সর্বোচ্চ যে কাজ করতে পারে।
- **উদাহরণ:** `findOrderIndex`-এ `targetId` array-তে না থাকলে পুরো `N`টি element scan করতে হয় → **Worst-case tight bound = `Θ(N)`**। Deterministic first-pivot Quicksort-এ sorted input-এ **Worst-case = `Θ(N^2)`**।
- **Backend SLA/SLO-তে এর গুরুত্ব:** Production backend-এ service level agreement (SLA) বা reliability target (SLO) রক্ষা করতে গেলে আপনাকে **কোনো input-ই যেন service-কে অচল করতে না পারে** সেই ঝুঁকি নিয়ন্ত্রণ করতে হয়। তাই কঠোর per-request সীমানার জন্য engineers **Worst-case bounds**-এর ওপর ভরসা করেন—অথবা যেখানে আলাদা tail analysis দিয়ে প্রমাণিত **defensible high-probability bounds** আছে, সেগুলোর ওপর (শুধু best-case, অপ্রমাণিত distributional average, কিংবা কেবল expectation-কে hard per-run latency bound ধরে নয়!)।

| Case Category | কিসের ওপর গড় বা সীমা নেওয়া হয়? | Input-এর ওপর assumption লাগে? | Backend Reliability / SLA-র জন্য ভরসাযোগ্য? |
| :--- | :--- | :---: | :--- |
| **Best-Case** | সবচেয়ে অনুকূল single input | হ্যাঁ (চরম ভাগ্যবান input) | **না** — production-এ প্রায় মূল্যহীন। |
| **Average-Case** *(Distributional)* | সব input-এর ওপর assumed probability distribution | **হ্যাঁ** (fragile — data skewed/sorted হতে পারে) | **সাবধান!** কেবল বাস্তব input distribution প্রমাণিত ও non-adversarial হলে। |
| **Expected-Case** *(Randomized)* | যেকোনো fixed input-এর জন্য algorithm-এর নিজস্ব random choices | **না** (যেকোনো fixed input-এ প্রযোজ্য) | **কেবল expectation!** Unjustified average-এর চেয়ে শক্তিশালী, কিন্তু hard per-run বা high-probability tail guarantee নয় (তাতে আলাদা tail analysis লাগে)। |
| **Worst-Case** | size `N`-এর সবচেয়ে প্রতিকূল input | **না** (সর্বজনীন upper envelope) | **হ্যাঁ** — hardest per-run guarantee, adversarial input-এও নিরাপদ। |

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-03-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> একটি logistics backend-এ দুটি sorting approach তুলনা করা হচ্ছে:
> - **Approach A (Deterministic):** প্রায়-sorted input-এ `Θ(N)`, uniformly random input-এ `Θ(N log N)`, কিন্তু reverse-sorted input-এ `Θ(N^2)` সময় নেয়।
> - **Approach B (Randomized):** Execution-এর ভেতর নিজের random choices ব্যবহার করে **যেকোনো** input-এর জন্য `Θ(N log N)` expected time দেয়।
> 1. একজন developer বলছেন: *"Approach A তো average-case-এ `O(N log N)`—তাহলে এটা নিরাপদ!"* যদি কোনো upstream partner newest-first (reverse-sorted) order batch পাঠায়, তবে এই যুক্তি কেন ভেঙে পড়বে?
> 2. Approach A-র **average-case** দাবি এবং Approach B-র **expected-case** দাবির মৌলিক পার্থক্য কী? আর শুধু "`O(N log N)`" লিখলেই কেন বোঝা যায় না কোন case-এর কথা বলা হচ্ছে?

এখন আমরা প্রথম রহস্যটা (Order Export Timeout) বুঝে ফেললাম। কিন্তু আমাদের দ্বিতীয় রহস্যটা আরও চমকপ্রদ!

ধরুন কোনো random input distribution নেই, কোনো random choice-ও নেই। আমাদের online store-এর batch-ingestion buffer-এ আমরা শুধু একটার পর একটা event `push` করে যাচ্ছি। অধিকাংশ `push` ১ ধাপেই শেষ হয়ে যায়, কিন্তু মাঝে মাঝে যখন buffer-এর জায়গা ফুরিয়ে যায়, তখন একটিমাত্র `push` বিশাল এক copy-র কাজ করে `Θ(N)` সময় নেয়!

এখন প্রশ্ন হলো: **একটি `push`-এর worst-case যেহেতু `Θ(N)`, তাহলে `N`টি `push` করতে কি মোট `Θ(N^2)` সময় লাগবে?** নাকি পুরো sequence-এর খরচ একসাথে হিসাব করলে একদম ভিন্ন ছবি বেরিয়ে আসে?

---

## ২. Dynamic Array Resizing ও Aggregate Method দিয়ে Amortized Analysis

উত্তরটা পেতে হলে আগে বুঝতে হবে একটি **Dynamic Array** (যেমন সাধারণ resizable array model) memory-তে ঠিক কীভাবে বড় হয়।

Memory-তে একটি সাধারণ array হলো পাশাপাশি থাকা (contiguous) fixed-size slot-এর সারি—যার শুরুতে একটি নির্দিষ্ট **`capacity`** থাকে (ধরুন `capacity = 4`) এবং বর্তমানে ভেতরে থাকা element সংখ্যা হলো **`size`** (শুরুতে `size = 0`)।
- যতক্ষণ `size < capacity`, একটি নতুন event শেষে যোগ করা (`append` / `push`) সহজ: পরের খালি slot-এ মানটা বসিয়ে `size += 1` → **মাত্র `1`টি write (`Θ(1)`)!**
- কিন্তু যেই মুহূর্তে `size === capacity` হয়ে যায়, সেই fixed block-এর ঠিক পরের memory-তে অন্য data থাকতে পারে—তাই ওখানেই বাড়ানো যায় না! তখন করতে হয় তিন ধাপ:
  1. নতুন, বড় **`newCapacity`** সাইজের একটি contiguous array allocate করা,
  2. পুরোনো array-র সব `size`টি element এক-এক করে নতুন array-তে **copy** করা (`size`টি copy write),
  3. নতুন element-টি শেষে বসানো (`1`টি write) এবং পুরোনো array ছেড়ে দেওয়া।

*(গুরুত্বপূর্ণ বাস্তব সতর্কতা: সব programming language বা runtime হুবহু একই growth factor বা initial capacity ব্যবহার করে না—কেউ `2×`, কেউ `1.5×`, কেউ hybrid strategy ব্যবহার করে। এখানে mechanism পরিষ্কারভাবে prove করার জন্য আমরা দুটি classic, সুনির্দিষ্ট model তুলনা করছি: **Fixed-Increment `+k`** বনাম **Geometric Doubling `×2`**।)*

### Model A: Fixed-Increment Growth (`+k` slots) — কেন এটি `Θ(N^2)` বিপর্যয়!

ধরুন আমরা একটি "মিতাচারী" policy বানালাম যাতে বাড়তি memory অপচয় না হয়:
- **Policy Assumptions:** Initial capacity `c0 = k` (যেখানে `k >= 1` একটি fixed constant, যেমন `k = 10` বা `k = 100`)। যখনই `size === capacity` অবস্থায় নতুন append আসে, capacity বাড়িয়ে `capacity + k` করা হয় এবং বিদ্যমান সব `capacity`টি element নতুন array-তে copy করা হয়।
- **Cost Model:** প্রতিটা ordinary append write = `1` unit; resize-এর সময় প্রতিটা element copy = `1` unit।

এবার ধরুন আমরা `N = m * k`টি event একে একে append করছি (`m >= 1`টি পূর্ণ block, যেখানে `m = N / k`)। ঠিক কখন কখন resize ঘটে, হাতে গুনুন:
- প্রথম `k`টি append (`1` থেকে `k`): কোনো resize নেই (`0` copies)। এখন `size = k, cap = k`।
- `(k + 1)`-তম append: Resize #1! `k`টি element copy হয় (`cap: k → 2k`)।
- `(2k + 1)`-তম append: Resize #2! `2k`টি element copy হয় (`cap: 2k → 3k`)।
- `(3k + 1)`-তম append: Resize #3! `3k`টি element copy হয় (`cap: 3k → 4k`)।
- ...
- `((m - 1)k + 1)`-তম append: Resize `#(m - 1)`! `(m - 1)k`টি element copy হয় (`cap: (m - 1)k → mk`)।

এবার **সব resize copy একসাথে যোগ করুন (Aggregate Method):**
```text
Total Copies = k + 2k + 3k + ... + (m - 1)k
             = k * [1 + 2 + 3 + ... + (m - 1)]
             = k * [(m - 1) * m / 2]
```
যেহেতু `m = N / k`, তাই `m`-এর মান বসালে:
```text
Total Copies = k * (N / k) * (N / k - 1) / 2
             = N * (N / k - 1) / 2
             = N^2 / (2k) - N / 2
             = Θ(N^2)
```
এর সাথে `N`টি ordinary append write যোগ করলে:
```text
Total Cost over N appends = N + N^2 / (2k) - N / 2
                          = N^2 / (2k) + N / 2  =  Θ(N^2)

Average cost per append (Amortized) = Total Cost / N
                                    = N / (2k) + 1 / 2  =  Θ(N) !!
```

**কেন এমনটা হলো?** কারণ প্রতিবার আপনি পুরো array-টা (যার সাইজ `N`-এর সাথে বড় হচ্ছে) কষ্ট করে copy করলেন, কিন্তু বিনিময়ে পরের resize-এর আগে কিনে নিলেন মাত্র **fixed `k`টি** সস্তা append! Array যত বড় হচ্ছে, copy খরচ বাড়ছে linearly, অথচ দম নেওয়ার সুযোগ fixed `k`-ই থেকে যাচ্ছে—তাই মোট copying cost হয়ে যায় **`Θ(N^2)`** এবং প্রতি append-এর amortized cost **`Θ(N)`**!

### Model B: Geometric Doubling (`×2`) — Aggregate Method Proof

এবার policy বদলে দিন: যখনই array ভরে যাবে, আমরা fixed `+k` না বাড়িয়ে capacity **দ্বিগুণ (`×2`)** করব!

- **Policy Assumptions:** Initial capacity `c0 = 1` (`size = 0`)। যখনই `size === capacity` অবস্থায় নতুন append আসে, capacity বাড়িয়ে `2 * capacity` করা হয়, বিদ্যমান সব `capacity`টি element copy করা হয়, এবং নতুন element write করা হয়।

প্রথমে ছোট সংখ্যায় (`N = 1` থেকে `9`) চোখে দেখুন কী ঘটে:

| Append `#i` | Append-এর আগে `(size, cap)` | Resize ঘটল? | Copy Writes | Ordinary Write | এই Op-এর মোট Cost | Append-এর পর `(size, cap)` |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `1` | `(0, 1)` | না | `0` | `1` | `1` | `(1, 1)` |
| `2` | `(1, 1)` | **হ্যাঁ** (`1 → 2`) | `1` | `1` | `2` | `(2, 2)` |
| `3` | `(2, 2)` | **হ্যাঁ** (`2 → 4`) | `2` | `1` | `3` | `(3, 4)` |
| `4` | `(3, 4)` | না | `0` | `1` | `1` | `(4, 4)` |
| `5` | `(4, 4)` | **হ্যাঁ** (`4 → 8`) | `4` | `1` | `5` | `(5, 8)` |
| `6` | `(5, 8)` | না | `0` | `1` | `1` | `(6, 8)` |
| `7` | `(6, 8)` | না | `0` | `1` | `1` | `(7, 8)` |
| `8` | `(7, 8)` | না | `0` | `1` | `1` | `(8, 8)` |
| `9` | `(8, 8)` | **হ্যাঁ** (`8 → 16`) | `8` | `1` | `9` | `(9, 16)` |

টেবিলের প্যাটার্নটা দেখেই চমকে যাবেন: resize ঘটছে কেবল যখন `size` হলো **`2`-এর ঘাত** (`1, 2, 4, 8, ...`)—অর্থাৎ প্রতিবার resize করার পর পরের resize-এর দূরত্বও **দ্বিগুণ** হয়ে যাচ্ছে!

এবার **যেকোনো `N >= 1`** সংখ্যক append-এর জন্য **Aggregate Method** দিয়ে exact proof দেখুন:

1. **Ordinary Append Writes:** প্রতিটি append নিজে নতুন element লিখতে নেয় `1` write → মোট **`N`** writes।
2. **Resize Copy Writes:** `N = 1` হলে কোনো resize-ই ঘটে না (`Total Copies = 0 < 2N`)। আর যেকোনো `N >= 2`-এর জন্য resize ঘটে কেবল তখন যখন `size` হয় `1, 2, 4, 8, ..., 2^p`, যেখানে `2^p` হলো `N`-এর চেয়ে strictly ছোট (`2^p < N`) সবচেয়ে বড় `2`-এর ঘাত।
   তাহলে মোট copy writes হলো একটি **geometric series**:
   ```text
   Total Copies = 1 + 2 + 4 + 8 + ... + 2^p
                = 2^(p + 1) - 1
   ```
   *(যদি মনে না থাকে, নিজেই যাচাই করুন: `1 + 2 + 4 + 8 = 15 = 16 - 1 = 2^4 - 1`!)*
   যেহেতু `2^p < N`, তাই `2^(p + 1) = 2 * 2^p < 2N`! অর্থাৎ:
   ```text
   Total Copies = 2^(p + 1) - 1  <  2N
   ```
3. **Total Cost (Copies + Ordinary Writes):**
   ```text
   Total Cost(N appends) = N + Total Copies
                         < N + 2N
                         = 3N   (অর্থাৎ Θ(N)!)
   ```
4. **Amortized Cost per Append:**
   ```text
   Amortized Cost = Total Cost(N appends) / N
                  < 3N / N
                  = 3  =  O(1)  (tightly Θ(1) amortized!)
   ```

এই পদ্ধতিটিরই আনুষ্ঠানিক নাম **Aggregate Method of Amortized Analysis**: পুরো `N`-operation sequence-এর **worst-case total cost `T(N)`** নির্ণয় করে তাকে `N` দিয়ে ভাগ করা (`T(N) / N`)!

### Banker's / Accounting Intuition (কেন `3` একক যথেষ্ট?)
Aggregate proof-এর পাশাপাশি **Banker's (Accounting) Method**-এর একটি অসাধারণ সহজ intuition মাথায় রাখলে কখনোই ভুলবেন না:
- কল্পনা করুন আপনি একটি ব্যালেন্স খাতা খুলে প্রতিটা `append`-এর জন্য **ঠিক `3`টি coin** charge করছেন:
  - **`1` coin** খরচ হয় ঠিক ওই মুহূর্তে নতুন element-টিকে array-তে লিখতে।
  - বাকি **`2` coin** জমা থাকে bank-এ পরের resize-এর বিলের জন্য!
- ধরুন array-টি এইমাত্র `C` থেকে `2C` হয়েছে—তাহলে এখন ভেতরে আছে `C`টি পুরোনো element আর `C`টি খালি ঘর।
- পরের `C`টি নতুন `append` এলে প্রতিটি `append` জমাল `2` coin করে → মোট জমা হলো **`2C` coin**!
- এখন array পুরো ভরে গেছে (মোট `2C`টি element: `C`টি পুরোনো + `C`টি নতুন), এবং `2C → 4C` resize-এ ঠিক **`2C`টি copy** করতে হবে। Bank-এ আপনার জমা কত? **ঠিক `2C` coin!** প্রতিটা element-এর copy খরচ আগেই পরিশোধ হয়ে গেছে—আপনার balance কখনো ঋণাত্মক হয় না!

*(Scope boundary: Formal Potential Method `Φ(D_i)` proof Phase 2-এর বাইরে; এই Aggregate proof + Accounting intuition-ই আপনার daily engineering reasoning-এর জন্য যথেষ্ট ও স্বচ্ছ।)*

---

## ৩. চরম ভুল ধারণা: Amortized ≠ Average-Case — এবং Live Latency Spikes

এখানে এসে backend engineering-এর অন্যতম জনপ্রিয় ও বিপজ্জনক বিভ্রান্তিটি একদম গোড়া থেকে উপড়ে ফেলুন। PR review বা tech interview-এ প্রায়ই শোনা যায়:
> *"আরে, `append` তো `O(1)` amortized—তার মানে average-এ প্রতিটা request `O(1)` সময়েই শেষ হবে!"*

**এই বাক্যটিতে দুটি আলাদা ভুল আছে!**

1. **Amortized ≠ Average-Case (কোনো probability নেই!):**
   - **Average-case** নির্ভর করে input-এর probability distribution-এর ওপর (কোনো বিশেষ skewed sequence এলে তা ভেঙে পড়তে পারে)।
   - **Amortized analysis**-এ কোনো randomness বা probability assumption **নেই**! এটি সম্পূর্ণ **deterministic worst-case guarantee**—অর্থাৎ `N`টি append-এর **যেকোনো** sequence-ই আসুক না কেন, মোট খরচ `< 3N`, তাই প্রতি append গড়ে `<= 3` (`O(1)` amortized)।
2. **`O(1)` Amortized ≠ Bounded Single-Request Latency!**
   - `O(1)` amortized কখনোই বলে না যে *প্রতিটি individual append* `O(1)` সময়ে শেষ হবে!
   - আমাদের doubling table-এ `#1,048,577`-তম append-এ যখন capacity `1,048,576` থেকে `2,097,152` হয়, তখন **ওই একটিমাত্র operation-এর worst-case cost হলো `Θ(N)`** (`1,048,576` element copy)!
   - এখন আমাদের opening **Checkout Tail-Latency Spike**-এর কথা ভাবুন: আগের `524,287` জন customer-এর checkout request microsecond-এ বের হয়ে গেল, কিন্তু ঠিক যেই customer-এর request-এ buffer capacity পূর্ণ হয়ে দ্বিগুণ হলো, সেই **একটি live latency-sensitive request**-কে পুরো `O(N)` copy-র pause নিজের কাঁধে নিতে হলো!
   - *(দ্রষ্টব্য: Large array resize-এর সময় memory allocation এবং পরবর্তীতে garbage collection-ও যোগ হতে পারে—তবে GC pause mechanics Phases 5 ও 11-এ, এবং p99 latency measurement Block 7 ও Phase 20-এ আসবে।)*

অর্থাৎ সহজ নিয়ম:
> **`O(1)` amortized আপনাকে দেয় একটি পুরো sequence-এর জন্য bounded total throughput work; এটি প্রতিটি individual HTTP request-এর জন্য bounded latency guarantee দেয় না!**

এবার Block 3-এর **পাঁচটি complexity lens** এক টেবিলে দেখুন:

| Analysis Category | কিসের ওপর প্রযোজ্য? | Probability / Randomness আছে? | Individual Single-Op Guarantee দেয়? |
| :--- | :--- | :---: | :---: |
| **Best-Case** | সবচেয়ে অনুকূল single input | না | কেবল ভাগ্যবান input-এ |
| **Average-Case** | Assumed input distribution | **হ্যাঁ** (input-এ) | না (skewed input-এ ভাঙতে পারে) |
| **Expected-Case** | Algorithm-এর internal random choices (যেকোনো fixed input-এ) | **হ্যাঁ** (algorithm-এ) | **না** — এটি কেবল expectation, কোনো individual-operation বা high-probability guarantee নয় |
| **Worst-Case** | সবচেয়ে প্রতিকূল single input/op | **না** | **হ্যাঁ** — প্রতিটি single op-এ hard upper bound |
| **Amortized** | যেকোনো valid sequence-এর total cost `÷ N` | **না** (100% deterministic) | **না!** Single op `O(N)` spike করতে পারে |

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-03-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> ধরুন একজন developer doubling (`×2`)-এর বদলে **tripling (`×3`)** dynamic array বানালেন: initial capacity `c0 = 1`, এবং যখনই `size === capacity` অবস্থায় append আসে, capacity `3 * capacity` করে সব `capacity` element copy করা হয়।
> 1. Aggregate Method ব্যবহার করে `N`টি append-এর মোট copy writes (`1 + 3 + 9 + ... + 3^p`, যেখানে সূত্র হলো `(3^(p + 1) - 1) / 2`) এবং মোট cost-এর upper bound derive করুন। Tripling কি এখনো `O(1)` amortized? *(Doubling-এর তুলনায় এর বাড়তি memory-waste trade-off কী?)*
> 2. Doubling-এর বদলে tripling ব্যবহার করলে কি কোনো live request-এ individual `O(N)` latency spike-এর ঝুঁকি পুরোপুরি দূর হয়? ব্যাখ্যা করুন।

এতক্ষণ আমরা শুধু `push` করে array বড় করা দেখলাম। কিন্তু বাস্তব backend buffer বা stack-এ তো data শুধু ঢোকে না—**`pop` / remove-ও হয়!** ধরুন flash sale-এ একটি buffer `1,000,000` item-এ পৌঁছেছিল, তারপর `999,990`টি item drain হয়ে গেছে। এখন array-তে মাত্র `10`টি item, অথচ memory দখল করে বসে আছে `1,048,576` slot!

স্বাভাবিকভাবেই মাথায় প্রথম যে সমাধানটা আসে তা হলো: *"যেই মুহূর্তে array অর্ধেক খালি হয়ে যাবে (`size === capacity / 2`), সাথে সাথেই capacity অর্ধেক (`capacity / 2`) করে memory ফেরত দিয়ে দিই!"*

শুনতে খুব যৌক্তিক মনে হচ্ছে—কিন্তু এই একটা innocent `if` condition আপনার পুরো **`O(1)` amortized guarantee-কে চুরমার করে `Θ(N)` per operation** বানিয়ে দিতে পারে! কীভাবে?

---

## ৪. Shrinking Hysteresis ও Resize Thrashing

আসুন সেই naive **"grow at full (`×2`), shrink immediately at half-full (`÷2`)"** policy-টিকে একটি নির্দিষ্ট অবস্থায় চালিয়ে দেখি।

ধরুন আমাদের buffer-এ এখন **`size = C` এবং `capacity = C`** (অর্থাৎ buffer ঠিক ভরা—যেমন `C = 65,536`)। এবার এই অবস্থায় একটি **alternating `push`, `pop`, `push`, `pop`, ...** workload চালান:

```text
Start: [size = C,     cap = C ]   (100% full)
         |
         |  Op 1: push  (size === cap -> GROW to 2C, copies C elements!)
         v
       [size = C + 1, cap = 2C]
         |
         |  Op 2: pop   (size becomes C === cap/2 -> SHRINK to C, copies C elements!)
         v
       [size = C,     cap = C ]   <-- একদম শুরুর boundary-তে ফিরে এলাম!!
         |
         |  Op 3: push  (আবার GROW to 2C, copies C elements!)
         v
       [size = C + 1, cap = 2C]
         |
         |  Op 4: pop   (আবার SHRINK to C, copies C elements!)
         v
       [size = C,     cap = C ]   ... অসীমভাবে চলতে থাকে!
```

দেখুন কী ভয়ংকর কাণ্ড ঘটল!
- `Op 1` (`push`): কপি করল **`C`** elements।
- `Op 2` (`pop`): কপি করল **`C`** elements।
- `Op 3` (`push`): কপি করল **`C`** elements।
- `Op 4` (`pop`): কপি করল **`C`** elements।

একেই বলে **Resize Thrashing**! মাত্র `K`টি alternating operation-এ মোট copying cost দাঁড়াল `K * C = Θ(K * N)`, অর্থাৎ **প্রতিটি single operation-এর amortized cost ভেঙে পড়ে `Θ(N)` হয়ে গেল!**

### কেন এমনটা হলো, আর **Hysteresis** কীভাবে এর সমাধান করে?

দৈনন্দিন জীবনের একটি thermostat (AC controller)-এর কথা ভাবুন। AC যদি `25°C` হলেই চালু হয় আর `25°C`-এর সামান্য নিচে নামলেই বন্ধ হয়ে যায়, তবে compressor প্রতি সেকেন্ডে ON-OFF-ON-OFF করে নষ্ট হয়ে যাবে! তাই বাস্তব thermostat-এ **দুটি আলাদা threshold** থাকে—যেমন `26°C`-এ চালু হবে, কিন্তু বন্ধ হবে `24°C`-এ নামার পর। এই **"চালু ও বন্ধ হওয়ার threshold-এর মাঝখানে ইচ্ছাকৃত ব্যবধান রাখা"**-কে engineering-এ বলে **Hysteresis**।

Naive half-full policy-র সমস্যা ছিল **grow-এর পরের state (`size ≈ cap / 2`) আর shrink-এর trigger (`size === cap / 2`)-এর মাঝে কোনো ব্যবধান ছিল না!** তাই এক ধাপ পিছিয়েই আবার resize শুরু হয়ে যেত।

সমাধান হলো **grow এবং shrink threshold-কে স্পষ্টভাবে আলাদা করা**—দুটি সুপ্রতিষ্ঠিত, সুনির্দিষ্ট policy দেখুন *(Mandatory 2: Pat Morin, Open Data Structures 2.1.2)*:

1. **Quarter-Full Policy (Classic):**
   - **Grow:** যখন `size === cap`, তখন capacity বাড়িয়ে `2 * cap` করুন (resize-এর পর occupancy = **`50%`**)।
   - **Shrink:** কেবল যখন `size <= cap / 4` (`25%` full!), তখনই capacity কমিয়ে `cap / 2` করুন (resize-এর পর occupancy আবার = **`50%`**!)।
2. **One-Third Policy (Pat Morin, *Open Data Structures* `ArrayStack` 2.1.2):**
   - **Grow:** যখন `size === cap`, তখন `resize()` নতুন array বানায় `2 * size` সাইজের (occupancy = **`50%`**)।
   - **Shrink:** কেবল যখন `cap >= 3 * size` (অর্থাৎ `size <= cap / 3`, প্রায় `33%` full!), তখনই `resize()` নতুন array বানায় `max(2 * size, 1)` সাইজের (occupancy আবার = **`50%`**!)।

এবার **পরিমাণগতভাবে (quantitatively)** দেখুন কেন এই ব্যবধান thrashing পুরোপুরি থামিয়ে দেয় (কোনো ভারী potential proof ছাড়াই!):
- **যেকোনো** resize (grow হোক বা shrink) সম্পন্ন হওয়ার সাথে সাথেই array-টি প্রায় **`50%` full** (`size ≈ cap / 2`; grow-এর পর নতুন element বসানো হলে `cap / 2 + 1`, আর shrink-এর পর `cap / 2`)।
- **Quarter-Full policy-তে:**
  - এখান থেকে আবার **grow** ট্রিগার করতে হলে আপনাকে প্রায় **`cap / 2`টি `push`** করতে হবে (`cap / 2 + 1` থেকে `cap`-এ পৌঁছাতে `cap / 2 - 1`টি `push`)।
  - আর এখান থেকে আবার **shrink** ট্রিগার করতে হলে আপনাকে অন্তত **`cap / 4`টি `pop`** করতে হবে (`cap / 2` থেকে `cap / 4`-এ নামতে)!
- **ODS One-Third policy-তে (`cap = 2 * n0`):**
  - আবার grow করতে হলে প্রায় **`n0`টি `push`**, আর shrink করতে হলে অন্তত **`n0 / 3`টি `pop`** (`n0` থেকে `2 * n0 / 3`-এ নামতে) করতে হবে!

অর্থাৎ দুটি `Θ(n)` resize-এর মাঝখানে **অবশ্যই অন্তত `Θ(n)`টি সস্তা `O(1)` operation** ঘটতে বাধ্য! *(সাবধান: যেকোনো random threshold বসালেই thrashing বন্ধ হয়ে যায় না—key invariant হলো post-resize occupancy-র সাথে grow trigger এবং shrink trigger উভয়েরই `Θ(n)` দূরত্ব থাকতে হবে।)* ফলে যেকোনো arbitrary `push`/`pop` sequence-এও amortized cost ফিরে আসে **`O(1)`**-এ, আর memory-ও কখনো `O(size)`-এর চেয়ে অসীম বড় হয়ে পড়ে থাকে না!

---

## ৫. যখন Worst-Case নিজেই অস্ত্র: Algorithmic Complexity Attacks

এতক্ষণ যে বিপর্যয়গুলো দেখলাম (sorted order export, naive shrink threshold, checkout resize spike), সেগুলোর কোনোটাতেই কেউ ইচ্ছা করে আমাদের server ভাঙার চেষ্টা করেনি।

কিন্তু production backend কখনো নিরাপদ ল্যাবরেটরিতে চলে না—এটি খোলা internet-এ untrusted input গ্রহণ করে। যদি কোনো attacker জেনে যায় যে আপনার backend-এর কোনো data structure বা parser-এর একটি **`Θ(N^2)` বা `Θ(2^N)` worst-case** আছে, তবে তাকে বিশাল botnet ভাড়া করতে হয় না! মাত্র কয়েক kilobyte-এর একটি **carefully crafted input** পাঠিয়েই সে আপনার CPU `100%` আটকে দিতে পারে।

একেই বলে **Algorithmic Complexity Attack** (বা Algorithmic Denial of Service / Algorithmic DoS)। এখানে আমরা **দুটি সম্পূর্ণ আলাদা mechanism** পরিষ্কারভাবে বুঝব এবং আলাদা রাখব।

### Mechanism 1: Hash-Table Collision Degradation (Abstract Mechanism ও ঐতিহাসিক প্রমাণ)

*(Hash table-এর পূর্ণাঙ্গ data-structure design ও comparison আমরা Block 4-এ পড়ব; এখানে কেবল attack-টির core mechanism বুঝতে যতটুকু দরকার তা দেখুন।)*

- **Conventional Hash-Table Model:** একটি hash table প্রতিটি key-কে একটি **hash function** `h(key)` দিয়ে একটি bucket index (`0 .. m - 1`)-এ ম্যাপ করে। যদি keys bucket-গুলোতে মোটামুটি সমানভাবে ছড়িয়ে পড়ে এবং load factor (`n / m`) সীমিত রাখা হয়, তবে প্রতিটি lookup বা insert-এর কাজ হলো **expected `O(1)`**।
- **যখন Hash Function-টি Deterministic ও Predictable:** ধরুন একটি সহজ, non-randomized hash table (যেমন কোনো custom table বা পুরোনো runtime-এর deterministic hash function) untrusted HTTP query parameters (`?k1=1&k2=1&...&kN=1`) key হিসেবে store করছে।
  - **Explicit Collision-Resolution Cost Model (আগে assumption স্পষ্ট করুন!):** সব hash-table collision strategy-তে insertion cost হুবহু এক রকম হয় না। উদাহরণস্বরূপ, কোনো separate-chaining table যদি duplicate check ছাড়াই কেবল bucket-এর মাথায় নতুন entry prepend করে দেয়, তবে insertion প্রতিবার `O(1)` হতে পারে (যদিও পরে ওই bucket-এ lookup করলে `O(N)` লাগবে)। এখানে আমরা একটি সাধারণ map/dictionary model ধরছি: **separate chaining যেখানে প্রতিটি insert-এর আগে duplicate key যাচাই করতে ওই bucket-এর বিদ্যমান chain পুরোটা scan করা হয়** *(একইভাবে open addressing with linear probing-এও একই probe cluster-এ পরপর colliding key বসাতে গেলে ক্রমবর্ধমান probe chain scan করতে হয়)*।
  - যেহেতু hash function-টি public ও deterministic, attacker আগে থেকেই নিজের machine-এ বসে **`N`টি সম্পূর্ণ ভিন্ন key** তৈরি করতে পারে যাদের **সবার hash মান হুবহু একই bucket-এ পড়ে (hash collision)**!
  - এবার server যখন আমাদের duplicate-checking chaining model-এ ওই `N`টি colliding key এক-এক করে insert করে:
    - ১ম key → bucket খালি (`0` comparison); ২য় key → `1` comparison; ৩য় key → `2` comparison; ...; `N`-তম key → `N - 1` comparison!
  - **মোট কাজ:** `0 + 1 + 2 + ... + (N - 1) = N(N - 1) / 2 = Θ(N^2)`!
  - উদাহরণ হিসেবে `N = 20,000`টি colliding key এই model-এ insert করতেই প্রায় `20,000 × 19,999 / 2 ≈ 200,000,000` comparison লেগে যেতে পারে—যা একটি single request-এর ভেতর server CPU-র ওপর মারাত্মক চাপ তৈরি করতে যথেষ্ট!
- **ঐতিহাসিক প্রমাণ (Mandatory 3):** **Scott A. Crosby & Dan S. Wallach (USENIX Security 2003)** তাঁদের যুগান্তকারী পেপার *"Denial of Service via Algorithmic Complexity Attacks"*-এ বাস্তবে দেখিয়েছিলেন যে ওই সময়ের নির্দিষ্ট কিছু non-randomized hash-table implementation (যেমন Perl, Squid, Bro) কীভাবে predictable hash collision-এ `O(N^2)` work-এর মুখে পড়ে অচল হয়ে যেত।
- **অত্যন্ত জরুরি সীমারেখা (Modern JS `Map` / `Set` নিয়ে ভুল দাবি করবেন না!):** Crosby & Wallach (2003) প্রমাণ করে **abstract non-randomized hash-table collision mechanism** এবং সেই সময়ের নির্দিষ্ট vulnerable implementation-গুলোর ঝুঁকি। এই ঐতিহাসিক গবেষণা **কখনোই প্রমাণ করে না যে আধুনিক native JavaScript `Map` বা `Set`-এ হুবহু একই vulnerability আছে!** *(Node.js/V8-এর `Map`/`Set` internal hashing implementation Phase 5-এ পড়া হবে।)*
- **Mitigation Concept (`Awareness` মাত্র):** কিছু আধুনিক runtime ও library **secret keyed / randomized hashing** (যার একটি পরিচিত `Awareness`-level উদাহরণ হলো **SipHash**, যেখানে process-এর জন্য একটি গোপন random seed ব্যবহার করা হয়—যদিও সব আধুনিক runtime একই strategy বা SipHash ব্যবহার করে না) অবলম্বন করে। এর মূল ধারণা হলো: secret seed জানা না থাকলে বাইরে থাকা attacker-এর জন্য আগে থেকে **precomputed** collision set তৈরি করা অনেক কঠিন হয়ে পড়ে, এবং একটি **stated threat model**-এর অধীনে (যেখানে seed গোপন থাকে) ওই secret seed-এর ওপর expected behavior বিশ্লেষণ করা যায়। তবে এটি কোনো **unconditional guarantee** নয়—প্রতিটি implementation বা যেকোনো adaptive attacker (যেমন timing/side-channel probing বা seed ফাঁস হওয়ার ঝুঁকি)-এর বিরুদ্ধে তা স্বয়ংক্রিয়ভাবে অজেয় নয়। *(Cryptographic hash math ও production hardening Phase 19-এ।)*

### Mechanism 2: Catastrophic Regular-Expression Backtracking (ReDoS)

এবার দ্বিতীয় mechanism-টি দেখুন—**ReDoS (Regular Expression Denial of Service)**। পরিষ্কার রাখুন: **এটি hash-table collision থেকে সম্পূর্ণ আলাদা একটি algorithmic mechanism!** এর সাথে hashing বা bucket-এর কোনো সম্পর্ক নেই।

অনেক সাধারণ regex engine (যেমন JavaScript-এর built-in backtracking regex matcher) একটি string-কে বাম থেকে ডানে match করার চেষ্টা করে। যদি pattern-এ **অস্পষ্ট, পুনরাবৃত্ত (ambiguous / nested) quantifiers** থাকে—যেমন `/^(a+)+$/` বা `/^([a-z0-9]+)+$/`—তখন কী ঘটে দেখুন:

- **সাধারণ valid input-এ (`"aaaaaaaaaa"`, `n = 10`):** Engine প্রথম চেষ্টাতেই পুরো `10`টা `'a'` একসাথে ধরে match করে বের হয়ে যায় → **`O(n)` (চোখের পলকে!)**।
- **Crafted near-match input-এ (`"aaaa!"`, যেখানে শেষে একটি invalid `'!'` আছে):**
  - প্রথমে engine চারটা `'a'` একসাথে নেয়: `(aaaa)` → পরের অক্ষর `'!'`, match fail!
  - হাল না ছেড়ে engine **backtrack** করে (পিছিয়ে আসে) এবং চারটা `'a'`-কে অন্যভাবে ভাগ করে চেষ্টা করে:
    1. `(aaaa)` → fail on `'!'`
    2. `(aaa)(a)` → fail on `'!'`
    3. `(aa)(aa)` → fail on `'!'`
    4. `(aa)(a)(a)` → fail on `'!'`
    5. `(a)(aaa)` → fail on `'!'`
    6. `(a)(aa)(a)` → fail on `'!'`
    7. `(a)(a)(aa)` → fail on `'!'`
    8. `(a)(a)(a)(a)` → fail on `'!'`
  - মাত্র `n = 4`টি `'a'`-এর জন্য **`2^(4 - 1) = 8`**টি ভিন্ন partition পরীক্ষা করতে হলো!
  - সাধারণভাবে `n`টি `'a'`-এর পর একটি `'!'` থাকলে nested `(a+)+` pattern-টি **`2^(n - 1) = Θ(2^n)`**টি উপায় চেষ্টা করে *সবগুলোই ব্যর্থ হওয়ার পর* কেবল `false` ফেরত দেয়! *(Pattern-এর ধরন অনুযায়ী ReDoS `Θ(N^2)` polynomial বা `Θ(2^N)` exponential দুটোই হতে পারে।)*

একটি ছোট, নিরাপদ Node.js উদাহরণে এই growth-টা দেখুন *(সাবধান: নিজের machine-এ পরীক্ষা করলে কখনোই `n`-এর মান ইচ্ছেমতো বড় করবেন না—যেমন `n = 40` দেবেন না; ছোট, bounded মান দিয়েই exponential growth-এর আচরণ নিরাপদে দেখা যায়!)*:

```javascript
// Safe, strictly bounded illustration of catastrophic backtracking (DO NOT increase n arbitrarily)
const ambiguousPromoPattern = /^([a-z]+)+$/;

function demoBacktrackingGrowth() {
  for (const n of [10, 14, 18, 22]) {
    const nearMatch = "a".repeat(n) + "!";
    const t0 = performance.now();
    const isValid = ambiguousPromoPattern.test(nearMatch);
    const elapsedMs = (performance.now() - t0).toFixed(3);
    console.log({ n, isValid, elapsedMs });
  }
}
```

এই illustrative backtracking model-এ `'a'`-এর সংখ্যা মাত্র `1` বাড়লেই সম্ভাব্য partition-এর সংখ্যা দ্বিগুণ (`2^(n - 1)`) হয়ে যায়—ফলে খুব ছোট একটি near-match string-ও exponential সংখ্যক backtracking path তৈরি করে। *(গুরুত্বপূর্ণ সতর্কতা: বাস্তবে ঠিক কত milliseconds বা seconds লাগবে তা regex engine, runtime version, internal optimizations, এবং hardware-এর ওপর নির্ভর করে—কেবল asymptotic partition count থেকে নির্দিষ্ট wall-clock runtime অনুমান করা যায় না। তবু exponential growth-এর কারণেই ইচ্ছেমতো বড় `n` দিলে একটি backtracking matcher মারাত্মকভাবে আটকে যেতে পারে, তাই demo-তে `n` কখনোই নির্বিচারে বাড়াবেন না!)*

*(সংক্ষিপ্ত production প্রসঙ্গ: ঠিক এই catastrophic backtracking mechanism-ই **Cloudflare-এর July 2, 2019 global outage**-এ একটি অস্পষ্ট WAF regular expression-এর মাধ্যমে বিশ্বজুড়ে server CPU `100%` করে দিয়েছিল। সেই বাস্তব postmortem-টি আমরা Block 6-এ বিস্তারিত পড়ব; এখানে মূল focus হলো ReDoS-এর worst-case complexity mechanism বোঝা।)*

- **Mitigation Concepts (`Awareness` মাত্র — production security defense ও WAF tuning Phase 19-এ):**
  - Ambiguous nested quantifiers (যেমন `(a+)+`, `(a|aa)+`, `.*.*`) এড়িয়ে সহজ, unambiguous pattern লেখা,
  - Regex চালানোর আগে input length-এ কঠোর সীমা (`maxLength`) বসানো,
  - এবং যেখানে untrusted regex বা strict SLA দরকার, সেখানে backtracking-এর বদলে **linear-time (`O(n)`) regex matching** approach-এর অস্তিত্ব জানা *(regex engine internals ও production hardening এই block-এর বাইরে)*।

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-03-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> দুটি আলাদা API endpoint-এ একটিমাত্র crafted HTTP request-এ backend CPU মারাত্মকভাবে আটকে গেছে:
> - **Endpoint A:** Untrusted JSON payload-এর keys একটি custom in-memory separate-chaining table-এ রাখে, যার hash function হলো `sumOfCharCodes(key) % bucketCount` এবং প্রতিটি insert-এর আগে duplicate key যাচাই করতে ওই bucket-এর বিদ্যমান chain scan করে।
> - **Endpoint B:** User-এর পাঠানো tag string যাচাই করে `/^([a-z0-9]+)+$/` দিয়ে; একজন caller পাঠিয়েছে `30`টি `'a'`-এর শেষে একটি `'!'`।
> 1. Endpoint A এবং Endpoint B-র পেছনে থাকা **দুটি আলাদা algorithmic mechanism** এবং তাদের worst-case growth class (`Θ(N^2)` বনাম `Θ(2^n)`) চিহ্নিত করুন। কেন এরা দুটি পৃথক failure mechanism?
> 2. কেউ যদি বলে *"Crosby & Wallach (2003) প্রমাণ করেছে সব hash table-ই vulnerable, তাই Node.js-এর built-in `Map`-ও Endpoint A-র মতোই সহজে ভেঙে পড়বে"*—এই যুক্তির ঠিক কোথায় technical error রয়েছে?

---

## Block 4-এর দিকে সেতুবন্ধন: কোন Workload-এ কোন Data Structure?

Block 3-এ আপনি জটিল backend performance reasoning-এর একটি বড় দেয়াল পেরোলেন:
- Best-case, distributional average-case, randomized expected-case, worst-case, এবং deterministic amortized cost-কে আলাদা করা;
- Aggregate Method ও Banker's intuition দিয়ে dynamic-array doubling (`O(1)` amortized) বনাম fixed-increment `+k` (`Θ(N)` amortized) prove করা, এবং বোঝা কেন `O(1)` amortized হওয়া সত্ত্বেও single-request `O(N)` tail-latency spike ঘটে;
- Shrinking hysteresis দিয়ে resize thrashing ঠেকানো; এবং
- Worst-case complexity কীভাবে hash-collision degradation ও ReDoS আকারে production availability ভেঙে দেয় তা বোঝা।

এবার পরের বাস্তব engineering প্রশ্নটা সামনে দাঁড়ায়:
> *আমরা দেখলাম dynamic array-র শেষে append করা `O(1)` amortized, কিন্তু unsorted array-তে খোঁজা এখনো `O(N)`—আবার hash table exact lookup-এ expected `O(1)` দিলেও ordered range query ("`USD 10` থেকে `USD 25`-এর সব product") করতে পারে না! তাহলে backend-এর ভিন্ন ভিন্ন workload-এর জন্য contiguous array, linked list, hash table, sorted array, balanced search tree, binary heap, এবং sorting algorithm-গুলোর মধ্যে কীভাবে সঠিক data structure বেছে নেব—আর প্রতিটি পছন্দে time বনাম space-এর exact trade-off কী?*

ঠিক এই প্রশ্নের উত্তর নিয়েই আসছে **Phase 2, Block 4 — Core Data Structures, Hashing, Searching, and Sorting Trade-Offs**!

---

## Study Plan, Learning Objectives, Terminology & Resources

> **Continuity & Ownership Notes:** আপনার সব Pause & Checks, দুটি Mandatory Exercise, এবং (যদি করতে চান) Optional Closed-Book Assessment-এর উত্তর স্বাধীনভাবে `phases/phase-02/notes/block-03-notes.md` ফাইলে লিখুন। AI-generated material কখনো completed learning বা learner study hours হিসেবে গণ্য হয় না।
>
> **Progression & Verification Policy:** এই block-এর Closed-Book Assessment সম্পূর্ণ **optional** এবং Block 4 শুরু করার জন্য mandatory gate নয়। পড়া ও দুটি mandatory exercise শেষ করে assessment skip করলে এই block-টির status হবে **`Studied — mastery not formally verified`**; যথেষ্ট evidence share করে ChatGPT evaluation সম্পন্ন হলে তা **`Mastery verified`** হিসেবে গণ্য হবে।

### Study Time Reference

*প্রতিটি activity-র বিস্তারিত time breakdown দেখতে এই নথির শুরুর **Study Time Guide** দেখুন। Approved dossier-এ Block 3-এর roadmap allocation হলো `3 hours (180 min)`—যা একটি flexible planning reference, কোনো strict limit নয়।*

### Block 3 Learning Objectives *(Primary Target Level: `Working Understanding` | Supported Phase 2 Outcome: Outcome 4)*

- **LO 3.1 (`Working Understanding` | Bloom: *Analyze*):** Contrast **best-case**, **average-case** (distributional assumption over inputs), **expected-case** (internal algorithm randomization), and **worst-case** complexity, explaining why backend guarantees cannot rely on best-case or unjustified average assumptions.
- **LO 3.2 (`Working Understanding` | Bloom: *Apply / Analyze*):** Use the **Aggregate Method** (supported by intuitive accounting/banker's intuition) to prove why **dynamic-array doubling** achieves **`O(1)` amortized** append cost over any sequence of `N` appends, while **fixed-increment `+k` resizing** degrades to **`Θ(N^2)` total copying** and **`Θ(N)` amortized copying per append** (and explain how shrinking hysteresis prevents resize thrashing).
- **LO 3.3 (`Working Understanding` | Bloom: *Evaluate*):** Explain precisely why **amortized complexity is not average-case complexity**, and why an **`O(1)` amortized** operation may still trigger an individual `O(N)` latency spike in a live latency-sensitive request.
- **LO 3.4 (`Working Understanding` | Bloom: *Analyze*):** Explain the abstract mechanism of **algorithmic-complexity attacks**, distinguishing historical **hash-table collision degradation** (Crosby & Wallach, USENIX Security 2003, explicitly separated from modern native JS `Map`/`Set` runtime implementations) from **catastrophic regular-expression backtracking (ReDoS)** (`Awareness` of high-level mitigations; production defenses deferred to Phase 19).

### Terminology Inventory

| Term | Official Target Level | Why It Matters in Block 3 |
| :--- | :---: | :--- |
| **Best-Case vs. Average-Case vs. Worst-Case** | `Working Understanding` | Prevents trusting happy-path speed or fragile input assumptions when skewed or adversarial traffic arrives. |
| **Expected-Case (Randomized) vs. Distributional Average** | `Working Understanding` | Separates algorithm-controlled internal randomness from assumptions about user inputs. |
| **Amortized Analysis (Aggregate Method & Banker's Intuition)** | `Working Understanding` | Deterministic worst-case average per operation across *any* sequence—zero probability assumptions needed. |
| **Amortized Spike / Tail-Latency Hazard** | `Working Understanding` | Clarifies why `O(1)` amortized bounds total sequence work, not single-request worst-case HTTP latency. |
| **Shrinking Hysteresis & Resize Thrashing** | `Working Understanding` | Explains why separated grow/shrink thresholds (`cap/3` or `cap/4`) prevent `Θ(N)` per-op oscillation. |
| **Algorithmic Complexity Attack (Hash Collisions & ReDoS)** | `Working Understanding` *(mechanism)* / `Awareness` *(mitigations)* | Connects worst-case complexity to real CPU-exhaustion DoS risks while keeping the two mechanisms and runtime boundaries distinct. |
| **Seeded / Randomized Hashing (e.g., SipHash) & Linear-Time Regex** | `Awareness` | High-level mitigation concepts; cryptographic math, V8 internals, and WAF hardening are deferred to Phases 5 and 19. |

### Mandatory Authoritative Resources *(Targeted Reading — ~55–65 min total)*

1. **Mandatory 1 (~25 min — assigned sections only):** MIT OpenCourseWare 6.006 *Introduction to Algorithms* (Spring 2020) — [Lecture 2: Data Structures](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/mit6_006s20_lec2/) (`LO 3.2`, `LO 3.3`):
   - **Assigned scope:** Lecture 2 Notes PDF — *Sequence Interface*, *Static vs. Dynamic Arrays*, এবং table doubling-এর *Amortized Analysis*।
   - **What to extract:** Static array বনাম dynamic array-র গঠন এবং doubling-এর Aggregate Method প্রমাণ।
2. **Mandatory 2 (~15–20 min — assigned subsections only):** Pat Morin, *Open Data Structures* — [Section 2.1 — ArrayStack](https://opendatastructures.org/ods-python/2_1_ArrayStack_Fast_Stack_O.html) (`LO 3.2`):
   - **Assigned scope:** কেবল Subsections **`2.1.2 Growing and Shrinking`** এবং **`2.1.3 Summary`**।
   - **What to extract:** `cap >= 3 * n`-এ shrink করার hysteresis নীতি এবং কেন `n / 2`-এ সাথে সাথে shrink করলে thrashing ঘটে।
3. **Mandatory 3 (~15–20 min — assigned sections only):** Scott A. Crosby & Dan S. Wallach (USENIX Security 2003) — [Denial of Service via Algorithmic Complexity Attacks](https://www.usenix.org/conference/12th-usenix-security-symposium/denial-service-algorithmic-complexity-attacks) (`LO 3.1`, `LO 3.4`):
   - **Assigned scope:** *Abstract*, *Section 1 (Introduction)*, এবং hash-table collision attack-এর conceptual overview। **Perl/Squid/Bro C-code exploit internals বাদ দিন।**
   - **What to extract:** ঐতিহাসিক non-randomized hash-table implementation-এ কীভাবে predictable collision worst-case `O(N^2)` CPU exhaustion ঘটায় *(এটি আধুনিক native JS `Map`/`Set`-এর vulnerability প্রমাণ নয়—তা খেয়াল রাখুন)*।

### Optional Supplementary Resources *(Outside Mandatory Budget — Do Not Treat as Mandatory)*

- **Supplementary (Commercial Book):** Thomas H. Cormen et al. (CLRS), *Introduction to Algorithms* (4th ed., MIT Press, 2022) — Chapter 17 (*Amortized Analysis* — `17.1 Aggregate analysis`, `17.2 Accounting method`, `17.4 Dynamic tables`)।

---

## Practical Exercises *(Mandatory — ~50–60 min total)*

> **Ownership Rule:** উভয় exercise-এর সব calculation, derivation, এবং risk analysis স্বাধীনভাবে `phases/phase-02/notes/block-03-notes.md` ফাইলে লিখুন। এখানে কোনো answer key দেওয়া হয়নি।

### Exercise 1: Dynamic Buffer Resizing Analysis *(Mandatory — ~25–30 min | `LO 3.2`, `LO 3.3`)*

একটি Node.js batch-ingestion buffer প্রথমে খালি অবস্থায় (`size = 0`) **initial capacity `10`** নিয়ে শুরু হয় এবং তাতে **`N = 100,000`**টি sequential event append করতে হবে।

**Explicit Cost Model & Assumptions (যাতে প্রতিটি হিসাব 100% determinate হয়):**
- **Grow trigger:** কোনো `append` আসার ঠিক আগে যদি `size === capacity` হয়, তবে policy অনুযায়ী নতুন array allocate করে বিদ্যমান সব `size`টি element নতুন array-তে copy করা হয় (`size`টি copy write), এবং তারপর নতুন element-টি লেখা হয় (`1`টি ordinary write)।
- **Operation cost:** `Total Cost = Total Element Copies + Total Operations` (প্রতিটি ordinary `append` বা `pop` = `1` unit; resize-এর প্রতিটি element copy = `1` unit)।

নিচের **তিনটি resizing policy** Aggregate Method দিয়ে গাণিতিকভাবে তুলনা করুন:

1. **Policy 1 (`+100` Fixed Increment):** Initial capacity `10` থেকে শুরু করে `N = 100,000`টি `append` চালান। প্রতিবার `size === capacity` হলে নতুন capacity = `capacity + 100`।
   - বের করুন: মোট কতবার resize ঘটে, প্রতিটি resize-এ কপি হওয়া সংখ্যার ধারার যোগফল (**total element copies**), **total cost**, **amortized cost per append**, এবং **peak single-operation copying cost**।
2. **Policy 2 (`×2` Geometric Doubling):** Initial capacity `10` থেকে শুরু করে `N = 100,000`টি `append` চালান। প্রতিবার `size === capacity` হলে নতুন capacity = `2 * capacity`।
   - বের করুন: Resize-গুলো ঠিক কোন কোন capacity-তে ঘটে, মোট কতবার resize ঘটে, **total element copies** (exact geometric sum সহ), **total cost**, **amortized cost per append**, এবং **peak single-operation copying cost**।
   - Policy 1 ও Policy 2-এর **total copies** এবং **peak single-operation copy spike** পাশাপাশি তুলনা করে ব্যাখ্যা করুন: Policy 2-এর amortized cost অনেক কম হওয়া সত্ত্বেও কেন এর single-operation latency spike-কে উপেক্ষা করা যায় না?
3. **Policy 3 (`×2` Grow / Immediate `÷2` Shrink at Half-Full — Thrashing Workload):**
   - Policy 2-এর মতোই initial capacity `10` থেকে প্রথমে `81,920`টি event append করে buffer-কে ঠিক পূর্ণ boundary অবস্থায় (`size = 81,920`, `capacity = 81,920`) আনুন।
   - এখন এই অবস্থা থেকে **`K = 1,000`টি alternating operation** (`push`, `pop`, `push`, `pop`, ... — মোট `500`টি `push` এবং `500`টি `pop`) চালান, যেখানে:
     - `push`-এর আগে `size === capacity` হলে capacity বেড়ে `2 * capacity` হয় (`size` copies);
     - `pop` শেষ element সরায় (`1` unit), এবং ঠিক তার পরপরই `size === capacity / 2` হলে capacity কমে `capacity / 2` হয় (`size` copies)।
   - এই `1,000`টি alternating operation-এর জন্য বের করুন: **total element copies**, **total cost**, **amortized cost per operation**, এবং কেন এই নীতি thrashing ঘটায়।
   - **Follow-up Design Question:** যদি shrink threshold-কে immediate `capacity / 2`-এর বদলে `size <= capacity / 4` (কমে `capacity / 2`) অথবা Pat Morin-এর `capacity >= 3 * size` (কমে `2 * size`) নীতিতে বদলানো হয়, তবে `(size = 81,920, capacity = 163,840)` অবস্থা থেকে প্রথম shrink ট্রিগার হতে ন্যূনতম কতগুলো `pop` লাগবে—এবং তাতে thrashing কীভাবে থেমে যায়?

---

### Exercise 2: Case-Analysis & Production Risk Audit *(Mandatory — ~25–30 min | `LO 3.1`, `LO 3.3`, `LO 3.4`)*

নিচের **৪টি বাস্তব backend engineering scenario** গভীরভাবে audit করুন। প্রতিটির জন্য লিখুন:
- **(ক)** ঠিক কোন **case-analysis বা complexity reasoning error** এখানে করা হয়েছে?
- **(খ)** ঠিক কোন **input বা operation sequence** worst-case behavior ট্রিগার করবে এবং তার complexity class কী?
- **(গ)** Live production backend-এ এর **বাস্তব operational failure mode** কী হবে?
- **(ঘ)** Block 3-এর conceptual স্তরে এর **সঠিক engineering direction / mitigation** কী? *(Phase 5 V8 internals বা Phase 19 WAF config লেখার দরকার নেই।)*

1. **Scenario 1 — Untrusted HTTP Query Parameters in a Naive Hash Table:** একটি internal proxy service untrusted HTTP query parameters store করার জন্য একটি custom, non-randomized separate-chaining hash table ব্যবহার করে (যা প্রতিটি insert-এর আগে duplicate key যাচাই করতে bucket chain scan করে)। Developer দাবি করছেন: *"Hash table তো `O(1)`, তাই query parameters যত বড়ই হোক CPU safe!"* *(আপনার উত্তরে স্পষ্টভাবে দেখান কেন এই abstract/historical collision mechanism-কে সরাসরি আধুনিক runtime-managed JavaScript `Map`/`Set`-এর সাথে এক করে ফেলা ভুল।)*
2. **Scenario 2 — Checkout Buffer Doubling at `1,048,576` Items:** একটি latency-critical checkout service প্রতিটি order event-কে একটি shared in-memory dynamic buffer-এ append করে। Capacity `1,048,576` পূর্ণ হতেই একটি live checkout request-এর ভেতর buffer দ্বিগুণ (`2,097,152`) হয়। টিম অবাক হয়ে বলছে: *"আমরা তো `O(1)` amortized buffer ব্যবহার করেছি—তাহলে checkout request-এ হঠাৎ latency spike এল কেন?"*
3. **Scenario 3 — First-Pivot Deterministic Quicksort on Order Exports:** আমাদের reporting worker প্রতিদিন database থেকে `createdAt` timestamp অনুযায়ী sorted order exports এনে custom deterministic Quicksort (যা সবসময় প্রথম element-কে pivot নেয়) দিয়ে পুনরায় সাজায়। PR-এ লেখা ছিল: *"Quicksort average-case `O(N log N)`, তাই `200,000` rows নিয়ে কোনো চিন্তা নেই।"*
4. **Scenario 4 — Confusing Amortized with Average-Case:** Design review-এ একজন engineer মন্তব্য করলেন: *"আমাদের data structure-টির operation `O(1)` amortized—এর মানে হলো random inputs দিলে average-এ প্রতিটা operation `O(1)` সময়ে শেষ হয়, আর কোনো request কখনোই `O(N)` সময় নেবে না।"* *(এই একটি বাক্যের ভেতর লুকানো সব কয়টি আলাদা reasoning error শনাক্ত করে সঠিকভাবে rewrite করুন।)*

---

## Closed-Book Synthesis Assessment *(Optional — ~20–25 min)*

এই assessment-টি সম্পূর্ণ **optional**—Block 4 শুরু করার জন্য এটি সম্পন্ন করা বা ChatGPT-র approval নেওয়া mandatory নয়। আপনি চাইলে:
- এই lesson ও সব browser tab বন্ধ রেখে নিচের ৫টি প্রশ্নের নিজের ভাষায় উত্তর `phases/phase-02/notes/block-03-notes.md` ফাইলে লিখে formal evaluation-এর জন্য **ChatGPT**-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে);
- অথবা আপনি যথেষ্ট বুঝেছেন মনে করলে assessment skip করে Block 4-এ এগোতে পারেন (`Studied — mastery not formally verified`);
- অথবা assessment না করেই যেকোনো unclear concept নিয়ে প্রশ্ন ও আলোচনা করতে পারেন।

*(Note: `~20–25 min` কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি approximate estimate—কোনো strict time limit নয়।)*

1. **(LO 3.1 / Outcome 4)** Best-case, distributional average-case, randomized expected-case, এবং worst-case complexity-র মধ্যে পার্থক্য ব্যাখ্যা করুন। কেন Big-O (`O(...)`) নিজে থেকে worst-case বোঝায় না, কেন distributional average-case-এর চেয়ে randomized expected-case ও worst-case বেশি শক্তিশালী—এবং কেন কেবল randomized expected-case bound একাই প্রতি single run-এর hard latency বা high-probability tail guarantee দেয় না?
2. **(LO 3.2 / Outcome 4)** Initial capacity `1` ধরে **Aggregate Method** দিয়ে সম্পূর্ণ step-by-step prove করুন কেন `×2` dynamic-array doubling `N`টি append-এ `O(1)` amortized cost দেয়, আর `+k` fixed-increment growth কেন `Θ(N^2)` total copying এবং `Θ(N)` amortized cost-এ নেমে যায়। পাশাপাশি Banker's/Accounting intuition-টি সংক্ষেপে ব্যাখ্যা করুন।
3. **(LO 3.2 / Outcome 4)** কেন `size === capacity`-এ `×2` grow করে `size === capacity / 2`-এ সাথে সাথে `÷2` shrink করলে resize thrashing ঘটে? Separated shrink threshold (`cap / 4` বা Pat Morin-এর `cap / 3` policy) কীভাবে hysteresis তৈরি করে `O(1)` amortized cost ফিরিয়ে আনে?
4. **(LO 3.3 / Outcome 4)** সুস্পষ্টভাবে ব্যাখ্যা করুন কেন **Amortized ≠ Average-Case**, এবং কেন একটি **`O(1)` amortized** operation live latency-sensitive backend request-এ এখনো individual `O(N)` tail-latency spike তৈরি করতে পারে।
5. **(LO 3.4 / Outcome 4)** Hash-table collision degradation এবং catastrophic regex backtracking (ReDoS)-এর abstract complexity mechanism দুটি আলাদাভাবে তুলনা করুন। Crosby & Wallach (2003) ঠিক কী প্রমাণ করে এবং কী প্রমাণ *করে না* (বিশেষত আধুনিক native JavaScript `Map`/`Set` প্রসঙ্গে)?

### Block 3 Readiness & Progression Checklist

- [ ] Guided lesson মনোযোগ দিয়ে পড়েছেন এবং Pause & Checks 1–3 `phases/phase-02/notes/block-03-notes.md`-এ লিখে (পরে Self-Check Guidance মিলিয়ে) সম্পন্ন করেছেন।
- [ ] তিনটি Mandatory Reading (**MIT 6.006 Lecture 2**, **Open Data Structures `2.1.2–2.1.3`**, এবং **Crosby & Wallach USENIX Security 2003** নির্ধারিত অংশ) সম্পন্ন করেছেন।
- [ ] `phases/phase-02/notes/block-03-notes.md` ফাইলে স্বাধীনভাবে **Exercise 1 (Dynamic Buffer Resizing Analysis)** এবং **Exercise 2 (Case-Analysis & Production Risk Audit)** সম্পন্ন করেছেন।
- [ ] *(Optional)* Notes/tabs বন্ধ রেখে Closed-Book Synthesis Assessment Questions 1–5 লিখেছেন এবং formal evaluation-এর জন্য ChatGPT-র কাছে share করেছেন।

> **Progression Status Distinction:** Block 4 কখন শুরু করবেন তা আপনার সিদ্ধান্ত। পড়া ও দুটি mandatory exercise শেষ করে optional assessment ছাড়া এগোলে এই block-টির status হবে **`Studied — mastery not formally verified`**। যথেষ্ট evidence share করে ChatGPT evaluation সম্পন্ন হলেই তা **`Mastery verified`** হিসেবে গণ্য হবে।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`phases/phase-02/notes/block-03-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের guidance দেখুন। Exercise 1, Exercise 2, বা Optional Closed-Book Assessment-এর কোনো solution এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও (চাইলে) optional ChatGPT review-এর জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* Approach A-র `Θ(N log N)` গড় performance নির্ভর করে **input-গুলো uniformly random** হওয়ার assumption-এর ওপর। কিন্তু upstream partner যখন newest-first (reverse-sorted) batch পাঠায়, তখন সেই distributional assumption ভেঙে পড়ে এবং Approach A সরাসরি তার **`Θ(N^2)` worst-case**-এ ঢুকে পড়ে—ফলে production batch timeout বা CPU stall হয়।
  2. *শক্তিশালী উত্তরে থাকবে:* Approach A-র **average-case** হলো input-এর assumed probability distribution-এর ওপর গড় (যা user/upstream data-র হাতে ভঙ্গুর)। বিপরীতে Approach B-র **expected-case** হলো algorithm-টির নিজের internal random choices-এর ওপর গড়—যা **যেকোনো fixed input**-এর (এমনকি reverse-sorted বা adversary-crafted input-এর) জন্যই `Θ(N log N)` **expected** cost বজায় রাখে *(যদিও কেবল expectation একাই প্রতি single run-এর hard latency বা high-probability tail guarantee নয়—সেজন্য আলাদা tail analysis প্রয়োজন)*। আর `O(...)` কেবল একটি asymptotic **upper bound** নির্দেশ করে—এটি best, average, expected, নাকি worst case-এর ওপর প্রয়োগ করা হয়েছে তা আলাদাভাবে উল্লেখ না করলে কিছুই স্পষ্ট হয় না।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* `N`টি append-এ resize ঘটে যখন `size` হলো `1, 3, 9, ..., 3^p` (যেখানে `3^p < N`)। মোট copies = `1 + 3 + 9 + ... + 3^p = (3^(p + 1) - 1) / 2`। যেহেতু `3^p < N`, তাই `3^(p + 1) = 3 * 3^p < 3N`, অর্থাৎ `Total Copies < 1.5N`! এর সাথে `N`টি ordinary write যোগ করলে `Total Cost < 2.5N` → **Amortized cost = `Total Cost / N < 2.5 = O(1)`**! *(Trade-off: doubling-এ সর্বোচ্চ প্রায় `50%` slot খালি থাকতে পারে, কিন্তু tripling-এ resize-এর পরপরই প্রায় `67%` (`2/3`) memory খালি পড়ে থাকে—অর্থাৎ কিছুটা কম amortized copying-এর বদলে বেশি peak memory overhead!)*
  2. *শক্তিশালী উত্তরে থাকবে:* **না!** Tripling resize-এর সংখ্যা সামান্য কমালেও যেকোনো একটি resize যখন ঘটে (যেমন `size = 3^p`), তখন সেই **একটিমাত্র live append request-কে পুরো `3^p = Θ(N)` element copy** করতেই হয়—তাই individual `O(N)` tail-latency spike-এর ঝুঁকি এখনো রয়ে যায়।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* **Endpoint A** আক্রান্ত হয়েছে **Hash-Table Collision Degradation** দ্বারা: deterministic ও predictable `sumOfCharCodes % bucketCount` hash function-এর কারণে attacker `N`টি ভিন্ন key একই bucket-এ ফেলে এবং duplicate-checking bucket scan-এর কারণে `0 + 1 + ... + (N - 1) = Θ(N^2)` comparison ঘটায়। **Endpoint B** আক্রান্ত হয়েছে **Catastrophic Regular-Expression Backtracking (ReDoS)** দ্বারা: ambiguous nested quantifier `([a-z0-9]+)+` শেষে invalid `'!'` দেখে `30`টি `'a'`-কে `2^(30 - 1) = Θ(2^n)` ভিন্নভাবে partition করে backtrack করে। এরা দুটি সম্পূর্ণ আলাদা mechanism—একটি deterministic bucket collision (`Θ(N^2)`), অন্যটি ambiguous pattern-এর exponential search tree (`Θ(2^n)`)।
  2. *শক্তিশালী উত্তরে থাকবে:* Crosby & Wallach (2003) দেখিয়েছিল নির্দিষ্ট কিছু ২০০৩-এর non-randomized hash-table implementation-এ collision degradation কীভাবে ঘটে। এটি আধুনিক runtime-managed JavaScript `Map`/`Set`-এর internal hashing implementation-কে পরীক্ষা বা vulnerable প্রমাণ **করে না** (আধুনিক runtimes ভিন্ন internal design ও randomized/defensive techniques ব্যবহার করতে পারে; V8 `Map`/`Set` internals Phase 5-এ)।
