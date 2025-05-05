### Kubernetes installation steps in ubuntu server.
#### kubernetes Nodes - single master, two worker
##### perform below steps in all the master and worker nodes. 

1. make the fqdn entry in /etc/hosts file 

		echo "172.16.0.15 worker1" | sudo tee -a /etc/hosts > /dev/null
		echo "172.16.0.16 worker2" | sudo tee -a /etc/hosts > /dev/null
		echo "172.16.0.17 master1" | sudo tee -a /etc/hosts > /dev/null

2. Disable the swap memory.
   
		sudo swapoff -a 
		sudo sed -i.bak '/\sswap\s/s/^/#/' /etc/fstab

4. Stop and disable the firewalld.

		sudo systemctl stop ufw.service
		sudo systemctl disable  ufw.service

5. Install the dependencies for adding repositories

		sudo apt-get update
		sudo apt-get install -y software-properties-common curl

6. Add the Kubernetes repository

		sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

		sudo echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

7. Add the CRI-O repository

		sudo curl -fsSL https://download.opensuse.org/repositories/isv:/cri-o:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

		sudo echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://download.opensuse.org/repositories/isv:/cri-o:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/cri-o.list

6. list and Install the specific version of kubelet, kubectl and kubeadm packages
   
   listing the packages

		sudo apt-get update
		sudo apt list -a kubeadm
		sudo apt list -a kubelet
		sudo apt list -a kubectl
		sudo apt list -a cri-o

   installing the packages

		sudo apt-get install cri-o=1.28.11-1.1 kubelet=1.28.12-1.1 kubeadm=1.28.12-1.1  kubectl=1.28.12-1.1 -y

8. Start and enable the CRI-O. Enable the kubelet as well.

		sudo systemctl enable crio.service
		sudo systemctl enable kubelet.service
		sudo systemctl start crio.service

9. udpate the network configuration. 

		sudo modprobe br_netfilter
		sudo sysctl -w net.ipv4.ip_forward=1

10. execute in the master node only.

		sudo kubeadm init --v=5

- output is similar to below

		Your Kubernetes control-plane has initialized successfully!

		To start using your cluster, you need to run the following as a regular user:

		  mkdir -p $HOME/.kube
		  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		  sudo chown $(id -u):$(id -g) $HOME/.kube/config

		Alternatively, if you are the root user, you can run:

		  export KUBECONFIG=/etc/kubernetes/admin.conf

		You should now deploy a pod network to the cluster.
		Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
		  https://kubernetes.io/docs/concepts/cluster-administration/addons/

		Then you can join any number of worker nodes by running the following on each as root:

		kubeadm join 192.168.4.205:6443 --token phdalh.k1xvjxb9gfus1gd8 \
        --discovery-token-ca-cert-hash sha256:a0a08554444f3b32e0aaefc8c0705586a013bfe141a296ddaba8c59a4e607e94

11. Join the worker nodes to cluster using above generated commands. Execute the below commands in worker node only.

		kubeadm join 192.168.4.205:6443 --token phdalh.k1xvjxb9gfus1gd8 \
        --discovery-token-ca-cert-hash sha256:a0a08554444f3b32e0aaefc8c0705586a013bfe141a296ddaba8c59a4e607e94

12. Execute the below commands in master node only to access the kubernetes cluster.


		mkdir -p $HOME/.kube
		sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		sudo chown $(id -u):$(id -g) $HOME/.kube/config

13. Execute the below command in master node only to validate the cluster is deployed as expected.

		kubectl get nodes -owide
		kubectl get pods -A -owide
 

		






		




   

   
