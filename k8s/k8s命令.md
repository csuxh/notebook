# kubectl create
`kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods`
`kubecttl create rolebinding test-binding --clusterrole=admin --user=jack --namespace=test`

`kubectl explain pv --recursive`

kubectl run -it cirros --image=busybox --rm --restart=Never sh

ansible部署k8s:
https://github.com/easzlab/kubeasz

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



https://github.com/easzlab/kubeasz



# 导出资源清单
```shell
#[root@s02-23-y32-aicp-03 ~]# cat backup/export.sh
#!/usr/bin/bash
BACKUP_PATH=/root/backup
#BACKUP_PATH_BIN=$BACKUP_PATH/bin
BACKUP_PATH_DATA=$BACKUP_PATH/`date +%Y%m%d%H%M%S`
#BACKUP_PATH_LOG=$BACKUP_PATH/log
#BACKUP_LOG_FILE=$BACKUP_PATH_LOG/k8s-backup.log

#set resource type
#CONFIG_TYPE="service deploy configmap secret job cronjob replicaset daemonset statefulset"
CONFIG_TYPE="deploy daemonset statefulset"

# make dir
#mkdir -p $BACKUP_PATH_BIN
mkdir -p $BACKUP_PATH_DATA
#mkdir -p $BACKUP_PATH_LOG
cd $BACKUP_PATH_DATA

# get namespace lists
ns_list=`kubectl get ns | awk '{print $1}' | grep -v NAME`
if [ $# -ge 1 ]; then
  ns_list="$@"
fi

#COUNT0=0

# backup resources
for ns in $ns_list; do
  #echo "`date` Backup No.${COUNT0} namespace [namespace: ${ns}]." 2>&1 >>$BACKUP_LOG_FILE
  ## loop for types
  for type in $CONFIG_TYPE; do
    #echo "`date` Backup type [namespace: ${ns}, type: ${type}]." 2>&1 >> $BACKUP_LOG_FILE
    item_list=`kubectl -n $ns get $type | awk '{print $1}' | grep -v NAME | grep -v "No "`
    ## loop for items
    for item in $item_list; do
        file_name=$BACKUP_PATH_DATA/${ns}_${type}_${item}.yaml
        #echo "`date` Backup kubernetes config yaml [namespace: ${ns}, type: ${type}, item: ${item}] to file: ${file_name}" 2>&1 >> $BACKUP_LOG_FILE
        echo "`date` Backup:[namespace: ${ns}, type: ${type}, item: ${item}] "
        kubectl -n $ns get $type $item -o yaml > $file_name
    done;
  done;
  #COUNT0=`expr $COUNT2 + 1`
done;
```


GVK列表：
kubectl api-resources



kubectl proxy --port=9090
curl http://127.0.0.1:9090/apis/apps/v1/daemonsets/



# 部署
kind&Sealos
https://www.hi-linux.com/posts/18971.html