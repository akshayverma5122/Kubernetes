### How Scheduling Works
- What do you do when you do not have a scheduler in your cluster?
  - Every POD has a field called NodeName that by default is not set. You don't typically specify this field when you create the manifest file, kubernetes adds it automatically.
  - Once identified it schedules the POD on the node by setting the nodeName property to the name of the node by creating a binding object.
### No Scheduler
- You can manually assign pods to node itself. Well without a scheduler, to schedule pod is to set nodeName property in your pod definition file while creating a pod.
- Another way to schedule the pods is by creating the pods and its binding object.
- **nodeName**  - if nodeName field is mentioned in pod spec then scheduler ignore the pods instead kubelet directly schdule the pods in that particular node. if that particluar node is not available, then pods will either not running or will get deleted automatically.
