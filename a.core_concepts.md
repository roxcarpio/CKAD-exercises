![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/core_concepts&empty)
# Core Concepts (13%)

### Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace

<details><summary>show</summary>
<p>

```bash
kubectl create namespace mynamespace
kubectl run --generator=run-pod/v1 nginx --image=nginx -n mynamespace
```

</p>
</details>

### Create the pod that was just described using YAML

<details><summary>show</summary>
<p>

Easily generate YAML with:

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx -o yaml --dry-run > pod.yaml
```

```bash
cat pod.yaml
```

```yaml
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
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod.yaml -n mynamespace
```

Alternatively, you can run in one line

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx -o yaml --dry-run | kubectl create -n mynamespace -f -
```

</p>
</details>

### Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 busybox --image=busybox -it -- sh -c 'env' # -it will help in seeing the output
# or, just run it without -it
kubectl run --generator=run-pod/v1 busybox --image=busybox -- sh -c 'env'
# and then, check its logs
kubectl logs busybox
```

</p>
</details>

### Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
# create a  YAML template with this command
kubectl run --generator=run-pod/v1 busybox --image=busybox -o yaml --dry-run -- sh -c 'env' > envpod.yaml
# see it
cat envpod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - sh
    - -c
    - env
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
# apply it and then see the logs
kubectl apply -f envpod.yaml
kubectl logs busybox
```

Alternatively, you can run in one line

```bash
kubectl run --generator=run-pod/v1 busybox --image=busybox -o yaml --dry-run -- sh -c 'env' | kubectl create -f -
```

</p>
</details>

### Get the YAML for a new namespace called 'myns' without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create namespace myns -o yaml --dry-run
```

</p>
</details>

### Get the YAML for a new ResourceQuota called 'myrq' without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run -o yaml
```

</p>
</details>

### Get pods on all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get po --all-namespaces
```

</p>
</details>

### Create a pod with image nginx called nginx and allow traffic on port 80

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx --port=80
```

</p>
</details>

### Change pod's image to nginx:1.7.1. Observe that the pod will be killed and recreated as soon as the image gets pulled

<details><summary>show</summary>
<p>

```bash
# kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
kubectl set image pod/nginx nginx=nginx:1.7.1
kubectl describe po nginx # you will see an event 'Container will be killed and recreated'
kubectl get po nginx -w # watch it
```
*Note*: you can check pod's image by running

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

</p>
</details>

### Get the pod's ip, use a temp busybox image to wget its '/'

<details><summary>show</summary>
<p>

```bash
kubectl get po -o wide # get the IP, will be something like '10.1.1.131'
# create a temp busybox pod
kubectl run --generator=run-pod/v1 busy --image=busybox -it --rm
# run wget on specified IP:Port
wget -O- 10.1.1.131:80
exit
```

Alternatively you can also try a more advanced option:

```bash
# Get IP of the nginx pod
NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')
# create a temp busybox pod
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- sh
# run wget on specified IP:Port
wget -O- $NGINX_IP:80
exit
``` 

</p>
</details>

### Get this pod's YAML without cluster specific information

<details><summary>show</summary>
<p>

```bash
kubectl get po nginx -o yaml --export
```

</p>
</details>

### Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx
```

</p>
</details>

### Get pod logs

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx
```

</p>
</details>

### If pod crashed and restarted, get logs about the previous instance

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx -p
```

</p>
</details>

### Connect to the nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl exec -it nginx -- /bin/sh
```

</p>
</details>

### Create a busybox pod that echoes 'hello world' and then exits

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 busybox --image=busybox -- sh -c 'echo Hello World'
```

</p>
</details>

### Do the same, but have the pod deleted automatically when it's completed

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 busybox --it --rm --image=busybox -- sh -c 'echo Hello World'
kubectl get po # nowhere to be found :)
```

</p>
</details>

### Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx --env=var1=val1
# then
kubectl exec -it nginx -- env
# or
kubectl describe po nginx | grep val1
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
```

</p>
</details>
