# Pods
- Kubernetes doesn't deploy containers directly on the worker node.
- Pod will have a one-to-one relationship with containers running your application.
- **Multi-Container PODs** - A single pod can have multiple containers except for the fact that they are usually not multiple containers of the **`same kind`**.
- Containers should only be scheduled together in a single Pod if they are tightly coupled and need to share resources such as disk.
- A Pod is a group of one or more application containers (such as Docker) and includes shared storage (volumes), IP address and information about how to run them. Those resources include:
  
   1. Shared storage, as Volumes,
   2. Networking, as a unique cluster IP address, 
   3. Information about how to run each container, such as the container image version or specific ports to use
   4. cgroups 
- Pod can contain init containers that run during Pod startup. You can also inject ephemeral containers for debugging a running Pod
- Restarting a container in a Pod should not be confused with restarting a Pod. A Pod is not a process, but an environment for running container(s). A Pod persists until it is deleted.
- You should set the .spec.os.name field to either windows or linux to indicate the OS on which you want the pod to run. These two are the only operating systems supported for now by Kubernetes.
- Controllers for workload resources create Pods from a pod template and manage those Pods on your behalf.
  
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
- https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/


