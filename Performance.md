### Task 1: Acess the MicroK8s Dashboard
Enable the Dashboard
```
microk8s enable dashboard
```
Check the Dashboard resources are up and running
```
microk8s kubectl -n kube-system get po,svc
```
Edit the service `kubernetes-dashboard` to change `type` to `NodePort`
```
microk8s kubectl -n kube-system edit svc kubernetes-dashboard
```
To check the port number on which the service is enabled
```
microk8s kubectl -n kube-system get svc kubernetes-dashboard
```
Generate the token
```
token=$(sudo microk8s kubectl -n kube-system describe secret microk8s-dashboard-token | awk '$1=="token:"{print $2}')
echo $token
```
Go to browser paste public ip of the Node with nodeport

`https://(public-ip of the node):(node-port)`


