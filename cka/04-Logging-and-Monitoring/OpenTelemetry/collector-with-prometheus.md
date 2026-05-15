### collector as daemonset integration with prometheus backend

* we should use the hostMetrics, kubernetesAttributes and kubeletMetrics when otel collector is deployed as daemonsets. 
* we should use the kubernetesEvents, clusterMetrics when otel collector is deployed as deployment object.
  
1. add the prometheus helm repo and save the default configuration values. 

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm search repo kube-prometheus-stack
helm show values prometheus-community/kube-prometheus-stack > prom-default-values.yaml
```

2. disable the extra component - node exporter, metrics server, alert manager, grafana etc. 
```
alertmanager:
  enabled: false
grafana:
  enabled: false
kubeApiServer:
  enabled: true
kubelet:
  enabled: true
kubeControllerManager:
  enabled: true
coreDns:
  enabled: true
kubeDns:
  enabled: false
kubeEtcd:
  enabled: true
kubeScheduler:
  enabled: true
kubeProxy:
  enabled: true
kubeStateMetrics:
  enabled: false
nodeExporter:
  enabled: false
prometheus:
  service:
    type: NodePort
```
3. deploy the prometheus.

```
helm install my-kube-prometheus-stack prometheus-community/kube-prometheus-stack --version 85.0.3 --values=prom-default-values.yaml
```
4. add the otel-collector helm repo and save the default configuration values.
```
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm show values open-telemetry/opentelemetry-collector  > otel-values.yaml
```
5. do the customization in otel-values.yaml to export the metrics to prometheus backened 

```
mode: daemonset
presets:
    hostMetrics:
    enabled: true
config:
  exporters:
    prometheus/custom:
      endpoint: 0.0.0.0:8889
      namespace: default
  service:
    pipelines:
      metrics:
        exporters:
          - prometheus/custom
      metrics:
        exporters:
          - debug
          - prometheus/custom
image:
  repository: otel/opentelemetry-collector-contrib
podMonitor:
  enabled: true
  extraLabels: 
     release: my-kube-prometheus-stack
ports:
  metrics:
    enabled: true
    containerPort: 8889
    servicePort: 8889
    protocol: TCP
```
6. install the otel-collector.
```
helm install otel-collector open-telemetry/opentelemetry-collector  --values=otel-values.yaml
```
7. enable the preset to test each of them
```
presets:
  hostMetrics:
    enabled: true
  kubernetesAttributes:
    enabled: true
    extractAllPodLabels: true
    extractAllPodAnnotations: true
  kubeletMetrics:
    enabled: true
```
8. use the following command to expose the collector and prometheus for testing. 

```
k port-forward pods/otel-collector-opentelemetry-collector-agent-bxhr5 8889:8889
k port-forward pods/prometheus-my-kube-prometheus-stack-prometheus-0 9090:9090
```
9. check the metrcis exported by collector in the following url. check the otel collector in prometheus target.
```
curl http://localhost:8889/metrics
```
### collector as daemonset integration with elasticsearch backend

1. install the elasticsearch and kibana [02-ElasticSearch-Kibana-Logstash](cka/04-Logging-and-Monitoring/02-ElasticSearch-Kibana-Logstash.md)
2. customize the otel-collector.

```
exporters:
    elasticsearch:
      endpoint: https://elasticsearch-cluster-es-http.elastic-system.svc.cluster.local:9200
      auth:
        authenticator: basicauth
      tls:
       insecure_skip_verify: true
extensions:
     basicauth:
      client_auth:
         username: elastic
         password: e090HfjXceOm4z2w296Ek0y7
receivers:
    filelog:
      include:
      - /var/log/pods/*/*/*.log
      exclude:
    # Exclude logs from all containers named otel-collector
      - /var/log/pods/*/otel-collector/*.log
      start_at: end
      include_file_path: true
      include_file_name: false
      operators:
    # parse container logs
      - type: container
        id: container-parser
service:
  extensions:
      - health_check
      - basicauth
  pipelines:
      logs:
        exporters:
          - debug
          - elasticsearch
        processors:
          - memory_limiter
          - batch
        receivers:
          - otlp
          - filelog
```
3. apply the changes
```
helm upgrade  otel-collector open-telemetry/opentelemetry-collector  --values=otel-values.yaml
```
### uninstallation of otel-collector 

1. uninstall the otelcollector and remove its helm repo. 
   ```
   helm uninstall otel-collector
   ```
   ```
   helm repo remove open-telemetry
   ```
2. uninstall the kube-prom-stack. delete the crds. 
   ```
   helm uninstall my-kube-prometheus-stack
   ```
   ```
   kubectl delete crd $(kubectl get crd | awk '/monitoring.coreos.com/ {print $1}')
   ```
   ```
   helm repo remove prometheus-community
   ```
   
