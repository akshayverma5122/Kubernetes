## Assigning Pods to Nodes
- if we create the pods then kube-scheduler automatically assign the pods to node while considering some rule like memory, disk and cpu pressure but if we want to decide to place the pods as per our bussiness and technical requirement then you can use any of the following methods to choose where Kube-scheduler will place the pods:
  
  - nodeSelector field matching against node labels
  - Affinity and anti-affinity
  - nodeName field
  - Pod topology spread constraints

## Node Selectors
- we need to assign the label to node and add new property called nodeSelector to the spec section of pod manifest file and specify the label. scheduler uses these labels to match and identify the right node to place the pods on.
  
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
## nodeName

## Affinity and anti-affinity
- although nodeSelector provides the way to place the pods in the particular nodes but it will not provide the advance expression which includes the soft and hard expression. For that we need to use the affinity scheduling feacture of kubernetes.
- Affinity and anti-affinity provide the more expressive with hard and soft or preferred language to place the pods in particular nodes. it also provide the option to schedule the pods with pods labels instead of node label (pods which are already running with specific label) to colocate the pods with same process.
- The affinity feature consists of two types of affinity:
  
  - Node affinity functions like the nodeSelector field but is more expressive and allows you to specify soft rules.
  - Inter-pod affinity/anti-affinity allows you to constrain Pods against labels on other Pods.
    
## Node affinity

- Node affinity is conceptually similar to nodeSelector, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity -
  
  - **requiredDuringSchedulingIgnoredDuringExecution**: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
  - **preferredDuringSchedulingIgnoredDuringExecution**: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

```
apiVersion: v1
kind: Pod
metadata:
  name: with-affinity-preferred-weight
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: label-1
            operator: In
            values:
            - key-1
      - weight: 50
        preference:
          matchExpressions:
          - key: label-2
            operator: In
            values:
            - key-2
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:3.8
```

**Note:**
If you specify both nodeSelector and nodeAffinity, both must be satisfied for the Pod to be scheduled onto a node.

If you specify multiple terms in nodeSelectorTerms associated with nodeAffinity types, then the Pod can be scheduled onto a node if one of the specified terms can be satisfied (terms are ORed).

If you specify multiple expressions in a single matchExpressions field associated with a term in nodeSelectorTerms, then the Pod can be scheduled onto a node only if all the expressions are satisfied (expressions are ANDed).

## Inter-pod affinity and anti-affinity 

  
#### K8s Reference Docs
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
- https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/
- https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/
- https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/  
- https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/







