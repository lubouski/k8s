# Kubernetes notes

Hope you already know how to install k8s 

## After install 
```
$ kubectl cluster-info
```
```
$ kubectl get nodes
```
### Run some thing from repo
```
$ kubectl create -f pod.yml
```
```
$ kubectl delete pods jsapp
```
```
$ kubectl describe pod jsapp
```
```
$ kubectl logs --tail=1 --since=24h jsapp 
```
### AutoScale:
```
$ kubectl autoscale deployment jsapp --min=2 --max=6 --cpu-percent=60
```
### Update deployment
```
$ kubectl apply -f jsapp-update.yml
```
### Service imperative way
```
$ kubectl expose deployment jsapp --type=LoadBalancer --name=my-service
```

