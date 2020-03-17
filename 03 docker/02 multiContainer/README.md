# Application with multiple containers

## Configuration

Most applications consist of several independent subsystems. 

It makes sense to build these subsystems into independent containers. This way you can scale them later independently of each other.

The tool 'docker-compose' with a yaml configuration file is used for this.

```
version: '3'

services:
  redis-server:
    image: 'redis'
  node-app:
    restart: always
    build: .
    image: 'visits'
    ports:
      - "4082:8082"
```

## Start and Stop

The network is started and stopped via 'docker-compose up' or 'docker-compose down'.

Optionally, the images can be rebuilt before starting with the '--build' option.

```
$ docker-compose up --build -d
Building node-app
Step 1/6 : FROM node:alpine
 ---> b01d82bd42de
Step 2/6 : WORKDIR '/app'
 ---> Using cache
 ---> f023b35af5a9
Step 3/6 : COPY package.json .
 ---> Using cache
 ---> 550368a5b509
Step 4/6 : RUN npm install
 ---> Using cache
 ---> 74dd19ca73df
Step 5/6 : COPY . .
 ---> Using cache
 ---> 747fce1f38e0
Step 6/6 : CMD ["npm", "start"]
 ---> Using cache
 ---> c0ad95708f8c
Successfully built c0ad95708f8c
Successfully tagged visits:latest
Starting 02multicontainer_redis-server_1 ... done
Starting 02multicontainer_node-app_1     ... done

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
c3aca05c7a62        visits              "docker-entrypoint.s…"   27 seconds ago      Up 3 seconds        0.0.0.0:4082->8082/tcp   02multicontainer_node-app_1
3aa744601eb2        redis               "docker-entrypoint.s…"   12 minutes ago      Up 4 seconds        6379/tcp                 02multicontainer_redis-server_1

$ docker-compose down
Stopping 02multicontainer_node-app_1     ... done
Stopping 02multicontainer_redis-server_1 ... done
Removing 02multicontainer_node-app_1     ... done
Removing 02multicontainer_redis-server_1 ... done
Removing network 02multicontainer_default

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

## Restart Policy

If something goes wrong in the created network and a container terminates, 'docker-compose' can automatically restart this container.

There are several strategies for this.

| Concept | Explanation | Example application |
| --- | --- | --- |
| no | Never restart. | Worker |
| always | Always restart. | Web Server |
| on-failure | Only restart if the return value of the container contains an error. | Batch Job |
| unless-stopped | Always restart until the container is stopped manually. | ? |
