### Setup
```console
source <(kubectl completion bash) ; echo "source <(kubectl completion bash)" >> ~/.bashrc
echo "alias k=kubectl; complete -F __start_kubectl k" >> ~/.bashrc
```

#### About grep
```console
kubectl describe nodes | grep -i Taint
kubectl describe pod secondapp |grep -i secret
sudo docker ps | grep simple
kubectl describe pod try1-76cc5ffcc6-tx4dz | grep -E 'State|Ready'
grep Cap /proc/1/status
```
*State: Running*
*Ready: True*
*CapInh: 00000000a80425fb*

#### About kubectl exec
```console
kubectl exec -it try1-9869bdb88-rtchc -- /bin/bash

kubectl exec -it try1-d4fbf76fd-46pkb -- /bin/bash -c env

kubectl exec -it secondapp -- sh

for name in try1-9869bdb88-2wfnr try1-9869bdb88-6bkn
do
kubectl exec $name touch /tmp/healthy
done
```

#### Command lines without yaml
```console
kubectl expose pod secondapp --type=NodePort --port=80
kubectl create service nodeport secondapp --tcp=80
kubectl run thirdpage --generator=run-pod/v1 --image=nginx --port=80 -l example=third

kubectl create deployment firstpod --image=nginx
kubectl create deployment try1 --image=10.110.186.162:5000/simpleapp:latest
kubectl scale deployment try1 --replicas=6


kubectl create configmap colors \
--from-literal=text=black \
--from-file=./favorite \
--from-file=./primary/

kubectl replace -f allclosed.yaml

kubectl edit ingress ingress-test

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
```console
//it requires labels in pod 
kubectl expose pod secondapp --type=NodePort --port=80
kubectl create service nodeport secondapp --tcp=80

//in service , make sure the selector refers to the label for the pod
kubectl edit svc secondapp
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
  name: sleeconsole
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
  name: sleeconsole
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
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluentd.conf: |
    <source>
      @type tail
      format none
      path /var/log/nginx/access.log
      tag count.format1
    </source>

    <match *.**>
      @type forward

      <server>
        name localhost
        host 127.0.0.1
      </server>
    </match>
```
```yaml
spec:
  containers:
  - image: 10.105.119.236:5000/simpleapp:latest
    env : # <-- assign a value from ConfigMap by key
    - name: ilike
      valueFrom:
        configMapKeyRef:
          name: colors
          key: favorite # 
```
```yaml
    envFrom : #<-- assign a configMap by name
    - configMapRef:
        name: colors
```
```yaml
spec:
  containers:
  - image: 0.105.119.236:5000/simpleapp:latest
    volumeMounts:
    - mountPath: /etc/cars
      name: car-vol #<-- volume name
---------------------
volumes:
- name: car-vol
  configMap:
    defaultMode: 420
    name: fast-car #<-- configMap in a volume
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
  nfs:   #<-- nfs as a volume with the host path
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
  hostPath: #<-- local path of the volume
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

---------------------------
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: weblog-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

```yaml
... in deploy
volumMounts:
- name: car-vol #<-- volume name
  mountPath: /etc/cars #<-- a path for mount getting resources from nfs host path
- name: nfs-vol #<-- volume name
  mountPath: /opt #<-- a local path for the mount getting resource from PV path

... at containers level
volumes:
- name: car-vol
  configMap:
    defaultMode: 420
    name: fast-car #<-- configMap name
- name: nfs-vol
  persistentVolumeClaim:
    claimName: pvc-one #<-- PVClaim points to the PV
```
#### PV and PVC in Ambassador containers
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: basicpod
  labels:
    type: webserver
spec:
  volumes:
    - name: weblog-pv-storage
      persistentVolumeClaim:
        claimName: weblog-pv-claim
    - name: log-config
      configMap:
        name: fluentd-config
  containers:
  - name: webcont
    image: nginx
    ports:
    - containersPort: 80
    volumeMounts:
      - mountPath: "/var/log/nginx/" #<-- different mount path
        name: weblog-pv-storage      #<-- same mount name cus of same volume
  - name: fdlogger
    image: fluent/fluentd
    env:
    - name: FLUENTD_ARGS #<-- just a name for configmap
      value: -c /etc/fluentd-config/fluentd.conf #<-- the value to be logged
    volumeMounts:
      - mountPath: "/var/log"    #<-- different mount path
        name: weblog-pv-storage  #<-- same mount name
      - mountPath: "/etc/fluentd-config" #<-- assign a path for the mount
        name: log-config                  
```
```console
tailf /var/log/nginx/access.log
```
```console
kubectl rollout history deployment try1 --revision=1 > one.out
kubectl rollout history deployment try1 --revision=2 > two.out
diff one.out two.out
kubectl rollout undo --dry-run=true deployment/try1
kubectl rollout undo deployment try1 --to-revision=1
```
### Security
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secondapp
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: busy
    image: busybox
    command:
      - sleep
      - "3600"
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
    volumeMounts: #<-- mount PV of secret
    - name: mysql
      mountPath: /mysqlpassword
  volumes: #<-- PV of secret
  - name: mysql
    secret:
      secretName: lfsecret
```
```console
kubectl exec -it secondapp -- sh
/$ ps aux
/$ grep Cap /proc/1/status

/$ cat /mysqlpassword/password
/$ exit

capsh --decode=00000000a80425fb
echo LFTr@1n | base64
```
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: lfsecret
data:
  password: TEZUckAxbgo=
```
### ServiceAccount
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
 name: secret-access-sa
```

### ClusterRole
```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
 name: secret-access-cr
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
  - list
```

### RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
 name: secret-rb
subjects:
- kind: ServiceAccount
  name: secret-access-sa
roleRef:
 kind: ClusterRole
 name: secret-access-cr
 apiGroup: rbac.authorization.k8s.io
```

#### ServiceAccount and ClusterRole through Rolebinding, pass secrets to pod secondapp
```yaml
...
  name: secondapp
spec:
  serviceAccountName: secret-access-sa #<-- pass SA to the pod, so that the secret is assigned to the pod
...
```
```yaml
kubectl describe pod secondapp |grep -i secret
```

### NetworkPolicy
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock: #<-- white list the ip block
        cidr: 192.168.0.0/16
    ports:
    - port: 80
      protocol: TCP #<-- allow TCP and 80 only
  - Egress
```
#### test ingress and egress
```console
kubectl replace -f allclosed.yaml

//ip a shows the ip of the container
kubectl exec -it secondapp -- sh
/$ nc -vz 127.0.0.1 80
127.0.0.1 (127.0.0.1:80) open
/$ nc -vz www.linux.com 80
www.linux.com (151.101.185.5:80) open
/$ ip a
/$ exit

ping -c5 192.168.55.91
```

### Ingress Controller
It requires **ClusterRole**, **ServiceAccount**, **ClusterRoleBinding**, **Daemonset**, **service**
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-test
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: www.example.com
    http:
      paths:
      - backend:
          serviceName: secondapp #<-- svc name
          servicePort: 80
        path: /
  - host: thirdpage.org
    http:
      paths:
      - backend:
          serviceName: thirdpage #<-- svc name
          servicePort: 80
        path: /
```
```console
kubectl run thirdpage --generator=run-pod/v1 --image=nginx --port=80 -l example=third

kubectl edit ingress ingress-test

curl -H "Host: thirdpage.org" http://10.128.0.7/
```

