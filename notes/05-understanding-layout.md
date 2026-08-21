# MAST5112: Mobile App Scripting – Class Notes

## 05 - Understanding Layout

---

## Introduction

In the previous lessons, we built interactive applications using components such as View, Text, TextInput, and Button. We applied basic styles using StyleSheet and used layout props like flex, justifyContent, and alignItems.

However, we have only scratched the surface of what is possible with React Native layout.

Layout determines how content is arranged on the screen. It controls where components appear, how they are spaced, how they respond to different screen sizes, and how they look.

In this lesson, we will explore the most important layout concepts in React Native:

1. Flexbox Fundamentals
2. Positioning
3. Spacing (Margin and Padding)
4. Sizing and Dimensions
5. Borders and Shadows
6. Transforms
7. Overflow and Clipping

By the end of this lesson, you will understand not only how these styles work, but when and why to use them.

---

## Layout Concepts in React Native

Before we look at individual concepts, it is useful to understand the relationship between layout and the components we have already learned.

Every component in React Native has a style prop. The style prop accepts an object containing layout properties.

```tsx
<View style={{ flex: 1, justifyContent: 'center' }}>
  <Text style={{ fontSize: 20 }}>Hello</Text>
</View>
```

The layout properties control how the component is positioned, sized, and spaced relative to other components.

These properties are based on CSS Flexbox, but they are implemented in JavaScript and applied to native views.

---

## 1. Flexbox Fundamentals

Flexbox is the most important layout system in React Native. It provides a way to arrange components in rows or columns and distribute space between them.

### The Main Flexbox Props

| Prop            | Purpose                                    | Common Values                     |
| --------------- | ------------------------------------------ | --------------------------------- |
| flexDirection   | Sets the direction of the layout           | 'row', 'column' (default)         |
| justifyContent  | Distributes space along the main axis      | 'center', 'flex-start', 'flex-end', 'space-between', 'space-around' |
| alignItems      | Distributes space along the cross axis     | 'center', 'flex-start', 'flex-end', 'stretch' |
| flexWrap        | Controls whether items wrap to the next line | 'nowrap' (default), 'wrap'      |
| flex            | Controls how a component grows or shrinks  | 1, 2, etc.                        |

### Understanding the Axes

The main axis is determined by flexDirection.

```text
flexDirection: 'column' (default)
    ↓
Main axis is vertical (top to bottom)
Cross axis is horizontal (left to right)

flexDirection: 'row'
    ↓
Main axis is horizontal (left to right)
Cross axis is vertical (top to bottom)
```

### ASCII Visualisation

Consider three boxes arranged in a row:

```text

flexDirection: 'row'
justifyContent: 'flex-start'
alignItems: 'center'

┌─────────────────────────────────────────┐
│                                         │
│    ┌───────┐  ┌───────┐  ┌───────┐      │
│    │ Box 1 │  │ Box 2 │  │ Box 3 │      │
│    └───────┘  └───────┘  └───────┘      │
│                                         │
│       ↑            ↑             ↑      │
│       └──── vertically centred ──┘      │
│                                         │
└─────────────────────────────────────────┘


flexDirection: 'row'
justifyContent: 'space-between'
alignItems: 'center'

┌───────────────────────────────────────────────┐
│                                               │
│  ┌───────┐        ┌───────┐        ┌───────┐  │
│  │ Box 1 │        │ Box 2 │        │ Box 3 │  │
│  └───────┘        └───────┘        └───────┘  │
│                                               │
│       ←────── evenly distributed ──────→      │
│                                               │
└───────────────────────────────────────────────┘
```

### Example Code

```tsx
function FlexDemo() {
  return (
    <View
      style={{
        flexDirection: 'row',           // Arrange horizontally
        justifyContent: 'space-around', // Space items evenly
        alignItems: 'center',           // Center vertically
        height: 150,
        backgroundColor: '#E0F7FA',
      }}
    >
      <View style={[styles.box, { backgroundColor: '#80DEEA' }]} />
      <View style={[styles.box, { backgroundColor: '#4DD0E1' }]} />
      <View style={[styles.box, { backgroundColor: '#26C6DA' }]} />
    </View>
  );
}
```

### Try It Yourself

Change the following values and observe the result:

- flexDirection: 'column' instead of 'row'
- justifyContent: 'center' instead of 'space-around'
- alignItems: 'flex-start' instead of 'center'

Each change affects how the boxes are arranged.

---

## 2. Positioning

Positioning controls where a component appears on the screen. There are two main positioning modes: relative and absolute.

### Relative Positioning (Default)

With relative positioning, the component is placed in the normal layout flow. Other components are positioned relative to it.

### Absolute Positioning

With absolute positioning, the component is removed from the normal layout flow and positioned relative to its parent container.

This is useful for:

- Overlays
- Tooltips
- Floating action buttons
- Badges on icons
- Positioning elements at specific screen coordinates

### ASCII Visualisation

```text
Relative Positioning (default)

┌─────────────────────────────────────────┐
│                                         │
│  ┌───────────────┐                      │
│  │     Text      │                      │
│  └───────────────┘                      │
│                                         │
│  ┌───────────────┐                      │
│  │      Box      │                      │
│  └───────────────┘                      │
│                                         │
│  ┌───────────────┐                      │
│  │     Text      │                      │
│  └───────────────┘                      │
│                                         │
└─────────────────────────────────────────┘
        ↓
Elements follow the normal layout flow

Absolute Positioning

┌────────────────────────────────────────────┐
│  ┌───────────────┐                         │
│  │     Text      │                         │
│  │   top-left    │                         │
│  └───────────────┘                         │
│                                            │
│                                            │
│                 ┌───────────────┐          │
│                 │      Box      │          │
│                 │    centre     │          │
│                 └───────────────┘          │
│                                            │
│                                            │
│                         ┌───────────────┐  │
│                         │     Text      │  │
│                         │  bottom-right │  │
│                         └───────────────┘  │
└────────────────────────────────────────────┘
        ↓
Elements are positioned independently
```

### Example Code

```tsx
function PositionDemo() {
  return (
    <View style={{ width: 200, height: 150, backgroundColor: '#FFF3E0' }}>
      <Text style={{ position: 'absolute', top: 10, left: 10 }}>
        Top-Left
      </Text>
      <Text style={{ position: 'absolute', bottom: 10, right: 10 }}>
        Bottom-Right
      </Text>
      <View style={{
        width: 50,
        height: 50,
        backgroundColor: '#FFB74D',
        position: 'absolute',
        top: 50,
        left: 70,
      }} />
    </View>
  );
}
```

### Key Points

- Use top, left, right, and bottom to position absolute elements
- These values are relative to the nearest parent with positioning
- zIndex controls stacking order when elements overlap

---

## 3. Spacing: Margin and Padding

Spacing is essential for creating clean, readable interfaces. React Native provides two types of spacing:

| Property  | Purpose                                       |
| --------- | --------------------------------------------- |
| margin    | Adds space outside the component              |
| padding   | Adds space inside the component               |

### Individual and Shorthand Properties

You can set spacing individually:

```tsx
marginTop: 10
marginBottom: 10
marginLeft: 10
marginRight: 10
paddingTop: 10
paddingBottom: 10
```

Or using shorthand:

```tsx
margin: 10              // All sides
marginHorizontal: 10    // Left and right
marginVertical: 10      // Top and bottom
padding: 10             // All sides
paddingHorizontal: 10   // Left and right
paddingVertical: 10     // Top and bottom
```

### ASCII Visualisation

```text
      ┌──────────────────────────────┐
      │            MARGIN            │
      │     Space OUTSIDE the box    │
      │                              │
      │   ┌──────────────────────┐   │
      │   │        PADDING       │   │
      │   │   Space INSIDE the   │   │
      │   │        box           │   │
      │   │                      │   │
      │   │    ┌────────────┐    │   │
      │   │    │  CONTENT   │    │   │
      │   │    │            │    │   │
      │   │    └────────────┘    │   │
      │   │                      │   │
      │   └──────────────────────┘   │
      │                              │
      └──────────────────────────────┘

        MARGIN
          ↓
   moves the box away
   from OTHER elements


        PADDING
          ↓
   moves the CONTENT away
   from the EDGE of its box
```

### Example Code

```tsx
function SpacingDemo() {
  return (
    <View style={{ flexDirection: 'row', backgroundColor: '#E8F5E9' }}>
      <View style={[styles.box, { backgroundColor: '#A5D6A7', margin: 10 }]} />
      <View style={[styles.box, { backgroundColor: '#81C784', padding: 20 }]} >
        <Text>Padding</Text>
      </View>
    </View>
  );
}
```

### When to Use Margin vs Padding

- Use margin to separate different components (e.g., spacing between cards in a list)
- Use padding to give content breathing room inside a component (e.g., spacing inside a button)

---

## 4. Sizing and Dimensions

Sizing determines how large a component appears on the screen. There are several ways to set size.

### Fixed Size

Use width and height with pixel values:

```tsx
width: 100
height: 100
```

### Percentage Size

Use percentage values relative to the parent container:

```tsx
width: '80%'
height: '50%'
```

### Aspect Ratio

Use aspectRatio to maintain proportions:

```tsx
width: '80%'
aspectRatio: 2    // Width is twice the height
```

### Min and Max Sizes

Use minWidth, maxWidth, minHeight, and maxHeight to set constraints:

```tsx
minWidth: 50
maxWidth: 200
```

### ASCII Visualisation

```text
Fixed Size
width: 100
height: 100

        100
    ←─────────→
    ┌─────────┐
    │         │
    │         │
100 │   100 × │ 100
    │   100   │
    │         │
    │         │
    └─────────┘
    ←─────────→
        100

    Width  = 100
    Height = 100

Aspect Ratio
aspectRatio: 2

        2 parts
    ←───────────────→
    ┌────────────────┐
    │                │
    │                │
    │     2 : 1      │  1 part
    │                │
    │                │
    └────────────────┘
    ←───────────────→

    Width : Height
       2  :   1

    The width is always
    twice the height.
```

### Example Code

```tsx
function SizingDemo() {
  return (
    <View style={{ width: '80%', aspectRatio: 2, backgroundColor: '#F3E5F5' }}>
      <Text>80% width, aspectRatio 2</Text>
    </View>
  );
}
```

### Real-World Use

- Percentage widths for responsive containers
- aspectRatio for image galleries and media previews
- min/max sizes to ensure components are usable on all screens

---

## 5. Borders and Shadows

Borders and shadows add visual depth and polish to your UI.

### Border Properties

```tsx
borderWidth: 4
borderColor: '#9575CD'
borderRadius: 12
```

### Individual Border Radius

```tsx
borderTopLeftRadius: 12
borderTopRightRadius: 12
borderBottomLeftRadius: 12
borderBottomRightRadius: 12
```

### Shadows (iOS)

```tsx
shadowColor: '#000'
shadowOffset: { width: 0, height: 4 }
shadowOpacity: 0.3
shadowRadius: 4
```

### Elevation (Android)

```tsx
elevation: 6
```

### Platform Differences

| Feature        | iOS                  | Android             |
| -------------- | -------------------- | ------------------- |
| Shadows        | shadow* properties   | elevation           |
| Appearance     | Soft, coloured       | Hard, grey          |

### Example Code

```tsx
function BorderDemo() {
  return (
    <View
      style={{
        width: 100,
        height: 100,
        borderWidth: 4,
        borderColor: '#9575CD',
        borderRadius: 12,
        // iOS shadow
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 4 },
        shadowOpacity: 0.3,
        shadowRadius: 4,
        // Android elevation
        elevation: 6,
        backgroundColor: '#FFFFFF',
      }}
    />
  );
}
```

### Real-World Use

- Borders: Input fields, cards, avatars
- Shadows: Cards, modals, elevated buttons
- Border radius: Buttons, avatars, image containers

---

## 6. Transforms

Transforms allow you to rotate, scale, translate, or skew components.

### Transform Functions

```tsx
transform: [
  { translateX: 10 },
  { translateY: 10 },
  { scale: 1.2 },
  { rotate: '45deg' },
]
```

### Multiple Transforms

```tsx
transform: [
  { rotate: '45deg' },
  { scale: 1.2 },
]
```

### Example Code

```tsx
function TransformDemo() {
  return (
    <View style={{ alignItems: 'center' }}>
      <View
        style={{
          width: 80,
          height: 80,
          backgroundColor: '#FFCDD2',
          transform: [
            { rotate: '45deg' },
            { scale: 1.2 },
          ],
        }}
      />
    </View>
  );
}
```

### Real-World Use

- rotate: Spinning icons, loading spinners, decorative elements
- scale: Hover or press animations, image zoom
- translateX/Y: Sliding menus, animated transitions

---

## 7. Overflow and Clipping

Overflow controls what happens when content is larger than its container.

### Overflow Values

```tsx
overflow: 'visible'   // Content can extend outside
overflow: 'hidden'    // Content is clipped to container bounds
```

### ASCII Visualisation

```text
overflow: 'visible'

    ┌──────────────────────┐
    │      Container       │
    │                      │
    │    ┌───────────────────────┐
    │    │        Child          │
    │    │                       │
    └────┼───────────────────────┤
         │                       │
         │  Child remains        │
         │  visible outside      │
         │  the container        │
         └───────────────────────┘

overflow: 'hidden'

    ┌──────────────────────┐
    │      Container       │
    │    ┌─────────────────┤
    │    │      Child      │
    │    │                 │
    └────┴─────────────────┘
             ↑
             │
        Container boundary
        clips anything
        outside this area
```

### Example Code

```tsx
function OverflowDemo() {
  return (
    <View style={{
      width: 120,
      height: 120,
      overflow: 'hidden',
      backgroundColor: '#C8E6C9',
    }}>
      <View style={{ width: 200, height: 200, backgroundColor: '#66BB6A' }} />
    </View>
  );
}
```

### Real-World Use

- overflow: hidden for image masks and rounded corners
- overflow: visible for tooltips and dropdowns
- Clipping for scrollable containers and animations

---

## Complete Reference Code

The following code combines all seven layout concepts into a single application that can be copied into App.tsx.

```tsx
import { View, Text, StyleSheet, ScrollView } from 'react-native';

export default function App() {
  return (
    <ScrollView contentContainerStyle={styles.appContainer}>
      <DemoSection title="1. Flexbox Fundamentals">
        <FlexDemo />
      </DemoSection>

      <DemoSection title="2. Positioning">
        <PositionDemo />
      </DemoSection>

      <DemoSection title="3. Spacing">
        <SpacingDemo />
      </DemoSection>

      <DemoSection title="4. Sizing & Dimensions">
        <SizingDemo />
      </DemoSection>

      <DemoSection title="5. Borders & Shadows">
        <BorderDemo />
      </DemoSection>

      <DemoSection title="6. Transforms">
        <TransformDemo />
      </DemoSection>

      <DemoSection title="7. Overflow & Clipping">
        <OverflowDemo />
      </DemoSection>
    </ScrollView>
  );
}

function DemoSection({ title, children }: any) {
  return (
    <View style={styles.section}>
      <Text style={styles.sectionTitle}>{title}</Text>
      <View style={styles.demoBox}>{children}</View>
    </View>
  );
}

// 1. Flexbox Fundamentals
function FlexDemo() {
  return (
    <View
      style={{
        flexDirection: 'row',
        justifyContent: 'space-around',
        alignItems: 'center',
        height: 150,
        backgroundColor: '#E0F7FA',
      }}
    >
      <View style={[styles.box, { backgroundColor: '#80DEEA' }]} />
      <View style={[styles.box, { backgroundColor: '#4DD0E1' }]} />
      <View style={[styles.box, { backgroundColor: '#26C6DA' }]} />
    </View>
  );
}

// 2. Positioning
function PositionDemo() {
  return (
    <View style={{ width: 200, height: 150, backgroundColor: '#FFF3E0' }}>
      <Text style={{ position: 'absolute', top: 10, left: 10 }}>Top-Left</Text>
      <Text style={{ position: 'absolute', bottom: 10, right: 10 }}>Bottom-Right</Text>
      <View style={{
        width: 50,
        height: 50,
        backgroundColor: '#FFB74D',
        position: 'absolute',
        top: 50,
        left: 70,
      }} />
    </View>
  );
}

// 3. Spacing
function SpacingDemo() {
  return (
    <View style={{ flexDirection: 'row', backgroundColor: '#E8F5E9' }}>
      <View style={[styles.box, { backgroundColor: '#A5D6A7', margin: 10 }]} />
      <View style={[styles.box, { backgroundColor: '#81C784', padding: 20 }]} >
        <Text>Padding</Text>
      </View>
    </View>
  );
}

// 4. Sizing & Dimensions
function SizingDemo() {
  return (
    <View style={{ width: '80%', aspectRatio: 2, backgroundColor: '#F3E5F5' }}>
      <Text>80% width, aspectRatio 2</Text>
    </View>
  );
}

// 5. Borders & Shadows
function BorderDemo() {
  return (
    <View
      style={{
        width: 100,
        height: 100,
        borderWidth: 4,
        borderColor: '#9575CD',
        borderRadius: 12,
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 4 },
        shadowOpacity: 0.3,
        shadowRadius: 4,
        elevation: 6,
        backgroundColor: '#FFFFFF',
      }}
    />
  );
}

// 6. Transforms
function TransformDemo() {
  return (
    <View style={{ alignItems: 'center' }}>
      <View
        style={{
          width: 80,
          height: 80,
          backgroundColor: '#FFCDD2',
          transform: [
            { rotate: '45deg' },
            { scale: 1.2 },
          ],
        }}
      />
    </View>
  );
}

// 7. Overflow & Clipping
function OverflowDemo() {
  return (
    <View style={{
      width: 120,
      height: 120,
      overflow: 'hidden',
      backgroundColor: '#C8E6C9',
    }}>
      <View style={{ width: 200, height: 200, backgroundColor: '#66BB6A' }} />
    </View>
  );
}

const styles = StyleSheet.create({
  appContainer: {
    padding: 16,
    backgroundColor: '#FAFAFA',
  },
  section: {
    marginBottom: 24,
  },
  sectionTitle: {
    fontSize: 18,
    marginBottom: 8,
  },
  demoBox: {
    padding: 12,
    backgroundColor: '#FFFFFF',
    borderRadius: 6,
    elevation: 2,
  },
  box: {
    width: 40,
    height: 40,
  },
});
```

---

## Guided Exploration Exercises

Try modifying each demo to see how the layout changes.

### Exercise 1: Flexbox

Change the following values in FlexDemo:

1. Change flexDirection from 'row' to 'column'. How does the arrangement change?
2. Change justifyContent from 'space-around' to 'flex-start' or 'center'.
3. Change alignItems from 'center' to 'flex-start' or 'flex-end'.

### Exercise 2: Positioning

In PositionDemo:

1. Add a fourth element positioned at top-right.
2. Change the top and left values of the box.
3. Add zIndex to control which element appears on top.

### Exercise 3: Spacing

In SpacingDemo:

1. Add marginHorizontal: 20 to one box.
2. Add paddingVertical: 30 to the other box.
3. Observe how the sizes and positions change.

### Exercise 4: Sizing

In SizingDemo:

1. Change width from '80%' to '50%'.
2. Change aspectRatio from 2 to 1.
3. Add minWidth: 100 to see how constraints work.

### Exercise 5: Borders and Shadows

In BorderDemo:

1. Change borderRadius from 12 to 24.
2. Change shadowOpacity from 0.3 to 0.6.
3. Change elevation from 6 to 12 (Android only).

### Exercise 6: Transforms

In TransformDemo:

1. Change rotate from '45deg' to '90deg'.
2. Change scale from 1.2 to 1.5.
3. Add translateX: 20 to move the box.

### Exercise 7: Overflow

In OverflowDemo:

1. Change overflow from 'hidden' to 'visible'.
2. Observe how the child view now extends outside the parent.

---

## Layout Properties Summary Table

| Category            | Properties                                    | Purpose                               |
| ------------------- | --------------------------------------------- | ------------------------------------- |
| Flexbox             | flexDirection, justifyContent, alignItems     | Arranging components                  |
| Flexbox             | flex, flexWrap                                | Grow/shrink behaviour                 |
| Positioning         | position, top, left, right, bottom            | Exact placement                       |
| Positioning         | zIndex                                        | Stacking order                        |
| Spacing             | margin, marginTop, marginHorizontal           | Space outside component               |
| Spacing             | padding, paddingTop, paddingHorizontal        | Space inside component                |
| Sizing              | width, height                                 | Fixed size                            |
| Sizing              | aspectRatio                                   | Maintain proportions                  |
| Sizing              | minWidth, maxWidth, minHeight, maxHeight      | Size constraints                      |
| Borders             | borderWidth, borderColor, borderRadius        | Visual boundaries                     |
| Shadows (iOS)       | shadowColor, shadowOffset, shadowOpacity      | Depth and elevation                   |
| Elevation (Android) | elevation                                     | Depth and elevation                   |
| Transforms          | translateX, translateY, scale, rotate         | Visual effects                        |
| Overflow            | overflow                                      | Content clipping                      |

---

## Next Steps

You now understand the most important layout concepts in React Native.

In future lessons, you will combine these layout skills with components, state, and navigation to build complete applications.

Remember that layout is not just about code. It is about crafting user experiences. Experiment with the examples, try new combinations, and observe how small changes affect the overall feel of the interface.

---