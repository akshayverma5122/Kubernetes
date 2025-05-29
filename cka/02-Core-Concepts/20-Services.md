## Services
- Kubernetes Services enables communication between various components within and outside of the application.
- Expose an application running in your cluster behind a single outward-facing endpoint, even when the workload is split across multiple backends.
- A Service can map any incoming port to a targetPort. By default and for convenience, the targetPort is set to the same value as the port field.
- The default protocol for Services is TCP; you can also use any other supported protocol like UDP. 
- if we are planning to use service without selector then we must create the endpoints slices seperately. The endpoint IP addresses cannot be the cluster IPs of other Kubernetes Services, because kube-proxy doesn't support virtual IPs as a destination.
- As with Kubernetes names in general, names for ports must only contain lowercase alphanumeric characters and -. Port names must also start and end with an alphanumeric character. For example, the names 123-abc and web are valid, but 123_abc and -web are not.
- The Service API, part of Kubernetes, is an abstraction to help you expose groups of Pods over a network.
- **Ingress** to control how web traffic reaches that workload. Ingress is not a Service type, but it acts as the entry point for your cluster. An Ingress lets you consolidate your routing rules into a single resource, so that you can expose multiple components of your workload, running separately in your cluster, behind a single listener.
- The **Gateway API** for Kubernetes provides extra capabilities beyond Ingress and Service. You can add Gateway to your cluster - it is a family of extension APIs, implemented using CustomResourceDefinitions - and then use these to configure access to network services that are running in your cluster. 
## EndpointSlices
- EndpointSlices are objects that represent a subset (a slice) of the backing network endpoints for a Service.
- By default, Kubernetes makes a new EndpointSlice once the existing EndpointSlices all contain at least 100 endpoints.
## Type of Services 
- **NodePort** - Exposes the Service on each Node's IP at a static port (the NodePort). To make the node port available, Kubernetes sets up a cluster IP address, the same as if you had requested a Service of type: ClusterIP. the Kubernetes control plane allocates a port from a range specified by --service-node-port-range flag (default: 30000-32767).
- **ClusterIP** - Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default that is used if you don't explicitly specify a type for a Service. You can expose the Service to the public internet using an Ingress or a Gateway.
- **LoadBalancer** - Exposes the Service externally using an external load balancer. Kubernetes does not directly offer a load balancing component; you must provide one, or you can integrate your Kubernetes cluster with a cloud provider. LB services create nodeport by default but you can optionally disable node port allocation for a Service of type: LoadBalancer, by setting the field spec.allocateLoadBalancerNodePorts to false. 
- **ExternalName** - Maps the Service to the contents of the externalName field (for example, to the hostname api.foo.bar.example). The mapping configures your cluster's DNS server to return a CNAME record with that external hostname value. No proxying of any kind is set up. Accessing externalName works in the same way as other Services but with the crucial difference that redirection happens at the DNS level rather than via proxying or forwarding.
- **Headless Services** - For headless Services, a cluster IP is not allocated, kube-proxy does not handle these Services, and there is no load balancing or proxying done by the platform for them. To define a headless Service, you make a Service with .spec.type set to ClusterIP (which is also the default for type), and you additionally set .spec.clusterIP to None. if we create the headleass service with selector then the endpoints controller creates EndpointSlices in the Kubernetes API and if we do no use selector then  the control plane does not create EndpointSlice objects. in the non selector service, dns server will create the cname, A and AAAA record to reach the pods. 

K8s Reference Docs:
- https://kubernetes.io/docs/concepts/services-networking/service/
