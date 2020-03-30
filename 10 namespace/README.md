# Namespaces

## Use of Namespaces

Namespaces are used to divide and order resources. If no namespace is specified, 'default' is used.

```console
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   18d
kube-node-lease   Active   18d
kube-public       Active   18d
kube-system       Active   18d
```

If you only want resources of a certain namespace you have to define this with the option '--namespace'.

```console
kubectl get pods --namespace==kube-system
```

To get the resources of all namespaces set the option '--namespace' to 'all'.

```console
kubectl get pods --all-namespaces
```

If you want to apply a command to a specific namespace you can append the option '--namespace'.

```console
$ kubectl create -f pod-definition.yml --namespace=dev
```

This information may also be declarative.

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
    namespace=dev
    labels:
        app: myapp
spec:
    containers:
        - name: nginx-container
          image: nginx
```

## Creating own namespaces

Imperative

```console
kubectl create namespace dev
```

Declarative

```yaml
apiVersion: v1
kind: Namespace
metadata:
    name: dev
```

If you want to work permanently in a different namespace without specifying the namespace for each command, you can switch your session to this namespace. 

```console
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

## Limits

A namespace is a space for resources. You can restrict this space to prevent overloading the cluster. To do this, an object of the type 'ResourceQuota' is created.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
    name: compute-quota
    namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```
