# Using Google Fonts in an Expo React Native App

> In-class practical activity · React Native · TypeScript · Expo

---

## Where this activity fits

You have already worked through UI design for your assignment (or another subject) and discussed typography in class. This activity moves from *design decisions* to *implementation*: choosing a font family, installing the correct package, loading the variants you actually need, and applying them through React Native styles in a TypeScript Expo project.

Starting point: `expo-template-blank-typescript`.

Scope: we will focus on **one** approach only — installing `@expo-google-fonts/*` packages and loading the required variants with the `useFonts` hook. Other approaches (local files, config plugins) are covered in the companion activity.

---

## What you should be able to do afterwards

- Explain why typography is an important part of UI design.
- Research and select suitable fonts for a mobile application.
- Identify the font variants needed for a particular design.
- Install Google Fonts packages in an Expo project.
- Import font resources using the correct names.
- Explain the purpose of the `useFonts` hook and its loading state.
- Use multiple font families in the same application without duplicating the hook.
- Apply the selected fonts through React Native styles.

---

## Part 1 — Homework and Design Research

Before opening the terminal, return to the UI design you have developed for the assignment or another subject.

Your first task is to decide which fonts your application needs. Do not begin by installing random fonts simply because they look interesting.

Typography should support the purpose of your application, its intended audience and the visual hierarchy of its screens.

### Step 1. Review your application design

Look at the screens you have designed and identify the different types of text used in the interface.

| UI element                    | Typography considerations                                                      |
| ----------------------------- | ------------------------------------------------------------------------------ |
| Main screen heading           | Should establish the identity of the screen and stand out clearly.             |
| Section headings              | Should distinguish sections from one another.                                  |
| Body text and descriptions    | Should be comfortable to read at mobile screen sizes.                          |
| Buttons and navigation labels | Should remain legible and easy to recognise.                                   |
| Supporting text               | May need a different size, weight or style to establish a secondary hierarchy. |

You do not necessarily need a different font family for every type of text. A well-chosen family with suitable weights and styles may be sufficient.

For your homework, identify the typography roles that matter most in your design and decide whether one or two font families would best support them.

### Step 2. Research suitable Google Fonts

Visit the Google Fonts website:

[Google Fonts — Font library](https://fonts.google.com/)

Explore fonts that suit the visual identity of your application. Consider whether your design calls for a clean sans-serif, a traditional serif, a decorative display font or another typographic style.

As you research, consider:

- **Readability** — Is the font comfortable to read on a small screen?
- **Personality** — Does it suit the intended audience and purpose of the application?
- **Hierarchy** — Can the font help distinguish headings from descriptions and other interface elements?
- **Consistency** — Will the chosen fonts work together across your screens?
- **Practicality** — Are the variants you need available in the Expo Google Fonts packages?

Select your font or fonts before moving on to installation.

### Step 3. Record your font choices

Complete the following notes before the practical activity.

**My typography research** — complete what you can. A second font is optional if one family meets your design needs.

**Font Family #1** 
[____________________]

*Intended use*

- [ ] Headings
- [ ] Body text and descriptions
- [ ] Buttons and navigation
- [ ] Multiple UI roles

*Variants to investigate*
[____________________]

---

**Font Family #2 (optional)**
[____________________]

*Intended use*

- [ ] Headings
- [ ] Body text and descriptions
- [ ] Buttons and navigation
- [ ] Multiple UI roles

*Variants to investigate*
[____________________]

---

You will use these decisions during the practical activity. You can change your selections if your research reveals a better option.

---

## Part 2 — Find the Font Package and its Variants

For this activity we use the Expo Google Fonts ecosystem. The Expo repository provides packages for many Google Fonts families, and the directory linked below makes it easier to find the correct package and variant names.

### Step 4. Open the font directory

Visit:

[Expo Google Fonts directory — search available fonts](https://directory-by-atiladev-com.netlify.app)

Search for the font family you selected during your research.

Select the font to view its available variants. You might see entries such as:

```text
Montserrat_400Regular
Montserrat_400Regular_Italic
Montserrat_700Bold
Montserrat_700Bold_Italic
```

The actual list depends on the family you selected.

> **Do not assume every font has the same variants.** A decorative font might only offer a regular style, while another family may provide several weights and italic variants.

### Step 5. Choose only the variants you need

Suppose your design uses Montserrat for descriptions and italic supporting text. You might select:

```text
Montserrat_400Regular
Montserrat_400Regular_Italic
```

You do not need a separate package for each variant. The *package* is for the font family; the *variants* are the individual font resources you choose to load and use.

Record the exact spelling and capitalisation of each variant. You will use those names in the import statements, the font-loading configuration and your styles.

---

## Part 3 — Install the required packages

### Step 6. Open your project in the terminal

Open the terminal in the root directory of your Expo project — the directory containing `package.json`.

First, install the font-loading library:

```bash
npx expo install expo-font
```

`expo-font` provides the functionality needed to load fonts into the application at *runtime* — that is, while the application is running, rather than when it is being built.

Next, install your chosen font family. For Montserrat (this would have been copied from the font directory website — see the `COPY INSTALLER` button):

```bash
npx expo install @expo-google-fonts/montserrat
```

For Alfa Slab One, the command would be:

```bash
npx expo install @expo-google-fonts/alfa-slab-one
```

If your design uses both families, install all three dependencies together:

```bash
npx expo install expo-font @expo-google-fonts/montserrat @expo-google-fonts/alfa-slab-one
```

**What does `npx expo install` do?**

It runs Expo's installation command, which adds the specified packages to your project and selects compatible package versions where applicable. The dependencies are recorded in `package.json`, so the project manages them as part of its development environment.

> **Common pitfall — package names.** The package names must be written correctly. For example, the package for Alfa Slab One is `@expo-google-fonts/alfa-slab-one`, *not* `@expo-google-fonts/Alfa Slab One`. Lowercase and hyphenation are required.

You only need to install the families your application actually uses.

---

## Part 4 — Import the font variants

### Step 7. Create the import statements

Return to the font directory and look at the import information for your selected family.

For this activity, begin with the following placeholder:

```tsx
import { } from '@expo-google-fonts/alfa-slab-one';
```

This is an incomplete import statement. The empty curly braces indicate where you must specify the named exports you want to import from the package.

Suppose you chose Alfa Slab One and the directory lists the variant `AlfaSlabOne_400Regular`. Your import becomes:

```tsx
import {
  AlfaSlabOne_400Regular,
} from '@expo-google-fonts/alfa-slab-one';
```

For Montserrat, you might write:

```tsx
import {
  Montserrat_400Regular,
  Montserrat_400Regular_Italic,
} from '@expo-google-fonts/montserrat';
```

The names inside the curly braces must match the exported identifiers exactly. Use the variants you recorded in Step 3 & Step 4.

Notice the distinction between the two parts of the statement:

- The names inside `{ }` identify the specific **font resources** being imported.
- The package path after `from` identifies the installed **font family package**.

You can import multiple variants from one package in the same statement. If your application uses multiple font families, create a separate import statement for each family.

> **Do not** import `useFonts` from every font package. We will use a single `useFonts` hook for the entire set of selected variants (see Part 6).

---

## Part 5 — Understand and use the `useFonts` hook

This is the central concept of the activity. Before copying the complete example, understand what the hook does.

### Step 8. Import the hook

Add the following import:

```tsx
import { useFonts } from 'expo-font';
```

`useFonts` is a function supplied by the `expo-font` library. It allows a React component to request that font resources be loaded and to determine when the requested fonts are ready.

> **What is a hook?** A React function that allows a component to use particular React functionality. Hooks follow specific rules — for example, they must be called at the top level of a function component, not inside a conditional statement or a loop.

### Step 9. Tell the hook which fonts to load

Inside your `App` component, write:

```tsx
const [fontsLoaded, fontError] = useFonts({
  Montserrat_400Regular,
  Montserrat_400Regular_Italic,
});
```

#### The object passed to `useFonts`

The object contains the variants the application needs. In this example, we request regular and italic Montserrat.

JavaScript allows shorthand property names when the property and variable share the same name. Therefore:

```tsx
{
  Montserrat_400Regular,
  Montserrat_400Regular_Italic,
}
```

is equivalent to:

```tsx
{
  Montserrat_400Regular: Montserrat_400Regular,
  Montserrat_400Regular_Italic: Montserrat_400Regular_Italic,
}
```

The imported font resources are the *values* supplied to the hook. Their **property names** are used as the `fontFamily` identifiers when we apply the fonts in our styles.

#### The values returned by the hook

The hook returns an array. We use array destructuring to obtain the values we need:

- `fontsLoaded` — a Boolean indicating whether the requested fonts have loaded successfully.
- `fontError` — provides information if font loading fails.

You will often see examples that use only the first value:

```tsx
const [fontsLoaded] = useFonts({
  Montserrat_400Regular,
});
```

That is valid when the second value is not needed. In our example we retain both values so we can distinguish *loading* from *failure*.

### Step 10. Wait for the fonts before rendering

We do not want the application to attempt to display text using fonts that are not ready yet.

A simple approach:

```tsx
if (fontError) {
  return null;
}

if (!fontsLoaded) {
  return null;
}
```

- The first condition checks for a loading error.
- The second prevents the component from rendering its normal interface until the fonts are ready.

> **Production note.** Returning `null` on an error is not ideal in a published application — it can leave the user looking at a blank screen. A polished implementation should display an error message or another appropriate fallback. We keep this example focused on the loading concept.

The important point is that `useFonts` does **not** immediately return a Boolean indicating success on the first render. Font loading is asynchronous, and React can render the component again when the loading state changes.

---

## Part 6 — What if the application needs more than one font family?

Suppose your application uses Montserrat for descriptions and Alfa Slab One for headings.

You do **not** need one `useFonts` call for Montserrat and another for Alfa Slab One.

Instead, import the required variants from both packages and place them in the **same object**:

```tsx
import { useFonts } from 'expo-font';

import {
  Montserrat_400Regular,
  Montserrat_400Regular_Italic,
} from '@expo-google-fonts/montserrat';

import {
  AlfaSlabOne_400Regular,
} from '@expo-google-fonts/alfa-slab-one';
```

Then load all the selected variants together:

```tsx
const [fontsLoaded, fontError] = useFonts({
  Montserrat_400Regular,
  Montserrat_400Regular_Italic,
  AlfaSlabOne_400Regular,
});
```

There is only **one** hook call, **one** loading state and **one** loading-error value. The object contains font resources from both families.

This is the solution to the potential conflict you may have identified earlier.

> **Why not import `useFonts` from both families?** If you write two imports that both declare a local identifier named `useFonts`, the module will have a duplicate declaration. That is a JavaScript import problem — not a conflict between Montserrat and Alfa Slab One. Using `useFonts` from `expo-font` avoids that confusion and keeps the separation between the loading library and font resources clear.

You can now apply either font to any suitable text element in the component.

---

## Part 7 — Complete the practical activity

### Step 11. Implement the fonts in `App.tsx`

The following example combines the concepts from this activity. It uses two font families, but you can adapt it to use your own selected font or fonts.

```tsx
import { Text, View, StyleSheet } from 'react-native';

import { useFonts } from 'expo-font';

import {
  Montserrat_400Regular,
  Montserrat_400Regular_Italic,
} from '@expo-google-fonts/montserrat';

import {
  AlfaSlabOne_400Regular,
} from '@expo-google-fonts/alfa-slab-one';

export default function App() {
  const [fontsLoaded, fontError] = useFonts({
    Montserrat_400Regular,
    Montserrat_400Regular_Italic,
    AlfaSlabOne_400Regular,
  });

  if (fontError) {
    return (
      <View style={styles.container}>
        <Text>There was a problem loading the fonts.</Text>
      </View>
    );
  }

  if (!fontsLoaded) {
    return null;
  }

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>
        My Mobile App
      </Text>

      <Text style={styles.description}>
        This text uses Montserrat.
      </Text>

      <Text style={styles.emphasis}>
        This text uses italic Montserrat.
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 24,
  },

  heading: {
    fontFamily: 'AlfaSlabOne_400Regular',
    fontSize: 28,
    marginBottom: 16,
  },

  description: {
    fontFamily: 'Montserrat_400Regular',
    fontSize: 16,
    textAlign: 'center',
    marginBottom: 12,
  },

  emphasis: {
    fontFamily: 'Montserrat_400Regular_Italic',
    fontSize: 14,
  },
});
```

> **Remember:** install both font packages and `expo-font` before running this example.

If you choose different families, replace the example imports, the entries in the `useFonts` object, and the corresponding `fontFamily` values in your styles. The names must match your selected variants.

### Step 12. Run and inspect the application

Run your Expo application using the normal command for your project, for example:

```bash
npx expo start
```

Check the rendered interface on your target device or simulator.

Confirm the following:

- The heading displays using Alfa Slab One.
- The description displays using regular Montserrat.
- The supporting text displays using italic Montserrat.
- No font-loading or missing-font errors appear in the terminal or application.
- The selected fonts improve the visual hierarchy without reducing readability.

If the application fails to load, check the package installation, import paths, variant spelling and the `fontFamily` names. A font imported into the file still needs to be included in the `useFonts` object before it is available through this runtime-loading approach.

---

## Part 8 — Reflection and extension

Once you have a working example, return to your assignment UI and apply your chosen typography to your own screens.

Use these questions to reflect on your implementation:

1. Why did you choose your font family or families?
2. Which variants did you select, and why were those variants necessary?
3. What role does `useFonts` play in the application?
4. Why is it useful to load all the required variants in one hook call?
5. What would happen if you used a font name in `fontFamily` that was not loaded?
6. Does your choice of typography improve the readability and consistency of the interface?

You should be able to explain the decisions behind your implementation, not simply demonstrate that the code runs.

---

## Further reading

- [Expo Google Fonts — GitHub repository](https://github.com/expo/google-fonts) — package usage, font variants and examples.
- [Expo Google Fonts directory](https://directory-by-atiladev-com.netlify.app) — search for font families and their exported variant names.
- [Expo documentation — Fonts](https://docs.expo.dev/develop/user-interface/fonts/) — broader guidance on font loading and other supported approaches.

---
