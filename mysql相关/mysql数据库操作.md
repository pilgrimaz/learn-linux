###数据库###
- 创建数据库   `create database 数据库名;`
- 查看数据库   `show databases;`
- 删除数据库   `drop database 数据库名;`
- 查看数据库支持的引擎  `show engines;`
- 查看默认的存储引擎  `show variables like 'storage_engine';`
####表####
- 创建表的语法格式  
  ```
  create table 表名 (属性名 数据类型[完整性约束条件],
                    属性名 数据类型[完整性约束条件],
                   ·
                   ·
                   ·
                   属性名 数据类型
                   );
  ```
  完整性约束条件：
  -  __主键__        
       单字段主键     `属性名 数据类型 primary key`
       多字段主键   属性定义完后统一设置  `primary key(属性名1，属性名2，...属性名n)`
  - __外键__
      ```
      constraint 外键别名 foreign key(属性1.1,属性1.2,...,属性1.n)
              references 表名(属性2.1,属性2.2,...,属性2.n)
      ```
  - __非空__   `属性名 数据类型 not null`
  - __唯一__   `属性名 数据类型 unique`
  - __自动增加__  `属性名 数据类型 auto_increment`
  - __默认值__  `属性名 数据类型 default 默认值`
- 查看表结构  `describe(desc) 表名`
- 查看表详细结构语句  `show create table`
- 修改表
  - 修改表名  `alter table 旧表名 rename [to] 新表名;`
  - 修改字段的数据类型  `alter table 表名 modify 属性名 数据类型;`
  - 修改字段名和数据类型  `alter table 表名 change 旧属性名 新属性名 新数据类型`
  - 增加字段  `alter table 表名 add 属性名1 数据类型 [完整性约束条件] [first]|[after] 属性名2;` 
  - 删除字段  `alter table 表名 drop 属性名;`
  - 修改字段的排列位置  `alter table 表名 modify 属性名1 数据类型 first|after 属性名2;`
  - 更改表的存储引擎  `alter table 表名 engine=存储引擎名;`
  - 删除表的外键约束  `alter table 表名 drop foreign key 外键别名`
  - 删除表(删除关联的父表需先删除外键依赖)  `drop table 表名`
####索引####
- 创建索引
    - 创建表的时候创建索引,属性后加上   `[unique|fulltext|spatial] index|key  索引名(属性1 [(长度)]) [ASC|DESC]`
      unique 唯一性索引   fulltext 全文索引   spatial 空间索引  不选为普通索引
      ASC 升序排列   DESC 降序排列
    - 在已经存在的表上创建索引   `create [unique|fulltext|spatial] index 索引名 on 表名(属性名 [(长度)]) [ASC|DESC]);`
    - 用alter table创建索引   `alter table 表名 add [unique|fulltext|apatial] index 索引名(属性名 [(长度)] [ASC|DESC]);`
- 删除索引   `drop index 索引名 on 表名;`
#### 视图####

-  创建视图 `create [algorithm={undefined|merge|temptable}] view 视图名[(属性清单)] as select语句 [with {cascaded|local} check option];`
  undefined 自动选择语法  
  merge 将视图语句与视图定义合并起来，使视图定义的某一部分取代语句对应部分   
  temptable  将视图的结果存入临时表，使用临时表执行语句
  cascaded  表示更新视图时要满足所有相关视图和表的条件
  local  表示更新视图时，要满足该视图本身的定义的条件即可。
- 查看视图 `desc 视图名`
- 查看视图基本信息  `show table status like '视图名'`
- 查看视图详细信息  `show create view 视图名`
  视图信息都存储在information_schema数据库下的views表中
- 修改视图
  - create or replace view 创建或修改视图
    ```
    create or replace [algorithm={undefined|merge|temptable}] view 视图名 [(属性清单)] as select语句 [with {cascaded|local} check option];
    ```
  - alter语句修改视图
    ```
    alter [algorithm={undefined|merge|temptable}] view 视图名 [(属性清单)] as select语句 [with {cascaded|local} check option];
    ```
- 删除视图  `drop view [if exists] 视图名列表 [restrict|cascade]`
####触发器####
- 创建触发器  
  - 创建只有一个执行语句的触发器  `create trigger 触发器名 before|after 触发事件 on 表名 for each row 执行语句`
  - 创建有多个执行语句的触发器  
    ```
    create trigger 触发器名 before|after 触发事件
    on 表名 for each row
    begin
         执行语句列表
    end
    ```
    mysql语句默认是以 ";"结束，在创建触发器过程中需要用到 ";",为了解决这个问题，可以用delimiter语句，如 "delimiter &&"，可以将结束符号变成 "&&"，当触发器创建完成后，可以用命令 "delimiter ;"来讲结束符号变成 ";".
- show triggers查看触发器信息 `show triggers\G`
- 在triggers表中查看触发器信息  `select * from information_schema.triggers;`
  触发器信息存储在 information_schema数据库的triggers表中
- 删除触发器  `drop trigger 触发器名`
