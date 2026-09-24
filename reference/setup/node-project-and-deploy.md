# Starting a Node Project and Deploying It

Every project and assignment this semester starts the same way. You need a `package.json` with a `start` script, a server in `src/server.js` that reads its port from the environment, a `.gitignore` that keeps `node_modules` out of Git, and a lint check that passes on GitHub. Then Heroku runs it.

The pipeline we're building toward:

```
VS Code -> Git -> GitHub -> GitHub Actions (lint check) -> Heroku -> the internet
```

## Quick example

The `scripts` section of `package.json`. Heroku runs `npm start`, and GitHub Actions runs `npm test`:

```json
"scripts": {
  "start": "node ./src/server.js",
  "pretest": "eslint ./src --fix",
  "test": "echo \"Tests complete\""
},
```

The smallest working server, `src/server.js`:

```js
const http = require('http');

const port = process.env.PORT || process.env.NODE_PORT || 3000;

const onRequest = (request, response) => {
  console.log(request.url);
  response.writeHead(200, { 'Content-Type': 'text/plain' });
  response.write('Hello server.');
  response.end();
};

http.createServer(onRequest).listen(port, () => {
  console.log(`Listening on 127.0.0.1:${port}`);
});
```

Heroku picks the port your app has to use and hands it to you in `process.env.PORT`. On your laptop that variable doesn't exist, so it falls back to 3000.

Sharing code between your own files (CommonJS):

```js
// src/myData.js
const getMessage = () => 'Hello World';
module.exports = { getMessage };

// src/server.js
const myData = require('./myData.js');
myData.getMessage();
```

## The npm commands we use

| Command | What it does |
|---|---|
| `npm init` | Asks some questions and writes `package.json`. Change the entry point to `./src/server.js`, press Enter for the rest |
| `npm install underscore` | Downloads a package into `node_modules` and adds it to `dependencies` |
| `npm install --save-dev eslint @eslint/js globals` | Same, but lists them under `devDependencies`. These are tools for writing code, and your app doesn't need them to run |
| `npm install` | Reads `package.json` and puts back everything in `node_modules`. Run this after every clone |
| `npm start` | Runs the `start` script |
| `npm test` | Runs `pretest` (ESLint) first, then `test`. If ESLint finds an error, `test` never runs |
| `npm ci` | A clean install from `package-lock.json`. This is what GitHub Actions runs |

`start` and `test` are special. Any other script you add runs with `npm run <name>`.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 1B (Wed 8/26) | Making the repo on GitHub with the Node `.gitignore`, cloning, `npm init`, Hello World | Your own `first-node` repo (no starter) |
| 1C (Fri 8/28) | The `start` script, `require` and `module.exports`, `npm install`, the first HTTP server, the zombie-server fix, deploying to Heroku | Done: [first-node-done](https://github.com/IGM-RichMedia-at-RIT/first-node-done) |
| 2A (Mon 8/31) | ESLint, `pretest`, `--save-dev`, deleting and restoring `node_modules`, GitHub Actions | Starter: [basic-http-class-example](https://github.com/IGM-RichMedia-at-RIT/basic-http-class-example) |
| | | Done: [basic-http-class-example-done](https://github.com/IGM-RichMedia-at-RIT/basic-http-class-example-done) |

Where to look:

- `first-node-done`: `package.json` for the scripts, `src/myData.js` for `module.exports`, `src/index.js` for the server
- `basic-http-class-example-done`: `package.json` for `pretest` and `devDependencies`
- Any starter repo: `.github/workflows/node.js.yml` is the GitHub Actions check, and `eslint.config.js` holds the lint rules. Don't edit either one, the assignments grade against them

`first-node-done` is a little different from what we typed in class. Its server is `src/index.js`, and in class we used `src/server.js` (every later repo uses `server.js` too). It also has a `Polygon.js` class example we didn't get to.

Videos:

- [Austin, Week 1.2: Tools Overview](https://www.youtube.com/watch?v=GL_BfMltuD4). An older recording. It calls Heroku free (it isn't anymore) and uses CircleCI for CI (we use GitHub Actions).
- [Austin, Week 1.3: First Node Demo](https://www.youtube.com/watch?v=xksZCkshgQM)
- [Austin, Week 2.1: Basic HTTP, ESLint, and CircleCI](https://www.youtube.com/watch?v=twQ8_tab6mc). The ESLint part still applies. Ignore the CircleCI part.

Used in: every assignment and both projects. Each submission needs a working Heroku link and a green check on your latest commit.

## Heroku costs

The student credit from the GitHub Student Developer Pack is $13 a month for 24 months. It doesn't roll over, so treat it as $13 a month, not a $312 balance.

Subscribe to the Eco Dynos plan on your Heroku Billing page. Eco is $5 a month for your whole account, with as many apps as you want, and the apps sleep after 30 minutes with no visitors. Without Eco, each app is a Basic dyno at $7 a month. The first app fits inside the credit, so nothing looks wrong at first, but by the fourth or fifth assignment your card starts getting charged. Go look at your Billing page now and check that it says Eco.

## Common mistakes

- **`EADDRINUSE: address already in use :::3000`.** A server is already running on that port, usually one you stopped with Ctrl+Z (that pauses it, it doesn't quit) or left open in another terminal. Your code is fine. Kill the old one: `npx kill-port 3000` works everywhere, or `lsof -ti:3000 | xargs kill -9` on Mac. Stop servers with Ctrl+C. Don't switch to port 3001 to get around it.
- **`node_modules` shows up in `git status`.** Your repo has no `.gitignore`. Run `echo "node_modules/" >> .gitignore`. If you already committed `node_modules`, you also need `git rm -r --cached node_modules` or Git keeps tracking it. Project 1 takes points off for this.
- **`myData.getMessage is not a function`.** You `require`d the file but never exported the function. Add `module.exports = { getMessage };` at the bottom of that file.
- **`sh: eslint: command not found` when you run `npm test`.** You haven't installed the dev dependencies on this machine, or you deleted `node_modules`. Run `npm install`.
- **`No files matching the pattern "./src" were found.`** ESLint is installed, but there's nothing in `src` yet. It goes away once `src/server.js` exists.
- **The GitHub check is red and ESLint passes on your machine.** Look at the `test` script. The default one from `npm init` is `echo "Error: no test specified" && exit 1`, and the `exit 1` fails on purpose. Replace it with `echo "Tests complete"`. Also make sure `package-lock.json` is committed, since `npm ci` refuses to run without it.
- **Your Actions tab says 0 workflow runs.** Forks come with Actions turned off. Turn them on from the Actions tab, and do it before your first push. Turning them on later doesn't go back and check old pushes, and the starter workflow has no "Run workflow" button. Push an empty commit to start a run: `git commit --allow-empty -m "trigger CI"`, then `git push`.
- **The starter repo's own checks are red.** Expected. The starter has no `package.json`, so `npm ci` fails. Yours turns green once you commit `package.json` and `package-lock.json`.
- **Heroku shows "Application error".** Open the logs (More, then View logs, on your app's dashboard). If they say the web process failed to bind to `$PORT`, you hard-coded 3000 instead of using `process.env.PORT`. If they say a module can't be found, you installed a package with `--save-dev` that your app needs while it runs. Heroku removes `devDependencies` after it builds.
- **You submitted the dashboard link.** Graders need the app's own URL, the one the View button opens (it ends in `herokuapp.com`).
- **You wrote `"engine"` instead of `"engines"`.** npm quietly ignores the misspelled key. The block is optional, but if you have it, spell it `engines`.

## What you'll find on Google

- **`import http from 'http'` and `"type": "module"`.** That's ES modules, the newer syntax. It works fine, but our repos and the ESLint setup use CommonJS (`require` and `module.exports`). Don't mix the two in one project.
- **A `Procfile` with `web: node index.js`.** Lots of Heroku tutorials add one. You don't need it: without one, Heroku runs `npm start`.
- **`heroku create` and `git push heroku main`.** Deploying from the Heroku CLI. It works, but we connect the GitHub repo on the Deploy tab of the dashboard instead.
- **`.eslintrc.json` and the Airbnb style guide.** The old ESLint config format. We use the newer `eslint.config.js` with ESLint's own `recommended` rules. Old tutorials and older course videos show the old way.
- **CircleCI, Travis, and others.** Other CI services. We use GitHub Actions, which is already set up in every starter repo.
- **Render, Railway, Vercel.** Other hosts. They're real options, but assignments are graded on a Heroku link.

## Redo it

There are two parts. The first is the Week 1 project you make yourself. The second is the lint and CI setup from 2A, done in the Basic HTTP starter.

### Part 1: Your first Node project (1B and 1C)

#### 1. Make the repo on GitHub

On [github.com/new](https://github.com/new): owner is your own account, name `first-node`, Public, check "Add a README file", and set "Add .gitignore" to `Node`. That last one is the step people skip.

Then clone it and go into the folder:

```bash
git clone <your repo's HTTPS url>
cd first-node
```

#### 2. `npm init`

```bash
npm init
```

Press Enter through the questions, except for the entry point: type `./src/server.js`.

Checkpoint: `package.json` exists, and `"main"` says `./src/server.js`.

#### 3. Hello World and the `start` script

Make a `src` folder with a `server.js` in it:

```js
console.log('Hello World');
```

In `package.json`, add a `start` line to the `scripts` section:

```json
"start": "node ./src/server.js",
```

Checkpoint: `npm start` prints `Hello World`.

#### 4. A module of your own

Make `src/myData.js`:

```js
const message = 'Hello World';

const getMessage = () => {
  console.log(message);
  return message;
};
```

Replace `server.js` with:

```js
const myData = require('./myData.js');

console.log(myData.getMessage());
```

Checkpoint: `npm start` crashes with `myData.getMessage is not a function`. That's expected, because nothing is exported yet.

#### 5. Export it

At the bottom of `myData.js`:

```js
module.exports = { getMessage };
```

Checkpoint: `npm start` prints `Hello World` twice, once from inside `getMessage` and once from `server.js`. `message` itself stays private to `myData.js`, because only `getMessage` is exported.

#### 6. Install a package

```bash
npm install underscore
```

Replace `server.js` with:

```js
const _ = require('underscore');
const myArray = [1, 2, 3, 4, 5];
console.log(_.chunk(myArray, 3));
```

Checkpoint: `npm start` prints `[ [ 1, 2, 3 ], [ 4, 5 ] ]`. Run `git status`: you see `package.json`, `package-lock.json`, and `src/`, but not `node_modules`.

#### 7. The HTTP server

Clear out `server.js` and set up the server:

```js
const http = require('http');

const port = process.env.PORT || process.env.NODE_PORT || 3000;
```

Then the request handler and the `listen` call:

```js
const onRequest = (request, response) => {
  console.log(request.url);
  response.writeHead(200, { 'Content-Type': 'text/plain' });
  response.write('Hello server.');
  response.end();
};

http.createServer(onRequest).listen(port, () => {
  console.log(`Listening on 127.0.0.1:${port}`);
});
```

Checkpoint: `npm start`, then open `http://localhost:3000`. You see `Hello server.`, and the terminal logs `/` (plus `/favicon.ico`, which the browser asks for on its own). Stop the server with Ctrl+C.

#### 8. Push it

```bash
git status
git add .
git commit -m "made a server"
git push
```

Checkpoint: your repo on GitHub has `package.json`, `package-lock.json`, and `src/`, and no `node_modules`.

#### 9. Deploy it on Heroku

Before your first app, check your Heroku Billing page. You should see the platform credit and the Eco Dynos plan (see Heroku costs above).

1. On the [Heroku dashboard](https://dashboard.heroku.com/), click New, then Create new app. Name it something like `first-node-yourname-2026`.
2. Open the Deploy tab. Under Deployment method, pick GitHub, find your `first-node` repo, and click Connect.
3. Scroll down to Manual deploy and click Deploy Branch.
4. Watch the build log. It downloads Node, installs your packages, and launches your app.

Checkpoint: click View. `Hello server.` loads from a `herokuapp.com` address. That URL is the one you submit.

### Part 2: ESLint and GitHub Actions (2A)

#### 1. Fork the starter and turn on Actions

Fork [basic-http-class-example](https://github.com/IGM-RichMedia-at-RIT/basic-http-class-example) to your account. On your fork, open the Actions tab and click the green button to turn workflows on. Do this before you push anything. Then clone your fork.

Checkpoint: `git remote -v` shows your username, not `IGM-RichMedia-at-RIT`.

#### 2. `npm init` and the scripts

Run `npm init` with entry point `./src/server.js`. Then replace the `scripts` section in `package.json`:

```json
"scripts": {
  "start": "node ./src/server.js",
  "pretest": "eslint ./src --fix",
  "test": "echo \"Tests complete\""
},
```

Checkpoint: `npm test` fails with `eslint: command not found`. Nothing's installed yet.

#### 3. Install ESLint

```bash
npm install --save-dev eslint @eslint/js globals
```

Checkpoint: `npm test` now fails with `No files matching the pattern "./src" were found.` It's a different error, which means ESLint is running.

#### 4. Give it something to lint

Make `src/server.js`. The server from Part 1 step 7 works for this. The rest of 2A (serving `client.html`) is covered in the HTTP servers section of this guide.

Checkpoint: `npm test` ends with `Tests complete`. The `console.log` warnings are fine. Only errors fail the check.

#### 5. Push and check

```bash
git add .
git commit -m "basic http server"
git push
```

Checkpoint: on GitHub, the Actions tab shows a "Node.js CI" run, and it finishes with a green check next to your commit. If it says 0 workflow runs, push an empty commit (see Common mistakes).
