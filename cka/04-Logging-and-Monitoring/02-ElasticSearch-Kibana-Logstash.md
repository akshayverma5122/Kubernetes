### ELK installation and configuration

1. Create the directory in nfs server for ElasticSearch and Logstash for storing the logs using persistent volume and persisten volume claim. 
```
sudo mkdir -p /nfs_data/ElasticSearch_data/ElasticMasterNode01
sudo mkdir -p /nfs_data/ElasticSearch_data/ElasticDataNode01
sudo mkdir -p /nfs_data/ElasticSearch_data/LogstashNode01
```
2. Create the persistent volume for ElasticSearch master & data nodes and logstash. 
```
kubectl create -f ElasticDataNode01.yaml
kubectl create -f ElasticMasterNode01.yaml
kubectl create -f LogstashNode01.yaml
```
3. Deploy the custom resource defintion and operator for ElasticSearch cluster. 
```
kubectl create -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/04-Logging-and-Monitoring/manifests/ElasticSearch_CRDS.yaml
kubectl create -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/04-Logging-and-Monitoring/manifests/elastic_search_operator.yaml
```
4. Deploy ElasticSearch Cluster with master and data nodes. 
```
kubectl create -n elastic-system -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/04-Logging-and-Monitoring/manifests/elasticsearch_cluser.yaml
```
5. Deploy the filebeat and logstash. 
```
kubectl -n elastic-system create -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/04-Logging-and-Monitoring/manifests/filebeat.yaml
kubectl -n elastic-system create -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/04-Logging-and-Monitoring/manifests/logstash.yaml
```

### ElasticSearch Reference URLs
- https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/install-using-yaml-manifest-quickstart
