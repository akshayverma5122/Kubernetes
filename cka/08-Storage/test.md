
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
2. Install chart
   ```
   kubectl create ns monitoring
   helm install monitoring-stack prometheus-community/kube-prometheus-stack --version 72.8.0 -n monitoring
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

