![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/configuration&empty)
# Configuration (18%)

## ConfigMaps

### Create a configmap named config with values foo=lala,foo2=lolo

<details><summary>show</summary>
<p>

```bash
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```

</p>
</details>

### Display its values

<details><summary>show</summary>
<p>

```bash
kubectl get cm config -o yaml --export
# or
kubectl describe cm config
```

</p>
</details>

### Create a configmap from the configmap-files directory with the following files. Called it config-files.

Create the files with

```bash
mkdir -p configmap-files
echo -e "License File" > configmap-files/LICENSE
echo -e "enemies=aliens\nlives=3\nenemies.color=green" > configmap-files/game.env
```

<details><summary>show</summary>
<p>

```bash
kubectl create configmap config-files --from-file=configmap-files/
```

</p>
</details>

### Mount a volume in a nginx pod on /mnt/space-game. Populate it with the data stored in the previous configmap. Check the files inside the pod.

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx -o yaml --dry-run > pod.yaml
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
    volumeMounts: # your volume mounts are listed here
    - name: game-volume # the name that you specified in pod.spec.volumes.name
      mountPath: /mnt/space-game # the path inside your container
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes: # add a volumes list
  - name: game-volume # just a name, you'll reference this in the pods
    configMap:
      name: config-files # name of your configmap
status: {}
```
```bash
kubectl create -f pod.yaml
kubectl exec nginx -- sh -c 'ls /mnt/space-game'
```

<details><summary>show</summary>
<p>

```bash
kubectl create configmap config-files --from-file=configmap-files/
```

</p>
</details>

### Modify the previous configmap (e.g change the LICENSE file). Check that the file has been updated automatically in the nginx pod.

<details><summary>show</summary>
<p>

```bash
kubectl exec nginx -- sh -c 'cat /mnt/space-game/LICENSE' # Check initial value
kubectl edit configmaps config-files
kubectl exec nginx -- sh -c 'cat /mnt/space-game/LICENSE' # Check updated value
```

</p>
</details>

### Create and display a configmap from a file

Create the file with

```bash
echo -e "foo3=lili\nfoo4=lele" > config.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap2 --from-file=config.txt
kubectl get cm configmap2 -o yaml --export
```

</p>
</details>


### Create and display a configmap called configmap-two-files from the following two files

Create the files with

```bash
echo -e "foo1=Hello\nfoo2=World" > welcome.txt
echo -e "foo3=Bye\nfoo4=World" > farewell.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap-two-files --from-file=welcome.txt  --from-file=farewell.txt
kubectl get cm configmap-two-files -o yaml --export
```

</p>
</details>

### Create and display a configmap from a .env file

Create the file with the command

```bash
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap3 --from-env-file=config.env
kubectl get cm configmap3 -o yaml --export
```

</p>
</details>

### Create and display a configmap from a file, giving the key 'special'

Create the file with

```bash
echo -e "var3=val3\nvar4=val4" > config4.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap4 --from-file=special=config4.txt
kubectl describe cm configmap4
kubectl get cm configmap4 -o yaml --export
```

</p>
</details>

### Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

<details><summary>show</summary>
<p>

```bash
kubectl create cm options --from-literal=var5=val5
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml > pod.yaml
vi pod.yaml
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
    env:
    - name: option # name of the env variable
      valueFrom:
        configMapKeyRef:
          name: options # name of config map
          key: var5 # name of the entity in config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep option # will show 'option=val5'
```

</p>
</details>

### Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl create configmap anotherone --from-literal=var6=val6 --from-literal=var7=val7
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml > pod.yaml
vi pod.yaml
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
    envFrom: # different than previous one, that was 'env'
    - configMapRef: # different from the previous one, was 'configMapKeyRef'
        name: anotherone # the name of the config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env
```

</p>
</details>

### Create a configMap 'bash-script' using the script.sh file. Load this file with permission 777 (octal) inside an nginx pod on path '/etc/example'. Create the pod and run the script.
Note that the JSON spec doesn’t support octal notation, therefore, translate the octal number to decimal.
```bash
echo "echo 'Executing a file inside of a pod :)'" > script.sh
```

<details><summary>show</summary>
<p>

```bash
kubectl create configmap bash-script --from-file=script.sh
kubectl run --generator=run-pod/v1 nginx --image=nginx -o yaml --dry-run -o yaml > pod.yaml
vi pod.yaml
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
  volumes: # add a volumes list
  - name: myvolume # just a name, you'll reference this in the pods
    configMap:
      name: bash-script # name of your configmap
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts: # your volume mounts are listed here
    - name: myvolume # the name that you specified in pod.spec.volumes.name
      mountPath: /etc/example # the path inside your container
      defaultMode: 510 # change permission
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec nginx -- sh -c '/etc/example/script.sh'
```

</p>
</details>

## SecurityContext

### Create the YAML for an nginx pod that runs with the UID 101. No need to create the pod

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx -o yaml --dry-run -o yaml> pod.yaml
vi pod.yaml
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
  securityContext: # insert this line
    runAsUser: 101 # UID for the user
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</p>
</details>


### Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added on its single container

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx -o yaml --dry-run -o yaml> pod.yaml
vi pod.yaml
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
  securityContext: # insert this line
    capabilities: # and this
      add: ["NET_ADMIN", "SYS_TIME"] # this as well
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</p>
</details>

### Create the YAML for an nginx pod that runs a container with UID 1000. No need to create the pod.

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx -o yaml --dry-run -o yaml > pod.yaml
vi pod.yaml
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
    name: nginx
    securityContext: # insert this line
      runAsUser: 1000 # UID for the user
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

</p>
</details>

## Requests and limits

### Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
```

</p>
</details>

## Secrets

### Create a secret called mysecret with the values password=mypass

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```

</p>
</details>

### Create a secret called mysecret2 that gets key/value from a file. Describe the secret.

Create a file called username with the value admin:

```bash
echo admin > username
```

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret2 --from-file=username
kubectl describe secret mysecret2
```

```bash
Name:         mysecret2
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
username:  6 bytes
```

</p>
</details>

### Get the value of mysecret2

<details><summary>show</summary>
<p>

```bash
kubectl get secret mysecret2 -o yaml --export
echo 'YWRtaW4K' | base64 -d # shows 'admin'
```

Alternative:

```bash
kubectl get secret mysecret2 -o jsonpath='{.data.username}{"\n"}' | base64 -d  # on MAC it is -D
```

</p>
</details>

### Create a nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml > pod.yaml
vi pod.yaml
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
  volumes: # specify the volumes
  - name: foo # this name will be used for reference inside the container
    secret: # we want a secret
      secretName: mysecret2 # name of the secret - this must already exist on pod creation
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts: # our volume mounts
    - name: foo # name on pod.spec.volumes
      mountPath: /etc/foo #our mount path
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx /bin/bash
ls /etc/foo  # shows username
cat /etc/foo/username # shows admin
```

or

```bash

kubectl exec nginx -- sh -c 'ls /etc/foo' # shows username
kubectl exec nginx -- sh -c 'cat /etc/foo/username' # shows admin
```

</p>
</details>

### Modify the previous deployment. Change the permission of the secret file to read-only.

<details><summary>show</summary>
<p>

```bash
kubectl delete -f pod.yaml
vi pod.yaml
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
  volumes:
  - name: foo
    secret: 
      secretName: mysecret2 
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts: 
    - name: foo 
      mountPath: /etc/foo 
      readOnly: true # Add this line
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx bash
# Try to edit the file
```

</p>
</details>

### Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml > pod.yaml
vi pod.yaml
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
    env: # our env variables
    - name: USERNAME # asked name
      valueFrom:
        secretKeyRef: # secret reference
          name: mysecret2 # our secret's name
          key: username # the key of the data in the secret
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep USERNAME | cut -d '=' -f 2 # will show 'admin'
```

</p>
</details>

### Create a secret called mysecret3 manually using YAML files. This secret saves two strings: username and password. Get the value of mysecret3


<details><summary>show</summary>
<p>

```bash
# For example:
echo -n 'pepe' | base64
echo -n 'pepito123' | base64

kubectl create secret generic mysecret3  --from-literal=username=xx --from-literal=password=xxx --dry-run -o yaml > secret.yaml
vim secret.yaml
```

```YAML
apiVersion: v1
data:
  password: eHh4 # Replace this value with cGVwZQ==
  username: eHg= # Replace this value with cGVwaXRvMTIz
kind: Secret
metadata:
  creationTimestamp: null
  name: mysecret3
```

```bash
kubectl create -f secret.yaml
kubectl get secrets mysecret3 -o yaml

echo 'cGVwZQ==' | base64 -d # shows 'pepe'
echo 'cGVwaXRvMTIz' | base64 -d # shows 'pepito123'
```

</p>
</details>

### Create a secret called mysecret4 with unencoded strings using the stringData map.
The stringData field is provided for convenience, and allows you to provide secret data as unencoded strings.

String data example: 
app_url=http://wwww.my-app.com
username=lola
password=lolita123


<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret4 --from-literal=app_url=x --from-literal=username=x --from-literal=password=x --dry-run -o yaml > secret.yaml
vim secret.yaml
```

Change the secret.yaml file from:

```YAML
apiVersion: v1
data:
  app_url: eA==
  password: eA==
  username: eA==
kind: Secret
metadata:
  creationTimestamp: null
  name: mysecret4
```

to:

```YAML
apiVersion: v1
stringData: # change the map from data to stringData 
  app_url: http://wwww.my-app.com # Add this line
  password: lola # Add this line
  username: lolita123 # Add this line
kind: Secret
metadata:
  creationTimestamp: null
  name: mysecret4
```

```bash
kubectl create -f secret.yaml
kubectl get secrets mysecret4 -o yaml

echo 'aHR0cDovL3d3d3cubXktYXBwLmNvbQ==' | base64 -d # shows 'http://wwww.my-app.com'
echo 'bG9sYQ==' | base64 -d # shows 'lola'
echo 'bG9saXRhMTIz' | base64 -d # shows 'lolita123'
```

</p>
</details>


### If a field is specified in both data and stringData maps, the value from stringData is used. Test it in a new secret.

String data examples:
data map --> alien
stringData map --> cowboy 

<details><summary>show</summary>
<p>

```bash
echo -n 'alien' | base64 # Enconde alien --> YWxpZW4=

# Create a secret file
vim secret.yaml
```


```YAML
apiVersion: v1
kind: Secret
metadata:
  name: mysecret5
type: Opaque
data:
  username: YWxpZW4=
stringData:
  username: cowboy
```


```bash
kubectl create -f secret.yaml
kubectl get secrets mysecret5 -o yaml

echo 'Y293Ym95' | base64 -d # shows 'cowboy'
```

</p>
</details>

## ServiceAccounts

### See all the service accounts of the cluster in all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get sa --all-namespaces
```

</p>
</details>

### Create a new serviceaccount called 'myuser'

<details><summary>show</summary>
<p>

```bash
kubectl create sa 'myuser' --dry-run -o yaml
```

Alternatively:

```bash
# let's get a template easily
kubectl get sa default -o yaml --export > sa.yaml
vim sa.yaml
```

```YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myuser
```

```bash
kubectl create -f sa.yaml
```

</p>
</details>

### Create a nginx pod that uses 'myuser' as a service account

<details><summary>show</summary>
<p>

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx --serviceaccount=myuser
```
Alternatively:

```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml > pod.yaml
vi pod.yaml
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
  serviceAccountName: myuser # we use pod.spec.serviceAccountName  
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx # will see that a new secret called myuser-token-***** has been mounted
```

</p>
</details>


### Create a new service account. Add imagePullsecrets to the already created service account. Create a nginx pod that uses the previous service. Describe the pod in order to see the imagePullPolicy.

<details><summary>show</summary>
<p>

```bash
kubectl create serviceaccount docker-registry-sa

kubectl create secret docker-registry my-registry --docker-server=myregistr.com --docker-username=user --docker-password=user --docker-email=user@mydomain.com


```

```YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2019-03-28T16:15:41Z"
  name: docker-registry-sa
  resourceVersion: "49644948" # Remove this line
  selfLink: /api/v1/namespaces/default/serviceaccounts/docker-registry-sa
  uid: bb8364d4-5174-11e9-a001-fa163e84ec56
secrets:
- name: docker-registry-sa-token-z9w2w
imagePullSecrets: # Add this line
- name: my-registry # Add this line
```

```bash
kubectl get pods -o yaml --export nginx | grep "serviceAccount:"
```

</p>
</details>
