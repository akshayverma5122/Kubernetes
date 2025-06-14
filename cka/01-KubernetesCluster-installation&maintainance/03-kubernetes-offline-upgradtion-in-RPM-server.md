### Kubernetes Cluster Upgradation
#### Kubernetes Cluster Upgradation in RPM Based server

- Peform below Steps where internet connectivity is available.
  
  - Add the Kubernetes repository
    ```
    cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
    enabled=1
    gpgcheck=1
    gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
    EOF
    ```
  - list the repository
    
    ```
    dnf repolist
    ```
  - check the installed version of kubeadm in offline server and download the latest version of it. for example my installed version of kubeadm in offline server is v1.30.10 and latest version in v1.30 is v1.30.13
    
    ```
    kubeadm version ###offline server
    dnf info kubeadm ###online server
    ```
  - Download the kubeadm, cri-tools, kubelet, kubernetes-cni and kubectl
    
    ```
    dnf download --arch=x86_64 --resolve kubeadm
    dnf download --arch=x86_64 --resolve kubectl
    dnf download --arch=x86_64 --resolve kubelet
    ```
  - install the kubeadm and list the images
    
    ```
    rpm -ivh kubeadm-1.30.13-150500.1.1.x86_64.rpm --nodeps --force
    kubeadm config images list
    ```
    output will be similar to below
    
    ```
    [root@ocnemaster1 kube_packages]# kubeadm config images list
    I0614 12:55:14.297930    3326 version.go:256] remote version is much newer: v1.33.1; falling back to: stable-1.30
    registry.k8s.io/kube-apiserver:v1.30.13
    registry.k8s.io/kube-controller-manager:v1.30.13
    registry.k8s.io/kube-scheduler:v1.30.13
    registry.k8s.io/kube-proxy:v1.30.13
    registry.k8s.io/coredns/coredns:v1.11.3
    registry.k8s.io/pause:3.9
    registry.k8s.io/etcd:3.5.15-0
    ```
  - Download all the images using podman or docker
    
    ```
    podman pull registry.k8s.io/kube-apiserver:v1.30.13
    podman pull registry.k8s.io/kube-controller-manager:v1.30.13
    podman pull registry.k8s.io/kube-scheduler:v1.30.13
    podman pull registry.k8s.io/kube-proxy:v1.30.13
    podman pull registry.k8s.io/coredns/coredns:v1.11.3
    podman pull registry.k8s.io/pause:3.9
    podman pull registry.k8s.io/etcd:3.5.15-0
    ```
  - tar all the images
    
    ```
    podman save -o kube-apiserver.tar registry.k8s.io/kube-apiserver:v1.30.13
    podman save -o kube-controller-manager.tar registry.k8s.io/kube-controller-manager:v1.30.13
    podman save -o kube-scheduler.tar registry.k8s.io/kube-scheduler:v1.30.13
    podman save -o kube-proxy.tar registry.k8s.io/kube-proxy:v1.30.13
    podman save -o coredns.tar registry.k8s.io/coredns/coredns:v1.11.3
    podman save -o pause.tar  registry.k8s.io/pause:3.9
    podman save -o etcd.tar  registry.k8s.io/etcd:3.5.15-0
    ```
  - compress the downloaded images and packages. transter the tar file to offline server.
    
    ```
    tar -cvzf kubernetes_v_1_30_13.tar.gz kubernetes_v_1_30_13
    ```
- Perform the below steps in local registry server
  
  - untar the container images 
    ```
    podman load -i kube-apiserver.tar
    podman load -i kube-controller-manager.tar
    podman load -i kube-scheduler.tar
    podman load -i kube-proxy.tar
    podman load -i coredns.tar
    podman load -i pause.tar
    podman load -i etcd.tar
    ```
  - tag the container images
    ```
    podman tag registry.k8s.io/kube-apiserver:v1.30.13
    podman tag registry.k8s.io/kube-controller-manager:v1.30.13
    podman tag registry.k8s.io/kube-scheduler:v1.30.13
    podman tag registry.k8s.io/kube-proxy:v1.30.13
    podman tag registry.k8s.io/coredns/coredns:v1.11.3
    podman tag registry.k8s.io/pause:3.9
    podman tag registry.k8s.io/etcd:3.5.15-0
    ```
    
    

  
