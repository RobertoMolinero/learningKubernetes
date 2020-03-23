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

## ConfigMap

Imperative

```
kubectl create configmap \
    app-config --from-literal=APP_COLOR=blue \
               --from-literal=APP_MOD=prod
```

Imperative with config file

```
kubectl create configmap \
    app-config --from-file=app_config.properties
```

Declarative

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

```
kubectl create -f config-map.yaml
```

### View

To list the current ConfigMaps you can use the command 'kubectl get configmap'. 

Important: The command 'kubectl get all' does not work here!

With the option 'describe' you can display the content of the configmap. 

```
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   20d

$ kubectl get configmap
NAME         DATA   AGE
app-config   1      18m

$ kubectl describe configmap/app-config
Name:         app-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
APP_COLOR:
----
blue
Events:  <none>
```
