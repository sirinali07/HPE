## Scaling and Encryption

### Task 1: Scaling
View information about all block devices on the system, including disks and partitions.
```
lsblk
``` 
### Task 2: Add a new disk to the Ceph cluster and wipe out any existing data on it.
Prepare the disk for use by Ceph.
```
sudo microceph disk add /dev/nvme4n1 –wipe
``` 
Display the current state of the Ceph cluster's OSD (Object Storage Daemon) tree, showing the hierarchy and status of OSDs.
```
sudo ceph osd tree
``` 
View information about the Ceph cluster's storage utilization, including the total storage capacity, used space, available space, and utilization percentage.
```
sudo ceph df
``` 
Check summary of the status of the Ceph cluster, including the health status, number of OSDs, number of placement groups (PGs), and overall cluster health
```
sudo ceph -s
``` 
View the status of the microceph service, which is responsible for managing and monitoring Ceph clusters
```
sudo microceph status
``` 
Remove the disk associated with OSD (Object Storage Daemon) ID 3 from the Ceph cluster. The --confirm-failure-domain-downgrade=true flag confirms the removal even if it leads to a downgrade in the failure domain
```
sudo microceph disk remove osd.3 --confirm-failure-domain-downgrade=true
``` 
Check the OSD tree after removing the disk to reflect the updated state of the cluster
```
sudo ceph osd tree
``` 
Check the status of the microceph service after performing the disk removal operation.
```
sudo microceph status
``` 
### Task 3: Disk Encryption
Attach a 5GB volume to the node manually thorugh the AWS management console.
View information about all block devices on the system, including disks and partitions.
```
lsblk
``` 
Establish a connection between the microceph snap and the dm-crypt interface. 
```
sudo snap connect microceph:dm-crypt
```
Restart the microceph.daemon service provided by the microceph snap.
```
sudo snap restart microceph.daemon
``` 
FDE for OSDs is activated by passing the optional --encrypt flag when adding disks:
```
sudo microceph disk add /dev/nvme5n1 --wipe --encrypt
```
```
sudo microceph disk list
```
```
sudo microceph status
```
```
sudo ceph status
```
