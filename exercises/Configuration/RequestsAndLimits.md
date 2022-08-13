## Requests and limits

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

### Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
```

Note: Use of `--requests` and `--limits` flags in the imperative `run` command is deprecated as of 1.21 K8s version and will be removed in the future. Instead, use `kubectl set resources` command in combination with `kubectl run --dry-run=client -o yaml ...` as shown below.

Alternative using `set resources` in combination with imperative `run` command:

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=100m,memory=256Mi --limits=cpu=200m,memory=512Mi --local -o yaml > nginx-pod.yml
```

```bash
kubectl create -f nginx-pod.yml
```

Alternative approach using pod.yaml: 
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
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
    resources:
      requests:
        memory: "256Mi"
        cpu: 100m
      limits:    
        memory: "512Mi"
        cpu: 200m
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
``` 

</p>
</details>
<br/>