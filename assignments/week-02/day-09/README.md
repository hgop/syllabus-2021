# Capacity Testing

## Prerequisites

Before starting this part of the assignment we need to make sure we have the following up and running:

- [ ] A running Circle CI pipeline
- [ ] A live working game with server
- [ ] An acceptance test that plays one whole game

## Objectives

Today we want to add additional steps to make sure we don't deploy our applications without them working correctly

- [ ] Add capacity testing to your pipeline

## Part 1 - Connect Four Server

### Part 1 - Setup capacity tests

Add `pytest-timeout` package to your dev requirements.

Where test_game is an acceptance test you created in day 8, that simulates two players playing a game. (if you named it something else you can just change it)

`tests/capacity/test_sequential.py`

~~~python
import pytest

@pytest.mark.timeout(30)
def test_sequential():
    from tests.acceptance.test_game import test_game

    for _ in range(10):
        test_game()
~~~

Test your capacity test against your production url:

~~~bash
API_URL=http://connect4-server.{{team-name}}.hgopteam.com/ pytest ./src/tests/capacity
~~~

### Part 3 - Deploy capacity environment

Setup create-capacity-environment like you did for acceptance in day 8.

### Part 4 - Run capacity tests

Setup capacity-test like you did for acceptance in day 8.

### Part 5 - Create another test

Create a test that spawns N number of threads that run X amount of games:

Note: There exist tools for capacity/performance testing for all major languages and frameworks, it's usually best not to reinvent the wheel as these tools usually come with more features. But we will be using our own "tool" to keep it simple.

`tests/capacity/test_parallel.py`

~~~python
import pytest
import threading
import time

N = 8
X = 10


class State:
    lock: threading.Lock
    gamesPlayed: int
    failed: bool

    def __init__(self) -> None:
        self.lock = threading.Lock()
        self.gamesPlayed = 0
        self.failed = False

    def increment_games_played(self) -> None:
        with self.lock:
            self.gamesPlayed += 1


def play_games(i: int, x: int, state: State) -> None:
    try:
        from tests.acceptance.test_game import test_game

        for j in range(x):
            test_game()
            state.increment_games_played()
            print(f"Thread {i}: finished game number {j}.")
    except:
        state.failed = True


@pytest.mark.timeout(120)
def test_parallel():
    threads = []
    state = State()

    for i in range(N):
        thread = threading.Thread(target=play_games, args=(i, X, state))
        threads.append(thread)

    start_time = time.time()

    for thread in threads:
        thread.start()

    while True:
        all_threads_finished = True
        for thread in threads:
            all_threads_finished = thread.is_alive() == False and all_threads_finished

        if all_threads_finished or state.failed:
            break

    print(f"Played: {state.gamesPlayed}")
    print(f"Time: {(time.time() - start_time)}")

    for thread in threads:
        thread.join()

    assert state.failed == False
~~~

Try running the test locally against your capacity environment.\
Notice the `-s` flag which tells pytest to show stdout.

~~~bash
export API_URL=https://connect4-server.capacity.dreamteam.hgopteam.com/
pytest -s ./src/tests/capacity/test_parallel.py
~~~

Run the test locally against production for:

| N   | X   |
| --- | --- |
| 1   | 10  |
| 2   | 10  |
| 4   | 10  |
| 8   | 10  |
| 16  | 10  |

Measure for each case (`./assignments/day09/answers.md`):

(The outcome should be in a readable table).

- Time
- Did it complete successfully
- Games played

Now scale up your `connect4-server` deployment to handle more load

~~~bash
kubectl --namespace capacity edit deployment connect4-server
~~~

And change `spec.replicas` to 3.

You should see 2 more pods start up:

~~~bash
kubectl --namespace capacity get pods
~~~

Run the previous test cases again and take those measurements as well.

Do you see any improvements?

### Part 6 - Replicas

Change `deployment.yaml.template` so that `spec.replicas` can be configured for each environment:\
`production: 2, acceptance: 1, capacity: 3`

Hint: `CONNECT4_SERVER_REPLICAS`

### Part 7 - Configure

Pick some sensible values for N, X and have it run as part of your capacity tests, it should:

- timeout at 30 seconds
- N > 1
- X >= 3
- Take about 15-20 seconds on average in the CI

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
    │           │   ├── __init__.py
    │           │   ├── config.py
    │           │   └── test_*.py
    │           ├── capacity
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
