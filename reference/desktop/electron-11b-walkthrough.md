# Electron: the 11B class walkthrough

> **A heads-up first.** This is the guide I taught from on Wednesday, Nov 4 (11B). I wrote it for myself to teach from, so it's rougher than the other pages in this reference and doesn't follow their format. There's no video for this day, so I'm sharing it in case it helps you redo the demo or catch a step you missed.

**Where we did this:** Week 11, Wednesday (11B)

- Starter: [Basic-Electron](https://github.com/IGM-RichMedia-at-RIT/Basic-Electron)
- Done: [Basic-Electron-Done](https://github.com/IGM-RichMedia-at-RIT/Basic-Electron-Done) (it differs from this page in a few small ways, listed [at the bottom](#how-basic-electron-done-differs))
- Video: none for this day

---

## How to use this page

- You type in five files: `electron/main.js`, `electron/preload.js`, `hosted/homePage.html` (new), `client/homePage.jsx`, and `package.json`.
- In the code blocks, `// already there` marks a line you already have, so you can find where the new code goes.
- **To restart the app:** `Ctrl+C` in the terminal, then `npm start`. There's no nodemon here, so you'll do this after almost every step. On a Mac, `Cmd+Q` on the app works too.
- **Where logs show up:** `main.js` prints in the terminal. The page and `preload.js` print in the window's own DevTools: View → Toggle Developer Tools (`Ctrl+Shift+I` on Windows, `Cmd+Opt+I` on a Mac).
- Windows and Mac look a little different in a few spots (Steps 6, 7, 20, and 22). Those steps say so.

**What runs when:**

| You do this | What runs |
|---|---|
| `npm start` | `webpack` (after Step 16), then `electron .` → `package.json` `"main"` → `electron/main.js` |
| `main.js` makes the window | `new BrowserWindow(...)`, then `loadFile` opens `hosted/homePage.html` |
| The page starts loading | `preload.js` runs first and fills in the version numbers |
| The page finishes loading | `homePageBundle.js` (built from `client/homePage.jsx`) → `init` → React draws |
| You close the window on a Mac | The app keeps running (Step 7). `Ctrl+C` or `Cmd+Q` to quit |

---

## Part 1: Tour the starter

### 1. Same code, different box

Electron is a copy of Chrome and a copy of Node, packed into a desktop app. You write the page the way you always have. VS Code, Discord, Slack, and Obsidian are all Electron apps.

```
main.js (Node)  →  creates a window (Chrome)  →  shows homePage.html
```

One thing to hold onto: even though there's Node inside, **this is not a server.** If your app needs data, your server is a separate project, like your Heroku app, and the desktop app calls it like any web page would.

### 2. `package.json`

- `"main": "electron/main.js"` is the file `electron .` runs. Same idea as `app.js` in our servers.
- `"description"` and `"author"` should be filled in. The packager reads them when it builds the app in Part 7.
- `"start": ""` is empty. Step 6 fills it in.
- `@electron/packager` is already in the devDependencies, for Part 7.

Electron itself is just an npm package, and the starter doesn't include it yet.

**2b. Install.** In the terminal:

```
npm install
npm install --save-dev electron
```

Don't skip the second line, or Step 6 fails. The first time Electron runs on a machine it downloads about 129 MB, so your first `npm start` takes a minute.

### 3. `webpack.config.js` and `.babelrc`

The entry is `client/homePage.jsx` and the output is `hosted/homePageBundle.js`. `.babelrc` only has the React preset.

There's no `preset-env` because Electron's Chrome is brand new, so there's no old browser to translate for. Babel only turns JSX into JavaScript.

`electron/main.js` and `client/homePage.jsx` start out empty.

---

## Part 2: A window

### 4. Make a window

**File:** `electron/main.js` (empty)

```js
const { app, BrowserWindow } = require('electron');

const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
  });
};
```

`app` is the whole program. `BrowserWindow` is one window. One app can have several.

### 5. Make it when the app is ready

**Where:** bottom of the file.

```js
app.whenReady().then(() => {
  createWindow();
});
```

`whenReady` is Electron's version of `window.onload`.

### 6. A start script

**File:** `package.json`

Change `"start": ""` to:

```json
    "start": "electron .",
```

**6b.** Run `npm start`. You should get an empty white 800×600 window titled **electrondemo**, plus Electron's default menu (File, Edit, View, Window). Where the menu shows up depends on your computer:

- **Windows:** inside the window, right under the title bar.
- **Mac:** in the strip at the very top of the screen, after the app name: electrondemo, File, Edit, View, Window. Nothing inside the window.

It's a browser window with no page in it yet.

Don't bother opening DevTools at this point. With no page loaded there's nothing for it to show. It starts working at Step 9.

### 7. Quitting, the Mac way

**File:** `electron/main.js`

**7a. Where:** inside `whenReady`, under `createWindow();`.

```js
  createWindow();   // already there

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow();
    }
  });
```

**7b. Where:** bottom of the file.

```js
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});
```

`darwin` is macOS. On a Mac, apps keep running after you close their last window (the dot stays in the dock), so we only quit on Windows and Linux. `activate` is clicking the dock icon: if there are no windows, make one.

**7c.** Restart, then close the window.

- **Windows:** the app quits and the terminal comes back.
- **Mac:** the terminal is still running. That's the line you just wrote. `Cmd+Q` or `Ctrl+C` to quit.

---

## Part 3: A page, locked down

### 8. The page

**New file:** `hosted/homePage.html` (make the `hosted` folder first)

**8a.**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Hello Electron!</title>
  <meta charset="UTF-8">
  <meta http-equiv="Content-Security-Policy" content="default-src 'self'">
</head>
```

The Content-Security-Policy (CSP) line says "this page may only load things from itself." Electron apps can read and delete files on your computer, so this goes in from the start. Leave it out and Electron puts a security warning in the console.

**8b. Where:** under `</head>`.

```html
<body>
  <section id="content"></section>

  <div>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.
  </div>
</body>
</html>
```

### 9. Load it

**File:** `electron/main.js`

**Where:** the last line inside `createWindow`. Here's the whole function after this step (one new line):

```js
const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
  });

  win.loadFile('./hosted/homePage.html');   // ← NEW
};
```

**9b.** Restart. The title is now **Hello Electron!** and the page says *We are using Node.js , Chromium , and Electron .* with blanks. Those get filled in next.

No server is involved. `loadFile` opens the file straight off the disk.

**9c.** Now open View → Toggle Developer Tools. It's Chrome's DevTools, same as in the browser, and the Elements tab shows your HTML. If it's cramped, drag the window wider. You'll need DevTools in Part 6.

---

## Part 4: Preload, the bridge

### 10. A script that runs before the page

**New file:** `electron/preload.js`

**10a.**

```js
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector);
    if (element) { element.innerHTML = text; }
  };
});
```

**10b. Where:** inside the listener, under `replaceText`'s closing `};`.

```js
  };   // already there

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency]);
  }
```

`process.versions` is a Node object listing version numbers. A normal web page can't see `process`. The preload script can see both the page and Node, which makes it a good place for anything that should be ready before the page shows.

### 11. Hook it up

**File:** `electron/main.js`

**Where:** in the `BrowserWindow` options, under `height: 600,`.

```js
    height: 600,   // already there
    webPreferences: {
      preload: `${__dirname}/preload.js`,
    },
```

The options split into two groups. `width` and `height` are about the window. `webPreferences` is about the web page inside it, and `preload` is a page setting.

Put `preload` **inside** `webPreferences`, not next to `width`. At the top level Electron silently ignores it and the versions stay blank, with no error.

**11b.** Restart. The version line fills in: Node.js 24, Chromium 156, and Electron 45 (the exact numbers after each dot depend on when you installed). That's Node and Chrome shipped inside the app, so it doesn't matter which browser the user has.

---

## Part 5: React, bundled in

### 12. A component

**File:** `client/homePage.jsx` (empty)

**12a.**

```js
const React = require('react');
const { createRoot } = require('react-dom/client');

const HelloUser = (props) => {
  return (
    <div>
      <h1>Hello {props.username}!</h1>
      <p>Welcome to Electron!</p>
    </div>
  );
};
```

**12b. Where:** bottom of the file. Same `init` as our Week 10 React demos.

```js
const init = () => {
  const root = createRoot(document.getElementById('content'));
  root.render(<HelloUser username="JP" />);
};

window.onload = init;
```

### 13. Build it

Run `npm run webpack`. It fails on purpose: two `ERROR in ./client/homePage.jsx` blocks, `Can't resolve 'react'` and `Can't resolve 'react-dom/client'`. No bundle gets written.

React isn't installed. We could load it from a CDN with a script tag, but then the desktop app only works when you're online. So we bundle it in.

### 14. Install React, build again

```
npm install --save-dev react react-dom
npm run webpack
```

This time it says `compiled successfully`, and `hosted/homePageBundle.js` appears. It's about 220 KB because React is inside it now.

`--save-dev` is right here: webpack copies React into the bundle, so the finished app never needs `node_modules`.

### 15. Load the bundle

**File:** `hosted/homePage.html`

**Where:** under the versions `</div>`, above `</body>`.

```html
  </div>   <!-- already there -->

  <script src="homePageBundle.js"></script>
```

**15b.** Restart. **Hello JP!** and *Welcome to Electron!* show up above the version line.

### 16. Build every time we start

**File:** `package.json`

Change the start script to:

```json
    "start": "webpack && electron .",
```

Build, then launch. Without this you'd keep forgetting to rebuild and wonder why nothing changed.

At this point you have a desktop window running React. Part 6 makes it talk to a server, and Part 7 makes it a real app.

---

## Part 6: Talk to a server

The name "JP" is typed into the code. A real app gets its data from a server, the way your Project 2 front end calls your Heroku app. So we'll ask a server for the name. It goes in three moves:

1. **Step 17** puts the name in React state, so it can change after the page loads.
2. **Step 18** fetches a name from a real server. It fails on purpose: the CSP from Step 8 only allows `'self'`, so Electron blocks the request, and DevTools shows you why.
3. **Step 19** adds that one server to the CSP, and it works.

This is the "not a server" idea from Step 1. The app doesn't hold the data, it calls a server for it, and out of the box it isn't allowed to call anyone. Every server it talks to, you allow by name.

### 17. Give the name some state

**File:** `client/homePage.jsx`

**17a. Where:** line 2, under the React require.

```js
const React = require('react');   // already there
const { useState, useEffect } = React;
```

**17b. Where:** first line inside `HelloUser`.

```js
const HelloUser = (props) => {   // already there
  const [username, setUsername] = useState(props.username);
```

**17c.** Change the `<h1>` to use the state:

```js
      <h1>Hello {username}!</h1>   // was {props.username}
```

### 18. Ask a server for a name

**Where:** under the `useState` line. Same pattern as Week 10: the async function lives inside the effect, because `useEffect` can't take an async function directly.

```js
  useEffect(() => {
    const loadName = async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/users/1');
      const data = await response.json();
      setUsername(data.name);
    };
    loadName();
  }, []);
```

jsonplaceholder.typicode.com is a free fake-data API. Pretend it's your Project 2 server on Heroku.

**18b.** Restart. It still says **Hello JP!** Nothing changed. Open DevTools, then the Console:

```
Connecting to 'https://jsonplaceholder.typicode.com/users/1' violates the following Content Security Policy directive: "default-src 'self'". Note that 'connect-src' was not explicitly set, so 'default-src' is used as a fallback. The action has been blocked.
Uncaught (in promise) TypeError: Failed to fetch
```

That's our own rule from Part 3. This app may only talk to itself.

### 19. Allow that one server

**File:** `hosted/homePage.html`

Change the CSP line's `content` to:

```html
  <meta http-equiv="Content-Security-Policy" content="default-src 'self' https://jsonplaceholder.typicode.com">   <!-- was default-src 'self' -->
```

**19b.** Restart. **Hello Leanne Graham!**

**Project 2:** Electron isn't required, and it only counts as above and beyond if you do more than this demo. If you build one, it's a second project that calls your Heroku app, and your Heroku URL goes in the CSP.

---

## Part 7: Lock it down, ship it

### 20. No DevTools, no menu

**File:** `electron/main.js`

**20a. Where:** inside `webPreferences`, under `preload`.

```js
      preload: `${__dirname}/preload.js`,   // already there
      devTools: false,
```

**20b. Where:** inside `createWindow`, between the `});` that closes `new BrowserWindow` and the `loadFile` line.

```js
  });   // already there (closes new BrowserWindow)

  win.removeMenu();   // ← NEW

  win.loadFile('./hosted/homePage.html');   // already there
```

**20c.** Restart. View → Toggle Developer Tools does nothing now.

- **Windows:** the menu bar in the window is gone.
- **Mac:** the menu bar stays. It lives at the top of the screen and belongs to the app, not the window, so `removeMenu()` has nothing to remove.

### 21. A package script

So far the app only runs with `npm start`, which needs Node and a terminal. Packaging turns it into something you could hand to anyone: your files plus a private copy of Electron, in one app.

- **Windows:** a folder with `electrondemo.exe` in it
- **Mac:** `electrondemo.app`

The tool is `@electron/packager`, which is already in the starter. The script runs webpack first, so the app always has your latest React.

**21a. Update the packager first.** In the terminal:

```
npm install --save-dev @electron/packager@latest
```

The version in the starter (18) quietly quits on Node 26: it prints `Packaging app for platform...`, then nothing. No error, no folder. The latest version works. Check `node -v` if you're curious; updating does no harm on older Node either.

**File:** `package.json`

**Where:** in `"scripts"`, at the end. Two changes: add a comma to the end of the `"webpack"` line, then add the new line under it.

```json
    "webpack": "webpack",
    "package": "webpack && npx electron-packager ."
```

Forget the comma and every `npm` command fails with `npm error code EJSONPARSE`.

### 22. Ship it

```
npm run package
```

It prints `Packaging app for platform win32 x64` (or `darwin arm64` on a Mac), then `Wrote new app to: ...`. A new folder appears next to `package.json`:

- **Windows:** `electrondemo-win32-x64`. File Explorer hides the `.exe` ending by default, so look for **electrondemo** with Type **Application**, among a lot of DLL files. Double-click it.
- **Mac:** `electrondemo-darwin-arm64`. Double-click **electrondemo.app**.

No terminal, no `npm`. That's a real app.

It's about 259 MB for roughly 50 lines of our code, because it brings its own copy of Chrome. That's the trade Electron makes.

**To share it, share the whole folder** (zip it). The `.exe` can't run without the files next to it. For ways to get closer to a single file, see [the questions below](#can-i-just-send-someone-the-exe).

**Don't commit that folder.** The starter's `.gitignore` doesn't ignore it, and one file inside is 198 MB. GitHub rejects anything over 100 MB.

---

## If something goes wrong

- **Window opens, page blank, title "electrondemo":** `loadFile` is missing, or the path isn't `./hosted/homePage.html`.
- **Versions stay blank:** `preload` is outside `webPreferences`, or the file isn't `electron/preload.js`.
- **No "Hello":** check the window's DevTools. `Can't resolve 'react'` in the terminal means you skipped Step 14. A misspelled bundle name in the script tag fails with no message at all.
- **`TypeError: ... render is not a function`:** that's `ReactDOM.render`, which React 19 removed. Use `createRoot` like Step 12b.
- **Still "Hello JP!" after Step 19:** the CSP URL is misspelled, or you didn't restart.
- **`electron: command not found` or `'electron' is not recognized`:** you skipped `npm install --save-dev electron` (Step 2b).
- **`npm start` says "Downloading Electron binary...":** first run on that machine. It's about 129 MB, so give it a minute.
- **Terminal won't take commands:** the app is still running. `Ctrl+C`.
- **Windows asks `Terminate batch job (Y/N)?` after `Ctrl+C`:** type `Y` and press Enter. That's normal on Windows.
- **`npm run package` stops after `Packaging app for platform...` with no error and no folder:** do Step 21a, then run it again.
- **`Skipping ... (output dir already exists, use --overwrite to force)`:** delete the old `electrondemo-...` folder and run it again.

## Questions that came up

### Can I just send someone the .exe?

No. The `.exe` needs everything else in that folder, because Electron ships its own copy of Chromium and Node, and those files live next to the `.exe`. Zip the whole folder and send that. There are ways to make it feel like one file:

- **A portable .exe:** [electron-builder](https://www.electron.build/) can build a single "portable" `.exe`. It's one file, but when you run it, it unpacks that same folder to a temp location first, so it's still big.
- **An installer:** electron-builder or [Electron Forge](https://www.electronforge.io/) can build a normal `Setup.exe` (or a `.dmg` on Mac). This is what most Electron apps you've installed do.
- **Don't ship Chromium at all:** frameworks like [Tauri](https://tauri.app/) and [Neutralinojs](https://neutralino.js.org/) use the browser engine already built into the operating system (WebView2 on Windows, WebKit on Mac), so apps are a few MB instead of a few hundred. The trade-offs: Tauri's back end is Rust instead of Node, and your page can look slightly different on each OS because you're no longer bringing your own Chrome.

### Why is it so big?

Every Electron app ships its own Chrome and Node. VS Code and Discord do too.

### Can it read files on my computer?

Yes, through Node's `fs`. That's why the CSP matters. Electron's docs recommend doing file work in `main.js` and passing results to the page (look up "IPC" and "contextBridge").

### Do I need CORS on my server for this?

Not for a server like ours. A plain Node server with no CORS headers answered the app fine.

### Can I put my Project 2 in Electron?

It's not required, and the web version still has to be deployed. As an extra, sure (see the Project 2 note in Step 19).

### Can I hide the Mac menu?

`Menu.setApplicationMenu(null)` removes it. It also removes `Cmd+Q`.

---

## The finished code

`electron/main.js`:

```js
const { app, BrowserWindow } = require('electron');

const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: `${__dirname}/preload.js`,
      devTools: false,
    },
  });

  win.removeMenu();

  win.loadFile('./hosted/homePage.html');
};

app.whenReady().then(() => {
  createWindow();

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow();
    }
  });
});

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});
```

`electron/preload.js`:

```js
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector);
    if (element) { element.innerHTML = text; }
  };

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency]);
  }
});
```

`hosted/homePage.html`:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Hello Electron!</title>
  <meta charset="UTF-8">
  <meta http-equiv="Content-Security-Policy" content="default-src 'self' https://jsonplaceholder.typicode.com">
</head>
<body>
  <section id="content"></section>

  <div>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.
  </div>

  <script src="homePageBundle.js"></script>
</body>
</html>
```

`client/homePage.jsx`:

```js
const React = require('react');
const { useState, useEffect } = React;
const { createRoot } = require('react-dom/client');

const HelloUser = (props) => {
  const [username, setUsername] = useState(props.username);

  useEffect(() => {
    const loadName = async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/users/1');
      const data = await response.json();
      setUsername(data.name);
    };
    loadName();
  }, []);

  return (
    <div>
      <h1>Hello {username}!</h1>
      <p>Welcome to Electron!</p>
    </div>
  );
};

const init = () => {
  const root = createRoot(document.getElementById('content'));
  root.render(<HelloUser username="JP" />);
};

window.onload = init;
```

`package.json` scripts:

```json
  "scripts": {
    "start": "webpack && electron .",
    "test": "echo \"Error: no test specified\" && exit 1",
    "webpack": "webpack",
    "package": "webpack && npx electron-packager ."
  },
```

## How Basic-Electron-Done differs

The done repo is the same app with a few small differences. All of them work:

- `devTools: false` and `win.removeMenu()` are commented out, so DevTools still opens.
- `window-all-closed` is inside `whenReady` instead of at the bottom of the file.
- The `<script>` tag is in `<head>`. That still works because `init` waits for `window.onload`.
- There's an extra `otherPage.html` with a link to it. Moving between pages is just a link, since the files load straight off the disk.
- It says "Hello User!" before the fetch finishes, and it uses Electron 39, so its version line shows older numbers.
- Its packager is the older one too, so on Node 26 do Step 21a there before `npm run package`.
- Running it only takes `npm install`, then `npm start`. Its `package.json` already lists Electron and React.
