
```
lvcreate -l 100%FREE -n lv_nfsdata nfs_vg
mkfs.ext4 /dev/nfs_vg/lv_nfsdata
systemctl daemon-reload
sudo apt install -y nfs-kernel-server
sudo chown nobody:nogroup /nfs_data
sudo chmod 777 /nfs_data
```
```
/srv/nfs/kubedata 172.16.0.0/24(rw,sync,no_subtree_check,no_root_squash)
sudo exportfs -ra
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
```

### kube-prometheus-stack Install Using Helm Chart
1. Add repository
   ```
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   ```
2. Search all the chart version and Install chart
   ```
   helm search repo  prometheus-community/kube-prometheus-stack  --versions
   kubectl create ns monitoring
    helm install monitoring-stack prometheus-community/kube-prometheus-stack --version 72.7.0 --set prometheus.prometheusSpec.maximumStartupDurationSeconds=900 -n monitoring
   ```
   output will be similar like below:
   ```
   NAME: monitoring-stack
   LAST DEPLOYED: Sat May 31 10:23:03 2025
   NAMESPACE: monitoring
   STATUS: deployed
   REVISION: 1
   NOTES:
   kube-prometheus-stack has been installed. Check its status by running:
   kubectl --namespace monitoring get pods -l "release=monitoring-stack"

   Get Grafana 'admin' user password by running:

   kubectl --namespace monitoring get secrets monitoring-stack-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

   Access Grafana local instance:

   export POD_NAME=$(kubectl --namespace monitoring get pod -l 
   "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=monitoring-stack" -oname)
   kubectl --namespace monitoring port-forward $POD_NAME 3000

   Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager 
   and Prometheus instances using the Operator.
   ```
3. Checking kube-prometheus-stack crds
   ```
   kubectl -n monitoring get prometheuses
   kubectl -n monitoring get alertmanagers
   ```
   
### Access prometheus, grafana and alertmanager using NodePort
1. Generate the nodeport service yaml file and change the labels & selector for grafana, prometheus and alertmanager.
   ```
   kubectl -n monitoring create service nodeport prom-dashboard --tcp=9090:9090 --node-port=30090 --dry-run=server -o yaml > prom-dashboard-np-service.yaml
   ```
### kube-prometheus-stack uninstallation Using Helm Chart
1. unistallation of kube-prometheus-stack
   ```
   helm uninstall monitoring-stack -n monitoring
   ```
2. CRDs created by this chart are not removed by default and should be manually cleaned up.
   ```
   kubectl delete crd alertmanagerconfigs.monitoring.coreos.com
   kubectl delete crd alertmanagers.monitoring.coreos.com
   kubectl delete crd podmonitors.monitoring.coreos.com 
   kubectl delete crd probes.monitoring.coreos.com
   kubectl delete crd prometheusagents.monitoring.coreos.com
   kubectl delete crd prometheuses.monitoring.coreos.com
   kubectl delete crd prometheusrules.monitoring.coreos.com
   kubectl delete crd scrapeconfigs.monitoring.coreos.com
   kubectl delete crd servicemonitors.monitoring.coreos.com
   kubectl delete crd thanosrulers.monitoring.coreos.com
   ```


### Reference Docs 
- https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack
- https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
- https://github.com/prometheus-operator/kube-prometheus

