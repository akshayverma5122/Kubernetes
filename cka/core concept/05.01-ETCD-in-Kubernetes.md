### Creating the ETCD Database backup and restoring using etcdctl and etcdutl command.  
1. Download the etcdctl binary for your os distribution. Below is for the ubuntu server.

		wget https://github.com/etcd-io/etcd/releases/download/v3.5.17/etcd-v3.5.17-linux-amd64.tar.gz
		tar -xvf etcd-v3.5.17-linux-amd64.tar.gz
		mv etcdctl /usr/local/bin
		mv etcdutl /usr/local/bin

2. check the version of etcd client.

		etcdctl version
		etcdutl version

3. create the etcd database backup using below command. 

		sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379  \
		--cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/peer.crt \
		--key=/etc/kubernetes/pki/etcd/peer.key  snapshot save /opt/etcd-backup.db

4. check whether the etcd database backup has been take successfully or not.

		ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/etcd-backup.db
		etcdutl --write-out=table snapshot status /opt/etcd-backup.db    ### using etcdutl 

5. Snapshot restoration process is mentioned below. restore the etcd backup using below command. 

- stop API server instances
- restore state of etcd instances
- restart API server instances

		ETCDCTL_API=3 etcdctl --data-dir=/var/lib/from-backup snapshot restore /opt/etcd-backup.db
		etcdutl snapshot restore  --data-dir=/opt/etcdbackup  etcd-backup.db  ### using etcdutl 

6. Edit the YAML manifest of the etcd Pod, which can be found at /etc/kubernetes/manifests/etcd.yaml. Change the value of the attribute spec.volumes.hostPath with the name etcd-data from the original value /var/lib/etcd to /var/lib/from-backup. 

		cd /etc/kubernetes/manifests/
		sudo vim etcd.yaml
		...
		spec:
		volumes:
		...
		- hostPath:
		    path: /var/lib/from-backup ### change this location to backup folder
		    type: DirectoryOrCreate
		  name: etcd-data
		...

7. check the etcd pods status. it will in running status. if it is not in running state then delete it. it will recreate the pods. 

		kubectl get po etcd-kmaster1 -n kube-system

