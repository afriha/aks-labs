# Resource Limits
## Resource Quota

Create a namespace so that the resources you create in this exercise are
isolated from the rest of your cluster.

```bash
kubectl create namespace limits
```

### Create a ResourceQuota

Here is a manifest for an example ResourceQuota:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    persistentvolumeclaims: "5"
    requests.storage: "5Gi"
```

Create the ResourceQuota:

```bash
kubectl apply -f quota-mem-cpu.yaml --namespace=limits
```

View detailed information about the ResourceQuota:

```bash
kubectl get resourcequota mem-cpu-demo --namespace=limits --output=yaml
```

The ResourceQuota places these requirements on the limits namespace:

* For every Pod in the namespace, each container must have a memory request, memory limit, cpu request, and cpu limit.
* The memory request total for all Pods in that namespace must not exceed 1 GiB.
* The memory limit total for all Pods in that namespace must not exceed 2 GiB.
* The CPU request total for all Pods in that namespace must not exceed 1 cpu.
* The CPU limit total for all Pods in that namespace must not exceed 2 cpu.

### Create a Pod

Here is a manifest for an example Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "800m"
      requests:
        memory: "600Mi"
        cpu: "400m"
```
Create the Pod:

```bash
kubectl apply -f pod.yaml --namespace=limits
```

Verify that the Pod is running and that its (only) container is healthy:

```bash
kubectl get pod quota-mem-cpu-demo --namespace=limits
```

Once again, view detailed information about the ResourceQuota:

```bash
kubectl get resourcequota mem-cpu-demo --namespace=limits --output=yaml
```

The output shows the quota along with how much of the quota has been used.
You can see that the memory and CPU requests and limits for your Pod do not
exceed the quota.

```
status:
  hard:
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.cpu: "1"
    requests.memory: 1Gi
  used:
    limits.cpu: 800m
    limits.memory: 800Mi
    requests.cpu: 400m
    requests.memory: 600Mi
```
### Attempt to create a second Pod

Here is a manifest for a second Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo-2
spec:
  containers:
  - name: quota-mem-cpu-demo-2-ctr
    image: redis
    resources:
      limits:
        memory: "1Gi"
        cpu: "800m"
      requests:
        memory: "700Mi"
        cpu: "400m"
```

In the manifest, you can see that the Pod has a memory request of 700 MiB.
Notice that the sum of the used memory request and this new memory
request exceeds the memory request quota: 600 MiB + 700 MiB > 1 GiB.

Attempt to create the Pod:

```bash
kubectl apply -f pod-2.yaml --namespace=limits
```

The second Pod does not get created. The output shows that creating the second Pod
would cause the memory request total to exceed the memory request quota.

```
Error from server (Forbidden): error when creating "examples/admin/resource/quota-mem-cpu-pod-2.yaml":
pods "quota-mem-cpu-demo-2" is forbidden: exceeded quota: mem-cpu-demo,
requested: requests.memory=700Mi,used: requests.memory=600Mi, limited: requests.memory=1Gi
```
## Limit Range
### Create a LimitRange and a Pod 

Create a LimitRange and a Pod:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-min-max-demo-lr
spec:
  limits:
  - default:
      cpu: 800m
      memory: 500Mi
    defaultRequest:
      cpu: 500m
      memory: 500Mi
    max:
      cpu: "1"
      memory: 2Gi
    min:
      cpu: "200m"
      memory: 256Mi
    type: Container
```
Create the LimitRange:
```bash
kubectl apply -f limitrange.yaml --namespace=limits
```

```yaml
Create the Pod:
apiVersion: v1
kind: Pod
metadata:
  name: default-mem-demo
spec:
  containers:
  - name: default-mem-demo-ctr
    image: nginx
```
```bash
kubectl apply -f podlimits.yaml --namespace=limits
```