# MAST5112: Mobile App Scripting – Class Notes

## 06 - Implement Navigation

---

## Introduction

In the previous lessons, we built applications that displayed a single screen. We learned about components, props, state, and layout.

However, real mobile applications typically have multiple screens. Users need to move between screens: from a home screen to a profile screen, from a list to a detail view, or from a login screen to the main application.

Navigation is the system that manages these screen transitions.

In this lesson, we will learn how to implement navigation in React Native using the React Navigation library.

Specifically, we will focus on Native Stack Navigation, which provides smooth, native-like transitions between screens.

By the end of this lesson, you will be able to:

- Install and configure React Navigation
- Define a stack navigator with multiple screens
- Navigate between screens
- Pass navigation props to screens
- Customise screen options

---

## What Is Navigation in React Native?

Navigation allows users to move between different screens in a mobile application.

In React Native, the most common navigation library is React Navigation. It provides several types of navigators:

| Navigator Type         | Description                                           |
| ---------------------- | ----------------------------------------------------- |
| Native Stack Navigator | Screens are stacked on top of each other (like cards) |
| Bottom Tab Navigator   | Screens are organised in tabs at the bottom           |
| Drawer Navigator       | A side menu slides in from the edge                   |
| Material Top Tab       | Tabs are displayed at the top of the screen           |

In this lesson, we focus on the Native Stack Navigator.

### Stack Navigation Concept

Think of a stack of cards.

```text
              Navigation Stack
                    ↓
        ┌─────────────────────┐
        │       Screen        │  ← Top of stack
        │      (VISIBLE)      │
        └─────────────────────┘
        ┌─────────────────────┐
        │       Screen        │
        └─────────────────────┘
        ┌─────────────────────┐
        │       Screen        │
        └─────────────────────┘
        ┌─────────────────────┐
        │       Screen        │  ← Bottom of stack
        └─────────────────────┘
```

When you navigate to a new screen, it is placed on top of the stack.

When you go back, the top screen is removed, revealing the screen below.

This is how stack navigation works in React Native.

---

## Key Concepts

Before we write any code, we need to understand three important concepts.

### NavigationContainer

NavigationContainer is the root component of your navigation system. It wraps your entire application and provides the navigation context.

Think of it as the foundation upon which all navigation is built.

### Stack.Navigator

Stack.Navigator is a component that holds all the screens in your stack. It manages the stack of screens and handles transitions.

### Stack.Screen

Stack.Screen represents a single screen in your navigation stack. Each screen has a name and a component that renders the screen content.

### Visual Relationship

```text
┌──────────────────────────────────────┐
│         NavigationContainer          │
│                                      │
│  Provides the navigation context     │
└──────────────────┬───────────────────┘
                   ↓
┌──────────────────────────────────────┐
│           Stack.Navigator            │
│                                      │
│       Manages the screen stack       │
└─────────────────┬────────────────────┘
                  │
        ┌─────────┼───────────┬──────────┐
        │         │           │          │
        ↓         ↓           ↓          ↓
     ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
     │  Home  │ │Profile │ │Details │ │Settings│
     │ Screen │ │ Screen │ │ Screen │ │ Screen │
     └────────┘ └────────┘ └────────┘ └────────┘
```

Each Screen is a component that contains the UI for that specific screen.

---

## Project Setup and Installation

### Step-by-Step Instructions

1. Open Visual Studio Code
2. Select File > Open Folder and select your MAST folder
3. Open a new terminal: Terminal > New Terminal
4. Create a new blank TypeScript Expo app:

```bash
npx create-expo-app -t expo-template-blank-typescript
```

5. When prompted, name the application: nav1
6. Navigate into the project folder:

```bash
cd nav1
```

7. Verify you are in the correct folder:

```bash
ls
```

You should see files such as App.tsx and package.json.

8. Install the navigation dependencies:

```bash
npm install @react-navigation/native
npx expo install react-native-screens react-native-safe-area-context
npm install @react-navigation/native-stack
```

### Important Note

Do not keep package.json open in the editor while installing dependencies. If you save the file manually during installation, you might overwrite the automatic updates.

---

## Understanding the Navigation Types

TypeScript helps us ensure that we navigate to screens that actually exist.

### Defining the Route Types

```ts
type RootStackParamList = {
  Home: undefined;
  Profile: undefined;
  Details: undefined;
  Settings: undefined;
};
```

This type definition does three things:

1. Defines the names of all screens in our navigation
2. Specifies what parameters each screen expects
3. Enables type checking and auto-completion

In this example, each screen has undefined parameters, meaning they do not expect any data to be passed.

### Passing Parameters

If a screen needs to receive data, you can define parameters:

```ts
type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Details: { itemId: number };
  Settings: undefined;
};
```

In this case, Profile expects a userId string, and Details expects an itemId number.

---

## Complete Code Example

Here is the complete code for a four-screen stack navigation application.

```tsx
import { Button, Text, View, StyleSheet } from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

// Define the route names and their parameters
type RootStackParamList = {
  Home: undefined;
  Profile: undefined;
  Details: undefined;
  Settings: undefined;
};

// Create the stack navigator with type safety
const Stack = createNativeStackNavigator<RootStackParamList>();

// Home Screen
function HomeScreen({ navigation }: { navigation: any }) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home Screen</Text>
      <Button
        title="Go to Profile"
        onPress={() => navigation.navigate('Profile')}
      />
    </View>
  );
}

// Profile Screen
function ProfileScreen({ navigation }: { navigation: any }) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Profile Screen</Text>
      <Button
        title="Go to Details"
        onPress={() => navigation.navigate('Details')}
      />
    </View>
  );
}

// Details Screen
function DetailsScreen({ navigation }: { navigation: any }) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Details Screen</Text>
      <Text style={styles.subtitle}>
        Use the back arrow to return (top left in the navigation header)
      </Text>
      <Button
        title="Go to Settings"
        onPress={() => navigation.navigate('Settings')}
      />
    </View>
  );
}

// Settings Screen
function SettingsScreen({ navigation }: { navigation: any }) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Settings Screen</Text>
      <Button
        title="Go Back One Screen"
        onPress={() => navigation.goBack()}
      />
      <Button
        title="Go back to Home"
        onPress={() => navigation.navigate('Home')}
      />
    </View>
  );
}

// App Entry Point
export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
        <Stack.Screen name="Settings" component={SettingsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

// Styles
const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: '#f0f4f7',
    padding: 20,
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
    color: '#333',
  },
  subtitle: {
    fontSize: 16,
    marginBottom: 10,
    color: '#666',
  },
});
```

---

## Code Breakdown

### NavigationContainer

```tsx
<NavigationContainer>
  {/* All navigation components go here */}
</NavigationContainer>
```

NavigationContainer wraps the entire navigation system. It must be the parent of all navigators.

### Stack.Navigator

```tsx
<Stack.Navigator initialRouteName="Home">
  {/* Screens go here */}
</Stack.Navigator>
```

Stack.Navigator holds all the screens in the stack. The initialRouteName prop tells the navigator which screen to display first.

### Stack.Screen

```tsx
<Stack.Screen name="Home" component={HomeScreen} />
```

Stack.Screen registers a screen with the navigator. The name prop is used for navigation, and the component prop specifies the screen component.

### Navigation Prop

Each screen receives a navigation prop automatically from React Navigation.

```tsx
function HomeScreen({ navigation }: { navigation: any }) {
  // navigation can be used to move to other screens
}
```

The navigation prop provides methods such as:

| Method              | Purpose                                          |
| ------------------- | ------------------------------------------------ |
| navigation.navigate('ScreenName') | Navigate to another screen             |
| navigation.goBack() | Return to the previous screen                    |
| navigation.push('ScreenName') | Add a new screen to the stack (even if already present) |
| navigation.replace('ScreenName') | Replace the current screen with another |

---

## Navigation Flow Diagram

The navigation flow for this application is:

```text
                    ┌─────────────────────────────┐
                    │     NavigationContainer     │
                    └──────────────┬──────────────┘
                                   ↓
                    ┌─────────────────────────────┐
                    │       Stack.Navigator       │
                    │                             │
                    │   initialRouteName: "Home"  │
                    └──────────────┬──────────────┘
                                   ↓
                          ┌─────────────────┐
                          │   Home Screen   │
                          └────────┬────────┘
                          navigate('Profile')
                                   ↓
                          ┌─────────────────┐
                          │ Profile Screen  │
                          └────────┬────────┘
                          navigate('Details')
                                   ↓
                          ┌─────────────────┐
                          │ Details Screen  │
                          └────────┬────────┘
                          navigate('Settings')
                                   ↓
                          ┌─────────────────┐
                          │ Settings Screen │
                          └─────────────────┘
                                   │
                                goBack()
                                   ↓
                          ┌─────────────────┐
                          │ Previous Screen │
                          └─────────────────┘

```

The arrows show the navigation flow. From Home, you can go to Profile. From Profile, you can go to Details. From Details, you can go to Settings. From Settings, you can go back to the previous screen or navigate directly to Home.

---

## Understanding the Navigation Prop

The navigation prop is passed to every screen component. It contains methods for navigating between screens.

### navigate

```tsx
navigation.navigate('Profile')
```

This navigates to the screen named Profile.

### goBack

```tsx
navigation.goBack()
```

This returns to the previous screen in the stack.

### push

```tsx
navigation.push('Details')
```

This adds a new screen to the stack, even if that screen is already present.

### replace

```tsx
navigation.replace('Home')
```

This replaces the current screen with another screen.

---

## Customising Screen Options

The Stack.Screen component accepts an options prop that customises the screen's appearance.

### Common Options

```tsx
<Stack.Screen
  name="Profile"
  component={ProfileScreen}
  options={{
    title: 'User Profile',           // Custom title in header
    headerStyle: { backgroundColor: '#6200ee' },
    headerTintColor: '#ffffff',
    headerShown: true,               // Show or hide the header
  }}
/>
```

### Available Options

| Option            | Purpose                                      |
| ----------------- | -------------------------------------------- |
| title             | The text displayed in the header             |
| headerStyle       | Styles for the header bar                    |
| headerTintColor   | Colour of header text and icons              |
| headerShown       | Whether to show or hide the header           |
| gestureEnabled    | Whether swipe gestures are enabled (iOS)     |

### Example with Custom Options

```tsx
<Stack.Navigator initialRouteName="Home">
  <Stack.Screen
    name="Home"
    component={HomeScreen}
    options={{ title: 'Welcome' }}
  />
  <Stack.Screen
    name="Profile"
    component={ProfileScreen}
    options={{
      title: 'User Profile',
      headerStyle: { backgroundColor: '#6200ee' },
      headerTintColor: '#ffffff',
    }}
  />
  <Stack.Screen
    name="Details"
    component={DetailsScreen}
    options={{ headerShown: false }}
  />
  <Stack.Screen
    name="Settings"
    component={SettingsScreen}
    options={{ title: 'Settings' }}
  />
</Stack.Navigator>
```

---

## Guided Exploration Exercises

### Exercise 1: Add a New Screen

1. Add a new screen called AboutScreen
2. Add it to the RootStackParamList type
3. Add it to Stack.Navigator
4. Add a button on the HomeScreen to navigate to AboutScreen

### Exercise 2: Pass Parameters

1. Update the Profile screen to expect a userId parameter
2. Pass a userId when navigating from Home to Profile
3. Display the userId on the Profile screen

### Exercise 3: Customise Headers

1. Change the title of each screen
2. Hide the header on the Details screen
3. Change the header colour on the Settings screen

### Exercise 4: Add Multiple Buttons

1. On the Home screen, add buttons to navigate to all other screens
2. On each screen, add a button to navigate back to Home

---

## Common Navigation Mistakes

### Mistake 1: Forgetting NavigationContainer

```tsx
// Wrong
<Stack.Navigator>
  <Stack.Screen name="Home" component={HomeScreen} />
</Stack.Navigator>

// Correct
<NavigationContainer>
  <Stack.Navigator>
    <Stack.Screen name="Home" component={HomeScreen} />
  </Stack.Navigator>
</NavigationContainer>
```

### Mistake 2: Navigating to a Screen That Doesn't Exist

```tsx
// Wrong (ScreenName does not exist in RootStackParamList)
navigation.navigate('WrongName')

// Correct (ScreenName exists in RootStackParamList)
navigation.navigate('Profile')
```

### Mistake 3: Using navigate when goBack is More Appropriate

```tsx
// Less efficient - creates a new instance of the screen
navigation.navigate('Profile')

// Better - returns to the previous screen
navigation.goBack()
```

---

## Summary

| Concept                      | Purpose                                          |
| ---------------------------- | ------------------------------------------------ |
| NavigationContainer          | Root component that provides navigation context  |
| Stack.Navigator              | Manages the stack of screens                     |
| Stack.Screen                 | Represents a single screen in the stack          |
| navigation.navigate()        | Navigates to another screen                      |
| navigation.goBack()          | Returns to the previous screen                   |
| navigation.push()            | Adds a new screen to the stack                   |
| navigation.replace()         | Replaces the current screen                      |
| RootStackParamList           | Type definition for screen names and parameters  |
| options                      | Customises the appearance of a screen            |

---

## Next Steps

You now understand how to implement Native Stack Navigation in React Native.

In the next lesson, we will explore other navigation patterns, including:

- Bottom Tab Navigation
- Drawer Navigation
- Material Top Tab Navigation
- Nested Navigation (combining different navigators)

---
