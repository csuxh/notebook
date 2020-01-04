# 并发和异步  优化  
## 参数调优
* 开启pipelining  
ansible.cfg中添加 pipelining = True
* 修改执行策略  
```yaml
- hosts: all
  strategy: free
  tasks:
```
* 设置facts缓存  
ansible.cfg: 
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /data/ansible_fact_dir  
* 设置ssh长连接  
ansible.cfg:
ssh_args = -C -o ControlMaster=auto -o ControlPersist=7d 

## ad-hoc参数  
-t directory_name   输出结果以按host保存
-B 10 -P 1  -o -f 6 (-B 后台运行超时时间; o: 压缩输出，一行显示； -P 检查是否成功间隔)
-T ssh超时时间，默认10s
