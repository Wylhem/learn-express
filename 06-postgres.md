# Hello Postgres

# Objectives:
By the end of this chapter, you should be able to:

- Introduce the `pg` module and how to make queries
- Describe the landscape of adapters, query builders and ORMs for postgres and node
- Execute simple SQL statements using the `pg` module

# Relational Databases with Node/Express

As with many other languages, there are multiple ways for your application server to interact with your database server. At the lowest level, our first option is the `pg` module. The `pg` module allows us to directly connect to a postgres database and execute SQL. This is what the remainder of the course will use as it forces you to really understand the SQL you are writing, and builds an excellent foundation to learn more abstract tools and libraries. You can read more about the pg module here. Before we move onto to getting set up with `pg` let's identify some other tools in the Node ecosystem.

# Knex

Knex labels itself as a "query builder". What this means is that we can use Knex to execute SQL without having to worry about writing SQL (we do it all in JavaScript). You can read more about Knex [here](https://knexjs.org/). Knex is a nice middle layer between having to write raw SQL and using a full-fledged ORM (which we'll learn about next). Knex also provides migrations, which allows you to keep track of changes to your schema (you can think of migrations like git for your database). Knex does not come with built in models, association helpers, or model validations. This is where ORMs can help us.

# Sequelize, Bookshelf, Objection

After Knex, the next set of technologies that we have are ORMs or Object Relational Mappers (you can read more about them [here](https://en.wikipedia.org/wiki/Object-relational_mapping)). ORMs provide migrations, validations, association helpers, and much more. The idea is to create objects that map to rows in your database table and classes (called `models`), which map to tables. ORMs abstract quite a bit of database logic from the developer, but do allow for building applications faster. In the Node ecosystem, there is not one go-to ORM (unlike Ruby and Python), here are some of the more popular ones:

- [Sequelize](http://docs.sequelizejs.com/)
- [Bookshelf](http://bookshelfjs.org/)
- [Objection](https://vincit.github.io/objection.js/)

# Setting up PG

Now that we've taken a look at some of the tools available for working with relational databases, let's see how we can get started setting up a connection using the pg module. Before you can start using this module you need to make sure it's installed using `npm install pg`, so let's use the following steps:

```sh
mkdir pg-lesson-one
cd pg-lesson-one
createdb pg-lesson-one
npm init -y
npm install pg
touch index.js
psql pg-lesson-one
```

Now in psql, let's do the following:

```sql
CREATE TABLE students (id SERIAL PRIMARY KEY, name TEXT);
INSERT INTO students (name) VALUES ('Elie');
INSERT INTO students (name) VALUES ('Michael');
INSERT INTO students (name) VALUES ('Joel');
INSERT INTO students (name) VALUES ('Matt');
\q
```

Once you have the table created, let's go back to our `index.js` and add the following:

```js
const { Client } = require("pg"); // include the Client constructor from the pg module

// make a new instance of the Client constructor and specify which db to connect to using the connectionString key
const client = new Client({
  connectionString: "postgresql://localhost/pg-lesson-one"
});

// connect!
client.connect();

// let's make a function to get all the rows in our students table!
async function getStudents() {
  const results = await client.query("SELECT * FROM students");
  console.log(results.rows);
}

// let's get our students and then stop the node process
// when we start using express, process.exit will be a response from the server instead
getStudents().then(() => process.exit(0));
```

If we run `node index.js` we should see the following output:

```js
[
  { id: 1, name: "Elie" },
  { id: 2, name: "Michael" },
  { id: 3, name: "Joel" },
  { id: 4, name: "Matt" }
];
```

You can see that the data comes back in a property called `rows` which is **always** an array!

# Handling asynchronous functions + the query function

Each time that we are querying the database, we are running an asynchronous function. Since the node environment is asynchronous, almost all functions will be asynchronous unless specified otherwise (this does not include built in methods to JavaScript that are always synchronous).

To manage our asynchronous code and wait until the database query has finished before our express server responds, we're going to use the `async` and `await` keywords. Remember, any function that is `async` will return a promise and have the ability to use the `await` keyword.

The `await` keyword will wait for whatever is passed to it to resolve (whatever you pass to the `await` keyword must always be a promise). Since the `pg` module supports promises, the result of the query() function is a promise, which is why we can use `await`. Always, always make sure that the function you are using the `query` method with is an `async` function, otherwise you can not use the `await` keyword.

Now that we have a better idea of what the code above is doing, let's write a function to add a record!

# Adding a record

Back in our `index.js`, let's write another function to add a student.

```js
// let's make a function to get all the rows in our students table!
async function addStudent(name) {
  const results = await client.query(
    "INSERT INTO students (name) VALUES ($1) RETURNING *", // We're using RETURNING * to get back the new record
    [name] // notice our use of $1 - NEVER EVER use string concatenation/interpolation in your SQL queries.
  );
  console.log(results.rows[0]); // we are using [0] because there is only 1 record here.
}

// when we start using express, process.exit will be a response from the server instead.
addStudent("Angelina").then(() => process.exit(0));
```

If we remove the previous `getStudents` function call and then run `node index.js`, we should see the following:

```js
{ id: 5, name: 'Angelina' }
```

Now we are able to read all students and create students!

# Client vs Pool

As you read through more documentation/tutorials on the `pg` module, you might come across the term `connection pool`. If you're working on a web application or other software which makes frequent queries you'll want to use a connection pool. The applications that we are building at this moment do not require pooling, so we're going to omit using it for simplicity. You can read more [here](https://node-postgres.com/features/pooling) why you might want to use a pool.

# Exercise

- Write the necessary function to find a student by its id. This function should return the found student.
- Write the necessary function to update a student. This function should return the updated student.
- Write the necessary function to delete a student. This function should return the updated student.
- Modify the students table so that null names can not be inserted.

When you're ready, move on to [CRUD with Express, Postgres, and Node](./07-crud.md)