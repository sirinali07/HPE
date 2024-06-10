## Upgradation to the Latest Stable Version (Reef/stable)
To begin with make sure an older version of Microceph is installed.

Before upgrading, set noout to prevent Ceph from marking OSDs as out during the upgrade:
```
sudo ceph osd set noout
```
```
sudo ceph -s
```
Refresh Microceph to the latest stable version:
```
sudo snap refresh microceph --channel=reef/stable
```
Check the versions and status post-upgrade:
```
sudo ceph versions
```
```
sudo microceph --version
```
```
sudo snap list
```
```
sudo microceph status
```
```
sudo ceph status
```
Unset noout
```
sudo ceph osd unset noout
```
```
sudo ceph status
```
