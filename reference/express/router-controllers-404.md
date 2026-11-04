# Express: routes, controllers, and the 404

Once `src/app.js` is set up ([the app and middleware](app-and-middleware.md)), the last line before `listen` is `router(app)`. That function lives in `src/router.js` and does one job: map each URL and method to a function. The functions live in `src/controllers/`. This is the `urlStruct` and `onRequest` from Project 1, split across two files, with Express running the loop.

The folder map, which you'll see again in every DomoMaker:

```
src/app.js                 setup: middleware, then router(app), then listen
src/router.js              which URL and method goes to which controller
src/controllers/index.js   the C in MVC: the functions that answer requests
views/                     the V: pages (templates, from Handlebars on)
src/models/                the M: comes with the database
```

The controllers folder holds one file per thing, not per page. Today everything fits in `index.js`. In DomoMaker it has `Account.js` and `Domo.js`, and `index.js` just re-exports them, so the router says `controllers.Account.signup`. Views are on [Handlebars templates](handlebars-templates.md). Models are on [MongoDB and Mongoose](../database/mongoose-connect-schema-save.md).

## Quick example

From `src/router.js` in [simple-mvc-controllers-done](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-controllers-done/blob/master/src/router.js), comments trimmed:

```js
const controllers = require('./controllers');

const router = (app) => {
  app.get('/', controllers.index);
  app.get('/page1', controllers.page1);
  app.get('/page2', controllers.page2);
  app.get('/getName', controllers.getName);

  app.get('/*wild', controllers.notFound);
  app.post('/setName', controllers.setName);
};

module.exports = router;
```

And the two controllers that aren't just `sendFile`, from [`src/controllers/index.js`](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-controllers-done/blob/master/src/controllers/index.js):

```js
const notFound = (req, res) => {
  return res.status(404).sendFile(path.resolve(`${__dirname}/../../views/notFound.html`));
};

const getName = (req, res) => {
  return res.json({ name });
};
```

`require('./controllers')` loads a folder. Node looks for `index.js` inside it. Whatever that file puts in `module.exports` is what the router can see, so a controller that isn't exported doesn't exist as far as the router is concerned.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 8B (Wed 10/14) | `router(app)`, three `app.get` routes, the crash when nothing is exported, the free HEAD, `notFound` and the `'/*wild'` wildcard, then `res.json` with `getName` and `setName` | Starter: [simple-mvc-controllers](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-controllers) · Done: [simple-mvc-controllers-done](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-controllers-done) |

Where to look in the done repo:

- `src/router.js`: the whole file. The long comment above the wildcard line explains route order.
- `src/controllers/index.js`: `hostIndex`, `hostPage1`, `hostPage2` (all `sendFile`), `notFound` (chaining `status` and `sendFile`), `getName` (`res.json`), `setName` (reading `req.body`, a 400, then an update).
- `views/page2.html`: the form that posts to `/setName`. The field names `firstname` and `lastname` are what `setName` reads out of `req.body`.

The done repo lists its routes in a different order than we typed them (`/page1`, `/page2`, `/getName`, then `/`). That's fine, because none of those paths overlap. Only the wildcard's position matters.

Video: [Austin, Week 7.2: Simple MVC, Controllers](https://www.youtube.com/watch?v=JthuX0jkFb0). Older Express, so the wildcard route is `app.get('/*', ...)`. On Express 5 that line crashes. See Common mistakes.

Used in: DomoMaker A through E, Project 2. DomoMaker's `router.js` is this file with more routes, and its controllers are split into two files.

## Common mistakes

- **`TypeError: argument handler must be a function`** when the server starts, with a stack trace that points at a line in `router.js`. A route points at something the controllers file never exported, or the name is spelled differently in the two files. `controllers.index` is `undefined` until `module.exports` includes `index`. Skip the `node_modules` lines in the trace. The first line that names one of your files is the one that matters.
- **`PathError [TypeError]: Missing parameter name at index 2: /*`.** Express 5 requires a name on a wildcard. Write `'/*wild'` (the name is up to you). Every tutorial written against Express 4 says `'/*'` or `'*'`, and both crash now.
- **A GET route 404s, and the code looks right.** It's below the wildcard. The wildcard catches every GET that reaches it, so a GET route under it never runs. Move the route above the wildcard. POST routes can sit below it, because `app.get` only matches GET and HEAD.
- **A bad URL shows Express's own `Cannot GET /nope` instead of your 404 page.** `notFound` is written but the wildcard route never made it into `router.js`, or the line is outside the function's closing `};`, or the file isn't saved.
- **`POST /anything` shows `Cannot POST`.** That's expected. The wildcard is a GET route. If you want a custom 404 for POSTs too, add `app.post('/*wild', controllers.notFound)` below your POST routes.
- **`res.status(404)` and nothing else.** `status` sets the code but sends nothing. The request hangs. Chain something that sends: `res.status(404).sendFile(...)` or `res.status(400).json({...})`.
- **`req.body.firstname` throws `Cannot read properties of undefined`.** The body-parsing middleware is missing from `app.js` or sits below `router(app)`. See [the app and middleware](app-and-middleware.md).
- **`403 Forbidden` from a `sendFile`.** The path has `../` in it and no `path.resolve`. Every `sendFile` in the controllers wraps its path in `path.resolve` for this reason.

## What you'll find on Google

- **`app.get('*', ...)` or `app.get('/*', ...)` for the 404.** Express 4. On Express 5 (what `npm install` gives you now) the wildcard needs a name: `'/*wild'`, `'/*splat'`, anything. If a library surprises you like this, check its current docs before assuming your code is wrong.
- **`app.use((req, res) => res.status(404).send('Not found'))` as the last line.** Also works, for every method at once, because `app.use` with no path matches everything. We use a GET route so it reads like the other routes, and because a POST to a wrong URL is a different mistake than a bad link.
- **`express.Router()`** (`const router = express.Router(); router.get(...); app.use('/api', router)`). The official way to group routes under a prefix. Our `router(app)` function does the same job with less to explain. DomoMaker uses our shape. Either is fine for Project 2.
- **`app.head(...)`.** You don't need it. Every `app.get` answers HEAD with the same headers and no body. `curl -I localhost:3000/page1` proves it. Compare [HEAD requests](../http/head-requests.md), where you wrote that by hand.
- **`res.send(obj)` instead of `res.json(obj)`.** `send` guesses the type from what you pass it. `json` always sends `application/json` and stringifies. Use `json` for API responses so the client knows what it's getting. Both set `Content-Length` for you, which is what [`respondJSON`](../http/status-codes.md) did by hand.
- **`res.sendStatus(404)`.** Sends the code with the status text (`Not Found`) as the body. Fine for an API. We send a page because this app is pages.
- **`(req, res, next)` with three parameters.** Middleware signature. A route handler that sends a response doesn't need `next`. If you see a tutorial calling `next()` inside a route, it's passing the request down to the next matching route.

## Redo it

Start from a fresh clone of the [starter](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-controllers) with `src/app.js` finished through `router(app)` (steps 1 to 7 on [the app and middleware](app-and-middleware.md)). In the starter, `router.js` exports an empty `router` function, and `controllers/index.js` has `hostIndex`, `hostPage1`, and `hostPage2` written but not exported. Run with `npm run nodemon`.

### 1. Three routes

In `src/router.js`, inside the empty function:

```js
const router = (app) => {   // already there
  app.get('/', controllers.index);
  app.get('/page1', controllers.page1);
  app.get('/page2', controllers.page2);
};   // already there
```

Save. The server crashes:

```
TypeError: argument handler must be a function
    at router (.../src/router.js:4:7)
```

That's on purpose. `controllers.index` is `undefined`, because nothing is exported yet. A route needs a function.

### 2. Export the controllers

At the bottom of `src/controllers/index.js`:

```js
module.exports = {
  index: hostIndex,
  page1: hostPage1,
  page2: hostPage2,
};
```

Checkpoint: `/`, `/page1`, and `/page2` all load. Click the links between them.

Now from the second terminal:

```
curl -I localhost:3000/page1
```

`200 OK`, `Content-Type: text/html; charset=utf-8`, a `Content-Length`, no body. That's HEAD, and you didn't write it. `sendFile` picked the content type from the `.html` extension.

### 3. The 404 controller

In `src/controllers/index.js`, above `module.exports`:

```js
const notFound = (req, res) => {
  return res.status(404).sendFile(path.resolve(`${__dirname}/../../views/notFound.html`));
};
```

Add it to the exports:

```js
  page2: hostPage2,   // already there
  notFound,
};   // already there
```

`status` sets the code. `sendFile` sends. Nothing points at `notFound` yet, so a bad URL still gets Express's plain `Cannot GET`.

### 4. The wildcard route

In `src/router.js`, as the last line inside the function:

```js
  app.get('/page2', controllers.page2);   // already there

  app.get('/*wild', controllers.notFound);
};   // already there
```

If you type `'/*'` the way older tutorials do, you get `Missing parameter name at index 2: /*`. Express 5 wants a name after the star.

Checkpoint: `localhost:3000/nope` and `/a/b/c` show your 404 page, with its image. So does `/assets/img/missing.png`. A file static hosting can't find falls through to the wildcard.

### 5. `getName`

In `src/controllers/index.js`, above `module.exports`:

```js
const getName = (req, res) => {
  return res.json({ name });
};
```

`name` is the `let name = 'unknown'` at the top of the file, the demo's stand-in for data. `res.json` stringifies, sets `Content-Type`, works out `Content-Length`, and sends.

### 6. `setName`: the check

Under `getName`:

```js
const setName = (req, res) => {
  console.dir(req.body);

  if (!req.body.firstname || !req.body.lastname) {
    return res.status(400).json({
      error: 'firstname and lastname are both required',
      id: 'setNameMissingParams',
    });
  }
};
```

`req.body` is already an object. The middleware in `app.js` did that before this function ran. Compare the 20 lines of [`parseBody`](../http/parsing-post-body.md).

### 7. `setName`: the update

Inside `setName`, after the `if` block:

```js
  }   // already there

  name = `${req.body.firstname} ${req.body.lastname}`;
  return res.json({ name });
};   // already there
```

Then export both:

```js
  notFound,   // already there
  getName,
  setName,
};   // already there
```

### 8. Two routes, on either side of the wildcard

In `src/router.js`:

```js
  app.get('/page2', controllers.page2);   // already there
  app.get('/getName', controllers.getName);

  app.get('/*wild', controllers.notFound);   // already there
  app.post('/setName', controllers.setName);
};   // already there
```

`getName` goes above the wildcard because the wildcard grabs every GET that reaches it. Put `/getName` below and it 404s. `setName` can go below, because it's a POST and `app.get` doesn't look at POSTs.

### 9. Four requests

Browser, `localhost:3000/getName`:

```
{"name":"unknown"}
```

Browser, `/page2`, fill in the form and submit. The page shows `{"name":"John Paul"}` (or whatever you typed), and the terminal shows `{ firstname: 'John', lastname: 'Paul' }`.

Second terminal, a JSON body:

```
curl -i -H "Content-Type: application/json" -d '{"firstname":"Ada","lastname":"Lovelace"}' localhost:3000/setName
```

`200` and `{"name":"Ada Lovelace"}`. One function took a form body and a JSON body, because of the two body-parsing lines in `app.js`.

A field missing:

```
curl -i -d "firstname=John" localhost:3000/setName
```

`400` and `{"error":"firstname and lastname are both required","id":"setNameMissingParams"}`.

### What replaced what

| Project 1, by hand | Express |
|---|---|
| `urlStruct` and the `if` in `onRequest` | `app.get(path, fn)` and `app.post(path, fn)` |
| `request.method === 'HEAD'` checks | free with every `app.get` |
| The `else` or default case in `onRequest` | `app.get('/*wild', notFound)` |
| `respondJSON(request, response, 200, obj)` | `res.json(obj)` |
| `response.writeHead(400, ...)` then `write` and `end` | `res.status(400).json(obj)` |
| `request.body` after `parseBody` | `req.body`, already parsed |

The order of operations, top to bottom: middleware in `app.js`, then GET routes in the order you wrote them with the wildcard last, then POST routes. When something is weird, that order is the first thing to check.
