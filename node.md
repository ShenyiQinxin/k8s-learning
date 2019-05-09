### Setup
```console
source <(kubectl completion bash) ; echo "source <(kubectl completion bash)" >> ~/.bashrc

```
### Node
```console
kubectl describe nodes | grep -i Taint
kubectl taint nodes --all node-role.kubernetes.io/master-
kubectl create deployment firstpod --image=nginx
kubectl create deployment try1 --image=10.110.186.162:5000/simpleapp:latest
kubectl scale deployment try1 --replicas=6
kubectl get deployment try1 -o yaml --export > simpleapp.yaml
```
### Deployment
```console
```
### readinessProbe and livenessProbe
```console
kubectl exec -it try1-9869bdb88-rtchc -- /bin/bash
for name in try1-9869bdb88-2wfnr try1-9869bdb88-6bkn
do
kubectl exec $name touch /tmp/healthy
done
```

```yaml

```
