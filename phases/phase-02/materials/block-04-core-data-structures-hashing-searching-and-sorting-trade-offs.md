# Phase 2, Block 4 — Core Data Structures, Hashing, Searching, and Sorting Trade-Offs

## Study Time Guide

| Activity | Assigned Scope | Estimated Time |
| :--- | :--- | :---: |
| **Guided Lesson + Pause & Checks 1–3** *(main priority)* | পুরো guided lesson মনোযোগ দিয়ে পড়া এবং `phases/phase-02/notes/block-04-notes.md`-এ Pause & Checks 1–3 লেখা | **~60–75 min** *(flexible — limit নয়)* |
| **Mandatory 1:** MIT OCW 6.006 (Spring 2020) — *Lecture 4: Hashing* | Lecture 4 Notes PDF — *Direct Access Array*, *Hashing*, *Chaining*, *Universal/Good Hashing*, *Load Factor & Expected `O(1)`* | ~20–25 min |
| **Mandatory 2:** Princeton — *Algorithms, 4th ed.* Booksite (`3.1` & `3.4`) | Section `3.1` (*Sequential search* বনাম *Binary search in an ordered array* ও cost summary) + Section `3.4` (*Separate chaining* বনাম *Linear probing* conceptual trade-offs) | ~25–30 min |
| **Mandatory 3:** Pat Morin — *Open Data Structures* (`10.1` & `11.1`) | Section `10.1` (*BinaryHeap* implicit array layout ও `O(log N)` ops) + Section `11.1` (*Merge-Sort*, *Quicksort*, ও `Ω(N log N)` lower-bound intuition — formal proofs বাদ দিন) | ~20–25 min |
| **Exercise 1 (Mandatory):** Workload-to-Data-Structure Decision Matrix | ৫টি বাস্তব online-store backend workload-এর জন্য data structure নির্বাচন, complexity, ordering ও failure-mode বিশ্লেষণ | ~25–30 min |
| **Exercise 2 (Mandatory):** Search Strategy & Sorting Trade-Off Analysis | `(N, Q)` regime-এ linear scan বনাম sort+binary search বনাম hash map তুলনা, price-range shift, ও hybrid sort-এ insertion sort | ~25–30 min |
| **Total (Required Core — without Optional Assessment)** | Guided Lesson + ৩টি Mandatory Reading + ২টি Mandatory Exercise | **~175–215 min (~2.9–3.6 h)** |
| **Optional Closed-Book Assessment** *(optional)* | Notes/tabs বন্ধ রেখে ৫টি synthesis প্রশ্নের উত্তর লেখা | ~25–30 min *(optional)* |
| **Total (with Optional Assessment)** | Required Core + Optional Closed-Book Assessment | **~200–245 min (~3.3–4.1 h)** |

> **Planning Note:** Approved Phase 2 Research Dossier-এ Block 4-এর roadmap target allocation হলো **3.5 hours (210 minutes)**—যা একটি flexible planning reference, কোনো strict deadline নয়। দ্রুত শেষ করার চেয়ে প্রতিটা data structure ও sorting trade-off গভীরভাবে understand করাই main priority; বিশেষ করে **Guided Lesson**-এ প্রয়োজনে আনুমানিক সময়ের চেয়ে বেশি সময় লাগা একদম স্বাভাবিক। প্রতিটি reading estimate কেবল নির্ধারিত sections-এর জন্য, পুরো বই বা course পড়ার জন্য নয়। **Optional Closed-Book Assessment** সম্পূর্ণ optional এবং Block 5 শুরু করার জন্য কোনো mandatory gate নয়।

---

Block 3-এ আপনি শিখেছেন একই algorithm কীভাবে best, average, expected, worst-case, আর amortized লেন্সে ভিন্ন ভিন্ন ঝুঁকি দেখায়। এখন আমাদের online store-এর catalog ও order service-এ এক সপ্তাহের পাঁচটি বাস্তব feature request একসাথে আপনার টেবিলে এল:

1. **Promo Validation:** প্রতি মিনিটে লাখ লাখ checkout-এ exact `promoCode` দিয়ে discount rule খুঁজে বের করা।
2. **Catalog Price Filter:** *"দাম `USD 10` থেকে `USD 25`-এর মধ্যে থাকা সব product দাম অনুযায়ী সাজিয়ে দেখাও"*—এবং মাঝে মাঝে নতুন product-ও যোগ হবে।
3. **Streaming VIP Leaderboard:** `2,000,000` order-এর stream থেকে কেবল **সর্বোচ্চ খরচ করা Top-10 customer** ধরে রাখা—সব `2,000,000` record একসাথে sorted রেখে RAM না ফুরিয়ে।
4. **Order Export Ordering:** Order export-কে `status` অনুযায়ী সাজানো, কিন্তু একই `status`-এর ভেতর আগে থেকে থাকা `createdAt` chronological ক্রম অটুট রাখা।
5. **Sequential Line-Item Processing:** একটি order-এর সব item একে একে append করা এবং দ্রুত iterate করা।

একটু থেমে দেখুন: **একই একটিমাত্র data structure দিয়ে কি এই পাঁচটি কাজই efficiently করা সম্ভব?**
- সবকিছু plain array-তে রাখলে exact lookup ও range update ধীর হয়ে যাবে (`O(N)`)।
- সবকিছু hash table (`Map`/`Set`)-এ রাখলে exact lookup ঝড়ের গতিতে চলবে, কিন্তু `USD 10`–`USD 25` range query বা Top-10 বের করতে পুরো table scan করতে হবে!
- সবকিছু প্রতিবার sort করে রাখলে `O(N log N)` CPU ও `O(N)` memory নষ্ট হবে যখন আমাদের দরকার ছিল মাত্র Top-10!

এটাই Block 4-এর কেন্দ্রীয় engineering প্রশ্ন:

> **Contiguous dynamic arrays, linked lists, hash tables, sorted arrays, balanced search trees, binary heaps, এবং core sorting strategies ভেতরে ঠিক কীভাবে data সাজায়—যার ফলে কোনো structure exact lookup-এ, কোনোটা ordered range-এ, আর কোনোটা Top-K-এ জেতে, এবং প্রতিটি পছন্দে time বনাম space-এর আসল trade-off কী?**

চলুন সবচেয়ে মৌলিক দুটি linear building block দিয়ে শুরু করি—যেখানে প্রায় সব beginner একটি প্রচলিত `O(1)` ফাঁদে পা দেন।

---

## ১. Contiguous Dynamic Arrays বনাম Linked Lists: "মাঝখানে `O(1)` Insert" মিথের সত্যতা

যেকোনো sequence data memory-তে রাখার দুটি মৌলিক কাঠামো আছে:

1. **Contiguous Dynamic Array:** সব element memory-তে **পাশাপাশি (contiguous)** একটি ধারাবাহিক block-এ সাজানো থাকে। `i`-তম element-এর ঠিকানা সরাসরি অঙ্ক কষে বের করা যায় (`baseAddress + i * slotSize`)—তাই index দিয়ে সরাসরি access (`arr[i]`) হয় **`Θ(1)`**! *(Block 3-এ আমরা দেখেছি, capacity পূর্ণ হলে geometric doubling-এ append হয় **`O(1)` amortized**, যদিও single resize-এ `O(N)` copy spike হতে পারে।)*
2. **Linked List (Singly বা Doubly):** প্রতিটি element আলাদা একটি **node object** হিসেবে heap memory-র যেকোনো খালি জায়গায় থাকে, এবং প্রতিটি node তার পরের node-এর memory reference (`next` pointer)—আর **doubly linked list** হলে আগের node-এর reference-ও (`prev` pointer)—ধরে রাখে।

```text
Contiguous Dynamic Array (পাশাপাশি memory slots — O(1) index math):
  index:   0      1      2      3      4
        +------+------+------+------+------+
        |  10  |  25  |  40  |  55  |  70  |
        +------+------+------+------+------+

Doubly Linked List (ছড়ানো node objects + prev/next pointers):
  head -> [null | 10 | *] <--> [* | 25 | *] <--> [* | 40 | null] <- tail
```

এবার সবচেয়ে জরুরি qualification-টি দেখুন, যা ছাড়া linked-list complexity-র তুলনা ভুল হয়ে যায়:

> **Linked list-এ `O(1)` insertion বা deletion কেবল তখনই সত্য যখন আপনার হাতে ইতিমধ্যেই target node (অথবা singly linked list-এ তার ঠিক আগের predecessor node)-এর সরাসরি reference থাকে!**
> যদি আপনাকে index `i` দিয়ে বা `itemId` দিয়ে ওই position খুঁজে বের করতে হয়, তবে `head` থেকে pointer ধরে হাঁটতে **`O(N)` traversal** লাগবেই—সুতরাং *খুঁজে নিয়ে* insert/delete করলে মোট খরচ **`O(N)`**, `O(1)` নয়!

পাশাপাশি exact operation-cost এবং memory-overhead তুলনা দেখুন (`N` = বর্তমান element সংখ্যা):

| Operation | Contiguous Dynamic Array | Singly Linked List (`head` + `tail`) | Doubly Linked List (`head` + `tail`) |
| :--- | :---: | :---: | :---: |
| **Index access (`get(i)`)** | **`Θ(1)`** | `O(N)` *(worst `Θ(N)` from `head`)* | `O(N)` *(worst `Θ(N)` from `head`/`tail`)* |
| **Unsorted search by value** | `O(N)` *(worst `Θ(N)`)* | `O(N)` *(worst `Θ(N)`)* | `O(N)` *(worst `Θ(N)`)* |
| **Append at tail (`push`)** | **`O(1)` amortized** *(worst `Θ(N)` resize)* | **`Θ(1)`** *(with `tail` pointer)* | **`Θ(1)`** *(with `tail` pointer)* |
| **Remove from tail (`pop`)** | **`O(1)` amortized** *(with hysteresis)* | **`Θ(N)`** *(prev node নেই!)* | **`Θ(1)`** *(via `tail.prev`)* |
| **Insert / delete at front** | **`Θ(N)`** *(সব element shift হয়)* | **`Θ(1)`** *(via `head`)* | **`Θ(1)`** *(via `head`)* |
| **Insert / delete in middle (position জানা থাকলে)** | `O(N)` *(worst `Θ(N)` shift)* | **`Θ(1)`** *(predecessor node হাতে থাকলে!)* | **`Θ(1)`** *(target node হাতে থাকলে!)* |
| **Insert / delete in middle (খুঁজে বের করতে হলে)** | `O(N)` *(worst `Θ(N)`)* | `O(N)` *(খুঁজতে `O(N)` + unlink `O(1)`)* | `O(N)` *(খুঁজতে `O(N)` + unlink `O(1)`)* |
| **Per-element memory overhead** | কম (শুধু unused capacity buffer) | বেশি (প্রতি node-এ object + `next` pointer) | আরও বেশি (object + `prev` ও `next` pointer) |

### তাহলে Sequential Scan দুটোরই `Θ(N)` হওয়া সত্ত্বেও Backend-এ Array কেন Dominate করে?

আপনি ভাবতে পারেন: *"Order-এর সব item যোগ করতে array আর linked list দুটোরই তো `Θ(N)`—তাহলে বাস্তবে array প্রায় সবসময় এত বেশি fast হয় কেন?"*

এর পেছনে দুটি বড় কারণ আছে *(CPU cache ও memory hierarchy-র exact latency সংখ্যা আমরা Block 5-এ বিস্তারিত পড়ব; এখানে শুধু core intuition-টুকু বুঝুন)*:
1. **Spatial Locality (পাশাপাশি থাকার সুবিধা):** আধুনিক CPU memory থেকে একটিমাত্র byte না এনে আশপাশের পুরো এক টুকরো contiguous memory (**cache line**) একসাথে দ্রুত L1/L2 cache-এ তুলে আনে। Array-র elements পাশাপাশি থাকায় `arr[0]` পড়ার সাথেই `arr[1], arr[2], ...` আগেই cache-এ চলে আসে! কিন্তু linked list-এর node-গুলো heap-এ ছড়ানো থাকে—প্রতিটা `.next` অনুসরণ করতে গিয়ে CPU-কে বারবার ভিন্ন memory ঠিকানায় লাফাতে (**pointer chasing**) হয়।
2. **কম Memory Overhead:** Linked list-এ প্রতিটি ছোট সংখ্যার জন্যও আলাদা node object ও `next`/`prev` reference রাখতে হয়, যা বেশি RAM খায় এবং cache-এ কম element ধরে।

*(FIFO Queue নোট: যদি এমন queue লাগে যেখানে শুধু tail-এ enqueue আর head-এ dequeue হয়, তবে plain array-তে `.shift()` করলে `Θ(N)` shift খরচ পড়ে—তাই সেখানে head/tail index-সহ **circular buffer (ring buffer)** dynamic array অথবা linked-list queue ব্যবহার করে `O(1)` enqueue/dequeue পাওয়া যায়!)*

কিন্তু array হোক বা linked list—একটি unsorted sequence-এ `promoCode` দিয়ে exact lookup করতে গেলে এখনো পুরো `N`টি element scan (`O(N)`) করতেই হচ্ছে! **আমরা কি `N` যত বড়ই হোক না কেন, প্রায় এক ধাপে (`expected O(1)`) exact key খুঁজে বের করতে পারি?**

---

## ২. Hash Tables: Exact-Key Lookup-এর যন্ত্র, Load Factor, এবং এর স্পষ্ট সীমারেখা

ধরুন আপনার store-এ `500,000`টি active promo code আছে। যদি প্রতিটি promo code-কে একটি বিশাল direct array-তে সরাসরি index বানিয়ে রাখা যেত, তাহলে lookup হতো `Θ(1)`—কিন্তু string key (`"EID2026"`, `"WELCOME50"`) বা `64`-bit user ID-র সব সম্ভাব্য মানের জন্য direct array বানাতে গেলে পৃথিবীর সব RAM-ও কুলোবে না! *(Mandatory 1: MIT 6.006 Lecture 4 — Direct Access Array বনাম Hashing।)*

এই সমস্যার সমাধানই হলো **Hash Table**:
1. একটি নির্দিষ্ট সাইজ **`m`**-এর **bucket array** নিন (যেখানে `m` হলো stored key সংখ্যা **`n`**-এর সমানুপাতিক, যেমন `m ≈ 2n`)।
2. একটি **Hash Function `h(key)`** ব্যবহার করে যেকোনো key-কে একটি integer-এ রূপান্তর করে bucket index `0 .. m - 1`-এ ম্যাপ করুন (যেমন `bucketIndex = h(key) mod m`)।
3. **Load Factor `α = n / m`:** প্রতি bucket-এ গড়ে কতগুলো element আছে, সেটাই load factor! `α` ছোট ও bounded রাখলে প্রতিটি bucket-এ গড়ে খুব অল্প (ধ্রুবক সংখ্যক) key থাকে।

### যখন দুটি ভিন্ন Key একই Bucket-এ পড়ে: Collision Resolution-এর দুটি প্রধান কৌশল

যেহেতু সম্ভাব্য key-র সংখ্যা bucket সংখ্যা `m`-এর চেয়ে অসীম বেশি, তাই কখনো না কখনো দুটি ভিন্ন key একই bucket index-এ পড়বেই—একে বলে **Collision**। Abstract hash-table design-এ collision সামলানোর দুটি প্রধান পদ্ধতি আছে *(Mandatory 2: Princeton `3.4`)*:

1. **Separate Chaining:** প্রতিটি bucket-এ একটি ছোট list/chain থাকে; একই bucket-এ পড়া সব key সেই chain-এ রাখা হয়।
   - *সুবিধা:* `α > 1` হলেও table কাজ চালিয়ে যেতে পারে; delete করা সহজ।
   - *অসুবিধা:* প্রতিটি entry-র জন্য আলাদা node/pointer overhead ও ছড়ানো memory allocation লাগে।
2. **Open Addressing (বিশেষত Linear Probing):** আলাদা chain নেই—সব key সরাসরি bucket array-র slot-গুলোতেই থাকে! যদি `bucketIndex` আগে থেকেই দখল থাকে, তবে `(bucketIndex + 1) mod m`, `(bucketIndex + 2) mod m`... করে পরের খালি slot খুঁজে নেওয়া হয়।
   - *সুবিধা:* কোনো pointer chasing নেই—contiguous array হওয়ায় cache locality দারুণ!
   - *অসুবিধা:* অবশ্যই **`α < 1`** বজায় রাখতে হয় (table ভরে গেলে আর কোনো খালি slot থাকে না!)। তাছাড়া `α` যখন `1`-এর কাছাকাছি পৌঁছায় (যেমন `> 0.7`), তখন দখল হওয়া slot-গুলো জমে লম্বা ব্লক তৈরি করে—যাকে বলে **Primary Clustering**—ফলে probe length দ্রুত বেড়ে যায়! *(Deletion-ও জটিল, কারণ মাঝখান থেকে slot সরাসরি খালি করলে পরের probe chain ভেঙে যেতে পারে—তাই tombstone marker বা re-shifting লাগে।)*

### Resizing এবং Rehashing: কেন Load Factor Bounded রাখতে হয়?
Table-এ `n` বাড়তে বাড়তে `α = n / m` যদি নির্ধারিত threshold পার করে (যেমন `α > 0.75`), তবে bucket সংখ্যা **`m` বাড়িয়ে (যেমন `2m`)** একটি নতুন array allocate করতে হয় এবং আগের সব `n`টি key-কে নতুন `mod 2m` অনুযায়ী পুনরায় hash করে বসাতে হয়—একে বলে **Rehashing**। Block 3-এর dynamic array doubling-এর মতোই, ভালো hash distribution-এর অধীনে একটি individual rehash নেয় **`O(N)` expected** সময় (সম্ভাব্য latency spike!), কিন্তু geometric growth-এর কারণে পুরো sequence জুড়ে এর **expected amortized cost থাকে `O(1)`**!

### অত্যন্ত জরুরি Precision Rule: Conventional Cost Model বনাম Runtime Reality

Hash table-এর complexity বলার সময় দুটি সতর্কতা কখনোই ভুলবেন না:
1. **Expected `O(1)` হলো Conventional Abstract Hash-Table Cost Model—Unconditional Runtime Guarantee নয়!**
   - Exact lookup, insert, এবং delete-এ **expected `O(1)`** পাওয়া যায় *এই শর্তে* যে: (ক) hash function key-গুলোকে bucket-গুলোতে ভালোভাবে ছড়িয়ে দিচ্ছে (যেমন universal/well-distributed hashing), এবং (খ) dynamic resizing দিয়ে load factor `α = n / m` bounded রাখা হচ্ছে।
   - Block 3-এ আমরা দেখেছি, এই শর্ত ভাঙলে (যেমন duplicate-checking chaining বা linear probing-এ সব key এক bucket/cluster-এ collide করলে) worst-case lookup `Θ(N)` এবং `N` operation `Θ(N^2)` হয়ে যেতে পারে। তাছাড়া expected `O(1)` একাই wall-clock SLA বা per-run latency guarantee করে না।
2. **Abstract Hash Table ≠ V8 `Map` / `Set` Implementation Internals:**
   - JavaScript/Node.js backend code-এ আমরা exact-key lookup ও deduplication-এর জন্য built-in **`Map`** এবং **`Set`** ব্যবহার করি (নিজে হাত দিয়ে hash table লিখি না)।
   - তবে আধুনিক JS engine (যেমন V8) ঠিক কোন internal hash table, probing strategy, insertion-order preservation, বা plain-object hidden class ব্যবহার করে—সেই **runtime-specific internals আমরা Phase 5-এ শিখব**। এখানে abstract hash-table model এবং `Map`/`Set`-এর ব্যবহারকে runtime internals থেকে আলাদা রাখুন।

### বাস্তব Node.js প্রয়োগ: `O(N * M)` থেকে `O(N + M)` (Time-Space Trade-Off)

ধরুন আমাদের backend-এ `N`টি order-কে `M`টি customer profile-এর সাথে `customerId` ধরে যুক্ত করতে হবে। Naive approach-এ প্রতিটি order-এর জন্য `customers.find(...)` চালালে cost হয় **`O(N * M)`**। কিন্তু আমরা যদি `O(M)` extra memory খরচ করে একবারে একটি `Map` index তৈরি করি:

```javascript
// Conventional hash-table cost model:
// Build Map: O(M) expected time, Θ(M) auxiliary space
// Enrich N orders: O(N) expected time -> Total: O(N + M) expected time, Θ(N + M) auxiliary space (including output)
function enrichOrdersWithCustomers(orders, customers) {
  const customerById = new Map();
  for (const customer of customers) {
    customerById.set(customer.id, customer);
  }

  const enriched = [];
  for (const order of orders) {
    const profile = customerById.get(order.customerId) ?? null;
    enriched.push({
      orderId: order.id,
      amountCents: order.amountCents,
      customerTier: profile ? profile.tier : "GUEST",
    });
  }
  return enriched;
}

// Event deduplication using Set: O(N) expected time, O(N) auxiliary space (replacing O(N^2) Array.includes!)
function deduplicateEventIds(eventIds) {
  const seen = new Set();
  const unique = [];
  for (const id of eventIds) {
    if (!seen.has(id)) {
      seen.add(id);
      unique.push(id);
    }
  }
  return unique;
}
```

### Hash Table-এর দুটি বড় সীমাবদ্ধতা (যেখানে এটি হেরে যায়!)
1. **Memory Overhead:** Load factor কম রাখতে bucket array-তে সবসময় উল্লেখযোগ্য খালি জায়গা রাখতে হয় (পাশাপাশি entry/hash metadata)। তাই একই `N` সংখ্যক item-এর জন্য hash table সাধারণত plain array-র চেয়ে বেশি RAM নেয়।
2. **Ordered Range Query করতে পারে না!** একটি ভালো hash function ইচ্ছাকৃতভাবে কাছাকাছি মানের key-গুলোকেও (`price = 1000`, `1001`, `1002`) পুরোপুরি ভিন্ন, এলোমেলো bucket-এ ছড়িয়ে দেয়! ফলে hash table-এ কোনো **natural sort order** থাকে না। আপনি যদি জানতে চান *"দাম `USD 10` থেকে `USD 25`-এর মধ্যে সব product দাও"* বা *"সবচেয়ে কম দামের product কোনটি?"*—তাহলে hash table দিয়ে পুরো `N`টি entry scan (`Θ(N)`) করা ছাড়া উপায় নেই!

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-04-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন developer বললেন: *"আমাদের order-items list-এ মাঝখান থেকে মাঝে মাঝে item delete করতে হয়, তাই dynamic array বাদ দিয়ে singly linked list নিই—linked list-এ তো deletion `O(1)`!"* যদি caller একটি `itemId` বা index `i` দিয়ে ওই item-টি delete করতে চায়, তবে এই যুক্তিতে ঠিক কোন ভুল আছে—এবং ঠিক কোন শর্ত পূরণ হলে linked-list deletion সত্যিই `O(1)` হয়?
> 2. একটি analytics service `N`টি incoming log থেকে unique `sessionId` গুনতে `Set` ব্যবহার করছে। (ক) Conventional hash-table cost model অনুযায়ী এর expected time ও auxiliary space কী, এবং কেন এই expected `O(1)` lookup-কে unconditional runtime guarantee বলা ভুল? (খ) পরদিন product যদি চায় *"গত ১০ মিনিটের `timestamp` রেঞ্জে থাকা সব session chronological ক্রমে দাও"*, তবে শুধু ওই hash table দিয়ে এটি কেন efficiently সম্ভব নয়?

তাহলে প্রশ্ন দাঁড়াল: **যখন আমাদের exact lookup নয়, বরং ordered range query (`USD 10`–`USD 25`), `min`/`max`, বা sorted iteration দরকার হয়—তখন কোন data structure ব্যবহার করব?**

---

## ৩. Searching ও Ordered Structures: Sorted Arrays বনাম Balanced Search Trees

Ordering বজায় রেখে দ্রুত খোঁজার তিনটি ধাপ একসাথে দেখুন *(Mandatory 2: Princeton `3.1`)*:

### ধাপ ১: Unsorted Array Scan — `O(N)`
কোনো সাজানো কাঠামো নেই। Insert (append) `O(1)` amortized, কিন্তু search, min/max, বা range query সবই **`Θ(N)`**।

### ধাপ ২: Sorted Array + Binary Search — `O(log N)` Search, কিন্তু `O(N)` Insert!
যদি catalog-এর product-গুলো `priceCents` অনুযায়ী একটি contiguous array-তে আগে থেকেই **sorted** রাখা থাকে:
- **Binary Search:** প্রতি ধাপে search range অর্ধেক করে **`O(log N)`** (worst-case `Θ(log N)`) সময়ে exact value বা range-এর শুরু (`lower_bound`, যেমন `>= USD 10`) খুঁজে পাওয়া যায়!
- **Range Query:** শুরুর index পেলে সেখান থেকে `<= USD 25` পর্যন্ত contiguous scan করে `K`টি matching item বের করা যায় **`O(log N + K)`** সময়ে!
- **Min / Max:** সরাসরি `arr[0]` ও `arr[N - 1]` → **`Θ(1)`**!
- **কিন্তু বড় সীমাবদ্ধতা (Static-Heavy Only!):** মাঝে একটিমাত্র নতুন product insert করতে গেলে (বা delete করতে গেলে) sorted order অটুট রাখতে পরের সব element এক ঘর করে সরাতে (**shift**) হয় → **`Θ(N)`**! তাই read-heavy static catalog-এ sorted array দারুণ, কিন্তু ঘন ঘন insert/delete হওয়া dynamic catalog-এ এটি ভেঙে পড়ে।

### ধাপ ৩: Binary Search Trees (BST) ও Balanced Search Trees

Dynamic insert/delete *এবং* `O(log N)` ordered search—দুটোই একসাথে পেতে হলে আমরা linear array-র বদলে একটি **Binary Search Tree (BST)** তৈরি করি।

**BST Invariant (মূল শর্ত):** প্রতিটি node `x`-এর জন্য—
- `x`-এর **left subtree**-র সব key `< x.key`, এবং
- `x`-এর **right subtree**-র সব key `> x.key`।

এই invariant-এর কারণে root থেকে নিচে নামতে নামতে প্রতি ধাপে আমরা বাম বা ডান দিক বেছে নিই—তাই search, insert, delete-এর খরচ সমানুপাতিক হয় tree-র **height `h`**-এর (`O(h)`)।

**কিন্তু Unbalanced BST-র ভয়ংকর Failure Mode:**
ধরুন আপনি একটি plain (unbalanced) BST-তে আজকের order-গুলো তাদের ক্রমবর্ধমান `orderId` (`101, 102, 103, 104, 105`) অনুযায়ী একটার পর একটা insert করলেন! কী আকৃতি তৈরি হবে?

```text
Unbalanced BST on sorted inserts (height h = N -> O(N) linked chain!):
  101
    \
    102
      \
      103
        \
        104
          \
          105

Balanced Search Tree on the same 5 keys (height h = O(log N)):
        103
       /   \
     101   104
       \     \
       102   105
```

দেখলেন? Sorted input পেলে plain BST একদিকে হেলে পড়ে একটি সাধারণ **linked list** হয়ে যায় (`height h = N`), ফলে search/insert/delete ভেঙে পড়ে **`Θ(N)`** হয়ে যায়!

**Balanced Search Trees ঠিক কী সমাধান করে (`Working Understanding`):**
- Insert এবং delete-এর সময় structural rebalancing rules বজায় রেখে tree-র height সবসময় **`h = O(log N)`** সীমার ভেতর আটকে রাখে—input যে ক্রমেই আসুক না কেন!
- ফলে আপনি একসাথে পান:
  - **Search / Insert / Delete:** **`O(log N)`** (deterministic worst-case!)
  - **Min / Max / Predecessor / Successor:** **`O(log N)`**
  - **Ordered Range Query (`T1 <= key <= T2`, returning `K` items):** **`O(log N + K)`**!
  - **Sorted Traversal (In-order):** **`Θ(N)`**।

*(**`Awareness` Only — Ecosystem Bridge:** মেমোরিতে balanced binary search tree হিসেবে **AVL Trees** এবং **Red-Black Trees** ব্যবহৃত হয়, আর disk/SSD-ভিত্তিক database engine (যেমন PostgreSQL)-এ প্রতিটি node-এ বহু key রেখে tree height অত্যন্ত ছোট রাখার জন্য **B-Trees / B+Trees** ব্যবহৃত হয়! এদের manual rotation proofs বা PostgreSQL B-Tree page mechanics এই block-এর বাইরে—B-Tree storage ও index plans আমরা Phase 10-এ গভীরভাবে পড়ব।)*

এবার আমাদের ৩ নম্বর feature request-টার দিকে তাকান: **Streaming VIP Leaderboard (`2,000,000` order থেকে কেবল Top-10 customer)**!
এখানে আমাদের পুরো `2,000,000` customer-এর arbitrary range query বা পুরো sorted order দরকার নেই—দরকার কেবল প্রতি মুহূর্তে **সবচেয়ে ছোট বা সবচেয়ে বড় একটিমাত্র element (`min` বা `max`)** দেখা ও বদলানো! এর জন্য কি পুরো balanced search tree-র জটিলতা ও pointer overhead লাগবে? **না!** এখানেই আসে একটি অসাধারণ, pointer-মুক্ত data structure: **Binary Heap**!

---

## ৪. Binary Heaps ও Priority Queues: Pointer ছাড়া `O(log N)` এবং `O(N log K)` Top-K

একটি **Priority Queue** হলো এমন একটি abstract interface যেখানে যেকোনো সময় নতুন item যোগ করা (`push` / `insert`) যায়, কিন্তু বের করার সময় (`pop` / `extractMin` বা `extractMax`) সবসময় সর্বোচ্চ priority (যেমন সবচেয়ে ছোট `runAt` timestamp বা সবচেয়ে কম spend)-এর item-টি আগে বের হয়।

এটিকে বাস্তবায়ন করার সবচেয়ে জনপ্রিয় কাঠামো হলো **Binary Heap** *(Mandatory 3: Pat Morin, Open Data Structures `10.1`)*—যার দুটি সহজ invariant আছে:
1. **Shape Invariant (Complete Binary Tree):** Tree-র শেষ level ছাড়া সব level সম্পূর্ণ ভর্তি, আর শেষ level-এ nodes বাম থেকে ডানে সাজানো—তাই `N`টি node-এর height সবসময় ঠিক **`floor(log2 N) = Θ(log N)`**!
2. **Heap-Order Invariant (Min-Heap):** প্রতিটি parent node-এর key তার দুই child-এর key-র চেয়ে ছোট বা সমান (`parent <= leftChild` এবং `parent <= rightChild`)। *(Max-Heap-এ ঠিক উল্টো: `parent >= children`।)* ফলে **সবচেয়ে ছোট মানটি সবসময় `root`-এ বসে থাকে!**

### Implicit Array Layout (কোনো `left`/`right` Pointer-ই নেই!)
Complete binary tree হওয়ায় আমরা node-গুলোকে level-by-level একটি **flat contiguous array**-তে সাজিয়ে ফেলতে পারি—একটাও node object বা pointer ছাড়াই!

```text
Min-Heap Tree View:                 Implicit 0-Based Array View:
          (10)                      index:  0    1    2    3    4    5
         /    \                           +----+----+----+----+----+----+
       (15)   (20)                        | 10 | 15 | 20 | 30 | 40 | 25 |
      /    \   /                          +----+----+----+----+----+----+
    (30) (40) (25)
```

`0`-based array index `i`-এর জন্য parent ও children সরাসরি সহজ পাটিগণিতে পাওয়া যায়:
- **`parent(i) = Math.floor((i - 1) / 2)`** *(যখন `i > 0`)*
- **`left(i)   = 2 * i + 1`**
- **`right(i)  = 2 * i + 2`**

*(নিজেই ছবিটার সাথে মিলিয়ে নিন: index `1` (`15`)-এর `left = 3` (`30`), `right = 4` (`40`), `parent = 0` (`10`)!)*

### Heap Operations কীভাবে `O(1)` ও `O(log N)` সময়ে চলে?
- **`peek()` → `Θ(1)`:** শুধু `heap[0]` return করুন!
- **`push(item)` → `O(log N)`:** নতুন item-কে array-র একদম শেষে বসিয়ে দিন (shape invariant অটুট থাকল), তারপর যতক্ষণ সে তার `parent`-এর চেয়ে ছোট, ততক্ষণ `parent`-এর সাথে swap করে ওপরে তুলুন (**bubble-up / sift-up**)। সর্বোচ্চ tree height `floor(log2 N)` ধাপ! *(Underlying dynamic array grow হলে rare resize বাদে `O(log N)`।)*
- **`popMin()` → `O(log N)`:** `heap[0]` (সর্বনিম্ন মান) সংরক্ষণ করুন, array-র একদম শেষের item-টিকে `heap[0]`-এ এনে array length `1` কমান, তারপর যতক্ষণ সে তার ছোট child-এর চেয়ে বড়, ততক্ষণ সেই ছোট child-এর সাথে swap করে নিচে নামান (**bubble-down / sift-down**)। সর্বোচ্চ `floor(log2 N)` ধাপ!

### বাস্তব Node.js প্রয়োগ: `O(N log K)` Streaming Top-K (কেন Top-K Largest-এ **Min-Heap**?)

অনেকে প্রথমবার অবাক হন: *"সর্বোচ্চ `K` জন VIP customer বের করতে **Max-Heap** না নিয়ে **Min-Heap** কেন নিচ্ছি?!"*
লজিকটা দেখুন—দারুণ চতুর:
- আমরা heap-এর সাইজ কখনো **`K`-এর বড় হতে দেব না**।
- `K`-সাইজের একটি **Min-Heap**-এর `root` (`heap[0]`)-এ বসে থাকে আমাদের বর্তমান Top-`K` তালিকার **সবচেয়ে দুর্বল সদস্যটি (cutoff threshold)**!
- নতুন কোনো customer এলে:
  - যদি তার spend `<= heap[0]` হয় → সে বর্তমান Top-`K`-এর সবচেয়ে দুর্বল সদস্যকেও হারাতে পারল না! **`O(1)` সময়ে সরাসরি বাতিল!**
  - যদি তার spend `> heap[0]` হয় → সে Top-`K`-এ ঢোকার যোগ্য! `root`-কে সরিয়ে তাকে ঢুকিয়ে `sift-down` করে নিই → মাত্র **`O(log K)`**!

নিচের self-contained, runnable Node.js implementation-টি দেখুন:

```javascript
class MinHeap {
  constructor(compareFn) {
    this.items = [];
    this.compare = compareFn;
  }

  get size() {
    return this.items.length;
  }

  peek() {
    return this.items.length === 0 ? undefined : this.items[0];
  }

  push(value) {
    this.items.push(value);
    this._siftUp(this.items.length - 1);
  }

  popMin() {
    if (this.items.length === 0) return undefined;
    const min = this.items[0];
    const last = this.items.pop();
    if (this.items.length > 0) {
      this.items[0] = last;
      this._siftDown(0);
    }
    return min;
  }

  _siftUp(idx) {
    while (idx > 0) {
      const parent = Math.floor((idx - 1) / 2);
      if (this.compare(this.items[idx], this.items[parent]) < 0) {
        [this.items[idx], this.items[parent]] = [this.items[parent], this.items[idx]];
        idx = parent;
      } else {
        break;
      }
    }
  }

  _siftDown(idx) {
    const n = this.items.length;
    while (true) {
      const left = 2 * idx + 1;
      const right = 2 * idx + 2;
      let smallest = idx;
      if (left < n && this.compare(this.items[left], this.items[smallest]) < 0) {
        smallest = left;
      }
      if (right < n && this.compare(this.items[right], this.items[smallest]) < 0) {
        smallest = right;
      }
      if (smallest === idx) break;
      [this.items[idx], this.items[smallest]] = [this.items[smallest], this.items[idx]];
      idx = smallest;
    }
  }
}

// Top-K highest spenders from N streaming records:
// Time: O(N log K) upper bound for K >= 2 (O(N) for K = 1), Auxiliary Space: O(K)
function topKCustomersBySpend(customerStream, k) {
  if (k <= 0) return [];
  const minHeap = new MinHeap((a, b) => a.totalCents - b.totalCents);

  for (const record of customerStream) {
    if (minHeap.size < k) {
      minHeap.push(record);
    } else if (record.totalCents > minHeap.peek().totalCents) {
      minHeap.popMin();
      minHeap.push(record);
    }
  }

  // Extract K winners (highest first): O(K log K)
  const winners = [];
  while (minHeap.size > 0) {
    winners.push(minHeap.popMin());
  }
  return winners.reverse();
}
```

### কখন Heap-ভিত্তিক Top-K (`O(N log K)`) পুরো Sorting (`O(N log N)`)-কে হারিয়ে দেয়?
- **`K << N` হলে (যেমন `N = 2,000,000`, `K = 10`):**
  - **Full Sort:** `O(N log N)` time (`~2,000,000 × 21`), এবং পুরো `2,000,000` record একসাথে RAM-এ ধরে রাখতে **`Θ(N)` space**!
  - **Min-Heap Top-K:** `O(N log K)` worst-case time (`log2(10) ≈ 3.3`—অনেক কম কাজ; এমনকি অনেক বাস্তব unordered stream-এ অধিকাংশ item `peek()`-এ `O(1)`-এই বাদ পড়ে, আর worst-case ascending stream-এও প্রতি item-এ সর্বোচ্চ `O(log K)` লাগে!), আর RAM-এ একসাথে থাকে মাত্র **`K = 10`টি record (`O(K)` space)**—ফলে বিশাল streaming dataset-এও OOM হওয়ার ভয় থাকে না!
- **কিন্তু `K ≈ N` হলে (যেমন Top-80% লাগলে):** `log K ≈ log N` এবং `O(K) = O(N)` হয়ে যায়—তখন heap-এর কোনো asymptotic সুবিধা থাকে না, বরং একবারে sort করাই সরল ও পরিষ্কার।

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-04-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একটি invoicing service ক্রমবর্ধমান (monotonically increasing) `invoiceId` (`1001, 1002, 1003, ...`) একটি **unbalanced** Binary Search Tree-তে insert করছে। Insert ও lookup-এর সময় complexity কীভাবে ভেঙে পড়বে, আর একটি **balanced search tree** কোন structural invariant বজায় রেখে উভয়ই `O(log N)` রাখে?
> 2. একটি job scheduler-এ `N`টি pending retry job আছে; প্রতি মুহূর্তে worker-কে নতুন job add করতে হয় এবং সবচেয়ে কাছের `runAt` timestamp-এর job-টি extract করতে হয়। Sorted array বনাম Binary Min-Heap-এর `insert` ও `extractMin` cost তুলনা করুন। আর একটি `0`-based binary heap array-তে index `5`-এ থাকা element-এর `parent`, `left child`, ও `right child`-এর index কত?

এবার আমরা এসেছি Block 4-এর শেষ বড় অধ্যায়ে: যখন সত্যিই পুরো dataset **sort** করতে হয়—যেমন আমাদের ৪ নম্বর feature request (Order Export Ordering) বা binary search-এর আগে catalog সাজানো—তখন কোন sorting algorithm বেছে নেবেন, কেন `O(N^2)` insertion sort আজও মরে যায়নি, আর কোনো comparison sort-ই কেন `Ω(N log N)`-এর দেয়াল ভাঙতে পারে না?

---

## ৫. Sorting Trade-Offs, `Ω(N log N)` Lower Bound, এবং Time-Space Trade-Offs

বাস্তবে আপনি Node.js-এ প্রতিদিন হাতে sorting algorithm লিখবেন না (`Array.prototype.sort` ব্যবহার করবেন)। তবু sorting-এর চারটি মৌলিক trade-off না জানলে আপনি ভুল জায়গায় sort করবেন, stability bug বানাবেন, বা memory blowup ডেকে আনবেন *(Mandatory 3: Pat Morin, Open Data Structures `11.1`)*।

### ক. Elementary `O(N^2)` Sorting — এবং কেন **Insertion Sort** এখনো অপরিহার্য!

```javascript
// In-place, stable Insertion Sort:
// Worst case (reverse-sorted): Θ(N^2) | Best case (already sorted): Θ(N) | Aux space: Θ(1)
function insertionSort(nums) {
  for (let i = 1; i < nums.length; i++) {
    const key = nums[i];
    let j = i - 1;
    while (j >= 0 && nums[j] > key) {
      nums[j + 1] = nums[j];
      j -= 1;
    }
    nums[j + 1] = key;
  }
  return nums;
}
```

Reverse-sorted input-এ inner `while` loop `1 + 2 + ... + (N - 1) = Θ(N^2)` বার চলে—তাই বড় unsorted array-তে এটি অচল। **তবু আধুনিক production hybrid sort-গুলো কেন আজও Insertion Sort ব্যবহার করে?** দুটি কারণে:
1. **Nearly Sorted Data-তে `Θ(N)`:** যদি array প্রায় সাজানো থাকে (প্রতিটি element তার সঠিক জায়গা থেকে মাত্র কয়েক ঘর দূরে), তবে inner `while` loop প্রতিটি `i`-র জন্য মাত্র `O(1)` বার চলে → পুরো sort শেষ হয় **`Θ(N)`** সময়ে!
2. **Tiny Partitions-এ অতি ক্ষুদ্র Constant Factor:** `N` যখন খুব ছোট (যেমন `8`–`32` elements), তখন Insertion Sort-এ কোনো recursion stack frame নেই, কোনো merge buffer allocation নেই, কোনো pivot partitioning overhead নেই, আর পুরো কাজটাই একটি tiny contiguous slice-এ in-place ও cache-friendly ভাবে ঘটে—তাই ছোট subarrays-এ এটি বাস্তবে `O(N log N)` divide-and-conquer sort-এর চেয়ে দ্রুত চলে!

### খ. Merge Sort বনাম Quicksort: চারটি Engineering Axis-এ তুলনা

Block 2 ও Block 3 থেকে আমরা এদের recurrence আগেই জানি—এবার চারটি বাস্তব trade-off axis একসাথে মিলিয়ে দেখুন:

- **Sort Stability (অত্যন্ত গুরুত্বপূর্ণ!):** একটি sorting algorithm **stable** হয় যদি সমান sort key-যুক্ত দুটি record sort হওয়ার *পরেও* তাদের **আগের relative order** ধরে রাখে।
  - *আমাদের Order Export উদাহরণ:* ধরুন orders আগে থেকেই `createdAt` (সকাল ৯টা, ১০টা, ১১টা) অনুযায়ী সাজানো। এবার আপনি `status` (`PAID`, `PENDING`) দিয়ে sort করলেন। **Stable sort** হলে সব `PAID` order-এর ভেতর সকাল ৯টার order-টি ১০টার order-এর আগেই থাকবে! **Unstable sort** হলে সমান `PAID` key-র ভেতর সেই chronological ক্রম এলোমেলো হয়ে যেতে পারে!

| Algorithm | Best Case | Average / Expected | Worst Case | Auxiliary Space | Stable? | মূল Engineering Trade-Off |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **Insertion Sort** | `Θ(N)` *(already sorted)* | `Θ(N^2)` *(random)* | `Θ(N^2)` *(reverse)* | **`Θ(1)`** *(in-place)* | **হ্যাঁ** | বড় unsorted data-তে অচল; **tiny বা nearly sorted** partition-এ দারুণ। |
| **Merge Sort** | `Θ(N log N)` | `Θ(N log N)` | **`Θ(N log N)`** *(deterministic!)* | **`Θ(N)`** *(merge buffer + `Θ(log N)` stack)* | **হ্যাঁ** | Adversarial input-এও `Θ(N log N)` ও stability নিশ্চিত, কিন্তু `Θ(N)` extra RAM চায়। |
| **Randomized Quicksort** *(distinct keys বা 3-way partition সহ)* | `Θ(N log N)` | **`Θ(N log N)` expected** *(internal randomness)* | `Θ(N^2)` *(rare pathological pivots)* | **`O(log N)` expected** / `O(N)` worst *(call stack; in-place array)* | **সাধারণত না** | `Θ(N)` extra buffer ছাড়াই in-place ও cache-friendly; তবে সাধারণ সংস্করণ unstable এবং Block 3-এর নিয়মমতো কেবল expectation একাই hard per-run bound নয়। |

*(বিশেষ নোট: Block 3-এর accuracy guardrail মনে রাখবেন—standard two-way Quicksort-এর `Θ(N log N)` expected-case বিশ্লেষণ **distinct keys** ধরে নেয়; অনেক duplicate key থাকলে three-way partitioning-এর মতো duplicate handling প্রয়োজন।)*

### গ. Comparison-Sorting Lower Bound: কেন কোনো General Comparison Sort `Ω(N log N)`-এর নিচে নামতে পারে না?

একটু ভেবে দেখুন: আমরা কি কখনো এমন কোনো জাদুকরি comparison-based sorting algorithm আবিষ্কার করতে পারব যা যেকোনো `N`টি element-কে শুধুমাত্র `a < b` তুলনা করে worst-case `O(N)` সময়ে সাজিয়ে দেবে?

**গাণিতিক উত্তর: না—অসম্ভব!** এর পেছনের **Decision-Tree Intuition** মাত্র ৩ লাইনের *(Mandatory 3: Open Data Structures `11.1`)*:
1. `N`টি distinct element-কে মোট **`N!`** (N factorial) ভিন্ন ক্রমে সাজানো সম্ভব। যেকোনো সঠিক comparison sort-কে প্রতিটি সম্ভাব্য permutation-এর জন্য আলাদা উত্তর বের করতে পারতে হবে—অর্থাৎ এর binary decision tree-তে অন্তত **`N!`টি leaf** থাকতে হবে।
2. প্রতিটি `a < b` comparison (`হ্যাঁ` বা `না`) অবশিষ্ট সম্ভাবনাকে সর্বোচ্চ অর্ধেক করে। তাই height `h`-এর একটি binary tree-তে সর্বোচ্চ `2^h`টি leaf থাকতে পারে।
3. সুতরাং `2^h >= N!` হতেই হবে → `h >= log2(N!)`। যেহেতু `N!` সংখ্যাটির অন্তত `N / 2`টি গুণনীয়ক `>= N / 2`, তাই `N! >= (N / 2)^(N / 2)` →
   ```text
   h >= log2((N / 2)^(N / 2)) = (N / 2) * log2(N / 2) = Ω(N log N)
   ```
তাই **comparison model-এর অধীনে** যেকোনো general sorting algorithm-এর worst-case comparison সংখ্যা অন্তত **`Ω(N log N)`**!

### ঘ. Non-Comparison Sorting ও Hybrid Sorts (`Awareness` মাত্র)
- **Counting Sort / Radix Sort (`Awareness`):** যদি আপনি general comparison model ছেড়ে দিয়ে **অতিরিক্ত key constraint** কাজে লাগান—যেমন সব key `0` থেকে `k` পর্যন্ত bounded small integer—তবে comparison না করেই bucket count করে **`O(N + k)`** সময়ে সাজানো সম্ভব! *(তবে এটি arbitrary comparable object বা unbounded string-এর universal replacement নয়।)*
- **Hybrid Sorts — যেমন Timsort (`Awareness`):** বাস্তব standard library ও runtime-গুলো (যেমন Python-এর **Timsort**, কিংবা অন্যান্য ভাষার hybrid library sort) একক textbook sort না চালিয়ে **hybrid sort** ব্যবহার করে—যেমন Timsort বাস্তব data-র ভেতর আগে থেকে থাকা sorted runs শনাক্ত করে Merge Sort-এর stability ও `O(N log N)` worst-case-এর সাথে tiny runs-এ Insertion Sort-এর কম constant-factor গতি যুক্ত করে!

### ঙ. Time-Space Trade-Offs: পুরো Block 4-কে এক সুতোয় বাঁধা

লক্ষ করুন, Block 4 জুড়ে প্রায় প্রতিটি speedup আসলে একটি সচেতন **Time-Space Trade-Off**:
1. **Hash Indexing (`Map`/`Set`):** `O(M)` extra memory খরচ করে `O(N * M)` nested lookup-কে `O(N + M)` expected সময়ে নামানো।
2. **Precomputation / Pre-sorting:** একবার `O(N log N)` upfront cost (ও প্রয়োজনে sorted copy-র জন্য `O(N)` space) দিয়ে পরবর্তী `Q`টি lookup বা range query-কে `O(Q log N)`-এ দ্রুত করা।
3. **Caching / Memoization:** আগের ব্যয়বহুল calculation বা query result memory-তে জমা রেখে পরের অনুরোধে পুনর্গণনা এড়ানো *(Phase 16-এ Caching Engineering-এ এর বিস্তারিত অর্থনীতি ও invalidation পড়ব)*।
4. **Streaming Top-K Min-Heap:** উল্টো trade-off! পুরো sorted order-এর বদলে শুধু Top-`K` চেয়ে auxiliary memory `O(N)` থেকে `O(K)`-এ নামিয়ে আনা!

**গুরুত্বপূর্ণ সতর্কতা (Crossover Points):** আপনি যখন `(N, Q)`-এর জন্য তিনটি strategy (যেমন linear scan `O(Q * N)`, sort + binary search `O(N log N + Q log N)`, আর hash map `O(N + Q)` expected) তুলনা করবেন, তখন asymptotic expressions দিয়ে আপনি পরিষ্কার বলতে পারবেন **কোন `(N, Q)` regime-এ কোন strategy asymptotically এগিয়ে** (যেমন `Q = 1` হলে sort বা hash map বানিয়ে লাভ নেই—একবার `O(N)` linear scan-ই সবচেয়ে সোজা; আবার `Q` বড় হলে `O(Q * N)` ধসে পড়ে)। **কিন্তু কেবল asymptotic expression দেখেই বাস্তব machine-এ exact crossover সংখ্যা (`Q = 7` নাকি `Q = 42`?) নির্ধারণ করা সম্ভব নয়**—তার জন্য explicit constant factors, একটি defined cost model, এবং বাস্তব measurement প্রয়োজন *(যা আমরা Block 5 ও Block 7-এ শিখব)*!

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-02/notes/block-04-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একটি invoice list আগে থেকেই `createdAt` (oldest → newest) অনুযায়ী sorted আছে। এবার user UI-তে "Sort by `paymentStatus`" চাপল যাতে প্রতিটি status-এর ভেতর invoice-গুলো আগের মতোই chronological থাকে। এখানে **stability** কেন জরুরি, এবং Merge Sort বনাম সাধারণ in-place Quicksort-এর কোনটি এই শর্ত রক্ষা করে (তার auxiliary space trade-off কী)?
> 2. একজন engineer বললেন: *"আমাদের সব product price `0` থেকে `50,000` cents-এর মধ্যে integer—কিন্তু `Ω(N log N)` lower bound প্রমাণ করেছে কোনো sorting algorithm-ই `O(N log N)`-এর চেয়ে দ্রুত হতে পারে না!"* এই যুক্তিতে ঠিক কোন assumption-এর ভুল রয়েছে?

---

## Block 5-এর দিকে সেতুবন্ধন: যখন দুটি `Θ(N)` Algorithm-এর বাস্তব গতিতে বহু গুণের ব্যবধান থাকে!

Block 4-এ আপনি backend engineering-এর core data-structure toolkit আয়ত্ত করলেন:
- Contiguous arrays বনাম linked lists (এবং "known node বনাম locating node" qualification),
- Hash tables (load factor, chaining বনাম linear probing, rehashing, conventional expected `O(1)` model, এবং range-query সীমাবদ্ধতা),
- Sorted arrays বনাম unbalanced/balanced search trees,
- Binary heaps ও `O(N log K)` streaming Top-K, এবং
- Sorting trade-offs (stability, in-place vs. extra space, `Ω(N log N)` comparison lower bound, hybrid sorts, ও time-space trade-offs)।

কিন্তু Block 4 জুড়ে দুটি অস্বস্তিকর বাস্তব ইঙ্গিত বারবার উঁকি দিয়েছে:
> *একই `Θ(N)` complexity হওয়া সত্ত্বেও contiguous array কেন linked list-এর চেয়ে বাস্তবে অনেক দ্রুত চলে? ছোট `N`-এ `O(N^2)` insertion sort কেন `O(N log N)` merge sort-কে হারিয়ে দেয়? আর একটা backend request-এ ছোটখাটো in-memory CPU optimization-এর তুলনায় একটিমাত্র network বা disk I/O call কেন সাধারণত কয়েক orders of magnitude বেশি ভারী হয়?*

Word-RAM model-এর সেই পর্দা সরিয়ে hardware reality, memory hierarchy, constant factors, ও back-of-the-envelope resource estimation নিয়ে আসছে **Phase 2, Block 5 — Beyond Big-O: Constant Factors, Hardware Reality, and Data-Access Cost**!

---

## Study Plan, Learning Objectives, Terminology & Resources

> **Continuity & Ownership Notes:** আপনার সব Pause & Checks, দুটি Mandatory Exercise, এবং (যদি করতে চান) Optional Closed-Book Assessment-এর উত্তর স্বাধীনভাবে `phases/phase-02/notes/block-04-notes.md` ফাইলে লিখুন। AI-generated material কখনো completed learning বা learner study hours হিসেবে গণ্য হয় না।
>
> **Progression & Verification Policy:** এই block-এর Closed-Book Assessment সম্পূর্ণ **optional** এবং Block 5 শুরু করার জন্য mandatory gate নয়। পড়া ও দুটি mandatory exercise শেষ করে assessment skip করলে এই block-টির status হবে **`Studied — mastery not formally verified`**; যথেষ্ট evidence share করে ChatGPT evaluation সম্পন্ন হলে তা **`Mastery verified`** হিসেবে গণ্য হবে।

### Study Time Reference

*প্রতিটি activity-র বিস্তারিত time breakdown দেখতে এই নথির শুরুর **Study Time Guide** দেখুন। Approved dossier-এ Block 4-এর roadmap target allocation হলো `3.5 hours (210 min)`—যা একটি flexible planning reference, কোনো strict deadline নয়।*

### Block 4 Learning Objectives *(Primary Target Level: `Practical Competence` | Supported Phase 2 Outcomes: Outcomes 4, 5)*

- **LO 4.1 (`Practical Competence` | Bloom: *Analyze / Evaluate*):** Compare exact operation costs (index access, search, append, front/middle insert/delete, memory overhead) of **contiguous dynamic arrays** vs. **linked lists**—explicitly qualifying that `O(1)` linked-list insertion/deletion applies **only when the relevant node or insertion position is already known** (locating that position still requires `O(N)` traversal)—and explain why contiguous arrays dominate most backend sequential workloads despite identical `Θ(N)` scans.
- **LO 4.2 (`Practical Competence` | Bloom: *Apply / Analyze*):** Explain abstract **hash-table** mechanics (hash function, bucket array, **load factor `α = n/m`**, collision resolution via **separate chaining** vs. **open addressing / linear probing**, dynamic rehashing) and apply hash-based `Map`/`Set` structures appropriately to exact-key lookup and deduplication workloads—explicitly framing **expected `O(1)`** as the **conventional hash-table cost model** (conditional on hash distribution and bounded load factor, not an unconditional runtime guarantee), separating abstract collision mechanics from modern JS runtime implementation details (V8 internals deferred to Phase 5), and recognizing memory overhead and inability to serve ordered range queries efficiently.
- **LO 4.3 (`Working Understanding` | Bloom: *Analyze*):** Contrast **unsorted scan (`O(N)`)**, **sorted array + binary search (`O(log N)` search, `O(N)` insert)**, **balanced search trees (`O(log N)` search/insert/delete + ordered range/min/max/prefix queries — conceptual capability without manual rotation proofs)**, and **binary heaps / priority queues (`O(log N)` push/pop-min, `O(1)` peek, `O(N log K)` Top-K)**—establishing the conceptual bridge to Phase 10 database indexes.
- **LO 4.4 (`Working Understanding` | Bloom: *Evaluate*):** Compare foundational **sorting trade-offs**—elementary `O(N^2)` sorts (insertion sort's strength on tiny or nearly sorted data), `O(N log N)` comparison sorts (merge sort deterministic worst-case + stability + `O(N)` space vs. quicksort in-place expected `O(N log N)` / `O(N^2)` worst-case + randomization; conceptual `Ω(N log N)` comparison lower bound), and `Awareness` of linear-time counting/radix sort for bounded integer keys and hybrid sorts (e.g., Timsort)—and apply **time-space trade-offs** cleanly.

### Terminology Inventory

| Term | Official Target Level | Why It Matters in Block 4 |
| :--- | :---: | :--- |
| **Contiguous Dynamic Array vs. Linked List** | `Practical Competence` | Balances `O(1)` indexing/locality (arrays) against `O(1)` unlink/link *only when node/position is already known* (linked lists—locating still costs `O(N)`). |
| **Hash Table, Load Factor (`α = n/m`) & Rehashing** | `Practical Competence` | Conventional expected `O(1)` cost model (not an unconditional runtime guarantee) behind `Map`/`Set`, deduplication, and in-memory joins—kept separate from V8 internals. |
| **Separate Chaining vs. Linear Probing** | `Working Understanding` | Contrasts pointer chains against contiguous open addressing, `α < 1` requirement, and primary clustering. |
| **Binary Search & Balanced Search Trees** | `Working Understanding` | Explains why sorted arrays fail on frequent writes (`O(N)`), why unbalanced BSTs degrade to `O(N)`, and what `O(log N)` balance buys for ordered range queries (`AVL`/`Red-Black`/`B-Tree` at `Awareness`). |
| **Binary Heap / Priority Queue & Top-K** | `Practical Competence` | Implicit 0-based array layout (`O(log N)` push/pop-min, `O(1)` peek) enabling `O(N log K)` time & `O(K)` space streaming Top-K. |
| **Sort Stability & Comparison Lower Bound (`Ω(N log N)`)** | `Working Understanding` | Explains multi-key ordering preservation, why general comparison sorts cannot beat `Ω(N log N)`, and why insertion sort stays valuable inside hybrid sorts. |
| **Non-Comparison Sorts (Counting/Radix) & Hybrid Sorts (Timsort)** | `Awareness` | Ecosystem awareness of bounded-integer linear sorting and real-world hybrid library designs. |
| **Time-Space Trade-Off** | `Practical Competence` | Deliberately trading auxiliary memory (hash indexing, precomputation, caching/memoization) or full ordering (Top-K heap) to match workload constraints. |

### Mandatory Authoritative Resources *(Targeted Reading — ~65–80 min total)*

1. **Mandatory 1 (~20–25 min — assigned sections only):** MIT OpenCourseWare 6.006 *Introduction to Algorithms* (Spring 2020) — [Lecture 4: Hashing](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/mit6_006s20_lec4/) (`LO 4.2`):
   - **Assigned scope:** Lecture 4 Notes PDF — *Direct Access Array*, *Hashing*, *Chaining*, *Universal/Good Hashing*, এবং *Load Factor & Expected `O(1)`*।
   - **What to extract:** Direct-access array কেন অবাস্তব এবং hashing + dynamic resizing কীভাবে `O(N)` space-এ conventional expected `O(1)` দেয়।
2. **Mandatory 2 (~25–30 min — assigned sections only):** Robert Sedgewick & Kevin Wayne (Princeton University), *Algorithms, 4th ed.* Booksite (`LO 4.2`, `LO 4.3`):
   - [Section 3.1 — Elementary Symbol Tables](https://algs4.cs.princeton.edu/31elementary/) — *Sequential search* বনাম *Binary search in an ordered array* এবং cost-comparison summary।
   - [Section 3.4 — Hash Tables](https://algs4.cs.princeton.edu/34hash/) — *Separate chaining* বনাম *Linear probing* conceptual trade-offs।
   - **What to extract:** Sequential search, ordered-array binary search, এবং hash tables-এর cost summary, ও ordered operations কেন tree বনাম hash table আলাদা করে।
3. **Mandatory 3 (~20–25 min — assigned sections only):** Pat Morin, *Open Data Structures* (`LO 4.3`, `LO 4.4`):
   - [Section 10.1 — BinaryHeap: An Implicit Binary Tree](https://opendatastructures.org/ods-python/10_1_BinaryHeap_Implicit_Bi.html) — implicit `0`-based array layout এবং `O(log N)` insert/remove mechanics।
   - [Section 11.1 — Comparison-Based Sorting](https://opendatastructures.org/ods-python/11_1_Comparison_Based_Sorti.html) — *Merge-Sort*, *Quicksort*, এবং `Ω(N log N)` comparison lower-bound intuition *(formal proofs বাদ দিন)*।
   - **What to extract:** Pointer ছাড়াই flat array-তে heap কীভাবে চলে এবং `O(N log N)` sorting algorithm-গুলোর exact trade-offs।

### Optional Supplementary Resources *(Outside Mandatory Budget — Do Not Treat as Mandatory)*

- **Supplementary (Free):** Robert Sedgewick & Kevin Wayne (Princeton), *Algorithms, 4th ed.* Booksite — [Section 3.3 — Balanced Search Trees](https://algs4.cs.princeton.edu/33balanced/) — 2-3 / balanced trees কীভাবে `O(log N)` height বজায় রাখে তার conceptual overview *(Red-Black Java rotation code বাদ দিন)*।
- **Supplementary (Commercial Book):** Thomas H. Cormen et al. (CLRS), *Introduction to Algorithms* (4th ed., MIT Press, 2022) — Chapter 11 (*Hash Tables*, Sections `11.1–11.4`)।

---

## Practical Exercises *(Mandatory — ~50–60 min total)*

> **Ownership Rule:** উভয় exercise-এর সব সিদ্ধান্ত, complexity derivation, ও trade-off ব্যাখ্যা স্বাধীনভাবে `phases/phase-02/notes/block-04-notes.md` ফাইলে লিখুন। এখানে কোনো answer key দেওয়া হয়নি।

### Exercise 1: Workload-to-Data-Structure Decision Matrix *(Mandatory — ~25–30 min | `LO 4.1`, `LO 4.2`, `LO 4.3`)*

আমাদের online-store backend-এর নিচের **৫টি বাস্তব workload** বিশ্লেষণ করুন:

1. **Promo Code Lookup:** প্রতি মিনিটে `1,000,000`টি checkout অনুরোধে exact `code` (যেমন `"WINTER25"`) দিয়ে active promo rule খোঁজা; promo তালিকা দিনে মাত্র কয়েকবার update হয়।
2. **Chronological Order Range Query:** নতুন order নিয়মিত যোগ হতে থাকার পাশাপাশি admin dashboard-এ ঘন ঘন জিজ্ঞেস করা: *"Timestamp `T1` থেকে `T2`-এর মধ্যে তৈরি সব order chronological ক্রমে দাও।"*
3. **Webhook Event Deduplication:** মেমোরিতে আসা `200,000`টি webhook `eventId`-এর মধ্যে duplicate বাদ দিয়ে প্রতিটি unique event কেবল একবার process করা।
4. **Streaming Top-10 VIP Customers:** `2,000,000`টি streaming order record-এর মধ্য থেকে সর্বোচ্চ মোট খরচ করা **Top-10 customer** বের করা—পুরো `2,000,000` record একসাথে sorted অবস্থায় RAM-এ না রেখে।
5. **FIFO Background Job Queue:** একটি in-memory worker queue যেখানে নতুন job সবসময় **tail**-এ যোগ (enqueue) হয় এবং পুরোনো job সবসময় **head** থেকে বের (dequeue) হয়।

প্রতিটি workload-এর জন্য একটি পরিষ্কার **Decision Matrix** তৈরি করে লিখুন:
- **(ক) নির্বাচিত Data Structure:** (`Contiguous Dynamic Array`, `Circular Queue / Doubly Linked List`, `Hash Table (Map/Set)`, `Sorted Array / Balanced Search Tree`, `Binary Min-Heap`—কোনটি বেছে নিলেন?)।
- **(খ) Operation Complexity ও Memory Overhead:** প্রধান operation-গুলোর exact time complexity (যথাযথ qualification সহ—যেমন conventional expected `O(1)` বা known-node `O(1)`) এবং auxiliary space / memory overhead।
- **(গ) Ordering ও Workload Justification:** এই workload-এর exact-lookup / ordering / streaming / mutation বৈশিষ্ট্যের সাথে structure-টি কেন মানানসই?
- **(ঘ) Wrong-Structure Failure Mode:** যদি এখানে একটি জনপ্রিয় **ভুল** structure বেছে নেওয়া হতো (যেমন Workload 2-এ hash table, Workload 4-এ full sort, বা Workload 5-এ plain array + `.shift()`), তবে ঠিক কোন operation-এ এবং কোন complexity class-এ এটি ভেঙে পড়ত?

---

### Exercise 2: Search Strategy, `(N, Q)` Regimes, and Sorting Trade-Off Analysis *(Mandatory — ~25–30 min | `LO 4.3`, `LO 4.4`)*

ধরুন আপনার কাছে **`N`টি product**-এর একটি প্রাথমিকভাবে **unsorted** catalog আছে (প্রতিটি product-এর `productId` distinct), এবং আপনাকে সেই একই static snapshot-এর ওপর **`Q`টি exact `productId` lookup query** চালাতে হবে।

নিচের **তিনটি strategy** তুলনা করুন:
- **Strategy A (Repeated Linear Scan):** কোনো preprocessing নেই; প্রতিটি query পুরো catalog scan করে।
- **Strategy B (Sort Once + Binary Search):** শুরুতে একবার `productId` দিয়ে catalog sort করুন (ব্যবহৃত sorting algorithm ও তার worst/expected-case assumption স্পষ্ট লিখুন), তারপর প্রতিটি query-তে binary search চালান।
- **Strategy C (Hash Map Index + Exact Lookup):** শুরুতে একবার `Map` তৈরি করে প্রতিটি `productId` index করুন, তারপর প্রতিটি query-তে exact lookup চালান (conventional hash-table cost model উল্লেখ করে)।

নিচের **৪টি বিশ্লেষণ** ধাপে ধাপে লিখুন:

1. **Exact `(N, Q)` Complexity & Regimes:** তিনটি strategy-র total time complexity (`N` ও `Q`-এ) এবং auxiliary space complexity বের করুন। তারপর স্পষ্টভাবে চিহ্নিত করুন কোন কোন **asymptotic ও abstract operation-count regime-এ** প্রতিটি strategy অন্যগুলোর চেয়ে সুবিধাজনক (যেমন `Q = 1`, `Q << log N`, `Q ≈ log N`, `Q >= N`, এবং কঠোর memory-constrained পরিবেশ)।
2. **Why Asymptotics Alone Cannot Pick an Exact Crossover Point:** ব্যাখ্যা করুন কেন কেবল `Q * N`, `N log N + Q log N`, আর `N + Q`-এর মতো asymptotic expression দেখেই বাস্তব server-এ ঠিক কোন `(N, Q)` মানের পর Strategy B বা C, Strategy A-কে হারাবে (exact crossover point) তা নির্ধারণ করা যায় না—কোন কোন তথ্য (explicit constants, defined cost model, hardware/runtime effects) অনুপস্থিত থাকে?
3. **Workload Shift — Exact Lookup → Ordered Price Range Query:** এখন ধরুন product team requirement বদলে বলল: *"Exact `productId` lookup নয়, বরং `Q` বার জিজ্ঞেস করা হবে: দাম `USD 10` থেকে `USD 25`-এর মধ্যে থাকা সব product দাম অনুযায়ী সাজিয়ে দাও!"*
   - তিনটি strategy-র মধ্যে **Strategy C (Hash Map)**-এর কী হবে?
   - **Strategy B (Sorted Array + Binary Search)** কীভাবে এই range query serve করবে (matching সংখ্যা `K` ধরে exact complexity লিখুন)?
   - আর যদি catalog-টি static না থেকে **ঘন ঘন product price update ও নতুন product insert** হতে থাকে, তবে Sorted Array-র বদলে কোন structure নিতে হবে এবং কেন?
4. **Insertion Sort Inside Hybrid Sorting:** Insertion Sort-এর worst-case `Θ(N^2)` হওয়া সত্ত্বেও আধুনিক production **hybrid sorting algorithms** কেন এখনো ইচ্ছাকৃতভাবে tiny partitions (এবং nearly sorted runs)-এ Insertion Sort-এ switch করে? অন্তত ৩টি সুনির্দিষ্ট technical কারণ লিখুন।

---

## Closed-Book Synthesis Assessment *(Optional — ~25–30 min)*

এই assessment-টি সম্পূর্ণ **optional**—Block 5 শুরু করার জন্য এটি সম্পন্ন করা বা ChatGPT-র approval নেওয়া mandatory নয়। আপনি চাইলে:
- এই lesson ও সব browser tab বন্ধ রেখে নিচের ৫টি প্রশ্নের নিজের ভাষায় উত্তর `phases/phase-02/notes/block-04-notes.md` ফাইলে লিখে formal evaluation-এর জন্য **ChatGPT**-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে);
- অথবা আপনি যথেষ্ট বুঝেছেন মনে করলে assessment skip করে Block 5-এ এগোতে পারেন (`Studied — mastery not formally verified`);
- অথবা assessment না করেই যেকোনো unclear concept নিয়ে প্রশ্ন ও আলোচনা করতে পারেন।

*(Note: `~25–30 min` কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি approximate estimate—কোনো strict time limit নয়।)*

1. **(LO 4.1 / Outcome 5)** Contiguous dynamic arrays এবং singly/doubly linked lists-এর প্রধান operation costs ও memory overhead তুলনা করুন। কেন *"linked list-এ middle insert/delete `O(1)`"* কথাটি শর্ত ছাড়া ভুল, এবং উভয়ের scan `Θ(N)` হওয়া সত্ত্বেও sequential backend workload-এ array কেন সাধারণত বেশি ভালো করে?
2. **(LO 4.2 / Outcomes 4, 5)** Abstract hash table কীভাবে কাজ করে (hash function, buckets, load factor `α = n/m`, separate chaining বনাম open addressing/linear probing, dynamic rehashing)? কেন expected `O(1)` একটি conditional cost model (unconditional guarantee নয়), কেন hash table ordered range query-তে ব্যর্থ হয়, এবং কেন abstract mechanics-কে V8 `Map`/`Set` internals থেকে আলাদা রাখা জরুরি?
3. **(LO 4.3 / Outcome 5)** Unsorted array, sorted array + binary search, unbalanced BST, এবং balanced search tree-র মধ্যে exact lookup, dynamic insert/delete, এবং ordered range query-র trade-off তুলনা করুন। Sorted keys insert করলে unbalanced BST কীভাবে `Θ(N)`-এ ধসে পড়ে?
4. **(LO 4.3 / Outcome 5)** Binary Min-Heap-এর implicit `0`-based array layout এবং `peek`/`push`/`popMin` cost ব্যাখ্যা করুন। Streaming `N`টি record থেকে Top-`K` largest মান বের করতে Max-Heap-এর বদলে `K`-সাইজের Min-Heap কীভাবে `O(N log K)` সময় ও `O(K)` space অর্জন করে—এবং `K ≈ N` হলে কী হয়?
5. **(LO 4.4 / Outcome 5)** Insertion Sort, Merge Sort, এবং Randomized Quicksort-কে time, auxiliary space, stability, এবং duplicate/sorted-input ঝুঁকির আলোকে তুলনা করুন। Decision-tree intuition দিয়ে ব্যাখ্যা করুন কেন কোনো general comparison sort `Ω(N log N)` ভাঙতে পারে না, এবং কোন শর্তে non-comparison sort (Counting/Radix) সেই সীমার বাইরে যায়।

### Block 4 Readiness & Progression Checklist

- [ ] Guided lesson মনোযোগ দিয়ে পড়েছেন এবং Pause & Checks 1–3 `phases/phase-02/notes/block-04-notes.md`-এ লিখে (পরে Self-Check Guidance মিলিয়ে) সম্পন্ন করেছেন।
- [ ] তিনটি Mandatory Reading (**MIT 6.006 Lecture 4**, **Princeton `3.1` & `3.4`**, এবং **Open Data Structures `10.1` & `11.1`** নির্ধারিত অংশ) সম্পন্ন করেছেন।
- [ ] `phases/phase-02/notes/block-04-notes.md` ফাইলে স্বাধীনভাবে **Exercise 1 (Workload-to-Data-Structure Decision Matrix)** এবং **Exercise 2 (Search Strategy, `(N, Q)` Regimes, and Sorting Trade-Off Analysis)** সম্পন্ন করেছেন।
- [ ] *(Optional)* Notes/tabs বন্ধ রেখে Closed-Book Synthesis Assessment Questions 1–5 লিখেছেন এবং formal evaluation-এর জন্য ChatGPT-র কাছে share করেছেন।

> **Progression Status Distinction:** Block 5 কখন শুরু করবেন তা আপনার সিদ্ধান্ত। পড়া ও দুটি mandatory exercise শেষ করে optional assessment ছাড়া এগোলে এই block-টির status হবে **`Studied — mastery not formally verified`**। যথেষ্ট evidence share করে ChatGPT evaluation সম্পন্ন হলেই তা **`Mastery verified`** হিসেবে গণ্য হবে।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`phases/phase-02/notes/block-04-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের guidance দেখুন। Exercise 1, Exercise 2, বা Optional Closed-Book Assessment-এর কোনো solution এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও (চাইলে) optional ChatGPT review-এর জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* Singly linked list-এ `itemId` বা index `i` দিয়ে মাঝখানের item-টি **খুঁজে বের করতে** `head` থেকে `O(N)` traversal লাগবেই—তাই মোট deletion cost হয় **`O(N)`**, `O(1)` নয়! Linked-list deletion কেবল তখনই `O(1)` যখন আপনার হাতে আগে থেকেই **predecessor node-এর reference** (singly linked list-এ) অথবা **target node-এর reference** (doubly linked list-এ) সরাসরি থাকে।
  2. *শক্তিশালী উত্তরে থাকবে:* (ক) Conventional hash-table cost model অনুযায়ী প্রতিটি lookup/insert expected `O(1)` হওয়ায় মোট সময় **`O(N)` expected** এবং unique session রাখার জন্য auxiliary space **`O(N)`**। এটি unconditional runtime guarantee নয় কারণ expected `O(1)` নির্ভর করে ভালো hash distribution ও bounded load factor-এর ওপর (collision বা rehashing-এ single-op spike হতে পারে, এবং abstract model সরাসরি V8 internals বা wall-clock SLA প্রমাণ করে না)। (খ) Hash function key-গুলোকে bucket-এ এলোমেলোভাবে ছড়িয়ে দেয়, ফলে timestamp-এর কোনো sorted order থাকে না—তাই ১০ মিনিটের range বের করতে পুরো `Set` scan ও sort করতে হবে।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* Monotonically increasing `invoiceId` (`1001, 1002, ...`) insert করলে প্রতিটি নতুন key আগেরটির ডান child হয়ে একটি ডান-মুখী linked chain তৈরি করে (height `h = N`)—ফলে insert ও lookup উভয়ই **`Θ(N)`**-এ নেমে যায় (`N`টি insert-এ মোট `Θ(N^2)`)। Balanced search tree rebalancing invariants বজায় রেখে height সবসময় **`O(log N)`**-এ আটকে রাখে, তাই search, insert, delete সবই `O(log N)` থাকে।
  2. *শক্তিশালী উত্তরে থাকবে:* **Sorted array**-তে নতুন job sorted জায়গায় `insert` করতে `O(N)` shifting লাগে (যদিও সর্বনিম্ন মান শেষে রাখলে `extractMin` `O(1)` হতে পারে, তবু প্রতি insert `O(N)`)। **Binary Min-Heap**-এ `insert` (`push`) এবং `extractMin` (`popMin`) উভয়ই **`O(log N)`** (এবং `peek` `O(1)`)—ফলে dynamic insert + extract-min workload-এ heap অনেক এগিয়ে! `0`-based heap array-তে index `5`-এর জন্য: **`parent = Math.floor((5 - 1) / 2) = 2`**, **`left = 2 * 5 + 1 = 11`**, **`right = 2 * 5 + 2 = 12`**।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* সমান `paymentStatus`-যুক্ত invoice-গুলো যেন sort হওয়ার পরেও তাদের আগের `createdAt` chronological ক্রম না হারায়, সেজন্য **stable sort** অপরিহার্য। **Merge Sort** stable এবং deterministic `Θ(N log N)` worst-case দেয়, কিন্তু এর বিনিময়ে **`Θ(N)` auxiliary space** (merge buffer) লাগে। বিপরীতে সাধারণ in-place **Quicksort** `Θ(N)` merge buffer না নিলেও **unstable** (ফলে একই status-এর ভেতর chronological order ভেঙে যেতে পারে)।
  2. *শক্তিশালী উত্তরে থাকবে:* `Ω(N log N)` lower bound প্রযোজ্য কেবল **general comparison-based sorting model**-এর অধীনে (যেখানে algorithm কেবল `a < b` তুলনা করে `N!` permutation আলাদা করে)। এখানে সব price `0..50,000` bounded small integer হওয়ায় তুলনার বদলে key-র numeric মান সরাসরি index/count করে **non-comparison sort** (যেমন **Counting Sort**) **`O(N + k)`** (এখানে `k = 50,001`) সময়ে সাজাতে পারে—যা comparison model-এর বাইরের একটি বিশেষ শর্তের সুফল।
