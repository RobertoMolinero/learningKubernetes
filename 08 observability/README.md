# Observability

...

## Probes

There are 3 different checks to determine whether a container is available.

The internal node management tool 'kubelet' uses all 3 checks, depending on the configuration, to determine which containers are available.

The containers that are in a bad state will be restarted or at least logged off from the load balancer, depending on the configuration.

### Liveness

Liveness probes prüfen ob ein Container verfügbar ist.

Liveness probe via a command from the console

```yaml
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      livenessProbe:
        exec:
          command:
            - cat
            - /app/is_ready
```

Liveness probe via TCP connection

```yaml
spec:
  containers:
    - name: redis-db
      image: redis
      livenessProbe:
        tcpSocket:
          port: 3306
```

Liveness probe via HTTP connection

```yaml
spec:
  containers:
    - name: simple-webapp
      image: nginx
      ports:
        - containerPort: 8080
      livenessProbe:
        httpGet:
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 8
```

### Readiness

Readiness probes prüfen ab wann ein gestarteter Container bereit ist Daten entgegenzunehmen. Dabei bedeutet bereit das alle enthaltenen Container bereit sind.

Readiness probe via a command from the console

```yaml
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      readinessProbe:
        exec:
          command:
            - cat
            - /app/is_ready
```

Readiness probe via TCP connection

```yaml
spec:
  containers:
    - name: redis-db
      image: redis
      readinessProbe:
        tcpSocket:
          port: 3306
```

Readiness probe via HTTP connection

```yaml
spec:
  containers:
    - name: simple-webapp
      image: nginx
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 8
```

### Startup

Startup probes check if a container has been started. If the check is configured, the above mentioned are deactivated. 

## Logging

...

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: event-simulator-pod

spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
    - name: image-processor
      image: some-image-processor
```

...

```console
kubectl logs -f event-simulator-pod event-simulator
```

...

```console
kubectl exec -it pod/webapp -- cat /log/app.log
```

## Monitoring

...

### Metrics Server

...

```console
minikube addons enable metrics-server
```
