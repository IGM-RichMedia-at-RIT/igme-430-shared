# Serving Files and MIME Types

As far as the network is concerned, an HTML page, a stylesheet, a PNG and an MP3 are all the same thing: a pile of bytes. The `Content-Type` header on the response is what tells the browser how to read them. Its value is a MIME type, like `text/html` or `text/css`.

The server is always authoritative. The browser doesn't look at the bytes and guess. It does whatever `Content-Type` says, even when the server is wrong. Send an HTML page as `text/plain` and the browser shows you the raw tags.

## Quick example

From `src/responses.js`, the version we ended with on 2C:

```js
const fs = require('fs');

const index = fs.readFileSync(`${__dirname}/../client/client.html`);
const style = fs.readFileSync(`${__dirname}/../client/style.css`);

const serveFile = (request, response, content, mimeType) => {
  response.writeHead(200, { 'Content-Type': mimeType });
  response.write(content);
  response.end();
};

const getIndex = (request, response) => serveFile(request, response, index, 'text/html');
const getCSS = (request, response) => serveFile(request, response, style, 'text/css');
```

And the route for the stylesheet in `src/server.js`:

```js
case '/style.css':
  responses.getCSS(request, response);
  break;
```

Two things are going on at the top of that file:

- `fs.readFileSync` runs once, when the server starts, and keeps the file in memory. Every request after that sends the copy that's already loaded, which is much faster than reading the disk each time. Reading synchronously is normally a bad idea on a server, but here it happens before the server takes any requests, so nothing is waiting on it.
- `__dirname` is the folder the current file lives in (`src/`). Building the path from it means the same code works on your laptop, a classmate's laptop and Heroku. `../client/` goes up out of `src/` and into `client/`.

Because the files are read once at startup, **editing `client.html` or `style.css` does nothing until you restart the server.**

## The MIME types we use

| Type | What it's for | Where it shows up |
|---|---|---|
| `text/html` | HTML pages | Every server since 2A |
| `text/css` | Stylesheets | 2C, and the `getCSS` handler in later demos |
| `text/plain` | Plain text | `/message` on 2B, `/hello` and `/time` in Simple HTTP |
| `application/json` | JSON | Simple HTTP, then nearly every API from 3B on |
| `application/xml` | XML | 3B and the HTTP API assignments. `text/xml` also works, just be consistent (see [The Accept header](accept-header.md)) |
| `application/javascript` | JavaScript files | `bundle.js` in the Week 5 webpack demo |
| `video/mp4`, `audio/mpeg` | Video and audio | Streaming Media assignment. The MP3 type is `audio/mpeg`, not `audio/mp3` |
| images | PNG, JPEG, and so on | Look these up on [MDN's common types list](https://developer.mozilla.org/en-US/docs/Web/HTTP/MIME_types/Common_types). Finding the PNG type yourself is part of Simple HTTP |

When you need a type that isn't here, MDN's list is the place to look.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 2A (Mon 8/31) | `fs.readFileSync`, `__dirname`, serving `client.html` as `text/html` | Starter: [basic-http-class-example](https://github.com/IGM-RichMedia-at-RIT/basic-http-class-example) · Done: [basic-http-class-example-done](https://github.com/IGM-RichMedia-at-RIT/basic-http-class-example-done) |
| 2B (Wed 9/2) | Moving responses into `responses.js`, a second page, `sendPage`, a `text/plain` endpoint | Starter: [extended-basic-http-example](https://github.com/IGM-RichMedia-at-RIT/extended-basic-http-example) · Done: [extended-basic-http-example-done](https://github.com/IGM-RichMedia-at-RIT/extended-basic-http-example-done) |
| 2C (Fri 9/4) | `if/else` to `switch`, MIME types broken on purpose, the CSS problem, `serveFile` | Same code as 2B, no separate repo |

Where to look:

- `src/responses.js` in extended-basic-http-example-done: the `readFileSync` lines at the top, `sendPage`, and `getMessage`
- `src/server.js` in basic-http-class-example-done: the single-file version from 2A, with long comments on `readFileSync` and `__dirname`

What we did in class goes further than Austin's done repos:

- We kept building in the same repo from Monday instead of switching to extended-basic-http-example. The code is the same idea either way.
- Our second page lives at `/page2`. Austin's done repo calls it `/otherPage`.
- Austin's done repo stops at `sendPage` and an `if/else`. The `switch`, `style.css`, `getCSS` and `serveFile` from 2C aren't in any Week 2 repo. You can see the same pattern later on: `src/htmlResponses.js` in [body-parse-example](https://github.com/IGM-RichMedia-at-RIT/body-parse-example) has a `getCSS`, and [nodemon-webpack-demo-done](https://github.com/IGM-RichMedia-at-RIT/nodemon-webpack-demo-done) has a `serveFile` that takes `(response, file, contentType)` and also sets `Content-Length`.

Video: [Austin, Week 2.1: Basic HTTP, ESLint, and CircleCI](https://www.youtube.com/watch?v=twQ8_tab6mc) covers `readFileSync`, `__dirname`, and setting `Content-Type` to `text/html`. The course uses GitHub Actions now instead of CircleCI. There's no video for the 2B and 2C material.

Routing (the `if/else`, the `switch`, and later the routing table) has its own page: [The server scaffold](server-scaffold.md).

Used in: Simple HTTP Assignment, Streaming Media Assignment, and any server that serves its own stylesheet or scripts, including both projects.

## Common mistakes

- **Your CSS doesn't load, and nothing looks broken.** When the page links `style.css`, the browser sends a second request to your server: `GET /style.css`. If there's no route for it, it falls to your default and gets the HTML page back with a 200. Open the Network tab and click `style.css`: the `Content-Type` is `text/html` and the Response tab shows your HTML. Add a `/style.css` route.
- **You added the route and the CSS still doesn't apply.** Check the type you're sending. A stylesheet sent as `text/html` gets refused, and Chrome's console says something like *"Refused to apply style ... because its MIME type ('text/html') is not a supported stylesheet MIME type."* It has to be `text/css`.
- **The browser shows your HTML as raw text.** You sent it as `text/plain`. The browser did exactly what you told it.
- **You edited `client.html` and nothing changed.** The file was read when the server started. Stop it and start it again. If it's still old, the browser may be caching it: check "Disable cache" in the Network tab.
- **The server crashes on startup with `ENOENT: no such file or directory`.** The path in a `readFileSync` is wrong. Remember `__dirname` is `src/`, so client files are at `` `${__dirname}/../client/...` ``. Since this runs at startup, one bad path means the server never boots.
- **You asked for one page and got a different one.** A `case` in your `switch` is missing its `break`, so it falls through into the next case. Every case gets a `break`, including `default`.
- **The file you link doesn't have to match the route name.** `href="style.css"` only needs a route called `/style.css`. The server makes up its URLs. They don't have to match anything on disk.

## What you'll find on Google

- **`app.use(express.static('public'))`.** The Express way to serve a whole folder. It picks the MIME type from the file extension for you. We do it by hand first so you know what it's doing. Express comes later in the semester.
- **`fs.readFile` (without Sync) inside the request handler.** This reads the file from disk on every request. It works, and it's the right call for big or changing files. Our files are small and don't change while the server runs, so we load them once.
- **The `mime` or `mime-types` packages.** They look up the type from a file extension. Handy when you serve lots of file types. We serve a handful, so we write the type ourselves.
- **`path.join(__dirname, '..', 'client', 'client.html')`.** The same path as our template string, built a different way. Either is fine. The Streaming Media assignment uses `path.resolve`.
- **`fs.createReadStream(file).pipe(response)`.** Streaming, instead of loading the whole file into memory. That's what the Streaming Media assignment does for video and audio, because those files are too big to hold in memory. Small pages and stylesheets don't need it.

## Redo it

Start from a fresh copy of [extended-basic-http-example](https://github.com/IGM-RichMedia-at-RIT/extended-basic-http-example). It's where 2A left off: `src/server.js` reads `client.html` and sends it for every URL. Run `npm install` once, then restart the server (`npm start`) after every step.

The checkpoints use `curl -i` from a second terminal, which prints the status line and headers above the body.

### 1. Make `responses.js` and read the page at startup

Create `src/responses.js`:

```js
const fs = require('fs');

const index = fs.readFileSync(`${__dirname}/../client/client.html`);
```

### 2. One helper that can send anything

Still in `responses.js`, under the `index` line:

```js
const serveFile = (request, response, content, mimeType) => {
  response.writeHead(200, { 'Content-Type': mimeType });
  response.write(content);
  response.end();
};

const getIndex = (request, response) => serveFile(request, response, index, 'text/html');

module.exports = { getIndex };
```

In class we got here in two stages (`sendPage` on 2B, then `serveFile` on 2C). This skips straight to the final version.

### 3. Use it from `server.js`

In `src/server.js`, replace the `fs` line and the `index` line at the top with:

```js
const responses = require('./responses.js');
```

Then replace everything inside `onRequest` with:

```js
console.log(request.url);
responses.getIndex(request, response);
```

Checkpoint: `curl -i localhost:3000` shows `200 OK`, `Content-Type: text/html`, and the page.

### 4. A second page, and a `switch`

Copy `client/client.html` to `client/client2.html` and change the `<h1>` text so you can tell them apart. In `responses.js`, under the `index` line:

```js
const client2 = fs.readFileSync(`${__dirname}/../client/client2.html`);
```

Under `getIndex`:

```js
const getClient2 = (request, response) => serveFile(request, response, client2, 'text/html');
```

Change the export to `module.exports = { getIndex, getClient2 };`.

### 5. Route to it

In `server.js`, replace the `responses.getIndex(...)` line in `onRequest` with:

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

Checkpoint: `localhost:3000/page2` shows the second page. Any other path shows the first.

### 6. Plain text

In `responses.js`, under `getClient2`:

```js
const getMessage = (request, response) => serveFile(request, response, 'Hello world', 'text/plain');
```

Add `getMessage` to the export. In `server.js`, add a case above `default`:

```js
case '/message':
  responses.getMessage(request, response);
  break;
```

Checkpoint: `curl -i localhost:3000/message` shows `Content-Type: text/plain` and `Hello world`.

### 7. Move the CSS out, and watch it break

Create `client/style.css` and move everything inside the `<style>` tags of `client.html` into it. In `client.html`, replace the whole `<style>` block with:

```html
<link rel="stylesheet" href="style.css">
```

Restart and reload with the Network tab open. The heading is stuck in the top left corner.

Checkpoint: `curl -i localhost:3000/style.css` gives `200 OK` with `Content-Type: text/html` and the HTML page as the body. That's the bug. The browser asked for the stylesheet and got your default route.

### 8. Serve the stylesheet as CSS

In `responses.js`, under the other `readFileSync` lines:

```js
const style = fs.readFileSync(`${__dirname}/../client/style.css`);
```

Under `getMessage`:

```js
const getCSS = (request, response) => serveFile(request, response, style, 'text/css');
```

Add `getCSS` to the export. In `server.js`, add a case above `default`:

```js
case '/style.css':
  responses.getCSS(request, response);
  break;
```

Checkpoint: `curl -i localhost:3000/style.css` shows `Content-Type: text/css` and your CSS. Reload the page and the heading is centered again.
