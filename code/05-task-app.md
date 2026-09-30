# 05 Task App

The project is a task list. The user types a task into a text input and presses a button to add it. Tasks appear in a list below. Later stages add the ability to delete a task, extract the per-task display into its own component, and finally split the component into a separate file. The arc of the project is not about hooks — there are only two `useState` calls in the entire app — but about **structure**. It is about how a list is rendered, how each item in a list is given a stable identity, how a component is given its own file, and how the boundary between parent and child is drawn.

---

## Stage 0 — App Shell

### What this stage introduces

This is the smallest possible screen. It renders a single empty `View` with a style that fills the screen and adds padding. Nothing is visible. The purpose of the stage is to establish the file's shape — imports at the top, a function component, a stylesheet, a default export — so that every later stage can be read as a modification of this skeleton.

### The code

```tsx
import { View, StyleSheet } from 'react-native';

function App() {
  return (
    <View style={styles.container}>

    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
});

export default App;
```

### Walkthrough

The file begins with an import. `View` and `StyleSheet` come from `react-native`.

`App` is a function component. It returns JSX: a `View` whose only content is a blank line. The `View` carries `style={styles.container}`, which is looked up from the `StyleSheet` created below. The blank line inside the `View` is not a stray newline — it is where the children of the `View` will go in later stages. The empty space is deliberate, and it makes the later diffs smaller.

`StyleSheet.create` is given a single object with one key, `container`. Two properties are set:

- `flex: 1` makes the `View` expand to fill its parent. Without it, the `View` would shrink to the size of its children — and since it has no children, it would have no size at all.
- `padding: 20` adds twenty units of space inside the `View`, on all four sides. When content is added in Stage 1, it will be inset from the screen edges by this amount.

The `export default App;` line makes the component the module's default export, which is what the React Native entry point expects.

### Revision notes and gotchas

- **`flex: 1` is doing more work than it appears to.** If it were removed, the `View` would have no height, and the padding would not be visible. Every later stage relies on this line, and it is the reason the screen fills the device rather than collapsing to nothing.
- **The empty line inside the `View` is intentional.** It is a placeholder for content that will be added at Stage 1. When reviewing a diff between Stage 0 and Stage 1, this is the line that will be replaced.

---

## Stage 1 — Placeholder Layout

### What this stage introduces

This stage gives the app a shape without giving it any behaviour. Two empty `View`s are placed inside the container, each preceded by a comment naming what will eventually go there: a heading, a task input, and a task list. Nothing renders yet, because the views are empty. The purpose of the stage is to commit to a layout — three regions, stacked vertically — before any logic is written.

### The code

```tsx
import { View, StyleSheet } from 'react-native';

function App() {
  return (
    <View style={styles.container}>
      {/* Heading */}
      <View>
        {/* Task Input */}
      </View>
      <View>
        {/* Task List Viewing */}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
});

export default App;
```

### Walkthrough

The container is unchanged from Stage 0. Inside it, three regions now exist:

- A comment `{/* Heading */}` marks where the heading will go. Comments in JSX are written inside braces with the block-comment syntax, because the braces switch back to JavaScript.
- An empty `View` holds a comment `{/* Task Input */}`. This view will eventually contain the `TextInput` and the add button.
- A second empty `View` holds a comment `{/* Task List Viewing */}`. This view will eventually contain the list of tasks.

None of the three regions has any content, and neither of the two inner `View`s has a style. They are placeholders. Nothing is displayed when the app runs except a blank white screen with the padding from the container.

The two inner `View`s are worth noticing, even though they do nothing. They establish that the input and the list are *separate* regions of the layout. When styles are added in Stage 2, the input view will become a horizontal row (input on the left, button on the right), and the list view will get a top margin to separate it from the input. If the two regions were not separated at this stage, the styles would have to be attached to the parent, and the layout would be harder to adjust.

### Revision notes and gotchas

- **JSX comments are not HTML comments.** `<!-- ... -->` will not compile inside JSX. The form is `{/* ... */}`. This is a common stumbling block when first writing JSX, and it is worth remembering.
- **The empty `View`s render nothing, but they are not free.** Each `View` is a real native view. Two empty views cost essentially nothing, but the habit of adding placeholder views that later never get filled is a small source of layout bugs. At this stage they are justified because they mark the regions that Stage 2 will fill.
- **No styles have been added to the inner views yet.** They will appear at Stage 2, when there is content whose layout matters.

### Key concepts

**Layout as a commitment.** Writing the region structure before the content forces a decision about how the screen is divided. Here the decision is: heading on top, input row below it, list below that. Making that decision before writing the logic means the logic can be written to fit the layout, rather than the layout being adjusted around the logic. It is a small example of a general habit — sketch the shape, then fill it in.

---

## Stage 2 — State, Input, and `.map()`

### What this stage introduces

This is the first stage with behaviour. Three ideas arrive together.

The first is **state for a list**. Two `useState` calls are added: one holds the current text of the input field, the other holds the array of tasks. This is the first time the project has state, and it is the first time a piece of state holds a collection rather than a single value.

The second is the **controlled input**. `TextInput` is given a `value` prop and an `onChangeText` prop. The value it displays is whatever the state variable holds, and typing fires the callback that updates the state. The input does not keep its own internal value; the state is the single source of truth.

The third is **`.map()` as a rendering loop**. To display the tasks, the array is mapped to an array of `Text` elements. This is the standard way to render a list in React before `FlatList` is introduced, and it is worth understanding on its own terms before it is replaced.

The stage also introduces the add handler, and with it the first opportunity to get a guard condition. 

### The code

```tsx
import { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
} from 'react-native';

function App() {
  const [task, setTask] = useState<string>(''); // current input
  const [tasks, setTasks] = useState<string[]>([]); // list of tasks

  const handleAddTask = (): void => {
    // Guard: do nothing when the input is empty.
    // Return early so the code below only runs for a real task.
    if (task === '') {
      return;
    }

    setTasks([...tasks, task]); // add new task to array
    setTask(''); // clear input
  };

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Tasks App</Text>

      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Enter a task"
          value={task}
          onChangeText={(text: string) => setTask(text)}
        />
        <TouchableOpacity style={styles.addButton} onPress={handleAddTask}>
          <Text style={styles.addButtonText}>+</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.tasksContainer}>
        {tasks.map((task: string, index: number) => (
          <Text key={index} style={styles.task}>
            {index} - {task}
          </Text>
        ))}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { 
    flex: 1, 
    padding: 20 
  },
  heading: { 
    fontSize: 24, 
    marginBottom: 10 
  },
  inputContainer: { 
    flexDirection: 'row', 
    alignItems: 'center' 
  },
  input: { 
    flex: 1, 
    height: 40, 
    borderColor: 'gray', 
    borderWidth: 1, 
    paddingHorizontal: 10 
  },
  addButton: { 
    width: 40, 
    height: 40, 
    backgroundColor: '#333', 
    justifyContent: 'center', 
    alignItems: 'center' 
  },
  addButtonText: { 
    fontSize: 24, 
    color: '#fff' 
  },
  tasksContainer: { 
    marginTop: 20 
  },
  task: { 
    fontSize: 18, 
    marginBottom: 10 
  },
});

export default App;
```

### Walkthrough

Two pieces of state are declared. `task` holds the current text in the input field, initialised to the empty string. `tasks` holds the array of tasks, initialised to an empty array. The type annotations — `<string>` and `<string[]>` — tell TypeScript what each state variable holds. `setTask` and `setTasks` are the setters returned alongside them.

`handleAddTask` is the handler for the add button. Reading it in order:

- The guard `if (task === '') { return; }` checks whether the input is empty. If it is, the function returns immediately and nothing is added. This is a **guard clause**: it handles the invalid case first, at the top of the function, so that the rest of the function can assume the input is valid. The alternative — wrapping the whole body in `if (task !== '') { ... }` — works too, but it adds a level of nesting for the entire body. The guard clause keeps the main body at one level.
- `setTasks([...tasks, task])` builds a new array consisting of every existing task followed by the new one, and passes it to the setter. This is the **immutable append**. The existing `tasks` array is not modified; a new array is produced. React compares the new state reference to the old one to decide whether to re-render, so the array must be replaced rather than mutated.
- `setTask('')` clears the input by setting the state back to the empty string. Because the input's `value` prop is bound to `task`, this is what visually clears the field.

The JSX has grown to three sections. The heading is a plain `Text` with a style. The input section is a `View` with `flexDirection: 'row'`, containing a `TextInput` and a `TouchableOpacity`. The list section is a `View` whose children are produced by `.map()`.

The `TextInput` has four props worth naming:

- `style={styles.input}` gives it a border, padding, and a height.
- `placeholder="Enter a task"` shows grey text when the field is empty.
- `value={task}` binds the displayed value to the `task` state variable. This is what makes the input **controlled**.
- `onChangeText={(text: string) => setTask(text)}` fires every time the user types. The callback receives the new text and calls `setTask` with it. Because `setTask` already has the right signature — it takes a string — the arrow function is redundant, and `onChangeText={setTask}` would do the same thing. Both forms are correct; the explicit arrow is retained here because it names the parameter and makes the flow visible.

The `.map()` call is the heart of this stage. It is worth reading carefully because the syntax is dense:

```tsx
{tasks.map((task: string, index: number) => (
  <Text key={index} style={styles.task}>
    {index} - {task}
  </Text>
))}
```

Reading it piece by piece:

- The outer braces `{ ... }` switch from JSX into JavaScript. Everything inside is an expression.
- `tasks.map(...)` calls the `map` method on the `tasks` array. `map` produces a new array by applying a function to each element.
- The function being applied is `(task: string, index: number) => ( ... )`. It takes two parameters — the element itself, named `task`, and its position, named `index` — and returns a `Text` element.
- The `Text` element has a `key` prop set to `index`, and a `style` prop. Its children are the text content `{index} - {task}`, which renders as something like `0 - Buy milk`.
- The outer `( ... )` around the `Text` element, immediately after the arrow, is an implicit return. The arrow body is a single expression wrapped in parentheses; `map` returns that expression's value for each element. If the body were written with braces `{ ... }`, it would need an explicit `return`. Both forms are valid; the parenthesised form is the one used here.

The result is an array of `Text` elements, one per task, and JSX renders that array as a list.

The `key` prop is required whenever an array of elements is rendered, because React uses it to identify which element in the new array corresponds to which element in the previous one. Here the key is the index. This works while the list is append-only — items are only ever added to the end — because the index of each item never changes. Once items can be removed, the index of the items after the removed one shifts, and the keys no longer correspond to the same items. This is the fragility that Stage 3's `FlatList` and Stage 5's deletion will expose. It is noted here so that when it matters, it is recognisable.

The `.map()` callback is written on multiple lines, with the opening `(` on the same line as `=>` and the closing `))}` on its own line. This is a formatting convention that makes the callback easier to read when the JSX inside it is more than a single line. It is worth getting used to, because nested arrows are a common source of bracket confusion.

The stylesheet has grown considerably. Each new style is named after the element it applies to, and the names line up with the JSX:

- `heading` sets the title's size and bottom margin.
- `inputContainer` uses `flexDirection: 'row'` and `alignItems: 'center'` to lay the input and the add button side by side, vertically centred.
- `input` gives the text field `flex: 1` (so it takes all available width next to the button), a fixed height, a grey border, and horizontal padding.
- `addButton` gives the button a fixed square size and a dark background.
- `addButtonText` styles the `+` inside the button.
- `tasksContainer` adds a top margin, separating the list from the input row.
- `task` sets the size and bottom margin of each task row.

### The `.map()` callback, in detail

The line `tasks.map((task: string, index: number) => ( ... ))` contains three nested pairs of brackets, and it is worth slowing down to see what each pair is doing.

```
tasks.map( (task, index) => ( <Text>...</Text> ) )
│       │   │              │ │                │ │
│       │   │              │ │                │ └── close of the map call
│       │   │              │ └────────────────── close of the arrow's body
│       │   └─────────────────────────────────── open of the arrow's body
│       └─────────────────────────────────────── open of the arguments to map
└─────────────────────────────────────────────── the array being iterated
```

The `map` method takes one argument: a function. The function is written in the arrow form. The arrow's body is a single expression — a `Text` element — wrapped in parentheses so it can span multiple lines. The braces `{ ... }` around the whole thing are the JSX expression container, which is how you tell JSX "evaluate this JavaScript and render the result".

When React renders this, it walks the `tasks` array, calls the function once per element, collects the results into a new array, and renders that array in place of the `{ ... }` expression. If `tasks` is empty, the result is an empty array, and React renders nothing.

It is worth noting what is *not* happening. The `map` call is not a loop in the imperative sense — it does not run "for each task, do something". It produces a new array, and JSX renders that array. The distinction matters when the callback becomes more complex, because anything that needs to happen "as a side effect of rendering" cannot be placed in a `map` callback without making the render impure.

### Revision notes and gotchas

- **The `key={index}` is correct for now and will not stay correct.** Because the list is append-only, indices never shift, and the index is a valid key. As soon as deletion is introduced, the indices of the items after a deleted item shift, and the keys no longer correspond to the same items. React will reuse the wrong DOM (or, in React Native, the wrong native view) for the wrong item, producing visible glitches. Stage 3 introduces `FlatList` with its `keyExtractor`, and Stage 5 introduces deletion — the moment the fragility becomes real.

- **`onChangeText={setTask}` would work here.** The explicit arrow `(text: string) => setTask(text)` is not necessary, because `setTask` already accepts a string and returns nothing. The explicit form is retained because it names the parameter, which makes the data flow visible at the point of reading. Both forms are acceptable; the explicit one is a legitimate choice when teaching, and the shorter one is a legitimate choice when writing.

- **The input does not clear on submission by pressing the keyboard's return key.** It clears only when the add button is pressed. If you want the return key to submit, you would add `onSubmitEditing={handleAddTask}` and `returnKeyType="done"` to the `TextInput`. This is not part of the in-class code and is mentioned only so that the omission is visible.

- **The `+` button has no pressed-state feedback.** `TouchableOpacity` fades when pressed by default, but the visual change is subtle on a dark background. This is a design choice, not a bug, and it is not corrected here.

### Key concepts

**State for a list.** `useState<string[]>([])` declares a state variable whose value is an array. Every update must produce a *new* array, not modify the existing one, because React compares the state reference to decide whether to re-render. `[...tasks, task]` is the standard way to produce a new array with one element appended. `tasks.push(task)` would modify the array in place, leave the state reference unchanged, and React would not re-render. The distinction between replacing and mutating is the single most important habit to develop when working with list state.

**The controlled input.** A controlled `TextInput` has its `value` prop bound to a state variable and its `onChangeText` prop bound to the setter for that variable. The input does not keep its own internal value; every keystroke updates the state, and every render passes the current state back to the input. This makes the state the single source of truth: if you want to clear the input, you set the state to `''`; if you want to read what the user has typed, you read the state. The alternative — an uncontrolled input that keeps its own value — is possible but requires a ref to read the value, and it does not support the pattern of clearing the input from outside, which this app uses after every add.

**`.map()` as a rendering loop.** Given an array, `map` produces a new array by applying a function to each element. In React, this is how a list of elements is produced from a list of data. The function passed to `map` is called once per element and must return something renderable — typically a JSX element. The result is an array of elements, which JSX renders in order. `.map()` is not a loop in the imperative sense; it is a transformation, and the rendering is a consequence of the array being placed in JSX.

**Guard clauses.** A guard clause is an `if` at the top of a function that handles an invalid or exceptional case by returning early. The rest of the function can then assume the case has been handled and the input is valid. The pattern reduces nesting compared to wrapping the whole body in an `if`, and it makes the "what the function does in the normal case" more visible. In `handleAddTask`, the guard handles the empty-input case, and the two lines after it handle the real case.

**Keys for list elements.** Whenever an array of elements is rendered, React needs a `key` on each element to identify which element in the new array corresponds to which element in the previous one. The key must be unique among siblings and stable across renders. The array index satisfies both conditions for an append-only list, but not once items are removed. The correct key is a stable identifier that travels with the item — an id — and Stage 5 discusses this at the point where it matters.

---

## Stage 3 — Rendering the List with FlatList

### What this stage introduces

Stage 2 rendered the tasks with `.map()`. That works, and for a small list it works well — but it has a limitation that does not appear until the list grows: every task is rendered, always, whether or not it is visible on screen. There is no scrolling machinery, no recycling, no laziness. `.map()` produces an array of `Text` elements, and React renders all of them into the `tasksContainer` `View`. Once the array is longer than the screen, the extra items are laid out off-screen and never shown; the container does not scroll.

This stage replaces `.map()` with `FlatList`. `FlatList` is the standard React Native component for rendering lists. It scrolls, it renders only the items currently visible plus a small buffer, and it recycles the native views of items that have scrolled off-screen. For a list of any size, it is the correct tool.

Three ideas arrive with it. The first is the **`data` prop**, which is the array to render. The second is **`renderItem`**, a function that produces a single row from a single item. The third is **`keyExtractor`**, a function that returns a stable string identifier for each item so that `FlatList` can track rows across renders.

This stage also introduces a subtlety about how `renderItem`'s function is shaped: it is not called with the item directly, but with an object that has an `item` property. That shape is named and typed in Stage 4; here it is written inline.

### The code

```tsx
import { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  StyleSheet,
} from 'react-native';

function App() {
  const [task, setTask] = useState<string>('');
  const [tasks, setTasks] = useState<string[]>([]);

  const handleAddTask = (): void => {
    // Guard: do nothing when the input is empty.
    // Return early so the code below only runs for a real task.
    if (task === '') {
      return;
    }

    setTasks([...tasks, task]); // add new task to array
    setTask(''); // clear input
  };

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Tasks App</Text>
      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Enter a task"
          value={task}
          onChangeText={(text: string) => setTask(text)}
        />
        <TouchableOpacity style={styles.addButton} onPress={handleAddTask}>
          <Text style={styles.addButtonText}>+</Text>
        </TouchableOpacity>
      </View>
      <View style={styles.tasksContainer}>
        <FlatList
          data={tasks}
          renderItem={({ item, index }) => (
            <Text style={styles.task}>
              {index} - {item}
            </Text>
          )}
          keyExtractor={(_item, index) => index.toString()}
        />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { 
    flex: 1, 
    padding: 20 
  },
  heading: { 
    fontSize: 24, 
    marginBottom: 10 
  },
  inputContainer: { 
    flexDirection: 'row', 
    alignItems: 'center' 
  },
  input: { 
    flex: 1, 
    height: 40, 
    borderColor: 'gray', 
    borderWidth: 1, 
    paddingHorizontal: 10 
  },
  addButton: { 
    width: 40, 
    height: 40, 
    backgroundColor: '#333', 
    justifyContent: 'center', 
    alignItems: 'center' 
  },
  addButtonText: { 
    fontSize: 24, 
    color: '#fff' 
  },
  tasksContainer: { 
    marginTop: 20 
  },
  task: { 
    fontSize: 18, 
    marginBottom: 10 
  },
});

export default App;
```

### Walkthrough

The imports gain `FlatList` from `react-native`. The state declarations, `handleAddTask`, the heading, and the input row are unchanged from Stage 2. The change is confined to the `tasksContainer`:

- The `tasks.map(...)` expression has been removed.
- A `<FlatList>` element has been placed inside the container.
- Three props are passed to it: `data`, `renderItem`, and `keyExtractor`.

Reading the `FlatList` in order:

**`data={tasks}`** is the array to render. `FlatList` reads this array, determines how many rows to display, and calls `renderItem` once per visible row (plus a small buffer). When `tasks` changes — because a new task was added — `FlatList` re-renders and includes the new row.

**`renderItem={({ item, index }) => ( ... )}`** is the function that produces a single row. Its shape is the important part. `FlatList` does not call `renderItem` with the item itself; it calls it with an **object** that has several properties, of which `item` and `index` are the two used here. The parameter is written `({ item, index })` — a destructuring pattern that pulls `item` and `index` out of the object the function receives. If it were written `(item, index) => ...`, then `item` would be the whole object, and `index` would be the second argument to the function (which `FlatList` does not pass). The destructured form is the correct one, and it is a common source of confusion precisely because the same words appear in the parameter list of both forms.

Inside the function, a single `Text` element is returned, with `style={styles.task}` and content `{index} - {item}`. The `index` and `item` are the ones destructured from the parameter. This is the same content that `.map()` produced at Stage 2, with the same index-and-text format.

**`keyExtractor={(_item, index) => index.toString()}`** is the function that returns a string identifier for each item. `FlatList` calls it once per item to obtain a key, which is used the same way the `key` prop was used in the `.map()` version: to identify which row corresponds to which item across renders. The signature is `(item, index) => string`. The item is named `_item` here, with a leading underscore, which is a convention for "this parameter is intentionally unused". The index is used, converted to a string with `.toString()`, and returned.

The name of the parameter is worth noting: it is `_item`, not `item`, and the underscore communicates that the function does not read the item's value. It only needs the index to produce a key. This is a small, deliberate signal.

The styles are unchanged from Stage 2. The `task` style still applies to each `Text` inside the `FlatList`.

### `.map()` and `FlatList` — the difference that matters

The two approaches produce the same visible result for a small list, and it is worth being clear about why `FlatList` is preferred anyway.

`.map()` produces a plain array of elements. React renders that array into the container. Every element is created and mounted, regardless of whether it fits on screen. If the array has a thousand items, a thousand `Text` elements are mounted. The container is not a scroller by default, so off-screen items are laid out beyond the visible area and are never reachable. To scroll, the container would need to be a `ScrollView`, and a `ScrollView` also mounts everything — it just allows the user to scroll to reach them.

`FlatList` is a virtualised list. It knows the total number of items, but it only mounts the ones currently visible on screen plus a small buffer above and below. As the user scrolls, items that leave the buffer are unmounted, and items entering the buffer are mounted. The native views are recycled. For a short list the difference is invisible; for a long list it is the difference between an app that responds and an app that freezes.

There is a second difference worth noting. `FlatList` is a scroller by default — it fills its container and scrolls its content if the content is taller. `.map()` inside a plain `View` does not scroll; to make it scroll, the `View` would have to be replaced by a `ScrollView`, and everything inside would still be mounted. `FlatList` provides both the scrolling and the virtualisation in one component.

The cost of using `FlatList` is that it needs to know more about the data: it needs `renderItem` and `keyExtractor`, and it needs them to behave in specific ways. `.map()` only needs the array and a function. That extra structure is what enables the virtualisation, and it is a price worth paying for any list that could grow.

### Revision notes and gotchas

- **The `renderItem` parameter is destructured from an object, not passed as the item.** `({ item, index }) => ...` and `(item, index) => ...` look similar but mean different things. The first pulls `item` and `index` out of the object that `FlatList` passes. The second would treat the first argument — the whole object — as `item`, and the second argument — which `FlatList` does not pass — as `index`. When a `FlatList` renders nothing useful, or shows `[object Object]`, this is usually the cause. Stage 4 names this shape with a type, which makes it harder to get wrong.

- **`keyExtractor` uses the index, exactly as the `key` prop did at Stage 2.** `keyExtractor={(_item, index) => index.toString()}` returns `"0"`, `"1"`, `"2"`, and so on. This is valid for an append-only list, for the same reason it was valid at Stage 2. It stops being valid the moment items can be removed from the middle of the list, because the indices shift. Stage 5 introduces deletion and addresses this.

- **The item parameter is named `_item`, not `item`.** The leading underscore is a convention that says "this parameter is intentionally unused". `keyExtractor` does not read the item; it only needs the index. Naming it `_item` signals that clearly. Some linting configurations enforce the underscore for unused parameters; others do not. Either way, using it is a small but useful signal to the reader.

- **`keyExtractor` is optional, but its default is usually not what you want.** If it is omitted, `FlatList` falls back to using the item's `key` property if the item is an object with one, and otherwise uses the index. Relying on the default is possible for very simple cases, but writing it explicitly is clearer and makes the choice of key visible. The in-class code writes it explicitly, which is the better habit.

- **`FlatList` must not be placed inside a `ScrollView` with the same orientation.** `FlatList` is itself a scroller; nesting it inside another scroller of the same orientation produces a warning and breaks the virtualisation. It can be placed inside a non-scrolling `View` — which is what this code does, inside `tasksContainer` — without issue. This is worth knowing because a reader who is tempted to wrap the whole screen in a `ScrollView` will encounter the warning.

- **The `task` style's `marginBottom: 10` still applies to each row.** `FlatList` does not add spacing between rows on its own; the spacing comes from the row's own style. This is the same as it was for the `.map()` version, and the visual result is unchanged.

### Key concepts

**`FlatList` and virtualisation.** `FlatList` is a virtualised list: it mounts only the rows currently visible plus a small buffer, and recycles the native views as the user scrolls. This is what allows it to handle long lists without degrading. The virtualisation is what `FlatList` provides that `.map()` cannot; `.map()` renders everything, and the container does not scroll.

**`renderItem` receives an object, not the item.** The function passed to `renderItem` is called with an object whose `item` and `index` properties are the current element and its position. The destructured parameter `({ item, index })` is how those properties are accessed. This shape is dictated by `FlatList`, and it is one of the most common points of confusion when first using the component.

**`keyExtractor` and stable identity.** `FlatList` needs a string key for each item so that it can identify which row corresponds to which item across renders. The key must be unique among the current items and stable — the same item must produce the same key across renders, even if its position changes. An index satisfies uniqueness but not stability once items can be removed. An id assigned when the item is created satisfies both. The choice of key is not cosmetic: it determines whether `FlatList` reuses the correct row when the data changes.

**The cost of structure.** `.map()` requires only the array and a function; `FlatList` requires `data`, `renderItem`, and `keyExtractor`, and each must be shaped the way `FlatList` expects. That extra structure is the price of virtualisation and scrolling. It is worth paying for any list that could grow, and it is the pattern you will see in production code.

---

## Stage 4 — Typed Render-Item Parameters

### What this stage introduces

Stage 3 wrote the `renderItem` function inline, with the destructured parameter `({ item, index })`. That works, and TypeScript infers the types from `FlatList`'s own type definitions. But the inference is not visible in the code, and a reader has to know what `FlatList` passes in order to know what `item` and `index` are.

This stage makes the shape explicit. A small type is declared that names the properties `renderItem` receives, and the parameter in the `renderItem` call is annotated with it. The result is that `item` is declared to be a `string` and `index` to be a `number`, and the compiler will complain if anything inconsistent is done with them.

There is a second, less obvious benefit. The type is declared once, at the top of the file, and can be used in more than one place if the same shape is needed again. This stage uses it once; Stage 5, when `TaskItem` is introduced, will use a related shape for the child component's props, and the naming convention established here will carry over.

### The code

```tsx
import { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  StyleSheet,
} from 'react-native';

// FlatList - Props specifically for 'renderItem' parameters
type FlatListProps = {
  item: string;
  index: number;
};

function App() {
  const [task, setTask] = useState<string>('');
  const [tasks, setTasks] = useState<string[]>([]);

  const handleAddTask = (): void => {
    // Guard: do nothing when the input is empty.
    // Return early so the code below only runs for a real task.
    if (task === '') {
      return;
    }

    setTasks([...tasks, task]); // add new task to array
    setTask(''); // clear input
  };

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Tasks App</Text>
      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Enter a task"
          value={task}
          onChangeText={(text: string) => setTask(text)}
        />
        <TouchableOpacity style={styles.addButton} onPress={handleAddTask}>
          <Text style={styles.addButtonText}>+</Text>
        </TouchableOpacity>
      </View>
      <View style={styles.tasksContainer}>
        <FlatList
          data={tasks}
          renderItem={({ item, index }: FlatListProps) => (
            <Text style={styles.task}>{item}</Text>
          )}
          keyExtractor={(_item, index) => index.toString()}
        />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { 
    flex: 1, 
    padding: 20 
  },
  heading: { 
    fontSize: 24, 
    marginBottom: 10 
  },
  inputContainer: { 
    flexDirection: 'row', 
    alignItems: 'center' 
  },
  input: { 
    flex: 1, 
    height: 40, 
    borderColor: 'gray', 
    borderWidth: 1, 
    paddingHorizontal: 10 
  },
  addButton: { 
    width: 40, 
    height: 40, 
    backgroundColor: '#333', 
    justifyContent: 'center', 
    alignItems: 'center' 
  },
  addButtonText: { 
    fontSize: 24, 
    color: '#fff' 
  },
  tasksContainer: { 
    marginTop: 20 
  },
  task: { 
    fontSize: 18, 
    marginBottom: 10 
  },
});

export default App;
```

### Walkthrough

A `type` declaration is added near the top of the file, above the component:

```tsx
type FlatListProps = {
  item: string;
  index: number;
};
```

This declares a type named `FlatListProps` with two properties: `item`, a string, and `index`, a number. It describes the shape of the object that `FlatList` passes to `renderItem`. The comment above it — "FlatList - Props specifically for 'renderItem' parameters" — names the purpose so that a reader does not have to guess.

The `renderItem` prop changes from:

```tsx
renderItem={({ item, index }) => (
  <Text style={styles.task}>
    {index} - {item}
  </Text>
)}
```

to:

```tsx
renderItem={({ item, index }: FlatListProps) => (
  <Text style={styles.task}>{item}</Text>
)}
```

Two things have changed, and only one of them was the point of the stage. The intended change is the type annotation: `({ item, index }: FlatListProps)` tells TypeScript that the object being destructured has the shape declared above. The `item` and `index` variables are now known to be a `string` and a `number`, and any use of them that violates those types will be a compile error.

The second change is that the display no longer includes the index. In Stage 3, the rendered content was `{index} - {item}`; in Stage 4, it is only `{item}`. The in-class code made this simplification at the same time as the type annotation, and it is worth noting explicitly rather than letting it pass. The reason is not technical — the index is still available in the type, and could still be rendered — but presentational. Once the type is declared, the reader can see that `index` is available without it needing to be on screen, and the display can be simplified. Whether to keep the index visible is a design choice; the code here removes it, and the choice is worth noticing because it explains why the rendered text differs from Stage 3 even though the data has not changed.

The `keyExtractor` is unchanged. It still returns the index as a string, still names the item parameter `_item`, and still works for the append-only list this stage has.

The styles are unchanged.

### What the type annotation buys

Without the annotation, TypeScript infers the types of `item` and `index` from `FlatList`'s own type definitions. The inference is usually correct, and the code compiles. So what is the benefit of writing the type out?

There are three reasons, and they are worth stating plainly because the benefit is not always obvious from a single example.

**The types are visible.** A reader who does not know what `FlatList` passes to `renderItem` can see, from the declaration, that the parameter has two properties — `item` and `index` — and what their types are. The declaration is a small piece of documentation that survives even if the code is copied into a context where the inference would be lost, such as a `.js` file or a project without `FlatList`'s type definitions available.

**The type can be reused.** `FlatListProps` is a named type. If another component in the same file needs the same shape — for example, if a helper function took `(props: FlatListProps)` — the type is already declared and can be referenced. In Stage 5, when `TaskItem` is introduced, a similar pattern is used for the child component's props, with a name in the same style.

**Errors are caught at the declaration.** If the code is later changed so that `item` is used as a number, or `index` as a string, the compiler will flag the mismatch at the point of use. The inference catches this too, but the explicit annotation makes the shape part of the code that a reader can point to, rather than something inferred in the background.

The cost is one small declaration and one annotation. The benefit is clarity, reusability, and a documented contract.

### The name collision with `FlatListProps`

React Native exports a type called `FlatListProps`. It is the type of the props that the `FlatList` component accepts — `data`, `renderItem`, `keyExtractor`, and many others. Declaring a local type with the same name does not break anything, because the local declaration shadows the imported one within this file. But it is a collision that will confuse anyone who has seen the real `FlatListProps`, because the two types are not related.

There are two ways to handle this. The first is to rename the local type to something that does not collide — for example, `RenderItemParams` or `TaskRenderItem`. The second is to leave it as it is and accept that the name is local to this file. The in-class code uses `FlatListProps`, and it is preserved here to match. The naming is noted as a small trap: if you ever import `FlatListProps` from `react-native` into the same file, the two will clash, and the one you are referring to will depend on the order of declarations.

The rename is worth considering, and the convention in most codebases is to avoid shadowing exported names. But this is a stylistic choice, not a correctness issue, and the in-class version is preserved.

### Revision notes and gotchas

- **The type is a description of the parameter, not of the component's props.** The name `FlatListProps` suggests the props of `FlatList`, and that is what React Native's own `FlatListProps` refers to. Here, the local `FlatListProps` describes the single argument that `renderItem` receives — which is a small object with `item` and `index`. The two are different things, and the shared name is a coincidence that will confuse readers. Renaming the local type to `RenderItemParams` or `TaskRenderItem` would remove the ambiguity; it is a stylistic choice and is not applied here.

- **The display dropped the index.** Stage 3 rendered `{index} - {item}`; Stage 4 renders `{item}`. This is not a consequence of the type annotation — the `index` is still available and still typed as a number. It is a presentational choice made in the same edit. It is worth noticing, because a reader comparing the two stages will otherwise assume that typing the parameter somehow made the index unavailable, which it did not.

- **The annotation is on the destructured parameter, not on the whole object.** Writing `({ item, index }: FlatListProps)` annotates the destructured parameter as a whole, which is what TypeScript requires. Writing `({ item, index }: { item: string; index: number })` would inline the same type without the name. Writing `({ item: string, index: number })` — which looks similar — is a syntax error, because the colons in a destructuring pattern are for renaming, not for typing. This last form is a common mistake for readers who have seen type annotations next to variable names and assume the same applies inside a destructuring pattern.

- **The type annotation does not change what `FlatList` passes.** `FlatList` passes whatever it passes, regardless of how the parameter is annotated. The annotation tells TypeScript what to expect, and the compiler checks that the code is consistent with that expectation. If the annotation were wrong — if `item` were declared as a number, say — the compiler would not complain at the `FlatList` call, because `FlatList`'s own type definitions are what determine the actual argument. The annotation would instead cause errors inside the function body wherever `item` was used. This is a subtlety of TypeScript annotations on callbacks: they are a promise about the argument, and the compiler checks the body against the promise, not the caller against the promise.

- **`FlatList` infers the item type from `data` when the annotation is absent.** If `data={tasks}` and `tasks` is `string[]`, `FlatList` infers that `item` is a `string`. This is why the unannotated version of Stage 3 compiles. The annotation in Stage 4 makes the inference explicit, but does not change the inferred type.

### Key concepts

**Type declarations as documentation.** A `type` declaration names a shape and gives it a reusable name. It documents, in the code, what a value is expected to look like. Even where TypeScript would infer the same thing, the declaration makes the shape visible to a reader and available for reuse elsewhere in the file.

**Type annotations on destructured parameters.** When a function parameter is destructured — `({ item, index })` — the annotation goes after the closing brace: `({ item, index }: FlatListProps)`. The annotation describes the whole object, not the individual properties. Annotating the individual properties inside the braces is not valid syntax, because the colon inside a destructuring pattern is reserved for renaming.

**The distinction between a type and a value.** `type FlatListProps = { ... }` declares a type, not a value. It exists only at compile time and has no runtime presence. It cannot be imported as a value, and it cannot be logged. Its only role is to be referenced by other type annotations, and to be checked by the compiler.

**Named types and reuse.** A named type can be referenced wherever its shape is needed. This is a small example of a general principle: shapes that appear in more than one place should have a name, so that a change to the shape is a change in one place. This stage uses the type once, but the pattern is the one that will be extended in Stage 5 when the `TaskItem` component's props are declared.

---

## Stage 5 — Extracting TaskItem, and Deleting a Task

### What this stage introduces

Up to now, the `renderItem` function inside `FlatList` has returned a `Text` element directly. That works while the row is a single piece of text, but this stage adds a delete button to each row, and the row becomes a small layout: the task text on the left, the delete button on the right. Adding that layout inline in `renderItem` would work, but it would make the `FlatList` prop harder to read, and it would hide the row's structure inside the parent component.

So this stage does two things at once. First, it introduces **deletion**: each row gains an `X` button that removes the task from the list. Second, it **extracts the row into its own component**, `TaskItem`, which lives in the same file at this stage and is passed the data it needs through props. The extraction is not cosmetic. It draws a boundary between "the list" and "one row of the list", and it introduces the pattern of passing a function from parent to child so that the child can trigger something in the parent.

The stage is presented in two parts. Part A presents the version as taught, which uses **index-based deletion** — the delete handler receives the index of the task to remove, and filters the array by index. This works, and it is the natural first version. Part B presents an alternative that uses **id-based deletion** — each task is given a unique id when it is created, and the delete handler receives that id. The two versions look similar, but they behave differently once the list is no longer append-only, and Part B is the version that survives that change. The comparison is the substance of the stage.

### Why the extraction matters

Before the code, it is worth being clear about what the extraction buys, because at this stage the row is only two elements and the extraction could be skipped.

The `renderItem` prop of a `FlatList` is a function that produces a row. If the row is one line, writing the function inline is fine. Once the row becomes a layout — a `View` containing a `Text` and a `TouchableOpacity` — the function grows, and the `FlatList` prop becomes harder to read at a glance. The `FlatList`'s job is to render a list; the row's job is to display one item. Mixing them means the reader of `FlatList` has to read the row's layout to understand the list, and the reader of the row has to know it is a row of a `FlatList`.

Extracting the row into `TaskItem` separates the two. `App` describes the list; `TaskItem` describes a row. The `FlatList`'s `renderItem` becomes one line — "produce a `TaskItem` with these props" — and the row's layout lives in its own function. When the row later needs to change (a checkbox, a due date, a swipe gesture), the change is confined to `TaskItem`, and `App` does not need to be touched.

The extraction also introduces a **callback prop**: `TaskItem` is given `onDelete`, a function that it calls when the delete button is pressed. `TaskItem` does not know what `onDelete` does — it does not have access to the `tasks` state, and it should not. It knows only that it has a button, and that pressing the button should call a function it was given. This is the contract between parent and child: the parent owns the data and the logic, and the child is given the ability to signal. The contract is worth naming, because it is the pattern that every child component in a React application follows.

---

## Stage 5, Part A — Index-Based Deletion (as Taught)

### What this part presents

This is the version taught in class. The row is extracted into `TaskItem`, and the delete handler receives the index of the task to remove. `handleDeleteTask` filters the array by index, producing a new array that omits the entry at that position. The `keyExtractor` continues to use the index as the key, as it did at Stage 3 and Stage 4.

The version works. It is a reasonable first implementation, and for a list that is only ever appended to, it is correct. The fragility is not in the code as it stands; it is in what happens when the list stops being append-only. Part B discusses that.

### The code

```tsx
import { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  StyleSheet,
} from 'react-native';

// FlatList - Props specifically for 'renderItem' parameters
type FlatListProps = {
  item: string;
  index: number;
};

// --- user defined component --- 
type TaskItemProps = {
  item: string;
  index: number;
  onDelete: (index: number) => void;
};

const TaskItem = ({ item, index, onDelete }: TaskItemProps) => {
  return (
    <View style={styles.taskItem}>
      <Text style={styles.taskText}>{index} - {item}</Text>
      <TouchableOpacity onPress={() => onDelete(index)} style={styles.deleteButton}>
        <Text style={styles.deleteButtonText}>X</Text>
      </TouchableOpacity>
    </View>
  );
};
// ------------------------------

function App() {
  const [task, setTask] = useState<string>('');
  const [tasks, setTasks] = useState<string[]>([]);

  const handleAddTask = (): void => {
    // Guard: do nothing when the input is empty.
    // Return early so the code below only runs for a real task.
    if (task === '') {
      return;
    }

    setTasks([...tasks, task]); // add new task to array
    setTask(''); // clear input
  };

  const handleDeleteTask = (index: number): void => {
    const updatedTasks = tasks.filter((_, i) => i !== index);
    setTasks(updatedTasks);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Tasks App</Text>
      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Enter a task"
          value={task}
          onChangeText={(text: string) => setTask(text)}
        />
        <TouchableOpacity style={styles.addButton} onPress={handleAddTask}>
          <Text style={styles.addButtonText}>+</Text>
        </TouchableOpacity>
      </View>
      <View style={styles.tasksContainer}>
        <FlatList
          data={tasks}
          renderItem={({ item, index }: FlatListProps) => (
            <TaskItem item={item} index={index} onDelete={handleDeleteTask} />
          )}
          keyExtractor={(_item, index) => index.toString()}
        />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  heading: {
    fontSize: 24,
    marginBottom: 10,
  },
  inputContainer: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  input: {
    flex: 1,
    height: 40,
    borderColor: 'gray',
    borderWidth: 1,
    paddingHorizontal: 10,
  },
  addButton: {
    width: 40,
    height: 40,
    backgroundColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
  },
  addButtonText: {
    fontSize: 24,
    color: '#fff',
  },
  tasksContainer: {
    marginTop: 20,
  },
  taskItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 10,
    paddingVertical: 6,
    paddingHorizontal: 10,
    backgroundColor: '#f2f2f2',
    borderRadius: 4,
  },
  taskText: {
    fontSize: 18,
  },
  deleteButton: {
    backgroundColor: '#e53935',
    paddingHorizontal: 10,
    paddingVertical: 4,
    borderRadius: 4,
  },
  deleteButtonText: {
    color: '#fff',
    fontWeight: 'bold',
  },
});

export default App;
```

### Walkthrough

A second type is declared above the component:

```tsx
type TaskItemProps = {
  item: string;
  index: number;
  onDelete: (index: number) => void;
};
```

This describes the props that `TaskItem` accepts. Three props: `item`, a string; `index`, a number; and `onDelete`, a function that takes a number and returns nothing. The `void` return type on `onDelete` is a way of saying "the caller should not expect a value back". The function is called for its effect, not for its result.

`TaskItem` is a function component declared above `App`:

```tsx
const TaskItem = ({ item, index, onDelete }: TaskItemProps) => {
  return (
    <View style={styles.taskItem}>
      <Text style={styles.taskText}>{index} - {item}</Text>
      <TouchableOpacity onPress={() => onDelete(index)} style={styles.deleteButton}>
        <Text style={styles.deleteButtonText}>X</Text>
      </TouchableOpacity>
    </View>
  );
};
```

Reading it in order:

- The parameter is destructured, pulling `item`, `index`, and `onDelete` out of the props object. The type annotation `: TaskItemProps` declares the shape of the props.
- The return value is a `View` with the `taskItem` style: a horizontal row with space between its children and its content vertically centred.
- Inside, a `Text` shows `{index} - {item}`, the same format used at Stage 3 and Stage 4.
- Beside it, a `TouchableOpacity` wraps a `Text` reading `X`. Its `onPress` prop is `() => onDelete(index)` — an arrow function that calls `onDelete` with `index`. The arrow is necessary here: `onPress={onDelete(index)}` would call `onDelete` immediately during rendering and pass its return value (`undefined`) as the handler. The arrow defers the call until the button is pressed.

The `onPress` line is worth dwelling on. `onDelete` is a prop, not a local function. The component does not know what `onDelete` does; it only knows the shape — a function that takes a number. When the button is pressed, the arrow function runs, calls `onDelete` with `index`, and `onDelete` does whatever it does. In this case, it is `handleDeleteTask` from `App`, but `TaskItem` has no way to know that. This is the callback prop pattern: the parent passes a function, the child calls it, and the child does not need to know what happens next.

Inside `App`, a new handler is declared:

```tsx
const handleDeleteTask = (index: number): void => {
  const updatedTasks = tasks.filter((_, i) => i !== index);
  setTasks(updatedTasks);
};
```

This is the deletion logic. `tasks.filter((_, i) => i !== index)` produces a new array containing every task whose index is not equal to `index`. The first parameter of the filter callback is the item itself, named `_` — an underscore, meaning "unused". The second is the index, named `i`. The callback returns `true` for items to keep and `false` for items to discard. Here, it returns `true` for every item except the one at position `index`. The result is a new array with the task at that position omitted. The array is then passed to `setTasks`, which replaces the state and triggers a re-render.

The filter callback is worth slowing down to read. `tasks.filter((_, i) => i !== index)` contains two arrows? No — one arrow, with two parameters. The `_` and the `i` are the two parameters of the same callback. The `_` is the task's text; the `i` is its position. The `_` is named with an underscore because the callback does not need the text; it only needs the position. The `i !== index` comparison is what determines whether the item is kept. This is a compact way of expressing "remove the item at position `index`", and it takes a moment to read because of the two parameters and the underscore convention.

The `renderItem` prop of the `FlatList` changes to:

```tsx
renderItem={({ item, index }: FlatListProps) => (
  <TaskItem item={item} index={index} onDelete={handleDeleteTask} />
)}
```

Instead of returning a `Text`, the function returns a `TaskItem`. Three props are passed: `item` and `index`, which come from the destructured parameter, and `onDelete`, which is `handleDeleteTask`. The `TaskItem` receives these and renders itself. The `FlatList` sees only that `renderItem` returns an element; it does not need to know what that element is, or what it contains.

The `keyExtractor` is unchanged from Stage 3 and Stage 4: `keyExtractor={(_item, index) => index.toString()}`. This is the point at which the choice of key becomes significant. Once items can be removed, the index of every item after the removed one shifts, and the keys no longer correspond to the same items. This is discussed in the Revision notes.

The stylesheet gains three new entries: `taskItem`, `taskText`, and `deleteButton` with its `deleteButtonText`. The `task` style from earlier stages has been removed, since it applied to the `Text` that is no longer rendered directly by `App`. The `taskItem` style arranges the row horizontally with space between the text and the button. `taskText` sets the text size. `deleteButton` gives the `X` its red background and padding. `deleteButtonText` styles the `X` itself in white bold.

### What works and what does not

The index-based version works for the list as it is. A task is added at the end, deleted by its current position, and the remaining tasks keep their relative order. For an append-only list where deletion happens one at a time, the index is a valid identifier for the delete operation.

Two things make it fragile, and both are worth naming.

The first is that the index is a *position*, not an *identity*. If two deletes are queued in the same render — for example, if the user taps two `X` buttons before React has processed the first update — both handlers run against the same `tasks` array, and the second delete acts on a stale index. In practice, React batches updates and the user is unlikely to tap two buttons within a single frame, so this is rare in a simple app. But it is not impossible, and the failure mode is subtle: the wrong task is deleted.

The second is the `keyExtractor`. Because the key is the index, when a task in the middle is removed, the indices of the tasks after it shift. React sees that the key `"2"` is still present (it was the third task, and now it is the second), and it reuses the row that was previously at key `"2"`. But the content at key `"2"` has changed — the task that was at index `3` is now at index `2`, and the task that was at index `2` has been removed. React does not know this; it assumes the row at key `"2"` is the same row as before. The result can be visible: text that flickers, rows that appear to keep stale content for a frame, or, with certain component structures, the wrong row's state persisting into the wrong task.

Neither of these failures is guaranteed. For a small list with single-tap deletes, the app will appear to work. The fragility is in what is not tested, not in what is.

### Revision notes and gotchas

- **`onPress={() => onDelete(index)}` is not the same as `onPress={onDelete(index)}`.** The first passes a function that calls `onDelete` when the button is pressed. The second calls `onDelete` immediately during rendering and passes its return value — `undefined` — as the handler, which then does nothing. This is the single most common mistake when passing a callback with an argument, and it applies to every button in the app.

- **`tasks.filter((_, i) => i !== index)` uses `_` for the unused item.** The underscore is a convention for "this parameter is intentionally unused". The callback needs only the index, not the item, to decide whether to keep it. Naming the first parameter `_` signals that clearly. Some linters accept `_` as a marker for unused parameters; others require a different convention.

- **`filter` returns a new array; it does not modify the original.** `tasks.filter(...)` produces a new array with the selected elements. `tasks` is unchanged. The new array is passed to `setTasks`, which replaces the state. This is the same immutable-update pattern used by `[...tasks, task]` in `handleAddTask`. Both produce a new array; both leave the old one untouched.

- **The key is the index, and the index is not stable once deletion is possible.** This is the fragility that Part B addresses. The `keyExtractor` returning `index.toString()` was correct for the append-only list of Stages 2 through 4. It is not correct for a list where items can be removed from the middle. The failure is not always visible, and it is often dismissed as "a rendering glitch" when it does appear. The right fix is a stable id per item, which Part B introduces.

- **The delete handler receives an index, which is the position at the time the row was rendered.** If the tasks array changes between the row being rendered and the button being pressed — for example, because another task was added or removed in the meantime — the index may no longer point to the intended task. In a single-user, single-screen app with one interaction at a time, this is unlikely. In an app with concurrent updates, it is a real bug. The id-based version avoids it because the id travels with the task and does not depend on position.

- **`TaskItem` does not know about the tasks array.** It is given an `item`, an `index`, and a function. It does not have access to `tasks`, `setTasks`, or `handleDeleteTask`. It only knows that it has a button, and that pressing the button should call the function it was given. This is the contract, and it is what makes the component reusable: it could be placed in a different list with a different delete handler, and it would work, as long as the function it is given has the right shape.

### Key concepts

**Component extraction.** A piece of JSX that has its own structure and its own concerns can be moved into a function component of its own. The new component receives its data through props and returns the JSX it used to inline. Extraction is not required by React; it is a decision about where to draw boundaries. The boundaries chosen here are "the list" (in `App`) and "one row of the list" (in `TaskItem`). The boundary makes each side easier to read and easier to change.

**The callback prop pattern.** A parent component owns state and logic. A child component is given a function through a prop, and calls that function when something happens. The child does not know what the function does; it knows only its shape. This is how `TaskItem` is given `onDelete`: it calls `onDelete(index)` when its button is pressed, and `App` decides what happens. The pattern is the basis of every parent-child interaction in React, and it is what allows a child component to be reused in different contexts.

**Passing a function versus calling it.** `onDelete={handleDeleteTask}` passes the function. `onDelete={handleDeleteTask(index)}` would call it. Inside `TaskItem`, `onPress={() => onDelete(index)}` passes an arrow that calls `onDelete` when the button is pressed; `onPress={onDelete(index)}` would call it during rendering. Both patterns — passing the function directly when no argument is needed, and wrapping it in an arrow when an argument is — are used throughout the code, and the difference is significant.

**Filter as a way of removing an element.** `filter` produces a new array containing only the elements for which the callback returns `true`. To remove one element at a known position, the callback returns `true` for every position except the one to remove. The result is a new array with that element omitted. This is the standard way to remove an element from an array without mutating it, and it pairs naturally with the immutable-update pattern of state setters.

**Index versus identity.** An index is a position; an identity is something that travels with the item. For an append-only list, position and identity happen to coincide, and an index can be used as an identifier. For a list where items can be added anywhere, removed, or reordered, position and identity diverge, and only an identity — an id — is stable. The `keyExtractor` needs an identity; the delete handler, in a list that can be modified concurrently, also benefits from one. Part B introduces ids for this reason.

---

## Stage 5, Part B — Id-Based Deletion (the Revision)

### What this part changes

Part A works, but it is fragile for the reasons given above. The fix is to give each task a **stable id** — a string that is unique for the task and never changes — and to use that id in place of the index wherever the task is identified. Two things change as a result:

1. The data structure for a task changes from `string` to an object with an `id` and a `text`. This is a bigger change than it first appears, because it touches `tasks`, `handleAddTask`, `handleDeleteTask`, `TaskItem`, and the `renderItem` in `FlatList`.
2. The `keyExtractor` uses `item.id` instead of `index`. The delete handler receives an `id` and filters by `id` rather than by index.

The change is presented here as an alternative to Part A, not as a replacement. Part A's version is what was taught in class, and it is a reasonable first version. Part B is what the same code looks like once the fragility is addressed. The comparison is the point.

### The code

```tsx
import { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  StyleSheet,
} from 'react-native';

// A task now has an id (stable) and a text (mutable content).
type Task = {
  id: string;
  text: string;
};

// FlatList - Props specifically for 'renderItem' parameters
type FlatListProps = {
  item: Task;
  index: number;
};

// --- user defined component ---
type TaskItemProps = {
  item: Task;
  index: number;
  onDelete: (id: string) => void;
};

const TaskItem = ({ item, index, onDelete }: TaskItemProps) => {
  return (
    <View style={styles.taskItem}>
      <Text style={styles.taskText}>{index} - {item.text}</Text>
      <TouchableOpacity onPress={() => onDelete(item.id)} style={styles.deleteButton}>
        <Text style={styles.deleteButtonText}>X</Text>
      </TouchableOpacity>
    </View>
  );
};
// ------------------------------

function App() {
  const [task, setTask] = useState<string>('');
  const [tasks, setTasks] = useState<Task[]>([]);

  const handleAddTask = (): void => {
    // Guard: do nothing when the input is empty.
    // Return early so the code below only runs for a real task.
    if (task === '') {
      return;
    }

    // Build a task with a unique id. Date.now() is unique enough for
    // one user tapping one button at a time; it is not a general-purpose
    // id generator, and a counter or uuid would be used in production.
    const newTask: Task = {
      id: Date.now().toString(),
      text: task,
    };

    setTasks([...tasks, newTask]); // add new task to array
    setTask(''); // clear input
  };

  const handleDeleteTask = (id: string): void => {
    const updatedTasks = tasks.filter(t => t.id !== id);
    setTasks(updatedTasks);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Tasks App</Text>
      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Enter a task"
          value={task}
          onChangeText={(text: string) => setTask(text)}
        />
        <TouchableOpacity style={styles.addButton} onPress={handleAddTask}>
          <Text style={styles.addButtonText}>+</Text>
        </TouchableOpacity>
      </View>
      <View style={styles.tasksContainer}>
        <FlatList
          data={tasks}
          renderItem={({ item, index }: FlatListProps) => (
            <TaskItem item={item} index={index} onDelete={handleDeleteTask} />
          )}
          keyExtractor={item => item.id}
        />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  heading: {
    fontSize: 24,
    marginBottom: 10,
  },
  inputContainer: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  input: {
    flex: 1,
    height: 40,
    borderColor: 'gray',
    borderWidth: 1,
    paddingHorizontal: 10,
  },
  addButton: {
    width: 40,
    height: 40,
    backgroundColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
  },
  addButtonText: {
    fontSize: 24,
    color: '#fff',
  },
  tasksContainer: {
    marginTop: 20,
  },
  taskItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 10,
    paddingVertical: 6,
    paddingHorizontal: 10,
    backgroundColor: '#f2f2f2',
    borderRadius: 4,
  },
  taskText: {
    fontSize: 18,
  },
  deleteButton: {
    backgroundColor: '#e53935',
    paddingHorizontal: 10,
    paddingVertical: 4,
    borderRadius: 4,
  },
  deleteButtonText: {
    color: '#fff',
    fontWeight: 'bold',
  },
});

export default App;
```

### Walkthrough

A new type is declared at the top:

```tsx
type Task = {
  id: string;
  text: string;
};
```

A task is now an object with two properties. `id` is a string and is intended to be unique for each task and stable across renders. `text` is the task's content. This replaces the bare `string` that was used before, and it is the reason the rest of the code changes.

`FlatListProps` is updated so that `item` is a `Task` instead of a `string`:

```tsx
type FlatListProps = {
  item: Task;
  index: number;
};
```

`TaskItemProps` is updated so that `item` is a `Task` and `onDelete` takes a string id:

```tsx
type TaskItemProps = {
  item: Task;
  index: number;
  onDelete: (id: string) => void;
};
```

The `onDelete` signature changes from `(index: number) => void` to `(id: string) => void`. This is the contract change: the child no longer reports a position; it reports an identity.

Inside `TaskItem`, the `Text` now reads `{item.text}` instead of `{item}`, because `item` is an object and the text is one of its properties. The `onPress` handler calls `onDelete(item.id)` instead of `onDelete(index)`:

```tsx
<TouchableOpacity onPress={() => onDelete(item.id)} style={styles.deleteButton}>
```

The `index` prop is still passed to `TaskItem`, and the `Text` still shows it. The index is used for display only; it is no longer used as an identifier. This is the key conceptual shift of the stage: the index is a *presentation detail*, not an *identity*.

Inside `App`, the `tasks` state is now `Task[]`:

```tsx
const [tasks, setTasks] = useState<Task[]>([]);
```

`handleAddTask` builds a `Task` object instead of pushing a raw string:

```tsx
const newTask: Task = {
  id: Date.now().toString(),
  text: task,
};

setTasks([...tasks, newTask]);
```

The id is `Date.now().toString()` — the current time in milliseconds as a string. This is the same technique used in an earlier project. It is unique enough for one user tapping one button at a time, because two tasks cannot be added within the same millisecond by a human. It is not a general-purpose id generator. A production app might use a counter, a `uuid`, or an id assigned by a server. `Date.now()` is the simplest thing that works for this app, and its limitation is worth naming.

`handleDeleteTask` changes signature and body:

```tsx
const handleDeleteTask = (id: string): void => {
  const updatedTasks = tasks.filter(t => t.id !== id);
  setTasks(updatedTasks);
};
```

It now takes an `id` instead of an index. The filter callback is simpler than before: `t => t.id !== id` keeps every task whose id is not the one being deleted. The two-parameter callback with the underscore is gone, because the identity is now in the task itself; the position is not needed. This is a small improvement in readability — the callback says exactly what it means.

The `keyExtractor` changes from using the index to using the id:

```tsx
keyExtractor={item => item.id}
```

The item parameter is no longer named `_item`, because it is now used. The function returns `item.id`. This is the stable identifier that `FlatList` needs to track rows across renders. When a task is removed from the middle of the list, the tasks after it shift position, but their ids do not change. `FlatList` sees that the rows with the remaining ids are the same rows as before, and reuses them rather than re-rendering them. The visual glitches that could appear in Part A do not appear here.

The stylesheet is unchanged from Part A. The change is entirely in the data shape and the functions that operate on it.

### The contrast, side by side

Part A's `handleAddTask`:

```tsx
setTasks([...tasks, task]); // task is a string
```

Part B's `handleAddTask`:

```tsx
const newTask: Task = {
  id: Date.now().toString(),
  text: task,
};
setTasks([...tasks, newTask]); // task is an object
```

Part A's `handleDeleteTask`:

```tsx
const handleDeleteTask = (index: number): void => {
  const updatedTasks = tasks.filter((_, i) => i !== index);
  setTasks(updatedTasks);
};
```

Part B's `handleDeleteTask`:

```tsx
const handleDeleteTask = (id: string): void => {
  const updatedTasks = tasks.filter(t => t.id !== id);
  setTasks(updatedTasks);
};
```

Part A's `keyExtractor`:

```tsx
keyExtractor={(_item, index) => index.toString()}
```

Part B's `keyExtractor`:

```tsx
keyExtractor={item => item.id}
```

Part A's `TaskItem` `onPress`:

```tsx
onPress={() => onDelete(index)}
```

Part B's `TaskItem` `onPress`:

```tsx
onPress={() => onDelete(item.id)}
```

Five small changes, all motivated by the same principle: the identity of a task is its id, not its position. Once that principle is applied, the rest follows.

### Why the id-based version is preferred

The id-based version is preferred because it uses an identifier that travels with the item, rather than one that depends on the item's position in the array. This has three consequences.

First, the delete operation is correct regardless of the order in which updates are processed. If two deletes are queued in the same render, each acts on an id, and the ids are independent. The wrong task cannot be deleted because the wrong index was used.

Second, the `FlatList`'s key is stable. When a task is removed, the tasks after it shift position but keep their ids. `FlatList` recognises the rows by their keys and reuses them correctly. There is no risk of a row's content being attributed to the wrong task.

Third, the id is available wherever the task is. The `TaskItem` receives the whole task and can read `item.id`, `item.text`, or any other property that might be added later. The index is a property of the row in the list, not of the task itself, and it is not available in contexts where the task is being referenced outside the list.

The cost is that the task is now an object rather than a string, which means every place that reads the task has to read `.text` rather than the string directly. That is a small cost, and it is worth paying for the correctness it buys.

### Revision notes and gotchas

- **`Date.now()` is not a proper id generator.** It produces a number of milliseconds since the epoch, which is unique enough for one user adding tasks one at a time. It is not unique if two tasks are added in the same millisecond, and it is not unique across devices. For a production app, use a counter (an incrementing integer stored in state), a `uuid` library, or an id assigned by a server. The in-class version uses `Date.now()` because it is the smallest change that demonstrates the principle.

- **The index is still passed to `TaskItem`, and is still displayed.** The `index` is now used for presentation only: the `Text` shows `{index} - {item.text}`. It is not used for identification. This distinction — index for display, id for identity — is worth naming. It is a common pattern: the visual position of an item in a list is useful for the user, and it is fine to display, but it should not be used as the item's identifier.

- **The `keyExtractor` now uses `item.id`, which is a string.** The `keyExtractor` must return a string; `item.id` is a string, so no conversion is needed. If the id were a number, `.toString()` would be required. `FlatList` will warn if the key is not a string.

- **`filter(t => t.id !== id)` is clearer than `filter((_, i) => i !== index)`.** The first says "keep every task whose id is not the one being deleted". The second says "keep every task whose position is not the one being deleted". The first is an expression of intent; the second is an expression of mechanism. Both work, but the first is easier to read and less likely to be misread.

- **The `Task` type is the beginning of a data model.** Once tasks have structure, more properties can be added — a `done` boolean, a `createdAt` timestamp, a `dueDate` — without changing the way the list works. The id is what makes this possible: each task remains identifiable across changes to its content. This is the direction the app would grow if it continued.

- **The change from `string[]` to `Task[]` touches every function that reads `tasks`.** `handleAddTask`, `handleDeleteTask`, `TaskItem`, and `renderItem` all change. This is the cost of the change, and it is worth noting because it illustrates a general point: the type of the data determines the shape of the code around it. Changing the type is not a local change.

### Key concepts

**Identity versus position.** An index is a position; an id is an identity. Position is meaningful for display and ordering. Identity is meaningful for tracking an item across changes to the list. Lists that can grow, shrink, or reorder need identity for anything that persists — keys, deletion, references. The distinction is the core of Part B.

**Stable keys.** A stable key is one that is the same for the same item across renders, regardless of the item's position. The index is not stable once items can be removed; an id is. `FlatList`'s `keyExtractor` needs a stable key, and the id-based version provides one.

**The data model drives the code.** Once a task is an object with an `id` and a `text`, the functions that operate on tasks are written in terms of those properties. Once it was a string, they were written in terms of the string. The change in the data type propagates through the code, and it is worth being conscious of this whenever the shape of a piece of state is reconsidered.

**The callback prop carries the identity.** `TaskItem`'s `onDelete` now takes a string id. The parent gives the child a function with this shape; the child calls it with `item.id`. The parent's `handleDeleteTask` receives the id and operates on it. The child never sees the array, and the parent never sees the button. The contract between them is a function that takes an id, and both sides of the contract are free to change as long as the shape is honoured.

---

## Stage 6 — Moving TaskItem to Its Own File

### What this stage introduces

Up to now, everything has lived in `App.tsx`: the types, the `TaskItem` component, the `App` component, and the stylesheet. That is fine while the file is short, and it is the right way to start. But the file has been growing across the last several stages, and it now contains two components with distinct responsibilities — the list and the row — sharing a single stylesheet and a single import block. This stage splits it.

`TaskItem` is moved into `components/TaskItem.tsx`. It takes with it the `TaskItemProps` type and the styles that belong to it. What remains in `App.tsx` is the `App` component, the `FlatList`, the `Task` type (still needed for `tasks` state and `renderItem`), and the styles that belong to the app's layout.

This is the first time the project introduces the idea of a file boundary, and it deserves its own stage rather than being treated as a tidy-up. The question being answered is not "how do you move code into a file" — that is mechanical — but "when does a single file stop being a useful unit, and what does a split communicate about the structure of the code?" That is the substance of the stage.

### Why the split is needed

A single file is not a problem in itself. `App.tsx` in Stage 5 is perhaps two hundred lines, which is short enough to read in one sitting, and it would be possible to keep going. But three pressures push toward the split, and they are worth naming because they are the reasons a split is usually justified.

The first is **responsibility**. `App` and `TaskItem` do different things. `App` owns the state, the input, the list, and the deletion logic. `TaskItem` displays one row and reports a delete when the button is pressed. These are separate concerns, and mixing them in one file means that a change to the row's appearance requires opening the same file as a change to the list's behaviour. A reader who wants to understand the row has to skip past the list; a reader who wants to understand the list has to skip past the row. The larger the file, the more skipping there is.

The second is **reuse**. `TaskItem` is a self-contained component. It could be used in a different list, in a different screen, with a different delete handler, and it would work, because it depends only on its props. A component that lives in its own file is available to be imported elsewhere; a component that lives inside `App.tsx` is not. The moment a component becomes reusable, the file boundary makes the reusability visible and practical.

The third is **the size of the stylesheet**. `App`'s styles and `TaskItem`'s styles are different. `container`, `heading`, `inputContainer`, `input`, `addButton`, `addButtonText`, and `tasksContainer` describe the app's layout. `taskItem`, `taskText`, `deleteButton`, and `deleteButtonText` describe one row. They happen to share a stylesheet in Stage 5 because they are in the same file, but they do not need to. Moving `TaskItem` into its own file lets it carry its own styles, which is the correct arrangement: a component's styles are part of its implementation, and they travel with it.

There is a fourth, less obvious reason: a file boundary is a **documentation of a decision**. Saying "there is a file called `TaskItem.tsx`" communicates that `TaskItem` is a component in its own right, that it has its own contract (its props), and that it can be reasoned about independently of `App`. A reader opening the project for the first time sees two files and immediately understands that there are two components. That is a small but real benefit.

None of these reasons requires a split at two hundred lines. They require a split when a component becomes reusable, or when the file's responsibilities are clearly separable, or when the reader would benefit from not having to hold both concerns in mind at once. In this project, all three are true at Stage 5, and the split is a natural next step.

### The code

#### `components/TaskItem.tsx`

```tsx
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

type TaskItemProps = {
  item: string;
  index: number;
  onDelete: (index: number) => void;
};

const TaskItem = ({ item, index, onDelete }: TaskItemProps) => {
  return (
    <View style={styles.taskItem}>
      <Text style={styles.taskText}>{index} - {item}</Text>
      <TouchableOpacity onPress={() => onDelete(index)} style={styles.deleteButton}>
        <Text style={styles.deleteButtonText}>X</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  taskItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 10,
    paddingVertical: 6,
    paddingHorizontal: 10,
    backgroundColor: '#f2f2f2',
    borderRadius: 4,
  },
  taskText: {
    fontSize: 18,
  },
  deleteButton: {
    backgroundColor: '#e53935',
    paddingHorizontal: 10,
    paddingVertical: 4,
    borderRadius: 4,
  },
  deleteButtonText: {
    color: '#fff',
    fontWeight: 'bold',
  },
});

export default TaskItem;
```

#### `App.tsx`

```tsx
import { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  StyleSheet,
} from 'react-native';
import TaskItem from './components/TaskItem'; // import user-defined component

// FlatList - Props specifically for 'renderItem' parameters
type FlatListProps = {
  item: string;
  index: number;
};

function App() {
  const [task, setTask] = useState<string>('');
  const [tasks, setTasks] = useState<string[]>([]);

  const handleAddTask = (): void => {
    // Guard: do nothing when the input is empty.
    // Return early so the code below only runs for a real task.
    if (task === '') {
      return;
    }

    setTasks([...tasks, task]); // add new task to array
    setTask(''); // clear input
  };

  const handleDeleteTask = (index: number): void => {
    const updatedTasks = tasks.filter((_, i) => i !== index);
    setTasks(updatedTasks);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Tasks App</Text>
      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Enter a task"
          value={task}
          onChangeText={(text: string) => setTask(text)}
        />
        <TouchableOpacity style={styles.addButton} onPress={handleAddTask}>
          <Text style={styles.addButtonText}>+</Text>
        </TouchableOpacity>
      </View>
      <View style={styles.tasksContainer}>
        <FlatList
          data={tasks}
          renderItem={({ item, index }: FlatListProps) => (
            <TaskItem item={item} index={index} onDelete={handleDeleteTask} />
          )}
          keyExtractor={(_item, index) => index.toString()}
        />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  heading: {
    fontSize: 24,
    marginBottom: 10,
  },
  inputContainer: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  input: {
    flex: 1,
    height: 40,
    borderColor: 'gray',
    borderWidth: 1,
    paddingHorizontal: 10,
  },
  addButton: {
    width: 40,
    height: 40,
    backgroundColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
  },
  addButtonText: {
    fontSize: 24,
    color: '#fff',
  },
  tasksContainer: {
    marginTop: 20,
  },
});

export default App;
```

### Walkthrough

The two files are read together, because the split is a relationship, not two independent artifacts.

**`components/TaskItem.tsx`** contains four things, in order:

1. The imports from `react-native`. `View`, `Text`, `TouchableOpacity`, and `StyleSheet` are the only `react-native` exports `TaskItem` needs. It does not import `FlatList`, `TextInput`, or anything else from the app — because it does not use them.

2. The `TaskItemProps` type. This declares the shape of the props that `TaskItem` accepts. It lives in this file because it describes this component's contract, and a reader of `TaskItem` needs it. If another file needs the type — for example, to declare a variable of that type — it can be imported from here, since `export default` on the component does not export the type. The type is currently local to the file.

3. The `TaskItem` component itself. It is unchanged from Stage 5 Part A: it takes `item`, `index`, and `onDelete` from its props, renders a `View` with a `Text` and a `TouchableOpacity`, and calls `onDelete(index)` when the button is pressed.

4. A `StyleSheet.create` call with the row's styles. These are the styles that were previously in `App.tsx` — `taskItem`, `taskText`, `deleteButton`, `deleteButtonText`. They have moved with the component because they describe the component's appearance. `App`'s stylesheet no longer contains them.

The file ends with `export default TaskItem;`, which makes the component the module's default export. This is what allows `App.tsx` to import it with `import TaskItem from './components/TaskItem'`.

**`App.tsx`** contains five things, in order:

1. The imports from `react` and `react-native` — unchanged from Stage 5, except for one addition: `import TaskItem from './components/TaskItem';`. This is the import that brings the extracted component into scope. The path `./components/TaskItem` means "the file `TaskItem.tsx` in the `components` subdirectory of the current directory". The `./` prefix is required for relative imports; without it, the module resolver would treat the path as a package name and look in `node_modules`.

2. The `FlatListProps` type. This is unchanged from Stage 4 and Stage 5: it describes the `renderItem` parameter as `{ item: string; index: number }`. It is still needed here because `renderItem` uses it, even though `TaskItem` no longer uses it — `TaskItem` receives `item` and `index` as props, not as the `renderItem` argument.

3. The `App` component. Its structure is unchanged. What is different is what happens at the `renderItem`:

   ```tsx
   renderItem={({ item, index }: FlatListProps) => (
     <TaskItem item={item} index={index} onDelete={handleDeleteTask} />
   )}
   ```

   The `FlatList` still calls this function with `{ item, index }`. The function still returns a `TaskItem`. But now `TaskItem` refers to the imported component from `./components/TaskItem`, not to a component defined in the same file. From `App`'s perspective, nothing has changed: it passes the same props to `TaskItem` as before. The component just happens to live elsewhere.

4. The stylesheet. It now contains only the styles that belong to `App`'s own layout: `container`, `heading`, `inputContainer`, `input`, `addButton`, `addButtonText`, `tasksContainer`. The row's styles are gone, because they moved with `TaskItem`.

5. The default export of `App`.

The net change between Stage 5 and Stage 6 is that the file has been split into two, with `TaskItem`, its props type, and its styles on one side, and `App`, its `FlatListProps` type, its state, its handlers, its JSX, and its styles on the other. The import of `TaskItem` in `App.tsx` reconnects the two halves.

### What the split communicates

The split is small — the code is the same, just in different places — but it changes how the project reads, and the change is worth making explicit.

Before the split, a reader opening `App.tsx` had to hold two things in mind: the app's structure and the row's structure. The file had two functions and two responsibilities, and the reader had to decide which parts belonged to which. After the split, a reader opening `App.tsx` sees only the app's structure, and a reader opening `TaskItem.tsx` sees only the row's structure. Each file has a single purpose, and its name states that purpose.

There is also a change in the kind of question the reader can ask. Before the split, asking "what does the row look like?" required searching for the row inside a file that also contained the list. After the split, the answer is the whole of `TaskItem.tsx`. Asking "what does the list look like?" similarly has a single answer. The file boundary makes the questions separable.

Finally, the split makes the child component's independence visible. `TaskItem` does not import anything from `App`. It does not know that `App` exists. It only knows its props: an item, an index, and a function. If you wanted to use `TaskItem` in a different screen with a different `onDelete`, you would import it and pass the props it needs, and it would work. The file boundary is not just a place to put the code; it is a statement that `TaskItem` can stand on its own.

### Revision notes and gotchas

- **The import path is relative and starts with `./`.** `import TaskItem from './components/TaskItem';` uses `./` to mean "starting from the current directory". Without the `./`, the path would be interpreted as a package name, and the module resolver would look in `node_modules` for a package called `components`, which does not exist. The `.tsx` extension is omitted in the import because the module resolver adds it automatically; writing `'./components/TaskItem.tsx'` also works, but is not conventional.

- **The export is a default export.** `TaskItem.tsx` ends with `export default TaskItem;`, and `App.tsx` imports it with `import TaskItem from './components/TaskItem';`. The default export is the component itself, not a named export. If `TaskItem.tsx` used `export { TaskItem };` instead, the import would need to be `import { TaskItem } from './components/TaskItem';` — with braces. Mixing the two forms is a common source of "X is not defined" errors, and the error is usually about the mismatch between the export style and the import style, not about the component itself.

- **The `TaskItemProps` type is not exported.** It is declared with `type TaskItemProps = { ... }`, which makes it local to `TaskItem.tsx`. If another file needed it — for example, a helper function that takes the same shape — it would need `export type TaskItemProps = { ... }`, and the importing file would write `import type { TaskItemProps } from './components/TaskItem';`. This is not needed in the current code, but it is worth knowing that types need their own export if they are to be shared.

- **`TaskItem.tsx` does not import `FlatList`, `TextInput`, or `React`'s hooks.** It does not use any of them. An import that is present but unused will not break the app, but it is a signal that the file once did something it no longer does, or that it was copied from elsewhere. Keeping imports minimal is a small discipline that pays off when reading a file for the first time.

- **The styles are split by ownership, not by name.** `App`'s stylesheet retains the styles that describe the app's layout, and `TaskItem`'s stylesheet contains the styles that describe the row. This split is not automatic — it required looking at each style and deciding which component it described. The rule used here is: a style belongs to the component that uses it. If `App` uses a style, it stays; if only `TaskItem` uses it, it moves. A style used by both would stay in one place and be imported, but that does not happen in this code.

- **The `renderItem` function is still inline in the `FlatList`.** It could be extracted into a named function in `App`, or defined outside `App` as a standalone function, but it is short and does not depend on anything except the props it receives and the `handleDeleteTask` function from the enclosing scope. Extracting it would make the `FlatList` read as `renderItem={renderTaskItem}`, which is marginally clearer, but the extraction is not required. The in-class code keeps it inline, and there is no correctness reason to change it.

- **The `TaskItemProps` type in `TaskItem.tsx` still says `item: string`.** This is correct for Part A of Stage 5, where tasks are strings. If Part B's id-based deletion were applied, the type would say `item: Task` and `onDelete: (id: string) => void`, and the `Task` type would need to be either declared in `TaskItem.tsx` or imported from a shared location. This is a small illustration of a real problem: when types are shared across files, they usually move to a third file, sometimes called `types.ts`, that both components import from. That refactoring is not done here because it would take the project beyond the scope of the in-class progression, but it is the direction the project would grow.

### Key concepts

**File boundaries as structure.** A file is not just a container. It is a statement about what belongs together and what does not. Splitting a file is a decision about where to draw a boundary — between the list and the row, between the app and a reusable component, between one concern and another. The boundary is made visible by the fact that the code in each file has to reach across it explicitly, through imports. That explicitness is what makes the structure legible.

**Module imports and exports.** A file is a module. What it makes available to other files is determined by its `export` statements: `export default X` makes `X` the module's default export, and `export { X }` (or `export const X`, `export function X`, etc.) makes `X` a named export. What a file uses from another is determined by its `import` statements: `import X from './path'` imports the default export, and `import { X } from './path'` imports a named export. The two forms must match — a default export cannot be imported as a named import, and vice versa — and the mismatch is a common source of errors.

**Reusability as the motivation for extraction.** A component that lives inside another component cannot be imported elsewhere. A component that lives in its own file can. When a component's logic does not depend on its parent except through props, and when it might be used in more than one place, moving it to its own file is the way to make that reusability concrete. The file boundary is the mechanism by which a component becomes a unit that can be assembled into larger structures.

**Styles as component implementation.** A component's styles are part of its implementation. They describe how it looks, in the same way its JSX describes what it renders. Moving a component into its own file moves its styles with it, and the stylesheet stays local to the component that uses it. This is the same reasoning that motivates keeping a function's helper variables inside the function: what belongs to the component belongs with the component.

---

## Closing note

The Tasks App is complete. Reading back through the stages, the arc is small but coherent:

- Stage 0 laid down an empty shell.
- Stage 1 sketched the layout with placeholders.
- Stage 2 introduced state, a controlled input, and `.map()` as a way of rendering an array.
- Stage 3 replaced `.map()` with `FlatList`, which scrolls and virtualises.
- Stage 4 named the shape of the `renderItem` parameter with a type.
- Stage 5 extracted the row into `TaskItem` and introduced deletion, then showed why an id is a better identifier than an index.
- Stage 6 moved `TaskItem` into its own file, and in doing so made the boundary between the list and the row explicit.

The recurring ideas are worth restating, because they are the ones that transfer to other projects.

**State shapes the code around it.** The `tasks` state goes from `string[]` to `Task[]` when the id-based version is introduced, and every function that reads `tasks` changes. This is not a coincidence; it is how type and code relate. When you change the shape of a piece of state, expect to change everything that touches it, and take that as a sign that the state's shape is a design decision, not an incidental one.

**A list needs an identity for each item, and identity is not position.** An index is a position. It happens to coincide with identity for an append-only list, which is why index-based code works at first and then fails. Once the list can change its order or its membership, identity has to be something that travels with the item. The id is that thing.

**Components have contracts.** `TaskItem` accepts an item, an index, and a function. That is its contract. It does not know about the tasks array or the delete handler's implementation. The parent satisfies the contract by passing the right props; the child honours it by calling them as declared. This is the basis of every React component, and it is what allows components to be reused in contexts the original author did not anticipate.

**A file is a boundary you draw, not a container you happen to fill.** The split at Stage 6 is small, but it changes how the project reads. Two files with one purpose each are easier to reason about than one file with two purposes, even if the total amount of code is the same. The choice of where to split is a design decision, and the reasoning behind it — responsibility, reusability, and the size of the stylesheet — is what makes the decision more than a matter of taste.

There is nothing in this app that a larger app does not also do. The patterns — a controlled input, a list with stable keys, a callback prop, a component in its own file — appear in every React Native project, at every scale. The Tasks App is a small place to learn them, and the shape of the code is the same shape it will take when the app is much larger.

---

