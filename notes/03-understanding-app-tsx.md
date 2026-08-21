# MAST5112: Mobile App Scripting – Class Notes

## 03 - Understanding App.tsx

---

## Overview

In the previous lesson, we learned about the individual components that we can use to build a React Native interface. We saw View, Text, TextInput, Button, and others.

We now turn our attention to one of the most important files in our project:

```text
App.tsx
```

This file contains the starting point for the application we are building.

In this lesson, we will answer the question:

> What am I actually looking at when I open App.tsx?

We will examine:

- Import statements
- The App component
- JSX
- View and Text
- Styling with StyleSheet
- Component hierarchy
- Adding interaction with useState, TextInput, and Button

The aim is not to memorise every piece of code. The aim is to understand what each part of the file is doing and how the parts work together.

---

## The Basic Structure of App.tsx

A simple React Native application can look like this:

```tsx
import { StyleSheet, Text, View } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text>Hello World</Text>
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

At first, this may look like a lot of unfamiliar syntax.

We can understand it by breaking the file into four main parts:

```text
1. Imports
      ↓
2. App component
      ↓
3. JSX / user interface
      ↓
4. Styles
```

We will examine each part separately.

---

## 1. Understanding the Import Statement

At the top of the file we have:

```tsx
import { StyleSheet, Text, View } from 'react-native';
```

The import statement allows us to bring components or other functionality into our file so that we can use them.

In this example, we are importing three things from React Native:

- View
- Text
- StyleSheet

We learned about each of these in the previous lesson.

| Component    | Purpose                             |
| ------------ | ----------------------------------- |
| View         | A container used to group components |
| Text         | Displays text on the screen         |
| StyleSheet   | Defines and organises styles        |

As our applications grow, we will import additional components and utilities.

---

## 2. Understanding the App Component

The next important part of the file is:

```tsx
export default function App() {
  return (
    <View>
      <Text>Hello World</Text>
    </View>
  );
}
```

This is a React component.

A component is a reusable part of a user interface.

In this case, our component is called App.

The component is created using a JavaScript or TypeScript function:

```tsx
function App() {
    ...
}
```

The function contains a return statement.

The return statement describes what we want the component to display.

### The Export Statement

The line:

```tsx
export default function App()
```

makes the App component available to other parts of the application.

Expo uses this exported component as the starting point for our application.

Without this export, Expo would not know which component to display.

---

## 3. What Does Return Give Us?

Inside the App function we have:

```tsx
return (
  <View>
    <Text>Hello World</Text>
  </View>
);
```

The code being returned is called JSX.

JSX allows us to describe the structure of our user interface using a syntax that looks similar to HTML.

However, it is important to understand:

> JSX is not HTML.

JSX is part of our JavaScript or TypeScript code. It is used by React to describe the interface that should appear on the screen.

For example:

```tsx
<Text>Hello World</Text>
```

means that we want the words Hello World to appear as text in our application.

---

## 4. Understanding JSX

JSX has a syntax that will look familiar if you have worked with HTML.

For example:

```tsx
<View>
  <Text>Hello World</Text>
</View>
```

Notice that components have an opening and closing tag:

```tsx
<Text>
  Hello World
</Text>
```

Components can also be written using a self-closing tag when they do not contain other content:

```tsx
<Text />
```

This is common for components like Image and TextInput.

### JSX Uses CamelCase

JSX uses camelCase for many attributes and style properties.

For example:

```tsx
backgroundColor
```

rather than:

```text
background-color
```

This is one of the differences students will notice when moving from HTML and CSS to React Native.

### JavaScript Expressions in JSX

We can include JavaScript or TypeScript expressions inside JSX by using curly braces:

```tsx
<Text>Hello, {name}</Text>
```

The curly braces tell JSX to evaluate the expression inside them.

We will see this used more in the interactive example later.

---

## 5. Understanding View

View is one of the most commonly used React Native components.

It is a general-purpose container that can:

- Group components together
- Control layout
- Contain other components
- Have styles applied to it

A useful way to think about View is:

> View provides a space in which other components can be arranged.

### Using View as a Container

```tsx
<View>
  <Text>First line</Text>
  <Text>Second line</Text>
  <Text>Third line</Text>
</View>
```

The View contains all three Text components.

### Nesting Views

Views can be nested inside each other:

```tsx
<View>
  <View>
    <Text>First section</Text>
  </View>
  <View>
    <Text>Second section</Text>
  </View>
</View>
```

This is how we build structured layouts.

---

## 6. Understanding Text

The Text component is used to display text.

For example:

```tsx
<Text>Hello World</Text>
```

Whenever we want to display text in a React Native interface, we should use the Text component.

### Multiple Text Components

We can place different pieces of text into separate components:

```tsx
<View>
  <Text>Welcome</Text>
  <Text>MAST5112</Text>
</View>
```

Each Text component can be styled independently.

---

## 7. Styling with StyleSheet

We can apply a style to a component using the style prop.

For example:

```tsx
<View style={styles.container}>
```

Notice the syntax:

```text
styles.container
```

This refers to a style called container.

That style is defined elsewhere in the file.

### Defining Styles

At the bottom of our file we define our styles:

```tsx
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});
```

StyleSheet.create allows us to define and organise styles using JavaScript or TypeScript objects.

The style has a name:

```text
container
```

and contains several style properties:

```text
flex
justifyContent
alignItems
```

We then use that style in our JSX:

```tsx
<View style={styles.container}>
```

The relationship is:

```text
Style definition
      ↓
styles.container
      ↓
<View style={styles.container}>
```

This allows us to keep our interface structure and our styling organised.

---

## 8. Understanding the Layout Props

The container style in our example uses three layout props:

```tsx
container: {
  flex: 1,
  justifyContent: 'center',
  alignItems: 'center',
}
```

We introduced these in the previous lesson.

```text
flex: 1                 The container fills all available space
justifyContent: center   Content is centred vertically
alignItems: center       Content is centred horizontally
```

For now, recognise that these props control how the contents of the View are positioned on the screen.

We will explore React Native layout and Flexbox in more detail later.

---

## 9. The Component Hierarchy

Look at this code:

```tsx
<View>
  <Text>Hello World</Text>
</View>
```

There are two components here.

The View is the parent:

```text
View
└── Text
```

The Text is a child of the View.

### Building More Complex Hierarchies

We can create larger interfaces by placing components inside other components.

```tsx
<View>
  <View>
    <Text>First Section</Text>
  </View>
  <View>
    <Text>Second Section</Text>
  </View>
</View>
```

The hierarchy can be represented as:

```text
View
├── View
│   └── Text
└── View
    └── Text
```

This is called the component hierarchy.

As our applications become more complex, understanding this hierarchy will become increasingly important.

### A Rule About JSX Return Values

JSX must return a single root element.

This means:

```tsx
// Valid
return (
  <View>
    <Text>Hello</Text>
  </View>
);
```

```tsx
// Invalid - multiple root elements
return (
  <Text>Hello</Text>
  <Text>World</Text>
);
```

If you need to return multiple elements without a container, you can use a View as the wrapper.

---

## 10. Putting the Basic Structure Together

We can now read the entire application from top to bottom:

```tsx
import { StyleSheet, Text, View } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text>Hello World</Text>
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

We can describe the file as:

```text
Import the tools we need
        ↓
Create the App component
        ↓
Return the user interface
        ↓
Use View to contain the interface
        ↓
Use Text to display information
        ↓
Apply styles to control the layout
```

This is the basic structure we will build upon.

---

## 11. Adding Interaction

So far, our application only displays information.

A real application needs to respond to the user.

For example, we may want the user to:

- Enter information
- Press a button
- Change something on the screen
- Submit a form

To demonstrate this, we can extend our application.

Consider the following example.

### Interactive Example

```tsx
import { useState } from 'react';
import {
  Button,
  StyleSheet,
  Text,
  TextInput,
  View,
} from 'react-native';

export default function App() {

  const [userInput, setUserInput] = useState<string>('');

  const handlePress = () => {
    console.log(userInput);
  };

  return (
    <View style={styles.container}>

      <Text>USER INPUT APP</Text>

      <TextInput
        placeholder="Enter your text here"
        onChangeText={setUserInput}
        value={userInput}
      />

      <Text>{userInput}</Text>

      <Button
        title="Click Here"
        onPress={handlePress}
      />

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

This example introduces several new concepts.

We will look at them one at a time.

---

## 12. The useState Hook

At the top of the file we now have:

```tsx
import { useState } from 'react';
```

We are importing something from React rather than React Native.

useState is a React Hook.

A Hook is a special React function that allows a functional component to use features such as state.

For now, the most important Hook for us is useState.

We will dedicate a full lesson to state hooks later. For now, we will focus on what this code does.

---

## 13. What Is State?

State is information that belongs to a component and can change while the application is running.

For example, a user might enter the name John into a text box.

The application needs somewhere to keep that value.

This is where state becomes useful.

We can create state using:

```tsx
const [userInput, setUserInput] = useState<string>('');
```

There are two important names here:

```text
userInput
setUserInput
```

userInput represents the current value stored in the state.

setUserInput is the function we use to change the value.

The initial value is:

```text
''
```

which represents an empty string.

So we can think of this as:

```text
userInput → Current value
setUserInput() → Changes the value
```

### Why <string>?

Because this project uses TypeScript, we can specify the type of our state:

```tsx
useState<string>('')
```

The <string> tells TypeScript that this state will contain text.

For simple examples, TypeScript can often determine the type automatically:

```tsx
useState('')
```

However, using the explicit type can help demonstrate what kind of data the state is expected to contain.

---

## 14. Understanding TextInput

The TextInput component allows the user to enter text.

We use:

```tsx
<TextInput
  placeholder="Enter your text here"
  onChangeText={setUserInput}
  value={userInput}
/>
```

There are three important properties here.

### placeholder

```tsx
placeholder="Enter your text here"
```

This displays a temporary instruction before the user enters any text.

### value

```tsx
value={userInput}
```

This connects the text box to our state.

The current value of userInput is displayed in the text input.

### onChangeText

```tsx
onChangeText={setUserInput}
```

This tells React Native what to do when the user changes the text.

In this case, we use setUserInput to update our state.

The relationship is:

```text
User types text
      ↓
TextInput detects change
      ↓
onChangeText
      ↓
setUserInput()
      ↓
userInput is updated
```

---

## 15. Displaying the State

We can display the current value of our state using:

```tsx
<Text>{userInput}</Text>
```

The curly braces:

```tsx
{userInput}
```

tell JSX that we want to use a JavaScript or TypeScript expression here.

So if the user enters:

```text
Hello
```

the component becomes conceptually:

```tsx
<Text>Hello</Text>
```

The important idea is that the displayed value comes from our state.

```text
TextInput
    ↓
userInput
    ↓
<Text>{userInput}</Text>
```

This is one of the fundamental ideas behind interactive React applications.

---

## 16. Understanding Button

The Button component allows the user to perform an action.

Our example contains:

```tsx
<Button
  title="Click Here"
  onPress={handlePress}
/>
```

There are two important properties.

### title

```tsx
title="Click Here"
```

This is the text displayed on the button.

### onPress

```tsx
onPress={handlePress}
```

This tells the button which function should run when the user presses it.

---

## 17. Understanding handlePress

Earlier in our component we created:

```tsx
const handlePress = () => {
  console.log(userInput);
};
```

This is a function.

When the button is pressed, the function runs:

```text
Button pressed → handlePress() → console.log(userInput)
```

For now, the result is displayed in the development console.

Later, we will use functions such as this to perform useful actions in our applications.

---

## 18. The Data Flow in Our Interactive Example

The important relationships in our example are:

Absolutely. I would make the flow a little more structured by separating the **input/update path** from the **two outcomes of the state**.

```text
                         ┌─────────────┐
                         │    User     │
                         └──────┬──────┘
                                │
                         enters text
                                │
                                ▼
                       ┌────────────────┐
                       │   TextInput    │
                       └───────┬────────┘
                               │
                               │ onChangeText
                               ▼
                       ┌────────────────┐
                       │ setUserInput() │
                       └───────┬────────┘
                               │
                               ▼
                       ┌────────────────┐
                       │  userInput     │
                       │     state      │
                       └───────┬────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
                    │                     │
                    ▼                     ▼
          ┌─────────────────┐     ┌─────────────────┐
          │  Text displays  │     │  Button pressed │
          │  current value  │     └────────┬────────┘
          └─────────────────┘              │
                                           ▼
                                  ┌─────────────────┐
                                  │   handlePress() │
                                  └────────┬────────┘
                                           │
                                           ▼
                                  ┌─────────────────┐
                                  │   console.log() │
                                  └─────────────────┘
```

**User → TextInput → `setUserInput()` → `userInput` state → UI / Button action**

This is our first example of a React Native application responding to user input.

---

## 19. What We Have Added

We started with a simple application:

```text
View
└── Text
```

We then introduced interaction:

```text
View
├── Text
├── TextInput
├── Text
└── Button
```

We also introduced state:

```text
userInput
```

and a function that responds to a button press:

```text
handlePress
```

The application is therefore beginning to move from a static interface to an interactive application.

---

## 20. Important Concepts to Remember

At this stage, you should understand the purpose of the following concepts.

| Concept               | Purpose                                                     |
| --------------------- | ----------------------------------------------------------- |
| import                | Brings required components or functions into the file       |
| function App()        | Defines our main React component                            |
| return                | Provides the JSX that describes the interface               |
| JSX                   | Syntax used to describe the user interface                  |
| View                  | Container used to group and arrange components              |
| Text                  | Displays text                                               |
| StyleSheet.create()   | Defines and organises styles                                |
| flex                  | Controls how much space a component takes up                |
| justifyContent        | Controls position along the main axis                       |
| alignItems            | Controls position along the cross axis                      |
| useState              | Creates state that can change while the application runs    |
| userInput             | Stores the current input value                              |
| setUserInput          | Updates the input value                                     |
| TextInput             | Allows the user to enter text                               |
| Button                | Provides a control that the user can press                  |
| onPress               | Specifies what should happen when a button is pressed       |
| handlePress           | Function that performs an action when the button is pressed |

---

## 21. The Bigger Picture

Our App.tsx file is beginning to show the basic pattern that we will use throughout the module:

```text
IMPORT
  ↓
Bring in the tools we need

COMPONENT
  ↓
Create our App component

STATE / LOGIC
  ↓
Store information and define behaviour

JSX
  ↓
Describe what the user sees

STYLING
  ↓
Control how the interface is arranged and displayed

INTERACTION
  ↓
Respond to what the user does
```

As we progress through the subject, our applications will become more complex, but these ideas will continue to appear.

The goal is therefore not to memorise this one App.tsx file.

The goal is to understand the structure well enough that we can build upon it.

---

## What You Have Learned

In this document, you have learned:

- The four main parts of an App.tsx file: imports, App component, JSX, and styles
- What import statements do and why they are needed
- How the App component is defined and exported
- What JSX is and how it differs from HTML
- The role of View and Text in building interfaces
- How to use StyleSheet to define and organise styles
- The basic layout props: flex, justifyContent, alignItems
- What a component hierarchy is and why it matters
- How to add interaction using useState, TextInput, and Button
- How state connects user input to the interface
- The basic data flow in an interactive React Native application

---

## Next Steps

You now understand the structure of App.tsx and how the pieces fit together.

In the next lesson, we will explore state hooks in more detail. We will learn why state is important, how useState works, and how to use it effectively in our applications.

---