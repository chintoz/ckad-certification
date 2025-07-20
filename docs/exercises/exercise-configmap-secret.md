# Exercise Working Session 11 - ConfigMaps and Secrets

## Exercise 1

### Description

In this exercise, you will first create a ConfigMap from a YAML configuration file as a source. Later, you’ll create a Pod, consume the ConfigMap as Volume, and inspect the key-value pairs as files.

Given the following YAML configuration file named `app-config.yaml`:

```yaml
dev:
  url: http://dev.bar.com
  name: Developer Setup
prod:
  url: http://foo.bar.com
  name: My Cool App
```

Create a new ConfigMap named app-config from that file.

Create a Pod named backend that consumes the ConfigMap as Volume at the mount path /etc/config. The container runs the image nginx:1.23.4-alpine.

Shell into the Pod and inspect the file at the mounted Volume path.

Destroy all the resources after inspection.


### Solution

Create a ConfigMap named `app-config` from the file `app-config.yaml`:

```bash
kubectl create configmap app-config --from-file=app-config.yaml -n ckad
```

We can describe the ConfigMap to see its contents:

```bash
kubectl get configmap app-config -o yaml -n ckad
```

Create a Pod named `backend` that consumes the ConfigMap as Volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  namespace: ckad
spec:
  containers:
    - name: nginx
      image: nginx:1.23.4-alpine
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

Save the above YAML to a file named `configmap-pod.yaml` and apply it:

```bash
kubectl apply -f configmap-pod.yaml -n ckad
```

Shell into the Pod to inspect the mounted Volume:

```bash
kubectl exec -it backend -n ckad -- sh
```
Once inside the Pod, you can check the contents of the mounted Volume:

```bash
ls /etc/config
cat /etc/config/app-config.yaml
```
You should see the file `app-config.yaml` with the contents of the ConfigMap.

Delete the Pod after inspection:

```bash
kubectl delete pod backend -n ckad
```

Delete the ConfigMap after inspection:

```bash
kubectl delete configmap app-config -n ckad
```

## Exercise 2

### Description

You will first create a Secret from literal values in this exercise. Next, you’ll create a Pod and consume the Secret as environment variables. Finally, you’ll print out its values from within the container.

Create a new Secret named db-credentials with the key/value pair db-password=passwd.

Create a Pod named backend that uses the Secret as an environment variable named DB_PASSWORD and runs the container with the image nginx:1.23.4-alpine.

Shell into the Pod and print out the created environment variables. You should be able to find the DB_PASSWORD variable.

Destroy all the resources after inspection.

### Solution

Create a Secret named `db-credentials` with the key/value pair `db-password=passwd`:

```bash
kubectl create secret generic db-credentials --from-literal=db-password=passwd -n ckad
```

Create a Pod named `backend` that uses the Secret as an environment variable:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  namespace: ckad
spec:
  containers:
    - name: nginx
      image: nginx:1.23.4-alpine
      env:
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: db-credentials
            key: db-password
```

Save the above YAML to a file named `secret-pod.yaml` and apply it:

```bash
kubectl apply -f secret-pod.yaml -n ckad
```

Shell into the Pod to print out the created environment variables:

```bash
kubectl exec -it backend -n ckad -- sh
```
Once inside the Pod, you can print the environment variables:

```bash
env | grep DB_PASSWORD
```
You should see the `DB_PASSWORD` variable with the value `passwd`.
Delete the Pod after inspection:

```bash
kubectl delete pod backend -n ckad
```
Delete the Secret after inspection:

```bash
kubectl delete secret db-credentials -n ckad
```
