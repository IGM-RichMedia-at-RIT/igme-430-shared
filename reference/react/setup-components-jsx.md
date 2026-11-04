# React: setup, components, and JSX

Every page before this one was built on the server. [Handlebars](../express/handlebars-templates.md) filled in the cat names before the page left. React flips that. The server sends an almost empty page and a JavaScript bundle, and the browser builds the page.

Three things to hold onto. Babel turns JSX (the HTML-looking code) into plain JavaScript, and webpack bundles it. A component is a function that returns JSX. And View Source shows an empty `div`, because the page you see was drawn after it loaded.

This is not a React class. Three days of it, enough to read React and to do DomoMaker D and Project 2, which both require it.

## Quick example

From `client/example1.jsx` in [React-Functional-Components-Done](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components-Done/blob/master/client/example1.jsx), with the comments cut:

```jsx
const React = require('react');
const { createRoot } = require('react-dom/client');

const HelloWorld = () => {
  return (
    <div>
      Hello World!
    </div>
  );
};

const init = () => {
  const root = createRoot(document.getElementById('app'));
  root.render(<HelloWorld />);
};

window.onload = init;
```

The page it draws into is `views/example1.handlebars`. The part that matters is a script tag and an empty `div` with an id:

```html
<script src="/assets/example1bundle.js"></script>
...
<div id="app">

</div>
```

webpack builds `client/example1.jsx` into `hosted/example1bundle.js`. Two terminals, both left running:

```
npm run watchBundle
npm start
```

Save the `.jsx`, wait for the first terminal to say `compiled successfully`, refresh the browser.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 10A (Mon 10/26) | What's installed and how webpack uses Babel, Hello World, View Source vs the Elements tab, what Babel did to the JSX, the JSX rules by breaking them, components inside components | Starter: [React-Functional-Components](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components) · Done: [React-Functional-Components-Done](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components-Done) |

Where to look in the done repo:

- `client/example1.jsx`: the whole Hello World. The done repo stops there. The second component and `className` from class aren't in it
- `webpack.config.js`: three entries (one per page), the babel-loader rule under `module.rules`, and `[name]bundle.js` as the output name
- `.babelrc`: one preset, `@babel/preset-react`. It's a hidden file
- `package.json`: the `watchBundle`, `buildBundle`, and `heroku-postbuild` scripts, and the five React-related devDependencies
- `views/example1.handlebars`: the script tag and the `#app` div
- `hosted/example1bundle.js`: the built output. Search it for `createElement` to see what Babel made of the JSX

Videos: [Austin, Week 10.1: React FSCs](https://www.youtube.com/watch?v=kAMb0sEp9js) is this page. It's older than our code: it renders with `ReactDOM.render`, which React 19 removed. The idea is the same. [Austin, Week 10.2: React Class Components](https://www.youtube.com/watch?v=EzgxSVN-AzI) is the older way to write a component. Worth watching so you can read it when you run into it. Don't write it.

Used in: DomoMaker D (steps 3 to 5 are this setup, and you install the packages, add the webpack rule, and make `.babelrc` yourself), Project 2 (at least five dynamic React components).

## Common mistakes

- **The page is blank and the webpack terminal says `ERROR`.** The build failed. webpack still writes a bundle, and the bundle throws (`Uncaught Error: Module build failed` in the console). Fix the line it names and save. When the page is blank, look at the webpack terminal first.
- **The page is blank and there are no errors anywhere.** `root.render(...)` or `window.onload = init;` is missing. Both fail silently.
- **You saved and nothing changed.** You didn't refresh, or `watchBundle` isn't running. There's no auto-refresh, and the server doesn't need a restart.
- **`Module parse failed: Unexpected token ... You may need an appropriate loader`.** No babel-loader rule in `webpack.config.js`. This is DomoMaker D step 5. After editing the config, restart `watchBundle`.
- **`Support for the experimental syntax 'jsx' isn't currently enabled`.** No `.babelrc`. It's a hidden file, so copying a project folder in Finder or Explorer can drop it. Copy it from the starter, or clone instead of copying, then restart `watchBundle`.
- **`Module not found: Error: Can't resolve 'react'`.** `npm install` hasn't been run.
- **Blank page, console says `Uncaught ReferenceError: React is not defined`.** The `const React = require('react');` line got deleted. It looks unused. It isn't: every JSX tag becomes a `React.createElement` call.
- **`Adjacent JSX elements must be wrapped in an enclosing tag`.** A component returned two top-level elements. Wrap them in one `<div>` or a fragment, `<>...</>`.
- **The component doesn't show, and the console says the tag is unrecognized or has incorrect casing.** The component name starts with a lowercase letter. Lowercase means HTML tag to React.
- **Console: `Invalid DOM property 'class'. Did you mean 'className'?`** JSX uses JavaScript's names. `className`, `htmlFor`, `onChange`.
- **`EADDRINUSE :::3000`.** Last week's server is still running. `lsof -ti tcp:3000 | xargs kill -9`

## What you'll find on Google

- **`npx create-react-app`, Vite, Next.js.** Scaffolds that generate a whole project with their own dev server. We bolt React onto an Express server we already have, so the server keeps serving our Handlebars pages and our API. Create React App is also deprecated now.
- **`import React from 'react'` and `export default`.** ES module syntax. webpack handles both, and we use `require` to match the starter and DomoMaker.
- **`ReactDOM.render(<App />, document.getElementById('app'))`.** The pre-React-18 way, removed in React 19, which is what `npm install` gives you today. `createRoot` then `root.render` is the current way.
- **`class HelloWorld extends React.Component` with a `render()` method.** Class components, the older style. Still runs on React 19, and plenty of tutorials and older code use it. Read it, don't write it. Hooks (the next two pages) replaced it.
- **"You don't need to import React anymore."** True with a newer Babel setting (the automatic JSX runtime). Our `.babelrc` uses the classic one, so the `require('react')` line stays.
- **`.js` instead of `.jsx`.** The webpack rule takes either. `.jsx` is what tells VS Code to highlight JSX.
- **`<script type="text/babel">` with React from a CDN.** Babel in the browser, compiling on every page load. Fine for a demo, too slow for anything real. We build ahead of time.
- **Server-side rendering with React (Next.js, Remix).** React can run on the server too. Not here. Our server sends files out of `hosted/`, and JSX only ever runs in the browser.

## Redo it

Start from a fresh clone of the [starter](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components) and run `npm install`. All three `client/*.jsx` files are empty. Everything else is set up. Open two terminals in the repo folder.

### 1. Look at what's already there

Three files, no typing:

- `package.json`, under `devDependencies`: `react`, `react-dom`, `@babel/core`, `@babel/preset-react`, `babel-loader`. They're devDependencies because webpack copies React into the bundle. The server never runs React
- `webpack.config.js`: three entries, and the rule that says any `.js` or `.jsx` file goes through babel-loader first. `.babelrc` next to it tells Babel it's React
- `views/example1.handlebars`: `<script src="/assets/example1bundle.js">` and the empty `<div id="app">`. Our code finds that div and draws into it

### 2. Pull in React

In `client/example1.jsx`:

```jsx
const React = require('react');
const { createRoot } = require('react-dom/client');
```

`react` makes components. `react-dom` puts them on the page. `createRoot` picks the element React draws into.

### 3. The component

Under the requires:

```jsx
const HelloWorld = () => {
  return (
    <div>
      Hello World!
    </div>
  );
};
```

A component is a function that returns JSX. That's the whole definition. The capital H is on purpose (step 12).

### 4. Draw it on the page

Under the component:

```jsx
const init = () => {
  const root = createRoot(document.getElementById('app'));
  root.render(<HelloWorld />);
};

window.onload = init;
```

You use a component like an HTML tag. `window.onload` waits until the page, and `#app`, exists.

### 5. Build and run

Terminal 1: `npm run watchBundle`. It says `compiled successfully` and keeps running. Terminal 2: `npm start`.

Checkpoint: `localhost:3000` says Hello World! Terminal 1 rebuilds every time you save. You still refresh the browser yourself.

### 6. View Source, then Elements

Right-click the page and pick View Page Source.

Checkpoint: `<div id="app">` with nothing in it. There is no Hello World on this page at all. Now open DevTools, Elements tab, and expand `#app`: there's the div with Hello World in it.

View Source is what the server sent. Elements is what's on screen now. React wrote that div in the browser, after the page loaded. Last week's Handlebars pages show the cat names in View Source, because the server filled them in. That's server-side rendering. This is client-side. One consequence: search engines mostly read what the server sent, so a Handlebars page gets indexed and a React page can hide its data. Sometimes that's what you want.

### 7. Find the JSX in the bundle

Open `hosted/example1bundle.js` and search for `createElement`.

Checkpoint: two hits, something like `React.createElement("div", null, "Hello World!")` and `React.createElement(HelloWorld, null)`. That's your JSX after Babel. `<div>` became a function call. JSX is a nicer way to write this, and it's why line 1 needs `React` even though you never type it. The file is over a megabyte because all of React is in it.

### 8. Two elements, no parent (wrong on purpose)

Replace the whole `return ( ... );` in `HelloWorld`:

```jsx
  return (
    <h1>Hello World!</h1>
    <p>2 + 2 = {2 + 2}</p>
  );
```

Checkpoint: Terminal 1 says `Adjacent JSX elements must be wrapped in an enclosing tag. Did you want a JSX fragment <>...</>?` Refresh, and the page is blank. Build failed, blank page. When the page is blank, look at Terminal 1 first.

### 9. Wrap it in one parent

```jsx
  return (
    <div>
      <h1>Hello World!</h1>   // already there
      <p>2 + 2 = {2 + 2}</p>   // already there
    </div>
  );
```

Checkpoint: Hello World! and 2 + 2 = 4. A component returns exactly one element. A fragment, `<>` and `</>`, works too and adds no extra div. The curly braces mean "run this JavaScript and show the result." Like Handlebars' `{{ }}`, but one brace, and any expression works.

### 10. `class` vs `className`

Change the outer `<div>` to `<div class="greeting">`.

Checkpoint: the page looks fine, and the console says `Invalid DOM property 'class'. Did you mean 'className'?` Change it to `className="greeting"` and the warning goes away. JSX uses JavaScript's names: `className` not `class`, `htmlFor` not `for`, `onChange` not `onchange`. Every label in DomoMaker D's forms uses `htmlFor`.

### 11. A component inside a component

Above `HelloWorld`:

```jsx
const Credits = () => {
  return <p>Made in IGME 430</p>;
};
```

One line of JSX needs no parentheses. Then use it inside `HelloWorld`'s div, under the `<p>`:

```jsx
      <p>2 + 2 = {2 + 2}</p>   // already there
      <Credits />
    </div>   // already there
```

Checkpoint: Made in IGME 430 shows under the math. A whole React app is one component at the top with others inside it. DomoMaker D is an `App` with a `DomoForm` and a `DomoList` inside it.

### 12. Lowercase breaks it

Rename `Credits` to `credits` in both places and refresh.

Checkpoint: the line disappears, and the console says `The tag <credits> is unrecognized in this browser. If you meant to render a React component, start its name with an uppercase letter.` Lowercase means HTML tag, so React went looking for an HTML element called `credits`. Change it back.

### What comes next

A component that takes data in and remembers things: [props and `useState`](props-and-usestate.md). Then loading data from your server into a component: [`useEffect` and loading data](useeffect-and-loading-data.md). For the webpack side on its own, see [nodemon and webpack](../tooling/nodemon-webpack.md).
