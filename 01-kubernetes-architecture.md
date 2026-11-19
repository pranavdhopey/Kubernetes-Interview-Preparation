# Kubernetes Architecture Made Simple: Understanding the Control Plane and Worker Nodes

*DevOps Interview Preparation | Question 1*

Kubernetes runs containerized applications across a cluster of machines. To understand its architecture, it helps to separate the components that manage the cluster from those that execute applications.

## 1. How Is a Kubernetes Cluster Structured, and What Are the Responsibilities of Its Core Components?

A Kubernetes cluster has two broad layers: the **control plane**, which makes cluster-wide decisions and maintains cluster state, and the **worker nodes** (sometimes called the data plane), which run application Pods. Both layers coordinate through the Kubernetes API.

### 1.1. Which Components Make Up the Kubernetes Control Plane?

- **API Server:** Entry point for Kubernetes API requests. It authenticates and authorizes requests, validates resources, and exposes cluster state to other components.
- **Scheduler:** Finds a suitable node for a Pod that has not yet been assigned to one. It considers resource requests and placement constraints such as node affinity, Pod affinity, and taints and tolerations.
- **etcd:** A key-value store that holds Kubernetes cluster state, including resource definitions and configuration.
- **Controller Manager:** Runs controllers that compare actual state with desired state. For example, the ReplicaSet controller works to maintain the requested number of Pod replicas.

### 1.2. What Are the Essential Components Running on Kubernetes Worker Nodes?

- **Kubelet:** Watches for Pods assigned to its node and ensures their containers are started and managed.
- **Container runtime:** Runs containers under Kubelet's direction. Common examples are containerd and CRI-O. Docker shim is a historical integration, not a current built-in runtime option.
- **Kube-proxy:** In clusters that use it, maintains networking rules that direct Service traffic toward eligible Pods.

### 1.3. How Do Kubernetes Components Coordinate to Deploy an Application?

Consider a Deployment submitted with `kubectl`:

1. The **API Server** receives and validates the request and records the desired state in **etcd**.
2. The **Deployment and ReplicaSet controllers** work to create the required Pods.
3. The **Scheduler** chooses a worker node for each unscheduled Pod and records the binding through the API Server.
4. The **Kubelet** on the selected node notices its assigned Pod and asks the **container runtime** to start the containers.
5. If a **Service** targets these Pods, Kubernetes maintains the matching endpoints; **kube-proxy**, where used, configures the network rules that direct traffic to them.
6. Controllers continue monitoring resources so that the actual cluster state moves toward the declared state.

The components do not simply pass a request directly down a chain. They observe and update resources through the API Server.

### 1.4. How Can You Describe Kubernetes Architecture Clearly During a DevOps Interview?

> Kubernetes has a control plane and worker nodes. The control plane includes the API Server, Scheduler, etcd, and Controller Manager. Worker nodes run Kubelet, a container runtime, and commonly kube-proxy. When I create a Deployment, the API Server accepts the configuration, controllers create the necessary Pods, the Scheduler selects nodes, and Kubelet uses the runtime to start containers. etcd stores cluster state, and controllers keep working to maintain the desired configuration.

**Quick revision:** Control plane = API Server, Scheduler, etcd, Controller Manager. Worker node = Kubelet, container runtime, kube-proxy.

**Tags:** Kubernetes, DevOps, Kubernetes Architecture, Cloud Native, Interview Preparation
