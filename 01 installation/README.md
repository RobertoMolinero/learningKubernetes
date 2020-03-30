# Installation Instructions

A current instruction for the installation of all required components can be found under this link:

https://kubernetes.io/docs/tasks/tools/install-minikube/

The following components must be installed:

* a container software like for example Docker
* kubectl
* a hypervisor such as VirtualBox
* minikube

## Docker Install

Kubernetes orchestrates container applications. And the most commonly used containers are docker containers.

Therefore Docker should be installed first. Afterwards, two small sample programs will be used to test the correct setup.

```console
$ su -s roberto

$ sudo apt-get update
$ sudo apt-get upgrade

$ sudo apt install docker.io

$ sudo groupadd docker
$ sudo usermod -aG docker roberto
```

The first test program is a variation of the 'Hello World' program.

The container displays the text 'Hello from Docker!' and some additional information about the program's structure.

```console
$ docker run hello-world

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

A minimally more complex example, in which the output message can be manipulated via a transferred parameter.

```console
$ docker run docker/whalesay cowsay Hello World!
 ______________ 
< Hello World! >
 -------------- 
    \
     \
      \     
                    ##        .            
              ## ## ##       ==            
           ## ## ## ##      ===            
       /""""""""""""""""___/ ===        
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
       \______ o          __/            
        \    \        __/             
          \____\______/
```
