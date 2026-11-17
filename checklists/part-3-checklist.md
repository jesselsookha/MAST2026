# MAST5112 — Chef's Menu Manager
## PoE Part 3 (Final PoE): Enhancing and Completing the Mobile Application

### Important Disclaimer

> **Please read this first.**

This checklist is aligned to the Part 3 marking rubric and the assignment brief. It is intended to help you prepare a stronger submission.

- It does **not** represent the exact marking allocation used during assessment.
- Individual checklist items should **not** be interpreted as carrying specific marks.
- Meeting every item does **not** automatically guarantee a particular mark.
- Final marks will be awarded using the official rubric and the overall quality of the completed application.

Use this guide to understand what must be submitted, what excellent work looks like, common mistakes that cost marks, and the professional software development practices expected in a final-year mobile scripting project.

---

## Section 1: Submission Checklist

Before submitting your Final PoE, ensure the following requirements have been completed.

### Application — Runs Correctly

- [ ] Application runs successfully on the target emulator/device via **Expo Go**
- [ ] Application compiles without errors
- [ ] All Part 3 features are working (edit, delete, search, filter, statistics)
- [ ] Part 2 features still work (add, view, validation, feedback)
- [ ] No broken navigation between screens
- [ ] No unfinished screens or placeholder content
- [ ] Application has been tested end-to-end before submission

### Expo SDK 54 / Expo Go Compatibility — **Critical**

> **This is the most common cause of "my app won't run" submissions in 2026.**

- [ ] You have **confirmed your project's SDK version** by checking the `expo` dependency in `package.json` (e.g. `"expo": "~54.0.x"` means SDK 54)
- [ ] You have **confirmed which SDK version the installed Expo Go app supports** (visible in the Expo Go app settings, or by checking the version you downloaded)
- [ ] If using **BlueStacks in the assigned VM**: you have installed the **SDK 54–compatible Expo Go build** from `expo.dev/go` by selecting SDK 54 → Android emulator, **not** a generic or newer build
- [ ] If using a **physical phone**: you understand that a newer Expo Go on your phone will **not** open an SDK 54 project, and you have either (a) installed the SDK 54–compatible Expo Go build, or (b) **upgraded the project** to match your Expo Go version — but only if all dependencies were updated consistently
- [ ] You have run `npx expo install --fix` and `npx expo-doctor` to align dependency versions with the installed SDK
- [ ] Your `app.json` does **not** contain a conflicting `sdkVersion` field that contradicts `package.json`
- [ ] You can demonstrate the app opening in Expo Go **in your video** — not just the terminal saying "Bundled successfully"

### GitHub Repository

- [ ] Latest version of the project pushed to the **assigned organisational repository**
- [ ] Repository is accessible to the marker
- [ ] All project files included (source code, assets, configuration — not `node_modules`)
- [ ] Repository is organised and professional
- [ ] **Minimum 6–10 meaningful commits** demonstrating development history across Part 3
- [ ] Commit messages clearly describe the work completed (e.g. "Added delete confirmation modal", "Implemented course filter with FlatList")
- [ ] **README edits do not count toward the commit history requirement** — the commits must reflect actual code/feature development
- [ ] Commit history is consistent with the CHANGELOG (see Documentation below)

### Video Demonstration

- [ ] Video uploaded to YouTube
- [ ] Visibility set to **Unlisted**
- [ ] Shared link works correctly
- [ ] Duration within recommended range (**approximately 5–8 minutes**)
- [ ] Video demonstrates **all required Part 3 functionality** (see Section 4)
- [ ] Video shows the app running in **Expo Go** (not just code in an editor)
- [ ] Audio is clear enough to follow the narration

### Documentation — README and CHANGELOG

- [ ] **README file** completed and pushed to the repository
- [ ] README contains: project title, student details, module code, project overview, features implemented, technologies used, **installation and run instructions**, screenshots of the running app, and challenges/lessons learned
- [ ] README includes an **updated flow diagram** for the final navigation — and, if it differs from the Part 1 diagram, a short note explaining why
- [ ] README notes any **design changes since Part 1** (screens added/removed/merged, colour scheme, typography, layout)
- [ ] **CHANGELOG section or file** included, listing **all changes implemented since Part 2**
- [ ] CHANGELOG explicitly documents **what was changed during refactoring** (not just new features)
- [ ] CHANGELOG explicitly notes any **design changes since Part 1** (including colour scheme and flow diagram changes)
- [ ] CHANGELOG aligns with the Git commit history

### References, Attribution & AI Declaration

- [ ] **Reference list is extensive and matches actual dependencies** — every package added to `package.json` is referenced, not just template defaults
- [ ] References include documentation actually consulted, including (where used) `react.dev`, `reactnative.dev`, `reactnavigation.org` or `docs.expo.dev/router`, `docs.expo.dev`, `expo.dev/go`, `typescriptlang.org`, `react-native-picker/picker`, `react-native-async-storage`, Expo Fonts, and the font source(s) used
- [ ] References are **formatted consistently** with access dates where appropriate
- [ ] **In-text code attribution** is present for any block copied or adapted from a tutorial, article, or documentation example — with the source named and linked in a comment at the point of use
- [ ] **In-text AI attribution** is present for AI-assisted code blocks — clearly marked at the point of use with a link to the chat
- [ ] **Declaration of AI Usage** included in the README, stating the tool(s), **linking the publicly visible chat(s)**, explaining what the AI helped with, what you did yourself, and how you reviewed/verified the AI-generated content

---

## Section 2: What Excellent Work Looks Like — Mapped to the Rubric

### 2.1 Manage Menu Items — Edit, Update, Delete (25 Marks)

**Baseline expectation (your three-screen model):**

- [ ] The `MenuManagementScreen` (or equivalent) allows the chef to **select an existing menu item** and **edit** its Dish Name, Description, Course, and Price
- [ ] Changes are **saved** and **immediately reflected** in every screen that displays that item (View Menu, Home statistics)
- [ ] **Delete** removes the item from the array and from all displays
- [ ] A **confirmation prompt** appears before deletion (prevents accidental data loss)

**Strong submissions typically demonstrate:**

- [ ] Edit opens with the **existing values pre-populated** in the input fields (not blank)
- [ ] The **same validation rules** apply to edits as to new items (required fields, price format)
- [ ] A **success message** confirms the edit or delete completed
- [ ] Deletion does not break the list or leave orphaned references
- [ ] If using separate screens for Add vs. Edit/Delete: navigation between them is clear and the chef always knows where they are
- [ ] State management is handled cleanly (e.g. a single `menuItems` array in a parent component or context, with helper functions for add/edit/delete)

**Common mistakes:**

- Editing creates a **duplicate** instead of updating the existing item
- Delete removes the item from the list but the **statistics don't update**
- Edit screen opens with **blank fields** instead of the selected item's data
- No confirmation before delete
- Changes require the app to be **restarted** to appear

---

### 2.2 Search and Filter Menu Items (20 Marks)

**Baseline expectation:**

- [ ] The `ViewMenuScreen` (or equivalent) includes a **search input** that filters by Dish Name
- [ ] A **course filter** (Starter / Main Course / Dessert / All) filters the displayed list
- [ ] A **clear button** (or equivalent) resets search and filter to show all items

**Strong submissions typically demonstrate:**

- [ ] Search updates the list **as the user types** (or on submit, with clear feedback)
- [ ] Filter and search can be **combined** (e.g. search "chicken" within "Main Course")
- [ ] The **empty state** message changes when filters return no results (e.g. "No menu items match your search")
- [ ] The **FlatList** (or equivalent) re-renders correctly without performance issues
- [ ] Clearing the search/filter restores the **full list immediately**
- [ ] The filter UI is intuitive (e.g. segmented buttons, picker, or chips — not a hidden text field)

**Common mistakes:**

- Search is **case-sensitive** and misses items (e.g. "pasta" doesn't find "Pasta")
- Filter and search **override each other** instead of combining
- Clearing search **doesn't restore** the full list
- The filtered list shows **stale data** until the screen is reloaded
- No indication to the user that a filter is active

---

### 2.3 Display Menu Statistics (20 Marks)

**Baseline expectation (your HomeScreen model):**

- [ ] **Total number of menu items** displayed
- [ ] **Average price** of all menu items displayed (formatted correctly, e.g. R 0.00)
- [ ] **Number of items per course** (Starter, Main Course, Dessert) displayed
- [ ] The course counts are **clickable** and navigate to a filtered view of that course (your `ViewMenuScreen` with filter pre-applied)

**Strong submissions typically demonstrate:**

- [ ] Statistics **recalculate automatically** when items are added, edited, or deleted
- [ ] Average price handles the **empty state** (e.g. "No items yet" or R 0.00 — not a crash or "NaN")
- [ ] The statistics are **visually separated** from the menu list (cards, a distinct section, or a dedicated screen) so they are easy to read
- [ ] Course counts use **meaningful labels** ("Starters: 3" not "S: 3")
- [ ] If a dedicated Stats screen is used: navigation to and from it is clear, and it reflects the current data

**Common mistakes:**

- Average price calculated **incorrectly** (e.g. dividing by a hard-coded number instead of the actual count)
- Statistics **don't update** after delete or edit
- Empty state shows **"NaN"** or **"Infinity"** for average price
- Course counts include items that were deleted
- Statistics are **hard-coded** rather than calculated from the array

---

### 2.4 Enhance the Application — Code Quality and Refactoring (25 Marks)

**This criterion rewards evidence of improvement over Part 2.** Strong submissions typically demonstrate:

- [ ] **Reusable components** extracted where appropriate (e.g. a `MenuItemCard` component used in both the full menu and filtered views; a `FormInput` component used in Add and Edit)
- [ ] **Reusable functions** for repeated logic (e.g. `calculateAveragePrice()`, `filterByCourse()`, `validateMenuItem()`)
- [ ] **Meaningful variable and function names** (e.g. `handleDeleteMenuItem` not `handleClick2`; `filteredMenuItems` not `arr2`)
- [ ] **Consistent code organisation** (e.g. `components/`, `screens/`, `utils/`, `types/` folders — or a clearly documented structure)
- [ ] **TypeScript types/interfaces** used meaningfully (e.g. a `MenuItem` interface with `id`, `dishName`, `description`, `course`, `price`)
- [ ] **Validation logic centralised** (not copy-pasted between Add and Edit screens)
- [ ] **Unused code removed** (no commented-out blocks, no unused imports)
- [ ] **CHANGELOG explicitly documents refactoring** — what was moved, renamed, extracted, or reorganised

**Common mistakes:**

- Refactoring is claimed in the README but **not visible in the code** (e.g. still one massive `App.tsx` with all screens and logic)
- "Reusable" components are created but **used only once**
- TypeScript is present but **`any` is used throughout**, defeating the purpose
- No CHANGELOG entry for refactoring — the marker cannot see what changed

---

### 2.5 User Experience (10 Marks)

**Strong submissions typically demonstrate:**

- [ ] **Navigation is intuitive**: the chef can move between Home, Menu Management, and View Menu without confusion; back navigation works correctly
- [ ] **Success and error messages** are meaningful and specific (e.g. "Menu item 'Grilled Chicken' updated successfully" — not just "Success")
- [ ] **Validation** covers all required fields and price format, with messages that explain the problem
- [ ] **Empty states** are handled on every screen (no items in the list; no items matching a filter; no items when statistics are displayed)
- [ ] **Consistent look and feel** across all screens: same fonts, colours, button styles, spacing, and header patterns
- [ ] The app **never crashes** during normal use, even with empty data or rapid interactions
- [ ] If screens were split differently from the baseline (e.g. separate Add screen): the navigation flow still feels **coherent and purposeful**, not fragmented

**Common mistakes:**

- Validation exists on Add but **not on Edit**
- Error messages say **"Error"** or **"Invalid input"** without explaining what is wrong
- Empty state is missing on the **filtered view** (shows a blank screen)
- Navigation **traps** the user (no back button, or back goes to the wrong screen)
- Inconsistent styling between screens (e.g. buttons look different on each screen)

---

### 2.6 Referencing, Code Attribution & AI Declaration (Required — Submission Integrity)

**This criterion is not a tick-box exercise. It is part of demonstrating that you can document your own coding, explain the reason for using code, and show the intended purpose of that code.**

**Reference list quality:**

- [ ] The reference list **covers every dependency you added** — every package in `package.json` beyond the template defaults
- [ ] References include the **documentation you actually consulted**, including (where used):
  - [ ] `https://react.dev/reference/react` — for hooks and core React concepts (`useState`, `useEffect`, `useMemo`, `useCallback`)
  - [ ] `https://reactnative.dev/docs/` — for components used (`View`, `Text`, `FlatList`, `TextInput`, `Pressable`, `StyleSheet`, `Modal`)
  - [ ] `https://reactnavigation.org/docs/` — if using React Navigation (state which navigators)
  - [ ] `https://docs.expo.dev/router/introduction/` — if using Expo Router
  - [ ] `https://docs.expo.dev/` — for SDK reference, `expo install`, `expo-doctor`, `app.json`
  - [ ] `https://expo.dev/go` — for the compatible Expo Go build
  - [ ] `https://www.typescriptlang.org/docs/` — for types, interfaces, union types
  - [ ] `https://github.com/react-native-picker/picker` — if using Picker
  - [ ] `https://react-native-async-storage.github.io/async-storage/` — if using AsyncStorage
  - [ ] `https://docs.expo.dev/develop/user-interface/fonts/` and the font source (Google Fonts or similar) — if using custom fonts
  - [ ] Any tutorial, article, or video followed — with link and access date
- [ ] References are **formatted consistently** with access dates where appropriate

**In-text code attribution:**

- [ ] Any block copied or adapted from a tutorial/article/docs has a **comment at the point of use** naming and linking the source
- [ ] Any AI-assisted block has a **comment at the point of use** identifying it as AI-assisted and linking the chat
- [ ] You can explain, for each attributed block, **why you used it, what it does, and what you changed** — and why
- [ ] There are **no unattributed blocks** that you did not write yourself

Example of acceptable in-text attribution:

```ts
// Adapted from React Native FlatList docs — https://reactnative.dev/docs/flatlist
// Reviewed and modified to fit my MenuItem shape
<FlatList
  data={filteredItems}
  keyExtractor={(item) => item.id}
  renderItem={({ item }) => <MenuItemCard item={item} />}
/>
```

```ts
// AI-assisted (ChatGPT chat: https://chat.openai.com/share/xxxx) — 2026-04-12
// Prompt: "How do I filter an array of objects by two properties?"
// Reviewed against MDN Array.filter and rewritten to match my types
const matches = menuItems.filter((item) => matchesSearch(item) && matchesCourse(item));
```

**AI usage declaration:**

- [ ] README contains a **Declaration of AI Usage** stating which tool(s) were used
- [ ] For each tool, a **link to the publicly visible chat(s)** is provided — not just the tool name
- [ ] The declaration explains **what the AI helped with** and **what you did yourself**
- [ ] The declaration states **how you reviewed and verified** AI-generated content (e.g. checked against docs, rewrote to match data model, tested in emulator)
- [ ] If a chat is **not publicly visible**, it has been made public before submission or the limitation is acknowledged with an explanation

---

## Section 3: Navigation and Screen Architecture — Guidance

Your baseline of **three screens** is a valid and efficient design:

| Screen | Purpose | Key Elements |
|---|---|---|
| **HomeScreen** | Statistics overview + quick navigation | Total items, average price, counts per course (clickable → filtered View Menu) |
| **MenuManagementScreen** | CRUD operations | Add form, list with edit/delete actions (or buttons leading to edit/delete flows) |
| **ViewMenuScreen** | Full menu display with filter | FlatList of items, search input, course filter, clear button |

**Students who choose more screens** (e.g. separate Add screen, separate Edit screen, dedicated Stats screen) are **not penalised** — provided:

- [ ] The **navigation between screens is logical** and the user always knows where they are
- [ ] **Data is shared correctly** across screens (e.g. editing on one screen immediately reflects on the Home statistics)
- [ ] The **PoE rubric criteria are all evidenced** — the marker does not have to hunt for features across excessive fragmentation
- [ ] The **README documents the screen architecture** clearly, so the marker can follow the flow

**Critical guidance for split-screen designs:**

- [ ] If Add and Edit are on separate screens: they must **share validation logic** (not duplicate it inconsistently)
- [ ] If Stats are on a dedicated screen: they must **update when the user returns to that screen** after editing/deleting
- [ ] **Back navigation** must return the user to a sensible place — not to a blank screen or the wrong flow

---

## Section 4: Video Demonstration Expectations (5–8 Minutes)

The purpose of the video is to demonstrate that the **completed application works correctly** and that you understand what you built.

### Your demonstration must include:

- [ ] **Brief introduction**: your name, student number, and the application name
- [ ] **Overview of the interface**: show the Home screen with statistics, and explain the navigation structure
- [ ] **Adding a menu item**: demonstrate the full flow from entry to the item appearing in the list
- [ ] **Editing a menu item**: show pre-populated fields, make a change, save, and show the updated value in the list
- [ ] **Deleting a menu item**: show the confirmation prompt (if implemented) and the item disappearing from all views (including statistics)
- [ ] **Searching**: type a search term and show the list filtering
- [ ] **Filtering by course**: select a course and show only those items
- [ ] **Clearing search/filter**: show all items returning
- [ ] **Displaying statistics**: show total count, average price, and per-course counts — and demonstrate that they **update after an edit or delete**
- [ ] **Navigation flow**: briefly show how the three (or more) screens connect

### Stronger demonstrations include:

- [ ] **Explaining design decisions** (e.g. "I chose a FlatList here because the menu can grow, and FlatList handles large lists efficiently")
- [ ] **Explaining how data is stored** (e.g. "Menu items are stored in a state array in the parent component and passed down; I did not use AsyncStorage because the brief did not require persistence across app restarts")
- [ ] **Explaining the refactoring** (e.g. "After Part 2, I extracted the menu item card into a reusable component used in both the full menu and filtered views")
- [ ] **Speaking confidently** and demonstrating **ownership** of the code — you should be able to answer questions about any part of the implementation
- [ ] **Showing the app in Expo Go**, not just the terminal

### Avoid:

- Demonstrating only **one or two features** and skipping the rest
- **Rushing** through the demonstration with no explanation
- Demonstrating a version of the app that **does not match the submitted code**
- **Reading from a script** without demonstrating the actual app
- **Poor audio** or a video that is too dark/blurry to follow

---

## Section 5: Common Reasons Students Lose Marks in Part 3

### Manage Menu Items

- Edit **duplicates** the item instead of updating it
- Delete **works on the screen but statistics still show the deleted item**
- Edit form **opens blank** instead of pre-populated
- **No confirmation** before delete (rubric expects a reliable, intuitive experience)
- Changes are **not reflected immediately** — user must restart the app

### Search and Filter

- Search is **case-sensitive** or only matches the **start** of the dish name
- Filter and search **do not combine** correctly
- **No clear/reset** function
- Filtered list **doesn't update** when items are edited or deleted
- **Empty state** is missing when no items match

### Statistics

- Average price shows **"NaN"** when the list is empty
- Statistics **don't recalculate** after add/edit/delete
- Course counts are **hard-coded** or incorrect
- Statistics are displayed **without labels** or context (just numbers floating on screen)

### Code Quality / Refactoring

- **No evidence of refactoring** — the code looks identical to Part 2 plus new features bolted on
- **Reusable components** are created but not actually reused
- **TypeScript is not used meaningfully** (`any` everywhere)
- **CHANGELOG does not mention refactoring** — the marker cannot award marks for invisible improvements
- **Commit history is a single commit** or has meaningless messages

### Referencing and Attribution

- **Reference list contains only the tool names** ("ChatGPT", "React Native docs") with no links, no versions, no access dates
- **Reference list does not match actual dependencies** — packages are in `package.json` but not in the references
- **AI declaration names the tool but does not link the chat** — the chat is not publicly visible or not provided
- **No in-text attribution** for copied code — the student cannot point to the source of a block they did not write
- **Copied code is pasted without formatting or comments**, and the student cannot explain what it does
- **Comments say only "source: internet"** — not specific enough to verify

### Expo SDK 54 / Expo Go

- App **does not open in Expo Go** on the marker's emulator because the student used a **newer SDK** on their phone and did not verify compatibility
- Student **assumed** that updating Expo Go on their phone would make it work, without checking the project's SDK version
- **No screenshots** in the README showing the app actually running
- Video shows the **terminal bundling successfully** but never shows the app in Expo Go

### Documentation

- **README is missing** or contains only a title
- **CHANGELOG is missing** or lists features without mentioning refactoring
- **No screenshots** of the running app
- **No run instructions** — the marker cannot verify the app easily
- **Flow diagram is out of date** — the README still shows the Part 1 flow, not the final navigation
- **Design changes** (colour scheme, screens added/merged) are not noted anywhere

### Git

- **Fewer than 6 commits** across Part 3
- Commit messages are **"update"**, **"final"**, **"changes"** — not descriptive
- **All work pushed in one commit** at the end
- **README-only commits** used to inflate the count
- Repository is **the wrong organisational repo** or is inaccessible

---

## Section 6: Professional Software Development Checklist

### Code Quality

- [ ] **Meaningful variable and function names** throughout
- [ ] **Consistent formatting** (indentation, spacing, semicolons — pick a style and stick to it)
- [ ] **TypeScript interfaces** defined for `MenuItem`, navigation params, and component props
- [ ] **Reusable functions** for validation, filtering, and statistics
- [ ] **Reusable components** for menu item display, form inputs, and buttons where appropriate
- [ ] **No unused imports, variables, or commented-out code**
- [ ] **No `console.log` statements** left in production code (except intentional error logging)
- [ ] **Comments where necessary** — explain *why*, not *what*

### File and Folder Organisation

A strong project structure (or equivalent) demonstrates maintainability:

```
src/
├── components/
│   ├── MenuItemCard.tsx
│   ├── FormInput.tsx
│   └── StatCard.tsx
├── screens/
│   ├── HomeScreen.tsx
│   ├── MenuManagementScreen.tsx
│   └── ViewMenuScreen.tsx
├── types/
│   └── index.ts
├── utils/
│   ├── validation.ts
│   └── statistics.ts
└── App.tsx
```

- [ ] If your structure differs, **document it in the README** so the marker can navigate your code

### README Quality

A strong README contains:

- [ ] **Project title**: Chef's Menu Manager — MAST5112 Final PoE
- [ ] **Student name and number**
- [ ] **Module code**: MAST5112
- [ ] **Project overview**: what the app does and who it is for
- [ ] **Features implemented**: a clear list of Part 3 features (edit, delete, search, filter, statistics) plus Part 2 features retained
- [ ] **Technologies used**: React Native, TypeScript, Expo SDK 54, navigation library (React Navigation or Expo Router — state which), Picker library, any other dependencies
- [ ] **Installation and run instructions**: how to clone, install dependencies, and run the app; **explicitly state the SDK version** and how to install the compatible Expo Go
- [ ] **Screenshots**: at least one per screen, showing the app running
- [ ] **Updated flow diagram** for the final navigation (with a note if it differs from Part 1)
- [ ] **Design changes since Part 1** noted (screens added/removed/merged, colour scheme, typography, layout)
- [ ] **Challenges encountered and lessons learned** (recommended)
- [ ] **References** — extensive, matching dependencies and documentation actually used
- [ ] **AI usage declaration** — including **links to publicly visible chats**

### CHANGELOG

The CHANGELOG (in the README or a separate `CHANGELOG.md`) must list:

- [ ] **All new features implemented since Part 2** (edit, delete, search, filter, statistics)
- [ ] **What was refactored** and **why** (e.g. "Extracted `MenuItemCard` component from `ViewMenuScreen` to reuse in filtered view")
- [ ] **Design changes since Part 1** — flow diagram changes, screens added/removed/merged, **colour scheme changes**
- [ ] **Any bugs fixed** and how
- [ ] **Any dependency changes** (e.g. navigation library added, Picker added, async-storage added, fonts added)

A strong CHANGELOG is **specific and verifiable** against the code and commit history.

### References, Code Attribution & AI Declaration

- [ ] **Reference list matches `package.json`** — every added package is referenced, and every reference corresponds to something actually used
- [ ] **Documentation references are specific** (pages/URLs, not just "React Native docs")
- [ ] **Access dates** are included where appropriate
- [ ] **In-text code attribution** is present at the point of use — a comment naming and linking the source, and explaining what was changed
- [ ] **In-text AI attribution** is present at the point of use — a comment identifying the block as AI-assisted and linking the chat
- [ ] **Declaration of AI Usage** in the README includes:
  - [ ] The tool(s) used
  - [ ] **Links to publicly visible chat(s)** — not just the tool name
  - [ ] What the AI helped with, and what you did yourself
  - [ ] How you reviewed and verified the AI-generated content

---

## Section 7: Final Advice

A student can successfully meet the **minimum requirements** for Part 3 with an application that:

- Allows edit, delete, search, filter, and statistics
- Runs without crashing
- Has a README (including an updated flow diagram) and CHANGELOG
- Has a video demonstrating the features
- Has at least 6 commits
- Has an **extensive reference list, in-text code attribution, and an AI declaration with chat links**

However, students aiming for the **higher achievement categories** should focus on:

- **Immediate data consistency** — every change reflects everywhere instantly
- **Clear, specific user feedback** — not generic "Success" messages
- **Evidence of genuine refactoring** — reusable components and functions that are actually reused, documented in the CHANGELOG
- **Expo SDK 54 compatibility verified** — the app opens in Expo Go in the video
- **A well-organised codebase** with TypeScript used meaningfully
- **A professional README** with screenshots, an updated flow diagram, design-change notes, and clear run instructions
- **Documented design evolution** from Part 1 — flow diagram, screens, colour scheme, all noted in README and CHANGELOG
- **A reference list that matches dependencies**, **in-text attribution for every block you did not write**, and an **AI declaration with public chat links**
- **A commit history** that tells the story of Part 3 development
- **A confident, explanatory video** that demonstrates ownership of the project

> **Remember:** Part 3 is the **completed application**. It should look and feel like a finished product compared to the Part 2 prototype. The rubric specifically rewards *improvement* — so make that improvement visible in the code, the CHANGELOG, the commit history, and the video. And make your **sources visible** — the reference list, in-text attribution, and AI declaration are how you prove the work is genuinely yours.

---
