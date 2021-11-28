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
├── .circleci
│   └── config.yml
├── .gitignore
├── docker-compose.yaml
├── Justfile
├── README.md
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
    │   ├── .yarn
    │   ├── .dockerignore
    │   ├── .gitignore
    │   ├── Dockerfile
    │   ├── k8s
    │   ├── public
    │   ├── src
    │   │   ├── components
    │   │   │   ├── App
    │   │   │   ├── Board
    │   │   │   ├── Column
    │   │   │   ├── index.ts
    │   │   │   ├── LocalCoopGame
    │   │   │   ├── OnlineMultiplayerGame
    │   │   │   ├── StartGame
    │   │   │   └── Tile
    │   │   ├── external_services
    │   │   └── pages
    │   ├── styles
    │   ├── tsconfig.json
    │   ├── package.json
    │   └── ...
    ├── connect4-server
    │   ├── .dockerignore
    │   ├── .gitignore
    │   ├── docker-compose.yaml
    │   ├── Dockerfile
    │   ├── Justfile
    │   ├── k8s
    │   │   ├── configmap.template.yaml
    │   │   ├── deployment.template.yaml
    │   │   ├── ingress.template.yaml
    │   │   ├── job.template.yaml
    │   │   ├── secret.template.yaml
    │   │   └── service.template.yaml
    │   ├── migrations
    │   │   ├── alembic.ini
    │   │   ├── env.py
    │   │   ├── README
    │   │   ├── script.py.mako
    │   │   └── versions
    │   │       └── c047b889bc99_initial_migration.py
    │   ├── README.md
    │   ├── requirements_dev.txt
    │   ├── requirements.txt
    │   └── src
    │       ├── connect4
    │       │   ├── app_logic.py
    │       │   ├── app.py
    │       │   ├── config.py
    │       │   ├── converter.py
    │       │   ├── database.py
    │       │   ├── exceptions.py
    │       │   ├── game_logic.py
    │       │   ├── __init__.py
    │       │   ├── models.py
    │       │   ├── tokens.py
    │       │   └── views.py
    │       └── tests
    │           ├── __init__.py
    │           ├── acceptance
    │           │   ├── __init__.py
    │           │   ├── config.py
    │           │   └── test_*.py
    │           ├── capacity
    │           │   ├── __init__.py
    │           │   ├── config.py
    │           │   └── test_*.py
    │           └── unit
    │               ├── helper.py
    │               ├── __init__.py
    │               ├── test_app_logic.py
    │               ├── test_converter.py
    │               ├── test_game_logic.py
    │               └── test_tokens.py
    └── httpbin
        └── k8s
            ├── deployment.template.yaml
            ├── ingress.template.yaml
            └── service.template.yaml
```

Your `README.md` file should include the URL to the instances running:
- The game client `connect4.{{team-name}}.hgopteam.com`
- The game server `connect-four-server-production.{{team-name}}.hgopteam.com`

If you did anything extra, you should list it up in the `README.md` if you
want us to take it into account.
