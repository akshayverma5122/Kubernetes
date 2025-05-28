## Services
- Kubernetes Services enables communication between various components within and outside of the application. There are 3 types of service types in kubernetes.
 
  - **NodePort** - Where the service makes an internal port accessible on a port on the NODE
  - **ClusterIP** - In this case the service creates a **`Virtual IP`** inside the cluster to enable communication between different services such as a set of frontend servers to a set of backend servers.
  - **LoadBalancer** - Where the service provisions a **`loadbalancer`** for our application in supported cloud providers.  
