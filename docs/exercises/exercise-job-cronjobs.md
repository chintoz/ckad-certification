# Exercise Working Group Session 3

## Description - Job

Create a Job named random-hash using the container image alpine:3.17.3 that executes the shell command echo $RANDOM | base64 | head -c 20. Configure the Job to execute with two Pods in parallel. The number of completions should be set to 5.

Identify the Pods that executed the shell command. How many Pods do you expect to exist?

Retrieve the generated hash from one of the Pods.

Delete the Job. Will the corresponding Pods continue to exist?

#### Solution

Create base YAML file with dry-run command

```bash

kubectl create job random-hash --image=alpine:3.17.3 -n ckad --dry-run=client -o yaml -- /bin/sh -c "echo $RANDOM | base64 | head -c 20"
```

Modify the output to match the requirements

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: random-hash
  namespace: ckad
spec:
  completions: 5
  parallelism: 2
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - echo 5852 | base64 | head -c 20
        image: alpine:3.17.3
        name: random-hash
        resources: {}
      restartPolicy: Never
status: {}
```

Check jobs

```bash
kubectl get jobs -n ckad
```

Check pods

```bash
kubectl get pods -n ckad
```

Solve the question: "Identify the Pods that executed the shell command. How many Pods do you expect to exist?"

In this case, we expect 5 pods to exist.

Retrieve the generated hash from one of the Pods.

```bash
kubectl logs random-hash-xxxxx -n ckad
```

Delete the Job. 


```bash
kubectl delete job random-hash -n ckad
```


Will the corresponding Pods continue to exist?

No, the pods will be deleted as well. 

```bash
kubectl get pods -n ckad
```


## Description - CronJob

Create a new CronJob named google-ping. When executed, the Job should run a curl command for google.com. Pick an appropriate image. The execution should occur every two minutes.

Watch the creation of the underlying Jobs managed by the CronJob. Check the command-line options of the relevant command or consult the Kubernetes documentation.

Reconfigure the CronJob to retain a history of seven executions.

Reconfigure the CronJob to disallow a new execution if the current execution is still running. Consult the Kubernetes documentation for more information.


#### Solution

```bash
kubectl create cronjob google-ping --image=nginx:1.25.1 --schedule="*/2 * * * *" --dry-run=client -o yaml -- /bin/sh -c "curl google.com"
```

Yaml generated:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: google-ping
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: google-ping
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - curl google.com
            image: nginx:1.25.1
            name: google-ping
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/2 * * * *'
status: {}
```

Reconfigure the CronJob to retain a history of seven executions.

```bash
kubectl patch cronjob google-ping --type='json' -p='[{"op": "replace", "path": "/spec/successfulJobsHistoryLimit", "value": 7}]'
```

An alternative could be editing the yaml file and adding the following line:

```bash
kubectl edit cronjob google-ping
```

Reconfigure the CronJob to disallow a new execution if the current execution is still running. Consult the Kubernetes documentation for more information.

```bash
kubectl patch cronjob google-ping --type='json' -p='[{"op": "replace", "path": "/spec/concurrencyPolicy", "value": "Forbid"}]'
```

or editing the field spec.concurrencyPolicy in the yaml file.

```bash
kubectl edit cronjob google-ping
```