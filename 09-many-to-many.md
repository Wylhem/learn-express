# Hello, Many-To-Many Relationships.

# Objectives:

By the end of this chapter, you should be able to:

- Build a simple API for a M:M application
- Transform data returned from the pg module into an API response

# A review of many-many

With a many-to-many association, it is not possible to place a foreign key in either of the tables (that would create a tremendous amount of duplication). Instead, we create a third table, which is often called a `join table` or `through table`. A `join table` consists of two foreign keys, each corresponding to their own table. When collecting data from a many to many across multiple tables a join is required and depending on the amount of data you want, you can select a LEFT JOIN, INNER JOIN, RIGHT JOIN, or FULL JOIN.

# An example

Let's get started working on a many to many API! We're going to build a simple application that stores messages and tags. Many different messages can have many different tags and vise versa. Let's get started in Terminal:

```sh
createdb messages-tags-node
mkdir messages-tags-node
cd messages-tags-node
touch app.js
npm init -y
npm install express morgan body-parser pg
mkdir routes db
touch db/index.js
touch routes/messages.js
touch routes/tags.js
psql messages-tags-node
```

```sql
CREATE TABLE messages (id SERIAL PRIMARY KEY, text TEXT);

CREATE TABLE tags (id SERIAL PRIMARY KEY, name TEXT);

CREATE TABLE messages_tags (id SERIAL PRIMARY KEY, message_id INTEGER REFERENCES messages (id) ON DELETE CASCADE, tag_id INTEGER REFERENCES tags (id) ON DELETE CASCADE);

INSERT INTO messages (text) VALUES ('first'), ('second'), ('third');

INSERT INTO tags (name) VALUES ('funny'), ('happy'), ('silly');

INSERT INTO messages_tags (message_id, tag_id) VALUES (1,1), (1,2), (2,1), (2,3), (3,3);

\q
```

Let's set up our `index.js` to have the following:

```js
const { Client } = require("pg");

const client = new Client({
  connectionString: "postgresql://localhost/messages-tags-node"
});

client.connect();

module.exports = client;
```

Now in our `routes/tags.js` we're going to make two simple endpoints for reading all the tags and creating a tag.

```js
const express = require("express");
const router = express.Router();

router.get("/", async (req, res, next) => {
  try {
    const data = await db.query("SELECT * FROM tags");
    res.json(data.rows);
  } catch (err) {
    return next(err);
  }
});

router.post("/", async (req, res, next) => {
  try {
    const result = await db.query(
      "INSERT INTO tags (name) VALUES ($1) RETURNING *",
      [req.body.name]
    );
    res.json(result.rows[0]);
  } catch (err) {
    return next(err);
  }
});

module.exports = router;
```

And in our `app.js` let's include those routes as well as middleware and an error handler.

```js
const express = require("express");
const app = express();
const bodyParser = require("body-parser");
const morgan = require("morgan");
const tagRoutes = require("./routes/tagRoutes");

app.use(morgan("tiny"));
app.use(bodyParser.json());
app.use("/tags", tagRoutes);

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

# Adding the Many to Many Routes:

What we've built so far is pretty standard, now it's time to join our tables together so that we can display all of the tags for all of the messages. The SQL we need to write looks like this:

```sql
SELECT m.id, m.text, t.name
     FROM messages m
       JOIN messages_tags mt ON m.id=mt.message_id
       JOIN tags t ON mt.tag_id = t.id
     ORDER BY m.id;
```

Instead of selecting all the columns, we are just going to the get the message id, message text, and tag name. In order to do that we need to use the join table, you'll see in a minute why we are ordering them by the message id.

The challenge here is what do we do with that data? This is where a tool like an ORM is useful, but we're going to do the formatting on our own! The problem here is not with SQL, but how to work with data in JavaScript. Here is what the SQL query above will return:

```sh
 id |  text  | name  
----+--------+-------
  1 | first  | funny
  1 | first  | happy
  2 | second | funny
  2 | second | silly
  3 | third  | silly
(5 rows)
```

What we want to do instead, is return one array with three message objects. However we have multiple tags for each message so we need to format this data differently. Let's see what that will look like. In our `routes/messages.js` let's add the following

```js
const express = require("express");
const router = express.Router();
const db = require("../db");

router.get("/", async (req, res, next) => {
  // get all the messages and tags
  const message_and_tags = await db.query(
    `SELECT m.id, m.text, t.name 
     FROM messages m 
       JOIN messages_tags mt ON m.id=mt.message_id 
       JOIN tags t ON mt.tag_id = t.id 
     ORDER BY m.id`
  );

  // we're going to start a counter at 0
  let startIdx = 0;
  // here is the array we will return containing three message objects
  let messages = [];

  // let's loop over the data we got back from the query (an array of 5 objects similar to the table above)
  for (let i = 0; i < message_and_tags.rows.length; i++) {
    let currentMessage = message_and_tags.rows[i];
    // if our counter is NOT the same as the message id
    if (startIdx !== currentMessage.id) {
      // set the counter to be that message id
      startIdx = currentMessage.id;
      // create a property called tags which is an empty array
      currentMessage.tags = [];
      // add the name of the tag to the tags array
      currentMessage.tags.push(currentMessage.name);
      // get rid of the key of .name on our current message
      delete currentMessage.name;
      // add that current message to our array
      messages.push(currentMessage);
    } else {
      // if the counter is the same as the message id (same message, different tag)
      // add the name of that tag to the message at the correct index in our messages array
      messages[startIdx - 1].tags.push(currentMessage.name);
    }
  }
  return res.send(messages);
});

router.post("/", async (req, res, next) => {
  try {
    const result = await db.query(
      "INSERT INTO messages (text) VALUES ($1) RETURNING *",
      [req.body.text]
    );
    res.json(result.rows[0]);
  } catch (err) {
    return next(err);
  }
});

module.exports = router;
```

And now in our `app.js`, let's add those routes:

```js
const express = require("express");
const app = express();
const bodyParser = require("body-parser");
const morgan = require("morgan");
const tagRoutes = require("./routes/tagRoutes");
const messagesRoutes = require("./routes/messagesRoutes");

app.use(morgan("tiny"));
app.use(bodyParser.json());
app.use("/tags", tagRoutes);
app.use("/messages", messagesRoutes);

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

The challenge here is correctly formatting the data the way we want it. However, once you get the hang of working with this data it becomes much easier. This is also a wonderful time to use libraries with functions that help format your data like [**lodash**](https://lodash.com/docs/).

# Your Turn!

Modify the `GET /tags` so that you show all the corresponding messages when you see all the tag. Add routes for updating and removing tags and messages