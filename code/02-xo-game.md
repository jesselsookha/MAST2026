# 02 The X & O Game (Tic Tac Toe!)

## Stage 0

### Introduction

Every project starts here: an empty canvas. Before there's a game, there's a game *screen* — a single `View` that fills the device, ready to hold whatever comes next. This stage exists so that everyone in the room, regardless of what they've built before, starts from the exact same file.

### Code

```tsx
import { Text, View, StyleSheet } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
});
```

### Concept Notes

`App` is a function that returns JSX — nothing more. At this point it returns a single, empty `View`. The `styles.container` object does the layout work before a single piece of content exists: `flex: 1` tells the view to claim the full available space, and `justifyContent` / `alignItems` centre whatever gets placed inside it later. Setting this up now, while the screen is empty, means every stage that follows inherits a properly centred, full-height canvas without having to think about layout again.

There's no state yet, no interaction, nothing dynamic. That's deliberate — this stage is about the *shape* of a React Native file: an import block, a default-exported function component, and a `StyleSheet.create` call at the bottom. Everything from here on is an elaboration of this same shape, not a departure from it.

---

## Stage 1

### Introduction

With the shape of the file settled in Stage 0, this stage introduces the first real piece of interactivity: collecting two player names before a game can begin. This is the "Player Setup Screen" — nothing about Tic-Tac-Toe itself happens yet, but this is where the game learns *who* is playing.

### Code

```tsx
// App.tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';

export default function App() {
  const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => console.log(`Player 1: ${player1}, Player 2: ${player2}`)}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
});
```

### Concept Notes

Two new ideas arrive together here: **state** and **controlled inputs**.

`const [player1, setPlayer1] = useState<string>('')` creates a piece of state — a value that `App` remembers between renders, unlike a plain variable that would reset every time the function runs. `player1` is the current value; `setPlayer1` is the *only* correct way to change it. Calling `setPlayer1` doesn't just update a variable somewhere — it tells React "this component needs to re-render," and the new value is what comes back next time `App` runs.

The `TextInput` is what makes this a *controlled* input: its `value` prop is bound to `player1`, and its `onChangeText` prop is bound to `setPlayer1`. That pairing means the on-screen text box never manages its own content — React does, via state. Every keystroke fires `onChangeText`, which calls `setPlayer1`, which triggers a re-render, which redraws the `TextInput` with the new `player1` value. It looks instantaneous, but it's a full render cycle every time.

Now to the code-placement distinction worth flagging from this stage onward: notice that `useState` calls and the returned JSX sit directly inside `App`'s body — they run every time `App` renders. The `Button`'s `onPress` handler, by contrast, is an arrow function: `() => console.log(...)`. 

That function is *defined* on every render, but its body — the `console.log` — only *executes* when the button is actually pressed. At this stage the distinction has low stakes, since the handler just prints to the console. But this is the same pattern that `handlePress` will take on in later stages, where what happens inside that arrow function becomes the entire logic of the game. Getting comfortable now with "this runs on every render" versus "this runs only when called" will matter a great deal once state updates start happening inside these handlers.

---

## Stage 2

### Introduction

Up to now, everything lived in one screen. A real Tic-Tac-Toe game needs at least two: a screen to enter names, and a screen to actually play. This stage introduces **navigation** — the mechanism that lets one screen hand control over to another — and splits `App.tsx` into three distinct roles that will remain in place for the rest of the project: `function App()` as the *navigator*, `function PlayerScreen()` as the setup screen, and `function GameScreen()` as the destination.

Because this stage needs a library that isn't part of a bare React Native project, it starts outside the code entirely — in the terminal.

### Installing Navigation

Three statements need to be executed in the terminal (inside the project folder):

1. `npm install @react-navigation/native`
2. `npx expo install react-native-screens react-native-safe-area-context`
3. `npm install @react-navigation/native-stack`

Before touching a single line of code, it's worth understanding what each of these does, because the imports in Part 1 map directly back to them:

- `@react-navigation/native` provides the *core* navigation machinery — the container that manages the overall navigation state for the whole app.
- `react-native-screens` and `react-native-safe-area-context` are supporting libraries that navigation depends on internally, for performance and for handling device-specific safe areas (notches, status bars, and so on). Students won't import these directly, but navigation won't work without them installed.
- `@react-navigation/native-stack` provides the specific *type* of navigator being used here: a stack, where screens are pushed on top of one another and can be popped back off — like a deck of cards.

### XO Game: Player > Game Screen

#### Part 1 — Getting Navigation Working

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={PlayerScreen} />
        <Stack.Screen name="Game" component={GameScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function PlayerScreen({ navigation }: any){
  const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => navigation.navigate('Game')}
      />
    </View>
  );
}

function GameScreen({ navigation, route} : any){
  return(
    <View style={styles.container}>
      <Text style={styles.title}>Player 1 vs. Player 2</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
});
```

#### Part 2 — Passing Data Between Screens

```typescript
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={PlayerScreen} />
        <Stack.Screen name="Game" component={GameScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function PlayerScreen({ navigation }: any){
    const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => navigation.navigate('Game', {player1: player1, player2: player2})}
      />
    </View>
  );
}

function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
});
```

### Concept Notes

**`App` changes role.** Up to Stage 1, `App` *was* the screen. From this stage on, `App` never renders any game content itself — it renders a `NavigationContainer`, which wraps a `Stack.Navigator`, which lists the available screens via `Stack.Screen`.  

`App` has become a controller: its job is to declare which screens exist and which one shows first (`initialRouteName="Home"`), not to display anything directly. This is a shift worth pausing on, because every stage from here onward keeps this same three-part shape: `App` (navigator), `PlayerScreen`, `GameScreen`.

**Why `PlayerScreen` and `GameScreen` are separate functions.** Each `Stack.Screen` needs a `component` to render — that component is just a function that returns JSX, exactly like `App` used to be.  

`PlayerScreen` is effectively the old Stage 1 body, renamed and placed on its own; `GameScreen` is brand new, currently showing only a placeholder title. Splitting the app into functions this way is what makes navigation possible at all — the navigator can only switch between components if there's more than one to switch between.

**Reading `{ navigation }: any` and `{ navigation, route }: any`.** Every component that `Stack.Screen` renders automatically receives a `navigation` object (and a `route` object) as props from React Navigation — the developer doesn't create or pass these; the library supplies them.  

`{ navigation }: any` is destructuring that prop object directly in the function signature — "give me the props, and pull `navigation` out of it immediately." The `: any` is a TypeScript type annotation telling the compiler not to check the shape of these props strictly; it's a shortcut appropriate here because the full type for navigation/route props is more involved than this stage needs. `PlayerScreen` only needs `navigation` (it navigates *away*), while `GameScreen` needs both `navigation` and `route` (route is where incoming data lands).

**Part 1 vs Part 2 — the actual problem being solved.** In Part 1, pressing "Start Game" calls `navigation.navigate('Game')` with no second argument. The navigator dutifully switches to `GameScreen`, but `GameScreen` has no way to know who was typed into those two `TextInput`s — the state `player1` and `player2` lived only inside `PlayerScreen`, and it dies with that component the moment navigation leaves. That's why `GameScreen` in Part 1 hardcodes `"Player 1 vs. Player 2"` — there is nothing else it *could* show.

Part 2 fixes exactly this. `navigation.navigate('Game', {player1: player1, player2: player2})` passes a second argument: an object of parameters.  

React Navigation carries that object along and makes it available to the destination screen as `route.params`. Inside `GameScreen`, `const { player1, player2 } = route.params;` destructures those two values back out. This is the same destructuring pattern from the `ProfileCard` props in Stage 1's Introduction reference — a props (or params) object comes in, and the values needed are pulled out by name.

**Code placement note.** All of this — the `useState` declarations, the destructuring of `route.params`, the JSX return — sits directly in the body of `PlayerScreen` or `GameScreen` and runs on every render of that component. The one exception is the arrow function passed to `onPress`: `() => navigation.navigate('Game', {...})` is defined every render but only executes when the button is actually pressed.  

This is the same pattern from Stage 1, just now triggering a screen change instead of a console log.

---

## Stage 3, Part 1

### Introduction

With names collected and handed over to `GameScreen`, it's time to actually build the board. This part focuses purely on layout: nine empty tiles arranged into a 3×3 grid. No state, no taps that do anything yet — just getting the visual structure of a Tic-Tac-Toe board on screen before any game logic touches it.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={PlayerScreen} />
        <Stack.Screen name="Game" component={GameScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function PlayerScreen({ navigation }: any){
    const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => navigation.navigate('Game', {player1: player1, player2: player2})}
      />
    </View>
  );
}

function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      {/* Board layout: 3 rows of 3 cells, each an empty tile for now */}
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
      </View>
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
      </View>
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
  row: {
    flexDirection: 'row',
  },
  cell: {
    width: 80,
    height: 80,
    borderWidth: 1,
    borderColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
    margin: 2,
  },
  cellText: {
    fontSize: 40,
    fontWeight: 'bold',
  },
});
```

### Concept Notes

A Tic-Tac-Toe board is a grid, and this part builds that grid the most literal way possible: three `View`s styled with `flexDirection: 'row'` (`styles.row`), each containing three `TouchableHighlight` tiles (`styles.cell`). There is no loop, no array, no map — nine tiles are written out by hand, one after another, three per row. That repetition is not an oversight. It is the starting point the rest of Stage 3 will react to.

`TouchableHighlight` is used instead of a plain `View` because each tile needs to eventually respond to a tap — it's the touchable building block, even though nothing is wired to its `onPress` yet. Each one currently wraps an empty `<Text style={styles.cellText}></Text>` — the visual slot where an X or an O will eventually appear.

Notice what has **not** changed since Stage 2, Part 2: `PlayerScreen`, the styles for `title` and `input`, and the overall file shape are identical. Only `GameScreen`'s returned JSX has grown. This is worth pointing out to students directly — when reviewing a new stage, the first useful question is always "what's different from last time?", not "what does the whole file do?"

By the end of this Part, the board *looks* complete but does nothing. Tapping a tile has no effect. That gap — a fully laid-out board with zero interactivity — is exactly what Part 2 begins to close.

---

## Stage 3, Part 2

### Introduction

The board exists visually. Now it needs to know something: whose turn is it? This part introduces the first piece of state tied directly to gameplay — `player1Turn` — and wires every tile's `onPress` to a single shared function, `handlePress`. Nothing gets written to a tile yet; this part is only about tracking and displaying *turns*, and proving that every one of the nine tiles can actually trigger code when pressed.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={PlayerScreen} />
        <Stack.Screen name="Game" component={GameScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function PlayerScreen({ navigation }: any){
    const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => navigation.navigate('Game', {player1: player1, player2: player2})}
      />
    </View>
  );
}

function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true); 
  // initially set to be Player 1 who plays first, always.

  const handlePress = (index: number) => {
    console.log(`Cell ${index} pressed`); 
    setPlayer1Turn(!player1Turn); // changes the player's turn to the next person
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Board layout: 3 rows of 3 cells, each an empty tile for now */}
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(0)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(1)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(2)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
      </View>
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(3)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(4)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(5)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
      </View>
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(6)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(7)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(8)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
  row: {
    flexDirection: 'row',
  },
  cell: {
    width: 80,
    height: 80,
    borderWidth: 1,
    borderColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
    margin: 2,
  },
  cellText: {
    fontSize: 40,
    fontWeight: 'bold',
  },
});
```

### Concept Notes

Two additions land in `GameScreen`: a new piece of state, `player1Turn`, and a new function, `handlePress`, shared by all nine tiles.

`const [player1Turn, setPlayer1Turn] = useState<boolean>(true)` tracks whose turn it currently is as a simple boolean — `true` means Player 1, `false` means Player 2.  

This is displayed directly in the JSX: ```{player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}``` — a ternary that reads the current state and renders one of two possible strings. This line runs on every render of `GameScreen`, so as soon as `player1Turn` changes, this text updates automatically without anything explicitly telling it to.

`handlePress` is where the code-placement distinction becomes important to sit with. `handlePress` itself is defined directly in `GameScreen`'s body — that definition happens on every render. But the *body* of `handlePress` — the `console.log` and the call to `setPlayer1Turn(!player1Turn)` — only runs when a tile is actually tapped. 

Each of the nine `TouchableHighlight`s calls `handlePress` with a different number: `onPress={() => handlePress(0)}` through `handlePress(8)`. That number is the tile's *index* — a label identifying which of the nine positions was pressed. At this point `handlePress` does nothing with that index except print it; it exists here to prove the wiring works, tile by tile, before Part 3 gives that index a real job.

`setPlayer1Turn(!player1Turn)` is the same state-update pattern from earlier stages: it doesn't just flip a variable, it schedules a re-render, and that re-render is what causes the turn text to change on screen. Note also that pressing *any* of the nine tiles toggles the same single `player1Turn` state — there is one shared turn tracker for the whole board, not nine independent ones.

What's still missing, and deliberately so: pressing a tile changes whose turn it is, but nothing yet marks *that tile* as played. There's no memory, per tile, of who pressed it. That's the exact gap Part 3 opens up next — and where the duplication begins in earnest.

---

## Stage 3, Part 3

### Introduction

Part 2 proved that every tile can trigger `handlePress`. Now the game needs to actually remember what's been played where. This part introduces the first piece of *board* state — `block1` — and shows how a single tile's value gets turned into a symbol on screen. Read this part carefully: **only block1 is wired up**. The other eight tiles are still visually present but functionally inert. That gap is not a mistake to fix quietly — it's the problem the rest of Stage 3 exists to solve.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={PlayerScreen} />
        <Stack.Screen name="Game" component={GameScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function PlayerScreen({ navigation }: any){
    const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => navigation.navigate('Game', {player1: player1, player2: player2})}
      />
    </View>
  );
}

function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // Each block holds one of three values:
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [block1, setBlock1] = useState<number>(0);
  const [block2, setBlock2] = useState<number>(0);
  const [block3, setBlock3] = useState<number>(0);
  const [block4, setBlock4] = useState<number>(0);
  const [block5, setBlock5] = useState<number>(0);
  const [block6, setBlock6] = useState<number>(0);
  const [block7, setBlock7] = useState<number>(0);
  const [block8, setBlock8] = useState<number>(0);
  const [block9, setBlock9] = useState<number>(0);

  // Decide what block1's tile should display, based on its current value
  let block1Content: string;
  if (block1 === 0) {
    block1Content = '';
  } else if (block1 === 1) {
    block1Content = 'X';
  } else {
    block1Content = 'O';
  }

  // This above block of statements (declaration and nested if..then) will need to be
  // duplicated for the 8 remaining blocks

  const handlePress = (index: number) => {
    console.log(`Cell ${index} pressed`);
    if (player1Turn){
      setBlock1(1); 
    } else {
      setBlock1(2);
    }
    setPlayer1Turn(!player1Turn);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Board layout: 3 rows of 3 cells
          Note the very first Touchable and its Text component 
      */}
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(0)}>
          <Text style={styles.cellText}>{block1Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(1)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(2)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
      </View>
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(3)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(4)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(5)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
      </View>
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(6)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(7)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(8)}>
          <Text style={styles.cellText}></Text>
        </TouchableHighlight>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
  row: {
    flexDirection: 'row',
  },
  cell: {
    width: 80,
    height: 80,
    borderWidth: 1,
    borderColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
    margin: 2,
  },
  cellText: {
    fontSize: 40,
    fontWeight: 'bold',
  },
});
```

### Concept Notes

Nine pieces of board state appear at once: `block1` through `block9`, each a number defaulting to `0`. The comment above them spells out the convention this entire project now runs on — `0` means unplayed, `1` means Player 1 marked this tile, `2` means Player 2 did.  

This numeric encoding, not `'X'` / `'O'` / `null`, is the actual source of truth for the board. Everything visual is *derived* from these numbers, never the other way around.

That derivation is what the `if / else if / else` block does for `block1`: it reads the number and decides what string to display — `''`, `'X'`, or `'O'` — storing the result in `block1Content`. This runs directly in `GameScreen`'s body, on every render, which matters: every time *anything* on the screen changes and triggers a re-render, this check re-runs and `block1Content` gets recalculated from whatever `block1` currently holds. It is never manually kept in sync — it can't drift out of date, because it's rebuilt from scratch every time.

Now look closely at `handlePress`. It has grown a real `if / else`: `if (player1Turn) { setBlock1(1) } else { setBlock1(2) }`. This is the first time a tile's *ownership* is actually recorded. But read it again — this always sets `block1`, regardless of which index was passed in. Tap tile 5, tile 8, tile 3 — `handlePress` still only ever touches `block1`. The `index` parameter is received and logged, but not yet used to decide *which* block's setter to call. This is the visible bug this Part is built around: the board looks like nine tiles, but only one of them has any memory behind it.

The comment left in the code — *this above block of statements... will need to be duplicated for the 8 remaining blocks* — is doing real pedagogical work. It's naming, out loud, what the "obvious" next step would be: copy the `if/else if/else` eight more times, once per block, changing the number each time. Part 4 does exactly that, and it's worth letting that duplication actually happen and feel repetitive before any refactor is introduced. 

The tedium is the lesson — a shortcut offered too early would rob you of the reason the later refactor (Stage 6) matters at all.

---

## Stage 3, Part 4

### Introduction

This part carries out, in full, the duplication that Part 3's comment predicted: the single `if/else if/else` for `block1Content` becomes a `switch` statement, and that `switch` is copied eight more times — once for every remaining block. The board now visually supports all nine positions. But look carefully at `handlePress`: it has **not changed**. 

It still only ever sets `block1`. This part deliberately separates two different problems — "can the board *display* nine values?" (yes, now) from "can the board *record* nine values?" (still no) — so that each can be understood on its own before Stage 4 closes the second gap.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={PlayerScreen} />
        <Stack.Screen name="Game" component={GameScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function PlayerScreen({ navigation }: any){
    const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => navigation.navigate('Game', {player1: player1, player2: player2})}
      />
    </View>
  );
}

function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // Each block holds one of three values:
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [block1, setBlock1] = useState<number>(0);
  const [block2, setBlock2] = useState<number>(0);
  const [block3, setBlock3] = useState<number>(0);
  const [block4, setBlock4] = useState<number>(0);
  const [block5, setBlock5] = useState<number>(0);
  const [block6, setBlock6] = useState<number>(0);
  const [block7, setBlock7] = useState<number>(0);
  const [block8, setBlock8] = useState<number>(0);
  const [block9, setBlock9] = useState<number>(0);

  // Decide what block's tile should display, based on its current value
  let block1Content: string;
  switch (block1) {
    case 0:
      block1Content = '';
      break;
    case 1:
      block1Content = 'X';
      break;
    case 2:
      block1Content = 'O';
      break;
    default:
      block1Content = '';
  }

  let block2Content: string;
  switch (block2) {
    case 0:
      block2Content = '';
      break;
    case 1:
      block2Content = 'X';
      break;
    case 2:
      block2Content = 'O';
      break;
    default:
      block2Content = '';
  }

  let block3Content: string;
  switch (block3) {
    case 0:
      block3Content = '';
      break;
    case 1:
      block3Content = 'X';
      break;
    case 2:
      block3Content = 'O';
      break;
    default:
      block3Content = '';
  }

  let block4Content: string;
  switch (block4) {
    case 0:
      block4Content = '';
      break;
    case 1:
      block4Content = 'X';
      break;
    case 2:
      block4Content = 'O';
      break;
    default:
      block4Content = '';
  }

  let block5Content: string;
  switch (block5) {
    case 0:
      block5Content = '';
      break;
    case 1:
      block5Content = 'X';
      break;
    case 2:
      block5Content = 'O';
      break;
    default:
      block5Content = '';
  }

  let block6Content: string;
  switch (block6) {
    case 0:
      block6Content = '';
      break;
    case 1:
      block6Content = 'X';
      break;
    case 2:
      block6Content = 'O';
      break;
    default:
      block6Content = '';
  }

  let block7Content: string;
  switch (block7) {
    case 0:
      block7Content = '';
      break;
    case 1:
      block7Content = 'X';
      break;
    case 2:
      block7Content = 'O';
      break;
    default:
      block7Content = '';
  }

  let block8Content: string;
  switch (block8) {
    case 0:
      block8Content = '';
      break;
    case 1:
      block8Content = 'X';
      break;
    case 2:
      block8Content = 'O';
      break;
    default:
      block8Content = '';
  }

  let block9Content: string;
  switch (block9) {
    case 0:
      block9Content = '';
      break;
    case 1:
      block9Content = 'X';
      break;
    case 2:
      block9Content = 'O';
      break;
    default:
      block9Content = '';
  }

  const handlePress = (index: number) => {
    console.log(`Cell ${index} pressed`);
    if (player1Turn){
      setBlock1(1); 
    } else {
      setBlock1(2);
    }
    setPlayer1Turn(!player1Turn);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Board layout: 3 rows of 3 cells - Note the <Text> component values */}
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(0)}>
          <Text style={styles.cellText}>{block1Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(1)}>
          <Text style={styles.cellText}>{block2Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(2)}>
          <Text style={styles.cellText}>{block3Content}</Text>
        </TouchableHighlight>
      </View>
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(3)}>
          <Text style={styles.cellText}>{block4Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(4)}>
          <Text style={styles.cellText}>{block5Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(5)}>
          <Text style={styles.cellText}>{block6Content}</Text>
        </TouchableHighlight>
      </View>
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(6)}>
          <Text style={styles.cellText}>{block7Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(7)}>
          <Text style={styles.cellText}>{block8Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(8)}>
          <Text style={styles.cellText}>{block9Content}</Text>
        </TouchableHighlight>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
  row: {
    flexDirection: 'row',
  },
  cell: {
    width: 80,
    height: 80,
    borderWidth: 1,
    borderColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
    margin: 2,
  },
  cellText: {
    fontSize: 40,
    fontWeight: 'bold',
  },
});
```

### Concept Notes

Nine `switch` statements now sit back-to-back in `GameScreen`, each identical in shape and differing only by which block variable they read and which `blockNContent` variable they assign. 

This is `if/else if/else` traded for `switch` — functionally equivalent, but `switch` reads more clearly once there are exactly three fixed cases (`0`, `1`, `2`) to match against, which is worth naming explicitly: the choice of `switch` over `if/else` here is about readability at this specific shape of problem, not about one being generally superior to the other.

All nine of these run directly in `GameScreen`'s body — on *every* render, all nine execute, regardless of which single block actually changed. If a student presses tile 5, React re-renders `GameScreen`, and all nine switches recalculate their content — including the eight that didn't change. 

This is worth sitting with: it's a small inefficiency, but it illustrates a core React idea plainly stated in the source notes above — state is what *persists* between renders (`block1`...`block9`), while `block1Content`...`block9Content` are *local variables*, recalculated fresh every single render from whatever the state currently holds. They are never stored, never assigned by `handlePress`, never touched outside this switch logic. They exist for exactly one render and are rebuilt from scratch on the next.

Compare this against `handlePress`, which is **unchanged from Part 3** — still only ever calling `setBlock1(...)`. This is the crux of Part 4: display logic and update logic are two separate concerns, and they don't have to be fixed together. The board can *show* nine independent values while still only being able to *record* one. That asymmetry is the whole point of this Part — it isolates the "read" side of the problem from the "write" side, so that Stage 4, which tackles `handlePress`, can be understood as its own distinct piece of work rather than getting tangled up with the switch statements introduced here.

The closing note in the material is worth reading aloud to yourself exactly as written: `handlePress` never touches `block1Content` directly — it only ever calls `setBlock1(...)`. The chain from there is entirely React's doing: the state changes, which causes `GameScreen` to re-render, which causes the switch to run again, which recalculates `block1Content` from the new value. Nothing is manually kept in sync. That chain — state change leads to re-render leads to recalculation — is the mental model this whole project is quietly built to teach, one stage at a time.

---

## Stage 4

### Introduction

Stage 3 ended with a clear asymmetry: the board could *display* all nine positions, but `handlePress` could only ever *record* a mark on `block1`, no matter which tile was actually tapped. Stage 4 closes that gap. The `index` parameter that has been received and quietly logged since Stage 3, Part 2 finally gets used for something real: deciding which of the nine `setBlock` functions to call. This stage also carries something worth pointing out before the code — a commented-out fragment left inside `handlePress`, showing the *previous* version of the logic sitting right next to its replacement. That's not clutter to skip past; it's a deliberate before/after, kept visible.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={PlayerScreen} />
        <Stack.Screen name="Game" component={GameScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function PlayerScreen({ navigation }: any){
    const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => navigation.navigate('Game', {player1: player1, player2: player2})}
      />
    </View>
  );
}

function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // Each block holds one of three values:
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [block1, setBlock1] = useState<number>(0);
  const [block2, setBlock2] = useState<number>(0);
  const [block3, setBlock3] = useState<number>(0);
  const [block4, setBlock4] = useState<number>(0);
  const [block5, setBlock5] = useState<number>(0);
  const [block6, setBlock6] = useState<number>(0);
  const [block7, setBlock7] = useState<number>(0);
  const [block8, setBlock8] = useState<number>(0);
  const [block9, setBlock9] = useState<number>(0);

  // Decide what block's tile should display, based on its current value
  let block1Content: string;
  switch (block1) {
    case 0:
      block1Content = '';
      break;
    case 1:
      block1Content = 'X';
      break;
    case 2:
      block1Content = 'O';
      break;
    default:
      block1Content = '';
  }

  let block2Content: string;
  switch (block2) {
    case 0:
      block2Content = '';
      break;
    case 1:
      block2Content = 'X';
      break;
    case 2:
      block2Content = 'O';
      break;
    default:
      block2Content = '';
  }

  let block3Content: string;
  switch (block3) {
    case 0:
      block3Content = '';
      break;
    case 1:
      block3Content = 'X';
      break;
    case 2:
      block3Content = 'O';
      break;
    default:
      block3Content = '';
  }

  let block4Content: string;
  switch (block4) {
    case 0:
      block4Content = '';
      break;
    case 1:
      block4Content = 'X';
      break;
    case 2:
      block4Content = 'O';
      break;
    default:
      block4Content = '';
  }

  let block5Content: string;
  switch (block5) {
    case 0:
      block5Content = '';
      break;
    case 1:
      block5Content = 'X';
      break;
    case 2:
      block5Content = 'O';
      break;
    default:
      block5Content = '';
  }

  let block6Content: string;
  switch (block6) {
    case 0:
      block6Content = '';
      break;
    case 1:
      block6Content = 'X';
      break;
    case 2:
      block6Content = 'O';
      break;
    default:
      block6Content = '';
  }

  let block7Content: string;
  switch (block7) {
    case 0:
      block7Content = '';
      break;
    case 1:
      block7Content = 'X';
      break;
    case 2:
      block7Content = 'O';
      break;
    default:
      block7Content = '';
  }

  let block8Content: string;
  switch (block8) {
    case 0:
      block8Content = '';
      break;
    case 1:
      block8Content = 'X';
      break;
    case 2:
      block8Content = 'O';
      break;
    default:
      block8Content = '';
  }

  let block9Content: string;
  switch (block9) {
    case 0:
      block9Content = '';
      break;
    case 1:
      block9Content = 'X';
      break;
    case 2:
      block9Content = 'O';
      break;
    default:
      block9Content = '';
  }

  const handlePress = (index: number) => {
    console.log(`Cell ${index} pressed`);

    /*
    if (index === 0) {
      if (player1Turn){
        setBlock1(1); 
      } else {
        setBlock1(2);
      }
    }
    */

    if (index === 0) {
      player1Turn ? setBlock1(1) : setBlock1(2);
    }

    if (index === 1) {
      player1Turn ? setBlock2(1) : setBlock2(2);
    }

    if (index === 2) {
      player1Turn ? setBlock3(1) : setBlock3(2);
    }

    if (index === 3) {
      player1Turn ? setBlock4(1) : setBlock4(2);
    }

    if (index === 4) {
      player1Turn ? setBlock5(1) : setBlock5(2);
    }
    
    if (index === 5) {
      player1Turn ? setBlock6(1) : setBlock6(2);
    }

    if (index === 6) {
      player1Turn ? setBlock7(1) : setBlock7(2);
    }

    if (index === 7) {
      player1Turn ? setBlock8(1) : setBlock8(2);
    }
    
    if (index === 8) {
      player1Turn ? setBlock9(1) : setBlock9(2);
    }

    setPlayer1Turn(!player1Turn);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Board layout: 3 rows of 3 cells */}
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(0)}>
          <Text style={styles.cellText}>{block1Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(1)}>
          <Text style={styles.cellText}>{block2Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(2)}>
          <Text style={styles.cellText}>{block3Content}</Text>
        </TouchableHighlight>
      </View>
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(3)}>
          <Text style={styles.cellText}>{block4Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(4)}>
          <Text style={styles.cellText}>{block5Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(5)}>
          <Text style={styles.cellText}>{block6Content}</Text>
        </TouchableHighlight>
      </View>
      <View style={styles.row}>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(6)}>
          <Text style={styles.cellText}>{block7Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(7)}>
          <Text style={styles.cellText}>{block8Content}</Text>
        </TouchableHighlight>
        <TouchableHighlight style={styles.cell} onPress={() => handlePress(8)}>
          <Text style={styles.cellText}>{block9Content}</Text>
        </TouchableHighlight>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
  row: {
    flexDirection: 'row',
  },
  cell: {
    width: 80,
    height: 80,
    borderWidth: 1,
    borderColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
    margin: 2,
  },
  cellText: {
    fontSize: 40,
    fontWeight: 'bold',
  },
});
```

### Concept Notes

The nine display-side `switch` statements from Stage 3, Part 4 are unchanged here — carried over exactly as they were. Everything new in this stage lives inside `handlePress`, so that's where attention should stay focused.

**Reading the commented-out block first.** The very first thing inside `handlePress` is a `/* ... */` comment containing what looks like ordinary code: `if (index === 0) { if (player1Turn) { setBlock1(1) } else { setBlock1(2) } }`. This is the Stage 3, Part 3 pattern for `block1` alone, deliberately left in place, disabled rather than deleted. 

It's shown here as a bridge: this is the shape the very next `if (index === 0) { player1Turn ? setBlock1(1) : setBlock1(2); }` line is replacing. Reading the two side by side, the nested `if/else` has become a ternary — same outcome, more compact. This is worth pausing on with students: commenting out old code temporarily, rather than deleting it outright, is a legitimate way to keep a working reference nearby while trying a rewrite. It's a habit, not a permanent practice — the comment would normally be removed once the replacement is trusted.

**The nine `if (index === N)` blocks.** This is the actual fix for the Stage 3 gap: `handlePress` now checks `index` against every possible tile position, `0` through `8`, and calls the matching `setBlockN` — `setBlock1` for index `0`, `setBlock2` for index `1`, and so on through `setBlock9` for index `8`. Every one of these runs inside the *body* of `handlePress`, which means none of them execute merely because `GameScreen` rendered — they only run when a tile is pressed, and even then, only one of the nine `if` conditions will actually be true on any given press (since `index` can only match one value). The other eight `if` statements are evaluated, found false, and skipped — quick, but not free. This is a second appearance of the same inefficiency noted with the display-side switches in Stage 3, Part 4: correctness first, efficiency to be considered later.

**What this stage actually achieves.** For the first time, every one of the nine tiles is fully functional: tap any tile, and the *correct* block's state updates, the correct switch recalculates its content, and the correct symbol appears. This is the point where the game becomes genuinely playable — X's and O's can be placed anywhere on the board, by either player, in the correct sequence. What's still missing is any notion of *winning* — nothing yet checks whether three matching marks line up. That's the whole subject of Stage 5.

**Why the duplication persists a little longer.** It would be reasonable to ask why `handlePress` isn't cleaned up here, given the display side was already carrying nine near-identical switches. The answer is sequencing: this stage's job is narrowly to make every tile recordable — proving `index` can correctly route to the right setter. Bundling a refactor into that same step would blur two different lessons together. The cleanup for both the display side and the update side comes later, once the board is fully working and there's a stable, tested version worth refactoring against — that's Stage 6 and Stage 7.

---

## Stage 5

### Introduction

The board is fully playable — every tile can be marked, turns alternate correctly — but nothing yet recognises a win. This stage adds exactly that: a block of checks that inspects the nine board values and decides if three matching marks line up anywhere a player could win. It also introduces the win announcement itself — and that piece of JSX is worth slowing down for, because it doesn't look like anything students have written before.

One correction worth making before the code: the win display uses the `&&` operator, not a ternary (`? :`). It's a different but related pattern, and the Concept Notes below walk through exactly how it works and why it fits this situation better than a ternary would.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={PlayerScreen} />
        <Stack.Screen name="Game" component={GameScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function PlayerScreen({ navigation }: any){
    const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => navigation.navigate('Game', {player1: player1, player2: player2})}
      />
    </View>
  );
}

function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // Each block holds one of three values:
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [block1, setBlock1] = useState<number>(0);
  const [block2, setBlock2] = useState<number>(0);
  const [block3, setBlock3] = useState<number>(0);
  const [block4, setBlock4] = useState<number>(0);
  const [block5, setBlock5] = useState<number>(0);
  const [block6, setBlock6] = useState<number>(0);
  const [block7, setBlock7] = useState<number>(0);
  const [block8, setBlock8] = useState<number>(0);
  const [block9, setBlock9] = useState<number>(0);

  // Decide what block's tile should display, based on its current value
  let block1Content: string;
  switch (block1) {
    case 0:
      block1Content = '';
      break;
    case 1:
      block1Content = 'X';
      break;
    case 2:
      block1Content = 'O';
      break;
    default:
      block1Content = '';
  }

  let block2Content: string;
  switch (block2) {
    case 0:
      block2Content = '';
      break;
    case 1:
      block2Content = 'X';
      break;
    case 2:
      block2Content = 'O';
      break;
    default:
      block2Content = '';
  }

  let block3Content: string;
  switch (block3) {
    case 0:
      block3Content = '';
      break;
    case 1:
      block3Content = 'X';
      break;
    case 2:
      block3Content = 'O';
      break;
    default:
      block3Content = '';
  }

  let block4Content: string;
  switch (block4) {
    case 0:
      block4Content = '';
      break;
    case 1:
      block4Content = 'X';
      break;
    case 2:
      block4Content = 'O';
      break;
    default:
      block4Content = '';
  }

  let block5Content: string;
  switch (block5) {
    case 0:
      block5Content = '';
      break;
    case 1:
      block5Content = 'X';
      break;
    case 2:
      block5Content = 'O';
      break;
    default:
      block5Content = '';
  }

  let block6Content: string;
  switch (block6) {
    case 0:
      block6Content = '';
      break;
    case 1:
      block6Content = 'X';
      break;
    case 2:
      block6Content = 'O';
      break;
    default:
      block6Content = '';
  }

  let block7Content: string;
  switch (block7) {
    case 0:
      block7Content = '';
      break;
    case 1:
      block7Content = 'X';
      break;
    case 2:
      block7Content = 'O';
      break;
    default:
      block7Content = '';
  }

  let block8Content: string;
  switch (block8) {
    case 0:
      block8Content = '';
      break;
    case 1:
      block8Content = 'X';
      break;
    case 2:
      block8Content = 'O';
      break;
    default:
      block8Content = '';
  }

  let block9Content: string;
  switch (block9) {
    case 0:
      block9Content = '';
      break;
    case 1:
      block9Content = 'X';
      break;
    case 2:
      block9Content = 'O';
      break;
    default:
      block9Content = '';
  }

   // Checks all 8 possible winning lines: 3 rows, 3 columns, 2 diagonals
  let winner: number = 0; // which player wins ? 1 or 2
  if (block1 > 0 && block1 === block2 && block2 === block3) winner = block1;
  if (block4 > 0 && block4 === block5 && block5 === block6) winner = block4;
  if (block7 > 0 && block7 === block8 && block8 === block9) winner = block7;
  if (block1 > 0 && block1 === block4 && block4 === block7) winner = block1;
  if (block2 > 0 && block2 === block5 && block5 === block8) winner = block2;
  if (block3 > 0 && block3 === block6 && block6 === block9) winner = block3;
  if (block1 > 0 && block1 === block5 && block5 === block9) winner = block1;
  if (block3 > 0 && block3 === block5 && block5 === block7) winner = block3;

  const handlePress = (index: number) => {
    if (index === 0) {
      player1Turn ? setBlock1(1) : setBlock1(2);
    }

    if (index === 1) {
      player1Turn ? setBlock2(1) : setBlock2(2);
    }

    if (index === 2) {
      player1Turn ? setBlock3(1) : setBlock3(2);
    }

    if (index === 3) {
      player1Turn ? setBlock4(1) : setBlock4(2);
    }

    if (index === 4) {
      player1Turn ? setBlock5(1) : setBlock5(2);
    }
    
    if (index === 5) {
      player1Turn ? setBlock6(1) : setBlock6(2);
    }

    if (index === 6) {
      player1Turn ? setBlock7(1) : setBlock7(2);
    }

    if (index === 7) {
      player1Turn ? setBlock8(1) : setBlock8(2);
    }
    
    if (index === 8) {
      player1Turn ? setBlock9(1) : setBlock9(2);
    }

    setPlayer1Turn(!player1Turn);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Board layout: 3 rows of 3 cells */}
      <View>
        <View style={styles.row}>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(0)}>
            <Text style={styles.cellText}>{block1Content}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(1)}>
            <Text style={styles.cellText}>{block2Content}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(2)}>
            <Text style={styles.cellText}>{block3Content}</Text>
          </TouchableHighlight>
        </View>
        <View style={styles.row}>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(3)}>
            <Text style={styles.cellText}>{block4Content}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(4)}>
            <Text style={styles.cellText}>{block5Content}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(5)}>
            <Text style={styles.cellText}>{block6Content}</Text>
          </TouchableHighlight>
        </View>
        <View style={styles.row}>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(6)}>
            <Text style={styles.cellText}>{block7Content}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(7)}>
            <Text style={styles.cellText}>{block8Content}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(8)}>
            <Text style={styles.cellText}>{block9Content}</Text>
          </TouchableHighlight>
        </View>
      </View>

      {/* A conditional statement that will display the respective winner's name 
      	  based on the value of winner. Note that there are 2 separate <Text> 
          components being coded
      */}      
      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
  row: {
    flexDirection: 'row',
  },
  cell: {
    width: 80,
    height: 80,
    borderWidth: 1,
    borderColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
    margin: 2,
  },
  cellText: {
    fontSize: 40,
    fontWeight: 'bold',
  },
});
```

### Concept Notes

**Two small housekeeping changes before the main content.** The `console.log` that used to open `handlePress` is gone — it did its job of proving the wiring worked back in Stage 3, Part 2, and by this stage it's just noise. And the three board rows are now wrapped in an extra plain `<View>` — a small structural change that doesn't affect layout on its own here, but groups "the board" as one unit, separate from the turn text above it and the winner announcement below it. It is Worth spotting both of these as "what changed" before moving on.

**Why eight `if` statements, and why they don't chain with `else`.** A win requires three *matching, non-zero* values in a straight line — and a Tic-Tac-Toe board has exactly eight such lines: three rows, three columns, two diagonals. Each `if` checks one specific line: `if (block1 > 0 && block1 === block2 && block2 === block3) winner = block1;` reads as "block1 has been played, and block1 equals block2, and block2 equals block3" — three separate conditions joined with `&&`, all of which must be true for that line to count as a win. The `> 0` check matters: without it, three empty blocks (all `0`) would satisfy `block1 === block2 && block2 === block3` and incorrectly declare a winner before the game even starts.

These eight checks are **not** `else if` — they're eight independent `if` statements, each able to run regardless of what the ones before it decided. This is intentional: unlike the earlier `switch` statements (which test one variable against fixed cases, where only one case can ever be true), these eight checks test entirely different combinations of blocks. Nothing about one line being a non-win tells you anything about whether a different line is a win, so there's no shared condition to branch on with `else`. Each line has to be checked on its own merits.

`winner` starts at `0` and is only reassigned inside these `if` blocks — `winner = block1` (or `block4`, or `block3`, depending on the line), because `block1` at that point already holds `1` or `2`, exactly the value that identifies the winning player. If no line matches, `winner` stays `0`, and nothing is announced.

**The win display — reading `{winner === 1 && <Text>...</Text>}` correctly.** This is the piece worth the most care, because it does look unfamiliar, and it's genuinely a different pattern from the ternary (`condition ? A : B`) used earlier for the turn indicator. The `&&` operator here is being used for what's often called a *short-circuit* pattern. In JavaScript, `&&` evaluates its left side first; if that left side is `false`, `&&` stops immediately and the whole expression evaluates to `false` — the right side is never even looked at. If the left side is `true`, `&&` evaluates and returns the right side instead.

Applied here: `winner === 1` is either `true` or `false`. If it's `false`, the whole expression `winner === 1 && <Text>...</Text>` becomes `false` — and React, when asked to render `false`, renders nothing at all. If `winner === 1` is `true`, the expression evaluates to whatever's on the right: the `<Text>` element itself, which React then renders normally. So this single line either renders a `<Text>` showing `"{player1} Wins!"` or renders nothing, purely depending on `winner`.

The reason there are **two** of these — one for `winner === 1`, one for `winner === 2` — rather than one ternary, is that a ternary always produces exactly one of two outcomes (`A` or `B`), whereas here there are genuinely *three* possible outcomes: Player 1 wins, Player 2 wins, or nobody has won yet (game still in progress, or a draw). Two independent `&&` checks handle this naturally — each one only renders when its specific condition is met, and if neither is met, both render nothing, which correctly represents "no winner yet." A ternary would have forced a choice between only two outcomes and wouldn't have left room for "neither."

**The curly braces themselves — why this looks different from ordinary JSX.** In every previous stage, what's gone inside a `View` has been other components, written directly: `<Text>...</Text>`, `<TouchableHighlight>...</TouchableHighlight>`. Here, instead, `{winner === 1 && <Text>...</Text>}` is a piece of *JavaScript* sitting inside curly braces, in the middle of JSX. That's the general rule worth naming explicitly: anywhere JSX allows a value — inside curly braces — actual JavaScript can run, and whatever that JavaScript *evaluates to* becomes what gets rendered there. It's the same rule that already made `{player1Turn ? ... : ...}` work for the turn text; this stage is really the second application of a pattern already seen, just using a different operator and producing either "something" or "nothing" instead of always producing one of two somethings.

---

## Stage 6

### Introduction

Every stage since Stage 3 has been building *outward* — adding blocks, adding win checks, adding a winner display. This stage is the first to build *inward*: no new game behaviour is introduced, and the game plays exactly as it did in Stage 5. What changes is the shape of the code underneath it. This is refactoring — restructuring working code without changing what it does — and it's worth spending real time on *why* it happens here and not earlier.

Before the code: one new, small addition unrelated to the refactor sits inside `handlePress` — `if (winner !== 0) return;`. Worth noticing that first, since it's easy to miss inside a stage that's mostly about deletion.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={PlayerScreen} />
        <Stack.Screen name="Game" component={GameScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function PlayerScreen({ navigation }: any){
    const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => navigation.navigate('Game', {player1: player1, player2: player2})}
      />
    </View>
  );
}

function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // Each block holds one of three values:
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [block1, setBlock1] = useState<number>(0);
  const [block2, setBlock2] = useState<number>(0);
  const [block3, setBlock3] = useState<number>(0);
  const [block4, setBlock4] = useState<number>(0);
  const [block5, setBlock5] = useState<number>(0);
  const [block6, setBlock6] = useState<number>(0);
  const [block7, setBlock7] = useState<number>(0);
  const [block8, setBlock8] = useState<number>(0);
  const [block9, setBlock9] = useState<number>(0);

  // Decide what block's tile should display, based on its current value
  const contentFor = (block: number): string => {
    switch (block) {
      case 1:
        return 'X';
      case 2:
        return 'O';
      default:
        return '';
    }
  };

   // Checks all 8 possible winning lines: 3 rows, 3 columns, 2 diagonals
  let winner: number = 0;
  if (block1 > 0 && block1 === block2 && block2 === block3) winner = block1;
  if (block4 > 0 && block4 === block5 && block5 === block6) winner = block4;
  if (block7 > 0 && block7 === block8 && block8 === block9) winner = block7;
  if (block1 > 0 && block1 === block4 && block4 === block7) winner = block1;
  if (block2 > 0 && block2 === block5 && block5 === block8) winner = block2;
  if (block3 > 0 && block3 === block6 && block6 === block9) winner = block3;
  if (block1 > 0 && block1 === block5 && block5 === block9) winner = block1;
  if (block3 > 0 && block3 === block5 && block5 === block7) winner = block3;

  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen

    // how do we stop the user from pressing the same button twice? 

    // still need to refactor the below statements
    if (index === 0) {
      player1Turn ? setBlock1(1) : setBlock1(2);
    }

    if (index === 1) {
      player1Turn ? setBlock2(1) : setBlock2(2);
    }

    if (index === 2) {
      player1Turn ? setBlock3(1) : setBlock3(2);
    }

    if (index === 3) {
      player1Turn ? setBlock4(1) : setBlock4(2);
    }

    if (index === 4) {
      player1Turn ? setBlock5(1) : setBlock5(2);
    }
    
    if (index === 5) {
      player1Turn ? setBlock6(1) : setBlock6(2);
    }

    if (index === 6) {
      player1Turn ? setBlock7(1) : setBlock7(2);
    }

    if (index === 7) {
      player1Turn ? setBlock8(1) : setBlock8(2);
    }
    
    if (index === 8) {
      player1Turn ? setBlock9(1) : setBlock9(2);
    }

    setPlayer1Turn(!player1Turn);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Board layout: 3 rows of 3 cells */}
      <View>
        <View style={styles.row}>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(0)}>
            <Text style={styles.cellText}>{contentFor(block1)}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(1)}>
            <Text style={styles.cellText}>{contentFor(block2)}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(2)}>
            <Text style={styles.cellText}>{contentFor(block3)}</Text>
          </TouchableHighlight>
        </View>
        <View style={styles.row}>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(3)}>
            <Text style={styles.cellText}>{contentFor(block4)}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(4)}>
            <Text style={styles.cellText}>{contentFor(block5)}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(5)}>
            <Text style={styles.cellText}>{contentFor(block6)}</Text>
          </TouchableHighlight>
        </View>
        <View style={styles.row}>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(6)}>
            <Text style={styles.cellText}>{contentFor(block7)}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(7)}>
            <Text style={styles.cellText}>{contentFor(block8)}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(8)}>
            <Text style={styles.cellText}>{contentFor(block9)}</Text>
          </TouchableHighlight>
        </View>
      </View>

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
  row: {
    flexDirection: 'row',
  },
  cell: {
    width: 80,
    height: 80,
    borderWidth: 1,
    borderColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
    margin: 2,
  },
  cellText: {
    fontSize: 40,
    fontWeight: 'bold',
  },
});
```

### Concept Notes

**The small addition first.** `if (winner !== 0) return;` is the very first line inside `handlePress` now. This is a *guard clause* — a condition checked at the top of a function that exits early if some precondition isn't met. Once `winner` holds `1` or `2`, this line means every subsequent tap on the board does nothing at all; `handlePress` returns immediately, before reaching any of the `if (index === N)` checks. 

Without this, a user could keep playing tiles after a win was already declared, silently corrupting the board state for no visible reason. This one line is a genuine behavioural fix — small, but real — sitting inside a stage that's otherwise about restructuring, not adding.

**Why refactor here, and not earlier.** Go back to Stage 3, Part 4: nine `switch` statements were written, each one identical in shape and differing only by which block they read. At that point, refactoring them into one shared function would have been *possible*, but premature — the game didn't work yet, `handlePress` only ever touched `block1`, and there was no way to confirm the display logic was even correct across all nine positions. 

Refactoring code that hasn't been proven to work risks fixing the wrong thing, or masking a bug inside a rewrite. By Stage 5, the full game was working end to end: every tile could be marked, wins were detected correctly, the winner displayed correctly. Only once that was true did it become safe to ask "does this working code have to look this repetitive?" That ordering — get it working first, then simplify — is the rule worth stating plainly to you: **refactor once you trust the behaviour, not before.**

**What `contentFor` replaces.** The nine `switch` statements and nine `blockNContent` variables from Stage 3, Part 4 and carried through Stage 4 and Stage 5 are gone entirely. In their place is a single arrow function:

```
const contentFor = (block: number): string => {
  switch (block) {
    case 1: return 'X';
    case 2: return 'O';
    default: return '';
  }
};
```

This is the same three-case logic as before, written once instead of nine times. The difference is that `contentFor` takes a parameter — `block` — rather than closing over one specific state variable like `block1`. That's what makes reuse possible: `contentFor(block1)`, `contentFor(block2)`, ... `contentFor(block9)` all call the *same* function body, just supplying a different value each time. Notice this also drops the explicit `case 0` — since `default` already returns `''`, and `0` is the only remaining unhandled case, the `default` branch quietly covers it. That's a small tightening worth pointing out, not a change in behaviour.

**Where this fits the render-cycle model already built up.** `contentFor` is *defined* once when `GameScreen` renders, exactly like `handlePress` has been since Stage 3, Part 2. But unlike `handlePress`, which only runs when a tile is pressed, `contentFor` is *called* directly inside the JSX — `{contentFor(block1)}`, `{contentFor(block2)}`, and so on — which means it still runs on every render, once per tile, exactly as the nine switches did before. 

The refactor changes how the logic is *written*, not when it *runs*. This is worth being explicit about with students: refactoring for readability doesn't automatically mean a change in performance or behaviour, and shouldn't be expected to.

**A general note on when refactoring is, and isn't, warranted.** This is a good moment to broaden the conversation beyond this one function. Refactoring is worth doing when: the same logic appears in multiple places and any future change would need to be made in all of them consistently; the repeated shape makes the code harder to read than a single reusable version would; and — critically — there's already a working, tested version to refactor *from*, so any mistake introduced during the rewrite is easy to notice by comparing behaviour before and after. 

Refactoring is premature, or simply unnecessary, when: the code isn't yet proven correct; the "duplication" is only superficial (two pieces of code that look similar but represent genuinely different concerns, which a shared function would awkwardly force together); or when the rewrite would make the code harder to follow for the sake of being shorter. Stage 6 is a clean example of the first case. 

Stage 7, next, applies the exact same reasoning to the *update* side of the code — `handlePress` still has its own nine near-identical blocks, deliberately left untouched here so this stage could stay focused on one refactor at a time.

---

## Stage 7

### Introduction

Stage 6 refactored the *display* side — nine `switch` statements became one `contentFor` function. This stage applies that exact same reasoning to the *update* side: the nine `if (index === N)` blocks inside `handlePress`, present unchanged since Stage 4, are finally collapsed into a single reusable pattern. This closes the loop opened back in Stage 3, Part 3 — display and update logic have now each been through the same journey: duplicate first, prove it works, then simplify.

A handful of key comments are added into the code below — not line-by-line narration, but the kind of comment a developer leaves for their future self, or for a teammate picking this up cold. This is a good stage to introduce that habit, since the code has reached a point worth explaining at a glance.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={PlayerScreen} />
        <Stack.Screen name="Game" component={GameScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function PlayerScreen({ navigation }: any){
    const [player1, setPlayer1] = useState<string>('');
  const [player2, setPlayer2] = useState<string>('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Tic-Tac-Toe!</Text>

      <TextInput
        placeholder="Player 1 Name"
        value={player1}
        onChangeText={setPlayer1}
        style={styles.input}
      />
      <TextInput
        placeholder="Player 2 Name"
        value={player2}
        onChangeText={setPlayer2}
        style={styles.input}
      />

      <Button
        title="Start Game"
        onPress={() => navigation.navigate('Game', {player1: player1, player2: player2})}
      />
    </View>
  );
}

function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // Each block holds one of three values:
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [block1, setBlock1] = useState<number>(0);
  const [block2, setBlock2] = useState<number>(0);
  const [block3, setBlock3] = useState<number>(0);
  const [block4, setBlock4] = useState<number>(0);
  const [block5, setBlock5] = useState<number>(0);
  const [block6, setBlock6] = useState<number>(0);
  const [block7, setBlock7] = useState<number>(0);
  const [block8, setBlock8] = useState<number>(0);
  const [block9, setBlock9] = useState<number>(0);

  // Decide what block's tile should display, based on its current value
  const contentFor = (block: number): string => {
    switch (block) {
      case 1:
        return 'X';
      case 2:
        return 'O';
      default:
        return '';
    }
  };

  // Checks all 8 possible winning lines: 3 rows, 3 columns, 2 diagonals
  // winner stays 0 while the game is still in progress or drawn
  let winner: number = 0;
  if (block1 > 0 && block1 === block2 && block2 === block3) winner = block1;
  if (block4 > 0 && block4 === block5 && block5 === block6) winner = block4;
  if (block7 > 0 && block7 === block8 && block8 === block9) winner = block7;
  if (block1 > 0 && block1 === block4 && block4 === block7) winner = block1;
  if (block2 > 0 && block2 === block5 && block5 === block8) winner = block2;
  if (block3 > 0 && block3 === block6 && block6 === block9) winner = block3;
  if (block1 > 0 && block1 === block5 && block5 === block9) winner = block1;
  if (block3 > 0 && block3 === block5 && block5 === block7) winner = block3;

  // handlePress is shared by all nine tiles.
  // block: the tile's current value (0, 1, or 2)
  // setBlock: the specific setter for that tile (setBlock1 ... setBlock9)
  // Passing the setter in as a parameter is what allows one function
  // to update any of the nine tiles, instead of needing nine functions.
  const handlePress = (block: number, setBlock: (value: number) => void) => {
    if (winner !== 0) return; // stop game after winner is chosen

    // how do we stop the user from pressing the same button twice? 
    if (block !== 0) {
      return; // tile already played
    }

    // set the block based on who played it
    player1Turn ? setBlock(1) : setBlock(2);

    setPlayer1Turn(!player1Turn);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Board layout: 3 rows of 3 cells */}
      <View>
        <View style={styles.row}>
          <TouchableHighlight 
	        style={styles.cell} 
            onPress={() => handlePress(block1, setBlock1)}
          >
            <Text style={styles.cellText}>{contentFor(block1)}</Text>
          </TouchableHighlight>
          <TouchableHighlight 
	        style={styles.cell} 
            onPress={() => handlePress(block2, setBlock2)}
       	  >
            <Text style={styles.cellText}>{contentFor(block2)}</Text>
          </TouchableHighlight>
          <TouchableHighlight 
	        style={styles.cell} 
            onPress={() => handlePress(block3, setBlock3)}
          >
            <Text style={styles.cellText}>{contentFor(block3)}</Text>
          </TouchableHighlight>
        </View>
        <View style={styles.row}>
          <TouchableHighlight 
	        style={styles.cell} 
            onPress={() => handlePress(block4, setBlock4)}
          >
            <Text style={styles.cellText}>{contentFor(block4)}</Text>
          </TouchableHighlight>
          <TouchableHighlight 
	        style={styles.cell} 
            onPress={() => handlePress(block5, setBlock5)}
          >
            <Text style={styles.cellText}>{contentFor(block5)}</Text>
          </TouchableHighlight>
          <TouchableHighlight 
	        style={styles.cell} 
            onPress={() => handlePress(block6, setBlock6)}
          >
            <Text style={styles.cellText}>{contentFor(block6)}</Text>
          </TouchableHighlight>
        </View>
        <View style={styles.row}>
          <TouchableHighlight 
	        style={styles.cell} 
	        onPress={() => handlePress(block7, setBlock7)}
          >
            <Text style={styles.cellText}>{contentFor(block7)}</Text>
          </TouchableHighlight>
          <TouchableHighlight 
	        style={styles.cell} 
            onPress={() => handlePress(block8, setBlock8)}
          >
            <Text style={styles.cellText}>{contentFor(block8)}</Text>
          </TouchableHighlight>
          <TouchableHighlight 
            style={styles.cell} 
            onPress={() => handlePress(block9, setBlock9)}
          >
            <Text style={styles.cellText}>{contentFor(block9)}</Text>
          </TouchableHighlight>
        </View>
      </View>

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginVertical: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 8,
    width: '80%',
  },
  row: {
    flexDirection: 'row',
  },
  cell: {
    width: 80,
    height: 80,
    borderWidth: 1,
    borderColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
    margin: 2,
  },
  cellText: {
    fontSize: 40,
    fontWeight: 'bold',
  },
});
```

### Concept Notes

**The core idea: passing a setter as a parameter.** `handlePress` now takes two arguments — `block: number` and `setBlock: (value: number) => void`. The second one is the interesting part: `setBlock` isn't a value like a number or a string, it's a *function itself*, being passed in the same way a number would be. 

This is only possible because functions in JavaScript are treated as ordinary values — they can be stored in variables, passed as arguments, and called later by whatever received them. Each `TouchableHighlight` now calls `handlePress(block1, setBlock1)`, `handlePress(block2, setBlock2)`, and so on — supplying both the tile's current value *and* the specific setter that belongs to that tile. 

Inside `handlePress`, the line `player1Turn ? setBlock(1) : setBlock(2)` doesn't know or care whether it's updating `block1` or `block7` — it just calls whichever setter was handed to it. This is precisely what turned nine near-identical `if (index === N)` blocks into a single reusable function, mirroring what `contentFor` did for the display side in Stage 6.

**A second, quieter fix worth pointing out.** `handlePress` gained a guard beyond the win check: `if (block !== 0) { return; }`. Previously, nothing stopped a player from tapping an already-marked tile and overwriting it. Now, if the tile's current value isn't `0` (unplayed), the function exits immediately. This directly answers the question left as a comment in the code itself — *"how do we stop the user from pressing the same button twice?"* — and it's answered using the same guard-clause pattern introduced in Stage 6 for the win condition. 

Worth noting: this fix only became possible *because* `handlePress` now receives `block` (the tile's current value) as a parameter — in earlier stages, `handlePress` only ever received `index`, the tile's position, and had no way to check what that tile already held.

**Reading the new comments.** The comments added at this stage don't explain every line — they explain *decisions*: what each state block represents, why `winner` can stay at `0`, and — most importantly — why `handlePress` takes a setter as a parameter rather than an index. 

That last comment is doing real work: it's the kind of note that saves a future reader (a user returning to this file months later) from having to reverse-engineer *why* the function is shaped this way. Good code comments explain reasoning and intent that isn't obvious just from reading the code; they don't restate what the code already says.

**What's still open.** The board is fully playable, wins are detected and displayed, tiles can't be overwritten, and the game correctly halts once someone wins. What isn't handled yet: a draw (all nine tiles filled, no winner), and any way to reset or replay without restarting the whole app. Those remain open threads for whatever comes next in this storyline — deliberately left that way, matching how it played out in class.

---

## Closing: Stages 0 through 7

Looking back at the full arc, a single thread runs through it, and it's worth naming plainly for students working through this on their own: **this project was never really about Tic-Tac-Toe.** It was about watching the same three ideas surface, get stretched to their breaking point, and get resolved — twice, on two different sides of the same code.

- **State versus derived values.** From Stage 3 onward, the project draws a hard line between what's *remembered* (`block1` through `block9`, `player1Turn`, `winner`) and what's *recalculated* fresh on every render (`block1Content`, then `contentFor(block1)`). Nothing on screen is ever set directly — it's always read back out of state, every render, from scratch.
- **Duplication before abstraction.** Stages 3 and 4 wrote the same shape of code nine times over — first for display, then for updates — before Stages 6 and 7 collapsed each into one reusable function. That ordering was deliberate throughout: working code came first, simplification came only once the behaviour could be trusted.
- **The two halves of a component.** Every stage from 3 onward split cleanly into "what gets shown" (the switches, then `contentFor`) and "what gets recorded" (the nine `if` blocks, then the parameterised `handlePress`). Keeping those two concerns visibly separate — even while both were still duplicated — made each one easier to reason about, and easier to refactor, on its own terms.

For a student catching up from this document alone: the destination — a working, fairly clean two-screen Tic-Tac-Toe game — matters less than the route taken to get there. Stage 3's deliberately half-broken `handlePress`, Stage 4's commented-out old code sitting next to its replacement, Stage 6's explicit "why refactor now and not sooner" — these aren't detours from the "real" lesson. They are the lesson, staged out one decision at a time.

This is Version 1. Version 2 picks up from here — likely toward handling a draw, resetting the game, and whatever else the logic still leaves open. That's for the next document.