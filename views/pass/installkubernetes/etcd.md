# etcd

#### 安装及配置CFSSL(主节点执行)
主节点创建证书工具
```shell
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
```
#### ssh-key认证(主节点执行)
配置主节点可以登录任意节点，为了分发文件到其他节点
**过程中：**文件名字 /root/.ssh/id_rsa 密码无
```shell
$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:FQjjiRDp8IKGT+UDM+GbQLBzF3DqDJ+pKnMIcHGyO/o root@qas-k8s-master01
The key's randomart image is:
+---[RSA 2048]----+
|o.==o o. ..      |
|ooB+o+ o.  .     |
|B++@o o   .      |
|=X**o    .       |
|o=O. .  S        |
|..+              |
|oo .             |
|* .              |
|o+E              |
+----[SHA256]-----+

$ ssh-copy-id 192.168.15.131
$ ssh-copy-id 192.168.15.132
```
#### 创建ca和证书(主节点执行)
**注意更改host字段**
```shell
# 可以随便建个etcd目录,这里我选择之前建好的/home/etcd,在目录里创建证书，后面用到的时候拷贝过去就好
cd /home/etcd
# 创建 ETCD 证书
cat << EOF | tee ca-config.json
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "www": {
         "expiry": "87600h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
EOF

# 创建 ETCD CA 配置文件
cat << EOF | tee ca-csr.json
{
    "CN": "etcd CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shenzhen",
            "ST": "Shenzhen"
        }
    ]
}
EOF

# 创建 ETCD Server 证书（注意更改host字段）
cat << EOF | tee server-csr.json
{
    "CN": "etcd",
    "hosts": [
    "192.168.15.130",
    "192.168.15.131",
    "192.168.15.132"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shenzhen",
            "ST": "Shenzhen"
        }
    ]
}
EOF

# 生成 ETCD CA 证书和私钥
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server

# 拷贝证书文件
# cd etcd证书目录
cp ca*pem server*pem /k8s/etcd/ssl
```
#### 找到etcd源码解压安装(主节点执行)
这里我从本地上传，安装到之前创建好的目录/k8s/etcd/bin/
```shell
cd /home/etcd
# 解压安装文件
tar -xvf etcd-v3.3.10-linux-amd64.tar.gz
cd etcd-v3.3.10-linux-amd64/
cp etcd etcdctl /k8s/etcd/bin/
```
#### 配置主节点etcd.service(主节点执行)
# 创建 etcd的 systemd unit 文件
vi /usr/lib/systemd/system/etcd.service 
```shell
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=/k8s/etcd/cfg/etcd
ExecStart=/k8s/etcd/bin/etcd \
--name=${ETCD_NAME} \
--data-dir=${ETCD_DATA_DIR} \
--listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
--listen-client-urls=${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
--advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
--initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
--initial-cluster=${ETCD_INITIAL_CLUSTER} \
--initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
--initial-cluster-state=new \
--cert-file=/k8s/etcd/ssl/server.pem \
--key-file=/k8s/etcd/ssl/server-key.pem \
--peer-cert-file=/k8s/etcd/ssl/server.pem \
--peer-key-file=/k8s/etcd/ssl/server-key.pem \
--trusted-ca-file=/k8s/etcd/ssl/ca.pem \
--peer-trusted-ca-file=/k8s/etcd/ssl/ca.pem
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
上面etcd.service使用到的EnvironmentFile；注意：当前ETCD_NAME="etcd01"，当前节点IP
vi /k8s/etcd/cfg/etcd  
```shell
#[Member]
ETCD_NAME="{当前name}"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://{当前节点IP}:2380"
ETCD_LISTEN_CLIENT_URLS="https://{当前节点IP}:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://{当前节点IP}:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://{当前节点IP}0:2379"
ETCD_INITIAL_CLUSTER="etcd01=https://192.168.15.130:2380,etcd02=https://192.168.15.131:2380,etcd03=https://192.168.15.132:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```
#### 分发etcd.service、EnvironmentFile、证书(主节点执行)
```shell
# 将启动文件、配置文件拷贝到 节点1、节点2
cd /k8s/ 
scp -r etcd 192.168.15.131:/k8s/
scp -r etcd 192.168.15.132:/k8s/
scp /usr/lib/systemd/system/etcd.service  192.168.15.131:/usr/lib/systemd/system/etcd.service
scp /usr/lib/systemd/system/etcd.service  192.168.15.132:/usr/lib/systemd/system/etcd.service 
```
#### 修改EnvironmentFile中的ETCD_NAME和当前节点IP(从点执行)
```shell
#[Member]
ETCD_NAME="{当前name}"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://{当前节点IP}:2380"
ETCD_LISTEN_CLIENT_URLS="https://{当前节点IP}:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://{当前节点IP}:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://{当前节点IP}0:2379"
ETCD_INITIAL_CLUSTER="etcd01=https://192.168.15.130:2380,etcd02=https://192.168.15.131:2380,etcd03=https://192.168.15.132:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```
#### 启动服务（三节点都执行）
```shell
# 所有的机器启动ETCD服务
systemctl daemon-reload
systemctl start etcd
```
配置开机自启动
```shell
systemctl enable etcd
```
#### 验证集群是否正常运行(主节点执行)
```shell
/k8s/etcd/bin/etcdctl --ca-file=/k8s/etcd/ssl/ca.pem --cert-file=/k8s/etcd/ssl/server.pem --key-file=/k8s/etcd/ssl/server-key.pem --endpoints="https://192.168.15.130:2379,https://192.168.15.131:2379,https://192.168.15.132:2379" cluster-health

```
#### 重装（三节点都执行）：
1、清理节点数据
rm -rf /var/lib/etcd/default.etcd
2、从新启动
systemctl daemon-reload
systemctl start etcd
