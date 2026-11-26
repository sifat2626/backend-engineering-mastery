# Phase 2, Block 1 — Computational Cost, Growth Rates, and Asymptotic Notation

## Study Time Guide

| Activity | Assigned Scope | Estimated Time |
| :--- | :--- | :---: |
| **Guided Lesson + Pause & Checks 1–3** *(main priority)* | পুরো guided lesson মনোযোগ দিয়ে পড়া এবং `phases/phase-02/notes/block-01-notes.md`-এ Pause & Checks 1–3 লেখা | **~50–65 min** *(flexible — limit নয়)* |
| **Mandatory 1:** Pat Morin — *Open Data Structures* | কেবল Sections `1.1`, `1.3.1`, `1.3.3`, `1.4`, এবং `1.5` *(explicitly skip `1.3.2` ও `1.3.4`)* | ~25–30 min |
| **Mandatory 2:** MIT OCW 6.006 (Spring 2020) — *Lecture 1 Notes* | Lecture 1 Notes PDF — কেবল *Problems vs. Algorithms*, *Word-RAM Model*, এবং *Asymptotic Notation (`O`, `Ω`, `Θ`)* | ~20–25 min |
| **Exercise 1 (Mandatory):** Scaling Explosion & Crossover Math | ৪টি abstract operation-count model `N = 50` থেকে `1,000,000` পর্যন্ত হিসাব, exact crossover point নির্ণয় ও flash-sale বিশ্লেষণ | ~20–25 min |
| **Exercise 2 (Mandatory):** Technical Claim & Notation Audit | PR/Slack-এর ৬টি engineering দাবি `True` / `Technically True but Loose/Misleading` / `False` হিসেবে classify ও precise rewrite | ~25–30 min |
| **Total (Required Core — without Optional Assessment)** | Guided Lesson + ২টি Mandatory Reading + ২টি Mandatory Exercise | **~140–175 min (~2.3–2.9 h)** |
| **Optional Closed-Book Assessment** *(optional)* | Notes/tabs বন্ধ রেখে ৫টি synthesis প্রশ্নের উত্তর লেখা | ~20–25 min *(optional)* |
| **Total (with Optional Assessment)** | Required Core + Optional Closed-Book Assessment | **~160–200 min (~2.7–3.3 h)** |

> **Planning Note:** Approved Phase 2 Research Dossier-এ Block 1-এর target allocation হলো **3 hours (180 minutes)**—যা একটি flexible planning reference, কোনো strict deadline নয়। দ্রুত শেষ করার চেয়ে concepts গভীরভাবে understand করাই main priority; বিশেষ করে **Guided Lesson**-এর জন্য প্রয়োজনে আনুমানিক সময়ের চেয়ে বেশি সময় লাগা একদম স্বাভাবিক। প্রতিটি reading estimate কেবল নির্ধারিত sections-এর জন্য, পুরো বই বা course পড়ার জন্য নয়। **Optional Closed-Book Assessment** সম্পূর্ণ optional এবং Block 2 শুরু করার জন্য কোনো mandatory gate নয়।

---

Phase 1-এর শেষে আমাদের online store-এর সেই promo-redemption backend module-টার কথা মনে করুন। আপনার laptop-এ `100`টা sample order আর `100`টা redemption record নিয়ে test চালালেন—পুরো duplicate check আর order matching শেষ হয়ে গেল **1 millisecond-এরও কম** সময়ে! Unit tests সব green, code পড়তে clean, আর staging-এ ছোট demo dataset-এও সব দারুণ fast। টিম আত্মবিশ্বাসের সাথে feature-টা production-এ deploy করে দিল।

তিন মাস পর store-এর বছরের সবচেয়ে বড় flash sale এল। এখন campaign-এ `100`টা নয়—জমা হয়েছে **`1,000,000` (১০ লাখ)** redemption record!

আর ঠিক সেই মুহূর্তে একটা অদ্ভুত ঘটনা ঘটল। কোনো syntax error নেই, কোনো business invariant ভাঙেনি, database-ও চালু আছে—তবু যে matching logic-টা `100`টা record-এ চোখের পলকে শেষ হচ্ছিল, তা হঠাৎ মিনিটের পর মিনিট আটকে থাকতে শুরু করল, server-এর memory ভরে গেল, আর checkout requests একের পর এক timeout হয়ে পড়ল!

একটু থেমে ভেবে দেখুন: **একই code `100`টা item-এ এত fast চলল, অথচ `1,000,000` item-এ এসে হঠাৎ ভেঙে পড়ল কেন?**

কারণ ছোট data (`N = 100`)-এ ভালো performance দেখতে পারাটা backend engineering-এর সবচেয়ে পরিচিত ফাঁদগুলোর একটি। Phase 1-এ আপনি **scale**, **trade-offs**, ও **evidence** নিয়ে qualitative mindset তৈরি করেছেন। Phase 2-এ আমরা সেই mindset-কে **quantitative (গাণিতিক)** রূপ দিচ্ছি—যাতে production ভেঙে পড়ার *আগেই* আপনি হিসাব করে বলতে পারেন input বড় হলে কোনো approach কীভাবে behave করবে।

এই পুরো Block 1-কে এক সুতোয় গেঁথে রাখবে একটি কেন্দ্রীয় প্রশ্ন:

> **কোনো backend approach-এর computational cost আসলে কী measure করে, input size বাড়লে বিভিন্ন growth rate কীভাবে বিস্ফোরিত হয়, `O`, `Ω` ও `Θ` দিয়ে কীভাবে তা নিখুঁতভাবে প্রকাশ করা হয়—এবং কোথায় এসে asymptotic analysis-এর সীমা শেষ হয়?**

---

## Milliseconds দিয়ে নয়: Computational Cost আসলে কী Measure করে?

প্রথম তাগিদটা খুব স্বাভাবিক: *"কোন approach বেশি fast, সেটা বুঝতে দুটো function-ই আমার laptop-এ চালিয়ে milliseconds মেপে নিলেই তো হয়!"*

কিন্তু একটু ভাবুন, আপনি যদি বলেন *"Approach A চলে `4 ms`-এ আর Approach B চলে `9 ms`-এ"*—তাহলে ওই সংখ্যার ভেতরে algorithm-এর নিজস্ব efficiency-র পাশাপাশি আরও অন্তত পাঁচটা অপ্রাসঙ্গিক জিনিস মিশে আছে:
- আপনার laptop-এর CPU কতটা fast (নতুন processor বনাম পুরোনো cloud VM),
- ঠিক ওই মুহূর্তে background-এ browser বা OS updates কতটা CPU খাচ্ছে,
- কোন programming language ও runtime version ব্যবহার করছেন,
- code-এর ছোটখাটো syntax detail,
- এবং সবচেয়ে গুরুত্বপূর্ণ: **ঠিক কোন input size-এ** আপনি test চালিয়েছেন (`100`টা order নাকি `1,000,000`টা order)!

একই algorithm পুরোনো laptop-এ `20 ms`, নতুন server-এ `3 ms`, আর ভিন্ন input size-এ `300,000 ms` নিতে পারে। তাই শুধু একটা fixed millisecond সংখ্যা আমাদের বলে না **algorithm-টার নিজস্ব কাঠামো input বড় হওয়ার সাথে কত দ্রুত ভারী হয়ে ওঠে।**

এই কারণেই computer science ও backend engineering-এ আমরা wall-clock milliseconds-এর বদলে প্রথমে measure করি **abstract computational cost**—অর্থাৎ input size-এর ফাংশন হিসেবে:
1. কতগুলো **elementary (primitive) operations** execute করতে হয় (**Time Complexity**), এবং
2. কাজটি সম্পন্ন করতে কতটুকু **memory** লাগে (**Space Complexity**)।

### Input Size `N` বলতে আসলে কী বোঝায়?

এখানেই একটা সূক্ষ্ম ভুল প্রায়ই ঘটে—অনেকে মনে করেন `N` মানে সবসময় "একটা array-র length"। বাস্তবে **`N` কখনোই স্বয়ংক্রিয়ভাবে এক জিনিস নয়; প্রতিটা problem-এ `N` কী নির্দেশ করছে তা স্পষ্টভাবে define করতে হয়!**

বাস্তব backend কাজের চারটি উদাহরণ দেখুন:
- **Order matching / promo lookup:** `N` হতে পারে catalog বা history-তে থাকা **records-এর সংখ্যা** (যেমন `N`টি order)।
- **JSON payload parsing বা string comparison:** `N` (বা `L`) হলো payload-এর **bytes বা characters-এর সংখ্যা**—একটা single HTTP request-এ `1`টা মাত্র string থাকতে পারে, কিন্তু সেই string যদি `5 MB` হয়, তবে `N = 5,000,000` characters!
- **Cryptography বা huge integer arithmetic:** একটি বিশাল সংখ্যার মান `X` নয়, বরং সেই সংখ্যাটি binary-তে লিখতে যতগুলো **bits** লাগে (`b = floor(log2 X) + 1`), অনেক ক্ষেত্রে সেটাই input size।
- **Multi-input backend operations:** অনেক সময় একটিমাত্র `N` যথেষ্ট নয়—যেমন `N`টি order-কে `M`টি active promo rule-এর সাথে match করতে গেলে cost নির্ভর করে **দুটো independent input size** (`N` এবং `M`)-এর ওপর *(multi-variable code analysis আমরা Block 2-এ হাতে-কলমে করব)*।

তাই কোনো complexity দাবি করার আগে প্রথম শর্ত হলো: **"এখানে `N` আসলে কী?"**—তা স্পষ্ট করে বলা।

---

## Word-RAM Model: কোন কাজটাকে "১ ধাপ" ধরব?

এবার পরের প্রশ্নটা আসে: *"আমরা বললাম primitive operations গুনব—কিন্তু কোন কাজটাকে ১টা primitive operation ধরব? একটা addition কি ১ ধাপ? একটা array lookup কি ১ ধাপ? দুটো 5 MB string compare করা কি ১ ধাপ?"*

এই প্রশ্নের একটা পরিষ্কার, machine-independent ভিত্তি দেওয়ার জন্য algorithms বিশ্লেষণে একটি standard abstract machine ব্যবহার করা হয়, যার নাম **Word-RAM (Random Access Machine) Model** *(Pat Morin, Open Data Structures 1.4; MIT 6.006 Lecture 1)*।

Word-RAM model-এর মূল ধারণা খুব পরিষ্কার—এটি আধুনিক কম্পিউটারের একটি **idealized mathematical model**, যা তিনটি সহজ assumption-এর ওপর দাঁড়িয়ে আছে:

1. **Fixed-size words (`w`-bit words):** Memory হলো অনেকগুলো cell-এর একটি ধারাবাহিক ভাণ্ডার, যেখানে প্রতিটি cell একটি নির্দিষ্ট আকারের **`w`-bit word** ধারণ করে (আধুনিক 64-bit backend server-এ সাধারণত `w = 64` bits; তাত্ত্বিক বিশ্লেষণে ধরা হয় `w >= log2 N`, যাতে একটি word-এর ভেতর input size `N` পর্যন্ত যেকোনো index বা integer সহজেই ধরে)।
2. **Uniform `O(1)` memory access:** Memory-র যেকোনো address (যেমন `arr[0]` বা `arr[999999]`) সরাসরি read বা write করতে সমান **1 unit step** সময় লাগে।
3. **Unit-cost primitive operations on words:** Fixed-size word-এর ওপর মৌলিক কাজগুলো—যেমন arithmetic (`+`, `-`, `*`, `/`), numeric comparison (`<`, `>`, `===`), boolean logic (`&&`, `||`), variable assignment, এবং array index access—প্রতিটি **1 unit step** নেয়।

কেন এই model এত useful? কারণ CPU clock speed, operating system, বা language syntax-এর ঝামেলা বাদ দিয়ে এটি আমাদের পরিষ্কারভাবে বুঝতে দেয় একটি algorithm-এর step count `N`-এর সাথে কীভাবে scale করে।

তবে **প্রথম দিন থেকেই একটি জরুরি সতর্কতা মাথায় গেঁথে রাখুন:** Word-RAM হলো একটি **useful abstraction**, বাস্তব hardware-এর হুবহু নকল নয়!
- দুটো 64-bit integer তুলনা করা (`orderIdA === orderIdB`) word-RAM-এ `1` unit step—কিন্তু দুটো **5 MB string** তুলনা করা (`payloadA === payloadB`) `1` step **নয়**! কারণ 5 MB একটা single 64-bit word-এ ধরে না; engine-কে worst case-এ মিলিয়নখানেক byte একটার পর একটা মিলিয়ে দেখতে হয়!
- একইভাবে, বাস্তব hardware-এ সব memory access সমান দ্রুত নয়—CPU cache, main RAM, আর disk/network-এর গতির মধ্যে বিশাল ব্যবধান আছে *(এই hardware reality আমরা Block 5-এ গভীরভাবে দেখব)*।

---

## Time Complexity বনাম Space Complexity: Input Space আর Auxiliary Space এক নয়

Backend system কেবল slow হয়ে (CPU/time) ভেঙে পড়ে না—অনেক সময় এটি **Out of Memory (OOM)** হয়ে হঠাৎ crash করে! তাই একটি algorithm-এর cost বিচার করতে হলে **Time Complexity**-র পাশাপাশি **Space Complexity**-ও সমান মনোযোগ দিয়ে দেখতে হয়।

কিন্তু Space Complexity নিয়ে সবচেয়ে সাধারণ ভুলটা হলো **Input Space** আর **Auxiliary Space**-কে একসাথে গুলিয়ে ফেলা:

- **Input Space:** Caller যে input data ফাংশনটিকে দিয়েছে (যেমন `N`টি order-এর array), সেটি memory-তে রাখতে যে জায়গা লাগে। এই memory-র মালিক caller—আপনার algorithm নিজে এটি তৈরি করেনি।
- **Auxiliary Space (Extra Working Space):** Input ছাড়া কাজটি সম্পন্ন করার জন্য **আপনার algorithm নিজে অতিরিক্ত যতটুকু memory ব্যবহার করে**। এর ভেতর পড়ে:
  - local variables, counters, এবং নতুন তৈরি করা arrays, objects, `Map`, বা `Set`,
  - built-in helper-এর ভেতরে লুকানো temporary copies (যেমন `arr.map(...)`, `arr.filter(...)`, `arr.slice(...)`, বা `JSON.stringify(...)`),
  - এবং recursive function call-এর **call stack** *(recursion-এর stack space আমরা Block 2-এ বিস্তারিত দেখব)*।

ছোট একটি বাস্তব Node.js উদাহরণে পার্থক্যটা চোখে দেখুন (এখানে `N` হলো `orders` array-তে থাকা order-এর সংখ্যা):

```javascript
// Approach 1: একটি মাত্র running counter ব্যবহার করে মোট মূল্য হিসাব
function sumOrderTotalsInPlace(orders) {
  let totalCents = 0;
  for (let i = 0; i < orders.length; i++) {
    totalCents += orders[i].amountCents;
  }
  return totalCents;
}

// Approach 2: আগে নতুন array তৈরি করে তারপর যোগ করা
function sumOrderTotalsWithCopy(orders) {
  const amounts = orders.map((order) => order.amountCents);
  let totalCents = 0;
  for (let i = 0; i < amounts.length; i++) {
    totalCents += amounts[i];
  }
  return totalCents;
}
```

ভালো করে দুটো function-এর **Auxiliary Space** তুলনা করুন:
- **Approach 1:** Input `orders` array-তে `N`টা element আছে (ওটা *input space*)। কিন্তু function-টি নিজে অতিরিক্ত তৈরি করেছে শুধু দুটি number variable (`totalCents` ও `i`)। `N` যদি `100` হয় তবু `2`টা variable; `N` যদি `1,000,000` হয় তবুও `2`টা variable! তাই এর **auxiliary space constant** (যা একটু পরেই আমরা `Θ(1)` বলব)।
- **Approach 2:** দেখে মনে হতে পারে *"আমি তো মাত্র একটা extra variable `amounts` নিয়েছি!"* কিন্তু `orders.map(...)` ভেতরে পুরো **`N`টি element-এর একটি নতুন array** memory-তে allocate করে। `N = 1,000,000` হলে এটি `1,000,000` সংখ্যার একটি বাড়তি array তৈরি করবে—তাই এর **auxiliary space linear** (`N`-এর সমানুপাতিক)!

তাই বাস্তব engineering আলোচনায় কেউ যখন জিজ্ঞাসা করেন *"এই function-টার space complexity কত?"*, তখন প্রায় সবসময় তাঁরা আসলে **auxiliary space**-এর কথাই জানতে চান—তবু ambiguity এড়াতে স্পষ্টভাবে *"auxiliary space"* বলাটাই সবচেয়ে নিরাপদ অভ্যাস।

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-01-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন teammate বললেন: *"আমি আমার MacBook-এ নতুন promo-validator চালিয়ে দেখলাম `1.2 ms` লাগে, তাই এর time complexity দারুণ!"* **Word-RAM model** এবং **wall-clock runtime বনাম abstract operation counts**-এর পার্থক্য দিয়ে বুঝিয়ে বলুন কেন এই একক `1.2 ms` পরিমাপ algorithm-টির scaling efficiency প্রমাণ করে না।
> 2. একটি backend function input হিসেবে `N`টি order-এর array নেয়, কোনো নতুন array বা object তৈরি না করে শুধু সর্বোচ্চ `amountCents` খুঁজে একটি integer return করে। আরেকজন developer দাবি করলেন এর auxiliary space `Θ(N)` কারণ function-টি `N`টি order পড়েছে। দাবিটি কি সঠিক? **Input space** বনাম **auxiliary space** দিয়ে ব্যাখ্যা করুন, এবং বলুন কেন `N` কী নির্দেশ করছে তা স্পষ্ট করা জরুরি।

---

## Growth Rates কীভাবে বিস্ফোরিত হয়—এবং Logarithm-এর সহজ Intuition

এবার আমাদের opening flash-sale রহস্যে ফিরে আসুন। ধরুন আমাদের কাছে `N`টি order আছে, এবং `N` ধীরে ধীরে `100` থেকে বেড়ে `1,000,000` হচ্ছে—অর্থাৎ input size বাড়ছে **`10,000×` (দশ হাজার গুণ)**।

বিভিন্ন algorithm-এর operation count এই `10,000×` বৃদ্ধিতে কীভাবে সাড়া দেয়, তা দেখার আগে একটা শব্দ অনেকের মাথায় অকারণ ভয় তৈরি করে: **Logarithm (`log N`)**।

এখানে কোনো calculus বা কঠিন সূত্র মুখস্থ করার দরকার নেই। Computer science-এ **`log2(N)`** (base-2 logarithm)-এর সবচেয়ে সহজ, অটুট intuition হলো **repeated halving (বারবার অর্ধেক করা)**:

> **`log2(N)` মানে হলো: সংখ্যা `N`-কে বারবার `2` দিয়ে ভাগ করে অর্ধেক করতে থাকলে কত ধাপে সংখ্যাটি `1`-এ নেমে আসে।**

হাতে গুনে দেখুন:
- `N = 8`: `8 → 4 → 2 → 1` = **3 ধাপ** (`2^3 = 8`, তাই `log2(8) = 3`)।
- `N = 16`: `16 → 8 → 4 → 2 → 1` = **4 ধাপ** (`2^4 = 16`, তাই `log2(16) = 4`)।
- `N = 1,024`: `2^10 = 1,024`, তাই মাত্র **10 ধাপ**!
- `N = 1,048,576` (~১ মিলিয়ন): `2^20 = 1,048,576`, তাই মাত্র **20 ধাপ**!
- `N = 1,073,741,824` (~১ বিলিয়ন): `2^30 ≈ 10^9`, তাই মাত্র **30 ধাপ**!

একটা বাস্তব ছবি কল্পনা করুন: আপনার কাছে `1,000,000`টি sorted order ID-র একটা বই আছে। আপনি প্রতিবার ঠিক মাঝখানের পাতা খুলে অর্ধেক বই বাদ দিয়ে দিচ্ছেন (binary search-এর মূল আইডিয়া)। মাত্র **20 বার** পাতা খুললেই ১ মিলিয়ন order-এর মধ্যে যেকোনো একটা order বের হয়ে যাবে! আর ১ মিলিয়ন থেকে ১ বিলিয়নে গেলে (১,০০০ গুণ বড় dataset!) বাড়তি ধাপ লাগবে মাত্র **10টা**!

*(একটা ছোট mathematical note: অনেক জায়গায় `log2(N)` না লিখে শুধু `log N` লেখা হয়। কেন base (`2` নাকি `10` নাকি `e`) উল্লেখ করা জরুরি হয় না? কারণ logarithm-এর change-of-base নিয়ম অনুযায়ী `log10(N) = log2(N) / log2(10) ≈ log2(N) / 3.32`। অর্থাৎ base বদলালে মান কেবল একটি **fixed constant factor** দিয়ে গুণ বা ভাগ হয়—growth curve-এর shape বদলায় না!)*

এবার computer science ও backend engineering-এর **৭টি মূল complexity class** এক টেবিলে পাশাপাশি দেখুন—প্রতিটি ঘরে দেওয়া সংখ্যা হলো **abstract operation count** (আবার মনে রাখবেন: এগুলো **milliseconds নয়**, বরং model operation counts, যা দেখায় `N` বাড়লে কাজের পরিমাণ কত গুণ বাড়ে!):

| Complexity Class | নাম (Common Name) | `N = 100` | `N = 1,000` | `N = 10,000` | `N = 100,000` | `N = 1,000,000` | বাস্তব Backend অনুভূতি |
| :--- | :--- | ---: | ---: | ---: | ---: | ---: | :--- |
| **`1`** | Constant | `1` | `1` | `1` | `1` | `1` | Input যতই বড় হোক, কাজের ধাপ স্থির থাকে (যেমন `arr[i]` index read)। |
| **`log2 N`** | Logarithmic | `~6.6` | `~10.0` | `~13.3` | `~16.6` | `~19.9` | `N` দশ হাজার গুণ বাড়লেও কাজ বাড়ে মাত্র `~3×` (যেমন sorted array-তে binary search)। |
| **`N`** | Linear | `100` | `1,000` | `10,000` | `100,000` | `1,000,000` | Input `10×` বাড়লে কাজ ঠিক `10×` বাড়ে (যেমন পুরো list একবার scan করা)। |
| **`N log2 N`** | Linearithmic | `~664` | `~9,966` | `~132,877` | `~1.66 × 10^6` | `~1.99 × 10^7` | Linear-এর চেয়ে কিছুটা বেশি, কিন্তু ১ মিলিয়ন item-এও সম্পূর্ণ manageable (যেমন ভালো comparison sort)। |
| **`N^2`** | Quadratic | `10,000` | `1,000,000` | `10^8` | `10^10` | `10^12` | `N = 100`-এ মাত্র `10,000` (চোখের পলকে শেষ!), কিন্তু `N = 1,000,000`-এ **`1 ট্রিলিয়ন`** (`100,000,000×` বৃদ্ধি!)—এখানেই flash sale ভেঙে পড়ে! |
| **`N^3`** | Cubic | `10^6` | `10^9` | `10^12` | `10^15` | `10^18` | এমনকি `N = 1,000`-এই ১ বিলিয়ন ops—বড় backend data-য় ব্যবহার অযোগ্য। |
| **`2^N`** | Exponential | `~1.27 × 10^30` | `~1.07 × 10^301` | অকল্পনীয় | অকল্পনীয় | অকল্পনীয় | `N = 50`-এই `2^50 ≈ 1.13 × 10^15`; প্রতিটি `+1` input কাজকে **দ্বিগুণ** করে দেয়! |

*(সম্পূর্ণ ক্রমটি মনে রাখার জন্য: `1 < log N < sqrt(N) < N < N log N < N^2 < N^3 < 2^N < N!`)*

ওপরের **`N^2`** সারিটির দিকে আরেকবার তাকান। আপনার opening mystery-র পুরো উত্তর ওই একটা সারিতেই লেখা আছে!
- `N = 100`-এ `N^2 = 10,000` ops—আধুনিক CPU-তে যা ১ millisecond-এর ভগ্নাংশে শেষ হয়ে যায়।
- `N = 1,000,000`-এ `N` বেড়েছে `10,000×`, কিন্তু `N^2` বেড়েছে `(10,000)^2 = 100,000,000×` (দশ কোটি গুণ!) → `10^12` ops!

কিন্তু বাস্তব code-এর operation count তো সাধারণত পরিষ্কার `N^2` হয় না—বরং হয় `f(N) = 3N^2 + 50N + 120` বা `f(N) = 2N + 10`-এর মতো জটিল expression। এখন প্রশ্ন হলো: **এই জটিল expression-গুলোকে একটি নিখুঁত, সর্বজনীন mathematical ভাষায় বাঁধব কীভাবে?**

---

## Big-O (`O`), Big-Omega (`Ω`), এবং Big-Theta (`Θ`) — নিখুঁত অর্থ ও সাধারণ ভুল

একটা function-এর exact operation count যেমন `f(n) = 2n + 10` বা `f(n) = 3n^2 + 50n + 120` নিয়ে কাজ করার সময় আমরা জানতে চাই: **`n` যখন অনেক বড় হয় (asymptotically, অর্থাৎ `n → ∞`), তখন function-টির growth curve কোন মানদণ্ডের নিচে থাকে, কোন মানদণ্ডের ওপরে থাকে, নাকি ঠিক কোন মানদণ্ডের সাথে সমান তালে বাড়ে?**

ঠিক এই তিনটি আলাদা প্রশ্নের জন্যই তিনটি আলাদা **asymptotic notation** আছে *(CLRS 4th ed., Ch. 3; MIT 6.006 Lecture 1)*। সংখ্যা তুলনা করার পরিচিত তিনটি চিহ্নের (`<=`, `>=`, `=`) সাথে মিলিয়ে নিলে এদের অর্থ এক সেকেন্ডে পরিষ্কার হয়ে যায়:

### ১. Big-O (`O`) — Asymptotic Upper Bound *(ধারণাগতভাবে `<=`)*

- **Formal Definition:** `f(n) = O(g(n))` (অনেক textbook-এ set notation দিয়ে `f(n) ∈ O(g(n))` লেখা হয়) বলতে বোঝায়: এমন দুটি positive constant **`c > 0`** এবং **`n0 >= 1`** খুঁজে পাওয়া যাবে যাতে **সব `n >= n0`-এর জন্য**
  ```text
  0 <= f(n) <= c * g(n)
  ```
- **সহজ বাংলায় অর্থ:** যথেষ্ট বড় `n`-এর (`n >= n0`) পর থেকে `f(n)` **কখনোই** `g(n)`-এর একটি constant multiple-এর চেয়ে দ্রুত বাড়বে না—অর্থাৎ `g(n)` হলো `f(n)`-এর একটি **ছাদ (ceiling / upper bound)**।
- **Worked Example 1 — `f(n) = 2n + 10` কি `O(n)`?**
  - আমরা এমন `c` ও `n0` খুঁজি যেন `2n + 10 <= c * n` হয়।
  - খেয়াল করুন: `n >= 10` হলে `10 <= n`, তাই `2n + 10 <= 2n + n = 3n`!
  - অতএব `c = 3` এবং `n0 = 10` নিলেই সব `n >= 10`-এর জন্য `0 <= 2n + 10 <= 3n` সত্য।
  - সুতরাং **`2n + 10 = O(n)`** প্রমাণিত!
- **Worked Example 2 (অত্যন্ত জরুরি সূক্ষ্মতা!) — `f(n) = 2n + 10` কি `O(n^2)`?**
  - একটু থেমে definition-টা মিলিয়ে দেখুন! আমরা কি এমন `c > 0` ও `n0 >= 1` পাব যাতে সব `n >= n0`-এর জন্য `2n + 10 <= c * n^2` হয়?
  - ধরুন `c = 1` এবং `n0 = 5` নিলাম:
    - `n = 5` হলে: `2(5) + 10 = 20`, আর `1 * 5^2 = 25` → `20 <= 25` (সত্য!)
    - `n = 6` হলে: `2(6) + 10 = 22`, আর `1 * 6^2 = 36` → `22 <= 36` (সত্য!)
    - যেকোনো `n >= 5`-এর জন্য `n^2 - 2n - 10 = (n - 1)^2 - 11 >= 4^2 - 11 = 5 > 0`, তাই `2n + 10 <= n^2` সর্বদা সত্য!
  - **চমকপ্রদ কিন্তু ১০০% গাণিতিক সত্য:** `2n + 10` হলো **technically `O(n^2)`** (এমনকি `O(n^3)`-ও বটে)!
  - কেন? কারণ `O` মানে হলো **upper bound (`<=`)**। ঠিক যেমন *"একজন adult-এর বয়স `<= 500` বছর"* কথাটা গাণিতিকভাবে **সত্য** (মিথ্যা নয়!), কিন্তু একজন engineer-এর জন্য এটি এতই **loose (ঢিলেঢালা)** যে কোনো কাজের তথ্য দেয় না। তাই আমরা সাধারণত **tightest possible bound** চাই।

### ২. Big-Omega (`Ω`) — Asymptotic Lower Bound *(ধারণাগতভাবে `>=`)*

- **Formal Definition:** `f(n) = Ω(g(n))` বলতে বোঝায়: এমন দুটি positive constant **`c > 0`** এবং **`n0 >= 1`** খুঁজে পাওয়া যাবে যাতে **সব `n >= n0`-এর জন্য**
  ```text
  0 <= c * g(n) <= f(n)
  ```
- **সহজ বাংলায় অর্থ:** যথেষ্ট বড় `n`-এর পর থেকে `f(n)` **অন্তত** `g(n)`-এর একটি constant multiple-এর সমান বা তার চেয়ে দ্রুত বাড়বে—অর্থাৎ `g(n)` হলো `f(n)`-এর একটি **মেঝে (floor / lower bound)**।
- **Worked Example — `f(n) = 2n + 10`:**
  - `c = 2` এবং `n0 = 1` নিলে সব `n >= 1`-এর জন্য `2n <= 2n + 10` সত্য। তাই **`2n + 10 = Ω(n)`**!
  - কিন্তু `2n + 10` কি `Ω(n^2)`? **না!** কারণ আপনি যত ছোট positive constant `c` নিন না কেন (ধরুন `c = 0.0001`), যথেষ্ট বড় `n` এলে `0.0001 * n^2` ঠিকই `2n + 10`-কে ছাড়িয়ে যাবে। তাই `n^2` কখনোই `2n + 10`-এর মেঝে (floor) হতে পারে না।

### ৩. Big-Theta (`Θ`) — Asymptotic Tight Bound *(ধারণাগতভাবে `=`, ছাদ ও মেঝে দুই-ই!)*

- **Formal Definition:** `f(n) = Θ(g(n))` বলতে বোঝায়: `f(n) = O(g(n))` **এবং** `f(n) = Ω(g(n))`—অর্থাৎ এমন positive constants `c1 > 0`, `c2 > 0`, এবং `n0 >= 1` থাকবে যাতে সব `n >= n0`-এর জন্য
  ```text
  0 <= c1 * g(n) <= f(n) <= c2 * g(n)
  ```
- **সহজ বাংলায় অর্থ:** `f(n)` ওপর ও নিচ—উভয় দিক থেকেই `g(n)`-এর constant multiples দিয়ে **আঁটসাঁটভাবে (tightly)** বন্দী। তাই `Θ` বলে দেয় `f(n)` ঠিক কোন order-of-growth-এ বাড়ে—না তার চেয়ে বেশি, না তার চেয়ে কম।
- **Worked Example — `f(n) = 2n + 10`:**
  - ওপরেই আমরা দেখেছি `c1 = 2`, `c2 = 3`, এবং `n0 = 10` নিলে সব `n >= 10`-এর জন্য:
    `2n <= 2n + 10 <= 3n`।
  - অতএব **`2n + 10`-এর tight bound হলো `Θ(n)`**—এটি `O(n^2)` হলেও **`Θ(n^2)` নয়**!

| Notation | সাধারণ তুলনা | কী নিশ্চিত করে | `f(n) = 2n + 10`-এর ক্ষেত্রে সত্য না মিথ্যা? |
| :---: | :---: | :--- | :--- |
| **`O(g(n))`** | `<=` (Ceiling) | যথেষ্ট বড় `n`-এ `f(n)` সর্বোচ্চ `c * g(n)`-এর ভেতর থাকবে। | `O(n)` **সত্য**; `O(n^2)` **technically সত্য** (কিন্তু loose!); `O(log n)` **মিথ্যা**। |
| **`Ω(g(n))`** | `>=` (Floor) | যথেষ্ট বড় `n`-এ `f(n)` অন্তত `c * g(n)` হবেই। | `Ω(n)` **সত্য**; `Ω(1)` **technically সত্য** (কিন্তু loose!); `Ω(n^2)` **মিথ্যা**। |
| **`Θ(g(n))`** | `=` (Tight sandwich) | `O(g(n))` এবং `Ω(g(n))` **দুটোই** একসাথে সত্য। | `Θ(n)` **সত্য (exact tight bound)**; `Θ(n^2)` **মিথ্যা**; `Θ(1)` **মিথ্যা**। |

### সবচেয়ে বড় Misconception ভাঙুন: Big-O ≠ Worst-Case, এবং Big-O ≠ Exact Runtime!

Industry-তে, blog post-এ, এমনকি অনেক senior developer-এর মুখেও আপনি শুনবেন: *"Big-O মানে worst-case, আর Big-Omega মানে best-case!"*

**এটি একটি সম্পূর্ণ ভুল ধারণা।** এখানে দুটি একদম আলাদা, পরস্পর-স্বাধীন (**orthogonal**) জিনিস গুলিয়ে ফেলা হয়:
1. **Input Case (Best / Average / Worst case):** আমরা *কোন input পরিস্থিতির* cost মাপছি? *(এগুলোর গভীর বিশ্লেষণ আসবে Block 3-এ।)*
2. **Asymptotic Bound (`O` / `Ω` / `Θ`):** ওই নির্দিষ্ট cost function-টির ওপর আমরা কোন গাণিতিক bound (ceiling, floor, নাকি tight sandwich) বসাচ্ছি?

একটা খুব সহজ backend উদাহরণ দিয়ে পুরো বিভ্রান্তি চিরতরে দূর করুন। ধরুন একটা unsorted array-তে `N`টা order আছে, আর আমরা শুরু থেকে শেষ পর্যন্ত খুঁজে নির্দিষ্ট একটা `targetOrderId` বের করছি (linear search):
- **Best case** (একদম প্রথম index-এই order-টা পাওয়া গেল): লাগে `1`টি comparison। এর **tight bound** হলো `Θ(1)`—যার মানে এর **Big-O upper bound-ও `O(1)`** এবং **Big-Omega lower bound-ও `Ω(1)`**! দেখলেন? Best-case-এরও Big-O আছে!
- **Worst case** (order-টা একদম শেষে আছে অথবা array-তেই নেই): লাগে `N`টি comparison। এর **tight bound** হলো `Θ(N)`—যার মানে এর **Big-O upper bound `O(N)`** *এবং* **Big-Omega lower bound `Ω(N)`**! দেখলেন? Worst-case-এরও Big-Omega আছে!
- **All-inputs statement:** যেকোনো input-এ linear search কমপক্ষে `1` ধাপ এবং সর্বোচ্চ `N` ধাপ নেয়—তাই সব input মিলিয়ে এর cost `Ω(1)` এবং `O(N)` (এখানে একক কোনো `Θ` নেই, কারণ best আর worst আলাদা rate-এ বাড়ে)।

আর দ্বিতীয় misconception: **Big-O কখনোই exact milliseconds বা exact execution speed নয়।** একটি `O(N)` function `1 ms`-এও চলতে পারে, আবার `500 ms`-এও চলতে পারে—এটি কেবল বলে input বড় হলে abstract operation count সর্বোচ্চ linear হারে বাড়বে।

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-01-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. ধরুন একটি function-এর exact operation count হলো `f(n) = 5n + 100`। (ক) নির্দিষ্ট `c` ও `n0` দেখিয়ে প্রমাণ করুন যে `f(n) = Θ(n)`। (খ) কেউ যদি বলে `f(n) = O(n^2)`—তা কি গাণিতিকভাবে মিথ্যা, নাকি সত্য কিন্তু বিভ্রান্তিকর? আর `f(n) = Θ(n^2)` বলা কি সত্য না মিথ্যা?
> 2. একজন সহকর্মী PR review-তে লিখলেন: *"এই search algorithm-টার best-case তো `O(1)` হতে পারে না, কারণ Big-O শুধু worst-case-এর জন্য ব্যবহার হয়!"* এই মন্তব্যটি কেন ভুল, এবং **input case** বনাম **asymptotic bound** কীভাবে পরস্পর থেকে স্বাধীন তা উদাহরণসহ ২–৩ বাক্যে ব্যাখ্যা করুন।

---

## Dominant Terms, Constant Factors, এবং Crossover Points

এখন একটা বাস্তব প্রশ্ন উঠবে: `f(N) = 3N^2 + 50N + 120` লিখলে asymptotic analysis কেন `3` (leading constant), `50N` (lower-order term), আর `120` (constant term)—সব ফেলে দিয়ে শুধু `Θ(N^2)` রাখে?

সংখ্যা বসিয়ে দেখলেই কারণটা চোখের সামনে ভেসে ওঠে:

| Input Size `N` | `3N^2` (Dominant Term) | `50N` (Lower-Order Term) | `120` (Constant) | মোট `f(N)` | মোট কাজে `3N^2`-এর অংশ |
| :---: | ---: | ---: | ---: | ---: | :---: |
| **`10`** | `300` | `500` | `120` | `920` | **32.6%** *(এখানে `50N` বেশি বড়!)* |
| **`100`** | `30,000` | `5,000` | `120` | `35,120` | **85.4%** |
| **`1,000`** | `3,000,000` | `50,000` | `120` | `3,050,120` | **98.4%** |
| **`10,000`** | `300,000,000` | `500,000` | `120` | `300,500,120` | **99.83%** |

দুটো বিপরীত অথচ সমান গুরুত্বপূর্ণ শিক্ষা এই একটি টেবিলেই লুকিয়ে আছে:
1. **`N` বড় হলে dominant term সবকিছু গিলে ফেলে:** `N = 10,000`-এ `50N + 120` মোট কাজের `0.2%`-এরও কম! তাই বড় scale-এ growth behavior বুঝতে কেবল dominant term (`N^2`) দেখাই যথেষ্ট।
2. **কিন্তু ছোট `N`-এ constant factors আর lower-order terms-ই রাজা!** `N = 10`-এ `50N` এবং `120` মিলে মোট কাজের **67.4%** তৈরি করেছে!

### Worked Crossover Calculation: যখন `Θ(N^2)` হারিয়ে দেয় `Θ(N)`-কে!

ধরুন একটি backend matching task-এর জন্য দুটি candidate approach-এর abstract operation-count model হলো:
- **Approach A (`Θ(N)`):** `fA(N) = 5,000 * N` *(linear, কিন্তু প্রতিটি ধাপে ভারী constant setup/overhead = `5,000`)*
- **Approach B (`Θ(N^2)`):** `fB(N) = 0.5 * N^2` *(quadratic, কিন্তু প্রতিটি ধাপ অত্যন্ত হালকা = `0.5`)*

কেউ যদি শুধু asymptotic label দেখে বলে *"Approach A (`Θ(N)`) সবসময় Approach B (`Θ(N^2)`)-এর চেয়ে দ্রুত!"*—তবে সে একটা বিশাল ভুল করবে। আসুন **Crossover Point** (যে positive `N`-এ দুটোর operation count ঠিক সমান হয়) বের করি:

1. সমীকরণ বসাই: `0.5 * N^2 = 5,000 * N`
2. যেহেতু `N > 0`, উভয় পক্ষকে `N` দিয়ে ভাগ করি: `0.5 * N = 5,000`
3. উভয় পক্ষকে `2` দিয়ে গুণ করি: **`N = 10,000`**

এবার crossover point-এর দুই পাশে কী ঘটে দেখুন:
- **`N = 500` (`N < 10,000`):**
  - `fA(500) = 5,000 * 500 = 2,500,000` ops
  - `fB(500) = 0.5 * (500)^2 = 125,000` ops
  - এখানে `Θ(N^2)` approach-টি `Θ(N)`-এর চেয়ে **20 গুণ কম** operations করছে!
- **`N = 10,000` (Exact Crossover):** দুটোই ঠিক `50,000,000` ops।
- **`N = 100,000` (`N > 10,000`):**
  - `fA(100,000) = 500,000,000` ops
  - `fB(100,000) = 5,000,000,000` ops
  - এবার `Θ(N)` approach-টি **10 গুণ কম** operations করছে—এবং `N = 1,000,000` হলে সেটি **100 গুণ** এগিয়ে যাবে!

বাস্তব backend engineering-এ এর অর্থ খুব গভীর: কোনো feature-এ যদি আপনার real `N` প্রুভড ছোট হয় (ধরুন একজন customer-এর cart-এ সর্বোচ্চ `15`টা coupon বা `10`টা shipping zone), তবে ভারী constant-factor ও জটিলতার "asymptotically better" সমাধান বানালে উল্টো system slow ও complex হতে পারে *(Phase 1 Block 6-এর overengineering মনে করুন!)*। আর বিপরীতভাবে, যে data-টা `50` থেকে `1,000,000`-এ বাড়তে পারে, সেখানে small-`N` test দেখে `Θ(N^2)` বেছে নিলে crossover পেরোনোর পর production ভেঙে পড়বে!

---

## Asymptotic Analysis-এর চারটি মূল সীমাবদ্ধতা

তাহলে কি asymptotic notation (`O`, `Ω`, `Θ`) একাই আমাদের বলে দিতে পারে production-এ কোনো API endpoint কত millisecond-এ response দেবে? **কখনোই না!**

একজন mature backend engineer asymptotic analysis-কে যতটা সম্মান করেন, এর **চারটি মৌলিক সীমাবদ্ধতাও** ততটাই পরিষ্কারভাবে মনে রাখেন:

1. **Hidden Constant Factors (`c`):** `2 * N` আর `2,000 * N`—দুটোরই tight bound `Θ(N)`। Asymptotic notation এই `1,000×` পার্থক্য সম্পূর্ণ মুছে দেয়—অথচ বাস্তব production-এ `1 ms` বনাম `1,000 ms` মানেই একটি fast checkout বনাম timeout outage!
2. **Crossover Regions ও Real Input Ranges:** Asymptotic definition বলে *"for all `n >= n0`"*—কিন্তু সেই `n0` (বা crossover point) কত বড়, আর আপনার বাস্তব production traffic সেই বিন্দুর ওপরে না নিচে থাকে, তা কেবল `O(...)` লেখা দেখে বোঝা যায় না।
3. **Fixed-Word Assumption-এর সীমা:** Word-RAM ধরে নেয় প্রতিটি value একটা fixed `w`-bit word-এ এঁটে যায় এবং basic operations `O(1)`। কিন্তু বাস্তব backend-এ যখন আপনি দুটো `5 MB` string compare করেন, বিশাল `BigInt` দিয়ে crypto math চালান, বা গভীর nested JSON object serialize করেন, তখন single operator-ও আর `O(1)` থাকে না—বরং payload size `L`-এর সাথে বাড়ে।
4. **Uniform-Memory ও No-I/O Assumption-এর সীমা:** Word-RAM ধরে নেয় memory-র যেকোনো cell read করতে সমান `1` unit step লাগে, এবং এটি disk, network, বা runtime overhead-কে হিসাবের বাইরে রাখে। কিন্তু বাস্তব কম্পিউটারে CPU cache (nanoseconds), main RAM, SSD (microseconds), এবং database বা network round-trip (milliseconds)-এর মধ্যে লক্ষ-কোটি গুণ গতির পার্থক্য আছে; পাশাপাশি memory allocation, garbage collection, ও runtime optimization-ও real latency বদলে দেয় *(এই hardware ও data-access reality আমরা Block 5-এ এবং controlled empirical measurement Block 7-এ হাতে-কলমে শিখব; আর OS paging, V8 internals, networking, ও SQL query planning আসবে যথাক্রমে Phases 3, 4, 5 ও 10-এ)*।

সুতরাং **Asymptotic analysis হলো আপনার প্রথম quantitative compass—শেষ পরিমাপ নয়।** এটি বলে দেয় *কোন approach scale-এ টিকবে আর কোনটা নিশ্চিতভাবে ভেঙে পড়বে*; আর exact production latency বুঝতে এর সাথে যুক্ত করতে হয় constant factors, hardware/I/O cost intuition, এবং controlled empirical measurement।

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-01-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. আমাদের online store-এ দুটি discount-eligibility approach আছে: Approach X করে `2,000 * log2(N)` abstract operations (`Θ(log N)`), আর Approach Y করে `40 * N` abstract operations (`Θ(N)`)। `N = 16` এবং `N = 100,000`-এ দুটোর আনুমানিক operation count তুলনা করুন এবং ব্যাখ্যা করুন কেন শুধুমাত্র `Θ(log N)` label দেখে সব `N`-এর জন্য Approach X-কে "দ্রুততর" দাবি করা ভুল।
> 2. একটি backend endpoint-এ আপনি দুইটি `Θ(N)` implementation compare করছেন, যেখানে `N` হলো order সংখ্যা। কোন **দুটি** কারণ (Word-RAM / asymptotic limitations থেকে) ব্যাখ্যা করতে পারে যে একই `Θ(N)` হওয়া সত্ত্বেও একটি implementation production-এ অন্যটির চেয়ে `10×` বা `100×` ধীর হতে পারে?

---

## Block 2-এর দিকে সেতুবন্ধন: Formula থেকে আসল Code-এ

একটু থেমে দেখুন Block 1-এ আপনি কী ভিত্তি তৈরি করলেন:
- আপনি এখন জানেন computational complexity আসলে কী measure করে (**Word-RAM** abstract operations ও **input vs. auxiliary space**), আর কেন `N` সবসময় context-এ define করতে হয়।
- আপনি `O` (upper bound), `Ω` (lower bound), ও `Θ` (tight bound)-এর exact অর্থ জানেন এবং *"Big-O = worst-case"* বা *"Big-O = milliseconds"* ভুল ধারণা থেকে মুক্ত।
- আপনি `O(1)` থেকে `O(2^N)` পর্যন্ত growth explosion, base-2 halving logarithm, crossover points, এবং asymptotic analysis-এর চারটি সীমাবদ্ধতা পরিষ্কারভাবে বোঝেন।

কিন্তু একটি বড় ব্যবহারিক প্রশ্ন এখনো অমীমাংসিত রয়ে গেছে:
> *এতক্ষণ আমরা তৈরি করা mathematical expressions (`2n + 10`, `3n^2 + 50n + 120`, `5,000 * N`, `0.5 * N^2`) বিশ্লেষণ করলাম। কিন্তু আগামীকাল যখন আপনার সামনে বাস্তব Node.js backend code থাকবে—যেখানে nested loops, dependent loops, multi-variable inputs (`N` orders ও `M` promos), `.includes()`/`.shift()`-এর মতো built-in calls, কিংবা recursive functions আছে—তখন আপনি নিজে code পড়ে কীভাবে তার exact time ও auxiliary space complexity বের করবেন?*

ঠিক এই দক্ষতাটাই গড়ে উঠবে **Phase 2, Block 2 — Analyzing Loops, Recursion, and Recurrence Reasoning**-এ!

---

## Study Plan, Learning Objectives, Terminology & Resources

> **Continuity & Ownership Notes:** আপনার সব Pause & Checks, দুটি Mandatory Exercise, এবং (যদি করতে চান) Optional Closed-Book Assessment-এর উত্তর স্বাধীনভাবে `phases/phase-02/notes/block-01-notes.md` ফাইলে লিখুন। AI-generated material কখনো completed learning বা learner study hours হিসেবে গণ্য হয় না।
>
> **Progression & Verification Policy:** এই block-এর Closed-Book Assessment সম্পূর্ণ **optional** এবং Block 2 শুরু করার জন্য mandatory gate নয়। পড়া ও দুটি mandatory exercise শেষ করে assessment skip করলে এই block-এর status হবে **`Studied — mastery not formally verified`**; যথেষ্ট evidence share করে ChatGPT evaluation সম্পন্ন হলে তা **`Mastery verified`** হিসেবে গণ্য হবে।

### Study Time Reference

*প্রতিটি activity-র বিস্তারিত time breakdown দেখতে এই নথির শুরুর **Study Time Guide** দেখুন। Approved dossier-এ Block 1-এর target allocation হলো `3 hours (180 min)`—যা একটি flexible planning target, কোনো deadline নয়।*

### Block 1 Learning Objectives *(Primary Target Level: `Working Understanding` | Supported Phase 2 Outcomes: Outcomes 1, 2)*

- **LO 1.1 (`Working Understanding` | Bloom: *Understand*):** Explain what **time complexity** and **space complexity** (input space vs. **auxiliary space**) actually measure under the **word-RAM model of computation**, and why unit-cost operation counts are distinct from wall-clock runtime.
- **LO 1.2 (`Working Understanding` | Bloom: *Analyze*):** Distinguish **Big-O (`O`)** (asymptotic upper bound), **Big-Omega (`Ω`)** (asymptotic lower bound), and **Big-Theta (`Θ`)** (asymptotic tight bound), correcting the pervasive misconception that "`O` means worst-case" or "`O` means exact execution time."
- **LO 1.3 (`Working Understanding` | Bloom: *Apply / Evaluate*):** Compare common complexity classes (`O(1)`, `O(log N)`, `O(N)`, `O(N log N)`, `O(N^2)`, `O(N^3)`, `O(2^N)`) quantitatively as input size `N` grows from `100` to `1,000,000`, explaining how growth rates dominate constant factors at large `N`—and when constant factors and lower-order terms still dominate at small `N`. *(Approved dossier note: Block 1-এ এটি Outcomes 1–2-এর সাথে সামঞ্জস্য রেখে intentionally `Working Understanding`-এ scoped; বাস্তব code থেকে hands-on complexity derivation Block 2-এ `Practical Competence`-এ উন্নীত হয়।)*
- **LO 1.4 (`Working Understanding` | Bloom: *Evaluate*):** Identify the explicit **limitations of asymptotic analysis** (hidden constant factors, crossover regions, fixed-word assumptions, and uniform-memory assumptions) so Big-O is never misused as a standalone runtime predictor.

### Terminology Inventory

| Term | Official Target Level | Why It Matters in Block 1 |
| :--- | :---: | :--- |
| **Word-RAM Model of Computation** | `Working Understanding` | Makes explicit the idealized unit-cost & fixed-word assumptions behind complexity analysis. |
| **Time Complexity vs. Auxiliary Space** | `Working Understanding` *(Block 1)* → `Practical Competence` *(Blocks 2, 6)* | Separates operation growth from extra working memory allocated beyond caller input. |
| **Asymptotic Notation (`O`, `Ω`, `Θ`)** | `Working Understanding` | Distinguishes upper bound, lower bound, and tight bound—independent of input case. |
| **Order of Growth & Dominant Term** | `Working Understanding` | Explains why higher growth classes crush lower ones once data scales up. |
| **Constant Factor & Crossover Point** | `Working Understanding` | Prevents assuming an asymptotically better class always wins on small or moderate inputs. |

### Mandatory Authoritative Resources *(Targeted Reading — ~45–55 min total)*

1. **Mandatory 1 (~25–30 min — assigned sections only):** Pat Morin, *Open Data Structures (in pseudocode)* (Carleton University) (`LO 1.1`, `LO 1.2`, `LO 1.3`, `LO 1.4`):
   - [Section 1.1 — The Need for Efficiency](https://opendatastructures.org/ods-python/1_1_Need_Efficiency.html)
   - [Section 1.3 — Mathematical Background](https://opendatastructures.org/ods-python/1_3_Mathematical_Background.html) — **কেবল `1.3.1` (*Exponentials and Logarithms*) এবং `1.3.3` (*Asymptotic Notation*) পড়ুন; `1.3.2` (*Factorials*) ও `1.3.4` (*Randomization and Probability*) explicitly skip করুন।**
   - [Section 1.4 — The Model of Computation](https://opendatastructures.org/ods-python/1_4_Model_Computation.html)
   - [Section 1.5 — Correctness, Time Complexity, and Space Complexity](https://opendatastructures.org/ods-python/1_5_Correctness_Time_Comple.html)
   - **What to extract:** Word-RAM model-এর মূল assumptions, base-2 logarithm ও asymptotic intuition, এবং correctness, time ও space complexity-র স্পষ্ট বিভাজন।
2. **Mandatory 2 (~20–25 min — assigned sections only):** MIT OpenCourseWare 6.006 *Introduction to Algorithms* (Spring 2020 — Jason Ku, Erik Demaine, Justin Solomon) — [Lecture 1: Introduction](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/mit6_006s20_lec1/) (`LO 1.1`, `LO 1.2`, `LO 1.3`):
   - **Assigned scope:** Lecture 1 Notes PDF-এর কেবল **Problems vs. Algorithms**, **Word-RAM Model**, এবং **Asymptotic Notation (`O`, `Ω`, `Θ`)** অংশটুকু পড়ুন।
   - **What to extract:** `O`, `Ω`, এবং `Θ`-র formal precision এবং কীভাবে computational problem থেকে algorithm-এর asymptotic bound লেখা হয়।

### Optional Supplementary Resource *(Outside Mandatory Budget — Do Not Treat as Mandatory)*

- **Supplementary (Commercial Book):** Thomas H. Cormen, Charles E. Leiserson, Ronald L. Rivest, Clifford Stein (CLRS), *Introduction to Algorithms* (4th ed., MIT Press, 2022) — Chapter 3 (*Characterizing Running Times*: `3.1 O, Ω, Θ-notation`, `3.2 Formal definitions`)। *(কেবল যাঁদের কাছে বইটি আছে এবং formal proofs দেখতে চান তাঁদের জন্য ঐচ্ছিক।)*

---

## Practical Exercises *(Mandatory — ~45–55 min total)*

> **Ownership Rule:** উভয় exercise-এর সব হিসাব, classification, ও ব্যাখ্যা স্বাধীনভাবে `phases/phase-02/notes/block-01-notes.md` ফাইলে লিখুন। এখানে কোনো answer key দেওয়া হয়নি।

### Exercise 1: Scaling Explosion & Crossover Math *(Mandatory — ~20–25 min | `LO 1.3`, `LO 1.4`)*

**Scenario:** আমাদের online store-এর flash sale-এর আগে একজন engineer order-matching operation-এর জন্য চারটি candidate algorithm-এর **abstract operation-count models** বের করেছেন (যেখানে `N` হলো match করার জন্য থাকা order-এর সংখ্যা):

- **Model A:** `500`
- **Model B:** `40 * log2(N)`
- **Model C:** `12 * N`
- **Model D:** `0.2 * N^2`

*(গুরুত্বপূর্ণ স্পষ্টীকরণ: ওপরের প্রতিটি expression হলো idealized word-RAM **abstract operation count**—এগুলো সরাসরি measured milliseconds বা wall-clock runtime নয়!)*

**Tasks (`phases/phase-02/notes/block-01-notes.md`):**
1. **Quantitative Growth Table:** নিচের পাঁচটি input size-এ চারটি model-এর প্রতিটির approximate operation count হিসাব করে একটি clean table তৈরি করুন:
   - `N = 50`
   - `N = 500`
   - `N = 5,000`
   - `N = 100,000`
   - `N = 1,000,000`
   *(Hint: `log2(50) ≈ 5.64`, `log2(500) ≈ 8.97`, `log2(5,000) ≈ 12.29`, `log2(100,000) ≈ 16.61`, `log2(1,000,000) ≈ 19.93`।)*
2. **Growth-Behavior Comparison:** `N = 50` থেকে `N = 1,000,000` (`20,000×` input growth) পর্যন্ত প্রতিটি model-এর operation count কত গুণ (ratio) বেড়েছে তা বের করুন এবং প্রতিটি model-এর tight asymptotic class (`Θ(...)`) লিখুন।
3. **Exact Positive Crossover Point:** **Model C (`12 * N`)** এবং **Model D (`0.2 * N^2`)**-এর মধ্যকার exact positive crossover point (`N > 0`) বীজগাণিতিক ধাপ দেখিয়ে নির্ণয় করুন। ওই crossover-এর নিচে এবং ওপরে কোন model কম abstract operations করে?
4. **Engineering Judgment & Model Limits:**
   - কোনো engineering team যদি শুধু `N = 50` দিয়ে staging test চালিয়ে Model C আর Model D তুলনা করে, তবে তারা flash sale-এর (`N = 100,000` বা `1,000,000`) আগে ঠিক কীভাবে বিভ্রান্ত হবে?
   - ২–৩ বাক্যে স্পষ্ট করে লিখুন: এই abstract operation counts আমাদের **কী বলে**, এবং production-এ actual milliseconds সম্পর্কে **কী বলে না**।

---

### Exercise 2: Technical Claim & Notation Audit *(Mandatory — ~25–30 min | `LO 1.1`, `LO 1.2`)*

**Scenario:** একটি backend team-এর PR review এবং Slack আলোচনা থেকে নিচের **৬টি realistic engineering statement** সংগ্রহ করা হয়েছে:

1. *"Big-O tells us our worst-case latency in milliseconds."*
2. *"Since `2N + 500` is `O(N^2)`, saying `O(N^2)` is technically false."*
3. *"Copying an array with `arr.map(...)` is `O(1)` space because we only used one variable."*
4. *"Scanning an array of `N` webhook payloads to count how many have `status === "FAILED"` (using one integer counter) has `O(N^2)` time complexity and `O(N)` auxiliary space."*
5. *"An algorithm whose abstract operation count is `Θ(N)` can still take longer to finish in production than a `Θ(N^2)` algorithm if `N` stays below their crossover point or if the `Θ(N)` implementation has much higher constant factors or per-item I/O overhead."*
6. *"Checking whether two `5 MB` JSON payload strings are equal with `payloadA === payloadB` takes `O(1)` time regardless of payload size because `===` is a single primitive operator in the word-RAM model."*

**Tasks (`phases/phase-02/notes/block-01-notes.md`):**
প্রতিটি statement-এর জন্য তিনটি কাজ করুন:
1. **Classification:** ঠিক একটি label বেছে নিন:
   - **`True`**
   - **`Technically True but Loose/Misleading`**
   - **`False`**
2. **Technical Explanation:** কেন ওই classification দিলেন (সংশ্লিষ্ট concept—`O`/`Ω`/`Θ`, input vs. auxiliary space, word-RAM fixed-word/uniform-memory limits, বা constant factors/crossover—উল্লেখ করে ২–৩ বাক্যে ব্যাখ্যা করুন)।
3. **Precise Engineering Rewrite:** Statement-টিকে সঠিক notation ও নির্ভুল engineering ভাষায় পুনরায় লিখুন।

---

## Closed-Book Synthesis Assessment *(Optional — ~20–25 min)*

এই assessment-টি সম্পূর্ণ **optional**—Block 2 শুরু করার জন্য এটি সম্পন্ন করা বা ChatGPT-র approval নেওয়া mandatory নয়। আপনি চাইলে:
- এই lesson ও সব browser tab বন্ধ রেখে নিচের ৫টি প্রশ্নের নিজের ভাষায় উত্তর `phases/phase-02/notes/block-01-notes.md` ফাইলে লিখে formal evaluation-এর জন্য **ChatGPT**-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে);
- অথবা আপনি যথেষ্ট বুঝেছেন মনে করলে assessment skip করে Block 2-এ এগোতে পারেন (`Studied — mastery not formally verified`);
- অথবা assessment না করেই যেকোনো unclear concept নিয়ে প্রশ্ন ও আলোচনা করতে পারেন।

*(Note: `~20–25 min` কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি approximate estimate—কোনো strict time limit নয়।)*

1. **(LO 1.1 / Outcome 1)** **Word-RAM model of computation** আসলে কী ধরে নেয়, computational complexity কী measure করে, আর একই code-এর efficiency বিচার করতে wall-clock milliseconds-এর বদলে abstract operation counts কেন বেশি নির্ভরযোগ্য? পাশাপাশি উদাহরণ দিয়ে বলুন কেন প্রতিটা problem-এ input size `N` স্পষ্টভাবে define করা জরুরি।
2. **(LO 1.1 / Outcome 1)** **Input space** এবং **auxiliary space**-এর পার্থক্য ব্যাখ্যা করুন। একটি বাস্তব JavaScript backend উদাহরণ দিয়ে দেখান—কেন মাত্র একটি variable ঘোষণা করার পরও কোনো function-এর auxiliary space `Θ(N)` হতে পারে।
3. **(LO 1.2 / Outcome 2)** Positive constants `c` ও `n0` ব্যবহার করে **`O`**, **`Ω`**, এবং **`Θ`**-র অর্থ ব্যাখ্যা করুন। গাণিতিকভাবে দেখান কেন `f(n) = 2n + 10` একইসাথে `O(n^2)` (technically true কিন্তু loose) এবং `Θ(n)` (tight bound)—এবং উদাহরণসহ explain করুন কেন asymptotic notation (`O`/`Ω`/`Θ`) ও input case (best/average/worst) দুটি সম্পূর্ণ আলাদা অক্ষ।
4. **(LO 1.3 / Outcome 2)** `O(1)`, `O(log N)`, `O(N)`, `O(N log N)`, `O(N^2)`, `O(N^3)`, এবং `O(2^N)`-এর growth behavior তুলনা করুন। Repeated halving দিয়ে `log2(N)`-এর intuition ব্যাখ্যা করুন, এবং একটি concrete crossover উদাহরণ দিয়ে দেখান—কখন growth rate জেতে আর কখন constant factors ও lower-order terms জেতে।
5. **(LO 1.4 / Outcomes 1, 2)** Asymptotic analysis-এর **চারটি মূল সীমাবদ্ধতা** (hidden constant factors, crossover regions, fixed-word assumption, এবং uniform-memory/no-I/O assumption) ব্যাখ্যা করে দেখান কেন `O(...)` একাই production backend-এর real latency predict করতে পারে না।

### Block 1 Readiness & Progression Checklist

- [ ] Guided lesson মনোযোগ দিয়ে পড়েছেন এবং Pause & Checks 1–3 `phases/phase-02/notes/block-01-notes.md`-এ লিখে (পরে Self-Check Guidance মিলিয়ে) সম্পন্ন করেছেন।
- [ ] দুটি Mandatory Reading (**Open Data Structures** `1.1`, `1.3.1`, `1.3.3`, `1.4`, `1.5` এবং **MIT 6.006 Lecture 1** নির্ধারিত অংশ) সম্পন্ন করেছেন।
- [ ] `phases/phase-02/notes/block-01-notes.md` ফাইলে স্বাধীনভাবে **Exercise 1 (Scaling Explosion & Crossover Math)** এবং **Exercise 2 (Technical Claim & Notation Audit)** সম্পন্ন করেছেন।
- [ ] *(Optional)* Notes/tabs বন্ধ রেখে Closed-Book Synthesis Assessment Questions 1–5 লিখেছেন এবং formal evaluation-এর জন্য ChatGPT-র কাছে share করেছেন।

> **Progression Status Distinction:** Block 2 কখন শুরু করবেন তা আপনার সিদ্ধান্ত। পড়া ও দুটি mandatory exercise শেষ করে optional assessment ছাড়া এগোলে এই block-টির status হবে **`Studied — mastery not formally verified`**। যথেষ্ট evidence share করে ChatGPT evaluation সম্পন্ন হলেই তা **`Mastery verified`** হিসেবে গণ্য হবে।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`phases/phase-02/notes/block-01-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের guidance দেখুন। Exercise 1, Exercise 2, বা Optional Closed-Book Assessment-এর কোনো solution এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও (চাইলে) optional ChatGPT review-এর জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* `1.2 ms` একটি single wall-clock runtime পরিমাপ, যা নির্দিষ্ট MacBook-এর CPU, OS load, runtime, এবং সম্ভবত একটি ছোট test input size-এর ওপর নির্ভরশীল। **Word-RAM model**-এ আমরা hardware-independent **abstract operation counts** input size `N`-এর ফাংশন হিসেবে দেখি; `N = 100`-এ `1.2 ms` নেওয়া একটা `Θ(N^2)` function `N = 100,000`-এ গিয়ে লাখো গুণ ধীর হয়ে timeout করতে পারে। তাই একক `1.2 ms` সংখ্যাটি scaling efficiency প্রমাণ করে না।
  2. *শক্তিশালী উত্তরে থাকবে:* দাবিটি **ভুল**। `N`টি order-এর array হলো caller-এর দেওয়া **input space**—function-টি নিজে শুধু দুটি local number variable ব্যবহার করেছে, যা `N` বড় হলেও স্থির থাকে। তাই এর **auxiliary space** হলো `Θ(1)` (constant), `Θ(N)` নয়—`N`টি element পড়া মানেই `Θ(N)` extra space allocate করা নয়। পাশাপাশি `N` কী নির্দেশ করছে (এখানে order-এর সংখ্যা) স্পষ্ট না করলে complexity দাবিটি অস্পষ্ট থাকে।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* (ক) `f(n) = 5n + 100`-এর জন্য: upper bound `O(n)` দেখাতে `c2 = 6, n0 = 100` নিলে সব `n >= 100`-এর জন্য `5n + 100 <= 5n + n = 6n`; আর lower bound `Ω(n)` দেখাতে `c1 = 5, n0 = 1` নিলে সব `n >= 1`-এর জন্য `5n <= 5n + 100`। অতএব সব `n >= 100`-এর জন্য `5n <= 5n + 100 <= 6n` → **`f(n) = Θ(n)`**। (খ) `f(n) = O(n^2)` গাণিতিকভাবে **সত্য** (যেমন `c = 1, n0 = 13` নিলে `5(13) + 100 = 165 <= 169 = 13^2` এবং সব `n >= 13`-এর জন্য সত্য), কিন্তু এটি **অত্যন্ত loose ও বিভ্রান্তিকর**। বিপরীতে `f(n) = Θ(n^2)` সম্পূর্ণ **মিথ্যা**, কারণ `5n + 100` কখনোই `Ω(n^2)` নয় (যেকোনো `c > 0`-এর জন্য বড় `n`-এ `c * n^2` একে ছাড়িয়ে যাবে)।
  2. *শক্তিশালী উত্তরে থাকবে:* মন্তব্যটি ভুল কারণ **input case** (best/average/worst) এবং **asymptotic bound** (`O`/`Ω`/`Θ`) দুটি সম্পূর্ণ স্বাধীন (orthogonal) ধারণা। `O` মানে স্রেফ upper bound (`<=`), worst-case নয়—তাই linear search-এর **best case** (`1` step)-এর upper bound `O(1)` (এবং tight bound `Θ(1)`), ঠিক যেমন তার **worst case** (`N` steps)-এর lower bound `Ω(N)` এবং tight bound `Θ(N)`!
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* `N = 16` হলে `log2(16) = 4`, তাই Approach X = `2,000 * 4 = 8,000` ops, আর Approach Y = `40 * 16 = 640` ops → এখানে `Θ(N)` Approach Y `12.5×` কম ops করছে! কিন্তু `N = 100,000` হলে `log2(100,000) ≈ 16.61`, তাই Approach X ≈ `33,220` ops, আর Approach Y = `4,000,000` ops → এখানে `Θ(log N)` Approach X `~120×` কম ops করছে। তাই ছোট `N`-এ ভারী constant factor (`2,000` বনাম `40`) asymptotic advantage-কে ছাপিয়ে যেতে পারে—শুধু `Θ(log N)` label দেখে সব `N`-এর জন্য দ্রুততর বলা ভুল।
  2. *শক্তিশালী উত্তরে থাকবে:* অন্তত দুটি valid কারণ: (ক) **Hidden constant factors** — একটি implementation হয়তো `2N` operations করে, অন্যটি `200N` operations (বা প্রতিটি iteration-এ heavy object allocation/serialization করে), যদিও দুটোই `Θ(N)`। (খ) **Uniform-memory / no-I/O assumption breakdown** — একটি implementation হয়তো RAM-এ contiguous memory পড়ে, আর অন্যটি প্রতিটি `N` item-এর জন্য disk read বা database/network round-trip করে (যার প্রতিটা millions of times slower), অথবা একটির প্রতিটা step-এ বড় string/payload comparison হয় যা fixed-word `O(1)` assumption ভাঙে।
