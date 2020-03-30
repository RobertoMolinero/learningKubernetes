# K8s Objects

## Definition of a Pod

The following attributes are the minimum that must be present in each configuration file.

```yaml
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

```console
kubectl create -f pod-definition.yml
```

To create the pod without a configuration file 'kubectl run' is used.

```console
kubectl run --generator=run-pod/v1 redis --image=redis
```

In order to really work with a Pod, its ports must be unblocked. This is done with an object of type 'service'.

```console
kubectl run --generator=run-pod/v1 redis --image=redis
kubectl expose pod redis --port=6379 --name redis-service
```

## From Docker to Kubernetes

### Docker build & run

Image

```dockerfile
FROM ubuntu:18.04
CMD ["echo", "Hello from the Docker-Image"]
```

Build & Run

```console
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

```dockerfile
FROM ubuntu:18.04
ENTRYPOINT ["echo"]
```

Build & Run

```console
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

```yaml
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

## Workloads

The previous Pods are applications that are permanently available once they are started.

But there are use cases where an application should only fulfill a certain task. If this task is finished, the application can also terminate.

### Job

A job is a task that is started manually, fulfils a defined task and is then completed.

```dockerfile
docker run ubuntu expr 3 + 2 
```

This can also be achieved with a Pod. But the Pod would be started again and again, because Kubernetes assumes that a Pod should always be available.

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: math-pod
spec:
  containers:
    - name: math-add
      image: ubuntu
      command: ['express', '3', '+', '2']
```

The restart can be prevented with the option 'restartPolicy'. 

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: math-pod
spec:
  containers:
    - name: math-add
      image: ubuntu
      command: ['express', '3', '+', '2']
  restartPolicy: Never
```

The correct type for the object is 'Job'. Here you can also define the number of parallel running tasks and the number of tasks to be completed. 

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: math-add-job

spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '+', '2']
      restartPolicy: Never
```

The object is controlled via the usual commands 'create', 'logs' and 'delete'. 

```console
kubectl create -f job-definition.yml

kubectl get jobs
kubectl get pods

kubectl logs math-add-job-.....

kubectl delete job math-add-job
```

### CronJob

Jobs that should run at certain times or in certain intervals are described with 'CronJobs'.

```yaml
apiVersion: batch/v1beta1
kind: CronJob

metadata:
  name: reporting-cron-job

spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      template:
        spec:
          containers:
            - name: math-add
              image: ubuntu
          restartPolicy: Never
```

For the time specification under 'schedule' the time is given in cron format.

[Wikipedia Cron](https://en.wikipedia.org/wiki/Cron)
