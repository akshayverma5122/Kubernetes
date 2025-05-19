## ReplicaSets
- A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. Usually, you define a Deployment and let that Deployment manage ReplicaSets automatically.
- When a ReplicaSet needs to create new Pods, it uses its Pod template.
- A ReplicaSet identifies new Pods to acquire by using its selector. If there is a Pod that has no OwnerReference or the OwnerReference is not a Controller and it matches a ReplicaSet's selector, it will be immediately acquired by said ReplicaSet.
- if we create the pods with same label as replicaset is having then replicaset will aquire this pod even if it is not created by the replicaset controller. this replicaset behaviour is similar if i create the pods before replicaset creation or after replicaset creation. 

## Replication Controller 
- Controllers are brain behind kubernetes

## Difference between ReplicaSet and Replication Controller
- **`Replication Controller`** is the older technology that is being replaced by a **`ReplicaSet`**.
- **`ReplicaSet`** is the new way to setup replication.
- **ReplicaSet** requires a selector definition when compare to **Replication Controller**.

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
## Creating a ReplicaSet
     
  - To Create the replicaset
    ```
    $ kubectl create -f replicaset-definition.yaml
    ```
  - To list all the replicaset
    ```
    $ kubectl get replicaset
    ```
  - To list pods that are launch by the replicaset
    ```
    $ kubectl get pods
    ``` 
    
## Labels and Selectors
#### What is the deal with Labels and Selectors? Why do we label pods and objects in kubernetes?
  
## How to scale replicaset
- There are multiple ways to scale replicaset
  - First way is to update the number of replicas in the replicaset-definition.yaml definition file. E.g replicas: 6 and then 

  ```
  $ kubectl apply -f replicaset-definition.yaml
  ```
  - Second way is to use **`kubectl scale`** command.
  ```
  $ kubectl scale --replicas=6 -f replicaset-definition.yaml
  ```
  - Third way is to use **`kubectl scale`** command with type and name
  ```
  $ kubectl scale --replicas=6 replicaset myapp-replicaset
  ```


#### K8s Reference Docs:
- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
- https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/
