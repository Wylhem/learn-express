# Exercises 

For this exercise we will be building a simple application where we will store a shopping list. You should use an **array** to store your items in the shopping list.

Our application should have the following routes:

- `GET /items` - this should respond with a list of shopping items.
- `POST /items` - this route should accept form data and add it to the shopping - list.
- `GET /items/:id` - this route should display a single item's name and price
- `PATCH /items/:id` - this route should accept edits to existing items.
- `DELETE /items/:id` - this route should allow you to delete a specific item from the array.