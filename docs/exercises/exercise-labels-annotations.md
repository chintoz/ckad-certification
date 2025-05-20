# Exercises Study Group - Session 6

## Description - Labels and Annotations

Create three Pods that use the image nginx:1.25.1. The names of the Pods should be pod-1, pod-2, and pod-3.

Assign the label tier=frontend to pod-1 and the label tier=backend to pod-2 and pod-3. All pods should also assign the label team=artemidis.

Assign the annotation with the key deployer to pod-1 and pod-3. Use your own name as the value.

From the command line, use label selection to find all Pods with the team artemidis or aircontrol and that are considered a backend service.

## Solution

Create three Pods with the following YAML manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: ckad
  labels:
    tier: frontend
    team: artemidis
  annotations:
    deployer: jacinto
spec:
    containers:
        - name: nginx
          image: nginx:1.25.1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
  namespace: ckad
  labels:
    tier: backend
    team: artemidis
  annotations:
    deployer:  jacinto  
spec:
  containers:
    - name: nginx
      image: nginx:1.25.1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  namespace: ckad
  labels:
    tier: backend
    team: artemidis
spec:
  containers:
    - name: nginx
      image: nginx:1.25.1
```

Create the Pods from the YAML manifest:

```bash
kubectl apply -f pods.yaml
```

Use label selection to find all Pods with the team artemidis or aircontrol and that are considered a backend service:

```bash
kubectl get pods -l 'team in (artemidis, aircontrol), tier=backend' -n ckad
```

## Description - Labels and Annotations

Create a Pod with the image nginx:1.25.1 that assigns two recommended labels: one for defining the application name with the value F5-nginx, and one for defining the tool used to manage the application named helm.

Render the assigned labels of the Pod object.

## Solution

Create a Pod with the following YAML manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: f5-nginx
  namespace: ckad
  labels:
    app: f5-nginx
    tool: helm
spec:
  containers:
    - name: nginx
      image: nginx:1.25.1
```

Render the assigned labels of the Pod object:

```bash
kubectl get pod f5-nginx -n ckad --show-labels
```

