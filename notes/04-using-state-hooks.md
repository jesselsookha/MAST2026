# MAST5112: Mobile App Scripting – Class Notes

## 04 - Using State Hooks

---

## Introduction

In the previous lesson, we saw our first example of an interactive React Native application. We used a TextInput to capture user input, displayed that input using Text, and used a Button to trigger an action.

At the centre of that example was a piece of code that looked like this:

```tsx
const [userInput, setUserInput] = useState<string>('');
```

This code uses a React feature called a Hook. Specifically, it uses the useState Hook to create and manage state.

In this lesson, we will explore state in detail. We will learn:

- What state is and why it matters
- How useState works
- The difference between state variables and regular variables
- How to use useState with different data types
- Common patterns and common mistakes

By the end of this lesson, you will understand why state is essential for building interactive applications and how to use it effectively.

---

## What Is State?

State is information that belongs to a component and can change while the application is running.

Consider a simple counter application. The counter starts at 0. When the user presses a button, the counter increases to 1, then 2, and so on.

The current number is information that changes over time. That information is state.

### State vs Regular Variables

In a regular JavaScript or TypeScript program, we might store a number in a variable:

```tsx
let count = 0;
count = count + 1;
```

In a React Native application, this approach does not work for values that need to appear on the screen. We will see why shortly.

---

## What Is a Render?

Before we go further, we need to understand one important concept: rendering.

A render is when React Native draws the user interface on the screen.

This happens every time the component function runs and returns JSX.

When you first open your application, the component runs and the interface is rendered.

When something changes, the component runs again and the interface is re-rendered.

Re-rendering is how the screen updates to show new information.

---

## Why Regular Variables Do Not Work

Consider this code:

```tsx
export default function App() {
  let count = 0;

  const increment = () => {
    count = count + 1;
    console.log(count);
  };

  return (
    <View>
      <Text>{count}</Text>
      <Button title="Increment" onPress={increment} />
    </View>
  );
}
```

At first glance, this seems like it should work. When the button is pressed, count increases. The console log shows the new value.

But the screen does not update.

Why?

Every time the App component re-renders, the function runs from the beginning.

The line:

```tsx
let count = 0;
```

runs again, and count is reset to 0.

React has no way of knowing that the value changed, because a regular variable does not tell React to re-render.

The screen shows 0, even though the console shows the correct increasing values.

---

## How useState Solves This Problem

The useState Hook solves both problems.

First, useState remembers the value between renders. The value is stored in React's internal state system, not in a regular variable.

Second, when we call the update function, React knows that the state has changed and automatically re-renders the component.

### The useState Pattern

```tsx
const [value, setValue] = useState(initialValue);
```

This pattern has three parts:

- value: The current state value
- setValue: The function used to update the value
- initialValue: The starting value for the state

### A Working Counter

Here is the counter example rewritten to use state:

```tsx
import { useState } from 'react';
import { Button, StyleSheet, Text, View } from 'react-native';

export default function App() {
  const [count, setCount] = useState<number>(0);

  const increment = () => {
    setCount(count + 1);
  };

  return (
    <View style={styles.container}>
      <Text>{count}</Text>
      <Button title="Increment" onPress={increment} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});
```

This works correctly. When the button is pressed:

```text
Button pressed
      ↓
increment() runs
      ↓
setCount(count + 1) is called
      ↓
React updates the state
      ↓
Component re-renders
      ↓
New count is displayed
```

---

## State vs Regular Variable: A Side-by-Side Comparison

| Aspect              | Regular Variable | State Variable |
| ------------------- | ----------------- | -------------- |
| Remembers value     | No (resets on render) | Yes |
| Triggers re-render  | No | Yes |
| Screen updates      | No | Yes |
| Syntax              | let count = 0 | const [count, setCount] = useState(0) |
| Update method       | count = count + 1 | setCount(count + 1) |

### Key Rule

> If a value needs to appear on the screen and can change while the application is running, it should be state.

Do not use regular variables for values that the user needs to see change.

---

## The Square Bracket Syntax

You may have noticed the unusual syntax:

```tsx
const [count, setCount] = useState<number>(0);
```

The square brackets are not an array. They are a JavaScript feature called destructuring assignment.

useState returns an array with two items:

1. The current state value
2. The function to update it

Destructuring assigns these two items to the variables count and setCount.

This is equivalent to:

```tsx
const result = useState(0);
const count = result[0];
const setCount = result[1];
```

But the destructuring syntax is shorter and more readable.

---

## Typing useState with TypeScript

Because we are using TypeScript, we should specify the type of our state values.

### Basic Types

```tsx
const [name, setName] = useState<string>('');         // String
const [count, setCount] = useState<number>(0);        // Number
const [isVisible, setIsVisible] = useState<boolean>(true);  // Boolean
```

### Type Inference

TypeScript can often infer the type from the initial value.

```tsx
const [name, setName] = useState('');         // TypeScript infers string
const [count, setCount] = useState(0);        // TypeScript infers number
```

In simple examples, type inference works well. In more complex examples, explicit typing is clearer.

### When to Use Explicit Typing

Use explicit typing when:

- The initial value is null or undefined
- The state will hold a complex object
- You want to be very clear about the expected type
- You are working in a larger codebase

---

## Common useState Patterns

### Pattern 1: Counter

```tsx
const [count, setCount] = useState<number>(0);

<Button title="Increase" onPress={() => setCount(count + 1)} />
<Button title="Decrease" onPress={() => setCount(count - 1)} />
```

### Pattern 2: Toggle

```tsx
const [isVisible, setIsVisible] = useState<boolean>(true);

<Button title="Toggle" onPress={() => setIsVisible(!isVisible)} />
{isVisible && <Text>You can see me!</Text>}
```

### Pattern 3: Text Input

```tsx
const [inputText, setInputText] = useState<string>('');

<TextInput
  placeholder="Type here"
  value={inputText}
  onChangeText={setInputText}
/>
<Text>You typed: {inputText}</Text>
```

### Pattern 4: Conditional Display

```tsx
const [mode, setMode] = useState<string>('day');

<Button
  title="Switch Mode"
  onPress={() => setMode(mode === 'day' ? 'night' : 'day')}
/>
<Text>Current Mode: {mode}</Text>
```

---

## Common Mistakes to Avoid

### Mistake 1: Using a Regular Variable Instead of State

```tsx
// Wrong
let count = 0;
count = count + 1;
```

The screen will not update.

### Mistake 2: Directly Changing the State Value

```tsx
// Wrong
count = count + 1;
```

State is read-only. You must use the update function.

```tsx
// Correct
setCount(count + 1);
```

### Mistake 3: Using State Outside a Component

```tsx
// Wrong - outside component
const [count, setCount] = useState(0);

export default function App() {
  // ...
}
```

useState can only be used inside a functional component.

### Mistake 4: Updating State with the Same Value

If you call setCount(count) when count has not changed, React may not re-render. This is usually not a problem, but it is worth knowing.

---

## Understanding State Updates

When you call the update function, React schedules a re-render.

The update is not immediate. React batches updates for performance.

This means:

```tsx
setCount(count + 1);
setCount(count + 1);
```

will only increment once, because both updates use the same count value.

If you need to update based on the previous state, use the function form:

```tsx
setCount(prevCount => prevCount + 1);
setCount(prevCount => prevCount + 1);
```

This will increment twice.

For most beginner applications, the simple form is sufficient.

---

## Multiple State Variables

A component can have multiple state variables.

```tsx
const [name, setName] = useState<string>('');
const [age, setAge] = useState<number>(0);
const [isStudent, setIsStudent] = useState<boolean>(false);
```

Each state variable is independent.

You can also use a single object to hold multiple values, but multiple state variables are simpler for beginners.

---

## State and Components

State belongs to the component that creates it.

If you have two components, each has its own state.

```tsx
function Counter() {
  const [count, setCount] = useState<number>(0);
  // This state belongs to this Counter component
}

function AnotherCounter() {
  const [count, setCount] = useState<number>(0);
  // This is a different state, independent of the first
}
```

Each Counter component maintains its own count.

---

## Hands-On Practice Exercises

### Exercise 1: Echo Input

Create an application that displays the user's input as they type.

```tsx
const [name, setName] = useState<string>('');

<TextInput onChangeText={setName} value={name} />
<Text>Hello {name}!</Text>
```

### Exercise 2: Show and Hide

Create an application that shows and hides a text when a button is pressed.

```tsx
const [visible, setVisible] = useState<boolean>(true);

<Button title="Toggle" onPress={() => setVisible(!visible)} />
{visible && <Text>You can see me!</Text>}
```

### Exercise 3: Mode Switch

Create an application that switches between day and night modes.

```tsx
const [mode, setMode] = useState<string>('day');

<Button title="Switch Mode" onPress={() => setMode(mode === 'day' ? 'night' : 'day')} />
<Text>Current Mode: {mode}</Text>
```

### Exercise 4: Combined Example

Create an application that combines multiple state concepts.

```tsx
import { useState } from 'react';
import { Button, StyleSheet, Text, TextInput, View } from 'react-native';

export default function App() {
  const [name, setName] = useState<string>('');
  const [count, setCount] = useState<number>(0);
  const [isVisible, setIsVisible] = useState<boolean>(true);

  const reset = () => {
    setName('');
    setCount(0);
    setIsVisible(true);
  };

  return (
    <View style={styles.container}>
      <TextInput
        placeholder="Enter your name"
        value={name}
        onChangeText={setName}
        style={styles.input}
      />
      <Text>Hello, {name || 'stranger'}</Text>

      <Text>Count: {count}</Text>
      <Button title="Increment" onPress={() => setCount(count + 1)} />

      <Button title="Toggle Greeting" onPress={() => setIsVisible(!isVisible)} />
      {isVisible && <Text>The greeting is visible</Text>}

      <Button title="Reset All" onPress={reset} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 16,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginVertical: 12,
    width: '80%',
  },
});
```

This example combines:

- Text input state (name)
- Counter state (count)
- Toggle state (isVisible)
- A function that resets all state values

---

## Summary

| Concept           | Description                                    | Example                |
| ----------------- | ---------------------------------------------- | ---------------------- |
| useState          | Creates a state variable                       | useState<number>(0)    |
| State value       | The current value                              | count                  |
| Update function   | Updates the state and triggers a re-render     | setCount(count + 1)    |
| Regular variable  | Does not persist across renders                | let count = 0          |
| Render            | When React Native draws the UI on the screen   | Component function runs |
| Re-render         | When the screen updates to show new state      | After setCount is called |
| TypeScript typing | Specifies the data type of state               | useState<string>('')   |

---

## Key Takeaways

1. Use state for values that change and need to appear on the screen
2. Do not use regular variables for values that need to trigger screen updates
3. useState returns a pair: the current value and an update function
4. Always use the update function to change state
5. State persists across renders
6. Changing state triggers a re-render
7. TypeScript can help enforce correct data types in state

---

## Next Steps

You now understand how to use useState to manage state in your React Native applications.

In the next lesson, we will explore layout in more detail. We will learn how Flexbox works and how to create more complex and responsive layouts.

---
