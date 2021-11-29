# Pipeline

## Prerequisites

Before starting this part of the assignment we need to make sure we have the following up and running:

- [ ] An initial React web application
- [ ] A running Circle CI pipeline that:
  - On every push:
    - [ ] Builds a docker image from your code
    - [ ] Publishes a docker image from your code
  - On every merge to the main branch:
    - [ ] Builds a docker image from your code
    - [ ] Publishes a docker image from your code
    - [ ] Deploys your application to your kubernetes cluster

## Get up to date

If something wasn't working for you in week 1 you can use the current solution repositories to review and update your own projects.\
NOTE: You'll still need to make sure that you replace the team name and the docker repository if you copy/paste code.

- [Week 1 solution](https://github.com/hgop/week-1-solution)

Make sure you get help as soon as possible if something isn't working from week 1.

## Objectives

Today we want to add the actual game code. We've already made a simple connect-four game client for you to use. You'll also be setting up your own server, but only after your client is ready.

- [ ] Deploy a running Connect Four game client
- [ ] Setup your own Connect Four game server
- [ ] Build the game server in your pipeline
- [ ] Connect your server to your client

## Part 1 - Connect Four Client

The best practices when creating frontend clients is to try to make it as "dumb" as possible. I.e. if you can, try to have most/all of the logic on the server. Sometimes this is not possible and it's fine, but in those cases always try to extract your logic away from your UI components.

We split the client into two main parts.

1. UI components (only worried about how the website looks and is structured)
2. External Services (handles connecting to external services, in our case the game server)

### Step 1 - Add the Connect Four Client code

Use this repository: [Client Reference](https://github.com/hgop/hgop-connect4-client-base-2021),
and replace the ones you already have in `./src/connect4-client/`.

You can start the client by doing `yarn start`, it will be communicating with the TA game server for now.

### Step 2 - Test

Start the project up locally and make sure it's working.

### Step 3 - Deploy

Commit and push your code and make sure the game is working.

## Part 2 - Connect Four Server

Now it's time to setup your own server. We've made a simple server for you to use as a base.

### Step 1 - Server Code

Use this repository: [Server Reference](https://github.com/hgop/hgop-connect4-server-base-2021) and copy it into `./src/connect4-server/`.

### Step 2 - Create Database

The Python connect four server uses a database. You are currently running all your applications on one kubernetes cluster and one environment. We'll take a better look at different environments later.\
A database would usually be setup outside of kubernetes ([AWS RDS](https://aws.amazon.com/rds/)), but we won't need such a robust solution for this course.\
To create the database you can use the script `scripts/create_database.sh` from the connect4 server base. We'll need to specify a few environment variables for our database:

```bash
export DATABASE_USERNAME="{{YOUR_CHOSEN_USERNAME}}"
export DATABASE_PASSWORD="{{YOUR_CHOSEN_PASSWORD}}"
./scripts/create_database.sh default
```

You can now check your cluster to see if your database deployment was a success:

```bash
kubectl get pods
```

Should a least include your connect4 client and your newly created database:

```bash
NAME                                 READY   STATUS    RESTARTS   AGE
connect4-client-544ffc6678-hqw2p     1/1     Running   0          11h
connect4-database-0                  1/1     Running   0          15m
```

Copy the name of your database pod to get the logs:

```bash
kubectl logs {{YOUR_DATABASE_POD}}
```

If the deployment was successful you should get logs that look something like this:

```bash
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.
...
..
...
2021-11-18 19:43:36.207 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2021-11-18 19:43:36.208 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2021-11-18 19:43:36.211 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2021-11-18 19:43:36.216 UTC [51] LOG:  database system was shut down at 2021-11-18 19:43:36 UTC
2021-11-18 19:43:36.220 UTC [1] LOG:  database system is ready to accept connections
```

## Step 3 - Server Pipeline

Integrate the server into your pipeline, make sure that you use a separate docker repository (not account) for your server docker images.

```bash
Build: Build both client and server
Publish: Publish both client and server
Deploy: Deploy client, server and httpbin
```

If the build or publish step fails for either the client or server, nothing should get deployed.\

To deploy the server we'll need to specify a few environment variable so instead of having a lot of config in the Circle CI config we've created a script to take care of it for us. You'll need to run the script `scripts/ci/deploy/create.sh` to generate a kubernetes yaml file for us.

```bash
# ./scripts/ci/deploy/create.sh <service> <tag>
./scripts/ci/deploy/create.sh "connect4-client" "${CIRCLE_SHA1}"
./scripts/ci/deploy/create.sh "connect4-server" "${CIRCLE_SHA1}"
./scripts/ci/deploy/create.sh "httpbin" "${CIRCLE_SHA1}"
```

Here is a base to help get you started:\
`scripts/ci/deploy/create.sh`

```bash
#!/bin/bash

set -euo pipefail

if [ "$#" -ne 2 ]; then
    echo "Invalid number of arguments"
    echo "./scripts/ci/deploy/create.sh <service> <tag>"
    exit 1
fi

SERVICE="$1"
TAG="$2"

env_prefix="$(echo ${SERVICE}- | awk '{print toupper($0)}' | sed 's/-/_/g')"
env_variables="$(printenv | { grep "${env_prefix}" || test $? = 1; } | sed "s/^${env_prefix}//g")"

content="$(./scripts/ci/yaml/merge.sh ./src/${SERVICE}/k8s)"

content="$(echo "${content}" | sed "s/{{IMAGE_TAG}}/${TAG}/g")"

for variable in ${env_variables}; do
    variable_key=#TODO
    variable_value=#TODO
    content=#TODO
done

echo "${content}"
```

The script should read all environment variables starting with all caps service name (e.g. `CONNECT4_CLIENT_`) and substitute the occurance in
kubernetes yaml template files for that service.

Example:

`CONNECT4_CLIENT_DATABASE_PASSWORD` should substitute `{{DATABASE_PASSWORD}}` in the connect4-client k8s template files.

It should use the tag argument to replace the deployment's image tag.

It should print the result to stdout.

Then we will use it in CircleCI:

```yaml
    - run:
        name: "Generating connect4 client yaml"
        command: ./scripts/ci/deploy/create.sh "connect4-client" "${CIRCLE_SHA1}" > connect4-client.yaml
    - run:
        name: "Generating connect4 server yaml"
        command: ./scripts/ci/deploy/create.sh "connect4-server" "${CIRCLE_SHA1}" > connect4-server.yaml
    - run:
        name: "Generating httpbin yaml"
        command: ./scripts/ci/deploy/create.sh "httpbin" "${CIRCLE_SHA1}" > httpbin.yaml
    - run:
        name: "Clean previous job"
        command: kubectl delete job connect4-migrations || exit 0
    - run:
        name: "Deploy"
        command: kubectl apply -f connect4-client.yaml -f connect4-server.yaml -f httpbin.yaml
```

You will need to setup the required environment variables in CircleCI to get it working.

### Step 4 - Connect server

NOTE: The server does not work the same as the server you're currently using, i.e. there is some logic missing, but you'll fix that on Day 7.

You will need to set the API_URL environment variable for the server.

## Part 3 - Local Development

To improve local development setup a `docker-compose.yaml` file in the root of your repository.

It should be very similar to the `./src/connect4-server/docker-compose.yaml` file from the server-base repository, except it should also
spin up the connect4-client (on port 3000) and httpbin (on port 4000).

Add a recipe `start` to the Justfile.

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
    │       └── connect4
    │           ├── app_logic.py
    │           ├── app.py
    │           ├── config.py
    │           ├── converter.py
    │           ├── database.py
    │           ├── exceptions.py
    │           ├── game_logic.py
    │           ├── __init__.py
    │           ├── models.py
    │           ├── tokens.py
    │           └── views.py
    └── httpbin
        └── k8s
            ├── deployment.template.yaml
            ├── ingress.template.yaml
            └── service.template.yaml
```
