###### Practice Excersize 
        https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

###### Display the current context. 
    kubectl config --kubeconfig=config-demo current-context
###### Delete the specified cluster,context and user from the kubeconfig.
    kubectl config --kubeconfig=config-demo delete-cluster development
    kubectl config --kubeconfig=config-demo delete-context test
    kubectl config --kubeconfig=config-demo delete-user developer
    
