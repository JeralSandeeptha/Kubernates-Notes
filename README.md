# Kubernates Notes

<br />

### Table of Contents
- [Introduction](#introduction)
- [Architecture](#architecture)
- [Objects](#objects)
- [Pod](#pod)
- [Deployment](#deployment)
- [Service](#service)
- [Environment Variables](#environment-variables)
- [Config Maps](#config-maps)
- [Secrets](#secrets)
- [Demonset](#demonset)
- [Job](#job)
- [Cron Job](#cron-job)
- [Kubernates Workflow](#kubernates-workflow)
- [Multi Container Pods](#multi-container-pods)

<br />
<br />

### Introduction
- Kubernetes` (K8s) is an open-source platform for managing containerized workloads and services.
- Developed by `Google`, now maintained by the `Cloud Native Computing Foundation` (CNCF).

#### ✅ Why Use Kubernetes?
- 🚀 `Automated Deployment & Scaling` – Easily deploy applications and scale them based on demand.
- 🛡️ `Self-Healing` – Automatically restarts failed containers and reschedules them on healthy nodes.
- 🌍 `Service Discovery & Load Balancing` – Exposes containers with DNS names or IPs and distributes traffic.
- 📦 `Configuration Management` – Manage secrets, configuration files, and environment-specific settings.
- 🔁 `Rolling Updates & Rollbacks` – Smoothly update apps without downtime, and roll back if needed.
- ☁️ `Cloud-Agnostic` – Works on-premises or across major cloud providers like AWS, Azure, and GCP.

<br />
<br />

### Architecture

- Kubernates has `Control Plane` which is called as a `Master Nodes` and it also has `Worker Nodes` also known as `Data Planes`
![Kubernates Nodes Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752929063/d7fbbad2-97d1-413f-9456-3b7c14aaba06.png)

![Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1753405389/b1685f44-6e11-4f0b-9729-ed2d63fe73d1.png)

#### Control Plane / Master Node

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

##### kube-apiserver (API Server)
- Entry point for all Kubernetes commands and communication (like kubectl).
- Exposes the REST API.
- All other components talk to the API Server.
- It validates requests and updates the cluster state in etcd.
- When you run `kubectl apply -f deployment.yaml`, it sends that request to the `kube-apiserver`.

##### etcd (Key-Value Store)
- The database of Kubernetes.
- Stores all cluster data and configuration as key-value pairs.
- Highly available and consistent.
- Only the `kube-apiserver` interacts directly with `etcd`.
- Deployment configurations, node statuses, and secrets are stored here.

##### kube-scheduler (Scheduler)
- Assigns Pods to available Nodes based on:
    - Resource requirements (CPU, memory)
    - Affinity/anti-affinity rules
    - Taints and tolerations
    - Node health and availability
    - Picks the best node for a new Pod, then notifies the API server.
- When you create a new Pod, the Scheduler decides which Node it should run on.

##### kube-controller-manager (Controller Manager)
- Runs multiple controllers as separate loops (reconciliations).
- Each controller watches the cluster state and makes changes to move it to the desired state.
- If a Pod crashes and disappears, the ReplicaSet controller recreates it.

| Controller                          | Role                                        |
| ----------------------------------- | ------------------------------------------- |
| Node Controller                     | Detects node failures and reacts            |
| Replication Controller              | Maintains desired pod count                 |
| Endpoints Controller                | Manages endpoint objects in services        |
| Service Account & Token Controllers | Manages default tokens for service accounts |

##### cloud-controller-manager (optional, cloud integration)
- Used when Kubernetes runs in cloud environments like `AWS`, `GCP`, or `Azure`.
- Integrates Kubernetes with cloud provider APIs.
- It handles:
    - Node lifecycle (e.g., shutdown)
    - Load balancer creation
    - Volume management

#### Data Plane / Worker Node

![Data Plane Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752934719/f42743b1-6ff8-43d5-a150-8bc0312ce651.png)

- In Kubernetes architecture, the `Data Plane` is responsible for running the actual application workloads – your containers, pods, services, and their communication.
- It contrasts with the `Control Plane`, which makes decisions about what should run where and manages the cluster state.

##### Kubelet
- An agent running on each worker node.
- Ensures that the containers are running in a Pod as expected.
- Receives instructions from the Control Plane (API Server).
- Reports node and pod status back to the API server.

##### Container Runtime
- This is the low-level software responsible for running containers.
- Kubernetes supports several runtimes like:
    - containerd (default now)
    - CRI-O
    - Docker (deprecated)
- Kubelet uses this to actually start/stop containers.

##### kube-proxy
- Handles networking on each node.
- Maintains network rules on the node to allow Pod-to-Pod, Pod-to-Service, and external access.
- Works with iptables or IPVS to route traffic.

##### Pods and Containers
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

##### How the Data Plane Works (Step-by-Step)
- Let’s say you want to deploy a Pod:
    - You run kubectl apply -f pod.yaml
    - The API Server (Control Plane) receives the request.
    - Scheduler chooses a node for the Pod.
    - Control Plane tells the selected Kubelet to create the Pod.
    - Kubelet asks the Container Runtime to pull the image and start containers.
    - kube-proxy ensures networking is set up for the Pod to communicate.
    - The Pod is now running in the Data Plane.

<br />
<br />

### Objects
| Object      | Description                              |
| ----------- | ---------------------------------------- |
| `Pod`         | Run containers                           |
| `Deployment`  | Manages stateless app replicas           |
| `StatefulSet` | Manages stateful apps (like DBs)         |
| `DaemonSet`   | One pod per node                         |
| `ReplicaSet`  | Ensures a specific number of pods        |
| `Job/CronJob` | Run once or on schedule                  |
| `Service`     | Exposes a set of pods via stable IP/name |
| `ConfigMap`   | Injects configuration data               |
| `Secret`      | Stores sensitive info like passwords     |
| `Volume`      | Persistent storage for Pods              |
| `Ingress`     | Exposes HTTP/S routes from outside       |

<br />
<br />

### Pod

![Pod Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752939295/f8416e8e-712a-4c53-8de8-066626d2688a.png)

#### What is a Pod in Kubernetes?
- A `Pod` is the smallest and simplest unit in the Kubernetes object model that you can create or deploy.
    - It represents one or more containers that are tightly coupled and share resources.
    - Pods are the `atomic unit` of deployment in Kubernetes.
    - They run on worker nodes within the Kubernetes cluster.

#### Why Pods?
- Containers are lightweight, but often need to be grouped:
    - Sometimes you want multiple containers to share storage, network, or lifecycle.
    - Pods allow this grouping of containers that logically belong together.
    - E.g., a web server container + a logging sidecar container inside the same Pod.

#### Pod Basics
- One or more containers (usually Docker or other OCI containers)
- Shared storage volumes (optional)
- Shared network namespace (IP address & port space)
- Specification for how to run the containers

#### Pod Lifecycle
- Pods are created, scheduled on nodes, run, and eventually terminated.
- Pods are ephemeral; when a Pod dies, it’s not resurrected automatically. Instead, controllers like Deployments create new Pods.

#### Pod Features
1. Shared Network Namespace
- Containers in a Pod share the same IP address and port space.
- They can communicate via localhost inside the Pod.
- From outside, Pods are accessed via their IP or through Services.

2. Shared Storage Volumes
- Containers can share mounted volumes for persistent or temporary data.

3. Single IP Address per Pod
- The Pod gets an IP address.
- Containers inside the Pod share this IP.

#### Pod Lifecycle Phases
- `Pending`: Pod is accepted but not running yet.
- `Running`: At least one container is running.
- `Succeeded`: All containers completed successfully.
- `Failed`: At least one container terminated with failure.
- `Unknown`: Status can’t be determined.

#### Examples
- Explain a Pod
```cmd
kubectl explain pod
```

- Create a Pod
```cmd
kubectl run --image=nginx nginx-pod
```

- Dry Run / Only show what happened when run this cmd
```cmd
kubectl run --image=nginx nginx-pod --dry-run=client
```

- Generate manifest file
```cmd
kubectl run --image=nginx nginx-pod --dry-run=client -o yaml > nginx-pod.yaml
```

- Get manifest file of a running pod
```cmd
kubectl get pod nginx-pod -o yaml
```

- Get all Pods
```cmd
kubectl get pods
```

- Get pods with additional details
```cmd
kubectl get pods -o wide
```

- More information about pod
```cmd
kubectl describe pod/nginx-pod
```

- Check logs
```cmd
kubectl logs nginx-pod
```

- Check logs realtime
```cmd
kubectl logs -f nginx-pod
```

- Edit a running pod
```cmd
kubectl edit pod nginx-pod
```

- Go inside of a pod
```cmd
kubectl exec -it nginx-pod -- sh
```

- Delete a pod
```cmd
kubectl delete nginx-pod
```

<br />
<br />

### ReplicaSet
- A ReplicaSet ensures a specified number of pod replicas are running at any time
- ReplicaSet is the one who load balance replica pods

![Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1753432598/7ae318c3-19a9-4675-a5d0-8f2165701ded.png)

### Deployment

![Deployment Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752941472/fc123505-c176-4c4b-bdb8-b294f6900e92.png)

#### What is a Deployment in Kubernetes?
- A Deployment is a Kubernetes resource object that provides declarative updates for Pods and ReplicaSets.
- This is also like a replicaset with rollout fashion. When pod scaliing and still it scale one by one so we don't have any downtimes.
    - It manages the lifecycle of Pods and ensures the desired number of replicas are running.
    - It helps in rolling updates, rollbacks, scaling, and self-healing of your app.
    - You never usually manage Pods directly in production. instead, you create and manage Deployments.

- Why Deployments?
- `Ensure availability`: Keep the specified number of Pods running.
- `Declarative updates`: You declare how you want your app to run, and Deployment handles the rest.
- `Rollback`: If a deployment update goes wrong, you can roll back to a previous stable version.
- `Scaling`: Easily scale up/down replicas.
- `Self-healing`: If a Pod crashes or node fails, Deployment creates replacements automatically.

#### Deployment Components & Concepts
- `ReplicaSet`
    - Deployment manages a ReplicaSet (or multiple during updates).
    - ReplicaSet ensures the specified number of Pod replicas exist.
    - You usually don’t manage ReplicaSets directly; Deployment does it for you.

- `Pods`
    - The actual units running your app (managed indirectly by Deployment).

#### How Deployment Works Internally
- When you create a Deployment, Kubernetes creates a ReplicaSet with the specified number of Pods.
- Deployment monitors ReplicaSet and Pods to maintain the desired state.
- On updates (like image changes), Deployment creates a new ReplicaSet, scales it up, and scales down the old one (rolling update).
- If something goes wrong, Deployment can rollback to the previous ReplicaSet.

#### Examples
- Explain a Deployment
```cmd
kubectl explain deployment
```

- Create a Deployment
```cmd
kubectl create deployment nginx-deployment --image=nginx --replicas=3
```

- Get deployments
```cmd
kubectl get deploy
```

- Create deployment manifest file
```cmd
kubectl create deployment --image=nginx --dry-run=client -o yaml > deployment.yaml
```

- Describe deployments
```cmd
kubectl describe deploy nginx-deployment
```

- Scale replicas
```cmd
kubectl scale deployment nginx-deployment --replicas=2
```

- Edit deployments
```cmd
kubectl edit deploy nginx-deployment
```

- Rollout
```cmd
kubectl rollout deployment/my-deployment
```

- Undo Rollout
```cmd
kubectl rollout undo deployment/my-deployment
```

- Check rollout history
```cmd
kubectl rollout history deployment/my-deployment
```

- Delete deployments
```cmd
kubectl delete deploy nginx-deployment
```

<br />
<br />

### Service

![Service Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752943857/f056d0da-b594-4f38-b052-a555ea0dcd13.png)
![Service External Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752944168/6317e539-88b4-4dcc-ada3-04aef1568430.png)

#### What is a Kubernetes Service?
- A Service in Kubernetes is an abstraction that defines a logical set of Pods and a policy by which to access them.
    - Pods have dynamic IPs and are ephemeral (can die/restart anytime).
    - A Service provides a stable IP and DNS name to reliably access Pods.
    - It enables communication between different parts of your app or with the outside world.

#### Why Services?
- Pods can come and go. Their IP addresses change.
- You don’t want clients to keep track of individual Pods.
- Services act as a load balancer and stable endpoint for a group of Pods.

#### How Services Work
- Services select Pods by label selectors.
- They route traffic to the matching Pods using round-robin load balancing.
- Kubernetes assigns a Cluster IP for internal access.
- Services are backed by iptables/IPVS rules.

#### Types of Services

##### ClusterIP (Default)
- Accessible only within the cluster.
- Ideal for internal communication between services.
- Example use: A backend service used by a frontend inside the same cluster.
```yaml
type: ClusterIP
```

##### NodePort
![Service Works Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752944279/62808821-5622-4abc-817e-4ff40fe95c99.png)
- Exposes the service on each Node's IP at a static port (30000–32767).
- You can access it as: http://<NodeIP>:<NodePort>
- Example use: Quick access for debugging, basic external traffic entry.
```cmd
type: NodePort
```

##### LoadBalancer
- Provisions an external load balancer (via cloud provider).
- Exposes the service externally using the cloud provider’s load balancer.
- When we create a load balancer it will go to the cloud provider and create a loadbalancer in the cloud service.
- Example use: Production-level public access to an app (e.g., AWS ELB, GCP LB).
```cmd
type: LoadBalancer
```

##### ExternalName
- Maps a service to an external DNS name.
- Used to redirect in-cluster traffic to external services.
```cmd
type: ExternalName
externalName: api.external.com
```

| Concept             | Description                                    |
| ------------------- | ---------------------------------------------- |
| **Selector**        | Matches Pods based on labels                   |
| **Port**            | Port exposed by the Service (clients use this) |
| **targetPort**      | Port on the Pod container                      |
| **Cluster IP**      | Internal IP assigned to the Service            |
| **NodePort**        | Port on each Node to access service externally |
| **LoadBalancer IP** | External IP allocated by cloud provider        |

#### Examples
- Expose a deployment
```cmd
kubectl expose deploy nginx-deployment --port=80
```

- Create Service yaml file
```cmd
kubectl get service --dry-run=client -o yaml > service.yaml
```

- Describe a Service
```cmd
kubectl describe service <service_name>
```

- How to use Minikube service
```cmd
minikube service <name_service>
```

- Get endpoints
```cmd
kubectl get endpoints
```

- Delete service
```cmd
kubectl delete service <service_name>
```

<br />
<br />

### Environment Variables
- `Environment variables` in Kubernetes are key-value pairs injected into containers inside Pods.
- They are commonly used to:
    - Configure apps (e.g., database URLs, secrets, API keys)
    - Pass dynamic data (like `Pod IP`, `namespace`, `Node name`)
    - Separate config from code (12-Factor App principle)

#### Types of Environment Variables
- Static Values
```cmd
env:
- name: ENV_VAR_NAME
  value: "some-value"
```

- ConfigMaps
```cmd
envFrom:
- configMapRef:
    name: my-config

env:
- name: APP_PORT
  valueFrom:
    configMapKeyRef:
      name: my-config
      key: port
```

- Secrets
```cmd
envFrom:
- secretRef:
    name: my-secret

env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: password
```

#### Testing 
```cmd
kubectl exec -it my-pod -- printenv         # Show all env vars
kubectl exec -it my-pod -- echo $MY_VAR     # Show specific one
```

#### Summary
| Type            | Use Case                             | Source       |
| --------------- | ------------------------------------ | ------------ |
| Static Value    | Hardcoded string                     | Pod spec     |
| ConfigMap       | Reusable non-sensitive configuration | ConfigMap    |
| Secret          | Sensitive values                     | Secret       |
| Pod Metadata    | Inject Pod/Node/Namespace info       | Downward API |
| Resource Fields | Inject memory/CPU info               | Downward API |

<br />
<br />

### Config Maps
- A ConfigMap is a Kubernetes object used to store non-sensitive key-value pairs. It allows you to decouple configuration from application code
- Use ConfigMaps to store configuration data like environment variables, command-line arguments, port numbers, URLs, and feature flags
- `If we use env inside of pods and deployments we can't reuse them. But with the ConfigMaps we can reuse them`
- We create kubernates object with all the envs

#### Why Use ConfigMaps?
- Keep your app configuration separate from container images
- Easily change app behavior without rebuilding the image
- Make config reusable and manageable across multiple pods
- Configurable at runtime via environment variables, command-line args, or mounted files

#### Basic Example
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: production
  APP_PORT: "8080"
  LOG_LEVEL: debug
```

#### Ways to Use a ConfigMap
- As Environment Variables
```yaml
envFrom:
- configMapRef:
    name: app-config

env:
- name: MODE
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_MODE
```

- As Mounted Files (Volumes)
```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config

volumeMounts:
- name: config-volume
  mountPath: /etc/config
```

#### Examples
- Create a Configmap
```cmd
kubectl create configmap my-config --from-literal=APP_MODE=dev --from-literal=PORT=3000

kubectl create configmap my-config --from-file=app.properties
```

- Get Configmaps
```cmd
kubectl get configmaps

kubectl get configmap app-config -o yaml
```

- Describe Configmaps
```cmd
kubectl describe configmap app-config
```

- Changes won’t reflect in existing Pods automatically
- You must restart or recreate the Pods to pick up new values
- Don't store passwords or secrets — use Secrets instead
- ConfigMaps are not encrypted and are stored in plaintext

#### Example
- Create a ConfigMap
```cmd
kubectl create cm sample-cm --from-literal=firstName=jeral --from-literal=lastName=sandeeptha
```

- Create a manifestfile of ConfigMap
```cmd
kubectl create cm sample-cm --from-literal=firstName=jeral --from-literal=lastName=sandeeptha --dry-run=client -o yaml > configmap.yaml
```

- Get ConfigMaps
```cmd
kubectl get cm
```

- Describe ConfigMaps
```cmd
kubectl describe cm
```

<br />
<br />

### Secrets

#### What is a Secret in Kubernetes?
- A `Secret` is a Kubernetes object used to store sensitive data in your cluster. 
- These include:
    - `Database passwords`
    - `API tokens`
    - `SSH keys`
    - `TLS certificates`
- `Secrets` help keep this information securely managed and accessible, without embedding it directly in your application code or container images.

#### Why Use Secrets (Instead of ConfigMaps or Hardcoding)?
- Secrets offer:
    - Base64 encoding for obfuscation
    - Optional encryption at rest (etcd encryption)
    - Controlled access via RBAC (Role-Based Access Control)
    - Mounted with file permission restrictions
    - Integration with Kubernetes ServiceAccounts and imagePullSecrets
- Unlike `ConfigMaps`, `Secrets` are not meant to be human-readable and are intended to store confidential data.

#### Types of Kubernetes Secrets
| Type                                    | Description                                              |
| --------------------------------------- | -------------------------------------------------------- |
| **Opaque**                              | Default; used for arbitrary user-defined key-value pairs |
| **kubernetes.io/basic-auth**            | For storing username/password credentials                |
| **kubernetes.io/ssh-auth**              | Stores SSH credentials                                   |
| **kubernetes.io/tls**                   | TLS cert/key pair (used with Ingress or HTTPS services)  |
| **kubernetes.io/service-account-token** | Automatically created and managed by Kubernetes for pods |

#### YAML Manifest
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=         # base64 of "admin"
  password: UzNjcjN0IQ==     # base64 of "S3cr3t!"
```

#### Using Secrets in Pods
```yaml
env:
- name: DB_USER
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: username
- name: DB_PASS
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

#### Examples
- Create a Secrets
```bash
kubectl create secret generic config-secret \
  --from-file=./config.txt
```
 
- Apply from a file
```bash
kubectl create secret generic config-secret \
  --from-file=./config.txt
```

<br />
<br />

### Demonset
- A `DaemonSet` ensures that a copy of a specific Pod runs on every node in a Kubernetes cluster (or on selected nodes using label selectors).

- Runs one Pod per node.
- Automatically runs new Pods when:
    - A node is added.
    - A node is removed → its Pod is deleted.
- Can target specific nodes using:
    - Node labels
    - Tolerations and taints
    - Node selectors or affinity rules

- It’s used to automate infrastructure-level tasks like:
    - Logging agents
    - Monitoring agents
    - Node-level storage
    - Network plugins
    - Security tools

| Use Case                        | Example Tool             |
| ------------------------------- | ------------------------ |
| Log collection                  | Fluentd, Logstash        |
| Metrics monitoring              | Prometheus Node Exporter |
| System security scanning        | Falco, Aqua Security     |
| Container runtime monitoring    | cAdvisor                 |
| Node local storage provisioning | Local volume provisioner |

<br />
<br />

### Job
- A `Job` in Kubernetes is used to run a one-time task to completion.
- It ensures that a specified number of Pods successfully terminate, and then the Job is considered complete.

| Feature             | Description                          |
| ------------------- | ------------------------------------ |
| One-shot tasks      | Run once, then exit on success       |
| Pod retry mechanism | Can retry failed Pods (with limits)  |
| Parallelism support | Can run multiple Pods simultaneously |
| Completion tracking | Stops when all Pods complete         |

<br />
<br />

### Cron Job
- A `CronJob` allows you to run Jobs on a time-based schedule (just like UNIX cron)
- It creates a Job at scheduled intervals, and each Job runs independently

| Feature             | Description                             |
| ------------------- | --------------------------------------- |
| Time-based trigger  | Uses cron format (e.g., `*/5 * * * *`)  |
| Job template        | Uses the same spec as a normal Job      |
| Independent runs    | Each execution is its own Job           |
| Missed run handling | Can skip or catch up on missed runs     |
| History tracking    | Keeps logs of completed and failed jobs |

<br />
<br />

### Kubernates Workflow

- Get clusters
```bash
kubectl config get-clusters
```

- Get current cluster
```bash
kubectl config current-context
```

- Get cluster information
```bash
kubectl cluster-info
```

<br />

- Get Nodes
```
kubectl get nodes

kubectl get nodes -o wide
```

- Describe node
```bash
kubectl describe node minikube
```

- Switch to another Node
```bash
kubectl 
```

<br />

- Get Kubernates default resources
```bash
kubectl get all
```

- Delete Kubernates default resources
```bash
kubectl delete all --all
```

<br />

- Create a Pod yaml file
```bash
kubectl run nginx-pod --image=nginx:latest --dry-run=client -o yaml > nginx-pod.yaml
```

- Get a Pods
```bash
kubectl get pods
```

- Get inside of the Pod
```bash
kubectl exec -it nginx-pod -- bash
```

<br />
<br />

### Multi Container Pods

![Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1753529854/ce36d536-d392-4d7c-a98a-0ae915209318.png)

- In Kubernetes, a Pod is the smallest deployable unit and can contain one or more containers. While most Pods run a single container, multi-container Pods allow you to group multiple tightly-coupled containers that:
    - Share the same network namespace (IP, port space)
    - Share storage volumes
    - Are scheduled together on the same node
    - Can communicate using localhost

#### When to Use Multi-Container Pods?

- Init Container Pattern
    - Main container depends on this container
- Sidecar / Helper Pattern
    - An auxiliary container that enhances or adds functionality to the main container
    - Eg: a logging or metrics collector that reads logs from a shared volume.

<br />
<br />
