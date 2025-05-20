# Kube Controller Manager

#### Kube Controller Manager manages various controllers in kubernetes.
- In kubernetes terms, a controller is a process that continuously monitors the state of the components within the system and works towards bringing the whole system to the desired functioning state.
## Controller
   - **Node Controller** - Responsible for monitoring the state of the Nodes and taking necessary actions to keep the application running.   
   - **Replication Controller** - It is responsible for monitoring the status of replicasets and ensuring that the desired number of pods are available at all time within the set.
   - **Other Controllers** - There are many more such controllers available within kubernetes
     - Deployment controller
     - Namespace controller
     - Endpoint controller
     - PV binder controller
     - PV protection controller
     - Stateful controller
     - CronJob Controller 
  ## Kube-Controller-Manager as systemd
  - When you install kube-controller-manager the different controllers will get installed as well.
  - Download the kube-controller-manager binary from the kubernetes release page. For example: You can download kube-controller-manager v1.13.0 here [kube-controller-manager](https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager)
    ```
    $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager
    ```
  - By default all controllers are enabled, but you can choose to enable specific one from **`kube-controller-manager.service`**
    ```
    $ cat /etc/systemd/system/kube-controller-manager.service
    ```
  - In a non-kubeadm setup, you can inspect the options by viewing the **`kube-controller-manager.service`**
  ```
  $ cat /etc/systemd/system/kube-controller-manager.service
  ```
- You can also see the running process and effective options by listing the process on master node and searching for kube-controller-manager.
  ```
  $ ps -aux | grep kube-controller-manager
  ```   
## kube-controller-manager as kubeadm
- kubeadm deploys the kube-controller-manager as a pod in kube-system namespace
  ```
  $ kubectl get pods -n kube-system
  ```  
- You can see the options within the pod located at **`/etc/kubernetes/manifests/kube-controller-manager.yaml`**
  ```
  $ cat /etc/kubernetes/manifests/kube-controller-manager.yaml
  ```
## kube-controller-manager default command-line-tools
- Node Monitor Period = 5s
- Node Monitor Grace Period = 40s
- Pod Eviction Timeout = 5m
  
K8s Reference Docs:
- https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/
- https://kubernetes.io/docs/concepts/overview/components/
   
     
