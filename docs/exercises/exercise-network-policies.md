# Exercise Working Session 14 — Network Policies

## Preconditions

Install Calico CNI mechanism following the instruction from here: https://docs.tigera.io/calico/latest/getting-started/kubernetes/kind

In summary:

Create a new cluster:

```bash
cat > values.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
networking:
  disableDefaultCNI: true
  podSubnet: 192.168.0.0/16
EOF
kind create cluster --config values.yaml --name dev
```

Once the cluster is created, Calico CNI can be installed:

First, install the Custom Resource Definitions and the Tigera operator:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/operator-crds.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/tigera-operator.yaml
```

Next, install Calico itself:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/custom-resources.yaml
```

Verify that all Calico pods are running:

```bash
kubectl get pods -l k8s-app=calico-node -A
```

Everything should be in the Running state. It takes a few minutes for all components to be up and running.

## Exercise 1

### Description

You have been tasked with setting up a network policy for an existing application stack that consists of a frontend Pod in the namespace end-user and a backend Pod in the namespace internal.

Given the following setup file `session14-exercise1-setup.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: end-user
  labels:
    access: outside
---
apiVersion: v1
kind: Namespace
metadata:
  name: internal
  labels:
    access: inside
---
kind: Pod
apiVersion: v1
metadata:
  name: frontend
  namespace: end-user
  labels:
    app: frontend
spec:
  containers:
  - name: busybox
    image: busybox:1.36.1
    command: ["sh", "-c", "sleep 1h"]
---
kind: Pod
apiVersion: v1
metadata:
  name: backend
  namespace: internal
  labels:
    app: backend
spec:
  containers:
  - name: nginx
    image: nginx:1.25.3-alpine
    ports:
    - containerPort: 80
      protocol: TCP

```

Inspect the objects in both namespaces.

Create a network policy named app-stack in the end-user namespace. Allow egress traffic only from the frontend Pod to the backend Pod. The backend Pod should be reachable only on port 80.

### Solution

Execute the following command to create the initial setup:

```bash
kubectl apply -f session14-exercise1-setup.yaml
```

Create a file named `session14-exercise1-network-policy.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: app-stack
    namespace: end-user
spec:
    podSelector:
        matchLabels:
            app: frontend
    policyTypes:
    - Egress
    egress:
      - to:
          - namespaceSelector:
              matchLabels:
                access: inside
            podSelector:
              matchLabels:
                app: backend
        ports: 
          - protocol: TCP
            port: 80
``` 

Apply the network policy:

```bash
kubectl apply -f session14-exercise1-network-policy.yaml
```

Verify the network policy is in place:

```bash
kubectl get networkpolicy -n end-user
```
Test the connectivity from the frontend Pod to the backend Pod:

```bash
kubectl exec -n end-user frontend -- wget -qO- http://backend.internal.svc.cluster.local
``` 

You should see the HTML content of the Nginx welcome page, indicating that the network policy is correctly allowing traffic from the frontend Pod to the backend Pod on port 80.

## Exercise 2

### Description

Given the following setup file `session-14-exercise2-setup.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: k1
  labels:
    role: consumer
---
apiVersion: v1
kind: Namespace
metadata:
  name: k2
  labels:
    role: producer
---
kind: Pod
apiVersion: v1
metadata:
  name: busybox
  namespace: k1
  labels:
    app: frontend
spec:
  containers:
  - name: busybox
    image: busybox:1.36.1
    command: ["sh", "-c", "sleep 1h"]
---
kind: Pod
apiVersion: v1
metadata:
  name: nginx
  namespace: k2
  labels:
    app: backend
spec:
  containers:
  - name: nginx
    image: nginx:1.23.4-alpine
    ports:
    - containerPort: 80
      protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: k2
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  
```

Create the objects from the YAML manifest setup.yaml. Inspect the objects in the namespace k1 and k2.

Determine the virtual IP address of Pod nginx in namespace k2. Try to make a wget call on port 80 from the Pod busybox in namespace k1 to the Pod nginx in namespace k2. The call will fail with the current setup.

Create a network policy that allows performing ingress calls for all Pods in namespace k1 to the Pod nginx in namespace k2. Pods in all other namespaces should be denied to make ingress calls to Pods in namespace k2.

Verify that a network connection can be established.

### Solution

Create the initial setup:

```bash
kubectl apply -f session-14-exercise2-setup.yaml
```

Get the IP address of the nginx Pod:

```bash
kubectl get pod -n k2 -o wide
```

Try to make a wget call from the busybox Pod to the nginx Pod:

```bash
kubectl exec -n k1 busybox -- wget -qO- http://<nginx-pod-ip>
```

The call will fail because of the default deny ingress policy.
Create a file named `session14-exercise2-network-policy.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: allow-k1-to-k2
    namespace: k2
spec:
    podSelector:
        matchLabels:
            app: backend
    policyTypes:
    - Ingress
    ingress:
        - from:
            - namespaceSelector:
                matchLabels:
                  role: consumer
```

Apply the network policy:

```bash
kubectl apply -f network-policy.yaml
```

Verify the network policy is in place:

```bash
kubectl get networkpolicy -n k2
```

Test the connectivity from the busybox Pod to the nginx Pod:

```bash
kubectl exec -n k1 busybox -- wget -qO- http://<nginx-pod-ip>
```

You should see the HTML content of the Nginx welcome page, indicating that the network policy is correctly allowing traffic from Pods in namespace k1 to the nginx Pod in namespace k2.


