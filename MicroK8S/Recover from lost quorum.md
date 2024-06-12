
By default, a three-node MicroK8s cluster is highly available (HA). In HA mode, the default datastore, dqlite, uses a Raft-based protocol where an elected leader holds the definitive copy of the database. Normally, copies of the database are also maintained by two additional nodes. If the majority of the cluster members serving as database nodes are permanently lost (e.g., in a three-node cluster, if two nodes are lost), the cluster will become unavailable. However, if at least one database node remains, you can recover the cluster using the following manual steps.

***Note:*** This recovery process applies only to clusters using the default dqlite datastore of MicroK8s. It does not recover any data stored in Persistent Volumes (PVs) on the lost nodes. Ensure that any lost nodes previously part of the cluster will not come back online. If a lost node can be reinstated, it must rejoin the cluster using the "microk8s leave", "microk8s add-node" and "microk8s join" commands.

### Task 1: Backup the database
Dqlite stores data and configuration files under `/var/snap/microk8s/current/var/kubernetes/backend/`. To make a safe copy of the current state log in to a surviving node and create tarball of the dqlite directory:
```
tar -cvf backup.tar /var/snap/microk8s/current/var/kubernetes/backend
```

### Task 2: Stop dqlite on all nodes
Stopping MicroK8s is done with:
```
microk8s stop
```

### Task 3: Set the new state of the database cluster
Edit the file `/var/snap/microk8s/current/var/kubernetes/backend/cluster.yaml` to remove the lost nodes entries leaving only the ones available

```
sudo vi /var/snap/microk8s/current/var/kubernetes/backend/cluster.yaml
```

```
sudo cat var/snap/microk8s/current/var/kubernetes/backend/cluster.yaml
```

### Task 4: Reconfigure dqlite
MicroK8s comes with a dqlite client utility for node reconfiguration. The command to run is:
```
sudo /snap/microk8s/current/bin/dqlite \
  -s 127.0.0.1:19001 \
  -c /var/snap/microk8s/current/var/kubernetes/backend/cluster.crt \
  -k /var/snap/microk8s/current/var/kubernetes/backend/cluster.key \
  k8s ".reconfigure /var/snap/microk8s/current/var/kubernetes/backend/ /var/snap/microk8s/current/var/kubernetes/backend/cluster.yaml"
```
The `/snap/microk8s/current/bin/dqlite` utility needs to be called with sudo and takes the following arguments:
* the endpoint to the (now stopped) dqlite service. We have used -s 127.0.0.1:19001 for this endpoint in the example above.
* the private and public keys needed to access the database. These keys are passed with the -c and -k arguments and are found in the directory where dqlite keeps the database.
* the name of the database. For MicroK8s the database is k8s.
* the operation to be performed in this case is “reconfigure”
* the path to the database we want to reconfigure is the current database under `var/snap/microk8s/current/var/kubernetes/backend`
* the end cluster configuration we want to recreate is reflected in the cluster.yaml we edited in the previous step.

### Task 5: Restart MicroK8s services
It should now be possible to bring the cluster back online with:
```
microk8s start
```

### Task 6: Check the workloads
```
microk8s kubectl get nodes
```
```
microk8s kubectl get all
```
