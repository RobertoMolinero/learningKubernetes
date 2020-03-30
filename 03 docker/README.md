# Docker

## Installation

I have already covered the installation of Docker in chapter '01 installation'. To build your own Docker applications you need the tool 'docker compose'.

The installation is currently documented here:

[Installation docker-compose](https://docs.docker.com/compose/install/)

```console
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   617  100   617    0     0   1377      0 --:--:-- --:--:-- --:--:--  1374
100 16.3M  100 16.3M    0     0  1187k      0  0:00:14  0:00:14 --:--:-- 1435k

$ sudo chmod +x /usr/local/bin/docker-compose
```

The command 'run' used so far is a combination of the commands 'create' and 'start'.

So you could also start the 'Hello World' from chapter '02 helloWorld' as follows.

```console
$ docker create hello-world
430a4f3e2d67a93dc465f7b745901dd582e2580a404bc2726fcaa7329a3ea6b8

$ docker start -a 430a4f3e2d67a93dc465f7b745901dd582e2580a404bc2726fcaa7329a3ea6b8

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## Local memory
   
The longer you use Docker the more images accumulate in the local memory. Normally 'docker ps' only shows the currently running containers. To get a list of all containers ever created you also need the option '--all'.

```console
$ docker ps --all
CONTAINER ID        IMAGE               COMMAND                 CREATED             STATUS                         PORTS               NAMES
554393ae15a6        busybox             "echo hi there"         7 seconds ago       Exited (0) 7 seconds ago                           competent_northcutt
430a4f3e2d67        hello-world         "/hello"                5 minutes ago       Exited (0) 4 minutes ago                           relaxed_tesla
9b66f22cb8e5        busybox             "ping google.com"       35 minutes ago      Exited (0) 35 minutes ago                          lucid_brown
...
```

This memory can be released again.

```console
$ docker system prune
WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - all dangling build cache
Are you sure you want to continue? [y/N] y
Deleted Containers:
554393ae15a6e2268e35be0c4a3002e3877e84a2c9691f87f71d9edd1aa346ba
430a4f3e2d67a93dc465f7b745901dd582e2580a404bc2726fcaa7329a3ea6b8
9b66f22cb8e59a04e35eec13877ba828c3da60410b124d355871fa5f1e129802
...

Total reclaimed space: 0B

$ docker ps --all
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

## Container Logs

The log outputs of a container can be queried with the option 'logs'.

```console
$ docker create busybox echo hi there
77d6ecde2864b171d89dbed290df5da71ed7b9b4132da1fd723d906afa36667d

$ docker start 77d6ecde2864b171d89dbed290df5da71ed7b9b4132da1fd723d906afa36667d
77d6ecde2864b171d89dbed290df5da71ed7b9b4132da1fd723d906afa36667d

$ docker logs 77d6ecde2864b171d89dbed290df5da71ed7b9b4132da1fd723d906afa36667d
hi there

$ docker start -a 77d6ecde2864b171d89dbed290df5da71ed7b9b4132da1fd723d906afa36667d
hi there

$ docker logs 77d6ecde2864b171d89dbed290df5da71ed7b9b4132da1fd723d906afa36667d
hi there
hi there

$ docker start 77d6ecde2864b171d89dbed290df5da71ed7b9b4132da1fd723d906afa36667d
77d6ecde2864b171d89dbed290df5da71ed7b9b4132da1fd723d906afa36667d

$ docker logs 77d6ecde2864b171d89dbed290df5da71ed7b9b4132da1fd723d906afa36667d
hi there
hi there
hi there
```

## Stop container
   
To stop a container there are 2 possibilities. In the first option, you can stop the container by using 'stop'. The container executes all commands that are intended for an orderly termination and then stops.

```console
$ docker create busybox ping google.com
624409fe10c47444e40390095d0da9fb5f31f60071d485358ce760c517c95fa9

$ docker start 624409fe10c47444e40390095d0da9fb5f31f60071d485358ce760c517c95fa9
624409fe10c47444e40390095d0da9fb5f31f60071d485358ce760c517c95fa9

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
624409fe10c4        busybox             "ping google.com"   16 seconds ago      Up 1 second                             nervous_jennings

$ docker logs 624409fe10c4
PING google.com (172.217.19.78): 56 data bytes
64 bytes from 172.217.19.78: seq=0 ttl=51 time=24.158 ms
64 bytes from 172.217.19.78: seq=1 ttl=51 time=24.784 ms
64 bytes from 172.217.19.78: seq=2 ttl=51 time=25.203 ms

$ docker logs 624409fe10c4
PING google.com (172.217.19.78): 56 data bytes
64 bytes from 172.217.19.78: seq=0 ttl=51 time=24.158 ms
64 bytes from 172.217.19.78: seq=1 ttl=51 time=24.784 ms
64 bytes from 172.217.19.78: seq=2 ttl=51 time=25.203 ms

$ docker stop 624409fe10c4
624409fe10c4
```

The other possibility is a hard abort with the 'kill' option. Nothing is cleaned up here.

```console
$ docker create busybox ping google.com
ae5543d37102976ac6e855584eb05b4084b4fd43f55f3cbbf798a9214e2bf647

$ docker start ae5543d37102976ac6e855584eb05b4084b4fd43f55f3cbbf798a9214e2bf647
ae5543d37102976ac6e855584eb05b4084b4fd43f55f3cbbf798a9214e2bf647

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
ae5543d37102        busybox             "ping google.com"   17 seconds ago      Up 1 second                             determined_rubin

$ docker logs ae5543d37102976ac6e855584eb05b4084b4fd43f55f3cbbf798a9214e2bf647
PING google.com (172.217.22.46): 56 data bytes
64 bytes from 172.217.22.46: seq=0 ttl=55 time=14.621 ms
64 bytes from 172.217.22.46: seq=1 ttl=55 time=16.058 ms
64 bytes from 172.217.22.46: seq=2 ttl=55 time=20.497 ms

$ docker logs ae5543d37102976ac6e855584eb05b4084b4fd43f55f3cbbf798a9214e2bf647
PING google.com (172.217.22.46): 56 data bytes
64 bytes from 172.217.22.46: seq=0 ttl=55 time=14.621 ms
64 bytes from 172.217.22.46: seq=1 ttl=55 time=16.058 ms
64 bytes from 172.217.22.46: seq=2 ttl=55 time=20.497 ms

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
ae5543d37102        busybox             "ping google.com"   40 seconds ago      Up 24 seconds                           determined_rubin

$ docker kill ae5543d37102
ae5543d37102
```

## Connecting to running containers
   
On a running container you can connect to a shell directly at startup or afterwards. With the commands of the shell you can then take a closer look at the structure of the container.

```console
$ docker run -it busybox sh
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ #
```

To connect to an already running container its container Id is used as parameter.

```console
$ docker run redis
...

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
c8a2ff40a46c        redis               "docker-entrypoint.s…"   26 minutes ago      Up 25 minutes       6379/tcp            vigorous_ptolemy

$ docker exec -it c8a2ff40a46c redis-cli
127.0.0.1:6379> set myvalue 5
OK
127.0.0.1:6379> get myvalue
"5"
127.0.0.1:6379> del myvalue
(integer) 1
127.0.0.1:6379> get myvalue
(nil)
127.0.0.1:6379>
```
