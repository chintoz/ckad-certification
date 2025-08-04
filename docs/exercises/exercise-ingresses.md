# Exercise Working Session 13 - Ingresses

## Preconditions

Create cluster with extra port mapping to allow traffic on ports 80 and 443:

```bash
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

Setup ingress controller:

```bash
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

Wait until the ingress controller is ready:

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```


## Exercise 1

### Description

Create a new Deployment named web that controls a single replica running the image bmuschko/nodejs-hello-world:1.0.0 on port 3000. Expose the Deployment with a Service named web of type ClusterIP. The Service routes traffic to the Pods controlled by the Deployment web. Make a request to the endpoint of the application on the context path /. You should see the message “Hello World.”

Create an Ingress that exposes the path / for the host hello-world.exposed. The traffic should be routed to the Service created earlier. List the Ingress object.

Add an entry in /etc/hosts that maps the load balancer IP address to the host hello-world.exposed. Make a request to http://hello-world.exposed. You should see the message “Hello World.”

### Solution

Create a deployment and service using the following YAML manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: bmuschko/nodejs-hello-world:1.0.0
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
  - port: 3000
    targetPort: 3000
```

Apply the above manifest to create the Deployment and Service:

```bash
kubectl apply -f session13-exercise1-service.yaml
```


Create an Ingress resource to expose the application:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: hello-world.exposed
    http:
      paths:
        - pathType: Prefix
          path: /
          backend:
            service:
              name: web
              port:
                number: 3000
```

Create the objects from the YAML manifest and apply them to your Kubernetes cluster:

```bash
kubectl apply -f session13-exercise1-ingress.yaml
```

Inspect the created objects in the cluster:

```bash
kubectl get deployments,services,ingresses
```

Do a request to the cluster to get ingress ip:

```bash
kubectl get ingress web-ingress
```

Add an entry in your `/etc/hosts` file that maps the Ingress IP to the hostname `hello-world.exposed`. For example:

```
127.0.0.1 hello-world.exposed
```

Make a request to the Ingress using `curl` or `wget`:

```bash
curl http://hello-world.exposed
```

You should see the response:

```
Hello World
```

Destroy all the resources created for this exercise:

```bash
kubectl delete deployment web
kubectl delete service web
kubectl delete ingress web-ingress
```

                
## Exercise 2

### Description

Any application has been exposed by an Ingress. Some of your end users report an issue with connecting to the application from outside of the cluster. Inspect the existing setup and fix the problem for your end users.

Given setup configuration session13-exercise2-setup.yaml:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: s96
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: s96
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.25.3-alpine
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: s96
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 9999
    targetPort: 80
    protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  namespace: s96
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: faulty.ingress.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx
                port:
                  number: 3333
```

Create the objects from the YAML manifest setup.yaml. Inspect the objects in the namespace s96. Create an entry in /etc/hosts for the hostname faulty.ingress.com.

Perform a HTTP call to faulty.ingress.com/ using wget or curl. Inspect the connection error.

Change the configuration to ensure that end users can connect to the Ingress. Verify proper connectivity by performing another HTTP call.


### Solution 

Create the objects from the provided YAML manifest:

```bash
kubectl apply -f session13-exercise2-setup.yaml
```

Inspect the created objects in the `s96` namespace:

```bash
kubectl get all -n s96
```

Add an entry in your `/etc/hosts` file that maps the Ingress IP to the hostname `faulty.ingress.com`. For example:

```
127.0.0.1 faulty.ingress.com
```

Make a request to the Ingress using `curl` or `wget`:

```bash
curl http://faulty.ingress.com/
``` 

You should see an error indicating that the connection is refused or the service is not reachable.

This is the error code:

```
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body>
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

To fix the issue, you need to correct the port number in the Ingress resource. The Ingress is currently trying to route traffic to port `3333`, which does not match the Service's exposed port `9999`. Update the Ingress resource as follows:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: nginx
    namespace: s96
spec:
    rules:
    - host: faulty.ingress.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx
                port:
                  number: 9999
```

Apply the updated Ingress configuration:

```bash
kubectl apply -f session13-exercise2-ingress.yaml
```

Verify the Ingress configuration again:

```bash
kubectl get ingress nginx -n s96
```

Make another request to the Ingress using `curl` or `wget`:

```bash
curl http://faulty.ingress.com/
```

Destroy all the resources created for this exercise:

```bash
kubectl delete namespace s96
```
