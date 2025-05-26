## rook with ceph 
- Ceph provides a comprehensive solution to the challenges of distributed storage within Kubernetes by leveraging its unique architecture and features. At the core of Ceph is RADOS (Reliable Autonomic Distributed Object Store), which serves as the foundation for data distribution, replication, and self-healing. The primary components of Ceph’s architecture
  -  **Object Storage Daemons (OSDs)** >>  it handle the storage of data and its replication
  -  **Monitors (MONs)** >> it maintain the overall health and configuration of the cluster.
  -  **Metadata Servers (MDS)** >> it is utilized when file storage is implemented, managing metadata for CephFS, enabling file system semantics within the Ceph architecture.
    

## Rook-ceph installation using helm 
- Clone the rook github repository.
  ```
  git clone --single-branch --branch v1.17.2 https://github.com/rook/rook.git
  ```
- Add and update the helm repo
  ```
  helm repo add rook-release https://charts.rook.io/release
  helm repo update
  helm repo list
  ```
- install ceph-operator
  ```
  helm install --create-namespace --namespace rook-ceph rook-ceph rook-release/rook-ceph -f rook/deploy/charts/rook-ceph/values.yaml
  ```
- install the ceph-cluster
  ```
  helm install --namespace rook-ceph rook-ceph-cluster --set operatorNamespace=rook-ceph rook-release/rook-ceph-cluster -f rook/deploy/charts/rook-ceph-cluster/values.yaml

- ugrade the ceph-cluster with custom values.
  ```
  helm upgrade --reuse-values --values rook/deploy/charts/rook-ceph-cluster/values.yaml rook-ceph-cluster rook-release/rook-ceph-cluster --version v1.17.2 -n rook-ceph
  ```
## Rook-ceph installation using kubectl 
- Clone the rook github repository.
  ```
  git clone --single-branch --branch v1.17.2 https://github.com/rook/rook.git
  ```
- Create the crd,operator and RBAC for ceph cluster
  ```
  cd rook/deploy/examples
  kubectl create -f crds.yaml -f common.yaml -f operator.yaml
  kubectl create -f cluster.yaml
  ```
- Create the cephfilesystem and storage class.
  ```
  cd /rook/deploy/examples
  kubectl create -f filesystem.yaml
  cd /rook/deploy/examples/csi/cephfs
  kubectl create -f storageclass.yaml
  ```
- Check the ceph filesystem and ceph cluster
  ```
  kubectl -n rook-ceph get cephfilesystem
  kubectl -n rook-ceph get cephcluster
  ```
- Create the nodeport service to access the ceph dashboard
  ```
  cd rook/deploy/examples
  kubectl create -f dashboard-external-http.yaml -n rook-ceph
  ```
- Rook creates a default user named admin and generates a secret called rook-ceph-dashboard-password in the namespace where the Rook Ceph cluster is running.
  ```
  kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
  ```
- Login in the ceph dashboard using the admin username and above password.
  
## Rook-ceph uninstallation using kubectl
- First clean up the resources from applications that consume the Rook storage. These commands will clean up the resources from the example application block and file walkthroughs (unmount volumes, delete volume claims, etc).
  ```
  kubectl delete  sc --all  -n rook-ceph
  kubectl delete CephBlockPool  --all -n rook-ceph
  kubectl delete CephFileSystem  --all -n rook-ceph
  ```
- Delete the CephCluster CRD.
  ```
  kubectl -n rook-ceph patch cephcluster rook-ceph --type merge -p '{"spec":{"cleanupPolicy":{"confirmation":"yes-really-destroy-data"}}}'
  kubectl -n rook-ceph delete cephcluster rook-ceph
  kubectl -n rook-ceph get cephcluster
  


https://github.com/rook/rook.git
https://rook.io/docs/rook/latest-release/Getting-Started/quickstart/#tldr
https://rook.io/docs/rook/latest-release/Helm-Charts/ceph-cluster-chart/#release
https://rook.io/docs/rook/latest-release/Storage-Configuration/ceph-teardown/#delete-the-operator-resources

