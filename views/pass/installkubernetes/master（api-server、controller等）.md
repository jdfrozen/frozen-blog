# master（api-server、controller等）

#### 将二进制文件解压拷贝到master 节点
```shell
tar -xvf kubernetes-server-linux-amd64.tar.gz 
cd kubernetes/server/bin/
cp kube-scheduler kube-apiserver kube-controller-manager kubectl /k8s/kubernetes/bin/
```
#### TLS Bootstrapping Token
```shell
$ head -c 16 /dev/urandom | od -An -t x | tr -d ' '
2366a641f656a0a025abb4aabda4511b

vim /k8s/kubernetes/cfg/token.csv
2366a641f656a0a025abb4aabda4511b,kubelet-bootstrap,10001,"system:kubelet-bootstrap"
```
#### apiserver
vim /k8s/kubernetes/cfg/kube-apiserver 
```shell
KUBE_APISERVER_OPTS="--logtostderr=true \
--v=4 \
--etcd-servers=https://192.168.226.130:2379,https://192.168.226.131:2379,https://192.168.226.132:2379 \
--bind-address=192.168.226.130 \
--secure-port=6443 \
--advertise-address=192.168.226.130 \
--allow-privileged=true \
--service-cluster-ip-range=10.0.0.0/24 \
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota,NodeRestriction \
--authorization-mode=RBAC,Node \
--enable-bootstrap-token-auth \
--token-auth-file=/k8s/kubernetes/cfg/token.csv \
--service-node-port-range=30000-50000 \
--tls-cert-file=/k8s/kubernetes/ssl/server.pem  \
--tls-private-key-file=/k8s/kubernetes/ssl/server-key.pem \
--client-ca-file=/k8s/kubernetes/ssl/ca.pem \
--service-account-key-file=/k8s/kubernetes/ssl/ca-key.pem \
--etcd-cafile=/k8s/etcd/ssl/ca.pem \
--etcd-certfile=/k8s/etcd/ssl/server.pem \
--etcd-keyfile=/k8s/etcd/ssl/server-key.pem"
```
vim /usr/lib/systemd/system/kube-apiserver.service 
```shell
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=-/k8s/kubernetes/cfg/kube-apiserver
ExecStart=/k8s/kubernetes/bin/kube-apiserver $KUBE_APISERVER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
```shell
# 启动服务
systemctl daemon-reload
systemctl enable kube-apiserver
systemctl restart kube-apiserver
```
#### kube-scheduler
vim  /k8s/kubernetes/cfg/kube-scheduler
```shell
KUBE_SCHEDULER_OPTS="--logtostderr=true --v=4 --master=127.0.0.1:8080 --leader-elect"
```
vim /usr/lib/systemd/system/kube-scheduler.service 
```shell
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=-/k8s/kubernetes/cfg/kube-scheduler
ExecStart=/k8s/kubernetes/bin/kube-scheduler $KUBE_SCHEDULER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
```shell
# 启动服务
systemctl daemon-reload
systemctl enable kube-scheduler.service 
systemctl restart kube-scheduler.service
```
#### kube-controller-manager 
vim /k8s/kubernetes/cfg/kube-controller-manager
```shell
KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=true \
--v=4 \
--master=127.0.0.1:8080 \
--leader-elect=true \
--address=127.0.0.1 \
--service-cluster-ip-range=10.0.0.0/24 \
--cluster-name=kubernetes \
--cluster-signing-cert-file=/k8s/kubernetes/ssl/ca.pem \
--cluster-signing-key-file=/k8s/kubernetes/ssl/ca-key.pem  \
--root-ca-file=/k8s/kubernetes/ssl/ca.pem \
--service-account-private-key-file=/k8s/kubernetes/ssl/ca-key.pem"
```
vim /usr/lib/systemd/system/kube-controller-manager.service 
```shell
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=-/k8s/kubernetes/cfg/kube-controller-manager
ExecStart=/k8s/kubernetes/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
```shell
# 启动服务
systemctl daemon-reload
systemctl enable kube-controller-manager
systemctl restart kube-controller-manager
```
#### kubectl
# 将可执行文件路/k8s/kubernetes/ 添加到 PATH 变量中
vim /etc/profile
source /etc/profile
```shell
PATH=/k8s/kubernetes/bin:$PATH:$HOME/bin
```
#### 验证：
```shell
[root@localhost kubernetes]# kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
etcd-2               Healthy   {"health":"true"}   
etcd-0               Healthy   {"health":"true"}   
controller-manager   Healthy   ok                  
etcd-1               Healthy   {"health":"true"}   
scheduler            Healthy   ok 
```
