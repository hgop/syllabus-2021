# Unit testing and code analysis

## Prerequisites

Before starting this part of the assignment we need to make sure we have the following up and running:

- [ ] A running Circle CI pipeline for client and server
- [ ] A live working game

## Objectives

Today we want to add steps to make sure we don't deploy our applications without testing them first.

- [ ] Add linting step
- [ ] Add unit tests step

## Part 1 - Connect4 Client

### Step 1 - Add test job to Connect Four client

First we want to make sure our Circle CI runs our tests.

You will need to setup a node executor for version 16.13

```yaml
  node-environment:
    docker:
      - image: cimg/node:16.13
```

Add a test job to your Circle CI config.

```yaml
  node:
    executor: node-environment
    environment:
        PROJECT_DIRECTORY: ./src/connect4-client
    steps:
      - checkout
      - run: cd $PROJECT_DIRECTORY && yarn test
```

Now make sure to add the test job to the workflow.\
Whe your pipeline has passed you can move to the next step.

### Step 2 - Test our application

Let's start by installing dependencies we need to use for testing our application:

```bash
yarn add -E -D jest babel-jest @testing-library/react @testing-library/jest-dom identity-obj-proxy react-test-renderer @babel/runtime
```

Create a `jest.config.js` file in your project's root directory and add the following configuration options:

```javascript
// jest.config.js

module.exports = {
  collectCoverageFrom: [
    '**/*.{js,jsx,ts,tsx}',
    '!**/*.d.ts',
    '!**/.yarn/**',
  ],
  moduleNameMapper: {
    /* Handle CSS imports (with CSS modules)
    https://jestjs.io/docs/webpack#mocking-css-modules */
    '^.+\\.module\\.(css|sass|scss)$': 'identity-obj-proxy',

    // Handle CSS imports (without CSS modules)
    '^.+\\.(css|sass|scss)$': '<rootDir>/__mocks__/styleMock.js',

    /* Handle image imports
    https://jestjs.io/docs/webpack#handling-static-assets */
    '^.+\\.(jpg|jpeg|png|gif|webp|avif|svg)$':
      '<rootDir>/__mocks__/fileMock.js',
  },
  testPathIgnorePatterns: ['<rootDir>/.yarn/', '<rootDir>/.next/'],
  testEnvironment: 'jsdom',
  transform: {
    /* Use babel-jest to transpile tests with the next/babel preset
    https://jestjs.io/docs/configuration#transform-objectstring-pathtotransformer--pathtotransformer-object */
    '^.+\\.(js|jsx|ts|tsx)$': ['babel-jest', { presets: ['next/babel'] }],
  },
  transformIgnorePatterns: [
    '/.yarn/',
    '^.+\\.module\\.(css|sass|scss)$',
  ],
}
```

Add the Jest executable in watch mode to the package.json scripts:

```json
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start",
  "test": "jest"
}
```

After you've installed the libraries you should create a test file for your `App.tsx` component, like so:

`src/components/App/App.test.js`

```typescript
import React from "react";
import { render, screen } from "@testing-library/react";
import "@testing-library/jest-dom";
import App from "./App";

it("should render welcome message", () => {
  render(<App />);

  expect(screen.getByText("Connect Three!")).toBeInTheDocument();
});
```

Now run the tests: `yarn test`
Update the test so it passes.

Now add these tests for the Board:

`src/components/Board/Board.test.js`

```typescript
import React from 'react';
import { render, screen } from '@testing-library/react';
import '@testing-library/jest-dom';
import Board from './Board';

it('should render correct number of columns', () => {
  render(<Board columns={3} rows={4} board={[[], [], []]} onTileClick={(_) => { }} />)

  expect(screen.getAllByLabelText('column')).toHaveLength(?)
});

it('should render correct number of tiles', () => {
  render(<Board columns={2} rows={3} board={[[], []]} onTileClick={(_) => { }} />)

  expect(screen.getAllByLabelText('tile')).toHaveLength(?)
});
```

Guess the values, then run the tests, and adjust until they pass.

Now use these [API docs](https://testing-library.com/docs/react-testing-library/api) for the component testing library and finish testing all the client's components.
NOTE: you should add tests to `App` and `Board` as needed. List of components:

- [ ] App
- [ ] Board
- [ ] Column
- [ ] LocalCoopGame
- [ ] OnlineMultiplayerGame
- [ ] StartGame
- [ ] Tile

You'll need to make sure you're testing all major states that each component can inhibit.

### Step 3 - Add a linter to our application

> A linter is great for identifying errors when you use standard rules. Remember, a linter analyzes your code for stylistic and programming errors against the rules it knows. If part of your code breaks the standard rules, this can pose a problem.

#### Add lint script

Eslint comes included when we uses the Create React App script to create our application. We want to add a yarn command to run lint on the following files: `ts, tsx` in the directory `src`.

You do that by adding a script in your `package.json`, like so:

```json
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "jest"
  },
```

#### Add lint Job

Now add a `lint` job in your Circle CI config that runs this script.

#### Actually have some lint rules

The default lint rules don't really help you keep your code consistent. We need more rules.
Let's fix it by initializing eslint in our application. You can do that by running:

`yarn lint`

And answering the following questions:

```text
# You'll see a prompt like this:
#
# ? How would you like to configure ESLint?
#
# ❯   Base configuration + Core Web Vitals rule-set (recommended)
#     Base configuration
#     None
```

Now run `yarn lint` and observe the warnings and errors you get. Fix the errors, commit and push your code.
NOTE: You can also remove rules that you don't agree with, e.g. if we don't like the rule that disallows empty interfaces we can set it as "off" in the `.eslintrc.js` config, like so:

```json
  "rules": {
    "@typescript-eslint/no-empty-interface": "off"
  }
```

## Part 2 - Connect4 Server

### Part 1 - Add a linter to our server

You will need to setup a python executor for version 3.8

```yaml
  python-environment:
    docker:
      - image: cimg/python:3.8
```

And add a lint job:

```yaml
  python:
    executor: python-environment
    environment:
        PROJECT_DIRECTORY: ./src/connect4-server
        PYTHONPATH: /home/circleci/project/src/connect4-server/src
    steps:
      - checkout
      - run: pip install -r $PROJECT_DIRECTORY/requirements.txt
      - run: pip install -r $PROJECT_DIRECTORY/requirements_dev.txt
      - run: mypy $PROJECT_DIRECTORY/src/tests/unit/
```

Remember to add lint to the config workflow.

### Part 2 - Setup unit tests

Add `pytest` to your dev requirements.

`tests/unit/helper.py`

```python
from connect4 import models
from typing import List


def get_empty_board() -> List[List[models.Tile]]:
    return [
        [
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
        ],
        [
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
        ],
        [
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
        ],
        [
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
        ],
        [
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
        ],
        [
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
        ],
        [
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
            models.Tile.EMPTY,
        ],
    ]


def get_initial_game_state() -> models.Game:
    return models.Game(
        gameId="",
        active=True,
        winner=None,
        activePlayer=models.Player.ONE,
        board=get_empty_board(),
    )
```

`tests/unit/test_converter.py`

```python
from connect4 import converter
from connect4.models import Player, Tile
from tests.unit import helper


class TestOptionalIntToPlayer:
    def test_none(self):
        assert converter.optional_int_to_player(None) == None

    def test_1(self):
        assert converter.optional_int_to_player(1) == Player.ONE

    def test_2(self):
        assert converter.optional_int_to_player(2) == Player.TWO


class TestOptionalPlayerToInt:
    def test_none(self):
        assert converter.optional_player_to_int(None) == None

    def test_1(self):
        assert converter.optional_player_to_int(Player.ONE) == 1

    def test_2(self):
        assert converter.optional_player_to_int(Player.TWO) == 2


class TestIntToPlayer:
    def test_1(self):
        assert converter.int_to_player(1) == Player.ONE

    def test_2(self):
        assert converter.int_to_player(2) == Player.TWO


class TestPlayerToInt:
    def test_1(self):
        assert converter.player_to_int(Player.ONE) == 1

    def test_2(self):
        assert converter.player_to_int(Player.TWO) == 2


class TestStrToBoard:
    def test_str_to_board_empty(self):
        empty_board = helper.get_empty_board()
        assert empty_board == converter.str_to_board(
            "000000" + "000000" + "000000" + "000000" + "000000" + "000000" + "000000"
        )

    def test_str_to_board_one_tile(self):
        board = helper.get_empty_board()
        board[1][0] = Tile.ONE
        assert board == converter.str_to_board(
            "000000" + "100000" + "000000" + "000000" + "000000" + "000000" + "000000"
        )
```

`tests/unit/test_game_logic.py`

```python
from connect4 import game_logic
from connect4.models import Tile, Player
from tests.unit import helper

# game_logic.make_move
def test_make_move_returns_new_object():
    game_input = helper.get_initial_game_state()

    game_output = game_logic.make_move(game_input, 0)

    assert game_input != game_output


def test_make_move_set_tile_one():
    game_input = helper.get_initial_game_state()

    game_output = game_logic.make_move(game_input, 0)

    assert Tile.ONE == game_output.board[0][0]


def test_make_move_set_tile_two():
    game_input = helper.get_initial_game_state()
    game_input.activePlayer = Player.TWO

    game_output = game_logic.make_move(game_input, 0)

    assert Tile.TWO == game_output.board[0][0]


def test_make_move_horizontal_win_one():
    game_input = helper.get_initial_game_state()
    game_input.board[0][0] = Tile.ONE
    game_input.board[1][0] = Tile.ONE
    game_input.board[2][0] = Tile.ONE

    game_output = game_logic.make_move(game_input, 3)

    assert False == game_output.active
    assert Player.ONE == game_output.winner


def test_make_move_horizontal_win_two():
    game_input = helper.get_initial_game_state()
    game_input.activePlayer = Player.TWO
    game_input.board[0][0] = Tile.TWO
    game_input.board[1][0] = Tile.TWO
    game_input.board[2][0] = Tile.TWO

    game_output = game_logic.make_move(game_input, 3)

    assert False == game_output.active
    assert Player.TWO == game_output.winner


def test_make_move_vertical_win_one():
    game_input = helper.get_initial_game_state()
    game_input.board[0][0] = Tile.ONE
    game_input.board[0][1] = Tile.ONE
    game_input.board[0][2] = Tile.ONE

    game_output = game_logic.make_move(game_input, 0)

    assert False == game_output.active
    assert Player.ONE == game_output.winner


def test_make_move_vertical_win_two():
    game_input = helper.get_initial_game_state()
    game_input.activePlayer = Player.TWO
    game_input.board[0][0] = Tile.TWO
    game_input.board[0][1] = Tile.TWO
    game_input.board[0][2] = Tile.TWO

    game_output = game_logic.make_move(game_input, 0)

    assert False == game_output.active
    assert Player.TWO == game_output.winner


# TODO test diagonal wins

# game_logic.is_column_full
def test_is_column_full_empty():
    game_input = helper.get_initial_game_state()

    is_column_full = game_logic.is_column_full(game_input, 0)

    assert False == is_column_full


def test_is_column_full_full():
    game_input = helper.get_initial_game_state()
    for y in range(6):
        game_input.board[0][y] = Tile.ONE

    is_column_full = game_logic.is_column_full(game_input, 0)

    assert True == is_column_full
```

`tests/unit/test_app_logic.py`

```python
from connect4 import app_logic, models
from unittest.mock import patch
from tests.unit import helper
from unittest.mock import Mock


def test_index():
    message, code = app_logic.index()
    assert message == "Game Server"
    assert code == 200


def test_status():
    message, code = app_logic.status()
    assert message == "Running"
    assert code == 200


@patch("connect4.database.create_game")
@patch("connect4.database.add_player_to_game")
@patch("connect4.tokens.generate_token")
def test_create_game(mock_generate_token, mock_add_player_to_game, mock_create_game):
    mock_generate_token.side_effect = ["token1", "token2"]
    mock_create_game.return_value = None
    mock_add_player_to_game.return_value = None
    message, code = app_logic.create_game({})
    assert {
        "gameId": "token1",
        "active": True,
        "playerId": "token2",
        "winner": None,
        "playerNumber": models.Player.ONE,
        "activePlayer": models.Player.ONE,
        "playerCount": 1,
        "board": helper.get_empty_board(),
    } == message
    assert code == 201


# TODO
```

You should be able to run the tests by doing:

```bash
pytest ./src/tests/unit
```

Your server is missing some game logic, so your test will fail. Still you should move on to the next step and add the game logic after you've added the test job to your CI pipeline.

### Part 3 - Setup unit tests job in CircleCI

```yaml
  python:
    executor: python-environment
    environment:
        PROJECT_DIRECTORY: ./src/connect4-server
        PYTHONPATH: /home/circleci/project/src/connect4-server/src
    steps:
      - checkout
      - run: pip install -r $PROJECT_DIRECTORY/requirements.txt
      - run: pip install -r $PROJECT_DIRECTORY/requirements_dev.txt
      - run: mypy $PROJECT_DIRECTORY/src/tests/unit/
      - run: [TODO run pytest]
```

### Part 4 - Coverage

Add `pytest-cov` package to your dev requirements.

Now you can see the code coverage of your tests using:

```bash
pytest --cov=connect4 tests/unit
```

### Part 5 - More tests and Implementation

- Create unit tests for all the server code except (`app.py`, `database.py` and `views.py`).

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
