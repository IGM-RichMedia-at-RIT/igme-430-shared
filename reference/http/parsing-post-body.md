# Parsing a POST Body

With a GET request, everything you need comes in the URL. A POST sends its data in the request body. Node doesn't hand you the body all at once. It arrives in pieces (chunks), and you have to collect them, glue them back together, and then parse the result based on its `Content-Type`.

This is the same idea as the streaming media assignment, pointed the other way. That time the server streamed to the client. Now the client streams to us.

## Quick example

From `src/server.js` in [body-parse-example-done](https://github.com/IGM-RichMedia-at-RIT/body-parse-example-done/blob/master/src/server.js):

```js
const query = require('querystring');

const parseBody = (request, response, handler) => {
  const body = [];

  request.on('error', (err) => {
    console.dir(err);
    response.statusCode = 400;
    response.end();
  });

  request.on('data', (chunk) => {
    body.push(chunk);
  });

  request.on('end', () => {
    const bodyString = Buffer.concat(body).toString();
    const type = request.headers['content-type'];
    if (type === 'application/x-www-form-urlencoded') {
      request.body = query.parse(bodyString);
    } else if (type === 'application/json') {
      request.body = JSON.parse(bodyString);
    } else {
      response.writeHead(400, { 'Content-Type': 'application/json' });
      response.write(JSON.stringify({ error: 'invalid data format' }));
      return response.end();
    }

    handler(request, response);
  });
};
```

Then any POST route calls it and passes in the function to run once the body is ready:

```js
const handlePost = (request, response, parsedUrl) => {
  if (parsedUrl.pathname === '/addUser') {
    parseBody(request, response, jsonHandler.addUser);
  }
};
```

By the time `addUser` runs, `request.body` is a normal object: `{ name: 'jp', age: '40' }`.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 4B (Wed 9/16) | POST routing, `parseBody`, URL-encoded bodies, `addUser`, 201 vs 204 | Starter: [body-parse-example](https://github.com/IGM-RichMedia-at-RIT/body-parse-example) |
| 4C (Fri 9/18) | Finished the client, then JSON bodies (the `content-type` check) | Done: [body-parse-example-done](https://github.com/IGM-RichMedia-at-RIT/body-parse-example-done) |

Where to look in the done repo:

- `src/server.js`: `parseBody`, `handlePost`, and the method check in `onRequest`
- `src/jsonResponses.js`: `addUser`, and the 204 check in `respondJSON`
- `client/client.html`: `sendPost`, which is the `fetch` that sends the body
- The JSON support was added in its own commit, [c7b9afb "Added json support"](https://github.com/IGM-RichMedia-at-RIT/body-parse-example-done/commit/c7b9afb). Clicking that shows you exactly what changed.

Video: [Austin, Week 4.2: POST Requests and Body Parsing](https://www.youtube.com/watch?v=QY5sBCg6Ksg). It's older than our code: the client uses XHR instead of `fetch`, and it stops before JSON bodies. The server side is the same idea.

Used in: HTTP API Assignment II.

## Common mistakes

- **The request hangs forever** (curl just sits there, the browser spins). `handler(request, response)` is missing from the `end` listener, or the POST never reaches `handlePost` because `onRequest` doesn't check the method.
- **`Cannot destructure property 'name' of 'request.body'`**, then the server exits and curl says `Empty reply from server`. You collected the body but never assigned `request.body`. This happened in class on 4B.
- **Every POST from your page gets a 400 "invalid data format".** Your `fetch` isn't sending the `Content-Type` the server is checking for. If you pass a string as `body` and don't set the header yourself, `fetch` sends `text/plain;charset=UTF-8`. Also, the server check is an exact match, so `application/json; charset=utf-8` won't match `application/json`.
- **Sending JSON that isn't valid JSON crashes the server.** `JSON.parse` throws on bad input and our class code doesn't catch it. Fine for the demo. Worth a `try/catch` that sends a 400 if you want your assignment to hold up.
- **You made a change and nothing happened.** Plain `node` doesn't reload. Stop the server and start it again (nodemon, from Week 5, does this for you).
- **The 204 still has a body.** Node quietly drops it, so nothing looks broken, but add the `status !== 204` check anyway. It's what the status code means.

## What you'll find on Google

- **`app.use(express.json())` or `body-parser`.** This is the Express way, and it does everything on this page in one line. That's exactly why we write it by hand first: it's what those libraries are doing for you. Express comes later in the semester.
- **`body += chunk` instead of an array and `Buffer.concat`.** Works fine for plain English text. It can garble characters that take more than one byte (accents, emoji) when they get split across two chunks. The array version doesn't have that problem.
- **`new URLSearchParams(bodyString)` instead of `querystring`.** A newer built-in way to parse URL-encoded data. Either works. We use `querystring` because it gives you a plain object directly.
- **Examples that use `axios` or jQuery `$.post` on the client.** Same request, different library. We use `fetch`, which is built into the browser.

## Redo it

Start from a fresh copy of the [starter](https://github.com/IGM-RichMedia-at-RIT/body-parse-example). `handlePost`, `onRequest`, and `addUser` are empty. Everything else is already there. Restart the server after each step (`npm start`).

The checkpoints use `curl` from a second terminal. `-i` shows the status line and headers.

### 1. Route by method

In `onRequest`, under the `parsedUrl` line:

```js
if (request.method === 'POST') {
  handlePost(request, response, parsedUrl);
} else {
  handleGet(request, response, parsedUrl);
}
```

Checkpoint: the page at `http://localhost:3000` still loads.

### 2. Send `/addUser` to `parseBody`

Fill in `handlePost`:

```js
if (parsedUrl.pathname === '/addUser') {
  parseBody(request, response, jsonHandler.addUser);
}
```

`parseBody` doesn't exist yet. That's the next step.

### 3. Collect the chunks

Above `handlePost`, start `parseBody`:

```js
const parseBody = (request, response, handler) => {
  const body = [];

  request.on('error', (err) => {
    console.dir(err);
    response.statusCode = 400;
    response.end();
  });

  request.on('data', (chunk) => {
    body.push(chunk);
  });
};
```

Nothing runs `handler` yet, so a POST will just hang. That's expected.

### 4. When it ends, parse it and hand it off

Still inside `parseBody`, after the `data` listener:

```js
request.on('end', () => {
  const bodyString = Buffer.concat(body).toString();
  request.body = query.parse(bodyString);

  handler(request, response);
});
```

Both of the last two lines matter. Leave out `handler(...)` and the request hangs forever. Leave out `request.body = ...` and `addUser` crashes (see Common mistakes).

### 5. `addUser`: reject missing fields

In `src/jsonResponses.js`, fill in `addUser`:

```js
const responseJSON = {
  message: 'Name and age are both required.',
};

const { name, age } = request.body;

if (!name || !age) {
  responseJSON.id = 'missingParams';
  return respondJSON(request, response, 400, responseJSON);
}
```

Checkpoint:

```
curl -i -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "name=jp" localhost:3000/addUser
```

You should get a `400` and the "both required" message.

### 6. `addUser`: create or update

Below the 400 check:

```js
let responseCode = 204;

if (!users[name]) {
  responseCode = 201;
  users[name] = { name: name };
}

users[name].age = age;
```

Then finish the function:

```js
if (responseCode === 201) {
  responseJSON.message = 'Created Successfully';
  return respondJSON(request, response, responseCode, responseJSON);
}

return respondJSON(request, response, responseCode, {});
```

A new name is a `201 Created`. A name we already have is an update, which is a `204 No Content`.

### 7. 204 gets no body

In `respondJSON`, change the check around `response.write` so a 204 doesn't write a body either:

```js
if (request.method !== 'HEAD' && status !== 204) {
  response.write(content);
}
```

Checkpoint: run this twice.

```
curl -i -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "name=jp&age=40" localhost:3000/addUser
```

The first time you get `201` with a message. The second time you get `204` with nothing after the headers. `curl localhost:3000/getUsers` should show the user.

### 8. Accept JSON too

Go back to the `end` listener in `parseBody` and replace the `request.body = query.parse(bodyString);` line with a check on the `Content-Type`:

```js
const type = request.headers['content-type'];
if (type === 'application/x-www-form-urlencoded') {
  request.body = query.parse(bodyString);
} else if (type === 'application/json') {
  request.body = JSON.parse(bodyString);
} else {
  response.writeHead(400, { 'Content-Type': 'application/json' });
  response.write(JSON.stringify({ error: 'invalid data format' }));
  return response.end();
}
```

Checkpoint:

```
curl -i -X POST -H "Content-Type: application/json" -d '{"name":"jp","age":41}' localhost:3000/addUser
```

That's a `204`, since `jp` already exists. Send the same thing with `-H "Content-Type: text/plain"` and you'll get a `400`.

### The client side

The server is done at this point. The page sends the POST from `sendPost` in `client/client.html`. The part that matters is the `fetch`:

```js
const response = await fetch(url, {
  method: method,
  headers: {
    'Content-Type': dataType,
    'Content-Length': formData.length,
    'Accept': 'application/json',
  },
  body: formData,
});
```

`dataType` comes from the dropdown on the page, and `formData` is either `name=jp&age=40` or the `JSON.stringify` version. The `Content-Type` you send here is what the server's `if` in step 8 checks, so the two sides have to agree. Compare your `client.html` with the done repo's for the rest (the form listener and `handleResponse`).
