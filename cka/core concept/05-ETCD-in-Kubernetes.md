## ETCD Datastore
- The ETCD Datastore stores information regarding the cluster such as **`Nodes`**, **`PODS`**, **`Configs`**, **`Secrets`**, **`Accounts`**, **`Roles`**, **`Bindings`** and **`Others`**.
- Every information you see when you run the **`kubectl get`** command is from the **`ETCD Server`**.

## Setup - Manual
- If you setup your cluster from scratch then you deploy **`ETCD`** by downloading ETCD Binaries yourself
- Installing Binaries and Configuring ETCD as a service in your master node yourself.
  ```
  $ wget -q --https-only "https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz"
  ```  
## Setup - Kubeadm
- If you setup your cluster using **`kubeadm`** then kubeadm will deploy etcd server for you as a pod in **`kube-system`** namespace.
  ```
  $ kubectl get pods -n kube-system
  ```
  
## Explore ETCD
- To list all keys stored by kubernetes, run the below command
  ```
  $ kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt get / --prefix --keys-only"
  ```
- Kubernetes Stores data in a specific directory structure, the root directory is the **`registry`** and under that you have various kubernetes constructs such as **`minions`**, **`nodes`**, **`pods`**, **`replicasets`**, **`deployments`**, **`roles`**, **`secrets`** and **`Others`**.
- To list all members of etcd cluster.

		etcdctl --endpoints=https://172.19.0.5:2379 --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt --write-out=table member list

- Check the health of etcd nodes.

		etcdctl --endpoints=https://172.19.0.5:2379 --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt --write-out=table endpoint health

- Check the status of etcd nodes.

		etcdctl --endpoints=https://172.19.0.5:2379 --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt --write-out=table endpoint status

- add the member node in etcd cluster

		etcdctl --endpoints=https://172.19.0.3:2379 --learner=true --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt member add multinode-control-plane3 --peer-urls=https://172.19.0.5:2380 

-- add the --peer-urls to existing the etcd member nodes.

		etcdctl --endpoints=https://172.19.0.3:2379  --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt  member update 8e9e05c52164694d --peer-urls=https://172.19.0.3:2380


    
  
## ETCD in HA Environment
   - In a high availability environment, you will have multiple master nodes in your cluster that will have multiple ETCD Instances spread across the master nodes.
   - Make sure etcd instances know each other by setting the right parameter in the **`etcd.service`** configuration. The **`--initial-cluster`** option where you need to specify the different instances of the etcd service.

## Important Notes - ETCD 

- etcd is a leader-based distributed system. Ensure that the leader periodically send heartbeats on time to all followers to keep the cluster stable.

- You should run etcd as a cluster with an odd number of members.
- **etcdctl:** This is the primary command-line client for interacting with etcd over a network. It is used for day-to-day operations such as managing keys and values, administering the cluster, checking health, and more.
- **etcdutl:** This is an administration utility designed to operate directly on etcd data files, including migrating data between etcd versions, defragmenting the database, restoring snapshots, and validating data consistency. For network operations, etcdctl should be used.
- **--peer-key-file** and **--peer-cert-file** for securing communication between etcd members, and **--key-file** and **--cert-file** for securing communication between etcd and its clients. **--client-cert-auth** along with TLS, it verifies the certificates from clients by using system CAs or the CA passed in by **--trusted-ca-file** flag
- Scaling out etcd clusters increases availability by trading off performance. Scaling does not increase cluster performance nor capability. A general rule is not to scale out or in etcd clusters. Do not configure any auto scaling groups for etcd clusters. It is strongly recommended to always run a static five-member etcd cluster for production Kubernetes clusters at any officially supported scale.

## Replacing a failed etcd member
 
   
K8s Reference Docs:
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#stacked-control-plane-and-etcd-nodes
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#external-etcd-nodes
