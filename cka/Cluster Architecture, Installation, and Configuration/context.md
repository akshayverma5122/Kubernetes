###### Practice Excersize 
    https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

###### Prepare kubeconfig file from scratch 
    kubectl config --kubeconfig=config-demo set-cluster development
    kubectl config --kubeconfig=config-demo set-cluster test
    
    kubectl config --kubeconfig=config-demo set-context dev-frontend
    kubectl config --kubeconfig=config-demo set-context dev-storage
    kubectl config --kubeconfig=config-demo set-context exp-test
    
    kubectl config --kubeconfig=config-demo set-credentials developer
    kubectl config --kubeconfig=config-demo set-credentials experimenter
    
###### Display the current context. 
    kubectl config --kubeconfig=config-demo current-context
###### Delete the specified cluster,context and user from the kubeconfig.
    kubectl config --kubeconfig=config-demo delete-cluster development
    kubectl config --kubeconfig=config-demo delete-context test
    kubectl config --kubeconfig=config-demo delete-user developer
    
