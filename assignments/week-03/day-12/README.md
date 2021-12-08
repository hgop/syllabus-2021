# E2E Testing

## Prerequisites
Before starting this part of the assignment we need to make sure we have the following up and running:
- [ ] A working Connect4 Game client application
- [ ] A running Circle CI pipeline for your client that:
    - On every push:
        - [ ] Builds a docker image from your code
        - [ ] Publishes a docker image from your code
    - On every merge to the main branch:
        - [ ] Builds a docker image from your code
        - [ ] Publishes a docker image from your code
        - [ ] Deploys your application to your kubernetes cluster

## Get up to date
You don't need everything from Week 2 to finish this part of the assignment. Make sure that your client is working, and you're able to play a local game, if it's not you can find the client base here, which will also be used for this assignment:
- [The client code can be found here](https://github.com/hgop/hgop-connect4-client-base-2021)
Make sure to replace the Team name and docker repository field.

## Objectives
Today we want to run E2E tests on our client.

PLEASE NOTE: In a production system we would always want to run E2E when deploying both client and server. If the E2E fail we would like to prevent deployment. E.g. if we make changes to our server we want the E2E tests to run before deploying to production and if they fail it's likely that our changes break the integration between our server and our client. However to make this simpler we're only running the E2E test on our client in this assignment. Also, we're testing the client against our production server and database, we would not want to do this in a live system.
What we want to finish today:

- [ ] Add E2E tests to our client repository
- [ ] Deploy an instance of our client to run the E2E tests on
- [ ] Run our E2E test on our E2E client instance
- [ ] Prevent our client from being deployed if the E2E test fail

## Step 1 - Add E2E tests to your project
We are going to use Cypress to create our E2E tests.

> What is Cypress?

> It is a next-generation front end testing tool constructed for the modern web. This tool addresses the critical pain points developers, and QA engineers used to face when testing modern applications, E.g., synchronization issues, the inconsistency of tests due to elements not visible or available.

> It is built on Node.js and comes packaged as an npm module. As its basis is Node.js, it uses JavaScript for writing tests. But 90% of coding can be done using Cypress inbuilt commands, which are easy to understand.

Setting up Cypress is very simple, but since we're not creating a separate cypress project for our test you can just copy/paste the cypress folder from the connect-four-client-base project, along with the cypress.json file

We also need to add cypress as a dependency to our project: `yarn add -E -D cypress`

Since we need to deploy a seperate instance of our client to run our tests against please copy/update the following files/folders from [this repository](https://github.com/hgop/hgop-connect4-client-base-2021).
```
cypress <- this folder contains the e2e test definition
cypress.json <- This file needs to include the URL for your application
```
When you've finished adding all the code and adding a package.json script to run the cypress tests `"cypress": "cypress open"` you should be able to run the tests in a browser in your local environment, like so:

```bash
yarn cypress
```

You can also run the tests in a terminal from inside the project folder. This is ideal for the CI. Add another package.json script: `"cypress:headless": "cypress run"` then run from inside the project:

```bash
yarn cypress:headless
```

This will run the tests on the url you added to the `cypress.json` file. If you want to run your client locally and run the tests on your local instance you can change the `baseUrl` in `cypress.json` to `http://localhost:3000`.

## Step 2 - Run E2E tests in your pipeline

* Deploy the client to the acceptance environment in the pipeline.
* After deploying the client we can run our cypress tests on the acceptance environment like so:
  
```yaml
  e2e-test:
    executor: node-environment
    steps:
      - checkout
      - run: yarn cypress:headless
```

Make sure your test are calling the acceptance environment.

## Step 3 - Add more E2E tests

Our current test runs through one game. It would catch a lot of errors that could come up. E2E tests are usually not used to tests minor details and we won't do that either. Just create two more tests:

1. should display a win for Player 2 where he wins via a diagonal row
2. should display a draw

NOTE: a local game does not display a draw. You'll need to add this functionality and test it.

ALSO NOTE: The current test has 500ms wait time between turns, this is for the server to catch up and prevent false negatives (which are unfortunately a bit common in these types of tests, but can be mitigated). If your test fails make sure you're giving the server and client enough time.

A great introduction on how to use Cypress can be found on their [website](https://docs.cypress.io/guides/core-concepts/introduction-to-cypress.html).

## Handin

You should store all the source files in your repository:

~~~bash
.
├── assignments
│   ├── day01
│   │   └── answers.md
│   ├── day02
│   │   └── answers.md
│   ├── day09
│   │   └── answers.md
│   └── day11
│       └── answers.md
├── .circleci
│   └── config.yml
├── .github
│   └── dependabot.yml
├── docker-compose.yaml
├── .gitignore
├── Justfile
├── README.md
├── scripts
│   ├── ci
│   │   ├── database
│   │   │   └── create_database.sh
│   │   ├── deploy
│   │   │   └── create.sh
│   │   └── yaml
│   │       └── merge.sh
│   └── verify_local_dev_environment.sh
└── src
    ├── connect4-client
    │   ├── Dockerfile
    │   ├── .dockerignore
    │   ├── .eslintrc.json
    │   ├── .gitignore
    │   ├── jest.config.js
    │   ├── k8s
    │   │   ├── deployment.template.yaml
    │   │   ├── ingress.template.yaml
    │   │   └── service.template.yaml
    │   ├── next.config.js
    │   ├── next-env.d.ts
    │   ├── package.json
    │   ├── .pnp.cjs
    │   ├── .pnp.loader.mjs
    │   ├── public
    │   │   ├── favicon.ico
    │   │   └── vercel.svg
    │   ├── src
    │   │   ├── components
    │   │   │   ├── App
    │   │   │   │   ├── App.test.js
    │   │   │   │   └── App.tsx
    │   │   │   ├── Board
    │   │   │   │   ├── Board.module.css
    │   │   │   │   ├── Board.test.js
    │   │   │   │   ├── Board.tsx
    │   │   │   │   └── types.ts
    │   │   │   ├── Column
    │   │   │   │   ├── Column.module.css
    │   │   │   │   ├── Column.test.js
    │   │   │   │   ├── Column.tsx
    │   │   │   │   └── types.ts
    │   │   │   ├── index.ts
    │   │   │   ├── LocalCoopGame
    │   │   │   │   ├── LocalCoopGame.module.css
    │   │   │   │   ├── LocalCoopGame.test.js
    │   │   │   │   └── LocalCoopGame.tsx
    │   │   │   ├── OnlineMultiplayerGame
    │   │   │   │   ├── OnlineMultiplayerGame.module.css
    │   │   │   │   ├── OnlineMultiplayerGame.test.js
    │   │   │   │   └── OnlineMultiplayerGame.tsx
    │   │   │   ├── StartGame
    │   │   │   │   ├── StartGame.module.css
    │   │   │   │   ├── StartGame.test.js
    │   │   │   │   └── StartGame.tsx
    │   │   │   └── Tile
    │   │   │       ├── Tile.module.css
    │   │   │       ├── Tile.test.js
    │   │   │       ├── Tile.tsx
    │   │   │       └── types.ts
    │   │   ├── external_services
    │   │   │   └── game_api_client.ts
    │   │   └── pages
    │   │       ├── api
    │   │       │   └── hello.ts
    │   │       ├── _app.tsx
    │   │       ├── index.css
    │   │       └── index.tsx
    │   ├── styles
    │   │   ├── globals.css
    │   │   └── Home.module.css
    │   ├── tsconfig.json
    │   ├── .yarn
    │   ├── yarn.lock
    │   ├── .yarnrc.yml
    │   ├── cypress.json
    │   └── cypress
    │       ├── fixtures
    │       ├── integration
    │       |   ├── playGame.spec.js
    │       |   ├── playGameDiagonalP2Win.spec.js
    |       │   └── playGameDraw.spec.js
    │       ├── plugins
    │       ├── support
    │       └── videos
    ├── connect4-server
    │   ├── docker-compose.yaml
    │   ├── Dockerfile
    │   ├── .dockerignore
    │   ├── .gitignore
    │   ├── Justfile
    │   ├── k8s
    │   │   ├── configmap.template.yaml
    │   │   ├── deployment.template.yaml
    │   │   ├── ingress.template.yaml
    │   │   ├── job.template.yaml
    │   │   ├── secret.template.yaml
    │   │   └── service.template.yaml
    │   ├── migrations
    │   │   ├── alembic.ini
    │   │   ├── env.py
    │   │   ├── README
    │   │   ├── script.py.mako
    │   │   └── versions
    │   │       ├── 418a443a1c97_add_created_field.py
    │   │       └── c047b889bc99_initial_migration.py
    │   ├── README.md
    │   ├── requirements_dev.txt
    │   ├── requirements_lock.txt
    │   ├── requirements.txt
    │   └── src
    │       ├── connect4
    │       │   ├── app_logic.py
    │       │   ├── app.py
    │       │   ├── config.py
    │       │   ├── converter.py
    │       │   ├── database.py
    │       │   ├── exceptions.py
    │       │   ├── game_logic.py
    │       │   ├── __init__.py
    │       │   ├── models.py
    │       │   ├── tokens.py
    │       │   └── views.py
    │       └── tests
    │           ├── acceptance
    │           │   ├── config.py
    │           │   ├── helper.py
    │           │   ├── __init__.py
    │           │   ├── test_game.py
    │           │   └── test_status.py
    │           ├── capacity
    │           │   ├── test_parallel.py
    │           │   └── test_sequential.py
    │           ├── __init__.py
    │           └── unit
    │               ├── helper.py
    │               ├── __init__.py
    │               ├── test_app_logic.py
    │               ├── test_converter.py
    │               ├── test_exceptions.py
    │               ├── test_game_logic.py
    │               ├── test_models.py
    │               └── test_tokens.py
    └── httpbin
        └── k8s
            ├── deployment.template.yaml
            ├── ingress.template.yaml
            └── service.template.yaml
~~~