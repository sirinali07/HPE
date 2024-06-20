## Deploy and Integrating AWS EBS CSI Storage with Kubernetes using Juju

### Task 1 : Deploy the AWS EBS storage charm:
```
juju deploy aws-k8s-storage --trust
```

Relate the storage charm to other components:
```
juju relate aws-k8s-storage:certificates easyrsa:client
```
```
juju relate aws-k8s-storage:kube-control kubernetes-control-plane:kube-control
```
```
juju relate aws-k8s-storage:aws-integration aws-integrator:aws
```

Check the status of the deployment:
```
juju status
```
Verify the storage classes integrated with kubernetes:
```
kubectl get sc
```

### Task 2 : Create PVC for Kubernetes Pod

Create a file named `ebs-pvc.yaml` with the following content:
```
vi ebs-pvc.yaml
```
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: ebs-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-aws-ebs-default

```
Apply the PVC configuration:
```
kubectl apply -f ebs-pvc.yaml
```
Verify the Persistent Volume and Claims:
```
kubectl get pv
```
```
kubectl get pvc
```

### Task 3 : Create Pod to store data 
Create a file named `storage-pod.yaml` with the following content:
```
vi storage-pod.yaml
```
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: ebs-pod
  labels:
    app: nginx-app
spec:
  volumes:
    - name: ebs-pv-storage
      persistentVolumeClaim:
        claimName: ebs-claim
  containers:
     - name: ebs-container
       image: nginx
       ports:
          - containerPort: 80
            name: "http-server"
       volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: ebs-pv-storage
```


Apply the Pod configuration:
```
kubectl apply -f storage-pod.yaml
```
Verify the Pod:
```
kubectl get pods
```
```
kubectl describe pod ebs-pod
```
Test writing data to the EBS volume:
```
kubectl exec -it ebs-pod -- bash
```
```
echo "this is ebs checking" > /usr/share/nginx/html/file.txt

exit
```
Verify the content:
```
kubectl exec -it ebs-pod -- cat /usr/share/nginx/html/file.txt
```
