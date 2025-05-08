# Pods
- Kubernetes doesn't deploy containers directly on the worker node.
- Pod will have a one-to-one relationship with containers running your application.
- **Multi-Container PODs** - A single pod can have multiple containers except for the fact that they are usually not multiple containers of the **`same kind`**.
  
## Deploy Pods using imperative commands 
- To deploy a docker container by creating a POD.
  ```
  $ kubectl run nginx --image nginx
  ```

- To get the list of pods
  ```
  $ kubectl get pods
  ```
K8s Reference Docs:
- https://kubernetes.io/docs/concepts/workloads/pods/pod/
- https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/


