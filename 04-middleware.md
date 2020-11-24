# Hello Middleware

# Objectives:
By the end of this chapter, you should be able to:

- Define what middleware is
- Write custom middleware to handle errors
- Install and use helpful middleware including morgan
- Create express applications faster using the express-generator

# Middleware

Middleware functions are functions that have access to the request object (req), the response object (res), and the next function in the application’s request-response cycle.

Middleware functions can perform the following tasks:

- Make changes to the request and the response objects (customize a flash message or information to be sent to the user)
- End the request-response cycle (if you are not logged in redirect somewhere else)
- Call the `next` middleware in the stack.

The `next` function is a function in the Express router which, when invoked, executes the middleware succeeding the current middleware.

Middleware is code that gets run in between the request and the response. We've seen some examples of middleware already like `body-parser`, but we can write our own middleware as well! We will also be using our own custom middleware to configure the express router and handle errors.

To include middleware we use the `app.use` function. Here are two examples:

```js
// on every single request, run the following
app.use((req, res, next) => {
  console.log("Middleware just ran!");
  return next();
});

// on any request to /users, run the following
app.use("/users", (req, res, next) => {
  console.log("Middleware just ran on a users route!");
  return next();
});
```

# next

If you see above in the two example routes, we just added a third parameter called next. Let's focus on this a bit more!

Next is used to pass control to the next middleware function. If not the request will be left hanging or open.

What's important to note is that it passes control to the next **matching** route.

# Examples using middleware

Let's imagine we have a very simple application:

```js
const express = require("express");
const app = express();

app.use(function(req, res, next) {
  console.log("our middleware ran!");
});

app.get("/", function(req, res, next) {
  return res.json("We made it to the root route!");
});

app.listen(3000, function() {
  console.log("server starting!");
});
```

If we start the server and try to go to `localhost:3000`, we will actually **never** see a response from the server, it will just hang! Can you spot the error here?

The problem here is that in our middleware we did not invoke the next function, which passes control to the **next matching route** (in our case /) - let's fix this issue!

```js
const express = require("express");
const app = express();

app.use(function(req, res, next) {
  console.log("our middleware ran!");
  return next();
});

app.get("/", function(req, res, next) {
  return res.json("We made it to the root route!");
});

app.listen(3000, function() {
  console.log("server starting!");
});
```

Much better! The important thing to note here is that the order is **very** important. If we placed our `app.get` before our `app.use` we would never see that middleware run, that is why you almost always include your middleware **before** your route listeners. The exception to this rule is when you are using middleware to do error handling - let's see what that looks like!

# Error Handling

Another tool that we can add to our applications is an error handler. This middleware allows errors to be built up and finally sent to an errors page. This pattern is also quite useful when you have lots of different levels of nesting and there's potential for errors to be uncaught.

```js
// catch 404 and forward to error handler
app.use((req, res, next) => {
  const err = new Error("Not Found");
  err.status = 404;
  return next(err); // pass the error to the next piece of middleware
});

/* 
  error handler - for a handler with four parameters, 
  the first is assumed to be an error passed by another
  handler's "next"
 */
app.use((err, req, res, next) => {
  res.status(err.status || 500);
  return res.json({
    message: err.message,
    /*
     if we're in development mode, include stack trace (full error object)
     otherwise, it's an empty object so the user doesn't see all of that
    */
    error: app.get("env") === "development" ? err : {}
  });
});
```

You can read more about error-handling [here](https://expressjs.com/en/guide/error-handling.html).

It is important to note that the error-handling middlware has to be defined last, because `next(err)` passes down the line of middleware until it reaches one that has the `err` parameter and a response is issued.

# Morgan

Before we continue on, let's examine a useful module that will help log out requests and responses to the terminal. This module is called morgan, and can be very helpful when you're trying to debug your application.

```sh
mkdir another_express_app && cd another_express_app
touch app.js
npm init -y
npm install --save express body-parser morgan
```

Here is what our `app.js` might look like with all our middleware set up!

```js
const bodyParser = require("body-parser");
const express = require("express");
const morgan = require("morgan");

const app = express();

app.use(morgan("tiny"));
app.use(bodyParser.json());

app.get("/", (req, res, next) => {
  return res.json("Hello!");
});

// catch 404 and forward to error handler
app.use((req, res, next) => {
  const err = new Error("Not Found");
  err.status = 404;
  return next(err);
});

// error handlers
app.use((err, req, res, next) => {
  res.status(err.status || 500);
  return res.json({
    message: err.message,
    /*
     if we're in development mode, include stack trace (full error object)
     otherwise, it's an empty object so the user doesn't see all of that
    */
    error: app.get("env") === "development" ? err : {}
  });
});

app.listen(3000, () => {
  console.log("Server is listening on port 3000");
});
```

When you're ready, move on to [Express.js Exercises](./05-exercises.md)

