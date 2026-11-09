# React and MongoDB: the 12A class walkthrough

> **A heads-up first.** This is the guide I taught from on Monday, Nov 9 (12A), adjusted so you can follow it on your own. We got the main loop working in class, but we ran out of time before finishing Delete. That's Part 4, so it's here for you to try. There's no video for this day.

**Where we did this:** Week 12, Monday (12A)

- Starter: [React-Mongo-Loop](https://github.com/IGM-RichMedia-at-RIT/React-Mongo-Loop)
- Done: [React-Mongo-Loop-Done](https://github.com/IGM-RichMedia-at-RIT/React-Mongo-Loop-Done) (it matches this page line for line)
- Video: none for this day
- Used in: DomoMaker D and E, Project 2

---

## What this demo shows

Week 9 was Mongo through Handlebars. Week 10 was React against a hardcoded array. This joins them. Using the songs list from Week 10, you read songs from MongoDB, save new ones from a form, make the list update by itself, and then delete one.

The moment to look for is Step 17: you add a song, and it shows up with no page reload and nobody pressing a button. Every feature in your project is the same trip, from the browser to the database and back.

## How to use this page

- You type in three files. Every typing step opens with **Stay in** or **Switch to**: 🟦 `server/controllers/Song.js`, 🟧 `server/router.js`, 🟩 `client/app.jsx`.
- Every typing step ends with **Save →** and what you should see, including "nothing yet."
- You need two terminals. **Terminal 1** runs `webpackWatch` and **Terminal 2** runs `nodemon`. "Terminal 2 restarts" means it prints `[nodemon] restarting due to changes...`, then `Listening on port 3000`. A save in `client/` rebuilds in Terminal 1 and also restarts Terminal 2, because nodemon watches `hosted/`.
- **Nothing refreshes by itself.** After a save, refresh the browser.
- `console.log` in `client/app.jsx` prints in the browser's DevTools. In `server/`, it prints in Terminal 2.
- The comments on the code lines tell you what to do with each line. `// add this` means type it. `// already there` is your anchor, so leave it. `// was ...` means change that line to this. `// delete this` means remove it. Don't type the comments themselves.
- If you only want what DomoMaker D needs, stop after Step 18.

**What runs when:**

| You do this | Client (`app.jsx`) | Server (`Song.js`) |
|---|---|---|
| Load `localhost:3000/` | `SongList`'s effect runs | `appPage`, then `getSongs` |
| Click **Reload songs** (gone after Step 15) | `setReload`, then the effect | `getSongs` |
| Click **Add Song** before Step 12 | `handleSong`, logs only | nothing |
| Click **Add Song** after Step 12 | `handleSong` | `addSong` |
| Click **Delete** (from Step 22) | `deleteSong` | `deleteSong` (same name, different file) |
| Open `localhost:3000/getSongs` | none, raw JSON | `getSongs` |

---

## Start here

### 1. Get the starter

```
git clone https://github.com/IGM-RichMedia-at-RIT/React-Mongo-Loop
cd React-Mongo-Loop
npm install
```

**Check:** `added 330 packages`. Yellow `npm warn EBADENGINE` blocks and `7 moderate severity vulnerabilities` are normal. Don't run `npm audit fix`.

### 2. The loop

```
BROWSER                                 SERVER
1. SongList needs data
2. fetch('/getSongs')           -->     3. getSongs runs
                                        4. Song.find({}) asks MongoDB
6. setSongs(...) redraws list   <--     5. res.json(docs)
```

Week 9 was the right half. Week 10 was the left half. DomoMaker D hands them to you already connected, and here you connect them yourself.

DomoMaker B used `res.render`: the server sends the whole page, once. This uses `res.json`: React asks for just the data, whenever it wants.

### 3. Start MongoDB

Start it the same way you did for DomoMaker A to C.

**Check:** Compass can connect to `mongodb://localhost:27017`.

### 4. Start both watchers

**4a.** In **Terminal 1**:

```
npm run webpackWatch
```

**Check:** `compiled successfully`.

**4b.** In **Terminal 2**:

```
npm run nodemon
```

**Check:** `Listening on port 3000`.

**4c.** Load `localhost:3000/`.

**Check:** a form (Artist, Title, Add Song) above **My favorite songs!**, a **Reload songs** button, and three songs. Open DevTools and keep it open.

### 5. What's already here

- `server/models/Song.js`: a schema, the same shape as Cat from Week 9.
- `server/app.js`: the `mongoose.connect` block and `express.json()`, same as DomoMaker.
- `client/app.jsx`: `SongList` is the `SongContainer` from Week 10, renamed because DomoMaker D calls its version `DomoList`. `SongForm` is the markup, and `handleSong` only logs.

Type anything in the form and click **Add Song**.

**Check:** the console shows `Form submitted: ...`. The form works, it just doesn't go anywhere yet. And those three songs are still a hardcoded array.

---

## Part 1: Read from Mongo

### 6. Bring in the model

🟦 **Switch to `server/controllers/Song.js`.** **Where:** very top of the file.

```js
const models = require('../models');   // add this

const { Song } = models;   // add this

const appPage = (req, res) => {   // already there
```

**Save →** Terminal 2 restarts. The page doesn't change.

### 7. The swap

🟦 **Stay in `Song.js`.** **Where:** inside `getSongs`. Delete the whole `const docs = [ ... ];` array (five lines) and type this one line in its place.

```js
  try {   // already there
    const docs = await Song.find({}).lean().exec();   // was the hardcoded array

    return res.json(docs);   // already there
```

**Save →** Terminal 2 restarts.

**7b.** Refresh `localhost:3000/`.

**Check:** **No Songs Yet!** under the form. One line changed, and not one line of React. It's the same URL and the same shape of answer. It's empty because the database is empty.

If the page goes blank about 10 seconds later, Mongo isn't running. See [If something goes wrong](#if-something-goes-wrong).

### 8. One song, by hand

In Compass, open **ReactMongoLoop**, then **songs** (refresh the database list if it isn't there), then **ADD DATA**, then **Insert document**. Replace everything in the box:

```json
{ "artist": "Men I Trust", "title": "Show Me How" }
```

**8b.** Refresh the browser.

**Check:** **Men I Trust - *Show Me How***.

**8c.** In a new tab, open `localhost:3000/getSongs`.

**Check:** `[{"_id":"6af0...","artist":"Men I Trust","title":"Show Me How"}]`. We didn't make that `_id`. Mongo did. It matters in Step 18.

Go back to the app tab.

### 9. A second song, no page refresh

In Compass, insert `{ "artist": "Washed Out", "title": "Feel It All Around" }`. Don't refresh the browser. Click **Reload songs**.

**Check:** it appears, and the Network tab shows one `getSongs`.

Step 8 needed a refresh because the button isn't drawn while the list is empty. That's the early `return` in `SongList`.

---

## Part 2: Save from a form

### 10. A route for saving

🟧 **Switch to `server/router.js`.** **Where:** under the `/getSongs` line.

```js
  app.get('/getSongs', controllers.Song.getSongs);   // already there
  app.post('/addSong', controllers.Song.addSong);   // add this
```

**Save →** Terminal 2 restarts. Nothing calls it yet.

### 11. Save the song

🟦 **Switch to `server/controllers/Song.js`.** **Where:** inside `addSong`'s `try`. Replace the `return res.status(501)` line with these three.

```js
  try {   // already there
    const newSong = new Song({ artist: req.body.artist, title: req.body.title });   // add this
    await newSong.save();   // add this
    return res.status(201).json({ artist: newSong.artist, title: newSong.title });   // add this (replaces the 501 line)
```

Don't skip that `return`. Without it the song still saves, but the server never answers, so `fetch` waits forever and the list never updates. You won't get an error anywhere.

`201` means "created." DomoMaker D's `makeDomo` ends the same way, with no redirect.

**Save →** Terminal 2 restarts. Nothing calls it yet.

### 12. The form asks the server

🟩 **Switch to `client/app.jsx`.** **Where:** `handleSong`, at the top. Replace the `console.log('Form submitted:'...)` line with this.

```js
  const title = e.target.querySelector('#songTitle').value;   // already there

  await fetch('/addSong', {   // add this (all 5 lines)
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ artist, title }),
  });
```

`JSON.stringify` turns the object into text to send it. This line in `server/app.js` (line 30, already in the starter) turns it back into an object and puts it on `req.body`, before your routes ever run:

```js
app.use(express.json());   // already there
```

`Content-Type` is what tells it to read the body. In Week 4 you checked that header yourselves and called `JSON.parse` by hand. This does that for you.

**Save →** Terminal 1 says `compiled successfully`, and Terminal 2 restarts.

### 13. Saved, but the list doesn't know

Refresh. Type `Rick Astley` and `Never Gonna Give You Up`. Click **Add Song**.

**Check:** the Network tab shows `addSong` with status **201**. The list doesn't change.

**13b.** Refresh Compass.

**Check:** Rick is there.

**13c.** Click **Reload songs**.

**Check:** Rick appears. It saved, and the list just didn't know. We have a button that fixes it, but we're not going to make users press it.

---

## Part 3: Move the state up

The form needs to press that button. But the form and the list are siblings, and neither can reach into the other. So the state moves up to the one component that holds both.

The technical version (React calls this "lifting state up"):

- `useState` is private to the component that calls it. `reload` and `setReload` only exist inside that component.
- The form and the button are JSX inside two different components, `SongForm` and `SongList`, and neither has a reference to the other. React data only moves from parent to child, through props. A child can only send something back up by calling a function its parent passed down.
- So the state moves to their closest shared parent, `App`. `App` passes the value down to the component that reads it (`reload={reload}`) and a function down to the component that changes it (`triggerReload`).
- When that function runs, `App`'s state changes and `App` redraws. `SongList` gets a new `props.reload`, and its `useEffect` runs again because that dependency changed.

```
BEFORE                         AFTER

      App                            App
     /   \                         reload
SongForm X SongList               /      \
Add Song    reload       triggerReload   reload
                                |          |
                            SongForm    SongList
                            Add Song    fetches again

Click Add Song:  1. form calls triggerReload
                 2. App flips reload
                 3. SongList sees the change and fetches
```

### 14. App owns it now

🟩 **Stay in `client/app.jsx`.** **Where:** `App`, near the bottom.

**14a.** One new line, above `App`'s `return`.

```jsx
const App = () => {   // already there
  const [reload, setReload] = useState(false);   // add this

  return (   // already there
```

**14b.** Change the two lines inside the `<div>`.

```jsx
      <SongForm triggerReload={() => setReload(!reload)} />   // was <SongForm />
      <SongList songs={[]} reload={reload} />   // was <SongList songs={[]} />
```

**Save →** Terminal 1 compiles, and Terminal 2 restarts.

**14c.** Refresh.

**Check:** the page looks the same. If it's blank with `reload is not defined` at `App`, 14a is missing. `SongList` still has its own `reload` right now, but `App` can't see it.

### 15. SongList gives its state up

🟩 **Stay in `client/app.jsx`.** **Where:** `SongList`. Three small edits, in this order.

**15a.** The last line of `useEffect`:

```js
  }, [props.reload]);   // was [reload]
```

The list used to watch its own `reload`. Now it watches the one `App` hands it, so `App` decides when the list reloads. After this edit, `SongList`'s own copy does nothing, which is why the next two edits remove it.

**15b.** Delete the button line in the main `return`:

```jsx
      <button onClick={() => setReload(!reload)}>Reload songs</button>   // delete this
```

**15c.** Delete the state line at the top of `SongList`:

```js
  const [reload, setReload] = useState(false);   // delete this
```

**Save →** Terminal 1 compiles, and Terminal 2 restarts.

If you go out of order and delete the state line while 15a still says `[reload]`, you get a blank page and `reload is not defined`. In order, there's no error.

**15d.** Refresh.

**Check:** songs load, no button, no console error. **Add Song** still saves, but the list won't update until Step 16.

### 16. The form presses it

🟩 **Stay in `client/app.jsx`.** Three small edits, in this order.

**16a. Where:** the `<form>` line in `SongForm`. Hand the function along.

```jsx
    <form id="songForm" onSubmit={(e) => handleSong(e, props.triggerReload)}>   // was handleSong(e)
```

`onSubmit` is the same idea as `onChange` from Week 10, just a different event. The new part is `props.triggerReload`. `App` handed the form a function, and we pass it along so `handleSong` can call it when the save is done.

**16b. Where:** the first line of `handleSong`, at the top of the file. Give that function a name here.

```js
const handleSong = async (e, onSongAdded) => {   // was async (e)
```

**16c. Where:** `handleSong`, right under the fetch. Call it.

```js
  });   // already there (the end of the fetch)

  onSongAdded();   // add this
```

It's one function, passed along twice. `App` writes it, `SongForm` gets it as `props.triggerReload`, and `handleSong` gets it as `onSongAdded`. Calling it runs `setReload(!reload)` back in `App`. `onSongAdded` isn't built in. It's just `handleSong`'s name for it, the same shape as DomoMaker D's `handleDomo(e, onDomoAdded)`.

**Save →** Terminal 1 compiles, and Terminal 2 restarts.

### 17. The payoff

Refresh. Type `Khruangbin` and `White Gloves`. Click **Add Song**.

**Check:** it appears right away, with no page reload. The Network tab shows `addSong` 201, then `getSongs` 200.

Nobody clicked reload. The form flipped a true/false that the list is watching. The value doesn't mean anything, it only has to change. That's `reloadDomos` in DomoMaker D.

### 18. Add the same song twice

Click **Add Song** again (the boxes are still filled).

**Check:** the console shows, in red, `Encountered two children with the same key`.

**18b.** 🟩 **Stay in `client/app.jsx`.** **Where:** the `.map` in `SongList`.

```jsx
      <div key={song._id}>   // was key={song.title}
```

**Save →** Terminal 1 compiles, and Terminal 2 restarts.

**18c.** Refresh.

**Check:** both rows are there, with no warning.

DomoMaker D gives you `key={domo.id}`. That's undefined, because the field is `_id`. D's webpack also runs in production mode, which hides this warning, so nothing will tell you. Change it to `_id`.

**This is the stopping point if you only need DomoMaker D.** Part 4 is the extra.

---

## Part 4: Delete

We didn't finish this in class. DomoMaker never builds delete, and your project probably needs it. It's the same stops a third time.

### 19. A route for deleting

🟧 **Switch to `server/router.js`.** **Where:** under `/addSong`.

```js
  app.post('/addSong', controllers.Song.addSong);   // already there
  app.post('/deleteSong', controllers.Song.deleteSong);   // add this
```

**Save →** Terminal 2 restarts. Nothing calls it yet.

### 20. Delete it from Mongo

🟦 **Switch to `server/controllers/Song.js`.** **Where:** inside `deleteSong`'s `try`, the last function. Replace the `return res.status(501)` line with these two.

```js
const deleteSong = async (req, res) => {   // already there
  try {   // already there
    await Song.deleteOne({ _id: req.body.id }).exec();   // add this
    return res.status(200).json({ message: 'Song deleted' });   // add this (replaces the 501 line)
```

Same trap as Step 11: don't skip that `return`. Without it the song still gets deleted, but the server never answers, so `fetch` waits forever and the list doesn't update.

**Save →** Terminal 2 restarts. Nothing calls it yet.

### 21. Ask the server to delete

🟩 **Switch to `client/app.jsx`.** **Where:** outside every component, in the gap between `handleSong`'s closing `};` and the line `const SongForm = (props) => {`. Not inside `SongForm`, or `SongList` can't see it.

```js
const deleteSong = async (id, onSongDeleted) => {   // add this (the whole function)
  await fetch('/deleteSong', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ id }),
  });

  onSongDeleted();
};

const SongForm = (props) => {   // already there. The new function goes ABOVE this line
```

That's the second time we've typed that fetch. DomoMaker D moves it into `helper.sendPost` for exactly this reason.

**Save →** Terminal 1 compiles, and Terminal 2 restarts. Nothing calls it yet.

### 22. The button

🟩 **Stay in `client/app.jsx`.** Two edits.

**22a. Where:** the `.map`, under the `<h2>`.

```jsx
        <h2>{song.artist} - <i>{song.title}</i></h2>   // already there
        <button onClick={() => deleteSong(song._id, props.triggerReload)}>Delete</button>   // add this
```

**22b. Where:** `App`. Add one prop to `SongList`.

```jsx
      <SongList songs={[]} reload={reload} triggerReload={() => setReload(!reload)} />   // was <SongList songs={[]} reload={reload} />
```

`reload={reload}` has been there since Step 14. It's how the list knows when to fetch. `triggerReload` is the new part: the same function the form got, handed to the list too, because the Delete buttons live in the list. So the list watches `reload`, and now it can also flip it.

**Save →** Terminal 1 compiles, and Terminal 2 restarts.

### 23. Delete one

Refresh. Click a **Delete**.

**Check:** the row disappears. The Network tab shows `deleteSong` 200, then `getSongs` 200. Refresh Compass, and it's gone.

---

## Part 5: Whose data, and the map to DomoMaker D

### 24. The filter is the design decision

```
Song.find({})                   everyone's songs (this demo)
Domo.find({ owner })            only mine (DomoMaker D)
Post.find({ isPublic: true })   only some (Project 2)
```

The loop never changes. "Data only some people can see" on the Project 2 spec comes down to this one line.

Right now anyone can delete anything. Once you have accounts, it's `deleteOne({ _id, owner })`, so you can only delete your own.

### 25. These lines in DomoMaker D

| This demo | DomoMaker D |
|---|---|
| `Song.find({}).lean().exec()` | `Domo.find(query).select('name age').lean().exec()` |
| `res.json(docs)`, client uses `data` | `res.json({ domos: docs })`, client uses `data.domos` |
| `handleSong` + `fetch` | `handleDomo` + `helper.sendPost` |
| `App`'s `reload` | `App`'s `reloadDomos` |
| `triggerReload` | `triggerReload` |
| `[props.reload]` | `[props.reloadDomos]` |
| `key={song._id}` | gives you `key={domo.id}`, change it |
| Delete button | not there, a good pick for E Part 2 |

---

## Test your query in Compass first

Get the right rows in Compass, then paste the query into your code. It's easy because a Mongo query is a plain object, so there's no string to build.

1. In Compass, type the filter in the top box beside the clock icon (it shows `{}`), and click **Find**. Don't use the **Project** box for this. A value there returns every row and makes them all display that value.
2. Copy that object into `find(...)`.
3. Swap the fixed value for a variable.

```js
// Compass, top box
{ artist: "Men I Trust" }

// In the controller
const docs = await Song.find({ artist: "Men I Trust" }).lean().exec();

// With a variable, and no string concatenation
const docs = await Song.find({ artist: req.query.artist }).lean().exec();
```

**Songs added in March, newest first.** In Compass:

- Top box: `{ createdDate: { $gte: new Date("2026-03-01"), $lt: new Date("2026-04-01") } }`
- Options, then Sort: `{ createdDate: -1 }`
- Options, then Limit: `5`

```js
const march = { createdDate: { $gte: new Date('2026-03-01'), $lt: new Date('2026-04-01') } };
const docs = await Song.find(march).sort({ createdDate: -1 }).limit(5).lean().exec();
```

For "*my* favorites added in March," add `owner: req.session.account._id` to that same object, the way DomoMaker D's `getDomos` does.

**If you've used SQL:**

| SQL | Compass | Mongoose |
|---|---|---|
| `WHERE artist = 'X'` | top box: `{ artist: "X" }` | `.find({ artist: 'X' })` |
| `WHERE artist = 'X' AND title = 'Y'` | top box: `{ artist: "X", title: "Y" }` | same object in `.find` |
| `WHERE artist LIKE '%men i%'` | top box: `{ artist: /men i/i }` | same object in `.find` |
| `WHERE d >= a AND d < b` | top box: `{ d: { $gte: a, $lt: b } }` | same object in `.find` |
| `SELECT artist, title` | Project: `{ artist: 1, title: 1 }` | `.select({ artist: 1, title: 1 })` |
| `ORDER BY createdDate DESC` | Sort: `{ createdDate: -1 }` | `.sort({ createdDate: -1 })` |
| `LIMIT 5` | Limit: `5` | `.limit(5)` |

**What trips people up:**

- Matching is case-sensitive. `{ artist: "men i trust" }` finds nothing. Use `/men i/i` for a loose match.
- For dates, use `new Date("...")`, which works in both. `ISODate("...")` works in Compass but doesn't exist in Node. A plain string like `"2026-03-01"` finds nothing in Compass but works in Mongoose, because Mongoose converts it using the schema.
- Documents typed in by hand through Compass have no `createdDate`, because the schema's default only runs when Mongoose saves. They won't show up in date filters.

More on queries: [Mongoose queries](../database/mongoose-queries.md).

---

## Where this goes

This song list isn't a project, the same way DomoMaker isn't. Copy the pattern all you want. But it's about two of your five components, with no accounts, no private data, and no reason for anyone to use it yet. If you start from this repo, say so in your documentation.

---

## If something goes wrong

- **Blank page about 10 seconds after Step 7, `songs.map is not a function`, `/getSongs` is 500, Terminal 2 says `buffering timed out after 10000ms`:** Mongo isn't running. At 30 seconds Terminal 2 crashes with `Could not connect to database`. Start Mongo, then run `npm run nodemon` again.
- **Blank page right away, same `songs.map` error, Terminal 2 says `ReferenceError: Song is not defined`:** Step 6 is missing.
- **Blank page, 404 for `appBundle.js`:** Terminal 1 isn't running.
- **Blank page, `reload is not defined` at `App`:** Step 14a's `useState` line is missing.
- **Blank page, `reload is not defined` at `SongList`:** 15c was done, 15a wasn't.
- **Compass has no `ReactMongoLoop`:** refresh Compass. It appears once the server has run.
- **`addSong` or `deleteSong` is 404:** Step 10 or 19 is missing.
- **Console says `deleteSong is not defined` at `onClick`:** Step 21's function went inside `SongForm`. Move it out to the top level, above `const SongForm`.
- **`addSong` is 501, `Not built yet`:** Step 11 wasn't saved.
- **The song is in Compass, the list doesn't update, no error, and `addSong` is stuck on (pending) in the Network tab:** Step 11's `return res.status(201)` line is missing, so the server never answers. Same for `deleteSong` and Step 20.
- **`addSong` is 500, Terminal 2 says `reading 'artist'`:** the `Content-Type` line is missing.
- **`addSong` is 400 in red:** a box was empty. That's expected.
- **Saved, the list doesn't update, console says `onSongAdded is not a function`:** the function isn't reaching `handleSong`. Step 14b or 16a is missing.
- **Saved, the list doesn't update, console says `onSongAdded is not defined`:** 16b is missing.
- **Saved, the list doesn't update, no error:** Step 15a says `[]`, or 16c is missing.
- **The row is gone from Mongo but still on the page, `onSongDeleted is not a function`:** Step 22b is missing.
- **Typed in `app.jsx`, nothing changed:** refresh the browser, then check Terminal 1 for a syntax error.

---

## Questions that come up

**Couldn't I just list these with Handlebars?** Yes, but the list would only update on a page reload, and it wouldn't count toward your five React components.

**Why is `reload` a true/false?** The value doesn't matter. It only has to change, because the effect runs again when `props.reload` is different from last time.

**Why not `app.delete`?** Express has it, and you can use it. Our class repos stick to GET and POST, and that's what Project 2 requires.

**Why fetch the whole list again after a save?** It's simple, and it's what DomoMaker D does. You could update state straight from the response instead, like `setSongs([...songs, newSong])`, and skip the second request.

**What do bigger React apps do?** A true/false works fine at this size. Bigger apps often use a data-fetching library like TanStack Query or SWR, where you name each kind of data and mark that name as stale after a save. You don't need one for Project 2, and learning one on your own would be a fair above-and-beyond idea.

**Related pages:** [React: props and `useState`](props-and-usestate.md), [React: `useEffect` and loading data](useeffect-and-loading-data.md), [Parsing a POST body](../http/parsing-post-body.md).

---

## The finished code

The done repo matches this page exactly, so here's where to look:

- [`client/app.jsx`](https://github.com/IGM-RichMedia-at-RIT/React-Mongo-Loop-Done/blob/master/client/app.jsx)
- [`server/controllers/Song.js`](https://github.com/IGM-RichMedia-at-RIT/React-Mongo-Loop-Done/blob/master/server/controllers/Song.js)
- [`server/router.js`](https://github.com/IGM-RichMedia-at-RIT/React-Mongo-Loop-Done/blob/master/server/router.js)
