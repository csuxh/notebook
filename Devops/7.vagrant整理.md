# 基础命令
```bash
vagrant box add centos/7 (或者直接URL)

vagrant box add node1 C:\vmware\work\boxes\vmware_desktop.box
vagrant init node1 (create vagrantfile)
vagrant up
vagrant status/resume/
vagrant ssh node1
vagrant destroy/reload/halt #销毁/重启/关机
vagrant package #打包成box
vagrant global-status #查看所有虚拟机

vagrant reload --provision

vagrant package --output xxx.box box_name
vagrant package --base vm_name -output base.box 
vagrant box add base boxes/base.box

#安装共享工具
vagrant plugin install vagrant-vbguest
排错：  
vagrant vbguest --status
vagrant vbguest --do install master2
内核问题导致vbguest无法安装：
yum install "kernel-devel-uname-r == $(uname -r)"
(必须安装和内核版本一致的kernel-devel和kernel-header)
服务器： systemctl status vboxadd.service  

```
./.vagrant.d/gems/2.4.9/gems/vagrant-vbguest-0.22.1/lib/vagrant-vbguest/installers/centos.rb

# provision
3种方式调用  
vagrant up/provision/reload --prevision
--provision-with(多个时指定某一个)

如果想在“提示消息”中提示用户有可选的 provisioner，或者 provisioner 在启动之前依赖其他配置，也可以将 run 选项设置为“never”。之后可以通过 vagrant provision --provision-with bootstrap 调用。
如果使用块格式，则必须在块外指定 run 这个选项，如下所示：

Vagrant.configure("2") do |config|
  config.vm.provision "shell", run: "always" do |s|
    s.inline = "echo hello"
  end
end

https://vagrantcloud.com/centos/boxes/7/versions/1905.1/providers/virtualbox.box

vagrant镜像下载地址：
http://www.vagrantbox.es/

网络：
▲NAT : 缺省创建，用于让vm可以通过host转发访问局域网甚至互联网。
▲host-only : 只有主机可以访问vm，其他机器无法访问它。
▲bridge : 此模式下vm就像局域网中的一台独立的机器，可以被其他机器访问。


ansible -i hosts elp -m shell -a "su - tomcat -c '/home/tomcat/apache-tomcat-8.5.23/restart.sh'" -uhccnansible -k -b

sar -P ALL  查看多核CPU使用