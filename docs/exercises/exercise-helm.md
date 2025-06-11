# Exercise Working Session 8 - Helm

## Exercise 1

### Description

Use Helm to install Kubernetes objects needed for the open source monitoring solution Prometheus. The easiest way to install Prometheus on top of Kubernetes is with the help of the prometheus-operator Helm chart.

You can search for the kube-prometheus-stack on Artifact Hub. Add the repository to the list of known repositories accessible by Helm with the name prometheus-community.

Update to the latest information about charts from the respective chart repository.

Run the Helm command for listing available Helm charts and their versions. Identify the latest chart version for kube-prometheus-stack.

Install the the chart kube-prometheus-stack. List the installed Helm chart.

List the Service named prometheus-operated created by the Helm chart. The object resides in the default namespace.

Use the kubectl port-forward command to forward the local port 8080 to the port 9090 of the Service. Open a browser and bring up the Prometheus dashboard.

Stop port forwarding and uninstall the Helm chart.


### Solution

Use Helm to install Kubernetes objects needed for the open source monitoring solution Prometheus. The easiest way to install Prometheus on top of Kubernetes is with the help of the prometheus-operator Helm chart.

You can search for the kube-prometheus-stack on Artifact Hub. Add the repository to the list of known repositories accessible by Helm with the name prometheus-community.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Update to the latest information about charts from the respective chart repository.

```bash
helm repo update
```

Run the Helm command for listing available Helm charts and their versions. Identify the latest chart version for kube-prometheus-stack.

```bash
helm search repo prometheus-community/kube-prometheus-stack --versions
```

Install the chart kube-prometheus-stack. List the installed Helm chart.

```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack
helm list
```

List the Service named prometheus-operated created by the Helm chart. The object resides in the default namespace.

```bash
kubectl get svc prometheus-operated
```

Use the kubectl port-forward command to forward the local port 8080 to the port 9090 of the Service. Open a browser and bring up the Prometheus dashboard.

```bash
kubectl port-forward svc/prometheus-operated 8080:9090
```

Open a browser and navigate to `http://localhost:8080` to access the Prometheus dashboard.

Stop port forwarding and uninstall the Helm chart.

```bash
kubectl port-forward --stop svc/prometheus-operated
helm uninstall kube-prometheus-stack
```

Delete the Helm repository.

```bash
helm repo remove prometheus-community
```
