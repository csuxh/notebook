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
```

https://vagrantcloud.com/centos/boxes/7/versions/1905.1/providers/virtualbox.box

vagrant镜像下载地址：
http://www.vagrantbox.es/

网络：
▲NAT : 缺省创建，用于让vm可以通过host转发访问局域网甚至互联网。
▲host-only : 只有主机可以访问vm，其他机器无法访问它。
▲bridge : 此模式下vm就像局域网中的一台独立的机器，可以被其他机器访问。