# Database Migration & Security

## Objectives

Today we want to add database migration to our server, so that we can upgrade or downgrade our database schema as needed.
We will also refactor our database code so that it's using an ORM to improve code readability and security.

- [ ] Add migrations for our database.
- [ ] Setup security scanning for our repository.

## Part 1 - Database Migrations

Now that we ORM and migration framework up and running lets try to update our schema:

Add `created` field to your game and player entities:

~~~python
    sa.Column('created', sa.DateTime, nullable=False, default=datetime.utcnow)
~~~

Fix the `add_migration` we gave you in the server base to handle new migrations:

~~~Justfile
add_migration message:
    docker kill "${DATABASE_CONTAINER}" || true
    docker rm "${DATABASE_CONTAINER}" || true
    docker run -d \
        --name="${DATABASE_CONTAINER}" \
        -e POSTGRES_USERNAME="${DATABASE_USERNAME}" \
        -e POSTGRES_PASSWORD="${DATABASE_PASSWORD}" \
        -e POSTGRES_DB="${DATABASE_NAME}" \
        -p "${DATABASE_PORT}:5432" \
        postgres:13-alpine
    sleep 10
    PYTHONPATH=./src FLASK_APP=./src/connect4/app.py flask db upgrade
    PYTHONPATH=./src FLASK_APP=./src/connect4/app.py flask db migrate -m "{{message}}"
    docker kill "${DATABASE_CONTAINER}" || true
    docker rm "${DATABASE_CONTAINER}" || true
~~~

Generate the migration plan for the new field:

~~~bash
just add_migration "add created field"
~~~

Notice the new migration script that was just generated in `migrations/versions/xxxxxxxxxxxx_add_created_field.py`.

Now update the game so that the game's creation time is returned in the BaseGameResponse.

### Part 2 - Security Scans

#### Step 1 - Lock file

Before we start security scanning our dependencies it's important to lock them so that they are not
changing between builds.

To generate a lock file we can use `pip-compile` from [pip-tools](https://pypi.org/project/pip-tools/):

~~~bash
pip-compile --upgrade --generate-hashes requirements.txt --output-file requirements_lock.txt
~~~

Create a recipe `lock` in your Justfile to generate a lock file.

Make sure you start using the lockfile when building the server.

In your `answer.md` write 3-5 sentances on why projects would lock their dependencies.

#### Step 2 - Dependabot

Setup `Dependabot` for your repository, it should scan both the server and client dependencies.

In your `answer.md` write 3-5 sentances on why dependabot is useful.

## Handin

repository:

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
    │   └── .yarnrc.yml
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
