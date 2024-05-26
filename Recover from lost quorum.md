By default, a three-node MicroK8s cluster automatically becomes highly available (HA). In HA mode the default datastore (dqlite) implements a Raft based protocol where an elected leader holds the definitive copy of the database. Under normal operation copies of the database are maintained by two more nodes. If you permanently lose the majority of the cluster members that serve as database nodes (for example, if you have a three-node cluster and you lose two of them), the cluster will become unavailable. However, if at least one database node has survived, you will be able to recover the cluster with the following manual steps.

Note: that the following recovery process applies to clusters using the default (dqlite) datastore of MicroK8s only. This process does not recover any data you have in PVs on the lost nodes.

### Task 1: Deploy some workloads
Exceute the below command.
```
microk8s kubectl deploy dep1 --image nginx --replicas 3
```
Expose the workload
```
microk8s kubectl expose deploy dep1 --type NodePort --name dep1-service --port 80 --target-port 80
```
Cross verify by accessing the Node port from your favorite browser
```
microk8s kubectl get po,svc
```

### Task 2: Backup the database
Dqlite stores data and configuration files under `/var/snap/microk8s/current/var/kubernetes/backend/`. To make a safe copy of the current state log in to a surviving node and create tarball of the dqlite directory:
```
tar -cvf backup.tar /var/snap/microk8s/current/var/kubernetes/backend
```

### Task 3: Stop dqlite on all nodes
Stopping MicroK8s is done with:
```
microk8s stop
```

### Task 4: Set the new state of the database cluster
```
cp backup.tar /var/snap/microk8s/current/var/kubernetes/backend/
```
```
cd /
```
```
sudo rm -rf var/snap/microk8s/current/var/kubernetes/backend
```
```
sudo cp ~/backup.tar .
```
```
ls
```
```
sudo tar -xvf backup.tar
```
```
sudo ls /var/snap/microk8s/current/var/kubernetes/backend/
```

### Task 5: Reconfigure dqlite
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

### Task 6: Restart MicroK8s services
It should now be possible to bring the cluster back online with:
```
microk8s start
```

### Task 7: Check the workloads
```
microk8s kubectl get all
```
