### Tracing pod to pod traffic on the same node

1. Deploy the two pods
```
kubectl run web1 --image=nginx
kubectl run web2 --image=nginx
```
![image](https://github.com/user-attachments/assets/b647409d-0d9c-4f1a-b7f1-cb1ec4c61b2a)

2. Both pods are running in the same worker node. Now, Login in the worker1 and get the process id of web1.
```
crictl ps | grep web1
crictl inspect <containerid> | grep "pid"
```
![image](https://github.com/user-attachments/assets/e1f78ba2-a770-4a90-a7c0-b1084bf39305)

3. Get the veth name which is in inside the pod.
```
nsenter -t <pid> -n ip addr show eth0
```
![image](https://github.com/user-attachments/assets/143b22cc-1314-4b6c-9b29-09d3981fd237)

4. get the host-side veth interface of pods. 
```
ip link | grep "^10:" -A1
```
![image](https://github.com/user-attachments/assets/c958a45f-5f4d-4c1e-bdfd-944a871cbd49)

5. now login in web2 pods and hit the web1 pods using below commands.
```
while true; do curl -s http://172.17.235.139; sleep 1; done
```
6. we are already having the pods web1 veth. So capture the traffic 
```
tcpdump -i cali15104693972 src host 172.17.235.157
```





