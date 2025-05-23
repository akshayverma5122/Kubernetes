### Prepare kubeconfig file from scratch 

###### 1. set cluster name, server and certificate-authority-data in config-demo kubeconfig file
###### For development cluster
    kubectl config --kubeconfig=config-demo set-cluster development \
    --server=https://1.2.3.4:6443 \
    --certificate-authority=/root/certs/ca.crt --embed-certs=true
###### For test cluster
    kubectl config --kubeconfig=config-demo set-cluster test \
    --server=https://5.6.7.8:6443 --insecure-skip-tls-verify=true
###### 2. set the username, client-certificate-data and client-key-data in config-demo kubeconfig file. 
###### For developer user
    kubectl config --kubeconfig=config-demo set-credentials developer \
    --client-certificate=/root/certs/developer/developer.crt \
    --client-key=/root/certs/developer/developer.key \
    --embed-certs=true

###### For experimenter user
    kubectl config --kubeconfig=config-demo set-credentials  experimenter \
    --username=exp --password=badpassword
###### 3. set the values like cluster,namespace,user details in dev-frontend, dev-storage and exp-test context    
###### for dev-frontend context
    kubectl config --kubeconfig=config-demo set-context dev-frontend \
    --cluster=development --namespace=frontend --user=developer
###### for dev-storage context
     kubectl config --kubeconfig=config-demo set-context dev-storage \
     --cluster=development --namespace=storage --user=developer
###### for exp-test context
    kubectl config --kubeconfig=config-demo set-context exp-test \
    --cluster=test --namespace=default --user=expirementer 
###### 4. Set the current context 
    kubectl config --kubeconfig=config-demo use-context dev-frontend
###### 5. Display the added details. 
    kubectl config --kubeconfig=config-demo view 
###### To see only the configuration information associated with the current context, use the --minify flag. 
    kubectl config --kubeconfig=config-demo view --minify             
###### Display the current context. 
    kubectl config --kubeconfig=config-demo current-context
###### 6. Change the current context to dev-storage and display the current context for verification. 
    kubectl config --kubeconfig=config-demo use-context dev-storage
    kubectl config --kubeconfig=config-demo current-context

### other config commands 
###### Get the cluster,context and user details from kubeconfig file. 
    kubectl config --kubeconfig=config-demo get-clusters
    kubectl config --kubecofig=config-demo get-users
    kubectl config --kubeconfig=config-demo get-contexts 
    kubectl config --kubeconfig=config-demo get-contexts -oname
###### Delete the specified cluster,context and user from the kubeconfig.
    kubectl config --kubeconfig=config-demo delete-cluster development
    kubectl config --kubeconfig=config-demo delete-context dev-storage
    kubectl config --kubeconfig=config-demo delete-user developer
###### or use unset command to delete the cluster, context and user from kubeconfig file. 
    kubectl config --kubeconfig=config-demo unset clusters.development
    kubectl config --kubeconfig=config-demo unset contexts.dev-storage
    kubectl config --kubeconfig=config-demo unset users.developer
###### Renames a context from the kubeconfig file.
    kubectl config --kubeconfig=config-demo rename-context exp-test research-test
    
###### Remaining commands available
    https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-set-em-
###### practice url 
    https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
