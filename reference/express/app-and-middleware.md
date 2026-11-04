# Express: the app and middleware

Everything you wrote by hand for Project 1 (serving files, parsing a body, setting `Content-Type` and `Content-Length`, answering HEAD, sending a 404), Express does in about a line each. The server is now an `app`, and you configure it by stacking `app.use` lines. Each one is a stop every request passes through, in the order you wrote them. That order is the hardest part of Express, and it's the first thing to check when something is weird.

This page is the setup half of the first Express demo: `src/app.js`. The routes and controllers are on [Express: routes, controllers, and the 404](router-controllers-404.md).

## Quick example

From `src/app.js` in [simple-mvc-controllers-done](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-controllers-done/blob/master/src/app.js), with the comments trimmed and the lines in the order we used in class:

```js
const path = require('path');
const express = require('express');
const compression = require('compression');
const favicon = require('serve-favicon');

const router = require('./router.js');

const port = process.env.PORT || process.env.NODE_PORT || 3000;

const app = express();

app.use(compression());
app.use('/assets', express.static(path.resolve(`${__dirname}/../client/`)));
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(favicon(`${__dirname}/../client/img/favicon.png`));

router(app);

app.listen(port, (err) => {
  if (err) { throw err; }
  console.log(`Listening on port ${port}`);
});
```

Read it top to bottom as what happens to one request. Compress the reply. Does the URL start with `/assets`? Send the file and stop. Is it the favicon? Send it and stop. Is there a form body? Parse it into `req.body`. A JSON body? Same. Then hand it to the router.

Compare it with [the server scaffold](../http/server-scaffold.md). The `http.createServer` and `onRequest` are gone. Express runs those for you.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 8B (Wed 10/14) | `express()` and `listen`, the free 404, `compression`, `express.static`, body parsing, `favicon`, then routes and controllers | Starter: [simple-mvc-controllers](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-controllers) · Done: [simple-mvc-controllers-done](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-controllers-done) |

Where to look in the done repo:

- `src/app.js`: all of it. The comments in the done repo explain each line at length.
- `package.json`: `express`, `compression`, and `serve-favicon` are the three new regular dependencies. `npm run nodemon` watches `src/`.

The done repo differs from what we typed in two spots. It puts `compression()` after `express.static`, which means static files never get compressed (see Common mistakes). And it has no `express.json()` line, so it only parses form bodies. Our version in class had both.

Video: [Austin, Week 7.2: Simple MVC, Controllers](https://www.youtube.com/watch?v=JthuX0jkFb0). Same demo, older Express, so the wildcard route is written `'/*'` there. On Express 5 that crashes; see the routes page.

Used in: DomoMaker A and everything after it, Project 2. The DomoMaker `app.js` is this file with more lines in the same stack (sessions, helmet, Handlebars).

## Common mistakes

- **`req.body` is `undefined` in a controller.** The two body-parsing `app.use` lines are missing, or they're below `router(app)`. Middleware only runs for a request if it comes before the route that handles it.
- **A big CSS or JS file isn't compressed** (no `Content-Encoding: gzip` in the response headers). `compression()` is below `express.static`. Static ends the request when it finds the file, so anything under it never runs. Put compression first.
- **An image 404s.** The URL needs both the `/assets` prefix and the extension: `/assets/img/internet.png`. Static hosting serves exact file names only. `/assets/img/internet` gives `Cannot GET`.
- **`403 Forbidden` on a page, `ForbiddenError` in the terminal.** A `sendFile` path with `../` in it and no `path.resolve` around it. Express refuses relative paths that climb out of a folder. `path.resolve` turns it into a clean absolute path first.
- **Nothing happens when you run it.** `node src/app.js` exits right away and prints nothing. There's no `app.listen` yet. With nodemon you'll see `clean exit - waiting for changes before restart`.
- **`EADDRINUSE :::3000`.** A server from earlier is still running. `lsof -ti tcp:3000 | xargs kill -9` on a Mac, or find and close the old terminal.
- **The favicon request still shows up in the Network tab after every page load.** The `favicon` line is missing, or the path to the image is wrong. When it's working, `curl -I localhost:3000/favicon.ico` shows `Cache-Control: public, max-age=31536000`, which is one year.

## What you'll find on Google

- **`body-parser` as a separate package** (`const bodyParser = require('body-parser'); app.use(bodyParser.json())`). That's the old way. Since Express 4.16 it's built in as `express.json()` and `express.urlencoded()`. Same code underneath, one less install.
- **`express.static('public')` with no `path.resolve`.** Works in tutorials because the folder sits next to the file that's run. Ours lives at `../client/` relative to `src/app.js`, so we resolve it to an absolute path.
- **`app.use(express.static(...))` with no `'/assets'` prefix.** Also fine. Files are then served from the root (`/img/internet.png`). The prefix keeps static URLs from colliding with your routes.
- **`extended: false`.** The common copy-paste. `true` lets form field names nest (`user[name]` becomes `req.body.user.name`). `false` keeps them flat. For a form with plain fields like ours, both give the same result. Express 5 defaults to `false` if you leave it out.
- **`import express from 'express'`.** ES module syntax. Our projects use `require`, which matches the starters, DomoMaker, and ESLint's config. Pick one per project and don't mix.
- **Writing your own middleware** (`(req, res, next) => { ...; next(); }`). That's what every `app.use` line is. We don't write one in this demo, but when you do, call `next()` or the request hangs.

## Redo it

Start from a fresh clone of the [starter](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-controllers) and run `npm install`. The audit warnings are normal. Work in `src/app.js`. Run the server with `npm run nodemon` so it restarts on save.

You'll want a second terminal for `curl`. `-i` shows the status line and headers, `-I` sends a HEAD.

### 1. Make the app and listen

At the bottom of `src/app.js`, under the `port` line:

```js
const port = process.env.PORT || process.env.NODE_PORT || 3000;   // already there

const app = express();

app.listen(port, (err) => {
  if (err) { throw err; }
  console.log(`Listening on port ${port}`);
});
```

Checkpoint: the terminal says `Listening on port 3000`.

### 2. See the free 404

Open `localhost:3000`. You get `Cannot GET /`. Now from the second terminal:

```
curl -i localhost:3000/
```

```
HTTP/1.1 404 Not Found
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 139
```

Zero routes, and it already sends a correct 404 with `Content-Type` and `Content-Length`. Those were two of the Project 1 self-checks.

### 3. Compression, first

Every `app.use` from here on goes between `const app = express();` and `app.listen`, in this order.

```js
const app = express();   // already there

app.use(compression());
```

Zip the response, send the zip, let the browser unzip it. It goes first on purpose: the next line can end a request, and anything below that line never runs for it.

### 4. Static hosting

```js
app.use(compression());   // already there
app.use('/assets', express.static(path.resolve(`${__dirname}/../client/`)));
```

Checkpoint: `localhost:3000/assets/img/internet.png` shows the image. It's 1.3 MB, so give it a second.

The URL maps straight onto the folder:

```
/assets/img/internet.png   ->   client/img/internet.png
```

Try `localhost:3000/assets/img/internet` with no extension. `Cannot GET`. Static hosting is full file name or nothing. Good for images, CSS, and JS. Not for pages.

### 5. Body parsing

```js
app.use('/assets', express.static(path.resolve(`${__dirname}/../client/`)));   // already there
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
```

That is the whole of [`parseBody`](../http/parsing-post-body.md) from Week 4. Form bodies and JSON bodies both land in `req.body` as an object before any controller runs. There's nothing to see yet, because no route reads `req.body`. The routes page gets there.

### 6. Favicon

```js
app.use(express.json());   // already there
app.use(favicon(`${__dirname}/../client/img/favicon.png`));
```

Checkpoint:

```
curl -I localhost:3000/favicon.ico
```

`200 OK`, `Content-Type: image/x-icon`, and `Cache-Control: public, max-age=31536000`. The browser asks for the favicon on every page load in a raw Node server. This answers once and tells it to cache for a year. The `Vary: Accept-Encoding` header you also see is compression doing its job, which only shows up because it's above this line.

### 7. Hand off to the router

```js
app.use(favicon(`${__dirname}/../client/img/favicon.png`));   // already there

router(app);
```

Middleware first, then routes. The `router` function is empty in the starter, so every page still 404s. Filling it in is [Express: routes, controllers, and the 404](router-controllers-404.md).

### 8. Prove the order matters

Drop any CSS file bigger than a few KB into `client/` (copy one from an old project), then:

```
curl -I localhost:3000/assets/your-file.css
```

With compression first, the headers include `Content-Encoding: gzip` and `Vary: Accept-Encoding`. Now cut the `app.use(compression())` line and paste it below the static line. Save and run the same `curl`. The `gzip` header is gone, and `Content-Length` is the full file size. Static hosting ended the request before compression ever saw it.

Put compression back at the top and delete the test file. The done repo has it in the lower spot, which is why it's worth seeing for yourself.

### What replaced what

| Project 1, by hand | Express |
|---|---|
| `http.createServer(onRequest)` | `const app = express();` |
| `server.listen(port)` | `app.listen(port, ...)` |
| A handler per file, `readFileSync`, picking the MIME type ([serving files](../http/serving-files-mime.md)) | `app.use('/assets', express.static(...))` |
| `parseBody` with chunks and `Buffer.concat` ([parsing a POST body](../http/parsing-post-body.md)) | `express.urlencoded()` and `express.json()` |
| A `favicon.ico` route, or ignoring the request | `app.use(favicon(...))` |
| `urlStruct` and `onRequest` | `router(app)` and `app.get` |

None of this is magic. You already wrote all of it. That's why we did it by hand first.
