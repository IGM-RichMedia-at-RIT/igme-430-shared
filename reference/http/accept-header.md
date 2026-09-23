# The Accept Header (JSON or XML from the Same URL)

A client can tell the server what format it wants back by sending an `Accept` header. The server reads that header and picks a format. The URL stays the same, and only that one header changes what comes back. This is called content negotiation.

There are two headers here, and they go in opposite directions. It's easy to mix them up:

- **`Accept`** is on the *request*. The client is saying "here's what I can handle."
- **`Content-Type`** is on the *response*. The server is saying "here's what I actually sent you."

The server reads `Accept`. The client reads `Content-Type`.

## Quick example

This is the version we typed in class on 3B.

In `src/server.js`, inside `onRequest`, turn the header into an array:

```js
request.acceptedTypes = request.headers.accept ? request.headers.accept.split(',') : [];
```

Then in `src/responses.js`, check it and pick a format:

```js
const getCats = (request, response) => {
  const cat = { name: 'Captain Peanut-Butter', age: 7 };

  if (request.acceptedTypes[0] === 'application/xml') {
    let responseXML = '<response>';
    responseXML += `<name>${cat.name}</name>`;
    responseXML += `<age>${cat.age}</age>`;
    responseXML += '</response>';
    return respond(request, response, responseXML, 'application/xml');
  }

  return respond(request, response, JSON.stringify(cat), 'application/json');
};
```

On the client, `fetch` sends the header:

```js
fetch('/cats', {
  method: 'GET',
  headers: { Accept: 'application/xml' },
});
```

Anything that doesn't ask for XML first gets JSON. That includes a browser typing the URL into the address bar, and a request with no `Accept` header at all.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 3B (Wed 9/9) | Routing table, client buttons, `fetch`, reading `Accept`, the XML branch | Starter: [accept-header-example](https://github.com/IGM-RichMedia-at-RIT/accept-header-example) |
| | | Done: [accept-header-example-done](https://github.com/IGM-RichMedia-at-RIT/accept-header-example-done) |

Where to look in the done repo:

- `src/server.js`: `onRequest`, where `acceptedTypes` gets set
- `src/responses.js`: `getCats` (the format check) and `respond` (which also sets `Content-Length`)
- `client/client.html`: `sendFetchRequest` sends the header, `handleResponse` reads `Content-Type` and parses JSON or XML

The done repo is a little different from what we typed in class. It checks for `text/xml` instead of `application/xml` (both are real XML types, just be consistent on both sides). It also doesn't have the `: []` fallback, so it crashes on a request with no `Accept` header. The class version is the one to copy.

Video: [Austin, Week 3.1: Accept header](https://www.youtube.com/watch?v=ElramkPkvaA). It uses XHR instead of `fetch` on the client, but the server side is the same idea.

Used in: HTTP API Assignment, and again in HTTP API Assignment II.

## Common mistakes

- **The server crashes the moment something connects.** `request.headers.accept.split(',')` without a check crashes when a request has no `Accept` header, because `.split` doesn't exist on `undefined`. The `? ... : []` fallback fixes it.
- **Works from `fetch`, breaks when you type the URL in the browser.** The browser sends its own `Accept` header, something like `text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8`. That's why you split it into an array instead of comparing the whole string. It's also why anything that isn't a clear request for XML should fall back to JSON.
- **The button fires once when the page loads, then never again.** You wrote `addEventListener('click', sendFetchRequest('/cats', ...))`. The parentheses call the function right away and hook up whatever it returns, which is `undefined`. Wrap it in an arrow function like step 5.
- **"Cannot set headers after they are sent."** You're missing a `return` in front of a `respond` call, so the XML branch responds and then the JSON line tries to respond again. Put `return` in front of every `respond`.
- **The server ignores your header.** Header names are standardized: send `Accept`. Also check that the MIME type is spelled the same on both sides. `application/xml` and `text/xml` are different strings.
- **Only checking `[0]`.** Our code only looks at the first type in the list. That's fine for this class. A more careful server would look through the whole list for the first type it supports. If you do that, note that `split(',')` keeps spaces, so `'application/json, text/xml'` gives you `' text/xml'` with a leading space. `.trim()` each one.

## What you'll find on Google

- **Express's `res.format({ json: ..., xml: ... })` or `req.accepts('xml')`.** The Express version of this whole page. It handles the browser's messy header and the `q=` weights for you. Express comes later in the semester.
- **Libraries like `xml2js` or `xmlbuilder` to build XML.** We build it with strings because our data is tiny and it shows you exactly what's being sent. There's also no single automatic way to turn JSON into XML, because nothing tells the converter what the tags should be named.
- **Checking `Content-Type` on the request instead of `Accept`.** On a request, `Content-Type` describes a body the client is *sending* (see [Parsing a POST body](parsing-post-body.md)). It says nothing about what format the client wants back.
- **Using file extensions or query strings like `/cats.xml` or `/cats?format=xml`.** Real APIs do this, and it works. But our assignments grade the `Accept` header specifically, so use the header.

## Redo it

Start from a fresh copy of the [starter](https://github.com/IGM-RichMedia-at-RIT/accept-header-example). Run `npm install` once, then restart the server (`npm start`) after each server step. Client changes only need a browser refresh, but the server reads `client.html` once when it starts, so restart after those too.

### 1. Give `getCats` something to do

The starter exports `getCats` but never defines it, so the server won't even boot. In `src/responses.js`, above `module.exports`:

```js
const getCats = (request, response) => {
  const cat = { name: 'Captain Peanut-Butter', age: 7 };
  return respond(request, response, JSON.stringify(cat), 'application/json');
};
```

### 2. The routing table

In `src/server.js`, fill in `urlStruct`:

```js
const urlStruct = {
  '/': responseHandler.getIndex,
  '/cats': responseHandler.getCats,
  default: responseHandler.getIndex,
};
```

No parentheses after the function names. We want the functions themselves, not the result of calling them.

### 3. Look up the route

Fill in `onRequest`:

```js
const protocol = request.connection.encrypted ? 'https' : 'http';
const parsedUrl = new URL(request.url, `${protocol}://${request.headers.host}`);

const handler = urlStruct[parsedUrl.pathname];
if (handler) {
  handler(request, response);
} else {
  urlStruct.default(request, response);
}
```

Checkpoint: `http://localhost:3000/cats` shows the cat as JSON. Any other path shows the page.

### 4. The client sends a request

In `client/client.html`, fill in `sendFetchRequest`:

```js
const options = {
  method: 'GET',
  headers: { Accept: acceptedType },
};

fetch(url, options).then((response) => console.log(response));
```

### 5. Hook up the buttons

At the bottom of `init`:

```js
jsonButton.addEventListener('click', () => sendFetchRequest('/cats', 'application/json'));
xmlButton.addEventListener('click', () => sendFetchRequest('/cats', 'application/xml'));
```

Checkpoint: open the Network tab and click both buttons. Each one sends a request to `/cats` with a different `Accept` header, and both get JSON back. The server isn't listening to the header yet.

### 6. Read the header on the server

In `onRequest`, under the `parsedUrl` line:

```js
request.acceptedTypes = request.headers.accept ? request.headers.accept.split(',') : [];
```

`request` is just an object, so we can put our own properties on it for later functions to use.

### 7. The XML branch

In `getCats`, above the JSON `return`:

```js
if (request.acceptedTypes[0] === 'application/xml') {
  let responseXML = '<response>';
  responseXML += `<name>${cat.name}</name>`;
  responseXML += `<age>${cat.age}</age>`;
  responseXML += '</response>';
  return respond(request, response, responseXML, 'application/xml');
}
```

Checkpoint: in the Network tab, the XML button now gets XML and the JSON button still gets JSON. Same URL both times.

### 8. The client reads `Content-Type`

Swap the `console.log` in `sendFetchRequest` for a `handleResponse(response)` call, and write `handleResponse` above it. Check the response's `Content-Type` and parse the body based on that:

```js
const handleResponse = async (response) => {
  const text = await response.text();
  const type = response.headers.get('Content-Type');

  if (type === 'application/json') {
    console.log(JSON.parse(text));
  } else if (type === 'application/xml') {
    console.log(new DOMParser().parseFromString(text, 'application/xml'));
  }
};
```

The done repo's `handleResponse` does the same thing and puts the name and age on the page.
