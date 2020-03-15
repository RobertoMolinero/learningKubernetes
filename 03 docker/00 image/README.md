# Building an own Dockerfile

To build an image from a docker file the option 'build' is used.

```
$ docker build .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM alpine
latest: Pulling from library/alpine
c9b1b535fdd9: Pull complete 
Digest: sha256:ab00606a42621fb68f2ed6ad3c88be54397f981a7b70a79db3d1172b11c4367d
Status: Downloaded newer image for alpine:latest
 ---> e7d92cdc71fe
Step 2/3 : RUN apk add --update redis
 ---> Running in 8611c43f8f2e
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/community/x86_64/APKINDEX.tar.gz
(1/1) Installing redis (5.0.7-r0)
Executing redis-5.0.7-r0.pre-install
Executing redis-5.0.7-r0.post-install
Executing busybox-1.31.1-r9.trigger
OK: 7 MiB in 15 packages
Removing intermediate container 8611c43f8f2e
 ---> 36c5bb3cd839
Step 3/3 : CMD ["redis-server"]
 ---> Running in 45f39e571801
Removing intermediate container 45f39e571801
 ---> 160838094f07
Successfully built 160838094f07
```

Afterwards the created image can be started via the output Id.

```
$ docker run 160838094f07
...
```

Names can be assigned to keep track of multiple images.

```
$ docker build -t robertomolinero/redis:latest .
```

To get the individual commands for a docker file, start the base image and switch to the running container with 'sh'.

Then execute all installation commands and check that everything is running correctly. The result of enough experiments is a docker file that automates this installation process in the future.

```
$ docker run -it alpine sh
/ # apk add --update redis
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/community/x86_64/APKINDEX.tar.gz
(1/1) Installing redis (5.0.7-r0)
Executing redis-5.0.7-r0.pre-install
Executing redis-5.0.7-r0.post-install
Executing busybox-1.31.1-r9.trigger
OK: 7 MiB in 15 packages
/ # 
```

If you want to reuse the created container you can make a snapshot of it in a second terminal.

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
07a7e86bf4a4        alpine              "sh"                About a minute ago   Up About a minute                       confident_meninsky

$ docker commit -c 'CMD ["redis-server"]' 07a7e86bf4a4
sha256:f262c6f75976ece533a9d9e3b3a9323b2e997c0551f9ae6a647ee108197123dd

$ docker run f262c6f75976e
1:C 08 Mar 2020 19:23:39.449 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 08 Mar 2020 19:23:39.449 # Redis version=5.0.7, bits=64, commit=bed89672, modified=0, pid=1, just started
1:C 08 Mar 2020 19:23:39.449 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1:M 08 Mar 2020 19:23:39.451 * Running mode=standalone, port=6379.
1:M 08 Mar 2020 19:23:39.451 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 08 Mar 2020 19:23:39.451 # Server initialized
1:M 08 Mar 2020 19:23:39.451 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 08 Mar 2020 19:23:39.451 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 08 Mar 2020 19:23:39.451 * Ready to accept connections
```
