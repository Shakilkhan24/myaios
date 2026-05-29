# 🤖 Claude Code দিয়ে আপনার নিজস্ব AI Operating System তৈরি করুন
### একটি সম্পূর্ণ বাংলা গাইড — Nate Herk-এর ২+ ঘণ্টার কোর্স অবলম্বনে

> **ভিডিও সোর্স:** [Build & Sell Claude Code Operating Systems](https://www.youtube.com/watch?v=bCljOfCH8Ms) by Nate Herk | AI Automation  
> **মোট সময়:** ~৩–৫ ঘণ্টা (প্রথমবার সেটআপ)  
> **স্তর:** একদম শুরু থেকে বিশেষজ্ঞ পর্যন্ত

---

## 📋 সূচিপত্র

1. [AI OS কী এবং কেন দরকার?](#-অধ্যায়-১--ai-os-কী-এবং-কেন-দরকার)
2. [তিনটি M — AI-এর মূল দর্শন](#-অধ্যায়-২--তিনটি-m--ai-এর-মূল-দর্শন)
3. [চারটি C — AIOS-এর কাঠামো](#-অধ্যায়-৩--চারটি-c--aios-এর-কাঠামো)
4. [আপনার টুল ম্যাপিং করুন](#-অধ্যায়-৪--আপনার-টুল-ম্যাপিং-করুন)
5. [পূর্বশর্ত ও ইন্সটলেশন](#-অধ্যায়-৫--পূর্বশর্ত-ও-ইন্সটলেশন)
6. [Repo Clone ও VS Code সেটআপ](#-অধ্যায়-৬--repo-clone-ও-vs-code-সেটআপ)
7. [Onboarding Skill — প্রথম কথোপকথন](#-অধ্যায়-৭--onboarding-skill--প্রথম-কথোপকথন)
8. [ClickUp সংযোগ করা](#-অধ্যায়-৮--clickup-সংযোগ-করা)
9. [Audit ও Level Up Skills](#-অধ্যায়-৯--audit-ও-level-up-skills)
10. [Google Workspace CLI সেটআপ](#-অধ্যায়-১০--google-workspace-cli-সেটআপ)
11. [Capabilities — Skills তৈরি করা](#-অধ্যায়-১১--capabilities--skills-তৈরি-করা)
12. [Live Skill Build — হাতে কলমে](#-অধ্যায়-১২--live-skill-build--হাতে-কলমে)
13. [Cadence — Cloud Routines](#-অধ্যায়-১৩--cadence--cloud-routines)
14. [Loop ও Reminders](#-অধ্যায়-১৪--loop-ও-reminders)
15. [Karpathy LLM Wiki — জ্ঞান ভান্ডার তৈরি](#-অধ্যায়-১৫--karpathy-llm-wiki--জ্ঞান-ভান্ডার-তৈরি)
16. [Dashboards with Artifacts](#-অধ্যায়-১৬--dashboards-with-artifacts)
17. [Daily Loop ও সাফল্যের মানদণ্ড](#-অধ্যায়-১৭--daily-loop-ও-সাফল্যের-মানদণ্ড)
18. [সাধারণ সমস্যা ও সমাধান](#-অধ্যায়-১৮--সাধারণ-সমস্যা-ও-সমাধান)
19. [পরবর্তী পদক্ষেপ](#-অধ্যায়-১৯--পরবর্তী-পদক্ষেপ)

---

## 🧠 অধ্যায় ১ — AI OS কী এবং কেন দরকার?

### সাধারণ Chatbot বনাম AI Operating System

অনেকেই Claude বা ChatGPT-কে একটি "খুব ভালো ক্যালকুলেটর" হিসেবে ব্যবহার করেন — কিছু জিজ্ঞেস করুন, উত্তর নিন, বন্ধ করুন। কিন্তু এটি মূল্যের মাত্র ৫% ব্যবহার।

| বৈশিষ্ট্য | সাধারণ Chatbot | AI Operating System |
|---|---|---|
| মেমোরি | নেই (প্রতি সেশনে নতুন) | আছে (আপনার সম্পর্কে সব জানে) |
| ডেটা অ্যাক্সেস | নেই | আপনার ক্যালেন্ডার, ইমেইল, টাস্ক দেখতে পারে |
| স্বয়ংক্রিয়তা | নেই | ঘুমের মধ্যেও কাজ করে |
| ব্যক্তিগততা | অপরিচিত | ৬ মাসের পুরনো চিফ অফ স্টাফের মতো |

**বাস্তব উদাহরণ:**
> আপনি Claude-কে জিজ্ঞেস করলেন: *"এই সপ্তাহে আমার কী করা উচিত?"*
>
> - **সাধারণ Claude:** জেনেরিক পরামর্শ দেবে কারণ সে আপনাকে চেনে না।
> - **আপনার AIOS:** আপনার `priorities.md` ফাইল পড়বে, ClickUp থেকে open tasks চেক করবে, Google Calendar স্ক্যান করবে এবং তারপর **আপনার** জন্য কাস্টম উত্তর দেবে।

### AI OS-এর মূল ধারণা

একটি Operating System (যেমন Windows, Mac) যেভাবে আপনার কম্পিউটারের সব অ্যাপ ও রিসোর্স ম্যানেজ করে, ঠিক তেমনি আপনার **AI OS** আপনার সমস্ত টুল, ডেটা এবং ওয়ার্কফ্লো ম্যানেজ করবে — এবং Claude Code হবে এর কেন্দ্র।

---

## 🔵 অধ্যায় ২ — তিনটি M: AI-এর মূল দর্শন

*(ভিডিওর timestamp: ৩:৩০)*

Nate Herk বলেন যে AI সঠিকভাবে ব্যবহার করতে হলে আপনাকে তিনটি M বুঝতে হবে:

### M1 — Mindset (মানসিকতা)

AI কে "টুল" হিসেবে নয়, **দলের সদস্য** হিসেবে ভাবুন। আপনার AI OS হবে আপনার ডিজিটাল চিফ অফ স্টাফ।

- AI কী করতে পারে না, সেটা নিয়ে হতাশ হবেন না।
- প্রতিটি ব্যর্থতাকে skill উন্নত করার সুযোগ হিসেবে দেখুন।
- সিস্টেম তৈরিতে সময় দিন, শুধু প্রশ্ন করতে নয়।

### M2 — Mapping (ম্যাপিং)

আপনার বিজনেস বা কাজের সব প্রক্রিয়া ম্যাপ করুন:

- কোথায় রাজস্ব আসে?
- কোথায় টাস্ক থাকে?
- কোথায় যোগাযোগ হয়?
- কোন কাজে সবচেয়ে বেশি সময় নষ্ট হয়?

### M3 — Mechanics (মেকানিক্স)

সিস্টেম কীভাবে কাজ করে তা বোঝা:

- Skills কীভাবে তৈরি হয়?
- Routines কীভাবে চলে?
- Context কীভাবে ব্যবহার হয়?

---

## 🟢 অধ্যায় ৩ — চারটি C: AIOS-এর কাঠামো

*(ভিডিওর timestamp: ১১:৩০)*

এটি পুরো সিস্টেমের মেরুদণ্ড। **এই ক্রম অনুসরণ করতেই হবে** — কোনো ধাপ এড়ানো যাবে না।

```
Context → Connections → Capabilities → Cadence
(প্রসঙ্গ)   (সংযোগ)      (সক্ষমতা)      (তাল)
```

### C1 — Context (প্রসঙ্গ)
AI আপনাকে কতটুকু চেনে। আপনার পরিচয়, ব্যবসা, লক্ষ্য, লেখার ধরন — সব।

### C2 — Connections (সংযোগ)
AI কোন ডেটা দেখতে পারে। আপনার ক্যালেন্ডার, ইমেইল, টাস্ক ম্যানেজমেন্ট টুল।

### C3 — Capabilities (সক্ষমতা)
AI কী কী কাজ করতে পারে। Skills হলো এর বাস্তব রূপ।

### C4 — Cadence (তাল)
AI কখন নিজে থেকে কাজ করে। Scheduled routines — আপনার ছাড়াও।

> **⚠️ গুরুত্বপূর্ণ সতর্কতা:** Cadence ছাড়া Context অর্থহীন নয়, কিন্তু Context ছাড়া Cadence একটি এলোমেলো অটোমেশন মাত্র। সঠিক ক্রমে করুন।

---

## 🗺️ অধ্যায় ৪ — আপনার টুল ম্যাপিং করুন

*(ভিডিওর timestamp: ১৫:০০)*

শুরু করার আগে ১০ মিনিট নিন এবং নিচের প্রশ্নগুলোর উত্তর লিখুন:

### ধাপ ৪.১ — তিনটি মূল এলাকা চিহ্নিত করুন

```
১. রাজস্ব/আয় কোথায় ট্র্যাক হয়?
   উদাহরণ: Stripe, বা Excel শিট?
   আপনার উত্তর: _______________________

২. টাস্ক/প্রজেক্ট কোথায় থাকে?
   উদাহরণ: ClickUp, Notion, Asana, Trello?
   আপনার উত্তর: _______________________

৩. যোগাযোগ কোথায় হয়?
   উদাহরণ: Gmail, Slack, WhatsApp?
   আপনার উত্তর: _______________________
```

### ধাপ ৪.২ — সময় নষ্টের জায়গা খুঁজুন

প্রতি সপ্তাহে আপনি কোন কাজগুলো বারবার করেন যা AI করতে পারে?

```
□ ইমেইল ড্রাফট করা
□ মিটিং সারসংক্ষেপ তৈরি করা
□ টাস্ক স্ট্যাটাস আপডেট করা
□ রিপোর্ট তৈরি করা
□ সোশ্যাল মিডিয়া পোস্ট লেখা
□ ক্লায়েন্ট আপডেট পাঠানো
□ অন্যান্য: _______________________
```

এই তালিকাটি পরে skills তৈরিতে কাজে আসবে।

---

## ⚙️ অধ্যায় ৫ — পূর্বশর্ত ও ইন্সটলেশন

*(ভিডিওর timestamp: ২০:৩০ এর আগে)*

### প্রয়োজনীয় অ্যাকাউন্ট

| টুল | পরিকল্পনা | খরচ |
|---|---|---|
| Claude | Pro বা Max | Pro: $২০/মাস, Max: $২০০/মাস |
| GitHub | Free | বিনামূল্যে |
| Visual Studio Code | Free | বিনামূল্যে |
| Google Account | Free | বিনামূল্যে (ঐচ্ছিক) |
| ClickUp | Free tier | বিনামূল্যে (ঐচ্ছিক) |

> **📌 নোট:** Remote Routines ব্যবহার করতে হলে কমপক্ষে Claude Pro লাগবে।
> - Pro Plan: দিনে ৫টি remote routine
> - Max Plan: দিনে ১৫টি remote routine

### ধাপ ৫.১ — Visual Studio Code ইন্সটল করুন

1. [https://code.visualstudio.com/](https://code.visualstudio.com/) এ যান
2. আপনার অপারেটিং সিস্টেম অনুযায়ী ডাউনলোড করুন (Windows/Mac/Linux)
3. ইন্সটল করুন এবং খুলুন

### ধাপ ৫.২ — Claude Code Extension ইন্সটল করুন

1. VS Code খুলুন
2. বাম দিকে Extensions আইকনে ক্লিক করুন (বর্গক্ষেত্রের মতো আইকন)
3. সার্চ বারে লিখুন: `Claude Code`
4. Anthropic-এর অফিশিয়াল Extension খুঁজুন
5. `Install` বাটনে ক্লিক করুন

### ধাপ ৫.৩ — GitHub অ্যাকাউন্ট তৈরি করুন (না থাকলে)

1. [https://github.com](https://github.com) এ যান
2. `Sign up` ক্লিক করুন
3. ইমেইল, পাসওয়ার্ড, ইউজারনেম দিন
4. অ্যাকাউন্ট ভেরিফাই করুন

### ধাপ ৫.৪ — Git ইন্সটল করুন (না থাকলে)

**Windows:**
```
https://git-scm.com/download/win
```
ডাউনলোড করে ইন্সটল করুন।

**Mac:**
Terminal খুলুন এবং লিখুন:
```bash
xcode-select --install
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update && sudo apt install git
```

---

## 📁 অধ্যায় ৬ — Repo Clone ও VS Code সেটআপ

*(ভিডিওর timestamp: ২০:৩০)*

### ধাপ ৬.১ — GitHub-এ Private Repo তৈরি করুন

1. GitHub এ লগইন করুন
2. ডানদিকে উপরে `+` আইকনে ক্লিক করুন → `New repository`
3. Repository name দিন: `my-aios` (বা আপনার পছন্দের নাম)
4. **`Private`** সিলেক্ট করুন (অবশ্যই, কারণ এতে আপনার ব্যক্তিগত তথ্য থাকবে)
5. `Add a README file` চেক করুন
6. `Create repository` ক্লিক করুন

### ধাপ ৬.২ — Repo Clone করুন

আপনার কম্পিউটারে Desktop-এ একটি ফোল্ডার তৈরি করুন। তারপর Terminal/Command Prompt খুলুন:

```bash
# Desktop এ যান
cd ~/Desktop

# Repo clone করুন (আপনার GitHub username দিয়ে পরিবর্তন করুন)
git clone https://github.com/YOUR_USERNAME/my-aios.git

# ফোল্ডারে প্রবেশ করুন
cd my-aios
```

### ধাপ ৬.৩ — VS Code এ খুলুন

```bash
code .
```
অথবা VS Code খুলুন → `File` → `Open Folder` → আপনার `my-aios` ফোল্ডার সিলেক্ট করুন।

### ধাপ ৬.৪ — প্রজেক্টের ফোল্ডার কাঠামো তৈরি করুন

VS Code-এ Terminal খুলুন (`Ctrl+`` ` বা `Cmd+`` `):

```bash
# প্রয়োজনীয় ফোল্ডার তৈরি করুন
mkdir -p .claude/skills
mkdir -p context
mkdir -p references
mkdir -p wiki/raw
mkdir -p wiki/processed

# মূল ফাইলগুলো তৈরি করুন
touch CLAUDE.md
touch context/about_me.md
touch context/about_business.md
touch context/priorities.md
touch .env
```

### ধাপ ৬.৫ — .gitignore ফাইল তৈরি করুন

**.env ফাইল কখনো GitHub-এ push করবেন না।** এটি নিশ্চিত করতে:

```bash
# .gitignore ফাইল তৈরি করুন
echo ".env" >> .gitignore
echo "*.env" >> .gitignore
echo ".env.local" >> .gitignore
```

### ধাপ ৬.৬ — CLAUDE.md ফাইল লিখুন

এটি আপনার পুরো AIOS-এর "মাস্টার প্রম্পট"। এই ফাইলটি Claude Code প্রতিটি সেশনে প্রথমে পড়বে।

`CLAUDE.md` ফাইলটি খুলুন এবং নিচের টেমপ্লেট অনুসরণ করুন:

```markdown
# আমার AI Operating System

## আমার পরিচয়
- নাম: [আপনার নাম]
- ভূমিকা: [আপনার পেশা/ভূমিকা]
- ব্যবসা: [আপনার ব্যবসার নাম]

## কীভাবে আমাকে সাহায্য করবে
তুমি আমার AI চিফ অফ স্টাফ। তুমি:
- আমার সম্পর্কে সব মনে রাখবে (context/ ফোল্ডার দেখো)
- আমার টুলগুলো ব্যবহার করতে পারবে
- আমার হয়ে কাজ করবে

## প্রতিটি সেশনে
১. context/ ফোল্ডার পড়ো
২. আমার বর্তমান priorities বোঝো
৩. উপযুক্ত skill ব্যবহার করো

## Skills অবস্থান
সব skills: .claude/skills/ ফোল্ডারে

## মূল নিয়ম
- সরাসরি এবং কার্যকর উত্তর দাও
- অনুমান করার আগে context ফাইল পড়ো
- প্রয়োজনে reference/ ফাইল দেখো
```

---

## 👋 অধ্যায় ৭ — Onboarding Skill: প্রথম কথোপকথন

*(ভিডিওর timestamp: ২৯:০০)*

Onboarding Skill হলো AIOS-এর প্রথম এবং সবচেয়ে গুরুত্বপূর্ণ অংশ। এটি Claude-কে আপনার সম্পর্কে বিস্তারিত জানতে সাহায্য করে।

### ধাপ ৭.১ — Onboarding Skill ফাইল তৈরি করুন

`.claude/skills/onboard/` ফোল্ডার তৈরি করুন এবং `skill.md` ফাইল লিখুন:

```bash
mkdir -p .claude/skills/onboard
```

`.claude/skills/onboard/skill.md` ফাইলের বিষয়বস্তু:

```markdown
---
name: onboard
description: নতুন ব্যবহারকারীকে AIOS-এ স্বাগত জানাও এবং তাদের সম্পর্কে বিস্তারিত জানো। 
যখন কেউ বলে "onboard", "শুরু করতে চাই", "setup করতে চাই" তখন এই skill চালাও।
---

# Onboarding Skill

## উদ্দেশ্য
ব্যবহারকারীর সাথে ৭টি প্রশ্নের সাক্ষাৎকার নাও এবং তিনটি context ফাইল তৈরি করো।

## ধাপসমূহ

### ধাপ ১: স্বাগত বার্তা
বলো: "স্বাগতম! আমি আপনার AI OS সেটআপ করতে সাহায্য করব। আমি আপনাকে ৭টি প্রশ্ন করব।"

### ধাপ ২: ৭টি প্রশ্ন জিজ্ঞেস করো (একটি একটি করে)

প্রশ্ন ১: "আপনার নাম কী এবং আপনি কী করেন?"
প্রশ্ন ২: "আপনার ব্যবসা বা প্রতিষ্ঠানের নাম কী এবং এটি কী করে?"
প্রশ্ন ৩: "আপনার ৩টি প্রধান লক্ষ্য কী (এই মাসের জন্য)?"
প্রশ্ন ৪: "আপনি কোন টুলগুলো ব্যবহার করেন? (ClickUp, Gmail, Slack, ইত্যাদি)"
প্রশ্ন ৫: "আপনি কোন ধরনের কাজে সবচেয়ে বেশি সময় নষ্ট করেন?"
প্রশ্ন ৬: "আপনার লেখার নমুনা দিন — একটি পোস্ট, ইমেইল, বা বার্তা।"
প্রশ্ন ৭: "আপনি কীভাবে চান যে AI আপনার সাথে কথা বলুক? (আনুষ্ঠানিক/অনানুষ্ঠানিক)"

### ধাপ ৩: তিনটি ফাইল তৈরি করো

**context/about_me.md** — ব্যক্তিগত তথ্য ও লেখার ধরন
**context/about_business.md** — ব্যবসার বিবরণ ও লক্ষ্য
**context/priorities.md** — এই মাসের/সপ্তাহের অগ্রাধিকার

### ধাপ ৪: নিশ্চিতকরণ
"আপনার context ফাইলগুলো তৈরি হয়েছে। এখন আমি আপনার সম্পর্কে অনেক কিছু জানি!"
```

### ধাপ ৭.২ — Onboarding চালান

VS Code-এ Claude Code panel খুলুন (উপরে Anthropic লোগো) এবং লিখুন:

```
আমি এইমাত্র এই প্রজেক্ট সেটআপ করেছি। আমাকে onboard করো।
```

Claude তখন প্রশ্নগুলো করবে। **ধীরে ধীরে, বিস্তারিত উত্তর দিন।** আপনার আসল লেখার নমুনা পেস্ট করুন (একটি ইমেইল, পোস্ট)।

### ধাপ ৭.৩ — Context যাচাই করুন

Onboarding শেষ হওয়ার পরে লিখুন:

```
এই সপ্তাহে আমার কী করা উচিত?
```

যদি Claude আপনার priorities.md থেকে নির্দিষ্ট পরামর্শ দেয়, তাহলে Context Layer সফলভাবে কাজ করছে।

---

## 🔗 অধ্যায় ৮ — ClickUp সংযোগ করা

*(ভিডিওর timestamp: ৩৬:৩০)*

ClickUp হলো সবচেয়ে সাধারণ টাস্ক ম্যানেজমেন্ট টুল যা Nate ব্যবহার করেন। আপনি Notion, Asana, বা Linear-ও ব্যবহার করতে পারেন — প্রক্রিয়া একই।

### ধাপ ৮.১ — ClickUp API Key নিন

1. ClickUp-এ লগইন করুন
2. উপরে ডানদিকে আপনার Avatar-এ ক্লিক করুন
3. `Settings` → `Apps` → `API Token`
4. Token কপি করুন

### ধাপ ৮.২ — .env ফাইলে সেভ করুন

```bash
# .env ফাইলে লিখুন
CLICKUP_API_KEY=your_api_key_here
CLICKUP_WORKSPACE_ID=your_workspace_id_here
```

> **⚠️ মনে রাখুন:** `.env` ফাইল কখনো GitHub-এ push করবেন না। `.gitignore` ফাইলে `.env` থাকা নিশ্চিত করুন।

### ধাপ ৮.৩ — ClickUp Reference ফাইল তৈরি করুন

Claude Code-এ লিখুন:

```
আমি ClickUp সংযোগ করতে চাই। ClickUp API documentation research করো এবং 
references/clickup-api.md ফাইলে সব endpoints সংরক্ষণ করো। 
তারপর .env ফাইলে API key এর জন্য placeholder তৈরি করো।
```

Claude নিজেই API docs research করে একটি local markdown ফাইল তৈরি করবে।

### ধাপ ৮.৪ — Connection যাচাই করুন

```
আমার ClickUp-এ এই সপ্তাহের সব open tasks দেখাও।
```

যদি আসল tasks দেখায়, Connection সফলভাবে কাজ করছে।

### ধাপ ৮.৫ — ClickUp Skill তৈরি করুন

`.claude/skills/clickup-tasks/skill.md`:

```markdown
---
name: clickup-tasks
description: ClickUp থেকে tasks দেখাও, তৈরি করো, আপডেট করো। 
"tasks দেখাও", "task তৈরি করো", "প্রজেক্ট আপডেট" এই ধরনের কথায় চালাও।
---

# ClickUp Tasks Skill

## উদ্দেশ্য
ClickUp API ব্যবহার করে tasks ম্যানেজ করা।

## Reference
API endpoints: references/clickup-api.md
API Key: .env ফাইলের CLICKUP_API_KEY

## সাধারণ কাজ

### Open Tasks দেখানো
GET /v2/team/{team_id}/task?status=open

### Task তৈরি করা
POST /v2/list/{list_id}/task
Body: {"name": "...", "description": "...", "due_date": ...}

### Task আপডেট করা
PUT /v2/task/{task_id}

## আউটপুট ফর্ম্যাট
Tasks দেখানোর সময় table format ব্যবহার করো:
| Task | অ্যাসাইনি | Due Date | Status |
```

---

## 📊 অধ্যায় ৯ — Audit ও Level Up Skills

*(ভিডিওর timestamp: ৪৭:৩০)*

এই দুটি বিশেষ skill আপনার AIOS-এর স্বাস্থ্য পরীক্ষা করে।

### ধাপ ৯.১ — Audit Skill তৈরি করুন

`.claude/skills/audit/skill.md`:

```markdown
---
name: audit
description: AIOS-এর চারটি C (Context, Connections, Capabilities, Cadence) মূল্যায়ন করো 
এবং ১০০ এর মধ্যে স্কোর দাও। "/audit" বললে চালাও।
---

# AIOS Audit Skill

## মূল্যায়নের মানদণ্ড

### C1 — Context (২৫ পয়েন্ট)
- about_me.md বিস্তারিত? (১০)
- about_business.md সম্পূর্ণ? (৮)
- priorities.md আপডেট? (৭)

### C2 — Connections (২৫ পয়েন্ট)
- কতটি টুল সংযুক্ত? (প্রতিটির জন্য ৫, সর্বোচ্চ ২৫)

### C3 — Capabilities (২৫ পয়েন্ট)
- কতটি কাস্টম skill আছে? (প্রতিটির জন্য ৫, সর্বোচ্চ ২৫)

### C4 — Cadence (২৫ পয়েন্ট)
- কতটি scheduled routine চলছে? (প্রতিটির জন্য ৫, সর্বোচ্চ ২৫)

## আউটপুট ফর্ম্যাট

```
🔍 AIOS Audit Report
====================
Context:      __/25 ██████░░░░
Connections:  __/25 ████░░░░░░
Capabilities: __/25 ██░░░░░░░░
Cadence:      __/25 █░░░░░░░░░
--------------------------
মোট স্কোর:  __/100

📋 উন্নতির পরামর্শ:
১. ...
২. ...
৩. ...
```
```

### ধাপ ৯.২ — Level Up Skill তৈরি করুন

`.claude/skills/level-up/skill.md`:

```markdown
---
name: level-up
description: AIOS উন্নত করার সুযোগ খুঁজে বের করো। ৫টি ডায়াগনস্টিক প্রশ্ন করো 
এবং নতুন automation সুযোগ খুঁজে দাও। "/level-up" বললে চালাও।
---

# Level Up Skill

## ৫টি ডায়াগনস্টিক প্রশ্ন

১. "গত সপ্তাহে আপনি কোন কাজটি ৩ বারের বেশি করেছেন?"
২. "কোন কাজটি আপনার সবচেয়ে বেশি সময় নেয়?"
৩. "এমন কোন রিপোর্ট আছে যা প্রতি সপ্তাহে তৈরি করতে হয়?"
৪. "কোন তথ্য আপনাকে প্রতিদিন চেক করতে হয়?"
৫. "আপনার দলকে কোন আপডেট নিয়মিত পাঠাতে হয়?"

## ফলাফল
উত্তরের ভিত্তিতে নতুন skill তৈরির প্রস্তাব দাও।
```

### ধাপ ৯.৩ — Audit চালান

```
/audit
```

প্রথমবার ৫০/১০০ পেলে হতাশ হবেন না — এটাই স্বাভাবিক। Audit বলে দেবে কী করতে হবে।

---

## 📧 অধ্যায় ১০ — Google Workspace CLI সেটআপ

*(ভিডিওর timestamp: ৫১:৩০)*

Google Workspace CLI (GWS CLI) একটি open-source টুল যা Claude-কে bash command-এর মাধ্যমে Gmail, Drive, Calendar, Docs, Sheets-এ অ্যাক্সেস দেয়। এটি সেটআপ করতে ~২০ মিনিট লাগে কিন্তু অনেক শক্তিশালী।

### ধাপ ১০.১ — Google Cloud Console-এ প্রজেক্ট তৈরি করুন

1. [https://console.cloud.google.com](https://console.cloud.google.com) যান
2. উপরে `Select a project` → `New Project`
3. নাম দিন: `my-aios` → `Create`

### ধাপ ১০.২ — APIs সক্রিয় করুন

বাম মেনু থেকে `APIs & Services` → `Library`:

নিচের APIs সক্রিয় করুন (সার্চ করুন এবং `Enable` ক্লিক করুন):
- ✅ Gmail API
- ✅ Google Drive API
- ✅ Google Calendar API
- ✅ Google Docs API
- ✅ Google Sheets API

### ধাপ ১০.৩ — OAuth Credentials তৈরি করুন

1. `APIs & Services` → `Credentials`
2. `+ Create Credentials` → `OAuth client ID`
3. Application type: **Desktop app**
4. নাম দিন: `AIOS Desktop`
5. `Create` ক্লিক করুন
6. JSON ফাইল ডাউনলোড করুন

### ধাপ ১০.৪ — GWS CLI ইন্সটল করুন

```bash
# GWS CLI ইন্সটল করুন
pip install gws-cli

# অথবা npm দিয়ে
npm install -g @google-workspace/gws-cli
```

### ধাপ ১০.৫ — Config ফোল্ডারে JSON সরান

```bash
# Config ফোল্ডার তৈরি করুন
mkdir -p ~/.config/gws

# ডাউনলোড করা JSON ফাইল সরান (ফাইলের নাম আপনার ডাউনলোড করা ফাইল অনুযায়ী দিন)
mv ~/Downloads/client_secret_*.json ~/.config/gws/credentials.json
```

### ধাপ ১০.৬ — Authentication করুন

```bash
gws auth login
```

Browser খুলবে — আপনার Google account দিয়ে লগইন করুন এবং permission দিন।

### ধাপ ১০.৭ — Connection যাচাই করুন

Claude Code-এ লিখুন:

```
GWS CLI ব্যবহার করে আমার Gmail থেকে আজকের অপঠিত ইমেইলগুলো দেখাও।
```

যদি ইমেইল দেখায়, সংযোগ সফল।

### ধাপ ১০.৮ — Gmail Skill তৈরি করুন

`.claude/skills/email-drafts/skill.md`:

```markdown
---
name: email-drafts
description: Gmail ইমেইল পড়ো, সারাংশ করো, ড্রাফট তৈরি করো। 
"ইমেইল দেখাও", "ইমেইল লেখো", "inbox চেক করো" বললে চালাও।
---

# Email Drafts Skill

## GWS CLI কমান্ড

### অপঠিত ইমেইল দেখানো
```bash
gws gmail list --unread --max 10
```

### ইমেইল পড়া
```bash
gws gmail get --id EMAIL_ID
```

### ড্রাফট তৈরি করা
context/about_me.md থেকে লেখার স্টাইল নাও এবং সেই অনুযায়ী লেখো।

## ইমেইল লেখার নিয়ম
১. context/about_me.md থেকে writing style পড়ো
২. সেই স্টাইলে ড্রাফট করো
৩. সরাসরি পাঠানো নয় — সবসময় review করতে দাও
```

---

## 🛠️ অধ্যায় ১১ — Capabilities: Skills তৈরি করা

*(ভিডিওর timestamp: ১:০৬:০০)*

Skills হলো AIOS-এর হৃদয়। প্রতিটি skill একটি `.md` ফাইল যা Claude-কে বলে কীভাবে একটি নির্দিষ্ট কাজ করতে হবে।

### Skill-এর গঠন

প্রতিটি skill ফাইলের দুটি অংশ:

```markdown
---
name: skill-name
description: এই skill কী করে এবং কখন চালাতে হবে। 
এই বর্ণনাটি গুরুত্বপূর্ণ — Claude এটি পড়ে বোঝে কখন এই skill ব্যবহার করবে।
---

# Skill-এর বিস্তারিত নির্দেশাবলী

ধাপে ধাপে SOP (Standard Operating Procedure) লিখুন।
```

### কখন Skill তৈরি করবেন?

**সহজ নিয়ম:** যদি কোনো কাজ আপনি দুইবারের বেশি করেন, তাহলে একটি skill তৈরি করুন।

### Skills-এর ফোল্ডার কাঠামো

```
.claude/skills/
├── onboard/
│   └── skill.md
├── audit/
│   └── skill.md
├── level-up/
│   └── skill.md
├── clickup-tasks/
│   └── skill.md
├── email-drafts/
│   └── skill.md
├── weekly-report/
│   └── skill.md
├── social-posts/
│   └── skill.md
└── meeting-summary/
    └── skill.md
```

### Skills তৈরির সেরা অভ্যাস

**১. YAML Description সুনির্দিষ্ট রাখুন:**

```yaml
# ❌ খুব ব্যাপক (সব সময় trigger হবে)
description: সাহায্য করো

# ✅ সুনির্দিষ্ট
description: সাপ্তাহিক টিম রিপোর্ট তৈরি করো। 
"weekly report", "সাপ্তাহিক রিপোর্ট", "team update" বললে চালাও।
```

**২. Skill ফাইল ৫০০ লাইনের নিচে রাখুন:**
- বিস্তারিত reference → আলাদা ফাইলে রাখুন
- Skill.md শুধু SOP

**৩. Supporting ফাইল আলাদা রাখুন:**

```
.claude/skills/weekly-report/
├── skill.md          ← SOP/নির্দেশাবলী
├── template.md       ← রিপোর্ট template
└── example-output.md ← উদাহরণ আউটপুট
```

---

## 🔨 অধ্যায় ১২ — Live Skill Build: হাতে কলমে

*(ভিডিওর timestamp: ১:২৫:৩০)*

এই অধ্যায়ে আমরা সম্পূর্ণ একটি skill তৈরি করব — Weekly Report Skill।

### ধাপ ১২.১ — Claude-কে দিয়েই Skill তৈরি করান

Claude Code-এ লিখুন:

```
প্রতি সোমবার আমি এই কাজগুলো করি:
১. ClickUp থেকে দলের open tasks চেক করি
২. কে কতটা ব্যস্ত দেখি
৩. #updates চ্যানেলে একটি স্ট্যাটাস আপডেট লিখি

এই প্রক্রিয়াটিকে একটি skill-এ পরিণত করো। 
Skill ফাইল .claude/skills/weekly-report/skill.md এ সেভ করো।
```

### ধাপ ১২.২ — প্রথম Run

```
/weekly-report
```

দেখুন কী হয়। প্রথমবার নিখুঁত নাও হতে পারে।

### ধাপ ১২.৩ — Skill উন্নত করুন

Skill চালানোর পরে feedback দিন:

```
রিপোর্টে due date উল্লেখ নেই। প্রতিটি task-এর পাশে due date যোগ করো। 
Skill ফাইল আপডেট করো।
```

এভাবে ৩-৫ বার iteration করলে skill অনেক ভালো হয়।

### ধাপ ১২.৪ — একটি সম্পূর্ণ Weekly Report Skill

`.claude/skills/weekly-report/skill.md`:

```markdown
---
name: weekly-report
description: সাপ্তাহিক টিম স্ট্যাটাস রিপোর্ট তৈরি করো ClickUp থেকে ডেটা নিয়ে।
"weekly report", "সাপ্তাহিক রিপোর্ট", "/weekly-report" বললে চালাও।
---

# Weekly Report Skill

## উদ্দেশ্য
প্রতি সোমবার দলের সাপ্তাহিক স্ট্যাটাস রিপোর্ট তৈরি করা।

## ধাপ ১: ডেটা সংগ্রহ
- ClickUp API ব্যবহার করো (references/clickup-api.md দেখো)
- এই সপ্তাহে complete হওয়া tasks আনো
- Open/overdue tasks আনো
- দলের প্রতিটি সদস্যের workload দেখো

## ধাপ ২: বিশ্লেষণ
- কে overloaded?
- কোন task আটকে আছে?
- কোন deadline মিস হতে পারে?

## ধাপ ৩: রিপোর্ট তৈরি করো

নিচের template ব্যবহার করো (template.md দেখো):

```
📊 সাপ্তাহিক স্ট্যাটাস আপডেট — [তারিখ]

✅ এই সপ্তাহে সম্পন্ন:
• [Task 1] — [ব্যক্তি]
• [Task 2] — [ব্যক্তি]

🔄 চলমান কাজ:
• [Task] — [ব্যক্তি] — Due: [তারিখ]

⚠️ মনোযোগ প্রয়োজন:
• [যেকোনো risk বা block]

📋 এই সপ্তাহের পরিকল্পনা:
• [Task 1]
• [Task 2]
```

## ধাপ ৪: পর্যালোচনা
রিপোর্ট সরাসরি পাঠানো নয়। আগে আমাকে দেখাও।
```

---

## ⏰ অধ্যায় ১৩ — Cadence: Cloud Routines

*(ভিডিওর timestamp: ১:৩৫:৩০)*

এখানেই AIOS একটি সাধারণ assistant থেকে একটি **সত্যিকারের Operating System**-এ পরিণত হয়। Remote Routines আপনার ল্যাপটপ বন্ধ থাকলেও চলে।

### Remote Routine কীভাবে কাজ করে?

```
Anthropic Cloud
     │
     ├── আপনার GitHub Repo Clone করে
     ├── নির্দিষ্ট Prompt চালায়
     ├── API Keys ব্যবহার করে (Cloud Variables থেকে)
     ├── কাজ সম্পন্ন করে
     └── Result সেভ করে বা পাঠায়
```

**গুরুত্বপূর্ণ:** Remote Routine-এ `.env` ফাইল কাজ করে না (কারণ এটি GitHub-এ নেই)। API Keys অবশ্যই Claude Desktop App-এর Cloud Environment Settings-এ সেট করতে হবে।

### ধাপ ১৩.১ — GitHub-এ Push করুন

```bash
# সব পরিবর্তন add করুন
git add .

# Commit করুন
git commit -m "Initial AIOS setup"

# GitHub-এ push করুন
git push origin main
```

> **মনে রাখুন:** `.env` ফাইল push হবে না (`.gitignore` এর কারণে) — এটাই সঠিক।

### ধাপ ১৩.২ — Cloud Environment Variables সেট করুন

Claude Desktop App খুলুন:
1. `Settings` → `Cloud Environment`
2. `Add Variable` ক্লিক করুন
3. প্রতিটি API key আলাদাভাবে যোগ করুন:

```
CLICKUP_API_KEY = your_key_here
CLICKUP_WORKSPACE_ID = your_id_here
GWS_CREDENTIALS = (JSON content as string)
```

### ধাপ ১৩.৩ — প্রথম Remote Routine তৈরি করুন

Claude Desktop App-এ:
1. `Scheduled Tasks` → `New Scheduled Task`
2. নিচের তথ্য দিন:

**উদাহরণ: সাপ্তাহিক রিপোর্ট Routine**

```
নাম: Weekly Team Report
Schedule: প্রতি সোমবার সকাল ৯টা
Prompt:
"/weekly-report চালাও। ClickUp API ব্যবহার করো। 
API key environment variable CLICKUP_API_KEY থেকে নাও 
(not from .env file)। রিপোর্ট তৈরি হলে ClickUp-এর 
#updates চ্যানেলে পোস্ট করো।"
```

### ধাপ ১৩.৪ — ভালো Routine Prompt লেখার নিয়ম

**❌ খারাপ Prompt (অস্পষ্ট):**
```
টিম চেক করো
```

**✅ ভালো Prompt (সুনির্দিষ্ট):**
```
weekly-report skill চালাও। ClickUp API থেকে এই সপ্তাহের tasks আনো 
(API key: environment variable CLICKUP_API_KEY)। রিপোর্ট তৈরি করে 
ClickUp workspace-এর "Weekly Updates" list-এ একটি task হিসেবে 
তৈরি করো। কোনো প্রশ্ন ছাড়াই সম্পন্ন করো।
```

**ভালো Prompt-এর বৈশিষ্ট্য:**
- কোন skill চালাতে হবে স্পষ্ট বলা
- API keys কোথায় পাবে বলা
- Output কোথায় যাবে বলা
- "কোনো প্রশ্ন ছাড়াই" উল্লেখ করা

### ধাপ ১৩.৫ — দৈনিক Routine উদাহরণ

```
নাম: Daily Priority Brief
Schedule: প্রতিদিন সকাল ৮:০০

Prompt:
"context/priorities.md পড়ো। ClickUp থেকে আজকের due tasks আনো। 
GWS CLI দিয়ে Gmail থেকে unread emails (সর্বোচ্চ ১০টি) আনো। 
এই তিনটি থেকে আজকের সবচেয়ে গুরুত্বপূর্ণ ৫টি কাজের তালিকা তৈরি করো। 
ClickUp-এ 'Daily Brief' task হিসেবে সেভ করো।"
```

---

## 🔁 অধ্যায় ১৪ — Loop ও Reminders

*(ভিডিওর timestamp: ১:৫৬:৩০)*

Loop হলো active session-এর মধ্যে scheduled tasks। এটি ৩ দিন পর এবং session বন্ধ হলে expire হয়।

### ধাপ ১৪.১ — Loop Skill কী?

Remote Routines চলে আপনার ল্যাপটপ ছাড়া। কিন্তু যখন আপনি কাজ করছেন, তখন in-session reminders দরকার হতে পারে।

```
Remote Routines  → ল্যাপটপ বন্ধ থাকলেও চলে (দীর্ঘমেয়াদী)
Loop/Cron Jobs   → সক্রিয় session-এ চলে (স্বল্পমেয়াদী)
```

### ধাপ ১৪.২ — Loop Skill তৈরি করুন

`.claude/skills/loop/skill.md`:

```markdown
---
name: loop
description: Session-এর মধ্যে recurring cron jobs তৈরি করো। 
"loop", "মনে করিয়ে দাও", "প্রতি X মিনিটে চেক করো" বললে চালাও।
---

# Loop Skill

## উপলব্ধ Tools
- cron_create: নতুন cron job তৈরি করো
- cron_list: সক্রিয় cron jobs দেখো
- cron_delete: cron job বাতিল করো

## ব্যবহারের উদাহরণ

### প্রতি ২০ মিনিটে deploy check
cron_create("deploy-check", "20m", "ClickUp-এ deployment status task চেক করো")

### ৩টায় reminder
cron_create("afternoon-reminder", "15:00", "আজকের বাকি tasks review করো")

### প্রতি ঘণ্টায় inbox check
cron_create("inbox-check", "1h", "Gmail থেকে urgent emails চেক করো")

## সীমাবদ্ধতা
- ৩ দিনের expiry
- Session বন্ধ হলে expire হয়
- দীর্ঘমেয়াদী কাজের জন্য Remote Routines ব্যবহার করো
```

### ধাপ ১৪.৩ — Loop ব্যবহার করুন

```
প্রতি ৩০ মিনিটে আমাকে মনে করিয়ে দাও এই task-এর progress review করতে।
```

```
আজকে বিকেল ৩টায় আমাকে মনে করিয়ে দাও draft review করতে।
```

---

## 📚 অধ্যায় ১৫ — Karpathy LLM Wiki: জ্ঞান ভান্ডার তৈরি

*(ভিডিওর timestamp: ২:০৫:০০)*

AI বিশেষজ্ঞ Andrej Karpathy-এর একটি পদ্ধতি অনুসরণ করে আপনি আপনার AIOS-এর জ্ঞান ভান্ডার তৈরি করতে পারেন যা সময়ের সাথে আরো স্মার্ট হয়।

### ধারণাটি কী?

```
প্রতিদিন নতুন তথ্য আসে
        │
        ▼
   wiki/raw/ ফোল্ডারে জমা হয়
        │
        ▼ (রাত্রের Routine)
Claude প্রসেস করে
        │
        ▼
   wiki/processed/ ফোল্ডারে সংরক্ষিত
        │
        ▼
AIOS এই তথ্য ব্যবহার করে স্মার্ট উত্তর দেয়
```

### ধাপ ১৫.১ — Wiki ফোল্ডার কাঠামো

```
wiki/
├── raw/           ← নতুন, প্রসেস না করা তথ্য
│   ├── 2026-05-26-meeting-notes.md
│   ├── 2026-05-25-client-feedback.md
│   └── article-to-read.pdf
├── processed/     ← প্রসেস করা, structured তথ্য
│   ├── clients/
│   ├── projects/
│   ├── team/
│   └── industry/
└── index.md       ← কোথায় কী আছে তার সূচি
```

### ধাপ ১৫.২ — Wiki Ingestion Skill তৈরি করুন

`.claude/skills/wiki-ingest/skill.md`:

```markdown
---
name: wiki-ingest
description: wiki/raw/ ফোল্ডারের নতুন ফাইলগুলো প্রসেস করে wiki/processed/ এ সংরক্ষণ করো।
রাতের routine এবং "/wiki-ingest" বললে চালাও।
---

# Wiki Ingestion Skill

## ধাপ ১: Raw ফাইল স্ক্যান করো
wiki/raw/ এ নতুন ফাইল খোঁজো (আজকের বা গতকালের)

## ধাপ ২: প্রতিটি ফাইল বিশ্লেষণ করো
- মূল বিষয়বস্তু কী?
- কোন category? (client, project, team, industry)
- Key insights কী?
- Action items আছে কি?

## ধাপ ৩: Structured Format-এ সেভ করো

wiki/processed/[category]/[filename].md:

```markdown
# [শিরোনাম]

**তারিখ:** [তারিখ]
**Category:** [বিভাগ]
**সোর্স:** [মূল ফাইল]

## সারাংশ
[৩-৫ লাইনে মূল বিষয়]

## Key Points
- [পয়েন্ট ১]
- [পয়েন্ট ২]

## Action Items
- [ ] [কাজ ১]

## Tags
[searchable keywords]
```

## ধাপ ৪: Index আপডেট করো
wiki/index.md এ নতুন এন্ট্রি যোগ করো।

## ধাপ ৫: Raw ফাইল Archive করো
wiki/raw/archive/ ফোল্ডারে সরাও।
```

### ধাপ ১৫.৩ — নাইট্লি Wiki Routine

Claude Desktop App-এ নতুন Routine:

```
নাম: Nightly Wiki Update
Schedule: প্রতিদিন রাত ১১:০০

Prompt:
"wiki-ingest skill চালাও। wiki/raw/ ফোল্ডারের সব নতুন ফাইল প্রসেস করো। 
wiki/processed/ এ সেভ করো। wiki/index.md আপডেট করো। 
সম্পন্ন হলে ClickUp-এ একটি brief summary task তৈরি করো।"
```

### ধাপ ১৫.৪ — Wiki থেকে Query করুন

```
আমাদের [ক্লায়েন্টের নাম] সম্পর্কে wiki-তে কী আছে?
```

```
গত মাসে আমরা কোন প্রজেক্টগুলোতে কাজ করেছি?
```

---

## 📊 অধ্যায় ১৬ — Dashboards with Artifacts

*(ভিডিওর timestamp: ২:২৩:৩০)*

Claude Code-এর Artifacts ফিচার ব্যবহার করে আপনি সুন্দর HTML Dashboard তৈরি করতে পারবেন।

### ধাপ ১৬.১ — Dashboard Skill তৈরি করুন

`.claude/skills/dashboard/skill.md`:

```markdown
---
name: dashboard
description: ClickUp, Gmail এবং অন্যান্য সোর্স থেকে ডেটা নিয়ে HTML Dashboard তৈরি করো।
"dashboard দেখাও", "overview দাও", "/dashboard" বললে চালাও।
---

# Dashboard Skill

## ধাপ ১: ডেটা সংগ্রহ করো
- ClickUp: open tasks, overdue tasks, completed this week
- Gmail: unread count, urgent emails
- context/priorities.md: এই সপ্তাহের priorities

## ধাপ ২: HTML Dashboard তৈরি করো

নিচের ডিজাইন অনুসরণ করো:
- Header: আজকের তারিখ ও স্বাগত বার্তা
- Left Panel: Tasks overview (ClickUp)
- Right Panel: Email summary (Gmail)
- Bottom: Priorities ও upcoming deadlines
- Color coding: 🟢 সম্পন্ন, 🟡 চলমান, 🔴 overdue

## ধাপ ৩: Artifact হিসেবে দেখাও
HTML কোড একটি artifact হিসেবে রেন্ডার করো।
```

### ধাপ ১৬.২ — Dashboard চালান

```
/dashboard
```

Claude একটি সুন্দর HTML dashboard তৈরি করবে আপনার সব ডেটা দিয়ে।

### ধাপ ১৬.৩ — Dashboard Template উদাহরণ

Claude-কে দিয়ে এই ধরনের HTML তৈরি করানো যায়:

```html
<!DOCTYPE html>
<html>
<head>
  <title>AIOS Dashboard</title>
  <style>
    body { font-family: Arial; background: #0f172a; color: #e2e8f0; padding: 20px; }
    .header { font-size: 24px; font-weight: bold; margin-bottom: 20px; }
    .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
    .card { background: #1e293b; border-radius: 12px; padding: 20px; }
    .card h2 { color: #60a5fa; margin-top: 0; }
    .task-item { padding: 8px; border-bottom: 1px solid #334155; }
    .overdue { color: #f87171; }
    .done { color: #4ade80; }
    .pending { color: #fbbf24; }
  </style>
</head>
<body>
  <div class="header">📊 আমার AIOS Dashboard — [তারিখ]</div>
  <div class="grid">
    <div class="card">
      <h2>📋 Tasks</h2>
      <!-- ClickUp data এখানে -->
    </div>
    <div class="card">
      <h2>📧 Emails</h2>
      <!-- Gmail data এখানে -->
    </div>
  </div>
</body>
</html>
```

---

## 🌅 অধ্যায় ১৭ — Daily Loop ও সাফল্যের মানদণ্ড

*(ভিডিওর timestamp: ২:২৭:৩০)*

### আপনার AIOS-এর দৈনিক লুপ

একটি সম্পূর্ণ AIOS-এর দৈনিক চক্র এরকম দেখায়:

```
🌙 রাত ১১:০০ PM
└── Wiki Ingest Routine
    └── নতুন তথ্য প্রসেস করা হয়

🌅 সকাল ৮:০০ AM
└── Daily Brief Routine
    ├── ClickUp tasks চেক
    ├── Gmail unread চেক
    ├── Calendar চেক
    └── দিনের ৫টি priority তৈরি

📱 সারাদিন (আপনার অনুরোধে)
├── /email-drafts → ইমেইল লেখা
├── /clickup-tasks → task আপডেট
├── /meeting-summary → মিটিং সারাংশ
└── /dashboard → overview দেখা

📅 প্রতি সোমবার সকাল ৯:০০ AM
└── Weekly Report Routine
    ├── সাপ্তাহিক স্ট্যাটাস তৈরি
    └── Team update পাঠানো
```

### AIOS সাফল্যের মানদণ্ড

আপনার AIOS সফলভাবে কাজ করছে যদি:

**✅ Context Layer সফল:**
```
প্রশ্ন করুন: "আমার ব্যবসার মূল লক্ষ্য কী?"
→ Claude সঠিক উত্তর দেয় about_business.md থেকে
```

**✅ Connection Layer সফল:**
```
প্রশ্ন করুন: "আজকের open tasks কী কী?"
→ Claude আসল ClickUp data দেখায়
```

**✅ Capabilities Layer সফল:**
```
যেকোনো skill চালান
→ সঠিক SOP অনুসরণ করে কাজ সম্পন্ন হয়
```

**✅ Cadence Layer সফল:**
```
সোমবার সকালে আপনার ক্লিক না করে
→ Weekly Report ClickUp-এ তৈরি হয়ে যায়
```

### প্রথম সপ্তাহের চেকলিস্ট

```
□ VS Code ও Claude Code Extension ইন্সটল
□ GitHub Repo তৈরি এবং private করা
□ CLAUDE.md লেখা
□ Onboarding সম্পন্ন (৩টি context ফাইল তৈরি)
□ একটি টুল সংযুক্ত (ClickUp বা Gmail)
□ ৩টি custom skill তৈরি
□ ১টি Remote Routine সেটআপ
□ Audit চালানো (৫০+ স্কোর লক্ষ্য)
□ GitHub-এ push করা
```

---

## 🔧 অধ্যায় ১৮ — সাধারণ সমস্যা ও সমাধান

### সমস্যা ১: Remote Routine-এ API Key কাজ করছে না

**কারণ:** `.env` ফাইল GitHub-এ নেই, তাই Cloud-এ নেই।

**সমাধান:**
1. Claude Desktop → Settings → Cloud Environment
2. API keys সরাসরি এখানে যোগ করুন
3. Routine Prompt-এ স্পষ্ট করুন: `"API key environment variable থেকে নাও, .env থেকে নয়"`

---

### সমস্যা ২: Skill বারবার ভুল সময়ে Trigger হচ্ছে

**কারণ:** YAML description খুব broad।

**সমাধান:** Description আরো সুনির্দিষ্ট করুন:

```yaml
# ❌ আগে
description: টাস্ক ম্যানেজমেন্ট সাহায্য করো

# ✅ পরে
description: ClickUp-এ নতুন task তৈরি করো বা বিদ্যমান task আপডেট করো। 
শুধুমাত্র "task তৈরি করো", "ClickUp update" বললে চালাও।
```

---

### সমস্যা ৩: দীর্ঘ Session-এ Claude ভুলে যাচ্ছে

**কারণ:** Context limit হয়ে গেছে।

**সমাধান:**
```
/clear    ← conversation history clear করুন (session বন্ধ হয় না)
```
প্রতিটি আলাদা কাজের জন্য নতুন session শুরু করুন।

---

### সমস্যা ৪: GWS CLI বারবার Authentication চাইছে

**কারণ:** এটি GWS CLI-এর beta version-এর পরিচিত সমস্যা।

**সমাধান:**
```bash
gws auth login
```
এটি আবার চালান। এই সমস্যাটি ভবিষ্যৎ version-এ সমাধান হওয়ার কথা।

---

### সমস্যা ৫: Remote Routine চুপচাপ fail করছে

**সমাধান:** প্রতিটি Routine Prompt-এর শেষে যোগ করুন:

```
"যদি কোনো কারণে এই routine fail হয়, 
ClickUp-এ 'AIOS Error' task তৈরি করো 
এবং error-এর সংক্ষিপ্ত বিবরণ সেখানে লেখো।"
```

---

### সমস্যা ৬: Skill কখনো Trigger হচ্ছে না

**সমাধান:** Skill সরাসরি force-trigger করুন:

```
/skill-name
```

যদি এভাবে কাজ করে কিন্তু automatic trigger না হয়, তাহলে YAML description পরিবর্তন করুন।

---

## 🚀 অধ্যায় ১৯ — পরবর্তী পদক্ষেপ

### স্তর ১ (প্রথম সপ্তাহ): ভিত্তি
- [x] সব ইন্সটলেশন সম্পন্ন
- [x] Onboarding করা
- [x] ১টি connection (ClickUp বা Gmail)
- [x] ৩টি basic skill
- [x] ১টি Remote Routine

### স্তর ২ (প্রথম মাস): শক্তি
- [ ] সব Google Workspace সংযুক্ত
- [ ] ১০+ custom skill
- [ ] Daily Brief Routine
- [ ] Wiki System চালু
- [ ] Audit স্কোর ৭৫+

### স্তর ৩ (তিন মাস পরে): দক্ষতা
- [ ] Skill Chains তৈরি (একটি skill আরেকটি চালায়)
- [ ] Content Pipeline (trend → research → draft → review)
- [ ] Mobile অ্যাক্সেস (Telegram channel)
- [ ] Audit স্কোর ৯০+

### উন্নত Skill Ideas

```
📱 Social Media Pipeline
   trending topics → research → draft → review queue

📊 Revenue Dashboard
   Stripe/payment data → weekly summary → forecast

🤝 Client Update Skill
   project status → personalized update → email draft

📰 Industry News Brief
   web search → summarize → morning brief

🎯 OKR Tracker
   weekly check-in → progress update → team report
```

### গুরুত্বপূর্ণ সম্পদ

| সম্পদ | লিঙ্ক |
|---|---|
| মূল ভিডিও | [YouTube](https://www.youtube.com/watch?v=bCljOfCH8Ms) |
| Nate-এর Community | [Skool](https://www.skool.com/ai-automation-society) |
| GWS CLI | [GitHub](https://github.com/google-workspace/gws-cli) |
| Claude Code Docs | [docs.claude.com](https://docs.claude.com) |
| VS Code | [code.visualstudio.com](https://code.visualstudio.com) |

---

## 📝 দ্রুত রেফারেন্স কার্ড

### সবচেয়ে গুরুত্বপূর্ণ Commands

```bash
# AIOS চালু করুন
code .              # VS Code খুলুন
# তারপর Claude Code panel খুলুন

# GitHub sync
git add . && git commit -m "update" && git push

# GWS Auth (প্রয়োজনে)
gws auth login

# Skill force-trigger
/skill-name
```

### প্রয়োজনীয় ফাইল চেকলিস্ট

```
my-aios/
├── CLAUDE.md                    ← মাস্টার প্রম্পট ✅
├── .gitignore                   ← .env বাদ দেওয়া ✅
├── .env                         ← API keys (কখনো push করবেন না) ✅
├── context/
│   ├── about_me.md              ← আপনার পরিচয় ✅
│   ├── about_business.md        ← ব্যবসার বিবরণ ✅
│   └── priorities.md            ← এই সপ্তাহের লক্ষ্য ✅
├── references/
│   └── clickup-api.md           ← API docs ✅
├── wiki/
│   ├── raw/                     ← নতুন তথ্য ✅
│   └── processed/               ← প্রসেস করা তথ্য ✅
└── .claude/skills/
    ├── onboard/skill.md         ✅
    ├── audit/skill.md           ✅
    ├── level-up/skill.md        ✅
    ├── clickup-tasks/skill.md   ✅
    ├── email-drafts/skill.md    ✅
    ├── weekly-report/skill.md   ✅
    ├── wiki-ingest/skill.md     ✅
    ├── dashboard/skill.md       ✅
    └── loop/skill.md            ✅
```

### Routine Schedule টেমপ্লেট

```
🌙 রাত ১১:০০ → Wiki Ingest
🌅 সকাল ৮:০০ → Daily Brief
📅 সোমবার ৯:০০ → Weekly Report
📊 শুক্রবার ৫:০০ → Weekly Audit
```

---

## 🎯 শেষ কথা

Nate Herk বলেন: *"প্রথম সপ্তাহে তৈরি করা AIOS তিন মাস পরে চেনা যায় না — এটাই লক্ষ্য।"*

আপনার AIOS একটি জীবন্ত সিস্টেম। প্রতিটি নতুন skill যোগ হলে এটি আরো শক্তিশালী হয়। প্রতিটি connection যোগ হলে এটি আরো বেশি জানে। প্রতিটি routine যোগ হলে এটি আরো বেশি কাজ করে — আপনার হয়ে।

**শুরু করুন আজই। একটি ছোট পদক্ষেপ দিয়ে।**

---

*এই tutorial Nate Herk-এর "Build & Sell Claude Code Operating Systems" কোর্স অবলম্বনে তৈরি।*  
*মূল ভিডিও: [https://www.youtube.com/watch?v=bCljOfCH8Ms](https://www.youtube.com/watch?v=bCljOfCH8Ms)*
