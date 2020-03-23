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
