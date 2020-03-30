# Persistence

Data stored on hard disk is normally volatile. When a Pod is closed, that data is gone. Whether the Pod terminates normally or crashes is irrelevant. 

When multiple Pods work together, for example in a pattern, they often need to exchange data. Again, a robust memory is required.

## Persistent Volume

A PersistentVolume (PV) is a piece of storage in the cluster.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

The attribute 'hostPath' means that a local storage is created on the system of the node. While this is still okay for test environments, you should not do this for production environments. Firstly, because deployments across multiple nodes would no longer have the same data. On the other hand, complete nodes can also fail.

As an alternative, it is a good idea to use a cloud service provider to store data. For example AWS EBS:

```yaml
  awsElasticBlockStore:
    volumeID: volume-id
    fsType: ext4
```

## Persistent Volume Claim

A PersistentVolumeClaim (PVC) is a request for storage by a user.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

The allocation to a real existing storage is done automatically by Kubernetes. The column 'Volume' shows to which memory the claim is assigned.

```console
$ kubectl get persistentvolumeclaims
NAME            STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
task-pv-claim   Bound    task-pv-volume   10Gi       RWO            manual         5h22m
```

You can see whether a storage currently has assignments to claims in the 'Status' column. If there are no assignments, the value 'Available' is displayed.

```console
$ kubectl get persistentvolumes
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
task-pv-volume   10Gi       RWO            Retain           Bound    default/task-pv-claim   manual                  5h31m
```

## Example

The example shown here is from the official Kubernetes documentation:

[Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

First, the storage location is created on the node. To do this, connect to the node with 'minikube ssh'. The following commands create an example file.

```console
sudo mkdir /mnt/data
sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
```

Afterwards a volume is defined which refers to this folder. And a demand for storage space is defined.

Then a Pod is generated that uses this claim in its definition.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storageded
```

To check the result, switch to the Pod.

```console
kubectl exec -it task-pv-pod -- /bin/bash
```

The 'curl' command to be used must first be installed.

```console
apt update
apt install curl
curl http://localhost/
```

If everything went well, the output looks like this.

```console
root@task-pv-pod:/# curl http://localhost/
Hello from Kubernetes storage
```

If everything went well and you have analyzed the result sufficiently, you can delete all resources again. The order is not unimportant.

As long as there is a claim with a valid allocation to a memory, the memory cannot be deleted.

```console
kubectl delete pod task-pv-pod
kubectl delete pvc task-pv-claim
kubectl delete pv task-pv-volume
```

And the test data can also be deleted again at the end.

```console
sudo rm /mnt/data/index.html
sudo rmdir /mnt/data
```
