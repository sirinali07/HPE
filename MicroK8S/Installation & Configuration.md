## To begin, log in to AWS Console.

### Task 1: Launching Instances on AWS

MicroK8s runs in as little as 540MB of memory, but to accommodate workloads, it is recommended a system with at least 20G of disk space and 4G of memory.
* `1` Instances of `t3.medium` instance with OS version as `Ubuntu 22.04 LTS` in your preferred region.
* `Storage: 20 GB`
* Enable SSH, HTTP and HTTPS in Security group. 
* Also Instead of opening all ports you can open these ports internally.
* `Type: Custom TCP`
* `Source type: Anywhere`
* `Port Number` : </br>

|   PORT       |   SERVICE |                                  ACCESS RESTRICTIONS                                |
|--------------|-----------|-------------------------------------------------------------------------------------|
| `16443`      | API server| SSL encrypted. Clients need to present a valid password from a Static Password File |
| `10250`      | kubelet     |  Anonymous authentication is disabled. X509 client certificate is required.       |
| `10255`  | kubelet |  Read only port for the Kubelet. |
| `25000`  | cluster-agent  |  Proper token required to authorise actions. |
| `12379`  | etcd  |  SSL encrypted. Client certificates required to connect. |
| `10257`  | kube-controller  |  Serve HTTPS with authentication and authorization.|
| `10259`  | kube-scheduler  |  Serve HTTPS with authentication and authorization.|
| `19001`  | dqlite |  SSL encrypted. Client certificates required to connect.|
| `4789/udp`  |calico  | Calico networking with VXLAN enabled.|
| `10248`  | kubelet |  Localhost healthz endpoint.|
| `10249`  | kube-proxy |  Port for the metrics server to serve on.|
| `10251`  | kube-schedule |  Port on which to serve HTTP insecurely.|
| `10252`  | kube-controller | Port on which to serve HTTP insecurely.|
| `10256`  | kube-proxy |  Port to bind the health check server.|
| `2380`  | etcd |  Used for peer connections.|
| `1338`  | containerd |  Metrics port |
|`30000-32767` | NodePort| NodePort |


Set the Hostname 
```
sudo hostnamectl hostname microk8s-node1
```
```
bash
```

### Task 2: Install MicroK8s

MicroK8s can be installed with a snap. </br>
When installing the MicroK8s you can specify a channel. The channel specified is made up of two components; the track and the risk level. For example to install MicroK8s v1.28 with risk level set to stable you need to</br>
Note: We will be installing version 1.28 as this cluster would be used to upgrade to 1.29 as well
```
sudo snap install microk8s --classic --channel=1.28/stable
```
If you do not set a channel at install time, the snap install will default to the latest current stable version


### Task 3: Join the group

MicroK8s creates a group to enable seamless usage of commands that require admin privilege. To add your current user to the group and gain access to the .kube caching directory, run the following three commands:
```
sudo usermod -a -G microk8s $USER
```
```
sudo mkdir -p ~/.kube
```
```
sudo chown -f -R $USER ~/.kube
```
You will also need to re-enter the session for the group update to take place:
```
su - $USER
```
or
```
sudo reboot
```

### Task 4: Check the details

MicroK8s has a built-in command to display its status. During installation you can use the --wait-ready flag to wait for the Kubernetes services to initialise.
```
microk8s status --wait-ready
```
```
microk8s version
```
```
microk8s inspect
```
Check the below file which has the details of the node
```
cat /var/snap/microk8s/6683/var/kubernetes/backend/localnode.yaml
```
MicroK8s bundles its own version of kubectl for accessing Kubernetes. Use it to run commands to monitor and control your Kubernetes. For example, to view your node:
```
microk8s kubectl get nodes
```
Or to see the running services:
```
microk8s kubectl get services
```
MicroK8s uses a namespaced kubectl command to prevent conflicts with any existing installs of kubectl. If you don’t have an existing install, it is easier to add an alias (append to ~/.bash_aliases) like this</br>
Open your preferred shell configuration file using a text editor. For example, if you're using bash:
```
nano ~/.bashrc
```
Add the alias to the file:
```
alias mkubectl='microk8s kubectl'
```
Save and exit the editor.</br>
Then, to apply the changes, either reopen your terminal or run:
```
source ~/.bashrc
```
After this, you can use mkubectl in your terminal to execute kubectl commands via MicroK8s. For instance:
```
mkubectl get nodes
```

### Task 5: Deploy an app

Of course, Kubernetes is meant for deploying apps and services. You can use the kubectl command to do that as with any Kubernetes. Try installing a demo app:
```
microk8s kubectl create deployment nginx --image=nginx --replicas 3
```
It may take a minute or two to install, but you can check the status:
```
microk8s kubectl get pods
```
Expose the deployment
```
microk8s kubectl expose deploy dep1 --type NodePort --name dep1-service --port 80
```

### Task 6: Starting and Stopping MicroK8s

MicroK8s will continue running until you decide to stop it. You can stop and start MicroK8s with these simple commands.
```
microk8s stop
```
… will stop MicroK8s and its services. You can start again any time by running.
```
microk8s start
```
Note that if you leave MicroK8s running, it will automatically restart after a reboot. If you don’t want this to happen, simply remember to run microk8s stop before you power down.
