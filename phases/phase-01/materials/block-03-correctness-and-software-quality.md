# Phase 1, Block 3 — Correctness and Software Quality

Block 2-এ আপনি একটি কঠিন অভ্যাস তৈরি করেছেন: এক লাইন কোড লেখার আগেই Problem Space-এ দাঁড়িয়ে অস্পষ্টতা খুঁজে বের করা এবং BCP 14-এ যাচাইযোগ্য শর্ত লেখা। এবার এক ধাপ এগিয়ে যান। ধরুন, আমাদের সেই বাড়তে থাকা online store-এর জন্য আপনি নতুন একটি campaign feature বানালেন—`LAUNCH50`, যেটি checkout-এ যেকোনো order থেকে ৫০ টাকা flat ছাড় দেয় এবং পুরো campaign-এ সর্বোচ্চ ১,০০০ বার ব্যবহার করা যাবে।

আপনি Node.js ও PostgreSQL-এ কোড লিখলেন, ১২টি unit test লিখলেন, আর প্রতিটি test সবুজ (green) হয়ে পাস করল। Staging-এ ৫০০ টাকার একটি order-এ `LAUNCH50` বসিয়ে দেখলেন দাম ৪৫০ টাকা হয়ে গেল, আর database-এ `used_count` শূন্য থেকে বেড়ে ১ হলো। কোডটি production-এ deploy করা হলো। টানা দুই সপ্তাহ ধরে স্বাভাবিক traffic-এ (ঘণ্টায় ১০–১৫টি order) একটিও error log নেই, কোনো customer অভিযোগ করেননি, monitoring dashboard ঝকঝকে সবুজ।

তৃতীয় সপ্তাহের শুক্রবার সন্ধ্যায় একজন জনপ্রিয় reviewer সোশ্যাল মিডিয়ায় `LAUNCH50` কোডটি শেয়ার করলেন। মাত্র দশ মিনিটের মধ্যে তিনটি অপ্রত্যাশিত ঘটনা একসাথে ঘটল:

1. একজন shopper ৩০ টাকার একটি phone sticker অর্ডার করে `LAUNCH50` (৫০ টাকা ছাড়) প্রয়োগ করলেন। আপনার কোড সরাসরি `finalTotal = orderTotal - 50` হিসাব করে order-এর মোট মূল্য বানিয়ে দিল `-20` টাকা! Payment gateway ঋণাত্মক (negative) মূল্য দেখে checkout crash করাল না—বরং শূন্য টাকা কেটে order confirm করল, আর পরদিন রাতের accounting reconciliation worker `-20` টাকা দেখে ধরে নিল store customer-এর কাছে ২০ টাকা ঋণী, তাই স্বয়ংক্রিয়ভাবে তাঁর wallet-এ ২০ টাকা credit করে দিল!
2. কয়েক মিনিট পর campaign-এর ব্যবহার ৯৯৯-এ পৌঁছাল। ঠিক সেই মুহূর্তে পাঁচজন shopper একসাথে `Place Order` চাপলেন। পাঁচটি request সমান্তরালভাবে database থেকে পড়ল `used_count = 999 < 1000`, পাঁচজনকেই ৫০ টাকা ছাড় দিল, আর `used_count` বেড়ে হয়ে গেল `1004`—অথচ campaign-এর সর্বোচ্চ সীমা ছিল ১,০০০।
3. তার একটু পর Marketing টিম নতুন একটি backup-coupon নিয়ম চালু করল: কোনো কোডের মেয়াদ শেষ হলে backend-এর একটি ছোট helper যেন তার `fallback_promo_id` ধরে পরবর্তী সচল কোড খুঁজে নেয়। ভুলবশত একজন operator দুটি expired কোডকে একে অপরের fallback বানিয়ে দিলেন (`SUMMER1 → SUMMER2 → SUMMER1`)। Helper-টির `while` loop-টি ঘুরতেই থাকল—কোনো ভুল ছাড় তো দিলই না, বরং কোনো উত্তরই আর ফেরত দিল না! ওই request-গুলো একের পর এক server-এর ক্ষমতা আটকে দিয়ে পুরো checkout service-কে স্তব্ধ করে দিল।

একটু থেমে পুরো দৃশ্যটির দিকে তাকান। শুক্রবার সন্ধ্যা ৬টার আগে এই একই কোডের সব unit test সবুজ ছিল, আর টানা চৌদ্দ দিন production-এ এটি বিনা ঝামেলায় চলেছে।

তাহলে একটি মৌলিক প্রশ্ন সামনে এসে দাঁড়ায়—যে প্রশ্নটি এই পুরো block-টিকে এক সুতোয় গেঁথে রাখবে:

> **একটি backend feature-এর সব test পাস করে এবং আজকের production-এও সেটি ঠিকঠাক চলছে। তবুও আমরা কীভাবে জানব যে software-টি আসলেই *সঠিক (correct)*—আর কেবল প্রত্যাশিত output দেওয়ার বাইরে একটি software *"ভালো মানের (high quality)"* হওয়ার প্রকৃত অর্থ কী?**

---

## "আমার মেশিনে তো চলছে" কেন Correctness নয়—এবং Correctness-এর প্রকৃত সংজ্ঞা

দৈনন্দিন কোডিংয়ে আমরা প্রায়ই বলি *"কোডটা একদম ঠিক (correct) আছে—আমার মেশিনে চালিয়ে দেখেছি, ৫০০ টাকার অর্ডারে ৪৫০ টাকাই আসছে!"* কিন্তু ওপরের শুক্রবারের ঘটনাটি চোখে আঙুল দিয়ে দেখিয়ে দিল, একটি বা দশটি স্বাভাবিক উদাহরণ (**happy path**) ঠিকমতো চলার সাথে প্রকৃত **Correctness**-এর বিশাল ফারাক আছে।

প্রথমেই একটি কঠোর কিন্তু অপরিহার্য সত্য মেনে নিন: **Specification ছাড়া Correctness কথাটার কোনো অর্থই হয় না।**

ধরুন, ৩০ টাকার স্টিকারে ৫০ টাকার `LAUNCH50` বসালে system-টির কী করা উচিত?
- Order-এর মূল্য ০ টাকায় আটকে দেবে?
- নাকি ন্যূনতম অর্ডার-মূল্য ৫০ টাকার কম হলে `PROMO_MIN_ORDER_NOT_MET` error দিয়ে কোডটি বাতিল করবে?

কোডের দিকে যতই তাকিয়ে থাকুন, কোড আপনাকে এর উত্তর দিতে পারবে না। একটি program তখনই **correct**, যখন তা একটি স্পষ্ট **specification**-এর (প্রত্যাশিত আচরণ, সীমানা ও সীমাবদ্ধতার সংজ্ঞা—যা আপনি Block 2-এ শিখেছেন) সাথে সম্পূর্ণভাবে মিলে যায় (**conformance to a specification**)। Specification না থাকলে কোন output-টি "সঠিক" আর কোনটি "ভুল", তা বিচার করার কোনো মাপকাঠিই নেই।

কিন্তু বাস্তব backend system-এ কোনো operation শুধু একটি বিচ্ছিন্ন গাণিতিক হিসাব নয়—এটি একটি চলমান **state** (database-এর order, wallet balance, promo usage count)-কে এক অবস্থা থেকে আরেক অবস্থায় নিয়ে যায়। তাই engineer-রা correctness-কে কেবল "সঠিক output" হিসেবে নয়, বরং তিনটি নির্ভুল শর্তের কাঠামোতে চিন্তা করেন: **Preconditions**, **Postconditions**, এবং **Invariants**।

আমাদের `applyPromoCode(order, promo)` operation-টির মাধ্যমে এই তিনটির মধ্যকার সম্পর্ক দেখুন:

- **Precondition (পূর্বশর্ত):** কোনো operation শুরু *হওয়ার আগে* যা সত্য থাকতে হবে। যেমন: `order.subtotal > 0`, `promo.code` বিদ্যমান ও সক্রিয়, caller-এর বৈধ পরিচয় আছে এবং `order` ইতিমধ্যেই paid বা cancelled অবস্থায় নেই। Precondition লঙ্ঘন হলে operation-টির ভেতরের যুক্তি নির্ভরযোগ্যভাবে কাজ করার ভিত্তি হারায় (অবশ্য public API boundary-তে caller যেহেতু অনির্ভরযোগ্য হতে পারে, তাই backend নিজেই input যাচাই করে invalid request প্রত্যাখ্যান করে)।
- **Postcondition (পরবর্তী নিশ্চয়তা):** Precondition পূরণ হয়ে operation-টি সফলভাবে শেষ *হওয়ার পর* system কী কী ফলাফল নিশ্চিত (guarantee) করবে। যেমন: যদি promo বৈধ ও যোগ্য হয়, তবে `order.final_total === Math.max(0, order.subtotal - promo.discount_amount)` হবে এবং সেই `promo`-র `used_count` ঠিক `1` বাড়বে; আর যদি promo অযোগ্য হয়, তবে `order.final_total` ও `used_count` অপরিবর্তিত থাকবে এবং নির্ধারিত structured error code ফেরত আসবে।
- **Invariant (অটুট নিয়ম বা অপরিবর্তনীয় সত্য):** এমন একটি শর্ত, যা কেবল কোনো একটি function শুরু বা শেষেই নয়—বরং **সিস্টেমের সমস্ত প্রাসঙ্গিক operation ও state transition জুড়ে সর্বদা সত্য থাকতে হবে**। আমাদের store-এ তিনটি অটুট invariant হলো:
  1. প্রতিটি order-এর জন্য `order.final_total >= 0`।
  2. প্রতিটি campaign-এর জন্য `0 <= promo.used_count <= promo.max_total_redemptions`।
  3. প্রতিটি customer-এর জন্য `wallet.balance >= 0` (যদি overdraft অনুমোদিত না হয়)।

ভালো করে লক্ষ করুন, এই তিনটি ধারণা কীভাবে একে অপরের সাথে শৃঙ্খলের মতো বাঁধা। একটি operation-এর **invariant ভেঙে যাওয়া** মানে কেবল ওই একটি moment-এর bug নয়—ওই ভাঙা state-টি পরবর্তী অন্য সমস্ত operation-এর **precondition ভেঙে দেয়**! আমাদের শুক্রবারের ঘটনায় ঠিক সেটাই ঘটেছিল: `applyPromoCode` যখন `order.final_total >= 0` invariant-টি ভাঙল এবং `-20` database-এ লিখে দিল, তখন পরদিন রাতের accounting worker-টি ধরে নিয়েছিল database-এর প্রতিটি order-মূল্য অঋণাত্মক (তার implicit precondition)। ফলে এক জায়গার একটি invariant লঙ্ঘন সম্পূর্ণ ভিন্ন একটি subsystem-এ গিয়ে নীরবে গ্রাহকের wallet-এ অযাচিত টাকা জমা করে দিল!

এবার শুক্রবারের **তৃতীয় ঘটনাটির** কথা মনে করুন—সেই `SUMMER1 → SUMMER2 → SUMMER1` চক্রাকার fallback-promo loop! এখানে কোনো ভুল discount দেওয়া হয়নি, `final_total >= 0` invariant-ও ভাঙেনি, `used_count`-ও সীমা ছাড়ায়নি। তবুও কি সেই `resolveFallbackPromo()` function-টিকে আপনি "correct" বলবেন?

মোটেই না! এখান থেকেই computer science ও software engineering-এ correctness-এর দুটি সূক্ষ্ম স্তরের জন্ম হয়েছে:

1. **Partial Correctness (আংশিক সঠিকতা):** কোনো program বা operation **যদি শেষ হয় (if it terminates)**, তবে তার ফলাফল specification ও postcondition মেনে চলে। লক্ষ করুন শর্তটা—*"যদি শেষ হয়"*! আমাদের চক্রাকার fallback helper-টি কোনো ভুল উত্তর দেয়নি, কারণ চক্রের ভেতর পড়লে এটি কখনোই শেষ হয় না। তাই গাণিতিক অর্থে সেটি *partially correct* হতে পারে, কিন্তু বাস্তবে তা ব্যবহারের অযোগ্য!
2. **Total Correctness (পূর্ণ সঠিকতা):** `Total Correctness = Partial Correctness + Termination`। অর্থাৎ বৈধ precondition-এ program-টি কেবল সঠিক উত্তরই দেবে না, **এটি অবশ্যই সসীম ধাপে শেষও হবে (must terminate)**—এবং বাস্তব backend system-এ এটি নির্ধারিত সময় বা liveness সীমার ভেতর উত্তর ফেরত দেবে।

Termination কোনো একাডেমিক খুঁতখুঁতেপনা নয়; production backend-এ non-termination নিয়মিত বড় outage ঘটায়। যেমন:
- কোনো category বা referral tree-তে চক্রাকার parent pointer (`A → B → A`) দেখে visited set ছাড়া traversal লুপ চিরকাল ঘুরতে থাকা।
- কোনো external API 500 error দিলে সর্বোচ্চ চেষ্টা (max retry) বা timeout ছাড়াই `while (!success)` লুপ চালিয়ে যাওয়া।
- কোনো paginated API থেকে data টানার সময় `next_cursor` আগের cursor-এর সমান হয়ে গেলে লুপ না থামা।
- দুটি database transaction একে অপরের lock ছাড়ার অপেক্ষায় অনন্তকাল আটকে থাকা (**deadlock**—যার ভেতরের কৌশল আমরা Phase 6 ও 10-এ পড়ব)।

এসব ক্ষেত্রেই কোনো ভুল সংখ্যা database-এ না লেখা হলেও service-টি আটকে গিয়ে resource নিঃশেষ করে দেয়। তাই প্রকৃত backend correctness-এ শুধু *"সঠিক ফলাফল দেবে"* নয়, *"অবশ্যই নিরাপদে শেষ হবে"*—এটাও সমান অপরিহার্য।

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-03-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একটি e-commerce inventory service-এ `reserveStock(productId, quantity)` নামে একটি operation আছে, যা order confirm হওয়ার সময় কোনো পণ্যের মজুত (`available_stock`) কমায়। এই operation-টির জন্য একটি করে স্পষ্ট **Precondition**, **Postcondition**, এবং **Invariant** লিখুন—এবং এক বাক্যে দেখান, ওই invariant ভেঙে গেলে পরের কোন downstream কাজটি নীরবে ভুল আচরণ করবে।
> 2. একটি backend function কীভাবে **partially correct** হয়েও **totally correct** না-ও হতে পারে? ওপরের fallback-promo উদাহরণ বাদ দিয়ে নিজের একটি বাস্তব backend উদাহরণ দিন।

এখন একটু গভীরে ঢুকুন। আমাদের `LAUNCH50` bug-টি কোডের ভেতরে টানা চৌদ্দ দিন ঘাপটি মেরে বসে ছিল, অথচ পনেরোতম দিন সন্ধ্যায় হঠাৎ কেন তা customer-দের টাকা ও accounting ledger-এ বিপর্যয় ঘটাল? কোডের ভেতরের একটি ছোট্ট ত্রুটি ঠিক কোন কার্যকারণ পথে এগিয়ে একটি production outage-এ পরিণত হয়—আর সেই পথের কোথায় কোথায় আমরা তাকে আটকাতে পারি?

---

## একটি ত্রুটি যেভাবে Outage-এ পরিণত হয়: Fault → Error → Failure

দৈনন্দিন সফটওয়্যার আলোচনায় আমরা প্রায় সবকিছুকেই এক নামে ডাকি—"bug" অথবা "error"। কেউ বলেন *"কোডে একটা error আছে"*, কেউ বলেন *"database-এ error data ঢুকে গেছে"*, আবার কেউ বলেন *"API-টা error দিচ্ছে"*। কিন্তু যখন কোনো production incident তদন্ত করতে হয় বা নির্ভরযোগ্যতা (dependability) ডিজাইন করতে হয়, তখন এই তিনটি আলাদা জিনিসকে এক শব্দে গুলিয়ে ফেললে কার্যকারণ সম্পর্ক ঝাপসা হয়ে যায়।

Dependable computing-এর সবচেয়ে প্রামাণ্য আন্তর্জাতিক ভিত্তিপ্রবন্ধ—Algirdas Avižienis, Jean-Claude Laprie, Brian Randell ও Carl E. Landwehr-এর ***"Basic Concepts and Taxonomy of Dependable and Secure Computing"* (*IEEE TDSC*, 2004)**—এই বিভ্রান্তি দূর করে একটি নির্ভুল, তিন-ধাপের কার্যকারণ শৃঙ্খল (causal chain) দাঁড় করিয়েছে:

```text
Dormant Fault  ──(Activation)──▶  Active Fault  ──▶  Internal Error (Latent / Detected)  ──(Propagation)──▶  Service Failure
```

আমাদের `LAUNCH50` (৩০ টাকার অর্ডারে ৫০ টাকা ছাড়) ঘটনাটি দিয়ে এই শৃঙ্খলের প্রতিটি ধাপ ধীরস্থিরভাবে মিলিয়ে নিন:

1. **Fault (ত্রুটি — The adjudged or hypothesized cause of an error):**
   কোড, design, configuration বা hardware-এর ভেতরে থাকা মূল অন্তর্নিহিত ত্রুটি—যা কোনো ভুল অভ্যন্তরীণ অবস্থার কারণ হতে পারে। আমাদের কোডে `finalTotal = orderTotal - discount` লেখার সময় নিম্নসীমা (`0` এর নিচে নামতে না দেওয়া বা ন্যূনতম অর্ডার-মূল্য যাচাই) বাদ পড়ে যাওয়া হলো **Fault**।
   - প্রথম চৌদ্দ দিন যখন প্রতিটি অর্ডার ৫০ টাকার বেশি ছিল, তখন ওই bug-টি কোডে থাকা সত্ত্বেও কোনো ভুল state তৈরি করেনি। Avižienis et al. এই অবস্থাকে বলেন **Dormant Fault (সুপ্ত ত্রুটি)**।
   - শুক্রবার সন্ধ্যায় ৩০ টাকার স্টিকারে `LAUNCH50` প্রয়োগ করা মাত্রই সেই নির্দিষ্ট input ও শর্ত ত্রুটিটিকে জাগিয়ে তুলল (**Activation**)। তখন সুপ্ত ত্রুটিটি পরিণত হলো **Active Fault**-এ।

2. **Error (ভুল অভ্যন্তরীণ অবস্থা — Part of the system state that may cause a service failure):**
   Active fault যখন system-এর ভেতরের state-কে (memory-র variable, database-এর row, message payload) তার সঠিক অবস্থা থেকে বিচ্যুত করে, তখন সেই বিকৃত অভ্যন্তরীণ অবস্থাকেই বলে **Error**। আমাদের ঘটনায় `order.final_total = -20` হিসাব হওয়া এবং database-এ `-20` সংরক্ষিত হওয়া হলো **Error**। লক্ষ করুন, এই মুহূর্ত পর্যন্ত বাইরের দুনিয়া বা accounting system হয়তো এখনো ক্ষতি দেখেনি—ক্ষতিটি সিস্টেমের ভেতরের state-এ জমে আছে।
   - Error-এর দুটি অবস্থা থাকতে পারে: যতক্ষণ সিস্টেম নিজেই টের না পায় যে state-টি ভুল হয়ে গেছে (আমাদের `orders` table-এ `-20` চুপচাপ বসে আছে), ততক্ষণ তা **Latent Error (লুকায়িত error)**। আর যদি কোনো assertion, validation check বা PostgreSQL-এর `CHECK (final_total >= 0)` constraint সঙ্গে সঙ্গে এই অবৈধ মানটিকে ধরে ফেলে, তবে তা হয় **Detected Error**।

3. **Service Failure (বাহ্যিক সেবা-ব্যর্থতা — Deviation of delivered service from correct service):**
   একটি latent error যখন ধরা না পড়ে সিস্টেমের সীমানা (service boundary) পর্যন্ত ছড়িয়ে পড়ে (**propagation**) এবং বাইরে প্রদত্ত সেবা তার specification থেকে বিচ্যুত হয়, তখন ঘটে **Service Failure**। আমাদের ঘটনায় ভুল `-20` state-টি পরদিন accounting worker-এ পৌঁছে গ্রাহকের wallet-এ অযাচিত ২০ টাকা credit করে দিল (কিংবা checkout boundary-তে অবৈধ মূল্য পাঠিয়ে payment ব্যর্থ করল)—এটাই বাইরের জগতে দৃশ্যমান **Service Failure**।

এখানে একটি অসাধারণ বাস্তব চক্র (recursion) খেয়াল করুন, যা Avižienis et al. বিশেষভাবে দেখিয়েছেন: **একটি উপাদানের Service Failure তার ব্যবহারকারী আরেকটি সিস্টেমের জন্য নতুন একটি External Fault হয়ে দাঁড়ায়!** যেমন, external payment provider যদি হঠাৎ timeout করে (provider-এর দৃষ্টিতে সেটি একটি *Service Failure*), তবে আমাদের checkout service-এর দৃষ্টিতে সেটি একটি বাহ্যিক **Fault**—যা সঠিকভাবে সামলাতে না পারলে আমাদের ভেতরে অর্ধসমাপ্ত order state (*Error*) তৈরি হয়ে শেষে আমাদের customer-এর কাছে checkout *Failure* হয়ে দেখা দেবে।

আর একটি সূক্ষ্ম সত্য মনে রাখবেন: প্রতিটি fault-ই error তৈরি করে না (সুপ্ত fault কখনো activate না-ও হতে পারে), আর প্রতিটি error-ও failure-এ পৌঁছায় না—**যদি আমরা মাঝপথেই error-টিকে ধরে ফেলে শোধরাতে পারি!**

ঠিক এই জায়গা থেকেই নির্ভরযোগ্য সিস্টেম তৈরির পুরো বিজ্ঞানটি খুলে যায়। Avižienis et al. (2004) **Dependability** *(কোনো সিস্টেমের প্রদত্ত সেবার ওপর যুক্তিসঙ্গত আস্থা রাখার সক্ষমতা)*-কে পাঁচটি মূল গুণাবলিতে ভাগ করেছেন—**Availability** (প্রয়োজনের সময় সঠিক সেবা দিতে প্রস্তুত থাকা), **Reliability** (সময়জুড়ে নিরবচ্ছিন্ন সঠিক সেবা দেওয়া), **Safety** (ব্যবহারকারী বা পরিবেশের ওপর মারাত্মক ক্ষতি না ঘটানো), **Integrity** (অবৈধ পরিবর্তন বা বিকৃতি না হওয়া), এবং **Maintainability** (নিরাপদে মেরামত ও পরিবর্তন করার সক্ষমতা)—আর এর সাথে **Confidentiality** যুক্ত হয়ে গঠিত হয় **Security**।

এই dependability অর্জন করার জন্য Avižienis et al. **চারটি স্বতন্ত্র উপায় (Four Means of Dependability)** চিহ্নিত করেছেন। ভালো করে দেখুন, প্রতিটি উপায় Fault → Error → Failure শৃঙ্খলের ঠিক কোন বিন্দুতে কাজ করে, এবং প্রতিটি উপায়ের নিজস্ব সীমাবদ্ধতা কোথায়:

| Dependability Mean | শৃঙ্খলের কোথায় কাজ করে | এর মূল উদ্দেশ্য ও বাস্তব Backend উদাহরণ | এর মৌলিক সীমাবদ্ধতা |
| :--- | :--- | :--- | :--- |
| **1. Fault Prevention** *(ত্রুটি প্রতিরোধ)* | Fault তৈরি হওয়ার **আগেই** | কোডে বা design-এ ত্রুটি ঢোকাতেই না দেওয়া। যেমন: Block 2-এর স্পষ্ট BCP 14 spec ও invariant সংজ্ঞা, strong type system, নিরাপদ abstraction (যেমন parameterized SQL ব্যবহারের নিয়ম), পরিচ্ছন্ন modular design ও secure coding অভ্যাস। | মানুষ, পরিবর্তনশীল requirement ও বাইরের dependency কখনোই শতভাগ নিখুঁত হয় না—তাই সব fault জন্মের আগেই আটকানো অসম্ভব। |
| **2. Fault Removal** *(ত্রুটি অপসারণ)* | Dormant বা Active Fault **খুঁজে দূর করা** | সিস্টেমে ইতিমধ্যে ঢুকে পড়া ত্রুটি সংখ্যা ও তীব্রতা কমানো—উন্নয়নকালে (code review, static analysis, automated testing, debugging) এবং production-এ চালানোর সময় (incident-এর পর root-cause bug fix ও patch করা)। | কোনো review বা finite test suite দিয়ে সিস্টেমে থাকা শতভাগ সুপ্ত fault খুঁজে বের করা যায় না। |
| **3. Fault Tolerance** *(ত্রুটি সহনশীলতা)* | Fault সক্রিয় হয়ে Error হওয়ার **পর**, Failure হওয়ার **আগে** | সিস্টেমে fault থাকা সত্ত্বেও এবং error ঘটলেও যেন বাইরের সেবা specification অনুযায়ী চালু থাকে—error detection এবং recovery-র মাধ্যমে। যেমন: PostgreSQL-এ `CHECK (final_total >= 0)` বা `UNIQUE` constraint দিয়ে অবৈধ মান ঢোকা মুহূর্তেই ধরা (error detection) ও নিরাপদে transaction rollback করা, timeout ও bounded retry, external email provider বন্ধ থাকলেও order confirmation চালু রাখা, বা redundant server রাখা। | **গুরুত্বপূর্ণ:** Fault tolerance কোনো fault *প্রতিরোধ করে না*—এটি fault-এর উপস্থিতিতে error ধরে ফেলে failure ঠেকায়। তাছাড়া অতিরিক্ত fault-tolerance স্তর নিজেই নতুন জটিলতা ও নতুন bug তৈরি করতে পারে, এবং ভুল specification-কে ঠিক করতে পারে না। |
| **4. Fault Forecasting** *(ত্রুটি পূর্বাভাস)* | পুরো lifecycle জুড়ে **মূল্যায়ন ও অনুমান** | বর্তমানে কত ত্রুটি থাকতে পারে, ভবিষ্যতে কত ঘন ঘন তা সক্রিয় হতে পারে এবং ঘটলে তার পরিণতি কী হবে তা অনুমান করা। যেমন: production-এ error rate ও latency trend বিশ্লেষণ, load/stress testing-এর মাধ্যমে ভেঙে পড়ার সীমা মাপা, বা failure-mode review করা (বিস্তারিত Phase 20–21 ও 25-এ)। | পূর্বাভাস অতীতের তথ্য ও অনুমানের ওপর নির্ভরশীল; হঠাৎ অপ্রত্যাশিত traffic বা নতুন কোড deploy-এর সময় আচরণ বদলে গেলে পুরোনো অনুমান ব্যর্থ হতে পারে। |

এখানে একটি বিভ্রান্তি কখনোই করবেন না: **Fault Prevention আর Fault Tolerance এক নয়।** শুরুতেই স্পষ্ট invariant, strong typing ও নিরাপদ coding অভ্যাস মেনে কোডে bug ঢোকাই ঠেকানো হলো **Fault Prevention**। অন্যদিকে application কোডে bug ঢুকে পড়লেও PostgreSQL-এর `CHECK (final_total >= 0)` constraint দিয়ে অবৈধ `-20` মানটিকে database-এ ঢোকার আগেই ধরে ফেলা (error detection) ও নিরাপদে rollback করা, কিংবা external payment gateway হঠাৎ timeout করলে hang না হয়ে bounded timeout ও graceful response দেওয়া—এসব হলো **Fault Tolerance** (fault থাকা সত্ত্বেও service failure ঠেকানো)। Production-grade backend কখনো এই চারটির যেকোনো একটির ওপর দাঁড়িয়ে থাকে না; চারটির সমন্বয়েই dependability তৈরি হয়।

এখন একটু থেমে একটি তীক্ষ্ণ প্রশ্ন করুন। ধরুন, আপনি **Fault Removal**-এ দারুণ পরিশ্রম করলেন: আপনার specification-এর প্রতিটি লাইনের বিপরীতে ডজনখানেক automated test লিখলেন, সব কটি পাস করল, আর কোনো crash নেই। **তবুও এমন কী দুটি মৌলিক কারণে আপনার পুরোপুরি tested system-টি এখনো ব্যর্থ হতে পারে?**

---

## Verification বনাম Validation—এবং Testing কেন কখনো Bug-এর অনুপস্থিতি প্রমাণ করে না

পেশাদার software engineering-এ প্রথম যে বড় ফাঁদটি টিমগুলোকে ডুবিয়ে দেয়, তা হলো দুটি সম্পূর্ণ ভিন্ন প্রশ্নকে এক করে ফেলা:

- **Verification (*"Are we building the system right?"* — সিস্টেমটিকে কি সঠিকভাবে বানাচ্ছি?)**
  এটি যাচাই করে যে আপনার design এবং কোড **লিখিত specification, contract ও invariant** হুবহু মেনে চলছে কি না। Code review, static analysis, unit test, integration test—এসবের মূল কাজ হলো Verification।
- **Validation (*"Are we building the right system?"* — আমরা কি সঠিক সিস্টেমটাই বানাচ্ছি?)**
  এটি যাচাই করে যে তৈরি হওয়া system (এবং এর পেছনের specification) আসলেই **ব্যবহারকারী ও ব্যবসার প্রকৃত সমস্যা** সমাধান করছে কি না।

এই দুটি প্রশ্ন আলাদা হওয়ার কারণে একটি সিস্টেম অনায়াসেই একটিতে ১০০% পাস করে অন্যটিতে পুরোপুরি ব্যর্থ হতে পারে! বাস্তব দুটি উদাহরণ দেখুন:

1. **Verification-এ পাস, কিন্তু Validation-এ ফেল:**
   Block 2-এর সেই nightly receipt cron job-এর কথা মনে করুন! যদি ticket ও specification-এ আক্ষরিকভাবে লেখা থাকে *"রাত ১২টায় cron job চালিয়ে সেদিনের receipt email করতে হবে"*, এবং আপনি দারুণ retry, idempotency, আর ১০০% automated test দিয়ে সেই cron job-টিকে নিখুঁতভাবে তৈরি করেন—তবে আপনার কাজ **Verification-এ সম্পূর্ণ পাস**। কিন্তু পরদিন সকালেই কর্পোরেট customer রসিদ না পেয়ে অর্ডার বাতিল করবেন, কারণ আপনার specification-টাই আসল মানবিক প্রয়োজন ধরতে ব্যর্থ হয়েছিল—অর্থাৎ system-টি **Validation-এ ফেল**! একইভাবে, ধরুন promo specification-এ ভুলবশত লেখা ছিল *"কোনো discount-যুক্ত পণ্যে promo code কাজ করবে না"*, আর আপনি তা নিখুঁতভাবে verify করলেন—অথচ Black Friday-তে Marketing-এর পুরো campaign-টাই ছিল clearance পণ্যের ওপর বাড়তি ১০% ছাড় দেওয়া!
2. **Validation-এ আপাতত পাস, কিন্তু Verification-এ ফেল:**
   উল্টোদিকে, ধরুন আপনার `LAUNCH50` feature-টি product demo-তে Marketing Manager-কে মুগ্ধ করল: ৫০০ টাকার অর্ডারে ঠিকঠাক ৪৫০ টাকা দেখাল, UI সুন্দর, checkout মসৃণ—ব্যবসায়িক দিক থেকে মনে হলো ঠিক যা চাওয়া হয়েছিল তা-ই হয়েছে। কিন্তু ভেতরে কোডটি `used_count <= max_total_redemptions` এবং `final_total >= 0` specification invariant-গুলো মানেনি। অর্থাৎ বাস্তব চাপ এলেই এটি **Verification-এ ভেঙে পড়ল**।

এবার ধরুন, আপনার specification একদম নির্ভুল (Validation ঠিক আছে)—তাহলে কি ১০০% সবুজ automated test suite-এর মাধ্যমে আমরা অন্তত **Verification** (এবং কোডে bug না থাকা) চূড়ান্তভাবে প্রমাণ করে ফেলতে পারি?

এখানেই ১৯৬৯–১৯৭০ সালে ডাচ কম্পিউটার বিজ্ঞানী **Edsger W. Dijkstra** তাঁর যুগান্তকারী প্রবন্ধ ***Notes on Structured Programming* (EWD249)**-এর Section 3 (*"On the Reliability of Mechanisms"*)-এ একটি অকাট্য যুক্তি তুলে ধরেছিলেন, যা আজও সফটওয়্যার প্রকৌশলের অন্যতম গভীর সত্য হিসেবে বিবেচিত হয়:

> *"Program testing can be used to show the presence of bugs, but never to show their absence!"*

কেন? এর পেছনে কোনো হতাশাবাদ নেই—আছে নিখাদ গাণিতিক ও যৌক্তিক অসামঞ্জস্য (logical asymmetry)।

আমাদের ছোট্ট `applyPromoCode` endpoint-টির কথাই ভাবুন। এর ভেতর দিয়ে কী কী input ও অবস্থা আসতে পারে?
- `orderTotal`-এর সম্ভাব্য মান (০, ১, ২৯.৯৯, ৫০, ৫০.০১, লক্ষ টাকা, ঋণাত্মক বা অসংখ্যা input),
- `promoCode`-এর ধরন (percentage, flat, expired ১ সেকেন্ড আগে, exact expiry সেকেন্ডে, অজানা কোড, খালি string, Unicode বা বিশাল string),
- `used_count`-এর অবস্থা (`0`, `999`, `1000`),
- ব্যবহারকারীর ধরন (নতুন, পুরোনো, নিষিদ্ধ, অনুমোদনহীন),
- সময় ও timezone-এর সীমানা (UTC midnight, leap year, clock skew),
- এবং একই মিলিসেকেন্ডে সমান্তরালভাবে আসা ২, ৫, বা ৫০০টি request-এর অসংখ্য সম্ভাব্য interleaving ক্রম!

এই কয়েকটি মাত্রার সম্ভাব্য সমন্বয় (**input ও state space**) গুণ করলে সংখ্যাটি এতই মহাজাগতিকভাবে বিশাল হয় যে পৃথিবীর সব কম্পিউটার একসাথে কোটি বছর চালালেও প্রতিটি সম্ভাব্য অবস্থা test করা অসম্ভব (**exhaustive testing is infeasible**)।

তাই যেকোনো বাস্তব test suite সেই মহাসাগর থেকে মাত্র কয়েক ডজন বা কয়েক হাজার বিন্দু নমুনা (sample) হিসেবে তুলে নিয়ে পরীক্ষা করে। এর যুক্তিগত ফলাফলটা খুব স্পষ্ট:
- **একটি test ফেল করলে** সেটি একটি অকাট্য **counterexample**—অর্থাৎ তা শতভাগ নিশ্চিতভাবে প্রমাণ করে যে অন্তত একটি bug আছে (*presence of bugs*)।
- **৫০০টি test পাস করলে** তা কেবল প্রমাণ করে যে ঠিক ওই ৫০০টি নির্দিষ্ট input ও পরিবেশে কোডটি আপনার test-এ লেখা assertion-গুলো মেনে চলেছে। ৫০১তম অজানা অবস্থা—যেমন ৩০ টাকার অর্ডার, দুটি concurrent request, বা চক্রাকার `fallback_promo_id`—যে ভাঙবে না, তা ৫০০টি সবুজ টিকচিহ্ন কখনোই প্রমাণ করতে পারে না (*never their absence*)।
- আরও একটি সূক্ষ্ম কারণ আছে: test-গুলো তো মানুষ (বা AI) লিখেছে! লেখক যদি কোনো edge case বা concurrency ঝুঁকির কথা কল্পনাই না করেন, তবে সেই একই blind spot কোড এবং test—দুই জায়গাতেই রয়ে যাবে, আর সব test নির্বিকারভাবে সবুজ দেখাবে!

**খুব সতর্ক থাকুন—Dijkstra-র এই কথাকে কখনো *"তাহলে testing লিখে লাভ কী?"* অর্থে নেবেন না!** Systematic automated testing (যা আমরা Phase 15-এ গভীরভাবে শিখব) হলো **Fault Removal**-এর সবচেয়ে কার্যকর ও অপরিহার্য হাতিয়ারগুলোর একটি—এটি দ্রুত regression ধরে, refactored কোডকে নিরাপদ রাখে এবং bug-এর সম্ভাবনা নাটকীয়ভাবে কমিয়ে আনে। Dijkstra-র আসল বার্তা হলো: **শুধু পরে test চালিয়ে কোনো কোডের ওপর নির্ভরযোগ্যতা চাপিয়ে দেওয়া যায় না।** Testing-এর পাশাপাশি আমাদের দরকার **গঠনগত যুক্তি (structural reasoning)**—পরিষ্কার specification, explicit preconditions/postconditions/invariants, শক্তিশালী database constraint ও type system, মননশীল code review, এবং edge case ও concurrency নিয়ে সচেতন চিন্তা।

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-03-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন developer বললেন: *"আমাদের SQL query-তে `WHERE user_id = $1` দিতে ভুলে গিয়েছিলাম, ফলে user-রা অন্যের invoice দেখে ফেলেছে—এটা একটা ছোট্ট error ছিল, পরে try/catch বসিয়ে fault tolerance করে দিয়েছি।"* Avižienis et al. (2004)-এর ভাষায় এই বক্তব্যে দুটি বড় পরিভাষাগত ও ধারণাগত ভুল কোথায়? (ক) এখানে কোনটি **Fault**, কোনটি **Error**, এবং কোনটি **Failure**? (খ) `WHERE` clause ঠিক করা কি **Fault Tolerance**, নাকি অন্য কোনো dependability mean?
> 2. একটি backend feature কীভাবে **Verification-এ পাস করেও Validation-এ ফেল** করতে পারে—নিজের একটি নতুন উদাহরণ দিন। আর কেন *"আমাদের ১০০% line test coverage আছে এবং সব test পাস করেছে"* বললেই software-টি bug-free বলে প্রমাণ হয় না?

এবার আমাদের কেন্দ্রীয় প্রশ্নের দ্বিতীয় অর্ধেকে প্রবেশ করুন। ধরুন, আপনি সত্যিই অসাধারণ কাজ করলেন: আপনার checkout ও promo service-টি Validation-এ উত্তীর্ণ, প্রতিটি functional invariant নির্ভুলভাবে মেনে চলে, প্রতিবার নিরাপদে terminate করে, আর একটিও ভুল টাকা হিসাব করে না। **তবুও এমন একটি "সঠিক" backend system কীভাবে production-এ জঘন্য মানের (low quality) হতে পারে?**

---

## শুধু "সঠিক Output" মানেই ভালো Software নয়: বহুমাত্রিক Quality ও ISO/IEC 25010:2023

নিচের তিনটি কাল্পনিক—অথচ বাস্তব দুনিয়ায় অতিপরিচিত—backend সার্ভিসের কথা ভাবুন, যেগুলোর প্রতিটিই তাদের মূল functional হিসাব শতভাগ সঠিকভাবে করে:

- **Service A:** প্রতিটি promo calculation নির্ভুল, কিন্তু প্রতিটি request-এ ৬ সেকেন্ড সময় নেয় এবং ৫০ জন user একসাথে এলেই server-এর memory ফুরিয়ে crash করে।
- **Service B:** হিসাব নির্ভুল ও দ্রুত, কিন্তু যেকোনো logged-in user URL-এ `order_id` বদলে অন্য customer-এর ব্যক্তিগত ঠিকানা ও ক্রয়ের ইতিহাস দেখতে পান।
- **Service C:** হিসাব নির্ভুল, দ্রুত ও নিরাপদ—কিন্তু কোডটি এতই জটপাকানো ও অস্পষ্ট যে নতুন একটি tax নিয়ম যোগ করতে তিনজন engineer-এর দুই সপ্তাহ লাগে এবং প্রতিবার নতুন bug ঢুকে পড়ে।

এদের কোনোটিকে কি আপনি "high-quality software" বলবেন? কখনোই না! ঠিক তেমনই উল্টো ভুল ধারণাটিও সমান ক্ষতিকর: অনেকেই মনে করেন *"পরিষ্কার, সুন্দর ফরম্যাটের কোড (clean code) মানেই ভালো মানের software।"* অথচ একদম ঝকঝকে, পরিপাটি syntax দিয়ে লেখা কোডও যদি customer-দের ব্যক্তিগত তথ্য (PII) ফাঁস করে, অসঙ্গত data invariant তৈরি করে, বা সামান্য চাপেই অচল হয়ে পড়ে—তবে সেটি নিঃসন্দেহে নিম্নমানের software।

সফটওয়্যার প্রকৌশলে "high quality" কোনো অস্পষ্ট বিশেষণ নয়—এটি একটি **বহুমাত্রিক (multidimensional)** কাঠামো। এর বর্তমান আন্তর্জাতিক মানদণ্ড হলো **ISO/IEC 25010:2023** *(Systems and software engineering — SQuaRE — Product quality model, ২য় সংস্করণ, নভেম্বর ২০২৩)*।

একটি জরুরি সতর্কতা: ইন্টারনেটের অনেক পুরোনো ব্লগ ও বইয়ে এখনো **ISO/IEC 25010:2011**-এর পুরোনো ৮-বৈশিষ্ট্যের তালিকা দেখা যায় (যেখানে *Usability* ও *Portability* ছিল এবং *Safety* আলাদাভাবে ছিল না)। কিন্তু **২০২৩ সালের হালনাগাদ সংস্করণে** তিনটি বড় পরিবর্তন এসেছে:
1. পুরোনো *Usability*-র নাম বদলে হয়েছে **Interaction Capability**।
2. পুরোনো *Portability*-র নাম বদলে হয়েছে **Flexibility**, এবং এর ভেতর **Scalability**-কে স্পষ্টভাবে যুক্ত করা হয়েছে।
3. **Safety** সম্পূর্ণ নতুন **নবম শীর্ষ-স্তরের বৈশিষ্ট্য** হিসেবে যুক্ত হয়েছে। *(পাশাপাশি Reliability-র ভেতরে আগের Maturity-র বদলে এসেছে Faultlessness, আর Security-র ভেতর যুক্ত হয়েছে Resistance।)*

এবার আধুনিক backend engineering-এর বাস্তব প্রেক্ষাপটে **ISO/IEC 25010:2023-এর নয়টি শীর্ষ-স্তরের বৈশিষ্ট্য** বুঝে নিন—সব কটিকে জোর করে একটি promo service-এ না গুঁজে, যেখানে যে বাস্তব backend উদাহরণটি সবচেয়ে স্পষ্ট সেখানেই তা দেখুন:

1. **Functional Suitability (কার্যগত উপযোগিতা):**
   সিস্টেমটি নির্ধারিত ও প্রত্যাশিত কাজগুলো কতটা সম্পূর্ণ, সঠিকভাবে ও যথাযথভাবে সম্পন্ন করে। যেমন: আমাদের promo service-এ discount, expiry ও usage-limit-এর প্রতিটি নিয়ম নির্ভুলভাবে পালন করা—এটিই আমাদের আগের অংশে আলোচিত functional correctness-এর সরাসরি প্রতিফলন।
2. **Performance Efficiency (কর্মক্ষমতার দক্ষতা):**
   ব্যবহৃত সম্পদের (CPU, memory, database I/O, network bandwidth) তুলনায় সিস্টেমটি কতটা দ্রুত ও দক্ষভাবে কাজ করে—এর মধ্যে response time ও throughput (**time behavior**), সম্পদের ব্যয় (**resource utilization**), এবং সর্বোচ্চ ধারণক্ষমতা (**capacity**) পড়ে। যেমন: ১০০,০০০ promo কোডের মধ্যে থেকে ১০০ RPS load-এ `p95 <= 150 ms` সময়ে validation শেষ করা এবং অযথা পুরো table RAM-এ টেনে না আনা।
3. **Compatibility (সামঞ্জস্যতা):**
   সিস্টেমটি অন্য software, API বা বাহ্যিক সিস্টেমের সাথে কতটা নির্বিঘ্নে তথ্য আদান-প্রদান করতে পারে (**interoperability**) এবং একই server বা পরিবেশে অন্য service-এর ক্ষতি না করে চলতে পারে (**co-existence**)। যেমন: একটি billing backend-এর প্রমিত JSON ও HTTP webhook contract মেনে payment gateway ও accounting ERP-এর সাথে কাজ করা, বা shared PostgreSQL cluster-এ একটি reporting job-এর কারণে live checkout query-গুলো অচল না হয়ে যাওয়া।
4. **Interaction Capability *(আগে Usability)*:**
   ব্যবহারকারীরা কতটা সহজে, নির্ভুলভাবে, কম বিভ্রান্তিতে ও নিরাপদভাবে সিস্টেমটির সাথে কাজ করতে পারেন। Backend engineer-রা প্রায়ই ভাবেন এটা শুধু frontend UI-এর বিষয়—আসলে মোটেই নয়! একটি public API-র ক্ষেত্রে স্পষ্ট ও অনুমেয় endpoint নকশা, অর্থবহ structured error code (`PROMO_EXPIRED`-এর সাথে কী কারণে বাতিল হলো তার স্পষ্ট ইঙ্গিত), নির্ভুল API documentation, এবং internal support/admin tool-এর সহজবোধ্যতা—সবই backend-এর Interaction Capability।
5. **Reliability (নির্ভরযোগ্যতা):**
   নির্দিষ্ট শর্ত ও সময়কাল জুড়ে সিস্টেমটি কতটা ধারাবাহিকভাবে সঠিক সেবা দেয়—স্বাভাবিক অবস্থায় ত্রুটিমুক্ত চলা (**faultlessness**), প্রয়োজনের সময় সচল থাকা (**availability**), ত্রুটি ঘটলেও সেবা ধরে রাখা (**fault tolerance**), এবং কোনো crash বা বিভ্রাটের পর data ও স্বাভাবিক অবস্থা পুনরুদ্ধার করতে পারা (**recoverability**)। যেমন: database restart বা external email outage-এর পরও কোনো confirmed order না হারানো এবং দ্রুত স্বাভাবিক সেবায় ফেরা।
6. **Security (নিরাপত্তা):**
   সিস্টেম ও তার তথ্যকে অননুমোদিত প্রবেশ, ফাঁস বা বিকৃতি থেকে সুরক্ষিত রাখা—গোপনীয়তা (**confidentiality**), তথ্যের অখণ্ডতা (**integrity**), প্রকৃত পরিচয় প্রমাণ (**authenticity**), কে কী করেছে তার চিহ্ন রাখা (**accountability** ও **non-repudiation**), এবং ইচ্ছাকৃত আক্রমণের মুখে টিকে থাকার সক্ষমতা (**resistance**)। যেমন: কোনো user যেন অন্য user-এর order ইতিহাস দেখতে না পারেন, password যেন নিরাপদে সংরক্ষিত থাকে, এবং credential stuffing বা SQL injection আক্রমণ প্রতিরোধ করা (বিস্তারিত Phase 18 ও 19-এ)।
7. **Maintainability (রক্ষণাবেক্ষণযোগ্যতা):**
   ভবিষ্যতের engineer-রা কতটা কম কষ্ট ও কম ঝুঁকিতে সিস্টেমটিকে বুঝতে (**analysability**), পরিবর্তন করতে (**modifiability**), অংশগুলোকে আলাদা রাখতে (**modularity**), পুনর্ব্যবহার করতে (**reusability**), এবং স্বয়ংক্রিয়ভাবে পরীক্ষা করতে (**testability**) পারেন। Block 1-এর software sustainability সরাসরি এই বৈশিষ্ট্যের ওপর দাঁড়িয়ে আছে।
8. **Flexibility *(আগে Portability; এতে Scalability যুক্ত হয়েছে)*:**
   পরিবর্তিত পরিবেশ, অবকাঠামো বা ক্রমবর্ধমান চাহিদার সাথে সিস্টেমটি কতটা সহজে খাপ খাওয়াতে পারে—যেমন ভিন্ন cloud বা Linux পরিবেশে সহজে deploy করা (**adaptability / installability**), পুরোনো উপাদান বদলে নতুনটি বসানো (**replaceability**), এবং ১০ গুণ user ও data বাড়লে মৌলিক পুনর্লিখন ছাড়াই পরিসর বাড়ানো (**scalability**)। যেমন: local development থেকে Docker container ও cloud PostgreSQL-এ কোনো hardcoded path ছাড়াই চলতে পারা, কিংবা single-instance app-কে stateless রেখে একাধিক instance-এ scale করতে পারা।
9. **Safety *(২০২৩ সংস্করণে নতুন শীর্ষ-স্তরের বৈশিষ্ট্য)*:**
   বিপজ্জনক বা অস্বাভাবিক পরিস্থিতিতে মানুষ, সম্পদ বা পরিবেশের ওপর **মারাত্মক ক্ষতি (catastrophic harm)** এড়ানোর সক্ষমতা—যেমন নিরাপদ সীমা আরোপ (**operational constraint**), ঝুঁকি শনাক্ত করা (**risk identification**), বিপজ্জনক অবস্থায় নিরাপদভাবে থেমে যাওয়া (**fail safe**), এবং আগাম সতর্কতা দেওয়া (**hazard warning**)। যেমন: একটি স্বাস্থ্যসেবা backend যেখানে রোগীর ওষুধের মাত্রা নিরাপদ সীমার বাইরে গেলে prescription অনুমোদন আটকে দেয়, কিংবা একটি স্বয়ংক্রিয় financial payout backend যেখানে এক মিনিটে অস্বাভাবিক পরিমাণে টাকা বেরিয়ে যেতে থাকলে স্বয়ংক্রিয় circuit-breaker সমস্ত payout থামিয়ে দিয়ে operator-কে alert করে।

এখন এই ৯টি বৈশিষ্ট্যের দিকে একটু দূর থেকে তাকিয়ে একটি অত্যন্ত গুরুত্বপূর্ণ বিভাজন দেখুন: **Internal Quality** বনাম **External Quality**।

- **External Quality (বাহ্যিক গুণমান):** সিস্টেম চলার সময় customer, API client, এবং বাইরের দুনিয়া সরাসরি যা অনুভব করে—যেমন Functional Suitability, Performance Efficiency, Reliability, Security, Interaction Capability, Compatibility ও Safety।
- **Internal Quality (অভ্যন্তরীণ গুণমান):** কোড ও স্থাপত্যের ভেতরের গঠনগত মান, যা প্রধানত engineer-রা প্রতিদিন কাজ করতে গিয়ে অনুভব করেন—মূলত **Maintainability** (modularity, analysability, modifiability, testability)।

অনেক টিম বিপজ্জনক একটি ভুল করে: তারা ভাবে *"Customer তো আর আমাদের কোডের ভেতরের সৌন্দর্য দেখে না, তাই শুধু External Quality থাকলেই চলবে!"* কিন্তু সময়ের সাথে (Block 1-এর **Time & Change**) কী ঘটে, তার কার্যকারণ শৃঙ্খলটা দেখুন:
**দুর্বল Internal Quality নীরবে External Quality-কে ধ্বংস করে দেয়।**
- কোড যদি বিশৃঙ্খল ও দুবোধ্য হয় (কম *analysability*), তবে production outage-এ root cause খুঁজে পেতে ১০ মিনিটের বদলে ৩ ঘণ্টা লাগে—সরাসরি ক্ষতিগ্রস্ত হয় **Reliability (availability/recoverability)**।
- Authorization নিয়ম যদি ২০টি ফাইলে কপি-পেস্ট করা থাকে (কম *modularity*), তবে নতুন endpoint-এ একটি চেক বাদ পড়ে—সরাসরি ভেঙে পড়ে **Security**।
- Automated test চালানো কঠিন হলে (কম *testability*), প্রতিটি ছোট পরিবর্তনেই নতুন bug ঢোকে—সরাসরি ক্ষয় হয় **Functional Suitability**।

উল্টোটাও সত্য: শুধু অভ্যন্তরীণ "clean code" বা জটিল design pattern সাজিয়ে রেখে যদি আপনি data invariant, latency, বা security অবহেলা করেন, তবে চকচকে কোডের ভেতরেও নিম্নমানের পণ্য তৈরি হবে।

এখান থেকে আমাদের শেষ ও সবচেয়ে বাস্তববাদী উপলব্ধিটি আসে: **আপনি কখনোই একসাথে ISO/IEC 25010:2023-এর সব ৯টি বৈশিষ্ট্যকে সর্বোচ্চ পর্যায়ে তুলতে পারবেন না—কারণ বাস্তব backend সিদ্ধান্তে এরা প্রায়ই পরস্পরের বিপরীতে টানে!**

কয়েকটি বাস্তব backend সংঘাত (quality attribute tensions) লক্ষ করুন:

1. **Security বনাম Performance Efficiency:**
   প্রতিটি API request-এ কঠোর payload validation, cryptographically signed token যাচাই, সংবেদনশীল field-এর encryption এবং বিস্তারিত tamper-evident audit log লেখা **Security** (integrity, confidentiality, accountability) অনেক বাড়ায়। কিন্তু ঠিক সেই প্রতিটি অতিরিক্ত ধাপ CPU, I/O এবং response latency বাড়িয়ে **Performance Efficiency**-কে নিচে টানে।
2. **Security বনাম Interaction Capability:**
   Login ব্যর্থ হলে যদি API স্পষ্টভাবে বলে *"এই email ঠিক আছে, কিন্তু password ভুল"*, তবে বৈধ ব্যবহারকারীর ভুল বুঝতে সুবিধা হয় (**Interaction Capability** বাড়ে)। কিন্তু একজন আক্রমণকারী এই বার্তা কাজে লাগিয়ে সহজেই খুঁজে বের করতে পারেন কোন কোন email আপনার সিস্টেমে নিবন্ধিত আছে (account enumeration)। তাই security-সচেতন backend প্রায়ই ইচ্ছাকৃতভাবে সাধারণ বার্তা দেয় *"Invalid email or password"*—অর্থাৎ **Security**-র জন্য সামান্য **Interaction Capability** ছাড় দেওয়া হয়।
3. **Reliability (Fault Tolerance) বনাম Maintainability:**
   একটি notification বা payment workflow-কে বিভ্রাট-সহনশীল করতে asynchronous retry queue, circuit breaker, multi-zone replica এবং fallback path যোগ করলে **Reliability** বাড়ে। কিন্তু এর বিনিময়ে সিস্টেমটির চলমান অংশের সংখ্যা বেড়ে যায়, debugging কঠিন হয় (কম *analysability*) এবং ভবিষ্যতে পরিবর্তন করা জটিল হয়—অর্থাৎ **Maintainability**-র ওপর চাপ পড়ে।
4. **Performance Efficiency বনাম Flexibility ও Maintainability:**
   কোনো analytics query-কে অত্যন্ত দ্রুত করতে নির্দিষ্ট database engine-এর জটিল, অপ্রচলিত low-level কৌশল ও বহুস্তরীয় manual cache বসালে **Performance Efficiency** অনেক বাড়তে পারে। কিন্তু সেই কোড অন্য database-এ সরানো বা নতুন টিম সদস্যের পক্ষে নিরাপদে পরিবর্তন করা কঠিন হয়ে পড়ে—অর্থাৎ **Flexibility** ও **Maintainability** কমে যায়।

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-03-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. *"আমাদের backend-এ কোড একটু এলোমেলো হলেও সমস্যা নেই, কারণ user-রা তো শুধু API response দেখে—তাদের কাছে External Quality ভালো থাকলেই হলো।"* — Cause-and-effect দেখিয়ে ব্যাখ্যা করুন, সময় গড়ানোর সাথে দুর্বল **Internal Quality** কীভাবে অন্তত **দুটি** আলাদা External Quality বৈশিষ্ট্যকে (ISO/IEC 25010:2023-এর নাম ব্যবহার করে) ক্ষতিগ্রস্ত করে।
> 2. একটি user-authentication বা financial-payout backend-এর কথা ভাবুন এবং ISO/IEC 25010:2023-এর ৯টি বৈশিষ্ট্যের (বিশেষত ২০২৩-এর নতুন বা হালনাগাদ নাম যেমন *Interaction Capability*, *Flexibility*, বা *Safety*-র অন্তত একটি ব্যবহার করে) মধ্যে এমন একটি বাস্তব নকশাগত সিদ্ধান্ত ব্যাখ্যা করুন, যেখানে একটি বৈশিষ্ট্য উন্নত করতে গেলে আরেকটি বৈশিষ্ট্যের ওপর চাপ পড়ে।

---

## সব সূত্র একসাথে বাঁধা—এবং Block 4-এর দিকে যাত্রা

এবার পেছনে তাকিয়ে দেখুন, শুরুতে যে প্রশ্নটি আমরা তুলেছিলাম (*"সব test সবুজ হলেই কি একটি backend সঠিক, আর ভালো software হওয়ার আসল অর্থ কী?"*), তার উত্তর এখন কীভাবে একটি সুসংহত engineering মানসিক কাঠামোতে দাঁড়িয়ে গেছে।

একজন পরিণত backend engineer কখনো *"আমার মেশিনে চলছে"* বা *"কোডটা দেখতে বেশ পরিষ্কার"* বলে কোনো সিস্টেমকে ভালো ঘোষণা করেন না। পরিবর্তে তিনি চারটি সংযুক্ত লেন্স দিয়ে সিস্টেমটিকে বিচার করেন:

1. **Specification ও Contracts:** কোন লিখিত specification-এর বিপরীতে আমরা conformance মাপছি? প্রতিটি operation-এর **preconditions**, **postconditions**, এবং পুরো সিস্টেমের **invariants** কী—এবং তা কি কেবল *partially correct*, নাকি নিরাপদ termination-সহ **totally correct**?
2. **Causal Chain ও Dependability:** কোথায় **dormant fault** লুকিয়ে থাকতে পারে, কোন শর্তে তা সক্রিয় হয়ে **internal error** (latent না detected?) তৈরি করবে এবং কীভাবে তা **service failure** হয়ে বাইরে বেরোতে পারে? আর এই শৃঙ্খল ভাঙতে **fault prevention**, **fault removal**, **fault tolerance**, ও **fault forecasting**-এর কোন সমন্বয় আমরা ব্যবহার করছি?
3. **Verification, Validation ও Testing-এর সীমা:** আমরা কি কেবল *"building the system right"* (Verification) করেছি, নাকি *"building the right system"* (Validation)-ও নিশ্চিত করেছি? আর আমরা কি মনে রেখেছি যে automated test কেবল bug-এর **উপস্থিতি** প্রমাণ করতে পারে, **অনুপস্থিতি** নয়?
4. **বহুমাত্রিক Quality (ISO/IEC 25010:2023):** শুধু Functional Suitability নয়—Performance Efficiency, Compatibility, Interaction Capability, Reliability, Security, Maintainability, Flexibility, এবং Safety—প্রতিটি মাত্রায়, এবং **internal বনাম external quality**-র ভারসাম্যে আমাদের সিস্টেম কোথায় দাঁড়িয়ে আছে?

কিন্তু ঠিক এই চতুর্থ বিন্দুতেই আমরা এক অনিবার্য ও রোমাঞ্চকর নতুন প্রশ্নের মুখোমুখি এসে দাঁড়ালাম।

যদি এই নয়টি quality attribute প্রায়ই পরস্পরের সাথে দ্বন্দ্বে জড়ায়—যদি Security বাড়ালে Performance Efficiency-তে চাপ পড়ে, Reliability বাড়ালে Maintainability জটিল হয়, আর প্রতিটি বাড়তি সুরক্ষাব্যবস্থার পেছনে সময়, টাকা ও টিমের সীমিত ক্ষমতা ব্যয় হয়—**তাহলে একজন backend engineer বাস্তব সীমাবদ্ধতার ভেতরে দাঁড়িয়ে কীভাবে যুক্তিসঙ্গতভাবে ঠিক করেন কোন বৈশিষ্ট্যকে অগ্রাধিকার দেবেন, কোন বিকল্প ছাড়বেন, আর কেন সেই trade-off বেছে নিলেন তা কীভাবে নথিভুক্ত করবেন?**

ঠিক এই প্রশ্নটিই আমাদের নিয়ে যাচ্ছে **Block 4: Engineering Constraints and Trade-offs**-এ।

---

## Study Plan, Learning Objectives & Mandatory Reading

> **Continuity & Ownership Notes:** আপনার সব inline check, practical exercise এবং (যদি করতে চান) optional closed-book assessment-এর উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-03-notes.md` ফাইলে লিখুন। AI-এর প্রস্তুতির সময় আপনার study hours-এ গণনা হয় না।
>
> **Progression & Verification Policy:** আনুষ্ঠানিক assessment **ঐচ্ছিক (optional)** এবং পরের block-এ এগোনোর নিয়ন্ত্রণ আপনার হাতে। আপনি চাইলে assessment সম্পন্ন করে মূল্যায়নের জন্য ChatGPT-র কাছে শেয়ার করতে পারেন (`Mastery verified`-এর প্রমাণ হিসেবে), নিজে যথেষ্ট শিখেছেন মনে করলে assessment বাদ দিয়ে এগিয়ে যেতে পারেন (`Studied — mastery not formally verified`), অথবা assessment ছাড়াই যেকোনো অস্পষ্ট ধারণা নিয়ে প্রশ্ন ও আলোচনা করতে পারেন। কোনো গুরুত্বপূর্ণ ফাঁক থাকলে ChatGPT বাড়তি যাচাইয়ের পরামর্শ দিতে পারেন, তবে assessment সম্পন্ন করা বা ChatGPT-র অনুমোদন পরের block শুরু করার বাধ্যতামূলক শর্ত নয়। Assessment বাদ দিলে তা স্বয়ংক্রিয়ভাবে `Mastery verified` বলে গণ্য হয় না।

### Flexible ~3.5-Hour (~210-Minute) Planning Reference

*এই ৩.৫ ঘণ্টার বিভাজন একটি নমনীয় পরিকল্পনা-নির্দেশিকা, কোনো কঠোর সময়সীমা নয়। মূল গুরুত্ব দিন **Guided Lesson** গভীরভাবে বোঝার ওপর—কোনো mental model পরিষ্কার হতে বাড়তি সময় লাগলে তাড়াহুড়ো করবেন না।*

| Stage | কাজ (Activity) | নমনীয় পরিকল্পনা (Planning Reference) |
| :---: | :--- | :---: |
| **1** | ওপরের Guided Lesson গভীরভাবে পড়া + `notes/block-03-notes.md`-এ Pause & Checks 1–3 সম্পন্ন করা *(মূল অগ্রাধিকার)* | ~90–100 min |
| **2** | Complementary Mandatory Reading (Avižienis et al. 2004 নির্বাচিত অংশ + arc42 ISO/IEC 25010:2023 + Dijkstra EWD249 Sec. 3) | ~45 min |
| **3** | `notes/block-03-notes.md`-এ স্বাধীনভাবে Practical Reasoning Exercises 1–2 সম্পন্ন করা | ~45–50 min |
| **4** | *(Optional)* বই/নোট বন্ধ রেখে Closed-Book Mastery Assessment Questions 1–5 লেখা (চাইলে মূল্যায়নের জন্য ChatGPT-র কাছে শেয়ার করা) | ~20–25 min *(soft target)* |
| **Total** | — | **~210 min (~3.5 h)** |

### Block 3 Learning Objectives *(Primary: Outcome 4; Supporting: Outcome 7)*

- **LO 3.1 (`Working Understanding`):** Specification conformance, **preconditions**, **postconditions**, এবং **invariants**-এর আলোকে **correctness**-কে নির্ভুলভাবে সংজ্ঞায়িত করা (partial বনাম total correctness-সহ) এবং ব্যাখ্যা করা কেন *"আমার মেশিনে happy path-এ চলছে"* বললে correctness প্রমাণ হয় না।
- **LO 3.2 (`Working Understanding`):** **Fault → Error → Failure** কার্যকারণ শৃঙ্খল (Avižienis et al., 2004) এবং dependability অর্জনের চারটি উপায় (**fault prevention**, **fault tolerance**, **fault removal**, **fault forecasting**) ব্যাখ্যা করা।
- **LO 3.3 (`Working Understanding`):** **Verification** (*"building the system right"*) বনাম **Validation** (*"building the right system"*) ব্যাখ্যা করা এবং কেন testing কেবল bug-এর *উপস্থিতি* প্রমাণ করে, কখনোই তার *অনুপস্থিতি* নয় (Dijkstra)—তা যুক্তিসহ ব্যাখ্যা করা।
- **LO 3.4 (`Working Understanding`):** **ISO/IEC 25010:2023**-এর নয়-বৈশিষ্ট্যের product quality model ব্যবহার করে বহুমাত্রিক software quality (এবং internal বনাম external quality) ব্যাখ্যা করা ও quality attribute-গুলোর মধ্যকার বাস্তব দ্বন্দ্ব চিহ্নিত করা।

### Complementary Authoritative Reading

- **Mandatory (~20 min — নির্বাচিত মূল অংশ):** Algirdas Avižienis, Jean-Claude Laprie, Brian Randell, Carl E. Landwehr, *"Basic Concepts and Taxonomy of Dependable and Secure Computing"*, *IEEE TDSC*, Vol. 1, No. 1, pp. 11–33 (Jan–Mar 2004) — [Author-Hosted PDF (Carl Landwehr)](https://www.landwehr.org/cs690/readings/avizienis04basic.pdf) | [IEEE DOI 10.1109/TDSC.2004.2](https://doi.org/10.1109/TDSC.2004.2)। মূলত **Fault, Error, Failure** সংজ্ঞা ও কার্যকারণ শৃঙ্খল, dependability **Attributes**, এবং চারটি **Means** (*Fault Prevention, Fault Tolerance, Fault Removal, Fault Forecasting*)-এ মনোযোগ দিন। Phase 1-এ গভীর hardware fault-tolerance taxonomy-র উপ-অধ্যায়গুলো বাদ দিন।
- **Mandatory (~15 min):** arc42 Quality Model — [ISO/IEC 25010:2023 Product Quality Model Overview](https://quality.arc42.org/iso-25010/)। ২০২৩ সংস্করণের **৯টি শীর্ষ-স্তরের quality characteristic** (*Functional Suitability, Performance Efficiency, Compatibility, Interaction Capability, Reliability, Security, Maintainability, Flexibility, Safety*) এবং পুরোনো ২০১১ সংস্করণ থেকে আসা পরিবর্তনগুলো দেখে নিন।
- **Mandatory (~10 min):** Edsger W. Dijkstra, *Notes on Structured Programming* (EWD249, 1969/1970) — [UT Austin PDF](https://www.cs.utexas.edu/users/EWD/ewd02xx/EWD249.PDF) | [HTML Transcription](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD02xx/EWD249.html)। **Section 3 (*"On the Reliability of Mechanisms"*)** পড়ুন—কেন testing bug-এর উপস্থিতি দেখাতে পারে কিন্তু অনুপস্থিতি নয় এবং কেন structural correctness reasoning অপরিহার্য।
- **Optional Supplementary (মূল ৩.৫ ঘণ্টার বাজেটের বাইরে, বাণিজ্যিক বই):** Martin Kleppmann, *Designing Data-Intensive Applications* (1st ed., O'Reilly, 2017 — Chapter 1-এর *Reliability* ও *Maintainability* অংশ; অথবা Chris Riccomini-র সাথে 2nd ed., O'Reilly, Feb 2026 — data systems trade-offs ও nonfunctional requirements সংক্রান্ত প্রারম্ভিক অধ্যায়)।
- **Reference (Paywalled Standard — কেনার প্রয়োজন নেই):** ISO/IEC 25010:2023, *Systems and software engineering — SQuaRE — Product quality model* (2nd ed., Nov 2023) — ওপরের বিনামূল্যে পাওয়া arc42 overview-এর পেছনের প্রামাণ্য আন্তর্জাতিক মানদণ্ড।

---

## Practical Reasoning Exercises (~45–50 Minutes Total)

> **Ownership Rule:** সব উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-03-notes.md` ফাইলে লিখুন। AI-কে দিয়ে উত্তর খসড়া করাবেন না। লক্ষ্য হলো ওপরের শেখা mental model-গুলোকে নতুন backend দৃশ্যপটে স্বাধীনভাবে প্রয়োগ করা।

### Exercise 1: Fault → Error → Failure Causal Trace (~25 minutes)

**দৃশ্যপট (Scenario — নিচের দুটি বাস্তব backend ত্রুটির যেকোনো *একটি* বেছে নিয়ে বিশ্লেষণ করুন; চাইলে বাড়তি অনুশীলনের জন্য দুটিই করতে পারেন):**

- **বিকল্প ক — Payment Idempotency Key-তে Uniqueness Constraint না থাকা:**
  একটি Node.js + PostgreSQL payment service-এ client-রা double-charge ঠেকাতে প্রতিটি চার্জ request-এ একটি `idempotency_key` পাঠায়। Developer কোডে লিখেছেন: প্রথমে `SELECT` করে দেখো `payments` table-এ ওই `idempotency_key` আগে থেকে আছে কি না; না থাকলে payment gateway-তে চার্জ করে database-এ নতুন row `INSERT` করো। কিন্তু `payments` table-এ `idempotency_key` column-এর ওপর database-স্তরে কোনো `UNIQUE` constraint দেওয়া হয়নি। কয়েক সপ্তাহ স্বাভাবিক মোবাইল নেটওয়ার্কে সব ঠিকঠাক চলছিল। একদিন দুর্বল মোবাইল সংযোগে একটি checkout app-এর প্রথম request-এর response পথে হারিয়ে যাওয়ায় app-টি স্বয়ংক্রিয়ভাবে ঠিক একই `idempotency_key` নিয়ে এক সেকেন্ডের ব্যবধানে আবার request পাঠায়—যখন প্রথম request-টি তখনো gateway-র উত্তরের অপেক্ষায় ছিল।
- **বিকল্প খ — Currency হিসাবে Binary Floating-Point Rounding ত্রুটি:**
  একটি subscription ও split-invoice backend-এ JavaScript-এর সাধারণ floating-point `Number` দিয়ে টাকার ভগ্নাংশ (পয়সা/cent) যোগ-বিয়োগ ও tax হিসাব করা হয় *(কম্পিউটারের binary floating-point সংখ্যা `0.1 + 0.2`-এর মতো অনেক দশমিক ভগ্নাংশকে একদম নিখুঁতভাবে ধরে রাখতে পারে না, ফলে সূক্ষ্ম rounding বিচ্যুতি ঘটে—এর বিস্তারিত Phase 5-এ আসবে)*। ছোটখাটো পূর্ণ-সংখ্যার (round-amount) অর্ডারে কোনো সমস্যা দেখা দেয়নি। কিন্তু মাস শেষে এক কর্পোরেট গ্রাহকের ৩টি সমান কিস্তি ও ৭.৫% ভ্যাটযুক্ত শত শত ভগ্নাংশ লাইন-আইটেম একসাথে যোগ হয়ে invoice তৈরি হওয়ার পর, গ্রাহকের পরিশোধিত মোট টাকার সাথে ledger-এর হিসাব ১–২ পয়সা অমিল থেকে যাওয়ায় automated billing reconciliation invoice-টিকে "unpaid" দেখিয়ে গ্রাহকের account স্বয়ংক্রিয়ভাবে suspend করে দেয়।

**কাজ (Tasks):**
আপনার নির্বাচিত দৃশ্যপটটির জন্য নিচের পাঁচটি অংশ নির্ভুলভাবে বিশ্লেষণ করুন:
1. **Dormant Fault:** কোড বা schema-র ভেতরে সুপ্ত অবস্থায় থাকা মূল fault-টি ঠিক কী? প্রথম কয়েক সপ্তাহ এটি কেন নীরব (dormant) ছিল?
2. **Activation Condition:** ঠিক কোন নির্দিষ্ট runtime ঘটনা বা input শর্তটি এই dormant fault-টিকে সক্রিয় (active) করল?
3. **Internal Error State:** সক্রিয় fault-টি সিস্টেমের ভেতরের কোন state-কে (memory বা database) বিকৃত করল? এটি কীভাবে শুরুতে *latent error* হিসেবে থেকে গেল (এবং কী ঘটলে তা *detected error* হতো)?
4. **Observable Service Failure:** শেষ পর্যন্ত বাইরের ব্যবহারকারী বা অন্য সিস্টেমের চোখে কোন specification-লঙ্ঘনকারী service failure-টি ঘটল?
5. **Dependability Means Interception:** Avižienis et al.-এর চারটি উপায় (**Fault Prevention**, **Fault Removal**, **Fault Tolerance**, এবং প্রযোজ্য হলে **Fault Forecasting**)-এর কোনটি এই শৃঙ্খলের ঠিক কোন ধাপে হস্তক্ষেপ করতে পারত—একটি করে বাস্তব কৌশল বলুন, এবং প্রতিটি কৌশলের একটি করে সীমাবদ্ধতা উল্লেখ করুন।

---

### Exercise 2: ISO/IEC 25010:2023 Quality Attribute Tension Matrix (~25 minutes)

**দৃশ্যপট (Scenario):**
ধরুন, আপনি একটি growing e-commerce প্ল্যাটফর্মের **Order-Processing API**-এর নকশা ও মান পর্যালোচনা করছেন—যেখানে প্রতি মিনিটে শত শত order তৈরি, যাচাই, payment-সমন্বয় এবং downstream fulfillment সিস্টেমে পাঠানো হয়।

**কাজ (Tasks):**
1. **ISO/IEC 25010:2023**-এর ৯টি শীর্ষ-স্তরের বৈশিষ্ট্য থেকে **৪টি বৈশিষ্ট্য** বেছে নিন (যার মধ্যে অন্তত **একটি** যেন ২০২৩-এর নতুন বা হালনাগাদ নাম—*Interaction Capability*, *Flexibility*, বা *Safety*—হয় এবং অন্তত **একটি** যেন *Internal Quality* হিসেবে *Maintainability* হয়)।
2. এই ৪টি বৈশিষ্ট্য ব্যবহার করে **কমপক্ষে ৪টি ভিন্ন জোড়া (pairs)** বা বাস্তব নকশাগত সিদ্ধান্ত তৈরি করুন, যেখানে একটি বৈশিষ্ট্য উন্নত করার জন্য নেওয়া কোনো সুনির্দিষ্ট backend সিদ্ধান্ত আরেকটি বৈশিষ্ট্যকে দুর্বল করে (বা তার ওপর বাস্তব চাপ সৃষ্টি করে)।
3. প্রতিটি নকশাগত সিদ্ধান্তের জন্য স্পষ্টভাবে লিখুন:
   - **(ক)** ঠিক কোন কারিগরি বা নকশাগত সিদ্ধান্তটি নেওয়া হয়েছে;
   - **(খ)** কোন বৈশিষ্ট্যটি উন্নত হচ্ছে এবং কীভাবে;
   - **(গ)** কোন বৈশিষ্ট্যটি ক্ষতিগ্রস্ত বা চাপে পড়ছে এবং ঠিক কোন cause-and-effect প্রক্রিয়ায়;
   - **(ঘ)** এখানে **Internal Quality** ও **External Quality**-র মধ্যে কোনো দীর্ঘমেয়াদি প্রভাব কাজ করছে কি না।

---

## Closed-Book Mastery Assessment (Optional — Soft Target ~20–25 Minutes)

এই assessment-টি **ঐচ্ছিক (optional)**—Block 4 শুরু করার জন্য এটি সম্পন্ন করা বা ChatGPT-র অনুমোদন নেওয়া বাধ্যতামূলক নয়। আপনি চাইলে:
- এই lesson ও বাইরের সব tab বন্ধ করে নিচের পাঁচটি প্রশ্নের স্পষ্ট, মৌলিক ও cause-and-effect সমৃদ্ধ উত্তর `phases/phase-01/notes/block-03-notes.md` ফাইলে লিখে আনুষ্ঠানিক মূল্যায়নের জন্য **ChatGPT**-র কাছে শেয়ার করতে পারেন (`Mastery verified`-এর প্রমাণ হিসেবে);
- অথবা আপনি যথেষ্ট শিখেছেন মনে করলে assessment বাদ দিয়েই পরের block-এ এগিয়ে যেতে পারেন (`Studied — mastery not formally verified`);
- অথবা assessment না করেই যেকোনো অস্পষ্ট ধারণা নিয়ে প্রশ্ন ও আলোচনা করতে পারেন।

*(নোট: ~২০–২৫ মিনিট কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি আনুমানিক লক্ষ্যমাত্রা—কঠোর সময়সীমা নয়। পাঁচটি বিশ্লেষণাত্মক প্রশ্ন গভীরভাবে লিখতে এর চেয়ে বেশি সময় লাগতেই পারে।)*

1. **(LO 3.1 / Outcome 4)** Specification conformance, **preconditions**, **postconditions**, এবং **invariants**-এর সাহায্যে software correctness-এর সংজ্ঞা দিন। **Partial correctness** ও **total correctness**-এর পার্থক্য কী, এবং production backend-এ termination বা liveness কেন গুরুত্বপূর্ণ?
2. **(LO 3.2 / Outcome 4)** একটি নিজস্ব backend উদাহরণের মাধ্যমে **Fault → Error → Failure** শৃঙ্খল (dormant বনাম active fault এবং latent বনাম detected error-সহ) ব্যাখ্যা করুন। এরপর **fault prevention**, **fault removal**, **fault tolerance**, এবং **fault forecasting**-এর পার্থক্য দেখান—বিশেষত fault tolerance কেন fault prevention নয় তা স্পষ্ট করুন।
3. **(LO 3.3 / Outcome 7)** **Verification** এবং **Validation**-এর পার্থক্য কী? এমন দুটি বিপরীত backend পরিস্থিতি বলুন যেখানে একটি সিস্টেম Verification-এ পাস করে Validation-এ ফেল করে, আর আরেকটি আপাতদৃষ্টিতে Validation-এ পাস করেও Verification-এ ফেল করে।
4. **(LO 3.3 / Outcome 7)** Edsger W. Dijkstra-র *"testing can show the presence of bugs, never their absence"* বক্তব্যের যৌক্তিক কারণ ব্যাখ্যা করুন। এই সীমাবদ্ধতা থাকা সত্ত্বেও automated testing কেন অপরিহার্য, এবং এর পাশাপাশি আর কোন কোন structural উপায়ে engineer-রা correctness-এর ওপর আস্থা তৈরি করেন?
5. **(LO 3.4 / Outcome 4)** **ISO/IEC 25010:2023**-এর ৯টি শীর্ষ-স্তরের বৈশিষ্ট্য (২০২৩-এর হালনাগাদ নামসহ) উল্লেখ করুন। *"Clean code মানেই high-quality software"* ধারণাটি কেন অসম্পূর্ণ, **Internal Quality** ও **External Quality** কীভাবে একে অপরকে প্রভাবিত করে, এবং দুটি বাস্তব backend quality-attribute tension ব্যাখ্যা করুন।

### Block 3 Readiness & Progression Checklist
- [ ] Guided lesson গভীরভাবে পড়েছেন এবং Pause & Checks 1–3 সম্পন্ন করেছেন (পরে Self-Check Guidance মিলিয়ে দেখেছেন)।
- [ ] Complementary mandatory reading (**Avižienis et al. 2004** নির্বাচিত অংশ, **arc42 ISO/IEC 25010:2023**, এবং **Dijkstra EWD249 Section 3**) সম্পন্ন করেছেন।
- [ ] `phases/phase-01/notes/block-03-notes.md` ফাইলে স্বাধীনভাবে Exercise 1 (Fault → Error → Failure Trace) এবং Exercise 2 (Quality Attribute Tension Matrix) লিখেছেন।
- [ ] *(Optional)* বই/নোট বন্ধ রেখে Mastery Assessment Questions 1–5 লিখেছেন এবং আনুষ্ঠানিক মূল্যায়নের জন্য ChatGPT-র কাছে শেয়ার করেছেন।
- [ ] *(Optional)* কোনো ধারণা অস্পষ্ট মনে হলে assessment ছাড়াই সেগুলো নিয়ে ChatGPT-র সাথে আলোচনা করে পরিষ্কার হয়েছেন।

> **অগ্রগতির অবস্থা (Progression Status Distinction):** Block 4 কখন শুরু করবেন তা আপনার সিদ্ধান্ত। পড়া ও চর্চা শেষ করে assessment ছাড়া এগোলে এই block-টির অবস্থা হবে **Studied — mastery not formally verified**। যথেষ্ট প্রমাণ (exercise ও/অথবা assessment) শেয়ার করে মূল্যায়ন সম্পন্ন করলেই তা **Mastery verified** হিসেবে গণ্য হবে; assessment বাদ দিলে তা স্বয়ংক্রিয়ভাবে `Mastery verified` হয় না।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`notes/block-03-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের নির্দেশিকা দেখুন। Practical Reasoning Exercises বা Optional Closed-Book Mastery Assessment-এর কোনো উত্তর এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও (চাইলে) ঐচ্ছিক ChatGPT মূল্যায়নের জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* **Precondition:** `productId` বিদ্যমান, `quantity` একটি ধনাত্মক পূর্ণসংখ্যা (`quantity >= 1`), এবং পর্যাপ্ত `available_stock >= quantity` আছে। **Postcondition:** সফল হলে `available_stock` ঠিক `quantity` পরিমাণ কমে (`new_stock = old_stock - quantity`) এবং একটি reservation record তৈরি হয়; অপর্যাপ্ত মজুত থাকলে stock অপরিবর্তিত থাকে ও নির্ধারিত error (যেমন `OUT_OF_STOCK`) ফেরত আসে। **Invariant:** যেকোনো সময়ে `available_stock >= 0` (এবং `reserved_stock + available_stock = total_physical_stock`)। এই invariant ভেঙে `available_stock` ঋণাত্মক (`-3`) হয়ে গেলে warehouse fulfillment বা backorder সিস্টেম (যা অঋণাত্মক stock ধরে কাজ করে) অস্তিত্বহীন পণ্যের জন্য shipping label তৈরি করবে বা গ্রাহকের অর্ডার বাতিলের সংকট তৈরি করবে।
  2. *শক্তিশালী উত্তরে থাকবে:* একটি backend function তখনই *partially correct* কিন্তু *totally correct* নয়, যখন এটি শেষ হলে সঠিক উত্তর দেয়, কিন্তু নির্দিষ্ট কোনো input বা অবস্থায় কখনোই শেষ (terminate) হয় না। বাস্তব উদাহরণ: কোনো external API-র সাময়িক ব্যর্থতায় max-retry সীমা ছাড়া অসীম retry loop চালানো, কোনো paginated query-তে একই cursor ফিরে এলে না থামা, কিংবা category/org-chart hierarchy-তে চক্রাকার parent link (`A → B → A`) পড়ে visited check ছাড়া traversal-এ অনন্ত লুপে আটকে যাওয়া।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* **(ক)** SQL query-তে `WHERE user_id = $1` বাদ পড়া হলো কোডের ভেতরের **Fault**; query চালিয়ে অন্য user-এর invoice data-গুলো response object/memory-তে তুলে আনা হলো অবৈধ অভ্যন্তরীণ অবস্থা বা **Error**; আর সেই ব্যক্তিগত invoice অন্য একজন ব্যবহারকারীর কাছে HTTP response হয়ে পৌঁছে যাওয়া হলো specification-লঙ্ঘনকারী **Service Failure**। **(খ)** বাদ পড়া `WHERE` clause যোগ করে bug ঠিক করা হলো **Fault Removal** (কোডে থাকা ত্রুটি দূর করা), কখনোই **Fault Tolerance** নয়। আর একটি ভুল SQL query-র চারপাশে নিছক `try/catch` বসালে তা কোনো exception-ও তুলবে না (কারণ query-টি SQL syntax অনুযায়ী বৈধ ছিল!)—ফলে data leak আটকাবেই না।
  2. *শক্তিশালী উত্তরে থাকবে:* **Verification পাস, Validation ফেল উদাহরণ:** ধরুন একটি search API-র specification-এ কেবল exact-match শিরোনাম খোঁজার শর্ত দেওয়া ছিল এবং সেটি ১০০% test-সহ নির্ভুলভাবে বাস্তবায়ন করা হলো (Verification পাস)—কিন্তু বাস্তব shopper-রা সামান্য বানান ভুল বা আংশিক শব্দ দিয়ে খুঁজে কোনো পণ্যই পেলেন না, ফলে বিক্রি পড়ে গেল (Validation ফেল)। **১০০% line coverage ≠ bug-free:** Line coverage কেবল দেখায় প্রতিটি লাইন অন্তত একবার চলেছে কি না—কিন্তু কোনো লাইনের সব সম্ভাব্য input মান, সীমানা-অবস্থা (null/zero/negative/timezone), concurrency interleaving, বা missing logic (যে শর্ত কোডে লেখাই হয়নি!) তা পরীক্ষা করে না। Dijkstra-র যুক্তি অনুযায়ী finite test কেবল bug-এর উপস্থিতি প্রমাণ করতে পারে, অনুপস্থিতি নয়।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* দুর্বল **Internal Quality** (যেমন অস্পষ্ট ও জটপাকানো Maintainability — কম *modularity*, *analysability*, ও *testability*) সময়ের সাথে একাধিক External Quality-কে সরাসরি ক্ষতিগ্রস্ত করে: (ক) **Reliability:** production incident-এ root cause খুঁজে বের করতে ও নিরাপদে fix deploy করতে দীর্ঘ সময় লাগে, ফলে downtime বাড়ে (availability/recoverability কমে); (খ) **Security:** ছড়িয়ে-ছিটিয়ে থাকা duplication-এর কারণে কোনো একটি নতুন endpoint-এ authorization বা validation বাদ পড়ে গিয়ে data leak ঘটে; (গ) **Functional Suitability:** জটিল, test-অযোগ্য কোডে একটি ছোট feature যোগ করতে গিয়ে অন্য জায়গায় regression bug ঢুকে পড়ে।
  2. *শক্তিশালী উত্তরে থাকবে:* যেকোনো বাস্তব trade-off যুক্তিযুক্ত হতে পারে—যেমন (ক) **Safety বনাম Performance Efficiency / Interaction Capability:** financial payout backend-এ অস্বাভাবিক বড় বা দ্রুতগতির payout আটকাতে অতিরিক্ত synchronous risk-check ও manual approval threshold বসালে মারাত্মক আর্থিক ক্ষতির ঝুঁকি (**Safety**) কমে, কিন্তু লেনদেন সম্পন্ন হওয়ার গতি (**Performance Efficiency**) ও তাৎক্ষণিক ব্যবহারকারী অভিজ্ঞতা (**Interaction Capability**) কিছুটা কমে; অথবা (খ) **Flexibility বনাম Performance Efficiency:** বহু database বা cloud পরিবেশে সহজে চালানোর জন্য (**Flexibility**) generic abstraction স্তর রাখলে নির্দিষ্ট PostgreSQL-ভিত্তিক low-level tuning (**Performance Efficiency**) সীমিত হতে পারে।
