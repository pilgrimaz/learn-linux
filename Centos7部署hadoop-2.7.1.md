#### Centos7部署hadoop-2.7.1

1. **安装环境**
   硬件环境：CentOS  7.0 服务器4台（一台为Master节点，三台为Slave节点）
   软件环境：Java 1.8.0_131、hadoop-2.7.1

2. **用户配置**(所有主机)
  - 2.1 添加一个用户hadoop
    adduser hadoop
    passwd hadoop

3. **sudo权限配置**
   编辑 /etc/sudoers
   缺省只有一条
   root  ALL=(ALL) ALL
   可在下面添加一条
   hadoop  ALL=(ALL) ALL

4. **配置网络主机名**(所有主机)
   配置/etc/hosts
   192.168.25.101 master.hadoop
   192.168.25.102 slave1.hadoop
   192.168.25.102 slave2.hadoop
   192.168.25.102 slave3.hadoop
   按照hosts配置/etc/hostname和IP地址

5. **关闭防火墙**(所有主机)
  systemctl stop firewalld.service
  关闭防火墙开机启动
  systemctl disable firewalld.service

  vi /etc/selinux/config
  设置SELINUX=disabled
  输入setenforce 0  临时关闭

6. **安装JDK(所有主机)**
   - 1. 可选择yum安装和rpm安装
     - yum安装 
        yum install -y java-1.8.0-openjdk
     - rpm安装
        下载rpm包  rpm -ivh 安装包
   - 2. 编辑配置文件编辑”/etc/profile”文件，在后面添加Java的”JAVA_HOME”、”CLASSPATH”以及”PATH”内容。

        vi /etc/profile 后面添加如下内容

        ```shell
        #set java environment
        JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64
        JRE_HOME=$JAVA_HOME/jre
        CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
        PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
        export JAVA_HOME JRE_HOME CLASS_PATH PATH

        #HADOOP
        export HADOOP_HOME=/usr/local/hadoop
        export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
        export HADOOP_HOME_WARN_SUPPRESS=1 
        ```

        分别为java和hadoop环境文件

   - 3. 使配置文件生效
        source /etc/profile
        配置完后可以用 `java -version`检查是否成功

7. **安装hadoop**

   - 1. 下载hadoop 2.7.1文件
        可用 wget下载
        wget ': wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz -P /tmp -b
        此处指定后台下载到tmp目录
   - 2. 解压hadoop压缩包并移动到/usr/local下改名为hadoop
        cd /tmp
        tar -zxvf hadoop-2.7.1.tar.gz
        mv hadoop-2.7.1 /usr/local/hadoop
   - 3. 将文件夹/hadoop分配给hadoop用户
        chown -R hadoop:hadoop /usr/local/hadoop
   - 4. 配置环境变量，环境变量前面已经配置好了

8. **配置SSH无密码验证** 
   Hadoop运行过程中需要管理远端Hadoop守护进程，在Hadoop启动以后，NameNode是通过SSH（Secure Shell）来启动和停止各个DataNode上的各种守护进程的。这就必须在节点之间执行指令的时候是不需要输入密码的形式，故我们需要配置SSH运用无密码公钥认证的形式，这样NameNode使用SSH无密码登录并启动DataName进程，同样原理，DataNode上也能使用SSH无密码登录到 NameNode。
   没有安装的话先安装ssh `yum install -y openssh-server`

   - 1. ssh无密码原理
        Master（NameNode | JobTracker）作为客户端，要实现无密码公钥认证，连接到服务器Salve（DataNode | Tasktracker）上时，需要在Master上生成一个密钥对，包括一个公钥和一个私钥，而后将公钥复制到所有的Slave上。当Master通过SSH连接Salve时，Salve就会生成一个随机数并用Master的公钥对随机数进行加密，并发送给Master。Master收到加密数之后再用私钥解密，并将解密数回传给Slave，Slave确认解密数无误之后就允许Master进行连接了。这就是一个公钥认证过程，其间不需要用户手工输入密码。
   - 2. 配置无密码登录
        在master上利用 ssh-keygen生成无密码密钥对
        需先切换到hadoop用户 
        su - hadoop
        ssh-keygen -t rsa -p ""
        运行后询问其保存路径时直接回车采用默认路径。生成的密钥对：id_rsa（私钥）和id_rsa.pub（公钥），默认存储在"/home/用户名/.ssh"目录下。
        接着在Master节点上做如下配置，把id_rsa.pub追加到授权的key里面去。
        `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`
        复制公钥到复制到远程机器中
        `ssh-copy-id -i .ssh/id_rsa.pub hadoop@slave1.hadoop`
        `ssh-copy-id -i .ssh/id_rsa.pub hadoop@slave2.hadoop`
        `ssh-copy-id -i .ssh/id_rsa.pub hadoop@slave3.hadoop`
        注：ssh-copy-id 的作用是将key写到远程机器的 ~/.ssh/authorized_keys文件中
        查看下authorized_keys的权限，如果权限不对则利用如下命令设置该文件的权限：
        `chmod 600 authorized_keys`
   - 3. 用root用户登录修改SSH配置文件"/etc/ssh/sshd_config"的下列内容。
        检查下面几行前面”#”注释是否取消掉：
        RSAAuthentication yes # 启用 RSA 认证
        PubkeyAuthentication yes # 启用公钥私钥配对认证方式
        AuthorizedKeysFile  %h/.ssh/authorized_keys # 公钥文件路径
   - 4. 如果在用命令ssh-copy-id时发现找不到该命令“ssh-copy-id：Command not found”，则可能是ssh服务的版本太低的原因，比如若你的机器是RedHat系统就可能该问题，解决办法是：手动复制本地的pubkey内容到远程服务器，命令如下：
        cat ~/.ssh/id_rsa.pub | ssh hadoop@slave1.Hadoop 'cat >> ~/.ssh/authorized_keys'
        该命令等价于下面两个命令：
        ①在本地机器上执行：scp ~/.ssh/id_rsa.pub hadoop@slave1.Hadoop:/~
        ②到远程机器上执行：cat ~/id_rsa.pub >> ~/.ssh/authorized_keys

9. **配置hadoop文件**

   \# mkdir -p /usr/local/hadoop/tmp/dfs/name

   \# mkdir -p /usr/local/hadoop/tmp/dfs/data

   \# mkdir -p /usr/local/hadoop/tmp/dfs/namesecondary

   1. 编辑 core-site.xml

   vi /usr/local/hadoop/etc/hadoop/core-site.xml

   ```xml
   <configuration>
   <property>
       <name>fs.defaultFS</name>
       <value>hdfs://master.hadoop:9000</value>
       <description>HDFS的URI，文件系统://namenode标识:端口号</description>
   </property>

   <property>
   <name>io.file.buffer.size</name>
   <value>131072</value>
   </property>

   <property>
       <name>hadoop.tmp.dir</name>
       <value>/usr/local/hadoop/tmp</value>
       <description>namenode上本地的hadoop临时文件夹</description>
   </property>
   </configuration>
   ```
   |       属性名       |            属性值            | 涉及范围 |
   | :-------------: | :-----------------------: | :--: |
   |  fs.defaultFS   | hdfs://master.hadoop:9000 | 所有节点 |
   | hadoop.tmp.dir  |   /usr/local/hadoop/tmp   | 所有节点 |
   | fs.default.name | hdfs://master.hadoop:9000 |      |
   2. 编辑 hdfs-site.xml

      vi /usr/local/hadoop/etc/hadoop/hdfs-site.xml

      ```xml
      <configuration>
      <!--hdfs-site.xml-->
      <property>
          <name>dfs.namenode.name.dir</name>
          <value>/usr/local/hadoop/hdfs/name</value>
          <description>namenode上存储hdfs名字空间元数据 </description>
      </property>

      <property>
          <name>dfs.datanode.data.dir</name>
          <value>/usr/local/hadoop/hdfs/data</value>
          <description>datanode上数据块的物理存储位置</description>
      </property>

      <property>
          <name>dfs.replication</name>
          <value>1</value>
          <description>副本个数，配置默认是3,应小于datanode机器数量</description>
      </property>

      <property>
      <name>dfs.blocksize</name>
      <value>268435456</value>
      </property>

      <property>
      <name>dfs.namenode.handler.count</name>
      <value>100</value>
      </property>

      <property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>master.hadoop:50090</value>
      </property>

      <property>
      <name>dfs.namenode.secondary.https-address</name>
      <value>master.hadoop:50091</value>
      </property>

      </configuration>
      ```

      | 属性名                                 | 属性值                 | 涉及范围                       |
      | ----------------------------------- | ------------------- | -------------------------- |
      | dfs.namenode.http-address           | master.hadoop:50091 | 所有节点                       |
      | dfs.namenode.http-bind-host         | master.hadoop       | 所有节点                       |
      | dfs.namenode.secondary.http-address | master.hadoop:50090 | NameNode、SecondaryNameNode |
      | dfs.replication                     | 1                   |                            |

   3. 编辑 mapred-site.xml

      vi  /usr/local/hadoop/etc/hadoop/mapred-site.xml

      ```xml
      <configuration>

      <property>

      <name>mapreduce.framework.name</name>

      <value>yarn</value>

      </property>

      <property>

      <name>mapreduce.jobtracker.http.address</name>

      <value>master.hadoop:50030</value>

      </property>

      <property>

      <name>mapreduce.jobhistory.address</name>

      <value>master.hadoop:10020</value>

      </property>

      <property>

      <name>mapreduce.jobhistory.webapp.address</name>

      <value>master.hadoop:19888</value>

      </property>

      <property>

      <name>mapreduce.jobhistory.admin.address</name>

      <value>master.hadoop:10033</value>

      </property>

      </configuration>
      ```

      | 属性名                               | 属性值                 | 涉及范围 |
      | --------------------------------- | ------------------- | ---- |
      | mapreduce.framework.name          | yarn                | 所有节点 |
      | mapreduce.jobtracker.http.address | master.hadoop:50030 |      |
      |                                   |                     |      |
      |                                   |                     |      |

   4. 编辑yarn-site.xml

      vi /usr/local/hadoop/etc/hadoop/yarn-site.xml

      ```xml
      <configuration>
      <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
      </property>
      <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>master.hadoop</value>
      </property>
      </configuration>
      ```

      | 属性名                           | 属性值               | 涉及范围 |
      | ----------------------------- | ----------------- | ---- |
      | yarn.resourcemanager.hostname | master.hadoop     | 所有节点 |
      | yarn.nodemanager.aux-services | mapreduce_shuffle | 所有节点 |

   5. 编辑 slaves

      vi /usr/local/hadoop/etc/hadoop/slaves

      slave1.hadoop

      slave2.hadoop

      slave3.hadoop

   6. 编辑 hadoop-env.sh

      vi /usr/local/hadoop/etc/hadoop/hadoop-en.sh

      添加java_home路径为实际路径通过变量有时候会调取不到

      \#export JAVA_HOME=${JAVA_HOME}

      export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7\_3.x86\_64

   7. 编辑yarn-env.sh

      vi /usr/local/hadoop/etc/hadoop/yarn-env.sh

      同样修改java_home路径

      \# export JAVA_HOME=/home/y/libexec/jdk1.6.0/

      export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64

   8. 现在在Master机器上的Hadoop配置就结束了，剩下的就是配置Slave机器上的Hadoop。最简单的方法是将 Master上配置好的hadoop所在文件夹"/usr/local/hadoop"复制到所有的Slave的"/usr/local"目录下（实际上Slave机器上的slavers文件是不必要的， 复制了也没问题）。用下面命令格式进行。（备注：此时用户可以为普通用户也可以为root）   

      scp -r /usr/local/hadoop root@服务器IP:/usr/local/

      例如：从"Master.Hadoop"到"Slave1.Hadoop"复制配置Hadoop的文件。

      scp -r /usr/local/hadoop root@Slave1.Hadoop:/usr/local

      以root用户进行复制，当然不管是用户root还是普通用户，虽然Master机器上的"/usr/local/hadoop"文件夹用户hadoop有权限，但是Slave1上的hadoop用户却没有"/usr/local"权限，所以没有创建文件夹的权限。所以无论是哪个用户进行拷贝，右面都是"root@机器 IP"格式。因为我们只是建立起了普通用户的SSH无密码连接，所以用root进行"scp"时，扔提示让你输入"Slave1.Hadoop" 服务器用户root的密码。

       查看"Slave1.Hadoop"服务器的"/usr"目录下是否已经存在"hadoop"文件夹，确认已经复制成功。 

      hadoop文件夹复制后可能发现hadoop权限是root，所以我们现在要给"Slave1.Hadoop"服务器上的用户hadoop添加对"/usr/local/hadoop"读权限。

      以root用户登录"Slave1.Hadoop"，执行下面命令。

      chown -R hadoop:hadoop（用户名：用户组） hadoop（文件夹）

      注：java和hadoop环境变量前面要配置好

10. 启动及验证

  1. 格式化HDFS文件系统

     在"Master.Hadoop"上使用**普通**用户**hadoop**进行操作。（**备注：**只需一次，下次启动不再需要格式化，只需 start-all.sh）

     `hadoop namenode -format`

  2. 启动hadoop

     在启动前关闭集群中所有机器的防火墙，不然会出现datanode开后又自动关闭。使用下面命令启动。

     start-all.sh 

     通过启动日志查看，启动namenode 接着启动datanode1，datanode2，…，然后启动secondary namenode。再启动yarn daemons,然后resourcemanager，最后是nodemanager1、2、3

     启动 hadoop成功后，在 Master 中的 tmp 文件夹中生成了 dfs 文件夹，在Slave 中的 tmp 文件夹中均生成了 dfs 文件夹和 mapred 文件夹。

  3. 用jps验证

     在master输入jps

     显示

     8050 Jps

     7654 ResourceManager

     7340 NameNode

     7500 SecondaryNameNode

     在slave输入jps

     显示

     8725 Jps

     8471 DataNode

     8556 NodeManager

  4. 2）验证方式二：用"hadoop dfsadmin -report"

     用这个命令可以查看Hadoop集群的状态。

#### 参考

>  Hadoop入门基础教程 http://www.linuxidc.com/Linux/2016-02/128149.htm 
>
>  Hadoop2.x 让你真正明白yarn http://dataunion.org/27202.html) http://dataunion.org/27202.html
>
>   一步步教你Hadoop多节点集群安装配置http://www.linuxidc.com/Linux/2015-03/114669p4.htm
>
>  