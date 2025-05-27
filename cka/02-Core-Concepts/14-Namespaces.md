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
- **Resource quota** tracks aggregate usage of resources in the Namespace and allows cluster operators to define Hard resource usage limits that a Namespace may consume.
- A **limit range** defines min/max constraints on the amount of resources a single entity can consume in a Namespace.

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
- list the pods along with its images.
  ```
   kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' |sort
  ```
- List all Container images only.
  ```
  kubectl get pods --all-namespaces -o jsonpath="{.items[*].spec['initContainers', 'containers'][*].image}" |tr -s '[[:space:]]' '\n' |sort |uniq -c
  ```
#### Configure Memory and CPU Quotas for a Namespace
- Create the quota-mem-cpu-example. Generate the pod yaml with cpu and memory requests & limits
  ```
  kubectl -n quota-mem-cpu-example run  web1 --image=nginx --port=80 --image-pull-policy=Always --dry-run=server -o yaml  > pod1.yaml
  kubectl set resources -f pod1.yaml --requests=cpu=400m,memory=600Mi --limits=cpu=800m,memory=800Mi --local -o yaml > finalpod2.yaml
  ```
- Create a ResourceQuota
  ```
  kubectl create quota mem-cpu-demo --hard=requests.cpu=1,requests.memory=1Gi,limits.cpu=2,limits.memory=2Gi,pods=2,services=1 -n quota-mem-cpu-example
  ```
- Create the pods and it will allow the pod creation request.
  ```
  kubectl create -f finalpod2.yaml
  ```
- Modify the name, request and limit of cpu & memory in  finalpod2.yaml.
  ```
  name: web2
  resources:
      limits:
        cpu: 800m
        memory: 1Gi
      requests:
        cpu: 400m
        memory: 700Mi
   ```
- Create the pods and this will give the below error because it is exceeding the cpu & memory usage.
  ```
  Error from server (Forbidden): error when creating "finalpod.yaml": pods "web2" is forbidden: exceeded quota: mem-cpu-demo, 
  requested: requests.memory=700Mi, used: requests.memory=600Mi, limited: requests.memory=1Gi
  ```
K8s Reference Docs:
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/
