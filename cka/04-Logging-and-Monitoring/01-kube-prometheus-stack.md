### NFS server installation and configuration for PV and PVC
1. Add /dev/sdb disk. create the logical volume and formate it. Create the /nfs_data folder and give the permission. 
```
pvcreate /dev/sdb
vgcreate nfs_vg /dev/sdb
lvcreate -l 100%FREE -n lv_nfsdata nfs_vg
mkfs.ext4 /dev/nfs_vg/lv_nfsdata
sudo mkdir /nfs_data
sudo chown nobody:nogroup /nfs_data
sudo chmod 777 /nfs_data
mount /dev/nfs_vg/lv_nfsdata  /nfs_data
```
2. install the nfs server and export the nfs directory.
   
```
sudo apt install -y nfs-kernel-server
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
/nfs_data 172.16.0.0/24(rw,sync,no_subtree_check,no_root_squash)
sudo exportfs -ra
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

### Customizing kube-prometheus-stack Using Helm Chart 
#### 1. Attaching the pvc in prometheus 
**A.** Create the storage class and persistent volume. please make sure the nfs-server and destination directory should be provision before the storageclass and persistent volume creation.
   ```
   mkdir -p /nfs_data/prometheus-server/node01
   chmod -R a+w prometheus-server/node01
   kubectl create -f storageclass.yaml
   kubectl create -f prometheus-pv-node-01.yaml
   ```
**B.** Create the custom volue file and render the below details for automatic persistent volume claim provisioning.
   ```
   prometheus:
    prometheusSpec:
     storageSpec:
       volumeClaimTemplate:
         spec:
           storageClassName: nfs-static
           accessModes: ["ReadWriteOnce"]
           resources:
             requests:
               storage: 10Gi
   ```
**C.** execute the helm upgrade command to customize the prometheus.
   ```
   helm -n monitoring upgrade --reuse-values --values=custom-value.yaml monitoring-stack  prometheus-community/kube-prometheus-stack --version 72.7.0
   ```
#### 2. Scaling the prometheus
**A.** Create the folder with write permission for node02 persistent volume. 
   ```
   mkdir -p /nfs_data/prometheus-server/node02
   chmod -R a+w /nfs_data/prometheus-server/node02
   ```
**B.** Create the persistent volume. 
   ```
   kubectl create -f prometheus-server-pv-02.yaml
   kubectl get pv
   ```
**C.** Render the replicas:2 details in custom-value.yaml for scaling the replicas.
   ```
   prometheus:
    prometheusSpec:
      replicas: 2
   ```
**D.** Scale the ReplicaSet to 2. 
   ```
   helm -n monitoring upgrade --reuse-values --values=custom-value.yaml monitoring-stack  prometheus-community/kube-prometheus-stack --version 72.7.0
   ```
### Access prometheus, grafana and alertmanager using NodePort
1. Generate the nodeport service yaml file and change the labels & selector for grafana, prometheus and alertmanager.
   ```
   kubectl -n monitoring create service nodeport prom-dashboard --tcp=9090:9090 --node-port=30090 --dry-run=server -o yaml > prom-dashboard-np-service.yaml
   ```
### Exploring the Node Exporter
1. Create and login in the test pods.
   ```
   kubectl run -it dnspods --image=nicolaka/netshoot --restart=Never
   kubectl exec -it dnspods -- /bin/bash
   ```
2. once node exported installed, by default it exports the metrics on http://localhost:9100/metrics. So verify the same.
   ```
   curl http://monitoring-stack-prometheus-node-exporter.monitoring.svc.cluster.local:9100/metrics
   curl http://monitoring-stack-prometheus-node-exporter.monitoring.svc.cluster.local:9100/metrics | grep "node_"
   ```
3. some node exporter metrics query.
   ```
   node_cpu_seconds_total
   node_exporter_build_info
   rate(node_cpu_seconds_total{mode="system"}[1m])
   node_filesystem_avail_bytes
   rate(node_network_receive_bytes_total[1m])
   ```
### Exporling the kube-state-metric server
- The metrics are exported on the HTTP endpoint /metrics on the listening port (default 8080).
- kube-state-metrics exposes its own general process metrics under --telemetry-host and --telemetry-port (default 8081).
- **kube-state-metrics vs. metrics-server** - The metrics-server is a project that has been inspired by Heapster and is implemented to serve the goals of core metrics pipelines in Kubernetes monitoring architecture. It is a cluster level component which periodically scrapes metrics from all Kubernetes nodes served by Kubelet through Metrics API. The metrics are aggregated, stored in memory and served in Metrics API format. The metrics-server stores the latest values only and is not responsible for forwarding metrics to third-party destinations. kube-state-metrics is focused on generating completely new metrics from Kubernetes' object state (e.g. metrics based on deployments, replica sets, etc.). It holds an entire snapshot of Kubernetes state in memory and continuously generates new metrics based off of it. And just like the metrics-server it too is not responsible for exporting its metrics anywhere. Having kube-state-metrics as a separate project also enables access to these metrics from monitoring systems such as Prometheus.
### Exploring prometheus
- Prometheus is a popular open-source monitoring and alerting system written in Golang, capable of collecting and processing metrics from various targets. collect the metrics from various targets using pull model.
- The general term for collecting metrics from the targets using Prometheus is called scraping. By default prometheus looks for metrics under /metrics path of the target.
- Prometheus primarily consists of the following.
  - Prometheus Server
  - Service Discovery
  - Time-Series Database (TSDB)
  - Targets
  - Exporters
  - Push Gateway
  - Alert Manager
  - Client Libraries
  - PromQL
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
- kube-prometheus-stack url 
  - https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack
  - https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
  - https://github.com/prometheus-operator/kube-prometheus
- Node exporter url
  - https://prometheus.io/docs/guides/node-exporter/
- Grafana Prebuilt Dashboard
  - https://grafana.com/grafana/dashboards/12486-node-exporter-full/
  - https://grafana.com/grafana/dashboards/21742-object-s-health-kube-state-metrics-v2/
- kube-state-metrics
  - https://github.com/kubernetes/kube-state-metrics
  - https://github.com/kubernetes/kube-state-metrics/tree/main/docs
- prometheus
  - https://devopscube.com/prometheus-architecture/
  - https://prometheus.io/docs/instrumenting/exporters
  - https://prometheus.io/docs/prometheus/1.8/configuration/configuration
