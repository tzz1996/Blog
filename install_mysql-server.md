#Windows下配置mysql-server-5.6
- 下载[mysql-server](http://dev.mysql.com/downloads/mysql/5.6.html#downloads)

##**安装步骤:**

###1. 配置my-default.ini文件

- 添加以下内容： 
```
[mysqld] 
character-set-server = utf8
basedir = 磁盘:/文件名/.../mysql-5.6.30-win64    
datadir = 磁盘:/文件名/.../mysql-5.6.30-win64/data
port = 3306   
[client]   
default-character-set = utf8    
```
- 配置完成后将文件改名为my.ini。
>注意:
>若显示无法修改此文件，则需要修改mysql-5.6.30-win64权限。

 - 修改方法：  
   右键选择安装目录属性->安全，选择Users，编辑并修改权限。  


###2.安装并启动mysql服务
- 打开命令提示符(admin):    
`cd 磁盘:/文件名/.../mysql-5.6.30-win64/bin`  
`mysqld -install`  
 - 若mysql安装成功，则显示：  
  `Service successfully installed`
- 启动mysql服务：  
 - 命令行方法：  
`net start mysql`    (停止服务命令：`net stop mysql`)  
 - 图形界面方法：  
  1.开始菜单->运行.  
  2.输入services.msc打开服务管理设置.  
  3.启动mysql服务.
- 登陆root用户并修改密码(命令行)：  
```
mysql -u root
mysql>use mysql;    
mysql>update user set password=password('yournewpassword') where user='root'; 
mysql>flush privileges; 
```
再`mysql -u root -p`并输入密码即可.  


###3.issues
- 若修改密码后登陆root用户出现Error 1045:Access Denied for user 'root'@'localhost',则需要重新设置密码。  
  
 - 设置方法：  
1.修改安装目录的my-default.ini文件(添加)：  
`[mysqld]`  
`skip-grant-tables`  
2.命令行中执行：  
`mysql -u root mysql`  
`mysql>update user set password=password('yournewpassword') where user='root';`  
`mysql>flush privileges;`  
3.去掉my-default.ini文件中的`skip-grant-tables`  
4.重启mysql服务：  
`net stop mysql`  
`net start mysql`  
5.重新登陆mysql即可：  
`mysql -u root -p`  

- 若用新建用户登录出现Error 1045 (28000):Access Denied for user 'yourusername'@'localhost' (using password: YES).

 - 解决办法：需用root用户登录，删除mysql数据库中user表里的host=''@'localhost'的用户([具体原因](http://stackoverflow.com/questions/10299148/mysql-error-1045-28000-access-denied-for-user-billlocalhost-using-passw))

- 可将 '磁盘:/文件名/.../mysql-5.6.30-win64/bin' 添加到系统环境变量'Path'中，即可在任意路径下登陆mysql。

**至此，windows下mysql-server配置完成.**
    



