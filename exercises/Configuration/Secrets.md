

## Secrets

kubernetes.io > Documentation > Concepts > Configuration > [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

kubernetes.io > Documentation > Tasks > Inject Data Into Applications > [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

### Create a secret called mysecret with the values password=mypass

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```

</p>
</details>
<br/>

### Create a secret called mysecret2 that gets key/value from a file

Create a file called username with the value admin:

```bash
echo -n admin > username
```

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret2 --from-file=username
```

</p>
</details>
<br/>

### Get the value of mysecret2

<details><summary>show</summary>
<p>

```bash
kubectl get secret mysecret2 -o yaml
echo -n YWRtaW4= | base64 -d # on MAC it is -D, which decodes the value and shows 'admin'
```

Alternative using `--jsonpath`:

```bash
kubectl get secret mysecret2 -o jsonpath='{.data.username}' | base64 -d  # on MAC it is -D
```

Alternative using `--template`:

```bash
kubectl get secret mysecret2 --template '{{.data.username}}' | base64 -d  # on MAC it is -D
```

</p>
</details>
<br/>

### Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # our volume mounts
    - name: foo # name on pod.spec.volumes
      mountPath: /etc/foo #our mount path
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx /bin/bash
ls /etc/foo  # shows username
cat /etc/foo/username # shows admin
```

</p>
</details>
<br/>

### Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
<br/>