### Perform the below steps where the internet connectivity is available 

1. Add the helm repo 
```
helm repo add elastic https://helm.elastic.co
helm repo update
```
2. View available configuration options and save it. 
```
helm show values elastic/eck-operator --version 3.0.0 > operator.yaml
helm show values elastic/eck-elasticsearch --version 0.15.0 > elasticsearch.yaml
helm show values elastic/eck-logstash --version 0.15.0 > logstash.yaml
helm show values elastic/eck-kibana --version 0.15.0 > kibana.yaml
```
3. download the helm charts for crds, operator, elasticsearch, kibana and logstash.  
```
helm pull elastic/eck-operator-crds  --version 3.0.0   
helm pull elastic/eck-operator --version 3.0.0
helm pull elastic/eck-elasticsearch    --version 0.15.0
helm pull elastic/eck-kibana  --version 0.15.0
helm pull elastic/eck-logstash    --version 0.15.0
```
### perform the below steps in offline server

1. Creating the folder structure for elastic search persistent volume.
```
sudo mkdir -p /u01/oracle/elasticsearch_cluster/elastic_master_node01
sudo mkdir -p /u01/oracle/elasticsearch_cluster/elastic_master_node02
sudo mkdir -p /u01/oracle/elasticsearch_cluster/elastic_master_node03
sudo mkdir -p /u01/oracle/elasticsearch_cluster/elastic_data_node01
sudo mkdir -p /u01/oracle/elasticsearch_cluster/elastic_data_node02
sudo mkdir -p /u01/oracle/elasticsearch_cluster/elastic_data_node03
sudo mkdir -p /u01/oracle/elasticsearch_cluster/logstash_node01
```
2. change the nfs server value in manifest file and provision the persistent volume.
```
kubectl create -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/04-Logging-and-Monitoring/manifests/eck_pv.yml
```
3. change the permission.
```
sudo chmod -R 777 /u01/oracle/elasticsearch_cluster
```
4. Create the namespace. 
```
kubectl create ns elastic-system
```
5. Create the implementation directory and upload all the downloaded charts and configuration file in that directory. Locate the created directory. 
```
mkdir -p /u01/oracle/elasticsearch_cluster/eck-setup
cd /u01/oracle/elasticsearch_cluster/eck-setup
```
6. install the cutome resource definition.
```
helm install -n elastic-system eck-crds ./eck-operator-crds-3.0.0.tgz
```
7. customize the below parameters in operator.yaml file in the case of private registry. Then, install the helm charts. 
```
image:
  # repository is the container image prefixed by the registry name.
  repository: container-registry.example.com/eck/eck-operator
config:
  containerRegistry: container-registry.example.com
```
```
helm install -n elastic-system eck-operator ./eck-operator-3.0.0.tgz --values=/u01/oracle/elasticsearch_cluster/eck-setup/operator.yaml
```
8. customize the below parameters in eck_elasticsearch.yaml and install the charts.
```
nodeSets:
- name: data
  count: 3
  config:
    node.store.allow_mmap: false
    node.roles: ["data", "ingest"]
  podTemplate:
   spec:
    containers:
      - name: elasticsearch
        resources:
          limits:
            # cpu: 1
            memory: 2Gi
          requests:
            # cpu: 1
            memory: 2Gi

    initContainers:
       - command:
         - sh
         - -c
         - chown -R 1000:1000 /usr/share/elasticsearch/data
         - chmod -R 777 /usr/share/elasticsearch
         name: fix-permissions
         image: container-registry.example.com/busybox:latest
         imagePullPolicy: IfNotPresent
         securityContext:
           privileged: true
           runAsUser: 0
  volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 100Gi
        storageClassName: nfs-storage
- name: master
  count: 3
  config:
    node.store.allow_mmap: false
    node.roles: ["master"]
  volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 20Gi
        storageClassName: nfs-storage
  podTemplate:
    spec:
      containers:
      - name: elasticsearch
        resources:
          limits:
            memory: 2Gi
          requests:
            memory: 2Gi
      initContainers:
       - command:
         - sh
         - -c
         - chown -R 1000:1000 /usr/share/elasticsearch/data
         - chmod -R 777 /usr/share/elasticsearch
         name: fix-permissions
         image: container-registry.example.com/busybox:latest
         imagePullPolicy: IfNotPresent
         securityContext:
           privileged: true
           runAsUser: 0
```





