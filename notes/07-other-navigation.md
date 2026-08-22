# MAST5112: Mobile App Scripting – Class Notes

## 07 - Other Navigation Patterns

---

## Introduction

In the previous lesson, we learned how to implement Native Stack Navigation. Stack navigation is the foundation of most React Native applications, but it is not the only navigation pattern available.

Real-world applications often require more than one navigation pattern. Consider a typical social media app: you might have tabs at the bottom for Home, Search, Notifications, and Profile. Within the Home tab, you might have a stack for navigating to individual posts. You might also have a side menu (drawer) for accessing settings or account options.

In this lesson, we will explore three additional navigation patterns:

1. Bottom Tab Navigation
2. Drawer Navigation
3. Material Top Tab Navigation

We will also examine how these navigators can be combined through nested navigation.

By the end of this lesson, you will understand when to use each navigation type and how to combine them effectively.

---

## Choosing the Right Navigation Pattern

Before we examine each navigation type, it is useful to understand the factors that influence which pattern to use.

### Consider the User's Mental Model

Different navigation patterns suggest different mental models to the user.

| Navigation Type     | User Mental Model                                  | Best For                                        |
| ------------------- | -------------------------------------------------- | ----------------------------------------------- |
| Stack               | "I am moving deeper into content"                  | Drill-down experiences, forms, detail views     |
| Bottom Tabs         | "I have several main sections of the app"          | Primary app sections, frequent switching        |
| Drawer              | "I need to access secondary or infrequent items"   | Settings, account, secondary sections           |
| Top Tabs            | "I am viewing different views of the same content" | Filtering, sorting, content segmentation        |

### Consider the Screen Layout

The physical space available on a mobile screen is limited. Navigation elements should not dominate the interface.

| Navigation Type     | Screen Space Used                       | Visual Impact                                   |
| ------------------- | --------------------------------------- | ----------------------------------------------- |
| Stack               | Minimal (back button in header only)    | Content-focused                                 |
| Bottom Tabs         | Significant (fixed bar at bottom)       | Always visible, high discoverability            |
| Drawer              | Variable (slides over content)          | Hidden until revealed, minimal default impact   |
| Top Tabs            | Moderate (tabs below header)            | Content-focused with segmentation               |

---

## 1. Bottom Tab Navigation

### What Is Bottom Tab Navigation?

Bottom Tab Navigation displays a row of tabs at the bottom of the screen. Each tab represents a different section of the application. Tapping a tab switches to the corresponding screen.

This navigation pattern is one of the most common in mobile applications because it provides easy access to the app's primary sections.

### When to Use Bottom Tab Navigation

Bottom Tab Navigation is most effective when:

- The application has between 3 and 5 primary sections
- Users need to switch between sections frequently
- The sections are of equal importance
- You want the navigation to be always visible

### When Not to Use Bottom Tab Navigation

Bottom Tab Navigation is less effective when:

- The application has more than 5 primary sections (tabs become too small)
- The sections have a hierarchical relationship (stack is better)
- You need to conserve screen space for content

### Sample Scenario: Social Media App

A social media app typically has:

- Home (feed of posts)
- Search (find people or content)
- Notifications (alerts and updates)
- Profile (user's own profile)

These four sections are equally important and frequently accessed. Bottom tabs allow users to switch between them quickly.

### Installation

To use Bottom Tab Navigation, install the required package:

```bash
npm install @react-navigation/bottom-tabs
```

### Code Example

```tsx
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { Text, View, StyleSheet } from 'react-native';

type TabParamList = {
  Home: undefined;
  Search: undefined;
  Notifications: undefined;
  Profile: undefined;
};

const Tab = createBottomTabNavigator<TabParamList>();

function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home</Text>
    </View>
  );
}

function SearchScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Search</Text>
    </View>
  );
}

function NotificationsScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Notifications</Text>
    </View>
  );
}

function ProfileScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Profile</Text>
    </View>
  );
}

export default function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator>
        <Tab.Screen name="Home" component={HomeScreen} />
        <Tab.Screen name="Search" component={SearchScreen} />
        <Tab.Screen name="Notifications" component={NotificationsScreen} />
        <Tab.Screen name="Profile" component={ProfileScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: '#f0f4f7',
  },
  title: {
    fontSize: 24,
    color: '#333',
  },
});
```

### Customising Bottom Tabs

You can customise the appearance of bottom tabs using the screenOptions prop.

```tsx
<Tab.Navigator
  screenOptions={{
    tabBarActiveTintColor: '#6200ee',
    tabBarInactiveTintColor: '#666',
    tabBarStyle: {
      backgroundColor: '#ffffff',
      borderTopWidth: 1,
      borderTopColor: '#e0e0e0',
    },
    headerStyle: {
      backgroundColor: '#6200ee',
    },
    headerTintColor: '#ffffff',
  }}
>
  {/* Screens here */}
</Tab.Navigator>
```

### Key Props for Bottom Tabs

| Prop                     | Purpose                                        |
| ------------------------ | ---------------------------------------------- |
| `tabBarActiveTintColor`  | Colour of active tab icon and label            |
| `tabBarInactiveTintColor`| Colour of inactive tab icon and label          |
| `tabBarStyle`            | Styles for the tab bar itself                  |
| `headerStyle`            | Styles for the header bar                      |
| `headerTintColor`        | Colour of header text and icons                |

---

## 2. Drawer Navigation

### What Is Drawer Navigation?

Drawer Navigation provides a side menu that slides in from the left edge of the screen. The menu contains navigation options that are hidden until the user reveals them.

This pattern is commonly used for secondary navigation items or for applications that need to present a large number of navigation options.

### When to Use Drawer Navigation

Drawer Navigation is most effective when:

- The application has secondary sections that are accessed less frequently
- You want to preserve screen space for content
- The navigation options are numerous (more than 5)
- You want to include additional content in the menu (user profile, branding)

### When Not to Use Drawer Navigation

Drawer Navigation is less effective when:

- The primary sections need to be immediately visible
- Users need to switch between sections frequently
- The application has a simple structure with only a few screens

### Sample Scenario: Settings and Account Access

A shopping app might have a bottom tab bar for browsing products, but a drawer for:

- Account settings
- Order history
- Wishlist
- Help and support
- Logout

These items are accessed less frequently than the primary browsing experience, making them ideal for a drawer.

### Installation

To use Drawer Navigation, install the required packages:

```bash
npm install @react-navigation/drawer
npx expo install react-native-gesture-handler react-native-reanimated
```

### Code Example

```tsx
import { NavigationContainer } from '@react-navigation/native';
import { createDrawerNavigator } from '@react-navigation/drawer';
import { Text, View, StyleSheet } from 'react-native';

type DrawerParamList = {
  Home: undefined;
  Settings: undefined;
  Orders: undefined;
  Wishlist: undefined;
  Help: undefined;
};

const Drawer = createDrawerNavigator<DrawerParamList>();

function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home</Text>
    </View>
  );
}

function SettingsScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Settings</Text>
    </View>
  );
}

function OrdersScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Order History</Text>
    </View>
  );
}

function WishlistScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Wishlist</Text>
    </View>
  );
}

function HelpScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Help & Support</Text>
    </View>
  );
}

export default function App() {
  return (
    <NavigationContainer>
      <Drawer.Navigator>
        <Drawer.Screen name="Home" component={HomeScreen} />
        <Drawer.Screen name="Orders" component={OrdersScreen} />
        <Drawer.Screen name="Wishlist" component={WishlistScreen} />
        <Drawer.Screen name="Settings" component={SettingsScreen} />
        <Drawer.Screen name="Help" component={HelpScreen} />
      </Drawer.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: '#f0f4f7',
  },
  title: {
    fontSize: 24,
    color: '#333',
  },
});
```

### Opening and Closing the Drawer

In a screen component, you can control the drawer using navigation methods:

```tsx
function HomeScreen({ navigation }: { navigation: any }) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home</Text>
      <Button title="Open Drawer" onPress={() => navigation.openDrawer()} />
      <Button title="Close Drawer" onPress={() => navigation.closeDrawer()} />
      <Button title="Toggle Drawer" onPress={() => navigation.toggleDrawer()} />
    </View>
  );
}
```

### Customising the Drawer

You can customise the drawer's appearance:

```tsx
<Drawer.Navigator
  screenOptions={{
    drawerActiveTintColor: '#6200ee',
    drawerInactiveTintColor: '#666',
    drawerStyle: {
      backgroundColor: '#ffffff',
      width: 280,
    },
    headerStyle: {
      backgroundColor: '#6200ee',
    },
    headerTintColor: '#ffffff',
  }}
>
  {/* Screens here */}
</Drawer.Navigator>
```

### Custom Drawer Content

For more control, you can create a custom drawer component:

```tsx
import {
  DrawerContentScrollView,
  DrawerItemList,
  DrawerItem,
} from '@react-navigation/drawer';

function CustomDrawerContent(props: any) {
  return (
    <DrawerContentScrollView {...props}>
      <View style={{ padding: 20, backgroundColor: '#6200ee' }}>
        <Text style={{ color: '#ffffff', fontSize: 18, fontWeight: 'bold' }}>
          My App
        </Text>
      </View>
      <DrawerItemList {...props} />
      <DrawerItem
        label="Logout"
        onPress={() => {
          // Handle logout
          console.log('Logout pressed');
        }}
      />
    </DrawerContentScrollView>
  );
}

// Usage in Drawer.Navigator
<Drawer.Navigator drawerContent={(props) => <CustomDrawerContent {...props} />}>
  {/* Screens here */}
</Drawer.Navigator>
```

### Key Props for Drawer Navigation

| Prop                     | Purpose                                        |
| ------------------------ | ---------------------------------------------- |
| `drawerActiveTintColor`  | Colour of active item in the drawer            |
| `drawerInactiveTintColor`| Colour of inactive items in the drawer         |
| `drawerStyle`            | Styles for the drawer container                |
| `drawerContent`          | Custom component for the drawer content        |
| `headerStyle`            | Styles for the header bar                      |

---

## 3. Material Top Tab Navigation

### What Is Material Top Tab Navigation?

Material Top Tab Navigation displays tabs at the top of the screen, below the header. Users can switch between tabs by tapping or swiping horizontally.

This pattern is commonly used for grouping related content within a single screen.

### When to Use Material Top Tab Navigation

Material Top Tab Navigation is most effective when:

- You need to display different views of the same content
- The content sections are related and of equal importance
- You want to support swipe gestures between views
- The content is best consumed in a horizontal scrolling pattern

### When Not to Use Material Top Tab Navigation

Material Top Tab Navigation is less effective when:

- The tabs need to be visible at the bottom for thumb reach
- The content sections are unrelated
- The screen is already crowded with other UI elements

### Sample Scenario: Product Details

A product details screen might have:

- Overview (product description)
- Reviews (customer feedback)
- Specifications (technical details)
- Related Products (similar items)

These are all related to the same product and are best presented as top tabs, allowing users to swipe between views.

### Installation

To use Material Top Tab Navigation, install the required package:

```bash
npm install @react-navigation/material-top-tabs
npx expo install react-native-pager-view
```

### Code Example

```tsx
import { NavigationContainer } from '@react-navigation/native';
import { createMaterialTopTabNavigator } from '@react-navigation/material-top-tabs';
import { Text, View, StyleSheet } from 'react-native';

type TabParamList = {
  Overview: undefined;
  Reviews: undefined;
  Specs: undefined;
  Related: undefined;
};

const Tab = createMaterialTopTabNavigator<TabParamList>();

function OverviewScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Product Overview</Text>
      <Text style={styles.description}>
        This is the product description. It provides details about the product.
      </Text>
    </View>
  );
}

function ReviewsScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Customer Reviews</Text>
      <Text style={styles.description}>★★★★☆ (4.5)</Text>
      <Text style={styles.description}>Great product! Highly recommended.</Text>
    </View>
  );
}

function SpecsScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Technical Specifications</Text>
      <Text style={styles.description}>Weight: 200g</Text>
      <Text style={styles.description}>Dimensions: 10 x 5 x 2 cm</Text>
    </View>
  );
}

function RelatedScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Related Products</Text>
      <Text style={styles.description}>Similar items you might like.</Text>
    </View>
  );
}

export default function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator
        screenOptions={{
          tabBarActiveTintColor: '#6200ee',
          tabBarInactiveTintColor: '#666',
          tabBarIndicatorStyle: {
            backgroundColor: '#6200ee',
          },
          tabBarStyle: {
            backgroundColor: '#ffffff',
          },
        }}
      >
        <Tab.Screen name="Overview" component={OverviewScreen} />
        <Tab.Screen name="Reviews" component={ReviewsScreen} />
        <Tab.Screen name="Specs" component={SpecsScreen} />
        <Tab.Screen name="Related" component={RelatedScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 20,
    backgroundColor: '#f0f4f7',
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 12,
  },
  description: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
  },
});
```

### Key Props for Material Top Tabs

| Prop                     | Purpose                                        |
| ------------------------ | ---------------------------------------------- |
| `tabBarActiveTintColor`  | Colour of active tab label                     |
| `tabBarInactiveTintColor`| Colour of inactive tab labels                  |
| `tabBarIndicatorStyle`   | Styles for the indicator line under active tab |
| `tabBarStyle`            | Styles for the tab bar container               |
| `swipeEnabled`           | Whether swipe gestures are enabled             |

---

## 4. Nested Navigation

### What Is Nested Navigation?

Nested navigation occurs when one navigator is placed inside a screen of another navigator .

For example, you might have a Bottom Tab Navigator with a Home tab. Inside the Home tab, you have a Stack Navigator that handles navigation to detail screens. This is a common and powerful pattern.

### Visual Representation

```text
    Bottom Tab Navigator (Parent)
        │
        ├──────────┬──────────┬─────────────┐
        ↓          ↓          ↓             ↓
    Home Tab    Search Tab  Notifications  Profile Tab
    (Stack)                  Tab
        │
        ├──────────┬───────────┐
        ↓          ↓           ↓
        Home       Post        Post
        Screen     Details     Comments
```

When a user taps a post on the Home Screen, they are pushed to the Post Details Screen. This is handled by the Stack Navigator inside the Home tab.

### Sample Scenario: Combined Navigation

A typical social media app uses a combination of navigators:

1. **Bottom Tabs** for primary sections: Home, Search, Notifications, Profile
2. **Stack Navigator** inside each tab for drill-down navigation
3. **Drawer Navigator** for secondary sections: Settings, Help, Logout

This combination provides a powerful and flexible navigation experience.

### Code Example: Bottom Tabs with Stack Inside Each Tab

```tsx
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { Text, View, StyleSheet, Button } from 'react-native';

type HomeStackParamList = {
  Home: undefined;
  PostDetails: { postId: string };
};

type ProfileStackParamList = {
  Profile: undefined;
  EditProfile: undefined;
};

const HomeStack = createNativeStackNavigator<HomeStackParamList>();
const ProfileStack = createNativeStackNavigator<ProfileStackParamList>();
const Tab = createBottomTabNavigator();

// Home Stack Screens
function HomeScreen({ navigation }: { navigation: any }) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home Feed</Text>
      <Button
        title="View Post"
        onPress={() => navigation.navigate('PostDetails', { postId: '123' })}
      />
    </View>
  );
}

function PostDetailsScreen({ route }: { route: any }) {
  const { postId } = route.params;
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Post Details</Text>
      <Text style={styles.description}>Post ID: {postId}</Text>
    </View>
  );
}

// Profile Stack Screens
function ProfileScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Profile</Text>
    </View>
  );
}

function EditProfileScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Edit Profile</Text>
    </View>
  );
}

// Home Stack Navigator
function HomeStackNavigator() {
  return (
    <HomeStack.Navigator>
      <HomeStack.Screen name="Home" component={HomeScreen} />
      <HomeStack.Screen name="PostDetails" component={PostDetailsScreen} />
    </HomeStack.Navigator>
  );
}

// Profile Stack Navigator
function ProfileStackNavigator() {
  return (
    <ProfileStack.Navigator>
      <ProfileStack.Screen name="Profile" component={ProfileScreen} />
      <ProfileStack.Screen name="EditProfile" component={EditProfileScreen} />
    </ProfileStack.Navigator>
  );
}

// Main App
export default function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator>
        <Tab.Screen name="HomeTab" component={HomeStackNavigator} />
        <Tab.Screen name="ProfileTab" component={ProfileStackNavigator} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}

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
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 12,
  },
  description: {
    fontSize: 16,
    color: '#666',
  },
});
```

### Nested Navigation with Drawer and Material Top Tabs

Another common pattern combines a Drawer Navigator with Material Top Tabs inside one of the screens:

```text
    Drawer Navigator (Parent)
        │
        ├──────────┬──────────┬──────────┐
        ↓          ↓          ↓          ↓
        Home     Settings   Help       Logout
        (Top Tabs)
        │
        ├──────────┬───────────┐
        ↓          ↓           ↓
        Overview   Reviews     Specs
```

### Important Considerations for Nested Navigation 

#### Each Navigator Has Its Own History

When you nest navigators, each maintains its own navigation history. This means that pressing the back button inside a nested stack will go back within that stack before affecting the parent navigator.

#### Navigation Actions Bubble Up

If a navigator cannot handle a navigation action, it will bubble up to the parent navigator . For example, calling `navigate('Settings')` from inside a tab might be handled by a parent drawer navigator if the tab navigator does not contain a screen named Settings.

#### Navigator-Specific Methods Are Available in Children

When a navigator is nested inside another, the child screens have access to methods from both navigators . For example, a screen inside a stack that is nested inside a drawer can call `navigation.openDrawer()`.

#### Parent UI Renders on Top

When you nest a stack navigator inside a drawer navigator, the drawer appears above the stack's header . When you nest a drawer inside a stack, the drawer appears below the stack's header. This is an important consideration when deciding how to structure your navigation.

### Common Nested Navigation Patterns 

| Pattern                                | Description                                                                         |
| -------------------------------------- | ----------------------------------------------------------------------------------- |
| Tab Navigator inside Stack Navigator   | New screens cover the tab bar when you push them.                                   |
| Stack Navigator inside Tab Navigator   | The tab bar is always visible. Pressing the tab again pops the stack to the top.    |
| Drawer Navigator inside Stack Navigator | The drawer can only be opened from the first screen of the stack.                   |
| Stack Navigator inside Drawer Navigator | The drawer appears over the header from the stack.                                  |

---

## Choosing the Right Combination

The choice of navigation combination depends on your application's structure and user needs.

### Common Combinations and Their Use Cases

| Combination                                      | Use Case                                                                          |
| ------------------------------------------------ | --------------------------------------------------------------------------------- |
| Bottom Tabs with Stack inside each tab           | Social media apps, e-commerce apps, apps with multiple primary sections            |
| Drawer with Stack inside each drawer item        | Apps with a large number of secondary sections, admin panels                     |
| Stack with Top Tabs inside a screen              | Detail screens with multiple views (product details, analytics)                   |
| Bottom Tabs with Drawer overlay                  | Apps that need both primary section access and secondary navigation               |

### Decision Framework

When designing your navigation structure, ask yourself:

1. What are the primary sections of my application? (Bottom Tabs or Drawer)
2. Do any sections require drill-down navigation? (Stack inside those sections)
3. Do I need to group related content within a single section? (Top Tabs inside a screen)
4. What is the best way for users to access secondary features? (Drawer)

---

## Summary Table

| Navigation Type     | Best For                                      | Installation Command                               |
| ------------------- | --------------------------------------------- | -------------------------------------------------- |
| Bottom Tab          | Primary sections, frequent switching          | `npm install @react-navigation/bottom-tabs`       |
| Drawer              | Secondary sections, infrequent access         | `npm install @react-navigation/drawer`            |
| Material Top Tab    | Related views within a screen                 | `npm install @react-navigation/material-top-tabs` |

### Key Concepts

| Concept             | Description                                                                 |
| ------------------- | --------------------------------------------------------------------------- |
| Nested Navigation   | Placing one navigator inside a screen of another navigator  |
| Navigation History  | Each nested navigator maintains its own history                 |
| Action Bubbling     | Navigation actions bubble up to parent navigators if not handled  |
| Parent UI Rendering | Parent navigator UI renders on top of child navigator          |

---

## Guided Exploration Exercises

### Exercise 1: Bottom Tabs with Icons

1. Add icons to your bottom tabs using react-native-vector-icons
2. Customise the tab bar with different colours
3. Add a badge to one of the tabs (e.g., notification count)

### Exercise 2: Drawer with Custom Content

1. Create a custom drawer component
2. Add a user profile section at the top of the drawer
3. Add a logout button at the bottom

### Exercise 3: Top Tabs with Swipe

1. Create a top tab navigator with three tabs
2. Enable swipe gestures between tabs
3. Customise the tab indicator colour

### Exercise 4: Nested Navigation

1. Create a bottom tab navigator with three tabs
2. In the first tab, add a stack navigator with two screens
3. In the second tab, add a material top tab navigator with three screens
4. Navigate between screens and observe the behaviour

### Exercise 5: Navigation Combination

1. Create a drawer navigator as the root
2. Inside one drawer item, add a bottom tab navigator
3. Inside one tab, add a stack navigator
4. Test navigation between all levels

---

## Next Steps

You now understand the major navigation patterns in React Native and how to combine them through nested navigation.

In future lessons, you will combine these navigation skills with state management, API integration, and user authentication to build complete applications.

Remember that navigation is about more than moving between screens. It is about creating an intuitive user experience that helps users achieve their goals. Choose your navigation patterns carefully and test them with actual users.