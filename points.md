## k8s Architecture
> linux commands
```
find ~ -name toBeFound.yaml
cp ~/folder1/toBeFound.yaml found1.yaml
```
> confige commands
```
source <(kubectl completion bash) ; \
echo "source <(kubectl completion bash)" >> ~/.bashrc

alias k=kubectl; complete -F __start_kubectl k ; \
echo "alias k=kubectl; complete -F __start_kubectl k" >> ~/.bashrc
```
> verify...
```
k get deploy,po,svc,ep,pv,pvc
```

### Deploy a cluster
> Deploy a master node using kubeadm init
```
sudo apt-get install -y kubeadm=1.14.1-00
sudo kubeadm init --kubernetes-version 1.14.1 --pod-network-cidr 192.168.0.0/16
```
> If kubeadm exists in master node...
```
kubeadm token create --print-join-command
--- below command is returned
kubeadm join 10.0.0.181:6443 --token 06d8ul.i2bncyx2ffk04imi --discovery-token-ca-cert-hash \
sha256:baa500ae2cd46e952ea2d481d18a5d67d173f0890f771520f0ccee4b31ca77f7

```
> In minion node
```
sudo kubeadm join 10.0.0.181:6443 --token 06d8ul.i2bncyx2ffk04imi --discovery-token-ca-cert-hash \
sha256:baa500ae2cd46e952ea2d481d18a5d67d173f0890f771520f0ccee4b31ca77f7

```
> back to master node
```
k describe nodes | grep -i taint
k taint node --all node-role.kubernetes.io/master-
```

### Create a pod (2 containers) exposed by the port, and a service to expose the pod in the cluster
```
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
```
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
```
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
```
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
```
kubectl create deployment try1 --image=10.110.186.162:5000/simpleapp:latest
kubectl scale deployment try1 --replicas=6
k get deployment try1 -o yaml > simpleapp.yaml
```
> verify container running on minion node
> scheduler will deploy equal number of pods on both nodes
```
sudo docker ps |grep simple
```
### readinessProbes and livenessProbes
> create the path to make container ready
> verify the pods, Ready  true, State running
```
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

## Deployment

## Security

## Exposing Applications

## Troubleshooting