# Using Local Font Files in an Expo React Native App

> Practical activity · React Native · TypeScript · Expo · Local font assets
> Companion activity to *Using Google Fonts in an Expo React Native App*. 

---

## Where this activity fits

In the previous activity we explored how to select Google Fonts and use the `@expo-google-fonts` packages to load font variants into an Expo application.

There is another approach: download the font files yourself, store them in your project, and reference those files directly.

This approach is useful when:

- You have selected a font from Google Fonts and want to keep the actual font files inside your project.
- A designer has supplied font files for your application.
- You want to use a custom font that is not available through an Expo Google Fonts package.
- You want to manage the exact font files that form part of your application.

In this tutorial we will use an Expo project created from the `expo-template-blank-typescript` template. We will download font files, organise them in a local assets directory, configure the project, and load the fonts in our TypeScript code.

We will also examine an important distinction: **configuring fonts in `app.json`** and **loading fonts with `require()`** are two different approaches. They should not be mixed together without understanding the purpose of each.

---

## What you should be able to do afterwards

- Download suitable font files and check their variants.
- Create and organise a local font directory.
- Install the Expo font library.
- Configure the Expo font config plugin in `app.json`.
- Load local fonts at runtime using `useFonts` and `require()`.
- Apply local font variants through React Native styles.
- Explain the difference between build-time font configuration and runtime font loading.

---

## Part 1 — Research and download your font files

### Step 1. Choose a font for your application

Before downloading anything, revisit your application's UI design.

Consider which text elements need a custom font. You might choose a single font family with different weights, or two complementary families for different roles.

You can browse Google Fonts here:

[Google Fonts — Browse font families](https://fonts.google.com/)

If you are using Google Fonts, select the family you want and examine its available styles. For example, you might decide to use:

- Montserrat Regular for body text.
- Montserrat Bold for headings.
- Montserrat Italic for supporting text.

You do not need to download every available style. Select only the variants that serve a purpose in your design.

### Step 2. Download the font files

Download the font files for your selected variants. Depending on the font family and download source, these may be provided as `.ttf` or `.otf` files.

For this tutorial, assume you have downloaded the following files:

```text
Montserrat-Regular.ttf
Montserrat-Bold.ttf
Montserrat-Italic.ttf
```

These are illustrative filenames. Your actual downloaded files may have different names — use the filenames you really have.

> **Important.** Do not simply rename a font file to make it appear to be a different weight or style. A regular font file does not become a genuine bold or italic font merely because you rename it.

Also check the licence associated with your chosen font, especially if you plan to distribute or publish the application.

### Step 3. Organise the files in your project

Inside the project root, create the following directory structure if it does not already exist:

```text
MyExpoApp/
├── app.json
├── App.tsx
├── package.json
└── assets/
    └── fonts/
        ├── Montserrat-Regular.ttf
        ├── Montserrat-Bold.ttf
        └── Montserrat-Italic.ttf
```

The conventional directory name is `assets/fonts/`. Expo allows other directory structures, but using a consistent convention makes projects easier to understand and maintain.

Make sure the font files are **physically inside** this folder. Creating the folder alone does not add any fonts to the application.

---

## Part 2 — Install the font-loading library

### Step 4. Install `expo-font`

Open a terminal at the root of your project and run:

```bash
npx expo install expo-font
```

This installs the Expo font library.

We will use it in two different ways in this exercise:

1. The **runtime approach**, where our TypeScript code loads local files using `require()`.
2. The **config plugin approach**, where `app.json` tells Expo to embed the font files during native builds.

These are *alternative* ways to make local fonts available. You generally do not need to configure the same fonts using both methods.

---

## Part 3 — Method A: Load local font files in TypeScript

For this method we work directly in `App.tsx`. It is particularly useful for learning how font loading works, because the font resources and the loading process are visible in the code.

### Step 5. Import `useFonts`

At the top of `App.tsx`, add:

```tsx
import { useFonts } from 'expo-font';
```

You will also need the usual React Native components:

```tsx
import { Text, View, StyleSheet } from 'react-native';
```

### Step 6. Use `require()` to reference the font files

Unlike the previous activity, where we imported named font resources from a Google Fonts package, this time we refer directly to files in our project.

```tsx
const [fontsLoaded, fontError] = useFonts({
  'Montserrat-Regular': require('./assets/fonts/Montserrat-Regular.ttf'),
  'Montserrat-Bold': require('./assets/fonts/Montserrat-Bold.ttf'),
  'Montserrat-Italic': require('./assets/fonts/Montserrat-Italic.ttf'),
});
```

Let's understand the two parts of each entry. For example:

```tsx
'Montserrat-Regular': require('./assets/fonts/Montserrat-Regular.ttf')
```

- `'Montserrat-Regular'` is the **name we assign** to the loaded font resource. We use this name in our `fontFamily` styles.
- `require('./assets/fonts/Montserrat-Regular.ttf')` tells the JavaScript bundler to include the local font file as a project dependency.

The path is relative to `App.tsx`. If your component is inside a different directory, you must adjust the relative path accordingly.

> **Path and bundling rules.** The filename and path must match the actual file in your project. Also, `require()` must reference a *statically identifiable* path — do not try to construct the filename dynamically from a variable.

### Step 7. Understand the loading state

The hook returns two values:

```tsx
const [fontsLoaded, fontError] = useFonts({
  'Montserrat-Regular': require('./assets/fonts/Montserrat-Regular.ttf'),
});
```

- `fontsLoaded` indicates whether the requested font has finished loading.
- `fontError` contains an error if loading fails.

Because loading is asynchronous, the component should account for both loading and failure before displaying text that depends on the custom font.

For this introductory example we display a simple error message if loading fails, and wait while the font is loading:

```tsx
if (fontError) {
  return (
    <View>
      <Text>Unable to load the font.</Text>
    </View>
  );
}

if (!fontsLoaded) {
  return null;
}
```

These checks must appear **inside** the component — after the hook call and before the normal UI is returned.

### Step 8. Apply the local fonts in your styles

Once the fonts are loaded, you can use their assigned names in your `StyleSheet`:

```tsx
const styles = StyleSheet.create({
  heading: {
    fontFamily: 'Montserrat-Bold',
    fontSize: 28,
  },

  description: {
    fontFamily: 'Montserrat-Regular',
    fontSize: 16,
  },

  emphasis: {
    fontFamily: 'Montserrat-Italic',
    fontSize: 14,
  },
});
```

Notice that the names in `fontFamily` match the **keys** we used in the `useFonts` object. They do not have to match the original filenames, because we assigned those names ourselves.

This is a useful difference from the previous activity. With Google Fonts packages, we generally use the exported variant identifier as the font-family name. With local font files, we can choose a meaningful name when defining the mapping.

### Step 9. Complete the application

Here is a complete example combining the local font imports, loading state, UI and styles:

```tsx
import { Text, View, StyleSheet } from 'react-native';
import { useFonts } from 'expo-font';

export default function App() {
  const [fontsLoaded, fontError] = useFonts({
    'Montserrat-Regular': require(
      './assets/fonts/Montserrat-Regular.ttf'
    ),
    'Montserrat-Bold': require(
      './assets/fonts/Montserrat-Bold.ttf'
    ),
    'Montserrat-Italic': require(
      './assets/fonts/Montserrat-Italic.ttf'
    ),
  });

  if (fontError) {
    return (
      <View style={styles.container}>
        <Text>Unable to load the custom fonts.</Text>
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
        This text uses the regular font.
      </Text>

      <Text style={styles.emphasis}>
        This text uses the italic font.
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
    fontFamily: 'Montserrat-Bold',
    fontSize: 28,
    marginBottom: 16,
  },

  description: {
    fontFamily: 'Montserrat-Regular',
    fontSize: 16,
    textAlign: 'center',
    marginBottom: 12,
  },

  emphasis: {
    fontFamily: 'Montserrat-Italic',
    fontSize: 14,
  },
});
```

This example assumes the three named files exist in `assets/fonts/`. If your downloaded files use different names, change the paths accordingly.

For this method, the font-loading code is what makes the local fonts available at runtime. You do not need to add these same files to the `expo-font` config plugin in `app.json` as well.

---

## Part 4 — Method B: Configure local fonts in `app.json`

Now let's explore the other approach mentioned in the introduction.

The Expo font config plugin allows font files to be **embedded in the native application during the build process**. Instead of waiting for your TypeScript component to load the font at runtime, Expo configures the native project to include the font.

This is useful when you want the fonts to be available immediately when the application starts. However, there is an important limitation: the config plugin does **not** work in Expo Go. It requires a native development build or another appropriate native build.

> **Note for this course.** We do not use this method, because our projects are built from an Expo-based template and executed through the Expo Go app. Method B is included so you understand what the alternative is and when a real project would use it.

### Step 10. Open `app.json`

An Expo project typically has an `app.json` file in its root directory. Its contents may look something like this:

```json
{
  "expo": {
    "name": "MyExpoApp",
    "slug": "my-expo-app",
    "version": "1.0.0"
  }
}
```

Your file will probably contain additional properties. Do **not** replace the entire file with the example above — we only want to *add* the font plugin configuration to your existing Expo settings.

### Step 11. Add the `expo-font` config plugin

Inside the existing `"expo"` object, add a `"plugins"` entry:

```json
{
  "expo": {
    "name": "MyExpoApp",
    "slug": "my-expo-app",
    "version": "1.0.0",
    "plugins": [
      [
        "expo-font",
        {
          "fonts": [
            "./assets/fonts/Montserrat-Regular.ttf",
            "./assets/fonts/Montserrat-Bold.ttf",
            "./assets/fonts/Montserrat-Italic.ttf"
          ]
        }
      ]
    ]
  }
}
```

This is an illustrative configuration. Retain your project's actual name, slug, version and other settings when making the change.

Examining the configuration:

- `"plugins"` — tells Expo which config plugins to run.
- `"expo-font"` — identifies the font plugin.
- `"fonts"` — an array of paths to font files.
- Each path is **relative to the project root**, where `app.json` is located.

The plugin uses these paths to configure the native application to include the fonts. It is **not** a runtime instruction to `App.tsx`.

### Step 12. Understand when a new build is needed

Changing `app.json` does **not** automatically modify an already-installed native application.

After adding the config plugin, you need to create and install a **new development build** or another appropriate native build for the changes to take effect.

For example, if you use an Android development build, follow your project's Expo development-build workflow rather than expecting the configuration change to take effect simply by refreshing Expo Go.

Also note that font family names can differ between platforms when fonts are embedded through the config plugin. On **Android**, the filename is used as the font family name in the simple configuration. On **iOS**, the font's *internal* name may be used instead. If you use this approach in a cross-platform application, verify the correct family name for the fonts and platforms you target.

### Step 13. Use the fonts in your TypeScript code

When using the config plugin, you do not need the `useFonts` hook to load the same fonts at runtime.

Your component can apply the embedded fonts directly:

```tsx
import { Text, View, StyleSheet } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.heading}>
        My Mobile App
      </Text>

      <Text style={styles.description}>
        This is my custom font.
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },

  heading: {
    fontFamily: 'Montserrat-Bold',
    fontSize: 28,
  },

  description: {
    fontFamily: 'Montserrat-Regular',
    fontSize: 16,
  },
});
```

The font-family names shown here are illustrative. In a real project, confirm the actual font names recognised by each target platform. If the native build does not recognise the names, the fonts may not appear as expected.

Notice that this version has no `useFonts` import, no `require()` calls and no font-loading condition. The config plugin has a different responsibility: embedding fonts in the native application **before** it runs.

---

## Part 5 — Compare the two local-font approaches

Both approaches use font files stored locally in the project, but they differ in how those files become available to the application.

| Consideration               | Method A: `useFonts` and `require()`                  | Method B: `expo-font` config plugin                                |
| --------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------ |
| Font files                  | Stored in `assets/fonts/` or another chosen directory | Stored locally and referenced in `app.json`                        |
| Configuration               | TypeScript code                                       | `app.json`                                                         |
| When fonts become available | Loaded asynchronously at runtime                      | Embedded during native build                                       |
| Loading state               | Check `fontsLoaded` and handle errors                 | No runtime font-loading hook required for embedded fonts           |
| Expo Go                     | Supported                                             | Not supported for config-plugin embedding                          |
| Web support                 | Supported                                             | Config plugin is for native platforms; use runtime loading for web |
| New native build            | Not normally needed just to add a runtime-loaded font | Required for the native configuration change to take effect        |

These distinctions follow Expo's guidance on local fonts and the font config plugin.

---

## Part 6 — Troubleshooting checklist

If your fonts are not displaying correctly, work through these checks.

- [ ] The font files are in the correct directory.
- [ ] The filenames and file extensions match the actual files.
- [ ] The `require()` paths are relative to the TypeScript file and point to the correct location.
- [ ] The font resources are included in the `useFonts` object when using Method A.
- [ ] The `fontFamily` values match the names assigned in `useFonts`.
- [ ] The `app.json` configuration is valid JSON when using Method B.
- [ ] A new native development build has been created when using the config plugin.
- [ ] The font licence permits the intended use.

---

## Part 7 — Reflection and application

Once you have successfully implemented your fonts, apply the approach to your own assignment UI.

Consider the following questions:

1. Why might a developer choose to store font files locally rather than install a Google Fonts package?
2. What does `require()` do in the runtime-loading example?
3. Why do we need to wait for `fontsLoaded` before displaying the interface?
4. What is the difference between the font-loading approach in `App.tsx` and the config plugin in `app.json`?
5. Why might a developer choose the config plugin for a published application?
6. Which approach is most suitable for your current development environment, and why?

### Final note

For the practical work, use **Method A**, since we are using Expo Go. It keeps the font files, `require()` paths, `useFonts` hook and styles together in one place, making it easier to understand the entire loading process.

**Method B** is an important alternative to be introduced to, to understand local assets. It demonstrates that fonts can also be incorporated into the native application during the build process.

Although both methods use the same local font files, they are **alternative loading strategies** — not consecutive steps that every student must perform. Understanding that distinction will help you choose the right approach for your own applications.

For reference, the official documentation is available here: [Expo — Fonts](https://docs.expo.dev/develop/user-interface/fonts/).

---
