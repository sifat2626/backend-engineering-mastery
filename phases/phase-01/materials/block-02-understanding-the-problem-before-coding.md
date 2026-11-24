# Phase 1, Block 2 — Understanding the Problem Before Coding

আমাদের সেই বাড়তে থাকা online store-এর কথাই ধরুন—যেখানে Block 1-এ আপনি `SAVE10` promo-code service-টিকে সময়ের সাথে টিকিয়ে রাখার চ্যালেঞ্জ দেখেছিলেন। সোমবার সকালে Operations টিমের একজন ম্যানেজার আপনাকে খুব পরিষ্কার, ছোট্ট একটি ticket পাঠালেন:

> *"প্রতিদিন রাত ১২টায় চালানোর জন্য একটি nightly cron job তৈরি করুন, যা সেদিনের সব customer-কে তাদের payment receipt email করে পাঠাবে।"*

নির্দেশনা দেখে আপনার মনে হলো, এর চেয়ে পরিষ্কার কাজ আর কী হতে পারে! কোনো ধোঁয়াশা নেই, সরাসরি বলে দেওয়া আছে কী বানাতে হবে। আপনি দেরি না করে কোড লিখতে বসে গেলেন। দুই ঘণ্টার মধ্যে Node.js-এ একটি পরিচ্ছন্ন **cron job** *(নির্দিষ্ট সময় পরপর স্বয়ংক্রিয়ভাবে চলা একটি scheduled background task)* লিখে ফেললেন—যা প্রতিদিন মধ্যরাতে PostgreSQL database থেকে ওই দিনের paid order-গুলো তুলে আনে, প্রতিটি customer-এর ঠিকানায় একটি external email service দিয়ে receipt পাঠায়, আর শেষে `"Done"` লগ করে। তিনটি sample order দিয়ে test করে দেখলেন প্রতিটি email নিখুঁতভাবে পৌঁছাচ্ছে। দুপুরের পরপরই কাজ live করে দিলেন।

এবার পরদিন সকালের দৃশ্য দেখুন।

সকাল ১০টা ০৫ মিনিটে একজন কর্পোরেট customer অফিসের খরচের জন্য পণ্য কিনলেন। ১০টা ১৫ মিনিটে তিনি customer support-এ ফোন করে ক্ষুব্ধ হয়ে জানতে চাইলেন—টাকা কেটে নেওয়ার পরও তাঁর inbox-এ receipt এল না কেন, এখনই তাঁকে অফিসের expense portal-এ রসিদ জমা দিতে হবে! Support টিম অবাক হয়ে জানাল, *"রাত ১২টা পর্যন্ত অপেক্ষা করতে হবে!"*—যা শুনে customer অর্ডার বাতিল করার হুমকি দিলেন।

কিন্তু আসল বিপর্যয় ঘটল ওইদিন রাত ১২টা ০৩ মিনিটে। সেদিন store-এ ১,০০০টি order হয়েছিল। ৬০০টি email পাঠানোর পর external email provider-টি সাময়িকভাবে timeout হয়ে গেল, আর আপনার cron job-টি crash করল। স্বয়ংক্রিয় scheduler দশ মিনিট পর job-টি আবার চালু করল—এবং যেহেতু কোন order-এ ইতিমধ্যেই receipt পাঠানো হয়ে গেছে তা track করার কোনো ব্যবস্থা ছিল না, তাই প্রথম ৬০০ জন customer-এর কাছে রাত ১২টা ১৫ মিনিটে দ্বিতীয়বার receipt গেল। কেউ কেউ ভাবলেন তাঁদের কার্ড থেকে দুবার টাকা কেটে নেওয়া হয়েছে! এদিকে অন্য একটি batch-এ database-এ `email` field-টি ফাঁকা থাকা একটি guest order-এ এসে script-টি আবার থেমে যাওয়ায় বাকি ৪০০ জন customer কখনোই কোনো receipt পেলেন না।

একটু থেমে গভীরভাবে ভাবুন: **এখানে আপনার কোডে কি কোনো syntax ভুল ছিল?** না। আপনার script কি ticket-এর প্রতিটি শব্দ হুবহু পালন করেছে? শতভাগ। তবুও feature-টি production-এ প্রথম চব্বিশ ঘণ্টাতেই মুখ থুবড়ে পড়ল।

কেন? কারণ ব্যর্থতাটি ঘটেছিল প্রথম লাইন কোড লেখারও *আগে*।

Operations টিম আপনাকে আসল **সমস্যাটা** বলেনি—তারা নিজেরাই একটি **সমাধান** কল্পনা করে আপনার হাতে ধরিয়ে দিয়েছিল (*"একটি nightly cron job বানাও"*)। আর আপনি সেই সমাধানটিকে বিনা প্রশ্নে মেনে নিয়ে কোড লিখতে বসে পড়েছিলেন। আপনি যদি কোড লেখার আগে মাত্র পনেরো মিনিট থেমে জিজ্ঞাসা করতেন—*"Customer-দের ঠিক কখন receipt দরকার? Email provider ব্যর্থ হলে কী হবে? একই receipt দুবার গেলে কী সমস্যা?"*—তাহলে বেরিয়ে আসত যে আসল প্রয়োজনটা কখনোই "রাত ১২টার cron job" ছিল না। ব্যবসায়িক ভাষায় আসল লক্ষ্যটি ছিল:

> *কোনো customer-এর payment নিশ্চিত হওয়ার ৫ মিনিটের মধ্যে তাঁর কাছে payment receipt পৌঁছানোর ব্যবস্থা করা—কোনো receipt যেন নীরবে হারিয়ে না যায় এবং ভুল retry-র কারণে customer-দের কাছে অকারণে একাধিকবার একই receipt না পাঠানো হয়।*

এখানেই একটি আরও গভীর engineering সত্য ধরা পড়ে, যা দেখায় **শুনতে পরিষ্কার লাগা ব্যবসায়িক বাক্যের ভেতরেও কীভাবে মারাত্মক অস্পষ্টতা লুকিয়ে থাকে**। কেউ যদি ওপরের প্রয়োজনটিকে চট করে লিখে ফেলেন *"Payment নিশ্চিত হওয়ার ৫ মিনিটের মধ্যে customer-এর কাছে ঠিক একবারই email পৌঁছাতে হবে (exactly-once email delivery)"*, তবে সেটি শুনতে চমৎকার হলেও দুটি বড় বাস্তবতা গুলিয়ে ফেলে: প্রথমত, **customer-এর নিজের account-এ রসিদ সহজলভ্য হওয়া (customer-accessible receipt availability)** বনাম **email notification পাঠানো**—এ দুটি এক জিনিস নয়। ব্যবসা যদি ৫ মিনিটের মধ্যে receipt পাওয়ার শর্ত দেয়, তবে কোড লেখার আগেই স্পষ্ট করতে হবে—customer-এর account বা order history-তে ৫ মিনিটের মধ্যে রসিদ পাওয়া গেলেই কি সেই মূল শর্তটি পূরণ হয় (যেখানে email একটি সহায়ক notification), নাকি email-ই একমাত্র গ্রহণযোগ্য মাধ্যম?

দ্বিতীয়ত, একটি backend system-এর সীমানার ভেতরে ও বাইরে তিনটি ভিন্ন স্তর কাজ করে:

- **আমাদের backend-এ exactly-once business processing, receipt generation ও account availability:** প্রতিটি confirmed payment-এর জন্য ঠিক একটিমাত্র প্রামাণ্য receipt তৈরি করা এবং সেটি customer-এর নিজের account-এ দেখানো—এটি আমাদের system-এর সরাসরি নিয়ন্ত্রণে, তাই এটি আমরা নির্ভরযোগ্যভাবে নিশ্চিত করতে পারি।
- **External email provider-এর কাছে idempotent submission—এবং এর বাস্তব সীমা:** যেখানে email provider idempotency বা submission-status যাচাই (reconciliation) সমর্থন করে, সেখানে retry-র সময় একই শনাক্তকারী key পাঠিয়ে duplicate email এড়ানো যায়। কিন্তু **provider যদি idempotency বা status reconciliation সমর্থন না করে**, তবে কোনো অস্পষ্ট network timeout-এর পর retry করলে এখনো duplicate email চলে যেতে পারে। তাই specification-এ এই সীমাবদ্ধতাটি স্পষ্টভাবে স্বীকার করতে হবে এবং অস্পষ্ট timeout-এ কোন fallback বা ঝুঁকি গ্রহণযোগ্য (যেমন মাঝেমধ্যে duplicate email মেনে নেওয়া, নাকি retry সীমিত রেখে customer-কে account-এ রসিদ দেখানো ও alert তোলা)—তা stakeholder-দের সম্মতিতে ঠিক করতে হবে।
- **বাস্তব email inbox delivery:** External email provider, মাঝপথের mail server এবং গ্রাহকের নিজস্ব mail infrastructure আমাদের নিয়ন্ত্রণের বাইরে। তাই **provider কোনো submission গ্রহণ (accept) করলেই যে email-টি ঠিক একবার গ্রাহকের inbox-এ পৌঁছাবে—তার কোনো guarantee নেই**।

এই একটি ঘটনার ভেতরেই backend engineering-এর সবচেয়ে মৌলিক বিভাজনরেখাটি লুকিয়ে আছে: **Problem Space** বনাম **Solution Space**।

| জগৎ (Space) | এতে আসলে কী থাকে | মূল প্রশ্ন (Driving Question) | উদাহরণ (Receipt ঘটনা) |
| :--- | :--- | :--- | :--- |
| **Problem Space** | ব্যবহারকারী বা ব্যবসার প্রকৃত প্রয়োজন, domain-এর বাস্তব নিয়ম, সীমাবদ্ধতা (constraints), এবং ব্যর্থ হলে কী ক্ষতি হবে তার সংজ্ঞা। | *"কাদের জন্য কোন বাস্তব সমস্যা সমাধান করতে হবে, কোন কোন সীমাবদ্ধতার ভেতর, এবং কী ঘটলে সমাধানটি ব্যর্থ বলে গণ্য হবে?"* | Payment নিশ্চিত হওয়ার ৫ মিনিটের মধ্যে customer-এর account-এ ঠিক একটি প্রামাণ্য receipt সহজলভ্য করা, এবং stakeholder-সম্মত retry/fallback নীতি মেনে email notification পাঠানো (যেখানে provider acceptance মানেই inbox delivery-র guarantee নয়)—যাতে customer দ্রুত রসিদ পান ও support-চাপ না বাড়ে। |
| **Solution Space** | প্রোগ্রামিং ভাষা, framework, cron job, Redis, Kafka, database schema, API route, বা কোডের অভ্যন্তরীণ গঠন। | *"এই নির্দিষ্ট সমস্যা ও সীমাবদ্ধতার জন্য কোন কারিগরি উপায়, স্থাপত্য ও কোড সবচেয়ে উপযুক্ত?"* | Nightly cron job লেখা, অথবা payment event এলেই একটি retry-সক্ষম background worker দিয়ে email পাঠানো। |

সফটওয়্যার টিমগুলোতে প্রতিদিন যে ভুলটি সবচেয়ে বেশি ঘটে, তা হলো Problem Space পরিষ্কার হওয়ার আগেই সরাসরি Solution Space-এ ঝাঁপ দেওয়া—যেমন সমস্যাটি ভালো করে না বুঝেই বলা *"চলো Kafka বসাই"* *(বড় পরিসরে event-streaming-এর জন্য ব্যবহৃত একটি distributed messaging platform)*, *"Redis-এ cache করে দিই"* *(দ্রুতগতির একটি in-memory data store)*, বা *"একটা cron job লিখে ফেলি"*। কোনো নির্দিষ্ট tool বা mechanism বেছে নেওয়ার মুহূর্তেই আপনি অজান্তে তার নিজস্ব সীমাবদ্ধতা (যেমন nightly cron job-এর ২৪ ঘণ্টার বিলম্ব) আপনার পুরো সিস্টেমের ওপর চাপিয়ে দেন।

এখান থেকে দুটি অত্যন্ত পরিচিত অথচ ক্ষতিকর ভুল ধারণা ভেঙে ফেলা দরকার:

- **ভুল ধারণা ১: *"তাড়াতাড়ি কোড লেখা শুরু করলেই কাজ দ্রুত শেষ হয়।"*** — প্রথম দুই ঘণ্টায় দ্রুত কোড জমে উঠতে দেখে মনে হয় দারুণ অগ্রগতি হচ্ছে। কিন্তু অস্পষ্ট বা ভুল mental model-এর ওপর দাঁড়িয়ে দ্রুত কোড লিখলে আসলে কেবল **দ্রুত ভুল জিনিস তৈরি হয়**। এরপর শুরু হয় ব্যয়বহুল পুনর্লিখন (rework), live database-এ ঝুঁকিপূর্ণ **schema migration** *(চলমান database-এর table বা column কাঠামো পরিবর্তনের প্রক্রিয়া, যা আমরা Phase 14-এ গভীরে শিখব)*, এবং ভাঙা API contract মেরামত। Block 1-এর **Shifting Left** এবং **Transaction ও Opportunity Cost**-এর কথা মনে করুন: কোড লেখার আগে ২০ মিনিট প্রশ্ন করলে যে দিকবদল বিনা খরচে হতো, production-এ যাওয়ার পর সেই ভুল শোধরাতে কয়েক দিন ও customer-দের আস্থা—দুটোই নষ্ট হয়।
- **ভুল ধারণা ২: *"Requirements বোঝা তো Product Manager-এর কাজ; engineer-এর কাজ শুধু ticket দেখে কোড লেখা।"*** — Product Manager বা business stakeholder-রা ব্যবসায়িক লক্ষ্য, user-দের চাহিদা ও অগ্রাধিকার নির্ধারণ করেন; কিন্তু তাঁদের লেখা ticket-এ প্রায় কখনোই **concurrency** *(একই মুহূর্তে একাধিক request সমান্তরালভাবে একই data স্পর্শ করলে কী হবে—Phase 6 ও 10-এ বিস্তারিত)*, **idempotency** *(কোনো কাজ দুর্ঘটনাবশত দুবার চললেও ফলাফল যেন একবারের মতোই থাকে, যেমন একই payment-এর জন্য backend থেকে একাধিকবার receipt তৈরি বা duplicate send-request না যাওয়া—Phase 9 ও 17-এ বিস্তারিত)*, partial failure-এ retry নীতি, response time-এর সীমা, বা data retention-এর মতো কারিগরি শর্ত লেখা থাকে না। এগুলো খুঁজে বের করা, প্রশ্ন করা এবং স্পষ্ট করা কোনো আমলাতান্ত্রিক বাড়তি কাজ নয়—এটি backend engineer-এরই মূল দায়িত্ব।

এবার একটি স্বাভাবিক পাল্টা পরিস্থিতি কল্পনা করুন। ধরুন, আপনি সচেতনভাবে Solution Space থেকে বেরিয়ে এলেন। পরদিন Product টিম আপনাকে এমন একটি ticket দিল, যেখানে কোনো cron job, Redis বা database-এর নাম চাপিয়ে দেওয়া নেই—পুরোটাই user-এর প্রয়োজনের ভাষায় লেখা, দেখতে একদম পরিষ্কার। **তবুও সেই "পরিষ্কার" মনে হওয়া request-এর ভেতরে কীভাবে মারাত্মক ত্রুটি লুকিয়ে থাকতে পারে?**

---

## পরিষ্কার মনে হওয়া Request-এর ভেতরে লুকানো ফাটল

আমাদের promo-code service-টি আরও বড় হয়েছে। এবার Product টিম আপনাকে Problem Space-এর ভাষাতেই একটি নতুন ticket দিল:

> *"Checkout-এর সময় customer-রা যেন দ্রুত ও নির্ভরযোগ্যভাবে promo code ব্যবহার করতে পারেন, এবং প্রতিটি promo code একজন customer যেন একবারই ব্যবহার করতে পারেন।"*

লক্ষ করুন, এখানে কোনো নির্দিষ্ট tool বা implementation চাপিয়ে দেওয়া হয়নি। বাক্যটি পড়তে খুব সরল ও পরিষ্কার মনে হচ্ছে। এখন দেখুন, একটি টিমের চারজন সদস্য এই **হুবহু একই বাক্যটি** পড়ে কীভাবে চারটি সম্পূর্ণ ভিন্ন পৃথিবী কল্পনা করে ফেললেন:

- **Marketing Manager** *"একজন customer একবারই ব্যবহার করতে পারবেন"* পড়ে ভাবলেন: একটি সফল, সম্পূর্ণ হওয়া order-এ একবার। কোনো customer-এর card payment ব্যর্থ হলে তিনি তো নিশ্চয়ই আবার চেষ্টা করে কোডটি ব্যবহার করতে পারবেন!
- **Finance / Fraud Lead** একই কথা পড়ে ভাবলেন: শুধু এক account-এ নয়, একই phone number, credit card বা shipping address দিয়ে নতুন account খুললেও যেন একই মানুষ দুবার ছাড় নিতে না পারেন।
- **Mobile App Developer** *"দ্রুত"* পড়ে ধরে নিলেন: user `Apply` বাটনে ট্যাপ করলে ৩০০ মিলিসেকেন্ডের মধ্যেই উত্তর আসবে, আর কোডটি cart-এ বসানো মাত্রই সেটি সেই user-এর জন্য সংরক্ষিত (reserved) থাকবে।
- **Backend Engineer** কোড লিখতে গিয়ে ভাবলেন: checkout button চাপার সময় database-এ চেক করব এই `user_id` আগে কোডটি ব্যবহার করেছে কি না—না করলে ছাড় দেব। কিন্তু কোনো user যদি দুর্ঘটনাবশত দুটি browser tab থেকে একই মিলিসেকেন্ডে `Place Order` চাপেন (**race condition**—অর্থাৎ একই data-য় সমান্তরাল দুটি request-এর সময়গত সংঘাতে ভুল ফলাফল তৈরি হওয়া), অথবা অর্ডার করার ১০ মিনিট পর customer সেটি বাতিল করে refund নেন, তখন কোডটি ফেরত পাবেন কি না—তা নিয়ে কেউ একটি কথাও বলেননি।

খেয়াল করুন, মিটিং শেষে চারজনই মাথা নেড়ে বললেন, *"Ticket একদম পরিষ্কার, আমরা সবাই একমত!"*—অথচ বাস্তবে **একজনও আরেকজনের সাথে একমত ছিলেন না।**

Requirements engineering-এর প্রবীণ বিশেষজ্ঞ Karl Wiegers (Wiegers & Beatty, 2013; Wiegers & Hokanson, 2023) এই ভয়ংকর ফাঁদটিকে বুঝতে **Ambiguity (অস্পষ্টতা)**-কে দুটি ভিন্ন শ্রেণিতে ভাগ করেছেন:

1. **Reader-Side Ambiguity (Single-Reader Ambiguity):** যখন *একজন* পাঠক একটি বাক্য পড়ে নিজেই টের পান যে এর একাধিক অর্থ হতে পারে। যেমন ticket-এ লেখা থাকল *"১০% ছাড় দিতে হবে"*—আপনি পড়েই থেমে গেলেন: *"আচ্ছা, এই ১০% কি tax ও shipping যোগ করার আগে হিসাব হবে, নাকি পরে?"* এই অস্পষ্টতা বিরক্তিকর হলেও তুলনামূলক নিরাপদ, কারণ আপনার মাথার ভেতর সতর্কঘণ্টা বেজে উঠেছে এবং আপনি প্রশ্ন করে এটি ঠিক করে নিতে পারবেন।
2. **Stakeholder-Side Ambiguity (Multiple-Reader Ambiguity):** যখন *একাধিক* মানুষ একই বাক্য পড়ে সম্পূর্ণ ভিন্ন অর্থ বের করেন—অথচ **প্রত্যেকেই ১০০% নিশ্চিত থাকেন যে বাক্যটি একদম স্পষ্ট এবং তাঁর বোঝাটাই একমাত্র অর্থ।** এটিই requirements-এর সবচেয়ে প্রাণঘাতী ত্রুটি। কারণ এখানে কারও মাথাতেই সতর্কঘণ্টা বাজে না; এক ধরনের ভুয়া ঐকমত্যের (false consensus) ছায়ায় পুরো টিম ভিন্ন ভিন্ন দিকে কোড লিখতে থাকে, আর সংঘর্ষটা ধরা পড়ে কয়েক সপ্তাহ পর integration testing কিংবা live customer-দের অভিযোগের মাধ্যমে।

এই অস্পষ্টতার নিচে আরও তিনটি জিনিস সব সময় একে অন্যের সাথে মিশে থাকে। একজন দক্ষ backend engineer কোনো request হাতে নিলে সবার আগে এই তিনটি স্তরকে ছুরি দিয়ে আলাদা করেন:

- **Fact (প্রমাণিত তথ্য):** যা সরাসরি যাচাইকৃত ও সত্য—অনুমান নয়। যেমন: *"আমাদের PostgreSQL `orders` table-এ প্রতিটি order-এর সাথে একটি `user_id` এবং `status` আছে"* অথবা *"গত মাসে গড়ে দিনে ১২,০০০ order হয়েছে।"*
- **Assumption (অনুমান):** যা প্রমাণ বা নিশ্চিতকরণ ছাড়াই সত্য ধরে নেওয়া হয়েছে। যেমন: *"একটি order-এ customer কেবল একটি promo code-ই ব্যবহার করবেন"*, *"একজন ব্যক্তির একটিমাত্র account আছে"*, অথবা *"payment gateway সব সময় ২ সেকেন্ডের মধ্যে জবাব দেবে।"* যাচাই না করা প্রতিটি assumption আসলে কোডের ভেতরে পুঁতে রাখা একটি **latent failure trigger**—অর্থাৎ সুপ্ত বিস্ফোরক, যা বাস্তব পৃথিবী আপনার অনুমান ভাঙা মাত্রই ফেটে পড়ে।
- **Dependency (নির্ভরতা):** এমন কোনো বাহ্যিক ব্যবস্থা, টিম, service বা পূর্বশর্ত, যার সঠিক আচরণের ওপর আপনার কাজ নির্ভরশীল—কিন্তু যা সরাসরি আপনার নিয়ন্ত্রণে নেই। যেমন: *"Fraud team-এর দেওয়া device-fingerprint API-টি চালু থাকতে হবে"* অথবা *"Payment provider-কে অবশ্যই payment failure-এর webhook পাঠাতে হবে।"*

একটি assumption-কে ভুলে fact ভেবে নিলে আপনি তার কোনো সুরক্ষা লিখবেন না; আর একটি dependency-কে চিহ্নিত না করলে সেটি ব্যর্থ হলে আপনার পুরো service ভেঙে পড়বে।

এবার খেয়াল করুন, আমাদের promo-code ticket-এ আরেকটি বড় ভারসাম্যহীনতা ছিল। Ticket-টি বলেছে system-টি **কী করবে** (promo code প্রয়োগ করবে, একবারের বেশি ব্যবহার আটকাবে)—কিন্তু প্রায় কিছুই বলেনি system-টি বাস্তব চাপ ও ঝুঁকির মধ্যে **কতটা ভালোভাবে আচরণ করবে**। Engineering-এ এই দুটি দিককে স্পষ্টভাবে আলাদা করা হয়:

- **Functional Requirements (কার্যগত চাহিদা):** নির্দিষ্ট input ও অবস্থায় system-টির কী কাজ বা আচরণ করতে হবে। যেমন: মেয়াদোত্তীর্ণ promo code বাতিল করা, বৈধ কোডে ১০% ছাড় হিসাব করা, সফল redemption-এর পর record সংরক্ষণ করা।
- **Non-Functional Requirements (অকার্যগত বা গুণগত সীমাবদ্ধতা):** system-টিকে কোন গুণগত মান ও বাস্তব সীমাবদ্ধতার ভেতরে সেই কাজটি সম্পন্ন করতে হবে—যেমন response time-এর সীমা (ধরুন **p95 latency**—অর্থাৎ ৯৫% request যে সময়ের মধ্যে শেষ হতে হবে, যা গড় সময়ের চেয়ে বাস্তব ধীরগতি ভালোভাবে ধরে; বিস্তারিত Phase 20-এ), প্রতি সেকেন্ডে কতগুলো request সামলাতে হবে (**throughput** বা load), concurrency safety, **authentication** *(caller আসলেই কে তা যাচাই করা)* ও **authorization** *(সেই caller-এর এই কাজ বা data দেখার অনুমতি আছে কি না তা যাচাই করা—Phase 18-এ বিস্তারিত)*, ব্যর্থতা থেকে পুনরুদ্ধার, এবং audit logging।

বাস্তব backend টিমে Product ticket-গুলোতে প্রায়ই কেবল সুখের পথের (**happy path**) Functional Requirements লেখা থাকে। Non-Functional Requirements এবং **Edge Cases** *(সীমান্তবর্তী বা অস্বাভাবিক পরিস্থিতি)* থাকে অদৃশ্য। একটু ভেবে দেখুন, আমাদের সাধারণ promo-code feature-এই কতগুলো বাস্তব edge case লুকিয়ে আছে:
- Customer রাত ১১টা ৫৯ মিনিট ৫৮ সেকেন্ডে কোড বসালেন, আর `Place Order` চাপলেন ১২টা ০১ সেকেন্ডে (কোড expire হয়ে গেছে—কোন সময়টা ধরবেন? আর কোন timezone-এ?)।
- Order-এর মোট মূল্য ১০০ টাকা, আর flat ২০০ টাকার promo code বসানো হলো—order-এর মূল্য কি ঋণাত্মক (-১০০ টাকা) হয়ে যাবে, নাকি ০ টাকায় আটকে থাকবে?
- একই customer দুটি browser tab থেকে ঠিক একই মিলিসেকেন্ডে `Place Order` চাপলেন—দুটি concurrent request কি একসাথে `"আগে ব্যবহৃত হয়নি"` পড়ে দুবারই ছাড় নিয়ে নেবে?
- Customer কোডটি ব্যবহার করলেন, কিন্তু ১০ মিনিট পর card payment ব্যর্থ হলো বা পুরো order refund হলো—কোডটি কি আবার ব্যবহারযোগ্য হবে?
- একজন customer যদি অন্য কোনো user-এর ব্যক্তিগত referral promo code চুরি করে বসান, তবে কী হবে?

এখানে আরেকটি প্রশ্ন অত্যন্ত জরুরি: সব প্রশ্ন করে সব চাহিদা জড়ো করার পর কি আপনি *সবকিছুই* একসাথে বানিয়ে ফেলবেন? না! বরং সমস্যাকে পুরোপুরি বোঝার সমান গুরুত্বপূর্ণ আরেকটি অংশ হলো **Non-Goals (Explicit Scope Boundaries)** নির্ধারণ করা—অর্থাৎ আমরা **সচেতনভাবে কী কী সমাধান করব না**, তা স্পষ্টভাবে লিখে রাখা।

Malte Ubl (*Design Docs at Google*, 2020) এখানে একটি অসাধারণ সূক্ষ্ম পার্থক্য দেখিয়েছেন: *"System crash করবে না"* বা *"আমরা security দুর্বলতা রাখব না"*—এগুলো কোনো Non-Goal নয়, এগুলো অর্থহীন নেতিবাচক বাক্য। একটি **সত্যিকারের Non-Goal** হলো এমন একটি যুক্তিসঙ্গত feature বা দিক, যা কোনো reviewer বা stakeholder স্বাভাবিকভাবেই এই কাজের অংশ বলে ভাবতে পারতেন—কিন্তু সময়, জটিলতা বা অগ্রাধিকারের কারণে টিম সচেতনভাবে বর্তমান ধাপে সেটি বাদ দিচ্ছে। আমাদের promo-code কাজের জন্য বাস্তব Non-Goal-এর উদাহরণ হতে পারে:
- *একই order-এ একাধিক promo code একসাথে (stacking) ব্যবহারের সুবিধা এই ধাপে দেওয়া হবে না।*
- *Account ছাড়া guest checkout-এর জন্য per-user promo limit সমর্থন করা হবে না।*
- *Multi-device বা ভিন্ন account তৈরি করে fraud ঠেকানো (device fingerprinting) এই প্রকল্পের আওতায় নয়; তা পরবর্তী Fraud-prevention ধাপে দেখা হবে।*

Explicit Non-Goals না থাকলে প্রকল্পের মাঝপথে নীরবে নতুন কাজ ঢুকতে থাকে (**scope creep**), আর engineer-রা অজান্তেই এমন অবকাঠামো বানিয়ে ফেলেন যা আসলে কেউ চায়নি (**hidden architectural drift**)।

তাহলে বাস্তব কর্মক্ষেত্রে একটি অস্পষ্ট backend ticket হাতে এলে আপনি কোড লেখার আগে ঠিক কীভাবে সেটিকে ব্যবচ্ছেদ করবেন? মুখস্থ কোনো ফর্ম নয়, বরং নিচের **পাঁচটি লেন্সের জেরা-পদ্ধতি (Five-Lens Request Interrogation)** অনুশীলন করুন:

1. **Behavior & State Transitions:** স্বাভাবিক কাজের পাশাপাশি প্রতিটি অস্বাভাবিক অবস্থা, বাতিল/refund, পুনরাবৃত্তি (retry) ও সীমানা-মুহূর্তে (boundary conditions) system-এর ঠিক কী করা উচিত?
2. **Data Bounds, Formats & Ownership:** Input/output-এর আকার সর্বোচ্চ কত হতে পারে? ফাঁকা (null/empty), অজানা format, বড় সংখ্যা, currency rounding, timezone, বা পুরোনো data-র ক্ষেত্রে নিয়ম কী?
3. **Security, Identity & Trust Boundaries:** কে এই operation চালাতে পারবে? একজন user কি কোনোভাবে অন্য user-এর data দেখতে বা পরিবর্তন করতে পারবে? অসৎ ব্যবহারকারী বা bot এটি কীভাবে অপব্যবহার করতে পারে?
4. **Performance, Scale & Failure Semantics:** কতগুলো request একসাথে আসতে পারে? সর্বোচ্চ কত latency গ্রহণযোগ্য? কোনো downstream database বা external API ধীর বা অচল হয়ে গেলে system কি fail-fast করবে, queue করবে, নাকি আংশিক ফল দেবে? একই request দুবার এলে (idempotency) কী হবে?
5. **Assumptions, Dependencies & Non-Goals:** আমরা কোন জিনিসগুলো বিনা প্রমাণে সত্য ধরে নিচ্ছি? কোন বাইরের টিম বা service-এর ওপর নির্ভর করছি? আর কোন যুক্তিসঙ্গত কাজগুলো ইচ্ছাকৃতভাবে এখন করছি না?

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-02-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন stakeholder ticket খুললেন: *"Database load কমাতে আমাদের একটি nightly script চালাতে হবে যা ৯০ দিনের পুরোনো log delete করে দেবে।"* এখানে **Solution Space**-এ কী চাপিয়ে দেওয়া হয়েছে, এবং অন্তর্নিহিত **Problem Space** বুঝতে আপনি কোন দুটি প্রশ্ন সবার আগে করবেন?
> 2. একটি e-commerce টিমের ticket-এ লেখা আছে: *"Order বাতিল হলে customer-কে সাথে সাথে টাকা ফেরত দিতে হবে।"* এর ওপর ভিত্তি করে **Stakeholder-Side Ambiguity**-র একটি বাস্তব উদাহরণ দেখান (যেখানে দুইজন মানুষ একই বাক্য পড়ে দুটি ভিন্ন ও পরস্পরবিরোধী অর্থ ধরে নেন, অথচ দুজনেই ভাবেন বাক্যটা একদম পরিষ্কার)।
> 3. *"আমাদের checkout API-তে প্রতিটি request-এর সাথে বৈধ user token আসে, payment gateway ৯৯.৯% সময় ৫০০ ms-এর মধ্যে উত্তর দেয়, এবং কোনো customer দিনে ৫টির বেশি order করেন না"*—এই তিনটি বিবৃতির কোনটি সম্ভাব্য **Fact**, কোনটি **Assumption**, এবং কোনটি **Dependency**? (কোনোটি যাচাই ছাড়া Fact ধরে নেওয়ার ঝুঁকি কী?)

ধরুন, পাঁচটি লেন্স দিয়ে খুঁটিয়ে প্রশ্ন করে আপনি লুকানো অনুমান, অস্পষ্টতা, edge case এবং non-goal—সব বের করে আনলেন। এখন সেই সিদ্ধান্তগুলো পরিষ্কারভাবে লিখে রাখার পালা। কিন্তু আপনি যদি আপনার নথিতে লেখেন: *"The checkout service should validate promo codes quickly and securely, and normally prevent duplicate usage and send a receipt"*—তাহলে দুই সপ্তাহ পর আরেকজন engineer বা একটি automated test কীভাবে নিশ্চিতভাবে প্রমাণ করবে যে কোডটি শর্ত পূরণ করেছে কি না?

---

## অস্পষ্ট ইচ্ছাকে যাচাইযোগ্য Specification-এ রূপান্তর

ওপরের বাক্যটির দিকে আরেকবার নিবিড়ভাবে তাকান:

> *"The checkout service should validate promo codes quickly and securely, and normally prevent duplicate usage and send a receipt."*

ধরুন, আপনি QA বা review engineer হিসেবে আজ কোডটি যাচাই করতে বসেছেন। আপনার কোড ৯০০ মিলিসেকেন্ডে উত্তর দেয়—এটা কি *"quickly"*? একজন বলবেন *"১ সেকেন্ডের কম তো দ্রুতই!"*, আরেকজন বলবেন *"checkout-এ ৯০০ ms ভয়াবহ ধীর!"* আপনার কোডে password-এর বদলে শুধু token check করা আছে, কিন্তু rate limit নেই—এটা কি *"securely"*? *"normally prevent duplicate usage"* মানে কি—৯৯% সময় আটকাবে, নাকি ২% ক্ষেত্রে দুবার ছাড় দিলেও চলবে? আবার এই এক বাক্যের মধ্যে promo validation, duplicate prevention এবং receipt পাঠানো—তিনটি আলাদা কাজ একসাথে গিঁট বাঁধা! সবচেয়ে বড় গোলমাল: ছোট হাতের *"should"* দিয়ে লেখক কি বোঝাচ্ছেন এটি বাধ্যতামূলক, নাকি কেবল একটি ইচ্ছা?

যখন কোনো শর্তের বিরুদ্ধে একটি নিরপেক্ষ মানুষ বা automated test দাঁড় করিয়ে নিশ্চিতভাবে **Pass** অথবা **Fail** রায় দেওয়া যায় না, তখন সেই শর্তটি **unverifiable (অযাচাইযোগ্য)**। আর যে শর্তের পূরণ হওয়া বা না-হওয়া প্রমাণই করা যায় না, তা engineering specification নয়—তা নিছকই একটি অস্পষ্ট কামনা। কোনো কাজ শেষ হয়েছে কি না তা প্রমাণ করার জন্য আগে থেকে নির্ধারিত এই সুনির্দিষ্ট, পরীক্ষাযোগ্য শর্তগুলোকেই বলা হয় **Acceptance Criteria**।

একটি একক requirement বিবৃতি কখন "ভালোভাবে গঠিত" (**well-formed**) হয়, তার আন্তর্জাতিক মানদণ্ড নির্ধারণ করেছে **ISO/IEC/IEEE 29148:2018** *(Systems and software engineering — Life cycle processes — Requirements engineering; যা পুরোনো IEEE 830 মানদণ্ডকে প্রতিস্থাপন করেছে)*। এই মানদণ্ড অনুযায়ী একটি আদর্শ একক requirement-এর **নয়টি বৈশিষ্ট্য** থাকা উচিত। এর মধ্যে যে **পাঁচটি বৈশিষ্ট্য** প্রতিটি backend engineer-কে হাতে-কলমে প্রয়োগ করতে জানতে হয়, সেগুলো cause-and-effect সহ দেখুন:

1. **Unambiguous (দ্ব্যর্থহীন):** বাক্যটির কেবল একটিমাত্র অর্থই সম্ভব। অস্পষ্ট বিশেষণ ও ফাঁকফোকর—যেমন *fast, scalable, secure, user-friendly, normally, usually, support, as appropriate, etc.*—সম্পূর্ণ বাদ দিতে হবে। নাহলে Stakeholder-Side Ambiguity আবার ফিরে আসবে।
2. **Complete (সম্পূর্ণ):** সেই নির্দিষ্ট শর্তটি বাস্তবায়ন ও যাচাই করার জন্য প্রয়োজনীয় সব তথ্য, trigger condition এবং ব্যর্থতার আচরণ ওই বিবৃতিতেই থাকতে হবে। *"মেয়াদ শেষ হলে promo code বাতিল হবে"* অসম্পূর্ণ—কোথায় কোন timezone-এ মেয়াদ মাপা হবে এবং caller ঠিক কী error response পাবে, তা বাদ পড়ে গেছে।
3. **Singular (একক):** একটি বিবৃতি কেবল *একটিই* স্বতন্ত্র আচরণ বা সীমাবদ্ধতা প্রকাশ করবে—`and` বা `or` দিয়ে তিন-চারটি চাহিদা এক বাক্যে জোড়া লাগাবে না। কেন? কারণ তিনটি কাজ এক বাক্যে থাকলে, দুটো পাস করে একটি ফেল করলে সেই requirement-কে Pass বলবেন না Fail বলবেন, আর ভবিষ্যতে কোনো একটি বদলালে বাকিগুলোর tracking ভেঙে পড়ে।
4. **Feasible (বাস্তবায়নযোগ্য):** বিদ্যমান প্রযুক্তি, সময়, বাজেট, পদার্থবিদ্যার সীমা ও পরিচালন পরিবেশের ভেতরে এটি অর্জন করা সম্ভব হতে হবে। যেমন *"বিশ্বের যেকোনো প্রান্ত থেকে checkout API-এর উত্তর 10 ms-এর মধ্যে পৌঁছাবে এবং 100% uptime থাকবে"* শুনতে চমৎকার হলেও আলোর গতি ও নেটওয়ার্ক ব্যর্থতার বাস্তবতায় এটি **infeasible**।
5. **Verifiable (যাচাইযোগ্য):** যুক্তিসঙ্গত খরচে testing, measurement, demonstration, বা inspection-এর মাধ্যমে এটি পূরণ হয়েছে কি না তা বস্তুনিষ্ঠভাবে Pass/Fail প্রমাণ করা যাবে। প্রতিটি সংখ্যা, সীমা বা অবস্থা পরিমাপযোগ্য হতে হবে।

এর পাশাপাশি ISO/IEC/IEEE 29148:2018 আরও **চারটি বৈশিষ্ট্য** নির্ধারণ করে, যা পুরো ছবিটিকে সম্পূর্ণ করে: **Necessary** *(অপরিহার্য—বাদ দিলে system-এ সত্যিকারের ঘাটতি তৈরি হবে, অপ্রয়োজনীয় শৌখিনতা নয়)*, **Appropriate** *(সঠিক বিমূর্ততার স্তরে লেখা—প্রয়োজনীয় সীমাবদ্ধতা না হলে অকারণে নির্দিষ্ট vendor বা implementation চাপিয়ে না দেওয়া)*, **Correct** *(তথ্যগতভাবে ও stakeholder-এর আসল চাহিদার সাথে নির্ভুল)*, এবং **Conforming** *(টিম বা মানদণ্ডের নির্ধারিত কাঠামো ও রীতি অনুসরণকারী)*।

কিন্তু এখানে একটি সূক্ষ্ম ভাষাগত সমস্যা রয়েই যায়। ধরুন আপনি অস্পষ্ট শব্দ বাদ দিয়ে একটি একক ও যাচাইযোগ্য বাক্য লিখলেন—তবুও একটি requirement কি **কঠোরভাবে বাধ্যতামূলক**, নাকি **ব্যতিক্রমসাপেক্ষে সুপারিশকৃত**, নাকি **সম্পূর্ণ ঐচ্ছিক**, তা বোঝাবেন কীভাবে? সাধারণ ইংরেজিতে `must`, `should`, `can`, `may`, বা `will`—মানুষ এগুলোকে একেক সময় একেক অর্থে ব্যবহার করে।

ইন্টারনেটের মূল প্রটোকলগুলো তৈরি করে যে মানদণ্ড সংস্থা (**IETF** — *Internet Engineering Task Force*), তারা ১৯৯৭ সালেই এই ভাষাগত বিভ্রান্তির সমাধান করেছিল একটি ছোট কিন্তু কিংবদন্তিতুল্য নথির মাধ্যমে: **RFC 2119** *(S. Bradner, March 1997 — "Key words for use in RFCs to Indicate Requirement Levels")*। আর বিশ বছর পর ২০১৭ সালে একটি বড় বাস্তব অস্পষ্টতা দূর করতে সেটিকে হালনাগাদ করা হয় **RFC 8174** *(B. Leiba, May 2017 — "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words")*-এর মাধ্যমে। এই দুটি RFC-কে একত্রে বলা হয় **BCP 14** *(Best Current Practice 14)*।

আধুনিক backend specification, API contract এবং design doc-এ এই **normative keywords** কীভাবে ব্যাখ্যা করতে ও ব্যবহার করতে হয়, তা নিচের সারণিতে দেখুন:

| Normative Keyword (ALL CAPS) | সমার্থক (Aliases) | RFC 2119 অনুযায়ী নির্ভুল অর্থ | Backend উদাহরণ ও বাস্তব তাৎপর্য |
| :--- | :--- | :--- | :--- |
| **`MUST`** | `REQUIRED`, `SHALL` | **পরম বাধ্যবাধকতা** (absolute requirement)। এর ব্যতিক্রম মানেই specification ভঙ্গ হয়েছে। | *"The redemption endpoint `MUST` reject any promo code whose `expires_at` timestamp (UTC) is less than or equal to the request processing time (UTC)."* — একটিও expired code গৃহীত হলে তা সরাসরি bug। |
| **`MUST NOT`** | `SHALL NOT` | **পরম নিষেধাজ্ঞা** (absolute prohibition)। কোনো পরিস্থিতিতেই এটি করা যাবে না। | *"The API response `MUST NOT` include another user's redemption history or internal database error stack traces."* — কোনো অবস্থাতেই এটি অনুমোদিত নয়। |
| **`SHOULD`** | `RECOMMENDED` | **শক্তিশালী সুপারিশ।** কোনো বিশেষ ও যুক্তিসঙ্গত পরিস্থিতিতে এটি এড়িয়ে যাওয়া যেতে পারে, তবে ভিন্ন পথ বেছে নেওয়ার আগে তার **পূর্ণ পরিণতি** গভীরভাবে বুঝতে ও বিচার করতে হবে। | *"Error responses `SHOULD` include a correlation request ID header for debugging."* — কোনো হালকা health-check বা বিশেষ ক্ষেত্রে না থাকলেও interoperability ভাঙবে না, তবে বাদ দেওয়ার যুক্তি স্পষ্ট হতে হবে। |
| **`SHOULD NOT`** | `NOT RECOMMENDED` | **শক্তিশালী নিরুৎসাহন।** বিশেষ পরিস্থিতিতে এটি গ্রহণযোগ্য বা উপকারী হতে পারে, তবে তা বাস্তবায়ন করার আগে পূর্ণ পরিণতি ভালোভাবে যাচাই করতে হবে। | *"The checkout service `SHOULD NOT` block order confirmation solely because the asynchronous receipt-email provider is unreachable."* — বিশেষ আইনি বাধ্যবাধকতা ছাড়া email না গেলে পুরো বিক্রি আটকে দেওয়া উচিত নয়। |
| **`MAY`** | `OPTIONAL` | **সত্যিকারের ঐচ্ছিক।** কোনো implementation এটি রাখতে পারে বা বাদও দিতে পারে। তবে RFC 2119-এর একটি জরুরি interoperability শর্ত আছে: যে পক্ষ এটি দেয় না, তাকে দেওয়া পক্ষের সাথে কাজ চালাতে পারতে হবে—এবং দেওয়া পক্ষকেও না-দেওয়া পক্ষের সাথে মানিয়ে চলতে হবে। | *"Clients `MAY` pass the query parameter `include_discount_breakdown=true` on the cart-quote endpoint to request line-item discount details; servers `MUST` still return the standard cart total when the parameter is omitted, and clients `MUST` work cleanly if a server omits the optional breakdown."* |

এখন **RFC 8174 (2017)** কেন এত গুরুত্বপূর্ণ এবং এটি RFC 2119-এর সাথে একসাথে পড়া কেন বাধ্যতামূলক, তা খেয়াল করুন।

ধরুন, একটি API specification-এর ব্যাখ্যায় আপনি সাধারণ ইংরেজিতে লিখলেন: *"To avoid confusion, operators should review logs weekly, and the service MUST return HTTP 409 on duplicate redemption."* RFC 2119-এর আদি সংস্করণে বড় হাতের ও ছোট হাতের অক্ষরের পার্থক্য পুরোপুরি স্পষ্ট করা ছিল না—ফলে পাঠকেরা বিভ্রান্ত হতেন: ছোট হাতের `should` কি RFC 2119-এর আনুষ্ঠানিক `SHOULD`, নাকি সাধারণ ইংরেজি পরামর্শ? এছাড়া RFC 2119-এর শুরুর boilerplate অনুচ্ছেদে `NOT RECOMMENDED` শব্দযুগলটি বাদ পড়ে গিয়েছিল (যদিও Section 4-এ এর সংজ্ঞা ছিল)।

**RFC 8174** এই দ্বিধা চিরতরে দূর করে দুটি নিয়ম স্থির করে দিয়েছে:
1. **কেবলমাত্র সম্পূর্ণ বড় হাতের অক্ষরে (ALL CAPITALS)** লেখা হলেই `MUST`, `MUST NOT`, `REQUIRED`, `SHALL`, `SHALL NOT`, `SHOULD`, `SHOULD NOT`, `RECOMMENDED`, `NOT RECOMMENDED`, `MAY`, এবং `OPTIONAL` শব্দগুলো BCP 14-এর বিশেষ normative অর্থ বহন করবে।
2. **ছোট হাতের বা মিশ্র অক্ষরে** (`must`, `should`, `Should`, `may`, `optional`) লেখা হলে সেগুলো কেবল **সাধারণ ইংরেজি অভিধানের অর্থ** বহন করবে—কোনো আনুষ্ঠানিক specification শর্ত নয়।

তাই যেকোনো আনুষ্ঠানিক spec বা engineering RFC-র শুরুতে **BCP 14-এর প্রমিত ঘোষণা (boilerplate)** রেখে দেওয়া ভালো অভ্যাস:

> The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals, as shown here.

আর একটি অত্যন্ত সূক্ষ্ম কিন্তু অমূল্য নীতি লুকিয়ে আছে **RFC 2119-এর Section 6 (*Guidance in the use of these Imperatives*)**-এ, যা নতুন engineer-রা প্রায়ই এড়িয়ে যান। Section 6 পরিষ্কার সতর্কবার্তা দেয়: **এই imperative শব্দগুলো অত্যন্ত সংযম ও সতর্কতার সাথে ব্যবহার করতে হবে।** বিশেষত `MUST` বা `MUST NOT` কেবল সেখানেই ব্যবহার করা যাবে যেখানে **interoperability** নিশ্চিত করা বা **ক্ষতি ঠেকানো (limiting behavior with potential for causing harm)** সত্যিই জরুরি। কোনো নির্দিষ্ট অভ্যন্তরীণ implementation পদ্ধতি জোর করে চাপিয়ে দেওয়ার জন্য `MUST` ব্যবহার করা যাবে না!

একটি খারাপ বনাম ভালো উদাহরণে পার্থক্যটা দেখুন:
- **খারাপ (RFC 2119 Section 6 ও ISO 29148 `Appropriate` লঙ্ঘন):** *"The backend `MUST` store active promo codes in an in-memory JavaScript `Map` and check them using a `for` loop."* — এটি Problem Space ছেড়ে অকারণে অভ্যন্তরীণ কোড-কৌশল চাপিয়ে দিচ্ছে।
- **ভালো (আচরণ ও ফলাফল নির্দিষ্ট করে, বাস্তবায়নের স্বাধীনতা রাখে):** *"For a single-use promo code, the redemption service `MUST` reject any second or concurrent redemption attempt by the same `user_id` with structured error code `PROMO_ALREADY_REDEEMED`, ensuring at most one confirmed redemption per user."*

এবার দেখুন, আমাদের শুরুর সেই অস্পষ্ট, জড়ানো, অযাচাইযোগ্য বাক্যটি (*"The checkout service should validate promo codes quickly and securely, and normally prevent duplicate usage and send a receipt"*) ISO/IEC/IEEE 29148 এবং BCP 14 প্রয়োগ করে ভেঙে নিলে কীভাবে ছয়টি ঝকঝকে, একক (`Singular`), যাচাইযোগ্য Acceptance Criteria-তে রূপান্তরিত হয়:

1. **Expiry Validation (Functional, Singular, Complete, Verifiable):** *"When a customer submits a promo code whose `expires_at` timestamp (UTC) is less than or equal to the server's request-processing timestamp (UTC), the checkout API `MUST` reject the code with HTTP `422` and error code `PROMO_EXPIRED`, without applying any discount."* → **Verification:** মেয়াদোত্তীর্ণ ও মেয়াদ-সীমানার (exact boundary second) কোড দিয়ে automated integration test চালিয়ে HTTP `422`, `PROMO_EXPIRED`, এবং unchanged total যাচাই করা।
2. **Duplicate Prevention Under Concurrency (Functional + Invariant, Singular, Verifiable):** *"For a promo code configured with `max_uses_per_user = 1`, the checkout service `MUST NOT` confirm more than one redemption for the same `user_id`, even when multiple checkout requests for that user execute concurrently."* → **Verification:** একই `user_id` ও single-use কোড দিয়ে সমান্তরাল (concurrent) request পাঠিয়ে নিশ্চিত করা যে database-এ ঠিক ১টি confirmed redemption তৈরি হয় এবং বাকি request-গুলো প্রত্যাখ্যাত হয়।
3. **Latency Bound (Non-Functional, Feasible, Singular, Verifiable):** *"Under a sustained load of up to 100 checkout requests per second against a catalog of 100,000 promo codes, the promo-code validation step `MUST` complete within 150 ms at the 95th percentile (`p95`)."* → **Verification:** নির্ধারিত dataset ও 100 RPS load test চালিয়ে পরিমাপ করা `p95` latency ≤ 150 ms কি না।
4. **Order Isolation from Email Outages (Failure Semantics, Singular, Verifiable):** *"If the external email provider returns an error or times out after payment confirmation, the checkout API `SHOULD NOT` fail the customer's confirmed order response."* → **Verification:** Test পরিবেশে email provider-কে simulated timeout/failure দিয়ে order confirmation সফল হওয়া যাচাই করা।
5. **User-Visible Receipt Availability (Functional + Measurable Outcome, Singular, Verifiable):** *"Within 5 minutes of payment confirmation, the system `MUST` make a single authoritative receipt for that order accessible to the authenticated customer through their account's order-receipt endpoint, independently of email notification status."* → **Verification:** External email provider অচল থাকা অবস্থাতেই confirmed payment-এর পর ৫ মিনিটের মধ্যে সেই authenticated customer-এর account order-receipt endpoint থেকে ঠিক একটি বৈধ receipt পাওয়া যাচ্ছে কি না তা automated integration test দিয়ে যাচাই করা।
6. **Durable Notification Recovery & Escalation (Failure Recovery, Singular, Verifiable):** *"If initial email-notification submission for a confirmed payment fails or is interrupted by a service restart, the notification workflow `MUST` recover and retry submission—using the provider's idempotency or status-reconciliation support where available, or applying the stakeholder-agreed fallback policy for ambiguous timeouts—until the provider accepts the submission or an operational alert is triggered within 5 minutes."* → **Verification:** Simulated provider outage ও মাঝপথে service restart ঘটিয়ে যাচাই করা যে pending notification হারিয়ে যায় না এবং provider সচল হলে submission accepted হয় (যা কেবল provider-এর গ্রহণযোগ্যতা প্রমাণ করে, inbox delivery-র guarantee নয়) অথবা ৫ মিনিটের মধ্যে alert ওঠে—পাশাপাশি provider idempotency না থাকলে ambiguous timeout-এ stakeholder-সম্মত fallback নীতি মানা হয়।

লক্ষ করুন, আগের একটি ধোঁয়াটে বাক্য থেকে বেরিয়ে এসে এখন প্রতিটি শর্তে একজন developer জানেন ঠিক কী বানাতে হবে, একজন QA বা reviewer জানেন ঠিক কীভাবে Pass/Fail পরীক্ষা করতে হবে, আর কোনো implementation detail (যেমন Redis বা cron job) অকারণে চাপিয়ে দেওয়া হয়নি।

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-02-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন engineer একটি spec-এ লিখলেন: *"The login service should lock an account after failed attempts, and it MUST use a PostgreSQL `SELECT ... FOR UPDATE` query to track attempts."* এখানে **RFC 8174**, **RFC 2119 Section 6**, এবং **ISO/IEC/IEEE 29148** (বিশেষত `Unambiguous`, `Complete`, `Singular`, `Appropriate`, `Verifiable`) অনুযায়ী মোট কী কী ত্রুটি আছে?
> 2. ওপরের বিবৃতিটিকে ভেঙে কেবল একটি অংশ—**একাধিক ব্যর্থ login চেষ্টার পর account সাময়িক লক করার নিয়ম**—নিয়ে একটি একক, সম্পূর্ণ ও যাচাইযোগ্য BCP 14 acceptance criterion লিখুন, এবং এক লাইনে বলুন এটি কীভাবে verify করবেন।

একটি ছোট কাজের শর্তগুলো আপনি এখন নিখুঁতভাবে যাচাইযোগ্য ভাষায় লিখতে পারেন। কিন্তু বাস্তব কোম্পানিতে যখন কোনো কাজ একাধিক service, database schema, security boundary বা একাধিক টিমকে স্পর্শ করে—তখন এই পটভূমি, অস্পষ্টতা, সীমাবদ্ধতা, non-goal আর বিকল্প চিন্তাগুলো আপনি কেবল chat message বা এক লাইনের ticket-এ ছড়িয়ে রাখলে চলবে না। **তাহলে কোড লেখার আগেই গোটা টিমকে এক জায়গায় এনে কীভাবে হালকা অথচ কার্যকরভাবে সমস্যাটাকে ফ্রেম করবেন—আবার ছোটখাটো কাজে ভারী কাগজপত্রের আমলাতন্ত্রও তৈরি করবেন না?**

---

## কোড লেখার আগে হালকা লিখিত সমস্যা-কাঠামো (Design Docs)

একটি মজার বাস্তব সত্য হলো: মাথার ভেতর কোনো চিন্তা পরিষ্কার মনে হওয়া আর সেটিকে কাগজে লিখে অন্য তিনজন engineer-কে বোঝানো—দুটির মধ্যে আকাশ-পাতাল তফাত। লেখার সময়ই হঠাৎ ধরা পড়ে যে *"আরে, refund হলে promo code ফেরত যাবে কি না, সেটা তো ভেবেই দেখিনি!"* বা *"checkout টিম আর finance টিম 'confirmed order' বলতে দুটি আলাদা অবস্থা বোঝাচ্ছে!"*

এ কারণেই Google-সহ বহু পরিপক্ব engineering সংগঠন কোনো মাঝারি বা বড় পরিবর্তনে কোড লেখার আগেই একটি সংক্ষিপ্ত, অনানুষ্ঠানিক নথি লেখে, যাকে বলা হয় **Design Doc**। Malte Ubl তাঁর বহুল পঠিত ও বাস্তব-অভিজ্ঞতাভিত্তিক প্রবন্ধ ***"Design Docs at Google"* (2020)**-এ পরিষ্কারভাবে দেখিয়েছেন, একটি ভালো Design Doc কোনো শত পৃষ্ঠার প্রাচীন waterfall বই নয়; এটি সাধারণত ১ থেকে ৩ পৃষ্ঠার (খুব বড় সিস্টেম হলে কিছুটা বেশি) একটি কার্যকর engineering চিন্তা-যন্ত্র, যার আসল লক্ষ্য হলো **যখন দিক বদলানো সস্তা, ঠিক তখনই ভুল অনুমান ও অস্পষ্টতা ধরে ফেলা** (Block 1-এর **Shifting Left**-এর সরাসরি প্রয়োগ!)।

একটি Google-ধাঁচের Design Doc-এর সবচেয়ে প্রভাবশালী অংশগুলো কীভাবে সমস্যা বোঝার কাজটি সহজ করে, তা ধাপে ধাপে দেখুন:

1. **Context and Scope:** নথির শুরুতেই অত্যন্ত সংক্ষিপ্ত ও বস্তুনিষ্ঠ পটভূমি (**objective background facts**)। এখন system-টি কীভাবে চলছে, কোন সমস্যাটি আমরা সমাধান করতে নামছি, এবং এই পরিবর্তন ঠিক কোন সীমার মধ্যে কাজ করবে—তা অল্প কথায় গুছিয়ে বলা হয়। Ubl বিশেষভাবে মনে করিয়ে দেন: এটি কোনো ভারী formal requirements বই নয়; পাঠকের মৌলিক ধারণা আছে ধরে নিয়ে গভীর তথ্যের জন্য link দেওয়া হয়, যাতে অন্য টিমের একজন engineer ৫ মিনিটেই পুরো প্রেক্ষাপট ধরে ফেলতে পারেন।
2. **Goals and Non-Goals:** সংক্ষিপ্ত বুলেট তালিকা। **Goals** সেই স্পষ্ট মানদণ্ড দাঁড় করায়, যার বিপরীতে পুরো নকশাটি মূল্যায়ন করা হবে। আর **Non-Goals** হলো সেই সচেতন সীমানা—এমন সব যুক্তিসঙ্গত feature যা কেউ সহজেই এই প্রকল্পের অংশ ভাবতে পারত, কিন্তু আমরা ইচ্ছাকৃতভাবে বাদ দিচ্ছি। Review মিটিংয়ে যখন কেউ বলেন *"আচ্ছা, এর সাথেই কি multi-currency promo support দিয়ে দেব?"*, তখন Non-Goals-এর দিকে আঙুল তুলে দুই সেকেন্ডেই আলোচনাকে পথে ফেরানো যায়।
3. **Overview, Detailed Design & Cross-Cutting Concerns:** লক্ষ্য ও সীমা স্থির হওয়ার *পরেই* আসে সমাধানের রূপরেখা, গুরুত্বপূর্ণ trade-off ও বাতিল করা বিকল্প (alternatives considered—যা আমরা Block 4-এ ADR-এর সাথে গভীরে শিখব), এবং security, privacy, observability ও failure handling-এর মতো cross-cutting বিষয়গুলো। লক্ষ করুন ক্রমটা: **প্রথমে Problem Space (Context, Scope, Goals, Non-Goals), তার পরেই Solution Space!**

আমাদের বাড়তে থাকা promo-code ও refund reconciliation ফিচারের জন্য একটি সংক্ষিপ্ত Design Doc-এর প্রথম আধা পৃষ্ঠা কেমন হতে পারে, তার একটি বাস্তব নমুনা দেখুন:

```markdown
## Context and Scope
Our online store processes ~12,000 orders/day via the Node.js checkout service
and PostgreSQL. Marketing wants single-use promo codes to prevent repeat
redemption by the same account, while keeping checkout fast and handling
cancelled/refunded orders consistently. Scope covers redemption validation,
concurrency-safe usage tracking, and full-order cancellation reversal inside
the checkout service.

## Goals
- Enforce `max_uses_per_user` accurately under concurrent checkout requests.
- Return deterministic, structured error codes (`PROMO_EXPIRED`,
  `PROMO_ALREADY_REDEEMED`) within `p95 <= 150 ms` at 100 RPS.
- Restore a single-use code automatically if payment authorization fails
  before order confirmation.

## Non-Goals
- Stacking multiple promo codes on a single order.
- Detecting multi-account abuse across different user IDs (owned by Fraud
  team in Q3).
- Building a self-serve Marketing admin UI for creating promo campaigns.
- Restoring promo codes on partial item refunds (only full cancellations
  before fulfillment are in scope).
```

ভেবে দেখুন: এই ১৫ লাইন লিখতে সময় লাগে ২০ মিনিট। কিন্তু এটি Checkout, Mobile, Finance এবং Fraud—চারটি টিমকে এক ছাতার নিচে এনে চারটি বড় ভুল বোঝাবুঝি (multi-account fraud, partial refund, stacking, admin UI) কোড লেখার আগেই মিটিয়ে ফেলল!

এবার দেখুন একটি Design Doc সময়ের সাথে কীভাবে বাঁচে—Ubl একে **Lifecycle of a Design Doc** হিসেবে চারটি ধাপে বর্ণনা করেছেন:

1. **Creation & Rapid Iteration:** শুরুতে লেখক নথিটিকে ইচ্ছাকৃতভাবে হালকা ও খসড়া রাখেন (whiteboard, bullet points, সহজ খসড়া)। কেন? কারণ প্রথম দিকে বড় পরিবর্তন অনিবার্য; আপনি যদি শুরুতেই ২০ পৃষ্ঠার নিখুঁত গদ্য লিখে ফেলেন, তবে মনস্তাত্ত্বিকভাবে নিজের লেখার প্রতি মায়া পড়ে যাবে এবং ভুল ধরলে দিক বদলাতে অনীহা তৈরি হবে।
2. **Review (Multiple Rounds):** খসড়াটি সংশ্লিষ্ট সহকর্মী, সিনিয়র engineer, এবং প্রয়োজনমতো security/privacy বা partner টিমের সাথে শেয়ার করা হয়। এই ধাপের মূল লাভ হলো **সস্তায় ভুল ধরা**, গুরুত্বপূর্ণ মতবিরোধগুলো কোড লেখার আগেই নিষ্পত্তি করা, এবং এগিয়ে যাওয়ার মতো পর্যাপ্ত বোঝাপড়া ও সম্মতি (**sufficient agreement**—অবশ্যই শতভাগ সর্বসম্মত **unanimous consensus** নয়) তৈরি করা।
3. **Implementation & Iteration:** গুরুত্বপূর্ণ প্রশ্ন ও মতবিরোধ মিটে পর্যাপ্ত সম্মতি তৈরি হলে কোড লেখা শুরু হয়। বাস্তবায়নের সময় অপ্রত্যাশিত কোনো কারিগরি সীমাবদ্ধতা ধরা পড়লে design doc-টিও সেই অনুযায়ী হালনাগাদ করা হয় যাতে তা বাস্তবতার সাথে মেলে।
4. **Maintenance & Organizational Memory:** কাজ শেষ হয়ে যাওয়ার পরও নথিটি মূল্যহীন হয়ে যায় না। এক বছর পর যখন নতুন একজন engineer codebase খুলে ভাববেন *"এরা partial refund-এ promo code ফেরত দেয় না কেন—এটা কি bug নাকি ইচ্ছাকৃত?"*, তখন এই Design Doc-ই উত্তর দিয়ে দেবে।

সবশেষে আসে সবচেয়ে পরিণত engineering প্রশ্ন—যা সরাসরি Block 1-এর **proportional rigor** নীতির সাথে যুক্ত: **কখন Design Doc লেখা উচিত নয়?**

Design Doc লেখা বিনা খরচে হয় না; এতে লেখক ও reviewer-দের **personnel** এবং **opportunity cost** ব্যয় হয়। Malte Ubl-এর মূল মানদণ্ডটি একটি শব্দে বাঁধা: **Ambiguity (অস্পষ্টতা)**—যা আসতে পারে সমস্যার জটিলতা থেকে, সমাধানের জটিলতা থেকে, অথবা একাধিক টিম ও ঝুঁকিপূর্ণ trade-off-এর সমন্বয় থেকে।
- **যখন Design Doc বাদ দেওয়া উচিত:** যখন সমস্যা ও সমাধান দুটোই স্পষ্ট, কোনো তাৎপর্যপূর্ণ বিকল্প বা trade-off নেই, blast radius ছোট, এবং পরিবর্তনটি সহজেই উল্টে দেওয়া যায় (যেমন একটি validation error-এর বানান ঠিক করা, বিদ্যমান প্যাটার্ন মেনে একটি ছোট internal helper যোগ করা, বা এক ফাইলের পরিষ্কার bug fix)। এখানে Design Doc লিখতে বসলে শুধু সময় নষ্ট হবে এবং শেখা ও delivery ধীর হয়ে যাবে।
- **একটি নির্ভুল সতর্কসংকেত (Red Flag):** লিখতে গিয়ে যদি দেখেন আপনার Design Doc-এ কোনো problem framing, ambiguity বা trade-off আলোচনা নেই—বরং তা ধাপে ধাপে **implementation manual**-এর মতো শোনাচ্ছে (*"প্রথমে `promoValidator.js` ফাইল খুলুন, তারপর `checkExpiry()` function যোগ করুন, তারপর `if` শর্ত বসান"*)—তাহলে থেমে যান। এটি স্পষ্ট সংকেত যে হয় এই কাজটির জন্য Design Doc-এর দরকারই ছিল না, নয়তো আপনি Problem Space ও architectural decision বাদ দিয়ে কোডকে গদ্যে অনুবাদ করছেন।

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-02-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. নিচের দুটি কাজের কোনটিতে সংক্ষিপ্ত Design Doc লেখা যুক্তিসঙ্গত, আর কোনটিতে তা অপ্রয়োজনীয় overhead? Malte Ubl-এর *ambiguity* ও trade-off মানদণ্ড ব্যবহার করে ব্যাখ্যা করুন:
>    - **(ক)** Checkout-এর একটি error message-এ `"Code invalid"`-এর বদলে structured `"PROMO_INVALID"` code ও timestamp লগ যোগ করা (যা ইতিমধ্যেই টিমের প্রমিত error format-এর অংশ)।
>    - **(খ)** Checkout, Inventory এবং Billing—তিনটি টিমের অংশগ্রহণে order বাতিল হলে স্বয়ংক্রিয়ভাবে promo code ও stock ফেরত দেওয়ার নতুন workflow তৈরি করা।
> 2. একটি Design Doc-এ কেউ Non-Goal হিসেবে লিখলেন: *"System-এ কোনো security bug থাকবে না এবং এটি ধীর হবে না।"* এটি ভালো Non-Goal না হওয়ার কারণ কী?

---

## AI যুগে Problem Framing কেন আরও নির্ণায়ক—এবং Block 3-এর দিকে যাত্রা

এক মুহূর্তের জন্য আধুনিক **AI coding assistants**-এর কথা ভাবুন। আপনি যদি কোনো AI agent-কে আমাদের শুরুর সেই অস্পষ্ট বাক্যটি দেন—*"Build a checkout promo-code feature fast and securely, one use per customer"*—তাহলে সেটি দশ সেকেন্ডের মধ্যে ২০০ লাইন ঝকঝকে Node.js আর SQL কোড লিখে দেবে। কিন্তু সে কখনোই থেমে আপনার সাথে Marketing বনাম Finance-এর দ্বন্দ্ব মেটাবে না; বরং সে নীরবে নিজের মতো করে একগাদা **hidden assumptions** (যেমন concurrency বাদ দেওয়া, local timezone ধরে নেওয়া, refund উপেক্ষা করা) কোডের ভেতরে গেঁথে দেবে! AI **Solution Space**-এ কোড তৈরির গতি বহুগুণ বাড়িয়ে দিয়েছে—ফলে **Problem Space** পরিষ্কারভাবে সংজ্ঞায়িত করা, অস্পষ্টতা দূর করা এবং যাচাইযোগ্য BCP 14 শর্ত লেখা এখন একজন backend engineer-এর সবচেয়ে দুর্লভ ও মূল্যবান দক্ষতা (Block 7-এ আমরা দেখব কীভাবে এই স্পষ্ট spec ও non-goal-গুলোকেই AI-এর জন্য **Context Engineering**-এ রূপান্তর করা হয়)।

এখন একটু সামনে তাকান। ধরুন, Block 2-এর প্রতিটি দক্ষতা প্রয়োগ করে আপনি সমস্যাটি নিখুঁতভাবে বুঝেছেন, লুকানো অনুমান ও অস্পষ্টতা ছেঁটে ফেলেছেন, Non-Goals ঠিক করেছেন, এবং BCP 14-এ যাচাইযোগ্য specification লিখে ফেলেছেন। এবার যখন বাস্তবে কোড লেখা হবে, তখন একটি গভীর প্রশ্ন সামনে এসে দাঁড়াবে: **আমরা কীভাবে প্রমাণ করব যে তৈরি হওয়া system-টি আসলেই "সঠিক" (correct) এবং এর "গুণমান" (quality) ভালো? সব unit test সবুজ দেখালেই কি correctness প্রমাণ হয়—নাকি তার গভীরে invariant, fault-error-failure শৃঙ্খল ও বহুমাত্রিক quality attribute কাজ করে?**

ঠিক এই প্রশ্ন নিয়েই শুরু হবে **Block 3: Correctness and Software Quality**।

---

## Study Plan, Learning Objectives & Mandatory Reading

> **Continuity & Ownership Notes:** Phase 1 Diagnostic ইতিমধ্যেই `completed`—এটি পুনরাবৃত্তি করবেন না। আপনার সব inline check, practical exercise এবং (আপনি যদি করতে চান) closed-book assessment-এর উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-02-notes.md` ফাইলে লিখুন। AI-এর প্রস্তুতির সময় আপনার study hours-এ গণনা হয় না।
>
> **Progression & Verification Policy:** এই প্রোগ্রামে আনুষ্ঠানিক assessment **ঐচ্ছিক (optional)** এবং পরের block-এ এগোনোর নিয়ন্ত্রণ আপনার হাতে। আপনি চাইলে assessment সম্পন্ন করে মূল্যায়নের জন্য ChatGPT-র কাছে শেয়ার করতে পারেন, নিজে যথেষ্ট শিখেছেন মনে করলে assessment বাদ দিয়ে এগিয়ে যেতে পারেন, অথবা assessment না করেই যেকোনো অস্পষ্ট ধারণা নিয়ে প্রশ্ন ও আলোচনা করতে পারেন। কোনো গুরুত্বপূর্ণ ফাঁক থাকলে ChatGPT বাড়তি যাচাইয়ের পরামর্শ দিতে পারেন, তবে assessment সম্পন্ন করা বা ChatGPT-র অনুমোদন পরের block শুরু করার বাধ্যতামূলক শর্ত নয়। তবে দুটি অবস্থার পার্থক্য সব সময় স্পষ্ট রাখুন: পড়া ও চর্চা শেষ করে assessment ছাড়া এগোলে সেটি হলো **Studied — mastery not formally verified**, আর যথেষ্ট প্রমাণ দিয়ে মূল্যায়ন সম্পন্ন করাই **Mastery verified** (assessment বাদ দিলে তা স্বয়ংক্রিয়ভাবে `Mastery verified` বলে গণ্য হয় না)।

### Target 3-Hour (180-Minute) Study Budget

| Stage | কাজ (Activity) | নির্ধারিত সময় (Target Time) |
| :---: | :--- | :---: |
| **1** | ওপরের guided lesson পড়া + `notes/block-02-notes.md`-এ Pause & Checks 1–3 সম্পন্ন করা | 65 min |
| **2** | Complementary Mandatory Reading (RFC 2119 + RFC 8174 + *Design Docs at Google*) | 35 min |
| **3** | `notes/block-02-notes.md`-এ স্বাধীনভাবে Practical Reasoning Exercises 1–2 সম্পন্ন করা | 55 min |
| **4** | *(Optional)* বই/নোট বন্ধ রেখে Closed-Book Mastery Assessment Questions 1–5 লেখা (চাইলে মূল্যায়নের জন্য ChatGPT-র কাছে শেয়ার করা) | 25 min *(target)* |
| **Total** | — | **180 min (3 h)** |

*(সময়-নির্দেশিকা: Stage 4-এর ২৫ মিনিট হলো যাঁরা ঐচ্ছিক assessment-টি করতে চান তাঁদের জন্য একটি আনুমানিক লক্ষ্যমাত্রা—কোনো কঠোর সময়সীমা বা পরের block-এ যাওয়ার বাধ্যতামূলক শর্ত নয়। পাঁচটি বিশ্লেষণাত্মক প্রশ্ন গভীরভাবে লিখতে ২৫ মিনিটের বেশি লাগতেই পারে; সময় বাঁচাতে গিয়ে উত্তরের গভীরতা কমাবেন না।)*

### Block 2 Learning Objectives *(Primary: Outcomes 2, 3; Supporting: Outcome 5)*

- **LO 2.1 (`Working Understanding`):** **Problem space** (ব্যবহারকারী/ব্যবসার প্রয়োজন, domain সীমাবদ্ধতা, ব্যর্থতার পরিণতি) এবং **solution space** (tools, frameworks, schemas, code structures)-এর মধ্যে স্পষ্ট পার্থক্য নির্ণয় করা।
- **LO 2.2 (`Practical Competence`):** কোড লেখার আগেই একটি অস্পষ্ট বা অসম্পূর্ণ engineering request-কে ভেঙে **hidden assumptions**, **ambiguities**, **missing requirements**, **edge cases**, এবং **non-goals** উন্মোচন করা।
- **LO 2.3 (`Working Understanding`):** Well-formed requirements-এর প্রমিত মানদণ্ড (বিশেষত **unambiguous**, **complete**, **singular**, **feasible**, এবং **verifiable**) এবং normative specification ভাষা (`MUST`, `SHOULD`, `MAY` ইত্যাদি — RFC 2119 ও RFC 8174 অনুযায়ী) প্রয়োগ করা।
- **LO 2.4 (`Working Understanding`):** কোড লেখা শুরু করার আগেই হালকা লিখিত problem framing (যেমন সংক্ষিপ্ত design docs) কীভাবে ব্যয়বহুল ভুল বোঝাবুঝি ধরিয়ে দেয় তা ব্যাখ্যা করা।

### Complementary Authoritative Reading

- **Mandatory (~10 min — RFC 8174-এর সাথে একসাথে পড়ুন):** IETF RFC 2119 (S. Bradner, March 1997) — [Key words for use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119)। পুরো সংক্ষিপ্ত RFC-টি (~২ পৃষ্ঠা, Sections 1–6) পড়ুন; বিশেষভাবে Section 3–4 (`SHOULD` / `SHOULD NOT`-এর শর্ত), Section 5 (`MAY`-এর interoperability শর্ত), এবং Section 6 (*Guidance in the use of these Imperatives*) লক্ষ করুন।
- **Mandatory (~5 min — RFC 2119-এর সাথে একসাথে পড়ুন):** IETF RFC 8174 (B. Leiba, May 2017) — [Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words](https://datatracker.ietf.org/doc/rfc8174/)। পুরো সংক্ষিপ্ত RFC-টি (~২ পৃষ্ঠা) পড়ুন; খেয়াল করুন কেন কেবল **all-uppercase** রূপগুলোই BCP 14-এর বিশেষ normative অর্থ বহন করে আর lowercase রূপগুলো সাধারণ ইংরেজি হিসেবেই থাকে।
- **Mandatory (~20 min):** Malte Ubl (2020) — [Design Docs at Google](https://www.industrialempathy.com/posts/design-docs-at-google/)। পুরো প্রবন্ধটি পড়ুন—বিশেষ গুরুত্ব দিন *Context and Scope*, *Goals and Non-Goals*, *When Not to Write a Design Doc*, এবং *Lifecycle of a Design Doc* অংশগুলোতে।
- **Reference (Paywalled Standard — কেনার প্রয়োজন নেই):** ISO/IEC/IEEE 29148:2018, *Systems and software engineering — Life cycle processes — Requirements engineering* — [ISO Standard 72089](https://www.iso.org/standard/72089.html)। একটি আদর্শ একক requirement-এর নয়টি বৈশিষ্ট্য (Necessary, Appropriate, Unambiguous, Complete, Singular, Feasible, Verifiable, Correct, Conforming) ওপরের lesson-এ সরাসরি শেখানো হয়েছে।
- **Optional Supplementary (৩ ঘণ্টার বাজেটের বাইরে, বাণিজ্যিক বই):** Karl Wiegers & Joy Beatty, *Software Requirements* (3rd ed., Microsoft Press, 2013) অথবা Karl Wiegers & Candase Hokanson, *Software Requirements Essentials* (2023) — reader-side vs. stakeholder-side ambiguity ও intrinsically ambiguous words নিয়ে আরও উদাহরণের জন্য।

---

## Practical Reasoning Exercises (~55 Minutes Total)

> **Ownership Rule:** সব উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-02-notes.md` ফাইলে লিখুন। AI-কে দিয়ে উত্তর খসড়া করাবেন না। লক্ষ্য হলো ওপরের শেখা ধারণাগুলোকে সম্পূর্ণ নতুন দুটি backend দৃশ্যপটে স্বাধীনভাবে প্রয়োগ করা।

### Exercise 1: Ambiguous Ticket Interrogation — Order History Export API (~30 minutes)

**দৃশ্যপট (Scenario):**
একটি e-commerce কোম্পানির Product ও Customer Support টিম আপনাকে নিচের এক-বাক্যের backend ticket-টি দিয়েছে:

> *"Build an API endpoint so users can export their order history quickly and securely."*

*(প্রসঙ্গ স্মরণিকা: এখানে **authentication** মানে caller আসলেই কোন user তা যাচাই করা, আর **authorization** মানে সেই user-এর নির্দিষ্ট order data দেখার বা export করার অনুমতি আছে কি না তা নিশ্চিত করা—এগুলোর কারিগরি বাস্তবায়ন Phase 18-এ আসবে; এখানে কেবল সমস্যা ও শর্তের স্তরে চিন্তা করুন।)*

**কাজ (Tasks):**
কোড, framework বা file library বেছে নেওয়ার *আগে* নিচের চারটি অংশ `notes/block-02-notes.md` ফাইলে সম্পন্ন করুন:

1. **৮+টি সুনির্দিষ্ট Clarifying Questions:** কমপক্ষে **৮টি** (চাইলে আরও বেশি) সুনির্দিষ্ট প্রশ্ন তৈরি করুন এবং সেগুলোকে নিচের চারটি দলে সাজিয়ে লিখুন:
   - **(a) Behavior** *(কার্যগত আচরণ, ফরম্যাট, এবং সীমান্তবর্তী অবস্থা)*
   - **(b) Data Bounds** *(তথ্যের পরিমাণ, সীমা, গঠন এবং প্রাসঙ্গিক সীমানা)*
   - **(c) Security & Authorization** *(পরিচয়, অনুমতি, সংবেদনশীল তথ্য এবং অপব্যবহার প্রতিরোধ)*
   - **(d) Failure & Performance Constraints** *(গতির প্রত্যাশা, বড় অনুরোধের চাপ এবং ব্যর্থতার আচরণ)*
   - প্রতিটি প্রশ্নের পাশে ১ বাক্যে লিখুন: এই প্রশ্নের উত্তর না জেনে কোড লিখলে কী বাস্তব সমস্যা তৈরি হতে পারে।
2. **Hidden Assumptions-এর স্পষ্ট তালিকা:** এই এক লাইনের ticket-টির ভেতরে লুকিয়ে থাকা কমপক্ষে **৫টি unvalidated assumption** লিখুন, এবং প্রতিটি ক্ষেত্রে উল্লেখ করুন সেটি ভুল প্রমাণিত হলে design-এ কী ভেঙে পড়বে।
3. **Explicit Non-Goals:** প্রথম সংস্করণের জন্য কমপক্ষে **৩টি** বাস্তবসম্মত Non-Goal লিখুন—এমন যুক্তিসঙ্গত feature যা কেউ scope-এর অংশ ভাবতে পারত, কিন্তু সচেতনভাবে বাদ দেওয়া হচ্ছে (অর্থহীন নেতিবাচক বাক্য যেমন *"API crash করবে না"* ব্যবহার করবেন না)।
4. **৫টি Verifiable Acceptance Criteria (BCP 14 / RFC 2119 + RFC 8174):** ইংরেজি technical specification লেখার বাস্তব চর্চা হিসেবে **৫টি** সুনির্দিষ্ট, একক (`Singular`) ও যাচাইযোগ্য (`Verifiable`) acceptance criteria ইংরেজিতে লিখুন—যেখানে কেবল বড় হাতের (`ALL CAPS`) RFC 2119 / RFC 8174 keyword (`MUST`, `MUST NOT`, `SHOULD`, `SHOULD NOT`, `MAY`) সঠিকভাবে ব্যবহৃত হবে।
   - ৫টি শর্তের মধ্যে যেন Functional behavior, Authorization/Security, Data/Boundary handling, এবং Performance/Failure—একাধিক মাত্রা থাকে।
   - প্রতিটি acceptance criterion-এর নিচে বাংলায় ১–২ বাক্যে লিখুন: এটি Pass না Fail তা কীভাবে বস্তুনিষ্ঠভাবে verify করবেন, এবং নিশ্চিত করুন যে কোনো শর্তেই অকারণে নির্দিষ্ট implementation চাপিয়ে দিয়ে RFC 2119 Section 6 ভঙ্গ হয়নি।

---

### Exercise 2: Problem vs. Solution Refactoring — Redis Session Caching (~25 minutes)

**দৃশ্যপট (Scenario):**
একজন stakeholder এবং একজন সহকর্মী মিলে নিচের ticket-টি খুলে আপনার নামে assign করেছেন:

> *"Cache user sessions in Redis for 24 hours."*

*(প্রসঙ্গ স্মরণিকা: **user session** হলো কোনো ব্যবহারকারী login করার পর তাঁর সক্রিয় লগইন অবস্থা ও পরিচয় ধরে রাখার state, আর **Redis** হলো বহুল ব্যবহৃত একটি দ্রুত in-memory key-value data store যা প্রায়ই cache হিসেবে ব্যবহৃত হয়—caching ও session architecture-এর গভীর কৌশল আমরা যথাক্রমে Phase 16 ও Phase 18-এ পড়ব। এখানে আপনাকে Redis-এর ভেতরের কমান্ড জানতে হবে না; আপনার কাজ হলো আগেভাগে চাপিয়ে দেওয়া সমাধান থেকে আসল সমস্যাটিকে আলাদা করা।)*

**কাজ (Tasks):**
নিচের চারটি অংশ `notes/block-02-notes.md` ফাইলে বিশ্লেষণ করুন:

1. **Solution Space ছেঁটে ফেলে Problem Statement পুনর্গঠন:**
   - মূল ticket-এর কোন কোন অংশ আগেভাগে চাপিয়ে দেওয়া **Solution Space**-এর সিদ্ধান্ত তা চিহ্নিত করুন।
   - *"Redis"*, *"cache"*, বা *"24 hours"* শব্দগুলো ব্যবহার না করেই অন্তর্নিহিত **Problem Statement** নতুন করে লিখুন (ব্যবহারকারী বা সিস্টেমের কোন সম্ভাব্য ব্যথা, ধীরগতি, খরচ বা অভিজ্ঞতার সমস্যা সমাধান করতে এই অনুরোধ এসেছে বলে ধারণা করা যায়—এবং কোন তথ্যগুলো এখনো প্রমাণিত Fact নয়, তা স্পষ্ট করুন)।
2. **Constraints ও ব্যর্থতার পরিণতি (Failure Consequences):**
   - যেকোনো কারিগরি সমাধান বাছাই করার আগে security ও session-বাতিলকরণ (revocation), তথ্যের সতেজতা (freshness/staleness), availability ও partial failure-এর প্রভাব, এবং প্রকৃত performance bottleneck-সংক্রান্ত কোন কোন **constraints** ও ব্যর্থতার ঝুঁকি পরিষ্কার করা অপরিহার্য—তা স্বাধীনভাবে বিশ্লেষণ করুন।
3. **সিদ্ধান্ত নেওয়ার আগে ৬+টি অপরিহার্য প্রশ্ন:**
   - কোনো নির্দিষ্ট কারিগরি সমাধান বাছাই করার আগে stakeholder ও engineering টিমকে জিজ্ঞাসা করতে হবে এমন কমপক্ষে **৬টি** সুনির্দিষ্ট প্রশ্ন লিখুন।
4. **Design Doc সিদ্ধান্ত যাচাই:**
   - এই কাজের জন্য কি একটি সংক্ষিপ্ত Design Doc লেখা উচিত, নাকি সরাসরি কোডে হাত দেওয়া উচিত? Malte Ubl-এর *ambiguity* ও trade-off মানদণ্ড দিয়ে ৩–৪ বাক্যে আপনার অবস্থান যুক্তিসহ লিখুন।

---

## Closed-Book Mastery Assessment (Optional — Target ~25 Minutes)

এই assessment-টি **ঐচ্ছিক (optional)**—পরের block শুরু করার জন্য এটি সম্পন্ন করা বা ChatGPT-র অনুমোদন নেওয়া বাধ্যতামূলক নয়। আপনি চাইলে:
- এই lesson ও বাইরের সব tab বন্ধ করে নিচের পাঁচটি প্রশ্নের স্পষ্ট, মৌলিক ও cause-and-effect সমৃদ্ধ উত্তর `phases/phase-01/notes/block-02-notes.md` ফাইলে লিখে আনুষ্ঠানিক মূল্যায়নের জন্য **ChatGPT**-র কাছে শেয়ার করতে পারেন (`Mastery verified`-এর প্রমাণ হিসেবে);
- অথবা আপনি যথেষ্ট শিখেছেন মনে করলে assessment বাদ দিয়েই পরের block-এ যেতে পারেন (`Studied — mastery not formally verified`);
- অথবা assessment ছাড়াই যেকোনো অস্পষ্ট ধারণা নিয়ে প্রশ্ন ও আলোচনা করতে পারেন।

*(নোট: ~২৫ মিনিট কেবল যাঁরা এটি সম্পন্ন করতে চান তাঁদের জন্য একটি আনুমানিক লক্ষ্যমাত্রা—কঠোর সময়সীমা নয়। পাঁচটি বিশ্লেষণাত্মক প্রশ্ন গভীরভাবে লিখতে ২৫ মিনিটের বেশি লাগলে তা স্বাভাবিক।)*

1. **(LO 2.1 / Outcome 2)** একটি নিজস্ব backend উদাহরণ ব্যবহার করে **Problem Space** ও **Solution Space**-এর পার্থক্য ব্যাখ্যা করুন। *"তাড়াতাড়ি কোড লেখা শুরু করলেই দ্রুত কাজ শেষ হয়"* এবং *"Requirements বোঝা কেবল Product Manager-এর কাজ"*—এই দুটি ধারণা বাস্তব production backend-এ কেন ব্যর্থ হয়?
2. **(LO 2.2 / Outcome 3)** **Reader-Side Ambiguity** এবং **Stakeholder-Side Ambiguity**-র মধ্যে পার্থক্য কী, এবং দ্বিতীয়টিকে কেন সবচেয়ে মারাত্মক requirements ত্রুটি বলা হয়? একটি backend ফিচারের উদাহরণ দিয়ে **Fact**, **Assumption**, এবং **Dependency**-কে আলাদা করে দেখান।
3. **(LO 2.2 / Outcomes 2 & 5)** একটি কার্যকর **Non-Goal** এবং সাধারণ নেতিবাচক বিবৃতির মধ্যে পার্থক্য কী? Functional Requirements-এর পাশাপাশি **Non-Functional Requirements** এবং **Explicit Non-Goals** আগে থেকে নির্ধারণ না করলে সিস্টেম ও টিমের কী ক্ষতি হয়?
4. **(LO 2.3 / Outcome 3)** **ISO/IEC/IEEE 29148:2018**-এর পাঁচটি মূল মানদণ্ড (*Unambiguous, Complete, Singular, Feasible, Verifiable*) সংক্ষেপে ব্যাখ্যা করুন। এরপর **RFC 2119** ও **RFC 8174 (BCP 14)**-এর আলোকে `MUST`, `SHOULD`, এবং `MAY`-এর অর্থ, বড় হাতের বনাম ছোট হাতের অক্ষরের (uppercase vs. lowercase) পার্থক্য, এবং Section 6-এ imperative সংযমীভাবে ব্যবহারের নির্দেশ কেন জরুরি তা ব্যাখ্যা করুন।
5. **(LO 2.4 / Outcome 2)** একটি হালকা Google-ধাঁচের **Design Doc** কীভাবে কোড লেখার আগেই ব্যয়বহুল ভুল বোঝাবুঝি ধরায় (*Context & Scope*, *Goals & Non-Goals*, এবং lifecycle-এর ভূমিকা উল্লেখ করে)? আর কোন পরিস্থিতিতে Design Doc না লেখাই শ্রেয় engineering বিচারবুদ্ধি?

### Block 2 Readiness & Progression Checklist
- [ ] Guided lesson পড়েছেন এবং Pause & Checks 1–3 সম্পন্ন করেছেন (পরে Self-Check Guidance মিলিয়ে দেখেছেন)।
- [ ] Complementary mandatory reading (**RFC 2119**, **RFC 8174**, এবং Malte Ubl-এর ***Design Docs at Google***) সম্পন্ন করেছেন।
- [ ] `phases/phase-01/notes/block-02-notes.md` ফাইলে স্বাধীনভাবে Exercise 1 (Export API Interrogation) এবং Exercise 2 (Redis Session Refactoring) লিখেছেন।
- [ ] *(Optional)* বই/নোট বন্ধ রেখে Mastery Assessment Questions 1–5 লিখেছেন এবং আনুষ্ঠানিক মূল্যায়নের জন্য ChatGPT-র কাছে শেয়ার করেছেন।
- [ ] *(Optional)* কোনো ধারণা অস্পষ্ট মনে হলে assessment ছাড়াই সেগুলো নিয়ে ChatGPT-র সাথে আলোচনা করে পরিষ্কার হয়েছেন।

> **অগ্রগতির অবস্থা (Progression Status Distinction):** Block 3 কখন শুরু করবেন তা আপনার সিদ্ধান্ত। পড়া ও চর্চা শেষ করে assessment ছাড়া এগোলে এই block-টির অবস্থা হবে **Studied — mastery not formally verified**। যথেষ্ট প্রমাণ (exercise ও/অথবা assessment) শেয়ার করে মূল্যায়ন সম্পন্ন করলেই তা **Mastery verified** হিসেবে গণ্য হবে; assessment বাদ দিলে তা স্বয়ংক্রিয়ভাবে `Mastery verified` হয় না।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`notes/block-02-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের নির্দেশিকা দেখুন। Practical Reasoning Exercises বা Optional Closed-Book Mastery Assessment-এর কোনো উত্তর এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও (চাইলে) ঐচ্ছিক ChatGPT মূল্যায়নের জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* এখানে **"nightly script"** চালানো এবং **"৯০ দিনের পুরোনো log delete করা"**—দুটোই Solution Space-এর চাপিয়ে দেওয়া সিদ্ধান্ত। Problem Space বুঝতে আগে জিজ্ঞাসা করা দরকার: (ক) database load বা storage-এর প্রকৃত কারণ কি সত্যিই পুরোনো log, নাকি কোনো অদক্ষ query বা অপ্রয়োজনীয় logging? (খ) আইনি, audit বা debugging প্রয়োজনে log কতদিন সংরক্ষণ (retention/archive) করা বাধ্যতামূলক, এবং সরাসরি delete করলে compliance বা incident তদন্তে কী ক্ষতি হবে?
  2. *শক্তিশালী উত্তরে থাকবে:* একজন customer বা Product Manager *"সাথে সাথে টাকা ফেরত"* বলতে বোঝেন cancel চাপার কয়েক সেকেন্ডের মধ্যে customer-এর bank account-এ টাকা ঢুকে যাবে; অন্যদিকে backend বা payment engineer বোঝেন আমাদের backend সাথে সাথে payment gateway-র কাছে refund request পাঠাবে (যদিও bank-এ টাকা ঢুকতে ৩–৭ কর্মদিবস লাগে)। দুজনেই একই বাক্য পড়ে নিজের অর্থ নিয়ে ১০০% নিশ্চিত থাকেন বলে আগে কেউ প্রশ্ন তোলেন না—এটাই Stakeholder-Side Ambiguity।
  3. *শক্তিশালী উত্তরে থাকবে:* প্রতিটি request-এ বৈধ user token আসে কি না—এটি API gateway/code যাচাই করে প্রমাণ করা থাকলে **Fact**, নাহলে একটি **Assumption** (যেমন কোনো public endpoint বা expired token-এর ক্ষেত্র বাদ পড়ে যেতে পারে)। Payment gateway-র সময়মতো উত্তর দেওয়া একটি **Dependency** (এবং "৯৯.৯% সময় ৫০০ ms"-কে পরিমাপ ছাড়া ধরে নিলে সেটি unvalidated assumption-ও)। *"কোনো customer দিনে ৫টির বেশি order করেন না"* প্রায় নিশ্চিতভাবেই একটি **Assumption**—কোনো reseller, corporate buyer বা flash sale এলে এটি ভাঙবেই। Assumption-কে Fact ধরে নিলে তার বিরুদ্ধে কোনো সুরক্ষা বা ব্যর্থতা-সামলানোর ব্যবস্থা লেখা হয় না।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* মোট চার ধরনের ত্রুটি আছে: (ক) **RFC 8174:** ছোট হাতের `should` সাধারণ ইংরেজি অর্থ বহন করে, কোনো BCP 14 normative শর্ত নয়—ফলে lock করা বাধ্যতামূলক না ঐচ্ছিক তা অস্পষ্ট। (খ) **ISO 29148 (`Unambiguous`, `Complete`, `Verifiable`):** কতবার ব্যর্থ চেষ্টার পর, কত সময়ের মধ্যে, কতক্ষণের জন্য account lock হবে এবং লক হলে caller কী response পাবে—কিছুই নির্দিষ্ট নেই, তাই Pass/Fail যাচাই করা অসম্ভব। (গ) **ISO 29148 (`Singular`):** account-locking নীতি এবং অভ্যন্তরীণ SQL query কৌশল—দুটি ভিন্ন বিষয় এক বাক্যে জোড়া লাগানো হয়েছে। (ঘ) **RFC 2119 Section 6 ও ISO 29148 (`Appropriate`):** `MUST use a PostgreSQL SELECT ... FOR UPDATE query` অকারণে অভ্যন্তরীণ implementation পদ্ধতি জোর করে চাপিয়ে দিচ্ছে, যা interoperability বা ক্ষতি ঠেকানোর জন্য অপরিহার্য নয়।
  2. *শক্তিশালী উত্তরে থাকবে (নমুনা কাঠামো):* *"If an account records 5 consecutive failed password authentication attempts within a rolling 15-minute window, the authentication service `MUST` reject subsequent password sign-in attempts for that account with HTTP `429` and error code `ACCOUNT_LOCKED` for the next 15 minutes."* — **Verification:** ১৫ মিনিটের মধ্যে ৫ বার ভুল password দিয়ে চেষ্টা করার পর ৬ষ্ঠ বার (সঠিক password হলেও) HTTP `429` ও `ACCOUNT_LOCKED` ফেরত আসছে এবং ১৫ মিনিট পর পুনরায় সচল হচ্ছে কি না তা automated test দিয়ে যাচাই করা।
     - *গুরুত্বপূর্ণ engineering সতর্কতা (Security Trade-off):* ওপরের বিবৃতিটি গঠনগতভাবে স্পষ্ট, একক ও পরিমাপযোগ্য হলেও, **একটি requirement পরিমাপযোগ্য হওয়া মানেই তা ভালো engineering সিদ্ধান্ত নয়।** বাস্তব production-এ account-wide lockout অপব্যবহারের শিকার হতে পারে—কোনো আক্রমণকারী ইচ্ছাকৃতভাবে একজন বৈধ user-এর পরিচিত account-এ ৫ বার ভুল password দিয়ে সেই বৈধ user-কেই নিজের account থেকে বাইরে আটকে রাখতে পারেন। তাই threat model অনুযায়ী অনেক পরিস্থিতিতে কঠোর account-wide lockout-এর বদলে **rate limiting**, ক্রমবর্ধমান বিলম্ব (**progressive delays**), বা অন্যান্য abuse-resistant নিয়ন্ত্রণ ব্যবস্থা বেশি উপযুক্ত হয় (authentication ও abuse defense-এর বিস্তারিত Phase 18 ও 19-এ আসবে)।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* **(ক)**-তে সমস্যা ও সমাধান দুটোই স্পষ্ট, টিমের বিদ্যমান প্রমিত error format আছে এবং কোনো বড় trade-off বা বহু-টিম সমন্বয়ের অস্পষ্টতা নেই—তাই এখানে Design Doc লেখা অপ্রয়োজনীয় overhead। **(খ)**-তে তিনটি টিম (Checkout, Inventory, Billing), data consistency, partial failure, এবং স্পষ্ট scope সীমানা জড়িত—অর্থাৎ উচ্চ ambiguity ও বড় blast radius আছে—তাই কোড লেখার আগে একটি সংক্ষিপ্ত Design Doc লিখে review-এর মাধ্যমে গুরুত্বপূর্ণ মতবিরোধ মিটিয়ে পর্যাপ্ত সম্মতি (sufficient agreement) তৈরি করা অত্যন্ত যুক্তিসঙ্গত।
  2. *শক্তিশালী উত্তরে থাকবে:* ভালো Non-Goal মানে কোনো অর্থহীন নেতিবাচক প্রতিশ্রুতি নয় (কোনো টিমই ইচ্ছে করে security bug বা ধীর system বানাতে চায় না)। বরং একটি কার্যকর Non-Goal এমন একটি যুক্তিসঙ্গত, বাস্তবায়নযোগ্য feature বা দিককে স্পষ্টভাবে বাদ দেয়, যা কেউ সহজেই প্রকল্পের আওতাভুক্ত বলে ধরে নিতে পারত—যাতে scope creep ও অপ্রাসঙ্গিক review বিতর্ক থামে।
