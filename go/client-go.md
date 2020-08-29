[client-go源码](https://blog.csdn.net/huwh_/article/details/78821805)


client-go对kubernetes资源对象的调用，需要先获取kubernetes的配置信息，即$HOME/.kube/config
整个调用的过程如下：

kubeconfig→rest.config→clientset→具体的client(CoreV1Client)→具体的资源对象(pod)→RESTClient→http.Client→HTTP请求的发送及响应

通过clientset中不同的client和client中不同资源对象的方法实现对kubernetes中资源对象的增删改查等操作，常用的client有CoreV1Client、AppsV1beta1Client、ExtensionsV1beta1Client等

推荐： https://liqiang.io/post/kubernetes-all-about-crd-part03-usage-for-client-go-d831d52e 
## Dynamic Client: 操作麻烦
```go
//构造client
kubeconfig := "xxx.config"
config, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
client, err := dynamic.NewForConfig(config)
// 获取资源
gvr := schema.GroupVersionResource{
  Group: "apps",
  Version: "v1",
  Resource: "deployments",
}
resp, err := client.Resource(gvr).Namespace(namespace).Get("deployment-name", metav1.GetOptions{})
// 使用
name, found, err := unstructured.NestedString(resp.Object, "metadata", "name")
log4go.Info("deployment name: %s", name)
```

## typed client: 使用方便，如何识别crd?

k8s.io\api\apps\v1beta2\types.go 结构描述


podSpec:  k8s.io\api\core\v1\types.go


### clientset
### Informer
Informer 的作用就是，监听 API Server 的资源变动事件，然后根据事件在本地缓存 Kubernetes 的资源信息（内部有个 Cache），然后 Client 关于 Kubernetes 资源的查询都从 Informer 上来，同时，Informer 也会同步得将 Client 关注的事件传回给 Client

### Event
订阅API Server中的资源变化
### Work Queue


## 自定义 controller 
https://liqiang.io/post/kubernetes-all-about-crd-part05-controller-32c93cc8
https://github.com/kubernetes/sample-controller 
