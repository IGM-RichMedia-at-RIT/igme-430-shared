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

## Later in the semester

MVC, templates, databases, React, Socket.IO, and the rest get pages when we get there.
