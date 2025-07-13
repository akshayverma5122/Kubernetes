## Assigning Pods to Nodes
- if we create the pods then kube-scheduler automatically assign the pods to node while considering some rule like memory, disk and cpu pressure but if we want to decide to place the pods as per our bussiness and technical requirement then you can use any of the following methods to choose where Kube-scheduler will place the pods:
  
  - nodeSelector field matching against node labels
  - Affinity and anti-affinity
  - nodeName field
  - Pod topology spread constraints

## Node Selectors
- we need to assign the label to node and add new property called nodeSelector to the spec section of pod manifest file and specify the label. scheduler uses these labels to match and identify the right node to place the pods on.
- 
- label the node
  ```
  kubectl label nodes node-1 size=Large
  ```
- Add the nodeSelector to the spec section.
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   nodeSelector:
    size: Large
  ```
## Node Affinity
 
#### The primary feature of Node Affinity is to ensure that the pods are hosted on particular nodes.
- With **`Node Selectors`** we cannot provide the advance expressions.
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   nodeSelector:
    size: Large
  ```

  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: In
              values: 
              - Large
              - Medium
  ```

  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: NotIn
              values: 
              - Small
  ```

  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: Exists
  ```  

## Node Affinity Types
- Available
  - requiredDuringSchedulingIgnoredDuringExecution
  - preferredDuringSchedulingIgnoredDuringExecution
- Planned
  - requiredDuringSchedulingRequiredDuringExecution
  - preferredDuringSchedulingRequiredDuringExecution
  
#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/
- https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector
- https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/






