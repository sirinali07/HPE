Ensure you have Juju installed and configured. If not follow the LAB : (https://github.com/sirinali07/Cononical_HPE/blob/main/Juju/Juju%20Setup.md)

### Task 1 : Deploy Charmed Kubernetes with Juju
Create a file named `k8s.yaml` with the following content:
```
vi k8s.yaml
```
```yaml
description: Charmed Kubernetes overlay.
applications:
  kubernetes-worker:
    constraints:  root-disk=20G
    num_units: 2
  kubernetes-control-plane:
    constraints:  root-disk=20G
    num_units: 1
  etcd:
    constraints:  root-disk=20G
    num_units: 1
  kubeapi-load-balancer:
    constraints:  root-disk=20G
    num_units: 1
  easyrsa:
    constraints:  root-disk=20G
    num_units: 1
```

Download the `aws-overlay.yaml` file for the Charmed Kubernetes bundle:

Note: Refer the given URL for more information (https://ubuntu.com/kubernetes/docs/install-manual)

```
wget https://raw.githubusercontent.com/charmed-kubernetes/bundle/main/overlays/aws-overlay.yaml
```
```
juju add-model k8s-model --credential aws-cred aws/us-west-2
```
```
juju models
```
Deploy the charmed-kubernetes bundle using Juju, referencing the created YAML files and specifying the channel:
```
juju deploy charmed-kubernetes --channel 1.28/stable --overlay k8s.yaml --overlay aws-overlay.yaml --trust
```
* Replace `1.28/stable` with the desired Kubernetes channel if needed.
* The `--trust` flag indicates trust for the charm source.

Verify Deployment:

```  
juju status
```

### Task 2 : Accessing Kubernetes Cluster with kubectl on Juju Server
Install kubectl using the snap package manager:
```
sudo snap install kubectl --classic --channel 1.28/stable
```
Create the `.kube` directory in your current working directory of the user:
```
mkdir ~/.kube
```
Copy the kubeconfig file from the `Kubernetes control plane` to your local `Juju server`:
```
juju ssh kubernetes-control-plane/0 -- sudo cat ~/.kube/config > ~/.kube/config
```
`Note:` If the configuration file is not present in the ~/.kube/config directory or if you notice the local IP within the cluster URL in the copied config file, follow these steps:
(Alternative Steps)

List the Config File:
```
juju ssh kubernetes-control-plane/0 -- sudo ls ~/
```
Copy the Configuration File:
(Alternative Steps)
```
juju ssh kubernetes-control-plane/0 -- sudo cat ~/config > ~/.kube/config
```
Verify the contents of the kubeconfig file on `Juju Server`:
```
cat ~/.kube/config
```
Now you can use `kubectl` to interact with your Kubernetes cluster on `Juju Server`.

List Pods:
```
kubectl get pods
```
List Namespaces:
```
kubectl get ns
```

Deploy a Test Application by Juju.

Run an Nginx pod:
```
kubectl run pod-1 --image nginx
```
Expose the Nginx pod with a LoadBalancer:
```
kubectl expose pod pod-1 --port 80 --type LoadBalancer
```
`Note:` If the aws-integrator charm is present, Kubernetes will automatically create an AWS Elastic Load Balancer for actions that need a load balancer.








