# Taints and Tolerations 
Node affinity is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite -- they allow a node to repel a set of pods.

Tolerations are applied to pods. Tolerations allow the scheduler to schedule pods with matching taints. Tolerations allow scheduling but don't guarantee scheduling: the scheduler also evaluates other parameters as part of its function.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

Taints and Tolerations do not tell the pod to go to a particular node. Instead, they tell the node to only accept pods with certain tolerations.

Taints and toleration can be used when we are having the requirement of Dedicated Nodes for pods, Nodes with Special Hardware and when we want to evict the pods based on taints. 
  
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

