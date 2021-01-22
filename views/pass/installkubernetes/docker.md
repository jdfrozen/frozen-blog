# docker

yum -y install yum-utils
```shell
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 上面的yum源不行的话建议换阿里yum源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum list docker-ce --showduplicates | sort -r
# 最新版本
yum install docker-ce -y
# 指定版本 yum install -y --setopt=obsoletes=0  docker-ce-18.06.1.ce-3.el7
systemctl start docker && systemctl enable docker

```


