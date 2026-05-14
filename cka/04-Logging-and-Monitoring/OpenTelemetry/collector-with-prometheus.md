### collector integration with prometheus backend

1. add the prometheus helm repo and save the default configuration values. 

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
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
```
3. deploy the prometheus.

```
helm install my-prometheus prometheus-community/prometheus --version 29.6.0
```
4. add the otel-collector helm repo and save the default configuration values.
```
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm show values open-telemetry/opentelemetry-collector  > otel-values.yaml
```
5. do the customization in otel-values.yaml to export the metrics to prometheus backened 

```
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
helm install otel-collector open-telemetry/opentelemetry-collector  --values  otel-values.yaml
```

