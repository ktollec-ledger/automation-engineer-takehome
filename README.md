# Automation Engineer take-home project

The goal of this take-home project is to evaluate your ability to work with various third-party APIs and tools.

Consider this situation : You are working in a bakery where every employee were previously software engineers.

As expected, employees started automating everything, from ordering ingredients to delivering cakes to clients but as the business boomed, the current process did not scale well.

Your role is to rebuild all the automation processes of the bakery.

> ⚠️ This document is meant to guide you but you are free to add / modify steps if you feel like you can do better or differently

> ⚠️ Some steps are marked *(optional)*, you are free to skip them if you feel like spending more time on other steps, steps are usually order by importance, meaning Step 2 is more valuable for us than step 7

To get started :
- Fork this project
- Make it private (this is important)
- Add `ktollec-ledger` to your fork with view permissions on the repository settings
- Good luck !

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

You can register [here](https://www.atlassian.com/software/jira/free) for a free account and then head [here](https://developer.atlassian.com/server/jira/platform/rest-apis/) to learn more about Jira APIs.

The goal is to add a few routes to your API :

| Method | Route | Description |
| ------ | ----- | ----------- |
| `GET` *(optional)* | `/orders` | Get all existing cake orders |
| `POST` | `/orders` | Place a new cake order |
| `GET` *(optional)* | `/orders/{id}` | Retrieve a cake order by id |
| `DELETE` *(optional)* | `/orders/{id}` | Cancel a cake order by id |

The schema of an order could look like this :

`GET /orders/21`
```json
{
  "customer": "Mr. Cakelover",
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

## Step 2.1 : Jira integration, order batching *(optional)*

Based on the previous step, instead of creating a Jira ticket one by one for each order, we could batch them to create a single ticket per day (created at 6am UTC for example).

This single Jira ticket would contain the sum of all ingredients of all orders passed since the last ticket created.

You will probably need an additional CronJob component for this task. If you need some ideas, it could be :
- A [Celery periodic-task](https://docs.celeryq.dev/en/stable/userguide/periodic-tasks.html)
- A [Kubernetes CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
- Something else ?

## Step 2.2 : Jira integration, cancelling order *(optional)*

Sometimes, the provider lacks some ingredients.

When it is the case, the provider will add a tag `missing-ingredients` to the ticket and close it.

This will automatically call the `DELETE /orders/{id}` route.

## Step 3 : Slack integration *(optional)*

Since all the bakery employees are ex-software engineers, most of the clients are also working in software development.

One of the client suggested that it could be nice to receive a Slack notification whenever an order is ready.

For that, we will need a new API route :

| Method | Route | Description |
| ------ | ----- | ----------- |
| `POST` | `/orders/{id}/ready` | Flag a cake order as "ready" |

Whenever we make a call to this API route, it will try to find the customer (using the `customer` field from the `order` schema) on the bakery Slack server and send them a private message using a service account.

You can create a free [Slack](https://slack.com/) server and head [here](https://api.slack.com/) to know more about the Slack API.

## Step 3.1 : Mail notification *(optional)*

Since some customers do not have a Slack account and want to stay old-school, we might need to send an email.

If the automation tool can't find the customer on Slack, fallback to sending an email to `{customer}@gmail.com`.

For example, if the customer is `Mr Cakelover`, the resulting email address could be `mr_cakelover@gmail.com`.

You can create a dummy GMail address to be able to interact with it using SMTP.

## Step 4 : Google Drive integration *(optional)*

We would like to keep track all cancelled orders into a Google Drive folder.

Whenever an order is cancelled using the `DELETE /orders/{id}` route, create / edit a document on a Google Drive directory per customer.

For example, the Google Drive directory should look like this :
```
Orders/
  mr_cakelover.doc
  ms_delicious.doc
```

Each document will contain the following informations :

```
Title: Mr Cakelover cancelled orders

- Order 21 [Chocolate Cake] : Date 01/01/2022 06:10
- Order 101 [Carrot Cake] : Date 20/04/2022 14:30
```

Once again, feel free to edit the template !

The document can either be a [Google Doc](https://developers.google.com/docs/api) or a [Google Sheet](https://developers.google.com/sheets/api)

## Step 5 : GitHub integration *(optional)*

Everyone at the bakery is super satisfied of your new automation tool. New hires would like to help you working on your project.

Since the bakery is quite big, it is tedious to grant access to every new hire manually. It would be quite useful to have a new-hire onboarding route on your API :

| Method | Route | Description |
| ------ | ----- | ----------- |
| `GET` *(optional)* | `/employees` | Get all bakery employees |
| `POST` | `/employees` | Create a new bakery employee |
| `GET` *(optional)* | `/employees/{id}` | Get an employee by id |
| `DELETE` *(optional)* | `/employees/{id}` | Fire a bakery employee |

The employee schema could look like this :

`GET /employees/10`
```json
{
  "firstName": "Jack",
  "lastName": "Harrot",
  "permissions": {
    "developer": true
  }
}
```

When an employee `permissions.developer` is `true`, we should grant them access to this repository (your fork).

You can check the GitHub API documentation right [here](https://docs.github.com/en/rest)

## Step 6 : Frontend *(optional)*

An API is quite useful but for non-technical people, it can be quite hard to use.

If you want to, you can create a small front app to interact with your API.

Ideally, it would cover some of the routes defined above (you don't have to implement all the routes).

Feel free to use your favorite framework !

## Step 7 : Deployment *(optional)*

Now that your application is quite complete, it is time to deploy it !

You are free to deploy it however you want, show us your skills !

Here are a few ideas :
- Application running on [Heroku](https://www.heroku.com/)
- Free Kubernetes cluster on [IBM Cloud](https://www.ibm.com/cloud/free)
- Something else ?

Ideally, you would be able to provide us a link to your application in production so we can test it.

Please fill out this form with all URLs used for this project :

| Component | URL |
| ------ | ----- |
| Python API | `YOUR_URL_HERE` |
| Frontend app | `YOUR_URL_HERE` |
| Jira Workspace | `YOUR_URL_HERE` |
| Slack server | `YOUR_URL_HERE` |

This is the perfect step to get extra fancy, here are a few ideas you can do (optional of course) :
- Use a real domain (you can book free ones) and a certificate (Let's Encrypt can provide free certificates as well)
- Add a CI/CD pipeline to your repository
  - Lint and perform tests on your API
  - Deploy / build image of your application and deploy it automatically
  - Send a Slack notification if a build went wrong
- Store all credentials using Hashicorp Vault, AWS or Kubernetes secrets
- More ?

## Step 8 : Feedback

On your fork repository, edit the `FEEDBACK.md` document :)
