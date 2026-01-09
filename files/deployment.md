# Deployment

<br />

![Deployment Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752941472/fc123505-c176-4c4b-bdb8-b294f6900e92.png)

<br />

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

<br />

### Examples
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
