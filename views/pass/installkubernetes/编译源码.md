# 编译源码

### 编译：
1、安装go环境：略
2、安装git：略
3、下载代码
```go
git clone https://github.com/kubernetes/kubernetes.git
```
4、编译kubelet：
其中KUBE_BUILD_PLATFORMS=linux/amd64指定目标平台为linux/amd64，GOFLAGS=-v开启verbose日志，GOGCFLAGS=”-N -l”禁止编译优化和内联，减小可执行程序大小
```go
#yum install -y rsync
KUBE_BUILD_PLATFORMS=linux/amd64 make all WHAT=cmd/kubelet GOFLAGS=-v GOGCFLAGS="-N -l"
```
5、编译kube-controller-manager：
```go
KUBE_BUILD_PLATFORMS=linux/amd64 make all WHAT=cmd/kube-controller-manager GOFLAGS=-v GOGCFLAGS="-N -l"
```
6、编译其他：略
#### 单组件逐个编译：
```shell
./hack/build-go.sh cmd/kubemark/
	cp $GOPATH/src/k8s.io/kubernetes/_output/bin/kubemark $GOPATH/src/k8s.io/kubernetes/cluster/images/kubemark/
	cd $GOPATH/src/k8s.io/kubernetes/cluster/images/kubemark/
	docker build -t harbor.58corp.com/k8s/kubemark:frozen .
	docker save > kubemark.tar harbor.58corp.com/k8s/kubemark:frozen
	sudo docker load < kubemark.tar
```


