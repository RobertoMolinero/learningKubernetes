# Services

## For a single Pod

There are 3 different port specifications within the service.

| Name          | Description                      |
| ------------- | -------------------------------- |
| targetPort    | Port of the Pod to be connected  |
| port          | Port of the service              |
| nodePort      | External port of the node        |

These 3 ports are defined in a separate file.

Which Pod this definition refers to is determined by the selector with values from the Pod.

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

Afterwards, the installed server can be accessed via the internal IP of the node.

```
$ kubectl create -f pod-definition.yml 
pod/myapp-pod created

$ kubectl create -f service-definition.yml
service/myapp-service created

$ kubectl get nodes -o wide
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE              KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready    master   44h   v1.17.3   192.168.99.101   <none>        Buildroot 2019.02.9   4.19.94          docker://19.3.6

$ curl http://192.168.99.101:30008
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

# For multiple containers

For a modern web application the different components are booted as separate containers.

```
docker run -d --name=redis redis
docker run -d --name=db postgresql:9.4
docker run -d --name=vote -p 5000:80 voting-app
docker run -d --name=result -p 5001:80 result-app
docker run -d --name=worker worker
```

All components function perfectly. However, they do not know about each other and cannot communicate with each other.

To get a stack of connected components, the links must be specified with the '--link' option.

```
docker run -d --name=redis redis
docker run -d --name=db postgresql:9.4
docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
docker run -d --name=result -p 5001:80 --link db:db result-app
docker run -d --name=worker --link redis:redis --link db:db worker
```

# Ingress

An API object that manages external access to the services in a cluster, typically HTTP.

Ingress may provide load balancing, SSL termination and name-based virtual hosting.

[Kubernetes Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)

# Network Policies

...

```
$ kubectl get networkpolicy
NAME             POD-SELECTOR   AGE
payroll-policy   name=payroll   5m6s
```

...

```
$ kubectl describe networkpolicy/payroll-policy
Name:         payroll-policy
Namespace:    default
Created on:   2020-03-27 14:21:59 +0000 UTC
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"networking.k8s.io/v1","kind":"NetworkPolicy","metadata":{"annotations":{},"name":"payroll-policy","namespace":"default"},"s...
Spec:
  PodSelector:     name=payroll
  Allowing ingress traffic:
    To Port: 8080/TCP
    From:
      PodSelector: name=internal
  Allowing egress traffic:
    <none> (Selected pods are isolated for egress connectivity)
  Policy Types: Ingress
```