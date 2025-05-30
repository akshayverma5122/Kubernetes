helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm search repo prometheus
prometheus-community/prometheus  v27.16.0  v3.4.0   Prometheus is a monitoring system and time
kubectl create ns  monitoring
helm  install prometheus prometheus-community/prometheus  --version  27.16.0 -n  monitoring
helm uninstall prometheus -n  monitoring
