# Controller

## Definition of a ReplicaSet

To make the application highly available, several instances of the application are started. A ReplicaSet then monitors the status of each individual instance and starts new ones if necessary.

Imperative

```
$ kubectl create deployment webapp --image=kodekloud/webapp-color
deployment.apps/webapp created
$ kubectl scale deployment webapp --replicas=3
deployment.apps/webapp scaled
```

Declarative

```
apiVersion: v1
kind: ReplicaSet
metadata:
    name: myapp-replicaset
    labels:
        app: myapp
        type: front-end
spec:
    template:
        metadata:
            name: myapp-pod
            labels:
                app: myapp
                type: front-end
        spec:
            containers:
                - name: nginx-container
                  image: nginx

replicas: 3

selector:
    matchLabels:
        type: front-end
```

To start the ReplicaSet use the command 'kubectl create' with the option '-f'.

```
$ kubectl create -f 02-replicaset-definition.yml 
replicaset.apps/myapp-replicaset created

$ kubectl get all
NAME                         READY   STATUS              RESTARTS   AGE
pod/myapp-replicaset-gnr4b   1/1     Running             0          10s
pod/myapp-replicaset-m6wkd   1/1     Running             0          10s
pod/myapp-replicaset-zhh5h   0/1     ContainerCreating   0          10s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   19h

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-replicaset   3         3         2       10s
```

The ReplicaSet is running and 3 predefined instances of the application are online.

To test the stability of the system you can now shut down a pod from the command line and see what happens.

```
$ kubectl delete pod myapp-replicaset-gnr4b
pod "myapp-replicaset-gnr4b" deleted

$ kubectl get all
NAME                         READY   STATUS              RESTARTS   AGE
pod/myapp-replicaset-m6wkd   1/1     Running             0          7m18s
pod/myapp-replicaset-rbzrj   0/1     ContainerCreating   0          3s
pod/myapp-replicaset-zhh5h   1/1     Running             0          7m18s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   19h

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-replicaset   3         3         2       7m18s
```

The ReplicaSet registers the failure and immediately starts a new Pod.

Even with too many Pods the ReplicaSet intervenes and corrects.

```
$ kubectl create -f 01-pod-definition.yml 
pod/myapp-pod created

$ kubectl get all
NAME                         READY   STATUS        RESTARTS   AGE
pod/myapp-pod                0/1     Terminating   0          1s
pod/myapp-replicaset-m6wkd   1/1     Running       0          80m
pod/myapp-replicaset-rbzrj   1/1     Running       0          72m
pod/myapp-replicaset-zhh5h   1/1     Running       0          80m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   20h

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-replicaset   3         3         3       80m
```

The new Pod will be recognized by the specified selector and terminated immediately.

Kubernetes is open source. The source code can be found [here](https://github.com/kubernetes/kubernetes).

The control of the Pod instances in a ReplicaSet can be read in the file 'kubernetes/pkg/controller/replicaset/replica_set.go'. 

```go
func (rsc *ReplicaSetController) manageReplicas(filteredPods []*v1.Pod, rs *apps.ReplicaSet) error {
	diff := len(filteredPods) - int(*(rs.Spec.Replicas))
	rsKey, err := controller.KeyFunc(rs)

	if err != nil {
		utilruntime.HandleError(fmt.Errorf("couldn't get key for %v %#v: %v", rsc.Kind, rs, err))
		return nil
	}

	if diff < 0 {
        // NOTE: Start additional instances
	} else if diff > 0 {
        // NOTE: Stop excess instances
	}

	return nil
}
```

The difference between the number of available and desired Pods is formed. Depending on whether this difference is smaller or larger than 0, Pods are then started or stopped.

To exit the system, the ReplicaSet itself is deleted.

```
$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/myapp-replicaset-m6wkd   1/1     Running   0          3h15m
pod/myapp-replicaset-rbzrj   1/1     Running   0          3h8m
pod/myapp-replicaset-zhh5h   1/1     Running   0          3h15m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   22h

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-replicaset   3         3         3       3h15m

$ kubectl delete replicaset myapp-replicaset
replicaset.apps "myapp-replicaset" deleted
```

## Imperative vs. Declarative

There are 2 possibilities to make changes to a cluster.

Imperative with a simple command from a console.

```
kubectl scale --replicas=6 -f replicaset-definition.yml
```

And declarative with a modified and reloaded Pod Definition file.

```
kubectl replace -f replicaset-definition.yml
```

Sending a command from the console is of course faster. However, the execution of this command cannot be tracked later.

Who changed what on the cluster and when? This question can no longer be answered. With declarative use, the Pod Definition file can be versioned.

A second problem arises when the imperative and declarative approach interfere. For example, when a changed deployment script accidentally reverses the effect of an imerperative operation.

A test system can be changed with imperative interventions. A production system should only be modified with declarative interventions!

If you want to be declarative, but have no configuration from the existing system, you can have this file created.

```
kubectl get pod myapp-pod -o yaml > output.yaml
```

To set up a new system declaratively, you can generate a kind of template. The option '--dry-run' ensures that nothing is created. The option '-o yaml' generates the output in the desired format.

```
kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml
```

## Update Strategies

> Recreate: All Pods are terminated simultaneously. Then all new Pods are started simultaneously. This procedure is the fastest. However, the availability of the deployment is interrupted between these two steps.
 
> Rolling: The Pods are terminated individually. After a Pod is closed, a new Pod with updated software is started in its place. In this way, functioning Pods are accessible at all times.

The individual commands for controlling the update cycle follow.

Create:

```
$ kubectl create -f deployment-definition.yml
```

Get:

```
$ kubectl get deployments
```

Update:

```
$ kubectl apply -f deployment-definition.yml
$ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
```

Update (Declarative, with generated yml File)

```
kubectl get pod myapp-pod -o yaml > output.yaml
```

Update (Imperative, without yml File)

```
kubectl edit pod redis
```

Status:

```
$ kubectl rollout status deployment/myapp-deployment
deployment "myapp-deployment" successfully rolled out
```

Rollback:

```
$ kubectl rollout undo deployment/myapp-deployment
```

## History

Without History.

```
$ kubectl create -f deployment-definition.yml
deployment.apps/myapp-deployment created

$ kubectl rollout history deployment/myapp-deployment
deployment.apps/myapp-deployment 
REVISION  CHANGE-CAUSE
1         <none>
```

With History.

```
$ kubectl create -f deployment-definition.yml --record --save-config
deployment.apps/myapp-deployment created

$ kubectl rollout history deployment/myapp-deployment
deployment.apps/myapp-deployment 
REVISION  CHANGE-CAUSE
1         kubectl create --filename=deployment-definition.yml --record=true --save-config=true
```

After 2 further updates (von 'nginx' nach 'nginx:1.12' und von 'replicas:3' nach 'replicas:6') the history is as follows.

```
$ kubectl apply -f deployment-definition.yml --record
deployment.apps/myapp-deployment configured

$ kubectl rollout history deployment/myapp-deployment
deployment.apps/myapp-deployment 
REVISION  CHANGE-CAUSE
1         kubectl create --filename=deployment-definition.yml --record=true --save-config=true
2         kubectl apply --filename=deployment-definition.yml --record=true
```
