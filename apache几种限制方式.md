#### apache几种限制方式

1. 禁止访问某些文件目录

   增加_Files_选项来控制，比如要不允许访问_.inc_扩展名的文件，保护php类库：

   `<File ~"\.insc$">`

2. 禁止访问某些指定的目录：（可以用_<DirectoryMatch>_来进行正则匹配）

   `<Directory ~"/var/www/(.+)*[0-9]{3}">`

   当然也可以写目录全局路径

   `<Directory /var/www/111>`

3. 通过文件匹配来进行禁止，比如禁止所有针对图片的访问

   `<Filesmatch (*.)php>`

4. 针对URL相对路径的禁止访问

   `<Location /dir/>`