# Exercise Working Session 10 - Container Probes

## Exercise 1

### Description

Define a new Pod named web-server with the image nginx:1.23.0 in a YAML manifest. Expose the container port 80. Do not create the Pod yet.

For the container, declare a startup probe of type httpGet. Verify that the kubelet can make a request to the root context endpoint. Use the default configuration for the probe.

For the container, declare a readiness probe of type httpGet.Verify that the kubelet can make a request to the root context endpoint. Wait five seconds before checking for the first time.

For the container, declare a liveness probe of type httpGet. Verify that the kubelet can make a request to the root context endpoint. Wait 10 seconds before checking for the first time. The probe should run the check every 30 seconds.

Create the Pod and follow the life cycle phases of the Pod during the process.

Inspect the runtime details of the Pod’s probes.


### Solution

Define a new Pod named web-server with the image nginx:1.23.0 in a YAML manifest. Expose the container port 80. Do not create the Pod yet.

For the container, declare a startup probe of type httpGet. Verify that the kubelet can make a request to the root context endpoint. Use the default configuration for the probe.

For the container, declare a readiness probe of type httpGet.Verify that the kubelet can make a request to the root context endpoint. Wait five seconds before checking for the first time.

For the container, declare a liveness probe of type httpGet. Verify that the kubelet can make a request to the root context endpoint. Wait 10 seconds before checking for the first time. The probe should run the check every 30 seconds.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server
  labels:
    app: web-server
spec:
  containers:
    - name: nginx
      image: nginx:1.23.0
      ports:
        - containerPort: 80
      startupProbe:
        httpGet:
          path: /
          port: 80
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 30
```

Create the Pod and follow the life cycle phases of the Pod during the process.

```bash
kubectl apply -f web-server.yaml
```
Inspect the runtime details of the Pod’s probes.

```bash
kubectl describe pod web-server
```
