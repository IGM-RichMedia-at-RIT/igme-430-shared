# HEAD Requests

A HEAD request is a GET with the body cut off. The server sends back the same status code and the same headers, including `Content-Length`, but no content.

It lets a client ask "what would I get, and how big is it?" without downloading anything. Google Drive can list a folder of files with their sizes without opening a single one of them, and a video player can check how big a file is before it starts pulling it down. HEAD is also how a client checks whether something exists at all: a 404 on a HEAD costs almost nothing.

## Quick example

The only HEAD-specific code in our whole server is one `if` in `respondJSON`, from `src/jsonResponses.js`:

```js
const respondJSON = (request, response, status, object) => {
  const content = JSON.stringify(object);

  const headers = {
    'Content-Type': 'application/json',
    'Content-Length': Buffer.byteLength(content, 'utf8'),
  };

  response.writeHead(status, headers);

  if (request.method !== 'HEAD') {
    response.write(content);
  }

  response.end();
};
```

Build the full response like it's a GET, then just don't write the body. Since every endpoint goes through `respondJSON`, every endpoint supports HEAD for free.

`Content-Length` is still the real size of the body a GET would get, not zero. The size is the whole point of asking. `Buffer.byteLength` counts bytes rather than characters, which is what the header wants.

You can see the difference in a terminal. `curl -i` sends a GET and shows the headers, and `curl -I` sends a HEAD:

```
curl -i localhost:3000/getUsers          curl -I localhost:3000/getUsers

HTTP/1.1 200 OK                          HTTP/1.1 200 OK
Content-Type: application/json           Content-Type: application/json
Content-Length: 12                       Content-Length: 12

{"users":{}}                             (nothing)
```

Same status, same headers, same length. Only one has a body.

On the client, `fetch` can send a HEAD by setting the method. There's no body in the response, so don't call `.json()` on it:

```js
const response = await fetch('/getUsers', { method: 'HEAD' });

console.log(response.status);                        // 200
console.log(response.headers.get('Content-Length')); // the size a GET would send
```

## GET vs HEAD

| | GET | HEAD |
|---|---|---|
| Status code | yes | the same one |
| `Content-Type` | yes | the same one |
| `Content-Length` | size of the body | the same size, even though no body comes |
| Body | yes | no |

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 4A (Mon 9/14) | `respondJSON` with the HEAD check, `getUsers`, `updateUser`, serving `style.css`, the client form and `preventDefault` | Starter: [head-request-class-example](https://github.com/IGM-RichMedia-at-RIT/head-request-class-example) |
| | | Done: [head-request-example-done](https://github.com/IGM-RichMedia-at-RIT/head-request-example-done) |

Where to look in the done repo:

- `src/jsonResponses.js`: `respondJSON` (the HEAD check), `getUsers`, `updateUser`
- `src/server.js`: the routing table, with `/getUsers`, `/updateUser`, and `/style.css` added
- `src/htmlResponses.js`: `getCSS`
- `client/client.html`: `requestUpdate` sends the request, `init` stops the form with `preventDefault`, and `handleResponse` only parses the body when the method was GET

What we typed in class matches the done repo apart from formatting.

Two days later, `respondJSON` also learns to skip the body on a 204. That version is in [Parsing a POST body](parsing-post-body.md). For how the status codes themselves work, see [Status codes and respondJSON](status-codes.md).

Video: [Austin, Week 4.1: HEAD Requests](https://www.youtube.com/watch?v=DPkIjyjVHTs). It's older than our code. The client uses XHR instead of `fetch`, the server uses separate handlers for HEAD (`getUsersMeta`, `notFoundMeta`) instead of one `if`, and it doesn't set `Content-Length`. It also says writing a body on a HEAD throws an error, which isn't true in Node today (see Common mistakes). The idea is the same, but copy the one-`if` version.

Used in: HTTP API Assignment II (the rubric checks `/getUsers` and `/notReal` with HEAD, and the client can't parse JSON on a HEAD) and Project 1 (HEAD support on your GET endpoints).

## Common mistakes

- **`Content-Length: 0` on a HEAD.** No body doesn't mean length zero. Send the length the GET would have. If you build the headers once in `respondJSON` like our code, you get this right automatically.
- **`Unexpected end of JSON input` on the client.** You called `response.json()` on a HEAD response, and there's nothing to parse. Our `handleResponse` takes a `parseResponse` flag and only parses when the method was GET.
- **You picked HEAD, pressed Send, and the whole page turned into raw JSON.** The form did its own built-in submit, which is a GET to `/getUsers`, and the browser navigated to the result. Your `fetch` ran too, but the navigation wins. Add `e.preventDefault()` in the submit handler.
- **You deleted the `if` and nothing broke.** Node quietly drops a body written on a HEAD, so it doesn't crash. Keep the `if` anyway. It's you following what HEAD means.
- **You can't test HEAD by typing the URL in the browser.** The address bar always sends a GET. Use `curl -I`, `fetch`, or the dropdown on the class page.
- **`response.writeHead is not a function`.** Your parameters are in the wrong order. Node always passes `(request, response)`, in that order, no matter what you name them.
- **Users disappear.** `users` lives in memory, so restarting the server wipes it. We get a database later in the semester.

## What you'll find on Google

- **Express answers HEAD for you.** An Express `app.get()` route also responds to HEAD with the body stripped, so you rarely see HEAD code in Express examples. That's the same thing our `if` does by hand. Express comes later in the semester.
- **Separate handlers or a routing table per method** (`{ GET: {...}, HEAD: {...} }`, `getUsersMeta`). Austin's older video does this. It works, but it's more code to keep in sync than one `if`.
- **`curl -X HEAD`.** It looks like it should work, but curl still waits for a body that never comes, so it hangs until it times out. curl even warns you about it. Use `curl -I`.
- **`XMLHttpRequest` and `xhr.getResponseHeader(...)`.** The older way to make the request. With `fetch`, it's `response.headers.get(...)`.

## Redo it

Start from a fresh copy of the [starter](https://github.com/IGM-RichMedia-at-RIT/head-request-class-example). `respondJSON`, `getUsers`, and `updateUser` are empty. `notFound` is written, but it calls the empty `respondJSON`, so right now `/getUsers` (and every other unknown URL) spins forever. Run `npm install` once, then restart the server (`npm start`) after each step. The server reads `client.html` once when it starts, so restart after client changes too.

### 1. `respondJSON`, as a normal GET

In `src/jsonResponses.js`, fill in the empty `respondJSON`:

```js
const content = JSON.stringify(object);

const headers = {
  'Content-Type': 'application/json',
  'Content-Length': Buffer.byteLength(content, 'utf8'),
};

response.writeHead(status, headers);
response.write(content);
response.end();
```

Checkpoint: `localhost:3000/anything` shows the "not found" JSON instead of spinning.

### 2. Skip the body on HEAD

Replace the `response.write(content);` line with:

```js
if (request.method !== 'HEAD') {
  response.write(content);
}
```

Checkpoint: `curl -I localhost:3000/notReal` shows `404 Not Found` and `Content-Length: 73`, with no body.

### 3. `getUsers`

Fill in the empty `getUsers`:

```js
const responseJSON = {
  users,
};

return respondJSON(request, response, 200, responseJSON);
```

Then in `src/server.js`, add it to `urlStruct` under the `'/'` line:

```js
'/getUsers': jsonHandler.getUsers,
```

Checkpoint: run `curl -i localhost:3000/getUsers` and then `curl -I localhost:3000/getUsers`. Both are `200` with `Content-Length: 12`, and only the first one has `{"users":{}}` after the headers.

### 4. `updateUser`

This is a GET that changes data, which GET should never do. We did it this way in class to keep things simple, and fixed it with POST two days later. Fill in `updateUser`:

```js
const newUser = {
  createdAt: Date.now(),
};

users[newUser.createdAt] = newUser;

return respondJSON(request, response, 201, newUser);
```

And add the route to `urlStruct`:

```js
'/updateUser': jsonHandler.updateUser,
```

Checkpoint: click the `/updateUser` link on the page twice (hit Back in between), then run `curl -I localhost:3000/getUsers`. `Content-Length` went from 12 to 99. HEAD told you the data grew without sending any of it. Restart the server and it's back to empty.

### 5. Serve the CSS

The page loads unstyled because the browser asks for `/style.css` and the server doesn't know that route. In `src/htmlResponses.js`, under the `index` line:

```js
const css = fs.readFileSync(`${__dirname}/../client/style.css`);
```

Add a function above `module.exports`, and add `getCSS` to the exports:

```js
const getCSS = (request, response) => {
  response.writeHead(200, { 'Content-Type': 'text/css' });
  response.write(css);
  response.end();
};
```

Then add the route in `urlStruct`:

```js
'/style.css': htmlHandler.getCSS,
```

Checkpoint: the page is styled, and the Network tab shows `style.css` as a `200`.

### 6. The client sends the request

In `client/client.html`, add `async` to `requestUpdate` and add this under the two lines that read the dropdowns:

```js
const response = await fetch(url, {
  method,
  headers: {
    'Accept': 'application/json',
  },
});

handleResponse(response, method === 'get');
```

`method,` is shorthand for `method: method`. The second argument to `handleResponse` is `true` only for a GET, which is how it knows whether there's a body to parse.

### 7. Hook up the form

In `init`, under the `userForm` line:

```js
const getUsers = (e) => {
  e.preventDefault();
  requestUpdate(userForm);
  return false;
};

userForm.addEventListener('submit', getUsers);
```

`preventDefault` stops the form's built-in submit. Try it without that line once: pick HEAD, press Send, and the page gets replaced by JSON. `return false` is an old habit and you don't really need it.

Checkpoint: pick GET and press Send. The page stays put and shows Success.

### 8. Only parse the body when there is one

Add `async` to `handleResponse`, then add this after the `switch`:

```js
if (parseResponse) {
  const obj = await response.json();
  content.innerHTML += `<p>${JSON.stringify(obj)}</p>`;
} else {
  content.innerHTML += '<p>Meta Data Received</p>';
}
```

Checkpoint: try all four combinations.

| URL | Method | Page shows |
|---|---|---|
| `/getUsers` | GET | Success and the users JSON |
| `/getUsers` | HEAD | Success and "Meta Data Received" |
| `/notReal` | GET | Resource Not Found and the message JSON |
| `/notReal` | HEAD | Resource Not Found and "Meta Data Received" |

In the Network tab, click a HEAD request: `Content-Length` is set, and the Response tab is empty.
