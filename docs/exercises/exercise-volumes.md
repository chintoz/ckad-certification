# Exercise Working Group Session 4

## Description - Mount Volumes

Create a Pod YAML manifest with two containers that use the image alpine:3.12.0. Provide a command for both containers that keep them running forever.

Define a Volume of type emptyDir for the Pod. Container 1 should mount the Volume to path /etc/a, and container 2 should mount the Volume to path /etc/b.

Open an interactive shell for container 1 and create the directory data in the mount path. Navigate to the directory and create the file hello.txt with the contents “Hello World.” Exit out of the container.

Open an interactive shell for container 2 and navigate to the directory /etc/b/data. Inspect the contents of file hello.txt. Exit out of the container.

### Solution

This will be the yaml file created for the exercise:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: two-container
  namespace: ckad
spec:
  volumes:
    - name: share-data
      emptyDir: {}
  containers:
    - name: container-1
      image: alpine:3.12.0
      args:
        - /bin/sh
        - -c
        - while true; do sleep 60; done;
      volumeMounts:
        - mountPath: "/etc/a"
          name: share-data
    - name: container-2
      image: alpine:3.12.0
      args:
        - /bin/sh
        - -c
        - while true; do sleep 60; done;
      volumeMounts:
        - mountPath: "/etc/b"
          name: share-data
```
  

Create the Pod:

```bash
kubectl apply -f two-container.yaml
```

Access to the first container:

```bash
kubectl exec -it two-container -c container-1 -n ckad -- /bin/sh
```

Create the directory and the file:

```bash
mkdir /etc/a/data
cd /etc/a/data
echo "Hello World" > hello.txt
```

Access to the second container:

```bash
kubectl exec -it two-container -c container-2 -n ckad -- /bin/sh
```

Check the content of the file:

```bash
cat /etc/b/data/hello.txt
```

## Description - Persistent Volumes

Create a PersistentVolume named logs-pv that maps to the hostPath /var/logs. The access mode should be ReadWriteOnce and ReadOnlyMany. Provision a storage capacity of 5Gi. Ensure that the status of the PersistentVolume shows Available.

Create a PersistentVolumeClaim named logs-pvc. It uses ReadWriteOnce access. Request a capacity of 2Gi. Ensure that the status of the PersistentVolume shows Bound.

Mount the PersistentVolumeClaim in a Pod running the image nginx at the mount path /var/log/nginx.

Open an interactive shell to the container and create a new file named my-nginx.log in /var/log/nginx. Exit out of the Pod.

Delete the Pod and re-create it with the same YAML manifest. Open an interactive shell to the Pod, navigate to the directory /var/log/nginx, and find the file you created before.


### Solution

Create the PersistentVolume:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: logs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /var/logs
```

Create persistent volume claim:

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: logs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ""
  resources:
    requests:
      storage: 2Gi
```

Mount the PersistentVolumeClaim in a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: logs-pvc
  containers:
  - image: nginx:1.17.10
    name: nginx
    volumeMounts:
      - mountPath: "/var/log/nginx"
        name: storage
```

Apply all the files:

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f pv-pod.yaml
```

Access to the Pod:

```bash
kubectl exec -it nginx-pod -n ckad -- /bin/sh
```

Navigate to the directory and check the file:

```bash
cd /var/log/nginx
ls
```

Create a new file:

```bash
echo "Hello World" > my-nginx.log
```

Delete the Pod:

```bash
kubectl delete pod nginx-pod -n ckad
```

Re-create the Pod:

```bash
kubectl apply -f pv-pod.yaml
```

Access to the Pod:

```bash
kubectl exec -it nginx-pod -n ckad -- /bin/sh
```

Navigate to the directory and check the file:

```bash
cd /var/log/nginx
cat my-nginx.log
```

Destroy everything:

```bash
kubectl delete -f pv-pod.yaml
kubectl delete -f pvc.yaml
kubectl delete -f pv.yaml
```
