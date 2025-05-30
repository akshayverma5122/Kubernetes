## kube-apiserver
- Kube-apiserver is the primary component in kubernetes. Kube-apiserver is responsible for **`authenticating`**, **`validating`** requests, **`retrieving`** and **`Updating`** data in ETCD key-value store. In fact kube-apiserver is the only component that interacts directly to the etcd datastore. The other components such as kube-scheduler, kube-controller-manager and kubelet uses the API-Server to update in the cluster in their respective areas.
  

  
### Installing kube-apiserver - systemd service

- If you are bootstrapping kube-apiserver using **`kubeadm`** tool, then you don't need to know this, but if you are setting up using the hardway then kube-apiserver is available as a binary in the kubernetes release page.
  - For example: You can downlaod the kube-apiserver v1.13.0 binary here [kube-apiserver](https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver)
    ```
    $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver
    ```

- In a Non-kubeadm setup, you can inspect the options by viewing the kube-apiserver.service
  ```
  $ cat /etc/systemd/system/kube-apiserver.service
  ```
  
   
- You can also see the running process and effective options by listing the process on master node and searching for kube-apiserver.
  ```
  $ ps -aux | grep kube-apiserver
  ```

 
### View kube-apiserver - Kubeadm
- kubeadm deploys the kube-apiserver as a pod in kube-system namespace on the master node.
  ```
  $ kubectl get pods -n kube-system
  ```
  
- You can see the options with in the pod definition file located at **`/etc/kubernetes/manifests/kube-apiserver.yaml`**
  ```
  $ cat /etc/kubernetes/manifests/kube-apiserver.yaml
  ```

### Accessing the kube-apiserver
#### 1. Using kubectl proxy
  - Execute the below command.
  
		kubectl proxy --port=8080
  
  - previous command will not leave the terminal. so execute the below command in another terminal.

		curl http://localhost:8080/api/

    The output is similar to this:

		{
		  "kind": "APIVersions",
		  "versions": [
		    "v1"
		  ],
		  "serverAddressByClientCIDRs": [
		    {
		      "clientCIDR": "0.0.0.0/0",
		      "serverAddress": "10.0.1.149:443"
		    }
		  ]
		}

#### 2. Without kubectl proxy
- create the Secret, requesting a token for the default ServiceAccount.

		kubectl create secret generic default-token   --type=kubernetes.io/service-account-token    --dry-run=client -o yaml |   kubectl annotate -f - kubernetes.io/service-account.name=default --local -o yaml |   kubectl apply -f -

- capture the apiserver endpoints and token.

		apiserver=$(kubectl config view --minify  -o jsonpath='{.clusters[0].cluster.server}')
		token=$(kubectl get secret default-token -o jsonpath='{.data.token}' | base64 -d)
		curl $apiserver/api --header "Authorization: Bearer $token" --insecure

The output is similar to this:

		{
		  "kind": "APIVersions",
		  "versions": [
		    "v1"
		  ],
		  "serverAddressByClientCIDRs": [
		    {
		      "clientCIDR": "0.0.0.0/0",
		      "serverAddress": "10.0.1.149:443"
		    }
		  ]
		}


#### 3. Accessing the API from a Pod

- Create and log in the pods with nginx image

		kubectl run web --image=nginx
		kubectl exec -it web -- /bin/bash

- Execute the below command inside the pods


		# Point to the internal API server hostname
		APISERVER=https://kubernetes.default.svc

		# Path to ServiceAccount token
		SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount

		# Read this Pod's namespace
		NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)

		# Read the ServiceAccount bearer token
		TOKEN=$(cat ${SERVICEACCOUNT}/token)

		# Reference the internal certificate authority (CA)
		CACERT=${SERVICEACCOUNT}/ca.crt

		# Explore the API with TOKEN
		curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api
    
- The output will be similar to this.


		{
		  "kind": "APIVersions",
		  "versions": ["v1"],
		  "serverAddressByClientCIDRs": [
		    {
		      "clientCIDR": "0.0.0.0/0",
		      "serverAddress": "10.0.1.149:443"
		    }
		  ]
		}
  


K8s Reference Docs:
- https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
- https://kubernetes.io/docs/concepts/overview/kubernetes-api/
- https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/
