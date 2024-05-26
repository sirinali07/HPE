There are two types of Addons:
* Core Addons maintained and officially supported by the MicroK8s team at Canonical, and
* Community Addons.</br>

Refer to (https://microk8s.io/docs/addons) for further details.

To be as lightweight as possible, MicroK8s only installs the basics of a usable Kubernetes install like `api-server`, `controller-manager`, `scheduler`, `kubelet`, `cni`, `kube-proxy`.
While this does deliver a pure Kubernetes experience with the smallest resource footprint possible, there are situations where you may require additional services. MicroK8s caters for this with the concept of “Addons” - extra services which can easily be added to MicroK8s. These addons can be enabled and disabled at any time, and most are pre-configured to ‘just work’ without any further set up.

### Task 1: Managing Core Add-ons and Community Add-ons
To enable the CoreDNS addon:
```
microk8s enable DNS
```
These add-ons can be disabled at anytime using the disable command:
```
microk8s disable DNS
```
You can check the list of available and installed addons at any time.
```
microk8s status
```

### Task 2: Managing Community Add-ons
Enable community add-on
```
microk8s enable community
```
You can check the list of available community addons at any time.
```
microk8s status
```
Deactivate coonection to the Community addons
```
microk8s disable community
```
