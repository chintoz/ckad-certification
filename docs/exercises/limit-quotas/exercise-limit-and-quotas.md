# Exercise Working Session 15 — Limit and Quotas

## Exercise 1

### Description

You have been tasked with creating a Pod for running an application in a container. During application development, you ran a load test for figuring out the minimum amount of resources needed and the maximum amount of resources the application is allowed to grow to. Define those resource requests and limits for the Pod.

Define a Pod named hello-world running the container image bmuschko/nodejs-hello-world:1.0.0. The container exposes the port 3000.

Add a Volume of type emptyDir and mount it in the container path /var/log.

For the container, specify the following minimum number of resources:

```
CPU: 100m
Memory: 500Mi
Ephemeral storage: 1Gi
```

For the container, specify the following maximum number of resources:

```
Memory: 500Mi
Ephemeral storage: 2Gi
```

Create the Pod from the YAML manifest. Inspect the Pod details. Which node does the Pod run on?

### Solution

Define the Pod manifest as follows and save it to a file named `hello-world-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
  - name: hello-world
    image: bmuschko/nodejs-hello-world:1.0.0
    ports:
    - containerPort: 3000
    resources:
      requests:
        memory: "500Mi"
        cpu: "100m"
        ephemeral-storage: "1Gi"
      limits:
        memory: "500Mi"
        ephemeral-storage: "2Gi"
    volumeMounts:
    - mountPath: /var/log
      name: log-volume
  volumes:
  - name: log-volume
    emptyDir: {}
```

Create the Pod using the following command:

```bash
kubectl apply -f hello-world-pod.yaml
```

Inspect the Pod details with:

```bash
kubectl get pod hello-world -o wide
``` 

The output will show the node on which the Pod is running in the `NODE` column.
The Pod should be running on one of the available nodes in your cluster.


## Exercise 2

### Description

In this exercise, you will create a resource quota with specific CPU and memory limits for a new namespace. Pods created in the namespace will have to adhere to those limits.

Create a ResourceQuota named app under the namespace rq-demo using the following YAML definition in the file `resourcequota.yaml:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: app
spec:
  hard:
    pods: "2"
    requests.cpu: "2"
    requests.memory: 500Mi
```

Create a new Pod that exceeds the limits of the resource quota requirements, e.g., by defining 1Gi of memory but stays below the CPU, e.g., 0.5. Write down the error message.

Change the request limits to fulfill the requirements to ensure that the Pod can be created successfully. Write down the output of the command that renders the used amount of resources for the namespace.

### Solution

Create the namespace:

```bash
kubectl create namespace rq-demo
```

Create the ResourceQuota:

```bash
kubectl apply -f resourcequota.yaml -n rq-demo
```

Create a Pod manifest that exceeds the memory limit, e.g., `exceeding-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exceeding-pod
  namespace: rq-demo
spec:
  containers:
  - name: app-container
    image: bmuschko/nodejs-hello-world:1.0.0
    resources:
      requests:
        memory: "1Gi"
        cpu: "0.5"
```

Try to create the Pod:

```bash
kubectl apply -f exceeding-pod.yaml
```

You should see an error message similar to:

```
Error from server (Forbidden): error when creating "exceeding-pod.yaml": pods "exceeding-pod" is forbidden: exceeded quota: app, requested: pods=1, requests.cpu=0.5, requests.memory=1Gi, used: pods=0, requests.cpu=0, requests.memory=0
```

Now, modify the Pod manifest to stay within the limits, e.g., `valid-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: valid-pod
  namespace: rq-demo
spec:
  containers:
  - name: app-container
    image: bmuschko/nodejs-hello-world:1.0.0
    resources:
      requests:
        memory: "500Mi"
        cpu: "0.5"
```

Create the valid Pod:

```bash
kubectl apply -f valid-pod.yaml
``` 

Check the resource usage in the namespace:

```bash
kubectl get resourcequota -n rq-demo
``` 

The output will show the used amount of resources for the namespace, e.g.:

```
NAME   PODS   REQUESTS.CPU   REQUESTS.MEMORY
app    1      0.5            500Mi
``` 

This indicates that one Pod is running, using 0.5 CPU and 500Mi of memory, which is within the defined resource quota limits.

## Exercise 3

### Description

A LimitRange can restrict resource consumption for Pods in a namespace, and assign default computing resources if no resource requirements have been defined. You will practice the effects of a LimitRange on the creation of a Pod in different scenarios.

Inspect the YAML manifest definition in the file setup.yaml. Create the objects from the YAML manifest file `setup.yaml`.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: d92
---
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
  namespace: d92
spec:
  limits:
  - min:
      cpu: 200m
    max:
      cpu: 500m
    type: Container
```

Create a new Pod named pod-without-resource-requirements in the namespace d92 that uses the container image nginx:1.23.4-alpine without any resource requirements. Inspect the Pod details. What resource definitions do you expect to be assigned?

Create a new Pod named pod-with-more-cpu-resource-requirements in the namespace d92 that uses the container image nginx:1.23.4-alpine with a CPU resource request of 400m and limits of 1.5. What runtime behavior do you expect to see?

Create a new Pod named pod-with-less-cpu-resource-requirements in the namespace d92 that uses the container image nginx:1.23.4-alpine with a CPU resource request of 350m and limits of 400m. What runtime behavior do you expect to see?

### Solution

Create the namespace and LimitRange:

```bash
kubectl apply -f setup.yaml
```

Create the Pod without resource requirements, e.g., `pod-without-resource-requirements.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-without-resource-requirements
  namespace: d92
spec:
  containers:
  - name: nginx
    image: nginx:1.23.4-alpine
```

Create the Pod:

```bash
kubectl apply -f pod-without-resource-requirements.yaml
```

Inspect the Pod details:

```bash
kubectl get pod pod-without-resource-requirements -n d92 -o yaml
```

You should see that the Pod has been assigned a CPU request of 200m and a limit of 500m as per the LimitRange. 

Create the Pod with more CPU resource requirements, e.g., `pod-with-more-cpu-resource-requirements.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-more-cpu-resource-requirements
  namespace: d92
spec:
  containers:
  - name: nginx
    image: nginx:1.23.4-alpine
    resources:
      requests:
        cpu: "400m"
      limits:
        cpu: "1.5"
```

Create the Pod:

```bash
kubectl apply -f pod-with-more-cpu-resource-requirements.yaml
```

You should see an error message similar to:

```Error from server (Forbidden): error when creating "pod-with-more-cpu-resource-requirements.yaml": pods "pod-with-more-cpu-resource-requirements" is forbidden: exceeded quota: cpu-limit-range, requested: requests.cpu=400m, limits.cpu=1.5, used: requests.cpu=200m, limits.cpu=500m```

This indicates that the Pod cannot be created because the CPU limit of 1.5 exceeds the maximum allowed by the LimitRange (500m).

Create the Pod with less CPU resource requirements, e.g., `pod-with-less-cpu-resource-requirements.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-less-cpu-resource-requirements
  namespace: d92
spec:
  containers:
  - name: nginx
    image: nginx:1.23.4-alpine
    resources:
      requests:
        cpu: "350m"
      limits:
        cpu: "400m"
```

Create the Pod:

```bash
kubectl apply -f pod-with-less-cpu-resource-requirements.yaml
```

The Pod should be created successfully, as the CPU request of 350m and limit of 400m are within the defined limits of the LimitRange (200m to 500m).

Inspect the Pod details:

```bash
kubectl get pod pod-with-less-cpu-resource-requirements -n d92 -o yaml
```
You should see that the Pod has the specified CPU request of 350m and limit of 400m.
