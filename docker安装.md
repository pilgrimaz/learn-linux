####docker安装
1. 内核支持
   官方要求linux kernel至少3.8以上，且docker只能运行在64位系统中
   内核版本低请先升级内核(方法见网络）
   Docker支持Aufs，Devicemapper，Btrfs和Vfs四种文件系统

2. 安装dock
   - 2.1 安装epel
     yum -y install epel-release
   - 2.2 yum安装docker-io
     yum -y install docker-io
   - 2.3 启动docker
     /bin/systemctl start docker.service
     注：如启动失败查看日志/var/log/docker
     ```shell
     $cat /var/log/docker
     time="2016-06-25T18:54:56.380986865+08:00" level=fatal msg="Error starting daemon: Error initializing network controller: Error creating default \"bridge\" network: can't find an address range for interface \"docker0\""
     ```
     出现上面错误说明docker0接口选择的网段有冲突，解决方法
     解决方法有两种：
      方法一：修改/etc/default/docker，添加DOCKER_OPTS=”–bip=192.168.17.1/24”，重启即可。注意不要用192.168.0.1/24，这段地址也被占用了。
      方法二：启动docker服务在指定的网段。sudo docker –bip 192.168.100.1/24 -d &
     建议第一种
   - 2.4 查看docker版本
     利用命令docker version
   - 2.5 开机启动docker
     /bin/systemctl enable docker.service

3. docker命令的使用
  - 3.1 直接输入docker命令来查看所有的optios和conmmands
    查看某一个command的详细使用方法：docker COMMAND -help 
  - 3.2 搜索可用的docker镜像：docker search NAME
    注：修改镜像源方式
    修改Docker配置文件/etc/default/docker
    DOCKER_OPTS="--registry-mirror=镜像源地址"
    修改后重启docker
    ervice docker restart
  - 3.3 下载镜像：docker pull centos[:TAG]
    比如获取最新的centos镜像：docker pull centos：lastest
    注：这里用写用docker search搜索到的完整的镜像名
  - 3.4 查看安装的镜像：docker images [NAME]
  - 3.5 在docker容器中运行命令：docker run IMAGE [COMMAND][ARG…]
    docker run命令有两个参数，一个是镜像名，一个是要在镜像中运行的命令。
    注意：IMAGE=REPOSITORY[:TAG]，如果IMAGE参数不指定镜像的TAG，默认TAG为latest。
  - 3.6 列出容器：docker ps -a
    查看最近生成的容器：docker ps -l
    查看正在运行的容器：docker ps
  - 3.7 显示容器的标准输出：docker logs CONTAINERID
    无需拷贝完整的id，一般写最开始的三至四个字符即可。
  - 3.8 在容器中安装新程序，比如安装ifconfig命令（centos7默认没有ifconfig）
    运行镜像，执行ifconfig，找不到此命令。此时进入镜像执行yum install net-tools。
  - 3.9 保存对容器的修改并生成新的镜像：docker commit CONTAINERID [REPOSITORY[:TAG]]
    REPOSITORY参数可以是新的镜像名字，也可以是旧的镜像名；如果和旧的镜像名和TAG都相同，会覆盖掉旧的镜像。
  - 3.10 停止正在运行的容器：docker stop CONTAINERID
    默认等待10秒钟再杀死指定容器。可以使用-t参数来设置等待时间。
  - 3.11 查看容器或镜像的详细信息：docker inspect CONTAINERID|IMAGE
    参数可以是容器的ID或者是镜像名（NAME:TAG）
  - 3.12 删除容器：docker rm CONTAINERID
    查看所有容器ID：docker ps -a -q
    删除所有的容器：docker rm $(docker ps -a -q)
  - 3.13 删除镜像：docker rmi IMAGE
  - 3.14 查看docker的信息，包括Containers和Images数目、kernel版本等。
    docker info

4. 创建容器并登入
- 4.1 创建一个新容器并登入：docker run -i -t IMAGE /bin/bash
- 4.2 启动一个退出的容器：docker start CONTAINERID
- 4.3 attach到运行中的容器：docker attach CONTAINERID