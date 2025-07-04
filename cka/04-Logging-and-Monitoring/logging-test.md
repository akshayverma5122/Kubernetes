```
apiVersion: batch/v1
kind: Job
metadata:
  name: filebeat-setup
  namespace: elastic-system
spec:
  template:
    spec:
      containers:
        - name: filebeat
          image: docker.elastic.co/beats/filebeat:8.17.4
          command: [ "filebeat", "setup", "-e" ]
          env:
            - name: ELASTIC_PASSWORD
              value: "YrQbPO9B0Y6L5800Z23uE5FA"
          args:
            - "-E"
            - "output.logstash.enabled=false"
            - "-E"
            - "output.elasticsearch.hosts=[\"https://elasticsearch-cluster-es-http.elastic-system.svc:9200\"]"
            - "-E"
            - "output.elasticsearch.username=elastic"
            - "-E"
            - "output.elasticsearch.password=${ELASTIC_PASSWORD}"
            - "-E"
            - "output.elasticsearch.ssl.verification_mode=none"
            - "-E"
            - "setup.kibana.host=https://kibana-kb-http.elastic-system.svc:5601"
            - "-E"
            - "setup.kibana.ssl.verification_mode=none"
      restartPolicy: Never

```
```
filebeat.autodiscover:
        providers:
        - templates:
          - condition:
              equals:
                kubernetes.labels.k8s-app: kube-dns
            config:
            - log:
                enabled: true
                var.paths:
                - /var/log/containers/*-${data.kubernetes.container.id}.log
                var.tags:
                - coredns
                - staging
              module: coredns
          type: kubernetes

```

growk pattern for coredns logs 

```
\[%{LOGLEVEL:log.level}\] %{IP:client.ip}:%{INT:client.port} - %{INT:dns.id} \"%{WORD:dns.question.type} IN %{HOSTNAME:dns.query.name} %{WORD:dns.protocol} %{INT:dns.size} %{WORD:dns.do} %{INT:dns.bufsize}\" %{WORD:dns.rcode} %{DATA:dns.flags} %{INT:dns.resp_size} %{NUMBER:dns.latency:float}s

```

```
PUT _index_template/logs-kubernetes-coredns
{
  "index_patterns": ["logs-kubernetes.coredns-*"],
  "data_stream": {},
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "client": {
          "properties": {
            "ip": { "type": "ip" },
            "port": { "type": "integer" }
          }
        },
        "dns": {
          "properties": {
            "id": { "type": "integer" },
            "query": {
              "properties": {
                "name": { "type": "keyword" },
                "type": { "type": "keyword" }
              }
            },
            "rcode": { "type": "keyword" },
            "latency": { "type": "float" },
            "protocol": { "type": "keyword" },
            "flags": { "type": "keyword" }
          }
        }
      }
    }
  },
  "priority": 500
}

```

```
filebeat.autodiscover:
        providers:
        - node: ${NODE_NAME}
          templates:
          - condition:
              equals:
                kubernetes.labels.k8s-app: kube-dns
            config:
            - exclude_lines:
              - ^\s+[\-`('.|_]
              id: coredns-input
              paths:
              - /var/log/containers/*-${data.kubernetes.container.id}.log
              processors:
              - add_fields:
                  fields:
                    component: coredns
                    kubernetes.namespace: ${data.kubernetes.namespace}
                  target: ""
              type: container
          - condition:
              equals:
                kubernetes.container.name: kube-apiserver
            config:
            - id: apiserver-input
              paths:
              - /var/log/containers/*-${data.kubernetes.container.id}.log
              processors:
              - add_fields:
                  fields:
                    component: kube-apiserver
                    kubernetes.namespace: ${data.kubernetes.namespace}
                  target: ""
              type: container
          - condition:
              equals:
                kubernetes.container.name: kube-controller-manager
            config:
            - id: controller-manager-input
              paths:
              - /var/log/containers/*-${data.kubernetes.container.id}.log
              processors:
              - add_fields:
                  fields:
                    component: kube-controller-manager
                    kubernetes.namespace: ${data.kubernetes.namespace}
                  target: ""
              type: container
          - condition:
              equals:
                kubernetes.container.name: kube-scheduler
            config:
            - id: scheduler-input
              paths:
              - /var/log/containers/*-${data.kubernetes.container.id}.log
              processors:
              - add_fields:
                  fields:
                    component: kube-scheduler
                    kubernetes.namespace: ${data.kubernetes.namespace}
                  target: ""
              type: container
          - condition:
              equals:
                kubernetes.namespace: kube-system
            config:
            - exclude_lines:
              - ^\s+[\-`('.|_]
              id: generic-kube-system-input
              paths:
              - /var/log/containers/*-${data.kubernetes.container.id}.log
              processors:
              - add_fields:
                  fields:
                    component: generic-kube-system
                    kubernetes.namespace: ${data.kubernetes.namespace}
                  target: ""
              type: container
          type: kubernetes
```
