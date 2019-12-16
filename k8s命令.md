# kubectl create
`kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods`
`kubecttl create rolebinding test-binding --clusterrole=admin --user=jack --namespace=test`

`kubectl explain pv --recursive`

kubectl get events

kubectl get xxx -o name
kubectl delete deployment,services -l app=nginx
kubectl get $(kubectl create -f docs/concepts/cluster-administration/nginx/ -o name | grep service)
kubectl apply -f project/k8s/development --recursive (-R) 目录递
kubectl get pods -Lapp -Ltier -Lrole (小l过滤指定label;大写l指定列)
kubectl get pods -lapp=guestbook,role=slave
kubectl label pods -l app=nginx tier=fe #所有app=nginx的打上标签tier=fe
kubectl annotate pods xxx description=xxx

kubectl autoscale deployment/my-nginx --min=1 --max=3

kubectl edit xxx 环境变量： EDITOR or KUBE_EDITOR
<details>
  <summary>Canary deployments</summary>

```yaml
     name: frontend
     replicas: 3
     ...
     labels:
        app: guestbook
        tier: frontend
        track: stable
     ...
     image: gb-frontend:v3
---
     name: frontend-canary
     replicas: 1
     ...
     labels:
        app: guestbook
        tier: frontend
        track: canary
     ...
     image: gb-frontend:v4
---
  selector:
     app: guestbook
     tier: frontend
```
</details>

