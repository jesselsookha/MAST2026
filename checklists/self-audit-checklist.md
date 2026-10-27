# Student-Facing Self-Audit 

> "Is This Mine, and Do I Understand It?"

**Purpose:** This is not an assignment rubric. It is a personal readiness check. Before you submit, and especially before the final build, use this to decide whether you genuinely understand your own work — or whether you have submitted something you cannot defend.

**How to use it:** Answer honestly. This is for you, not for marks. If you answer "not yet" to something, that is your study list — not a failure. If you answer "yes" but you are not sure, test yourself: close the editor, close the AI tool, and explain it out loud or on paper. If you cannot, the answer is "not yet."

**Important:** Do not paste this checklist into an AI tool and ask it to verify your work or generate your documentation. That defeats the purpose. The checklist is a mirror, not a tick-box. The only honest verification is you, without assistance.

---

## Section A — Technology and setup

- [ ] I know **which language and project template** I am using (e.g. TypeScript vs JavaScript, `App.tsx` vs a file-based routing template), and I can explain **why** I used it.
- [ ] I can explain the **difference between what I was taught and what I chose**, and I can justify my choice if it differs.
- [ ] I can explain **what each file in my project does** — not just the ones I edited, but the config and entry files too.
- [ ] If someone deleted a file, I could explain **what would break** and why.

## Section B — Reading my own code

- [ ] I can read **any function in my app** and explain, in my own words, what it does, why it exists, and what happens if it is removed.
- [ ] I can explain **any line** that I did not write myself (if I copied or generated it) — and if I cannot, I have removed it or rewritten it until I can.
- [ ] I can explain **why the code is formatted the way it is** (multi-line, readable) rather than pasted as one long block.
- [ ] I can explain **why I used each component** (`View`, `Text`, `FlatList`, `TextInput`, `Picker`, `Pressable`, etc.) and what would happen if I used a different one.

## Section C — Data and state

- [ ] I can explain **what data my app stores**, what shape it is in (e.g. an array of objects), and **why** it is shaped that way.
- [ ] I can explain the **difference between state and props**, and give an example of each from my own app.
- [ ] I can explain **why state must not be changed directly**, and show where in my code I create new arrays/objects instead.
- [ ] I can explain **where the "source of truth" for my data lives** — and why it lives there.
- [ ] I can explain **what happens to my data when the app closes** — and whether that is intentional.

## Section D — Typing and structure

- [ ] I can explain **what types or interfaces I defined**, and why each field has the type it does.
- [ ] I can explain **what a union type is** and where I used one (or why I did not need one).
- [ ] I can read a **type error** and fix it **without** pasting it into an AI tool.
- [ ] I can explain **why typing helps** — not just that it is required — and give a concrete example from my own code where typing caught a bug.

## Section E — Side effects and persistence

- [ ] I can explain **what a side effect is** and where my app has them (loading data, saving data, timers, subscriptions).
- [ ] I can explain **the dependency array** in any `useEffect` I wrote — what `[]`, `[x]`, and no array each do, and why I chose what I chose.
- [ ] I can explain **cleanup** — why some effects must return a function, and what would happen if mine didn't.
- [ ] I can explain **how my app saves data** (if it does), **when** it saves, and **what happens if saving fails**.
- [ ] I can explain **how my app loads data** (if it does), **when** it loads, and **what happens if the stored data is malformed**.

## Section F — Navigation (required)

**This section is mandatory. Navigation is a taught concept, and you must be able to explain your navigation choices — not just that "it works."**

- [ ] I can explain **which navigation approach I used** (stack, tabs, drawer, file-based routing, or a custom switch/state-based screen change) and **why** I chose it.
- [ ] I can explain **where the navigation library or pattern was taught** (e.g. the official React Navigation documentation at reactnavigation.org, or the file-based routing approach in my project template) and how my choice relates to it.
- [ ] I can explain **where my app starts**, what the **first screen** is, and why.
- [ ] I can explain **every screen in my app**, what it shows, and what the user can do on it.
- [ ] I can explain **how the user moves from every screen to every other screen** — including back navigation and cancel/return actions.
- [ ] I can explain **where the "important actions" happen** (e.g. saving an item, confirming a delete) and what happens after each.
- [ ] I have drawn (by hand, or in a diagram tool) a **flow diagram** of my navigation, and I can walk someone through it without looking at my code.
- [ ] If I used a file-based routing template, I can explain **how the file structure maps to the flow diagram** — not just that it "works."
- [ ] If I used a custom switch/state-based screen change, I can explain **why I did not use a navigation library**, and whether that was the right choice.
- [ ] I can explain **what happens if the user presses the back button** on each screen — and whether my app handles it intentionally.

## Section G — User experience

- [ ] I can explain **the difference between validation and feedback**, and give an example of each from my own app.
- [ ] I can explain **how my app tells the user when something is wrong**, and **how it tells them when something worked** — and whether those are visible on the screen or only in a pop-up.
- [ ] I can explain **what the user sees when there is no data** (empty state) — and whether it is instructive or just a blank line.
- [ ] I can explain **how my app behaves when the user makes a mistake** — does it guide them, or just reject them?
- [ ] I can explain **what happens on a small screen or a notched device** — and whether I have handled it.

## Section H — Process and professionalism

- [ ] I can explain **what I built first, what broke, and how I fixed it** — the actual story of my development, not a summary.
- [ ] My **commit history** shows my work over time, with messages that describe what I did — not a single bulk commit at the end.
- [ ] My **README** is in my own words, describes my actual project, and includes a reflection on what I learned and what I would improve.
- [ ] My **changelog** records real, dated changes — not placeholders.
- [ ] My **references** list the sources I actually used (documentation, tutorials, notes) — and if I used an AI tool, I have **disclosed it and explained what it helped with**.
- [ ] My **video demonstration** shows my app running on a real device or emulator, with my own narration explaining what I did and why.

## Section I — The final test

- [ ] **With no editor, no AI, and no notes**, I can open my app and **explain, trace, and modify** any part of it while someone watches.
- [ ] If a marker asked me **"why did you do it this way?"** about any decision, I could answer without guessing.
- [ ] If someone **changed a line of my code**, I could explain what would happen — and whether it would still work.
- [ ] If I had to **rebuild this app from scratch**, I could do it — maybe not faster, but I could do it — because I understand every part.

---

## A note for you (the student)

If you answered **"not yet"** to more than a few of these, that is not a reason to panic — it is a reason to act **before** the final submission. The final build is harder than this one: it adds editing, deleting, searching, filtering, statistics, persistence, and (usually) navigation. Every "not yet" you close now is one fewer thing that will break later.

The single most important question on this list is the last one in Section I:

> **Can you explain, trace, and modify your own app — with no editor, no AI, and no notes?**

If the answer is yes, you are ready. If the answer is no, the app is not yet yours — and the best thing you can do is make it yours, line by line, before you submit.

---
