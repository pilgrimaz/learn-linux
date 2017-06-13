#### Mysql忘记root密码怎么办

如果忘记root密码或者其他用户密码，不要急，按下面操作即可。

1. 编辑mysql主要配置文件 my.cnf

   `vim /etc/mycnf`

   在[mysqld]字段下添加参数 skip-grant

2. 重启数据库服务

   `service mysqld restart`

3. 这样就可以进入数据库不用授权了

   `mysql -uroot`

4. 修改相应用户密码

   `use mysql`

   `update user set password=password('your password')where user='root';`

   `flush privileges;`

5. 修改/etc/my.cnf去掉skip-grant,重启mysql服务