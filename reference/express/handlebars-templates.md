# Handlebars templates and res.render

With `sendFile`, a page is a finished HTML file and the server hands it over as-is. With a template, the page has holes in it, and the controller fills the holes before the page leaves the server. The browser never sees the template. View source proves it. That's the V in MVC.

Handlebars is the template engine this course uses. A hole looks like `{{title}}`, and `res.render` fills it from an object you pass in.

## Quick example

The setup, from `server/app.js` in [simple-mvc-templates-done](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-templates-done/blob/master/server/app.js). Three lines, after the body-parsing middleware and before `router(app)`:

```js
const expressHandlebars = require('express-handlebars');

app.engine('handlebars', expressHandlebars.engine({ defaultLayout: '' }));
app.set('view engine', 'handlebars');
app.set('views', `${__dirname}/../views`);
```

A controller that fills a template, from [`server/controllers/index.js`](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-templates-done/blob/master/server/controllers/index.js):

```js
const hostPage1 = (req, res) => {
  const myArray = ['Austin', 'Cody', 'Binky', 'Travis'];
  const filtered = myArray.filter((x) => x.length <= 4);
  res.render('page1', {
    title: 'Page 1',
    names: filtered,
  });
};
```

And the template it fills, [`views/page1.handlebars`](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-templates-done/blob/master/views/page1.handlebars):

```html
<title>{{title}}</title>
...
{{#if names}}
<h2>Names:</h2>
<ul>
  {{#each names}}
  <li>{{this}}</li>
  {{/each}}
</ul>
{{/if}}
```

`render('page1', data)` means `views/page1.handlebars`. Every key in `data` becomes a hole the template can use. The browser gets plain HTML with the holes already filled.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 8C (Fri 10/16) | Turning on Handlebars, holes, `#if` and `#each`, a 404 that names the URL, filtering in the controller | Starter: [simple-mvc-templates](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-templates) · Done: [simple-mvc-templates-done](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-templates-done) |

Where to look in the done repo:

- `server/app.js`: the `app.engine` and two `app.set` lines
- `server/controllers/index.js`: `hostIndex`, `hostPage1`, and `notFound` all call `render`; `getName` and `setName` answer with `res.json`
- `views/index.handlebars`, `views/page1.handlebars`, `views/notFound.handlebars`: the holes
- `server/router.js` is the same shape as the [Controllers demo](router-controllers-404.md) and we never touched it

The done repo differs from class in a few small ways. It keeps a list of names (`names.push(...)`) instead of a single `name`, the home page hole is called `currentName` instead of `userName`, and the page 1 filter drops `'unknown'` instead of keeping short names. Same ideas, different data.

Video: [Austin, Week 8.2: Simple MVC, Views](https://www.youtube.com/watch?v=pvC7moS6FeQ) (30:54). Same starter and the same demo. He does the setup before the first `render`, so the on-purpose 500 in step 2 below isn't in it.

Used in: DomoMaker A and B (the login page and the app page are Handlebars views), Project 2 (Handlebars views are required).

## Common mistakes

- **500, `Error: No default engine was specified and no extension was provided.`** The `app.engine` line or the `app.set('view engine', ...)` line is missing or misspelled. You called `render` before Express knew what a template was.
- **500, `ENOENT: no such file or directory, open '.../views/layouts/main.handlebars'`.** You left out `{ defaultLayout: '' }`. express-handlebars wraps every page in a shared layout unless you tell it not to.
- **500, `Failed to lookup view "index" in views directory ...`.** The `app.set('views', ...)` path is wrong, or the name you passed to `render` doesn't match a file in `views/`.
- **A hole shows up blank, with no error.** The key in `render` doesn't match the `{{hole}}`. Case counts: `userName` and `username` are different holes. A missing variable is not an error in Handlebars, just an empty string, so nothing warns you.
- **You edited a controller and nothing changed.** Controller edits need nodemon's restart. Check the terminal. Template edits don't need a restart while you're developing, so if a `.handlebars` change doesn't show, you probably didn't refresh.
- **It worked locally, but on Heroku a template edit doesn't show.** With `NODE_ENV=production` (Heroku sets it) Express caches templates. Restart the app.
- **`{{#if names.length > 3}}` gives a parse error.** Handlebars doesn't do comparisons. Work it out in the controller and pass a true or false.
- **`#if` on an empty array is false.** In JavaScript, `[]` is truthy. In Handlebars it isn't. An empty list hides the whole block, which is usually what you want.
- **Every page spins forever.** That controller function is still empty. Express waits for a response that never comes.
- **`EADDRINUSE :::3000`.** Another server is still running on the port. Stop it, or `lsof -ti tcp:3000 | xargs kill -9` on a Mac.

## What you'll find on Google

- **`exphbs({ ... })` or `expressHandlebars({ ... })` called as a function.** That's the express-handlebars API from version 5 and earlier. Version 6 and later use `.engine(...)`, which is what we have. Copy an old tutorial and you get `exphbs is not a function`.
- **A `views/layouts/main.handlebars` file.** Most tutorials use a layout: a shared wrapper with the `<head>`, nav, and footer that every page slots into. We turned it off with `defaultLayout: ''` so each view is a whole page. Layouts are fine for Project 2 if you want a shared nav.
- **EJS, Pug, Nunjucks.** Same idea, different syntax (`<%= title %>`, or indentation instead of tags). `app.set('view engine', 'ejs')` swaps them in. We use Handlebars because DomoMaker and Project 2 do.
- **Template strings: `` res.send(`<h1>${name}</h1>`) ``.** It works. You lose escaping, and your HTML ends up inside your JavaScript. Templates keep HTML in HTML files and escape by default.
- **`{{{triple}}}` braces.** Raw output, no escaping. `{{x}}` turns `<` and `=` into safe codes, which is why `/nope?x=1` shows as `x&#x3D;1` in view source. `{{{x}}}` doesn't. Never triple-brace anything a user typed. That's how XSS happens.
- **`Handlebars.registerHelper`.** Custom helpers let you put logic in templates. They exist. For this class, the logic goes in the controller and the template shows what it's handed.
- **"Server-side rendering" vs "client-side rendering."** Handlebars is the server side: the HTML arrives finished and view source shows the data. React is the other end: view source is nearly empty and the browser builds the page. See [React: setup, components, and JSX](../react/setup-components-jsx.md).
- **`.lean()` in Mongoose examples.** Once your data comes from MongoDB, a template can't read a Mongoose document directly. See [Mongoose queries](../database/mongoose-queries.md).

## Redo it

Start from a fresh copy of the [starter](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-templates) and run `npm install`. All six controller functions are empty. The middleware in `app.js` is already written (see [Express: the app and middleware](app-and-middleware.md)), `router.js` is finished, and every view is a `.handlebars` file with only `{{title}}` in it so far. Run with `npm run nodemon`.

### 1. Run it

`localhost:3000` spins forever. The route exists, but `hostIndex` never sends anything.

### 2. Render before setup (wrong on purpose)

In `server/controllers/index.js`, inside `hostIndex`:

```js
const hostIndex = (req, res) => {   // already there
  res.render('index', {
    title: 'Home',
  });
};   // already there
```

Refresh. You get a 500 and `Error: No default engine was specified and no extension was provided.` The server stays up. Express caught it, but `render` doesn't know what a template is yet.

### 3. Turn on Handlebars

In `server/app.js`, with the other requires:

```js
const favicon = require('serve-favicon');   // already there
const expressHandlebars = require('express-handlebars');
```

Then after `express.json()` and before `router(app)`:

```js
app.use(express.json());   // already there

app.engine('handlebars', expressHandlebars.engine({ defaultLayout: '' }));
app.set('view engine', 'handlebars');
app.set('views', `${__dirname}/../views`);

router(app);   // already there
```

Line one: here's a template engine for `.handlebars` files. Line two: use it by default, so `render('index')` means `index.handlebars`. Line three: the templates live in `views/`.

`app.set` is different from `app.use`. `use` adds a stop every request passes through. `set` stores a setting and nothing runs per request.

Checkpoint: refresh. The tab says `Home`. Right-click, View Page Source: `<title>Home</title>`, no curly braces.

### 4. Add holes

In `views/index.handlebars`, under `<body>`:

```html
<body>   <!-- already there -->
  <h1>Welcome to {{pageName}}!</h1>
  <p>How are you today, {{userName}}?</p>
```

Refresh without restarting. You get `Welcome to !` and `How are you today, ?`. A missing variable is just empty. And nodemon didn't restart, because it only watches `server/`, and Express re-reads templates while you develop.

### 5. Fill them

Back in `hostIndex`:

```js
  res.render('index', {   // already there
    title: 'Home',   // already there
    pageName: 'Home Page',
    userName: name,
  });   // already there
```

`name` is the `let name = 'unknown'` at the top of the file. The key on the left has to match the hole exactly.

Checkpoint: `Welcome to Home Page!` and `How are you today, unknown?`

### 6. The other pages

Fill `hostPage1` and `hostPage2`:

```js
const hostPage1 = (req, res) => {   // already there
  res.render('page1', {
    title: 'Page 1',
  });
};   // already there

const hostPage2 = (req, res) => {   // already there
  res.render('page2');
};   // already there
```

Click Page 1. It loads, but the tab shows the URL instead of a title. In `views/page1.handlebars`, the `<head>` is empty:

```html
<head>   <!-- already there -->
  <title>{{title}}</title>
</head>   <!-- already there -->
```

Refresh: the tab says `Page 1`. You were already sending `title`. The template just had no hole for it.

### 7. A 404 that says what you asked for

In `notFound`:

```js
const notFound = (req, res) => {   // already there
  res.status(404).render('notFound', {
    page: req.url,
  });
};   // already there
```

And in `views/notFound.handlebars`, under `<body>`:

```html
<body>   <!-- already there -->
  <h1>404! Could not find {{page}}!</h1>
```

Checkpoint: `localhost:3000/a/b` shows `404! Could not find /a/b!` Same chaining as the Controllers demo: `status` sets the code, `render` sends.

### 8. The form changes the page

`getName` and `setName` are the same two functions as the Controllers demo, so if you have that done repo open, copy them. Otherwise:

```js
const getName = (req, res) => {   // already there
  res.json({ name });
};   // already there

const setName = (req, res) => {   // already there
  if (!req.body.firstname || !req.body.lastname) {
    return res.status(400).json({
      error: 'firstname and lastname are both required',
      id: 'setNameMissingParams',
    });
  }

  name = `${req.body.firstname} ${req.body.lastname}`;
  return res.json({ name });
};   // already there
```

Checkpoint: Page 2, type a first and last name, Submit. You get JSON back. Go to `localhost:3000` says `How are you today, John Paul?` The form changed the data, the template showed the new data, and the controller sat in between.

### 9. `#if`

In `views/page1.handlebars`, under the `<h1>`:

```html
<h1>Welcome to page 1 </h1>   <!-- already there -->
{{#if names}}
<h2>Names:</h2>
{{/if}}
```

`#` opens a block and `/` closes it. The inside only renders if `names` is truthy. Refresh: no "Names:", because you never passed `names`. Pass something in `hostPage1`:

```js
  res.render('page1', {   // already there
    title: 'Page 1',   // already there
    names: ['test'],
  });   // already there
```

Checkpoint: `Names:` appears.

### 10. `#each`

Between `<h2>Names:</h2>` and `{{/if}}`:

```html
<h2>Names:</h2>   <!-- already there -->
<ul>
  {{#each names}}
  <li>{{this}}</li>
  {{/each}}
</ul>
{{/if}}   <!-- already there -->
```

`#each` repeats its inside once per item, and `{{this}}` is the current item. Refresh: one bullet, `test`.

### 11. Filter on the server

In `hostPage1`, two edits. Add the array and the filter, and change what `names` points at:

```js
const hostPage1 = (req, res) => {   // already there
  const myArray = ['Austin', 'Cody', 'Binky', 'Travis'];
  const filtered = myArray.filter((x) => x.length <= 4);
  res.render('page1', {   // already there
    title: 'Page 1',   // already there
    names: filtered,
  });
};   // already there
```

The filter alone changes nothing on the page. The template only sees what's in `render`, so `names` has to point at `filtered`.

Checkpoint: one bullet, `Cody`. The server decides what's in the list. The template shows whatever it's handed.

### 12. A list the form adds to (optional)

This is where the done repo ends up. Replace `let name = 'unknown';` with `const names = [];`, have `setName` push the new name instead of assigning it, pass `names` straight to page 1, and show `names[names.length - 1] || 'unknown'` on the home page and in `getName`. Page 1 starts empty (an empty array makes `#if` false), and each submit adds a bullet.

Restart the server and it's all gone. That's why the next topic is a database: [MongoDB and Mongoose](../database/mongoose-connect-schema-save.md).
