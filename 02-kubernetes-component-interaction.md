# Behind the Scenes of Kubernetes: What Happens When You Create a Pod?

*DevOps Interview Preparation | Question 2*

Running a `kubectl` command may take only a moment, but several Kubernetes components participate in turning a Pod specification into a running workload.

## 2. How Does Kubernetes Process a Pod Creation Request Across Its Different Components?

Kubernetes coordinates Pod creation through the **API Server**, **Scheduler**, **Kubelet**, **container runtime**, and **etcd**. Controllers are also important when Pods belong to a managed workload, such as a Deployment.

### 2.1. What Happens Internally After Executing `kubectl apply`?

Suppose you have a valid Pod manifest named `pod.yaml`:

```bash
kubectl apply -f pod.yaml
```

The API Server accepts the request, Kubernetes records the resource, the Scheduler assigns a node, and Kubelet starts the Pod's containers through the container runtime. These steps are coordinated through API objects rather than direct handoffs between components.

### 2.2. How Does the Kubernetes API Server Validate and Process a Pod Creation Request?

The API Server checks the caller's identity (**authentication**), permissions (**authorization**), and the resource's validity. Once accepted, the Pod definition becomes part of the cluster's stored desired state in **etcd**.

Other Kubernetes components watch the API Server for changes and respond to the resources they are responsible for.

### 2.3. What Factors Does the Kubernetes Scheduler Consider Before Assigning a Pod to a Node?

The Scheduler checks which nodes can accommodate the Pod and then chooses a suitable one. Important considerations include:

- Node selectors and node affinity
- Pod affinity or anti-affinity
- Node taints and Pod tolerations
- Available resources relative to the Pod's requests

The Scheduler records the node assignment through the API Server.

### 2.4. How Do Kubelet and the Container Runtime Work Together to Start a Pod?

Kubelet on the selected node detects the assigned Pod and requests container creation from the container runtime. The runtime starts the application containers, while Kubelet monitors their state and reports updates through the API Server.

### 2.5. How Does Kubernetes Store and Maintain the State of Running Workloads?

**etcd** stores Kubernetes resource and cluster-state information. It is updated as part of API operations, not only after a Pod is running.

For controller-managed workloads, controllers keep reconciling the desired and actual state. For instance, a ReplicaSet controller can create a replacement when one of its managed Pods disappears. A standalone Pod does not automatically receive ReplicaSet replacement behavior.

### 2.6. How Can You Explain the Complete Pod Creation Process in a Technical Interview?

> When I apply a Pod manifest, the API Server authenticates, authorizes, and validates the request, then stores the resource state in etcd. The Scheduler evaluates eligible nodes and assigns one. Kubelet on that node sees the assignment and uses the container runtime to start the containers. Kubelet reports status through the API Server. If the Pod belongs to a managed workload, controllers help maintain the desired number of Pods.

**Quick revision:** `kubectl` → API Server → stored Pod → Scheduler assigns node → Kubelet → container runtime. etcd stores state; controllers reconcile managed workloads.

**Tags:** Kubernetes, DevOps, Pod Lifecycle, Kubernetes Scheduler, Interview Preparation
