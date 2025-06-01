# Exercise Working group Session 8 - Deployment Strategy

## Exercise 1

One of your teammates created a Deployment YAML manifest to operate the container image grafana/grafana:9.5.9. Create the Deployment object from the YAML manifest file deployment-grafana.yaml:

deployment-grafana.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: ckad
spec:
  replicas: 6
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - image: grafana/grafana:9.5.9
        name: grafana
        ports:
        - containerPort: 3000
```


You need to update all replicas with the container image grafana/grafana:10.1.2. Make sure that the rollout happens in batches of two replicas at a time. Ensure that a readiness probe is defined.

### Solution

Create the Deployment object from the YAML manifest file `deployment-grafana.yaml`:

```bash
kubectl apply -f deployment-grafana.yaml
```

Update the Deployment to use the new image and set the strategy to RollingUpdate with maxSurge and maxUnavailable set to 2:

deployment-grafana-update.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: ckad
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 2
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - image: grafana/grafana:10.1.2
          name: grafana
          ports:
            - containerPort: 3000
          readinessProbe:
            httpGet:
              path: /
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
```

Update the Deployment with the new manifest:

```bash
kubectl apply -f deployment-grafana-update.yaml
```

Verify the rollout status:

```bash
kubectl rollout status deployment/grafana -n ckad
```

Check the Pods to ensure that the new image is being used:

```bash
kubectl get pods -n ckad -l app=grafana
```

## Exercise 2

In this exercise, you will set up a blue-green Deployment scenario. You’ll first create the initial (blue) Deployment and expose it with a Service. Later, you will create a second (green) Deployment and switch over traffic.

Create a Deployment named nginx-blue with 3 replicas. The Pod template of the Deployment should use container image nginx:1.23.0 and assign the label version=blue.

Expose the Deployment with a Service of type ClusterIP named nginx. Map the incoming and outgoing port to 80. Select the Pod with label version=blue.

Run a temporary Pod with the container image alpine/curl:3.14 to make a call against the Service using curl.

Create a second Deployment named nginx-green with 3 replicas. The Pod template of the Deployment should use container image nginx:1.23.4 and assign the label version=green.

Change the Service’s label selection so that traffic will be routed to the Pods controlled by the Deployment nginx-green.

Delete the Deployment named nginx-blue.

Run a temporary Pod with the container image alpine/curl:3.14 to make a call against the Service.

### Solution

Create a deployment named nginx-blue with 3 replicas using the following YAML manifest:

deployment-nginx-blue.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-blue
  namespace: ckad
spec: 
  replicas: 3
  selector:
    matchLabels:
      version: blue
  template:
    metadata:
      labels:
        version: blue
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.0
        ports:
        - containerPort: 80
```

Expose the Deployment with a Service of type ClusterIP named nginx:

service-nginx-v1.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: ckad
spec:
  type: ClusterIP
  selector:
    version: blue
  ports:
  - port: 80
    targetPort: 80
```

Run a temporary Pod with the container image alpine/curl:3.14 to make a call against the Service:

```bash
kubectl run curl-pod --image=alpine/curl:3.14 --restart=Never --rm -it -- /bin/sh -c "apk add curl && curl nginx.ckad.svc.cluster.local"
```

Create a second Deployment named nginx-green with 3 replicas using the following YAML manifest:

deployment-nginx-green.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-green
  namespace: ckad
spec:
    replicas: 3
    selector:
      matchLabels:
        version: green
    template:
      metadata:
        labels:
            version: green
      spec:
        containers:
          - name: nginx
            image: nginx:1.23.4
            ports:
            - containerPort: 80
```

Change the Service’s label selection to route traffic to the Pods controlled by the Deployment nginx-green:

service-nginx-v2.yaml
```yaml
apiVersion: v1 
kind: Service
metadata:
  name: nginx
  namespace: ckad
spec:
  type: ClusterIP
  selector:
    version: green
  ports:
  - port: 80
    targetPort: 80
```

Apply the updated Service manifest:

```bash
kubectl apply -f service-nginx.yaml
```

Delete the Deployment named nginx-blue:

```bash
kubectl delete deployment nginx-blue -n ckad
```

Run a temporary Pod with the container image alpine/curl:3.14 to make a call against the Service:

```bash
kubectl run curl-pod --image=alpine/curl:3.14 --restart=Never --rm -it -- /bin/sh -c "apk add curl && curl nginx.ckad.svc.cluster.local"
```

