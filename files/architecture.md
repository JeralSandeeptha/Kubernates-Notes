# Architecture

- Kubernates has `Control Plane` which is called as a `Master Nodes` and it also has `Worker Nodes` also known as `Data Planes`
![Kubernates Nodes Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752929063/d7fbbad2-97d1-413f-9456-3b7c14aaba06.png)

![Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1753405389/b1685f44-6e11-4f0b-9729-ed2d63fe73d1.png)

## Control Plane / Master Node

![Control Plane Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752934631/15e2be66-8dc7-47d0-ae3c-4ee44206f248.png)
- The `Control Plane` is the central management layer of Kubernetes. It makes global decisions (like scheduling), detects and responds to cluster events, and maintains the desired state of the cluster.

- In simple terms: `If the worker nodes are the body, then the control plane is the brain of Kubernetes`

- Control Plane Workflow (Simplified)
    - You create a Deployment (via kubectl or YAML)
    - `kube-apiserver` validates and stores it in etcd
    - `kube-scheduler` finds a suitable Node for the Pod
    - `kube-controller-manager` ensures the Pod is running as expected
    - If you're on a cloud provider, `cloud-controller-manager` manages things like networking

| Component                    | Description                                   | Type     |
| ---------------------------- | --------------------------------------------- | -------- |
| **kube-apiserver**           | Handles all REST requests, gateway to cluster | Core     |
| **etcd**                     | Stores persistent cluster data                | Database |
| **kube-scheduler**           | Schedules Pods to nodes                       | Core     |
| **kube-controller-manager**  | Runs controllers to maintain cluster state    | Core     |
| **cloud-controller-manager** | Integrates Kubernetes with cloud platforms    | Optional |

### kube-apiserver (API Server)
- Entry point for all Kubernetes commands and communication (like kubectl).
- Exposes the REST API.
- All other components talk to the API Server.
- It validates requests and updates the cluster state in etcd.
- When you run `kubectl apply -f deployment.yaml`, it sends that request to the `kube-apiserver`.

### etcd (Key-Value Store)
- The database of Kubernetes.
- Stores all cluster data and configuration as key-value pairs.
- Highly available and consistent.
- Only the `kube-apiserver` interacts directly with `etcd`.
- Deployment configurations, node statuses, and secrets are stored here.

### kube-scheduler (Scheduler)
- Assigns Pods to available Nodes based on:
    - Resource requirements (CPU, memory)
    - Affinity/anti-affinity rules
    - Taints and tolerations
    - Node health and availability
    - Picks the best node for a new Pod, then notifies the API server.
- When you create a new Pod, the Scheduler decides which Node it should run on.

### kube-controller-manager (Controller Manager)
- Runs multiple controllers as separate loops (reconciliations).
- Each controller watches the cluster state and makes changes to move it to the desired state.
- If a Pod crashes and disappears, the ReplicaSet controller recreates it.

| Controller                          | Role                                        |
| ----------------------------------- | ------------------------------------------- |
| Node Controller                     | Detects node failures and reacts            |
| Replication Controller              | Maintains desired pod count                 |
| Endpoints Controller                | Manages endpoint objects in services        |
| Service Account & Token Controllers | Manages default tokens for service accounts |

### cloud-controller-manager (optional, cloud integration)
- Used when Kubernetes runs in cloud environments like `AWS`, `GCP`, or `Azure`.
- Integrates Kubernetes with cloud provider APIs.
- It handles:
    - Node lifecycle (e.g., shutdown)
    - Load balancer creation
    - Volume management

## Data Plane / Worker Node

![Data Plane Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752934719/f42743b1-6ff8-43d5-a150-8bc0312ce651.png)

- In Kubernetes architecture, the `Data Plane` is responsible for running the actual application workloads – your containers, pods, services, and their communication.
- It contrasts with the `Control Plane`, which makes decisions about what should run where and manages the cluster state.

### Kubelet
- An agent running on each worker node.
- Ensures that the containers are running in a Pod as expected.
- Receives instructions from the Control Plane (API Server).
- Reports node and pod status back to the API server.

### Container Runtime
- This is the low-level software responsible for running containers.
- Kubernetes supports several runtimes like:
    - containerd (default now)
    - CRI-O
    - Docker (deprecated)
- Kubelet uses this to actually start/stop containers.

### kube-proxy
- Handles networking on each node.
- Maintains network rules on the node to allow Pod-to-Pod, Pod-to-Service, and external access.
- Works with iptables or IPVS to route traffic.

### Pods and Containers
- The actual workloads (apps, APIs, microservices).
- Pods are the smallest deployable unit in Kubernetes.
- A pod wraps one or more containers with shared network/storage.

| Component                   | Location    | Role                                                                               |
| --------------------------- | ----------- | ---------------------------------------------------------------------------------- |
| **Kubelet**                 | Worker Node | Agent that ensures containers are running as instructed by the control plane.      |
| **Container Runtime**       | Worker Node | Executes containers (e.g., containerd, CRI-O). Pulls images and manages lifecycle. |
| **kube-proxy**              | Worker Node | Manages networking rules for Pod-to-Pod and external communication.                |
| **Pods**                    | Worker Node | Smallest deployable unit; runs one or more containers.                             |
| **Containers**              | Inside Pods | The actual applications or services being run.                                     |
| **CNI Plugin** *(optional)* | Worker Node | Manages network connectivity between Pods (e.g., Calico, Flannel).                 |

### How the Data Plane Works (Step-by-Step)
- Let’s say you want to deploy a Pod:
    - You run kubectl apply -f pod.yaml
    - The API Server (Control Plane) receives the request.
    - Scheduler chooses a node for the Pod.
    - Control Plane tells the selected Kubelet to create the Pod.
    - Kubelet asks the Container Runtime to pull the image and start containers.
    - kube-proxy ensures networking is set up for the Pod to communicate.
    - The Pod is now running in the Data Plane.