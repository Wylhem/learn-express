# Hello, One-To-Many Relationships

# Objectives:

By the end of this chapter, you should be able to:

- Build a simple API for a 1:M application
- Execute SQL to create tables and seed a database

# A review of one-to-many

With a one-to-many it's very common to have a foreign key in one table which corresponds to the `id` of another table. You'll very commonly hear that one item in a table `has many` of an item in another table. The inverse of that is one item `belongs to` another item in a table.

# An example

Let's get started working on a one to many API! Let's start in terminal:

```sh
createdb grads-offers-node
mkdir grads-offers-node
cd grads-offers-node
touch app.js
npm init -y
npm install express morgan body-parser pg
mkdir routes db
touch db/index.js
touch routes/grads.js
touch routes/offers.js
psql grads-offers-node
```

Now in `psql`:

```sh
CREATE TABLE graduates (id SERIAL PRIMARY KEY, name TEXT);

CREATE TABLE offers (id SERIAL PRIMARY KEY, title TEXT, graduate_id INTEGER REFERENCES graduates (id) ON DELETE CASCADE);

INSERT INTO gradates (text) VALUES ('Elie'), ('Michael'), ('Matt'), ('Joel');

INSERT INTO offers (name) VALUES ('Teacher', 1), ('Super Teacher', 2), ('Mathematician', 3), ('Developer', 4), ('Super Doctor 1', 3), ('Super Doctor 2', 4), ('Super Developer 1', 2);

\q
```

Let's set up our `index.js` to have the following:

```js
const { Client } = require("pg");

const client = new Client({
  connectionString: "postgresql://localhost/grads-offers-node"
});

client.connect();

module.exports = client;
```

Now in our `routes/grads.js`

```js
const express = require("express");
const router = express.Router();
const db = require("../db");

router.get("/", async (req, res, next) => {
  try {
    const result = await db.query("SELECT * FROM graduates");
    return res.json(result.rows);
  } catch (e) {
    return next(e);
  }
});

router.post("/", async (req, res, next) => {
  try {
    const result = await db.query(
      "INSERT INTO graduates (name) VALUES ($1) RETURNING *",
      [req.body.name]
    );
    return res.json(result.rows[0]);
  } catch (e) {
    return next(e);
  }
});

router.patch("/:id", async (req, res, next) => {
  try {
    const result = await db.query(
      "UPDATE graduates SET name=$1 WHERE id=$2 RETURNING *",
      [req.body.name, req.params.id]
    );
    return res.json(result.rows[0]);
  } catch (e) {
    return next(e);
  }
});

router.delete("/:id", async (req, res, next) => {
  try {
    const result = await db.query(
      "DELETE FROM graduates WHERE id=$1 RETURNING *",
      [req.params.id]
    );
    return res.json(result.rows[0]);
  } catch (e) {
    return next(e);
  }
});

module.exports = router;
```

And in our `app.js`

```js
const express = require("express");
const app = express();
const bodyParser = require("body-parser");
const morgan = require("morgan");
const gradRoutes = require("./routes/gradRoutes");

app.use(morgan("tiny"));
app.use(bodyParser.json());
app.use("/graduates", gradRoutes);

// catch 404 and forward to error handler
app.use((req, res, next) => {
  var err = new Error("Not Found");
  err.status = 404;
  return next(err);
});

// development error handler
// will print stacktrace
if (app.get("env") === "development") {
  app.use((err, req, res, next) => {
    res.status(err.status || 500);
    return res.json({
      message: err.message,
      error: err
    });
  });
}

app.listen(3000, () => {
  console.log("Getting started on port 3000!");
});
```

# Adding the One to Many Routes:

Now in our `routes/offers.js`, we're going to make a route that displays all of the offers for a specific graduate. Before we write that code let's think how we can get all of the offers for a specific graduate. Here are some options:

1. Join the offers table with the graduates wherever the graduate_id is the same as the id in the URL
2. Make two queries:
- Get the specific graduate based on the id in the URL
- Get the offers where the graduate_id is the same as on the one in the URL

There is no one right answer here, but it's usually more efficient to make two separate queries rather than joining the tables together, and that's what we'll do. We'll then take all of the data we get back from our second query and add it to the result of the first query.

Now let's add the following in our `routes/offers.js`

```js
const express = require("express");
const router = express.Router({ mergeParams: true });
const db = require("../db");

app.get("/graduates/:id/offers", async (req, res, next) => {
  try {
    // Get the specific graduate based on the id in the URL
    const graduate = await db.query("SELECT * FROM graduates WHERE id=$1", [
      req.params.id
    ]);
    // Get all the offers where the graduate_id is the same as on the one in the URL
    const offers = await db.query(
      "SELECT company,title FROM offers WHERE graduate_id=$1",
      [req.params.id]
    );
    // set a property on graduate.rows[0] (which is the specific grad found) called offers
    // the value of the offers property will be an array of offers we get back from the 2nd query
    graduate.rows[0].offers = offers.rows;
    return res.json(graduate.rows[0]);
  } catch (e) {
    return next(e);
  }
});

// Here we are just adding another route/endpoint to add an offer for a specific grad
app.post("/graduates/:id/offers", async (req, res, next) => {
  try {
    const graduate = await db.query(
      "INSERT INTO offers (company, title, graduate_id) VALUES ($1, $2, $3)",
      [req.body.company, req.body.title, req.params.id]
    );
    // depending on what we want our API to respond with, we might need to make some additional queries, or we can just send back a simple message.
    return res.json({ message: "Created" });
  } catch (e) {
    return next(e);
  }
});

module.exports = router;
```

And now in our app.js, let's add those routes:

```js
const express = require("express");
const app = express();
const bodyParser = require("body-parser");
const morgan = require("morgan");
const gradRoutes = require("./routes/gradRoutes");
const offerRoutes = require("./routes/offerRoutes");

app.use(morgan("tiny"));
app.use(bodyParser.json());
app.use("/graduates", gradRoutes);
app.use("/graduates/:id/offers", offerRoutes);

// catch 404 and forward to error handler
app.use((req, res, next) => {
  var err = new Error("Not Found");
  err.status = 404;
  return next(err);
});

// development error handler
// will print stacktrace
if (app.get("env") === "development") {
  app.use((err, req, res, next) => {
    res.status(err.status || 500);
    return res.json({
      message: err.message,
      error: err
    });
  });
}

app.listen(3000, () => {
  console.log("Getting started on port 3000!");
});
```

# Your Turn!

Add the routes for showing a specific offer, updating an offer, and deleting an offer. You can respond with whatever data you think is the most pertinent. The routes should be:

- `GET /graduates/:graduate_id/offers/:id`
- `PATCH /graduates/:graduate_id/offers/:id`
- `DELETE /graduates/:graduate_id/offers/:id`

When you're ready, move on to [Many-To-Many Relationships](./09-many-to-many.md)