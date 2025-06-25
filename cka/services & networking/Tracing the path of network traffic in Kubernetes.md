ip netns list
lsns -t net


when i create the pods then cri creates network namespace and the cni takes lead and assign the ip address to pods and attach the pods with rest of network. 

physical network interface hold the root network namespace. when create the multicontainer pods then container shares the same network namespace along with port. 


https://medium.com/@sumuduliyan/kubernetes-networking-with-calico-623f4583ae8d
https://learnk8s.io/kubernetes-services-and-load-balancing
https://learnk8s.io/kubernetes-network-packets

