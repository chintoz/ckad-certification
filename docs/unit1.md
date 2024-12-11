# Unit 1 - Core Concepts

**Node** is a machine (physical or virtual) on which Kubernetes is installed.  A node is a worker machine in Kubernetes. It was known as minions in the past. Therefore, name is interchangeable.  Each node has the services necessary to run pods and is managed by the control plane.  The services on a node include the container runtime, kubelet and kube-proxy.

If one node fails our application will fail. To prevent this, we could have multiple nodes. Then a **cluster** is a set of nodes grouped together. If one node fails the application will still be running on the other nodes.