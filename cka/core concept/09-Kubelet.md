# Kubelet
#### Kubelet is the sole point of contact for the kubernetes cluster
- The **`kubelet`** will create the pods on the nodes, the scheduler only decides which pods goes where.
- You can manage the configuration of your kubelets manually, but kubeadm now provides a KubeletConfiguration API type for managing your kubelet configurations centrally  
## Install kubelet
- Kubeadm does not deploy kubelet by default. You must manually download and install it.
- Download the kubelet binary from the kubernetes release pages [kubelet](https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet). For example: To download kubelet v1.13.0, Run the below command.
  ```
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet
  ```
- Extract it
- Run it as a service  
## View kubelet options
- You can also see the running process and affective options by listing the process on worker node and searching for kubelet.
  ``` 
  $ ps -aux |grep kubelet
  ```
## Viewing the kubelet configuration
- Start a proxy server using kubectl proxy in your terminal.

		kubectl proxy

- Open another terminal window and use curl to fetch the kubelet configuration. Replace <node-name> with the actual name of your node

		curl -X GET http://127.0.0.1:8001/api/v1/nodes/<node-name>/proxy/configz | jq .

  output of the above commands
  ```
  {
  "kubeletconfig": {
    "enableServer": true,
    "staticPodPath": "/etc/kubernetes/manifests",
    "syncFrequency": "1m0s",
    "fileCheckFrequency": "20s",
    "httpCheckFrequency": "20s",
    "address": "0.0.0.0",
    "port": 10250,
    "tlsCertFile": "/var/lib/kubelet/pki/kubelet.crt",
    "tlsPrivateKeyFile": "/var/lib/kubelet/pki/kubelet.key",
    "rotateCertificates": true,
    "authentication": {
      "x509": {
        "clientCAFile": "/etc/kubernetes/pki/ca.crt"
      },
      "webhook": {
        "enabled": true,
        "cacheTTL": "2m0s"
      },
      "anonymous": {
        "enabled": false
      }
    },
    "authorization": {
      "mode": "Webhook",
      "webhook": {
        "cacheAuthorizedTTL": "5m0s",
        "cacheUnauthorizedTTL": "30s"
      }
    },
    "registryPullQPS": 5,
    "registryBurst": 10,
    "eventRecordQPS": 50,
    "eventBurst": 100,
    "enableDebuggingHandlers": true,
    "healthzPort": 10248,
    "healthzBindAddress": "127.0.0.1",
    "oomScoreAdj": -999,
    "clusterDomain": "cluster.local",
    "clusterDNS": [
      "10.96.0.10"
    ],
    "streamingConnectionIdleTimeout": "4h0m0s",
    "nodeStatusUpdateFrequency": "10s",
    "nodeStatusReportFrequency": "5m0s",
    "nodeLeaseDurationSeconds": 40,
    "imageMinimumGCAge": "2m0s",
    "imageGCHighThresholdPercent": 85,
    "imageGCLowThresholdPercent": 80,
    "volumeStatsAggPeriod": "1m0s",
    "cgroupsPerQOS": true,
    "cgroupDriver": "systemd",
    "cpuManagerPolicy": "none",
    "cpuManagerReconcilePeriod": "10s",
    "memoryManagerPolicy": "None",
    "topologyManagerPolicy": "none",
    "topologyManagerScope": "container",
    "runtimeRequestTimeout": "2m0s",
    "hairpinMode": "promiscuous-bridge",
    "maxPods": 110,
    "podPidsLimit": -1,
    "resolvConf": "/run/systemd/resolve/resolv.conf",
    "cpuCFSQuota": true,
    "cpuCFSQuotaPeriod": "100ms",
    "nodeStatusMaxImages": 50,
    "maxOpenFiles": 1000000,
    "contentType": "application/vnd.kubernetes.protobuf",
    "kubeAPIQPS": 50,
    "kubeAPIBurst": 100,
    "serializeImagePulls": true,
    "evictionHard": {
      "imagefs.available": "15%",
      "memory.available": "100Mi",
      "nodefs.available": "10%",
      "nodefs.inodesFree": "5%"
    },
    "evictionPressureTransitionPeriod": "5m0s",
    "enableControllerAttachDetach": true,
    "makeIPTablesUtilChains": true,
    "iptablesMasqueradeBit": 14,
    "iptablesDropBit": 15,
    "failSwapOn": true,
    "memorySwap": {},
    "containerLogMaxSize": "10Mi",
    "containerLogMaxFiles": 5,
    "configMapAndSecretChangeDetectionStrategy": "Watch",
    "enforceNodeAllocatable": [
      "pods"
    ],
    "volumePluginDir": "/usr/libexec/kubernetes/kubelet-plugins/volume/exec/",
    "logging": {
      "format": "text",
      "flushFrequency": "5s",
      "verbosity": 0,
      "options": {
        "json": {
          "infoBufferSize": "0"
        }
      }
    },
    "enableSystemLogHandler": true,
    "enableSystemLogQuery": false,
    "shutdownGracePeriod": "0s",
    "shutdownGracePeriodCriticalPods": "0s",
    "enableProfilingHandler": true,
    "enableDebugFlagsHandler": true,
    "seccompDefault": false,
    "memoryThrottlingFactor": 0.9,
    "registerNode": true,
    "localStorageCapacityIsolation": true,
    "containerRuntimeEndpoint": "unix:///var/run/crio/crio.sock"
  }
	}

  ```

- same info can be printed using the below commands

		kubeadm config print init-defaults --component-configs KubeletConfiguration

## kubelet configuration files 
- The KubeConfig file to use for the TLS Bootstrap is /etc/kubernetes/bootstrap-kubelet.conf, but it is only used if /etc/kubernetes/kubelet.conf does not exist.
- The KubeConfig file with the unique kubelet identity is /etc/kubernetes/kubelet.conf.
- The file containing the kubelet's ComponentConfig is /var/lib/kubelet/config.yaml.
- The dynamic environment file that contains KUBELET_KUBEADM_ARGS is sourced from /var/lib/kubelet/kubeadm-flags.env.
- The file that can contain user-specified flag overrides with KUBELET_EXTRA_ARGS is sourced from /etc/default/kubelet (for DEBs), or /etc/sysconfig/kubelet (for RPMs). KUBELET_EXTRA_ARGS is last in the flag chain and has the highest priority in the event of conflicting settings.

K8s Reference Docs:
- https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/
- https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/kubelet-integration/
