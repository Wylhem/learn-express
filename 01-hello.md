# Hello Express

# Objectives:

By the end of this chapter, you should be able to:

- Understand what express.js is and how to create a simple application
- Listen for requests and send responses using `express`
- Collect data from a URL using `request.params`

# Express.js Introduction

So far we have seen how to run Node files from the command line using `node filename.js`; create our own modules; and install external modules using `npm install`. Let's now move onto using the e**xpress.js framework** to build server-side applications with ease.

We previously saw the core HTTP module which allows us to make requests and issue responses, but we also mentioned that using the HTTP module requires a fair amount code and complexity. Thankfully, express abstracts away much of this complexity. So let's get started with it! In the terminal, let's type the following:

```sh
# create a folder and cd into it
mkdir first_express_app && cd first_express_app
# create a file called app.js (doing this before npm init is important, we will see why later!)
touch app.js
# create a package json file
npm init -y
# install the express module and save the module name and version to our package.json so that when we work with other developers they can easily install all of our dependencies
npm install express
```

Inside of our `app.js` let's add the following:

```js
// require the express module
const express = require("express");
// create an object from the express function which we contains methods for making requests and starting the server
const app = express();

// create a route for a GET request to '/' - when that route is reached, run a function
app.get("/", function(request, response) {
  /* inside of this callback we have two large objects, request and response
        request - can contain data about the request (query string, url parameters, form data)
        response - contains useful methods for determining how to respond (with html, text, json, etc.)
    let's respond by sending the text Hello World!
    */
  return response.send("Hello World!");
});

// let's tell our server to listen on port 3000 and when the server starts, run a callback function that console.log's a message
app.listen(3000, function() {
  console.log(
    "The server has started on port 3000. Head to localhost:3000 in the browser and see what's there!"
  );
});
```

Now let's start the server using `node app.js` and head over to `localhost:3000.` To stop the server press `control + c` (which is a standard process interrupt command). Remember that while the server is running, you will not be able to type commands in that tab in the terminal. If you want to run other command line commands, open a different terminal tab (command + t).

# Nodemon
`Nodemon` is very useful for restarting the server automatically when you edit your files or if the server crashes. We will need to install Nodemon globally so we can use it from the command line.

To install **nodemon** type the following command anywhere in the terminal `npm install -g nodemon` if you are getting errors when installing this, you can use sudo `npm install -g nodemon` and type in your password and press enter to install it; however, if you need to use sudo you've got a permissions issue with npm; follow these instructions to fix the problem.

To start the server now you can use `nodemon app.js` instead of `node app.js`.

# URL Parameters

So far we have just seen how to build static routes, but a key part of building servers is dynamic routing using route parameters. This is how applications can have one single route that provides many different kinds of responses depending on the data that is passed in the URL. Dynamic values to be passed in the URL are called "URL parameters" and are stored in the `params` object which exists in `request` object given to us in our callback functions.

To specify that a part of a URL will be a "parameter", we add a colon : character and then give our URL parameter a name. This name will be translated into a key in the `request.params` object. Let's see what that looks like below. It is also important to note that all of our URL parameters are strings! Let's take a look at an `app.js` file that has a route with URL parameters (notice the :)

```js
// same pattern as above, require express, invoke the express function
const express = require("express");
const app = express();

// same as above, listen for a GET request
app.get("/", function(request, response) {
  return response.send("Hello World!");
});

// when a request comes in to /instructors/ANYTHING
app.get("/instructor/:firstName", function(request, response) {
  // let's capture the "dynamic" part of the URL, which we have called "firstName". The name that we give to this dynamic part of the URL will become a key in the params object, which exists on the request object.

  // let's send back some text with whatever data came in the URL!
  return response.send(
    `The name of this instructor is ${request.params.firstName}`
  );
});

app.listen(3000, function() {
  console.log(
    "The server has started on port 3000. Head to localhost:3000 in the browser and see what's there!"
  );
});
```

Now if we head over to `localhost:3000/instructor/elie` and then `localhost:3000/instructor/majdi` and then `localhost:3000/dylan/michael`, what do you notice? Even though we created just a single route, we can now issue different responses depending on what the user has typed in the browser! This is the foundation for how to build dynamic applications (the URL parameter could be a value we look up in a database to display different profiles for the same route).

It is also important to note that **ALL URL parameters are strings**, so if we try to work with anything inside of `request.params`, it will always be a `string`.

# Live coding
You can see an example of the code here.

When you're ready, move on to [**Serving JSON with Express.js**](./02-serving-json.md)