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

### Deploy a cluster
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
> In minion node
```console
sudo kubeadm join 10.0.0.181:6443 --token 06d8ul.i2bncyx2ffk04imi --discovery-token-ca-cert-hash \
sha256:baa500ae2cd46e952ea2d481d18a5d67d173f0890f771520f0ccee4b31ca77f7

```
> back to master node
```console
k describe nodes | grep -i taint
k taint node --all node-role.kubernetes.io/master-
```

### Create a pod (2 containers) exposed by the port, and a service to expose the pod in the cluster
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
    ports: fluent/fluentd
    - containerPort: 80 #port 80 is defined in the container
  - name: fdlogger
    image: nginx
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
### Create a deployment in `mynamespace`
```console
k create deployment firstdeploy --image=nginx -n mynamespace
kubectl scale deployment try1 --replicas=6
```
## Build
### build a docker image
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
### configure a local docker repo
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
### create a deployment from image of local registry on master node
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
### readinessProbes and livenessProbes
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
    - containerPort: 8080
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
> verify the cni Calico is configured for the cluster
```console
ps -ef | grep cni
sudo find / -name install-cni.sh

```
### Designing Applications With Duration: Create a Job and a CronJob
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
## Deployment
### configMap
> configMap with path
```console
mkdir primary;
echo sky > primary/blue;
echo "known as blue sky" >> primary/blue;
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
### attaching storage with PV with NFS and PVC
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
### configure ambassador containers using pv, pvc
```console
sudo mkdir /tmp/weblog
```
```yaml
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
      volumeMounts:
        - mountPath: "/var/log"
          name: weblog-pv-storage
```
> verify the log is saved in the mountpath
```console
k exec -c nginx -it test-pv-pod -- bash
ls -l /var/log/nginx/access.log
curl <svc ip http://192.168.213.181>
tailf /var/log/nginx/access.log
```
## Security

## Exposing Applications

## Troubleshooting
