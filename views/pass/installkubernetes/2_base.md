# 环境基本处理

#### 
#### 注意：
文中所有IP都要替换：
#### 主机信息：
192.168.15.130 master
192.168.15.131 node1
192.168.15.132 node2
#### 配置机器名称
```shell
cat <<EOF >>/etc/hosts
192.168.15.130 master
192.168.15.131 node1
192.168.15.132 node2
EOF
```
#### 基础环境配置（三节点都执行）：
**关闭防火墙：**
```shell
systemctl stop firewalld && systemctl disable firewalld
setenforce 0

vi /etc/selinux/config
SELINUX=disabled
```
**关闭swap：**
```shell
swapoff -a && sysctl -w vm.swappiness=0

vi /etc/fstab
#UUID=7bff6243-324c-4587-b550-55dc34018ebf swap                    swap    defaults        0 0
```
#### 安装目录创建（三节点都执行）：
```shell
mkdir /home/etcd -p
mkdir /k8s/etcd/{bin,cfg,ssl} -p
mkdir /k8s/kubernetes/{bin,cfg,ssl} -p
```
