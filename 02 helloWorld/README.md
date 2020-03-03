# Hello World

## Start minikube

In the following a Hello World scenario with a simple Kubernetes cluster will be demonstrated and analyzed.

The first step is to start minikube itself.

```
$ minikube start
😄  minikube v1.7.3 on Linuxmint 19.3
✨  Using the virtualbox driver based on existing profile
⌛  Reconfiguring existing host ...
🔄  Starting existing virtualbox VM for "minikube" ...
🐳  Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
🚀  Launching Kubernetes ... 
🌟  Enabling addons: dashboard, default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube"
```

To ensure that all systems are working properly, the status can be queried.

```
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

## Start deployment and service

The cluster is initially empty. A deployment must be created and then published as a service.

```
$ kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
deployment.apps/hello-minikube created
```

To make the deployment accessible as a service.

```
$ kubectl expose deployment hello-minikube --type=NodePort --port=8080
service/hello-minikube exposed
```

## Observation and analysis of the resulting system

The content of the cluster can be retrieved with 'kubectl get <nodes/pods/services>'. Further details can be added with the option '-o wide'.

If the state of the resource changes you can wait for these changes with the option '-w'. Each change results in a new line with updated state.

```
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-797f975945-dpjzb   1/1     Running   0          4m36s

$ kubectl get pods -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
hello-minikube-797f975945-dpjzb   1/1     Running   0          4m43s   172.17.0.6   minikube   <none>           <none>

$ kubectl get pods -o wide -w
NAME                              READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
hello-minikube-797f975945-dpjzb   1/1     Running   0          4m48s   172.17.0.6   minikube   <none>           <none>
```

Get the URL of the published service. This URL can be opened in any browser.

```
$ minikube service hello-minikube --url
http://192.168.99.100:31309
```

One of the extensions of Kubernetes offers a web interface with all information about the existing cluster.

To be able to access this page a small HTTP server is started with the following command.

```
$ minikube dashboard
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:41567/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
Opening in existing browser session.
```

The 'kubectl watch' command is used to analyze the cluster in the console. The 'dump' option is used to output all known parameters.

```
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## Stop service and deployment

The test environment can be dismantled in reverse order with the command 'kubectl' and the option 'delete'.

```
$ kubectl delete service hello-minikube
service "hello-minikube" deleted

$ kubectl delete deployment hello-minikube
deployment.apps "hello-minikube" deleted
```

## Stop minikube

The last step is to shut down and delete the cluster.

```
$ minikube stop
✋  Stopping "minikube" in virtualbox ...
🛑  "minikube" stopped.

$ minikube delete
🔥  Deleting "minikube" in virtualbox ...
💀  Removed all traces of the "minikube" cluster.
```