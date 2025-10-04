# Exercise Working Session 16 — Authentication, Authorization, and Admission Control

## Exercise 1

### Description

The premise of this exercise is to create a new user and add her to the kubeconfig file. You will then define a context that uses the user, switch to the context, and execute a `kubectl` command.

Create a certificate for a user named `mary`. Do not provide any permissions to the user.

Add the user to the kubeconfig file. Define the context named `mary-context` that assigns the user to a cluster already available in the kubeconfig file.

Set the currently selected context to `mary-context`. Create a Pod using `kubectl`. What result do you expect to see?

### Solution

Create a private key and a certificate signing request (CSR) for the user `mary`:

```bash
openssl genrsa -out mary.key 2048
openssl req -new -key mary.key -out mary.csr -subj "/CN=mary/O=developers"
```

Sign the CSR with the cluster's Certificate Authority (CA). You will need access to the CA key and certificate. The following command assumes you have access to these files:

```bash
openssl x509 -req -in mary.csr -CA ./ca.crt -CAkey ./ca.key -CAcreateserial -out mary.crt -days 365
``` 

Add the user to the kubeconfig file:

```bash
kubectl config set-credentials mary --client-certificate=mary.crt --client-key=mary.key
```

Define the context named `mary-context`:

```bash
kubectl config set-context mary-context --cluster=kind --user=mary
```

Set the currently selected context to `mary-context`:

```bash
kubectl config use-context mary-context
```

Try to create a Pod:

```bash
kubectl run test-pod --image=nginx
``` 

You should see an error message indicating that the user `mary` does not have permission to create Pods, as no permissions were granted to the user.

Destroy all the resources created during the exercise.

```bash
kubectl delete pod test-pod
kubectl config delete-context mary-context
kubectl config unset users.mary
rm mary.key mary.csr mary.crt
```


## Exercise 2

### Description

You will use RBAC to grant permissions to a service account. The permissions should apply only to certain API resources and operations.

Create a new namespace named `t23`. Create a Pod named service-list in the namespace `t23`. The container uses the image `alpine/curl:3.14` and makes a `curl` call to the Kubernetes API that lists Service objects in the default namespace in an infinite loop.

Create and attach the service account `api-call` to the Pod.

Inspect the container logs after the Pod has been started. What response do you expect to see from the `curl` command?

Assign a ClusterRole and RoleBinding to the service account that allows only the operation needed by the Pod. Note the response from the curl command.

### Solution

Create the namespace:

```bash
kubectl create namespace t23
```

Create the Pod manifest `pod-exercise2.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: service-list
  namespace: t23
spec:
  containers:
  - name: curl-container
    image: alpine/curl:3.14
    command: ["sh", "-c", "while true; do curl -k https://kubernetes.default.svc/api/v1/namespaces/default/services; sleep 5; done"]    
    env:
    - name: KUBERNETES_SERVICE_HOST
      value: "kubernetes.default.svc"
    - name: KUBERNETES_SERVICE_PORT
      value: "443"
  serviceAccountName: api-call
```

Create the Pod:

```bash
kubectl apply -f pod.yaml
```

Create the service account:

```bash
kubectl create serviceaccount api-call -n t23
```

Inspect the container logs:

```bash
kubectl logs -f service-list -n t23
```

You should see a `403 Forbidden` response from the `curl` command, as the service account does not have permissions to list Services.

Create a Role that allows listing Services in the default namespace. Create a RoleBinding to bind the Role to the service account. Create the manifest `role-binding.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: service-list-role
rules:
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: service-list-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: api-call
  namespace: t23
roleRef:
  kind: Role
  name: service-list-role
  apiGroup: rbac.authorization.k8s.io
```

Apply the Role and RoleBinding:

```bash
kubectl apply -f role-binding.yaml
```

Inspect the container logs again:

```bash
kubectl logs -f service-list -n t23
```

You should now see a list of Services in the default namespace.

Destroy all the resources created during the exercise:

```bash
kubectl delete pod service-list -n t23
kubectl delete serviceaccount api-call -n t23
kubectl delete rolebinding service-list-binding -n default
kubectl delete role service-list-role -n default
kubectl delete namespace t23
```

## Exercise 3

### Description

Identify the admission controller plugins that have been configured for the API server.

Locate the configuration file of the API server.

Inspect the command-line flag that defines the admission controller plugins. Capture the value

### Solution

Inspect the API server manifest file. The location of the file may vary depending on your Kubernetes setup. For a kubeadm-based cluster, it is typically located at `/etc/kubernetes/manifests/kube-apiserver.yaml`.

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Look for the `--enable-admission-plugins` flag in the command section of the manifest. The value of this flag lists the enabled admission controller plugins.

For example, you might see something like this:

```yaml
- --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota
```

This indicates that the following admission controller plugins are enabled:

- NamespaceLifecycle
- LimitRanger
- ServiceAccount
- DefaultStorageClass
- ResourceQuota
