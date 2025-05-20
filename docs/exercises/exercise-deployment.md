# Exercise Working Group Session 7

## Exercise 1

### Description

Create a Deployment named nginx with 3 replicas. The Pods should use the nginx:1.23.0 image and the name nginx. The Deployment uses the label tier=backend. The Pod template should use the label app=v1.

List the Deployment and ensure that the correct number of replicas is running.

Update the image to nginx:1.23.4.

Verify that the change has been rolled out to all replicas.

Assign the change cause “Pick up patch version” to the revision.

Scale the Deployment to 5 replicas.

Have a look at the Deployment rollout history. Revert the Deployment to revision 1.

Ensure that the Pods use the image nginx:1.23.0.

### Solution

Create a deployment named nginx with 3 replicas using yaml file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: ckad
spec:
    replicas: 3
    selector:
        matchLabels:
          app: v1
          tier: backend
    template:
        metadata:
          labels:
            app: v1
            tier: backend
        spec:
          containers:
            - name: nginx
              image: nginx:1.23.0
```

List the deployment and ensure that the correct number of replicas is running:

```bash
kubectl get deployment nginx -n ckad
```

Update the image to nginx:1.23.4:

```bash
kubectl set image deployment/nginx nginx=nginx:1.23.4 -n ckad
```

After the update, verify that the change has been rolled out to all replicas:

```bash
kubectl rollout status deployment/nginx -n ckad
```

Assign the change cause “Pick up patch version” to the revision:

```bash
kubectl annotate deployment nginx -n ckad kubernetes.io/change-cause="Pick up patch version"
```

Scale the deployment to 5 replicas:

```bash
kubectl scale deployment nginx --replicas=5 -n ckad
```

Have a look at the deployment rollout history:

```bash
kubectl rollout history deployment/nginx -n ckad
```

Revert the deployment to revision 1:

```bash
kubectl rollout undo deployment/nginx --to-revision=1 -n ckad
```

Ensure that the Pods use the image nginx:1.23.0:

```bash
kubectl get pods -l app=v1 -n ckad
```

```bash
kubectl describe pod <pod-name> -n ckad
```

Delete the deployment:

```bash
kubectl delete deployment nginx -n ckad
```


## Exercise 2

### Description

Create a Deployment named nginx with 1 replica. The Pod template of the Deployment should use container image nginx:1.23.4; set the CPU resource request to 0.5 and the memory resource request/limit to 500Mi.

Create a HorizontalPodAutoscaler for the Deployment named nginx-hpa that scales to a minimum of 3 and a maximum of 8 replicas. Scaling should happen based on an average CPU utilization of 75% and an average memory utilization of 60%.

Inspect the HorizontalPodAutoscaler object and identify the currently-utilized resources. How many replicas do you expect to exist?


### Solution

Create a deployment named nginx with 1 replica using yaml file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: ckad
spec:
    replicas: 1
    selector:
        matchLabels:
          app: v1
          tier: backend
    template:
        metadata:
          labels:
            app: v1
            tier: backend
        spec:
          containers:
            - name: nginx
              image: nginx:1.23.4
              resources:
                requests:
                  memory: "500Mi"
                  cpu: "0.5"
                limits:
                  memory: "500Mi"
                  cpu: "0.5"
```

Create the deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

List the deployment and ensure that the correct number of replicas is running:

```bash
kubectl get deployment nginx -n ckad
``` 

Create a HorizontalPodAutoscaler for the Deployment named nginx-hpa:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
  namespace: ckad
spec:
    scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: nginx
    minReplicas: 3
    maxReplicas: 8
    metrics:
    - type: Resource
      resource:
        name: cpu
        target:
            type: Utilization
            averageUtilization: 75
    - type: Resource
      resource:
        name: memory
        target:
            type: Utilization
            averageUtilization: 60
```

Create the HorizontalPodAutoscaler:

```bash
kubectl apply -f nginx-hpa.yaml
``` 

Inspect the HorizontalPodAutoscaler object and identify the currently-utilized resources:

```bash
kubectl describe hpa nginx-hpa -n ckad
```

How many replicas do you expect to exist? 3

```bash
kubectl get hpa nginx-hpa -n ckad
```

Delete the deployment:

```bash
kubectl delete deployment nginx -n ckad
```

Delete the HorizontalPodAutoscaler:

```bash
kubectl delete hpa nginx-hpa -n ckad
```


