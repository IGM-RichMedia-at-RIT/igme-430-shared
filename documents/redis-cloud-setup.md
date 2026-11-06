# Setting Up Redis Cloud (For DomoMaker C)

DomoMaker C keeps your sessions in Redis, so your app needs a Redis database it can reach from your laptop and from Heroku. That's Redis Cloud. You make one free database and end up with one URL, `REDISCLOUD_URL`. You can keep using that same URL for the rest of the DomoMakers and Project 2, and you only get one free database per account anyway.

The Redis Cloud screens changed after the older guide on myCourses was made, so if something looks different from that guide, go with this one.

**Already have a database and just need the URL?** Skip to [Part 2](#part-2-get-your-url).

---

## Part 1: Make the free database

1. Go to [redis.io/try-free](https://redis.io/try-free/) and sign up with Google. Use your RIT account in the `@g.rit.edu` format.
2. On **Choose your plan**, ignore the two big boxes, **Essentials** and **Pro**. Those are paid. The free plan is a thin strip underneath them on the left that says "Not ready? **Try 30 MB for free**." It's easy to scroll right past. Click it. If it's already outlined in blue, it's already selected.

   If you ever see a credit card form, you're on a paid plan. Back out.

   If you made an account earlier and don't see this page, go to [cloud.redis.io](https://cloud.redis.io) and click **New database** first.
3. Redis makes up a database name for you. Keep it or change it, anything works. The defaults for cloud vendor, region, and version are fine.
4. Click **Create database**. When the status icon goes from an orange clock to a green check, you're ready.

---

## Part 2: Get your URL

You need two things from your database's **Configuration** tab. They're in two sections, one right above the other.

### 1. The public endpoint (in the Access section)

The older guide said this was under General. Now it's under **Access**. Find **Public endpoint** and click the copy icon next to it. It looks something like this, though yours may start with `redis-` instead:

```
mouth-grip-feeling-12345.db.redis.io:12345
```

The `:12345` at the end is part of it. You need that too.

### 2. The password (in the Security section)

The password isn't sitting on the page anymore. In **Security**, click **Default user** (or **Configure** next to it). A box pops up with the username, `default`, and a hidden password. Click the eye to see it or the copy icon to copy it, and save it somewhere safe. Then click **Cancel** so nothing changes.

### 3. Put them together

```
redis://default:YOUR_PASSWORD@YOUR_PUBLIC_ENDPOINT
```

So you end up with something like this:

```
redis://default:aBcD1234eFgH5678@mouth-grip-feeling-12345.db.redis.io:12345
```

`default` is the real username, so type that word. No `< >` around the password, and no quotes around the whole thing.

### What about the blue Connect button?

You don't need it if you built the URL above. It opens a box with Redis Insight, Redis CLI, and Redis Client.

The older guide has you open **Redis CLI**, copy the command, and delete `redis-cli -u ` from the front. That should leave you with the same URL, so if you already did it that way, you're fine. Just check that the password in it is your real one and not stars or `<password>`.

Skip **Redis Client**. Its Node.js option shows sample code with the host and port split apart. Don't paste that into `app.js`. The DomoMaker code reads one URL from `process.env.REDISCLOUD_URL`.

---

## Part 3: Put it in your app

**On your machine:** this is step 3 of DomoMaker C. In a file named `.env` next to `package.json`:

```
REDISCLOUD_URL=redis://default:aBcD1234eFgH5678@mouth-grip-feeling-12345.db.redis.io:12345
```

**On Heroku:** this is step 13. Open your app, go to **Settings**, click **Reveal Config Vars**, and add the key `REDISCLOUD_URL` with the same URL as the value.

Unlike Atlas, there's no IP address list to set up. The same URL works from your laptop and from Heroku.

---

## If it still won't connect

| What you see | What it usually means |
|---|---|
| `ECONNREFUSED` with `6379` in it | Your code never got the URL, so it's looking for Redis on your own machine. Check that `.env` is next to `package.json`, the name is exactly `REDISCLOUD_URL`, and `require('dotenv').config();` is the very first line of `app.js` |
| `WRONGPASS invalid username-password pair` | Wrong password, or the `< >` is still around it. Copy it again from Part 2 |
| `NOAUTH HELLO must be called...` | `default:` is missing from the URL |
| `is not a valid Redis protocol` | The URL doesn't start with `redis://` |
| `TypeError: Invalid URL` | Something extra got into the URL, usually quotes around it in Heroku's config var |
| `ENOTFOUND` | The endpoint part is wrong. Copy it again from the Access section |
| Works locally, crashes on Heroku | `REDISCLOUD_URL` is missing or misspelled in your Heroku config vars |

If a screen doesn't match this page, post a screenshot in Slack and I'll fix the page.
