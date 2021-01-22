# flannel

#### 向etcd写入网段信息
# 注意：
1.flanneld 当前版本 (v0.10.0) 不支持 etcd v3，故使用 etcd v2 API 写入配置 key 和网段数据；
2.！！！写入的 Pod 网段 ${CLUSTER_CIDR} 必须是 /16 段地址，必须与 kube-controller-manager 的 –cluster-cidr 参数值一致；
export ETCDCTL_API=2
```shell
cd /k8s/etcd/ssl/

/k8s/etcd/bin/etcdctl \
--ca-file=ca.pem --cert-file=server.pem \
--key-file=server-key.pem \
--endpoints="https://192.168.226.130:2379,\
https://192.168.226.131:2379,https://192.168.226.132:2379" \
set /coreos.com/network/config  '{ "Network": "172.18.0.0/16", "Backend": {"Type": "vxlan"}}'
```
#### 解压安装
tar -xvf flannel-v0.10.0-linux-amd64.tar.gz
mv flanneld mk-docker-opts.sh /k8s/kubernetes/bin/
#### 配置Flannel配置
```shell
# 配置Flannel
vim /k8s/kubernetes/cfg/flanneld

FLANNEL_OPTIONS="--etcd-endpoints=https://10.67.34.130:2379,https://10.67.34.131:2379,https://10.67.34.132:2379 -etcd-cafile=/k8s/etcd/ssl/ca.pem -etcd-certfile=/k8s/etcd/ssl/server.pem -etcd-keyfile=/k8s/etcd/ssl/server-key.pem"

```
#### 创建 flanneld 的 systemd 文件
```shell
vim /usr/lib/systemd/system/flanneld.service

[Unit]
Description=Flanneld overlay address etcd agent
After=network-online.target network.target
Before=docker.service

[Service]
Type=notify
EnvironmentFile=/k8s/kubernetes/cfg/flanneld
ExecStart=/k8s/kubernetes/bin/flanneld --ip-masq $FLANNEL_OPTIONS
ExecStartPost=/k8s/kubernetes/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/subnet.env
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
#### 配置Docker启动指定子网段
EnvironmentFile=/run/flannel/subnet.env
ExecStart=/usr/bin/dockerd $DOCKER_NETWORK_OPTIONS
```shell
vim /usr/lib/systemd/system/docker.service 

[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=/run/flannel/subnet.env
ExecStart=/usr/bin/dockerd $DOCKER_NETWORK_OPTIONS
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```
#### 启动服务
systemctl daemon-reload
systemctl start flanneld
systemctl enable flanneld
systemctl restart docker
#### 查看是否生效
```shell
ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:e3:57:a4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.226.130/24 brd 172.16.8.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fee3:57a4/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:cf:5d:a7:af brd ff:ff:ff:ff:ff:ff
    inet 172.18.25.1/24 brd 172.18.25.255 scope global docker0
       valid_lft forever preferred_lft forever
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
    link/ether 0e:bf:c5:3b:4d:59 brd ff:ff:ff:ff:ff:ff
    inet 172.18.25.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::cbf:c5ff:fe3b:4d59/64 scope link 
       valid_lft forever preferred_lft forever
```
#### 分发：
```shell
scp -r kubernetes 192.168.226.131:/k8s/
scp -r kubernetes 192.168.226.132:/k8s/
scp /k8s/kubernetes/cfg/flanneld 192.168.226.131:/k8s/kubernetes/cfg/flanneld
scp /k8s/kubernetes/cfg/flanneld 192.168.226.132:/k8s/kubernetes/cfg/flanneld
scp /usr/lib/systemd/system/docker.service  192.168.226.131:/usr/lib/systemd/system/docker.service 
scp /usr/lib/systemd/system/docker.service  192.168.226.132:/usr/lib/systemd/system/docker.service
scp /usr/lib/systemd/system/flanneld.service  192.168.226.131:/usr/lib/systemd/system/flanneld.service 
scp /usr/lib/systemd/system/flanneld.service  192.168.226.132:/usr/lib/systemd/system/flanneld.service
scp /k8s/kubernetes/bin/flanneld 192.168.226.131:/k8s/kubernetes/bin/flanneld
scp /k8s/kubernetes/bin/flanneld 192.168.226.132:/k8s/kubernetes/bin/flanneld
scp /k8s/kubernetes/bin/mk-docker-opts.sh 192.168.226.131:/k8s/kubernetes/bin/mk-docker-opts.sh
scp /k8s/kubernetes/bin/mk-docker-opts.sh 192.168.226.132:/k8s/kubernetes/bin/mk-docker-opts.sh
```


