# Running an Expo React Native Project in BlueStacks 5

> **Student Reference Guide — Institutional Windows VM**

---

## 1. Purpose

This guide explains how to run an **Expo React Native application** inside the **BlueStacks 5 Android emulator** provided in the institutional Windows Virtual Machine.

Our development environment uses:

* Visual Studio Code
* Node.js and npm
* Expo SDK 54
* Android SDK / ADB
* BlueStacks 5
* Expo Go

The process described below allows you to:

1. Start your React Native project.
2. Start the Expo development server.
3. Open the project in Expo Go.
4. Run the application inside BlueStacks.
5. See changes to your code reflected in the application.

---

# 2. Important: Our BlueStacks Workflow

In a normal Expo setup, you may see instructions telling you to:

```text
npx expo start
```

and then press:

```text
a
```

to open the application on Android.

**For our institutional VM environment, do not use `a` to launch the application.**

We found that Expo's automatic Android launch does not work reliably with the BlueStacks configuration provided in our VM.

Instead, we will:

```text
Start BlueStacks
       ↓
Connect BlueStacks using ADB
       ↓
Start Expo / Metro
       ↓
Open Expo Go manually
       ↓
Enter the Expo URL
       ↓
Application loads in BlueStacks
```

This is our **recommended working procedure**.

---

# 3. Step 1 — Open BlueStacks

Start **BlueStacks 5** from the Windows desktop.

Wait for the Android phone interface to **fully load**.

Do not proceed while BlueStacks is still starting.

You should eventually be able to see the normal Android home screen.

---

# 4. Step 2 — Check Expo Go

Before attempting to run your project, make sure that **Expo Go** is working correctly inside BlueStacks.

Open:

**Play Store**

Sign in using your **own Google Account** if required.

> **Important:** Use your own account. Do not use another student's Google Account.

---

## 4.1 Find Expo Go

Search the Play Store for:

```text
Expo Go
```

---

## 4.2 Uninstall Expo Go

If Expo Go is already installed:

1. Open the Expo Go application page.
2. Select **Uninstall**.
3. Wait for the application to be removed.

---

## 4.3 Reinstall Expo Go

Install **Expo Go** again from the Play Store.

Wait for the installation to complete.

---

## 4.4 Open Expo Go

Open the application.

**Do not continue until Expo Go has successfully opened.**

You should be able to see the Expo Go **Home** screen, including the option to enter an Expo project URL.

You should see something similar to:

```text
Expo Go

Projects

Enter URL
[________________________]

       or

Scan QR Code
```

The exact appearance may differ depending on the version of Expo Go.

### Important

The purpose of this step is to confirm that **Expo Go itself is working correctly before we attempt to connect it to your project**.

If Expo Go remains stuck on a blank/white screen and does not reach its Home screen, **stop here and ask your lecturer for assistance**.

---

# 5. Step 3 — Connect BlueStacks using ADB

Once BlueStacks is fully running and Expo Go has opened successfully, locate the **ADB connection batch file** provided for the module.

Run the `.bat` file.

The script will connect BlueStacks to ADB.

You should see a successful connection message referring to:

```text
127.0.0.1:5555
```

You do **not** need to change the port number.

You do **not** need to edit any BlueStacks configuration files.

---

# 6. Step 4 — Open Your Project in Visual Studio Code

Open your React Native / Expo project in **Visual Studio Code**.

Make sure that the VS Code terminal is positioned in the **root folder of your project**.

For example:

```text
MyReactNativeApp
│
├── app
├── assets
├── node_modules
├── package.json
└── ...
```

Your terminal should be working from:

```text
MyReactNativeApp>
```

or the equivalent path for your project.

---

# 7. Step 5 — Start Expo

In the VS Code terminal, execute:

```bash
npx expo start
```

Wait for Expo to start the development server.

You should see information about the Expo development server and a URL.

You may see something similar to:

```text
› Metro waiting on
  exp://10.0.0.4:8081
```

Your address may be different.

### Important

**Do not press `a`.**

Instead, look at the URL displayed by Expo.

---

# 8. Step 6 — Find Your Expo URL

Look in the VS Code terminal for the URL beginning with:

```text
exp://
```

For example:

```text
exp://10.0.0.4:8081
```

### Important

Your URL may **not** be:

```text
exp://10.0.0.4:8081
```

The IP address can be different.

**Always use the URL displayed by your own Expo development server.**

---

# 9. Step 7 — Open Expo Go

Return to **BlueStacks**.

Open:

**Expo Go**

You should already have confirmed that Expo Go reaches its Home screen during Step 2.

---

# 10. Step 8 — Enter the Expo URL

In Expo Go, locate the option to enter a project URL.

Enter the complete URL displayed by Expo.

For example:

```text
exp://10.0.0.4:8081
```

Then select the appropriate option to open/connect to the project.

---

# 11. Step 9 — Wait for Your Application to Load

Expo Go will connect to the Expo development server running in your VM.

You may see the loading/build process in the VS Code terminal.

For example:

```text
Building JavaScript bundle...
████████████████████
```

The application should then appear inside BlueStacks.

If you have just created a new blank Expo project, you should see the default Expo template.

---

# 12. Step 10 — Test That Everything Is Working

Now let's perform a simple test.

Open your project's source code in VS Code.

Locate the default:

```tsx
<Text>...</Text>
```

Change the text so that it displays your name.

For example:

```tsx
<Text>My name is Jessel</Text>
```

Save the file.

Return to BlueStacks.

The application should update automatically.

You should now see your new text displayed in Expo Go.

This confirms that the complete development workflow is working:

```text
Your Code
    ↓
Visual Studio Code
    ↓
Expo / Metro
    ↓
Expo Go
    ↓
BlueStacks
    ↓
Your Updated Application
```

---

# 13. Your Standard Workflow

Once everything has been configured and tested, your normal development process is:

```text
1. Open BlueStacks
        ↓
2. Wait for Android to load
        ↓
3. Open Play Store / confirm Expo Go is working
        ↓
4. Open Expo Go and confirm the Home screen appears
        ↓
5. Run the ADB connection .bat file
        ↓
6. Open your project in VS Code
        ↓
7. Run:
   npx expo start
        ↓
8. DO NOT press "a"
        ↓
9. Copy/note the exp:// URL
        ↓
10. Open Expo Go
        ↓
11. Enter the exp:// URL
        ↓
12. Application loads
        ↓
13. Edit your code
        ↓
14. Save
        ↓
15. Changes appear in BlueStacks
```

---

# 14. If Something Goes Wrong

## Expo reports an error involving port 5554

If you see an error similar to:

```text
could not connect to TCP port 5554
```

this is the issue we encountered with the automatic Android launch.

### Do not:

* change the BlueStacks ADB port;
* edit `bluestacks.conf`;
* delete Android SDK files;
* create another Android emulator;
* reinstall Android Studio.

Instead:

1. Press `Ctrl + C` to stop Expo.
2. Start Expo again:

```bash
npx expo start
```

3. **Do not press `a`.**
4. Use the `exp://...` URL shown by Expo.
5. Open Expo Go manually.
6. Enter the URL.

---

## Expo Go remains on a white screen

First check that:

* BlueStacks has completely finished loading.
* You are connected to the internet.
* Expo Go was installed successfully.

If necessary, close Expo Go and open it again.

You should reach the Expo Go Home screen before attempting to connect to your project.

If Expo Go still does not load correctly, **ask your lecturer for assistance rather than changing the VM configuration.**

---

## The project does not load

Check:

### Is Expo still running?

The terminal running:

```bash
npx expo start
```

must remain open.

### Are you using the correct URL?

Use the `exp://...` address currently displayed by **your own Expo terminal**.

### Is BlueStacks running?

BlueStacks must remain open.

### Did you run the ADB connection script?

If necessary, run the provided `.bat` file again.

---

# 15. Important: Do Not Change the VM Configuration

The institutional VM has been prepared with the software required for this subject.

Students should **not** modify:

```text
BlueStacks configuration files
ADB configuration
ADB ports
Android SDK configuration
Android Studio emulator settings
```

unless specifically instructed to do so by the lecturer.

In particular, **do not change the BlueStacks `adb_port` setting**.

The working procedure described in this guide does not require you to do this.

---

# 16. Why Are We Using This Method?

The problem we encountered is related to the way Expo attempts to automatically identify and launch an Android emulator when the `a` command is used.

In our institutional VM environment, that automatic process does not reliably work with the BlueStacks configuration.

However, the important part is that **the development server itself works correctly**.

By starting Expo manually and then connecting **Expo Go** to the URL provided by Expo, we avoid the problematic automatic Android-launch step.

The result is still a complete React Native development workflow:

```text
VS Code
  │ Your React Native code
  ↓
Expo / Metro
  │ Development server
  ↓
Expo Go
  ↓
BlueStacks 5
```

And because Expo Go remains connected to the development server, you can continue editing your code and seeing your changes reflected in the application.

---

# 17. Quick Reference

When you are in a hurry, remember:

### Start BlueStacks

Wait for Android to completely load.

### Check Expo Go

Open Expo Go and make sure the **Home** screen appears.

### Connect ADB

Run the provided:

```text
ADB connection .bat
```

### Start Expo

```bash
npx expo start
```

### **DO NOT PRESS `a`**

Find the:

```text
exp://...
```

URL displayed by Expo.

### Open Expo Go

Enter the `exp://...` URL.

### Run your application

Wait for the project to load.

### Edit → Save → See the change

Your changes should appear in BlueStacks through Expo Go.

---

## The one sequence students should remember

```text
BlueStacks
    ↓
Expo Go
    ↓
ADB .bat
    ↓
VS Code
    ↓
npx expo start
    ↓
DO NOT press "a"
    ↓
Copy exp:// URL
    ↓
Expo Go
    ↓
Enter URL
    ↓
Application runs
```

**This is the working procedure for running our Expo React Native projects inside BlueStacks 5 in the institutional VM.**
