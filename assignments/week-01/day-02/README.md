# Docker

## What is Docker?

[Read this article before starting the assignment](https://opensource.com/resources/what-docker)

## Part 1

### Getting started with docker

- [ ] Install Docker

**Linux only**: After the installation you should add your user to the docker user group so that you do not have to use sudo to run commands.

When you are finished you should be able to run:\
`docker run hello-world`\
and get the following:

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
...(snipped)...
```

### How do I know I'm done?

- [ ] Docker is installed.
- [ ] The docker command works without sudo privileges.

### FAQ

#### What Ubuntu do I have?

Run:\
`lsb_release -a`\
Should display (depending on your Ubuntu version):

```text
Distributor ID:   Ubuntu
Description:      Ubuntu 20.04.1 LTS
Release:          20.04
Codename:         focal
```

## Part 2

### Image vs Container

An instance of an image is called a container. You have an image, which is a set
of layers as you describe. If you start this image, you have a running container
of this image. You can have many running containers of the same image.

### Your new development environment

In the past, if you were to start writing a React app, your first order of
business was to install NodeJS and other dependencies onto your machine. But,
that creates a situation where the environment on your machine has to be just so
in order for your app to run as expected; ditto for the server that runs your
app.

With Docker, you can just grab a portable NodeJS runtime as an image, no
installation necessary. Then, your build can include the base NodeJS image right
alongside your app code, ensuring that your app, its dependencies, and the
runtime, all travel together.

These portable images are defined by something called a Dockerfile.

### Define a container with a Dockerfile

Dockerfile will define what goes on in the environment inside your container.
Access to resources like networking interfaces and disk drives is virtualized
inside this environment, which is isolated from the rest of your system, so you
have to map ports to the outside world, and be specific about what files you
want to “copy in” to that environment. However, after doing that, you can expect
that the build of your app defined in this Dockerfile will behave exactly the
same wherever it runs.

### The application

Let's create a NextJS application and run it in a Docker container.

Enter the following commands to create the application:

In your repository create a `./src` directory, once inside it:

`yarn create next-app connect4-client --typescript`

`cd connect4-client`

These commands will create a Typescript application in the folder `connect4-client`

To make sure your new application is working as expected you can run it locally on your machine by:

- Installing the dependencies:

`yarn install`

- Building the app:

`yarn build`

- Starting the application:

`yarn start`

- The application should be running on `http://localhost:3000/`

#### Yarn Berry

Let's change our package management in the new client application.

We are going to be migrating to [Yarn V3](https://yarnpkg.com/getting-started/migration)

- Start by setting the yarn version

`yarn set version berry`

- Commit the changes you have so far. This should have added a couple of new files
- We need to add a few lines to our .gitignore file configurations can be found [here](https://yarnpkg.com/getting-started/qa#which-files-should-be-gitignored)
- run `yarn install` again and commit the new lock file
- Follow the guide on how to switch to [Plug'n'Play](https://yarnpkg.com/getting-started/migration#switching-to-plugnplay)
  - Change the version of typescript in package.json to "4.4.4"
  - Run `yarn install`
- Notice that the node_modules folder is not in our project anymore
- Make sure your application build with the new file system `yarn build`

### Dockerfile

Now we want to be able to run the application easily in multiple environments that might be very different from your local setup. We can use docker to build your application into a docker image which can be run on all kinds of environments.

Create a file called `Dockerfile` in the `connect4-client` folder and copy-paste the following content into the file and replace the TODOs with the correct commands.

NextJS provides an example Dockerfile which we can use for reference.
Take notice of the different stages used [here](https://nextjs.org/docs/deployment#docker-image). This is done to minimize the final image size as only artifacts needed are available in the 'runner' stage.

Our Dockerfile can be a bit simpler, but feel free to use the approach presented in the NextJS documentation if you feel like exploring more with Dockerfiles.

```Dockerfile
# First we want to build on an image the includes the environment we need to build and run our code, we can use 'node:alpine':
TODO

# Now set the working directory to /app:
TODO

# Let's make sure all our dependencies are available
# copy package.json, yarn.lock and the .yarn directory into the current working directory:
TODO

# Now we need the rest of our application code, copy your entire folder into the image working directory:
# You should also create a .dockerignore file to improve build performance and keep unwanted files out of your container.
TODO

# Now we want to build our application code. HINT: To run package.json scripts, we can use `yarn {script}`
TODO

# Lastly we need to run the application when the container starts, set the command as yarn start:
TODO
```

### Build the app

We are ready to build the app. The inside of your repository's `./src/connect4-client` directory should contain:

```bash
connect4-client
├── .next
│   └── ...
├── .yarn
│   └── ...
├── pages
│   ├── api
│   │   └── hello.ts
│   ├── _app.tsx
│   ├── index.tsx
├── public
│   ├── favicon.ico
│   ├── vercel.svg
├── styles
│   └── ...
├── .pnp.cjs
├── .pnp.loader.mjs
├── .yarnrc.yml
├── package.json
├── tsconfig.json
├── next.config.js
├── yarn.lock
├── Dockerfile
└── ...
```

Now run the build command. This creates a Docker image, which we’re going to tag using -t so it has a friendly name.

```bash
docker build -t connect4-client .
```

Where is your built image? It’s in your machine’s local Docker image registry:

```bash
$ docker images

REPOSITORY            TAG                 IMAGE ID
connect4-client   latest              326387cea398
...
```

### Run the app's image in a docker container

**Note**: Make sure to stop the local run of the application by ctrl-c before running the docker.

Run the app and map your machine’s port 3000 to the container’s published port 3000 using -p (If we don't map the ports there is no way for us to access the port because it's only available inside the docker container):

```bash
$ docker run -p 3000:3000 connect4-client

> connect4-client@0.1.0 start /app
> react-scripts start

ℹ ｢wds｣: Project is running at http://172.17.0.2/
ℹ ｢wds｣: webpack output is served from 
ℹ ｢wds｣: Content not from webpack is served from /app/public
ℹ ｢wds｣: 404s will fallback to /
Starting the development server...

Compiled successfully!

You can now view connect4-client in the browser.

  Local:            http://localhost:3000
  On Your Network:  http://172.17.0.2:3000

Note that the development build is not optimized.
To create a production build, use npm run build.
```

Your shell will enter the docker container and won't allow you to ctrl+c
out of the process.

The app is a simple React App and can be visited by going to `http://localhost:3000` in your preferred browser.

Now finally to exit the docker container, using another shell kill the
docker container by:

```bash
$ docker container list

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
931853651e89        connect4-client "/bin/sh -c 'node ap…"   53 seconds ago      Up 51 seconds       0.0.0.0:3000->3000/tcp hopeful_golick

$ docker container stop 931853651e89
```

Now let’s run the app in the background, in detached mode (-d flag):

```bash
docker run -d -p 3000:3000 connect4-client
```

You get the long container ID for your app and then are kicked back to your terminal. Your container is running in the background. You can also see the abbreviated container ID with docker container ls (and both work interchangeably when running commands):

```bash
$ docker container list

CONTAINER ID        IMAGE               COMMAND                   CREATED           PORTS
1fa4ab2cf395        connect4-client "docker-entrypoint.s…"    28 seconds ago   0.0.0.0:3000->3000/tcp
```

You’ll see that that the app is running on `http://localhost:3000/`.

Now use docker container stop to end the process, using the CONTAINER ID, like
so:

```bash
docker container stop 1fa4ab2cf395
```

**Pro Tip** When running commands that require the `CONTAINER ID`, e.g.\
`docker container stop 1fa4ab2cf395`\
it's usually enough to write the first three characters or so, i.e.\
`docker container stop 1fa`

### Share your image

To demonstrate the portability of what we just created, let’s upload our built
image and run it somewhere else. After all, you’ll need to learn how to push to
registries when you want to deploy containers to production.

A registry is a collection of repositories, and a repository is a collection of
images—sort of like a GitHub repository, except the code is already built. An
account on a registry can create many repositories. The docker CLI uses Docker’s
public registry by default.

Note: We’ll be using Docker’s public registry here just because it’s free and
pre-configured, but there are many public ones to choose from, and you can even
set up your own private registry using Docker Trusted Registry.

### Log in with your Docker ID

If you don’t have a Docker account, sign up for one at
[cloud.docker.com](https://cloud.docker.com/). Make note of your username.

Log in to the Docker public registry on your local machine.

```bash
$ docker login
```

### Tag the image

The notation for associating a local image with a repository on a registry is
username/repository:tag. The tag is optional, but recommended, since it is the
mechanism that registries use to give Docker images a version. Give the
repository and tag meaningful names for the context, such as hgop:part2.

Now, put it all together to tag the image. Run docker tag image with your
username, repository, and tag names so that the image will upload to your
desired destination. The syntax of the command is:

```bash
docker tag image username/repository:tag
```

For example:

```bash
docker tag connect4 john/connect4-client:day2
```

Run docker images to see your newly tagged image. (You can also use docker image
ls.)

```bash
$ docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
connect4-client      latest              d9e555c53008        3 minutes ago       195MB
john/connect4-client day2                d9e555c53008        3 minutes ago       195MB
...
```

### Publish the image

Upload your tagged image to the repository:

```bash
docker push username/repository:tag
```

Once complete, the results of this upload are publicly available. If you log in
to Docker Hub, you will see the new image there, with its pull command.

Pull and run the image from the remote repository From now on, you can use
docker run and run your app on any machine with this command:

```bash
docker run -p 3000:3000 username/repository:tag
```

If the image isn’t available locally on the machine, Docker will pull it from
the repository. If you're working in a team try to pull images and run then from each other.

```bash
$ docker run -p 3000:3000 my-team-member/connect4-client:day2
Unable to find image 'my-team-member/connect4-client:day2' locally
part2: Pulling from my-team-member/connect4-client
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for my-team-member/connect4-client:day2
 * Running on http://0.0.0.0:3000/ (Press CTRL+C to quit)
```

Note: If you don’t specify the :tag portion of these commands, the tag of
:latest will be assumed, both when you build and when you run images. Docker
will use the last version of the image that ran without a tag specified (not
necessarily the most recent image). No matter where docker run executes, it
pulls your image, along with Node and runs your code. It all travels together in
a neat little package, and the host machine doesn’t have to install anything but
Docker to run it.

### How do I know I'm done?

- [ ] I've created a Dockerfile
- [ ] I've created a .dockerignore file
- [ ] I've created a docker image
- [ ] I ran an instance of my image (a docker container) and saw the results in
      a browser (curl)
- [ ] I've stored my docker image on Docker Cloud
- [ ] A team member has pulled my image and got it running
- [ ] I've committed all my code to my team's GitHub repo

## Part 4 - Questions

Create an `answers.md` file that describes the following
(only 1-2 sentences each)

```Dockerfile
# Docker Exercise
TODO: What was this assignment about

## What is Docker?
TODO: short description

## What is the difference between:
* Virtual Machine
* Docker Container
* Docker Image
TODO: short comparison

## What is the purpose of a package.json file:
TODO: short description

## When creating a Dockerfile why is it better to only copy the dependency files first, build the dependencies and then copy the rest of the application code? (hint: docker layering):
TODO: short description

## Results
TODO: What was accomplished in this exercise
```

## Handin

This is how your repositories should look after todays assignment which you
will submit on Friday.

connect4-client repository:

```bash
.
├── assignments
│   ├── day01
│   │   └── answers.md
│   └── day02
│       └── answers.md
├── docker-compose.yml
├── Justfile
├── README.md
├── scripts
│   └── verify_local_dev_environment.sh
└── src
    └── connect4-client    
        ├── .dockerignore
        ├── .eslintrc.json
        ├── .gitignore
        ├── .pnp.cjs
        ├── .pnp.loader.mjs
        ├── .yarnrc.yml
        ├── Dockerfile
        ├── next.config.js
        ├── next-env.d.ts
        ├── package.json
        ├── pages
        │   ├── api
        │   │   └── hello.ts
        │   ├── _app.tsx
        │   └── index.tsx
        ├── public
        │   ├── favicon.ico
        │   └── vercel.svg
        ├── README.md
        ├── styles
        │   ├── globals.css
        │   └── Home.module.css
        ├── tsconfig.json
        └── yarn.lock
```
