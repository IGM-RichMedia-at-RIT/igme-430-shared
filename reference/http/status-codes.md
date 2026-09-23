# Status Codes and respondJSON

Every response starts with a three-digit status code. The client checks it first, before it looks at the body. It tells the client whether things worked and, if they didn't, whose fault it was.

The pattern you'll use on almost every endpoint this semester:

| They ask for | Do you have it? | You answer |
|---|---|---|
| nothing, or they left out something required | n/a | **400** Bad Request |
| something specific | yes | **200** OK, plus the data |
| something specific | no | **404** Not Found |

Short version: bad input is a 400, a missing resource is a 404, and everything else is a 200.

## Quick example

The helper every JSON response goes through, from `src/jsonResponses.js`:

```js
const respondJSON = (request, response, status, object) => {
  const content = JSON.stringify(object);
  response.writeHead(status, {
    'Content-Type': 'application/json',
    'Content-Length': Buffer.byteLength(content, 'utf8'),
  });
  response.write(content);
  response.end();
};
```

A handler decides the status and the message, then hands off to it:

```js
const badRequest = (request, response) => {
  const responseJSON = {
    message: 'This request has the required parameters',
  };

  if (!request.query.valid || request.query.valid !== 'true') {
    responseJSON.message = 'Missing valid query parameter set to true';
    responseJSON.id = 'badRequest';
    return respondJSON(request, response, 400, responseJSON);
  }

  return respondJSON(request, response, 200, responseJSON);
};
```

`request.query` comes from one line in `onRequest` in `src/server.js`:

```js
request.query = Object.fromEntries(parsedUrl.searchParams);
```

So `/badRequest?valid=true` gives you `request.query.valid === 'true'`.

On the client, check `response.status` before you use the body:

```js
const response = await fetch('/badRequest');
if (response.status === 400) {
  // show the error
}
const obj = await response.json();
```

## The codes we use

| Code | Name | When we use it |
|---|---|---|
| 200 | OK | It worked, here's the data |
| 201 | Created | A POST made something new ([Parsing a POST body](parsing-post-body.md)) |
| 204 | No Content | It worked, and there's nothing to send back (an update) |
| 206 | Partial Content | Here's the piece of the file you asked for (streaming media) |
| 400 | Bad Request | The client sent something wrong or left something out |
| 401 | Unauthorized | You need to log in first |
| 403 | Forbidden | You're logged in, but you aren't allowed to do this |
| 404 | Not Found | That doesn't exist |
| 500 | Internal Server Error | The server broke |
| 501 | Not Implemented | The server doesn't support that yet |

The first digit tells you the family: 2xx worked, 4xx is the client's fault, 5xx is the server's fault. [Wikipedia's list](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) is the one we looked at in class.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 3C (Fri 9/11) | Status code families, query parameters, 400 vs 404, the client reading `response.status` | Starter: [status-code-example](https://github.com/IGM-RichMedia-at-RIT/status-code-example) |
| | | Done: [status-code-example-done](https://github.com/IGM-RichMedia-at-RIT/status-code-example-done) |

Where to look in the done repo:

- `src/server.js`: the `request.query` line in `onRequest`, and the `notFound` fallback in the routing table
- `src/jsonResponses.js`: `respondJSON`, `badRequest`, `notFound`
- `client/client.html`: `sendFetch` and `handleResponse`

`respondJSON` keeps growing after this day. In 4A it learns to skip the body on a HEAD request, and in 4B it skips the body on a 204. The version in [body-parse-example-done](https://github.com/IGM-RichMedia-at-RIT/body-parse-example-done/blob/master/src/jsonResponses.js) has both.

Video: [Austin, Week 3.2: Status Codes](https://www.youtube.com/watch?v=vHSb7GjmMxA). It uses XHR on the client instead of `fetch`, but the server side is the same idea.

Used in: the Streaming Media assignment (206), HTTP API Assignment, HTTP API Assignment II, and both projects.

## Common mistakes

- **Using 400 when it should be 404.** If they asked for a user named `bob` and you don't have one, nothing was wrong with the request. It's a 404. Save 400 for when the request itself is missing or malformed.
- **"Cannot set headers after they are sent."** Two `respondJSON` calls ran for one request, usually because the 400 branch is missing its `return` and the code falls through to the 200. Put `return` in front of every `respondJSON`.
- **A route that never answers.** The handler runs but never calls `respondJSON`, so the request hangs. This is what the starter does before you fill in `badRequest` and `notFound`.
- **`?valid=true` still gives a 400.** Query values are always strings. Compare to `'true'`, not `true`.
- **Your client shows "success" for a 404.** `fetch` only fails when it can't reach the server at all. A 404 or a 500 still counts as a response, so the promise resolves normally. Always check `response.status` (or `response.ok`, which is true for any 2xx).
- **`response.json()` throws on a 204.** A 204 has no body, so there's nothing to parse. Check for 204 and stop before calling `.json()`. The client in `body-parse-example-done` does this.

## What you'll find on Google

- **`res.status(404).json({ ... })`.** The Express version of `respondJSON`, all in one line. Express comes later in the semester.
- **`if (!response.ok) throw new Error(...)` with a `try/catch`.** Common in fetch tutorials, and it works. We use a `switch` on `response.status` because our assignments need a different message for each code.
- **Packages like `http-status-codes`** that give you names instead of numbers (`StatusCodes.NOT_FOUND`). Nice in big projects. Plain numbers are fine here.
- **`url.parse(request.url, true).query` or `require('querystring')` for query strings.** Older ways to do the same thing. `url.parse` is deprecated. Use `new URL(...)` and `searchParams` like our code does.

## Redo it

Start from a fresh copy of the [starter](https://github.com/IGM-RichMedia-at-RIT/status-code-example). The routing table and `respondJSON` are already there. `success` already works. `badRequest` and `notFound` build a message but never send it, and the client functions are empty. Run `npm install` once, then restart the server (`npm start`) after each step.

### 1. Read the query string

In `src/server.js`, in `onRequest`, under the `parsedUrl` line:

```js
request.query = Object.fromEntries(parsedUrl.searchParams);
```

`parsedUrl.searchParams` holds everything after the `?`. `Object.fromEntries` turns it into a plain object.

### 2. `badRequest`: the 400

In `src/jsonResponses.js`, at the bottom of `badRequest`, under the `responseJSON` it already has:

```js
if (!request.query.valid || request.query.valid !== 'true') {
  responseJSON.message = 'Missing valid query parameter set to true';
  responseJSON.id = 'badRequest';
  return respondJSON(request, response, 400, responseJSON);
}
```

### 3. `badRequest`: the 200

Right after that `if` block:

```js
return respondJSON(request, response, 200, responseJSON);
```

Checkpoint: in the browser, `localhost:3000/badRequest` shows the 400 message and `localhost:3000/badRequest?valid=true` shows the 200 message. The Network tab shows the status codes.

### 4. `notFound`: the 404

At the bottom of `notFound`:

```js
return respondJSON(request, response, 404, responseJSON);
```

Checkpoint: `localhost:3000/anything` gives a 404 with the "not found" message.

### 5. The client sends the request

In `client/client.html`, fill in `sendFetch` and make it `async`:

```js
const sendFetch = async (url) => {
  const response = await fetch(url);
  handleResponse(response);
};
```

The buttons in `init` are already wired to it.

### 6. The client checks the status

Fill in `handleResponse` and make it `async` too:

```js
const handleResponse = async (response) => {
  const content = document.getElementById('content');

  switch (response.status) {
    case 200:
      content.innerHTML = '<b>Success</b>';
      break;
    case 400:
      content.innerHTML = '<b>Bad Request</b>';
      break;
    case 404:
      content.innerHTML = '<b>Not Found</b>';
      break;
    default:
      content.innerHTML = '<p>Status Code not Implemented By Client</p>';
      break;
  }
```

### 7. Then read the body

Still in `handleResponse`, after the `switch`:

```js
  const resObj = await response.json();
  content.innerHTML += `<p>${resObj.message}</p>`;
};
```

Checkpoint: each of the three buttons shows its own heading and the message from the server.
