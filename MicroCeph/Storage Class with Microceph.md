## Storage Class with Microceph

### Task 1: Install MicroK8s on all the nodes
```
sudo snap install microk8s --classic --channel=1.28/stable
```
```
sudo usermod -a -G microk8s $USER
```
```
sudo mkdir -p ~/.kube
```
```
sudo chown -f -R $USER ~/.kube
``` 
```
sudo reboot
```
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
cat /var/snap/microk8s/6750/var/kubernetes/backend/localnode.yaml
```
```
microk8s kubectl get services
``` 
Repeat the above commands for microceph-node02 and microceph-node03.
 
### Task 2: Microk8s Cluster
There are three components necessary for a highly available Kubernetes cluster:
* There must be more than one node available at any time.
* The control plane must be running on more than one node so that losing a single node would not render the cluster inoperable.
* The cluster state must be in a datastore that is itself highly available.
  
On the microceph-node01 node, run:
```
microk8s add-node
``` 
This will output a join command with a generated token. Copy this command and run it from the microceph-node02 (make sure to execute it without a --worker flag).
Joining a node to the cluster should only take a few seconds. Afterwards you should be able to see all two nodes have joined.
```
microk8s kubectl get no
``` 
Again, On the microk8s-node1 node, run:
```
microk8s add-node
```
This will again output a join command with a generated token. Copy this command and run it from the microceph-node3 (make sure to execute it without a --worker flag). Now you should be able to see all three nodes have joined.
  
Afterwards you should be able to see all the three nodes have joined.
```
microk8s kubectl get no
``` 
Run the status command to check the status
```
microk8s status
```

### Task 3: Storage Class
```
sudo microk8s kubectl get sc
```
The above command will display “no resources found”. Enable the Rook Ceph addon in MicroK8s. 
```
sudo microk8s enable rook-ceph
``` 
This command sets up a connection to an external Ceph cluster.
```
sudo microk8s connect-external-ceph
```
Retrieve the status of rook-ceph pods, run the following command.
```
sudo microk8s kubectl get po -A
``` 
Check the storage classes again to verify that the Rook Ceph storage classes have been added or configured correctly
```
sudo microk8s kubectl get sc
```
```
sudo microk8s kubectl describe sc
```
Create a YAML configuration that defines a PersistentVolumeClaim (PVC) and a Deployment for a simple application using a Nginx container. The PVC is intended to use Ceph RBD as its storage backend.
```
vi storageclass.yml
```
Add the below lines, save and exit using esc + :wq!
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: microceph-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ceph-rbd
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: microceph-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: microceph-pod
  template:
    metadata:
      labels:
        app: microceph-pod
    spec:
      volumes:
      - name: microceph-pv-storage
        persistentVolumeClaim:
           claimName: microceph-claim
      containers:
      - name: microceph-container
        image: nginx
        ports:
          - containerPort: 80
            name: "http-server"
        volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: microceph-pv-storage

```
Apply the configuration defined in storageclass.yml to your Kubernetes cluster
```
sudo microk8s kubectl apply -f storageclass.yml
``` 
List all Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) in your Kubernetes cluster.
```
sudo microk8s kubectl get pv,PVC
```
List all Pods in your Kubernetes cluster
```
sudo microk8s kubectl get po
``` 
