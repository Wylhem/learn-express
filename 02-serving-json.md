# Serving JSON

# Objectives:
By the end of this chapter, you should be able to:

- Understand how to respond with JSON
- Include body-parser to collect form values and request information

# Sending back JSON

Fortunately in Express, it's quite easy to respond with JSON - all we need to do is use the `.json()` method on the response object! Previously we called these parameters `request` and `response`, but more commonly you will see them shortened to `req` and `res`.

```js
app.get("/", (req, res) => {
  res.json({ message: "That's it!" });
});
app.get("/instructor", (req, res) => {
  res.json({ name: "Elie" });
});
```

# .send() vs .json()

When sending back JSON from an Express application you may see two different ways, using the `json()` method as well as the `send()` method.

There is no actual difference between `.send` and .`json`, both methods are almost identical. `.json` calls `.send` at the end.

However, the main difference between `.json` and .`send` comes into picture when you have to pass non objects as a response. `.json` will convert non objects (ex. null, undefined etc) to JSON whereas res.send will not convert them. `.json` will also format the response using settings defined in your application.

In short, they are very similar, but it is preferable to use .json when sending back JSON.

# bodyParser

As we saw earlier, it's quite easy to send back JSON using `.json`, but it's a little trickier to accept user input when sent as JSON or other values. To do that, we need to include another module called `body-parser`, which parses the "body" of the response. This kind of module is called `middleware` as it sits in the middle of the request/response cycle and can modify either the request or response. We'll see much more about that soon.

When using external middleware, there is a 3 step process:

- Install it - `npm install body-parser`
- Require it - `const bodyParser = require("body-parser")`
- Use it - `app.use(bodyParser.json())`

If you see above, we included a module called `body-parser`, but set a variable called `bodyParser` - the reason is so that JavaScript does not try to subtract `body` from `parser` (since JS treats the - as subtraction). When data is sent to the server, we can now collect that data inside of `req.body`.

Here we are just using `bodyParser.json()`, but depending on how request data is being sent (usually determined by the `Content-Type` header), it may be important to include other ways of parsing the body. Here are two examples:

```js
app.use(bodyParser.json()); // for parsing application/json
app.use(bodyParser.urlencoded({ extended: true })); // for parsing application/x-www-form-urlencoded
```

If your API is listening for any kind of request where data may be sent to the server via a POST, make sure you have `bodyParser` included and are using it to collect data inside of `req.body`.

# Adding a status code

When our Express server issues an HTTP response, there will always be an accompanying HTTP status code. Express will set sensible defaults for you, but if you would like to select the specific status code, you can do that using the `.status` method and chain `.json` to it. Here are some small examples:

```js
app.get("/", (req, res) => {
  res.status(200).json({ name: "Elie" });
});

app.get("/secret", (req, res) => {
  res.status(401).json({ message: "Unauthorized" });
});
```

When you're ready, move on to [*Express Router**](./03-router.md)