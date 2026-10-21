# Using React Native Context API — A Note-Taking App

## Section 1 — The Problem: Why Do We Need Context?

### 1.1 A familiar pain: prop drilling

Imagine our Note-Taking App has three screens:

- `HomeScreen` — lists all notes
- `AddNoteScreen` — creates a new note
- `SearchScreen` — filters notes by a search term

A naïve approach stores the notes array in `HomeScreen` with `useState` and *passes* it down:

```tsx
// A naïve, painful approach — do not do this
function HomeScreen() {
  const [notes, setNotes] = useState<Note[]>([]);
  return <NoteList notes={notes} onAdd={setNotes} />;
}
```

Now `SearchScreen` also needs `notes`. But `SearchScreen` is a *sibling* of `HomeScreen`, not a child. So the state must be lifted to their common parent (`App`), and passed down through every intermediate component — even ones that don't use it. This is called **prop drilling**, and it becomes painful quickly:

- Every new screen that needs the data requires threading props through its ancestor chain.
- Intermediate components become "plumbing" — they accept props only to pass them on.
- Changes ripple through the component tree.
- A screen deep in the tree cannot *trigger* a change in a sibling without going all the way up and back down again.

### 1.2 What we actually want

We want a single, central place — a **"global store"** — that:

1. **Owns** the notes array as the single source of truth.
2. **Exposes** functions to *add*, *update*, *delete*, *search*, and *filter* notes.
3. Is **accessible from any screen** that asks for it, without prop drilling.
4. **Notifies** every screen when the data changes, so all consumers re-render with the latest data.

This is exactly what React's **Context API** provides.

### 1.3 What "global" means in a React Native app

It is tempting to think of "global" as a file you `import` and whose variables everyone shares — like a module-level singleton. That is **not** how Context works. The word *global* here means:

- **A single Provider component** placed at (or near) the root of the component tree.
- Every component **below** that Provider can `useContext` to reach the shared data.
- Outside that subtree, the context is invisible.

Two consequences worth noting now, because they shape everything that follows:

- **Placement matters.** The Provider must *wrap* the tree. If a screen is not a descendant of the Provider, `useContext` will fail or return the default.
- **It is reactive.** Unlike a module-level variable, changing the state inside the Provider triggers re-renders in all consumers. That is the whole point — the data stays *live*.

### 1.4 Our goal statement

> **Build one context file** (`NotesContext.tsx`) that owns the notes array, exposes add/delete/search/filter operations, and is consumed by every screen. Any screen that reads from it will see the **latest** data and can **manipulate** it, while remaining decoupled from the component hierarchy.

---

## Section 2 — The Three Core Pieces (Mental Model)

Before we look at any code, let's build the mental model. The Context API has **three primary pieces** plus a **fourth idea (`children`)** that ties them together.

### 2.1 `createContext` — the **contract / channel**

`createContext` creates a **channel** — a named contract describing *what shape of data and functions will be shared*.

Think of it as a **blueprint**:

- It does *not* hold any data itself.
- It defines the *type* of value any consumer should expect.
- It creates two things in tandem: a **Provider** component and a consumer path (via `useContext`).

In our app, the contract says: "Anyone who tunes in will receive the notes array plus functions to add, delete, search, and filter."

### 2.2 `Provider` — the **supplier**

The `Provider` is the **actual instance** that *holds* the values and broadcasts them to every descendant.

- It is a *component* — you render it.
- It holds the actual `useState` variables, the `useEffect` hooks, and the manipulation functions.
- It exposes everything through a single `value` prop.
- Everything you render *inside* the `<Provider>` receives access.

Where there is a channel, there must be a **broadcaster**.

### 2.3 `useContext` — the **consumer hook**

A screen calls `useContext(NotesContext)` (or, in our design, a typed `useNotes()` hook) to **tune in** and receive whatever the Provider is currently broadcasting.

Key points:

- Calling `useContext` **subscribes** the component to the context.
- When the Provider's `value` changes, all subscribed components **re-render** with the new value.
- `useContext` does *not* let the screen *change* the context directly — the screen *calls the functions* the context exposes, and those functions change state *inside* the Provider, which then re-broadcasts.

### 2.4 `children` — the **sub-tree inside the Provider**

Every Provider is written as a wrapper:

```tsx
<NotesProvider>
  {/* everything here is "inside" the provider */}
  <AppNavigator />
</NotesProvider>
```

The JSX *inside* the Provider is passed to it as a special prop called `children`. Rendering `{children}` inside the Provider's return value is what places the sub-tree beneath it — and therefore what gives the sub-tree access to the context.

If you forget to render `{children}`, the sub-tree **disappears** (nothing renders) — and, unsurprisingly, nothing can consume the context. We'll revisit this in Section 5.

### 2.5 The full picture (visual)

```
        ┌───────────────────────────────┐
        │  NotesContext (createContext) │  ← blueprint / channel
        └───────────────────────────────┘
                        ▲
                        │  typed by
                        │
        ┌───────────────┴────────────────────────────┐
        │  NotesProvider (component)                 │
        │  ┌───────────────┐  ┌───────────────────┐  │
        │  │ useState      │  │ useEffect         │  │
        │  │ notes array   │  │ (load, sync)      │  │
        │  └───────────────┘  └───────────────────┘  │
        │  ┌───────────────────────────────────────┐ │
        │  │ arrow functions: addNote, deleteNote… │ │
        │  └───────────────────────────────────────┘ │
        │                                            │
        │  <NotesContext.Provider value={value}>     │  ← broadcaster
        │      {children}                            │  ← sub-tree
        │  </NotesContext.Provider>                  │
        └────────────────────────────────────────────┘
                        ▲
                        │  useContext(NotesContext)
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   HomeScreen      AddNoteScreen    SearchScreen   ← consumers
```

### 2.6 The critical rule: ancestry

A component can only consume a context if it is **a descendant of the Provider** in the render tree. If you render `<HomeScreen />` *outside* `<NotesProvider>`, `useContext(NotesContext)` returns the context's **default value** — which, in our design, will be `undefined`, and we'll make the typed hook **throw an error** to catch this mistake loudly.

---

## Section 3 — Why TypeScript Changes the Story

TypeScript is not just decoration here — it fundamentally shapes how we build the context. Let's walk through the reasoning.

### 3.1 The Interface as a **contract**

In plain JavaScript, `createContext()` returns a context whose value can be *anything*, and any consumer can call *any* method on it. Bugs sneak in silently.

In TypeScript, we define an **interface** that captures exactly *what the context promises to expose*:

- The **data** (the notes array, the search term, the loading flag, etc.).
- The **functions** (add, delete, update, set the search term, derive a filtered list, etc.).

```tsx
export interface Note {
  id: string;
  createdAt: string;     // ISO 8601 timestamp
  title: string;
  body: string;
  category: string;
}

export interface NotesContextValue {
  // data
  notes: Note[];
  searchTerm: string;
  filteredNotes: Note[];      // derived, but exposed for convenience

  // manipulation
  addNote: (title: string, body: string, category: string) => void;
  deleteNote: (id: string) => void;
  updateNote: (id: string, updates: Partial<Omit<Note, 'id' | 'createdAt'>>) => void;
  setSearchTerm: (term: string) => void;
  getNoteById: (id: string) => Note | undefined;
}
```

This interface is the **specification**. If a screen tries to call `context.addTask(...)` — a function that doesn't exist — TypeScript flags it *at compile time*, before the app even runs. If you later rename a field, the compiler tells you every screen that needs updating.

### 3.2 Where the interface fits

The interface is defined **first**, then passed to `createContext` as a type parameter:

```tsx
export const NotesContext = createContext<NotesContextValue | undefined>(undefined);
```

Note the `| undefined`. Why?

- `createContext` requires a default value. If we passed a *fake* `NotesContextValue` (with dummy functions), then consumers *outside* the Provider would silently receive harmless-looking garbage instead of a clear error.
- Passing `undefined` means: *"there is no legitimate value outside the Provider"* — a fact we can exploit for safety.

### 3.3 The custom hook that guards against misuse

Because the type is `NotesContextValue | undefined`, the raw `useContext` call returns a possibly-`undefined` value. So we wrap it in a **custom hook** that:

1. Calls `useContext(NotesContext)`.
2. Throws a helpful error if the result is `undefined`.
3. Otherwise returns the **narrowed** type — `NotesContextValue` (no `undefined`).

```tsx
export function useNotes(): NotesContextValue {
  const context = useContext(NotesContext);
  if (context === undefined) {
    throw new Error('useNotes must be used within a NotesProvider');
  }
  return context;
}
```

Now, in *any* screen:

```tsx
const { notes, addNote } = useNotes();   // fully typed, non-nullable
```

And if someone accidentally forgets the Provider, they get a **clear runtime error** at the exact moment the offending component mounts — rather than mysterious `undefined is not a function` crashes far away.

### 3.4 The takeaway

TypeScript's role in the context file is to:

- **Define the contract** (the interface) before writing behaviour.
- **Prevent the context from being "empty"** outside the Provider (the `| undefined` type + throwing hook).
- **Give every consumer autocomplete** for fields and functions — a first-class benefit for a class audience learning the pattern.

---

## Section 4 — Anatomy of the Context File (Walkthrough)

Now we open the file and walk through it, **block by block**, before showing it whole. We'll open `src/context/NotesContext.tsx`.

### 4.1 Imports

```tsx
import React, {
  createContext,
  useContext,
  useState,
  useEffect,
  useMemo,
  ReactNode,
} from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';
```

- **`createContext`, `useContext`** — the core of the context mechanism.
- **`useState`** — the local state *inside* the Provider (source of truth).
- **`useEffect`** — side effects (initial load, persistence) — the subject of Section 7.
- **`useMemo`** — we introduce it *briefly* here and return to it in Section 6 (with a note that its full study is out of scope for this unit).
- **`ReactNode`** — the TypeScript type for the `children` prop.
- **`AsyncStorage`** — optional persistence so notes survive app restarts; a nice touch that lets `useEffect` do real work.

> **Install note:** this example uses `@react-native-async-storage/async-storage`, installed with `npx expo install @react-native-async-storage/async-storage`. If you'd prefer to keep the example dependency-free, you can replace AsyncStorage with a simple in-memory store — but the load-on-mount effect is a better illustration of `useEffect([])`.

### 4.2 Types — `Note` and `NotesContextValue`

```tsx
export interface Note {
  id: string;
  createdAt: string;       // ISO 8601 timestamp
  title: string;
  body: string;
  category: string;
}

export interface NotesContextValue {
  notes: Note[];
  searchTerm: string;
  filteredNotes: Note[];

  addNote: (title: string, body: string, category: string) => void;
  deleteNote: (id: string) => void;
  updateNote: (
    id: string,
    updates: Partial<Omit<Note, 'id' | 'createdAt'>>
  ) => void;
  setSearchTerm: (term: string) => void;
  getNoteById: (id: string) => Note | undefined;
}
```

Key choices, spelled out:

- **`Note.createdAt`** is a string (ISO 8601). Storing `Date` objects in state looks attractive but is an anti-pattern when you persist to storage — you'd have to rehydrate them on load. Strings are simpler and sort correctly with `.localeCompare()`. Screens can format the display date with `new Date(note.createdAt).toLocaleString()`.
- **`id`** is a string. A common approach is `Date.now().toString()` prefixed by a random suffix to reduce collisions in rapid additions: `` `${Date.now()}-${Math.random().toString(36).slice(2, 8)}` ``.
- **`Partial<Omit<Note, 'id' | 'createdAt'>>`** — the `updateNote` function forbids callers from changing `id` or `createdAt`. This is how the interface itself encodes a business rule.
- **`filteredNotes`** is exposed *on the context* so every screen that needs a filtered view uses the *same* logic — no duplicated filter code.

### 4.3 Creating the context

```tsx
export const NotesContext = createContext<NotesContextValue | undefined>(undefined);
```

This is the **channel**. It carries the type contract via the generic parameter, and its default of `undefined` is what makes the throwing `useNotes` hook (Section 4.5) possible.

Notice what this line is *not* doing: it holds no state, no functions, no data. It is pure **shape**. All behaviour lives in the Provider.

### 4.4 The Provider component — signature and state

```tsx
interface NotesProviderProps {
  children: ReactNode;
}

export function NotesProvider({ children }: NotesProviderProps) {
  const [notes, setNotes] = useState<Note[]>([]);
  const [searchTerm, setSearchTermState] = useState<string>('');

  // ... (effects and functions — next blocks)
}
```

Observations:

- The Provider's props are **only `children`** — that is the whole point of a context Provider: it *wraps* a sub-tree and broadcasts to it.
- **Two pieces of state**: `notes` (the data) and `searchTerm` (search input). Both live *inside the Provider* — no screen owns them.
- Note the slight naming difference: state setter is `setSearchTermState` locally, but the *exposed* API calls the function `setSearchTerm` (below). This avoids a name clash while keeping the public contract readable.

### 4.5 The `useEffect` hooks

We will cover these in full detail in **Section 7** — for now, note their *shape*:

```tsx
  // Effect A — empty dependency array: runs once on mount
  useEffect(() => {
    (async () => {
      const raw = await AsyncStorage.getItem('@notes');
      if (raw) setNotes(JSON.parse(raw));
    })();
  }, []);

  // Effect B — runs whenever `notes` changes (after the initial mount)
  useEffect(() => {
    AsyncStorage.setItem('@notes', JSON.stringify(notes)).catch(() => {});
  }, [notes]);

  // Effect C — runs whenever `searchTerm` changes
  useEffect(() => {
    // e.g. log or analytics on the current search term
    console.log('Search term changed to:', searchTerm);
  }, [searchTerm]);
```

The full discussion of *why* Effect A runs only once, *why* Effect B runs after every `notes` update, and *why* a screen never "calls" these — is coming in Section 7. Consider this a placeholder.

### 4.6 The arrow functions (data manipulation)

```tsx
  const addNote = (title: string, body: string, category: string): void => {
    const newNote: Note = {
      id: `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`,
      createdAt: new Date().toISOString(),
      title: title.trim(),
      body: body.trim(),
      category,
    };
    setNotes((prev) => [newNote, ...prev]);   // newest first
  };

  const deleteNote = (id: string): void => {
    setNotes((prev) => prev.filter((n) => n.id !== id));
  };

  const updateNote = (
    id: string,
    updates: Partial<Omit<Note, 'id' | 'createdAt'>>
  ): void => {
    setNotes((prev) =>
      prev.map((n) => (n.id === id ? { ...n, ...updates } : n))
    );
  };

  const setSearchTerm = (term: string): void => {
    setSearchTermState(term);
  };

  const getNoteById = (id: string): Note | undefined => {
    return notes.find((n) => n.id === id);
  };
```

Points to highlight for students:

- **Immutability** — every function produces a *new* array via `[newNote, ...prev]`, `filter`, or `map`. Never `prev.push(...)` or `n.title = ...`. React relies on reference changes to know the state changed.
- **Functional setState** — `setNotes(prev => …)` is used so these functions never read a stale `notes` from closure.
- **`updateNote` uses spread** to overlay only the fields the caller passed — thanks to `Partial`.
- **`getNoteById`** closes over `notes` — it reads the *current* array at call time. Fine for a read-only helper; note the caveat: if `getNoteById` is called from a memoised child with the same closure, it reflects the `notes` value captured when the function was created. (In our case, the `value` object is re-created on every render, so this is safe — we'll discuss this trade-off briefly in Section 6.)

### 4.7 Derived value: `filteredNotes`

```tsx
  const filteredNotes = useMemo(() => {
    const term = searchTerm.trim().toLowerCase();
    if (term === '') return notes;
    return notes.filter(
      (n) =>
        n.title.toLowerCase().includes(term) ||
        n.body.toLowerCase().includes(term) ||
        n.category.toLowerCase().includes(term)
    );
  }, [notes, searchTerm]);
```

**Brief introduction to `useMemo`** — and then we step away:

`useMemo` caches the result of a computation so that it is only recomputed when one of its **dependencies** (`[notes, searchTerm]`) has changed. Here, the filtering of the notes array only needs to happen when the list itself changes or the search term changes — not when, say, an unrelated state variable changes. So `useMemo` is a performance optimisation: *same inputs → same cached output*.

> **Side-step:** `useMemo` is out of scope for a thorough study here. For this unit, understand the *dependency array concept* — the same list that governs `useEffect`. The important part of these notes is Context; `useMemo` is just protecting the derived `filteredNotes` value from being recomputed too often. We will mention it again only in the pitfalls section (Section 10) for re-render performance.

### 4.8 The `value` object — what gets broadcast

```tsx
  const value: NotesContextValue = {
    notes,
    searchTerm,
    filteredNotes,
    addNote,
    deleteNote,
    updateNote,
    setSearchTerm,
    getNoteById,
  };
```

This is **the single object** that every consumer receives. It bundles:

- **Read-only data** (`notes`, `searchTerm`, `filteredNotes`) — for display and filtering.
- **Functions** — for manipulating the data.

The `: NotesContextValue` annotation is *crucial*. It forces TypeScript to verify this object **matches the interface**. If you add a function to the interface but forget to include it here, or you mistype a field, the compiler catches it immediately — the interface and the implementation stay in lockstep.

### 4.9 Returning the Provider

```tsx
  return (
    <NotesContext.Provider value={value}>
      {children}
    </NotesContext.Provider>
  );
}
```

- `NotesContext.Provider` is *not* the same thing as our `NotesProvider` component — it is the *primitive* Provider component that `createContext` supplied, and it's the one that actually broadcasts `value` down the tree.
- **`{children}`** is the sub-tree passed in from `App.tsx`. Everything inside — including every screen in our navigator — becomes a consumer.
- If you omit `{children}`, screens will not render. If you render `{children}` *outside* the `.Provider`, they will render but `useNotes()` will throw. This is the most common beginner mistake, and it is exactly what Section 5 unpacks.

### 4.10 The custom hook

```tsx
export function useNotes(): NotesContextValue {
  const context = useContext(NotesContext);
  if (context === undefined) {
    throw new Error('useNotes must be used within a NotesProvider');
  }
  return context;
}
```

This is the **public entry point** for every screen. Screens never call `useContext(NotesContext)` directly — they call `useNotes()`. Benefits:

- **Enforces the Provider** at runtime with a clear error.
- **Narrows the type** so screens see `NotesContextValue`, never `… | undefined`.
- **Provides a seam for future refactors** — if we later split the context, only this hook changes.

### 4.11 The whole file, at a glance

```tsx
import React, {
  createContext,
  useContext,
  useState,
  useEffect,
  useMemo,
  ReactNode,
} from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

export interface Note {
  id: string;
  createdAt: string;
  title: string;
  body: string;
  category: string;
}

export interface NotesContextValue {
  notes: Note[];
  searchTerm: string;
  filteredNotes: Note[];

  addNote: (title: string, body: string, category: string) => void;
  deleteNote: (id: string) => void;
  updateNote: (
    id: string,
    updates: Partial<Omit<Note, 'id' | 'createdAt'>>
  ) => void;
  setSearchTerm: (term: string) => void;
  getNoteById: (id: string) => Note | undefined;
}

export const NotesContext = createContext<NotesContextValue | undefined>(undefined);

interface NotesProviderProps {
  children: ReactNode;
}

export function NotesProvider({ children }: NotesProviderProps) {
  const [notes, setNotes] = useState<Note[]>([]);
  const [searchTerm, setSearchTermState] = useState<string>('');

  // Effect A — load once on mount
  useEffect(() => {
    (async () => {
      const raw = await AsyncStorage.getItem('@notes');
      if (raw) setNotes(JSON.parse(raw));
    })();
  }, []);

  // Effect B — persist whenever notes change
  useEffect(() => {
    AsyncStorage.setItem('@notes', JSON.stringify(notes)).catch(() => {});
  }, [notes]);

  // Effect C — react to search term changes
  useEffect(() => {
    console.log('Search term changed to:', searchTerm);
  }, [searchTerm]);

  const addNote = (title: string, body: string, category: string): void => {
    const newNote: Note = {
      id: `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`,
      createdAt: new Date().toISOString(),
      title: title.trim(),
      body: body.trim(),
      category,
    };
    setNotes((prev) => [newNote, ...prev]);
  };

  const deleteNote = (id: string): void => {
    setNotes((prev) => prev.filter((n) => n.id !== id));
  };

  const updateNote = (
    id: string,
    updates: Partial<Omit<Note, 'id' | 'createdAt'>>
  ): void => {
    setNotes((prev) =>
      prev.map((n) => (n.id === id ? { ...n, ...updates } : n))
    );
  };

  const setSearchTerm = (term: string): void => {
    setSearchTermState(term);
  };

  const getNoteById = (id: string): Note | undefined => {
    return notes.find((n) => n.id === id);
  };

  const filteredNotes = useMemo(() => {
    const term = searchTerm.trim().toLowerCase();
    if (term === '') return notes;
    return notes.filter(
      (n) =>
        n.title.toLowerCase().includes(term) ||
        n.body.toLowerCase().includes(term) ||
        n.category.toLowerCase().includes(term)
    );
  }, [notes, searchTerm]);

  const value: NotesContextValue = {
    notes,
    searchTerm,
    filteredNotes,
    addNote,
    deleteNote,
    updateNote,
    setSearchTerm,
    getNoteById,
  };

  return (
    <NotesContext.Provider value={value}>
      {children}
    </NotesContext.Provider>
  );
}

export function useNotes(): NotesContextValue {
  const context = useContext(NotesContext);
  if (context === undefined) {
    throw new Error('useNotes must be used within a NotesProvider');
  }
  return context;
}
```

### 4.12 Recap of Section 4

| Block | Purpose |
|---|---|
| Imports | Core hooks + AsyncStorage |
| Interfaces (`Note`, `NotesContextValue`) | The contract |
| `createContext<…｜undefined>(undefined)` | The channel, with a `undefined` default for safety |
| `NotesProvider({ children })` | The supplier, wrapping the sub-tree |
| `useState` ×2 | Source of truth for notes + searchTerm |
| `useEffect` ×3 | Side effects — load once, persist on change, react to search |
| Arrow functions | Manipulation (`add`, `delete`, `update`, `setSearchTerm`, `getNoteById`) |
| `useMemo` | Derives `filteredNotes` (briefly introduced) |
| `value` object typed as `NotesContextValue` | The broadcast |
| `<NotesContext.Provider value={value}>{children}</…>` | Return |
| `useNotes()` | Guarded consumer hook |

---

## Section 5 — The `children` Prop: How the Provider "Wraps" the App

### 5.1 What `children` really is

In JSX, whatever you write **between an opening and closing tag** is passed to that component as a special prop called `children`.

```tsx
<NotesProvider>
  <AppNavigator />
</NotesProvider>
```

Here, `<AppNavigator />` is the `children` of `NotesProvider`. So `NotesProvider` *receives* the navigator as a prop, and decides *where* in its own output to render it.

In TypeScript, we type `children` as `ReactNode` — the widest type for "anything React can render":

```tsx
interface NotesProviderProps {
  children: ReactNode;
}
```

`ReactNode` covers JSX elements, strings, numbers, arrays, fragments, `null`, and so on. For a Provider this is exactly what we want — it should be able to wrap *anything*.

### 5.2 Why `{children}` sits *inside* the `.Provider`

Look again at the return statement of our Provider:

```tsx
return (
  <NotesContext.Provider value={value}>
    {children}
  </NotesContext.Provider>
);
```

Two structural facts are doing the work here:

1. **`{children}`** is rendered *as a child* of `NotesContext.Provider`.
2. Therefore everything inside the sub-tree passed to `NotesProvider` sits **below** `NotesContext.Provider` in the render tree.

The Context API works by **ancestry**. The `NotesContext.Provider` broadcasts a value to *every descendant*, no matter how deep. Rendering `{children}` inside the `.Provider` is what places the screens into that descendant set — and therefore what makes `useNotes()` succeed there.

If you move `{children}` **outside** `NotesContext.Provider`, or forget it entirely:

```tsx
// ❌ WRONG — children rendered outside the Provider
return (
  <>
    <NotesContext.Provider value={value} />
    {children}
  </>
);
```

…screens will still **render** (they exist in the output), but they will not be **subscribers** — because they are *siblings* of the Provider, not *descendants*. Calling `useNotes()` from them hits the `undefined` default and throws our helpful error. This is the classic first-day-of-Context mistake; it's worth showing students this broken version once, deliberately, so they recognise the symptom.

### 5.3 Where to place the Provider in `App.tsx`

Rule of thumb: **as high as possible, just below other app-level wrappers you can't move** (theme providers, gesture handler roots, navigation containers *if* they need to be above).

In our Note-Taking App, `NavigationContainer` is *not* a context consumer — it just holds navigation state. So we can wrap it with `NotesProvider` from either side. We'll choose to place `NotesProvider` **outside** `NavigationContainer`, so the whole navigation tree is a descendant:

```tsx
// App.tsx
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { NotesProvider } from './src/context/NotesContext';
import HomeScreen from './src/screens/HomeScreen';
import AddNoteScreen from './src/screens/AddNoteScreen';
import SearchScreen from './src/screens/SearchScreen';

export type RootStackParamList = {
  Home: undefined;
  AddNote: undefined;
  Search: undefined;
};

const Stack = createNativeStackNavigator<RootStackParamList>();

export default function App() {
  return (
    <NotesProvider>
      <NavigationContainer>
        <Stack.Navigator initialRouteName="Home">
          <Stack.Screen name="Home" component={HomeScreen} />
          <Stack.Screen name="AddNote" component={AddNoteScreen} />
          <Stack.Screen name="Search" component={SearchScreen} />
        </Stack.Navigator>
      </NavigationContainer>
    </NotesProvider>
  );
}
```

Every screen we define is now a **descendant** of `NotesContext.Provider`. Whether a screen is *pushed* at navigation time or is the current stack screen, it is always *inside* the tree wrapped by `NotesProvider` — so `useNotes()` always resolves.

### 5.4 Common mistakes (worth showing explicitly)

```tsx
// ❌ Mistake 1: Provider rendered but children forgotten
return <NotesContext.Provider value={value} />;
// → nothing below it renders; the app is blank.

// ❌ Mistake 2: Provider rendered but children outside
return (
  <>
    <NotesContext.Provider value={value} />
    {children}
  </>
);
// → screens render but useNotes() throws.

// ❌ Mistake 3: Provider placed inside a screen
function HomeScreen() {
  return (
    <NotesProvider>
      {/* only this screen's subtree is covered */}
    </NotesProvider>
  );
}
// → other screens (siblings in the navigator) are NOT covered.
```

The correct pattern is the one in Section 5.3 — `NotesProvider` wrapping the whole navigator.

---

## Section 6 — Inside the Provider: State + Functions

### 6.1 State as the single source of truth

Inside `NotesProvider` we declared exactly two pieces of state:

```tsx
const [notes, setNotes] = useState<Note[]>([]);
const [searchTerm, setSearchTermState] = useState<string>('');
```

These are the **only** mutable data in our shared model. Everything the app displays or derives flows from these two. This has important consequences:

- **Screens do not own notes.** Screens are *readers* and *callers*, not *owners*. If a screen held its own `useState` for notes, we'd be back to disconnected data and prop drilling.
- **Two screens editing at once?** No conflict — there's one array, and every mutation goes through the Provider's functions.
- **Restart the app?** Effect A rehydrates `notes` from storage; the state setter fires; every consumer updates.

### 6.2 Manipulation functions — the shape and the rules

All our functions have the same skeleton: they read the parameters, call the setter with a **functional updater**, and never touch the array directly.

```tsx
const addNote = (title: string, body: string, category: string): void => {
  const newNote: Note = {
    id: `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`,
    createdAt: new Date().toISOString(),
    title: title.trim(),
    body: body.trim(),
    category,
  };
  setNotes((prev) => [newNote, ...prev]);   // newest first
};
```

Two things deserve emphasis:

**(a) Functional updater (`prev => …`)** — we never write `setNotes([newNote, ...notes])`. Even though it usually works, it reads `notes` *from the closure*. If two updates happen in the same tick (e.g. `addNote` called twice from two very fast taps), the second might operate on a stale `notes`. Using `prev` ensures we always see the **latest** state. The same rule applies to `deleteNote` and `updateNote`.

**(b) Immutability** — every function produces a **new array**:

| Function | Produces |
|---|---|
| `addNote` | A new array with `newNote` at the front |
| `deleteNote` | A new array without the matching `id` |
| `updateNote` | A new array with one object replaced by a merged copy |

Never `prev.push(...)`, never `note.title = …`, never `delete note['id']`. React compares by reference — if the array reference didn't change, the UI won't know to re-render.

```tsx
const deleteNote = (id: string): void => {
  setNotes((prev) => prev.filter((n) => n.id !== id));
};

const updateNote = (
  id: string,
  updates: Partial<Omit<Note, 'id' | 'createdAt'>>
): void => {
  setNotes((prev) =>
    prev.map((n) => (n.id === id ? { ...n, ...updates } : n))
  );
};
```

`updateNote` uses `{ ...n, ...updates }` to produce a **new object** — again, never mutating `n` in place. And `Partial<Omit<Note, 'id' | 'createdAt'>>` ensures callers can only change `title`, `body`, and `category` — not the identifiers.

### 6.3 Derived data — `filteredNotes`

Sometimes we don't want to expose raw state, but a **view** of it:

```tsx
const filteredNotes = useMemo(() => {
  const term = searchTerm.trim().toLowerCase();
  if (term === '') return notes;
  return notes.filter(
    (n) =>
      n.title.toLowerCase().includes(term) ||
      n.body.toLowerCase().includes(term) ||
      n.category.toLowerCase().includes(term)
  );
}, [notes, searchTerm]);
```

The reason we expose this from the context (rather than letting each screen filter on its own):

- **Consistency** — every screen filters identically.
- **Single source of truth for behaviour** — refine the filter in one place, all screens benefit.
- **Performance** — computed once per (notes, searchTerm) change, not once per screen render.

The `useMemo` wrapper is briefly introduced here as a **performance optimisation only**: it caches `filteredNotes` between renders, recomputing only when `notes` or `searchTerm` changes. We are **not** studying `useMemo` in depth; the only concept to carry forward is the **dependency array** (`[notes, searchTerm]`), which is the same concept `useEffect` uses (Section 7).

### 6.4 A note on `getNoteById` and closures

```tsx
const getNoteById = (id: string): Note | undefined => {
  return notes.find((n) => n.id === id);
};
```

This closes over the current `notes` — it captures whatever value was in scope when the function was defined. Because the Provider **re-renders** every time `notes` changes, and each render creates a fresh `getNoteById` referring to the new `notes`, calls from consumers always see the current list.

The risk only appears in a **memoised** child that receives the function as a prop and is *not* re-rendered when `notes` changes (because `React.memo` skips it) — such a child could hold a stale `getNoteById`. We are not using `React.memo` in this app, so this is not an issue here. But it's worth naming as a real-world caveat in Section 10.

### 6.5 The `value` object — recreating on each render

```tsx
const value: NotesContextValue = {
  notes,
  searchTerm,
  filteredNotes,
  addNote,
  deleteNote,
  updateNote,
  setSearchTerm,
  getNoteById,
};
```

A common question: *why not `useMemo` this object too?*

Because **everything in it changes often** — `notes` after every mutation, `searchTerm` after every keystroke in the search field. A `useMemo` on `value` would almost always be invalidated. Additionally, each render produces a new `value` object anyway, and that's *desirable*: it's how consumers know to re-render.

The only scenario where memoising `value` matters is when the Provider has other, **unrelated** state that could trigger re-renders. If we later added a `theme` state to this same Provider, we'd want to memoise the value to avoid re-rendering every note consumer on a theme change. This is precisely why some teams **split** their contexts (e.g. `NotesContext` and `ThemeContext`) — a topic we mention but do not need to implement here. For a single-context Notes app, recreating `value` each render is idiomatic and safe.

### 6.6 How the broadcast reaches a screen

When any function mutates state, the chain is:

1. `setNotes(...)` marks the Provider's state as changed.
2. React re-renders `NotesProvider`.
3. During that render, a **new** `value` object is built (containing the new `notes`, new `filteredNotes`, and — freshly — the functions).
4. `NotesContext.Provider` is given `value={newValue}`.
5. React compares the previous `value` object reference with the new one. Different → **all consumers re-render**.
6. Each consumer receives the new values via `useNotes()`.

This is the mechanism that makes "the screen that calls the function" cause "every other screen to update" — without any explicit notification code.

---

## Section 7 — `useEffect` in the Provider: The "Invisible" Execution

This is the section where the "aha" needs to land. A screen *never* calls our `useEffect` code — yet the code runs. Let's unpack why, when, and how.

### 7.1 What `useEffect` is (one sentence)

`useEffect(callback, deps)` tells React: **"After this component renders (and after the browser/UI has committed that render), run this `callback` — but only under the conditions described by `deps`."**

Three crucial facts to internalise:

1. **`useEffect` runs *after* render, not during it.** Your render function should be pure — no side effects, no `AsyncStorage` calls, no timers, no subscriptions. Effects are the sanctioned place for that work.
2. **`useEffect` is tied to a component.** React runs the effect when *the component that declared it* renders, re-renders, mounts, or unmounts. In our app, that component is `NotesProvider`.
3. **The dependency array is the "when".** It doesn't affect *what* the effect does — only *how often* it runs.

### 7.2 The three dependency-array forms

| Form | When does it run? |
|---|---|
| `useEffect(fn)` — no dependency array | After **every** render (mount + every re-render) |
| `useEffect(fn, [])` — empty array | **Once**, after the initial mount (plus cleanup on unmount, if provided) |
| `useEffect(fn, [a, b])` — with dependencies | On mount, **and** again whenever `a` or `b` **changes** (React uses `Object.is` comparison — for primitives, standard equality; for objects/arrays, reference identity) |

A subtle but important "why": **React does not know what your effect *does*** — it cannot guess that your effect reads `searchTerm`. You *declare* that dependency by listing it. If you forget, React will not re-run the effect when `searchTerm` changes. This is the "dependency array is a contract" idea: you promise to list everything the effect reads from the surrounding scope.

### 7.3 Our three effects — annotated

```tsx
// Effect A — [] — runs once when NotesProvider first mounts
useEffect(() => {
  (async () => {
    const raw = await AsyncStorage.getItem('@notes');
    if (raw) setNotes(JSON.parse(raw));
  })();
}, []);
```

- On the very first mount of `NotesProvider` (i.e. when the app starts), this runs *once*.
- It kicks off a fetch from persistent storage and, once resolved, calls `setNotes(...)`.
- Calling `setNotes` re-renders the Provider with the loaded array. Every consumer receives the restored notes.
- **Why `[]`?** We only want to load from storage **once**, on startup. If we left the array empty-but-present, it would run every render — and each `setNotes` inside would trigger another render → another effect → another `setNotes`… an infinite loop.

```tsx
// Effect B — [notes] — runs on mount, then whenever `notes` changes
useEffect(() => {
  AsyncStorage.setItem('@notes', JSON.stringify(notes)).catch(() => {});
}, [notes]);
```

- On mount, this runs and writes the (likely empty) `notes` array to storage.
- Effect A's `setNotes` updates `notes`, which triggers a re-render → Effect B re-runs → the freshly-loaded array is written back — which is redundant but harmless.
- From then on, every call to `addNote`/`deleteNote`/`updateNote` changes `notes` → Effect B re-runs → persistence stays in sync.
- **Why `[notes]`?** Persistence is only meaningful when the *content* changed. Other state changes (e.g. `searchTerm`) should *not* trigger a disk write.

```tsx
// Effect C — [searchTerm] — runs on mount, then whenever `searchTerm` changes
useEffect(() => {
  console.log('Search term changed to:', searchTerm);
}, [searchTerm]);
```

- Every time the search term changes (typically, on every keystroke in the search box), this logs.
- In a real app this might be where you debounce and fire an analytics event, or trigger a server-side search.
- **Why `[searchTerm]`?** It reads `searchTerm` — so `searchTerm` must be listed. It doesn't read `notes`, so `notes` is not listed.

### 7.4 The mount timeline (visual)

Consider a cold start, `notes` initially `[]`, `searchTerm` initially `''`:

```
Step 1  App mounts → NotesProvider is created with:
            notes = []
            searchTerm = ''
Step 2  Render of NotesProvider runs:
            value = { notes: [], searchTerm: '', … }
            <NotesContext.Provider value={value}>{children}</…>
Step 3  Commit: React paints the screen. Consumers receive value for the first time.
Step 4  Effects run, in declaration order:
            Effect A: []  → fetch AsyncStorage → schedules setNotes([…restored])
            Effect B: [notes] → runs because dependencies are new on mount
                               → AsyncStorage.setItem(JSON.stringify([]))
            Effect C: [searchTerm] → runs because dependencies are new on mount
                               → log('Search term changed to: ', '')
Step 5  Effect A's promise resolves → setNotes([restoredNotes])
Step 6  State change → NotesProvider re-renders:
            value = { notes: [restoredNotes], searchTerm: '', … }
Step 7  Commit: consumers re-render with the restored notes.
Step 8  Effects re-run only for changed dependencies:
            Effect A: [] → does NOT re-run (empty deps, already ran on mount)
            Effect B: [notes] → re-runs (notes changed)
                               → AsyncStorage.setItem(JSON.stringify(restoredNotes))
            Effect C: [searchTerm] → does NOT re-run (searchTerm unchanged)
Step 9  Stable state: no further effects scheduled.
```

Now the user types `"meeting"` into the search box:

```
Step 1  SearchScreen calls setSearchTerm('meeting') via the context
Step 2  NotesProvider re-renders:
            value = { notes: [restoredNotes], searchTerm: 'meeting',
                      filteredNotes: [filtered], … }
Step 3  Commit: consumers re-render. SearchScreen displays filtered notes.
Step 4  Effects with changed dependencies re-run:
            Effect A: [] → no
            Effect B: [notes] → no (notes unchanged)
            Effect C: [searchTerm] → yes → logs 'meeting'
```

Now the user adds a note:

```
Step 1  AddNoteScreen calls addNote(title, body, category)
Step 2  setNotes(prev => [newNote, ...prev]) fires
Step 3  NotesProvider re-renders; value rebuilt with new notes + new filteredNotes
Step 4  Commit: all consumers re-render
Step 5  Effect A: [] → no
        Effect B: [notes] → yes → persists new array
        Effect C: [searchTerm] → no
```

### 7.5 Why the screen "never calls" the effect — but it happens anyway

A very natural student question: *"I never call `AsyncStorage.setItem` from my screen — how does the write happen when I add a note?"*

The answer has three parts:

1. **Effects are declared inside the Provider**, and React runs them **whenever the Provider renders or re-renders** — subject to the dependency array. The Provider's re-render is triggered by its state changing, and its state changes because a **screen called a context function**.
2. The context function (`addNote`, `setSearchTerm`, …) doesn't do the side-effect work itself — it only calls `setNotes` / `setSearchTermState`. The **effect** then *reacts*.
3. So the sequence is:

   > Screen calls `addNote` → `addNote` calls `setNotes` → React re-renders Provider → effect with `[notes]` runs → storage written → *all* consumers update.

Effects are **reactions**, not **commands**. They are the Provider's way of responding to its own state changes and lifecycle events.

### 7.6 Cleanup — the missing half of `useEffect`

Every effect can return a **cleanup function**. React runs it:

- Before the effect re-runs (to "undo" the previous run's side effect), and
- On unmount (when the component that declared it goes away).

Our three effects don't return cleanups — because nothing they do needs undoing (a one-shot fetch, a write, a log). But you must know the pattern for effects that open **subscriptions, timers, or event listeners**:

```tsx
useEffect(() => {
  const id = setInterval(() => { /* … */ }, 1000);
  return () => clearInterval(id);
}, []);
```

Without `clearInterval`, the timer keeps running forever, even after the Provider unmounts — a classic memory leak. For a note-taking app this might appear if you added, say, a live "sync with server every 30 s" effect — you'd need cleanup to cancel the interval on unmount. We don't need it here, but students must recognise it.

One more subtlety: if our Effect A (the storage fetch) is slow and the Provider *unmounts before the fetch resolves*, `setNotes` would be called on an unmounted component. Modern React no longer logs a warning for this, but the fix is a cancellation flag:

```tsx
useEffect(() => {
  let cancelled = false;
  (async () => {
    const raw = await AsyncStorage.getItem('@notes');
    if (!cancelled && raw) setNotes(JSON.parse(raw));
  })();
  return () => { cancelled = true; };
}, []);
```

We've kept the simpler version for clarity in the file, but this is the "correct in production" variant — worth showing as a refinement.

### 7.7 Predict-the-output exercise

Before moving on, try this. The file is as in Section 4.11. For each user action below, predict **which effects run** and **in what order**. Write your answer down, then check the spoiler.

**Scenario P1:** App cold-starts, `notes` array in storage is `[{ id:'a', createdAt:'…', title:'A', body:'…', category:'work' }]`.
(a) Immediately after mount, before the storage fetch resolves — which effects run?
(b) After the fetch resolves — which effects run, and in what order?

<details>
<summary>Answer P1</summary>

(a) **All three**, in declaration order: Effect A (fires the fetch), Effect B (writes `[]` to storage — the in-memory value hasn't updated yet), Effect C (logs the empty search term). Note Effect B does run even though `notes` will *soon* change — dependencies are evaluated on **this render**, not the pending one.

(b) Effect A's promise resolves → `setNotes([...])` → Provider re-renders → **Effect B** runs (its dependency `notes` changed) → writes `[restoredNotes]`. **Effect C** does **not** re-run (searchTerm unchanged). **Effect A** does **not** re-run (empty deps — already ran on mount).

</details>

**Scenario P2:** User types `"a"` into the search box on `SearchScreen`.
Which effects run?

<details>
<summary>Answer P2</summary>

**Only Effect C** (searchTerm changed → re-run). Effect A does not re-run. Effect B does not re-run (notes unchanged).

</details>

**Scenario P3:** User navigates back to Home, then taps Add and creates a new note.
Which effects run?

<details>
<summary>Answer P3</summary>

`addNote` → `setNotes(...)` → Provider re-renders → **Effect B** runs (notes changed) → persists. Effect C does not (searchTerm unchanged). Effect A does not (already ran on mount).

One subtle note: navigating back to Home does **not** unmount the `NotesProvider` — the navigator is *inside* the Provider, so the Provider stays mounted across screen pushes/pops. Effects are therefore not re-run merely by navigating. This is a key insight — navigation is not a mount/unmount event for the Provider.

</details>

**Scenario P4 (thinking):** Why would you *not* want Effect A to have `[notes]` as a dependency? What would go wrong?

<details>
<summary>Answer P4</summary>

Effect A loads notes *from* storage. If it re-ran whenever `notes` changed, every mutation would trigger another load from storage — which would then `setNotes` with the *old* persisted array, potentially overwriting or racing the in-memory updates. Worse: the `setNotes` inside would re-trigger the effect again → **infinite loop**. The intent of Effect A is "load once on startup"; expressing that intent correctly requires `[]`.

</details>

---

## Section 8 — Consuming from a Screen (`useContext` in practice)

### 8.1 The three-line import recipe

At the top of any screen:

```tsx
import { useNotes, Note } from '../context/NotesContext';
```

Inside the component:

```tsx
const { notes, addNote, deleteNote, filteredNotes, searchTerm, setSearchTerm } = useNotes();
```

That's the whole "connect to the store" step. No props, no wrapping, no `Consumer` component. The screen has the data and the functions.

### 8.2 `HomeScreen` — displaying and deleting

```tsx
// src/screens/HomeScreen.tsx
import React from 'react';
import {
  View, Text, FlatList, TouchableOpacity, StyleSheet, Button,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { useNotes, Note } from '../context/NotesContext';
import { RootStackParamList } from '../../App';

type HomeNav = NativeStackNavigationProp<RootStackParamList, 'Home'>;

export default function HomeScreen() {
  const { notes, deleteNote } = useNotes();
  const navigation = useNavigation<HomeNav>();

  const renderItem = ({ item }: { item: Note }) => (
    <View style={styles.card}>
      <Text style={styles.title}>{item.title}</Text>
      <Text style={styles.meta}>
        {item.category} • {new Date(item.createdAt).toLocaleString()}
      </Text>
      <Text numberOfLines={2} style={styles.body}>{item.body}</Text>
      <TouchableOpacity onPress={() => deleteNote(item.id)}>
        <Text style={styles.delete}>Delete</Text>
      </TouchableOpacity>
    </View>
  );

  return (
    <View style={styles.container}>
      <Button title="Add Note" onPress={() => navigation.navigate('AddNote')} />
      <Button title="Search" onPress={() => navigation.navigate('Search')} />
      <FlatList
        data={notes}
        keyExtractor={(item) => item.id}
        renderItem={renderItem}
        ListEmptyComponent={<Text>No notes yet.</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  card: { padding: 12, borderWidth: 1, borderRadius: 6, marginVertical: 6 },
  title: { fontWeight: 'bold', fontSize: 16 },
  meta: { color: '#666', fontSize: 12, marginVertical: 4 },
  body: { color: '#333' },
  delete: { color: 'red', marginTop: 8 },
});
```

What this screen demonstrates:

- **Reads** `notes` from context — the list.
- **Calls** `deleteNote(item.id)` — one line, and the whole app updates (Home re-renders with one fewer item; Search, if open, would too).
- **Navigates** via React Navigation — unrelated to context, but shows our stack wiring.

Notice what is *absent*: no props flow between Home and its siblings; Home has no `useState` for the notes array. All shared state lives in the Provider.

### 8.3 `AddNoteScreen` — calling a function to update shared state

```tsx
// src/screens/AddNoteScreen.tsx
import React, { useState } from 'react';
import {
  View, TextInput, Button, StyleSheet, Text,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { useNotes } from '../context/NotesContext';
import { RootStackParamList } from '../../App';

type AddNav = NativeStackNavigationProp<RootStackParamList, 'AddNote'>;

export default function AddNoteScreen() {
  const { addNote } = useNotes();
  const navigation = useNavigation<AddNav>();

  const [title, setTitle] = useState('');
  const [body, setBody] = useState('');
  const [category, setCategory] = useState('general');

  const onSave = () => {
    if (title.trim() === '') return;
    addNote(title, body, category);
    navigation.goBack();
  };

  return (
    <View style={styles.container}>
      <Text style={styles.label}>Title</Text>
      <TextInput
        style={styles.input}
        value={title}
        onChangeText={setTitle}
        placeholder="Note title"
      />

      <Text style={styles.label}>Body</Text>
      <TextInput
        style={[styles.input, styles.body]}
        value={body}
        onChangeText={setBody}
        multiline
        placeholder="Write something…"
      />

      <Text style={styles.label}>Category</Text>
      <TextInput
        style={styles.input}
        value={category}
        onChangeText={setCategory}
        placeholder="general / work / personal"
      />

      <Button title="Save Note" onPress={onSave} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  label: { fontWeight: 'bold', marginTop: 12 },
  input: { borderWidth: 1, borderColor: '#ccc', padding: 8, borderRadius: 4 },
  body: { height: 100, textAlignVertical: 'top' },
});
```

Key ideas:

- **Local state** is used only for the *form* — the draft values. They are not shared with anyone else, so they stay local.
- **`addNote(...)` is the only interaction with the shared model.** One call. Then `navigation.goBack()`.
- By the time Home re-renders (after the user lands back there), the new note is already in `notes` and appears at the top.

This is the central teaching point of the section: **local state for form inputs; context for shared data**. Don't put every piece of state in the context — only what must be shared.

### 8.4 `SearchScreen` — reading derived data + setting a context state

```tsx
// src/screens/SearchScreen.tsx
import React from 'react';
import {
  View, TextInput, FlatList, Text, StyleSheet,
} from 'react-native';
import { useNotes, Note } from '../context/NotesContext';

export default function SearchScreen() {
  const { searchTerm, setSearchTerm, filteredNotes } = useNotes();

  const renderItem = ({ item }: { item: Note }) => (
    <View style={styles.card}>
      <Text style={styles.title}>{item.title}</Text>
      <Text style={styles.meta}>
        {item.category} • {new Date(item.createdAt).toLocaleString()}
      </Text>
    </View>
  );

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        value={searchTerm}
        onChangeText={setSearchTerm}
        placeholder="Search notes…"
      />
      <FlatList
        data={filteredNotes}
        keyExtractor={(item) => item.id}
        renderItem={renderItem}
        ListEmptyComponent={<Text>No matches.</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  input: { borderWidth: 1, borderColor: '#ccc', padding: 8, borderRadius: 4, marginBottom: 12 },
  card: { padding: 12, borderWidth: 1, borderRadius: 6, marginVertical: 6 },
  title: { fontWeight: 'bold', fontSize: 16 },
  meta: { color: '#666', fontSize: 12, marginTop: 4 },
});
```

Note how the **search input is wired**:

```tsx
value={searchTerm}            // ← from context
onChangeText={setSearchTerm}  // ← from context
```

Every keystroke calls `setSearchTerm`, which updates **Provider state**. The Provider re-renders, recomputes `filteredNotes`, and broadcasts. This screen re-renders and displays the new `filteredNotes`. Effect C (from Section 7) fires too, logging the new term.

This is the neat thing about putting `searchTerm` in context: **the term is shared**. If a future feature lets Home show "recent searches" or pre-fill the search from Home, it works with no new plumbing.

### 8.5 When does a consumer re-render?

A component that calls `useNotes()` will re-render whenever the **`value` object reference** passed to `NotesContext.Provider` changes. In our design that's **every time the Provider re-renders**, which is:

- Whenever `notes` changes (add / delete / update / storage load).
- Whenever `searchTerm` changes.

So both Home and Search re-render on either kind of change — even if Home doesn't display the search term. For our small app, that's fine and even desirable (Search is usually the screen with the input; Home isn't mounted at the same time unless it's the previous screen in the stack — and React Navigation handles that by keeping it mounted, so it re-renders too).

The trade-off is what we mentioned in Section 6.5: a single context = simple, but every consumer re-renders on every change. Splitting the context (one for notes, one for search) or memoising `value` in a larger app is the mitigation. **Out of scope** for these notes — but students should know the concept exists.

---

## Section 9 — Data Flow Summary (End-to-End)

Let's pull the whole narrative together as a sequence of user actions, listing what happens *and in what order*.

### 9.1 Cold start of the app

```
App renders
  └── <NotesProvider>
        ├── state initialised: notes = [], searchTerm = ''
        ├── render → value = { notes: [], searchTerm: '', filteredNotes: [], fns… }
        ├── <NotesContext.Provider value={value}>
        │       └── <NavigationContainer>… screens mount…
        └── commit, then effects fire:
              Effect A: fetch AsyncStorage  (async)
              Effect B: persist []          (fires because notes changed from init)
              Effect C: log searchTerm ''

AsyncStorage resolves → setNotes([…restoredNotes])
  └── NotesProvider re-renders → value = { notes: [restored], … }
        └── consumers re-render (Home shows restored notes)
              Effect B re-runs (notes changed) → persist restored
              Effect A: no re-run ([])
              Effect C: no re-run ([searchTerm] unchanged)
```

### 9.2 Adding a note (from `AddNoteScreen`)

```
User taps "Save"
  └── AddNoteScreen: addNote(title, body, category)
        └── setNotes(prev => [newNote, ...prev])   ← functional updater
              └── NotesProvider re-renders
                    ├── value = { notes: [new, ...restored], filteredNotes: …, … }
                    ├── commit → Home/Search re-render (if mounted)
                    └── effects: B re-runs (persist), A/C unchanged

User navigates back to Home
  └── Home re-renders (or already did) → FlatList shows new note at top
```

### 9.3 Searching

```
User types in SearchScreen input
  └── onChangeText(term) → setSearchTerm(term)
        └── setSearchTermState(term)
              └── NotesProvider re-renders
                    ├── filteredNotes recomputed (useMemo deps [notes, searchTerm])
                    ├── value = { …, searchTerm: term, filteredNotes: […] }
                    └── commit → consumers re-render → SearchScreen shows filtered
              └── effects: C re-runs (log), A/B unchanged
```

### 9.4 Deleting a note

```
User taps "Delete" on HomeScreen
  └── deleteNote(id)
        └── setNotes(prev => prev.filter(n => n.id !== id))
              └── NotesProvider re-renders
                    └── commit → Home shows one fewer item
                          Search (if mounted) also re-filters
                          effects: B re-runs (persist)
```

### 9.5 The unifying picture

Everything in the app reduces to four relationships:

| Relationship | Example |
|---|---|
| **State → rendered UI** | `notes` / `filteredNotes` / `searchTerm` drive what's displayed |
| **Screen → context function** | `addNote`, `deleteNote`, `setSearchTerm`, `updateNote` |
| **Function → state setter** | `setNotes(...)`, `setSearchTermState(...)` |
| **State change → effect** | persistence (`[notes]`), logging (`[searchTerm]`), one-shot load (`[]`) |

Effects are the *side-effect layer*, functions are the *mutation layer*, state is the *truth layer*, and context is the *distribution layer*.

### 9.6 A visual flow (one diagram to rule them all)

```
             ┌────────────────────────────────────────────────┐
             │                  NotesProvider                 │
             │                                                │
             │  useState(notes) ──┐                           │
             │  useState(search) ─┤                           │
             │                    ▼                           │
             │              render: value = {…}               │
             │                    │                           │
             │  ┌─────────────────┴──────────────────┐        │
             │  │  useEffect []      : load once     │        │
             │  │  useEffect [notes] : persist on Δ  │        │
             │  │  useEffect [search]: react to Δ    │        │
             │  └─────────────────┬──────────────────┘        │
             │                    │                           │
             │   NotesContext.Provider value={value}          │
             │              {children}                        │
             └────────────────────┬───────────────────────────┘
                                  │
             ┌────────────────────┼────────────────────┐
             ▼                    ▼                    ▼
        HomeScreen            AddNoteScreen        SearchScreen
       (read: notes)         (call: addNote)    (read: filteredNotes,
      (call: deleteNote)                         call: setSearchTerm)
                                  │
                                  │ addNote() → setNotes(…)
                                  ▼
                       NotesProvider re-renders
                       every consumer re-renders
                       effects with changed deps re-run
```

### 9.7 What we've now proven

- A single Provider at the top of the app **distributes** shared state and functions.
- Screens consume only what they need via `useNotes()`, decoupled from each other.
- Any screen can **read**, **manipulate**, and **trigger** updates that every other screen sees.
- `useEffect` is not "called by a screen" — it is a **reaction** inside the Provider to its own state and lifecycle, governed by the dependency array.
- TypeScript's interface is the contract that keeps the Provider and every consumer honest.

---

## Section 10 — Pitfalls & Best Practices

This section collects the mistakes students most often make when first encountering Context. Each pitfall is paired with a *symptom* (what you'll observe) and a *fix* (what to do about it).

### 10.1 Using the context outside the Provider

**Symptom:** Runtime error `Error: useNotes must be used within a NotesProvider`.

**Cause:** A component calls `useNotes()` but is *not* a descendant of `NotesProvider`. Two common variants:

- The Provider was never placed in `App.tsx` at all.
- The Provider exists but the offending component is rendered **outside** `{children}` (Section 5.4).

**Fix:** Ensure `NotesProvider` wraps the whole navigator (Section 5.3). If you're intentionally rendering a component outside the Provider (e.g. a splash screen above it), use the raw context with a null-check — or simply don't call `useNotes()` there.

**Why our design catches this early:** Without the custom `useNotes()` hook, `useContext(NotesContext)` would return `undefined` and crash with a confusing *"Cannot read property 'addNote' of undefined"* much later. Our throwing hook moves the failure to the exact offending component, with a clear message. This is *the* reason to always wrap `useContext` in a custom hook.

### 10.2 Forgetting `{children}` or rendering it outside

**Symptom:** The app is blank, or `useNotes()` throws in every screen.

**Cause / Fix:** Covered in detail in Section 5.2. The correct shape is:

```tsx
return (
  <NotesContext.Provider value={value}>
    {children}
  </NotesContext.Provider>
);
```

If you're seeing a blank screen: check that `{children}` is present. If `useNotes()` throws everywhere: check that `{children}` is *inside* `.Provider`, not a sibling.

### 10.3 Mutating the array (or an object) in place

**Symptom:** The screen *sometimes* doesn't update after a context function runs. The new note appears only after a navigation or unrelated action.

**Cause:** Something like:

```tsx
// ❌ WRONG
const addNote = (…) => {
  notes.push(newNote);   // mutating the array
  setNotes(notes);       // same reference → React might skip the update
};

const updateNote = (id, updates) => {
  const n = notes.find(x => x.id === id);
  if (n) n.title = updates.title;   // mutating the object
  setNotes(notes);                  // same array reference
};
```

**Fix:** Always produce a new array and new object:

```tsx
// ✅ RIGHT
setNotes(prev => [newNote, ...prev]);
setNotes(prev => prev.map(n => n.id === id ? { ...n, ...updates } : n));
setNotes(prev => prev.filter(n => n.id !== id));
```

**Rule:** treat state as **read-only**. Never `push`, `splice`, `sort`, or assign into the existing array or its objects. React compares by reference — same reference, no re-render.

### 10.4 Stale state — reading `notes` instead of using `prev`

**Symptom:** Occasional wrong results when two functions run in the same tick (rapid taps, batched updates, asynchronous calls).

**Cause:**

```tsx
// ❌ risky
setNotes([newNote, ...notes]);
```

This reads `notes` from the *closure*. If two updates are batched, the second one operates on the pre-update `notes` and can overwrite the first.

**Fix:** always use the functional updater:

```tsx
// ✅ safe
setNotes(prev => [newNote, ...prev]);
```

This also matters inside effects that fire asynchronously (e.g. after `AsyncStorage.getItem` resolves) — the `notes` in scope is whatever it was at the time the effect was *created*, not when the promise resolves. The functional form sidesteps the issue entirely.

### 10.5 Putting too much into one context

**Symptom:** Every keystroke on the search screen causes every consumer to re-render — including screens that show nothing related to the search term.

**Cause:** We have a single `NotesProvider` broadcasting both `notes` and `searchTerm`. Every state change → new `value` object → every consumer re-renders.

**In this app**, that's acceptable — a handful of screens and small data. But in a production app with many consumers, you'd consider:

- **Splitting** into two contexts (`NotesContext` and `SearchContext`) so that a search-term change doesn't re-render consumers of `notes`.
- **Memoising** the context value (Section 10.6) so that `value` only changes when *its* content changes.
- **Splitting components** so fewer components consume the whole context.

We are *mentioning* these — not implementing them — because they obscure the core mechanics. But you should know the design trade-off exists.

### 10.6 Recreating `value` on every render (with a `useMemo` side-step)

Our code assigns a fresh object on every Provider render:

```tsx
const value: NotesContextValue = { notes, searchTerm, filteredNotes, …fns };
```

Every re-render of `NotesProvider` (including renders caused by unrelated state we might add later) gives consumers a *new* `value` reference, forcing them all to re-render.

**When this matters:** when the Provider holds state unrelated to what the consumers care about (e.g. a future `theme` flag, a `lastSyncTime` counter). Those changes would cause needless re-renders of every consumer.

**The pattern (introduced, not studied):**

```tsx
const value = useMemo<NotesContextValue>(
  () => ({ notes, searchTerm, filteredNotes, addNote, deleteNote, updateNote, setSearchTerm, getNoteById }),
  [notes, searchTerm, filteredNotes, /* function identities — see below */]
);
```

For a `useMemo` to actually help, the *functions* in the value need stable identities too — which means wrapping each in `useCallback`. That's a real-world pattern, but it's a topic for a later unit.

> **Side-step:** `useMemo` and `useCallback` are *optimisation tools*. They change *how often* things happen, not *what* happens. Until your app is large enough to notice re-render jank, prefer the simpler, more legible version. Learn the dependency-array concept now; learn `useMemo`/`useCallback` when you need to tune performance.

### 10.7 Overusing context

**Symptom:** Every piece of state — form drafts, temporary UI flags, navigation state — ends up in the context.

**Cause:** Confusing "state that must be *shared*" with "state that *exists*".

**Rule of thumb:**

- **Local state (`useState` inside a screen)** — data used only by that screen (form drafts, text-input values, toggle flags, animation state).
- **Context** — data that at least two screens need, or that must survive navigation, or that needs to be manipulated by one screen and displayed by another.

In our app, `searchTerm` is in context because (a) the Search screen sets it and (b) the `filteredNotes` derivation uses it. If Search were the only screen that cared about filtering, `searchTerm` could stay local — but we put it in context because we'd like to reuse the same term across screens (e.g. a "recent searches" feature on Home). That's a design *choice*, not a requirement.

The title/body/category **form inputs** on `AddNoteScreen` are a good example of state that should **stay local** — they are transient drafts. Only on "Save" do they cross into the shared model (via `addNote`). A common beginner mistake is to put the drafts into context too; don't.

### 10.8 Naming conventions

Follow these conventions consistently; they make the codebase *self-documenting*:

| Thing | Convention | Example |
|---|---|---|
| Context object | `XxxContext` | `NotesContext` |
| Provider component | `XxxProvider` | `NotesProvider` |
| Consumer hook | `useXxx` | `useNotes` |
| Value interface | `XxxContextValue` | `NotesContextValue` |
| Domain model | Singular noun | `Note` |

The hook name starts with `use` — this is not cosmetic. ESLint's `react-hooks/rules-of-hooks` rule relies on the `use*` prefix to know it's a hook and to enforce the Rules of Hooks (only call hooks from components or other hooks, not conditionally). Always name consumer hooks `useXxx`.

### 10.9 Where to place the Provider

- **Too low** (inside one screen) → other screens can't consume (Section 5.4).
- **Too high** (above things that don't need it, e.g. above error boundaries that should reset the context) → harder to reset state on logout, harder to test.
- **Just right** → wrap the navigator (or wrap the relevant navigation stack). This is the sweet spot for our app: it survives navigation between screens and is easily replaceable in tests.

### 10.10 Not resetting the context on "logout" / "clear data"

A consideration for real apps: if the user logs out, all notes in the context remain in memory unless you explicitly call `setNotes([])`. To support this, expose a `resetNotes()` function on the context — one line in the provider (`setNotes([])`), one line in the interface — and let the logout handler call it.

Our notes' context is not user-scoped, so we don't need to add it now — but *thinking* about this in the design phase is exactly the kind of consideration that separates a demo from a production design.

### 10.11 Summary of do's and don'ts

| ✅ Do | ❌ Don't |
|---|---|
| Define an interface for the context value | Use `any` or `unknown` in the context value |
| Use `createContext<T \| undefined>(undefined)` + a throwing `useXxx()` hook | Use a fake default object |
| Wrap the navigator with the Provider | Wrap one screen, or forget `{children}` |
| Use functional updaters (`prev => …`) | Read `notes` from closure inside setters |
| Produce new arrays/objects | Mutate `notes` or its objects |
| Keep transient form state **local** | Put every piece of state into the context |
| List every dependency in `useEffect` | Omit dependencies (and get stale values) |
| Cleanup timers / subscriptions in effects | Leak them on unmount |
| Use naming conventions (`NotesProvider`, `useNotes`) | Invent names ad hoc |

---

## Section 11 — Putting It Together (Full Worked Solution)

Below is the complete, runnable application. It targets the `expo-template-blank-typescript` template.

### 11.1 Project setup

```bash
npx create-expo-app NoteApp --template expo-template-blank-typescript
cd NoteApp

# Navigation
npx expo install @react-navigation/native @react-navigation/native-stack
npx expo install react-native-screens react-native-safe-area-context

# Persistence
npx expo install @react-native-async-storage/async-storage
```

### 11.2 Folder structure

```
NoteApp/
├── App.tsx
├── src/
│   ├── context/
│   │   └── NotesContext.tsx
│   └── screens/
│       ├── HomeScreen.tsx
│       ├── AddNoteScreen.tsx
│       └── SearchScreen.tsx
├── app.json
├── package.json
└── tsconfig.json
```

### 11.3 `src/context/NotesContext.tsx`

```tsx
import React, {
  createContext,
  useContext,
  useState,
  useEffect,
  useMemo,
  ReactNode,
} from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

export interface Note {
  id: string;
  createdAt: string;       // ISO 8601 timestamp
  title: string;
  body: string;
  category: string;
}

export interface NotesContextValue {
  notes: Note[];
  searchTerm: string;
  filteredNotes: Note[];

  addNote: (title: string, body: string, category: string) => void;
  deleteNote: (id: string) => void;
  updateNote: (
    id: string,
    updates: Partial<Omit<Note, 'id' | 'createdAt'>>
  ) => void;
  setSearchTerm: (term: string) => void;
  getNoteById: (id: string) => Note | undefined;
}

export const NotesContext = createContext<NotesContextValue | undefined>(undefined);

interface NotesProviderProps {
  children: ReactNode;
}

const STORAGE_KEY = '@notes';

export function NotesProvider({ children }: NotesProviderProps) {
  const [notes, setNotes] = useState<Note[]>([]);
  const [searchTerm, setSearchTermState] = useState<string>('');

  // Effect A — load once on mount
  useEffect(() => {
    (async () => {
      try {
        const raw = await AsyncStorage.getItem(STORAGE_KEY);
        if (raw) setNotes(JSON.parse(raw) as Note[]);
      } catch {
        // ignore read errors (e.g. corrupted JSON) — start with empty list
      }
    })();
  }, []);

  // Effect B — persist whenever notes change
  useEffect(() => {
    AsyncStorage.setItem(STORAGE_KEY, JSON.stringify(notes)).catch(() => {});
  }, [notes]);

  // Effect C — react to search term changes
  useEffect(() => {
    // placeholder for analytics / debounced search / etc.
    // console.log('Search term:', searchTerm);
  }, [searchTerm]);

  const addNote = (title: string, body: string, category: string): void => {
    const newNote: Note = {
      id: `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`,
      createdAt: new Date().toISOString(),
      title: title.trim(),
      body: body.trim(),
      category: category.trim() || 'general',
    };
    setNotes((prev) => [newNote, ...prev]);
  };

  const deleteNote = (id: string): void => {
    setNotes((prev) => prev.filter((n) => n.id !== id));
  };

  const updateNote = (
    id: string,
    updates: Partial<Omit<Note, 'id' | 'createdAt'>>
  ): void => {
    setNotes((prev) =>
      prev.map((n) => (n.id === id ? { ...n, ...updates } : n))
    );
  };

  const setSearchTerm = (term: string): void => {
    setSearchTermState(term);
  };

  const getNoteById = (id: string): Note | undefined => {
    return notes.find((n) => n.id === id);
  };

  const filteredNotes = useMemo(() => {
    const term = searchTerm.trim().toLowerCase();
    if (term === '') return notes;
    return notes.filter(
      (n) =>
        n.title.toLowerCase().includes(term) ||
        n.body.toLowerCase().includes(term) ||
        n.category.toLowerCase().includes(term)
    );
  }, [notes, searchTerm]);

  const value: NotesContextValue = {
    notes,
    searchTerm,
    filteredNotes,
    addNote,
    deleteNote,
    updateNote,
    setSearchTerm,
    getNoteById,
  };

  return (
    <NotesContext.Provider value={value}>
      {children}
    </NotesContext.Provider>
  );
}

export function useNotes(): NotesContextValue {
  const context = useContext(NotesContext);
  if (context === undefined) {
    throw new Error('useNotes must be used within a NotesProvider');
  }
  return context;
}
```

### 11.4 `App.tsx`

```tsx
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { NotesProvider } from './src/context/NotesContext';
import HomeScreen from './src/screens/HomeScreen';
import AddNoteScreen from './src/screens/AddNoteScreen';
import SearchScreen from './src/screens/SearchScreen';

export type RootStackParamList = {
  Home: undefined;
  AddNote: undefined;
  Search: undefined;
};

const Stack = createNativeStackNavigator<RootStackParamList>();

export default function App() {
  return (
    <NotesProvider>
      <NavigationContainer>
        <Stack.Navigator initialRouteName="Home">
          <Stack.Screen
            name="Home"
            component={HomeScreen}
            options={{ title: 'My Notes' }}
          />
          <Stack.Screen
            name="AddNote"
            component={AddNoteScreen}
            options={{ title: 'Add Note' }}
          />
          <Stack.Screen
            name="Search"
            component={SearchScreen}
            options={{ title: 'Search' }}
          />
        </Stack.Navigator>
      </NavigationContainer>
    </NotesProvider>
  );
}
```

### 11.5 `src/screens/HomeScreen.tsx`

```tsx
import React from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  StyleSheet,
  Button,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { useNotes, Note } from '../context/NotesContext';
import { RootStackParamList } from '../../App';

type HomeNav = NativeStackNavigationProp<RootStackParamList, 'Home'>;

export default function HomeScreen() {
  const { notes, deleteNote } = useNotes();
  const navigation = useNavigation<HomeNav>();

  const renderItem = ({ item }: { item: Note }) => (
    <View style={styles.card}>
      <Text style={styles.title}>{item.title}</Text>
      <Text style={styles.meta}>
        {item.category} • {new Date(item.createdAt).toLocaleString()}
      </Text>
      <Text numberOfLines={2} style={styles.body}>{item.body}</Text>
      <TouchableOpacity onPress={() => deleteNote(item.id)}>
        <Text style={styles.delete}>Delete</Text>
      </TouchableOpacity>
    </View>
  );

  return (
    <View style={styles.container}>
      <View style={styles.buttons}>
        <Button title="Add Note" onPress={() => navigation.navigate('AddNote')} />
      </View>
      <View style={styles.buttons}>
        <Button title="Search" onPress={() => navigation.navigate('Search')} />
      </View>
      <FlatList
        data={notes}
        keyExtractor={(item) => item.id}
        renderItem={renderItem}
        ListEmptyComponent={<Text style={styles.empty}>No notes yet.</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  buttons: { marginBottom: 8 },
  card: { padding: 12, borderWidth: 1, borderRadius: 6, marginVertical: 6 },
  title: { fontWeight: 'bold', fontSize: 16 },
  meta: { color: '#666', fontSize: 12, marginVertical: 4 },
  body: { color: '#333' },
  delete: { color: 'red', marginTop: 8 },
  empty: { textAlign: 'center', marginTop: 24, color: '#666' },
});
```

### 11.6 `src/screens/AddNoteScreen.tsx`

```tsx
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  Button,
  StyleSheet,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { useNotes } from '../context/NotesContext';
import { RootStackParamList } from '../../App';

type AddNav = NativeStackNavigationProp<RootStackParamList, 'AddNote'>;

export default function AddNoteScreen() {
  const { addNote } = useNotes();
  const navigation = useNavigation<AddNav>();

  const [title, setTitle] = useState('');
  const [body, setBody] = useState('');
  const [category, setCategory] = useState('general');

  const onSave = () => {
    if (title.trim() === '') return;
    addNote(title, body, category);
    navigation.goBack();
  };

  return (
    <View style={styles.container}>
      <Text style={styles.label}>Title</Text>
      <TextInput
        style={styles.input}
        value={title}
        onChangeText={setTitle}
        placeholder="Note title"
      />

      <Text style={styles.label}>Body</Text>
      <TextInput
        style={[styles.input, styles.body]}
        value={body}
        onChangeText={setBody}
        multiline
        placeholder="Write something…"
      />

      <Text style={styles.label}>Category</Text>
      <TextInput
        style={styles.input}
        value={category}
        onChangeText={setCategory}
        placeholder="general / work / personal"
      />

      <View style={styles.save}>
        <Button title="Save Note" onPress={onSave} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  label: { fontWeight: 'bold', marginTop: 12 },
  input: { borderWidth: 1, borderColor: '#ccc', padding: 8, borderRadius: 4 },
  body: { height: 100, textAlignVertical: 'top' },
  save: { marginTop: 16 },
});
```

### 11.7 `src/screens/SearchScreen.tsx`

```tsx
import React from 'react';
import {
  View,
  Text,
  TextInput,
  FlatList,
  StyleSheet,
} from 'react-native';
import { useNotes, Note } from '../context/NotesContext';

export default function SearchScreen() {
  const { searchTerm, setSearchTerm, filteredNotes } = useNotes();

  const renderItem = ({ item }: { item: Note }) => (
    <View style={styles.card}>
      <Text style={styles.title}>{item.title}</Text>
      <Text style={styles.meta}>
        {item.category} • {new Date(item.createdAt).toLocaleString()}
      </Text>
      <Text numberOfLines={2} style={styles.body}>{item.body}</Text>
    </View>
  );

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        value={searchTerm}
        onChangeText={setSearchTerm}
        placeholder="Search notes…"
      />
      <FlatList
        data={filteredNotes}
        keyExtractor={(item) => item.id}
        renderItem={renderItem}
        ListEmptyComponent={<Text style={styles.empty}>No matches.</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    padding: 8,
    borderRadius: 4,
    marginBottom: 12,
  },
  card: { padding: 12, borderWidth: 1, borderRadius: 6, marginVertical: 6 },
  title: { fontWeight: 'bold', fontSize: 16 },
  meta: { color: '#666', fontSize: 12, marginVertical: 4 },
  body: { color: '#333' },
  empty: { textAlign: 'center', marginTop: 24, color: '#666' },
});
```

### 11.8 What you should see when you run it

1. **Cold start** — Home screen with "No notes yet." The storage effect fires; if any previous session's notes exist, they appear.
2. **Add Note** — fill the form, tap Save. Navigates back to Home; the new note appears at the top with a timestamp.
3. **Delete** — tap Delete on a note; it disappears immediately. Restart the app — it stays gone (persistence works).
4. **Search** — type in the field. The list filters live as you type. Navigate Home and back — the term persists (it lives in the Provider, not the screen).
5. **Cross-screen reactivity** — with two notes, open Search, add a note via a hypothetical path (or just add, then navigate to Search): the filtered list reflects the change.

### 11.9 Recap checklist

Every screen and every line in this app exists for a reason. Use this checklist as a *summary* of what the whole document has taught:

- [x] **`Note`** — the domain model (id, createdAt, title, body, category).
- [x] **`NotesContextValue`** — the interface contract.
- [x] **`createContext<… | undefined>(undefined)`** — the channel.
- [x] **`NotesProvider`** — state + functions + effects + broadcast.
- [x] **Two `useState` calls** — one for `notes`, one for `searchTerm`.
- [x] **Three `useEffect` hooks** — load once (`[]`), persist on notes change (`[notes]`), react to search (`[searchTerm]`).
- [x] **Five functions** — `addNote`, `deleteNote`, `updateNote`, `setSearchTerm`, `getNoteById`.
- [x] **`filteredNotes`** via `useMemo` — derived data, briefly introduced.
- [x] **`value` typed as `NotesContextValue`** — the broadcast object.
- [x] **`{children}` inside `.Provider`** — the sub-tree gains access.
- [x] **`useNotes()`** — guarded consumer hook.
- [x] **`App.tsx`** — wraps navigator; every screen is a consumer.
- [x] **Three screens** — Home (read + call), Add (call), Search (read + set).

If you can explain each of these items out loud — *what it is, why it exists, what would break without it* — you have understood the Context API.

---

## Section 12 — Exercises and Discussion Questions

Each exercise lists a task and its solution. Attempt the task before revealing the answer.

---

### Exercise 1 — Add a `resetNotes` function

**Task:** Add a function to the interface and the Provider that clears all notes. Then call it from a new button on `HomeScreen`, guarded by a confirmation alert.

<details>
<summary>Solution</summary>

**In `NotesContextValue`:**

```tsx
resetNotes: () => void;
```

**In the Provider (anywhere before `value`):**

```tsx
const resetNotes = (): void => {
  setNotes([]);
};
```

**In `value`:** add `resetNotes` to the object literal.

**In `HomeScreen`:**

```tsx
import { Alert } from 'react-native';

// inside the component:
const { notes, deleteNote, resetNotes } = useNotes();

const confirmReset = () => {
  Alert.alert(
    'Delete all notes?',
    'This cannot be undone.',
    [
      { text: 'Cancel', style: 'cancel' },
      { text: 'Delete all', style: 'destructive', onPress: resetNotes },
    ]
  );
};

// add a button:
<Button title="Reset All" color="red" onPress={confirmReset} />
```

**Discussion:** Why does `setNotes([])` work? Because effects listen to `notes`, and clearing state re-renders the Provider → Effect B fires → empty array persisted. The chain of reasoning from Section 9 is exactly what makes this a five-minute change rather than a redesign.

</details>

---

### Exercise 2 — Change a dependency and predict the behaviour

**Task:** Suppose we changed Effect B from `[notes]` to `[]`. Describe what happens in each of these scenarios:

(a) Cold start with an existing notes array in storage.
(b) Adding a note.
(c) Deleting a note.

<details>
<summary>Solution</summary>

Effect B becomes:

```tsx
useEffect(() => {
  AsyncStorage.setItem(STORAGE_KEY, JSON.stringify(notes)).catch(() => {});
}, []);
```

(a) **Cold start:** Effect B runs **once on mount**, writing whatever `notes` is *at that moment* — the **initial empty array**. This overwrites the persisted notes with `[]` **before** Effect A's async load completes. The storage is wiped and the app has no notes. **Data loss on every launch.**

(b) **Adding a note:** State updates in memory (Home shows the new note), but Effect B **does not re-run** because its dependency array is empty. The new note is never persisted. If you kill the app and reopen, it's gone.

(c) **Deleting a note:** Same as (b) — the deletion is in memory only, never persisted. On restart, the note returns — as if the deletion never happened.

**Key lesson:** the dependency array is a **contract**. You promise to list every value the effect *reads*. `notes` is read — leaving it out is a lie, and React has no way to detect it. This is what "stale closures" mean in practice.

</details>

---

### Exercise 3 — Why didn't my screen update?

**Task:** A student writes `updateNote` like this:

```tsx
const updateNote = (id, updates) => {
  const found = notes.find(n => n.id === id);
  if (found) {
    Object.assign(found, updates);
  }
  setNotes(notes);
};
```

After calling `updateNote`, the change sometimes appears and sometimes doesn't. Explain *why*, and fix it.

<details>
<summary>Solution</summary>

The bug has *two* independent causes:

1. **`Object.assign(found, updates)` mutates the existing object.** The object `found` is the *same reference* as the one already in `notes`. So the "updated" note is *identical by reference* to its old self.
2. **`setNotes(notes)` passes the same array reference.** React uses `Object.is` to compare the new state with the old. Since `notes === notes`, React concludes "nothing changed" and **skips the re-render**.

Why "sometimes"? If a *different* state change (e.g. adding a note) happens right after, that re-render picks up the mutated object and shows it. Hence the intermittent appearance.

**Fix:**

```tsx
const updateNote = (id, updates) => {
  setNotes(prev => prev.map(n => n.id === id ? { ...n, ...updates } : n));
};
```

Three improvements in one line:

- `prev => …` — functional updater (no stale state).
- `.map(...)` — produces a *new array* (reference changes).
- `{ ...n, ...updates }` — produces a *new object* (reference changes).

**Key lesson:** immutable updates are not stylistics — they are the mechanism through which React *knows* to re-render.

</details>

---

### Exercise 4 — Explain the "invisible execution"

**Task:** In your own words (2–4 sentences), answer a student's question: *"I never call `AsyncStorage.setItem` from a screen, so how does the note get saved when I tap Save?"*

<details>
<summary>Solution</summary>

When I tap Save on `AddNoteScreen`, the screen calls `addNote(...)` — which is a function the Provider exposed via the context. That function calls `setNotes(...)`, changing the Provider's state. Because the Provider re-renders, React re-runs `useEffect` hooks whose dependencies changed — specifically Effect B, which depends on `[notes]`. So the save isn't initiated by a screen; it's a **reaction inside the Provider** to its own state change. The screen only triggers the state update; the effect handles the side effect.

</details>

---

### Exercise 5 — Should this state go into the context?

**Task:** For each item, decide whether it should live in the **context** (`NotesProvider`) or as **local state** in a screen, and justify your choice.

| # | State | Context or local? |
|---|---|---|
| 1 | The list of notes | ? |
| 2 | The search term | ? |
| 3 | The text typed in the "Title" field on AddNoteScreen | ? |
| 4 | A "dark mode" toggle that will affect all screens | ? |
| 5 | Which note is currently being edited (shared across two screens) | ? |
| 6 | The currently selected category on a future "CategoryPicker" UI inside AddNoteScreen | ? |
| 7 | A loading flag while notes are being fetched from storage | ? |

<details>
<summary>Solution</summary>

| # | Decision | Justification |
|---|---|---|
| 1 | **Context** | Shared across Home, Search, and any future screen — must survive navigation. |
| 2 | **Context** | Multiple screens read/derive from it (Search displays filtered notes; the term may be pre-filled from Home). |
| 3 | **Local** | Transient form draft — used only by AddNoteScreen. Crosses into the shared model only on Save. |
| 4 | **Context** (typically a *separate* ThemeContext) | Affects the whole app — but should be its own context so changing it doesn't re-render note consumers (Section 10.5). |
| 5 | **Context** | Two screens must agree on it. Could be a navigation param, but if both screens read/write it freely, context is simpler. |
| 6 | **Local** | A UI selection inside one screen, not needed elsewhere. (Unless there's a "recently used categories" feature — then reconsider.) |
| 7 | **Context** — *if consumers need to know* (e.g. show a spinner). Otherwise local. In our app, Effect A's fetch is so fast it doesn't need a visible flag; if the fetch became slow, a `loading` context flag would be appropriate. |

**Takeaway:** the question isn't "is it important?" but "does more than one screen need it, or must it survive navigation?".

</details>

---

### Exercise 6 — Add a "sort" option to the context

**Task:** Extend the context so that consumers can set a sort order — `'newest'`, `'oldest'`, `'title'` — and have `filteredNotes` (rename it `visibleNotes` or keep the name; your choice) reflect the sort. Show a picker in `HomeScreen`.

<details>
<summary>Solution sketch</summary>

**Interface additions:**

```tsx
sortOrder: 'newest' | 'oldest' | 'title';
setSortOrder: (order: 'newest' | 'oldest' | 'title') => void;
```

**Provider additions:**

```tsx
const [sortOrder, setSortOrder] = useState<'newest' | 'oldest' | 'title'>('newest');

const visibleNotes = useMemo(() => {
  const term = searchTerm.trim().toLowerCase();
  const filtered = term === ''
    ? notes
    : notes.filter(
        (n) =>
          n.title.toLowerCase().includes(term) ||
          n.body.toLowerCase().includes(term) ||
          n.category.toLowerCase().includes(term)
      );

  const sorted = [...filtered];   // new array — do not sort in place
  if (sortOrder === 'newest') {
    sorted.sort((a, b) => b.createdAt.localeCompare(a.createdAt));
  } else if (sortOrder === 'oldest') {
    sorted.sort((a, b) => a.createdAt.localeCompare(b.createdAt));
  } else {
    sorted.sort((a, b) => a.title.localeCompare(b.title));
  }
  return sorted;
}, [notes, searchTerm, sortOrder]);
```

Then `value = { notes, searchTerm, visibleNotes, sortOrder, setSortOrder, … }` and update consumers accordingly.

**HomeScreen addition:** three buttons (or a segmented control) to call `setSortOrder('newest' | 'oldest' | 'title')`. The list will re-render with the new order because `visibleNotes` is a dependency-driven derived value — the chain from Section 9 still holds.

**Important:** `[...filtered]` is required. `.sort()` **mutates** in place — sorting `filtered` directly would mutate a slice of state and break React's reference comparison (Section 10.3).

</details>

---

### Exercise 7 — Discussion questions

These have no single "right answer" — use them to spark in-class discussion.

1. **Why must `createContext` have a default value**, even if the Provider always supplies a real one? What design signal does `createContext<T | undefined>(undefined)` send to the codebase?

2. **Where should `NotesProvider` sit in `App.tsx`** — above or below `NavigationContainer`? What would change if we placed it *inside* one specific screen?

3. **If you added a `loading` state to the context**, what effect would set it to `false`? Which dependency array would that effect have? Why?

4. **Our `filteredNotes` is a derived value** — computed, not stored. What would go wrong if we stored `filteredNotes` in its *own* `useState` and updated it inside a `useEffect` that depended on `[notes, searchTerm]`?

   *Hint: there is a well-known anti-pattern here. Think about extra renders, and about the period between state updates.*

5. **Consider splitting `NotesContext` into two** — one for `notes` (data + manipulation), one for `searchTerm` (input). What would improve? What complexity would it add? When would the trade-off be worth it?

6. **The `id` uses `Date.now() + Math.random()`.** In a real-world app, would this be sufficient? What would you use instead, and why?

7. **What would happen if `useNotes()` were *not* guarded** — i.e. if it simply returned `useContext(NotesContext)` (typed as `NotesContextValue | undefined`) without throwing? How would students discover their Provider-placement mistake?

---

### Exercise 8 — Cross-check your understanding

Answer *without looking back*:

1. What are the four items in the mental model, and one sentence on each? *(createContext, Provider, useContext, children)*
2. What does the `| undefined` in `createContext<NotesContextValue | undefined>(undefined)` achieve?
3. List the three dependency-array forms and what each one means.
4. In what order do the three effects run on mount, and why?
5. If `addNote` is called, which effects re-run, and which do not?
6. Why do we use `setNotes(prev => …)` instead of `setNotes([...notes, newNote])`?
7. What is `{children}` doing, and what are the two ways of getting it wrong?
8. Why does the `useXxx` naming convention matter (technically, not stylistically)?

<details>
<summary>Answers</summary>

1. **`createContext`** — creates the typed channel / contract. **`Provider`** — holds state, functions, effects, and broadcasts them. **`useContext`** — subscribes a component to the current value. **`children`** — the sub-tree rendered inside the Provider; this is what makes descendants consumers.
2. It declares that the context's default (outside the Provider) is `undefined`, enabling the throwing `useNotes()` hook to catch misuse with a clear error.
3. **No array** — runs after every render. **`[]`** — runs once on mount (and cleanup on unmount). **`[deps]`** — runs on mount + whenever a dep changes (`Object.is`).
4. Declaration order: Effect A, then B, then C. On mount all three run because React treats initial mount as "dependencies are new".
5. **Effect B** re-runs (`notes` changed). **Effect A** does not (empty deps). **Effect C** does not (`searchTerm` unchanged).
6. Because `notes` is captured in the closure — using `prev` guarantees you operate on the *latest* state, not a potentially stale snapshot.
7. `{children}` renders the sub-tree inside `NotesContext.Provider`, placing screens *below* the provider in the tree. **Two mistakes:** omitting `{children}` (nothing renders), or rendering it outside `.Provider` (screens render but `useNotes()` throws).
8. ESLint's `react-hooks/rules-of-hooks` rule uses the `use*` prefix to identify hooks and enforce the Rules of Hooks. A hook not named `useXxx` would not be recognised, and the rule could not warn about misuse (conditional calls, loops, etc.).

</details>

---

### Closing remarks

Context is a simple idea — a channel, a supplier, a subscriber, and a wrapped sub-tree — wrapped in TypeScript types to enforce the contract. The subtlety lies entirely in **where state lives**, **how it flows**, and **when effects run in response to it**.

The Note-Taking App you've now studied demonstrates all of these in one small, coherent application:

- A single **interface** describes the entire API that consumers can use.
- A single **Provider** owns the state, hosts the effects, and exposes the functions.
- A single **throwing hook** gives every screen typed access — and protects against misuse.
- A **stack navigator** demonstrates that the Provider survives screen transitions (navigation is not unmount/mount for the Provider).
- A **dependency array** silently governs *when* each effect does its work — invisibly, precisely, and in declaration order.

The end-of-unit exercise is simple: **rebuild this app from an empty template without referring to the notes**. If you can, you've mastered the pattern.

---
