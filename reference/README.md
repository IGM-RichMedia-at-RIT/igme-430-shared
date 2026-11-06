# IGME-430 Node Reference Guide

This guide is organized by topic, not by week. Use it when you know *what* you need to do ("how do I read the data from a POST?") but not *where* we covered it.

Every page has the same parts:

- **Quick example:** the smallest piece of working code, taken from our class repos
- **Where we did this:** the day, the starter and done repos, the exact file and function, and the video if there is one
- **Common mistakes:** the errors people actually hit in this class
- **What you'll find on Google:** the other ways people solve this online, and why our code looks different
- **Redo it:** short steps that get you from the starter repo to the done repo. Use this if you missed the day, or if you have the finished code but couldn't write it yourself yet. It's the longest part, so it's always last

A note on Google: Node is like any big platform. Search for anything and you'll find ten ways to do it, often with Express or libraries we haven't added, and sometimes ones that are years out of date. They aren't wrong, but they're solving a slightly different problem than your assignment. Start here, then branch out.

Pages get added as the semester goes. If a topic is listed without a link, it doesn't have a page yet.

---

## Getting started (Weeks 1 and 2)

- [Starting a Node project and deploying it](setup/node-project-and-deploy.md): `npm init`, modules, `package.json` scripts, Heroku, and the GitHub Actions check

## HTTP servers (Weeks 2 to 4)

- [The server scaffold](http/server-scaffold.md): the `server.js` setup at the top of every demo (`http.createServer`, the port, parsing the URL, routing)
- [Serving files and MIME types](http/serving-files-mime.md): `readFileSync`, `Content-Type`, and why your CSS won't load
- [Status codes and `respondJSON`](http/status-codes.md): 200 vs 400 vs 404, query parameters, checking the status on the client
- [The `Accept` header](http/accept-header.md): responding with JSON or XML from the same URL
- [HEAD requests](http/head-requests.md): the headers without the body, `curl -I`, and `fetch` with `method: 'HEAD'`
- [Parsing a POST body](http/parsing-post-body.md): chunks, `Buffer.concat`, URL-encoded vs JSON, 201 vs 204

## Tooling (Week 5)

- [nodemon and webpack](tooling/nodemon-webpack.md): automatic restarts, bundling the client, and the scripts in `package.json`
- [Debugging a Node server](tooling/debugging.md): reading a stack trace, the debugger, and the Network tab

## Project 1: the client page

Your main page needs forms that let someone view, filter, and add data without writing any code. Most of the client pieces are already on the pages above, mixed in with the server code. Here's where each one is:

- [Sending form input in a GET](client/form-to-query-string.md): reading a text box and putting it in the URL as `?name=...`, which is how your filtering endpoint gets its query parameters
- Stopping the form's built-in submit with `preventDefault`: [HEAD requests](http/head-requests.md), step 7
- Sending form data in a POST, as URL-encoded or JSON: [Parsing a POST body](http/parsing-post-body.md), "The client side"
- Sending the `Accept` header, and reading `Content-Type` on the response: [The `Accept` header](http/accept-header.md)
- Showing a different message for each status code: [Status codes and `respondJSON`](http/status-codes.md), steps 6 and 7

## Express (Week 8)

Express replaces the hand-written server from Weeks 2 to 4. The same jobs (serve files, parse a body, route a URL, answer a 404) each turn into one line.

- [Express: the app and middleware](express/app-and-middleware.md): `express()`, `listen`, compression, static files, body parsing, favicon, and why the `app.use` order matters
- [Express: routes, controllers, and the 404](express/router-controllers-404.md): the folder map, `router(app)`, `app.get`, exporting controllers, the wildcard and where it has to sit, `res.json`
- [Handlebars templates and `res.render`](express/handlebars-templates.md): turning the engine on, `{{holes}}`, `#if` and `#each`, escaping, and why the data has to be in `render`

## Databases (Week 9)

- [MongoDB and Mongoose: connect, define a model, save](database/mongoose-connect-schema-save.md): the connection string and the env var, the schema, `new Cat(...).save()` in a `try`, reading `E11000`, Compass, and the restart test
- [Mongoose queries: find, search, and update](database/mongoose-queries.md): `findOne`, `sort`, `select`, `lean`, 404 on nothing found, `findOneAndUpdate` with `$inc` and `returnDocument`
- [Setting up MongoDB Atlas](../documents/mongodb-atlas-setup.md): the cloud database for a deployed app. This is a setup doc, not a reference page

## React (Week 10)

- [React: setup, components, and JSX](react/setup-components-jsx.md): the babel rule, `createRoot`, View Source, the JSX rules, components inside components
- [React: props and `useState`](react/props-and-usestate.md): props, the text box that fights you, why a plain variable doesn't work, state, and the whole function running again
- [React: `useEffect` and loading data from the server](react/useeffect-and-loading-data.md): `fetch` inside a component, the infinite loop, `[]` vs `[reload]`, `.map` and `key`

## Desktop apps (Week 11)

- [Electron: the 11B class walkthrough](desktop/electron-11b-walkthrough.md): a window, a locked-down page, preload, React bundled in, talking to a server through the CSP, and packaging a real app. Electron isn't required for any assignment. This page is here so you can recreate the demo, so it's a step-by-step walkthrough instead of the usual format.

## Real-time (Week 11)

- [Socket.IO: the 11C class walkthrough](realtime/socket-io-11c-walkthrough.md): the server talking first, two windows chatting with no refresh, and rooms. Socket.IO isn't required for any assignment. This is my teaching guide, shared for anyone who wants to run the demo start to finish, so it's a step-by-step walkthrough instead of the usual format. The note at the top of the page says how it was made.

## DomoMaker and Project 2: where each piece is

DomoMaker A through D is the Week 8 to 10 material assembled into one app, with login added. Project 2 is the same stack again, on your own idea. Where each piece lives:

- The `app.js` shape in DomoMaker A (compression, static, body parsing, Handlebars, then `router(app)`), plus Helmet and sessions on top: [Express: the app and middleware](express/app-and-middleware.md)
- `router.js`, `controllers/`, and a controller answering with `res.json({ redirect: '/maker' })`: [Express: routes, controllers, and the 404](express/router-controllers-404.md)
- The login, signup, and app pages, served with `res.render`: [Handlebars templates](express/handlebars-templates.md)
- The `Account` and `Domo` models, and saving a new domo inside a `try`: [MongoDB and Mongoose: connect, define a model, save](database/mongoose-connect-schema-save.md)
- Each user seeing only their own domos (`find` by `owner`, `select`, `lean`), and the `_id` that React needs for `key`: [Mongoose queries](database/mongoose-queries.md)
- DomoMaker D's toolchain steps (the babel rule, `.babelrc`, two bundles, `createRoot`): [React: setup, components, and JSX](react/setup-components-jsx.md)
- `App` holding `reloadDomos` in state and handing `triggerReload` down to `DomoForm`: [React: props and `useState`](react/props-and-usestate.md)
- `DomoList` fetching `/getDomos` inside `useEffect` with `[props.reloadDomos]`, then `.map` with a `key`: [React: `useEffect` and loading data](react/useeffect-and-loading-data.md)
- Project 2 needs all of the above plus at least five dynamic React components. The sessions and Redis parts (DomoMaker B and C) don't have pages yet; the assignment docs walk through them. For the Redis Cloud signup, use [Setting up Redis Cloud](../documents/redis-cloud-setup.md).

## Later in the semester

Sessions, Redis, Sass, file uploads, and config get pages when we get there.
