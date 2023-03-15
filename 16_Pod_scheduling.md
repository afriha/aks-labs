# Pod Scheduling
## NodeSelector

Create a namespace so that the resources you create in this exercise are
isolated from the rest of your cluster.

```bash
kubectl create namespace scheduling
```

List the nodes in your cluster, along with their labels:


Choose one of your nodes, and add a label to it:
kubectl label nodes <your-node-name> env=prod
```bash
kubectl get nodes --show-labels
```

### Create a pod that gets scheduled to your chosen node 


Here is a manifest for the pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    env: prod
```

Create the pod:

```bash
kubectl apply -f pod-nodeselector.yaml --namespace=scheduling
```

Verify that the pod is running on your chosen node:

```bash
kubectl get pods -n scheduling -o wide
```

### Create a pod that gets scheduled to specific node
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: foo-node # schedule pod to specific node
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```
Use the configuration file to create a pod that will get scheduled on your node only.

Verify that the pod is running on your chosen node:

```bash
kubectl get pods -n scheduling -o wide
```
## NodeAffinity

Here is a manifest for an example Pod with nodeAffinity:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: In
            values:
            - prod
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0
```
Create the Pod:

```bash
kubectl apply -f pod-nodeAffinity.yaml --namespace=scheduling
```
Verify that the pod is running on your chosen node:

```bash
kubectl get pods -n scheduling -o wide
```
## PodAffinity

Similar to node affinity are two types of Pod affinity and anti-affinity as follows:
* requiredDuringSchedulingIgnoredDuringExecution
* preferredDuringSchedulingIgnoredDuringExecution

Here is a manifest for an example Deployment with podAffinity:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis

```
Create the redis deployment:

```bash
kubectl apply -f redis-podAntiAffinity.yaml --namespace=scheduling
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx
```

Create the redis deployment:

```bash
kubectl apply -f nginx-podAffinity.yaml --namespace=scheduling
```
Creating the two preceding Deployments results in the following cluster layout,
where each web server is co-located with a cache, on three separate nodes.

|    node-1     |    node-2     |    node-3     |
| :-----------: | :-----------: | :-----------: |
| *webserver-1* | *webserver-2* | *webserver-3* |
|   *cache-1*   |   *cache-2*   |   *cache-3*   |


The overall effect is that each cache instance is likely to be accessed by a single client, that is running on the same node. This approach aims to minimize both skew (imbalanced load) and latency