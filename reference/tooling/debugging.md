# Debugging a Node Server

When your server breaks, it usually tells you where. A crash prints a stack trace in the terminal that points at a file and a line number. When nothing crashes but the answer is still wrong, you pause the server partway through a request and look at the variables with a debugger, the same kind you've used for client-side JavaScript or C#.

Where to look first:

| What you see | Where to look |
|---|---|
| The server crashed and the terminal shows an error | The stack trace. Find the first line that points into your own `src/` folder |
| The request hangs forever | A handler that never sends a response (see [Status codes](../http/status-codes.md#common-mistakes)) |
| No crash, but the data or the format is wrong | Put a breakpoint in the handler and use the debugger |
| The page shows the wrong thing | The browser's Network tab: check the status, the response headers, and the response body |
| It works on your machine but not on Heroku | `heroku logs` |

## Quick example

Reading a stack trace. This is a real one from the class debugging demo:

```
/your-project/src/server.js:43
    htmlHandler.getCSS(request, response);
                ^

TypeError: htmlHandler.getCSS is not a function
    at handleGet (/your-project/src/server.js:43:17)
    at Server.onRequest (/your-project/src/server.js:58:5)
    at Server.emit (node:events:507:28)
    at parserOnIncoming (node:_http_server:1153:12)
```

- The top line gives you the file and line: `server.js`, line 43. It even prints the line and puts a `^` under the problem.
- `TypeError: htmlHandler.getCSS is not a function` says what went wrong. Something we expected to be a function isn't one.
- The `at ...` lines are the path that led there, most recent first. `handleGet` was called by `onRequest`. Lines that start with `node:` are Node's own internals, so skip past them to the lines in your files.

Starting the debugger. Add a `debug` script to `package.json`, next to `start`:

```json
"scripts": {
  "start": "node ./src/server.js",
  "debug": "node --inspect ./src/server.js"
}
```

Run `npm run debug`. The terminal should say `Debugger listening on ws://127.0.0.1:9229/...`. Then open `chrome://inspect` in Chrome (or `edge://inspect` in Edge) and click **Open dedicated DevTools for Node**. From there, it works like the browser debugger you already know: open a file in the Sources panel, click a line number to set a breakpoint, and make a request.

## The debugger controls

| Control | What it does |
|---|---|
| Resume | Keep running until the next breakpoint |
| Step over | Run this line and stop on the next one |
| Step into | Go inside the function this line calls |
| Step out | Finish this function and stop where it was called from |
| Deactivate breakpoints | Keep your breakpoints but ignore them for now |
| Pause on exceptions | Stop on the line that throws an error, even if you didn't put a breakpoint there |

While you're paused, hover over any variable to see its value, or add it to the Watch panel to follow it as you step.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 5C (Fri 9/25) | Reading stack traces, the Node debugger with `--inspect` | Starter: [debugging-demo](https://github.com/IGM-RichMedia-at-RIT/debugging-demo) |

There's no done repo for this one, because the demo is about finding the bugs rather than writing new code. The fixes are all in Redo it below. The server is the same one as [Parsing a POST body](../http/parsing-post-body.md), with four bugs planted in it:

- `src/server.js`: `handleGet`, where the first crash shows up
- `src/htmlResponses.js`: the `module.exports` at the bottom, and the `readFileSync` for the CSS file
- `src/jsonResponses.js`: `respondJSON` and `addUser`

Austin also has a longer written guide, "Debugging Node," on myCourses. It covers the VS Code debugger and the Heroku CLI in more detail.

Video: [Austin, Week 6.1: Debugging Node](https://www.youtube.com/watch?v=LRNKmqNYh98). Same `node --inspect` and `chrome://inspect` steps as this page. Partway through, Chrome's inspector stops cooperating and he switches to Edge, which is worth watching if it happens to you.

Used in: everything from here on, starting with Project 1.

## Common mistakes

- **Reading the stack trace from the wrong end.** Start at the top and work down until you hit a line in your own code. Sometimes that first line is a helper that works everywhere else (like `respondJSON`). If so, keep reading down to the line that called it. That's usually the real problem.
- **Your breakpoint never gets hit.** Check that you started with `npm run debug`, not `npm start` or nodemon. Also check that the request actually reaches that line: a breakpoint inside `/getUsers` won't do anything if you're loading `/`.
- **You started two debug servers.** Only one program can use the debugger port. The second one prints `Starting inspector on 127.0.0.1:9229 failed: address already in use` and then runs without a debugger, so your breakpoints silently do nothing. Stop the old one first.
- **The browser spins while you're stopped at a breakpoint.** That's expected. The server is paused in the middle of the request. Press Resume.
- **`chrome://inspect` doesn't show your server, or DevTools won't connect.** Try `edge://inspect` in Edge. It's built on the same engine and works the same way.
- **The response is cut off, or the client says `Unexpected end of JSON input`.** The `Content-Length` header doesn't match what you actually wrote. The browser stops reading at the length you gave it. Measure the same string you send (step 3 below).
- **"Cannot write headers after they are sent to the client."** Two responses to one request. See [Status codes](../http/status-codes.md#common-mistakes). Older videos and docs call this "write after end," which is the same mistake.
- **`Cannot destructure property 'name' of 'request.body'`.** The body was never parsed onto `request.body`. See [Parsing a POST body](../http/parsing-post-body.md#common-mistakes).
- **Ignoring the linter.** Run `npm test` before you start guessing. On the debugging demo, ESLint catches the first bug before you even start the server: `'getCSS' is assigned a value but never used`.
- **Works locally, crashes on Heroku.** File names on Mac and Windows usually ignore capital letters, and Heroku's don't. `Style.css` and `style.css` are the same file on your laptop and different files on Heroku (step 4 below).

## What you'll find on Google

- **The VS Code debugger.** Open `server.js`, choose Run, then Start Debugging, then pick Node.js. It starts the server for you and gives you the same breakpoints, stepping, variables, and watches inside the editor. It's the same Node debugger with a different window around it, so use whichever you like.
- **`node inspect` (no dashes).** That's a text-only debugger that runs in the terminal. `node --inspect` is the one that connects to Chrome or VS Code.
- **`--inspect-brk`.** Pauses before the first line of your code runs, so you can press Resume when you're ready. Useful when the crash happens at startup, before any request comes in.
- **`console.log` everywhere.** Totally fine, and often the fastest check. The debugger is better when you don't know where to look yet, because you can see every variable at once without adding and removing logs.
- **Postman, Thunder Client, and `curl`.** Different ways to send a request to your server without building a page for it. The pages in this guide use `curl`. Pick whichever you like.
- **`DEBUG=express:*` and similar.** Logging options built into Express and other libraries. They don't do anything for our plain `http` server.

## Redo it

Clone the [debugging-demo](https://github.com/IGM-RichMedia-at-RIT/debugging-demo) repo, run `npm install`, then `npm start`, and open `http://localhost:3000`. Restart the server after every fix. The line numbers below are from the repo as it is now. Older videos and docs may say something slightly different.

### 1. The crash on page load

The page starts to load, and then the server crashes. The terminal shows the stack trace from the Quick example: `server.js`, line 43, `htmlHandler.getCSS is not a function`.

`htmlHandler` is whatever `htmlResponses.js` exports. Open that file. `getCSS` is written there, but it's missing from the exports at the bottom. Add it:

```js
module.exports = {
  getIndex,
  getCSS,
};
```

Checkpoint: restart, and the page loads with its styles. `curl -i localhost:3000/style.css` gives a `200`.

### 2. The crash when you add a user

Fill in the form and submit it, or:

```
curl -i -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "name=jp&age=40" localhost:3000/addUser
```

The server crashes with `Cannot write headers after they are sent to the client`. Find the first lines in your own code:

```
    at respondJSON (/your-project/src/jsonResponses.js:7:12)
    at addUser (/your-project/src/jsonResponses.js:48:12)
```

Line 7 is the `writeHead` inside `respondJSON`, and that works fine for every other response, so keep reading. Line 48 is in `addUser`, and the line right above it calls `respondJSON` too. That's two responses for one request. Delete the call without the `return` (line 47), so the block reads:

```js
if (responseCode === 201) {
  responseJSON.message = 'Created Successfully';
  return respondJSON(request, response, responseCode, responseJSON);
}
```

Checkpoint: run the curl twice. You get `201`, then `204`, and the server stays up.

### 3. The wrong answer, no crash (use the debugger)

Go to `localhost:3000/getUsers`. You get `{"users":` and nothing else. There's no crash, so there's no stack trace. Time for the debugger.

Add the `debug` script from the Quick example to `package.json` and start the server with `npm run debug`. Open `chrome://inspect`, click **Open dedicated DevTools for Node**, and open `jsonResponses.js` in the Sources panel (Ctrl+P or Cmd+P finds files by name). Set a breakpoint on the `response.writeHead` line in `respondJSON`, then load `/getUsers` again.

Checkpoint: the server pauses on that line. Hover over `content`. It's the string `"content"`, not your data. The line above it stringifies the word `'content'` instead of the `object`. `Content-Length` gets measured from that 9-character string, so the browser stops reading after 9 characters, which is `{"users":`.

Fix both lines, so the header measures what actually gets sent:

```js
const content = JSON.stringify(object);
```

and further down:

```js
response.write(content);
```

Checkpoint: press Resume and restart. Users are only kept in memory, so the list is empty after a restart. Add one again with the curl from step 2, and `/getUsers` shows the whole object: `{"users":{"jp":{"name":"jp","age":"40"}}}`.

### 4. The one that only breaks on Heroku

Everything works on your machine now. Deploy it to Heroku, and the app crashes as soon as it starts. Your terminal isn't there to show you the error, so ask Heroku for its logs:

```
heroku logs --tail -a your-app-name
```

In the log you'll find:

```
Error: ENOENT: no such file or directory, open '/app/src/../client/Style.css'
```

`ENOENT` means the file doesn't exist. Look in the `client` folder: the file is `style.css` with a lowercase s. Your laptop doesn't care about the capital letter, but Heroku runs on Linux, which does. Fix the path in `htmlResponses.js`:

```js
const css = fs.readFileSync(`${__dirname}/../client/style.css`);
```

Checkpoint: commit, push, and the app starts on Heroku.
