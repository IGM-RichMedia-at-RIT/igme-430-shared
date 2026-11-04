# Mongoose queries: find, search, and update

[The previous page](mongoose-connect-schema-save.md) put cats into the database. This one gets them back out and changes one in place. The idea underneath all of it: let the database do the work. Sorting, filtering, and adding a bed all happen inside Mongo. You never pull every cat into JavaScript to find the one you want.

Every query here has the same shape. Ask the model a question, `await` the answer inside a `try`, then decide what to send: 500 if the database failed, 404 if nothing matched, 200 with the data otherwise.

## Quick example

From `server/controllers/index.js` in [simple-mvc-example-done](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-example-done/blob/master/server/controllers/index.js). The newest cat's name, as JSON:

```js
const getName = async (req, res) => {
  try {
    const doc = await Cat.findOne({})
      .sort({ createdDate: 'descending' })
      .lean()
      .exec();

    if (!doc) {
      return res.status(404).json({ error: 'No cat found' });
    }
    return res.json({ name: doc.name });
  } catch (err) {
    console.log(err);
    return res.status(500).json({ error: 'Something went wrong contacting the database' });
  }
};
```

Read the chain top to bottom: find one cat, newest first, as a plain object, go. Only the last line gets a semicolon.

To change a document without pulling it out first, `findOneAndUpdate` takes three arguments: which document, what to change, how to pick:

```js
const updatePromise = Cat.findOneAndUpdate(
  {},
  { $inc: { bedsOwned: 1 } },
  { returnDocument: 'after', sort: { createdDate: 'descending' } }
).lean().exec();
```

That finds the newest cat, adds one to its beds inside the database, and hands back the version after the change. One trip to Mongo.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 9B (Wed 10/21) | Connect, the `Cat` schema, `.save()`, listing every cat on page 1 with `find({})` | [Previous page](mongoose-connect-schema-save.md) |
| 9C (Fri 10/23) | `findOne` with `.sort`, 404 on `null`, the home page, search by name with `.select`, `findOneAndUpdate` with `$inc` and `returnDocument` | Starter: [Simple-MVC-Example](https://github.com/IGM-RichMedia-at-RIT/Simple-MVC-Example) · Done: [simple-mvc-example-done](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-example-done) |

Where to look in the done repo:

- `server/controllers/index.js`: `getName` (sort as `.sort()`), `hostIndex` (the same sort, passed in the options object instead), `searchName`, `updateLast`
- `server/router.js`: which URL runs which function. `/getName` and `/findByName` are GETs, `/updateLast` is a POST
- `views/page2.handlebars`: the search form (GET, so the name rides in the URL as `?name=`) and the Update Last button

Two places the done repo differs from what we typed. Its `searchName` has no `.select()` and does the 404 check after the `try` instead of inside it. Same result. Its `updateLast` hangs `.then` and `.catch` off the promise as two separate statements instead of one chain. That one matters: see the crash in Common mistakes.

Video: [Austin, Week 9.2: Simple MVC, Models](https://www.youtube.com/watch?v=2DgCCVpRRbM). The second half is this page. It matches the done repo's shapes, including the unchained `.then` / `.catch`.

Used in: Simple Models (look up a dog by name and add 1 to its age, with an error if there is no such dog), DomoMaker B (each user gets back only their own domos), Project 2.

## Common mistakes

- **Show Name gives you the oldest cat.** No `.sort`. `findOne({})` with no sort returns the first one Mongo finds, which is the oldest.
- **`SyntaxError: await is only valid in async functions`** and the server won't start. You added an `await` to a function that isn't marked `async`. Add the word to the function's first line.
- **The home page still says unknown, and page 1 shows cats.** Look at the terminal. An error there means the query failed. No error means the `if (doc)` block that copies the name is missing.
- **Search 404s on a cat you can see on page 1.** It's an exact match. Case and spaces count, so `mr whiskers` does not find `MR Whiskers`.
- **Search returns a name but no beds.** A typo in `.select()`. The field is `bedsOwned`, so `'name beds'` returns only the name.
- **Update Last sets beds to 1 instead of adding 1.** The update argument is `{ bedsOwned: 1 }`. That's a set. `{ $inc: { bedsOwned: 1 } }` is an add.
- **Update Last's response is one behind page 1.** No `returnDocument: 'after'`. Mongo handed you the document from before the change.
- **The server dies on Update Last with `Cannot read properties of null (reading 'name')`.** The `.then` and `.catch` are two separate statements (the done repo's shape) and nothing matched the filter, so `doc` is `null`, the `.then` throws, and the `.catch` isn't attached to the `.then`. Chain them, or use `await` with `try/catch`, and check for `null` before you touch `doc.name`. Your dog lookup in Simple Models is exactly this case.
- **`SyntaxError: Unexpected token '.'`** A semicolon in the middle of a chain, usually after `findOne({})` or after `}))`. Only the last line of a chain gets one.
- **Page 1 says ` has  beds.` with the names blank.** No `.lean()` on the `find`. Handlebars needs plain objects. See [Handlebars templates](../express/handlebars-templates.md).
- **ESLint: `'err' is defined but never used`.** A `catch (err)` with no `console.log(err)` inside. The homework needs zero errors, so log it.
- **Crash about 30 seconds after start, `ECONNREFUSED 127.0.0.1:27017`.** MongoDB isn't running. Same fix as the [previous page](mongoose-connect-schema-save.md).

## What you'll find on Google

- **`{ new: true }` instead of `returnDocument: 'after'`.** The older Mongoose spelling of the same option. It still works. `returnDocument` is the current name and matches the MongoDB driver's.
- **`findById(id)`.** Shorthand for `findOne({ _id: id })`. Handy once you have an `_id` from a previous query, which is how DomoMaker looks things up.
- **`updateOne` and `updateMany`.** They change documents but don't hand the document back, just a count. Use `findOneAndUpdate` when you want the result, which for us is always.
- **Callbacks as the last argument, `Cat.find({}, (err, docs) => { ... })`.** Removed in Mongoose 7. Any tutorial that does this is out of date, and current Mongoose throws `Model.find() no longer accepts a callback`. Use `await` or `.then`.
- **No `.exec()`.** Queries work without it, since they're thenable. We keep it because Mongoose gives better error messages when you do, and it makes "this is where the query runs" visible.
- **`.sort('-createdDate')` or `.sort({ createdDate: -1 })`.** Same as `'descending'`. All three spellings work.
- **`Cat.find({}).sort(...).limit(1)`.** Same answer as `findOne` with a sort, but you get an array with one thing in it.
- **Sorting in JavaScript, `docs.sort(...)` after `find({})`.** It works, and it's slower. You pull every cat over the network to keep one. Let Mongo sort.
- **`.then` versus `await`.** Both are fine and you'll read both. The class code uses `await` with `try/catch` everywhere except `updateLast`, which uses `.then` on purpose so you've seen it. Pick one style per project.

## Redo it

Start where the [previous page](mongoose-connect-schema-save.md) ends: connected, `Cat` model, `setName` saves, page 1 lists every cat. The four functions below are empty in the starter and the routes are already wired. Everything you type is in `server/controllers/index.js`. Run `npm run nodemon` and keep that terminal visible, because `console.log` in a controller prints there, not in the browser.

Each function only runs when you hit its URL. A `console.log` in `getName` stays silent when you refresh the home page.

| You do this | URL | Runs |
|---|---|---|
| Page 1, the Show Name link | `/getName` | `getName` |
| Load or refresh the home page | `/` | `hostIndex` |
| Page 2, the search box | `/findByName?name=...` | `searchName` |
| Page 2, the Update Last button | `/updateLast` | `updateLast` |

Add a cat or two on page 2 first so there's something to find.

### 1. `getName`: find one cat

Replace the empty function. Note the `async`.

```js
const getName = async (req, res) => {
  try {
    const doc = await Cat.findOne({})
      .lean()
      .exec();
    return res.json({ name: doc.name });
  } catch (err) {
    console.log(err);
    return res.status(500).json({ error: 'Something went wrong contacting the database' });
  }
};
```

`findOne` is `find` that stops at the first match. `{}` matches every cat, so this means "any one cat."

Checkpoint: `localhost:3000/getName` shows `{"name":"..."}` with your oldest cat. Not the one you just added.

### 2. Sort newest first

Add one line to the chain, between `findOne({})` and `.lean()`:

```js
    const doc = await Cat.findOne({})   // already there
      .sort({ createdDate: 'descending' })
      .lean()   // already there
```

Checkpoint: refresh `/getName`. Now it's the newest cat. Mongo sorted and sent one. Nothing got sorted in JavaScript.

### 3. No cat? 404

Between the end of the chain and the `return res.json`:

```js
      .exec();   // already there

    if (!doc) {
      return res.status(404).json({ error: 'No cat found' });
    }
    return res.json({ name: doc.name });   // already there
```

An empty database isn't an error. `findOne` hands back `null`, and without this check `doc.name` throws and the user gets a 500 blaming the database.

### 4. The home page: find the newest cat and log it

In `hostIndex`, add `async` to the first line, then type under `let name = 'unknown';`:

```js
const hostIndex = async (req, res) => {
  let name = 'unknown';   // already there
  try {
    const doc = await Cat.findOne(
      {},
      {},
      { sort: { createdDate: 'descending' } }
    ).lean().exec();
    console.log(doc);
  } catch (err) {
    console.log(err);
  }

  res.render('index', {   // already there
```

Same query as `getName`, with the sort in the third argument instead of a `.sort()` call. Both spellings are in the docs and in the done repo. The three arguments are filter, fields, options: any cat, all fields, newest first.

Checkpoint: refresh the home page. It still says unknown, but the terminal prints the whole cat, `_id` and `__v` included.

### 5. Hand the name to the page

Inside the `try`, under the `console.log`:

```js
    console.log(doc);   // already there
    if (doc) {
      name = doc.name;
    }
  } catch (err) {   // already there
```

Checkpoint: refresh. The home page names the newest cat. Notice there's no 500 in this `catch`. If Mongo is down the page still renders with "unknown." A database hiccup shouldn't take down the front door.

### 6. Search: find by name and log it

Look at the third form in `views/page2.handlebars` first. It's a GET, so the name arrives as `req.query.name`, and the 400 for an empty box is already written. In `searchName`, add `async` to the first line, then type under the 400 check's closing brace:

```js
  }   // already there, the end of the 400 check

  try {
    const doc = await Cat.findOne({ name: req.query.name })
      .select('name bedsOwned')
      .exec();
    console.log(doc);
  } catch (err) {
    console.log(err);
    return res.status(500).json({ error: 'Something went wrong' });
  }
};   // already there
```

Now the filter has a condition in it. `.select` says which fields to send back.

Checkpoint: search for a cat by its exact name. The browser spins, because you haven't answered yet, and the terminal prints `{ _id, name, bedsOwned }`. No `createdDate`, because `select` did its job. `_id` comes along whether you asked or not. Search again with the wrong capitalization: the terminal prints `null`.

### 7. Answer: 404 or the cat

Inside the `try`, under the `console.log`:

```js
    console.log(doc);   // already there
    if (!doc) {
      return res.status(404).json({ error: 'No cat found' });
    }
    return res.json({ name: doc.name, beds: doc.bedsOwned });
  } catch (err) {   // already there
```

Checkpoint: the exact name gives `{"name":"...","beds":...}`. The wrong case gives a 404. An empty box gives the 400 that was already there. Three outcomes: Mongo is down, 500. No cat by that name, 404. Found it, 200.

### 8. `updateLast`, wrong on purpose

The goal: the Update Last button on page 2 adds one bed to the newest cat and sends back its name and new bed count. You'll get there in three passes, and the first two are wrong on purpose so you see what each fix does.

Inside the empty `updateLast`. No `async` on this one; it uses plain promises so you've read both styles.

```js
const updateLast = (req, res) => {   // already there
  const updatePromise = Cat.findOneAndUpdate(
    {},
    { bedsOwned: 1 },
    { sort: { createdDate: 'descending' } }
  ).lean().exec();

  updatePromise
    .then((doc) => res.json({
      name: doc.name,
      beds: doc.bedsOwned,
    }))
    .catch((err) => {
      console.log(err);
      return res.status(500).json({ error: 'Something went wrong' });
    });
};
```

Three arguments, three lines: which cat, what to change, how to pick. No semicolon after `}))`, so the `.catch` is on the same chain and covers everything above it.

Checkpoint: click Update Last on page 2, then look at page 1. The newest cat now has exactly 1 bed, whatever it had before. You said `bedsOwned: 1` and Mongo did exactly that: it set it.

### 9. Fix 1: `$inc`

Change the second argument only:

```js
    {},   // already there
    { $inc: { bedsOwned: 1 } },
    { sort: { createdDate: 'descending' } }   // already there
```

`$inc` is Mongo's built-in "add this much," run inside the database.

Checkpoint: click Update Last. The response says 1 bed. Page 1 says 2. The database is right now, but Mongo handed back the cat from before the change.

### 10. Fix 2: `returnDocument: 'after'`

Change the third argument only:

```js
    { $inc: { bedsOwned: 1 } },   // already there
    { returnDocument: 'after', sort: { createdDate: 'descending' } }
  ).lean().exec();   // already there
```

Checkpoint: click Update Last. The response and page 1 agree, and each click adds one. Add a new cat and it becomes the target, because "last" means newest `createdDate`.

Why not find the cat, add one in JavaScript, and `.save()` it? That's two round trips to the database. This is one.

### Deleting

There's no delete in the demo, but it's the same shape minus the update:

```js
const doc = await Cat.findOneAndDelete({ name: req.query.name }).lean().exec();
```

`doc` is the deleted cat, or `null` if nothing matched. Check for `null` the way `getName` does.

For the Simple Models homework, aging a dog is search's filter in the first argument and Update Last's `$inc` in the second. A dog that isn't there comes back `null`, not as an error, so check for it and send the message the assignment asks for.
