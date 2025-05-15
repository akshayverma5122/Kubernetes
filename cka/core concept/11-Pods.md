# Pods
- Kubernetes doesn't deploy containers directly on the worker node.
- Pod will have a one-to-one relationship with containers running your application.
- **Multi-Container PODs** - A single pod can have multiple containers except for the fact that they are usually not multiple containers of the **`same kind`**.
- Multiple containers should only be scheduled together in a single Pod if they are tightly coupled and need to share resources such as disk.
- A Pod is a group of one or more application containers (such as Docker) and includes shared storage (volumes), IP address and information about how to run them. Those resources include:
  
   1. Shared storage, as Volumes,
   2. Networking, as a unique cluster IP address, 
   3. Information about how to run each container, such as the container image version or specific ports to use
   4. cgroups 
- Pod can contain init containers that run during Pod startup. You can also inject ephemeral containers for debugging a running Pod
- Restarting a container in a Pod should not be confused with restarting a Pod. A Pod is not a process, but an environment for running container(s). A Pod persists until it is deleted.
- You should set the .spec.os.name field to either windows or linux to indicate the OS on which you want the pod to run. These two are the only operating systems supported for now by Kubernetes.
- Controllers for workload resources create Pods from a pod template and manage those Pods on your behalf.
- Most of the metadata about a Pod is immutable. For example, you cannot change the namespace, name, uid, or creationTimestamp fields.
- Multi container pods - init container, sidecar container.
- single container pods types - static pods, ephemeral pods 
  
## Deploy Pods using imperative commands 
- Start the web1 pods with nginx image without exposing the container port

		kubectl run web1 --image=nginx -n webserver

- Start the web2 pod with nginx image with exposing the container port on 80.

		kubectl run web2 --image=nginx --port=80 -n webserver

- Start the web3 pod and set environment variables "key1=value1" and "key2=value2" in the container

		kubectl run web3 --image=nginx --env="key1=value1" --env="key2=value2" -n webserver
		
- Start the web4 pod and set labels "app=middleware" and "env=prod" in the container
  
		kubectl run web4 --image --labels="app=middleware,env=prod" -n webserver

- Dry run; print the corresponding API objects without creating them

		kubectl run web5 --image=nginx --port=80 --labels="app=middleware,env=prod" --env="key1=value1" -n webserver --dry-run=client -oyaml

- Start a busybox pod and keep it in the foreground, don't restart it if it exits

		kubectl run -it web6 --image=nginx --restart=Never

- Start the dns pod using the default command, but use custom arguments (arg1 .. argN) for that command

		kubectl run dnspod --rm -it --image=tutum/dnsutils --restart=Never -- nslookup kubernetes.default

- Start the nginx pod using a different command and custom arguments
  
		kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>
	

	

		
    
K8s Reference Docs:
- https://kubernetes.io/docs/concepts/workloads/pods/pod/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/


