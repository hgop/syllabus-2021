# Week 2 Assignment

**Turn in your GitHub repository URL into Canvas solo or in pairs (please add `kthorri`, and `ironpeak` as collaborators)**

Your Circle CI Pipeline should be running:
- Lint
- Unit-Test
- Build
- Publish
- Acceptance-Test
- Capacity-Test
- Deploy

You should store all the source files in your repository:
```bash
.
├── assignments
│   ├── day01
│   │   └── answers.md
│   ├── day02
│   │   └── answers.md
│   └── day09
│       └── answers.md
├── .circleci
│   └── config.yml
├── docker-compose.yml
├── .git
├── .gitignore
├── Justfile
├── package.json
├── README.md
├── scripts
│   ├── ci
│   │   ├── database
│   │   │   └── create_database.sh
│   │   ├── deploy
│   │   │   └── create.sh
│   │   └── yaml
│   │       └── merge.sh
│   └── verify_local_dev_environment
├── src
│   ├── connect4-client
│   │   ├── Dockerfile
│   │   ├── .dockerignore
│   │   ├── .eslintrc.json
│   │   ├── .gitignore
│   │   ├── jest.config.js
│   │   ├── k8s
│   │   │   ├── configmap.template.yaml
│   │   │   ├── deployment.template.yaml
│   │   │   ├── ingress.template.yaml
│   │   │   └── service.template.yaml
│   │   ├── next.config.js
│   │   ├── next-env.d.ts
│   │   ├── package.json
│   │   ├── .pnp.cjs
│   │   ├── .pnp.loader.mjs
│   │   ├── public
│   │   │   ├── favicon.ico
│   │   │   └── vercel.svg
│   │   ├── src
│   │   │   ├── components
│   │   │   │   ├── App
│   │   │   │   │   ├── App.test.js
│   │   │   │   │   └── App.tsx
│   │   │   │   ├── Board
│   │   │   │   │   ├── Board.module.css
│   │   │   │   │   ├── Board.test.js
│   │   │   │   │   ├── Board.tsx
│   │   │   │   │   └── types.ts
│   │   │   │   ├── Column
│   │   │   │   │   ├── Column.module.css
│   │   │   │   │   ├── Column.test.js
│   │   │   │   │   ├── Column.tsx
│   │   │   │   │   └── types.ts
│   │   │   │   ├── index.ts
│   │   │   │   ├── LocalCoopGame
│   │   │   │   │   ├── LocalCoopGame.module.css
│   │   │   │   │   ├── LocalCoopGame.test.js
│   │   │   │   │   └── LocalCoopGame.tsx
│   │   │   │   ├── OnlineMultiplayerGame
│   │   │   │   │   ├── OnlineMultiplayerGame.module.css
│   │   │   │   │   ├── OnlineMultiplayerGame.test.js
│   │   │   │   │   └── OnlineMultiplayerGame.tsx
│   │   │   │   ├── StartGame
│   │   │   │   │   ├── StartGame.module.css
│   │   │   │   │   ├── StartGame.test.js
│   │   │   │   │   └── StartGame.tsx
│   │   │   │   └── Tile
│   │   │   │       ├── Tile.module.css
│   │   │   │       ├── Tile.test.js
│   │   │   │       ├── Tile.tsx
│   │   │   │       └── types.ts
│   │   │   ├── external_services
│   │   │   │   └── game_api_client.ts
│   │   │   ├── .idea
│   │   │   │   └── ..
│   │   │   └── pages
│   │   │       ├── api
│   │   │       │   └── hello.ts
│   │   │       ├── _app.tsx
│   │   │       ├── index.css
│   │   │       └── index.tsx
│   │   ├── styles
│   │   │   ├── globals.css
│   │   │   └── Home.module.css
│   │   ├── tsconfig.json
│   │   ├── .vscode
│   │   │   ├── extensions.json
│   │   │   └── settings.json
│   │   ├── .yarn
│   │   │   └── ..
│   ├── connect4-server
│   │   ├── docker-compose.yaml
│   │   ├── Dockerfile
│   │   ├── .dockerignore
│   │   ├── .gitignore
│   │   ├── Justfile
│   │   ├── k8s
│   │   │   ├── configmap.template.yaml
│   │   │   ├── deployment.template.yaml
│   │   │   ├── ingress.template.yaml
│   │   │   ├── job.template.yaml
│   │   │   ├── secret.template.yaml
│   │   │   └── service.template.yaml
│   │   ├── migrations
│   │   │   ├── alembic.ini
│   │   │   ├── env.py
│   │   │   ├── README
│   │   │   ├── script.py.mako
│   │   │   └── versions
│   │   │       └── c047b889bc99_initial_migration.py
│   │   ├── README.md
│   │   ├── scripts
│   │   │   └── create_database.sh
│   │   ├── src
│   │   │   ├── connect4
│   │   │   │   ├── app_logic.py
│   │   │   │   ├── app.py
│   │   │   │   ├── config.py
│   │   │   │   ├── converter.py
│   │   │   │   ├── database.py
│   │   │   │   ├── exceptions.py
│   │   │   │   ├── game_logic.py
│   │   │   │   ├── __init__.py
│   │   │   │   ├── models.py
│   │   │   │   ├── tokens.py
│   │   │   │   └── views.py
│   │   │   ├── .idea
│   │   │   │   └── workspace.xml
│   │   │   ├── requirements_dev.txt
│   │   │   ├── requirements.txt
│   │   │   ├── tests
│   │   │   │   ├── acceptance
│   │   │   │   │   ├── config.py
│   │   │   │   │   ├── config.pyc
│   │   │   │   │   ├── helper.py
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   ├── __init__.pyc
│   │   │   │   │   ├── test_gameplay.py
│   │   │   │   │   └── test_status.py
│   │   │   │   ├── capacity
│   │   │   │   │   ├── config.py
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   ├── __pycache__
│   │   │   │   │   │   └── ..
│   │   │   │   │   ├── test_parallel.py
│   │   │   │   │   └── test_sequential.py
│   │   │   │   ├── __init__.py
│   │   │   │   ├── __init__.pyc
│   │   │   │   ├── __pycache__
│   │   │   │   │   └── __init__.cpython-38.pyc
│   │   │   │   └── unit
│   │   │   │       ├── helper.py
│   │   │   │       ├── __init__.py
│   │   │   │       ├── test_app_logic.py
│   │   │   │       ├── test_converter.py
│   │   │   │       ├── test_exceptions.py
│   │   │   │       ├── test_game_logic.py
│   │   │   │       ├── test_generate_token.py
│   │   │   │       └── test_models.py
│   │   │   └── venv
│   │   │       └── ..
│   ├── httpbin
│   │   └── k8s
│   │       ├── deployment.template.yaml
│   │       ├── ingress.template.yaml
│   │       └── service.template.yaml
│   └── .pytest_cache
│       └── ..
├── .yarn
│   └── releases
└── yarn.lock
```

Your `README.md` file should include the URL to the instances running:
- The game client `connect4.{{team-name}}.hgopteam.com`
- The game server `connect-four-server-production.{{team-name}}.hgopteam.com`

If you did anything extra, you should list it up in the `README.md` if you
want us to take it into account.
