<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=3 orderedList=false} -->
# Storage 7%
1. PV/PVC, access mode
2. Storage objects
3. Application with storage
   
## 支持的类型
awsEBS, azureDisk/File, cephfs, cinder, csi, gce, clusterfs,rbd
emptyDir,hostPath,local,nfs,configMap, gitRepo(不推荐使用),secret
pvc

configMap:
kubectl create configmap cmname --from-file=xxx / --from-literal=key1=value1

Projected类型：

## DownwardAPI: env and volume mount
### 环境变量注入
status.podIP, status.hostIP
<details>
  <summary>示例</summary>

```yaml
env:
xxx
  - name: POD_NAME
    valueFrom:
      fieldRef:
        fieldPath: metadata.name
  - name: CPU_LIMIT
    valueFrom:
      resourceFieldRef:
        resource: limits.cpu
```
</details>

### 通过volume挂载 
pod field(label, ip, annotation); container field(requests, limits)
<details>
  <summary>示例</summary>

```yaml
xxx
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "cpu_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.cpu
              divisor: 1m
```
</details>

mount时候指定subpath时无法自动更新数据（not  symlink）
Note: A container using Downward API as a subPath volume mount will not receive Downward API updates.

## secret
自动生成secret: `kubectl apply -k .`
kcustomize.yaml
```yaml
secretGenerator:
- name: db-user-pass
  literals:
  - username=admin
  - password=secret
```
`kubectl create secret generic ssh-key-secret --from-file=ssh-private=/root/.ssh/id_rsa --from-file=ssh-public=/root/.ssh/id_rsa.pub`


# Security 12%
1. Authentication and authorization config
2. Network policies
3. TLS create and manage for cluster components
4. Image securety
5. Security contexts
6. Secure persistent key value store

## Authentication认证  

### openssl  
* 生成私钥文件  
`(umask 077; openssl genrsa -out jackxia.key 2048)`
* 生成证书请求  
`openssl req -new -key jackxia.key -out jackxia.csr -subj "/CN=csuxh/O=linux"`
* 基于k8s集群CA签署证书
  `openssl x509 -req -in jackxia.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out jackxia.crt -days=3650`
* 查看证书  
`openssl x509 -in ca.crt -text -noout`
* 基于证书创建secret  
`kubectl create secret generic jackxia --from-file=jackxia.key=jackxia.key --from-file=jackxia.crt=jackxia.crt`
` kubectl get secret | awk '/^jackxia/{print $1}' ` 
* 上下文 security contexts  
1. 设置集群  
`kubectl config set-cluster cluster02 --embed-certs=true --certificate-authority=jackxia.crt --server="https://192.168.56.210:6443"`  
2. 设置证书
 `kubectl config set-credentials jackxia --embed-certs=true --client-certificate=jackxia.crt --client-key=jackxia.key #add user`
3. 设置上下文
 `kubectl config set-context jackxia --cluster=kubernetes --user=jackxia --namespace=kube-system`  
4. 配置权限
`kubectl create clusterrolebinding jackxia-admin --clusterrole=cluster-admin --group=linux --context=kubernetes-admin@kubernetes`
5. 其他命令
切换context  
kubectl config use-context jackxia  
临时指定context  
kubectl get pod --context=kubernetes-admin@kubernetes  
6. 生成kubeconfig文件
* kubectl config set-cluster kubernetes --embed-certs=true --certificate-authority=/etc/kubernetes/pki/ca.crt --server="https://192.168.152.128:6443" --kubeconfig=./jackxia.kubeconfig
* __配置token__  
获取token  
`dash_token=$(kubectl get secret/dashboard-admin-token-jkf99 -o jsonpath={.data.token} | base64 -d)`  
`kubectl config set-credentials jackxia --token=${dash_token} --kubeconfig=jackxia.kubeconfig`
* 设置context  
kubectl config set-context jackxia --cluster=kubernetes --user=jackxia --kubeconfig=./jackxia.kubeconfig


### 关系：
角色(role,clusterrole) 对应权限(verb, resource ...) verb支持 get list watch create update patch proxy redirect delete deletecollection
ServiceAccount: 服务账户(admin,view...)
User,Group：openssl生成
RoleBinding,ClusterroleBinding 



  

