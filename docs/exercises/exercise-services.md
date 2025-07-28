# Exercise Working Session 12 - Services

## Exercise 1

### Description

Create a Service named myapp of type ClusterIP that exposes port 80 and maps to the target port 80.

Create a Deployment named myapp that creates 1 replica running the image nginx:1.23.4-alpine. Expose the container port 80. Scale the Deployment to 2 replicas.

Create a temporary Pod using the image busybox:1.36.1 and execute a wget command against the IP of the service.

Change the service type to NodePort so that the Pods can be reached from outside of the cluster. Execute a wget command against the service from outside of the cluster.


### Solution

Create a Service named `myapp` of type `ClusterIP` that exposes port 80 and maps to the target port 80:

`myapp-service.yaml` file:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: ckad
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: myapp
```

Create a Deployment named `myapp` that creates 1 replica running the image `nginx:1.23.4-alpine`. Expose the container port 80:

`myapp-deployment.yaml` file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: ckad
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.4-alpine
        ports:
        - containerPort: 80
```

Scale the Deployment to 2 replicas:

```bash
kubectl scale deployment myapp --replicas=2 -n ckad
```

Create a temporary Pod using the image `busybox:1.36.1` and execute a wget command against the IP of the service:

```bash
kubectl run temp-pod --image=busybox:1.36.1 -n ckad --restart=Never --command -- sh -c "wget -qO- http://myapp:80"
```

Change the service type to `NodePort` so that the Pods can be reached from outside of the cluster:

`myapp-service-v2.yaml` file:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: ckad
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 30080
  selector:
    app: myapp
``` 

Apply the updated Service configuration:

```bash
kubectl apply -f myapp-service-v2.yaml
```

Get node IPs to access the service from outside the cluster:

```bash
kubectl get nodes -o wide
```

Execute a wget command against the service from outside of the cluster:

```bash
wget -qO- http://<node-ip>:30080
```

Destroy all resources created during this exercise:

```bash
kubectl delete service myapp -n ckad
kubectl delete deployment myapp -n ckad
kubectl delete pod temp-pod -n ckad
```

## Exercise 2

### Description

Jacinto is a developer in charge of implementing a web-based application stack. He is not familiar with Kubernetes, and asked if you could help out. The relevant objects have been created; however, connection to the application cannot be established from within the cluster. Help Jacinto with fixing the configuration of her YAML manifests.

`setup.yaml` contains the following objects:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: y72
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: y72
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - image: bmuschko/nodejs-hello-world:1.0.0
        name: web-app
        ports:
        - containerPort: 3000
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: web-app
  namespace: y72
spec:
  type: ClusterIP
  selector:
    run: myapp
  ports:
  - port: 80
    protocol: TCP
    targetPort: 3001
```

Create the objects from the YAML manifest setup.yaml. Inspect the objects in the namespace `y72`.

Create a temporary Pod using the image busybox:1.36.1 in the namespace `y72`. The container command should make a wget call to the Service web-app. The wget call will not be able to establish a successful connection to the Service.

Identify the root cause for the connection issue and fix it. Verify the correct behavior by repeating the previous step. The wget call should return a successful response.

### Solution    

Create the objects from the YAML manifest `setup.yaml`:

```bash
kubectl apply -f setup.yaml
```

Create a temporary Pod using the image `busybox:1.36.1` in the namespace `y72`:

```bash
kubectl run temp-pod --image=busybox:1.36.1 --restart=Never -n y72 --command -- sh -c "wget -qO- http://web-app:80"
```

The wget call will not be able to establish a successful connection to the Service due to a misconfiguration in the Service selector and target port.
Identify the root cause for the connection issue:
The Service selector is incorrect. It should match the label `app: web-app` instead of `run: myapp`. Also, the target port should be `3000` instead of `3001`.

Fix the Service configuration by updating the selector and target port:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app
  namespace: y72
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    protocol: TCP
    targetPort: 3000
```     

Apply the updated Service configuration:

```bash
kubectl apply -f updated-service.yaml
```

Verify the correct behavior by repeating the previous step:

```bash
kubectl run temp-pod --image=busybox:1.36.1 --restart=Never -n y72 --command -- sh -c "wget -qO- http://web-app:80"
```

The wget call should now return a successful response.
Destroy all resources created during this exercise:

```bash
kubectl delete namespace y72
kubectl delete pod temp-pod -n y72
kubectl delete service web-app -n y72
kubectl delete deployment web-app -n y72
```

