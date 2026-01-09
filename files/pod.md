# Pod

<br />

![Pod Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752939295/f8416e8e-712a-4c53-8de8-066626d2688a.png)

<br />

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

<br />

## Pod Lifecycle
- Pods are created, scheduled on nodes, run, and eventually terminated.
- Pods are ephemeral; when a Pod dies, it’s not resurrected automatically. Instead, controllers like Deployments create new Pods.

<br />

## Pod Features
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

<br />

### Examples
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