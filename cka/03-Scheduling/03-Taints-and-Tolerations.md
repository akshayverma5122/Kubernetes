# Taints and Tolerations 
Node affinity is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite -- they allow a node to repel a set of pods.

Tolerations are applied to pods. Tolerations allow the scheduler to schedule pods with matching taints. Tolerations allow scheduling but don't guarantee scheduling: the scheduler also evaluates other parameters as part of its function.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

Taints and Tolerations do not tell the pod to go to a particular node. Instead, they tell the node to only accept pods with certain tolerations.

Taints and toleration can be used when we are having the requirement of Dedicated Nodes for pods, Nodes with Special Hardware and when we want to evict the pods based on taints. 

You can specify tolerationSeconds for a Pod to define how long that Pod stays bound to a failing or unresponsive Node or tainted node with noexecute effect. 

if we taint the node with noexecute then node will evict the pods except daemonset. 



- if we cordon the pod then new pod is not going to schedule because it will add the below parameter in the node. 
```
Taints:             node.kubernetes.io/unschedulable:NoSchedule
Unschedulable:      true
```
- and if uncordon the pods then it will add the below parameter - 
```
Taints:             <none>
Unschedulable:      false
```
- and if we drain the node then it will evict the pod and add the below parameter in node -
```
kubectl drain <nodename> --delete-emptydir-data  --ignore-daemonsets
```
```
Taints:             node.kubernetes.io/unschedulable:NoSchedule
Unschedulable:      true
```

  
## Taints
- Assign taint to node
  ```
  kubectl taint nodes <node-name> key=value:<taint-effect>
  ```
- To see this taint, run the below command
  ```
  kubectl describe node <node-name> |grep Taint
  ```  
- Remove taint from node
  ```
  kubectl taint nodes <node-name> key=value:<taint-effect>-
  ``` 
- The taint effect defines what would happen to the pods if they do not tolerate the taint. There are 3 taint effects
  - **`NoSchedule`** - No new Pods will be scheduled on the tainted node unless they have a matching toleration. Pods currently running on the node are not evicted.
  - **`PreferNoSchedule`** - PreferNoSchedule is a "preference" or "soft" version of NoSchedule. The control plane will try to avoid placing a Pod that does not tolerate the taint on the node, but it is not guaranteed.
  - **`NoExecute`** - This affects pods that are already running on the node as follows:
       - Pods that do not tolerate the taint are evicted immediately
       - Pods that tolerate the taint without specifying tolerationSeconds in their toleration specification remain bound forever
       - Pods that tolerate the taint with a specified tolerationSeconds remain bound for the specified amount of time. After that time elapses, the node lifecycle controller evicts the Pods from the node.

## Taint based Evictions
The node controller automatically taints a Node when certain conditions are true - 
```
node.kubernetes.io/not-ready
node.kubernetes.io/unreachable
node.kubernetes.io/memory-pressure
node.kubernetes.io/disk-pressure
node.kubernetes.io/pid-pressure
node.kubernetes.io/network-unavailable
node.kubernetes.io/unschedulable
```
## Tolerations
   - Tolerations are added to pods by adding a **`tolerations`** section in pod definition.
     ```
     apiVersion: v1
     kind: Pod
     metadata:
      name: myapp-pod
     spec:
      containers:
      - name: nginx-container
        image: nginx
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"
     ```     
#### K8s Reference Docs
- https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

