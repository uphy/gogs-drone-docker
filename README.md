## Introduction

Start CI quickly with docker-compose.

"gogs" for Git repository.
"Drone" for CI tool.
"Docker Registry(Distribution)" for Docker registry.

## How to use

### Start services

```bash
$ git clone https://github.com/uphy/gogs-drone-docker.git
$ cd gogs-drone-docker
$ cat << EOF > .env
> DOMAIN_NAME=YOUR_DOMAIN_OR_IP
> ADMIN_USERS=YOUR_USERNAME
> EOF
$ docker-compose up -d
```

### Setup gogs

Open http://YOUR_DOMAIN_OR_IP:10080/

Database Type: SQLite3
Domain: YOUR_DOMAIN_OR_IP
SSH Port: <Blank>
Application URL: http://YOUR_DOMAIN_OR_IP:10080/
Admin Account Settings: <Register admin user with "YOUR_USERNAME">

Click "Install gogs".

### Create your first repository

Click the "+" button and select "New Repository".
Input the name of the repository and click the "Create Repository" button.

I develop a simple docker image here.

```bash
$ git clone http://YOUR_DOMAIN_OR_IP:10080/USERNAME/demo
warning: You appear to have cloned an empty repository.
$ cd demo
$ cat << EOF > Dockerfile
FROM bash

CMD echo hello world
EOF
```

Also, create .drone.yml for build configuration.

```bash
$ cat << EOF > .drone.yml
workspace:
  base: /build
  path: app

pipeline:
  restore-cache:
    image: drillster/drone-volume-cache
    restore: true
    mount:
      - /build/docker
    volumes:
      - /tmp/drone/cache:/cache

  publish:
    image: plugins/docker
    repo: YOUR_DOMAIN_OR_IP:5000/YOUR_USERNAME/demo
    registry: YOUR_DOMAIN_OR_IP:5000
    tags: [ "latest" ]
    dockerfile: Dockerfile
    insecure: true
    storage_path: /build/docker

  rebuild-cache:
    image: drillster/drone-volume-cache
    rebuild: true
    mount:
      - /build/docker
    volumes:
      - /tmp/drone/cache:/cache

  run:
    image: docker
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    commands:
      - docker run -i --rm YOUR_DOMAIN_OR_IP:5000/YOUR_USERNAME/demo
EOF
```

Commit them.

```bash
$ git add .
$ git commit -m "Initial commit."
```

## Setup webhook

Login to drone with the gogs admin account(YOUR_USERNAME).
http://YOUR_DOMAIN_OR_IP:8000/

You can see the created Git repository here.
Activate the repository by clicking the toggle switch.
This operation add webhook to your repository in gogs.
You can confirm it in gogs's "Settings" screen.

## Setup drone

You need to setup "Project Settings" of Drone to enable caching of Docker build.

Click the project in Drone.
Click the hamburger menu and select "Settings".
Click the "Trusted" checkbox in the Project settings.

## Push

Push your commit to the gogs repository.

```bash
$ git push -u origin master
```

