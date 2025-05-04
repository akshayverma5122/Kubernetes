### Control plane components

1. kube-apiserver
2. etcd
3. kube-scheduler
4. kube-controller-manager
5. cloud-controller-manager

### Node components 

1. kubelet
2. kube-proxy (optional)
3. Container runtime

### Addons 

1. DNS (coredns)
2. Web UI (Dashboard)
3. Container resource monitoring
4. Cluster-level Logging
5. Network plugins - container network interface (CNI)

### Control plane deployment options

1. Traditional deployment - managed as systemd services
2. Static Pods - managed by kubeadm
3. Self-hosted - kubernetes inside pods 
4. Managed Kubernetes services - cloud provider managed kubernetes

### Cluster management tools

1. kubeadm
2. kops
3. Kubespray
