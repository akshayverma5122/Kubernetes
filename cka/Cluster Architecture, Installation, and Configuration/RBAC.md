#### Role Based Access Control (RBAC)  

1. Subject     >>> User, Group, ServiceAccount
2. Resources   >>> Deployment, pods 
3. Verbs       >>> create,delete,update,patch
4. Namespaced  >>> Role & Rolebinding 
5. Clusterwide >>> ClusterRole & ClusterRoleBinding
6. RBAC aggregation 

#### Default cluster role 
1. cluster-admin
2. Admin
3. edit
4. view 

#### User Creation 
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

kubectl create -f john.yaml

3. Get the csr and approve it.

kubectl get csr	
kubectl certificate approve john

4. Retrieve the certificate for john after approval. 

kubectl get csr john -o jsonpath='{.status.certificate}' | base64 -d  > john.crt

Kubeconfig file preparation for john:-
------------------------------------

1. Embed the user certificate and key in kubeconfig file. 

kubectl --kubeconfig=john.kubeconfig config set-credentials john --client-key=john.key --client-certificate=john.crt --embed-certs=true

2. Embed the cluster endpoint and cluster certificate authority data into kubeconfig file. 

kubectl --kubeconfig=john.kubeconfig config set-cluster dev-cluster --server=https://0.0.0.0:6443
kubectl --kubeconfig=john.kubeconfig config set-cluster dev-cluster --embed-certs=true --certificate-authority=/etc/kubernetes/pki/ca.crt

3. configure the context in kubecofig file. 

kubectl --kubeconfig=john.kubeconfig config set-context dev --cluster=dev-cluster --user=john
kubectl --kubeconfig=john.kubeconfig config use-context dev

Assiging the roles for john user:-
---------------------------------

1. Create the namespace and role.

kubectl create ns devapp
kubectl create role johnrole --verb=get,list,watch,create --resource=pods -n devapp

2. Create the rolebinding for john user. 

kubectl create rolebinding johnrolebinding --role=johnrole --user=john -n devapp

3. Validate the given permission. 

kubectl auth can-i --list --as john
kubectl --kubeconfig=john.kubeconfig auth can-i --list  -n devapp
kubectl --kubeconfig=john.kubeconfig auth can-i create pods  -n devapp


########################################## Aggregating RBAC Rules #############################################################

1. Create the cluster role with name list-pods and delete-services. 

vi list-pods.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: list-pods
  namespace: rbac-example
  labels:
    rbac-pod-list: "true"
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - list

vi delete-services.yaml 

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: delete-services
  namespace: rbac-example
  labels:
    rbac-service-delete: "true"
rules:
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - delete

2. Create the ClusterRole with aggregated rules. 

vi pods-services-aggregation-rules.yaml 

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pods-services-aggregation-rules
  namespace: rbac-example
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac-pod-list: "true"
  - matchLabels:
      rbac-service-delete: "true"
rules: []

3. Describe the aggregated cluster role and check. this will include the permission of both clusterroles delete-services and list-pods.

k describe clusterrole pods-services-aggregation-rules

################################################# Notes Section ##############################################################
1. user & group is not stored in etcd but serviceaccount will get stored as object in etcd.
2. 
############################################ Practice RBAC Commands ############################################################

Clusterrole & Clusterrolebinding -  
--------------------------------








