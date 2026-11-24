# Phase 1, Block 1 — What Is Software Engineering?

ধরুন, একটি ছোট online store আপনাকে একটি promotional code যোগ করতে বলল: checkout-এর সময় কোনো customer যদি `SAVE10` টাইপ করে, তাহলে order-এর মোট মূল্য থেকে 10% ছাড় দেওয়া হবে।

প্রথম দিন কাজটি খুব সহজ মনে হয়। আপনি দশ লাইনের একটি JavaScript function লিখলেন—যেটি যাচাই করে code-টি `"SAVE10"` কি না, মোট মূল্যকে `0.9` দিয়ে গুণ করে, আর ছাড় দেওয়া দামটা return করে। ১০০ ডলারের একটি order দিয়ে test করে দেখলেন ৯০ ডলার আসছে। দুপুরের আগেই কাজ শেষ।

দুই সপ্তাহ পর customer-দের website এবং mobile app—দুই জায়গা থেকেই `SAVE10` ব্যবহার করতে হবে। আপনার ল্যাপটপে পড়ে থাকা একটা আলাদা function তো আর live shopper-দের service দিতে পারবে না। তাই আপনি function-টিকে একটি **backend service**-এর ভেতর নিয়ে এলেন—অর্থাৎ এমন একটি program, যা একটি **server**-এ সারাক্ষণ চালু থাকে, **network**-এর মাধ্যমে **client app** থেকে request গ্রহণ করে এবং **database**-এ সংরক্ষিত data পড়ে বা লেখে। আপনি চালু থাকা promo code-গুলো database-এ রাখলেন, service-টিকে checkout-এর সাথে যুক্ত করলেন, একটি স্বাভাবিক order দিয়ে test করলেন, আর feature-টি live করে দিলেন। বাস্তব customer-রা `SAVE10` ব্যবহার করে ছাড় পাচ্ছেন, সবাই খুশি।

এবার চৌদ্দ মাস এগিয়ে যান।

যে engineer শুরুতে promo feature-টি লিখেছিলেন, তিনি কোম্পানি ছেড়ে চলে গেছেন। ইতিমধ্যে order-এর সংখ্যা দশ গুণ বেড়েছে। Marketing টিম এখন এমন code চায় যেগুলো মধ্যরাতে expire হয়ে যাবে, শুধু প্রথমবার কেনাকাটা করা buyer-দের জন্য কাজ করবে, কিংবা অন্য কোনো ছাড়ের সাথে একসাথে (combine) ব্যবহার করা যাবে না। নতুন একজন developer যখন expiration date যোগ করতে **codebase** খোলেন, তখন দেখেন discount-সংক্রান্ত নানা অনুমান এক ডজন ফাইলে ছড়িয়ে আছে; কোনো edit করার পর checkout ঠিকমতো কাজ করছে কি না তা যাচাই করার মতো কোনো **automated tests** নেই; এমনকি mobile app-এর কোন অংশ exact **response format**-এর ওপর নির্ভর করছে, তাও কেউ নিশ্চিতভাবে জানে না। যে পরিবর্তনটা এক বিকেলে হয়ে যাওয়ার কথা ছিল, তা এক সপ্তাহের ঝুঁকিপূর্ণ জুয়ায় পরিণত হলো—যেখানে সামান্য ভুলে store-এর প্রতিটি order-এ 100% ছাড় চলে যেতে পারে।

একটু থেমে খেয়াল করুন, এখানে আসলে কী ঘটল। প্রথম দিন কোনো ভুল ছিল না—আপনার দশ লাইনের function একদম সঠিক হিসাব দিয়েছিল। এমনকি দ্বিতীয় সপ্তাহে checkout feature-টি ship করাও সফল হয়েছিল। যা ব্যর্থ হয়েছে, তা হলো **প্রথম দিনের পরের পুরো সময়টা**।

এই ব্যবধানটাই **Programming** এবং **Software Development**-কে **Software Engineering** থেকে আলাদা করে।

চাকরির বিজ্ঞাপন বা দৈনন্দিন কথাবার্তায় এই তিনটি শব্দ প্রায়ই একই অর্থে ব্যবহার করা হয়, এবং এদের আলাদা করার কোনো একক সর্বজনীন ISO standard নেই। তবুও এই তিনটি *কাজকে* গুলিয়ে ফেললে বাস্তব production system-এ বড় ধরনের ব্যর্থতা ঘটে। Titus Winters, Tom Manshreck ও Hyrum Wright (*Software Engineering at Google*, 2020) এবং IEEE-র software engineering body of knowledge-এর ওপর ভিত্তি করে আমরা এই পুরো প্রোগ্রামে নিচের স্পষ্ট working distinction ব্যবহার করছি:

| দৃষ্টিভঙ্গি (Lens) | মূল কাজ (Core Focus) | মূল প্রশ্ন (Driving Question) | এখানেই থেমে গেলে কী ভেঙে পড়ে |
| :--- | :--- | :--- | :--- |
| **Programming** | স্পষ্টভাবে নির্দিষ্ট কোনো কাজকে কার্যকর কোডে রূপান্তর করা। | *"এই কোডটা কি এখনই চলে এবং সঠিক উত্তর দেয়?"* | Requirement, caller, বা data বদলালেই ভেঙে পড়ে। |
| **Software Development** | ব্যবহারকারীদের জন্য software feature তৈরি, integrate ও ship করা। | *"Feature-টি কি ঠিকমতো কাজ করে এবং user-দের কাছে পৌঁছাল?"* | ভঙ্গুর contract, জমে থাকা upgrade debt এবং operational blind spot তৈরি হয়। |
| **Software Engineering** | **Time (সময়)**, **Scale (পরিসর)** এবং **Competing Constraints (পরস্পরবিরোধী সীমাবদ্ধতা)**-এর মধ্য দিয়ে software-কে নির্ভরযোগ্যভাবে টিকিয়ে রাখা, verify করা, evolve করা ও পরিচালনা করা। | *"সময়ের সাথে টিম বদলালেও কি এই system-টিকে এর expected lifespan জুড়ে নিরাপদে পরিবর্তন, upgrade, verify এবং operate করা যাবে?"* | *এর নিজের ব্যর্থতার ধরন:* ছোট ও ক্ষণস্থায়ী কাজে অতিরিক্ত ভারী engineering প্রক্রিয়া চাপিয়ে দেওয়া। |

খেয়াল করুন, Software Engineering মানে কী *নয়*। অনেকেই মনে করেন Software Engineering মানে শুধু আরও সুন্দর, পরিচ্ছন্ন syntax লেখা বা বেশি framework জানা। পরিচ্ছন্ন কোড অবশ্যই গুরুত্বপূর্ণ—তা কোড পড়ে বোঝা সহজ করে—কিন্তু এক বছর পর যে checkout service-টি কেউ নিরাপদে test, upgrade বা পরিবর্তন করতে পারে না, নিখুঁত variable নাম দিয়ে তাকে বাঁচানো যায় না। Software Engineering কখনো Programming-এর বিকল্প নয়; ভালো Programming আপনাকে জানতেই হবে। Software Engineering কেবল এই বাস্তবতাটাকে স্বীকার করে যে **প্রথমবার কোড লেখা সাধারণত একটি production system-এর জীবনের সবচেয়ে ছোট অধ্যায়।**

এখানে একটা স্বাভাবিক পাল্টা প্রশ্ন উঠতে পারে। *ধরুন, marketing টিম কখনোই নতুন কোনো discount rule চাইল না। আমরা যদি দ্বিতীয় সপ্তাহের কার্যকর promo service-টাতে আর কোনো হাত না দিয়ে একদম যেমন আছে তেমনই রেখে দিই, তাহলে কি এটা চিরকাল নিরাপদে চলতে থাকবে না?*

---

## হাত না দিলেও Software কেন ভেঙে পড়ে

Software-এ লোহার মতো জং ধরে না বা এটা ভৌতভাবে ক্ষয়ে যায় না। তবুও কোনো কোডে হাত না দিলেও production service প্রায়ই ভেঙে পড়ে। কারণটা খুব সোজা: **আপনার কোড কখনো শূন্যতার মধ্যে চলে না—এর চারপাশের পরিবেশ প্রতিনিয়ত বদলাতেই থাকে।**

আমাদের ছোট `SAVE10` backend-টিও একটি চলমান প্রযুক্তিগত স্তরের (stack) ওপর দাঁড়িয়ে আছে। আপনার JavaScript কোড চলে একটি **runtime**-এর ভেতর—আমাদের মূল stack-এ সেটি হলো **Node.js**, এমন একটি **runtime environment**, যা **V8 JavaScript engine** ব্যবহার করে server-side JavaScript execute করে এবং server-side কাজের জন্য প্রয়োজনীয় API দেয়। এর পাশাপাশি আপনার কোড অন্যদের লেখা external library বা **dependencies** (যেমন `npm` দিয়ে install করা package)-এর ওপর নির্ভর করে, একটি **database** (যেমন PostgreSQL) থেকে data পড়ে ও লেখে, এবং ইন্টারনেটের মাধ্যমে client app-গুলোর সাথে যোগাযোগ করে।

এবার ভাবুন, আপনার টিম আঠারো মাস ধরে promo service-এ এক লাইন কোডও commit করল না। তা সত্ত্বেও এর চারপাশে কী কী বদলে যায় দেখুন:

- **নতুন security vulnerability ধরা পড়ে।** গবেষকেরা নিয়মিত runtime এবং বহুল ব্যবহৃত library-গুলোতে নিরাপত্তার দুর্বলতা খুঁজে পান। অনেক publicly disclosed vulnerability-কে শনাক্ত করার একটি common উপায় হিসেবে **CVE** *(Common Vulnerabilities and Exposures)* identifier দেওয়া হয়, এবং maintainer-রা সেগুলো বন্ধ করতে **security patches** প্রকাশ করেন (আমরা Phase 19-এ security-র কৌশলগুলো গভীরে পড়ব)। কোডে হাত না দিলে নতুন আবিষ্কৃত আক্রমণের সামনে আপনার service অরক্ষিত থেকে যায়।
- **নিচের platform-গুলোর মেয়াদ শেষ হয়ে যায়।** প্রতিটি বড় runtime version—এমনকি Long-Term Support (**LTS**) release-ও—একদিন **End-of-Life (EOL)** অবস্থায় পৌঁছায়, অর্থাৎ সেই বিন্দুতে maintainer-রা আর কোনো security fix দেওয়া বন্ধ করে দেন। Database এবং cloud platform-গুলোও একইভাবে পুরোনো আচরণ ধাপে ধাপে বাদ দেয় এবং পুরোনো version অবসর দেয়।
- **Data এবং traffic বাড়তে থাকে।** ৫০টি promo record খুঁজে দেখতে এক মিলিসেকেন্ডের ভগ্নাংশ সময় লাগে; কিন্তু ৫০ লাখ redemption record ঘেঁটে দেখতে গেলে checkout দৃশ্যমানভাবে ধীর হয়ে যায় (বৃদ্ধির হার বা growth rate আমরা Phase 2-এ এবং **database indexing** Phase 10-এ শিখব)।
- **বাইরের বাস্তবতা বদলায়।** করের নিয়ম (tax laws), payment processor, mobile operating system, আর partner integration—আপনার repository বদলাক বা না বদলাক, এগুলো বদলাতেই থাকে।

দীর্ঘমেয়াদি যেকোনো service-এ বাইরের পরিবর্তন অনিবার্য হওয়ায় নিয়মিত রক্ষণাবেক্ষণ এড়িয়ে চললে একটি পরিচিত ফাঁদ তৈরি হয়—যাকে বলে **deferral spiral**। ধরুন, ৬ষ্ঠ মাসে Node.js ও package-গুলোর একটি নিয়মিত upgrade করতে চার ঘণ্টা লাগত; টিম ভাবল *"এখন তো কিছুই ভাঙেনি, পরে করা যাবে।"* ১৮তম মাসে গিয়ে দেখা গেল Node.js ও ডজনখানেক library-র পাঁচটি breaking change একটার ওপর আরেকটা জমে গেছে, শুরুর engineer-রা চলে গেছেন, আর test coverage দুর্বল। ২৪তম মাসে এসে EOL হয়ে যাওয়া runtime-এ একটি গুরুতর vulnerability ধরা পড়ল—যার মাধ্যমে আক্রমণকারীরা আপনার server-এ অননুমোদিত কোড চালাতে পারে। তখন upgrade করা ছাড়া আর কোনো উপায় থাকে না, অথচ জরুরি চাপের মধ্যে production অরক্ষিত রেখে কয়েক সপ্তাহ ধরে ঝুঁকিপূর্ণ অস্ত্রোপচার চালাতে হয়।

এই কারণেই Winters et al. (2020) Software Engineering-কে সংজ্ঞায়িত করেছেন **"programming integrated over time"** হিসেবে এবং **software sustainability**-র একটি ব্যবহারিক সংজ্ঞা দিয়েছেন: একটি codebase ও সংগঠন তখনই sustainable, যখন তারা software-টির **expected lifespan** (প্রত্যাশিত জীবনকাল) জুড়ে প্রয়োজনীয় প্রতিটি মূল্যবান কারিগরি বা ব্যবসায়িক পরিবর্তনে নিরাপদে সাড়া দিতে পারে। *(খেয়াল রাখবেন, এখানে sustainability বলতে strictly technical ও organizational lifecycle sustainability বোঝানো হচ্ছে—পরিবেশগত বা environmental sustainability নয়।)*

যে service-এ আজ একটিও bug ticket নেই, অথচ সেটি পরিবর্তন বা upgrade করার কথা ভাবলেই প্রতিটি engineer আতঙ্কিত হয়ে পড়েন—তা একটি engineered system হিসেবে ইতিমধ্যেই ব্যর্থ। তবে *"ঠিকঠাক চললে হাত দিয়ো না"* কথাটা *সব সময়* ভুল নয়: ধরুন, একটি ছোট ও বিচ্ছিন্ন (isolated) internal script আগামী সপ্তাহেই ফেলে দেওয়া হবে—কোনো security বা upgrade ঝুঁকি আসার আগেই। সে ক্ষেত্রে তাতে হাত না দেওয়াই বুদ্ধিমানের কাজ। **Expected lifespan** এখানে engineering সিদ্ধান্ত বদলে দেয়।

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-01-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. এই মাসে কোনো open bug ticket না থাকা এবং 100% uptime থাকার পরও একটি backend service কীভাবে *unsustainable* হতে পারে?
> 2. একজন সহকর্মী বললেন: *"আমাদের login service-এর dependencies-এ হাত দিয়ো না—এখন তো ঠিকই চলছে।"* Cause-and-effect দেখিয়ে এই যুক্তির দুর্বলতা ব্যাখ্যা করুন, এবং এমন একটি বাস্তব পরিস্থিতি বলুন যেখানে কার্যকর কোডে হাত না দেওয়াটা আসলেই যুক্তিসঙ্গত।

দীর্ঘমেয়াদি software-কে যে পরিবর্তন করতেই হয়, এটা মেনে নিলে আমরা সরাসরি আরেকটি অস্বস্তিকর ধাঁধার সামনে পড়ি। ধরুন, আপনি দায়িত্বশীল engineer-এর মতোই কাজ করলেন: promo service-টিকে maintainable রাখলেন এবং একটি সচেতন, সম্পূর্ণ বৈধ internal improvement করলেন, যা আপনার documentation-এর প্রতিটি শব্দের সাথে শতভাগ মেলে। **তবুও সম্পূর্ণ নিয়মমাফিক এমন একটি পরিবর্তন কীভাবে live customer-দের কাজ ভেঙে দিতে পারে?**

---

## "নিরাপদ" পরিবর্তনও যেভাবে User-দের ভেঙে দেয়

একজন ছোট দোকানদারের কথা ভাবুন, যিনি প্রতিদিন customer-দের কাগজের receipt-গুলো একটা জুতার বাক্সে ফেলে রাখেন। তিনি কখনো প্রতিশ্রুতি দেননি যে receipt-গুলো তারিখের ক্রম অনুযায়ী সাজানো থাকবে—শুধু এটুকু বলেছিলেন যে সব receipt বাক্সের ভেতর আছে। তবুও দোকানের হিসাবরক্ষক লক্ষ করলেন, সবচেয়ে নতুন receipt-টা সব সময় ওপরে পড়ে থাকে; তাই তিনি রোজ ওপরের কাগজটা তুলে নেওয়ার অভ্যাস গড়ে তুললেন। একদিন বিকেলে দোকানদার দ্রুত customer খুঁজে পাওয়ার সুবিধার্থে সব receipt বর্ণানুক্রমে (alphabetically) একটি ফোল্ডারে গুছিয়ে রাখলেন। কোনো প্রতিশ্রুতি ভাঙা হয়নি, প্রতিটি receipt-ই জায়গামতো আছে—তবু পরদিন সকালেই হিসাবরক্ষকের কাজ নিঃশব্দে ভেঙে পড়ল।

Backend system-এ ঠিক একই ঘটনা প্রতিনিয়ত ঘটে।

আমাদের বাড়তে থাকা promo-code service-এ ফিরে আসুন। ধরুন, এটি একটি **API** *(Application Programming Interface — যে সীমানা বা interface দিয়ে client app এবং partner program-গুলো request পাঠায় ও response পায়)* উন্মুক্ত করে, যার দুটি কাজ আছে: একটি endpoint কোনো customer-এর জন্য প্রযোজ্য (eligible) promo code-গুলোর তালিকা দেয়, আরেকটি code redeem করে—এবং redeem করা না গেলে একটি error message ফেরত দেয়। আপনার documentation মাত্র দুটি প্রতিশ্রুতি দেয়: *"Returns a list of the customer's eligible promo codes; returns an error message if a code cannot be redeemed."*

লক্ষ করুন, দুটি বিষয়ে আপনি কোনো প্রতিশ্রুতি দেননি: তালিকার কোনো নির্দিষ্ট sort order, এবং error message-এর হুবহু ইংরেজি বাক্য।

এক বছর ধরে নতুন promo code-গুলো একটার পর একটা জমা হওয়ায় তালিকাটি সব সময় পুরোনো-আগে, নতুন-শেষে—এই ক্রমেই আসত; আর মেয়াদোত্তীর্ণ কোডে সব সময় `"Code expired"` লেখাটা আসত। Mobile app টিম বা external partner-রা তো আর আপনার মনের কথা পড়তে পারে না; তারা আপনার service বাস্তবে যা ফেরত দেয় তার ওপর ভিত্তি করেই test করেছে। নীরবে mobile app-টি home screen-এ *"Newest Offer"* দেখাতে `codes[codes.length - 1]` ব্যবহার করল এবং মেয়াদ ফুরানোর banner দেখাতে `if (error.message === "Code expired")` লিখে রাখল। Testing আর production—দুই জায়গাতেই সব দারুণ কাজ করছিল।

১৬তম মাসে একজন backend engineer eligible code খোঁজার ভেতরের পদ্ধতিটা দ্রুততর করলেন—ফলে ভেতরের lookup order বদলে গেল—আর error-এর ভাষাটা মার্জিত করে করলেন `"This promotional code has expired."` আপনার documentation-এর প্রতিটি লিখিত প্রতিশ্রুতি তখনো অটুট। কিন্তু রাতারাতি mobile shopper-রা এক বছরের পুরোনো offer-কে *"Newest Offer"* হিসেবে দেখতে লাগলেন, আর expired-code banner পুরোপুরি গায়েব হয়ে গেল।

*Software Engineering at Google* (2020) বইয়ে Hyrum Wright এই বাস্তব অভিজ্ঞতাভিত্তিক ঘটনাটি সূত্রবদ্ধ করেন, যা আজ **Hyrum's Law** নামে সুপরিচিত:

> *"With a sufficient number of users of an API, it does not matter what you promise in the contract: all observable behaviors of your system will be depended on by somebody."*

অর্থাৎ কোনো API-এর পর্যাপ্তসংখ্যক ব্যবহারকারী তৈরি হলে, আপনি **contract**-এ (লিখিত প্রতিশ্রুতিতে) কী দিয়েছেন তা আর মূল বিষয় থাকে না—আপনার system-এর প্রতিটি **observable behavior** (বাইরে থেকে দৃশ্যমান আচরণ)-এর ওপর কেউ না কেউ নির্ভর করে বসবে।

ভালো করে লক্ষ করুন, Hyrum's Law "খারাপ" বা "অলস" client developer-দের কারণে ঘটে না। যেকোনো interface সীমানায় এটি একটি স্পষ্ট চার-ধাপের mechanism অনুসরণ করে:

1. **Caller-রা আপনার ভেতরের উদ্দেশ্য নয়, বাস্তব আচরণ দেখে।** অন্য কোনো system যখন আপনার সাথে integrate করে, তখন আসল request-এ কী ফেরত আসছে সেটাই তার কাছে সত্য।
2. **অনিয়ন্ত্রিত অংশগুলো (unconstrained degrees of freedom) বাইরে বেরিয়ে যায়।** Sort order, error-এর ভাষা, response time, বা বাড়তি field-গুলো যদি আপনি নিজে নির্ধারণ না করেন, তবে আপনার বর্তমান implementation ঘটনাক্রমে যা তৈরি করে সেটাই বাইরে দৃশ্যমান হয়ে পড়ে।
3. **Client কোড ও test সেই পর্যবেক্ষণ ঘিরেই শক্ত হয়ে বসে।** Downstream engineer-রা যা নিয়মিত কাজ করছে, তার ওপর ভিত্তি করেই নিজেদের logic আর automated test লিখে ফেলেন।
4. **ভবিষ্যতের বৈধ internal পরিবর্তন অদৃশ্য ব্যবহারকারীদের সাথে সংঘর্ষে জড়ায়।** কয়েক মাস পর কোনো সঠিক internal উন্নয়ন যখন সেই ঘটনাক্রমিক আচরণটা বদলে দেয়, তখন আপনার অজান্তেই নির্ভরশীল caller-রা ভেঙে পড়ে।

এই mechanism-টা বুঝে ফেললে তিনটি বাস্তব engineering প্রতিরোধব্যবস্থা সরাসরি বেরিয়ে আসে:

- **ঘটনাক্রমিক অনিশ্চয়তাগুলো ইচ্ছাকৃতভাবে বন্ধ করুন।** কোনো endpoint যদি eligible promo code-এর তালিকা দেয়, তাহলে storage layout-কে ঘটনাক্রমে ক্রম ঠিক করতে দেওয়ার বদলে product-এর প্রয়োজন অনুযায়ী একটি সচেতন ordering policy বেছে নিয়ে তা explicitly enforce করুন—যেমন নতুন তৈরি হওয়া offer আগে রাখা (`created_at DESC`) এবং একই সময়ে তৈরি কোডগুলোর tie-breaker হিসেবে একটি stable ID ব্যবহার করা—এবং সেটি documentation-এ লিখে দিন।
- **কাঁচা বাক্য বা internal detail নয়, structured contract দিন।** `{ "code": "PROMO_EXPIRED", "message": "This promotional code has expired." }`-এর মতো stable machine-readable error code ফেরত দিন, যাতে client-রা ভঙ্গুর ইংরেজি বাক্য বা raw database error-এর বদলে `PROMO_EXPIRED`-এর ওপর নির্ভর করে।
- **কোন কোন field বাইরে যাবে, তা স্পষ্টভাবে map করুন।** ভেতরের পুরো raw database record সরাসরি পাঠিয়ে না দিয়ে সচেতনভাবে নির্দিষ্ট field-গুলো বাছাই করে response তৈরি করুন, যাতে আগামীকাল ভেতরে নতুন কোনো column বা field যোগ করলেও API response নীরবে বদলে না যায়।

তবে Hyrum's Law-কে অন্ধ কুসংস্কারে পরিণত করবেন না। এর দুটি স্পষ্ট সীমা আছে। প্রথমত, এটি **scale-dependent**: এর জন্য *"পর্যাপ্তসংখ্যক ব্যবহারকারী"* লাগে। আপনার নিজের নিয়ন্ত্রণাধীন একটি repository-র ভেতর মাত্র দুই জায়গায় ডাকা একটি private helper function কখনো ৪০০ external partner-এর ব্যবহৃত public API-এর মতো জমে শক্ত হয়ে যায় না। দ্বিতীয়ত, এর মানে এই **নয়** যে আপনি কখনোই observable behavior বদলাতে পারবেন না। এর অর্থ হলো আপনাকে অদৃশ্য নির্ভরতার সম্ভাবনা *আগে থেকেই ধরে নিতে হবে*, সম্ভব হলে প্রকৃত ব্যবহার পরিমাপ করতে হবে, পুরোনো আচরণ বাদ দেওয়ার আগে caller-দের আগাম সতর্কবার্তা দিতে হবে, এবং পরিবর্তনের লাভটা migration-এর কষ্টের চেয়ে বড় কি না তা সচেতনভাবে বিচার করতে হবে। Production-এ *"আমরা তো doc-এ order-এর কথা লিখিনি, দোষ client-এর"*—এই যুক্তি চলে না; কারণ টাকা দেওয়া customer-এর সেবা বন্ধ হয়ে গেলে সেটা আপনারই outage।

এতক্ষণ আমরা দেখলাম এক-দুজন engineer সময়ের সাথে একটি ছোট promo service সামলাচ্ছেন। এবার দৃশ্যটা বড় করুন: **দোকানটি যদি সফল হয় এবং engineering টিম ২ জন engineer ও ৩টি endpoint থেকে বেড়ে ৩০ জন engineer ও ২০০টি endpoint-এ পৌঁছায়, তখন কী ঘটে?**

---

## পরিসর বাড়লে পুরোনো অভ্যাস কেন ভেঙে পড়ে—এবং সময়ের হিসাব কেন জরুরি

Software Engineering-কে শুধু **time** (সময়) নয়, **scale** (পরিসর)-এর চাপও সামলাতে হয়—codebase-এর আকার, engineer-এর সংখ্যা, service-এর সংখ্যা, user traffic এবং external dependency-র বৃদ্ধি। ছোট পরিসরে যে অভ্যাসগুলো একেবারে ঝামেলামুক্ত মনে হয়, টিম বড় হলেই সেগুলো পুরোপুরি ভেঙে পড়ে।

ধরুন, আমাদের promo service-এ যখন মাত্র ২ জন engineer, তখন একটি সহজ নিয়ম ছিল: *"আমাদের shared discount-validation helper-এ কোনো পরিবর্তন করলে অন্য engineer-কে message দিয়ে জানিও এবং যে ৩টি endpoint এটা ব্যবহার করে সেগুলো update করে দিয়ো।"* ২ জন engineer ও ৩টি endpoint-এর জন্য এতে দশ মিনিট লাগে—একদমই যুক্তিসঙ্গত।

এবার এগিয়ে যান ৩০ জন engineer-এর টিমে, যারা checkout, billing, subscriptions আর mobile backend জুড়ে ২০০টি endpoint সামলাচ্ছেন। এখন ওই একই manual নিয়ম মানতে গেলে দশটি টিম খুঁজে বের করতে হবে, ডজনখানেক message পাঠাতে হবে, উত্তরের জন্য সপ্তাহের পর সপ্তাহ অপেক্ষা করতে হবে, আর যারা notification দেখেনি তাদের পেছনে দৌড়াতে হবে। ফলাফল: production-এ পুরোনো ও নতুন validation আচরণ পাশাপাশি চলতে থাকবে, আর প্রতিটি ভবিষ্যৎ upgrade আরও ধীর হয়ে পড়বে।

কেন এমন হলো, তার পেছনের হিসাবটা দেখুন। কোনো নিয়মিত প্রক্রিয়ার (recurring policy) মানবিক পরিশ্রম যদি codebase-এর আকার বা টিমের লোকসংখ্যার সাথে সমানুপাতে বাড়তে থাকে—অর্থাৎ endpoint দ্বিগুণ হলে manual কাজও দ্বিগুণ (**linear** growth), কিংবা আরও খারাপ, প্রতিটি টিমকে অন্য প্রতিটি টিমের সাথে সমন্বয় করতে হয় (**super-linear** growth; growth curve আমরা Phase 2-এ গাণিতিকভাবে শিখব)—তাহলে একসময় মানুষের সক্ষমতা পুরোপুরি ফুরিয়ে আসে। তাই **উচ্চ-পুনরাবৃত্তি ও ব্যাপক পরিসরের যান্ত্রিক নিয়মগুলোর (high-frequency, broad-scope mechanical policies)**—যেমন code formatting, automated test চালানো, পরিচিত vulnerability-র জন্য dependency যাচাই, বা নিয়মিত deployment—জন্য engineer-রা automated tooling-এ বিনিয়োগ করেন, যাতে system বড় হওয়ার সাথে মানুষের পরিশ্রম অনেক ধীরে বাড়ে (**sub-linearly**)।

এই নীতির একটি অত্যন্ত জরুরি সীমারেখা আছে: **সব engineering কাজকে sub-linearly automate করা যাবে বা করা উচিত—এমনটা কখনো ভাববেন না।** উচ্চ বিচারবুদ্ধিনির্ভর কাজ—যেমন অস্পষ্ট ব্যবসায়িক লক্ষ্য পরিষ্কার করা, বড় architectural trade-off নিয়ে বিতর্ক করা, আগে কখনো দেখেনি এমন production outage-এর মূল কারণ খুঁজে বের করা, কিংবা জুনিয়র engineer-দের mentor করা—স্বভাবতই মানুষের চিন্তাশক্তি দাবি করে। ভালো engineering পুনরাবৃত্তিমূলক যান্ত্রিক কাজগুলো automate করে মূলত এজন্যই, যাতে দুর্লভ মানবিক বিচারবুদ্ধি সত্যিকারের গুরুত্বপূর্ণ সিদ্ধান্তের জন্য সংরক্ষিত থাকে।

এখান থেকে **timing** (সময়জ্ঞান) নিয়ে একটি তীক্ষ্ণ প্রশ্ন উঠে আসে। টিম যখন design আলোচনা, automated test এবং peer review-এ বিনিয়োগ করে, তখন একটি পরিবর্তনের জীবনচক্রে **কোন পর্যায়ে** ত্রুটি ধরা পড়া উচিত?

Software-এর প্রতিটি পরিবর্তন বাম থেকে ডানে কয়েকটি ধাপ পেরিয়ে এগোয়:

```text
Problem Framing & Design → Coding → Automated Checks & Tests → Code Review → Staging → Production
```

সহজ ভাষায় বললে, **Problem Framing & Design** হলো কোড টাইপ করার আগেই system-টির কী করা উচিত তা স্থির করা; **Coding** হলো সেই অনুযায়ী implementation লেখা; **Automated Checks & Tests** হলো এমন program যা প্রতিটি পরিবর্তনে স্বয়ংক্রিয়ভাবে চলে expected behavior যাচাই করে; **Code Review** হলো আরেকজন engineer-এর পরিবর্তনটি খুঁটিয়ে দেখা; **Staging** হলো system-এর একটি pre-production অনুলিপি যেখানে আসল user-দের সামনে যাওয়ার আগেই পরিবর্তন নিরাপদে পরখ করা হয় (Phase 24-এ আমরা এটা আরও গভীরে দেখব); আর **Production** হলো বাস্তব customer-দের সেবা দেওয়া live পরিবেশ।

এবার একটি সূক্ষ্ম bug-কে এই pipeline-এর দুই প্রান্ত দিয়ে চালিয়ে দেখুন: ধরুন, কোনো engineer একই shopping cart-এ এক customer যাতে একাধিকবার `SAVE10` প্রয়োগ করতে না পারেন, সেই বাধাটা দিতে ভুলে গেছেন।

- **Problem Framing**-এর সময়ই যদি কেউ প্রশ্ন করেন *"একই cart-এ কি একই promo code দুবার বসানো যাবে?"*—কিংবা একটি local **automated test**-এ এটি ধরা পড়ে—তাহলে ঠিক করতে লাগবে মাত্র পাঁচ মিনিট। পুরো মানসিক চিত্র তখনো লেখকের মাথায় তাজা, কোনো customer এটা দেখেননি, আর কোনো ভুল order data-ও তৈরি হয়নি।
- কিন্তু একই bug যদি Black Friday-তে **Production**-এ পৌঁছে যায়, তাহলে হাজার হাজার shopper একাধিকবার ছাড় নিয়ে checkout করে ফেলবেন। জরুরি ভিত্তিতে on-call engineer-দের জাগাতে হবে, চাপের মুখে checkout থামিয়ে patch করতে হবে, finance টিমকে ভুল order-গুলো অডিট করতে হবে, আর customer-দের আস্থাও নষ্ট হবে।

কোনো ত্রুটি বাইরে বেরিয়ে গেলে কতজন user, কতটুকু data, অর্থ বা downstream system ক্ষতিগ্রস্ত হয়, সেই ক্ষতির পরিসরকে বলা হয় **blast radius**। সচেতনভাবে defect detection, validation এবং design-পর্যালোচনাকে এই lifecycle-এর আরও বাম দিকে—অর্থাৎ শুরুর ধাপগুলোতে—নিয়ে আসাকেই বলা হয় **Shifting Left**, কারণ সেখানে context তাজা থাকে, blast radius প্রায় শূন্য হয়, আর জরুরি সমন্বয়ের ঝামেলাই তৈরি হয় না।

Hyrum's Law-এর মতোই, Shifting Left-কেও অনমনীয় সূত্র বানালে ভুল বোঝার ঝুঁকি থাকে। বহু প্রবন্ধে দাবি করা হয় যে production-এ একটি bug ঠিক করা design পর্যায়ে ধরার চেয়ে *"সব সময় exponentially (10× থেকে 100×) বেশি ব্যয়বহুল।"* এটিকে কোনো সর্বজনীন বৈজ্ঞানিক সূত্র নয়, বরং একটি **directional heuristic** (দিকনির্দেশক নীতি) হিসেবে দেখুন। প্রকৃত মেরামত খরচ অনেকটাই নির্ভর করে **reversibility** (কত সহজে আগের অবস্থায় ফেরা যায়), **blast radius**, এবং deployment-এর গতির ওপর: দ্রুত automated deployment থাকা কোনো checkout বাটনের সামান্য বানান ভুল production-এ ঠিক করতে হয়তো তিন মিনিট লাগে, অথচ ভুল design-এর কারণে বিকৃত হয়ে যাওয়া billing ledger ঠিক করতে মাসের পর মাস লেগে যেতে পারে। একইভাবে Shifting Left বাড়াবাড়ি পর্যায়েও যেতে পারে—ছোট ও সহজে উল্টে দেওয়া যায় এমন কোনো experiment-এর জন্য ১৫ পৃষ্ঠার formal specification আর তিনটি approval committee বসালে শেখার গতি থেমে যায় এবং engineering সময় নষ্ট হয়। **Blast radius এবং irreversibility-র সাথে সামঞ্জস্য রেখে Shift Left করুন।**

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-01-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. টিম ২ জন থেকে ৩০ জন engineer-এ বড় হলে ভেঙে পড়ে এমন একটি recurring backend policy-র উদাহরণ দিন, এবং এমন একটি engineering কাজের কথা বলুন যা কেবল automate করে sub-linearly scale করা যায় না।
> 2. কোনো ত্রুটি আগে ধরলে তা *সব সময়* exponentially কম খরুচে হয়—এই দাবিটি কেন পুরোপুরি সঠিক নয়? Release-এর পর ঠিক করা সস্তা এমন একটি ছোট সমস্যার সাথে, আগে না ধরলে বিশাল ক্ষতি হয় এমন একটি ত্রুটির তুলনা করুন।

এখানে একটু থেমে খেয়াল করুন, গত তিনটি অংশ মিলে আমরা এক দারুণ দ্বন্দ্বের সামনে এসে দাঁড়িয়েছি। স্পষ্ট contract, নিয়মিত dependency upgrade, automated tooling এবং আগেভাগে verification (Shifting Left)—সব কটিই বাস্তব production ব্যর্থতা ঠেকায়। **তাহলে একজন engineer কেন তাঁর লেখা প্রতিটি script ও feature-এ সর্বোচ্চ মাত্রার engineering rigor প্রয়োগ করবেন না?**

---

## সব জায়গায় সর্বোচ্চ Rigor প্রয়োগ করাও কেন খারাপ Engineering

উত্তরটা খুব পরিষ্কার: **প্রতিটি engineering সুরক্ষাব্যবস্থা সীমিত সম্পদ খরচ করে—আর cloud-এর মাসিক বিলের টাকা সেই খরচের মাত্র একটি অংশ।**

টিমগুলো প্রায়ই যে দুই ধরনের বাস্তব দ্বিধায় পড়ে, তা ভেবে দেখুন।

প্রথমত, ধরুন আমাদের store-এর backend চালানো ছোট startup-টি cloud ফি থেকে মাসে $300 বাঁচানোর জন্য managed PostgreSQL না নিয়ে নিজেরাই raw server-এ PostgreSQL চালানোর সিদ্ধান্ত নিল। ছয় মাস পর দেখা গেল একজন senior engineer মাসে ১৫ ঘণ্টা ব্যয় করছেন alert সামলাতে, backup পরীক্ষা করতে আর server update দিতে—যে সময়টা সরাসরি কেটে যাচ্ছে আসল checkout bug ঠিক করা আর customer feature তৈরি করা থেকে।

দ্বিতীয়ত, ধরুন আমাদের store-এর প্রথম `SAVE10` কোডটি তৈরি করতে গিয়ে একজন engineer তিন সপ্তাহ ব্যয় করে বিশাল এক generic campaign-rules engine বানালেন—*"যদি আগামী বছর marketing-এর ৫০০ রকম dynamic rule লাগে!"* ইতিমধ্যে store-টি মৌসুমি বিক্রির সময়সীমা হারাল, আর প্রতিযোগী কোম্পানি সেই customer-দের দখল করে নিল।

দুটি ক্ষেত্রেই টিম কেবল উপরিভাগের একটিমাত্র সংখ্যা দেখেছে—একটিতে মাসিক cloud বিল, অন্যটিতে ভবিষ্যতের তাত্ত্বিক সৌন্দর্য—অথচ আসল যেসব লুকানো মূল্য চুকাতে হলো তা এড়িয়ে গেছে। Winters et al. (2020) বাস্তব engineering সিদ্ধান্তের খরচকে **ছয়টি স্বতন্ত্র মাত্রায় (six cost dimensions)** ভাগ করেছেন:

| Cost Dimension | এটি আসলে কী মাপে | বাস্তব Backend উদাহরণ |
| :--- | :--- | :--- |
| **Financial** | সরাসরি অর্থ ব্যয় (cloud hosting, license, third-party service)। | সস্তা raw server ভাড়া নেওয়ার বদলে managed PostgreSQL database-এর জন্য টাকা দেওয়া। |
| **Resource** | Compute, memory, storage এবং network bandwidth-এর ব্যবহার। | কোনো customer-এর ৫০,০০০ পুরোনো order ছোট ছোট page-এ না পাঠিয়ে একসাথে পাঠালে database-এর কাজ ও network traffic বেড়ে যাওয়া। |
| **Personnel** | তৈরি, review, debug ও পরিচালনায় ব্যয় হওয়া engineer-দের সময়। | নিজে database সামলাতে গিয়ে backup, patch ও alert-এ মাসে ১৫ ঘণ্টা engineering সময় নষ্ট হওয়া। |
| **Transaction** | কোনো পরিবর্তন বা migration *সম্পন্ন করতে* যে শ্রম ও ঝুঁকি লাগে। | External app-এর ব্যবহৃত live API field-এর নাম বদলাতে আগাম সতর্কতা, সাময়িক compatibility ও সমন্বিত rollout করা। |
| **Opportunity** | এখানে সময় দেওয়ায় যে মূল্যবান product বা reliability কাজ *করা হলো না*। | ভাঙা checkout edge case ঠিক না করে অনুমাননির্ভর promo engine বানাতে তিন সপ্তাহ খরচ করা। |
| **Societal / User** | ব্যবহারকারীদের বাস্তব ক্ষতি (অর্থক্ষতি, ব্যক্তিগত data ফাঁস, downtime)। | Dependency security patch বা validation এড়িয়ে যাওয়ায় customer account ঝুঁকিতে পড়া বা ভুল চার্জ কাটা। |

এই ছয়টি খরচের উপাদান—টিমের আকার, traffic, আয় এবং আইনি বা regulatory ঝুঁকি—সময়ের সাথে বদলায় বলে **ভালো engineering সিদ্ধান্তেরও মেয়াদ (expiration date) থাকে।** ২০০ জন user-এর জন্য একটি সরল Node.js service আর একটি PostgreSQL database বেছে নেওয়া আজকের জন্য সাধারণত দারুণ বুদ্ধিমানের সিদ্ধান্ত। সেই সিদ্ধান্তকে চিরদিনের জন্য পবিত্র মেনে নেওয়া—কিংবা উল্টোদিকে, বড় টেক কোম্পানি করে বলে ২০০ জন user-এর জন্যই জটিল multi-service infrastructure বানানো—উভয়ই বাস্তব প্রেক্ষাপট অগ্রাহ্য করে। পরিমাপকৃত প্রমাণ (measured evidence) যখন দেখায় যে বাস্তব উপাদান বদলে গেছে, তখনই বিচক্ষণ engineer সিদ্ধান্ত পুনর্বিবেচনা করেন।

Rigor বা কঠোরতার মাত্রা কীভাবে পরিমাপ করতে হয় তা পরিষ্কার দেখতে, এবার ইচ্ছাকৃতভাবে আমাদের promo service-এর বাইরে বেরিয়ে সম্পূর্ণ বিপরীত দুটি backend কাজের তুলনা করুন:

- **যেখানে উচ্চ rigor অপরিহার্য:** ৪০০টি পেইং কোম্পানির সাথে যুক্ত একটি public business-to-business billing API, যার প্রত্যাশিত জীবনকাল পাঁচ বছর। এটি ভাঙলে partner-দের দুবার চার্জ কাটা পড়বে, invoicing pipeline ভেঙে যাবে এবং আইনি দায় তৈরি হবে। এখানে স্পষ্ট versioned contract, নির্দিষ্ট ordering, structured error code, automated test, নিয়মিত dependency upgrade এবং খুঁটিনাটি review—প্রতিটি ঘণ্টার বিনিয়োগ সার্থক। এখানে **under-engineering** মানেই বড় ধরনের ক্ষতি।
- **যেখানে ভারী rigor ক্ষতিকর বোঝা:** একজন engineer আজ বিকেলে একটি ঐতিহাসিক CSV ফাইল import করার আগে বাড়তি space পরিষ্কার করার জন্য নিজের ল্যাপটপে একবার চালানোর মতো ২৫ লাইনের একটি Node.js script লিখলেন। কোনো row ভুলভাবে format হলে test copy-তেই তিনি তা ধরে ১২ নম্বর লাইন ঠিক করে আবার চালিয়ে নেবেন। বিকেল ৫টায় ফেলে দেওয়া হবে এমন একটি script-এর জন্য দুদিন ধরে modular abstraction, automated deployment pipeline আর formal versioning বানালে ঝুঁকি তেমন না কমিয়েই personnel ও opportunity cost নষ্ট হয়। এখানে **over-engineering**-ও Software Engineering-এরই একটি ব্যর্থতা।

Engineering বিচারবুদ্ধি কখনোই *"সব জায়গায় সর্বোচ্চ প্রক্রিয়া চাপিয়ে দাও"* নয়। বরং **expected lifespan**, **scale**, এবং **blast radius**-এর সাথে সামঞ্জস্য রেখে rigor নির্ধারণ করাই আসল engineering।

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-01-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. ওপরের startup উদাহরণে (নিজে database সামলে মাসে $300 বাঁচানো, কিন্তু patch, backup ও alert-এ মাসে ~১৫ ঘণ্টা senior engineering সময় ব্যয় করা) কোন কোন cost dimension-এর মধ্যে trade-off হয়েছে, এবং এটি কেন সামগ্রিকভাবে লোকসান হতে পারে?
> 2. ২৫ লাইনের একটি একবার-ব্যবহারযোগ্য (throwaway) CSV script-কে over-engineer করা কেবল "বাড়তি সতর্কতা" নয়, বরং *Software Engineering*-এর ব্যর্থতা কেন?

---

## কোড লেখা পুরো শৃঙ্খলার একটিমাত্র অংশ—এবং এর পর আমরা কোথায় যাচ্ছি

একটু থেমে পেছনে তাকিয়ে দেখুন, আমাদের একটিমাত্র `SAVE10` feature আমাদের কোন কোন বিষয়ের মুখোমুখি দাঁড় করাল।

একটি order থেকে 10% ছাড় দেওয়ার দশ লাইনের JavaScript লিখতে লেগেছিল কয়েক মিনিট। অথচ প্রতিটি সত্যিকারের engineering চ্যালেঞ্জ ছিল সেই দশ লাইনের *চারপাশে*: সময়ের সাথে requirement কীভাবে বদলায়, dependency ও security patch কীভাবে সামলাতে হয়, Hyrum's Law-এর সামনে অঘোষিত API contract কীভাবে রক্ষা করতে হয়, মানুষের বিচারবুদ্ধি নষ্ট না করে টিমের কাজের পরিসর কীভাবে বাড়াতে হয়, blast radius অনুযায়ী কীভাবে Shift Left করতে হয়, আর পুরো lifespan জুড়ে ছয়টি পরস্পরবিরোধী খরচের ভারসাম্য কীভাবে রাখতে হয়। কোড কখনো বিশুদ্ধ কারিগরি শূন্যতায় চলে না; এটি user, সংগঠন, operation এবং অর্থনীতি দিয়ে ঘেরা বৃহত্তর **system boundaries**-এর ভেতরে কাজ করে।

এই বাস্তবতারই সরাসরি প্রতিফলন দেখা যায় IEEE Computer Society-র ***Guide to the Software Engineering Body of Knowledge* (SWEBOK Guide V4.0, 2024; updated as V4.0a in 2025)**-এ, যা Software Engineering-কে আনুষ্ঠানিকভাবে **১৮টি Knowledge Area**-তে সাজিয়েছে (চতুর্থ সংস্করণে *Software Architecture*, *Software Engineering Operations*, এবং *Software Security* নতুনভাবে যুক্ত হয়েছে)। শেখার সুবিধার জন্য নিচে সেই ১৮টি ক্ষেত্রকে পাঁচটি সহজবোধ্য teaching cluster-এ সাজানো হলো *(এগুলো বোঝার সুবিধার জন্য তৈরি গ্রুপ, SWEBOK-এর official category নয়)*:

- **কী বানাতে হবে তা বোঝা ও নকশা করা:** Software Requirements, Software Architecture *(V4)*, Software Design, Software Engineering Models and Methods.
- **কোড লেখা (১৮টির মধ্যে মাত্র ১টি):** **Software Construction**.
- **যাচাই ও সুরক্ষা দেওয়া:** Software Testing, Software Quality, Software Security *(V4)*.
- **সময়ের সাথে চালানো ও টিকিয়ে রাখা:** Software Engineering Operations *(V4)*, Software Maintenance, Software Configuration Management.
- **সমন্বয়, পরিমাপ ও মৌলিক ভিত্তি:** Software Engineering Management, Software Engineering Process, Software Engineering Economics, Software Engineering Professional Practice, Computing Foundations, Mathematical Foundations, Engineering Foundations.

মাঝের বুলেটটির দিকে তাকান: **কোড লেখা (*Software Construction*) ১৮টি Knowledge Area-র মাত্র একটি।**

এই একটি তথ্যই আধুনিক backend engineering-এর মূল বাস্তবতা স্পষ্ট করে দেয়। আজকের **AI coding assistants** কয়েক সেকেন্ডেই syntax তৈরি করে দিতে পারে—ফলে *Software Construction*-এর টাইপিং খরচ নাটকীয়ভাবে কমে গেছে। কিন্তু দ্রুত কোড তৈরি হয়ে যাওয়া বাকি ১৭টি ক্ষেত্রের সমাধান করে না; বরং সেগুলোকে আগের চেয়ে *আরও বেশি নির্ণায়ক* করে তোলে। দশ সেকেন্ডে ৪০০ লাইন promo-code logic তৈরি হলে ঝুঁকিও তত দ্রুত বাড়ে, যদি না একজন engineer যাচাই করতে পারেন যে কোডটি আসল সমস্যার সমাধান করছে কি না, ভঙ্গুর কোনো contract ফাঁস করছে কি না, আগামী বছরও maintainable থাকবে কি না, এবং বাস্তব security ও cost সীমাবদ্ধতা মানছে কি না। যে engineer প্রথম দিনের পর AI-তৈরি কোডের কী হবে তা বিচার করতে পারেন না, তিনি তখনো কেবল Programming স্তরেই আটকে আছেন (Block 7-এ আমরা AI-augmented engineering গভীরে শিখব)।

আর সবচেয়ে গুরুত্বপূর্ণ বিষয়—ভালো করে খেয়াল করুন, আমাদের `SAVE10` যাত্রার প্রায় প্রতিটি মাথাব্যথার শুরু হয়েছিল কোথা থেকে? একটি কোড expire হলে কী হবে, দুবার ব্যবহার করলে কী হবে, refund হলে কী হবে, বা অন্য discount-এর সাথে ধাক্কা লাগলে কী হবে—এই প্রশ্নগুলো করার আগেই আমরা `SAVE10`-এর কোড লিখতে বসে পড়েছিলাম। একজন engineer কোনো contract ডিজাইন করার, trade-off বেছে নেওয়ার, বা এক লাইন কোড লেখার আগেই মূল সমস্যাটাকেই জেরা করেন—যাতে দিক পরিবর্তন করা সস্তা থাকা অবস্থাতেই লুকানো অনুমান, অস্পষ্টতা আর বাদ পড়া নিয়মগুলো বেরিয়ে আসে।

আর সেই কারণেই এরপর আসছে **Block 2: Understanding the Problem Before Coding**।

---

## Study Plan, Learning Objectives & Mandatory Reading

> **Continuity & Ownership Notes:** Phase 1 Diagnostic ইতিমধ্যেই `completed`—এটি পুনরাবৃত্তি করবেন না। আপনার সব inline check, exercise এবং closed-book assessment-এর উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-01-notes.md` ফাইলে লিখুন। এই lesson-টি পড়া মানেই Block 1 `completed` বা `approved` হয়ে যাওয়া নয়, এবং AI-এর প্রস্তুতির সময় আপনার study hours-এ গণনা হয় না।

### Target 3-Hour (180-Minute) Study Budget

| Stage | কাজ (Activity) | নির্ধারিত সময় (Target Time) |
| :---: | :--- | :---: |
| **1** | ওপরের guided lesson পড়া + `notes/block-01-notes.md`-এ Pause & Checks 1–3 সম্পন্ন করা | 65 min |
| **2** | Complementary Mandatory Reading (*SWE at Google* Ch. 1 + SWEBOK V4.0 18-KA skim) | 35 min |
| **3** | `notes/block-01-notes.md`-এ স্বাধীনভাবে Practical Reasoning Exercises 1–3 সম্পন্ন করা | 55 min |
| **4** | বই/নোট বন্ধ রেখে Mastery Assessment Questions 1–5 লেখা → ChatGPT-র কাছে জমা দেওয়া | 25 min |
| **Total** | — | **180 min (3 h)** |

### Block 1 Learning Objectives *(Primary: Outcome 1; Early Bridge: Outcome 10)*

- **LO 1.1 (`Working Understanding`):** কার্যভিত্তিক মানদণ্ডে **programming**, **software development**, এবং **software engineering**-এর পার্থক্য নির্ণয় করা—এটা মাথায় রেখে যে এই তিন-ভাগ বিভাজন কোনো সর্বজনীন ISO taxonomy নয়, বরং আমাদের course working distinction।
- **LO 1.2 (`Working Understanding`):** একটি system-এর lifespan জুড়ে **time and change**, **scale and efficiency**, এবং **trade-offs and costs** কীভাবে engineering সিদ্ধান্ত বদলে দেয় তা বিশ্লেষণ করা।
- **LO 1.3 (`Working Understanding`):** দীর্ঘমেয়াদি backend system-এ **Hyrum's Law** এবং **shifting left**-এর mechanism, ফলাফল ও সীমাবদ্ধতা ব্যাখ্যা করা *(Exercise 1-এর মাধ্যমে সুদৃঢ় হবে)*।
- **LO 1.4 (`Awareness`):** **IEEE SWEBOK Guide V4.0 (2024; updated as V4.0a in 2025)-এর ১৮টি Knowledge Area**-র প্রেক্ষাপটে code construction-এর অবস্থান বোঝা এবং দ্রুত syntax লেখাই কেন software engineering নয় তা ব্যাখ্যা করা।

### Complementary Authoritative Reading

- **Mandatory (~30 min — Guided Lesson শেষ করার পর পড়ুন):** Titus Winters, Tom Manshreck, Hyrum Wright, *Software Engineering at Google* (O'Reilly, 2020; free HTML edition) — [Chapter 1: What Is Software Engineering?](https://abseil.io/resources/swe-book/html/ch01.html)। পুরো অধ্যায়টি পড়ুন (*Time and Change* incl. Hyrum's Law and hash ordering; *Scale and Efficiency* incl. policies that don't/do scale, compiler upgrade, Shifting Left; *Trade-offs and Costs* incl. markers, inputs to decision making, distributed builds, deciding between time and scale, revisiting decisions; *Software Engineering Versus Programming*; *Conclusion*; এবং *TL;DRs*)। ওপরে গড়ে ওঠা mental model-গুলোকে Google-scale case study দিয়ে যাচাই করে নিন।
- **Reference Skim (~5 min):** IEEE Computer Society, *Guide to the Software Engineering Body of Knowledge (SWEBOK Guide V4.0, 2024; updated as V4.0a in 2025)* — [Official SWEBOK Portal](https://www.computer.org/education/bodies-of-knowledge/software-engineering-swebok) (শুধু Introduction এবং 18 Knowledge Areas-এর তালিকা চোখ বুলিয়ে দেখুন; Phase 1-এ প্রতিটি অধ্যায় পড়ার প্রয়োজন নেই)।
- **Optional Supplementary (৩ ঘণ্টার বাজেটের বাইরে):** Mary Shaw, *"Prospects for an Engineering Discipline of Software"*, *IEEE Software*, Vol. 7, No. 6, pp. 15–24 (Nov 1990) — [CMU Figshare](https://doi.org/10.1184/R1/6608717) | [IEEE DOI](https://doi.org/10.1109/52.60586).

---

## Practical Reasoning Exercises (~55 Minutes Total)

> **Ownership Rule:** সব উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-01-notes.md` ফাইলে লিখুন। AI-কে দিয়ে উত্তর খসড়া করাবেন না।

### Exercise 1: Hyrum's Law, Implicit Contracts & Proportional Shifting Left (~20 minutes)

**দৃশ্যপট (Scenario):**
আপনার টিম Node.js + PostgreSQL দিয়ে তৈরি একটি backend পরিচালনা করে, যেখানে `GET /api/orders` নামে একটি **HTTP endpoint** আছে *(যা কোনো customer-এর order-গুলো **JSON**—অর্থাৎ প্রমিত structured text data—আকারে ফেরত দেয়)*। API documentation-এ শুধু লেখা আছে: *"Returns a list of the authenticated user's orders."* ভেতরের **SQL** query-টি `ORDER BY` clause ছাড়াই মিলে যাওয়া order-গুলো তুলে আনে *(SQL এবং PostgreSQL-এ `ORDER BY` ছাড়া কোনো `SELECT` query কোন ক্রমে row ফেরত দেবে তার কোনো guarantee দেয় না)*।

১৮ মাস ধরে PostgreSQL ঘটনাক্রমে তৈরি হওয়ার ক্রমেই row ফেরত দিচ্ছিল; ফলে আপনার mobile app এবং দুটি enterprise partner নীরবে ধরে নিয়েছে যে `orders[0]` হলো সবচেয়ে পুরোনো order আর `orders[orders.length - 1]` হলো সবচেয়ে নতুন order। পরে table বড় হয়ে যাওয়ায় ধীর lookup দ্রুত করতে একজন engineer একটি **database index** যোগ করেন *(এমন একটি বাড়তি lookup structure যা database-কে মিলে যাওয়া row দ্রুত খুঁজে পেতে সাহায্য করে—আমরা Phase 10-এ indexing-এর ভেতরের কৌশল পড়ব)*। Index যোগ করার পর PostgreSQL ভিন্ন একটি **query plan** বেছে নিতে পারে এবং ভিন্ন observed order-এ row ফেরত দিতে পারে—এই ঘটনায় ঠিক সেটাই ঘটেছে। Mobile shopper-রা এখন তাঁদের স্ক্রিনের একদম ওপরে পুরোনো cancelled order দেখতে পাচ্ছেন, আর একজন partner-এর billing script ভুল invoice টেনে নিচ্ছে—PostgreSQL কোনো প্রতিশ্রুতি ভেঙেছে বলে নয়, বরং query-টি কখনোই কোনো নির্দিষ্ট order-এর guarantee দেয়নি বলে।

**কাজ (Tasks):**
1. **Cause-and-Effect Diagnosis:** প্রথম দিনের ওই বাদ পড়া বিষয়, Hyrum's Law, এবং ৫৪০তম দিনের outage-কে যুক্ত করে পুরো কার্যকারণ শৃঙ্খল (chain of events) বিশ্লেষণ করুন।
2. **Programmer vs. Software Engineer Response:** কেবল তাৎক্ষণিক কাজ-কেন্দ্রিক একজন programmer-এর প্রতিক্রিয়া (*"আমরা তো doc-এ কোনো order লিখিনি; client-দের নিজেদের কোড ঠিক করতে হবে"*) এবং একজন software engineer-এর তাৎক্ষণিক সমাধান (immediate mitigation) ও দীর্ঘমেয়াদি contract fix-এর তুলনা করুন।
3. **Proportional Shift Left:** আগের lifecycle-এর এমন **দুটি** checkpoint চিহ্নিত করুন যেখানে এই ঝুঁকিটি কম খরচে ধরা যেত—এবং প্রাথমিক minimal product-এর জন্য অপ্রয়োজনীয় বোঝা হতো এমন **একটি** ভারী সুরক্ষাব্যবস্থার (heavyweight safeguard) নাম বলুন।

---

### Exercise 2: Calibrating Engineering Rigor Across Four Backend Artifacts (~20 minutes)

*(২০ মিনিটকে কঠিন সময়সীমা নয়, একটি rough target হিসেবে ধরুন—তাড়াহুড়োর বদলে স্পষ্ট engineering যুক্তি দেওয়াকে অগ্রাধিকার দিন।)*

নিচের চারটি ভিন্ন backend artifact মূল্যায়ন করুন:
- **Artifact A:** আজ একবারের জন্য একটি পুরোনো CSV ফাইল import করার আগে বাড়তি space পরিষ্কার করতে ল্যাপটপে চালানো ৩০ লাইনের একটি Node.js script।
- **Artifact B:** প্রতিদিন সকালে ব্যর্থ shipment-গুলোর সারসংক্ষেপ তিনজন operations কর্মীকে email করতে ব্যবহৃত একটি internal nightly **cron job** *(নির্দিষ্ট সময়ে স্বয়ংক্রিয়ভাবে চলা একটি scheduled background task)*।
- **Artifact C:** ৪০০টি পেইং external কোম্পানির সাথে integrated একটি public business-to-business billing **webhook** *(কোনো ঘটনা ঘটলে partner system-এ স্বয়ংক্রিয়ভাবে পাঠানো HTTP notification)* এবং invoice API।
- **Artifact D:** প্রতিটি backend request-এ ডাকা হয় এমন কোম্পানির কেন্দ্রীয় **authentication** এবং **session-validation** *(আগত প্রতিটি request বৈধভাবে logged-in কোনো user-এর কি না তা যাচাই করা)* service।

**কাজ (Tasks):**
**প্রতিটি** artifact (A–D)-এর জন্য:
1. এর প্রত্যাশিত **lifespan** এবং এটি ব্যর্থ হলে **blast radius** কতটা হতে পারে তা অনুমান করুন।
2. এর জন্য উপযুক্ত engineering rigor-এর মাত্রা নির্ধারণ করুন (testing, contracts/versioning, upgrade policy, review depth)।
3. ঠিক ওই artifact-টির জন্য **under-engineering-এর একটি বাস্তব ব্যর্থতা** এবং **over-engineering-এর একটি বাস্তব ব্যর্থতা** উল্লেখ করুন।

---

### Exercise 3: Multi-Dimensional Cost & Sustainability Analysis (~15 minutes)

**দৃশ্যপট (Scenario):**
৩ জন engineer-এর একটি backend টিম এমন একটি Node.js version-এ production API চালাচ্ছে, যা ৪ মাস পর End-of-Life (EOL) হয়ে যাবে। এখন upgrade করতে লাগবে প্রায় ১.৫ engineering দিন (dependencies যাচাই, automated test চালানো এবং **staged rollout**—অর্থাৎ সবার কাছে ছাড়ার আগে অল্প কিছু traffic-এ update-টি পরখ করা)। একজন manager প্রস্তাব দিলেন: *"আগামী দুই বছর upgrade বাদ রাখো, তাহলে feature দ্রুত ship করা যাবে—এখন তো কিছুই ভাঙেনি।"*

**কাজ (Tasks):**
1. Winters et al.-এর ছয়টি cost dimension (financial, resource, personnel, transaction, opportunity, societal/user)-এর **কমপক্ষে চারটি** দিয়ে manager-এর প্রস্তাবটি বিশ্লেষণ করুন।
2. দুই বছর upgrade পিছিয়ে দেওয়ার ফলে **software sustainability**-র ওপর কী cause-and-effect প্রভাব পড়বে তা ব্যাখ্যা করুন।
3. এমন **একটি বাস্তব পরিস্থিতি** বর্ণনা করুন যেখানে একটি ছোট ও নির্দিষ্ট সময়ের জন্য (যেমন ৩ সপ্তাহ) upgrade পেছানো যুক্তিসঙ্গত engineering trade-off *হতে পারে*—এবং সেটি যাতে অনির্দিষ্টকালের বিলম্বে পরিণত না হয়, তার জন্য কী সুনির্দিষ্ট শর্ত বা অঙ্গীকার (condition/commitment) দরকার তা উল্লেখ করুন।

---

## Closed-Book Mastery Assessment (~25 Minutes)

এই lesson এবং বাইরের সব tab বন্ধ করে দিন। নিচের পাঁচটি প্রশ্নের স্পষ্ট ও মৌলিক উত্তর `phases/phase-01/notes/block-01-notes.md` ফাইলে লিখুন, এবং মূল্যায়নের জন্য **ChatGPT (Curriculum Owner & Instructor)**-এর কাছে জমা দিন:

1. **(LO 1.1 / Outcome 1)** একটি বাস্তব backend উদাহরণ ব্যবহার করে **programming**, **software development**, এবং **software engineering**-এর কার্যভিত্তিক পার্থক্য ব্যাখ্যা করুন। এই তিন-ভাগ বিভাজনকে আমরা কেন সর্বজনীন ISO taxonomy না বলে course working distinction হিসেবে ধরি?
2. **(LO 1.2 / Outcome 1)** **time and change**, **scale and efficiency**, এবং **trade-offs and costs**—প্রতিটি কীভাবে এমন সব সিদ্ধান্ত নিতে বাধ্য করে যা একবার-ব্যবহারযোগ্য (single-use) program-এ আসে না?
3. **(LO 1.3)** **Hyrum's Law** কীভাবে mechanism-এর দিক থেকে কাজ করে, **shifting left** কেন অনেক (যদিও সর্বজনীনভাবে সব নয়) defect-এর খরচ কমায়, এবং এই দুটির বাস্তব সীমা কোথায়?
4. **(LO 1.2 / Outcome 10)** এই প্রোগ্রামে ব্যবহৃত অর্থে **software sustainability**-র সংজ্ঞা দিন এবং ব্যাখ্যা করুন কেন *"যে কার্যকর কোডে কেউ হাত দিতে সাহস করে না"* তা বাস্তব পরিণতিসহ একটি engineering ব্যর্থতা।
5. **(LO 1.4)** SWEBOK V4.0-এর বিস্তৃতির (18 Knowledge Areas) সূত্র ধরে ব্যাখ্যা করুন, নিজে বা AI দিয়ে দ্রুত কোড তৈরি করলেই কেন software engineering-এর সমস্যা সমাধান হয় না।

### Block 1 Readiness Checklist (Block 2 শুরুর আগে)
- [ ] Guided lesson পড়েছেন এবং Pause & Checks 1–3 সম্পন্ন করেছেন (পরে Self-Check Guidance মিলিয়ে দেখেছেন)।
- [ ] Complementary mandatory reading (*Software Engineering at Google* Ch. 1) এবং ৫ মিনিটের SWEBOK V4.0 KA skim সম্পন্ন করেছেন।
- [ ] `phases/phase-01/notes/block-01-notes.md` ফাইলে স্বাধীনভাবে Exercises 1–3 লিখেছেন।
- [ ] বই/নোট বন্ধ রেখে স্বাধীনভাবে Mastery Assessment Questions 1–5 লিখেছেন।
- [ ] Block 2 শুরু করার আগে Block 1-এর কাজ feedback ও অনুমোদনের জন্য ChatGPT-র কাছে জমা দিয়েছেন।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`notes/block-01-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের নির্দেশিকা দেখুন। Practical Reasoning Exercises বা Closed-Book Mastery Assessment-এর কোনো উত্তর এখানে দেওয়া হয়নি—সেগুলো ChatGPT review-এর জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* আপনার repository-তে হাত না দিলেও বাইরের runtime, library, security vulnerability, data-র পরিমাণ এবং ব্যবসায়িক চাহিদা বদলাতে থাকে। সেই বাইরের পরিবর্তনগুলো এলে টিম যদি service-টিকে নিরাপদে পরিবর্তন বা upgrade করতে না পারে, তবে আজ একটিও bug ticket না থাকলেও system-টি unsustainable।
  2. *শক্তিশালী উত্তরে থাকবে:* দীর্ঘমেয়াদি login service-এ dependency update এড়িয়ে গেলে একাধিক version জুড়ে CVE এবং breaking change জমতে থাকে, যতক্ষণ না বাধ্যতামূলক upgrade একটি উচ্চ-ঝুঁকির জরুরি সংকটে পরিণত হয়—অথবা user-রা অরক্ষিত থেকে যান। কার্যকর কোডে হাত না দেওয়াটা যুক্তিসঙ্গত হতে পারে কেবল সত্যিকারের স্বল্পমেয়াদি ও বিচ্ছিন্ন (isolated) কোনো artifact-এ যা EOL বা security ঝুঁকি আসার আগেই বাদ যাবে, অথবা এমন cosmetic rewrite এড়ানোর ক্ষেত্রে যা lifecycle মূল্য না বাড়িয়ে উল্টো ঝুঁকি বাড়ায়।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* লোকসংখ্যা বাড়লে manual cross-team সমন্বয়ের নিয়ম (যেমন shared helper বদলালে প্রতিটি টিমকে আলাদা করে message দেওয়া বা প্রত্যেকের ল্যাপটপে manually check চালানো) মানুষের সক্ষমতা ফুরিয়ে ফেলে। অন্যদিকে অস্পষ্ট সমস্যা বিশ্লেষণ, architectural trade-off বিচার, অপরিচিত outage-এর তদন্ত এবং mentoring-এর মতো উচ্চ বিচারবুদ্ধিনির্ভর কাজগুলোকে কেবল automate করে sub-linearly scale করা যায় না।
  2. *শক্তিশালী উত্তরে থাকবে:* প্রকৃত মেরামত খরচ সর্বজনীন কোনো exponential সূত্রের বদলে blast radius, reversibility এবং deployment-এর গতির ওপর নির্ভর করে। দ্রুত automated deployment থাকলে ছোট কোনো wording typo release-এর পর ঠিক করা সস্তা; কিন্তু ভুল API বা schema design-এর কারণে বিকৃত billing record ঠিক করা বা ৪০০টি external integration ভেঙে যাওয়া release-এর পর সামলানো design পর্যায়ের চেয়ে বহুগুণ ব্যয়বহুল।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* Startup-টি সরাসরি **financial** খরচ ($300/month) কমিয়েছে, কিন্তু এর বিনিময়ে **personnel** খরচ (মাসে ১৫ ঘণ্টা senior engineering সময়) এবং **opportunity** খরচ (অসম্পূর্ণ checkout bug ও feature) অনেক বাড়িয়ে ফেলেছে—পাশাপাশি manual patching বা backup-এ ভুল হলে data হারানো বা ফাঁস হওয়ার **societal/user** খরচের ঝুঁকিও নিয়েছে।
  2. *শক্তিশালী উত্তরে থাকবে:* Software Engineering বাস্তব সীমাবদ্ধতার মধ্যে lifespan, scale ও blast radius অনুযায়ী rigor নির্ধারণ করে। একবার চালিয়ে ফেলে দেওয়া হবে এমন script-এর পেছনে দিনব্যাপী জটিল abstraction, pipeline আর versioning বানালে বাস্তব ঝুঁকি না কমিয়েই personnel ও opportunity cost নষ্ট হয়।
