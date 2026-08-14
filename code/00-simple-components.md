# 00 Simple Components

This is an introductory React Native application designed to establish an understanding of some of the basic building blocks used to create a mobile interface.

Rather than trying to build a complete application immediately, we will work with a small program and examine how its individual parts work together.

The application introduces:

* Components such as `View`, `Text`, and `Button`
* State using the `useState` Hook
* Functions that respond to user interaction
* Displaying JavaScript values inside JSX
* Basic styling using `StyleSheet`

The aim is not simply to make the program work. The aim is to become comfortable reading the code and explaining what each part of the program is doing.

---

## Stage 1: Project Setup

The first stage establishes the basic structure of the React Native application.

At this point, there is no user interaction and no application data. We are simply creating a `View` that will act as the main container for the content on the screen.

```tsx
import { Text, View, StyleSheet } from 'react-native'; 

export default function App() { 
  return ( 
    <View style={styles.container}> 
    
    </View> ); 
} 

const styles = StyleSheet.create({ 
  container: { 
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  }, 
});
```

The `View` provides the main area in which we will place our interface.

The `StyleSheet` defines how this container should appear. For now, the important point is that React Native separates the structure of the interface from the styles that control its appearance.

---

We will now build on this basic structure and introduce several of the fundamental building blocks that you will encounter throughout React Native development:

* `View` as the main container for your content
* How buttons work and how user actions trigger changes
* The concept of **state** and how your app updates and re-renders
* How text is displayed and connected to state using `{}`

The goal is to become comfortable reading and explaining React Native code.

The application we are going to build is intentionally small. This allows us to concentrate on the individual concepts rather than becoming distracted by a large amount of application code.

---

```tsx
import { useState } from 'react';
import { Button, StyleSheet, Text, View } from 'react-native';

export default function App() {
  const [count,setCount] = useState<number>(0);

  //const handlePress = () => {console.log('you clicked the button')};
  const handlePress = () => {setCount(count + 1)}; 

  return (
    <View style={styles.container}>
      <Text>COMPONENTS V1</Text>
      <Text>{count}</Text>
      <Button 
        title="CLICK HERE"
        onPress={handlePress}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});

```

---

## Overview: What This App Does

This is a basic React Native application that:

* Displays a title (`COMPONENTS V1`)
* Shows a number on the screen
* Increases the number each time the button is clicked

Although the application is small, it introduces several important ideas that will be used repeatedly in later React Native programs.

The application demonstrates a simple flow:

**User action → function executes → state changes → interface updates**

Understanding this relationship is important because React Native applications are interactive. The screen is not simply displayed once; it can respond to actions performed by the user.

---

## Key Pieces Explained

### 1. `useState` Hook

```tsx
const [count, setCount] = useState<number>(0);
```

This line creates a piece of **state** for the application.

* `count` is the state variable.
* `setCount` is the function used to change the value of `count`.
* The initial value of `count` is `0`.
* When the state changes, React re-renders the relevant part of the interface so that the updated value can be displayed.

The important idea is that state allows a component to keep track of information that can change while the application is running.

In this example, the information being tracked is simply a number.

The `<Text>` component later displays this value:

```tsx
<Text>{count}</Text>
```

When `count` changes, the value shown on the screen changes as well.

---

### 2. `handlePress` Function

```tsx
const handlePress = () => { setCount(count + 1) };
```

This function defines what should happen when the button is pressed.

When `handlePress` runs:

1. The current value of `count` is obtained.
2. `1` is added to that value.
3. `setCount` is used to store the new value.
4. React updates the interface to display the new state.

The original code also contains the following commented-out function:

```tsx
//const handlePress = () => {console.log('you clicked the button')};
```

This is useful because it demonstrates two different ways of responding to an action.

The first version would simply write a message to the console.

The current version changes the application's state, which results in a visible change to the user interface.

This is an important transition from simply testing whether an event occurred to using that event to change the application.

---

### 3. `<View>` Component

```tsx
<View style={styles.container}>...</View>
```

`View` is one of the fundamental components in React Native.

It can be used as a container for other components and is commonly used to organise the layout of an interface.

In this application, the `View` acts as the main container for:

* The title
* The number
* The button

The `style` property connects the `View` to the `container` style defined in the `StyleSheet`.

This allows us to control how the contents of the `View` are positioned on the screen.

---

### 4. `<Text>` Component

```tsx
<Text>COMPONENTS V1</Text>
<Text>{count}</Text>
```

The `Text` component is used to display text in a React Native application.

The first `Text` component contains a fixed piece of text:

```tsx
<Text>COMPONENTS V1</Text>
```

The second displays the current value of the `count` variable:

```tsx
<Text>{count}</Text>
```

The `{}` syntax is significant.

Inside JSX, curly braces allow us to insert JavaScript expressions into the interface.

In this case, `{count}` tells React Native to evaluate the `count` variable and display its current value.

This means that the second `Text` component is not displaying a fixed value. It displays whatever value `count` currently contains.

---

### 5. `<Button>` Component

```tsx
<Button 
  title="CLICK HERE"
  onPress={handlePress}
/>
```

The `Button` component provides a simple way for the user to perform an action.

The `title` property determines the text displayed on the button:

```tsx
title="CLICK HERE"
```

The `onPress` property determines what should happen when the user presses the button:

```tsx
onPress={handlePress}
```

Here, `handlePress` is passed to `onPress` so that the function can be executed when the user interacts with the button.

This connects the user interface to the program logic.

The button therefore becomes the starting point of the application's interaction:

**Button press → `handlePress()` → `setCount()` → updated `count` → updated display**

---

### 6. `StyleSheet` for Styling

```tsx
const styles = StyleSheet.create({ ... });
```

`StyleSheet` is used to organise the styles used by the React Native components.

In this application, the `container` style defines several layout properties:

```tsx
container: {
  flex: 1,
  backgroundColor: '#fff',
  alignItems: 'center',
  justifyContent: 'center',
},
```

The important properties are:

* `flex: 1` — allows the container to fill the available screen space.
* `backgroundColor: '#fff'` — gives the container a white background.
* `alignItems: 'center'` — centres the contents horizontally.
* `justifyContent: 'center'` — centres the contents vertically.

Together, these properties determine how the components inside the main `View` are positioned.

---

## What Should You Take Away

The program is small, but it introduces several ideas that will become the foundation for larger React Native applications.

* **React Native applications are built from components** such as `<View>`, `<Text>`, and `<Button>`.
* **State**, created using `useState`, allows an application to keep track of information that can change.
* **Functions** can respond to user actions such as button presses.
* **JSX** allows JavaScript values and expressions to be included in the user interface using `{}`.
* **Styles** define how components are presented on the screen.
* Everything returned from `return(...)` describes the interface that React Native renders.

Most importantly, begin to look at the application as a relationship between its parts:

**Components provide the interface.**

**Functions provide behaviour.**

**State provides information that can change.**

**Events connect the user's actions to that behaviour.**

This simple application is therefore more than a button and a number. It provides the first foundation for understanding how an interactive React Native application works.