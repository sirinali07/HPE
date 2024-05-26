### Task 1: Enable Ingress Add-on
Enable the Ingress add-on in MicroK8s:
```
microk8s enable ingress
```
Check the status
```
microk8s status
```

### Task 2: Create the Ingress
In the below Ingress rule, it uses path-based routing instead of host-based routing. The Ingress will route requests with the path /echo to the echo-service.
```
vi ingress.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: echo-server
  labels:
    app: echo-server
spec:
  containers:
  - name: echo-server
    image: hashicorp/http-echo
    args:
    - "-text=Hello from echo server!"
---
apiVersion: v1
kind: Service
metadata:
  name: echo-service
spec:
  selector:
    app: echo-server
  ports:
    - protocol: TCP
      port: 5678
      targetPort: 5678
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-ingress
spec:
  rules:
  - http:
      paths:
      - path: /echo
        pathType: Prefix
        backend:
          service:
            name: echo-service
            port:
              number: 5678
```
```
microk8s kubectl apply -f ingress.yaml
```
After applying the YAML, you can access the application at http://<your-microk8s-ip>/echo. Replace <your-microk8s-ip> with the IP address of your MicroK8s cluster.





