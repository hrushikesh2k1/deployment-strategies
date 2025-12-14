I am using minikube cluster for showing this.

## Enable the ingress, it will enable nginx ingress control by default
```
minikube enable ingress
```

## Create the deployment and service for version-1 with 4 replicas(deployment.yaml)
```
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: version-1
  labels:
    app: version-1
spec:
  replicas: 4
  selector:
    matchLabels:
      app: version-1
  template:
    metadata:
      labels:
        app: version-1
    spec:
      containers:
      - name: version-1
        image: registry.k8s.io/ingress-nginx/e2e-test-echo:v1.2.5@sha256:d7b3143152261e918e2197ef86840986ba7fbbd5ca72e0e79d6ec4ea99d2abc3
        ports:
        - containerPort: 80
        env:
          - name: NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: version-1
  labels:
    app: version-1
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: version-1
```

## Deploying the deployment and service
```
kubectl apply -f deployment.yaml
```

## Creating the deployment and service for version-2 with 4 replicas (canary.yaml)
```
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: version-2
  labels:
    app: version-2
spec:
  replicas: 4
  selector:
    matchLabels:
      app: version-2
  template:
    metadata:
      labels:
        app: version-2
    spec:
      containers:
      - name: version-2
        image: registry.k8s.io/ingress-nginx/e2e-test-echo:v1.2.5@sha256:d7b3143152261e918e2197ef86840986ba7fbbd5ca72e0e79d6ec4ea99d2abc3
        ports:
        - containerPort: 80
        env:
          - name: NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: version-2
  labels:
    app: version-2
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: version-2
```

## deploy the deployment and service with version-2 with 4 replicas (canary.yaml)
```
kubectl apply -f canary.yaml
```
<img width="813" height="539" alt="blue-1" src="https://github.com/user-attachments/assets/4379c1eb-72d7-4043-8bb2-de5a50903f49" />

## Creating the ingress resource for both version-1 & version-2 (ingress-v1.yaml)
```
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: version-1
  annotations:
spec:
  ingressClassName: nginx
  rules:
  - host: echo.prod.mydomain.com
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: verison-1
            port:
              number: 80
```
We are going to use this same ingress manifest for both the version, we will change the service to version-1 if we want to route traffic to pods with version-1 and vice versa.

<img width="453" height="430" alt="blue-2" src="https://github.com/user-attachments/assets/8cac2727-29bd-4e46-b83a-8d0dd49420d1" />


## Get the minikube ip
```
minikube ip
```

## ssh into minikube cluster
```
minikube ssh
```

## Execute the below command
Execute the following command to see the traffic going to particular version when we change the service in ingress respectively
```
for i in $(seq 1 10); do curl -s --resolve echo.prod.mydomain.com:80:$INGRESS_CONTROLLER_IP echo.prod.mydomain.com  | grep "Hostname"; done
```
<img width="810" height="517" alt="blue-3" src="https://github.com/user-attachments/assets/9478f72f-3cc0-4424-bb1c-f1627b59ca0e" />


Now, you can see traffic is routing to version-1 when ingress is pointing to version-1 and if you change the ingress and point to version-2 traffic will route to version. You can the same in above image.
