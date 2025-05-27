## Namespace 
- In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc.) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc.).
- For a production cluster, consider not using the default namespace. Instead, make other namespaces and use those.
- A mechanism to attach authorization and policy to a subsection of the cluster.
- Kubernetes starts with four initial namespaces: default,kube-system,kube-public,kube-node-lease
  - **default** - Kubernetes includes this namespace so that you can start using your new cluster without first creating a namespace.
  - **kube-system** - The namespace for objects created by the Kubernetes system.
  - **kube-public** - This namespace is readable by all clients (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.
  - **kube-node-lease** - This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure.
- Avoid creating namespaces with the prefix kube-, since it is reserved for Kubernetes system namespaces.
- By creating namespaces with the same name as public top-level domains, Services in these namespaces can have short DNS names that overlap with public DNS records. Workloads from any namespace performing a DNS lookup without a trailing dot will be redirected to those services, taking precedence over public DNS. To mitigate this, limit privileges for creating namespaces to trusted users. If required, you could additionally configure third-party security controls, such as admission webhooks, to block creating any namespace with the name of public TLDs.

## namespace kubectl commands 
- set and verify the current namespace.
  ```
  kubectl config set-context --current --namespace=kube-system
  kubectl config view --minify | grep namespace:
  ```
- check the namespace scaped and cluster wide resources
  ```
  kubectl api-resources --namespaced=true
  kubectl api-resources --namespaced=false
  ```
K8s Reference Docs:
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/


  
