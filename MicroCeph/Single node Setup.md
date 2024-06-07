## Single node Setup

### Task 1: Launching Instances on AWS
*	Launch `1 Instance` with OS version Ubuntu 22.04 LTS in your preferred region.
*	Storage: `20 GB`
*	Instance type: `t3.medium` – minimum requirement [ For Good Performance choose : t3.xlarge ] 
*	Enable SSH, HTTP, and HTTPS in the Security group.
*	Also, instead of opening all ports you can open these ports internally.
*	Type: Custom TCP
*	Source type: Anywhere
*	Port Number:


 | PORT(S)          | SERVICE                         |
|------------------|---------------------------------|
| 16443            | API server                      |
| 10250            | kubelet                         |
| 10255            | kubelet                         |
| 25000            | cluster-agent                   |
| 12379            | etcd                            |
| 10257            | kube-controller                 |
| 10259            | kube-scheduler                  |
| 19001            | dqlite                          |
| 4789/udp         | calico                          |
| 10248            | kubelet                         |
| 10249            | kube-proxy                      |
| 10251            | kube-scheduler                  |
| 10252            | kube-controller                 |
| 10256            | kube-proxy                      |
| 2380             | etcd                            |
| 1338             | containerd                      |
| 30000-32767      | NodePort                        |
| 6789             | Ceph Monitor (MON)              |
| 3300             | Ceph Monitor (MON)              |
| 9283             | Ceph Manager (MGR)              |
| 6800-7300        | Ceph OSD (Object Storage Daemon)|
| 6800-7300        | Ceph Metadata Server (MDS)      |
| 7480             | Ceph RADOS Gateway (RGW) (HTTP) |
| 8080             | Ceph RADOS Gateway (RGW) (HTTPS)|
| 9000             | Prometheus                      |
| 9001             | Prometheus                      |
| 9090             | Prometheus                      |
| 9100             | Prometheus                      |
| 22               | SSH                             |
| 80               | HTTP                            |
| 443              | HTTPS                           |
| 10248-10259      | kubelet/kube-proxy              |
| 6800-7568        | Ceph (OSD/MDS extended)         |






Set the Hostname

```
sudo hostnamectl hostname microceph-node01
bash
```
### Task 2: Install Microceph
```
sudo snap install microceph --channel=quincy/stable
``` 
Next, prevent the software from being auto-updated
```
sudo snap refresh --hold microceph
``` 
Note: Allowing the snap to be auto-updated can lead to unintended consequences. In enterprise environments especially, it is better to research the ramifications of software changes before those changes are implemented.
Run the below command to see the available microceph commands:
```
microceph
``` 
Check the version of microceph
```
microceph --version
``` 
Check the status of Microceph
```
sudo microceph status
``` 

### Task 3: Initialize the cluster
```
sudo microceph cluster –help
```
This displays the help information for managing Microceph clusters.</br>
Begin by initializing the cluster with the cluster bootstrap command
```
sudo microceph cluster bootstrap
```
View the cluster information
```
sudo microceph cluster list
``` 
Then look at the status of the cluster with the status command:
```
sudo microceph status
```
Create loopback device for virtual disks: Use the below command to add a loop device with a size of 4GB and a count of 3 (optional: if you don't have physical disk)
```
sudo microceph disk add loop,4G,3
```
lsblk, is used to list information about block devices (like hard drives) in the system
```
lsblk
``` 
Attach 3 volumes of 5GB to the microceph node01 from the AWS management console. Ensure the volumes and the node belong to the same availability zone.
 
To verify, list information about block devices (like hard drives) in the system
```
lsblk
``` 
Add physical disks (/dev/nvmeXn1) to the Ceph storage cluster
```
sudo microceph disk add /dev/nvme1n1
```
```
sudo microceph disk add /dev/nvme2n1
```
```
sudo microceph disk add /dev/nvme3n1
``` 
List the disks
```
sudo microceph disk list
``` 
View the overall status of the Ceph cluster
```
sudo ceph status
``` 
Then look at the status of the cluster with the status command:
```
sudo microceph status
```
``` 
sudo microceph.ceph -s
```
``` 
sudo ceph -s
``` 
View detailed information about the health of the Ceph cluster. 
```
sudo ceph health detail
```
It provides information about any issues or warnings that might be affecting the cluster's health.
 
Then look at the status of the cluster with the status command:
```
sudo microceph status
``` 
View the overall status of the Ceph cluster
```
sudo ceph status
``` 
View the current usage of the Ceph cluster's storage
```
sudo ceph df
``` 
List all the pools in the Ceph cluster
```
sudo ceph osd pool ls
``` 
To view detailed information
```
sudo ceph osd pool ls detail
``` 
### Task 4 : Working with OSD pool
We will create a new pool named rbd_pool with a size of 16 (which means the data will be replicated across 16 OSDs) and a minimum of 16 placement groups. Placement groups (PGs) are a fundamental concept in Ceph for data distribution and replication
```
sudo ceph osd pool create rbd_pool 16 16
``` 
We can view the statistics about a specific pool in your Ceph cluster. 
```
sudo ceph osd pool stats rbd_pool
``` 
Let’s enable the rbd application for the rbd_pool pool. 
In MicroCeph, an "application" refers to a specific use case or type of data that will be stored in the pool. Here, rbd stands for Rados Block Device, which is used for providing block storage to clients.
```
sudo ceph osd pool application enable rbd_pool rbd
``` 
The above command initializes the rbd_pool pool for use with RBD (Rados Block Device). It sets up the necessary configuration and structures within the pool to support RBD.
```
sudo rbd pool init rbd_pool
```
Create an RBD image named rbd_volume with a size of 4GB in the rbd_pool pool.
```
sudo rbd create --size 4G rbd_pool/rbd_volume
```
```
sudo rbd ls rbd_pool
```
Now we will load the RBD (Rados Block Device) kernel module, which is necessary for interacting with RBD devices
```
sudo modprobe rbd
```
List the loaded kernel modules and filter the output to display only the ones related to RBD, confirming that the RBD module has been successfully loaded.
```
lsmod | grep rbd
``` 
The below command updates the package lists for apt and installs the ceph-common package, which contains utilities and libraries commonly used for interacting with Ceph clusters
```
sudo apt-get update && sudo apt-get install ceph-common
``` 
Check the status of the rbdmap.service, which is responsible for mapping RBD images to block devices on the system
```
sudo systemctl status rbdmap.service
``` 
List the contents of the /etc/ceph/ directory, where Ceph configuration files are typically stored.
```
ls /etc/ceph/
``` 
This keyring file contains authentication credentials for the Ceph admin user, which are necessary for administrative tasks.
```
sudo cp /var/snap/microceph/current/conf/ceph.client.admin.keyring  /etc/ceph/
```
Set the permissions of the ceph.client.admin.keyring file to 777, giving read, write, and execute permissions to all users.
```
sudo chmod 777 /etc/ceph/ceph.client.admin.keyring 
``` 
Copy the Ceph configuration file . The Ceph configuration file contains settings and configurations for the Ceph cluster.
```
sudo cp /var/snap/microceph/current/conf/ceph..conf /etc/ceph/
```
Set the permissions of the ceph.conf file to 777, providing read, write, and execute permissions to all users.
```
sudo chmod 777 /etc/ceph/ceph.conf 
``` 

Map the RBD (Rados Block Device) volume named rbd_volume from the rbd_pool pool to a block device on the system. After executing this command, the RBD volume will be accessible as a block device.
```
sudo rbd map rbd_pool/rbd_volume
``` 
View information about all block devices on the system, including disks and partitions.
```
lsblk
``` 
Launch the fdisk utility for managing disk partitions on the block device /dev/rbd0.
```
sudo fdisk /dev/rbd0 
``` 
Create a primary partition with the default values.
View information about all block devices on the system, including disks and partitions.
```
lsblk
``` 
Format the first partition (/dev/rbd0p1) on the RBD block device with the XFS file system.
```
sudo mkfs.xfs /dev/rbd0p1 
``` 
Create a directory named rbd_mount in the /tmp/ directory. This directory will be used as the mount point for mounting the RBD block device.
```
sudo mkdir /tmp/rbd_mount
```
Mount the first partition (/dev/rbd0p1) of the RBD block device to the /tmp/rbd_mount/ directory. Once mounted, the RBD volume will be accessible as a file system under /tmp/rbd_mount/
```
sudo mount /dev/rbd0p1 /tmp/rbd_mount/
``` 
View information about all block devices on the system, including disks and partitions.
```
lsblk
```
