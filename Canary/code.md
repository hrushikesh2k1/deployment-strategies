To use canary deployment we need to have ingress controller as well as ingress resource created as we discussed we need to limit the
traffic to new version.

I am using minikube cluster for showing this.

## Enable the ingress, it will enable nginx ingress control by default
```
minikube enable ingress
```

## Create the deployment and service for version-1 (deployment.yaml)
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

## Creating the deployment and service for version-2 (canary.yaml)
```
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: version-2
  labels:
    app: version-2
spec:
  replicas: 1
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

## Creating the ingress resource for version-1 (ingress-v1.yaml)
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

## Creating the ingress resource for verison-2 (ingress-v2.yaml)
```
echo "
---
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: version-2
  annotations:
    nginx.ingress.kubernetes.io/canary: \"true\"
    nginx.ingress.kubernetes.io/canary-weight: \"10\"
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
            name: version-2
            port:
              number: 80
```
The nginx.ingress.kubernetes.io/canary: "true" annotation is required and defines this as a canary annotation.

The nginx.ingress.kubernetes.io/canary-weight: "10" annotation dictates the weight of the routing, 
in this case there is a "10%" chance a request will hit the canary deployment over the main deployment

