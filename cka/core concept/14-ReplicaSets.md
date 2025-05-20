### ReplicaSets
- A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. Usually, you define a Deployment and let that Deployment manage ReplicaSets automatically.
- When a ReplicaSet needs to create new Pods, it uses its Pod template.
- A ReplicaSet identifies new Pods to acquire by using its selector. If there is a Pod that has no OwnerReference or the OwnerReference is not a Controller and it matches a ReplicaSet's selector, it will be immediately acquired by said ReplicaSet.
- if we create the pods with same label as replicaset is having then replicaset will aquire this pod even if it is not created by the replicaset controller. this replicaset behaviour is similar if i create the pods before replicaset creation or after replicaset creation.
- You can remove Pods from a ReplicaSet by changing their labels. This technique may be used to remove Pods from service for debugging, data recovery, etc.
- **Pod deletion cost** - Using the **controller.kubernetes.io/pod-deletion-cost** annotation, users can set a preference regarding which pods to remove first when downscaling a ReplicaSet.The annotation should be set on the pod, the range is [-2147483648, 2147483647]. It represents the cost of deleting a pod compared to other pods belonging to the same ReplicaSet. Pods with lower deletion cost are preferred to be deleted before pods with higher deletion cost. The implicit value for this annotation for pods that don't set it is 0; negative values are permitted. Invalid values will be rejected by the API server.
  
### Difference between ReplicaSet and Replication Controller
- **`Replication Controller`** is the older technology that is being replaced by a **`ReplicaSet`**.
- **`ReplicaSet`** is the new way to setup replication.
- **`ReplicaSet`** requires a selector definition when compare to **`Replication Controller`**.
  
## Creating a ReplicaSet
     
  - To Create the replicaset
    ```
     kubectl create -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/core%20concept/manifest/replicaset-definition.yaml
    ```
  - To list all the replicaset
    ```
     kubectl get replicaset
    ```
  - To list pods that are launch by the replicaset
    ```
     kubectl get pods
    ```
### How to scale replicaset
- There are multiple ways to scale replicaset
  - First way is to update the number of replicas in the replicaset-definition.yaml definition file. E.g replicas: 6 and then 

  ```
   kubectl apply -f replicaset-definition.yaml
  ```
  - Second way is to use **`kubectl scale`** command.
  ```
   kubectl scale --replicas=6 -f replicaset-definition.yaml
  ```
  - Third way is to use **`kubectl scale`** command with type and name
  ```
   kubectl scale --replicas=6 replicaset myapp-replicaset
  ```
  - create the Horizontal Pod Autoscaler for replicaset
  ```
  kubectl autoscale rs frontend --max=10 --min=3 --cpu-percent=50

## ReplicaSet using REST API
- Execute the below command to access the apis in localhost
  ```
  kubect proxy --port=8080
  ```
- list the all replicasets
  ```
  curl -X GET localhost:8080/apis/apps/v1/namespaces/default/replicasets
  ```
- list the frontend replicasets
  ```
  curl -X GET localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend
  ```
- delete the frontend replicasets and its pods 
  ```
  curl -X DELETE  'localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend'   -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}'   -H "Content-Type: application/json"

  kubectl delete rs frontend
  ```
- delete the frontend replicasets only.
  ```
   curl -X DELETE  'localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend' -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' -H "Content-Type: application/json"

   kubectl delete rs frontend --cascade=orphan
  ```    
## Replication Controller 
- Controllers are brain behind kubernetes
  
## Creating a Replication Controller
  - To Create the replication controller
    ```
    $ kubectl create -f rc-definition.yaml
    ```
  - To list all the replication controllers
    ```
    $ kubectl get replicationcontroller
    ```
  - To list pods that are launch by the replication controller
    ```
    $ kubectl get pods
    ```    
  


## Labels and Selectors
#### What is the deal with Labels and Selectors? Why do we label pods and objects in kubernetes?
  


 

#### K8s Reference Docs:
- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
- https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/
- https://kubernetes.io/docs/concepts/architecture/garbage-collection/
