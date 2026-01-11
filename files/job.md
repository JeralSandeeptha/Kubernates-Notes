# Job
- A `Job` in Kubernetes is used to run a one-time task to completion.
- It ensures that a specified number of Pods successfully terminate, and then the Job is considered complete.

| Feature             | Description                          |
| ------------------- | ------------------------------------ |
| One-shot tasks      | Run once, then exit on success       |
| Pod retry mechanism | Can retry failed Pods (with limits)  |
| Parallelism support | Can run multiple Pods simultaneously |
| Completion tracking | Stops when all Pods complete         |
