# kubernetes handbook
## 用户与权限
1. 取消sa自动挂载(支持配置到sa和pod)
```yaml
apiVersion: v1
xxx
automountServiceAccountToken: false
```
