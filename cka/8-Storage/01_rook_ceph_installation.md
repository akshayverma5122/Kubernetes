
git clone --single-branch --branch v1.17.2 https://github.com/rook/rook.git

kubectl create  -f rook/deploy/examples/crds.yaml

helm install --create-namespace --namespace rook-ceph rook-ceph-cluster --set operatorNamespace=rook-ceph rook-release/rook-ceph-cluster -f rook/deploy/charts/rook-ceph-cluster/values.yaml


https://rook.io/docs/rook/latest-release/Getting-Started/quickstart/#tldr
https://rook.io/docs/rook/latest-release/Helm-Charts/ceph-cluster-chart/#release

