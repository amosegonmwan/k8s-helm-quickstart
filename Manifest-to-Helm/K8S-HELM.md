**1. Create k8s deployment:**

```yaml
#k8s-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: demo
  name: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: demo
    spec:
      containers:
      - image: wemadevops/kubenginx:1.0.0
        name: kubernetes
        resources: {}
status: {}
```

**2.Deploy the application**
```shell
kubectl apply -f k8s-deployment.yaml
kubectl get deployment
```
**3. Create a service:**
```yaml
#k8s-service.yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: demo
  name: demo-service
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: demo
  type: NodePort
status:
  loadBalancer: {}
```

**4. Deploy the service**
```shell
kubectl apply -f k8s-service.yaml
kubectl get svc
```
**5. Access the app on the browser**
```shell
http://<ip>:<nodePort>   #ip - ip address of the node where the pod is running
```
**6. Convert k8s yaml to Helm Chart**

**a) Create a helm chart:**
```shell
helm create demochart
```
**b)  View the yaml files create:**
```shell
tree demochart
```
**c) Update the `Chart.yaml`, `deployment.yaml`, `service.yaml` and `values.yaml`**

**charts.yaml** - update the description of the chart

**deployment.yaml** - disable the liveliness and readiness probe (delete)
                    - update the container port
                    - Update the image from
         image: "{{ .Values.image.repository }}:{{ .Values.image.tag}}"
                        to
         image: "{{ .Values.image.repository}}"
**values.yaml** - update the image repository
                - update the type of service and port under service tag

**d) Verify the version of the chart:**
```shell
helm show chart demochart
```
**e) Verify the validity of the chart:**
```shell
helm template 0.1.0 demochart     
```
**f) Run and install helm chart:**
```shell
helm install k8sapp demochart
```
**g) Verify that the app is running:**         
```shell
http://<ip>:<nodePort>
```














  - 

    kubectl apply -f k8s-service.yaml