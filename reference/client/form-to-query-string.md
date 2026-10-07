# Sending Form Input in a GET

A GET request has no body. So when a form needs to send something with a GET, like a name to search for, the value goes in the URL after a `?`:

```
/getUsers?name=jp
/getUsers?name=jp&age=40
```

Everything after the `?` is the query string. Each piece is `key=value`, and `&` goes between them.

You've already built this string. The POST form in 4B sent `name=jp&age=40` as its body. A GET form sends the same string on the end of the URL instead. And if you've used `fetch` with someone else's API before, you've built URLs like this. The only difference now is that the API on the other end is yours.

## Quick example

A form with one text box:

```html
<form id="searchForm" action="/getUsers" method="get">
  <label for="searchField">Name: </label>
  <input id="searchField" type="text" name="name" />
  <input type="submit" value="Find User" />
</form>
```

The client reads the box, builds the URL, and sends it:

```js
const sendGet = async (searchForm) => {
  const action = searchForm.getAttribute('action');
  const name = searchForm.querySelector('#searchField').value;

  const url = `${action}?name=${encodeURIComponent(name)}`;

  const response = await fetch(url, {
    method: 'GET',
    headers: {
      'Accept': 'application/json',
    },
  });

  handleResponse(response);
};
```

It hooks up to the form the same way the POST form did, with `preventDefault`:

```js
searchForm.addEventListener('submit', (e) => {
  e.preventDefault();
  sendGet(searchForm);
});
```

On the server, the value is waiting in `request.query`, from the line in `onRequest` that [Status codes and respondJSON](../http/status-codes.md) adds:

```js
request.query = Object.fromEntries(parsedUrl.searchParams);
```

So `/getUsers?name=jp` gives you `request.query.name === 'jp'`.

## GET form vs POST form

| | POST form (4B) | GET form (this page) |
|---|---|---|
| Where `name=jp&age=40` goes | the body | the URL, after the `?` |
| `Content-Type` header | yes, and the server checks it | no, there's no body to describe |
| Server reads it from | `request.body`, after `parseBody` | `request.query`, from `searchParams` |
| Use it for | adding or changing data | asking for data, including filtering it |

## What `encodeURIComponent` is for

Some characters already mean something in a URL. If someone types `Tom & Jerry`, the URL comes out as `?name=Tom & Jerry`, the `&` splits it, and the server gets `name` as `Tom `.

`encodeURIComponent('Tom & Jerry')` turns it into `Tom%20%26%20Jerry`, which is safe to put in a URL. `searchParams` turns it back into `Tom & Jerry` on the server, so there's nothing to decode yourself.

## Where we did this

We built the POST version of this form in class. The GET version uses the same pieces with the data moved into the URL, which is what a Project 1 filtering endpoint needs.

| Day | What we covered | Code |
|---|---|---|
| 3C (Fri 9/11) | Reading the query string on the server with `searchParams` and `request.query` | Done: [status-code-example-done](https://github.com/IGM-RichMedia-at-RIT/status-code-example-done) |
| 4A (Mon 9/14) | A form that sends a `fetch` and stops its own submit with `preventDefault` | Done: [head-request-example-done](https://github.com/IGM-RichMedia-at-RIT/head-request-example-done) |
| 4B and 4C (Wed 9/16, Fri 9/18) | Reading input values and building `name=...&age=...` for a POST body | Done: [body-parse-example-done](https://github.com/IGM-RichMedia-at-RIT/body-parse-example-done) |

Where to look:

- `status-code-example-done`, `src/server.js`: the `request.query` line in `onRequest`
- `body-parse-example-done`, `client/client.html`: `sendPost` reads each box with `.value` and builds the string. Your GET version does the same, then puts the string after the `?` instead of in `body`

Used in: Project 1 (at least one GET endpoint with query parameters, and a main page where people can use it without writing code).

## Common mistakes

- **The server always gets an empty name.** You read `.value` in `init` or at the top of the script, when the page first loaded and the box was still empty. Read it inside the function the submit handler calls, so you get what's in the box when they press the button.
- **The URL looks right, but `request.query.name` is `undefined`.** The key in your URL and the key the server looks for don't match. `id="searchField"` is only how `querySelector` finds the box. The key is whatever you write before the `=` in your URL string, and that's the name the server has to ask for.
- **`Cannot read properties of undefined (reading 'name')`.** `request.query` was never set. The body-parse repos don't have that line, so add it from [Status codes, step 1](../http/status-codes.md#1-read-the-query-string).
- **The page turns into raw JSON, and the address bar says `/getUsers?name=jp`.** That's the form's built-in submit. The browser built the query string itself, from each input's `name` attribute. Your `fetch` does that step by hand. Add `e.preventDefault()`.
- **Two question marks.** `/getUsers?name=jp?age=40` is wrong. Only the first pair gets a `?`, and every pair after it starts with `&`.
- **An empty box sends `?name=`.** The server gets `request.query.name === ''`. Decide what an empty filter means for your API (send everything, or send a 400) and handle it. An empty string is falsy, so `if (request.query.name)` treats it the same as missing.
- **Numbers arrive as strings.** `?age=40` gives you `'40'`, the same as a URL-encoded POST body. Convert it with `Number(...)` before you compare.
- **Filtering on the client.** Fetching everything and filtering it in the browser works, but Project 1 says the API does the filtering. The query string is how the client tells the server what to filter by.

## What you'll find on Google

- **`new URLSearchParams({ name, age }).toString()`.** Builds `name=jp&age=40` and encodes it for you. Nice once you have more than one or two keys. This page uses a template string because it's the same one you already wrote for the POST body.
- **`new URLSearchParams(new FormData(form))`.** Reads every input in the form by its `name` attribute and builds the whole query string in one line. It works, as long as every input has the right `name`.
- **`axios.get(url, { params: { name } })`.** Same request, different library. We use `fetch`, which is built into the browser.
- **`req.query.name` in Express.** Express does the `searchParams` line for you. It comes after Project 1, and Project 1 doesn't allow it.
- **Route parameters like `/users/jp`** instead of `/users?name=jp`. Real APIs use both. A route parameter usually names one specific thing, and a query parameter filters or limits a list. Our servers match exact paths, so query parameters are the easier fit for Project 1.

## Try it

Start from a copy of [body-parse-example-done](https://github.com/IGM-RichMedia-at-RIT/body-parse-example-done). Run `npm install` once, then restart the server (`npm start`) after each step. The server reads `client.html` once when it starts, so restart after client changes too. `users` lives in memory, so every restart empties it. Add a user or two with the POST form before you test.

### 1. Read the query string on the server

In `src/server.js`, in `onRequest`, under the `parsedUrl` line:

```js
request.query = Object.fromEntries(parsedUrl.searchParams);
```

Then, as the first line of `getUsers` in `src/jsonResponses.js`:

```js
console.log(request.query);
```

Checkpoint: visit `localhost:3000/getUsers?name=jp`. The terminal prints `{ name: 'jp' }`.

### 2. Add a GET form

In `client/client.html`, under the closing `</form>` of `nameForm`:

```html
<form id="searchForm" action="/getUsers" method="get">
  <label for="searchField">Name: </label>
  <input id="searchField" type="text" name="name" />
  <input type="submit" value="Find User" />
</form>
```

Checkpoint: type a name and press Find User. The page turns into JSON, and the address bar shows `/getUsers?name=` and what you typed. The browser built that URL for you from the input's `name` attribute. The next two steps get `fetch` to send the same request without leaving the page.

### 3. Send it with `fetch`

In the script, under `sendPost`:

```js
const sendGet = async (searchForm) => {
  const action = searchForm.getAttribute('action');
  const name = searchForm.querySelector('#searchField').value;

  const url = `${action}?name=${encodeURIComponent(name)}`;

  const response = await fetch(url, {
    method: 'GET',
    headers: {
      'Accept': 'application/json',
    },
  });

  handleResponse(response);
};
```

### 4. Hook up the form

In `init`, under the `nameForm.addEventListener` line:

```js
const searchForm = document.querySelector('#searchForm');

searchForm.addEventListener('submit', (e) => {
  e.preventDefault();
  sendGet(searchForm);
});
```

Checkpoint: press Find User. The page stays put and shows Success. In the Network tab, the request URL ends in `?name=` and what you typed, and the terminal prints it again from step 1.

### 5. Your turn: make `getUsers` use it

No code for this one. It's the part Project 1 is actually about. Right now `getUsers` ignores `request.query` and sends everyone back. Change it so that:

- no `name` in the query sends every user, like it does now
- a `name` that exists sends just that user
- a `name` that doesn't exist sends a 404 (see [Status codes and respondJSON](../http/status-codes.md)), and `handleResponse` gets a `case 404`, since it doesn't have one yet

Then add an age box to the search form and send both, joined with `&`.

In Project 1 the shape is the same, with your data. Decide which keys your endpoint takes, do the filtering on the server, and list each key on your documentation page with its type and whether it's required.
