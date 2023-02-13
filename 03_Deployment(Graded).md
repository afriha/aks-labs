# Deployment configuration
The code snippet creates a simple nginx deployement:

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.14.2
        name: nginx
        ports: 
        - containerPort: 80
```

Create the deployment from this manifest.
Do not forget to create the namespace before!


```bash

yumemaru@Azure:~/LabAKS$ kubectl apply -f deployement.yaml
yumemaru@Azure:~/LabAKS$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx              0/3     0            0           1s
yumemaru@Azure:~/LabAKS$ 

```

Update your deployment with a new nginx image: 

```bash

yumemaru@Azure:~/LabAKS$ kubectl set image deployment/nginx nginx=nginx:1.16.1
yumemaru@Azure:~/LabAKS$ kubectl get deployments nginx
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx              2/3     0            0           30s
yumemaru@Azure:~/LabAKS$ 

```

Scale your deployment to 5 pods: 

```bash

yumemaru@Azure:~/LabAKS$ kubectl scale deployment/nginx --replicas=5
yumemaru@Azure:~/LabAKS$ kubectl get deployments nginx
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx              3/5     0            0           90s
yumemaru@Azure:~/LabAKS$ 

```

