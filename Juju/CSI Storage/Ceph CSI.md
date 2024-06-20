## Deploy and Integrate Ceph CSI Storage with Kubernetes using Juju

### Task 1 : Deploy Ceph Storage Cluster:
Deploy Ceph Monitors:
```
juju deploy -n 3 ceph-mon --channel quincy/stable
```
* The `-n 3 flag` specifies deploying three MON units for redundancy.
* Replace `quincy/stable` with a different channel if desired.

Deploy Ceph OSDs:
```
juju deploy -n 3 ceph-osd --storage osd-devices=32G,2 --storage osd-journals=8G,1
```
Integrate Ceph Services:
```
juju integrate ceph-osd:mon ceph-mon:osd
```
```
juju integrate ceph-mon:client kubernetes-control-plane
```
### Task 2: Integrate Ceph CSI with Kubernetes
Deploy Ceph CSI charm:
```
juju deploy ceph-csi
```
Integrate Ceph CSI with Services, and with kubernetes control plane.
```
juju integrate ceph-csi:kubernetes kubernetes-control-plane:juju-info
```
```
juju integrate ceph-csi:ceph-client ceph-mon:client
```
Verify the Deployment:
```
juju status
```
Verify the storage classes integrated with kubernetes:
```
kubectl get sc
```
### Task 3: Create a Persistent Volume Claim (PVC)
Create a file named ceph-pvc.yaml with the following content:
```
vi ceph-pvc.yaml
```
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: ceph-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ceph-xfs
 ```
Apply the PVC configuration:
```
kubectl apply -f ceph-pvc.yaml
```
Check the status of the created PVC and the corresponding Persistent Volume (PV) 
```
kubectl get pv,pvc
```
### Task 3 : Create Pod to store data
Create a file named ceph-pod.yaml with the following content:
```

vi ceph-pod.yaml
```
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: ceph-pod
  labels:
    app: httpd-app
spec:
  volumes:
    - name: ceph-pv-storage
      persistentVolumeClaim:
        claimName: ceph-claim
  containers:
     - name: ceph-container
       image: nginx
       ports:
          - containerPort: 80
            name: "http-server"
       volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: ceph-pv-storage
```

Apply the Pod configuration:
```
kubectl apply -f ceph-pod.yaml
```
Verify and inspect the Pod:
```
kubectl get pods
```
```
kubectl describe pod ceph-pod
```
Test writing data to the ceph volume:
```
kubectl exec -it ceph-pod -- bash
```
```
echo "this is ceph storage testinging" > /usr/share/nginx/html/index.html

exit
```
### Task 4: Scale Down and Remove Applications

Check the Status of Deployed Applications in the previous steps:
```
juju status
```
* To scale down specific units of an application use the syntax: 
`juju remove-unit ceph-mon/<unit-number>`
* Replace <unit-number> with the specific number of the unit you want to remove.
* Juju will prompt you to confirm the removal of each application. Confirm by typing `yes` when prompted.

To completely remove an application and all its associated units (e.g., ceph-mon, ceph-osd, ceph-csi):
```
juju remove-application ceph-mon
```
```
juju remove-application ceph-osd
```
```
juju remove-application ceph-csi
```
After scaling down, use the below command again to verify that the applications and units have been successfully removed from your Juju model.
```
juju status
```



