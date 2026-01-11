# Multi Container Pods

![Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1753529854/ce36d536-d392-4d7c-a98a-0ae915209318.png)

- In Kubernetes, a Pod is the smallest deployable unit and can contain one or more containers. While most Pods run a single container, multi-container Pods allow you to group multiple tightly-coupled containers that:
    - Share the same network namespace (IP, port space)
    - Share storage volumes
    - Are scheduled together on the same node
    - Can communicate using localhost

## When to Use Multi-Container Pods?

- Init Container Pattern
    - Main container depends on this container
- Sidecar / Helper Pattern
    - An auxiliary container that enhances or adds functionality to the main container
    - Eg: a logging or metrics collector that reads logs from a shared volume.
