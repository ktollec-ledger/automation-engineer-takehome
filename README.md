# Automation Engineer take-home project

The goal of this take-home project is to evaluate your ability to work with various third-party APIs and tools.

Consider this situation : You are working in a bakery where every employee were previously software engineers.

As expected, employees started automating everything, from ordering ingredients to delivering cakes to clients but as the business boomed, the current process did not scale well.

Your role is to rebuild all the automation processes of the bakery.

> ⚠️ This document is meant to guide you but you are free to add / modify steps if you feel like you can do better or differently

## Step 1 : The API

The old automation system was messy and all over the place. The bakery learned from their mistake and will not do it twice.

This time, the starting point of all automated processes will be an API.

Your goal is to build an API with a Python framework (such as FastAPI or Flask).

This API will feature the following routes :

| Method | Route | Description |
| ------ | ----- | ----------- |
| `GET`  | `/cakes` | Get all existing cakes and their recipes |
| `POST` | `/cakes` | Create a new cake recipe |
| `GET` | `/cakes/{id}` | Get a specific cake recipe by id |
| `DELETE` | `/cakes/{id}` | Delete a cake recipe by id |

The cake id will be the snake_case version of the cake name (such as `chocolate_cake` or `carrot_cake`).

The schema will look like this :

`GET /cakes/chocolate_cake`
```json
{
  "ingredients": {
    "chocolate": 33.5,
    "butter": 16.5,
    "flour": 8.5,
    "sugar": 16.5,
    "eggs": 1
  }
}
```

The ingredient amount is either expressed using a weight in grams (float form) or a unit quantity (integer form).
All recipes are made for a single eater, you need to multiply quantities to adapt the recipe to multiple eaters.
