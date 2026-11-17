# Student-Facing Self-Audit — MAST5112 Final PoE

> "Is This Mine, and Do I Understand It?"

**Purpose:** This is not an assignment rubric. It is a personal readiness check. Before you submit the Final PoE, use this to decide whether you genuinely understand your own work — or whether you have submitted something you cannot defend.

**How to use it:** Answer honestly. This is for you, not for marks. If you answer "not yet" to something, that is your study list — not a failure. If you answer "yes" but you are not sure, test yourself: close the editor, close the AI tool, and explain it out loud or on paper. If you cannot, the answer is "not yet."

**Important:** Do not paste this checklist into an AI tool and ask it to verify your work or generate your documentation. That defeats the purpose. The checklist is a mirror, not a tick-box. The only honest verification is you, without assistance.

---

## Section A — Technology and setup

- [ ] I know **which language and project template** I am using (e.g. TypeScript vs JavaScript, `App.tsx` vs a file-based routing template), and I can explain **why** I used it.
- [ ] I can explain the **difference between what I was taught and what I chose**, and I can justify my choice if it differs.
- [ ] I can explain **what each file in my project does** — not just the ones I edited, but the config and entry files too (`app.json`, `package.json`, `tsconfig.json`, `babel.config.js`, `metro.config.js`, and the entry file).
- [ ] If someone deleted a file, I could explain **what would break** and why.
- [ ] I know **which SDK version my project targets** (e.g. Expo SDK 54) and I can explain how I confirmed it — from `package.json` and from the Expo Go app I am using.
- [ ] I can explain **why my project and my Expo Go version must be compatible**, and what happens if they are not.
- [ ] I can show **where the compatible Expo Go build came from** (`expo.dev/go`, selected SDK, Android emulator) and explain the steps I followed.

## Section B — Reading my own code

- [ ] I can read **any function in my app** and explain, in my own words, what it does, why it exists, and what happens if it is removed.
- [ ] I can explain **any line** that I did not write myself (if I copied or generated it) — and if I cannot, I have removed it or rewritten it until I can.
- [ ] I can explain **why the code is formatted the way it is** (multi-line, readable) rather than pasted as one long block.
- [ ] I can explain **why I used each component** (`View`, `Text`, `FlatList`, `TextInput`, `Picker`, `Pressable`, `ScrollView`, `Modal`, etc.) and what would happen if I used a different one.
- [ ] I can explain **where each third-party package is used** in my code (e.g. `@react-navigation/native`, `@react-native-picker/picker`, `@react-native-async-storage/async-storage`, `expo-font`), and **why** I added it.
- [ ] I can explain what would happen if I **removed each dependency** from `package.json` — what would break, and why.

## Section C — Data and state

- [ ] I can explain **what data my app stores**, what shape it is in (e.g. an array of `MenuItem` objects), and **why** it is shaped that way.
- [ ] I can explain the **difference between state and props**, and give an example of each from my own app.
- [ ] I can explain **why state must not be changed directly**, and show where in my code I create new arrays/objects instead.
- [ ] I can explain **where the "source of truth" for my data lives** — and why it lives there.
- [ ] I can explain **what happens to my data when the app closes** — and whether that is intentional.
- [ ] I can explain **where `useState` and `useEffect` (or other hooks) are used** in my app, and why I chose them.

## Section D — Typing and structure

- [ ] I can explain **what types or interfaces I defined** (e.g. `MenuItem`, navigation params, component props), and why each field has the type it does.
- [ ] I can explain **what a union type is** and where I used one (e.g. `'Starter' | 'Main Course' | 'Dessert'`) — or why I did not need one.
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
- [ ] I can explain **where the navigation library or pattern was taught** (e.g. the official React Navigation documentation at `reactnavigation.org`, or the file-based routing approach in my project template) and how my choice relates to it.
- [ ] I can explain **where my app starts**, what the **first screen** is, and why.
- [ ] I can explain **every screen in my app**, what it shows, and what the user can do on it.
- [ ] I can explain **how the user moves from every screen to every other screen** — including back navigation and cancel/return actions.
- [ ] I can explain **where the "important actions" happen** (e.g. saving an item, confirming a delete) and what happens after each.
- [ ] I have drawn (by hand, or in a diagram tool) an **updated flow diagram** of my final navigation, and I can walk someone through it without looking at my code.
- [ ] I can explain **how my final navigation differs from the flow diagram I submitted in Part 1**, and I have documented why those changes were made.
- [ ] If I used a file-based routing template, I can explain **how the file structure maps to the flow diagram** — not just that it "works."
- [ ] If I used a custom switch/state-based screen change, I can explain **why I did not use a navigation library**, and whether that was the right choice.
- [ ] I can explain **what happens if the user presses the back button** on each screen — and whether my app handles it intentionally.

## Section G — User experience

- [ ] I can explain **the difference between validation and feedback**, and give an example of each from my own app.
- [ ] I can explain **how my app tells the user when something is wrong**, and **how it tells them when something worked** — and whether those are visible on the screen or only in a pop-up.
- [ ] I can explain **what the user sees when there is no data** (empty state) — and whether it is instructive or just a blank line.
- [ ] I can explain **what the user sees when a filter or search returns no results** — and whether the message is specific to that situation.
- [ ] I can explain **how my app behaves when the user makes a mistake** — does it guide them, or just reject them?
- [ ] I can explain **what happens on a small screen or a notched device** — and whether I have handled it.

## Section H — Referencing, attribution, and AI declaration (required)

**This section is mandatory. Referencing is not a formality — it is part of being able to document your own coding. If a reference is not in your list, it is because you did not actually use it — and if you used it, it must be in your list.**

### Dependency and documentation references

- [ ] My **reference list is extensive and matches my actual dependencies** — every package in `package.json` that I added (not just the ones that came with the template) appears in my references.
- [ ] My reference list includes **all relevant documentation I used**, such as:
  - [ ] React core concepts and hooks — `https://react.dev/reference/react` (e.g. `useState`, `useEffect`, `useMemo`, `useContext` as used)
  - [ ] React Native components — `https://reactnative.dev/docs/` (e.g. `View`, `Text`, `FlatList`, `TextInput`, `TouchableHighlight`, `TouchableOpacity`, `Pressable`, `StyleSheet`)
  - [ ] Navigation — `https://reactnavigation.org/docs/` (whichever navigators I actually used, e.g. Native Stack, Bottom Tabs) or the Expo Router documentation at `https://docs.expo.dev/router/introduction/` if I used file-based routing
  - [ ] Expo — `https://docs.expo.dev/` (SDK reference, `expo install`, `expo-doctor`, `app.json` / `app.config.js`)
  - [ ] Expo SDK 54 / Expo Go compatibility — `https://expo.dev/go` (the page used to download the compatible Expo Go build)
  - [ ] TypeScript — `https://www.typescriptlang.org/docs/` (types, interfaces, union types)
  - [ ] Picker — `https://github.com/react-native-picker/picker`
  - [ ] Async storage — `https://react-native-async-storage.github.io/async-storage/` (if used)
  - [ ] Fonts — `https://docs.expo.dev/develop/user-interface/fonts/` and the Google Fonts page for the font(s) used (if used)
  - [ ] Any tutorial, article, or video I followed — with the link and the date accessed
- [ ] If I used a **taught resource** (module notes, lecture slides, prescribed textbook), it appears in my references.
- [ ] My references are **formatted consistently** (same style throughout) and include **access dates** where appropriate.

### In-text code attribution

- [ ] Where I used **code from a tutorial, article, or documentation example**, I have added an **in-text comment** at the point of use, naming the source and linking to it.
- [ ] Where I used **AI-generated code**, I have commented the block with a clear marker, for example:
  ```ts
  // Adapted from <tool> chat <public link>, <date>
  // Reviewed and verified against <documentation link>
  ```
- [ ] For each attributed block, I can explain **why I used it**, **what it does**, and **whether I changed it** — and if I changed it, why.
- [ ] I have **not** pasted unattributed blocks from external sources without a comment — if a block has no attribution and I did not write it, I have either written it myself or removed it.
- [ ] My code is **formatted as my own readable code** — not pasted as a single block — and the formatting reflects deliberate choices I can explain.

### AI usage declaration

- [ ] I have included a **Declaration of AI Usage** in my README that states **which AI tool(s)** I used.
- [ ] For each AI tool, I have included a **link to the publicly visible chat(s)** where I asked for help — **not just the tool name**.
- [ ] I have explained **what the AI helped with** (e.g. "debugging a `FlatList` key warning", "drafting a type for `MenuItem`") and **what I did myself**.
- [ ] I have stated **how I reviewed and verified** the AI-generated content (e.g. "checked against the React Native documentation at `reactnative.dev`", "rewrote to match my data model", "tested in the emulator").
- [ ] If my chat is **not publicly visible**, I have either made it public before submission or acknowledged that I cannot provide the link and explained why.

## Section I — Process, continuity, and professionalism

- [ ] I can explain **what I built first, what broke, and how I fixed it** — the actual story of my development, not a summary.
- [ ] My **commit history** shows my work over time, with messages that describe what I did — not a single bulk commit at the end.
- [ ] I have at least **6–10 commits** across Part 3, and **README-only commits are not counted** toward that number.
- [ ] My **commit history matches my changelog** — every feature and refactor in the changelog can be traced to a commit.
- [ ] My **README** is in my own words, describes my actual project, and includes a reflection on what I learned and what I would improve.
- [ ] My **changelog records real, dated changes** since Part 2, including **what was refactored and why** — not placeholders.
- [ ] My **changelog explicitly notes any design changes** since Part 1, for example:
  - [ ] An **updated flow diagram** if my navigation has changed
  - [ ] Any **screen additions, removals, or merges** since Part 1
  - [ ] Any **colour scheme, typography, or layout changes** since Part 1 or Part 2
  - [ ] Any **dependency changes** since Part 2 (e.g. added navigation, picker, async-storage, fonts)
- [ ] My **README includes the updated flow diagram** (or a link to it) and, if it differs from Part 1, a short note explaining why.
- [ ] My **README includes screenshots** of my app running, not just code screenshots.
- [ ] My **video demonstration** shows my app running on a real device or emulator **in Expo Go**, with my own narration explaining what I did and why.
- [ ] My **video demonstrates every Part 3 feature**: edit, delete, search, filter, statistics — and shows that Part 2 features still work.
- [ ] My **video shows the statistics updating** after an edit or delete, not just at rest.
- [ ] My **video shows the app launching in Expo Go** — not just the terminal bundling successfully.
- [ ] My **references list the sources I actually used** (documentation, tutorials, notes) — and if I used an AI tool, I have **disclosed it, linked the chat, and explained what it helped with**.

## Section J — The final test

- [ ] **With no editor, no AI, and no notes**, I can open my app and **explain, trace, and modify** any part of it while someone watches.
- [ ] If a marker asked me **"why did you do it this way?"** about any decision — including a design decision — I could answer without guessing.
- [ ] If someone **changed a line of my code**, I could explain what would happen — and whether it would still work.
- [ ] If someone **asked me to point to the source of any block of code I did not write myself**, I could — and my reference list or in-text comment would back me up.
- [ ] If I had to **rebuild this app from scratch**, I could do it — maybe not faster, but I could do it — because I understand every part.

---

## A note for you (the student)

If you answered **"not yet"** to more than a few of these, that is not a reason to panic — it is a reason to act **before** the final submission. The Final PoE adds editing, deleting, searching, filtering, statistics, and (usually) navigation — and it asks you to show that you can **document** your code as well as write it. Every "not yet" you close now is one fewer thing that will break later.

The two most important questions on this list are:

> **Can you explain, trace, and modify your own app — with no editor, no AI, and no notes?**

> **Can you point to the source of every block of code you did not write yourself — and does your reference list back you up?**

If the answer to both is yes, you are ready. If the answer to either is no, the app is not yet yours — and the best thing you can do is make it yours, line by line, before you submit.

---
