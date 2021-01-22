# node（kubelet、kube-proxy）

#### 创建 kubelet bootstrap kubeconfig 文件
cd 到kubernetes证书目录，在目录下创建environment.sh
vim  environment.sh
注意：BOOTSTRAP_TOKEN=2366a641f656a0a025abb4aabda4511b 为之前的api-server的那个 TLS Bootstrapping Token
chmod +x environment.sh && ./environment.sh
```shell
# 创建kubelet bootstrapping kubeconfig 
BOOTSTRAP_TOKEN=2366a641f656a0a025abb4aabda4511b
KUBE_APISERVER="https://192.168.226.130:6443"
# 设置集群参数
kubectl config set-cluster kubernetes \
  --certificate-authority=./ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=bootstrap.kubeconfig

# 设置客户端认证参数
kubectl config set-credentials kubelet-bootstrap \
  --token=${BOOTSTRAP_TOKEN} \
  --kubeconfig=bootstrap.kubeconfig

# 设置上下文参数
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kubelet-bootstrap \
  --kubeconfig=bootstrap.kubeconfig

# 设置默认上下文
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig

#----------------------

# 创建kube-proxy kubeconfig文件

kubectl config set-cluster kubernetes \
  --certificate-authority=./ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config set-credentials kube-proxy \
  --client-certificate=./kube-proxy.pem \
  --client-key=./kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```
#### 分发
```shell
#kubelet、kube-proxy二进制文件
cp kubelet kube-proxy /k8s/kubernetes/bin/
scp kubelet kube-proxy 192.168.226.131:/k8s/kubernetes/bin/
scp kubelet kube-proxy 192.168.226.132:/k8s/kubernetes/bin/
# 将bootstrap kubeconfig kube-proxy.kubeconfig 文件
cp bootstrap.kubeconfig kube-proxy.kubeconfig /k8s/kubernetes/cfg/
scp bootstrap.kubeconfig kube-proxy.kubeconfig 192.168.226.131:/k8s/kubernetes/cfg/
scp bootstrap.kubeconfig kube-proxy.kubeconfig 192.168.226.132:/k8s/kubernetes/cfg/
```
#### kubelet
vim /k8s/kubernetes/cfg/kubelet.config
```shell
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 192.168.226.130
port: 10250
readOnlyPort: 10255
cgroupDriver: cgroupfs
clusterDNS: ["10.0.0.2"]
clusterDomain: cluster.local.
failSwapOn: false
authentication:
  anonymous:
    enabled: true
```
vim /k8s/kubernetes/cfg/kubelet
```shell
KUBELET_OPTS="--logtostderr=true \
--v=4 \
--hostname-override=192.168.226.130 \
--kubeconfig=/k8s/kubernetes/cfg/kubelet.kubeconfig \
--bootstrap-kubeconfig=/k8s/kubernetes/cfg/bootstrap.kubeconfig \
--config=/k8s/kubernetes/cfg/kubelet.config \
--cert-dir=/k8s/kubernetes/ssl \
--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0"
```
vim /usr/lib/systemd/system/kubelet.service 
```shell
[Unit]
Description=Kubernetes Kubelet
After=docker.service
Requires=docker.service

[Service]
EnvironmentFile=/k8s/kubernetes/cfg/kubelet
ExecStart=/k8s/kubernetes/bin/kubelet $KUBELET_OPTS
Restart=on-failure
KillMode=process

[Install]
WantedBy=multi-user.target
```
```shell
# 启动服务
systemctl daemon-reload
systemctl enable kubelet
systemctl restart kubelet
```
#### kube-proxy
vim /k8s/kubernetes/cfg/kube-proxy
```shell
KUBE_PROXY_OPTS="--logtostderr=true \
--v=4 \
--hostname-override=192.168.226.130 \
--cluster-cidr=10.0.0.0/24 \
--kubeconfig=/k8s/kubernetes/cfg/kube-proxy.kubeconfig"
```
vim /usr/lib/systemd/system/kube-proxy.service
```shell
[Unit]
Description=Kubernetes Proxy
After=network.target

[Service]
EnvironmentFile=-/k8s/kubernetes/cfg/kube-proxy
ExecStart=/k8s/kubernetes/bin/kube-proxy $KUBE_PROXY_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
```shell
# 启动服务
systemctl daemon-reload
systemctl enable kube-proxy
systemctl restart kube-proxy
```
#### approve kubelet CSR（master执行）
```shell
# 查看 CSR 列表：
$ kubectl get csr
NAME                                                   AGE    REQUESTOR           CONDITION
node-csr-An1VRgJ7FEMMF_uyy6iPjyF5ahuLx6tJMbk2SMthwLs   39m    kubelet-bootstrap   Pending
node-csr-dWPIyP_vD1w5gBS4iTZ6V5SJwbrdMx05YyybmbW3U5s   5m5s   kubelet-bootstrap   Pending

$ kubectl certificate approve node-csr-dWPIyP_vD1w5gBS4iTZ6V5SJwbrdMx05YyybmbW3U5s  
certificatesigningrequest.certificates.k8s.io/node-csr-dWPIyP_vD1w5gBS4iTZ6V5SJwbrdMx05YyybmbW3U5s approved

$ kubectl get csr
NAME                                                   AGE     REQUESTOR           CONDITION
node-csr-An1VRgJ7FEMMF_uyy6iPjyF5ahuLx6tJMbk2SMthwLs   41m     kubelet-bootstrap   Approved,Issued
node-csr-dWPIyP_vD1w5gBS4iTZ6V5SJwbrdMx05YyybmbW3U5s   7m32s   kubelet-bootstrap   Approved,Issued

# 查看集群状态
$ kubectl get node,cs
NAME               STATUS   ROLES    AGE    VERSION
node/192.168.226.130   Ready    master   137m   v1.13.0
node/192.168.226.131   Ready    node     114m   v1.13.0
node/192.168.226.132  Ready    node     93m    v1.13.0

NAME                                 STATUS    MESSAGE             ERROR
componentstatus/controller-manager   Healthy   ok                  
componentstatus/scheduler            Healthy   ok                  
componentstatus/etcd-0               Healthy   {"health":"true"}   
componentstatus/etcd-1               Healthy   {"health":"true"}   
componentstatus/etcd-2               Healthy   {"health":"true"}
```


