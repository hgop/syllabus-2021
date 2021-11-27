# Week 1 Assignment

**Turn in your GitHub repository URLs into Canvas solo or in pairs (please add `kthorri` and `ironpeak` as collaborators)**

repository:

```text
.
├── .circleci
│   └── config.yml
├── assignments
│   ├── day01
│   │   └── answers.md
│   └── day02
│       └── answers.md
├── scripts
│   ├── ci
│   │   └── yaml
│   │       └── merge.sh
│   └── verify_local_dev_environment.sh
└── src
    ├── connect4-client
    │   ├── k8s
    │   │   ├── deployment.template.yaml
    │   │   ├── ingress.template.yaml
    │   │   └── service.template.yaml
    │   ├── public
    │   │   ├── ...
    │   │   └── index.html
    │   ├── src
    │   │   ├── ...
    │   │   ├── index.tsx
    │   │   └── App.tsx
    │   ├── .dockerignore
    │   ├── .gitignore
    │   ├── Dockerfile
    │   ├── tsconfig.json
    │   ├── package-lock.json
    │   └── package.json
    └── httpbin
        └── k8s
            ├── deployment.template.yaml
            ├── ingress.template.yaml
            └── service.template.yaml
```

Add a `./README.md` file, it should include the URL to the instance running
the game client `connect4.{{team-name}}.hgopteam.com`.

If you did anything extra, you should list it up in the `./README.md` if you
want us to take it into account.
