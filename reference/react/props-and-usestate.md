# React: props and useState

The Hello World component from [the setup page](setup-components-jsx.md) always says the same thing. A real component needs two more things: data handed in from outside (props) and data it remembers between redraws (state). The one idea to hold onto from this page: **when state changes, React runs your whole component function again, top to bottom.** That's why `useState` exists, and it's also why the infinite loop on [the useEffect page](useeffect-and-loading-data.md) happens.

## Quick example

From `client/example2.jsx` in [React-Functional-Components-Done](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components-Done/blob/master/client/example2.jsx):

```jsx
const React = require('react');
const { useState } = React;
const { createRoot } = require('react-dom/client');

const HelloUser = (props) => {
  const [username, setUsername] = useState(props.username);

  return (
    <div>
      <p>Hello {username}</p>
      <label>Change Name: </label>
      <input
        type="text"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
      />
    </div>
  );
};

const init = () => {
  const root = createRoot(document.getElementById('app'));
  root.render(<HelloUser username="JP" />);
};

window.onload = init;
```

Three pieces:

- **`props`** is the object React builds from the attributes on the tag. `<HelloUser username="JP" />` gives you `props.username`.
- **`useState(props.username)`** says "remember a value for this component." It hands back the current value and a function that changes it.
- **`setUsername(x)`** stores `x`, then runs `HelloUser` again so the page shows it.

The done repo writes `React.useState(...)` instead of pulling `useState` out at the top. Same function. We destructure it because the next page adds `useEffect` to the same line.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 10B (Wed 10/28) | Props, a text box React won't let you type in, why a plain variable doesn't work, `useState`, the whole function re-runs | Starter: [React-Functional-Components](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components) · Done: [React-Functional-Components-Done](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components-Done) |

Where to look in the done repo:

- `client/example2.jsx`: `HelloUser` (the component) and `init` (renders it once)
- `server/router.js`: `/example2` is the page that loads this bundle. Each example page loads only its own bundle, so typing in `example2.jsx` and refreshing `/` shows you Monday's Hello World instead.

Video: [Austin, Week 10.1: React FSCs](https://www.youtube.com/watch?v=kAMb0sEp9js) (43:47) covers props and the function-component shape. It predates hooks being standard, so it re-renders from outside with `ReactDOM.render`, which React 19 removed. [Austin, Week 10.2: React Class Components](https://www.youtube.com/watch?v=EzgxSVN-AzI) (39:12) builds the same HelloUser the old way, with `this.state` and `setState`. Watch it if you want to recognize that style in older code. Don't write it.

Used in: DomoMaker D (`App` holds a true/false in `useState` and hands `DomoForm` a `triggerReload` function as a prop), Project 2 (every dynamic component).

## Common mistakes

- **The page says `Hello ` with no name.** The prop is spelled differently on the tag and in the component. `userName="JP"` on the tag and `props.username` inside gives you nothing, and no error. Props are case-sensitive.
- **You can't type in the box.** You gave the input a `value` and no `onChange`. The console says so: `You provided a 'value' prop to a form field without an 'onChange' handler. This will render a read-only field.` React owns the box now and keeps putting the value back. This is the "React is gonna fight you" moment, and it's on purpose: React won't let what's on screen drift from the data.
- **The variable changes but the page doesn't.** You wrote `let username = props.username;` and assign to it in `onChange`. The console shows the new value, the box snaps back. A plain variable never tells React anything, and the next run starts over from `props.username`. That's the job `useState` does.
- **Build error `Identifier 'username' has already been declared`.** You added the `useState` line and kept the `let` line. The `useState` line replaces it.
- **Typing throws `Assignment to constant variable`.** You swapped in `useState` but `onChange` still does `username = e.target.value`. Call the setter instead: `setUsername(e.target.value)`.
- **The page goes blank on the first keystroke, console says `Objects are not valid as a React child (found: object with keys {_reactName, ...})`.** You wrote `onChange={setUsername}`. That hands the setter the whole event object, not the text. Use `(e) => setUsername(e.target.value)`.
- **Blank page, console `useState is not defined`.** The `const { useState } = React;` line is missing.
- **Blank page, Terminal 1 says `ERROR`.** The build failed. A broken build leaves the page blank with nothing in the browser console. Terminal 1 has the file and line.
- **You saved and nothing changed.** `npm run watchBundle` isn't running, you didn't refresh, or you're on `/` instead of `/example2`.

## What you'll find on Google

- **Class components: `constructor`, `this.state`, `this.setState`, `render()`.** The old way, and most older tutorials and Stack Overflow answers use it. It still runs on React 19, but you shouldn't write new code in it. The table below maps it onto what we did. Austin's class-component video walks through the whole thing if you want to see it.

  | Class version | Ours |
  |---|---|
  | `constructor` + `this.state = { username: ... }` | `useState(props.username)` |
  | `this.state.username` | `username` |
  | `this.setState({ username: ... })` | `setUsername(...)` |
  | `render() { return ... }` | the function itself returns |

- **`import { useState } from 'react'` instead of `require`.** Webpack handles both. We match the starter and DomoMaker, which use `require`.
- **`React.useState(...)` instead of `useState(...)`.** Same function, with or without the destructuring line. Austin's done repo writes it the long way.
- **State that holds an object, updated with spread: `setForm({ ...form, name: e.target.value })`.** Fine once you have several fields in one form. We keep one piece of state per value, which is also how DomoMaker D does it.
- **Uncontrolled inputs with `useRef`, or `defaultValue`.** Leave off `value` and React leaves the box alone, then you read it on submit. DomoMaker D's forms work that way. The controlled version on this page (`value` plus `onChange`) is what you want when something else on the page has to react as the user types.
- **Form libraries (Formik, React Hook Form).** Overkill for anything in this class.

## Redo it

Start in your clone of the [starter](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components) with the setup from [the setup page](setup-components-jsx.md) working: Terminal 1 running `npm run watchBundle`, Terminal 2 running `npm start`, the browser on `localhost:3000/example2` with the DevTools console open. `client/example2.jsx` is empty in the starter, and that's the only file you touch. After every save, wait for Terminal 1 to say `compiled successfully`, then refresh. The browser doesn't refresh itself.

This is browser code, so `console.log` prints in the browser console, not the terminal.

### 1. The two requires

```jsx
const React = require('react');
const { createRoot } = require('react-dom/client');
```

Same two lines as `example1.jsx`.

### 2. A component that takes props

Under the requires:

```jsx
const HelloUser = (props) => {
  return (
    <div>
      <p>Hello {props.username}</p>
    </div>
  );
};
```

`{props.username}` means "put this JavaScript value here," the same curly braces as `{2 + 2}` on Monday. Save. The page is still blank because nothing renders it yet.

### 3. Render it with a username

Bottom of the file:

```jsx
const init = () => {
  const root = createRoot(document.getElementById('app'));
  root.render(<HelloUser username="JP" />);
};

window.onload = init;
```

Checkpoint: refresh. **Hello JP**. Props look like HTML attributes, and that's what they are: whatever you write on the tag shows up on the `props` object.

Optional: render two of them and see that each gets its own props.

```jsx
root.render(
  <div>
    <HelloUser username="JP" />
    <HelloUser username="IGME 430" />
  </div>
);
```

Put it back to a single `<HelloUser username="JP" />` before going on. The checkpoints below assume one.

### 4. A text box React owns

Inside the `<div>`, under the `<p>`:

```jsx
      <p>Hello {props.username}</p>   // already there
      <label>Change Name: </label>
      <input type="text" value={props.username} />
```

Checkpoint: refresh, click in the box, type. Nothing happens. The box stays `JP`, and the console warns you about a `value` prop with no `onChange`. React now owns what's in that box, and it shows the value you gave it and nothing else.

### 5. A plain variable, changed on every keystroke (it won't work, on purpose)

Two edits. First, add a `let` as the first line inside `HelloUser`, and change the `<p>` to read from it:

```jsx
const HelloUser = (props) => {   // already there
  let username = props.username;

  return (   // already there
    <div>   // already there
      <p>Hello {username}</p>   // was {props.username}
```

Then replace the whole `<input ... />` line:

```jsx
      <input
        type="text"
        value={username}
        onChange={(e) => {
          username = e.target.value;
          console.log(username);
        }}
      />
```

`onChange` is camelCase, and it fires on every keystroke, not when you leave the box like HTML's `onchange`. `e.target.value` is what the box would say after this keystroke.

Checkpoint: refresh, click at the end of the box, type `a`, then `b`. The console prints `JPa`, then `JPb`. Not `JPab`. The page still says **Hello JP** and the box still says `JP`.

The variable changed. It's right there in the console. But React doesn't know, so it put the box back, and the next run of `HelloUser` started over at `props.username`. That's why the second keystroke gives `JPb`.

### 6. Pull in useState and swap the variable for state

A new line between the two requires. Nothing gets replaced here:

```jsx
const React = require('react');   // already there
const { useState } = React;
const { createRoot } = require('react-dom/client');   // already there
```

Now replace the `let username = props.username;` line. Don't keep both.

```jsx
  const [username, setUsername] = useState(props.username);
```

`useState(startValue)` hands back two things in an array: the current value and a function that changes it. `const [a, b] = ...` takes item 0 and item 1 and names them. Calling the second one `setSomething` is a habit, not a rule.

Checkpoint: refresh. **Hello JP**, no error yet. Type one letter and the console throws `Assignment to constant variable`, because `onChange` is still assigning to `username`. That's the next step.

### 7. Call the setter instead

Replace the four-line `onChange` with one line:

```jsx
        value={username}   // already there
        onChange={(e) => setUsername(e.target.value)}
      />   // already there
```

Checkpoint: refresh, type `a`, then `b`. **Hello JPa**, then **Hello JPab**, and the box says `JPab`.

Same box, same event. The only difference is that you told React about the change through `setUsername`, so React drew it again.

### 8. Watch the whole function run again

First line inside `HelloUser`:

```jsx
const HelloUser = (props) => {   // already there
  console.log('HelloUser ran');
```

Checkpoint: refresh. `HelloUser ran` once. Type `a`, then `b`. Two more. One per keystroke.

Every time state changes, React calls your whole function again from the top. That's why the plain variable in step 5 kept resetting, and `useState` is what survives from one run to the next. `const` is fine for `username` because it never changes during one run. Each run gets a fresh `const` with the new value.

If you rendered two `HelloUser`s back in step 3, typing in one changes only that one, and only that one logs. Each copy has its own state.

### What's a prop and what's state

- **Props** are handed to you by whoever renders you. You read them. You don't change them.
- **State** is yours. You change it only through its setter, and the setter is what makes React redraw.

A prop can hold anything, including a function. DomoMaker D's `App` keeps a true/false in state and passes `DomoForm` a function that flips it. That's this page, with the setter handed to a child.
