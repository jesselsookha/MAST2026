# 03 Register User

### About this document

This document follows the incremental construction of a single React Native application built inside one file, `App.tsx`. The application is a simple user registration form: the user fills in fields, submits them, the data is validated, stored in an array, and displayed in a list. Over the course of eight stages (Stage 0 through Stage 7) the application grows from a static screen into a form with validation, a list, a custom card component, delete functionality, and finally edit-and-cancel functionality.

Each stage is presented in the same structure:

- **What this stage introduces** — a short framing of the new idea.
- **The code** — the complete `App.tsx` at that point in development.
- **Walkthrough** — a prose explanation of what the code does, in the order it executes.
- **Key concepts** — the React and React Native ideas worth naming.
- **Revision notes and gotchas** — common mistakes, and any corrections made to the original in-class code.

---

## Stage 0 — Static Scaffold

### What this stage introduces

Nothing dynamic yet. This stage establishes the smallest possible React Native screen: a single `View` containing a single `Text` element, centred on a white background. It confirms that the project compiles, that the development environment is working, and that a component can be rendered.

### The code

```tsx
import { StyleSheet, Text, View } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text>Registration</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 10,
  },
});
```

### Walkthrough

The file imports three things from `react-native`: `StyleSheet`, `Text`, and `View`. It then declares a single functional component, `App`, and exports it as the default export. React Native's entry point looks for this default export and renders it.

The component returns JSX describing a `View` (a container) which contains a `Text` element reading "Registration". Nothing in this component changes over time, so no state is involved.

Beneath the component, `StyleSheet.create` defines a single style object named `container`. That object is referenced from JSX via `styles.container`. The style centres its children horizontally and vertically, fills the available space (`flex: 1`), gives the screen a white background, and adds a small amount of padding.

### Revision notes and gotchas

- `StyleSheet.create` is not strictly required — a plain object would work — but it is idiomatic in React Native, and it lets the runtime validate style keys and optimise their use.
- `flex: 1` on the outermost container is what causes the view to occupy the whole screen. Without it, the container would collapse to the height of its content and the centring would have no visible effect.

---

## Stage 1 — User Input

### What this stage introduces

The first interactive elements. Five `TextInput` fields are added, one for each piece of registration data, along with a `TouchableHighlight` acting as a submit button. Each input is wired to its own piece of React state via `useState`, so that what the user types is reflected in the component's memory.

### The code

```tsx
import { useState } from 'react'; 
import { 
  StyleSheet, 
  Text, 
  TextInput, 
  TouchableHighlight, 
  View 
} from 'react-native';

export default function App() {
  const [firstName, setFirstName] = useState<string>(''); 
  const [surname, setSurname] = useState<string>(''); 
  const [email, setEmail] = useState<string>(''); 
  const [phoneNumber, setPhoneNumber] = useState<string>(''); 
  const [age, setAge] = useState<string>('');

  const handleSave = () => {

  };

  return (
    <View style={styles.container}>
      <Text>Registration</Text>
      <TextInput 
        value={firstName}
        onChangeText={setFirstName}
        style={styles.input}
        placeholder='First name'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={surname}
        onChangeText={setSurname}        
        style={styles.input}
        placeholder='Surname'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={email}
        onChangeText={setEmail}
        style={styles.input}
        placeholder='Email'
        maxLength={60}
        inputMode='email'
      />

      <TextInput 
        value={phoneNumber}
        onChangeText={setPhoneNumber}
        style={styles.input}
        placeholder='Phone Number'
        maxLength={15}
        inputMode='tel'
      />

      <TextInput 
        value={age}
        onChangeText={setAge}
        style={styles.input}
        placeholder='Age'
        maxLength={3}
        inputMode='numeric'
      />

      <TouchableHighlight onPress={handleSave}>
        <Text>Submit</Text>
      </TouchableHighlight>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 10,
  },
  input: {
    height: 40,
    margin: 12,
    borderWidth: 1,
    padding: 10,
  },
});
```

### Walkthrough

The import line now includes `useState` from `react` (not from `react-native` — this distinction matters and is a frequent source of errors). The `react-native` import grows to include `TextInput` and `TouchableHighlight`.

Inside the component, five `useState` calls are declared, one per field. Each call returns a pair: the current value and a setter. The generic type `<string>` tells TypeScript that each piece of state is a string, and the initial value is the empty string `''`.

A `handleSave` function is declared but does nothing yet. It is attached to the `TouchableHighlight` via the `onPress` prop, so tapping the button currently produces no visible result. This is intentional at this stage — the function is a placeholder for Stage 2.

Each `TextInput` follows the same pattern:

- `value` binds the input to a piece of state. This makes the input **controlled**.
- `onChangeText` is given the setter directly. React Native calls this with the new text whenever the user types, so the state updates on every keystroke.
- `style` applies the shared `input` style.
- `placeholder` provides the greyed-out hint text.
- `maxLength` caps how much the user can type.
- `inputMode` hints at the appropriate on-screen keyboard. `'text'` for names, `'email'` for the email keyboard (which includes `@` and `.`), `'tel'` for the phone keypad, `'numeric'` for the number pad on age.

The `styles.input` object gives each field a fixed height, a border, an outer margin, and internal padding so the text does not touch the border.

### Revision notes and gotchas

- A `TextInput` is *controlled* when its `value` prop is tied to state and its changes are routed back through `onChangeText`. If you set `value` but forget `onChangeText`, the field will appear frozen — the user types but nothing appears, because the displayed value is always the state value, which never changes.
- `useState` must be imported from `'react'`, not from `'react-native'`. This is one of the most common early mistakes.
- `inputMode` is the modern prop for keyboard hints. On older React Native versions you may see `keyboardType` used instead. Both work; `inputMode` is the current recommendation and aligns with the web platform.
- The button at this stage has no styling, so it renders as plain text. That is fine for now; visual polish arrives later.
- `handleSave` is defined but empty. An empty function is valid JavaScript and simply does nothing when called.

---

## Stage 2 — Validation and the Registration Object

### What this stage introduces

This is the first stage where the form actually *does* something. Two ideas arrive together:

1. A **type** describing the shape of a registration record.
2. An **array of registrations** held in state, into which each valid submission is placed.

The stage also introduces **validation**: before anything is stored, the fields are checked to ensure none are empty. If any are, the function exits early and a message is logged.

### The code

```tsx
import { useState } from 'react'; 
import { 
  StyleSheet, 
  Text, 
  TextInput, 
  TouchableHighlight, 
  View 
} from 'react-native';

// structure for the user object 
type Registration = {
  firstName: string;
  surname: string;
  email: string;
  phoneNumber: string;
  age: string;
};

export default function App() {
  const [firstName, setFirstName] = useState<string>(''); 
  const [surname, setSurname] = useState<string>(''); 
  const [email, setEmail] = useState<string>(''); 
  const [phoneNumber, setPhoneNumber] = useState<string>(''); 
  const [age, setAge] = useState<string>('');

  // array of objects ("registered users") 
  const [registrations, setRegistrations] = useState<Registration[]>([]);

  const handleSave = () => {
    // Initial Validation
    if (
      firstName.trim() === '' ||
      surname.trim() === '' ||
      email.trim() === '' ||
      phoneNumber.trim() === '' ||
      age.trim() === ''
    ) {
      console.log('Please complete all fields');
      return;
    }

    // More validation of data needs to be performed

    // Create new registration object
    const newRegistration: Registration = {
      firstName: firstName.trim(),
      surname: surname.trim(),
      email: email.trim(),
      phoneNumber: phoneNumber.trim(),
      age: age.trim(),
    };

    // Add object to array
    setRegistrations([...registrations, newRegistration]);

    console.log('Registration saved:', newRegistration);

  };

  return (
    <View style={styles.container}>
      <Text>Registration</Text>
      <TextInput 
        value={firstName}
        onChangeText={setFirstName}
        style={styles.input}
        placeholder='First name'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={surname}
        onChangeText={setSurname}        
        style={styles.input}
        placeholder='Surname'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={email}
        onChangeText={setEmail}
        style={styles.input}
        placeholder='Email'
        maxLength={60}
        inputMode='email'
      />

      <TextInput 
        value={phoneNumber}
        onChangeText={setPhoneNumber}
        style={styles.input}
        placeholder='Phone Number'
        maxLength={15}
        inputMode='tel'
      />

      <TextInput 
        value={age}
        onChangeText={setAge}
        style={styles.input}
        placeholder='Age'
        maxLength={3}
        inputMode='numeric'
      />

      <TouchableHighlight onPress={handleSave}>
        <Text>Submit</Text>
      </TouchableHighlight>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 10,
  },
  input: {
    height: 40,
    margin: 12,
    borderWidth: 1,
    padding: 10,
  },
});
```

### Walkthrough

Above the component, a TypeScript `type` alias called `Registration` is declared. It describes an object with five string properties: `firstName`, `surname`, `email`, `phoneNumber`, and `age`. This type is not yet used for anything at runtime — it exists purely to describe, to the compiler and to the reader, what a "registration" looks like. It will be used as the element type of the array below.

Inside the component, a sixth `useState` call is added:

```tsx
const [registrations, setRegistrations] = useState<Registration[]>([]);
```

The type parameter `Registration[]` means "an array of Registration objects", and the initial value `[]` is an empty array. So the component starts with no registered users and grows the array as submissions arrive.

The `handleSave` function is now implemented. Its logic runs in three steps.

**Step 1 — Validation.** The `if` statement checks each field in turn. The expression inside uses `||` (logical OR): if *any* one of the conditions is true, the whole expression is true and the block runs. Each condition is of the form `field.trim() === ''`.

Why `.trim()`? Because a field containing only spaces — `"   "` — would pass a naive `=== ''` check. Calling `.trim()` removes leading and trailing whitespace, so `"   "` becomes `""`, which then equals `''`. This is a small but meaningful defence against accidental empty submissions.

When the validation fails, a message is logged and the `return` statement exits the function immediately. Nothing below it runs. This pattern — validate, bail out early, otherwise continue — is often called a *guard clause*.

**Step 2 — Build the record.** A new constant `newRegistration` is created, typed as `Registration`, with each property set to the trimmed value of the corresponding state variable. Trimming again here means that even if the user typed `"  Ada  "`, what gets stored is `"Ada"`. The data in the array is therefore clean, regardless of how the user typed it.

**Step 3 — Store the record.** The setter is called with a new array:

```tsx
setRegistrations([...registrations, newRegistration]);
```

The `...registrations` is the spread operator. It expands the existing array into individual elements, and then `newRegistration` is appended after them. The result is a brand new array containing everything the old array had, plus the new record. This new array is handed to `setRegistrations`.

Finally, a `console.log` confirms the save. At this point the application stores data correctly, but nothing is displayed on screen yet — you would need to open the debug console to see the log messages. Displaying the array is the task of a later stage.

### Key concepts

**State as the single source of truth.** The five input fields are driven by their state variables, and the array of registrations is itself state. At no point does the component read a value directly from a `TextInput` DOM node or from a variable outside React. Everything flows through state.

**Immutability.** The array is never modified in place. There is no `registrations.push(...)`. Instead a new array is created and assigned. This is a core React principle: state is replaced, not mutated. React compares the old and new references to decide whether to re-render, so creating a new array is what signals "something changed".

**Type aliases as documentation.** The `Registration` type carries no runtime weight, but it makes the intent of the code explicit and lets TypeScript catch mistakes such as misspelling `phoneNumber` or assigning a number where a string is expected.

**Guard clauses.** Handling the invalid case first, and exiting early, keeps the "happy path" of the function un-indented and readable. The alternative — nesting the entire save logic inside an `else` — is harder to read as the function grows.

### Revision notes and gotchas

- The comment `// More validation of data needs to be performed` is a deliberate marker. At this stage, validation only checks that fields are not empty. It does not check that the email contains an `@`, that the phone number consists of digits, that the age is a sensible number, or that the age is within a reasonable range. Adding those checks is a natural extension exercise and does not change the structure of the function.
- This stage uses the **stale-closure form** of the setter:

  ```tsx
  setRegistrations([...registrations, newRegistration]);
  ```

  This reads the value of `registrations` *as it was when the function was created*. In a single-threaded, one-submission-at-a-time scenario it behaves correctly, which is why it was taught here. However, it is subtly fragile: if two updates were scheduled before a re-render, the second could overwrite the first because both would have captured the same starting array. Stage 3 introduces the safer alternative, the functional updater, and explains the difference. For now, the behaviour is correct, and the concept of spreading into a new array is the important takeaway.

- The `console.log('Registration saved:', newRegistration)` logs the *newly created* object, not the array. That is accurate and useful. In Stage 3 the log statement is changed to log the array instead — and this exposes a subtlety worth understanding, which is discussed there.

---

## Stage 3 — Functional State Updates

### What this stage introduces

Two refinements, neither of which changes what the user sees, but both of which matter for correctness and for naming clarity.

1. The array of registrations is **renamed** from `registrations` to `registeredUsers`. The new name better describes what the array holds — a list of users who have registered, rather than a list of registration events.
2. The setter call switches from the stale-closure form to the **functional updater** form. Instead of passing a new array computed from the current state variable, a function is passed that receives the previous state and returns the next state.

The new user object is also renamed from `newRegistration` to `newUser`, consistent with the array rename.

### The code

```tsx
import { useState } from 'react'; 
import { 
  StyleSheet, 
  Text, 
  TextInput, 
  TouchableHighlight, 
  View 
} from 'react-native';

// structure for the user object 
type Registration = {
  firstName: string;
  surname: string;
  email: string;
  phoneNumber: string;
  age: string;
};

export default function App() {
  const [firstName, setFirstName] = useState<string>(''); 
  const [surname, setSurname] = useState<string>(''); 
  const [email, setEmail] = useState<string>(''); 
  const [phoneNumber, setPhoneNumber] = useState<string>(''); 
  const [age, setAge] = useState<string>('');

  // array of objects ("registered users") 
  const [registeredUsers, setRegisteredUsers] = useState<Registration[]>([]);

  const handleSave = () => {
    // Initial Validation
    if (
      firstName.trim() === '' ||
      surname.trim() === '' ||
      email.trim() === '' ||
      phoneNumber.trim() === '' ||
      age.trim() === ''
    ) {
      console.log('Please complete all fields');
      return;
    }

    // More validation of data needs to be performed

    // Create new registration object
    const newUser: Registration = {
      firstName: firstName.trim(),
      surname: surname.trim(),
      email: email.trim(),
      phoneNumber: phoneNumber.trim(),
      age: age.trim(),
    };

    // Add object to array
    setRegisteredUsers(prevUsers => [...prevUsers, newUser]);

    console.log('Registration saved:', registeredUsers);

  };

  return (
    <View style={styles.container}>
      <Text>Registration</Text>
      <TextInput 
        value={firstName}
        onChangeText={setFirstName}
        style={styles.input}
        placeholder='First name'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={surname}
        onChangeText={setSurname}        
        style={styles.input}
        placeholder='Surname'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={email}
        onChangeText={setEmail}
        style={styles.input}
        placeholder='Email'
        maxLength={60}
        inputMode='email'
      />

      <TextInput 
        value={phoneNumber}
        onChangeText={setPhoneNumber}
        style={styles.input}
        placeholder='Phone Number'
        maxLength={15}
        inputMode='tel'
      />

      <TextInput 
        value={age}
        onChangeText={setAge}
        style={styles.input}
        placeholder='Age'
        maxLength={3}
        inputMode='numeric'
      />

      <TouchableHighlight onPress={handleSave}>
        <Text>Submit</Text>
      </TouchableHighlight>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 10,
  },
  input: {
    height: 40,
    margin: 12,
    borderWidth: 1,
    padding: 10,
  },
});
```

### Walkthrough

Most of the file is unchanged. The differences are confined to three places.

**The state declaration.** The array's name is now `registeredUsers`, and its setter is `setRegisteredUsers`:

```tsx
const [registeredUsers, setRegisteredUsers] = useState<Registration[]>([]);
```

The type parameter remains `Registration[]`, and the initial value remains `[]`. This is a rename only.

**The new user object.** The constant that holds the freshly built record is renamed from `newRegistration` to `newUser`. Its shape is unchanged. The rename aligns the vocabulary: the array holds users, so the object added to it is a user.

**The setter call.** This is the substantive change:

```tsx
setRegisteredUsers(prevUsers => [...prevUsers, newUser]);
```

Compare this with Stage 2's version:

```tsx
setRegistrations([...registrations, newRegistration]);
```

In Stage 2, the array passed to the setter was computed *before* the setter was called, using whatever value `registrations` happened to hold at that moment. In Stage 3, a *function* is passed instead. React will call that function itself, at the moment it applies the update, and will hand it the most recent value of the state. The parameter `prevUsers` receives that value. The function returns a new array — the previous users plus the new user.

The result is the same in a single submission. The difference appears when several updates are queued before React has had a chance to re-render. In that case, the stale-closure form can overwrite earlier updates because each closure captured the same starting array. The functional form cannot, because each queued function receives the result of the previous one.

**The console.log.** The log statement now reads:

```tsx
console.log('Registration saved:', registeredUsers);
```

This is a *teaching moment*, and it is deliberately left in. Because React schedules state updates and re-renders asynchronously, the variable `registeredUsers` inside `handleSave` still refers to the array as it was *before* the update. The log will therefore print the array without the new user in it. This is not a bug in the application logic — the array state is updating correctly — it is simply a consequence of how React batches updates. The new user appears in the array on the next render.

In Stage 4 the log statement is commented out, because displaying the array on screen replaces the need for console inspection.

### Key concepts

**Functional updater.** Passing `prev => next` to a state setter, rather than `next` directly, is the recommended form whenever the next state depends on the previous state. It removes an entire class of bugs around batching and stale closures.

**Asynchronous state updates.** `setState` does not change the variable it was called for. It schedules a change. The variable you read immediately after the call is the old value. This is why the log statement shows the old array.

**Naming as a design activity.** The rename from `registrations` to `registeredUsers` is small, but it clarifies what the array contains. In a teaching context it also gives a natural opportunity to discuss why names matter: `registrations` suggests the *event* of registering, while `registeredUsers` names the *collection of people*.

### Revision notes and gotchas

- The type alias is still called `Registration`, even though the variable is now `registeredUsers`. This is intentional: the type describes the *shape* of one record. Renaming the type is possible but would ripple through the file for no real gain. It is worth explaining to students that a type name and a variable name do not need to match, though consistency helps.
- The stale-closure versus functional-updater distinction is easily lost in a small classroom example, because both versions work for a single click. The way to make the difference concrete is to imagine two `setRegisteredUsers` calls scheduled in the same event handler — for example, if you later added a feature to register two users at once. The Stage 2 form would lose one; the Stage 3 form would keep both.
- The `console.log` of `registeredUsers` is a good teaching device but can confuse students into thinking the code is broken. The explanation is: the log shows the state as it was at the start of this call, and the state is updated afterwards. A way to *see* the new value is to log inside a `useEffect` (hook) that watches `registeredUsers`, but that is a topic for later.
- The array passed to `setRegisteredUsers` is a new array, not the old one mutated. This is the same immutability principle as Stage 2; only the mechanism for computing the new array has changed.

---

## Stage 4 — FlatList Display

### What this stage introduces

The application now displays its data. A `FlatList` is added beneath the form, bound to the `registeredUsers` array. Each item in the array is rendered as a single `Text` element showing the user's first name.

This is intentionally minimal. Stage 4 is about establishing the connection between state and a scrolling list. The visual design of the list items is the work of Stage 5.

Two structural additions accompany the FlatList:

1. A `keyExtractor` is provided, so that React can identify each row reliably.
2. The `console.log` in `handleSave` is commented out, since the on-screen list now serves the purpose the log served.

### The code

```tsx
import { useState } from 'react'; 
import { 
  FlatList,
  StyleSheet, 
  Text, 
  TextInput, 
  TouchableHighlight, 
  View 
} from 'react-native';

// structure for the user object 
type Registration = {
  firstName: string;
  surname: string;
  email: string;
  phoneNumber: string;
  age: string;
};

export default function App() {
  const [firstName, setFirstName] = useState<string>(''); 
  const [surname, setSurname] = useState<string>(''); 
  const [email, setEmail] = useState<string>(''); 
  const [phoneNumber, setPhoneNumber] = useState<string>(''); 
  const [age, setAge] = useState<string>('');

  // array of objects ("registered users") 
  const [registeredUsers, setRegisteredUsers] = useState<Registration[]>([]);

  const handleSave = () => {
    // Initial Validation
    if (
      firstName.trim() === '' ||
      surname.trim() === '' ||
      email.trim() === '' ||
      phoneNumber.trim() === '' ||
      age.trim() === ''
    ) {
      console.log('Please complete all fields');
      return;
    }

    // More validation of data needs to be performed

    // Create new registration object
    const newUser: Registration = {
      firstName: firstName.trim(),
      surname: surname.trim(),
      email: email.trim(),
      phoneNumber: phoneNumber.trim(),
      age: age.trim(),
    };

    // Add object to array
    setRegisteredUsers(prevUsers => [...prevUsers, newUser]);

    //console.log('Registration saved:', registeredUsers);

  };

  return (
    <View style={styles.container}>
      <Text>Registration</Text>
      <TextInput 
        value={firstName}
        onChangeText={setFirstName}
        style={styles.input}
        placeholder='First name'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={surname}
        onChangeText={setSurname}        
        style={styles.input}
        placeholder='Surname'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={email}
        onChangeText={setEmail}
        style={styles.input}
        placeholder='Email'
        maxLength={60}
        inputMode='email'
      />

      <TextInput 
        value={phoneNumber}
        onChangeText={setPhoneNumber}
        style={styles.input}
        placeholder='Phone Number'
        maxLength={15}
        inputMode='tel'
      />

      <TextInput 
        value={age}
        onChangeText={setAge}
        style={styles.input}
        placeholder='Age'
        maxLength={3}
        inputMode='numeric'
      />

      <TouchableHighlight onPress={handleSave}>
        <Text>Submit</Text>
      </TouchableHighlight>

      <FlatList 
        data={registeredUsers}
        renderItem={({item}) => <Text>{item.firstName}</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 10,
  },
  input: {
    height: 40,
    margin: 12,
    borderWidth: 1,
    padding: 10,
  },
});
```

### Walkthrough

`FlatList` is added to the import list from `react-native`. It is the standard component for rendering a scrollable collection of items, and is preferred over `ScrollView` combined with `.map()` because it virtualises the list — it only renders the rows currently visible on screen, which matters when the list is long.

The FlatList is placed inside the container, after the Submit button. It is given two props:

```tsx
<FlatList 
  data={registeredUsers}
  renderItem={({item}) => <Text>{item.firstName}</Text>}
/>
```

`data` is the array to display. It is bound to the `registeredUsers` state, so whenever a new user is registered, the array changes, React re-renders, and the FlatList shows one more row.

`renderItem` is a function that FlatList calls once per item. It receives an object with several properties, of which the most important is `item` — the actual element from the array. The destructuring `({item})` pulls that property out directly. The function returns the JSX for one row. Here, that JSX is a `Text` element containing the user's first name.

When the list is empty — as it is on first render — FlatList renders nothing. No error occurs.

### Key concepts

**FlatList versus ScrollView with map.** A `ScrollView` renders all its children at once, regardless of how many there are. A `FlatList` renders only what fits on screen, plus a small buffer. For a handful of users either would work, but FlatList is the correct habit to form.

**`renderItem` as a render prop.** `renderItem` is a function passed as a prop, which FlatList calls internally. This pattern — passing a function that returns JSX — is used throughout React Native (for example, in `ListHeaderComponent`, `ListEmptyComponent`, and `ListFooterComponent`).

**`keyExtractor`.** React needs a stable, unique identifier for each row so that, when the array changes, it can work out which rows are new, which have moved, and which are unchanged. By default, FlatList looks for an `id` or `key` property on each item, or falls back to the array index. The index is a poor identifier because it changes when items are removed or reordered. The correct approach is to give each item a stable `id` and provide a `keyExtractor` that returns it.

### Revision notes and gotchas

**The missing `keyExtractor` was added in this document's version.** The in-class code omitted it, which in a small list often produces no visible problem but generates a warning in the console and can lead to incorrect rendering once items are deleted or reordered — as they are in Stage 6.

The record type does not yet have an `id` field. To support a stable `keyExtractor`, an `id` is added to the `Registration` type and to `newUser`, and the `keyExtractor` is supplied to the FlatList. The changes, relative to the original Stage 4 code, are:

```tsx
type Registration = {
  id: string;
  firstName: string;
  surname: string;
  email: string;
  phoneNumber: string;
  age: string;
};
```

```tsx
const newUser: Registration = {
  id: Date.now().toString(),
  firstName: firstName.trim(),
  surname: surname.trim(),
  email: email.trim(),
  phoneNumber: phoneNumber.trim(),
  age: age.trim(),
};
```

```tsx
<FlatList 
  data={registeredUsers}
  keyExtractor={(item) => item.id}
  renderItem={({item}) => <Text>{item.firstName}</Text>}
/>
```

The `id` is generated with `Date.now().toString()`, which returns the current time in milliseconds as a string. This is not a rigorous unique-id scheme — two users added within the same millisecond would collide — but for a classroom application it is sufficient, and it introduces the idea that records need identities. A production application would use a library such as `uuid` or a server-assigned id.

The full corrected code for Stage 4 is:

```tsx
import { useState } from 'react'; 
import { 
  FlatList,
  StyleSheet, 
  Text, 
  TextInput, 
  TouchableHighlight, 
  View 
} from 'react-native';

// structure for the user object 
type Registration = {
  id: string;
  firstName: string;
  surname: string;
  email: string;
  phoneNumber: string;
  age: string;
};

export default function App() {
  const [firstName, setFirstName] = useState<string>(''); 
  const [surname, setSurname] = useState<string>(''); 
  const [email, setEmail] = useState<string>(''); 
  const [phoneNumber, setPhoneNumber] = useState<string>(''); 
  const [age, setAge] = useState<string>('');

  // array of objects ("registered users") 
  const [registeredUsers, setRegisteredUsers] = useState<Registration[]>([]);

  const handleSave = () => {
    // Initial Validation
    if (
      firstName.trim() === '' ||
      surname.trim() === '' ||
      email.trim() === '' ||
      phoneNumber.trim() === '' ||
      age.trim() === ''
    ) {
      console.log('Please complete all fields');
      return;
    }

    // More validation of data needs to be performed

    // Create new registration object
    const newUser: Registration = {
      id: Date.now().toString(),
      firstName: firstName.trim(),
      surname: surname.trim(),
      email: email.trim(),
      phoneNumber: phoneNumber.trim(),
      age: age.trim(),
    };

    // Add object to array
    setRegisteredUsers(prevUsers => [...prevUsers, newUser]);

    //console.log('Registration saved:', registeredUsers);

  };

  return (
    <View style={styles.container}>
      <Text>Registration</Text>
      <TextInput 
        value={firstName}
        onChangeText={setFirstName}
        style={styles.input}
        placeholder='First name'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={surname}
        onChangeText={setSurname}        
        style={styles.input}
        placeholder='Surname'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={email}
        onChangeText={setEmail}
        style={styles.input}
        placeholder='Email'
        maxLength={60}
        inputMode='email'
      />

      <TextInput 
        value={phoneNumber}
        onChangeText={setPhoneNumber}
        style={styles.input}
        placeholder='Phone Number'
        maxLength={15}
        inputMode='tel'
      />

      <TextInput 
        value={age}
        onChangeText={setAge}
        style={styles.input}
        placeholder='Age'
        maxLength={3}
        inputMode='numeric'
      />

      <TouchableHighlight onPress={handleSave}>
        <Text>Submit</Text>
      </TouchableHighlight>

      <FlatList 
        data={registeredUsers}
        keyExtractor={(item) => item.id}
        renderItem={({item}) => <Text>{item.firstName}</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 10,
  },
  input: {
    height: 40,
    margin: 12,
    borderWidth: 1,
    padding: 10,
  },
});
```

A further point worth noting: the `console.log('Please complete all fields')` in the validation branch is still present. In Stage 6 this is replaced by an on-screen error message, so that the user — not just the developer — is informed.

---

## Stage 5 — A Custom UserCard Component

### What this stage introduces

The list rows are no longer bare first names. A new user-defined component, `UserCard`, is introduced. It takes the fields of a registration as props and renders a structured card: a header row containing the user's full name and age badge, and a details section containing the email and phone number, each with a small label.

Introducing `UserCard` teaches three ideas at once: how to define a component in the same file, how to type its props, and how to pass data to it from a parent.

A large number of style rules are also added, giving the card a distinctive visual design — a light blue panel with a coloured left border, a bold name, and a rounded age pill.

### The code

```tsx
import { useState } from 'react'; 
import { 
  FlatList,
  StyleSheet, 
  Text, 
  TextInput, 
  TouchableHighlight, 
  View 
} from 'react-native';

// ---------------------------------------
// creating a user-defined component

type UserProps = { 
  fn: string; // abbrerviated to help differentiate variable names
  sn: string; 
  em: string; 
  pn: string; 
  ag: string;
};

function UserCard({fn, sn, em, pn, ag} : UserProps) {
  return(
    <View style={styles.card}>
      <View style={styles.cardHeader}>
        <Text style={styles.cardName}>{fn} {sn}</Text>
        <Text style={styles.cardAge}>{ag}</Text>
      </View>
      <View style={styles.cardDetails}>
        <Text style={styles.cardLabel}>Email</Text>
        <Text style={styles.cardText}>{em}</Text>
        <Text style={styles.cardLabel}>Phone</Text>
        <Text style={styles.cardText}>{pn}</Text>
      </View>
    </View>
  );
}
// ---------------------------------------

// structure for the user object 
type Registration = {
  id: string;
  firstName: string;
  surname: string;
  email: string;
  phoneNumber: string;
  age: string;
};

export default function App() {
  const [firstName, setFirstName] = useState<string>(''); 
  const [surname, setSurname] = useState<string>(''); 
  const [email, setEmail] = useState<string>(''); 
  const [phoneNumber, setPhoneNumber] = useState<string>(''); 
  const [age, setAge] = useState<string>('');

  // array of objects ("registered users") 
  const [registeredUsers, setRegisteredUsers] = useState<Registration[]>([]);

  const handleSave = () => {
    // Initial Validation
    if (
      firstName.trim() === '' ||
      surname.trim() === '' ||
      email.trim() === '' ||
      phoneNumber.trim() === '' ||
      age.trim() === ''
    ) {
      console.log('Please complete all fields');
      return;
    }

    // More validation of data needs to be performed

    // Create new registration object
    const newUser: Registration = {
      id: Date.now().toString(),
      firstName: firstName.trim(),
      surname: surname.trim(),
      email: email.trim(),
      phoneNumber: phoneNumber.trim(),
      age: age.trim(),
    };

    // Add object to array
    setRegisteredUsers(prevUsers => [...prevUsers, newUser]);

    //console.log('Registration saved:', registeredUsers);

  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Registration</Text>
      <TextInput 
        value={firstName}
        onChangeText={setFirstName}
        style={styles.input}
        placeholder='First name'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={surname}
        onChangeText={setSurname}        
        style={styles.input}
        placeholder='Surname'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={email}
        onChangeText={setEmail}
        style={styles.input}
        placeholder='Email'
        maxLength={60}
        inputMode='email'
      />

      <TextInput 
        value={phoneNumber}
        onChangeText={setPhoneNumber}
        style={styles.input}
        placeholder='Phone Number'
        maxLength={15}
        inputMode='tel'
      />

      <TextInput 
        value={age}
        onChangeText={setAge}
        style={styles.input}
        placeholder='Age'
        maxLength={3}
        inputMode='numeric'
      />

      <TouchableHighlight onPress={handleSave}>
        <Text>Submit</Text>
      </TouchableHighlight>

      <FlatList 
        data={registeredUsers}
        keyExtractor={(item) => item.id}
        renderItem={({item} : {item : Registration}) => 
          <UserCard
            fn={item.firstName}
            sn={item.surname}
            em={item.email}
            pn={item.phoneNumber}
            ag={item.age}
          />
        }
      />
    </View>
  );
}

const styles = StyleSheet.create({ 
  container: { 
    flex: 1, 
    justifyContent: 'center', 
    backgroundColor: '#fff', 
    padding: 10, 
    alignItems: 'center', 
  }, 
  errMessage: { 
    fontSize: 12, 
    color: '#ff0000', 
    fontWeight: 'bold', 
    height: 20, 
  }, 
  title: { 
    fontSize: 24, 
    fontWeight: 'bold', 
  }, 
  input: { 
    height: 35, 
    margin: 12, 
    borderWidth: 1, 
    padding: 10, 
  }, 
  button: { 
    height: 40, 
    margin: 10, 
    padding: 10, 
    justifyContent: 'center', 
    alignItems: 'center', 
    borderRadius: 5, 
    backgroundColor: '#333', 
  }, 
  buttonText: { 
    fontSize: 18, 
    color: '#fefefe', 
  }, 
  card: { 
    width: '95%', 
    backgroundColor: '#EAF4FF', 
    borderRadius: 8, 
    marginVertical: 5, 
    padding: 10, 
    borderLeftWidth: 5, 
    borderLeftColor: '#1976D2', 
  },
  cardHeader: { 
    flexDirection: 'row', 
    justifyContent: 'space-between', 
    alignItems: 'center', 
    marginBottom: 6, 
  },
  cardName: { 
    fontSize: 16, 
    fontWeight: 'bold', 
    color: '#1A1A1A', 
    flex: 1, 
  },
  cardAge: { 
    fontSize: 13, 
    fontWeight: 'bold', 
    color: '#1976D2', 
    backgroundColor: '#D6EBFF', 
    paddingHorizontal: 8, 
    paddingVertical: 3, 
    borderRadius: 10, 
  },
  cardDetails: { 
    borderTopWidth: 1, 
    borderTopColor: '#C9DFF5', 
    paddingTop: 6, 
  },
  cardLabel: { 
    fontSize: 10, 
    fontWeight: 'bold', 
    color: '#1976D2', 
    marginTop: 2, 
  },
  cardText: { 
    fontSize: 13, 
    color: '#333', 
    marginBottom: 3, 
  },
});
```

### Walkthrough

Above the `App` component, a new block of code begins. First, a type alias `UserProps` is declared, describing the props that `UserCard` expects. Five string props are listed: `fn`, `sn`, `em`, `pn`, and `ag`. These abbreviations are deliberate — they are shorter than `firstName`, `surname`, `email`, `phoneNumber`, and `age`, and using them makes the destructuring line short and the component's internals readable without long variable names repeated many times. The comment in the original code explains this choice.

Next, the `UserCard` function is declared. It destructures the five props directly in its parameter list:

```tsx
function UserCard({fn, sn, em, pn, ag} : UserProps) {
```

The colon and `UserProps` after the destructuring pattern is the TypeScript type annotation for the whole props object. This is a common shape for a typed functional component: destructure the props, then annotate the result as the props type.

The component returns JSX structured in three parts.

- The outermost `View` uses `styles.card`. That style gives the card its width, background colour, rounded corners, vertical margin, padding, and — importantly — a thick coloured left border. The left border is what gives each card its visual anchor.
- The first inner `View` uses `styles.cardHeader`. It is a horizontal row (`flexDirection: 'row'`) with its children pushed to opposite ends (`justifyContent: 'space-between'`). It contains the full name and the age. The name uses `styles.cardName` and is given `flex: 1` so that it takes the available horizontal space and pushes the age to the far right. The age uses `styles.cardAge`, which gives it a pale blue background and rounded corners — a pill shape.
- The second inner `View` uses `styles.cardDetails`. It has a thin top border, separating the details from the header. Inside are two labelled pairs: the label "Email" in a small blue uppercase-style font, followed by the email text; then the label "Phone" followed by the phone number.

Back inside `App`, the FlatList's `renderItem` has changed. Instead of returning a `Text` element, it now returns a `UserCard`:

```tsx
renderItem={({item} : {item : Registration}) => 
  <UserCard
    fn={item.firstName}
    sn={item.surname}
    em={item.email}
    pn={item.phoneNumber}
    ag={item.age}
  />
}
```

The type annotation `({item} : {item : Registration})` tells TypeScript that the argument to this function is an object whose `item` property is a `Registration`. This matters because, without it, TypeScript would not know the shape of `item` and would complain when you access `item.firstName`. Earlier stages got away without the annotation because the destructured `item` was only used inside a template expression where type checking was less strict; as soon as `item` is passed as a prop to a typed component, the annotation becomes necessary.

The style sheet has grown substantially. Most of the new entries — `title`, `button`, `buttonText`, and the `card*` family — are used to give the screen a more deliberate look. The `errMessage` style is also declared here, ready for use in Stage 6, though nothing currently references it.

### Key concepts

**User-defined components.** A component is a function that returns JSX and whose name begins with a capital letter. `UserCard` is defined in the same file as `App`, which is fine for a small application; in a larger one it would live in its own file and be imported.

**Props.** Props are the inputs to a component. They are read-only. A component cannot modify its own props; if it needs to change based on something, that change flows through state in a parent, and the parent passes the new value down.

**Props type annotation.** Declaring a `UserProps` type and annotating the component with it lets TypeScript check that callers supply exactly the right props. If you tried to render `<UserCard fn="Ada" />` without the other four, TypeScript would flag it.

**Separation of concerns.** Before this stage, the rendering logic for a list row lived inside `renderItem` in `App`. Now it lives in `UserCard`. `App` is responsible for the form, the state, and the validation; `UserCard` is responsible for presenting a single user. Keeping these separate makes each piece easier to reason about and to change independently.

**Abbreviated prop names.** The names `fn`, `sn`, `em`, `pn`, and `ag` are not ideal for a large codebase, where clarity matters more than brevity. They are used here to keep the destructuring line and the JSX uncluttered in a teaching context, and the comment in the code records this intention. A real project would likely use the full names, or spread the entire registration object as a single `user` prop.

### Revision notes and gotchas

- The in-class version of `UserCard` referenced `styles.card`, `styles.cardHeader`, and so on, and those styles are present in the style sheet — but only from Stage 5 onward. In earlier stages the card styles are absent, and in this document they have been introduced here, in the stage where the component that uses them is introduced. This is the correct point of introduction.
- The `errMessage` style is declared in this stage but not used until Stage 6. Declaring a style slightly before its use is common and harmless, but it is worth noting so that students do not expect an on-screen error message yet.
- The `renderItem` type annotation is more verbose than in earlier stages. An alternative, and a slightly cleaner one, is to annotate the FlatList itself with a generic type parameter:

  ```tsx
  <FlatList<Registration>
    data={registeredUsers}
    keyExtractor={(item) => item.id}
    renderItem={({item}) => (
      <UserCard ... />
    )}
  />
  ```

  With the generic `FlatList<Registration>`, TypeScript infers the type of `item` inside `renderItem` and the explicit annotation becomes unnecessary. This is a useful pattern to mention, but the explicit annotation matches what was taught and is kept here.

- `UserCard` receives five separate props rather than one `user` object. Both are valid. A single prop — `<UserCard user={item} />` — would reduce the boilerplate at the call site, at the cost of coupling `UserCard` more tightly to the `Registration` shape. The five-prop form makes the component's inputs explicit and self-contained, which is pedagogically clearer.

- The card width is `'95%'`, which is relative to its parent. Because the FlatList itself is inside a `View` with `alignItems: 'center'`, the card is centred horizontally with a small margin on each side.

---

## Stage 6 — Delete and Error Messaging

### What this stage introduces

Three things arrive together, all of them user-facing in a way earlier stages were not.

1. A **Delete** button on each `UserCard`, so that a registered user can be removed from the list.
2. An **on-screen error message**, replacing the `console.log` used in earlier stages. The user is now told directly, in the UI, that fields are incomplete.
3. A **form reset** after a successful save, so that the fields are cleared and ready for the next entry.

The stage also formalises the identity of each record. The `id` field, added in Stage 4 to support `keyExtractor`, is now used as the basis for deletion. Deleting by `id` rather than by array index is a correction to the in-class code, and the reasoning is set out in the Revision notes.

### The code

```tsx
import { useState } from 'react'; 
import { 
  FlatList,
  StyleSheet, 
  Text, 
  TextInput, 
  TouchableHighlight, 
  View 
} from 'react-native';

// ---------------------------------------
// creating a user-defined component

type UserProps = { 
  fn: string; // abbrerviated to help differentiate variable names
  sn: string; 
  em: string; 
  pn: string; 
  ag: string;
  onDelete: () => void;
};

function UserCard({fn, sn, em, pn, ag, onDelete} : UserProps) {
  return(
    <View style={styles.card}>
      <View style={styles.cardHeader}>
        <Text style={styles.cardName}>{fn} {sn}</Text>
        <Text style={styles.cardAge}>{ag}</Text>
      </View>
      <View style={styles.cardDetails}>
        <Text style={styles.cardLabel}>Email</Text>
        <Text style={styles.cardText}>{em}</Text>
        <Text style={styles.cardLabel}>Phone</Text>
        <Text style={styles.cardText}>{pn}</Text>
      </View>
      <TouchableHighlight
        style={styles.deleteButton}
        onPress={onDelete}
      >
        <Text style={styles.deleteButtonText}>Delete</Text>
      </TouchableHighlight>
    </View>
  );
}
// ---------------------------------------

// structure for the user object 
type Registration = {
  id: string; 
  firstName: string;
  surname: string;
  email: string;
  phoneNumber: string;
  age: string;
};

export default function App() {
  const [firstName, setFirstName] = useState<string>(''); 
  const [surname, setSurname] = useState<string>(''); 
  const [email, setEmail] = useState<string>(''); 
  const [phoneNumber, setPhoneNumber] = useState<string>(''); 
  const [age, setAge] = useState<string>('');

  // array of objects ("registered users") 
  const [registeredUsers, setRegisteredUsers] = useState<Registration[]>([]);

  const [errMsg, setErrMsg] = useState<string>('');

  const handleSave = () => {
    // Initial Validation
    if (
      firstName.trim() === '' ||
      surname.trim() === '' ||
      email.trim() === '' ||
      phoneNumber.trim() === '' ||
      age.trim() === ''
    ) {
      setErrMsg('Please complete all fields');
      return;
    }

    // More validation of data needs to be performed

    // Create new registration object
    const newUser: Registration = {
      id: Date.now().toString(),
      firstName: firstName.trim(),
      surname: surname.trim(),
      email: email.trim(),
      phoneNumber: phoneNumber.trim(),
      age: age.trim(),
    };

    // Add object to array
    setRegisteredUsers(prevUsers => [...prevUsers, newUser]);

    // Clear the form 
    setFirstName('');
    setSurname('');
    setEmail('');
    setPhoneNumber('');
    setAge('');
  };

  const handleDelete = (id: string) => {
    setRegisteredUsers(prevUsers =>
      prevUsers.filter((user) => user.id !== id)
    );
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Registration</Text>
      <TextInput 
        value={firstName}
        onChangeText={setFirstName}
        style={styles.input}
        placeholder='First name'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={surname}
        onChangeText={setSurname}        
        style={styles.input}
        placeholder='Surname'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={email}
        onChangeText={setEmail}
        style={styles.input}
        placeholder='Email'
        maxLength={60}
        inputMode='email'
      />

      <TextInput 
        value={phoneNumber}
        onChangeText={setPhoneNumber}
        style={styles.input}
        placeholder='Phone Number'
        maxLength={15}
        inputMode='tel'
      />

      <TextInput 
        value={age}
        onChangeText={setAge}
        style={styles.input}
        placeholder='Age'
        maxLength={3}
        inputMode='numeric'
      />

      <TouchableHighlight style={styles.button} onPress={handleSave}>
        <Text style={styles.buttonText}>Submit</Text>
      </TouchableHighlight>

      <Text style={styles.errMessage}>{errMsg}</Text>

      <FlatList 
        data={registeredUsers}
        keyExtractor={(item) => item.id}
        renderItem={({item} : {item : Registration}) => 
          <UserCard
            fn={item.firstName}
            sn={item.surname}
            em={item.email}
            pn={item.phoneNumber}
            ag={item.age}
            onDelete={() => handleDelete(item.id)}
          />
        }
      />
    </View>
  );
}

const styles = StyleSheet.create({ 
  container: { 
    flex: 1, 
    justifyContent: 'center', 
    backgroundColor: '#fff', 
    padding: 10, 
    alignItems: 'center', 
  }, 
  errMessage: { 
    fontSize: 12, 
    color: '#ff0000', 
    fontWeight: 'bold', 
    height: 20, 
  }, 
  title: { 
    fontSize: 24, 
    fontWeight: 'bold', 
  }, 
  input: { 
    height: 35, 
    margin: 12, 
    borderWidth: 1, 
    padding: 10, 
  }, 
  button: { 
    height: 40, 
    margin: 10, 
    padding: 10, 
    justifyContent: 'center', 
    alignItems: 'center', 
    borderRadius: 5, 
    backgroundColor: '#333', 
  }, 
  buttonText: { 
    fontSize: 18, 
    color: '#fefefe', 
  }, 
  card: { 
    width: '95%', 
    backgroundColor: '#EAF4FF', 
    borderRadius: 8, 
    marginVertical: 5, 
    padding: 10, 
    borderLeftWidth: 5, 
    borderLeftColor: '#1976D2', 
  },
  cardHeader: { 
    flexDirection: 'row', 
    justifyContent: 'space-between', 
    alignItems: 'center', 
    marginBottom: 6, 
  },
  cardName: { 
    fontSize: 16, 
    fontWeight: 'bold', 
    color: '#1A1A1A', 
    flex: 1, 
  },
  cardAge: { 
    fontSize: 13, 
    fontWeight: 'bold', 
    color: '#1976D2', 
    backgroundColor: '#D6EBFF', 
    paddingHorizontal: 8, 
    paddingVertical: 3, 
    borderRadius: 10, 
  },
  cardDetails: { 
    borderTopWidth: 1, 
    borderTopColor: '#C9DFF5', 
    paddingTop: 6, 
  },
  cardLabel: { 
    fontSize: 10, 
    fontWeight: 'bold', 
    color: '#1976D2', 
    marginTop: 2, 
  },
  cardText: { 
    fontSize: 13, 
    color: '#333', 
    marginBottom: 3, 
  },
  deleteButton: {
    height: 35,
    marginTop: 8,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 5,
    backgroundColor: '#D32F2F',
  },
  deleteButtonText: {
    fontSize: 14,
    fontWeight: 'bold',
    color: '#FFFFFF',
  },
});
```

### Walkthrough

The changes fall into four areas: the `UserCard` component, the `App` state, the `handleSave` function, and the FlatList's `renderItem`.

**UserCard gains a Delete button.** The `UserProps` type gains a prop:

```tsx
onDelete: () => void;
```

The type `() => void` means "a function that takes no arguments and returns nothing". The name `onDelete` follows React convention for callback props: the prefix `on` signals that the prop is a function to be called in response to an event. `UserCard` does not know what deletion means — it only knows to call this function when its Delete button is tapped.

The component's parameter list now destructures `onDelete` alongside the five data props. At the bottom of the card's JSX, a `TouchableHighlight` is added:

```tsx
<TouchableHighlight
  style={styles.deleteButton}
  onPress={onDelete}
>
  <Text style={styles.deleteButtonText}>Delete</Text>
</TouchableHighlight>
```

Two styles are used here — `deleteButton` and `deleteButtonText` — and both are defined in the style sheet at the bottom of the file. The button is styled with a dark red background, white text, and rounded corners, visually distinguishing it from the rest of the card.

**App gains an error message state.** A new piece of state is declared:

```tsx
const [errMsg, setErrMsg] = useState<string>('');
```

The initial value is the empty string, meaning no error is shown. Whenever `setErrMsg` is called with a non-empty string, the message appears on screen. Whenever it is set back to `''`, the message disappears.

**handleSave changes in two ways.** First, the validation branch no longer logs to the console:

```tsx
if (
  firstName.trim() === '' ||
  ...
) {
  setErrMsg('Please complete all fields');
  return;
}
```

The message is now placed into state, which causes a re-render, and the re-render shows the message on screen via the `Text` element bound to `errMsg`.

Second, after a successful save, the form is cleared:

```tsx
// Clear the form 
setFirstName('');
setSurname('');
setEmail('');
setPhoneNumber('');
setAge('');
```

Without this, the fields would still contain the values the user just submitted, which is confusing when the new card appears in the list. Clearing them gives the form a fresh, ready-for-the-next-entry appearance.

Note that the error message is *not* cleared at this point. In practice it will usually be empty already, since the user only reaches the save logic by having filled all fields — and the last successful save would have set it to `''` had it been set. A more defensive version would clear it here explicitly. That is discussed in the Revision notes.

**handleDelete.** A new function is declared in `App`:

```tsx
const handleDelete = (id: string) => {
  setRegisteredUsers(prevUsers =>
    prevUsers.filter((user) => user.id !== id)
  );
};
```

It takes an `id` string and returns a new array containing only those users whose `id` is *not* the one passed in. `Array.prototype.filter` is the appropriate tool here: it keeps the elements for which the predicate returns `true`. So every user except the one to be deleted is kept; the deleted user's id fails the check and is excluded.

The functional updater form is used again — `prevUsers => ...` — for the same reasons as in Stage 3: the new array is computed from the most recent state, not from a possibly stale snapshot.

**The renderItem passes the id.** The FlatList's `renderItem` now supplies the delete callback:

```tsx
renderItem={({item} : {item : Registration}) => 
  <UserCard
    fn={item.firstName}
    sn={item.surname}
    em={item.email}
    pn={item.phoneNumber}
    ag={item.age}
    onDelete={() => handleDelete(item.id)}
  />
}
```

The crucial detail is the arrow function: `() => handleDelete(item.id)`. If the code had written `onDelete={handleDelete(item.id)}`, the function would be *called immediately* during render, and its return value — `undefined` — would become the value of `onDelete`. Passing a function that, *when called later*, invokes `handleDelete` with the correct id is what makes the button work as intended. This distinction between passing a function and calling it is one of the most common early mistakes in React, and it is worth emphasising.

### Key concepts

**Callback props.** A parent passes a function to a child so the child can notify the parent of an event. This is the standard way for data to flow *upwards* in React — parents pass data down via ordinary props and receive events up via callback props. `UserCard` does not know about `registeredUsers` or `setRegisteredUsers`; it only knows that when its button is tapped, it should call `onDelete`.

**Identity versus index.** Two ways to remove an element from an array are available: by its position (`index`) or by its identity (`id`). Position is fragile. Identity is stable. Stage 6 uses identity. The reasoning is laid out in the Revision notes.

**Derived user feedback.** The error message is not a separate concern bolted on top; it is ordinary state, rendered like any other state. When the validation fails, `setErrMsg` changes the state; when the next save succeeds, the state is unchanged and the message that was shown remains. A defensive version would clear it on success, and this is a good point for students to reason about.

**Form reset.** Clearing the form after save is a small piece of housekeeping that makes a large difference to how the application feels. It also reinforces the idea that the form's state and the list's state are separate: clearing the form does not affect the list.

### Revision notes and gotchas

**Deletion by id, not by index.** The in-class version of this stage deleted by array index:

```tsx
const handleDelete = (index: number) => {
  setRegisteredUsers(prevUsers =>
    prevUsers.filter((_, i) => i !== index)
  );
};
```

and the `renderItem` passed the index:

```tsx
renderItem={({item, index} : {item : Registration}) => 
  <UserCard
    ...
    onDelete={() => handleDelete(index)}
  />
}
```

This works correctly for the simple case — click Delete on the third card, the third element is removed. But it is fragile in three ways.

First, the `index` provided by `renderItem` is the position of the item *at the time the row was rendered*. If the list changes between render and click — for instance, if another user is added, or a previous user is deleted — the index will refer to the wrong element. React will usually re-render and supply a fresh index, but the coupling between the visual position of a row and its data is an unnecessary liability.

Second, `keyExtractor` uses `item.id`. React reconciles the list by matching keys. If a row's `key` is its id but its delete callback uses its index, the two identities can disagree. In practice this produces subtle bugs where clicking Delete on one card removes another.

Third, using `id` is self-documenting. The `id` was created for the purpose of identifying a record. Using it here is the natural expression of that purpose.

The recommended form, used in this document, is:

```tsx
const handleDelete = (id: string) => {
  setRegisteredUsers(prevUsers =>
    prevUsers.filter((user) => user.id !== id)
  );
};
```

with `renderItem` supplying `onDelete={() => handleDelete(item.id)}`. The predicate now reads "keep every user whose id is not the one to delete", which is exactly what the code intends.

The index-based version is still worth showing to students, because it makes the difference between position and identity concrete. But the version students take away should be the id-based one.

**Passing a function versus calling it.** The single most common mistake in this stage is writing `onPress={handleDelete(item.id)}` instead of `onPress={() => handleDelete(item.id)}`. The first calls `handleDelete` during render and passes its result. The second passes a function to be called later. If the button appears to do nothing, or if deleting one item deletes the wrong one, this is the first thing to check.

**The error message is not cleared on success.** The code leaves the error state as it is after a successful save. In practice this is usually harmless, because the only way to reach the success branch is to have cleared all the fields — and any prior error message was the consequence of *not* having cleared them. But the sequence "fail validation, then succeed" leaves the message on screen for one render before the next save. A fully defensive version would add `setErrMsg('')` at the top of the success branch, or inside the form-clearing block. This is a reasonable improvement for students to make themselves.

**The `keyExtractor` and `onDelete` are now paired correctly.** Both use `item.id`. This consistency is what makes the list reliable when items are added and removed. Mismatched identities — a `keyExtractor` based on `id` but a delete callback based on `index` — are a common source of bugs.

**`TouchableHighlight` versus `Pressable`.** `TouchableHighlight` is the older API and applies a highlight overlay when pressed. `Pressable` is the modern replacement and gives more control over pressed states. For this application, either works; the class used `TouchableHighlight` throughout, and that is preserved here for consistency.

**The `errMessage` style has a fixed height of 20.** This reserves space for the message even when there is none, so the layout does not shift vertically when an error appears or disappears. This is a small but useful detail.

---

## Stage 7 — Edit and Cancel

This stage is presented in two parts. Part A shows the code as it was taught in class, including a logic bug in the edit branch of `handleSave`. Part B shows the corrected version, with the fix explained. Presenting both is deliberate: the bug illustrates a real and easily-made mistake, and understanding the fix is more valuable than simply seeing corrected code.

### Part A — The buggy version

#### What this part introduces

The stage adds the ability to edit an existing user. Two pieces of state are introduced to support this:

1. A boolean-like piece of state, `editingUserId`, which is either `null` (meaning "we are adding a new user") or a string id (meaning "we are editing the user with this id").
2. New functions `handleEdit` and, in the JSX, a **Cancel** button.

`UserCard` also gains an `onEdit` callback prop and a corresponding Edit button.

#### The code

```tsx
import { useState } from 'react'; 
import { 
  FlatList,
  StyleSheet, 
  Text, 
  TextInput, 
  TouchableHighlight, 
  View 
} from 'react-native';

// ---------------------------------------
// creating a user-defined component

type UserProps = { 
  fn: string; // abbrerviated to help differentiate variable names
  sn: string; 
  em: string; 
  pn: string; 
  ag: string;
  onDelete: () => void;
  onEdit: () => void;
};

function UserCard({fn, sn, em, pn, ag, onDelete, onEdit} : UserProps) {
  return(
    <View style={styles.card}>
      <View style={styles.cardHeader}>
        <Text style={styles.cardName}>{fn} {sn}</Text>
        <Text style={styles.cardAge}>{ag}</Text>
      </View>
      <View style={styles.cardDetails}>
        <Text style={styles.cardLabel}>Email</Text>
        <Text style={styles.cardText}>{em}</Text>
        <Text style={styles.cardLabel}>Phone</Text>
        <Text style={styles.cardText}>{pn}</Text>
      </View>
      <View>
        <TouchableHighlight style={styles.deleteButton} onPress={onDelete}>
          <Text style={styles.deleteButtonText}>Delete</Text>
        </TouchableHighlight>
        <TouchableHighlight onPress={onEdit}>
          <Text>Edit</Text>
        </TouchableHighlight>
      </View>
    </View>
  );
}
// ---------------------------------------

// structure for the user object 
type Registration = {
  id: string; 
  firstName: string;
  surname: string;
  email: string;
  phoneNumber: string;
  age: string;
};

export default function App() {
  const [firstName, setFirstName] = useState<string>(''); 
  const [surname, setSurname] = useState<string>(''); 
  const [email, setEmail] = useState<string>(''); 
  const [phoneNumber, setPhoneNumber] = useState<string>(''); 
  const [age, setAge] = useState<string>('');

  // array of objects ("registered users") 
  const [registeredUsers, setRegisteredUsers] = useState<Registration[]>([]);

  const [errMsg, setErrMsg] = useState<string>('');

  // This state tells us whether we are editing an existing user.
  // null  = we are adding a new user
  // "value"   = we are editing the user whose id is "value"
  const [editingUserId, setEditingUserId] = useState<string | null>(null);

  const handleSave = () => {
    // Initial Validation
    if (
      firstName.trim() === '' ||
      surname.trim() === '' ||
      email.trim() === '' ||
      phoneNumber.trim() === '' ||
      age.trim() === ''
    ) {
      setErrMsg('Please complete all fields');
      return;
    }

    // More validation of data needs to be performed

    if (editingUserId !== null) {
      const updatedUsers = registeredUsers.map((user) => {
        user.id === editingUserId ? 
          { 
            ...user,
            firstName: firstName.trim(),
            surname: surname.trim(),
            email: email.trim(),
            phoneNumber: phoneNumber.trim(),
            age: age.trim(),
          } : user
        }
      });

      setRegisteredUsers(updatedUsers); 
      setEditingUserId(null); 
    } else {
      // Create new registration object
      const newUser: Registration = {
        id: Date.now().toString(),
        firstName: firstName.trim(),
        surname: surname.trim(),
        email: email.trim(),
        phoneNumber: phoneNumber.trim(),
        age: age.trim(),
      };
      // Add object to array
      setRegisteredUsers(prevUsers => [...prevUsers, newUser]);

      // Clear the form 
      setFirstName('');
      setSurname('');
      setEmail('');
      setPhoneNumber('');
      setAge('');
    }
  };

  const handleDelete = (id: string) => {
    setRegisteredUsers(prevUsers =>
      prevUsers.filter((user) => user.id !== id)
    );
  };

  const handleEdit = (user : Registration) => {
    setFirstName(user.firstName); 
    setSurname(user.surname); 
    setEmail(user.email); 
    setPhoneNumber(user.phoneNumber); 
    setAge(user.age); 
    
    setEditingUserId(user.id);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Registration</Text>
      <TextInput 
        value={firstName}
        onChangeText={setFirstName}
        style={styles.input}
        placeholder='First name'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={surname}
        onChangeText={setSurname}        
        style={styles.input}
        placeholder='Surname'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={email}
        onChangeText={setEmail}
        style={styles.input}
        placeholder='Email'
        maxLength={60}
        inputMode='email'
      />

      <TextInput 
        value={phoneNumber}
        onChangeText={setPhoneNumber}
        style={styles.input}
        placeholder='Phone Number'
        maxLength={15}
        inputMode='tel'
      />

      <TextInput 
        value={age}
        onChangeText={setAge}
        style={styles.input}
        placeholder='Age'
        maxLength={3}
        inputMode='numeric'
      />

      <TouchableHighlight style={styles.button} onPress={handleSave}>
        <Text style={styles.buttonText}>Submit</Text>
      </TouchableHighlight>

      <Text style={styles.errMessage}>{errMsg}</Text>

      {editingUserId !== null && (
        <TouchableHighlight 
          style={styles.button} 
          onPress={() => {
            setEditingUserId(null);
            setFirstName('');
            setSurname('');
            setEmail('');
            setPhoneNumber('');
            setAge('');
          }}>
        <Text style={styles.buttonText}>Cancel</Text>
      </TouchableHighlight>
      )}

      <Text style={styles.title}>Users</Text>

      <FlatList 
        data={registeredUsers}
        keyExtractor={(item) => item.id}
        renderItem={({item, index} : {item : Registration}) => 
          <UserCard
            fn={item.firstName}
            sn={item.surname}
            em={item.email}
            pn={item.phoneNumber}
            ag={item.age}
            onDelete={() => handleDelete(item.id)}
            onEdit={() => handleEdit(item)}
          />
        }
      />
    </View>
  );
}

const styles = StyleSheet.create({ 
  container: { 
    flex: 1, 
    justifyContent: 'center', 
    backgroundColor: '#fff', 
    padding: 10, 
    alignItems: 'center', 
  }, 
  errMessage: { 
    fontSize: 12, 
    color: '#ff0000', 
    fontWeight: 'bold', 
    height: 20, 
  }, 
  title: { 
    fontSize: 24, 
    fontWeight: 'bold', 
  }, 
  input: { 
    height: 35, 
    margin: 12, 
    borderWidth: 1, 
    padding: 10, 
  }, 
  button: { 
    height: 40, 
    margin: 10, 
    padding: 10, 
    justifyContent: 'center', 
    alignItems: 'center', 
    borderRadius: 5, 
    backgroundColor: '#333', 
  }, 
  buttonText: { 
    fontSize: 18, 
    color: '#fefefe', 
  }, 
  card: { 
    width: '95%', 
    backgroundColor: '#EAF4FF', 
    borderRadius: 8, 
    marginVertical: 5, 
    padding: 10, 
    borderLeftWidth: 5, 
    borderLeftColor: '#1976D2', 
  },
  cardHeader: { 
    flexDirection: 'row', 
    justifyContent: 'space-between', 
    alignItems: 'center', 
    marginBottom: 6, 
  },
  cardName: { 
    fontSize: 16, 
    fontWeight: 'bold', 
    color: '#1A1A1A', 
    flex: 1, 
  },
  cardAge: { 
    fontSize: 13, 
    fontWeight: 'bold', 
    color: '#1976D2', 
    backgroundColor: '#D6EBFF', 
    paddingHorizontal: 8, 
    paddingVertical: 3, 
    borderRadius: 10, 
  },
  cardDetails: { 
    borderTopWidth: 1, 
    borderTopColor: '#C9DFF5', 
    paddingTop: 6, 
  },
  cardLabel: { 
    fontSize: 10, 
    fontWeight: 'bold', 
    color: '#1976D2', 
    marginTop: 2, 
  },
  cardText: { 
    fontSize: 13, 
    color: '#333', 
    marginBottom: 3, 
  },
  deleteButton: {
    height: 35,
    marginTop: 8,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 5,
    backgroundColor: '#D32F2F',
  },
  deleteButtonText: {
    fontSize: 14,
    fontWeight: 'bold',
    color: '#FFFFFF',
  },
});
```

#### Walkthrough

**UserCard gains `onEdit`.** The `UserProps` type includes a second callback prop, `onEdit: () => void`, and the component destructures it alongside `onDelete`. Inside the card, a second `TouchableHighlight` is rendered below the Delete button:

```tsx
<TouchableHighlight onPress={onEdit}>
  <Text>Edit</Text>
</TouchableHighlight>
```

Note that this Edit button has **no style applied** in the in-class version — no `style` prop on the `TouchableHighlight`, and no `style` on the inner `Text`. It therefore renders as plain text with the default touchable behaviour. Part B addresses this.

**App gains `editingUserId` state.** The declaration:

```tsx
const [editingUserId, setEditingUserId] = useState<string | null>(null);
```

The type `string | null` is a *union*: the value is either a string or `null`. The initial value is `null`, meaning the form is in "add" mode. When the user taps Edit on a card, the id of that card's user is stored, and the mode switches to "edit". The comment above the declaration records this convention.

**handleEdit.** This function is called when the user taps Edit on a card:

```tsx
const handleEdit = (user : Registration) => {
  setFirstName(user.firstName); 
  setSurname(user.surname); 
  setEmail(user.email); 
  setPhoneNumber(user.phoneNumber); 
  setAge(user.age); 
  
  setEditingUserId(user.id);
};
```

It receives the full user object, copies its fields into the form state, and records the id in `editingUserId`. The effect is that the form fields are populated with the values of the card the user chose to edit, and the application now knows which record those values correspond to.

**handleSave gains a branch.** The function now checks `editingUserId`:

```tsx
if (editingUserId !== null) {
  const updatedUsers = registeredUsers.map((user) => {
    user.id === editingUserId ? 
      { 
        ...user,
        firstName: firstName.trim(),
        surname: surname.trim(),
        email: email.trim(),
        phoneNumber: phoneNumber.trim(),
        age: age.trim(),
      } : user
    }
  });

  setRegisteredUsers(updatedUsers); 
  setEditingUserId(null); 
} else {
  // ... existing add logic
}
```

The intent is clear. If `editingUserId` is not null, we are in edit mode. Build a new array by mapping over the existing users: for the user whose id matches `editingUserId`, produce an updated object; for every other user, keep it unchanged. Then set the array and reset `editingUserId` back to `null`, returning the application to add mode.

**The bug.** The map callback has two defects.

First, the ternary expression — `user.id === editingUserId ? {...} : user` — is not returned. A `map` callback must **return** the value that should replace the current element. Here, the callback is written as an arrow function with a *block body* (curly braces), which means the value of the expression is discarded unless preceded by `return`. The result is that every iteration returns `undefined`, and `updatedUsers` becomes an array of `undefined` values.

Second, the closing brace of the block is placed after the ternary rather than before. The structure

```tsx
{
  ...ternary...
  } : user
}
```

has the `} : user` outside the block. The intent was:

```tsx
{
  return user.id === editingUserId ? {...} : user;
}
```

There is also a missing semicolon inside the block after the ternary, though this is the least serious of the three issues.

The visible symptom is that after tapping Edit, changing a value, and pressing Submit, the list of users disappears and is replaced by blank rows. The data has been wiped, not because the add path is broken, but because the edit path replaced the array with `undefined` placeholders.

**Cancel button.** Below the Submit button, a Cancel button is rendered conditionally:

```tsx
{editingUserId !== null && (
  <TouchableHighlight 
    style={styles.button} 
    onPress={() => {
      setEditingUserId(null);
      setFirstName('');
      setSurname('');
      setEmail('');
      setPhoneNumber('');
      setAge('');
    }}>
  <Text style={styles.buttonText}>Cancel</Text>
</TouchableHighlight>
)}
```

The `&&` operator in JSX is a common pattern for conditional rendering: the expression `editingUserId !== null && (...)` evaluates to the JSX on the right if the condition is true, and to `false` if not. React does not render `false`. So the Cancel button appears only when in edit mode.

Its `onPress` handler clears the editing state and resets all form fields, returning the application to add mode. This is the same reset logic that the add path uses after a successful save, duplicated here for clarity.

**The "Users" heading.** A second heading is rendered above the FlatList, giving the list a visible label. This is a purely cosmetic addition, using the same `title` style as the form heading.

#### Key concepts

**Union types for mode.** `string | null` is a common way to model "a value or the absence of one". Using `null` rather than `''` is deliberate: an empty string could be a legitimate id (if ids were ever allowed to be empty), whereas `null` unambiguously means "no value". The pattern is often called a *discriminated* or *tagged* mode, since the presence or absence of a value determines the behaviour of the form.

**Update by mapping.** To update one element of an array without mutating it, `map` is used. The callback returns the updated object for the matching element and the original object for everything else. This preserves immutability and gives React a new array to compare with the old one.

**Conditional rendering with `&&`.** The `condition && jsx` pattern is idiomatic in React for showing something only when a condition holds. It is important to remember that if the left side is a number or a string, `&&` will render that number or string rather than nothing — so the pattern is safest when the left side is a boolean.

**The danger of block-bodied arrow functions.** A common source of bugs is confusing an arrow function with an expression body — `(x) => x * 2` — with one that has a block body — `(x) => { x * 2 }`. The first returns the result; the second does not, unless it contains an explicit `return`. The Stage 7 bug is a direct example of this.

### Part B — The corrected version

#### What this part changes

Three corrections and two completions.

**Corrections.**

1. The `map` callback in the edit branch is rewritten to return the ternary result, with the brace placement corrected.
2. The functional-updater form is used for the array update in the edit branch, for consistency with the add and delete paths.
3. The form is cleared after a successful edit, matching the add path.

**Completions.**

1. The Edit button is styled, matching the Delete button's shape but in a different colour.
2. The Submit button's label changes to "Update" while in edit mode, so the user can see which mode they are in. This is a small UI affordance that makes the application's state visible.

A further detail is added: the error message is cleared on a successful save or update. This is the defensive improvement noted in Stage 6, and it is natural to introduce here since the save logic is being touched.

#### The code

```tsx
import { useState } from 'react'; 
import { 
  FlatList,
  StyleSheet, 
  Text, 
  TextInput, 
  TouchableHighlight, 
  View 
} from 'react-native';

// ---------------------------------------
// creating a user-defined component

type UserProps = { 
  fn: string; // abbrerviated to help differentiate variable names
  sn: string; 
  em: string; 
  pn: string; 
  ag: string;
  onDelete: () => void;
  onEdit: () => void;
};

function UserCard({fn, sn, em, pn, ag, onDelete, onEdit} : UserProps) {
  return(
    <View style={styles.card}>
      <View style={styles.cardHeader}>
        <Text style={styles.cardName}>{fn} {sn}</Text>
        <Text style={styles.cardAge}>{ag}</Text>
      </View>
      <View style={styles.cardDetails}>
        <Text style={styles.cardLabel}>Email</Text>
        <Text style={styles.cardText}>{em}</Text>
        <Text style={styles.cardLabel}>Phone</Text>
        <Text style={styles.cardText}>{pn}</Text>
      </View>
      <View style={styles.cardActions}>
        <TouchableHighlight 
          style={styles.editButton} 
          onPress={onEdit}
        >
          <Text style={styles.editButtonText}>Edit</Text>
        </TouchableHighlight>
        <TouchableHighlight 
          style={styles.deleteButton} 
          onPress={onDelete}
        >
          <Text style={styles.deleteButtonText}>Delete</Text>
        </TouchableHighlight>
      </View>
    </View>
  );
}
// ---------------------------------------

// structure for the user object 
type Registration = {
  id: string; 
  firstName: string;
  surname: string;
  email: string;
  phoneNumber: string;
  age: string;
};

export default function App() {
  const [firstName, setFirstName] = useState<string>(''); 
  const [surname, setSurname] = useState<string>(''); 
  const [email, setEmail] = useState<string>(''); 
  const [phoneNumber, setPhoneNumber] = useState<string>(''); 
  const [age, setAge] = useState<string>('');

  // array of objects ("registered users") 
  const [registeredUsers, setRegisteredUsers] = useState<Registration[]>([]);

  const [errMsg, setErrMsg] = useState<string>('');

  // This state tells us whether we are editing an existing user.
  // null      = we are adding a new user
  // "value"   = we are editing the user whose id is "value"
  const [editingUserId, setEditingUserId] = useState<string | null>(null);

  // Small helper so the same form-reset logic is not duplicated
  const clearForm = () => {
    setFirstName('');
    setSurname('');
    setEmail('');
    setPhoneNumber('');
    setAge('');
  };

  const handleSave = () => {
    // Initial Validation
    if (
      firstName.trim() === '' ||
      surname.trim() === '' ||
      email.trim() === '' ||
      phoneNumber.trim() === '' ||
      age.trim() === ''
    ) {
      setErrMsg('Please complete all fields');
      return;
    }

    // More validation of data needs to be performed

    // Clear any previous error now that the data is valid
    setErrMsg('');

    if (editingUserId !== null) {
      // Edit mode: replace the matching user with an updated object
      setRegisteredUsers(prevUsers =>
        prevUsers.map((user) =>
          user.id === editingUserId
            ? {
                ...user,
                firstName: firstName.trim(),
                surname: surname.trim(),
                email: email.trim(),
                phoneNumber: phoneNumber.trim(),
                age: age.trim(),
              }
            : user
        )
      );

      setEditingUserId(null);
      clearForm();
    } else {
      // Add mode: create a new user object and append it
      const newUser: Registration = {
        id: Date.now().toString(),
        firstName: firstName.trim(),
        surname: surname.trim(),
        email: email.trim(),
        phoneNumber: phoneNumber.trim(),
        age: age.trim(),
      };

      setRegisteredUsers(prevUsers => [...prevUsers, newUser]);
      clearForm();
    }
  };

  const handleDelete = (id: string) => {
    setRegisteredUsers(prevUsers =>
      prevUsers.filter((user) => user.id !== id)
    );
  };

  const handleEdit = (user : Registration) => {
    setFirstName(user.firstName); 
    setSurname(user.surname); 
    setEmail(user.email); 
    setPhoneNumber(user.phoneNumber); 
    setAge(user.age); 
    
    setEditingUserId(user.id);
  };

  const handleCancel = () => {
    setEditingUserId(null);
    clearForm();
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>
        {editingUserId !== null ? 'Edit Registration' : 'Registration'}
      </Text>
      <TextInput 
        value={firstName}
        onChangeText={setFirstName}
        style={styles.input}
        placeholder='First name'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={surname}
        onChangeText={setSurname}        
        style={styles.input}
        placeholder='Surname'
        maxLength={30}
        inputMode='text'
      />

      <TextInput 
        value={email}
        onChangeText={setEmail}
        style={styles.input}
        placeholder='Email'
        maxLength={60}
        inputMode='email'
      />

      <TextInput 
        value={phoneNumber}
        onChangeText={setPhoneNumber}
        style={styles.input}
        placeholder='Phone Number'
        maxLength={15}
        inputMode='tel'
      />

      <TextInput 
        value={age}
        onChangeText={setAge}
        style={styles.input}
        placeholder='Age'
        maxLength={3}
        inputMode='numeric'
      />

      <TouchableHighlight style={styles.button} onPress={handleSave}>
        <Text style={styles.buttonText}>
          {editingUserId !== null ? 'Update' : 'Submit'}
        </Text>
      </TouchableHighlight>

      <Text style={styles.errMessage}>{errMsg}</Text>

      {editingUserId !== null && (
        <TouchableHighlight 
          style={styles.button} 
          onPress={handleCancel}>
          <Text style={styles.buttonText}>Cancel</Text>
        </TouchableHighlight>
      )}

      <Text style={styles.title}>Users</Text>

      <FlatList 
        data={registeredUsers}
        keyExtractor={(item) => item.id}
        renderItem={({item} : {item : Registration}) => 
          <UserCard
            fn={item.firstName}
            sn={item.surname}
            em={item.email}
            pn={item.phoneNumber}
            ag={item.age}
            onDelete={() => handleDelete(item.id)}
            onEdit={() => handleEdit(item)}
          />
        }
      />
    </View>
  );
}

const styles = StyleSheet.create({ 
  container: { 
    flex: 1, 
    justifyContent: 'center', 
    backgroundColor: '#fff', 
    padding: 10, 
    alignItems: 'center', 
  }, 
  errMessage: { 
    fontSize: 12, 
    color: '#ff0000', 
    fontWeight: 'bold', 
    height: 20, 
  }, 
  title: { 
    fontSize: 24, 
    fontWeight: 'bold', 
  }, 
  input: { 
    height: 35, 
    margin: 12, 
    borderWidth: 1, 
    padding: 10, 
  }, 
  button: { 
    height: 40, 
    margin: 10, 
    padding: 10, 
    justifyContent: 'center', 
    alignItems: 'center', 
    borderRadius: 5, 
    backgroundColor: '#333', 
  }, 
  buttonText: { 
    fontSize: 18, 
    color: '#fefefe', 
  }, 
  card: { 
    width: '95%', 
    backgroundColor: '#EAF4FF', 
    borderRadius: 8, 
    marginVertical: 5, 
    padding: 10, 
    borderLeftWidth: 5, 
    borderLeftColor: '#1976D2', 
  },
  cardHeader: { 
    flexDirection: 'row', 
    justifyContent: 'space-between', 
    alignItems: 'center', 
    marginBottom: 6, 
  },
  cardName: { 
    fontSize: 16, 
    fontWeight: 'bold', 
    color: '#1A1A1A', 
    flex: 1, 
  },
  cardAge: { 
    fontSize: 13, 
    fontWeight: 'bold', 
    color: '#1976D2', 
    backgroundColor: '#D6EBFF', 
    paddingHorizontal: 8, 
    paddingVertical: 3, 
    borderRadius: 10, 
  },
  cardDetails: { 
    borderTopWidth: 1, 
    borderTopColor: '#C9DFF5', 
    paddingTop: 6, 
  },
  cardLabel: { 
    fontSize: 10, 
    fontWeight: 'bold', 
    color: '#1976D2', 
    marginTop: 2, 
  },
  cardText: { 
    fontSize: 13, 
    color: '#333', 
    marginBottom: 3, 
  },
  cardActions: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginTop: 8,
  },
  editButton: {
    flex: 1,
    height: 35,
    marginRight: 4,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 5,
    backgroundColor: '#1976D2',
  },
  editButtonText: {
    fontSize: 14,
    fontWeight: 'bold',
    color: '#FFFFFF',
  },
  deleteButton: {
    flex: 1,
    height: 35,
    marginLeft: 4,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 5,
    backgroundColor: '#D32F2F',
  },
  deleteButtonText: {
    fontSize: 14,
    fontWeight: 'bold',
    color: '#FFFFFF',
  },
});
```

#### Walkthrough of the corrections

**The corrected `handleSave`.** The edit branch now reads:

```tsx
setRegisteredUsers(prevUsers =>
  prevUsers.map((user) =>
    user.id === editingUserId
      ? {
          ...user,
          firstName: firstName.trim(),
          surname: surname.trim(),
          email: email.trim(),
          phoneNumber: phoneNumber.trim(),
          age: age.trim(),
        }
      : user
  )
);
```

Three things have changed relative to Part A.

First, the `map` callback uses an **expression body** rather than a block body. Written as `(user) => user.id === ... ? {...} : user`, the arrow function returns the value of the ternary automatically. No explicit `return` is needed because there are no curly braces around the body. This is the simplest correction to the bug.

Second, the assignment `const updatedUsers = ...` followed by `setRegisteredUsers(updatedUsers)` has been folded into a single `setRegisteredUsers(prevUsers => ...)` call. This uses the functional updater form, consistent with the add and delete paths. It also removes the need for the intermediate variable.

Third, the ternary is now the entire body of the arrow function, so there is no `} : user` placement problem. The brace that closed the block in Part A, and the stray `} : user`, are gone.

**Clearing the form after edit.** After the array has been updated and `editingUserId` has been reset to `null`, the form is cleared:

```tsx
setEditingUserId(null);
clearForm();
```

`clearForm` is a small helper introduced at the top of the component:

```tsx
const clearForm = () => {
  setFirstName('');
  setSurname('');
  setEmail('');
  setPhoneNumber('');
  setAge('');
};
```

It replaces the five separate setter calls that appeared in Part A's add branch and Cancel handler. The change is not strictly necessary for correctness, but it removes duplication and ensures that all three paths that reset the form — add, edit, and cancel — do so identically. If the form gained a sixth field, only `clearForm` would need updating.

**handleCancel.** In Part A, the Cancel button's `onPress` contained an inline function that reset five pieces of state and cleared `editingUserId`. In Part B, that logic is extracted into a named function:

```tsx
const handleCancel = () => {
  setEditingUserId(null);
  clearForm();
};
```

The button now calls `onPress={handleCancel}`. This keeps the JSX clean and gives the behaviour a name that can be referenced in explanations and tests.

**Clearing the error message on success.** At the top of `handleSave`, immediately after the validation branch, is:

```tsx
// Clear any previous error now that the data is valid
setErrMsg('');
```

This is the defensive improvement noted in Stage 6. If the user submits an incomplete form, sees the error, then fills the fields and submits again, the error is removed as soon as the second submission passes validation. Without this line, the error would remain on screen until the next render cycle, which is a minor but visible glitch.

**Styling the Edit button.** The Edit and Delete buttons are now styled to match each other in shape and to differ in colour. The `UserProps` and `UserCard` code adds a wrapper `View` around both buttons:

```tsx
<View style={styles.cardActions}>
  <TouchableHighlight style={styles.editButton} onPress={onEdit}>
    <Text style={styles.editButtonText}>Edit</Text>
  </TouchableHighlight>
  <TouchableHighlight style={styles.deleteButton} onPress={onDelete}>
    <Text style={styles.deleteButtonText}>Delete</Text>
  </TouchableHighlight>
</View>
```

The `cardActions` style is a horizontal row with `justifyContent: 'space-between'` and a small top margin. Each button has `flex: 1`, so they share the available width equally. The Edit button has a small `marginRight`, the Delete button a small `marginLeft`, so there is a gap between them without either touching the card edges.

The Edit button uses the blue already used in the card's left border, `#1976D2`, with white text. The Delete button keeps the dark red, `#D32F2F`. The two are immediately distinguishable and the row is visually balanced.

**The Submit button label.** The main action button now shows a label that depends on the mode:

```tsx
<Text style={styles.buttonText}>
  {editingUserId !== null ? 'Update' : 'Submit'}
</Text>
```

When the form is in add mode, the button says "Submit"; in edit mode, it says "Update". This is a small change, but it turns the application's internal state into something the user can see. It also reduces the chance that a user believes they are adding a new record when in fact they are editing an existing one.

**The form heading.** A similar conditional is used for the title:

```tsx
<Text style={styles.title}>
  {editingUserId !== null ? 'Edit Registration' : 'Registration'}
</Text>
```

In add mode, the heading reads "Registration"; in edit mode, "Edit Registration". Together with the button label, this makes the mode visible in two places, which is a reasonable balance between clarity and noise.

#### Key concepts

**Expression bodies versus block bodies in arrow functions.** The bug in Part A is a direct consequence of confusing these two forms. An arrow function written as `(x) => expr` returns `expr`. An arrow function written as `(x) => { expr }` runs `expr` and returns `undefined`, unless there is an explicit `return`. Recognising this difference is an essential skill when reading and writing React code, since render props and callbacks frequently use the expression form.

**Updating one element of an array.** The `map`-and-conditional pattern is the standard way to update a single element without mutation. It is worth comparing with the alternative — finding the index of the element and splicing it out — which mutates the array and is therefore not appropriate for React state.

**Mode-dependent rendering.** The pattern `condition ? A : B` used inside JSX lets a single component present different labels or even different structures depending on state. Overusing it makes JSX hard to read, but for a small number of mode-dependent pieces — a heading, a button label — it is clear and concise.

**Extracting helpers.** `clearForm` and `handleCancel` are examples of the general principle: if the same sequence of operations appears in more than one place, give it a name. The name is the documentation, and future changes happen in one place.

#### Revision notes and gotchas

- The corrected `map` callback uses an expression body. If a reader is more comfortable with block bodies, an equally valid form is `(user) => { return user.id === editingUserId ? {...} : user; }`. Both are correct. What matters is that the value is returned, and the brace placement is consistent.

- The `...user` spread inside the ternary is important. Writing only the changed fields would produce a new object missing `id`, which would break `keyExtractor` and `onDelete`. The spread carries forward every field not explicitly overridden — here, just `id` — so the updated record is still a complete `Registration`.

- After a successful edit, the form is cleared and `editingUserId` is reset to `null`. It is important that both happen. Leaving `editingUserId` set would leave the button reading "Update" and the next submission would attempt to edit a record that may no longer exist.

- The `clearForm` helper is a very small refactor, but it is the kind of habit worth encouraging. The duplication in Part A — five setter calls repeated in the add branch and again in the Cancel handler — is exactly the kind of thing that drifts out of sync as an application grows.

- The Edit button's colour matches the card's left border colour. This is a deliberate visual consistency. Neither the colour nor the styling affects the logic, but applying a small amount of design intent makes the application feel finished.

- `onDelete` and `onEdit` both take no arguments in `UserProps`, and the parent supplies argument-free closures — `() => handleDelete(item.id)` and `() => handleEdit(item)`. This is the correct pattern when a child needs to report an event that refers to specific parent state. The child does not know about ids or users; it only knows that a button was pressed. The parent supplies the context.

- `handleEdit` does not clear the error message. If a user submits an incomplete form, then taps Edit, the old error remains visible above the form while the form is now populated. A defensive version would call `setErrMsg('')` inside `handleEdit`. This is left as an exercise, but it is the kind of detail that distinguishes a classroom example from a polished application.

---

### Closing note

The application now supports the full lifecycle: add a user, view users, edit a user, and delete a user. Each capability arrived in its own stage, and each stage introduced a small number of React or React Native ideas. The cumulative structure of the code reflects that progression: the state grows, the functions multiply, the JSX is refined, and the style sheet expands.

The most important ideas to carry forward from this sequence are: state is replaced, not mutated; updates that depend on previous state use the functional updater form; identity is more reliable than position; props flow down and callbacks flow up; and a surprising number of bugs come down to a missing `return`, a function called instead of passed, or a stray brace.