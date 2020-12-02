# Hello CRUD

# Objectives:
By the end of this chapter, you should be able to:

- Build an API with full CRUD on a single resource
- Add a front-end to interact with the API

# Building the API

For this application we're going to build an API that performs full CRUD (create, read, update, delete) on a single resource - `fishes`. Each row in our `fishes` table will have a name and type. Our API will consist of the following four routes:

`GET /fishes `- returns an array of fish objects
`POST /fishes` - creates a new fish object and returns the object
`PATCH /fishes/:id `- updates an existing fish object
`DELETE /fishes/:id` - removes an existing fish object

To get started, we'll create an initial folder structure:

```sh
mkdir fishes-app
cd fishes-app
touch app.js
npm init -y
npm install express body-parser morgan pg
mkdir db
touch db/index.js
mkdir routes
touch routes/fishes.js
createdb fishes-app
psql fishes-app
```

In psql let's run the following:

```sql
CREATE TABLE fishes (id SERIAL PRIMARY KEY, name TEXT, type TEXT);
\q
```

Our folder structure should now look like contain the following:

```sh
.
├── app.js
├── db
│   └── index.js
├── package-lock.json
├── package.json
└── routes
    └── fishes.js
```

Let's get started in our `db/index.js` by creating a connection and exporting it:

```js
const { Client } = require("pg");
const client = new Client({
  connectionString: "postgresql://localhost/fishes-app"
});

client.connect();

module.exports = client;
```

Now in our `routes/fishes.js` let's add the following routes to read all the fishes, create a fish, update a fish, and delete a fish. If you would like to try writing this on your own before looking at the solution, feel free!

```js
const express = require("express");
const router = express.Router();
const db = require("../db");

router.get("/", async function(req, res, next) {
  try {
    const results = await db.query("SELECT * FROM fishes");
    return res.json(results.rows);
  } catch (err) {
    return next(err);
  }
});

router.post("/", async function(req, res, next) {
  try {
    const result = await db.query(
      "INSERT INTO fishes (name,type) VALUES ($1,$2) RETURNING *",
      [req.body.name, req.body.type]
    );
    return res.json(result.rows[0]);
  } catch (err) {
    return next(err);
  }
});

router.patch("/:id", async function(req, res, next) {
  try {
    const result = await db.query(
      "UPDATE fishes SET name=$1, type=$2 WHERE id=$3 RETURNING *",
      [req.body.name, req.body.type, req.params.id]
    );
    return res.json(result.rows[0]);
  } catch (err) {
    return next(err);
  }
});

router.delete("/:id", async function(req, res, next) {
  try {
    const result = await db.query("DELETE FROM fishes WHERE id=$1", [
      req.params.id
    ]);
    return res.json({ message: "Deleted" });
  } catch (err) {
    return next(err);
  }
});

module.exports = router;
```

Now in our `app.js` let's add the following middleware and start up our server!

```
const express = require("express");
const app = express();
const bodyParser = require("body-parser");
const fishesRoutes = require("./routes/fishes");
const morgan = require("morgan");

app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

app.use(morgan("tiny"));
app.use("/fishes", fishesRoutes);

app.use(function(req, res, next) {
  let err = new Error("Not Found");
  err.status = 404;
  next(err);
});

if (app.get("env") === "development") {
  app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.send({
      message: err.message,
      error: err
    });
  });
}

app.listen(3000, function() {
  console.log("Server starting on port 3000!");
});
```

# Adding a front-end

Now that we have a working API, we're going to start with another application that is going to contain the html, js and css for this app. We will be running this application on another port and will be communicating with our API via AJAX. Let's get started!

```sh
mkdir fishes-app-frontend
cd fishes-app-frontend
touch index.html
touch script.js
```

Inside of our HTML, we're going to place the following:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Fishes App!</title>
</head>
<body>
    <ul id="container"></ul>
    <script src="https://code.jquery.com/jquery-3.3.1.js"></script>
    <script src="./script.js"></script>
</body>
</html>
```

# Showing all of the fishes

When the page loads, we're going to make an AJAX call to get all of the fishes and display them on the page! In our `script.js`, let's add the following

```js
$(document).ready(function() {
  $.getJSON("http://localhost:3000/fishes").then(function(fishes) {
    fishes.forEach(function(fish) {
      console.log("let's see each fish!", fish);
    });
  });
});
```

Let's now try to see what the console outputs by running this application on a server. To do that we're going to use `http-server`, which you can install using `npm install -g http-server`. Once that's installed, let's run `http-server -p 3001` to start our server on port 3001 (since our backend is on port 3000). Make sure your backend is running before you continue!

# CORS

Once you load the page, you should see the following in the console:

```sh
Failed to load http://localhost:3000/fishes:
No 'Access-Control-Allow-Origin' header is present on the requested resource.
Origin 'http://localhost:3001' is therefore not allowed access.
```

What's going on here? Remember that there is a security policy called the Same Origin Policy which restricts javascript being loaded on a different origin (which includes a different port number). To get around this we are going to use a technology called CORS or Cross Origin Resource Sharing. Once CORS is enabled, a header will be sent from the server, allowing access to `localhost:3000`. To configure this, we need to go back to our `fishes-app` folder and run `npm install cors`. In our `app.js` we need to do the following:

```js
const express = require("express");
const app = express();
const bodyParser = require("body-parser");
const fishesRoutes = require("./routes/fishes");
const morgan = require("morgan");
const cors = require("cors");

app.use(cors());
app.use(bodyParser.json());
app.use(morgan("tiny"));
app.use("/fishes", fishesRoutes);
```

Now that you have this working, you should be able to restart the backend server and run `nodemon` again. When you go back to the browser and refresh the page, you should be able to see the fishes in the console! Now let's actually append them to the page, let's change our script.js to do the following:

```js
$(document).ready(function() {
  const $container = $("#container");
  $.getJSON("http://localhost:3000/fishes").then(function(fishes) {
    fishes.forEach(function(fish) {
      let $newFish = $("<li>", {
        text: `${fish.name} : ${fish.type}`
      });
      $container.append($newFish);
    });
  });
});
```

Now if you refresh the page, you should see all of the fishes on the page!

# Adding a new fish

To add a new fish, let's add a form in our HTML and then some JS to make sure it gets sent to the server and appending correctly. In our HTML let's add the following:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Fishes App!</title>
</head>
<body>
    <ul id="container"></ul>
    <form action="#" id="new-fish-form">
        <div>
            <label for="name">Name:</label>
            <input type="text" name="name" id="name"/>
        </div>
        <div>
            <label for="type">Type:</label>
            <input type="text" name="type" id="type" />
        </div>
        <button>Add a fish!</button>
    </form>
    <script src="https://code.jquery.com/jquery-3.3.1.js"></script>
    <script src="./script.js"></script>
</body>
</html>
```

Now that we have this form working, let's add the necessary JavaScript to make sure we can send this data to the server when the form is submitted, let's make sure our `script.js` contains the following:

```js
$(document).ready(function() {
  const $container = $("#container");
  $.getJSON("http://localhost:3000/fishes").then(function(fishes) {
    fishes.forEach(function(fish) {
      let $newFish = $("<li>", {
        text: `${fish.name} ${fish.type}`
      });
      $container.append($newFish);
    });
  });

  $("#new-fish-form").on("submit", function(e) {
    e.preventDefault();
    const name = $("#name").val();
    const type = $("#type").val();
    $.post("http://localhost:3000/fishes", { name, type }).then(function(fish) {
      let $newFish = $("<li>", {
        text: `${fish.name} ${fish.type}`
      });
      $container.append($newFish);
      $("#new-fish-form").trigger("reset");
    });
  });
});
```

Now when a fish is submitted, we take the response from the server and append to the page and reset form values. We're now able to load and create the fishes!

# Removing a fish

To remove a fish, we're going to first change what each `li` looks like when we create it. We'll replace the `text` key with `html` so that we can add a button element to this `li`

The button will have a value of X. We're also going to give this `li` a `data` attribute to uniquely identify each fish and ensure that we delete the correct fish when we click the X button.

Next, we're going to add an event listener for all buttons clicked inside the container (we're using event delegation so that we only need one event listener and can listen for elements that are not yet appended to the DOM). Once a button is clicked, we will get the correct `id` using the `data` attribute we set above and then issue an HTTP DELETE request to the correct endpoint.

Once that request has finished, we will remove that `li` from the DOM. Here's what that looks like:

```js
$(document).ready(function() {
  const $container = $("#container");
  $.getJSON("http://localhost:3000/fishes").then(function(fishes) {
    fishes.forEach(function(fish) {
      let $newFish = $("<li>", {
        html: `
            ${fish.name} ${fish.type}
            <button class="delete">X</button>
        `,
        "data-id": `${fish.id}`
      });
      $container.append($newFish);
    });
  });

  $("#new-fish-form").on("submit", function(e) {
    e.preventDefault();
    const name = $("#name").val();
    const type = $("#type").val();
    $.post("http://localhost:3000/fishes", { name, type }).then(function(fish) {
      let $newFish = $("<li>", {
        html: `
            ${fish.name} ${fish.type}
            <button class="delete">X</button>
        `,
        "data-id": `${fish.id}`
      });
      $container.append($newFish);
      $("#new-fish-form").trigger("reset");
    });
  });

  $container.on("click", ".delete", function(e) {
    e.preventDefault();
    const id = $(e.target)
      .parent()
      .data("id");
    const type = $
      .ajax({
        method: "DELETE",
        url: `http://localhost:3000/fishes/${id}`
      })
      .then(function() {
        $(e.target)
          .parent()
          .remove();
      });
  });
});
```

As you can see above, there is some duplication here, so now would be a good time to go back and refactor. What we have now is an application that allows us to create, read and delete as a single page application backed by an Express and Postgres API!

When you're ready, move on to [One-To-Many Relationships](./08-one-to-many.md)