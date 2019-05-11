### Setup
```console
source <(kubectl completion bash) ; echo "source <(kubectl completion bash)" >> ~/.bashrc
echo "alias k=kubectl; complete -F __start_kubectl k" >> ~/.bashrc
```
### Node
```console
kubectl describe nodes | grep -i Taint
kubectl taint nodes --all node-role.kubernetes.io/master-
```
### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: basicpod
spec:
  containers:
  - name: webcont
    image: nginx
    ports:
    - containerPort: 80
  - name: fdlogger
    image: fluent/fluentd
```
```console
kubectl get pod -n kube-system
kubectl get pod --all-namespaces
```
### Service
```yaml
apiVersion: v1
kind: Service
metadata:
    name: basicservice
spec:
  selector:
    type: webserver
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
```
### Deployment
```console
kubectl create deployment firstpod --image=nginx
kubectl create deployment try1 --image=10.110.186.162:5000/simpleapp:latest
kubectl scale deployment try1 --replicas=6
kubectl get deployment try1 -o yaml --export > simpleapp.yaml
sudo docker ps | grep simple

```
### readinessProbe and livenessProbe
```console
kubectl exec -it try1-9869bdb88-rtchc -- /bin/bash

for name in try1-9869bdb88-2wfnr try1-9869bdb88-6bkn
do
kubectl exec $name touch /tmp/healthy
done

kubectl describe pod try1-76cc5ffcc6-tx4dz | grep -E 'State|Ready'
```

```yaml
spec:
  containers:
  - image: 10.111.235.60:5000/simpleapp:latest
    imagePullPolicy: Always
    name: simpleapp
    readinessProbe:         #<--This line and next five
      exec:
      command:
      - cat
      - /tmp/healthy
    periodSeconds: 5
  resources: {}
```
### cni
#### Which of the plugins allow vxlans?
Canal, Flannel, Kopeio-networking, Weave Net
#### Which are layer 2 plugins?
Canal, Flannel, Kopeio-networking, Weave Net
#### Which are layer 3?
Project Calico, Romana, Kube Router
#### Which allow network policies?
Project Calico, Canal, Kube Router, Romana Weave Net
#### Which can encrypt all TCP and UDP traffic?
Project Calico, Kopeio, Weave Net
#### Which deployment method would allow the most flexibility, multiple applications per pod or one per Pod?
One per pod
#### Which deployment method allows for the most granular scalability?
One per pod
#### Which have the best inter-container performance? Multiple per pod.
#### How many IP addresses are assigned per pod?
One
#### What are some ways containers can communicate within the same pod?
IPC, loopback or shared filesystem access.
#### What are some reasons you should have multiple containers per pod?
Lean containers may not have functionality like logging. Able to maintain lean execution but add functionality as necessary, like Ambassadors and Sidecar containers.

### Job
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleepy
spec:
  completions: 5
  parallelism: 2
  activeDeadlineSeconds: 15
  template
    spec:
      containers:
      - name: resting
        image: busybox
        command: ["/bin/sleep"]
        args: ["3"]
      restartPolicy: Never
```
### CronJob
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: sleepy
spec:
  schedule: "*/2 * * * *"
  jobTemplate
    spec:
      template:
        spec:
          activeDeadlineSeconds: 10
          containers:
          - name: resting
            image: busybox
            command: ["/bin/sleep"]
            args: ["3"]
          restartPolicy: Never
```
### ConfigMap
```console
kubectl create configmap colors \
--from-literal=text=black \
--from-file=./favorite \
--from-file=./primary/

kubectl exec -it try1-d4fbf76fd-46pkb -- /bin/bash -c env

```
```yaml
spec:
  containers:
  - image: 10.105.119.236:5000/simpleapp:latest
    env : # Add from here
    - name: ilike
      valueFrom:
        configMapKeyRef:
          name: colors
          key: favorite # To here
```
```yaml
    envFrom : #<-- Add this and the following two lines
    - configMapRef:
        name: colors
     imagePullPolicy: Always
```
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fast-car
  namespace: default
data:
  car.make: Ford
  car.model: Mustang
  car.trim: Shelby
```
```yaml
spec:
  containers:
  - image: 0.105.119.236:5000/simpleapp:latest
    volumeMounts:
    - mountPath: /etc/cars
      name: car-vol
    env:
    - name: ilike
```
```yaml
volumes:
- name: car-vol
  configMap:
    defaultMode: 420
    name: fast-car
```
### PersistentVolume and PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvvol-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /opt/sfw
    server: master
    readOnly: false
-----------------------------
kind: PersistentVolume
apiVersion: v1
metadata:
  name: weblog-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/weblog"
```
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-one
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
```
```yaml
... in deploy
volumMounts:
- name: car-vol 
  mountPath: /etc/cars #<-- just like a path for mount storing resources from nfs
- name: nfs-vol
  mountPath: /opt

... at containers level
volumes:
- name: car-vol
  configMap:
    defaultMode: 420
    name: fast-car
- name: nfs-vol
  persistentVolumeClaim:
    claimName: pvc-one
dnsPolicy: ClusterFirst


```
