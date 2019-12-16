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

## openssl  
* 生成私钥文件  
`(umask 077; openssl genrsa -out jackxia.key 2048)`
* 生成证书请求  
`openssl req -new -key jackxia.key -out jackxia.csr -subj "/CN=csuxh/O=linux"`
* 基于k8s集群CA签署证书
  `openssl x509 -req -in jackxia.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out jackxia.crt -days=3650`
* 查看证书  
`openssl x509 -in ca.crt -text -noout`
