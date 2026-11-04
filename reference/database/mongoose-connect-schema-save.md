# MongoDB and Mongoose: connect, define a model, save

Until now, every piece of data in our apps has been a variable, and it died when the server restarted. MongoDB fixes that. It's a separate server that writes to disk, and our Node server talks to it over the network. That's why everything on this page is `async`: a save is a request to another server, and we wait for the answer.

Mongoose is the library in between. Raw Mongo will store anything. Mongoose adds a schema, so a cat is guaranteed to have a name and a number of beds, and it gives you `Cat.find(...)` and `newCat.save()` instead of hand-written database commands.

This page is the first half of the Simple MVC Models demo: connect, define the model, save a cat, list the cats. Finding, searching, and updating are on [Mongoose queries](mongoose-queries.md).

## Quick example

The model, from `server/models/Cat.js` in [simple-mvc-example-done](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-example-done/blob/master/server/models/Cat.js):

```js
const mongoose = require('mongoose');

const CatSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true,
    unique: true,
  },
  bedsOwned: {
    type: Number,
    min: 0,
    required: true,
  },
  createdDate: {
    type: Date,
    default: Date.now,
  },
});

const CatModel = mongoose.model('Cat', CatSchema);
module.exports = CatModel;
```

Saving one, from `setName` in [`server/controllers/index.js`](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-example-done/blob/master/server/controllers/index.js):

```js
const { Cat } = models;

const setName = async (req, res) => {
  // ...400 check on the form fields...

  const catData = {
    name: `${req.body.firstname} ${req.body.lastname}`,
    bedsOwned: req.body.beds,
  };

  const newCat = new Cat(catData);

  try {
    await newCat.save();
    return res.status(201).json({ name: newCat.name, beds: newCat.bedsOwned });
  } catch (err) {
    console.log(err);
    return res.status(500).json({ error: 'failed to create cat' });
  }
};
```

`new Cat(catData)` builds a document in the right shape. Nothing is in the database until `save()` comes back.

## Where we did this

| Day | What we covered | Code |
|---|---|---|
| 9A (Mon 10/19) | Databases slides: why a database, SQL vs Mongo, where Mongoose sits | No code |
| 9B (Wed 10/21) | Connection string, the Cat schema, `new Cat`, `save`, Compass, listing cats, the restart test | Starter: [Simple-MVC-Example](https://github.com/IGM-RichMedia-at-RIT/Simple-MVC-Example) · Done: [simple-mvc-example-done](https://github.com/IGM-RichMedia-at-RIT/simple-mvc-example-done) |

Where to look in the done repo:

- `server/app.js`: `dbURI` and `mongoose.connect`, right after the `router` require
- `server/models/Cat.js`: the schema and the model
- `server/models/index.js`: one line, and it's the line everyone forgets for the homework
- `server/controllers/index.js`: `setName` (build and save), `hostPage1` (`find({})` for the list)
- The done repo also has Friday's work in it (`getName`, `searchName`, `updateLast`, and a `hostIndex` that finds the newest cat). Those are on [Mongoose queries](mongoose-queries.md).

The done repo doesn't have the optional 400 for a taken name (step 13 below). Every failed save is a 500 there.

Videos: [Austin, Week 9.1: Databases](https://www.youtube.com/watch?v=BnUBOLjXVZU) (36:59) is the Monday slides. [Austin, Week 9.2: Simple MVC, Models](https://www.youtube.com/watch?v=2DgCCVpRRbM) (1:04:33) is the whole demo, this page and the queries page in one sitting. Same repo as ours.

Used in: Simple Models assignment (the same app with dogs), DomoMaker A (the `Account` and `Domo` models), Project 2.

## Before you start: MongoDB has to be running

Your server connects to Mongo on your own machine while you develop. If Mongo isn't up, the server starts, waits about 30 seconds, prints `Could not connect to database`, and exits.

- **Mac (Homebrew):** `brew services start mongodb-community`. Once started it stays up.
- **Windows (installed as a service):** it's already running. If not, open the Services app, find MongoDB, and Start it.
- **Lab machine:** double-click `mongod.exe` in `C:\Program Files\MongoDB\Server\<version>\bin`. A console window opens. Leave it open. Mongo runs only while that window is up, and it is not a service on those machines.

Compass is the viewer. Connect it to `mongodb://localhost:27017`. Your app doesn't need Compass. It's for you.

For a deployed app, the database is in the cloud: [Setting Up MongoDB Atlas](../../documents/mongodb-atlas-setup.md). The code doesn't change. Only `MONGODB_URI` does.

## Common mistakes

- **Server starts, then about 30 seconds later: `Could not connect to database` and `MongooseServerSelectionError: connect ECONNREFUSED 127.0.0.1:27017`.** Mongo isn't running. See above. On a lab machine, you closed the `mongod.exe` window.
- **`mongod.exe` flashes and closes.** No data folder. Make `C:\data`, then `db` inside it, and run it again.
- **A save takes 10 seconds, then 500 with `Operation 'cats.insertOne()' buffering timed out after 10000ms`.** Same cause. Mongoose queued the save while waiting for a connection that never came.
- **`SyntaxError: await is only valid in async functions`.** You wrote `await` in a function that isn't marked `async`. Nodemon says the app crashed. Add the word and save.
- **`ReferenceError: Cat is not defined`.** The `const { Cat } = models;` line is missing from the controller.
- **`TypeError: Cat is not a constructor`.** `Cat.js` is missing its `module.exports = CatModel;` line.
- **You built `Dog.js` and `models.Dog` is undefined.** You forgot to add `module.exports.Dog = require('./Dog.js');` to `models/index.js`. Requiring a folder loads only its `index.js`, so a file that `index.js` never requires never runs. This is the number one Simple Models mistake.
- **Every save is a 500.** Read the terminal. `E11000 duplicate key error collection: ... index: name_1` means the name is taken. `Cat validation failed: bedsOwned: Path 'bedsOwned' (-1) is less than minimum allowed value (0)` means the schema rejected it. `Cast to Number failed` means beds wasn't a number.
- **Page 1 says ` has  beds.` with the words missing.** You left `.lean()` off the `find`. Handlebars can't read a full Mongoose document, only a plain object.
- **Every cat has the same `createdDate`.** You wrote `default: Date.now()` with parentheses. That runs once, when the server starts, and every cat gets that time. Hand Mongoose the function, `Date.now`, and it calls it per cat.
- **Submit spins forever.** The `try` block has no `return res...` in it, so nothing ever answers.
- **Compass shows nothing.** Hit refresh. It doesn't update by itself. If it's still empty, nothing has saved yet, since the database only appears after the first save.
- **You pushed a real connection string to GitHub.** Your password is in it, and GitHub will flag it. The string belongs in an environment variable, never in code.

## What you'll find on Google

- **`mongoose.connect(uri, { useNewUrlParser: true, useUnifiedTopology: true })`.** Those options were needed years ago. Since Mongoose 6 they do nothing, and on ours (Mongoose 8) the driver prints a deprecation warning for them. Just `mongoose.connect(uri)`.
- **`mongoose.connect(uri, (err) => { ... })` with a callback.** Callbacks were removed in Mongoose 7. Use `.catch(...)`, like our `app.js`, or `await`.
- **`Cat.create(catData)`.** Same as `new Cat(catData)` followed by `save()`, in one call. Either works. We split them so you can `console.log` the document before it goes in and see the `_id` and the cast types.
- **The `mongodb` package itself (`MongoClient`, `db.collection('cats').insertOne(...)`).** That's the native driver. No schema, no model, no validation. Mongoose sits on top of it. Tutorials that use it are solving the same problem with less structure.
- **`timestamps: true` as a schema option.** Mongoose adds `createdAt` and `updatedAt` for you. Fine for Project 2. We write `createdDate` by hand so you see what `default` does.
- **`unique: true` "isn't a validator."** True, and worth knowing. `required` and `min` are checked by Mongoose before the save and come back as a validation error. `unique` creates an index in Mongo, and a duplicate comes back from Mongo as `E11000`. That's why step 13 checks `err.code === 11000` instead of looking for a validation message.

## Redo it

Start from a fresh copy of the [starter](https://github.com/IGM-RichMedia-at-RIT/Simple-MVC-Example) and run `npm install`. `mongoose` is already in `package.json`. The views and `router.js` are finished (`router.js` has the shape from [Express: routes, controllers, and the 404](../express/router-controllers-404.md)). `server/models/Cat.js` is empty and `models/index.js` already exports it. In the controller, `setName` has only the 400 check and `hostPage1` is empty. Make sure Mongo is running, then `npm run nodemon`.

### 1. Run it

Home says `The newest cat is unknown!` and Page 1 spins, because `hostPage1` is empty.

### 2. Connect

In `server/app.js`, under the `router` require and above `const port`:

```js
const router = require('./router.js');   // already there

const dbURI = process.env.MONGODB_URI || 'mongodb://127.0.0.1/simpleMVCExample';
mongoose.connect(dbURI).catch((err) => {
  if (err) {
    console.log('Could not connect to database');
    throw err;
  }
});
```

`mongodb://127.0.0.1/simpleMVCExample` is Mongo on this machine and a database called `simpleMVCExample`. Mongo creates it the first time you save something. The `process.env.MONGODB_URI ||` part is the same trick as `PORT`: in the cloud the real address comes from an environment variable, and on your laptop it falls back to local.

You connect once, here. Every file that requires `mongoose` shares the connection.

Checkpoint: save and wait. The terminal still says `Listening on port 3000` and nothing else. Connecting is silent. If Mongo isn't running, the crash shows up in about 30 seconds.

### 3. The schema, one field

In `server/models/Cat.js`:

```js
const mongoose = require('mongoose');

const CatSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true,
    unique: true,
  },
});
```

`required`: no name, no save. `trim`: strips spaces off both ends. `unique`: no two cats with the same name. Unrealistic for cats, exactly right for usernames.

### 4. Two more fields

Inside the schema, under `name`'s closing `},`:

```js
  },   // already there
  bedsOwned: {
    type: Number,
    min: 0,
    required: true,
  },
  createdDate: {
    type: Date,
    default: Date.now,
  },
```

`Date.now` with no parentheses. You're handing Mongoose the function, and it calls it for each new cat.

### 5. Turn the schema into a model

At the bottom of the file:

```js
const CatModel = mongoose.model('Cat', CatSchema);
module.exports = CatModel;
```

`mongoose.model('Cat', ...)` is the thing you actually use: `new Cat(...)`, `Cat.find(...)`. Mongo stores these in a collection called `cats` (lowercased and pluralized for you).

Now open `server/models/index.js` and look, don't type:

```js
module.exports.Cat = require('./Cat.js');
```

Requiring a folder loads its `index.js`, nothing else. When you add a `Dog.js` for the homework, it needs a line here too.

### 6. Pull `Cat` into the controller

In `server/controllers/index.js`, right under the `models` require:

```js
const models = require('../models');   // already there

const { Cat } = models;
```

Destructuring. It's the same as `const Cat = models.Cat;`. With dogs it becomes `const { Cat, Dog } = models;`.

### 7. Build a cat and look at it

Inside `setName`, after the 400 check's closing `}`:

```js
  }   // already there (end of the 400 check)

  const catData = {
    name: `${req.body.firstname} ${req.body.lastname}`,
    bedsOwned: req.body.beds,
  };

  const newCat = new Cat(catData);
  console.log(newCat);
};   // already there
```

Checkpoint: Page 2, enter `Mr`, `Whiskers`, `3`, submit. The browser spins (nothing responds yet). The terminal shows:

```
{
  name: 'Mr Whiskers',
  bedsOwned: 3,
  _id: new ObjectId('...'),
  createdDate: 2026-10-21T18:16:04.744Z
}
```

Three things to notice. `bedsOwned` is `3`, not `'3'`: forms send strings and the schema cast it. `_id` is Mongo's own id, which you never made. `createdDate` came from the default, in UTC. This is a Mongoose document, not a plain object, and it isn't in the database yet.

### 8. Save it

Make `setName` async by adding one word to its first line:

```js
const setName = async (req, res) => {
```

Then under `console.log(newCat);`:

```js
  console.log(newCat);   // already there

  try {
    await newCat.save();
    return res.status(201).json({
      name: newCat.name,
      beds: newCat.bedsOwned,
    });
  } catch (err) {
    console.log(err);
    return res.status(500).json({ error: 'failed to create cat' });
  }
};   // already there
```

Saving is a request to Mongo's server. Waiting means `await`, and `await` needs `async`. The database can be down, slow, or say no, so the save lives in a `try`. 201 Created, not 200, because you made something new.

### 9. Three submits

Checkpoint: Page 2, `Mr` / `Whiskers` / `3` again. You get `{"name":"Mr Whiskers","beds":3}`.

Submit the same name again. You get `{"error":"failed to create cat"}` and the terminal shows `E11000 duplicate key error ... dup key: { name: "Mr Whiskers" }`.

Submit a cat with `-1` beds. Same 500. The terminal says `Cat validation failed: bedsOwned: Path 'bedsOwned' (-1) is less than minimum allowed value (0)`. You never wrote that check. The schema did it.

### 10. See it in Compass

Compass, `mongodb://localhost:27017`, then `simpleMVCExample`, then `cats`. One document with `_id`, `name`, `bedsOwned`, `createdDate`, and `__v`. `__v` is Mongoose's version counter. Ignore it.

Add another cat on Page 2, then hit refresh in Compass. It doesn't update on its own.

### 11. List the cats on page 1

Replace the empty `hostPage1`. Note the `async`:

```js
const hostPage1 = async (req, res) => {
  try {
    const docs = await Cat.find({}).lean().exec();
    return res.render('page1', { cats: docs });
  } catch (err) {
    console.log(err);
    return res.status(500).json({ error: 'failed to find cats' });
  }
};
```

`find({})` with an empty filter means no conditions: every cat. `.lean()` hands back plain objects instead of Mongoose documents, and Handlebars needs that. `.exec()` runs the query now. It's optional, but Mongoose's error messages are better with it.

Checkpoint: Page 1 shows one bullet per cat, like `Mr Whiskers has 3 beds.` The template hasn't changed since the Handlebars demo. The data just comes from somewhere real now.

### 12. Restart the server

Ctrl+C in the terminal, then `npm run nodemon` again. Refresh Page 1.

Same cats. Last week a restart wiped `name`. Mongo wrote these to disk.

### 13. A real 400 for a taken name (optional)

A duplicate name is the user's fault, not the server's, so it deserves a 400. Inside `setName`'s `catch`, after `console.log(err);` and before the 500 `return`:

```js
  } catch (err) {   // already there
    console.log(err);   // already there
    if (err.code === 11000) {
      return res.status(400).json({ error: 'A cat with that name already exists' });
    }
    return res.status(500).json({ error: 'failed to create cat' });   // already there
  }   // already there
```

It has to come before the 500 line, because once that `return` runs nothing below it does. 11000 is Mongo's code for a duplicate key.

Checkpoint: a duplicate name is now a 400 with the new message. `-1` beds is still a 500, because a validation error has no `code`.

Next: the newest cat on the home page, search by name, and an update. [Mongoose queries](mongoose-queries.md).
