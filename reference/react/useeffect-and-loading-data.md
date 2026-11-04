# React: useEffect and loading data from the server

[The props and state page](props-and-usestate.md) ends on this: every time state changes, React runs your whole component function again. On this page that bites. We load a list of songs from the server with `fetch`, the page looks fine, and the Network tab shows thousands of requests. `useEffect` is how you tell React "run this for me, and only when I say." It's also the exact shape of DomoMaker D's `DomoList`.

## Quick example

From `client/example3.jsx` in [React-Functional-Components-Done](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components-Done/blob/master/client/example3.jsx), with the comments trimmed:

```jsx
const React = require('react');
const { useState, useEffect } = React;
const { createRoot } = require('react-dom/client');

const SongContainer = (props) => {
  const [songs, setSongs] = useState(props.songs);

  useEffect(() => {
    const loadSongsFromServer = async () => {
      const response = await fetch('/getSongs');
      const data = await response.json();
      setSongs(data);
    };
    loadSongsFromServer();
  }, []);

  if (songs.length === 0) {
    return (
      <div>
        <h3>No Songs Yet!</h3>
      </div>
    );
  }

  const songList = songs.map((song) => {
    return (
      <div key={song.title}>
        <h2>{song.artist} - <i>{song.title}</i></h2>
      </div>
    );
  });

  return (
    <div>
      <h1>My favorite songs!</h1>
      {songList}
    </div>
  );
};

const init = () => {
  const root = createRoot(document.getElementById('app'));
  root.render(<SongContainer songs={[]} />);
};

window.onload = init;
```

Four pieces:

- **`useEffect(fn, [])`**: "React, run this function for me after you draw." The function is the effect, meaning anything that isn't drawing: fetching data, timers. The `[]` at the end is when to run it again. Empty means only the first time.
- **The `async` function sits inside the effect** and gets called right away. React wants the effect itself to be a plain function.
- **The early `return`**: a component can return different JSX depending on its data. No songs yet, show a message. Otherwise keep going.
- **`.map` turns an array of data into an array of JSX**, and each item gets a `key` so React can tell them apart when it redraws.

In class we also added a reload button and changed `[]` to `[reload]`. That version is in the Redo section, and it's the pattern DomoMaker D uses.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 10C (Fri 10/30) | `SongContainer`, the empty state, `fetch` inside a component, the infinite loop, `useEffect` with `[]`, a reload button with `[reload]`, what the second argument decides, `.map` and `key` | Starter: [React-Functional-Components](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components) · Done: [React-Functional-Components-Done](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components-Done) |

Where to look in the done repo:

- `client/example3.jsx`: `SongContainer` and `init`
- `server/controllers/index.js`: `getSongs`, which sends a hardcoded array with `res.json`. This is the server side of the `fetch`. See [routes and controllers](../express/router-controllers-404.md) for how it's wired up.
- `server/router.js`: `/example3` is the page, `/getSongs` is the data. The page loads once, then asks for small JSON like this.

Video: [Austin, Week 10.1: React FSCs](https://www.youtube.com/watch?v=kAMb0sEp9js) (43:47). It's older than our code: it re-renders from outside with `ReactDOM.render`, which React 19 removed, and it doesn't cover `useEffect`. The `.map` explanation and the YouTube gray-boxes example are worth it.

Used in: DomoMaker D (`getDomos` sends JSON, `DomoList` loads it in `useEffect` with `[props.reloadDomos]` and draws it with `.map` and `key`), Project 2 (any component that shows data from your server).

## Common mistakes

- **Thousands of requests in the Network tab, and the page looks completely normal.** You called the loader directly in the component body. The fetch sets state, state change re-runs the function, the function fetches again. React 19 gives no warning for this. The Network tab (filter: Fetch/XHR) is the only tell. Wrap the loader in `useEffect` with a dependency array.
- **Still looping after you added `useEffect`.** Either the old tab is still open (refresh it), or you left off the second argument. `useEffect(fn)` with no array runs after every render, which is the same meltdown.
- **Looping again after you added `[songs]`.** This is the one you'll actually make. The effect is about songs, so `songs` goes in the array, right? Trace it: the effect calls `setSongs`, `songs` changes, React sees a change in the array, runs the effect again. You rebuilt the loop with the thing meant to prevent it. It loops even though the three songs never change, because `response.json()` makes a new array every time and React compares whether it's the same array, not the same contents. In DomoMaker D, `domos` sits right there looking like it belongs in the array next to `props.reloadDomos`. It doesn't.
- **Stuck on "No Songs Yet!"** The loader is written but never called. The call goes inside the effect, right after the function.
- **Console says `useEffect must not return anything besides a function, which is used for clean-up. It looks like you wrote useEffect(async () => ...)`.** You made the effect itself `async`. The songs still load, but nest the `async` function inside a plain effect and call it, like the quick example.
- **Blank page, `ReferenceError: useEffect is not defined`.** Line 2 still says `const { useState } = React;`. Add `useEffect` to it.
- **`SyntaxError: Unexpected token '<', "<!DOCTYPE "... is not valid JSON`.** Typo in the fetch URL. The server sent its 404 page, which starts with `<!DOCTYPE`, and `response.json()` choked on it. Check the URL against the route in `router.js`.
- **The reload button does nothing.** The array is still `[]`. A click re-runs the component, but the effect only re-runs when something in the array changes.
- **Console warning `Each child in a list should have a unique "key" prop`.** Missing or undefined key on the `.map` output. The page works, and it keeps working until your list reorders, and then you get a bug that looks unrelated. Use a field from the data. DomoMaker D's done code has `key={domo.id}`, and that's `undefined`: Mongo's field is `_id`. Use `key={domo._id}`.
- **Your laptop fan spins up.** A looping tab is open somewhere. Close it.

## What you'll find on Google

- **`componentDidMount` and `this.setState`.** The class-component version of this page. `componentDidMount` is roughly `useEffect` with `[]`. You'll see it in older tutorials and answers. Don't write new code that way.
- **`import { useState, useEffect } from 'react'`.** Same thing as our `require` lines. Webpack handles both.
- **`axios.get(...)` instead of `fetch`.** Same request, with a library. `fetch` is built into the browser, and it's what DomoMaker uses.
- **`.then()` chains instead of `async` / `await`.** `fetch('/getSongs').then((r) => r.json()).then(setSongs)` does the same job. Either reads fine. Pick one per project.
- **React Query, SWR, or a framework like Next.js for data loading.** Bigger React apps use these so they don't write this effect by hand in every component. For DomoMaker and Project 2, the hand-written version is the pattern.
- **A custom `useFetch` hook.** A function that bundles the state and the effect from this page so you can write `const songs = useFetch('/getSongs')`. Reasonable once you have five components doing the same thing. Write it yourself after you understand this page, not before.
- **`key={index}`.** Silences the warning and fixes nothing. The index changes the moment the list reorders, which is the exact case the key exists for. Use a field from the data.
- **A cleanup function returned from the effect.** React runs it before the next run of the effect, and when the component goes away. Timers and subscriptions need it. A one-shot fetch doesn't.
- **"Why not just call the loader once at the top of the file, outside the component?"** It has to call `setSongs`, which only exists inside the component.

## Redo it

Start in your clone of the [starter](https://github.com/IGM-RichMedia-at-RIT/React-Functional-Components) with the setup from [the setup page](setup-components-jsx.md) working: Terminal 1 running `npm run watchBundle`, Terminal 2 running `npm start`, the browser on `localhost:3000/example3` with DevTools open on the Network tab, filtered to Fetch/XHR. `client/example3.jsx` is empty in the starter, and it's the only file you type in. After every save, wait for Terminal 1 to say `compiled successfully`, then refresh.

First, look at the data raw. Open `localhost:3000/getSongs` in another tab:

```
[{"artist":"Rick Astley","title":"Never Gonna Give You Up"},{"artist":"Washed Out","title":"Feel It All Around"},{"artist":"Men I Trust","title":"Show Me How"}]
```

That route is already in the starter. It's not a page, it's data. The browser gets your React code once, then asks for small JSON like this, and React draws it.

### 1. The top of the file

```jsx
const React = require('react');
const { useState } = React;
const { createRoot } = require('react-dom/client');
```

### 2. The component, holding songs in state, and render it

Under the requires:

```jsx
const SongContainer = (props) => {
  const [songs, setSongs] = useState(props.songs);

  return (
    <div>
      <h1>My favorite songs!</h1>
    </div>
  );
};
```

Bottom of the file:

```jsx
const init = () => {
  const root = createRoot(document.getElementById('app'));
  root.render(<SongContainer songs={[]} />);
};

window.onload = init;
```

`songs={[]}`: the outer braces mean "JavaScript here," the `[]` inside is an empty array. Start with no songs.

Checkpoint: refresh. **My favorite songs!**

### 3. Say so when there's nothing yet

Right under the `useState` line, before the `return`:

```jsx
  const [songs, setSongs] = useState(props.songs);   // already there

  if (songs.length === 0) {
    return (
      <div>
        <h3>No Songs Yet!</h3>
      </div>
    );
  }
```

Checkpoint: refresh. **No Songs Yet!** This is YouTube's gray boxes: show something right away, fill it in when the data shows up.

### 4. A function that asks the server

Under the `useState` line, above the `if`:

```jsx
  const loadSongsFromServer = async () => {
    console.log('Loading songs');
    const response = await fetch('/getSongs');
    const data = await response.json();
    setSongs(data);
  };
```

`fetch('/getSongs')` asks your own server for that URL, from the browser. `response.json()` turns the JSON text into a real array, and it also needs an `await`. `setSongs(data)` puts the array in state, which makes React redraw. The `console.log` is there so you can count runs in the browser console.

Checkpoint: refresh. Still **No Songs Yet!**, console empty. You wrote it; nobody called it.

### 5. Call it (the wrong way, on purpose)

Right under the function's closing `};`:

```jsx
  };   // already there
  loadSongsFromServer();
```

Checkpoint: refresh. **My favorite songs!** with no list under it. The empty message is gone, so songs arrived.

### 6. Turn the songs into JSX

After the `if` block's closing `}`, before the main `return`:

```jsx
  const songList = songs.map((song) => {
    return (
      <div key={song.title}>
        <h2>{song.artist} - <i>{song.title}</i></h2>
      </div>
    );
  });
```

`.map` is like `forEach`, but it hands back a new array with one result per item. `[1, 2, 3].map((n) => n * 10)` gives `[10, 20, 30]`. Here: songs in, JSX out. `key={song.title}` is a unique label per item so React can tell list items apart when it redraws (step 11).

Then show the list. In the main `return`, under the `<h1>`:

```jsx
      <h1>My favorite songs!</h1>   // already there
      {songList}
```

Checkpoint: refresh. All three songs. It looks done. If you stopped here, you'd turn it in.

### 7. Look at the Network tab

`getSongs`, `getSongs`, `getSongs`, and the request count at the bottom climbs into the thousands. The console shows `Loading songs` over and over (Chrome stacks repeats with a count bubble).

Here's the circle: the component runs, which calls `loadSongsFromServer`, which calls `setSongs`, which changes state, which runs the component again, which calls the loader again. Forever. You put the fetch in a function React keeps re-running.

### 8. Hand the loading to React

Line 2 only:

```jsx
const { useState, useEffect } = React;
```

Then wrap step 4's function and step 5's call. Type the first and last lines, indent the middle:

```jsx
  useEffect(() => {
    const loadSongsFromServer = async () => {   // already there
      ...                                        // already there
    };                                           // already there
    loadSongsFromServer();                       // already there
  }, []);
```

Checkpoint: **refresh first.** The old page keeps looping until you do. Then: the same three songs, one `Loading songs` in the console, one `getSongs` in the Network tab.

You don't want your component to call this function. You want to tell React "here's a function, call it for me when the time is right." Empty array means the first time only.

Optional: to see the empty state for real, change `loadSongsFromServer();` inside the effect to `setTimeout(loadSongsFromServer, 2000);`. Refresh: **No Songs Yet!** for two seconds, then the list. Change it back.

### 9. A reload button

"Load once and stop" only works because this page never changes. The moment somebody adds a song, the fix you know is "refresh the page," which is what React was supposed to end. So: what if you want the effect to run again, on purpose? You put something in the array.

Under the `songs` line:

```jsx
  const [songs, setSongs] = useState(props.songs);   // already there
  const [reload, setReload] = useState(false);
```

In the main `return`, under the `<h1>`:

```jsx
      <h1>My favorite songs!</h1>   // already there
      <button onClick={() => setReload(!reload)}>Reload songs</button>
```

`reload` is a true/false you flip. The value doesn't matter. The change does.

Checkpoint: refresh, click **Reload songs** three times. Still one `Loading songs`. The component re-ran three times (a state change always does that), but the effect didn't, because its array is empty.

### 10. Watch `reload`

The effect's last line:

```jsx
  }, [reload]);
```

Checkpoint: refresh, click three times. One `Loading songs` on load, plus one per click. Four requests in the Network tab.

This is DomoMaker D, split across three components: `App` holds a true/false called `reloadDomos`, the form gets a `triggerReload` function that flips it, and `DomoList`'s effect watches `[props.reloadDomos]`. Add a domo, the list reloads.

What the second argument decides:

| What you wrote | The effect runs |
|---|---|
| no `useEffect` at all | every render, forever |
| `useEffect(fn)`, no array | after every render, forever |
| `useEffect(fn, [])` | once |
| `useEffect(fn, [reload])` | once, then whenever `reload` changes |

No array is the same meltdown, which is the surprise. The wrapper isn't the fix. The array is. `[]` is an empty checklist: before each re-run React looks down it asking "did any of these change?" Nothing on the list, so the answer can never be yes. `[reload]` is a checklist with one line. Still `false`, skip. Flipped by a click, run again.

And the trap: don't put `songs` in that array. See Common mistakes.

### 11. Break the key on purpose

In the `.map`, change `<div key={song.title}>` to `<div>`. Refresh. The list still shows, and the console warns:

```
Each child in a list should have a unique "key" prop.
```

Put `key={song.title}` back.

On redraw, React lines the old items up against the new ones to see what changed, and position is a bad answer: insert one song at the top and every row below looks changed. A key is a name that travels with the item, so React can tell "same song, moved" from "new song." Use a field from the data. For anything out of MongoDB that's `_id`, which is where [the queries page](../database/mongoose-queries.md) leaves your data.

### Checking the response

The quick example trusts the server. For your own routes, check `response.status` before calling `response.json()`, the same way the client code on [the status codes page](../http/status-codes.md) does. A 404 or 500 from your server comes back as HTML, and `response.json()` throws on it.
