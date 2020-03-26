# Configuration

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

## Secret

Imperative

```
kubectl create secret generic \
    app-secret --from-literal=DB_Host=mysql \
    app-secret --from-literal=DB_User=root \
    app-secret --from-literal=DB_Password=secret
```

Imperative with config file

```
kubectl create secret generic \
    app-secret --from-file=app_secret.properties
```

Declarative

```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql
  DB_User: root
  DB_Password: secret
```

To generate a hash value from any string you can use the following command in the console.

```
$ echo -n 'secret' | base64
c2VjcmV0
```

### View

To list the current Secrets you can use the command 'kubectl get secret'. 

Important: The command 'kubectl get all' does not work here!

```
$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
app-secret            Opaque                                3      2m32s
default-token-95g78   kubernetes.io/service-account-token   3      20d
```

Details about a specific object are requested as usual with describe.

```
$ kubectl describe secret app-secret
Name:         app-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
DB_Host:      5 bytes
DB_Password:  6 bytes
DB_User:      4 bytes
```

## Service Account

A Service Account is an account for a technical process.

```
kubectl create serviceaccount dashboard-sa
```

To list the current Service Accounts you can use the command 'kubectl get serviceaccounts'. 

Important: The command 'kubectl get all' does not work here!

```
kubectl get serviceaccounts
```

Details of a single service account.

```
kubectl describe serviceaccount dashboard-sa
```

The information that defines the service account is located in the object under the folder '/var/run/secrets/kubernetes.io/serviceaccount'. 

```
kubectl exec -it my-kubernetes-dashboard ls /var/run/secrets/kubernetes.io/serviceaccount
```

To read the token from the service account. This is useful to test the token manually.

```
kubectl exec -it my-kubernetes-dashboard cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

Other possibility:

```
kubectl describe secret dashboard-sa
```

## Resources

Objects can use the attribute 'requests' to specify how much memory and CPU they are expected to use during execution.

With the attribute 'limits' the maximum value is set. If one of the specified limits is exceeded during execution, Kubernetes will terminate this Pod.

```
apiVersion: v1
kind: Pod

metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color

spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "1Gi"
          cpu: 1
        limits:
          memory: "2Gi"
          cpu: 2
```

It is also possible to create a limit object individually and assign it to another object later.

```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
    - default:
        cpu: 1
      defaultRequest:
        cpu: 0.5
      type: Container
```

## Taints & Tolerations

The allocation of resources can be controlled with a kind of markers. For example, you can assign a marker (or tag) to a node in the form of a key-value pair.

```
kubectl taint nodes node-name key=value:taint-effect
```

In this example the node 'node1' is marked with the key 'app' and the value 'blue'.

```
kubectl taint nodes node1 app=blue:NoSchedule
```

The final keyword, in this case 'NoSchedule', indicates how this marker should be used when allocating resources. The following specifications are possible.

| taint-effect      | Description                                                              |
| ----------------- |--------------------------------------------------------------------------|
| NoSchedule        | Pod will not be scheduled on the node                                    |
| PreferNoSchedule  | Pod will not be scheduled on the node (not guaranteed!)                  |
| NoExecute         | Pod will not be scheduled on the node and existing pods will be evicted  |

To remove a set marker, simply repeat the command with an appended '-'.

```
kubectl taint nodes node1 app=blue:NoSchedule-
```

The taints of a specific node can be viewed with 'describe'. Since the 'describe' of a node can be very detailed, it is best to filter the result with 'grep'.

```
kubectl describe node kubemaster | grep Taint
```

The resource to be distributed to the nodes now has a list of tolerations indicating which markers should be used.

```
apiVersion: v1
kind: Pod

metadata:
    name: myapp-pod

spec:
  containers:
    - name: nginx-container
      image: nginx
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

## Node Affinity

With node affinities more complex rules for assignment are possible.

An example: The own nodes get the key 'size' and the values 'small', 'medium' and 'large' as markers. Then you can define that a Pod can run on all nodes that are not marked as 'small'.

```
apiVersion: v1
kind: Pod

metadata:
  name: myapp-pod

spec:
  containers:
    - name: data-processor
      image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: NotIn
                values:
                  - Small
```

The following operators can be used for evaluation: In, NotIn, Exists, DoesNotExist, Gt and Lt.

## Labels & Selectors

The constructs 'Labels' and 'Selectors' are there for easy assignment. Each resource can get a label.

```
kubectl label nodes node-1 size=Large
```

Another resource then refers by means of a 'selector' to all resources that have this label with the specified value.

This is often used for 'ReplicaSets' which are supposed to keep a lot of Pods running. Since there may be more than the own Pods in the namespace, they are distinguished by their labels.  

```
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-End
  annotations:
    buildversion: 1.34

spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-End
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```
