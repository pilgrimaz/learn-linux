#### nginx 访问需要用户验证

1 首先需要安装apache，可以使用yum install 安装

2 生成密码文件，创建用户

htpasswd -c /usr/local/nginx/conf/htpasswd  test // 添加test用户，第一次添加时需要加-c参数，第二次添加时不需要-c参数

3 在nginx的配置文件中添加
```shell
location  / {
​                      root /data/www/wwwroot/count;
​                      auth_basic              "Auth";
​                      auth_basic_user_file   /usr/local/nginx/conf/htpasswd;
​            }								
```
