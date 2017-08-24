Mongodb介绍
- Mongodb是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统，属于NoSQL。
- 在高负载的情况下，可以添加更多的节点，可以保证服务器性能。
- MongoDB旨在为WEB应用提供可扩展的高性能数据存储解决方案。
- MongoDB将数据存储为一个文档，数据结构由键值(key-value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。


Mongodb安装
- 搭建yum源
 创建mongodb的yum源
 vim /etc/yum.repos/mongodb-org-3.0.repo加入yum内容(可到mongodb官网查看)
 安装 yum install -y mongodb-org
 编辑配置文件 vim /etc/mongod.conf
 fork:true
 pidFilePath:/var/run/mongodb/mongod.pid
 把这两行#后面注释删掉，否则重启的时候会有问题
 要想绑定多个ip，在bind_ip后写多个ip，中间用逗号分隔，监听全部ip留空即可

最好执行下面命令，防止报错
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
vim /etc/security/limits.conf加入
mongod soft nofile 64000
mongod hard nofile 64000
mongod soft nproc 32000
mongod hard nproc 32000
启动：systemctl start mongod.service
启动过程会比较慢，这是它在写数据 /var/lib/mongo

在本机可以直接运行命令mongo进入到mongodb shell中
如果mongodb监听端口并不是默认的27017，则在连接的时候需要加 --port 选项，例如
mongo --port 27018
连接远程mongodb，需要加--host，例如
mongo --host 127.0.0.1
如果设置了验证，则在连接的时候需要带用户名和密码
mongo -uusername -ppasswd  #这个和MySQL挺像


Mongodb备份与恢复
- 备份指定库
mongodbdump -h ip -d dbname -o dir  //-h 后面跟服务器ip，-d后面跟database名字，不加则备份所有库，-o后指定备份到哪里，它是一个目录
- 备份所有库
 mongodump -h ip -o dir
- 备份指定集合
 mongodump -d mydb -c testc -o /tmp/testc.json   //-o后面跟的是一个文件名字
 
- 恢复所有库
 mongorestore --drop dir/  #其中dir是备份所有库的目录名字，其中--drop可选，意思是当恢复之前先把之前的数据删除，不建议使用
- 恢复指定库
 mongorestore -d mydb dir/ #-d跟要恢复的库名字，dir就是该库备份时所在的目录
- 恢复集合
 mongorestore -d mydb -c testc dir/mydb/testc.bson #-c后面跟要恢复的集合名字，dir是备份mydb库时生成文件所在路径，这里是一个bson文件的路径
- 导入集合
 mongoimport -d mydb -c testc --file /tmp/testc.json
