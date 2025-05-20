### 🔐 Kubernetes RBAC Cheat Sheet

- **Subject**: User | Group | ServiceAccount  
- **Resources**: Deployments | Pods  
- **Verbs**: create | delete | update | patch  
- **Scope**:  
  - **Namespaced** → `Role` & `RoleBinding`  
  - **Cluster-wide** → `ClusterRole` & `ClusterRoleBinding`  
- **Aggregation**:  
  - Combine multiple `ClusterRoles` using `aggregationRule`


### Default cluster role 
1. cluster-admin
2. Admin
3. edit
4. view 

### User Creation 
1. Generate the private key and certificate signing request.
   
		openssl genrsa -out john.key 2048
		openssl req -new -key john.key -out john.csr -subj "/CN=john/O=developer"

2. submit the csr to kubernetes clusters.
   
		kubectl create -f - <<EOF
		apiVersion: certificates.k8s.io/v1
		kind: CertificateSigningRequest
		metadata:
  		 name: john # example
		spec:
		  request: $(cat john.csr | base64 | tr -d '\n')
		  signerName: kubernetes.io/kube-apiserver-client
		  expirationSeconds: 86400  
		  usages:
		  - client auth
		EOF

3. Get the csr and approve it.

		kubectl get csr	
		kubectl certificate approve john

4. Retrieve the certificate for john after approval.
   
		kubectl get csr john -o jsonpath='{.status.certificate}' | base64 -d  > john.crt

### Kubeconfig file preparation for john

1. Embed the user certificate and key in kubeconfig file. 

		kubectl --kubeconfig=john.kubeconfig config set-credentials john \
		--client-key=john.key --client-certificate=john.crt --embed-certs=true

2. Embed the cluster endpoint and cluster certificate authority data into kubeconfig file. 

		kubectl --kubeconfig=john.kubeconfig config set-cluster dev-cluster \
		--server=https://0.0.0.0:6443 --embed-certs=true --certificate-authority=/etc/kubernetes/pki/ca.crt

3. configure the context in kubecofig file. 

		kubectl --kubeconfig=john.kubeconfig config set-context dev --cluster=dev-cluster --user=john
		kubectl --kubeconfig=john.kubeconfig config use-context dev

### Assiging the roles for john user
1. Create the namespace and role.
   
		kubectl create ns devapp
		kubectl create role johnrole --verb=get,list,watch,create --resource=pods -n devapp

2. Create the rolebinding for john user. 

		kubectl create rolebinding johnrolebinding --role=johnrole --user=john -n devapp

3. Validate the given permission. 

		kubectl auth can-i --list --as john
		kubectl --kubeconfig=john.kubeconfig auth can-i --list  -n devapp
		kubectl --kubeconfig=john.kubeconfig auth can-i create pods  -n devapp


### Aggregating RBAC Rules 

1. Create the cluster role with name list-pods and delete-services.

		kubectl create clusterrole list-pods --verb=list --resource=pods
		kubectl create clusterrole delete-services --verb=delete --resource=services

2. assign the label to clusterrole.

		kubectl label clusterrole list-pods rbac-pod-list="true"
		kubectl label clusterrole delete-services rbac-service-delete="true"

3. Create the ClusterRole with aggregated rules.

	 	kubectl create clusterrole pods-services-aggregation-rules --aggregation-rule=rbac-service-delete="true"
   
4. edit the clusterrole pods-services-aggregation-rules and add the matchLabels for list-pods cluster role as well.

		kubectl edit clusterrole pods-services-aggregation-rules
		aggregationRule:
		  clusterRoleSelectors:
		  - matchLabels:
		      rbac-service-delete: "true"
		  - matchLabels:                 ### Add this line for list-pods clusterrole 
		      rbac-pod-list: "true"      ### Add this line for list-pods clusterrole 
		apiVersion: rbac.authorization.k8s.io/v1
		kind: ClusterRole
		metadata:
		  creationTimestamp: "2025-05-03T05:11:37Z"
		  name: pods-services-aggregation-rules
		  resourceVersion: "538310"
		  uid: 489c636e-aae6-4fba-a01e-adeffa44c5ef
		rules:
		- apiGroups:
		  - ""
		  resources:
		  - services
		  verbs:
		  - delete

	
5. Describe the aggregated cluster role and check. this will include the permission of both clusterroles delete-services and list-pods.

		kubectl describe clusterrole pods-services-aggregation-rules

### Notes Section
1. user & group is not stored in etcd but serviceaccount will get stored as object in etcd.
   








