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

openssl x509  -noout -text -in  kubernetes.pem
cfssl-certinfo -cert kubernetes.pem
### cfssl  
1. 配置文件ca-config.json, ca-csr.json  
    
2. 生成 CA 证书和私钥    
cfssl gencert -initca ca-csr.json | cfssljson -bare ca

3. 创建kubectl专用证书
* admin-csr.json  
该证书只会被 kubectl 当做 client 证书使用，所以 hosts 字段为空
* 创建证书和私钥 
`cfssl gencert -ca=/opt/k8s/work/ca.pem \
  -ca-key=/opt/k8s/work/ca-key.pem \
  -config=/opt/k8s/work/ca-config.json \
  -profile=kubernetes admin-csr.json | cfssljson -bare admin`
  
4. 配置kubeconfig
* 设置集群参数
kubectl config set-cluster kubernetes \
  --certificate-authority=/cfssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kubectl.kubeconfig
* 设置客户端认证参数
kubectl config set-credentials admin \
  --client-certificate=cfssl/admin.pem \
  --client-key=cfssl/admin-key.pem \
  --embed-certs=true \
  --kubeconfig=kubectl.kubeconfig
* 设置上下文参数
kubectl config set-context kubernetes \
  --cluster=kubernetes \
  --user=admin \
  --kubeconfig=kubectl.kubeconfig
## 设置默认上下文
kubectl config use-context kubernetes --kubeconfig=kubectl.kubeconfig
* 检查：
kubectl config view --kubeconfig=kubectl.kubeconfig

## 基于CA签署证书


## ETCD

命令：
ETCDCTL_API=3 etcdctl --endpoints=https://192.168.56.211:2379 --cert=/etc/etcd/cert/etcd.pem --cacert=/etc/kubernetes/cert/ca.pem --key=/etc/etcd/cert/etcd-key.pem  member list
endpoint status
endpoint health

export ETCD_ENDPOINTS="https://192.168.56.211:2379,https://192.168.56.212:2379,https://192.168.56.213:2379"

ETCDCTL_API=3 etcdctl --endpoints=${ETCD_ENDPOINTS} --cert=/etc/etcd/cert/etcd.pem --cacert=/etc/kubernetes/cert/ca.pem --key=/etc/etcd/cert/etcd-key.pem -w table  endpoint status


etcdctl --endpoints="https://192.168.56.211:2379,https://192.168.56.212:2379,https://192.168.56.213:2379" --cacert=/etc/kubernetes/cert/ca.pem --cert=/etc/etcd/cert/etcd.pem --key=/etc/etcd/cert/etcd-key.pem endpoint status

## apiserver
启动参数：  
--bind-address： https 监听的 IP，不能为 127.0.0.1，否则外界不能访问它的安全端口 6443 (这里配了之后,controller-manager/kube-scheduler的kubeconfig文件里需要配cluster-ip为nodeip, 不能直接写127.0.0.1)
--insecure-port=0：关闭监听 http 非安全端口(8080) 

* 默认apiserver用户貌似只有get list权限  
(error: unable to upgrade connection: Forbidden (user=apiserver, verb=create, resource=nodes, subresource=proxy))
kubectl create clusterrolebinding apiserver-kubelet-admin --user=apiserver --clusterrole=system:kubelet-api-admin

## controller-manager
生成key  
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager  
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler   

查看当前leader  
kubectl get endpoints kube-controller-manager --namespace=kube-system  -o yaml
## scheduler

curl -s http://192.168.56.212:10251/metrics
curl -s --cacert ca.pem --cert admin.pem --key admin-key.pem https://192.168.56.212:10259/metrics
查看当前leader:  
kubectl get endpoints kube-scheduler --namespace=kube-system  -o yaml

## network  
### flanneld  
etcdctl --endpoints=${ETCD_ENDPOINTS} --ca-file=ca.pem --cert-file=flanneld.pem --key-file=flanneld-key.pem mk ${FLANNEL_ETCD_PREFIX}/config '{"Network":"'${CLUSTER_CIDR}'", "SubnetLen": 21, "Backend": {"Type": "vxlan"}}'

etcdctl \
  --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/cert/ca.pem \
  --cert-file=/etc/kubernetes/cert/flanneld.pem \
  --key-file=/etc/kubernetes/cert/flanneld-key.pem \
  get ${FLANNEL_ETCD_PREFIX}/config 或者 ls ${FLANNEL_ETCD_PREFIX}/subnets 

* 部署flannel需要修改docker启动参数
 EnvironmentFile=-/run/flannel/docker
 ExecStart=/usr/bin/dockerd --log-level=error $DOCKER_NETWORK_OPTIONS

## client端
### kubelet
kubeadm token list 
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers
kubectl get csr
kubectl get nodes

kubelet 启动后  
`
[root@master1 installation]# netstat -tnlp | grep kubelet
tcp        0      0 192.168.56.211:10248    0.0.0.0:*               LISTEN      14504/kubelet
tcp        0      0 192.168.56.211:10250    0.0.0.0:*               LISTEN      14504/kubelet
tcp        0      0 127.0.0.1:35451         0.0.0.0:*               LISTEN      14504/kubelet
`


### harbor和imagepullsecret  
* 生成证书->配置到harbor.cfg->拷贝证书到/etc/docker/certs.d/harbor-domain-name/ca.crt,docker login harbor-domain-name，会在~/.docker/docker下自动生成dockerconfig  
* 配置dockerregistry类型的secret  
kubectl create secret  docker-registry harbor-secret --docker-username=k8s --docker-email=k8s@jack.com --docker-server=harbor.jack.com --docker-password=Jack@123456 -n dev


# 九.Network 11%
1. Network configuration on cluster nodes
2. Pod network concepts
3. Service network
4. Load balancer
5. Ingress rules
6. Cluster DNS 
7. Understand CNI

## flannel
* 运行配置  
[root@master1 ~]# cat /run/flannel/docker
DOCKER_OPT_BIP="--bip=172.30.64.1/21"
DOCKER_OPT_IPMASQ="--ip-masq=false"
DOCKER_OPT_MTU="--mtu=1450"
DOCKER_NETWORK_OPTIONS=" --bip=172.30.64.1/21 --ip-masq=false --mtu=1450"  
[root@master1 ~]# cat /run/flannel/subnet.env
FLANNEL_NETWORK=172.30.0.0/16
FLANNEL_SUBNET=172.30.64.1/21
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true

### 网络命名空间  
docker网络  
docker network ls
docker network inspect 1242c6c31700
route -n

a. ip netns help/ls/add xxx/'exec xxx ip addr'  
b. 新创建的network namespace默认在/var/run/netns/, 非netns创建的可以链接过来再管理  
   ln -s /var/run/docker/netns/e9ee44902de3  /var/run/netns/docker01  
   ip netns exec docker01 ip addr  
c. 切换到指定netns  
ip netns exec ns1 /bin/bash --rcfile <(echo "PS1=\"namespace ns1> \"")  

ip link add type veth (创建veth对)  
ip link(只能看到链路层状态)  
ip link set veth0 netns netns01  
ip link set veth1 netns netns02  
![veth对](https://cizixs-blog.oss-cn-beijing.aliyuncs.com/728b3d6dgy1fcl8ox6rsyj213o0f8ab7.jpg)

yum install -y bridge-utils  
ip link add br0 type bridge(创建bridge)  
1个veth放到netns01,另一个放到bridge上：  
ip link set dev veth1 netns netns01  && ip netns exec netns01 ip link set dev eth0 up  
ip link set dev veth0 master br0  && ip link set dev veth0 up  
查看bridge状态：  
bridge link/ brctl show  
![bridge模式](https://cizixs-blog.oss-cn-beijing.aliyuncs.com/728b3d6dgy1fcl8khvmjfj21hc0u0770.jpg)  

### backend  
udp/vxlan(基于tunnel协议) 和 host-gw
udp backend通过用户态进程的proxy转发数据；vxlan backend通过内核转发数据；

etcdctl --endpoints=${ETCD_ENDPOINTS}   --ca-file=/etc/kubernetes/cert/ca.pem   --cert-file=/etc/kubernetes/cert/flanneld.pem   --key-file=/etc/kubernetes/cert/flanneld-key.pem   get /kubernetes/network/config
{"Network":"172.30.0.0/16", "SubnetLen": 21, "Backend": {"Type": "vxlan"}}


## calico  
架构  
![架构](https://upload-images.jianshu.io/upload_images/5194360-26e8b4ec4d0a6c59.png?imageMogr2/auto-orient/strip|imageView2/2/w/623/format/webp)
