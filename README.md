# Automation Engineer take-home project

The goal of this take-home project is to evaluate your ability to work with various third-party APIs and tools.

Consider this situation : You are working in a bakery where every employee were previously software engineers.

As expected, employees started automating everything, from ordering ingredients to delivering cakes to clients but as the business boomed, the current process did not scale well.

Your role is to rebuild all the automation processes of the bakery.

> ⚠️ This document is meant to guide you but you are free to add / modify steps if you feel like you can do better or differently
> ⚠️ Some steps are marked *(optional)*, you are free to skip them if you feel like spending more time on other steps

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

> 🆘 You do not have to use a database for this API, you can of course use something like PostgreSQL but if you do not feel like it, you can also store everything in-memory

## Step 2 : Jira integration *(optional)*

For ordering ingredients, your tool of choice is [Jira](https://www.atlassian.com/software/jira).

The goal is to add a few routes to your API :

| Method | Route | Description |
| ------ | ----- | ----------- |
| `GET` *(optional)* | `/orders` | Get all existing cake orders |
| `POST` | `/orders` | Place a new cake order |
| `GET` *(optional)* | `/orders/{id}` | Retrieve a cake order by id |
| `DELETE` *(optional)* | `/orders/{id}` | Delete a cake order by id |

The schema of an order could look like this :

`GET /orders/21`
```json
{
  "recipe": "chocolate_cake",
  "persons": 4,
  "quantity": 2,
  "extra_ingredients": {
    "peanuts": 10
  }
}
```

The resulting required ingredients for this specific order would be :


| Ingredient | Quantity |
| ------ | ----- |
| Chocolate | 268g |
| Butter | 132g |
| Flour | 68g |
| Sugar | 132g |
| Eggs | 8 |
| Peanuts | 10 |

The logic behind the quantities if the following :
- Take the base quantity for an ingredient (we will take `sugar` here) : 16.5g
- Multiply by the amount of persons that will be eating the cake : 16.5 * `persons` = 16.5 * 4 = 66g
- Multiply by the amount of cakes for the order : 66 * `quantity` = 66 * 2 = 132g
- Add all extra ingredients (without multiplying them, they are shared for all the cakes)

Once you have computed all the ingredients quantities, it is time to create a Jira ticket to order these ingredients to your providers.

All your providers uses Jira to keep track of your orders so it makes sense to automate this process by creating a Jira ticket automatically whenever an order is transmitted to the API.

The Jira ticket could have the following structure :

```
Title : Order #{id}

Hello dear provider,

We would like to order these ingredients :

- {ingredient_name}: {quantity}{unit}
- {ingredient_name}: {quantity}{unit}

Have a nice day !
```

This is only a simple template, feel free to make it fancier :
- Better formatting of ingredients (a table ?)
- Add extra metadata on Jira ticket (due date, order link...)
