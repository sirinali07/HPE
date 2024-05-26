## Upgrade a 3-node cluster
For production clusters, 
* Always make sure that you have a working backup of the Kubernetes cluster database before starting.
* To minimize the margin of error, and ensure that you can always rollback to a working state, only upgrade a single Kubernetes node at a time.
* Start with the Kubernetes control plane nodes. After all control plane nodes are upgraded, proceed with the Kubernetes worker nodes, upgrading them one-by-one (or, for larger clusters, in small batches) as well.
* Make sure that you update by one minor version at a time.
* Before proceeding with any upgrade, refer to the Kubernetes release notes to read about any breaking changes, removed and/or deprecated APIs, and make sure that they do not affect your cluster.
* Cordon and drain any nodes prior to upgrading and restore them afterwards, to ensure that the application workloads hosted in your Kubernetes cluster are not affected.
  

We have the following cluster, running on microk8s-node1, microk8s-node2 and microk8s-node3. Service running in the workload is nginx with 3 replicas
```
microk8s kubectl get node
```
```
microk8s kubectl get pod -o wide
```

### Task 1: Upgrade first node
We will the start the cluster upgrade with `microk8s-node1`.

Execute the below command to cordon the node (marking it with the NoSchedule taint, so that no new workloads are scheduled on it), as well as evicting all running pods to other nodes:
```
microk8s kubectl drain microk8s-node1 --ignore-daemonsets
```
```
microk8s kubectl get node
```
```
microk8s kubectl get pod -o wide
```
Note that no pods are seen running on `microk8s-node1`.</br>
Refresh the MicroK8s snap to track the 1.29/stable channel.
```
sudo snap refresh microk8s --channel 1.29/stable
```
```
microk8s kubectl uncordon microk8s-node1
```
```
microk8s kubectl get node
```

### Task 2: Upgrade second node
Follow the same steps as previously. Note that all kubectl commands can run from any node in the cluster.</br>
Drain and cordon the node
```
microk8s kubectl drain microk8s-node2 --ignore-daemonsets
```
Ensure that all workloads have been moved to other cluster nodes:
```
microk8s kubectl get node
```
```
microk8s kubectl get pod -o wide
```
Notice the output, showing SchedulingDisabled for `microk8s-node2`, and no pods running on it.
```
sudo snap refresh microk8s --channel 1.29/stable
```
Verify that `microk8s-node2` is also now running on 1.29.0:
```
microk8s kubectl uncordon microk8s-node2
```
```
microk8s kubectl get node
```

### Task 3: Upgrade third node
The process is exactly the same as with the previous two nodes</br>
Drain and cordon the node
```
microk8s kubectl drain microk8s-node3 --ignore-daemonsets
```
Ensure that all workloads have been moved to other cluster nodes:
```
microk8s kubectl get node
```
```
microk8s kubectl get pod -o wide
```
Notice the output, showing SchedulingDisabled for `microk8s-node3`, and no pods running on it.
```
sudo snap refresh microk8s --channel 1.29/stable
```
Verify that `microk8s-node3` is also now running on 1.29.0:
```
microk8s kubectl uncordon microk8s-node2
```
```
microk8s kubectl get node
```


### Task 4: Test a new deployment
```
microk8s kubectl create deploy --image dontrebootme/microbot:v1 microbot-2
```

### Revert to previous MicroK8s version. This will re-installed the previous snap revision, and restore all configuration files of the control plane services:
```
sudo snap revert microk8s
```
```
microk8s kubectl scale deploy microbot-2 --replicas 4
```
After deployment is finished, check:
```
microk8s kubectl get pod -l app=microbot-2 -o wide
```
```
microk8s kubectl get node
```
