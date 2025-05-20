# Exercise Working Group Session 5

## Description - Init container

Create a YAML manifest for a Pod named complex-pod. The main application container named app should use the image nginx:1.25.1 and expose the container port 80. Modify the YAML manifest so that the Pod defines an init container named setup that uses the image busybox:1.36.1. The init container runs the command wget -O- google.com.

Create the Pod from the YAML manifest.

Download the logs of the init container. You should see the output of the wget command.

Open an interactive shell to the main application container and run the ls command. Exit out of the container.

Force-delete the Pod.

## Solution

Create a YAML manifest for the Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: complex-pod
  namespace: ckad
spec:
    containers:
      - name: app
        image: nginx:1.25.1
        ports:
            - containerPort: 80
    initContainers:
      - name: setup
        image: busybox:1.36.1
        command: ["wget", "-O-", "google.com"]
```

Download the logs of the init container:

```bash
kubectl logs complex-pod -c setup -n ckad
```

Open an interactive shell to the main application container:

```bash
kubectl exec -it complex-pod -c app -n ckad -- /bin/sh
```

Run the ls command:

```bash
ls
```

Exit out of the container:

```bash
exit
```

Delete the Pod:

```bash
kubectl delete pod complex-pod --force --grace-period=0
```



## Description - Sidecar pattern

Create a YAML manifest for a Pod named data-exchange. The main application container named main-app should use the image busybox:1.36.1. The container runs a command that writes a new file every 30 seconds in an infinite loop in the directory /var/app/data. The filename follows the pattern {counter++}-data.txt. The variable counter is incremented every interval and starts with the value 1.

Modify the YAML manifest by adding a sidecar container named sidecar. The sidecar container uses the image busybox:1.36.1 and runs a command that counts the number of files produced by the main-app container every 60 seconds in an infinite loop. The command writes the number of files to standard output.

Define a Volume of type emptyDir. Mount the path /var/app/data for both containers.

Create the Pod. Tail the logs of the sidecar container.

Delete the Pod.

## Solution

Create a YAML manifest for the Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-exchange
  namespace: ckad
spec:
    containers:
      - name: main-app
        image: busybox:1.36.1
        command: ["sh", "-c", "counter=1; while true; do echo \"${counter}-data.txt\" > /var/app/data/${counter}-data.txt; counter=$((counter + 1)); sleep 30; done"]
        volumeMounts:
          - mountPath: /var/app/data
            name: data-volume
      - name: sidecar
        image: busybox:1.36.1
        command: ["sh", "-c", "while true; do ls /var/app/data | wc -l; sleep 60; done"]
        volumeMounts:
          - mountPath: /var/app/data
            name: data-volume
    volumes:
      - name: data-volume
        emptyDir: {}
```

Create the Pod:

```bash
kubectl apply -f data-exchange.yaml
```

Tail the logs of the sidecar container:

```bash
kubectl logs -f data-exchange -c sidecar -n ckad
```

Delete the Pod:

```bash
kubectl delete pod data-exchange --force --grace-period=0
```