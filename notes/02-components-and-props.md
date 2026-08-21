# MAST5112: Mobile App Scripting – Class Notes

## 02 - Components and Props

---

## Introduction

In React Native, components are the building blocks of your application's user interface. A component can represent a button, some text, an input field, an image, or a container that holds other components.

Each component has a set of properties, known as props, which configure how the component looks and behaves.

Props can be:

- Compulsory: Required for the component to work. For example, the `title` prop in a Button is compulsory.
- Optional: Used to enhance appearance or interactivity. For example, the `color` prop in a Button is optional.

In this document, you will learn:

- The role of the most common React Native components
- Where and why each component is used
- How to use props to change component behaviour
- How to apply styles to components

By the end of this document, you will be able to identify and use the core components needed to build a simple React Native interface.

---

## Styling Components with StyleSheet

Before we look at individual components, we need to understand how styling works in React Native.

Styles are defined using JavaScript or TypeScript objects. React Native provides a utility called StyleSheet to organise these styles.

### The StyleSheet Pattern

The typical pattern is:

```tsx
import { StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  nameOfStyle: {
    property: value,
    property: value,
  },
});
```

The style is then applied to a component using the `style` prop:

```tsx
<View style={styles.nameOfStyle}>
```

### Example

```tsx
import { StyleSheet, Text, View } from 'react-native';

const styles = StyleSheet.create({
  heading: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
  },
});

// Usage:
<Text style={styles.heading}>Welcome</Text>
```

### Why Use StyleSheet?

- Styles are defined in one place and can be reused
- The code is easier to read and maintain
- React Native can optimise performance compared to inline styles

### Important Note About Property Names

React Native uses camelCase for style properties, not kebab-case.

| CSS (Web)           | React Native        |
| ------------------- | ------------------- |
| `background-color`  | `backgroundColor`   |
| `font-size`         | `fontSize`          |
| `margin-top`        | `marginTop`         |
| `border-radius`     | `borderRadius`      |

This is a common source of errors when students first start using React Native.

---

## Basic Layout Props

When you use components such as View, you will often need to control how they are positioned on the screen.

React Native uses a layout system based on Flexbox. For now, we will introduce three important layout props. A later document will cover layout in more detail.

### flex

The `flex` prop determines how a component expands to fill available space.

```tsx
flex: 1
```

This means the component will take up all available space in its container.

### justifyContent

The `justifyContent` prop controls how content is distributed along the main axis of the layout.

Common values:

```text
center         Centers content
flex-start     Positions content at the start
flex-end       Positions content at the end
space-between  Distributes content evenly
```

### alignItems

The `alignItems` prop controls how content is distributed along the cross axis of the layout.

Common values:

```text
center         Centers content
flex-start     Positions content at the start
flex-end       Positions content at the end
stretch        Stretches content to fill
```

These three props will appear frequently in our examples. When you see them, remember:

```text
flex           How much space the component takes up
justifyContent How content is positioned along the main direction
alignItems     How content is positioned across the other direction
```

---

## The View Component

### Purpose

View is a general-purpose container. It is used to group other components together and to control their layout.

### Scenario

You want to create a section of your interface that contains multiple elements, such as a title and a button, and you want to control how they are arranged.

### Example

```tsx
<View style={styles.container}>
  <Text>This is inside a View</Text>
  <Text>So is this</Text>
</View>
```

### Key Props

| Prop            | Purpose                                          |
| --------------- | ------------------------------------------------ |
| `style`         | Controls layout, spacing, background, alignment  |
| `flexDirection` | Sets the direction of the layout (row or column) |
| `padding`       | Adds space inside the View                       |
| `margin`        | Adds space outside the View                      |

### When to Use View

Use View whenever you need to group components together. Most interfaces are built by nesting View components inside one another.

---

## The Text Component

### Purpose

Text is used to display text content on the screen.

### Scenario

You want to show a heading, a label, a message, or any other text.

### Example

```tsx
<Text style={styles.heading}>Welcome to My App</Text>
<Text>This is a paragraph of text.</Text>
```

### Key Props

| Prop            | Purpose                                                    |
| --------------- | ---------------------------------------------------------- |
| `style`         | Controls font size, colour, weight, spacing                |
| `numberOfLines` | Limits text to a specific number of lines, adds "..."      |
| `ellipsizeMode` | Determines how overflow text is clipped (head, middle, tail) |

### When to Use Text

Use Text whenever you want to display any written information. All text in a React Native application must be wrapped in a Text component.

---

## The TextInput Component

### Purpose

TextInput allows the user to enter or edit text.

### Scenario

You want the user to provide their name, email address, a message, or any other text-based input.

### Example

```tsx
<TextInput
  placeholder="Enter your name"
  onChangeText={(text) => console.log(text)}
  style={styles.input}
/>
```

### Key Props

| Prop              | Purpose                                           |
| ----------------- | ------------------------------------------------- |
| `placeholder`     | Grey text shown when the input is empty           |
| `value`           | The current value of the input (used with state)  |
| `onChangeText`    | Function called when the user types               |
| `secureTextEntry` | Hides the text (used for passwords)               |
| `keyboardType`    | Sets the type of keyboard (email, numeric, etc.)  |

### When to Use TextInput

Use TextInput whenever the user needs to enter text. This includes forms, search bars, and any other input field.

---

## The Button Component

### Purpose

Button creates a simple, pressable control that performs an action when tapped.

### Scenario

You want to submit a form, confirm an action, or trigger an event.

### Example

```tsx
<Button title="Submit" onPress={() => alert("Submitted!")} />
```

### Key Props

| Prop      | Purpose                                    |
| --------- | ------------------------------------------ |
| `title`   | The text displayed on the button           |
| `onPress` | The function called when the button is tapped |
| `color`   | Changes the button colour (Android only)   |

### When to Use Button

Use Button for simple actions where you need a standard, platform-consistent button. For custom-styled buttons, use TouchableOpacity or TouchableHighlight instead.

---

## The TouchableOpacity Component

### Purpose

TouchableOpacity is a button-like component that provides visual feedback when pressed. The component fades slightly when the user touches it.

### Scenario

You want a custom-styled button, an image that responds to touches, or any touchable element that needs custom styling.

### Example

```tsx
<TouchableOpacity
  style={styles.customButton}
  onPress={() => alert("Pressed!")}
>
  <Text style={styles.buttonText}>Tap Me</Text>
</TouchableOpacity>
```

### Key Props

| Prop            | Purpose                                             |
| --------------- | --------------------------------------------------- |
| `onPress`       | The function called when the component is pressed   |
| `style`         | Controls the appearance (colour, padding, border)   |
| `activeOpacity` | Controls the opacity when pressed (default is 0.2)  |

### When to Use TouchableOpacity

Use TouchableOpacity when you need a custom button that does not use the standard Button appearance. This is common when building branded or styled interfaces.

---

## The TouchableHighlight Component

### Purpose

TouchableHighlight is similar to TouchableOpacity, but it provides feedback by showing a background colour behind the component when pressed.

### Scenario

You want a button that highlights with a colour when pressed, rather than fading.

### Example

```tsx
<TouchableHighlight
  underlayColor="#ddd"
  onPress={() => alert("Highlight Pressed!")}
>
  <Text style={styles.buttonText}>Highlight Button</Text>
</TouchableHighlight>
```

### Key Props

| Prop            | Purpose                                               |
| --------------- | ----------------------------------------------------- |
| `underlayColor` | The colour shown behind the component when pressed    |
| `onPress`       | The function called when the component is pressed     |
| `style`         | Controls the appearance                               |

### When to Use TouchableHighlight

Use TouchableHighlight when you prefer a colour highlight effect over a fade effect. This is often used in lists and menus.

---

## The Image Component

### Purpose

Image is used to display static images from a local file or from the web.

### Scenario

You want to add a logo, a profile picture, or any other image to your interface.

### Example

```tsx
<Image
  source={require('./assets/logo.png')}
  style={{ width: 100, height: 100 }}
/>
```

### Key Props

| Prop     | Purpose                                      |
| -------- | -------------------------------------------- |
| `source` | The location of the image (local file or URL) |
| `style`  | Controls width, height, border, and resizing |
| `resizeMode` | Controls how the image scales to fit     |

### Loading a Local Image

Use require() with a relative path:

```tsx
source={require('./assets/logo.png')}
```

### Loading a Web Image

Use an object with a uri property:

```tsx
source={{ uri: 'https://example.com/image.jpg' }}
```

### When to Use Image

Use Image whenever you need to display an image. This includes icons, logos, photos, and any other visual content.

---

## The ImageBackground Component

### Purpose

ImageBackground is used to display an image as the background of a container. Other components can be placed on top of the image.

### Scenario

You want a screen with a background image, with text and buttons layered over it.

### Example

```tsx
<ImageBackground
  source={require('./assets/background.jpg')}
  style={styles.background}
>
  <Text style={styles.overlayText}>Overlay Text</Text>
</ImageBackground>
```

### Key Props

| Prop         | Purpose                                          |
| ------------ | ------------------------------------------------ |
| `source`     | The location of the background image             |
| `style`      | Controls the size and appearance of the container |
| `resizeMode` | Controls how the image scales to fit             |

### When to Use ImageBackground

Use ImageBackground when you need a background image with content layered on top. This is common for landing screens, profile screens, and hero sections.

---

## Component Summary Table

The following table provides a quick reference for the components introduced in this document.

| Component          | Used For                      | Must-Know Prop        |
| ------------------ | ----------------------------- | --------------------- |
| View               | Layout and container          | style                 |
| Text               | Displaying text               | style                 |
| TextInput          | Capturing user input          | onChangeText, value   |
| Button             | Basic interactions            | title, onPress        |
| TouchableOpacity   | Custom pressable elements     | onPress, style        |
| TouchableHighlight | Pressable with colour feedback | onPress, underlayColor |
| Image              | Displaying images             | source, style         |
| ImageBackground    | Background images with content overlay | source, style   |

---

## Hands-On Practice

The following example combines several of the components we have discussed.

Try copying this code into your App.tsx file and running it.

```tsx
import { useState } from 'react';
import { Button, Image, StyleSheet, Text, TextInput, View } from 'react-native';

export default function App() {
  const [name, setName] = useState<string>('');

  return (
    <View style={styles.container}>
      <Image source={require('./assets/avatar.png')} style={styles.image} />
      <Text style={styles.title}>Welcome!</Text>
      <TextInput
        placeholder="Enter your name"
        value={name}
        onChangeText={setName}
        style={styles.input}
      />
      <Text>Hello, {name}</Text>
      <Button title="Press me" onPress={() => alert(`Hi ${name || 'stranger'}!`)} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 16,
    backgroundColor: '#fff',
  },
  image: {
    width: 100,
    height: 100,
    marginBottom: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
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

### Understanding the Layout Props in This Example

Take a moment to look at the styles defined above.

```tsx
container: {
  flex: 1,
  justifyContent: 'center',
  alignItems: 'center',
  padding: 16,
  backgroundColor: '#fff',
}
```

- `flex: 1` means the container fills all available space on the screen
- `justifyContent: 'center'` positions the content vertically in the centre
- `alignItems: 'center'` positions the content horizontally in the centre
- `padding: 16` adds space inside the container
- `backgroundColor: '#fff'` sets a white background

Because of these props, all the components inside the View are centred on the screen. The Image appears at the top of the centred group, followed by the title, the input field, the greeting, and the button.

A future document will cover layout in more detail. For now, recognise that these props control where components appear on the screen.

---

## What You Have Learned

In this document, you have learned:

- What components are and how they are used to build interfaces
- What props are and how they configure component behaviour
- How to use StyleSheet to define and organise styles
- The purpose and key props of the most common React Native components:
  - View
  - Text
  - TextInput
  - Button
  - TouchableOpacity
  - TouchableHighlight
  - Image
  - ImageBackground
- Basic layout props: flex, justifyContent, alignItems
- How to combine multiple components in a single interface

---

## Next Steps

With these components, you can now build a static user interface.

In the next document, we will look at how these components fit together in a complete App.tsx file and how to structure a React Native application.

---