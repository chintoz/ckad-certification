# Exercise Working Group Session 2

## Description

Create a new Pod named nginx running the image nginx:1.17.10. Expose the container port 80. The Pod should live in the namespace named ckad.

Get the details of the Pod including its IP address.

Create a temporary Pod that uses the busybox:1.36.1 image to execute a wget command inside of the container. The wget command should access the endpoint exposed by the nginx container. You should see the HTML response body rendered in the terminal.

Get the logs of the nginx container.

Add the environment variables DB_URL=postgresql://mydb:5432 and DB_USERNAME=admin to the container of the nginx Pod.

Open a shell for the nginx container and inspect the contents of the current directory ls -l. Exit out of the container.


## Solution

### Using CLI

1. Creaate namespace

```bash
kubectl create namespace ckad
```

2. Create pod

```bash
kubectl run nginx --image=nginx:1.17.10 --port=80 --namespace=ckad
```
3. Get pod details

```bash
kubectl get pod nginx -n ckad -o wide
```

4. Create temporary pod to access nginx (use the ip returned from the step number 3)

```bash
kubectl run busybox --rm -ti --image=busybox:1.36.1 --namespace=ckad
```

Call wget, this example with IP

```bash
wget http://10.244.0.5
```

5. Get nginx logs

```bash
kubectl logs nginx -n ckad
```

6. Add environment variables

```bash
kubectl run nginx --image=nginx:1.17.10 --port=80 --namespace=ckad --env="DB_URL=postgresql://mydb:5432" --env="DB_USERNAME=admin"
```

7. Open a shell for the nginx container and inspect the contents of the current directory ls -l. Exit out of the container.

```bash
kubectl exec -it nginx -n ckad -- /bin/sh
```

Print environment variables

```bash
printenv
```

### Using YAML

0. How to create dry runs

```bash
kubectl create namespace ckad -o yaml --dry-run=client
kubectl run nginx --image=nginx:1.17.10 --port=80 --namespace=ckad -o yaml --dry-run=client
```

1. Create namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
    name: ckad
```

2. Create pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: ckad
spec:
  containers:
  - image: nginx:1.17.10
    name: nginx
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

3. Get pod details

```bash
kubectl get pod nginx -n ckad -o wide
```

4. Create temporary pod to access nginx (use the ip returned from the step number 3)

```bash
kubectl run busybox --rm -ti --image=busybox:1.36.1 --namespace=ckad
```

Call wget, this example with IP

```bash
wget http://10.244.0.5
```

5. Get nginx logs

```bash
kubectl logs nginx -n ckad
```

6. Add environment variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: ckad
spec:
  containers:
  - image: nginx:1.17.10
    name: nginx
    env:
      - name: DB_URL
        value: "postgresql://mydb:5432"
      - name: DB_USERNAME
        value: "admin"
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```


7. Open a shell for the nginx container and inspect the contents of the current directory ls -l. Exit out of the container.

```bash
kubectl exec -it nginx -n ckad -- /bin/sh
```

Print environment variables

```bash
printenv
```