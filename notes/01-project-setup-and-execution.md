# MAST5112: Mobile App Scripting – Class Notes
 
## 01 - Project Setup and Execution
---

## Preface

As discussed in class, we will use the **pre-configured Virtual Machine (VM)** to create and run our React Native projects.

The VM already contains the software and configuration required for our development environment. This allows us to focus on learning React Native rather than spending our first lessons solving installation and configuration problems.

Once you are confident with creating and running projects inside the VM, you can consult the **installation guide** in the **Stages RN TS** book if you wish to configure your own development environment.

---

# Creating a New React Native Project

We will use the **Expo framework** to create and run our React Native applications.

For this module, we will use a TypeScript-based Expo project.

The process below is the standard process we will use when creating our class projects.

> **Important:** This procedure is intended for our small-scale class projects. Follow the steps in order and do not change the project structure unless instructed to do so.

---

## Understanding Our Development Environment

Before creating the project, it is useful to understand the main parts of the environment we will be using.

* **Visual Studio Code** — where we write and edit our code.
* **Terminal** — where we run commands to create and start our projects.
* **Expo** — provides the development tools used to create and run our React Native applications.
* **Expo Go** — provides a development environment in which our Expo project can run.
* **BlueStacks** — provides a virtual Android device that we can use to test our application.

The basic workflow is:

```text
Write code
    ↓
Save code
    ↓
Expo detects the change
    ↓
Application refreshes
    ↓
Test the result
```

This development cycle will become very familiar as we work through the module.

---

## Step 1: Organise Your Project Files

1. Open **Windows Explorer**.
2. Navigate to your `Documents` folder.
3. Inside `Documents`, create a folder named:

```text
MAST
```

We will use this folder as the parent folder for our React Native projects.

Your folder structure should currently be:

```text
Documents
└── MAST
```

---

## Step 2: Open the Parent Folder in VS Code

1. Open **Visual Studio Code**.
2. If another project or folder is already open, select:

```text
File > Close Folder
```

3. Select:

```text
File > Open Folder
```

4. Navigate to your `MAST` folder.
5. Select **MAST** and click **Select Folder**.

At this point, VS Code is working with the folder that will contain our projects.

---

## Step 3: Open the Terminal

In Visual Studio Code, select:

```text
Terminal > New Terminal
```

The terminal should open inside the `MAST` folder.

You can check your current location by looking at the path shown in the terminal.

> **Important:** The location of the terminal matters. Commands such as `npx expo start` need to be executed from inside the React Native project folder.

---

# Step 4: Create the React Native Project

We can now create our project.

In the terminal, run:

```bash
npx create-expo-app -t expo-template-blank-typescript
```

This command uses Expo's project creation tool to create a new React Native project using the **blank TypeScript template**.

### What happens next?

You may be asked to confirm the installation of `create-expo-app`.

If prompted, type:

```text
y
```

and press **Enter**.

You will then be asked to provide a name for your application.

For this example, use:

```text
CounterApp
```

You will also be asked to select the Expo SDK version.

> **For this module, use SDK 54.** Our class environment and Expo Go setup are configured around this version. Do not select a different SDK unless instructed to do so.

Expo will then create a new project folder containing the files required for the application.

Your folder structure should now look similar to:

```text
Documents
└── MAST
    └── CounterApp
```

---

## Step 5: Open the Specific Project in VS Code

Once the project has been created, close the parent folder:

```text
File > Close Folder
```

Then select:

```text
File > Open Folder
```

and open:

```text
CounterApp
```

We do this because we want VS Code to work directly with the project we are currently developing.

Our structure is now:

```text
MAST
└── CounterApp
    ├── App.tsx
    ├── package.json
    └── ...
```

As the module progresses, additional files and folders will become important.

For now, we will keep our attention on the main application code.

---

# Step 6: Locate `App.tsx`

In the VS Code Explorer, locate:

```text
App.tsx
```

This is the main file we will initially use when learning the basic React Native concepts.

For now, `App.tsx` will be where we write and test much of our introductory application code.

As our applications become more complex, we will introduce additional files and folders.

> **Do not worry about understanding every file in the project yet.** We will introduce the purpose of the important files as they become relevant.

---

# Step 7: Start the Android Emulator

Our standard class environment uses **BlueStacks** as the Android device.

1. Launch **BlueStacks**.
2. Allow BlueStacks to boot completely.
3. Double-click the **Run for BlueStacks** batch file.

The batch file connects BlueStacks to the development environment using **ADB (Android Debug Bridge)**.

In our first class, we used:

```text
Android Studio > Device Manager
```

to check whether the Android device was visible.

If BlueStacks appears as an available device, the connection has been established.

---

# Step 8: Start the Expo Development Server

Return to your React Native project in VS Code.

Open a terminal inside the project folder.

You should be working inside:

```text
CounterApp
```

You can check the contents of the current folder by running:

```bash
ls
```

You should see files such as:

```text
App.tsx
package.json
```

Now start the Expo development server:

```bash
npx expo start
```

Expo will start the development server for your project.

You should now see the Expo development interface in the terminal.

---

# Step 9: Run the Application in BlueStacks

After the Expo development server has started, press:

```text
a
```

in the terminal.

The `a` command tells Expo that we want to open the project on an Android device.

BlueStacks should open **Expo Go**, and Expo Go should load the project.

### First-Time Use

If Expo Go asks you to update or install something, follow the instructions provided on screen.

Once the process has completed, your application should appear inside Expo Go.

---

# Troubleshooting: BlueStacks Is Not Recognised

There is currently a known issue in our class environment where BlueStacks may be connected through ADB but is not always recognised correctly when we attempt to run the Expo project.

If this happens, we can try the following:

1. Open:

```text
Android Studio > Device Manager
```

2. Remove the Android phone/emulator that is currently listed there.
3. Restart BlueStacks.
4. Run the **Run for BlueStacks** batch file again.
5. Check Android Studio's Device Manager.

The intention is that BlueStacks should now be the Android device recognised by the development environment.

Next, open BlueStacks and:

1. Sign in to your Google account.
2. Open Gmail if necessary.
3. Open the **Google Play Store**.
4. Update the **Expo Go** application.

Return to the VS Code terminal.

If the Expo development server is still running, stop it with:

```text
Ctrl + C
```

Then start it again:

```bash
npx expo start
```

Finally, press:

```text
a
```

to try running the application on BlueStacks again.

> This is a troubleshooting procedure for our class environment. If the standard process works correctly, there is no need to perform these additional steps.

---

# Alternative: Run the Application on a Physical Phone

BlueStacks is the standard Android environment we use during class.

However, you can also run your project on a physical Android or iPhone using the **Expo Go** application.

This can be useful because you can see and interact with your application on a real device.

## Prerequisite

Your phone should be able to connect to the development server running on your computer or VM.

When using a normal local network connection, the phone and development machine generally need to be on the same Wi-Fi network.

If the normal connection does not work, Expo provides a **tunnel** connection that can sometimes be used instead.

---

## Start Expo Using Tunnel Mode

From inside your project folder, run:

```bash
npx expo start --tunnel
```

Expo will start the development server and display a QR code.

---

## Using the QR Code

### Android

1. Install **Expo Go** from the Google Play Store.
2. Open Expo Go.
3. Select the option to scan a QR code.
4. Scan the QR code displayed by Expo.

### iPhone

1. Install **Expo Go** from the App Store.
2. Open the standard **Camera** application.
3. Point the camera at the QR code.
4. Tap the notification that appears.
5. The project will open in Expo Go.

---

# What Is Actually Running on the Phone?

At this stage, we are not installing our application as a finished native Android or iOS application.

Instead, **Expo Go provides a development environment that loads and runs our Expo project**.

This makes it possible to develop and test our application quickly without creating a complete standalone application each time we make a change.

This is particularly useful while learning.

---

# Fast Refresh

One of the useful features of the Expo development environment is **Fast Refresh**.

Fast Refresh allows changes to our source code to appear in the running application without restarting the entire development process.

For example:

```text
Edit App.tsx
      ↓
Save the file
      ↓
Fast Refresh
      ↓
Application updates
```

In VS Code, save the file using:

```text
Ctrl + S
```

You should then see the changes reflected in the running application.

---

# BlueStacks vs Physical Phone

| Feature           | BlueStacks             | Physical Phone      |
| ----------------- | ---------------------- | ------------------- |
| Device            | Virtual Android device | Real Android/iPhone |
| Expo Go           | Required               | Required            |
| Connection        | VM / ADB               | Network connection  |
| Touch interaction | Simulated              | Real                |
| Performance       | Depends on VM          | Usually faster      |
| Fast Refresh      | Yes                    | Yes                 |

For this module, **BlueStacks is our standard class environment**. The physical phone is an alternative for students who want to test their applications on a real device.

---

# Important Notes

### Always work inside the project folder

Commands such as:

```bash
npx expo start
```

should be run from inside the React Native project folder.

For our example:

```text
CounterApp
```

not from:

```text
MAST
```

The parent folder contains our projects, while the project folder contains the files and configuration required by Expo.

---

### Do not worry about every project file yet

When Expo creates a project, you will see several files and folders.

For now, we will mainly work with:

```text
App.tsx
```

As we introduce additional React Native concepts, we will learn what the other files and folders are used for.

---

# What Have We Just Done?

We have created our first React Native project and established the basic development workflow.

The important parts of our environment are:

* **VS Code** — where we write our code.
* **Terminal** — where we run commands.
* **Expo** — manages the React Native development process.
* **Expo Go** — provides the environment in which our project runs during development.
* **BlueStacks** — provides a virtual Android device for testing.
* **App.tsx** — the file we will initially use to write our application code.

The development cycle we will repeat throughout this module is:

```text
Write code
    ↓
Save code
    ↓
Expo / Fast Refresh
    ↓
Application updates
    ↓
Test the result
```

This is the basic workflow we will use as we begin building our React Native applications.

---

# You Are Ready to Build

You can now:

* Create a blank React Native project using Expo and TypeScript.
* Open and work with the project in VS Code.
* Start the Expo development server.
* Run the project using BlueStacks and Expo Go.
* Run the project on a physical phone using Expo Go.
* Make changes to `App.tsx`.
* Save your code and observe the changes using Fast Refresh.

From this point forward, our focus will move away from **setting up the environment** and towards **writing React Native code**.

#### References

[1]: https://docs.expo.dev/get-started/create-a-project/ "Create a project - Expo Documentation"
[2]: https://docs.expo.dev/get-started/start-developing/ "Start developing - Expo Documentation"
[3]: https://docs.expo.dev/get-started/set-up-your-environment/ "Set up your environment - Expo Documentation"
