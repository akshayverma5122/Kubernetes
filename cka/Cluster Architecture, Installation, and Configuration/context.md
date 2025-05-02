###### Practice Excersize 
    https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

### Prepare kubeconfig file from scratch 

###### 1. set cluster name, server and certificate-authority-data in config-demo kubeconfig file
###### For development cluster
    kubectl config --kubeconfig=config-demo set-cluster development \
    --server=https://1.2.3.4:6443 \
    --certificate-authority=/root/certs/ca.crt --embed-certs=true
###### For test cluster
    kubectl config --kubeconfig=config-demo set-cluster test \
    --server=https://5.6.7.8:6443 \
    --certificate-authority=/root/certs/ca.crt --embed-certs=true


###### 2. set the username, client-certificate-data and client-key-data for developer and experimenter
    kubectl config --kubeconfig=config-demo set-credentials developer
    kubectl config --kubeconfig=config-demo set-credentials experimenter
###### 3. set context name, cluster and user   
    kubectl config --kubeconfig=config-demo set-context dev-frontend
    kubectl config --kubeconfig=config-demo set-context dev-storage
    kubectl config --kubeconfig=config-demo set-context exp-test
    

    
###### Display the current context. 
    kubectl config --kubeconfig=config-demo current-context
###### Delete the specified cluster,context and user from the kubeconfig.
    kubectl config --kubeconfig=config-demo delete-cluster development
    kubectl config --kubeconfig=config-demo delete-context test
    kubectl config --kubeconfig=config-demo delete-user developer
    
