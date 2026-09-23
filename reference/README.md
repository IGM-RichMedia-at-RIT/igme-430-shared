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

## HTTP servers (Weeks 2 to 4)

- The server scaffold: the `server.js` setup at the top of every demo (`http.createServer`, the port, parsing the URL, routing)
- Serving files and MIME types (`Content-Type`, `fs.readFile`)
- [Status codes and `respondJSON`](http/status-codes.md): 200 vs 400 vs 404, query parameters, checking the status on the client
- [The `Accept` header](http/accept-header.md): responding with JSON or XML from the same URL
- HEAD requests (headers, no body)
- [Parsing a POST body](http/parsing-post-body.md): chunks, `Buffer.concat`, URL-encoded vs JSON, 201 vs 204

## Tooling (Week 5)

- nodemon and webpack
- Debugging a Node server

## Later in the semester

MVC, templates, databases, React, Socket.IO, and the rest get pages when we get there.
