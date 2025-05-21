## Deployments



#### Deployment using kubectl
- Once the file is ready, create the deployment using deployment definition file
  ```
  kubectl create -f deployment-definition.yaml
  ```
- To see the created deployment
  ```
  kubectl get deployment
  ```
- The deployment automatically creates a **`ReplicaSet`**. To see the replicasets
  ```
  kubectl get replicaset
  ```
- The replicasets ultimately creates **`PODs`**. To see the PODs
  ```
  kubectl get pods
  ``` 
- To see the all objects at once
  ```
  kubectl get all
  ```
 
  
K8s Reference Docs:
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/
- https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/
- https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/
