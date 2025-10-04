# Exercise Working Session 16 — CRD

## Exercise 1

### Description

You decide to manage a [MongoDB](https://www.mongodb.com/) installation in Kubernetes with the help of the [official community operator](https://github.com/mongodb/mongodb-kubernetes-operator). This operator provides a CRD. After installing the operator, you will interact with the CRD.

Navigate to the directory app-a/ch16/mongodb-operator of the checked-out GitHub repository [bmuschko/ckad-study-guide](https://github.com/bmuschko/ckad-study-guide). Install the operator using the following command: kubectl apply -f mongodbcommunity.mongodb.com_mongodb​community.yaml.

List all CRDs using the appropriate kubectl command. Can you identify the CRD that was installed by the installation procedure?

Inspect the schema of the CRD. What are the type and property names of this CRD?

### Solution


Install the operator:

```bash
kubectl apply -f https://raw.githubusercontent.com/mongodb/mongodb-kubernetes-operator/v0.13.1/config/crd/bases/mongodbcommunity.mongodb.com_mongodbcommunity.yaml
```

List all CRDs:

```bash
kubectl get crds
```

You should see an entry for `mongodbcommunity.mongodb.com`.
Inspect the schema of the CRD:

```bash
kubectl get crd mongodbcommunity.mongodb.com -o yaml
```

You should see that the CRD has a `spec` property of type `object` with several properties, including `members`, `type`, `version`, and `security`.

Destroy all the resources created during the exercise:

```bash
kubectl delete -f https://raw.githubusercontent.com/mongodb/mongodb-kubernetes-operator/v0.13.1/config/crd/bases/mongodbcommunity.mongodb.com_mongodbcommunity.yaml
```

## Exercise 2

### Description

As an application developer, you may want to install Kubernetes functionality that extends the platform using the Kubernetes operator pattern. The objective of this exercise is to familiarize yourself with creating and managing CRDs. You will not need to write a controller.

Create a CRD resource named backup.example.com with the following specifications:

```
Group: example.com
Version: v1
Kind: Backup
Singular: backup
Plural: backups
```

Properties of type string: `cronExpression, podName, path`

Retrieve the details for the Backup custom resource created in the previous step.

Create a custom object named nginx-backup for the CRD. Provide the following property values:

```
cronExpression: 0 0 * * *
podName: nginx
path: /usr/local/nginx
```

Retrieve the details for the nginx-backup object created in the previous step.


### Solution

Create a CRD resource named backup.example.com:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backups.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronExpression:
                  type: string
                podName:
                  type: string
                path:
                  type: string
  scope: Namespaced
  names:
    plural: backups
    singular: backup
    kind: Backup
    shortNames:
      - bk
```

Create definition can be applied using the following command:

```bash
kubectl apply -f backup-crd.yaml
```

Create a custom object named nginx-backup:

```yaml
apiVersion: example.com/v1
kind: Backup
metadata:
  name: nginx-backup
  namespace: ckad
spec:
  cronExpression: "0 0 * * *"
  podName: nginx
  path: /usr/local/nginx
```

Create the custom object using the following command:

```bash
kubectl apply -f nginx-backup.yaml
```

Retrieve the details for the nginx-backup object:

```bash
kubectl get backup nginx-backup -n ckad -o yaml
```
