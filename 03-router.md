# Express Router

# Objectives:

By the end of this chapter, you should be able to:

- Move routing logic into a separate file using the express router
- Use the express router to refactor routing code

# Router

So far we have seen how to create routes in express and render templates and even respond with redirects! As we start building more and more routes, our `app.js` can get very messy quickly.

In order to clean this up, we will move our routes to a file in a folder called `routes`. The files in this folder will contain all of our routes and we will export them using the express router. Let's get started with a simple application.

# RESTful Review

Let's build out routes based on some standard RESTful routes:

|HTTP| Verb|	Path|	Description|	CRUD
POST|	/users	|Create a user using a payload|	Create
GET|	/users	|Display a list of users	| Read
GET|	/users/:id|	Display a single user	| Read
PATCH or PUT *|	/users/:id	Edit a user using a payload	| Update
DELETE|	/users/:id	|Delete a user	| Delete

Notice that the routes to render a page with a form do not exist here because we are building an API!

# Building a Router
Now that we have a good structure for our RESTful routes, let's use the express router to re-create our users application. Let's get started with a new express app. In the terminal, let's run the following:

```sh
mkdir express-router && cd express-router
touch app.js
npm init -y
npm install express body-parser
mkdir routes
touch routes/index.js
```

Now let's add the following to our `routes/index.js`

```js
const express = require("express");
/*
  The Router is an object on the base express module.
  This router object will have similar methods (.get, .post, .patch, .delete) to the 
  app object we have previously been using.
*/
const router = express.Router();

const users = []; // this would ideally be a database, but we'll start with something simple
var id = 1; // this will help us identify unique users

// instead of app.get...
router.get("/users", (req, res) => {
  return res.json(users);
});

router.get("/users/:id", (req, res) => {
  const user = users.find(val => val.id === Number(req.params.id));
  return res.json(user);
});

// instead of app.post...
router.post("/users", (req, res) => {
  users.push({
    name: req.body.name,
    id: ++id
  });
  return res.json({ message: "Created" });
});

// instead of app.patch...
router.patch("/users/:id", (req, res) => {
  const user = users.find(val => val.id === Number(req.params.id));
  user.name = req.body.name;
  return res.json({ message: "Updated" });
});

// instead of app.delete...
router.delete("/users/:id", (req, res) => {
  const userIndex = users.findIndex(val => val.id === Number(req.params.id));
  users.splice(userIndex, 1);
  return res.json({ message: "Deleted" });
});

// Now that we have built up all these routes - let's export this module for use in our `app.js`!
module.exports = router;
```

Now, how do we actually use these routes? Let's add the following in our app.js

```js
const express = require("express");
const bodyParser = require("body-parser");

const app = express();
// require our routes/index.js file
const userRoutes = require("./routes");

app.use(bodyParser.json());

// Now let's tell our app about those routes we made!
app.use(userRoutes);

app.get("/", (req, res) => {
  return res.json("Start with /users");
});

app.listen(3000, () => {
  console.log("Server is listening on port 3000");
});
```

# A More Declarative Router Syntax

We could also write these routes using an alternative, more declarative syntax for express routing. When we say 'more declarative' in this case, we're basically saying the code is using methods (i.e. .`route('/users')`) to be more readable and implicit rather than redundant and explicit. The code becomes more 'self-documenting' so to speak, because we're more concerned with what it tells us (route definitions in this case) rather than how it does it.

```js
const express = require("express");
const router = express.Router();

const users = [];
var id = 1;

// declare the route first, then all the methods on it
router
  .route("/users")
  .get(() => {
    return res.json(users);
  })
  .post(() => {
    users.push({
      name: req.body.name,
      id: ++id
    });
    return res.json({ message: "Created" });
  });

router
  .route("/users/:id")
  .get((req, res) => {
    const user = users.find(val => val.id === Number(req.params.id));
    return res.json(user);
  })
  .patch((req, res) => {
    user.name = req.body.name;
    return res.json({ message: "Updated" });
  })
  .delete((req, res) => {
    users.splice(user.id, 1);
    return res.json({ message: "Deleted" });
  });

module.exports = router;
```

# Declarative Router Syntax with a Prefix

Even better - we can prefix all of our routes to start with /users if we would like.

All we have to do is make a slight change to app.js:

```js
// prefix every single route in here with /users
app.use("/users", userRoutes);
```

Here is what that looks like in our `routes/index.js`

```js
const express = require("express");
const router = express.Router();

const users = [];
var id = 1;

// declare all the methods on the /users route (prefix specified in app.js)
router
  .route("")
  .get((req, res) => {
    return res.json(users);
  })
  .post((req, res) => {
    users.push({
      name: req.body.name,
      id: ++id
    });
    return res.json({message: "Created"});
  });

router
  .route("/:id")
  .get((req, res) => {
    const user = users.find(val => val.id === Number(req.params.id));
    return res.json(user);
  })
  .patch((req, res) => {
    const user = users.find(val => val.id === Number(req.params.id));
    user.name = req.body.name;
    return res.json({ message: "Updated" });
  })
  .delete((req, res) => {
    const userIndex = users.findIndex(val => val.id === Number(req.params.id));
    users.splice(userIndex, 1);
    return res.json({ message: "Deleted" });
  });

});

module.exports = router;
```

When you're ready, move on to [**Express Middleware**](./04-middleware.md)