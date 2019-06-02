### 1. Untaint master node
```console
k describe nodes | grep -i taint
k taint node --all node-role.kubernetes.io/master-
```

### 2. Create a pod with 2 containers, and a service
> exposed by the port, and a service to expose the pod in the cluster
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
kubectl create deployment firstdeploy --image=nginx -n mynamespace

kubectl scale deployment try1 --replicas=6
```
## Build
#### create a deployment from image of local registry on master node
> store deployment in yaml file for futher editing
```console
kubectl create deployment try1 --image=10.110.186.162:5000/simpleapp:latest

kubectl scale deployment try1 --replicas=6

k get deployment try1 -o yaml > simpleapp.yaml
```
### 6. Configure probes -- readinessProbes and livenessProbes
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
### 12. (Updating Resources) Rolling Updates and Rollbacks
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
