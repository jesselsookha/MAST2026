# 04 Tap Game

## About 

The project is a tap game. A button is pressed as many times as possible within a fixed time window. When the window closes, the score is compared against the top five stored on the device. The stages move from a static screen, through state and side effects, into persistent storage and list rendering, and end with a correction to a subtle bug that appears only once scores are being saved.

---

## Stage 0 — Empty Template

### What this stage introduces

This is the smallest possible screen. It renders a single `View` with a single `Text` child, and defines a stylesheet with one entry. Nothing is interactive. The purpose of the stage is to establish the file's shape — imports at the top, a function component, a stylesheet, a default export — so that every later stage can be read as a modification of this skeleton.

### The code

```tsx
import { useState } from 'react'; 
import { View, Text, StyleSheet } from 'react-native';  

function App() {
  return(
    <View style={styles.container}>
      <Text>Tap Game</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fefefe', 
    padding: 20, 
    color: '#333',  
  },
});

export default App; 
```

### Walkthrough

The file begins with two imports. `useState` is imported from `react` but is not yet used — it is there because the next stage will need it, and because leaving it in place makes the diff between Stage 0 and Stage 1 smaller. `View`, `Text`, and `StyleSheet` come from `react-native`.

`App` is a function component. It returns JSX: a `View` whose only child is a `Text` reading "Tap Game". The `View` carries `style={styles.container}`, which is looked up from the `StyleSheet` created below.

`StyleSheet.create` is given a single object with one key, `container`. The properties are worth noting individually:

- `flex: 1` makes the `View` expand to fill its parent. Without it, the `View` would shrink to the size of its children, and the white background would not cover the screen.
- `backgroundColor: '#fefefe'` sets a near-white background.
- `padding: 20` adds twenty units of space inside the `View`, on all four sides.
- `color: '#333'` is present but has no effect here. `color` controls text colour and belongs on a `Text` style, not a `View` style. It is a harmless leftover — the kind of thing that appears when a stylesheet is built by copying — and it is worth noticing now, because later stages will put text styles where they actually apply.

The `export default App;` line makes the component the module's default export, which is what the React Native entry point expects.

### Revision notes and gotchas

- **`color` on a `View` does nothing.** Move it to a `Text` style if the intent is to colour text. It is retained here because it was in the in-class code, and because recognising dead style properties is part of reading a stylesheet critically.
- **The unused `useState` import is deliberate.** It is not an oversight. Later stages will use it; keeping it here means the reader can focus on the component's structure without also tracking which imports change between Stage 0 and Stage 1.
- **`flex: 1` is doing more work than it appears to.** If it were removed, the background colour would collapse to the height of the text, and the layout would not behave the way later stages assume. Every later stage relies on this line.

---

## Stage 1 — A Tappable Button and State

### What this stage introduces

This stage adds the first piece of interactivity: a button that responds to being pressed, and a number on screen that increases each time it is pressed. Two new ideas appear together. The first is `TouchableOpacity`, a React Native component that renders its children and calls a function when pressed. The second is `useState`, the hook that gives a function component a piece of memory that survives re-renders and whose update triggers the next render.

The stage also introduces a distinction that will matter more later: the difference between calling a function and passing it. `onPress={handleTap}` passes the function; `onPress={handleTap()}` would call it immediately and pass its return value.

### The code

```tsx
import { useState } from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';

function App() {
  const [taps, setTaps] = useState<number>(0);

  const handleTap = () => {
    setTaps(taps + 1);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Tap Game</Text>
      <TouchableOpacity style={styles.button} onPress={handleTap}>
        <Text style={styles.buttonText}>{taps}</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fefefe',
    padding: 20,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
    color: '#333',
  },
  button: {
    width: 200,
    height: 200,
    borderRadius: 100,
    backgroundColor: '#007bff',
    justifyContent: 'center',
    alignItems: 'center',
    elevation: 5,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
  },
  buttonText: {
    fontSize: 48,
    color: '#fff',
    fontWeight: 'bold',
  },
});

export default App;
```

### Walkthrough

The component now holds state. `const [taps, setTaps] = useState<number>(0);` declares a state variable named `taps`, initialised to `0`, and a setter named `setTaps`. The angle-bracket `<number>` is a TypeScript type argument: it tells the compiler that `taps` is a number and that `setTaps` accepts a number. The initial value `0` is passed to `useState` and is used only on the first render; on subsequent renders, React returns whatever value is currently stored.

`handleTap` is a function declared inside the component. It calls `setTaps(taps + 1)`. Reading this line carefully: it takes the current value of `taps`, adds one, and passes the result to `setTaps`. Because `taps` is captured from the render in which `handleTap` was created, this form depends on the render being current. In this stage that is fine, because the button is the only thing that changes `taps`, and React re-renders after each set. Stage 2 will introduce a form of `setTaps` that does not depend on the captured value.

The JSX has grown. A `Text` with the title sits above a `TouchableOpacity`. The `TouchableOpacity` carries two props:

- `style={styles.button}`, which gives it the circular blue appearance.
- `onPress={handleTap}`, which passes the function itself. Note the absence of parentheses: `handleTap` is being handed to the component, not called. If it were written `onPress={handleTap()}`, the function would run during rendering and its return value — `undefined` — would be passed as the press handler.

Inside the `TouchableOpacity` is a `Text` whose content is `{taps}`. The braces switch from JSX into JavaScript, so the current value of `taps` is rendered. When `setTaps` is called, React schedules a re-render, `taps` has a new value, and this `Text` updates.

The stylesheet has grown to match. `container` gains `justifyContent: 'center'` and `alignItems: 'center'`, which centre its children along the main and cross axes respectively. `title` sets the heading's size, colour, and bottom margin. `button` gives the touchable its circular shape through the combination of equal `width` and `height` with `borderRadius: 100`, and its lift through `elevation` (used on Android) and the four `shadow*` properties (used on iOS). `buttonText` sets the size, colour, and weight of the number inside the button.

### Revision notes and gotchas

- **`onPress={handleTap}` vs `onPress={handleTap()}`.** The first passes the function; the second calls it. This is the single most common mistake at this stage. If the button appears to do nothing, or if the number jumps by one the instant the screen renders, this is the line to check.
- **The `setTaps(taps + 1)` form is correct here but fragile in general.** It reads the value of `taps` from the render in which `handleTap` was created. If two updates were queued in the same render — for example, if a single press triggered two increments — the second would overwrite the first, because both would compute `taps + 1` from the same starting value. Stage 2 introduces the functional updater form, `setTaps(prev => prev + 1)`, which does not have this problem. The switch is worth noticing when it happens.
- **`TouchableOpacity` is not the only touch component.** `Pressable`, `TouchableHighlight`, and `Button` all exist, and each has trade-offs. `TouchableOpacity` is used here because it is the simplest to style and because its name describes what it does: it fades the opacity of its children while pressed.
- **The unused `useState` import from Stage 0 is now used.** This is the only change to the import block between the two stages.

### Key concepts

**State and re-rendering.** A function component runs top to bottom every time it renders. Local variables are re-created on each run and are lost when the render finishes. `useState` is what makes a value survive across renders: React stores it outside the component, returns the current value on each render, and schedules a new render when the setter is called. The number of taps is state because it changes over time and the screen must reflect the change. The title text is not state because it never changes.

**Controlled rendering from state.** The `Text` inside the button does not hold its own value. It displays whatever `taps` currently is. This is the essence of rendering from state: the UI is a function of the data, not a separate copy of it. If the state is correct, the screen is correct; there is no second place where the tap count lives and could drift out of sync.

---

## Stage 2 — useEffect, the Timer, and Reset

### What this stage introduces

This stage turns the counter into a game. A five-second countdown runs while the player taps, and the score is whatever was reached when the countdown reaches zero. Three new ideas appear together.

The first is `useEffect`, the hook that runs code in response to a component rendering, and that can clean up after itself when the situation it was set up for no longer applies. The second is `setInterval` and its counterpart `clearInterval`, which together schedule a repeated action and cancel it. The third is a small piece of state, `gameActive`, that records whether the game is currently running; the countdown effect is gated on it.

The stage also introduces the reset button, which restores the game to its starting condition. Reset is not a separate mechanism — it is another set of state updates, and it demonstrates that the game's condition is entirely captured by its state variables.

### The code

```tsx
import { useState, useEffect } from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';

const GAME_DURATION = 5; // The duration of the game in seconds

function App() {
  const [taps, setTaps] = useState<number>(0);
  const [timeLeft, setTimeLeft] = useState<number>(GAME_DURATION);
  const [gameActive, setGameActive] = useState<boolean>(false);

  // useEffect to handle the timer logic
  useEffect(() => {
    let timer: NodeJS.Timeout | null = null;
    if (gameActive && timeLeft > 0) {
      timer = setInterval(() => {
        setTimeLeft(prevTime => prevTime - 1);
      }, 1000);
    } else if (timeLeft === 0) {
      setGameActive(false);
    }

    return () => {
      if (timer) {
        clearInterval(timer);
      }
    };
  }, [gameActive, timeLeft]); // useEffect dependent on gameActive and timeLeft state variables

  const handleTap = () => {
    if (!gameActive) {
      setGameActive(true);
      setTaps(1);
    } else {
      setTaps(prevTaps => prevTaps + 1);
    }
  };

  const handleReset = () => {
    setGameActive(false);
    setTaps(0);
    setTimeLeft(GAME_DURATION);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Tap Game</Text>
      
      <View style={styles.infoContainer}>
        <Text style={styles.timerText}>Time Left: {timeLeft}s</Text>
        <Text style={styles.scoreText}>Taps: {taps}</Text>
      </View>

      <TouchableOpacity 
        style={[styles.button, !gameActive && styles.startButton]} 
        onPress={handleTap}
        disabled={!gameActive && timeLeft === 0}
      >
        <Text style={styles.buttonText}>{gameActive ? "TAP" : (timeLeft === 0 ? "GAME OVER" : "START")}</Text>
      </TouchableOpacity>
      
      {timeLeft === 0 && (
        <TouchableOpacity style={styles.resetButton} onPress={handleReset}>
          <Text style={styles.buttonText}>RESET</Text>
        </TouchableOpacity>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fefefe',
    padding: 20,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
    color: '#333',
  },
  infoContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    width: '80%',
    marginBottom: 20,
  },
  timerText: {
    fontSize: 20,
    color: '#d9534f',
  },
  scoreText: {
    fontSize: 20,
    color: '#337ab7',
  },
  button: {
    width: 200,
    height: 200,
    borderRadius: 100,
    backgroundColor: '#007bff',
    justifyContent: 'center',
    alignItems: 'center',
    elevation: 5,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
  },
  startButton: {
    backgroundColor: '#28a745',
  },
  buttonText: {
    fontSize: 24,
    color: '#fff',
    fontWeight: 'bold',
  },
  resetButton: {
    marginTop: 20,
    paddingHorizontal: 20,
    paddingVertical: 10,
    backgroundColor: '#dc3545',
    borderRadius: 5,
  },
});

export default App;
```

### Walkthrough

The component now holds three pieces of state. `taps` is unchanged in meaning from Stage 1. `timeLeft` is initialised to `GAME_DURATION`, a constant declared above the component. `gameActive` starts `false`; the game has not begun.

`handleTap` has been rewritten. On the first press, `gameActive` is `false`, so the `if` branch runs: `setGameActive(true)` and `setTaps(1)`. The tap that started the game counts as the first tap, which is why `taps` is set to `1` rather than incremented from `0`. On subsequent presses, `gameActive` is `true`, so the `else` branch runs: `setTaps(prevTaps => prevTaps + 1)`. This is the functional updater form. Instead of reading `taps` from the surrounding render, it hands React a function that receives the previous value and returns the next. React is responsible for supplying that previous value, which means the update is correct even if several are queued in the same render. Stage 1 used the direct form; the switch to the functional form is deliberate, and it is the form that will be used from here on.

The countdown lives in the first `useEffect`. Reading it top to bottom:

- A local variable `timer` is declared, typed `NodeJS.Timeout | null`, and initialised to `null`. This variable exists only for the duration of this run of the effect. It holds the handle returned by `setInterval` so that the cleanup function can cancel it. Its type is discussed separately below.
- The first branch runs when `gameActive` is `true` and `timeLeft` is greater than zero. `setInterval` is called with a callback and an interval of `1000` milliseconds. The callback calls `setTimeLeft(prevTime => prevTime - 1)` — again the functional updater form, which is the correct form here because the effect will run again on each tick, and the value of `timeLeft` in the surrounding closure will be stale by the time the callback fires. The handle returned by `setInterval` is assigned to `timer`.
- The `else if` branch runs when `timeLeft` is exactly zero. It sets `gameActive` to `false`. The effect does not schedule a new interval; the game is over.
- The effect returns a cleanup function. This function will be called by React before the effect runs again, and again when the component unmounts. It checks whether `timer` is non-null, and if so, calls `clearInterval(timer)`. The `if (timer)` guard is necessary because the effect can run without having scheduled an interval — for example, when `timeLeft` has just reached zero, no interval was created, so there is nothing to clear.

The dependency array is `[gameActive, timeLeft]`. This tells React to re-run the effect whenever either of those two values changes. When `gameActive` flips from `false` to `true`, the effect re-runs and the interval is scheduled. When `timeLeft` decrements, the effect re-runs, the previous interval is cleared by the cleanup, and a new one is scheduled. When `timeLeft` reaches zero, the effect re-runs, the `else if` branch fires, and no new interval is scheduled.

`handleReset` restores the three state variables to their starting values. `setGameActive(false)` stops the countdown. `setTaps(0)` clears the score. `setTimeLeft(GAME_DURATION)` restores the time. Because the effect depends on `timeLeft` and `gameActive`, changing them causes the effect to re-run and no new interval is scheduled, so the countdown stops cleanly.

The JSX has grown in two places. Above the button is an `infoContainer`, a horizontally arranged `View` holding the timer and the score side by side. Below the button is a conditional render: `{timeLeft === 0 && ( <TouchableOpacity …> )}`. The `&&` operator returns its right-hand side when the left is truthy and `false` when the left is falsy. React renders nothing for `false`, so the reset button appears only when the countdown has reached zero.

The main button's `style` prop is now an array: `style={[styles.button, !gameActive && styles.startButton]}`. React Native accepts an array of style objects and merges them, with later entries overriding earlier ones. When `gameActive` is `false`, `!gameActive` is `true`, and the array is `[styles.button, styles.startButton]`; the green `startButton` background overrides the blue. When `gameActive` is `true`, `!gameActive` is `false`, and the array is `[styles.button, false]`; React Native ignores the `false` entry and the blue background applies.

The button's `disabled` prop is `!gameActive && timeLeft === 0`. The button is disabled only when the game is not running and the time has run out — that is, after game over. During play, `gameActive` is `true`, so `!gameActive` is `false` and the whole expression is `false`, so the button is enabled. Before the game starts, `timeLeft` is greater than zero, so the second half of the expression is also `false`. In all states except game over, the button is enabled. This is correct, but it reads ambiguously; a comment or a named variable would improve it. The reason the button is disabled at all is to prevent the player from starting a new game by tapping, since the only allowed next action is to reset. The reset button is the intended path.

The button's label is a nested ternary: `{gameActive ? "TAP" : (timeLeft === 0 ? "GAME OVER" : "START")}`. When the game is running, the label is `TAP`. When the game is not running, the inner ternary is evaluated: if time has run out, the label is `GAME OVER`; otherwise it is `START`. Three visual states, one expression. Nested ternaries are readable when they are short, but they become hard to follow when longer; a small helper function or an `if` chain outside the JSX would be clearer if more states were added.

The stylesheet has gained `infoContainer`, `timerText`, `scoreText`, `startButton`, and `resetButton`. `infoContainer` sets `flexDirection: 'row'` to arrange the timer and score horizontally, and `justifyContent: 'space-between'` to push them to opposite ends. `startButton` overrides only the background colour, leaving the rest of `button` in place — which is why the two styles are combined in an array rather than written as a second complete style.

### Further explanation — what `timer` is, and why it is typed `NodeJS.Timeout`

The `timer` variable holds the value returned by `setInterval`. To understand what that value is, and why it is typed the way it is, it helps to start one level down with `setTimeout`, and then see `setInterval` as its repeating sibling.

**`setTimeout` schedules a single callback.** `setTimeout(fn, ms)` tells the runtime: after `ms` milliseconds, call `fn` once. It returns a value immediately — a handle — that identifies the scheduled call. That handle can be passed to `clearTimeout` to cancel the call before it fires. This is the pattern behind every "cancel this before it happens" idiom: a modal that closes after a delay, a debounce, a retry.

**`setInterval` schedules a repeating callback.** `setInterval(fn, ms)` tells the runtime: call `fn` every `ms` milliseconds, forever, until told to stop. The value it returns is the same kind of handle as `setTimeout`'s, and it is passed to `clearInterval` to stop the repetition. In this project, the interval is what makes `timeLeft` go down by one each second: the callback calls `setTimeLeft(prev => prev - 1)`, and because the effect depends on `timeLeft`, the effect re-runs after each tick — but only after clearing the previous interval first. If the previous interval were not cleared, each render would leave another interval running, and the countdown would tick faster and faster as the game progressed. This is the memory leak the cleanup function exists to prevent.

**What the handle actually is.** The specification does not say. In browsers, timer handles have historically been plain integers, and code that treated them as such — doing arithmetic on the return value of `setTimeout` — used to work and still sometimes appears in older tutorials. The HTML spec does not guarantee this, and Node.js does not follow it: there, the handle is an object, specifically an instance of Node's internal `Timeout` class, with methods and internal fields. This is why the two annotated types exist:

- `NodeJS.Timeout` is the type Node's runtime uses. Because many React Native projects include `@types/node` (usually as a transitive dependency of the toolchain), this type is available, and it is what the in-class code uses.
- `ReturnType<typeof setInterval>` is the type that asks TypeScript to work out what `setInterval` returns *in the current environment*, without naming that type explicitly. On a device running React Native's JavaScript engine, that may be `NodeJS.Timeout` or a numeric handle, depending on the engine; `ReturnType<typeof setInterval>` produces the correct one automatically.

**Why the in-class code uses `NodeJS.Timeout`.** It is the form that appears in most tutorials and in much of the surrounding ecosystem, and it works. The trade-off is that it commits the code to a specific runtime's type. If the same file were built for a browser or a different engine, the annotation could be wrong even though the code would still run — because TypeScript's type checking happens at compile time and the runtime value is unaffected. `ReturnType<typeof setInterval>` avoids that commitment by asking the compiler to resolve the type from the environment. It is more portable, and it is the form to prefer in code that may be reused across runtimes. It is worth knowing both, and worth knowing why the choice is largely cosmetic: the value is only ever stored and handed back to `clearInterval`, never inspected.

**Where the handle's opacity matters.** Because the handle is opaque, the code must not do anything with it other than store it and pass it back to `clearInterval`. Assigning it to a number variable, comparing it to another handle, or assuming it is truthy would all be mistakes. The `if (timer)` guard in the cleanup function is safe precisely because it only checks for the `null` initial value; it does not interrogate the handle itself.

**Further reading.**

- Node.js timers reference: https://nodejs.org/api/timers.html
- Node.js guide, "Discover JavaScript Timers": https://nodejs.org/learn/asynchronous-work/discover-javascript-timers

### Key concepts

**`useEffect` and the dependency array.** `useEffect(fn, deps)` runs `fn` after the component renders, and re-runs it whenever any value in `deps` has changed since the previous render. The dependency array is not a list of variables the effect *uses*; it is a list of variables whose change should cause the effect to run again. The distinction matters: an effect can read a value without depending on it if the effect does not need to re-run when that value changes. Getting this wrong is the most common source of both missing updates and unnecessary re-runs.

**Cleanup functions.** The function returned by `useEffect` is called by React before the effect runs again and again when the component unmounts. It exists to undo whatever the effect set up. Here, the effect sets up an interval, so the cleanup clears it. If the cleanup were omitted, every re-run would leave the previous interval running, and the countdown would tick at an accelerating rate — a memory leak expressed as a visible bug.

**`setInterval` and `clearInterval`.** `setInterval(fn, ms)` schedules `fn` to run every `ms` milliseconds and returns a handle. `clearInterval(handle)` cancels the schedule. The pair is used together, and the handle must be stored somewhere the cleanup function can reach it — which is why `timer` is a local variable inside the effect, not a piece of state. State would cause a re-render every time the handle changed; a local variable does not, and the handle does not need to be rendered.

**Functional updaters.** `setTimeLeft(prev => prev - 1)` and `setTaps(prev => prev + 1)` pass a function to the setter instead of a new value. React calls that function with the previous state and uses the return value as the next state. This is the correct form when the update depends on the previous value *and* the update may be queued alongside others. In the interval callback, the surrounding `timeLeft` is stale by the time the callback fires — the callback closes over the render in which the effect ran, not the render in which it fires — so the functional form is not just preferable but required.

**Conditional rendering with `&&`.** `{timeLeft === 0 && <Reset />}` renders the reset button only when `timeLeft` is zero. The idiom relies on React rendering `false`, `null`, and `undefined` as nothing. It is compact and idiomatic, but it has one trap: if the left-hand side can be `0`, React will render the `0` itself. `{count && <Thing />}` renders `0` when `count` is zero. The fix is to write the condition so it evaluates to a boolean, as `timeLeft === 0` does here.

**Style arrays.** `style={[styles.button, !gameActive && styles.startButton]}` merges style objects, later ones overriding earlier ones, and ignores falsy entries. This is how a component carries a base style and a variation without duplicating the base. It is also the reason `startButton` only sets `backgroundColor`: everything else comes from `button`.

### Revision notes and gotchas

- **The `else if (timeLeft === 0)` branch does not call `clearInterval`.** It does not need to: no interval was created on that run of the effect, so `timer` is still `null`, and the cleanup's `if (timer)` guard skips the call. The cleanup runs unconditionally in every case, but it only acts when there is something to clear. This is the intended shape, and it is why the `if (timer)` guard is present rather than optional.
- **The functional updater is required inside the interval callback.** The callback captures `timeLeft` from the render in which the effect ran. By the time the callback fires, a second has passed, and the surrounding `timeLeft` is the value from the previous render, not the current one. Writing `setTimeLeft(timeLeft - 1)` would decrement from a stale value and the countdown would appear to skip. The `prev => prev - 1` form avoids this because React supplies the current value at the moment the update is processed.
- **`NodeJS.Timeout` is the in-class annotation, and it works.** It is not wrong, but it is more specific than the code requires: the handle is never inspected, only stored and passed back to `clearInterval`. The portable alternative is `ReturnType<typeof setInterval>`, which resolves to whichever type `setInterval` returns in the current environment. See the "Further explanation" section above for the reasoning. The in-class code is preserved here so that later stages match what was written on the board; the alternative is noted rather than applied.
- **The `disabled` expression reads backwards.** `disabled={!gameActive && timeLeft === 0}` is correct, but a reader has to think through it. A named variable — `const gameOver = !gameActive && timeLeft === 0;` — would make the JSX read as `disabled={gameOver}`, which is clearer. The in-class form is preserved; the refactor is worth considering when the component grows.
- **The nested ternary in the button label has three states.** `gameActive ? "TAP" : (timeLeft === 0 ? "GAME OVER" : "START")` produces `TAP`, `GAME OVER`, or `START`. This is fine at three states. Beyond three, an `if` chain outside the JSX, or a small helper function, is easier to read and easier to change.
- **The `color` property is still on `container`, where it does nothing.** It was noted in Stage 0 and has not been removed. The intent may have been to colour the title; if so, it belongs on `title`, which already sets its own colour. Keeping it is harmless, but it is the kind of leftover that accumulates.
- **Debugging the timer.** A `console.log` inside the interval callback — `console.log("tick", timeLeft)` — combined with a `console.log("cleanup")` inside the returned function will show, in the console, the alternation of "cleanup" then "tick" once per second. If the cleanup were removed, the log would show multiple ticks per second, which is the leak made visible. This is worth demonstrating once, live, because it makes the abstract idea of "cleanup" concrete.
- **The `useEffect` will re-run once per tick, which is expected.** Some readers expect the interval to be scheduled once and left alone. It is not: each render with a new `timeLeft` triggers a new run of the effect, and the previous interval is cleared first. The alternative — scheduling the interval once and reading `timeLeft` from state inside the callback — is possible but requires care with stale closures, and it is not what the in-class code does. The current shape is the more teachable one, because the dependency array and the cleanup are both doing visible work.

---

## Stage 3 — Persistent High Scores with AsyncStorage and FlatList

### What this stage introduces

This stage makes the game remember. Up to now, the score exists only for as long as the app is running: close the app, and the tap count, the top score, and the fact that a game was ever played are all gone. Persistent storage changes that. It also introduces the list that displays the stored scores.

Three ideas arrive together. The first is `AsyncStorage`, a key-value store that survives the app being closed, and the fact that it is asynchronous — its operations return promises, and the code that uses it must be `async` and must `await` the results. The second is `FlatList`, the component for rendering lists efficiently, and its three required props: `data`, `renderItem`, and `keyExtractor`. The third is the logic of maintaining a top-5 list: append the new score, sort by taps descending, slice off the first five.

The stage is presented in two parts. Part A is the version taught in class. It works — scores are saved, they persist, they are sorted, they are trimmed. But it contains a bug that was noticed live: while the game button is being pressed repeatedly, the countdown appears to freeze. Part B corrects that bug and explains why the correction is the right shape.

The reason for the two-part presentation is that the bug is not a typo. It is a consequence of how `useEffect`'s dependency array interacts with a piece of state that the effect does not actually need to watch. The lesson is about the dependency array itself, and it is more useful to see the failure and understand it than to be shown the fix directly.

---

## Stage 3, Part A — The In-Class Version

### What this part introduces

The in-class version adds persistent storage and the high-score list. It also adds a dependency to the timer effect that was not present in Stage 2: `taps`. That addition is what produces the bug. The addition is not arbitrary — it is what the editor's lint warning asked for, and the class accepted the suggestion. That is the important part of the story: the warning was correct in the narrow sense that the effect's body does reference `taps`, but the fix it suggested introduced a different problem.

### The code

```tsx
import { useState, useEffect } from 'react';
import { View, Text, StyleSheet, TouchableOpacity, FlatList } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

const GAME_DURATION = 5;

// Define the type for our score objects
type Score = {
  id: string;
  taps: number;
  date: string;
};

// User-defined component to render each score item for the FlatList
function renderScoreItem({ item }: { item: Score }) {
  return (
    <View style={styles.scoreItem}>
      <Text style={styles.scoreItemText}>Taps: {item.taps}</Text>
      <Text style={styles.scoreItemText}>Date: {item.date}</Text>
    </View>
  );
}

function App() {
  const [taps, setTaps] = useState<number>(0);
  const [timeLeft, setTimeLeft] = useState<number>(GAME_DURATION);
  const [gameActive, setGameActive] = useState<boolean>(false);
  const [highScores, setHighScores] = useState<Score[]>([]);

  // useEffect to handle the timer logic
  useEffect(() => {
    let timer: NodeJS.Timeout | null = null;

    if (gameActive && timeLeft > 0) {
      timer = setInterval(() => {
        setTimeLeft(prevTime => prevTime - 1);
      }, 1000);
    } else if (timeLeft === 0) {
      setGameActive(false);
      saveScore(taps); // Call saveScore when the game ends
    }

    return () => {
      if (timer) {
        clearInterval(timer);
      }
    };

  }, [gameActive, timeLeft, taps]); // dependent array updated to monitor 'taps' state variable 

  // useEffect to load high scores from storage when the app loads
  useEffect(() => {
    loadScores();
  }, []); // empty dependency array 

  // Function to load scores from AsyncStorage
  const loadScores = async () => {
    try {
      const storedScores = await AsyncStorage.getItem('#highScores');
      if (storedScores !== null) {
        // Parse the stored JSON string back into an array
        setHighScores(JSON.parse(storedScores));
      }
    } catch (e) {
      console.error("Failed to load scores", e);
    }
  };

  // Function to save a new score
  const saveScore = async (newTaps: number) => {
    const now = new Date();
    const newScore: Score = {
      id: now.getTime().toString(), // Unique ID based on timestamp
      taps: newTaps,
      date: now.toLocaleString(),
    };

    // this would be the next logical step, but we are choosing to only store the top 5 scores  
    // setHighScores([...highScores, newScore]); 
 
    // So instead of directly saving the newScore into the array using the setter - we are creating a new (temporary) array with the new score and all existing high scores

    const updatedScores = [...highScores, newScore];
    // setHighScores(updatedScores); // this was shown in class 
    // The above line now takes the temporary array with all the scores and sets it back into the highScores state variable


    // Let's consider that there will be too many scores to store 
    // So instead of storing all - sort the list based on taps &
    // only store the top 5 scores in the async storage
 
    // Sort the scores in descending order based on taps
    updatedScores.sort((a, b) => b.taps - a.taps);

    // Keep only the top 5 scores using the slice() method
    const top5Scores = updatedScores.slice(0, 5);

    // Now we use the setter, setHighScores to accept the new temporary array top5Scores to be the new set of highScores
    setHighScores(top5Scores); // Update state to trigger re-render

    try {
      // Save the updated top 5 scores to AsyncStorage
      await AsyncStorage.setItem('#highScores', JSON.stringify(top5Scores));
    } catch (e) {
      console.error("Failed to save score", e);
    }
  };

  const handleTap = () => {
    if (!gameActive) {
      setGameActive(true);
      setTaps(1);
    } else {
      setTaps(prevTaps => prevTaps + 1);
    }
  };

  const handleReset = () => {
    setGameActive(false);
    setTaps(0);
    setTimeLeft(GAME_DURATION);
  };
  

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Tap Game</Text>
      
      <View style={styles.infoContainer}>
        <Text style={styles.timerText}>Time Left: {timeLeft}s</Text>
        <Text style={styles.scoreText}>Taps: {taps}</Text>
      </View>

      <TouchableOpacity 
        style={[styles.button, !gameActive && styles.startButton]} 
        onPress={handleTap}
        disabled={!gameActive && timeLeft === 0}
      >
        <Text style={styles.buttonText}>{gameActive ? "TAP" : (timeLeft === 0 ? "GAME OVER" : "START")}</Text>
      </TouchableOpacity>
      
      {timeLeft === 0 && (
        <TouchableOpacity style={styles.resetButton} onPress={handleReset}>
          <Text style={styles.buttonText}>RESET</Text>
        </TouchableOpacity>
      )}

      <Text style={styles.highScoresTitle}>High Scores</Text>
      <FlatList
        data={highScores}
        renderItem={renderScoreItem}
        keyExtractor={item => item.id}
        style={styles.list}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fefefe',
    padding: 20,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
    color: '#333',
  },
  infoContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    width: '80%',
    marginBottom: 20,
  },
  timerText: {
    fontSize: 20,
    color: '#d9534f',
  },
  scoreText: {
    fontSize: 20,
    color: '#337ab7',
  },
  button: {
    width: 200,
    height: 200,
    borderRadius: 100,
    backgroundColor: '#007bff',
    justifyContent: 'center',
    alignItems: 'center',
    elevation: 5,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
  },
  startButton: {
    backgroundColor: '#28a745',
  },
  buttonText: {
    fontSize: 24,
    color: '#fff',
    fontWeight: 'bold',
  },
  resetButton: {
    marginTop: 20,
    paddingHorizontal: 20,
    paddingVertical: 10,
    backgroundColor: '#dc3545',
    borderRadius: 5,
  },
  highScoresTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginTop: 30,
    marginBottom: 10,
    color: '#333',
  },
  list: {
    width: '100%',
  },
  scoreItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    padding: 10,
    backgroundColor: '#eee',
    borderRadius: 5,
    marginBottom: 5,
  },
  scoreItemText: {
    fontSize: 16,
    color: '#555',
  },
});

export default App;
```

### Walkthrough

The import block gains `FlatList` and `Alert` from `react-native`, and `AsyncStorage` from its package. 

Above the component, a `type Score` declares the shape of a single high-score entry: a string `id`, a number `taps`, and a string `date`. TypeScript will enforce this shape wherever `Score` is used.

Also above the component, `renderScoreItem` is a small function that takes `{ item }` and returns a `View` containing two `Text` elements, one for the tap count and one for the date. It is the `renderItem` for the `FlatList`. Its signature — `({ item }: { item: Score })` — is dictated by `FlatList`: the function is called with an object that has an `item` property, and the type annotation narrows `item` to a `Score`. Defining it outside `App` matters: the function is not re-created on each render, so the `FlatList` sees a stable reference and does not need to re-render its rows unnecessarily.

The component gains a fourth piece of state: `highScores`, an array of `Score` objects, initialised empty.

The first `useEffect` is largely the same as in Stage 2, with two changes. The `else if (timeLeft === 0)` branch now calls `saveScore(taps)` in addition to `setGameActive(false)`. The dependency array is now `[gameActive, timeLeft, taps]` — the `taps` dependency has been added, with the in-class comment noting it. The addition of `taps` is the source of the bug. It is worth reading the effect with that in mind:

- On every render where any of `gameActive`, `timeLeft`, or `taps` has changed since the previous render, this effect re-runs.
- On each re-run, the cleanup from the previous run fires first, clearing the previous interval.
- A new interval is then created if `gameActive` is `true` and `timeLeft > 0`.

While the game button is being pressed, `taps` increments on each press. Each increment triggers a render, which triggers the effect to re-run, which clears the current interval and creates a new one. But `setInterval` does not fire immediately — it fires after one second. If the effect re-runs before that second has elapsed, the interval is cleared and replaced, and the countdown never gets its chance to tick. The timer appears frozen while the button is being pressed and resumes when the pressing stops. This is exactly what was observed in class.

The second `useEffect` has an empty dependency array and calls `loadScores()`. It runs once, after the first render, when the component mounts. Its purpose is to read any previously stored scores from `AsyncStorage` and put them into state.

`loadScores` is declared `async`. It calls `await AsyncStorage.getItem('#highScores')`. The `#` in the key is unusual — `AsyncStorage` keys are ordinary strings, and `#` is not a special character, but it is also not conventional. Most examples use a plain name like `'highScores'`. It works, but the `#` is noise; Part B drops it. If the stored value is not `null`, the string is parsed with `JSON.parse` and the resulting array is passed to `setHighScores`. If anything throws — a corrupt string, a storage error — the `catch` logs the error and leaves the state unchanged.

`saveScore` is also `async`. It builds a new `Score` object from the current time: `id` is the timestamp in milliseconds as a string, which is unique enough for this purpose because two scores cannot be created in the same millisecond; `taps` is the score passed in; `date` is a human-readable string. It then constructs a new array — `const updatedScores = [...highScores, newScore]` — combining the existing scores with the new one. This does not mutate `highScores`; it creates a fresh array. This matters because React state must be replaced, not mutated, for changes to be detected.

The array is then sorted in place by `updatedScores.sort((a, b) => b.taps - a.taps)`. The comparator `b.taps - a.taps` returns a negative number when `b` should come first, which for numbers means the array is sorted in descending order of `taps`. The `slice(0, 5)` then takes the first five entries. `slice` does not mutate its receiver, so a new array of at most five scores is produced. `setHighScores(top5Scores)` updates state, causing a re-render, and the `FlatList` displays the new array.

Finally, `await AsyncStorage.setItem('#highScores', JSON.stringify(top5Scores))` writes the array to storage. `AsyncStorage` can only store strings, so the array is serialised with `JSON.stringify`. The `await` ensures the write completes before the function returns; the surrounding `try`/`catch` handles a write failure.

The JSX is unchanged above the high-scores section. Below the reset button, a `Text` with the title "High Scores" is followed by the `FlatList`. The `FlatList` takes four props: `data={highScores}` is the array to render; `renderItem={renderScoreItem}` is the function that turns each item into a row; `keyExtractor={item => item.id}` tells React how to identify each row uniquely, using the score's `id`; and `style={styles.list}` gives the list a full width. The `keyExtractor` is what allows React to keep track of rows as the array changes — when a new score is added and the list re-sorts, React uses the keys to preserve the identity of the existing rows and only update what has moved or changed.

The stylesheet gains three entries: `highScoresTitle`, `list`, and two for the individual score rows — `scoreItem` and `scoreItemText`. `scoreItem` uses `flexDirection: 'row'` and `justifyContent: 'space-between'` to place the tap count on the left and the date on the right of each row.

### The bug, as observed in class

When the game was played in class, the countdown behaved incorrectly. While the player was tapping the button, the timer stopped advancing. It resumed once the tapping stopped. The behaviour was reproducible: tap continuously, and `timeLeft` would sit at whatever value it had reached when the tapping started. Pause, and it would continue from there. The game was still playable, but the timer was no longer measuring real seconds during active play, so scores were inflated.

The cause is the `taps` entry in the dependency array. The reasoning is straightforward once the effect's lifecycle is traced:

1. A press calls `handleTap`, which calls `setTaps(prev => prev + 1)`. React schedules a re-render.
2. The re-render produces a new value of `taps`. The effect's dependency array has changed, so React will re-run the effect.
3. Before re-running the effect, React calls the cleanup function from the previous run. The cleanup calls `clearInterval(timer)`, cancelling the interval that was scheduled one second earlier.
4. The effect re-runs. `gameActive` is `true` and `timeLeft > 0`, so `setInterval` is called, scheduling a new callback for one second in the future.
5. If another press arrives before that second has elapsed, steps 1–4 repeat. The interval is cleared and rescheduled before it ever fires.

The countdown only advances when a full second passes without an intervening press. Continuous tapping prevents any full second from passing, so the countdown appears frozen.

The `taps` dependency was added because the effect's body calls `saveScore(taps)`, and the linter flagged `taps` as a referenced-but-not-listed variable. The linter's rule is correct in general: an effect that reads a value from the surrounding render should declare that value as a dependency, so that when the value changes, the effect re-runs with the new value. The problem is that the effect does not *need* to read the freshest `taps` on every render. It only needs the freshest `taps` at the moment `timeLeft` reaches zero. Reacting to every change in `taps` is over-reacting, and the over-reaction breaks the interval.

The correct fix, therefore, is not to remove the read of `taps` but to restructure so that the effect does not react to `taps`. The effect that manages the interval should react only to `gameActive` and `timeLeft`. The read of `taps` — which is only needed at game end — should live in a separate effect that fires once, when `timeLeft` reaches zero. That separation is the content of Part B.

### Key concepts

**`AsyncStorage` is a key-value store.** It maps strings to strings. Anything more complex — an array of objects, in this case — must be serialised with `JSON.stringify` before being written and deserialised with `JSON.parse` after being read. It is unencrypted and not suited to sensitive data, but it is the right tool for small, non-critical preferences and scores.

**`AsyncStorage` is asynchronous.** Both `getItem` and `setItem` return promises. Reading from device storage takes time, and the code must not block the UI while it happens. `await` pauses the enclosing `async` function until the promise resolves, without blocking the rest of the application. The rest of the app keeps rendering, responding to touches, and running effects; only the function that awaited is paused.

**Analogy — ordering at a coffee shop with a buzzer.** When you order a coffee at a counter, you are handed a buzzer and told to sit down. You do not stand at the counter, blocking the queue, waiting for your drink. You sit, you read, you talk. When the buzzer goes off, you get up and collect the drink. Three things are happening at once, and it is worth naming them separately because `async`/`await` does the same three things:

- The barista making the drink is the **asynchronous operation** running in the background. This is `AsyncStorage.getItem` doing its work.
- You sitting down and reading is the **rest of the program continuing**. The app is still rendering, still responding to touches, still running other effects. Nothing is blocked.
- The buzzer going off, and you getting up to collect the drink, is **`await` resuming the function**. The `await` line is where the function paused; that is where it continues, once the operation finishes.

The analogy is worth keeping in mind because it separates three things that are often collapsed into one: the work, the waiting, and the resumption. In code, the work is the `AsyncStorage` call, the waiting is implicit, and the resumption is the line after the `await`. If you ever find yourself wondering "what is blocked while this runs?", the answer is: only the function that awaited. Everything else in the app is the café, still full of people.

**`FlatList` renders lists efficiently.** Unlike `ScrollView`, which renders all its children regardless of whether they are visible, `FlatList` renders only the rows currently on screen, plus a small buffer, and recycles them as the user scrolls. For long lists this is a significant performance difference. For the five rows this app stores, the benefit is academic, but the pattern is the one to learn, because it is what production lists use.

**`keyExtractor` gives each row a stable identity.** `FlatList` needs to know which item in `data` corresponds to which rendered row, so that when the data changes — items added, removed, reordered — it can reuse the correct rows rather than re-rendering everything. The `keyExtractor` returns a string that is unique per item and stable across renders. Here, that is `item.id`, a timestamp that does not change. Using the array index as the key is a common mistake: the index changes when items are inserted or reordered, which defeats the purpose of the key and can cause visible glitches.

**Replacing state rather than mutating it.** `const updatedScores = [...highScores, newScore]` produces a new array; `setHighScores(top5Scores)` replaces the state with that array. React compares the new state reference to the old one to decide whether a re-render is needed. If `highScores.push(newScore)` had been called, the array reference would be unchanged and React would not know to re-render. The spread-and-slice pattern is the standard way to produce a new array from an old one without mutating.

### Revision notes and gotchas

- **The `taps` dependency is the bug.** It is not a mistake in the sense of a typo — it was added on the recommendation of the linter — but it produces incorrect behaviour. The reasoning is above; the correction is in Part B.
- **The `#` in the storage key is unconventional.** `'#highScores'` works, but `#` has no special meaning to `AsyncStorage` and serves only to distinguish this key from any other. A plain `'highScores'` is clearer. Part B drops it.
- **`Alert` is imported but unused.** It was in the in-class version and has been retained to match. Unused imports are harmless at runtime but will produce a lint warning, and they accumulate.
- **`renderScoreItem` is defined outside `App` for a reason.** If it were defined inside, it would be re-created on each render, and the `FlatList` would see a new function reference each time, which can defeat some of its optimisation. Defining it outside gives it a stable identity, which is the correct form for a `renderItem` that does not depend on the component's state.
- **The `saveScore` function closes over `highScores`.** This will matter in Stage 4, when the correction in Part B changes the effect that calls `saveScore`. In Part A, the effect that calls `saveScore` has `taps` in its dependencies, which means it re-runs often, which means `saveScore` is re-created often, and the closure it captures is usually fresh. Once the effect is corrected in Part B, that freshness is no longer guaranteed, and the closure can be stale. Stage 4 addresses this; it is noted here so that the connection is visible.
- **The dependency array warning is not always right to follow.** The lint rule that flagged `taps` is correct in general but wrong in this specific case, because the effect's relationship to `taps` is not "react to it" but "read it at a specific moment". The right response to the warning is not to silence it, but to restructure so that the effect's dependencies reflect what it actually reacts to. Part B does that.

---

## Stage 3, Part B — The Corrected Version

### What this part corrects

Part A's single effect does two jobs: it manages the interval, and it detects the end of the game. These two jobs react to different things. The interval should react to whether the game is running and how much time is left. The end-of-game detection should react to the moment `timeLeft` reaches zero. Combining them forces the effect to depend on everything either job reacts to, which is why `taps` ended up in the dependency array.

The correction is to split the single effect into two. One effect manages the interval and depends on `[gameActive, timeLeft]`. The other detects the end of the game and depends on `[timeLeft]`. Neither depends on `taps`, and the countdown is no longer interrupted by the player's presses.

This is not a workaround. It is the correct shape, and the reason it is correct is worth stating plainly: **each effect should do one thing, and should depend on exactly what that one thing reacts to**. When an effect does two things, its dependency array is forced to be the union of both things' dependencies, and it re-runs whenever either thing changes. Splitting the effect allows each half to react only to what it needs. This is not an optimisation; it is a correctness fix, and the bug in Part A is a direct consequence of not following it.

### The code

```tsx
import { useState, useEffect } from 'react';
import { View, Text, StyleSheet, TouchableOpacity, FlatList } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

const GAME_DURATION = 5; // Game duration in seconds

// Define the type for each high score entry
type Score = {
  id: string;
  taps: number;
  date: string;
};

// User-defined component to render each score item for the FlatList
function renderScoreItem({ item }: { item: Score }) {
  return (
    <View style={styles.scoreItem}>
      <Text style={styles.scoreItemText}>Taps: {item.taps}</Text>
      <Text style={styles.scoreItemText}>Date: {item.date}</Text>
    </View>
  );
}


function App() {
  const [taps, setTaps] = useState<number>(0); // Number of user taps
  const [timeLeft, setTimeLeft] = useState<number>(GAME_DURATION); // Remaining game time
  const [gameActive, setGameActive] = useState<boolean>(false); // Is the game currently running?
  const [highScores, setHighScores] = useState<Score[]>([]); // List of top 5 scores

  // 1 Effect: Timer logic - runs every second while game is active
  useEffect(() => {
    let timer: NodeJS.Timeout | null = null;

    if (gameActive && timeLeft > 0) {
      timer = setInterval(() => {
        setTimeLeft(prevTime => prevTime - 1);
      }, 1000);
    } 

    return () => {
      if (timer) {
        clearInterval(timer);
      }
    };
  }, [gameActive, timeLeft]); 

  // 2 Effect: Detects when the game ends and saves the score
  useEffect(() => {
    if (timeLeft === 0 && gameActive) {
      setGameActive(false);        // End the game
      saveScore(taps);             // Save the score once
    }
  }, [timeLeft]); // Only triggers when timeLeft changes

  // 3 Effect: Load saved high scores from AsyncStorage on first render
  useEffect(() => {
    loadScores();
  }, []);

  // Loads scores from persistent storage
  const loadScores = async () => {
    try {
      const storedScores = await AsyncStorage.getItem('#highScores');
      if (storedScores !== null) {
        setHighScores(JSON.parse(storedScores)); // Load and update state
      }
    } catch (e) {
      console.error("Failed to load scores", e);
    }
  };

  // Saves the current score to AsyncStorage and updates the top 5 list
  const saveScore = async (newTaps: number) => {
    const now = new Date();
    const newScore: Score = {
      id: now.getTime().toString(), // Use timestamp as unique ID
      taps: newTaps,
      date: now.toLocaleString(),   // Save human-readable date
    };

    const updatedScores = [...highScores, newScore];

    // Sort and keep top 5 scores
    const top5Scores = updatedScores.sort((a, b) => b.taps - a.taps).slice(0, 5);
    setHighScores(top5Scores);

    try {
      await AsyncStorage.setItem('#highScores', JSON.stringify(top5Scores));
    } catch (e) {
      console.error("Failed to save score", e);
    }
  };

  // Called when user taps the main game button
  const handleTap = () => {
    if (!gameActive) {
      // Start new game
      setGameActive(true);
      setTaps(1);
      setTimeLeft(GAME_DURATION);
    } else {
      // Increment tap count
      setTaps(prevTaps => prevTaps + 1);
    }
  };

  // Reset game state to allow replay
  const handleReset = () => {
    setGameActive(false);
    setTaps(0);
    setTimeLeft(GAME_DURATION);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Tap Game</Text>

      <View style={styles.infoContainer}>
        <Text style={styles.timerText}>Time Left: {timeLeft}s</Text>
        <Text style={styles.scoreText}>Taps: {taps}</Text>
      </View>

      {/* 2 styles applied to same 1 component - 2nd style applied based on condition */}
      <TouchableOpacity 
        style={[styles.button, !gameActive && styles.startButton]} 
        onPress={handleTap}
        disabled={!gameActive && timeLeft === 0} // Disable after game ends
      >
        {/* Text display based on nested conditional statement */} 
        <Text style={styles.buttonText}>
          {gameActive ? "TAP" : (timeLeft === 0 ? "GAME OVER" : "START")}
        </Text>
      </TouchableOpacity>

      {timeLeft === 0 && (
        <TouchableOpacity style={styles.resetButton} onPress={handleReset}>
          <Text style={styles.buttonText}>RESET</Text>
        </TouchableOpacity>
      )}

      <Text style={styles.highScoresTitle}>High Scores</Text>
      <FlatList
        data={highScores}
        renderItem={renderScoreItem}
        keyExtractor={item => item.id}
        style={styles.list}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fefefe',
    padding: 20,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
    color: '#333',
  },
  infoContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    width: '80%',
    marginBottom: 20,
  },
  timerText: {
    fontSize: 20,
    color: '#d9534f',
  },
  scoreText: {
    fontSize: 20,
    color: '#337ab7',
  },
  button: {
    width: 200,
    height: 200,
    borderRadius: 100,
    backgroundColor: '#007bff',
    justifyContent: 'center',
    alignItems: 'center',
    elevation: 5,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
  },
  startButton: {
    backgroundColor: '#28a745',
  },
  buttonText: {
    fontSize: 24,
    color: '#fff',
    fontWeight: 'bold',
  },
  resetButton: {
    marginTop: 20,
    paddingHorizontal: 20,
    paddingVertical: 10,
    backgroundColor: '#dc3545',
    borderRadius: 5,
  },
  highScoresTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginTop: 30,
    marginBottom: 10,
    color: '#333',
  },
  list: {
    width: '100%',
  },
  scoreItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    padding: 10,
    backgroundColor: '#eee',
    borderRadius: 5,
    marginBottom: 5,
  },
  scoreItemText: {
    fontSize: 16,
    color: '#555',
  },
});

export default App;
```

### Walkthrough

There are now three effects, each with a single responsibility.

The first effect manages the interval. Its dependency array is `[gameActive, timeLeft]` — `taps` is gone. The body is the same as before, minus the `else if (timeLeft === 0)` branch, which has moved to the second effect. Now the effect does only one thing: when the game is running and there is time remaining, schedule an interval; when the effect re-runs or the component unmounts, clear whatever interval was scheduled. Because the effect no longer reacts to `taps`, a press no longer triggers a re-run, and the interval is allowed to complete its second. The countdown ticks at the correct rate regardless of how often the player presses.

The second effect detects the end of the game. Its dependency array is `[timeLeft]`. The body checks `if (timeLeft === 0 && gameActive)`. Both conditions are needed. `timeLeft === 0` says the countdown has finished. `gameActive` says a game was in progress. Without the second check, the effect would also fire on the initial mount, when `timeLeft` is `GAME_DURATION` — but that is not zero, so the first condition already filters that case out. The second condition guards against a subtler case: if `timeLeft` reaches zero, the player resets, and then `timeLeft` somehow returns to zero without a game being played, the effect should not fire again. In practice, the combination of "reset sets `timeLeft` back to `GAME_DURATION`" and "`handleTap` sets it back too" means `timeLeft` cannot return to zero without a new game having started, so the `gameActive` check is defensive. It is worth keeping, because it makes the effect's intent explicit: it fires when a game *ends*, not merely when the counter is zero.

Inside the branch, `setGameActive(false)` ends the game and `saveScore(taps)` writes the score. The order matters slightly: setting `gameActive` to `false` first means that if anything in `saveScore` triggers a render — and it does, via `setHighScores` — the render sees `gameActive` as `false` and does not re-trigger the interval effect. If the order were reversed, a render could happen with `gameActive` still `true` and `timeLeft` at zero, which would re-run the first effect, but the interval would not be scheduled because `timeLeft > 0` is false. The order is therefore not strictly required for correctness, but it is the clearer order.

The third effect loads the scores on mount. It is unchanged from Part A. Its dependency array is `[]`, and it runs once.

The remaining code — `loadScores`, `saveScore`, `handleTap`, `handleReset`, and the JSX — is the same as Part A, with two cosmetic differences. The storage key is now `'highScores'` rather than `'#highScores'`. The `Alert` import has been dropped, since it was unused. Comments in the JSX have been added at the two places that combine styles and nested ternaries, to make those spots easier to find.

### Why the correction works

The bug in Part A was caused by the effect re-running in response to a value it did not need to react to. In Part B, no effect depends on `taps`. The first effect depends on `gameActive` and `timeLeft`, both of which change on a predictable schedule tied to the game's progress. The second depends only on `timeLeft`, which changes exactly once per second during play and once at the end of the game. Neither depends on a value that changes on every press.

When the player presses the button, `taps` changes, the component re-renders, and the first and second effects both re-run. But re-running an effect whose dependencies have not changed is not what React does: React skips the effect's body and does not call its cleanup when its dependency array is unchanged between renders. So a press triggers a render, but the effects that would have torn down the interval do not fire. The interval keeps ticking.

This is worth stating as a general principle: an effect re-runs only when one of the values in its dependency array has changed since the previous render. The value being rendered can change, other state can change, the component can re-render for any number of reasons, and if the effect's own dependencies are unchanged, the effect does not run. This is why splitting the effect fixed the bug — the effect that manages the interval now only depends on values that change on the interval's own timescale.

### The contrast, side by side

The change, in full, is that Part A's single effect

```tsx
useEffect(() => {
  let timer: NodeJS.Timeout | null = null;

  if (gameActive && timeLeft > 0) {
    timer = setInterval(() => {
      setTimeLeft(prevTime => prevTime - 1);
    }, 1000);
  } else if (timeLeft === 0) {
    setGameActive(false);
    saveScore(taps);
  }

  return () => {
    if (timer) {
      clearInterval(timer);
    }
  };
}, [gameActive, timeLeft, taps]);
```

becomes two effects:

```tsx
useEffect(() => {
  let timer: NodeJS.Timeout | null = null;

  if (gameActive && timeLeft > 0) {
    timer = setInterval(() => {
      setTimeLeft(prevTime => prevTime - 1);
    }, 1000);
  } 

  return () => {
    if (timer) {
      clearInterval(timer);
    }
  };
}, [gameActive, timeLeft]);

useEffect(() => {
  if (timeLeft === 0 && gameActive) {
    setGameActive(false);
    saveScore(taps);
  }
}, [timeLeft]);
```

The `taps` dependency is removed from the timer effect and does not appear in the new game-end effect's array either. The new effect reads `taps` when it fires, and at that moment `taps` is the value from the most recent render — which, since the game is over, is the final score. The effect does not need `taps` in its dependency array because it does not need to react to changes in `taps`; it only needs to read its current value at the moment `timeLeft` reaches zero. React's rule allows this: an effect's dependency array lists what it should *react to*, not everything it reads. The `taps` in the body is read at effect-run time, and the effect runs when `timeLeft` reaches zero, which is exactly when the current value of `taps` is the final score.

It is worth being clear about the trade-off this introduces. The effect now reads a value — `taps` — that is not in its dependency array. This is legitimate when the effect's job is to read the value at a specific moment, not to react to it. It is illegitimate when the effect needs to stay in sync with the value over time. The distinction is subtle, and it is the source of Stage 4's issue: `saveScore` itself reads `highScores` from the enclosing scope, and by the time the game-end effect fires, that closure may be stale. Stage 4 addresses it.

### Key concepts

**One effect, one job.** Each effect should have a single responsibility, and its dependency array should list exactly the values whose change should cause that responsibility to re-run. When an effect's body does two things that react to different values, the dependency array is forced to be the union, and the effect re-runs when either thing changes. Splitting the effect lets each half react only to what it needs. This is the fix for the Part A bug, and it is a general pattern.

**Effects react to dependencies, not to renders.** A re-render alone does not re-run an effect. React compares the new dependency array to the previous one, element by element, and only re-runs the effect if some element differs. This is why removing `taps` from the array stopped the interval from being torn down on every press: the press changed `taps` and caused a render, but neither of the effect's remaining dependencies had changed, so the effect did not re-run.

**Reading a value without depending on it.** An effect can read a value from the enclosing render without listing it as a dependency, if the effect does not need to re-run when that value changes. The `taps` read inside the game-end effect is an example: the effect needs the current `taps` at the moment the game ends, not at every change. This pattern is legitimate but requires care, because the value read is captured from the render in which the effect last ran, and if that render is old, the value is stale. Stage 4 explores this in detail.

**Why multiple `useEffect` hooks are good practice.** React does not restrict how many effects a component has. Each is an independent "reaction" to a specific change in state or props. Splitting logic across effects makes each one easier to read, easier to reason about, and easier to change without affecting the others. Three effects, each with a clear name in a comment — "timer logic", "detects when the game ends", "load saved scores" — are easier to maintain than one effect that does all three. The caution is that too many effects, or effects whose responsibilities overlap, can become as hard to follow as one large effect; the discipline is naming and separation, not the count itself.

### Revision notes and gotchas

- **The `taps` dependency is removed, not silenced.** The lint warning that flagged it in Part A will return once the dependency is removed. This is expected. The warning is a heuristic: it flags values that the effect reads and that are not in its dependencies, on the assumption that the effect should react to them. When the assumption is wrong — as here — the warning is a false positive. Silencing it with a comment is acceptable; understanding why it is firing, and confirming that the effect genuinely should not react to `taps`, is the more important step.
- **The game-end effect's `gameActive` check is defensive.** In the current code, `timeLeft` cannot reach zero without a game having started, because both `handleTap` and `handleReset` set it back to `GAME_DURATION`. The `gameActive` check is therefore redundant in practice, but it documents the effect's intent: it fires when a game *ends*, not merely when the counter hits zero. Keeping it is reasonable; removing it would make the effect's purpose less explicit.
- **`saveScore` is called from the game-end effect and reads `highScores` from the enclosing render.** Because the game-end effect's dependency array is `[timeLeft]`, the effect runs on each change to `timeLeft`. On the run that matters — the one where `timeLeft` is zero — the effect captures `highScores` from whatever render most recently ran the effect. That is not necessarily the render in which the score was finalised. In practice the difference is small: the render that set `timeLeft` to zero is the one whose closure will be used, and `highScores` has not changed since the previous game ended. But it is a stale closure, and Stage 4 addresses it.
- **The storage key is now `'highScores'`.** Part A used `'#highScores'`. The `#` had no special meaning and has been dropped. If you have run Part A's code and stored scores under `'#highScores'`, Part B will not find them; they are still in storage under the old key. This is worth knowing when transitioning between versions during development.
- **`Alert` is no longer imported.** It was unused in Part A and has been removed. If you intend to use it later, it needs to come back.

---

## Stage 4 — Stale Closures and the Functional Updater

### What this stage introduces

Stage 3 Part B fixed the timer bug by removing `taps` from the timer effect's dependencies and splitting the game-end detection into its own effect. That correction worked, and the countdown now ticks at the correct rate regardless of how often the player presses the button. But it left a second issue in place, quieter than the first: `saveScore` reads `highScores` from the render in which it was created, and by the time the game-end effect calls it, that render may no longer be the most recent one.

This stage addresses that issue. It introduces two related ideas. The first is the **stale closure** — a function that captures a value from an older render and uses that older value when it runs. The second is the **functional updater** for `useState`, which is the standard fix: instead of reading the current state into a variable and computing the next state from it, the setter is given a function that receives the previous state from React and returns the next state. React is responsible for supplying a current value, so the function never has to rely on a captured one.

The change is small — a few lines in `saveScore` — but it is a change of shape, not just of detail. The score-saving function moves from "read `highScores`, compute `top5Scores`, call `setHighScores(top5Scores)`" to "call `setHighScores` with a function that computes the next state from the previous state, and write to storage from inside that function". The write to `AsyncStorage` moves with it, because the value to be written is now computed inside the updater rather than outside it.

This stage also discusses why this class of bug is easy to miss. The symptom is intermittent, it depends on timing, and it often does not appear during casual testing. Naming the pattern — "this function reads a value from a render that may be old" — is the defence.

---

### The bug, in detail

The relevant code from Stage 3 Part B is:

```tsx
// 2 Effect: Detects when the game ends and saves the score
useEffect(() => {
  if (timeLeft === 0 && gameActive) {
    setGameActive(false);        // End the game
    saveScore(taps);             // Save the score once
  }
}, [timeLeft]); // Only triggers when timeLeft changes
```

and:

```tsx
const saveScore = async (newTaps: number) => {
  const now = new Date();
  const newScore: Score = {
    id: now.getTime().toString(),
    taps: newTaps,
    date: now.toLocaleString(),
  };

  const updatedScores = [...highScores, newScore];
  const top5Scores = updatedScores.sort((a, b) => b.taps - a.taps).slice(0, 5);
  setHighScores(top5Scores);

  try {
    await AsyncStorage.setItem('#highScores', JSON.stringify(top5Scores));
  } catch (e) {
    console.error("Failed to save score", e);
  }
};
```

`saveScore` reads `highScores` from the surrounding scope. That `highScores` is not "the current high scores" in a general sense; it is "the value of `highScores` in the render during which this particular instance of `saveScore` was created". Each render of `App` creates a new `saveScore` function, closing over the `highScores` value that was current during that render. When the game-end effect runs and calls `saveScore`, it calls whichever instance of `saveScore` was captured by that effect's closure.

The game-end effect's dependency array is `[timeLeft]`. The effect re-runs when `timeLeft` changes. The `saveScore` it calls is the one from the render in which the effect last re-ran. Normally that render is the one that produced the new `timeLeft`, which is also the render in which `highScores` has its most recent value — because nothing else has changed `highScores` between the previous game's end and this one's. So in practice, the closure is almost always fresh.

But "almost always" is not "always". There are scenarios where the closure can be stale:

- If `setHighScores` is called from somewhere else between the game-end effect's last run and the moment it fires — for example, if a future feature added a "clear scores" button, or if `loadScores` resolved after the game started — then `highScores` in the render that produced the current `timeLeft` might not be the most recent `highScores`.
- If a single render queue contains both a `setHighScores` from a previous `saveScore` call and the `setTimeLeft` that triggers the game-end effect, the order in which they are processed can affect which `highScores` value the effect sees.

The bug is subtle because it depends on the order of updates in the render queue, and because the current code has only one place that writes to `highScores`. It is the kind of bug that is easier to prevent than to reproduce. The prevention is the functional updater.

There is a second, related issue. When `saveScore` is called from inside the effect, the `AsyncStorage.setItem` writes `top5Scores` — a value computed outside the updater. Even if the state update is correct, the value written to storage is derived from `updatedScores`, which was derived from the captured `highScores`. So a stale read produces both a stale state update and a stale write. Moving the computation inside the updater fixes both, because the array React provides to the updater is the freshest one available.

---

### The code

```tsx
import { useState, useEffect } from 'react';
import { View, Text, StyleSheet, TouchableOpacity, FlatList } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

const GAME_DURATION = 5; // Game duration in seconds

// Define the type for each high score entry
type Score = {
  id: string;
  taps: number;
  date: string;
};

// User-defined component to render each score item for the FlatList
function renderScoreItem({ item }: { item: Score }) {
  return (
    <View style={styles.scoreItem}>
      <Text style={styles.scoreItemText}>Taps: {item.taps}</Text>
      <Text style={styles.scoreItemText}>Date: {item.date}</Text>
    </View>
  );
}


function App() {
  const [taps, setTaps] = useState<number>(0); // Number of user taps
  const [timeLeft, setTimeLeft] = useState<number>(GAME_DURATION); // Remaining game time
  const [gameActive, setGameActive] = useState<boolean>(false); // Is the game currently running?
  const [highScores, setHighScores] = useState<Score[]>([]); // List of top 5 scores

  // 1 Effect: Timer logic - runs every second while game is active
  useEffect(() => {
    let timer: NodeJS.Timeout | null = null;

    if (gameActive && timeLeft > 0) {
      timer = setInterval(() => {
        setTimeLeft(prevTime => prevTime - 1);
      }, 1000);
    } 

    return () => {
      if (timer) {
        clearInterval(timer);
      }
    };
  }, [gameActive, timeLeft]); 

  // 2 Effect: Detects when the game ends and saves the score
  useEffect(() => {
    if (timeLeft === 0 && gameActive) {
      setGameActive(false);        // End the game
      saveScore(taps);             // Save the score once
    }
  }, [timeLeft]); // Only triggers when timeLeft changes

  // 3 Effect: Load saved high scores from AsyncStorage on first render
  useEffect(() => {
    loadScores();
  }, []);

  // Loads scores from persistent storage
  const loadScores = async () => {
    try {
      const storedScores = await AsyncStorage.getItem('#highScores');
      if (storedScores !== null) {
        setHighScores(JSON.parse(storedScores)); // Load and update state
      }
    } catch (e) {
      console.error("Failed to load scores", e);
    }
  };

  // Saves the current score to AsyncStorage and updates the top 5 list
  const saveScore = async (newTaps: number) => {
    const now = new Date();
    const newScore: Score = {
      id: now.getTime().toString(), // Use timestamp as unique ID
      taps: newTaps,
      date: now.toLocaleString(),   // Save human-readable date
    };

    // Use the functional updater so the new array is built from the
    // freshest highScores React has, not from a value captured in an
    // older render.
    setHighScores(prevScores => {
      const updatedScores = [...prevScores, newScore];
      const top5Scores = updatedScores.sort((a, b) => b.taps - a.taps).slice(0, 5);

      // Write to AsyncStorage from inside the updater so the value
      // persisted matches the value that was just committed to state.
      AsyncStorage.setItem('#highScores', JSON.stringify(top5Scores))
        .catch(e => console.error("Failed to save score", e));

      return top5Scores;
    });
  };

  // Called when user taps the main game button
  const handleTap = () => {
    if (!gameActive) {
      // Start new game
      setGameActive(true);
      setTaps(1);
      setTimeLeft(GAME_DURATION);
    } else {
      // Increment tap count
      setTaps(prevTaps => prevTaps + 1);
    }
  };

  // Reset game state to allow replay
  const handleReset = () => {
    setGameActive(false);
    setTaps(0);
    setTimeLeft(GAME_DURATION);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Tap Game</Text>

      <View style={styles.infoContainer}>
        <Text style={styles.timerText}>Time Left: {timeLeft}s</Text>
        <Text style={styles.scoreText}>Taps: {taps}</Text>
      </View>

      {/* 2 styles applied to same 1 component - 2nd style applied based on condition */}
      <TouchableOpacity 
        style={[styles.button, !gameActive && styles.startButton]} 
        onPress={handleTap}
        disabled={!gameActive && timeLeft === 0} // Disable after game ends
      >
        {/* Text display based on nested conditional statement */} 
        <Text style={styles.buttonText}>
          {gameActive ? "TAP" : (timeLeft === 0 ? "GAME OVER" : "START")}
        </Text>
      </TouchableOpacity>

      {timeLeft === 0 && (
        <TouchableOpacity style={styles.resetButton} onPress={handleReset}>
          <Text style={styles.buttonText}>RESET</Text>
        </TouchableOpacity>
      )}

      <Text style={styles.highScoresTitle}>High Scores</Text>
      <FlatList
        data={highScores}
        renderItem={renderScoreItem}
        keyExtractor={item => item.id}
        style={styles.list}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fefefe',
    padding: 20,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
    color: '#333',
  },
  infoContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    width: '80%',
    marginBottom: 20,
  },
  timerText: {
    fontSize: 20,
    color: '#d9534f',
  },
  scoreText: {
    fontSize: 20,
    color: '#337ab7',
  },
  button: {
    width: 200,
    height: 200,
    borderRadius: 100,
    backgroundColor: '#007bff',
    justifyContent: 'center',
    alignItems: 'center',
    elevation: 5,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
  },
  startButton: {
    backgroundColor: '#28a745',
  },
  buttonText: {
    fontSize: 24,
    color: '#fff',
    fontWeight: 'bold',
  },
  resetButton: {
    marginTop: 20,
    paddingHorizontal: 20,
    paddingVertical: 10,
    backgroundColor: '#dc3545',
    borderRadius: 5,
  },
  highScoresTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginTop: 30,
    marginBottom: 10,
    color: '#333',
  },
  list: {
    width: '100%',
  },
  scoreItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    padding: 10,
    backgroundColor: '#eee',
    borderRadius: 5,
    marginBottom: 5,
  },
  scoreItemText: {
    fontSize: 16,
    color: '#555',
  },
});

export default App;
```

---

### Walkthrough

Only `saveScore` has changed. The rest of the file — imports, types, the three effects, `loadScores`, `handleTap`, `handleReset`, the JSX, and the stylesheet — is the same as Stage 3 Part B.

The new `saveScore` still builds a `newScore` object from the current time, and still takes `newTaps` as its argument. What changes is how the next state is computed and how the storage write is performed.

In Stage 3 Part B, the computation was:

```tsx
const updatedScores = [...highScores, newScore];
const top5Scores = updatedScores.sort((a, b) => b.taps - a.taps).slice(0, 5);
setHighScores(top5Scores);
await AsyncStorage.setItem('#highScores', JSON.stringify(top5Scores));
```

This reads `highScores` from the enclosing scope, computes the new array outside React, sets it, and then writes the same array to storage. The read is the problem: `highScores` is whatever value was current in the render that created this instance of `saveScore`.

The new version is:

```tsx
setHighScores(prevScores => {
  const updatedScores = [...prevScores, newScore];
  const top5Scores = updatedScores.sort((a, b) => b.taps - a.taps).slice(0, 5);

  AsyncStorage.setItem('highScores', JSON.stringify(top5Scores))
    .catch(e => console.error("Failed to save score", e));

  return top5Scores;
});
```

Reading it in order: `setHighScores` is called with a function. That function receives `prevScores` — which React supplies, and which is the most recent value of `highScores` at the time the update is processed. Inside, `updatedScores` is built from `prevScores` and the new score. The sort and slice produce `top5Scores`. The storage write is called with `top5Scores`, and the function returns `top5Scores`, which React uses as the next state.

The storage write has moved inside the updater. This is deliberate. The value to be written must be the same value that was just committed to state; if the write were performed outside, it would be computing its own array from a possibly different starting point. By placing the write inside the updater, the array that is written and the array that is committed are the same object, produced by the same computation.

Note that the storage write is no longer `await`ed. The `setHighScores` updater function is synchronous — React calls it, takes the return value, and moves on. An `async` updater function would return a promise, which is not a valid next-state value. So the write is fired and its result is handled with `.catch(...)` instead of `try`/`await`. This is a small but necessary change: the function passed to `setHighScores` cannot itself be `async`, because `setHighScores` expects a synchronous function returning the next state.

There is a trade-off here worth naming. The storage write is no longer awaited by `saveScore`, which means `saveScore` returns before the write completes. In the previous version, `await AsyncStorage.setItem(...)` meant the function only returned once the write was done. In the new version, the function returns as soon as the state update is scheduled. For this application, that is fine — the user is at the game-over screen and nothing depends on the write having completed. If something *did* depend on it — a subsequent read, a confirmation message — the code would need to be restructured so that the write result could be observed. The current shape is the simpler one and is correct for the game as it stands.

The updater also has a subtle property that is worth stating: React may call the function more than once. In development mode with `StrictMode`, React intentionally double-invokes updater functions to help catch impure updaters. This means the `AsyncStorage.setItem` call inside the updater may fire twice in development. Writing to the same key twice with the same value is harmless, but it is worth knowing that updater functions are expected to be **pure** — they should compute and return, and avoid side effects. The storage write here is a side effect, and it is placed inside the updater for a practical reason: to ensure the written value matches the committed value. A purer design would compute `top5Scores` inside the updater, store it in a variable outside, and write to storage after `setHighScores` returns — but that reintroduces the very problem this stage is solving, because the outer variable would have to be updated synchronously from inside the updater, which is fragile. The trade-off is between purity and correctness of the written value; here, correctness of the written value is the priority, and the double-write in StrictMode is a known and accepted consequence.

---

### The contrast, side by side

Stage 3 Part B's `saveScore`:

```tsx
const saveScore = async (newTaps: number) => {
  const now = new Date();
  const newScore: Score = {
    id: now.getTime().toString(),
    taps: newTaps,
    date: now.toLocaleString(),
  };

  const updatedScores = [...highScores, newScore];
  const top5Scores = updatedScores.sort((a, b) => b.taps - a.taps).slice(0, 5);
  setHighScores(top5Scores);

  try {
    await AsyncStorage.setItem('#highScores', JSON.stringify(top5Scores));
  } catch (e) {
    console.error("Failed to save score", e);
  }
};
```

Stage 4's `saveScore`:

```tsx
const saveScore = async (newTaps: number) => {
  const now = new Date();
  const newScore: Score = {
    id: now.getTime().toString(),
    taps: newTaps,
    date: now.toLocaleString(),
  };

  setHighScores(prevScores => {
    const updatedScores = [...prevScores, newScore];
    const top5Scores = updatedScores.sort((a, b) => b.taps - a.taps).slice(0, 5);

    AsyncStorage.setItem('#highScores', JSON.stringify(top5Scores))
      .catch(e => console.error("Failed to save score", e));

    return top5Scores;
  });
};
```

The change has four parts:

1. The read of `highScores` from the enclosing scope is replaced by `prevScores`, which React supplies.
2. The computation moves inside the updater function.
3. The storage write moves inside the updater function, so the written value matches the committed value.
4. The `await` and `try`/`catch` are replaced by `.catch(...)`, because the updater function must be synchronous.

---

### Key concepts

**Stale closures.** A closure is a function that captures variables from the scope in which it was defined. In React, every render creates new instances of every function declared inside the component, and each instance captures the values that were current in that render. A function that is called later — from an effect, from a callback, from an event handler — uses the values from *its* render, not from the render in which it is called. If the value has changed since, the function is said to have captured a stale closure. The pattern is not a bug in itself; it becomes a bug when the function relies on the captured value being current. The fix is either to include the value in the dependencies of whatever keeps the function alive (so the function is re-created and re-captured when the value changes), or to avoid reading the value directly and let React supply it — which is what the functional updater does.

**The functional updater.** `setState(prev => next)` passes a function to the setter instead of a value. React calls the function with the most recent state and uses the return value as the next state. The function does not have to read the state variable from the enclosing scope, so it cannot capture a stale value. It is the correct form whenever the next state depends on the previous state. This is true for the obvious cases — incrementing a counter, appending to an array, toggling a boolean — and it is true regardless of whether the code is currently showing signs of the stale-closure bug. Using the functional updater is a defensive habit; the direct form works until it does not, and when it does not, the failure is timing-dependent and hard to reproduce.

**Where to put side effects when using an updater.** An updater function should ideally be pure: given the previous state, return the next state, and do nothing else. Side effects — writes to storage, network calls, logging — belong outside. In practice, though, there is a case for putting the side effect inside the updater: when the effect's input is the new state, and the new state is only available inside the updater. Writing to storage with the value that was just committed is such a case. The cost is that the updater is no longer pure, and in React's development mode with `StrictMode`, the updater may be called twice, causing the side effect to fire twice. The benefit is that the persisted value always matches the committed value. The choice is a trade-off, and the priority in this game is correctness of the persisted value.

**Sync functions and async work.** The function passed to `setHighScores` must return the next state synchronously. It cannot be `async`, because an `async` function returns a promise, and a promise is not a valid state value. Any asynchronous work that needs to happen around the state update — such as the storage write — must be expressed without `await` inside the updater. In this code, that means `.catch(...)` instead of `try`/`await`. The result is that `saveScore` no longer waits for the write to complete; it returns as soon as the state update is scheduled. For this application, that is fine; for code where the write's result must be observed, a different structure would be needed.

---

### Revision notes and gotchas

- **The bug this stage fixes is timing-dependent, and may not reproduce in casual testing.** If you run the Stage 3 Part B code and play a few games, it will almost certainly behave correctly. The stale-closure issue surfaces only when `highScores` changes between the render that produces the current `timeLeft` and the moment the game-end effect fires. With the current code, that window is narrow and the value rarely changes within it. The fix is worth making anyway, because it removes the dependency on that narrowness. It also prepares the code for any future change that might make the window wider — for example, a second writer to `highScores`, or a `loadScores` that resolves late.
- **The updater function should be pure, and this one is not.** The `AsyncStorage.setItem` call inside the updater is a side effect. React's documentation recommends that updaters be pure, and in development mode with `StrictMode`, React will call the updater twice to help detect impure ones. In this code, that means the storage write may fire twice per game-end. Writing the same value to the same key twice is harmless, and the write is idempotent, so the practical cost is one extra write in development. A purer alternative is to compute `top5Scores` inside the updater, return it, and perform the storage write outside — but the value to write is only known after the updater runs, and the updater's return value is consumed by React, not by the caller. The two common patterns are to move the write inside the updater (as here) or to perform the write in a separate effect that watches `highScores`. The second is purer, and it is worth considering for code where the write is more expensive or less idempotent than a single `AsyncStorage.setItem`.
- **`saveScore` is now `async` but does not `await` anything.** The `async` keyword is retained from the earlier version, but the function no longer contains an `await`. It could be simplified to a non-async function. It is left as `async` here to minimise the diff from Stage 3 Part B and to leave the door open for future asynchronous work. Removing the keyword is a small cleanup.
- **The `prevScores` parameter shadows the outer `highScores` state variable's role, but does not conflict with it.** `prevScores` is the name given to the value React supplies to the updater. It is not the same variable as `highScores`; it is the current value at the moment the updater runs. Naming it `prevScores` rather than `highScores` makes it clear that this is the value being built upon, not the value from the enclosing render. This is a deliberate naming choice and worth keeping.
- **The storage write is no longer awaited.** If the app is closed immediately after a game ends, the write may not have completed. `AsyncStorage` writes are typically fast, and on modern devices the write will almost always finish before the app is suspended, but it is not guaranteed. For a game where the score is not critical, this is acceptable. For data where loss is unacceptable, the write should be awaited, which requires a structure where the awaited function is not inside the updater — for example, by computing the next array outside the updater using a ref, or by writing from a separate effect. Neither is needed here.
- **The pattern generalises beyond this function.** Anywhere a setter is called with a value computed from the current state, the functional updater is the safer form. `setTaps(taps + 1)` in Stage 1 had the same latent issue, even though it did not manifest. The codebase switched to `setTaps(prev => prev + 1)` in Stage 2 and never went back. `saveScore` was the last place still using the direct form; this stage brings it in line.

---

### Closing note

The project now has a complete and correct shape for a small game: state for the game's condition, effects for the timer and for loading and detecting the end of the game, persistent storage for the scores, and a list to display them. The two bugs that were found along the way — the interrupted timer in Stage 3 and the stale closure in `saveScore` — are not unusual. They are the kind of issue that appears whenever state, effects, and asynchronous work are combined, and the fixes are the standard ones: split effects by responsibility, and use functional updaters when the next state depends on the previous one.

The more important lesson is not the specific fixes but the pattern of reasoning behind them. When an effect behaves unexpectedly, the first question is which values in its dependency array have changed and why. When a value read inside a function seems wrong, the first question is which render that function was created in. Both questions lead to the same answer: React's model is that every render has its own values, and functions created in one render see that render's values, not the latest ones. The functional updater is the tool that lets a function operate on the latest values without needing to be re-created. Once that model is internalised, most of the surprising behaviour in React stops being surprising.

---
