# operator-sdk  
https://github.com/operator-framework/operator-sdk/releases

https://sdk.operatorframework.io/docs/building-operators/golang/quickstart/

## go 
1. create project：
operator-sdk new demo-operator --repo=github.com/owenliang/demo-operator
operator-sdk init --domain=example.com --repo=github.com/example-inc/memcached-operator
2. create api:
operator-sdk create api --group cache --version v1 --kind Memcached --resource=true --controller=true

operator-sdk add api --api-version=app.example.com/v1alpha1 --kind=PodSet



3. 修改结构 api/v1/podset_types.go
reload: 