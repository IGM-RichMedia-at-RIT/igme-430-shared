# The Server Scaffold

Every `server.js` in this class starts with the same twenty or so lines. They create the server, pick a port, and decide which function handles each request. Once you know what each line does, a blank `server.js` stops being scary, because you're typing the same shape every time and only the routes change.

## Quick example

This is the `server.js` shape we've used since Week 3, from `accept-header-example` in 3B:

```js
const http = require('http');
const responseHandler = require('./responses.js');

const port = process.env.PORT || process.env.NODE_PORT || 3000;

const urlStruct = {
  '/': responseHandler.getIndex,
  '/cats': responseHandler.getCats,
  default: responseHandler.getIndex,
};

const onRequest = (request, response) => {
  const protocol = request.connection.encrypted ? 'https' : 'http';
  const parsedUrl = new URL(request.url, `${protocol}://${request.headers.host}`);

  const handler = urlStruct[parsedUrl.pathname];
  if (handler) {
    handler(request, response);
  } else {
    urlStruct.default(request, response);
  }
};

http.createServer(onRequest).listen(port, () => {
  console.log(`Listening on 127.0.0.1: ${port}`);
});
```

And the other half, `responses.js`, which holds the functions that actually send something back:

```js
const fs = require('fs');

const index = fs.readFileSync(`${__dirname}/../client/client.html`);

const getIndex = (request, response) => {
  response.writeHead(200, { 'Content-Type': 'text/html' });
  response.write(index);
  response.end();
};

module.exports = {
  getIndex,
};
```

`server.js` decides *which* function runs. `responses.js` knows *how* to send each thing.

## What each piece does

| Line | What it's for |
|---|---|
| `require('http')` | Node's built-in module for making a server. No `npm install` needed |
| `require('./responses.js')` | Pulls in whatever `responses.js` put in `module.exports`. The `./` means "a file of mine," not a package |
| `process.env.PORT \|\| process.env.NODE_PORT \|\| 3000` | Heroku tells your app which port to use through `PORT`. On your laptop that isn't set, so it falls back to 3000 |
| `urlStruct` | The routing table: each path maps to the function that handles it |
| `onRequest(request, response)` | Runs once for every request. `request` is what the browser sent (URL, method, headers). `response` is what you fill in and send back |
| `new URL(request.url, ...)` | `request.url` is only the path, like `/cats?name=bob`. `new URL` needs a full address, so we build one from the protocol and the `Host` header, and it hands back the pieces (`pathname`, `searchParams`, and so on) |
| `urlStruct[parsedUrl.pathname]` | Looks up the handler for this path. `undefined` if there isn't one |
| `http.createServer(onRequest).listen(port, ...)` | Makes the server, tells it to call `onRequest` for every request, and starts listening. The callback runs once, when it's ready |

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 1C (Fri 8/28) | `require` and `module.exports` | [first-node-done](https://github.com/IGM-RichMedia-at-RIT/first-node-done): `src/index.js`, `src/myData.js` |
| 2A (Mon 8/31) | `createServer`, `listen`, the port line, serving `client.html` with `fs.readFileSync` | Starter: [basic-http-class-example](https://github.com/IGM-RichMedia-at-RIT/basic-http-class-example) · Done: [basic-http-class-example-done](https://github.com/IGM-RichMedia-at-RIT/basic-http-class-example-done) |
| 2B (Wed 9/2) | Moving handlers into `responses.js`, a second page, `if/else` routing | Austin's version: [extended-basic-http-example](https://github.com/IGM-RichMedia-at-RIT/extended-basic-http-example) · Done: [extended-basic-http-example-done](https://github.com/IGM-RichMedia-at-RIT/extended-basic-http-example-done) |
| 2C (Fri 9/4) | `if/else` to `switch` | Same repo as 2B |
| 3B (Wed 9/9) | `new URL`, the `urlStruct` routing table | [accept-header-example](https://github.com/IGM-RichMedia-at-RIT/accept-header-example) (see [The Accept header](accept-header.md)) |

A few differences between what we typed and Austin's done repos:

- Austin's 2B code is its own repo that starts where 2A ended. In class we kept building in the same repo from Monday. Either way you end up with the same code.
- Our second page lives at `/page2`. In `extended-basic-http-example-done` it's `/otherPage`. The server picks the path, so both work.
- `extended-basic-http-example-done` stops at `if/else`. The `switch` version we wrote in 2C is in step 8 below.

Videos:

- [Austin, Week 2.1: Basic HTTP, ESLint, and CircleCI](https://www.youtube.com/watch?v=twQ8_tab6mc). The server part is the same as ours. The CI part is out of date, since we use GitHub Actions now instead of CircleCI.
- [Austin, Week 2.2: Servers](https://www.youtube.com/watch?v=STwvHi4yYFY). No code. Explains IP addresses and ports, which is what that `port` line is about.
- [Austin, Week 3.1: Accept Headers](https://www.youtube.com/watch?v=ElramkPkvaA). This is where the routing table comes in.

Used in: every assignment and both projects. [Status codes](status-codes.md) and [The Accept header](accept-header.md) both start from this scaffold. For the file-serving half of it (`readFileSync`, `Content-Type`, the CSS that won't load), see [Serving files and MIME types](serving-files-mime.md).

## Common mistakes

- **The browser spins forever on some URLs.** Some path doesn't reach a function that calls `response.end()`. Usually an `if` with no `else`, or a routing table with no fallback. Every request has to get an answer, even if it's the index page or a 404.
- **The page loads, then the server crashes with `Cannot write headers after they are sent to the client`.** A `switch` case is missing its `break`. The right handler runs and sends the page, then the code falls through into the next case, which tries to send a second response. Every case gets a `break`, including `default`.
- **A route shows the index page instead of what it should.** The path doesn't match exactly, so it fell through to the default. Common versions: `case 'helloJSON'` without the leading `/` (the path is always `/helloJSON`), a capital letter that doesn't match the assignment (`/dankMeme` vs `/dankmemes`), or your own name for a route the assignment names for you. Nothing errors, because the fallback answers with a page, so this one is easy to miss. Type the exact URL from the assignment into the browser and check.
- **`responses.getIndex is not a function`.** You wrote the function in `responses.js` but didn't add it to `module.exports`, or the name is spelled differently in the two files.
- **`'parsedUrl' is not defined` (or `request`, or `response`).** The handler lookup ended up outside `onRequest`, up next to `urlStruct`. Those variables only exist inside `onRequest`.
- **The server crashes as soon as it starts, with something like `Cannot read properties of undefined (reading 'writeHead')`.** You wrote `'/cats': responseHandler.getCats()` in the routing table. The parentheses call the function right away, before any request exists. The table should hold the function itself, so leave them off.
- **`ENOENT: no such file or directory` when the server starts.** The path in `readFileSync` is wrong. `__dirname` is the folder the current file is in (`src/`), so `../client/client.html` goes up one level and then into `client/`.
- **`EADDRINUSE`.** A server from earlier is still running on that port. Find the old terminal and hit Ctrl+C.
- **You changed the code and nothing changed.** The server only reads your code (and `client.html`) when it starts. Stop it and run `npm start` again. From Week 5 on, nodemon does this for you.

## What you'll find on Google

- **`const app = express(); app.get('/cats', ...); app.listen(3000);`** This is Express. It's the same idea as our routing table, with the table built for you. Express comes later in the semester, and it'll make more sense having built the table by hand first.
- **`import http from 'http'`** instead of `require`. That's the newer module syntax (ES modules). Our projects use `require` and `module.exports` (CommonJS), and mixing the two in one project causes errors.
- **`url.parse(request.url)`**. An older way to split up a URL. It's deprecated. Use `new URL(...)`.
- **Examples that hardcode `.listen(3000)` or `.listen(8080)`.** Fine on your laptop, but Heroku picks the port, so a hardcoded one breaks once you deploy. Keep the `process.env.PORT` line.
- **`fs.readFile` with a callback, or `fs.promises`, inside the request handler.** That reads the file from disk on every request. We read our files once with `readFileSync` when the server starts and keep them in memory. It's fast, and blocking is fine there because no one is connected yet.

## Redo it

This goes from an empty `src` folder to the routing table, in the order we did it in class. Start from a fresh copy of the [starter](https://github.com/IGM-RichMedia-at-RIT/basic-http-class-example). It has `client/client.html` and an ESLint config, but no `package.json` and no `src` folder.

Restart the server after every step (Ctrl+C, then `npm start`).

### 1. Set up `npm start`

In the project folder, run `npm init -y`. Then open `package.json` and set the `start` script:

```json
"scripts": {
  "start": "node ./src/server.js"
},
```

(Getting ESLint and `npm test` working is its own setup. If you need it, copy the scripts and `devDependencies` from the done repo's `package.json` and run `npm install`.)

### 2. The smallest server that works

Create `src/server.js`:

```js
const http = require('http');

const port = process.env.PORT || process.env.NODE_PORT || 3000;

const onRequest = (request, response) => {
  console.log(request.url);
  response.writeHead(200, { 'Content-Type': 'text/html' });
  response.end();
};
```

### 3. Start listening

At the bottom of `server.js`:

```js
http.createServer(onRequest).listen(port, () => {
  console.log(`Listening on 127.0.0.1:${port}`);
});
```

Checkpoint: `npm start`, then open `http://127.0.0.1:3000`. The page is blank, but the terminal logs `/`, and probably `/favicon.ico` too. The server answered with a 200 and an empty body.

### 4. Send the page

Under the `http` line, add:

```js
const fs = require('fs');
```

Under the `port` line:

```js
const index = fs.readFileSync(`${__dirname}/../client/client.html`);
```

In `onRequest`, just above `response.end();`:

```js
response.write(index);
```

Checkpoint: refresh and `client.html` shows up.

### 5. Move the handler into `responses.js`

Create `src/responses.js`:

```js
const fs = require('fs');

const index = fs.readFileSync(`${__dirname}/../client/client.html`);

const getIndex = (request, response) => {
  response.writeHead(200, { 'Content-Type': 'text/html' });
  response.write(index);
  response.end();
};

module.exports = {
  getIndex,
};
```

### 6. Point `server.js` at it

In `server.js`, delete the `fs` line and the `index` line. Under the `http` line, add:

```js
const responses = require('./responses.js');
```

Then make `onRequest` hand off:

```js
const onRequest = (request, response) => {
  console.log(request.url);
  responses.getIndex(request, response);
};
```

Checkpoint: the page looks exactly the same. Only the code moved.

### 7. A second page, and `if/else`

Make `client/client2.html` (anything, as long as it looks different). In `responses.js`, under the `index` line:

```js
const client2 = fs.readFileSync(`${__dirname}/../client/client2.html`);
```

Add a `getClient2` that's a copy of `getIndex` but writes `client2`, and add `getClient2` to `module.exports`. Then in `onRequest`, replace the `getIndex` line:

```js
if (request.url === '/page2') {
  responses.getClient2(request, response);
} else {
  responses.getIndex(request, response);
}
```

Checkpoint: `/page2` shows the new page. `/` and `/anything-else` show the first one.

### 8. `if/else` to `switch`

Same logic, easier to read once you have more than a few routes. Replace the `if/else`:

```js
switch (request.url) {
  case '/page2':
    responses.getClient2(request, response);
    break;
  default:
    responses.getIndex(request, response);
    break;
}
```

Checkpoint: same results as step 7. If you want to see why `break` matters, delete the one after the `/page2` case and visit `/page2`. The page loads, and then the server crashes in the terminal. Put the `break` back.

### 9. Parse the URL

From here on we route on `parsedUrl.pathname` instead of `request.url`, so a query string like `?name=bob` doesn't break the match. At the top of `onRequest`:

```js
const protocol = request.connection.encrypted ? 'https' : 'http';
const parsedUrl = new URL(request.url, `${protocol}://${request.headers.host}`);
console.log(parsedUrl);
```

Checkpoint: visit `/page2?name=bob`. The terminal prints a `URL` object with `pathname: '/page2'` and the query in `search` and `searchParams`.

### 10. The routing table

Above `onRequest`, below the `port` line:

```js
const urlStruct = {
  '/': responses.getIndex,
  '/page2': responses.getClient2,
  default: responses.getIndex,
};
```

No parentheses after the function names. We're storing the functions, not calling them.

### 11. Look up the route

Inside `onRequest`, delete the `console.log(parsedUrl)` and the whole `switch`, and replace them with:

```js
const handler = urlStruct[parsedUrl.pathname];
if (handler) {
  handler(request, response);
} else {
  urlStruct.default(request, response);
}
```

Checkpoint: `/`, `/page2`, `/page2?name=bob`, and `/garbage` all work, and adding a route is now one new line in `urlStruct`.
