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
```
helm install -n elastic-system elasticsearch  ./eck-elasticsearch-0.15.0.tgz --values=/u01/oracle/elasticsearch_cluster/eck-setup/elasticsearch.yaml
```
9. customize the kibana.yml file and install kibana. 
```
elasticsearchRef:
      name: elasticsearch-eck-elasticsearch
config:
  server:
    basePath: /kibana
    rewriteBasePath: true
ingress:
  enabled: true
  annotations:
      kubernetes.io/ingress.class: nginx
      meta.helm.sh/release-name: kibana
      meta.helm.sh/release-namespace: elastic-system
      nginx.ingress.kubernetes.io/affinity: cookie
      nginx.ingress.kubernetes.io/affinity-mode: persistent
      nginx.ingress.kubernetes.io/backend-protocol: HTTPS
      nginx.ingress.kubernetes.io/connection-proxy-header: keep-alive
      nginx.ingress.kubernetes.io/enable-access-log: "false"
      nginx.ingress.kubernetes.io/proxy-body-size: 15m
      nginx.ingress.kubernetes.io/proxy-buffer-size: 48k
      nginx.ingress.kubernetes.io/proxy-buffering: "on"
      nginx.ingress.kubernetes.io/session-cookie-expires: "172800"
      nginx.ingress.kubernetes.io/session-cookie-hash: sha1
      nginx.ingress.kubernetes.io/session-cookie-max-age: "172800"
      nginx.ingress.kubernetes.io/session-cookie-name: kibana
      nginx.ingress.kubernetes.io/ssl-passthrough: "false"
  hosts:
    - host: ""
      path: /kibana
```
```
helm install -n elastic-system  kibana ./eck-kibana-0.15.0.tgz  --values=/u01/oracle/elasticsearch_cluster/eck-setup/kibana.yaml
```
9. obtain the elasticsearch username and password. 
```
kubectl get secret  elasticsearch-eck-elasticsearch-es-elastic-user -n elastic-system -o json | jq .data.elastic   ### base64 encoded values
echo "djZuZDhxeDAwcVc1VzVYRzY5UnMxSjFs" | base64 -d
```
10. change the password value in eck-logstash-pipeline-secret.yaml file and create the secret. 
```
kubectl create -n elastic-system -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/04-Logging-and-Monitoring/ELK/manifest/eck-logstash-pipeline-secret.yaml
```
11. Customize the logstash values file. install the logstash and create the logstash service for filebeat to consume it.
```
podTemplate: 
 spec:
  containers:
  - env:
    - name: LS_JAVA_OPTS
      value: -Xmx2g -Xms2g
    name: logstash
    resources:
      limits:
        cpu: "1"
        memory: 4Gi
      requests:
        cpu: "500m"
        memory: 2Gi
    volumeMounts:
    - mountPath: /usr/share/logstash/pipeline/inputs/beats-input.conf
      name: logstash-beats-input
      subPath: beats-input.conf
    - mountPath: /usr/share/logstash/pipeline/outputs/logstash-kubernetes-cluster-output-pipeline.conf
      name: logstash-kubernetes-cluster-output-pipeline
      subPath: logstash-kubernetes-cluster-output-pipeline.conf
    - mountPath: /usr/share/logstash/pipeline/outputs/coredns-output.conf
      name: logstash-coredns-output
      subPath: coredns-output.conf
    - mountPath: /usr/share/logstash/pipeline/outputs/kube-apiserver-output.conf
      name: logstash-kube-apiserver-output
      subPath: kube-apiserver-output.conf
    - mountPath: /usr/share/logstash/pipeline/outputs/oarm-application-logs.conf
      name: oarm-application-logs
      subPath: oarm-application-logs.conf
  volumes:
  - name: logstash-beats-input
    secret:
      secretName: logstash-beats-input
  - name: logstash-kubernetes-cluster-output-pipeline
    secret:
      secretName: logstash-kubernetes-cluster-output-pipeline
  - name: logstash-coredns-output
    secret:
      secretName: logstash-coredns-output
  - name: logstash-kube-apiserver-output
    secret:
      secretName: logstash-kube-apiserver-output
  - name: oarm-application-logs
    secret:
      secretName: oarm-application-logs
  
  initContainers:
   - command:
     - sh
     - -c
     - chown -R 1000:1000 /usr/share/logstash/data
     - chmod -R 777 /usr/share/logstash/data
     name: fix-permissions
     image: container-registry.exmaple.com/busybox:latest
     imagePullPolicy: IfNotPresent
     securityContext:
       privileged: true
       runAsUser: 0
pipelines:
  - path.config: /usr/share/logstash/pipeline/inputs/beats-input.conf
    pipeline.id: main
  - path.config: /usr/share/logstash/pipeline/outputs/logstash-kubernetes-cluster-output-pipeline.conf         
    pipeline.id: logstash-kubernetes-cluster-output-pipeline
  - path.config: /usr/share/logstash/pipeline/outputs/coredns-output.conf
    pipeline.id: coredns
  - path.config: /usr/share/logstash/pipeline/outputs/kube-apiserver-output.conf
    pipeline.id: kubeapiserver
  - path.config: /usr/share/logstash/pipeline/outputs/oarm-application-logs.conf
    pipeline.id: oarm-application-logs
volumeClaimTemplates:
- metadata:
    name: logstash-data
  spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 8Gi
    storageClassName: nfs-storage
elasticsearchRefs:
 - namespace: elastic-system
   name: elasticsearch-eck-elasticsearch
   clusterName: elasticsearch-eck-elasticsearch
```
```
helm install -n elastic-system logstash ./eck-logstash-0.15.0.tgz  --values=/u01/oracle/elasticsearch_cluster/eck-setup/logstash.yaml
kubectl create -f logstash-svc.yaml -n elastic-system
````
12. Create the filebeat secret and install the filebeat daemonset to collect the logs.
```
kubectl -n elastic-system create -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/04-Logging-and-Monitoring/ELK/manifest/eck_filebeat_config.yaml
kubectl -n elastic-system create -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/04-Logging-and-Monitoring/ELK/manifest/eck_filebeat.yaml 
```

13. finally create the kibana service to access the dashboard.
```
kubectl -n elastic-system create -f https://raw.githubusercontent.com/akshayverma5122/Kubernetes/refs/heads/master/cka/04-Logging-and-Monitoring/ELK/manifest/eck_kibana_svc.yaml
```
14. Access the kibana dashboard.
```
kibana url - https://172.16.0.90:32000
username - elastic
password - v6nd8qx00qW5W5XG69Rs1J1l
```







