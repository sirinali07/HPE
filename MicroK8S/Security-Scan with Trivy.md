### Task 1: Enable the Trivy addon
```
microk8s enable community
```
```
microk8s enable trivy
```
Once the operator has been installed you can verify that it is running by inspecting the pods.The output should show that the namespace ‘trivy-system’ and the pod ‘trivy-operator’ have been created
```
microk8s kubectl get all -A
```

### Task 2: Using the Trivy Operator
The Trivy-Operator runs trivy security tools and incorporates their outputs into Kubernetes CRDs (Custom Resource Definitions). From there, security reports are accessible through the Kubernetes API, making it easy for users to find and view the risks that relate to different resources in a Kubernetes-native way.

In order to perform scans, Trivy needs to find resources to scan. By default it is configured to scan resources in all namespaces.
To test this, you can try deploying the Kubernetes bootcamp image:
```
microk8s kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```
Vulnerability reports are available within the cluster. The output will list the vulnerability issues detected in each of the namespaces in the cluster.
```
microk8s kubectl get vulnerabilityreports --all-namespaces -o wide
```
As Trivy exposes the details of scans through the API, you can use the ‘describe’ command to retrieve more details
```
microk8s kubectl describe vulnerabilityreports --all-namespaces
```

### Task 3: Run a configuration audit
As with vulnerability scans, configuration audits are exposed through the Kubernetes API. The output shows the number configuration issues identified by Trivy. 
```
microk8s kubectl get configauditreports --all-namespaces -o wide
```
It is possible to get more details about each of the issues detected by running the command below.
```
microk8s.kubectl describe configauditreports --all-namespaces
```
