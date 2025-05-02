###### Practice Excersize 
    https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

### Prepare kubeconfig file from scratch 

###### 1. set cluster name, server and certificate-authority-data in config-demo kubeconfig file
###### For development cluster
    kubectl config --kubeconfig=config-demo set-cluster development \
    --server=https://1.2.3.4:6443 \
    --certificate-authority=/root/certs/ca.crt --embed-certs=true
###### For test cluster
    kubectl config --kubeconfig=config-demo set-cluster test --server=https://5.6.7.8:6443 --insecure-skip-tls-verify=true
###### 2. set the username, client-certificate-data and client-key-data in config-demo kubeconfig file. 
###### For developer user
    kubectl config --kubeconfig=config-demo set-credentials developer \
    --client-certificate=/root/certs/developer/developer.crt \
    --client-key=/root/certs/developer/developer.key \
    --embed-certs=true

###### For experimenter user
    kubectl config --kubeconfig=config-demo set-credentials  experimenter --username=exp --password=badpassword
###### 3. set the values like cluster,namespace,user details in dev-frontend, dev-storage and exp-test context    
###### for dev-frontend context
    kubectl config --kubeconfig=config-demo set-context dev-frontend --cluster=development --namespace=frontend -- 
    user=developer
###### for dev-storage context
     kubectl config --kubeconfig=config-demo set-context dev-storage --cluster=development --namespace=storage -- 
     user=developer
###### for exp-test context
    kubectl config --kubeconfig=config-demo set-context exp-test --cluster=test --namespace=default --user=expirementer
###### 4. Display the added details
    kubectl config --kubeconfig=config-demo view 
###### 5. Set the current context 
    kubectl config --kubeconfig=config-demo use-context dev-frontend
        
    
###### Display the current context. 
    kubectl config --kubeconfig=config-demo current-context
###### Delete the specified cluster,context and user from the kubeconfig.
    kubectl config --kubeconfig=config-demo delete-cluster development
    kubectl config --kubeconfig=config-demo delete-context test
    kubectl config --kubeconfig=config-demo delete-user developer
    
