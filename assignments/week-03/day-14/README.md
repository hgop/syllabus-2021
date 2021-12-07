# Week 3 Assignment

**Turn in your GitHub repository URL into Canvas solo or in pairs (please add `kthorri`, and `ironpeak` as collaborators)**

Your Circle CI Pipeline should be running:

- Lint
- Unit-Test
- Build
- Publish
- Acceptance-Test + E2E
- Capacity-Test
- Deploy

You should store all the source files in your repository:

```bash
├── Justfile
├── assignments
│   ├── day01
│   │   └── answers.md
│   ├── day02
│   │   └── answers.md
│   │── day09
│   │   └── answers.md
│   ├── day11
│   │   └── answers.md
│   └── readme.md
├── docker-compose.yaml
├── scripts
│   ├── ci
│   │   ├── database
│   │   │   └── create_database.sh
│   │   ├── deploy
│   │   │   └── create.sh
│   │   └── yaml
│   │       └── merge.sh
│   └── verify_local_dev_environment.sh
└── src
    ├── connect4-client
    │   ├── cypress
    │   │    ├── downloads
    │   │    ├── fixtures
    │   │    ├── integration
    │   │    ├── plugins
    │   │    ├── screenshots
    │   │    ├── support
    │   │    └── videos
    │   ├── cypress.json
    │   ├── Dockerfile
    │   ├── jest.config.js
    │   ├── k8s
    │   │   ├── configmap.template.yaml
    │   │   ├── deployment.template.yaml
    │   │   ├── ingress.template.yaml
    │   │   └── service.template.yaml
    │   ├── next-env.d.ts
    │   ├── next.config.js
    │   ├── package.json
    │   ├── public
    │   │   ├── favicon.ico
    │   │   └── vercel.svg
    │   ├── src
    │   │   ├── components
    │   │   │   ├── App
    │   │   │   │   ├── App.tsx
    │   │   │   │   └── app.test.js
    │   │   │   ├── Board
    │   │   │   │   ├── Board.module.css
    │   │   │   │   ├── Board.test.js
    │   │   │   │   ├── Board.tsx
    │   │   │   │   └── types.ts
    │   │   │   ├── Column
    │   │   │   │   ├── Column.module.css
    │   │   │   │   ├── Column.test.js
    │   │   │   │   ├── Column.tsx
    │   │   │   │   └── types.ts
    │   │   │   ├── LocalCoopGame
    │   │   │   │   ├── LocalCoopGame.module.css
    │   │   │   │   ├── LocalCoopGame.test.js
    │   │   │   │   └── LocalCoopGame.tsx
    │   │   │   ├── OnlineMultiplayerGame
    │   │   │   │   ├── OnlineMultiplayerGame.module.css
    │   │   │   │   ├── OnlineMultiplayerGame.test.js
    │   │   │   │   └── OnlineMultiplayerGame.tsx
    │   │   │   ├── StartGame
    │   │   │   │   ├── StartGame.module.css
    │   │   │   │   ├── StartGame.test.js
    │   │   │   │   └── StartGame.tsx
    │   │   │   ├── Tile
    │   │   │   │   ├── Tile.module.css
    │   │   │   │   ├── Tile.test.js
    │   │   │   │   ├── Tile.tsx
    │   │   │   │   └── types.ts
    │   │   │   └── index.ts
    │   │   ├── external_services
    │   │   │   └── game_api_client.ts
    │   │   └── pages
    │   │       ├── _app.tsx
    │   │       ├── api
    │   │       │   └── hello.ts
    │   │       ├── index.css
    │   │       └── index.tsx
    │   ├── styles
    │   │   ├── Home.module.css
    │   │   └── globals.css
    │   ├── tsconfig.json
    │   └── yarn.lock
    ├── connect4-server
    │   ├── Dockerfile
    │   ├── Justfile
    │   ├── README.md
    │   ├── docker-compose.yaml
    │   ├── k8s
    │   │   ├── configmap.template.yaml
    │   │   ├── deployment.template.yaml
    │   │   ├── ingress.template.yaml
    │   │   ├── job.template.yaml
    │   │   ├── secret.template.yaml
    │   │   └── service.template.yaml
    │   ├── migrations
    │   │   ├── README
    │   │   ├── alembic.ini
    │   │   ├── env.py
    │   │   ├── script.py.mako
    │   │   └── versions
    │   │       ├── 0068d3ebb99e_add_created_field.py
    │   │       └── c047b889bc99_initial_migration.py
    │   ├── requirements.txt
    │   ├── requirements_dev.txt
    │   ├── requirements_lock.txt
    │   └── src
    │       ├── connect4
    │       │   ├── __init__.py
    │       │   ├── app.py
    │       │   ├── app_logic.py
    │       │   ├── config.py
    │       │   ├── converter.py
    │       │   ├── database.py
    │       │   ├── exceptions.py
    │       │   ├── game_logic.py
    │       │   ├── models.py
    │       │   ├── tokens.py
    │       │   └── views.py
    │       └── tests
    │           ├── __init__.py
    │           ├── acceptance
    │           │   ├── __init__.py
    │           │   ├── config.py
    │           │   ├── helper.py
    │           │   ├── test_game.py
    │           │   └── test_status.py
    │           ├── capacity
    │           │   ├── test_parallel.py
    │           │   └── test_sequential.py
    │           └── unit
    │               ├── __init__.py
    │               ├── helper.py
    │               ├── test_app_logic.py
    │               ├── test_converter.py
    │               ├── test_exceptions.py
    │               ├── test_game_logic.py
    │               ├── test_models.py
    │               └── test_tokens.py
    └── httpbin
        └── k8s
            ├── deployment.template.yaml
            ├── ingress.template.yaml
            └── service.template.yaml
```

Your `README.md` file should include the URL to the instances running:

- The game client `connect4.{{team-name}}.hgopteam.com`
- The game server `connect4-server.{{team-name}}.hgopteam.com`

If you did anything extra, you should list it up in the `README.md` if you
want us to take it into account.
