# 06 The X & O Game (Tic Tac Toe!) 

> Continuing from [02-x0-game](https://github.com/jesselsookha/MAST2026/blob/main/code/02-xo-game.md)

## Introduction — Version 2

### Recap: Stages 0–7

Version 1 took the X & O game from nothing to a fully working two-player board, one deliberate step at a time. Stage 0 began with an empty screen. Stage 1 added the Player Setup form — two `TextInput`s, controlled by state. Stage 2 introduced navigation, splitting one function into three permanent pieces: `App` as navigator, `PlayerScreen`, and `GameScreen`, with player names carried between them via `route.params`. Stage 3 built the board itself, across four parts — static layout, then turn tracking, then the first (deliberately incomplete) board state, then a display refactor that outpaced the update logic on purpose, so each half of the problem could be understood on its own. Stage 4 closed that gap, making every one of the nine tiles genuinely playable. Stage 5 added win detection — eight independent checks, one per possible line — and introduced the `&&` short-circuit pattern for displaying a winner. Stages 6 and 7 then refactored what Stage 3 and 4 had deliberately left duplicated: nine `switch` statements became one `contentFor` function, and nine `if (index === N)` blocks became one `handlePress` taking a block's value and its setter as parameters.

That was Version 1's finish line: a working, playable game, with its two most obvious pieces of repetition cleaned up — but still nine separate state variables, nine separate `TouchableHighlight`s, eight separate win-check `if` statements, and everything living inside a single `App.tsx` file.

### What Version 2 covers

Version 2 doesn't change what the game does — it changes how the code is *shaped*, pushing the same refactoring instinct from Stages 6 and 7 much further. Four themes run through the stages ahead:

- **Arrays over individual variables** — nine `block` state variables become one array, and the nine-fold repetition that array makes possible gets removed from both the board layout and the win-check logic.
- **Structured win-checking** — eight hand-written `if` statements become a small, data-driven list of winning combinations, walked with a loop.
- **File and folder structure** — `App.tsx` currently holds everything: the navigator, both screens, and all the game logic. Version 2 introduces a proper `src/` structure and moves code into dedicated files — `gameLogic.ts`, separate screen files, a typed navigation file, and eventually a reusable, user-defined `Cell` component with its own `Props` interface.
- **Typed navigation** — the `{ navigation }: any` shortcut from Stage 2 gets replaced with a proper `RootStackParamList`, giving TypeScript real visibility into what each screen expects.

As before, every stage presents complete, working code — nothing abbreviated, nothing left as "the rest is the same." This is Response 1 of 8, covering the Introduction and Stage 8.

---

## Stage 8

### Introduction

Nine separate state variables — `block1` through `block9` — have been doing the same job since Stage 3: each one holds a single number, `0`, `1`, or `2`. This stage replaces all nine with a single array, `blocks`. It looks like a small change, but it forces a real rethink of `handlePress`, because there's no longer a `setBlock1`, `setBlock2`, and so on to pass around — there's only one `setBlocks`, for the whole array. This is the first stage where **immutability** becomes something students need to actively think about: array state can't just be edited in place, it has to be replaced with a new array every time it changes.

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

  // All nine tile values now live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>([0, 0, 0, 0, 0, 0, 0, 0, 0]);

  // Decide what a block's tile should display, based on its current value
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
  if (blocks[0] > 0 && blocks[0] === blocks[1] && blocks[1] === blocks[2]) winner = blocks[0];
  if (blocks[3] > 0 && blocks[3] === blocks[4] && blocks[4] === blocks[5]) winner = blocks[3];
  if (blocks[6] > 0 && blocks[6] === blocks[7] && blocks[7] === blocks[8]) winner = blocks[6];
  if (blocks[0] > 0 && blocks[0] === blocks[3] && blocks[3] === blocks[6]) winner = blocks[0];
  if (blocks[1] > 0 && blocks[1] === blocks[4] && blocks[4] === blocks[7]) winner = blocks[1];
  if (blocks[2] > 0 && blocks[2] === blocks[5] && blocks[5] === blocks[8]) winner = blocks[2];
  if (blocks[0] > 0 && blocks[0] === blocks[4] && blocks[4] === blocks[8]) winner = blocks[0];
  if (blocks[2] > 0 && blocks[2] === blocks[4] && blocks[4] === blocks[6]) winner = blocks[2];

  // handlePress now only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Board layout: 3 rows of 3 cells, each an empty tile for now */}
      <View>
        <View style={styles.row}>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(0)}>
            <Text style={styles.cellText}>{contentFor(blocks[0])}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(1)}>
            <Text style={styles.cellText}>{contentFor(blocks[1])}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(2)}>
            <Text style={styles.cellText}>{contentFor(blocks[2])}</Text>
          </TouchableHighlight>
        </View>
        <View style={styles.row}>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(3)}>
            <Text style={styles.cellText}>{contentFor(blocks[3])}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(4)}>
            <Text style={styles.cellText}>{contentFor(blocks[4])}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(5)}>
            <Text style={styles.cellText}>{contentFor(blocks[5])}</Text>
          </TouchableHighlight>
        </View>
        <View style={styles.row}>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(6)}>
            <Text style={styles.cellText}>{contentFor(blocks[6])}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(7)}>
            <Text style={styles.cellText}>{contentFor(blocks[7])}</Text>
          </TouchableHighlight>
          <TouchableHighlight style={styles.cell} onPress={() => handlePress(8)}>
            <Text style={styles.cellText}>{contentFor(blocks[8])}</Text>
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

### Explanation — Before / After

**State declarations.**

Before (Stage 7):
```
const [block1, setBlock1] = useState<number>(0);
const [block2, setBlock2] = useState<number>(0);
...
const [block9, setBlock9] = useState<number>(0);
```

After (Stage 8):
```
const [blocks, setBlocks] = useState<number[]>([0, 0, 0, 0, 0, 0, 0, 0, 0]);
```

Nine declarations become one. The type changes from `number` to `number[]` — an array of numbers — and the single starting value `0` becomes an array of nine zeros, one per tile.

**`handlePress`.**

Before (Stage 7):
```
const handlePress = (block: number, setBlock: (value: number) => void) => {
  if (winner !== 0) return;
  if (block !== 0) { return; }
  player1Turn ? setBlock(1) : setBlock(2);
  setPlayer1Turn(!player1Turn);
};
```

After (Stage 8):
```
const handlePress = (index: number) => {
  if (winner !== 0) return;
  if (blocks[index] !== 0) return;

  const updatedBlocks = [...blocks];
  updatedBlocks[index] = player1Turn ? 1 : 2;
  setBlocks(updatedBlocks);

  setPlayer1Turn(!player1Turn);
};
```

Stage 7's version received a value *and* a setter — because there were nine different setters to choose from. Stage 8's version only needs an `index`, because there is now only one setter, `setBlocks`, and the array itself provides random access to any tile via `blocks[index]`. In exchange, the function body has to do one extra thing Stage 7 never had to: build a new array before calling `setBlocks`.

**Calling `handlePress`.**

Before: `onPress={() => handlePress(block1, setBlock1)}` (and so on, with a different value/setter pair per tile)

After: `onPress={() => handlePress(0)}` (and so on, with a different index per tile)

The call site got simpler — a single number instead of a value-and-setter pair — even though, as Stage 9 will show, these nine calls are still written out by hand for now.

### Concept Notes

**Why `[...blocks]` instead of editing `blocks` directly.** It would look, at first glance, perfectly reasonable to write `blocks[index] = player1Turn ? 1 : 2; setBlocks(blocks);` — change the array, then tell React about it. This does not work reliably, and it's worth explaining exactly why. React decides whether to re-render by comparing the *old* state value to the *new* one — and for arrays and objects, that comparison checks whether it's the same array in memory, not whether the contents are different. `blocks[index] = ...` mutates the existing array in place — it's still the exact same array afterward, just with one value changed. When `setBlocks(blocks)` is then called with that same array reference, React can see no difference between "before" and "after," because as far as it can tell, nothing changed. The fix is `const updatedBlocks = [...blocks]` — the spread syntax (`...`) copies every element of `blocks` into a brand new array. `updatedBlocks` and `blocks` hold the same values, but they are two different arrays in memory. Editing `updatedBlocks` and then calling `setBlocks(updatedBlocks)` gives React a genuinely new reference to compare against the old one, and the re-render happens correctly. This is the single most important habit to build around array (and object) state: **never mutate it directly — always build a new copy, change the copy, then set state to the copy.**

**Where this new logic runs.** The whole `handlePress` function, copy-and-all, still only runs when a tile is tapped — nothing about that has changed since Stage 3, Part 2. What's different is what happens *inside* that call: building `updatedBlocks` is new work that happens fresh every single press, discarded once the function finishes. It's a local variable, not state — much like `block1Content` was back in Stage 3, except here it's built, used, and thrown away within a single function call rather than within a single render.

**What hasn't changed yet, on purpose.** The win-check `if` statements still read `blocks[0]`, `blocks[3]`, and so on — eight hand-written checks, structurally identical to Stage 7's, just with array indexing instead of named variables. The board is still nine hand-written `TouchableHighlight` blocks. Both of these are now *array-shaped* problems rather than *nine-separate-variable* problems — and that reshaping is exactly what makes Stage 9 (the board) and Stage 10 (the win check) possible as proper refactors, rather than the kind of change that was blocked by having nine independent variables in the first place.

---

## Stage 9, Part 1

### Introduction

With `blocks` now a single array, the nine hand-written `TouchableHighlight` components can finally be generated instead of typed out one at a time. This part takes the most direct route: a single `.map()` over the whole `blocks` array, producing all nine tiles in one pass. It's a genuine simplification — but it comes with a visible trade-off worth sitting with before fixing it in Part 2.

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

  // All nine tile values now live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>([0, 0, 0, 0, 0, 0, 0, 0, 0]);

  // Decide what a block's tile should display, based on its current value
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
  if (blocks[0] > 0 && blocks[0] === blocks[1] && blocks[1] === blocks[2]) winner = blocks[0];
  if (blocks[3] > 0 && blocks[3] === blocks[4] && blocks[4] === blocks[5]) winner = blocks[3];
  if (blocks[6] > 0 && blocks[6] === blocks[7] && blocks[7] === blocks[8]) winner = blocks[6];
  if (blocks[0] > 0 && blocks[0] === blocks[3] && blocks[3] === blocks[6]) winner = blocks[0];
  if (blocks[1] > 0 && blocks[1] === blocks[4] && blocks[4] === blocks[7]) winner = blocks[1];
  if (blocks[2] > 0 && blocks[2] === blocks[5] && blocks[5] === blocks[8]) winner = blocks[2];
  if (blocks[0] > 0 && blocks[0] === blocks[4] && blocks[4] === blocks[8]) winner = blocks[0];
  if (blocks[2] > 0 && blocks[2] === blocks[4] && blocks[4] === blocks[6]) winner = blocks[2];

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Board layout: one map() over the whole array, instead of
          nine hand-written TouchableHighlight components. Notice
          the board is no longer split into three row Views - that
          structure is addressed in Part 2. */}
      <View style={styles.board}>
        {blocks.map((block, index) => (
          <TouchableHighlight
            key={index}
            style={styles.cell}
            onPress={() => handlePress(index)}
          >
            <Text style={styles.cellText}>{contentFor(block)}</Text>
          </TouchableHighlight>
        ))}
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
  board: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    width: 3 * 84, // 3 cells per row, each cell 80 wide + 2+2 margin
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

### Explanation — Before / After

**The board JSX.**

Before (Stage 8):
```tsx
<View>
  <View style={styles.row}>
    <TouchableHighlight style={styles.cell} onPress={() => handlePress(0)}>
      <Text style={styles.cellText}>{contentFor(blocks[0])}</Text>
    </TouchableHighlight>
    <TouchableHighlight style={styles.cell} onPress={() => handlePress(1)}>
      <Text style={styles.cellText}>{contentFor(blocks[1])}</Text>
    </TouchableHighlight>
    <TouchableHighlight style={styles.cell} onPress={() => handlePress(2)}>
      <Text style={styles.cellText}>{contentFor(blocks[2])}</Text>
    </TouchableHighlight>
  </View>
  {/* ...two more rows, same shape, blocks[3] through blocks[8]... */}
</View>
```

After (Stage 9, Part 1):
```tsx
<View style={styles.board}>
  {blocks.map((block, index) => (
    <TouchableHighlight
      key={index}
      style={styles.cell}
      onPress={() => handlePress(index)}
    >
      <Text style={styles.cellText}>{contentFor(block)}</Text>
    </TouchableHighlight>
  ))}
</View>
```

Nine written-out `TouchableHighlight`s — twenty-seven lines, counting the surrounding row `View`s — become one `.map()` call. The row structure is gone for now; `styles.board` uses `flexWrap: 'wrap'` with a fixed width to force exactly three tiles per line.

### Concept Notes

**Reading `blocks.map((block, index) => ...)`.** `.map()` is called on the `blocks` array, and it runs the function passed to it once for every element in that array — nine times, since `blocks` has nine entries. Each time, it receives two things: `block`, the value at that position (`0`, `1`, or `2`), and `index`, that position's place in the array (`0` through `8`). Whatever the function *returns* on each call — here, one `<TouchableHighlight>` — gets collected into a new array, and that array of nine JSX elements is what actually renders inside `styles.board`. This is the same idea as `.map()` in plain JavaScript, just producing components instead of numbers or strings.

**Why `key={index}` is required, not optional.** Every element produced inside a `.map()` call in React needs a `key` prop — a value unique among its siblings. React uses it behind the scenes to track which rendered element corresponds to which array entry across renders, so it can update efficiently rather than tearing down and rebuilding the whole list every time something changes. `index` works as a key here because the tiles never get reordered or removed — they're always nine fixed positions. Leaving `key` off doesn't break functionality immediately, but React will warn about it, and it's worth treating that warning as a rule, not a suggestion.

**Why `handlePress(index)` and `contentFor(block)` together, and not `contentFor(index)`.** Inside the `.map()` callback, both pieces of information arrive automatically — `block` is "what's stored here" and `index` is "where this is." `contentFor(block)` needs the *value* to decide what symbol to show; `handlePress(index)` needs the *position* to know which array slot to update. Mixing these up — say, passing `index` into `contentFor` — would be a meaningful bug, since `contentFor` has no idea what to do with a position, only a value. Keeping the two separate is a trade worth naming for students: array-based code gives you both "what" and "where" for free on every iteration, but only if the right one is used in the right place.

**The trade-off worth naming before Part 2.** `styles.board` uses `flexWrap: 'wrap'` together with a fixed width calculated to fit exactly three cells per line, forcing the layout to look like a 3×3 grid even though there's no code that actually groups the tiles into rows of three — it's a visual illusion created entirely through styling. It works, and it's a legitimate technique. But it's fragile: change the cell width, the margin, or the screen size, and the "three per row" grouping could silently break, because nothing in the code actually *knows* about rows — only about one flat list of nine tiles. Part 2 addresses this by generating real row structure through code rather than relying on layout math to produce it as a side effect.

---

## Stage 9, Part 2

### Introduction

Part 1 traded explicit rows for a flat `.map()`, relying on `flexWrap` and a calculated width to make nine tiles *look* like three rows of three. This part restores real row structure — but generated with code, not typed out by hand. The array is split into chunks of three, and two `.map()` calls — one for rows, one for the cells within each row — rebuild the exact visual layout Stage 8 had, without giving up the reduction Part 1 achieved.

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

  // All nine tile values now live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>([0, 0, 0, 0, 0, 0, 0, 0, 0]);

  // Decide what a block's tile should display, based on its current value
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
  if (blocks[0] > 0 && blocks[0] === blocks[1] && blocks[1] === blocks[2]) winner = blocks[0];
  if (blocks[3] > 0 && blocks[3] === blocks[4] && blocks[4] === blocks[5]) winner = blocks[3];
  if (blocks[6] > 0 && blocks[6] === blocks[7] && blocks[7] === blocks[8]) winner = blocks[6];
  if (blocks[0] > 0 && blocks[0] === blocks[3] && blocks[3] === blocks[6]) winner = blocks[0];
  if (blocks[1] > 0 && blocks[1] === blocks[4] && blocks[4] === blocks[7]) winner = blocks[1];
  if (blocks[2] > 0 && blocks[2] === blocks[5] && blocks[5] === blocks[8]) winner = blocks[2];
  if (blocks[0] > 0 && blocks[0] === blocks[4] && blocks[4] === blocks[8]) winner = blocks[0];
  if (blocks[2] > 0 && blocks[2] === blocks[4] && blocks[4] === blocks[6]) winner = blocks[2];

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Split the flat 9-element array into 3 rows of 3, so the board
  // can be rendered as real rows again, generated instead of typed out.
  const rows: number[][] = [];
  for (let i = 0; i < blocks.length; i += 3) {
    rows.push(blocks.slice(i, i + 3));
  }

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      <View>
        {rows.map((row, rowIndex) => (
          <View style={styles.row} key={rowIndex}>
            {row.map((block, cellIndex) => {
              const index = rowIndex * 3 + cellIndex;
              return (
                <TouchableHighlight
                  key={index}
                  style={styles.cell}
                  onPress={() => handlePress(index)}
                >
                  <Text style={styles.cellText}>{contentFor(block)}</Text>
                </TouchableHighlight>
              );
            })}
          </View>
        ))}
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

### Explanation — Before / After

**Preparing the data.**

Before (Stage 9, Part 1): no preparation — `blocks.map()` was called directly in the JSX, flat.

After (Stage 9, Part 2):
```ts
const rows: number[][] = [];
for (let i = 0; i < blocks.length; i += 3) {
  rows.push(blocks.slice(i, i + 3));
}
```

A new step appears *before* the JSX is even returned: `blocks` (one array of 9) is turned into `rows` (an array of 3 arrays, each holding 3). This runs directly in `GameScreen`'s body, on every render — exactly like `contentFor` and the win-check `if` statements above it.

**The board JSX.**

Before (Stage 9, Part 1):
```tsx
<View style={styles.board}>
  {blocks.map((block, index) => (
    <TouchableHighlight key={index} style={styles.cell} onPress={() => handlePress(index)}>
      <Text style={styles.cellText}>{contentFor(block)}</Text>
    </TouchableHighlight>
  ))}
</View>
```

After (Stage 9, Part 2):
```tsx
<View>
  {rows.map((row, rowIndex) => (
    <View style={styles.row} key={rowIndex}>
      {row.map((block, cellIndex) => {
        const index = rowIndex * 3 + cellIndex;
        return (
          <TouchableHighlight key={index} style={styles.cell} onPress={() => handlePress(index)}>
            <Text style={styles.cellText}>{contentFor(block)}</Text>
          </TouchableHighlight>
        );
      })}
    </View>
  ))}
</View>
```

One `.map()` becomes two, nested — an outer one over `rows`, an inner one over each row's three cells. `styles.board`'s `flexWrap` trick is gone; `styles.row` (unchanged since Stage 3) is back, now generated instead of hand-placed three times.

### Concept Notes

**What `blocks.slice(i, i + 3)` does, and why the loop steps by 3.** `.slice(start, end)` returns a new array containing the elements from `start` up to, but not including, `end`. The loop runs with `i` at `0`, `3`, then `6` — `i += 3` each time — so `rows.push(blocks.slice(i, i + 3))` pushes `blocks.slice(0, 3)` (positions 0, 1, 2), then `blocks.slice(3, 6)` (positions 3, 4, 5), then `blocks.slice(6, 9)` (positions 6, 7, 8). The result, `rows`, is an array of three arrays — `[[row0], [row1], [row2]]` — which is exactly the shape needed to map over rows and then cells.

**Why the inner map needs to recompute `index`.** Once `blocks` has been split into `rows`, the original position in the flat array isn't directly available inside the inner `.map()` anymore — `cellIndex` only counts `0`, `1`, `2` *within its own row*, starting over for every row. `rowIndex * 3 + cellIndex` recovers the true flat-array position: row `0` contributes offsets `0`, `1`, `2`; row `1` contributes `3`, `4`, `5` (since `1 * 3 = 3`); row `2` contributes `6`, `7`, `8`. This is the one piece of arithmetic in this part worth making sure students can trace by hand — it's the bridge between "where something sits visually" (row and column) and "where it sits in `blocks`" (a single flat index), and `handlePress` still only understands the flat version.

**Why the inner callback needs a full function body, not just an arrow expression.** Every other `.map()` callback used so far — `blocks.map((block, index) => (<TouchableHighlight ...>))` — has been a single expression, implicitly returned. Here, the inner callback has two statements: `const index = rowIndex * 3 + cellIndex;` and then `return (<TouchableHighlight ...>);`. That's why it's written with curly braces, `{ ... }`, and an explicit `return` — a small but important TypeScript/JavaScript rule worth naming directly: an arrow function with a `{ }` body doesn't return anything automatically, unlike the shorthand `=> (...)` form used everywhere else in this stage. Forgetting the `return` here is a very easy mistake to make, and worth demonstrating live if a student runs into a blank board.

**What this part has actually achieved.** The board looks and behaves identically to Stage 8 — same grid, same taps, same results — but every row and every cell is now generated from `blocks`, not typed by hand. Nothing about the *game* changed across either Part of this stage; both were pure display refactors, matching the spirit of Stage 6. What has changed is that the board's visual structure is now something the code actively understands and produces, rather than something achieved as a side effect of CSS wrapping. That distinction — generated structure versus accidental layout — is worth stating directly, since Part 1 worked, but for the wrong reasons.

---

## Stage 10

### Introduction

Since Stage 5, the win check has been eight `if` statements, each one a near-copy of the last. Stage 8 changed the variable names from `block1` to `blocks[0]`, but the shape stayed the same: eight lines that differ only in which three tiles they look at. This stage separates the two things those lines were mixing together. The *data* (which three positions make a winning line) moves into an array of arrays, and the *rule* (are those three tiles matching and played) is written once inside a loop.

Students will meet two new pieces of syntax here: `for...of`, and array destructuring. The destructuring is a close relative of the `const { player1, player2 } = route.params;` line from Stage 2, so it is worth drawing that connection when you reach it in class.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

// The 8 possible winning lines, each listed as three board positions (0-8).
// This data never changes while the app runs, so it lives outside the
// component and is not rebuilt on every render.
const winningLines: number[][] = [
  [0, 1, 2], // top row
  [3, 4, 5], // middle row
  [6, 7, 8], // bottom row
  [0, 3, 6], // left column
  [1, 4, 7], // middle column
  [2, 5, 8], // right column
  [0, 4, 8], // diagonal, top-left to bottom-right
  [2, 4, 6], // diagonal, top-right to bottom-left
];

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

  // All nine tile values now live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>([0, 0, 0, 0, 0, 0, 0, 0, 0]);

  // Decide what a block's tile should display, based on its current value
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

  // Walk through every winning line. Each line is destructured into three
  // positions (a, b, c), and the same single rule is applied to each one:
  // the first tile must be played, and all three tiles must match.
  // winner stays 0 while the game is still in progress or drawn
  let winner: number = 0;
  for (const [a, b, c] of winningLines) {
    if (blocks[a] > 0 && blocks[a] === blocks[b] && blocks[b] === blocks[c]) {
      winner = blocks[a];
    }
  }

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Split the flat 9-element array into 3 rows of 3, so the board
  // can be rendered as real rows again, generated instead of typed out.
  const rows: number[][] = [];
  for (let i = 0; i < blocks.length; i += 3) {
    rows.push(blocks.slice(i, i + 3));
  }

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      <View>
        {rows.map((row, rowIndex) => (
          <View style={styles.row} key={rowIndex}>
            {row.map((block, cellIndex) => {
              const index = rowIndex * 3 + cellIndex;
              return (
                <TouchableHighlight
                  key={index}
                  style={styles.cell}
                  onPress={() => handlePress(index)}
                >
                  <Text style={styles.cellText}>{contentFor(block)}</Text>
                </TouchableHighlight>
              );
            })}
          </View>
        ))}
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

### Explanation: Before / After

**The win check.**

Before (Stage 9, Part 2):
```ts
let winner: number = 0;
if (blocks[0] > 0 && blocks[0] === blocks[1] && blocks[1] === blocks[2]) winner = blocks[0];
if (blocks[3] > 0 && blocks[3] === blocks[4] && blocks[4] === blocks[5]) winner = blocks[3];
if (blocks[6] > 0 && blocks[6] === blocks[7] && blocks[7] === blocks[8]) winner = blocks[6];
if (blocks[0] > 0 && blocks[0] === blocks[3] && blocks[3] === blocks[6]) winner = blocks[0];
if (blocks[1] > 0 && blocks[1] === blocks[4] && blocks[4] === blocks[7]) winner = blocks[1];
if (blocks[2] > 0 && blocks[2] === blocks[5] && blocks[5] === blocks[8]) winner = blocks[2];
if (blocks[0] > 0 && blocks[0] === blocks[4] && blocks[4] === blocks[8]) winner = blocks[0];
if (blocks[2] > 0 && blocks[2] === blocks[4] && blocks[4] === blocks[6]) winner = blocks[2];
```

After (Stage 10):
```ts
const winningLines: number[][] = [
  [0, 1, 2], [3, 4, 5], [6, 7, 8],
  [0, 3, 6], [1, 4, 7], [2, 5, 8],
  [0, 4, 8], [2, 4, 6],
];

let winner: number = 0;
for (const [a, b, c] of winningLines) {
  if (blocks[a] > 0 && blocks[a] === blocks[b] && blocks[b] === blocks[c]) {
    winner = blocks[a];
  }
}
```

Eight lines of repeated logic become two things: a list of the eight lines, and one `if` written once. Read the "Before" version carefully and the pattern is clear. Every line has the same structure, and the only thing that changes from line to line is the three numbers inside the square brackets. Those numbers are now the data in `winningLines`, and the structure is the `if` inside the loop.

**Where the data lives.**

Before: the position numbers were scattered across eight lines, inside the logic.

After: they sit in `winningLines`, declared above `App`, outside every function. They are not state, because they never change, and they do not need to be recreated every time `GameScreen` renders.

### Concept Notes

**Separating data from logic.** This is the main lesson of the stage. In Stage 5, each win check mixed two different things together: *which tiles to look at* and *what to check about them*. The "what to check" part is identical every time: the first tile has been played, and all three tiles match. Only the "which tiles" part varies. Pulling the varying part out into a list leaves the repeated part as a single piece of code that can be written once. A useful question to give students for any repetitive code is: "What actually differs between these copies?" If the answer is a handful of values, those values are probably data, and the rest is probably a loop.

**An array of arrays.** `winningLines` has the type `number[][]`, which reads as "an array of number-arrays." Each inner array holds one winning combination: `[0, 1, 2]` is the top row, `[0, 4, 8]` is one diagonal. This is sometimes called a jagged array, because in general the inner arrays are allowed to be different lengths. Here every inner array happens to hold exactly three numbers, but the structure is the same, and TypeScript does not distinguish between the two cases. It is worth having students tick off the eight lines against the board: three rows, three columns, two diagonals. A mistake in this list produces a silent bug, such as a win that is never detected, so it is worth checking by eye.

**Destructuring with square brackets.** The line `for (const [a, b, c] of winningLines)` combines two ideas. `for...of` walks through an array one element at a time, and here each element is itself a three-number array. `const [a, b, c]` then unpacks that array as it arrives, assigning the first number to `a`, the second to `b`, and the third to `c`. This is the same unpacking idea as `const { player1, player2 } = route.params;` from Stage 2, with one difference. Object destructuring uses curly braces and matches by *name*; array destructuring uses square brackets and matches by *position*. Tracing the first pass of the loop by hand makes this concrete. The first element is `[0, 1, 2]`, so `a` is `0`, `b` is `1` and `c` is `2`. The condition then reads `blocks[0] > 0 && blocks[0] === blocks[1] && blocks[1] === blocks[2]`, which is exactly the first `if` statement from Stage 9, Part 2.

**Where this code runs.** The loop sits directly in the body of `GameScreen`, not inside an arrow function. As with every win check since Stage 5, it therefore runs on every render. After each tap, `handlePress` updates `blocks`, React re-renders `GameScreen`, the loop re-checks all eight lines, and `winner` is recalculated from scratch. Nothing stores the result between renders, and nothing needs to.

**Why `winningLines` sits outside the component.** Anything declared inside `GameScreen` is recreated on every render. For `contentFor` and `handlePress` that is fine, because they close over state and props. `winningLines` depends on nothing. It is the same eight lines whether the board is empty or nearly full. Declaring it at the top of the file, above `App`, says plainly that it is a fixed fact about the game and not part of any one screen's behaviour. This is also the first step towards the file structure planned for Stages 13 and 14, where this constant, together with the win-check logic, will move into its own file.

**What the stage achieves.** The game plays exactly as it did in Stage 9. Winning with any row, column or diagonal still ends the game and shows the winner. What has changed is that the rules of the game are now readable as data: a student can look at `winningLines` and see every way to win without reading any logic. Adding or correcting a line is a one-line edit to the list. Stage 11 builds on this array-based approach by adding the case the game still ignores, a full board with no winner.

---

## Stage 11

### Introduction

Since Stage 5 the game has recognised a win, but it has never recognised a draw. If all nine tiles fill up and nobody has three in a line, the board simply stops responding, and nothing on screen explains why. This stage closes that gap with one new calculated value, `isDraw`, and one new line of JSX to display it.

The interesting part is how little code this takes. Because the board is now a single array, "is the board full?" is a question the array can answer directly. This is the first payoff of the Stage 8 refactor that adds new behaviour rather than just removing repetition.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

// The 8 possible winning lines, each listed as three board positions (0-8).
// This data never changes while the app runs, so it lives outside the
// component and is not rebuilt on every render.
const winningLines: number[][] = [
  [0, 1, 2], // top row
  [3, 4, 5], // middle row
  [6, 7, 8], // bottom row
  [0, 3, 6], // left column
  [1, 4, 7], // middle column
  [2, 5, 8], // right column
  [0, 4, 8], // diagonal, top-left to bottom-right
  [2, 4, 6], // diagonal, top-right to bottom-left
];

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

  // All nine tile values now live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>([0, 0, 0, 0, 0, 0, 0, 0, 0]);

  // Decide what a block's tile should display, based on its current value
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

  // Walk through every winning line. Each line is destructured into three
  // positions (a, b, c), and the same single rule is applied to each one:
  // the first tile must be played, and all three tiles must match.
  // winner stays 0 while the game is still in progress or drawn
  let winner: number = 0;
  for (const [a, b, c] of winningLines) {
    if (blocks[a] > 0 && blocks[a] === blocks[b] && blocks[b] === blocks[c]) {
      winner = blocks[a];
    }
  }

  // A draw means the board is completely full AND nobody has won.
  // The winner check matters: the ninth tile can complete a winning line,
  // and that must be reported as a win, not a draw.
  const isDraw: boolean = winner === 0 && blocks.every((block) => block !== 0);

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Split the flat 9-element array into 3 rows of 3, so the board
  // can be rendered as real rows again, generated instead of typed out.
  const rows: number[][] = [];
  for (let i = 0; i < blocks.length; i += 3) {
    rows.push(blocks.slice(i, i + 3));
  }

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      <View>
        {rows.map((row, rowIndex) => (
          <View style={styles.row} key={rowIndex}>
            {row.map((block, cellIndex) => {
              const index = rowIndex * 3 + cellIndex;
              return (
                <TouchableHighlight
                  key={index}
                  style={styles.cell}
                  onPress={() => handlePress(index)}
                >
                  <Text style={styles.cellText}>{contentFor(block)}</Text>
                </TouchableHighlight>
              );
            })}
          </View>
        ))}
      </View>

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
      {isDraw && <Text style={styles.title}>It is a draw!</Text>}
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

### Explanation: Before / After

**Calculating the draw.**

Before (Stage 10): nothing. The code decided who had won, and had no opinion about a full board.

After (Stage 11):
```ts
const isDraw: boolean = winner === 0 && blocks.every((block) => block !== 0);
```

This one line sits directly after the win-check loop, because it depends on the finished value of `winner`. It reads as two conditions joined by `&&`: nobody has won, and every tile has been played.

**Displaying the draw.**

Before (Stage 10):
```ts
{winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
{winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
```

After (Stage 11):
```ts
{winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
{winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
{isDraw && <Text style={styles.title}>It is a draw!</Text>}
```

A third `&&` line is added beneath the two from Stage 5. It follows the same pattern exactly: when `isDraw` is `true` the `<Text>` renders, and when it is `false` nothing renders.

### Concept Notes

**Reading `blocks.every((block) => block !== 0)`.** `.every()` is an array method, in the same family as the `.map()` from Stage 9. Where `.map()` transforms every element and collects the results, `.every()` asks a yes-or-no question of every element and answers with a single `true` or `false`. It returns `true` only if the question is answered "yes" for all of them. Here the question is "has this tile been played?", written as `block !== 0`. If even one tile is still `0`, `.every()` stops looking and returns `false`. If all nine are `1` or `2`, it returns `true`. A sibling method, `.some()`, is worth mentioning for later: it returns `true` if the answer is "yes" for at least one element. Together, `.every()` and `.some()` replace a lot of hand-written loops.

**Why `winner === 0` is part of the draw condition.** Consider the final move of a game. The ninth tile is played, the board is now full, and that move happens to complete a diagonal. A full board is not enough to call this a draw, because someone has won. Without the `winner === 0` check, both "Wins!" and "It is a draw!" would appear at once. This is a good example of an edge case that only shows up when asked "what if the last move is also the winning move?". It is worth asking students to find the problem themselves before revealing the answer.

**A calculated value, not state.** `isDraw` follows the same rule as `winner` and `rows`: it is declared directly in the body of `GameScreen`, so it is recalculated from `blocks` on every render. There is no `useState` for it and no `setIsDraw` anywhere. The board changes, the screen re-renders, and `isDraw` is simply worked out again. This is the Stage 3, Part 4 lesson about state versus calculated values applied to something new. A useful rule for students: if a value can always be worked out from existing state, it should be calculated, not stored. Storing it creates a second copy that has to be kept in sync by hand, and that is where bugs come from.

**Why a draw ends the game without any new guard code.** When the board is full, every tile is non-zero, so the `if (blocks[index] !== 0) return;` guard from Stage 7 already blocks every further tap. No new code was needed to stop play. That guard was written to prevent overwriting a tile, and it happens to cover this case as well.

---

## Stage 12

### Introduction

A finished game currently has nowhere to go. After a win or a draw, the only way to play again is to close and reopen the app. This stage adds a Restart Game button that returns the board to its starting position.

The stage is short, but it asks a useful question: what exactly does it mean to reset a screen? The answer depends on which values are state and which are calculated, and that is the distinction the project has been building since Stage 3. The stage also introduces a small constant, `emptyBoard`, so that the starting board is written down in only one place.

### Code

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

// The 8 possible winning lines, each listed as three board positions (0-8).
// This data never changes while the app runs, so it lives outside the
// component and is not rebuilt on every render.
const winningLines: number[][] = [
  [0, 1, 2], // top row
  [3, 4, 5], // middle row
  [6, 7, 8], // bottom row
  [0, 3, 6], // left column
  [1, 4, 7], // middle column
  [2, 5, 8], // right column
  [0, 4, 8], // diagonal, top-left to bottom-right
  [2, 4, 6], // diagonal, top-right to bottom-left
];

// The board at the start of a game: nine tiles, none played yet.
// Used for the initial state AND for restarting, so the starting
// position is defined in exactly one place.
const emptyBoard: number[] = [0, 0, 0, 0, 0, 0, 0, 0, 0];

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

  // All nine tile values live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>(emptyBoard);

  // Decide what a block's tile should display, based on its current value
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

  // Walk through every winning line. Each line is destructured into three
  // positions (a, b, c), and the same single rule is applied to each one:
  // the first tile must be played, and all three tiles must match.
  // winner stays 0 while the game is still in progress or drawn
  let winner: number = 0;
  for (const [a, b, c] of winningLines) {
    if (blocks[a] > 0 && blocks[a] === blocks[b] && blocks[b] === blocks[c]) {
      winner = blocks[a];
    }
  }

  // A draw means the board is completely full AND nobody has won.
  // The winner check matters: the ninth tile can complete a winning line,
  // and that must be reported as a win, not a draw.
  const isDraw: boolean = winner === 0 && blocks.every((block) => block !== 0);

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Restart: put every piece of GAME state back to its starting value.
  // winner and isDraw are not listed here because they are calculated
  // from blocks on every render - they reset automatically.
  const resetGame = () => {
    setBlocks(emptyBoard);
    setPlayer1Turn(true);
  };

  // Split the flat 9-element array into 3 rows of 3, so the board
  // can be rendered as real rows again, generated instead of typed out.
  const rows: number[][] = [];
  for (let i = 0; i < blocks.length; i += 3) {
    rows.push(blocks.slice(i, i + 3));
  }

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      <View>
        {rows.map((row, rowIndex) => (
          <View style={styles.row} key={rowIndex}>
            {row.map((block, cellIndex) => {
              const index = rowIndex * 3 + cellIndex;
              return (
                <TouchableHighlight
                  key={index}
                  style={styles.cell}
                  onPress={() => handlePress(index)}
                >
                  <Text style={styles.cellText}>{contentFor(block)}</Text>
                </TouchableHighlight>
              );
            })}
          </View>
        ))}
      </View>

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
      {isDraw && <Text style={styles.title}>It is a draw!</Text>}

      <View style={styles.restart}>
        <Button title="Restart Game" onPress={resetGame} />
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
  restart: {
    marginTop: 16,
  },
});
```

### Explanation: Before / After

**The starting board.**

Before (Stage 11):
```ts
const [blocks, setBlocks] = useState<number[]>([0, 0, 0, 0, 0, 0, 0, 0, 0]);
```

After (Stage 12):
```ts
const emptyBoard: number[] = [0, 0, 0, 0, 0, 0, 0, 0, 0];
// ...and inside GameScreen:
const [blocks, setBlocks] = useState<number[]>(emptyBoard);
```

The array of nine zeros moves out of `useState` and into a named constant above `App`, next to `winningLines`. Restarting needs the same starting value that the game began with, and this way it only has to be written once.

**The reset function.**

Before (Stage 11): no such function existed.

After (Stage 12):
```ts
const resetGame = () => {
  setBlocks(emptyBoard);
  setPlayer1Turn(true);
};
```

Two state updates, one for each piece of game state, each returning to its starting value.

**The button.**

Before (Stage 11): the JSX ended after the three result lines.

After (Stage 12):
```tsx
<View style={styles.restart}>
  <Button title="Restart Game" onPress={resetGame} />
</View>
```

A `Button`, the same component used on `PlayerScreen` since Stage 1, wrapped in a `View` so that the new `styles.restart` can give it some space above.

### Concept Notes

**What a reset actually has to reset.** The question to put to students is: "Which values on this screen need to go back to where they started?" Looking through `GameScreen`, there are only two pieces of state: `blocks` and `player1Turn`. Both are set back to their starting values, and that is the whole reset. What matters just as much is what is absent. `winner` and `isDraw` are not reset anywhere, because neither is state. Both are calculated from `blocks` on every render, so once `blocks` is empty again, the next render works out `winner` as `0` and `isDraw` as `false` without any instruction to do so. Had they been stored with `useState`, the reset function would need two more lines, and forgetting them would leave a stale "Player 1 Wins!" on an empty board. This is the clearest demonstration in the project so far of why calculated values are better than stored ones.

**The player names do not need resetting either.** `player1` and `player2` are not state in `GameScreen`. They arrive through `route.params`, as set up in Stage 2, Part 2, and nothing on this screen changes them. A restart therefore keeps the same two players, which is the behaviour most people expect from a "play again" button. Letting the players enter new names is a different feature: it means leaving `GameScreen` and returning to `PlayerScreen`, and is better left for a future version.

**`onPress={resetGame}` versus `onPress={() => resetGame()}`.** Every `onPress` in the project until now has used an arrow function wrapper, such as `onPress={() => handlePress(index)}`. This one passes `resetGame` directly, with no parentheses. The difference is that `handlePress(index)` needs an argument, so it has to be wrapped to supply one when the tap happens. `resetGame` takes no arguments, so the function itself can be handed to the button, and the button calls it when pressed. The version to warn students about is `onPress={resetGame()}`. With the parentheses, `resetGame` would be called immediately while the screen renders, rather than when the button is pressed, and the screen would fall into a loop of resetting and re-rendering.

**Where this code runs.** `resetGame` is defined in `GameScreen`'s body on every render, but its two lines only execute when the button is pressed. This is the same distinction that has run through the project since the first `onPress` in Stage 1. Calling two state setters in a single handler is also worth noting: React groups them together and re-renders once with both changes applied, so the board never appears on screen in a half-reset state.

**Why sharing `emptyBoard` is safe.** A careful student may ask a pointed question: if `useState` and `resetGame` both use the same `emptyBoard` array, could editing the board accidentally edit `emptyBoard` itself? The answer is no, and the reason is Stage 8. `handlePress` never edits `blocks` in place. It copies the array with `[...blocks]`, changes the copy, and sets state to the copy. The original `emptyBoard` is never touched, so it is always still nine zeros when the next restart needs it. The habit of never mutating state is what makes this shared constant safe.

**What these two stages complete.** With a draw and a restart in place, the game itself is finished. Every outcome is handled (win, draw, or play on), and a finished game can start again. Both open threads from the closing note of Version 1 are now closed. Everything from Stage 13 onward changes nothing about how the game behaves. It is about where the code lives, starting with the folder structure that `winningLines` and `emptyBoard` are now ready to move into.

---

## Stage 13

### Introduction

Everything in the project still lives in one file, `App.tsx`: the navigator, both screens, the styles, the winning lines and every piece of game logic. That worked while the project was small, but the file keeps growing, and every stage from here on would make it harder to find anything in it.

This stage begins the move to a proper project structure. Students create a `src/` folder with five sub-folders, then move the first pieces of code into it: the two fixed values the game depends on. Nothing about the game's behaviour changes. This stage is about where code lives, not what it does, so the app should look and play exactly as it did at the end of Stage 12.

### Code

**The new folder structure.** Create a `src` folder at the root of the project, beside `App.tsx`, with these five sub-folders inside it:

```
my-xo-game/
├── App.tsx
├── app.json
├── package.json
└── src/
    ├── components/
    ├── constants/
    │   └── gameConstants.ts
    ├── navigation/
    ├── screens/
    └── utils/
```

Only `constants/` has a file in it at this stage. The other four folders are created now so students can see the full plan, and they are filled in over the stages that follow.

**New file: `src/constants/gameConstants.ts`**

```ts
// The 8 possible winning lines, each listed as three board positions (0-8).
// This data never changes while the app runs, so it lives outside any
// component and is not rebuilt on every render.
export const winningLines: number[][] = [
  [0, 1, 2], // top row
  [3, 4, 5], // middle row
  [6, 7, 8], // bottom row
  [0, 3, 6], // left column
  [1, 4, 7], // middle column
  [2, 5, 8], // right column
  [0, 4, 8], // diagonal, top-left to bottom-right
  [2, 4, 6], // diagonal, top-right to bottom-left
];

// The board at the start of a game: nine tiles, none played yet.
// Used for the initial state AND for restarting, so the starting
// position is defined in exactly one place.
export const emptyBoard: number[] = [0, 0, 0, 0, 0, 0, 0, 0, 0];
```

**Updated file: `App.tsx`**

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

// The fixed game values now live in their own file
import { winningLines, emptyBoard } from './src/constants/gameConstants';

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

  // All nine tile values live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>(emptyBoard);

  // Decide what a block's tile should display, based on its current value
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

  // Walk through every winning line. Each line is destructured into three
  // positions (a, b, c), and the same single rule is applied to each one:
  // the first tile must be played, and all three tiles must match.
  // winner stays 0 while the game is still in progress or drawn
  let winner: number = 0;
  for (const [a, b, c] of winningLines) {
    if (blocks[a] > 0 && blocks[a] === blocks[b] && blocks[b] === blocks[c]) {
      winner = blocks[a];
    }
  }

  // A draw means the board is completely full AND nobody has won.
  // The winner check matters: the ninth tile can complete a winning line,
  // and that must be reported as a win, not a draw.
  const isDraw: boolean = winner === 0 && blocks.every((block) => block !== 0);

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Restart: put every piece of GAME state back to its starting value.
  // winner and isDraw are not listed here because they are calculated
  // from blocks on every render - they reset automatically.
  const resetGame = () => {
    setBlocks(emptyBoard);
    setPlayer1Turn(true);
  };

  // Split the flat 9-element array into 3 rows of 3, so the board
  // can be rendered as real rows again, generated instead of typed out.
  const rows: number[][] = [];
  for (let i = 0; i < blocks.length; i += 3) {
    rows.push(blocks.slice(i, i + 3));
  }

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      <View>
        {rows.map((row, rowIndex) => (
          <View style={styles.row} key={rowIndex}>
            {row.map((block, cellIndex) => {
              const index = rowIndex * 3 + cellIndex;
              return (
                <TouchableHighlight
                  key={index}
                  style={styles.cell}
                  onPress={() => handlePress(index)}
                >
                  <Text style={styles.cellText}>{contentFor(block)}</Text>
                </TouchableHighlight>
              );
            })}
          </View>
        ))}
      </View>

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
      {isDraw && <Text style={styles.title}>It is a draw!</Text>}

      <View style={styles.restart}>
        <Button title="Restart Game" onPress={resetGame} />
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
  restart: {
    marginTop: 16,
  },
});
```

### Explanation: Before / After

**Where the constants are declared.**

Before (Stage 12), inside `App.tsx`:
```ts
const winningLines: number[][] = [
  [0, 1, 2], // top row
  ...
];

const emptyBoard: number[] = [0, 0, 0, 0, 0, 0, 0, 0, 0];
```

After (Stage 13), inside `src/constants/gameConstants.ts`:
```ts
export const winningLines: number[][] = [
  [0, 1, 2], // top row
  ...
];

export const emptyBoard: number[] = [0, 0, 0, 0, 0, 0, 0, 0, 0];
```

The declarations are identical apart from one new word at the start of each, `export`. The comments travel with the code.

**How `App.tsx` finds them.**

Before (Stage 12): no import was needed, because the constants were declared in the same file.

After (Stage 13):
```tsx
import { winningLines, emptyBoard } from './src/constants/gameConstants';
```

This single line sits with the other imports at the top of `App.tsx`. Everything else in the file, including `GameScreen`, is untouched, because `winningLines` and `emptyBoard` are used under the same names as before.

### Concept Notes

**Why a project needs structure.** A single file is easy to start with, but it is hard to live in. At this point `App.tsx` holds the navigator, two screens, nine styles and every piece of game logic. Finding the win check means scrolling past everything else, and changing one thing risks disturbing another. Splitting code into files lets each file have one clear job, so a student, or you returning to this project months later, can open the file that matches the question being asked. If the question is "what are the winning lines?", the answer is in `gameConstants.ts`, and nothing else in that file competes for attention.

**What each folder is for.** Five folders are created now, and each one will receive a specific kind of code over the stages ahead:

- `constants/` holds fixed values that never change while the app runs, such as `winningLines` and `emptyBoard`.
- `utils/` holds plain functions that do some work and have nothing to do with the screen, such as the win check. This is the next stage.
- `screens/` holds the full-screen views that the navigator shows. `PlayerScreen` and `GameScreen` will move here in Stage 15.
- `navigation/` holds the navigator's setup and its types, arriving in Stage 16.
- `components/` holds small reusable pieces of interface. The `Cell` component arrives in Stage 17.

The names are a widely used convention rather than a rule of React Native, which is worth saying to students directly. Other projects organise things differently. What matters is that the structure is consistent and that a newcomer can guess where something lives.

**`export` and `import`: the two halves of sharing code.** By default, everything declared in a file is private to that file. Putting `export` in front of a declaration makes it available to other files. Putting `import { name } from 'path'` in another file brings it in. The two always come as a pair, and the name inside the curly braces must match the exported name exactly. This is called a *named* export. It differs from `export default function App()`, which every project has used since Stage 0. A file can have only one default export, and it is imported without curly braces. A file can have as many named exports as it needs, and each is imported by name inside curly braces.

**Reading the path `'./src/constants/gameConstants'`.** The `./` at the start means "starting from the folder this file is in". `App.tsx` sits at the project root, so the path goes into `src`, then into `constants`, then to the file `gameConstants`. The file extension is left off. Later stages will also use `../`, which means "go up one folder", and it is worth explaining both now so they are familiar by then.

**What stays the same.** Run the app and play a game, and nothing looks different. The data is the same and the code that uses it is the same. Only the address of that data has changed. This is a point worth making to students: a refactor that goes well is invisible to the person using the app, and a quick play-through at the end is the simplest way to confirm nothing broke.

**A note on empty folders and Git.** Students were asked to commit after every stage as proof of their work. Git does not track empty folders, only files, so `components/`, `navigation/`, `screens/` and `utils/` will not appear in this stage's commit. That is expected, and they will appear naturally as soon as each folder holds a file. If a student worries that their commit looks incomplete, this is the explanation.

---

## Stage 14

### Introduction

Stage 13 moved the game's fixed *data* out of `App.tsx`. This stage moves the game's *rules*: the code that works out who has won, whether the board is full, what symbol a tile should show, and how to arrange nine tiles into three rows. All four jobs go into a new file, `src/utils/gameLogic.ts`.

What makes this possible is that none of these jobs needs anything from React. Each takes a board in and gives an answer back. Code that behaves like that is easy to move, which is why this is the first piece of logic to leave `App.tsx`. `handlePress` and `resetGame` stay where they are, and the Concept Notes explain why.

### Code

**New file: `src/utils/gameLogic.ts`**

```ts
import { winningLines } from '../constants/gameConstants';

// Decide what a block's tile should display, based on its current value
// 0 - not yet played | 1 - Player 1 (X) | 2 - Player 2 (O)
export const contentFor = (block: number): string => {
  switch (block) {
    case 1:
      return 'X';
    case 2:
      return 'O';
    default:
      return '';
  }
};

// Walk through every winning line. Each line is destructured into three
// positions (a, b, c), and the same single rule is applied to each one:
// the first tile must be played, and all three tiles must match.
// Returns 1 or 2 for the winning player, or 0 if nobody has won.
export const checkWinner = (blocks: number[]): number => {
  let winner: number = 0;
  for (const [a, b, c] of winningLines) {
    if (blocks[a] > 0 && blocks[a] === blocks[b] && blocks[b] === blocks[c]) {
      winner = blocks[a];
    }
  }
  return winner;
};

// True when every tile has been played.
// Whether that means a draw also depends on there being no winner,
// so the caller combines this with checkWinner.
export const isBoardFull = (blocks: number[]): boolean => {
  return blocks.every((block) => block !== 0);
};

// Split the flat 9-element array into 3 rows of 3, so the board
// can be rendered as real rows, generated instead of typed out.
export const toRows = (blocks: number[]): number[][] => {
  const rows: number[][] = [];
  for (let i = 0; i < blocks.length; i += 3) {
    rows.push(blocks.slice(i, i + 3));
  }
  return rows;
};
```

**Updated file: `App.tsx`**

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

// Fixed game values
import { emptyBoard } from './src/constants/gameConstants';
// Game rules: plain functions that take a board and return an answer
import { checkWinner, contentFor, isBoardFull, toRows } from './src/utils/gameLogic';

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

  // All nine tile values live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>(emptyBoard);

  // Values worked out from blocks on every render.
  // The rules themselves now live in gameLogic.ts.
  const winner: number = checkWinner(blocks);
  const isDraw: boolean = winner === 0 && isBoardFull(blocks);
  const rows: number[][] = toRows(blocks);

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Restart: put every piece of GAME state back to its starting value.
  // winner and isDraw are not listed here because they are calculated
  // from blocks on every render - they reset automatically.
  const resetGame = () => {
    setBlocks(emptyBoard);
    setPlayer1Turn(true);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      <View>
        {rows.map((row, rowIndex) => (
          <View style={styles.row} key={rowIndex}>
            {row.map((block, cellIndex) => {
              const index = rowIndex * 3 + cellIndex;
              return (
                <TouchableHighlight
                  key={index}
                  style={styles.cell}
                  onPress={() => handlePress(index)}
                >
                  <Text style={styles.cellText}>{contentFor(block)}</Text>
                </TouchableHighlight>
              );
            })}
          </View>
        ))}
      </View>

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
      {isDraw && <Text style={styles.title}>It is a draw!</Text>}

      <View style={styles.restart}>
        <Button title="Restart Game" onPress={resetGame} />
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
  restart: {
    marginTop: 16,
  },
});
```

### Explanation: Before / After

**The win check.**

Before (Stage 13), inside `GameScreen`:
```ts
let winner: number = 0;
for (const [a, b, c] of winningLines) {
  if (blocks[a] > 0 && blocks[a] === blocks[b] && blocks[b] === blocks[c]) {
    winner = blocks[a];
  }
}
```

After (Stage 14), inside `GameScreen`:
```ts
const winner: number = checkWinner(blocks);
```

The loop has moved into `checkWinner` in `gameLogic.ts`, unchanged except that it now ends with `return winner;`. Back in `GameScreen`, the six-line block becomes a single call. Notice that `let` has become `const`: `winner` is now assigned once, with its final value, and never reassigned.

**The draw, the rows and the symbols.**

Before (Stage 13), inside `GameScreen`:
```ts
const isDraw: boolean = winner === 0 && blocks.every((block) => block !== 0);

const rows: number[][] = [];
for (let i = 0; i < blocks.length; i += 3) {
  rows.push(blocks.slice(i, i + 3));
}

const contentFor = (block: number): string => {
  switch (block) { ... }
};
```

After (Stage 14), inside `GameScreen`:
```ts
const isDraw: boolean = winner === 0 && isBoardFull(blocks);
const rows: number[][] = toRows(blocks);
```

`contentFor` is no longer defined in `GameScreen` at all. The JSX still calls `contentFor(block)` exactly as before, but the function now arrives through an import.

**The imports.**

After (Stage 14), at the top of `App.tsx`:
```tsx
import { emptyBoard } from './src/constants/gameConstants';
import { checkWinner, contentFor, isBoardFull, toRows } from './src/utils/gameLogic';
```

`winningLines` is no longer imported here. `App.tsx` does not use it any more, because only `checkWinner` needs it, so `gameLogic.ts` imports it instead.

### Concept Notes

**Pure functions: the idea behind this stage.** All four functions share one important property: they use only what they are given. `checkWinner` receives a board and returns a number. `isBoardFull` receives a board and returns `true` or `false`. Given the same board, each one always gives the same answer, and none of them reads state, calls a setter, or touches the screen. Functions like this are called *pure*. They are the easiest kind of code to move, to reuse and to check, because they depend on nothing outside themselves. To test `checkWinner`, you hand it an array and look at what comes back. No app needs to be running.

**Why `handlePress` and `resetGame` stay in `GameScreen`.** A good question to put to students is: "Why didn't these two move as well?" They are the opposite of pure. `handlePress` reads `blocks`, `winner` and `player1Turn`, and it calls `setBlocks` and `setPlayer1Turn`. `resetGame` calls two setters. Both are tied to this screen's state, and state can only be changed from inside the component that owns it. Moving them out would mean passing in every value and every setter, and that makes the code harder to follow. The dividing line between the two groups is simple: code that *calculates* can move out, and code that *changes state* stays in the component that owns it.

**Why the file ends in `.ts` and not `.tsx`.** `gameLogic.ts` contains no JSX, which is the HTML-like markup used inside components. The `.tsx` extension is for files that do contain JSX, and `.ts` is for plain TypeScript. Using the right one signals at a glance what a file holds: a `.tsx` file builds part of the screen, and a `.ts` file does not.

**Where this code runs now.** `checkWinner(blocks)`, `isBoardFull(blocks)` and `toRows(blocks)` are called directly in `GameScreen`'s body, so they still run on every render, exactly as the loop, the `.every()` and the `rows` code did before. The behaviour is the same, and only the place where the logic is written has changed. There is one small difference underneath: the functions in `gameLogic.ts` are created once, when the file is first loaded. The old `contentFor` was created again on every render, because it was defined inside `GameScreen`. The saving is tiny, but it follows from the same idea as `winningLines` sitting outside the component in Stage 10: anything that does not depend on the screen's state has no reason to be rebuilt along with the screen.

**Dependencies run in one direction.** Look at the two import lines in `gameLogic.ts` and `App.tsx`. `gameLogic.ts` imports from `constants/`, and `App.tsx` imports from both. Nothing in `constants/` imports from `utils/`. This one-way flow is deliberate: the more basic a folder is, the less it should depend on anything else. If `constants/` ever imported from `utils/` while `utils/` imported from `constants/`, the two files would depend on each other in a circle, which causes problems and is a habit worth avoiding from the start.

**The `../` in action.** The path in `gameLogic.ts` is `'../constants/gameConstants'`. The file sits in `src/utils/`, so `../` goes up one level to `src/`, and the rest of the path goes down into `constants/`. This is the first time the project has needed it, and it is worth tracing on the folder tree from Stage 13 with a finger.

**What `App.tsx` has become.** Compare the file with Stage 12. The winning lines, the starting board, the symbol lookup, the win check, the draw check and the row-building have all gone, and what remains is almost entirely navigation, state, event handlers and layout. `GameScreen` now reads as a description of what the screen *does*, with the details of how the game's rules work left to the file that owns them. The next step is the screens themselves: Stage 15 moves `PlayerScreen` and `GameScreen` out of `App.tsx` into `src/screens/`, leaving `App.tsx` with only the navigator.

---

## Stage 15, Part 1

### Introduction

`App.tsx` still holds the navigator and both screens. This part moves the simpler screen, `PlayerScreen`, into its own file in `src/screens/`. `GameScreen` stays in `App.tsx` for now.

Students will see three things happen together: the component moves, its styles move with it, and `App.tsx` imports it back. The game behaves exactly as it did at the end of Stage 14.

### Code

**The folder structure after this part.**

```
my-xo-game/
├── App.tsx
├── app.json
├── package.json
└── src/
    ├── components/
    ├── constants/
    │   └── gameConstants.ts
    ├── navigation/
    ├── screens/
    │   └── PlayerScreen.tsx
    └── utils/
        └── gameLogic.ts
```

**New file: `src/screens/PlayerScreen.tsx`**

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';

export default function PlayerScreen({ navigation }: any) {
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

**Updated file: `App.tsx`**

```tsx
import { useState } from 'react';
import { View, Text, Button, StyleSheet, TouchableHighlight } from 'react-native';

// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

// Fixed game values
import { emptyBoard } from './src/constants/gameConstants';
// Game rules: plain functions that take a board and return an answer
import { checkWinner, contentFor, isBoardFull, toRows } from './src/utils/gameLogic';
// The player setup screen now lives in its own file
import PlayerScreen from './src/screens/PlayerScreen';

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

function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // All nine tile values live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>(emptyBoard);

  // Values worked out from blocks on every render.
  // The rules themselves live in gameLogic.ts.
  const winner: number = checkWinner(blocks);
  const isDraw: boolean = winner === 0 && isBoardFull(blocks);
  const rows: number[][] = toRows(blocks);

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Restart: put every piece of GAME state back to its starting value.
  // winner and isDraw are not listed here because they are calculated
  // from blocks on every render - they reset automatically.
  const resetGame = () => {
    setBlocks(emptyBoard);
    setPlayer1Turn(true);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      <View>
        {rows.map((row, rowIndex) => (
          <View style={styles.row} key={rowIndex}>
            {row.map((block, cellIndex) => {
              const index = rowIndex * 3 + cellIndex;
              return (
                <TouchableHighlight
                  key={index}
                  style={styles.cell}
                  onPress={() => handlePress(index)}
                >
                  <Text style={styles.cellText}>{contentFor(block)}</Text>
                </TouchableHighlight>
              );
            })}
          </View>
        ))}
      </View>

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
      {isDraw && <Text style={styles.title}>It is a draw!</Text>}

      <View style={styles.restart}>
        <Button title="Restart Game" onPress={resetGame} />
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
  restart: {
    marginTop: 16,
  },
});
```

### Explanation: Before / After

**The component itself.**

Before (Stage 14), inside `App.tsx`:
```ts
function PlayerScreen({ navigation }: any){
  const [player1, setPlayer1] = useState<string>('');
  ...
}
```

After (Stage 15, Part 1), inside `src/screens/PlayerScreen.tsx`:
```ts
export default function PlayerScreen({ navigation }: any) {
  const [player1, setPlayer1] = useState<string>('');
  ...
}
```

The body of the function is identical. The only change to the declaration is the two words added at the start, `export default`, which make it available to other files.

**The imports.**

Before (Stage 14), top of `App.tsx`:
```tsx
import { View, Text, TextInput, Button, StyleSheet, TouchableHighlight } from 'react-native';
```

After (Stage 15, Part 1), top of `App.tsx`:
```tsx
import { View, Text, Button, StyleSheet, TouchableHighlight } from 'react-native';
import PlayerScreen from './src/screens/PlayerScreen';
```

`TextInput` has been removed from `App.tsx`, because only `PlayerScreen` uses it and `PlayerScreen` has left. The new line brings the screen back in. Note that there are no curly braces around `PlayerScreen`.

**The styles.**

Before (Stage 14), one `StyleSheet` in `App.tsx` with `container`, `title`, `input`, `row`, `cell`, `cellText` and `restart`.

After (Stage 15, Part 1): `PlayerScreen.tsx` has its own `StyleSheet` with `container`, `title` and `input`. The `StyleSheet` in `App.tsx` loses `input` and keeps the other six.

**What `Stack.Screen` sees.**

```tsx
<Stack.Screen name="Home" component={PlayerScreen} />
```

This line is unchanged. It receives the same component under the same name. It neither knows nor cares that the function now comes from another file.

### Concept Notes

**Default exports versus named exports.** Stage 13 introduced named exports, such as `export const emptyBoard`, which are imported with curly braces: `import { emptyBoard } from '...'`. This part uses the other kind. A file may have one *default* export, and it is imported without curly braces and under any name the importer chooses: `import PlayerScreen from '...'`. React Native components are conventionally default exports, one per file, and `App` has been one since Stage 0. A simple rule to give students: curly braces mean "pick this item out by name", and no curly braces means "give me the file's main thing". Putting braces around a default import is one of the most common mistakes at this stage, so it is worth demonstrating on purpose.

**The file name matches the component name.** The file is `PlayerScreen.tsx` and the function inside is `PlayerScreen`. Component names and their files start with a capital letter, which is called PascalCase. Matching the two means a student looking at an import line knows exactly which file to open. The extension is `.tsx` because the file contains JSX.

**Imports belong to each file.** Moving the screen out shows something that is easy to miss: every file declares for itself what it needs. `PlayerScreen.tsx` imports `useState`, `TextInput` and the rest, because it uses them. `App.tsx` no longer needs `TextInput`, so the import is removed. Leaving unused imports in place would not break the app, but it would mislead anyone reading the file. After moving code, a good habit is to check the import lines at the top of both the old and the new file.

**Styles travel with the component that uses them.** `PlayerScreen` took `container`, `title` and `input` with it, and `input` is gone from `App.tsx` because nothing there uses it. This keeps a component's appearance in the same file as its structure. It also leaves a visible duplication. `container` and `title` are now defined in both files, with identical contents. This is a deliberate trade-off at this stage, and it is worth naming rather than hiding. Two files each owning a copy is simple to follow, and the cost is that a future change to the title size would need to be made twice. Stage 17 reorganises the styles again when the board's tiles become a component of their own.

**Behaviour is unchanged.** Run the app and enter two names. The navigation, the passing of `player1` and `player2`, and the whole game behave as before. As in Stage 13, a refactor that goes well should be invisible to the person using the app, and a quick play-through is the simplest confirmation.

**Where this leaves `App.tsx`.** It now holds the navigator, `GameScreen` and one `StyleSheet`. Part 2 finishes the job by moving `GameScreen` out as well.

---

## Stage 15, Part 2

### Introduction

`GameScreen` is the last piece left in `App.tsx` besides the navigator. This part moves it into `src/screens/GameScreen.tsx`, using exactly the technique from Part 1: add `export default`, move the styles with it, and import the screen back.

Once that is done, `App.tsx` contains nothing but the navigator. That is the whole point of the stage. The file that starts the app now describes only the shape of the app: which screens exist and which one comes first.

### Code

**The folder structure after this part.**

```
my-xo-game/
├── App.tsx
├── app.json
├── package.json
└── src/
    ├── components/
    ├── constants/
    │   └── gameConstants.ts
    ├── navigation/
    ├── screens/
    │   ├── GameScreen.tsx
    │   └── PlayerScreen.tsx
    └── utils/
        └── gameLogic.ts
```

**New file: `src/screens/GameScreen.tsx`**

```tsx
import { useState } from 'react';
import { View, Text, Button, StyleSheet, TouchableHighlight } from 'react-native';

// Fixed game values
import { emptyBoard } from '../constants/gameConstants';
// Game rules: plain functions that take a board and return an answer
import { checkWinner, contentFor, isBoardFull, toRows } from '../utils/gameLogic';

export default function GameScreen({ navigation, route} : any){
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // All nine tile values live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>(emptyBoard);

  // Values worked out from blocks on every render.
  // The rules themselves live in gameLogic.ts.
  const winner: number = checkWinner(blocks);
  const isDraw: boolean = winner === 0 && isBoardFull(blocks);
  const rows: number[][] = toRows(blocks);

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Restart: put every piece of GAME state back to its starting value.
  // winner and isDraw are not listed here because they are calculated
  // from blocks on every render - they reset automatically.
  const resetGame = () => {
    setBlocks(emptyBoard);
    setPlayer1Turn(true);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      <View>
        {rows.map((row, rowIndex) => (
          <View style={styles.row} key={rowIndex}>
            {row.map((block, cellIndex) => {
              const index = rowIndex * 3 + cellIndex;
              return (
                <TouchableHighlight
                  key={index}
                  style={styles.cell}
                  onPress={() => handlePress(index)}
                >
                  <Text style={styles.cellText}>{contentFor(block)}</Text>
                </TouchableHighlight>
              );
            })}
          </View>
        ))}
      </View>

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
      {isDraw && <Text style={styles.title}>It is a draw!</Text>}

      <View style={styles.restart}>
        <Button title="Restart Game" onPress={resetGame} />
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
  restart: {
    marginTop: 16,
  },
});
```

**Updated file: `App.tsx`**

```tsx
// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

// Each screen now lives in its own file
import PlayerScreen from './src/screens/PlayerScreen';
import GameScreen from './src/screens/GameScreen';

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
```

### Explanation: Before / After

**`App.tsx` as a whole.**

Before (Stage 15, Part 1), `App.tsx` held imports from React, React Native, the navigation libraries, the constants and the game logic. It also held the `App` function, all of `GameScreen`, and a `StyleSheet` with six styles. It was roughly 100 lines long.

After (Stage 15, Part 2), `App.tsx` holds four imports, one constant and one function. It is about 17 lines long, and the only thing it does is declare the navigator.

**The imports of `App.tsx`.**

Before (Stage 15, Part 1):
```tsx
import { useState } from 'react';
import { View, Text, Button, StyleSheet, TouchableHighlight } from 'react-native';
import { emptyBoard } from './src/constants/gameConstants';
import { checkWinner, contentFor, isBoardFull, toRows } from './src/utils/gameLogic';
```

After (Stage 15, Part 2): all four of those lines are gone. `App.tsx` no longer uses `useState`, any React Native component, any constant or any game logic function, so none of them need importing there.

**The same imports, now in `GameScreen.tsx`.**

```tsx
import { emptyBoard } from '../constants/gameConstants';
import { checkWinner, contentFor, isBoardFull, toRows } from '../utils/gameLogic';
```

The imports moved with the code that uses them, and the paths changed from `./src/...` to `../...`. `GameScreen.tsx` sits inside `src/screens/`, so it goes up one level, out of `screens/` and into `src/`, and then down into `constants/` or `utils/`.

**The declaration.**

Before: `function GameScreen({ navigation, route} : any){`

After: `export default function GameScreen({ navigation, route} : any){`

As with `PlayerScreen`, the body of the function is untouched.

### Concept Notes

**What `App.tsx` is for now.** Compare the final `App.tsx` with the one from Stage 0. It is back to being small, but its job has changed. In Stage 0 it was the whole app. Now it is the *entry point*: the file that assembles the pieces. It states which screens exist (`Home` and `Game`), what each is called, which component each shows and which one opens first. It contains no game logic, no state and no styling. This is a useful way to think about a project's top file. It should read like a table of contents, with the real content living in the files it points to.

**Paths from the new location.** `GameScreen.tsx` is two folders deep: `src/` and then `screens/`. Every import in the file has to be written relative to *that* position, which is why `./src/constants/gameConstants` became `../constants/gameConstants`. Wrong relative paths are the most common error when moving files, and the error message, "Unable to resolve module", is worth showing students once so they recognise it. The fix is always the same. Start from the folder the file is in, and count the `../` steps up to the common parent.

**Dependencies still flow in one direction.** Stage 14 noted that dependencies should run one way. The structure is now clear enough to state in full:

```
App.tsx  →  screens/  →  utils/  →  constants/
```

`App.tsx` imports screens. Screens import utils and constants. Utils import constants. Nothing points backwards: no screen imports `App.tsx`, and no constant imports a screen. A project that keeps to this rule stays easy to follow, because a student can always trace a piece of code downwards to where it comes from.

**A common mistake worth demonstrating.** Forgetting the word `default` when exporting a screen gives a confusing error in the app: "Element type is invalid", mentioning that the component is `undefined`. The cause is that `App.tsx` imports `GameScreen` without braces, which expects a default export, and finds none. If a student reports this error after copying the code, check the first line of the screen file first.

**Why `navigation` is still declared and still unused in `GameScreen`.** The parameter list is still `{ navigation, route } : any`, exactly as it has been since Stage 2. `GameScreen` receives `navigation` but never uses it. It was included in Stage 2 for completeness, and nothing has needed it since. It has been left alone deliberately, because the whole parameter list is about to be replaced. The `: any` is a gap in TypeScript's checking: it tells the compiler to accept anything, so a mistake such as asking for `route.params.player3` would not be caught. Stage 16 closes the gap with proper types for the navigator, so that `navigation` and `route` are described instead of left as `any`.

**What this stage achieves.** The project now has a real structure. Looking at the folder tree, a newcomer could guess where to find the winning lines, the win check, each screen and the navigator without opening a single file. The game itself is unchanged. It is the same board, the same rules and the same restart button, in about the same number of lines, now spread across files that each have a single job.

---

Yes, that works. This response covers Stages 16 and 17 and the closing note for the whole project. Each stage is split into two Parts, so there are four working checkpoints for students to commit, and the closing note comes after Stage 17, Part 2.

---

## Stage 16, Part 1

### Introduction

Since Stage 2, both screens have declared their props as `: any`. That tells TypeScript to stop checking: if a student mistypes a screen name or forgets to pass `player2`, nothing complains until the app misbehaves. This stage replaces `any` with real types, in two steps.

This part writes down the map of the navigator: which screens exist and what each one expects to receive. It also hands that map to the navigator. The screens themselves are not touched until Part 2. This is also the first file in `src/navigation/`.

### Code

**The folder structure after this part.**

```
my-xo-game/
├── App.tsx
├── app.json
├── package.json
└── src/
    ├── components/
    ├── constants/
    │   └── gameConstants.ts
    ├── navigation/
    │   └── types.ts
    ├── screens/
    │   ├── GameScreen.tsx
    │   └── PlayerScreen.tsx
    └── utils/
        └── gameLogic.ts
```

**New file: `src/navigation/types.ts`**

```ts
import type { NativeStackScreenProps } from '@react-navigation/native-stack';

// A map of every screen in the navigator to the params that screen expects.
// 'undefined' means the screen is opened without any params.
export type RootStackParamList = {
  Home: undefined;
  Game: { player1: string; player2: string };
};

// The full set of props React Navigation passes to each screen
// (navigation and route), typed for that specific screen.
export type HomeScreenProps = NativeStackScreenProps<RootStackParamList, 'Home'>;
export type GameScreenProps = NativeStackScreenProps<RootStackParamList, 'Game'>;
```

**Updated file: `App.tsx`**

```tsx
// NavigationContainer wraps the whole app and manages navigation state
import { NavigationContainer } from '@react-navigation/native';
// createNativeStackNavigator builds a stack-based navigator using native transitions
import { createNativeStackNavigator } from '@react-navigation/native-stack';

// The map of screens and their params
import type { RootStackParamList } from './src/navigation/types';

// Each screen lives in its own file
import PlayerScreen from './src/screens/PlayerScreen';
import GameScreen from './src/screens/GameScreen';

// The navigator now knows exactly which screens exist and what they expect
const Stack = createNativeStackNavigator<RootStackParamList>();

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
```

`PlayerScreen.tsx`, `GameScreen.tsx`, `gameConstants.ts` and `gameLogic.ts` are unchanged in this part.

### Explanation: Before / After

**The navigator.**

Before (Stage 15, Part 2):
```ts
const Stack = createNativeStackNavigator();
```

After (Stage 16, Part 1):
```ts
const Stack = createNativeStackNavigator<RootStackParamList>();
```

The only change is the `<RootStackParamList>` between the function name and its brackets. From this line on, TypeScript knows that the only valid screen names are `Home` and `Game`.

**The map itself.**

```ts
export type RootStackParamList = {
  Home: undefined;
  Game: { player1: string; player2: string };
};
```

Each line is a screen name followed by what that screen receives. `Home` receives nothing, so it is `undefined`. `Game` receives an object with two strings, which are exactly the two values that `navigation.navigate('Game', {...})` has been passing since Stage 2, Part 2.

**What the compiler now catches.** Try each of these in the editor:

```tsx
<Stack.Screen name="Hom" component={PlayerScreen} />
```

This now shows an error, because `"Hom"` is not a key of `RootStackParamList`. Before this stage, a typo like this would only show up as a broken screen at runtime.

### Concept Notes

**What `any` was really doing.** Every `{ navigation }: any` in the project was a way of telling TypeScript "trust me". It was a reasonable shortcut in Stage 2, when navigation was new and the types were a distraction. But `any` switches checking off completely, so the code was still plain JavaScript in practice. This stage is where the project starts using the one thing TypeScript is for: catching mistakes before the app runs.

**A `ParamList` is a map.** `RootStackParamList` is a plain list of screen names, each paired with the shape of the data that screen is given. It is the single place that says what each screen needs. When a new screen is added later, it gets one new line here, and everything that touches it is checked against that line. The word "Root" refers to this being the top-level navigator. Larger apps nest navigators inside navigators, and each level has its own param list.

**Angle brackets pass a type into a function.** Students have written this pattern since Stage 1, in `useState<string>('')` and `useState<number[]>(emptyBoard)`. The angle brackets hand a type to a function so that it knows what it is working with. `createNativeStackNavigator<RootStackParamList>()` works the same way. Showing the three side by side helps students see that this is one idea used in three places, not a new piece of syntax.

**`import type`.** The type-only import tells TypeScript, and the bundler, that this import exists purely for checking. Types are erased when the app is built, so nothing from `types.ts` exists at runtime. This is important to say out loud: this stage changes **nothing** about how the app behaves. It runs exactly as it did in Stage 15. The whole stage lives in the checking, not the running.

**The route name and the component name are different things.** The screen registered as `name="Home"` is displayed by a component called `PlayerScreen`. They have differed since Stage 2. The names in `RootStackParamList` must match the `name` values, not the component names. It is a good question to put to students: "Why is it `Home` in the map, not `PlayerScreen`?"

**Seeing the errors.** Expo does not stop the app from running when there is a type error. The errors show as red underlines in the editor, and running `npx tsc --noEmit` in the terminal lists them all. Students who do not see any red underlines can use that command to confirm the checking is working.

**What is still `any`.** Nothing in the screens has changed yet, so both still declare `: any`. TypeScript is now checking the navigator, but not yet the screens. Part 2 closes that gap.

---

## Stage 16, Part 2

### Introduction

The map exists, and the navigator uses it. This part makes the two screens use it as well. `PlayerScreen` and `GameScreen` each swap `: any` for the typed props from `types.ts`.

The change is small in lines but large in what it protects. `navigation.navigate('Game', {...})` is now checked against the `Game` entry in the map, and `route.params` is known to hold two strings.

### Code

**Updated file: `src/screens/PlayerScreen.tsx`**

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';

// The typed props for this specific screen
import type { HomeScreenProps } from '../navigation/types';

export default function PlayerScreen({ navigation }: HomeScreenProps) {
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

**Updated file: `src/screens/GameScreen.tsx`**

```tsx
import { useState } from 'react';
import { View, Text, Button, StyleSheet, TouchableHighlight } from 'react-native';

// Fixed game values
import { emptyBoard } from '../constants/gameConstants';
// Game rules: plain functions that take a board and return an answer
import { checkWinner, contentFor, isBoardFull, toRows } from '../utils/gameLogic';
// The typed props for this specific screen
import type { GameScreenProps } from '../navigation/types';

export default function GameScreen({ route }: GameScreenProps) {
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // All nine tile values live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>(emptyBoard);

  // Values worked out from blocks on every render.
  // The rules themselves live in gameLogic.ts.
  const winner: number = checkWinner(blocks);
  const isDraw: boolean = winner === 0 && isBoardFull(blocks);
  const rows: number[][] = toRows(blocks);

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Restart: put every piece of GAME state back to its starting value.
  // winner and isDraw are not listed here because they are calculated
  // from blocks on every render - they reset automatically.
  const resetGame = () => {
    setBlocks(emptyBoard);
    setPlayer1Turn(true);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      <View>
        {rows.map((row, rowIndex) => (
          <View style={styles.row} key={rowIndex}>
            {row.map((block, cellIndex) => {
              const index = rowIndex * 3 + cellIndex;
              return (
                <TouchableHighlight
                  key={index}
                  style={styles.cell}
                  onPress={() => handlePress(index)}
                >
                  <Text style={styles.cellText}>{contentFor(block)}</Text>
                </TouchableHighlight>
              );
            })}
          </View>
        ))}
      </View>

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
      {isDraw && <Text style={styles.title}>It is a draw!</Text>}

      <View style={styles.restart}>
        <Button title="Restart Game" onPress={resetGame} />
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
  restart: {
    marginTop: 16,
  },
});
```

`App.tsx`, `types.ts`, `gameConstants.ts` and `gameLogic.ts` are unchanged in this part.

### Explanation: Before / After

**The two signatures.**

Before (Stage 16, Part 1):
```ts
export default function PlayerScreen({ navigation }: any) {
export default function GameScreen({ navigation, route} : any){
```

After (Stage 16, Part 2):
```ts
export default function PlayerScreen({ navigation }: HomeScreenProps) {
export default function GameScreen({ route }: GameScreenProps) {
```

`any` is replaced with the specific type for each screen. `GameScreen` also stops declaring `navigation`. It has never used it, and with a proper type in place there is no reason to keep an unused parameter.

**What the compiler now knows.**

Inside `PlayerScreen`:
```tsx
onPress={() => navigation.navigate('Game', {player1: player1, player2: player2})}
```

This line is unchanged, but it is now checked. TypeScript compares it against `Game: { player1: string; player2: string }`. These two edits, tried temporarily in the editor, each produce an error:

```tsx
navigation.navigate('Game', {player1: player1})
```
Error: `player2` is missing.

```tsx
navigation.navigate('Gam', {player1: player1, player2: player2})
```
Error: `"Gam"` is not a valid screen name.

Inside `GameScreen`:
```ts
const { player1, player2 } = route.params;
```

`player1` and `player2` are now known to be strings. Before this stage, TypeScript saw `any` and accepted anything.

### Concept Notes

**Props are a typed contract.** Students met `ProfileCard` at the very start of this series, where `type Props` described what the component needed: an image, a name and a description. `HomeScreenProps` and `GameScreenProps` do the same job for screens. Each one is a written agreement about what the screen receives. The difference is that React Navigation supplies these props, instead of a parent passing them by hand, so the types come from a helper, `NativeStackScreenProps`, rather than being written out one field at a time.

**One type per screen, not one for all.** `PlayerScreen` receives `HomeScreenProps` and `GameScreen` receives `GameScreenProps`. They are separate because each screen has different route params. `Home` takes none, and `Game` takes two strings. Giving a single type to both would reintroduce the ambiguity that this stage removes.

**Destructuring only what is needed.** `GameScreen` now reads `{ route }` and nothing else. Destructuring in the function signature has been used since the `ProfileCard` example in the first document, and the parameter list is a good place to see what a component actually depends on. A screen that asks only for `route` is saying it reads data and never navigates.

**Where the checking happens.** Nothing in this part changes how the app runs. Types are erased before the code runs, so a student who runs the app will see no difference. The benefit arrives while the code is being written, in the red underlines and in `npx tsc --noEmit`, which is why it is valuable to show a mistake on screen rather than only describe it. A good in-class moment is to remove `player2` from the `navigate` call, show the error, and then put it back.

**The pattern for any new screen.** Adding a screen to this app now follows three steps: add a line to `RootStackParamList`, add a `...ScreenProps` type below it, and register the screen with `<Stack.Screen>`. The compiler then guides everything else. That is a repeatable recipe, and it is worth giving students in those terms.

---

## Stage 17, Part 1

### Introduction

The project opened with a `ProfileCard`, a small component that could be reused with different props. This stage finally brings that idea into the game. The `TouchableHighlight` and `Text` pair that draws a single tile is pulled out of `GameScreen` into its own component, `Cell`, with an interface that describes exactly what a `Cell` needs.

It is the first file in `src/components/`, and it is the stage where the project's folder structure is fully in use.

### Code

**The folder structure after this part.**

```
my-xo-game/
├── App.tsx
├── app.json
├── package.json
└── src/
    ├── components/
    │   └── Cell.tsx
    ├── constants/
    │   └── gameConstants.ts
    ├── navigation/
    │   └── types.ts
    ├── screens/
    │   ├── GameScreen.tsx
    │   └── PlayerScreen.tsx
    └── utils/
        └── gameLogic.ts
```

**New file: `src/components/Cell.tsx`**

```tsx
import { TouchableHighlight, Text, StyleSheet } from 'react-native';

import { contentFor } from '../utils/gameLogic';

// Describes what a Cell needs from whoever uses it
interface CellProps {
  value: number;       // 0 - not played | 1 - Player 1 | 2 - Player 2
  onPress: () => void; // what to do when the tile is tapped
}

// One tile on the board. It owns how a tile looks,
// but knows nothing about the game's rules or whose turn it is.
export default function Cell({ value, onPress }: CellProps) {
  return (
    <TouchableHighlight style={styles.cell} onPress={onPress}>
      <Text style={styles.cellText}>{contentFor(value)}</Text>
    </TouchableHighlight>
  );
}

const styles = StyleSheet.create({
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

**Updated file: `src/screens/GameScreen.tsx`**

```tsx
import { useState } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';

// Fixed game values
import { emptyBoard } from '../constants/gameConstants';
// Game rules: plain functions that take a board and return an answer
import { checkWinner, isBoardFull, toRows } from '../utils/gameLogic';
// The typed props for this specific screen
import type { GameScreenProps } from '../navigation/types';
// One tile of the board
import Cell from '../components/Cell';

export default function GameScreen({ route }: GameScreenProps) {
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // All nine tile values live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>(emptyBoard);

  // Values worked out from blocks on every render.
  // The rules themselves live in gameLogic.ts.
  const winner: number = checkWinner(blocks);
  const isDraw: boolean = winner === 0 && isBoardFull(blocks);
  const rows: number[][] = toRows(blocks);

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Restart: put every piece of GAME state back to its starting value.
  // winner and isDraw are not listed here because they are calculated
  // from blocks on every render - they reset automatically.
  const resetGame = () => {
    setBlocks(emptyBoard);
    setPlayer1Turn(true);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      <View>
        {rows.map((row, rowIndex) => (
          <View style={styles.row} key={rowIndex}>
            {row.map((block, cellIndex) => {
              const index = rowIndex * 3 + cellIndex;
              return (
                <Cell
                  key={index}
                  value={block}
                  onPress={() => handlePress(index)}
                />
              );
            })}
          </View>
        ))}
      </View>

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
      {isDraw && <Text style={styles.title}>It is a draw!</Text>}

      <View style={styles.restart}>
        <Button title="Restart Game" onPress={resetGame} />
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
  row: {
    flexDirection: 'row',
  },
  restart: {
    marginTop: 16,
  },
});
```

`App.tsx`, `PlayerScreen.tsx`, `types.ts`, `gameConstants.ts` and `gameLogic.ts` are unchanged in this part.

### Explanation: Before / After

**The tile in the JSX.**

Before (Stage 16, Part 2):
```tsx
<TouchableHighlight
  key={index}
  style={styles.cell}
  onPress={() => handlePress(index)}
>
  <Text style={styles.cellText}>{contentFor(block)}</Text>
</TouchableHighlight>
```

After (Stage 17, Part 1):
```tsx
<Cell
  key={index}
  value={block}
  onPress={() => handlePress(index)}
/>
```

Five lines become one element, and it reads in plain language: a cell, with a value, that does something when pressed. The `TouchableHighlight`, the `Text` and the call to `contentFor` have all moved into `Cell`.

**What a `Cell` needs.**

```tsx
interface CellProps {
  value: number;
  onPress: () => void;
}
```

This is the component's contract. It must be given a number and a function. The function takes no arguments and returns nothing, which is what `() => void` means.

**The styles.**

Before (Stage 16, Part 2): `GameScreen` had `container`, `title`, `row`, `cell`, `cellText` and `restart`.

After (Stage 17, Part 1): `GameScreen` keeps `container`, `title`, `row` and `restart`. `cell` and `cellText` have moved to `Cell.tsx`, along with the component that uses them.

**The imports of `GameScreen`.** `TouchableHighlight` is gone from the React Native import, and `contentFor` is gone from the `gameLogic` import. `GameScreen` no longer uses either. `Cell` is imported in their place.

### Concept Notes

**The `ProfileCard` idea, completed.** In the first document, `ProfileCard` was introduced as "your own Lego block". It had props, destructuring in the signature, and a promise that it would one day move to its own file. `Cell` is that promise kept. It has the same three ingredients: a typed description of what it needs, destructuring in the function signature, and a file of its own. A student who followed the first document will recognise the pattern. A student who is new to it can look back at the `ProfileCard` stage to see where it began.

**`interface` and `type`.** The profile-card document used `type Props`. This stage uses `interface CellProps`, and `types.ts` used `type` for the navigation maps. Both describe the shape of an object and, for props, they are interchangeable. A reasonable rule for students is that `interface` is the common choice for the props of a component, and `type` is used when giving a name to something more computed, as in `NativeStackScreenProps<...>`. It matters less which one is picked than that a project picks one approach and uses it consistently.

**A function passed as a prop.** `onPress` is a function handed from parent to child, and `Cell` calls it when the tile is tapped. This is the same idea as Stage 7, where `handlePress` received a state setter as a parameter. In both cases, a function is treated as an ordinary value that can be passed around. A naming convention helps here: props that carry an event handler start with `on`, as in `onPress`, and the function in the parent is the one that actually decides what happens.

**`Cell` has no state and no rules.** Look at what `Cell` does not do. It does not know whose turn it is. It does not know whether the game is over. It has no `useState`. It receives a value, shows the matching symbol, and reports a tap. It is purely about how one tile *looks*. All of the thinking stays in `GameScreen`, which owns the state. A component like this is sometimes called a presentational component, and keeping the two roles apart makes both easier to understand and to change.

**Why `key` stays on the outside.** `key={index}` is still written where `Cell` is used, inside the `.map()`, and it is not part of `CellProps`. React uses `key` itself, behind the scenes, to track each element in a list. It is never passed down as a prop, so `Cell` cannot read it. This is a common point of confusion, and it is worth pointing out directly.

**Where this code runs.** `Cell` is a function, so React calls it once for every tile each time `GameScreen` renders, nine times in all. Inside it, `contentFor(value)` runs in the body, and the work is the same as before. The `onPress` function is created in `GameScreen`'s `.map()` and is only called when a tile is tapped. This is the same render-versus-tap distinction as every stage since Stage 1.

**The duplicated `container` and `title` styles.** Stage 15, Part 1 noted that two files each held a copy of `container` and `title`. That remains true, and the `cell` styles show the other side of the trade: they now live in exactly one place, next to the only component that uses them. Duplication and sharing both have a cost. The right choice depends on whether the duplicated code is likely to change together.

---

## Stage 17, Part 2

### Introduction

`Cell` took over one tile. `GameScreen` still builds the whole board itself, with the row-splitting, the double `.map()` and the arithmetic that works out each tile's position. This part moves all of that into a second component, `Board`.

After this part, `GameScreen` no longer knows how a board is laid out. It hands over the nine values and a function to call when a tile is tapped, and `Board` does the rest.

### Code

**The folder structure after this part.**

```
my-xo-game/
├── App.tsx
├── app.json
├── package.json
└── src/
    ├── components/
    │   ├── Board.tsx
    │   └── Cell.tsx
    ├── constants/
    │   └── gameConstants.ts
    ├── navigation/
    │   └── types.ts
    ├── screens/
    │   ├── GameScreen.tsx
    │   └── PlayerScreen.tsx
    └── utils/
        └── gameLogic.ts
```

**New file: `src/components/Board.tsx`**

```tsx
import { View, StyleSheet } from 'react-native';

import Cell from './Cell';
import { toRows } from '../utils/gameLogic';

// Describes what a Board needs from whoever uses it
interface BoardProps {
  blocks: number[];                    // the nine tile values, in order
  onCellPress: (index: number) => void; // told which tile (0-8) was tapped
}

// The 3 x 3 grid. It owns the layout of the board, but knows nothing
// about whose turn it is or who has won.
export default function Board({ blocks, onCellPress }: BoardProps) {
  // Split the flat 9-element array into 3 rows of 3
  const rows: number[][] = toRows(blocks);

  return (
    <View>
      {/* Outer map walks the 3 rows; inner map walks the 3 cells
          in each row. rowIndex * 3 + cellIndex recovers the tile's
          original position (0-8) in the flat blocks array. */}
      {rows.map((row, rowIndex) => (
        <View style={styles.row} key={rowIndex}>
          {row.map((block, cellIndex) => {
            const index = rowIndex * 3 + cellIndex;
            return (
              <Cell
                key={index}
                value={block}
                onPress={() => onCellPress(index)}
              />
            );
          })}
        </View>
      ))}
    </View>
  );
}

const styles = StyleSheet.create({
  row: {
    flexDirection: 'row',
  },
});
```

**Updated file: `src/screens/GameScreen.tsx`**

```tsx
import { useState } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';

// Fixed game values
import { emptyBoard } from '../constants/gameConstants';
// Game rules: plain functions that take a board and return an answer
import { checkWinner, isBoardFull } from '../utils/gameLogic';
// The typed props for this specific screen
import type { GameScreenProps } from '../navigation/types';
// The 3 x 3 grid of tiles
import Board from '../components/Board';

export default function GameScreen({ route }: GameScreenProps) {
  const { player1, player2 } = route.params; 
  const [player1Turn, setPlayer1Turn] = useState<boolean>(true);

  // All nine tile values live in a single array.
  // Index 0-8 maps left-to-right, top-to-bottom across the board.
  // 0 - not yet played | 1 - Player 1 played here | 2 - Player 2 played here
  const [blocks, setBlocks] = useState<number[]>(emptyBoard);

  // Values worked out from blocks on every render.
  // The rules themselves live in gameLogic.ts.
  const winner: number = checkWinner(blocks);
  const isDraw: boolean = winner === 0 && isBoardFull(blocks);

  // handlePress only needs the tile's index - the array holds
  // everything else. A new array is built with the one changed value,
  // then handed to setBlocks as a whole. The array is never edited
  // in place.
  const handlePress = (index: number) => {
    if (winner !== 0) return; // stop game after winner is chosen
    if (blocks[index] !== 0) return; // tile already played

    const updatedBlocks = [...blocks]; // copy the existing array
    updatedBlocks[index] = player1Turn ? 1 : 2; // change just this one position
    setBlocks(updatedBlocks); // replace the whole array in state

    setPlayer1Turn(!player1Turn);
  };

  // Restart: put every piece of GAME state back to its starting value.
  // winner and isDraw are not listed here because they are calculated
  // from blocks on every render - they reset automatically.
  const resetGame = () => {
    setBlocks(emptyBoard);
    setPlayer1Turn(true);
  };

  return(
    <View style={styles.container}>
      <Text style={styles.title}>{player1} vs. {player2}</Text>

      <Text>
        {player1Turn ? `${player1}'s Turn` : `${player2}'s Turn`}
      </Text>

      {/* The board is told what to show and who to tell when a tile is tapped */}
      <Board blocks={blocks} onCellPress={handlePress} />

      {winner === 1 && <Text style={styles.title}>{player1} Wins!</Text>}
      {winner === 2 && <Text style={styles.title}>{player2} Wins!</Text>}
      {isDraw && <Text style={styles.title}>It is a draw!</Text>}

      <View style={styles.restart}>
        <Button title="Restart Game" onPress={resetGame} />
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
  restart: {
    marginTop: 16,
  },
});
```

`App.tsx`, `PlayerScreen.tsx`, `Cell.tsx`, `types.ts`, `gameConstants.ts` and `gameLogic.ts` are unchanged in this part.

### Explanation: Before / After

**The board in `GameScreen`.**

Before (Stage 17, Part 1):
```tsx
const rows: number[][] = toRows(blocks);
...
<View>
  {rows.map((row, rowIndex) => (
    <View style={styles.row} key={rowIndex}>
      {row.map((block, cellIndex) => {
        const index = rowIndex * 3 + cellIndex;
        return (
          <Cell key={index} value={block} onPress={() => handlePress(index)} />
        );
      })}
    </View>
  ))}
</View>
```

After (Stage 17, Part 2):
```tsx
<Board blocks={blocks} onCellPress={handlePress} />
```

Around fifteen lines of layout become a single line. The `rows` variable, the nested `.map()` calls and the arithmetic have all moved into `Board.tsx`, in the same form as before. The explanatory comment moved with them.

**What a `Board` needs.**

```tsx
interface BoardProps {
  blocks: number[];
  onCellPress: (index: number) => void;
}
```

A `Board` is given the nine values and a function that takes a tile's position. This is a different shape from `Cell`'s `onPress`, which takes nothing. The `Board` bridges the two: it receives `onCellPress(index)` from above, and gives each `Cell` an `onPress` that calls it with that cell's index.

**What `GameScreen` no longer needs.** `toRows` is gone from its imports, the `row` style is gone from its styles, and the `rows` variable is gone from its body. `GameScreen` has stopped knowing that a board has rows at all.

### Concept Notes

**Data flows down, events flow up.** This is the pattern at the centre of how React components work together, and it is now visible in three levels of the project. `GameScreen` owns `blocks` and passes it **down** to `Board`. `Board` passes each value **down** to a `Cell`. When a tile is tapped, the news travels back **up**: `Cell` calls `onPress`, which calls `Board`'s `onCellPress`, which is `GameScreen`'s `handlePress`. The state is changed in exactly one place, at the top, and every level below simply displays what it has been given. A useful exercise is to have students trace a single tap from the screen to the state change and back, saying out loud which file they are in at each step.

**One job per file, now at every level.** Each file in the project can now be described in a sentence. `Cell` draws one tile. `Board` arranges nine tiles into a grid. `GameScreen` owns the game's state and decides what each tap does. `gameLogic.ts` answers questions about a board. `gameConstants.ts` holds the fixed facts. `types.ts` describes the shape of the navigator. `App.tsx` assembles the screens. If a bug appears, that list tells a student which file to open first.

**Why `Board` has no state.** Like `Cell`, `Board` has no `useState`. It is given `blocks` and displays it. If the board kept its own copy, there would be two sources of truth, and the Stage 3 lesson about derived values warns about exactly that: any second copy has to be kept in sync by hand, and that is where bugs come from.

**Where the layout knowledge lives.** The idea that "a board has three columns" now sits in `Board`, which is where a reader would look for it. There is one honest loose end, which is worth raising with students: the number three appears in two files. `toRows` in `gameLogic.ts` steps through the array in threes, and `Board` multiplies by 3 to work out the position of a tile. If the board size ever changed, both would need updating together. A shared constant, such as `BOARD_SIZE` in `gameConstants.ts`, would fix it. It has been left out of this version deliberately, so that it can serve as a ready-made exercise for students who finish early, or as a candidate for a later version.

**Naming the props.** `onCellPress` is named for what happened, a cell was pressed, and not for what the parent will do with it. `Board` has no idea that the parent will update a game. It only reports that a tile at some position was tapped. This keeps `Board` reusable: another game with a grid could use it with an entirely different `onCellPress`.

**Where this code runs.** `Board` is called once per render of `GameScreen`, and it in turn creates the nine `Cell`s. `toRows(blocks)` runs inside `Board`'s body, so it still runs on every render, as it has since Stage 9. The only difference is which file the work happens in. The game plays exactly as it did in Stage 12.

**What `GameScreen` has become.** Read the return statement from top to bottom: a title, the current turn, the board, the result text and a restart button. The file reads almost like a description of the screen. That is the real payoff of the structural stages: a student can understand `GameScreen` without having to understand how a grid is drawn or how a win is detected.

---

## Closing: Stages 0 through 17

The project began with an empty `View` and ends with seven small files that each do one job. In between, the game never stopped working for long. Every stage was a step that students could run, check and commit. That is the habit this series was built to teach, and it is worth saying plainly to students: the finished game matters less than the way it was reached, one understandable change at a time.

### The whole journey

| Stage | What changed | The idea |
|---|---|---|
| 0 | An empty screen | The shape of a React Native file |
| 1 | Two name inputs | State and controlled inputs |
| 2 | Two screens and navigation | Components, props and passing data between screens |
| 3 | The board, turns, and tile values | Building up display and update logic, on purpose, in halves |
| 4 | Every tile playable | Using the index to route to the right setter |
| 5 | Win detection | Computed values, and `&&` as a conditional component |
| 6 | One `contentFor` function | Refactoring display logic once behaviour is trusted |
| 7 | One `handlePress` function | Refactoring update logic, and passing a setter as a parameter |
| 8 | One `blocks` array | Array state, and never mutating it directly |
| 9 | `.map()` over the board | Generating structure instead of typing it |
| 10 | A list of winning lines | Separating data from logic; destructuring arrays |
| 11 | Draw detection | Calculated values again, and `.every()` |
| 12 | Restart | What a reset has to reset, and what it does not |
| 13 | `src/` and the constants | Folder structure, `export` and `import` |
| 14 | `gameLogic.ts` | Pure functions, and what can safely move |
| 15 | Screens in their own files | Default exports, and `App.tsx` as an entry point |
| 16 | `RootStackParamList` and typed screens | Replacing `any` with a contract |
| 17 | `Cell` and `Board` components | Reusable components, data down and events up |

### The threads that ran through it

**State versus calculated values.** From Stage 3 onward, the project kept a firm line between what is remembered (`blocks`, `player1Turn`) and what is worked out fresh on every render (the symbols, `winner`, `isDraw`, `rows`). Nothing on screen was ever set directly. Stage 12 gave the clearest proof: restarting the game needed two lines because everything else reset itself. A value that can always be worked out from existing state should be calculated, and not stored.

**Duplication before abstraction.** Stages 3 and 4 wrote the same shape of code nine times, and Stages 6 and 7 collapsed it. The same rhythm repeated at a larger scale: the nine tiles of Stage 3 became `.map()` in Stage 9, and the eight win checks of Stage 5 became a loop in Stage 10. In each case the repetition came first, and the simplification came only once the behaviour could be trusted. The deliberate tedium was the teaching tool, because a shortcut offered too early would not have shown why it mattered.

**Two halves of every component.** The code that decides what to show and the code that records what happened were kept visibly apart from Stage 3, Part 4 onward. The distinction between code that runs on every render and code that runs only when called, such as `handlePress` or `resetGame`, was drawn from the first `onPress` in Stage 1. It never stopped mattering, and Stage 14 turned it into a rule: code that calculates can move out, and code that changes state stays with the component that owns it.

**Flat to structured.** Nine variables became one array. One file became many. Nine hand-written tiles became a component, and then a board made of components. Each step gave the code a shape that matched the problem, and each made the next question easier to answer: where does the winner get decided, and where does a tile get drawn?

**Untyped to typed.** `{ navigation }: any` stood for fifteen stages as an honest shortcut. Stage 16 replaced it with a written contract, and Stage 17 gave components their own interfaces. This thread started back in the `ProfileCard` example that opened the series, with `type Props`, and came full circle with `CellProps` and `BoardProps`.

**Data apart from logic, and one-way dependencies.** The winning lines became a list, the rule became a loop, and then both moved to files that matched their job. The folder structure settled into a direction: `App.tsx` imports screens, screens import components and utils, and utils import constants. Nothing points backwards. A project that keeps to this rule stays easy to trace.

### Where the project stands

```
my-xo-game/
├── App.tsx                       the navigator, and nothing else
└── src/
    ├── components/
    │   ├── Board.tsx             the 3 x 3 grid
    │   └── Cell.tsx              one tile
    ├── constants/
    │   └── gameConstants.ts      winningLines, emptyBoard
    ├── navigation/
    │   └── types.ts              RootStackParamList and screen props
    ├── screens/
    │   ├── GameScreen.tsx        game state and what a tap does
    │   └── PlayerScreen.tsx      the setup form
    └── utils/
        └── gameLogic.ts          contentFor, checkWinner, isBoardFull, toRows
```

### For the students

If you have reached this point with a commit at every stage, you have a record of how a program grows, and it is more valuable than the final code. Go back to your Stage 3 commit and compare it with Stage 17. The game is the same, but the code is something you can now read and change with confidence. Two loose ends from the notes are yours to try: a `BOARD_SIZE` constant to replace the repeated number three, and a way to go back to the setup screen and enter new names.
