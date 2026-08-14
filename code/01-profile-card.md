# 01 Build a React Native Profile Card

This exercise develops a React Native profile card progressively, introducing layout, styling, images, and component reusability.

The program is developed in stages. Each stage builds on the previous version of the application so that you can see how a relatively simple screen can gradually become a reusable component.

---

## Stage 1: Project Setup

> **Create the project and begin with the basic container setup.**

At this stage, we are establishing the basic structure of the application. The `View` component provides the initial container, while `StyleSheet` allows us to define the visual properties of that container.

The `flex: 1` property allows the container to occupy the available screen space, while `justifyContent: 'center'` positions its contents vertically in the centre.

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
    backgroundColor: '#ecf0f1',
    padding: 8,
  },
});
```

---

## Stage 2: Basic Layout with Text Components

> **Add name and description text elements.**

The next stage introduces content into the container.

React Native provides the `Text` component for displaying text. We will use two `Text` components: one for the person's name and another for a short description.

Notice that each `Text` component is associated with a style defined in the `StyleSheet`. This allows the structure of the interface and its visual presentation to remain clearly separated.

```tsx
import { Text, View, StyleSheet } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.name}>Your Name</Text>
      <Text style={styles.description}>
        Mobile Application Developer passionate about programming
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    backgroundColor: '#ecf0f1',
    padding: 8,
  },
  name: {
    fontSize: 24,
    fontWeight: 'bold',
  },
  description: {
    fontSize: 16,
    marginTop: 4,
  },
});
```

---

## Stage 3: Add Profile Image

> **Introduce an image using the `Image` component and improve the layout.**

A profile card normally contains more than text. In this stage, we introduce the React Native `Image` component.

The image is loaded from the application's local `assets` folder using `require()`. We also introduce a new style called `image` to control its dimensions and appearance.

The `borderRadius` value is used to create a circular image. Because the width and height are both `100`, a `borderRadius` of `50` produces a circular appearance.

```tsx
import { Image, Text, View, StyleSheet } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Image
        style={styles.image}
        source={require('./assets/snack-icon.png')}
      />
      <Text style={styles.name}>Your Name</Text>
      <Text style={styles.description}>
        Mobile Application Developer passionate about programming
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    backgroundColor: '#ecf0f1',
    padding: 8,
  },
  name: {
    fontSize: 24,
    fontWeight: 'bold',
  },
  description: {
    fontSize: 16,
    marginTop: 4,
  },
  image: {
    width: 100,
    height: 100,
    borderRadius: 50,
    marginBottom: 10,
  },
});
```

---

## Stage 4: Style with a Card Layout

> **Add a card container for a cleaner user interface.**

The previous stage contains all of the required information, but the elements are still placed directly inside the main container.

In this stage, we introduce another `View` to act as the profile card itself.

This demonstrates an important idea in React Native: a `View` does not have to represent an entire screen. It can also be used to group related elements into smaller sections of an interface.

The new `card` style controls the appearance of this group, including its spacing, alignment, background, rounded corners, and shadow.

```tsx
import { Image, Text, View, StyleSheet } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <View style={styles.card}>
        <Image
          style={styles.image}
          source={require('./assets/snack-icon.png')}
        />
        <Text style={styles.name}>Your Name</Text>
        <Text style={styles.description}>
          Mobile Application Developer passionate about programming
        </Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    backgroundColor: '#ecf0f1',
    padding: 8,
  },
  card: {
    alignItems: 'center',
    padding: 16,
    borderRadius: 8,
    backgroundColor: '#fff',
    shadowColor: '#000',
    shadowOpacity: 0.1,
    shadowRadius: 6,
    elevation: 3,
  },
  image: {
    width: 100,
    height: 100,
    borderRadius: 50,
    marginBottom: 10,
  },
  name: {
    fontSize: 24,
    fontWeight: 'bold',
  },
  description: {
    fontSize: 16,
    marginTop: 4,
    textAlign: 'center',
  },
});
```

---

## Stage 5: Create a Reusable `ProfileCard` Component

### Concept: Componentisation with Props in TypeScript

In Stage 4, we created a profile card by placing the image and text inside a `View`.

The interface works well for a single person. However, consider what would happen if an application needed to display several profile cards.

The structure of each card would remain the same, while only the information would change:

```tsx
<View style={styles.card}>
  <Image source={...} />
  <Text>...</Text>
  <Text>...</Text>
</View>
```

We could copy and paste this entire block and change the names and images. While this would work, it creates unnecessary repetition.

* It takes more time to write.
* Repeated code is more difficult to maintain.
* Changes to the layout would need to be made in multiple places.

A better approach is to create our own component.

This is one of the important ideas behind component-based development: instead of thinking about an application as one large piece of code, we can divide the interface into smaller, reusable components.

---

### What Is a Component?

A **component** is a function that returns JSX describing part of the user interface.

We have already been using React Native components such as `<View>`, `<Text>`, and `<Image>`.

We can create our own component in exactly the same general way. In this case, we will create a component called `ProfileCard`.

The `ProfileCard` component will contain the structure of our card, while the information displayed by the card will be supplied through **props**.

The relationship can be viewed as:

* `ProfileCard` defines the structure of the card.
* `View`, `Text`, and `Image` define the elements inside the card.
* `props` provide the information that changes from one card to another.

---

### Define Props and Use Destructuring

```tsx
type Props = {
  image: ImageSourcePropType;
  fullName: string;
  devDescription: string;
};

function ProfileCard({ image, fullName, devDescription }: Props) {
  return (
    <View style={styles.card}>
      <Image style={styles.image} source={image} />
      <Text style={styles.name}>{fullName}</Text>
      <Text style={styles.description}>{devDescription}</Text>
    </View>
  );
}
```

The `Props` type describes the information that the `ProfileCard` component expects to receive.

There are three properties:

* `image` describes the image that should be displayed.
* `fullName` contains the person's name.
* `devDescription` contains the person's description.

Because this is TypeScript, each property also has a defined type.

For example:

```tsx
fullName: string;
```

means that `fullName` is expected to contain a string value.

The component then receives these properties through its parameter:

```tsx
function ProfileCard({ image, fullName, devDescription }: Props)
```

The `{ image, fullName, devDescription }` syntax is called **destructuring**.

Instead of receiving the complete `Props` object and then accessing each property separately, we can extract the properties we need immediately.

Conceptually, we are saying:

> Give the component a `Props` object and provide direct access to `image`, `fullName`, and `devDescription`.

The `Props` type therefore acts as a description of the information required by the component.

---

Inside `App`, the component can then be used like this:

```tsx
<ProfileCard
  image={require('./assets/snack-icon.png')}
  fullName="Your Name"
  devDescription="Your Description"
/>
```

This is different from creating the entire card layout again.

Instead, we are using the `ProfileCard` component and supplying it with the information it needs.

The component determines **how the card is structured**, while the props determine **what information the card displays**.

---

### Why Is This Helpful? — Especially With Multiple Cards

Imagine that an application needs to display three people:

```tsx
<ProfileCard
  image={require('./assets/jax.png')}
  fullName="Jax Shepard"
  devDescription="iOS Developer from Johannesburg"
/>

<ProfileCard
  image={require('./assets/jada.png')}
  fullName="Jada Lynch"
  devDescription="Android Developer exploring AI"
/>

<ProfileCard
  image={require('./assets/joey.png')}
  fullName="Joey Styles"
  devDescription="React Native Developer and mentor"
/>
```

The layout of the card does not need to be written three times.

Instead, the same `ProfileCard` component is used three times, with different values supplied through props.

This is the key advantage of componentisation: **one component can describe a common interface structure while different data is supplied to each instance of that component.**

---

### Preview of the Next Stage: Splitting the Component Into Its Own File

At this point, we are keeping the `ProfileCard` component in the same file as `App`. This keeps the example together while we are learning how the component works.

In a larger application, however, it is often useful to place components into their own files.

For example, we could:

* Move `ProfileCard` to a separate file such as `ProfileCard.tsx`.
* Import the component into the file where it is needed.

This allows the component to be organised independently and reused across different screens or applications.

The important idea is that **componentisation is not only about making code shorter**. It is also about creating clear, reusable units of an application's user interface.

---

### Complete Code

```tsx
import React from 'react';
import { View, Text, Image, StyleSheet, ImageSourcePropType } from 'react-native';

// Define a Props type for clarity and safety
type Props = {
  image: ImageSourcePropType;
  fullName: string;
  devDescription: string;
};

// Receive props and destructure them
function ProfileCard({ image, fullName, devDescription }: Props) {
  return (
    <View style={styles.card}>
      <Image style={styles.image} source={image} />
      <Text style={styles.name}>{fullName}</Text>
      <Text style={styles.description}>{devDescription}</Text>
    </View>
  );
}

export default function App() {
  return (
    <View style={styles.container}>
      <ProfileCard
        image={require('./assets/snack-icon.png')}
        fullName="Your Name"
        devDescription="Mobile Application Developer passionate about programming"
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    backgroundColor: '#ecf0f1',
    padding: 8,
  },
  card: {
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOpacity: 0.1,
    shadowRadius: 6,
    elevation: 3,
  },
  image: {
    width: 100,
    height: 100,
    borderRadius: 50,
    marginBottom: 16,
  },
  name: {
    fontSize: 18,
    fontWeight: 'bold',
  },
  description: {
    fontSize: 16,
    textAlign: 'center',
  },
});
```
