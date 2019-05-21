## k8s Architecture
> linux commands
```console
find ~ -name toBeFound.yaml
cp ~/folder1/toBeFound.yaml found1.yaml
```
> confige commands
```console
source <(kubectl completion bash) ; \
echo "source <(kubectl completion bash)" >> ~/.bashrc

alias k=kubectl; complete -F __start_kubectl k ; \
echo "alias k=kubectl; complete -F __start_kubectl k" >> ~/.bashrc
```
> verify...
```console
k get deploy,po,svc,ep,pv,pvc
```

### 1. Deploy a cluster
> Deploy a master node using kubeadm init
```console
sudo apt-get install -y kubeadm=1.14.1-00
sudo kubeadm init --kubernetes-version 1.14.1 --pod-network-cidr 192.168.0.0/16
```
> If kubeadm exists in master node...
```console
kubeadm token create --print-join-command
--- below command is returned
kubeadm join 10.0.0.181:6443 --token 06d8ul.i2bncyx2ffk04imi --discovery-token-ca-cert-hash \
sha256:baa500ae2cd46e952ea2d481d18a5d67d173f0890f771520f0ccee4b31ca77f7

```
> Deploy a minion node
```console
sudo kubeadm join 10.0.0.181:6443 --token 06d8ul.i2bncyx2ffk04imi --discovery-token-ca-cert-hash \
sha256:baa500ae2cd46e952ea2d481d18a5d67d173f0890f771520f0ccee4b31ca77f7

```
> back to master node
```console
k describe nodes | grep -i taint
k taint node --all node-role.kubernetes.io/master-
```

### 2. Create a pod with 2 containers, and a service
> exposed by the port, and a service to expose the pod in the cluster
```console
curl http://rawfileofsomepodfromdoc.yaml >> singleContainerPod.yaml
k create -f ingleContainerPod.yaml; k get pod; k delete pod ingleContainerPod
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: basicpod
  labels:
    type: webserver
spec:
  containers:
  - name: webcont
    image: nginx
    ports:
    - containerPort: 80 #port 80 is defined in the container
  - name: fdlogger
    image: fluent/fluentd
---
apiVersion: v1
kind: Service
metadata:
  name: basicSvc
spec:
  selector:
    type: my-webserver
  ports:
  - port: 80
    protocol: TCP
```
### 3. Create a simple deployment in `mynamespace`
```console
k create deployment firstdeploy --image=nginx -n mynamespace
kubectl scale deployment try1 --replicas=6
```
## Build
### 4. deploy a new application (build a docker image)
> check python
> make script executable
> Dockerfile
> build container
> run container
> view the content log of the container
```console
which python
chmod +x simple.py
---
./simple.py
FROM python:2
ADD simple.py
CMD ["python", "./simple.py"]
---
sudo docker build -t simpleapp .
sudo docker images
sudo docker run simpleapp
---
sudo find / -name date.out
sudo tail /var/lib/docker/aufs/diff/..../date.out
```
### 5. configure a local docker repo
> install docker-compose
> create a docker compose yaml file
> build container for docker repo
> Translate a Docker Compose File to Kubernetes Resources
> create 2 PV for repo, use the hostPath strorage class for the volumes
> convert docker-compose to localregistry
> test registry using clusterIP:5000/v2/ of service/registry
> edit docker daemon.json
> test on pulling ubuntu image
> to register for an existing docker image
> config the minion node to use the registry
```console
cd ; sudo -i
apt-get install -y docker-compose apache2-utils
mkdir -p /localdocker/data; cd /localdocker/ ; vim docker-compose.yaml
docker-compose up
curl http://127.0.0.1:5000/v2/
curl -L <url for kompose linux64> -o kompose
chmod +x kompose
mv ./kompose /usr/local/bin/kompose
sudo kompose convert -f docker-compose.yaml -o localregistry.yaml
kubectl create -f localregistry.yaml
curl http://10.110.186.162:5000/v2/
---
sudo vim /etc/docker/daemon.json
{ "insecure-registries":["10.110.186.162:5000"] }
sudo systemctl restart docker.service
---
sudo docker pull ubuntu
sudo docker tag ubuntu:latest 10.110.186.162:5000/tagtest
sudo docker push 10.110.186.162:5000/tagtest
sudo docker image remove ubuntu:latest; sudo docker image remove 10.110.186.162:5000/tagtest
sudo docker pull 10.110.186.162:5000/tagtest
---
sudo docker tag simpleapp 10.110.186.162:5000/simpleapp
sudo docker push 10.110.186.162:5000/simpleapp
sudo docker pull 10.110.186.162:5000/simpleapp
```
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/data" # decided by docker-compose.yaml --> /localdocker/data:/data

```
#### create a deployment from image of local registry on master node
> store deployment in yaml file for futher editing
```console
kubectl create deployment try1 --image=10.110.186.162:5000/simpleapp:latest
kubectl scale deployment try1 --replicas=6
k get deployment try1 -o yaml > simpleapp.yaml
```
> verify container running on minion node
> scheduler will deploy equal number of pods on both nodes
```console
sudo docker ps |grep simple
```
### 6. Configure probes -- readinessProbes and livenessProbes
> create the path to make container ready
> verify the pods, Ready  true, State running
```console
for name in <pod1> <pod2>
do
kubectl exec $name touch /tmp/healthy
done
---
k describe po <pod1> | grep -E 'State|Ready'
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webpod
  labels:
    app: webpod
spec:
  containers:
  - name: webcont
    image: nginx
    ports:
    - containerPort: 80
    readinessProbe:
      periodSeconds: 5
      exec:
        command:
        - cat
        - /tmp/healthy
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 20
```
## Design
### 7. Evaluate network plugins
> verify the cni Calico is configured for the cluster
```console
ps -ef | grep cni
sudo find / -name install-cni.sh

```
### 8. Designing Applications With Duration: Create a Job and a CronJob
> Jobs run the application for a particular number of times
> restartPolicy: Never --> sleeps for 3 seconds, then stop
> Parallelism: 2 --> run 2 at a time
> Completions: 5 --> run 5 times in total
> activeDeadlineSeconds: 15 --> after the 15 second, the job stops
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  completions: 5
  parallelism: 2
  activeDeadlineSeconds: 15
  template:
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["/bin/sleep", "3"]
      restartPolicy: Never
  backoffLimit: 4
```
> CronJobs create a watch loop that create jobs when time is up
> schedule: "*/1 * * * *" --> every 2 minutes
> activeDeadlineSeconds: 10 --> the job continues within 10 seconds
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          activeDeadlineSeconds: 10
          containers:
          - name: busybox
            image: busybox
            command: ["/bin/sleep", "3"]
          restartPolicy: Never
```
## Deployment configuration
### 9. Configure the Deployment using configMap
> configMap with path
```console
mkdir primary;
echo blue > primary/blue;
echo "known as sky blue" >> primary/blue;
echo pink > favorite;
k create configmap colors \
--from-literal=text=black \
--from-file=./favorite \
--from-file=./primary/
```
> configMap is used below
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: test-container
      image: nginx
      env:
        - name: ilike
          valueFrom:
            configMapKeyRef:
              name: colors
              key: favorite  #<-- just the key
      envFrom:               #<-- whole configmap
      - configMapRef:
          name: colors
  restartPolicy: Always

```
> configMap without path
> volume containes the configMap
> volumMount mount the volume with a path
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: test-container
      image: nginx
      readinessProbe:
      periodSeconds: 5
      exec:
        command:
        - cat
        - /etc/config # <-- mountPath
      volumeMounts:
      - name: config-volume # <-- volume name
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: fast-car
  restartPolicy: Always
```
### 10. Configure the Deployment: attaching storage with PV with NFS and PVC
> configure a NFS server with pv and pvc
```console
find ~ -name CreateNFS.sh
cp <path to nfs.sh> ~
bash ~/CreateNFS.sh
```
> output mount path: /opt/sfw *
> on minion node, install nfs and mount the nfs
```console
sudo apt-get -y install nfs-common nfs-kernel-server
showmount -e master
sudo mount master:/opt/sfw /mnt
ls -l /mnt
```
> output mount path: /opt/sfw *
> data from host nfs is stored in /mnt
> on master node
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pvvol-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: "/opt/sfw"
    server: master
    readOnly: false
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-one
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
---
kind: Pod
apiVersion: v1
metadata:
  name: test-pv-pod
spec:
  volumes:
    - name: nfs-vol
      persistentVolumeClaim:
       claimName: pvc-one
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/mnt"
          name: nfs-vol
```
> verify mounts by describe pod
```console
Mounts:
/mnt from nfs-vol (rw)
```
### 11. configure ambassador containers using pv, pvc
```console
sudo mkdir /tmp/weblog
```
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluentd.conf: |
    <source>
      type tail
      format none
      path /var/log/1.log
      pos_file /var/log/1.log.pos
      tag count.format1
    </source>

    <source>
      type tail
      format none
      path /var/log/2.log
      pos_file /var/log/2.log.pos
      tag count.format2
    </source>

    <match **>
      type google_cloud
    </match>
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: weblog-pv-volume
spec:
  storageClassName: manual
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/weblog"
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: weblog-pvc-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
kind: Pod
apiVersion: v1
metadata:
  name: test-pv-pod
spec:
  volumes:
    - name: weblog-pv-storage
      persistentVolumeClaim:
       claimName: weblog-pvc-claim
    - name: log-config
      configMap:
        name: fluentd-config
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/var/log/nginx/"
          name: weblog-pv-storage
    - name: fdlogger
      image: fluent/fluentd
      env:
      - name: FLUENTD_ARGS
        value: -c /etc/fluentd-config/fluentd.conf
      volumeMounts:
        - mountPath: "/var/log"
          name: weblog-pv-storage
        - mountPath: "/etc/fluentd-config"
          name: log-config
```
> verify the log is saved in the mountpath
```console
k exec -c nginx -it test-pv-pod -- bash
ls -l /var/log/nginx/access.log
curl <svc ip http://192.168.213.181>
tailf /var/log/nginx/access.log
k logs test-pv-pod fdlogger #contains output
k logs test-pv-pod webcont
```
### 12. Rolling Updates and Rollbacks
> build the same image again
```console
sudo docker build -t simpleapp .
sudo docker tag simpleapp 10.105.119.236:5000/simpleapp:v2
sudo docker push 10.105.119.236:5000/simpleapp:v2
kubectl edit deployment try1
containers:
- image: 10.105.119.236:5000/simpleapp:v2 #<-- Edit tag
```
> `v2` tag replaced `latest`
> in minion node
```console
sudo docker pull 10.105.119.236:5000/simpleapp
sudo docker pull 10.105.119.236:5000/simpleapp:v2
```
> view the events and change of deployment
```console
kubectl get events
kubectl describe pod try1-895fccfb-ttqdn |grep -i Image
kubectl rollout history deployment try1 #view the history update
kubectl rollout history deployment try1 --revision=1 > one.out
kubectl rollout history deployment try1 --revision=2 > two.out
diff one.out two.out
kubectl rollout undo --dry-run=true deployment/try1
kubectl rollout undo deployment try1 --to-revision=1
```
## Security
### 13. Set SecurityContext for a Pod and Container
```console
kubectl exec -it secondapp -- sh
/$ ps aux #Check the user ID of the shell and other processes
/$ grep Cap /proc/1/status #check the capabilities of the kernel
/$ exit

capsh --decode=00000000a80425fb #decode the output
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-2
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: busybox
    image: busybox
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities:
        add: [ "NET_ADMIN" , "SYS_TIME" ]
```
### 14. Create and consume Secrets
```console
echo LFTr@1n | base64 #generating an encoded password.
```
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
data:
  password: TEZUckAxbgo=
---
``yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-2
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: busybox
    image: busybox
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities:
        add: [ "NET_ADMIN" , "SYS_TIME" ]
    volumeMounts:
    - name: mysql
      mountPath: /mysqlpassword
  volumes:
  - name: mysql
    secret:
      secretName: test-secret

```
```console
kubectl exec -ti secondapp -- /bin/sh
/ $ cat /mysqlpassword/password
LFTr@1n
/ $ cd /mysqlpassword/
/mysqlpassword $ ls -al
```
> /mysqlpassword/password is a symbolic link to ../data, which is also a symbolic link
### 14.1 using ServiceAccounts to assign cluster roles, or the ability to use particular HTTP verbs
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: secret-access-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-access-cr
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: secret-rb
subjects:
- kind: ServiceAccount
  name: secret-access-sa
roleRef:
  kind: ClusterRole #this must be Role or ClusterRole
  name: secret-access-cr # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
---
# in pod
spec:
  serviceAccountName: secret-access-sa
  securityContext:
    runAsUser: 1000

```
> verify secret
```console
kubectl describe pod secondapp |grep -i secret
```
### 15. Implement a NetworkPolicy
> Calico could, Flannel could not
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.0.0/16
  - Egress
```
> pod and svc to test on
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: podtestsecurity
  labels:
    app: podtestsecurity
spec:
  #securityContext:
  #  runAsUser: 1000
  containers:
  - name: nginx
    image: nginx
  - name: busy
    image: busybox
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities:
        add: [ "NET_ADMIN" , "SYS_TIME" ]
    volumeMounts:
    - name: mysql
      mountPath: /mysqlpassword
  volumes:
  - name: mysql
    secret:
      secretName: test-secret
```
```console
k expose pod podtestsecurity --type=NodePort --port=80
k edit svc podtestsecurity
```
> verify ingress and egress
```console
k exec -it -c busy podtestsecurity sh
nc -vz 127.0.0.1 80  #ingress
nc -vz www.linux.com 80 #egress
ip a #get ip of eth0 of the container, so we could white list this ip
# inet 192.168.55.91/32 scope global eth0
ping -c5 192.168.55.91
```
## Exposing Applications
### 16. Expose a Service
> 1. modify svc as NodePort type
> 2. modify svc as LoadBalancer -> NodePort and request an external loadbalancer for the external ip
### Ingress Controller
> for a large number of svc to expose pods
> ingress RBAC configures CR, SA, and CRB
> ingress controller configures SA, daemonset, svc
> ingress rule
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: www.example.com
    http:
      paths:
      - backend:
          serviceName: podtestsecurity
          servicePort: 80
        path: /
  - host: thirdpage.org
    http:
      paths:
      - backend:
          serviceName: thirdpage
          servicePort: 80
        path: /
```
>
```console
ip a
ens4: 10.128.0.7
curl -H "Host: www.example.com" http://10.128.0.7/
curl -H "Host: www.example.com" http://192.168.1.208/
---
k run thirdpage --generator=run-pod/v1 --image=nginx --port=80 -l example=third
k expose thirdpage --type=NodePort
```
