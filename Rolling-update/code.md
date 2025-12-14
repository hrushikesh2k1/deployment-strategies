## creating the deployment.yaml file with 2 replicas with nginx image
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mypod
  labels:
    app: mypod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mypod
  template:
    metadata:
      labels:
        app: mypod
    spec:
      containers:
      - name: nginx
        image: nginx
```

## depploying the pods
```
kubectl apply -f deployment.yaml
```

## see the pods
```
kubectl get pods
```

## Update the deployment.yaml with apache image
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mypod
  labels:
    app: mypod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mypod
  template:
    metadata:
      labels:
        app: mypod
    spec:
      containers:
      - name: apache
        image: httpd:latest
```

## watching the pods live
```
kubectl get pods -w
```
