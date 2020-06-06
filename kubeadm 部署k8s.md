![timg.jpg](https://upload-images.jianshu.io/upload_images/9797242-4ee3ee3920aacd8a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




如果kubernetes的版本为1.8-1.11，docker版本必须为1.11.2-1.13.1和docker-ce版本为17.03.x
如果kubernetes的版本从1.12开始，docker版本必须为17.06/17.09/18.06



#### 1 环境准备
 1. 三台centos服务器
    master 192.168.32.130
    node1  192.168.32.131
    node2  192.168.32.132
    node3  192.168.32.133

 2. 关闭selinux和firewalld
      
```
 setenforce=0
 修改  /etc/selinux/config 中的  SELINUX=disabled

 systemctl stop firewalld
 systemctl disable firewalld
  ```


 3. 这里使用yum安装，需要配置好yum源（这里使用aliyun）  
   分别在三台服务器安装相关源
```
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

1添加docker-ce的yum源


cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

按照官方文档添加k8s的yum源
```

4. 打开桥接
```
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables
```


#### 2 环境部署
1. 安装相关程序执行
 * master节点
```
yum intall -y docker-ce kubelet kubeadm kubectl
```

* node节点(node节点不操作可以不安装kubectl)
```
yum install -y docker-ce kubelet kubeadm 
```

2. 启动docker并设置开机启动（所有节点）
```
systemctl start docker
systemctl enable docker
```
3. 设置kubelet开机启动
```
systemctl enable kubelet
```

4. 初始化主节点
```
kubeadm init --kubernetes-version=v1.11 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=swap
```
这里设置了初始化的版本，pod的网段，service的网段，可以依据`kubeadm init --help`自行修改。同时忽略的swap的报错，在初始化的过程中报错可能会中断初始化，可以根据实际情况依据在`--ignore-preflight-errors=swap`后添加忽略信息忽略。
  
 初始化成功后会生成提示信息
![](https://upload-images.jianshu.io/upload_images/9797242-54249821717c8643.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 按成功提示执行命令(最后一条是当前root用户可不执行)
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
保存`kubeadm jion`信息用于node节点的加入

可用 `kubectl get cs`查看健康情况
可用`kubectl get nodes`查看节点情况

5. 主节点安装配置flannel
Flannel是CoreOS团队针对Kubernetes设计的一个Overlay网络规划服务，简单来说，它的功能是让集群中的不同节点主机创建的Docker容器都具有全集群唯一的虚拟IP地址。项目托管于github上地址https://github.com/coreos/flannel
Kubernetes v1.7以上可直接使用yml安装
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

安装完成后可使用`kubectl get pods -n kube-system`查看各系统名称空间组件READY和STATUS情况

6. 加入node节点
使用master节点初始化时候产生的`kubeadm join`命令进行node节点的加入，可以使用`--ignore-preflight-errors=swap`添加忽略


安装完成后可使用`kubectl get nodes`查看各节点加入情况
可使用`kubectl get pods -n kube-system -o wide`查看各节点系统名称空间组件READY和STATUS情况



#### 3. 坑点
1. 关于swap的忽略
如果初始化是在 `--ignore-preflight-errors=swap`上设置了依然报错则需要
```
vim /etc/sysconfig/kubelet
添加KUBELET_EXTRA_ARGS="--fail-swap-on=false"
```

2. 关于镜像无法加载
在初始化过程中master节点会下载`kube-proxy,kube-controller-manager,kube-apiserver,kube-scheduler,coredns,etcd,pause`等一些镜像，现阶段网络可能访问不到，可在报错信息中或者`/var/log/message`中查看镜像，将文件中镜像地址进行内容替换即可：
将`k8s.gcr.io`替换为
 >registry.cn-hangzhou.aliyuncs.com/google_containers
 registry.aliyuncs.com/google_containers
 mirrorgooglecontainers

以上三选一
使用`docker pull`将下载下来
然后再使用 `docekr tag`将镜像打上需要的`k8s.gcr.io`标签的镜像
node节点需要的镜像可以通过`docker save`将master节点的镜像保存拷贝到node节点之后通过`docker load -i`加载

或者
在`kubeadm init`时加上` --image-repository=registry.aliyuncs.com/google_containers` 
也就是上面的三个镜像仓库地址中的一个都行

3. K8s集群初始化成功后，kubectl get nodes 查看节点信息时报错
报错信息：The connection to the server localhost:8080 was refused - did you specify the right host or port?

解决方法：

执行以下命令
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
再次查看节点信息即正常

4.解决GitHub的raw.githubusercontent.com无法连接问题

修改hosts 
`sudo vi /etc/hosts`
 添加以下内容保存即可 （IP地址查询后相应修改，可以ping不同IP的延时 选择最佳IP地址）

```
# GitHub Start
52.74.223.119 github.com
192.30.253.119 gist.github.com
54.169.195.247 api.github.com
185.199.111.153 assets-cdn.github.com
151.101.76.133 raw.githubusercontent.com
151.101.108.133 user-images.githubusercontent.com
151.101.76.133 gist.githubusercontent.com
151.101.76.133 cloud.githubusercontent.com
151.101.76.133 camo.githubusercontent.com
151.101.76.133 avatars0.githubusercontent.com
151.101.76.133 avatars1.githubusercontent.com
151.101.76.133 avatars2.githubusercontent.com
151.101.76.133 avatars3.githubusercontent.com
151.101.76.133 avatars4.githubusercontent.com
151.101.76.133 avatars5.githubusercontent.com
151.101.76.133 avatars6.githubusercontent.com
151.101.76.133 avatars7.githubusercontent.com
151.101.76.133 avatars8.githubusercontent.com
# GitHub End
```

5. 其它问题
其它问题如果看不出报错的都可以在`/var/log/message`查看问题
一些不可达的问题和解析问题基本都是跟`/etc/hosts`解析相关，需要添加解析信息
同时跟一些忽略相关的可以通过在`--ignore-preflight-errors=`上添加忽略信息
如果显示初始化完成可以使用`kubeadm reset`重置初始化信息

