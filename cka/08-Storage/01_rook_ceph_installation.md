## Rook-ceph installation
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

- disable the cephBlockPools and ceph-objectstore
  helm upgrade --reuse-values rook-ceph rook-release/rook-ceph --namespace rook-ceph  -f rook/deploy/charts/rook-ceph-cluster/values.yaml

- ugrade the ceph-cluster with custom values.
  ```
  helm upgrade --reuse-values --values rook/deploy/charts/rook-ceph-cluster/values.yaml rook-ceph-cluster rook-release/rook-ceph-cluster --version v1.17.2 -n rook-ceph
  ```

https://github.com/rook/rook.git
https://rook.io/docs/rook/latest-release/Getting-Started/quickstart/#tldr
https://rook.io/docs/rook/latest-release/Helm-Charts/ceph-cluster-chart/#release

