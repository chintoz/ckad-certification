# CKAD Certification

## Resources to be used:

- Book:  https://www.oreilly.com/library/view/certified-kubernetes-application/9781098152857/
- Course: https://www.oreilly.com/library/view/certified-kubernetes-application/9780135349700/
- Course: https://www.udemy.com/course/certified-kubernetes-application-developer/

## My notes

There is a Github Pages with all the information gathered through the resources: https://chintoz.github.io/ckad-certification/

## Table of contents

- Unit 1 - Core Concepts
- Unit 2 - Configuration
- Unit 3 - Multi-Container Pods
- Unit 4 - Observability
- Unit 5 - Pod Design
- Unit 6 - Services & Networking
- Unit 7 - State Persistence
- Unit 8 - Security
- Unit 9 - Helm Fundamentals

## Exam Competencies

* Application Design and Build 20% 
  * Define, build and modify container images
  * Choose and use the right workload resource (Deployment, DaemonSet, CronJob, etc.)
  * Understand multi-container Pod design patterns (e.g. sidecar, init and others)
  * Utilize persistent and ephemeral volumes 
* Application Deployment 20%
  * Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)
  * Understand Deployments and how to perform rolling updates
  * Use the Helm package manager to deploy existing packages
  * Kustomize
* Application Observability and Maintenance 15%
  * Understand API deprecations
  * Implement probes and health checks
  * Use built-in CLI tools to monitor Kubernetes applications
  * Utilize container logs
  * Debugging in Kubernetes
* Application Environment, Configuration and Security 25%
  * Discover and use resources that extend Kubernetes (CRD, Operators)
  * Understand authentication, authorization and admission control
  * Understand requests, limits, quotas
  * Understand ConfigMaps
  * Define resource requirements
  * Create & consume Secrets
  * Understand ServiceAccounts
  * Understand Application Security (SecurityContexts, Capabilities, etc.)
* Services and Networking 20%
  * Demonstrate basic understanding of NetworkPolicies 
  * Provide and troubleshoot access to applications via services 
  * Use Ingress rules to expose applications