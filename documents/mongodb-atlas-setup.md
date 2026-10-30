# Setting Up MongoDB Atlas (Your Cloud Database)

Your app uses a local MongoDB while you develop it. Once it's on Heroku, it needs a database in the cloud. That's MongoDB Atlas. You make one free cluster and use it for Simple Models, every DomoMaker, and Project 2. Each assignment just gets its own database name.

**Already started and got stuck?** Skip to [Part 2](#part-2-get-your-connection-string). Most people who get lost clicked Edit Config instead of Connect, or Atlas did part of the setup for them without saying much.

The Atlas screens changed after the older PDF guide was made, so if something looks different from that guide, go with this one.

---

## Part 1: First time only, make the cluster

1. Go to [mongodb.com/cloud/atlas/register](https://www.mongodb.com/cloud/atlas/register) and choose **Sign in with Google**. Use your RIT account in the `@g.rit.edu` format.
2. Accept the privacy policy and terms.
3. On the "Welcome to Atlas" questions, you can click **Skip personalization**.
4. On **Deploy your cluster**, pick **Free**. Leave the name as `Cluster0`, the provider as AWS, and the region as it is. Click **Create Deployment**.
5. A **Connect to Cluster0** box should open. Add `0.0.0.0/0` as an IP address, create a database user, and **copy that password somewhere safe**. You need it in Part 2. Then click **Choose a connection method** and pick up at Part 2, step 4.

If the box never opened, or you closed it, that's fine. Part 2 starts from the main Atlas page.

> If you left **Automate security setup** checked when you made the cluster, Atlas added your user and your current IP address for you. That only covers the network you were on at the time, so do [Part 3](#part-3-let-heroku-in-00000) too.

---

## Part 2: Get your connection string

### 1. Go to Clusters

Click **Clusters** in the left sidebar. Your cluster has a **Connect** button next to its name.

![Clusters page with the Connect button](img/mongodb-atlas/01-clusters-page.jpg)

Two buttons on this page are not what you want:

- **Edit Config** (top right) takes you to "Editing Cluster0," which is for changing your plan. If you end up there, click **← Back**.
- **+ Create** makes a *second* cluster, and you only get one free one. The second one defaults to a paid plan. If you see the page below, back out.

![Editing Cluster0, the wrong page](img/mongodb-atlas/02-edit-config-wrong-page.png)

![The Create page defaults to a paid cluster once you have a free one](img/mongodb-atlas/10-create-is-paid.jpg)

### 2. Click Connect

### 3. Step 1 of the box: connection security

If you see green checks like this, Atlas already set up your IP address and a database user. Click **Choose a connection method**.

![Connection security already done by Auto Setup](img/mongodb-atlas/03-connect-security-auto.jpg)

Two things to know if you see this screen:
- It added **only the IP address you were on**. Heroku isn't on it. Do [Part 3](#part-3-let-heroku-in-00000) before you deploy.
- If you never saw or saved the password for that user, do [Part 4](#part-4-dont-know-your-database-password).

If instead it asks you to add an IP address and create a user, do both (use `0.0.0.0/0` for the IP) and save the password.

### 4. Pick Drivers and Client Libraries

It's the first option, under "Connect to your application."

![Choose a connection method](img/mongodb-atlas/04-choose-connection-method.jpg)

### 5. Copy the connection string

The older guide called this "Select your driver and version." Now it's two dropdowns:

- **Language:** JavaScript
- **Client Library:** Mongoose (Node.js Driver gives you the exact same string, so either is fine)

Leave **Enable network compression** unchecked and **Legacy URI String** off. Click the copy button next to the string.

![The connection string](img/mongodb-atlas/05-connection-string.jpg)

### 6. Fill in your password and a database name

The string you copied looks like this:

```
mongodb+srv://yourname_db_user:<db_password>@cluster0.abc123.mongodb.net/
```

- Replace `<db_password>` with your password, and **delete the `< >`** too.
- Add a database name to the very end. Use the assignment's name, and change it for each assignment so they don't step on each other.

```
mongodb+srv://yourname_db_user:yourPassword123@cluster0.abc123.mongodb.net/SimpleModels
```

For DomoMaker, the end becomes `/DomoMaker` instead. Same cluster, different database.

The older guide's strings end in `?appName=Cluster0` and tell you to put the name before the `?`. Atlas doesn't add that anymore. If yours does have a `?`, put the name right before it.

---

## Part 3: Let Heroku in (0.0.0.0/0)

Atlas only accepts connections from IP addresses on its list. Heroku doesn't have a fixed IP address on the plan we use, so we allow every address with `0.0.0.0/0`. Normally that would be a bad idea, since it means anyone with your username and password can connect. For class projects, it's what we need.

In the left sidebar, open **Database & Network Access** (under Security), then **IP Access List**.

If you only see one address that says "Created as part of the Auto Setup process," your app works from that network and nowhere else. That includes Heroku, and it includes your own laptop once you're home.

![Only the Auto Setup address is on the list](img/mongodb-atlas/06-ip-access-list-auto.jpg)

Click **+ Add IP Address**, type `0.0.0.0/0` into **Access List Entry**, add a comment like `Heroku`, and click **Confirm**. The yellow warning is expected. Leave "This entry is temporary" off.

![Adding 0.0.0.0/0](img/mongodb-atlas/07-add-ip-anywhere.jpg)

It takes a minute or two to go live.

---

## Part 4: Don't know your database password?

This happens when Atlas made the user for you. You can't look the old password up, but you can set a new one.

1. In **Database & Network Access**, open **Database Users** and click **Edit** next to your user.

   ![Database Users](img/mongodb-atlas/08-database-users.jpg)

2. Click **Edit Password**, then **Autogenerate Secure Password**, then **Copy**. Save it somewhere.

   ![Edit Password and Autogenerate](img/mongodb-atlas/09-edit-user-password.jpg)

3. Scroll down and click **Update User**.

Then update the password in your connection string everywhere you used it, including Heroku.

Use the autogenerated password, or at least one with only letters and numbers. A password with `@`, `:` or `/` in it breaks the connection string, and the error (`querySrv ENOTFOUND`) points nowhere near the password.

---

## Part 5: Put it in Heroku

1. Open your app on the Heroku dashboard and go to **Settings**.
2. Click **Reveal Config Vars**.
3. Add a config var with the key `MONGODB_URI` (that's a capital **I** at the end, not an L). For the value, paste your whole connection string with your password and database name filled in.

Your code reads it as `process.env.MONGODB_URI`.

---

## Optional: look at your data in Compass

Compass is a viewer for MongoDB databases. Open it, start a new connection, paste the same connection string, and click **Connect**. You can also get there from the Connect box in Part 2: pick **Compass** instead of Drivers.

---

## If it still won't connect

| What you see | What it usually means |
|---|---|
| `bad auth : authentication failed` | Wrong password, or the `< >` is still around it. Atlas can also take a few minutes after a change |
| An error about your IP not being on the list (or a timeout) | Do Part 3 |
| Worked at RIT, fails at home | Atlas only had your RIT IP address. Do Part 3 |
| `querySrv ENOTFOUND` | Your password has `@`, `:` or `/` in it. Do Part 4 |
| Works locally, fails on Heroku | `MONGODB_URI` is missing or misspelled in Heroku, or Part 3 isn't done |
