With the v1.28 MicroK8s release a cis-hardening addon is included as part of the core addons.
This addon reconfigures the cluster nodes to comply with the CIS recommendations v1.24.

We will review each of the CIS recommendations so that the interested reader (auditors, security teams) can asses
the level at which the respective security concerns are addressed. This section also shows how to CIS harden MicroK8s versions
where the cis-hardening addon is not available for (pre 1.28 releases).

CIS recommendations are in one of the following categories:
* Control Plane Security Configuration, checks 1.x.y
* Etcd Node Configuration, checks 2.x.y. Tailored to dqlite instead of etcd.
* Control Plane Configuration, checks 3.x.y
* Worker Node Security Configuration, checks 4.x.y
* Kubernetes Policies, checks 5.x.y
  

### Task 1 : Check 1.1.1
Ensure that the API server pod specification file permissions are set to 600 or more restrictive (Automated)

In MicroK8s the control plane is not self-hosted and therefore it does not run in pods. Instead the API server is a
systemd service with its configuration found at `/var/snap/microk8s/current/args/kube-apiserver`. The permissions of this
file need to be set to 600 or more restrictive.</br>
```
sudo /bin/sh -c 'if test -e /var/snap/microk8s/current/args/kube-apiserver; then stat -c permissions=%a /var/snap/microk8s/current/args/kube-apiserver; fi'
```

### Task 2: Configure MicroK8s according to the CIS recommendations.
The addon is enabled with elevated priveleges
```
sudo microk8s enable cis-hardening
```
You would get a message as below.</br>
`CIS hardening configuration has been applied. All microk8s commands require sudo from now on.`

To set back the permissions to `ubuntu` user, execute the below command
```
sudo chown -R ubuntu:ubuntu /var/snap/microk8s
```
```
sudo /bin/sh -c 'if test -e /var/snap/microk8s/current/args/kube-apiserver; then stat -c permissions=%a /var/snap/microk8s/current/args/kube-apiserver; fi'
```
`Expected output: permissions=600`</br>

Inspect the CIS benchmark results with:
```
sudo microk8s kube-bench
```
All kube-bench arguments can be passed to the microk8s kube-bench command. For example to check the results of a single test, 1.2.3:
```
sudo microk8s kube-bench --check 1.2.3
```
To undo the CIS hardening disable the addon with elevate priveleges
```
sudo microk8s disable cis-hardening
```
