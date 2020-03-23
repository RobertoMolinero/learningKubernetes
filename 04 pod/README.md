# Pod

## Definition of a Pod

The following attributes are the minimum that must be present in each configuration file.

```
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
    labels:
        app: myapp
spec:
    containers:
        - name: nginx-container
          image: nginx
```

To use this configuration file the command 'kubectl create' with the option '-f' is used.

```
kubectl create -f pod-definition.yml
```

To create the pod without a configuration file 'kubectl run' is used.

```
kubectl run --generator=run-pod/v1 redis --image=redis
```

In order to really work with a Pod, its ports must be unblocked. This is done with an object of type 'service'.

```
kubectl run --generator=run-pod/v1 redis --image=redis
kubectl expose pod redis --port=6379 --name redis-service
```

## From Docker to Kubernetes

### Docker build & run

Image

```
FROM ubuntu:18.04
CMD ["echo", "Hello from the Docker-Image"]
```

Build & Run

```
$ docker build -t ubuntu-echo .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM ubuntu:18.04
 ---> 4e5021d210f6
Step 2/2 : CMD ["echo", "Hello from the Docker-Image"]
 ---> Using cache
 ---> 8eefaee0b7ad
Successfully built 8eefaee0b7ad
Successfully tagged ubuntu-echo:latest

$ docker run ubuntu-echo
Hello from the Docker-Image
```

### Docker build & run (with arguments)

Image

```
FROM ubuntu:18.04
ENTRYPOINT ["echo"]
```

Build & Run

```
$ docker build -t ubuntu-echo -f Dockerfile.arg .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM ubuntu:18.04
 ---> 4e5021d210f6
Step 2/2 : ENTRYPOINT ["echo"]
 ---> Using cache
 ---> c0fe340a9088
Successfully built c0fe340a9088
Successfully tagged ubuntu-echo:latest

$ docker run ubuntu-echo "Hello from the Docker-Image (with args)"
Hello from the Docker-Image (with args)
```

### Kubernetes create -f (with arguments)

Start the last image inside as a Pod. The argument "Hello!" is passed.

```
apiVersion: v1
kind: Pod

metadata:
    name: ubuntu-echo-pod

spec:
  containers:
    - name: ubuntu-echo
      image: ubuntu-echo
      args: ["Hello!"]
```
