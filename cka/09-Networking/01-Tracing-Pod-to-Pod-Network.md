### Tracing pod to pod traffic on the same node

1. Deploy the two pods
```
kubectl run web1 --image=nginx
kubectl run web2 --image=nginx
```
![image](https://github.com/user-attachments/assets/7f3d28af-c1de-40a5-93a0-ac27d939265b)


2. Both pods are running in the same worker node. Now, Login in the worker1 and get the process id of web1.
```
crictl ps | grep web1
crictl inspect 5341f5d7617cf  | grep "pid"
```
![image](https://github.com/user-attachments/assets/a4e980fd-991b-487e-8173-c07fdfa8f65f)


3. Get the veth name which is in inside the pod.
```
nsenter -t 2799 -n ip addr show eth0
```
![image](https://github.com/user-attachments/assets/ce12a712-d2c8-4676-bcb4-86e5afae53c6)


4. get the host-side veth interface of pods. 
```
ip link | grep "^8:" -A1
ip route
```
![image](https://github.com/user-attachments/assets/a697b5df-bff7-41d7-b96c-38cf4b096d3b)

![image](https://github.com/user-attachments/assets/3f292fae-2b46-4b14-9954-fbad187381f4)

5. now login in web2 pods and hit the web1 pods using below commands.
```
while true; do curl -s http://172.17.235.158; sleep 1; done
```
![image](https://github.com/user-attachments/assets/b1695605-3278-47f0-89a8-3b86ad6c8be2)

6. we are already having the details of pods web1 veth. So capture the traffic 
```
tcpdump -i cali15104693972 src host 172.17.235.156 and port 80
tcpdump -i any src host 172.17.235.156 and dst host 172.17.235.158 and port 80
```
![image](https://github.com/user-attachments/assets/7505176f-b558-4987-889e-c2403a9d6005)
![image](https://github.com/user-attachments/assets/93da525c-4328-4948-9933-f232b3a31fea)


### Tracing pod to pod communication on different nodes

### Pod Networking Diagram 


![image](https://github.com/user-attachments/assets/a75ba65b-926a-49be-a591-abe232523330)








