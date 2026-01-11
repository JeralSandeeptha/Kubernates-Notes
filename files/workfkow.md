# Kubernates Workflow

- Get Cluster Status
```bash
minikube status
```

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

- Log into a specific Node
```bash
minikube ssh -n=<node_name> 
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
