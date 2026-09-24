# nodemon and webpack

Two tools we use for the rest of the semester.

- **nodemon** restarts your server every time you save, so you stop doing Ctrl-C, up arrow, Enter.
- **webpack** takes the browser code you write in `client/`, follows every `require`, and builds it into one file, `hosted/bundle.js`. That's how npm packages (and later, React) get into the page.

Every project from here on uses the same three folders:

```
src/      server code            (Node runs this)
client/   your browser JS        (you write here)
hosted/   what the browser gets  (webpack writes bundle.js here)
```

You write in `client/`. Webpack puts the result in `hosted/`. The server serves from `hosted/`.

## Quick example

`webpack.config.js`, at the root of the project next to `package.json`:

```js
const path = require('path');

module.exports = {
  entry: './client/client.js',
  mode: 'development',
  output: {
    path: path.resolve(__dirname, 'hosted'),
    filename: 'bundle.js',
  },
};
```

The scripts in `package.json` you end up with:

```json
"scripts": {
  "start": "node ./src/server.js",
  "pretest": "eslint ./src --fix",
  "test": "echo \"Tests complete\"",
  "nodemon": "nodemon -e js,html,css --watch ./src --watch ./hosted ./src/server.js",
  "buildBundle": "webpack",
  "watchBundle": "webpack watch",
  "heroku-postbuild": "webpack --mode production"
}
```

While you work, keep two terminals open:

```
npm run nodemon        (terminal 1: runs the server, restarts on changes)
npm run watchBundle    (terminal 2: rebuilds bundle.js on changes)
```

`start` and `test` are built in, so `npm start` works. Anything you make up needs `run`: `npm run nodemon`, not `npm nodemon`.

## Development vs production

| | Development | Production |
|---|---|---|
| Who runs it | You, with `npm run buildBundle` or `npm run watchBundle` | Heroku, with `heroku-postbuild` |
| What `bundle.js` looks like | Readable-ish, so you can debug it | One long minified line |
| Size, starter client code | 2.43 KiB | 674 bytes |
| Server started with | `npm run nodemon` | `npm start` (plain `node`) |

`--mode production` on the command line beats `mode: 'development'` in the config file. That's how your laptop gets the readable version and the live site gets the small one.

Heroku does this on every deploy, in this order: installs everything in `package.json` (including devDependencies), runs `heroku-postbuild`, removes the devDependencies, then runs `npm start`. That's why nodemon and webpack can be devDependencies. By the time the app runs, the bundle is already built, and nothing restarts on a live server, so plain `node` is all it needs.

## What goes where

| File | Who writes it | Committed? |
|---|---|---|
| `src/*.js` | you | yes |
| `client/*.js` | you | yes |
| `hosted/client.html`, `hosted/style.css` | you | yes |
| `hosted/bundle.js` | webpack. Never edit it by hand, it gets overwritten | yes, in our repos |
| `webpack.config.js` | you | yes |
| `node_modules/` | npm | no, it's in `.gitignore` |

Our repos commit `hosted/bundle.js` because the server reads it when it starts, so a fresh clone can run right away. Heroku rebuilds it anyway.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 5B (Wed 9/23) | nodemon, the three folders, webpack config, `require` in the browser, watch mode, `heroku-postbuild` | Starter: [nodemon-webpack-demo](https://github.com/IGM-RichMedia-at-RIT/nodemon-webpack-demo) |
| | | Done: [nodemon-webpack-demo-done](https://github.com/IGM-RichMedia-at-RIT/nodemon-webpack-demo-done) |

Where to look in the done repo:

- `package.json`: the `scripts`
- `webpack.config.js`: entry, mode, output
- `src/htmlResponses.js`: reads `hosted/bundle.js` and serves it with `getBundle`
- `src/server.js`: the `'/bundle.js'` route in `urlStruct`
- `client/client.js` and `client/other.js`: browser code using `require`
- `hosted/client.html`: `<script src="/bundle.js"></script>` instead of an inline script

The done repo is a little different from what we typed in class:

- It lists `webpack` and `webpack-cli` under `dependencies`. We installed them with `--save-dev`. Both work on Heroku, since webpack is only needed at build time.
- Its config has `watchOptions: { aggregateTimeout: 200 }` and a commented-out `watch: true`. We use the `webpack watch` script instead, which does the same job.
- It jumps straight to the full nodemon script. In class we started with `--watch ./src` and added the rest once the stale bundle showed us why.
- It uses `_.chunk` from underscore. We used `_.shuffle`. Any function works, the point is that an npm package runs in the browser.

Video: [Austin, Week 5.2: Dev Tools, Nodemon and Babel](https://www.youtube.com/watch?v=A9YxDfUCMm8). The nodemon part matches what we did. The rest of the video uses Babel, because it was recorded before the course switched to webpack. For webpack, trust the repo.

Used in: every DomoMaker assignment and Project 2. HTTP API Assignment II and Project 1 don't need it.

## Common mistakes

- **`ENOENT ... hosted/bundle.js` when the server starts.** The bundle hasn't been built yet. Run `npm run buildBundle` first.
- **You changed client code and the page didn't change.** Webpack rebuilt `bundle.js`, but the server read the old one when it started and is still holding it in memory. Restart the server (type `rs` in the nodemon terminal), or use the full nodemon script, which watches `hosted/` and restarts after each rebuild.
- **Buttons do nothing and the Console says `require is not defined`.** The browser got your raw `client/client.js`. The script tag has to point at `/bundle.js`.
- **`bundle.js` is a 404 in the Network tab.** The `'/bundle.js'` route is missing from `urlStruct`, or `getBundle` isn't exported from `htmlResponses.js`.
- **You edited `hosted/bundle.js` and your change disappeared.** Webpack overwrites it on every build. Edit the files in `client/`.
- **`Unknown command: "nodemon"`.** You typed `npm nodemon`. Custom scripts need `npm run nodemon`. If it says `Missing script`, check the spelling in `package.json` and the comma on the line above.
- **`EADDRINUSE` on startup.** An old server is still using the port, which can happen after you Ctrl-C nodemon. Close the other terminal, or on a Mac run `lsof -ti tcp:3000 | xargs kill -9`.
- **`npm test` fails with `Cannot find module '@eslint/js'`.** ESLint 10 moved `@eslint/js` and `globals` into their own packages. The starter already lists them, so run `npm install`. Don't reinstall ESLint with no version number: `npm install eslint` grabs whatever the newest major version is, and that's exactly how a new ESLint release broke everyone's `npm test` last spring. The `^10` in `package.json` keeps you on version 10.
- **A warning about "engines" when installing `webpack-cli`.** The newest `webpack-cli` needs Node 20.9 or newer. Update Node, or run `npm install --save-dev webpack-cli@5` (the version the done repo uses).
- **`npm install` takes minutes.** Your project is inside OneDrive, iCloud, or a network drive. Move it to a regular local folder.

## What you'll find on Google

- **Vite.** The popular choice now, and faster. It hides more of what's going on, which is why we learn webpack first. The ideas carry over.
- **Babel and `babel-loader`.** Lots of webpack tutorials (and Austin's older video) add Babel to convert newer JavaScript for older browsers. We don't need it for this.
- **`node --watch`.** Newer versions of Node can restart on changes without nodemon. It works, but nodemon gives us `rs` and the `-e` option for HTML and CSS, and it's what the course repos use.
- **`import` / `export` instead of `require` / `module.exports`.** Webpack understands both. Our repos use `require` on both the server and the client, so stick with that.
- **Putting the build folder (`dist/` or `build/`) in `.gitignore`.** Very common, and fine in a lot of projects. We commit `hosted/bundle.js` so the server can start from a fresh clone.
- **`webpack-dev-server` and hot reloading.** A separate server that reloads the page for you. We serve `bundle.js` from our own Node server, so we don't use it.

## Redo it

Start from a fresh copy of the [starter](https://github.com/IGM-RichMedia-at-RIT/nodemon-webpack-demo). It's the status codes demo with `style.css` added. Run `npm install` once, then `npm start` to make sure it works on `http://localhost:3000`.

### 1. Install nodemon

```
npm install --save-dev nodemon
```

`--save-dev` means you need it while you're working, and the live site doesn't.

Checkpoint: `package.json` lists `nodemon` under `devDependencies`.

### 2. Add a `nodemon` script

In `package.json`, under `"test"` (add a comma to the end of the `test` line):

```json
"nodemon": "nodemon --watch ./src ./src/server.js"
```

`--watch ./src` is what to watch. `./src/server.js` is what to run.

### 3. Run it and edit something

```
npm run nodemon
```

Change the message in `success` in `src/jsonResponses.js` and save.

Checkpoint: the terminal prints `[nodemon] restarting due to changes...` and `/success` shows the new message without you touching the terminal. Put the message back. Type `rs` and Enter any time you want to force a restart.

### 4. Move the page into `hosted/`

Leave nodemon running. Make a `hosted/` folder and move `client.html` and `style.css` into it. Then type `rs`.

Checkpoint: the server crashes with `Error: ENOENT: no such file or directory, open '.../src/../client/client.html'`, and nodemon says `app crashed - waiting for file changes before starting...`. That's expected. The server is still looking in `client/`.

### 5. Fix the two paths

At the top of `src/htmlResponses.js`, change `client` to `hosted`:

```js
const index = fs.readFileSync(`${__dirname}/../hosted/client.html`);
const css = fs.readFileSync(`${__dirname}/../hosted/style.css`);
```

Checkpoint: save, and nodemon restarts on its own. The page loads and the buttons work.

### 6. Install webpack

In a second terminal (leave nodemon running):

```
npm install --save-dev webpack webpack-cli
```

### 7. Move the script into `client/client.js`

Open `hosted/client.html`. Cut everything between `<script>` and `</script>` and paste it into a new file, `client/client.js`. Then replace the empty tag pair with:

```html
<link rel="stylesheet" type="text/css" href="/style.css">   <!-- already there -->
<script src="/bundle.js"></script>
```

The page now asks for `bundle.js`, and nothing makes it yet.

### 8. `webpack.config.js`

New file at the root of the project, next to `package.json`:

```js
const path = require('path');

module.exports = {
  entry: './client/client.js',
  mode: 'development',
  output: {
    path: path.resolve(__dirname, 'hosted'),
    filename: 'bundle.js',
  },
};
```

`entry` is where webpack starts. `output` is where the bundle goes.

### 9. Build it

In `package.json`, under `"nodemon"`:

```json
"buildBundle": "webpack"
```

Then run `npm run buildBundle`.

Checkpoint: the output ends with `asset bundle.js 2.43 KiB [emitted]` and `compiled successfully`, and `hosted/bundle.js` exists. If you want to see the production version, run `npx webpack --mode production` (674 bytes, one line), then `npm run buildBundle` again to go back.

### 10. Read the bundle on the server

In `src/htmlResponses.js`, under the `css` line:

```js
const bundle = fs.readFileSync(`${__dirname}/../hosted/bundle.js`);
```

Above `module.exports`:

```js
const getBundle = (request, response) => {
  serveFile(response, bundle, 'application/javascript');
};
```

Then add `getBundle,` to `module.exports` under `getCSS,`.

### 11. Route `/bundle.js`

In `src/server.js`, in `urlStruct`:

```js
'/style.css': htmlHandler.getCSS,        // already there
'/bundle.js': htmlHandler.getBundle,
```

Checkpoint: hard refresh (Ctrl-F5). The buttons work. In the Network tab, `bundle.js` is a `200` with `Content-Type: application/javascript`.

### 12. `require` another file in the browser

New file, `client/other.js`:

```js
const print = () => {
  console.log('testing');
};

module.exports = {
  print,
};
```

At the very top of `client/client.js`, add `const test = require('./other.js');`. As the last line inside `init`, add `test.print();`. Then run `npm run buildBundle`.

Checkpoint: the build output lists `./client/other.js` too. Refresh the page, and the Console is empty. The server is still holding the old bundle. Type `rs` in the nodemon terminal, refresh again, and the Console says `testing`.

npm packages work the same way: `npm install underscore`, then `const _ = require('underscore');` in `client/client.js`.

### 13. Watch mode, both sides

In `package.json`, add under `"buildBundle"`:

```json
"watchBundle": "webpack watch"
```

And replace the `nodemon` script with:

```json
"nodemon": "nodemon -e js,html,css --watch ./src --watch ./hosted ./src/server.js"
```

`--watch ./hosted` makes nodemon restart when webpack writes a new bundle. `-e js,html,css` adds HTML and CSS, since by default nodemon only watches `.js` and `.json` files.

Stop nodemon, then run `npm run nodemon` in one terminal and `npm run watchBundle` in the other.

Checkpoint: change `'testing'` in `client/other.js` and save. Webpack rebuilds, then nodemon restarts, and a refresh shows the new message without typing `rs`.

### 14. `heroku-postbuild`

In `package.json`, under `"watchBundle"`:

```json
"heroku-postbuild": "webpack --mode production"
```

Checkpoint: `npm run heroku-postbuild` prints `[minimized]` next to `bundle.js`. Run `npm run buildBundle` afterward to get the readable version back for local work. Then `npm test` should still say `Tests complete`.
