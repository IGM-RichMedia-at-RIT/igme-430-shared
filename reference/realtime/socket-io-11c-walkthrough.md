# Socket.IO: the 11C class walkthrough

> **A heads-up first.** This page is optional. It isn't assigned, and Project 2 doesn't need sockets. There's no video for this day, so the choice was no guide at all or the guide I teach from. I'm sharing it for anyone who wants to run through the sockets demo start to finish. It's rougher than the other pages in this reference and doesn't follow their format.
>
> **About AI:** the write-up around the code was drafted with AI help. The code was written by people: it comes from our class repos, and I typed and ran all of it myself. The one exception is the `console.log(socket.rooms)` in Step 24, a checkpoint line AI suggested. It isn't in the done repo.

**Where we did this:** Week 11, Friday (11C)

- Starter: [basic-socket-io](https://github.com/IGM-RichMedia-at-RIT/basic-socket-io)
- Done: [basic-socket-io-done](https://github.com/IGM-RichMedia-at-RIT/basic-socket-io-done) and [socket-io-rooms-done](https://github.com/IGM-RichMedia-at-RIT/socket-io-rooms-done). Neither matches this page line for line, and the differences are listed [at the bottom](#how-the-done-repos-differ)
- Video: none for this day

---

## What this demo shows

Every request all semester has gone one way: the browser asks, the server answers, done. Here the connection stays open and the server can talk first. The moment to look for is two windows side by side: type in one, and it shows up in the other with no refresh. Then rooms, so the server only sends a message to the people who should get it.

## How to use this page

- You type in four files. Every typing step opens with **Stay in** or **Switch to**: 🟦 `server/io.js` (new), 🟧 `server/app.js`, 🟩 `views/index.handlebars`, 🟪 `hosted/client.js`.
- Every typing step ends with **Save →** and what you should see, including "nothing yet." Steps 3 to 10 show almost nothing, and that's expected. The first real output is Step 11.
- You need two terminals. **Terminal 1** runs the server and **Terminal 2** is for the install. "Terminal 1 restarts" means it prints `[nodemon] restarting due to changes...`, then `Listening on port 3000`.
- **Where logs show up:** `console.log` in `io.js` prints in Terminal 1. `client.js` errors print in the browser console.
- **nodemon only watches `./server`.** Edit a view or `client.js` and you have to refresh the browser. Edit `io.js` and nodemon restarts, and open pages reconnect on their own in about 2 seconds, so `a user connected` lines appear with nobody touching anything.
- In the code blocks, `// already there` marks a line you already have, so you can find where the new code goes.
- If you only want the main idea, stop after Step 20. Part 5 adds rooms.

**What runs when:**

| You do this | Client (`client.js`) | Server (`io.js`) |
|---|---|---|
| Load `localhost:3000` (from Step 11 on) | `io()` opens the connection | `'connection'` fires → `a user connected` |
| Refresh an open page (from Step 12 on) | old connection closes, a new one opens | `a user disconnected`, then `a user connected` |
| Close a window | | `'disconnect'` → `a user disconnected` |
| Save `io.js` (nodemon restarts) | reconnects by itself | `'connection'` again, once per open page |
| Save a view or `client.js` | nothing until you refresh | nothing |
| Type + Enter (or Submit) | submit listener → `socket.emit('chat message', ...)` | `handleChatMessage` |
| Change the dropdown (Part 5) | change listener → `socket.emit('room change', ...)` | `handleRoomChange` |

`localhost:3000/socket.io/socket.io.js` is served by Socket.IO itself. Nobody writes that route.

---

## Start here

### 1. The one-way street

```
HTTP:       browser → request → server → response → (connection closes)
WebSocket:  browser ⇄ server (stays open; either side can send, any time)
```

Everything we've built this semester works the first way: the browser asks and the server answers. The server can't speak first. If you wanted a chat app with `fetch`, every browser would have to ask "anything new?" every second. Here we open a connection that stays open, and either side can send whenever it wants.

Chat, notifications, multiplayer games, and two people editing one Google Doc are all built on this. Socket.IO wraps the browser's WebSocket API and handles reconnecting for us.

### 2. Start the starter

**2a.** Get the starter and install it:

```
git clone https://github.com/IGM-RichMedia-at-RIT/basic-socket-io.git
cd basic-socket-io
npm install
```

`npm install` prints warnings that look worse than they are. On Node 23 you get ten `npm warn EBADENGINE` blocks, because ESLint doesn't list that version. On other versions you may not see them. It also reported `13 vulnerabilities (8 moderate, 3 high, 2 critical)` when I ran it, and your number may differ. Neither one stops the demo. Don't run `npm audit fix --force`, which can install breaking changes and leave you with a different starter than this page expects.

**2b.** In **Terminal 1**:

```
npm run nodemon
```

**Check:** `Listening on port 3000`. Leave it running. Everything from `io.js` prints here for the rest of the demo.

**2c.** Load `localhost:3000` in **two browser windows side by side**. Use `Ctrl+N` on Windows or `Cmd+N` on a Mac for the second one, not a second tab. Each one stands in for a different user, and "both windows" means these two from here on. Open the DevTools Console in one of them.

**Check:** a blank page in each, tab title `Socket Chat!`.

It's Express + Handlebars like Simple MVC Templates: an empty `<body>` in `views/index.handlebars` and an empty `init` in `hosted/client.js`.

---

## Part 1: Give the server a socket

### 3. Install socket.io

Open a second terminal in the same folder. That's **Terminal 2**.

```
npm install socket.io
```

**Check:** Terminal 2 says about `added 29 packages`, with the same warnings as Step 2. Terminal 1 stays quiet. `package.json` dependencies gain `"socket.io"` (`^4.8.4` when I ran it). It's a normal dependency because the server needs it at runtime.

### 4. A new file for socket code

🟦 **New file: `server/io.js`.**

```js
const http = require('http');
const { Server } = require('socket.io');

let io;
```

`http` is Node's own module, the one from Week 2. Express has been sitting on top of it all along.

`let io;` is one variable for the whole file, so every function below can use it.

**Save →** Terminal 1 restarts, because nodemon saw a new file in `./server`. You'll see this after every save in `server/` from here on. Nothing uses `io.js` yet.

### 5. The setup function

🟦 **Stay in `server/io.js`.** **Where:** under `let io;`.

```js
const socketSetup = (app) => {
  const server = http.createServer(app);
  io = new Server(server);

  return server;
};

module.exports = socketSetup;
```

`http.createServer(app)` is a plain Node server that hands every normal request to Express. Socket.IO attaches to *this* server, not to Express.

**Save →** Terminal 1 restarts.

### 6. Say when someone connects

🟦 **Stay in `server/io.js`.** **Where:** inside `socketSetup`, between `io = new Server(server);` and `return server;`.

```js
  io = new Server(server);   // already there

  io.on('connection', (socket) => {
    console.log('a user connected');
  });
```

`socket` is one object per connected browser. Everything we do with that person goes through it.

**Don't drop the `(socket)`.** Nothing uses it until Step 12, so leaving it out shows no error here. At Step 12 the server crashes with `ReferenceError: socket is not defined`.

**Save →** Terminal 1 restarts.

### 7. Wire it into `app.js`

🟧 **Switch to `server/app.js`.**

**7a. Where:** under the router require.

```js
const router = require('./router.js');   // already there
const socketSetup = require('./io.js');
```

**7b. Where:** under `router(app);`.

```js
router(app);   // already there
const server = socketSetup(app);
```

**7c. Change** `app.listen(port, ...` **to** `server.listen(port, ...`. Only that word changes.

```js
server.listen(port, (err) => {   // was app.listen
```

`app.listen` quietly makes its own server, and Socket.IO isn't attached to that one. We built our own server in `io.js` and attached Socket.IO to it, so that's the one that has to listen. Express still handles every normal route, because we handed it `app`.

**Save →** Terminal 1 restarts.

### 8. Check

**Check:** Terminal 1 says `Listening on port 3000`, no errors. Your two windows still show the blank page.

Open a **new tab** (`Ctrl+T` or `Cmd+T`) and go to `localhost:3000/socket.io/socket.io.js`.

**Check:** a wall of JavaScript, and the first lines say `Socket.IO v4.8.4` or newer. Terminal 1 prints nothing. **Close that tab**, because Step 11 needs both windows back on `localhost:3000`.

We never wrote that route. Socket.IO serves its own browser library. Nobody has connected yet, because our page doesn't load it.

---

## Part 2: Connect the browser

### 9. Load the library, then our code

🟩 **Switch to `views/index.handlebars`.** **Where:** inside the empty `<body>`.

```html
  <script src="/socket.io/socket.io.js"></script>
  <script src="/assets/client.js"></script>
```

Order matters. `socket.io.js` defines `io`, and `client.js` uses it.

**Save →** nothing to see yet. Terminal 1 stays quiet (nodemon doesn't watch views), and the page stays blank even if you refresh. Nothing connects until `client.js` calls `io()`.

### 10. Connect

🟪 **Switch to `hosted/client.js`.** **Where:** line 1, above `const init`.

```js
const socket = io();
```

**Save →** nothing yet. The open pages still have the old `client.js` until the refresh in Step 11.

### 11. Refresh `localhost:3000` in both windows

**Check:** Terminal 1 says `a user connected`, twice. This is the first real output. The page itself is still blank. (Only one line means one window isn't on `localhost:3000`.)

Nobody clicked anything. The page loaded, opened a connection, and it's still open right now.

### 12. Say when someone leaves

🟦 **Switch to `server/io.js`.** **Where:** inside the `'connection'` handler, under the `console.log`.

```js
    console.log('a user connected');   // already there

    socket.on('disconnect', () => {
      console.log('a user disconnected');
    });
```

**Save →** then watch Terminal 1. That's Step 13.

### 13. Watch the terminal after saving

**Check:** Terminal 1 restarts, then `a user connected` twice, with no refresh, after about 2 seconds. The browser console stays clean.

The server restarted and both pages found it again by themselves. You'll see this every time you save this file, so don't let the extra lines confuse you.

Close the right window. **Check:** `a user disconnected`.

Reopen it (`Ctrl+N` or `Cmd+N`, `localhost:3000`, back on the right). **Check:** `a user connected`.

If you get `ReferenceError: socket is not defined`, then `app crashed`, Step 6's `(socket)` is missing. Add it and save, and nodemon restarts on its own.

---

## Part 3: Browser → server

### 14. A box to type in

🟩 **Switch to `views/index.handlebars`.** **Where:** inside `<body>`, **above** the two script tags.

```html
  <form id="editForm">
    <input id="editBox" type="text" />
    <input type="submit" />
  </form>

  <div id="messages"></div>
```

**Save →** nothing yet. The form appears when you refresh at Step 17.

### 15. Catch the submit

🟪 **Switch to `hosted/client.js`.**

**15a. Where:** under `const socket = io();`.

```js
const handleEditBox = () => {
    const editForm = document.getElementById('editForm');
    const editBox = document.getElementById('editBox');

    editForm.addEventListener('submit', (e) => {
        e.preventDefault();
    });
};
```

**15b. Where:** inside `init`.

```js
const init = () => {   // already there
    handleEditBox();
```

**15c. Where:** inside the submit listener, under `e.preventDefault();`.

```js
        e.preventDefault();   // already there

        if (editBox.value) {
            socket.emit('chat message', editBox.value);
            editBox.value = '';
        }
```

`socket.emit(name, data)` sends `data` under an event name. We made up `'chat message'`. Both sides just have to spell it the same.

**Save →** nothing yet. The server isn't listening for `'chat message'` until Step 16.

### 16. Hear it on the server

🟦 **Switch to `server/io.js`.**

**16a. Where:** under `let io;`, above `socketSetup`.

```js
const handleChatMessage = (msg) => {
  console.log(msg);
};
```

**16b. Where:** inside the `'connection'` handler, under the `disconnect` block.

```js
    });   // already there (end of disconnect)

    socket.on('chat message', handleChatMessage);
```

**Save →** Terminal 1 restarts, then `a user connected` twice. That's your open pages reconnecting. They still have the old `client.js` until Step 17's refresh.

### 17. Refresh, type `hello`, press Enter

Refresh both windows.

**Check:** the form appears. Terminal 1 prints `a user disconnected`, then `a user connected`, twice. Every refresh from now on prints these pairs.

Left window: click the box, type `hello`, press Enter.

**Check:** Terminal 1 says `hello`. The box clears, and the page doesn't change yet.

That's the same direction as `fetch` so far: browser to server. Next is the other way.

---

## Part 4: Server → everyone

### 18. Send it to everybody

🟦 **Stay in `server/io.js`.** **Where:** `handleChatMessage`. **Change** `console.log(msg);` **to:**

```js
  io.emit('chat message', msg);   // was console.log(msg);
```

**Save →** Terminal 1 restarts, `a user connected` twice.

### 19. Show what arrives

🟪 **Switch to `hosted/client.js`.**

**19a. Where:** above `const init`.

```js
const displayMessage = (msg) => {
    const messageDiv = document.createElement('div');
    messageDiv.textContent = msg;
    document.getElementById('messages').appendChild(messageDiv);
};
```

**19b. Where:** inside `init`, under `handleEditBox();`.

```js
    handleEditBox();   // already there
    socket.on('chat message', displayMessage);
```

**Save →** nothing until the refresh in Step 20.

### 20. Two windows

This is the whole point of the demo.

Refresh both. Type in the left one, press Enter. **Check:** it appears in both.

Type in the right one, press Enter. **Check:** both again. Terminal 1 prints nothing for messages now, because Step 18 replaced the `console.log`.

Nothing on the right asked for that. No `fetch`, no refresh. The server pushed it into a page that was just sitting there.

There are three ways the server can send:

| Code | Who gets it |
|---|---|
| `socket.emit(...)` | just that one person |
| `io.emit(...)` | everyone connected |
| `io.to(room).emit(...)` | everyone in one room (Part 5) |

**This is a good place to stop.** Real-time chat works. Part 5 makes the server choose who gets each message.

---

## Part 5: Rooms

### 21. A room picker

🟩 **Switch to `views/index.handlebars`.** **Where:** inside the form, under the text input.

```html
    <input id="editBox" type="text" />   <!-- already there -->
    <select id="channelSelect">
      <option value="general">General</option>
      <option value="memes">Memes</option>
    </select>
```

**Save →** nothing until the refresh in Step 23.

### 22. Tell the server when it changes

🟪 **Switch to `hosted/client.js`.**

**22a. Where:** above `const init`.

```js
const handleChannelSelect = () => {
    const channelSelect = document.getElementById('channelSelect');
    const messages = document.getElementById('messages');

    channelSelect.addEventListener('change', () => {
        messages.innerHTML = '';
        socket.emit('room change', channelSelect.value);
    });
};
```

**22b. Where:** last line of `init`.

```js
    socket.on('chat message', displayMessage);   // already there
    handleChannelSelect();
```

**Save →** nothing until the refresh in Step 23.

### 23. The dropdown lies

Refresh both. Right window's dropdown → **Memes**. **Check:** Terminal 1 prints nothing, because the server isn't listening for `'room change'` yet.

Type in the left window, press Enter. **Check:** it still shows up on the right.

The server doesn't know rooms exist. It's still `io.emit`, so everyone gets everything. You could make each page ignore other rooms, but the data still reaches every browser. If those were private messages, that's a leak. The server has to decide.

### 24. Move a socket between rooms

🟦 **Switch to `server/io.js`.** **Where:** under `handleChatMessage`, above `socketSetup`.

```js
const handleRoomChange = (socket, roomName) => {
  socket.rooms.forEach((room) => {
    if (room === socket.id) return;
    socket.leave(room);
  });
  socket.join(roomName);
  console.log(socket.rooms);
};
```

Leave every room, then join the new one. Step 26 explains the `socket.id` line.

**Save →** Terminal 1 restarts, `a user connected` twice. Nothing calls this function until Step 25.

### 25. Start everyone in General, and listen for changes

🟦 **Stay in `server/io.js`.**

**25a. Where:** `'connection'` handler, under the `console.log`.

```js
    console.log('a user connected');   // already there
    socket.join('general');
```

**25b. Where:** under the `socket.on('chat message', ...)` line.

```js
    socket.on('chat message', handleChatMessage);   // already there
    socket.on('room change', (room) => handleRoomChange(socket, room));
```

`handleRoomChange` needs to know *which* socket, and wrapping it in an arrow function lets us hand `socket` in.

**Save →** Terminal 1 restarts, `a user connected` twice.

### 26. Check the rooms

Refresh both. nodemon put everyone back in General, but a dropdown can still say Memes until you do.

**Check:** both dropdowns on General. Terminal 1 prints the disconnect and connect pairs.

Right window's dropdown → **Memes**.

**Check:** Terminal 1 prints something like `Set(2) { 'VL5mf_o4qTg7f1_YAAAB', 'memes' }` (your id will differ). Messages still go to everyone until Step 27, so don't bother sending one yet.

Two rooms, and one of them is a random string. Socket.IO puts every socket in a room named after its own id. That's why we skip `socket.id` when we leave rooms.

### 27. Send only to the sender's room

🟦 **Stay in `server/io.js`.**

**27a. Where:** **replace** all of `handleChatMessage`:

```js
const handleChatMessage = (socket, msg) => {
  socket.rooms.forEach((room) => {
    if (room === socket.id) return;
    io.to(room).emit('chat message', msg);
  });
};
```

**27b. Change** the `'chat message'` line in the connection handler to pass `socket` in:

```js
    socket.on('chat message', (msg) => handleChatMessage(socket, msg));   // was handleChatMessage
```

Miss 27b and the first message crashes the server with `TypeError: Cannot read properties of undefined (reading 'forEach')`, and nodemon says `app crashed`.

**Save →** Terminal 1 restarts, `a user connected` twice. Finish both 27a and 27b before sending anything.

### 28. Rooms

Refresh both. **Check:** both dropdowns on General.

Right window's dropdown → **Memes**. **Check:** Terminal 1 prints `Set(2) { ..., 'memes' }`.

Left window: type `general only`, press Enter. **Check:** left only.

Right window: type `memes only`, press Enter. **Check:** right only.

Right window's dropdown → **General**. **Check:** its list clears, and Terminal 1 prints `Set(2) { ..., 'general' }`.

Left window: type another message, press Enter. **Check:** both.

The client code hasn't changed: it listens for `'chat message'` and nothing else. The server decides who hears it now.

---

## Where this goes

### 29. Going further

Project 2 doesn't require sockets. Its spec calls real-time games like Agar.io the "upper limit" of what's possible for the project. If you want to go past this demo, a few directions: save messages to Mongo and send the history on connect, use your DomoMaker login so messages have names, or build a turn-based game where each match is a room.

---

## If something goes wrong

- **`ReferenceError: socket is not defined`, then `app crashed`:** Step 6's `(socket)` is missing. Add it and save. Ctrl+C and restarting won't help, because open pages reconnect right away and crash it again.
- **Red `net::ERR_CONNECTION_REFUSED` or `WebSocket connection ... failed` repeating in the browser console:** the server is down, so look at Terminal 1. The page keeps retrying and reconnects by itself once the server is back.
- **`io is not defined` in the browser console:** the `socket.io.js` script tag is missing, or it's below `client.js`.
- **`io is not defined` plus a 404 for `socket.io.js`:** `app.js` still says `app.listen`. See Step 7c.
- **Page blinks, box clears, terminal shows `disconnected` and `connected` pairs, nothing arrives:** `e.preventDefault()` is missing. The form did a real submit and reloaded the page.
- **Sent, nothing in the terminal, no error:** the event names don't match (`'chat message'` vs `'chat-message'`).
- **Server crashed, `reading 'forEach'`:** see Step 27b.
- **Rooms act strange after a save:** nodemon put everyone back in General. Refresh both windows. If a dropdown still says Memes after the refresh (Chrome resets it, other browsers may not), pick General, then Memes. Choosing the option it's already on sends nothing.
- **Typed in `client.js`, nothing changed:** refresh. nodemon doesn't watch `hosted/`.
- **`EADDRINUSE :::3000`:** another server still has the port. Find the old terminal and press Ctrl+C, or on a Mac run `lsof -ti tcp:3000 | xargs kill -9`.

## Questions that come up

- **Can the server send to everyone except me?** `socket.broadcast.emit(...)`.
- **Does this work on Heroku?** It should. Heroku supports WebSockets and this is still one Express app, so one dyno is fine. I haven't tested it this semester.
- **How does the socket know who's logged in?** It doesn't, by default. Socket.IO has a documented way to share `express-session` with sockets. Look it up when a project needs it.
- **Why not fetch every second?** You can, with `setInterval`. Every page asks constantly, even when nothing's new. A socket sends only when something happens.
- **What if wifi drops?** It reconnects on its own. That's what you saw after every save.

---

## The finished code

`server/io.js`. The `console.log(socket.rooms)` is a checkpoint line, so delete it after Step 26 if you like.

```js
const http = require('http');
const { Server } = require('socket.io');

let io;

const handleChatMessage = (socket, msg) => {
  socket.rooms.forEach((room) => {
    if (room === socket.id) return;
    io.to(room).emit('chat message', msg);
  });
};

const handleRoomChange = (socket, roomName) => {
  socket.rooms.forEach((room) => {
    if (room === socket.id) return;
    socket.leave(room);
  });
  socket.join(roomName);
  console.log(socket.rooms);
};

const socketSetup = (app) => {
  const server = http.createServer(app);
  io = new Server(server);

  io.on('connection', (socket) => {
    console.log('a user connected');
    socket.join('general');

    socket.on('disconnect', () => {
      console.log('a user disconnected');
    });

    socket.on('chat message', (msg) => handleChatMessage(socket, msg));
    socket.on('room change', (room) => handleRoomChange(socket, room));
  });

  return server;
};

module.exports = socketSetup;
```

`server/app.js`, the changed lines only:

```js
const router = require('./router.js');
const socketSetup = require('./io.js');
// ...
router(app);
const server = socketSetup(app);

server.listen(port, (err) => {
  if (err) {
    throw err;
  }
  console.log(`Listening on port ${port}`);
});
```

`hosted/client.js`:

```js
const socket = io();

const handleEditBox = () => {
    const editForm = document.getElementById('editForm');
    const editBox = document.getElementById('editBox');

    editForm.addEventListener('submit', (e) => {
        e.preventDefault();

        if (editBox.value) {
            socket.emit('chat message', editBox.value);
            editBox.value = '';
        }
    });
};

const displayMessage = (msg) => {
    const messageDiv = document.createElement('div');
    messageDiv.textContent = msg;
    document.getElementById('messages').appendChild(messageDiv);
};

const handleChannelSelect = () => {
    const channelSelect = document.getElementById('channelSelect');
    const messages = document.getElementById('messages');

    channelSelect.addEventListener('change', () => {
        messages.innerHTML = '';
        socket.emit('room change', channelSelect.value);
    });
};

const init = () => {
    handleEditBox();
    socket.on('chat message', displayMessage);
    handleChannelSelect();
};

window.onload = init;
```

`views/index.handlebars`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Socket Chat!</title>
</head>
<body>
  <form id="editForm">
    <input id="editBox" type="text" />
    <select id="channelSelect">
      <option value="general">General</option>
      <option value="memes">Memes</option>
    </select>
    <input type="submit" />
  </form>

  <div id="messages"></div>

  <script src="/socket.io/socket.io.js"></script>
  <script src="/assets/client.js"></script>
</body>
</html>
```

## How the done repos differ

At Step 20 your code is plain `io.emit('chat message', msg)`, and neither done repo stops there. Both go a step further, and both have long comments that explain each piece.

**`basic-socket-io-done`** handles channels on the client, a version this page skips:

- The client sends the channel along, and the server uses it as the event name: `io.emit(msg.channel, msg.message)`.
- Changing the dropdown swaps which event the page listens to, with `socket.off` and `socket.on`.
- The problem with this version is the one from Step 23: every message still reaches every browser. Rooms fix that.
- `init` has a leftover `socket.on('testing', 'hello');`. It passes a string where a function belongs. Nothing emits `testing`, so it never runs. Don't copy that line.

**`socket-io-rooms-done`** is the same rooms code as Part 5, with a few small differences. All of them work:

- There's no `console.log(socket.rooms)`.
- It uses `innerText` instead of `textContent`.
- `handleEditBox` has an extra `channelSelect` it doesn't use, and a `return false`.
- The `<select>` has a `name`, which nothing reads.
- It's on Express 4 with `body-parser`, where the starter is on Express 5 with `express.urlencoded`.
- `init` starts with two leftover `getLeaderboardData` lines. The server never handles that event, so don't copy them.
