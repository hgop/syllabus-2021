# Acceptance Testing

## Prerequisites

Before starting this part of the assignment we need to make sure we have the following up and running:

- [ ] A running Circle CI pipeline
- [ ] A live working game with server

## Objectives

Today we want to add additional steps to make sure we don't deploy our applications without them working correctly

- [ ] Add acceptance testing to your pipeline
- [ ] Add acceptance testing to your server

## Part 1 - Connect Four Server

### Step 1 - Setup acceptance tests

Add `requests` package to your test requirements.

`tests/acceptance/config.py`

```python
import os

API_URL = os.environ["API_URL"]
```

`tests/acceptance/test_status.py`

```python
from tests.acceptance import config
from typing import List
import requests

def test_status():
    response = requests.get(config.API_URL + "/status")
    assert "Running" == response.text
```

Test your acceptance test against your production url:

```bash
API_URL=https://connect4-server.{{team-name}}.hgopteam.com/ pytest ./src/tests/acceptance
```

### Step 2 - Deploy acceptance environment

We will deploy another instance of our game server to test against in it's own namespace in
kubernetes.

```yaml
  create-acceptance-environment:
    docker:
    - image: cimg/base:stable
    environment:
      DATABASE_USERNAME: connect4
      DATABASE_PASSWORD: connect4
      CONNECT4_SERVER_DATABASE_USERNAME: Y29ubmVjdDQ=
      CONNECT4_SERVER_DATABASE_PASSWORD: Y29ubmVjdDQ=
      CONNECT4_SERVER_HOST: connect4-server.acceptance.{{TEAM_NAME}}.hgopteam.com
    steps:
    - checkout
    - kubernetes/install-kubectl
    - kubernetes/install-kubeconfig:
        kubeconfig: KUBECONFIG_DATA
    - run: ./scripts/ci/database/create_database.sh "acceptance"
    - run: ./scripts/ci/deploy/create.sh "connect4-server" "${CIRCLE_SHA1}" > connect4-server.yaml
    - run: kubectl delete --namespace acceptance job connect4-migrations || exit 0
    - run: kubectl apply --namespace acceptance -f connect4-server.yaml
```

You can check the status of pods by using the namespace flag:

```bash
kubectl get pods --namespace acceptance
# or
kubectl get pods -n acceptance
```

### Step 3 - Run acceptance tests

Run your acceptance tests agains the newly deployed instance:

```yaml
  acceptance-test:
    executor: python-environment
    environment:
      PROJECT_DIRECTORY: ./src/connect4-server
      PYTHONPATH: /home/circleci/project/src/connect4-server/src
      API_URL: "https://connect4-server.acceptance.{{TEAM_NAME}}.hgopteam.com/"
    steps:
      - checkout
      - run: pip install -r $PROJECT_DIRECTORY/requirements_dev.txt
      - run: pytest $PROJECT_DIRECTORY/src/tests/acceptance
```

### Step 4 - Create another test

Create a test that plays through an entire game, simulating two users.

Here is a helper class you can use:

`tests/acceptance/helper.py`

```python
from connect4 import models, converter
from tests.acceptance import config
from typing import List
import requests

class InitializedGame:
    gameId: str
    playerOneId: str
    playerTwoId: str

    def __init__(
        self,
        gameId: str,
        playerOneId: str,
        playerTwoId: str
    ) -> None:
        self.gameId = gameId
        self.playerOneId = playerOneId
        self.playerTwoId = playerTwoId

def initialize_game() -> InitializedGame:
    response = requests.post(config.API_URL + "/create_game")
    assert response.status_code == 201

    json = response.json()
    gameId = json["gameId"]
    playerOneId = json["playerId"]

    response = requests.post(config.API_URL + "/join_game", json={
        "gameId": gameId
    })
    assert response.status_code == 202

    json = response.json()
    playerTwoId = json["playerId"]

    return InitializedGame(
        gameId=gameId,
        playerOneId=playerOneId,
        playerTwoId=playerTwoId
    )

def make_move(gameId: str, playerId: str, column: int) -> models.MakeMoveResponse:
    response = requests.post(config.API_URL + "/make_move", json={
        "gameId": gameId,
        "playerId": playerId,
        "column": column
    })
    assert response.status_code == 202

    json = response.json()
    return models.MakeMoveResponse(
        gameId=json["gameId"],
        active=json["active"],
        playerId=json["playerId"],
        winner=converter.optional_int_to_player(json["winner"]),
        playerNumber=converter.int_to_player(json["playerNumber"]),
        activePlayer=converter.int_to_player(json["activePlayer"]),
        playerCount=json["playerCount"],
        board=converter.str_to_board(converter.board_to_str(json["board"])),
    )
```

## Handin

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
    |           |   ├── helper.py
    │           |   ├── __init__.py
    │           |   ├── config.py
    │           |   └── test_*.py
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
