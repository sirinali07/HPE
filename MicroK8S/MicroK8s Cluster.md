### Task 1: Launch Instances and configure MicroK8s on them

Refer to the below link to launch 2 more instances and change the hostname to `microk8s-node2` and `microk8s-node3`.</br>
https://github.com/Mehar-Nafis/Canonical/blob/main/MicroK8s/Installation%20%26%20Configuration.md</br>
Make sure the below command is executed on both nodes before you move to the next task here.
```
sudo snap install microk8s --classic --channel=1.28/stable
```

### Task 2: Adding a worker node

To create a cluster out of three or already-running MicroK8s instances, use the microk8s add-node. The MicroK8s instance on which this command is run will be the master of the cluster and will host the Kubernetes control plane. </br>
Execute the below command on `microk8s-node1`
```
microk8s add-node
```
This will return some joining instructions which should be executed on the MicroK8s instance that you wish to join to the cluster (NOT THE NODE YOU RAN add-node FROM)</br>
Make sure to pick up the command which has the `--worker` flag.</br>
Execute the join command on `microk8s-node2` </br>
Joining a node to the cluster should only take a few seconds. Afterwards, you should be able to see two nodes by executing the below command.
```
microk8s kubectl get no
```
Now to add the third node to the cluster, execute the below command on `microk8s-node1`
```
microk8s add-node
```
This will return some joining instructions which should be executed on the MicroK8s instance that you wish to join to the cluster (NOT THE NODE YOU RAN add-node FROM)</br>
Make sure to pick up the command which has the `--worker` flag.</br>
Execute the join command on `microk8s-node3`</br>
Afterwards you should be able to see all the three nodes have joined.
```
microk8s kubectl get no
```

### Task 2: Removing a node

First, on the node you want to remove,in this case `microk8s-node3` run microk8s leave. MicroK8s on the departing node will restart its own control plane and resume operations as a full single node cluster.
```
microk8s leave
```
To complete the node removal, call microk8s remove-node from the remaining nodes to indicate that the departing (unreachable now) node should be removed permanently.
```
microk8s remove-node <ip-addr>
```
Now on `microk8s-node2` run microk8s leave. 
```
microk8s leave
```
To complete the node removal, call microk8s remove-node from the remaining node `microk8s-node1` to indicate that the departing (unreachable now) node should be removed permanently.
```
microk8s remove-node <ip-addr>
```

### Task 3: High Availability

A highly available Kubernetes cluster is a cluster that can withstand a failure on any one of its components and continue serving workloads without interruption. There are three components necessary for a highly available Kubernetes cluster:
* There must be more than one node available at any time.
* The control plane must be running on more than one node so that losing a single node would not render the cluster inoperable.
* The cluster state must be in a datastore that is itself highly available.

On the `microk8s-node1` node, run:
```
microk8s add-node
```
This will output a join command with a generated token. Copy this command and run it from the `microk8s-node2` (make sure to execute it without a `--worker` flag). </br>
Joining a node to the cluster should only take a few seconds. Afterwards you should be able to see all two nodes have joined.
```
microk8s kubectl get no
```
Again, On the `microk8s-node1` node, run:
```
microk8s add-node
```
This will again output a join command with a generated token. Copy this command and run it from the `microk8s-node3` (make sure to execute it without a `--worker` flag). Now you should be able to see all three nodes have joined.
```
microk8s kubectl get no
```
Run the status command to check the status
```
microk8s status
```
You should be able to see something like below
```
high-availability: yes
  datastore master nodes: 10.128.63.86:19001 10.128.63.166:19001 10.128.63.43:19001
  datastore standby nodes: none`
```
All nodes of the HA cluster run the master control plane. A subset of the cluster nodes (at least three) maintain a copy of the Kubernetes dqlite database. Database maintenance involves a voting process through which a leader is elected. Apart from the voting nodes there are non-voting nodes silently keeping a copy of the database. These nodes are on standby to take over the position of a departing voter. Finally, there are nodes that neither vote nor replicate the database. These nodes are called spare. To sum up, the three node roles are:
* voters: replicating the database, participating in leader election
* standby: replicating the database, not participating in leader election
* spare: not replicating the database, not participating in leader election
Cluster formation, database syncing, voter and leader elections are all transparent to the administrator.

Since all nodes of the HA cluster run the master control plane the `microk8s *` commands are now available everywhere. Should one of the nodes crash we can move to any other node and continue working without much disruption.

Almost all of the HA cluster management is transparent to the admin and requires minimal configuration. The administrator can only add or remove nodes. To ensure the health of the cluster the following timings should be taken into account:
* If the leader node gets “removed” ungracefully, e.g. it crashes and never comes back, it will take up to 5 seconds for the cluster to elect a new leader.
* Promoting a non-voter to a voter takes up to 30 seconds. This promotion takes place when a new node enters the cluster or when a voter crashes.
  








