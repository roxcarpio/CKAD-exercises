![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/observability&empty)
# Observability (18%)

## Liveness and readiness probes

### Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod-1.yaml. Run it, check its probe status, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx -o yaml --dry-run > pod-1.yaml
vi pod-1.yaml


```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe: # our probe
      exec: # add this line
        command: # command definition
        - ls # ls command
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod-1.yaml
kubectl describe pod nginx | grep -i liveness # run this to see that liveness probe works
kubectl delete -f pod-1.yaml
```

</p>
</details>

### Modify the pod-1.yaml file so that liveness probe starts kicking in after 5 seconds whereas the period of probing would be 10 seconds. Run it, check the probe, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl explain pod.spec.containers.livenessProbe # get the exact names
vim pod-1.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe:
      initialDelaySeconds: 5 # add this line
      periodSeconds: 10 # add this line as well
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod-1.yaml
kubectl describe po nginx | grep -i liveness
kubectl delete -f pod-1.yaml
```

</p>
</details>

### Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx --port=80 -o yaml --dry-run > pod-2.yaml
vi pod-2.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    ports:
      - containerPort: 80
    readinessProbe: # declare the readiness probe
      httpGet: # add this line
        path: / #
        port: 80 #
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod-2.yaml
kubectl describe pod nginx | grep -i readiness # to see the pod readiness details
kubectl delete -f pod-2.yaml
```

</p>
</details>

### Create an nginx pod (that includes port 80) with an TCP readinessProbe on port 80. Save its YAML in pod-3.yaml, run it, check the readinessProbe, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx --port=80 -o yaml --dry-run > pod-3.yaml
vim pod-3.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 80
    readinessProbe: # declare the readiness probe
      tcpSocket: # add the following lines
        port: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod-3.yaml
kubectl describe pod nginx | grep -i readiness # to see the pod readiness details
kubectl delete -f pod-3.yaml
```

</p>
</details>


## Logging

### Create a busybox pod that runs 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'. Check its logs

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 busybox --image=busybox  -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
kubectl logs busybox -f # follow the logs
```

</p>
</details>

## Debugging

### Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 busybox --image=busybox -- /bin/sh -c 'ls /notexist'
# show that there's an error
kubectl logs busybox
kubectl describe po busybox
kubectl delete po busybox
```

</p>
</details>

### Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 busybox --image=busybox -- notexist
kubectl logs busybox # will bring nothing! container never started
kubectl describe po busybox # in the events section, you'll see the error
# also...
kubectl get events | grep -i error # you'll see the error here as well
kubectl delete po busybox --force --grace-period=0
```

</p>
</details>

### Get CPU/memory utilization for nodes (heapster must be running)

<details><summary>show</summary>
<p>

```bash
kubectl top nodes
```

</p>
</details>
